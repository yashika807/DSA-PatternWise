Interactive Visual: [maximum-absolute-sum-of-any-subarray.html]

(https://yashika807.github.io/DSA-PatternWise/Kadane's-pattern/maximum-absolute-sum-of-any-subarray.html)

# 1749. Maximum Absolute Sum of Any Subarray 🎯

> **Pattern:** Kadane's Algorithm → *do Kadane's ek saath (max + min), 0 se seed*
> **Difficulty:** 🟡 Medium
> **Problem:** [LeetCode 1749](https://leetcode.com/problems/maximum-absolute-sum-of-any-subarray/)

---

## 📌 Problem kya keh raha hai

Ek integer array `nums` diya hai. Iske kisi bhi **contiguous** subarray ka sum nikaalo, uska **absolute value** lo, aur poore array mein se sabse bada absolute value return karo.

Matlab: `max(|sum(subarray)|)` — chahe woh subarray bahut **positive** ho ya bahut **negative**, dono ginte hain!

### Examples

| Input | Output | Kyun |
|---|---|---|
| `[1,-3,2,3,-4]` | `5` | `[2,3]` → `|2+3| = 5` |
| `[2,-5,1,-4,3,-2]` | `8` | `[-5,1,-4]` → `|-5+1-4| = 8` |
| `[-1,-2,-3]` | `6` | poora array → `|-6| = 6` |

**Constraints:** `1 ≤ nums.length ≤ 10⁵`, `-10⁴ ≤ nums[i] ≤ 10⁴`

---

## ❌ Galat approaches (jo trap karte hain)

**1. "Sirf normal Kadane's (max sum) laga do"**
Lagta hai bas maximum subarray sum nikal lo aur uska abs le lo. Par **negative subarrays ko poora ignore kar diya**! `[2,-5,1,-4,3,-2]` mein normal Kadane's ka max sum sirf `3` milega, jabki sahi answer `8` hai — jo aata hai **negative** subarray `[-5,1,-4]` se (`|-8+... | = 8`). Sirf positive side dekhna kaafi nahi. ❌

**2. "Har element ka abs le lo, phir Kadane's chalao"**
`abs(sum(subarray))` aur `sum(abs(subarray)))` **bilkul alag** cheezein hain! `[2,-5,1,-4,3,-2]` ke saare elements ka abs `[2,5,1,4,3,2]` — inka sum karne se `17` milega (sab positive hain to poora array hi best), jo sahi answer `8` se kaafi door hai. Abs **sum ke baad** lagana hai, individual elements pe nahi. ❌

**3. "Poore array ka sum ka abs le lo (subarray consider hi mat karo)"**
`[1,-3,2,3,-4]` ka poora array sum `= 1-3+2+3-4 = -1`, abs `= 1`. Par sahi answer `5` hai (`[2,3]` se). Poora array best subarray ho, zaroori nahi. ❌

---

## 💡 Sahi intuition

Maximum `|sum|` do jagah se aa sakta hai:

- Ya to **sabse bada positive-sum subarray** se (`maxSum`)
- Ya **sabse bada negative-sum subarray** se, jiska abs value bada hoga (`-minSum`)

Isliye **do Kadane's ek saath** chalao — ek `max` ke liye, ek `min` ke liye:

```
curMax = max(curMax + num, num);   maxSum = max(maxSum, curMax);
curMin = min(curMin + num, num);   minSum = min(minSum, curMin);
```

**Final answer:**

```
answer = max(maxSum, -minSum)
```

> 🔑 **Key insight:** Yahan `curMax`/`curMin`/`maxSum`/`minSum` sabko **`0`** se seed karte hain (`nums[0]` se nahi)! Yeh safe hai kyunki jab tak array mein koi non-zero element hai, uska khud ka candidate (`num` akela) `maxSum` ya `minSum` ko turant `0` se door kar dega. Aur agar poora array `0` hi ho, to sahi answer bhi `0` hai — to `0` seed kabhi galat answer nahi deta.

---

## 🧮 Dry Run — `[1, -3, 2, 3, -4]`

Init: `curMax = maxSum = curMin = minSum = 0`

| i | num | curMax=max(curMax+num,num) | maxSum | curMin=min(curMin+num,num) | minSum |
|:-:|:---:|:---:|:---:|:---:|:---:|
| 0 | 1 | max(0+1,1)=**1** | **1** | min(0+1,1)=**1** | 0 |
| 1 | -3 | max(1-3,-3)=**-2** | 1 | min(1-3,-3)=**-3** | **-3** |
| 2 | 2 | max(-2+2,2)=**2** | **2** | min(-3+2,2)=**-1** | -3 |
| 3 | 3 | max(2+3,3)=**5** | **5** | min(-1+3,3)=**2** | -3 |
| 4 | -4 | max(5-4,-4)=**1** | 5 | min(2-4,-4)=**-4** | **-4** |

**Final:** `maxSum = 5`, `minSum = -4` → `answer = max(5, -(-4)) = max(5, 4) = ` **`5`** ✅

---

## ☕ Java Solution

```java
class Solution {
    public int maxAbsoluteSum(int[] nums) {
        int curMax = 0, maxSum = 0;
        int curMin = 0, minSum = 0;

        for (int num : nums) {
            curMax = Math.max(curMax + num, num);
            maxSum = Math.max(maxSum, curMax);

            curMin = Math.min(curMin + num, num);
            minSum = Math.min(minSum, curMin);
        }

        return Math.max(maxSum, -minSum);
    }
}
```

## 🐍 Python Solution

```python
class Solution:
    def maxAbsoluteSum(self, nums: list[int]) -> int:
        cur_max = max_sum = 0
        cur_min = min_sum = 0

        for num in nums:
            cur_max = max(cur_max + num, num)
            max_sum = max(max_sum, cur_max)

            cur_min = min(cur_min + num, num)
            min_sum = min(min_sum, cur_min)

        return max(max_sum, -min_sum)
```

---

## ⏱ Complexity

| | |
|---|---|
| **Time** | `O(n)` — ek hi single pass, do Kadane's saath saath |
| **Space** | `O(1)` — sirf 4 variables |

---

## 🧪 Edge cases (dhyaan se!)

| Case | Example | Kya hota hai |
|---|---|---|
| **Saare positive** | `[1,2,3]` | `minSum` `0` pe hi atka rahega (kabhi negative nahi hota), `maxSum = 6` jeetega |
| **Saare negative** | `[-1,-2,-3]` | `maxSum` `0` pe hi atka rahega, `minSum = -6` → answer `-(-6) = 6` |
| **Single element** | `[5]` ya `[-5]` | Turant `maxSum=5` ya `minSum=-5` ban jaata hai → answer `5` |
| **Saare zero** | `[0,0,0]` | `maxSum` aur `minSum` dono `0` pe hi rahenge → answer `0` (sahi bhi yahi hai) |

> ⚠️ **`0` se seed karna yahan bug nahi hai** — is problem mein `maxSum`/`minSum` dono ek saath combine ho rahe hain (`max(maxSum, -minSum)`), isliye "empty subarray" wala placeholder kabhi asli answer ko override nahi karta jab tak array mein koi non-zero element ho.

---

## 🔁 Quick Revision (30 second recap)

1. Answer do jagah se aa sakta hai: **best positive-sum subarray** (`maxSum`) ya **best negative-sum subarray ka negation** (`-minSum`).
2. Ek hi loop mein **max aur min dono** Kadane's chala lo, dono `0` se seed karke.
3. **Final: `max(maxSum, -minSum)`.**
4. Trap: abs **sum ke baad** lagana hai, har element pe abs lagake sum karna galat hai.

---
