# Primes, GCD & Modular Arithmetic

**Summary**: The three pillars of computational number theory — prime numbers, greatest common divisors, and modular arithmetic — that underpin every modern cryptographic system, hash table, and checksum algorithm.

**Pre-Req**: Basic algebra, loops & recursion, [[adsa-overview]] — Big-O notation fundamentals.

**Sources**: `raw/1 - Primes-GCD-and-Modular-Arithmetic.pdf`

**Last updated**: 2026-06-09

---

## Why This Exists

Imagine you need to secure a message so that only one person can read it. The trick: multiply two enormous prime numbers together. The product is easy to compute, but reversing it — factoring that giant number back into its primes — would take classical computers billions of years. That asymmetry is the bedrock of RSA encryption, which protects over 95% of HTTPS connections on the internet today (source: 1 - Primes-GCD-and-Modular-Arithmetic.pdf).

Three tools make this possible:

```
+------------------+--------------------------------------------+
| Tool             | What it does                               |
+------------------+--------------------------------------------+
| Prime numbers    | Multiplicative atoms — every integer       |
|                  | factors uniquely into primes               |
| GCD (Euclid)     | Finds shared factors in O(log n) time      |
| Modular arith.   | Keeps numbers bounded — "clock arithmetic" |
+------------------+--------------------------------------------+
```

Without primes: no asymmetric cryptography.
Without efficient GCD: key generation is impractical.
Without modular arithmetic: numbers grow to astronomical size.

---

## Part 1 — Prime Numbers

### What is a Prime?

A prime number is an integer greater than 1 with exactly **two** divisors: 1 and itself.

```
Primes:     2,  3,  5,  7, 11, 13, 17, 19, 23, 29 ...
Not primes: 4 = 2×2,  6 = 2×3,  9 = 3×3,  15 = 3×5
1 is NOT prime — it has only one divisor (itself)
```

The **Fundamental Theorem of Arithmetic**: every integer >= 2 factors uniquely into primes.

```
Example:  360 = 2^3 × 3^2 × 5
          There is no other way to factor 360 into primes.
```

### Trial Division — The Naive Test

To check if n is prime, try dividing by every integer from 2 up to sqrt(n).

Why sqrt(n)? If n has a factor d > sqrt(n), then n/d < sqrt(n), so the smaller factor would have been found first.

```
Is 37 prime?
sqrt(37) ≈ 6.08 → test divisors 2, 3, 4, 5, 6

37 / 2 = 18.5  (no)
37 / 3 = 12.3  (no)
37 / 4 = 9.25  (no)
37 / 5 = 7.4   (no)
37 / 6 = 6.16  (no)

No divisors found → 37 is prime!
```

Complexity: O(sqrt(n)). Fine for n < 10^12, too slow for 2048-bit RSA numbers.

### Sieve of Eratosthenes — Finding All Primes Up To n

Instead of testing each number individually, cross out multiples in bulk.

```
Find all primes up to 30:

Step 1: Write 2 through 30
  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30

Step 2: 2 is prime. Cross out all multiples of 2: 4,6,8,10,...
  2  3  X  5  X  7  X  9  X 11  X 13  X 15  X 17  X 19  X 21  X 23  X 25  X 27  X 29  X

Step 3: 3 is the next unmarked. Cross out multiples of 3: 9,15,21,27,...
  2  3  X  5  X  7  X  X  X 11  X 13  X  X  X 17  X 19  X  X  X 23  X 25  X  X  X 29  X

Step 4: 5 is next. Cross out multiples of 5: 25 (others already crossed)
  2  3  X  5  X  7  X  X  X 11  X 13  X  X  X 17  X 19  X  X  X 23  X  X  X  X  X 29  X

Step 5: sqrt(30) ≈ 5.5. We've reached sqrt(n). Done!

Result: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29
```

**Why stop at sqrt(n)?** Any composite number left unmarked must have a prime factor > sqrt(n), but then its cofactor would be < sqrt(n) and would have been used to cross it out already.

