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

## Why Greedy Works — Formal Proof

### Lemma: Subpaths of Shortest Paths Are Shortest Paths

If `P` is a shortest path from `source` to `v`, and `x` is any vertex on `P`, the portion of `P` from `source` to `x` is itself a shortest path to `x`.

**Why**: if a cheaper path to `x` existed, splicing it with the rest of `P` (from `x` to `v`) would give a path to `v` cheaper than `P` — contradicting that `P` is shortest. This relies on non-negative weights, so "the rest of `P`" is unaffected by the swap.

### Claim

When the algorithm pops vertex `v` with heap value `dist[v]`, that value equals `δ(source, v)`, the true shortest-path distance — for every `v`, not just direct neighbors of the source.

### Proof — Induction on Pop Order

Let `u₁, u₂, u₃, …` be the order vertices are popped (finalized). By the heap property, `dist[u₁] ≤ dist[u₂] ≤ dist[u₃] ≤ …`.

**Base case**: `u₁ = source`, `dist[source] = 0 = δ(source, source)`. Trivially true.

**Inductive hypothesis**: for the first `k` pops, `dist[uᵢ] = δ(source, uᵢ)` for all `i ≤ k`.

**Inductive step**: let `v` be the `(k+1)`-th pop. Since `dist[v]` is the length of *some* real relaxed path, `dist[v] ≥ δ(source, v)` always holds. The only thing left to rule out is `dist[v] > δ(source, v)`.

Suppose, for contradiction, the true shortest path `P` from `source` to `v` is strictly shorter than `dist[v]`. Walk along `P` from `source`. Since `source` is finalized and `v` is not yet, there must be a **first unfinalized vertex** on `P` — call it `y` (possibly `y = v`).

Let `x` be `y`'s predecessor on `P`. Since `y` is the *first* unfinalized vertex, `x` is already finalized, so by the inductive hypothesis `dist[x] = δ(source, x)`. By the Lemma, the prefix of `P` up to `x` is a shortest path, so `δ(source, x) + weight(x, y) = δ(source, y)`.

When `x` was popped, edge `(x, y)` was relaxed, so:

```
dist[y] ≤ dist[x] + weight(x, y) = δ(source, x) + weight(x, y) = δ(source, y)
```

**Case `y = v`**: then `dist[v] ≤ δ(source, v)`, which combined with `dist[v] ≥ δ(source, v)` forces equality — contradicting the assumption that they differ.

**Case `y ≠ v`**: `y` lies strictly before `v` on `P`. Since all remaining edges from `y` to `v` along `P` have non-negative weight, `δ(source, y) ≤ δ(source, v) < dist[v]`. Chaining: `dist[y] ≤ δ(source, y) < dist[v]`. But `y` is **unfinalized** and `v` was just popped as the *minimum* of everything unfinalized in the heap, so `dist[v] ≤ dist[y]` must hold — contradicting `dist[y] < dist[v]`.

Either case is a contradiction, so `dist[v] > δ(source, v)` is false → **`dist[v] = δ(source, v)` exactly when `v` is popped.** ∎

### The Key Insight

The argument hinges on one fact: **a shortest path can never duck into unfinalized territory and come back out cheaper**, because non-negative weights mean re-entering finalized territory can only add cost, never refund it. The destination's value, the moment it's popped, is already the true minimum — not a heuristic, but a structural consequence of pop order.

### Why Negative Weights Break This

The proof leans on `δ(source, y) ≤ δ(source, v)` for `y` earlier on the shortest path than `v` — true only because all subsequent edge weights from `y` to `v` are non-negative. With a negative edge, a vertex already finalized as "final" could later be reached more cheaply through a path discovered afterward, but by then it's locked in and never revisited. This is exactly why **Bellman-Ford** exists for negative weights — it never locks anything in early; it just relaxes every edge V−1 times.

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
