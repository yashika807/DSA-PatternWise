# 2074. Reverse Nodes in Even Length Groups 🔗

> **Pattern:** Linked List → *Reverse in increasing-size groups (with truncation handling)*
> **Difficulty:** Hard · **Platform:** [LeetCode 2074](https://leetcode.com/problems/reverse-nodes-in-even-length-groups/)
> **Visualizer:** [reverse-nodes-in-even-length-groups.html](https://yashika807.github.io/DSA-PatternWise/Linked-List/reverse-nodes-in-even-length-groups.html) — step-by-step debugger with live pointers (`←` `→` `Space` `R`)

---

## 🧩 Problem

Ek singly linked list di hai. List ko groups mein todo jinki sizes **badhte kram mein** hain: group 1 ki size `1`, group 2 ki size `2`, group 3 ki size `3`, ... waise hi aage. **Aakhri group** intended size se **chhota** ho sakta hai agar list utne nodes tak khatam ho jaaye — usko jitne nodes mile utne hi maano (koi padding nahi).

Har group ki **actual (asli) node-count** dekho: agar wo **even** hai to us group ko reverse karo; **odd** hai to jaisa hai waisa chhod do.

```
Input:  5 → 2 → 6 → 3 → 9 → 1 → 7 → 3 → 8 → 4
Groups: [5] [2,6] [3,9,1] [7,3,8,4]
         odd  even    odd      even
        skip  rev    skip     rev
Output: 5 → 6 → 2 → 3 → 9 → 1 → 4 → 8 → 3 → 7
```

---

## ❌ Galat approaches (pehle inhe reject karo)

### WA1 — "Reverse-decision **intended** group size se lo, **actual** truncated count se nahi"
List end ke paas jab group truncate ho jaata hai (list khatam ho jaati hai beech mein), tab intended size aur actual size ka parity (even/odd) **alag** ho sakta hai. Bug: decision intended size se lena.

**Counterexample — 12-node list `1..12`:**
- g1(1):`[1]` odd, skip. g2(2):`[2,3]` even, reverse → `3,2`. g3(3):`[4,5,6]` odd, skip. g4(4):`[7,8,9,10]` even, reverse → `10,9,8,7`.
- g5 **intended size 5**, lekin sirf **2 nodes bache** (`11,12`) → **actual count = 2 (even)** → reverse karna chahiye → `12,11`.
- WA1 (intended size `5` = odd use karke) galti se **skip** kar dega → last do nodes as-is reh jaayenge (`11,12`), jabki sahi output `12,11` hona chahiye.
- **Fix:** hamesha `min(groupSize, remaining nodes)` se **actual count nikaal ke uski parity** check karo, intended `groupSize` ki nahi.

### WA2 — Odd (no-reverse) group ke baad `prevTail` update karna bhool jaana
`prevTail` = pichhle **finalized** group ki tail. Reverse-branch mein ise update karna yaad rehta hai, lekin **odd/skip branch** mein bhool jaate hain.

**Counterexample — `5→2→6→3→9→1→7→3→8→4`:**
- Group 3 (`3,9,1`, odd) process hone ke baad, agar `prevTail` ko `node` (=`1`, group 3 ki last node) pe update **nahi** kiya, `prevTail` abhi bhi group 2 ki tail (`2`) pe atka reh jaata hai.
- Agli iteration (`groupSize=4`) `node = prevTail` se shuru hogi — matlab **`2` se dobara** walk karega, `3,9,1,7` ko ek naya group maan lega, jinme se `3,9,1` **pehle se hi finalized** the.
- Isse nodes duplicate-process hote hain, links corrupt ho sakte hain, aur `while (prevTail.next != null)` ka `prevTail` kabhi asli aakhri node tak nahi pahunch pata → **potential infinite loop**.
- **Fix:** har branch (reverse ho ya na ho) ke end mein `prevTail` ko is group ki **last node** pe explicitly set karo.

### WA3 — Reversal ke baad purani head (naya tail) ko aage wale part se jodna bhool jaana
Group ko reverse karte waqt sirf andar ka 3-pointer reversal kar diya, lekin **do boundary connections** bhool gaye: (a) `prevTail.next = <naya head>`, (b) `<purani head, ab tail>.next = <agle group ka pehla node>`.

**Counterexample — group 2 (`2,6`) reverse karte waqt:**
- Reversal ke andar `curr.next = reversedHead` assignments ke through purani head (node `2`) ka `.next` last step mein `null` ban jaata hai (kyunki standalone reversal `prev=null` se shuru hota hai).
- Agar explicitly `groupOldHead.next = afterGroup` (yaani `2.next = 3`) **nahi likha**, to list `5 → 6 → 2 → null` pe **prematurely khatam** ho jaayegi — baaki saare groups (`3,9,1,7,3,8,4`) permanently list se kat jaayenge.
- **Fix:** reversal se pehle hi `afterGroup = node.next` save karo, aur reversal ke baad dono connections explicitly karo: `prevTail.next = reversedHead` aur `groupOldHead.next = afterGroup`.

---

## 💡 Intuition

`prevTail` ek anchor hai — **pichhle finalized group ki last node**. Har round mein:

1. **Group ki actual last node dhoondo:** `prevTail` se `groupSize` steps aage walk karo (`node.next != null` tak hi, taaki list se aage na nikal jao). Jitne steps chal paaye, wahi **actual `count`** hai.
2. **Parity check:** `count % 2 == 0` → **is group ko reverse karo**. Warna as-is chhod do.
3a. **Even (reverse):** `afterGroup = node.next` bacha lo (group ke baad ka pehla node), phir `prevTail.next` se `node` tak ka segment standard 3-pointer se reverse karo. Reversal ke baad **do stitches** zaroori: `prevTail.next = <naya head>` aur `<purani head, ab tail>.next = afterGroup`. Naya `prevTail` = purani head (ab tail).
3b. **Odd (skip):** kuch reverse nahi hota, bas `prevTail = node` (is group ki last node) — taaki agla round sahi jagah se shuru ho.
4. `groupSize++` karke agle round mein jao. Jab `prevTail.next == null` ho jaaye, list khatam — loop ruk jaata hai.

Yehi is problem ka "Hard" twist hai: group sizes **fixed nahi, badhti** hain, aur **aakhri group truncate** ho sakta hai — isliye har baar **actual count** nikaalna aur **dono pointers (`prevTail` + boundary stitching)** sambhal ke rakhna zaroori hai.

---

## 🔍 Dry run — `5 → 2 → 6 → 3 → 9 → 1 → 7 → 3 → 8 → 4`

| Group | intended size | actual count | parity | action | List state after |
|:---:|:---:|:---:|:---:|:---:|:---|
| 1 | 1 | 1 | odd | skip (`prevTail = 5`) | 5 → 2 → 6 → 3 → 9 → 1 → 7 → 3 → 8 → 4 |
| 2 | 2 | 2 | even | reverse `[2,6]` → `[6,2]` | 5 → **6 → 2** → 3 → 9 → 1 → 7 → 3 → 8 → 4 |
| 3 | 3 | 3 | odd | skip (`prevTail = 1`) | 5 → 6 → 2 → 3 → 9 → 1 → 7 → 3 → 8 → 4 |
| 4 | 4 | 4 | even | reverse `[7,3,8,4]` → `[4,8,3,7]` | 5 → 6 → 2 → 3 → 9 → 1 → **4 → 8 → 3 → 7** |

Loop check `prevTail.next` (`prevTail = 7`) → `null` → list khatam, loop ruk gaya.

**Final:** `5 → 6 → 2 → 3 → 9 → 1 → 4 → 8 → 3 → 7 → null` ✅

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
    public ListNode reverseEvenLengthGroups(ListNode head) {
        if (head == null) return null;

        ListNode prevTail = head;   // last node of the previous finalized group
        int groupSize = 2;

        while (prevTail.next != null) {
            // 1) walk up to groupSize steps to find this group's ACTUAL last node
            ListNode node = prevTail;
            int count = 0;
            for (int i = 0; i < groupSize && node.next != null; i++) {
                node = node.next;
                count++;
            }

            if (count % 2 == 0) {                    // 2) even -> reverse
                ListNode afterGroup = node.next;       // save what's after the group
                ListNode groupOldHead = prevTail.next;
                ListNode reversedHead = null;
                ListNode curr = groupOldHead;

                for (int i = 0; i < count; i++) {
                    ListNode next = curr.next;
                    curr.next = reversedHead;
                    reversedHead = curr;
                    curr = next;
                }

                prevTail.next = reversedHead;          // 3a) stitch: prev anchor -> new head
                groupOldHead.next = afterGroup;         //      stitch: new tail -> rest of list
                prevTail = groupOldHead;                // new anchor for next round
            } else {                                    // odd -> skip, just advance anchor
                prevTail = node;
            }

            groupSize++;
        }
        return head;
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
    def reverseEvenLengthGroups(self, head):
        if head is None:
            return None

        prevTail = head            # last node of the previous finalized group
        groupSize = 2

        while prevTail.next is not None:
            # 1) walk up to groupSize steps to find this group's ACTUAL last node
            node = prevTail
            count = 0
            for _ in range(groupSize):
                if node.next is None:
                    break
                node = node.next
                count += 1

            if count % 2 == 0:                     # 2) even -> reverse
                afterGroup = node.next              # save what's after the group
                groupOldHead = prevTail.next
                reversedHead = None
                curr = groupOldHead

                for _ in range(count):
                    nxt = curr.next
                    curr.next = reversedHead
                    reversedHead = curr
                    curr = nxt

                prevTail.next = reversedHead        # 3a) stitch: prev anchor -> new head
                groupOldHead.next = afterGroup      #      stitch: new tail -> rest of list
                prevTail = groupOldHead             # new anchor for next round
            else:                                   # odd -> skip, just advance anchor
                prevTail = node

            groupSize += 1

        return head
```

---

## ⏱️ Complexity

| | |
|---|---|
| **Time** | `O(n)` — har node do baar tak touch hota hai (ek baar group-boundary count karne mein, ek baar reversal mein agar even hai) — dono milake bhi total work linear hi rehta hai, groups ki sankhya (`O(√n)`) irrelevant hai kyunki per-group work uske apne size ke proportional hai. |
| **Space** | `O(1)` — sirf `prevTail`, `node`, `curr`, `reversedHead` jaise pointers, koi extra structure nahi. |

---

## 🧪 Edge cases

- **`head == null`** → seedha `null` return.
- **Single node** (`[5]`) → `prevTail.next` shuru se hi `null`, loop chalta hi nahi — as-is return.
- **List exactly triangular-number length** (`1,3,6,10,...` jaise `1+2+3+4=10`) → aakhri group bhi poora banta hai, koi truncation nahi (jaisa main example).
- **Aakhri group truncated** (list beech mein khatam ho jaati hai) → actual `count < groupSize`, uski **apni parity** follow hoti hai (WA1 dekho).
- **Truncated group ka actual count `1`** (sirf ek node bacha) → hamesha odd, single-node "reversal" waise bhi no-op hoti.
- **Poori list ek hi group mein aa jaaye** (chhoti list, e.g. `[1,2]` — g1 size1 odd skip, g2 intended2 but actual2 bhi hoti hai agar 2 nodes bache) → normal case, koi special handling nahi chahiye.

---

## 🔁 Revision recap (10-second re-derive)

1. `prevTail = head` (group 1 hamesha size-1, odd, kabhi reverse nahi hota — free mein skip ho jaata hai). `groupSize = 2` se shuru.
2. Loop jab tak `prevTail.next != null`.
3. `prevTail` se `groupSize` steps (ya jab tak list khatam na ho) aage walk karke **actual `count`** aur last node (`node`) nikaalo.
4. `count` **even** → reverse: `afterGroup` bacha lo → segment reverse karo → **dono stitches** (`prevTail.next = newHead`, `oldHead.next = afterGroup`) → `prevTail = oldHead`.
5. `count` **odd** → sirf `prevTail = node` (skip, but anchor zaroor aage badhao — WA2 yehi bhoolta hai).
6. `groupSize++`, repeat.
7. **Yaad rakho:** decision hamesha **actual truncated count** ki parity se lo, **intended groupSize** ki nahi (WA1).
