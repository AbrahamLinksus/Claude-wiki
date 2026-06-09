# Double-Ended Queue (Deque)

**Summary**: A data structure supporting O(1) push and pop at both ends. Several implementations exist with different trade-offs on random access, cache locality, and reallocation cost.

**Pre-Req**: [[adsa-queues]], [[adsa-arrays]], [[adsa-circular-ll]]

**Sources**: Session-built (2026-06-05). Updated with `raw/1 - Deque-Double-Ended-Queue.pdf`.

**Last updated**: 2026-06-09

---

## Deque ADT

A deque ("deck") is a generalisation of both stack and queue:

| Operation | Meaning |
|---|---|
| `push_front(x)` | Add x to the front |
| `push_back(x)` | Add x to the back |
| `pop_front()` | Remove and return front element |
| `pop_back()` | Remove and return back element |
| `peek_front()` / `peek_back()` | Read without removing |

All four push/pop operations must be O(1) for a deque to be useful. A plain dynamic array only gives O(1) at the back — `push_front` on a dynamic array is O(n) (shifts every element right).

---

## Implementation 1 — Doubly Linked List

Use a DLL with `head` and `tail` pointers.

```
head                              tail
 ↓                                 ↓
[←|10|→] ↔ [←|20|→] ↔ [←|30|→] ↔ [←|40|→]
```

- `push_front` / `push_back`: allocate node, wire up `prev`/`next`. O(1).
- `pop_front` / `pop_back`: unlink node, free. O(1).

**Problem**: cache-miss penalty on every operation — each node is at a random heap address (see [[adsa-queues]] — cache-miss analysis). For large deques this dominates real-world performance.

---

## Implementation 2 — Circular Buffer (Ring Buffer)

A fixed-capacity array with two indices, `head` and `tail`, that wrap around using modular arithmetic.

```
capacity = 8
head = 2, tail = 6

index:  0    1    2    3    4    5    6    7
      [ -- | -- | 10 | 20 | 30 | 40 | -- | -- ]
                  ↑ head              ↑ tail
```

- `push_back`: write at `arr[tail]`, `tail = (tail + 1) % capacity`. O(1).
- `push_front`: `head = (head - 1 + capacity) % capacity`, write at `arr[head]`. O(1).
- `pop_front`: read `arr[head]`, `head = (head + 1) % capacity`. O(1).
- `pop_back`: `tail = (tail - 1 + capacity) % capacity`, read `arr[tail]`. O(1).

**Problem**: fixed capacity. When full, must reallocate — allocate 2× buffer, copy in order, reset `head = 0`. The reallocation copies the entire data block: O(n).

**This is the circular linked list [[adsa-circular-ll]] concept translated to array indices** — instead of `next` pointers wrapping around, array indices wrap around via `% capacity`.

---

## Implementation 3 — Unrolled Linked List (Node-with-Array)

**The idea**: instead of each LL node holding one element, each node holds a small fixed-size array (chunk size `k`).

```
k = 4

head
 ↓
Node A: [ 10 | 20 | 30 | 40 ] → Node B: [ 50 | 60 | -- | -- ] → NULL
         └── contiguous ──┘              └── contiguous ──┘
```

**Benefits**:
- **No full reallocation**: when a node fills up, just allocate a new node with a new `k`-element array. No global copy.
- **Better cache locality than pure LL**: iterating through a node's array is sequential — k elements load in the same cache lines. The cache miss only occurs at each node boundary.

**Drawback — pointer chasing on index access**:

Even if you know the index `i`, to access it you must:
1. Compute which node: `node_index = i / k`
2. Compute offset within that node: `offset = i % k`
3. **Traverse `node_index` nodes** via pointer chasing to reach the right node

There is no way to jump directly to node number `node_index` — they are scattered in heap memory just like regular LL nodes. Random access is O(n/k), not O(1).

> The improvement over pure LL is that *sequential* access has k-fold fewer pointer hops — but *indexed* access is still bottlenecked by pointer traversal.

---

## Implementation 4 — Chunked Array (Dynamic Array of Fixed Chunks)

**The idea**: maintain a dynamic array (the "index") where each slot holds a **pointer** to a fixed-size chunk of `k` elements.

