# Remove Nodes From Linked List — LeetCode 2487

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Stack/remove-nodes-from-linked-list.html)**
> _(node/arrow chain + monotonic stack side-by-side, synced Java code)_

**Difficulty:** Medium · **Pattern:** Monotonic Decreasing Stack (on a linked list) · **Tags:** Linked List, Stack, Monotonic Stack

---

## Problem

Ek linked list `head` diya hai. Har node ko **hata do** jiske **right side** mein (kahin bhi, adjacent zaroori nahi) koi node hai jiski value **usse zyada** ho. Baaki bache nodes ka linked list return karo.

```
Input:  head = [5, 2, 13, 3, 8]
Output: [13, 8]
```

`5` hata diya kyunki `13` uske right mein hai aur bada hai. `2` bhi isi wajah se gaya. `13` reh gaya kyunki uske right mein koi usse bada nahi. `3` gaya kyunki `8` bada hai. `8` reh gaya (end mein hai, koi right nahi).

---

## The story (yaad rakhne ke liye) 🧠

Yeh bilkul wahi **monotonic decreasing stack** pattern hai jo `next-greater-element` aur `daily-temperatures` mein use kiya tha — bas ab data ek **array** nahi, ek **linked list** hai, aur stack sirf "answer nikaalne" ke liye nahi, balki **result list khud banane** ke liye use hota hai.

Socho tum linked list ke nodes ko **plates ki tarah stack pe rakh rahe ho**, left se right. Jab bhi koi **nayi, bhaari plate** (bada value) aati hai, neeche ki **halki plates** (jo usse chhoti hain) **fenk do** — woh gir jaaengi kyunki koi bhaari plate unke "right" mein aa gayi (jo bilkul problem ki condition hai: "iske right mein koi bada node hai to yeh hatega").

- Node `cur` process karo → jab tak stack ka **top chhota** hai `cur.val` se, use **pop karo** (permanently discard — woh node result mein nahi aayega).
- `cur` ko push karo — abhi ke liye yeh survive kar gaya, par future ka koi bada node ise bhi hata sakta hai.
- Poori list traverse hone ke baad, stack mein jo bacha hai — **bottom se top tak** — wahi answer hai, **original relative order** mein.

> 🔑 **Key insight:** Jab `cur.val` stack ke top se bada hota hai, top wala node **hamesha ke liye discard** ho jaata hai — usse kabhi wapas result list mein nahi jodna. Isiliye popped nodes ko simply "bhool jao"; sirf **survivors** (jo end tak stack mein rehte hain) hi final answer ka hissa banenge.

---

## Approach 1 — Brute force 🟥

Har node ke liye, uske **right** mein saare nodes check karo — agar koi bada mila to is node ko hata do.

```java
class Solution {
    public ListNode removeNodes(ListNode head) {
        ListNode dummy = new ListNode(0, head);
        ListNode prev = dummy;
        ListNode cur = head;

        while (cur != null) {
            boolean shouldRemove = false;
            ListNode scan = cur.next;
            while (scan != null) {                    // check everything to the right
                if (scan.val > cur.val) { shouldRemove = true; break; }
                scan = scan.next;
            }
            if (shouldRemove) {
                prev.next = cur.next;                  // unlink cur
            } else {
                prev = cur;
            }
            cur = cur.next;
        }
        return dummy.next;
    }
}
```

**Problem kya hai:** Har node ke liye poori right-side list scan karna — worst case `O(n²)` (jaise strictly increasing list `[1,2,3,4,5]`, jahan har node ke liye scan poore end tak jaata hai).

---

## Approach 2 — Optimal (Monotonic Decreasing Stack) ✅

Standard `ListNode` definition:

```java
class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}
```

```java
class Solution {
    public ListNode removeNodes(ListNode head) {
        Deque<ListNode> stack = new ArrayDeque<>();   // holds NODES, values decreasing bottom→top
        ListNode cur = head;

        while (cur != null) {
            // jab tak stack ka top "chhota" hai cur se, woh permanently discard
            while (!stack.isEmpty() && stack.peek().val < cur.val) {
                stack.pop();
            }
            stack.push(cur);
            cur = cur.next;
        }

        // stack ke survivors se list wapas jodo — top pop karke "next" bana lo
        ListNode next = null;
        while (!stack.isEmpty()) {
            ListNode node = stack.pop();
            node.next = next;      // is node ke baad jo abhi tak joda hai
            next = node;
        }
        return next;               // aakhri pop hi naya head hai
    }
}
```

**Kyun kaam karta hai:** Traverse karte waqt jo bhi node stack ke top se **chhota** hai use `cur` discard kar deta hai — bilkul waisa hi jaisa problem chahta hai ("iske right mein koi bada node hai"). Traverse khatam hone ke baad stack mein sirf **survivors** bache hain, left-to-right order preserve karte hue (bottom = pehla survivor, top = aakhri survivor). Stack se pop karke reverse order mein `next` jodne se original order wapas ban jaata hai.

---

## Dry run — `[5, 2, 13, 3, 8]`

**Pass 1 — build stack (left to right):**

| cur.val | Stack before (bottom→top) | Pops (discarded) | Push | Stack after |
|:---:|:---|:---|:---:|:---|
| 5 | `[]` | — | ✅ | `[5]` |
| 2 | `[5]` (5 not < 2) | — | ✅ | `[5, 2]` |
| 13 | `[5, 2]` (2<13 pop, 5<13 pop) | **2, 5** | ✅ | `[13]` |
| 3 | `[13]` (13 not < 3) | — | ✅ | `[13, 3]` |
| 8 | `[13, 3]` (3<8 pop; 13 not<8) | **3** | ✅ | `[13, 8]` |

**Pass 2 — relink from top down:**

| Pop | node.next set to | `next` becomes |
|:---:|:---|:---|
| `8` | `null` | `8 → null` |
| `13` | `8` | `13 → 8 → null` |

**Result:** `13 → 8 → null` ✅ (matches `[13, 8]`)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(n²)` | `O(1)` extra | worst case: strictly increasing list |
| Monotonic Decreasing Stack | `O(n)` | `O(n)` | each node pushed once, popped at most once |

---

## Gotchas 🪤

- **Stack mein `ListNode` references rakho, sirf values nahi** — kyunki hume relinking ke liye `.next` pointer manipulate karna hai.
- **`<` strictly less use karo** — equal values ko discard mat karo; sirf strictly **greater** node hi kisi ko hata sakta hai (`[1,1,1,1]` → koi hataata nahi, sab survive).
- **Relinking do-pass mein hota hai** — pehla pass survivors decide karta hai, doosra pass unhe reverse order mein pop karke `.next` set karta hai (kyunki stack se pop karne pe reverse order milta hai, jo humein original order wapas dilata hai).
- **Discarded nodes ka `.next` touch mat karo** — unhe bas "bhool" jao; unka memory Java khud garbage-collect kar lega, explicit unlink zaroori nahi.
- **Empty list / single node** — `head = null` → loop turant khatam, stack empty, `return null`. Single node → stack `[node]`, uska `.next = null` ho jaata hai (already tha), same node return.
- **Poori tarah decreasing list** (`[5,4,3,2,1]`) → kabhi koi pop nahi hota, poori list survive karti hai as-is.
