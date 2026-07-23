# Search a 2D Matrix — LeetCode 74

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/search-a-2d-matrix.html)**
> _(flatten trick — treat the whole matrix as one sorted array, live row/col mapping, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search — Flattened 2D Array  ·  **Tags:** Array, Binary Search, Matrix

---

## Problem

`m x n` matrix diya hai jisme yeh **special properties** hain:
1. Har row **left se right ascending sorted** hai.
2. Har row ka **pehla element**, pichhli row ke **aakhri element se bada** hai.

`target` diya hai, batao ki woh matrix mein hai ya nahi. `O(log(m*n))` mein solve karo.

```
Input:  matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
Output: true

Input:  matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 13
Output: false
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh do properties milke matrix ko ek **poori tarah sorted 1D array** bana deti hain — bas ek row ke baad doosri row **seedha continue** hoti hai, jaise koi kitaab ki lines wrap ho rahi hon.

Socho matrix ke saare elements ko row-by-row **ek line mein likh do**: `[1,3,5,7,10,11,16,20,23,30,34,60]` — yeh **poora sorted array** hai! Toh humein **naya algorithm nahi chahiye** — bas `binary-search-basics` wala **standard binary search** chalao, sirf ek **index-mapping trick** ke saath:

> 🔑 **Flatten trick:** matrix mein `n` columns hain. Flattened index `idx` ko row/col mein convert karo: **`row = idx / n`**, **`col = idx % n`**. `lo = 0`, `hi = m*n - 1` — bas!

Isse hume 2D matrix allocate/copy karne ki zaroorat nahi — hum **virtually** flatten kar rahe hain, on-the-fly index math se.

---

## Approach 1 — Brute force 🟥

Har row pe linear scan (ya har row pe binary search) karo.

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        for (int[] row : matrix) {
            for (int val : row) {
                if (val == target) return true;
            }
        }
        return false;
    }
}
```

**Problem kya hai:** `O(m*n)` — poora matrix scan ho raha hai, jabki dono sorted properties ka fayda uthaya ja sakta hai.

---

## Approach 2 — Optimal (Binary Search on flattened index) ✅

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length, n = matrix[0].length;
        int lo = 0, hi = m * n - 1;

        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;

            int row = mid / n;     // flattened index → row
            int col = mid % n;     // flattened index → column
            int val = matrix[row][col];

            if (val == target) {
                return true;
            } else if (val < target) {
                lo = mid + 1;       // target aage kahin hai
            } else {
                hi = mid - 1;       // target peeche kahin hai
            }
        }

        return false;
    }
}
```

---

## Dry run — `matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]]`, `target = 3` (`n = 4`)

| lo | hi | mid | row, col | matrix[row][col] | Verdict |
|:---:|:---:|:---:|:---:|:---:|:---|
| 0 | 11 | 5 | `1, 1` | 11 | `11 > 3` → `hi = 4` |
| 0 | 4 | 2 | `0, 2` | 5 | `5 > 3` → `hi = 1` |
| 0 | 1 | 0 | `0, 0` | 1 | `1 < 3` → `lo = 1` |
| 1 | 1 | 1 | `0, 1` | 3 | `3 == 3` → **found!** |

**Result:** `true` ✅

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Full scan | `O(m*n)` | `O(1)` |
| Binary Search (flattened) | `O(log(m*n))` | `O(1)` |

---

## Gotchas 🪤

- **`row = mid / n`, `col = mid % n`** — dono **integer division/modulo** hain, `n` = number of columns (row-length). Ulta karna (row ki jagah `m` use karna) common bug hai.
- **Yeh sirf tab kaam karta hai jab matrix "fully sorted" ho** — matlab har row ka first element, pichhli row ke last se bada ho. Agar sirf rows aur columns independently sorted hon (par cross-row guarantee na ho), yeh trick fail karega — dekho `search-a-2d-matrix-ii` ke liye alag approach.
- **`m*n` overflow** — bahut bade matrices ke liye (`m, n` dono `~10^3`), `m*n` `int` range mein fit ho jaata hai, par agar constraints aur bade hon to `long` use karo.
- **Empty matrix / empty rows** — `matrix.length == 0` ya `matrix[0].length == 0` check pehle karo, warna `hi = -1` ho jaayega aur loop bhi safely skip ho jaayega (`lo=0 > hi=-1`), par explicit guard rakhna safer hai.
- **Standard binary search hi hai** — is problem ka asli insight sirf yeh samajhna hai ki matrix ko **virtually flatten** kaise karna hai. Baaki poora `binary-search-basics.md` wala template hi hai.
