# Circular Linked List

**Summary**: A linked list variant where the tail node's `next` points back to the head, forming a closed loop — no `NULL` terminator.

**Pre-Req**: Basic singly/doubly linked list ([[adsa-linked-lists]]), [[adsa-queues]]

**Sources**: Session-built (2026-06-05). Updated with `raw/2 - Circular-Linked-Lists-A-Comprehensive-Learning-Guide.pdf`.

**Last updated**: 2026-06-09

---

## What Makes It Circular?

In a normal singly linked list, the tail's `next` is `NULL`. In a circular linked list (CLL), it points back to the head.

```
 ┌─────────────────────────────┐
 ↓                             |
[10|→] → [20|→] → [30|→] → [40|→]
 ↑
head
```

There is no sentinel `NULL`. Traversal must stop when it revisits the starting node, not when it hits `NULL`.

---

## Keeping a Tail Pointer (not head)

Keeping a **tail** pointer instead of head is the practical default:

- `tail.next` gives the head in O(1) — no extra pointer needed
- Insert at **front**: `new.next = tail.next`, `tail.next = new` — O(1)
- Insert at **back**: same as front, then advance `tail = new` — O(1)
- Both ends reachable in O(1) without a separate head pointer

With only a head pointer, inserting at the tail requires walking to the last node: O(n).

---

## Traversal

Because there is no `NULL`, the loop terminates on revisiting the start node:

```
current = head
do:
    visit(current)
    current = current.next
while current != head
```

A common bug: using `while current != NULL` — this loops forever on a CLL.

---

## Singly vs Doubly Circular

| | Singly CLL | Doubly CLL |
|---|---|---|
| Pointers per node | 1 (`next`) | 2 (`next`, `prev`) |
| Traverse backward | No | Yes |
| Delete arbitrary node | O(n) — need predecessor | O(1) — use `prev` directly |
| Memory | Lower | Higher |

Doubly circular LL combines the "no NULL" property with full bidirectional traversal. Used in the Linux kernel's `list_head` intrusive list implementation.

---

## Operations Summary

| Operation | Singly CLL (tail ptr) | Doubly CLL |
|---|---|---|
| Insert front | O(1) | O(1) |
| Insert back | O(1) | O(1) |
| Delete front | O(1) | O(1) |
| Delete arbitrary node | O(n) | O(1) |
| Search | O(n) | O(n) |

---

## Use Cases

- **Round-robin scheduling**: OS cycles through process list endlessly — natural fit; no special end-of-list handling needed.
- **Circular buffer backing**: the ring buffer in [[adsa-deque]] wraps indices the same way a CLL wraps pointers.
- **Multiplayer game turns**: player list is traversed in a cycle each round.
- **Music/video playlist loops**: "repeat all" is a circular traversal.

---

## Gotchas

- **Empty list**: a CLL with one node points to itself (`node.next = node`). Zero-node state requires a `NULL` head separately.
- **Infinite loop**: forgetting to use `do-while` with a start-node check is the most common bug.
- **Deletion edge case**: deleting the node that `tail` points to requires updating `tail` first.

---

## Floyd's Cycle Detection

Floyd's algorithm detects whether a linked list contains a cycle using two pointers and no extra memory. While a correctly constructed circular linked list is intentionally cyclic, the algorithm is critical for detecting *unintended* cycles caused by bugs or corrupted pointers.

### The Tortoise and Hare

```
slow = head   ← moves 1 step per iteration
fast = head   ← moves 2 steps per iteration

while fast != NULL and fast.next != NULL:
    slow = slow.next
    fast = fast.next.next
    if slow == fast:
        RETURN "cycle exists"

RETURN "no cycle"
```

### Why They Must Meet

