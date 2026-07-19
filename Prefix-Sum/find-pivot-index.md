Interactive Visual: [find-pivot-index.html]

(https://yashika807.github.io/DSA-PatternWise/Prefix-Sum/find-pivot-index.html)

# 724. Find Pivot Index ⚖️

> **Pattern:** Prefix Sum → *running total, koi hashmap nahi chahiye*
> **Difficulty:** 🟢 Easy
> **Problem:** [LeetCode 724](https://leetcode.com/problems/find-pivot-index/)

**Difficulty:** Easy  ·  **Pattern:** Prefix Sum  ·  **Tags:** Array, Prefix Sum

---

## Problem

Ek integer array `nums` diya hai. **Pivot index** woh index hai jahan **left side ke saare elements ka sum** aur **right side ke saare elements ka sum** bilkul **equal** ho. Pivot index khud kisi bhi side ke sum mein count nahi hota.

Sabse **chhota** (leftmost) aisa index return karo. Agar koi pivot nahi milta, `-1` return karo.

```
Input:  nums = [1, 7, 3, 6, 5, 6]
Output: 3
Explanation: index 3 (value 6) pe — left sum = 1+7+3 = 11, right sum = 5+6 = 11. Balance! ⚖️
```

---

## The story (yaad rakhne ke liye) 🧠

Socho `nums` ek **seesaw (see-saw / weighing rod)** hai — har element ek weight hai jo rod pe rakha hai. Tumhe woh **fulcrum point** dhoondhna hai jahan rod pe rakhi ye saari weights **perfectly balance** ho jaayein — jo weight fulcrum pe khud baithi hai, woh kisi side mein count nahi hoti.

Yeh bhi ek **prefix sum problem hai**, lekin apne cousin `subarray-sum-equals-k` se ek badi baat mein alag: **yahan HashMap ki zaroorat nahi padti.**

Kyun? Kyunki hum kabhi *purane* balance ko dobara dhoondhte nahi — hum bas **is waqt** ka comparison kar rahe hain:

```
leftSum  = prefixSum jo pivot se pehle jama hua (running total)
rightSum = totalSum - leftSum - nums[pivot]   (jo bacha, pivot value minus karke)
```

Har index pe seedha check karo `leftSum == rightSum`. Koi memory/history maintain karne ki zaroorat nahi — bas ek **running total** (`leftSum`) chalao aur poora array ka `totalSum` pehle se pata ho.

> 🔑 **Key insight:** Prefix sum family mein sabhi problems ko HashMap nahi chahiye hota. Jab tumhe sirf *"abhi ka split kaisa dikhta hai"* jaanna ho (na ki *"pehle kabhi yeh balance dekha tha kya"*), ek simple running total kaafi hai.

---

## Approach 1 — Brute force 🟥

Har index `i` ke liye, left aur right dono taraf ke elements dobara sum karo.

```java
class Solution {
    public int pivotIndex(int[] nums) {
        int n = nums.length;

        for (int i = 0; i < n; i++) {
            int leftSum = 0, rightSum = 0;
            for (int j = 0; j < i; j++) leftSum += nums[j];
            for (int j = i + 1; j < n; j++) rightSum += nums[j];

            if (leftSum == rightSum) return i;
        }

        return -1;
    }
}
```

**Problem kya hai:** har `i` pe poora left aur right dobara sum karna — `O(n²)`. Wahi sums baar-baar recompute ho rahe hain.

---

## Approach 2 — Optimal (Prefix Sum, running total) ✅

Poore array ka `totalSum` ek baar nikaal lo. Phir ek hi pass mein `leftSum` running rakho — `rightSum` seedha formula se milta hai, kisi loop ki zaroorat nahi.

```java
class Solution {
    public int pivotIndex(int[] nums) {
        int totalSum = 0;
        for (int num : nums) totalSum += num;

        int leftSum = 0;

        for (int i = 0; i < nums.length; i++) {
            // baaki bacha total, pivot ki apni value minus karke, wahi right sum hai
            int rightSum = totalSum - leftSum - nums[i];

            if (leftSum == rightSum) return i;

            leftSum += nums[i];   // pivot cross karke aage badho
        }

        return -1;
    }
}
```

**Kyun kaam karta hai:** `leftSum` hamesha `nums[0..i-1]` ka sum hota hai (prefix, index `i` exclude). `totalSum - leftSum - nums[i]` automatically `nums[i+1..n-1]` ka sum ban jaata hai — koi extra loop nahi, bas arithmetic.

---

## Dry run — `nums = [1, 7, 3, 6, 5, 6]`

`totalSum = 1+7+3+6+5+6 = 28`, init `leftSum = 0`

| i | nums[i] | leftSum | rightSum = total − leftSum − nums[i] | leftSum == rightSum? | next leftSum |
|:-:|:---:|:---:|:---:|:---:|:---:|
| 0 | 1 | 0 | 28−0−1 = 27 | 0 ≠ 27 | 1 |
| 1 | 7 | 1 | 28−1−7 = 20 | 1 ≠ 20 | 8 |
| 2 | 3 | 8 | 28−8−3 = 17 | 8 ≠ 17 | 11 |
| 3 | 6 | 11 | 28−11−6 = **11** | **11 == 11 ✅** | — |

**Result:** pivot index = `3` ✅ (return immediately, no need to keep scanning)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (recompute left/right each i) | `O(n²)` | `O(1)` | wasteful re-summing |
| Prefix Sum (running total) | `O(n)` | `O(1)` | one pass for total, one pass for scan |

---

## Gotchas 🪤

- **Pivot khud kisi side mein count nahi hota** — `rightSum` formula mein `- nums[i]` isiliye hai; bhoolo mat.
- **Leftmost pivot chahiye** — jaise hi `leftSum == rightSum` mile, turant `return i`. Aage scan mat karo, warna baad ka koi pivot (agar ho) galti se pehle chun loge.
- **`leftSum` ko update karo comparison ke *baad*** — pehle check karo `leftSum == rightSum` (jo `nums[0..i-1]` vs `nums[i+1..n-1]` compare karta hai), *phir* `leftSum += nums[i]` karo agle index ke liye. Order ulta karoge to galat comparison hoga.
- **Single-element array** — `nums = [5]` pe `leftSum=0`, `rightSum = 5-0-5 = 0` → turant match, index `0` return hota hai. Edge case apne aap handle ho jaata hai.
- **Yahan HashMap nahi chahiye** — is family ki baaki problems (`subarray-sum-equals-k` waghera) mein "pehle yeh balance dekha tha?" poochna padta hai, isliye HashMap lagta hai. Yahan sirf **is waqt ka split** dekhna hai, history ki zaroorat nahi.
