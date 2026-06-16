---
# Cyclic vs Acyclic Graphs

**Summary**: A cyclic graph contains at least one path that loops back to a vertex already visited; an acyclic graph has no such path.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

A graph is **cyclic** if there exists a path that starts and ends at the same vertex without repeating edges.

- **Directed cycle**: follows edge direction, e.g. A→B→C→A
- **Undirected cycle**: any loop back via edges, e.g. A-B-C-A

```
Directed cycle:        Undirected cycle:
A → B                  A - B
↑     \                 \ /
C  ←---                  C
```

A graph is **acyclic** if no such loop exists — every path eventually dead-ends without returning to a prior vertex.

- **Directed + acyclic** = [[adsa-dag]] (Directed Acyclic Graph)
- **Undirected + acyclic + connected** = a tree (see [[adsa-binary-trees]] for the special case where every node has at most 2 children)

## Advantages

**Cyclic:**
- Can model feedback loops and recurring processes naturally (state machines, retry loops).

**Acyclic:**
- Guarantees a valid processing order exists (topological sort) — useful whenever "do X before Y" relationships matter.
- No risk of infinite traversal loops.
- Easier to reason about: every path has a definite end.

## Disadvantages

**Cyclic:**
- Risk of infinite loops during traversal if visited-tracking is forgotten.
- Harder to reason about a "correct" order of operations.

**Acyclic:**
- Cannot model genuinely circular/feedback relationships.
- Enforcing acyclicity costs extra work when inserting edges (must check the new edge doesn't create a cycle).

## Representation

Same adjacency list/matrix as any graph (see [[adsa-graphs]]). What differs is the **cycle-detection step**, typically run via DFS:
- **Directed graphs**: track a recursion stack; a "back edge" to a vertex currently on the stack means a cycle.
- **Undirected graphs**: track the parent of each vertex; an edge to a visited vertex that isn't the parent means a cycle.

## Pseudocode for Creation (with Cycle Detection, Directed)

```
function hasCycleDirected(graph):
    visited = set()
    inStack = set()

    function dfs(u):
        visited.add(u)
        inStack.add(u)
        for v in graph[u]:
            if v not in visited:
                if dfs(v): return True
            elif v in inStack:
                return True   # back edge -> cycle
        inStack.remove(u)
        return False

    for v in graph:
        if v not in visited:
            if dfs(v): return True
    return False
```

## Use Cases

- **Cyclic**: state machines (traffic lights, game states), circular dependency detection
- **Acyclic (DAG)**: build systems, task scheduling, version control history, spreadsheet recalculation order — see [[adsa-dag]] for the dedicated deep dive

## Examples

- Traffic light state cycle: Red → Green → Yellow → Red (cyclic, directed)
- A Makefile's target dependency graph (acyclic, directed = DAG)
- A simple ring network topology (cyclic, undirected)

## Combined Classification Table

| | **Directed** | **Undirected** |
|---|---|---|
| **Cyclic** | A→B→C→A | A-B-C-A |
| **Acyclic** | [[adsa-dag]] | Tree (if connected) |

#adsa
