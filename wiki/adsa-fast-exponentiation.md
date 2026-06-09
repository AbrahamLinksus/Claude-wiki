# Fast Exponentiation, Floor/Ceil & Mod Tricks

**Summary**: Binary exponentiation reduces a^b from O(b) multiplications to O(log b) by squaring repeatedly; combined with integer floor/ceil formulas and bitwise mod tricks, these techniques are the performance backbone of cryptography, hash tables, and competitive programming.

**Pre-Req**: Basic loops, binary number representation. [[adsa-number-theory]] — modular arithmetic properties.

**Sources**: `raw/3 - Fast-Exponentiation-FloorCeilMod-Tricks.pdf`

**Last updated**: 2026-06-09

---

## The Core Problem

Computing a^b naively means multiplying b times:

```
3^13 = 3×3×3×3×3×3×3×3×3×3×3×3×3   (12 multiplications)
```

For b = 2^1024 (a 1024-bit RSA exponent), this is 10^308 multiplications. A computer doing 10^9 multiplications per second would need 10^299 years. The universe is only ~14 billion (10^10) years old.

Binary exponentiation does the same computation in ~1500 multiplications. That is the difference between impossible and instant.

---

## Part 1 — Binary Exponentiation

### The Key Insight

Every number can be expressed in binary. Use the binary representation of the exponent to decide what to square and what to multiply.

```
13 in binary = 1101

13 = 8 + 4 + 0 + 1 = 2^3 + 2^2 + 2^0

So:  a^13 = a^(8+4+1) = a^8 × a^4 × a^1

We can compute a^1, a^2, a^4, a^8 by repeated squaring:
  a^1  = a
  a^2  = (a^1)^2
  a^4  = (a^2)^2
  a^8  = (a^4)^2

Then multiply together only the powers where the binary bit is 1.
```

This takes at most **2 × log2(b)** multiplications (log2 squarings + at most log2 multiplications).

### Full Dry Run: 3^13 mod 10

```
b = 13 = 1101 in binary (read right to left: bits are 1, 0, 1, 1)

Initialization:
  result = 1
  base   = 3

Iteration 1 (bit 0 of 13 = 1):
  bit is 1  → result = result × base = 1 × 3 = 3
  base = base^2 = 3^2 = 9
  exp  = 13 >> 1 = 6

Iteration 2 (bit 0 of 6 = 0):
  bit is 0  → skip (don't multiply into result)
  base = 9^2 = 81   [→ mod 10 = 1 if doing modular]
  exp  = 6 >> 1 = 3

Iteration 3 (bit 0 of 3 = 1):
  bit is 1  → result = result × base = 3 × 81 = 243
  base = 81^2 = 6561
  exp  = 3 >> 1 = 1

Iteration 4 (bit 0 of 1 = 1):
  bit is 1  → result = result × base = 243 × 6561 = 1,594,323
  base = 6561^2 = ...
  exp  = 1 >> 1 = 0

exp = 0, stop.

3^13 = 1,594,323.

Verify: 3^13 = 3^8 × 3^4 × 3^1
       = 6561 × 81 × 3
       = 531441 × 3
       = 1,594,323  ✓

Total multiplications: 4  (vs. 12 naïve)
```

### Modular Version: 3^13 mod 10

In practice we reduce mod m at every step to keep numbers small:

```
result = 1,  base = 3,  mod = 10

Iter 1: exp=13 (odd)  → result = (1×3) % 10 = 3;   base = (3×3)%10 = 9;   exp=6
Iter 2: exp=6  (even) → skip;                        base = (9×9)%10 = 1;   exp=3
Iter 3: exp=3  (odd)  → result = (3×1) % 10 = 3;    base = (1×1)%10 = 1;   exp=1
Iter 4: exp=1  (odd)  → result = (3×1) % 10 = 3;    base = (1×1)%10 = 1;   exp=0

3^13 mod 10 = 3  ✓

Intermediate values never exceed mod^2 = 100 — no big integers needed!
```

### Implementation

```python
def fast_pow(base: int, exp: int) -> int:
    """Compute base^exp in O(log exp) multiplications."""
    result = 1
    while exp > 0:
        if exp & 1:          # if lowest bit is 1
            result *= base
        base *= base
        exp >>= 1            # shift right (divide by 2, floor)
    return result

def mod_pow(base: int, exp: int, mod: int) -> int:
    """Compute (base^exp) % mod in O(log exp) multiplications."""
    result = 1
    base %= mod              # reduce base first
    while exp > 0:
        if exp & 1:
            result = (result * base) % mod
        base = (base * base) % mod
        exp >>= 1
    return result

# Examples:
# fast_pow(3, 13)       → 1,594,323
# mod_pow(3, 13, 10)    → 3
# mod_pow(65, 17, 3233) → 2790  (RSA encryption: n=3233=61×53, e=17)
# pow(65, 17, 3233)     → 2790  (Python built-in, equally good)
```

