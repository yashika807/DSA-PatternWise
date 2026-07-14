# 3Sum Closest — LeetCode 16

> 🔗 **[Interactive Visualizer dekho »]
> (https://yashika807.github.io/DSA-PatternWise/Two-pointer/triplet-sum-close-to-target.html)**
> _(step-through Archer's Post + two Arrows closing in on the bullseye, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sort + Two Pointers  ·  **Tags:** Array, Two Pointers, Sorting

---

## Problem

Ek integer array `nums` aur ek `target` diya hai. Exactly teen integers dhoondo jinka sum **`target` ke sabse close** ho. Assume **exactly ek solution** hai.

```
Input:  nums = [-1, 2, 1, -4], target = 1
Output: 2        // (-1 + 2 + 1 = 2, jo 1 ke sabse close hai)
```

---

## The story (yaad rakhne ke liye) 🎯

Ek **archer** socho jo teen teer chalata hai ek target board pe, aur board pe ek **bullseye** hai jiska number `target` hai. Archer ka goal — teeno teeron ka combined sum bullseye ke **sabse paas** le jaana, chahe **exact** hit ho ya nahi.

| Role | Kaun | Kaam |
|---|---|---|
| **Post** (`i`) | left→right chalta hai (anchor) | "Mera teer fixed hai. Baaki do se best combo dhoondo." |
| **Left Arrow** (`left`) | post ke just right | "kam sum" ki taraf |
| **Right Arrow** (`right`) | array ke end pe | "zyada sum" ki taraf |

Har combo pe ek **"Best Shot So Far"** record hoti hai — jo sum abhi tak `target` ke sabse close hai:

- **sum &lt; target** → shot bahut halka gaya → Left Arrow ek kadam aage (`left++`) taaki sum badhe
- **sum &gt; target** → shot bahut door gaya → Right Arrow ek kadam peeche (`right--`) taaki sum ghate
- **sum == target** → **perfect bullseye!** 🎯 Isse aur close nahi ho sakta, turant return kar do.

> ⚠️ Har combination ke baad **`|sum - target|`** ko best-so-far se compare karo — sirf `sum == target` dhoondna kaafi nahi hai (woh guaranteed nahi milega).

---

## Approach 1 — Brute force 🟥

Teeno loops se har triplet try karo.

```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        int n = nums.length;
        int closestSum = nums[0] + nums[1] + nums[2];

        for (int i = 0; i < n; i++)
            for (int j = i + 1; j < n; j++)
                for (int k = j + 1; k < n; k++) {
                    int sum = nums[i] + nums[j] + nums[k];
                    if (Math.abs(sum - target) < Math.abs(closestSum - target)) {
                        closestSum = sum;
                    }
                }

        return closestSum;
    }
}
```

**Problem kya hai:** `O(n³)` time — sorting ka koi fayda nahi uthaya, aur bade `n` pe TLE ho jaayega.

---

## Approach 2 — Sort + Two Pointers (Optimal) ✅

```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        Arrays.sort(nums);                          // sort the range
        int n = nums.length;
        int closestSum = nums[0] + nums[1] + nums[2];   // seed with first triplet

        for (int i = 0; i < n - 2; i++) {
            int left = i + 1, right = n - 1;

            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];

                // update best shot so far
                if (Math.abs(sum - target) < Math.abs(closestSum - target)) {
                    closestSum = sum;
                }

                if (sum == target) {
                    return sum;              // perfect bullseye, can't get closer
                } else if (sum < target) {
                    left++;                  // too low → move toward bigger
                } else {
                    right--;                 // too high → move toward smaller
                }
            }
        }

        return closestSum;
    }
}
```

### Kyun yeh sabse close guarantee karta hai

- Sorted array pe two-pointer **monotonic search** karta hai — `left++` se sum sirf badhta hai, `right--` se sirf ghatta hai, isliye har possible "direction" explore ho jaati hai bina kisi combo ko skip kiye.
- Har `i` fix karne pe, inner `while` loop us anchor ke liye **best possible pair** dhoondh leta hai `O(n)` mein — total `O(n²)`.

---

## Dry run — `[-1, 2, 1, -4]`, `target = 1`

Sorted: **`[-4, -1, 1, 2]`** &nbsp;·&nbsp; seed `closestSum = nums[0]+nums[1]+nums[2] = -4-1+1 = -4`

| i | left, right | nums[left], nums[right] | sum | \|sum-target\| | closestSum (after) | Verdict |
|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| 0 (`-4`) | 1, 3 | -1, 2 | -3 | 4 (< 5) | **-3** | too low → `left++` |
| 0 (`-4`) | 2, 3 | 1, 2 | -1 | 2 (< 4) | **-1** | too low → `left++`, then `left==right`, stop |
| 1 (`-1`) | 2, 3 | 1, 2 | 2 | 1 (< 2) | **2** | too high → `right--`, then `left==right`, stop |

**Result:** `closestSum = 2` ✅ (`-1 + 2 + 1 = 2`, distance `1` from target `1`)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (3 loops) | `O(n³)` | `O(1)` | TLE on large `n` |
| Sort + Two Pointers | `O(n²)` | `O(1)` extra* | sort `O(n log n)`, dominated by `n²` scan |

\* sort ke recursion stack ko chhod kar.

---

## Gotchas 🪤

- **`sum == target` pe turant return** — usse close koi aur combo ho hi nahi sakta, early exit se time bachao.
- **Seed `closestSum` initialize karna zaroori** — pehle triplet (`nums[0]+nums[1]+nums[2]`) se seed karo, warna comparison ke liye koi baseline nahi hoga.
- **`Math.abs(sum - target)` se compare karo, `sum` se direct nahi** — matlab hai closeness dono directions (kam ya zyada) mein equally important hai.
- **Duplicate skip yahan zaroori nahi** — 3Sum ke ulat, yahan hume sirf **ek best sum value** chahiye, saare unique triplets nahi — isliye duplicate-anchor skip optimization optional hai (performance ke liye helpful ho sakta hai, correctness ke liye zaroori nahi).
- **`i < n - 2`** — anchor ke aage kam se kam do elements bache hone chahiye left/right ke liye.
- **Tie-breaking** — agar do sums equally close hain target se (`|sum1-target| == |sum2-target|`), koi bhi valid answer hai; problem guarantee karta hai unique answer, to practically yeh case nahi aayega standard test cases mein.
