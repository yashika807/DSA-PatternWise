# Longest Repeating Character Replacement — LeetCode 424

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Sliding-Window/longest-substring-with-same-letters-after-replacement.html)**
> _(step-through grow/shrink window, live dominant-count readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sliding Window (variable size, majority-count trick)  ·  **Tags:** String, Hash Map, Sliding Window

---

## Problem

Ek uppercase string `s` aur ek number `k` diya hai. Tum **kisi bhi `k` characters ko kisi bhi doosre uppercase character se replace** kar sakte ho. Uske baad **sabse lambi substring** ki length nikalo jismein **saare characters same** hon.

```
Input:  s = "AABABBA", k = 1
Output: 4
Reason: "AABA" ko "AAAA" bana sakte ho (ek B ko A se replace) — length 4
```

---

## The story (yaad rakhne ke liye) 🧠

Socho tumhare paas ek **lakdi ki fence** hai, har plank ek letter se rangi hai. Tumhare paas **`k` cans of paint-over** hain — tum kisi bhi `k` planks ko **majority color** se repaint kar sakte ho, taaki poori stretch ek hi color ki dikhe.

Window ke andar jo **sabse zyada frequent character** hai, wahi tumhara "**free**" color hai — baaki sab planks ko repaint karna padega. Formula seedha hai:

```
planks jo repaint karne padenge = windowSize − mostFrequentCount
```

Jab tak yeh number `<= k` hai, window **valid** hai — grow karte raho. Jaise hi yeh `k` se zyada ho jaaye, **left se shrink karo** ek step, jab tak wapas valid na ho jaaye.

> 🔑 **Trick:** `mostFrequentCount` (jise hum `maxCount` kehte hain) ko **shrink karte waqt update nahi karte**. Yeh thoda **stale** ho sakta hai, par ismein koi harm nahi — kyunki agar window ko shrink karna pada, matlab yeh window purane best se lambi nahi ban sakti thi beech mein. Hum sirf tab tak grow karte hain jab tak window **strictly badi** na ho jaaye purane max se, aur stale `maxCount` bhi upper bound ke roop mein kaam kar jaata hai. (Neeche "Dry run" mein isko live dekhna.)

---

## Approach 1 — Brute force 🟥

Har substring ke liye character frequencies ginno, sabse frequent character ka count nikaalo, check karo baaki `<= k`.

```java
class Solution {
    public int characterReplacement(String s, int k) {
        int n = s.length();
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            int[] count = new int[26];
            int mostFreq = 0;
            for (int j = i; j < n; j++) {
                count[s.charAt(j) - 'A']++;
                mostFreq = Math.max(mostFreq, count[s.charAt(j) - 'A']);
                int windowLen = j - i + 1;
                if (windowLen - mostFreq <= k) {
                    maxLen = Math.max(maxLen, windowLen);
                }
            }
        }
        return maxLen;
    }
}
```

**Problem kya hai:** `O(n²)` (ya `O(26n²)` frequency array ke saath) — har start se fresh scan. Slow for large `n`.

---

## Approach 2 — Optimal (Sliding Window + Majority Count) ✅

`right` se grow karo, frequency array maintain karo, `maxCount` track karo (kabhi decrease nahi karte, sirf increase). Jab `windowSize - maxCount > k`, shrink karo.

```java
class Solution {
    public int characterReplacement(String s, int k) {
        int[] count = new int[26];
        int left = 0, maxCount = 0, maxLen = 0;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            count[c - 'A']++;
            maxCount = Math.max(maxCount, count[c - 'A']);   // dominant color ka count

            // (window size - dominant count) = kitne planks repaint karne padenge
            while ((right - left + 1) - maxCount > k) {
                count[s.charAt(left) - 'A']--;                // shrink: ek plank window se bahar
                left++;
            }

            maxLen = Math.max(maxLen, right - left + 1);
        }

        return maxLen;
    }
}
```

> ⚠️ `maxCount` **kabhi wapas neeche nahi laate** — yeh intentional hai. Isse window kabhi *poore* purane best se chhoti nahi hoti (sirf same size pe slide karti hai jab tak koi genuinely badi window na mile).

---

## Dry run — `s = "AABABBA", k = 1`

Index: `0:A 1:A 2:B 3:A 4:B 5:B 6:A`

| right | char | count | maxCount | windowSize | repaint = size−maxCount | shrink? | left | maxLen |
|:---:|:---:|---|:---:|:---:|:---:|---|:---:|:---:|
| 0 | A | {A:1} | 1 | 1 | 0 | no | 0 | 1 |
| 1 | A | {A:2} | 2 | 2 | 0 | no | 0 | 2 |
| 2 | B | {A:2,B:1} | 2 | 3 | 1 | no (≤1) | 0 | 3 |
| 3 | A | {A:3,B:1} | 3 | 4 | 1 | no (≤1) | 0 | 4 |
| 4 | B | {A:3,B:2} | 3 | 5 | 2 | **yes** → remove `A`(0), left=1 | 1 | 4 |
| | | {A:2,B:2} | 3 (stale) | 4 | 1 | ok, stop | 1 | 4 |
| 5 | B | {A:2,B:3} | 3 | 5 | 2 | **yes** → remove `A`(1), left=2 | 2 | 4 |
| | | {A:1,B:3} | 3 (stale) | 4 | 1 | ok, stop | 2 | 4 |
| 6 | A | {A:2,B:3} | 3 | 5 | 2 | **yes** → remove `B`(2), left=3 | 3 | 4 |
| | | {A:2,B:2} | 3 (stale) | 4 | 1 | ok, stop | 3 | **4** |

**Result:** `4` ✅ (window `[0,3]` = `"AABA"` → repaint one `B` → `"AAAA"`)

> Dekho `maxCount` step 4 ke baad bhi `3` hi rehta hai (asal mein window `{A:2,B:2}` mein sach maxCount `2` hai) — stale hone ke bawajood answer sahi hai, kyunki window ab bhi lamba nahi ho paayega jab tak koi *naya* character 4+ baar na dikhe.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(n²)` (26 constant factor) | `O(26)` | fresh count array har start ke liye |
| Sliding Window + majority count | `O(n)` | `O(26)` | har char ek baar add, ek baar remove |

---

## Gotchas 🪤

- **`maxCount` ko shrink pe update mat karo** — yeh galti se lagta hai bug jaisa, par yeh hi optimization hai. Update karoge to bhi answer sahi aayega, bas code thoda slower/complex ho jaayega.
- **Formula `windowSize - maxCount > k`** — `>` use karo shrink trigger ke liye, `>=` nahi, warna exactly-`k`-replacements waali valid window bhi shrink ho jaayegi.
- **`maxLen` record shrink loop ke *baad* hoti hai**, condition check karne se pehle nahi — window pehle valid banao, phir length lo.
- **26-size array kaafi hai** uppercase letters ke liye; agar lowercase ya mixed case allowed ho to array size adjust karo ya `HashMap` use karo.
- **`k >= n`** edge case — poori string ek hi character bana sakte ho, answer `n` hoga (loop khud-ba-khud yeh handle kar leta hai, extra check ki zaroorat nahi).