**In Python:** `pow(base, exp, mod)` with three arguments is the built-in equivalent — highly optimised in C. Use it in production (source: 3 - Fast-Exponentiation-FloorCeilMod-Tricks.pdf).

### Counting the Multiplications

```
For exponent b:
  Number of squarings:    floor(log2(b))
  Number of multiplies:   popcount(b)  [number of 1-bits in b]
  Total:                  floor(log2(b)) + popcount(b)

Examples:
  b = 13 = 1101:   floor(log2(13)) = 3 squarings + popcount(1101) = 3 mult = 6 total
  b = 2^100:       100 squarings + 1 mult = 101 total  (naïve: 2^100 ≈ 10^30)
  b = 2^1024:      ~1500 total  (naïve: 10^308 — impossible)
```

### Why exp & 1 Works

The bitwise AND with 1 checks if the lowest bit is set (i.e., whether exp is odd):

```
exp = 13 = 1101 in binary
  1:         0001 in binary
13 & 1:      0001 = 1  → odd!

exp = 6 = 0110 in binary
  1:         0001 in binary
 6 & 1:      0000 = 0  → even!
```

And `exp >>= 1` shifts right, which is integer division by 2 (floor). These bit operations are extremely fast.

---

## Part 2 — Floor and Ceil Integer Arithmetic

### Definitions

**Floor(a/b):** the largest integer ≤ a/b. "Round down toward negative infinity."

```
floor(7/2)   = 3    (7/2 = 3.5, floor = 3)
floor(8/2)   = 4    (exact, no rounding)
floor(-7/2)  = -4   (floor rounds toward -∞, not toward 0!)
```

**Ceil(a/b):** the smallest integer ≥ a/b. "Round up."

```
ceil(7/2)    = 4    (7/2 = 3.5, ceil = 4)
ceil(8/2)    = 4    (exact, no rounding)
ceil(-7/2)   = -3   (ceil rounds toward +∞)
```

### Computing Floor and Ceil with Integer Arithmetic

Avoid floating-point entirely — it can give wrong answers for large integers due to precision limits.

**Floor:** in Python (and most languages), `a // b` is floor division for positive a, b.

```python
a // b     # floor division (Python)
a / b      # FLOAT — may be inaccurate for large a, b!

7 // 2 = 3    ✓
8 // 2 = 4    ✓
```

**Ceil — Formula 1 (standard, may overflow for large a, b):**

```
ceil(a / b) = (a + b - 1) // b

Proof:
  If b divides a:       a/b is exact, (a + b - 1) = a + (b-1), divided by b gives
                        a/b + (b-1)/b = a/b (since b-1 < b, floor removes it)
  If b does not divide a: a = q*b + r where 1 <= r <= b-1
                           (a + b - 1) = q*b + r + b - 1 = (q+1)*b + (r-1)
                           floor = q + 1 = ceil(a/b) ✓

Example: ceil(17/5) = (17 + 5 - 1) // 5 = 21 // 5 = 4
  Verify: 17/5 = 3.4, ceil = 4 ✓

Example: ceil(15/5) = (15 + 5 - 1) // 5 = 19 // 5 = 3
  Verify: 15/5 = 3.0, ceil = 3 ✓
```

**Ceil — Formula 2 (safer, no overflow risk for positive a):**

```
ceil(a / b) = (a - 1) // b + 1    [when a > 0]

Example: ceil(17/5) = (17-1)//5 + 1 = 16//5 + 1 = 3 + 1 = 4  ✓
Example: ceil(15/5) = (15-1)//5 + 1 = 14//5 + 1 = 2 + 1 = 3  ✓
```

Use Formula 2 when a + b - 1 might overflow a 64-bit integer (when both a and b are near 2^63 - 1).

### Worked Examples

```
How many pages of 5 items do you need for n items?

n = 17:  ceil(17/5) = (17+4)//5 = 21//5 = 4 pages
n = 20:  ceil(20/5) = (20+4)//5 = 24//5 = 4 pages
n = 21:  ceil(21/5) = (21+4)//5 = 25//5 = 5 pages

How many groups of 8 servers for 100 machines?
ceil(100/8) = (100+7)//8 = 107//8 = 13 groups

How many memory blocks of size B needed for array of size n?
ceil(n/B) = (n + B - 1) // B   [overflow-safe when n, B are small]
          = (n - 1) // B + 1   [always overflow-safe for n > 0]
```

### Floor/Ceil Comparison

