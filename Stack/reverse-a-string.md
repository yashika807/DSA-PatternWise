# Reverse a String using a Stack

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Stack/reverse-a-string.html)**
> _(push-all-then-pop-all, live stack column, synced Java code)_

**Difficulty:** Easy · **Pattern:** Stack (LIFO mechanics — foundational) · **Tags:** String, Stack

---

## Problem

Ek string `s` diya hai. Use ek **stack** use karke reverse karo (built-in `reverse()` ya two-pointer trick nahi — sirf push/pop se).

```
Input:  s = "hello"
Output: "olleh"
```

Yeh LeetCode ka koi numbered problem nahi hai — yeh series ka **sabse basic stack problem** hai, jo aage waale trickier monotonic-stack problems (Next Greater Element, Daily Temperatures, Remove K Digits) se pehle stack ki **raw mechanics** samjhata hai.

---

## The story (yaad rakhne ke liye) 🧠

Socho tumhare paas **plates ka ek dher (stack of plates)** hai kitchen mein. Tum plates ek ek karke **upar rakhte jaate ho** — pehli plate sabse **neeche** chali jaati hai, aakhri plate sabse **upar**.

Ab jab wapas **utaarte ho**, kaunsi plate sabse pehle uthaogे? Woh jo **sabse upar** hai — matlab **sabse aakhri** rakhi gayi thi!

```
Push order:  h → e → l → l → o     (h sabse neeche, o sabse upar)
Pop order:   o → l → l → e → h     (jo last gaya, woh first aata hai)
```

Push karne ka order aur pop karne ka order **automatically ulta** ho jaata hai — yehi **Last In, First Out (LIFO)** hai, aur yehi property kisi bhi sequence ko **naturally reverse** kar deti hai. Koi extra logic nahi chahiye — bas:

1. Har character ko **push** karo left se right.
2. Sab **pop** karo — jo order mein niklenge, wahi reversed string hai.

> 💡 Yeh trick isliye important hai kyunki aage ke saare stack problems (bracket matching, monotonic stacks, etc.) isi **LIFO se order palatne** wali property pe based hain. Pehle ismein comfortable ho jao, phir monotonic stacks (jahan stack "sorted order" maintain karta hai) samajhna aasan ho jaayega.

---

## Approach 1 — Brute force (bina stack ke) 🟥

String ko end se start tak manually copy karo naya `StringBuilder` mein.

```java
class Solution {
    public String reverseString(String s) {
        StringBuilder sb = new StringBuilder();
        for (int i = s.length() - 1; i >= 0; i--) {
            sb.append(s.charAt(i));
        }
        return sb.toString();
    }
}
```

**Yeh kaam toh karta hai** (`O(n)` hi hai), par iska maqsad stack ki mechanics samajhna nahi hai — yeh sirf array indexing use kar raha hai. Interview mein agar explicitly "use a stack" bola jaaye, to yeh approach reject ho jaayegi.

---

## Approach 2 — Optimal (Stack) ✅

Pehle **saare characters push** karo, phir **saare pop** karo.

```java
class Solution {
    public String reverseString(String s) {
        Deque<Character> stack = new ArrayDeque<>();

        // Phase 1: push every character — order preserved bottom→top
        for (char c : s.toCharArray()) {
            stack.push(c);
        }

        // Phase 2: pop everything — LIFO automatically reverses it
        StringBuilder sb = new StringBuilder();
        while (!stack.isEmpty()) {
            sb.append(stack.pop());
        }

        return sb.toString();
    }
}
```

**Kyun kaam karta hai:** Push phase ke baad stack ka top hamesha **sabse aakhri pushed character** hota hai (string ka last character). Jab pop karna shuru karte ho, woh last character **sabse pehle** nikalta hai — bas isi wajah se poori string ulti ho jaati hai, koi index arithmetic ki zaroorat nahi.

---

## Dry run — `"hello"`

**Phase 1 — push everything:**

| i | char | Action | Stack (bottom→top) |
|:---:|:---:|:---|:---|
| 0 | `h` | push | `[h]` |
| 1 | `e` | push | `[h, e]` |
| 2 | `l` | push | `[h, e, l]` |
| 3 | `l` | push | `[h, e, l, l]` |
| 4 | `o` | push | `[h, e, l, l, o]` |

**Phase 2 — pop everything:**

| Pop # | Popped char | Result so far | Stack (bottom→top) |
|:---:|:---:|:---|:---|
| 1 | `o` | `"o"` | `[h, e, l, l]` |
| 2 | `l` | `"ol"` | `[h, e, l]` |
| 3 | `l` | `"oll"` | `[h, e]` |
| 4 | `e` | `"olle"` | `[h]` |
| 5 | `h` | `"olleh"` | `[]` |

**Result:** **`"olleh"`** ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Manual reverse (index loop) | `O(n)` | `O(n)` | for output string; no stack mechanics |
| Push-then-pop (Stack) | `O(n)` | `O(n)` | one pass to push, one pass to pop |

---

## Gotchas 🪤

- **Do phases zaroor alag rakho** — pehle sab push, *phir* sab pop. Beech beech mein push/pop mix karoge to order galat ban jaayega.
- **Empty string edge case** — `""` pe stack kabhi kuch push hi nahi hoga, `pop` loop turant skip ho jaata hai, result `""`. Crash nahi hona chahiye.
- **Single character** — `"a"` → push `a`, pop `a` → `"a"`. Reverse ka koi visible effect nahi, par logic phir bhi sahi chalta hai.
- **Yeh sirf ek intro hai** — real interviews mein string reverse karne ke liye stack "optimal" nahi mana jaata (two-pointer swap `O(1)` extra space deta hai); yahan maksad sirf **stack ki LIFO mechanics** internalize karna hai jo aage complex problems mein kaam aayegi.
