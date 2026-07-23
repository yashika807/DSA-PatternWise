# Find Minimum in Rotated Sorted Array — LeetCode 153

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/find-minimum-in-rotated-sorted-array.html)**
> _(step-through the "which half is sorted" check, live seam/minimum readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search on a Rotated Array  ·  **Tags:** Array, Binary Search

---

## Problem

Ek **sorted** array ko kisi unknown pivot pe **rotate** kar diya gaya hai (saare elements **distinct**). **Minimum element** dhoondo. `O(log n)` mein.

```
Input:  nums = [3, 4, 5, 1, 2]
Output: 1

Input:  nums = [4, 5, 6, 7, 0, 1, 2]
Output: 0

Input:  nums = [11, 13, 15, 17]
Output: 11   (rotation = 0, khud hi sorted hai)
```

---

## The story (yaad rakhne ke liye) 🧠

Socho ek **sorted clock face** hai — `1, 2, 3, ..., 12` clockwise badhte hue. Koi aaya aur usse **kisi bhi angle pe ghumaa diya**. Ab numbers `9, 10, 11, 12, 1, 2, 3...` jaise dikhte hain — zyadatar **badhte** hain, par kahin ek jagah achanak se **girte** hain (`12` se `1`). Wahi girne wali jagah — woh **seam** — hi tumhara **minimum** hai.

Trick: `mid` ko **`hi` se compare karo**, `lo` se nahi:

- **`nums[mid] > nums[hi]`** → matlab `mid` us "bade numbers wale" hisse mein hai jo **rotation se pehle** ka hai. Seam (minimum) `mid` ke **aage** kahin hai — `lo = mid + 1`.
- **`nums[mid] <= nums[hi]`** → matlab `mid` se `hi` tak sab **already sorted** hai (koi seam beech mein nahi). Toh minimum **`mid` hi ho sakta hai ya usse pehle** — `hi = mid`.

> 🔑 **`hi` se compare karna zaroori hai, `lo` se nahi** — kyunki humesha `[mid, hi]` ka comparison batata hai ki *is right half* mein seam hai ya nahi. `lo` se compare karne mein ambiguity aati hai jab `mid == lo`.

---

## Approach 1 — Brute force 🟥

Left se right scan karke minimum dhoondo.

```java
class Solution {
    public int findMin(int[] nums) {
        int min = nums[0];
        for (int num : nums) {
            min = Math.min(min, num);
        }
        return min;
    }
}
```

**Problem kya hai:** `O(n)` — array "mostly sorted" hone ka koi fayda nahi liya. Rotation ki structure waste ho rahi hai.

---

## Approach 2 — Optimal (Binary Search on Rotation) ✅

```java
class Solution {
    public int findMin(int[] nums) {
        int lo = 0, hi = nums.length - 1;

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;

            if (nums[mid] > nums[hi]) {
                lo = mid + 1;    // seam is to the right of mid
            } else {
                hi = mid;        // mid..hi already sorted → seam is here or to the left
            }
        }

        return nums[lo];   // == nums[hi], the minimum
    }
}
```

---

## Dry run — `nums = [3, 4, 5, 1, 2]`

| lo | hi | mid | nums[mid], nums[hi] | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 0 | 4 | 2 | 5, 2 | `5 > 2` → seam is to the right → `lo = 3` |
| 3 | 4 | 3 | 1, 2 | `1 <= 2` → sorted from here → `hi = 3` |
| 3 | 3 | — | — | `lo == hi`, **stop** |

**Result:** `nums[3] = 1` ✅

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear scan | `O(n)` | `O(1)` |
| Binary Search on Rotation | `O(log n)` | `O(1)` |

---

## Gotchas 🪤

- **`nums[mid]` vs `nums[hi]`, na ki `nums[lo]`** — yeh sabse common mistake hai. `nums[lo]` se compare karoge to `mid == lo` waale case mein galat decision aa sakta hai. `hi` se compare karna hamesha safe hai kyunki `mid < hi` (loop condition `lo < hi` ki wajah se).
- **`hi = mid`, na ki `hi = mid - 1`** — `mid` khud minimum ho sakta hai (jab `nums[mid] <= nums[hi]`), use range se bahar mat nikaalo.
- **No rotation case (`nums` already sorted)** — algorithm apne aap handle karta hai kyunki `nums[mid] <= nums[hi]` hamesha true hoga, aur `hi` continuously shrink hokar `lo` tak pahunch jaayega (index `0`).
- **Distinct elements ka assumption** — is problem (LC 153) mein duplicates nahi hote. Agar duplicates ho (LC 154, harder variant), `nums[mid] == nums[hi]` case mein `hi--` karke ambiguity todni padti hai — thoda alag, dhyaan se dekhna.
- **Return `nums[lo]`, index nahi** — problem "minimum value" maangta hai, agar tumhe **rotation count** (index of minimum) chahiye, wahi `lo` hai — dekho agli problem `number-of-rotations-in-sorted-array`.
