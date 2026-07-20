# Middle of the Linked List — LeetCode 876

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/fast-and-slow/middle-of-the-linkedlist.html)**
> _(tortoise vs hare, live index readout, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** Fast & Slow Pointers  ·  **Tags:** Linked List, Two Pointers

---

## Problem

Ek singly linked list ka `head` diya hai. **Middle node** return karo. Agar do middle nodes hain (even length), to **doosra** middle node return karo.

```
Input:  1 → 2 → 3 → 4 → 5
Output: 3 → 4 → 5   (node value 3 return hota hai)

Input:  1 → 2 → 3 → 4 → 5 → 6
Output: 4 → 5 → 6   (node value 4 return hota hai — second middle)
```

---

## The story (yaad rakhne ke liye) 🐢🐇🏁

List ki length pehle se pata nahi (ek baar poori list traverse kiye bina), toh seedha `length/2` nahi nikaal sakte bina do passes kiye. Yahan wahi tortoise-hare trick kaam aati hai jo cycle detection mein use hoti hai — bas is baar list mein loop nahi hai, seedha ek **finish line** (`null`) hai.

Kachhua (`slow`) aur khargosh (`fast`) dono **saath saath start line** (`head`) se daudte hain. Khargosh hamesha **do guna speed** se chalta hai. Jab khargosh **finish line** (list ka end) tak pahunch jaaye, kachhua ne theek **aadha raasta** hi tay kiya hoga — kyunki uski speed hamesha khargosh ki aadhi thi. Ek hi pass mein, koi extra length-count ki zaroorat nahi!

---

## Approach 1 — Brute force (Two-pass)

Pehle poori list traverse karke `length` count karo, phir `length/2` steps chal ke middle tak pahuncho.

```java
class Solution {
    public ListNode middleNode(ListNode head) {
        int len = 0;
        for (ListNode curr = head; curr != null; curr = curr.next) len++;

        ListNode curr = head;
        for (int i = 0; i < len / 2; i++) curr = curr.next;
        return curr;
    }
}
```

**Time:** `O(n)` — do passes, lekin dono linear hain (`2n` = `O(n)`)  
**Space:** `O(1)`

Sahi jawaab deta hai, lekin list ko **do baar** traverse karta hai. Agar list streaming ho ya ek hi pass allowed ho, yeh approach fail ho jaayegi.

---

## Approach 2 — Optimal (Fast & Slow Pointers, single pass)

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode middleNode(ListNode head) {
        ListNode slow = head, fast = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;          // +1
            fast = fast.next.next;     // +2
        }

        return slow;    // fast ran out first -> slow sits at the middle
    }
}
```

**Time:** `O(n)` — sirf ek pass  
**Space:** `O(1)`

---

## Dry run

**Odd length** — `1 → 2 → 3 → 4 → 5` (indices `0..4`)

| Step | slow (idx) | fast (idx) | Note |
|:---:|:---:|:---:|:---|
| start | 0 | 0 | dono head |
| move 1 | 1 | 2 | `fast.next != null`, continue |
| move 2 | 2 | 4 | `fast.next == null` (index 4 = last) → **loop stop** |

**Result:** `slow = index 2` → value **`3`** ✅

**Even length** — `1 → 2 → 3 → 4 → 5 → 6` (indices `0..5`)

| Step | slow (idx) | fast (idx) | Note |
|:---:|:---:|:---:|:---|
| start | 0 | 0 | dono head |
| move 1 | 1 | 2 | continue |
| move 2 | 2 | 4 | continue |
| move 3 | 3 | — | `fast (idx 4).next` (idx 5) exists lekin uska `.next` = `null` → loop condition `fast.next != null` false, **stop** |

**Result:** `slow = index 3` → value **`4`** ✅ (second middle, jaisa problem maangta hai)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Two-pass (count then walk) | `O(n)` | `O(1)` | list do baar traverse hoti hai |
| Fast & Slow Pointers | `O(n)` | `O(1)` | optimal — sirf ek pass |

---

## Gotchas 🪤

- **Loop condition dono check kare** — `fast != null && fast.next != null`. Sirf `fast != null` likhoge to `fast.next.next` pe NPE aa sakta hai jab `fast.next` khud `null` ho.
- **Even-length ke liye "second middle" chahiye** — is exact loop condition (`fast != null && fast.next != null`) se automatically second middle milta hai. Agar condition `fast.next != null && fast.next.next != null` kar do, to **first** middle milega — dono variants interview mein pooche ja sakte hain, condition yaad rakho.
- **Single node** (`[1]`) → loop turant `fast.next == null` ki wajah se skip ho jaata hai, `slow` head par hi reh jaata hai, correct.
- **`slow`/`fast` reference hote hain, index nahi** — Java mein actual `ListNode` pointers hote hain; upar diagram/dry-run mein indices sirf samajhne ke liye use kiye hain.
- **Koi extra length-counting ki zaroorat nahi** — yehi is approach ka poora fayda hai: ek hi pass, koi pre-computation nahi.
