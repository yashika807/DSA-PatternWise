# Minimum Number of Days to Make m Bouquets — LeetCode 1482

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/minimum-days-to-make-m-bouquets.html)**
> _(step-through binary search over the CALENDAR DAY, live garden/feasibility readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search **on the Answer**  ·  **Tags:** Array, Binary Search, Greedy Feasibility Check

---

## Problem

`bloomDay[i]` = wo din jab `i`-vi flower **khilegi**. Tumhe `m` bouquets banane hain, har bouquet mein **exactly `k` ADJACENT (consecutive index) khile hue flowers** chahiye. Wo **minimum day** dhoondo jispe `m` bouquets ban sakein. Agar kabhi mumkin hi na ho, `-1`.

```
Input:  bloomDay = [1, 10, 3, 10, 2], m = 3, k = 1
Output: 3

Input:  bloomDay = [1, 10, 3, 10, 2], m = 3, k = 2
Output: -1   (m*k = 6 > 5 flowers total — kabhi mumkin nahi)
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh bhi `koko-eating-bananas` jaisi hi **"Binary Search on the Answer"** family ki problem hai — **phir se array ke index pe search nahi kar rahe.**

Socho ek **baagbaan (gardener)** ho. Har flower ka apna **khilne ka din** fix hai (`bloomDay[i]`) — yeh koi sorted list nahi hai, bas ek array hai. Tumhe poochha jaa raha hai: *"Sabse jaldi kaunsa din aayega jab main bagiche mein ghoom ke `m` bouquets bana paaunga, har bouquet ke liye `k` **paas-paas** khili flowers chunte hue?"*

> 🔑 **Answer-space:** `lo` = `min(bloomDay)` (sabse jaldi kisi flower ka khilna), `hi` = `max(bloomDay)` (sabse aakhri flower ka khilna — is din tak **sab** khil chuki hongi). Beech ke **har din** ek candidate answer hai.

Har candidate `day` (`mid`) ke liye ek **feasibility check** karo — *"Is din tak, kya `m` bouquets ban sakti hain?"*

- Array ko left se right scan karo, ek **running counter** rakho jitni **consecutive** flowers `bloomDay[i] <= day` hain.
- Jab counter `k` tak pahunche, ek bouquet ban gaya — counter reset karo, bouquet-count badhao.
- Jaise hi koi flower `bloomDay[i] > day` mile (abhi khili nahi), **consecutive run tut jaata hai** — counter reset (0) karo.

Agar total bouquets `>= m`, yeh `day` **feasible** hai — shaayad **pehle** ka din bhi kaam kar jaaye, `hi = mid`. Warna abhi **aur intezaar** karna padega, `lo = mid + 1`.

> ⚠️ **"Adjacent" ka matlab important hai** — array mein **consecutive indices**, calendar mein consecutive din nahi. Isiliye ek single `bloomDay[i] > day` poore running streak ko todta hai.

---

## Approach 1 — Brute force 🟥

Day `min(bloomDay)` se `max(bloomDay)` tak try karo.

```java
class Solution {
    public int minDays(int[] bloomDay, int m, int k) {
        if ((long) m * k > bloomDay.length) return -1;

        int lo = Integer.MAX_VALUE, hi = Integer.MIN_VALUE;
        for (int b : bloomDay) { lo = Math.min(lo, b); hi = Math.max(hi, b); }

        for (int day = lo; day <= hi; day++) {
            if (canMake(bloomDay, m, k, day)) return day;
        }
        return -1;   // unreachable given the feasibility pre-check above
    }

    private boolean canMake(int[] bloomDay, int m, int k, int day) {
        int bouquets = 0, streak = 0;
        for (int b : bloomDay) {
            if (b <= day) {
                streak++;
                if (streak == k) { bouquets++; streak = 0; }
            } else {
                streak = 0;
            }
        }
        return bouquets >= m;
    }
}
```

**Problem kya hai:** `O((maxDay - minDay) × n)` — bloom days bade ho sakte hain (`10^9` tak), toh yeh **bahut** slow.

---

## Approach 2 — Optimal (Binary Search on the Answer) ✅

```java
class Solution {
    public int minDays(int[] bloomDay, int m, int k) {
        // pehle hi check karo ki possible hai ya nahi
        if ((long) m * k > bloomDay.length) return -1;

        int lo = Integer.MAX_VALUE, hi = Integer.MIN_VALUE;
        for (int b : bloomDay) { lo = Math.min(lo, b); hi = Math.max(hi, b); }

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;   // candidate day

            if (canMake(bloomDay, m, k, mid)) {
                hi = mid;        // feasible — maybe an earlier day works too
            } else {
                lo = mid + 1;    // not enough bloomed yet — wait longer
            }
        }

        return lo;   // earliest feasible day
    }

    private boolean canMake(int[] bloomDay, int m, int k, int day) {
        int bouquets = 0, streak = 0;
        for (int b : bloomDay) {
            if (b <= day) {
                streak++;
                if (streak == k) { bouquets++; streak = 0; }
            } else {
                streak = 0;      // consecutive run broken
            }
        }
        return bouquets >= m;
    }
}
```

---

## Dry run — `bloomDay = [1, 10, 3, 10, 2]`, `m = 3`, `k = 1`

`m*k = 3 <= 5` → possible. `lo = 1, hi = 10`.

| lo | hi | mid (candidate day) | streak-scan bouquets | Feasible (`>= 3`)? | Verdict |
|:---:|:---:|:---:|:---:|:---:|:---|
| 1 | 10 | 5 | `1,✗,3,✗,2` khile → bouquets (k=1, har khila flower apna bouquet) = 3 | ✅ Yes | try earlier → `hi = 5` |
| 1 | 5 | 3 | khile: `1,3,2` → 3 bouquets | ✅ Yes | try earlier → `hi = 3` |
| 1 | 3 | 2 | khile: `1,2` → 2 bouquets | ❌ No | wait more → `lo = 3` |
| 3 | 3 | — | — | — | `lo == hi`, **stop** |

**Result:** `3` ✅

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear day scan | `O((maxDay − minDay) × n)` | `O(1)` |
| Binary Search on Answer | `O(n log(maxDay − minDay))` | `O(1)` |

---

## Gotchas 🪤

- **`m * k > n` check sabse pehle karo** — agar total flowers hi kaafi nahi hain, koi bhi day kabhi feasible nahi hoga. Yeh check na karo to loop chal to jaayega par galat/infinite-jaisa waqt lagega — hamesha `-1` shortcut pehle laga do. `long` use karo overflow se bachne ke liye (`m`, `k` dono bade ho sakte hain).
- **"Adjacent" = consecutive ARRAY indices**, calendar days nahi — `streak` ko reset karna hai jaise hi koi flower abhi tak nahi khili, chahe agla wala already khila ho.
- **`streak == k` ke baad turant reset karo** (`streak = 0`) — agar reset na karo to overlapping bouquets galat tarike se count ho jaayenge.
- **`hi = mid`, na ki `hi = mid - 1`** — `mid` khud feasible day ho sakta hai, wahi `koko-eating-bananas` jaisa hi trick.
- **`lo`/`hi` ki initial values `min`/`max` of `bloomDay`, `0`/`Integer.MAX_VALUE` nahi** — kisi bhi din se pehle koi feasible answer ho hi nahi sakta, aur `max` ke baad check karne ki zaroorat nahi (sab already khil chuki hongi).
- **Yeh bhi ek "Binary Search on the Answer" hai** — `koko-eating-bananas` jaisa hi shell (`lo`/`hi`/`mid` = candidate answers, feasibility check = asli logic). Is pattern ko pehchanna seekh lo: jab bhi "minimum X taaki Y possible ho" jaisa sawaal dikhe, aur "X badhne se Y kabhi impossible se possible ki taraf hi jaaye" (monotonic), socho — binary search on the answer!
