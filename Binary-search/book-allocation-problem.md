# Book Allocation Problem — GfG

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/book-allocation-problem.html)**
> _(binary search over MAX PAGES PER STUDENT, minimize variant — live shelf-split readout, synced Java code)_

**Difficulty:** Hard  ·  **Pattern:** Binary Search **on the Answer**  ·  **Tags:** Array, Binary Search, Greedy Feasibility Check

---

## Problem

`pages` array diya hai — `pages[i]` = `i`-vi book mein kitne pages hain (books ek **fixed order** mein shelf pe rakhi hain). `m` students ko books baantni hain, is rule ke saath: har student ko **consecutive books** milti hain (order todna mana hai), har book **sirf ek** student ko milti hai, aur **har student ko kam se kam ek book** milni chahiye.

Allocation aisi chahiye ki **jo student sabse zyada pages padhta hai, uske pages minimum ho** (yaani maximum load minimize karo).

```
Input:  pages = [10, 20, 30, 40], m = 2
Output: 60
(student 1: books [10,20,30] = 60 pages, student 2: book [40] = 40 pages → max = 60)

Input:  pages = [12, 34, 67, 90], m = 2
Output: 113
```

---

## The story (yaad rakhne ke liye) 🧠

**Yeh `capacity-to-ship-packages` ka hi doosra naam hai** — bilkul same shape, bas "ship + days" ki jagah "student + max pages" hai.

> 🔑 **Structural mapping:** `weights → pages`, `capacity → maxPagesPerStudent`, `days → m (students)`. Feasibility check bhi identical hai — **greedily** consecutive items ko groups (days/students) mein bharo jab tak overflow na ho.

Range: `lo = max(pages)` (sabse mota book kam se kam itna to lagega hi ek student ko) se `hi = sum(pages)` (ek hi student sab kuch padh le) tak.

**Feasibility check** (`studentsNeeded`): candidate `maxPages` ke liye, books ko **greedily** students mein baato — jitni fit ho jaayen ek student ke paas utni do, jab agli book overflow kare to **naya student** shuru karo. Agar total students `<= m`, `maxPages` **feasible** hai — shaayad **aur chhota** bhi chal jaaye. Agar `> m`, **bada** `maxPages` chahiye.

**Special case:** agar `m > pages.length` (students books se zyada hain), allocation possible hi nahi — return `-1`.

---

## Approach 1 — Brute force 🟥

`maxPages = max(pages)` se shuru karo, badhate jao jab tak feasible na ho jaaye.

```java
class Solution {
    public int allocateBooks(int[] pages, int m) {
        int n = pages.length;
        if (m > n) return -1;

        int maxP = 0, total = 0;
        for (int p : pages) { maxP = Math.max(maxP, p); total += p; }

        for (int cand = maxP; cand <= total; cand++) {
            if (studentsNeeded(pages, cand) <= m) return cand;
        }
        return total;   // unreachable given valid input
    }

    private int studentsNeeded(int[] pages, int maxPages) {
        int students = 1, cur = 0;
        for (int p : pages) {
            if (cur + p > maxPages) { students++; cur = p; }
            else cur += p;
        }
        return students;
    }
}
```

**Problem kya hai:** `O((sum - maxP) × n)` — bade `pages` values ke saath yeh bahut slow ho jaata hai.

---

## Approach 2 — Optimal (Binary Search on the Answer) ✅

```java
class Solution {
    public int allocateBooks(int[] pages, int m) {
        int n = pages.length;
        if (m > n) return -1;   // more students than books — impossible

        int lo = 0, hi = 0;
        for (int p : pages) { lo = Math.max(lo, p); hi += p; }

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;         // candidate max pages per student

            if (studentsNeeded(pages, mid) <= m) {
                hi = mid;        // feasible! maybe an even smaller max works
            } else {
                lo = mid + 1;    // needs too many students, allow more pages
            }
        }

        return lo;   // smallest feasible max-pages-per-student
    }

    private int studentsNeeded(int[] pages, int maxPages) {
        int students = 1, cur = 0;
        for (int p : pages) {
            if (cur + p > maxPages) { students++; cur = p; }   // overflow → new student
            else cur += p;
        }
        return students;
    }
}
```

---

## Dry run — `pages = [10, 20, 30, 40]`, `m = 2`

| lo | hi | mid (candidate max) | students needed | Feasible (`<= 2`)? | Verdict |
|:---:|:---:|:---:|:---:|:---:|:---|
| 40 | 100 | 70 | `[10,20,30] [40]` → 2 | ✅ Yes | try smaller → `hi = 70` |
| 40 | 70 | 55 | `[10,20] [30] [40]` → 3 | ❌ No | need bigger → `lo = 56` |
| 56 | 70 | 63 | `[10,20,30] [40]` → 2 | ✅ Yes | try smaller → `hi = 63` |
| 56 | 63 | 59 | `[10,20] [30] [40]` → 3 | ❌ No | need bigger → `lo = 60` |
| 60 | 63 | 61 | `[10,20,30] [40]` → 2 | ✅ Yes | try smaller → `hi = 61` |
| 60 | 61 | 60 | `[10,20,30] [40]` → 2 | ✅ Yes | try smaller → `hi = 60` |
| 60 | 60 | — | — | — | `lo == hi`, **stop** |

**Result:** `60` ✅ (student 1 reads `10+20+30=60`, student 2 reads `40` → max `= 60`)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear scan on max-pages | `O((sum-maxP) × n)` | `O(1)` |
| Binary Search on Answer | `O(n log(sum))` | `O(1)` |

---

## Gotchas 🪤

- **`m > n` check pehle karo** — agar students books se zyada hain, har student ko ek book bhi nahi mil sakti; `-1` return karo, warna binary search galat/misleading answer de sakta hai.
- **`lo = max(pages)`** — kisi bhi ek student ko kam se kam sabse moti book jitna to padhna hi padega.
- **Greedy split order-preserving hai** — books ka order fixed hai (shelf order), isliye `studentsNeeded` sirf sequential scan karta hai.
- **`cur + p > maxPages` (strict `>`)** — agar `cur + p == maxPages`, wahi student usi book ko le sakta hai, naya student shuru karne ki zaroorat nahi.
- **Yeh aur `capacity-to-ship-packages` structurally identical hain** — same binary search shell, sirf naming alag (`weights→pages`, `days→students`). Ek baar samajh lo, dono free mein aa jaate hain.
- **`hi = mid`, na ki `hi = mid - 1`** — `mid` khud feasible max ho sakta hai, poori tarah range se nikaalna galat hai.
