# Two Sum II (Sorted Array) — LeetCode 167

> 🔗 **[Interactive Visualizer dekho »]
> (https://yashika807.github.io/DSA-PatternWise/Two-pointer/pair-with-target-sum.html)**
> _(step-through Light End + Heavy End seesaw, live weight readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Two Pointers (sorted array)  ·  **Tags:** Array, Two Pointers, Binary Search

---

## Problem

Ek **1-indexed sorted (ascending)** integer array `numbers` aur ek `target` diya hai. Do aise indices `[i, j]` (`i != j`) dhoondo jinke elements ka sum **exactly `target`** ho. Guarantee hai ki **exactly ek solution** hai, aur same element do baar use nahi kar sakte.

```
Input:  numbers = [2, 7, 11, 15], target = 9
Output: [1, 2]        // numbers[1] + numbers[2] = 2 + 7 = 9  (1-indexed)
```

---

## The story (yaad rakhne ke liye) ⚖️

Ek **seesaw** socho jispe array ke elements sorted order mein khade hain — sabse **halka** (chhota) left pe, sabse **bhaari** (bada) right pe.

| Role | Kaun | Kaam |
|---|---|---|
| **Light End** (`left`) | array ke shuru mein | "sabse halka weight, isse badhaunga" |
| **Heavy End** (`right`) | array ke end mein | "sabse bhaari weight, isse ghataunga" |

Dono ends ka combined weight (`sum`) hamesha **`target`** ke against measure hota hai:

- **sum &lt; target** (seesaw target se halka hai) → Light End ko aur bhaari banao → `left++`
- **sum &gt; target** (seesaw target se bhaari hai) → Heavy End ko halka banao → `right--`
- **sum == target** → perfectly **balanced** ⚖️, indices record karo, done — problem guarantee karta hai unique solution.

> ⚠️ Array **already sorted** hai, isliye left move karne se sum hamesha badhega aur right move karne se hamesha ghategga — yehi property two-pointer ko kaam karne deti hai. Koi sorting khud se nahi karni!

---

## Approach 1 — Brute force 🟥

Har pair try karo.

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int n = numbers.length;
        for (int i = 0; i < n; i++)
            for (int j = i + 1; j < n; j++)
                if (numbers[i] + numbers[j] == target)
                    return new int[]{i + 1, j + 1};   // 1-indexed
        return new int[]{-1, -1};
    }
}
```

**Problem kya hai:** `O(n²)` time — sorted property ka koi use hi nahi ho raha. Array already sorted hai, isse fayda uthana chahiye.

---

## Approach 2 — Two Pointers (Optimal) ✅

Sorted property use karke ends se andar chalo.

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int left = 0, right = numbers.length - 1;   // Light End, Heavy End

        while (left < right) {
            int sum = numbers[left] + numbers[right];

            if (sum == target) {
                return new int[]{left + 1, right + 1};   // 1-indexed, balanced!
            } else if (sum < target) {
                left++;     // too light → move toward heavier
            } else {
                right--;    // too heavy → move toward lighter
            }
        }

        return new int[]{-1, -1};   // guaranteed not to hit this per constraints
    }
}
```

### Kyun kaam karta hai

- Array sorted hai, to `left++` karne se sum sirf badh sakta hai (ya same reh sakta hai duplicates ke saath), aur `right--` se sum sirf ghat sakta hai.
- Isliye ek baar `sum < target` dekh liya, to `left` ko chhod kar peeche jaana kabhi zaroori nahi — hum ek monotonic search space explore kar rahe hain.
- Har step mein ya `left` badhta hai ya `right` ghatta hai → total `O(n)` steps.

---

## Dry run — `numbers = [2, 7, 11, 15]`, `target = 9`

| left | right | numbers[left], numbers[right] | sum | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 0 | 3 | 2, 15 | 17 | too heavy → `right--` |
| 0 | 2 | 2, 11 | 13 | too heavy → `right--` |
| 0 | 1 | 2, 7 | **9** | ✅ balanced! return `[1, 2]` |

**Result:** `[1, 2]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force (2 loops) | `O(n²)` | `O(1)` | ignores the sorted property |
| Two Pointers | `O(n)` | `O(1)` | single pass, ends move inward |

---

## Gotchas 🪤

- **Return 1-indexed, `left+1`/`right+1`** — LeetCode 167 explicitly wants 1-indexed positions, `0`-indexed nahi.
- **Array already sorted maan lo** — is problem mein tumhe sort *nahi* karna, woh already sorted diya hai. Agar unsorted diya jaaye (jaise plain "Two Sum" LC 1), pehle sort karoge to original indices kho jaayenge — us case mein HashMap approach use karo, two-pointer nahi.
- **`left < right` strict** — same element do baar use nahi kar sakte, isliye `<=` nahi, `<` chahiye.
- **Unique solution guarantee** — is version mein exactly ek answer milta hai, isliye duplicate-skip logic (3Sum jaisi) yahan zaroori nahi.
- **Sum overflow** — bade constraints mein `int` overflow ho sakta hai agar values bahut bade hain; yahan constraints chhote hain to `int` theek hai.
