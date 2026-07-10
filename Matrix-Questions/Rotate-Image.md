# 48. Rotate Image

**Difficulty:** Medium  
**Pattern:** Matrix / In-place transforms  
**Link:** https://leetcode.com/problems/rotate-image/

> 🔗 **[Open the interactive visualizer →]



---

## Problem

You are given an `n x n` 2D `matrix` representing an image. Rotate it by **90 degrees clockwise**, **in place** — do **not** allocate another 2D matrix.

**Example 1**
```
Input:  [[1,2,3],[4,5,6],[7,8,9]]
Output: [[7,4,1],[8,5,2],[9,6,3]]
```

**Example 2**
```
Input:  [[5,1,9,11],[2,4,8,10],[13,3,6,7],[15,14,12,16]]
Output: [[15,13,2,5],[14,3,4,1],[12,6,8,9],[16,7,10,11]]
```

**Constraints**
- `n == matrix.length == matrix[i].length`
- `1 <= n <= 20`
- `-1000 <= matrix[i][j] <= 100`

---

## The key mapping

For a clockwise 90° turn, the cell at `(i, j)` lands at `(j, n-1-i)`:

```
new[j][n-1-i] = old[i][j]
```

The top row becomes the right column, the left column becomes the top row, and so on. Everything below is just different ways to realize that mapping — the interesting jump is doing it **without** a second matrix.

---

## Brute Force — copy into a new matrix · `O(n²)` space

Directly apply the mapping into a fresh grid, then copy it back. Simple and obviously correct — but it **allocates another matrix**, which the problem explicitly forbids. Keep it only as a sanity check / intuition builder.

```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;
        int[][] res = new int[n][n];

        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                res[j][n - 1 - i] = matrix[i][j];

        // copy back into the original
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                matrix[i][j] = res[i][j];
    }
}
```

- **Time:** `O(n²)`
- **Space:** `O(n²)` — disqualified by the in-place rule.

---

## Optimal A — transpose, then reverse each row · `O(1)` space ✅

Two in-place flips compose into a rotation:

1. **Transpose** across the main diagonal → swap `m[i][j]` with `m[j][i]`. This turns rows into columns (a mirror over the `↘` diagonal).
2. **Reverse each row** → mirror left-to-right.

Transpose + horizontal flip = 90° clockwise. This is the version to reach for in an interview; it's short and hard to get wrong.

```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;

        // 1. transpose across the main diagonal
        for (int i = 0; i < n; i++)
            for (int j = i + 1; j < n; j++) {
                int t = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = t;
            }

        // 2. reverse each row
        for (int i = 0; i < n; i++)
            for (int l = 0, r = n - 1; l < r; l++, r--) {
                int t = matrix[i][l];
                matrix[i][l] = matrix[i][r];
                matrix[i][r] = t;
            }
    }
}
```

> ⚠️ **Only loop `j` from `i + 1`** in the transpose. If you start `j` at `0`, you swap every pair twice and end up back where you started.

- **Time:** `O(n²)`
- **Space:** `O(1)`

---

## Optimal B — four-way cyclic swap, layer by layer · `O(1)` space

Rotate the matrix as a set of concentric square rings. For each ring, move four elements at a time in one rotation cycle (top → right → bottom → left), so nothing needs a temp copy of the whole row. It's the "purest" single-pass rotation — one write per cell — but the index bookkeeping is easy to fumble, so most people prefer Optimal A.

```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;

        for (int layer = 0; layer < n / 2; layer++) {
            int first = layer, last = n - 1 - layer;

            for (int i = first; i < last; i++) {
                int offset = i - first;
                int top = matrix[first][i];                       // save top

                matrix[first][i]          = matrix[last - offset][first];   // left  -> top
                matrix[last - offset][first] = matrix[last][last - offset]; // bottom -> left
                matrix[last][last - offset]  = matrix[i][last];             // right  -> bottom
                matrix[i][last]           = top;                            // top   -> right
            }
        }
    }
}
```

- **Time:** `O(n²)`
- **Space:** `O(1)`

---

## Complexity summary

| Approach | Time | Extra Space | In-place? |
|---|---|---|---|
| Brute (new matrix) | `O(n²)` | `O(n²)` | ❌ violates the constraint |
| **Transpose + reverse** | `O(n²)` | `O(1)` | ✅ interview go-to |
| Four-way cyclic swap | `O(n²)` | `O(1)` | ✅ one write per cell |

---

## What to actually say in an interview

1. State the mapping: `(i, j) → (j, n-1-i)` for clockwise.
2. Mention the naive copy is `O(n²)` space and is ruled out by "in place".
3. Give **transpose + reverse rows** — explain *why* it works (mirror over diagonal, then mirror horizontally).
4. Call out the two classic bugs: transposing the full square (double swap) and, for counter-clockwise, that you'd **reverse columns** instead of rows.
