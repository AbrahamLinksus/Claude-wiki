---
# Explicit vs Implicit Graphs

**Summary**: An explicit graph is fully stored in memory before any algorithm runs; an implicit graph is generated on the fly, vertex by vertex, as the algorithm explores.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

**Explicit graph**: the complete vertex and edge set exists in memory upfront, stored as an adjacency list/matrix (see [[adsa-graphs]]). The algorithm just reads this pre-built structure.

```
graph = {A: [B, C], B: [A], C: [A]}   ← already fully built
```

**Implicit graph**: no graph is stored. Instead, a rule or function defines what the neighbors of any given vertex are, and the graph is generated lazily — only the parts actually visited get computed.

```
function neighbors(state):
    return [apply_move(state, m) for m in legal_moves(state)]
```

The vertex set may be too large (or even infinite) to ever fully materialize — it only "exists" conceptually until something asks for a specific vertex's neighbors.

## Advantages

**Explicit:**
- Fast, repeated O(1)/O(degree) lookups once built.
- Easy to inspect, debug, and visualize the whole structure.

**Implicit:**
- No upfront memory cost — works even when the state space is astronomically large or infinite (e.g. chess positions, Rubik's cube states).
- Naturally fits problems where the graph is really a rule, not data (e.g. "neighbors of this maze cell" or "next board states from this move").

## Disadvantages

**Explicit:**
- Must fit in memory before you start — impossible for huge or infinite state spaces.
- Wasted work/space building parts of the graph that the algorithm never visits.

**Implicit:**
- Repeated neighbor generation can be more expensive per step than an array/list lookup.
- Requires a visited-set (often a hash set, since vertices aren't pre-indexed) to avoid revisiting states and looping forever.

## Representation

| | Explicit | Implicit |
|---|---|---|
| **Stored as** | Adjacency list / matrix | A generator function: `neighbors(vertex) → list of vertices` |
| **Vertex set known in advance?** | Yes | No — discovered as you go |
| **Visited tracking** | Often an array/boolean flag indexed by vertex ID | Usually a hash set, since vertices are values (e.g. board states), not array indices |

## Pseudocode

**Explicit** — just iterate the pre-built structure (see [[adsa-graphs]] for creation pseudocode):
```
function neighborsExplicit(graph, u):
    return graph[u]
```

**Implicit** — generate neighbors on demand, typically inside a BFS/DFS-style search:
```
function search(start):
    visited = {start}
    frontier = [start]
    while frontier:
        u = frontier.pop()
        if isGoal(u):
            return u
        for v in neighbors(u):          # generated, not looked up
            if v not in visited:
                visited.add(v)
                frontier.append(v)
```

## Use Cases

- **Explicit**: road maps, social networks, any dataset-backed graph that's small/static enough to load fully
- **Implicit**: puzzle solving (8-puzzle, Rubik's cube, sliding puzzles), game tree search (chess, tic-tac-toe), maze/grid pathfinding generated from rules, state-space search in AI planning

## Examples

- Explicit: a city's road network loaded from a `.csv` of roads — every vertex and edge exists in memory before pathfinding starts.
- Implicit: solving the 8-puzzle — there's no stored "puzzle graph"; each state's neighbors (one slide move away) are computed only when the search reaches that state.
- Implicit: a chess engine searching possible games — the game tree has more nodes than atoms in the universe; it's explored implicitly, a few moves at a time.

## Where This Connects

This distinction matters most once we get to [[adsa-graphs]] traversal (BFS/DFS) — the algorithms are the same shape, but implicit graphs need lazy neighbor generation plus visited-tracking by value instead of by index. It's also the foundation for state-space search in AI planning and game-playing agents.

#adsa
