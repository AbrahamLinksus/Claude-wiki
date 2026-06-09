# Circular Queue

**Summary**: A circular queue (ring buffer) fixes the dead-slot waste of a linear array queue by wrapping the rear index back to 0 using modulo arithmetic — giving O(1) enqueue and dequeue with full array utilisation.

**Pre-Req**: [[adsa-queues]] — FIFO principle, linear array queue, dead-slot problem

**Sources**: `4 - Circular-Queue-Concept.pdf`, `1 - Stack-and-Queue-Using-Arrays-and-Linked-Lists.pdf`

**Last updated**: 2026-06-09

---

## The Problem: Why Linear Queues Waste Memory

Before diving into circular queues, it helps to understand the exact problem they solve. This was introduced in [[adsa-queues]] but is worth recapping concretely here.

A linear array queue tracks two indices: `front` (where to dequeue from) and `rear` (where to enqueue next).

```
Initial: capacity = 6, front = 0, rear = 0
[ _ | _ | _ | _ | _ | _ ]
  0   1   2   3   4   5

After enqueue 10, 20, 30:
front = 0, rear = 3
[ 10 | 20 | 30 | _  | _  | _  ]
  0    1    2    3    4    5

After dequeue x2 (removes 10 and 20):
front = 2, rear = 3
[ ?? | ?? | 30 | _  | _  | _  ]
  0    1    2    3    4    5
       DEAD SLOTS
```

Slots 0 and 1 are now permanently dead. The program still considers the array to start at index 2. As you enqueue more items, `rear` marches rightward. Eventually `rear` hits index 5 and the array reports "full" — even though two perfectly good slots sit unused at the front.

This is called the **dead-slot problem**. The only fixes in a linear queue are:
- Shift all elements left on every dequeue — **O(n), unacceptably slow**
- Accept the waste and reallocate when rear reaches the end — **wasteful and unpredictable**

The circular queue solves this cleanly by connecting the end of the array back to the beginning.

---

## The Circular Queue Concept

A circular queue treats the array as a **ring** — when `rear` reaches the last slot and would go out of bounds, it wraps back to index 0 and reuses the dead slots left behind by previous dequeues.

```
The ring mental model (capacity = 6):

         index 0
            |
    5 ------+------ 1
    |       |       |
    |    [RING]     |
    |               |
    4 ------+------ 2
            |
         index 3

Rear wraps: after slot 5, the next write goes to slot 0.
Front wraps: after dequeueing from slot 5, the next read comes from slot 0.
```

Both `front` and `rear` advance using the same formula:

```
new_index = (old_index + 1) % capacity
```

This single formula is the entire mathematical engine of a circular queue.

---

## The Modulo Formula — Why It Works

The `%` operator (modulo) returns the **remainder after division**. When the index reaches `capacity`, the remainder resets to 0.

Walk through it with capacity = 6:

```
index:  0  1  2  3  4  5  6  7  8  9  10  11
% 6:    0  1  2  3  4  5  0  1  2  3   4   5
                            ^
                     wraps back to 0
```

So `(5 + 1) % 6 = 0`. The index wraps seamlessly. No `if` statement needed. No bounds check needed. One arithmetic operation handles every case.

Applied to enqueue:

```
// Before writing, advance rear
arr[rear] = value
rear = (rear + 1) % capacity
```

Applied to dequeue:

```
// Read from front, then advance front
value = arr[front]
front = (front + 1) % capacity
```

Both operations are **O(1) always** — no loops, no shifting.

---

## ASCII Ring Diagram

Here is a 6-slot circular queue with front=1 and rear=4, holding three live elements (20, 30, 40):

```
         [_]  <- slot 0 (dead, was dequeued)
        /     \
  [_]           [20]  <- slot 1 = front (oldest live element)
  slot 5        slot 2

  [40]          [30]  <- slot 3
  slot 4        
        \     /
         [??] <- slot 4 = rear (next write goes here)

Direction of travel: rear advances clockwise (0→1→2→3→4→5→0→...)
                     front advances clockwise behind rear
```

More precisely as a flat array with labels:

```
Slot:    0     1     2     3     4     5
Array: [ _   | 20  | 30  | 40  | _   | _  ]
              ^front              ^rear

Live elements: arr[1], arr[2], arr[3]
Next enqueue: arr[4] = value; rear → 5
Next dequeue: return arr[1]; front → 2
```

---

## Full vs. Empty: The Off-by-One Danger

