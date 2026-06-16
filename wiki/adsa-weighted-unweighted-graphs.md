---
# Weighted vs Unweighted Graphs

**Summary**: A weighted graph assigns a cost or value to each edge; an unweighted graph treats every edge as equal.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Unweighted Graphs

Every edge is treated identically — the only thing that matters is whether a connection exists.

```
A - B - C
```

"Distance" between vertices here just means **number of edges** (hop count). This is what plain BFS computes: shortest path in an unweighted graph = fewest edges.

## Weighted Graphs

Each edge carries a numeric value — typically representing cost, distance, time, or capacity.

```
A --5-- B --2-- C
```

Here, the path A→B→C costs 5+2=7, even though it's still only 2 edges. "Shortest path" now means **minimum total weight**, not fewest hops — BFS alone no longer guarantees correctness; this is where algorithms like **Dijkstra's** and **Bellman-Ford** come in.

## Real-World Examples

| Weighted | Unweighted |
|---|---|
| Road networks — weight = distance/time | Social network "friend" graph — connection either exists or doesn't |
| Flight routes — weight = cost or duration | Web link graph (basic) — a link exists or doesn't |
| Network routing — weight = latency/bandwidth | Maze solving — each cell move costs the same |

## Negative Weights

Weights can be negative (e.g. modeling a discount or gain), but this introduces a subtlety:
- **Dijkstra's algorithm** assumes non-negative weights and breaks with negative ones.
- **Bellman-Ford** handles negative weights and can detect negative-weight cycles (a cycle whose total weight is negative — meaning there's no true "shortest" path, since you could loop forever to decrease cost).

## Where This Connects

Weight is independent of [[adsa-directed-undirected-graphs]] and [[adsa-cyclic-acyclic-graphs]] — any combination is possible (e.g. a weighted DAG is common in scheduling where edges represent task duration).

#adsa
