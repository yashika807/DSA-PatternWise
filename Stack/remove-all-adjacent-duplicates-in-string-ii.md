# Remove All Adjacent Duplicates in String II — LeetCode 1209

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Stack/remove-all-adjacent-duplicates-in-string-ii.html)**
> _(stack of (char, count) pairs, live counter badges, synced Java code)_

**Difficulty:** Medium · **Pattern:** Stack ((char, count) pairs) · **Tags:** String, Stack

---

## Problem

Ek lowercase string `s` aur integer `k` diya hai. Baar baar **`k` adjacent aur equal characters** ka group hata do jab tak koi aisa group na bache. Final string return karo.

```
Input:  s = "deeedbbcccbdaa", k = 3
Output: "aa"
```

`"eee"` (3 e's) hata do → `"ddbbcccbdaa"`. `"ccc"` hata do → `"ddbbbdaa"`. `"bbb"` hata do → `"ddaa"`... wait dhyaan do, `"dd"` sirf 2 hai (k=3 nahi), so woh nahi hatega. Final: process khatam hone tak → `"aa"`.

---

## The story (yaad rakhne ke liye) 🧠

Yaad hai humara **pehla problem, `remove-adjacent-duplicates`**? Wahan stack ka top character match hone pe **pop** kar dete the — matlab woh sirf `k = 2` ka special case tha (do same characters mile → cancel).

Yeh **wahi idea hai, generalized to any `k`**, sirf ab hum sirf "match hua ya nahi" track nahi karte — hum stack mein **(character, count) pairs** rakhte hain. Socho stack pe plates nahi, balki **labelled crates** rakhe hain: har crate pe likha hai "kaunsa character" aur "**kitni copies** abhi tak jama hui hain".

- Naya character `c` aaya → agar stack ke top crate ka character **same** hai, uska **count +1** kar do.
- Count `k` tak pahunch gaya? → poora crate **pop** kar do (poora group cancel, jaise `k` matching plates ek saath gayab ho gayi).
- Match nahi hua (naya character ya different) → ek **naya crate** push karo count `1` ke saath.

Aakhir mein jo crates bache hain, unhe bottom se top tak padho — har crate ka character uske count times repeat karo — wahi final answer hai.

> 💡 `k=2` ke liye yeh algorithm exactly `remove-adjacent-duplicates` (Problem 1) jaisa ban jaata hai — bas count `2` pe pop hoga instead of turant match pe. Same **cascading** property bhi kaam karti hai: jab ek crate pop hota hai, naya top automatically agle character se compare hota hai, isliye multi-level cancellations (jaise `"deeedbbcccbdaa"`) ek hi pass mein resolve ho jaate hain.

---

## Approach 1 — Brute force 🟥

Baar baar poori string scan karo, `k`-length ka adjacent-equal run dhoondo, hata do, phir se scan karo.

```java
class Solution {
    public String removeDuplicates(String s, int k) {
        StringBuilder sb = new StringBuilder(s);
        boolean removedSomething = true;

        while (removedSomething) {
            removedSomething = false;
            int i = 0;
            while (i < sb.length()) {
                int j = i;
                while (j < sb.length() && sb.charAt(j) == sb.charAt(i)) j++;
                if (j - i >= k) {
                    sb.delete(i, i + k);      // remove first k of the run
                    removedSomething = true;
                    break;                     // restart scan
                }
                i = j;
            }
        }
        return sb.toString();
    }
}
```

**Problem kya hai:** Har removal ke baad poora string dobara scan hota hai — worst case `O(n²/k)` ya usse bhi zyada, kaafi slow bade inputs pe.

---

## Approach 2 — Optimal (Stack of (char, count) pairs) ✅

```java
class Solution {
    public String removeDuplicates(String s, int k) {
        // Deque<int[]> — har entry [character (as int), current count]
        Deque<int[]> stack = new ArrayDeque<>();

        for (char c : s.toCharArray()) {
            if (!stack.isEmpty() && stack.peek()[0] == c) {
                stack.peek()[1]++;                 // same character — count badhao
                if (stack.peek()[1] == k) {
                    stack.pop();                    // poora group cancel!
                }
            } else {
                stack.push(new int[]{c, 1});        // naya character, naya crate
            }
        }

        // stack top se bottom order mein hai (push = addFirst),
        // isliye bottom (tail) se nikalo taaki original left-to-right order mile
        StringBuilder sb = new StringBuilder();
        while (!stack.isEmpty()) {
            int[] pair = stack.pollLast();          // bottom-most crate pehle
            char ch = (char) pair[0];
            for (int i = 0; i < pair[1]; i++) sb.append(ch);
        }

        return sb.toString();
    }
}
```

**Kyun kaam karta hai:** Har crate apne character ke "abhi tak ke consecutive count" ko track karta hai. Jaise hi count `k` chhoo leta hai, poora group ek saath gayab (pop) — aur naya top automatically pichhle character se compare hota hai, jisse cascading removals (jaise `"bbb"` cancel hone ke baad naya-bana adjacent group) bhi handle ho jaate hain, bina dobara scan kiye.

---

## Dry run — `"deeedbbcccbdaa"`, `k = 3`

| char | Top crate before | Action | Stack (bottom→top) after |
|:---:|:---|:---|:---|
| `d` | — | new crate | `[(d,1)]` |
| `e` | `(d,1)` ≠ e | new crate | `[(d,1), (e,1)]` |
| `e` | `(e,1)` = e | count→2 | `[(d,1), (e,2)]` |
| `e` | `(e,2)` = e | count→3 == k → **pop** | `[(d,1)]` |
| `d` | `(d,1)` = d | count→2 | `[(d,2)]` |
| `b` | `(d,2)` ≠ b | new crate | `[(d,2), (b,1)]` |
| `b` | `(b,1)` = b | count→2 | `[(d,2), (b,2)]` |
| `c` | `(b,2)` ≠ c | new crate | `[(d,2), (b,2), (c,1)]` |
| `c` | `(c,1)` = c | count→2 | `[(d,2), (b,2), (c,2)]` |
| `c` | `(c,2)` = c | count→3 == k → **pop** | `[(d,2), (b,2)]` |
| `b` | `(b,2)` = b | count→3 == k → **pop** | `[(d,2)]` |
| `d` | `(d,2)` = d | count→3 == k → **pop** | `[]` |
| `a` | — | new crate | `[(a,1)]` |
| `a` | `(a,1)` = a | count→2 | `[(a,2)]` |

Loop khatam. Stack (bottom→top): `[(a,2)]` → **`"aa"`** ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (rescan loop) | worse than `O(n²)` | `O(n)` | repeated scanning after each removal |
| Stack of (char,count) | `O(n)` | `O(n)` | each character processed in `O(1)` amortized |

---

## Gotchas 🪤

- **`int[]{char, count}` ka pehla element `char` hai (as int)** — comparison `stack.peek()[0] == c` mein `c` ek `char` hai, auto-widened to `int`, so comparison sahi chalta hai.
- **Count `k` tak pahunchte hi turant pop karo** — `if (stack.peek()[1] == k)`, na ki `>=` ki zaroorat (count kabhi `k` se aage nahi badhega kyunki `k` hi pe hum pop kar dete hain).
- **Output build karte waqt bottom→top order zaroori hai** — `ArrayDeque` mein `push` head pe add karta hai, isliye `pollLast()` (ya list ko reverse karke iterate) use karo taaki original left-to-right string order mile. Seedha `pop()` loop se ulta order aayega.
- **`k = 2` special case ki tarah check ho sakta hai** — dekh lo yeh Problem 1 (`remove-adjacent-duplicates`) ka hi generalization hai; agar `k=2` diya jaaye to same behavior milega.
- **All-cancel edge case** — poori string ek hi character ke `k` multiples mein ho (jaise `"aaaaaa"`, k=3) → sab cancel, result `""`.
- **Mutable array element modify karna safe hai** — `stack.peek()[1]++` seedha stack ke top element ke count ko in-place update karta hai (kyunki `int[]` reference hai), naya object banane ki zaroorat nahi.