This is the trickiest part of circular queue design. After wrapping, both `front == rear` when the queue is empty (just initialised or everything was dequeued). But if you fill all N slots, `front == rear` again! You cannot tell them apart.

```
Empty queue (N=4):
  front = 0, rear = 0
  [ _ | _ | _ | _ ]

Full queue (N=4), 4 elements enqueued, 0 dequeued:
  front = 0, rear = 0  (rear wrapped all the way around!)
  [ A | B | C | D ]

Both states: front == rear. Indistinguishable.
```

There are three standard solutions:

### Convention 1: Waste One Slot (Recommended)

Declare the queue full when `(rear + 1) % N == front` — meaning rear is one step *behind* front. This leaves one slot permanently unused. Maximum real capacity = N - 1.

```
Empty:  front == rear
Full:   (rear + 1) % N == front

Example with N=4 (max 3 elements):
[ A | B | C | _ ]   <- full, slot 3 is the wasted sentinel
  ^front         ^rear   (rear points past the last written slot)
```

**Why this works:** The wasted slot creates a permanent gap between `rear` and `front` when full, so the two conditions never coincide.

### Convention 2: Maintain a Count

Track the number of live elements in a separate `count` variable. No slot is wasted — full capacity = N.

```
Empty:  count == 0
Full:   count == N

Enqueue: arr[rear] = val; rear = (rear+1)%N; count++
Dequeue: val = arr[front]; front = (front+1)%N; count--
```

This uses every slot but adds one integer of overhead per queue.

### Convention 3: Boolean Flag (Rare)

Maintain a `isFull` flag, set on the last enqueue, cleared on any dequeue. Error-prone and rarely used in production.

**Rule of thumb:**
- Use Convention 1 (waste a slot) for simplicity when the exact capacity is not critical.
- Use Convention 2 (count) when you need all N slots and can tolerate the extra variable.

---

## Step-by-Step Dry Run: 6-Slot Circular Queue

We use **Convention 2 (count)** so all 6 slots are usable.

Initial state: capacity = 6, front = 0, rear = 0, count = 0

```
Slot:  [ _  | _  | _  | _  | _  | _  ]
         0    1    2    3    4    5
front=0, rear=0, count=0
```

**Enqueue 10** — write at slot 0, rear → (0+1)%6 = 1, count = 1

```
Slot:  [ 10 | _  | _  | _  | _  | _  ]
         ^f    r
front=0, rear=1, count=1
```

**Enqueue 20** — write at slot 1, rear → 2, count = 2

```
Slot:  [ 10 | 20 | _  | _  | _  | _  ]
         ^f        r
front=0, rear=2, count=2
```

**Enqueue 30** — write at slot 2, rear → 3, count = 3

```
Slot:  [ 10 | 20 | 30 | _  | _  | _  ]
         ^f             r
front=0, rear=3, count=3
```

**Enqueue 40** — write at slot 3, rear → 4, count = 4

```
Slot:  [ 10 | 20 | 30 | 40 | _  | _  ]
         ^f                  r
front=0, rear=4, count=4
```

**Enqueue 50** — write at slot 4, rear → 5, count = 5

```
Slot:  [ 10 | 20 | 30 | 40 | 50 | _  ]
         ^f                       r
front=0, rear=5, count=5
```

**Dequeue** — return arr[0]=10, front → (0+1)%6=1, count = 4

```
Slot:  [ ?? | 20 | 30 | 40 | 50 | _  ]
               ^f                  r
front=1, rear=5, count=4
Slot 0 is now dead — but circular queue will reuse it!
```

**Dequeue** — return arr[1]=20, front → 2, count = 3

```
Slot:  [ ?? | ?? | 30 | 40 | 50 | _  ]
                    ^f              r
front=2, rear=5, count=3
Slots 0 and 1 are now available again.
```

**Enqueue 60** — write at slot 5, rear → (5+1)%6 = 0 (WRAPS!), count = 4

```
Slot:  [ ?? | ?? | 30 | 40 | 50 | 60 ]
                    ^f   r(=0)
front=2, rear=0, count=4
rear just wrapped from slot 5 back to slot 0.
```

**Enqueue 70** — write at slot 0 (REUSING DEAD SLOT), rear → 1, count = 5

```
Slot:  [ 70 | ?? | 30 | 40 | 50 | 60 ]
               r    ^f
front=2, rear=1, count=5
Slot 0 is alive again with value 70!
```

