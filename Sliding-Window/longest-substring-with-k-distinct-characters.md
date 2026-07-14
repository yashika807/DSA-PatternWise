# Longest Substring with K Distinct Characters — GFG

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Sliding-Window/longest-substring-with-k-distinct-characters.html)**
> _(step-through grow/shrink window, live frequency-map readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sliding Window (variable size, frequency map)  ·  **Tags:** String, Hash Map, Sliding Window

---

## Problem

Ek string `s` aur ek number `k` diya hai. **Sabse lambi substring** dhoondo jismein **exactly `k` distinct characters** hon. Agar aisi koi substring exist nahi karti, `-1` return karo.

```
Input:  s = "aabacbebebe", k = 3
Output: 7
Reason: substring "cbebebe" mein exactly 3 distinct chars (c, b, e) hain aur yeh sabse lambi hai
```

---

## The story (yaad rakhne ke liye) 🧠

Socho tumhare paas ek **paint palette** hai jismein **exactly `k` slots** hain — har slot ek alag color ke liye. Tum string ko left-se-right paint kar rahe ho, aur jab tak naya color palette mein fit ho raha hai, tum **canvas ko lamba karte jaate ho** (right edge grow).

Jaise hi ek **naya (k+1)-waan color** aata hai — palette **overflow** ho gaya! Ab tumhe **purane colors ko left se scrape** karna padega jab tak wapas `k` colors na bach jaayen.

- **Frequency map** = palette — kaunsa color kitni baar use ho raha hai.
- Jab `map.size() > k` → **overflow**, left se shrink karo.
- Jab `map.size() == k` → yeh ek **valid painting** hai, iski length check karo — record karo agar yeh sabse lambi hai.

> Important: hum sirf **exactly `k`** track karte hain, `<= k` nahi. Jab tak `> k` hai tabhi shrink karo — `size == k-1` bhi chalega (bas record nahi hoga, kyunki humein exactly `k` chahiye).

---

## Approach 1 — Brute force 🟥

Har substring generate karo, uske distinct characters count karo, agar exactly `k` hain to length compare karo.

```java
class Solution {
    public int longestKSubstr(String s, int k) {
        int n = s.length();
        int maxLen = -1;

        for (int i = 0; i < n; i++) {
            Set<Character> seen = new HashSet<>();
            for (int j = i; j < n; j++) {
                seen.add(s.charAt(j));           // har substring ke liye fresh set
                if (seen.size() == k) {
                    maxLen = Math.max(maxLen, j - i + 1);
                } else if (seen.size() > k) {
                    break;                        // aage badhne se distinct count sirf badhega
                }
            }
        }
        return maxLen;
    }
}
```

**Problem kya hai:** `O(n²)` substrings, har ek ke liye set maintain karna. Bada `n` ho to slow.

---

## Approach 2 — Optimal (Sliding Window + Frequency Map) ✅

`right` se grow karo, `HashMap` mein frequency track karo. Jaise hi distinct count `k` se zyada ho jaaye, `left` se shrink karo jab tak wapas `<= k` na ho jaaye.

```java
class Solution {
    public int longestKSubstr(String s, int k) {
        Map<Character, Integer> freq = new HashMap<>();
        int left = 0, maxLen = -1;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            freq.put(c, freq.getOrDefault(c, 0) + 1);   // naya char andar, count++

            // overflow: palette mein zyada colors aa gaye, shrink karo
            while (freq.size() > k) {
                char lc = s.charAt(left);
                freq.put(lc, freq.get(lc) - 1);
                if (freq.get(lc) == 0) {
                    freq.remove(lc);                      // yeh color palette se hi hata do
                }
                left++;
            }

            // exactly k distinct chars → valid window, length check karo
            if (freq.size() == k) {
                maxLen = Math.max(maxLen, right - left + 1);
            }
        }

        return maxLen;
    }
}
```

> ⚠️ `freq.get(lc) == 0` hone par map se **remove** karna zaroori hai, warna `freq.size()` galat count karega (0-count entries bhi "distinct char" gin li jaayengi).

---

## Dry run — `s = "aabacbebebe", k = 3`

Index:&nbsp; `0:a 1:a 2:b 3:a 4:c 5:b 6:e 7:b 8:e 9:b 10:e`

| right | char | freq map (after add) | size | shrink? | left | window (size==k?) | maxLen |
|:---:|:---:|---|:---:|---|:---:|---|:---:|
| 0 | a | {a:1} | 1 | no | 0 | — | -1 |
| 1 | a | {a:2} | 1 | no | 0 | — | -1 |
| 2 | b | {a:2,b:1} | 2 | no | 0 | — | -1 |
| 3 | a | {a:3,b:1} | 2 | no | 0 | — | -1 |
| 4 | c | {a:3,b:1,c:1} | 3 | no | 0 | ✅ len=5 `"aabac"` | 5 |
| 5 | b | {a:3,b:2,c:1} | 3 | no | 0 | ✅ len=6 `"aabacb"` | 6 |
| 6 | e | {a:3,b:2,c:1,e:1} | 4 | **yes** | shrink `a`×3, `b`→1 ... left goes 0→4 | left=4 | ✅ len=3 `"cbe"` | 6 |
| 7 | b | {c:1,b:2,e:1} | 3 | no | 4 | ✅ len=4 `"cbeb"` | 6 |
| 8 | e | {c:1,b:2,e:2} | 3 | no | 4 | ✅ len=5 `"cbebe"` | 6 |
| 9 | b | {c:1,b:3,e:2} | 3 | no | 4 | ✅ len=6 `"cbebeb"` | 6 |
| 10 | e | {c:1,b:3,e:3} | 3 | no | 4 | ✅ len=7 `"cbebebe"` | **7** |

**Result:** `7` ✅ (substring `"cbebebe"`)

> Row 6 mein shrink 4 baar chalti hai (`left` 0→1→2→3→4) kyunki `a` ke saare 3 occurrences aur ek extra step lagta hai `size` ko `4` se `3` tak laane mein — sirf ek `a` count 0 hote hi map se `a` remove hota hai aur size 3 ho jaata hai (`{b:1,c:1,e:1}` jaisa kuch beech mein), phir left = 4 pe rukta hai.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force (nested loop + Set) | `O(n²)` | `O(k)` | fresh set har start ke liye |
| Sliding Window + HashMap | `O(n)` | `O(k)` | har char zyada se zyada ek baar add, ek baar remove |

---

## Gotchas 🪤

- **`size == k` check `if`, na ki `while` ke andar** — length sirf tab record hoti hai jab window mein *exactly* `k` distinct chars hain, shrink loop ke turant baad.
- **0-count entries map se remove karo** — warna `freq.size()` "kitne distinct chars *currently* window mein hain" ki jagah "kitne chars *kabhi* dekhe" ban jaayega.
- **`k = 0` edge case** — koi bhi non-empty window `0` distinct nahi rakh sakti (empty string ke alawa), to answer `-1` (ya `0` agar khaali substring allowed ho — GFG mein `-1`).
- **`k >` string ke total distinct characters** — kabhi `freq.size() == k` true hi nahi hoga, poora loop chalega aur `maxLen = -1` hi rahega.
- **Sirf `> k` pe shrink karo, `>= k` pe nahi** — `size == k` bilkul valid state hai, use shrink mat karo warna window unnecessarily chhoti ho jaayegi.
