# Arrays & Dynamic Arrays

**Summary**: Static arrays, dynamic array reallocation, amortized O(1) append, prefix sum arrays for O(1) range queries, and difference arrays for O(1) range updates.

**Pre-Req**: Basic understanding of memory layout (stack vs heap), Big-O notation.

**Sources**: Session-built (no raw file). Updated with `raw/2 - Arrays.pdf`.

**Last updated**: 2026-06-09 (updated: 2D arrays, binary search, array rotation)

---

## Static Arrays

A static array is a contiguous block of memory holding elements of the same type. Because elements are contiguous:

- **Random access** is O(1) — `base_address + index * element_size` is a direct pointer arithmetic calculation.
- **Size is fixed** at allocation time. You cannot grow it in place.

If you need more capacity than you allocated, you must allocate a new block and copy everything over — O(n).

---

## Dynamic Arrays

A dynamic array (e.g. Python `list`, C++ `std::vector`, Java `ArrayList`) wraps a static array and handles growth automatically. It tracks three things:

| Field | Meaning |
|-------|---------|
| `data` | pointer to the underlying static array |
| `size` | number of elements currently stored |
| `capacity` | number of slots allocated |

When `size == capacity` and you append, the dynamic array **reallocates**.

---

## Reallocation

Reallocation is the process of growing the underlying buffer:

1. Allocate a **new**, larger block of memory.
2. **Copy** all `size` existing elements into the new block.
3. **Free** the old block.
4. Update `data` to point to the new block, update `capacity`.

A single reallocation costs **O(n)** — copying n elements is unavoidable.

The key design question is: **how much larger should the new block be?**

---

## Growth Factor and Amortized O(1) Append

### Naive strategy — grow by 1

If you grow capacity by 1 each time, every append triggers a reallocation. Appending n elements costs:

```
1 + 2 + 3 + ... + n = n(n+1)/2  →  O(n²) total
```

Per append: **O(n)** amortized. Terrible.

### Doubling strategy — grow by 2×

The standard approach: when full, allocate `2 × capacity`.

Reallocation events happen at capacities: 1, 2, 4, 8, 16, ..., up to n.
Copy costs at each event: 1, 2, 4, 8, ..., n.

Total copy work for n appends:

```
1 + 2 + 4 + ... + n  =  2n - 1  →  O(n) total
```

Spread across n appends: **O(n) / n = O(1) amortized per append**.

### Amortized Analysis (Accounting / Potential Method)

Assign each element a "credit" of 2 units when it is written:
- 1 credit pays for its own insertion.
- 1 credit is saved to pay for copying it in the next reallocation.

At any reallocation, the right half of the array (elements added since the last reallocation) have exactly enough saved credit to pay for copying the entire array. The amortized cost is O(1) per append.

> The actual cost of individual appends alternates between O(1) (no reallocation) and O(n) (reallocation), but the **average over any sequence of n appends** is O(1).

### Growth factor trade-off

| Growth factor | Memory waste | Reallocation frequency |
|--------------|-------------|----------------------|
| 1.25× | Low | High |
| 1.5× | Moderate | Moderate |
| 2× | Up to 50% wasted | Low (standard choice) |
| 3× | Up to 67% wasted | Very low |

Any constant factor > 1 gives amortized O(1) append. Factor of 2 is the common default. Some implementations (e.g. CPython) use a sub-2 factor to save memory.

---

---

## Prefix Sum Array

Used when you have a **static** array and need to answer many range sum queries efficiently.

**Construction** — O(n):
```
P[0] = A[0]
P[i] = P[i-1] + A[i]
```

**Range sum query** `sum(A[l..r])` — O(1):
```
P[r] - P[l-1]       (define P[-1] = 0 for l = 0)
```

Without prefix sums, each range query costs O(n). With them, you pay O(n) once at build time and answer any number of queries in O(1).

**Constraint**: The underlying array must not change. Any update to `A` invalidates `P` and requires a full O(n) rebuild.

---

## Difference Array

The dual of prefix sums — used when you need many **range updates** (add a constant to every element in a range), then read the final state of the array once.

**Construction** from array `A` — O(n):
```
D[0] = A[0]
D[i] = A[i] - A[i-1]
```

**Range update**: add value `v` to every element in `A[l..r]` — O(1):
```
D[l]   += v
D[r+1] -= v      (skip if r+1 >= n)
```

**Reconstruct** final `A` from `D` via prefix sum — O(n):
```
A[0] = D[0]
A[i] = A[i-1] + D[i]
```

**Why it works**: `D[l] += v` propagates `v` forward through the prefix sum to all indices ≥ l. `D[r+1] -= v` cancels it at index r+1, bounding the update to exactly `[l, r]`.

**Cost comparison** for k range updates:

| Approach | Per update | Reconstruct | Total |
|----------|-----------|-------------|-------|
| Naive (update A directly) | O(n) | — | O(k·n) |
| Difference array | O(1) | O(n) | O(k + n) |

---

## Kadane's Algorithm — Maximum Subarray

