---
# Graphs

**Summary**: A graph is a non-linear data structure made of vertices (elements) and edges (connections), where the connections themselves carry the meaning of the structure. Unlike trees, graphs have no required root and can contain cycles.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## What is a Graph?

A graph is **a group of elements where their connections matter**. The vertices alone are just data — the structure and meaning come from how they're connected via edges.

- **Vertices (nodes)** — the entities
- **Edges** — the connections between vertices

This is the key shift from trees: in a tree, structure is implied by parent/child pointers and there's exactly one path between any two nodes. In a graph, there's no such restriction — multiple paths can exist, and a path can loop back on itself.

A tree is actually a special case of a graph: a graph that is connected, acyclic, and undirected.

## Classification

Graphs are classified along two independent axes:

| | **Directed** | **Undirected** |
|---|---|---|
| **Cyclic** | Cycle follows edge direction, e.g. A→B→C→A | Any loop back to a node via edges, e.g. A-B-C-A |
| **Acyclic** | DAG (Directed Acyclic Graph) — e.g. task dependencies, git commit history | A tree, if connected |

- **Directed edge**: has a direction — A→B does not imply B→A (e.g. a Twitter follow).
- **Undirected edge**: goes both ways — A-B implies B-A (e.g. a Facebook friendship).
- **Cyclic**: at least one path loops back to a vertex it already visited.
- **Acyclic**: no such path exists.

A **DAG** is especially important in practice: it's used for scheduling, build systems, and version history — git commits form a DAG.

A **weighted** graph additionally assigns a cost/value to each edge (e.g. distance, time); an **unweighted** graph treats all edges as equal.

## Real-World Examples

- Road networks — cities = vertices, roads = edges (often weighted by distance)
- Social networks — people = vertices, relationships = edges (directed for "follows", undirected for "friends")
- The web — pages = vertices, hyperlinks = edges (directed)
- Git history — commits = vertices, parent links = edges (directed, acyclic)

## Roadmap to Mastery

1. Classification (cyclic/acyclic, directed/undirected, weighted/unweighted) — done above
2. Representation (adjacency list vs adjacency matrix) — _next topic_
3. Traversal (BFS/DFS on graphs vs trees) — _next topic_
4. Shortest path algorithms (Dijkstra, Bellman-Ford) — _next topic_
5. Minimum spanning trees (Prim's, Kruskal's) — _next topic_
6. Topological sort (for DAGs) — _next topic_

#adsa
