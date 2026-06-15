# B+ Tree Operations

**Summary**: Search always reaches a leaf; insert splits leaves by copying the first right-leaf key up, and splits internal nodes by pushing the middle key up; delete merges or redistributes from a sibling when a leaf underflows.

**Pre-Req**: [[adsa-b-plus-trees]]

**Sources**: Session-built (no raw file).

**Last updated**: 2026-06-15

---

## Notation

All dry runs use **order m = 4**:
- Max keys per node (internal or leaf): m−1 = **3**
- Max children per internal node: m = **4**
- Min leaf entries: ⌈(m−1)/2⌉ = **2**
- Min children per non-root internal node: ⌈m/2⌉ = **2**
- A node with 3 keys is *full*

---

## Search

Always descend to a leaf. At each internal node scan keys to pick the right child pointer; at the leaf scan for the exact key.

```
B+Tree-Search(node, k):
    if node is leaf:
        scan node for key k
        return record if found, else NIL
    i = first index where node.key[i] > k
    return B+Tree-Search(node.child[i-1], k)
```

Separator key convention: child[0] holds keys < key[1]; child[i] holds keys ≥ key[i] and < key[i+1].

**Complexity**: O(log_m n) disk reads, always exactly h+1 where h is the tree height.

---

## Insert

Two split behaviours depending on which level overflows:

### Leaf Split — copy up

When a leaf overflows to m keys, split it into two leaves and **copy** the first key of the right leaf up to the parent. The key stays in the leaf because data must remain there.

```
Before (m=4, leaf overflows to 4 entries):
  leaf: [2 | 3 | 5 | 7]   ← temporarily 4 entries

After split:
  left leaf:  [2 | 3]
  right leaf: [5 | 7]   ← first key 5 is COPIED up, stays here too
  parent receives: 5
```

### Internal Node Split — push up

When an internal node overflows to m keys, split it and **push** the middle key up to the parent. The middle key does not stay at this level — it becomes a separator above.

```
Before (m=4, internal overflows to 4 keys):
  internal: [5 | 11 | 19 | 29]   ← 4 keys, 5 children

Middle key = 11 (position ⌊(m+1)/2⌋ = 2, 1-indexed) is PUSHED up.
After split:
  left internal:  [5]   (children: left two)
  pushed up:      11
  right internal: [19 | 29]   (children: right three)
```

### Insert Algorithm

```
B+Tree-Insert(T, k, data):
    if root is full:
        create new root s
        s.child[1] = old root
        split old root
    insert-nonfull(root, k, data)

insert-nonfull(node, k, data):
    if node is leaf:
        insert (k, data) in sorted position
        if leaf overflows: leaf-split(node)
    else:
        find child c to descend into
        if c is full: split c first
        insert-nonfull(c, k, data)
```

**Complexity**: O(m · log_m n) — O(log_m n) levels, O(m) work per split.

---

## Insert Dry Run (m = 4)

Insert in order: **2, 3, 5, 7, 11, 17, 19, 23, 29, 31**

**Insert 2, 3, 5** — single leaf, not yet full:
```
[2 | 3 | 5]
```

**Insert 7** — leaf hits 4 entries, split (copy 5 up):
```
      [5]
     /    \
  [2|3]  [5|7]
  ↗              (leaf chain: [2|3] → [5|7])
```

**Insert 11** — 11≥5, right leaf [5|7] not full:
```
      [5]
     /    \
  [2|3]  [5|7|11]   ← full
```

**Insert 17** — 17≥5, right leaf full → split (copy 11 up):
```
         [5|11]
        /   |   \
    [2|3] [5|7] [11|17]
    (leaf chain: [2|3] → [5|7] → [11|17])
```

**Insert 19** — 19≥11, rightmost leaf [11|17] not full:
```
         [5|11]
        /   |   \
    [2|3] [5|7] [11|17|19]   ← full
```

**Insert 23** — 23≥11, rightmost leaf full → split (copy 19 up):
- Root [5|11] receives 19 → becomes [5|11|19] (full)

```
            [5|11|19]
           /   |  |   \
       [2|3] [5|7] [11|17] [19|23]
       (leaf chain: [2|3]→[5|7]→[11|17]→[19|23])
```

**Insert 29** — 29≥19, rightmost leaf not full:
```
            [5|11|19]
           /   |  |   \
       [2|3] [5|7] [11|17] [19|23|29]   ← full
```

