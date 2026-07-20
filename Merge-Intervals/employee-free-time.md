# Employee Free Time — LeetCode 759

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Merge-Intervals/employee-free-time.html)**
> _(step-through flatten + merge + gap-scan, live free-slot readout, synced Java code)_

**Difficulty:** Hard  ·  **Pattern:** Flatten + Merge + Gap Scan  ·  **Tags:** Array, Sorting, Intervals

---

## Problem

`schedule` diya hai — ek list of employees, har employee ki apni list of **busy intervals** (already sorted + non-overlapping within khud unki list). Saare employees ka **common free time** nikaalo — matlab woh time-windows jab **koi bhi employee busy nahi** hai (company-wide).

```
Input:  schedule = [ [[1,2],[5,6]], [[1,3]], [[4,10]] ]
        (3 employees, har ek ki apni busy-intervals list)
Output: [[3,4]]
Reason: 3 se 4 ke beech koi bhi employee busy nahi hai — baaki poora
        din kisi na kisi employee ki koi na koi meeting chal rahi hai
```

---

## The story (yaad rakhne ke liye) 🧠

Socho pura office ek **hi shared calendar** dekh raha hai — sabki meetings ek saath overlay ho gayi hain. Tumhe woh **gaps** dhoondhne hain jab **poora office khaali** ho (na koi meeting chal rahi ho, kisi bhi employee ki).

Teen seedhe steps:

1. **Flatten** — sabhi employees ki saari busy-intervals ko ek hi badi list mein daal do. Ab yeh **exactly Merge Intervals wala problem** ban jaata hai.
2. **Merge** — start time se sort karo aur overlapping busy blocks ko merge kar do — jaise poore office ke **combined busy periods**.
3. **Gap Scan** — merged blocks ke **beech ke gaps** hi free time hain. Consecutive merged blocks ke beech `[block[i].end, block[i+1].start]` — bas yehi free windows hain.

| Step | Kaam | Result |
|---|---|---|
| Flatten | sab employees ki intervals ek list mein | raw unsorted intervals |
| Merge | Merge Intervals ka standard algorithm | company-wide busy blocks |
| Gap Scan | consecutive merged blocks ke beech ka fasla | free time windows |

> 🔑 Yehi trick hai — **koi naya algorithm nahi seekhna**, sirf Merge Intervals ke result pe ek chhota extra pass lagana hai.

---

## Approach 1 — Brute force 🟥

Sabse chhoti se sabse badi time tak, **har unit time check karo** ki koi employee busy hai ya nahi (sirf integer times ke liye conceptually).

```java
class Solution {
    public List<int[]> employeeFreeTime(List<List<int[]>> schedule) {
        List<int[]> all = new ArrayList<>();
        for (List<int[]> emp : schedule) all.addAll(emp);
        if (all.isEmpty()) return new ArrayList<>();

        int minT = Integer.MAX_VALUE, maxT = Integer.MIN_VALUE;
        for (int[] iv : all) { minT = Math.min(minT, iv[0]); maxT = Math.max(maxT, iv[1]); }

        boolean[] busy = new boolean[maxT - minT + 1];
        for (int[] iv : all)
            for (int t = iv[0]; t < iv[1]; t++) busy[t - minT] = true;

        List<int[]> free = new ArrayList<>();
        int i = 0;
        while (i < busy.length) {
            if (!busy[i]) {
                int start = i;
                while (i < busy.length && !busy[i]) i++;
                if (start != 0 && i != busy.length) free.add(new int[]{start + minT, i + minT});
            } else i++;
        }
        return free;
    }
}
```

**Problem kya hai:** Time range bahut bada ho sakta hai (`boolean[]` explode ho jaayega), aur `O(range)` time — intervals ke count `n` se related hi nahi hai. Bahut wasteful.

---

## Approach 2 — Flatten + Merge + Gap Scan (Optimal) ✅

