# 3Sum — LeetCode 15

> 🔗 **[Interactive Visualizer dekho »]
(https://yashika807.github.io/DSA-PatternWise/Arrays/3sum.html)**
> _(step-through Anchor + two Scouts, live sum readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sort + Two Pointers  ·  **Tags:** Array, Two Pointers, Sorting

---

## Problem

Ek integer array `nums` diya hai. Saare **distinct triplets** `[nums[i], nums[j], nums[k]]` return karo jinki `i != j != k` ho aur **teeno ka sum exactly `0`** ho.

Catch: answer mein **duplicate triplets nahi** hone chahiye.

```
Input:  nums = [-1, 0, 1, 2, -1, -4]
Output: [[-1, -1, 2], [-1, 0, 1]]
```

---

## The story (yaad rakhne ke liye) 🧠

Array ko ek **number line pe khadi logon ki line** samjho — sabse zyada **kangaal** (bahut negative) se sabse zyada **ameer** (bahut positive) tak. Humein 3 aise log dhoondhne hain jinki **net worth cancel hoke exactly 0** ho jaaye.

Pehle **sort** karna hi asli trick hai — ek baar line wealth se sorted ho jaaye, toh humein *dikh jaata hai* ki richer/poorer kis direction mein jaana hai.

| Role | Kaun | Kaam |
|---|---|---|
| **Anchor** (`i`) | left→right chalta hai | "Meri wealth fix hai. Ab do aise dhoondo jo mujhe cancel kar dein." |
| **Poor Scout** (`left`) | anchor ke just right | "aur debt / kam wealth" ka side |
| **Rich Scout** (`right`) | line ke bilkul end pe | "aur wealth" ka side |

Dono scouts ek dusre ki taraf chalte hain (tug-of-war):

- **sum &lt; 0** (trio too poor) → Poor Scout right jaata hai → `left++`
- **sum &gt; 0** (trio too rich) → Rich Scout left jaata hai → `right--`
- **sum == 0** → jackpot 🎯, trio record karo, phir dono twins skip karke andar move.

> ⚠️ Verdict hamesha **`sum` ko `0` se compare** karta hai — `nums[i]` se nahi. "Too poor" ka matlab hai teeno ka total abhi bhi `< 0` hai, kisi ek element ki comparison nahi.

---

## Approach 1 — Brute force 🟥

Teeno loops se har possible triplet try karo, aur duplicates ko `Set` se hatao.

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Set<List<Integer>> set = new HashSet<>();
        int n = nums.length;

        for (int i = 0; i < n; i++)
            for (int j = i + 1; j < n; j++)
                for (int k = j + 1; k < n; k++)
                    if (nums[i] + nums[j] + nums[k] == 0) {
                        List<Integer> t = Arrays.asList(nums[i], nums[j], nums[k]);
                        Collections.sort(t);      // dedupe ke liye canonical order
                        set.add(t);
                    }

        return new ArrayList<>(set);
    }
}
```

**Problem kya hai:** `O(n³)` time — `n = 3000` pe ~2.7 billion operations, TLE pakka. Plus `Set` ka extra space. Interview mein turant "can you do better?" aayega.

---

## Approach 2 — Sort + Two Pointers (Optimal) ✅

Array sort karo. Phir har `i` ko **anchor** fix karke, remaining part pe **two-pointer** se `-nums[i]` waali pair dhoondo. Yehi standard, interviewer-expected solution hai.

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(nums);                          // sort the line

        for (int i = 0; i < nums.length - 2; i++) {

            // skip duplicate anchor
            if (i > 0 && nums[i] == nums[i - 1]) continue;

            // anchor already positive → sorted rest is bigger → impossible
            if (nums[i] > 0) break;

            int left = i + 1, right = nums.length - 1;

            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];

                if (sum == 0) {
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));

                    // skip twins on both sides
                    while (left < right && nums[left] == nums[left + 1]) left++;
                    while (left < right && nums[right] == nums[right - 1]) right--;

                    left++;
                    right--;
                } else if (sum < 0) {
                    left++;     // too poor → need bigger sum
                } else {
                    right--;    // too rich → need smaller sum
                }
            }
        }

        return result;
    }
}
```

