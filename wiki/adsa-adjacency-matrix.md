---
# Adjacency Matrix

**Summary**: An adjacency matrix represents a graph as a V×V grid, where `matrix[i][j]` indicates whether (and how strongly) vertex i connects to vertex j.

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

A V×V matrix where each cell `matrix[i][j]` encodes the edge between vertex i and vertex j.

- **Unweighted**: `matrix[i][j] = 1` if an edge exists, `0` otherwise.
- **Weighted** (see [[adsa-weighted-unweighted-graphs]]): `matrix[i][j] = weight` if an edge exists, `∞` (or 0) otherwise.

```
Graph: A-B, A-C, B-C (undirected)

      A  B  C
   A [ 0  1  1 ]
   B [ 1  0  1 ]
   C [ 1  1  0 ]
```

## Key Properties

- **Undirected** (see [[adsa-directed-undirected-graphs]]) → matrix is symmetric: `matrix[i][j] == matrix[j][i]`.
- **Directed** → not necessarily symmetric; row i = outgoing edges from i, column i = incoming edges to i.
- **Self-loops** (see [[adsa-cyclic-acyclic-graphs]]) appear on the diagonal: `matrix[i][i] = 1` (or a weight). In an undirected graph this is sometimes stored as `2` instead, to reflect that the self-loop contributes 2 to the vertex's degree.
- Edge existence check: **O(1)** — direct index lookup.
- Iterating all neighbors of a vertex: **O(V)** — must scan the entire row, even if the vertex has only one neighbor.
- Space: always **O(V²)**, regardless of actual edge count.

## Advantages

- O(1) edge lookup — checking "does an edge exist between u and v" is a single array access.
- Simple to implement and reason about, especially for dense graphs.
- Naturally supports weighted graphs by storing the weight directly in the cell.
- Easy to detect symmetry (undirected) or asymmetry (directed) by inspection.

## Disadvantages

- O(V²) space even for sparse graphs — wasteful when most possible edges don't exist (e.g. a graph with 1000 vertices but only 50 edges still needs a 1000×1000 matrix).
- O(V) to enumerate a single vertex's neighbors, vs O(degree) for an adjacency list — slower traversal (BFS/DFS) on sparse graphs.
- Adding/removing a vertex is expensive — requires resizing the whole matrix (O(V²) to rebuild).

## Pseudocode for Creation

```
function createAdjacencyMatrix(n):           # n = number of vertices
    matrix = [[0] * n for _ in range(n)]
    return matrix

function addEdge(matrix, u, v, weight = 1, directed = False):
    matrix[u][v] = weight
    if not directed:
        matrix[v][u] = weight

function hasEdge(matrix, u, v):
    return matrix[u][v] != 0
```

## Use Cases

- Dense graphs, where the number of edges approaches V² (e.g. fully or near-fully connected networks).
- Algorithms that repeatedly ask "is there an edge between u and v?" (benefits from O(1) lookup).
- Small, fixed-size graphs where O(V²) memory is not a concern.
- Graph algorithms expressed naturally in matrix form (e.g. computing connectivity via matrix multiplication, Floyd-Warshall all-pairs shortest path).

## Examples

- A small social circle (≤ a few hundred people) where you frequently check "are X and Y connected?"
- Floyd-Warshall all-pairs shortest path, which operates directly on a weighted adjacency matrix.
- A complete graph (every vertex connected to every other) — adjacency list gives no space savings here, so the matrix is the natural choice.

## Where This Connects

The adjacency matrix is one of the two standard ways to store any graph from [[adsa-graphs]] — the other being the adjacency list, which trades O(1) edge lookup for better space and faster neighbor iteration on sparse graphs.

#adsa