```
k = 4

index (dynamic array of pointers):
[ ptr0 | ptr1 | ptr2 | ... ]
    ↓       ↓       ↓
 Chunk0  Chunk1  Chunk2
[10|20|  [50|60|  [90|--|
 30|40]   70|80]   --|--]
```

**Random access is O(1)**:

```
element at index i:
    chunk_index = i / k        → index into the pointer array  (O(1))
    offset      = i % k        → index within the chunk         (O(1))
    return index[chunk_index][offset]
```

Two array indexing operations, no pointer traversal. True O(1) for any `i`.

**Append (push_back)**:
1. Write to the last chunk if it has space.
2. If the last chunk is full: allocate a new chunk of size `k`, append its pointer to the index array.
3. The index array itself grows via doubling — but it only stores **pointers**, not data. Copying the index array costs O(n/k), not O(n). The actual data chunks are never moved.

**Push_front**:
Maintain a `front_offset` into the first chunk so you can fill it backwards. When the first chunk is full, prepend a new chunk pointer to the index array front.

---

## Comparison: All Four Implementations

| | DLL | Circular Buffer | Unrolled LL | Chunked Array |
|---|---|---|---|---|
| Push / pop both ends | O(1) | O(1) | O(1) | Amortized O(1) |
| Random access `arr[i]` | O(n) | O(1) | O(n/k) | **O(1)** |
| Reallocation on grow | No | O(n) full copy | No (add node) | O(n/k) pointer copy |
| Cache on sequential scan | Poor (1 miss/node) | Excellent | k-fold better than DLL | Excellent per chunk |
| Memory overhead | 2 pointers/element | Wasted slots | 2 pointers/k elements | 1 pointer/k elements |

---

## Why Chunked Array Wins for General Use

The unrolled LL avoids all reallocation (no central index to copy) but pays O(n/k) for random access.
The chunked array copies the index on growth (O(n/k) pointers — small), and gets O(1) random access in return.

For most workloads, O(1) random access + cache-friendly chunks + cheap index reallocation makes the chunked array the practical choice. This is essentially how `std::deque` is implemented in C++.

---

## When to Use Each

| Use case | Best choice |
|---|---|
| Need O(1) push/pop both ends, no random access, low overhead | Circular buffer (fixed capacity known) or DLL |
| Need O(1) push/pop + O(1) random access at scale | Chunked array |
| Want zero-copy growth with decent sequential performance | Unrolled LL |
| Embedded / fixed-memory environments | Circular buffer |

---

## Sliding Window Maximum

**Problem**: Given an array of n integers and a window size k, find the maximum value in every contiguous window of size k. Output has n-k+1 values.

**Naive approach**: for each of the n-k+1 windows, scan all k elements to find the max. Time: **O(n×k)**. For n=10^6, k=1000: 10^9 operations — too slow.

**Deque approach**: maintain a monotonic deque of indices where:
- The front always holds the index of the current window's maximum
- The back holds indices of elements that are still "candidates" for future windows
- Any index whose value is ≤ the new element being added is useless (the new element will always beat it), so it gets popped from the back

```
FUNCTION slidingWindowMax(nums, k):
    result = []
    deque = empty deque   // stores INDICES, not values

    FOR i = 0 TO len(nums)-1:

        // Step 1: Remove indices that have fallen outside the window
        WHILE deque not empty AND deque.peek_front() <= i - k:
            deque.pop_front()

        // Step 2: Remove indices whose values are <= nums[i]
        // (they can never be the max for any future window)
        WHILE deque not empty AND nums[deque.peek_back()] <= nums[i]:
            deque.pop_back()

        // Step 3: Add current index
        deque.push_back(i)

        // Step 4: Record max once window is fully formed
        IF i >= k - 1:
            result.append(nums[deque.peek_front()])

    RETURN result
```

### Full Dry Run — 8-Element Array

```
Input: nums = [1, 3, -1, -3, 5, 3, 6, 7],  k = 3
Expected output: [3, 3, 5, 5, 6, 7]
```

Step-by-step (deque stores indices; showing values in brackets for clarity):

