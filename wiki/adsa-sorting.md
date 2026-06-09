# Advanced Sorting Algorithms

**Summary**: A deep dive into Merge Sort, Quick Sort, Heap Sort, Counting Sort, and Radix Sort — the algorithms powering modern data systems — with full step-by-step illustrations, complexity proofs, and a selection guide.

**Pre-Req**: [[adsa-complexity]] — Big-O notation, recurrence relations, and the Master Theorem are used throughout. [[adsa-arrays]] — dynamic arrays and in-place operations.

**Sources**: `2 - Advanced-Sorting-Algorithms-in-Complexity-Analysis.pdf` (Talenciaglobal)

**Last updated**: 2026-06-09

---

## Why Sorting Matters

Sorting is among the most fundamental operations in computing. Searching, ranking, deduplication, join operations in databases, binary search, compression — all of them either require sorted data or become dramatically more efficient when input is sorted.

The scale gap between a naive sort and an advanced sort is enormous:

| Input size | O(n²) operations | O(n log n) operations | Ratio |
|------------|-----------------|----------------------|-------|
| 1,000 | 1,000,000 | ~10,000 | 100× |
| 100,000 | 10,000,000,000 | ~1,700,000 | ~6,000× |
| 1,000,000 | 10^12 (infeasible) | ~20,000,000 | ~50,000× |

An O(n²) sort on 1 million records requires roughly 10^12 operations — completely infeasible in production. O(n log n) reduces this to ~20 million operations. That 50,000× difference directly translates to server costs and user-facing latency.

**Real impact**: Retail platforms sort 50 million product recommendations nightly. Without O(n log n) sorting, this pipeline would require 100× more infrastructure. Early Facebook and Twitter suffered massive outages due to quadratic algorithms in feeds and search. Complexity analysis transforms algorithm choice from guesswork into engineering.

### The O(n log n) Lower Bound

For **comparison-based sorting** (any algorithm that orders elements solely by comparing pairs), it is mathematically proven that no algorithm can do better than O(n log n) in the worst case.

**Why?** A sorting algorithm on n elements must distinguish between n! possible orderings. A comparison has two outcomes (less-than or greater-than). A decision tree of binary choices needs at least log₂(n!) leaves to cover all orderings.

By Stirling's approximation: log₂(n!) ≈ n log₂ n

```
n=3 example — 3! = 6 possible orderings: [1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]

Decision tree (each node is one comparison):
          a < b?
         /      \
      YES        NO
     b < c?    a < c?
    /    \     /    \
 [a,b,c] a<c? [a,c,b] [c,a,b]
         / \
      [b,a,c] [b,c,a]

Depth of tree = ceil(log₂(6)) = 3 comparisons minimum
```

This means Merge Sort, Heap Sort, and Quick Sort (average case) are all **optimal** for comparison-based sorting — no comparison-based algorithm can beat them asymptotically.

---

## Key Terminology

Before diving in, here are terms used throughout this page:

| Term | Meaning |
|------|---------|
| **In-place** | Sorts without requiring extra arrays — O(1) auxiliary space beyond the input |
| **Stable** | Preserves the original relative order of equal elements |
| **Divide & Conquer** | Splits the problem into subproblems, solves each, combines results |
| **Pivot** | A chosen element in Quick Sort used to partition the array |
| **Heapify** | Restoring the heap property after a modification |

---

## Merge Sort

### Concept

Merge Sort is a **divide-and-conquer** algorithm invented by John von Neumann in 1945. It is the gold standard for stability and predictable performance.

The strategy:
1. **Divide** — split the array in half, down to single elements
2. **Conquer** — single elements are trivially sorted (base case)
3. **Merge** — combine two sorted halves into one sorted array

The key insight: merging two sorted arrays is O(n) — just walk a pointer through each and pick the smaller element each time.

### Divide-and-Conquer Diagram

