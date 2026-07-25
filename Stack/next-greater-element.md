# Next Greater Element II — LeetCode 503

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Stack/next-greater-element.html)**
> _(monotonic decreasing stack of indices, circular sweep, synced Java code)_

**Difficulty:** Medium · **Pattern:** Monotonic Decreasing Stack · **Tags:** Array, Stack, Circular Array

---

## Problem

Ek **circular** integer array `nums` diya hai (last element ke baad wapas first se jud jaata hai). Har element ke liye uska **next greater element** dhoondo — circular order mein aage dekhte hue. Agar koi bhi greater element na mile, `-1`.

```
Input:  nums = [1, 2, 1]
Output: [2, -1, 2]
```

`nums[0]=1` ke liye next greater `2` (index 1). `nums[1]=2` ke liye — koi bhi element usse bada nahi (circular ghoom ke bhi wapas khud pe aa jaata hai) → `-1`. `nums[2]=1` ke liye — circular wraparound karke index 0 ka value dekhna padta hai, phir index 1 ka `2` milta hai.

---

## The story (yaad rakhne ke liye) 🧠

Pehle simpler warm-up soचो: **LeetCode 496 "Next Greater Element I"** — non-circular, seedha ek left-to-right sweep. Us mein trick yeh hai: **right to left** chalo, aur ek stack maintain karo jo hamesha **decreasing order** mein rahe (bottom se top tak values ghatte jaate hain — jaise sidha giri hui *staircase*, isiliye "**monotonic decreasing stack**" kehte hain).

Jab bhi koi naya element `x` aata hai:
- Stack ke top pe jo bhi **chhote-ya-equal** values hain, unko **pop** kar do — woh apna "next greater" nahi ban sakte (`x` khud unse bada hai, to `x` hi unka jawaab hai, agar woh already answer na paa chuke hon — asal mein humesha left-to-right version isko differently karta hai, neeche dekho).
- Jo bache (stack mein), unka top hi tumhara **immediate next greater** hai — kyunki stack **decreasing** hai, top hamesha sabse chhota **abhi tak ka valid candidate** hota hai.
- `x` ko khud push kar do — ab yeh **future elements** ke liye ek candidate ban gaya.

> **Analogy:** Socho log ek building ke roof pe khade hai, height ke hisab se peeche se dekh rahe hain (jaise skyline). Koi bhi banda apne **left** mein pehla **taller building** dhoondh raha hai. Chhote buildings jo uske beech mein aate hain, woh "invisible" ho jaate hain (pop) — sirf taller wale hi **visible candidates** reh jaate hain stack mein. Yehi wajah hai stack hamesha decreasing rehta hai — chhote elements ka koi future use nahi (kisi bhi future element ke liye bhi woh taller candidate se pehle hi block ho jaayenge).

**Circular wraparound kya add karta hai (LeetCode 503):**

Non-circular version mein hum sirf `0` se `n-1` tak ek baar chalte hain. Circular mein array "wrap around" karta hai — matlab `nums[n-1]` ke baad phir se `nums[0]` aa sakta hai answer ke liye. Isko simulate karne ka simplest tareeka: **array ko do baar traverse karo** — `i` ko `0` se `2n-1` tak chalao, aur actual index nikaalne ke liye `i % n` use karo. Isse jaise array **do baar chipka ke** rakh diya ho (`[nums, nums]`), bina extra array banaye.

- Pehli pass (`i < n`) mein hum **push bhi karte hain** (yeh "original" occurrence hai).
- Doosri pass (`i >= n`) mein hum **sirf pop/compare karte hain, push nahi** — kyunki humein sirf original indices ka answer chahiye, unke duplicate "wraparound copy" ka nahi.

---

## Approach 1 — Brute force 🟥

Har index `i` ke liye, agle `n-1` elements circular order mein check karo jab tak koi bada na mil jaaye.

```java
class Solution {
    public int[] nextGreaterElements(int[] nums) {
        int n = nums.length;
        int[] res = new int[n];

        for (int i = 0; i < n; i++) {
            res[i] = -1;
            for (int k = 1; k < n; k++) {                 // check next n-1 elements circularly
                int j = (i + k) % n;
                if (nums[j] > nums[i]) {
                    res[i] = nums[j];
                    break;
                }
            }
        }
        return res;
    }
}
```

