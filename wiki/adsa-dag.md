---
# DAG (Directed Acyclic Graph)

**Summary**: A DAG is a graph that is both directed and acyclic — edges have direction, and no path ever loops back to a vertex already visited. This combination guarantees a valid processing order always exists.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

A DAG combines two properties covered in [[adsa-directed-undirected-graphs]] and [[adsa-cyclic-acyclic-graphs]]:
- **Directed** — every edge has a one-way direction
- **Acyclic** — no path ever returns to a vertex it already visited

```
A → B → D
 \       ↑
  → C ---┘
```

Because there's no cycle, the vertices can always be arranged in a line where every edge points forward — this ordering is called a **topological sort**.

## Advantages

- Guarantees a topological ordering exists — there's always at least one valid sequence to process every vertex such that dependencies come first.
- No circular-dependency risk by construction.
- Many problems (longest path, shortest path, dependency resolution) become solvable in O(V + E) on a DAG without needing more expensive general-graph algorithms.

## Disadvantages

- Cannot represent mutual or cyclic dependencies (e.g. "A needs B and B needs A" is impossible to express).
- Maintaining the DAG property costs extra work: every new edge must be checked to confirm it won't create a cycle before insertion.

## Representation

Same adjacency list as any directed graph (see [[adsa-directed-undirected-graphs]]), commonly paired with an **in-degree array** (count of incoming edges per vertex) — this is what Kahn's algorithm uses to find a topological order.

```
graph:      {A: [B, C], B: [D], C: [D], D: []}
in-degree:  {A: 0, B: 1, C: 1, D: 2}
```

## Pseudocode for Creation (with Cycle Prevention)

```
function addEdgeDAG(graph, u, v):
    graph[u].append(v)
    if hasCycleDirected(graph):      # see adsa-cyclic-acyclic-graphs
        graph[u].remove(v)
        raise Error("edge would create a cycle — not a valid DAG")
```

### Topological Sort (Kahn's Algorithm)

The canonical algorithm associated with DAGs — produces a valid processing order:

```
function topologicalSort(graph):
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
        raise Error("graph has a cycle — not a DAG")
    return order
```

If the resulting `order` doesn't contain every vertex, the graph had a cycle — this doubles as a cycle-detection method.

See [[adsa-toposort]] for the dedicated page covering this Kahn's algorithm dry run plus the alternative DFS-based method.

## Use Cases

- Build systems (Make, Bazel) — build targets and their dependencies
- Task scheduling — "must finish before" relationships
- Course prerequisite ordering
- Spreadsheet formula evaluation order
- Package dependency resolution (npm, pip)

## Examples

- A Makefile/Bazel dependency graph
- University course prerequisites (can't take Algorithms II before Algorithms I)
- An npm/pip dependency tree
- Git commit history (each commit points to its parent(s); no commit can be its own ancestor)

#adsa
