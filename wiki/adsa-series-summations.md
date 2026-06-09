# Series and Summations

**Summary**: Series are sums of sequences; mastering closed-form formulas and summation techniques lets you derive Big-O bounds rigorously, replace O(n) loops with O(1) computations, and model financial, physical, and algorithmic systems precisely.

**Pre-Req**: Basic algebra (variables, equations). [[adsa-overview]] — Big-O notation helps for the algorithm analysis section.

**Sources**: `raw/4 - Series and Summations.pdf`

**Last updated**: 2026-06-09

---

## Why Summations Matter

A double nested loop where the inner loop runs from 1 to i yields iterations:
`1 + 2 + 3 + ... + n = n(n+1)/2`

Without series theory, you cannot derive O(n^2) rigorously — you can only guess it by running tests. In 2023, over 40% of production performance regressions in large-scale systems traced to unanalysed nested loops (source: 4 - Series and Summations.pdf).

Series also power:
- Loan payments (geometric series of discounted payments)
- Sliding window averages (incremental sum updates)
- Physics simulations (sum of odd numbers = n^2)
- Signal processing (Fourier series)

---

## Part 1 — Vocabulary: Sequences vs. Series

### Sequence

A **sequence** is an ordered list of numbers: a_1, a_2, a_3, ..., a_n

Each number is a **term**. The rule generating terms is the **general term** a_k.

```
Arithmetic sequence (constant difference d=3):
  2, 5, 8, 11, 14, ...
  a_k = 2 + (k-1) × 3

Geometric sequence (constant ratio r=2):
  3, 6, 12, 24, 48, ...
  a_k = 3 × 2^(k-1)

Square sequence:
  1, 4, 9, 16, 25, ...
  a_k = k^2
```

### Series

A **series** is the sum of terms of a sequence.

```
Finite series:   S_n = a_1 + a_2 + ... + a_n    (definite sum)
Infinite series: S   = a_1 + a_2 + a_3 + ...    (may or may not converge)
```

### Sigma (Summation) Notation

Sigma notation compresses long sums into one line:

```
   n
   Σ  a_k   means:  a_m + a_{m+1} + ... + a_n
  k=m

Examples:

  5
  Σ k   = 1 + 2 + 3 + 4 + 5 = 15
 k=1

  5
  Σ k^2 = 1 + 4 + 9 + 16 + 25 = 55
 k=1

  4
  Σ 2^k = 2 + 4 + 8 + 16 = 30
 k=1
```

The variable k is a **dummy variable** — any letter works. The index goes from the bottom value to the top value, one step at a time.

---

## Part 2 — Arithmetic Series

### Definition

An arithmetic series sums a sequence with **constant difference** d between consecutive terms.

```
General term:   a_k = a_1 + (k-1) × d

Examples:
  a_1=2, d=3:   2, 5, 8, 11, 14    (add 3 each time)
  a_1=10, d=-2: 10, 8, 6, 4, 2     (subtract 2 each time)
  a_1=1, d=1:   1, 2, 3, 4, 5, ... (the counting numbers)
```

### Closed-Form Sum

```
          n
S_n  =  -----  × (2a_1 + (n-1)d)
          2

Equivalently:

S_n  =  n/2 × (first_term + last_term)

Memory aid: "First plus last, half the count."
```

**Why it works — Gauss's pairing trick:**

```
S = 1 + 2 + 3 + ... + 98 + 99 + 100
S = 100 + 99 + 98 + ... + 3 + 2 + 1
                              (write it backwards)

Add them:
2S = (1+100) + (2+99) + (3+98) + ... + (100+1)
   = 101 + 101 + 101 + ... (100 times)
   = 100 × 101
   = 10100

S  = 10100 / 2 = 5050

This is the classic "Gauss at age 9" result.
```

### Worked Examples

```
Example 1: Sum 1 + 2 + 3 + ... + n

S_n = n(n+1)/2   [special case: a_1=1, d=1]

n=100: S = 100 × 101 / 2 = 5050
n=1000: S = 1000 × 1001 / 2 = 500,500

Example 2: Sum 3 + 7 + 11 + 15 + ... to 20 terms

a_1 = 3,  d = 4,  n = 20
Last term: a_20 = 3 + 19×4 = 3 + 76 = 79

S_20 = 20/2 × (3 + 79) = 10 × 82 = 820

Example 3: Loop analysis — total work in a triangular loop

for i in range(1, n+1):
    for j in range(1, i+1):   ← inner runs i times
        do_work()

Total iterations = 1 + 2 + 3 + ... + n = n(n+1)/2 = Θ(n^2)
```

