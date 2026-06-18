# Prim's Algorithm

**Summary**: Vertex-centric greedy MST algorithm — grows a single tree from a start vertex, repeatedly adding the cheapest edge that crosses from the visited set to the unvisited set.

**Pre-Req**: [[adsa-mst]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-18

---

## Algorithm

```
function primMST(graph, start):
    visited = {start}
    mst = []
    pq = MinHeap of (weight, u, v) for all edges from start
    while len(visited) < V:
        weight, u, v = pq.popMin()
        if v in visited: continue
        visited.add(v)
        mst.append((u, v, weight))
        for each edge (v, w, wt) where w not in visited:
            pq.push((wt, v, w))
    return mst
```

## Why It's Correct — The Cut Property

For any way of splitting the graph's vertices into two non-empty groups S and V−S, the minimum-weight edge crossing that split must belong to some MST (assuming distinct weights, or ties broken consistently).

**Why**: suppose the min crossing edge `e` were *not* in the MST. The MST must still cross from S to V−S somehow — say via some other edge `f`, with weight(f) ≥ weight(e). Swapping `f` out for `e` still yields a valid spanning tree, with total weight no higher. So `e` can always be substituted in without making things worse — it's always safe to add.

At every step of Prim's, the current tree (S) and everything else (V−S) form exactly such a cut. The edge picked — minimum weight, not yet visited — is by construction the minimum weight edge crossing that cut, so the Cut Property guarantees it's safe. Chaining safe choices together still produces a global optimum, even though each decision is made locally.

This is what makes greedy work for MST specifically, unlike problems such as knapsack or general TSP where a locally best choice can lock out the global optimum — the set of spanning trees forms a matroid, and the greedy-choice property is a proven theorem here, not a heuristic.

## Optimality

Always returns a spanning tree of minimum total weight, on any connected weighted undirected graph. If edge weights aren't all distinct, multiple MSTs of equal weight can exist — Prim's still finds one of them, and it's always optimal.

## Complexity

With a binary heap: **O(E log V)**.

#adsa
