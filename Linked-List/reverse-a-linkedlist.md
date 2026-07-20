# 206. Reverse Linked List 🔗

> **Pattern:** Linked List → *In-place pointer reversal* (yeh poore "reverse-in-groups" family ki foundation hai)
> **Difficulty:** Easy · **Platform:** [LeetCode 206](https://leetcode.com/problems/reverse-linked-list/)
> **Visualizer:** [reverse-a-linkedlist.html](https://yashika807.github.io/DSA-PatternWise/Linked-List/reverse-a-linkedlist.html) — step-by-step debugger with live pointers (`←` `→` `Space` `R`)

---

## 🧩 Problem

Ek singly linked list di hai. Puri list ko reverse karke naya head return karna hai — **in-place**, O(1) extra space (koi naya node ya array nahi banana).

```
Input:  1 → 2 → 3 → 4 → 5
Output: 5 → 4 → 3 → 2 → 1
```

---

## ❌ Galat approaches (pehle inhe reject karo)

### WA1 — "`curr.next = prev` pehle kar do, `next` bachana baad mein"
Order galat rakh diya:

```java
curr.next = prev;   // pehle link tod diya
next = curr.next;   // ❌ ab yeh hamesha prev hi milega, asli next kho gaya
curr = next;
```

**Counterexample — `1 → 2 → 3`:**
- `curr = 1`, `prev = null`. `curr.next = prev` → `1.next = null`.
- Ab `next = curr.next` karoge to `next = null` milega — jabki asli next node `2` tha, wo **permanently lost** ho gaya (koi reference nahi bacha).
- `curr = next` → `curr = null` → loop turant ruk jaata hai. Sirf node `1` process hua, `2 → 3` poori list **memory se udd gayi**.

Isiliye `next` ko **link todne se pehle hi** save karna zaroori hai: `next = curr.next` → phir `curr.next = prev`.

### WA2 — Recursive reversal mein `head.next = null` bhool jaana
Standard recursive idea sahi hai, lekin ek line miss karna cycle bana deta hai:

```java
ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverseList(head.next);
    head.next.next = head;
    // ❌ BHOOL GAYE: head.next = null;
    return newHead;
}
```

**Counterexample — `1 → 2 → 3`:**
- Recursion andar jaati hai: `reverseList(3)` → base case, `newHead = 3`.
- Wapas `head = 2`: `head.next.next = head` → `3.next = 2`. (`2.next` abhi bhi purana `3` hi hai kyunki humne null nahi kiya.)
- Wapas `head = 1`: `head.next.next = head` → `2.next = 1`. (`1.next` abhi bhi purana `2` hai.)
- Final links: `3 → 2`, `2 → 1`, aur `1 → 2` (bhoola hua purana pointer) → **`1` aur `2` ke beech cycle ban gaya** (`1 → 2 → 1 → 2 → ...`), list kabhi `null` pe terminate nahi hoti.

`head.next = null` add karna zaroori hai taaki purana forward pointer explicitly tootta jaaye.

### WA3 — "Values ko array mein daal ke ends se swap karo" (value-swap, extra space)
Sochte ho: values ek array/list mein utaar lo, two-pointer se ends se swap karke wapas node.val mein daal do.

**Refutation:** Ismein `O(n)` extra space lagta hai (array), jabki asli pointer-reversal `O(1)` space mein hoti hai — interview follow-up "can you do it without extra space?" seedha fail. Aur agar array bhi na banao, sirf singly linked list pe two-pointer se end se swap karna ho (bina backward traversal ke), to tail tak har baar naye sirey se walk karna padega → `O(n²)` time. Yeh problem **structure** (`next` pointers) reverse karne ke baare mein hai, values copy karne ke baare mein nahi — aur yehi seekh aage LC 92 / LC 25 mein zaroori banti hai.

---

## 💡 Intuition

Teen pointers rakho — `prev`, `curr`, `next` — aur ek-ek node ko "peeche ki taraf" mod dete jao:

1. `curr.next` ko change karne se **pehle** `next = curr.next` mein save karo (WA1 yahi bhoolta hai).
2. `curr.next = prev` — is node ka arrow ab peeche point karta hai.
3. `prev = curr`, `curr = next` — dono aage khisko.
4. Jab `curr == null` ho jaaye, `prev` hi naya head hai (last processed node).

Har node sirf **ek baar** touch hota hai, aur har baar sirf pointer ki direction badalti hai — koi naya memory allocate nahi hoti. Yehi teen-pointer waltz aage **LC 92 (reverse sublist)** aur **LC 25 (reverse k-group)** mein bhi core building block ban jaata hai — bas kahan se start/stop karna hai wo badalta hai.

---

## 🔍 Dry run — `1 → 2 → 3 → 4 → 5`

| Iter | prev (start) | curr | next = curr.next | curr.next = prev | prev = curr | curr = next |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | null | 1 | 2 | `1 → null` | prev=1 | curr=2 |
| 2 | 1 | 2 | 3 | `2 → 1` | prev=2 | curr=3 |
| 3 | 2 | 3 | 4 | `3 → 2` | prev=3 | curr=4 |
| 4 | 3 | 4 | 5 | `4 → 3` | prev=4 | curr=5 |
| 5 | 4 | 5 | null | `5 → 4` | prev=5 | curr=null |

Loop khatam (`curr == null`). **Return `prev` = node `5`.**

**Final:** `5 → 4 → 3 → 2 → 1 → null` ✅

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
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode curr = head;

        while (curr != null) {
            ListNode next = curr.next;  // 1) aage wala bacha lo
            curr.next = prev;           // 2) arrow palato
            prev = curr;                // 3) prev aage badhao
            curr = next;                // 4) curr aage badhao
        }

        return prev;   // naya head
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
    def reverseList(self, head):
        prev = None
        curr = head

        while curr is not None:
            nxt = curr.next     # 1) aage wala bacha lo
            curr.next = prev    # 2) arrow palato
            prev = curr         # 3) prev aage badhao
            curr = nxt          # 4) curr aage badhao

        return prev             # naya head
```

---

## ⏱️ Complexity

| | |
|---|---|
| **Time** | `O(n)` — har node ek hi baar visit hota hai. |
| **Space** | `O(1)` iterative version mein (sirf 3 pointers). Recursive version `O(n)` call-stack space leti hai. |

---

## 🧪 Edge cases

- **Empty list** (`head == null`) → loop chalta hi nahi, `prev = null` return hota hai. ✅
- **Single node** (`[1]`) → `curr = 1`, `next = null`, `1.next = null` (already), `prev = 1` → same node return. ✅
- **Two nodes** (`[1,2]`) → `2 → 1 → null`.
- **Already palindrome-ish / any order** → algorithm order-agnostic hai, sirf links palatta hai, values se farak nahi padta.

---

## 🔁 Revision recap (10-second re-derive)

1. Teen pointers: `prev = null`, `curr = head`.
2. Loop jab tak `curr != null`.
3. **Pehle** `next = curr.next` bacha lo (link todne se pehle) — yehi WA1 ka fix hai.
4. `curr.next = prev` → arrow palato.
5. `prev = curr`, `curr = next` → dono aage khisko.
6. **Yaad rakho:** loop khatam hote hi `prev` hi naya head hai — `curr`, `head` nahi.
