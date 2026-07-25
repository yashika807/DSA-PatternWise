# Remove All Adjacent Duplicates In String — LeetCode 1047

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Stack/remove-adjacent-duplicates.html)**
> _(char-by-char push/pop, live stack column, synced Java code)_

**Difficulty:** Easy · **Pattern:** Stack (annihilation pairs) · **Tags:** String, Stack

---

## Problem

Ek lowercase string `s` diya hai. Baar baar do **adjacent aur equal** characters ko hata do jab tak koi adjacent duplicate na bache. Final string return karo.

```
Input:  s = "abbaca"
Output: "ca"
```

`"abbaca"` mein `"bb"` adjacent duplicate hai — hatao → `"aaca"`. Ab `"aa"` bhi adjacent duplicate ban gaya — hatao → `"ca"`. Ab koi adjacent duplicate nahi bacha.

---

## The story (yaad rakhne ke liye) 🧠

Socho tumhare paas **matching-tiles ka ek dher** hai — jaise koi table pe letter-tiles rakh rahe ho ek line mein, left se right. Jab bhi tum ek naya tile rakhte ho aur woh **upar wale tile se match** kar jaaye, dono **ek dusre ko cancel/vanish** kar dete hain (jaise antimatter — matching pair mile toh dono gayab). Match nahi kiya toh naya tile bas dher ke upar rakh do.

Yeh bilkul **stack** hai:

- Har naya character `c` aata hai → stack ka **top dekho**.
- Top === `c`? → **pop karo** (dono cancel, jaise woh kabhi the hi nahi).
- Top !== `c` (ya stack empty)? → `c` ko **push** kar do.

Poori string process hone ke baad, stack ke andar jo bacha hai — bottom se top tak padho — wahi answer hai. Koi recursion nahi, koi "phir se scan karo" nahi — stack khud hi automatically handle kar leta hai ki cancel karne ke baad **naya adjacent pair** ban gaya (jaise `"aaca"` → `"aa"` cancel hone ke baad `c` aur agla `a`... wait yahan `c` bacha, phir `a` aata hai, dono match nahi karte).

> 💡 Yeh sabse simple stack pattern hai is series mein — bas ek **pair-cancellation** trick. Aage jo monotonic-stack problems (Next Greater Element, Daily Temperatures, Remove K Digits) aayenge unmein stack **order** maintain karta hai; yahan stack sirf **matching pairs cancel** karta hai. Dono alag mental model hain — inhe mila mat dena.

---

## Approach 1 — Brute force 🟥

Har baar poori string scan karo, pehla adjacent duplicate pair dhoondo, use hata do, aur phir se scan karo — jab tak koi duplicate na bache.

```java
class Solution {
    public String removeDuplicates(String s) {
        StringBuilder sb = new StringBuilder(s);
        boolean removedSomething = true;

        while (removedSomething) {
            removedSomething = false;
            for (int i = 0; i < sb.length() - 1; i++) {
                if (sb.charAt(i) == sb.charAt(i + 1)) {
                    sb.delete(i, i + 2);   // remove the pair
                    removedSomething = true;
                    break;                 // restart scan from beginning
                }
            }
        }
        return sb.toString();
    }
}
```

**Problem kya hai:** Har removal ke baad poori string dobara scan ho rahi hai. Worst case (`"aaaaaaaa..."`) mein yeh `O(n²)` ban jaata hai — string chhoti ho rahi hai lekin baar baar restart ho raha hai.

---

## Approach 2 — Optimal (Stack) ✅

Ek single left-to-right pass mein `Deque<Character>` ko stack ki tarah use karo.

```java
class Solution {
    public String removeDuplicates(String s) {
        Deque<Character> stack = new ArrayDeque<>();

        for (char c : s.toCharArray()) {
            // top of stack se match kiya to cancel (pop), warna push
            if (!stack.isEmpty() && stack.peek() == c) {
                stack.pop();          // annihilate the pair
            } else {
                stack.push(c);        // no match, keep it
            }
        }

        // stack top-to-bottom = reverse order, isliye reverse karke build karo
        StringBuilder sb = new StringBuilder();
        while (!stack.isEmpty()) {
            sb.append(stack.pop());
        }
        return sb.reverse().toString();
    }
}
```

**Kyun kaam karta hai:** Cancel hone ke baad jo naya top banta hai woh automatically agle character ke saath compare hota hai — matlab ek hi pass mein cascading cancellations (jaise `"aaca"` mein pehle `"bb"`, phir naya-bana `"aa"`) handle ho jaate hain, koi dobara scan zaroori nahi.

---

## Dry run — `"abbaca"`

| i | char | Top se match? | Action | Stack (bottom→top) |
|:---:|:---:|:---|:---|:---|
| 0 | `a` | stack empty | push | `[a]` |
| 1 | `b` | top=`a` ≠ `b` | push | `[a, b]` |
| 2 | `b` | top=`b` == `b` | **pop** | `[a]` |
| 3 | `a` | top=`a` == `a` | **pop** | `[]` |
| 4 | `c` | stack empty | push | `[c]` |
| 5 | `a` | top=`c` ≠ `a` | push | `[c, a]` |

**Final stack (bottom→top):** `[c, a]` → **`"ca"`** ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (rescan loop) | `O(n²)` | `O(n)` | restarts scan after every removal |
| Stack (single pass) | `O(n)` | `O(n)` | each char pushed/popped at most once |

---

## Gotchas 🪤

- **Compare with `stack.peek()` before push, not with the previous original character** — after cancellations the "previous" character in the *original* string might already be gone from the stack.
- **`ArrayDeque` as stack** — use `push`/`pop`/`peek` (not `offer`/`poll`, jo queue-semantics hain).
- **Empty-stack check** — `stack.peek() == c` se pehle `!stack.isEmpty()` zaroor check karo, warna `NullPointerException` (auto-unboxing `Character` → `char`).
- **Final order** — stack se pop karte hue characters **reverse order** mein nikalte hain; ya toh `sb.reverse()` karo, ya seedha `StringBuilder` ko stack jaisa treat karके `deleteCharAt(length-1)` use kar lo (dono valid).
- **All-cancel edge case** — `"aaaa"` → sab cancel ho jaata hai, result `""` (empty string), crash nahi hona chahiye.
