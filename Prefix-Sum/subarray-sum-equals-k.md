Interactive Visual: [subarray-sum-equals-k.html]

(https://yashika807.github.io/DSA-PatternWise/Prefix-Sum/subarray-sum-equals-k.html)

# 560. Subarray Sum Equals K 💰

> **Pattern:** Prefix Sum + HashMap → *running balance yaad rakho*
> **Difficulty:** 🟡 Medium
> **Problem:** [LeetCode 560](https://leetcode.com/problems/subarray-sum-equals-k/)

**Difficulty:** Medium  ·  **Pattern:** Prefix Sum + HashMap  ·  **Tags:** Array, Hash Table, Prefix Sum

---

## Problem

Ek integer array `nums` aur ek integer `k` diya hai. Total **kitne continuous subarrays** hain jinka sum **exactly `k`** hai, yeh count karke return karo.

```
Input:  nums = [1,1,1], k = 2
Output: 2
Explanation: subarray [1,1] (index 0-1) aur [1,1] (index 1-2) — dono ka sum 2 hai.
```

Array mein **negative numbers bhi** aa sakte hain, isliye sliding window seedha kaam nahi karega (window shrink kab karein pata nahi chalega).

---

## The story (yaad rakhne ke liye) 🧠

Socho tum ek **daily bank ledger** maintain kar rahe ho. Har din ek transaction hota hai (`nums[i]`), aur `prefixSum` tumhara **running balance** hai — day 0 (shuru, koi transaction nahi) se lekar aaj tak ka total.

Ab tumhe puchna hai: *"kya kabhi aisa din tha jab mera balance bilkul `prefixSum - k` tha?"*

Kyun? Kyunki agar kisi purane din `j` pe balance `prefixSum[j]` tha, aur aaj (`i`) balance `prefixSum[i]` hai, to beech ke transactions ka sum:

```
prefixSum[i] - prefixSum[j] = subarray sum (j+1 ... i)
```

Hum chahte hain yeh `= k`. Toh equation ghumao:

```
prefixSum[j] = prefixSum[i] - k
```

Matlab: *"maine pehle kabhi is exact balance ko dekha hai kya?"* — yehi sawaal har din poochna hai. Iske liye ek **HashMap<balance, kitni baar yeh balance aaya>** rakho — ek chhota sa **ledger index** jo batata hai har balance value kitni baar dikha.

> ⚠️ **Din 0 ka balance `0` hota hai** (koi transaction se pehle) — isko map mein `{0: 1}` se initialize karna zaroori hai, warna wo subarrays miss ho jaayenge jo **array ki shuruaat se hi** sum `k` bana lete hain.

---

## Approach 1 — Brute force 🟥

Har starting point `i` se shuru karke, running sum badhate jao aur check karo kab `k` ban raha hai.

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int n = nums.length;
        int count = 0;

        for (int i = 0; i < n; i++) {
            int sum = 0;
            for (int j = i; j < n; j++) {
                sum += nums[j];
                if (sum == k) count++;
            }
        }

        return count;
    }
}
```

**Problem kya hai:** `O(n²)` time — `n = 2·10⁴` pe ~4·10⁸ operations, borderline/TLE. Har baar naya starting point, purani information reuse nahi ho rahi.

---

## Approach 2 — Optimal (Prefix Sum + HashMap) ✅

Ek hi pass mein running `prefixSum` maintain karo, aur har step pe ledger (`HashMap`) mein check karo ki `prefixSum - k` kitni baar dikh chuka hai.

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        Map<Integer, Integer> ledger = new HashMap<>();
        ledger.put(0, 1);          // day 0: balance 0, seen once (empty prefix)

        int prefixSum = 0, count = 0;

        for (int num : nums) {
            prefixSum += num;                       // aaj ka balance

            // kya balance "prefixSum - k" pehle kabhi dikha tha?
            count += ledger.getOrDefault(prefixSum - k, 0);

            // aaj ka balance ledger mein record karo
            ledger.merge(prefixSum, 1, Integer::sum);
        }

        return count;
    }
}
```

**Kyun kaam karta hai:** har baar jab `prefixSum - k` ledger mein milta hai, uska matlab hai utne hi purane "checkpoints" the jahan se yahan tak ka sum bilkul `k` tha. Unko count mein jod do — ek saath saare valid subarrays gin liye, bina inner loop ke.

---

## Dry run — `nums = [1,1,1], k = 2`

Init: `prefixSum = 0`, `ledger = {0: 1}`, `count = 0`

| i | num | prefixSum | need (`prefixSum - k`) | ledger.get(need) | count | ledger after |
|:-:|:---:|:---------:|:-----------------------:|:-----------------:|:-----:|:---|
| — | — | 0 | — | — | 0 | `{0:1}` |
| 0 | 1 | 1 | -1 | 0 | 0 | `{0:1, 1:1}` |
| 1 | 1 | 2 | 0 | 1 | **1** | `{0:1, 1:1, 2:1}` |
| 2 | 1 | 3 | 1 | 1 | **2** | `{0:1, 1:1, 2:1, 3:1}` |

**Result:** `count = 2` ✅ (subarray `nums[0..1]` aur `nums[1..2]`, dono sum = 2)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (nested loop, running sum) | `O(n²)` | `O(1)` | koi extra structure nahi, par slow |
| Prefix Sum + HashMap | `O(n)` | `O(n)` | ek pass, ledger mein max `n+1` distinct balances |

---

## Gotchas 🪤

- **`ledger.put(0, 1)` bhoolna sabse common mistake hai** — bina iske, `nums[0..i]` jaisa subarray (jo array ki shuruaat se hi `k` bana leta hai) count hi nahi hoga.
- **`getOrDefault` pehle, `merge`/`put` baad mein** — aaj ka balance record karne se *pehle* lookup karo, warna `prefixSum - k == prefixSum` (yani `k == 0`) waale case mein khud ko hi count kar loge, jo galat hai.
- **Negative numbers ki wajah se sliding window fail karta hai** — yahi is problem ki asli trick hai; isliye hashmap approach chahiye, do-pointer nahi.
- **Count karna hai, index nahi** — isliye map value `Integer` (frequency) hai, na ki last-seen index. `contiguous-array` jaisi problems mein iska ulta hota hai (wahan first index chahiye).
- **`k` khud negative ho sakta hai** — formula (`prefixSum - k`) waise hi kaam karta hai, koi special case nahi chahiye.
