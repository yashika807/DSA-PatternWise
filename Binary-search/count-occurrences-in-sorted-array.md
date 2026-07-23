# Count Occurrences in a Sorted Array — GFG

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/count-occurrences-in-sorted-array.html)**
> _(reuses the [first, last] range from the previous problem, then does subtraction)_

**Difficulty:** Easy  ·  **Pattern:** Binary Search — Lower Bound × 2 (+ subtraction)  ·  **Tags:** Array, Binary Search

**Problem link:** [GFG — Number of occurrence](https://www.geeksforgeeks.org/problems/number-of-occurrence2259/1)

---

## Problem

Sorted integer array diya hai. Ek `target` (GFG pe usually `x` bola jaata hai) **kitni baar** array mein occur karta hai, wo count karo.

```
Input:  nums = [1, 1, 2, 2, 2, 3], target = 2
Output: 3

Input:  nums = [1, 1, 2, 2, 2, 3], target = 4
Output: 0
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh **seedha follow-up** hai `first-and-last-position` ka. Humein wahaan pichhli problem mein **first aur last platform** (occurrence) mil chuki hai jahaan se 8:00 baje ki trains chalti hain. Ab bas poochna hai: *"Total kitni platforms hain?"*

Simple arithmetic: agar first platform `3` hai aur last platform `4` hai, to platforms `3, 4` — **do** platforms, `last - first + 1`.

> 🔑 Koi naya binary search nahi likhna — **`first-and-last-position` ka code hi copy-paste** karo aur ek line jodo. Yeh dikhata hai ki DSA mein techniques kaise **layer** hoti hain: ek building block (`lowerBound`) → doosri technique (`[first,last]`) → teesri (`count`).

---

## Approach 1 — Brute force 🟥

Poore array ko scan karke count badhao.

```java
class Solution {
    public int countOccurrences(int[] nums, int target) {
        int count = 0;
        for (int num : nums) {
            if (num == target) count++;
        }
        return count;
    }
}
```

**Problem kya hai:** `O(n)` — chhote arrays ke liye theek hai, par sorted property waste ho rahi hai. Bade `n` (GFG constraints `10^6` tak) pe slow.

---

## Approach 2 — Optimal (Binary Search) ✅

`first-and-last-position` ka `searchRange` reuse karo, phir subtract karo.

```java
class Solution {
    public int countOccurrences(int[] nums, int target) {
        int[] range = searchRange(nums, target);
        if (range[0] == -1) return 0;          // target hai hi nahi
        return range[1] - range[0] + 1;         // inclusive count
    }

    private int[] searchRange(int[] nums, int target) {
        int first = lowerBound(nums, target);
        if (first == nums.length || nums[first] != target) {
            return new int[]{-1, -1};
        }
        int last = lowerBound(nums, target + 1) - 1;
        return new int[]{first, last};
    }

    private int lowerBound(int[] nums, int t) {
        int lo = 0, hi = nums.length;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] >= t) hi = mid;
            else lo = mid + 1;
        }
        return lo;
    }
}
```

---

## Dry run — `nums = [1, 1, 2, 2, 2, 3]`, `target = 2`

`searchRange(nums, 2)` chalao (jaisa pichhli problem mein dekha):

- `lowerBound(nums, 2)` → ceiling of `2` → **first = 2** (`nums[2] = 2`)
- `lowerBound(nums, 3)` → ceiling of `3` → index `5` (`nums[5] = 3`), so **last = 5 - 1 = 4**

| Step | Value |
|---|---|
| first | `2` |
| last | `4` |
| count | `4 - 2 + 1 = 3` |

**Result:** `3` ✅ (`nums[2..4] = [2, 2, 2]`)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear scan | `O(n)` | `O(1)` |
| Binary Search (reuse) | `O(log n)` | `O(1)` |

---

## Gotchas 🪤

- **`last - first + 1`, na ki `last - first`** — dono indices **inclusive** hain, off-by-one mat karo.
- **`range[0] == -1` check pehle** — agar target array mein nahi hai, seedha `0` return karo. `-1 - (-1) + 1 = 1` galat answer dega agar yeh check chhoot gaya!
- **Poora array ek hi value ho sakta hai** — edge case jahaan `first = 0` aur `last = n - 1`. Formula abhi bhi kaam karta hai, par test zaroor karo.
- **Yeh technique har jagah generalize hoti hai** jahaan "kitni baar X hua" poocha jaaye sorted data mein — range dhoondo, subtract karo, `O(log n)`.
