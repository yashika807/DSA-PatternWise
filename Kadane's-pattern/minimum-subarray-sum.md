Interactive Visual: [minimum-subarray-sum.html]

(https://yashika807.github.io/DSA-PatternWise/Kadane's-pattern/minimum-subarray-sum.html)

# Smallest Sum Contiguous Subarray 📉

> **Pattern:** Kadane's Algorithm → *Kadane's ulta (min version)*
> **Difficulty:** 🟢 Easy-Medium
> **Problem:** [GFG — Smallest Sum Contiguous Subarray](https://www.geeksforgeeks.org/problems/smallest-sum-contiguous-subarray/1)

---

## 📌 Problem kya keh raha hai

Ek integer array `arr` diya hai. Iske andar se koi bhi **contiguous** (non-empty) subarray chuno jiska **sum sabse kam** ho, aur woh minimum sum return karo.

Yeh bilkul **normal Kadane's ka mirror image** hai — bas `max` ki jagah har jagah `min` use karna hai.

### Examples

| Input | Output | Kyun |
|---|---|---|
| `[3,-4,-5,8]` | `-9` | `[-4,-5]` — dono negatives ka contiguous run |
| `[1,2,3]` | `1` | saare positive → sabse chhota single element |
| `[-1,-2,-3]` | `-6` | saare negative → poora array hi minimum |

**Constraints (typical GFG range):** `1 ≤ n ≤ 10⁵`, `-10⁴ ≤ arr[i] ≤ 10⁴`

---

## ❌ Galat approaches (jo trap karte hain)

**1. "Bhool se normal (max) Kadane's copy-paste kar diya"**
Sabse common galti — Kadane's template yaad hota hai `max` ke saath, aur bas `min` mein badalna bhool jaate hain. `[3,-4,-5,8]` pe agar `max` Kadane's chala do, to answer `8` aayega (maximum subarray), jabki poochha gaya hai **minimum** — sahi answer `-9` hai. Har `max()` ko `min()` se replace karna zaroori hai, sirf ek jagah nahi, **dono jagah** (`curMin` aur `minSum` dono). ❌

**2. "Sirf global minimum single element dhoondo" (greedy)**
Lagta hai: array mein sabse chhota element hi answer hoga. `[3,-4,-5,8]` mein sabse chhota single element `-5` hai, par **contiguous** `[-4,-5]` ka sum `-9` hai jo usse bhi chhota hai! Ek se zyada negative numbers **saath mein** milkar aur chhota sum bana sakte hain — single element check kaafi nahi. ❌

**3. "curMin aur minSum ko 0 se seed karo"**
`nums[0]` ki jagah agar `0` se start karte ho, to all-positive array pe trap lagta hai. `[1,2,3]` mein seed `0` se: `curMin = min(1, 0+1) = 1`, par `minSum = min(0, 1) = 0` — jo **galat** hai, kyunki `0` sum wala koi valid non-empty subarray exist hi nahi karta! Sahi answer `1` hai (sabse chhota single element). Seeding hamesha `arr[0]` se karo. ❌

---

## 💡 Sahi intuition

Har index `i` pe wahi purana sawaal, bas ulta:

> **"Pichla running subarray extend karu (agar woh sum ko aur chhota banata hai), ya yahin se naya subarray shuru karu?"**

Agar pichla `curMin` **positive** hai, to usse aage carry karna sum ko badha dega — better hai wahi se fresh restart. Agar `curMin` **negative/zero** hai, to use extend karna hi sum ko aur neeche le jaata hai.

```
curMin = min(arr[i], curMin + arr[i])
```

Aur **har step** pe global minimum update karte raho:

```
minSum = min(minSum, curMin)
```

> 🔑 **Key insight:** Bilkul maximum-subarray wala hi logic hai — bas har `Math.max` ko `Math.min` se replace kar do. Dono seed bhi `arr[0]` se hote hain, `0` se nahi.

---

## 🧮 Dry Run — `[3, -4, -5, 8]`

Init: `curMin = 3`, `minSum = 3`

| i | num | curMin = min(num, curMin+num) | minSum |
|:-:|:---:|:-------------------------------:|:------:|
| 0 | 3 | init | **3** |
| 1 | -4 | min(-4, 3-4=-1) = **-4** | **-4** |
| 2 | -5 | min(-5, -4-5=-9) = **-9** | **-9** |
| 3 | 8 | min(8, -9+8=-1) = **-1** | -9 |

**Answer: `-9`** — subarray `[-4,-5]` (indices 1..2). ✅

---

## ☕ Java Solution

```java
class Solution {
    int minSubArraySum(int[] arr) {
        int n = arr.length;
        int curMin = arr[0];
        int minSum = arr[0];

        for (int i = 1; i < n; i++) {
            curMin = Math.min(arr[i], curMin + arr[i]);
            minSum = Math.min(minSum, curMin);
        }

        return minSum;
    }
}
```

## 🐍 Python Solution

```python
class Solution:
    def smallestSumSubarray(self, arr):
        cur_min = arr[0]
        min_sum = arr[0]

        for num in arr[1:]:
            cur_min = min(num, cur_min + num)
            min_sum = min(min_sum, cur_min)

        return min_sum
```

---

## ⏱ Complexity

| | |
|---|---|
| **Time** | `O(n)` — ek hi single pass |
| **Space** | `O(1)` — bas do variables, `curMin` aur `minSum` |

---

## 🧪 Edge cases (dhyaan se!)

| Case | Example | Kya hota hai |
|---|---|---|
| **Saare positive** | `[1,2,3]` | `curMin` kabhi extend se fayda nahi paata → sabse chhota single element jeetega → `1` |
| **Saare negative** | `[-1,-2,-3]` | Extend karna hamesha sum ko aur neeche le jaata hai → poora array hi answer → `-6` |
| **Single element** | `[5]` | Loop chalta hi nahi, seedha `arr[0] = 5` answer |
| **Mixed run** | `[3,-4,-5,8]` | Do negatives saath milke `-9` banate hain, single `-5` se bhi chhota |

> ⚠️ **Seeding trap:** `curMin` aur `minSum` dono **`arr[0]`** se seed karo, `0` se nahi — warna all-positive array pe `0` jaisa invalid (empty subarray) answer aa sakta hai.

---

## 🔁 Quick Revision (30 second recap)

1. Bilkul normal Kadane's jaisa, bas **har `max` → `min`**: `curMin = min(arr[i], curMin+arr[i])`.
2. Har step pe **`minSum = min(minSum, curMin)`** update karo.
3. Seed dono `arr[0]` se — `0` se seed karna all-positive arrays pe galat answer deta hai.
4. Single global-minimum-element check kaafi nahi — connected negative runs milke aur chhota sum bana sakte hain.

---