---

## Part 3 — Geometric Series

### Definition

A geometric series sums a sequence where each term is multiplied by a constant **ratio** r.

```
General term:   a_k = a_1 × r^(k-1)

Examples:
  a_1=3, r=2:    3, 6, 12, 24, 48     (double each time)
  a_1=1, r=0.5:  1, 0.5, 0.25, ...   (halve each time)
  a_1=1, r=1/3:  1, 1/3, 1/9, 1/27, ...
```

### Closed-Form Sum (Finite)

```
          a_1 × (1 - r^n)
S_n  =  -------------------    (r ≠ 1)
                1 - r

Special case r = 1:  S_n = a_1 × n   (all terms equal)

Memory aid: "First times one minus r-to-n, over one minus r."
```

**Derivation (why it works):**

```
S   = a + ar + ar^2 + ... + ar^(n-1)
rS  = ar + ar^2 + ar^3 + ... + ar^n

Subtract:   S - rS = a - ar^n
            S(1-r) = a(1 - r^n)
            S = a(1 - r^n) / (1 - r)       ✓
```

### Worked Examples

```
Example 1: Sum 3 + 6 + 12 + 24 + ... + 3072 (10 terms, r=2)

a_1 = 3,  r = 2,  n = 10
S_10 = 3 × (1 - 2^10) / (1 - 2)
     = 3 × (1 - 1024) / (-1)
     = 3 × (-1023) / (-1)
     = 3 × 1023
     = 3069

Verify: 3+6+12+24+48+96+192+384+768+1536 = 3069  ✓

Example 2: Doubling array reallocation cost

Dynamic array doubles capacity: sizes 1, 2, 4, 8, ..., n
Total copy work = 1 + 2 + 4 + ... + n

With a = 1, r = 2:
S = 1 × (2^(log2(n)+1) - 1) / (2-1) = 2n - 1 ≈ 2n = O(n)

This is why amortized O(1) append works — see [[adsa-arrays]].

Example 3: 30-year mortgage (financial)

Monthly payment P on loan L at monthly rate r for n months:
  P = L × r / (1 - (1+r)^(-n))

This is derived from summing a geometric series of discounted payments.
n = 360 terms (30 years), yet the formula gives the answer in O(1).
```

### Infinite Geometric Series

When |r| < 1, the series converges (approaches a finite value):

```
         a
S∞ = -------     (only when |r| < 1)
        1 - r

Example: 1 + 1/2 + 1/4 + 1/8 + ...

a = 1,  r = 1/2
S∞ = 1 / (1 - 1/2) = 1 / (1/2) = 2

Intuition: each term covers half the remaining gap to 2.

Example: 0.999... = 0.9 + 0.09 + 0.009 + ...
a = 0.9,  r = 0.1
S∞ = 0.9 / (1 - 0.1) = 0.9/0.9 = 1.0   (so 0.999... = 1 exactly!)
```

When |r| >= 1, the series **diverges** (sum grows without bound). No closed form exists.

---

## Part 4 — Closed-Form Reference Table

These are the most important summation identities in algorithm analysis.

```
+----------------------------------+------------------------+----------+
| Series                           | Closed Form            | Growth   |
+----------------------------------+------------------------+----------+
| Σ c  (k=1 to n)                  | c × n                  | O(n)     |
|   constant                       |                        |          |
| Σ k  (k=1 to n)                  | n(n+1)/2               | O(n^2)   |
|   integers                       |                        |          |
| Σ k^2 (k=1 to n)                 | n(n+1)(2n+1)/6         | O(n^3)   |
|   squares                        |                        |          |
| Σ k^3 (k=1 to n)                 | [n(n+1)/2]^2           | O(n^4)   |
|   cubes                          |                        |          |
| Σ 2^k (k=0 to n-1)               | 2^n - 1                | O(2^n)   |
|   powers of 2                    |                        |          |
| Σ ar^k (k=0 to n-1)              | a(1-r^n)/(1-r)         | varies   |
|   geometric                      |                        |          |
| Σ 1/k (k=1 to n)                 | H_n ≈ ln(n) + 0.5772  | O(log n) |
|   harmonic                       | (diverges to infinity) |          |
| Σ 1/(k(k+1)) (k=1 to n)          | 1 - 1/(n+1)            | O(1)     |
|   telescoping                    |                        |          |
+----------------------------------+------------------------+----------+
```