```
Input: [38, 27, 43, 3, 9, 82, 10]

                  DIVIDE PHASE
         ┌────────────────────────────┐
         │    [38, 27, 43, 3, 9, 82, 10]    ← split
         └───────────┬────────────────┘
                    / \
        [38, 27, 43, 3]    [9, 82, 10]       ← split again
              / \              / \
        [38, 27]  [43, 3]  [9, 82]  [10]     ← split again
          / \      / \      / \
        [38] [27] [43] [3] [9] [82]          ← base case: size 1

                  CONQUER (merge) PHASE
        [38] [27] → [27, 38]
        [43] [3]  → [3, 43]
        [9] [82]  → [9, 82]

        [27,38] + [3,43] → [3, 27, 38, 43]
        [9,82]  + [10]   → [9, 10, 82]

        [3,27,38,43] + [9,10,82] → [3, 9, 10, 27, 38, 43, 82]
```

### The Merge Step Illustrated

The merge function takes two sorted sub-arrays and produces one sorted array. Walk through `merge([27, 38], [3, 43])`:

```
Left:  [27, 38]    i=0
Right: [3,  43]    j=0
Result: []

Step 1: left[0]=27 vs right[0]=3  → 3 is smaller  → result=[3],   j=1
Step 2: left[0]=27 vs right[1]=43 → 27 is smaller → result=[3,27], i=1
Step 3: left[1]=38 vs right[1]=43 → 38 is smaller → result=[3,27,38], i=2
Step 4: left exhausted              → append rest of right
        result = [3, 27, 38, 43]
```

Each element is placed exactly once → merge is **O(n)**.

### Recurrence Relation

```
T(n) = 2T(n/2) + O(n)
         ↑         ↑
   2 recursive   merge step
   calls, each   costs O(n)
   on n/2 items
```

Applying the Master Theorem: a=2, b=2, f(n)=n.
- Critical exponent: log₂(2) = 1 → n¹ = n
- f(n) = n = Θ(n¹) → Case 2

**Result: T(n) = Θ(n log n) always** — no worst case degradation.

Drawing the recursion tree confirms this:

```
Level 0:  n elements, 1 merge → n work
Level 1:  n/2 + n/2 elements, 2 merges → n work
Level 2:  n/4 × 4, 4 merges → n work
...
Level log n: 1 × n, n merges → n work

Total levels: log n
Work per level: n
Total: n log n
```

### Why Merge Sort Is Stable

When merging, ties are broken by choosing the **left** element first. So equal elements from the left sub-array always appear before equal elements from the right — original relative order is preserved.

```
Input: [(A,3), (B,3), (C,1)]   ← sort by second value

After merge sort: [(C,1), (A,3), (B,3)]
                                  ↑    ↑
                        (A,3) before (B,3) — original order kept
```

### When to Use Merge Sort

- You **need guaranteed** O(n log n) — no worst-case risk
- You need a **stable** sort (preserving order of equal elements)
- Sorting **linked lists** — merge sort is the natural fit (no random access needed)
- Sorting data **larger than RAM** (external sort) — merge chunks from disk
- Pipelines with strict **SLA-bound latency** requirements

**Trade-off**: Requires O(n) auxiliary space for the temporary arrays during merge.

---

## Quick Sort

### Concept

Quick Sort, invented by Tony Hoare in 1961, is the fastest practical sorting algorithm on average. It is **in-place** (O(log n) stack only) and cache-friendly because it accesses memory sequentially.

The strategy:
1. **Choose a pivot** — pick an element from the array
2. **Partition** — rearrange so everything less than pivot is on its left, everything greater is on its right. The pivot is now in its final sorted position.
3. **Recurse** — apply the same process to the left and right sub-arrays

### Partition Step Illustrated Step-by-Step

We'll use Lomuto partition scheme with the last element as pivot.

**Input: `[8, 3, 1, 5, 2, 9, 4]`, pivot = 4 (last element)**

