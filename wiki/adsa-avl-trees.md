# AVL Trees

**Summary**: A self-balancing BST where every node's left and right subtrees differ in height by at most 1, guaranteeing O(log n) for search, insert, and delete regardless of insertion order.

**Pre-Req**: [[adsa-binary-trees]], [[adsa-tree-traversal]]

**Sources**: Session-built (no raw file).

**Last updated**: 2026-06-15

---

## Why AVL Trees Exist

A plain BST degenerates when data is inserted in sorted order. Every new node goes to the right, producing a linked list with O(n) search:

```
Insert 10, 20, 30, 40:

10
  \
   20
     \
      30
        \
         40    ← height = n−1, search is O(n)
```

AVL trees fix this by enforcing a **height balance invariant** after every insert and delete, keeping the tree's height bounded at O(log n).

Named after inventors Adelson-Velsky and Landis (1962) — the first self-balancing BST ever published. (source: session)

---

## The Balance Factor

Every node in an AVL tree carries a **balance factor (BF)**:

```
BF(node) = height(left subtree) − height(right subtree)
```

**Convention**: height of a null pointer = −1. A leaf node has height = 0, BF = 0.

| BF value | Meaning | Action |
|----------|---------|--------|
| −1, 0, +1 | Balanced | None |
| +2 | Left-heavy by one too many | Rebalance (rotation) |
| −2 | Right-heavy by one too many | Rebalance (rotation) |

A BF of ±2 is the only case that triggers a fix. After every insert or delete, BFs are recomputed bottom-up from the affected node to the root.

---

## Height Guarantee

An AVL tree with n nodes has height at most **1.44 × log₂(n)**.

This bound comes from the **Fibonacci AVL tree** — the sparsest possible AVL tree of a given height h. Its minimum node count follows the recurrence:

```
N(h) = N(h−1) + N(h−2) + 1     (one root + two minimal subtrees)
N(−1) = 0,  N(0) = 1
```

This is the Fibonacci recurrence, so N(h) grows like φ^h. Inverting: h ≤ 1.44 log₂(n+2). In practice, random AVL trees have height very close to log₂(n).

---

## Comparison: AVL vs Red-Black Tree

| Property | AVL Tree | Red-Black Tree |
|----------|----------|----------------|
| Balance condition | Strict: BF ∈ {−1, 0, +1} | Loose: longest path ≤ 2× shortest |
| Max height | 1.44 log₂ n | 2 log₂(n+1) |
| Lookup speed | Faster (shorter tree) | Slightly slower |
| Insert/delete rotations | Up to O(log n) | At most 2–3 |
| Best for | Read-heavy workloads | Write-heavy workloads |
| Used in | Some DBs, embedded systems | C++ `std::map`, Java `TreeMap` |

---

## Topic Map

| Page | What it covers |
|------|----------------|
| [[adsa-avl-rotations]] | The 4 rotation cases (LL, RR, LR, RL) with ASCII diagrams and pseudocode |
| [[adsa-avl-operations]] | Insert and delete algorithms with full step-by-step dry runs and Python code |

---

## Learning Roadmap

```
Binary Trees → BST → AVL Tree → Red-Black Tree
(structure)   (order)  (strict balance)  (loose balance, faster writes)
```

Master rotations before operations — every AVL operation is just BST + rotation logic.

#adsa
