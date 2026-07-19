Interactive Visual: [maximum-product-subarray.html]

(https://yashika807.github.io/DSA-PatternWise/Kadane's-pattern/maximum-product-subarray.html)

# 152. Maximum Product Subarray ✖️

> **Pattern:** Kadane's Algorithm → *do Kadane's ek saath (max + min, negative flip ke liye)*
> **Difficulty:** 🟡 Medium
> **Problem:** [LeetCode 152](https://leetcode.com/problems/maximum-product-subarray/)

---

## 📌 Problem kya keh raha hai

Ek integer array `nums` diya hai. Iske andar se koi bhi **contiguous** (non-empty) subarray chuno jiska **product sabse zyada** ho, aur woh maximum product return karo.

Sum wale Kadane's se yeh alag isliye hai kyunki **do negative numbers milkar positive ban jaate hain** — isliye sirf running maximum track karna kaafi nahi, running **minimum** bhi track karna padta hai (kyunki woh minimum, ek naye negative se multiply hoke sabse bada positive ban sakta hai).

### Examples

| Input | Output | Kyun |
|---|---|---|
| `[2,3,-2,4]` | `6` | `[2,3]` — `-2` aane se pehle hi best mil gaya |
| `[-2,3,-4]` | `24` | poora array — do negatives milkar positive `24` bana dete hain |
| `[-2,0,-1]` | `0` | zero beech mein hai, koi bhi negative-run usse bada nahi |

**Constraints:** `1 ≤ nums.length ≤ 2·10⁴`, `-10 ≤ nums[i] ≤ 10`

---

## ❌ Galat approaches (jo trap karte hain)

**1. "Sirf ek running max track karo, min ki zaroorat nahi"**
Sum wale Kadane's ki tarah lagta hai bas `curMax = max(num, curMax*num)` kaafi hoga. Par **galat**! `[-2,3,-4]` pe try karo: `curMax` sequence → `-2, max(3,-6)=3, max(-4,3*-4=-12)=-4` → result sirf `3` milega. Par sahi answer `24` hai (`-2 * 3 * -4`)! Jo `curMin` bana tha beech mein (`-6` at i=1), woh agle negative number se multiply hoke sabse bada `24` ban gaya — jo hum track hi nahi kar rahe the. ❌

**2. "Poora array ka product hi le lo (subarray consider hi mat karo)"**
Lagta hai poora array multiply karke dekh lo. `[2,-5,-2,-4,3]` ka poora product = `-240` (5 negatives mein se 3 negative hai — odd, so negative). Par sahi answer `24` hai, jo `[-2,-4,3]` subarray se aata hai. Poore array ka sign consider karna kaafi nahi — **contiguous subarray boundary** dhoondni padti hai. ❌

**3. "curMax update karne ke turant baad, usi NAYE curMax ko curMin nikaalne mein use kar liya"**
Yeh subtle bug hai — snapshot lena bhool gaye. Sahi tareeka: `curMax` aur `curMin` dono **purani (is-step-se-pehle wali)** values se ek saath compute hone chahiye — ya to swap trick se, ya dono ek saath ek temp variable mein store karke. Agar `minProd` ki calculation mein galti se **already-updated** `maxProd` use ho jaaye, to `minProd` corrupt ho jaata hai, aur woh corruption agle step mein galat (fake) bada answer bana sakta hai. Bilkul waise hi jaise "Maximum Subarray Sum with One Deletion" mein `prevNoDel` snapshot lena zaroori tha — yahan bhi purani value save karna must hai. ❌

---

## 💡 Sahi intuition — swap trick se do Kadane's

Har index `i` pe do quantities track karo — **maximum product ending at `i`** aur **minimum product ending at `i`**:

- `maxProd` — kaam ka isliye kyunki agla positive number isse aur bada bana sakta hai.
- `minProd` — kaam ka isliye kyunki agla **negative** number isse (bahut chhota/negative value) sabse **bada positive** bana sakta hai!

**Trick:** jab current number `num` **negative** ho, to multiply karne se `maxProd` aur `minProd` ka role **flip** ho jaata hai — jo pehle max tha woh ab min ban jaayega aur vice versa. Isliye pehle **swap** kar do, phir normal max/min formula lagao:

```
if (num < 0) swap(maxProd, minProd)

maxProd = max(num, maxProd * num)
minProd = min(num, minProd * num)

result = max(result, maxProd)
```

> 🔑 **Key insight:** Sum wale Kadane's mein sirf "extend ya restart" decide karna hota hai. Product wale Kadane's mein ek **teesra factor** hai — **sign flip** — isliye do parallel states (max + min) chahiye, bilkul jaise circular-subarray mein do Kadane's (max + min) chalte hain, bas yahan sign-flip ke liye.

---

## 🧮 Dry Run — `[2, 3, -2, 4]`

Init: `maxProd = minProd = result = 2`

| i | num | swap? | maxProd = max(num, maxProd·num) | minProd = min(num, minProd·num) | result |
|:-:|:---:|:---:|:---:|:---:|:---:|
| 0 | 2 | — | init | init | **2** |
| 1 | 3 | no | max(3, 2·3=6) = **6** | min(3, 2·3=6) = **3** | **6** |
| 2 | -2 | **yes** (max↔min: 3,6) | max(-2, 3·-2=-6) = **-2** | min(-2, 6·-2=-12) = **-12** | 6 |
| 3 | 4 | no | max(4, -2·4=-8) = **4** | min(4, -12·4=-48) = **-48** | 6 |

**Answer: `6`** — subarray `[2,3]` (indices 0..1). ✅

---

## ☕ Java Solution

```java
class Solution {
    public int maxProduct(int[] nums) {
        int maxProd = nums[0];
        int minProd = nums[0];
        int result = nums[0];

        for (int i = 1; i < nums.length; i++) {
            int num = nums[i];

            if (num < 0) {
                int temp = maxProd;
                maxProd = minProd;
                minProd = temp;
            }

            maxProd = Math.max(num, maxProd * num);
            minProd = Math.min(num, minProd * num);

            result = Math.max(result, maxProd);
        }

        return result;
    }
}
```

## 🐍 Python Solution

```python
class Solution:
    def maxProduct(self, nums: list[int]) -> int:
        max_prod = min_prod = result = nums[0]

        for num in nums[1:]:
            if num < 0:
                max_prod, min_prod = min_prod, max_prod

            max_prod = max(num, max_prod * num)
            min_prod = min(num, min_prod * num)

            result = max(result, max_prod)

        return result
```

---

## ⏱ Complexity

| | |
|---|---|
| **Time** | `O(n)` — ek hi single pass |
| **Space** | `O(1)` — sirf `maxProd`, `minProd`, `result` |

---

## 🧪 Edge cases (dhyaan se!)

| Case | Example | Kya hota hai |
|---|---|---|
| **Zero beech mein** | `[-2,0,-1]` | Zero se multiply hote hi `maxProd`/`minProd` dono `0` pe reset ho jaate hain (`0` khud best candidate ban jaata hai us index pe) → answer `0` |
| **Saare negative, even count** | `[-1,-2,-3,-4]` | Poora array multiply karne se do negative-pairs cancel ho jaate hain → `24` |
| **Saare negative, odd count** | `[-1,-2,-3]` | Ek negative ko chhodna padta hai; swap trick khud-ba-khud best 2-length subarray dhoondh leta hai → `6` |
| **Single element** | `[-5]` | Loop chalta hi nahi, seedha `nums[0] = -5` answer |

> ⚠️ **Zero ka special-case likhne ki zaroorat nahi:** formula khud handle kar leta hai — `max(0, anything*0=0)` hamesha `0` deta hai, jo agla fresh restart point ban jaata hai. Alag `if (num==0)` branch likhne ki zaroorat nahi.

---

## 🔁 Quick Revision (30 second recap)

1. Sirf `max` track karna kaafi nahi — **`min` bhi track karo**, kyunki negative × negative = positive.
2. Jab `num < 0` ho, **`maxProd` aur `minProd` ko swap** kar do multiply karne se pehle — phir wahi max/min formula chalti hai.
3. Dono updates **ek hi purani (is-step-se-pehle) state** se hone chahiye — order-of-update bug se bacho.
4. Zero apne aap reset kar deta hai, alag se handle karne ki zaroorat nahi.

---
