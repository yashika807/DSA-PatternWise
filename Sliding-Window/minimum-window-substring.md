# Minimum Window Substring — LeetCode 76 (Hard)

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Sliding-Window/minimum-window-substring.html)**
> _(step-through grow/shrink window, live shopping-list readout, synced Java code)_

**Difficulty:** Hard  ·  **Pattern:** Sliding Window (variable size, need/have frequency maps)  ·  **Tags:** String, Hash Map, Sliding Window

---

## Problem

Do strings `s` aur `t` diye hain. `s` ki **sabse chhoti substring** dhoondo jismein `t` ke **saare characters** (unki required frequency ke saath) maujood hon. Agar aisi koi substring nahi hai, `""` return karo.

```
Input:  s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"
Reason: "BANC" mein A, B, C teeno hain (aur yeh sabse chhoti aisi substring hai)
```

---

## The story (yaad rakhne ke liye) 🧠

Socho tumhare paas ek **grocery shopping list** hai (`t`) — "2 apples, 1 banana" jaisa kuch. Tum ek **lambe supermarket aisle** (`s`) se guzar rahe ho, apna **cart** (window) bharte jaate ho.

- `need` = tumhari shopping list — kaunsa item, kitni quantity chahiye.
- `window` = cart mein abhi kya-kya hai.
- `formed` = list ke kitne **items fully satisfied** ho chuke hain (exact ya zyada quantity mil chuki).

Jab tak `formed < required` (list poori nahi hui), tum **aage badhte raho** (right se grow) aur cart bharte raho. Jaise hi **poori list satisfy ho jaaye** (`formed == required`), tum **checkout line ki taraf** sabse chhota rasta dhoondte ho — left se items **hatate jaao** (shrink) jab tak list phir se incomplete na ho jaaye, **har valid moment pe length record karte hue**.

> Yeh ab tak ke sabse "layered" sliding window hai — do frequency maps (`need` aur `window`) aur ek counter (`formed`) milke batate hain "kya window abhi valid hai?" bina har baar poora map compare kiye.

---

## Approach 1 — Brute force 🟥

Har possible substring nikaalo, check karo `t` ke saare characters (frequency ke saath) usmein hain ki nahi, sabse chhoti record karo.

```java
class Solution {
    public String minWindow(String s, String t) {
        int n = s.length();
        String best = "";

        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                String sub = s.substring(i, j + 1);
                if (contains(sub, t) && (best.isEmpty() || sub.length() < best.length())) {
                    best = sub;
                }
            }
        }
        return best;
    }

    private boolean contains(String sub, String t) {
        int[] count = new int[128];
        for (char c : sub.toCharArray()) count[c]++;
        for (char c : t.toCharArray()) {
            if (--count[c] < 0) return false;
        }
        return true;
    }
}
```

**Problem kya hai:** `O(n² · (n + m))` — bohot slow. Har substring ke liye fresh frequency check.

---

## Approach 2 — Optimal (Sliding Window + Need/Have Maps) ✅

`right` se grow karo, `window` map update karo. Jab koi character apni **exact required count** tak pahunch jaaye, `formed++`. Jab `formed == required`, shrink karo jab tak valid rahe, best length record karte hue.

```java
class Solution {
    public String minWindow(String s, String t) {
        if (t.isEmpty() || s.length() < t.length()) return "";

        Map<Character, Integer> need = new HashMap<>();
        for (char c : t.toCharArray()) {
            need.put(c, need.getOrDefault(c, 0) + 1);   // shopping list bano
        }

        Map<Character, Integer> window = new HashMap<>();
        int required = need.size();     // list mein kitne distinct items hain
        int formed = 0;                 // ab tak kitne items fully satisfy ho chuke

        int left = 0, bestLen = Integer.MAX_VALUE, bestStart = 0;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            window.put(c, window.getOrDefault(c, 0) + 1);   // cart mein daalo

            if (need.containsKey(c) && window.get(c).intValue() == need.get(c).intValue()) {
                formed++;                                    // yeh item ab poora satisfy hua
            }

            // poori list complete — ab shrink karke sabse chhota valid window dhoondo
            while (formed == required) {
                if (right - left + 1 < bestLen) {
                    bestLen = right - left + 1;
                    bestStart = left;
                }

                char lc = s.charAt(left);
                window.put(lc, window.get(lc) - 1);
                if (need.containsKey(lc) && window.get(lc) < need.get(lc)) {
                    formed--;                                 // ab yeh item phir se short hai
                }
                left++;
            }
        }

        return bestLen == Integer.MAX_VALUE ? "" : s.substring(bestStart, bestStart + bestLen);
    }
}
```

