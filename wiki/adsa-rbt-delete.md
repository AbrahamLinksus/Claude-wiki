# RBT Delete

**Summary**: The hardest balanced-BST operation — BST delete followed by a fix-up that resolves a "double-black" violation using recoloring and at most 3 rotations.

**Pre-Req**: [[adsa-rbt-insert]]

**Sources**: Session-built (no raw file).

**Last updated**: 2026-06-15

---

## Why Delete Is Harder Than Insert

When you remove a **Black** node, some paths to NIL lose a Black node — property 5 is violated. The fix-up assigns the "extra black responsibility" to whoever replaced the deleted node. That replacement carries a conceptual **double-black** label: it must behave as two Black nodes to compensate, even if it is physically Red or even NIL.

If the deleted node was **Red**, no black-height changes and no fix-up is needed.

---

## BST Delete — Three Sub-Cases

Before fix-up, do a standard BST delete:

| Case | Condition | Action |
|------|-----------|--------|
| Leaf | No children | Remove node; replacement = NIL |
| One child | Exactly one non-NIL child | Splice out node; replacement = that child |
| Two children | Both children non-NIL | Replace node's key with **in-order successor** (leftmost node in right subtree), then delete that successor (which has at most one child) |

After splicing, let:
- `y` = the node actually removed from the tree (the original node, or the in-order successor in the two-child case)
- `x` = the node that takes y's place (y's child, possibly NIL sentinel)
- `y_original_color` = y's color before removal

If `y_original_color == BLACK` → run the double-black fix-up on `x`.
If `y_original_color == RED` → no violation. Done.

---

## Cases That Need No Fix-Up

These resolve without entering the fix-up loop:

1. **Deleted node is Red** — removing a Red node changes no Black-heights.
2. **Deleted node is Black, replacement is Red** — color the replacement Black. The Black count on that path is restored. Done.

---

## The Double-Black Fix-Up Loop

`x` starts as the replacement node carrying the double-black.
`w` = x's sibling (the other child of x's parent).

The loop terminates when:
- x becomes Red → color x Black. Done.
- x reaches the root → simply color root Black. Done.
- A Case 4 rotation fully resolves the violation.

All 4 cases below assume `x` is a **left** child. Mirror applies when x is a right child (swap left↔right, left-rotate↔right-rotate).

---

## Case 1 — Sibling w is Red

**Condition**: `w.color == RED`

w being Red means parent must be Black (otherwise red-red). Rotate to convert w to Black.

**Fix**: Left rotate at parent. Swap colors of parent and w.
```
       parent(B)                          w(B)
      /         \    Left rotate         /    \
     x(DB)      w(R)   at parent  →  parent(R)  D(B)
                / \                   /     \
               C(B)  D(B)           x(DB)   C(B)
```
After: parent is Red, x's new sibling = C (which is Black). Re-enter loop — x is now in Case 2, 3, or 4 with a Black sibling.

Case 1 never terminates the loop alone. It just transforms the problem.

---

## Case 2 — Sibling w is Black, Both w's Children are Black

**Condition**: `w.color == BLACK`, `w.left.color == BLACK`, `w.right.color == BLACK`

Neither subtree of w can absorb x's extra black. Transfer the problem upward.

**Fix**: Recolor w to Red. Move double-black to parent(x). Set x = parent.
```
       parent(?)                         parent(DB or resolved)
      /         \    Recolor             /         \
     x(DB)      w(B)    w→R   ──►      x(B)        w(R)
                / \                                  / \
              C(B) D(B)                            C(B) D(B)
```
- If parent was **Red** → parent absorbs the extra black, becomes Black. Loop ends. ✓
- If parent was **Black** → parent becomes double-black. Loop repeats with x = parent.

Case 2 is the only case that can propagate the fix-up upward. In the worst case it climbs to the root, giving O(log n) recolorings.

---

## Case 3 — Sibling w is Black, w's Near Child is Red, Far Child is Black

**Condition**: `w.color == BLACK`, `w.left.color == RED`, `w.right.color == BLACK`
(Near child = same side as x; far child = opposite side from x.)

w's "useful" Red child is on the wrong side. Rotate to move it to the far side.

**Fix**: Right rotate at w. Swap colors of w and w's left child.
```
       parent(?)                         parent(?)
      /         \    Right rotate        /         \
     x(DB)      w(B)    at w   ──►     x(DB)       C(B)
                / \                                   \
              C(R)  D(B)                              w(R)
                                                       \
                                                       D(B)
```
After: new sibling of x = C (Black), C's right child = old w (Red). This is now **Case 4**.

Case 3 always transitions to Case 4 immediately.

---

## Case 4 — Sibling w is Black, w's Far Child is Red

**Condition**: `w.color == BLACK`, `w.right.color == RED`

The far child can absorb the extra black after a rotation.

**Fix**: Left rotate at parent. Assign colors: w gets parent's old color, parent → Black, w's right child → Black.
```
       parent(c)                          w(c)
      /         \    Left rotate         /    \
     x(DB)      w(B)   at parent  → parent(B)   D(B)
                / \                   /
               C(?)  D(R)            x(B)   (double-black resolved)
```
`c` = parent's original color (could be Red or Black — w inherits it to preserve the Black count above).

