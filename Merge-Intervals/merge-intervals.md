# Merge Intervals — LeetCode 56

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Merge-Intervals/merge-intervals.html)**
> _(step-through sort + linear scan, live merge readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sort + Linear Scan  ·  **Tags:** Array, Sorting, Intervals

---

## Problem

Ek array `intervals` diya hai jahan `intervals[i] = [start_i, end_i]`. Saare **overlapping intervals** ko merge karo aur ek naya array return karo jismein sirf non-overlapping intervals ho jo input ke saare intervals ko cover karte hon.

```
Input:  intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Reason: [1,3] aur [2,6] overlap karte hain → merge ho gaye [1,6] mein
```

---

## The story (yaad rakhne ke liye) 🧠

Socho tumhare paas ek **shared office calendar** hai — poore din ki bookings ek scattered list mein padi hain, koi order nahi. Kuch bookings ek dusre se **overlap** karti hain (do log same room double-book kar gaye), aur tumhe calendar ko **clean-up** karna hai — jitni bhi bookings time mein touch/overlap karti hain unhe ek hi badi booking mein **merge** kar do, taaki final calendar mein sirf clean non-overlapping blocks dikhein.

Trick yeh hai: bookings ko pehle **start time se time-order mein saja do**. Ek baar line time-order mein lag jaaye, to bas ek pass mein chalte jao — agar agli booking abhi chal rahi booking khatam hone se **pehle (ya usi pal)** shuru ho rahi hai, wo overlap karti hai → merge kar do (end ko badha do agar zaroorat ho). Warna, purani booking finalize karo aur nayi booking se shuru karo.

| Role | Kaun | Kaam |
|---|---|---|
| **Sorted queue** | saari bookings, start-time order mein | ek-ek karke process hongi |
| **Current merged block** | jo booking abhi "open" hai | naye overlaps ko apne andar absorb karta hai |
| **Result list** | finalize ho chuke blocks | jab current block se agli booking overlap na kare, tabhi yahan push hota hai |

---

## Approach 1 — Brute force 🟥

Sorting ke bina, har interval ko baaki sabse compare karo aur jo bhi overlap kare use usi group mein absorb karo — jab tak koi naya merge na ho tab tak repeat karte raho (jaise connected-components dhoondna).

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        int n = intervals.length;
        List<int[]> result = new ArrayList<>();
        boolean[] used = new boolean[n];

        for (int i = 0; i < n; i++) {
            if (used[i]) continue;
            int start = intervals[i][0], end = intervals[i][1];
            used[i] = true;

            boolean mergedSomething = true;
            while (mergedSomething) {                  // repeat sweep jab tak stable na ho
                mergedSomething = false;
                for (int j = 0; j < n; j++) {
                    if (used[j]) continue;
                    if (intervals[j][0] <= end && intervals[j][1] >= start) {
                        start = Math.min(start, intervals[j][0]);
                        end   = Math.max(end, intervals[j][1]);
                        used[j] = true;
                        mergedSomething = true;
                    }
                }
            }
            result.add(new int[]{start, end});
        }
        return result.toArray(new int[result.size()][]);
    }
}
```

**Problem kya hai:** Har group ke liye poora array baar-baar sweep karna padta hai — worst case `O(n³)`. Interviewer turant "sort karke dekho" bolega.

---

## Approach 2 — Sort + Linear Scan (Optimal) ✅

Intervals ko `start` se sort karo. Phir ek `current` block rakho — agar agla interval `current` se overlap kare (`interval.start <= current.end`), toh `current.end` ko badha do; warna `current` ko result mein finalize karke naya `current` bana do.

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        if (intervals.length <= 1) return intervals;

        // sort by start time — is se hi linear scan chalta hai
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);

        List<int[]> result = new ArrayList<>();
        int[] current = intervals[0];
        result.add(current);

        for (int[] interval : intervals) {
            if (interval[0] <= current[1]) {
                // overlap (ya touch) — current block ko extend karo
                current[1] = Math.max(current[1], interval[1]);
            } else {
                // gap mil gaya — purana block finalize, naya shuru
                current = interval;
                result.add(current);
            }
        }
        return result.toArray(new int[result.size()][]);
    }
}
```

### Guardrail jo galti rokta hai

- **`interval[0] <= current[1]`** — `<=` hai, `<` nahi. Touching intervals jaise `[1,2]` aur `[2,3]` bhi merge hote hain (`[1,3]`), kyunki `2 <= 2` sach hai.

---

## Dry run — `[[1,3],[2,6],[8,10],[15,18]]`

Already sorted by start.

| Interval | current block | Comparison | Verdict |
|:---:|:---:|:---|:---|
| `[1,3]` | `[1,3]` | first block, seed | `current = [1,3]` |
| `[2,6]` | `[1,3]` | `2 <= 3` → overlap | extend: `current = [1,6]` |
| `[8,10]` | `[1,6]` | `8 <= 6`? no | gap → finalize `[1,6]`, `current = [8,10]` |
| `[15,18]` | `[8,10]` | `15 <= 10`? no | gap → finalize `[8,10]`, `current = [15,18]` |
| — | `[15,18]` | loop khatam | finalize `[15,18]` |

**Result:** `[[1,6],[8,10],[15,18]]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (repeated sweep) | `O(n³)` worst case | `O(n)` | koi sorting nahi, group-wise repeated scan |
| Sort + Linear Scan | `O(n log n)` | `O(n)` output ke liye | sort dominate karta hai, scan `O(n)` hai |

---

## Gotchas 🪤

- **`<=` na ki `<`** — touching intervals `[1,2]` aur `[2,3]` bhi merge hote hain. `<` use kiya toh galat, alag-alag reh jaayenge.
- **Sort pehle, warna scan bekaar** — bina sort ke `current.end` ka comparison meaningless hai, kyunki agla interval kahin bhi ho sakta hai.
- **`current[1] = Math.max(current[1], interval[1])`** — sirf `interval[1]` assign mat karo, kyunki nested interval jaise `[1,10]` ke andar `[2,3]` aa sakta hai jo current ka end **chhota** kar dega agar max na liya.
- **Empty ya single-interval input** — `intervals.length <= 1` pe seedha return karo, warna `intervals[0]` access crash kar sakta hai empty array pe.
- **`intervals[i]` ko directly mutate karna** — brute force wale approach mein agar `intervals[i]` ke reference ko modify karo (jaise `current = intervals[0]`), original array bhi badal sakta hai — output array banate waqt naya array banao, reference reuse mat karo agar input immutable rakhna hai.
- **In-place sort input array ko modify karta hai** — agar caller original order expect kar raha hai, copy pe kaam karo.
