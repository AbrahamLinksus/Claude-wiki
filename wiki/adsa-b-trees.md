# B-Trees

**Summary**: A self-balancing m-way search tree that generalises BSTs to multiple keys per node, designed to minimise disk I/O by fitting each node into one disk block and keeping height at O(log_t n).

**Sources**: Session-built (no raw file).

**Last updated**: 2026-06-15

---

## Why B-Trees?

Binary trees (AVL, RBT) hold one key per node and achieve O(log₂ n) height — about 30 levels for 10⁹ nodes. In RAM that is fine. On disk each pointer hop is a separate I/O read: 30 × 5 ms = 150 ms per lookup. Completely unusable for a database.

B-trees pack many keys into each node so the tree is very wide and very flat. With 500 keys per node the same 10⁹ nodes need only log₅₀₀(10⁹) ≈ 3 levels — 3 disk reads instead of 30.

The core idea: **each node corresponds to one disk block** (typically 4 KB–16 KB). Pack as many keys as fit in that block, and let the fan-out collapse the height.

---

## Terminology

Two equivalent ways to parameterise a B-tree appear in textbooks:

| Term | Meaning |
|------|---------|
| **Minimum degree t** (CLRS) | Each non-root node has ≥ t−1 keys and ≤ 2t−1 keys |
| **Order m** | Each non-root node has ≥ ⌈m/2⌉−1 keys and ≤ m−1 keys |
| Relationship | m = 2t |

This wiki uses **minimum degree t** (CLRS convention).

**Special cases**:
- t = 2 → each node holds 1–3 keys, called a **2-3-4 tree**; isomorphic to a [[adsa-red-black-trees|Red-Black Tree]]
- t = 500 → nodes hold 499–999 keys; typical for a database index page

---

## B-Tree Properties

A valid B-tree of minimum degree t satisfies all six simultaneously:

1. **All leaves are at the same depth** — perfectly height-balanced
2. Every node stores its keys in **sorted order**
3. A non-root node has **≥ t−1 keys** — prevents underflow
4. Every node has **≤ 2t−1 keys** — prevents overflow; a node with exactly 2t−1 keys is called *full*
5. A non-leaf node with k keys has exactly **k+1 children**
6. **Subtree separation**: if a node has keys [k₁ … kₙ], all keys in child i satisfy k_{i-1} < key < k_i (BST property extended to multiple keys)

The root is special: it may have as few as 1 key (and 2 children) or, if the tree is a single leaf, 0 keys.

---

## Height

For a B-tree with n keys and minimum degree t:

```
height h ≤ log_t ⌈(n+1)/2⌉
```

With t = 500 and n = 10⁹:  h ≤ log₅₀₀(5 × 10⁸) ≈ 3 levels.

Each root-to-leaf path touches h+1 nodes, so at most **h+1 disk reads** are needed for any operation.

---

## B-Tree vs Other Trees

| Property | BST | AVL | RBT | B-Tree (t=500) |
|----------|-----|-----|-----|----------------|
| Keys per node | 1 | 1 | 1 | 499–999 |
| Height for 10⁹ nodes | up to 10⁹ | ~30 | ~60 | ~3 |
| Disk reads per lookup | ~30 | ~30 | ~60 | ~3 |
| Extra storage per node | none | height int | 1 color bit | none |
| Optimised for | RAM | RAM (reads) | RAM (writes) | Disk / SSD |

---

## Real-World Uses

- **MySQL InnoDB**: B+ tree (a B-tree variant where data lives only in leaves)
- **PostgreSQL**: default index type is B-tree
- **SQLite**: B-tree for table rows, B+ tree for indexes
- **NTFS / ext4 / HFS+**: directory trees stored as B-trees
- **MongoDB WiredTiger**: B-tree storage engine

---

## Connection to Red-Black Trees

A B-tree of minimum degree t = 2 (a 2-3-4 tree) is structurally isomorphic to a Red-Black Tree:

```
2-node (1 key, 2 children)  →  one Black node
3-node (2 keys, 3 children) →  one Black node + one Red child
4-node (3 keys, 4 children) →  one Black node + two Red children
```

The five RBT properties are consequences of this encoding. See [[adsa-red-black-trees]].

---

## Topic Map

| Page | What it covers |
|------|----------------|
| [[adsa-b-tree-operations]] | Search, insert with proactive splitting, delete with merge/redistribute; two full dry runs |

#adsa
