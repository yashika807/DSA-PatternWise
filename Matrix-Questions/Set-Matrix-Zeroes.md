# 73. Set Matrix Zeroes

**Difficulty:** Medium  
**Pattern:** Matrix / In-place bookkeeping  
**Link:** https://leetcode.com/problems/set-matrix-zeroes/

> 🔗 **[Open the interactive visualizer →](./set-matrix-zeroes.html)** — step through the O(1) approach and watch row 0 / column 0 turn into a marker rail.

---

## Problem

Given an `m x n` integer matrix, if an element is `0`, set its **entire row and column** to `0`. You must do it **in place**.

**Example 1**
```
Input:  [[1,1,1],[1,0,1],[1,1,1]]
Output: [[1,0,1],[0,0,0],[1,0,1]]
```

**Example 2**
```
Input:  [[0,1,2,0],[3,4,5,2],[1,3,1,5]]
Output: [[0,0,0,0],[0,4,5,0],[0,3,1,0]]
```

**Constraints**
- `m == matrix.length`, `n == matrix[0].length`
- `1 <= m, n <= 200`
- `-2^31 <= matrix[i][j] <= 2^31 - 1`

---

## The trap

You **cannot** zero a row/column the moment you see a `0`. If you do, those fresh zeros get mistaken for original zeros and the clearing spreads across the whole matrix. So every approach is really the same idea: **first remember which rows and columns must die, then clear them in a second pass.** The three versions below only differ in *where* that memory lives — and shrinking the memory is the whole point of the follow-up.

---

## Brute Force — mark with a sentinel · `O(m·n)` extra passes

Walk the matrix. Wherever the value is `0`, mark the surrounding row and column cells with a placeholder (any value that can't appear as real data — here `Integer.MIN_VALUE`) *without* overwriting real zeros. Then convert every placeholder to `0`.

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        int R = matrix.length, C = matrix[0].length;
        int MARK = Integer.MIN_VALUE;

        for (int i = 0; i < R; i++) {
            for (int j = 0; j < C; j++) {
                if (matrix[i][j] == 0) {
                    // mark the row
                    for (int k = 0; k < C; k++)
                        if (matrix[i][k] != 0) matrix[i][k] = MARK;
                    // mark the column
                    for (int k = 0; k < R; k++)
                        if (matrix[k][j] != 0) matrix[k][j] = MARK;
                }
            }
        }

        for (int i = 0; i < R; i++)
            for (int j = 0; j < C; j++)
                if (matrix[i][j] == MARK) matrix[i][j] = 0;
    }
}
```

- **Time:** `O((m·n)·(m+n))` — every zero triggers a full row+column sweep.
- **Space:** `O(1)` extra, **but** it assumes a value never used by the data. That assumption breaks here because the constraints allow the full `int` range, so `Integer.MIN_VALUE` could be a legit entry. Good for building intuition; not interview-safe.

---

## Better — remember rows & columns separately · `O(m + n)` space

Keep two boolean arrays: one flags rows to clear, the other flags columns. Fill them in pass one, apply them in pass two. Clean, correct, and easy to explain.

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        int R = matrix.length, C = matrix[0].length;
        boolean[] rows = new boolean[R];
        boolean[] cols = new boolean[C];

        // pass 1: record which rows and columns contain a zero
        for (int i = 0; i < R; i++)
            for (int j = 0; j < C; j++)
                if (matrix[i][j] == 0) {
                    rows[i] = true;
                    cols[j] = true;
                }

        // pass 2: clear anything whose row or column was flagged
        for (int i = 0; i < R; i++)
            for (int j = 0; j < C; j++)
                if (rows[i] || cols[j])
                    matrix[i][j] = 0;
    }
}
```

- **Time:** `O(m·n)`
- **Space:** `O(m + n)`

---

## Optimal — rails as notepad · `O(1)` space ✅

The two boolean arrays above are just per-row and per-column flags. We already *have* a spare row and a spare column sitting in the matrix: **row 0 and column 0**. Reuse them as the flag storage.

One collision: `matrix[0][0]` would have to flag both row 0 and column 0 at once. Solve it by pulling row 0 and column 0 out into two separate `boolean` variables first, then using the rails only for rows `1..R-1` and columns `1..C-1`.

**Order matters** — apply the inner grid *before* wiping the rails, otherwise you erase the flags you still need.

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        int R = matrix.length, C = matrix[0].length;
        boolean firstRowZero = false, firstColZero = false;

        // 1. does row 0 / col 0 themselves contain a zero?
        for (int j = 0; j < C; j++)
            if (matrix[0][j] == 0) firstRowZero = true;
        for (int i = 0; i < R; i++)
            if (matrix[i][0] == 0) firstColZero = true;

        // 2. use rails (row 0 & col 0) to flag the inner grid
        for (int i = 1; i < R; i++)
            for (int j = 1; j < C; j++)
                if (matrix[i][j] == 0) {
                    matrix[i][0] = 0;
                    matrix[0][j] = 0;
                }

        // 3. clear inner cells whose rail is flagged
        for (int i = 1; i < R; i++)
            for (int j = 1; j < C; j++)
                if (matrix[i][0] == 0 || matrix[0][j] == 0)
                    matrix[i][j] = 0;

        // 4. finally, clear row 0 and col 0 if they were flagged
        if (firstRowZero)
            for (int j = 0; j < C; j++) matrix[0][j] = 0;
        if (firstColZero)
            for (int i = 0; i < R; i++) matrix[i][0] = 0;
    }
}
```

- **Time:** `O(m·n)`
- **Space:** `O(1)` — no auxiliary structures, just two booleans.

---

## Complexity summary

| Approach | Time | Extra Space | Interview-safe? |
|---|---|---|---|
| Brute (sentinel) | `O((m·n)(m+n))` | `O(1)`* | ❌ assumes an unused value |
| Better (two arrays) | `O(m·n)` | `O(m + n)` | ✅ |
| **Optimal (rails)** | `O(m·n)` | `O(1)` | ✅✅ this is the follow-up answer |

---

## What to actually say in an interview

1. State the trap: you must record before you clear.
2. Give the `O(m + n)` two-array version first — it proves you understand the structure.
3. Then compress the two arrays into row 0 and column 0, pulling out two booleans to break the `[0][0]` overlap.
4. Emphasize the ordering: **flag → apply inner grid → wipe rails last.**
