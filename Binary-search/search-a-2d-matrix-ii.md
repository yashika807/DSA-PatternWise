# Search a 2D Matrix II — LeetCode 240

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/search-a-2d-matrix-ii.html)**
> _(staircase search from the top-right corner — live path trace, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Search Space Reduction (NOT classic binary search) · Staircase Walk  ·  **Tags:** Array, Binary Search, Matrix, Divide & Conquer

---

## Problem

`m x n` matrix diya hai jisme:
1. Har **row** left se right **ascending sorted** hai.
2. Har **column** top se bottom **ascending sorted** hai.

(Lekin `search-a-2d-matrix` jaisi guarantee **nahi** hai — yahaan row ka first element pichhli row ke last se bada hona zaroori **nahi** hai.)

`target` diya hai, batao ki woh matrix mein hai ya nahi.

```
Input:
matrix = [[1,4,7,11,15],
          [2,5,8,12,19],
          [3,6,9,16,22],
          [10,13,14,17,24],
          [18,21,23,26,30]]
target = 5
Output: true

target = 20
Output: false
```

---

## The story (yaad rakhne ke liye) 🧠

**⚠️ Ruko — is folder mein yeh first problem hai jo "pure" binary search NAHI hai!** Rows aur columns **independently** sorted hain, par cross-row guarantee (jaisi `search-a-2d-matrix` mein thi) yahaan **nahi** hai — isliye poori matrix ko ek flat sorted array ki tarah treat nahi kar sakte. Phir bhi yeh problem **isi folder mein rakhi jaati hai** kyunki iski spirit binary-search-jaisi hi hai: **har step pe search space ka ek bada hissa eliminate karna**.

Socho tum matrix ke **top-right corner** pe khade ho. Yeh corner special hai: uski **row mein sabse bada** element hai (row ka last), aur uske **column mein sabse chhota** element hai (column ka first). Ab compare karo:

- Agar `matrix[row][col] == target` → **mil gaya!**
- Agar `matrix[row][col] > target` → poori **current column** eliminate ho sakti hai (kyunki neeche ke sab elements is corner se bhi bade hain, target se dur) → **`col--`** (ek left jao).
- Agar `matrix[row][col] < target` → poori **current row** eliminate ho sakti hai (kyunki left ke sab elements is corner se bhi chhote hain) → **`row++`** (ek neeche jao).

> 🔑 **Har step ek poori row YA poora column eliminate karta hai** — isliye yeh **"binary search"** nahi, balki **"linear elimination"** hai — `O(m + n)`, na ki `O(log(m*n))`. Phir bhi extremely efficient hai kyunki har step guaranteed progress karta hai, aur DSA courses/interviews mein iss "binary search" family mein hi padhaya jaata hai kyunki philosophy same hai: **har comparison se search space chhota karo**.

**Bottom-left corner se bhi kaam chalta hai** (symmetric trick), par top-right sabse common convention hai.

---

## Approach 1 — Brute force 🟥

Poora matrix scan karo.

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

**Problem kya hai:** `O(m*n)` — dono sorted properties ka fayda bilkul nahi uthaya.

---

## Approach 2 — Optimal (Staircase Search from Top-Right) ✅

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length, n = matrix[0].length;
        int row = 0, col = n - 1;   // start at top-right corner

        while (row < m && col >= 0) {
            int val = matrix[row][col];

            if (val == target) {
                return true;
            } else if (val > target) {
                col--;   // eliminate this whole column
            } else {
                row++;   // eliminate this whole row
            }
        }

        return false;
    }
}
```

---

## Dry run — `matrix` (5×5 above), `target = 5`

| row | col | matrix[row][col] | Compare | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 0 | 4 | 15 | `15 > 5` | eliminate column 4 → `col = 3` |
| 0 | 3 | 11 | `11 > 5` | eliminate column 3 → `col = 2` |
| 0 | 2 | 7 | `7 > 5` | eliminate column 2 → `col = 1` |
| 0 | 1 | 4 | `4 < 5` | eliminate row 0 → `row = 1` |
| 1 | 1 | 5 | `5 == 5` | **found!** |

**Result:** `true` ✅ — 4 steps, eliminated 3 columns + 1 row before hitting the target.

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Full scan | `O(m*n)` | `O(1)` |
| Staircase Search | `O(m + n)` | `O(1)` |

---

## Gotchas 🪤

- **Yeh binary search NAHI hai — `O(m+n)`, na ki `O(log(m*n))`** — is folder mein isse isliye rakha gaya hai kyunki search-space-elimination philosophy same hai, par mechanism alag hai. Interview mein bol dena "yeh technically binary search ki family ka hai par pure binary search nahi" — sahi impression banega.
- **Start hamesha top-right (ya bottom-left) se karo, top-left ya bottom-right se NAHI** — top-left corner se dono directions (row++ aur col++) mein value badhti hai, isse koi elimination decide nahi ho sakta. Sirf **anti-diagonal corners** (top-right, bottom-left) hi kaam karte hain kyunki wahaan ek direction mein value badhti hai aur doosri mein ghatti hai.
- **`search-a-2d-matrix` (LC 74) se confuse mat karo** — us problem mein har row ka first element pichhli row ke last se bada hota hai (extra guarantee), isliye woh poora **flatten + pure binary search** ban jaata hai. Yahaan woh guarantee nahi hai, isliye staircase approach chahiye.
- **Loop condition `row < m && col >= 0`** — dono bounds check karo; agar sirf ek check kiya to array-out-of-bounds ho sakta hai.
- **`val > target` → `col--`, `val < target` → `row++`** — inhe ulta mat karo. Yaad rakhne ka tareeka: "bada mila to left jao (chhota dhoondo), chhota mila to neeche jao (bada dhoondo)."