```
arr = [8, 3, 1, 5, 2, 9, 4]
pivot = 4
i = -1   ← boundary of "less-than" region
j starts at 0, scans right

j=0: arr[j]=8, 8 > 4 → do nothing
     [8, 3, 1, 5, 2, 9, 4]   i=-1

j=1: arr[j]=3, 3 < 4 → i++, swap arr[i] and arr[j]
     i becomes 0, swap arr[0]=8 and arr[1]=3
     [3, 8, 1, 5, 2, 9, 4]   i=0

j=2: arr[j]=1, 1 < 4 → i++, swap arr[i] and arr[j]
     i becomes 1, swap arr[1]=8 and arr[2]=1
     [3, 1, 8, 5, 2, 9, 4]   i=1

j=3: arr[j]=5, 5 > 4 → do nothing
     [3, 1, 8, 5, 2, 9, 4]   i=1

j=4: arr[j]=2, 2 < 4 → i++, swap arr[i] and arr[j]
     i becomes 2, swap arr[2]=8 and arr[4]=2
     [3, 1, 2, 5, 8, 9, 4]   i=2

j=5: arr[j]=9, 9 > 4 → do nothing
     [3, 1, 2, 5, 8, 9, 4]   i=2

End: swap arr[i+1] and pivot (arr[6])
     swap arr[3]=5 and arr[6]=4
     [3, 1, 2, 4, 8, 9, 5]   pivot 4 now at index 3
                 ↑
         PIVOT IN FINAL POSITION

Left partition:  [3, 1, 2]   (all < 4)
Right partition: [8, 9, 5]   (all > 4)
```

Recurse on each partition independently.

### Pivot Selection Strategy

The choice of pivot is critical. A bad pivot creates unbalanced partitions.

**Fixed first/last element** (naive):
```
Input: [1, 2, 3, 4, 5]   pivot = 5 (last)

After partition: [1, 2, 3, 4] | [5]   ← 4 elements left, 0 right
After partition: [1, 2, 3] | [4]      ← 3 elements left, 0 right
...
Depth: n levels → O(n²) worst case!
```

**Median-of-three** (common improvement):
```
Pick first, middle, and last elements. Use their median as pivot.
arr = [8, 3, 1, 5, 2, 9, 4]
first=8, middle=5, last=4  →  median=5
Pivot = 5 (more balanced partition)
```

**Randomized pivot** (production-grade):
```python
import random
pivot_idx = random.randint(lo, hi)
arr[pivot_idx], arr[hi] = arr[hi], arr[pivot_idx]
# then run standard partition
```

Randomization eliminates adversarial inputs — no one can craft an input to force O(n²) behavior when the pivot is random.

### Best, Average, and Worst Cases

**Best case** — pivot always lands at the exact middle (perfectly balanced splits):
```
T(n) = 2T(n/2) + O(n)   ← same as merge sort
Result: O(n log n)
```

**Average case** — pivot lands at a random position (expected behavior with random input or randomized pivot):
```
Even a 90/10 split gives:
T(n) = T(0.9n) + T(0.1n) + O(n)

This still resolves to O(n log n) — any constant-ratio split gives n log n.
Randomized pivot achieves O(n log n) in expectation.
```

**Worst case** — pivot is always the smallest or largest element (sorted/reverse-sorted input with fixed pivot):
```
T(n) = T(n-1) + T(0) + O(n)
     = T(n-1) + O(n)
     = O(n²)

Partition depths:
n → n-1 → n-2 → ... → 1   (n levels, O(n) work each)
```

### Tail-Call Optimization

In standard recursion, Quick Sort makes two recursive calls. With **tail-call optimization**, always recurse on the smaller partition first, then loop for the larger:

```python
def quicksort(arr, lo, hi):
    while lo < hi:
        pivot = partition(arr, lo, hi)
        if (pivot - lo) < (hi - pivot):
            quicksort(arr, lo, pivot - 1)    # recurse on smaller
            lo = pivot + 1                   # loop on larger
        else:
            quicksort(arr, pivot + 1, hi)    # recurse on smaller
            hi = pivot - 1                   # loop on larger
```

