Interactive Visual: [circular-array-loop.html]

(https://yashika807.github.io/DSA-PatternWise/Kadane's-pattern/circular-subarray.html)

# 918. Maximum Sum Circular Subarray 🔄

> **Pattern:** Kadane's Algorithm → *do Kadane's ek saath (max + min)*
> **Difficulty:** 🟡 Medium
> **Problem:** [LeetCode 918](https://leetcode.com/problems/maximum-sum-circular-subarray/)

---

## 📌 Problem kya keh raha hai

Ek **circular** integer array `nums` diya hai (length `n`). Non-empty subarray ka **maximum possible sum** return karo.

"Circular" ka matlab — array ka end wapas start se jud jaata hai. `nums[i]` ke baad `nums[(i+1) % n]` aata hai. To subarray **wrap around** bhi kar sakta hai (end se start tak), **lekin** koi element do baar use nahi hoga.

### Examples

| Input | Output | Kyun |
|---|---|---|
| `[1,-2,3,-2]` | `3` | seedha `[3]` — koi wrap nahi |
| `[5,-3,5]` | `10` | `[5, _, 5]` — wrap karke, beech ka `-3` chhoda |
| `[-3,-2,-3]` | `-2` | saare negative — sabse kam negative single element |

**Constraints:** `1 ≤ n ≤ 3·10⁴`, `-3·10⁴ ≤ nums[i] ≤ 3·10⁴`

---

## ❌ Galat approaches (jo trap karte hain)

**1. "Bas normal Kadane's chala do"**
Yeh sirf **no-wrap** case pakadta hai. `[5,-3,5]` pe yeh `7` dega (`[5,-3,5]`), par sahi answer `10` hai (wrap karke `5 + 5`). Wrap wala case chhoot gaya. ❌

**2. "Global minimum element hata do" (greedy)**
Man mein aata hai: *total sum le lo, aur sabse chhota single element minus kar do*. Par galat! Wrap case mein jo hissa **hataya** jaata hai woh ek **contiguous chunk** hota hai — koi bhi random element nahi. Subarray ki contiguity constraint isko todti hai. Greedy yahan fail. ❌

**3. "total − minSubarray, bina check ke"**
Yeh formula sahi hai *par* ek trap hai. Agar **poora array negative** ho (jaise `[-3,-2,-3]`), to `minSubarray` = poora array ban jaata hai, aur `total − min = 0` aa jaata hai — matlab **empty subarray**, jo allowed nahi! Isliye all-negative case alag se handle karna padta hai. ❌

---

## 💡 Sahi intuition — sirf 2 cases hain

Circular array mein best subarray ke **do hi** roop ho sakte hain:

**Case 1 — No wrap (seedha subarray)**
Array ke andar hi start aur end. Yeh milega normal **Kadane's max** se → `maxKadane`.

**Case 2 — Wrap (end se start tak)**
Agar subarray seam ke aar-paar wrap kar raha hai, to iska matlab **beech ka koi contiguous chunk chhoot raha hai**. Us chhode hue chunk ko total mein se nikaal do:

```
wrapSum = totalSum − minSubarraySum
```

Aur `minSubarraySum` bhi Kadane's se hi milta hai — bas `max` ki jagah `min` track karo. Isiliye hum **do Kadane's ek saath** chalate hain — ek `max` ke liye, ek `min` ke liye.

**Final decision:**

```
if (maxKadane < 0)                     // saare negative → wrap 0 dega (empty), skip
    answer = maxKadane
else
    answer = max(maxKadane, total − minKadane)
```

> 🔑 **Key insight:** `maxKadane < 0` ka matlab hai poora array negative hai. Sirf tabhi wrap ko ignore karo — warna dono cases ka max lo.

---

## 🧮 Dry Run — `[5, -3, 5]`

Init: `total=0`, `curMax=0`, `maxKadane=5`, `curMin=0`, `minKadane=5`

| i | num | total | curMax = max(curMax+num, num) | maxKadane | curMin = min(curMin+num, num) | minKadane |
|:-:|:---:|:-----:|:-----------------------------:|:---------:|:-----------------------------:|:---------:|
| 0 | 5 | `5` | max(0+5, 5) = **5** | **5** | min(0+5, 5) = **5** | **5** |
| 1 | -3 | `2` | max(5-3, -3) = **2** | 5 | min(5-3, -3) = **-3** | **-3** |
| 2 | 5 | `7` | max(2+5, 5) = **7** | **7** | min(-3+5, 5) = **2** | -3 |

**Decision:**
- `maxKadane = 7` (≥ 0, so wrap allowed)
- `wrap = total − minKadane = 7 − (-3) = 10`
- `answer = max(7, 10) = ` **`10`** ✅

Beech ka `-3` chunk hata do → jo wrap-around bacha (`5 + 5`) = **10**. 🎯

---

## ☕ Java Solution

```java
class Solution {
    public int maxSubarraySumCircular(int[] nums) {
        int totalSum = 0;
        int maxKadane = nums[0], curMax = 0;   // no-wrap max
        int minKadane = nums[0], curMin = 0;   // min subarray (for wrap)

        for (int num : nums) {
            totalSum += num;

            // Case 1: normal Kadane's — maximum subarray
            curMax = Math.max(curMax + num, num);
            maxKadane = Math.max(maxKadane, curMax);

            // Case 2: Kadane's variant — minimum subarray
            curMin = Math.min(curMin + num, num);
            minKadane = Math.min(minKadane, curMin);
        }

        // all-negative trap: wrap would give empty subarray (0)
        if (maxKadane < 0) return maxKadane;

        // otherwise best of no-wrap vs wrap
        return Math.max(maxKadane, totalSum - minKadane);
    }
}
```

## 🐍 Python Solution

```python
class Solution:
    def maxSubarraySumCircular(self, nums: list[int]) -> int:
        total = 0
        cur_max, max_k = 0, nums[0]   # no-wrap max
        cur_min, min_k = 0, nums[0]   # min subarray (for wrap)

        for num in nums:
            total += num
            cur_max = max(cur_max + num, num)
            max_k = max(max_k, cur_max)
            cur_min = min(cur_min + num, num)
            min_k = min(min_k, cur_min)

        # all-negative trap
        if max_k < 0:
            return max_k

        return max(max_k, total - min_k)
```

---

## ⏱ Complexity

| | |
|---|---|
| **Time** | `O(n)` — ek hi loop, dono Kadane's saath chal rahe |
| **Space** | `O(1)` — sirf kuch variables, koi extra array nahi |

---

## 🧪 Edge cases (dhyaan se!)

| Case | Example | Kya hota hai |
|---|---|---|
| **Saare negative** | `[-3,-2,-3]` | `maxKadane < 0` → seedha `maxKadane` return (`-2`). Wrap skip. |
| **Single element** | `[5]` | Loop ek baar → `maxKadane = 5`. Answer `5`. |
| **Koi wrap nahi** | `[1,-2,3,-2]` | `maxKadane` jeet jaata hai (`3`). |
| **Pura array best** | `[3,1,2]` | `wrap = total − min = 6 − 1 = 5`... but `maxKadane = 6` wins. Non-empty min chunk zaroori. |
| **Wrap wins** | `[5,-3,5]` | `wrap = 10 > maxKadane = 7`. |

> ⚠️ **Overflow note:** Constraints chhote hain (`|nums[i]| ≤ 3·10⁴`, `n ≤ 3·10⁴`), to `int` kaafi hai. Bade constraints hote to `long` chahiye hota.

---

## 🎮 Interactive Visualizer

Step-by-step dry run dekhne ke liye — do parallel Kadane's tracks + circular ring reveal:

👉 **[Dry Run Debugger kholo](https://htmlpreview.github.io/?https://github.com/yashika807/DSA-PatternWise/blob/main/Kadane%27s-pattern/lc918-circular-subarray.html)**

> *Note:* link `htmlpreview.github.io` ke through render hoga. Folder `Kadane's-pattern` mein apostrophe `%27` se encode kiya hai. Branch `main` na ho to link mein `main` → `master` kar dena.

---

## 🔁 Quick Revision (30 second recap)

1. **Do cases:** no-wrap (`maxKadane`) + wrap (`total − minKadane`).
2. Ek hi loop mein **max aur min dono** Kadane's chala lo.
3. **Trap:** `maxKadane < 0` → saare negative → sirf `maxKadane` return, warna wrap `0` (empty) de dega.
4. Wrap = "beech ka **contiguous** chunk hatao" — single element hatane wala greedy galat hai.

---
