# Find All Anagrams in a String — LeetCode 438

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Sliding-Window/string-anagrams.html)**
> _(step-through fixed-size window slide, live match-list readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sliding Window (fixed size, exact frequency match, collect all)  ·  **Tags:** String, Hash Map, Sliding Window

---

## Problem

Do strings `s` aur `p` diye hain. `s` mein `p` ke **saare anagram substrings ke start indices** return karo.

```
Input:  s = "cbaebabacd", p = "abc"
Output: [0, 6]
Reason: s[0..2] = "cba" aur s[6..8] = "bac" — dono "abc" ke anagrams hain
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh bilkul **`permutation-in-a-string`** wala hi survey hai, bas is baar tum **ek match milte hi ruk nahi jaate** — tum **poore field ko survey karte raho** aur **har matching plot** ka address (start index) note karte jaate ho.

Same fixed-size window, same `need` / `window` / `formed` machinery — bas jaha pehle `formed == required` pe `return true` tha, ab wahan `result.add(left)` karke **aage badhte raho**.

- `need` = recipe (`p` ki letter-frequency)
- `window` = current fixed-size spice-rack snapshot
- `formed == required` → is window ka start index **result list mein daalo**, phir slide continue karo

> Ek hi mechanism, do alag sawal: "**kya koi match hai?**" (permutation-in-a-string) vs "**sab matches kahan-kahan hain?**" (yeh problem). Jab bhi aisa "find one" → "find all" upgrade dikhe, samajh jaao ki early `return` ek `list.add()` + `continue` ban jaayega.

---

## Approach 1 — Brute force 🟥

Har window ke liye frequency array banao, compare karo, match ho to index record karo.

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> result = new ArrayList<>();
        int n1 = p.length(), n2 = s.length();
        if (n1 > n2) return result;

        int[] need = new int[26];
        for (char c : p.toCharArray()) need[c - 'a']++;

        for (int i = 0; i + n1 <= n2; i++) {
            int[] window = new int[26];
            for (int j = i; j < i + n1; j++) {
                window[s.charAt(j) - 'a']++;       // har window fresh se banti hai
            }
            if (Arrays.equals(need, window)) {
                result.add(i);
            }
        }
        return result;
    }
}
```

**Problem kya hai:** `O(n2 · 26)` — har window ke liye fresh array + compare. Bade `n2` pe slow.

---

## Approach 2 — Optimal (Fixed Sliding Window + formed/required) ✅

Same fixed-window slide jo `permutation-in-a-string` mein use hui — bas match milne pe `return` ki jagah **result list mein add karke aage badho**.

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> result = new ArrayList<>();
        int n1 = p.length(), n2 = s.length();
        if (n1 > n2) return result;

        Map<Character, Integer> need = new HashMap<>();
        for (char c : p.toCharArray()) {
            need.put(c, need.getOrDefault(c, 0) + 1);
        }

        Map<Character, Integer> window = new HashMap<>();
        int required = need.size();
        int formed = 0;

        for (int right = 0; right < n2; right++) {
            char rc = s.charAt(right);
            window.put(rc, window.getOrDefault(rc, 0) + 1);          // enter
            if (need.containsKey(rc) && window.get(rc).intValue() == need.get(rc).intValue()) {
                formed++;
            }

            if (right >= n1) {
                char lc = s.charAt(right - n1);
                if (need.containsKey(lc) && window.get(lc).intValue() == need.get(lc).intValue()) {
                    formed--;
                }
                window.put(lc, window.get(lc) - 1);                  // exit
            }

            // match mila? add karo aur aage badho — return mat karo
            if (right >= n1 - 1 && formed == required) {
                result.add(right - n1 + 1);
            }
        }
        return result;
    }
}
```

> ⚠️ **`return true` → `result.add(start); continue scanning`** — yehi is problem ka sabse bada conceptual switch hai. Baaki poora mechanism identical hai `permutation-in-a-string` se.

---

## Dry run — `s = "cbaebabacd", p = "abc"`

`need = {a:1, b:1, c:1}`, `required = 3`, `n1 = 3`

Index: `0:c 1:b 2:a 3:e 4:b 5:a 6:b 7:a 8:c 9:d`

| right | enter | exit (right≥3) | formed (after both) | check (right≥2) | match start |
|:---:|:---:|---|:---:|---|:---:|
| 0 | c | — | 1 | no | — |
| 1 | b | — | 2 | no | — |
| 2 | a | — | **3** | yes → formed==3 ✅ | **0** |
| 3 | e | exit `c` (1→0, was matching → formed−1) | 2 | no | — |
| 4 | b | exit `b` (2→1, wasn't exactly matching → no change) | 2 | no | — |
| 5 | a | exit `a` (2→1, wasn't exactly matching → no change) | 2 | no | — |
| 6 | b | exit `e` (1→0, not in need → no change) | 2 | no | — |
| 7 | a | exit `b` (2→1, wasn't exactly matching → no change) | 2 | no | — |
| 8 | c | exit `a` (2→1, wasn't exactly matching → no change) | **3** | yes → formed==3 ✅ | **6** |
| 9 | d | exit `b` (1→0, was matching → formed−1) | 2 | no | — |

**Result:** `[0, 6]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(n2 · 26)` | `O(26)` | fresh frequency array + compare, har window ke liye |
| Fixed Sliding Window + formed | `O(n1 + n2)` | `O(26)` | `p` ek baar scan, `s` ek baar slide |

---

## Gotchas 🪤

- **Early `return` mat karo** — `permutation-in-a-string` se ulta, yahan match milne ke baad bhi **loop continue** karna hai, poori string scan karni hai.
- **`right >= n1 - 1` guard** — sirf tab check karo jab window poori size (`n1`) tak pahunch chuki ho.
- **Overlapping matches allowed hain** — window fixed-size slide karta hai (ek index aage), isliye consecutive/overlapping anagrams (jaise `"abab"` mein `p="ab"` → `[0,1,2]`) sab pakde jaate hain, koi skip nahi hota.
- **Result order** — indices naturally increasing order mein aate hain kyunki `right` hamesha aage badhta hai; extra sorting ki zaroorat nahi.
- **Empty result** — agar `p` kabhi match nahi karta, `result` khaali list rahegi (`null` nahi), yehi expected behavior hai.