> ⚠️ `formed` compare karta hai `window.get(c) == need.get(c)`, **`>=` nahi** — extra copies (`window.get(c) > need.get(c)`) bhi valid hain, bas `formed` tab tak increment nahi hota jab tak exactly required count na pahunch jaaye. Shrink karte waqt `< need.get(c)` check karta hai (equal se neeche giregi tabhi list wapas incomplete hogi).

---

## Dry run — `s = "ADOBECODEBANC", t = "ABC"`

`need = {A:1, B:1, C:1}`, `required = 3`

Index: `0:A 1:D 2:O 3:B 4:E 5:C 6:O 7:D 8:E 9:B 10:A 11:N 12:C`

| right | char | window (relevant) | formed | shrink loop | bestLen |
|:---:|:---:|---|:---:|---|:---:|
| 0..4 | A,D,O,B,E | {A:1,B:1,...} | 2 (A,B satisfied) | — | ∞ |
| 5 | C | {A:1,B:1,C:1,...} | **3** = required | shrink: window `[0,5]`="ADOBEC" len 6 → bestLen=6, bestStart=0; remove `A`(idx0), formed=2, left=1, stop shrink | 6 |
| 6,7,8 | O,D,E | unchanged relevant | 2 | — | 6 |
| 9 | B | {B:2,...} | 2 (B already had 1, ab 2 — `formed` tha already satisfied for B pehle hi, no change) | — | 6 |
| 10 | A | {A:1,B:2,C:1,...} | **3** again | shrink: window `[1,10]` too long (len 10) not better; remove chars not matching need until formed drops: remove `D`(1),`O`(2) (no effect on formed) ... eventually remove `B`(idx3) → formed=2, left=4, stop | 6 |
| 11 | N | unchanged | 2 | — | 6 |
| 12 | C | {A:1,B:1,C:2,...} (after previous removals, B count back to 1 in range) | **3** | shrink: window `[left,12]` shrinks until best found: `[9,12]`="BANC" len 4 → bestLen=**4**, bestStart=9; continue shrinking, formed drops below 3, stop | **4** |

**Result:** `"BANC"` ✅ (`bestStart=9, bestLen=4` → `s.substring(9,13)`)

> Exact left-pointer arithmetic thoda intricate hai jab multiple copies (`B` twice) involved hon — visualizer mein **har micro-step** dekho, hath se poora trace karna galti-prone hai. Yehi is problem ko "Hard" banata hai.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(n² · (n+m))` | `O(n+m)` | fresh check har substring ke liye |
| Sliding Window + need/have maps | `O(n + m)` | `O(n + m)` | `right` aur `left` dono milke max `2n` steps, `t` ek baar scan |

---

## Gotchas 🪤

- **`formed` sirf exact match pe increment hota hai** — `window.get(c) == need.get(c)`, agar `>` use kiya to double-counting ho sakti hai jab extra copies aayein.
- **Shrink mein `<` check** — `window.get(lc) < need.get(lc)` hi batata hai ki item ab short hai; `<=` galat hoga (jab equal ho, list still satisfied hai us character ke liye).
- **`t` mein duplicate characters** — `need` map counts store karta hai, sirf presence nahi. `t = "AABC"` ka matlab hai `A` do baar chahiye.
- **`s.length() < t.length()`** — turant `""` return karo, valid window ban hi nahi sakti.
- **`bestLen == Integer.MAX_VALUE`** check zaroori hai — agar koi valid window mili hi nahi, `""` return karo, invalid substring index se crash mat karo.
- **Characters jo `need` mein nahi hain** — unko bhi `window` map mein daal sakte ho (extra baggage), bas unka `formed` pe koi asar nahi padta kyunki `need.containsKey(c)` check unhe skip kar deta hai.
