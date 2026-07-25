# Remove K Digits — LeetCode 402

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Stack/remove-k-digits.html)**
> _(monotonic increasing stack of digits, live "budget" counter, synced Java code)_

**Difficulty:** Hard · **Pattern:** Monotonic Increasing Stack · **Tags:** String, Stack, Greedy

---

## Problem

Ek non-negative integer string `num` aur integer `k` diya hai. Usme se **`k` digits hata do** taaki bacha hua number **sabse chhota possible** ho (same relative order maintain karte hue). Leading zeros na ho (jab tak result `"0"` khud na ho).

```
Input:  num = "1432219", k = 3
Output: "1219"
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh ab tak ke sabse "opposite" monotonic stack hai! `next-greater-element` aur `daily-temperatures` mein humne **decreasing stack** rakha (bade elements chhote candidates ko resolve karte the). Yahan hum chahte hain **sabse chhota number**, isliye hum ek **increasing stack** rakhenge — **chhote digits ko age priority** denge.

**Intuition:** Kisi bhi number ko chhota banane ka sabse tez tareeka hai uske **leftmost (sabse significant) digits** ko chhota karna — jaise `"432"` mein agar ek digit hatani hai, `4` (leftmost, sabse zyada weight) ko hataana best hai, `"32"` bachega, jo `"42"` ya `"43"` se chhota hai.

Socho ek **stack of digit-tiles**. Har naya digit `c` aata hai:
- Jab tak stack ka **top digit `c` se bada** hai **aur** humare paas abhi bhi **budget bacha hai** (`k > 0`), us bade top-digit ko **pop** kar do (discard — usse chhota digit uski jagah aa gaya hai, jo number ko chhota banayega).
- `c` ko push karo.

Yeh bilkul waisa hi hai jaise tum **"greedy replace"** kar rahe ho — jab bhi koi chhota digit kisi bade digit ke **right** mein aata hai, woh bada digit **turant discard** ho jaata hai (agar budget bacha hai), kyunki usko replace karna number ko turant chhota bana deta hai.

**Kya agar loop khatam hone ke baad bhi `k > 0` bacha ho?** Iska matlab poori string **already non-decreasing** thi (jaise `"12345"`, k=2) — koi bhi digit apne se **bade** digit ke pehle nahi tha, isliye kabhi pop trigger nahi hua. Ab bache hue `k` ko **stack ke end (top/right side)** se hata do — kyunki non-decreasing sequence mein **sabse bade digits hamesha sabse right** mein hote hain.

**Leading zeros ka jhanjhat:** `"10200"`, k=1 → digits process karne pe result stack banega `"0200"` (kyunki `1 > 0` pop hota hai). Par `"0200"` ek invalid representation hai — leading zeros strip karke `"200"` banega.

---

## Approach 1 — Brute force 🟥

`k` baar, har baar poori string scan karo aur **sabse pehla aisa digit dhoondo jo apne next digit se bada ho** (matlab "yahan se hatane se sabse zyada fayda"), use hata do.

```java
class Solution {
    public String removeKdigits(String num, int k) {
        while (k > 0) {
            int i = 0;
            // find first digit that's bigger than the one right after it
            while (i < num.length() - 1 && num.charAt(i) <= num.charAt(i + 1)) i++;
            num = num.substring(0, i) + num.substring(i + 1);
            k--;
        }
        // strip leading zeros
        int i = 0;
        while (i < num.length() - 1 && num.charAt(i) == '0') i++;
        num = num.substring(i);
        return num.isEmpty() ? "0" : num;
    }
}
```

**Problem kya hai:** Har removal ke baad `O(n)` scan + `O(n)` string rebuild — `k` baar repeat, total `O(k·n)`. `k` bade hone par (`k ≈ n`) yeh `O(n²)` tak ban jaata hai.

---

## Approach 2 — Optimal (Monotonic Increasing Stack) ✅

```java
class Solution {
    public String removeKdigits(String num, int k) {
        Deque<Character> stack = new ArrayDeque<>();   // increasing bottom→top

        for (char c : num.toCharArray()) {
            // budget bacha hai aur top bada hai → discard karo (number chhota banega)
            while (!stack.isEmpty() && k > 0 && stack.peek() > c) {
                stack.pop();
                k--;
            }
            stack.push(c);
        }

        // agar abhi bhi k bacha hai, string already non-decreasing thi —
        // sabse bade digits end (top) mein hain, unhe hatao
        while (k > 0 && !stack.isEmpty()) {
            stack.pop();
            k--;
        }

        // stack se result banao (bottom→top order)
        StringBuilder sb = new StringBuilder();
        while (!stack.isEmpty()) sb.append(stack.pop());
        sb.reverse();

        // leading zeros strip karo
        int i = 0;
        while (i < sb.length() - 1 && sb.charAt(i) == '0') i++;
        String result = sb.substring(i);

        return result.isEmpty() ? "0" : result;
    }
}
```

**Kyun kaam karta hai:** Increasing stack ensure karta hai ki jab bhi koi chhota digit kisi bade digit ke right mein aaye, woh bada digit (agar budget bacha ho) turant discard ho jaaye — greedy choice jo hamesha optimal hoti hai number ko chhota karne ke liye (leftmost digits ka weight sabse zyada hota hai).

---

## Dry run — `"1432219"`, `k = 3`

| char | Stack before (bottom→top) | Pops (k budget check) | k after | Push | Stack after |
|:---:|:---|:---|:---:|:---:|:---|
| `1` | `[]` | — | 3 | ✅ | `[1]` |
| `4` | `[1]` (1 not > 4) | — | 3 | ✅ | `[1,4]` |
| `3` | `[1,4]` (4>3, k=3>0) | pop `4` | 2 | ✅ | `[1,3]` |
| `2` | `[1,3]` (3>2, k=2>0) | pop `3` | 1 | ✅ | `[1,2]` |
| `2` | `[1,2]` (2 not > 2) | — | 1 | ✅ | `[1,2,2]` |
| `1` | `[1,2,2]` (2>1, k=1>0) | pop `2` | 0 | — | `[1,2,1]` (k=0, stop popping) |
| — | (k=0, 1 not popped even though 2>1 possible) | — | 0 | ✅ | `[1,2,1]` |
| `9` | `[1,2,1]` (1 not > 9) | — | 0 | ✅ | `[1,2,1,9]` |

Loop khatam. `k = 0` (budget khatam), koi extra pop nahi.

Stack (bottom→top): `[1,2,1,9]` → **`"1219"`** ✅ (no leading zeros to strip)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(k·n)` | `O(n)` | worst case `O(n²)` when `k ≈ n` |
| Monotonic Increasing Stack | `O(n)` | `O(n)` | each digit pushed once, popped at most once |

---

## Gotchas 🪤

- **`k > 0` check zaroori hai `while` loop mein** — budget khatam hote hi popping rukni chahiye, chahe stack ka top ab bhi bada ho.
- **Loop khatam hone ke baad bhi `k` bach sakta hai** — agar poora string non-decreasing tha (jaise `"12345"`, k=2), koi pop trigger nahi hua. Bacha hua `k` **end se** (stack ke top se) hatao.
- **Leading zeros strip karna mat bhoolna** — `"10200"`, k=1 → raw stack result `"0200"` ban sakta hai, jo `"200"` mein convert karna hoga.
- **All-digits-removed edge case** — `"10"`, k=2 → poori string hat jaati hai, result empty string → **return `"0"`**, empty string nahi.
- **`k == num.length()`** — poora number hat sakta hai, dhyaan rakho ki result hamesha `"0"` fallback ho, kabhi bhi truly-empty string return na ho.
- **Stack mein `Character` rakho, comparison `>` operator se** — auto-unboxing ki wajah se `stack.peek() > c` seedha kaam karta hai `char` values pe.
