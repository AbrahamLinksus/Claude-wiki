---
# Directed vs Undirected Graphs

**Summary**: A directed graph's edges have a one-way direction; an undirected graph's edges go both ways.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

**Directed graph**: an edge A→B is one-way — it does not imply B→A. The relationship only holds in the direction the arrow points.

```
A → B → C
```

**Undirected graph**: an edge A-B goes both ways — A-B implies B-A. There's no arrow, just a connection.

```
A - B - C
```

## Advantages

**Directed:**
- Models asymmetric relationships naturally (one-way streets, follows, hyperlinks).
- Enables algorithms that need direction to be meaningful: topological sort, strongly connected components, dependency resolution.

**Undirected:**
- Simpler model when the relationship is inherently mutual (friendship, physical connection).
- Symmetric adjacency simplifies algorithms like connected components and minimum spanning trees.

## Disadvantages

**Directed:**
- Traversal is more constrained — you can't always walk backward along an edge.
- Requires separate in-degree/out-degree bookkeeping for some algorithms.

**Undirected:**
- Cannot model one-way relationships at all.
- If forced into a directed representation, each edge must be stored twice (A→B and B→A), wasting space.

## Representation

| | Adjacency List | Adjacency Matrix |
|---|---|---|
| **Directed** | `graph[u]` holds only outgoing edges to `v` | Not necessarily symmetric: `matrix[i][j] ≠ matrix[j][i]` in general |
| **Undirected** | Edge u-v stored in both `graph[u]` and `graph[v]` | Always symmetric: `matrix[i][j] == matrix[j][i]` |

## Pseudocode for Creation

```
function addDirectedEdge(graph, u, v):
    graph[u].append(v)

function addUndirectedEdge(graph, u, v):
    graph[u].append(v)
    graph[v].append(u)
```

## Use Cases

- **Directed**: task scheduling, web link graphs, "follows" graphs, citation networks
- **Undirected**: friendship networks, two-way road networks, circuit diagrams, collaboration networks

## Examples

- Twitter/X follow graph — directed (following someone doesn't mean they follow back)
- Facebook friend graph — undirected (friendship is mutual by definition)
- One-way street vs two-way street in a city map

## Where This Connects

Combined with [[adsa-cyclic-acyclic-graphs]], direction determines important special cases — e.g. a [[adsa-dag]] (Directed Acyclic Graph) requires both "directed" and "acyclic" together.

#adsa
