# Count Triplets with Sum Smaller than X — GFG

> 🔗 **[Interactive Visualizer dekho »]
> (https://yashika807.github.io/DSA-PatternWise/Two-pointer/triplets-with-smaller-sum.html)**
> _(step-through Cart Anchor + bulk-approve batch counting, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sort + Two Pointers (batch counting)  ·  **Tags:** Array, Two Pointers, Sorting, Counting

---

## Problem

Ek integer array `arr` aur ek `target` diya hai. Count karo **kitne triplets** `(i, j, k)` (`i < j < k`) aise hain jinka sum **`target` se strictly kam** ho.

```
Input:  arr = [5, 1, 3, 4, 7], target = 12
Output: 4
// Valid triplets: (1,3,4)=8, (1,3,5)=9, (1,3,7)=11, (1,4,5)=10
```

---

## The story (yaad rakhne ke liye) 🛒

Ek **shopping cart** socho jismein already ek item (`arr[i]`, **Anchor**) daala hua hai, aur tumhara **budget** (`target`) fix hai. Ab do aur items add karne hain.

| Role | Kaun | Kaam |
|---|---|---|
| **Anchor** (`i`) | cart mein pehla item, fixed | "yeh item already cart mein hai" |
| **Cheap End** (`left`) | sabse sasta option | "isse add karke dekhte hain" |
| **Price Ceiling** (`right`) | sabse mehnga option (test) | "sabse mehnga item — agar yeh fit hota hai to sab fit honge" |

Trick yeh hai: agar `arr[i] + arr[left] + arr[right] < target` (**Price Ceiling bhi budget mein fit ho gaya**), to array **sorted** hai, isliye `left` aur `right` ke **beech ka har item** `right` se **sasta ya barabar** hai. Matlab **`right - left` combinations** (item `left` ke saath, kisi bhi index `left+1` se `right` tak) **sab budget mein fit hongi** — inhe ek saath **bulk-approve** kar do, gin lo, aage badho.

- **sum &lt; target** (ceiling bhi fit) → `count += (right - left)` bulk mein approve, phir `left++` (aur sasta combo explore karo)
- **sum &ge; target** (ceiling bahut mehnga) → `right--` (ek sasta option try karo)

---

## Approach 1 — Brute force 🟥

Teeno loops se har triplet check karo.

```java
class Solution {
    public int countTriplets(int[] arr, int target) {
        int n = arr.length;
        int count = 0;

        for (int i = 0; i < n; i++)
            for (int j = i + 1; j < n; j++)
                for (int k = j + 1; k < n; k++)
                    if (arr[i] + arr[j] + arr[k] < target)
                        count++;

        return count;
    }
}
```

**Problem kya hai:** `O(n³)` — sorting ka fayda uthaya hi nahi, aur "batch counting" ka trick miss ho gaya jisse pura `O(n)` chunk ek saath count ho sakta tha.

---

## Approach 2 — Sort + Two Pointers, batch count (Optimal) ✅

```java
class Solution {
    public int countTriplets(int[] arr, int target) {
        Arrays.sort(arr);              // sort so "right fits → everyone between fits"
        int n = arr.length;
        int count = 0;

        for (int i = 0; i < n - 2; i++) {
            int left = i + 1, right = n - 1;

            while (left < right) {
                int sum = arr[i] + arr[left] + arr[right];

                if (sum < target) {
                    // arr[left+1..right] are all <= arr[right], so all of them
                    // paired with arr[left] also satisfy sum < target — bulk approve!
                    count += right - left;
                    left++;
                } else {
                    right--;    // ceiling too expensive → try a cheaper one
                }
            }
        }

        return count;
    }
}
```

### Kyun `count += right - left` sahi hai

- Array sorted hai, isliye `arr[left] <= arr[left+1] <= ... <= arr[right]`.
- Agar `arr[i] + arr[left] + arr[right] < target` (sabse mehnga combo bhi fit ho gaya), to `arr[i] + arr[left] + arr[k]` for any `k` in `(left, right]` **definitely `< target`** hoga (kyunki `arr[k] <= arr[right]`).
- Aise `k` ki count = `right - left` (indices `left+1` se `right` tak).

---

## Dry run — `[5, 1, 3, 4, 7]`, `target = 12`

Sorted: **`[1, 3, 4, 5, 7]`**

| i | left, right | arr[i]+arr[left]+arr[right] | sum vs target | Action | count (after) |
|:---:|:---:|:---:|:---:|:---|:---:|
| 0 (`1`) | 1, 4 | 1+3+7=11 | < 12 | bulk approve `right-left=3`, `left++` | 3 |
| 0 (`1`) | 2, 4 | 1+4+7=12 | &ge; 12 | `right--` | 3 |
| 0 (`1`) | 2, 3 | 1+4+5=10 | < 12 | bulk approve `right-left=1`, `left++` | 4 |
| 0 (`1`) | 3, 3 | — | `left==right` | stop inner loop | 4 |
| 1 (`3`) | 2, 4 | 3+4+7=14 | &ge; 12 | `right--` | 4 |
| 1 (`3`) | 2, 3 | 3+4+5=12 | &ge; 12 | `right--` | 4 |
| 1 (`3`) | 2, 2 | — | `left==right` | stop | 4 |
| 2 (`4`) | 3, 4 | 4+5+7=16 | &ge; 12 | `right--` | 4 |
| 2 (`4`) | 3, 3 | — | `left==right` | stop | 4 |

**Result:** `count = 4` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (3 loops) | `O(n³)` | `O(1)` | checks every triplet individually |
| Sort + Two Pointers (batch) | `O(n²)` | `O(1)` extra* | sort `O(n log n)`, dominated by `n²` scan |

\* sort ke recursion stack ko chhod kar.

---

## Gotchas 🪤

- **`count += right - left`, na ki `count++`** — yehi is problem ka core trick hai; agar bhool gaye to tum wapas `O(n³)`-jaisi individual counting kar rahe hoge, ya galat answer aayega.
- **Sort pehle zaroori** — bina sorting ke "beech ke sab items bhi fit honge" waali guarantee nahi milti.
- **Strict `<`, na ki `<=`** — problem "strictly smaller than target" maangta hai; agar `<=` use kar doge to galat (zyada) count aayega.
- **`i < n - 2`** — anchor ke aage kam se kam do elements bache hone chahiye left/right ke liye.
- **Overflow with large arrays** — agar array bahut bada ho aur values bhi badi, `count` ek `int` mein overflow ho sakta hai (`n` bade hone pe `n³` tak triplets possible hain) — competitive settings mein `long` use karna safer hai.
