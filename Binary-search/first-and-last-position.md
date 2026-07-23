# Find First and Last Position of Element in Sorted Array — LeetCode 34

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/first-and-last-position.html)**
> _(step-through two ceiling searches back-to-back, live [first, last] readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search — Lower Bound Twice  ·  **Tags:** Array, Binary Search

---

## Problem

Sorted integer array `nums` aur ek `target` diya hai. `target` ki **first aur last occurrence ka index** dono return karo. Agar `target` array mein hai hi nahi, `[-1, -1]` return karo. Solution `O(log n)` mein chahiye.

```
Input:  nums = [5, 7, 7, 8, 8, 10], target = 8
Output: [3, 4]

Input:  nums = [5, 7, 7, 8, 8, 10], target = 6
Output: [-1, -1]
```

---

## The story (yaad rakhne ke liye) 🧠

Wahi railway timetable — par ab socho **multiple platforms** se **same departure time** pe trains chal sakti hain (duplicates!). Tumhe poochna hai: *"8:00 baje jitni bhi trains hain, unme se **sabse pehli platform** kaunsi hai aur **sabse aakhri platform** kaunsi hai?"*

Yeh dono sawaal `upper-bound-ceiling` waali **ceiling technique** se hi solve hote hain — bas do baar, thodi si trick ke saath:

1. **First occurrence** = ceiling of `target` khud → "sabse pehla index jahaan `nums[idx] >= target`." Agar wahaan value `target` nahi hai, matlab `target` array mein hai hi nahi.
2. **Last occurrence** = ceiling of `target + 1`, **minus 1** → "sabse pehla index jahaan `nums[idx] >= target+1`" ka matlab hai "target se strictly bada kuch shuru ho raha hai yahan se" — usse ek peeche hata do, wahi `target` ka **aakhri** occurrence hai.

> 🔑 **Building block reuse:** dono calls bilkul same `lowerBound()` function hain jo humne pichhli problem mein likha tha — sirf argument alag. Yeh pattern is poore folder mein baar-baar dikhega.

---

## Approach 1 — Brute force 🟥

Poore array ko scan karo, first aur last match track karo.

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int first = -1, last = -1;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == target) {
                if (first == -1) first = i;
                last = i;
            }
        }
        return new int[]{first, last};
    }
}
```

**Problem kya hai:** `O(n)` — agar sirf duplicates ki ginti karni ho (jaise saara array hi `target` ho), tab bhi poora scan karna padta hai. Sorted hone ka fayda nahi liya.

---

## Approach 2 — Optimal (Binary Search, twice) ✅

`lowerBound` ko helper bana lo, phir usse do baar call karo.

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int first = lowerBound(nums, target);

        // target khud array mein hai hi nahi
        if (first == nums.length || nums[first] != target) {
            return new int[]{-1, -1};
        }

        // "target+1 ka ceiling" - 1  ==  target ka last occurrence
        int last = lowerBound(nums, target + 1) - 1;

        return new int[]{first, last};
    }

    // same building block as upper-bound-ceiling.md
    private int lowerBound(int[] nums, int target) {
        int lo = 0, hi = nums.length;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] >= target) hi = mid;
            else lo = mid + 1;
        }
        return lo;
    }
}
```

---

## Dry run — `nums = [5, 7, 7, 8, 8, 10]`, `target = 8`

**Call 1 — `lowerBound(nums, 8)`** (find first occurrence)

| lo | hi | mid | nums[mid] | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 0 | 6 | 3 | 8 | `8 >= 8` → candidate → `hi = 3` |
| 0 | 3 | 1 | 7 | `7 < 8` → rule out → `lo = 2` |
| 2 | 3 | 2 | 7 | `7 < 8` → rule out → `lo = 3` |
| 3 | 3 | — | — | stop → **first = 3** (`nums[3] = 8` ✅ matches target) |

**Call 2 — `lowerBound(nums, 9)`** (find first index `>= target+1`)

| lo | hi | mid | nums[mid] | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 0 | 6 | 3 | 8 | `8 < 9` → rule out → `lo = 4` |
| 4 | 6 | 5 | 10 | `10 >= 9` → candidate → `hi = 5` |
| 4 | 5 | 4 | 8 | `8 < 9` → rule out → `lo = 5` |
| 5 | 5 | — | — | stop → ceiling(9) = 5 → **last = 5 - 1 = 4** |

**Result:** `[3, 4]` ✅ (`nums[3..4] = [8, 8]`)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear scan | `O(n)` | `O(1)` |
| Binary Search (twice) | `O(log n)` | `O(1)` |

---

## Gotchas 🪤

- **`target + 1` trick** — last occurrence dhoondhne ke liye seedha `target` pe binary search mat chalao (woh first occurrence hi degi). Ek pass aage `target+1` ki ceiling lo, phir `-1` karo.
- **Integer overflow on `target + 1`** — agar `target` already `Integer.MAX_VALUE` ho sakta hai (constraints check karo), to `long` use karo warna overflow ho jaayega. LeetCode 34 mein `target` bounded hota hai to usually safe hai, par production code mein hamesha dhyaan rakho.
- **"Not found" check pehli call ke turant baad** — `first == nums.length` (off the end) **ya** `nums[first] != target` (koi bada element mila par target nahi) — dono cases check karo, sirf ek nahi.
- **Dusri call `first` ke baad hi karo** — agar pehli call se `target` mila hi nahi, dusri call bekaar hai (aur `nums[first]` access karne se crash bhi ho sakta hai agar `first == n`).
- **Reuse the helper, don't rewrite it** — dono calls same `lowerBound` function use karte hain, sirf target alag. Copy-paste karke do alag binary searches likhna galtiyon ka source hai.
