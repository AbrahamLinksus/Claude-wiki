# Queues

**Summary**: Queue ADT via dynamic arrays and linked lists — FIFO principle, O(1) enqueue/dequeue, memory-efficiency problem with arrays, BFS graph traversal, level-order tree traversal, and round-robin scheduling.

**Pre-Req**: [[adsa-arrays]], [[adsa-stacks]]

**Sources**: Session-built (2026-06-04), `1 - Stack-and-Queue-Using-Arrays-and-Linked-Lists.pdf`, `3 - Queue-FIFO-Round-Robin-and-BFS.pdf`

**Last updated**: 2026-06-09

---

## FIFO Principle

A queue is **First In, First Out** — elements leave in the order they arrived. Think of a checkout line.

```
Enqueue →  [ 10 | 20 | 30 | 40 ]  → Dequeue
             ↑ head (front)   ↑ tail (rear)
```

Two operations:
- **enqueue(x)** — add to the rear
- **dequeue()** — remove from the front

---

## Queue ADT Contract

An **Abstract Data Type** defines the interface — what operations exist and what they guarantee — without specifying the implementation. The contract is the same whether the queue runs on an array or a linked list underneath.

| Operation | Description | Complexity |
|---|---|---|
| `enqueue(x)` | Add x to the rear | O(1) amortized |
| `dequeue()` | Remove and return front element | O(1) |
| `peek()` | Read front without removing | O(1) |
| `isEmpty()` | True if no elements | O(1) |
| `size()` | Number of elements | O(1) |

(source: `1 - Stack-and-Queue-Using-Arrays-and-Linked-Lists.pdf`)

---

## Queue using a Dynamic Array

Track two indices: `head` (front) and `tail` (rear).

```
head=0, tail=3
index:  0    1    2    3
      [ 10 | 20 | 30 | 40 ]
        ↑ head         ↑ tail
```

**Enqueue**: write at `arr[tail]`, increment `tail`. Amortized O(1).

**Dequeue**: read `arr[head]`, increment `head`. **O(1)** — no shifting needed, just move the index forward.

```
After dequeue:
head=1, tail=3
index:  0    1    2    3
      [ ?? | 20 | 30 | 40 ]
              ↑ head    ↑ tail
```

### Is it memory efficient?

**No.** Every dequeue advances `head` and leaves a dead slot at the front — permanently wasted. A queue with 3 live elements might be sitting inside a 10,000-slot array with 9,997 dead slots ahead of it.

The standard fix is a **circular buffer (ring buffer)**: when `tail` reaches the end of the array, wrap it back to index 0 and reuse the dead slots. This keeps the array fully utilised without reallocation. See [[adsa-circular-queue]] for the full treatment.

---

## Queue using a Linked List

Use a singly linked list with two pointers: `head` (front) and `tail` (rear).

```
head                            tail
 ↓                               ↓
[10|*]→[20|*]→[30|*]→[40|NULL]
```

**Enqueue**: allocate new node, `tail.next = new`, `tail = new`. **O(1)**.

**Dequeue**: read `head.val`, `head = head.next`. **O(1)**.

The `tail` pointer is critical — without it, enqueue would require walking to the end every time: O(n).

---

## Cache-Miss Problem with Linked Lists

Both LL stacks and LL queues pay a hidden cost that array-based structures avoid.

### Array — contiguous memory

```
RAM:  [ 10 | 20 | 30 | 40 | 50 | 60 | 70 | 80 ]
       └──────────── one cache line (64 bytes) ─────────────┘
```

One cache miss loads the whole line. For 8-byte integers: **1 miss → 8 elements ready**.
**Ratio: 1 miss : 7 hits.**

### Linked list — scattered memory

```
0x1000: [10 | → 0x4FA0]
0x4FA0: [20 | → 0xB210]
0xB210: [30 | → 0x0340]
  ...
```

Each node is at an arbitrary address. Every access is a fresh cache miss.
**8 accesses → 8 misses. Ratio: 1 miss : 0 hits.**

### Comparison

| | Dynamic Array | Linked List |
|---|---|---|
| Enqueue / Push | Amortized O(1) | O(1) |
| Dequeue / Pop | O(1) | O(1) |
| Memory waste | Up to 2× (growth factor) / dead slots | 8 bytes pointer overhead per node |
| Cache behaviour | 1 miss : 7 hits | 1 miss : 0 hits |
| Resize needed | Yes (amortized) | No |

For most workloads, **array-backed structures win** in practice because cache efficiency dominates at scale, despite the occasional resize cost.

---

## Breadth-First Search (BFS)

### Why a Queue (not a Stack) gives Level-by-Level Expansion