```
+------------------------+-------+-----------+----------------------+
| Approach               | Speed | Precision | Safe for large ints? |
+------------------------+-------+-----------+----------------------+
| Integer floor: a // b  | Fast  | Exact     | Yes                  |
| math.floor(a / b)      | Slow  | Float!    | No (large int risk)  |
| Ceil: (a+b-1) // b     | Fast  | Exact     | May overflow if a+b-1|
|                        |       |           | exceeds int limit    |
| Safer ceil: (a-1)//b+1 | Fast  | Exact     | Yes (for a > 0)      |
+------------------------+-------+-----------+----------------------+
```

---

## Part 3 — Modular Arithmetic Tricks

### The Mod Operation

`a mod m` is the remainder when a is divided by m. Always lies in [0, m-1].

```
17 mod 5 = 2    (17 = 3×5 + 2)
20 mod 5 = 0    (20 = 4×5 + 0)
 3 mod 5 = 3    (3  = 0×5 + 3)
```

### Four Core Properties

These let you reduce numbers at every step during computation:

```
Property 1 — Addition:
  (a + b) mod m = ((a mod m) + (b mod m)) mod m

  Example: (1000 + 999) mod 7
    = (1000 mod 7 + 999 mod 7) mod 7
    = (6 + 5) mod 7
    = 11 mod 7 = 4
  Direct: 1999 mod 7 = 4 ✓

Property 2 — Subtraction:
  (a - b) mod m = ((a mod m) - (b mod m) + m) mod m
  (+m prevents negative results)

Property 3 — Multiplication:
  (a × b) mod m = ((a mod m) × (b mod m)) mod m

  Example: (1000 × 999) mod 7
    = (6 × 5) mod 7
    = 30 mod 7 = 2
  Direct: 999000 mod 7 = 2 ✓

Property 4 — Exponentiation:
  (a^k) mod m = ((a mod m)^k) mod m
  (Combined with binary exponentiation, this keeps numbers tiny)
```

### Negative Modulo Fix

In Python, `x % m` always returns a non-negative result. In C and Java, it follows the sign of the dividend.

```
Python:  -7 % 5 = 3    (correct modulo)
C/Java:  -7 % 5 = -2   (truncation remainder — NOT modulo!)

True modulo formula (works in all languages):
  true_mod(x, m) = ((x % m) + m) % m

true_mod(-7, 5) = ((-7 % 5) + 5) % 5 = (-2 + 5) % 5 = 3 ✓
true_mod(-1, 12) = ((-1 % 12) + 12) % 12 = 11  (11 o'clock on a 12-hr clock) ✓
```

```python
def true_mod(x: int, m: int) -> int:
    return ((x % m) + m) % m
```

### Bitwise AND for Power-of-Two Modulo

When the modulus m is a power of 2 (m = 2^k), bitwise AND is ~2× faster than division:

```
x mod m = x & (m - 1)     [only when m = 2^k]

Why it works:
  m = 2^k means m in binary is 1 followed by k zeros: 10...0
  m - 1 in binary is k ones:                           01...1
  x & (m-1) keeps only the lower k bits of x
  Those lower k bits ARE the remainder when divided by 2^k

Example: m = 16 = 2^4
  x = 37 = 100101 in binary
  15 = 001111 in binary
  37 & 15 = 000101 = 5
  37 mod 16 = 37 - 2×16 = 5  ✓

Example: m = 1024 = 2^10
  x = 5000 = 1001110001000 in binary
  1023 = 0001111111111 in binary
  5000 & 1023 = 0001110001000 ... wait, let's compute:
  5000 mod 1024 = 5000 - 4×1024 = 5000 - 4096 = 904
  5000 & 1023:  5000 = 4096+512+256+128+8 = 0001001110001000
               1023 = 0000001111111111
               AND  = 0000001110001000 = 512+256+128+8 = 904  ✓
```

**Hash table application:**

```python
# Standard modulo (works for any size):
bucket = hash_value % table_size

# Bitwise AND (only when table_size is a power of 2 — ~2× faster):
bucket = hash_value & (table_size - 1)

# This is why hash tables are typically sized as powers of 2!
# Java HashMap, Python dict internals use this trick.
```

Performance benefit: ~2x faster in tight loops. At 100 million lookups per second, the difference is 50ms vs 100ms (source: 3 - Fast-Exponentiation-FloorCeilMod-Tricks.pdf).

---

## Part 4 — Putting It Together: RSA Use Case

RSA decryption computes: `m = c^d mod n`

Where d is a 2048-bit number. Naïve exponentiation is impossible. Binary mod_pow makes it instantaneous:

