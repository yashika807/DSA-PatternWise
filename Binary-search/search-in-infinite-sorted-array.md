# Search in a Sorted Array of Unknown / Infinite Size — GFG / Classic

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/search-in-infinite-sorted-array.html)**
> _(step-through galloping bounds-finding, then a normal binary search inside them, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Exponential (Galloping) Search + Binary Search  ·  **Tags:** Array, Binary Search

**Problem link:** [GFG — Find position of an element in an infinite sorted array](https://www.geeksforgeeks.org/find-position-element-sorted-array-infinite-numbers/)

---

## Problem

Ek sorted array diya hai jiska **size tumhe pata nahi hai** (bahut bada, ya "infinite" treat karo). Tumhare paas sirf ek `get(index)` function hai jo `index` pe value deta hai — agar `index` array ke bahar hai to woh `Integer.MAX_VALUE` (ya koi sentinel) return karta hai. Ek `target` diya hai — uska index dhoondo, warna `-1`.

```
Input:  nums = [1,3,5,7,9,11,13,15,17,19,23,29,31,37,41,...] (size unknown), target = 23
Output: 10

Input:  same array, target = 4
Output: -1
```

---

## The story (yaad rakhne ke liye) 🧠

Socho ek **endless library** hai — kitabein alphabetically sorted hain, par tumhe pata hi nahi library **kahaan khatam hoti hai**. `binary-search-basics` ka trick seedha kaam nahi karega kyunki uske liye tumhe `hi` (end boundary) pehle se pata hona chahiye.

Solution **do phases** mein hota hai:

**Phase 1 — Galloping (bounds dhoondo):** Shelf pe `1, 2, 4, 8, 16, 32...` — **doubling jumps** lo. Har jump pe dekho kya wahaan ki kitab tumhare target se aage nikal gayi (ya shelf khatam ho gayi). Jaise hi aisa ho, tumhe pata chal gaya target **`lo` aur `hi` ke beech** kahin hai. Bas `O(log(target position))` jumps lagte hain — bahut fast.

**Phase 2 — Normal Binary Search:** Ab tumhare paas ek **normal, bounded** range `[lo, hi]` hai — bas Template A (`binary-search-basics` wala) laga do.

> 🔑 **Key insight:** Phase 1 sirf ek **"bounds finder"** hai — asli searching kaam Phase 2 karta hai, jo tumne pehle hi seekh liya hai. Yeh dikhata hai ki binary search **compose** hoti hai — chhoti techniques milke bada solution banati hain.

---

## Approach 1 — Brute force 🟥

`get(0), get(1), get(2)...` ek-ek karke check karo jab tak `target` mile ya value `target` se bada ho jaaye.

```java
class Solution {
    public int search(ArrayReader reader, int target) {
        int i = 0;
        while (reader.get(i) < target) i++;
        return reader.get(i) == target ? i : -1;
    }
}
```

**Problem kya hai:** `O(position of target)` — agar target 10 lakh index pe hai, to 10 lakh `get()` calls. Sorted structure ka fayda binary search jitna nahi utha rahe.

---

## Approach 2 — Optimal (Galloping + Binary Search) ✅

```java
interface ArrayReader {
    int get(int index);   // returns Integer.MAX_VALUE if index is out of bounds
}

class Solution {
    public int search(ArrayReader reader, int target) {
        if (reader.get(0) == target) return 0;

        // Phase 1: exponentially grow the window until it overshoots target
        int lo = 0, hi = 1;
        while (reader.get(hi) < target) {
            lo = hi;
            hi *= 2;
        }

        // Phase 2: normal binary search (Template A) inside [lo, hi]
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            int val = reader.get(mid);

            if (val == target) return mid;
            else if (val < target) lo = mid + 1;
            else hi = mid - 1;
        }

        return -1;
    }
}
```

---

## Dry run — `target = 23` on `[1,3,5,7,9,11,13,15,17,19,23,29,31,37,41]` (indices `0..14`)

**Phase 1 — galloping**

| lo | hi | get(hi) | Verdict |
|:---:|:---:|:---:|:---|
| 0 | 1 | 3 | `3 < 23` → `lo=1, hi=2` |
| 1 | 2 | 5 | `5 < 23` → `lo=2, hi=4` |
| 2 | 4 | 9 | `9 < 23` → `lo=4, hi=8` |
| 4 | 8 | 17 | `17 < 23` → `lo=8, hi=16` |
| 8 | 16 | ∞ (out of bounds) | `∞ ≥ 23` → **stop galloping**, window = `[8, 16]` |

**Phase 2 — binary search in `[8, 16]`**

| lo | hi | mid | get(mid) | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 8 | 16 | 12 | 31 | `31 > 23` → `hi = 11` |
| 8 | 11 | 9 | 19 | `19 < 23` → `lo = 10` |
| 10 | 11 | 10 | 23 | `23 == 23` → ✅ **found at index 10** |

**Result:** `10` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Linear `get()` calls | `O(p)` | `O(1)` | `p` = target's actual position |
| Galloping + Binary Search | `O(log p)` | `O(1)` | doubling reaches `p` in `~log₂p` steps, then `O(log p)` more for the search |

---

## Gotchas 🪤

- **`get(0)` alag se check karo** — galloping `hi = 1` se shuru hoti hai, agar target khud index `0` pe ho to galloping loop use kabhi test hi nahi karega jab tak hum shuru mein explicitly check na karein.
- **`hi *= 2`, na ki `hi += 1`** — yehi to poora point hai. Linear growth karoge to yeh brute force jaisa ban jaayega.
- **Overflow ka dhyaan** — `hi *= 2` bahut bada ho sakta hai agar target bahut door hai. Real systems mein `hi` ko `Integer.MAX_VALUE / 2` se cap karo taaki `hi *= 2` khud overflow na kare.
- **"Out of bounds" sentinel `>= target` maano, kabhi `== target` nahi** — sentinel (jaise `Integer.MAX_VALUE`) galti se real target ban sakta hai agar target khud `MAX_VALUE` ho — edge case interviews mein poochha jaata hai, discuss karna sahi rahega.
- **Phase 2 ka range `[lo, hi]` inclusive hai** — Phase 1 khatam hote hi seedha Template A (closed interval, `lo <= hi`) laga do, ek naya template mat banao.
- **Galloping khud `O(log p)` hai, `O(log n)` nahi** — `n` pata hi nahi hai! Isiliye complexity target ki *position* `p` pe depend karti hai, poore array size pe nahi.
