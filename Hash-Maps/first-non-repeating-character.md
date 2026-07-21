# First Unique Character in a String — LeetCode 387

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Hash-Maps/first-non-repeating-character.html)**
> _(step-through attendance register, live count-ledger, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** HashMap Frequency Count  ·  **Tags:** String, HashMap, Counting

---

## Problem

Ek string `s` diya hai. Uska **pehla non-repeating character** dhoondo aur uska **index** return karo. Agar koi non-repeating character exist nahi karta, `-1` return karo.

```
Input:  s = "leetcode"
Output: 0        (l sirf ek baar aata hai, sabse pehle)

Input:  s = "loveleetcode"
Output: 2        (v sirf ek baar aata hai)

Input:  s = "aabb"
Output: -1       (koi bhi character akela nahi hai)
```

---

## The story (yaad rakhne ke liye) 🧠

Socho ek **classroom attendance register** hai. String ke har character ek "student" hai jo class mein enter hota hai. Jab bhi koi student class mein aata hai, register mein uske naam ke saamne ek **tally mark** lag jaata hai.

Poori string ek baar padh lo — register complete ho gaya, sabko pata hai kaun **kitni baar** aaya.

Ab dobara string padho, **left se right**, aur register check karo: sabse pehla student jiski attendance **exactly 1** hai — wahi tumhara **"akela" (lonely) character** hai. Usi ka index answer hai.

> Do passes kyun? Pehla pass "kitni baar aaya" ginta hai, dusra pass "sabse pehle kaun akela tha" dhoondta hai — order sirf original string mein hi surakshit hai, register mein nahi (HashMap order guarantee nahi karta).

---

## Approach 1 — Brute force 🟥

Har character ke liye poori string dobara scan karo aur check karo koi aur uske jaisa hai ya nahi.

```java
class Solution {
    public int firstUniqChar(String s) {
        int n = s.length();

        for (int i = 0; i < n; i++) {
            boolean unique = true;
            for (int j = 0; j < n; j++) {
                if (i != j && s.charAt(i) == s.charAt(j)) {
                    unique = false;
                    break;
                }
            }
            if (unique) return i;
        }

        return -1;
    }
}
```

**Problem kya hai:** `O(n²)` time — har index ke liye poori string dobara scan. `n` bada ho toh bahut slow, aur same comparisons baar baar ho rahe hain jo waste hai.

---

## Approach 2 — Optimal (HashMap) ✅

Pehle poori string se ek **frequency ledger (`HashMap<Character, Integer>`)** bana lo. Phir dobara string scan karo aur pehla aisa character dhoondo jiski ledger mein count `1` ho.

```java
class Solution {
    public int firstUniqChar(String s) {
        Map<Character, Integer> freq = new HashMap<>();

        // Pass 1: attendance register bharo — har character ki tally badhao
        for (char c : s.toCharArray()) {
            freq.put(c, freq.getOrDefault(c, 0) + 1);
        }

        // Pass 2: left se right, sabse pehla "akela" (count == 1) dhoondo
        for (int i = 0; i < s.length(); i++) {
            if (freq.get(s.charAt(i)) == 1) {
                return i;
            }
        }

        return -1;   // koi bhi akela nahi mila
    }
}
```

- **Pass 1** register banata hai — order se koi matlab nahi, sirf counts chahiye.
- **Pass 2** original string ke order mein hi scan karta hai — isiliye **pehla** unique character hi milta hai, register ka koi random order nahi.

---

## Dry run — `s = "leetcode"`

**Pass 1 — Register banao (left → right):**

| i | char | Ledger after this char |
|:---:|:---:|---|
| 0 | `l` | `{l:1}` |
| 1 | `e` | `{l:1, e:1}` |
| 2 | `e` | `{l:1, e:2}` |
| 3 | `t` | `{l:1, e:2, t:1}` |
| 4 | `c` | `{l:1, e:2, t:1, c:1}` |
| 5 | `o` | `{l:1, e:2, t:1, c:1, o:1}` |
| 6 | `d` | `{l:1, e:2, t:1, c:1, o:1, d:1}` |
| 7 | `e` | `{l:1, e:3, t:1, c:1, o:1, d:1}` |

Final ledger: `{l:1, e:3, t:1, c:1, o:1, d:1}`

**Pass 2 — Left se right check karo:**

| i | char | freq[char] | Verdict |
|:---:|:---:|:---:|---|
| 0 | `l` | 1 | ✅ pehla akela mila → **return 0** |

**Result:** `0` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (nested scan) | `O(n²)` | `O(1)` | har index ke liye poora scan, TLE bade `n` pe |
| HashMap (two pass) | `O(n)` | `O(k)` | `k` = distinct characters (lowercase eng → ≤ 26, effectively `O(1)`) |

---

## Gotchas 🪤

- **Do pass zaroori hai** — ek hi pass mein "final count" nahi pata hota jab tak poori string na padh lo, isliye counting aur checking alag rakho.
- **`getOrDefault(c, 0) + 1`** bhoolna mat — pehli baar milne pe default `0` nahi diya toh `NullPointerException` (agar seedha `freq.get(c) + 1` likha).
- **Order register ka nahi, string ka use karo** — `HashMap` ka iteration order guaranteed nahi hota; Pass 2 mein original string index-wise scan karna hi correctness ka raaz hai.
- **Empty string** → loop chalta hi nahi, seedha `-1` return.
- **Poora string sab unique** (jaise `"abcd"`) → pehla character hi answer, sabki count `1` hogi.
- **Koi bhi unique nahi** (jaise `"aabb"`) → Pass 2 kabhi match nahi karega, `-1` return hoga.
