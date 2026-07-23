# Find Peak Element — LeetCode 162

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/find-peak-element.html)**
> _(step-through the uphill/downhill slope check on a bumpy range, live peak readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search on Slope  ·  **Tags:** Array, Binary Search

---

## Problem

Ek integer array `nums` diya hai jahaan `nums[i] != nums[i+1]` (adjacent elements kabhi barabar nahi). Koi bhi **peak element** ka index return karo — jahaan `nums[i] > nums[i-1]` aur `nums[i] > nums[i+1]` ho. Array ke **edges** ko `-∞` maano (matlab `nums[-1] = nums[n] = -∞`).

```
Input:  nums = [1, 2, 3, 1]
Output: 2   (nums[2] = 3 is a peak)

Input:  nums = [1, 2, 1, 3, 5, 6, 4]
Output: 1 or 5   (dono valid peaks hain — koi bhi ek chalega)
```

Constraint: `O(log n)` chahiye.

---

## The story (yaad rakhne ke liye) 🧠

`peak-index-in-mountain-array` mein ek **single, guaranteed mountain** thi — sirf ek chadhaai, ek utraai. Yahaan situation **badal gayi hai**: yeh ek **poora mountain RANGE** hai, fog mein, jisme **kai chotiyaan** ho sakti hain. Tumse koi ek specific peak nahi maanga — bas **"koi bhi ek chooti dhoondh do."**

> 🔑 **Yehi hai is problem ka asli twist:** array **sorted nahi hai**, na hi single mountain-shaped hai. Phir bhi wahi binary-search-on-slope trick kaam karti hai — kyun?

Socho tum ek dhundhle mountain range mein khade ho (kisi bhi random spot pe). Har step pe apne **agle kadam ka slope** dekho (`nums[mid]` vs `nums[mid+1]`):

- **Upar ja rahe ho** (`nums[mid] < nums[mid+1]`)? Us direction mein **guaranteed** ek peak milegi — kyunki agar tum chadhte hi jaate ho aur array khatam ho jaaye, to edge khud `-∞` hai, matlab kahin na kahin **neeche aana hi padega** — wahi ek peak hai. Toh **confidently right jao**: `lo = mid + 1`.
- **Neeche ja rahe ho ya barabar khade ho** (`nums[mid] >= nums[mid+1]`)? Same logic — is taraf (left mein, including `mid`) bhi guaranteed peak hai. `hi = mid`.

Dono directions mein **kam se kam ek peak ka guarantee** hai — bas humein **koi ek** chahiye, sabse badi nahi. Isliye greedy slope-following kaam karta hai, chahe array messy ho.

---

## Approach 1 — Brute force 🟥

Har index pe check karo ki woh peak hai ya nahi (edges ko `-∞` treat karke).

```java
class Solution {
    public int findPeakElement(int[] nums) {
        int n = nums.length;
        for (int i = 0; i < n; i++) {
            int left = (i == 0) ? Integer.MIN_VALUE : nums[i - 1];
            int right = (i == n - 1) ? Integer.MIN_VALUE : nums[i + 1];
            if (nums[i] > left && nums[i] > right) return i;
        }
        return -1;   // unreachable given constraints — a peak always exists
    }
}
```

**Problem kya hai:** `O(n)` — jabki problem `O(log n)` maangta hai, aur array ka koi structural fayda nahi liya.

---

## Approach 2 — Optimal (Binary Search on Slope) ✅

```java
class Solution {
    public int findPeakElement(int[] nums) {
        int lo = 0, hi = nums.length - 1;

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;

            if (nums[mid] < nums[mid + 1]) {
                lo = mid + 1;      // uphill → a peak is guaranteed to the right
            } else {
                hi = mid;          // downhill/flat-top → a peak is guaranteed here or left
            }
        }

        return lo;   // == hi, a valid peak
    }
}
```

---

## Dry run — `nums = [1, 2, 3, 1]`

| lo | hi | mid | nums[mid], nums[mid+1] | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 0 | 3 | 1 | 2, 3 | `2 < 3` → uphill → `lo = 2` |
| 2 | 3 | 2 | 3, 1 | `3 >= 1` → downhill → `hi = 2` |
| 2 | 2 | — | — | `lo == hi`, **stop** |

**Result:** `2` ✅ (`nums[2] = 3`)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear scan | `O(n)` | `O(1)` |
| Binary Search on Slope | `O(log n)` | `O(1)` |

---

## Gotchas 🪤

- **Array sorted hone ki zaroorat NAHI hai** — yeh binary search "monotonic property pe" chalta hai (uphill vs downhill), sorted-ness pe nahi. Bahut log yahaan confuse ho jaate hain ki binary search sirf sorted array pe chalti hai — galat!
- **Multiple peaks ho sakte hain, koi bhi ek valid hai** — algorithm greedy slope follow karta hai aur jo bhi peak pehle milta hai wahi return karta hai. LeetCode kisi specific peak ki demand nahi karta.
- **Edges `-∞` treat karo (mentally)** — code mein explicit nahi likhna padta kyunki loop `mid+1` hamesha `lo<hi` guard ki wajah se valid index hota hai, aur agar `lo` ya `hi` khud array ke end pe hain to bhi wahi "edge is -∞" guarantee automatically satisfy hoti hai.
- **`hi = mid`, na ki `hi = mid - 1`** — `mid` khud peak ho sakta hai, `peak-index-in-mountain-array` jaisa hi trick.
- **`nums[i] != nums[i+1]` guarantee** — is wajah se `<` aur `>=` mein koi ambiguous "equal" case nahi bachta comparison mein — simplifies to clean if/else.
