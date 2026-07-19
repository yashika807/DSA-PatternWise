Interactive Visual: [count-range-sum.html]

(https://yashika807.github.io/DSA-PatternWise/Prefix-Sum/count-range-sum.html)

# 327. Count of Range Sum 📚

> **Pattern:** Prefix Sum + Fenwick Tree (BIT, coordinate compression) → *sorted shelf pe range query*
> **Difficulty:** 🔴 Hard
> **Problem:** [LeetCode 327](https://leetcode.com/problems/count-of-range-sum/)

**Difficulty:** Hard  ·  **Pattern:** Prefix Sum + Binary Indexed Tree  ·  **Tags:** Array, Binary Search, Divide and Conquer, Binary Indexed Tree, Segment Tree, Merge Sort

---

## Problem

Ek integer array `nums` aur do integers `lower`, `upper` diye hain. Ek **range sum** `S(i, j)` matlab `nums[i] + nums[i+1] + ... + nums[j]` (`i <= j`). Total **kitne range sums** hain jo `lower <= S(i,j) <= upper` satisfy karte hain, yeh count karke return karo.

```
Input:  nums = [-2,5,-1], lower = -2, upper = 2
Output: 3
Explanation: teen range sums range mein aate hain — S(0,0) = -2, S(2,2) = -1, S(0,2) = 2.
             (baaki range sums: S(0,1)=3, S(1,1)=5, S(1,2)=4 — sab range se bahar hain.)
```

---

## The story (yaad rakhne ke liye) 🧠

Yeh poori family (`subarray-sum-equals-k`, `subarray-sums-divisible-by-k`, `contiguous-array`) ek hi jagah se shuru hoti hai:

```
prefixSum[j] - prefixSum[i] = subarray sum (i+1 ... j)
```

Ab tak humne poocha: *"exact match"* (`== k`), *"same remainder"* (`% k`), *"same balance, first time"* (longest span). Is problem mein sawaal thoda **bada** hai:

```
lower <= prefixSum[j] - prefixSum[i] <= upper
    ⟺   prefixSum[j] - upper <= prefixSum[i] <= prefixSum[j] - lower
```

Matlab har `j` pe humein poochna hai: *"kitne purane `prefixSum[i]` is **poore range** `[prefixSum[j]-upper, prefixSum[j]-lower]` ke andar aate hain?"* — yeh ab exact match nahi, ek **range query** hai. HashMap sirf exact key dhoondh sakta hai, range nahi.

Socho tumhare paas ek **library shelf** hai jisme har purana `prefixSum` value ek **sorted position** pe rakha hai. Jab bhi naya balance aata hai, tumhe poochna hai: *"shelf ke kitne books (values) is range ke andar hain?"* — agar shelf **sorted** rahe, to yeh count fast (O(log n)) mil sakta hai ek **Fenwick Tree (Binary Indexed Tree)** se:

```
count(range [lo, hi]) = query(hi) - query(lo - 1)
```

Lekin prefix sums bahut bade/sparse ho sakte hain (`10⁹` tak), isliye seedha array index nahi bana sakte — pehle **coordinate compression** karo: saare prefix sums ko sort karo, har value ki ek **rank** (position) nikaalo, aur BIT us rank pe kaam kare.

> 🔑 **Key insight:** yeh poori Prefix-Sum family ka **capstone** hai — same core trick (`prefixSum[j] - prefixSum[i]`), bas "lookup" har baar upgrade hota gaya: **HashMap (exact match)** → **HashMap (remainder match)** → **HashMap (first-occurrence)** → **Monotonic Deque (shortest, with negatives)** → **Fenwick Tree (arbitrary range match)**.

---

## Approach 1 — Brute force 🟥

Prefix sums bana lo (O(1) range-sum lookup ke liye), phir har `(i, j)` pair check karo.

```java
class Solution {
    public int countRangeSum(int[] nums, int lower, int upper) {
        int n = nums.length;
        long[] prefix = new long[n + 1];
        for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + nums[i];

        int count = 0;
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j <= n; j++) {
                long rangeSum = prefix[j] - prefix[i];
                if (rangeSum >= lower && rangeSum <= upper) count++;
            }
        }
        return count;
    }
}
```

**Problem kya hai:** `O(n²)` time — `n = 10⁵` pe TLE. Prefix sum se range-sum `O(1)` mein mil jaata hai, par pairs `O(n²)` hi rehte hain.

---

## Approach 2 — Optimal (Prefix Sum + Fenwick Tree, coordinate compression) ✅

Har prefix sum ko process karte waqt, pehle poocho **"kitne purane prefix sums range ke andar hain"** (BIT range query), phir aaj ka value BIT mein **insert** karo (uski compressed rank pe).

```java
import java.util.Arrays;

class Solution {
    public int countRangeSum(int[] nums, int lower, int upper) {
        int n = nums.length;
        long[] prefix = new long[n + 1];
        for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + nums[i];

        long[] sorted = prefix.clone();
        Arrays.sort(sorted);                       // coordinate compression ke liye

        int[] bit = new int[n + 2];                // Fenwick tree, 1-indexed
        int count = 0;

        insert(bit, sorted, 0L, n);                 // seed: prefixSum 0, before any element

        long prefixSum = 0;
        for (int num : nums) {
            prefixSum += num;

            int lo = lowerBound(sorted, prefixSum - upper);  // first index with value >= prefixSum-upper
            int hi = upperBound(sorted, prefixSum - lower);  // first index with value >  prefixSum-lower

            // kitne purane prefix sums [prefixSum-upper, prefixSum-lower] range mein hain?
            count += query(bit, hi) - query(bit, lo);

            // aaj ka prefix sum BIT mein uski rank pe daal do
            insert(bit, sorted, prefixSum, n);
        }

        return count;
    }

    private void insert(int[] bit, long[] sorted, long value, int n) {
        int rank = lowerBound(sorted, value) + 1;    // 1-indexed
        update(bit, rank, n + 1);
    }

    private void update(int[] bit, int i, int size) {
        for (; i <= size; i += i & (-i)) bit[i]++;
    }

    private int query(int[] bit, int i) {
        int sum = 0;
        for (; i > 0; i -= i & (-i)) sum += bit[i];
        return sum;
    }

    private int lowerBound(long[] arr, long target) {
        int lo = 0, hi = arr.length;
        while (lo < hi) {
            int mid = (lo + hi) >>> 1;
            if (arr[mid] < target) lo = mid + 1; else hi = mid;
        }
        return lo;
    }

    private int upperBound(long[] arr, long target) {
        int lo = 0, hi = arr.length;
        while (lo < hi) {
            int mid = (lo + hi) >>> 1;
            if (arr[mid] <= target) lo = mid + 1; else hi = mid;
        }
        return lo;
    }
}
```

**Kyun kaam karta hai:** `lowerBound`/`upperBound` range ke dono ends ko sorted array mein rank de dete hain (chahe woh exact value present ho ya na ho), aur BIT do prefix-count queries (`query(hi) - query(lo)`) se batata hai kitne inserted values un ranks ke beech hain — sab `O(log n)` mein.

---

## Dry run — `nums = [-2, 5, -1], lower = -2, upper = 2`

`prefix = [0, -2, 3, 2]`. (Neeche "seen" ek simplified view hai — asal code isko BIT + coordinate compression se `O(log n)` mein karta hai, par result same hota hai.)

Init: `seen = {}`, `count = 0`

| i | prefix[i] | range = `[p-upper, p-lower]` | matches in seen | count | seen after |
|:-:|:---:|:---:|:---|:-:|:---|
| 0 | 0 | `[0-2, 0+2] = [-2, 2]` | none (empty) | 0 | `{0:1}` |
| 1 | -2 | `[-2-2, -2+2] = [-4, 0]` | `0` is in range | **1** | `{0:1, -2:1}` |
| 2 | 3 | `[3-2, 3+2] = [1, 5]` | none (`0`,`-2` both outside) | 1 | `{0:1, -2:1, 3:1}` |
| 3 | 2 | `[2-2, 2+2] = [0, 4]` | `0` aur `3` dono in range | **3** | `{0:1, -2:1, 3:1, 2:1}` |

**Result:** `count = 3` ✅ — matches expected output exactly.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (prefix diff, all pairs) | `O(n²)` | `O(n)` | prefix array se range-sum O(1), par pairs O(n²) |
| Prefix Sum + Fenwick Tree | `O(n log n)` | `O(n)` | har prefix sum ke liye 2 binary search + 1 BIT query + 1 BIT update, sab `O(log n)` |

*(Note: merge-sort-based counting bhi ek popular `O(n log n)` alternative hai, jo inversion-counting jaisa divide-and-conquer use karta hai — BIT approach isliye chuna gaya kyunki yeh isi folder ke "prefix sum + lookup structure" pattern ko seedha extend karta hai.)*

---

## Gotchas 🪤

- **`long` zaroori hai prefix sums ke liye** — `n <= 10⁵` aur `|nums[i]| <= 2³¹-1` ho sakta hai, sum bahut bada ho sakta hai, `int` overflow ho jaayega.
- **Coordinate compression zaroori hai** — prefix sums itne bade/sparse ho sakte hain ki seedha array index banana possible nahi; pehle unique-ish sorted values ki **rank** nikaalni padti hai.
- **`lowerBound` aur `upperBound` alag hote hain, ek jaisa mat samjho** — `lowerBound(x)` first index deta hai jahan value `>= x` ho; `upperBound(x)` first index deta hai jahan value `> x` ho. Range ke **inclusive** dono ends ke liye dono chahiye (`lower <= sum <= upper` ko compressed ranks mein sahi se map karne ke liye).
- **BIT 1-indexed hota hai** — rank `0` pe update/query karna galat result dega; hamesha `+1` karke 1-indexed banao.
- **Query pehle, insert baad mein** — jaise `subarray-sum-equals-k` mein, aaj ka value record karne se *pehle* lookup karo, warna khud ko hi count kar loge.
- **Yeh folder ka sabse "upgraded" lookup hai** — `subarray-sum-equals-k` mein HashMap ne *exact match* diya, `subarray-sums-divisible-by-k` mein HashMap ne *remainder match* diya, `contiguous-array` mein HashMap ne *first-occurrence* diya, `shortest-subarray-with-sum-at-least-k` mein Deque ne *order-aware shortest* diya — yahan **range match** chahiye, jo sirf ek sorted/indexed structure (Fenwick Tree, Segment Tree, ya Balanced BST) de sakta hai.
