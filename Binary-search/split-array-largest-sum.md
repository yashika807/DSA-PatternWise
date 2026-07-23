# Split Array Largest Sum — LeetCode 410

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/split-array-largest-sum.html)**
> _(binary search over MAX SUBARRAY SUM, minimize variant — live split readout, synced Java code)_

**Difficulty:** Hard  ·  **Pattern:** Binary Search **on the Answer**  ·  **Tags:** Array, Binary Search, Greedy Feasibility Check

---

## Problem

`nums` array diya hai (non-negative integers). Isse **`k` contiguous subarrays** mein split karo (order todna mana hai, jaise humesha). Split aisi chahiye ki **sabse badi subarray-sum jitni ho, woh minimum ho** (maximum load minimize karo).

```
Input:  nums = [7, 2, 5, 10, 8], k = 2
Output: 18
(split: [7,2,5]=14, [10,8]=18 → max = 18, best possible)

Input:  nums = [1, 2, 3, 4, 5], k = 2
Output: 9
```

---

## The story (yaad rakhne ke liye) 🧠

**Teesri baar wahi template** — `capacity-to-ship-packages` aur `book-allocation-problem` ke baad, yeh **third instance** hai us **exact same shape** ki: *"array ko k consecutive groups mein baato taaki max group-sum minimum ho."*

> 🔑 **Ek hi template, teen naam:**
> - **Ship capacity:** `weights → capacity` groups mein, `days` groups chahiye.
> - **Book allocation:** `pages → maxPagesPerStudent` groups mein, `m` groups (students) chahiye.
> - **Split array:** `nums → maxSubarraySum` groups mein, `k` groups (subarrays) chahiye.
>
> Teeno mein **`lo = max(element)`, `hi = sum(all)`**, aur feasibility check **greedy consecutive grouping** hai. Naming badalti hai, logic bilkul same rehta hai. Yeh pattern-recognition hi DSA mastery ki chaabi hai — ek baar shell crack kar liya, poori family free mein mil jaati hai.

Range: `lo = max(nums)` (kam se kam itni "capacity" chahiye ki sabse bada element apne aap mein fit ho) se `hi = sum(nums)` (ek hi subarray mein sab kuch) tak.

**Feasibility check** (`partsNeeded`): candidate `maxSum` ke liye, elements ko **greedily** groups mein bharo — jab agla element overflow kare to **naya group** shuru karo. Agar total groups `<= k`, `maxSum` **feasible** hai — shaayad **aur chhota** bhi chal jaaye. Agar `> k`, **bada** `maxSum` chahiye.

---

## Approach 1 — Brute force 🟥

`maxSum = max(nums)` se shuru karo, badhate jao jab tak feasible na ho jaaye.

```java
class Solution {
    public int splitArray(int[] nums, int k) {
        int maxN = 0, total = 0;
        for (int n : nums) { maxN = Math.max(maxN, n); total += n; }

        for (int cand = maxN; cand <= total; cand++) {
            if (partsNeeded(nums, cand) <= k) return cand;
        }
        return total;   // unreachable given valid input
    }

    private int partsNeeded(int[] nums, int maxSum) {
        int parts = 1, cur = 0;
        for (int n : nums) {
            if (cur + n > maxSum) { parts++; cur = n; }
            else cur += n;
        }
        return parts;
    }
}
```

**Problem kya hai:** `O((sum - maxN) × n)` — bade sums ke saath yeh bahut slow hai.

---

## Approach 2 — Optimal (Binary Search on the Answer) ✅

```java
class Solution {
    public int splitArray(int[] nums, int k) {
        int lo = 0, hi = 0;
        for (int n : nums) { lo = Math.max(lo, n); hi += n; }

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;         // candidate max subarray sum

            if (partsNeeded(nums, mid) <= k) {
                hi = mid;        // feasible! maybe an even smaller max works
            } else {
                lo = mid + 1;    // needs too many parts, allow a bigger sum
            }
        }

        return lo;   // smallest feasible max subarray sum
    }

    private int partsNeeded(int[] nums, int maxSum) {
        int parts = 1, cur = 0;
        for (int n : nums) {
            if (cur + n > maxSum) { parts++; cur = n; }   // overflow → new part
            else cur += n;
        }
        return parts;
    }
}
```

---

## Dry run — `nums = [7, 2, 5, 10, 8]`, `k = 2`

| lo | hi | mid (candidate max sum) | parts needed | Feasible (`<= 2`)? | Verdict |
|:---:|:---:|:---:|:---:|:---:|:---|
| 10 | 32 | 21 | `[7,2,5] [10,8]` → 2 | ✅ Yes | try smaller → `hi = 21` |
| 10 | 21 | 15 | `[7,2,5] [10] [8]` → 3 | ❌ No | need bigger → `lo = 16` |
| 16 | 21 | 18 | `[7,2,5] [10,8]` → 2 | ✅ Yes | try smaller → `hi = 18` |
| 16 | 18 | 17 | `[7,2,5] [10] [8]` → 3 | ❌ No | need bigger → `lo = 18` |
| 18 | 18 | — | — | — | `lo == hi`, **stop** |

**Result:** `18` ✅ (split `[7,2,5]=14`, `[10,8]=18` → max `= 18`)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear scan on max-sum | `O((sum-maxN) × n)` | `O(1)` |
| Binary Search on Answer | `O(n log(sum))` | `O(1)` |

---

## Gotchas 🪤

- **`lo = max(nums)`** — kisi bhi split mein sabse bada single element apni khud ki subarray mein reh sakta hai, isse chhoti `maxSum` kabhi feasible nahi hogi.
- **Greedy grouping order-preserving hai** — subarrays contiguous honi chahiye, isliye `partsNeeded` sequential scan hi karta hai.
- **`cur + n > maxSum` (strict `>`)** — agar `cur + n == maxSum`, wahi element **usi group** mein fit ho sakta hai.
- **`k` exactly ya `<= k` parts?** — humein `<= k` parts mein hona chahiye (agar kam parts mein bhi ho jaaye to fine hai, extra parts nahi banane), lekin practically `partsNeeded` monotonic decreasing hai `maxSum` ke against isliye jaise-jaise `maxSum` chhota hota hai parts sirf badhte hain — `k` se zyada kabhi nahi lagenge jab tak feasible ho.
- **Yeh, `capacity-to-ship-packages`, aur `book-allocation-problem` — teeno EK HI TEMPLATE hain** — is folder mein teen jagah dikh chuka hai. Naming: `weights/pages/nums`, `days/students/k`. Ek baar samajh lo, sab free mein.
- **`hi = mid`, na ki `hi = mid - 1`** — `mid` khud feasible max ho sakta hai, poori tarah range se nikaalna galat hai.
