# Shortest Unsorted Continuous Subarray — LeetCode 581

> 🔗 **[Interactive Visualizer dekho »]
> (https://yashika807.github.io/DSA-PatternWise/Two-pointer/minimum-window-sort.html)**
> _(step-through two Sorting Detectives scanning from both ends, live boundary readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Two Pointers (boundary scan from both ends)  ·  **Tags:** Array, Two Pointers

---

## Problem

Ek integer array `nums` diya hai. Sabse **chhota continuous subarray** dhoondo jisko sort karne se **poora array sorted** ho jaaye. Uski **length** return karo (agar array already sorted hai, `0` return karo).

```
Input:  nums = [2, 6, 4, 8, 10, 9, 15]
Output: 5        // sort [6,4,8,10,9] → poora array sorted ho jaata hai
```

---

## The story (yaad rakhne ke liye) 🕵️

Do **Sorting Detectives** socho jo array ke **dono ends se** apni jaanch shuru karte hain, "mess" (out-of-order zone) ki **exact boundaries** dhoondne ke liye.

| Role | Kaun | Kaam |
|---|---|---|
| **Detective A** (left→right) | ek **"highest seen so far"** watch rakhta hai (`max`) | "agar current element mere max se **chhota** hai, to yahan kuch gadbad hai" |
| **Detective B** (right→left) | ek **"lowest seen so far"** watch rakhta hai (`min`) | "agar current element mere min se **bada** hai, to yahan kuch gadbad hai" |

**Detective A** left se right chalta hai, `max` ko update karta jaata hai. Jab bhi `nums[i] < max` milta hai (matlab yeh element apne se pehle wale kisi bade element se **peeche** hona chahiye tha — out of order), woh index **`right` boundary** ke liye candidate ban jaata hai — **last aisa index jo mile, woh final `right`** hai.

**Detective B** right se left chalta hai, `min` ko update karta jaata hai. Jab bhi `nums[i] > min` milta hai (yeh element apne se aage wale kisi chhote element se **aage** hona chahiye tha), woh index **`left` boundary** ke liye candidate ban jaata hai — **last aisa index jo mile (yaani sabse left), woh final `left`** hai.

Jitna zone dono detectives ne "suspicious" flag kiya (`[left, right]`), utna hi minimum window sort karna padega.

---

## Approach 1 — Brute force (sort + compare) 🟥

Copy banao, sort karo, dono ends se pehla/aakhri mismatch dhoondo.

```java
class Solution {
    public int findUnsortedSubarray(int[] nums) {
        int n = nums.length;
        int[] sorted = nums.clone();
        Arrays.sort(sorted);

        int left = 0;
        while (left < n && nums[left] == sorted[left]) left++;
        if (left == n) return 0;   // already sorted

        int right = n - 1;
        while (nums[right] == sorted[right]) right--;

        return right - left + 1;
    }
}
```

**Problem kya hai:** `O(n log n)` sorting ke liye, plus `O(n)` extra space naye sorted array ke liye. Two-pointer approach isse `O(n)` time aur `O(1)` extra space mein kar sakta hai.

---

## Approach 2 — Two Pointers, boundary scan (Optimal) ✅

```java
class Solution {
    public int findUnsortedSubarray(int[] nums) {
        int n = nums.length;
        int left = -1, right = -2;      // default: already sorted (length 0)
        int max = Integer.MIN_VALUE;
        int min = Integer.MAX_VALUE;

        // Detective A: left → right, tracks running max, flags dips
        for (int i = 0; i < n; i++) {
            max = Math.max(max, nums[i]);
            if (nums[i] < max) right = i;      // out of order → extend right boundary
        }

        // Detective B: right → left, tracks running min, flags rises
        for (int i = n - 1; i >= 0; i--) {
            min = Math.min(min, nums[i]);
            if (nums[i] > min) left = i;       // out of order → extend left boundary
        }

        return right - left + 1;
    }
}
```

### Kyun yeh sahi window deta hai

- Agar `nums[i]` apne prefix ke max se chhota hai, iska matlab hai isse **pehle** aana chahiye tha — yeh disorder ka signal hai. Aakhri baar jahan yeh hua, wahi disorder ka **right edge** hai (kyunki uske baad sab kuch already-correct max ke barabar ya bada rehta hai jab tak sorted ho).
- Symmetric logic **min** ke saath right se left scan mein **left edge** deta hai.
- `left == -1` (koi flag nahi hua) matlab array **already sorted** hai → `right - left + 1 = -2 - (-1) + 1 = 0`.

---

## Dry run — `[2, 6, 4, 8, 10, 9, 15]`

**Detective A (left → right, tracks max):**

| i | nums[i] | max (after) | nums[i] &lt; max? | right (updated) |
|:---:|:---:|:---:|:---:|:---:|
| 0 | 2 | 2 | no | -2 |
| 1 | 6 | 6 | no | -2 |
| 2 | 4 | 6 | **yes** | **2** |
| 3 | 8 | 8 | no | 2 |
| 4 | 10 | 10 | no | 2 |
| 5 | 9 | 10 | **yes** | **5** |
| 6 | 15 | 15 | no | 5 |

**Detective B (right → left, tracks min):**

| i | nums[i] | min (after) | nums[i] &gt; min? | left (updated) |
|:---:|:---:|:---:|:---:|:---:|
| 6 | 15 | 15 | no | -1 |
| 5 | 9 | 9 | no | -1 |
| 4 | 10 | 9 | **yes** | **4** |
| 3 | 8 | 8 | no | 4 |
| 2 | 4 | 4 | no | 4 |
| 1 | 6 | 4 | **yes** | **1** |
| 0 | 2 | 2 | no | 1 |

**Result:** `left = 1`, `right = 5` → length `= 5 - 1 + 1 = 5` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Sort + compare | `O(n log n)` | `O(n)` | copies + sorts the array |
| Two Pointers (boundary scan) | `O(n)` | `O(1)` | two linear passes, no extra array |

---

## Gotchas 🪤

- **`left = -1, right = -2` default** — yeh purposely aisi values hain ki agar kabhi koi disorder na mile, `right - left + 1 = 0` khud hi correct answer de de (already-sorted case).
- **Do independent passes, ek converge hote pointers nahi** — is problem mein `left`/`right` 3Sum jaisi ek-doosre ki taraf nahi chalte; yeh do **alag directions se boundary-detecting scans** hain jo end mein combine hoti hain.
- **`max`/`min` string ki initial values** — `Integer.MIN_VALUE`/`MAX_VALUE` use karo, warna pehle hi element pe galat comparison ho sakta hai.
- **Duplicate values pe bhi kaam karta hai** — agar `nums[i] == max` (barabar), condition `nums[i] < max` false rahegi, isliye duplicates false-positive disorder flag nahi karte.
- **Single element ya already-sorted array** — dono cases mein loop chalega but koi `left`/`right` update nahi hoga, default values se `0` return hoga — correct.
