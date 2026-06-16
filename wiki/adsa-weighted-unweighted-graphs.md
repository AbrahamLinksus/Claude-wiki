---
# Weighted vs Unweighted Graphs

**Summary**: A weighted graph assigns a cost or value to each edge; an unweighted graph treats every edge as equal.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

**Unweighted graph**: every edge is treated identically — the only thing that matters is whether a connection exists.

```
A - B - C
```

**Weighted graph**: each edge carries a numeric value — typically cost, distance, time, or capacity.

```
A --5-- B --2-- C
```

## Advantages

**Weighted:**
- Models real-world costs precisely (distance, time, latency, capacity).
- Enables optimization algorithms that find the cheapest, not just the shortest, path.

**Unweighted:**
- Simple — plain BFS gives the shortest path (fewest hops) in O(V + E).
- Less storage — no need to store a value alongside each edge.

## Disadvantages

**Weighted:**
- More storage — each edge needs a value, not just an existence flag.
- Shortest-path algorithms (Dijkstra's, Bellman-Ford) are more expensive than plain BFS.

**Unweighted:**
- Can't distinguish a "cheap" connection from an "expensive" one — loses real-world fidelity when costs actually differ.

## Representation

| | Adjacency List | Adjacency Matrix |
|---|---|---|
| **Unweighted** | `graph[u] = [v1, v2, ...]` | `matrix[i][j] = 1` if edge exists, else 0 |
| **Weighted** | `graph[u] = [(v1, w1), (v2, w2), ...]` | `matrix[i][j] = weight`, else ∞ (or 0) |

## Pseudocode for Creation

```
function addWeightedEdge(graph, u, v, weight, directed = False):
    graph[u].append((v, weight))
    if not directed:
        graph[v].append((u, weight))
```

## Shortest Path: Why Weight Changes the Algorithm

In an unweighted graph, "distance" = number of edges (hop count) — BFS computes this directly.

In a weighted graph, "shortest path" means **minimum total weight**, not fewest hops. The path A→B→C with weights 5 and 2 costs 7, even though it's only 2 edges — BFS alone no longer guarantees correctness. This is where **Dijkstra's algorithm** (non-negative weights) and **Bellman-Ford** (handles negative weights, detects negative cycles) come in.

A **negative-weight cycle** is a cycle whose total weight is negative — meaning there's no true "shortest" path, since looping through it repeatedly keeps decreasing the cost. Dijkstra's assumes non-negative weights and breaks in their presence; Bellman-Ford can detect this case explicitly.

## Use Cases

- **Weighted**: GPS navigation (distance/time), network routing (latency/bandwidth), flight booking (cost), task scheduling (duration)
- **Unweighted**: friend-of-friend search, maze solving, web crawler reachability, simple connectivity checks

## Examples

- Google Maps route — weighted by distance/traffic
- "Degrees of separation" on a social network (e.g. Six Degrees of Kevin Bacon) — unweighted, hop-count based
- A flight network where edges are weighted by ticket price

## Where This Connects

Weight is independent of [[adsa-directed-undirected-graphs]] and [[adsa-cyclic-acyclic-graphs]] — any combination is possible. A weighted [[adsa-dag]] is common in scheduling, where edges represent task duration.

#adsa