**Enqueue 80** — write at slot 1, rear → 2, count = 6 (FULL)

```
Slot:  [ 70 | 80 | 30 | 40 | 50 | 60 ]
                    ^f  ^r
front=2, rear=2, count=6
Queue is full — count==capacity. front and rear coincide.
```

The live elements in order are: 30, 40, 50, 60, 70, 80 — FIFO order preserved perfectly. The dead slots from earlier dequeues were completely reclaimed.

---

## Static Array Implementation (C-style Pseudocode)

```c
#define CAPACITY 6

int arr[CAPACITY];
int front = 0;
int rear  = 0;
int count = 0;

// Returns 1 on success, 0 if full
int enqueue(int value) {
    if (count == CAPACITY) return 0;   // full
    arr[rear] = value;
    rear = (rear + 1) % CAPACITY;
    count++;
    return 1;
}

// Returns the value, or -1 if empty
int dequeue() {
    if (count == 0) return -1;         // empty
    int value = arr[front];
    front = (front + 1) % CAPACITY;
    count--;
    return value;
}

int peek() {
    if (count == 0) return -1;
    return arr[front];
}

int isEmpty() { return count == 0; }
int isFull()  { return count == CAPACITY; }

// Size formula (works with both conventions):
// With count variable: just return count
// With waste-slot convention: return (rear - front + CAPACITY) % CAPACITY
```

Key points:
- `CAPACITY` is fixed at compile time — no dynamic allocation, no heap.
- `front` and `rear` are private indices, never exposed directly.
- Every index update applies `% CAPACITY` — forgetting this causes an index-out-of-bounds crash.
- This pattern is exactly what the Linux kernel's `kfifo` circular buffer uses (source: `4 - Circular-Queue-Concept.pdf`).

---

## Growing a Circular Queue (Dynamic Array)

A static circular queue has a fixed capacity decided at creation. If you need to grow it, the process is non-trivial because elements may be **split across the array boundary** due to wrap-around.

```
Before grow — capacity=4, front=2, rear=1, count=4:
[ C | D | A | B ]
       r   f
Live order: A, B, C, D (A is at index 2, wraps to D at index 1)

Naive copy to new array of size 8 would give:
[ C | D | A | B | _ | _ | _ | _ ]
       r   f
This is WRONG — C and D appear before A and B.
```

The correct grow procedure:

```
1. Allocate new_arr of size (2 * old_capacity)
2. Copy elements in logical order starting from front:
   for i in range(count):
       new_arr[i] = arr[(front + i) % old_capacity]
3. Set front = 0, rear = count
4. Replace arr with new_arr

After grow (correct):
[ A | B | C | D | _ | _ | _ | _ ]
  f=0              r=4
```

This is an **O(n)** operation — you must touch every element. Because of this cost, circular queues are usually given a carefully chosen fixed capacity upfront, and growth is avoided in real-time systems (source: `4 - Circular-Queue-Concept.pdf`).

**Grow trigger strategy:** When `count == capacity`, allocate a buffer of `2 * capacity`, copy in order, reset indices.

---

## Complexity Summary

| Operation | Time | Notes |
|---|---|---|
| `enqueue(x)` | O(1) | Write + modulo increment |
| `dequeue()` | O(1) | Read + modulo increment |
| `peek()` | O(1) | Read arr[front] |
| `isEmpty()` | O(1) | count == 0 |
| `isFull()` | O(1) | count == capacity |
| `size()` | O(1) | return count |
| `grow()` | O(n) | Rare; re-copy all elements |
| Space | O(N) | N = capacity, fixed at creation |

All normal operations are O(1) — this is the core advantage over the O(n) shifting of a naive linear queue.

---

## Real-World Applications

### Producer-Consumer Buffer

The canonical use case. A **producer** thread writes data into the queue; a **consumer** thread reads from it. They operate at independent rates. The circular queue smooths the bursts without dynamic allocation.

```
Producer             Circular Queue            Consumer
[generates data] --> [ | | | | | ] --> [processes data]

When queue is full:  producer blocks (or drops, depending on policy)
When queue is empty: consumer blocks (or polls)
```

The producer-consumer pattern powers: web server request queues, audio/video pipelines, OS I/O subsystems.

### Audio Streaming Buffer

