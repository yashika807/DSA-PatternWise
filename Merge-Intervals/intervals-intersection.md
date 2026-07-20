# Interval List Intersections — LeetCode 986

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Merge-Intervals/intervals-intersection.html)**
> _(step-through two-pointer scan across two schedules, live intersection readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Two Pointers on Two Sorted Lists  ·  **Tags:** Array, Two Pointers, Intervals

---

## Problem

Do lists `firstList` aur `secondList` di gayi hain — dono apne andar **sorted aur non-overlapping** hain (har list ke apne intervals ek dusre se overlap nahi karte). Dono lists ka **pairwise intersection** return karo (jitne bhi common time-ranges hain).

```
Input:  firstList  = [[0,2],[5,10],[13,23],[24,25]]
        secondList = [[1,5],[8,12],[15,24],[25,26]]
Output: [[1,2],[5,5],[8,10],[15,23],[24,24],[25,25]]
```

---

## The story (yaad rakhne ke liye) 🧠

Do employees hain — **A** aur **B** — dono ki apni **busy schedule** hai, already sorted aur non-overlapping (koi employee khud se do jagah busy nahi ho sakta). Tumhe woh saare time-windows nikalne hain jab **dono ek saath busy** the.

Kyunki dono schedules already sorted hain, tum ek **two-pointer walk** kar sakte ho — `i` A ki current meeting pe, `j` B ki current meeting pe. Har step pe unka **overlap nikaalo** (agar ho), phir jis employee ki current meeting **pehle khatam** hoti hai, uska pointer aage badhao — kyunki uski agli meeting hi naya overlap la sakti hai.

| Role | Kaun | Kaam |
|---|---|---|
| **Pointer `i`** | A ki current meeting | jab tak A ki meeting khatam na ho, wahi rehta hai |
| **Pointer `j`** | B ki current meeting | jab tak B ki meeting khatam na ho, wahi rehta hai |
| **Overlap window** | `[max(start), min(end)]` | agar valid (`start <= end`), toh yeh ek result hai |

> 🔑 Jis meeting ka `end` pehle aata hai, uska pointer aage badhta hai — us employee ki **is meeting se ab koi naya overlap nahi ban sakta**, agli meeting try karo.

---

## Approach 1 — Brute force 🟥

Har `A[i]` ko har `B[j]` se compare karo, jahan bhi overlap mile record karo.

```java
class Solution {
    public int[][] intervalIntersection(int[][] firstList, int[][] secondList) {
        List<int[]> result = new ArrayList<>();

        for (int[] a : firstList) {
            for (int[] b : secondList) {
                int start = Math.max(a[0], b[0]);
                int end = Math.min(a[1], b[1]);
                if (start <= end) {
                    result.add(new int[]{start, end});
                }
            }
        }
        return result.toArray(new int[result.size()][]);
    }
}
```

**Problem kya hai:** `O(n·m)` — dono lists sorted hone ka koi fayda nahi utha rahe. Bade inputs pe slow.

---

## Approach 2 — Two Pointers (Optimal) ✅

Dono sorted lists pe ek-ek pointer rakho. Overlap check karo, phir jiski meeting pehle khatam hoti hai uska pointer aage badhao.

```java
class Solution {
    public int[][] intervalIntersection(int[][] firstList, int[][] secondList) {
        List<int[]> result = new ArrayList<>();
        int i = 0, j = 0;

        while (i < firstList.length && j < secondList.length) {
            int start = Math.max(firstList[i][0], secondList[j][0]);
            int end = Math.min(firstList[i][1], secondList[j][1]);

            if (start <= end) {                    // valid overlap
                result.add(new int[]{start, end});
            }

            // jiski meeting pehle khatam hoti hai, uska pointer badhao
            if (firstList[i][1] < secondList[j][1]) {
                i++;
            } else {
                j++;
            }
        }
        return result.toArray(new int[result.size()][]);
    }
}
```

### Guardrail jo galti rokta hai

- **`start <= end` mein `<=` hai, `<` nahi** — single-point intersections (jaise `[5,5]`) bhi valid overlap maane jaate hain, jab do meetings sirf ek instant ke liye touch karti hain.
- **Advance decision `firstList[i][1] < secondList[j][1]` par based hai, `start`/`end` par nahi** — hamesha woh pointer badhao jiski **meeting pehle khatam** ho rahi hai.

---

## Dry run — `A=[[0,2],[5,10],[13,23],[24,25]]`, `B=[[1,5],[8,12],[15,24],[25,26]]`

| i | j | A[i] | B[j] | start,end | Valid? | Advance |
|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| 0 | 0 | `[0,2]` | `[1,5]` | max(0,1)=1, min(2,5)=2 | ✅ `[1,2]` | `2<5` → `i++` |
| 1 | 0 | `[5,10]` | `[1,5]` | max(5,1)=5, min(10,5)=5 | ✅ `[5,5]` | `10<5`? no → `j++` |
| 1 | 1 | `[5,10]` | `[8,12]` | max(5,8)=8, min(10,12)=10 | ✅ `[8,10]` | `10<12` → `i++` |
| 2 | 1 | `[13,23]` | `[8,12]` | max(13,8)=13, min(23,12)=12 | ❌ `13>12` | `23<12`? no → `j++` |
| 2 | 2 | `[13,23]` | `[15,24]` | max(13,15)=15, min(23,24)=23 | ✅ `[15,23]` | `23<24` → `i++` |
| 3 | 2 | `[24,25]` | `[15,24]` | max(24,15)=24, min(25,24)=24 | ✅ `[24,24]` | `25<24`? no → `j++` |
| 3 | 3 | `[24,25]` | `[25,26]` | max(24,25)=25, min(25,26)=25 | ✅ `[25,25]` | `25<26` → `i++` |
| 4 | 3 | — | — | `i==firstList.length` | — | **loop ends** |

**Result:** `[[1,2],[5,5],[8,10],[15,23],[24,24],[25,25]]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (nested loop) | `O(n·m)` | `O(k)` | `k` = result size; ignores sorted-ness |
| Two Pointers | `O(n + m)` | `O(k)` | ek hi combined pass, dono lists ek baar traverse |

---

## Gotchas 🪤

- **`start <= end`, `<` nahi** — point-intersections (`[5,5]`) valid hain; unhe miss mat karo.
- **Advance decision hamesha `end` compare karke** — kabhi bhi `start` compare karke pointer mat badhao, warna valid overlaps miss ho jaayenge.
- **Ek list khaali ho sakti hai** — `while` condition dono lengths check karta hai, isliye empty list pe loop turant khatam ho jaata hai aur result empty rehta hai.
- **Jab dono ends equal hon** (`firstList[i][1] == secondList[j][1]`) — `else` branch chalta hai, matlab `j++` hota hai (na ki dono ek saath badhte hain). Yeh bhi sahi hai kyunki agli iteration mein `i` ka overlap check firse hoga.
- **Dono lists apne andar non-overlapping honi chahiye** — yeh algorithm tabhi kaam karta hai jab har list internally sorted + non-overlapping ho; agar nahi hai to pehle Merge Intervals se clean karo.
- **Output khud automatically sorted aata hai** — kyunki dono pointers sirf aage badhte hain, kabhi peeche nahi jaate.
