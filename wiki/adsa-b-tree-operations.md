# B-Tree Operations

**Summary**: Search, insert, and delete on a B-tree — insert proactively splits full nodes on the way down; delete proactively fills thin nodes on the way down; both maintain all B-tree properties without a second upward pass.

**Pre-Req**: [[adsa-b-trees]]

**Sources**: Session-built (no raw file).

**Last updated**: 2026-06-15

---

## Notation

All dry runs use **minimum degree t = 2** (a 2-3-4 tree):
- Min keys per non-root node: t−1 = **1**
- Max keys per node: 2t−1 = **3** (a node with 3 keys is *full*)
- Min children per non-root internal node: t = **2**
- Max children per internal node: 2t = **4**

---

## Search

Identical to BST search except each node holds multiple keys, so you scan across keys within the node before choosing a child pointer.

```
B-Tree-Search(x, k):
    i = 1
    while i ≤ x.n and k > x.key[i]:
        i = i + 1
    if i ≤ x.n and k == x.key[i]:
        return (x, i)              # found
    elif x.leaf:
        return NIL                 # not present
    else:
        disk-read(x.child[i])
        return B-Tree-Search(x.child[i], k)
```

**Complexity**: O(t · log_t n) — at most log_t n+1 disk reads, each scanning ≤ 2t−1 keys in O(t).

---

## Insert

**Strategy: proactive (top-down) splitting.**

Before descending into a child, check if that child is full. If it is, split it immediately. This guarantees the parent always has room to absorb the median key promoted by any split below.

### Split a Full Child

Split full child y (the i-th child of parent x) into two nodes:

1. Create new node z
2. Move the right half of y's keys to z: keys y.key[t+1 … 2t−1]
3. If y is not a leaf, move y's right children to z: children y.child[t+1 … 2t]
4. The median key y.key[t] rises into parent x at position i
5. x gains a new child pointer z at position i+1
6. y is left with t−1 keys; z has t−1 keys; both are valid

```
Before split (t=2, full child has 3 keys):

       parent:   [ ... | 20 | ... ]
                         ↓
       full child y:   [5 | 10 | 15]

After split — median 10 rises:

       parent:   [ ... | 10 | 20 | ... ]
                        ↙         ↘
                      [5]         [15]
```

### Insert Algorithm

```
B-Tree-Insert(T, k):
    r = T.root
    if r.n == 2t-1:                   # root is full — tree grows taller here
        s = new-node()
        T.root = s
        s.leaf = false
        s.n = 0
        s.child[1] = r
        B-Tree-Split-Child(s, 1)      # split old root, s gets median
        B-Tree-Insert-Nonfull(s, k)
    else:
        B-Tree-Insert-Nonfull(r, k)

B-Tree-Insert-Nonfull(x, k):
    i = x.n
    if x.leaf:
        shift keys right to make room, insert k in sorted position
    else:
        find i so x.key[i] < k ≤ x.key[i+1]
        disk-read(x.child[i+1])
        if x.child[i+1].n == 2t-1:           # child is full — split before descending
            B-Tree-Split-Child(x, i+1)
            if k > x.key[i+1]: i = i + 1     # figure out which of the two halves to enter
        B-Tree-Insert-Nonfull(x.child[i+1], k)
```

**Complexity**: O(t · log_t n) — O(log_t n) levels, O(t) work per split.

---

## Insert Dry Run (t = 2)

Insert keys in order: **3, 7, 1, 5, 11, 17, 13**

**Insert 3** — root is empty leaf, place directly:
```
[3]
```

**Insert 7** — leaf not full, insert:
```
[3 | 7]
```

**Insert 1** — leaf not full, insert in sorted position:
```
[1 | 3 | 7]      ← full (3 keys = 2t−1)
```

**Insert 5** — root is full, split it before anything else:
- Median = 3, left child = [1], right child = [7], new root = [3]
- Now insert 5: 5 > 3 → go to right child [7]; not full → insert:
```
        [3]
       /    \
     [1]   [5 | 7]
```

**Insert 11** — root [3] not full; 11 > 3 → right child [5|7]; not full → insert:
```
        [3]
       /    \
     [1]   [5 | 7 | 11]      ← full
```