```python
def sieve_of_eratosthenes(n: int) -> list[int]:
    """Return list of all primes <= n using O(n log log n) sieve."""
    if n < 2:
        return []
    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False
    p = 2
    while p * p <= n:
        if is_prime[p]:
            for multiple in range(p * p, n + 1, p):
                is_prime[multiple] = False
        p += 1
    return [i for i, prime in enumerate(is_prime) if prime]

# sieve_of_eratosthenes(30) → [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

Complexity: **O(n log log n)** time, O(n) space. The double-log factor is nearly linear in practice.

### Miller-Rabin Probabilistic Primality Test

Trial division is deterministic but slow. For large numbers (RSA-scale: 2048-bit), we need a faster test that is "almost certainly" correct.

**The setup:** Write n-1 = 2^s * d where d is odd. (Factor out all the 2s from n-1.)

```
Example: n = 221
n - 1 = 220 = 4 × 55 = 2^2 × 55
So s = 2, d = 55
```

**One round of Miller-Rabin:**

```
Input: n (the number to test, n > 2 and odd)
       a (a random base, 2 <= a <= n-2)

1. Write n-1 = 2^s * d  (d odd)
2. Compute x = a^d mod n
3. If x == 1 or x == n-1: this round passes (probably prime)
4. Repeat s-1 times:
      x = x^2 mod n
      if x == n-1: this round passes
5. If we never got 1 or n-1: n is COMPOSITE (certain)
```

If n is truly prime, every base must pass. If n is composite, most bases will expose it.

**Error rate:** Each round has at most 1/4 chance of a composite sneaking through (Rabin's theorem). With 40 rounds, error probability < (1/4)^40 = 2^-80. For 64-round crypto use: error < 2^-128.

```
Dry run: Is n = 221 prime?

n-1 = 220 = 2^2 × 55, so s=2, d=55
Choose a = 174

x = 174^55 mod 221 = 47

Round 0: x = 47, not 1 or 220
Square: x = 47^2 mod 221 = 2209 mod 221 = 220 = n-1  → PASS

One base passed, but try more bases to be confident.
(221 = 13 × 17, so it IS composite — other bases reveal it)
```

**Deterministic shortcut for small n:** For n < 3,215,031,751, testing only bases {2, 3, 5, 7} is always correct (no probabilistic error).

### Prime Number Variants

```
+------------------+----------------------------------+------------------------+
| Type             | Definition                       | Use Case               |
+------------------+----------------------------------+------------------------+
| Probable prime   | Passes Miller-Rabin (k rounds)   | RSA key generation     |
| Strong prime     | p-1 and p+1 have large factors   | Secure RSA             |
| Safe prime       | p = 2q + 1, q prime              | Diffie-Hellman groups  |
| Mersenne prime   | 2^p - 1, p prime                 | Large prime research   |
+------------------+----------------------------------+------------------------+
```

---

## Part 2 — GCD: Greatest Common Divisor

### What is GCD?

GCD(a, b) is the largest positive integer that divides both a and b without remainder.

```
GCD(12, 18):
Divisors of 12: 1, 2, 3, 4, 6, 12
Divisors of 18: 1, 2, 3, 6, 9, 18
Common:         1, 2, 3, 6
Largest:        6

So GCD(12, 18) = 6
```

If GCD(a, b) = 1, we call a and b **coprime** (or relatively prime). This is critical for modular inverses.

### Euclidean Algorithm

The naive approach (listing all divisors) is O(n). The Euclidean algorithm achieves O(log min(a,b)).

**Core insight:** GCD(a, b) = GCD(b, a mod b)

Why? Any common divisor of a and b also divides a - b, so it divides a mod b. The GCDs are identical.

```
Base case: GCD(a, 0) = a

Dry run: GCD(270, 192)

