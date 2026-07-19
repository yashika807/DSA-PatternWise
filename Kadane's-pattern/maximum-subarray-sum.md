Interactive Visual: [maximum-subarray-sum.html]

(https://yashika807.github.io/DSA-PatternWise/Kadane's-pattern/maximum-subarray-sum.html)

# 53. Maximum Subarray 📈

> **Pattern:** Kadane's Algorithm → *extend karu ya restart karu*
> **Difficulty:** 🟡 Medium
> **Problem:** [LeetCode 53](https://leetcode.com/problems/maximum-subarray/)

---

## 📌 Problem kya keh raha hai

Ek integer array `nums` diya hai. Iske andar se koi bhi **contiguous** (non-empty) subarray chuno jiska **sum sabse zyada** ho, aur woh maximum sum return karo.

Yeh Kadane's Algorithm ka **sabse classic, sabse original** problem hai — baaki saare variants (circular, one-deletion, product) isi ke upar bane hain.

### Examples

| Input | Output | Kyun |
|---|---|---|
| `[-2,1,-3,4,-1,2,1,-5,4]` | `6` | `[4,-1,2,1]` — beech ka positive-heavy chunk |
| `[1]` | `1` | sirf ek element, wahi answer |
| `[5,4,-1,7,8]` | `23` | poora array hi best subarray hai |

**Constraints:** `1 ≤ nums.length ≤ 10⁵`, `-10⁴ ≤ nums[i] ≤ 10⁴`

---

## ❌ Galat approaches (jo trap karte hain)

**1. "Sirf positive numbers chun ke jod do"**
Lagta hai simple hai — array mein se saare positive numbers nikal ke unka sum kar do. Par yeh **contiguity** todta hai! `[-2,1,-3,4,-1,2,1,-5,4]` mein saare positives ka sum = `1+4+2+1+4 = 12`, lekin yeh contiguous nahi hai. Sahi answer `6` hai (`[4,-1,2,1]`). ❌

**2. "curSum ko 0 pe reset karo jab negative ho, aur sirf last curSum return karo"**
Yeh Kadane's jaisa dikhta hai par **maxSum track karna bhool jaata hai** — sirf final `curSum` return kar deta hai. All-negative array pe fail: `[-3,-2,-5]` mein reset karte karte `curSum` `0` pe atak jaata hai (ya last element pe), aur galat answer `0` ya `-5` de sakta hai — jabki sahi answer `-2` hai (single best negative element). Har step pe **best-so-far** track karna zaroori hai, sirf running value nahi. ❌

**3. "Poore array ka total sum hi answer hai"**
Bilkul galat sochte hain kuch log — pura array le lo, done. `[-2,1,-3,4,-1,2,1,-5,4]` ka total sum = `1`, jabki sahi answer `6` hai. Negative numbers beech mein aakar total ko neeche khींch dete hain. ❌

---

## 💡 Sahi intuition

Har index `i` pe ek hi decision leni hai:

> **"Pichla running subarray extend karu, ya yahin se naya subarray shuru karu?"**

Agar pichla `curSum` **negative** hai, to usse aage carry karna sirf bojh hai — better hai wahi se fresh restart. Agar `curSum` **positive/zero** hai, to use extend karna hi fayda hai.

```
curSum = max(nums[i], curSum + nums[i])
```

Aur **har step** pe global best update karte raho:

```
maxSum = max(maxSum, curSum)
```

> 🔑 **Key insight:** `maxSum` sirf loop ke end mein nahi, **har single index pe** update hota hai — kyunki best subarray kahin bhi beech mein khatam ho sakta hai, zaroori nahi last index pe.

---

## 🧮 Dry Run — `[-2, 1, -3, 4, -1, 2, 1, -5, 4]`

Init: `curSum = -2`, `maxSum = -2`

| i | num | curSum = max(num, curSum+num) | maxSum |
|:-:|:---:|:------------------------------:|:------:|
| 0 | -2 | init | **-2** |
| 1 | 1 | max(1, -2+1=-1) = **1** | **1** |
| 2 | -3 | max(-3, 1-3=-2) = **-2** | 1 |
| 3 | 4 | max(4, -2+4=2) = **4** | **4** |
| 4 | -1 | max(-1, 4-1=3) = **3** | 4 |
| 5 | 2 | max(2, 3+2=5) = **5** | **5** |
| 6 | 1 | max(1, 5+1=6) = **6** | **6** |
| 7 | -5 | max(-5, 6-5=1) = **1** | 6 |
| 8 | 4 | max(4, 1+4=5) = **5** | 6 |

**Answer: `6`** — subarray `[4,-1,2,1]` (indices 3..6). ✅

---

## ☕ Java Solution

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int curSum = nums[0];
        int maxSum = nums[0];

        for (int i = 1; i < nums.length; i++) {
            curSum = Math.max(nums[i], curSum + nums[i]);
            maxSum = Math.max(maxSum, curSum);
        }

        return maxSum;
    }
}
```

## 🐍 Python Solution

```python
class Solution:
    def maxSubArray(self, nums: list[int]) -> int:
        cur_sum = nums[0]
        max_sum = nums[0]

        for num in nums[1:]:
            cur_sum = max(num, cur_sum + num)
            max_sum = max(max_sum, cur_sum)

        return max_sum
```

---

## ⏱ Complexity

| | |
|---|---|
| **Time** | `O(n)` — ek hi single pass |
| **Space** | `O(1)` — bas do variables, `curSum` aur `maxSum` |

---

## 🧪 Edge cases (dhyaan se!)

| Case | Example | Kya hota hai |
|---|---|---|
| **Saare negative** | `[-3,-2,-5]` | `curSum` bhi restart karte hue negative hi rahega; `maxSum` sabse kam-negative single element pakadta hai → `-2` |
| **Single element** | `[5]` | Loop chalta hi nahi (start `i=1` se), seedha `nums[0] = 5` answer |
| **Saare positive** | `[5,4,-1,7,8]` | Kabhi restart nahi hota, poora array ek hi subarray ban jaata hai → `23` |
| **Ek hi negative beech mein** | `[8,-1,9]` | `curSum` kabhi negative nahi hota, extend hi jeetega → `16` |

> ⚠️ **Empty subarray allowed nahi:** isiliye `curSum` aur `maxSum` dono **`nums[0]`** se seed hote hain, `0` se nahi — warna all-negative array pe galat `0` answer aa jaata.

---

## 🔁 Quick Revision (30 second recap)

1. Har index pe ek decision: **extend karu ya restart** → `curSum = max(nums[i], curSum+nums[i])`.
2. Har step pe **`maxSum = max(maxSum, curSum)`** update karo — best kahin bhi beech mein mil sakta hai.
3. Seed `curSum` aur `maxSum` dono `nums[0]` se, `0` se nahi — all-negative case ke liye zaroori.
4. `O(n)` time, `O(1)` space — Kadane's ka sabse pure form, baaki variants isi pe based hain.

---