Given an array of integers (negatives allowed), find the contiguous subarray with the largest sum.

At each index `i`, make a greedy local decision: extend the previous subarray or start fresh at `A[i]`.

```
current = max(A[i], current + A[i])
```

- If `current + A[i] < A[i]` — the running sum is a liability, drop it and restart.
- Otherwise, extend.

`current` always holds the maximum subarray sum **ending exactly at i**. The global answer is the best `current` across all indices.

```
current = A[0]
best    = A[0]

for i in 1..n-1:
    current = max(A[i], current + A[i])
    best    = max(best, current)

return best
```

Starting from `A[0]` (not 0) handles all-negative arrays correctly — returns the least negative element rather than 0.

**Complexity**: O(n) time, O(1) space.

---

## Summary Table

| Operation | Static Array | Dynamic Array | Prefix Sum Array | Difference Array |
|-----------|-------------|---------------|-----------------|-----------------|
| Access by index | O(1) | O(1) | O(1) | O(n) reconstruct |
| Range sum query | O(n) | O(n) | **O(1)** | — |
| Range update (+v to [l,r]) | O(n) | O(n) | — | **O(1)** |
| Append (amortized) | — | **O(1)** | — | — |
| Build cost | — | — | O(n) | O(n) |

---

## 2D and Multidimensional Arrays

A 2D array is conceptually a grid of rows and columns. In memory it is still a flat, contiguous block — the 2D structure is an illusion maintained by index arithmetic.

### Memory Layout: Row-Major vs Column-Major

```
2D array A[3][4] (3 rows, 4 columns):

Conceptual grid:
  col:   0    1    2    3
row 0: [ a00| a01| a02| a03 ]
row 1: [ a10| a11| a12| a13 ]
row 2: [ a20| a21| a22| a23 ]
```

**Row-major (C, C++, Python NumPy default)**: rows are stored contiguously in memory.

```
Physical memory:
[ a00, a01, a02, a03, a10, a11, a12, a13, a20, a21, a22, a23 ]
  └──── row 0 ────┘  └──── row 1 ────┘  └──── row 2 ────┘
```

**Column-major (Fortran, MATLAB, R)**: columns are stored contiguously.

```
Physical memory:
[ a00, a10, a20, a01, a11, a21, a02, a12, a22, a03, a13, a23 ]
  └─col0─┘  └─col1─┘  └─col2─┘  └─col3─┘
```

### Index Formula

For a row-major 2D array with `cols` columns, element at row `i`, column `j`:

```
flat_index = i * cols + j
```

Example — A[3][4], access A[2][1]:
```
flat_index = 2 * 4 + 1 = 9
```

Verify: row 0 = indices 0–3, row 1 = 4–7, row 2 = 8–11. A[2][1] = index 9. Correct.

For column-major with `rows` rows:
```
flat_index = j * rows + i
```

### Cache Impact of Access Pattern

The layout choice dramatically affects performance. A cache line loads ~64 bytes (8 integers) at once. Accessing elements in the stored order causes sequential cache loads; going against the storage order causes a cache miss per element.

```
Row-major array, iterating row by row (GOOD):
  for i in range(rows):
    for j in range(cols):
      process(A[i][j])       ← accesses a00,a01,a02,a03... in memory order
                             ← each cache line loads 8 elements ahead

Row-major array, iterating column by column (BAD):
  for j in range(cols):
    for i in range(rows):
      process(A[i][j])       ← accesses a00,a10,a20,a01... jumps rows in memory
                             ← every access is a cache miss if rows > cache line size
```

Real-world consequence: traversing a 1000×1000 matrix in the wrong order can be **5–10× slower** than the correct order on modern hardware (source: `raw/2 - Arrays.pdf`). NumPy/PyTorch memory layout (C vs Fortran order) directly impacts deep learning training throughput for this reason.

### When to Use Which Layout

| Layout | Language Default | Fast For | Use Case |
|---|---|---|---|
| Row-major | C, C++, Python, Java | Row traversal | Image processing (pixels per row), matrix-vector multiply |
| Column-major | Fortran, MATLAB, R | Column traversal | Statistics (column = one variable), matrix-matrix multiply in BLAS |

---

## Binary Search on Sorted Arrays

Binary search solves the question "is target in this sorted array, and if so where?" in **O(log n)** by repeatedly halving the search space.

**Precondition**: the array must be sorted in ascending order. Binary search on an unsorted array produces incorrect results without error.

### The lo-hi-mid Pattern

```
FUNCTION binarySearch(arr, target):
    lo = 0
    hi = len(arr) - 1

    while lo <= hi:
        mid = lo + (hi - lo) / 2    ← avoid integer overflow vs (lo+hi)/2
        if arr[mid] == target:
            RETURN mid              ← found
        elif arr[mid] < target:
            lo = mid + 1            ← target is in right half
        else:
            hi = mid - 1            ← target is in left half

    RETURN -1                       ← not found
```

### Worked Example — 10-Element Array

