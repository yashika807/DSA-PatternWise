# Valid Parentheses — LeetCode 20

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Stack/balanced-parentheses.html)**
> _(bracket-by-bracket push/pop, live stack column, synced Java code)_

**Difficulty:** Easy · **Pattern:** Stack (matching pairs) · **Tags:** String, Stack

---

## Problem

Sirf `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'` waala ek string `s` diya hai. Check karo ki brackets **valid** hain ya nahi:

1. Har opening bracket ka **same type ka closing bracket** hona chahiye.
2. Opening brackets **correct order** mein close hone chahiye.

```
Input:  s = "()[]{}"
Output: true

Input:  s = "([)]"
Output: false
```

---

## The story (yaad rakhne ke liye) 🧠

Socho tum apne kamre ke darwaaze khol rahe ho — pehle **bahar wala darwaaza**, phir uske andar **ek aur darwaaza**, phir uske andar **ek aur**. Ab close karte waqt kaunsa darwaaza sabse **pehle** band hona chahiye? Jo **sabse aakhri mein khola** tha! Bahar wale darwaaze ko tab tak close nahi kar sakte jab tak andar wale sab close na ho jaayein.

Yehi hai **Last In, First Out (LIFO)** — aur yeh exactly stack hai:

- Opening bracket (`(`, `[`, `{`) dekha → **push karo** (ek naya darwaaza khula, undo-history mein add ho gaya).
- Closing bracket (`)`, `]`, `}`) dekha → stack ka **top pop karo** aur check karo ki woh **matching opening** hai ya nahi. Agar nahi, ya stack pehle se **empty** hai (band karne ko kuch hai hi nahi), to turant `false`.
- Aakhir mein stack **empty** hona chahiye — warna kuch darwaaze khule reh gaye (unmatched opening brackets).

> 💡 Yeh bhi ek **matching-pair cancellation** hai (jaisa `remove-adjacent-duplicates` mein tha), bas fark itna hai ki yahan match "same character" nahi, balki "**correct partner**" (`(` ka partner `)`, etc.) dhoondhna hai — aur order (LIFO) hi validity decide karta hai.

---

## Approach 1 — Brute force 🟥

Baar baar string mein `"()"`, `"[]"`, `"{}"` jaise adjacent matching pairs dhoondo aur unhe hata do, jab tak kuch hata na sake.

```java
class Solution {
    public boolean isValid(String s) {
        while (s.contains("()") || s.contains("[]") || s.contains("{}")) {
            s = s.replace("()", "").replace("[]", "").replace("{}", "");
        }
        return s.isEmpty();
    }
}
```

**Problem kya hai:** Har `replace` call `O(n)` hai aur worst case mein `O(n)` baar chalti hai (`"((((...))))"` jaisi nested strings) — total `O(n²)`. Interview mein iski jagah `O(n)` stack solution expected hai.

---

## Approach 2 — Optimal (Stack) ✅

`Deque<Character>` ko stack ki tarah use karo — opening brackets push karo, closing pe pop karke match check karo.

```java
class Solution {
    public boolean isValid(String s) {
        Deque<Character> stack = new ArrayDeque<>();

        for (char c : s.toCharArray()) {
            if (c == '(' || c == '[' || c == '{') {
                stack.push(c);                     // door opened, remember it
            } else {
                // closing bracket — stack empty matlab close karne ko kuch nahi
                if (stack.isEmpty()) return false;

                char top = stack.pop();
                if ((c == ')' && top != '(') ||
                    (c == ']' && top != '[') ||
                    (c == '}' && top != '{')) {
                    return false;                   // wrong partner
                }
            }
        }

        return stack.isEmpty();   // koi door khula reh gaya to false
    }
}
```

---

## Dry run — `"([)]"`

| i | char | Type | Stack action | Stack (bottom→top) | Verdict |
|:---:|:---:|:---|:---|:---|:---|
| 0 | `(` | opening | push | `[(]` | ok so far |
| 1 | `[` | opening | push | `[(, []` | ok so far |
| 2 | `)` | closing | pop → top was `[` | `[(]` | `)` ka partner `(` hona chahiye tha, mila `[` → **mismatch!** |
| — | — | — | **return false immediately** | — | ❌ |

Compare karo valid case `"{[]}"` se:

| i | char | Type | Stack action | Stack (bottom→top) |
|:---:|:---:|:---|:---|:---|
| 0 | `{` | opening | push | `[{]` |
| 1 | `[` | opening | push | `[{, []` |
| 2 | `]` | closing | pop → top was `[` ✅ match | `[{]` |
| 3 | `}` | closing | pop → top was `{` ✅ match | `[]` |

Loop khatam, stack **empty** → **`true`** ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (repeated `replace`) | `O(n²)` | `O(n)` | nested brackets worst case |
| Stack (single pass) | `O(n)` | `O(n)` | each bracket pushed/popped at most once |

---

## Gotchas 🪤

- **Closing bracket pe stack empty check pehle karo** — warna `pop()` empty `Deque` pe `NoSuchElementException` fenkega.
- **Odd-length string** → kabhi valid nahi ho sakta (brackets hamesha pairs mein aate hain) — code khud handle kar leta hai kyunki final `stack.isEmpty()` false aayega, par yeh quick short-circuit sochne mein help karta hai.
- **Sirf opening dekh ke push mat karo, matching bhi check karo** — `"([)]"` jaisa case dikhne mein "balanced count" lagta hai (2 open, 2 close) par **order galat** hai. Sirf count check karna kaafi nahi.
- **Aakhir mein `stack.isEmpty()` bhoolna mat** — `"((("` jaisa case loop ke andar kabhi `false` nahi karta (koi mismatch nahi), sirf end mein leftover opening brackets se pakड़ा jaata hai.
- **Map/switch se readability better** — bade production code mein `Map<Character, Character>` use karo bracket-pairs store karne ke liye, chained `if` thoda verbose ho jaata hai zyada bracket types ke saath.
