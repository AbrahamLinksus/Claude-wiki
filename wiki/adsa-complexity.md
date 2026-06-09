# Time & Space Complexity Analysis

**Summary**: The mathematical framework for measuring how an algorithm's resource usage — time and memory — grows as input size increases, enabling you to predict and compare algorithm performance before writing a single line of production code.

**Pre-Req**: [[adsa-arrays]] — familiarity with basic loops and memory layout helps; no formal math required.

**Sources**: `1 - Time-and-Space-Complexity-Analysis.pdf` (Talenciaglobal)

**Last updated**: 2026-06-09

---

## What Is Complexity Analysis?

When you write an algorithm, two questions immediately matter:

1. **How long does it take to run?** (time complexity)
2. **How much memory does it need?** (space complexity)

You could just time it with a stopwatch — but that answer only applies to *your* machine, *your* data, and *today's* hardware. Complexity analysis gives you a **machine-independent** answer that tells you how the algorithm behaves as the input grows from 10 items to 10 million items.

**Time complexity** counts the number of basic operations (comparisons, additions, assignments) an algorithm performs as a function of input size `n`.

**Space complexity** counts the amount of memory consumed — including the input itself, any extra arrays, and the call stack built up by recursion.

**Core purpose**: Make informed decisions about algorithm selection *before* writing production code, so systems remain responsive and cost-effective at scale.

### The Highway Analogy

Think of driving from city A to city B. Time complexity tells you how travel time changes with distance:

- A direct highway → O(n) — time grows linearly with distance
- Visiting every side street → O(n²) — time explodes as the city grows
- Teleporting → O(1) — arrival time is constant, regardless of distance

The route you choose determines whether you arrive in minutes or days. At scale, algorithm choice determines whether your system survives.

### Why It Matters in Practice

Early Facebook, Twitter, and many e-commerce platforms suffered massive outages due to quadratic algorithms in news feeds and search. Companies wasted millions on hardware while still facing performance cliffs.

Concrete numbers make this real:

| n (input size) | O(n) ops | O(n log n) ops | O(n²) ops |
|---------------|----------|----------------|-----------|
| 1,000 | 1,000 | ~10,000 | 1,000,000 |
| 100,000 | 100,000 | ~1,700,000 | 10,000,000,000 |
| 1,000,000 | 1,000,000 | ~20,000,000 | 10^12 (infeasible) |

An O(n²) sort on 1 million records requires roughly 10^12 operations — completely infeasible in production. O(n log n) reduces this to ~20 million operations, making real-time processing achievable. That is a **50,000x difference** for the same 1 million records.

---

## Asymptotic Notation

Asymptotic notation describes how a function *grows* as `n` gets large. Three notations form the language of complexity analysis.

### Big-O — Upper Bound

**Big-O (O)** says: "the algorithm never does *worse* than this rate."

Formal definition: f(n) = O(g(n)) if there exist constants `c > 0` and `n₀` such that for all `n ≥ n₀`:

```
f(n) ≤ c × g(n)
```

Visually — after some threshold n₀, the O function is always above f(n):

```
running
 time
  |                             c·g(n)  ← upper bound
  |                          ../
  |                       ../
  |               f(n) ../
  |            ../ ../
  |         ../. /
  |      ../ /
  |---n₀-----------→ n
```

Big-O is the most commonly used notation in practice because it **guarantees the algorithm won't exceed this rate** — you can promise your users worst-case behavior.

### Big-Omega — Lower Bound

**Big-Omega (Ω)** says: "the algorithm will *always* take at least this long, regardless of input."

Formal definition: f(n) = Ω(g(n)) if there exist constants `c > 0` and `n₀` such that for all `n ≥ n₀`:

```
f(n) ≥ c × g(n)
```

Visually — f(n) is always *above* the Ω function after n₀:

```
running
 time
  |       f(n)
  |      /
  |     /   c·g(n)  ← lower bound (always below f)
  |    /  /
  |   / /
  |  //
  |-----------→ n
```

Omega is useful for proving **no algorithm can do better** than a certain rate for a given problem class. For example, sorting by comparisons has Ω(n log n) — no comparison-based sort can beat n log n.

### Big-Theta — Tight Bound

**Big-Theta (Θ)** says: "the function grows *exactly* at this rate — neither faster nor slower."

f(n) = Θ(g(n)) means f(n) = O(g(n)) AND f(n) = Ω(g(n)) simultaneously. There exist `c₁, c₂ > 0` such that:

