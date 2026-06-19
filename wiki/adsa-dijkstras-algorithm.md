# Dijkstra's Algorithm

**Summary**: Greedy single-source shortest-path algorithm for non-negative weighted graphs — repeatedly locks in the unvisited vertex with the smallest known distance, using a min-heap for fast extraction.

**Pre-Req**: [[adsa-weighted-unweighted-graphs]], [[adsa-prims-algorithm]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-19

---

## The Problem

Given a weighted graph with non-negative edges and a source vertex, find the minimum total-weight path from the source to every other vertex.

## Core Idea

Greedy, structurally close to [[adsa-prims-algorithm]] — but the priority isn't "cheapest edge crossing the cut," it's **"smallest known cumulative distance from the source."**

Maintain a `dist[]` array (0 for the source, ∞ for everyone else). Repeatedly pull the *unvisited* vertex with the smallest `dist[]`, lock it in as final, then **relax** its neighbors — check whether routing through it beats their current best.

## Why a Min-Heap

The "pull smallest unvisited dist" step is the bottleneck. A linear scan costs O(V) per extraction → O(V²) total. A **min-heap keyed on `dist[]`** drops extraction to O(log V); since each edge triggers at most one push, total cost is **O(E log V)** — same complexity class as Prim's, for the same structural reason.

## Algorithm

```
function dijkstra(graph, source):
    dist = {v: infinity for v in graph}
    dist[source] = 0
    pq = MinHeap of (0, source)
    visited = set()

    while pq not empty:
        d, u = pq.popMin()
        if u in visited: continue
        visited.add(u)

        for each edge (u, v, weight) in graph[u]:
            if d + weight < dist[v]:
                dist[v] = d + weight
                pq.push((dist[v], v))

    return dist
```

The heap can hold **stale entries** for a vertex (pushed before a shorter path was found) — the `if u in visited: continue` guard makes it safe to discard them rather than needing a "decrease-key" operation.

## Why Greedy Works — and Why Non-Negative Weights Matter

When `u` is popped with the smallest tentative distance, that distance is **final** — no other path to `u` can be shorter, because any alternative path must pass through some other unvisited vertex first, and every unvisited vertex already has `dist[] ≥ dist[u]` (that's why `u` was popped first, not it). Stacking more non-negative weights onto a value already ≥ `dist[u]` cannot produce something smaller than `dist[u]`.

This breaks immediately with negative weights: a vertex can be locked in as "final," then a later negative edge could reveal a shorter path through a vertex that was already visited — but by then it's too late to revise. This is exactly why **Bellman-Ford** exists for negative weights — it never locks anything in early; it just relaxes every edge V−1 times.

## Dry Run

```
Graph:        A --4-- B
              |       |
              1       2
              |       |
              C --5-- D

Source: A
```

| Step | Pop (d, u) | Relax | dist after |
|---|---|---|---|
| 1 | (0, A) | B: 0+4=4, C: 0+1=1 | A:0, B:4, C:1, D:∞ |
| 2 | (1, C) | D: 1+5=6 | B:4, C:1, D:6 |
| 3 | (4, B) | D: 4+2=6 (tie, no update) | D stays 6 |
| 4 | (6, D) | — done | **A:0, B:4, C:1, D:6** |

Shortest path A→D ties between via-B (4+2=6) and via-C (1+5=6) — Dijkstra finds *a* shortest path, not a unique one when ties exist.

## Complexity

| Implementation | Time |
|---|---|
| Linear scan for min | O(V²) |
| Binary min-heap | **O(E log V)** |
| Fibonacci heap | O(E + V log V) |

## Dijkstra vs Prim's — Structural Twins

| | Prim's (MST) | Dijkstra (SSSP) |
|---|---|---|
| Priority key | weight of edge crossing the cut | total distance from source |
| Greedy choice | cheapest edge into the tree | smallest cumulative distance |
| Correctness proof | Cut Property | non-negative relaxation argument |
| Heap entries | (weight, vertex) | (distance, vertex) |
| Complexity | O(E log V) | O(E log V) |

The only real difference is *what number is compared*: isolated edge weight (Prim's) vs. accumulated path cost (Dijkstra's) — which is exactly why Dijkstra breaks on negative weights while Prim's has no such failure mode (MST minimizes total weight directly, with no path-accumulation logic to corrupt).

## Theory: Relaxation as a Monotone Invariant

Define `dist[v]` at any point in the algorithm as the length of the shortest path found *so far* using only visited vertices as intermediates. Relaxation enforces one invariant: `dist[v]` only ever decreases, never increases. Combined with non-negative weights, this gives a **monotone, well-ordered frontier** — the set of finalized distances expands outward in strictly non-decreasing order, like ripples from a stone dropped in water. This is the same mental model as BFS's "wavefront," generalized from unit edge weight to arbitrary non-negative weight.

In fact, **Dijkstra's algorithm degenerates to plain BFS when every edge weight is 1** — the min-heap, at that point, behaves like a FIFO queue, because all neighbors at the same "ring" get popped before anything farther out. This is why [[adsa-bfs]] is correct for unweighted shortest paths: it's Dijkstra's special case, not a separate technique.

### Exchange-Argument / Optimal-Substructure View

Dijkstra also has a dynamic-programming reading: `dist[v] = min over all neighbors u of (dist[u] + weight(u, v))`. This is the **Bellman equation** for shortest paths — the same recursive structure Bellman-Ford solves by brute-force relaxation (no ordering assumptions) and the same one Floyd-Warshall solves for *all pairs* via a different recurrence. Dijkstra's contribution over plain DP is the **processing order**: by always expanding the globally-smallest frontier vertex next, it guarantees each subproblem (`dist[u]`) is already optimal before it's used to compute others — eliminating the need to revisit or re-relax anything. That ordering guarantee is precisely what non-negative weights make possible, and precisely what negative weights destroy.

### Generalization: Priority-First Search

Dijkstra's, Prim's, and even A* search are instances of a single template — **priority-first search**: explore the graph via a min-heap keyed on some cost function, finalize the minimum on each pop, relax neighbors. The differences are only in the key:
- Prim's: edge weight to the tree.
- Dijkstra's: `g(v)` — cumulative path cost from source.
- A*: `g(v) + h(v)` — cumulative cost plus a heuristic estimate to the goal, which lets it prune the search space toward a single target instead of computing distances to every vertex.

This shared template is why all three share the same O(E log V) heap-based complexity profile and the same "lock in on pop" correctness argument, with A*'s extra correctness condition being that `h` must be *admissible* (never overestimate the true remaining cost).

#adsa
