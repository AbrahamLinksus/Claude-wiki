---
# BFS (Breadth-First Search) — Graphs

**Summary**: BFS explores a graph level by level using a queue, visiting all neighbors of a vertex before moving to the next depth. On graphs (unlike trees) it requires a visited set to avoid re-processing vertices in cycles.

**Pre-Req**: [[adsa-graphs]], [[adsa-queues]], [[adsa-tree-traversal]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

BFS visits vertices in order of increasing distance from the start vertex. It uses a **queue** to hold the frontier (discovered, not-yet-visited vertices) — the oldest discovered vertex is always processed next, so all vertices at distance `d` are visited before any vertex at distance `d+1`.

Because a graph can contain cycles ([[adsa-cyclic-acyclic-graphs]]), BFS must track a **visited set** — without it, a vertex could be re-enqueued forever via a cycle.

```
function BFS(graph, start):
    visited = {start}
    queue = [start]
    while queue is not empty:
        v = dequeue(queue)
        visit(v)
        for neighbor in graph[v]:
            if neighbor not in visited:
                visited.add(neighbor)
                enqueue(queue, neighbor)
```

## Illustration

```
    A — B — D
    |       |
    C ——————E
```

Starting BFS from A:

| Step | Dequeue | Visit | Enqueue (newly discovered) |
|---|---|---|---|
| 1 | A | A | B, C |
| 2 | B | B | D |
| 3 | C | C | E |
| 4 | D | D | — (E already discovered) |
| 5 | E | E | — |

**Visit order**: A → B → C → D → E (level by level: distance 0, then 1, then 2)

Note `C` enqueues nothing new at step 3 except `E`, and `D`/`E` connect back — without the visited set, `E` would be enqueued twice (once from D, once from C) and could cause infinite re-processing if a cycle fed back to an unvisited check that wasn't tracked.

## Advantages

- Guarantees the **shortest path** (fewest edges) from the start vertex in an **unweighted** graph.
- Predictable, level-by-level exploration — useful when "closest first" matters.
- Iterative by nature (queue-based), so no recursion-depth/stack-overflow risk on deep graphs.

## Disadvantages

- Space cost: O(V) in the worst case, since an entire level (which can be most of the graph) may sit in the queue at once.
- Does not generalize to **weighted shortest path** — a vertex reached via fewer hops isn't necessarily reached via lower total cost (need Dijkstra/Bellman-Ford instead, see [[adsa-weighted-unweighted-graphs]]).
- Wastes work exploring "sideways" when the goal is known to be deep in one direction (DFS or A* may be more direct).

## Example

Finding the shortest number of hops between two people in a social network: starting from person A, BFS will reach their direct friends first (1 hop), then friends-of-friends (2 hops), etc. — the first time the target is dequeued, that distance is guaranteed minimal.

## Use Cases

- Shortest path in unweighted graphs (e.g. minimum number of moves in a puzzle)
- Web crawlers (visit pages level by level from a seed URL)
- Social network "degrees of separation" / friend suggestions
- Finding all nodes reachable within k steps
- Broadcasting/network layer protocols (e.g. routing table construction)

#adsa
