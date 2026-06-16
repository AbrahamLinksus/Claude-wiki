---
# Cyclic vs Acyclic Graphs

**Summary**: A cyclic graph contains at least one path that loops back to a vertex already visited; an acyclic graph has no such path.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Cyclic Graphs

A graph is cyclic if there exists a path that starts and ends at the same vertex without repeating edges.

- **Directed cycle**: follows edge direction, e.g. A→B→C→A
- **Undirected cycle**: any loop back via edges, e.g. A-B-C-A

```
Directed cycle:        Undirected cycle:
A → B                  A - B
↑     \                 \ /
C  ←---                  C
```

## Acyclic Graphs

A graph is acyclic if no such loop exists — every path eventually dead-ends without returning to a prior vertex.

- **Directed + acyclic** = **DAG** (Directed Acyclic Graph)
- **Undirected + acyclic + connected** = a tree (see [[adsa-binary-trees]] for the special case where every node has at most 2 children)

## DAGs: Why They Matter

DAGs show up constantly in real systems because they encode dependency relationships with no circular requirement:

| Use case | Vertices | Edges |
|---|---|---|
| Task scheduling | Tasks | "must finish before" |
| Build systems (Make, Bazel) | Build targets | Dependencies |
| Git commit history | Commits | Parent links |
| Spreadsheet formulas | Cells | "depends on" |

A cycle in any of these would be a logical contradiction — e.g. task A depending on task B which depends on task A can never be scheduled. This is exactly why **topological sort** (ordering vertices so every edge points forward) only works on DAGs.

## Combined Classification Table

| | **Directed** | **Undirected** |
|---|---|---|
| **Cyclic** | A→B→C→A | A-B-C-A |
| **Acyclic** | DAG | Tree (if connected) |

#adsa