Double-black is fully resolved. Set x = root to exit the loop.

---

## All 4 Cases — Quick Reference

| Case | w color | w.left | w.right | Fix | Leads to |
|------|---------|--------|---------|-----|----------|
| 1 | Red | — | — | Left rotate at parent, swap colors | Case 2/3/4 |
| 2 | Black | Black | Black | Recolor w→Red, move x up | Loop continues or ends |
| 3 | Black | Red | Black | Right rotate at w, swap colors | Case 4 |
| 4 | Black | — | Red | Left rotate at parent, recolor | Done |

---

## Dry Run 1 — Case 4 (Single Rotation)

**Build tree** by inserting 10, 5, 15, 20:

- Insert 10: root → 10(B)
- Insert 5: left of 10, Red. Parent Black → no violation.
- Insert 15: right of 10, Red. Uncle=5(R) → Case 1: recolor 5→B, 15→B, 10→R. Root→B.
  Wait — parent 10 is the root (Black). 15 is right of 10. No red-red since parent=10(B). No violation.

Let me redo:
- Insert 10: 10(B) root
- Insert 5: 5(R) left of 10. Parent=10(B). No violation.
- Insert 15: 15(R) right of 10. Parent=10(B). No violation.
- Insert 20: 20(R) right of 15. Parent=15(R) → red-red! Uncle=5(R). → Case 1: recolor 5→B, 15→B, 10→R. Root→B.

Tree after all inserts:
```
    10(B)
   /     \
  5(B)   15(B)
            \
            20(R)
```

**Delete 5(B)**:
5 is a leaf (no children). Remove it. y=5, y_color=Black, x=NIL sentinel, x is left child of parent=10.

x is double-black. w = x's sibling = 15(B). w.right = 20(R) — far child is Red → **Case 4**.

**Case 4 — Left rotate at parent=10:**
- w=15 inherits parent=10's color (B).
- parent=10 → Black.
- w.right=20 → Black.

Left rotate at 10:
- 15 is new root. 10 is 15's left child. 15's old left (NIL) becomes 10's right.

```
Before:                         After:
    10(B)                           15(B)
   /     \                         /     \
 NIL(DB)  15(B)                  10(B)  20(B)
              \                  /   \
              20(R)            NIL   NIL
```

Double-black resolved. x = root → loop ends. ✓

**Result:**
```
    15(B)
   /     \
  10(B)  20(B)
```

---

## Dry Run 2 — Case 2 (Recolor and Propagate)

**Starting tree:**
```
    10(B)
   /     \
  5(B)   15(B)
```

**Delete 5(B)**:
5 is a leaf. y=5, y_color=Black, x=NIL, x is left child of parent=10.

x is double-black. w=15(B). w.left=NIL(B), w.right=NIL(B) — both Black → **Case 2**.

**Case 2 — Recolor w=15 to Red. Move x up to parent=10:**
x = 10. parent was Black → 10 is now double-black. But 10 is the root → simply remove extra black (root absorbs it). Root stays Black.

Recolor 15 → Red:
```
Before:                         After:
    10(B)                           10(B)
   /     \                             \
 NIL(DB)  15(B)                       15(R)
```

x = root. Loop exits. Root color set to Black. ✓

**Result:**
```
  10(B)
     \
     15(R)
```

---

## Complexity

| Aspect | Cost |
|--------|------|
| BST walk down to target | O(log n) |
| Recoloring passes (Case 2) | O(log n) worst case |
| Rotations total | **At most 3** (Case 1 → Case 3 → Case 4) |
| Total | O(log n) |

RBT delete uses ≤ 3 rotations. AVL delete can use O(log n) rotations. This is the main practical advantage of RBTs for write-heavy workloads.

---

## Full Python Implementation

