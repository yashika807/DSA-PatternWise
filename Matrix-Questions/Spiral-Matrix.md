# 54. Spiral Matrix

**Difficulty:** Medium  
**Pattern:** Matrix / Boundary traversal  
**Link:** https://leetcode.com/problems/spiral-matrix/

> 🔗 **[Open the interactive visualizer →]
>
> 
> 

---

## Problem

Given an `m x n` `matrix`, return **all elements in spiral order** (clockwise, starting top-left).

**Example 1**
```
Input:  [[1,2,3],[4,5,6],[7,8,9]]
Output: [1,2,3,6,9,8,7,4,5]
```

**Example 2**
```
Input:  [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
Output: [1,2,3,4,8,12,11,10,9,5,6,7]
```

**Constraints**
- `m == matrix.length`, `n == matrix[i].length`
- `1 <= m, n <= 10`
- `-100 <= matrix[i][j] <= 100`

---

## The idea

Hold four walls: `top`, `bottom`, `left`, `right`. Repeat four moves in order — walk the **top** row left→right, the **right** column top→bottom, the **bottom** row right→left, the **left** column bottom→top — and after finishing each edge, pull that wall one step inward. When the walls cross, you're done. The whole problem is really just *managing the boundaries carefully*.

Note there is no space-tier trade-off here like in the other matrix problems: the result list is the required output, so aside from it we already use `O(1)` extra space. The two versions below differ in **how they track "visited"**, not in Big-O.

---

## Approach A — visited grid · `O(m·n)` extra space

The most literal translation: walk in the four directions, turn clockwise when you hit an edge or an already-seen cell, and mark cells in a `boolean[][]` so you know where you've been. Easy to reason about, but it carries an extra grid.

```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        List<Integer> res = new ArrayList<>();
        boolean[][] seen = new boolean[m][n];

        int[] dr = {0, 1, 0, -1};   // right, down, left, up
        int[] dc = {1, 0, -1, 0};
        int r = 0, c = 0, dir = 0;

        for (int k = 0; k < m * n; k++) {
            res.add(matrix[r][c]);
            seen[r][c] = true;

            int nr = r + dr[dir], nc = c + dc[dir];
            // turn clockwise if next step is off-grid or already visited
            if (nr < 0 || nr >= m || nc < 0 || nc >= n || seen[nr][nc]) {
                dir = (dir + 1) % 4;
                nr = r + dr[dir];
                nc = c + dc[dir];
            }
            r = nr;
            c = nc;
        }
        return res;
    }
}
```

- **Time:** `O(m·n)`
- **Space:** `O(m·n)` for the `seen` grid (plus the output).

---

## Approach B — shrinking boundaries · `O(1)` extra space ✅

Drop the visited grid entirely. Track the four walls as integers and move them inward after each edge. The two `if` guards handle non-square matrices (a single leftover row or column) so you don't double-count.

```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> res = new ArrayList<>();
        int top = 0, bottom = matrix.length - 1;
        int left = 0, right = matrix[0].length - 1;

        while (top <= bottom && left <= right) {
            // → across the top row
            for (int c = left; c <= right; c++) res.add(matrix[top][c]);
            top++;

            // ↓ down the right column
            for (int r = top; r <= bottom; r++) res.add(matrix[r][right]);
            right--;

            // ← back across the bottom row
            if (top <= bottom)
                for (int c = right; c >= left; c--) res.add(matrix[bottom][c]);
            bottom--;

            // ↑ up the left column
            if (left <= right)
                for (int r = bottom; r >= top; r--) res.add(matrix[r][left]);
            left++;
        }
        return res;
    }
}
```

> ⚠️ The two `if` checks are the whole ballgame. Without `if (top <= bottom)` and `if (left <= right)`, a single remaining row or column gets traversed twice — the classic bug on inputs like `[[1,2,3]]` or `[[1],[2],[3]]`.

- **Time:** `O(m·n)`
- **Space:** `O(1)` extra (output aside).

---

## Complexity summary

| Approach | Time | Extra Space | Notes |
|---|---|---|---|
| Visited grid + turn | `O(m·n)` | `O(m·n)` | intuitive, direction-vector style |
| **Shrinking boundaries** | `O(m·n)` | `O(1)` | preferred — clean and minimal |

---

## What to actually say in an interview

1. Describe the four walls and the fixed move order: right, down, left, up.
2. Move each wall inward right after you finish its edge.
3. Flag the edge case out loud: after the first two edges, a lone middle row or column can be revisited, which is exactly what the two `if` guards prevent.
4. If asked for the direction-vector version, mention the `dr/dc` arrays and turning clockwise on a blocked or off-grid next step — same `O(m·n)` time, but `O(m·n)` space for the visited grid.
