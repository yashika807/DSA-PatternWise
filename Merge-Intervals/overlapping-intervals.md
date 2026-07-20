# Check if Any Two Intervals Overlap — GFG

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Merge-Intervals/overlapping-intervals.html)**
> _(step-through sort + adjacent-pair scan, live true/false readout, synced Java code)_

**Difficulty:** Easy-Medium  ·  **Pattern:** Sort + Adjacent Scan  ·  **Tags:** Array, Sorting, Intervals

---

## Problem

Ek array `intervals` diya hai. Batao ki kya inme se **koi bhi do intervals overlap** karte hain — bas `true`/`false` chahiye (aur ho sake to konsi pair overlap karti hai).

```
Input:  intervals = [[1,3],[5,7],[2,6],[8,10]]
Output: true
Reason: [1,3] aur [2,6] overlap karte hain (2 < 3)
```

> ⚠️ **Convention:** Sirf **touch** karne wale intervals (jaise `[1,2]` aur `[2,3]`) overlap **nahi** maane jaate — overlap ke liye ek **common, non-zero-width region** chahiye.

---

## The story (yaad rakhne ke liye) 🧠

Socho tumhe ek **conference room** ki booking list check karni hai — bas yeh jaanna hai ki kya **koi double-booking** hui hai ya nahi (poori merge nahi karni, sirf haan/na chahiye).

Trick same hai jo Merge Intervals mein thi — **bookings ko start time se sort karo**. Lekin ab ek chhota sa **mathematical fact** kaam aata hai: agar sorted list mein **koi bhi do bookings** (chahe wo saath-saath ho ya door-door) overlap karti hain, toh unke **beech mein khadi HAR consecutive pair bhi** overlap karegi. Isliye humein **poore pairs check karne ki zaroorat nahi** — bas ek-ek karke **pados wali (adjacent) pairs** check karo. Agar kahin bhi ek adjacent pair overlap kare, jawaab `true` hai.

| Kyun kaam karta hai | Reason |
|---|---|
| Sorted list mein `start` values badhte order mein hain | Agar `intervals[q]` `intervals[p]` (p aage) se overlap karta hai (`start_q < end_p`), toh unke beech ka **immediate next** interval bhi zaroor `end_p` se pehle start hoga (kyunki starts sorted hain) |
| Isliye | Sirf `intervals[i].start < intervals[i-1].end` check karna kaafi hai — pura pairwise scan nahi chahiye |

---

## Approach 1 — Brute force 🟥

Har pair `(i, j)` check karo — dono ka overlap nikaalo aur dekho non-zero hai ya nahi.

```java
class Solution {
    public boolean isOverlap(int[][] intervals) {
        int n = intervals.length;

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                int start = Math.max(intervals[i][0], intervals[j][0]);
                int end   = Math.min(intervals[i][1], intervals[j][1]);
                if (start < end) {          // strict — touching counts as no overlap
                    return true;
                }
            }
        }
        return false;
    }
}
```

**Problem kya hai:** `O(n²)` — sabhi pairs check karna, jabki sorted order mein sirf adjacent pairs check karne se hi kaam ho jaata hai.

---

## Approach 2 — Sort + Adjacent Scan (Optimal) ✅

Sort karo start se, phir sirf **consecutive pairs** check karo.

```java
class Solution {
    public boolean isOverlap(int[][] intervals) {
        if (intervals.length <= 1) return false;

        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);   // sort by start

        for (int i = 1; i < intervals.length; i++) {
            if (intervals[i][0] < intervals[i - 1][1]) {  // strict < : touch != overlap
                return true;                               // found an overlapping pair
            }
        }
        return false;
    }
}
```

### Guardrail jo galti rokta hai

- **`<` hai, `<=` nahi** — `intervals[i][0] < intervals[i-1][1]`. Agar `==` ho (touch), overlap nahi maana jaata is problem mein.
- **Running-max ki zaroorat nahi** — Merge Intervals se alag, yahan `intervals[i-1][1]` (seedha pichla end) compare karna hi kaafi hai, current-open-block ka max maintain karne ki zaroorat nahi — kyunki sirf **existence** (true/false) chahiye, poora merge nahi.

---

## Dry run — `[[1,3],[5,7],[2,6],[8,10]]`

Sorted by start: **`[[1,3],[2,6],[5,7],[8,10]]`**

| i | intervals[i] | intervals[i-1] | Check | Verdict |
|:---:|:---:|:---:|:---|:---|
| 1 | `[2,6]` | `[1,3]` | `2 < 3` | ✅ overlap! → **return true** |

**Result:** `true` (pair `[1,3]` & `[2,6]`) ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (all pairs) | `O(n²)` | `O(1)` | poore pairs check karta hai |
| Sort + Adjacent Scan | `O(n log n)` | `O(1)` extra* | sort dominate karta hai, scan `O(n)` hai |

\* in-place sort maan kar; kuch languages mein sort ka apna `O(log n)` stack use ho sakta hai.

---

## Gotchas 🪤

- **Touching intervals overlap NAHI hain** — `[1,2]` aur `[2,3]` ka `2 < 2` false hai. Strict `<` use karo, warna galat `true` de doge.
- **Sirf adjacent pairs check karna sahi hai, lekin isiliye ki array sorted hai** — bina sort ke yeh trick kaam nahi karega, kyunki tab "adjacent in array" aur "adjacent in start-order" alag ho sakte hain.
- **Running max maintain mat karo yahan** — is problem mein sirf existence chahiye, isliye Merge Intervals jaisa `current.end = max(...)` zaroori nahi; seedha `intervals[i-1][1]` se compare karna mathematically sahi hai (proof: sorted starts ki wajah se, koi bhi overlap ho toh uska sabse pehla symptom hamesha kisi adjacent pair mein dikhega).
- **Empty ya single-interval array** — `intervals.length <= 1` pe seedha `false` return karo, loop chalane ki zaroorat nahi.
- **Unsorted input** — original array order pe directly adjacent check karna galat hoga; hamesha **pehle sort karo**.
- **Duplicate intervals** — same interval do baar diya ho (jaise `[2,4]` twice) toh woh khud overlap maane jaayenge (`2 < 4` true) — yeh sahi hai, do identical bookings bhi ek doosre se clash karti hain.
