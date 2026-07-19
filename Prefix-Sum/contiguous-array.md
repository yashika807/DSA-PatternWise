Interactive Visual: [contiguous-array.html]

(https://yashika807.github.io/DSA-PatternWise/Prefix-Sum/contiguous-array.html)

# 525. Contiguous Array 📸

> **Pattern:** Prefix Sum + HashMap (first occurrence) → *purani photo yaad rakho*
> **Difficulty:** 🟡 Medium
> **Problem:** [LeetCode 525](https://leetcode.com/problems/contiguous-array/)

**Difficulty:** Medium  ·  **Pattern:** Prefix Sum + HashMap  ·  **Tags:** Array, Hash Table, Prefix Sum

---

## Problem

Ek **binary array** `nums` diya hai (sirf `0` aur `1`). **Sabse lamba (longest) contiguous subarray** dhoondo jisme `0` aur `1` ki **ginti bilkul barabar** ho, aur uski **length** return karo.

```
Input:  nums = [0,1,0,0,1,1,0]
Output: 6
Explanation: subarray [0,1,0,0,1,1] (index 0 se 5 tak) mein 3 zeros aur 3 ones hain — length 6,
             aur yeh sabse lamba aisa subarray hai. (Note: [1,0,0,1,1,0], index 1-6, bhi ek
             valid length-6 answer hota — dono ka sum zero hai, LeetCode sirf length maangta hai.)
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh bhi wahi **prefix sum + ledger** family hai, lekin ek chhota sa **trick** ke saath: `0` aur `1` ko seedha count karne ke bajaye, `0` ko **`-1`** treat karo aur `1` ko **`+1`**. Ab sawaal ban jaata hai: *"sabse lamba subarray dhoondo jiska sum bilkul `0` ho"* — bilkul wahi shape jo humne `subarray-sum-equals-k` mein dekhi thi (bas `k = 0`)!

Lekin yahan hume **count nahi, length** chahiye. Isliye ledger ka role badal jaata hai:

> Har balance ki **sirf pehli photo (first index)** rakho — jab wahi balance dobara mile, purani photo se distance naapo. Purani photo se lekar aaj tak — beech ka sara `+1`/`-1` cancel ho gaya, matlab utne hi zeros aur ones the.

```
balance[i] == balance[j]  (j < i)
→  subarray (j+1 ... i) ka sum 0 hai
→  usme equal zeros aur ones hain
→  length = i - j
```

Sabse lambi length chahiye, isliye **sabse purani** photo (smallest `j`) use karni hai — isiliye **overwrite mat karo**, balance pehli baar dikhe tabhi map mein daalo.

> ⚠️ **`subarray-sum-equals-k` se sabse bada farak yehi hai:** wahan HashMap **count** store karta tha (`balance → kitni baar dikha`), yahan HashMap **sirf first index** store karta hai (`balance → pehli baar kahan dikha`). Dono prefix-sum family ke hain, par kya store karna hai woh problem ki demand pe depend karta hai.

---

## Approach 1 — Brute force 🟥

Har starting point `i` se shuru karke, zeros aur ones ka count badhate jao aur jab dono barabar ho, length update karo.

```java
class Solution {
    public int findMaxLength(int[] nums) {
        int n = nums.length;
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            int zeros = 0, ones = 0;
            for (int j = i; j < n; j++) {
                if (nums[j] == 0) zeros++; else ones++;
                if (zeros == ones) maxLen = Math.max(maxLen, j - i + 1);
            }
        }

        return maxLen;
    }
}
```

**Problem kya hai:** `O(n²)` time — `n = 10⁵` pe TLE pakka. Har starting point purana kaam bhool ke naya count shuru karta hai.

---

## Approach 2 — Optimal (Prefix Sum + HashMap of first occurrence) ✅

`0 → -1` convert karke ek running `balance` maintain karo. Har balance ki **pehli** occurrence yaad rakho — dobara wahi balance mile to distance nikaal ke `maxLen` update karo.

```java
class Solution {
    public int findMaxLength(int[] nums) {
        Map<Integer, Integer> firstSeen = new HashMap<>();
        firstSeen.put(0, -1);      // balance 0, "before index 0" (empty prefix)

        int balance = 0, maxLen = 0;

        for (int i = 0; i < nums.length; i++) {
            balance += (nums[i] == 1) ? 1 : -1;   // 1 → +1, 0 → -1

            if (firstSeen.containsKey(balance)) {
                // balance dobara dikha — beech ka subarray sum 0 hai
                maxLen = Math.max(maxLen, i - firstSeen.get(balance));
            } else {
                // pehli baar dikha — is index ko photo ki tarah save karo
                firstSeen.put(balance, i);
            }
        }

        return maxLen;
    }
}
```

**Kyun kaam karta hai:** `firstSeen.get(balance)` hamesha **sabse purani** index deta hai jahan yeh balance tha (kyunki hum sirf first occurrence hi store karte hain, overwrite kabhi nahi karte). Sabse purani index se distance naapna hamesha **sabse lambi** valid length degi.

---

## Dry run — `nums = [0,1,0,0,1,1,0]`

Init: `balance = 0`, `firstSeen = {0: -1}`, `maxLen = 0`

| i | nums[i] | ±1 | balance | firstSeen has it? | maxLen calc | firstSeen after |
|:-:|:---:|:---:|:---:|:---:|:---|:---|
| 0 | 0 | -1 | -1 | No | — | `{0:-1, -1:0}` |
| 1 | 1 | +1 | 0 | Yes (at -1) | `max(0, 1-(-1)) = 2` | `{0:-1, -1:0}` |
| 2 | 0 | -1 | -1 | Yes (at 0) | `max(2, 2-0) = 2` | `{0:-1, -1:0}` |
| 3 | 0 | -1 | -2 | No | — | `{0:-1, -1:0, -2:3}` |
| 4 | 1 | +1 | -1 | Yes (at 0) | `max(2, 4-0) = 4` | `{0:-1, -1:0, -2:3}` |
| 5 | 1 | +1 | 0 | Yes (at -1) | `max(4, 5-(-1)) = 6` | `{0:-1, -1:0, -2:3}` |
| 6 | 0 | -1 | -1 | Yes (at 0) | `max(6, 6-0) = 6` | `{0:-1, -1:0, -2:3}` |

**Result:** `maxLen = 6` ✅ (subarray `nums[0..5]` = `[0,1,0,0,1,1]`, 3 zeros aur 3 ones — is match ke waqt `firstSeen.get(0) = -1` tha, isliye span `0` se `5` tak hai)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (nested loop, running counts) | `O(n²)` | `O(1)` | koi extra structure nahi, par slow |
| Prefix Sum + HashMap (first occurrence) | `O(n)` | `O(n)` | balance `-n` se `n` tak ja sakta hai, isliye map mein max `2n+1` distinct keys |

---

## Gotchas 🪤

- **`0 → -1` trick sabse pehle yaad rakho** — bina isके, "equal zeros and ones" ko prefix-sum shape mein convert nahi kar paoge. Yeh problem ko `subarray-sum-equals-k` (with `k=0`) mein badal deta hai.
- **HashMap yahan COUNT nahi, FIRST INDEX store karta hai** — `subarray-sum-equals-k` ka ulta. Wahan hume "kitni baar" chahiye tha (count), yahan hume "sabse purani kab" chahiye (index), kyunki humein max **length** chahiye, total **count** nahi.
- **Overwrite mat karo** — `firstSeen.containsKey(balance)` check karke sirf tabhi `put` karo jab balance **pehli** baar dikhe. Agar har baar overwrite karoge, to purani (sabse lambi length dene wali) index kho jaayegi.
- **`firstSeen.put(0, -1)` zaroori hai** — index `-1` istemal isliye hota hai kyunki agar balance 0 hi index `i` pe wapas aaye, to poora `nums[0..i]` equal hona chahiye, aur length `i - (-1) = i + 1` (poora prefix) sahi aani chahiye.
- **Balance negative ho sakta hai** — koi dikkat nahi, Java ka `HashMap<Integer,Integer>` negative keys ko bilkul normal handle karta hai (yeh `subarray-sums-divisible-by-k` ke `%` operator wale sign-issue se alag hai).