**Insert 31** — 31≥19, rightmost leaf full → split (copy 29 up):
- Root [5|11|19] receives 29 → temporarily [5|11|19|29] — root is now full too → **internal split**

Internal split of [5|11|19|29] with 5 children:
- Middle key at position 2 (1-indexed) = **11** is PUSHED up to new root
- Left internal: [5], children: [[2|3], [5|7]]
- Right internal: [19|29], children: [[11|17], [19|23], [29|31]]
- New root: [11]

```
                     [11]
                   /       \
              [5]             [19|29]
             /   \           /   |   \
         [2|3] [5|7]   [11|17] [19|23] [29|31]

Leaf chain: [2|3]→[5|7]→[11|17]→[19|23]→[29|31]→NIL
```

Tree height = 2, all leaves at same depth, all properties satisfied. ✓

Note: 11 appears as both the root separator and the first entry of leaf [11|17]. This is normal in a B+ tree — the root's 11 is a routing copy, the leaf's 11 carries the actual data record.

---

## Delete

When a leaf underflows (drops below ⌈(m−1)/2⌉ = 2 entries):

1. **Redistribute** — borrow from an adjacent sibling (under the same parent) that has ≥ min+1 entries:
   - Borrow from left: move sibling's rightmost entry to current leaf; update the separator in parent to the current leaf's new first key
   - Borrow from right: move sibling's leftmost entry to current leaf; update the separator in parent to sibling's new first key

2. **Merge** — if no sibling can lend: merge current leaf with one sibling; the separator between them is removed from the parent; the merge may cause the parent to underflow (recurse up)

Internal node underflow is handled the same way — borrow a separator from parent + a child from sibling, or merge two internal nodes with the separator between them (push down, not copy).

---

## Delete Dry Run (m = 4)

Starting tree:
```
                     [11]
                   /       \
              [5]             [19|29]
             /   \           /   |   \
         [2|3] [5|7]   [11|17] [19|23] [29|31]
```

**Delete 11** — 11 is in leaf [11|17]. After deletion: [17] — underflow (needs ≥ 2).

Siblings of [11|17] under parent [19|29]:
- No left sibling (it is the leftmost child of [19|29])
- Right sibling: [19|23], has 2 entries — minimum, cannot lend

**Merge** [17] with right sibling [19|23]:
- Merged leaf: [17|19|23]
- Separator 19 (between them in parent [19|29]) is removed from parent → parent becomes [29]
- Link the merged leaf into the chain: … → [5|7] → [17|19|23] → [29|31]

Parent [29] now has 1 key and 2 children — valid (min = 1 key). Root [11] unchanged.

```
                     [11]
                   /       \
              [5]             [29]
             /   \           /    \
         [2|3] [5|7]   [17|19|23] [29|31]

Leaf chain: [2|3]→[5|7]→[17|19|23]→[29|31]→NIL
```

Note: the root still contains 11 as a separator even though the leaf entry 11 was deleted. This is correct — 11 still correctly routes traffic: keys < 11 go left, keys ≥ 11 go right. Some implementations update stale separators eagerly; others leave them since correctness is preserved.

---

## Range Query

Once insert and delete are understood, the core advantage of a B+ tree over a B-tree is clear:

```
Range-Query(T, lo, hi):
    leaf = B+Tree-Search(T, lo)      # O(log_m n) disk reads
    results = []
    while leaf != NIL and leaf.key[0] ≤ hi:
        for (k, data) in leaf:
            if lo ≤ k ≤ hi: results.append(data)
        leaf = leaf.next              # O(1) — pointer already in hand
    return results
```

Total cost: **O(log_m n + k)** disk reads for k results. A B-tree would need O(k log_m n) since there is no leaf chain and each result requires re-traversal from the root.

This is why every major relational database defaults to B+ trees for indexes.

---

## Leaf Split vs Internal Split Summary

| | Leaf split | Internal split |
|--|------------|----------------|
| **Trigger** | Leaf overflows to m entries | Internal node overflows to m keys |
| **Action** | Split into two leaves of equal size | Split at middle key |
| **Key movement** | First key of right leaf **copied** up — stays in leaf | Middle key **pushed** up — removed from this level |
| **Why different** | Data must remain in leaves | Internal keys are routing guides only — no data to preserve |

#adsa
