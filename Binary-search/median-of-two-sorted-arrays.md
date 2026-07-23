# Median of Two Sorted Arrays — LeetCode 4

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/median-of-two-sorted-arrays.html)**
> _(partition-based binary search — live left/right split of BOTH arrays, synced Java code)_

**Difficulty:** Hard  ·  **Pattern:** Binary Search on **Partition Point**  ·  **Tags:** Array, Binary Search, Divide & Conquer

---

## Problem

Do sorted arrays `nums1` aur `nums2` diye hain. Dono ko combine karke unka **median** dhoondo — **bina actually merge kiye**, `O(log(min(m, n)))` mein.

```
Input:  nums1 = [1, 3], nums2 = [2]
Output: 2.0
(merged: [1,2,3], median = 2)

Input:  nums1 = [1, 2], nums2 = [3, 4]
Output: 2.5
(merged: [1,2,3,4], median = (2+3)/2)
```

---

## The story (yaad rakhne ke liye) 🧠

**⚠️ Sabse tricky problem is folder ki — poore array ko merge kiye bina median chahiye.** Iske liye hum ek **naya idea** use karte hain: **binary search kisi element pe nahi, balki ek "PARTITION POINT" pe.**

Socho dono arrays ko **ek hi combined sorted line** mein rakh diya (bina actually merge kiye). Median ka matlab hai: is line ko **do halves** mein todna — **left half** aur **right half**, jahaan:
- `left half` ke saare elements `<=` `right half` ke saare elements.
- Dono halves ka size (roughly) barabar ho.

Hum yeh split **dono arrays ko EK SAATH** partition karke banate hain: `nums1` ko `cut1` pe kaato, `nums2` ko `cut2` pe kaato, aisa ki **`cut1 + cut2` = total left-half size**. Sirf `cut1` (`0` se `nums1.length` tak) pe **binary search** karo — `cut2` automatically formula se nikal jaata hai (`cut2 = totalLeft - cut1`).

**Kaise pata chale ki partition sahi hai?** Chaar boundary values nikaalo:
- `l1` = `nums1`'s left-half ka last element (ya `-infinity` agar left half empty hai)
- `l2` = `nums2`'s left-half ka last element (ya `-infinity`)
- `r1` = `nums1`'s right-half ka first element (ya `+infinity` agar right half empty hai)
- `r2` = `nums2`'s right-half ka first element (ya `+infinity`)

Partition **valid** hai jab `l1 <= r2` **aur** `l2 <= r1` (yaani dono arrays ke left-half elements, dono arrays ke right-half elements se chhote-ya-barabar hain — cross-check).

> 🔑 **Agar `l1 > r2`:** `nums1` ka partition **bahut right** mein hai (bahut zyada elements le liye) → `cut1` **ghatao** (`hi = cut1 - 1`).
> **Agar `l2 > r1`:** `nums1` ka partition **bahut left** mein hai (kam elements liye) → `cut1` **badhao** (`lo = cut1 + 1`).

Jab valid partition mil jaaye: agar total length **odd** hai, median = `max(l1, l2)` (left half ka sabse bada). Agar **even** hai, median = `(max(l1,l2) + min(r1,r2)) / 2`.

**Chhote array pe binary search karo** (`nums1` ko humesha chhota rakho, swap kar do agar zaroorat ho) — isse `O(log(min(m,n)))` milta hai.

---

## Approach 1 — Brute force 🟥

