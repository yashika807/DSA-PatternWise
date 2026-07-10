# Majority Element II — Boyer–Moore Voting

Find **every** element in an array that appears **more than ⌊n/3⌋ times**, in **O(n) time** and **O(1) extra space**.

Ships with an interactive, election-themed visualizer 


---

## The problem

> Given an integer array `nums` of size `n`, return all elements that appear more than `⌊n/3⌋` times.

The key insight that makes constant space possible: **at most two** elements can appear more than `n/3` times. If three distinct elements each appeared more than `n/3` times, their counts would sum to more than `n` — impossible. So we only ever need to track **two** candidates.

This is a two-candidate generalization of the classic Boyer–Moore Majority Vote algorithm (which handles the `> n/2` case with a single candidate).

---

## The algorithm

Two phases:

1. **Voting** — a single pass that narrows the field down to (at most) two *candidates* using cancellation. Candidates are not yet guaranteed to be valid.
2. **Verification** — a second pass that recounts the real frequency of each candidate and keeps only those that actually clear the `⌊n/3⌋` threshold.

The second phase is **not optional**: Phase 1's counters are "surviving votes," not true frequencies, so a false candidate can slip through and must be filtered out.

### Solution

```java
class Solution {
    public List<Integer> majorityElement(int[] nums) {
        int el1 = Integer.MIN_VALUE;
        int el2 = Integer.MIN_VALUE;
        int cnt1 = 0, cnt2 = 0;

        // Phase 1 — voting: find two candidates
        for (int i = 0; i < nums.length; i++) {
            if (cnt1 == 0 && nums[i] != el2) {
                cnt1 = 1;
                el1 = nums[i];
            } else if (cnt2 == 0 && nums[i] != el1) {
                cnt2 = 1;
                el2 = nums[i];
            } else if (el1 == nums[i]) {
                cnt1++;
            } else if (el2 == nums[i]) {
                cnt2++;
            } else {
                cnt1--;
                cnt2--;
            }
        }

        // Phase 2 — verification: recount honestly
        cnt1 = 0; cnt2 = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == el1) cnt1++;
            if (nums[i] == el2) cnt2++;
        }

        int mini = nums.length / 3 + 1;
        List<Integer> result = new ArrayList<>();
        if (cnt1 >= mini) result.add(el1);
        if (cnt2 >= mini && el2 != el1) result.add(el2);
        return result;
    }
}
```

---

## How it works

### Phase 1 — voting by cancellation

Two candidate slots (`el1`, `el2`) each hold a value and a vote count. For each element, exactly one branch fires:

| Situation | What happens |
|---|---|
| Slot 1 empty **and** value ≠ el2 | Value **claims slot 1**, `cnt1 = 1` |
| Slot 2 empty **and** value ≠ el1 | Value **claims slot 2**, `cnt2 = 1` |
| Value == el1 | **Vote** for candidate 1, `cnt1++` |
| Value == el2 | **Vote** for candidate 2, `cnt2++` |
| Value matches neither | **Cancel** — `cnt1--` **and** `cnt2--` |

The two `!= el2` / `!= el1` guards matter: they stop the same value from occupying both slots.

**Why cancellation works.** A value matching neither candidate "spends itself" to knock one vote off *each* candidate — three distinct values effectively annihilating together. An element that truly appears more than `n/3` times simply can't be cancelled out completely: there isn't enough opposing volume in the rest of the array to drain it, so it survives (or reappears) as a candidate.

### Phase 2 — verification

Reset both counts and do an exact recount over the whole array. Only now do we know each candidate's real frequency.

```
mini = n / 3 + 1   // smallest integer strictly greater than n/3
```

A candidate is a winner iff its true count is `>= mini`. The `el2 != el1` guard is a defensive check against emitting the same value twice (relevant for tiny/degenerate inputs where both slots may still hold the sentinel).

---

## Worked example

`nums = [1, 1, 1, 3, 3, 2, 2, 2]`, so `n = 8`, threshold `⌊8/3⌋ = 2` → need **> 2**, i.e. at least 3 occurrences.

**Phase 1 — voting**

| i | num | branch | el1 | cnt1 | el2 | cnt2 |
|---|-----|--------|-----|------|-----|------|
| 0 | 1 | claim slot 1 | 1 | 1 | — | 0 |
| 1 | 1 | vote el1 | 1 | 2 | — | 0 |
| 2 | 1 | vote el1 | 1 | 3 | — | 0 |
| 3 | 3 | claim slot 2 | 1 | 3 | 3 | 1 |
| 4 | 3 | vote el2 | 1 | 3 | 3 | 2 |
| 5 | 2 | cancel | 1 | 2 | 3 | 1 |
| 6 | 2 | cancel | 1 | 1 | 3 | 0 |
| 7 | 2 | claim slot 2 | 1 | 1 | 2 | 1 |

Candidates: `el1 = 1`, `el2 = 2`. Note `3` was briefly a candidate but got cancelled out — exactly the false positive Phase 2 must catch.

**Phase 2 — recount**

Real counts: `1 → 3`, `2 → 3`. Both `>= mini = 3`.

**Result:** `[1, 2]`. The decoy `3` (true count 2) is correctly rejected.

---

## Complexity

| | |
|---|---|
| **Time** | `O(n)` — two linear passes |
| **Space** | `O(1)` — a fixed set of scalar variables |

Compared with a hash-map frequency count (also `O(n)` time), this approach avoids the `O(n)` auxiliary space.

---

## The visualizer

`majority-vote-visualizer.html` is a single self-contained file — no build, no dependencies.

**Run it:** open the file in any modern browser (double-click, or drag it into a browser tab).

**What it shows**
- The array with a moving pointer on the current element.
- Two candidate cards with live values and vote bars.
- A per-step narration that names the exact branch taken (`claim`, `vote`, `cancel`, `count`).
- A phase indicator, and the final winners with the threshold used.

**Controls**
- `Step ▶` / `◀ Back` — move one iteration at a time.
- `▶ Play` — auto-advance, with a speed slider.
- `↻ Reset` — jump back to the start.
- Presets for the interesting cases (two winners, one winner, no winner, decoys, single element).
- Type any comma- or space-separated integers and hit **Load**.

**Keyboard:** `←` / `→` to step, `space` to play/pause, `R` to reset.

Each visual step corresponds to exactly one loop iteration in the Java code, so you can read the code and the animation side by side.

---

## Edge cases worth trying in the visualizer

| Input | Winners | Why it's interesting |
|---|---|---|
| `1,1,1,3,3,2,2,2` | `1, 2` | Two winners; `3` is a cancelled decoy |
| `3,2,3` | `3` | Classic small case |
| `1` | `1` | Single element; threshold is 1 |
| `1,2,3,4,5,6` | *(none)* | No element clears the bar |
| `4,4,4,4,5,6,7,8,4` | `4` | One strong winner amid noise |
