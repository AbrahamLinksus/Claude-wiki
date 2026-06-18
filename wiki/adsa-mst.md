---
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

**Greedy — Kruskal's Algorithm**: sort all edges by weight ascending. Walk the sorted list and add each edge to the MST as long as it does not form a cycle with edges already chosen. Stop once V−1 edges have been added (V = number of vertices).

```
function kruskalMST(graph):
    edges = sortByWeightAscending(graph.edges)
    mst = []
    for (u, v, weight) in edges:
        if not formsCycle(mst, u, v):
            mst.append((u, v, weight))
        if len(mst) == V - 1:
            break
    return mst
```

### Cycle Detection — Union-Find (Disjoint Set Union)

Kruskal's `formsCycle` check is implemented with Union-Find:

- Each vertex starts in its own set.
- `find(x)` returns the representative (root) of x's set.
- `union(u,v)` merges the two sets.
- Adding edge (u,v) creates a cycle **iff** `find(u) == find(v)` (they're already connected some other way).

```
function find(parent, x):
    while parent[x] != x:
        x = parent[x]
    return x

function union(parent, u, v):
    ru, rv = find(parent, u), find(parent, v)
    if ru == rv:
        return False   # would form a cycle
    parent[ru] = rv
    return True
```

With path compression + union by rank, `find`/`union` are nearly O(1) amortized, making Kruskal's overall **O(E log E)** (dominated by sorting the edges).

**Greedy — Prim's Algorithm**: vertex-centric (vs Kruskal's edge-centric). Grows one tree from a start vertex by always picking the cheapest edge crossing from the visited set to the unvisited set.

```
function primMST(graph, start):
    visited = {start}
    mst = []
    pq = MinHeap of (weight, u, v) for all edges from start
    while len(visited) < V:
        weight, u, v = pq.popMin()
        if v in visited: continue
        visited.add(v)
        mst.append((u, v, weight))
        for each edge (v, w, wt) where w not in visited:
            pq.push((wt, v, w))
    return mst
```

Complexity with a binary heap: **O(E log V)**.

### Kruskal's vs Prim's

| | Kruskal's | Prim's |
|---|---|---|
| Strategy | Sort all edges, pick globally smallest safe edge | Grow one tree, pick locally smallest crossing edge |
| Best for | Sparse graphs (E close to V) | Dense graphs (E close to V²) |
| Data structure | Union-Find | Min-heap / priority queue |
| Complexity | O(E log E) | O(E log V) |
| Needs connected start? | No (works component-wise → forest) | Yes (starts from one vertex) |

#adsa
