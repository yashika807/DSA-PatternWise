# Maximum Subarray Sum with One Deletion

> **LeetCode 1186** · Difficulty: **Medium** · Topics: `Array`, `Dynamic Programming`

Given an array of integers, return the maximum sum of a **non-empty** subarray (contiguous elements), where you may **optionally delete one element** from the chosen subarray. After the deletion, at least one element must remain.

---

## Examples

| Input | Output | Why |
|-------|--------|-----|
| `[1, -2, 0, 3]` | `4` | Take `[1, -2, 0, 3]`, drop `-2` → `1 + 0 + 3 = 4` |
| `[1, -2, -2, 3]` | `3` | One deletion can't remove both `-2`s, so just take `[3]` |
| `[-1, -1, -1, -1]` | `-1` | Deleting from `[-1]` would leave it empty (not allowed), so the best is a single `-1` |

**Constraints**

- `1 <= arr.length <= 10^5`
- `-10^4 <= arr[i] <= 10^4`

---

## The wrong instinct (and why it fails)

A tempting first idea: run normal Kadane's, track the **minimum element**, and check whether deleting that minimum improves the sum.

This breaks. The value of deleting an element depends entirely on **what subarray surrounds it**, not on how negative the element itself is.

**Counterexample:** `[1, 2, -3, 4]`

- The global structure tempts you toward deleting the most negative element.
- But the right move is to delete `-3` — it sits in the middle of a run of positives, so removing it costs nothing to reach: `1 + 2 + 4 = 7`.

Now compare with `[1, -10, 1, 2, -3, 4]`. The global minimum is `-10`, but deleting it forces you to give up contiguity with the `4` on the far side. Deleting the smaller, better-placed `-3` wins instead.

> **Takeaway:** the deletion decision can't be made once, globally. It has to be re-evaluated at **every ending index**, because the best surrounding context changes as the window slides.

---

## The idea: two Kadane's running in parallel

Instead of asking *which* element to delete globally, track two quantities for every subarray **ending at index `i`**:

| State | Meaning |
|-------|---------|
| `noDel` | best sum ending at `i`, **no deletion used** |
| `oneDel` | best sum ending at `i`, **exactly one deletion used** |

`noDel` is just plain Kadane's. The interesting one is `oneDel`.

### The core transition

There are exactly **two ways** to arrive at the "one deletion used, ending at `i`" state:

- **Possibility A — delete `arr[i]` itself.** Then the sum is whatever the *no-deletion* best was at `i-1`, because `arr[i]` isn't counted.
- **Possibility B — keep `arr[i]`; the deletion already happened earlier.** Then you extend the previous `oneDel` by `arr[i]`.

```
noDel[i]  = max(arr[i], noDel[i-1] + arr[i])          // plain Kadane
oneDel[i] = max(noDel[i-1], oneDel[i-1] + arr[i])     //   A          ,   B
answer    = max over all i of max(noDel[i], oneDel[i])
```

Because we only ever look back one step, we can collapse the arrays into a few variables — **O(1) space**.

---

## Solution (Java)

```java
class Solution {
    public int maximumSum(int[] arr) {
        int n = arr.length;
        int noDel = arr[0];
        int oneDel = 0;          // placeholder: deletion is invalid at the first element
        int res = arr[0];

        for (int i = 1; i < n; i++) {
            int prevNoDel = noDel;                          // save BEFORE noDel is overwritten
            noDel  = Math.max(arr[i], noDel + arr[i]);      // plain Kadane
            oneDel = Math.max(prevNoDel, oneDel + arr[i]);  // A: delete arr[i] | B: keep, deleted earlier
            res = Math.max(res, Math.max(noDel, oneDel));
        }
        return res;
    }
}
```

### Two details that trip people up

1. **Order matters.** `oneDel`'s update needs the **old** `noDel` (the value at `i-1`). That's why we snapshot it into `prevNoDel` *before* overwriting `noDel`. Update `noDel` first without saving, and Possibility A silently uses the wrong value.

2. **Seed `oneDel = 0`, not `Integer.MIN_VALUE`.** `MIN_VALUE` looks "safe," but `oneDel + arr[i]` when `arr[i]` is negative causes **integer overflow** (`MIN_VALUE + negative` wraps to a huge positive), producing a wrong answer. Seeding with `0` never overflows and behaves as a harmless empty prefix — the deletion stays "unused" until it's actually spent.

---

## Solution (Python)

```python
class Solution:
    def maximumSum(self, arr: List[int]) -> int:
        no_del = res = arr[0]
        one_del = 0
        for x in arr[1:]:
            prev_no_del = no_del
            no_del = max(x, no_del + x)
            one_del = max(prev_no_del, one_del + x)
            res = max(res, no_del, one_del)
        return res
```

Python has arbitrary-precision integers, so the overflow caveat doesn't apply — but seeding `one_del = 0` still keeps the logic clean.

---

## Dry run: `[1, -2, 1, 2, -3, 4]`

Best answer is `1 + 2 + 4 = 7`, deleting `-3`.

| i | arr[i] | prevNoDel | noDel | oneDel | res | What `oneDel` did |
|---|--------|-----------|-------|--------|-----|-------------------|
| 0 | 1  | –  | 1  | 0  | 1 | init (placeholder) |
| 1 | -2 | 1  | -1 | 1  | 1 | A: delete `-2` → keeps the `1` |
| 2 | 1  | -1 | 1  | 2  | 2 | B: extend, `1 + 1` |
| 3 | 2  | 1  | 3  | 4  | 4 | B: extend, `2 + 2` |
| 4 | -3 | 3  | 0  | 3  | 4 | A: delete `-3` → falls back to `noDel[i-1] = 3` |
| 5 | 4  | 0  | 4  | 7  | **7** | B: extend, `3 + 4` |

Notice how `oneDel` "chooses" to spend its deletion on `-3` at `i = 4` (Possibility A), then rides the `4` at `i = 5` (Possibility B) to reach `7`. The deletion was decided **locally**, not by any global minimum scan.

---

## Complexity

| | |
|---|---|
| **Time** | `O(n)` — single pass |
| **Space** | `O(1)` — a handful of variables |

---

## Edge cases to verify

- **All negatives** (`[-1,-1,-1,-1]`): answer is the largest single element (`-1`). Deletion never helps because it would either empty a single-element subarray or leave you with more negatives.
- **Single element** (`[5]`): the loop never runs; answer is `arr[0]`.
- **Deletion never used**: if the array is all positive, `noDel` always wins and `oneDel` never gets to spend its deletion — still correct.

---

## Related problems

- **53. Maximum Subarray** — the same problem without the deletion (just the `noDel` track).
- **152. Maximum Product Subarray** — a two-state Kadane variant tracking min and max instead.
