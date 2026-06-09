# XOR Linked List

**Summary**: A memory-efficient doubly linked list that stores a single `npx` pointer per node instead of separate `prev` and `next` pointers, using bitwise XOR of the two addresses.

**Pre-Req**: [[adsa-arrays]], basic pointer arithmetic, bitwise XOR

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-04

---

## The Problem It Solves

A standard doubly linked list stores two pointers per node (`prev` and `next`). An XOR linked list collapses both into one field by exploiting the reversibility of XOR.

---

## The npx Value

Each node holds one field: `npx = id(prev) XOR id(next)`

In Python, `id()` gives the memory address of an object — the equivalent of a pointer.

| Position | prev     | next     | npx value              |
|----------|----------|----------|------------------------|
| Head     | `None`   | next     | `0 ^ id(next) = id(next)` |
| Middle   | prev     | next     | `id(prev) ^ id(next)`  |
| Tail     | prev     | `None`   | `id(prev) ^ 0 = id(prev)` |

The key property: if you know one neighbour, you can recover the other with a single XOR.

```
id(next) = curr.npx ^ id(prev)
id(prev) = curr.npx ^ id(next)
```

---

## Node Structure

```python
import ctypes

class Node:
    def __init__(self, data):
        self.data = data
        self.npx = 0  # stores XOR of id(prev) and id(next)

def get_node(address):
    if address == 0:
        return None
    return ctypes.cast(address, ctypes.py_object).value
```

`ctypes.cast(..., ctypes.py_object).value` converts a raw memory address back to the Python object — this is how we recover a node from an XOR'd address.

---

## Insertion (at head)

```python
def insert(head, data):
    node = Node(data)
    node.npx = id(head)          # prev=None (0), next=head → 0 ^ id(head)

    if head:
        # old head's npx was: 0 ^ id(old_next)
        # update it to: id(node) ^ id(old_next)
        head.npx = id(node) ^ head.npx

    return node                  # new head
```

---

## Forward Traversal

Start with `prev = None`, `curr = head`. Each step recovers `next` via XOR.

```python
def traverse_forward(head):
    prev = None
    curr = head
    while curr:
        print(curr.data, end=" ")
        prev_id = id(prev) if prev else 0
        next_id = curr.npx ^ prev_id
        prev = curr
        curr = get_node(next_id)
```

Logic: `curr.npx = id(prev) ^ id(next)`, so `id(next) = curr.npx ^ id(prev)`.

---

## Backward Traversal

Mirror of forward — seed `next = None` and work from the tail.

```python
def traverse_backward(tail):
    nxt = None
    curr = tail
    while curr:
        print(curr.data, end=" ")
        next_id = id(nxt) if nxt else 0
        prev_id = curr.npx ^ next_id
        nxt = curr
        curr = get_node(prev_id)
```

The XOR relationship is symmetric — direction is just a matter of which neighbour you already know.

---

## Full Example

```python
head = None
for val in [1, 2, 3, 4]:
    head = insert(head, val)
# list is now: 4 <-> 3 <-> 2 <-> 1

traverse_forward(head)   # 4 3 2 1

# walk to tail for backward traversal
prev, curr = None, head
while curr:
    prev_id = id(prev) if prev else 0
    next_id = curr.npx ^ prev_id
    prev, curr = curr, get_node(next_id)
tail = prev

traverse_backward(tail)  # 1 2 3 4
```

---

## Complexity

| Operation         | Time | Space vs doubly linked list  |
|-------------------|------|------------------------------|
| Insert at head    | O(1) | saves one pointer per node   |
| Forward traverse  | O(n) | —                            |
| Backward traverse | O(n) | —                            |
| Random access     | O(n) | —                            |

---

## Limitations

- Python's garbage collector can collect nodes if no other reference holds them — you must keep explicit references (e.g. a list) to prevent this.
- `ctypes` access to raw addresses is unsafe; if the GC moves or collects an object, the address becomes invalid.
- Rarely used in practice — the memory saving is niche and the complexity cost is real.
- Requires a `tail` pointer to traverse backwards without a full forward pass first.

#adsa
