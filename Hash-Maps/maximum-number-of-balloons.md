# Maximum Number of Balloons — LeetCode 1189

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Hash-Maps/maximum-number-of-balloons.html)**
> _(step-through stock ledger, live recipe-check, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** HashMap Frequency Count  ·  **Tags:** String, HashMap, Counting

---

## Problem

Ek string `text` diya hai (lowercase letters). Uske letters use karke tum kitni baar `"balloon"` word **bana sakte ho**, yeh batao. Har letter sirf utni baar use ho sakta hai jitni baar woh `text` mein maujood hai.

```
Input:  text = "nlaebolko"
Output: 1

Input:  text = "loonbalxballpoon"
Output: 2

Input:  text = "leetcode"
Output: 0        (koi 'b' hi nahi hai)
```

---

## The story (yaad rakhne ke liye) 🧠

Socho tum ek **party supply store** chala rahe ho aur customer ne "balloon" banwane ka order diya hai. Har balloon ki **recipe fix** hai:

| Letter | `b` | `a` | `l` | `o` | `n` |
|---|:---:|:---:|:---:|:---:|:---:|
| Quantity per balloon | 1 | 1 | **2** | **2** | 1 |

Tumhare paas `text` ke letters **stock** ki tarah pade hain — ek **stock ledger** bana lo jisme har letter ki available quantity likhi ho. Ab har ingredient ke stock ko uski recipe-requirement se divide karo — jitne complete balloons ek ingredient se ban sakte hain. **Sabse kam wala ingredient (bottleneck)** decide karta hai total kitne balloons ban paayenge — kyunki ek bhi ingredient khatam ho gaya toh agla balloon adhoora reh jaayega.

> `e` aur `k` jaise extra letters (jo "balloon" word mein hain hi nahi) — unka stock bekaar hai, unko ignore karo.

---

## Approach 1 — Brute force 🟥

Ek counter `k = 0` se shuru karo. Har baar poori `text` ko **dobara scan** karke check karo ki `k+1` balloons banane laayak letters bache hain ya nahi. Jab tak ban sake, `k` badhao.

```java
class Solution {
    public int maxNumberOfBalloons(String text) {
        int k = 0;
        while (canBuild(text, k + 1)) {
            k++;
        }
        return k;
    }

    // check karo ki k balloons banane ke liye text mein kaafi letters hain ya nahi
    private boolean canBuild(String text, int k) {
        int b = 0, a = 0, l = 0, o = 0, n = 0;
        for (char c : text.toCharArray()) {
            if (c == 'b') b++;
            else if (c == 'a') a++;
            else if (c == 'l') l++;
            else if (c == 'o') o++;
            else if (c == 'n') n++;
        }
        return b >= k && a >= k && l >= 2 * k && o >= 2 * k && n >= k;
    }
}
```

**Problem kya hai:** Har attempt pe **poori string dobara scan** hoti hai. Answer `k` ho toh roughly `O(n · k)` time lagta hai — jitne zyada balloons ban sakte hain, utni hi baar poora text redundantly scan hota hai.

---

## Approach 2 — Optimal (HashMap) ✅

Ek hi pass mein poora **stock ledger (`HashMap<Character, Integer>`)** bana lo. Phir sirf 5 relevant letters ka stock uski recipe-requirement se divide karo aur **minimum** le lo.

```java
class Solution {
    public int maxNumberOfBalloons(String text) {
        Map<Character, Integer> freq = new HashMap<>();

        // ek hi pass mein poora stock ledger bana lo
        for (char c : text.toCharArray()) {
            freq.put(c, freq.getOrDefault(c, 0) + 1);
        }

        // "balloon" recipe: b1  a1  l2  o2  n1
        int b = freq.getOrDefault('b', 0);
        int a = freq.getOrDefault('a', 0);
        int l = freq.getOrDefault('l', 0) / 2;   // 2 lagte hain har balloon mein
        int o = freq.getOrDefault('o', 0) / 2;   // 2 lagte hain har balloon mein
        int n = freq.getOrDefault('n', 0);

        // bottleneck ingredient hi jawaab hai
        return Math.min(b, Math.min(a, Math.min(l, Math.min(o, n))));
    }
}
```

- Sirf **ek baar** text scan hota hai — poora stock ledger tayyar.
- `l` aur `o` ko `/2` karna zaroori hai kyunki har balloon ko **do-do** chahiye.
- `getOrDefault(..., 0)` — agar koi letter text mein hai hi nahi, uska stock `0` maan lo.

---

## Dry run — `text = "nlaebolko"`

**Pass 1 — Stock ledger banao (left → right):**

| i | char | Ledger after this char |
|:---:|:---:|---|
| 0 | `n` | `{n:1}` |
| 1 | `l` | `{n:1, l:1}` |
| 2 | `a` | `{n:1, l:1, a:1}` |
| 3 | `e` | `{n:1, l:1, a:1, e:1}` |
| 4 | `b` | `{n:1, l:1, a:1, e:1, b:1}` |
| 5 | `o` | `{n:1, l:1, a:1, e:1, b:1, o:1}` |
| 6 | `l` | `{n:1, l:2, a:1, e:1, b:1, o:1}` |
| 7 | `k` | `{n:1, l:2, a:1, e:1, b:1, o:1, k:1}` |
| 8 | `o` | `{n:1, l:2, a:1, e:1, b:1, o:2, k:1}` |

Final ledger: `{n:1, l:2, a:1, e:1, b:1, o:2, k:1}` (e, k irrelevant hain)

**Pass 2 — Recipe check:**

| Letter | needed/balloon | stock | balloons possible = `stock / needed` |
|:---:|:---:|:---:|:---:|
| `b` | 1 | 1 | **1** |
| `a` | 1 | 1 | **1** |
| `l` | 2 | 2 | **1** |
| `o` | 2 | 2 | **1** |
| `n` | 1 | 1 | **1** |

**Answer** = `min(1, 1, 1, 1, 1)` = **`1`** ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (rescan per attempt) | `O(n · k)` | `O(1)` | `k` = answer; jitna bada answer, utna slow |
| HashMap (single pass) | `O(n)` | `O(1)`* | `*` alphabet fixed (≤ 26 keys), effectively constant space |

---

## Gotchas 🪤

- **Sirf 5 letters matter** — `b, a, l, o, n`. Baaki sab letters (`e`, `k`, etc.) ledger mein aa sakte hain par unka koi use nahi, unhe ignore karo.
- **`l` aur `o` dono ko `/2`** karna mat bhoolna — "balloon" mein dono letters double hain. Yeh sabse common mistake hai.
- **Integer division** hi chahiye — `l/2` floor le leta hai, jo bilkul sahi hai (adhoora balloon nahi ban sakta).
- **`getOrDefault(c, 0)`** zaroori hai — agar `text` mein koi required letter ho hi nahi (jaise `"leetcode"` mein `b` nahi hai), toh stock `0` maano, `NullPointerException` mat khao.
- **Case sensitivity** — problem constraints lowercase letters guarantee karti hain, extra uppercase-handling ki zaroorat nahi.
- **Empty text** → ledger khaali → sab counts `0` → answer `0`.
