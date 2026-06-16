---
# Graphs

**Summary**: A graph is a non-linear data structure made of vertices (elements) and edges (connections), where the connections themselves carry the meaning of the structure. Unlike trees, graphs have no required root and can contain cycles.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Topic Map

- [[adsa-directed-undirected-graphs]] — Edge direction: one-way (directed) vs two-way (undirected)
- [[adsa-cyclic-acyclic-graphs]] — Whether paths can loop back; DAGs and their uses
- [[adsa-dag]] — Directed Acyclic Graphs as their own dedicated topic: topological sort, dependency modeling
- [[adsa-weighted-unweighted-graphs]] — Whether edges carry a cost/value
- [[adsa-explicit-implicit-graphs]] — Whether the graph is fully pre-built or generated on the fly during search
- [[adsa-adjacency-matrix]] — Representing a graph as a V×V grid; O(1) edge lookup vs O(V²) space
- [[adsa-adjacency-list]] — Representing a graph as per-vertex neighbor lists; O(V+E) space, weighted vs unweighted forms
- [[adsa-indegree-outdegree]] — Edges in vs edges out per vertex; directed-only concept; cost of computing from each representation

---

## Definition

A graph is **a group of elements where their connections matter**. The vertices alone are just data — the structure and meaning come from how they're connected via edges.

- **Vertices (nodes)** — the entities
- **Edges** — the connections between vertices

This is the key shift from trees: in a tree, structure is implied by parent/child pointers and there's exactly one path between any two nodes. In a graph, there's no such restriction — multiple paths can exist, and a path can loop back on itself.

A tree is actually a special case of a graph: a graph that is connected, acyclic, and undirected.

## Advantages

- Models complex many-to-many relationships that arrays, lists, and trees cannot (a vertex can connect to any number of other vertices, with no parent/child restriction).
- Flexible — direction and weight can be added independently to fit almost any real-world relationship.
- Well-studied algorithms exist for traversal, shortest path, connectivity, and ordering.

## Disadvantages

- Can be memory-intensive for dense graphs (adjacency matrix is O(V²) regardless of how many edges actually exist).
- Cycles add complexity — traversal must track visited vertices to avoid infinite loops (trees don't need this).
- No inherent ordering — unlike a sorted array or a BST, there's no single "natural" way to visit all vertices.

## Representation

| Method | Structure | Space | Edge lookup | Best for |
|---|---|---|---|---|
| **Adjacency List** | Each vertex maps to a list of its neighbors | O(V + E) | O(degree) | Sparse graphs |
| **Adjacency Matrix** | V × V matrix; `matrix[i][j]` = 1 if edge exists | O(V²) | O(1) | Dense graphs |

```
Adjacency List:          Adjacency Matrix:
A: [B, C]                    A  B  C
B: [A]                   A [ 0  1  1 ]
C: [A]                   B [ 1  0  0 ]
                          C [ 1  0  0 ]
```

## Pseudocode for Creation

```
function createGraph(vertices):
    graph = {}
    for v in vertices:
        graph[v] = []
    return graph

function addEdge(graph, u, v, directed = False):
    graph[u].append(v)
    if not directed:
        graph[v].append(u)
```

## Use Cases

- Social networks (people + relationships)
- Maps and navigation (locations + roads)
- Recommendation engines (users/items + interactions)
- Network routing (routers + links)
- Dependency resolution (packages/tasks + requirements)

## Examples

- A road map: cities = vertices, roads = edges
- A social graph: people = vertices, friendships/follows = edges
- The web: pages = vertices, hyperlinks = edges

## Roadmap to Mastery

1. Edge direction → [[adsa-directed-undirected-graphs]]
2. Cycles and DAGs → [[adsa-cyclic-acyclic-graphs]], [[adsa-dag]]
3. Edge weights → [[adsa-weighted-unweighted-graphs]]
4. Traversal (BFS/DFS on graphs vs trees) — _next topic_
5. Shortest path algorithms (Dijkstra, Bellman-Ford) — _next topic_
6. Minimum spanning trees (Prim's, Kruskal's) — _next topic_

#adsa
