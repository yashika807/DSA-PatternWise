# Ransom Note — LeetCode 383

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Hash-Maps/ransom-note.html)**
> _(step-through magazine stock ledger, live cut-by-cut simulation, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** HashMap Frequency Count  ·  **Tags:** String, HashMap, Counting

---

## Problem

Do strings diye hain — `ransomNote` aur `magazine`. Batao ki `ransomNote` **ban sakta hai ya nahi**, sirf `magazine` ke letters **cut karke**. Magazine ka har letter **sirf ek baar** use ho sakta hai.

```
Input:  ransomNote = "aa", magazine = "aab"
Output: true       (do 'a' magazine mein hain, use karo)

Input:  ransomNote = "aa", magazine = "ab"
Output: false      (magazine mein sirf ek 'a' hai, do chahiye)

Input:  ransomNote = "a", magazine = "b"
Output: false
```

---

## The story (yaad rakhne ke liye) 🧠

Socho ek kidnapper hai jo ransom note likhna chahta hai, par apni handwriting use nahi kar sakta — usse **magazine se letters cut karke chipkane** padte hain. Magazine ek **letter-stock ki dukaan** hai — har letter ki ek fixed quantity available hai.

Pehle poori magazine padh kar ek **stock ledger** bana lo (kitne kis letter ke available hain). Ab ransom note ka **har letter left se right cut karo** — ledger check karo, agar stock hai toh **cut karo aur stock -1 kar do**. Agar kisi bhi letter ka stock **khatam** mil jaaye, note adhoora reh jaata hai — turant `false`.

> Ek baar cut hua letter **dobara use nahi ho sakta** — isiliye ledger ko **mutate** karna zaroori hai (decrement), sirf check karna kaafi nahi.

---

## Approach 1 — Brute force 🟥

Ransom note ke har letter ke liye, magazine mein **left se right dhoondo** koi unused matching letter. Mil gaya toh use `used[]` mark kar do, nahi mila toh `false`.

```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        char[] mag = magazine.toCharArray();
        boolean[] used = new boolean[mag.length];

        for (char c : ransomNote.toCharArray()) {
            boolean found = false;
            for (int j = 0; j < mag.length; j++) {
                if (!used[j] && mag[j] == c) {
                    used[j] = true;
                    found = true;
                    break;
                }
            }
            if (!found) return false;
        }

        return true;
    }
}
```

**Problem kya hai:** Ransom note ke har letter ke liye **poori magazine dobara scan** hoti hai — `O(n · m)` time, jahan `n` = ransomNote length, `m` = magazine length. Bade inputs pe slow.

---

## Approach 2 — Optimal (HashMap) ✅

Magazine ka poora **stock ledger (`HashMap<Character, Integer>`)** ek pass mein bana lo. Phir ransom note ka har letter ledger se **"cut"** karo — count check karo aur turant decrement karo.

```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        Map<Character, Integer> freq = new HashMap<>();

        // magazine ka pura letter-stock ledger bana lo
        for (char c : magazine.toCharArray()) {
            freq.put(c, freq.getOrDefault(c, 0) + 1);
        }

        // ransom note ka har letter ledger se "cut" karo
        for (char c : ransomNote.toCharArray()) {
            int available = freq.getOrDefault(c, 0);
            if (available == 0) {
                return false;   // stock khatam, note nahi ban sakta
            }
            freq.put(c, available - 1);
        }

        return true;
    }
}
```

- Magazine sirf **ek baar** scan hoti hai — poora stock ledger tayyar.
- Ransom note scan karte waqt ledger ko **decrement** karte jao — ek letter ek hi baar cut ho sakta hai.
- `available == 0` milte hi turant `false` — early exit, poora ransom note scan karne ki zaroorat nahi.

---

## Dry run — `ransomNote = "aa"`, `magazine = "aab"`

**Pass 1 — Magazine se stock ledger banao:**

| i | char | Ledger after this char |
|:---:|:---:|---|
| 0 | `a` | `{a:1}` |
| 1 | `a` | `{a:2}` |
| 2 | `b` | `{a:2, b:1}` |

Final ledger: `{a:2, b:1}`

**Pass 2 — Ransom note ke letters cut karo:**

| i | char | stock before | Verdict | ledger after |
|:---:|:---:|:---:|---|---|
| 0 | `a` | 2 | ✅ available, cut karo → stock-1 | `{a:1, b:1}` |
| 1 | `a` | 1 | ✅ available, cut karo → stock-1 | `{a:0, b:1}` |

Sab letters successfully cut ho gaye → **return `true`** ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (`used[]` linear search) | `O(n · m)` | `O(m)` | har ransomNote letter ke liye poori magazine scan |
| HashMap (build + cut) | `O(n + m)` | `O(k)` | `k` = distinct characters (≤ 26, effectively `O(1)`) |

---

## Gotchas 🪤

- **Ledger `mutate` karo, sirf `contains` check mat karo** — ek letter ek hi baar cut hota hai. Sirf `freq.containsKey(c)` check karna galat hoga (ek hi letter baar-baar "available" dikhega).
- **`getOrDefault(c, 0)`** zaroori hai — ransom note mein aisa letter ho sakta hai jo magazine mein hai hi nahi.
- **Early return `false`** — jaise hi ek letter ka stock khatam mile, turant return karo, baaki ransomNote scan karne ki zaroorat nahi.
- **Empty `ransomNote`** → loop chalta hi nahi → seedha `true` (kuch cut karne ki zaroorat hi nahi), chahe magazine khaali ho ya na ho.
- **Empty `magazine`, non-empty `ransomNote`** → ledger khaali → pehla hi letter check `false` de dega.
- **Case-sensitive** — `'a'` aur `'A'` alag maane jaate hain, jaisa problem statement lowercase letters guarantee karta hai.
