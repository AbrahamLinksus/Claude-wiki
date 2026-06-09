# Combinatorics: Factorials, nCr & Pascal's Triangle

**Summary**: Combinatorics answers the question "how many ways?" — counting subsets, arrangements, and selections using factorials, binomial coefficients, and Pascal's triangle, with applications from lottery odds to dynamic programming.

**Pre-Req**: Basic multiplication and division. No prior math background needed.

**Sources**: `raw/2 - Combinatorics-nCr-Factorials-and-Pascals-Triangle.pdf`

**Last updated**: 2026-06-09

---

## Why Counting Matters

You write code to assign 5 tasks to 3 servers. How many assignments exist? You want to select 6 lottery numbers from 49. What are your odds? You're counting grid paths from one corner of a 20×20 board to the other. How many routes exist?

All of these reduce to the same mathematical machinery: **counting unordered selections without replacement**. Getting this wrong means wrong probabilities, wrong algorithm complexity bounds, and subtle bugs in DP solutions.

```
Examples of "how many ways?":

Lottery:      Choose 6 from 49 numbers         → C(49, 6) = 13,983,816
Committees:   Choose 3 people from 10          → C(10, 3) = 120
Grid paths:   Move 20 right + 20 down on grid  → C(40, 20) = 137,846,528,820
A/B test:     12 successes in 20 trials        → C(20, 12) = 125,970 ways
```

---

## Part 1 — Factorials: The Foundation

### What is a Factorial?

n! (read "n factorial") means: multiply all integers from 1 up to n.

```
Formula:  n! = n × (n-1) × (n-2) × ... × 2 × 1

0! = 1                    (by definition — the empty product)
1! = 1
2! = 2 × 1 = 2
3! = 3 × 2 × 1 = 6
4! = 4 × 3 × 2 × 1 = 24
5! = 5 × 4 × 3 × 2 × 1 = 120
10! = 3,628,800
20! = 2,432,902,008,176,640,000  (fits in a 64-bit integer — barely!)
21! = 51,090,942,171,709,440,000 (OVERFLOWS 64-bit signed integer!)
```

**Critical overflow boundary:** 20! fits in a 64-bit integer (max ~9.2 × 10^18). 21! exceeds it. Always use big integers (Python `int`, Java `BigInteger`) for n >= 21. (source: 2 - Combinatorics-nCr-Factorials-and-Pascals-Triangle.pdf)

### Why Does 0! = 1?

Think of it as "the number of ways to arrange 0 objects": there is exactly one way — do nothing. The empty arrangement.

Also required for the nCr formula to work: C(n, 0) = n! / (0! × n!) = 1. There is exactly 1 way to choose nothing from n items.

### Factorial Growth Rate

```
n      n!              Rough scale
--     --              -----------
0      1
5      120
10     3,628,800       ~3.6 million
15     1.3 × 10^12     ~1.3 trillion
20     2.4 × 10^18     ~2.4 quintillion (near 64-bit limit)
25     1.6 × 10^25     Big integer required
50     3.0 × 10^64     Astronomically large
100    9.3 × 10^157    Incomprehensibly large
```

Factorial grows faster than any exponential. n! > 2^n for all n >= 3.

```python
import math

def factorial_iterative(n: int) -> int:
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result

# Python's arbitrary precision int handles any size:
print(factorial_iterative(100))  # exact 158-digit number

# Built-in (fastest, implemented in C):
print(math.factorial(100))
```

### Factorials Count Permutations

n! is the number of ways to arrange n distinct objects in a sequence.

```
Arrange 3 letters A, B, C in all possible orders:

Position 1: 3 choices (A, B, or C)
Position 2: 2 choices (whatever's left)
Position 3: 1 choice  (the last one)

Total = 3 × 2 × 1 = 3! = 6

The 6 arrangements: ABC, ACB, BAC, BCA, CAB, CBA
```

---

## Part 2 — Combinations (nCr): Choosing Without Order

### The Problem: Order Doesn't Matter

If you choose 2 people from {Alice, Bob, Carol} to form a team:
- {Alice, Bob} is the same team as {Bob, Alice}
- We're choosing a **subset**, not an ordered sequence

