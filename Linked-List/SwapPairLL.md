# 24. Swap Nodes in Pairs 🔗

> **Pattern:** Linked List → *Reverse in groups of `k`* (yahan `k = 2`)
> **Difficulty:** Medium · **Platform:** [LeetCode 24](https://leetcode.com/problems/swap-nodes-in-pairs/)
> **Visualizer:** [`swapPairs-visualizer.html`]
(./swapPairs-visualizer.html) — step-by-step debugger with live pointers (`←` `→` `Space` `R`)

---

## 🧩 Problem

Ek singly linked list di hai. Har **do adjacent nodes** ko aapas mein swap karna hai, aur nayi list ka head return karna hai.

> ⚠️ **Constraint:** Node ki **values ko modify nahi kar sakte** — sirf **pointers (`next`) badalne hain**. (Yeh line poore problem ka twist hai, isko ignore mat karna.)

```
Input:  1 → 2 → 3 → 4
Output: 2 → 1 → 4 → 3

Input:  1 → 2 → 3          (odd length)
Output: 2 → 1 → 3          ← akela bacha node jaisa-ka-waisa
```

---

## ❌ Galat approaches (pehle inhe reject karo)

### WA1 — "Bas values swap kar do, pointers chhod do"
Sochne mein simple: `1 ↔ 2` values swap, `3 ↔ 4` values swap, ho gaya.

**Refutation:** Problem **explicitly mana karti hai** values modify karna. Interview mein yeh instant reject hai, aur follow-up (LC 25, group size `k`) mein jab `k = 5` ho to value-shuffle bilkul clumsy ho jaata hai. Pointer manipulation seekhna hi is problem ka asli maqsad hai — value-swap us seekh ko bypass kar deta hai.

### WA2 — "Har pair ko swap karta ja, previous group bhulke"
Tum har jodi ko independently palat dete ho, lekin **pichhle swapped pair ki tail ko agle swapped pair ke head se jodna bhool jaate ho.**

**Counterexample — `1 → 2 → 3 → 4`:**
- Pair `(1,2)` swap → `2 → 1`
- Pair `(3,4)` swap → `4 → 3`
- Par tumne `1.next` ko `4` se **connect nahi kiya**.
- `1.next` abhi bhi purana `3` point kar raha hai → list ban gayi: `2 → 1 → 3 → 4` ❌ (galat) ya phir dangling.

Isiliye code mein **`prevleft`** chahiye — wo pichhle group ki tail yaad rakhta hai, taaki `prevleft.next = right` se cross-link jud sake.

### WA3 — "Poori list reverse kar do" / "Group check kiye bina 2-2 palatte jao"
Kabhi log poori list reverse karne lagte hain, ya bina check kiye har baar 2 nodes uthaate hain.

**Counterexample A (full reverse) — `1 → 2 → 3 → 4`:**
Full reverse deta hai `4 → 3 → 2 → 1` ❌ — humein sirf **adjacent** pairs swap karne the, poori list nahi.

**Counterexample B (group check missing) — `1 → 2 → 3`:**
Agar last mein tum bina check kiye 3rd node ke saath "2nd node of pair" dhoondhne jaao, to `null.next` pe **NullPointerException** ya galat link ban jaata hai. Isiliye pehle **poora group exist karta hai ya nahi** check karna zaroori hai (`if (right != null)`).

---

## 💡 Intuition — generalized "reverse in groups of `size`"

Yeh solution swap-pairs ko ek **general template** ki tarah dekhta hai:
> "List ko `size` ke groups mein baanto, har poore group ko in-place reverse karo, aur groups ko wapas stitch karo."

`size = 2` daalo → **swap pairs**. `size = k` daalo → **LC 25 (reverse in k-group)**. Ek hi code, do problems. Yahi is approach ka payoff hai.

Har iteration mein 3 kaam hote hain:

1. **Group verify karo** — `left` se `size-1` step aage chal ke `right` nikaalo. Agar `right == null`, group poora nahi bana → us bache hisse ko chhod do.
2. **In-place reverse** — `reverse(left, size)` group ko palat deta hai. Ab **`right` naya head** hai, **`left` nayi tail** (aur `left.next` temporarily `null`, isliye baaki list detach ho jaati hai — usse `next` mein pehle hi bacha liya).
3. **Stitch karo:**
   - `prevleft.next = right` → pichhle group ki tail ko naye head se jodo (ya pehla group ho to `res = right`).
   - `left.next = next` → nayi tail ko aage wale part se jodo.
   - `prevleft = left`, `left = next` → agle group ke liye aage badho.

Do "connection points" hamesha yaad rakhne hote hain: **peechhe** (`prevleft`) aur **aage** (`next`). Bas.

---

## 🔍 Dry run — `1 → 2 → 3 → 4 → 5 → 6 → 7`

| Iter | left | right | next | reverse baad | stitch (peechhe) | `left.next = next` | List after iter |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| 1 | 1 | 2 | 3 | `2 → 1` | `res = 2` (pehla group) | `1.next = 3` | **2 → 1** → 3 → 4 → 5 → 6 → 7 |
| 2 | 3 | 4 | 5 | `4 → 3` | `1.next = 4` | `3.next = 5` | 2 → 1 → **4 → 3** → 5 → 6 → 7 |
| 3 | 5 | 6 | 7 | `6 → 5` | `3.next = 6` | `5.next = 7` | 2 → 1 → 4 → 3 → **6 → 5** → 7 |
| 4 | 7 | **null** | — | — | else-branch: `5.next = 7` → `break` | — | 2 → 1 → 4 → 3 → 6 → 5 → **7** |

**Final:** `2 → 1 → 4 → 3 → 6 → 5 → 7 → null` ✅

Iter 4 dekho — node `7` akela bacha, for-loop mein `right` `null` ho gaya, isliye **else-branch** ne use jaisa-ka-waisa `prevleft (5)` ke aage jod ke loop tod diya. Yehi odd-length ka clean handling hai.

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
    public ListNode swapPairs(ListNode head) {
        int size = 2;                       // size = k karo to yeh LC 25 ban jaata hai
        if (head == null) return head;

        ListNode left = head;
        ListNode prevleft = null;           // pichhle group ki tail
        ListNode res = null;                // final answer ka head
        ListNode right;

        while (true) {
            // 1) group verify: left se size-1 step aage
            right = left;
            for (int i = 0; i < size - 1; i++) {
                if (right == null) break;
                right = right.next;
            }

            if (right != null) {            // poora group mila
                ListNode next = right.next; // group ke baad wala part bacha lo
                reverse(left, size);        // 2) in-place reverse -> right=head, left=tail

                // 3a) peechhe stitch
                if (prevleft != null) {
                    prevleft.next = right;
                } else {
                    res = right;            // pehla group -> res set
                }
                left.next = next;           // 3b) aage stitch
                prevleft = left;            // agle group ke liye tail update
                left = next;                // aage badho
            } else {                        // group poora nahi bana (0/1 node)
                if (prevleft != null) {
                    prevleft.next = left;   // bacha node jaisa hai waisa jod do
                } else {
                    res = left;             // list mein 0/1 node hi tha
                }
                break;
            }
        }
        return res;
    }

    // left se `size` nodes ko in-place reverse karta hai
    private void reverse(ListNode head, int size) {
        ListNode prev = null;
        ListNode curr = head;
        for (int i = 0; i < size && curr != null; i++) {
            ListNode nextNode = curr.next;
            curr.next = prev;
            prev = curr;
            curr = nextNode;
        }
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
    def swapPairs(self, head):
        size = 2                       # size = k -> LC 25
        if head is None:
            return head

        left = head
        prevleft = None                # pichhle group ki tail
        res = None                     # final head

        while True:
            # 1) group verify
            right = left
            for _ in range(size - 1):
                if right is None:
                    break
                right = right.next

            if right is not None:              # poora group mila
                nxt = right.next               # aage wala part bacha lo
                self.reverse(left, size)       # 2) in-place reverse

                # 3a) peechhe stitch
                if prevleft is not None:
                    prevleft.next = right
                else:
                    res = right                # pehla group
                left.next = nxt                # 3b) aage stitch
                prevleft = left
                left = nxt
            else:                              # 0/1 node bacha
                if prevleft is not None:
                    prevleft.next = left
                else:
                    res = left
                break

        return res

    def reverse(self, head, size):
        prev, curr = None, head
        for _ in range(size):
            if curr is None:
                break
            nxt = curr.next
            curr.next = prev
            prev = curr
            curr = nxt
```

---

## ⏱️ Complexity

| | |
|---|---|
| **Time** | `O(n)` — har node ko constant baar touch karte hain (ek baar right-scan, ek baar reverse). `size` constant hai. General `k` ke liye bhi `O(n)`. |
| **Space** | `O(1)` — sirf pointers, koi recursion stack ya extra structure nahi. |

---

## 🧪 Edge cases

- **Empty list** (`head == null`) → seedha `head` return.
- **Single node** (`[1]`) → group nahi banta, else-branch `res = left`, output `1`.
- **Exactly two nodes** (`[1,2]`) → ek swap, `2 → 1`.
- **Odd length** (`[1,2,3]`) → last node akela bacha, `prevleft.next = left` se as-is jud jaata hai (`2 → 1 → 3`).
- **Even length** → clean, har node swap ho jaata hai.

---

## 🔁 Revision recap (10-second re-derive)

1. Har iteration: `left` fix karo, `right` ko `size-1` aage bhejo.
2. `right == null` → group adhoora → bacha node as-is jodo → `break`.
3. Warna: `next` bacha lo → `reverse(left, size)` → ab `right`=head, `left`=tail.
4. Stitch: `prevleft.next = right` (ya `res = right`), phir `left.next = next`.
5. Aage badho: `prevleft = left`, `left = next`.
6. **Yaad rakho:** do connection points — **peechhe** (`prevleft`) aur **aage** (`next`). `size = k` → yehi LC 25 ho jaata hai.

---

> 🎯 Interactive dry-run yahan dekho: [`swapPairs-visualizer.html`](./swapPairs-visualizer.html)