```
c₁ × g(n) ≤ f(n) ≤ c₂ × g(n)
```

Visually — f(n) is sandwiched between two multiples of g(n):

```
running
 time
  |          c₂·g(n)  ← ceiling
  |         /   f(n)  ← actual (stays between)
  |        /  /
  |       / /
  |      //  c₁·g(n) ← floor
  |     /  /
  |----//----------→ n
```

Theta is the **most precise and meaningful notation** for characterizing algorithm behavior. For production systems, always prefer Theta-bounded algorithms when possible — they give predictable, reliable performance guarantees for SLAs.

### Comparison Table

| Notation | Meaning | Analogy | Use case |
|----------|---------|---------|----------|
| O(g(n)) | Upper bound | "at most this fast" | Worst-case guarantees |
| Ω(g(n)) | Lower bound | "at least this fast" | Proving hardness limits |
| Θ(g(n)) | Tight bound | "exactly this rate" | Precise characterization |

---

## The Growth Rate Hierarchy

Seven complexity classes appear constantly in practice, arranged from fastest to slowest growth:

```
SLOW (good)                                               FAST (bad)
  O(1) → O(log n) → O(n) → O(n log n) → O(n²) → O(2ⁿ) → O(n!)

 Constant   Logarithmic  Linear  Linearithmic  Quadratic  Exponential  Factorial
```

As a staircase diagram (each step is dramatically worse):

```
              ┌──────────────────────┐
              │  O(2ⁿ) / O(n!)       │  Exponential / Factorial
              │  Only for tiny n     │
         ┌────┤                      │
         │    └──────────────────────┘
    ┌────┤
    │    │  O(n²) / O(n³)  Quadratic / Cubic
    │    │  Avoid at scale
┌───┤    │
│   │  O(n log n)  Linearithmic
│   │  Common for sorting
│
│  O(n)  Linear  — base performance
│
│  O(log n)  Logarithmic
│
│  O(1)  Constant
└──────────────────────────────────→ n
```

Each step up this hierarchy represents a **dramatic** increase in resource consumption. An O(n²) algorithm on 1 million records means approximately 1 trillion operations — impossible in real time.

---

## Common Complexities: Deep Dive with Worked Examples

### O(1) — Constant

The number of operations does not depend on input size at all.

```python
def get_first(arr):
    return arr[0]        # always 1 operation, regardless of arr length
```

**Concrete example**: Array has 5 elements or 5 million — `arr[0]` is always one memory lookup.

```
arr = [42, 17, 8, 99, 3]
get_first(arr) → 1 operation: read arr[0] = 42

arr = [42, 17, 8, ..., 5000000 elements ..., 3]
get_first(arr) → still 1 operation: read arr[0] = 42
```

Hash table lookup (average case) is O(1). The trade-off: usually requires extra space or preprocessing.

---

### O(log n) — Logarithmic

Each step **halves** the remaining problem. If n doubles, you only do one more operation.

```python
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target:   return mid
        elif arr[mid] < target:  lo = mid + 1
        else:                    hi = mid - 1
    return -1
```

**Concrete example**: Search for 7 in `[1, 3, 5, 7, 9, 11, 13]`

```
Array:  [1, 3, 5, 7, 9, 11, 13]
         0  1  2  3  4   5   6

Step 1: lo=0, hi=6 → mid=3 → arr[3]=7 → FOUND in 1 step

Harder example, search for 11:
Step 1: lo=0, hi=6, mid=3, arr[3]=7 < 11  → lo = 4
Step 2: lo=4, hi=6, mid=5, arr[5]=11 = 11 → FOUND in 2 steps

For n=1,000,000: only ~20 steps needed (log₂(1,000,000) ≈ 20)
```

Requires sorted/preprocessed data. Scales extremely well.

---

### O(n) — Linear

The algorithm touches each element exactly once (or a constant number of times).

```python
def linear_search(arr, target):
    for i in range(len(arr)):   # n iterations
        if arr[i] == target:
            return i
    return -1
```

**Concrete example**: Search for 7 in `[3, 1, 9, 2, 7, 5]`

```
i=0: arr[0]=3 ≠ 7
i=1: arr[1]=1 ≠ 7
i=2: arr[2]=9 ≠ 7
i=3: arr[3]=2 ≠ 7
i=4: arr[4]=7 = 7 → return 4    (5 comparisons for n=6)
```

In the worst case (target not present): n comparisons. Good for one-time processing; expensive at billions of records.

---