BFS explores a graph **one layer at a time** — all nodes at distance 1 from the source, then all at distance 2, and so on. This layered behaviour falls naturally out of the FIFO property: every node enqueued at level k is processed and its neighbours discovered before any level-(k+1) node is touched.

A stack would give DFS (depth-first) behaviour — diving deep into one path before backtracking. A queue guarantees breadth-first expansion. See [[adsa-stacks]] for the DFS comparison.

```
BFS algorithm:
  enqueue(source); mark source visited; distance[source] = 0
  while queue is not empty:
      u = dequeue()
      for each neighbour v of u:
          if v not visited:
              mark v visited
              distance[v] = distance[u] + 1
              parent[v] = u
              enqueue(v)
```

### Illustrated on a 6-Node Graph

Graph (undirected edges):

```
    A --- B --- E
    |     |
    C --- D --- F
```

Adjacency list:
- A: [B, C]
- B: [A, D, E]
- C: [A, D]
- D: [B, C, F]
- E: [B]
- F: [D]

BFS from source **A**:

**Start:**
```
Queue: [A]    visited: {A}    dist: {A:0}
```

**Step 1 — dequeue A, enqueue A's unvisited neighbours B, C:**
```
Process A. Neighbours: B (unvisited → enqueue), C (unvisited → enqueue)
Queue: [B, C]    visited: {A,B,C}    dist: {A:0, B:1, C:1}
```

**Step 2 — dequeue B, enqueue B's unvisited neighbours D, E:**
```
Process B. Neighbours: A (visited), D (unvisited → enqueue), E (unvisited → enqueue)
Queue: [C, D, E]    visited: {A,B,C,D,E}    dist: {A:0, B:1, C:1, D:2, E:2}
```

**Step 3 — dequeue C, no new neighbours:**
```
Process C. Neighbours: A (visited), D (visited)
Queue: [D, E]    no new nodes
```

**Step 4 — dequeue D, enqueue F:**
```
Process D. Neighbours: B (visited), C (visited), F (unvisited → enqueue)
Queue: [E, F]    visited: {A,B,C,D,E,F}    dist: {A:0, B:1, C:1, D:2, E:2, F:3}
```

**Step 5 — dequeue E, no new neighbours:**
```
Process E. Neighbours: B (visited)
Queue: [F]
```

**Step 6 — dequeue F, no new neighbours:**
```
Process F. Neighbours: D (visited)
Queue: []    BFS complete.
```

**Result — shortest distances from A:**
```
A:0   B:1   C:1   D:2   E:2   F:3
```

BFS found the shortest (hop-count) path from A to every reachable node.

### Why BFS Guarantees Shortest Paths (Unweighted)

When BFS first visits a node, it does so via the shortest route. Here is the intuition:

- All level-1 nodes are processed before any level-2 node.
- A level-2 node can only be reached from a level-1 node.
- So any level-2 node discovered has distance exactly 2.

By induction, the first time BFS visits any node v, `distance[v]` is the minimum number of edges from the source.

**Note**: BFS gives shortest paths only in **unweighted** graphs. For weighted graphs, use Dijkstra's algorithm.

### Complexity Analysis

```
Every vertex is enqueued once and dequeued once: O(V)
Every edge is examined when its endpoint is dequeued: O(E)
Total: O(V + E)

Space: O(V) for the queue and visited array
```

(source: `3 - Queue-FIFO-Round-Robin-and-BFS.pdf`)

---

## Level-Order Tree Traversal

Level-order traversal is BFS applied to a binary tree. It visits all nodes at depth 0, then depth 1, then depth 2, etc. This is the natural output order when you want to print a tree row by row.

### Example Binary Tree

```
          1          ← depth 0
        /   \
       2     3       ← depth 1
      / \     \
     4   5     6     ← depth 2
```

### Step-by-Step Trace

**Start:** enqueue root (1)
```
Queue: [1]
Output so far: (nothing)
```

**Step 1:** dequeue 1, enqueue children 2 and 3
```
Queue: [2, 3]
Output: 1
```

**Step 2:** dequeue 2, enqueue children 4 and 5
```
Queue: [3, 4, 5]
Output: 1 2
```

**Step 3:** dequeue 3, enqueue child 6 (no left child)
```
Queue: [4, 5, 6]
Output: 1 2 3
```

**Step 4:** dequeue 4, no children
```
Queue: [5, 6]
Output: 1 2 3 4
```

**Step 5:** dequeue 5, no children
```
Queue: [6]
Output: 1 2 3 4 5
```

**Step 6:** dequeue 6, no children
```
Queue: []
Output: 1 2 3 4 5 6
```

