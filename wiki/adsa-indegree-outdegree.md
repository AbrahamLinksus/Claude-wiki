---
# Indegree and Outdegree

**Summary**: Indegree is the number of edges pointing into a vertex; outdegree is the number of edges pointing out of it. Both only apply to directed graphs — undirected graphs have a single undifferentiated "degree" per vertex.

**Pre-Req**: [[adsa-graphs]], [[adsa-directed-undirected-graphs]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

For a vertex `v` in a **directed graph**:

- **Indegree(v)** — count of edges where `v` is the destination (edges pointing *into* `v`)
- **Outdegree(v)** — count of edges where `v` is the source (edges pointing *out of* `v`)

```
A → B → C
    ↑
    D

Indegree(B)  = 2   (from A, from D)
Outdegree(B) = 1   (to C)
```

In an **undirected graph**, an edge has no direction, so there's just **degree(v)** — the count of all edges touching `v`. The indegree/outdegree split doesn't apply (source: session discussion, [[adsa-directed-undirected-graphs]]).

## Computing from Each Representation

| Representation | Outdegree(v) | Indegree(v) |
|---|---|---|
| **Adjacency List** | `len(graph[v])` — O(1) if cached, O(degree) to count | Must scan all lists looking for `v`, or maintain a running counter on insert — O(V+E) total |
| **Adjacency Matrix** | Sum of row `v` | Sum of column `v` |

```
Adjacency Matrix:
        A  B  C
    A [ 0  1  0 ]   Outdegree(A) = 1 (row sum)
    B [ 0  0  1 ]   Indegree(C)  = 1 (column sum of C)
    C [ 0  0  0 ]
```

This is why adjacency list is asymmetric in cost: outdegree is cheap (it's literally the list you already have), but indegree is expensive unless you track it separately as edges are added.

## Handshake Identity (Directed Version)

For any directed graph:

```
sum(indegree(v) for all v) == sum(outdegree(v) for all v) == E
```

Every edge contributes exactly one unit of outdegree (at its source) and one unit of indegree (at its destination) — so both totals always equal the edge count.

## Use Case: Topological Sort

A vertex with **indegree 0** has no prerequisites — nothing points into it. Kahn's algorithm for topological sort (see [[adsa-dag]]) repeatedly removes indegree-0 vertices, decrementing the indegree of their neighbors as edges are "removed," producing a valid ordering.

This is the practical reason indegree gets tracked explicitly in many graph implementations rather than computed on demand.

#adsa