### O(n log n) — Linearithmic

The algorithm does O(log n) work for each of n elements, or divides the problem log n times with O(n) work at each level.

Merge sort is the canonical example. At each of the log n levels of recursion, you do O(n) work merging:

```
Level 0 (1 merge of n):          n comparisons total
Level 1 (2 merges of n/2):       n comparisons total
Level 2 (4 merges of n/4):       n comparisons total
...
Level log n (n merges of 1):     n comparisons total

Total: n × log n comparisons
```

Sweet spot for sorting large datasets. Higher constants than linear, but vastly better than O(n²).

---

### O(n²) — Quadratic

Typically two nested loops, each running n times.

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):           # outer: n times
        for j in range(n - i):   # inner: ~n times
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
```

**Concrete example**: `arr = [4, 3, 2, 1]`, n=4

```
i=0: j runs 0,1,2,3 → 4 comparisons
i=1: j runs 0,1,2   → 3 comparisons
i=2: j runs 0,1     → 2 comparisons
i=3: j runs 0       → 1 comparison

Total: 4+3+2+1 = 10 comparisons for n=4
For n=1000: ~500,000 comparisons
For n=1,000,000: ~500,000,000,000 comparisons (catastrophic)
```

Acceptable only for n < 10,000.

---

### O(2ⁿ) — Exponential

Each additional input element **doubles** the work. Classic example: generating all subsets.

```python
def all_subsets(arr):
    if not arr:
        return [[]]
    rest = all_subsets(arr[1:])
    return rest + [[arr[0]] + s for s in rest]