```python
RED, BLACK = 0, 1

class Node:
    def __init__(self, key):
        self.key    = key
        self.color  = RED
        self.left   = None
        self.right  = None
        self.parent = None

class RBTree:
    def __init__(self):
        self.NIL       = Node(0)
        self.NIL.color = BLACK
        self.root      = self.NIL

    # ── rotations ────────────────────────────────────────────────────────────

    def _left_rotate(self, x):
        y = x.right
        x.right = y.left
        if y.left != self.NIL:
            y.left.parent = x
        y.parent = x.parent
        if x.parent is None:
            self.root = y
        elif x is x.parent.left:
            x.parent.left = y
        else:
            x.parent.right = y
        y.left, x.parent = x, y

    def _right_rotate(self, x):
        y = x.left
        x.left = y.right
        if y.right != self.NIL:
            y.right.parent = x
        y.parent = x.parent
        if x.parent is None:
            self.root = y
        elif x is x.parent.right:
            x.parent.right = y
        else:
            x.parent.left = y
        y.right, x.parent = x, y

    # ── insert ────────────────────────────────────────────────────────────────

    def insert(self, key):
        z = Node(key)
        z.left = z.right = self.NIL
        y, x = None, self.root
        while x != self.NIL:
            y = x
            x = x.left if z.key < x.key else x.right
        z.parent = y
        if y is None:
            self.root = z
        elif z.key < y.key:
            y.left = z
        else:
            y.right = z
        self._insert_fixup(z)

    def _insert_fixup(self, z):
        while z.parent and z.parent.color == RED:
            if z.parent is z.parent.parent.left:
                u = z.parent.parent.right
                if u.color == RED:                  # Case 1
                    z.parent.color = BLACK
                    u.color = BLACK
                    z.parent.parent.color = RED
                    z = z.parent.parent
                else:
                    if z is z.parent.right:         # Case 2
                        z = z.parent
                        self._left_rotate(z)
                    z.parent.color = BLACK          # Case 3
                    z.parent.parent.color = RED
                    self._right_rotate(z.parent.parent)
            else:
                u = z.parent.parent.left
                if u.color == RED:                  # Case 1
                    z.parent.color = BLACK
                    u.color = BLACK
                    z.parent.parent.color = RED
                    z = z.parent.parent
                else:
                    if z is z.parent.left:          # Case 2
                        z = z.parent
                        self._right_rotate(z)
                    z.parent.color = BLACK          # Case 3
                    z.parent.parent.color = RED
                    self._left_rotate(z.parent.parent)
        self.root.color = BLACK

    # ── delete ────────────────────────────────────────────────────────────────

    def _transplant(self, u, v):
        if u.parent is None:
            self.root = v
        elif u is u.parent.left:
            u.parent.left = v
        else:
            u.parent.right = v
        v.parent = u.parent

    def _minimum(self, x):
        while x.left != self.NIL:
            x = x.left
        return x

    def delete(self, key):
        z = self.root
        while z != self.NIL and z.key != key:
            z = z.left if key < z.key else z.right
        if z == self.NIL:
            return

        y = z
        y_orig_color = y.color

        if z.left == self.NIL:
            x = z.right
            self._transplant(z, z.right)
        elif z.right == self.NIL:
            x = z.left
            self._transplant(z, z.left)
        else:
            y = self._minimum(z.right)     # in-order successor
            y_orig_color = y.color
            x = y.right
            if y.parent is z:
                x.parent = y
            else:
                self._transplant(y, y.right)
                y.right = z.right
                y.right.parent = y
            self._transplant(z, y)
            y.left = z.left
            y.left.parent = y
            y.color = z.color

        if y_orig_color == BLACK:
            self._delete_fixup(x)

    def _delete_fixup(self, x):
        while x is not self.root and x.color == BLACK:
            if x is x.parent.left:
                w = x.parent.right
                if w.color == RED:                             # Case 1
                    w.color = BLACK
                    x.parent.color = RED
                    self._left_rotate(x.parent)
                    w = x.parent.right
                if w.left.color == BLACK and w.right.color == BLACK:  # Case 2
                    w.color = RED
                    x = x.parent
                else:
                    if w.right.color == BLACK:                 # Case 3
                        w.left.color = BLACK
                        w.color = RED
                        self._right_rotate(w)
                        w = x.parent.right
                    w.color = x.parent.color                   # Case 4
                    x.parent.color = BLACK
                    w.right.color = BLACK
                    self._left_rotate(x.parent)
                    x = self.root
            else:                                              # mirror
                w = x.parent.left
                if w.color == RED:                             # Case 1
                    w.color = BLACK
                    x.parent.color = RED
                    self._right_rotate(x.parent)
                    w = x.parent.left
                if w.right.color == BLACK and w.left.color == BLACK:  # Case 2
                    w.color = RED
                    x = x.parent
                else:
                    if w.left.color == BLACK:                  # Case 3
                        w.right.color = BLACK
                        w.color = RED
                        self._left_rotate(w)
                        w = x.parent.left
                    w.color = x.parent.color                   # Case 4
                    x.parent.color = BLACK
                    w.left.color = BLACK
                    self._right_rotate(x.parent)
                    x = self.root
        x.color = BLACK

    # ── in-order traversal ────────────────────────────────────────────────────

    def inorder(self, node=None, first=True):
        if first:
            node = self.root
        if node != self.NIL:
            self.inorder(node.left, False)
            c = 'B' if node.color == BLACK else 'R'
            print(f"{node.key}({c})", end=' ')
            self.inorder(node.right, False)

# ── demo ──────────────────────────────────────────────────────────────────────

tree = RBTree()
for k in [41, 38, 31, 12, 19, 8]:
    tree.insert(k)

tree.inorder()    # 8(R) 12(B) 19(R) 31(B) 38(B) 41(B)
tree.delete(12)
tree.inorder()    # 8(B) 19(R) 31(B) 38(B) 41(B)  (8 recolored to Black)
```

---

## Insert vs Delete Summary

| Aspect | Insert | Delete |
|--------|--------|--------|
| Violation type | Red-Red (property 4) | Double-Black (property 5) |
| Fix-up cases | 3 | 4 |
| Max rotations | 2 | 3 |
| Recoloring passes | O(log n) | O(log n) |
| Total | O(log n) | O(log n) |

#adsa