This guarantees O(log n) stack depth even in bad cases — the larger partition is handled iteratively.

### In-Place Nature

Quick Sort rearranges elements within the original array. The only extra memory needed is the recursion stack: O(log n) average, O(n) worst case (fixed by tail-call optimization above).

This makes Quick Sort cache-friendly: it accesses the same contiguous block of memory throughout, maximizing CPU cache hit rates.

---

## Heap Sort

### Binary Heap Structure

A **binary max-heap** is a complete binary tree where every parent is greater than or equal to its children. It is stored in an array — no explicit tree nodes needed.

For a node at index `i` (0-indexed):
```
Parent:       (i - 1) // 2
Left child:   2*i + 1
Right child:  2*i + 2
```

**Example: Heap for `[82, 38, 43, 27, 9, 10, 3]`**

```
Array index:  0   1   2   3   4   5   6
Array value: [82, 38, 43, 27, 9, 10,  3]

As a tree:
             82        ← index 0
           /    \
         38      43    ← index 1, 2
        /  \    /  \
       27   9  10   3  ← index 3, 4, 5, 6

Max-heap property: every parent > children ✓
```

### Max-Heapify (Sifting Down)

Heapify restores the heap property at a node that may violate it, assuming both children already satisfy the heap property.

**Heapify at index 1 in `[82, 3, 43, 27, 9, 10, 38]`** (3 is out of place):

```
Before:
             82
           /    \
          3      43       ← 3 < 38 (right child of 43's subtree), violated
        /  \    /  \
       27   9  10  38

Step 1: Compare 3 with its children (27, 9). Largest child = 27 at index 3.
        3 < 27 → swap arr[1] and arr[3]

             82
           /    \
          27     43
        /  \    /  \
        3   9  10  38

Step 2: Now 3 is at index 3. Compare with its children (none — it's a leaf).
        Done.
```

Heapify moves a value *down* until the heap property is satisfied. Cost: O(log n) — at most tree height steps.

### Building a Heap (Heapify-All)

To turn an arbitrary array into a max-heap, call heapify on every non-leaf node from bottom to top:

```python
def build_heap(arr):
    n = len(arr)
    for i in range(n // 2 - 1, -1, -1):   # from last non-leaf up to root
        heapify_down(arr, n, i)
```

**Building heap from `[3, 27, 43, 9, 82, 10, 38]`:**

```
Initial:  [3, 27, 43, 9, 82, 10, 38]

Non-leaf nodes: index 0,1,2  (n=7, n//2-1 = 2)

heapify(arr, 7, i=2): node=43, children=10,38 → 43>38>10, no swap
heapify(arr, 7, i=1): node=27, children=9,82 → 82>27, swap 27↔82
  → [3, 82, 43, 9, 27, 10, 38]
  → now at index 4: node=27, no children in range, done
heapify(arr, 7, i=0): node=3, children=82,43 → 82>3, swap 3↔82
  → [82, 3, 43, 9, 27, 10, 38]
  → now at index 1: node=3, children=9,27 → 27>3, swap 3↔27
  → [82, 27, 43, 9, 3, 10, 38]
  → now at index 4: leaf, done

Final heap: [82, 27, 43, 9, 3, 10, 38]
```

Building a heap costs **O(n)** total (not O(n log n) — the heapify calls at lower levels are cheaper).

### Extract-Max and Heap Sort

To extract the maximum:
1. The root (index 0) holds the maximum.
2. Move the last element to the root, shrink heap size by 1.
3. Heapify-down from the root to restore the heap property.

**Full Heap Sort on `[3, 1, 4, 1, 5, 9, 2, 6]`:**