```

**Concrete example**: `arr = [1, 2, 3]`

```
n=1: [1]           → 2¹ = 2  subsets: [], [1]
n=2: [1,2]         → 2² = 4  subsets: [], [1], [2], [1,2]
n=3: [1,2,3]       → 2³ = 8  subsets
n=40:              → 2⁴⁰ ≈ 1 trillion subsets
```

Only feasible for very small n (< 40). Requires dynamic programming or heuristics at scale.

---

### O(n!) — Factorial

Grows faster than exponential. Generating all permutations or brute-force travelling salesman.

```
n=1: 1
n=5: 120
n=10: 3,628,800
n=15: 1,307,674,368,000   (over 1 trillion)
n=20: 2,432,902,008,176,640,000
```

Only usable for n ≤ 10–12 in practice.

---

## Asymptotic Simplification Rules

When you count operations, the raw formula often contains multiple terms and constants. Asymptotic analysis strips them away to reveal the dominant growth behavior.

**Rule 1 — Drop constants:**
```
5n²      → O(n²)     (the 5 doesn't change growth shape)
100n     → O(n)
3n log n → O(n log n)
```

**Rule 2 — Drop lower-order terms:**
```
n² + n           → O(n²)      (n² dominates n for large n)
n³ + 100n² + 10  → O(n³)
n log n + n      → O(n log n)
```

**Rule 3 — Nested loops multiply:**
```
Outer loop n times, inner loop n times → O(n) × O(n) = O(n²)
Outer loop n times, inner loop log n times → O(n log n)
```

**Rule 4 — Sequential code adds:**
```
Loop A doing O(n), then Loop B doing O(n²) → O(n) + O(n²) = O(n²)
```

---

## Best Case, Average Case, Worst Case

Every algorithm has three complexity scenarios. Always analyze all three.

### Worked Example: Linear Search

```
arr = [3, 7, 1, 9, 2, 5, 8, 4]    (n = 8)
```

**Best case** — target is the first element:
```
Search for 3:
i=0: arr[0]=3 → FOUND
→ 1 comparison = O(1) best case
```

**Average case** — target is somewhere in the middle (on average, n/2 comparisons):
```
Search for 9:
i=0: 3≠9
i=1: 7≠9
i=2: 1≠9
i=3: 9=9 → FOUND
→ 4 comparisons ≈ n/2 = O(n) average case
```

**Worst case** — target is the last element or not present:
```
Search for 4 (last element):
i=0: 3≠4
i=1: 7≠4
i=2: 1≠4
i=3: 9≠4
i=4: 2≠4
i=5: 5≠4
i=6: 8≠4
i=7: 4=4 → FOUND
→ 8 comparisons = n = O(n) worst case

Search for 6 (not present):
→ All 8 comparisons, return -1 = O(n)
```

**Summary table for linear search:**

| Case | Condition | Comparisons | Notation |
|------|-----------|-------------|----------|
| Best | Target at index 0 | 1 | O(1) |
| Average | Target at random position | n/2 | O(n) |
| Worst | Target at end or missing | n | O(n) |

For production systems, **always analyze worst case**. Average case is useful context but never a guarantee — production systems will hit worst cases.

---

## How to Analyze Loops

### Single Loop

Count exactly how many times the loop body executes.

```python
for i in range(n):       # n iterations
    print(i)             # O(1) work each

Total: n × O(1) = O(n)
```

---

### Nested Loops

Multiply the counts when loops are nested.

```python
for i in range(n):           # n iterations
    for j in range(n):       # n iterations
        print(i, j)          # O(1) work each

Total: n × n × O(1) = O(n²)
```

**Non-symmetric nested loop:**

```python
for i in range(n):           # n iterations
    for j in range(i):       # i iterations (0, 1, 2, ..., n-1)
        print(i, j)

Total iterations: 0 + 1 + 2 + ... + (n-1) = n(n-1)/2 → O(n²)
```

---

### Loops That Halve the Input

When a loop variable is divided (not decremented) each iteration, the count is logarithmic.

```python
i = n
while i > 1:
    print(i)
    i = i // 2          # halve each time
```

**Trace with n = 16:**

```
i = 16  → print
i = 8   → print
i = 4   → print
i = 2   → print
i = 1   → stop

4 iterations = log₂(16)
```

How many times can you halve n before reaching 1? log₂(n) times. Therefore: **O(log n)**.

---

### Mixed Loop Example

```python
for i in range(n):       # n iterations (outer)
    j = n
    while j > 1:         # log n iterations (inner, halving)
        j = j // 2

Total: n × log n = O(n log n)
```

---

## Recurrence Relations

When an algorithm calls itself recursively, you can't count iterations directly. Instead you write a **recurrence relation** — an equation that defines the runtime T(n) in terms of the runtime of smaller inputs.

### What They Are

A recurrence has two parts:
1. **Base case**: the cost for the trivially small input
2. **Recursive case**: the cost of one call in terms of smaller calls

### Binary Search Recurrence

Binary search on n elements:
- Does O(1) work (one comparison)
- Then recurses on n/2 elements

```
T(n) = T(n/2) + O(1)
T(1) = O(1)           ← base case
```

Solving by expansion:
```
T(n) = T(n/2) + 1
     = T(n/4) + 1 + 1
     = T(n/8) + 1 + 1 + 1
     = T(1)  + log₂(n) × 1
     = O(log n)
```

### Merge Sort Recurrence

Merge sort on n elements:
- Splits into 2 halves: 2 recursive calls on n/2 elements
- Merges them: O(n) work

```
T(n) = 2T(n/2) + O(n)
T(1) = O(1)
```

Solving by expansion (drawing the recursion tree):
```
Level 0:   n work                     (1 call,  n work each)
Level 1:   n work total               (2 calls, n/2 work each)
Level 2:   n work total               (4 calls, n/4 work each)
...
Level k:   n work total               (2^k calls, n/2^k work each)
...
Level log n: n work total             (n calls, 1 work each)

Total levels: log n
Total work:   n × log n = O(n log n)
```

---

## The Master Theorem

The Master Theorem is a formula that solves many divide-and-conquer recurrences in one step without expanding them manually.

**General form**: `T(n) = aT(n/b) + f(n)`

Where:
- `a` = number of subproblems each call creates
- `b` = factor by which the input shrinks
- `f(n)` = work done outside the recursive calls (split + combine step)

The theorem compares `f(n)` against `n^(log_b a)` — the "critical exponent":

```
                    critical exponent = log_b(a)
                           ↑
       Compare f(n) against n^(log_b a)
```

### Three Cases

**Case 1 — Work decreases toward leaves**: f(n) = O(n^(log_b a - ε)) for some ε > 0
- The recursive calls dominate. Result: **T(n) = Θ(n^(log_b a))**

**Case 2 — Work is equal at every level**: f(n) = Θ(n^(log_b a))
- All levels do equal work. Result: **T(n) = Θ(n^(log_b a) × log n)**

**Case 3 — Work increases toward root**: f(n) = Ω(n^(log_b a + ε)) for some ε > 0, and a×f(n/b) ≤ c×f(n)
- The root call dominates. Result: **T(n) = Θ(f(n))**

### Case-by-Case Examples

**Example 1: Binary Search** — `T(n) = T(n/2) + O(1)`
```
a=1, b=2, f(n)=1
Critical exponent: log₂(1) = 0  →  n⁰ = 1

Compare: f(n) = 1 = Θ(n⁰) = Θ(1)   ← Case 2

Result: T(n) = Θ(n⁰ × log n) = Θ(log n)   ✓
```

**Example 2: Merge Sort** — `T(n) = 2T(n/2) + O(n)`
```
a=2, b=2, f(n)=n
Critical exponent: log₂(2) = 1  →  n¹ = n

Compare: f(n) = n = Θ(n¹) = Θ(n)   ← Case 2

Result: T(n) = Θ(n¹ × log n) = Θ(n log n)   ✓
```

**Example 3: Strassen Matrix Multiplication** — `T(n) = 7T(n/2) + O(n²)`
```
a=7, b=2, f(n)=n²
Critical exponent: log₂(7) ≈ 2.807  →  n^2.807

Compare: f(n) = n² = O(n^(2.807 - ε))   ← Case 1
(n² grows slower than n^2.807)

Result: T(n) = Θ(n^log₂7) ≈ Θ(n^2.807)
(Better than naive O(n³)!)
```

**Example 4: Naive Matrix Multiply** — `T(n) = 8T(n/2) + O(n²)`
```
a=8, b=2, f(n)=n²
Critical exponent: log₂(8) = 3  →  n³

Compare: f(n) = n² = O(n^(3 - 1))   ← Case 1

Result: T(n) = Θ(n³)   ✓
```

### Master Theorem Quick Reference

| Recurrence | a | b | f(n) | Case | Result |
|------------|---|---|------|------|--------|
| T(n/2) + O(1) | 1 | 2 | O(1) | 2 | O(log n) |
| 2T(n/2) + O(n) | 2 | 2 | O(n) | 2 | O(n log n) |
| 4T(n/2) + O(n) | 4 | 2 | O(n) | 1 | O(n²) |
| 2T(n/2) + O(n²) | 2 | 2 | O(n²) | 3 | O(n²) |
| 7T(n/2) + O(n²) | 7 | 2 | O(n²) | 1 | O(n^2.807) |

---

## Space Complexity

Space complexity measures how much memory an algorithm requires as a function of input size `n`.

### Two Types of Space

**Total space**: everything — input storage + auxiliary space + call stack.

**Auxiliary space**: extra memory beyond the input. This is usually the more useful measure because the input size is fixed by the problem.

### Auxiliary Space Examples

**In-place algorithm** (O(1) auxiliary):
```python
def swap(arr, i, j):
    arr[i], arr[j] = arr[j], arr[i]   # only 1 temp variable, no new array
```

**Linear auxiliary space** (O(n)):
```python
def merge(left, right):
    result = []            # new array of size n — O(n) extra
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    result.extend(left[i:]); result.extend(right[j:])
    return result
```

### Recursion Stack Space

Every recursive call pushes a frame onto the call stack. The depth of recursion determines extra stack space needed.

**Binary search — O(log n) stack:**
```
binary_search(arr, 0, n-1)
  binary_search(arr, 0, n/2)
    binary_search(arr, 0, n/4)
      ...
        binary_search(arr, 0, 1)   ← deepest frame

Depth = log₂(n) frames on the stack simultaneously
```

**Merge sort — O(n) stack** (due to O(log n) depth but O(n) total frames):
```
merge_sort(0..n)           depth 0
  merge_sort(0..n/2)       depth 1
    merge_sort(0..n/4)     depth 2
      ...
        merge_sort(0..1)   depth log n

Maximum frames at once: log n, each O(1) → O(log n) stack
But auxiliary arrays across all merge calls: O(n) total
→ Total auxiliary space: O(n)
```

**Quick sort — O(log n) average, O(n) worst case:**
- Average case: recursion depth ≈ log n → O(log n) stack
- Worst case (sorted input, bad pivot): depth n → O(n) stack

### Space for Common Algorithms

| Algorithm | Auxiliary Space | Notes |
|-----------|----------------|-------|
| Merge Sort | O(n) | Merge step needs temp arrays |
| Quick Sort | O(log n) avg | Recursion stack only; O(n) worst |
| Heap Sort | O(1) | In-place — sorts the input array directly |
| Binary Search | O(1) iter / O(log n) recursive | Iterative version uses no stack |
| Hash Table | O(n) | Space-time trade-off for O(1) lookups |

---

## Time vs Space Trade-Off

More memory often reduces time. This tension is fundamental.

```
FAST BUT MEMORY-HUNGRY                  SLOW BUT MEMORY-EFFICIENT
        ←──────────────────────────────────────────→
   Hash lookup O(1)                  Linear scan O(n)
   uses O(n) memory                  uses O(1) memory

   Precomputed lookup table           Recompute every time
   Memoization (DP)                   Recursive recomputation
   Prefix sum array                   Loop over range each query
```

### Classic Trade-Off: Prefix Sum vs Direct Scan

Suppose you have array A and need to answer 1000 range-sum queries.

**Naive (no extra space):**
```
Each query sum(A[l..r]): O(n) scan
1000 queries × O(n) each = O(1000n) total
```

**Prefix sum (O(n) extra space):**
```
Build prefix array P: O(n) once
Each query: O(1) lookup
1000 queries × O(1) = O(n + 1000) total → dramatic speedup
```

(See [[adsa-arrays]] for full prefix sum details.)

### Trade-Off Summary Table

| Approach | Time | Space | Best when |
|----------|------|-------|-----------|
| Precompute / cache | O(1) query | O(n) extra | Many queries, static data |
| On-the-fly compute | O(n) query | O(1) extra | Few queries, memory scarce |
| Memoization (DP) | O(n) total | O(n) extra | Overlapping subproblems |
| Tail-recursive | O(n) | O(1) stack | Recursion depth is a concern |
| Hash table | O(1) avg | O(n) | Fast lookups are priority |

---

## The Four-Step Analysis Process

When you encounter a piece of code, apply this systematic process:

```
Step 1: IDENTIFY OPERATIONS
   → What are the basic operations? (comparisons, assignments, arithmetic)

Step 2: COUNT IN TERMS OF n
   → How many times does each run as a function of n?

Step 3: APPLY ASYMPTOTIC RULES
   → Drop constants, drop lower-order terms

Step 4: EXPRESS WITH NOTATION
   → Write the result as O(...), Ω(...), or Θ(...)
```

**Applied example — find maximum element:**

```python
def find_max(arr):
    max_val = arr[0]        # Step 1: 1 assignment
    for i in range(1, n):   # Step 2: n-1 iterations
        if arr[i] > max_val:    # 1 comparison per iteration
            max_val = arr[i]    # at most 1 assignment per iteration
    return max_val          # 1 return
```

Step 1: Assignment, comparison, conditional assignment, return.
Step 2: 1 + (n-1)×2 + 1 = 2n operations.
Step 3: Drop the 2 constant → n operations.
Step 4: **T(n) = Θ(n)** — must touch every element at least once.

---

## Scalability Quick Guide

The right algorithm depends on the scale of your data:

| Scale | n range | Acceptable complexities | Notes |
|-------|---------|------------------------|-------|
| Small | n < 10,000 | O(n²) and better | Most algorithms work; focus on clarity |
| Medium | 10K – 1M | O(n log n) or better | Avoid quadratic; profile with real data |
| Enterprise | 1M – billions | O(n log n) or better | Distributed approaches required (Spark, MapReduce) |

**Real cost of O(n²) at scale**: A payment gateway using naive O(n²) fraud detection on 10 million daily transactions would require massive over-provisioning — and still fail. Proper analysis leads to O(n log n) or better solutions using sorted features and binary search.

---

## Common Mistakes to Avoid

| Level | Mistake | Consequence |
|-------|---------|-------------|
| Beginner | Dropping constants incorrectly | Misclaiming O(n) when it's really O(n²) |
| Beginner | Confusing best-case with worst-case | System fails under real load |
| Intermediate | Forgetting recursion stack space | Stack overflow on large n |
| Intermediate | Assuming average case always occurs | Production outages |
| Advanced | Ignoring cache misses | Theory says O(n) but hardware says 10× slower |
| Production | Not re-analyzing after data volume growth | Assumptions expire at new scale |

---

## Cheat Sheet

```
Pattern                     Complexity      Example
──────────────────────────────────────────────────────
Single loop over n          O(n)            linear search
Two nested loops            O(n²)           bubble sort
Halving each step           O(log n)        binary search
Loop + halving inner        O(n log n)      merge sort
Three nested loops          O(n³)           naive 3-sum
Divide & Conquer            Master Theorem  merge sort, binary search
Generate all subsets        O(2ⁿ)           power set
Generate all permutations   O(n!)           brute force TSP
```

Related pages: [[adsa-sorting]], [[adsa-arrays]], [[adsa-stacks]]

#adsa
