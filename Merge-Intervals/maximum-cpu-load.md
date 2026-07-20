# Maximum CPU Load — GFG

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Merge-Intervals/maximum-cpu-load.html)**
> _(step-through sort + min-heap with running load sum, live peak readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sort + Min-Heap (weighted)  ·  **Tags:** Array, Heap, Greedy, Intervals

---

## Problem

`n` jobs di gayi hain, har job ka apna `start`, `end`, aur `load` (CPU usage) hai. Kisi bhi ek time-instant pe **maximum total CPU load** nikaalo — matlab jitne bhi jobs us waqt overlap kar rahi hon, unke `load` ka sum sabse zyada kitna ho sakta hai.

```
Input:  jobs = [[1,4,3], [2,5,4], [7,9,6]]
        (format: [start, end, load])
Output: 7
Reason: [1,4,3] aur [2,5,4] overlap karti hain (time 2 se 4 tak dono chal rahi hain)
        → load = 3 + 4 = 7, jo sabse zyada hai
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh **Minimum Meeting Rooms** ka hi cousin hai — bas is baar har "meeting" (job) ka apna **weight (CPU load)** bhi hai, aur humein rooms ki **ginti** nahi, balki us waqt **kitna total load chal raha hai** woh chahiye — aur uska **peak** dhoondhna hai.

Wahi min-heap trick: jobs ko **start time se sort** karo, aur ek heap rakho jisme **abhi chal rahi jobs ka `(end, load)` pair** ho, `end` ke hisaab se sabse pehle khatam hone wali top pe. Har nayi job aane se pehle:

1. **Jo bhi jobs is job ke start se pehle khatam ho chuki hain**, unhe heap se nikaal do aur unka `load` current sum se **ghata** do (CPU free ho gaya).
2. **Is job ko heap mein daalo**, uska `load` current sum mein **jodo**.
3. Har step ke baad current sum ko **running maximum** se compare karo.

| Meeting Rooms se farak | Kya |
|---|---|
| Rooms mein sirf **ginti** chahiye (heap.size()) | CPU Load mein **weighted sum** chahiye (`currentLoad`) |
| Rooms mein sirf **final heap size** dekhte hain | CPU Load mein **har step pe max track** karna padta hai (peak kahin bhi beech mein aa sakta hai) |

---

## Approach 1 — Brute force 🟥

Har job ke `start` time pe, saari jobs check karo ki us waqt active hain ya nahi, unka load jod do.

```java
class Solution {
    public int maxCpuLoad(int[][] jobs) {
        int maxLoad = 0;
        for (int[] job : jobs) {
            int t = job[0];             // check load exactly at this job's start
            int loadAtT = 0;
            for (int[] other : jobs) {
                if (other[0] <= t && t < other[1]) {
                    loadAtT += other[2];
                }
            }
            maxLoad = Math.max(maxLoad, loadAtT);
        }
        return maxLoad;
    }
}
```

**Problem kya hai:** `O(n²)` — har candidate time pe saari jobs dobara scan ho rahi hain. Aur candidate times sirf job-starts tak limited rakhna bhi ek extra assumption hai jise prove karna padta hai.

---

## Approach 2 — Sort + Min-Heap with Running Load (Optimal) ✅

```java
class Solution {
    public int maxCpuLoad(int[][] jobs) {
        if (jobs.length == 0) return 0;

        Arrays.sort(jobs, (a, b) -> a[0] - b[0]);           // sort by start
        PriorityQueue<int[]> minHeap =
            new PriorityQueue<>((a, b) -> a[0] - b[0]);      // {end, load}, min by end

        int currentLoad = 0, maxLoad = 0;

        for (int[] job : jobs) {
            int start = job[0], end = job[1], load = job[2];

            // jitni jobs is job ke start se pehle khatam ho chuki hain, unhe hata do
            while (!minHeap.isEmpty() && minHeap.peek()[0] <= start) {
                currentLoad -= minHeap.poll()[1];
            }

            minHeap.offer(new int[]{end, load});
            currentLoad += load;
            maxLoad = Math.max(maxLoad, currentLoad);
        }
        return maxLoad;
    }
}
```

### Guardrails jo galti rokte hain

- **`minHeap.peek()[0] <= start`** — `<=` hai. Job jo **exactly is job ke start pe** khatam ho rahi hai, ab active nahi hai (touching endpoints overlap nahi maane jaate).
- **`while`, na ki `if`** — ek saath **multiple jobs** khatam ho sakti hain is job ke start se pehle; sabko nikalna zaroori hai, sirf ek nahi.
- **`maxLoad` ka update har job ke baad** — peak load kisi bhi job insert hone ke turant baad ban sakta hai, isliye check hamesha loop ke andar hona chahiye, bahar nahi.

---

## Dry run — `[[1,4,3],[2,5,4],[7,9,6]]`

Sorted already by start.

| Job `[s,e,load]` | Heap se remove (expired) | Heap (after) | currentLoad | maxLoad |
|:---:|:---:|:---|:---:|:---:|
| `[1,4,3]` | — | `{(4,3)}` | `3` | `3` |
| `[2,5,4]` | `(4,3)`? `4<=2`? no | `{(4,3),(5,4)}` | `3+4=7` | **`7`** |
| `[7,9,6]` | `(4,3)`: `4<=7`✅ remove; `(5,4)`: `5<=7`✅ remove | `{(9,6)}` | `7-3-4+6=6` | `7` (unchanged) |

**Result:** `maxLoad = 7` ✅ (jobs `[1,4,3]` aur `[2,5,4]` overlap karke `7` dete hain)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (check load at each start) | `O(n²)` | `O(1)` | har start time pe saari jobs scan |
| Sort + Min-Heap | `O(n log n)` | `O(n)` | sort `O(n log n)` + har job pe amortized `O(log n)` heap ops |

---

## Gotchas 🪤

- **`while` loop se saari expired jobs nikalo, sirf ek nahi** — `if` use kiya toh multiple simultaneously-ending jobs miss ho jaayengi, aur `currentLoad` galat reh jaayega.
- **`maxLoad` update loop ke andar, har iteration ke baad** — peak kabhi bhi beech mein aa sakta hai, sirf end mein check karna galat hoga.
- **Meeting Rooms se confuse mat ho** — wahan sirf `heap.size()` chahiye tha (unweighted count); yahan `currentLoad` (weighted sum) track karna padta hai, jo heap size se bilkul alag cheez hai.
- **`<=` na ki `<` expiry check mein** — touching jobs (ek `4` pe khatam, dusri `4` pe shuru) overlap nahi maani jaati.
- **Empty jobs array** — seedha `0` return karo.
- **Saari jobs ek hi waqt active** — koi bhi job expire nahi hogi jab tak saari process na ho jaayein, `currentLoad` sabka sum ban jaayega — sahi behavior hai.
- **Negative ya zero load edge case** — agar `load = 0` ho kisi job ka, woh phir bhi heap mein jaayegi (rooms/time ke hisaab se), bas sum mein kuch add nahi karegi.
