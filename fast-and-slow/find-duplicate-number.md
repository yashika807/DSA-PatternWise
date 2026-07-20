# Find the Duplicate Number — LeetCode 287

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/fast-and-slow/find-duplicate-number.html)**
> _(array khud ek chain ban jaata hai, hop-arcs ke saath live phase 1 + phase 2, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Fast & Slow Pointers (Floyd's Cycle Detection on an implicit chain)  ·  **Tags:** Array, Two Pointers, Binary Search

---

## Problem

Ek array `nums` diya hai jismein `n+1` integers hain, har ek `[1, n]` range mein. Guaranteed hai ki **exactly ek number repeat** hota hai (ek ya zyada baar) — baaki sab unique. Wo duplicate number dhoondo.

**Constraints:** array ko modify nahi kar sakte, aur `O(1)` extra space mein karna hai (matlab sorting ya extra array bhi nahi).

```
Input:  nums = [1,3,4,2,2]
Output: 2

Input:  nums = [3,1,3,4,2]
Output: 3
```

---

## The story (yaad rakhne ke liye) 🔗➡️🔗

Yahan koi linked list nahi di gayi — lekin array khud ek bana sakte ho! Socho **index `i` ek node hai**, aur uska **"next pointer" hai `nums[i]`** (index ki jagah value ko next-index ki tarah use karo). Values `1..n` range mein hain, aur array ka size `n+1` hai — matlab indices `0..n` hain, par values kabhi `0` nahi hoti. Isliye **index `0`** ek safe "phantom head" hai jo khud kabhi kisi cycle ka part nahi banta.

Ab jaadu yeh hai: chunki koi ek value **do baar** aati hai, matlab **do alag indices** us **same target index** ko point karte hain. Do arrows ek hi node par converge — aur jahan do arrows milte hain, wahan se aage ka safar automatically ek **cycle** ban jaata hai (ek baar cycle mein ghuse to bahar nikalne ka koi raasta nahi, kyunki har node ka exactly ek hi outgoing arrow hota hai).

Toh: **duplicate dhoondna = is implicit linked-list mein cycle ka entry-point dhoondna.** Bilkul LC 141 + LC 142 wala hi combo — Phase 1 mein tortoise/hare se meeting point pakdo, Phase 2 mein head se reset karke entry (jo yahan seedha duplicate **value** hi hai) nikaalo.

---

## Approach 1 — Brute force (Sorting)

Array sort karo, phir adjacent equal elements dhoondo.

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int[] sorted = nums.clone();
        Arrays.sort(sorted);
        for (int i = 1; i < sorted.length; i++) {
            if (sorted[i] == sorted[i - 1]) return sorted[i];
        }
        return -1;
    }
}
```

**Time:** `O(n log n)` — sorting  
**Space:** `O(n)` (clone banaya, warna original modify ho jaata — jo allowed nahi) ya `O(log n)` sort ke internal stack ke liye agar in-place sort kiya

Kaam karta hai, lekin problem explicitly `O(1)` space aur "don't modify array" maangta hai — yeh dono violate karta hai (clone ki wajah se).

---

## Better — Binary Search on value range

`1..n` range pe binary search karo: mid choose karo, count karo kitne elements `<= mid` hain. Agar count `> mid`, duplicate `[1, mid]` mein hai, warna `(mid, n]` mein.

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int lo = 1, hi = nums.length - 1;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            int count = 0;
            for (int x : nums) if (x <= mid) count++;
            if (count > mid) hi = mid; else lo = mid + 1;
        }
        return lo;
    }
}
```

**Time:** `O(n log n)` — har binary-search step mein poora array scan  
**Space:** `O(1)` — extra structure nahi, lekin array ko modify bhi nahi karta ✅

Space requirement pooori hoti hai, par time `O(n log n)` hai — `O(n)` optimal se slow.

---

## Approach 2 — Optimal (Fast & Slow Pointers)

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int slow = 0, fast = 0;   // index 0 = phantom head

        // Phase 1: find a meeting point inside the cycle
        do {
            slow = nums[slow];             // +1 hop
            fast = nums[nums[fast]];       // +2 hops
        } while (slow != fast);

        // Phase 2: reset one pointer to index 0, move both +1
        int ptr1 = 0, ptr2 = slow;
        while (ptr1 != ptr2) {
            ptr1 = nums[ptr1];
            ptr2 = nums[ptr2];
        }
        return ptr1;                       // the duplicate value
    }
}
```

**Time:** `O(n)` — dono phases milke linear  
**Space:** `O(1)` — sirf pointers, array untouched

---

## Dry run

Input: `nums = [1, 3, 4, 2, 2]`  (indices `0:1, 1:3, 2:4, 3:2, 4:2`)

**Phase 1** (`slow = fast = 0`):

| Step | slow | fast | Verdict |
|:---:|:---:|:---:|:---|
| start | 0 | 0 | dono index 0 |
| hop 1 | 1 | 3 | `1 != 3`, continue |
| hop 2 | 3 | 2 | `3 != 2`, continue |
| hop 3 | 2 | 2 | `slow == fast == 2` → **meeting point mila**, Phase 2 shuru |

**Phase 2** (`ptr1 = 0`, `ptr2 = 2`, dono +1 hop):

| Step | ptr1 | ptr2 | Verdict |
|:---:|:---:|:---:|:---|
| init | 0 | 2 | `ptr1 != ptr2` |
| hop 1 | 1 | 4 | `ptr1 != ptr2` |
| hop 2 | 3 | 2 | `ptr1 != ptr2` |
| hop 3 | **2** | **2** | `ptr1 == ptr2 == 2` → **duplicate = nums[2]'s target = 2** ✅ |

**Result:** duplicate number = **`2`**.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Sorting | `O(n log n)` | `O(n)`* | *clone chahiye, array modify allowed nahi |
| Binary Search on range | `O(n log n)` | `O(1)` | array untouched, but slower than optimal |
| Fast & Slow Pointers | `O(n)` | `O(1)` | optimal — array bhi untouched |

---

## Gotchas 🪤

- **`slow`/`fast` `nums[0]` se nahi, index `0` se shuru hote hain** — `int slow = 0, fast = 0;` (index, not value). Yeh "phantom head" hai jo cycle ka part nahi hota kyunki koi value `0` nahi ho sakti.
- **`do...while` zaroori hai** — `slow`/`fast` dono `0` se start hote hain, isliye pehle move karna hoga warna `slow==fast` turant (0th step pe hi) true ho jaayega bina kuch check kiye.
- **Phase 2 mein `ptr2 = slow` (meeting point), `ptr2 = 0` nahi** — sirf `ptr1` naye se `0` se start hota hai, `ptr2`/`slow` apni jagah rehta hai. Yeh LC 142 jaisa hi hai.
- **Answer node ki *value* hai, index nahi** — is problem mein cycle entry ka *index* aur us index ki *value* dono barabar hote hain (kyunki `nums[i]` hi next-pointer hai), isliye `ptr1` seedha return ho sakta hai.
- **Array read-only treat karo** — koi bhi approach jo `nums` ko sort/modify kare, constraint todta hai. Fast & slow pointers array ko sirf *read* karte hain.
- **Multiple duplicates (same value 3+ baar) bhi chalta hai** — algorithm sirf itna guarantee maangta hai ki kam se kam ek value repeat ho; extra repeats bhi usi cycle ke andar aa jaate hain.