If a cycle of length L exists, consider the moment slow enters the cycle. At that point, fast is already somewhere inside the cycle. Fast gains 1 step on slow every iteration. The gap between them (measured in the cycle's direction) reduces by 1 each step. The gap starts at most L and reaches 0 in at most L iterations — they must meet.

Step-by-step trace on a list with a cycle from node 5 back to node 2:

```
Nodes: head → [1] → [2] → [3] → [4] → [5]
                      ↑________________________↑

Step 0: slow=1, fast=1
Step 1: slow=2, fast=3
Step 2: slow=3, fast=5
Step 3: slow=4, fast=3  (fast wrapped: 5.next = 2, 2.next = 3)
Step 4: slow=5, fast=5  ← MEET. Cycle confirmed.
```

Time: **O(n)**. Space: **O(1)** — only two pointers regardless of list size.

### Finding the Cycle Start (Phase 2)

Once slow and fast meet, reset slow to head and keep fast at the meeting point. Advance both one step at a time. They will meet at exactly the cycle's entry node.

```
slow = head
while slow != fast:
    slow = slow.next
    fast = fast.next
RETURN slow           ← this is the cycle start node
```

This works because the distance from head to the cycle start equals the distance from the meeting point to the cycle start (a property provable with modular arithmetic). (source: `raw/2 - Circular-Linked-Lists-A-Comprehensive-Learning-Guide.pdf`)

---

## Doubly Circular Linked List (CDLL)

The Circular Doubly Linked List adds a `prev` pointer to every node in addition to `next`. Both the forward and backward directions wrap around.

### Structure and Invariants

```
┌─────────────────────────────────────────┐
│    ↙prev                    next↗        │
│  [A] ←──── [B] ←──── [C] ←──── [D]     │
│   │                              ↑       │
│   └──────── next ────────────────┘       │
│             forward                      │
└─────────────────────────────────────────┘

Invariants:
  head.prev == tail (last node)
  tail.next == head
  For any node x: x.next.prev == x
                  x.prev.next == x
```

Visual diagram with 4 nodes A, B, C, D:

```
      ┌────────────────────────────────────┐
      ↓                                    │
   ←prev [A] next→  ←prev [B] next→  ←prev [C] next→  ←prev [D] next→
      ↑                                    │
      └────────────────────────────────────┘

Forward:  A → B → C → D → A → ...
Backward: A → D → C → B → A → ...
```

### When to Use CDLL

| Situation | Why CDLL? |
|---|---|
| Frequent deletion of arbitrary nodes | `node.prev` gives predecessor in O(1); no traversal |
| OS process scheduler (Linux list_head) | Tasks removed from run queue in O(1) at any position |
| Turn-based multiplayer games | Player elimination is O(1); backward traversal for "previous player" |
| LRU cache eviction | Move accessed node to front in O(1) using both pointers |
| Text editor cursor movement | Move forward/backward through document nodes in O(1) |

### O(1) Arbitrary Deletion — Why It Matters

In a singly circular list, to delete node X you must find its predecessor (walk from head until `current.next == X`) — that is O(n). With prev pointers, deletion is just:

```
FUNCTION deleteNode(node):
    node.prev.next = node.next     ← bypass node in forward direction
    node.next.prev = node.prev     ← bypass node in backward direction
    if node == head:
        head = node.next           ← update head if needed
    free(node)
```

This is the critical advantage in OS schedulers that remove processes at any position millions of times per second (source: `raw/2 - Circular-Linked-Lists-A-Comprehensive-Learning-Guide.pdf`).

---

## Applications

### Round-Robin CPU Scheduling

The Linux kernel's CFS (Completely Fair Scheduler) manages the ready queue as a circular structure. After each process runs its time quantum, it moves to the back. The circular structure means "next process" is always one pointer dereference away — no end-of-queue detection needed.

```
Ready Queue:
Head → [P1] → [P2] → [P3] → [P4]
         ↑___________________________↑

Scheduler loop (simplified):
  current = dequeue_front()
  run_process(current, time_quantum)
  if process not finished:
      enqueue_back(current)        ← goes back into the circle
  context_switch_to(next)

// No "end of list" check ever needed.
// The circular structure handles wrap automatically.
```

Industry figure: round-robin with quantum=4ms achieves average response times of 8–12ms for interactive processes, well within the 100ms human perception threshold. Linux's CFS services over **1 billion** Linux devices using circular queue principles (source: `raw/2 - Circular-Linked-Lists-A-Comprehensive-Learning-Guide.pdf`).

### Music Playlist Repeat Mode

Spotify, Apple Music, VLC, and YouTube Music — collectively serving over 700 million active users — implement "repeat all" using circular playlist logic:

```
Playlist (circular):
[Song1] → [Song2] → [Song3] → [Song4]
    ↑________________________________↑

Playback loop:
  current = head
  while repeat_all and player_active:
      play(current)
      current = current.next    ← no "if current == NULL" check needed
                                ← current always points to a valid song
```

Without a circular list, every song transition requires checking `if current == NULL: current = head` — adding a branch that is error-prone and slightly slower (source: `raw/2 - Circular-Linked-Lists-A-Comprehensive-Learning-Guide.pdf`).

### Memory Pool Rotation

Dynamic-size circular buffers for audio/video streaming use a circular linked list of memory blocks. Unlike fixed-size ring arrays, linked circular buffers grow on demand by inserting new blocks into the cycle. The producer writes to the current block, advances the tail; the consumer reads from the head and releases blocks. The circular structure means neither producer nor consumer ever needs to check for "end of buffer".

### Josephus Problem

n people stand in a circle; every k-th person is eliminated until one remains. This maps naturally onto a circular linked list simulation:

```
Josephus(n=5, k=2):
  Circle: [1] → [2] → [3] → [4] → [5] → back to [1]

  Count k=2 steps from current: eliminate person 2 → Circle: [1,3,4,5]
  Count 2 steps from 3: eliminate person 4 → Circle: [1,3,5]
  Count 2 steps from 5: eliminate person 1 → Circle: [3,5]
  Count 2 steps from 3: eliminate person 5 → Survivor: 3
```

Circular list simulation is O(n×k). The mathematical formula `survivor = (survivor + k) % i` reduces it to O(n). Historical note: Josephus (1st century AD) reportedly used this calculation at position 31 of 41 soldiers to determine the safe position (source: `raw/2 - Circular-Linked-Lists-A-Comprehensive-Learning-Guide.pdf`).

---

## Circular vs. Linear List: Decision Guide

| Use case | Best choice | Reason |
|---|---|---|
| Round-robin scheduling, dynamic | Circular singly LL | O(1) rotation; dynamic add/remove |
| Fixed-size audio ring buffer | Circular array | Cache locality; 5–10× faster than linked |
| Playlist with repeat | Either | Linked for dynamic; array for fixed |
| Frequent arbitrary deletions | Circular doubly LL | O(1) delete any node |
| Josephus / cyclic elimination | Circular singly LL | Natural model for the problem |

Cache locality is the primary reason circular arrays outperform linked variants in high-throughput streaming — CPU cache lines hold contiguous array elements, reducing cache misses by up to **10×** compared to pointer-chased linked nodes (source: `raw/2 - Circular-Linked-Lists-A-Comprehensive-Learning-Guide.pdf`).

#adsa
