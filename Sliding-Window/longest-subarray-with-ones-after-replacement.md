# Max Consecutive Ones III — LeetCode 1004

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Sliding-Window/longest-subarray-with-ones-after-replacement.html)**
> _(step-through grow/shrink window, live zero-count readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sliding Window (variable size, budget counter)  ·  **Tags:** Array, Sliding Window

---

## Problem

Ek binary array `nums` (sirf `0` aur `1`) aur ek number `k` diya hai. Tum **kisi bhi `k` zeros ko `1` mein flip** kar sakte ho. Uske baad **sabse lambi contiguous subarray** ki length nikalo jismein saare `1` hon.

```
Input:  nums = [1,1,1,0,0,0,1,1,1,1,0], k = 2
Output: 6
Reason: window [4..9] = [0,0,1,1,1,1] — dono zeros ko 1 mein flip karo → 6 consecutive 1s
```

---

## The story (yaad rakhne ke liye) 🧠

Socho ek **road trip** pe ho, aur road mein jagah-jagah **potholes** (`0`s) hain. Tumhare paas **`k` asphalt patches** hain — har patch ek pothole ko fix kar sakta hai, taaki woh stretch bilkul **smooth** (`1`s only) ban jaaye.

Tum **right se aage badhte jaate ho** (grow), aur har pothole dikhne pe apna **patch budget** use karte ho (`zeroCount++`). Jab tak budget (`zeroCount <= k`) hai, sab thik hai. Jaise hi **budget khatam** ho jaaye (`zeroCount > k`), tumhe **peeche se** road chhodni padegi (`left` shrink) — jab tak koi ek pothole "un-patch" na ho jaaye aur budget wapas na aa jaaye.

- `zeroCount` = kitne potholes abhi window mein hain (yeh hi budget use ho raha hai).
- `zeroCount > k` → shrink karo left se, jab tak zero exit na ho.
- Har valid step pe (kyunki `zeroCount <= k` hamesha window valid hai) length compare karo.

> Yeh bilkul `fruits-into-baskets` jaisa hi "at most" pattern hai — bas "types" ki jagah "zeros ka budget" track ho raha hai, aur frequency map ki jagah ek simple counter kaafi hai (kyunki hume sirf 0 vs 1 ka fark chahiye).

---

## Approach 1 — Brute force 🟥

Har start se aage badho jab tak zero count `k` se zyada na ho jaaye.

```java
class Solution {
    public int longestOnes(int[] nums, int k) {
        int n = nums.length;
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            int zeros = 0;
            int j = i;
            while (j < n && (zeros + (nums[j] == 0 ? 1 : 0)) <= k) {
                if (nums[j] == 0) zeros++;
                j++;
            }
            maxLen = Math.max(maxLen, j - i);
        }
        return maxLen;
    }
}
```

**Problem kya hai:** `O(n²)` worst case — har start se dobara scan karte hain.

---

## Approach 2 — Optimal (Sliding Window + Budget Counter) ✅

`right` se grow karo, `zeroCount` maintain karo. `zeroCount > k` hote hi `left` se shrink karo jab tak ek zero exit na ho jaaye.

```java
class Solution {
    public int longestOnes(int[] nums, int k) {
        int left = 0, zeroCount = 0, maxLen = 0;

        for (int right = 0; right < nums.length; right++) {
            if (nums[right] == 0) {
                zeroCount++;                 // ek naya pothole, budget use hua
            }

            // budget khatam — patches se zyada potholes ho gaye
            while (zeroCount > k) {
                if (nums[left] == 0) {
                    zeroCount--;              // yeh pothole ab window se bahar, budget wapas
                }
                left++;
            }

            // window hamesha valid hai yahan (zeroCount <= k)
            maxLen = Math.max(maxLen, right - left + 1);
        }

        return maxLen;
    }
}
```

> ⚠️ Shrink loop mein `left++` **hamesha** hota hai, par `zeroCount--` **sirf tab** jab `nums[left] == 0` — agar exiting element `1` hai, budget change nahi hota, sirf window chhoti hoti hai.

---

## Dry run — `nums = [1,1,1,0,0,0,1,1,1,1,0], k = 2`

Index: `0:1 1:1 2:1 3:0 4:0 5:0 6:1 7:1 8:1 9:1 10:0`

| right | val | zeroCount | shrink? | left (after) | window | maxLen |
|:---:|:---:|:---:|---|:---:|:---:|:---:|
| 0 | 1 | 0 | no | 0 | [0,0] | 1 |
| 1 | 1 | 0 | no | 0 | [0,1] | 2 |
| 2 | 1 | 0 | no | 0 | [0,2] | 3 |
| 3 | 0 | 1 | no (≤2) | 0 | [0,3] | 4 |
| 4 | 0 | 2 | no (≤2) | 0 | [0,4] | 5 |
| 5 | 0 | 3 | **yes** → skip `nums[0..2]=1` (left→3, zeroCount stays 3), then `nums[3]=0` → zeroCount=2, left=4, stop | 4 | [4,5] | 5 |
| 6 | 1 | 2 | no | 4 | [4,6] | 5 |
| 7 | 1 | 2 | no | 4 | [4,7] | 5 |
| 8 | 1 | 2 | no | 4 | [4,8] | 5 |
| 9 | 1 | 2 | no | 4 | [4,9] | **6** |
| 10 | 0 | 3 | **yes** → `nums[4]=0` → zeroCount=2, left=5, stop | 5 | [5,10] | 6 |

**Result:** `6` ✅ (window `[4, 9]` = `[0,0,1,1,1,1]` — do zeros ko `1` bana do)

> Row `right=5` dikhata hai ki `while` loop ek `right` step ke liye **multiple baar** chal sakta hai — pehle teen `1`s ko cross karna padta hai (unse budget wapas nahi milta) jab tak ek `0` na mile jo `zeroCount` ko actually kam kare.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(n²)` worst case | `O(1)` | re-scans from every start |
| Sliding Window + counter | `O(n)` | `O(1)` | `left` aur `right` dono milke max `2n` steps |

---

## Gotchas 🪤

- **`zeroCount--` sirf `nums[left] == 0` pe** — `1`s exit karne se budget nahi badhta, sirf `left` aage badhta hai.
- **`while`, not `if`** — dry run mein dekha, ek hi `right` step ke liye left kai baar move ho sakta hai jab tak ek `0` na nikal jaaye.
- **`maxLen` update har step pe**, sirf shrink ke baad nahi — "at most k" pattern hai (`fruits-into-baskets` jaisa), "exactly k" nahi.
- **`k >= n` ya saare zeros`**: agar `k` itna bada hai ki poora array flip ho sake, answer `n` hoga automatically.
- **`k = 0` edge case** — koi flip allowed nahi, answer sabse lambi existing run of `1`s honi chahiye — algorithm khud handle karta hai (zeroCount kabhi `1` se zyada nahi hone dega, `left` turant us `0` ke aage aa jaayega).
