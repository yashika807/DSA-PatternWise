Interactive Visual: [shortest-subarray-with-sum-at-least-k.html]

(https://yashika807.github.io/DSA-PatternWise/Prefix-Sum/shortest-subarray-with-sum-at-least-k.html)

# 862. Shortest Subarray with Sum at Least K 🚂

> **Pattern:** Prefix Sum + Monotonic Deque → *sirf best checkpoints ki line lagao*
> **Difficulty:** 🔴 Hard
> **Problem:** [LeetCode 862](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/)

**Difficulty:** Hard  ·  **Pattern:** Prefix Sum + Monotonic Deque  ·  **Tags:** Array, Binary Search, Queue, Sliding Window, Heap, Prefix Sum, Monotonic Queue

---

## Problem

Ek integer array `nums` aur ek integer `k` diya hai. **Sabse chhoti (shortest) non-empty contiguous subarray** ki length return karo jiska sum `>= k` ho. Agar koi aisa subarray exist nahi karta, `-1` return karo.

```
Input:  nums = [2,-1,2], k = 3
Output: 3
Explanation: poora array [2,-1,2] ka sum = 3, jo >= k hai. Chhota koi subarray nahi milta
             jiska sum >= 3 ho, isliye poora array hi answer hai.
```

Array mein **negative numbers bhi** aa sakte hain — isi wajah se yeh problem `subarray-sum-equals-k` se bhi zyada tricky hai: sliding window (do pointer) yahan **fail** karta hai, aur seedha HashMap wala trick bhi kaam nahi aata.

---

## The story (yaad rakhne ke liye) 🧠

Pehle wahi purana kaam karo: prefix sum array bana lo, `prefix[0] = 0` aur `prefix[i] = prefix[i-1] + nums[i-1]`. Ab sawaal ban jaata hai — **sabse chhota `j - i` dhoondo jisme `prefix[j] - prefix[i] >= k`** (aur `j > i`).

Agar sirf positive numbers hote, to do-pointer sliding window kaam kar jaata (jaise-jaise sum badhta hai, window shrink karo). Lekin **negative numbers ki wajah se prefix sum array monotonic nahi rehta** — kabhi upar jaata hai, kabhi neeche. Isliye humein ek **smart waiting line (monotonic deque)** banani padti hai jisme sirf **"promising" checkpoints** rakhte hain, sorted by increasing balance:

1. **Purane checkpoint ko phenk do (back se)** agar uska balance naye checkpoint se **bada ya barabar** hai — kyunki naya checkpoint **baad mein aaya hai (chhoti window dega)** *aur* uska balance **chhota-ya-barabar hai (bada ya barabar gap dega future ke liye)**. Purana checkpoint ab **kabhi kaam ka nahi** — usko line se nikaal do.
2. **Front wale checkpoint ko use karo aur phenk do** jaise hi `prefix[j] - prefix[front] >= k` ho jaaye — window record karo (`j - front`), aur us checkpoint ko deque se nikaal do, kyunki **future mein wahi checkpoint sirf aur lambi window** dega (jitna aage jaoge, utna bada `j`, utni badi window) — ab uska koi fayda nahi.

> 🔑 **Key insight:** deque hamesha **strictly increasing balance order** mein rehta hai, front se back tak. Isse front pe hamesha "sabse purana + sabse chhota balance" wala checkpoint milta hai — jo **shortest window** dene ke liye sabse behtar candidate hai.

> ⚠️ Yeh is folder ki **ek hi problem hai jisme HashMap nahi, Deque chahiye** — kyunki humein "kya yeh balance pehle dikha tha" nahi, balki **"order maintain karke sabse promising checkpoint"** chahiye. HashMap order nahi rakhta, Deque rakhta hai.

---

## Approach 1 — Brute force 🟥

Har starting point `i` se shuru karke, running sum badhate jao aur dekho kab woh `>= k` ban raha hai.

```java
class Solution {
    public int shortestSubarray(int[] nums, int k) {
        int n = nums.length;
        int best = Integer.MAX_VALUE;

        for (int i = 0; i < n; i++) {
            long sum = 0;                       // long — overflow se bachne ke liye
            for (int j = i; j < n; j++) {
                sum += nums[j];
                if (sum >= k) {
                    best = Math.min(best, j - i + 1);
                }
            }
        }

        return best == Integer.MAX_VALUE ? -1 : best;
    }
}
```

**Problem kya hai:** `O(n²)` time — `n = 5·10⁴` pe TLE. Negative numbers ki wajah se early break bhi risky hai (sum dobara badh sakta hai), isliye poora inner loop chalana padta hai.

---

## Approach 2 — Optimal (Prefix Sum + Monotonic Deque) ✅

Pehle prefix sum array banao. Phir ek monotonic deque maintain karo jisme sirf **increasing-balance checkpoints** rehte hain, aur do trimming rules follow karo.

```java
class Solution {
    public int shortestSubarray(int[] nums, int k) {
        int n = nums.length;
        long[] prefix = new long[n + 1];
        for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + nums[i];

        Deque<Integer> deque = new ArrayDeque<>();  // indices into prefix[], strictly increasing balance
        int best = Integer.MAX_VALUE;

        for (int j = 0; j <= n; j++) {
            // FRONT: itna bada gap mil gaya? window record karo, is checkpoint ko phenk do
            while (!deque.isEmpty() && prefix[j] - prefix[deque.peekFirst()] >= k) {
                best = Math.min(best, j - deque.pollFirst());
            }

            // BACK: naya checkpoint purane se behtar-ya-barabar hai? purana phenk do
            while (!deque.isEmpty() && prefix[j] <= prefix[deque.peekLast()]) {
                deque.pollLast();
            }

            deque.offerLast(j);   // aaj ka checkpoint line ke end mein jodo
        }

        return best == Integer.MAX_VALUE ? -1 : best;
    }
}
```

**Kyun kaam karta hai:** har index `j` deque mein **ek hi baar** enter aur **ek hi baar** exit hota hai (ya to front se "used" hoke, ya back se "dominated" hoke) — isliye total work `O(n)` hai, poore `n` checkpoints ke through amortized constant work.

---

## Dry run — `nums = [2, -1, 2], k = 3`

`prefix = [0, 2, 1, 3]` (index 0 se 3 tak)

| j | prefix[j] | Front check (`prefix[j] - prefix[front] >= k`?) | best after front | Back check (trim) | deque after |
|:-:|:---:|:---|:---:|:---|:---|
| 0 | 0 | deque empty | ∞ | deque empty | `[0]` |
| 1 | 2 | `2 - 0 = 2 < 3` no | ∞ | `2 <= prefix[0]=0`? no | `[0, 1]` |
| 2 | 1 | `1 - 0 = 1 < 3` no | ∞ | `1 <= prefix[1]=2`? yes → pop 1. `1 <= prefix[0]=0`? no | `[0, 2]` |
| 3 | 3 | `3 - prefix[0]=0 = 3 >= 3` yes → pop 0, best = min(∞, 3-0) = **3**. Next: `3 - prefix[2]=1 = 2 < 3` no | **3** | `3 <= prefix[2]=1`? no | `[2, 3]` |

**Result:** `best = 3` ✅ (poora array `[2,-1,2]`, length 3 — koi chhota subarray `>= 3` sum nahi banata)

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (nested loop, running sum) | `O(n²)` | `O(1)` | negatives ki wajah se early-exit possible nahi |
| Prefix Sum + Monotonic Deque | `O(n)` | `O(n)` | har index deque mein ek baar enter, ek baar exit — amortized O(1) per index |

---

## Gotchas 🪤

- **`long` use karo prefix sums ke liye** — `n <= 5·10⁴` aur `|nums[i]| <= 10⁵` ho sakta hai, matlab total sum `~5·10⁹` tak ja sakta hai, jo 32-bit `int` mein overflow ho jaayega.
- **Deque mein INDICES store hote hain, values nahi** — kyunki window length (`j - i`) *aur* balance (`prefix[i]`) dono chahiye hote hain, aur index se dono mil jaate hain.
- **Front-trim aur back-trim ka logic alag hai, mat ghulao:** front se pop tab hota hai jab **valid window mil jaaye** (kyunki wahi checkpoint future mein sirf lambi window dega — ab bekaar hai); back se pop tab hota hai jab naya checkpoint **balance mein behtar-ya-barabar** ho (purana checkpoint permanently dominated ho gaya).
- **Yeh iss folder ki akeli problem hai jisme HashMap nahi chalta** — humein "pehle dekha tha kya" nahi, balki **order-aware "sabse promising checkpoint"** chahiye, jo sirf ek monotonic structure (deque) de sakta hai.
- **Sirf positive numbers hote to plain sliding window kaafi tha** — negative numbers hi wajah hain ki prefix sum non-monotonic ho jaata hai, aur do-pointer approach fail ho jaata hai.
- **Loop `j = 0` se `n` tak chalta hai** (prefix array ki size `n+1` hai), na ki `0` se `n-1` — off-by-one yahan common mistake hai.
- **Agar koi valid subarray na mile, `best` `Integer.MAX_VALUE` hi rahega** — isliye return karte waqt `-1` explicitly check karna zaroori hai.
