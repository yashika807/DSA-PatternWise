# Reorder List — LeetCode 143

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/fast-and-slow/rearrange-a-linkedlist.html)**
> _(multi-phase step debugger: middle-finder → split → reverse → zip-merge, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Fast & Slow Pointers + In-place Reversal + Merge  ·  **Tags:** Linked List, Two Pointers, Recursion

---

## Problem

List `L0 → L1 → L2 → … → Ln-1 → Ln` di gayi hai. Isko **in-place reorder** karo is pattern mein:

```
L0 → Ln → L1 → Ln-1 → L2 → Ln-2 → ...
```

Node values copy nahi kar sakte — sirf `next` pointers rearrange karne hain.

```
Input:  1 → 2 → 3 → 4
Output: 1 → 4 → 2 → 3

Input:  1 → 2 → 3 → 4 → 5
Output: 1 → 5 → 2 → 4 → 3
```

---

## The story (yaad rakhne ke liye) 🥪

Socho list ke do sirF hain — ek **front** (`L0, L1, L2...`) aur ek **back** (`Ln, Ln-1...`). Target pattern hai: front se ek, back se ek, alternately — jaise **sandwich banate waqt** ek slice neeche, ek upar, ek neeche...

Linked list mein "back se pick karna" seedha possible nahi (backward traverse nahi hota), isliye teen chaal, palindrome-check (LC 234) jaisi hi:

1. **Middle dhoondo** (tortoise/hare) — list ko do halves mein todne ke liye landmark chahiye.
2. **List ko physically do halves mein split karo**, aur **second half ko reverse karo** — ab second half bhi "aage se" traverse ho sakta hai, bas uska order **`Ln, Ln-1, ..., Lmid+1`** ban jaata hai (yaani back-to-front).
3. **Dono halves ko zip-merge karo** — ek node first half se, ek node (reversed) second half se, alternately jodte jaao. Bas!

Yehi teen chaal palindrome-check mein bhi thi — bas wahan sirf **compare** karte the, yahan values ko **actually relink** karte hain.

---

## Approach 1 — Brute force (Copy into array, rebuild)

Saare nodes ek `ArrayList<ListNode>` mein daalo, phir do-pointer se front aur back se alternately pick karke relink karo.

```java
class Solution {
    public void reorderList(ListNode head) {
        if (head == null) return;
        List<ListNode> nodes = new ArrayList<>();
        for (ListNode curr = head; curr != null; curr = curr.next) nodes.add(curr);

        int left = 0, right = nodes.size() - 1;
        while (left < right) {
            nodes.get(left).next = nodes.get(right);
            left++;
            if (left == right) break;
            nodes.get(right).next = nodes.get(left);
            right--;
        }
        nodes.get(left).next = null;
    }
}
```

**Time:** `O(n)`  
**Space:** `O(n)` — saare node references ek list mein store ho rahe hain

Kaam karta hai, lekin `O(1)` space follow-up (jo aksar poocha jaata hai) fail karta hai.

---

## Approach 2 — Optimal (Fast & Slow Pointers + Reverse + Merge)

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
    public void reorderList(ListNode head) {
        if (head == null || head.next == null) return;

        // 1) find the middle
        ListNode slow = head, fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        // 2) split into two halves
        ListNode secondHalf = slow.next;
        slow.next = null;

        // 3) reverse the second half
        ListNode prev = null, curr = secondHalf;
        while (curr != null) {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }
        secondHalf = prev;

        // 4) zip-merge the two halves, alternating
        ListNode p1 = head, p2 = secondHalf;
        while (p2 != null) {
            ListNode n1 = p1.next;
            ListNode n2 = p2.next;
            p1.next = p2;
            if (n1 != null) p2.next = n1;
            p1 = n1;
            p2 = n2;
        }
    }
}
```

**Time:** `O(n)` — middle-finding, reverse, aur merge sab `O(n)`, total `O(n)`  
**Space:** `O(1)` — sirf pointers, koi extra array/recursion nahi

---

## Dry run

Input: `1 → 2 → 3 → 4 → 5`  (node ids `0:1, 1:2, 2:3, 3:4, 4:5`)

**Phase 1 — middle, split, reverse:**

| Step | Action | Result |
|:---:|---|---|
| find middle | slow/fast chalao | `slow = id2` (value `3`) |
| split | `secondHalf = slow.next` (`id3`), `slow.next = null` | first half: `1→2→3`, second half: `4→5` |
| reverse second half | prev/curr flip | second half ab: `5→4` (naya head `id4`, value `5`) |

**Phase 2 — zip merge** (`p1 = id0`, `p2 = id4`):

| Step | p1 (val) | p2 (val) | n1 | n2 | Action |
|:---:|:---:|:---:|:---:|:---:|---|
| 1 | 1 | 5 | 2 | 4 | `1.next = 5`; `5.next = 2` |
| 2 | 2 | 4 | 3 | null | `2.next = 4`; `n2 == null` → `4` stays tail |
| — | `p2 = null` | — | — | — | loop khatam |

**Result:** `1 → 5 → 2 → 4 → 3 → null` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Copy into ArrayList | `O(n)` | `O(n)` | simple, extra memory |
| Fast & Slow + Reverse + Merge | `O(n)` | `O(1)` | optimal — pure pointer surgery |

---

## Gotchas 🪤

- **`n1`/`n2` pehle hi save kar lo, phir relink karo** — `p1.next = p2` set karne se **pehle** `p1.next` (original next) `n1` mein save karna zaroori hai, warna original "aage ka node" reference kho jaayega. Yehi is problem ka sabse common bug hai.
- **`if (n1 != null) p2.next = n1;`** — jab first half khatam ho jaaye (`n1 == null`, odd-length list mein aakhri middle node ke baad), to `p2` (jo second-half ka last bacha node hai) ko already `null`-terminated chhod do — usse explicitly `null` set karne ki zaroorat nahi (already hai, reverse ke baad).
- **Middle-finding condition wahi hai jo palindrome check mein** — `fast.next != null && fast.next.next != null`, taaki second half hamesha first half se **chhota ya barabar** ho (isliye merge loop `p2` pe based hai, `p1` pe nahi).
- **Split step (`slow.next = null`) mat bhoolna** — agar list ko explicitly do independent halves mein nahi todoge, reverse function first half ko bhi ulat dega (poori list ek hi chain hai abhi tak).
- **Single node ya empty list** — guard clause (`head == null || head.next == null`) se seedha return, kuch reorder karne ko hota hi nahi.
- **Yeh function value `void` return karta hai** — LC 234 (palindrome) ki tarah `boolean` nahi; kaam **in-place list mutation** hai, return value nahi.