### Deriving Sum of Squares

This derivation shows the power of telescoping — a technique that turns complex sums elegant.

**Goal:** find closed form for Σ k^2 from k=1 to n.

```
Step 1: Start with the cubic identity:
  (k+1)^3 - k^3 = 3k^2 + 3k + 1

Step 2: Sum both sides from k=1 to n:
  Σ[(k+1)^3 - k^3] = 3 Σk^2 + 3 Σk + n

Step 3: Left side telescopes:
  (2^3 - 1^3) + (3^3 - 2^3) + ... + ((n+1)^3 - n^3)
  = (n+1)^3 - 1      ← all intermediate terms cancel

Step 4: Substitute known sum Σk = n(n+1)/2:
  (n+1)^3 - 1 = 3 Σk^2 + 3 × n(n+1)/2 + n

Step 5: Solve for Σk^2:
  3 Σk^2 = (n+1)^3 - 1 - 3n(n+1)/2 - n
  3 Σk^2 = n(n+1)(2n+1) / ... (algebra)
  Σk^2 = n(n+1)(2n+1) / 6
```

**Quick check:** n=5: 1+4+9+16+25 = 55. Formula: 5×6×11/6 = 330/6 = 55 ✓

---

## Part 5 — Loop Complexity Analysis

### Single Loop

```python
for i in range(1, n+1):
    work()                  # O(1) each
```

Total iterations = n → **O(n)**

### Nested Loop (Independent Bounds)

```python
for i in range(n):
    for j in range(n):
        work()
```

Inner runs n times per outer iteration; outer runs n times.
Total = n × n = n^2 → **O(n^2)**

```python
for i in range(n):
    for j in range(m):
        work()
```

Total = n × m → **O(nm)**

### Nested Loop (Dependent Bounds — Triangular)

```python
for i in range(1, n+1):
    for j in range(1, i+1):  ← inner depends on outer
        work()
```

```
When i=1: 1 iteration
When i=2: 2 iterations
When i=3: 3 iterations
...
When i=n: n iterations

Total = 1 + 2 + 3 + ... + n = n(n+1)/2 ≈ n^2/2 → O(n^2)
```

### Triple Nested (Cubic)

```python
for i in range(n):
    for j in range(i):
        for k in range(j):
            work()
```

```
Total = Σ_{i=0}^{n-1} Σ_{j=0}^{i-1} j
      = Σ_{i=0}^{n-1} i(i-1)/2
      ≈ n^3/6  →  O(n^3)
```

### Logarithmic Loop

```python
i = n
while i > 1:
    i //= 2        # halve each time
    work()
```

After k iterations: i = n / 2^k. Stops when n/2^k <= 1, i.e., k >= log2(n).
Total iterations = **O(log n)**

### Compound Examples

```python
for i in range(n):           # outer: n
    j = 1
    while j < n:             # inner: log(n)
        j *= 2
        work()
```

Total = n × log n → **O(n log n)**

```python
for i in range(n):           # outer: n
    for j in range(i):       # inner: i (triangular)
        k = 1
        while k < j:         # innermost: log(j)
            k *= 2
            work()
```

This requires summing Σ_{i=0}^{n-1} Σ_{j=0}^{i-1} log(j).
Approximation: O(n^2 log n).

---

## Part 6 — Telescoping Series

A telescoping series is one where consecutive terms cancel, leaving only a few boundary terms.

### The Pattern

```
Σ (f(k+1) - f(k)) = f(n+1) - f(1)

The sum "collapses" because:
  (f(2)-f(1)) + (f(3)-f(2)) + (f(4)-f(3)) + ... + (f(n+1)-f(n))
   ^f(1) cancels     ^f(2) cancels
                                              = f(n+1) - f(1)
```

### Classic Example: Sum of 1/(k(k+1))

