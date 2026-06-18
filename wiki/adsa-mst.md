# Minimum Spanning Tree (MST)

**Summary**: An MST is the subset of edges from a connected, undirected, weighted graph that connects all vertices with the minimum possible total edge weight.

**Pre-Req**: [[adsa-graphs]], [[adsa-directed-undirected-graphs]], [[adsa-weighted-unweighted-graphs]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-18

---

## Requirements

An MST applies to a graph that is:

- **Connected** — every vertex must be reachable, otherwise no single spanning tree exists (a disconnected graph instead needs a minimum spanning *forest*, one tree per component).
- **Undirected** — standard MST algorithms (Prim's, Kruskal's) assume undirected edges. Directed graphs have an analogous but distinct problem: minimum arborescence (Chu-Liu/Edmonds' algorithm).
- **Weighted** — without edge weights, any spanning tree is "minimum" by default, since there's nothing to optimize.

A common misconception: the graph does **not** need to contain a cycle. If the graph is already a tree (acyclic), it trivially is its own MST. Cycles aren't a precondition — they just mean multiple spanning trees exist to choose between, which is what makes the "minimum" part non-trivial. See [[adsa-cyclic-acyclic-graphs]].

## Approaches

**Brute force**: enumerate every possible spanning tree of the graph, compute each one's total edge weight, and keep the minimum. Not every *path* — a spanning tree connects all vertices with no cycles, which is a distinct object from a path (path-finding is the domain of Dijkstra/Bellman-Ford, a different problem).

**Greedy**: both standard algorithms below make a series of locally-best choices (cheapest edge available at each step) and still land on a *globally* optimal answer. This works for MST specifically because the set of spanning trees forms a matroid — each algorithm's safe-edge rule (Cut Property for Prim's, Cycle Property for Kruskal's) is a proven guarantee, not a heuristic. Unlike greedy on problems like knapsack or general TSP, a locally cheapest choice here never locks out the global optimum.

## Optimality Guarantee

Both algorithms always return a spanning tree of **minimum total weight** — guaranteed, on any connected weighted undirected graph. The only caveat: if edge weights aren't all distinct, multiple different edge sets can tie for that minimum weight (multiple valid MSTs). Each algorithm still finds *one* of them, and its total weight is always optimal.

## Topic Map

| Page | What it covers |
|------|----------------|
| [[adsa-kruskals-algorithm]] | Edge-centric, sorts all edges, Union-Find cycle detection, Cycle Property |
| [[adsa-prims-algorithm]] | Vertex-centric, grows one tree, min-heap frontier, Cut Property |

## Kruskal's vs Prim's

| | Kruskal's | Prim's |
|---|---|---|
| Strategy | Sort all edges, pick globally smallest safe edge | Grow one tree, pick locally smallest crossing edge |
| Best for | Sparse graphs (E close to V) | Dense graphs (E close to V²) |
| Data structure | Union-Find | Min-heap / priority queue |
| Complexity | O(E log E) | O(E log V) |
| Needs connected start? | No (works component-wise → forest) | Yes (starts from one vertex) |

#adsa
