# B+ Trees

**Summary**: A B-tree variant where all data records live exclusively in leaf nodes, internal nodes serve as a pure routing index, and leaves are linked into a sorted list — giving O(log n) point queries and O(log n + k) range scans.

**Pre-Req**: [[adsa-b-trees]]

**Sources**: Session-built (no raw file).

**Last updated**: 2026-06-15

---

## What Is a B+ Tree?

A B+ tree is a modification of the [[adsa-b-trees|B-tree]] that splits responsibility between two node types:

- **Internal (index) nodes**: hold only keys and child pointers. Keys act as separators/routing guides — no data records live here.
- **Leaf nodes**: hold all key–data pairs. Every leaf also carries a *next-leaf pointer* so all leaves form a singly-linked sorted list.

Because data lives only at leaves, **every search reaches a leaf**. This costs one extra level of traversal compared to a B-tree (which can terminate at an internal node), but it enables range queries that B-trees cannot do efficiently.

---

## Structure

```
Internal node (routing only):
  ┌────┬────┬────┐
  │ k1 │ k2 │ k3 │     keys k1 < k2 < k3
  └────┴────┴────┘
  ↙    ↓    ↓    ↘
 c0   c1   c2   c3    all keys in ci satisfy: k(i-1) ≤ key < k(i)

Leaf node (data here):
  ┌──────┬──────┬──────┬──────┐
  │ k1,d1│ k2,d2│ k3,d3│  →   │     keys + data + pointer to next leaf
  └──────┴──────┴──────┴──────┘
```

**Separator key convention**: an internal key k means all keys in the right subtree are ≥ k. Duplicates of leaf keys appear in internal nodes as routing guides — they do not mean those records are stored there.

---

## B-Tree vs B+ Tree

| Property | B-Tree | B+ Tree |
|----------|--------|---------|
| Data location | Any node (internal or leaf) | Leaves only |
| Search terminates at | First node containing the key | Always a leaf |
| Internal node keys | Unique — key stored once, at one node | Copies — key may appear in internal node AND leaf |
| Range query | Must traverse tree repeatedly | Walk linked leaf list: O(log n + k) |
| Leaf linking | No | Yes — sorted linked list |
| Fan-out per internal node | Lower (keys carry data payload) | Higher (internal nodes hold only small routing keys) |

In practice the higher fan-out of B+ trees means one fewer level of I/O for the same dataset, and the linked leaf list makes range queries trivially fast.

---

## The Linked Leaf Layer

All leaf nodes are connected in key order via next-leaf pointers:

```
[2|3] → [5|7] → [11|17] → [19|23] → [29|31] → NIL
```

**Range query** for all keys in [7, 23]:
1. Walk the index tree to find the leaf containing 7 — O(log n) disk reads
2. Walk right along the linked list, collecting 7, 11, 17, 19, 23 — O(k) reads
3. Stop when you pass the upper bound

Total: **O(log n + k)**, where k is the number of results. A B-tree would need O(k log n) because each result requires re-traversing from the root.

This is why databases overwhelmingly prefer B+ trees for range-heavy workloads (e.g., `WHERE date BETWEEN x AND y`).

---

## Parameters

This wiki uses **order m**:

| Parameter | Value |
|-----------|-------|
| Max children per internal node | m |
| Max keys per internal node | m−1 |
| Max key–data pairs per leaf | m−1 |
| Min children per non-root internal node | ⌈m/2⌉ |
| Min keys per non-root internal node | ⌈m/2⌉ − 1 |
| Min key–data pairs per leaf | ⌈(m−1)/2⌉ |

---

## Real-World Uses

- **MySQL InnoDB**: primary key index is always a B+ tree; data rows are the leaf entries; secondary indexes point to primary keys
- **PostgreSQL**: default index type; stores (key, TID) pairs at leaves
- **SQLite**: table rows stored in B+ tree; leaves hold entire row records
- **NTFS**: `$INDEX_ALLOCATION` attribute for directory lookup
- **Oracle DB**: all table indexes use B+ trees by default

---

## Topic Map

| Page | What it covers |
|------|----------------|
| [[adsa-b-plus-tree-operations]] | Search, insert (leaf split copy-up vs internal split push-up), delete (redistribute/merge), insert dry run inserting 10 keys, delete dry run |

#adsa
