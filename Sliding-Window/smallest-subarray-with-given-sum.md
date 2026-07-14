# Smallest Subarray with a Given Sum — LeetCode 209 (Minimum Size Subarray Sum)

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Sliding-Window/smallest-subarray-with-given-sum.html)**
> _(step-through grow/shrink window, live running-sum readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sliding Window (variable size, shrinkable)  ·  **Tags:** Array, Sliding Window, Binary Search

---

## Problem

Ek array `nums` (saare elements **positive**) aur ek target `target` diya hai. **Sabse chhoti length** waali contiguous subarray dhoondo jiska sum `>= target` ho. Agar koi nahi mile, `0` return karo.

```
Input:  target = 7, nums = [2, 3, 1, 2, 4, 3]
Output: 2
Reason: subarray [4, 3] ka sum 7 hai, aur yeh sabse chhoti hai
```

---

## The story (yaad rakhne ke liye) 🧠

Socho tumhare paas ek **elastic rubber band** hai jo array ke upar phaila hua hai. Iska kaam hai **kam se kam stretch mein target weight uthana**.

- Jab tak weight (sum) **kam** hai, band ko **right se aur khींचो** — naya element jodo.
- Jaise hi weight target `>=` ho jaaye, socho: *"kya main isse aur tight kar sakta hoon?"* — **left se andar kheecho** (shrink), length note karo, phir aur kheencho jab tak weight target se kam na ho jaaye.

Yeh **grow-then-shrink** rhythm hi is pattern ka dil hai: `right` hamesha aage badhta hai (grow), aur jab bhi condition satisfy ho jaaye, `left` andar aata hai jab tak condition toot na jaaye (shrink) — **dono milke O(n) mein poora array cover karte hain**, kyunki har pointer zyada se zyada `n` baar move karta hai.

> Yeh trick sirf **positive numbers** ke saath kaam karti hai — sum monotonically badhta hai jab window grow hoti hai, isliye shrink karna "safe" hai (sum kam hi hoga, kabhi random jump nahi karega).

---

## Approach 1 — Brute force 🟥

Har start `i` se har end `j` tak subarray sum nikalo, jo bhi `>= target` ho aur sabse chhoti ho use rakho.

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int n = nums.length;
        int minLen = Integer.MAX_VALUE;

        for (int i = 0; i < n; i++) {
            int sum = 0;
            for (int j = i; j < n; j++) {
                sum += nums[j];
                if (sum >= target) {
                    minLen = Math.min(minLen, j - i + 1);
                    break;              // aage badhane se length hi badhegi
                }
            }
        }
        return minLen == Integer.MAX_VALUE ? 0 : minLen;
    }
}
```

**Problem kya hai:** `O(n²)` — har start se fresh sum banate hain. `n` bada ho to slow.

---

## Approach 2 — Optimal (Sliding Window) ✅

`right` se window grow karo. Jaise hi `sum >= target`, `left` se **jitna shrink ho sake** karo — har baar length record karte hue.

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int n = nums.length;
        int left = 0, sum = 0;
        int minLen = Integer.MAX_VALUE;

        for (int right = 0; right < n; right++) {
            sum += nums[right];                 // grow: naya element andar

            // shrink jab tak condition satisfy ho rahi hai
            while (sum >= target) {
                minLen = Math.min(minLen, right - left + 1);
                sum -= nums[left];               // left se element nikalo
                left++;
            }
        }

        return minLen == Integer.MAX_VALUE ? 0 : minLen;
    }
}
```

> ⚠️ Shrink karte waqt **pehle length record karo, phir left ko andar lao** — order ulta karoge to sabse tight window miss ho jaayegi.

---

## Dry run — `target = 7, nums = [2, 3, 1, 2, 4, 3]`

| right | nums[right] | sum (after add) | shrink? | left moves | minLen |
|:---:|:---:|:---:|---|---|:---:|
| 0 | 2 | 2 | no (< 7) | — | ∞ |
| 1 | 3 | 5 | no (< 7) | — | ∞ |
| 2 | 1 | 6 | no (< 7) | — | ∞ |
| 3 | 2 | 8 | yes | len=4 → minLen=4; sum-=2→6, left=1 | 4 |
| 4 | 4 | 10 | yes | len=4(1..4) → minLen=4; sum-=3→7, left=2 | 4 |
| | | 7 | yes | len=3(2..4) → minLen=3; sum-=1→6, left=3 | 3 |
| 5 | 3 | 9 | yes | len=3(3..5) → minLen=3; sum-=2→7, left=4 | 3 |
| | | 7 | yes | len=2(4..5) → minLen=**2**; sum-=4→3, left=5 | **2** |

**Result:** `2` ✅ (subarray `[4, 3]`)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force (nested loop) | `O(n²)` | `O(1)` | fresh sum from every start |
| Sliding Window | `O(n)` | `O(1)` | `left` aur `right` dono mil ke max `2n` steps |

---

## Gotchas 🪤

- **Sirf positive numbers ke liye valid** — agar array mein negative ho sakte hain, `sum` shrink karne pe unpredictably badh/ghat sakta hai aur yeh technique tootegi (tab prefix-sum + binary search ya deque approach chahiye).
- **`while`, not `if`** — shrink loop `while` honi chahiye kyunki ek hi `right` step ke baad multiple elements shrink ho sakte hain (jaise dry run mein `right=4` pe do baar shrink hua).
- **Length record shrink loop ke *andar*, har iteration pe** — sirf ek baar bahar record karoge to tightest window miss ho jaayega.
- **`minLen == Integer.MAX_VALUE` check** — agar koi valid subarray hi nahi mila (total sum bhi target se kam hai), to `0` return karna hai, `MAX_VALUE` nahi.
- **Empty subarray allowed nahi** — `right - left + 1` hamesha `>= 1` rahega kyunki `left <= right` guaranteed hai jab tak `sum >= target` (kam se kam current element to window mein hai).
