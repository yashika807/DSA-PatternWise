# Aggressive Cows — GfG / SPOJ (AGGRCOW)

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/aggressive-cows.html)**
> _(binary search on the MINIMUM DISTANCE, maximize variant — live stall placement, synced Java code)_

**Difficulty:** Hard  ·  **Pattern:** Binary Search **on the Answer** (Maximize)  ·  **Tags:** Array, Binary Search, Greedy Feasibility Check

---

## Problem

`stalls` array diya hai jisme `stalls[i]` = `i`-vi stall ki position hai. Humein `k` **aggressive cows** ko in stalls mein rakhna hai taaki **koi bhi do cows ke beech ki minimum distance** jitni ho sake utni **maximize** ho jaaye (cows itni aggressive hain ki paas rakhoge to ladengi).

**Maximum possible "minimum distance"** dhoondo.

```
Input:  stalls = [1, 2, 4, 8, 9], k = 3
Output: 3
(cows at 1, 4, 8 → min gap = min(3, 4) = 3, best possible)

Input:  stalls = [1, 2, 3, 4, 5], k = 2
Output: 4
(cows at 1, 5 → gap = 4)
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh bhi **koko-eating-bananas** wala hi pattern hai — **binary search on the answer** — bas is baar hum **maximize** kar rahe hain minimize ki jagah.

> 🔑 **Minimize vs Maximize on the answer:**
> - **Koko (minimize):** chhoti se chhoti feasible speed chahiye → feasible mile to **aur chhota** try karo (`hi = mid`).
> - **Aggressive Cows (maximize):** badi se badi feasible distance chahiye → feasible mile to **aur bada** try karo (`lo = mid + 1`), aur is `mid` ko as candidate answer **save** kar lo (kyunki loop khatam hone par `lo`/`hi` khud guaranteed feasible nahi hote — closed-interval style mein `ans` variable rakhna padta hai).

Socho tum in cows ke liye stalls assign kar rahe ho: *"kya distance `d` rakh ke saari `k` cows fit ho jaayengi?"* — yeh ek **greedy feasibility check** hai: stalls ko **sort** karo, **pehli cow pehli (leftmost) stall** pe rakho, phir jitni bhi agli stall mile jiski **last-placed-cow se distance `>= d`** ho, wahan agli cow rakh do. Agar iss tarah `>= k` cows fit ho jaayein, `d` **feasible** hai — **badi** distance try karo. Nahi to `d` **bahut zyada** hai — **chhoti** distance try karo.

Range: `lo = 1` (minimum possible gap) se `hi = max(stalls) - min(stalls)` (do extreme stalls ke beech ka poora fasla — sabse badi possible min-distance) tak.

---

## Approach 1 — Brute force 🟥

Distance `1` se shuru karo, jab tak feasible ho tab tak badhate jao; jaise hi infeasible ho jaaye, pichhla answer return karo.

```java
class Solution {
    public int aggressiveCows(int[] stalls, int k) {
        java.util.Arrays.sort(stalls);
        int maxGap = stalls[stalls.length - 1] - stalls[0];

        int ans = 1;
        for (int d = 1; d <= maxGap; d++) {
            if (canPlace(stalls, k, d)) ans = d;
            else break;
        }
        return ans;
    }

    private boolean canPlace(int[] stalls, int k, int d) {
        int count = 1, last = stalls[0];
        for (int i = 1; i < stalls.length; i++) {
            if (stalls[i] - last >= d) {
                count++;
                last = stalls[i];
            }
        }
        return count >= k;
    }
}
```

**Problem kya hai:** `O(maxGap × n)` — `maxGap` bahut bada ho sakta hai (positions `10^9` tak jaa sakti hain), to yeh linear scan bahut slow hai.

---

## Approach 2 — Optimal (Binary Search on the Answer, Maximize) ✅

```java
class Solution {
    public int aggressiveCows(int[] stalls, int k) {
        java.util.Arrays.sort(stalls);
        int n = stalls.length;
        int lo = 1, hi = stalls[n - 1] - stalls[0];
        int ans = 0;   // best feasible distance found so far

        while (lo <= hi) {                       // closed interval — note <=, not <
            int mid = lo + (hi - lo) / 2;         // candidate minimum distance

            if (canPlace(stalls, k, mid)) {
                ans = mid;      // feasible! remember it, try an even bigger distance
                lo = mid + 1;
            } else {
                hi = mid - 1;   // too far apart to fit k cows, shrink down
            }
        }

        return ans;
    }

    private boolean canPlace(int[] stalls, int k, int d) {
        int count = 1, last = stalls[0];   // first cow always at the first stall
        for (int i = 1; i < stalls.length; i++) {
            if (stalls[i] - last >= d) {
                count++;
                last = stalls[i];
            }
        }
        return count >= k;
    }
}
```

---

## Dry run — `stalls = [1, 2, 4, 8, 9]` (sorted), `k = 3`

| lo | hi | mid (candidate distance) | cows placed | Feasible (`>= 3`)? | Verdict |
|:---:|:---:|:---:|:---:|:---:|:---|
| 1 | 8 | 4 | stalls 1, 8 → 2 cows | ❌ No | too far → `hi = 3` |
| 1 | 3 | 2 | stalls 1, 4, 8 → 3 cows | ✅ Yes | `ans = 2`, try bigger → `lo = 3` |
| 3 | 3 | 3 | stalls 1, 4, 8 → 3 cows | ✅ Yes | `ans = 3`, try bigger → `lo = 4` |
| 4 | 3 | — | — | — | `lo > hi`, **stop** |

**Result:** `3` ✅ (cows at `1, 4, 8` → gaps `3, 4` → min gap `= 3`, best possible)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear distance scan | `O(maxGap × n)` | `O(1)` (excl. sort) |
| Binary Search on Answer | `O(n log(maxGap))` | `O(1)` (excl. sort) |

---

## Gotchas 🪤

- **Sort `stalls` first** — feasibility check greedy placement sirf sorted stalls pe hi kaam karta hai. Ye step bhoolna sabse common bug hai.
- **Closed interval `lo <= hi`, na ki `lo < hi`** — kyunki maximize karne mein `lo`/`hi` khud loop ke end pe guaranteed valid answer nahi hote; isliye ek alag `ans` variable mein best feasible value save karna padta hai. Yeh koko/first-and-last-position ke `lo < hi` pattern se **alag** hai — dhyaan rakho.
- **`hi = mid - 1` / `lo = mid + 1`** — dono jagah `mid` ko range se poora nikaal do, kyunki `mid` already test ho chuka hai aur `ans` mein record ho chuka hai (agar feasible tha).
- **`d = 0` invalid hai** — `lo` hamesha `1` se shuru karo, `0` se nahi (0 distance ka koi matlab nahi, aur `canPlace` degenerate case mein galat answer de sakta hai).
- **Greedy placement ka proof samajhna zaroori hai** — pehli cow hamesha **sabse pehli (leftmost) stall** pe rakhna optimal hai (isse aage sabse zyada room bachta hai). Yeh greedy choice hi feasibility check ko `O(n)` banata hai.
- **`k == 1` edge case** — sirf ek cow ho to koi bhi distance feasible hai; binary search khud-ba-khud `hi` (max possible gap) pe converge ho jaayega, extra special-casing ki zaroorat nahi.
