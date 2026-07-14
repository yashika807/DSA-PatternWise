# Substring with Concatenation of All Words — LeetCode 30 (Hard)

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Sliding-Window/words-concatenation.html)**
> _(step-through word-chunk sliding window, live train-car readout, synced Java code)_

**Difficulty:** Hard  ·  **Pattern:** Sliding Window (word-granularity, multiple offsets)  ·  **Tags:** String, Hash Map, Sliding Window

---

## Problem

Ek string `s` aur ek array `words` diya hai — **saare words ki length equal hai** (`wordLen`). `s` mein woh **saare starting indices** dhoondo jaha se ek substring ban rahi hai jo `words` ke **saare words ka ek concatenation** ho (**har word exactly ek baar**, **kisi bhi order mein**).

```
Input:  s = "barfoothefoobarman", words = ["foo","bar"]
Output: [0, 9]
Reason: s[0..5] = "barfoo" (bar+foo) aur s[9..14] = "foobar" (foo+bar) — dono valid concatenations
```

---

## The story (yaad rakhne ke liye) 🧠

Ab tak humari sliding window **character-by-character** chal rahi thi. Yeh problem ek **twist** deti hai: yahan **ek "unit" ek poora word hai**, character nahi — jaise ek **train** jismein har **box-car ki length fixed hai** (`wordLen`).

Tumhe track ke ek stretch pe **exactly woh saare box-cars** (kisi bhi order mein, har ek exactly ek baar) jodni hain jo tumhari **shopping list** (`words`) mein hain. Yeh `minimum-window-substring` jaisa hi need/have/count tracking hai, bas granularity **character ki jagah "word chunk"** hai.

**Sabse bada gotcha:** words `wordLen` ke multiples pe start ho sakte hain, par `s` mein **starting offset** `0` se `wordLen-1` tak kuch bhi ho sakta hai! Isliye humein **`wordLen` alag-alag "tracks"** try karni padti hain — offset `0` se shuru karke chunk karo, phir offset `1` se, ... offset `wordLen-1` tak. Har offset apni khud ki independent sliding window chalata hai.

- `need` = shopping list (word → kitni baar chahiye)
- `window` = current chunk-window mein kaunse words hain
- `count` = kitne total words (with multiplicity) abhi match ho rahe hain
- Jab **koi chunk `need` mein hai hi nahi** → **poori window reset karo** (yeh ek deal-breaker hai, kisi bhi valid window mein yeh chunk nahi ho sakta)
- Jab **ek word zyada baar aa gaya** (`window[w] > need[w]`) → shrink karo left se **jab tak duplicate na nikal jaaye**
- Jab **`count == numWords`** → match mila! Record karo, phir **left ko ek word aage khiska do** (agla match dhoondne ke liye, jaise `fruits-into-baskets` mein continue karna)

---

## Approach 1 — Brute force 🟥

Har starting index pe `totalLen` ki substring nikaalo, usko `wordLen`-size chunks mein todo, check karo woh `words` ka hi ek permutation hai.

```java
class Solution {
    public List<Integer> findSubstring(String s, String[] words) {
        List<Integer> result = new ArrayList<>();
        int wordLen = words[0].length();
        int numWords = words.length;
        int totalLen = wordLen * numWords;
        int n = s.length();

        Map<String, Integer> need = new HashMap<>();
        for (String w : words) need.put(w, need.getOrDefault(w, 0) + 1);

        for (int i = 0; i + totalLen <= n; i++) {
            Map<String, Integer> seen = new HashMap<>();
            int j = 0;
            for (; j < numWords; j++) {
                String w = s.substring(i + j * wordLen, i + (j + 1) * wordLen);
                seen.put(w, seen.getOrDefault(w, 0) + 1);
                if (seen.get(w) > need.getOrDefault(w, 0)) break;   // already too many
            }
            if (j == numWords) result.add(i);
        }
        return result;
    }
}
```

**Problem kya hai:** `O(n · numWords · wordLen)` — har starting index ke liye fresh chunk-check. Bade inputs pe slow, kyunki overlapping windows ka kaam dobara ho raha hai.

---

## Approach 2 — Optimal (Word-granularity Sliding Window per Offset) ✅

Har offset (`0` se `wordLen-1`) ke liye ek independent sliding window chalao — chunks ko ek "unit" maan ke grow/shrink karo.

