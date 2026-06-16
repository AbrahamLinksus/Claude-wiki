---
# Adjacency List

**Summary**: An adjacency list represents a graph as a map from each vertex to a list of its neighbors — optionally paired with edge weights — instead of a full V×V grid.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

Each vertex maps to a list (or set) containing only the vertices it's actually connected to.

**Unweighted**: each entry is just the neighboring vertex.
```
A: [B, C]
B: [A]
C: [A]
```

**Weighted** (see [[adsa-weighted-unweighted-graphs]]): each entry is a `(neighbor, weight)` pair.
```
A: [(B, 5)]
B: [(A, 5), (C, 2)]
C: [(B, 2)]
```

For a **directed** graph (see [[adsa-directed-undirected-graphs]]), `graph[u]` holds only outgoing edges. For an **undirected** graph, each edge u-v is stored twice — once in `graph[u]` and once in `graph[v]`.

## Key Properties

- Space: **O(V + E)** — only existing edges are stored, unlike the matrix's fixed O(V²).
- Iterating a vertex's neighbors: **O(degree)** — just walk that vertex's list.
- Edge existence check (`does u-v exist?`): **O(degree)** — must scan the list (unless backed by a hash set, which gives O(1)).
- Adding a vertex: O(1) — just add a new empty list/entry.

## Advantages

- Space-efficient for sparse graphs — most real-world graphs (social networks, road maps, the web) have far fewer edges than V², so this avoids wasting memory on non-existent edges.
- Fast neighbor iteration — directly returns only the vertices that matter, with no need to skip zeros like a matrix row.
- Easy to grow — adding vertices/edges doesn't require resizing a fixed structure.

## Disadvantages

- Edge existence check is slower than a matrix's O(1), unless the per-vertex list is backed by a hash set/map (trading some space for O(1) lookup).
- Less convenient for algorithms that are naturally matrix-shaped (e.g. Floyd-Warshall, which relies on direct `matrix[i][j]` access for all pairs).
- Slightly more bookkeeping for weighted/directed graphs, since each entry is a pair and undirected edges must be inserted twice.

## Pseudocode for Creation

**Unweighted:**
```
function createGraph(vertices):
    graph = {v: [] for v in vertices}
    return graph

function addEdge(graph, u, v, directed = False):
    graph[u].append(v)
    if not directed:
        graph[v].append(u)
```

**Weighted:**
```
function addWeightedEdge(graph, u, v, weight, directed = False):
    graph[u].append((v, weight))
    if not directed:
        graph[v].append((u, weight))
```

## Use Cases

- Sparse graphs — social networks, road networks, the web, dependency graphs, anywhere E << V².
- BFS/DFS traversal, where you repeatedly need "give me all neighbors of this vertex" — the list's natural strength.
- Dijkstra's/Bellman-Ford on weighted graphs, where the algorithm reads `(neighbor, weight)` pairs directly off each vertex.

## Examples

- A friend graph with millions of users but each person has only a few hundred friends — a matrix would need (millions)² cells, almost all zero; the list stores only the real connections.
- A road network where each city connects to a handful of nearby cities, with the weight being distance — exactly the weighted adjacency list shown above.
- A package dependency graph (npm, pip) — each package lists only its direct dependencies.

## Where This Connects

The adjacency list is the sparse-graph counterpart to [[adsa-adjacency-matrix]] — same graph from [[adsa-graphs]], different space/lookup trade-off. Choose list for sparse graphs and frequent neighbor iteration (most traversal and shortest-path algorithms); choose matrix for dense graphs and frequent single-edge lookups.

#adsa
