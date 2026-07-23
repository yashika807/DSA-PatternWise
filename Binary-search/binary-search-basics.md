# Binary Search — LeetCode 704

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/binary-search-basics.html)**
> _(step-through `lo` / `mid` / `hi`, live eliminated-half readout, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** Binary Search  ·  **Tags:** Array, Binary Search

---

## Problem

Ek **sorted** (ascending) integer array `nums` (saare elements distinct) aur ek `target` diya hai. Agar `target` array mein hai to uska **index** return karo, warna `-1`.

```
Input:  nums = [-1, 0, 3, 5, 9, 12], target = 9
Output: 4

Input:  nums = [-1, 0, 3, 5, 9, 12], target = 2
Output: -1
```

Constraint: solution **`O(log n)`** mein chahiye — matlab linear scan allowed nahi, binary search hi karna hai.

---

## The story (yaad rakhne ke liye) 🧠

Socho ek **mota dictionary** (ya phone book) hai aur tumhe usme ek word dhoondhna hai. Tum shuru se ek-ek page palat ke nahi dhoondte — tum **beech mein** kholte ho, dekhte ho current page ka word tumhare target se pehle aata hai ya baad mein, aur **poori aadhi dictionary phek dete ho**. Fir bachi hui aadhi ke beech mein wahi karte ho. Har step pe search space **aadha** ho jaata hai — isi wajah se yeh `O(log n)` hai.

Teen characters yaad rakho:

| Role | Kaun | Kaam |
|---|---|---|
| **`lo`** | left boundary | "yahan se pehle kuch nahi bacha" |
| **`hi`** | right boundary | "yahan ke baad kuch nahi bacha" |
| **`mid`** | current guess | `lo` aur `hi` ke beech ka page jo hum abhi kholte hain |

> 🔑 Yeh poora **`Binary-search/` folder** isi ek idea pe khada hai: **"har step pe aadha search space eliminate karo."** Kabhi hum array ke *indices* pe search karte hain (jaise yeh problem), kabhi *possible answers* ki ek range pe (jaise Koko Bananas) — but core loop hamesha same rehta hai.

### Do canonical templates (dono yaad rakho, aage kaam aayenge)

**Template A — Exact match, closed interval `[lo, hi]`**
Jab tumhe **exact target chahiye** (ya `-1`), aur range `lo <= hi` tak valid hai:
```java
int lo = 0, hi = n - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] == target) return mid;
    else if (nums[mid] < target) lo = mid + 1;
    else hi = mid - 1;
}
return -1;
```

**Template B — Lower bound, half-open interval `[lo, hi)`**
Jab tumhe **"pehla index jahaan condition true ho"** chahiye (chahe target mile ya na mile) — is folder ke baaki problems (upper bound, first/last position, koko bananas...) **isi template pe** based hain:
```java
int lo = 0, hi = n; // hi EXCLUSIVE
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] < target) lo = mid + 1;
    else hi = mid;              // nums[mid] >= target, could be the answer
}
// lo == first index with nums[lo] >= target (or n if none)
```

Is problem mein hum **Template A** ko animate karenge (kyunki humein sirf exact index chahiye), lekin Template B ka structure yaad rakhna — aage bahut kaam aayega.

---

## Approach 1 — Brute force 🟥

Ek-ek element check karo left se right.

```java
class Solution {
    public int search(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == target) return i;
        }
        return -1;
    }
}
```

**Problem kya hai:** Array **sorted hone ka fayda hi nahi utha rahe** — `O(n)` time, jabki sorting se `O(log n)` mumkin hai. `n = 10⁴` tak toh chalega, par asli maksad miss ho gaya.

---

## Approach 2 — Optimal (Binary Search) ✅

Template A: `lo`/`hi` dono inclusive boundaries. Har step `mid` check karo — match mile toh return, warna galat aadhi range phek do.

```java
class Solution {
    public int search(int[] nums, int target) {
        int lo = 0, hi = nums.length - 1;   // closed interval [lo, hi]

        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;   // overflow-safe mid

            if (nums[mid] == target) {
                return mid;                  // found it
            } else if (nums[mid] < target) {
                lo = mid + 1;                // target right half mein hai
            } else {
                hi = mid - 1;                // target left half mein hai
            }
        }

        return -1;                           // lo > hi → space khatam, nahi mila
    }
}
```

---

## Dry run — `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`

| lo | hi | mid | nums[mid] | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 0 | 5 | 2 | 3 | `3 < 9` → too small → `lo = 3` |
| 3 | 5 | 4 | 9 | `9 == 9` → ✅ **found at index 4** |

**Result:** `4` ✅

Ek aur trace — `target = 2` (not present):

| lo | hi | mid | nums[mid] | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 0 | 5 | 2 | 3 | `3 > 2` → too big → `hi = 1` |
| 0 | 1 | 0 | -1 | `-1 < 2` → too small → `lo = 1` |
| 1 | 1 | 1 | 0 | `0 < 2` → too small → `lo = 2` |
| 2 | 1 | — | — | `lo > hi` → **stop, not found** |

**Result:** `-1` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Linear scan | `O(n)` | `O(1)` | sorted hone ka fayda nahi liya |
| Binary Search | `O(log n)` | `O(1)` | har step search space aadha |

---

## Gotchas 🪤

- **`mid = lo + (hi - lo) / 2`, na ki `(lo + hi) / 2`** — bade arrays mein `lo + hi` **overflow** kar sakta hai. Yeh habit shuru se hi daal lo, poore folder mein yehi likhoge.
- **`lo <= hi` (Template A) vs `lo < hi` (Template B)** — exact match dhoondhte ho toh `<=` (closed interval), "pehla valid index" dhoondhte ho toh `<` (half-open). Dono mix mat karo, infinite loop ya wrong answer aayega.
- **`hi = mid - 1` / `lo = mid + 1`, kabhi `hi = mid` ya `lo = mid` nahi (Template A mein)** — kyunki `mid` already check ho chuka hai aur match nahi tha, use range se poori tarah bahar karo, warna infinite loop.
- **Empty array** (`n = 0`) → `lo = 0, hi = -1` → loop chalta hi nahi, seedha `-1` return — apne aap handle ho jaata hai, extra `if` ki zaroorat nahi.
- **Array mein duplicates nahi** is problem mein — isliye "exact index" well-defined hai. Duplicates hote toh "first/last occurrence" jaisa alag sawaal ban jaata (dekho `first-and-last-position`).
