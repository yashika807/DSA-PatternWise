# Permutation in String — LeetCode 567

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Sliding-Window/permutation-in-a-string.html)**
> _(step-through fixed-size window slide, live recipe-match readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sliding Window (fixed size, exact frequency match)  ·  **Tags:** String, Hash Map, Sliding Window

---

## Problem

Do strings `s1` aur `s2` diye hain. Check karo ki `s2` mein `s1` ka **koi permutation (anagram) substring ke roop mein maujood hai** ya nahi.

```
Input:  s1 = "ab", s2 = "eidbaooo"
Output: true
Reason: "ba" (index 3-4) "ab" ka hi ek permutation hai
```

---

## The story (yaad rakhne ke liye) 🧠

Socho `s1` ek **recipe card** hai — "2 pinch namak, 1 pinch mirchi" jaisa exact ingredient list. Tum `s2` ke ek **fixed-size spice rack window** (size = `s1.length()`) se guzar rahe ho, check karte hue ki kya current window ka **exact ingredient mix** recipe se match karta hai — **order koi matter nahi karta, sirf quantities match honi chahiye**.

Window hamesha `s1.length()` **fixed** rehti hai (jaise problem 1 ka "camera viewfinder"), bas is baar match ka criteria "sabse zyada sum" nahi, "**exact frequency match**" hai:

- `need` = recipe (kaunsa letter, kitni baar chahiye)
- `window` = current spice-rack ka snapshot
- `formed` = kitne distinct letters **exactly** recipe jaisi quantity mein hain

Jab `formed == required` (recipe ke saare letters exact match), **aur window size bhi exactly `s1.length()` hai** (isliye koi extra letter chhup nahi sakta) — **match mil gaya!**, `true` return karo.

> Yeh `fruits-into-baskets` aur `minimum-window-substring` ka **mix** hai: fixed-size slide (jaise problem 1) + need/have/formed tracking (jaise problem 8), bas yahan "at least" nahi, **"exactly equal"** chahiye.

---

## Approach 1 — Brute force 🟥

Har window nikaalo, sort karke ya frequency array bana ke compare karo.

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        int n1 = s1.length(), n2 = s2.length();
        if (n1 > n2) return false;

        int[] need = new int[26];
        for (char c : s1.toCharArray()) need[c - 'a']++;

        for (int i = 0; i + n1 <= n2; i++) {
            int[] window = new int[26];
            for (int j = i; j < i + n1; j++) {
                window[s2.charAt(j) - 'a']++;      // har window fresh se banti hai
            }
            if (Arrays.equals(need, window)) return true;
        }
        return false;
    }
}
```

**Problem kya hai:** `O(n2 · 26)` — har window ke liye fresh 26-size array banate aur compare karte hain. Bade `n2` pe repeated work.

---

## Approach 2 — Optimal (Fixed Sliding Window + formed/required) ✅

`need` map ek baar banao. Fixed-size window slide karo (enter + exit, jaise problem 1), `formed` counter se track karo ki kitne letters exact match kar rahe hain.

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        int n1 = s1.length(), n2 = s2.length();
        if (n1 > n2) return false;

        Map<Character, Integer> need = new HashMap<>();
        for (char c : s1.toCharArray()) {
            need.put(c, need.getOrDefault(c, 0) + 1);
        }

        Map<Character, Integer> window = new HashMap<>();
        int required = need.size();
        int formed = 0;

        for (int right = 0; right < n2; right++) {
            char rc = s2.charAt(right);
            window.put(rc, window.getOrDefault(rc, 0) + 1);          // enter
            if (need.containsKey(rc) && window.get(rc).intValue() == need.get(rc).intValue()) {
                formed++;
            }

            // window fixed size se badi ho gayi — leftmost ko exit karo
            if (right >= n1) {
                char lc = s2.charAt(right - n1);
                if (need.containsKey(lc) && window.get(lc).intValue() == need.get(lc).intValue()) {
                    formed--;                                          // exit se pehle match tha, ab tootega
                }
                window.put(lc, window.get(lc) - 1);
            }

            // pehli baar window poori size ko pahunchi ho, tabhi check karo
            if (right >= n1 - 1 && formed == required) {
                return true;
            }
        }
        return false;
    }
}
```

> ⚠️ `formed--` **exit se *pehle*** check hota hai (`window.get(lc) == need.get(lc)`, phir hi decrement) — kyunki hum yeh jaanna chahte hain ki *before* removal woh letter match kar raha tha ya nahi.

---

## Dry run — `s1 = "ab", s2 = "eidbaooo"`

`need = {a:1, b:1}`, `required = 2`, `n1 = 2`

Index: `0:e 1:i 2:d 3:b 4:a 5:o 6:o 7:o`

| right | char enter | window (relevant) | formed | exit (right≥n1) | check (right≥1) | match? |
|:---:|:---:|---|:---:|---|---|:---:|
| 0 | e | {} | 0 | — | no (right<1) | — |
| 1 | i | {} | 0 | — | yes → formed(0)≠2 | false |
| 2 | d | {} | 0 | exit `e`(idx0) — no effect | formed(0)≠2 | false |
| 3 | b | {b:1} | 1 | exit `i`(idx1) — no effect | formed(1)≠2 | false |
| 4 | a | {b:1,a:1} | **2** | exit `d`(idx2) — no effect | formed(2)==2 ✅ | **true** |

**Result:** `true` ✅ (window `s2[3..4] = "ba"`, permutation of `"ab"`)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(n2 · 26)` | `O(26)` | fresh frequency array + compare, har window ke liye |
| Fixed Sliding Window + formed | `O(n1 + n2)` | `O(26)` | `s1` ek baar scan, `s2` ek baar slide |

---

## Gotchas 🪤

- **Window size fixed hai, `s1.length()`** — isliye `formed == required` kaafi hai "exact match" ke liye; extra characters chhup nahi sakte kyunki total count already fixed hai.
- **`right >= n1 - 1` guard** — check tabhi karo jab window poori size tak pahunch chuki ho, warna partial window ko false-positive match mat maano.
- **Exit `formed--` order** — pehle check karo purana count match kar raha tha ya nahi, *phir* decrement karo. Ulta karoge to galat formed count aayega.
- **`n1 > n2`** — turant `false`, `s1` khud `s2` se bada hai to fit hi nahi sakta.
- **Case-sensitivity aur non-lowercase characters** — agar sirf lowercase guaranteed hai to `int[26]` array bhi use kar sakte ho (thoda fast), warna `HashMap` safe rehta hai.