```
i=0, nums[0]=1
  Step 1: deque empty, nothing to remove
  Step 2: deque empty, nothing to remove
  Step 3: push 0 → deque=[0]  (values: [1])
  Step 4: i=0 < k-1=2, no output yet

i=1, nums[1]=3
  Step 1: front=0, 0 > 1-3=-2, keep
  Step 2: nums[0]=1 <= nums[1]=3 → pop 0
          deque empty, stop
  Step 3: push 1 → deque=[1]  (values: [3])
  Step 4: i=1 < 2, no output

i=2, nums[2]=-1
  Step 1: front=1, 1 > 2-3=-1, keep
  Step 2: nums[1]=3 > nums[2]=-1, stop
  Step 3: push 2 → deque=[1,2]  (values: [3,-1])
  Step 4: i=2 >= 2 → output nums[front=1] = 3
  Output so far: [3]

i=3, nums[3]=-3
  Step 1: front=1, 1 > 3-3=0, keep
  Step 2: nums[2]=-1 > nums[3]=-3, stop
  Step 3: push 3 → deque=[1,2,3]  (values: [3,-1,-3])
  Step 4: output nums[front=1] = 3
  Output so far: [3, 3]

i=4, nums[4]=5
  Step 1: front=1, 1 <= 4-3=1 → pop 1
          front=2, 2 > 1, stop
  Step 2: nums[3]=-3 <= 5 → pop 3
          nums[2]=-1 <= 5 → pop 2
          deque empty, stop
  Step 3: push 4 → deque=[4]  (values: [5])
  Step 4: output nums[front=4] = 5
  Output so far: [3, 3, 5]

i=5, nums[5]=3
  Step 1: front=4, 4 > 5-3=2, keep
  Step 2: nums[4]=5 > 3, stop
  Step 3: push 5 → deque=[4,5]  (values: [5,3])
  Step 4: output nums[front=4] = 5
  Output so far: [3, 3, 5, 5]

i=6, nums[6]=6
  Step 1: front=4, 4 > 6-3=3, keep
  Step 2: nums[5]=3 <= 6 → pop 5
          nums[4]=5 <= 6 → pop 4
          deque empty, stop
  Step 3: push 6 → deque=[6]  (values: [6])
  Step 4: output nums[front=6] = 6
  Output so far: [3, 3, 5, 5, 6]

i=7, nums[7]=7
  Step 1: front=6, 6 > 7-3=4, keep
  Step 2: nums[6]=6 <= 7 → pop 6
          deque empty, stop
  Step 3: push 7 → deque=[7]  (values: [7])
  Step 4: output nums[front=7] = 7
  Output: [3, 3, 5, 5, 6, 7]  ← correct!
```

**Why it's O(n)**: each element is pushed into the deque exactly once and popped at most once. Total work across all iterations: O(n), regardless of k.

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force | O(n×k) | O(1) | Scan every window |
| Deque (monotonic) | **O(n)** | O(k) | Each element enters/exits once |
| Speedup at k=1000 | 1000× | — | For n=10^6 stream |

Key insight: **store indices, not values**. Indices let you check both window boundary (`index <= i-k`) and value comparisons in O(1) (source: `raw/1 - Deque-Double-Ended-Queue.pdf`).

---

## Palindrome Checking with Deque

A palindrome reads the same forward and backward. The deque provides an elegant O(n) check by loading all characters and comparing the front and back simultaneously.

```
FUNCTION isPalindrome(input_string):
    // Step 1: Clean the string (lowercase, alphanumeric only)
    cleaned = ""
    FOR each char c IN input_string:
        IF c is letter or digit:
            cleaned += lowercase(c)

    // Step 2: Load all characters into a deque
    dq = DynamicDeque()
    FOR each char c IN cleaned:
        dq.push_back(c)

    // Step 3: Compare front and back until at most 1 char remains
    WHILE dq.size() > 1:
        front_char = dq.pop_front()
        back_char  = dq.pop_back()
        IF front_char != back_char:
            RETURN false

    RETURN true
```

Trace on "racecar":

```
Load: deque = [r, a, c, e, c, a, r]

Iteration 1: pop r (front), pop r (back) → match → [a, c, e, c, a]
Iteration 2: pop a (front), pop a (back) → match → [c, e, c]
Iteration 3: pop c (front), pop c (back) → match → [e]
size = 1 → RETURN true  (palindrome)
```

