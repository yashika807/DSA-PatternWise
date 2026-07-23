# Upper Bound / Ceiling — The Building-Block Technique

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/upper-bound-ceiling.html)**
> _(step-through `lo` / `mid` / `hi` on the half-open template, live candidate-zone readout, synced Java code)_

**Difficulty:** Easy (concept)  ·  **Pattern:** Binary Search — Lower Bound Template  ·  **Tags:** Array, Binary Search, Building Block

---

## Problem

Ek **sorted** array `nums` aur ek `target` diya hai. **Sabse chhota index** `idx` dhoondo jahaan `nums[idx] >= target` ho — isko **ceiling** ya **upper bound** bolte hain. Agar koi bhi element `target` se bada-ya-barabar nahi hai, to `n` (array length) return karo — matlab "target insert karna ho to sabse end mein karo".

> ⚠️ **Naming ka confusion:** C++ STL mein isi function ko `lower_bound` kehte hain (kyunki yeh "lower boundary" hai un sab indices ki jinki value `>= target` hai). Common DSA baat-cheet mein isko **"ceiling"** ya **"upper bound"** bolte hain. Naam bhool jao, sirf definition yaad rakho: **"smallest index jahaan `nums[idx] >= target`."**

```
nums = [1, 3, 3, 5, 7, 9]

target = 4   →  idx = 3   (nums[3] = 5, sabse chhota >= 4)
target = 3   →  idx = 1   (exact match hai, par duplicates mein SABSE PEHLA)
target = 10  →  idx = 6   (koi element >= 10 nahi, insertion point = end)
```

Yeh koi single canonical LeetCode problem nahi hai — yeh ek **technique** hai jo aage `first-and-last-position` (LC 34) aur is folder ke kai problems ka foundation banti hai.

---

## The story (yaad rakhne ke liye) 🧠

Socho tumhare paas ek **railway timetable** hai — trains ka departure time sorted order mein likha hai. Tumhari train **chhoot gayi**. Ab tumhe station master se poochna hai: *"Meri arrival time ke **baad ya usi waqt** chalne waali sabse jaldi train kaun si hai?"*

Woh timetable ke **beech mein** ungli rakhta hai (`mid`). Agar us waqt ki train tumhare arrival se **pehle** hai, woh timetable ka **left half phek deta hai** (woh sab trains bhi chhoot chuki hain) aur right mein dhoondhta hai. Agar us waqt ki train tumhare arrival ke **baad-ya-barabar** hai, woh usse **candidate** rakhta hai — lekin fir bhi check karta hai ki **isse pehle koi aur bhi jaldi wali candidate train hai kya**, isliye right half nahi phekta, balki candidate ko record karke **left mein aur dhoondhta hai**.

Yehi wajah hai ki is template mein **`hi = mid`** hota hai, `hi = mid - 1` nahi — kyunki `mid` khud ek valid candidate ho sakta hai, use range se poori tarah nikaalna nahi hai.

| Role | Kaam |
|---|---|
| **`lo`** | "yahan tak ki sab trains chhoot chuki hain (too early)" |
| **`hi`** | "sabse achhi candidate train jo abhi tak mili — ya ek se aage kuch dekha hi nahi" |
| **`mid`** | current train jise check kar rahe hain |

Loop khatam hone pe `lo == hi` — yehi tumhara answer hai: **sabse jaldi train jo miss nahi hui**.

---

## Approach 1 — Brute force 🟥

Left se right scan karo, pehla element jo `>= target` ho use return karo.

```java
class Solution {
    public int lowerBound(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] >= target) return i;
        }
        return nums.length;   // koi bhi element target se bada nahi
    }
}
```

**Problem kya hai:** `O(n)` — sorted array ka fayda waste. Bade `n` pe slow.

---

## Approach 2 — Optimal (Binary Search, Template B) ✅

Half-open interval `[lo, hi)`. Jab `nums[mid] >= target`, wo **candidate** hai — usko range mein rakho (`hi = mid`) aur bayi taraf aur behtar (chhoti index) candidate dhoondo. Jab `nums[mid] < target`, wo kabhi answer nahi ban sakta — use poori tarah nikaal do (`lo = mid + 1`).

```java
class Solution {
    public int lowerBound(int[] nums, int target) {
        int lo = 0, hi = nums.length;      // hi EXCLUSIVE — answer range [0, n]

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;   // overflow-safe mid

            if (nums[mid] >= target) {
                hi = mid;                    // candidate! but maybe a smaller idx works too
            } else {
                lo = mid + 1;                // definitely too small, rule it out
            }
        }

        return lo;   // == hi here; n means "no ceiling exists"
    }
}
```

---

## Dry run — `nums = [1, 3, 3, 5, 7, 9]`, `target = 4`

| lo | hi | mid | nums[mid] | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 0 | 6 | 3 | 5 | `5 >= 4` → candidate! → `hi = 3` |
| 0 | 3 | 1 | 3 | `3 < 4` → rule out → `lo = 2` |
| 2 | 3 | 2 | 3 | `3 < 4` → rule out → `lo = 3` |
| 3 | 3 | — | — | `lo == hi`, **stop** |

**Result:** `idx = 3` → `nums[3] = 5` ✅ (sabse chhota element `>= 4`)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear scan | `O(n)` | `O(1)` |
| Binary Search (Template B) | `O(log n)` | `O(1)` |

---

## Gotchas 🪤

- **`hi = mid`, na ki `hi = mid - 1`** — sabse important trap. `mid` khud potential answer hai (`nums[mid] >= target`), use range se bahar mat nikaalo, sirf apne aap ko yaad dilao ki "aur behtar (chhota) candidate ho sakta hai left mein."
- **Loop condition `lo < hi`, na ki `lo <= hi`** — half-open interval hai; `lo == hi` pe range empty ho jaati hai, aur wahi answer hai.
- **`hi` ki initial value `nums.length`, na ki `nums.length - 1`** — kyunki "koi bhi ceiling nahi mila" ka valid answer `n` hai, aur `hi` exclusive hai to `n` ko range mein rakhna zaroori hai.
- **Return value `n` ho sakta hai** — caller code mein `nums[idx]` access karne se pehle **hamesha `idx < nums.length` check karo**, warna `ArrayIndexOutOfBounds`.
- **Duplicates ke case mein** yeh function **sabse pehla** occurrence deta hai jahaan `nums[idx] >= target` — yehi property `first-and-last-position` mein "first occurrence" dhoondhne ke liye seedha use hoti hai.
- **Strict "greater than" chahiye (upper bound, not ceiling)?** Bas `lowerBound(nums, target + 1)` call karo (integers ke liye) — yeh trick `first-and-last-position` mein "last occurrence" nikaalne ke liye use hoti hai.