Dono arrays merge karo, sort karo (ya merge-sort style combine), middle nikaalo.

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int[] merged = new int[nums1.length + nums2.length];
        int i = 0, j = 0, k = 0;
        while (i < nums1.length && j < nums2.length) {
            merged[k++] = (nums1[i] <= nums2[j]) ? nums1[i++] : nums2[j++];
        }
        while (i < nums1.length) merged[k++] = nums1[i++];
        while (j < nums2.length) merged[k++] = nums2[j++];

        int n = merged.length;
        if (n % 2 == 1) return merged[n / 2];
        return (merged[n / 2 - 1] + merged[n / 2]) / 2.0;
    }
}
```

**Problem kya hai:** `O(m + n)` time aur space — problem explicitly `O(log(min(m,n)))` maangta hai, merge karna required se bahut zyada kaam hai.

---

## Approach 2 — Optimal (Binary Search on Partition Point) ✅

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        // hamesha chhote array pe binary search karo
        if (nums1.length > nums2.length) {
            return findMedianSortedArrays(nums2, nums1);
        }

        int n1 = nums1.length, n2 = nums2.length;
        int lo = 0, hi = n1;
        int totalLeft = (n1 + n2 + 1) / 2;   // left half ka target size

        while (lo <= hi) {
            int cut1 = lo + (hi - lo) / 2;    // nums1 mein kitne elements left half mein
            int cut2 = totalLeft - cut1;       // nums2 mein bacha hua left half

            int l1 = (cut1 == 0) ? Integer.MIN_VALUE : nums1[cut1 - 1];
            int l2 = (cut2 == 0) ? Integer.MIN_VALUE : nums2[cut2 - 1];
            int r1 = (cut1 == n1) ? Integer.MAX_VALUE : nums1[cut1];
            int r2 = (cut2 == n2) ? Integer.MAX_VALUE : nums2[cut2];

            if (l1 <= r2 && l2 <= r1) {
                // valid partition mil gaya!
                if ((n1 + n2) % 2 == 0) {
                    return (Math.max(l1, l2) + Math.min(r1, r2)) / 2.0;
                } else {
                    return Math.max(l1, l2);
                }
            } else if (l1 > r2) {
                hi = cut1 - 1;   // nums1 se kam elements lo
            } else {
                lo = cut1 + 1;   // nums1 se zyada elements lo
            }
        }

        throw new IllegalArgumentException("Input arrays not sorted");
    }
}
```

---

## Dry run — `nums1 = [1, 2]`, `nums2 = [3, 4]` (`n1=2, n2=2, totalLeft=2`)

| lo | hi | cut1 | cut2 | l1, l2, r1, r2 | Valid? | Verdict |
|:---:|:---:|:---:|:---:|:---|:---:|:---|
| 0 | 2 | 1 | 1 | `1, 3, 2, 4` | ❌ (`l2=3 > r1=2`) | `lo = 2` |
| 2 | 2 | 2 | 0 | `2, -∞, +∞, 3` | ✅ | valid! total even → `(max(2,-∞)+min(∞,3))/2 = (2+3)/2` |

**Result:** `2.5` ✅ (merged `[1,2,3,4]`, median `= (2+3)/2 = 2.5`)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Merge both arrays | `O(m + n)` | `O(m + n)` |
| Binary Search on Partition | `O(log(min(m, n)))` | `O(1)` |

---

## Gotchas 🪤

- **Hamesha chhote array pe binary search karo** — agar `nums1.length > nums2.length`, arguments **swap** karke recurse/loop karo. Bade array pe search karne se `cut2` negative ho sakta hai (invalid index) aur complexity bhi worse ho jaati hai.
- **`Integer.MIN_VALUE` / `MAX_VALUE` sentinels** — jab left ya right half khaali ho (`cut == 0` ya `cut == n`), sentinel values use karo taaki comparisons hamesha sahi behave karein, edge-case special-casing na karni pade.
- **`totalLeft = (n1+n2+1)/2`** — yeh formula **dono** odd aur even total length ke liye kaam karta hai. Odd ke liye left half ek extra element leta hai (jahaan se median seedha milta hai); even ke liye dono halves barabar hote hain.
- **`l1 > r2` vs `l2 > r1` mein confuse mat ho** — `l1 > r2` ka matlab `nums1` ne bahut zyada le liya (`cut1` ghatao, `hi = cut1-1`); `l2 > r1` (ya equivalently condition ka `else` branch) ka matlab `nums1` ne kam liya (`cut1` badhao, `lo = cut1+1`). Dono ko ulta likhna is problem ka sabse common bug hai.
- **Empty array edge case** — agar ek array bilkul khaali hai (`n1 = 0`), `lo=0, hi=0`, loop sirf ek baar chalega aur `cut1` hamesha `0` rahega — `cut2 = totalLeft` seedha doosre array mein sahi jagah point karega. Extra special-casing ki zaroorat nahi, formula khud handle karta hai.
- **`(max(l1,l2) + min(r1,r2)) / 2.0`** — `2.0` likhna mat bhoolo (integer division se bachne ke liye), warna even-length case mein galat (truncated) answer milega.
- **Yeh is folder ka sabse hard problem hai** — agar pehli baar mein samajh na aaye to normal hai. Partition-point binary search ek **fundamentally different** flavor hai (index ya value pe search nahi, balki **split configuration** pe search) — isse dobara-dobara dry-run karke intuition banao.