```
Step 1: Build max-heap
  → [9, 6, 4, 3, 5, 1, 2, 1]
     Heap: 9 at root

Step 2: Repeated extract-max
  Extract 9: swap arr[0]=9 with arr[7]=1, heap size=7
  → [1, 6, 4, 3, 5, 1, 2, | 9]    (9 sorted at end)
  Heapify root: 1→6→5 sifts down
  → [6, 5, 4, 3, 1, 1, 2, | 9]

  Extract 6: swap arr[0]=6 with arr[6]=2, heap size=6
  → [2, 5, 4, 3, 1, 1, | 6, 9]    (6 sorted)
  Heapify root: 2→5 sifts down
  → [5, 3, 4, 2, 1, 1, | 6, 9]

  Extract 5: swap arr[0]=5 with arr[5]=1, heap size=5
  → [1, 3, 4, 2, 1, | 5, 6, 9]    (5 sorted)
  Heapify root: 1→4 sifts down
  → [4, 3, 1, 2, 1, | 5, 6, 9]

  ... continue until heap size = 0

  Final sorted array: [1, 1, 2, 3, 4, 5, 6, 9]
```

### Complexity Analysis

- **Build heap**: O(n)
- **n extract-max operations**: each costs O(log n) → total O(n log n)
- **Total**: O(n log n) — guaranteed, no worst case

**Why not O(n) total?** Each extract-max shrinks the heap by 1 and calls heapify which costs O(log (remaining size)). The total is:

```
log(n) + log(n-1) + log(n-2) + ... + log(1)
= log(n!)
≈ n log n   (by Stirling's approximation)
```

### Heap Sort Properties

- **In-place**: Only O(1) auxiliary space (sorted in the original array)
- **Not stable**: Extract-max can skip over equal elements and change their relative order
- **Guaranteed O(n log n)**: No pivot selection issues, no worst case
- **Cache performance**: Heap's non-sequential access pattern causes more cache misses than Quick Sort — so Quick Sort is often faster in practice despite identical asymptotic complexity

---

## Algorithm Comparison Table

| Algorithm | Best Time | Avg Time | Worst Time | Space | Stable | In-place | Best For |
|-----------|-----------|----------|------------|-------|--------|----------|----------|
| Merge Sort | Θ(n log n) | Θ(n log n) | Θ(n log n) | O(n) | Yes | No | Large data, external sort, linked lists, SLA-bound systems |
| Quick Sort | Ω(n log n) | Θ(n log n) | O(n²) | O(log n) | No | Yes | General-purpose arrays, cache-sensitive systems |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No | Yes | When O(1) space required and stability not needed |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes | No | Small integer range, known bounds |
| Radix Sort | O(d(n+k)) | O(d(n+k)) | O(d(n+k)) | O(n+k) | Yes | No | Multi-digit integers, fixed-length strings |

Notes on the table:
- k = range of input values (for Counting/Radix Sort)
- d = number of digits (for Radix Sort)
- Quick Sort space is O(log n) stack with tail-call optimization; O(n) without it

---

## Counting Sort (Non-Comparison Based)

### When Standard Sorts Are Overkill

When your input values are **bounded integers** in a known range [0, k], you can sort without any comparisons at all. Counting Sort exploits this to run in **O(n + k)** time.

If k is small (e.g., sorting exam scores 0–100, or sorting characters in a string), Counting Sort demolishes comparison-based sorts.

### How It Works

The idea: count how many times each value appears, then reconstruct the sorted array from those counts.

**Three phases:**
1. **Count**: Tally how many times each value appears in a frequency array.
2. **Prefix sum**: Convert frequency counts into starting positions.
3. **Place**: Place each element into its correct position in the output.

### Worked Example

**Input: `[4, 2, 2, 8, 3, 3, 1]`, k=8 (values range 1–8)**

**Phase 1: Build frequency array `count[0..8]`**

