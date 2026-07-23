# Maximum Candies Allocated to K Children — LeetCode 2226

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/maximum-candies-allocated-to-k-children.html)**
> _(binary search on the MAX CANDIES PER CHILD, maximize variant — live pile-split readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search **on the Answer** (Maximize)  ·  **Tags:** Array, Binary Search, Greedy Feasibility Check

---

## Problem

`candies` array diya hai — `candies[i]` = `i`-vi pile mein kitni candies hain. Yeh candies `k` bacchon mein baatni hain, is rule ke saath: **har pile ko independently split** kar sakte ho (jaise `candies[i]` candies se `floor(candies[i] / x)` bacchon ko `x`-`x` candies mil sakti hain, bachi hui candies waste), aur **har bacche ko utni hi (same) candies** milni chahiye.

**Maximum possible `x`** (per-child candy count) dhoondo taaki `k` bacchon ko `x`-`x` candies mil jaayein.

```
Input:  candies = [5, 8, 6], k = 3
Output: 5
(3+1 splits: pile1→1 child(5), pile2→1 child(5, 3 wasted), pile3→1 child(5, 1 wasted) = 3 children @ 5 each)

Input:  candies = [2, 5], k = 11
Output: 0
(not enough candy to give even 1 each to 11 kids)
```

---

## The story (yaad rakhne ke liye) 🧠

Bilkul **aggressive-cows** jaisa hi shape: **binary search on the answer, maximize**. Bas is baar candidate answer hai **per-child candy count `x`**, aur range hai `1` se `max(candies)` tak.

**Feasibility check** (`canAllocate`): kisi bhi candidate `x` ke liye, har pile se `floor(candies[i] / x)` bacchon ko candies mil sakti hain — sab piles ka total nikaalo. Agar total `>= k`, `x` **feasible** hai — shaayad **aur bada** `x` bhi chal jaaye (badi candy per child, thoda risky). Agar `< k`, `x` **bahut zyada demanding** hai — **chhota** `x` try karo.

> 🔑 **Monotonic property:** jaise-jaise `x` badhta hai, har pile se milne waale bacchon ki sankhya (`floor(candies[i]/x)`) **kabhi nahi badhti** (sirf ghatti ya same rehti hai). Isliye total children count `x` ke against **monotonically non-increasing** hai — yeh property hi binary search ko valid banati hai.

**Edge case yaad rakho:** agar `x = 0` hi sirf feasible ho (matlab `k` itna bada hai ki koi bhi positive `x` kaam nahi karega), answer `0` hi hoga — binary search `lo = 1` se shuru karke kabhi feasible nahi paayega, `ans` apne default `0` pe hi reh jaayega.

---

## Approach 1 — Brute force 🟥

`x = max(candies)` se shuru karo, ghatate jao jab tak feasible na ho jaaye.

```java
class Solution {
    public int maximumCandies(int[] candies, int k) {
        int maxPile = 0;
        for (int c : candies) maxPile = Math.max(maxPile, c);

        for (int x = maxPile; x >= 1; x--) {
            if (canAllocate(candies, k, x)) return x;
        }
        return 0;
    }

    private boolean canAllocate(int[] candies, long k, int x) {
        long count = 0;
        for (int c : candies) count += c / x;
        return count >= k;
    }
}
```

**Problem kya hai:** `O(maxPile × n)` — `candies[i]` `10^7` tak ho sakta hai, to yeh linear scan bahut slow hai.

---

## Approach 2 — Optimal (Binary Search on the Answer, Maximize) ✅

```java
class Solution {
    public int maximumCandies(int[] candies, int k) {
        int lo = 1, hi = 0;
        for (int c : candies) hi = Math.max(hi, c);

        int ans = 0;   // 0 = fallback if even x=1 is infeasible

        while (lo <= hi) {                        // closed interval — note <=
            int mid = lo + (hi - lo) / 2;          // candidate candies-per-child

            if (canAllocate(candies, k, mid)) {
                ans = mid;       // feasible! remember, try a bigger x
                lo = mid + 1;
            } else {
                hi = mid - 1;    // too many kids demanded, shrink x
            }
        }

        return ans;
    }

    private boolean canAllocate(int[] candies, long k, int x) {
        long count = 0;
        for (int c : candies) count += c / x;   // floor division, integer x != 0
        return count >= k;
    }
}
```

---

## Dry run — `candies = [5, 8, 6]`, `k = 3`

| lo | hi | mid (candidate x) | children fed | Feasible (`>= 3`)? | Verdict |
|:---:|:---:|:---:|:---:|:---:|:---|
| 1 | 8 | 4 | `1+2+1=4` | ✅ Yes | `ans = 4`, try bigger → `lo = 5` |
| 5 | 8 | 6 | `0+1+1=2` | ❌ No | too demanding → `hi = 5` |
| 5 | 5 | 5 | `1+1+1=3` | ✅ Yes | `ans = 5`, try bigger → `lo = 6` |
| 6 | 5 | — | — | — | `lo > hi`, **stop** |

**Result:** `5` ✅ (each pile gives exactly 1 child 5 candies → 3 children total)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear scan on x | `O(maxPile × n)` | `O(1)` |
| Binary Search on Answer | `O(n log maxPile)` | `O(1)` |

---

## Gotchas 🪤

- **`x = 0` division by zero** — search range `lo` hamesha `1` se shuru karo, kabhi `0` se nahi, warna `candies[i] / x` mein crash ho jaayega.
- **`count` ko `long` mein rakho** — `candies[i]` bade ho sakte hain aur `x = 1` ho to `count` `10^9`-scale tak jaa sakta hai, `int` overflow ka risk hai `k` bhi bade constraints mein.
- **Closed interval `lo <= hi`, alag `ans` variable** — yeh maximize wala pattern hai (jaise `aggressive-cows`), `lo`/`hi` khud answer guarantee nahi karte, isliye `ans` mein best feasible value explicitly save karo.
- **Floor division, ceiling nahi** — yahan `candies[i] / x` (floor) sahi hai, kyunki bachi hui candies waste ho jaati hain, ek extra bacche ko nahi mil sakti. Koko-jaisi ceiling division se confuse mat karo.
- **`ans` ka default `0` hi correct fallback hai** — agar `k` bahut bada ho (jaise `[2,5], k=11`), koi bhi positive `x` feasible nahi milega, aur `ans = 0` hi sahi answer hai. Ismein extra special-casing ki zaroorat nahi.
- **Same "binary search on the answer, maximize" shell jaisa `aggressive-cows`** — sirf feasibility check badalta hai (`floor` sum vs `>= k` yahan, greedy placement wahan). Shell ek baar samajh lo, poore family ko crack kar loge.
