# Happy Number — LeetCode 202

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/fast-and-slow/happy-number.html)**
> _(digit-square chain grow hote dekho, live slow/fast readout, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** Fast & Slow Pointers (Floyd's Cycle Detection on an implicit chain)  ·  **Tags:** Math, Hashing, Two Pointers

---

## Problem

Ek positive integer `n` diya hai. Baar-baar `n` ko uske **digits ke squares ka sum** se replace karo (e.g. `19 → 1² + 9² = 82`). Agar kabhi `n == 1` ban jaaye, `n` **happy** hai. Agar process **infinite loop** mein phas jaaye (kabhi `1` na bane), `n` **unhappy** hai.

```
Input:  n = 19
19 → 82 → 68 → 100 → 1
Output: true

Input:  n = 2
2 → 4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4 → ... (loop)
Output: false
```

---

## The story (yaad rakhne ke liye) 🔢🔁

Yahan koi array ya diya hua linked list nahi hai — lekin socho ki har number **apna khud ka "next node"** generate kar leta hai: `next(x) = sum of squares of digits(x)`. `19` "point" karta hai `82` ko, `82` point karta hai `68` ko, aur aage. Yeh ek **invisible linked list** hai jo chalte-chalte khud ban rahi hai.

Ab yahan ek maths ka mazedaar fact hai: **har number ki chain hamesha kisi cycle mein khatam hoti hai** — koi bhi number kabhi `null` tak nahi pahunchta. Do hi possibilities hain:

1. Chain `1` tak pahunchti hai, aur `next(1) = 1² = 1` — matlab `1` **khud apna next** hai (ek self-loop, cycle length 1). Yeh **happy** case hai.
2. Chain kabhi `1` nahi banti, aur ek fixed **8-number cycle** mein phas jaati hai: `4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4 → ...`. Yeh **unhappy** case hai.

Chunki chain hamesha cycle mein hi khatam hoti hai, yeh bilkul **Linked List Cycle Detection (LC 141)** jaisa hi structure ban jaata hai — bas "node.next" ki jagah `next(x) = digitSquareSum(x)` use ho raha hai. Tortoise (`slow`, +1 hop) aur hare (`fast`, +2 hops) chalao, jab dono milein — dekho meeting point `1` hai ya kuch aur.

---

## Approach 1 — Brute force (Hashing)

Har intermediate number ko `HashSet` mein daalo. Agar koi number **repeat** ho jaaye (already set mein hai), matlab loop mil gaya — aur agar chain mein kabhi `1` nahi mila, to `unhappy`.

```java
class Solution {
    public boolean isHappy(int n) {
        Set<Integer> seen = new HashSet<>();
        while (n != 1 && !seen.contains(n)) {
            seen.add(n);
            n = next(n);
        }
        return n == 1;
    }

    private int next(int n) {
        int sum = 0;
        while (n > 0) {
            int d = n % 10;
            sum += d * d;
            n /= 10;
        }
        return sum;
    }
}
```

**Time:** `O(log n)` amortized-ish per step, chain length choti hoti hai practically  
**Space:** `O(k)` — `k` = chain mein distinct numbers ka count, seen set ke liye

Kaam sahi karta hai, lekin extra `HashSet` space use hoti hai — follow-up mein "constant space?" zaroor poocha jaayega.

---

## Approach 2 — Optimal (Fast & Slow Pointers)

```java
class Solution {
    public boolean isHappy(int n) {
        int slow = n, fast = n;

        do {
            slow = next(slow);            // +1 hop
            fast = next(next(fast));      // +2 hops
        } while (slow != fast);

        return slow == 1;                 // meeting point 1? -> happy
    }

    private int next(int n) {
        int sum = 0;
        while (n > 0) {
            int d = n % 10;
            sum += d * d;
            n /= 10;
        }
        return sum;
    }
}
```

**Time:** `O(log n)` — practically bahut chhoti chain (digit-square-sum numbers bahut jaldi single/double digit mein simat jaate hain), aur cycle bhi bada nahi hota  
**Space:** `O(1)` — sirf do int variables, koi extra structure nahi

---

## Dry run

**`n = 19`** (happy) — chain: `19 → 82 → 68 → 100 → 1 → 1 → ...`

| Step | slow | fast | Verdict |
|:---:|:---:|:---:|:---|
| start | 19 | 19 | dono `n` se shuru |
| move 1 | 82 | 68 | `82 != 68`, continue |
| move 2 | 68 | 1 | `68 != 1`, continue |
| move 3 | 100 | 1 | `100 != 1`, continue *(fast `next(1)=1` par hi loop kar raha hai)* |
| move 4 | 1 | 1 | `slow == fast == 1` → **HAPPY** ✅ |

**`n = 2`** (unhappy) — chain girti hai famous 8-cycle mein: `2 → 4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4 → ...`. Slow &amp; fast eventually cycle ke andar hi kisi number (jaise `20`) par mil jaate hain — jo `1` nahi hai, isliye **NOT happy** ❌.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Hashing (visited set) | `O(log n)` | `O(k)` | `k` = chain ke distinct numbers |
| Fast & Slow Pointers | `O(log n)` | `O(1)` | optimal — Floyd's magic, koi extra memory nahi |

---

## Gotchas 🪤

- **`do...while` use karo, `while` nahi** — `slow` aur `fast` dono `n` se hi start hote hain, isliye pehle move karna zaroori hai warna `slow == fast` turant true ho jaayega (0th step pe hi).
- **Meeting point `1` hi ho, zaroori nahi** — jab `slow == fast`, unhappy case mein meeting value 8-cycle ka **koi bhi** number ho sakta hai (`4`, `16`, `37`, ...) — sirf check karo ki wo `1` hai ya nahi, specific value pe depend mat karo.
- **`next(1) == 1` yaad rakho** — yehi wajah hai ki happy chain "terminate" hoti dikhti hai (`1` par khud loop kar leti hai), warna Floyd's algorithm ko cycle chahiye hota hai kaam karne ke liye.
- **Digit extraction order matter nahi karta** — `sum of squares` order-independent hai, isliye `n % 10` se right-to-left nikaalna bilkul sahi hai.
- **Overflow ki chinta mat karo** — kisi bhi practical `int` input ke liye `next(n)` bahut jaldi chhota ho jaata hai (max ~3 digit sum of squares of a large number bhi manageable range mein rehta hai), int overflow nahi hoga.
- **Yeh LC 141 (Linked List Cycle) ka hi mental model hai** — bas "node.next" ki jagah ek pure function `digitSquareSum()` next value compute karta hai.
