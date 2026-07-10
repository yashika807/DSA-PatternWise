# Pascal's Triangle — LeetCode 118

> 🔗 **[Interactive Visualizer dekho »]

(https://yashika807.github.io/DSA-PatternWise/arrays/pascals-triangle.html)**

**Difficulty:** Easy  ·  **Pattern:** Build-from-previous-row  ·  **Tags:** Array, Dynamic Programming

---

## Problem

`numRows` diya hai. Pascal's Triangle ki pehli `numRows` rows return karo.

Pascal's Triangle mein har number apne **just upar wale do numbers ka sum** hota hai. Rows ke dono **edges hamesha `1`** rehte hain.

```
Input: numRows = 5

Output:
        [1]
       [1,1]
      [1,2,1]
     [1,3,3,1]
    [1,4,6,4,1]
```

---

## Core insight

Do simple rules se poora triangle ban jaata hai:

1. **Edge cell** (`j == 0` ya `j == i`) → value = `1`
2. **Interior cell** → value = `ans[i-1][j-1] + ans[i-1][j]` (upar-left + upar-right)

Matlab har row apne se pehli row par depend karti hai. Isiliye pehle wali row store rakhni padti hai — yahi "build-from-previous" pattern hai.

---

## Approach 1 — Brute (Binomial formula) 🟥

Har cell independently `nCr` formula se nikaalo: value at `(i, j)` = `iCj`.

```java
class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> ans = new ArrayList<>();

        for (int i = 0; i < numRows; i++) {
            List<Integer> row = new ArrayList<>();
            for (int j = 0; j <= i; j++) {
                row.add(nCr(i, j));   // har cell ke liye alag se compute
            }
            ans.add(row);
        }
        return ans;
    }

        long res = 1;
        for (int k = 0; k < r; k++) {
            res = res * (n - k) / (k + 1);
        }
        return (int) res;
    }
}
```

**Problem kya hai:** logic sahi hai, par har cell ke liye alag calculation redundant hai — pehle se calculated values reuse nahi ho rahi. Bade `n` pe factorial-style multiplication overflow ka risk bhi. Interview mein "you're recomputing, can you reuse?" wala follow-up aayega.

---

## Approach 2 — Better / Optimal (Build from previous row) ✅

Har row ko pichhli row se banao. Yehi standard, interviewer-expected solution hai.

```java
class Solution {
    public List<List<Integer>> generate(int numRows) {

        List<List<Integer>> ans = new ArrayList<>();

        for (int i = 0; i < numRows; i++) {

            List<Integer> row = new ArrayList<>();

            for (int j = 0; j <= i; j++) {

                if (j == 0 || j == i) {
                    row.add(1);                        // edge → 1
                } else {
                    row.add(
                        ans.get(i - 1).get(j - 1)      // upar-left
                        + ans.get(i - 1).get(j)        // upar-right
                    );
                }
            }

            ans.add(row);
        }

        return ans;
    }
}
```

Har value bas ek addition se milti hai — koi recomputation nahi. `ans` list khud hi "previous row" ka role play kar rahi hai, toh extra memory bhi nahi lagti.

---

## Approach 3 — Optimal variant (single row, multiplicative) 🟩

Agar pichhli row ko dobara index nahi karna, toh ek hi row ko multiplicative rule se build kar sakte ho:
`C(i, j) = C(i, j-1) * (i - j + 1) / j`

```java
class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> ans = new ArrayList<>();

        for (int i = 0; i < numRows; i++) {
            List<Integer> row = new ArrayList<>();
            long val = 1;                       // pehla element hamesha 1
            for (int j = 0; j <= i; j++) {
                row.add((int) val);
                val = val * (i - j) / (j + 1);  // next element
            }
            ans.add(row);
        }
        return ans;
    }
}
```

Same time complexity, bas har cell ko previous-row lookup ke bina, running product se nikaalta hai. Approach 2 zyada readable hai; ye acha "can you do it another way?" wala answer hai.

---

## Dry run (Approach 2, numRows = 5)

| i | j | Rule | Kaise | Value |
|---|---|------|-------|-------|
| 0 | 0 | edge | j==0 && j==i | `1` |
| 1 | 0 | edge | j==0 | `1` |
| 1 | 1 | edge | j==i | `1` |
| 2 | 0 | edge | j==0 | `1` |
| 2 | 1 | sum  | ans[1][0] + ans[1][1] = 1+1 | `2` |
| 2 | 2 | edge | j==i | `1` |
| 3 | 1 | sum  | ans[2][0] + ans[2][1] = 1+2 | `3` |
| 3 | 2 | sum  | ans[2][1] + ans[2][2] = 2+1 | `3` |
| 4 | 2 | sum  | ans[3][1] + ans[3][2] = 3+3 | `6` |

Final:
```
[1]
[1,1]
[1,2,1]
[1,3,3,1]
[1,4,6,4,1]
```

---

## Complexity

| Approach | Time | Space (extra) |
|----------|------|---------------|
| 1 · Binomial per cell | O(numRows²) but heavier constant | O(1) |
| 2 · Build from previous | **O(numRows²)** | **O(1)** |
| 3 · Multiplicative row | O(numRows²) | O(1) |

> "Extra space O(1)" ka matlab: output list ke alawa koi extra structure nahi. Output khud O(numRows²) jagah leti hai — wo unavoidable hai kyunki wahi toh answer hai.

---

## Gotchas 🪤

- **`j <= i`, not `j < i`** — row `i` mein `i + 1` elements hote hain (row 0 mein 1, row 4 mein 5). Off-by-one yahan aasani se ho jaata hai.
- **Edge check pehle** — `j == 0 || j == i` ko interior sum se pehle handle karo, warna `j-1` ya previous row pe out-of-bounds lag jaayega.
- **`ans.get(i - 1)` safe kab?** — sirf tab jab hum interior branch mein hain (i.e. `i >= 2` guaranteed, kyunki interior cell row 2 se pehle exist hi nahi karti). Row 0 aur 1 fully edges hain.
- **Overflow (Approach 1 & 3)** — factorial/product waale approaches mein `long` use karo. Approach 2 mein sirf addition hai, toh int-range mein safe (LeetCode constraints ke andar).
