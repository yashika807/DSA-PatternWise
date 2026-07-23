# Peak Index in a Mountain Array — LeetCode 852

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Binary-search/peak-index-in-mountain-array.html)**
> _(step-through the uphill/downhill slope check, live summit readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Binary Search on Slope  ·  **Tags:** Array, Binary Search

---

## Problem

Ek **mountain array** `arr` diya hai — matlab ek index `peak` aisa hai ki:
- `arr[0] < arr[1] < ... < arr[peak]` (strictly badhta hai)
- `arr[peak] > arr[peak+1] > ... > arr[n-1]` (strictly ghatta hai)

`peak` ka **index** return karo.

```
Input:  arr = [0, 10, 5, 2]
Output: 1

Input:  arr = [24, 69, 100, 99, 79, 78, 67, 36, 26, 19]
Output: 2
```

Constraint: `O(log n)` chahiye — linear scan allowed nahi (well, chalega par interview mein nahi).

---

## The story (yaad rakhne ke liye) 🧠

Tum ek **pahaad chadh rahe ho**, aankhon pe patti bandhi hai — tumhe sirf yeh feel ho sakta hai ki agla kadam **upar** ja raha hai ya **neeche**. Tumhe **summit (chooti)** dhoondhni hai.

Har step pe tum apne current position (`mid`) se **ek kadam aage** (`mid+1`) ka height compare karte ho:

- **Agar aage ka kadam upar hai** (`arr[mid] < arr[mid+1]`) → tum abhi bhi **chadh rahe ho**, summit tumse **aage** hai. Poori left half (jahaan tak tum pahunche ho) ko bhool jao → `lo = mid + 1`.
- **Agar aage ka kadam neeche hai** (`arr[mid] >= arr[mid+1]`) → tum ya to **summit pe khade ho**, ya **utraai** shuru ho chuki hai. Summit **yahin ya peeche** hai, aage kuch nahi bacha → `hi = mid`.

> 🔑 **Note `hi = mid`, na ki `hi = mid - 1`** — `mid` khud summit ho sakta hai, use range se bahar mat phenko (bilkul `upper-bound-ceiling` jaisa trick).

Jab `lo == hi`, wahi summit hai — tumne pahaad khoj liya bina neeche dekhe!

---

## Approach 1 — Brute force 🟥

Left se right scan karo, jahaan `arr[i] > arr[i+1]` pehli baar ho, wahi peak hai.

```java
class Solution {
    public int peakIndexInMountainArray(int[] arr) {
        for (int i = 1; i < arr.length - 1; i++) {
            if (arr[i] > arr[i + 1]) return i;
        }
        return arr.length - 1;   // theoretically unreachable given constraints
    }
}
```

**Problem kya hai:** `O(n)` — mountain ki sorted-jaisi structure ka fayda nahi liya. `n = 10^5` tak bhi chalega, par asli maksad `O(log n)` hai.

---

## Approach 2 — Optimal (Binary Search on Slope) ✅

```java
class Solution {
    public int peakIndexInMountainArray(int[] arr) {
        int lo = 0, hi = arr.length - 1;

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;

            if (arr[mid] < arr[mid + 1]) {
                lo = mid + 1;      // still climbing → peak is ahead
            } else {
                hi = mid;          // descending (or at peak) → peak is here or behind
            }
        }

        return lo;   // == hi, the summit
    }
}
```

---

## Dry run — `arr = [0, 10, 5, 2]`

| lo | hi | mid | arr[mid], arr[mid+1] | Verdict |
|:---:|:---:|:---:|:---:|:---|
| 0 | 3 | 1 | 10, 5 | `10 >= 5` → descending → `hi = 1` |
| 0 | 1 | 0 | 0, 10 | `0 < 10` → climbing → `lo = 1` |
| 1 | 1 | — | — | `lo == hi`, **stop** |

**Result:** `1` ✅ (`arr[1] = 10` is the summit)

---

## Complexity

| Approach | Time | Space |
|---|:---:|:---:|
| Linear scan | `O(n)` | `O(1)` |
| Binary Search on Slope | `O(log n)` | `O(1)` |

---

## Gotchas 🪤

- **`arr[mid]` vs `arr[mid+1]`, na ki `arr[mid-1]`** — hamesha **aage** dekho, peeche nahi. Isse `mid+1` array ke andar rahe, isliye loop `lo < hi` hai (agar `mid == hi` ho jaaye to `mid+1` out of bounds ho sakta tha, par `lo<hi` guarantee karta hai `mid < hi <= n-1`, so `mid+1 <= n-1` hamesha safe).
- **`hi = mid`, na ki `hi = mid - 1`** — `mid` khud summit ho sakta hai. Isse range se poori tarah nikaalna galat answer dega.
- **Guaranteed unique peak** — problem guarantee karta hai strict increase phir strict decrease, koi plateau (`==`) nahi. Isliye `arr[mid] < arr[mid+1]` aur `arr[mid] > arr[mid+1]` ke beech koi third case nahi — simplifies the comparison to a simple `if/else`.
- **Peak kabhi index `0` ya `n-1` nahi hoti** (constraint) — isliye brute force ka fallback `return n-1` theoretically kabhi trigger nahi hota, bas safety ke liye hai.
- **Yeh technique `find-peak-element` (LC 162) ka special case hai** — wahaan array guaranteed mountain-shaped nahi hota (multiple local peaks ho sakte hain), par same binary-search-on-slope idea kaam karti hai. Dekho agli problem.
