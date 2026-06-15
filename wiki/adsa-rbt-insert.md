# RBT Insert

**Summary**: BST insert followed by a fix-up loop that restores the Red-Black properties using recoloring and at most 2 rotations.

**Pre-Req**: [[adsa-red-black-trees]], [[adsa-avl-rotations]]

**Sources**: Session-built (no raw file).

**Last updated**: 2026-06-15

---

## Algorithm Overview

```
1. Insert new node z using standard BST logic.
2. Color z Red.
3. If z is the root → color it Black. Done.
4. If z's parent is Black → no violation. Done.
5. If z's parent is Red → property 4 violated. Run fix-up.
```

The fix-up loop maintains this invariant: **z and z's parent are both Red** — the only violation. It resolves the violation by moving it upward (recoloring) or eliminating it entirely (rotation + recolor). The loop terminates because either the problem is fixed at z, or z climbs the tree and eventually reaches a Black parent or the root.

**Notation used below:**
- `z` — current violation node (Red)
- `p` — parent of z
- `g` — grandparent of z (exists because p is Red and root is always Black, so g ≠ NIL)
- `u` — uncle of z (the other child of g, sibling of p)

All three cases assume `p` is the **left** child of `g`. Mirror cases apply when `p` is the right child (swap left↔right and left-rotate↔right-rotate).

---

## Case 1 — Uncle is Red (Recolor)

**Condition**: `u.color == RED`

**Fix**: Recolor p and u to Black, recolor g to Red. Move z up to g and repeat.

```
          g(B)                             g(R)  ← z moves here
         / \           Recolor             / \
        p(R) u(R)      ────────►          p(B)  u(B)
       /                                 /
      z(R)                              z(R)
```

The violation is pushed up two levels. If g's parent is Black, the loop stops. If g's parent is Red, the loop repeats with z = g.

At the end, the root is forced to Black regardless — this handles the edge case where z climbs all the way to the root.

---

## Case 2 — Uncle is Black, Triangle Shape

**Condition**: `u.color == BLACK` and z is the **opposite** child direction from p.
(p is left child of g, z is right child of p — they form a "bent" or triangle shape.)

**Fix**: Rotate at p to straighten the path (convert to Case 3). z and p swap roles.

```
        g(B)                            g(B)
       / \       Left rotate            / \
      p(R) u(B)     at p    ──►        z(R) u(B)
       \                              /
        z(R)                         p(R)
                                    ↑ new z = old p
```

After the rotation, the new z (old p) is a left child of the new parent (old z). This is now a Line shape → falls directly into Case 3.

Case 2 never stands alone — it always transitions immediately to Case 3.

---

## Case 3 — Uncle is Black, Line Shape

**Condition**: `u.color == BLACK` and z is the **same** child direction as p.
(p is left child of g, z is left child of p — they form a straight line.)

**Fix**: Right rotate at g. Recolor p → Black, g → Red. Violation resolved.

```
          g(B)                              p(B)
         / \       Right rotate            / \
        p(R) u(B)     at g    ──►         z(R) g(R)
       /                                       \
      z(R)                                     u(B)
```

After this rotation:
- p is the new subtree root, colored Black.
- g dropped to be p's right child, colored Red.
- z remains Red under p.
- No red-red violation exists. Fix-up loop exits.

---

## Case Detection Summary

```
p is left child of g:

    u is Red            →  Case 1 (recolor, move z up)
    u is Black, z right →  Case 2 (left rotate at p) → Case 3
    u is Black, z left  →  Case 3 (right rotate at g, recolor)

p is right child of g (mirror):

    u is Red            →  Case 1 (recolor, move z up)
    u is Black, z left  →  Case 2 (right rotate at p) → Case 3
    u is Black, z right →  Case 3 (left rotate at g, recolor)
```

---

## Insert Dry Run — Sequence: 41, 38, 31, 12, 19, 8

Notation: `B` = Black, `R` = Red, `NIL` = sentinel (always Black).

---

### Insert 41
41 is the first node → it becomes the root → colored Black.
```
41(B)
```

---

### Insert 38
38 < 41 → left child of 41. Colored Red. Parent 41 is Black → no violation.
```
  41(B)
 /
38(R)
```

---

### Insert 31 — triggers **Case 3**
31 < 41, go left. 31 < 38, go left. Colored Red.
z=31(R), p=38(R) → red-red! g=41(B), u=41.right=NIL(B).
p is left child of g. z is left child of p → **Line → Case 3**.

Right rotate at g=41. Recolor p=38→B, g=41→R.
38 is now the tree root (was g=41 which was root) → root stays Black ✓.

```
   38(B)
  /    \
31(R)  41(R)
```

---

### Insert 12 — triggers **Case 1**
12 < 38, go left. 12 < 31, go left. Colored Red.
z=12(R), p=31(R) → red-red! g=38(B), u=38.right=41(R).
Uncle is Red → **Case 1**.

Recolor: p=31→B, u=41→B, g=38→R. Move z up to g=38.
38 is root → force root to Black. Done.

```
    38(B)
   /    \
 31(B)  41(B)
 /
12(R)
```

---