**Problem kya hai:** Har element ke liye `O(n)` tak scan — total `O(n²)`. Bade arrays pe slow.

---

## Approach 2 — Optimal (Monotonic Decreasing Stack) ✅

Array ko `2n` baar traverse karo (`i % n` se circular simulate), ek **decreasing stack of indices** maintain karo.

```java
class Solution {
    public int[] nextGreaterElements(int[] nums) {
        int n = nums.length;
        int[] res = new int[n];
        Arrays.fill(res, -1);

        Deque<Integer> stack = new ArrayDeque<>();   // stores INDICES, values decreasing bottom→top

        for (int i = 0; i < 2 * n; i++) {
            int idx = i % n;                          // simulate wraparound

            // jab tak stack ka top "chhota" hai current value se, woh apna answer paa gaya
            while (!stack.isEmpty() && nums[stack.peek()] < nums[idx]) {
                res[stack.pop()] = nums[idx];
            }

            // sirf pehli pass mein push karo — doosri pass sirf resolving ke liye hai
            if (i < n) {
                stack.push(idx);
            }
        }

        return res;
    }
}
```

**Kyun kaam karta hai:** Stack mein jo indices baithe hain unke "next greater" abhi tak nahi mile. Jaise hi koi naya bada element aata hai, woh un sab ka jawaab ban jaata hai jo usse chhote hain (isliye `while` loop se ek saath multiple pop ho sakte hain). Doosri pass array ko "wrap around karke ek aur mauka" deti hai un elements ko jinka answer pehli pass mein nahi mila (jo array ke end ke paas the).

---

## Dry run — `[1, 2, 1]` (n = 3, so i goes 0..5)

| i | idx (i%n) | nums[idx] | Stack before (bottom→top, as indices) | Pops (index → res) | Push? | Stack after |
|:---:|:---:|:---:|:---|:---|:---:|:---|
| 0 | 0 | 1 | `[]` | — | ✅ push 0 | `[0]` |
| 1 | 1 | 2 | `[0]` (nums[0]=1 < 2) | pop 0 → `res[0]=2` | ✅ push 1 | `[1]` |
| 2 | 2 | 1 | `[1]` (nums[1]=2, not < 1) | — | ✅ push 2 | `[1, 2]` |
| 3 | 0 | 1 | `[1, 2]` (nums[2]=1, not < 1) | — | ❌ (i≥n) | `[1, 2]` |
| 4 | 1 | 2 | `[1, 2]` (nums[2]=1 < 2) | pop 2 → `res[2]=2` | ❌ | `[1]` (nums[1]=2, not < 2, stop) |
| 5 | 2 | 1 | `[1]` (nums[1]=2, not < 1) | — | ❌ | `[1]` |

Loop khatam. `res[1]` kabhi set nahi hua → `-1` (already initialized).

**Result:** `[2, -1, 2]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(n²)` | `O(1)` extra | worst case: strictly decreasing array |
| Monotonic Decreasing Stack | `O(n)` | `O(n)` | each index pushed once, popped at most once (over `2n` iterations) |

---

## Gotchas 🪤

- **Stack mein indices rakho, values nahi** — comparison ke liye value chahiye hoti hai (`nums[stack.peek()]`), par `res[]` ko fill karne ke liye index chahiye.
- **Doosri pass mein push mat karo** (`if (i < n)`) — warna duplicate indices stack mein aa jaayenge aur galat/extra resolving ho jaayegi.
- **`< ` use karo, `<=` nahi** — problem mein "greater" chahiye, "greater-or-equal" nahi. Equal values ko pop karna galat answer dega.
- **`2n` baar chalna zaroori hai poore circular coverage ke liye** — sirf `n` baar chalne se end ke elements ko unke circular "next greater" (jo array ke start mein ho sakta hai) nahi milega.
- **All-decreasing array** (`[5,4,3,2,1]`) → koi bhi element apne se bada khud se pehle nahi paata original pass mein; doosri pass mein sirf `nums[0]=5` sabko resolve karta hai jinke liye woh circularly next greater hai.
- **Warm-up se confuse mat ho — LC 496 mein multiple answer arrays ka mapping (Map<val, nextGreater>) hota hai kyunki elements unique guaranteed hote; LC 503 mein hum seedha `res[]` array index se fill karte hain, koi map nahi chahiye.**