A microphone produces **44,100 samples per second** (standard 44.1 kHz). The audio codec consumes at the same rate. A circular buffer of 4,096 samples sits between them, absorbing the microsecond jitter in timing so the output never glitches (source: `4 - Circular-Queue-Concept.pdf`).

```
Microphone → [circular buffer: 4096 samples] → Audio codec
              ↑
              If this overflows: audio crackle (dropped samples)
              If this underflows: audio dropout (silence)
```

Key requirement: the buffer must never overflow or underflow. O(1) guaranteed — no blocking from dynamic allocation.

### OS Scheduler Time Slots (Round Robin)

The operating system's process scheduler keeps a circular queue of runnable processes. Each process gets a fixed time quantum (typically 10–100 ms on Linux). After its quantum expires, it re-joins the rear of the queue. See [[adsa-queues]] for the full Round-Robin section.

```
Ready queue (circular):
[ P1 | P2 | P3 | P4 ]
  ^CPU runs this one, then moves it to rear
```

The circular buffer ensures no process starves — every process cycles through in bounded time.

### Network Router Packet Queue

A 100 Gbps router port may receive over 148 million packets per second. It uses a fixed-size circular queue with a **tail-drop policy**: when the queue is full, new packets are simply dropped rather than causing unbounded memory growth (source: `4 - Circular-Queue-Concept.pdf`).

### Real-Time Logging (Overwriting Buffer)

Keep only the last N log messages. When full, **overwrite the oldest entry** instead of rejecting the new one. This is a variant called an **overwriting circular buffer** — used in automotive ECU black boxes and avionics flight recorders.

```
Overwriting policy: when full, advance front one step (discarding oldest)
then write at rear. The buffer always holds the last N entries.
```

---

## Comparison: Linear Array Queue vs. Circular Queue vs. Linked List Queue

| Factor | Linear Array Queue | Circular Queue (Array) | Linked List Queue |
|---|---|---|---|
| Memory usage | Wastes dead slots at front | Full utilisation, fixed size | Per-node pointer overhead (8 bytes each) |
| enqueue | Amortized O(1) | O(1) strict | O(1) |
| dequeue | O(1) (but slots wasted) | O(1) strict | O(1) |
| Cache behaviour | Good (contiguous) | Excellent (contiguous) | Poor (pointer chasing) |
| Max size | Grows dynamically | Fixed at creation | Unlimited |
| Dynamic allocation | On resize | None (static) | Every enqueue |
| Real-time safe | No (resize unpredictable) | Yes | No (malloc may block) |
| Dead-slot problem | Yes | No | N/A |
| Implementation complexity | Simple | Moderate (modulo logic) | Simple |
| Best for | General purpose, unknown max size | Buffers, streaming, real-time | Unknown max size, GC languages |

The circular queue is the **default choice** for any fixed-capacity, performance-sensitive queue. The linked list queue is preferred when the maximum size is unknown and dynamic allocation is acceptable (source: `4 - Circular-Queue-Concept.pdf`).

---

## Common Mistakes and How to Avoid Them

| Mistake | Symptom | Fix |
|---|---|---|
| Forgetting `% capacity` on index update | Index goes past end of array, crash | Always write `rear = (rear+1) % N` |
| Using `front == rear` for both empty and full | Incorrect full/empty detection | Use count variable or waste-slot convention |
| Not resetting both front and rear on clear | Stale indices after a clear operation | Set `front = rear = count = 0` |
| Using modulo with negative numbers | Wrap produces negative index | Always ensure indices are non-negative |
| Growing without reordering | Elements appear out of order after resize | Copy in logical order starting from front |

---

## Quick Reference

```
STRUCTURE:
  array[N], front=0, rear=0, count=0

ENQUEUE(x):
  if count == N: return FULL
  array[rear] = x
  rear = (rear + 1) % N
  count++

DEQUEUE():
  if count == 0: return EMPTY
  x = array[front]
  front = (front + 1) % N
  count--
  return x

CONDITIONS:
  empty : count == 0
  full  : count == N
  size  : count

POWER-OF-TWO OPTIMISATION:
  If N is a power of 2: replace (i+1) % N with (i+1) & (N-1)
  Bitwise AND is faster than integer division on modern CPUs.
```

Memory aid: **"Equal means empty, front catches rear when full."**

---

## Related Pages

- [[adsa-queues]] — FIFO principle, linear array queue, BFS, round robin
- [[adsa-deque]] — double-ended queue, extends the circular concept
- [[adsa-stacks]] — LIFO counterpart

#adsa
