# AVL Tree Operations

**Summary**: How AVL insert and delete work — BST operation followed by bottom-up height update and rebalancing via rotations. Includes full dry-run examples for both.

**Pre-Req**: [[adsa-avl-trees]], [[adsa-avl-rotations]]

**Sources**: Session-built (no raw file).

**Last updated**: 2026-06-15

---

## Height and Balance Utilities

Every AVL operation depends on two helpers:

```python
def height(node):
    return node.height if node else -1   # null nodes have height -1

def update_height(node):
    node.height = 1 + max(height(node.left), height(node.right))

def balance_factor(node):
    return height(node.left) - height(node.right) if node else 0
```

And the unified rebalance function that checks BF and picks the right rotation:

```python
def rebalance(node):
    update_height(node)
    bf = balance_factor(node)

    if bf == 2:                              # left-heavy
        if balance_factor(node.left) < 0:   # LR case
            node.left = left_rotate(node.left)
        return right_rotate(node)

    if bf == -2:                             # right-heavy
        if balance_factor(node.right) > 0:  # RL case
            node.right = right_rotate(node.right)
        return left_rotate(node)

    return node                              # already balanced
```

See [[adsa-avl-rotations]] for `left_rotate` and `right_rotate` implementations.

---

## Node Structure

```python
class Node:
    def __init__(self, key):
        self.key    = key
        self.left   = None
        self.right  = None
        self.height = 0      # leaf starts at 0
```

---

## Insert

**Algorithm**:
1. Walk down exactly like a BST insert (go left if key < current, right if key > current).
2. Create the new leaf at the correct position.
3. On the way **back up** the recursion, call `rebalance` at every ancestor.
4. At most one rotation (single or double) fires per insert — after that, all ancestors are balanced.

```python
def insert(root, key):
    if root is None:
        return Node(key)
    if key < root.key:
        root.left  = insert(root.left, key)
    elif key > root.key:
        root.right = insert(root.right, key)
    else:
        return root          # duplicate key — no insert
    return rebalance(root)
```

**Complexity**: O(log n) to walk down, O(log n) to retrace up, O(1) per rotation → **O(log n) total**.
At most **one** rotation (single or double) per insert call.

---

## Insert Dry Run — Sequence: 30, 20, 10, 25, 27

### Step 1: Insert 30
```
30 (h=0, BF=0)
```
No imbalance.

---

### Step 2: Insert 20
20 < 30, goes left.
```
  30 (h=1, BF=+1)
 /
20 (h=0, BF=0)
```
BF(30) = 1 — still balanced.

---

### Step 3: Insert 10
10 < 30, go left. 10 < 20, go left.
```
      30 (h=2, BF=+2)  ← UNBALANCED
     /
    20 (h=1, BF=+1)
   /
  10 (h=0, BF=0)
```
Retrace back up hits node 30 first. BF(30) = +2, BF(30.left=20) = +1 ≥ 0 → **LL case → Right Rotate at 30**.

```
After Right Rotate at 30:

    20 (h=1, BF=0)
   /  \
  10   30
(h=0) (h=0)
```

Tree is balanced. ✓

---

### Step 4: Insert 25
25 > 20, go right. 25 < 30, go left.
```
    20 (h=2, BF=-1)
   /  \
  10   30 (h=1, BF=+1)
      /
     25 (h=0, BF=0)
```
Retrace: BF(30) = +1, BF(20) = −1 — all within {−1, 0, +1}. No rotation needed. ✓

---

### Step 5: Insert 27
27 > 20, go right. 27 < 30, go left. 27 > 25, go right.

```
    20
   /  \
  10   30
      /
     25
       \
        27
```

**Bottom-up height update and BF check:**

| Node | h(left) | h(right) | BF | Status |
|------|---------|----------|-----|--------|
| 27 | −1 | −1 | 0 | ✓ |
| 25 | −1 | 0 | −1 | ✓ |
| 30 | 1 | −1 | **+2** | ← UNBALANCED |

BF(30) = +2, BF(30.left=25) = −1 < 0 → **LR case**.

**Step 5a — Left Rotate at 25 (z.left):**
```
    30
   /
  27
 /
25
```
Now 30.left = 27, BF(25) = 0, BF(27) = +1.

**Step 5b — Right Rotate at 30:**
```
    27
   /  \
  25   30
```
BF(25) = 0, BF(30) = 0, BF(27) = 0. ✓

Retrace continues up to 20. 20.right is now 27 (height = 1).
BF(20) = h(10) − h(27) = 0 − 1 = −1 ✓.

**Final tree after all 5 inserts:**
```
      20 (BF=-1)
     /  \
    10   27 (BF=0)
        /  \
       25   30
```

All BFs ∈ {−1, 0, +1}. Height = 2 (vs height = 4 for an unbalanced BST). ✓

---

## Delete

Delete is structurally more complex than insert because:
- There are three BST sub-cases for the node being removed.
- **Multiple rotations** may fire as you retrace — each ancestor may independently become unbalanced.

**Three BST deletion cases:**

| Situation | Action |
|-----------|--------|
| Node is a leaf | Simply remove it |
| Node has one child | Replace node with its child |
| Node has two children | Replace node's key with its **in-order successor** (leftmost node in right subtree), then delete that successor from the right subtree |

**Algorithm:**

```python
def min_node(node):
    while node.left:
        node = node.left
    return node

def delete(root, key):
    if root is None:
        return None

    if key < root.key:
        root.left  = delete(root.left, key)
    elif key > root.key:
        root.right = delete(root.right, key)
    else:
        # Found the node to delete
        if root.left is None:      # no left child (covers leaf too)
            return root.right
        elif root.right is None:   # no right child
            return root.left
        else:
            # Two children: swap with in-order successor
            successor    = min_node(root.right)
            root.key     = successor.key
            root.right   = delete(root.right, successor.key)

    return rebalance(root)         # rebalance on the way back up
```