The number of ways to choose r items from n items (where order doesn't matter) is C(n, r), also written nCr or "n choose r".

### The Formula

```
        n!
C(n,r) = --------
         r! × (n-r)!
```

**Formula explained:**

1. Start with n! — all possible orderings of n items
2. Divide by r! — we don't care about the order of the chosen r items
3. Divide by (n-r)! — we don't care about the order of the unchosen items

```
Worked example: C(5, 2) — choose 2 from 5

         5!          120
C(5,2) = ------  =  ----- = 10
         2! × 3!    2 × 6

The 10 pairs from {A, B, C, D, E}:
{A,B}, {A,C}, {A,D}, {A,E}
{B,C}, {B,D}, {B,E}
{C,D}, {C,E}
{D,E}
Count them: 4+3+2+1 = 10 ✓
```

### Key Properties

**Symmetry:** C(n, r) = C(n, n-r)

Choosing 3 to include is the same as choosing n-3 to exclude.

```
C(10, 3) = C(10, 7) = 120
C(49, 6) = C(49, 43)
```

**This halves computation:** always use min(r, n-r) to pick the smaller loop count.

**Pascal's Rule (recurrence):** C(n, r) = C(n-1, r-1) + C(n-1, r)

Intuition: pick a specific element x. Either x is in our chosen subset (choose the remaining r-1 from n-1), or x is not (choose all r from n-1).

**Boundary cases:**
```
C(n, 0) = 1     (one way to choose nothing)
C(n, n) = 1     (one way to choose everything)
C(n, 1) = n     (n ways to choose exactly one item)
C(n, r) = 0     if r < 0 or r > n
```

### Multiplicative Formula — Faster for Single Queries

The division formula risks overflow with large factorials. The multiplicative form avoids it:

```
         n × (n-1) × (n-2) × ... × (n-r+1)
C(n,r) = -----------------------------------
                  r × (r-1) × ... × 1
```

Key insight: the product of any r consecutive integers is always divisible by r!. So we can interleave multiplication and division, keeping intermediate values small.

**Step-by-step: C(7, 3)**

```
C(7, 3):  numerator = 7 × 6 × 5,  denominator = 3 × 2 × 1

Step 1: result = 7/1 = 7       (integer, 7 is divisible by 1)
Step 2: result = 7×6/2 = 21    (integer, 7×6=42 divisible by 2)
Step 3: result = 21×5/3 = 35   (integer, 21×5=105 divisible by 3)

C(7, 3) = 35
```

The intermediate result after step i always equals C(n-r+i, i), which is always an integer. No fractions, no overflow for reasonable n.

```python
def ncr_multiplicative(n: int, r: int) -> int:
    """Compute C(n,r) without computing full factorials."""
    if r < 0 or r > n:
        return 0
    r = min(r, n - r)     # use symmetry: C(n,r) = C(n,n-r)
    result = 1
    for i in range(r):
        result = result * (n - i) // (i + 1)
    return result

# ncr_multiplicative(49, 6) → 13,983,816  (lottery odds denominator)
# ncr_multiplicative(40, 20) → 137,846,528,820  (grid paths on 20x20 board)
```

**Computation method comparison:**

```
+---------------------------+----------+----------+-------------------+
| Method                    | Time     | Memory   | Best Use Case     |
+---------------------------+----------+----------+-------------------+
| Direct formula (n!/r!/(n-r)!) | O(n)  | O(n)    | Small n only      |
| Multiplicative            | O(r)     | O(1)     | Single query      |
| Pascal DP (precompute)    | O(n^2)   | O(n^2)   | Many queries, n small |
| Modular (mod prime)       | O(n)     | O(n)     | Large n, mod result   |
+---------------------------+----------+----------+-------------------+
```

### Overflow Safety Limits

```
C(66, 33) ≈ 7.22 × 10^18  →  last value fitting in signed 64-bit int
C(67, 33) ≈ 1.42 × 10^19  →  OVERFLOW

For safe exact computation on 64-bit: n <= 66
For n > 66: use Python big integers or compute mod prime
```

---

## Part 3 — Pascal's Triangle

### Structure

Pascal's triangle is an infinite triangle where each number equals the sum of the two directly above it.

```
Row 0:             1
Row 1:           1   1
Row 2:         1   2   1
Row 3:       1   3   3   1
Row 4:     1   4   6   4   1
Row 5:   1   5  10  10   5   1
Row 6: 1   6  15  20  15   6   1

Row n contains: C(n,0), C(n,1), C(n,2), ..., C(n,n)

Row 5: C(5,0)=1, C(5,1)=5, C(5,2)=10, C(5,3)=10, C(5,4)=5, C(5,5)=1
```

**Pascal's rule in the triangle:** every interior entry = sum of two entries above it.
```
      4   6
        10     ← because 4 + 6 = 10 = C(5,3)
```

This is exactly C(n, r) = C(n-1, r-1) + C(n-1, r) from Part 2.

### Properties of Pascal's Triangle

**Row sum = 2^n:**
```
Sum of row n = C(n,0) + C(n,1) + ... + C(n,n) = 2^n

Row 0: 1         = 2^0 = 1
Row 1: 1+1       = 2^1 = 2
Row 2: 1+2+1     = 2^2 = 4
Row 3: 1+3+3+1   = 2^3 = 8
Row 5: 1+5+10+10+5+1 = 2^5 = 32

Proof: set x=y=1 in the binomial theorem (x+y)^n = Σ C(n,k) x^(n-k) y^k
```

**Binomial theorem connection:**
```
(x + y)^n = C(n,0)x^n + C(n,1)x^(n-1)y + C(n,2)x^(n-2)y^2 + ... + C(n,n)y^n

Example: (x + y)^3 = x^3 + 3x^2y + 3xy^2 + y^3
Coefficients: 1, 3, 3, 1  ← Row 3 of Pascal's triangle!
```

**Hockey stick identity:**
```
C(r,r) + C(r+1,r) + C(r+2,r) + ... + C(n,r) = C(n+1, r+1)

Example (r=2):
C(2,2) + C(3,2) + C(4,2) + C(5,2) = C(6,3)
  1    +   3    +   6    +   10    = 20  ✓
```

### Building Pascal's Triangle (DP)

```python
def build_pascal(max_n: int) -> list[list[int]]:
    """Build Pascal's triangle up to row max_n. O(n^2) time and space."""
    C = [[0] * (max_n + 1) for _ in range(max_n + 1)]
    for n in range(max_n + 1):
        C[n][0] = 1                  # C(n,0) = 1
        for r in range(1, n + 1):
            C[n][r] = C[n-1][r-1] + C[n-1][r]
    return C

# C[5][2] = 10, C[7][3] = 35, etc.
```

**Memory-efficient (rolling array):**

```python
def build_pascal_1d(max_n: int) -> list[list[int]]:
    """O(n) space using a single rolling row."""
    rows = []
    row = [1]
    rows.append(row[:])
    for n in range(1, max_n + 1):
        new_row = [1]
        for r in range(1, n):
            new_row.append(row[r-1] + row[r])
        new_row.append(1)
        row = new_row
        rows.append(row[:])
    return rows
```

**Important: update in reverse order when using a 1D array** to avoid using updated values in the same row.

### Pascal's Triangle — Complexity

```
Building all C(n,r) for n up to max_n:
  Time:  O(max_n^2)
  Space: O(max_n^2) for full table, O(max_n) for single row

Lookup after precomputation: O(1)

When to use Pascal DP vs. multiplicative:
  Many queries for the same n     → precompute Pascal O(1) lookup
  Few queries, large n            → multiplicative O(r)
  Need modular result, large n    → precompute factorials mod prime
```

---

## Part 4 — nCr Modulo a Prime (Competitive Programming)

When n is very large (e.g., 10^6) and you need C(n, r) mod p where p is prime, compute:

```
C(n, r) mod p = (n! mod p) × inverse(r! mod p) × inverse((n-r)! mod p)  mod p
```

Where the modular inverse uses Fermat's little theorem (since p is prime):
```
inverse(x) = x^(p-2) mod p
```

**Precomputation approach:**

```python
def precompute_factorials(max_n: int, p: int):
    """Precompute factorials and inverse factorials mod prime p."""
    fact = [1] * (max_n + 1)
    for i in range(1, max_n + 1):
        fact[i] = fact[i-1] * i % p

    inv_fact = [1] * (max_n + 1)
    inv_fact[max_n] = pow(fact[max_n], p - 2, p)   # Fermat's little theorem
    for i in range(max_n - 1, -1, -1):
        inv_fact[i] = inv_fact[i+1] * (i+1) % p

    return fact, inv_fact

def ncr_mod(n: int, r: int, p: int, fact, inv_fact) -> int:
    if r < 0 or r > n:
        return 0
    return fact[n] * inv_fact[r] % p * inv_fact[n-r] % p

# After O(n) precomputation, each query is O(1)
```

### Lucas' Theorem — When n > p

If p is prime and n is very large (n > p), Lucas' theorem decomposes the computation:

```
C(n, r) mod p = product of C(n_i, r_i) mod p

where n_i and r_i are the digits of n and r in base p.

Example: C(10, 4) mod 3
  10 in base 3 = 101  (1×9 + 0×3 + 1×1)
   4 in base 3 = 011  (0×9 + 1×3 + 1×1)

C(10,4) mod 3 = C(1,0) × C(0,1) × C(1,1) mod 3
              = 1 × 0 × 1 = 0

Verify: C(10,4) = 210 = 70×3 ≡ 0 (mod 3) ✓
```

**Important:** Lucas' theorem is only valid when p is prime. Applying it with composite modulus gives wrong results. (source: 2 - Combinatorics-nCr-Factorials-and-Pascals-Triangle.pdf)

---

## Part 5 — Applications

### Lottery: C(49, 6)

The UK lottery: choose 6 numbers from 1-49 (order doesn't matter, no repetition).

```
C(49, 6) = (49 × 48 × 47 × 46 × 45 × 44) / (6 × 5 × 4 × 3 × 2 × 1)
         = 10,068,347,520 / 720
         = 13,983,816

Your chance of winning: 1 in 13,983,816 ≈ 0.0000071%
```

### Grid Paths: C(40, 20)

On a 20×20 grid, moving only right or down, how many paths from top-left to bottom-right?

```
Every path requires exactly 20 right moves and 20 down moves.
Total 40 moves, choose which 20 are "right" (rest are "down"):

C(40, 20) = 137,846,528,820  (~138 billion paths)
```

### A/B Testing

A new feature shows 12 successes in 20 trials. How likely is this if the true success rate is 50%?

```
P(exactly 12 successes) = C(20, 12) × 0.5^12 × 0.5^8
                        = 125,970 × 0.5^20
                        ≈ 0.12 (12%)

C(20, 12) = C(20, 8) = 125,970   (symmetry: choosing 12 = choosing 8 to exclude)
```

### nCr vs nPr: Permutations vs Combinations

```
+----------------+-----------------------+------------------+
| Feature        | nCr (Combinations)    | nPr (Permutations)|
+----------------+-----------------------+------------------+
| Formula        | n! / (r!(n-r)!)       | n! / (n-r)!      |
| Order matters? | No                    | Yes              |
| Growth rate    | Slower                | Faster (larger)  |
| Use case       | Lottery, committees,  | Rankings,        |
|                | subsets               | passwords,       |
|                |                       | scheduling       |
| Relationship   | nCr = nPr / r!        | nPr = nCr × r!   |
+----------------+-----------------------+------------------+

Example: Choose 2 from {A, B, C}
  nCr: {A,B}, {A,C}, {B,C}                 → 3 combinations
  nPr: AB, BA, AC, CA, BC, CB             → 6 permutations = 3 × 2!
```

---

## Part 6 — Common Pitfalls

```
+-----------------------------------+-------------------------------------+
| Mistake                           | Fix                                 |
+-----------------------------------+-------------------------------------+
| C(n,0) != 1 or C(n,1) != n       | Memoize base cases explicitly       |
| Assuming 0! = 0                   | 0! = 1 by definition                |
| Overflow with n=30, r=15          | n=30 already exceeds 2^31;          |
|                                   | use Python int or check first       |
| Integer division without ensuring | Multiplicative form: interleave     |
| exact divisibility               | multiply then divide; always exact  |
| Updating Pascal row in forward    | Update row in reverse to avoid      |
| order (1D array)                  | using partially-updated values      |
| Applying Lucas theorem for        | Lucas is only valid for prime       |
| non-prime modulus                 | modulus                             |
| Forgetting C(n,r) = C(n,n-r)     | Always set r = min(r, n-r) first    |
+-----------------------------------+-------------------------------------+
```

**Production warning:** An API computing C(10^9, 5×10^8) could exhaust server memory — the result has ~300,000 decimal digits. Always enforce upper bounds on n (e.g., n <= 10^6) and return modular results for large inputs. (source: 2 - Combinatorics-nCr-Factorials-and-Pascals-Triangle.pdf)

---

## Summary Table

```
+-------------------------------+------------------------------------------+
| Concept                       | Formula / Rule                           |
+-------------------------------+------------------------------------------+
| Factorial                     | n! = n × (n-1)!, 0! = 1                 |
| Binomial coefficient          | C(n,r) = n! / (r!(n-r)!)               |
| Symmetry                      | C(n,r) = C(n, n-r)                     |
| Pascal's rule                 | C(n,r) = C(n-1,r-1) + C(n-1,r)        |
| Binomial theorem              | (x+y)^n = Σ C(n,r) x^(n-r) y^r        |
| Multiplicative form           | C(n,r) = Π_{i=1}^{r} (n-r+i)/i        |
| Lucas theorem                 | C(n,r) mod p = Π C(n_i, r_i) mod p   |
| Row sum                       | Σ_{r=0}^{n} C(n,r) = 2^n              |
| 64-bit overflow               | C(66,33) is last safe value            |
+-------------------------------+------------------------------------------+

Key takeaways:
1. Factorials grow explosively: 20! fits in 64-bit; 21! overflows
2. nCr counts unordered subsets — fundamental in probability and DP
3. Pascal's triangle uses only addition — overflow-resistant and fast
4. Multiplicative formula is optimal for single large-n queries
5. Always set r = min(r, n-r) to halve computation via symmetry
```

#adsa