```
Input: [4, 2, 2, 8, 3, 3, 1]

Scan left to right, increment count[value] for each element:

count: [0, 0, 0, 0, 0, 0, 0, 0, 0]   ← initial (indices 0..8)
         ↑ after 4:
count: [0, 0, 0, 0, 1, 0, 0, 0, 0]
         ↑ after 2:
count: [0, 0, 1, 0, 1, 0, 0, 0, 0]
         ↑ after 2:
count: [0, 0, 2, 0, 1, 0, 0, 0, 0]
         ↑ after 8:
count: [0, 0, 2, 0, 1, 0, 0, 0, 1]
         ↑ after 3:
count: [0, 0, 2, 1, 1, 0, 0, 0, 1]
         ↑ after 3:
count: [0, 0, 2, 2, 1, 0, 0, 0, 1]
         ↑ after 1:
count: [0, 1, 2, 2, 1, 0, 0, 0, 1]

Final count:
Index: 0  1  2  3  4  5  6  7  8
Value: 0  1  2  2  1  0  0  0  1
```

**Phase 2: Prefix sum to find starting positions**

```
count[i] += count[i-1]   for i = 1 to k

count[0] = 0
count[1] = 0+1 = 1
count[2] = 1+2 = 3
count[3] = 3+2 = 5
count[4] = 5+1 = 6
count[5] = 6+0 = 6
count[6] = 6+0 = 6
count[7] = 6+0 = 6
count[8] = 6+1 = 7

count now holds: for value v, count[v] is the index AFTER the last v in output
```

**Phase 3: Place elements (scan input right to left for stability)**

```
output = [_, _, _, _, _, _, _]   (7 slots)

Process input right to left: [1, 3, 3, 8, 2, 2, 4]  (reversed)

value=1: count[1]=1, place at output[1-1]=output[0], count[1]-- → count[1]=0
  output = [1, _, _, _, _, _, _]

value=3: count[3]=5, place at output[5-1]=output[4], count[3]-- → count[3]=4
  output = [1, _, _, _, 3, _, _]

value=3: count[3]=4, place at output[4-1]=output[3], count[3]-- → count[3]=3
  output = [1, _, _, 3, 3, _, _]

value=8: count[8]=7, place at output[7-1]=output[6], count[8]-- → count[8]=6
  output = [1, _, _, 3, 3, _, 8]

value=2: count[2]=3, place at output[3-1]=output[2], count[2]-- → count[2]=2
  output = [1, _, 2, 3, 3, _, 8]

value=2: count[2]=2, place at output[2-1]=output[1], count[2]-- → count[2]=1
  output = [1, 2, 2, 3, 3, _, 8]

value=4: count[4]=6, place at output[6-1]=output[5], count[4]-- → count[4]=5
  output = [1, 2, 2, 3, 3, 4, 8]

Sorted: [1, 2, 2, 3, 3, 4, 8]  ✓
```

### Complexity

- **Time**: O(n + k) — O(k) to build frequency array, O(n) to place elements
- **Space**: O(n + k) — output array of size n, count array of size k
- **Stable**: Yes (right-to-left placement preserves relative order of equals)

**When is O(n + k) actually good?**

| k value | Effective complexity | Verdict |
|---------|---------------------|---------|
| k = O(n) | O(n + n) = O(n) | Excellent — beats comparison sorts |
| k = O(n log n) | O(n + n log n) = O(n log n) | Break-even |
| k = O(n²) | O(n + n²) = O(n²) | Worse than comparison sorts |

Counting Sort wins when k is small relative to n — for example, sorting student grades (0–100), ASCII characters (0–127), or integers in a narrow range.

---

## Radix Sort

### The Idea: Sort Digit by Digit

Radix Sort extends Counting Sort to handle multi-digit numbers by sorting **one digit position at a time**, from least significant digit (LSD) to most significant digit (MSD).

The key insight: if you sort by a digit using a *stable* sort, and you repeat from least to most significant digit, the final result is correctly sorted.

### Why Stability is Non-Negotiable

Consider sorting `[329, 457, 657, 839, 436, 720, 355]`:

After sorting by the 1s digit (stably):
```
720 (digit=0), 355 (digit=5), 436 (digit=6), 457 (digit=7), 657 (digit=7), 329 (digit=9), 839 (digit=9)
→ [720, 355, 436, 457, 657, 329, 839]

Note: 457 appears before 657 (both end in 7) — their relative order from original is preserved.
```

