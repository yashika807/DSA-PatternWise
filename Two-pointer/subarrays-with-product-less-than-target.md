# Subarray Product Less Than K — LeetCode 713

> 🔗 **[Interactive Visualizer dekho »]
> (https://yashika807.github.io/DSA-PatternWise/Two-pointer/subarrays-with-product-less-than-target.html)**
> _(step-through Register Start + Scanner sliding window, live running-product readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Two Pointers (sliding window)  ·  **Tags:** Array, Two Pointers, Sliding Window

---

## Problem

Ek array `nums` (saare **positive** integers) aur ek integer `k` diya hai. Count karo **kitne contiguous subarrays** hain jinka **product `k` se strictly kam** ho.

```
Input:  nums = [10, 5, 2, 6], k = 100
Output: 8
// [10], [5], [2], [6], [10,5], [5,2], [2,6], [5,2,6]  (product < 100 for each)
```

---

## The story (yaad rakhne ke liye) 🛒

Ek **billing counter par sliding window** socho — ek **Register Start** (`left`) jahan se current basket shuru hota hai, aur ek **Scanner** (`right`) jo naye items scan karta jaata hai basket mein daalte hue. Basket ka **running product** (total bill, but multiply karke) track hota hai.

| Role | Kaun | Kaam |
|---|---|---|
| **Register Start** (`left`) | basket ka left edge | "yahan se basket shuru hota hai" |
| **Scanner** (`right`) | naya item add karta hai | "yeh item bhi basket mein daalo" |

Har naye item pe (`right` aage badhne pe):

1. `product *= nums[right]` — naya item basket mein daala.
2. Jab tak `product >= k` (**budget cross ho gaya**), Register Start ko aage khiskao (`product /= nums[left]; left++`) — jab tak basket **wapas budget ke andar** na aa jaaye.
3. Ab window `[left, right]` **valid** hai (product `< k`). Is window ke andar **`right` pe end hone waale saare subarrays** valid hain — unki count hai `right - left + 1` (kyunki `[left, right]`, `[left+1, right]`, ..., `[right, right]` sab ka product `nums[right..end]` se chhota ya usi range mein hoga, sab automatically `< k`).

> 🔑 Yeh trick 3Sum-style two-pointer se thoda alag hai — yahan **dono pointers same direction** mein chalte hain (sliding window), opposite ends se nahi milte.

---

## Approach 1 — Brute force 🟥

Har subarray ka product nikaal ke check karo.

```java
class Solution {
    public int numSubarrayProductLessThanK(int[] nums, int k) {
        int n = nums.length;
        int count = 0;

        for (int i = 0; i < n; i++) {
            long product = 1;
            for (int j = i; j < n; j++) {
                product *= nums[j];
                if (product < k) count++;
                else break;   // further extension only makes product bigger (all positive)
            }
        }

        return count;
    }
}
```

**Problem kya hai:** `O(n²)` worst case — chalta hai, but sliding window se `O(n)` mein hi ho sakta hai, kyunki humein poore subarray dobara traverse karne ki zaroorat nahi.

---

## Approach 2 — Sliding Window / Two Pointers (Optimal) ✅

```java
class Solution {
    public int numSubarrayProductLessThanK(int[] nums, int k) {
        if (k <= 1) return 0;    // all nums >= 1, so no product can be < 1

        int left = 0;
        long product = 1;
        int count = 0;

        for (int right = 0; right < nums.length; right++) {
            product *= nums[right];             // Scanner adds new item

            // shrink window while over budget
            while (product >= k) {
                product /= nums[left];
                left++;
            }

            // every subarray ending at `right`, starting from left..right, is valid
            count += right - left + 1;
        }

        return count;
    }
}
```

### Kyun `right - left + 1` sahi hai

- Window `[left, right]` valid hai (`product < k`).
- Isi window ke andar `right` pe end hone waale sabhi subarrays — `[left..right]`, `[left+1..right]`, ..., `[right..right]` — bhi valid hain, kyunki chhota subarray ka product bade window ke product se **kam ya barabar** hoga (saare elements positive hain).
- Aise subarrays ki count = `right - left + 1`.

---

## Dry run — `[10, 5, 2, 6]`, `k = 100`

| right | nums[right] | product (before shrink) | shrink? | left (after) | product (final) | count += | count (total) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 0 | 10 | 10 | no | 0 | 10 | 1 | 1 |
| 1 | 5 | 50 | no | 0 | 50 | 2 | 3 |
| 2 | 2 | 100 | yes → ÷10 | 1 | 10 | 2 | 5 |
| 3 | 6 | 60 | no | 1 | 60 | 3 | 8 |

**Result:** `count = 8` ✅

> Row 2 detail: `product` `100 >= k(100)` hai, isliye shrink: `product /= nums[0]=10 → 10`, `left=1`. Ab `product(10) < 100`, shrink ruk gaya. Window `[1,2]` = `[5,2]`, size `2`, `count += 2`.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(n²)` worst | `O(1)` | recomputes product per start index |
| Sliding Window | `O(n)` | `O(1)` | each element enters/leaves window at most once |

---

## Gotchas 🪤

- **`k <= 1` edge case** — saare `nums[i] >= 1` hone ki wajah se koi bhi non-empty subarray ka product `< 1` nahi ho sakta. Yeh check na karo to shrink loop infinite ya galat ho sakta hai.
- **`product` ko `long` rakho** — `nums[i] <= 1000` aur `n` bada ho sakta hai, product jaldi `int` overflow kar sakta hai.
- **Saare elements positive hone chahiye** — is technique ka core assumption hai ki naya element add karne se product **hamesha badhta hai** (monotonic). Agar `0` ya negative allowed hote, sliding window ka logic tootc jaata — division bhi undefined ho sakta (`0` se divide).
- **`while`, na ki `if`, shrink ke liye** — kabhi-kabhi ek hi step mein multiple shrink chahiye hote hain jab tak product wapas budget mein na aaye.
- **Count `right - left + 1` window ke turant baad**, shrink loop se pehle nahi — pehle ensure karo window valid hai, phir count karo.
