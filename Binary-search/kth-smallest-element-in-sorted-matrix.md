# Kth Smallest Element in a Sorted Matrix — LeetCode 378

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/kth-smallest-element-in-sorted-matrix.html)**
> _(binary search on VALUE range with a staircase counting helper — live count readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search **on the Answer** (Value Range + Counting)  ·  **Tags:** Array, Binary Search, Matrix, Heap

---

## Problem

`n x n` matrix diya hai jisme **har row aur har column ascending sorted** hai. Matrix ka **`k`-vaan sabse chhota element** dhoondo (sorted order mein, duplicates included).

```
Input:  matrix = [[1,5,9],[10,11,13],[12,13,15]], k = 8
Output: 13

Input:  matrix = [[-5]], k = 1
Output: -5
```

---

## The story (yaad rakhne ke liye) 🧠

**Ek naya twist:** ab tak humne "binary search on the answer" mein feasibility check dekha hai (Koko, ship-capacity, etc.). Yahaan hum **binary search on the VALUE range** karte hain — candidate answers **matrix ke values** hain, indices nahi.

Socho: agar humein pata chal jaaye ki *"matrix mein kitne elements `<= x` hain"*, to hum yeh count `k` se compare karke bata sakte hain ki actual `k`-vaan element `x` se chhota hai, bada hai, ya bilkul `x` hi hai — **bina poora matrix sort kiye!**

**Range:** `lo = matrix[0][0]` (sabse chhota element, top-left) se `hi = matrix[n-1][n-1]` (sabse bada, bottom-right) tak.

**Counting helper (`countLE`)** — kisi candidate value `x` ke liye, matrix mein `<= x` elements kitne hain yeh **staircase trick** se `O(n)` mein count karo (`search-a-2d-matrix-ii` jaisa hi corner-walk, bas **bottom-left** se shuru): agar current cell `<= x`, to **poora upar ka column-segment** (`row+1` elements) `<= x` hai (kyunki column top-se-bottom sorted hai) — count mein add karo, **right** jao. Agar `> x`, **up** jao (upar ki row try karo).

> 🔑 **Binary search condition:** `countLE(x) < k` ka matlab `x` bahut **chhota** hai (abhi tak `k` elements nahi mile) → `lo = mid + 1`. `countLE(x) >= k` ka matlab `x` **kaafi bada (ya theek)** hai → `hi = mid` (shaayad aur chhota bhi kaam kare). Yeh **lower-bound-style** search hai — hum smallest `x` dhoondh rahe hain jahaan `countLE(x) >= k`.

---

## Approach 1 — Brute force 🟥

Poora matrix ek array mein daal ke sort karo, `k`-vaan element utha lo.

```java
class Solution {
    public int kthSmallest(int[][] matrix, int k) {
        int n = matrix.length;
        int[] flat = new int[n * n];
        int idx = 0;
        for (int[] row : matrix) {
            for (int v : row) flat[idx++] = v;
        }
        java.util.Arrays.sort(flat);
        return flat[k - 1];
    }
}
```

**Problem kya hai:** `O(n^2 log(n^2))` — sorting ka poora overhead, jabki matrix already **partially sorted** hai (rows/columns dono).

---

## Approach 2 — Optimal (Binary Search on Value + Staircase Count) ✅

```java
class Solution {
    public int kthSmallest(int[][] matrix, int k) {
        int n = matrix.length;
        int lo = matrix[0][0], hi = matrix[n - 1][n - 1];

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;   // candidate value

            if (countLE(matrix, mid) < k) {
                lo = mid + 1;    // not enough elements <= mid, need bigger
            } else {
                hi = mid;        // enough (or more) — maybe smaller also works
            }
        }

        return lo;
    }

    // count elements <= x, walking from bottom-left corner — O(n)
    private int countLE(int[][] matrix, int x) {
        int n = matrix.length;
        int row = n - 1, col = 0, count = 0;

        while (row >= 0 && col < n) {
            if (matrix[row][col] <= x) {
                count += row + 1;   // whole column-segment above (incl. this) is <= x
                col++;               // move right, try a bigger column
            } else {
                row--;               // too big, move up
            }
        }
        return count;
    }
}
```

---

## Dry run — `matrix = [[1,5,9],[10,11,13],[12,13,15]]`, `k = 8`

| lo | hi | mid | countLE(mid) | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 1 | 15 | 8 | `2` | `2 < 8` → `lo = 9` |
| 9 | 15 | 12 | `6` | `6 < 8` → `lo = 13` |
| 13 | 15 | 14 | `8` | `8 >= 8` → `hi = 14` |
| 13 | 14 | 13 | `8` | `8 >= 8` → `hi = 13` |
| 13 | 13 | — | — | `lo == hi`, **stop** |

**Result:** `13` ✅ (elements `<= 13`: `1,5,9,10,11,12,13,13` = 8 of them, and `13` is the smallest value achieving that)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Flatten + sort | `O(n^2 log(n^2))` | `O(n^2)` |
| Binary Search on Value + Staircase Count | `O(n log(max-min))` | `O(1)` |

---

## Gotchas 🪤

- **`countLE` starts from bottom-left, na ki top-left** — yahi corner hai jahaan ek direction mein value badhti hai (right) aur doosri mein ghatti hai (up), jo staircase elimination ko valid banata hai (`search-a-2d-matrix-ii` jaisa hi trick, opposite corner).
- **`count += row + 1`, na ki `count++`** — jab `matrix[row][col] <= x`, to us column mein `row` se upar ke **saare** elements (`0..row`, total `row+1`) bhi `<= x` hain (column sorted hai), ek sath count kar lo — yehi is technique ko `O(n)` banata hai.
- **`hi = mid`, na ki `hi = mid - 1`** — hum lower-bound dhoondh rahe hain (smallest `x` jahaan `countLE(x) >= k`), `mid` khud answer ho sakta hai.
- **Answer hamesha matrix ka koi actual element hoga** — is binary search ka range matrix ki **values** pe hai, lekin final `lo` hamesha kisi real cell ki value hogi (proof: `countLE` sirf actual values pe "jump" karta hai jahaan count badhta hai).
- **Duplicates handle ho jaate hain automatically** — `countLE` duplicates ko bhi count karta hai, isliye `k`-vaan element sahi nikalta hai chahe repeats ho.
- **`kth-smallest-in-multiplication-table` ka twin hai** — dono ek hi technique use karte hain: binary search on value range + "count elements `<= x`" helper. Sirf counting helper ka implementation badalta hai (yahaan staircase, wahaan simple row-division).
