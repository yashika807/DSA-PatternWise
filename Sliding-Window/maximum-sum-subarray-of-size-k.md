# Maximum Sum Subarray of Size K — GFG

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Sliding-Window/maximum-sum-subarray-of-size-k.html)**
> _(step-through fixed-size window, live running-sum readout, synced Java code)_

**Difficulty:** Easy  ·  **Pattern:** Sliding Window (fixed size)  ·  **Tags:** Array, Sliding Window

---

## Problem

Ek integer array `arr[]` aur ek number `k` diya hai. Size-`k` ke **contiguous subarray** ka **maximum sum** nikalo.

```
Input:  arr = [100, 200, 300, 400], k = 2
Output: 700
Reason: subarray [300, 400] ka sum sabse zyada hai
```

---

## The story (yaad rakhne ke liye) 🧠

Socho tumhare paas ek **camera ka viewfinder** hai jiski width hamesha **fixed `k`** hai — na chhota hoga na bada, bas track ke upar left se right sarakta rahega.

Har frame mein tumhe **total brightness** (yaani sum) dikhta hai. Jab viewfinder ek step aage sarकता hai, ek naya pixel **andar aata hai** (right edge pe) aur ek purana pixel **bahar nikal jaata hai** (left edge se). Baaki sab pixels wahi ke wahi rehte hain — unko dobara ginne ki zaroorat nahi!

| Role | Kaun | Kaam |
|---|---|---|
| **Frame** | `[left, right]` | hamesha exactly `k` wide |
| **Enter** | `arr[right]` | naya pixel andar, sum mein `+` |
| **Exit** | `arr[left]` | purana pixel bahar, sum mein `-` |

Yehi hai fixed-size sliding window ka core idea: **puri window ko baar-baar sum mat karo — sirf jo change hua (ek entry, ek exit) usko update karo.**

---

## Approach 1 — Brute force 🟥

Har possible start index `i` se `k` elements ka sum nikalo, aur max track karo.

```java
class Solution {
    public int maxSumSubarray(int[] arr, int k) {
        int n = arr.length;
        int maxSum = Integer.MIN_VALUE;

        for (int i = 0; i + k <= n; i++) {
            int sum = 0;
            for (int j = i; j < i + k; j++) {
                sum += arr[j];          // har window fresh se sum ho rahi hai
            }
            maxSum = Math.max(maxSum, sum);
        }
        return maxSum;
    }
}
```

**Problem kya hai:** Har window ke `k` elements dobara se add ho rahe hain, jab ki sirf ek element enter/exit hua tha. `O(n·k)` time — `n`, `k` dono bade honge to slow.

---

## Approach 2 — Optimal (Sliding Window) ✅

Pehli window ka sum ek baar calculate karo. Uske baad har step pe sirf **exit element ghatao, enter element jodo** — O(1) update.

```java
class Solution {
    public int maxSumSubarray(int[] arr, int k) {
        int n = arr.length;

        // 1) first window [0, k-1] ka sum seed karo
        int windowSum = 0;
        for (int i = 0; i < k; i++) {
            windowSum += arr[i];
        }
        int maxSum = windowSum;

        // 2) window ko ek-ek step slide karo
        for (int right = k; right < n; right++) {
            int left = right - k;              // window ka left edge jo ab exit karega

            windowSum += arr[right];           // naya element andar aaya
            windowSum -= arr[left];            // purana element bahar gaya

            maxSum = Math.max(maxSum, windowSum);
        }

        return maxSum;
    }
}
```

> ⚠️ Window **kabhi shrink ya grow nahi** hoti — size hamesha `k` fixed. Isliye "left" pointer ko explicit maintain karne ki bhi zaroorat nahi, `right - k` se hi mil jaata hai.

---

## Dry run — `arr = [100, 200, 300, 400], k = 2`

| Step | right | left (= right−k) | Action | windowSum | maxSum |
|:---:|:---:|:---:|---|:---:|:---:|
| seed | — | — | `arr[0]+arr[1] = 100+200` | **300** | 300 |
| 1 | 2 | 0 | `+arr[2](300) −arr[0](100)` | **500** | 500 |
| 2 | 3 | 1 | `+arr[3](400) −arr[1](200)` | **700** | **700** |

**Result:** `700` ✅ (window `[300, 400]`)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force (nested loop) | `O(n·k)` | `O(1)` | re-sums every window from scratch |
| Sliding Window | `O(n)` | `O(1)` | each element added once, removed once |

---

## Gotchas 🪤

- **`k > n`** — agar array chhota hai `k` se, to valid window banti hi nahi. Pehle guard laga lo (`if (k > n) return -1;` ya problem ke hisaab se).
- **Left index derive karo, store mat karo** — fixed-size window mein `left = right - k` hamesha valid hai, ek alag pointer maintain karna galti ki gunjaish badhata hai.
- **Seed loop ka range** — pehli window `[0, k-1]` hai, `for (i = 0; i < k; i++)` sahi hai, `i <= k` nahi.
- **Negative numbers bhi ho sakte hain** — `maxSum` ko `Integer.MIN_VALUE` se init karo agar brute force likh rahe ho aur array poora negative ho sakta hai; optimal approach mein hum seed window se hi init karte hain isliye woh apne aap safe hai.
- **`right` loop `k` se shuru hota hai**, `0` se nahi — pehli window already seed ho chuki hai, dobara mat process karo.
