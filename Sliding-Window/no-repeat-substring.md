# Longest Substring Without Repeating Characters — LeetCode 3

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Sliding-Window/no-repeat-substring.html)**
> _(step-through grow/shrink window, live "guest list" readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sliding Window (variable size, no-duplicate set)  ·  **Tags:** String, Hash Set, Sliding Window

---

## Problem

Ek string `s` diya hai. **Sabse lambi substring** ki length nikalo jismein **koi character repeat na ho**.

```
Input:  s = "abcabcbb"
Output: 3
Reason: "abc" sabse lambi hai bina kisi repeat ke
```

---

## The story (yaad rakhne ke liye) 🧠

Socho ek **VIP room** hai jismein **koi do log same naam ke** nahi ho sakte — ek naam, ek hi entry. Tum logon ko **right se ek-ek karke andar bhej rahe ho** (grow).

Jab tak naya aane wala **already room mein nahi hai**, sab thik hai — room aur bada ho gaya. Lekin jaise hi koi aisa naam aata hai jo **already room mein maujood hai**, tumhe **front se logon ko nikalna** padega — ek-ek karke, jab tak woh purana duplicate khud room se bahar na chala jaaye. Tabhi naye guest ko andar aane do.

- **Set** = room ke andar abhi kaun-kaun hai (koi count nahi, sirf "hai ya nahi").
- Naya char agar set mein **already hai** → shrink karo (front se nikaalo) jab tak woh duplicate hata na do.
- Phir naya char add karo, aur current window ki length se **best** compare karo.

> Yeh `fruits-into-baskets` jaisा hi grow/shrink rhythm hai, bas condition badal gayi: waha "**2 se zyada types**" trigger tha, yahan "**yeh character already andar hai**" trigger hai.

---

## Approach 1 — Brute force 🟥

Har substring generate karo, check karo saare characters unique hain ki nahi.

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int n = s.length();
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            Set<Character> seen = new HashSet<>();
            for (int j = i; j < n; j++) {
                if (seen.contains(s.charAt(j))) {
                    break;                    // duplicate mila, yeh substring yahin tak
                }
                seen.add(s.charAt(j));
                maxLen = Math.max(maxLen, j - i + 1);
            }
        }
        return maxLen;
    }
}
```

**Problem kya hai:** `O(n²)` — har start se fresh set banate hain. Bade `n` pe slow.

---

## Approach 2 — Optimal (Sliding Window + Set) ✅

`right` se grow karo. Jab tak current char set mein hai, `left` se shrink karo jab tak duplicate khud nikal na jaaye.

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        Set<Character> window = new HashSet<>();  // room ke andar kaun hai
        int left = 0, maxLen = 0;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);

            // duplicate mil gaya — front se nikaalte raho jab tak c ka purana copy na hat jaaye
            while (window.contains(c)) {
                window.remove(s.charAt(left));
                left++;
            }

            window.add(c);                          // ab guest andar aa sakta hai
            maxLen = Math.max(maxLen, right - left + 1);
        }

        return maxLen;
    }
}
```

> ⚠️ Shrink loop **`c` ke liye check karta hai**, `s.charAt(left)` ke liye nahi — hum tab tak nikaalte rehte hain jab tak *incoming* character duplicate na ho jaaye, chahe beech mein aur bhi log nikalne padein.

---

## Dry run — `s = "abcabcbb"`

Index: `0:a 1:b 2:c 3:a 4:b 5:c 6:b 7:b`

| right | char | duplicate in window? | shrink (left moves) | window (after add) | maxLen |
|:---:|:---:|---|---|---|:---:|
| 0 | a | no | — | {a} | 1 |
| 1 | b | no | — | {a,b} | 2 |
| 2 | c | no | — | {a,b,c} | 3 |
| 3 | a | **yes** | remove `a`(idx0), left=1 | {b,c,a} | 3 |
| 4 | b | **yes** | remove `b`(idx1), left=2 | {c,a,b} | 3 |
| 5 | c | **yes** | remove `c`(idx2), left=3 | {a,b,c} | 3 |
| 6 | b | **yes** | remove `a`(idx3), left=4; still has b → remove `b`(idx4), left=5 | {c,b} | 3 |
| 7 | b | **yes** | remove `c`(idx5), left=6; still has b → remove `b`(idx6), left=7 | {b} | 3 |

**Result:** `3` ✅ (substring `"abc"`)

> Row `right=6` aur `right=7` dikhaate hain ki shrink `while` loop **ek se zyada baar** chal sakta hai ek hi `right` step ke liye — jab tak duplicate poori tarah na nikal jaaye.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force (nested loop + Set) | `O(n²)` | `O(min(n, charset))` | fresh set har start ke liye |
| Sliding Window + Set | `O(n)` | `O(min(n, charset))` | har char zyada se zyada ek baar add, ek baar remove |

---

## Gotchas 🪤

- **`while`, not `if`** — ek `right` step ke liye multiple removals lag sakte hain (dry run ke `right=6,7` dekho).
- **Empty string** — loop chalega hi nahi, `maxLen = 0` correctly return hoga.
- **Saare characters unique already** — kabhi shrink hi nahi hoga, `maxLen` poori string ki length ban jaayegi.
- **Case-sensitivity** — `'a'` aur `'A'` alag characters hain jab tak problem explicitly case-insensitive na kahe.
- **Set vs Map trade-off** — yahan `HashSet` kaafi hai (sirf "hai ya nahi" chahiye). Agar last-seen *index* bhi chahiye (O(n) single-pass "jump" optimization ke liye, bina inner while ke), to `HashMap<Character, Integer>` use karke `left = max(left, lastSeen.get(c)+1)` bhi likha ja sakta hai — dono O(n) hi hain, bas constant factor thoda better.