```java
class Solution {
    public List<int[]> employeeFreeTime(List<List<int[]>> schedule) {
        // Step 1: flatten sab employees ki intervals ek list mein
        List<int[]> all = new ArrayList<>();
        for (List<int[]> emp : schedule) all.addAll(emp);

        all.sort((a, b) -> a[0] - b[0]);   // sort by start

        // Step 2: merge overlapping busy blocks (Merge Intervals ka standard scan)
        List<int[]> merged = new ArrayList<>();
        int[] current = all.get(0);
        merged.add(current);

        for (int[] iv : all) {
            if (iv[0] <= current[1]) {
                current[1] = Math.max(current[1], iv[1]);
            } else {
                current = iv;
                merged.add(current);
            }
        }

        // Step 3: consecutive merged blocks ke beech ka gap = free time
        List<int[]> free = new ArrayList<>();
        for (int i = 1; i < merged.size(); i++) {
            free.add(new int[]{merged.get(i - 1)[1], merged.get(i)[0]});
        }
        return free;
    }
}
```

### Guardrails jo galti rokte hain

- **Sirf consecutive merged blocks ke beech gap nikalo** — merged list already non-overlapping + sorted hai, isliye bas ek linear pass mein `end[i-1]` se `start[i]` tak ka gap le lo.
- **Pehle busy block se pehle ya aakhri ke baad ka time free time NAHI hai** — sirf **beech ke** gaps count hote hain (company ke "office hours" se bahar ka time undefined maana jaata hai).

---

## Dry run — `[[[1,2],[5,6]], [[1,3]], [[4,10]]]`

**Step 1 — Flatten:** `[1,2]`, `[5,6]`, `[1,3]`, `[4,10]`

**Step 2 — Sort + Merge:**

| Interval | current block | Verdict |
|:---:|:---:|:---|
| `[1,2]` | `[1,2]` | seed |
| `[1,3]` | `[1,2]` | `1<=2` → extend: `[1,3]` |
| `[4,10]` | `[1,3]` | `4<=3`? no → finalize `[1,3]`, new block `[4,10]` |
| `[5,6]` | `[4,10]` | `5<=10` → extend (no change, already covers): `[4,10]` |

Merged busy blocks: **`[[1,3], [4,10]]`**

**Step 3 — Gap scan:**

| i | merged[i-1] | merged[i] | Gap |
|:---:|:---:|:---:|:---|
| 1 | `[1,3]` | `[4,10]` | `[3, 4]` |

**Result:** `[[3,4]]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (mark every time unit) | `O(range)` | `O(range)` | range bahut bada ho sakta hai, `n` se independent |
| Flatten + Merge + Gap Scan | `O(n log n)` | `O(n)` | `n` = total intervals across all employees |

---

## Gotchas 🪤

- **Pehle aur aakhri block se bahar ka time free time nahi hai** — sirf merged blocks ke **beech** ke gaps chahiye, `i` loop `1` se shuru hota hai `merged.size()-1` tak, extremes ko exclude karte hue.
- **Sirf ek hi merged block bacha** (jaise ek employee, ek interval) — `merged.size() == 1` toh gap-scan loop chalega hi nahi, free time empty list hogi.
- **Flatten karte waqt employee ka context kho jaata hai — aur yeh theek hai** — kyunki humein **kisne kya busy kiya** nahi jaanna, sirf company-wide busy/free chahiye.
- **Intervals employee ke andar already sorted/non-overlapping maane jaate hain, lekin employees ke beech nahi** — isliye poora flatten karke phir se sort + merge karna zaroori hai.
- **Touching busy intervals bhi merge hote hain** (`iv[0] <= current[1]`) — agar `[1,3]` aur `[3,5]` do alag employees ki hon, woh ek hi busy block `[1,5]` ban jaayenge, beech mein koi free gap nahi banega.
- **Empty schedule ya sab employees ki khaali list** — `all` khaali reh jaayega, is case ko handle karna zaroori hai (warna `all.get(0)` crash karega).