Step 1: GCD(270, 192)  →  270 mod 192 = 78    →  GCD(192, 78)
Step 2: GCD(192, 78)   →  192 mod 78  = 36    →  GCD(78, 36)
Step 3: GCD(78, 36)    →  78  mod 36  = 6     →  GCD(36, 6)
Step 4: GCD(36, 6)     →  36  mod 6   = 0     →  GCD(6, 0)
Step 5: GCD(6, 0)  →  return 6

Answer: GCD(270, 192) = 6
```

The numbers decrease by at least half every two steps, so it takes at most 2 * log_phi(min(a,b)) ≈ 2.08 * log2(n) steps.

```python
def gcd(a: int, b: int) -> int:
    while b:
        a, b = b, a % b
    return a

# gcd(270, 192) → 6
```

**Worst case:** Consecutive Fibonacci numbers (GCD = 1, maximum steps).
```
GCD(89, 55): 89→55→34→21→13→8→5→3→2→1→0  (10 steps)
```

### Extended Euclidean Algorithm

Beyond just finding the GCD, the extended version also finds coefficients x, y such that:

```
a*x + b*y = GCD(a, b)    (Bezout's identity)
```

This is essential for computing **modular inverses** (needed for RSA private key computation and division in modular arithmetic).

```
Example: Find x, y such that 3x + 11y = GCD(3, 11) = 1

Extended Euclidean trace:
GCD(11, 3):
  11 = 3*3 + 2      →  2 = 11 - 3*3
  3  = 1*2 + 1      →  1 = 3  - 1*2
  2  = 2*1 + 0      →  done, GCD = 1

Back-substitute:
  1 = 3 - 1*2
    = 3 - 1*(11 - 3*3)
    = 3 - 11 + 3*3
    = 4*3 - 1*11

So x = 4, y = -1  →  3*4 + 11*(-1) = 12 - 11 = 1 ✓
```

The modular inverse of 3 mod 11 is 4 (since 3*4 = 12 ≡ 1 mod 11).

```python
def egcd(a: int, b: int) -> tuple[int, int, int]:
    """Return (g, x, y) s.t. a*x + b*y = g = gcd(a, b)."""
    if b == 0:
        return (a, 1, 0)
    g, x1, y1 = egcd(b, a % b)
    return (g, y1, x1 - (a // b) * y1)

def mod_inverse(a: int, m: int) -> int:
    """Return x s.t. a*x ≡ 1 mod m. Requires gcd(a, m) = 1."""
    g, x, _ = egcd(a, m)
    if g != 1:
        raise ValueError(f"Inverse doesn't exist, gcd={g}")
    return x % m

# mod_inverse(3, 11) → 4  (since 3*4 = 12 ≡ 1 mod 11)
```

### GCD Algorithm Variants

```
+------------------------+------------------+----------------------+
| Variant                | Complexity       | Best For             |
+------------------------+------------------+----------------------+
| Euclidean (subtraction)| O(n) worst-case  | Historical only      |
| Euclidean (modulo)     | O(log min(a,b))  | General purpose      |
| Binary GCD (Stein)     | Shift-based      | Embedded hardware    |
| Extended Euclidean     | O(log min(a,b))  | Modular inverses     |
+------------------------+------------------+----------------------+
```

---

## Part 3 — Modular Arithmetic

### The Clock Analogy

Modular arithmetic is exactly like a clock. A clock has 12 positions (0-11). After 12, you wrap back to 0.

```
10 + 5 = 15  →  15 mod 12 = 3   (three hours past noon = 3pm)
10 + 3 = 13  →  13 mod 12 = 1   (one hour past noon = 1pm)
```

Formally: **a mod m** is the remainder when a is divided by m. It always lies in [0, m-1].

```
17 mod 5  = 2   (17 = 3*5 + 2)
-1 mod 12 = 11  (in proper modular arithmetic — Python gives 11, C gives -1!)
100 mod 7 = 2   (100 = 14*7 + 2)
```

**Negative mod trap:** In Python, `-7 % 5 = 3` (always non-negative). In C/Java, `-7 % 5 = -2` (sign follows dividend). For portable code:

```python
def true_mod(x: int, m: int) -> int:
    return ((x % m) + m) % m

true_mod(-7, 5)  # → 3  (correct in all languages)
```

### Congruence Notation

We write **a ≡ b (mod m)** to mean "a and b have the same remainder when divided by m."

```
17 ≡ 5  (mod 12)   because 17 mod 12 = 5, and 5 mod 12 = 5
24 ≡ 0  (mod 6)    because both are divisible by 6
-1 ≡ 11 (mod 12)   because -1 + 12 = 11
```

### Arithmetic Properties

All standard arithmetic operations preserve congruence. This is what makes computations "in modulo" valid.

```
+----------------------------------+-------------------------------------+
| Property                         | Example (mod 7)                     |
+----------------------------------+-------------------------------------+
| (a + b) mod m = ((a mod m) +     | (10 + 11) mod 7                     |
|                  (b mod m)) mod m|  = (3 + 4) mod 7 = 0 ✓             |
|                                  |  = 21 mod 7 = 0 ✓                  |
+----------------------------------+-------------------------------------+
| (a × b) mod m = ((a mod m) ×     | (10 × 11) mod 7                     |
|                  (b mod m)) mod m|  = (3 × 4) mod 7 = 12 mod 7 = 5 ✓  |
|                                  |  = 110 mod 7 = 5 ✓                  |
+----------------------------------+-------------------------------------+
| (a^k) mod m =                    | 10^3 mod 7                          |
| ((a mod m)^k) mod m              |  = 3^3 mod 7 = 27 mod 7 = 6 ✓      |
|                                  |  = 1000 mod 7 = 6 ✓                 |
+----------------------------------+-------------------------------------+
```

**Why this matters:** We can keep numbers small throughout a computation. Instead of computing 3^1000000 (a number with ~477,000 digits) and then taking mod, we reduce at each step and never exceed m^2.

### Modular Exponentiation

Computing a^b mod m efficiently requires [[adsa-fast-exponentiation]] (binary exponentiation). The key identity:

```
a^b mod m, step by step:

a^13 mod 10  (13 = 1101 in binary)

Using repeated squaring:
base = 3, exp = 13, mod = 10

exp=13 (odd):  result = result * 3 = 3;   base = 3^2 = 9;   exp = 6
exp=6  (even): (skip);                    base = 9^2 = 81 mod 10 = 1; exp = 3
exp=3  (odd):  result = 3 * 1 = 3;        base = 1^2 = 1;   exp = 1
exp=1  (odd):  result = 3 * 1 = 3;        base = 1^2 = 1;   exp = 0

Answer: 3^13 mod 10 = 3  ✓
```

```python
def mod_pow(base: int, exp: int, mod: int) -> int:
    """Compute (base^exp) % mod in O(log exp) time."""
    result = 1
    base %= mod
    while exp > 0:
        if exp & 1:
            result = (result * base) % mod
        base = (base * base) % mod
        exp >>= 1
    return result

# mod_pow(3, 13, 10) → 3
# mod_pow(65, 17, 3233) → 2790  (RSA-like example, n=61×53)
```

### Fermat's Little Theorem

If p is prime and a is NOT a multiple of p:

```
a^(p-1) ≡ 1  (mod p)
```

**Worked example:**
```
p = 7, a = 3

3^6 mod 7 = 729 mod 7 = ?
729 = 104*7 + 1
3^6 mod 7 = 1  ✓
```

**Why it matters:**
1. Gives a fast primality test: if a^(p-1) ≢ 1 (mod p), then p is definitely NOT prime.
2. Provides modular inverses: a^(p-2) ≡ a^-1 (mod p). So for prime moduli, we can divide using fast exponentiation instead of extended Euclidean.

```python
def fermat_inverse(a: int, p: int) -> int:
    """Modular inverse via Fermat's little theorem. p must be prime."""
    return mod_pow(a, p - 2, p)

# fermat_inverse(3, 7) = 3^5 mod 7 = 243 mod 7 = 5
# Verify: 3 * 5 = 15 ≡ 1 (mod 7) ✓
```

**Caveat:** The Fermat test alone is unsafe for primality — **Carmichael numbers** satisfy a^(n-1) ≡ 1 (mod n) for all coprime a even though n is composite (e.g., 561 = 3×11×17). Always use Miller-Rabin instead.

### Euler's Totient Function

Euler extended Fermat's theorem. The totient φ(n) counts integers in [1, n] that are coprime to n.

```
φ(1)  = 1
φ(p)  = p-1     (for prime p — all integers 1..p-1 are coprime to p)
φ(p*q) = (p-1)(q-1)   for distinct primes p, q

Example: φ(12) = ?
Integers 1-12: 1,2,3,4,5,6,7,8,9,10,11,12
Coprime to 12: 1,5,7,11  → φ(12) = 4
```

**Euler's theorem:** If gcd(a, n) = 1:
```
a^φ(n) ≡ 1  (mod n)
```

This generalizes Fermat (φ(p) = p-1 for prime p).

---

## Part 4 — RSA: Putting It All Together

RSA is a textbook application of everything above. Understanding it shows why all three pillars are necessary.

### Key Generation (Small Example)

```
Step 1: Choose two primes
  p = 61,  q = 53

Step 2: Compute modulus
  n = p × q = 61 × 53 = 3233

Step 3: Compute totient
  φ(n) = (p-1)(q-1) = 60 × 52 = 3120

Step 4: Choose public exponent
  e = 17   (must satisfy: gcd(e, φ(n)) = 1)
  gcd(17, 3120) = 1  ✓

Step 5: Compute private exponent
  d = e^-1 mod φ(n) = 17^-1 mod 3120
  Using extended Euclidean: d = 2753
  Verify: 17 × 2753 = 46801 = 15×3120 + 1 ≡ 1 (mod 3120) ✓

Public key:  (e=17, n=3233)
Private key: (d=2753, n=3233)
```

### Encrypt and Decrypt

```
Message: m = 65  ("Hi" encoded)

Encrypt:  c = m^e mod n = 65^17 mod 3233 = 2790
Decrypt:  m = c^d mod n = 2790^2753 mod 3233 = 65  ✓
```

Both operations use [[adsa-fast-exponentiation]] — mod_pow(). The security relies on the hardness of factoring n = 3233 back into 61 × 53. At RSA-2048 scale, factoring would take billions of years.

### RSA Key Generation: Real Scale

```
Real RSA-2048:
  p, q: ~1024-bit primes (~308 decimal digits each)
  n:    2048-bit number (~617 decimal digits)
  
  Miller-Rabin with 64 rounds tests primality in <0.5ms per candidate
  Expected candidates tested: O(ln n) ≈ O(1400) per prime
  Total keygen time: typically <1 second on modern CPUs
  
  Factoring n at this scale: billions of years on classical hardware
```

---

## Part 5 — Real-World Applications

### Hash Tables — Bitwise Mod Trick

When a hash table has a **power-of-two** size m = 2^k, the modulo operation can be replaced by bitwise AND:

```
x mod m  =  x & (m - 1)    [only works when m is a power of 2]

Example: m = 16 = 2^4
x = 37 = 100101 in binary
m-1 = 15 = 001111 in binary

37 & 15 = 100101 & 001111 = 000101 = 5

Verify: 37 mod 16 = 37 - 2*16 = 37 - 32 = 5  ✓
```

The AND operation keeps only the lower k bits, which is exactly the remainder. This is ~2x faster than division-based modulo in tight loops (source: 1 - Primes-GCD-and-Modular-Arithmetic.pdf).

### Consistent Hashing — CDN Caching

Content Delivery Networks map keys to servers on a virtual circle of 2^32 positions. Servers are placed at positions using hash(server_ip) mod 2^32. When a server is added or removed, only K/N keys need remapping (vs. nearly all with naive modulo hashing).

### ISBN-13 and Luhn Checksum

Credit card numbers use the Luhn algorithm: double every second digit, sum all digits, the total must be ≡ 0 (mod 10). Mod 10 catches all single-digit errors. ISBN-10 uses mod 11 to catch transpositions too.

```
Luhn check for 4532015112830366:

Digits doubled (every 2nd from right): 8,5,6,2,0,1,1,1,2,8,6,0,3,6,6
(doubled and digit-summed if > 9)

Sum = 40 → 40 mod 10 = 0  ✓ Valid card number
```

---

## Performance Reference

```
+------------------------------------+---------------------+---------------+
| Algorithm                          | Time Complexity     | Space         |
+------------------------------------+---------------------+---------------+
| Sieve of Eratosthenes (up to n)    | O(n log log n)      | O(n)          |
| Trial division primality test      | O(sqrt(n))          | O(1)          |
| Miller-Rabin (k rounds, n-bit)     | O(k log^3 n)        | O(log n)      |
| Euclidean GCD                      | O(log min(a,b))     | O(1)          |
| Extended Euclidean                 | O(log min(a,b))     | O(1)          |
| Modular exponentiation             | O(log b) mult.      | O(1)          |
+------------------------------------+---------------------+---------------+

Speed benchmarks (from source):
  Miller-Rabin on 2048-bit:  < 0.5ms per candidate
  Sieve of 10^8:             ~0.5s (optimized C), ~20s (pure Python)
  TLS connections/s:         ~100k achievable with O(log n) per connection
```

---

## Common Pitfalls

```
+-----------------------------+-------------------------------------------+
| Mistake                     | Fix                                       |
+-----------------------------+-------------------------------------------+
| 1 is prime                  | 1 is NOT prime — start sieve from 2       |
| % on negatives in C/Java    | Use ((x % m) + m) % m for true modulus   |
| Trial division to n         | Loop only while i*i <= n, not i <= n     |
| Using Fermat test alone     | Carmichael numbers defeat it — use MR    |
| pow(base,exp) then % mod    | Causes huge intermediates; use           |
|                             | pow(base, exp, mod) in Python            |
| Too few Miller-Rabin rounds | 1 round → >25% error; use >=40 for      |
|                             | crypto                                   |
| Shared prime in RSA keys    | 2012 incident: 0.2% of 7.1M public keys |
|                             | shared a prime — GCD attack breaks both  |
+-----------------------------+-------------------------------------------+
```

**Production incident (2012):** Researchers scanned 7.1 million RSA public keys and found 0.2% shared a prime factor. Taking GCD of any two such keys immediately revealed the private key. Always generate fresh, independent primes per deployment (source: 1 - Primes-GCD-and-Modular-Arithmetic.pdf).

---

## Summary Table

```
+--------------------+--------------------+-------------------------------+
| Concept            | Core Formula       | Use Case                      |
+--------------------+--------------------+-------------------------------+
| Primality (fast)   | Miller-Rabin MR    | RSA key generation            |
| All primes to n    | Sieve: O(n log n)  | Precomputation, hash tables   |
| GCD                | gcd(a,b)=gcd(b,a%b)| Key derivation, simplify fracs|
| Modular inverse    | Extended Euclidean | RSA private key, division mod |
| Modular inverse    | a^(p-2) mod p      | (Only when p is prime)        |
| Fermat's theorem   | a^(p-1) ≡ 1 mod p  | Primality hint, base for MR   |
| Euler's theorem    | a^φ(n) ≡ 1 mod n   | RSA correctness proof         |
| RSA keypair        | ed ≡ 1 mod φ(n)    | Public key cryptography       |
+--------------------+--------------------+-------------------------------+
```

#adsa
