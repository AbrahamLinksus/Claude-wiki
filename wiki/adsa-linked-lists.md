# Linked Lists

**Summary**: A linked list is a chain of nodes where each node holds data and a pointer to the next node. Unlike arrays, nodes are scattered in heap memory — giving O(1) insert/delete at a known position but O(n) random access and poor cache locality.

**Pre-Req**: Pointers and dynamic memory allocation, Big-O notation, [[adsa-arrays]]

**Sources**: `raw/3 - Linked-Lists.pdf`

**Last updated**: 2026-06-09

---

## What Is a Linked List?

An array stores elements in a single contiguous block — every element lives right next to its neighbour. A linked list abandons contiguity entirely. Each element lives in an independently allocated **node**, and nodes are connected only through pointers.

```
Array (contiguous):
[ 10 | 20 | 30 | 40 ]
  base+0  +4  +8  +12   ← arithmetic jumps get you anywhere in O(1)

Linked List (scattered):
 [10|*]──►[20|*]──►[30|*]──►[40|NULL]
  0x4a00   0x71f0   0x2c10   0x9830   ← arbitrary heap addresses
  ↑
 head
```

Every node contains exactly two things:

```
Node {
    data    ← the value stored (any type)
    next    ← pointer to the next node (NULL if last)
}
```

There are three main variants:
- **Singly Linked List (SLL)** — each node has one pointer: `next`
- **Doubly Linked List (DLL)** — each node has two: `next` and `prev`
- **Circular Linked List (CLL)** — the tail's `next` wraps back to head (see [[adsa-circular-ll]])

---

## Why Does It Exist? (vs Arrays)

Arrays are fast for random access but slow when you need to insert or delete in the middle — every element after the insertion point must shift one position:

```
Insert 99 at index 1 in [10, 20, 30, 40]:
  Step 1: shift 40 → index 3
  Step 2: shift 30 → index 2     ← O(n) shifts
  Step 3: shift 20 → index 1
  Step 4: write 99 at index 1
Result: [10, 99, 20, 30, 40]
```

With a linked list, if you already hold a pointer to the predecessor node, insertion is just pointer rewiring — three assignments, regardless of list size:

```
Insert 99 after node [10|*]:
  new.data = 99
  new.next = prevNode.next        ← point new node at the old successor
  prevNode.next = new             ← point predecessor at new node
  Done in O(1)
```

Key trade-off summary:

| Situation | Choose |
|---|---|
| Random access by index is dominant | Array |
| Frequent insert/delete at known positions | Linked List |
| Cache performance is critical | Array |
| Size is highly dynamic and unpredictable | Linked List |
| OS process management (add/remove processes) | Linked List |

Real-world evidence: `std::vector` outperforms `std::list` by **3–10× for sequential traversal** on modern hardware due to cache locality. Use linked lists deliberately, not by default (source: `raw/3 - Linked-Lists.pdf`).

---

## Node Structure — ASCII Diagrams

### Singly Linked Node

```
┌──────────┬──────────┐
│   data   │   next   │
│ (any type)│  (ptr)  │
└──────────┴──────────┘
              │
              ▼
         next node or NULL
```

Memory: **~16 bytes** on 64-bit systems (8 bytes data + 8 bytes pointer).

### Doubly Linked Node

```
┌──────────┬──────────┬──────────┐
│   prev   │   data   │   next   │
│  (ptr)   │          │  (ptr)   │
└──────────┴──────────┴──────────┘
     │                      │
     ▼                      ▼
prev node              next node
```

Memory: **~24 bytes** on 64-bit systems (8 prev + 8 data + 8 next).

The `prev` pointer costs 8 extra bytes per node but buys O(1) arbitrary deletion and backward traversal.

---

## Singly Linked List — Core Operations

### Head Pointer and Traversal

The entire list is accessible through a single `head` pointer. Lose `head` and the list is gone forever.

```
head
 ↓
[10|*]──►[20|*]──►[30|*]──►[40|NULL]

Traversal:
  current = head
  while current != NULL:
      visit(current.data)
      current = current.next
```

Complexity: **O(n)** — must visit every node to reach the last.

### Search