Final level-order output: **1, 2, 3, 4, 5, 6**

The pattern: for each dequeued node, enqueue its left child (if it exists), then its right child (if it exists). The FIFO ordering ensures all nodes at depth k finish before any at depth k+1.

### Pseudocode

```
level_order(root):
    if root is null: return
    queue.enqueue(root)
    while queue is not empty:
        node = queue.dequeue()
        process(node)
        if node.left  is not null: queue.enqueue(node.left)
        if node.right is not null: queue.enqueue(node.right)
```

**Complexity**: O(N) time, O(W) space where W is the maximum width of the tree (the widest level). For a perfectly balanced tree, W = N/2, so space is O(N) in the worst case.

---

## Round-Robin Scheduling

Round-robin is a **preemptive scheduling algorithm** that gives each process a fixed **time quantum** — the maximum CPU time it gets before being interrupted. When a process's quantum expires, it is re-enqueued at the rear. The next process at the front gets to run. This continues cyclically until all processes finish.

The queue structure enforces **fairness**: no process waits more than (N-1) × quantum time before getting another turn, where N is the number of processes in the ready queue.

```
Round-Robin queue cycle:

     dequeue        run for quantum        re-enqueue (if not done)
  [front] -------> [CPU]  -------> done? remove : [rear]
     ↑                                              ↓
     └──────────── circular FIFO ──────────────────┘
```

(source: `3 - Queue-FIFO-Round-Robin-and-BFS.pdf`)

### Step-by-Step Example: 3 Processes, Quantum = 2 Time Units

Initial ready queue (processes listed with their total CPU burst time):

```
Ready queue:  [ P1(burst=5) | P2(burst=3) | P3(burst=4) ]
               front                        rear
```

**Time 0–2: Run P1 for quantum=2.** P1 has 5-2=3 units remaining. Re-enqueue P1.

```
After time 2:
CPU ran: P1 (used 2 units, needs 3 more)
Queue:  [ P2(3) | P3(4) | P1(3) ]
```

**Time 2–4: Run P2 for quantum=2.** P2 has 3-2=1 unit remaining. Re-enqueue P2.

```
After time 4:
CPU ran: P2 (used 2 units, needs 1 more)
Queue:  [ P3(4) | P1(3) | P2(1) ]
```

**Time 4–6: Run P3 for quantum=2.** P3 has 4-2=2 units remaining. Re-enqueue P3.

```
After time 6:
CPU ran: P3 (used 2 units, needs 2 more)
Queue:  [ P1(3) | P2(1) | P3(2) ]
```

**Time 6–8: Run P1 for quantum=2.** P1 has 3-2=1 unit remaining. Re-enqueue P1.

```
After time 8:
Queue:  [ P2(1) | P3(2) | P1(1) ]
```

**Time 8–9: Run P2 for quantum=1 (only 1 left).** P2 finishes. Remove from queue.

```
After time 9:
P2 DONE (total wait time was fair — it never waited more than 2 quanta at a time)
Queue:  [ P3(2) | P1(1) ]
```

**Time 9–11: Run P3 for quantum=2.** P3 finishes.

```
After time 11:
P3 DONE
Queue:  [ P1(1) ]
```

**Time 11–12: Run P1 for quantum=1 (only 1 left).** P1 finishes.

```
After time 12:
P1 DONE
Queue:  [] — all processes complete
```

**Summary table:**

| Process | Burst | Finish Time | Turnaround |
|---|---|---|---|
| P1 | 5 | 12 | 12 |
| P2 | 3 | 9 | 9 |
| P3 | 4 | 11 | 11 |

**Key observations:**
- No process waits indefinitely — **no starvation**.
- A smaller quantum improves responsiveness but increases context-switch overhead.
- A larger quantum approaches FCFS (First-Come First-Served) behaviour.
- Linux's default quantum is approximately **4–100 ms** depending on system load (source: `3 - Queue-FIFO-Round-Robin-and-BFS.pdf`).

### Quantum Choice Trade-off

```
Quantum too small:
  + Very responsive to interactive tasks
  - High context-switch overhead (saving/restoring registers is not free)

Quantum too large:
  + Low context-switch overhead
  - Poor responsiveness; approaches FCFS

Optimal (studies show):
  10–20 ms for interactive workloads on modern multi-core systems
```

---

## Related Pages

- [[adsa-circular-queue]] — fixes the dead-slot problem with modulo wrap
- [[adsa-stacks]] — LIFO counterpart; DFS uses a stack where BFS uses a queue
- [[adsa-deque]] — double-ended queue generalisation

#adsa
