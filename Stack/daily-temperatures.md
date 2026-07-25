# Daily Temperatures — LeetCode 739

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Stack/daily-temperatures.html)**
> _(monotonic decreasing stack of day-indices, live wait-count fill, synced Java code)_

**Difficulty:** Medium · **Pattern:** Monotonic Decreasing Stack · **Tags:** Array, Stack

---

## Problem

Ek array `temperatures` diya hai jahan `temperatures[i]` = din `i` ka temperature. Har din ke liye batao ki **kitne din baad** temperature usse **zyada** hoga. Agar kabhi na ho, `0`.

```
Input:  temperatures = [73, 74, 75, 71, 69, 72, 76, 73]
Output: [1, 1, 4, 2, 1, 1, 0, 0]
```

Din `0` (73°) ke agle hi din (74°) zyada hai → `1`. Din `2` (75°) ke liye din `6` (76°) tak wait karna padta hai → `6 - 2 = 4` din.

---

## The story (yaad rakhne ke liye) 🧠

Yeh bilkul `next-greater-element` jaisa hi pattern hai — bas is baar humein "**kaun sa element**" nahi, balki "**kitne din wait karna**" chahiye. Same **monotonic decreasing stack of indices**, bas answer ka formula badal jaata hai: `res[i] = today - i` (index ka difference, value ka nahi).

**Analogy:** Socho tum ek **weather diary** maintain kar rahe ho — har din likhte jaate ho. Jab bhi ek naya **garam din** aata hai, tumhare diary ke saare **purane thande din** apna jawaab paa jaate hain: *"itne din baad garmi aayi"*. Jo din abhi bhi **stack mein baithe** hain (kyunki koi ab tak unse garam din nahi aaya), unka jawaab `0` reh jaata hai (default).

- Naya din `i` aaya → jab tak stack ke top wala din **thanda** hai `temperatures[i]` se, use **pop** karo aur uska wait-count = `i - poppedIndex` set karo.
- `i` khud ko push karo — yeh ab **future**, garam dino ke liye ek candidate hai.

> 🔑 **Key insight:** Stack **decreasing temperature** order maintain karta hai (bottom = sabse garam abhi tak jiska jawaab nahi mila, top = sabse thanda abhi tak). Jaise hi ek garam din aata hai, woh multiple purane thande dino ko ek saath resolve kar sakta hai — isiliye `while` loop, na ki single `if`.

---

## Approach 1 — Brute force 🟥

Har din ke liye aage ke sab din check karo jab tak temperature zyada na mile.

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] res = new int[n];

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (temperatures[j] > temperatures[i]) {
                    res[i] = j - i;
                    break;                      // res[i] already 0 by default agar kabhi na mile
                }
            }
        }
        return res;
    }
}
```

**Problem kya hai:** `O(n²)` — worst case (strictly decreasing temperatures, jaise `[80,70,60,50]`) mein har din ko poora baaki array scan karna padta hai.

---

## Approach 2 — Optimal (Monotonic Decreasing Stack) ✅

Ek hi left-to-right pass mein **indices** ka stack maintain karo, decreasing temperature order mein.

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] res = new int[n];                    // default 0 — "kabhi warm nahi hoga"
        Deque<Integer> stack = new ArrayDeque<>();  // stores day-INDICES, decreasing temp bottom→top

        for (int i = 0; i < n; i++) {
            // jab tak stack ka top "thanda" hai aaj se, uska jawaab mil gaya
            while (!stack.isEmpty() && temperatures[stack.peek()] < temperatures[i]) {
                int coldDay = stack.pop();
                res[coldDay] = i - coldDay;         // kitne din wait karna pada
            }
            stack.push(i);                          // aaj ka din future ke liye candidate
        }

        return res;   // stack mein bache indices ka res[] already 0 hai
    }
}
```

---

## Dry run — `[73, 74, 75, 71, 69, 72, 76, 73]`

| i | temp | Stack before (indices, bottom→top) | Pops → res[] update | Push | Stack after |
|:---:|:---:|:---|:---|:---:|:---|
| 0 | 73 | `[]` | — | 0 | `[0]` |
| 1 | 74 | `[0]` (73<74) | pop 0 → `res[0]=1-0=1` | 1 | `[1]` |
| 2 | 75 | `[1]` (74<75) | pop 1 → `res[1]=2-1=1` | 2 | `[2]` |
| 3 | 71 | `[2]` (75 not < 71) | — | 3 | `[2,3]` |
| 4 | 69 | `[2,3]` (71 not < 69) | — | 4 | `[2,3,4]` |
| 5 | 72 | `[2,3,4]` (69<72, then 71<72) | pop 4 → `res[4]=5-4=1`; pop 3 → `res[3]=5-3=2` | 5 | `[2,5]` (75 not < 72, stop) |
| 6 | 76 | `[2,5]` (75<76, then 72<76) | pop 5 → `res[5]=6-5=1`; pop 2 → `res[2]=6-2=4` | 6 | `[6]` |
| 7 | 73 | `[6]` (76 not < 73) | — | 7 | `[6,7]` |

Loop khatam. `res[6]` aur `res[7]` kabhi set nahi hue → `0` (already default).

**Result:** `[1, 1, 4, 2, 1, 1, 0, 0]` ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Brute force | `O(n²)` | `O(1)` extra | worst case: strictly decreasing temps |
| Monotonic Decreasing Stack | `O(n)` | `O(n)` | each index pushed once, popped at most once |

---

## Gotchas 🪤

- **`res[]` ka default `0` hi answer hai jinke liye koi warmer din nahi aata** — koi extra "not found" marker set karne ki zaroorat nahi (jaise NGE mein `-1` karna padta hai).
- **Formula `i - coldDay` hai, value ka difference nahi** — yeh problem "kitne din" poochta hai, "kitna temperature difference" nahi.
- **`<` strictly less use karo** — equal temperature "warmer" nahi mana jaata, isliye equal temps stack mein rehte hain.
- **Stack mein indices hi rakho, temperature value nahi** — comparison ke liye `temperatures[stack.peek()]` chahiye, aur answer ke liye index (`i - coldDay`) chahiye.
- **`while`, `if` nahi** — ek naya garam din ek saath **multiple** purane thande dino ko resolve kar sakta hai (jaise upar `i=5` pe do pops hue).