After sorting by the 10s digit (stably), *preserving existing order of ties*:
```
720 (tens=2), 329 (tens=2), 436 (tens=3), 839 (tens=3), 355 (tens=5), 457 (tens=5), 657 (tens=5)
→ [720, 329, 436, 839, 355, 457, 657]
```

After sorting by the 100s digit (stably):
```
329 (hundreds=3), 355 (hundreds=3), 436 (hundreds=4), 457 (hundreds=4), 657 (hundreds=6), 720 (hundreds=7), 839 (hundreds=8)
→ [329, 355, 436, 457, 657, 720, 839]   ✓ Sorted!
```

If any single pass were **unstable**, earlier passes' work would be undone.

### Full Worked Example

**Input: `[170, 45, 75, 90, 802, 24, 2, 66]`**

**Pass 1 — Sort by 1s digit:**
```
Digits:  170→0, 45→5, 75→5, 90→0, 802→2, 24→4, 2→2, 66→6

Buckets (0-9):
  0: [170, 90]
  2: [802, 2]
  4: [24]
  5: [45, 75]
  6: [66]

After pass 1: [170, 90, 802, 2, 24, 45, 75, 66]
```

**Pass 2 — Sort by 10s digit:**
```
Digits: 170→7, 90→9, 802→0, 2→0, 24→2, 45→4, 75→7, 66→6

Buckets:
  0: [802, 2]
  2: [24]
  4: [45]
  6: [66]
  7: [170, 75]
  9: [90]

After pass 2: [802, 2, 24, 45, 66, 170, 75, 90]
```

**Pass 3 — Sort by 100s digit:**
```
Digits: 802→8, 2→0, 24→0, 45→0, 66→0, 170→1, 75→0, 90→0

Buckets:
  0: [2, 24, 45, 66, 75, 90]
  1: [170]
  8: [802]

After pass 3: [2, 24, 45, 66, 75, 90, 170, 802]   ✓ Sorted!
```

### Complexity Analysis

- `d` = number of digits (or digit positions)
- `k` = range of each digit (10 for decimal, 2 for binary, 256 for bytes)
- `n` = number of elements

Each pass uses Counting Sort on n elements with range k → O(n + k) per pass.
Total passes = d.

**Total time: O(d(n + k))**

For 32-bit integers sorted in base-256 (4 bytes = 4 passes, k=256):
```
d=4, k=256, n elements
O(4(n + 256)) = O(4n + 1024) = O(n)
```

For large n (say n = 1,000,000), this is effectively **O(n)** — faster than any comparison sort.

### When to Use Radix Sort

| Condition | Radix Sort? | Reason |
|-----------|-------------|--------|
| n large, d small, k small | Yes | O(n) beats O(n log n) |
| Fixed-length strings (e.g., zip codes) | Yes | Same analysis as integers |
| Variable-length strings | Careful | Pad to max length or use MSD variant |
| Floating-point numbers | Complicated | Needs sign/exponent handling |
| General objects (no digit structure) | No | Use comparison sort |

---

## Sorting Algorithm Selection Guide

Use this decision flowchart when choosing a sort:

```
START
  │
  ├─ Input is bounded integers or fixed-width data?
  │     YES → k small relative to n?
  │              YES → Use COUNTING SORT (O(n+k), stable, simple)
  │              Multi-digit → Use RADIX SORT (O(d(n+k)), stable)
  │     NO  → (continue with comparison sorts)
  │
  ├─ Need guaranteed worst-case O(n log n)?
  │     YES → Need stability?
  │              YES → MERGE SORT (predictable, stable, O(n) space)
  │              NO  → HEAP SORT (O(1) space, in-place, not stable)
  │     NO  → (Quick Sort family)
  │
  ├─ General purpose, in-place preferred?
  │     Use RANDOMIZED QUICK SORT (fastest avg case, O(n log n) expected)
  │
  └─ Real-world data with partial order / nearly-sorted input?
        Use TIMSORT (Python's built-in — hybrid Merge+Insertion, O(n) best case)
```

