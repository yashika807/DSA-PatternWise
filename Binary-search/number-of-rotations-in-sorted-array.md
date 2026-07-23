# Number of Rotations in a Sorted Array — GFG-style

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/number-of-rotations-in-sorted-array.html)**
> _(same seam-finding search as the previous problem — this time we keep the index, not the value)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search on a Rotated Array  ·  **Tags:** Array, Binary Search

---

## Problem

Ek sorted array ko kisi unknown number of baar **rotate** kiya gaya hai (saare elements distinct). Batao ki array ko **kitni baar rotate** kiya gaya — matlab **rotation count**.

```
Input:  nums = [3, 4, 5, 1, 2]
Output: 3   (original [1,2,3,4,5] ko 3 baar left-rotate kiya gaya)

Input:  nums = [11, 13, 15, 17]
Output: 0   (bilkul rotate nahi hua)
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh **seedha follow-up** hai `find-minimum-in-rotated-sorted-array` ka — **same clock face, same seam**. Pichhli problem mein humne seam **dhoondhi** aur uski **value** return ki. Is baar bas poochha jaa raha hai: *"Seam kis position pe hai?"* — yaani seam ka **index**.

> 🔑 **Insight:** rotation count **hamesha minimum element ka index hota hai**. Socho: original sorted array mein minimum hamesha index `0` pe hota hai. Har rotation minimum ko ek position aage dhakelta hai. Toh minimum jitni jagah khisak gaya hai, utni hi baar rotation hui hai.

Koi naya algorithm nahi — **wahi binary search**, bas end mein `nums[lo]` (value) ki jagah `lo` (index) return karo.

---

## Approach 1 — Brute force 🟥

Minimum element ka index dhoondo linear scan se.

```java
class Solution {
    public int countRotations(int[] nums) {
        int minIdx = 0;
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] < nums[minIdx]) minIdx = i;
        }
        return minIdx;
    }
}
```

**Problem kya hai:** `O(n)` — wahi purani baat, rotation ki structure ka fayda nahi utha rahe.

---

## Approach 2 — Optimal (Binary Search on Rotation) ✅

```java
class Solution {
    public int countRotations(int[] nums) {
        int lo = 0, hi = nums.length - 1;

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;

            if (nums[mid] > nums[hi]) {
                lo = mid + 1;    // seam is to the right
            } else {
                hi = mid;        // mid..hi already sorted
            }
        }

        return lo;   // index of the minimum == rotation count
    }
}
```

---

## Dry run — `nums = [3, 4, 5, 1, 2]`

| lo | hi | mid | nums[mid], nums[hi] | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 0 | 4 | 2 | 5, 2 | `5 > 2` → seam right → `lo = 3` |
| 3 | 4 | 3 | 1, 2 | `1 <= 2` → sorted → `hi = 3` |
| 3 | 3 | — | — | `lo == hi`, **stop** |

**Result:** `lo = 3` → array was rotated **3** times ✅ (`[1,2,3,4,5]` → rotate 3 → `[3,4,5,1,2]`)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear scan | `O(n)` | `O(1)` |
| Binary Search on Rotation | `O(log n)` | `O(1)` |

---

## Gotchas 🪤

- **Sirf return statement badla hai** — algorithm bilkul same hai `find-minimum-in-rotated-sorted-array` ka. Yeh dikhata hai ki interview mein "value do" vs "index do" ka farq **sirf ek line** ka ho sakta hai agar underlying search sahi hai.
- **Rotation count `0` se `n-1` tak ho sakti hai** — array khud sorted ho (rotation `0`) ya poori tarah rotate ho chuki ho, dono valid outputs.
- **`nums[hi]` se compare karo, `nums[lo]` se nahi** — same trap jo pichhli problem mein tha.
- **`hi = mid` (na `mid-1`)** — `mid` khud minimum index ho sakta hai.
- **Distinct elements chahiye** — duplicates hone pe rotation count ambiguous ho sakta hai (jaise `[1,1,1,0,1]` — kaunsa `1` "original start" tha?), is technique ke liye distinct assumption zaroori hai.
