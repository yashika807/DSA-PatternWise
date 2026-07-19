Interactive Visual: [subarray-sums-divisible-by-k.html]

(https://yashika807.github.io/DSA-PatternWise/Prefix-Sum/subarray-sums-divisible-by-k.html)

# 974. Subarray Sums Divisible by K ⏰

> **Pattern:** Prefix Sum + HashMap (remainders) → *clock dial yaad rakho*
> **Difficulty:** 🟡 Medium
> **Problem:** [LeetCode 974](https://leetcode.com/problems/subarray-sums-divisible-by-k/)

**Difficulty:** Medium  ·  **Pattern:** Prefix Sum + HashMap  ·  **Tags:** Array, Hash Table, Math, Prefix Sum

---

## Problem

Ek integer array `nums` aur ek integer `k` diya hai. Total **kitne (non-empty) continuous subarrays** hain jinka sum **`k` se divisible** hai (yani `sum % k == 0`), yeh count karke return karo.

```
Input:  nums = [4,5,0,-2,-3,1], k = 5
Output: 7
Explanation: subarrays [4,5,0,-2,-3,1], [5], [5,0], [5,0,-2,-3], [0], [0,-2,-3], [-2,-3]
             — in saaton ka sum 5 se divisible hai.
```

Array mein **negative numbers bhi** aa sakte hain — isi wajah se `%` operator ke saath careful rehna padega.

---

## The story (yaad rakhne ke liye) 🧠

Socho tumhare paas ek **`k`-ghante wala circular clock** hai — normal clock mein 12 slots hote hain, is clock mein sirf `k` slots hain: `0, 1, 2, ..., k-1`. Har din ka running balance (`prefixSum`) is clock pe kahin na kahin **land** karta hai — uske landing slot ka number hai `prefixSum % k`.

Ab yaad karo `subarray-sum-equals-k` wala trick:

```
prefixSum[i] - prefixSum[j] = subarray sum (j+1 ... i)
```

Yeh subarray `k` se divisible tabhi hoga jab `(prefixSum[i] - prefixSum[j])` khud `k` se divisible ho — matlab **dono prefix sums clock ke same slot pe land kiye ho**:

```
prefixSum[i] % k == prefixSum[j] % k
```

Toh sawaal wahi purana hai: *"is clock-slot pe pehle bhi kabhi koi balance land hua tha?"* — usi **ledger (HashMap<slot, kitni baar>)** se poochna hai, bas key ab exact balance nahi, uska **clock slot (remainder)** hai.

> ⚠️ **Java ka `%` operator negative numbers ke liye negative result de sakta hai!** `-7 % 5` Java mein `-2` deta hai, na ki `3` (jo mathematically sahi remainder hai). Isliye normalize karna zaroori hai: `((prefixSum % k) + k) % k` — tabhi slot hamesha `[0, k-1]` range mein rahega.

> 🔑 **Din 0 ka slot bhi `0` hota hai** (koi transaction se pehle, balance 0, jo `0 % k = 0` slot pe land karta hai) — `ledger.put(0, 1)` se initialize karo, warna array ki shuruaat se hi divisible subarrays miss ho jaayenge.

---

## Approach 1 — Brute force 🟥

Har starting point `i` se shuru karke, running sum badhate jao aur check karo kab woh `k` se divisible ban raha hai.

```java
class Solution {
    public int subarraysDivByK(int[] nums, int k) {
        int n = nums.length;
        int count = 0;

        for (int i = 0; i < n; i++) {
            int sum = 0;
            for (int j = i; j < n; j++) {
                sum += nums[j];
                if (sum % k == 0) count++;   // "== 0" check safe hai, sign issue nahi
            }
        }

        return count;
    }
}
```

**Problem kya hai:** `O(n²)` time — `n = 3·10⁴` pe ~9·10⁸ operations, TLE ho jaayega. Har starting point purani information reuse nahi karta.

---

## Approach 2 — Optimal (Prefix Sum + HashMap of remainders) ✅

Ek hi pass mein running `prefixSum` maintain karo, uska **normalized remainder** nikaalo, aur ledger mein check karo ki yeh remainder pehle kitni baar dikh chuka hai.

```java
class Solution {
    public int subarraysDivByK(int[] nums, int k) {
        Map<Integer, Integer> ledger = new HashMap<>();
        ledger.put(0, 1);           // day 0: remainder 0, seen once (empty prefix)

        int prefixSum = 0, count = 0;

        for (int num : nums) {
            prefixSum += num;                          // aaj ka balance

            // Java's % negative de sakta hai (e.g. -7 % 5 == -2) — normalize to [0, k-1]
            int mod = ((prefixSum % k) + k) % k;

            // yeh clock-slot pehle kitni baar dikha tha?
            count += ledger.getOrDefault(mod, 0);

            // aaj ka slot ledger mein record karo
            ledger.merge(mod, 1, Integer::sum);
        }

        return count;
    }
}
```

**Kyun kaam karta hai:** har baar jab aaj ka `mod` ledger mein `c` baar mil jaata hai, uska matlab hai `c` purane checkpoints the jinka remainder bilkul same tha — un sab checkpoints se leke aaj tak ka subarray `k` se divisible hai. Sab ek saath count mein jod do.

---

## Dry run — `nums = [4,5,0,-2,-3,1], k = 5`

Init: `prefixSum = 0`, `ledger = {0: 1}`, `count = 0`

| i | num | prefixSum | mod (normalized) | ledger.get(mod) | count | ledger after |
|:-:|:---:|:---------:|:-----------------:|:-----------------:|:-----:|:---|
| — | — | 0 | — | — | 0 | `{0:1}` |
| 0 | 4 | 4 | 4 | 0 | 0 | `{0:1, 4:1}` |
| 1 | 5 | 9 | 4 | 1 | **1** | `{0:1, 4:2}` |
| 2 | 0 | 9 | 4 | 2 | **3** | `{0:1, 4:3}` |
| 3 | -2 | 7 | 2 | 0 | 3 | `{0:1, 4:3, 2:1}` |
| 4 | -3 | 4 | 4 | 3 | **6** | `{0:1, 4:4, 2:1}` |
| 5 | 1 | 5 | 0 | 1 | **7** | `{0:2, 4:4, 2:1}` |

**Result:** `count = 7` ✅ — matches the expected output exactly.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (nested loop, running sum) | `O(n²)` | `O(1)` | koi extra structure nahi, par slow |
| Prefix Sum + HashMap (remainders) | `O(n)` | `O(min(n, k))` | ledger mein sirf `k` distinct remainders ho sakte hain, unlike `subarray-sum-equals-k` jahan `n+1` distinct sums ho sakte the |

---

## Gotchas 🪤

- **Java ka `%` normalize karna sabse zaroori step hai** — `((prefixSum % k) + k) % k` na likha to negative `prefixSum` waale test cases silently galat answer denge (negative remainder ledger mein alag key ban jaayegi).
- **`ledger.put(0, 1)` bhoolna** — bina iske, array ki shuruaat se hi `k`-divisible subarrays count nahi honge (same mistake jo `subarray-sum-equals-k` mein hoti hai).
- **`getOrDefault` pehle, `merge` baad mein** — order ulta karoge to aaj ka khud ka remainder galti se count ho jaayega.
- **Yahan "need" seedha `mod` hi hai, `mod - k` nahi** — `subarray-sum-equals-k` se yeh sabse bada farak hai: wahan hum ek shifted value dhoondhte the (`prefixSum - k`), yahan hum sirf **same remainder** dhoondh rahe hain, koi shift nahi chahiye.
- **`k` ki constraint `2 <= k <= 10⁴`** — matlab `k` khud kabhi negative ya zero nahi hoga, sirf `prefixSum` negative ho sakta hai. Isliye normalization sirf `prefixSum % k` ke result pe lagti hai, `k` pe nahi.
