# Bellman-Ford Algorithm

**Summary**: Single-source shortest-path algorithm that handles negative edge weights by relaxing every edge V−1 times — slower than Dijkstra's but correct where Dijkstra's breaks, and able to detect negative-weight cycles.

**Pre-Req**: [[adsa-weighted-unweighted-graphs]], [[adsa-dijkstras-algorithm]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-19

---

## The Problem

Given a weighted graph (edges may be negative) and a source vertex, find the minimum total-weight path from the source to every other vertex — or correctly report that no such minimum exists because a negative-weight cycle is reachable.

## Core Idea

[[adsa-dijkstras-algorithm]] greedily locks in the smallest unvisited distance, betting that once popped, that distance can never be beaten later. That bet only holds with non-negative weights. Bellman-Ford makes no such bet — it never "locks in" anything early. Instead it repeatedly **relaxes every edge in the graph**, for enough rounds that any genuine shortest path, however it's shaped, has had time to fully propagate.

## Why V−1 Relaxation Passes

A shortest path in a graph with no negative cycle is always a **simple path** (no repeated vertex) — otherwise the repeated segment could be cut out for a cheaper path. A simple path through `V` vertices has at most `V−1` edges. So after `V−1` full passes of relaxing every edge, every shortest path — no matter how many hops it takes — has been fully built up one edge at a time.

## Algorithm

```
function bellmanFord(V, edges, source):
    dist = [infinity] * V
    dist[source] = 0

    repeat (V - 1) times:
        for each edge (u, v, weight) in edges:
            if dist[u] + weight < dist[v]:
                dist[v] = dist[u] + weight

    // one extra pass to detect negative cycles
    for each edge (u, v, weight) in edges:
        if dist[u] + weight < dist[v]:
            return "negative-weight cycle detected"

    return dist
```

Unlike Dijkstra's, there's no heap, no visited set, and **no required edge-processing order** within a pass — any order converges to the same final `dist[]` after enough passes, since relaxation only ever tightens a value, never loosens it.

## Correctness Proof

**Claim**: After `k` passes, `dist[v] ≤` the true shortest-path weight using **at most `k` edges**, for every vertex `v`.

**Base case** (`k = 0`): `dist[source] = 0` (a 0-edge path), everything else `∞` — trivially true for 0-edge paths.

**Inductive step**: Assume the claim holds after `k−1` passes. Consider the shortest path to `v` using at most `k` edges. Either:
- it uses **fewer than `k` edges** — already correct by the inductive hypothesis, and relaxation never increases `dist[]`, so it stays correct, or
- it uses **exactly `k` edges**, ending with edge `(u, v, weight)`. The prefix to `u` is then a shortest path using at most `k−1` edges, so by the inductive hypothesis `dist[u]` is already correct *before* pass `k`. Since pass `k` relaxes every edge including `(u, v, weight)`, it sets `dist[v] ≤ dist[u] + weight`, which equals the true shortest-path weight.

By induction, after `V−1` passes, `dist[v]` is correct for every shortest path using at most `V−1` edges — which, in a graph with no negative cycle, is *every* shortest path. ∎

## Detecting Negative-Weight Cycles

If a graph has **no** negative cycle, `dist[]` stabilizes by pass `V−1` — a `V`-th pass changes nothing. If an edge can *still* be relaxed on an additional pass, the only way a path can keep improving past `V−1` edges is by looping through a cycle that keeps subtracting cost — i.e. a **negative-weight cycle** reachable from the source. This is exactly what [[adsa-weighted-unweighted-graphs]] flags as the case where "shortest path" stops being well-defined: a negative cycle can be looped indefinitely to drive the total toward `-∞`.

## Dry Run

```
Graph:    A --4--> B --(-3)--> C --4--> D
          |                    ^
          5--------------------|
          A --5--> C       B --5--> D

Source: A
Edges (in processing order): (A,B,4) (A,C,5) (B,C,-3) (C,D,4) (B,D,5)
```

| Pass | Edge | Check | Update | dist after pass |
|---|---|---|---|---|
| 1 | A→B (4) | 0+4=4 < ∞ | B=4 | A:0 B:4 C:∞ D:∞ |
| 1 | A→C (5) | 0+5=5 < ∞ | C=5 | A:0 B:4 C:5 D:∞ |
| 1 | B→C (-3) | 4-3=1 < 5 | C=1 | A:0 B:4 C:1 D:∞ |
| 1 | C→D (4) | 1+4=5 < ∞ | D=5 | A:0 B:4 C:1 D:5 |
| 1 | B→D (5) | 4+5=9 ≥ 5 | — | A:0 B:4 C:1 D:5 |
| 2 | all edges | no check improves | — | unchanged (converged) |
| 3 | all edges | no check improves | — | unchanged |

Final: **A:0 B:4 C:1 D:5**. Shortest A→D is `A→B→C→D` = `4 + (-3) + 4 = 5`, correctly found even though it uses a negative edge that Dijkstra's heap-based "lock in on pop" logic could not safely handle. The graph converged after just 1 pass here — `V−1` passes is only the *worst-case* guarantee, not the typical case. The extra pass run for negative-cycle detection finds no further improvement, confirming no negative cycle exists.

## Complexity

| Step | Cost |
|---|---|
| Relaxation passes | `O(V)` passes × `O(E)` per pass |
| **Total** | **O(V · E)** |
| Negative-cycle check | +1 extra pass, `O(E)` |

Significantly worse than Dijkstra's `O(E log V)` on dense or large graphs — the price paid for correctness under negative weights and for cycle detection.

## Bellman-Ford vs Dijkstra's

| | Dijkstra's | Bellman-Ford |
|---|---|---|
| Negative weights | Breaks (proof relies on non-negative relaxation) | Handles correctly |
| Negative cycles | Undefined / infinite loop risk | Explicitly detected |
| Strategy | Greedy, locks in on pop | Brute-force relaxation, no locking |
| Edge processing order | Must be heap-ordered (smallest dist first) | Any order works |
| Complexity | O(E log V) | O(V · E) |
| Data structure | Min-heap | None (plain edge list) |

Both solve the same Bellman equation `dist[v] = min over edges (u,v) of (dist[u] + weight(u,v))` — see [[adsa-dijkstras-algorithm]]'s DP framing. Dijkstra's earns its speed by exploiting non-negative weights to guarantee a safe processing order; Bellman-Ford gives up that order entirely and instead just relaxes enough times that the order stops mattering.

#adsa
