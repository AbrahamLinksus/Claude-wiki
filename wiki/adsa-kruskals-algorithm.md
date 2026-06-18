# Kruskal's Algorithm

**Summary**: Edge-centric greedy MST algorithm — sort all edges by weight ascending, then add each edge unless it would form a cycle, stopping once V−1 edges are chosen.

**Pre-Req**: [[adsa-mst]], [[adsa-union-find]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-18

---

## Algorithm

```
function kruskalMST(graph):
    edges = sortByWeightAscending(graph.edges)
    mst = []
    for (u, v, weight) in edges:
        if not formsCycle(mst, u, v):
            mst.append((u, v, weight))
        if len(mst) == V - 1:
            break
    return mst
```

## Cycle Detection — Union-Find (Disjoint Set Union)

The `formsCycle` check is implemented with Union-Find:

- Each vertex starts in its own set.
- `find(x)` returns the representative (root) of x's set.
- `union(u,v)` merges the two sets.
- Adding edge (u,v) creates a cycle **iff** `find(u) == find(v)` (they're already connected some other way).

```
function find(parent, x):
    while parent[x] != x:
        x = parent[x]
    return x

function union(parent, u, v):
    ru, rv = find(parent, u), find(parent, v)
    if ru == rv:
        return False   # would form a cycle
    parent[ru] = rv
    return True
```

With path compression + union by rank, `find`/`union` are nearly O(1) amortized.

## Why It's Correct — The Cycle Property

The maximum-weight edge in any cycle is never part of an MST: if it were, removing it and reconnecting via any other edge in that cycle would produce a spanning tree of equal or lower weight. Kruskal's skips exactly these edges (any edge that would close a cycle on the edges chosen so far is, among the choices available, never required), so every edge it discards is safe to discard.

## Complexity

Dominated by the sort: **O(E log E)**. Union-Find operations are effectively O(1) amortized per edge.

#adsa
