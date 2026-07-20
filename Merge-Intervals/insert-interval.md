# Insert Interval — LeetCode 57

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Merge-Intervals/insert-interval.html)**
> _(step-through 3-phase scan, live merge-window readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Linear Scan (3-phase)  ·  **Tags:** Array, Intervals

---

## Problem

Ek `intervals` array diya hai jo already **sorted by start time** hai aur **non-overlapping** hai. Ek naya `newInterval` diya hai. Isse insert karo aur agar zaroorat pade to overlapping intervals ko merge karo, taaki result bhi sorted + non-overlapping rahe.

```
Input:  intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]
Reason: [2,5] overlaps [1,3] → merge ho gaya [1,5], [6,9] untouched
```

---

## The story (yaad rakhne ke liye) 🧠

Tumhara calendar pehle se hi **clean** hai — saari bookings time-order mein hain aur koi do bookings overlap nahi karti. Ab boss ne ek **nayi meeting** book karne ko bola hai. Teen kaam karne hain, ekdum order mein:

1. **Purani bookings jo nayi meeting shuru hone se pehle hi khatam ho jaati hain** — unhe touch mat karo, seedha copy kar do.
2. **Jo bookings nayi meeting se time-overlap karti hain** — unhe nayi meeting ke andar **absorb** kar lo, ek badi merged meeting bana do.
3. **Baaki bachi bookings jo nayi meeting ke baad shuru hoti hain** — unhe bhi seedha copy kar do.

Kyunki calendar pehle se sorted hai, humein poora **sort karne ki zaroorat nahi** — bas ek hi seedha left-to-right scan kaafi hai, teen phases mein.

| Phase | Kya | Kab rukta hai |
|---|---|---|
| 1️⃣ Before | as-is copy | jab tak `interval.end < newInterval.start` |
| 2️⃣ Merge window | absorb + expand bounds | jab tak `interval.start <= mergedEnd` |
| 3️⃣ After | as-is copy | list khatam hone tak |

---

## Approach 1 — Brute force 🟥

`newInterval` ko list mein daal do, phir poore array ko **Merge Intervals** ki tarah sort karke merge kar do — nayi booking ko special treat mat karo.

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        int n = intervals.length;
        int[][] all = new int[n + 1][];
        System.arraycopy(intervals, 0, all, 0, n);
        all[n] = newInterval;

        Arrays.sort(all, (a, b) -> a[0] - b[0]);   // sort everything again

        List<int[]> result = new ArrayList<>();
        int[] current = all[0];
        result.add(current);

        for (int[] interval : all) {
            if (interval[0] <= current[1]) {
                current[1] = Math.max(current[1], interval[1]);
            } else {
                current = interval;
                result.add(current);
            }
        }
        return result.toArray(new int[result.size()][]);
    }
}
```

**Problem kya hai:** Existing array **already sorted** tha — usko dobara `O(n log n)` sort karna waste hai. Interviewer poochega "array already sorted hai, isko use kyu nahi kar rahe?"

---

## Approach 2 — 3-Phase Linear Scan (Optimal) ✅

Sort ki zaroorat nahi. Ek hi pass mein teeno phases chala do.

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        List<int[]> result = new ArrayList<>();
        int i = 0, n = intervals.length;

        // Phase 1: intervals ending strictly before newInterval starts
        while (i < n && intervals[i][1] < newInterval[0]) {
            result.add(intervals[i]);
            i++;
        }

        // Phase 2: merge everything that overlaps newInterval
        int start = newInterval[0], end = newInterval[1];
        while (i < n && intervals[i][0] <= end) {
            start = Math.min(start, intervals[i][0]);
            end = Math.max(end, intervals[i][1]);
            i++;
        }
        result.add(new int[]{start, end});

        // Phase 3: everything remaining, untouched
        while (i < n) {
            result.add(intervals[i]);
            i++;
        }

        return result.toArray(new int[result.size()][]);
    }
}
```

### Guardrails jo galti rokte hain

- **Phase 1 ka condition `intervals[i][1] < newInterval[0]`** — `<` hai, `<=` nahi. Agar touch bhi karte hain (`end == newInterval.start`), overlap maana jaata hai, isliye Phase 2 mein jaayega.
- **Phase 2 ka condition `intervals[i][0] <= end`** — yahan `end` **updated `end`** hai (merge window ka current end), `newInterval[1]` nahi. Chain-merge isi wajah se kaam karta hai.

---

## Dry run — `intervals=[[1,2],[3,5],[6,7],[8,10],[12,16]]`, `newInterval=[4,8]`

| i | intervals[i] | Phase | Action | merge window |
|:---:|:---:|:---:|:---|:---:|
| 0 | `[1,2]` | 1 | `2 < 4` → copy as-is | — |
| 1 | `[3,5]` | 1→2 | `5 < 4`? no → Phase 1 stop. `3 <= 8` → absorb | `start=3, end=8`* |
| 2 | `[6,7]` | 2 | `6 <= 8` → absorb | `start=3, end=8` |
| 3 | `[8,10]` | 2 | `8 <= 8` → absorb | `start=3, end=10` |
| 4 | `[12,16]` | 2→3 | `12 <= 10`? no → Phase 2 stop, push `[3,10]`. `12` copy as-is | — |

\* `start = min(4,3)=3`, `end = max(4,5)=8` (window seed the newInterval `[4,8]` first, then absorb `[3,5]`).

**Result:** `[[1,2],[3,10],[12,16]]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (re-sort everything) | `O(n log n)` | `O(n)` | wastes the fact that input is already sorted |
| 3-Phase Linear Scan | `O(n)` | `O(n)` output ke liye | ek hi pass, koi sorting nahi |

---

## Gotchas 🪤

- **Phase 1 `<` vs Phase 2 `<=`** — touching intervals overlap count hote hain, isliye boundary conditions asymmetric hain: Phase 1 ruk jaata hai jaise hi touch bhi ho jaaye, Phase 2 usi touch ko absorb kar leta hai.
- **Phase 2 ka `end` continuously update hota hai** — comparison hamesha **current merge window ke end** se hota hai, original `newInterval[1]` se nahi. Isse chain-merges (`[3,5]`→`[6,7]`→`[8,10]`) sahi se pakde jaate hain.
- **Empty `intervals` array** — Phase 1 aur Phase 2 dono loops turant skip ho jaate hain (`i < n` false), aur `newInterval` seedha result mein add ho jaata hai.
- **`newInterval` poori tarah kisi existing interval ke andar** — merge window ka `start`/`end` change hi nahi hoga (existing interval already bada hai), phir bhi ek hi interval result mein jaayega, do nahi.
- **Input already sorted hai — dobara sort mat karo** — yehi is problem ka poora point hai; agar sort kar diya to `O(n)` optimal solution `O(n log n)` ban jaayega.
- **`newInterval` sabse aage ya sabse peeche jaa sakta hai** — agar sabse pehle (koi overlap nahi Phase 1 mein), Phase 1 turant khatam; agar sabse aakhri mein, Phase 3 kabhi chalega hi nahi.