```
        1         1     1
------ = ------ = --- - ---      (partial fractions)
k(k+1)  k(k+1)    k    k+1

  n
  Σ   1/(k(k+1))
 k=1

= (1/1 - 1/2) + (1/2 - 1/3) + (1/3 - 1/4) + ... + (1/n - 1/(n+1))

= 1/1 - 1/(n+1)       ← everything else cancels!

= 1 - 1/(n+1)

= n/(n+1)
```

**Numerical check (n=4):**
```
1/(1×2) + 1/(2×3) + 1/(3×4) + 1/(4×5)
= 1/2 + 1/6 + 1/12 + 1/20
= 30/60 + 10/60 + 5/60 + 3/60
= 48/60 = 4/5 = n/(n+1) = 4/5  ✓
```

**When the limit is infinite (n→∞):**
```
1 - 1/(n+1) → 1 - 0 = 1

So Σ_{k=1}^{∞} 1/(k(k+1)) = 1  (the series converges to exactly 1)
```

### Another Example: Sum of k/(k+1)!

```
     k          (k+1) - 1      1         1
  ------- = ------------- = ------- - -------
  (k+1)!       (k+1)!       k!        (k+1)!

This is f(k) - f(k+1) where f(k) = 1/k!

  n                n
  Σ  k/(k+1)! =   Σ  (1/k! - 1/(k+1)!)
 k=1             k=1

= (1/1! - 1/2!) + (1/2! - 1/3!) + ... + (1/n! - 1/(n+1)!)

= 1 - 1/(n+1)!
```

**Recognising telescoping:** write out the first few terms and look for cancellation. If f(k) - f(k+1) pattern emerges, it telescopes.

---

## Part 7 — Harmonic and Special Series

### Harmonic Series

```
H_n = 1 + 1/2 + 1/3 + 1/4 + ... + 1/n

Approximation: H_n ≈ ln(n) + γ
where γ ≈ 0.5772 (Euler-Mascheroni constant)

H_1   = 1
H_2   ≈ 1.50
H_5   ≈ 2.28
H_10  ≈ 2.93
H_100 ≈ 5.19
H_1000 ≈ 7.49

This diverges: H_n → ∞ as n → ∞, just very slowly.
```

The harmonic series appears in:
- Analysis of linear search in hash tables
- Expected depth of random BSTs
- Coupon collector problem: expected tries ≈ n × H_n ≈ n ln n

### p-Series

```
Σ 1/k^p   (k=1 to ∞)

Converges if and only if p > 1.

p = 2:  Σ 1/k^2 = 1 + 1/4 + 1/9 + ... = π^2/6 ≈ 1.6449   (Euler's Basel problem)
p = 1:  Σ 1/k   = harmonic series, diverges
p = 0.5: Σ 1/√k, diverges (slower than harmonic)
p = 3:  Σ 1/k^3 = Apéry's constant ≈ 1.202
```

### Power Series

A power series has the form Σ c_k x^k. It converges within a radius of convergence.

```
e^x = 1 + x + x^2/2! + x^3/3! + ...    (converges for all x)
sin(x) = x - x^3/3! + x^5/5! - ...      (converges for all x)
1/(1-x) = 1 + x + x^2 + x^3 + ...       (converges for |x| < 1)
```

Power series are the foundation of Taylor/Maclaurin series used in numerical computing and automatic differentiation in machine learning.

---

## Part 8 — Precision and Floating-Point Summation

Naïve summation in floating-point accumulates O(n × ε) error (where ε is machine epsilon ~10^-16 for float64). For long sums, this matters.

### Kahan Summation Algorithm

Compensated summation reduces error from O(n·ε) to O(ε):

```python
def kahan_sum(numbers):
    """High-precision floating-point summation."""
    total = 0.0
    c = 0.0    # compensation term
    for x in numbers:
        y = x - c             # compensated value
        t = total + y         # sum with compensation
        c = (t - total) - y  # capture lost low bits
        total = t
    return total

# Python's math.fsum uses a similar algorithm (even better precision)
import math
math.fsum([0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1])
# → 1.0 exactly (naive sum gives 0.9999999999999999)
```

### When Precision Matters

