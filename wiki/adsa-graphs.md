---
# Graphs

**Summary**: A graph is a non-linear data structure made of vertices (elements) and edges (connections), where the connections themselves carry the meaning of the structure. Unlike trees, graphs have no required root and can contain cycles.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Topic Map

- [[adsa-directed-undirected-graphs]] — Edge direction: one-way (directed) vs two-way (undirected)
- [[adsa-cyclic-acyclic-graphs]] — Whether paths can loop back; DAGs and their uses
- [[adsa-weighted-unweighted-graphs]] — Whether edges carry a cost/value

---

## What is a Graph?

A graph is **a group of elements where their connections matter**. The vertices alone are just data — the structure and meaning come from how they're connected via edges.

- **Vertices (nodes)** — the entities
- **Edges** — the connections between vertices

This is the key shift from trees: in a tree, structure is implied by parent/child pointers and there's exactly one path between any two nodes. In a graph, there's no such restriction — multiple paths can exist, and a path can loop back on itself.

A tree is actually a special case of a graph: a graph that is connected, acyclic, and undirected.

## Real-World Examples

- Road networks — cities = vertices, roads = edges (often weighted by distance)
- Social networks — people = vertices, relationships = edges (directed for "follows", undirected for "friends")
- The web — pages = vertices, hyperlinks = edges (directed)
- Git history — commits = vertices, parent links = edges (directed, acyclic)

## Roadmap to Mastery

1. Edge direction → [[adsa-directed-undirected-graphs]]
2. Cycles and DAGs → [[adsa-cyclic-acyclic-graphs]]
3. Edge weights → [[adsa-weighted-unweighted-graphs]]
4. Representation (adjacency list vs adjacency matrix) — _next topic_
5. Traversal (BFS/DFS on graphs vs trees) — _next topic_
6. Shortest path algorithms (Dijkstra, Bellman-Ford) — _next topic_
7. Minimum spanning trees (Prim's, Kruskal's) — _next topic_
8. Topological sort (for DAGs) — _next topic_

#adsa
