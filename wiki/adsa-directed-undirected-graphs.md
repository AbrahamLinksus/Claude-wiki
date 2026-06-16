---
# Directed vs Undirected Graphs

**Summary**: A directed graph's edges have a one-way direction; an undirected graph's edges go both ways.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Directed Graphs

An edge A→B is one-way: it does not imply B→A. The relationship only holds in the direction the arrow points.

```
A → B → C
```

Real-world example: a Twitter/X follow. If A follows B, that doesn't mean B follows A.

## Undirected Graphs

An edge A-B goes both ways: A-B implies B-A. There's no arrow, just a connection.

```
A - B - C
```

Real-world example: a Facebook friendship — if A is friends with B, B is friends with A by definition.

## Why It Matters

The direction of edges changes which algorithms and traversals are valid:
- Traversal order in a directed graph depends on which way edges point — you can't walk backward along a directed edge.
- Many algorithms (topological sort, strongly connected components) only make sense on directed graphs.
- An undirected graph can be modeled as a directed graph by storing each edge twice (A→B and B→A) — this is a common implementation trick for adjacency lists.

## Where This Connects

Combined with [[adsa-cyclic-acyclic-graphs]], direction determines important special cases — e.g. a **DAG** (Directed Acyclic Graph) requires both "directed" and "acyclic" together.

#adsa