### Insert 19 — triggers **Case 2 → Case 3**
19 < 38, go left. 19 < 31, go left. 19 > 12, go right. Colored Red.
z=19(R), p=12(R) → red-red! g=31(B), u=31.right=NIL(B).
p=12 is left child of g=31. z=19 is **right** child of p → **Triangle → Case 2**.

**Case 2 — Left rotate at p=12:**
19 rises to be 31's left child. 12 drops to be 19's left child. z pointer = old p = 12.

After rotate:
```
    38(B)
   /    \
 31(B)  41(B)
 /
19(R)
/
12(R)
```
Now z=12(R), p=19(R), g=31(B), u=NIL(B). z is left child of p, p is left child of g → **Line → Case 3**.

**Case 3 — Right rotate at g=31:**
19 rises to 31's position. 31 drops to be 19's right child. 19's old right (NIL) becomes 31's left.
Recolor: p=19→B, g=31→R.

19's subtree is now:
```
   19(B)
  /    \
12(R)  31(R)
```
19 becomes the left child of 38.

Full tree:
```
      38(B)
     /    \
   19(B)  41(B)
   /    \
  12(R)  31(R)
```

---

### Insert 8 — triggers **Case 1**
8 < 38, left. 8 < 19, left. 8 < 12, left. Colored Red.
z=8(R), p=12(R) → red-red! g=19(B), u=19.right=31(R).
Uncle is Red → **Case 1**.

Recolor: p=12→B, u=31→B, g=19→R. Move z up to g=19.
Parent of 19 = 38(B) → no red-red violation. Done.

Final tree:
```
        38(B)
       /      \
    19(R)    41(B)
    /    \
  12(B)  31(B)
  /
 8(R)
```

---

### Verification

| Node | Color | Children | Red-Red violation? | BH from node |
|------|-------|----------|--------------------|--------------|
| 38 | B | 19(R), 41(B) | No | 1 |
| 19 | R | 12(B), 31(B) | No (parent 38 is B) | 1 |
| 41 | B | NIL, NIL | No | 0 |
| 12 | B | 8(R), NIL | No | 0 |
| 31 | B | NIL, NIL | No | 0 |
| 8 | R | NIL, NIL | No (parent 12 is B) | 0 |

All paths from 38 to NIL: count Black nodes (not counting NIL sentinel):
- 38→19→12→8→NIL: Black nodes = 38, 12 = **bh 2** from 38 (counting below 38: 12, that's 1 black below 19... wait)

Let me count carefully. BH from root (38) = count of Black nodes strictly below 38 on any path:
- 38→19(R)→12(B)→8(R)→NIL: below 38: 19(R), 12(B), 8(R) → 1 Black = **bh 1**
- 38→19(R)→12(B)→NIL(right): below 38: 19(R), 12(B) → 1 Black = **bh 1** ✓
- 38→19(R)→31(B)→NIL: below 38: 19(R), 31(B) → 1 Black = **bh 1** ✓
- 38→41(B)→NIL: below 38: 41(B) → 1 Black = **bh 1** ✓

All paths have bh = 1 from the root. Property 5 holds. ✓

---

## Complexity

| Aspect | Cost |
|--------|------|
| BST walk down | O(log n) |
| Recoloring passes (Case 1) | O(log n) worst case |
| Rotations | **At most 2** (one Case 2 + one Case 3) per insert |
| Total | O(log n) |

Note: Case 1 can repeat O(log n) times, but each iteration just recolors two nodes and moves z upward. Case 2 and Case 3 each happen at most once — Case 2 transitions to Case 3, and Case 3 terminates the loop.

---

## Python Code

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
        self.NIL        = Node(0)
        self.NIL.color  = BLACK
        self.root       = self.NIL

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
        z.color = RED

        # BST walk
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
                u = z.parent.parent.right          # uncle
                if u.color == RED:                 # Case 1
                    z.parent.color = BLACK
                    u.color = BLACK
                    z.parent.parent.color = RED
                    z = z.parent.parent
                else:
                    if z is z.parent.right:        # Case 2 → 3
                        z = z.parent
                        self._left_rotate(z)
                    z.parent.color = BLACK         # Case 3
                    z.parent.parent.color = RED
                    self._right_rotate(z.parent.parent)
            else:                                  # mirror
                u = z.parent.parent.left
                if u.color == RED:                 # Case 1
                    z.parent.color = BLACK
                    u.color = BLACK
                    z.parent.parent.color = RED
                    z = z.parent.parent
                else:
                    if z is z.parent.left:         # Case 2 → 3
                        z = z.parent
                        self._right_rotate(z)
                    z.parent.color = BLACK         # Case 3
                    z.parent.parent.color = RED
                    self._left_rotate(z.parent.parent)
        self.root.color = BLACK

    # ── in-order (verify BST property) ───────────────────────────────────────

    def inorder(self, node=None, first=True):
        if first:
            node = self.root
        if node != self.NIL:
            self.inorder(node.left, False)
            c = 'B' if node.color == BLACK else 'R'
            print(f"{node.key}({c})", end=' ')
            self.inorder(node.right, False)
```

See [[adsa-rbt-delete]] for the complete implementation including delete.

#adsa
