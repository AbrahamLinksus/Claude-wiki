# AVL Rotations

**Summary**: The 4 structural fixes (LL, RR, LR, RL) that restore balance after an AVL insert or delete. Every rotation preserves the BST ordering property while adjusting the tree's shape.

**Pre-Req**: [[adsa-avl-trees]]

**Sources**: Session-built (no raw file).

**Last updated**: 2026-06-15

---

## What a Rotation Does

A rotation re-links three nodes and up to four subtrees to reduce the height of a tall side by 1 and increase the other by 1. Crucially, in-order traversal order is identical before and after — the BST property is fully preserved.

There are two primitive rotations:
- **Right Rotate** — pulls the left child up
- **Left Rotate** — pulls the right child up

The four AVL cases use these primitives: two use one rotation, two use two rotations (double rotations).

---

## How to Detect Which Case

When a node `z` becomes unbalanced (BF = ±2), look at the sign of its heavier child's BF:

```
BF(z) = +2  (left-heavy):
    BF(z.left) >= 0  →  LL case  →  single right rotate at z
    BF(z.left) <  0  →  LR case  →  left rotate at z.left, then right rotate at z

BF(z) = -2  (right-heavy):
    BF(z.right) <= 0  →  RR case  →  single left rotate at z
    BF(z.right) >  0  →  RL case  →  right rotate at z.right, then left rotate at z
```

Memory aid: **same-sign BF = single rotation. Opposite-sign BF = double rotation.**

---

## Case 1 — LL (Right Rotation)

**Trigger**: `BF(z) = +2` and `BF(z.left) = +1` (or 0 in delete).
The new node was inserted in the **left subtree of the left child**.

```
Before:                         After Right Rotate at z:

        z  (BF=+2)                      y  (BF=0)
       / \                             / \
      y   T4  (BF=+1)               x     z
     / \                           / \   / \
    x   T3                        T1  T2 T3  T4
   / \
  T1  T2
```

**Pseudocode**:
```
RIGHT-ROTATE(z):
    y  = z.left
    T3 = y.right          # T3 will become z's new left child

    y.right = z
    z.left  = T3

    update_height(z)      # z is now lower — update first
    update_height(y)      # y is now higher

    return y              # y is the new root of this subtree
```

**What moves**: z drops down to become y's right child. T3 (y's old right subtree) re-attaches as z's new left subtree. Heights of z and y are updated after the swap.

---

## Case 2 — RR (Left Rotation)

**Trigger**: `BF(z) = -2` and `BF(z.right) = -1` (or 0 in delete).
The new node was inserted in the **right subtree of the right child**. Mirror of LL.

```
Before:                         After Left Rotate at z:

  z  (BF=-2)                          y  (BF=0)
 / \                                 / \
T1   y  (BF=-1)                     z     x
    / \                            / \   / \
   T2   x                        T1  T2 T3  T4
       / \
      T3  T4
```

**Pseudocode**:
```
LEFT-ROTATE(z):
    y  = z.right
    T2 = y.left           # T2 will become z's new right child

    y.left  = z
    z.right = T2

    update_height(z)
    update_height(y)

    return y
```

---

## Case 3 — LR (Left-Right Double Rotation)

**Trigger**: `BF(z) = +2` and `BF(z.left) = -1`.
The new node was inserted in the **right subtree of the left child**. A single right rotation at z does not fix this — the tree would just "lean" in a different direction.

Fix: first left-rotate the left child (bringing x up), then right-rotate z.

**Step 1 — Left Rotate at y (z's left child):**
```
Before:                        After left rotate at y:

      z (BF=+2)                       z (BF=+2)
     / \                             / \
    y   T4  (BF=-1)                 x   T4
   / \                             / \
  T1   x                          y   T3
      / \                        / \
     T2  T3                     T1  T2
```

**Step 2 — Right Rotate at z:**
```
Before:                        After right rotate at z:

      z (BF=+2)                        x (BF=0)
     / \                              / \
    x   T4                           y   z
   / \                              / \ / \
  y   T3                          T1 T2 T3 T4
 / \
T1  T2
```

**Pseudocode**:
```
LR-FIX(z):
    z.left = LEFT-ROTATE(z.left)
    return RIGHT-ROTATE(z)
```

**Why x rises to the top**: x is between y and z in BST order (y < x < z), so x is the natural median — it correctly becomes the new root.

---

## Case 4 — RL (Right-Left Double Rotation)

**Trigger**: `BF(z) = -2` and `BF(z.right) = +1`.
The new node was inserted in the **left subtree of the right child**. Mirror of LR.

Fix: first right-rotate the right child, then left-rotate z.

**Step 1 — Right Rotate at y (z's right child):**
```
Before:                        After right rotate at y:

  z (BF=-2)                         z (BF=-2)
 / \                               / \
T1   y  (BF=+1)                   T1   x
    / \                               / \
   x   T4                            T2   y
  / \                                    / \
 T2  T3                                T3  T4
```

**Step 2 — Left Rotate at z:**
```
Before:                        After left rotate at z:

  z (BF=-2)                          x (BF=0)
 / \                                / \
T1   x                             z   y
    / \                           / \ / \
   T2   y                       T1 T2 T3 T4
       / \
      T3  T4
```

**Pseudocode**:
```
RL-FIX(z):
    z.right = RIGHT-ROTATE(z.right)
    return LEFT-ROTATE(z)
```

---

## All Four Cases — Quick Reference

| Case | BF(z) | BF(child) | Fix | Rotations |
|------|--------|-----------|-----|-----------|
| LL | +2 | ≥ 0 | Right rotate at z | 1 |
| RR | −2 | ≤ 0 | Left rotate at z | 1 |
| LR | +2 | < 0 | Left rotate at z.left, then right rotate at z | 2 |
| RL | −2 | > 0 | Right rotate at z.right, then left rotate at z | 2 |

---

## Complexity of Rotations

| Operation | Time | Space |
|-----------|------|-------|
| Single rotation | O(1) | O(1) |
| Double rotation | O(1) | O(1) |

Rotations only re-link pointers and update heights of 2–3 nodes. They are constant-time regardless of subtree sizes.

---

## Key Invariants After Any Rotation

1. BST ordering is preserved — in-order traversal gives the same sorted sequence.
2. Height of the subtree that was rooted at z is the same or smaller by 1 than before.
3. Every node in the rotated subtree now has BF ∈ {−1, 0, +1}.

See [[adsa-avl-operations]] for how these rotations are triggered inside insert and delete.

#adsa