**Insert 17** — root [3] not full; 17 > 3 → right child [5|7|11] is full — split before descending:
- Median = 7 rises into root → root becomes [3|7]
- Left half [5], right half [11]
- 17 > 7 → descend into [11]; not full → insert:
```
          [3 | 7]
         /   |   \
       [1]  [5]  [11 | 17]
```

**Insert 13** — root [3|7] not full; 13 > 7 → right child [11|17]; not full → insert in sorted position:
```
          [3 | 7]
         /   |   \
       [1]  [5]  [11 | 13 | 17]      ← full
```

Final tree: height 1, all leaves at depth 1, all properties satisfied. ✓

---

## Delete

Delete is more complex because removing a key may cause a node to fall below t−1 keys (underflow).

**Strategy: proactive (top-down) filling.**

Before descending into a child with only t−1 keys (the minimum), fill it first — either borrow from a sibling or merge with a sibling. This guarantees every node you touch during descent can afford to give up one key.

### 3 Cases

**Case 1 — k is in a leaf node x with ≥ t keys:**
Delete k directly. Done.

---

**Case 2 — k is in an internal node x:**

- **2a**: Left child y (precedes k) has ≥ t keys → find in-order predecessor k' (rightmost key in y's subtree) → replace k with k' → delete k' from y
- **2b**: Right child z (follows k) has ≥ t keys → find in-order successor k' (leftmost key in z's subtree) → replace k with k' → delete k' from z
- **2c**: Both y and z have exactly t−1 keys → merge: pull k down from x, concatenate into y: [y's keys | k | z's keys]. y now has 2t−1 keys. Delete k from the merged node. x loses one key and one child.

---

**Case 3 — k is not in current node x; we need to descend to child x.child[i], which has only t−1 keys:**

Fill x.child[i] first, then proceed:

- **3a**: An immediate sibling (left or right) has ≥ t keys → rotate: pull the separator key from x down into x.child[i], push the sibling's nearest key up to x as the new separator. x.child[i] now has t keys.
- **3b**: Both immediate siblings have only t−1 keys → merge x.child[i] with one sibling and the separator between them from x. x loses one key and one child.

---

**Complexity**: O(t · log_t n).

**Height shrinks** only when merges cascade all the way to the root and the root ends up with 0 keys — its sole child becomes the new root. The tree shrinks from the top.

---

## Delete Dry Run (t = 2)

Starting tree (the result of the insert dry run):
```
          [3 | 7]
         /   |   \
       [1]  [5]  [11 | 13 | 17]
```

---

**Delete 5** — 5 is in middle leaf [5], which has t−1=1 key (minimum — Case 3).

Descending from root [3|7] to middle child [5]:
- Left sibling [1] has 1 key — cannot lend (t−1 = minimum)
- Right sibling [11|13|17] has 3 keys — can lend (≥ t)

**Case 3a (borrow from right sibling):**
- Separator 7 (between middle and right children in root) moves down into middle child: [5|7]
- Leftmost key of right sibling, 11, moves up to root to replace 7
- Root: [3|11], right child: [13|17]

Middle child is now [5|7], has ≥ t keys. Delete 5:
```
          [3 | 11]
         /    |    \
       [1]  [7]  [13 | 17]
```

---

**Delete 1** — 1 is in left leaf [1], which has 1 key (minimum — Case 3).

Descending from root [3|11] to left child [1]:
- No left sibling
- Right sibling [7] has 1 key — cannot lend (minimum)

**Case 3b (merge):**
- Merge left child [1], separator 3 from root, right sibling [7] → merged node: [1|3|7]
- Root loses key 3 and one child pointer: root becomes [11]
- Two children remain: [1|3|7] and [13|17]

Now descend into [1|3|7] (has ≥ t keys). Delete 1:
```
          [11]
         /    \
     [3 | 7]  [13 | 17]
```

Root still has 1 key — valid. ✓

---

## Strategy Summary

| | Insert | Delete |
|--|--------|--------|
| **Pre-emptive action** | Split full nodes before descending | Fill thin nodes before descending |
| **Why** | Ensures parent has room for a promoted median | Ensures child can afford to give up a key |
| **Pass direction** | Single top-down pass | Single top-down pass |
| **Tree grows taller** | Only when root is split | Never during insert |
| **Tree shrinks shorter** | Never during delete | Only when merge reaches root |

Both operations complete in a single top-down pass — O(log_t n) disk reads — with no second upward pass needed.

#adsa