### Do guardrails jo kaam bachaate / galti rokte hain

- **`if (nums[i] > 0) break;`** — anchor khud positive hai aur sorted line mein aage sab isse bade hain. Teeno positive kabhi `0` nahi honge. Show over.
- **`if (i > 0 && nums[i] == nums[i-1]) continue;`** — same wealth waale do anchors same scouts recruit karenge aur same trio banayenge. Duplicate show mat chalao.

---

## Dry run — `[-1, 0, 1, 2, -1, -4]`

Sorted: **`[-4, -1, -1, 0, 1, 2]`** &nbsp;(indices `0..5`)

| i (Anchor) | left, right | nums[left], nums[right] | sum | Verdict |
|:---:|:---:|:---:|:---:|:---|
| **0** (`-4`) | 1, 5 | -1, 2 | **-3** | too poor → `left++` |
| | 2, 5 | -1, 2 | -3 | too poor → `left++` |
| | 3, 5 | 0, 2 | -2 | too poor → `left++` |
| | 4, 5 | 1, 2 | -1 | too poor → `left++` |
| | 5, 5 | — | — | `left==right`, **stop** — no trio for -4 |
| **1** (`-1`) | 2, 5 | -1, 2 | **0** | ✅ record `[-1,-1,2]` → both move in |
| | 3, 4 | 0, 1 | **0** | ✅ record `[-1,0,1]` → both move in |
| | 4, 3 | — | — | `left > right`, **stop** |
| **2** (`-1`) | — | — | — | `nums[2]==nums[1]` → **duplicate anchor, skip** |
| **3** (`0`) | 4, 5 | 1, 2 | 3 | too rich → `right--` |
| | 4, 4 | — | — | `left==right`, **stop** |

**Result:** `[[-1,-1,2], [-1,0,1]]` ✅

> Note: `-3` verdict `sum` vs `0` ka hai. `-3 > -4` sahi hai par woh comparison kahin use hi nahi hoti — sirf teeno ka total `0` se compare hota hai.

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute (3 loops + Set) | `O(n³)` | `O(m)` | `m` = result size; TLE on large `n` |
| Sort + Two Pointers | `O(n²)` | `O(1)` extra* | sort `O(n log n)`, dominated by `n²` scan |

\* Output list aur sort ke recursion stack ko chhod kar. Output khud unavoidable jagah leti hai — wahi toh answer hai.

---

## Gotchas 🪤

- **Sort pehle, warna two-pointer kaam nahi karega** — poora logic "left = richer, right = poorer" pe tika hai, jo sirf sorted array mein sach hai.
- **Verdict `sum` vs `0`, na ki element vs element** — `sum < 0` = "trio too poor", `nums[i]` ki kisi se tulna nahi.
- **Teeno jagah duplicate skip** — anchor (`i`), Poor Scout (`left`), Rich Scout (`right`). Teeno chhoote toh duplicate triplets aayenge.
- **Twin-skip `sum == 0` ke *baad* hi** — pehle trio record karo, *phir* `nums[left]==nums[left+1]` waale twins skip karo. Ulta karoge toh valid trio miss ho jaayega.
- **`left < right` guard har inner `while` mein** — twin-skip loops mein bhi, warna pointers cross karke out-of-bounds / galat pair ban jaayega.
- **`i < nums.length - 2`** — anchor ke aage kam se kam do elements bache hone chahiye (`left` aur `right` ke liye).
- **`break` vs `continue`** — positive anchor pe `break` (aage sab bekaar), duplicate anchor pe `continue` (sirf yeh skip, aage try karo). Inko mat mila dena.
