# Linked List Cycle — LeetCode 141

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/fast-and-slow/linkedlist-cycle.html)**
> _(tortoise vs hare, live index readout, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** Fast & Slow Pointers (Floyd's Cycle Detection)  ·  **Tags:** Linked List, Two Pointers, Hashing

---

## Problem

Ek singly linked list ka `head` diya hai. Pata karo ki list mein **cycle** hai ya nahi — yaani koi node aisa hai jiska `next` kisi pichhle node ko point kar raha ho, jisse traversal kabhi `null` par khatam na ho.

```
Input:  3 → 2 → 0 → -4 → (wapas 2 se jud jaata hai)
Output: true

Input:  1 → 2 → null
Output: false
```

Values ko chhoo ke pata nahi chalega — sirf `next` pointers follow karke dekhna hoga.

---

## The story (yaad rakhne ke liye) 🐢🐇

Socho ek **gol race track** (ya seedha track jo kabhi curve leke loop ban jaata hai). Ek **kachhua** (`slow`) har round mein 1 kadam chalta hai, ek **khargosh** (`fast`) har round mein 2 kadam. Dono same starting point (`head`) se chalte hain.

- Agar track **seedha rasta hai jo kahin khatam ho jaata hai** (no cycle), khargosh bas pehle finish line (`null`) pe pahunch jaayega — race khatam, dono kabhi nahi milte.
- Agar track mein **loop hai**, khargosh loop ke andar chakkar kaatta rehta hai aur — kyunki wo kachhue se tez hai — ek na ek din kachhue ko **peeche se lap maar ke pakad legi**. Jis pal woh donon same node par khade honge, cycle confirm ho jaata hai.

Yeh guarantee hai: agar cycle exist karta hai, to speed-gap (fast ekstra +1 step/round) ki wajah se dono zaroor milenge — kabhi cross nahi karenge bina milen (jaise ek circular track pe tez runner ek din slow runner ko zaroor lap karega).

---

## Approach 1 — Brute force (Hashing)

Har node visit karte waqt use ek `HashSet` mein daal do. Agar koi node dobara mile (already set mein hai), matlab cycle hai.

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        Set<ListNode> seen = new HashSet<>();
        ListNode curr = head;
        while (curr != null) {
            if (seen.contains(curr)) return true;
            seen.add(curr);
            curr = curr.next;
        }
        return false;
    }
}
```

**Time:** `O(n)` — har node ek baar visit  
**Space:** `O(n)` — set mein saare nodes store ho sakte hain

Kaam to karta hai, lekin extra `O(n)` space use hoti hai. Interviewer turant "constant space mein kar sakte ho?" poochega.

---

## Approach 2 — Optimal (Fast & Slow Pointers)

Floyd's Cycle Detection — do pointers, dono `head` se start, `slow` +1 aur `fast` +2. Agar kabhi `slow == fast` mile, cycle hai. Agar `fast` (ya `fast.next`) `null` ho jaaye, cycle nahi hai.

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
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) return false;   // 0/1 node -> no cycle

        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;          // +1 step
            fast = fast.next.next;     // +2 steps

            if (slow == fast) {        // hare lapped the tortoise
                return true;
            }
        }
        return false;                  // hare fell off the end -> no cycle
    }
}
```

**Time:** `O(n)` — worst case fast pointer poori list + cycle ek do baar hi ghoomta hai pakadne se pehle  
**Space:** `O(1)` — bas do pointers, koi extra structure nahi

---

## Dry run

Input: `3 → 2 → 0 → -4 → (tail se index 1 = value 2 tak wapas)`  (indices: `0:3, 1:2, 2:0, 3:-4`)

| Step | slow (idx) | fast (idx) | Verdict |
|:---:|:---:|:---:|:---|
| start | 0 | 0 | dono head par |
| move 1 | 1 | 2 | `1 != 2`, continue |
| move 2 | 2 | 1 | `2 != 1`, continue |
| move 3 | 3 | 3 | `slow == fast` → **cycle mila, return true** ✅ |

Fast pointer index 3 (`-4`) par pahunch ke wrap hoke index 1 (`2`) pe jaata hai, kyunki cycle tail se index 1 par jud-ta hai — isi wraparound ki wajah se fast, slow ko "lap" kar paata hai.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Hashing (visited set) | `O(n)` | `O(n)` | simple, but extra memory |
| Fast & Slow Pointers | `O(n)` | `O(1)` | optimal — no extra structure |

---

## Gotchas 🪤

- **Guard clause pehle** — `head == null` ya `head.next == null` ho to loop mein ghuse bina hi `false` return karo, warna `fast.next.next` pe NPE aa sakta hai.
- **Loop condition dono check kare** — `fast != null && fast.next != null`. Sirf `fast != null` likhoge to `fast.next.next` par crash ho sakta hai jab `fast.next` khud `null` ho.
- **`slow == fast` reference comparison hai, value nahi** — Java mein `ListNode` object identity check hoti hai (`==`), values equal hone se kuch nahi hota; do alag nodes ka value same ho sakta hai bina cycle ke.
- **Single node self-loop bhi valid cycle hai** — agar `head.next == head`, wahi ek step mein `slow == fast` ban jaata hai (dono immediately same node).
- **Yeh sirf boolean detect karta hai, entry point nahi deta** — cycle ka *starting node* chahiye ho to LeetCode 142 (Linked List Cycle II) dekho, jo isi approach ko ek extra phase ke saath extend karta hai.