**Complexity**: O(log n) walk down + O(log n) retrace, O(log n) rotations in the worst case → **O(log n) total**.

### Special case: BF(child) = 0 during delete

When the heavier child has BF = 0 (impossible during insert, can happen during delete), you pick the same-direction rotation. The result will have BF = ±1 — still valid AVL. You **must** continue retracing upward because the subtree height decreased by 1.

---

## Delete Dry Run — Delete 10 from the final tree

Starting tree:
```
      20 (BF=-1)
     /  \
    10   27 (BF=0)
        /  \
       25   30
```

**Delete 10** — it is a leaf, remove it directly.

```
      20
        \
         27
        /  \
       25   30
```

**Bottom-up BF check at 20:**

| Node | h(left) | h(right) | BF | Status |
|------|---------|----------|-----|--------|
| 25 | −1 | −1 | 0 | ✓ |
| 30 | −1 | −1 | 0 | ✓ |
| 27 | 0 | 0 | 0 | ✓ |
| 20 | −1 | 1 | **−2** | ← UNBALANCED |

BF(20) = −2 (right-heavy), BF(20.right=27) = 0 → **RR case** (BF of child ≤ 0) → **Left Rotate at 20**.

Note: when BF(child) = 0, the result will have BF = +1, not 0 — this is valid but the subtree height decreases, so we continue retracing.

**Left Rotate at 20** (y=27, T2=27.left=25):
- 27.left = 20
- 20.right = 25 (T2)

```
      27 (BF=+1)
     /  \
    20   30 (BF=0)
      \
       25 (BF=0)
```

| Node | h(left) | h(right) | BF |
|------|---------|----------|-----|
| 25 | −1 | −1 | 0 ✓ |
| 30 | −1 | −1 | 0 ✓ |
| 20 | −1 | 0 | −1 ✓ |
| 27 | 1 | 0 | +1 ✓ |

Tree is balanced. 27 is the new root. ✓

---

## Delete Dry Run — Delete node with two children

Starting from:
```
    27 (BF=0)
   /  \
  25   30
```

**Delete 27** — it has two children.
- In-order successor = leftmost in right subtree = 30.
- Copy 30's key into 27's position. Delete 30 from the right subtree.

```
    30 (h=1, BF=+1)   ← key replaced, 30 removed from right
   /
  25
```
BF(30_new) = h(25) − h(null) = 0 − (−1) = +1 ✓ — no rotation needed.

---

## Complexity Summary

| Operation | Average | Worst case | Rotations |
|-----------|---------|------------|-----------|
| Search | O(log n) | O(log n) | 0 |
| Insert | O(log n) | O(log n) | ≤ 1 (single or double) |
| Delete | O(log n) | O(log n) | ≤ O(log n) |
| Space | O(n) | O(n) | — |

The height bound of 1.44 log₂ n guarantees the O(log n) path in all cases — no degenerate scenario exists for a correctly maintained AVL tree.

---

## Full Python Implementation

```python
class Node:
    def __init__(self, key):
        self.key    = key
        self.left   = None
        self.right  = None
        self.height = 0

# ── helpers ──────────────────────────────────────────────────────────────────

def height(node):
    return node.height if node else -1

def update_height(node):
    node.height = 1 + max(height(node.left), height(node.right))

def bf(node):
    return height(node.left) - height(node.right) if node else 0

# ── primitive rotations ───────────────────────────────────────────────────────

def right_rotate(z):
    y, T3 = z.left, z.left.right
    y.right, z.left = z, T3
    update_height(z)
    update_height(y)
    return y

def left_rotate(z):
    y, T2 = z.right, z.right.left
    y.left, z.right = z, T2
    update_height(z)
    update_height(y)
    return y

# ── rebalance ─────────────────────────────────────────────────────────────────

def rebalance(node):
    update_height(node)
    b = bf(node)
    if b == 2:
        if bf(node.left) < 0:           # LR
            node.left = left_rotate(node.left)
        return right_rotate(node)
    if b == -2:
        if bf(node.right) > 0:          # RL
            node.right = right_rotate(node.right)
        return left_rotate(node)
    return node

# ── insert ────────────────────────────────────────────────────────────────────

def insert(root, key):
    if root is None:
        return Node(key)
    if key < root.key:
        root.left  = insert(root.left, key)
    elif key > root.key:
        root.right = insert(root.right, key)
    return rebalance(root)

# ── delete ────────────────────────────────────────────────────────────────────

def _min_node(node):
    while node.left:
        node = node.left
    return node

def delete(root, key):
    if root is None:
        return None
    if key < root.key:
        root.left  = delete(root.left, key)
    elif key > root.key:
        root.right = delete(root.right, key)
    else:
        if root.left is None:
            return root.right
        if root.right is None:
            return root.left
        succ       = _min_node(root.right)
        root.key   = succ.key
        root.right = delete(root.right, succ.key)
    return rebalance(root)

# ── in-order traversal (verifies BST property) ───────────────────────────────

def inorder(root):
    if root:
        inorder(root.left)
        print(root.key, end=' ')
        inorder(root.right)

# ── demo ──────────────────────────────────────────────────────────────────────

root = None
for k in [30, 20, 10, 25, 27]:
    root = insert(root, k)

inorder(root)   # 10 20 25 27 30
root = delete(root, 10)
inorder(root)   # 20 25 27 30
```

#adsa
