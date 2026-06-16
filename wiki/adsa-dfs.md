---
# DFS (Depth-First Search) — Graphs

**Summary**: DFS explores a graph by going as deep as possible down one path before backtracking, using a stack (explicit or via recursion). On graphs (unlike trees) it requires a visited set to avoid looping forever on cycles.

**Pre-Req**: [[adsa-graphs]], [[adsa-stacks]], [[adsa-tree-traversal]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

DFS visits a vertex, then immediately recurses into one unvisited neighbor, continuing depth-first until no unvisited neighbor remains — then backtracks. It uses a **stack** (explicit, or the call stack via recursion) to hold the path being explored.

As with [[adsa-bfs]], a graph's cycles mean DFS needs a **visited set**; without it, a cycle would cause infinite recursion/looping.

```
function DFS(graph, start, visited = {}):
    visited.add(start)
    visit(start)
    for neighbor in graph[start]:
        if neighbor not in visited:
            DFS(graph, neighbor, visited)
```

Iterative (explicit stack) version:
```
function DFS(graph, start):
    visited = {start}
    stack = [start]
    while stack is not empty:
        v = pop(stack)
        visit(v)
        for neighbor in graph[v]:
            if neighbor not in visited:
                visited.add(neighbor)
                push(stack, neighbor)
```

## Illustration

```
    A — B — D
    |       |
    C ——————E
```

Starting DFS from A (visiting neighbors in alphabetical order):

| Step | Visit | Action |
|---|---|---|
| 1 | A | go to B (first unvisited neighbor) |
| 2 | B | go to D |
| 3 | D | go to E |
| 4 | E | C is a neighbor but unvisited — go to C |
| 5 | C | all neighbors (A, E) visited — backtrack fully |

**Visit order**: A → B → D → E → C (depth-first, one full path before others)

Without the visited set, stepping from E back toward C, then C's neighbor A, would re-trigger A → B → D → E → C endlessly since the graph has a cycle (A-C-E-D-B-A).

## Advantages

- Low memory footprint: O(h) where h is the depth of the current path, vs BFS's O(V) frontier — much cheaper on deep, narrow graphs.
- Naturally suited to problems framed as "find *a* path" rather than "find the *shortest* path" (e.g. maze solving, backtracking puzzles).
- Directly supports algorithms built on visit-order: topological sort, cycle detection ([[adsa-cyclic-acyclic-graphs]]), connected components.

## Disadvantages

- No shortest-path guarantee — may find a long, winding path before a short one.
- Recursive implementations risk stack overflow on very deep/large graphs (mitigated by the iterative explicit-stack form).
- Can get "stuck" going deep down an unproductive branch before backtracking, wasting work compared to BFS when the target is shallow.

## Example

Solving a maze: DFS commits to one corridor, following it to a dead end before backtracking to try the next branch — it doesn't need to fully explore every room at a given distance first, unlike BFS.

## Use Cases

- Cycle detection in graphs ([[adsa-cyclic-acyclic-graphs]])
- Topological sort for DAGs ([[adsa-dag]])
- Maze/puzzle solving, backtracking algorithms (N-Queens, Sudoku)
- Finding connected components / strongly connected components (Tarjan's, Kosaraju's)
- Path existence checks where any valid path (not necessarily shortest) is acceptable

#adsa