```
+----------------------------+------------------------+------------------+
| Situation                  | Method                 | Error            |
+----------------------------+------------------------+------------------+
| Small n, non-alternating   | Naive loop             | O(n × ε)         |
| Precision required         | Kahan / math.fsum      | O(ε)             |
| Parallel computation       | Pairwise summation     | O(ε × log n)     |
| Integer arithmetic         | Closed form            | Exact             |
+----------------------------+------------------------+------------------+
```

---

## Part 9 — Code Examples

```python
# Example 1: Closed Form vs Loop — 10^6x speedup

def sum_loop(n: int) -> int:
    total = 0
    for i in range(1, n + 1):
        total += i
    return total

def sum_closed(n: int) -> int:
    return n * (n + 1) // 2    # O(1), exact

n = 1_000_000
print(sum_closed(n))    # 500000500000 — instantaneous
# sum_loop(n) gives same result but ~10^6x slower

# Example 2: Geometric Series — Compound Interest / Annuity

def annuity_future_value(payment, rate, periods):
    """Future value of regular payments (geometric series)."""
    if rate == 0:
        return payment * periods
    return payment * ((1 + rate) ** periods - 1) / rate

# Monthly savings: £100/month at 5%/year for 12 months
monthly_rate = 0.05 / 12
fv = annuity_future_value(100, monthly_rate, 12)
print(f"{fv:.2f}")   # ~1227.10

# Example 3: Telescoping — exact O(1) computation

def sum_telescoping_example(n: int) -> float:
    """Compute sum_{k=1}^{n} 1/(k(k+1)) iteratively."""
    total = 0.0
    for k in range(1, n + 1):
        total += 1 / (k * (k + 1))
    return total

def closed_telescoping(n: int) -> float:
    return 1 - 1 / (n + 1)    # O(1) exact closed form

n = 100
print(sum_telescoping_example(n))  # 0.9900990099...
print(closed_telescoping(n))       # 0.9900990099...  (identical!)
```

---

## Part 10 — Common Pitfalls

```
+-----------------------------+----------------------------------------------+
| Mistake                     | Fix                                          |
+-----------------------------+----------------------------------------------+
| Off-by-one: Σ from i=1      | Check carefully: Σ_{i=1}^{n} vs             |
| vs i=0 confusion            | Σ_{i=0}^{n-1} — different sums!             |
| Integer division for r in   | If r < 1 and you use integer arithmetic,     |
| geometric series            | r truncates to 0. Use float or fractions.    |
| Assuming closed forms       | Harmonic series has no simple closed form.   |
| always exist               | Use approximation H_n ≈ ln n + 0.5772       |
| Writing O(n) loops for sums | n(n+1)/2 is O(1) — never loop for this!     |
| with O(1) formulas          |                                              |
| Cancellation in alternating | Use pairwise summation to avoid error        |
| harmonic series             | accumulation in floating-point               |
| Missing telescoping pattern | Write out first few terms to see cancellation|
| Geometric r=1 edge case     | Special-case to S_n = n×a for r=1           |
+-----------------------------+----------------------------------------------+
```

---

## Summary Table

```
+--------------------------------+------------------+------------------------+
| Series                         | Sum (first n)    | Grows as               |
+--------------------------------+------------------+------------------------+
| 1 + 2 + ... + n                | n(n+1)/2         | O(n^2)                 |
| 1^2 + 2^2 + ... + n^2          | n(n+1)(2n+1)/6   | O(n^3)                 |
| 1^3 + 2^3 + ... + n^3          | [n(n+1)/2]^2     | O(n^4)                 |
| a+(a+d)+...+(a+(n-1)d)         | n/2 × (2a+(n-1)d)| O(n^2) for arithmetic  |
| a+ar+...+ar^(n-1)              | a(1-r^n)/(1-r)   | O(r^n) / O(1)          |
| 1+1/2+1/4+... (infinite, r=0.5)| 2                | converges to 2         |
| 1+1/2+1/3+... (harmonic)       | Diverges ≈ ln n  | O(log n)               |
| Σ 1/(k(k+1)) telescoping       | 1 - 1/(n+1)      | O(1)                   |
+--------------------------------+------------------+------------------------+

Decision: How to compute a sum?
  1. Closed form known?       → Use it, O(1)
  2. n small (< 10 million)?  → Iterative loop, O(n)
  3. n large, no closed form? → Approximation / NumPy vectorization
```

#adsa
