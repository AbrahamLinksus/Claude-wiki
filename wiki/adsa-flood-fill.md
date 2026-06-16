---
# Flood Fill

**Summary**: Flood fill visits all cells connected to a starting cell that share a target property (e.g. color), replacing or marking them — it's BFS or DFS run on an implicit grid graph.

**Pre-Req**: [[adsa-bfs]], [[adsa-dfs]], [[adsa-explicit-implicit-graphs]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-16

---

## Definition

Flood fill treats a 2D grid as an **implicit graph**: each cell is a vertex, and each cell's up/down/left/right (4-directional) or +diagonal (8-directional) neighbors are edges — generated on the fly rather than stored explicitly, since the grid coordinates themselves encode adjacency (see [[adsa-explicit-implicit-graphs]]).

Starting from a seed cell, flood fill visits every cell reachable through neighbors that match the seed's original value, replacing each with a new value (or just marking it visited, for connectivity/counting problems).

```
function floodFill(grid, startRow, startCol, newColor):
    oldColor = grid[startRow][startCol]
    if oldColor == newColor: return

    queue = [(startRow, startCol)]
    while queue is not empty:
        (r, c) = dequeue(queue)
        if out of bounds or grid[r][c] != oldColor:
            continue
        grid[r][c] = newColor
        enqueue(queue, (r+1, c)); enqueue(queue, (r-1, c))
        enqueue(queue, (r, c+1)); enqueue(queue, (r, c-1))
```

This is exactly [[adsa-bfs]] (queue version above) or [[adsa-dfs]] (stack/recursion) — the grid itself acts as the visited set, since a cell is only enqueued/processed while it still matches `oldColor`; once repainted to `newColor` it's naturally skipped on revisit.

## Illustration

Grid (0 = white, 1 = blue), flood fill from `(1,1)` with new color 2:

```
Before:              After:
0 0 0 0               0 0 0 0
0 1 1 0               0 2 2 0
0 1 0 0               0 2 0 0
0 0 0 0               0 0 0 0
```

BFS dry run from (1,1):

| Step | Dequeue | Match? | Paint | Enqueue neighbors |
|---|---|---|---|---|
| 1 | (1,1) | yes (1) | → 2 | (2,1), (0,1), (1,2), (1,0) |
| 2 | (2,1) | yes (1) | → 2 | (3,1)✗, (1,1)done, (2,2)✗, (2,0)✗ |
| 3 | (0,1) | no (0) | skip | — |
| 4 | (1,2) | yes (1) | → 2 | (2,2)✗, (0,2)✗, (1,3)✗, (1,1)done |
| 5 | (1,0) | no (0) | skip | — |

(✗ = doesn't match `oldColor=1`, skipped on dequeue)

**Result**: only the connected block of `1`s reachable from (1,1) gets repainted — the diagonal-adjacent or disconnected cells are untouched.

## Advantages

- Simple to implement on top of existing [[adsa-bfs]]/[[adsa-dfs]] machinery — just swap the "neighbor" function for grid-adjacency.
- No extra visited-set bookkeeping needed in the in-place repaint version — the grid mutation itself prevents reprocessing.
- Works identically for counting/marking (connected components) or repainting (visual fill) use cases.

## Disadvantages

- DFS (recursive) version risks stack overflow on large contiguous regions (e.g. filling a huge blank canvas) — BFS or an explicit stack avoids this.
- 4-directional vs 8-directional connectivity must be chosen carefully; using the wrong one leaks fill across diagonal gaps or fails to fill diagonally-touching regions, depending on intent.
- If `newColor == oldColor`, naive implementations infinite-loop — must guard this case explicitly (shown in pseudocode above).

## Example

In MS Paint's bucket tool, clicking inside a white shape with the fill set to red triggers flood fill from the clicked pixel: every contiguous white pixel reachable via the shape's interior turns red, while pixels outside the shape's boundary (a different color) are never touched.

## Use Cases

- Paint bucket / fill tool in image editors
- Minesweeper's "reveal connected empty cells" when clicking a zero-adjacent-mine cell
- Connected-component labeling in image processing (e.g. counting distinct blobs)
- "Number of islands" style problems (counting connected regions in a grid)
- Maze generation/solving variants where regions need to be identified or merged

#adsa
