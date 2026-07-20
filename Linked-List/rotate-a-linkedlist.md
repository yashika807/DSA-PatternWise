# 61. Rotate List 🔗

> **Pattern:** Linked List → *Circular-link trick (make a ring, then cut it at the right spot)*
> **Difficulty:** Medium · **Platform:** [LeetCode 61](https://leetcode.com/problems/rotate-list/)
> **Visualizer:** [rotate-a-linkedlist.html](https://yashika807.github.io/DSA-PatternWise/Linked-List/rotate-a-linkedlist.html) — step-by-step debugger with live pointers (`←` `→` `Space` `R`)

---

## 🧩 Problem

Ek singly linked list aur ek integer `k` di hai. List ko **`k` positions right** rotate karna hai — matlab last `k` nodes list ke **aage** aa jaayenge, baaki apni relative order mein unke peeche.

```
Input:  1 → 2 → 3 → 4 → 5,  k = 2
Output: 4 → 5 → 1 → 2 → 3
        └─┘   └───────┘
      last k    baaki list (order same)
```

---

## ❌ Galat approaches (pehle inhe reject karo)

### WA1 — `k` ko list ki length se **modulo karna bhool jaana**
`k` list ki length se **bada** ho sakta hai (ya bahut bada, jaise `4000000000`). Modulo kiye bina seedha formula use karna galat position de deta hai.

**Counterexample — `1→2→3`, `k=4` (`n=3`, to sahi effective `k = 4 % 3 = 1`):**
- Sahi answer: `k=1` rotate → `[3,1,2]`.
- Agar `k=4` seedha formula mein daalo (`hops = n - k - 1 = 3 - 4 - 1 = -2`), loop `for (i=0;i<-2;i++)` **0 baar** chalta hai (negative condition kabhi true nahi) → `newTail` galti se `head` (node `1`) hi reh jaata hai.
- Result: `newHead = 2`, chain `2 → 3 → 1 → null` ban jaata hai — **`[2,3,1]`** ❌, jabki sahi tha `[3,1,2]`.
- **Fix:** hamesha `k = k % n` sabse pehle karo — isse `k` hamesha `[0, n-1]` range mein aa jaata hai.

### WA2 — Circular link banane ke baad **todna bhool jaana**
List ko temporarily circular banate ho (`tail.next = head`), lekin naya cut point set karne ke baad `newTail.next = null` likhna bhool jaate ho.

**Counterexample — `1→2→3→4→5`, `k=2`:**
- Circular bana di: `5.next = 1`.
- `newTail` (node `3`) sahi se mil gaya, `newHead` (node `4`) bhi sahi mila.
- Lekin `newTail.next = null` **nahi likha** → list abhi bhi ring hai: `4 → 5 → 1 → 2 → 3 → 4 → 5 → 1 → ...` **infinite cycle**, kabhi `null` pe khatam nahi hoti.
- Koi bhi traversal (print, length count, ya LeetCode ka checker khud) is list pe **hang** ho jaayega.
- **Fix:** naya cut point set karne ke turant baad `newTail.next = null` explicitly likho.

### WA3 — Length count karte waqt `tail` ka reference **kho dena**
Length nikaalne ke liye loop condition galat likh di — `tail != null` (chalta rehta hai jab tak `tail` khud `null` na ho) instead of `tail.next != null`.

```java
int n = 0;
ListNode tail = head;
while (tail != null) {      // ❌ galat condition
    tail = tail.next;
    n++;
}
// yahan tail hamesha null hai!
tail.next = head;           // NullPointerException 💥
```

**Counterexample — `1→2→3→4→5`:**
- `n` toh sahi count ho jaata hai (`5`), **lekin** loop ke end mein `tail` khud `null` ban chuka hota hai (list ke aakhri node ke aage ek extra step chal gaya).
- Ab circular link banane ke liye `tail.next = head` likhoge to **`null.next` pe NullPointerException**.
- **Fix:** loop condition `tail.next != null` honi chahiye — isse loop **aakhri actual node** (`5`) par hi रुकता hai, ek step aage nahi jaata.

---

## 💡 Intuition — ring banao, sahi jagah kaato

Circular linked list ka trick hai — list ko **temporarily ek ring** bana do, phir sahi jagah **cut** kar do:

1. **Length `n` nikaalo aur `tail` (aakhri node) dhoondo** — ek pass mein.
2. **`k = k % n`** — agar `k` list se bada hai to usse effective range `[0, n-1]` mein le aao. Agar `k == 0` ho jaaye (ya shuru se hi tha), koi rotation zaroorat nahi — seedha `head` return.
3. **Ring banao:** `tail.next = head` — ab list ek circle hai.
4. **Naya cut point dhoondo:** head se `n - k - 1` steps aage chalo — yeh **naya tail** hai (last node of the "first part" jo ab peeche jaayegi). Iska agla node (`newTail.next`) **naya head** hai.
5. **Ring todo:** `newTail.next = null`.

Bas! Koi extra reversal nahi, koi extra list nahi — sirf ek circle banao aur usse sahi jagah kaato.

---

## 🔍 Dry run — `1 → 2 → 3 → 4 → 5`, `k = 2`

| Step | Action | Result |
|---|---|---|
| 1 | Length scan | `n = 5`, `tail = node 5` |
| 2 | `k = k % n` | `k = 2 % 5 = 2` (`≠ 0`, rotation zaroori hai) |
| 3 | `tail.next = head` | Ring: `1→2→3→4→5→1→2→...` |
| 4 | `hops = n - k - 1 = 5 - 2 - 1 = 2`, head se 2 steps aage | `newTail = node 3` (`1 → 2 → 3`) |
| 5 | `newHead = newTail.next` | `newHead = node 4` |
| 6 | `newTail.next = null` | Ring toot gayi node `3` ke baad |

**Final:** `4 → 5 → 1 → 2 → 3 → null` ✅

---

## ✅ Solution (Java)

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
    public ListNode rotateRight(ListNode head, int k) {
        if (head == null || head.next == null || k == 0) return head;

        // 1) length nikaalo aur tail dhoondo
        int n = 1;
        ListNode tail = head;
        while (tail.next != null) {
            tail = tail.next;
            n++;
        }

        k = k % n;              // 2) effective k, [0, n-1] range mein
        if (k == 0) return head;

        tail.next = head;       // 3) ring banao

        // 4) naya cut point: head se (n - k - 1) steps aage
        ListNode newTail = head;
        for (int i = 0; i < n - k - 1; i++) {
            newTail = newTail.next;
        }
        ListNode newHead = newTail.next;

        newTail.next = null;    // 5) ring todo

        return newHead;
    }
}
```

## ✅ Solution (Python)

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next

class Solution:
    def rotateRight(self, head, k):
        if head is None or head.next is None or k == 0:
            return head

        # 1) length nikaalo aur tail dhoondo
        n = 1
        tail = head
        while tail.next is not None:
            tail = tail.next
            n += 1

        k = k % n                # 2) effective k
        if k == 0:
            return head

        tail.next = head          # 3) ring banao

        # 4) naya cut point: head se (n - k - 1) steps aage
        newTail = head
        for _ in range(n - k - 1):
            newTail = newTail.next
        newHead = newTail.next

        newTail.next = None       # 5) ring todo

        return newHead
```

---

## ⏱️ Complexity

| | |
|---|---|
| **Time** | `O(n)` — ek pass length/tail ke liye, ek pass naya cut-point dhoondne ke liye. Dono milake bhi linear. |
| **Space** | `O(1)` — sirf `tail`, `newTail`, `newHead` pointers, koi extra structure nahi. |

---

## 🧪 Edge cases

- **Empty list** (`head == null`) → seedha `head` (`null`) return.
- **Single node** (`[1]`) → `head.next == null` check se turant `head` return, kisi bhi `k` ke liye.
- **`k == 0`** → koi rotation nahi, seedha `head` return (ring banane ki zaroorat hi nahi).
- **`k` list ki length ka exact multiple** (e.g. `n=5, k=5` ya `k=10`) → `k % n == 0` → list waisi hi rehti hai.
- **`k > n`** (bahut bada) → `k % n` se automatically sahi range mein aa jaata hai (WA1 dekho).
- **`n = 2`** → sabse chhota case jahan real rotation dikhti hai, formula waise hi kaam karta hai.

---

## 🔁 Revision recap (10-second re-derive)

1. `head == null`, `head.next == null`, ya `k == 0` → seedha `head` return (fast exits).
2. Ek pass mein length `n` aur `tail` dhoondo (`while (tail.next != null)`).
3. `k = k % n` — agar ab `k == 0` ho gaya, `head` return.
4. `tail.next = head` — ring banao.
5. Head se `n - k - 1` steps aage jaake `newTail` nikaalo, `newHead = newTail.next`.
6. `newTail.next = null` — ring todo. **(WA2 yehi bhoolta hai — hamesha last step verify karo.)**
7. `newHead` return karo, `head` nahi.
