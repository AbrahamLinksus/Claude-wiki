---
# Topological Sort

**Summary**: A topological sort is a linear ordering of a directed graph's vertices such that for every edge u → v, u comes before v. It only exists if the graph is a [[adsa-dag]] (no cycles).

**Pre-Req**: [[adsa-dag]], [[adsa-bfs]], [[adsa-dfs]], [[adsa-indegree-outdegree]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

Given a DAG, a topological order arranges all vertices in a sequence so that every directed edge points "forward" in that sequence — no edge ever points back to an earlier vertex.

```
A → B → D
 \       ↑
  → C ---┘
```

Valid topological orders here: `A, B, C, D` or `A, C, B, D` (multiple valid orders can exist — order is not unique unless the DAG happens to be a total order).

A topological order exists **if and only if** the graph is acyclic. A cycle would force some vertex to come both before and after another, which is impossible.

## Method 1 — Kahn's Algorithm (BFS-based)

Repeatedly remove vertices with no remaining incoming edges (in-degree 0).

```
function topoSortKahn(graph):
    inDegree = {v: 0 for v in graph}
    for u in graph:
        for v in graph[u]:
            inDegree[v] += 1

    queue = [v for v in graph if inDegree[v] == 0]
    order = []
    while queue:
        u = queue.pop(0)
        order.append(u)
        for v in graph[u]:
            inDegree[v] -= 1
            if inDegree[v] == 0:
                queue.append(v)

    if len(order) != len(graph):
        raise Error("cycle detected — no valid topological order")
    return order
```

### Dry Run

Graph: `A → B`, `A → C`, `B → D`, `C → D`

| Step | in-degree (A,B,C,D) | queue | pop | order so far |
|---|---|---|---|---|
| start | 0,1,1,2 | [A] | — | [] |
| 1 | 0,0,0,2 | [B,C] | A | [A] |
| 2 | 0,0,0,1 | [C,D]→[C] then D added | B | [A,B] |
| 3 | 0,0,0,0 | [D] | C | [A,B,C] |
| 4 | 0,0,0,0 | [] | D | [A,B,C,D] |

**Result**: `A, B, C, D`

## Method 2 — DFS-based

Run DFS; when a vertex finishes (all its descendants have been visited), push it onto a stack. Popping the stack gives the topological order.

```
function topoSortDFS(graph):
    visited = set()
    stack = []   # holds finished vertices

    function dfs(u):
        visited.add(u)
        for v in graph[u]:
            if v not in visited:
                dfs(v)
        stack.append(u)        # u is "finished" — no unvisited descendants left

    for v in graph:
        if v not in visited:
            dfs(v)

    return stack[::-1]          # reverse = topological order
```

### Why reversing the finish order works

A vertex finishes only after all vertices reachable from it have finished. So in the finish-time stack, `u` always finishes before anything `u` points to *unless* that thing was already finished via another path — either way, `u` ends up below everything it points to in the stack. Reversing puts `u` before its dependents.

### Dry Run

Same graph: `A → B`, `A → C`, `B → D`, `C → D`

```
dfs(A):
  visit A
  dfs(B):
    visit B
    dfs(D):
      visit D
      no neighbors → finish D, stack = [D]
    finish B, stack = [D, B]
  dfs(C):
    visit C
    D already visited → skip
    finish C, stack = [D, B, C]
  finish A, stack = [D, B, C, A]

reverse → [A, C, B, D]
```

**Result**: `A, C, B, D` — a different but equally valid order from Kahn's `A, B, C, D` (both satisfy every edge constraint).

## Kahn's vs DFS-based — Comparison

| | Kahn's (BFS) | DFS-based |
|---|---|---|
| Underlying structure | Queue + in-degree array | Stack (finish-time) + recursion |
| Cycle detection | Order length < V → cycle | Track "in-progress" set; revisiting an in-progress vertex → cycle |
| Natural extra output | Yes — "levels" of independent tasks (good for parallel scheduling) | No — just one linear order |
| Implementation | Iterative, no recursion depth limit | Recursive (or needs explicit stack for deep graphs) |

## Use Cases

- Build systems — compile targets in dependency order
- Task scheduling — sequence steps where some must precede others
- Course prerequisite ordering
- Spreadsheet cell evaluation order (formula dependencies)
- Package manager install order (npm, pip, apt)
- Detecting deadlock / circular dependency (failure to produce a full order signals a cycle)

## Examples

- `pip install` resolving which packages to install before others that depend on them
- A Makefile deciding which targets to build first
- Course registration systems blocking "Algorithms II" until "Algorithms I" is marked complete

#adsa
