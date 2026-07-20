# Palindrome Linked List — LeetCode 234

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/fast-and-slow/palindrome-linkedlist.html)**
> _(multi-phase step debugger: middle-finder → in-place reverse → compare → restore, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** Fast & Slow Pointers + In-place Reversal  ·  **Tags:** Linked List, Two Pointers, Recursion

---

## Problem

Ek singly linked list ka `head` diya hai. Check karo ki list **palindrome** hai ya nahi.

```
Input:  1 → 2 → 2 → 1
Output: true

Input:  1 → 2
Output: false
```

**Follow-up:** `O(n)` time aur `O(1)` space mein karo.

---

## The story (yaad rakhne ke liye) 🪞

Array hota to bas `left`/`right` do pointers se andar-andar milaate — ek shuru se, ek end se. Par linked list mein **peeche jaana** possible nahi hai (`prev` pointer nahi hota), isliye seedha yeh trick kaam nahi aati.

Toh plan yeh hai: **list ke doosre half ko khud hi ulta ghuma do**, taaki wo bhi "aage" ki taraf traverse ho sake — bas ab uski direction reverse hai. Teen chaal:

1. **Middle dhoondo** — tortoise/hare (`slow`/`fast`) se, jaisa LC 876 mein.
2. **Second half ko in-place reverse karo** — ab second half "peeche se aage" ban gaya, matlab uska pehla node original list ka **last node** hai.
3. **Dono halves ko saath-saath walk karo** — `p1` asli head se, `p2` reversed-second-half ke naye head se. Agar kahin values mismatch hui, palindrome nahi. Agar `p2` `null` tak pahunch gaya bina mismatch ke, palindrome hai.

Bonus touch: interview mein achha lagta hai agar list ko **wapas original order mein restore** kar do reversal se pehle jaisi thi — function "side-effect free" reh jaata hai.

---

## Approach 1 — Brute force (Copy into array/list)

Poori list ko `ArrayList` mein daal do, phir do-pointer se check karo.

```java
class Solution {
    public boolean isPalindrome(ListNode head) {
        List<Integer> vals = new ArrayList<>();
        for (ListNode curr = head; curr != null; curr = curr.next) vals.add(curr.val);

        int left = 0, right = vals.size() - 1;
        while (left < right) {
            if (!vals.get(left).equals(vals.get(right))) return false;
            left++;
            right--;
        }
        return true;
    }
}
```

**Time:** `O(n)`  
**Space:** `O(n)` — extra list mein saare values copy ho rahe hain

Simple aur samajhne mein easy, lekin `O(1)` space follow-up fail karta hai.

---

## Approach 2 — Optimal (Fast & Slow Pointers + In-place Reversal)

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
    public boolean isPalindrome(ListNode head) {
        if (head == null || head.next == null) return true;

        // 1) find the middle (slow ends at the tail of the first half)
        ListNode slow = head, fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        // 2) reverse the second half in place
        ListNode secondHalf = reverse(slow.next);

        // 3) compare both halves from their respective heads
        ListNode p1 = head, p2 = secondHalf;
        boolean result = true;
        while (p2 != null) {
            if (p1.val != p2.val) { result = false; break; }
            p1 = p1.next;
            p2 = p2.next;
        }

        // 4) restore the list (good practice, don't mutate input)
        slow.next = reverse(secondHalf);
        return result;
    }

    private ListNode reverse(ListNode head) {
        ListNode prev = null;
        while (head != null) {
            ListNode next = head.next;
            head.next = prev;
            prev = head;
            head = next;
        }
        return prev;
    }
}
```

**Time:** `O(n)` — middle-finding `O(n)`, reverse `O(n)`, compare `O(n)`, restore `O(n)` — sab linear, total `O(n)`  
**Space:** `O(1)` — sirf pointers, koi extra array/recursion stack nahi

---

## Dry run

Input: `1 → 2 → 2 → 1`  (node ids `0:1, 1:2, 2:2, 3:1`)

**Phase 1 — middle:**

| Step | slow | fast | Note |
|:---:|:---:|:---:|:---|
| start | node 1 (id0) | node 1 (id0) | dono head |
| move 1 | node 2 (id1) | node 2 (id2) | `fast.next.next` exists, continue |
| — | id1 | id2 | `fast.next` (id3) not null, `fast.next.next` (null) → **stop** |

`slow = id1` (first half ki tail). `secondHalf = reverse(slow.next)` → reverse hota hai `id2 → id3` (values `2 → 1`), naya head **id3** (value `1`), aur `id2.next = null`.

**Phase 2 — compare** (`p1 = id0`, `p2 = id3`):

| Step | p1 (val) | p2 (val) | Verdict |
|:---:|:---:|:---:|:---|
| 1 | id0 (1) | id3 (1) | match ✅, aage badho |
| 2 | id1 (2) | id2 (2) | match ✅, aage badho |
| — | — | `p2 = null` | loop khatam, koi mismatch nahi |

**Result:** `true` — **palindrome** ✅. Restore step list ko wapas `1→2→2→1` bana deta hai.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Copy into ArrayList | `O(n)` | `O(n)` | simple, extra memory |
| Fast & Slow + In-place Reverse | `O(n)` | `O(1)` | optimal, list bhi restore ho jaati hai |

---

## Gotchas 🪤

- **Middle-finding condition** — `fast.next != null && fast.next.next != null` use karo (LC 876 se thoda alag), taaki odd-length mein `slow` exact middle par ruke aur even-length mein first-half ki tail par (taaki second half strictly chhota ya barabar rahe, kabhi bada nahi).
- **`reverse(slow.next)` ko `slow.next = null` set kiye bina call karo** — reverse function khud `head`-parameter ke through traverse karta hai; usse pehle first-half ko explicitly cut karne ki zaroorat nahi, kyunki reverse sirf traversal-direction ke nodes ke `.next` badalta hai, `slow.next` khud tabhi update hota hai jab reverse complete ho jaaye (transiently ek node do chains se reachable hota hai — visualizer mein isi wajah se "chain 2" dikhta hai).
- **Comparison `p2` driven honi chahiye, `p1` nahi** — loop `while (p2 != null)` use karo kyunki second half hamesha first half se **chhota ya barabar** hota hai (odd-length mein middle node second half mein include nahi hota), isliye `p2` pehle khatam hoga.
- **Restore step mat bhoolo** (agar "don't mutate input" matter karta ho) — `slow.next = reverse(secondHalf);` — yeh list ko wapas forward order mein jod deta hai. Competitive coding mein skip kar sakte ho, real-world/interview mein achha practice hai.
- **Single node / empty list** — `head == null || head.next == null` guard se turant `true`; in dono cases second-half khud khaali hota, loop chalta hi nahi.
- **Values compare karo, references nahi** — `p1.val != p2.val`, na ki `p1 != p2` (jo galat hoga, kyunki yeh alag nodes hain).
