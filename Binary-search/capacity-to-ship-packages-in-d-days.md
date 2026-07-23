# Capacity To Ship Packages Within D Days — LeetCode 1011

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/capacity-to-ship-packages-in-d-days.html)**
> _(binary search over SHIP CAPACITY, minimize variant — live conveyor-belt readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search **on the Answer**  ·  **Tags:** Array, Binary Search, Greedy Feasibility Check

---

## Problem

`weights` array diya hai — `weights[i]` = `i`-vi package ka weight, jo **conveyor belt order** mein ship karni hain (order badal nahi sakte). Ek ship hai jiski **daily weight capacity** fixed hai — har din woh **utni hi packages** utha sakti hai jitni uski capacity allow kare (**consecutive** packages, order todna mana hai).

**Minimum ship capacity** dhoondo taaki saari packages `days` din ke andar ship ho jaayein.

```
Input:  weights = [1,2,3,4,5,6,7,8,9,10], days = 5
Output: 15

Input:  weights = [3,2,2,4,1,4], days = 3
Output: 6
```

---

## The story (yaad rakhne ke liye) 🧠

**Bilkul Koko-jaisa hi pattern** — bas "speed" ki jagah "ship capacity" hai. Yeh **binary search on the answer, minimize** hai.

Range: `lo = max(weights)` (kam se kam itni capacity chahiye taaki sabse bhaari package akela bhi fit ho jaaye) se `hi = sum(weights)` (poori shipment ek hi din mein) tak.

**Feasibility check** (`daysNeeded`): kisi candidate `capacity` ke liye, **greedily** packages ko days mein bharo — jitni fit ho jaayen ek din mein utni bharo, jab agli package overflow kare to **naya din** shuru karo. Agar total days `<= d`, capacity **feasible** hai — shaayad **aur chhoti** capacity bhi chal jaaye. Agar `> d`, capacity **kam** hai — **badi** capacity chahiye.

> 🔑 **Koko ka hi twin:** dono problems mein exact same shell hai — `lo`/`hi`/`mid` candidate answers hain, feasibility check monotonic hai (capacity badhne se days kabhi nahi badhte), aur `hi = mid` / `lo = mid + 1` pattern identical hai. Sirf feasibility function badalta hai: Koko mein per-pile ceiling-division sum, yahaan greedy day-splitting.

---

## Approach 1 — Brute force 🟥

Capacity `max(weights)` se shuru karo, badhate jao jab tak feasible na ho jaaye.

```java
class Solution {
    public int shipWithinDays(int[] weights, int days) {
        int maxW = 0, total = 0;
        for (int w : weights) { maxW = Math.max(maxW, w); total += w; }

        for (int cap = maxW; cap <= total; cap++) {
            if (daysNeeded(weights, cap) <= days) return cap;
        }
        return total;   // unreachable given valid input
    }

    private int daysNeeded(int[] weights, int cap) {
        int days = 1, cur = 0;
        for (int w : weights) {
            if (cur + w > cap) { days++; cur = w; }
            else cur += w;
        }
        return days;
    }
}
```

**Problem kya hai:** `O((sum - maxW) × n)` — `weights` bade ho sakte hain, to yeh linear scan bahut slow hai. Capacity range monotonic hai, linear scan waste hai.

---

## Approach 2 — Optimal (Binary Search on the Answer) ✅

```java
class Solution {
    public int shipWithinDays(int[] weights, int days) {
        int lo = 0, hi = 0;
        for (int w : weights) { lo = Math.max(lo, w); hi += w; }

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;         // candidate capacity

            if (daysNeeded(weights, mid) <= days) {
                hi = mid;        // feasible! maybe an even smaller capacity works
            } else {
                lo = mid + 1;    // not enough capacity, need bigger
            }
        }

        return lo;   // smallest feasible capacity
    }

    private int daysNeeded(int[] weights, int cap) {
        int days = 1, cur = 0;
        for (int w : weights) {
            if (cur + w > cap) { days++; cur = w; }   // overflow → new day
            else cur += w;
        }
        return days;
    }
}
```

---

## Dry run — `weights = [3, 2, 2, 4, 1, 4]`, `days = 3`

| lo | hi | mid (candidate capacity) | days needed | Feasible (`<= 3`)? | Verdict |
|:---:|:---:|:---:|:---:|:---:|:---|
| 4 | 16 | 10 | `[3,2,2] [4,1,4]` → 2 days | ✅ Yes | try smaller → `hi = 10` |
| 4 | 10 | 7 | `[3,2,2] [4,1] [4]` → 3 days | ✅ Yes | try smaller → `hi = 7` |
| 4 | 7 | 5 | `[3,2] [2,1] [4]... ` → 4 days | ❌ No | need bigger → `lo = 6` |
| 6 | 7 | 6 | `[3,2] [2,4] [1,4]` → 3 days | ✅ Yes | try smaller → `hi = 6` |
| 6 | 6 | — | — | — | `lo == hi`, **stop** |

**Result:** `6` ✅ (capacity 6 → days `[3,2]→5, [2,4]→6, [1,4]→5` = 3 days)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear capacity scan | `O((sum-maxW) × n)` | `O(1)` |
| Binary Search on Answer | `O(n log(sum))` | `O(1)` |

---

## Gotchas 🪤

- **`lo = max(weights)`, kabhi kam nahi** — capacity kam se kam itni honi chahiye ki sabse bhaari akeli package bhi ek din mein fit ho jaaye, warna woh package kabhi ship hi nahi hogi.
- **`hi = sum(weights)`** — poori shipment ek din mein bhi ho sakti hai agar capacity utni bhi ho.
- **Greedy day-split order-preserving hai** — packages ka order todna mana hai, isliye `daysNeeded` sequential scan hi karta hai, sorting nahi.
- **`cur + w > cap` (strict `>`), na ki `>=`** — agar `cur + w == cap`, wahi package **usi din** mein fit ho sakti hai, naya din shuru karne ki zaroorat nahi.
- **`hi = mid`, na ki `hi = mid - 1`** — `mid` khud feasible capacity ho sakti hai (bilkul Koko jaisa), poori tarah range se nikaalna galat hoga.
- **Same shell as Koko** — feasibility function hi problem-specific hai, baaki binary search structure identical. Ek baar Koko crack kar liya to yeh pattern turant pehchan jaoge.
