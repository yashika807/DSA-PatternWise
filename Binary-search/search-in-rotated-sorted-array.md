# Search in Rotated Sorted Array — LeetCode 33

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/search-in-rotated-sorted-array.html)**
> _(step-through "which half is sorted", live target-in-range readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search on a Rotated Array  ·  **Tags:** Array, Binary Search

---

## Problem

Ek **sorted, distinct** array ko kisi unknown pivot pe rotate kiya gaya hai. Ek `target` diya hai — uska index dhoondo, warna `-1`. `O(log n)` mein.

```
Input:  nums = [4, 5, 6, 7, 0, 1, 2], target = 0
Output: 4

Input:  nums = [4, 5, 6, 7, 0, 1, 2], target = 3
Output: -1
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh **do techniques ka combo** hai jo tumne pehle hi seekhi hain — same rotated clock face (`find-minimum-in-rotated-sorted-array` wala), par ab humein **specific number** dhoondhna hai, seam nahi.

Trick: har step pe tum khud se ek sawaal poochte ho — *"in do halves mein se, kaunsa hissa **guaranteed sorted** hai?"* (bilkul seam-finding jaisa check: `nums[lo] <= nums[mid]`?)

Jo half sorted hai, usme tum **normal binary search ka range-check** laga sakte ho (`binary-search-basics` wala): *"kya target is sorted range ke andar aata hai?"*

- **Haan** → target usi (sorted) half mein hai, wahi search karo.
- **Nahi** → target doosri (possibly-unsorted, lekin still-valid) half mein hoga — wahaan jao.

| Step | Kya poochha |
|---|---|
| 1 | `nums[mid] == target`? Mil gaya, return karo. |
| 2 | `nums[lo] <= nums[mid]`? → **Left half sorted hai.** |
| 3a | Agar left sorted: `target` `[nums[lo], nums[mid])` mein hai? → left mein jao, warna right mein. |
| 3b | Agar right sorted (else case): `target` `(nums[mid], nums[hi]]` mein hai? → right mein jao, warna left mein. |

---

## Approach 1 — Brute force 🟥

Linear scan.

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

**Problem kya hai:** `O(n)` — rotation ki sorted structure completely waste.

---

## Approach 2 — Optimal (Binary Search + Half-Sorted Check) ✅

```java
class Solution {
    public int search(int[] nums, int target) {
        int lo = 0, hi = nums.length - 1;

        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;

            if (nums[mid] == target) return mid;

            if (nums[lo] <= nums[mid]) {
                // left half [lo..mid] is sorted
                if (nums[lo] <= target && target < nums[mid]) {
                    hi = mid - 1;      // target in the sorted left half
                } else {
                    lo = mid + 1;      // target must be in the right half
                }
            } else {
                // right half [mid..hi] is sorted
                if (nums[mid] < target && target <= nums[hi]) {
                    lo = mid + 1;      // target in the sorted right half
                } else {
                    hi = mid - 1;      // target must be in the left half
                }
            }
        }

        return -1;
    }
}
```

---

## Dry run — `nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 0`

| lo | hi | mid | nums[mid] | Which half sorted? | Target in it? | Verdict |
|:---:|:---:|:---:|:---:|:---|:---|:---|
| 0 | 6 | 3 | 7 | `nums[0]=4 <= nums[3]=7` → **left sorted** `[4,5,6,7]` | `4 <= 0`? No | target in right → `lo = 4` |
| 4 | 6 | 5 | 1 | `nums[4]=0 <= nums[5]=1` → **left sorted** `[0,1]` | `0 <= 0 < 1`? Yes | target in left → `hi = 4` |
| 4 | 4 | 4 | 0 | `nums[4] == target` | — | ✅ **found at index 4** |

**Result:** `4` ✅

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear scan | `O(n)` | `O(1)` |
| Binary Search + Half-Sorted Check | `O(log n)` | `O(1)` |

---

## Gotchas 🪤

- **`nums[lo] <= nums[mid]`, na ki `<`** — equal hone ka case bhi "sorted" maano (jab `lo == mid`, single element hamesha "sorted" hota hai).
- **Range check mein boundary dhyaan se** — left sorted case mein `nums[lo] <= target < nums[mid]` (mid khud exclude, kyunki already check ho chuka `!=`). Right sorted case mein `nums[mid] < target <= nums[hi]`. In `<=`/`<` ko galat jagah lagaoge to kabhi-kabhi wrong half mein chale jaoge.
- **Match check sabse pehle karo** (`nums[mid] == target`) — us decision ke baad hi "kaunsi half sorted hai" wala logic chalao, warna unnecessary complexity.
- **Yeh technique sirf tab kaam karti hai jab elements distinct hon** — duplicates ke saath (LC 81), `nums[lo] == nums[mid] == nums[hi]` jaisa case "kaunsi half sorted hai" ko ambiguous bana deta hai, extra handling chahiye (`lo++, hi--` karke shrink karo jab ambiguous ho).
- **Do independent decisions mat mix karo** — pehle "kaunsi half sorted" phir "target us range mein hai kya" — dono ko ek hi `if` mein cram karne ki koshish mat karo, bugs aayenge.
