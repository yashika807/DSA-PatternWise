# Longest Palindrome — LeetCode 409

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Hash-Maps/longest-palindrome.html)**
> _(step-through mirror ledger, live pair-count, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** HashMap Frequency Count  ·  **Tags:** String, HashMap, Greedy

---

## Problem

Ek string `s` diya hai (lowercase aur/ya uppercase letters). Uske letters use karke jo **sabse lamba palindrome** ban sakta hai, uski **length** return karo. Har letter jitni baar `s` mein hai, utni hi baar use ho sakta hai — palindrome mein order matter nahi karta, sirf letters rearrange kar sakte ho.

```
Input:  s = "abccccdd"
Output: 7        (e.g. "dccaccd" — a ya b beech mein akela baithega)

Input:  s = "a"
Output: 1

Input:  s = "bb"
Output: 2
```

---

## The story (yaad rakhne ke liye) 🧠

Socho ek **mirror (aaina)** hai, aur tumhe uske dono taraf letters symmetric arrange karne hain. Har character ka ek **partner** chahiye — agar `'c'` teen baar hai, toh do `'c'` ek **pair** bana kar mirror ke left-right symmetric baith sakte hain, teesra `'c'` **akela** bach jaata hai.

Ek **ledger** banao — har character ki total count. Ab har entry ke **pairs nikaalo** (`count / 2` pairs, jo `2 * (count/2)` letters use karte hain — symmetric contribution). Baaki bacha hua **odd leftover** (agar kisi bhi character ka count odd hai) — sirf **ek** aisa leftover mirror ke bilkul **beech (center)** mein baith sakta hai, palindrome tod'e bina.

> Zaroori nahi ki sirf ek hi character odd ho — kayi characters odd ho sakte hain (jaise `a` aur `b` dono sirf 1 baar), par **sirf ek** unmein se center mein baith sakta hai. Baaki odd leftovers wapas discard karne padte hain.

---

## Approach 1 — Brute force 🟥

Har character ke liye poori string mein **manually count** karo (nested loop se, already-counted ko `used[]` array se mark karte hue), phir pairs aur odd-leftover track karo.

```java
class Solution {
    public int longestPalindrome(String s) {
        int n = s.length();
        boolean[] used = new boolean[n];
        int length = 0;
        boolean hasOdd = false;

        for (int i = 0; i < n; i++) {
            if (used[i]) continue;
            int count = 1;
            used[i] = true;
            for (int j = i + 1; j < n; j++) {
                if (!used[j] && s.charAt(j) == s.charAt(i)) {
                    used[j] = true;
                    count++;
                }
            }
            length += (count / 2) * 2;
            if (count % 2 == 1) hasOdd = true;
        }

        return hasOdd ? length + 1 : length;
    }
}
```

**Problem kya hai:** Har character ke liye **poori baaki string** dobara scan hoti hai — `O(n²)` time. Chhoti strings ke liye theek hai, par scale nahi karta.

---

## Approach 2 — Optimal (HashMap) ✅

Ek hi pass mein poora **frequency ledger** bana lo. Phir ledger ke saare counts pe loop chala kar pairs jodo aur ek odd-leftover ka mauka rakho.

```java
class Solution {
    public int longestPalindrome(String s) {
        Map<Character, Integer> freq = new HashMap<>();

        // ek hi pass mein poora ledger bana lo
        for (char c : s.toCharArray()) {
            freq.put(c, freq.getOrDefault(c, 0) + 1);
        }

        int length = 0;
        boolean hasOdd = false;

        // ledger ke har entry se pairs nikaalo
        for (int count : freq.values()) {
            length += (count / 2) * 2;      // pairs symmetric use ho sakte hain
            if (count % 2 == 1) hasOdd = true;
        }

        // ek hi odd leftover center mein baith sakta hai
        return hasOdd ? length + 1 : length;
    }
}
```

- `freq.values()` pe loop — order se koi farak nahi padta, sirf counts chahiye.
- `(count / 2) * 2` = integer division ka trick — pairs ka pura symmetric contribution.
- `hasOdd` sirf **ek baar** `true` set hoke reh jaata hai chahe kitne bhi characters odd ho — final `+1` sirf ek hi baar lagta hai.

---

## Dry run — `s = "abccccdd"`

**Pass 1 — Ledger banao (left → right):**

| i | char | Ledger after this char |
|:---:|:---:|---|
| 0 | `a` | `{a:1}` |
| 1 | `b` | `{a:1, b:1}` |
| 2 | `c` | `{a:1, b:1, c:1}` |
| 3 | `c` | `{a:1, b:1, c:2}` |
| 4 | `c` | `{a:1, b:1, c:3}` |
| 5 | `c` | `{a:1, b:1, c:4}` |
| 6 | `d` | `{a:1, b:1, c:4, d:1}` |
| 7 | `d` | `{a:1, b:1, c:4, d:2}` |

Final ledger: `{a:1, b:1, c:4, d:2}`

**Pass 2 — Har entry se pairs nikaalo:**

| char | count | pairs contribution `(count/2)*2` | odd? | running length |
|:---:|:---:|:---:|:---:|:---:|
| `a` | 1 | 0 | ✅ odd (hasOdd = true) | 0 |
| `b` | 1 | 0 | ✅ odd (hasOdd already true) | 0 |
| `c` | 4 | 4 | no | 4 |
| `d` | 2 | 2 | no | 6 |

`hasOdd = true` → **`length + 1 = 6 + 1 = 7`** ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (nested count + `used[]`) | `O(n²)` | `O(n)` | har character ke liye baaki string dobara scan |
| HashMap (single pass) | `O(n)` | `O(k)` | `k` = distinct characters (lower+upper → ≤ 52, effectively `O(1)`) |

---

## Gotchas 🪤

- **`+1` sirf ek baar** — chahe 10 characters odd count ke hon, sirf **ek** unmein se center mein baithega. `hasOdd` ek boolean hi rakho, counter nahi.
- **`(count / 2) * 2`**, na ki `count` seedha — integer division floor karti hai, jo bilkul sahi hai (adhoora pair use nahi hoga).
- **Case-sensitive** — `'a'` aur `'A'` alag characters hain, ledger mein alag entries banengi.
- **Empty string** → ledger khaali → `length = 0`, `hasOdd = false` → answer `0`.
- **Single character** (jaise `"a"`) → `count=1` → pairs contribution `0`, par `hasOdd = true` → answer `1`.
- **Saare characters even count ke** (jaise `"bb"`) → seedha `length` return, `+1` ki zaroorat nahi.
