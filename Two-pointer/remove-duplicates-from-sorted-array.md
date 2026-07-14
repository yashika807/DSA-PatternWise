# Remove Duplicates from Sorted Array — LeetCode 26

> 🔗 **[Interactive Visualizer dekho »]
> (https://yashika807.github.io/DSA-PatternWise/Two-pointer/remove-duplicates-from-sorted-array.html)**
> _(step-through Shelf Writer + Scanner deduping, live shelf readout, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** Two Pointers (slow/fast, in-place)  ·  **Tags:** Array, Two Pointers, In-place

---

## Problem

Ek **sorted (ascending)** integer array `nums` diya hai. Duplicates ko **in-place** remove karo taaki har unique element **sirf ek baar** aaye, relative order maintain rahe. Naye length `k` return karo — pehle `k` elements hi answer maane jaate hain, baaki array ka content matter nahi karta.

```
Input:  nums = [0,0,1,1,1,2,2,3,3,4]
Output: k = 5, nums = [0,1,2,3,4,_,_,_,_,_]
```

---

## The story (yaad rakhne ke liye) 📚

Ek **library shelf** socho jahan books **sorted by title** hain, lekin kuch duplicate copies bhi rakhi hain. Ek **Shelf Writer** hai jo sirf **unique books** ko finalize karta hai, aur ek **Scanner** poori shelf padhta jaata hai.

| Role | Kaun | Kaam |
|---|---|---|
| **Shelf Writer** (`slow`) | last confirmed-unique book ka index | "yahan tak sab unique hai, verified" |
| **Scanner** (`fast`) | aage padhta jaata hai | "yeh next book dekh, naya title hai kya?" |

Scanner har book padhta hai:

- **Same title jo Shelf Writer ke paas hai** → duplicate copy hai, ignore karo, sirf `fast++`.
- **Naya title mila** → Shelf Writer ek slot aage badhta hai (`slow++`) aur naya title wahan **likhta** hai (`nums[slow] = nums[fast]`).

Jab Scanner poori shelf padh le, Shelf Writer ki position (`slow + 1`) hi final unique-book count hai.

---

## Approach 1 — Brute force (extra space) 🟥

`Set`/`LinkedHashSet` mein unique values daalo, phir wapas array mein copy karo.

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        LinkedHashSet<Integer> set = new LinkedHashSet<>();
        for (int num : nums) set.add(num);

        int i = 0;
        for (int val : set) nums[i++] = val;

        return set.size();
    }
}
```

**Problem kya hai:** `O(n)` extra space use ho raha hai jab ki array **already sorted** hai — duplicates hamesha **adjacent** honge. Iska matlab hai humein extra data structure ki zaroorat hi nahi, sirf do pointers se in-place kaam ho sakta hai.

---

## Approach 2 — Two Pointers / Slow-Fast (Optimal) ✅

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums.length == 0) return 0;

        int slow = 0;   // last confirmed-unique index (Shelf Writer)

        for (int fast = 1; fast < nums.length; fast++) {
            if (nums[fast] != nums[slow]) {
                slow++;                    // new unique slot
                nums[slow] = nums[fast];   // write it
            }
            // nums[fast] == nums[slow] → duplicate, skip (fast just moves on)
        }

        return slow + 1;   // count of unique elements
    }
}
```

### Kyun sorted property zaroori hai

- Sorted array mein saare duplicates **ek doosre ke paas** hote hain — isliye sirf `nums[slow]` (last unique) se compare karna kaafi hai, poora seen-set track karne ki zaroorat nahi.
- `slow` hamesha `fast` se peeche ya barabar rehta hai, kabhi aage nahi — writes hamesha "already scanned" region mein hoti hain, isliye future data overwrite nahi hota.

---

## Dry run — `[0,0,1,1,1,2,2,3,3,4]`

| fast | nums[fast] | nums[slow] | Same? | Action | slow (after) |
|:---:|:---:|:---:|:---:|:---|:---:|
| 1 | 0 | 0 | yes | skip | 0 |
| 2 | 1 | 0 | no | slow++, write | 1 |
| 3 | 1 | 1 | yes | skip | 1 |
| 4 | 1 | 1 | yes | skip | 1 |
| 5 | 2 | 1 | no | slow++, write | 2 |
| 6 | 2 | 2 | yes | skip | 2 |
| 7 | 3 | 2 | no | slow++, write | 3 |
| 8 | 3 | 3 | yes | skip | 3 |
| 9 | 4 | 3 | no | slow++, write | 4 |

**Result:** `k = slow + 1 = 5`, array prefix = `[0,1,2,3,4]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Set + copy back | `O(n)` | `O(n)` | ignores sorted structure |
| Two Pointers (slow/fast) | `O(n)` | `O(1)` | single pass, in-place |

---

## Gotchas 🪤

- **Array must be sorted** — yeh technique sirf sorted array pe kaam karti hai kyunki duplicates adjacent guaranteed hain. Unsorted array pe pehle sort karna padega (ya HashSet approach use karo).
- **Return value `k`, array poora nahi** — LeetCode judge sirf pehle `k` elements check karta hai; baaki array ka content "don't care" hai, use uske baare mein worry mat karo.
- **Empty array edge case** — `nums.length == 0` pe seedha `0` return karo, warna `nums[slow]` access karte hi `ArrayIndexOutOfBounds`.
- **`slow` shuru `0` se, `fast` shuru `1` se** — dono `0` se start karoge to pehla comparison hamesha khud se hoga (useless), aur pehla element galti se duplicate maan liya jaayega.

### Related variants (isi pattern ka extension)

- **LeetCode 83 — Remove Duplicates from Sorted List:** yehi slow/fast idea, par array index ke bajaye **linked list node pointers** pe. Logic: `if (fast.val != slow.val) { slow.next = fast; slow = fast; }`, phir end mein `slow.next = null` karna zaroori hai warna list ke aakhri hisse mein purane duplicate nodes latak jaayenge.
- **LeetCode 80 — Remove Duplicates II (allow max 2 copies):** ab har unique value **do baar tak** allowed hai. Trick: naya element `nums[fast]` sirf `slow` se compare mat karo — `nums[slow - 1]` se compare karo (`if (nums[fast] != nums[slow-1]) { slow++; nums[slow]=nums[fast]; }`, `slow` ko `1` se init karo taaki pehle 2 elements hamesha safe rahein). Yeh generalize hota hai "allow `k` duplicates" tak: `nums[fast] != nums[slow-k]` compare karo.