```
FUNCTION search(head, target):
    current = head
    while current != NULL:
        if current.data == target: RETURN current
        current = current.next
    RETURN NULL           ← not found
```

Complexity: **O(n)** worst case. No index arithmetic — must walk from head.

---

## Insertion Operations

### Insert at Head — O(1)

The cheapest insertion. Wire the new node to point at the current head, then update head.

```
Before:  head ──► [20|*]──►[30|NULL]

Insert 10 at head:
  Step 1: new.next = head        new──►[20|*]──►[30|NULL]
  Step 2: head = new             head──►[10|*]──►[20|*]──►[30|NULL]

After:   head ──► [10|*]──►[20|*]──►[30|NULL]
```

No traversal needed. Exactly 2 pointer assignments.

### Insert at Tail — Without Tail Pointer: O(n)

```
FUNCTION insertAtTail(head, value):
    new = Node(value)
    if head == NULL: head = new; RETURN
    current = head
    while current.next != NULL:    ← walk to the last node
        current = current.next
    current.next = new             ← wire last node to new node
```

Walk takes O(n). Fix: maintain a `tail` pointer.

### Insert at Tail — With Tail Pointer: O(1)

```
tail.next = new       ← old tail points to new node
tail = new            ← advance tail pointer
```

Two assignments. This is why production implementations always maintain both `head` and `tail`.

### Insert at Arbitrary Position — O(n)

To insert after position k, walk to node k-1 (the predecessor), then rewire:

```
Insert after index 1 in: head──►[10]──►[20]──►[30]──►NULL
                                         ↑ insert 99 here

Step 1: Walk to node at index 1 (value 10)
Step 2: new.next = prevNode.next        [99]──►[20]──►[30]──►NULL
Step 3: prevNode.next = new             [10]──►[99]──►[20]──►[30]──►NULL

Result: head──►[10]──►[99]──►[20]──►[30]──►NULL
```

The walk is O(n). The rewiring is O(1). Combined: O(n).

---

## Deletion

To delete a node, you must locate its **predecessor** — the node whose `next` points to the target. Then skip over the target:

```
Delete node with value 20:

Before: head──►[10|*]──►[20|*]──►[30|NULL]

Step 1: Walk until prevNode.next.data == 20
        prevNode = [10|*]
Step 2: temp = prevNode.next              ← save target node
Step 3: prevNode.next = temp.next         ← skip over target
Step 4: free(temp)                        ← release memory (in C/C++)

After:  head──►[10|*]──►[30|NULL]
```

Edge case: deleting the head node requires no walk — just advance head:

```
FUNCTION deleteHead(head):
    temp = head
    head = head.next
    free(temp)
    RETURN head
```

### Why Doubly Linked Lists Help Deletion

In a singly linked list, deleting a node given only that node's reference is O(n) — you need to find its predecessor first. In a doubly linked list, `node.prev` gives the predecessor directly, so deletion is O(1):

```
DLL deletion (given node reference):
  node.prev.next = node.next
  node.next.prev = node.prev
  free(node)
```

This is the reason the Linux kernel uses a doubly linked `list_head` for process scheduling.

---

## In-Place Reversal

Reversing a singly linked list in-place is a classic operation. It requires three pointers and one pass.

### Three-Pointer Iterative Approach

```
prev = NULL
current = head

while current != NULL:
    nextTemp = current.next    ← save before overwriting
    current.next = prev        ← reverse the pointer
    prev = current             ← advance prev
    current = nextTemp         ← advance current

head = prev                    ← prev is now at the new front
```

Step-by-step trace on [10, 20, 30]:

```
Initial:
  prev=NULL  current=[10|*]──►[20|*]──►[30|NULL]

Iteration 1:
  nextTemp = [20|*]──►[30|NULL]
  [10].next = NULL              ← pointer reversed
  prev = [10|NULL]
  current = [20|*]──►[30|NULL]

Iteration 2:
  nextTemp = [30|NULL]
  [20].next = [10|NULL]         ← pointer reversed
  prev = [20|*]──►[10|NULL]
  current = [30|NULL]

Iteration 3:
  nextTemp = NULL
  [30].next = [20|*]──►[10|NULL]   ← pointer reversed
  prev = [30|*]──►[20|*]──►[10|NULL]
  current = NULL

head = prev = [30|*]──►[20|*]──►[10|NULL]
```

