# 25. Reverse Nodes in k-Group 🔗

> **Pattern:** Linked List → *Reverse in groups of `k`* (yeh wahi generalized template hai, `size = 2` par jo **Swap Pairs** ban jaata hai)
> **Difficulty:** Hard · **Platform:** [LeetCode 25](https://leetcode.com/problems/reverse-nodes-in-k-group/)
> **Visualizer:** [reverse-k-element-sublist.html](https://yashika807.github.io/DSA-PatternWise/Linked-List/reverse-k-element-sublist.html) — step-by-step debugger with live pointers (`←` `→` `Space` `R`)

---

## 🧩 Problem

Ek singly linked list aur ek integer `k` di hai. List ko `k`-size ke groups mein todo aur **har poore group** ko individually reverse karo. Agar **aakhri group poora `k` size ka nahi banta**, use **jaisa hai waisa hi chhod do** (reverse mat karo).

```
Input:  1 → 2 → 3 → 4 → 5 → 6 → 7,  k = 3
Output: 3 → 2 → 1 → 6 → 5 → 4 → 7
        └───────┘   └───────┘   └── akela bacha, k=3 poora nahi banta → as-is
```

> 🔗 Yeh usi "reverse in groups of `size`" template ka general version hai jo iss folder mein **Swap Pairs (LC 24)** mein `size = 2` fix karke dekha tha. Wahi logic, bas ab `size = k` variable hai.

---

## ❌ Galat approaches (pehle inhe reject karo)

### WA1 — "Poori list reverse kar do, `k` ko ignore karke"
Sochte ho `k` sirf ek "hint" hai, poori list normal reverse kar do.

**Refutation:** Poori-list reverse `k`-boundaries ko totally todta hai. `1→2→3→4→5→6→7`, `k=3` ke liye full reverse dega `7→6→5→4→3→2→1` ❌ — jabki humein sirf **har 3-size group ke andar** reverse karna tha (`3→2→1→6→5→4→7`), poori list nahi.

### WA2 — "Aakhri incomplete group ko bhi reverse kar do"
Yahi is problem ka **asli twist** hai jo Swap Pairs se harder banata hai — bahut log bhool jaate hain ki last leftover group (jab `< k` nodes bache) reverse **nahi** karna.

**Counterexample — `1→2→3→4→5`, `k=3`:**
- Group 1 (`1,2,3`) poora hai → reverse → `3,2,1` ✅.
- Bacha `4,5` — sirf **2 nodes**, `k=3` se kam. Agar tum check kiye bina inhe bhi reverse kar do → `5,4` mil jaata hai.
- Final (galat): `3,2,1,5,4` ❌
- Sahi: `3,2,1,4,5` ✅ — leftover jaisa tha waisa hi rehna chahiye.
- Isiliye reverse karne se **pehle** `right != null` check zaroori hai (poora group scan karke confirm karo ki `k` nodes maujood hain).

### WA3 — "Pehle reverse kardo, phir dekho poora group tha ya nahi"
Order ulta kar diya: pehle blindly `k` nodes reverse kar diye, phir check kiya ki group poora tha ya nahi.

**Counterexample — `1→2→3→4→5`, `k=3`:**
- Bacha hua group `4,5` (sirf 2 nodes) ko bhi reverse-loop `k=3` baar chalane ki koshish ki: `curr=4` → reverse, `curr=5` → reverse, `curr=null` → agar `curr.next` access kiya bina null-check ke, **NullPointerException**.
- Chahe crash na bhi ho (agar loop mein `curr != null` guard hai), phir bhi tumhare paas ab **already-reversed leftover** (`5,4`) hai aur use "un-reverse" karna extra kaam aur error-prone hai.
- Sahi approach: **pehle** `right` ko `k-1` steps aage bhejke **verify** karo poora group hai ya nahi, **tabhi** `reverse()` call karo — kabhi reverse-then-check mat karo.

---

## 💡 Intuition — same template, `size = k`

Har iteration mein 3 kaam:

1. **Group verify karo** — `left` se `k-1` steps aage chal ke `right` nikaalo. Agar beech mein `right == null` ho jaaye, group poora nahi bana → us bache hisse ko **as-is chhod do** (yehi WA2/WA3 ka fix hai — verify pehle, reverse baad mein).
2. **In-place reverse** — poore `right != null` group ko standard 3-pointer (`prev/curr/next`) se reverse karo. Ab `right` naya head hai, `left` nayi tail.
3. **Stitch karo** — pichhle group ki tail (`prevleft`) ko naye head (`right`) se jodo (ya pehla group ho to `res = right`), aur nayi tail (`left`) ko aage bache list (`next`) se jodo.

`k = 2` daalo → yeh bilkul **Swap Pairs (LC 24)** ban jaata hai. Ek hi code, do (asal mein infinite `k`) problems.

---

## 🔍 Dry run — `1 → 2 → 3 → 4 → 5 → 6 → 7`, `k = 3`

| Iter | left | right (k-1=2 steps aage) | next | reverse baad | stitch (peechhe) | `left.next = next` | List after iter |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| 1 | 1 | 3 | 4 | `3 → 2 → 1` | `res = 3` (pehla group) | `1.next = 4` | **3 → 2 → 1** → 4 → 5 → 6 → 7 |
| 2 | 4 | 6 | 7 | `6 → 5 → 4` | `1.next = 6` | `4.next = 7` | 3 → 2 → 1 → **6 → 5 → 4** → 7 |
| 3 | 7 | **null** (sirf 1 node bacha, `k-1=2` steps mein hi null mil gaya) | — | — | else: `4.next = 7` → `break` | — | 3 → 2 → 1 → 6 → 5 → 4 → **7** |

**Final:** `3 → 2 → 1 → 6 → 5 → 4 → 7 → null` ✅

Iter 3 mein `right` scan karte hue hi `null` mil gaya (sirf node `7` bacha tha, `k=3` chahiye tha) — group incomplete, isliye **reverse call hi nahi hua**, `7` jaisa tha waisa hi jud gaya.

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
    public ListNode reverseKGroup(ListNode head, int k) {
        if (head == null) return head;

        ListNode left = head;
        ListNode prevleft = null;           // pichhle group ki tail
        ListNode res = null;                // final answer ka head
        ListNode right;

        while (true) {
            // 1) group verify: left se k-1 step aage
            right = left;
            for (int i = 0; i < k - 1; i++) {
                if (right == null) break;
                right = right.next;
            }

            if (right != null) {            // poora group mila
                ListNode next = right.next; // group ke baad wala part bacha lo
                reverse(left, k);           // 2) in-place reverse -> right=head, left=tail

                // 3a) peechhe stitch
                if (prevleft != null) {
                    prevleft.next = right;
                } else {
                    res = right;            // pehla group -> res set
                }
                left.next = next;           // 3b) aage stitch
                prevleft = left;            // agle group ke liye tail update
                left = next;                // aage badho
            } else {                        // group poora nahi bana (< k nodes)
                if (prevleft != null) {
                    prevleft.next = left;   // bache nodes jaisa hai waisa jod do
                } else {
                    res = left;             // list mein hi k se kam nodes the
                }
                break;
            }
        }
        return res;
    }

    // left se `k` nodes ko in-place reverse karta hai
    private void reverse(ListNode head, int k) {
        ListNode prev = null;
        ListNode curr = head;
        for (int i = 0; i < k && curr != null; i++) {
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
    def reverseKGroup(self, head, k):
        if head is None:
            return head

        left = head
        prevleft = None                # pichhle group ki tail
        res = None                     # final head

        while True:
            # 1) group verify
            right = left
            for _ in range(k - 1):
                if right is None:
                    break
                right = right.next

            if right is not None:              # poora group mila
                nxt = right.next               # aage wala part bacha lo
                self.reverse(left, k)          # 2) in-place reverse

                # 3a) peechhe stitch
                if prevleft is not None:
                    prevleft.next = right
                else:
                    res = right                # pehla group
                left.next = nxt                # 3b) aage stitch
                prevleft = left
                left = nxt
            else:                              # < k nodes bache
                if prevleft is not None:
                    prevleft.next = left
                else:
                    res = left
                break

        return res

    def reverse(self, head, k):
        prev, curr = None, head
        for _ in range(k):
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
| **Time** | `O(n)` — har node ko constant baar touch karte hain (ek baar right-scan ke liye, ek baar reverse ke liye). `k` fix nahi bhi ho to bhi total scan+reverse work `O(n)` hi rehta hai. |
| **Space** | `O(1)` — sirf pointers, iterative hai. (Recursive version likhoge to `O(n/k)` call-stack lagegi.) |

---

## 🧪 Edge cases

- **Empty list** (`head == null`) → seedha `head` return.
- **`k = 1`** → har "group" ek hi node ka hai, reverse ka koi effect nahi — list unchanged.
- **`k = length`** → poori list ek hi group hai → poori list reverse ho jaati hai (jaisa LC 206).
- **`k > length`** → pehla hi group scan mein `right == null` mil jaata hai → koi reversal hota hi nahi, list as-is return.
- **Length `k` ka exact multiple nahi** → aakhri incomplete group untouched rehta hai (yehi is problem ka core twist hai, WA2 dekho).
- **Single node list** (`[1]`, koi bhi `k ≥ 1`) → agar `k=1` no-op, warna (`k>1`) group incomplete → as-is.

---

## 🔁 Revision recap (10-second re-derive)

1. Har iteration: `left` fix karo, `right` ko `k-1` steps aage bhejo.
2. `right == null` (beech mein hi) → group incomplete → bacha hissa **as-is** jodo → `break`. **(Pehle verify, phir reverse — kabhi ulta nahi.)**
3. Warna: `next` bacha lo → `reverse(left, k)` → ab `right`=head, `left`=tail.
4. Stitch: `prevleft.next = right` (ya `res = right`), phir `left.next = next`.
5. Aage badho: `prevleft = left`, `left = next`.
6. **Yaad rakho:** `k = 2` daaloge to yehi **Swap Pairs (LC 24)** ban jaata hai — same template, different `k`.
