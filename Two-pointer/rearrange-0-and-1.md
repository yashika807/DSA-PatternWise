# Segregate 0s and 1s — GFG

> 🔗 **[Interactive Visualizer dekho »]
> (https://yashika807.github.io/DSA-PatternWise/Two-pointer/rearrange-0-and-1.html)**
> _(step-through Low Basket + High Scanner sorting, live bin readout, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** Two Pointers (partition)  ·  **Tags:** Array, Two Pointers, In-place

---

## Problem

Sirf `0` aur `1` waala ek binary array `arr` diya hai. Isko **in-place** rearrange karo taaki saare `0`s pehle aa jaayein aur saare `1`s uske baad — extra array use kiye bina, ek hi pass mein.

```
Input:  arr = [0, 1, 0, 1, 1, 0]
Output: [0, 0, 0, 1, 1, 1]
```

---

## The story (yaad rakhne ke liye) 🧺

Socho tumhare paas ek **laundry basket line** hai — `0`s ke liye ek section (light-color socks) aur `1`s ke liye doosra (dark-color socks), dono **same basket ke andar hi**, bas properly divide karke.

| Role | Kaun | Kaam |
|---|---|---|
| **Low Basket** (`low`) | jahan tak `0`s ki guarantee hai | "isse pehle sab kuch confirmed `0` hai" |
| **Scanner** (`i`) | poore array ko scan karta hai | "yeh element check karo — `0` ya `1`?" |

Scanner poore array mein chalta hai:

- **`arr[i] == 0`** → yeh sock light-section mein jaani chahiye → `arr[low]` se **swap** karo, phir `low++` aur `i++` (dono aage badhte hain, kyunki jo swap hoke `i` pe aaya woh already processed `0`/`1` tha)
- **`arr[i] == 1`** → yeh already sahi jagah pe hai (dark section mein hi rahegi) → bas `i++`, `low` ko chhedo mat.

`low` hamesha "next `0` yahan jaani chahiye" ki position batata hai — jaise ek dedicated basket-slot jo dheere-dheere bharta jaata hai.

> ⚠️ Yeh **Dutch National Flag** ka simpler (2-color) version hai — full 3-color (0,1,2) version is folder mein alag se `dutch-national-flag.md` mein hai.

---

## Approach 1 — Brute force 🟥

Count karo kitne `0`s hain, phir array ko overwrite karo.

```java
class Solution {
    public void segregate(int[] arr) {
        int n = arr.length;
        int countZero = 0;
        for (int num : arr) if (num == 0) countZero++;

        for (int i = 0; i < n; i++) {
            arr[i] = (i < countZero) ? 0 : 1;
        }
    }
}
```

**Problem kya hai:** Yeh kaam to karta hai `O(n)` mein, lekin **do passes** lagte hain (ek count ke liye, ek overwrite ke liye), aur yeh "single-pass swap-based partition" ka core two-pointer skill nahi dikhata jo interview mein test hoti hai.

---

## Approach 2 — Two Pointers / Single Pass (Optimal) ✅

`low` aur `i` dono `0` se shuru, ek hi pass mein partition karo.

```java
class Solution {
    public void segregate(int[] arr) {
        int low = 0;              // next slot for a 0
        int n = arr.length;

        for (int i = 0; i < n; i++) {
            if (arr[i] == 0) {
                // swap current 0 into the low-section
                int temp = arr[low];
                arr[low] = arr[i];
                arr[i] = temp;
                low++;
            }
            // arr[i] == 1 → already in the right zone, just move i
        }
    }
}
```

### Kyun `low <= i` hamesha safe hai

- `low` kabhi `i` se aage nahi ja sakta, kyunki `low` sirf tabhi badhta hai jab `i` bhi ek `0` process karke badh raha ho.
- Jab swap hota hai, `arr[low]` **jo bhi tha** (ya to already-seen `1`, ya khud `low`), woh `arr[i]` (jo abhi `0` mila) ke saath swap hoke `i` pe aa jaata hai. Chunki region `[low, i-1]` mein sirf `1`s hi hote hain (invariant), swapped-in value bhi `1` (ya khud), koi extra check nahi chahiye.

---

## Dry run — `[0, 1, 0, 1, 1, 0]`

| i | arr[i] | Action | low (after) | Array state |
|:---:|:---:|:---|:---:|:---|
| 0 | 0 | swap(arr[0],arr[0]), low++ | 1 | `[0,1,0,1,1,0]` |
| 1 | 1 | skip, i++ | 1 | `[0,1,0,1,1,0]` |
| 2 | 0 | swap(arr[1],arr[2]), low++ | 2 | `[0,0,1,1,1,0]` |
| 3 | 1 | skip, i++ | 2 | `[0,0,1,1,1,0]` |
| 4 | 1 | skip, i++ | 2 | `[0,0,1,1,1,0]` |
| 5 | 0 | swap(arr[2],arr[5]), low++ | 3 | `[0,0,0,1,1,1]` |

**Result:** `[0, 0, 0, 1, 1, 1]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Count + overwrite | `O(n)` | `O(1)` | two passes |
| Two Pointers (swap) | `O(n)` | `O(1)` | single pass, in-place swaps |

---

## Gotchas 🪤

- **Swap even when `low == i`** — self-swap is harmless, easier to write unconditionally than special-casing it.
- **`low` sirf `0` milne pe hi badhta hai** — `1` milne pe `low` ko touch mat karo, warna partition boundary galat ho jaayega.
- **Stability guarantee nahi hai** — relative order of `1`s ke andar shuffle ho sakta hai (swap-based approach stable nahi hai); agar stability chahiye to stable-partition / extra-array approach use karo.
- **In-place required** — extra array banake copy karna technically kaam karega but interview mein "O(1) extra space" expectation hoti hai, isliye swap-based hi standard answer hai.
- **Edge cases** — all-zeros ya all-ones array pe bhi loop safely chalta hai (koi swap trigger hi nahi hoga ya har element pe hoga), empty array pe loop bilkul skip.
