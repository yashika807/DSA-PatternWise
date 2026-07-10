# 4Sum — LeetCode 18

> 🔗 **[Interactive Visualizer dekho »]
> 
> (https://yashika807.github.io/DSA-PatternWise/Arrays/4sum.html)**
> 
> _(two Anchors + two Scouts, ledger tape readout, synced Java code)_

**Difficulty:** Medium  ·  **Pattern:** Sort + Two Pointers  ·  **Tags:** Array, Two Pointers, Sorting

---

## Problem

Ek integer array `nums` aur ek `target` diya hai. Saare **distinct quadruplets** `[nums[a], nums[b], nums[c], nums[d]]` return karo jinke indices alag-alag hon (`a != b != c != d`) aur **chaaron ka sum exactly `target`** ho.

Catch: answer mein **duplicate quadruplets nahi** hone chahiye.

```
Input:  nums = [1, 0, -1, 0, -2, 2], target = 0
Output: [[-2, -1, 1, 2], [-2, 0, 0, 2], [-1, 0, 0, 1]]
```

---

## The story (yaad rakhne ke liye) 🧠

4Sum koi naya jaanwar nahi hai — ye **3Sum ka bada bhai** hai. 3Sum mein tune **ek** number fix karke baaki do ko two-pointer se dhoondha tha. 4Sum mein bas **ek aur** number fix karna hai — phir wahi purana two-pointer chalega.

Number line ko phir se **wealth se sorted logon ki line** samjho. Ab humein 4 aise log chahiye jinki net worth milke **exactly `target`** ho jaaye.

| Role | Kaun | Kaam |
|---|---|---|
| **Anchor A** (`i`) | sabse bahar wala loop | "Meri wealth fix." |
| **Anchor B** (`j`) | `i` ke andar wala loop | "Meri bhi wealth fix. Ab do aur dhoondo." |
| **Poor Scout** (`left`) | `j` ke just right | "kam wealth" side |
| **Rich Scout** (`right`) | line ke bilkul end | "zyada wealth" side |

Dono anchors ke baad, scouts wahi tug-of-war khelte hain (target ab `0` nahi, general `target` hai):

- **sum &lt; target** (chaaron milke abhi bhi kam) → Poor Scout aage → `left++`
- **sum &gt; target** (chaaron milke zyada ho gaye) → Rich Scout peeche → `right--`
- **sum == target** → jackpot 🎯, quad record karo, phir dono twins skip karke andar move.

> ⚠️ Verdict hamesha **`sum` ko `target`** se compare karta hai — kisi single element se nahi. 3Sum mein target `0` tha, yahan koi bhi ho sakta hai — logic same.

> 🧮 **`long sum`** kyun? Chaar `int` jab `Integer.MAX_VALUE` ke aas-paas hon toh 32-bit mein overflow ho sakta hai. 3Sum mein teen numbers the, aksar bach jaate the — chautha number ye risk laata hai. Isiliye `(long)` cast.

---

## Approach 1 — Brute force 🟥

Chaaron loops se har possible quadruplet try karo, aur duplicates ko `Set` se hatao.

```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        Arrays.sort(nums);
        Set<List<Integer>> set = new HashSet<>();
        int n = nums.length;
        for (int a = 0; a < n - 3; a++)
            for (int b = a + 1; b < n - 2; b++)
                for (int c = b + 1; c < n - 1; c++)
                    for (int d = c + 1; d < n; d++) {
                        long sum = (long) nums[a] + nums[b] + nums[c] + nums[d];
                        if (sum == target)
                            set.add(Arrays.asList(nums[a], nums[b], nums[c], nums[d]));
                    }
        return new ArrayList<>(set);
    }
}
```

- **Time:** `O(n⁴)` — chaar nested loops.
- **Space:** `O(k)` for the `Set` of results (plus output).
- Sochne mein simple, par bada `n` aate hi TLE.

---

## Approach 2 — Sort + Two Pointers (optimal) 🟩

Do numbers ko loops se **fix** karo (`i`, `j`), baaki do ko **two pointers** se convergence pe pakdo — bilkul 3Sum jaisa, bas ek extra outer layer.

```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        Arrays.sort(nums);
        List<List<Integer>> result = new ArrayList<>();
        int n = nums.length;
        for (int i = 0; i < n - 3; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            for (int j = i + 1; j < n - 2; j++) {
                if (j > i + 1 && nums[j] == nums[j - 1]) continue;
                int left = j + 1;
                int right = n - 1;
                while (left < right) {
                    long sum = (long) nums[i] + nums[j] + nums[left] + nums[right];
                    if (sum == target) {
                        result.add(Arrays.asList(nums[i], nums[j], nums[left], nums[right]));
                        while (left < right && nums[left] == nums[left + 1]) left++;
                        while (left < right && nums[right] == nums[right - 1]) right--;
                        left++;
                        right--;
                    } else if (sum < target) {
                        left++;
                    } else {
                        right--;
                    }
                }
            }
        }
        return result;
    }
}
```

### Duplicate-skipping — teen alag jagah, teen alag kaam

Ye 4Sum ka sabse confusing part hota hai. Teen tarah ke dedup hain:

1. **`i > 0 && nums[i] == nums[i-1] → continue`** — Anchor A ka same value dobara process mat karo.
2. **`j > i+1 && nums[j] == nums[j-1] → continue`** — Anchor B ka same value dobara mat karo. Dhyaan: check `j > i+1` hai, `j > 0` nahi — `nums[j]` `nums[i]` ke barabar ho sakta hai (jaise `[i]=2, [j]=2` valid pair), bas usi `i` ke andar wale pichle `j` jaisa nahi hona chahiye.
3. **`sum == target` block ke andar wale do `while`** — jab match mil gaya, tabhi scouts ke duplicates skip hote hain, taaki **same quadruplet output mein dobara na aaye**. Duplicate output sirf tabhi ban sakta hai jab hum kuch `result` mein `add` kar rahe hon — isiliye ye skip sirf match branch mein hai, `sum < target` / `sum > target` mein nahi.

### Aur `left++; right--;` alag se kyun? (jo tune poocha tha)

`while` loop sirf **agle** duplicates tak jaata hai — wo ruk jaata hai jab `nums[left] != nums[left+1]`, matlab **aakhri copy pe khada** rehta hai, us value se **aage nahi** jaata. Toh:

- **while loops** = "extra copies skip karo" (optimization — warna ek-ek karke slowly hi hatega).
- **plain `left++` / `right--`** = "ab jis value ko use kiya usse actually hato" (correctness — warna wahi pair phir se check hoke double-count ho sakta hai).

Do alag kaam, back-to-back.

---

## Dry run 🎬

`nums = [1, 0, -1, 0, -2, 2]`, `target = 0` → sort → `[-2, -1, 0, 0, 1, 2]` (n = 6)

| i (val) | j (val) | left→ | ←right | sum | verdict | action |
|---|---|---|---|---|---|---|
| 0 (-2) | 1 (-1) | 2 (0) | 5 (2) | -1 | &lt; 0 | left++ |
| 0 (-2) | 1 (-1) | 3 (0) | 5 (2) | -1 | &lt; 0 | left++ |
| 0 (-2) | 1 (-1) | 4 (1) | 5 (2) | **0** | 🎯 | add `[-2,-1,1,2]`, move in |
| 0 (-2) | 2 (0) | 3 (0) | 5 (2) | **0** | 🎯 | add `[-2,0,0,2]`, move in |
| 0 (-2) | 3 (0) | — | — | — | dup j | `nums[3]==nums[2]` → continue |
| 0 (-2) | 4 (1) | 5 (2) | 5 (2) | — | — | left==right, inner loop skip |
| 1 (-1) | 2 (0) | 3 (0) | 5 (2) | **0** | 🎯 | add `[-1,0,0,1]`, move in |
| 1 (-1) | 3 (0) | — | — | — | dup j | continue |
| … | | | | | | baaki koi naya nahi |

**Output:** `[[-2,-1,1,2], [-2,0,0,2], [-1,0,0,1]]` ✅

---

## Complexity 📊

| Approach | Time | Space |
|---|---|---|
| Brute force (`Set`) | `O(n⁴)` | `O(k)` results |
| Sort + Two Pointers | `O(n³)` | `O(1)` extra (output alag) |

Optimal mein sort `O(n log n)`, phir `i` × `j` × two-pointer = `O(n) × O(n) × O(n)` = **`O(n³)`**. 3Sum ke `O(n²)` se ek layer zyada, kyunki ek anchor extra hai.

---

## Gotchas 🪤

- **`long sum`** — chaar bade `int` overflow kar sakte hain. `(long)` cast pehle number pe lagao taaki poori expression long mein evaluate ho.
- **Loop bounds** — `i < n - 3`, `j < n - 2`, `left = j + 1`, `right = n - 1`. Har fixed pointer ke aage utne elements bachne chahiye jitne baaki pointers ko chahiye.
- **`j > i + 1`, not `j > 0`** — Anchor B ka dedup uske apne inner range se hota hai. `j > 0` likha toh valid pairs miss ho jaayenge jahan `nums[j] == nums[i]`.
- **Skip loops sirf `sum == target` mein** — duplicate output tabhi banta hai jab hum `add` karte hain. `sum < target` / `sum > target` branches mein skip loop ki zaroorat nahi.
- **`left++; right--;` while loops ke baad zaroori hai** — while loop aakhri duplicate pe rukta hai, us value se aage nahi jaata; plain increment/decrement hi actually us value se hatata hai.