```java
class Solution {
    public List<Integer> findSubstring(String s, String[] words) {
        List<Integer> result = new ArrayList<>();
        if (words.length == 0) return result;

        int wordLen = words[0].length();
        int numWords = words.length;
        int n = s.length();

        Map<String, Integer> need = new HashMap<>();
        for (String w : words) need.put(w, need.getOrDefault(w, 0) + 1);

        // s mein words wordLen ke multiples pe alag-alag "offset" se start ho sakte hain
        for (int offset = 0; offset < wordLen; offset++) {
            int left = offset, count = 0;
            Map<String, Integer> window = new HashMap<>();

            for (int right = offset; right + wordLen <= n; right += wordLen) {
                String w = s.substring(right, right + wordLen);

                if (need.containsKey(w)) {
                    window.put(w, window.getOrDefault(w, 0) + 1);
                    count++;

                    // yeh word already zyada baar aa chuka — duplicate hatao jab tak theek na ho
                    while (window.get(w) > need.get(w)) {
                        String leftWord = s.substring(left, left + wordLen);
                        window.put(leftWord, window.get(leftWord) - 1);
                        count--;
                        left += wordLen;
                    }

                    // saare words ka count match ho gaya — match mila!
                    if (count == numWords) {
                        result.add(left);
                        String leftWord = s.substring(left, left + wordLen);
                        window.put(leftWord, window.get(leftWord) - 1);
                        count--;
                        left += wordLen;               // agle match ke liye slide karo
                    }
                } else {
                    // yeh chunk list mein hai hi nahi — poori window ke liye deal-breaker
                    window.clear();
                    count = 0;
                    left = right + wordLen;
                }
            }
        }

        return result;
    }
}
```

> ⚠️ Jab koi unknown word milta hai, hum **poori window reset** karte hain (`window.clear()`, `left = right + wordLen`) — kyunki koi bhi valid window jo is unknown chunk ko include kare wo bilkul invalid hai, isse partial credit ki koi tuk nahi.

---

## Dry run — `s = "barfoothefoobarman", words = ["foo","bar"]`

`wordLen = 3`, `numWords = 2`, `need = {foo:1, bar:1}`

**Offset 0** (chunks start karte hain 0,3,6,9,...): `bar|foo|the|foo|bar|man`

| right | chunk | in need? | window | count | action | result |
|:---:|:---:|---|---|:---:|---|---|
| 0 | "bar" | yes | {bar:1} | 1 | grow | — |
| 3 | "foo" | yes | {bar:1,foo:1} | **2** | count==numWords → **match at left=0**; slide: remove "bar"(left=0), left=3 | **[0]** |
| 6 | "the" | no | reset: window={}, count=0, left=9 | 0 | reset (deal-breaker) | [0] |
| 9 | "foo" | yes | {foo:1} | 1 | grow | [0] |
| 12 | "bar" | yes | {foo:1,bar:1} | **2** | count==numWords → **match at left=9**; slide left=12 | **[0, 9]** |
| 15 | "man" | no | reset | 0 | reset | [0, 9] |

**Offset 1 aur 2** apni khud ki chunk-boundaries try karenge (`arf|oot|hef|...` aur `rfo|oth|efo|...`) — is example mein koi match nahi milega un offsets pe.

**Final Result:** `[0, 9]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(n · numWords · wordLen)` | `O(numWords)` | fresh chunk-check har start ke liye |
| Word-granularity Sliding Window | `O(wordLen · (n / wordLen))` ≈ `O(n)` | `O(numWords)` | har offset apna window ek baar scan karta hai |

---

## Gotchas 🪤

- **`wordLen` offsets try karna zaroori hai** — sirf offset `0` se chunk karoge to beech ke valid matches miss ho jaayenge (words kisi bhi character position pe start ho sakte hain, sirf apas mein `wordLen` ke multiple pe aligned hone chahiye).
- **Unknown chunk = full reset**, sirf shrink nahi — `while` loop se ek-ek karke nikalna galat hoga, poori window turant invalid ho jaati hai.
- **Match milne ke baad bhi `count--` aur `left += wordLen`** — taaki overlapping ya adjacent matches bhi mil sake (jaise `string-anagrams` mein continue karna).
- **`words.length == 0`** ya **`s.length() < wordLen * numWords`** — turant empty result return karo.
- **Duplicate words in `words` array** — `need` map counts store karta hai, isliye `["foo","foo"]` ka matlab hai "foo" do baar chahiye; `window[w] > need[w]` check yehi duplicate-handling enable karta hai.
