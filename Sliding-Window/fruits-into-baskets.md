# Fruits Into Baskets — LeetCode 904

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Sliding-Window/fruits-into-baskets.html)**
> _(step-through grow/shrink window, live 2-basket readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sliding Window (variable size, "at most 2" frequency map)  ·  **Tags:** Array, Hash Map, Sliding Window

---

## Problem

Ek row mein ped lage hain, `fruits[i]` batata hai `i`-th ped ka fruit **type**. Tumhare paas **exactly 2 baskets** hain — har basket mein **sirf ek type** ka fruit ja sakta hai (par unlimited quantity). Ek continuous stretch of trees se **maximum fruits** collect karo jo dono baskets mein fit ho jaayein.

```
Input:  fruits = [1, 2, 3, 2, 2]
Output: 4
Reason: [2, 3, 2, 2] — sirf types 2 aur 3, total 4 fruits
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh problem literally apni khud ki metaphor hai — **do baskets, do hi types allowed**. Tum ek continuous line mein ped ke paas se chal rahe ho, har ped se fruit pluck karte jaate ho (right edge grow).

Jab tak sirf **2 ya kam distinct types** tumhare paas hain, chalte raho — sab kuch valid hai. Jaise hi **3rd type** ka fruit aata hai, ek basket **overflow** ho jaata hai! Ab tumhe **peeche jaana padega** (left edge shrink) aur purane fruits phekne padenge jab tak wapas sirf 2 types na bach jaayein.

- **Frequency map** = do baskets — kaunsa type kitni quantity mein hai.
- `map.size() > 2` → overflow → shrink karo.
- **Har step** pe (chahe shrink hua ho ya na hua ho) current window ki length se **best** update karo — kyunki `size <= 2` hamesha valid hai (2 se kam types bhi chalta hai, "at most 2").

> Pichle problem (`longest-k-distinct`) se fark: wahan **exactly k** chahiye tha, yahan **at most 2** chalta hai. Isliye best update **har** valid step pe hota hai, sirf `size == k` pe nahi.

---

## Approach 1 — Brute force 🟥

Har start se aage badhte jaao jab tak 2 se zyada types na ho jaayein.

```java
class Solution {
    public int totalFruit(int[] fruits) {
        int n = fruits.length;
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            Set<Integer> types = new HashSet<>();
            int j = i;
            while (j < n && (types.size() < 2 || types.contains(fruits[j]))) {
                types.add(fruits[j]);
                j++;
            }
            maxLen = Math.max(maxLen, j - i);
        }
        return maxLen;
    }
}
```

**Problem kya hai:** `O(n²)` worst case — har start se dobara scan. Bade `n` pe slow.

---

## Approach 2 — Optimal (Sliding Window + Frequency Map) ✅

`right` se grow karo, `HashMap` mein counts track karo. `size() > 2` hote hi `left` se shrink karo. **Har** step pe length compare karo.

```java
class Solution {
    public int totalFruit(int[] fruits) {
        Map<Integer, Integer> basket = new HashMap<>();  // type -> count in window
        int left = 0, maxLen = 0;

        for (int right = 0; right < fruits.length; right++) {
            int type = fruits[right];
            basket.put(type, basket.getOrDefault(type, 0) + 1);   // pluck karo, basket mein daalo

            // 3rd type aa gaya → ek basket overflow, shrink karo
            while (basket.size() > 2) {
                int lt = fruits[left];
                basket.put(lt, basket.get(lt) - 1);
                if (basket.get(lt) == 0) {
                    basket.remove(lt);                             // basket ab khaali, type free
                }
                left++;
            }

            // window hamesha valid hai yahan (size <= 2) — har baar best check karo
            maxLen = Math.max(maxLen, right - left + 1);
        }

        return maxLen;
    }
}
```

> ⚠️ `maxLen` update **loop ke andar, `while` ke *baad*, har `right` step pe** hota hai — `longest-k-distinct` waale problem se ulta, yahan "exactly" ki koi extra `if` condition nahi chahiye.

---

## Dry run — `fruits = [1, 2, 3, 2, 2]`

| right | type | basket (after add) | size | shrink? | left | maxLen |
|:---:|:---:|---|:---:|---|:---:|:---:|
| 0 | 1 | {1:1} | 1 | no | 0 | 1 |
| 1 | 2 | {1:1,2:1} | 2 | no | 0 | 2 |
| 2 | 3 | {1:1,2:1,3:1} | 3 | **yes** — remove `1` (0 left), left=1 | 1 | — |
| | | {2:1,3:1} | 2 | stop shrink | 1 | 2 |
| 3 | 2 | {2:2,3:1} | 2 | no | 1 | 3 |
| 4 | 2 | {2:3,3:1} | 2 | no | 1 | **4** |

**Result:** `4` ✅ (window `[1,4]` = fruits `[2, 3, 2, 2]`)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(n²)` worst case | `O(1)` (max 2 types in set) | re-scans from every start |
| Sliding Window + HashMap | `O(n)` | `O(1)` (map size ≤ 3 at any point) | each tree added once, removed once |

---

## Gotchas 🪤

- **"At most 2", not "exactly 2"** — `maxLen` **har** `right` step pe update hota hai, `size == 2` ka wait mat karo. Single-type array (`[5,5,5]`) ka answer poora array hai.
- **0-count entries map se remove karo** — warna `basket.size()` galat "3 types" bata dega jab actually sirf 2 active hain.
- **`while`, not `if`** — ek `right` step ke baad zaroorat pade to left kai baar move ho sakta hai (yahan is example mein ek hi baar laga, par general case mein nahi).
- **Fruit "type" ek `int` label hai**, uski value se koi matlab nahi — do alag numbers matlab do alag types, chahe woh 1 aur 100 ho.
- **Basket = HashMap<type, count>**, isse count bhi milta hai (agar kabhi quantity chahiye ho) aur `size()` se distinct types bhi — same map do kaam karta hai.
