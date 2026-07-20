# 92. Reverse Linked List II 🔗

> **Pattern:** Linked List → *In-place partial reversal (head-insertion technique)*
> **Difficulty:** Medium · **Platform:** [LeetCode 92](https://leetcode.com/problems/reverse-linked-list-ii/)
> **Visualizer:** [reverse-a-sublist.html](https://yashika807.github.io/DSA-PatternWise/Linked-List/reverse-a-sublist.html) — step-by-step debugger with live pointers (`←` `→` `Space` `R`)

---

## 🧩 Problem

Ek singly linked list aur do positions `left` aur `right` (1-indexed, `left ≤ right`) di hain. Sirf position `left` se `right` tak ka **sub-segment** reverse karna hai — baaki list jaisi hai waisi hi rehni chahiye. Ek hi pass mein, in-place.

```
Input:  1 → 2 → 3 → 4 → 5,  left = 2, right = 4
Output: 1 → 4 → 3 → 2 → 5
              └──────┘
           sirf yeh hissa palta
```

---

## ❌ Galat approaches (pehle inhe reject karo)

### WA1 — Off-by-one: loop `right - left + 1` baar chalao
Sochte ho har position ke liye ek iteration chahiye, isliye `right - left + 1` baar plucking karte ho — lekin sahi count hai `right - left` (ek node hamesha fixed tail rehta hai, wo count nahi hota).

**Counterexample — `1→2→3→4→5`, `left=2, right=4`:**
- Do sahi iterations ke baad list ban chuki hoti: `1 → 4 → 3 → 2 → 5`, `curr(=2).next = 5`.
- Teesri (extra, galat) iteration chala di: `next = curr.next = 5`, `curr.next = next.next = null`, `next.next = prev.next = 4`, `prev.next = next = 5`.
- Result: `1 → 5 → 4 → 3 → 2 → null` — **node 5 galti se reversal mein khinch liya gaya**, aur node `2.next` `null` ho gaya (list wahin toot gayi). Ek extra iteration = ek extra node corrupt.

### WA2 — Sub-segment ko "isolate" karke standalone reverse karo, phir jodna bhool jao
Tumne socha: `left..right` ke nodes nikaalo, unhe alag se standard 3-pointer (`prev=null,curr,next`) se reverse karo, phir wapas jod do.

**Counterexample — `1→2→3→4→5`, `left=2, right=4` (segment `2→3→4`):**
- Standalone reverse (`prev=null` se shuru): result `4 → 3 → 2 → null` — ek **disconnected chain**.
- Agar ab **bhool gaye** ki (a) node `1` (yaani `left-1`) ko naye head `4` se jodna tha, aur (b) purani head `2` (ab tail) ko `right+1` yaani node `5` se jodna tha:
- `1.next` abhi bhi **stale** `2` hi point kar raha hai, aur `2.next` reversal ke baad `null` hai.
- List `head` se traverse karo: `1 → 2 → null` — **nodes 3, 4, 5 permanently lost**, sirf floating unreachable chain `4→3→2` bach jaati hai jo kisi se linked nahi.
- Isiliye is problem mein **`prev` (anchor before segment) ko kabhi move nahi karna** — usi ke reference se incremental insertion karni hai, taaki list kabhi bhi fully disconnect na ho.

### WA3 — 1-indexed `left`/`right` ko 0-indexed jaisa treat karna
`prev` ko `left-1` steps aage le jaane ki jagah galti se `left` steps le jaate ho:

```java
for (int i = 0; i < left; i++) prev = prev.next;   // ❌ ek extra step
```

**Counterexample — `1→2→3→4→5`, `left=2, right=4`:**
- Sahi: `prev` ko `left-1=1` step aage jaana chahiye → `prev = node 1`.
- Galat version: `prev` `left=2` steps aage chala gaya → `prev = node 2`.
- Ab `curr = prev.next = node 3`, aur reversal position `3..4` pe hone lagta hai instead of `2..4`.
- Result kuch aisa banega: `1 → 2 → 4 → 3 → 5` — **node 2 galti se reversal se bahar reh gaya**, jabki expected output mein wo bhi palatna tha (`1 → 4 → 3 → 2 → 5`). Off-by-one in indexing seedha wrong-range bug deta hai.

---

## 💡 Intuition — "head insertion" / front-plucking

Trick yeh hai: `curr` ko **fix** rakho (yeh sub-segment ka pehla node hai, aur reversal ke baad yeh **tail** ban jaayega — kabhi move nahi karta). `prev` bhi **fix** rakho (segment se pehle wala node — kabhi move nahi karta). Bas `curr.next` wale node ko baar-baar **pluck** karo aur usse `prev` ke turant baad **insert** kar do:

1. `dummy` node banao (taaki `left = 1` — matlab head se hi reversal — bhi bina special-case ke handle ho jaaye).
2. `prev` ko `left - 1` steps aage bhejo (segment se theek pehle wala node).
3. `curr = prev.next` — yeh sub-segment ka pehla node, **hamesha fixed rehta hai** (segment ki naya tail banega).
4. `(right - left)` baar repeat karo:
   - `next = curr.next` (jo node pluck karna hai)
   - `curr.next = next.next` (usse curr ki chain se nikaal do)
   - `next.next = prev.next` (usse turant prev ke baad insert karne ki taiyari)
   - `prev.next = next` (ab wo segment ka naya front hai)

Har iteration mein list **kabhi disconnect nahi hoti** — `prev` aur `curr` hamesha valid reference rakhte hain (yehi WA2 ka fix hai). Ek-ek node front se pluck hoke insert hota jaata hai, isliye order reverse ho jaata hai.

---

## 🔍 Dry run — `1 → 2 → 3 → 4 → 5`, `left = 2, right = 4`

`dummy → 1 → 2 → 3 → 4 → 5`. `prev` → node `1` (1 step from dummy). `curr` → node `2` (fixed).

| Iter | next = curr.next | curr.next = next.next | next.next = prev.next | prev.next = next | List after iter |
|:---:|:---:|:---:|:---:|:---:|:---|
| 1 | 3 | `2.next = 4` | `3.next = 2` | `1.next = 3` | 1 → **3 → 2** → 4 → 5 |
| 2 | 4 | `2.next = 5` | `4.next = 3` | `1.next = 4` | 1 → **4 → 3 → 2** → 5 |

Loop `right - left = 2` baar chala, ab ruk gaya. `curr (=2)` poori tarah tail ban gaya, `prev (=1)` untouched anchor raha.

**Final:** `1 → 4 → 3 → 2 → 5 → null` ✅

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
    public ListNode reverseBetween(ListNode head, int left, int right) {
        if (head == null) return null;

        ListNode dummy = new ListNode(0);
        dummy.next = head;

        ListNode prev = dummy;
        for (int i = 0; i < left - 1; i++) {   // prev = node just before `left`
            prev = prev.next;
        }

        ListNode curr = prev.next;             // curr = fixed, becomes new tail
        for (int i = 0; i < right - left; i++) {
            ListNode next = curr.next;         // node to pluck
            curr.next = next.next;             // remove it from after curr
            next.next = prev.next;             // point it at current segment front
            prev.next = next;                  // insert it right after prev
        }

        return dummy.next;
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
    def reverseBetween(self, head, left, right):
        if head is None:
            return None

        dummy = ListNode(0)
        dummy.next = head

        prev = dummy
        for _ in range(left - 1):        # prev = node just before `left`
            prev = prev.next

        curr = prev.next                 # curr = fixed, becomes new tail
        for _ in range(right - left):
            nxt = curr.next              # node to pluck
            curr.next = nxt.next         # remove it from after curr
            nxt.next = prev.next         # point it at current segment front
            prev.next = nxt              # insert it right after prev

        return dummy.next
```

---

## ⏱️ Complexity

| | |
|---|---|
| **Time** | `O(n)` — `left-1` steps to reach `prev`, phir `right-left` plucking steps. Single pass. |
| **Space** | `O(1)` — sirf `dummy`, `prev`, `curr`, `next` pointers, koi recursion ya extra structure nahi. |

---

## 🧪 Edge cases

- **`left == right`** → inner loop `0` baar chalta hai, list waisi hi rehti hai (single node "reverse" = no-op).
- **`left == 1`** → `prev` dummy pe hi rehta hai (`0` steps), reversal seedha head se shuru — `dummy` isi wajah se zaroori hai, taaki `head` khud update na karna pade.
- **`right == length`** → sub-segment list ke end tak jaata hai, `curr.next` aakhri iteration mein `null` ban jaata hai — algorithm bina extra check ke handle karta hai.
- **`left = 1, right = n`** (poori list) → poori list reverse ho jaati hai, jaisे LC 206.
- **Single node list** (`[1]`, `left=right=1`) → no-op, `1` return.
- **`head == null`** → seedha `null` return.

---

## 🔁 Revision recap (10-second re-derive)

1. `dummy → head` banao — `left=1` case ko free mein handle karta hai.
2. `prev` ko `left-1` steps aage bhejo — yeh segment se **pehle wala anchor**, kabhi move nahi hota.
3. `curr = prev.next` — segment ka pehla node, **fixed rehta hai**, reversal ke baad **tail** banega.
4. `(right-left)` baar: `curr.next` wale node ko **pluck karo** aur `prev` ke turant baad **insert** karo (4 lines: save next → unlink from curr → link to prev's old next → prev.next = next).
5. **Yaad rakho:** `prev` aur `curr` dono **kabhi move nahi hote** — sirf beech ka node baar-baar front mein plug hota hai. Isiliye list kabhi disconnect nahi hoti (WA2 ka fix).
6. Return `dummy.next`, `head` nahi.
