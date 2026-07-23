# H-Index II — LeetCode 275

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/h-index-ii.html)**
> _(step-through the "tipping point" search over indices, live h-index readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search — Index Boundary Search  ·  **Tags:** Array, Binary Search

---

## Problem

Ek researcher ke `citations` array diya hai, jo **ascending order mein sorted** hai — `citations[i]` = `i`-vi paper ko mile citations. Uska **h-index** nikaalo: sabse bada `h` aisa ki researcher ke paas kam se kam `h` papers hain jinme se **har ek** ko `>= h` citations mile hain (aur baaki papers ko `<= h` citations mile hain).

`O(log n)` mein solve karo (array pehle se sorted hai).

```
Input:  citations = [0, 1, 3, 5, 6]
Output: 3
(3 papers with >= 3 citations each: values 3, 5, 6)

Input:  citations = [1, 2, 100]
Output: 2
(2 papers with >= 2 citations each: values 2, 100)
```

---

## The story (yaad rakhne ke liye) 🧠

Array **ascending sorted** hai, to socho isse **right se left** dekhte hain: agar hum index `i` pe khade hain, to **`n - i` papers** aise hain jinke citations `citations[i]` se **bade ya barabar** hain (kyunki array sorted hai, `i` se aage ke sab `>= citations[i]`).

Toh sawaal ban jaata hai: **sabse chhota index `i`** dhoondo jahaan `citations[i] >= n - i` ho — yahi woh **"tipping point"** hai jahaan se paper ke citations, uske right side ke papers ki ginti se overtake karna shuru karte hain. Wahaan se `h = n - i`.

> 🔑 **Yeh bhi ek boundary search hai, bas condition thodi twisted hai:** `upper-bound-ceiling` mein hum `nums[mid] >= target` check karte the (target ek fixed value tha). Yahaan target khud **`mid` par depend** karta hai (`n - mid`) — condition har step pe "move" karta hai. Phir bhi property **monotonic** hai (isliye binary search valid hai): jaise-jaise `i` badhta hai, `citations[i]` badhta/same rehta hai (sorted array) jabki `n - i` **ghatta** hai — dono opposite directions mein move karte hain, isliye ek clean crossing point guaranteed hai.

Agar koi bhi index yeh condition satisfy na kare (sab papers ke citations bahut kam hain), to `h = 0`.

---

## Approach 1 — Brute force 🟥

Har candidate `h` (n se 0 tak) try karo, dekho kitne papers `>= h` citations rakhte hain.

```java
class Solution {
    public int hIndex(int[] citations) {
        int n = citations.length;
        for (int h = n; h > 0; h--) {
            int count = 0;
            for (int c : citations) {
                if (c >= h) count++;
            }
            if (count >= h) return h;
        }
        return 0;
    }
}
```

**Problem kya hai:** `O(n^2)` — har candidate `h` ke liye poora array scan ho raha hai. Sorted hone ka fayda bilkul nahi liya.

---

## Approach 2 — Optimal (Binary Search on the index) ✅

```java
class Solution {
    public int hIndex(int[] citations) {
        int n = citations.length;
        int lo = 0, hi = n - 1;

        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;

            if (citations[mid] >= n - mid) {
                // is index se h-index bana sakte hain — par ek chhota index
                // (jyada papers) mile to h aur bhi bada ho sakta hai
                hi = mid - 1;
            } else {
                // itne kam citations, is index ko chhod ke aage badho
                lo = mid + 1;
            }
        }

        return n - lo;   // lo hi tipping point hai
    }
}
```

---

## Dry run — `citations = [0, 1, 3, 5, 6]` (`n = 5`)

| lo | hi | mid | citations[mid] | n - mid | Condition | Verdict |
|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| 0 | 4 | 2 | 3 | 3 | `3 >= 3` ✅ | candidate! smaller index? → `hi = 1` |
| 0 | 1 | 0 | 0 | 5 | `0 >= 5` ❌ | too few → `lo = 1` |
| 1 | 1 | 1 | 1 | 4 | `1 >= 4` ❌ | too few → `lo = 2` |
| 2 | 1 | — | — | — | `lo > hi`, **stop** |

**Result:** `h = n - lo = 5 - 2 = 3` ✅ (papers with citations `3, 5, 6` — three of them, each `>= 3`)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Try every h, count matches | `O(n^2)` | `O(1)` |
| Binary Search on index | `O(log n)` | `O(1)` |

---

## Gotchas 🪤

- **Condition depends on `mid`, not on a fixed `target`** — `n - mid` badalta rehta hai har step pe. Isse `upper-bound-ceiling` se confuse mat karo; yahan hum "compare against a moving target" kar rahe hain, lekin monotonicity phir bhi hold karti hai isliye binary search valid hai.
- **`hi = mid - 1`, na ki `hi = mid`** — yahaan hum closed-interval (`lo <= hi`) style use kar rahe hain (jaise `aggressive-cows`), isliye jab condition true ho tab bhi `mid` ko range se poora nikaalna hai — `n - lo` answer khud loop khatam hone ke baad compute hota hai, `mid` ko dobara touch karne ki zaroorat nahi.
- **Return `n - lo`, na ki `lo` khud** — bahut common mistake hai seedha `lo` return kar dena. Answer **h-index** hai, index nahi.
- **`h = 0` edge case** — agar koi bhi paper `>= 1` citation na rakhta ho, `lo` poora array scan karke `n` tak pahunch jaata hai, aur `n - n = 0` sahi answer hota hai — special-case likhne ki zaroorat nahi, formula khud handle karta hai.
- **Array already sorted ascending honi chahiye** — yeh LeetCode 275 (H-Index II) hai, jisme input guaranteed sorted hai. Agar unsorted diya ho (H-Index I, LeetCode 274), pehle sort karo ya counting-sort trick use karo — yeh `O(log n)` approach sirf sorted input pe hi lagu hoti hai.
