# Backspace String Compare — LeetCode 844

> 🔗 **[Interactive Visualizer dekho »]
> (https://yashika807.github.io/DSA-PatternWise/Two-pointer/comparing-strings-with-backspaces.html)**
> _(step-through two Editors reading manuscripts backward, live eraser-count readout, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** Two Pointers (from the end)  ·  **Tags:** String, Two Pointers, Stack

---

## Problem

Do strings `s` aur `t` diye hain jinmein `#` character **backspace** (pichla character delete) represent karta hai. Check karo dono strings **type karne ke baad same** ban jaate hain ya nahi.

```
Input:  s = "ab#c", t = "ad#c"
Output: true      // both become "ac"
```

---

## The story (yaad rakhne ke liye) ✍️

Do **Editors** socho jo apne-apne manuscript **peeche se aage** (right-to-left) padh rahe hain, kyunki backspace ka effect sirf **peeche se hi** pata chalta hai — jab tak tum end tak na pahuncho, tumhe nahi pata ki koi character delete ho chuka hai ya nahi.

| Role | Kaun | Kaam |
|---|---|---|
| **Editor S** | string `s` ko end se padhta hai | apna "final" character dhoondta hai |
| **Editor T** | string `t` ko end se padhta hai | apna "final" character dhoondta hai |
| **Eraser count** (`skip`) | har editor ka apna counter | "kitne characters abhi delete hone baaki hain" |

Har Editor apni **manuscript ke end se** chalta hai:

- **`#` mila** → `skip++` (ek aur backspace pending hai), peeche jao.
- **`skip > 0` aur normal character mila** → yeh character **delete ho chuka** hai (kisi backspace ne khaya) → `skip--`, ignore karke peeche jao.
- **`skip == 0` aur normal character mila** → yeh **survive** kar gaya, yeh Editor ka "next real character" hai — ruk jao.

Dono Editors apna **next real character** dhoondh ke compare karte hain. Match nahi kiya, ya ek Editor ke paas characters bache aur doosre ke nahi — **`false`**. Dono end tak pahunch gaye bina kisi mismatch ke — **`true`**.

---

## Approach 1 — Brute force (build final strings) 🟥

Har string ko `Stack`/`StringBuilder` se actually "type" karo, phir compare karo.

```java
class Solution {
    public boolean backspaceCompare(String s, String t) {
        return build(s).equals(build(t));
    }

    private String build(String str) {
        StringBuilder sb = new StringBuilder();
        for (char c : str.toCharArray()) {
            if (c == '#') {
                if (sb.length() > 0) sb.deleteCharAt(sb.length() - 1);
            } else {
                sb.append(c);
            }
        }
        return sb.toString();
    }
}
```

**Problem kya hai:** Yeh `O(n + m)` time hai (theek hai), lekin `O(n + m)` **extra space** use karta hai naye strings banane ke liye. Two-pointer se hum yeh **`O(1)` extra space** mein kar sakte hain — bina naya string banaye.

---

## Approach 2 — Two Pointers from the end (Optimal) ✅

```java
class Solution {
    public boolean backspaceCompare(String s, String t) {
        int i = s.length() - 1, j = t.length() - 1;

        while (i >= 0 || j >= 0) {
            i = nextValidIndex(s, i);   // Editor S finds its next real character
            j = nextValidIndex(t, j);   // Editor T finds its next real character

            if (i >= 0 && j >= 0) {
                if (s.charAt(i) != t.charAt(j)) return false;   // mismatch
            } else if (i >= 0 || j >= 0) {
                return false;   // one string still has chars, other ran out
            }

            i--;
            j--;
        }

        return true;   // both editors reached the very start, all matched
    }

    // walks backward from `index`, skipping backspaced characters
    private int nextValidIndex(String str, int index) {
        int skip = 0;
        while (index >= 0) {
            if (str.charAt(index) == '#') {
                skip++;
                index--;
            } else if (skip > 0) {
                skip--;
                index--;
            } else {
                break;   // found a surviving character
            }
        }
        return index;
    }
}
```

### Kyun end se shuru karna zaroori hai

- Backspace ka effect **hamesha peeche ki taraf** jaata hai — `#` apne se **pehle** aaye character ko delete karta hai, aage waale ko nahi.
- Isliye **string ke end se** padhna natural hai: end se peeche jaate hue, jaise hi `#` milta hai, humein pata chal jaata hai ki agla non-`#` character delete hone waala hai — front se padhoge to yeh pehle se pata nahi chal sakta bina lookahead ke.

---

## Dry run — `s = "ab#c"`, `t = "ad#c"`

**s = a(0) b(1) #(2) c(3)** &nbsp;·&nbsp; **t = a(0) d(1) #(2) c(3)**

| Round | i (before) | nextValidIndex(s) → i | j (before) | nextValidIndex(t) → j | s[i] vs t[j] | Verdict |
|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| 1 | 3 | `c` not `#`, skip=0 → **i=3** | 3 | `c` not `#`, skip=0 → **j=3** | `c` == `c` | match, `i--,j--` |
| 2 | 2 | `#`→skip=1,i=1; `b` skip&gt;0→skip=0,i=0; `a` skip=0 → **i=0** | 2 | `#`→skip=1,j=1; `d` skip&gt;0→skip=0,j=0; `a` skip=0 → **j=0** | `a` == `a` | match, `i--,j--` |
| 3 | -1 | loop `i>=0` false, skip | -1 | loop `j>=0` false, skip | both exhausted | `i<0 && j<0` → loop ends |

**Result:** `true` ✅ (both strings resolve to `"ac"`)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Build strings (StringBuilder/Stack) | `O(n + m)` | `O(n + m)` | extra space for reconstructed strings |
| Two Pointers (from end) | `O(n + m)` | `O(1)` | no extra strings built |

---

## Gotchas 🪤

- **End se shuru karo, shuru se nahi** — front se try karoge to backspace ka effect predict karne ke liye extra bookkeeping (ya stack) chahiye hoga; end se natural tareeke se milta hai.
- **`skip` counter reset hota hai har `nextValidIndex` call mein** — yeh local hai, global tracker nahi, kyunki har baar humein sirf **next** real character chahiye, poori history nahi.
- **Length mismatch bhi handle karo** — agar ek string ke characters khatam ho gaye par doosre ke nahi (`i >= 0 XOR j >= 0`), woh bhi `false` hai — sirf character-mismatch check kaafi nahi.
- **Trailing/leading `#`s pe extra backspace ka effect nahi** — agar `#` ke pehle koi character hi na bacha ho (jaise `"#a#c"` mein pehla `#`), `skip` bas empty stack pe "no-op" ki tarah behave karta hai, code khud isko handle kar leta hai (`index` bas `-1` tak chala jaata hai).
- **Equal strings jinke saath extra `#` pairs hon** — jaise `s="a##c"`, `t="#a#c"` dono `"c"` resolve karte hain (`true`) — algorithm ise correctly handle karta hai bina kisi special-case ke.