### By Use Case

| Situation | Recommended sort | Reason |
|-----------|----------------|--------|
| Large random arrays | Randomized Quick Sort | Best cache behavior, O(n log n) avg |
| External file sort (data > RAM) | Merge Sort | Sequential access, merge chunks |
| Need stable sort of objects | Merge Sort | Guaranteed stability |
| Embedded system, O(1) space | Heap Sort | No extra arrays needed |
| Integers, narrow range | Counting Sort | O(n+k) beats all comparison sorts |
| Multi-key records, fixed width | Radix Sort | O(n) effective time |
| Nearly-sorted data | Timsort / Insertion Sort | Adaptive, O(n) on already-sorted data |
| Small arrays (n < 16) | Insertion Sort | Low overhead beats O(n log n) for tiny n |
| SLA-bound production system | Merge Sort | No variance in worst case |

### Scalability by System Size

| Scale | n range | Recommendation |
|-------|---------|---------------|
| Small | n < 10,000 | Any advanced sort works; prefer readability |
| Medium | 10K – 1M | Randomized Quick Sort (in-memory hybrid) |
| Enterprise | 1M – Billions | External Merge Sort + distributed frameworks (Spark sort, MapReduce, Hadoop) |

---

## Real-World Examples from the Source

**High-Frequency Trading (Finance)**: Sort millions of market orders by price/time in microseconds. Solution: Optimized Quick Sort with 3-way partitioning and custom comparators. Randomization prevents worst-case attacks on pivot selection, reducing slippage and increasing throughput. (source: `2 - Advanced-Sorting-Algorithms-in-Complexity-Analysis.pdf`)

**Search Engine Indexing (Technology)**: External sorting of billions of web documents. Solution: Multi-way Merge Sort on distributed systems via MapReduce/Spark. Disk I/O dominates — optimizing for sequential access is critical for index building and updates.

**E-commerce Recommendations (Retail)**: Real-time sorting of scored products per user session. Solution: Hybrid Timsort-like algorithm with early termination for top-k results. Adaptive behavior on nearly-sorted data saves significant compute under heavy load.

---

## Common Mistakes by Experience Level

| Level | Mistake | Fix |
|-------|---------|-----|
| Beginner | Using unstable sort when relative order of equal elements matters | Use Merge Sort or Timsort instead |
| Beginner | Using Bubble Sort / Selection Sort on n > 10,000 | Switch to O(n log n) sort |
| Intermediate | Forgetting to handle duplicate elements — 3-way partition in Quick Sort | Use 3-way Lomuto/Hoare partition |
| Intermediate | Pure recursive Quick Sort on large inputs hitting Python's recursion limit | Set sys.setrecursionlimit or use iterative + tail-call optimization |
| Advanced | Ignoring hardware cache effects — Heap Sort vs Quick Sort in practice | Profile on real hardware; Quick Sort's cache locality usually wins |
| Production | Not re-validating sort choice after data volume growth | Re-analyze when n grows 10× |

---

## Quick Reference Cheat Sheet

```
Need stability?          → Merge Sort     Θ(n log n), O(n) space
In-place, fast average?  → Quick Sort     Θ(n log n) avg, O(log n) space
Guaranteed, in-place?    → Heap Sort      Θ(n log n), O(1) space
Bounded integers?        → Counting Sort  O(n + k) time and space
Multi-digit fixed width? → Radix Sort     O(d(n + k)) time

Recurrences:
  Merge Sort: T(n) = 2T(n/2) + O(n)  →  O(n log n) by Master Theorem Case 2
  Quick Sort: T(n) = 2T(n/2) + O(n)  →  O(n log n) average (T(n)=T(n-1)+O(n) worst)

Lower bound: No comparison-based sort can beat O(n log n) worst case.
```

Related pages: [[adsa-complexity]], [[adsa-arrays]], [[adsa-stacks]]

#adsa
