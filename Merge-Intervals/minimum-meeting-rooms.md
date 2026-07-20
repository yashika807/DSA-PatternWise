# Minimum Meeting Rooms — GFG (Attend all Meetings II)

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Merge-Intervals/minimum-meeting-rooms.html)**
> _(step-through sort + min-heap simulation, live room readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sort + Min-Heap  ·  **Tags:** Array, Heap, Greedy, Intervals

---

## Problem

`n` meetings ki `start` aur `end` times di gayi hain. Batao **minimum kitne meeting rooms** chahiye taaki saari meetings bina kisi clash ke ho sakein.

```
Input:  intervals = [[0,30],[5,10],[15,20]]
Output: 2
Reason: [0,30] chalti rehti hai poore time, isliye [5,10] aur [15,20]
        ke liye ek dusra room chahiye — lekin dono ek saath kabhi
        nahi chal rahi, to unhe same second room mein fit kiya ja sakta hai.
```

---

## The story (yaad rakhne ke liye) 🧠

Tum ek **office admin** ho jisko meeting rooms allot karne hain. Meetings **start time se sort** karke ek-ek karke process karo. Tumhare paas ek **"jaldi khaali hone wale rooms" ka heap** hai — jo room sabse pehle free ho raha hai, uska end-time heap ke top pe hoga.

Har nayi meeting aane par:
1. **Sabse pehle khaali hone wala room check karo** (heap ka top). Agar uska end-time is meeting ke start se **pehle ya usi waqt** hai, toh woh room **abhi free ho chuka hai** — usko **reuse** karo (heap se nikaal do).
2. **Is meeting ko ek room do** (naya ho ya reused) — uska end-time heap mein push kar do.

Jitne **room heap mein bache** loop ke end tak, utne hi minimum rooms chahiye — kyunki heap ka size hamesha "abhi kitne rooms use mein hain" batata hai.

| Heap top vs current start | Matlab |
|---|---|
| `heap.top <= meeting.start` | Ek room abhi free ho chuka hai → **reuse** |
| `heap.top > meeting.start` (ya heap khaali) | Koi room free nahi → **naya room** chahiye |

---

## Approach 1 — Brute force 🟥

Har meeting ke liye, ek array of "room free-time" maintain karo aur linearly check karo koi room free hai ya nahi.

```java
class Solution {
    public int minMeetingRooms(int[][] intervals) {
        if (intervals.length == 0) return 0;

        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
        List<Integer> roomEndTimes = new ArrayList<>();   // unsorted list of room free-times

        for (int[] interval : intervals) {
            boolean assigned = false;
            for (int i = 0; i < roomEndTimes.size(); i++) {
                if (roomEndTimes.get(i) <= interval[0]) {   // yeh room free hai
                    roomEndTimes.set(i, interval[1]);
                    assigned = true;
                    break;
                }
            }
            if (!assigned) roomEndTimes.add(interval[1]);   // naya room
        }
        return roomEndTimes.size();
    }
}
```

**Problem kya hai:** Har meeting ke liye **saare existing rooms linearly scan** karne padte hain — worst case `O(n²)`. Heap se "sabse jaldi free hone wala room" turant mil jaata, linear scan mein nahi.

---

## Approach 2 — Sort + Min-Heap (Optimal) ✅

Heap mein hamesha **sabse jaldi free hone wala room top pe** rehta hai — `O(log n)` mein milta hai.

```java
class Solution {
    public int minMeetingRooms(int[][] intervals) {
        if (intervals.length == 0) return 0;

        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);         // sort by start
        PriorityQueue<Integer> minHeap = new PriorityQueue<>(); // room end-times, smallest on top

        for (int[] interval : intervals) {
            // agar sabse jaldi free hone wala room ab tak free ho chuka hai, reuse karo
            if (!minHeap.isEmpty() && minHeap.peek() <= interval[0]) {
                minHeap.poll();
            }
            minHeap.offer(interval[1]);   // is meeting ko room mila — naya ya reused
        }
        return minHeap.size();   // heap size = abhi kitne rooms use mein hain
    }
}
```

### Guardrail jo galti rokta hai

- **`minHeap.peek() <= interval[0]`** — `<=` hai, `<` nahi. Agar ek meeting **exactly usi waqt** shuru hoti hai jab dusri khatam ho rahi hai, room reuse ho sakta hai (touch = no real overlap).
- **Hamesha `offer` karo, poll ke baad bhi** — chahe reuse hua ho ya naya room khula ho, current meeting ka end-time hamesha heap mein jaayega.

---

## Dry run — `[[0,30],[5,10],[15,20]]`

Sorted already: `[0,30],[5,10],[15,20]`

| Meeting | Heap top (before) | Reuse? | Heap (after) |
|:---:|:---:|:---:|:---|
| `[0,30]` | — (empty) | naya room | `{30}` |
| `[5,10]` | `30` | `30 <= 5`? no → naya room | `{10, 30}` |
| `[15,20]` | `10` | `10 <= 15`? yes → reuse, poll `10` | push `20` → `{20, 30}` |

**Final heap size = 2** → **Minimum rooms = 2** ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (linear scan per meeting) | `O(n²)` | `O(n)` | har meeting ke liye saare rooms scan |
| Sort + Min-Heap | `O(n log n)` | `O(n)` | sort `O(n log n)` + har meeting pe `O(log n)` heap op |

---

## Gotchas 🪤

- **`<=` na ki `<`** — touching meetings (`[0,10]` aur `[10,20]`) same room reuse kar sakti hain.
- **Heap ka final size hi answer hai** — koi separate `maxRooms` counter maintain karne ki zaroorat nahi, kyunki hum sirf tabhi pop karte hain jab room turant reuse ho raha ho (net effect: heap size hamesha current concurrent usage reflect karta hai).
- **Sort zaroori hai start time se** — bina sort ke, meetings ko wrong order mein process karoge aur heap ka logic tootega.
- **Empty input** — `intervals.length == 0` pe seedha `0` return karo.
- **Saari meetings overlapping** (jaise `[1,5],[2,6],[3,7],[4,8]`) — kabhi bhi reuse nahi hoga, har meeting apna naya room maangegi → rooms = `n`.
- **Koi meeting overlap na kare** — har meeting turant pichli ke room ko reuse karegi → rooms = `1`, chahe kitni bhi meetings ho.
- **PriorityQueue min-heap by default hai Java mein** — `new PriorityQueue<>()` integers ke liye already min-heap hai, comparator ki zaroorat nahi.