Result: [30, 20, 10]. Complexity: **O(n) time, O(1) space**.

---

## Floyd's Cycle Detection (Tortoise and Hare)

Given a linked list that may contain a cycle (tail's `next` points back to some earlier node, creating an infinite loop), detect whether a cycle exists.

**The Algorithm**: Use two pointers — `slow` moves 1 step per iteration, `fast` moves 2 steps. If there is a cycle, fast will eventually lap slow and they will meet at the same node. If there is no cycle, fast will reach NULL.

```
slow = head
fast = head

while fast != NULL and fast.next != NULL:
    slow = slow.next
    fast = fast.next.next
    if slow == fast:
        RETURN "cycle detected"

RETURN "no cycle"
```

Visual trace (list with cycle from node 4 back to node 2):

```
Nodes: 1 ──► 2 ──► 3 ──► 4 ──► 5
                    ↑               │
                    └───────────────┘

Step 0: slow=1, fast=1
Step 1: slow=2, fast=3
Step 2: slow=3, fast=5
Step 3: slow=4, fast=4  ← MEET! Cycle confirmed.
```

**Why they must meet**: In a cycle of length L, fast gains 1 step on slow per iteration. The gap closes to zero in at most L iterations. Complexity: **O(n) time, O(1) space**.

### Finding the Cycle Start (Floyd Phase 2)

After detecting the meeting point, reset `slow` to `head`. Advance both `slow` and `fast` at the same speed (1 step each). They will meet exactly at the cycle's start node.

```
slow = head
while slow != fast:
    slow = slow.next
    fast = fast.next
RETURN slow            ← cycle start node
```

This works due to a mathematical property: the distance from head to cycle start equals the distance from the meeting point to cycle start. (source: `raw/3 - Linked-Lists.pdf`)

---

## Types and Variants

| Type | Pointers/Node | Backward Traversal | Delete Arbitrary | Use Case |
|---|---|---|---|---|
| Singly LL | 1 (`next`) | No | O(n) | Simple forward chains |
| Doubly LL | 2 (`next`, `prev`) | Yes | O(1) | LRU cache, undo/redo |
| Circular Singly | 1 (`next`→head) | No | O(n) | Round-robin scheduling |
| Circular Doubly | 2 (both wrap) | Yes | O(1) | OS schedulers, playlists |

Special variants worth knowing:
- **Sentinel Node**: a dummy head/tail node that is never removed — eliminates empty-list edge cases in insertion/deletion code
- **XOR Linked List**: stores `prev XOR next` in a single pointer field, halving memory; not portable (requires pointer arithmetic)
- **Unrolled Linked List**: each node holds a small array — better cache locality than pure LL, used in `std::deque` style allocations (see [[adsa-deque]])
- **Skip List**: layered LL structure achieving O(log n) search; used in Redis sorted sets

---

## Memory Layout and Cache Performance

This is the core weakness of linked lists. Every node is independently allocated and can be anywhere on the heap:

```
Array (cache-friendly):
Physical memory: [10][20][30][40][50][60][70][80]
                  one cache line loads 8 elements at once

Linked List (cache-hostile):
Physical memory:
  addr 0x4a00: [10|ptr→0x71f0]
  addr 0x2c10: [30|ptr→0x9830]   ← node 3 is far from node 2
  addr 0x71f0: [20|ptr→0x2c10]
  addr 0x9830: [40|NULL]
  Every node access = possible cache miss
```

Cache line is typically 64 bytes. An array loads ~8 integers per cache miss. A linked list loads 1 node per cache miss — even if the processor prefetches correctly, it cannot predict the next pointer's destination until it actually loads the current node.

**Industry benchmark**: pointer chasing is **3–5× slower** than sequential array access on modern hardware (source: `raw/3 - Linked-Lists.pdf`).

---

## Array vs. Linked List — Full Comparison

| Feature | Array | Singly LL | Doubly LL |
|---|---|---|---|
| Random access | O(1) | O(n) | O(n) |
| Insert at head | O(n) shift | O(1) | O(1) |
| Insert at tail | O(1) amortized | O(1) w/tail ptr | O(1) |
| Insert at middle | O(n) shift | O(1) given prev | O(1) given node |
| Delete arbitrary | O(n) shift | O(n) find prev | O(1) given node |
| Search | O(n) / O(log n) sorted | O(n) | O(n) |
| Cache locality | Excellent | Poor | Poor |
| Memory per element | 0 pointers | 1 pointer (~8 B) | 2 pointers (~16 B) |
| Prefix sums possible | Yes | No | No |

**When to choose arrays**: random access by index is dominant, cache performance is critical, advanced techniques (prefix sums, sliding window) are needed.

**When to choose linked lists**: frequent insertions/deletions at arbitrary positions, size is highly dynamic, memory fragmentation is acceptable, implementing certain graph algorithms (adjacency lists).

---

## Real-World Applications

### LRU Cache (Most Common Interview Application)

A Least Recently Used cache is implemented with a **doubly linked list + hash map**. The DLL maintains access order; the hash map gives O(1) lookup. On access: move the node to the front of the DLL in O(1) using `prev`/`next` pointers. On eviction: remove from the back in O(1).

```
hashmap: key → DLL node pointer
DLL: [MRU] ← [B] ← [A] ← [C] → [LRU]

get(B): find B in hashmap, move to front → O(1)
put(D): insert at front, evict LRU tail → O(1)
```

Powers Redis, Memcached — serving billions of requests per day (source: `raw/3 - Linked-Lists.pdf`).

### OS Process Scheduler

Linux maintains all process descriptors (`task_struct`) in an intrusive doubly circular linked list using the `list_head` pattern. The scheduler cycles through them in O(1) per context switch. Linux manages millions of processes globally using this structure.

### Undo/Redo Stack

A doubly linked list where "current position" is a pointer into the list. Undo = follow `prev`. Redo = follow `next`. Inserting a new action truncates the forward chain.

---

## Common Gotchas

| Mistake | Why it happens | Impact | Fix |
|---|---|---|---|
| Losing the head pointer | Advancing head without saving it | Entire list is leaked | Always save or never directly modify `head` in traversal |
| NULL dereference | Not checking `current != NULL` before `current.next` | Crash | Check NULL before every dereference |
| Forgetting to update tail | Inserting at end without moving tail pointer | O(n) appends instead of O(1) | Always update `tail` on every tail insertion |
| Cycle injection | Accidentally setting `node.next` to an ancestor | Infinite loop | Never set `next` without verifying it doesn't close a cycle |
| Use-after-free (C/C++) | Freeing a node but keeping a pointer to it | Memory corruption | Set pointer to NULL immediately after free |
| Incorrect DLL reversal | Swapping `next` but not `prev` | Broken backward links | Swap both `next` and `prev` for every node |

Memory aid: "Insert after is easy, insert before needs a door" (you need the previous node). "Tortoise and hare meet where the cycle's there." "Prev null, curr head, walk and flip, reverse ahead."

---

## Performance Summary

| Operation | Singly LL | Doubly LL | Array |
|---|---|---|---|
| Access by index | O(n) | O(n) | O(1) |
| Insert at head | O(1) | O(1) | O(n) |
| Insert after known node | O(1) | O(1) | O(n) |
| Insert before known node | O(n) | O(1) | O(n) |
| Delete head | O(1) | O(1) | O(n) |
| Delete given node | O(n) | O(1) | O(n) |
| Search | O(n) | O(n) | O(n) |
| Memory per node | ~16 bytes | ~24 bytes | ~8 bytes |

Scale considerations (source: `raw/3 - Linked-Lists.pdf`):
- n < 1,000: LL is acceptable, performance difference negligible
- n ~ 10^5: LL noticeably slower for traversal-heavy workloads
- n ~ 10^7: LL poor for sequential workloads; only use where insert/delete dominates

---

## Recommended Next Topics

- [[adsa-circular-ll]] — tail wraps to head, natural round-robin scheduling
- [[adsa-stacks]] — stack implementation via linked list (O(1) push/pop)
- [[adsa-queues]] — queue implementation using head + tail pointers
- [[adsa-deque]] — doubly linked list as the backing structure for deque
- [[adsa-hashing]] — LRU cache = DLL + hash map, the canonical combined structure

#adsa
