# Sort Colors — LeetCode 75

> 🔗 **[Interactive Visualizer dekho »]
> (https://yashika807.github.io/DSA-PatternWise/Two-pointer/dutch-national-flag.html)**
> _(step-through Red/White/Blue three-way partition, live flag readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Three Pointers (Dutch National Flag)  ·  **Tags:** Array, Two Pointers, Sorting, Partition

---

## Problem

Ek array `nums` diya hai jismein sirf `0`, `1`, `2` hain (jo **red, white, blue** colors represent karte hain). Isko **in-place** sort karo taaki same color elements **adjacent** ho jaayein, order: `0`s → `1`s → `2`s. Ek hi pass mein, koi sorting library use kiye bina.

```
Input:  nums = [2, 0, 2, 1, 1, 0]
Output: [0, 0, 1, 1, 2, 2]
```

---

## The story (yaad rakhne ke liye) 🇳🇱

Yeh problem ka naam hi **Dutch National Flag** hai — Dutch flag mein teen horizontal stripes hain: **Red (upar), White (beech), Blue (neeche)**. Humein array ko exactly aisi hi teen zones mein baantna hai.

| Role | Kaun | Kaam |
|---|---|---|
| **`low`** | Red zone ka boundary | "isse pehle sab confirmed Red (0)" |
| **`mid`** | Scanner, current element check karta hai | "yeh element kaunsa color hai?" |
| **`high`** | Blue zone ka boundary | "isse baad sab confirmed Blue (2)" |

`mid` poore array ko scan karta hai (`low` se `high` tak), aur teen decisions:

- **`nums[mid] == 0` (Red)** → Red zone mein belong karta hai → `nums[low]` se **swap** karo, `low++` **aur** `mid++` (dono aage, kyunki jo swap hoke `mid` pe aaya woh already-verified `White` hi ho sakta hai — invariant se).
- **`nums[mid] == 1` (White)** → already sahi jagah hai (White zone) → bas `mid++`.
- **`nums[mid] == 2` (Blue)** → Blue zone mein belong karta hai → `nums[high]` se **swap** karo, `high--`. **Lekin `mid` mat badhao** — jo naya element swap hoke `mid` pe aaya hai, uska color abhi tak unknown hai, usko bhi check karna hai.

---

## Approach 1 — Brute force (counting sort) 🟥

Count karo kitne `0`, `1`, `2` hain, phir overwrite karo.

```java
class Solution {
    public void sortColors(int[] nums) {
        int[] count = new int[3];
        for (int num : nums) count[num]++;

        int idx = 0;
        for (int color = 0; color < 3; color++)
            for (int c = 0; c < count[color]; c++)
                nums[idx++] = color;
    }
}
```

**Problem kya hai:** Yeh `O(n)` hai, lekin **do passes** lagti hain, aur yeh "in-place, single-pass, three-way partition" ka core interview-tested skill nahi dikhata.

---

## Approach 2 — Dutch National Flag (Three Pointers, Optimal) ✅

```java
class Solution {
    public void sortColors(int[] nums) {
        int low = 0, mid = 0, high = nums.length - 1;

        while (mid <= high) {
            if (nums[mid] == 0) {
                // Red → swap into low-zone, both boundaries advance
                swap(nums, low, mid);
                low++;
                mid++;
            } else if (nums[mid] == 1) {
                // White → already correct zone
                mid++;
            } else {
                // Blue → swap into high-zone, only high shrinks
                swap(nums, mid, high);
                high--;
                // mid NOT incremented — must re-check the swapped-in value
            }
        }
    }

    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

### Kyun `mid` case-2 mein nahi badhta

- Jab `nums[high]` se swap hota hai, humein **nahi pata** ki `nums[high]` (jo ab `nums[mid]` pe aaya) `0`, `1`, ya `2` tha. Isse turant re-check karna zaroori hai, isliye `mid` same rehta hai.
- Case `0` mein yeh problem nahi hoti kyunki `nums[low]` (jo swap hoke `mid` pe aata hai) **hamesha `1`** hota hai — invariant guarantee karta hai ki `[low, mid)` region mein sirf `White` hote hain (jab tak koi processed nahi hua tab tak `low==mid`, aur jab `low<mid` hota hai to beech ka sab already `1` verified hai).

---

## Dry run — `[2, 0, 2, 1, 1, 0]`

| low | mid | high | nums[mid] | Action | Array state |
|:---:|:---:|:---:|:---:|:---|:---|
| 0 | 0 | 5 | 2 | swap(mid,high), high-- | `[0,0,2,1,1,2]` |
| 0 | 0 | 4 | 0 | swap(low,mid), low++, mid++ | `[0,0,2,1,1,2]` |
| 1 | 1 | 4 | 0 | swap(low,mid), low++, mid++ | `[0,0,2,1,1,2]` |
| 2 | 2 | 4 | 2 | swap(mid,high), high-- | `[0,0,1,1,2,2]` |
| 2 | 2 | 3 | 1 | mid++ | `[0,0,1,1,2,2]` |
| 2 | 3 | 3 | 1 | mid++ | `[0,0,1,1,2,2]` |
| 2 | 4 | 3 | — | `mid > high`, stop | `[0,0,1,1,2,2]` |

**Result:** `[0, 0, 1, 1, 2, 2]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Counting sort (2 passes) | `O(n)` | `O(1)` | two passes: count, then overwrite |
| Dutch National Flag | `O(n)` | `O(1)` | single pass, in-place swaps |

---

## Gotchas 🪤

- **`mid` case `2` mein mat badhao** — yeh sabse common bug hai. `high` se swap hone ke baad `nums[mid]` ka naya value unverified hai, usko dobara check karna hi hoga.
- **Loop condition `mid <= high`, na ki `mid < high`** — jab `mid == high`, us akhri element ko bhi process karna zaroori hai.
- **Teen invariant zones yaad rakho:** `[0, low)` = confirmed Red, `[low, mid)` = confirmed White, `(high, n-1]` = confirmed Blue, `[mid, high]` = unexplored.
- **Ek hi pass mein poora sort** — koi separate counting phase nahi, koi extra array nahi — yehi is pattern ka poora point hai.
- **Self-swap harmless hai** — `low == mid` ya `mid == high` ho to bhi swap karna safe hai, extra condition check karne ki zaroorat nahi.
