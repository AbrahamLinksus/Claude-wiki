# Red-Black Trees

**Summary**: A self-balancing BST that stores a color bit (Red/Black) per node and enforces 5 invariants to bound the height at 2 log₂(n+1), with at most 2–3 rotations per insert or delete.

**Pre-Req**: [[adsa-avl-trees]], [[adsa-avl-rotations]]

**Sources**: Session-built (no raw file).

**Last updated**: 2026-06-15

---

## What Is a Red-Black Tree?

A Red-Black Tree (RBT) is a BST where every node carries one extra bit of information: its color, either Red or Black. The color rules enforce an implicit structural balance: no root-to-leaf path can be more than twice as long as any other path. This keeps height at O(log n) while needing far fewer rotations per write than AVL.

RBTs are the dominant balanced BST in practice. C++ `std::map`/`std::set`, Java `TreeMap`/`TreeSet`, and the Linux kernel's process scheduler all use Red-Black Trees internally.

---

## The 5 Properties

Every valid Red-Black Tree must satisfy all five simultaneously:

| # | Property | Explanation |
|---|----------|-------------|
| 1 | Every node is Red or Black | Defines the color bit |
| 2 | The root is Black | Avoids ambiguity at the top of the tree |
| 3 | Every NIL leaf (sentinel) is Black | NIL pointers are treated as Black leaves, simplifying case analysis |
| 4 | If a node is Red, both its children are Black | No two consecutive Red nodes on any root-to-leaf path |
| 5 | For every node, all paths to descendant NIL leaves contain the same number of Black nodes | The **black-height invariant** — this is the core balance guarantee |

Properties 4 and 5 together ensure the tree cannot become severely unbalanced. Property 4 means a path can have at most alternating R-B-R-B nodes, making the longest path at most twice the shortest.

### NIL Sentinel
In implementation, a single shared sentinel node (colored Black) replaces all null pointers. This means every leaf node's left and right children point to `NIL` rather than Python `None`, eliminating null checks in all fix-up code.

---

## Black-Height

The **black-height** of a node x, written `bh(x)`, is the number of Black nodes on any path from x down to a NIL leaf, **not counting x itself**.

Property 5 guarantees `bh(x)` is identical on every downward path from x — so black-height is well-defined for every node.

```
bh(NIL sentinel) = 0
bh(Red leaf)     = 0   (both NIL children contribute 0)
bh(Black leaf)   = 0   (same — bh counts nodes below, not the node itself)
bh(root)         = number of Black nodes on any root-to-NIL path
```

---

## Height Guarantee

**Theorem**: An RBT with n internal nodes has height h ≤ 2 log₂(n+1).

**Proof in 2 steps:**

**Step 1** — A subtree with black-height bh contains at least 2^bh − 1 internal nodes.

By induction: a single NIL leaf has bh = 0 and 0 = 2⁰ − 1 nodes. For any internal node x with bh(x) = bh, each child has bh ≥ bh − 1 (it's bh − 1 if the child is Black, bh if the child is Red). By induction, each child's subtree has ≥ 2^(bh−1) − 1 nodes. Adding the root: total ≥ 2×(2^(bh−1) − 1) + 1 = 2^bh − 1. ■

**Step 2** — The root's black-height is at least h/2.

By property 4 (no consecutive Reds), on any path of length h, at least ⌈h/2⌉ nodes are Black. So bh(root) ≥ h/2.

**Combining:**
```
n ≥ 2^(h/2) − 1
n + 1 ≥ 2^(h/2)
log₂(n+1) ≥ h/2
h ≤ 2 log₂(n+1)   ■
```

In practice, RBTs have height roughly 1.7–2.0 × log₂ n vs AVL's 1.0–1.44 × log₂ n.

---

## AVL vs Red-Black Tree — Complete Comparison

| Property | AVL Tree | Red-Black Tree |
|----------|----------|----------------|
| Balance invariant | BF ∈ {−1, 0, +1} (strict) | No path > 2× another (loose) |
| Extra storage per node | Height integer | 1 color bit |
| Max height | 1.44 log₂ n | 2 log₂(n+1) |
| Lookup speed | Faster (shorter tree) | Slightly slower |
| Insert rotations | ≤ 1 (single or double) | ≤ 2 |
| Delete rotations | O(log n) worst case | ≤ 3 |
| Recoloring passes | None | O(log n) on insert, O(log n) on delete |
| Best workload | Read-heavy | Write-heavy |
| Real-world uses | Some DBs, embedded | `std::map`, `TreeMap`, Linux kernel |

AVL's stricter balance makes lookups slightly faster. RBT's looser balance means fewer structural changes per write — critical for workloads that insert and delete frequently.

---

## Topic Map

| Page | What it covers |
|------|----------------|
| [[adsa-rbt-insert]] | 3 fix-up cases (Uncle Red, Triangle, Line), full dry-run inserting 6 keys showing all cases, Python code |
| [[adsa-rbt-delete]] | Double-black concept, 4 fix-up cases, two dry runs, complete Python implementation |

---

## Isomorphism with 2-3-4 Trees

A 2-3-4 tree (B-tree of order 4) and a Red-Black Tree encode the same data. Every 2-3-4 node maps to a small cluster of RBT nodes:

```
2-node  →  one Black node
3-node  →  one Black node + one Red child
4-node  →  one Black node + two Red children
```

The 5 RBT properties are direct consequences of this encoding. Understanding the isomorphism makes the RBT rules feel less arbitrary — they are just B-tree structure expressed in binary form.

#adsa