```
RSA-2048 decryption:
  d has 2048 bits
  Binary exponentiation: ~2048 squarings + ~1024 multiplications ≈ 3072 ops
  Each operation: multiply two 2048-bit numbers ≈ microseconds
  Total: ~1ms per decryption

  Without fast expo: 2^2048 multiplications ≈ 10^616 ops
                     Impossible in any finite time.

100,000 TLS connections/second means 100,000 such decryptions/second.
This is only achievable because of O(log b) fast exponentiation.
(source: 3 - Fast-Exponentiation-FloorCeilMod-Tricks.pdf)
```

---

## Part 5 — Algorithm Comparison

### Exponentiation Methods

```
+----------------------+------------------+---------------+-------------------+
| Method               | Multiplications  | Constant-time?| Use Case          |
+----------------------+------------------+---------------+-------------------+
| Naive loop           | b                | Yes           | b <= 10           |
| Binary right-to-left | ~1.5 log2(b)     | No (branches) | General non-crypto|
| Binary left-to-right | ~1.5 log2(b)     | Yes (cmov)    | Crypto            |
| k-ary (k=4)          | log2(b) + 2^k    | No            | Optimised big-int |
| Sliding window       | log2(b) + small  | No            | OpenSSL           |
+----------------------+------------------+---------------+-------------------+
```

**Crypto warning:** Right-to-left binary expo branches on each bit of the secret exponent. An attacker measuring execution time can infer bits of the private key (**timing attack**). Left-to-right with conditional moves (or using blinding) prevents this.

### Decision Tree: Which Method?

```
Need modular exponent?
├── Yes → use pow(base, exp, mod)   [C-optimised, often constant-time]
└── No → is exponent large (> 1000)?
    ├── Yes → use binary exponentiation (fast_pow)
    └── No  → naive loop is fine
```

---

## Part 6 — Common Pitfalls

```
+-----------------------------------------+--------------------------------------+
| Mistake                                 | Fix                                  |
+-----------------------------------------+--------------------------------------+
| Using loop for large exponent           | Program hangs; use binary expo       |
| % on negatives in C/Java gives negative | Use ((x%m)+m)%m for true modulus     |
| Off-by-one in ceil formula              | Use (a+b-1)//b or (a-1)//b+1         |
| pow(base,exp) then % mod                | Causes huge intermediates;           |
|                                         | use pow(base, exp, mod) 3-arg form   |
| Using math.floor(a/b) for large ints    | Use a//b (integer division)          |
| Right-to-left expo in crypto w/o blind  | Timing attack; add blinding factor   |
| Not reducing base before mod_pow        | base %= mod at start; prevents slow  |
|                                         | multiplication of large intermediates|
| Overflow in ceil (a+b-1)                | Use (a-1)//b+1 when a or b near max  |
+-----------------------------------------+--------------------------------------+
```

**Production incident:** Using `math.floor(a / b)` for large integers can give wrong answers because Python floats have 53-bit mantissa precision. For integers > 2^53, use integer floor division `a // b` instead (source: 3 - Fast-Exponentiation-FloorCeilMod-Tricks.pdf).

---

## Quick Reference: All Formulas

```
+----------------------------+----------------------------------+
| Operation                  | Formula / Code                   |
+----------------------------+----------------------------------+
| Binary exponentiation      | while b: if b&1: r*=a; a*=a;    |
|                            |          b>>=1                   |
| Floor division             | a // b                           |
| Ceil (standard)            | (a + b - 1) // b                 |
| Ceil (overflow-safe)       | (a - 1) // b + 1                 |
| True modulus (negative x)  | ((x % m) + m) % m                |
| Mod by power-of-two        | x & (m-1)   [m = 2^k only]      |
| Modular exponentiation     | pow(base, exp, mod)              |
| Check if exp bit is set    | exp & 1                          |
| Shift right (divide by 2)  | exp >>= 1                        |
+----------------------------+----------------------------------+

Memory aid: "Square and multiply, bits decide;
             ceil add divisor minus one divide."
```

---

## Summary Table

```
+-------------------+---------------------------+-----------------------------+
| Concept           | Key fact                  | Application                 |
+-------------------+---------------------------+-----------------------------+
| Fast expo         | O(log b) vs O(b)          | RSA, modular inverse        |
| Modular expo      | reduce at every step      | Crypto, large exponents     |
| Floor div         | a // b                    | Integer arithmetic          |
| Ceil div          | (a-1)//b+1                | Pagination, memory blocks   |
| Negative mod      | ((x%m)+m)%m               | Index wrapping, ring buffer |
| Bitwise mod trick | x & (m-1)                 | Hash tables, buffers        |
| Timing attack     | Branch on secret bit leaks| Use constant-time or blind  |
+-------------------+---------------------------+-----------------------------+
```

#adsa
