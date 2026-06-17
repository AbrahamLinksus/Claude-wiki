---
# Minimum Spanning Tree (MST)

**Summary**: An MST is the subset of edges from a connected, undirected, weighted graph that connects all vertices with the minimum possible total edge weight.

**Pre-Req**: [[adsa-graphs]], [[adsa-directed-undirected-graphs]], [[adsa-weighted-unweighted-graphs]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-17

---

## Requirements

An MST applies to a graph that is:

- **Connected** — every vertex must be reachable, otherwise no single spanning tree exists (a disconnected graph instead needs a minimum spanning *forest*, one tree per component).
- **Undirected** — standard MST algorithms (Prim's, Kruskal's) assume undirected edges. Directed graphs have an analogous but distinct problem: minimum arborescence (Chu-Liu/Edmonds' algorithm).
- **Weighted** — without edge weights, any spanning tree is "minimum" by default, since there's nothing to optimize.

A common misconception: the graph does **not** need to contain a cycle. If the graph is already a tree (acyclic), it trivially is its own MST. Cycles aren't a precondition — they just mean multiple spanning trees exist to choose between, which is what makes the "minimum" part non-trivial. See [[adsa-cyclic-acyclic-graphs]].

#adsa
