# Squares of a Sorted Array — LeetCode 977

> 🔗 **[Interactive Visualizer dekho »]
> (https://yashika807.github.io/DSA-PatternWise/Two-pointer/squaring-a-sorted-array.html)**
> _(step-through West Cliff + East Cliff echo comparison, live jar-fill readout, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** Two Pointers (fill from back)  ·  **Tags:** Array, Two Pointers, Sorting

---

## Problem

Ek **sorted (ascending)** integer array `nums` diya hai, jismein **negative numbers bhi ho sakte hain**. Har element ka square nikaal ke ek **naya sorted array** return karo.

```
Input:  nums = [-4, -1, 0, 3, 10]
Output: [0, 1, 9, 16, 100]
```

---

## The story (yaad rakhne ke liye) 🏔️

Socho ek **echo canyon** hai — number-line ka `0` canyon ka center hai. Do log khade hain canyon ke dono **cliffs (kinaaron)** pe — **West Cliff** (`left`, sabse negative) aur **East Cliff** (`right`, sabse positive). Jo bhi `0` se **zyada door** hai, uski **echo sabse loud** hoti hai — matlab uska square sabse bada hoga.

| Role | Kaun | Kaam |
|---|---|---|
| **West Cliff** (`left`) | array ke shuru mein | "main `0` se kitna door hoon (negative side)?" |
| **East Cliff** (`right`) | array ke end mein | "main `0` se kitna door hoon (positive side)?" |

Dono cliffs apni **`0` se distance** (absolute value) compare karte hain:

- Jiski echo **loudest** (bada `|value|`) hai, uska square **result jar mein sabse peeche (top se)** rakha jaata hai — kyunki woh **sabse bada** square hoga.
- Phir woh cliff ek kadam **andar** aata hai, aur agla comparison hota hai.
- Yeh **result array ko peeche se aage (right to left)** bharta hai — jar upar se neeche bharne jaisa.

> 🔑 Sorted array mein sabse bada square **hamesha kisi ek end pe hota hai** — beech mein kabhi nahi, kyunki `0` ke paas values sabse chhoti hoti hain. Isi property se two-pointer kaam karta hai.

---

## Approach 1 — Brute force 🟥

Square karo, phir sort karo.

```java
class Solution {
    public int[] sortedSquares(int[] nums) {
        int n = nums.length;
        int[] result = new int[n];
        for (int i = 0; i < n; i++) result[i] = nums[i] * nums[i];
        Arrays.sort(result);
        return result;
    }
}
```

**Problem kya hai:** `O(n log n)` — squaring khud `O(n)` hai, par sorting `O(n log n)` extra lagati hai jab ki input **already sorted** tha. Yeh sorted-ness ka fayda waste kar rahe hain.

---

## Approach 2 — Two Pointers, fill from back (Optimal) ✅

```java
class Solution {
    public int[] sortedSquares(int[] nums) {
        int n = nums.length;
        int[] result = new int[n];

        int left = 0, right = n - 1;
        int writeIdx = n - 1;             // fill result back-to-front

        while (left <= right) {
            int leftSq = nums[left] * nums[left];
            int rightSq = nums[right] * nums[right];

            if (leftSq > rightSq) {
                result[writeIdx] = leftSq;   // West Cliff's echo louder
                left++;
            } else {
                result[writeIdx] = rightSq;  // East Cliff's echo louder (or tie)
                right--;
            }
            writeIdx--;
        }

        return result;
    }
}
```

### Kyun peeche se bharna zaroori hai

- Har step pe humein **sabse bada** remaining square milta hai (jo dono cliffs mein se bada hai), aur sabse bada square **result array ke end** mein jaana chahiye (ascending sorted output).
- Isliye `writeIdx` `n-1` se shuru hoke ghatega — hum "biggest first" nikaal rahe hain, to unhe **piche se** bharna hi sahi jagah pe daalna hai.

---

## Dry run — `[-4, -1, 0, 3, 10]`

| left | right | nums[left]², nums[right]² | Bigger | result[writeIdx] | writeIdx (after) |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 0 | 4 | 16, 100 | East (100) | result[4]=100 | 3 |
| 0 | 3 | 16, 9 | West (16) | result[3]=16 | 2 |
| 1 | 3 | 1, 9 | East (9) | result[2]=9 | 1 |
| 1 | 2 | 1, 0 | West (1) | result[1]=1 | 0 |
| 2 | 2 | 0, 0 | East (tie→right) | result[0]=0 | -1 |

**Result:** `[0, 1, 9, 16, 100]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Square + sort | `O(n log n)` | `O(n)` output | ignores sorted-ness of input |
| Two Pointers (fill from back) | `O(n)` | `O(n)` output only | single pass, no sorting needed |

---

## Gotchas 🪤

- **Tie-breaking (`leftSq == rightSq`)** — chahe left le lo ya right, result correct hi rahega (equal squares), code mein `>` use kiya (strict), tie pe `right` ka square jaata hai — dono equally valid hain.
- **All-negative ya all-positive array** — algorithm phir bhi kaam karta hai; ek cliff apni values consume karta rahega, doosra turant `left > right` ho jaayega us side pe (loop khud handle kar leta hai, extra check nahi chahiye).
- **In-place mat try karo** — result array alag banana zaroori hai, kyunki original `nums` ko read karte waqt overwrite karoge to data loss ho sakta hai (medium-hard variant mein in-place possible hai but tricky, yahan avoid karo).
- **`left <= right`, na ki `<`** — jab `left == right` (single element bacha), usse bhi process karna hai, isliye `<=` chahiye warna last element miss ho jaayega.
- **Integer overflow** — agar `nums[i]` bahut bada ho (jaise `±10⁵`), to square `10^10` tak jaa sakta hai jo `int` range se bahar hai — bade constraints mein `long` use karna padega. LeetCode 977 constraints (`|nums[i]| ≤ 10⁴`) mein `int` safe hai.
