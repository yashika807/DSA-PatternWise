# Kth Smallest Number in Multiplication Table — LeetCode 668

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/kth-smallest-in-multiplication-table.html)**
> _(binary search on VALUE range with a per-row counting formula — live count readout, synced Java code)_

**Difficulty:** Hard  ·  **Pattern:** Binary Search **on the Answer** (Value Range + Counting)  ·  **Tags:** Binary Search, Matrix, Math

---

## Problem

`m x n` multiplication table diya hai (`table[i][j] = i * j`, `1`-indexed, `1 <= i <= m`, `1 <= j <= n`) — **implicitly**, poori table banani nahi hai (bahut badi ho sakti hai). Iska **`k`-vaan sabse chhota element** dhoondo.

```
Input:  m = 3, n = 3, k = 5
Output: 3
(table = [[1,2,3],[2,4,6],[3,6,9]], sorted: 1,2,2,3,3,4,6,6,9 → 5th = 3)

Input:  m = 2, n = 3, k = 6
Output: 6
```

---

## The story (yaad rakhne ke liye) 🧠

**Bilkul `kth-smallest-element-in-sorted-matrix` ka hi twin** — same **binary search on value range + counting** technique, bas counting helper ka implementation badal jaata hai kyunki yahaan matrix **explicit nahi hai**, sirf ek **formula** (`i * j`) se implicitly define hoti hai.

**Range:** `lo = 1` (sabse chhota possible value) se `hi = m * n` (sabse bada, `table[m][n]`) tak.

**Counting helper** (`countLE`) — kisi candidate value `x` ke liye, poori table scan kiye bina yeh count karo ki `<= x` kitne elements hain: **har row `i`** (`1` se `m` tak) mein, elements hain `i*1, i*2, ..., i*n` — yeh **arithmetic progression** hai! Row `i` mein `<= x` elements ki ginti seedhi formula se milti hai: **`min(n, floor(x / i))`** (kitne `j` values aisi hain ki `i*j <= x`, capped at `n`). Sab rows ka sum le lo — `O(m)` mein poora count mil jaata hai, **row scan bhi nahi karna padta!**

> 🔑 **Yeh `kth-smallest-in-sorted-matrix` se aur bhi tez hai** — wahaan counting `O(n)` staircase thi (explicit matrix), yahaan counting `O(m)` **pure math** hai (implicit "matrix"). Dono ka binary-search shell **bilkul same** hai — sirf `countLE` ka andar ka kaam badalta hai.

---

## Approach 1 — Brute force 🟥

Poori table banao, sort karo, `k`-vaan utha lo.

```java
class Solution {
    public int findKthNumber(int m, int n, int k) {
        int[] flat = new int[m * n];
        int idx = 0;
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                flat[idx++] = i * j;
            }
        }
        java.util.Arrays.sort(flat);
        return flat[k - 1];
    }
}
```

**Problem kya hai:** `O(m*n log(m*n))` time **aur** `O(m*n)` space — `m, n` `~10^5` tak ho sakte hain, to poori table banana hi impossible ho jaata hai (memory explode).

---

## Approach 2 — Optimal (Binary Search on Value + Row-Formula Count) ✅

```java
class Solution {
    public int findKthNumber(int m, int n, int k) {
        int lo = 1, hi = m * n;

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;   // candidate value

            if (countLE(m, n, mid) < k) {
                lo = mid + 1;    // not enough — go bigger
            } else {
                hi = mid;         // enough — maybe smaller also works
            }
        }

        return lo;
    }

    // count elements <= x in the implicit m x n table, O(m)
    private int countLE(int m, int n, int x) {
        int count = 0;
        for (int i = 1; i <= m; i++) {
            count += Math.min(n, x / i);   // row i has i, 2i, 3i, ... — count how many <= x
        }
        return count;
    }
}
```

---

## Dry run — `m = 3, n = 3, k = 5`

| lo | hi | mid | countLE(mid) | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 1 | 9 | 5 | `3+2+1=6` | `6 >= 5` → `hi = 5` |
| 1 | 5 | 3 | `3+1+1=5` | `5 >= 5` → `hi = 3` |
| 1 | 3 | 2 | `2+1+0=3` | `3 < 5` → `lo = 3` |
| 3 | 3 | — | — | `lo == hi`, **stop** |

**Result:** `3` ✅ (sorted table: `1,2,2,3,3,4,6,6,9` — 5th element is `3`)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Build + sort full table | `O(m·n log(m·n))` | `O(m·n)` |
| Binary Search on Value + Row Count | `O(m log(m·n))` | `O(1)` |

---

## Gotchas 🪤

- **`min(n, x/i)`, na ki sirf `x/i`** — row `i` mein sirf `n` columns hain, agar `x/i > n` ho to bhi row mein max `n` elements hi hain, cap lagana zaroori hai warna overcounting hogi.
- **Integer division `x / i`** — yeh **floor** division deta hai, jo exactly wahi hai jo chahiye: "kitne `j` (`1..n`) aise hain ki `i*j <= x`" = `floor(x/i)`, capped at `n`.
- **`countLE` ka `O(m)` hona hi is problem ka dil hai** — `m` iterate karo, `n` nahi (agar `n < m` ho to `min(m,n)` pe iterate karo for efficiency, though not required for correctness).
- **`hi = mid`, na ki `hi = mid - 1`** — lower-bound-style search hai (jaise `kth-smallest-in-sorted-matrix`), `mid` khud answer ho sakta hai.
- **`kth-smallest-in-sorted-matrix` ka parallel yaad rakho** — same shell, different counting internals: explicit matrix → staircase count (`O(n)`); implicit multiplication table → row-formula count (`O(m)`). Ek baar samajh lo, dono free.
- **`m*n` int overflow ka dhyaan rakho** — bade `m, n` (`~10^5` each) ke saath `m*n` `int` range (`~2×10^9`) ke paas ja sakta hai; agar constraints aur bade hon to `long` use karo `hi` ke liye.
