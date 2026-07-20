# Linked List Cycle II — LeetCode 142

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/fast-and-slow/start-of-linkedlist-cycle.html)**
> _(Phase 1 meeting point + Phase 2 entry-finder, live index readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Fast & Slow Pointers (Floyd's Cycle Detection — 2 Phase)  ·  **Tags:** Linked List, Two Pointers, Math

---

## Problem

Ek singly linked list ka `head` diya hai. Agar list mein cycle hai, to cycle **shuru kahan se hota hai** wahi node return karo. Agar cycle nahi hai, `null` return karo.

```
Input:  3 → 2 → 0 → -4 → (wapas node "2" se jud jaata hai)
Output: node with value 2

Input:  1 → 2 → null
Output: null
```

Bas `true/false` bataana kaafi nahi — is baar **exact node** chahiye jahan se loop shuru hota hai. Aur haan, list ko modify nahi kar sakte.

---

## The story (yaad rakhne ke liye) 🐢🐇📍

LC 141 wala race track yaad hai? Kachhua (`slow`) aur khargosh (`fast`) chalte hain, khargosh ek din kachhue ko cycle ke andar **lap** kar leta hai — wahi unka *meeting point* hai. Lekin meeting point khud loop ka **starting gate** nahi hota — wo to bas loop ke *kisi bhi* point pe ho sakta hai.

Ab ek **maths ka jaadu** kaam aata hai: agar tum ek naya runner (`ptr`) bilkul **race ke shuru** (`head`) se chalao — usi speed se jitni kachhua chal raha hai (+1 step) — to `ptr` aur kachhua **exactly cycle ke starting gate par milenge**. Kyun? Kyunki jitni distance `head` se gate tak hai, utni hi distance meeting-point se gate tak (loop ghoom ke) bhi hai — yeh Floyd's algorithm ki distance-proof property hai. Isiliye do runners jo alag jagah se start karke same speed se chalte hain, **gate par hi sync** ho jaate hain.

Do phases: **Phase 1** — meeting point dhoondo (jaisa LC 141). **Phase 2** — ek pointer ko head par reset karo, dono ko +1-+1 chalao jab tak mile nahi. Jahan milenge, wahi entry hai.

---

## Approach 1 — Brute force (Hashing)

Traverse karte waqt har node ko `HashSet` mein daalo. Jo node **pehli baar dobara mile** (already set mein present), wahi cycle ka start hai.

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        Set<ListNode> seen = new HashSet<>();
        ListNode curr = head;
        while (curr != null) {
            if (seen.contains(curr)) return curr;   // pehla repeat = entry
            seen.add(curr);
            curr = curr.next;
        }
        return null;
    }
}
```

**Time:** `O(n)`  
**Space:** `O(n)` — set mein saare visited nodes

Sahi jawaab deta hai, but interview mein constant-space follow-up expected hota hai.

---

## Approach 2 — Optimal (Fast & Slow Pointers, 2 Phase)

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head, fast = head;

        // Phase 1: detect a meeting point inside the cycle
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;

            if (slow == fast) {                 // they met -> cycle exists

                // Phase 2: reset one pointer to head, move both +1
                ListNode ptr = head;
                while (ptr != slow) {
                    ptr = ptr.next;
                    slow = slow.next;
                }
                return ptr;                      // cycle entry node
            }
        }
        return null;                             // fast fell off the end -> no cycle
    }
}
```

### Why phase 2 works (quick proof, taaki ratna na pade)

Maan lo `head` se entry gate tak distance = `a`, gate se meeting point tak (cycle ke andar, jis direction traverse hota hai) = `b`, aur meeting point se wapas gate tak (loop poora karke) = `c`. Cycle length = `b + c`.

Jab `slow`/`fast` milte hain: `slow` ne `a + b` steps chale, `fast` ne `2(a+b)` steps chale (double speed), aur fast ne extra `k` poore cycles kaate hain (`k >= 1`):

```
2(a+b) = a+b + k(b+c)
a+b = k(b+c)
a = k(b+c) - b = (k-1)(b+c) + c
```

Matlab `a` (head→gate) **barabar hai** `c` (meeting-point→gate, poore k-1 extra loops ke saath) ke. Isiliye `head` se +1-+1 chalne wala `ptr`, aur meeting-point se +1-+1 chalne wala `slow`, **exactly gate par mil jaate hain**.

**Time:** `O(n)` — dono phases milke O(n)  
**Space:** `O(1)` — sirf pointers

---

## Dry run

Input: `3 → 2 → 0 → -4 → (tail wapas index 1 = value 2 se jud-ta hai)`  (indices: `0:3, 1:2, 2:0, 3:-4`)

**Phase 1:**

| Step | slow (idx) | fast (idx) | Verdict |
|:---:|:---:|:---:|:---|
| start | 0 | 0 | dono head |
| move 1 | 1 | 2 | continue |
| move 2 | 2 | 1 | continue |
| move 3 | 3 | 3 | **meet!** → Phase 2 shuru |

**Phase 2** (`ptr = head = index 0`, `slow` meeting point `index 3` se aage badhta hai, dono +1):

| Step | ptr (idx) | slow (idx) | Verdict |
|:---:|:---:|:---:|:---|
| init | 0 | 3 | `ptr != slow` |
| move 1 | 1 | 1 *(index 3 tail se wraparound karke index 1 par)* | `ptr == slow == 1` → **entry = index 1 (value 2)** ✅ |

**Final answer:** node with value **`2`** — yehi cycle ka starting gate hai.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Hashing (visited set) | `O(n)` | `O(n)` | simple, extra memory |
| Fast & Slow, 2-phase | `O(n)` | `O(1)` | optimal — distance-proof magic |

---

## Gotchas 🪤

- **Phase 2 mein `slow` ko reset mat karo** — `slow` apni meeting-point wali jagah se hi aage badhta hai; sirf `ptr` naya `head` se shuru hota hai. Ulta karoge to jawaab galat aayega.
- **Dono phase 2 mein same speed (+1) chalte hain** — yeh LC 141 se alag hai jahan fast +2 tha. Phase 2 ka pura proof isi equal-speed pe tika hai.
- **`ptr == slow` (reference check), values nahi** — same duplicate value wale do alag nodes ko match mat samajhna.
- **No-cycle case** — agar `fast`/`fast.next` `null` ho jaaye, seedha `null` return karo; phase 2 kabhi trigger nahi hoga.
- **Single-node self-loop** — `head.next == head` ho to `slow==fast` turant step 1 pe hi ho jaata hai; phase 2 mein `ptr` bhi turant `head` par match kar jaata hai (`a = 0` case).
- **Yeh LC 141 ka direct extension hai** — Phase 1 bilkul same hai; sirf Phase 2 ka "reset + equal speed" naya concept hai.
