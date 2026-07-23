# Koko Eating Bananas — LeetCode 875

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/koko-eating-bananas.html)**
> _(step-through binary search over EATING SPEED, not the array — live feasibility check, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search **on the Answer**  ·  **Tags:** Array, Binary Search, Greedy Feasibility Check

---

## Problem

Koko ke paas `piles` naam ka array hai — `piles[i]` = `i`-vi pile mein kitne bananas hain. Koh ko `h` ghanton mein **saari piles khatam** karni hain. Har ghante woh **ek pile** choose karti hai aur **`speed`** bananas khaati hai us pile se (agar pile mein `speed` se kam bananas hain, to woh pile *khatam* ho jaati hai aur baaki ghanta waste ho jaata hai — agli pile ke liye use nahi hota).

**Sabse chhoti (minimum) `speed`** dhoondo taaki Koko `h` ghanton mein saari piles khatam kar sake.

```
Input:  piles = [3, 6, 7, 11], h = 8
Output: 4

Input:  piles = [30, 11, 23, 4, 20], h = 5
Output: 30
```

---

## The story (yaad rakhne ke liye) 🧠

**Ruko — yeh iss folder ki pehli problem hai jahaan binary search array ke *indices* pe nahi chalti!**

Ab tak humne hamesha `lo`/`hi`/`mid` ko array ke **positions** pe move kiya — "is index ke left ya right." Yahaan **koi bhi sorted array hi nahi hai** jispe search karein! `piles` array sorted bhi nahi hai, aur humein usme kuch dhoondhna bhi nahi hai.

Iske bajaye, hum **"possible speeds" ki ek range** pe binary search karte hain — **`1` se `max(piles)` tak**. Yeh hai **"Binary Search on the Answer"** — ek bilkul alag lekin extremely powerful pattern:

> 🔑 **Array-index search vs Answer search:**
> - **Array-index search** (is folder ki problems 1-10): `lo`/`hi`/`mid` array ke **positions** hain. Hum poochte hain *"target yahan hai kya?"*
> - **Binary search on the answer**: `lo`/`hi`/`mid` khud **candidate answers** hain (yahaan: eating speeds). Hum poochte hain *"kya yeh speed KAAM KAREGI?"* — ek **feasibility check** function se.

Socho tum Koko ko coach kar rahe ho: *"1 banana/hour try karo... bahut slow, time khatam ho gaya. 1000/hour try karo... aasaani se ho gaya, par itni fast khane ki zaroorat nahi."* Dekha — **speed jitni badhaoge, time utna hi kam ya barabar lagega** (kabhi badhta nahi). Yeh hi **monotonic property** hai jo binary search ko valid banati hai — chahe koi array sorted na ho!

**Feasibility check (`canFinish`)** — kisi bhi candidate `speed` ke liye, total ghante calculate karo: `sum(ceil(pile / speed))` har pile ke liye. Agar total `<= h`, speed **feasible** hai — shaayad aur bhi **choti** speed try kar sakein. Agar `> h`, speed **feasible nahi** — hume **badi** speed chahiye.

---

## Approach 1 — Brute force 🟥

Speed `1` se shuru karo, badhate jao jab tak feasible na ho jaaye.

```java
class Solution {
    public int minEatingSpeed(int[] piles, int h) {
        int maxPile = 0;
        for (int p : piles) maxPile = Math.max(maxPile, p);

        for (int speed = 1; speed <= maxPile; speed++) {
            if (canFinish(piles, speed, h)) return speed;
        }
        return maxPile;   // unreachable given constraints (h >= piles.length)
    }

    private boolean canFinish(int[] piles, int speed, int h) {
        long hours = 0;
        for (int p : piles) hours += (p + speed - 1) / speed;   // ceil division
        return hours <= h;
    }
}
```

**Problem kya hai:** `O(maxPile × n)` — agar `maxPile = 10^9`, to yeh **bahut** slow hai. Speed range pe linear scan waste hai jab woh **monotonic** hai.

---

## Approach 2 — Optimal (Binary Search on the Answer) ✅

```java
class Solution {
    public int minEatingSpeed(int[] piles, int h) {
        int lo = 1, hi = 0;
        for (int p : piles) hi = Math.max(hi, p);   // hi = max possible useful speed

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;            // candidate speed

            if (canFinish(piles, mid, h)) {
                hi = mid;        // feasible! maybe an even smaller speed works
            } else {
                lo = mid + 1;    // too slow, need a bigger speed
            }
        }

        return lo;   // smallest feasible speed
    }

    private boolean canFinish(int[] piles, int speed, int h) {
        long hours = 0;
        for (int p : piles) hours += (p + speed - 1) / speed;   // ceil division
        return hours <= h;
    }
}
```

---

## Dry run — `piles = [3, 6, 7, 11]`, `h = 8`

| lo | hi | mid (candidate speed) | hours needed | Feasible (`<= 8`)? | Verdict |
|:---:|:---:|:---:|:---:|:---:|:---|
| 1 | 11 | 6 | `1+1+2+2=6` | ✅ Yes | try smaller → `hi = 6` |
| 1 | 6 | 3 | `1+2+3+4=10` | ❌ No | need bigger → `lo = 4` |
| 4 | 6 | 5 | `1+2+2+3=8` | ✅ Yes | try smaller → `hi = 5` |
| 4 | 5 | 4 | `1+2+2+3=8` | ✅ Yes | try smaller → `hi = 4` |
| 4 | 4 | — | — | — | `lo == hi`, **stop** |

**Result:** `4` ✅ (speed `4` → hours `= ceil(3/4)+ceil(6/4)+ceil(7/4)+ceil(11/4) = 1+2+2+3 = 8 <= 8`)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear speed scan | `O(maxPile × n)` | `O(1)` |
| Binary Search on Answer | `O(n log(maxPile))` | `O(1)` |

---

## Gotchas 🪤

- **`hi = max(piles)`, kabhi `+1` nahi** — speed `maxPile` guarantee karti hai har pile **1 ghante** mein khatam ho, isse zyada tez khane ki zaroorat kabhi nahi.
- **Ceil division sahi likho**: `(p + speed - 1) / speed` — Java integer division mein `Math.ceil((double)p/speed)` bhi chalta hai par double precision ke risk se bachne ke liye integer trick better hai.
- **`hours` ko `long` mein rakho** — bade `piles` (`10^9` tak) aur chhoti `speed` (`1`) ke saath `hours` `int` overflow kar sakta hai.
- **`hi = mid`, na ki `hi = mid - 1`** — `mid` khud feasible speed ho sakti hai, use range se poori tarah nikaalna galat hai (bilkul `upper-bound-ceiling` jaisa trick, par ab "answer space" pe).
- **Feasibility function hi asli kaam hai** — binary search ka shell to same rehta hai har "binary search on the answer" problem mein; jo badalta hai woh hai `canFinish`-jaisa check. Isko samajhna is pattern ki asli chaabi hai.
- **Monotonicity zaroori hai** — yeh technique tabhi kaam karti hai jab "speed badhne se feasibility kabhi worse nahi hoti" (yahaan: hamesha better ya same). Agar property monotonic na ho, binary search galat answer degi.