Trace on "hello":

```
Load: deque = [h, e, l, l, o]

Iteration 1: pop h (front), pop o (back) → h != o → RETURN false
```

**Validation matrix** (source: `raw/1 - Deque-Double-Ended-Queue.pdf`):

| Input | Expected |
|---|---|
| "racecar" | true |
| "hello" | false |
| "123454321" | true |
| "A man a plan a canal Panama" | true (after cleaning) |
| "" (empty) | true |
| "ab" | false |

**Why a deque is the right tool here**: the O(1) access to both ends is exactly what "compare outermost characters" requires. Using a stack requires building a reversed copy separately (O(n) extra space). The deque does it in O(n) time, O(n) space for the deque itself — no reversal copy needed.

---

## Input-Restricted and Output-Restricted Deque Variants

The full deque permits insertion and deletion at both ends. Two restricted variants appear in algorithms literature and certain scheduling scenarios:

### Input-Restricted Deque

Insertion is permitted at **one end only** (typically the back). Deletion is permitted at both ends.

```
Allowed:   push_back(x)
           pop_front()
           pop_back()
NOT allowed: push_front(x)
```

Effectively: you have a queue (FIFO) that also allows undoing the most recent dequeue from the front by peeking/popping the back.

**Use case**: certain scheduling scenarios where new tasks always arrive at the back but can be cancelled from either end. The sliding window maximum algorithm uses this pattern — it only pushes to the back but pops from both.

### Output-Restricted Deque

Deletion is permitted at **one end only** (typically the front). Insertion is permitted at both ends.

```
Allowed:   push_front(x)
           push_back(x)
           pop_front()
NOT allowed: pop_back()
```

Effectively: a queue that allows items to "cut in line" by being pushed to the front.

**Use case**: priority insertion where most items are processed FIFO but urgent items can be prepended. Browser history navigation uses this — new pages push to back, urgent "back" navigations pop from back (actually pop_back to get the most recent page).

### Comparison of All Variants

| Variant | push_front | push_back | pop_front | pop_back |
|---|---|---|---|---|
| Full deque | Yes | Yes | Yes | Yes |
| Input-restricted | No | Yes | Yes | Yes |
| Output-restricted | Yes | Yes | Yes | No |
| Queue (FIFO) | No | Yes | Yes | No |
| Stack (LIFO) | N/A | Yes | No | Yes |

(source: `raw/1 - Deque-Double-Ended-Queue.pdf`)

---

## Deque vs Stack vs Queue — Full Comparison

| Criteria | Deque | Queue (FIFO) | Stack (LIFO) | Vector/ArrayList |
|---|---|---|---|---|
| push_front | O(1) | N/A | N/A | O(n) shift |
| push_back | O(1) | O(1) | O(1) | O(1) amortized |
| pop_front | O(1) | O(1) | N/A | O(n) shift |
| pop_back | O(1) | N/A | O(1) | O(1) |
| random access | O(1)* | N/A | N/A | O(1) |
| Flexibility | Both ends | One direction | One direction | Index only |
| Use when | Need both ends | FIFO ordering | LIFO/backtrack | Random access |

*Array-based deque only; linked-list deque has O(n) random access.

**Decision rule**: use a deque when you need O(1) operations at both ends. Use a plain queue or stack when only one end is needed — the deque's additional complexity is unnecessary overhead in those cases.

### When the Deque Uniquely Enables an Algorithm

| Problem | Without Deque | With Deque |
|---|---|---|
| Sliding window max | O(n×k) brute force | O(n) monotonic deque |
| Task stealing scheduler | Load imbalance, idle cores | O(1) steal from back, O(1) execute from front |
| Palindrome check | O(n) extra space for reversed copy | O(n) space, clean two-pointer pop |
| Bounded undo/redo history | Grow infinitely or shift entire array | pop_front evicts oldest, O(1) |

The most dramatic case is sliding window: without deques, a 10M-element stream with k=1000 requires **10 billion operations**. With a monotonic deque, it requires **10 million** — a 1000× speedup (source: `raw/1 - Deque-Double-Ended-Queue.pdf`).

#adsa