```
arr = [3, 7, 11, 15, 19, 23, 28, 34, 41, 50]
idx:   0  1   2   3   4   5   6   7   8   9

Search for target = 23:

Iteration 1:
  lo=0, hi=9, mid=4
  arr[4]=19 < 23  →  lo = mid+1 = 5

Iteration 2:
  lo=5, hi=9, mid=7
  arr[7]=34 > 23  →  hi = mid-1 = 6

Iteration 3:
  lo=5, hi=6, mid=5
  arr[5]=23 == 23  →  RETURN 5

Total comparisons: 3  (vs 10 for linear search)
```

Search for target = 99 (not present):

```
Iteration 1: lo=0, hi=9, mid=4, arr[4]=19 < 99 → lo=5
Iteration 2: lo=5, hi=9, mid=7, arr[7]=34 < 99 → lo=8
Iteration 3: lo=8, hi=9, mid=8, arr[8]=41 < 99 → lo=9
Iteration 4: lo=9, hi=9, mid=9, arr[9]=50 < 99 → lo=10
lo > hi: RETURN -1
```

### Complexity

| n | Max comparisons (log₂ n) |
|---|---|
| 10 | 4 |
| 1,000 | 10 |
| 1,000,000 | 20 |
| 1,000,000,000 | 30 |

At n=10^9, binary search finds the answer in 30 comparisons. Linear search averages 500 million.

### Common Off-by-One Pitfalls

```
WRONG: mid = (lo + hi) / 2
  Problem: lo+hi can overflow int on large arrays
  Fix: mid = lo + (hi - lo) / 2

WRONG: while lo < hi:
  Problem: misses single-element case where lo==hi is the answer
  Fix: while lo <= hi

WRONG: lo = mid (instead of mid+1) when arr[mid] < target
  Problem: infinite loop when lo==mid
  Fix: always lo = mid + 1, hi = mid - 1

WRONG: hi = len(arr) (instead of len(arr)-1)
  Problem: arr[mid] out of bounds when lo==hi==last valid index+1
  Fix: hi = len(arr) - 1
```

---

## Array Rotation

Rotating an array right by k positions means each element shifts k places to the right, with elements wrapping around from the end to the front.

```
Original: [1, 2, 3, 4, 5, 6, 7]   k=3
Rotated:  [5, 6, 7, 1, 2, 3, 4]
          └─moved─┘ └─stayed──┘
```

### Naive Approach — O(k × n)

Rotate by 1 position k times. Each single rotation is O(n). Total: O(k × n).

```
rotate_right_by_one(arr):
    last = arr[n-1]
    for i from n-1 down to 1:
        arr[i] = arr[i-1]
    arr[0] = last
```

For n=1000, k=333: 333,000 operations. Slow.

### Reversal Trick — O(n), In-Place

The key insight: rotate right by k = reverse the whole array, then reverse the first k elements, then reverse the last n-k elements.

```
FUNCTION reverseRange(arr, lo, hi):
    while lo < hi:
        swap(arr[lo], arr[hi])
        lo++; hi--

FUNCTION rotateRight(arr, k):
    n = len(arr)
    k = k % n                   ← handle k > n
    reverseRange(arr, 0, n-1)   ← Step 1: reverse entire array
    reverseRange(arr, 0, k-1)   ← Step 2: reverse first k elements
    reverseRange(arr, k, n-1)   ← Step 3: reverse remaining n-k elements
```

Step-by-step trace on [1, 2, 3, 4, 5, 6, 7], k=3:

```
Original:            [1, 2, 3, 4, 5, 6, 7]

Step 1 — reverse all:
  [7, 6, 5, 4, 3, 2, 1]

Step 2 — reverse first k=3:
  reverse [7,6,5] → [5,6,7]
  Result: [5, 6, 7, 4, 3, 2, 1]

Step 3 — reverse last n-k=4:
  reverse [4,3,2,1] → [1,2,3,4]
  Result: [5, 6, 7, 1, 2, 3, 4]  ← correct!
```

**Why it works**: reversing all scrambles everything; reversing the two segments in place restores their internal order but in the new positions.

Complexity: **O(n) time, O(1) space** — only 3 passes, each doing n/2 swaps at most (source: `raw/2 - Arrays.pdf`).

### Comparison

| Approach | Time | Space | Notes |
|---|---|---|---|
| Naive (rotate-by-1, k times) | O(k×n) | O(1) | Simple but slow |
| Extra array | O(n) | O(n) | Copy to shifted positions, then copy back |
| Reversal trick | O(n) | O(1) | Best: fast and in-place |

### Common Mistakes

```
Forgetting k % n:
  If k == n, rotation is a no-op. k=7 on n=7 array → k%7 = 0.
  Without this, the reversal still gives the right answer but
  wastes 3 full passes. More importantly, if k > n it wraps.

Rotating left vs right:
  Rotate right by k = rotate left by (n-k).
  To rotate left by k: reverseRange(0,k-1), reverseRange(k,n-1), reverseRange(0,n-1).
  Same 3 steps, different order.
```

#adsa
