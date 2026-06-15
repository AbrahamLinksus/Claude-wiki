# Trees and Compiler Optimization

**Summary**: How the tree data structures from ADSA directly appear inside compilers — from AST traversal to symbol tables to IR layout.

**Pre-Req**: [[adsa-binary-trees]], [[adsa-tree-traversal]], [[adsa-red-black-trees]]

**Sources**: Q&A discussion (2026-06-15)

**Last updated**: 2026-06-15

---

## Abstract Syntax Trees (ASTs)

The compiler's primary internal representation is a tree. Every expression maps directly to a binary tree:

```
a + b * c
      +
     / \
    a   *
       / \
      b   c
```

An AST node is structurally identical to a [[adsa-binary-trees]] node: value at the node, left child, right child. The only difference is the values are tokens (operators, identifiers, literals) instead of integers.

### Traversal in code generation

[[adsa-tree-traversal]] DFS orderings map directly to compiler passes:

| DFS Order | Compiler use |
|-----------|--------------|
| **Post-order** | Code generation, constant folding — evaluate children before the parent so operands are resolved before the operator |
| **Pre-order** | Top-down rewrites: macro expansion, some algebraic simplification |
| **BFS** | Dataflow analysis across control flow graphs |

Post-order is the default for code generation because a compiler must emit code for `b * c` before it can emit the `+` instruction that consumes that result.

---

## Symbol Tables

During parsing and semantic analysis, every identifier lookup (variable type, function signature, scope check) hits the symbol table. This is a high-write, high-read workload.

**Why balanced BSTs?** O(log n) worst-case for both insert and lookup. Hash tables give O(1) average but O(n) worst-case and no ordering.

[[adsa-red-black-trees]] are the standard choice (C++ `std::map`, GCC internal data structures all use RBT). The reason matches the AVL vs RBT comparison in [[adsa-avl-trees]]: RBT allows faster inserts (at most 2 rotations vs AVL's O(log n)) which matters during parsing where declarations are being inserted constantly.

The 2 log n height bound on RBT directly bounds the worst-case lookup depth per identifier during compilation.

---

## IR Node Layout — Pointer vs Array

[[adsa-binary-tree-representation]] covers the two storage options:

- **Pointer-linked nodes** — flexible, supports arbitrary tree shapes, but scattered in memory → cache misses on every pointer dereference
- **Array-backed** — fixed shape required, index arithmetic instead of pointers, cache-friendly

Compilers face this exact trade-off for IR node storage. Modern compilers like LLVM use **arena allocators**: a large contiguous block where IR nodes are bump-allocated sequentially. This gives the cache locality of the array approach while still allowing pointer-linked structure. Optimization passes that walk the entire AST benefit significantly from nodes being contiguous in cache.

---

## B+ Trees in Query Compilers

[[adsa-b-plus-trees]] is less relevant to classical AOT compilers but central to **SQL query compilers**:

- The query optimizer chooses between full table scan and index scan based on whether a B+ tree index exists on the queried column
- Range queries (`WHERE age BETWEEN 20 AND 30`) use the linked leaf layer of the B+ tree — exactly the O(log n + k) range query algorithm in [[adsa-b-plus-tree-operations]]
- The internal nodes act as the routing index the wiki describes; leaf nodes hold the actual row pointers

---

## The Missing Piece: Dominator Trees

Not yet in the wiki, but the most important tree structure in advanced compiler optimization is the **dominator tree**:

- Built from the control flow graph (CFG), not the source AST
- Node A **dominates** node B if every path from the program entry to B must pass through A
- The dominator tree is the backbone of **SSA (Static Single Assignment)** construction, which underlies nearly all modern optimizations: loop invariant code motion, dead code elimination, inlining decisions, strength reduction

This is the natural next page if this topic continues.

---

## Summary Map

| Wiki page | Compiler use |
|-----------|--------------|
| [[adsa-binary-trees]] | AST node structure |
| [[adsa-tree-traversal]] | Code generation (post-order), dataflow (BFS) |
| [[adsa-avl-trees]] / [[adsa-red-black-trees]] | Symbol table — O(log n) lookup during parsing |
| [[adsa-binary-tree-representation]] | IR arena allocation vs pointer-linked nodes |
| [[adsa-b-plus-trees]] / [[adsa-b-plus-tree-operations]] | SQL query optimizer, index scan, range queries |
