# Subset and Subsequence DP (Take / Not-Take)

> **Estimated Time:** 4 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - DP Intuition](../01-dp-intuition/README.md), [02 - One-D DP](../02-one-d-dp/README.md)

---

## What You'll Learn
- You can write the **take / not-take** recursion for any "pick some items" problem and turn it into memo, a 2D table and a 1D array.
- You can solve **subset sum** and spot when other problems (partition, difference, target sum, last stone) are just subset sum in disguise.
- You can switch from "is it possible" (booleans) to "how many ways" (counts) and handle **zeros** correctly.
- You can handle **negative numbers** with an offset.
- You can explain why the 1D loop runs **backwards** for subsets, and **forwards over targets** for ordered combinations (Combination Sum IV).

## The Big Idea

Think about packing a suitcase. You look at each item one by one. For each item you ask: "Do I take it, or leave it?" You never take the same item twice. Your only worry is the total weight at the end.

That is the whole chapter. For every element there are two choices: **take** it (the target shrinks by its value) or **not take** it (the target stays). If you try all choices, you get 2^n subsets. But many choices lead to the same state `(index, remaining target)`. There are only n x target such states, so we remember them. That is DP.

Why interviewers love it: it is the entry door to the 0/1 knapsack family. A dozen "different" problems reduce to one table. If you see the reduction, the code is 8 lines. If you do not, you are lost. It tests pattern recognition, not typing.

The state:

```
dp[i][t] = using only the first i items, can we (or in how many ways can we) make sum t ?
```

The transition:

```
dp[i][t] = dp[i-1][t]                      // not take item i
         OR / +  dp[i-1][t - nums[i-1]]    // take item i (only if nums[i-1] <= t)
```

Booleans use OR. Counting uses `+`. Minimizing uses `min`. Same shape.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "Is there a subset with sum = K" | Boolean subset sum, `dp[t]` |
| "Split into two equal-sum parts" | Subset sum with target = total / 2 |
| "Split into two parts, minimize the difference" | Subset sum table, check sums up to total / 2 |
| "Count subsets with sum K" | Counting subset sum (`+` instead of OR) |
| "Count ways to split with difference D" | s1 = (total + D) / 2, count subsets with sum s1 |
| "Put + or - before each number to reach target" | Same as above: count subsets |
| "Smash stones, smallest remaining" | Minimum difference of two groups |
| "Count ordered sequences that add up to target" (1, 2 and 2, 1 differ) | Combination Sum IV: loop targets outside, items inside |
| "Array has zeros" / "array has negatives" | Careful base case / offset the sums |
| "Each item at most once" | Backward loop in 1D (0/1 style) |

## Core Concepts / Template

### 1. Memoized recursion (top-down)

State is `(i, t)`: we are deciding item `i`, and `t` is still needed. Base cases: `t === 0` means success (take nothing more). At `i === 0` there is only one item left, so success only if it equals `t`.

```js
function subsetSumMemo(nums, target) {
  if (nums.length === 0) return target === 0;
  const memo = new Map();
  function go(i, t) {
    if (t === 0) return true;
    if (i === 0) return nums[0] === t;
    const key = i * 100000 + t; // simple unique key; fine for small t
    if (memo.has(key)) return memo.get(key);
    let res = go(i - 1, t);                          // not take
    if (!res && nums[i] <= t) res = go(i - 1, t - nums[i]); // take
    memo.set(key, res);
    return res;
  }
  return go(nums.length - 1, target);
}
```

Time O(n x target), Space O(n x target) plus O(n) recursion stack.

### 2. Tabulation (bottom-up, 2D)

Row `i` means "first i items". Column `t` is the sum. Column 0 is always true (pick nothing).

```js
function subsetSumTab(nums, target) {
  const n = nums.length;
  const dp = Array.from({ length: n + 1 }, () => new Array(target + 1).fill(false));
  for (let i = 0; i <= n; i++) dp[i][0] = true;
  for (let i = 1; i <= n; i++) {
    for (let t = 1; t <= target; t++) {
      dp[i][t] = dp[i - 1][t];
      if (!dp[i][t] && nums[i - 1] <= t) dp[i][t] = dp[i - 1][t - nums[i - 1]];
    }
  }
  return dp[n][target];
}
```

Tiny trace: `nums = [2, 3, 5]`, `target = 5`. Rows are "first i items", columns are sums 0..5 (T = true).

| i (item added) | 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| 0 (none) | T | F | F | F | F | F |
| 1 (2) | T | F | T | F | F | F |
| 2 (3) | T | F | T | T | F | T |
| 3 (5) | T | F | T | T | F | T |

Read `dp[2][5]`: not take 3 gives `dp[1][5] = F`. Take 3 gives `dp[1][2] = T`. So T (2 + 3). Row 3 only copies row 2 plus `dp[2][0]` going to column 5 (already T).

### 3. The 1D optimization

Row `i` only reads row `i-1`. So keep a single row. The catch is the direction of the loop.

```js
function subsetSum(nums, target) {
  if (target < 0) return false;
  const dp = new Array(target + 1).fill(false);
  dp[0] = true;
  for (const x of nums)
    for (let t = target; t >= x; t--) // BACKWARD: dp[t - x] must still be "previous row"
      if (dp[t - x]) dp[t] = true;
  return dp[target];
}
```

Why backward? When we update `dp[t]` we read `dp[t - x]`, a smaller index. If we went forward, `dp[t - x]` might already include item `x`, so we would use the same item twice (that is unbounded knapsack, a different problem). Going backward, the smaller indexes are not yet touched, so they still mean "without this item".

Trace with `nums = [1, 2, 3]`, `target = 4` (array is indexes 0..4):

```
start        T F F F F
after x=1    T T F F F      dp[1] = dp[0]
after x=2    T T T T F      dp[3] = dp[1], dp[2] = dp[0]   (t=4: dp[2] was F, so stays F)
after x=3    T T T T T      dp[4] = dp[1] (1+3)
```

### 4. Memo vs tab vs 1 array: when to use what

| Version | Space | Pros | Cons |
|---|---|---|---|
| Memo (top-down) | O(n x T) + stack | Easiest to derive; only visits reachable states | Recursion depth n; Map overhead |
| Tab 2D | O(n x T) | No recursion; easy to debug; lets you rebuild the chosen items | More memory |
| 1D array | O(T) | Smallest and fastest in practice | Cannot rebuild the subset; direction matters |

In an interview: derive with recursion, say the state, then show tab, then say "I can cut space to O(T) by looping backwards".

### 5. Counting version and zeros

Swap OR for `+`. `dp[0] = 1` (one way to make 0: take nothing).

```js
function countSubsets(nums, k) {
  if (k < 0) return 0;
  const dp = new Array(k + 1).fill(0);
  dp[0] = 1;
  for (const x of nums)
    for (let t = k; t >= x; t--) dp[t] += dp[t - x];
  return dp[k];
}
```

Zeros: a zero can be taken or not taken without changing the sum, so every zero **doubles** the answer. The loop above already does this, because for `x = 0` the loop runs down to `t = 0` and does `dp[t] += dp[t]`. Note the condition is `t >= x`, not `t > x`. In the recursive version the same trap lives in the base case: at `i === 0`, if `nums[0] === 0` and `t === 0` there are **2** ways (take or skip), not 1.

## Problems (easy -> hard)

### 1. Subset Sum Equals Target — Easy
**Companies:** Amazon, Microsoft, Adobe
**Problem:** Given an array of non-negative integers and a target K, return true if some subset sums to K. Example: `[3, 34, 4, 12, 5, 2]`, K = 9 -> true (4 + 5).
**Intuition:** For each item choose take or skip. We only care about the sum so far, not which items. So the state is just `(index, sum)`.
**Approach 1 - Brute force:** Try all 2^n subsets. Time O(2^n), Space O(n).
**Approach 2 - Better:** Memoized `(i, t)` recursion or the 2D table shown above. Time O(n x K), Space O(n x K).
**Approach 3 - Optimal:** 1D boolean array, loop `t` backward. On `[2, 3, 5]`, K = 5: start `T F F F F F`; after 2 -> `T F T F F F`; after 3 -> `T F T T F T`; after 5 -> unchanged. `dp[5]` is true.

```js
function subsetSum(nums, target) {
  if (target < 0) return false;
  const dp = new Array(target + 1).fill(false);
  dp[0] = true;
  for (const x of nums)
    for (let t = target; t >= x; t--)
      if (dp[t - x]) dp[t] = true;
  return dp[target];
}
```

**Complexity:** Time O(n x K), Space O(K). This is "pseudo-polynomial": it depends on the value K, not just on the input length.
**Watch out:** Empty array with K = 0 is true. K = 0 is always true (empty subset). A `t >= x` loop with `x > K` simply does nothing, which is correct.

### 2. Partition Equal Subset Sum — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Can the array be split into two subsets with equal sum? Example: `[1, 5, 11, 5]` -> true (`[1, 5, 5]` and `[11]`).
**Intuition:** If the total is odd, impossible. Otherwise we need one subset with sum `total / 2`; the rest automatically gets the other half. So it is Problem 1 with a computed target.
**Approach 1 - Brute force:** Try all subsets. Time O(2^n), Space O(n).
**Approach 2 - Better:** Memo / 2D table on `(i, total/2)`. Time O(n x S), Space O(n x S).
**Approach 3 - Optimal:** Odd total -> false. Otherwise 1D subset sum with target `S = total / 2`. On `[1, 5, 11, 5]`: total 22, target 11. After 1: {0,1}. After 5: {0,1,5,6}. After 11: includes 11 -> true.

```js
function canPartition(nums) {
  const sum = nums.reduce((a, b) => a + b, 0);
  if (sum % 2) return false;
  const target = sum / 2;
  const dp = new Array(target + 1).fill(false);
  dp[0] = true;
  for (const x of nums)
    for (let t = target; t >= x; t--)
      if (dp[t - x]) dp[t] = true;
  return dp[target];
}
```

**Complexity:** Time O(n x S), Space O(S), where S = total / 2.
**Watch out:** Check the odd total first. If one number is larger than `S`, the loop skips it, and the answer is correctly false unless the rest can reach `S`. A common bug is writing `sum / 2` for an odd `sum` and getting a decimal index.

### 3. Count Subsets with Sum K (Perfect Sum, with zeros) — Medium
**Companies:** Amazon, Flipkart, Microsoft
**Problem:** Count subsets whose sum is K. The array can contain zeros. Example: `[1, 2, 2, 3]`, K = 3 -> 3 (`{1,2}`, `{1,2}` using the other 2, `{3}`). `[0, 0, 1]`, K = 1 -> 4.
**Intuition:** Same take / not-take, but add the counts of the two branches instead of OR-ing. Zeros are special: taking a zero changes nothing, so each zero doubles the count.
**Approach 1 - Brute force:** Enumerate all subsets and count. Time O(2^n), Space O(n).
**Approach 2 - Better:** Memo on `(i, t)`: `go(i,t) = go(i-1,t) + go(i-1,t-nums[i])`. Base at `i === 0`: if `t === 0 && nums[0] === 0` return 2; if `t === 0 || nums[0] === t` return 1; else 0. Time O(n x K), Space O(n x K).
**Approach 3 - Optimal:** 1D count array, backward loop, `t >= x` so zeros are included. Trace `[1,2,2,3]`, K = 3:

```
start        [1, 0, 0, 0]      (index 0..3)
x = 1        [1, 1, 0, 0]
x = 2        [1, 1, 1, 1]      dp[3] += dp[1], dp[2] += dp[0]
x = 2        [1, 1, 2, 2]
x = 3        [1, 1, 2, 3]      dp[3] += dp[0]
```

Answer `dp[3] = 3`.

```js
function countSubsets(nums, k) {
  if (k < 0) return 0;
  const dp = new Array(k + 1).fill(0);
  dp[0] = 1; // empty subset
  for (const x of nums)
    for (let t = k; t >= x; t--) dp[t] += dp[t - x]; // x = 0 doubles dp[t]
  return dp[k];
}
```

**Complexity:** Time O(n x K), Space O(K).
**Watch out:** Zeros. With `[0, 0]`, K = 0 the answer is 4 (`{}`, `{0}`, `{0}`, `{0,0}`). Using `t > x` instead of `t >= x`, or returning 1 at the base case when `nums[0] === 0`, gives wrong answers. Counts can grow large; if the problem asks for modulo (usually 1e9 + 7), apply `% MOD` on each addition.

### 4. Partition a Set Into Two Subsets with Minimum Absolute Difference — Medium
**Companies:** Amazon, Google, Adobe
**Problem:** Split the array into two groups to make `|sum1 - sum2|` as small as possible. Example: `[1, 6, 11, 5]` -> 1 (`{1,5,6}` = 12 and `{11}` = 11).
**Intuition:** If one group has sum `s`, the other has `total - s`, so the difference is `total - 2s`. We want `s` as close as possible to `total / 2`, from below. So find every reachable subset sum, then pick the best `s <= total / 2`.
**Approach 1 - Brute force:** Try all subsets, track the best. Time O(2^n), Space O(n).
**Approach 2 - Better:** 2D reachable-sum table, then scan the last row. Time O(n x total), Space O(n x total).
**Approach 3 - Optimal:** 1D reachability up to `total`, then scan `s` from 0 to `total / 2`. On `[1,6,11,5]`: total 23, reachable sums include 0,1,5,6,7,11,12,... The best `s <= 11` is 11, giving `23 - 22 = 1`.

```js
function minSubsetDiff(nums) {
  const sum = nums.reduce((a, b) => a + b, 0);
  const dp = new Array(sum + 1).fill(false);
  dp[0] = true;
  for (const x of nums)
    for (let t = sum; t >= x; t--)
      if (dp[t - x]) dp[t] = true;
  let best = sum;
  for (let s = 0; s <= sum >> 1; s++)
    if (dp[s]) best = Math.min(best, sum - 2 * s);
  return best;
}
```

**Complexity:** Time O(n x total), Space O(total).
**Watch out:** Only scan `s` up to `total / 2` (the other half is the mirror). Empty array gives 0. You can limit the table to `total / 2` instead of `total` to halve the work.

### 5. Count Partitions with Given Difference — Medium
**Companies:** Amazon, Flipkart, Walmart
**Problem:** Count ways to split the array into two subsets (S1, S2) with `S1 - S2 = D`. Example: `[5, 2, 6, 4]`, D = 3 -> 1 (`{6,4}` vs `{5,2}`: 10 - 7 = 3).
**Intuition:** We know two things: `S1 + S2 = total` and `S1 - S2 = D`. Add them: `S1 = (total + D) / 2`. Now the question is just "count subsets with sum S1" (Problem 3).
**Approach 1 - Brute force:** Try every subset as S1. Time O(2^n).
**Approach 2 - Better:** Memo on `(i, S1)`. Time O(n x S1), Space O(n x S1).
**Approach 3 - Optimal:** If `D > total` or `(total - D)` is odd, return 0. Else count subsets with sum `(total - D) / 2`, same as `(total + D) / 2` for the complement. Example: total 17, D = 3, so the smaller group S2 must sum to (17 - 3) / 2 = 7. Subsets with sum 7: `{5,2}` only -> 1. Each split is counted once, because we always count the smaller side S2 (the complement is S1).

```js
function countPartitions(nums, d) {
  const sum = nums.reduce((a, b) => a + b, 0);
  if (d > sum || (sum - d) % 2) return 0;
  return countSubsets(nums, (sum - d) / 2); // from Problem 3, handles zeros
}
```

**Complexity:** Time O(n x (total - D) / 2), Space O(total).
**Watch out:** Parity check, and `D > total`. Zeros in the array again double the answer, which is why we reuse the zero-safe counter. This version assumes `D >= 0`.

### 6. Target Sum — Medium
**Companies:** Facebook, Google, Amazon, Bloomberg
**Problem:** Put `+` or `-` in front of each number so the expression equals `target`. Count the ways. Example: `[1,1,1,1,1]`, target = 3 -> 5.
**Intuition:** Numbers with `+` form group P, numbers with `-` form group N. Then `P - N = target` and `P + N = total`. This is exactly "count partitions with difference = target" (Problem 5).
**Approach 1 - Brute force:** Try both signs for every number. Time O(2^n), Space O(n).
**Approach 2 - Better:** Memo on `(index, running sum)` using a map, since the sum ranges from `-total` to `total`. Time O(n x total), Space O(n x total).
**Approach 3 - Optimal:** Reduce to subset count with sum `(total + target) / 2`. For `[1,1,1,1,1]`, target 3: total 5, `(5 + 3) / 2 = 4`. Ways to pick four 1s out of five: 5. Done.

```js
function findTargetSumWays(nums, target) {
  const sum = nums.reduce((a, b) => a + b, 0);
  if (Math.abs(target) > sum || (sum + target) % 2) return 0;
  const k = (sum + target) / 2; // sum of the "+" group
  const dp = new Array(k + 1).fill(0);
  dp[0] = 1;
  for (const x of nums)
    for (let t = k; t >= x; t--) dp[t] += dp[t - x];
  return dp[k];
}
```

**Complexity:** Time O(n x k), Space O(k).
**Watch out:** Negative target: use `Math.abs` in the range check. `(sum + target) % 2` works for negatives in JS because we only test truthiness (`-3 % 2` is `-1`, which is truthy). Zeros: `[0,0,0,0,0,0,0,0,1]`, target 1 -> 256, because each zero can be `+0` or `-0`. The `t >= x` loop handles this.

### 7. Last Stone Weight II — Medium
**Companies:** Amazon, Google
**Problem:** Repeatedly pick any two stones and smash them: equal stones both vanish, otherwise the heavier one keeps the difference. Return the smallest possible weight of the last stone (0 if none). Example: `[2,7,4,1,8,1]` -> 1.
**Intuition:** Smashing a and b leaves `a - b`. Over the whole process, each stone ends up with a `+` or a `-` sign in one big expression. So we split stones into two groups and the answer is the difference of group sums. Minimize it: Problem 4.
**Approach 1 - Brute force:** Simulate all smash orders. Exponential and hard to code.
**Approach 2 - Better:** Try all sign assignments. Time O(2^n).
**Approach 3 - Optimal:** Min subset difference. Stones sum to 23, so half = 11. The sum 11 is reachable (`{7,4}`), so the answer is `23 - 2 x 11 = 1`.

```js
function lastStoneWeightII(stones) {
  const sum = stones.reduce((a, b) => a + b, 0);
  const half = sum >> 1;
  const dp = new Array(half + 1).fill(false);
  dp[0] = true;
  for (const x of stones)
    for (let t = half; t >= x; t--)
      if (dp[t - x]) dp[t] = true;
  for (let s = half; s >= 0; s--)
    if (dp[s]) return sum - 2 * s;
}
```

**Complexity:** Time O(n x sum / 2), Space O(sum / 2).
**Watch out:** Do not confuse with "Last Stone Weight" (the heap version). This one is DP because you may choose any pair, not the two heaviest. Table only needs size `sum / 2 + 1`; scanning down from `half` finds the best `s` first.

### 8. Combination Sum IV — Medium
**Companies:** Google, Amazon, Facebook
**Problem:** Given distinct positive numbers, count ordered sequences that add up to target (order matters: `1+2` and `2+1` are different). Example: `[1,2,3]`, target 4 -> 7.
**Intuition:** This is NOT a subset problem: items can repeat and order counts. Think of "last number used". `ways(t) = sum over x of ways(t - x)`. Put the **target loop outside** and the **items inside**; that is what makes different orders count separately.
**Approach 1 - Brute force:** Recursion `ways(t) = sum ways(t - x)`. Exponential.
**Approach 2 - Better:** Memoize on `t`. Time O(T x n), Space O(T).
**Approach 3 - Optimal:** Bottom-up array. For `[1,2,3]`: `dp = [1, 1, 2, 4, 7]` (`dp[4] = dp[3] + dp[2] + dp[1] = 4 + 2 + 1`).

```js
function combinationSum4(nums, target) {
  const dp = new Array(target + 1).fill(0);
  dp[0] = 1;
  for (let t = 1; t <= target; t++)
    for (const x of nums)          // items INSIDE -> order matters
      if (x <= t) dp[t] += dp[t - x];
  return dp[target];
}
```

**Complexity:** Time O(target x n), Space O(target).
**Watch out:** Swap the loops (items outside, target inside) and you count **combinations** (order ignored), which is the Coin Change II answer. For `[1,2,3]`, target 4 that is 4: `{1,1,1,1}`, `{1,1,2}`, `{2,2}`, `{1,3}`. Know which one the question wants. Answers can overflow in other languages; in JS numbers go inexact past 2^53, so use BigInt or modulo if asked. The classic follow-up: if negatives are allowed, infinite sequences exist, so you must limit the length.

### 9. Subset Sum with Negative Numbers (Offset) — Medium
**Companies:** -
**Problem:** The array may contain negative numbers. Is there a subset with sum exactly K? Example: `[-3, 2, 5]`, K = -1 -> true (`-3 + 2`).
**Intuition:** Array indexes cannot be negative. But reachable sums live in a known range: from `neg` (sum of all negatives) to `pos` (sum of all positives). Shift every sum by `-neg` so the smallest sum maps to index 0.
**Approach 1 - Brute force:** All subsets. Time O(2^n).
**Approach 2 - Better:** Memo with a Map keyed by `(i, sum)` (Map handles negative keys naturally). Time O(n x range), Space O(n x range).
**Approach 3 - Optimal:** Boolean array of size `pos - neg + 1`; index `s + offset` means sum `s`. Because the 1D backward trick relies on moving only in one direction, with mixed signs it is safer to build a **new row** from the old one each round.
Trace `[-3, 2, 5]`: neg = -3, pos = 7, offset = 3, size 11. Reachable sums start {0}. After -3: {0, -3}. After 2: {0,-3,2,-1}. After 5: {0,-3,2,-1,5,7,4}. K = -1 is in the set -> true.

```js
function subsetSumNeg(nums, target) {
  let neg = 0, pos = 0;
  for (const x of nums) x < 0 ? (neg += x) : (pos += x);
  if (target < neg || target > pos) return false;
  const off = -neg, size = pos - neg + 1;
  let dp = new Array(size).fill(false);
  dp[off] = true; // sum 0 reachable by the empty subset
  for (const x of nums) {
    const next = dp.slice(); // not-take keeps old sums
    for (let s = 0; s < size; s++)
      if (dp[s] && s + x >= 0 && s + x < size) next[s + x] = true; // take
    dp = next;
  }
  return dp[target + off];
}
```

**Complexity:** Time O(n x (pos - neg)), Space O(pos - neg).
**Watch out:** Quick range check first (`target` outside `[neg, pos]` is impossible). The index of sum 0 is `off`, not 0. Copy the row; updating in place would let one item be used twice when `x` is negative (the loop would move left, so a backward loop no longer protects you). For counting instead of boolean, the same offset idea works with `+=`.

## Common Mistakes
- **"I can loop `t` forward in 1D."** Forward means an item can be reused (unbounded knapsack). For each item used once, loop `t` from high to low.
- **"Base case: `t === 0` returns 1 only when index is 0."** With zeros in the array, `dp[0][0]` can be 2. For counting, use the zero-safe 1D version or handle `nums[0] === 0` explicitly.
- **"Use `t > x` in the loop."** That skips `x === t`, and skips zeros. Use `t >= x`.
- **"Odd total can be split in half."** It cannot. Check `total % 2` before dividing.
- **"Target Sum needs a big sign-tracking DP."** It reduces to counting subsets with sum `(total + target) / 2`. Check the parity and `|target| <= total`.
- **"Combination Sum IV is a subset problem."** It allows repeats and counts order. Items inside, target outside.
- **"The result is O(n x K), so it is fast."** It is pseudo-polynomial. If K is 10^9 it is useless; mention it.
- **Mutating the table while iterating with negatives.** Use a fresh row (or an offset and a copy).

## Interview Tips
- Say the state out loud first: "`dp[i][t]` = can the first i items make sum t". Then give the take / not-take transition. This alone earns most of the credit.
- Name the reduction early: "Partition equal subset is subset sum with target total / 2." "Target Sum is count subsets with `(total + target) / 2`." Interviewers love hearing the algebra (`S1 + S2 = total`, `S1 - S2 = D`).
- Mention complexity as **pseudo-polynomial**, and say what happens when K is huge (meet in the middle: O(2^(n/2)), or bitset shifts: `bits |= bits << x` with BigInt in JS).
- Edge cases to mention: empty array, K = 0, zeros, negative numbers, odd total, a single huge element, target larger than the total.
- Follow-ups: "Print the subset" (use the 2D table and walk back from `dp[n][K]`: if `dp[i-1][t]` is true skip item i, else take it). "Reduce memory" (1D backward loop). "Count modulo 1e9 + 7". "What if numbers are huge but n is small?" (meet in the middle).
- Write the memo version first if the tab version does not come quickly. Correct and slower beats fast and wrong.

## Practice List
**Easy**
- Subset Sum Problem (GfG / TUF)
- Last Stone Weight (heap version, to contrast)
- Climbing Stairs (1D take-style warm up)

**Medium**
- Partition Equal Subset Sum
- Target Sum
- Last Stone Weight II
- Combination Sum IV
- Coin Change II (unbounded, combinations)
- Perfect Sum Problem
- Partitions with Given Difference
- Ones and Zeroes (2D budget knapsack)
- Partition to K Equal Sum Subsets (needs bitmask or backtracking)
- Tallest Billboard

**Hard**
- Number of Ways to Reorder Array to Get Same BST (counting with combinations)
- Split Array With Same Average
- Minimum Difference Subsets of a 2n array (meet in the middle)
- Closest Subsequence Sum

## Key Takeaways
- Every item has two choices: **take or not take**. State is `(index, remaining target)`.
- Booleans use **OR**, counts use **+**, optimization uses **min / max**. The shape stays the same.
- For items used once, the 1D loop runs **backward** over the target.
- Partition, min difference, Target Sum and Last Stone Weight II are all **subset sum in disguise**: use `S1 + S2 = total` and `S1 - S2 = D`.
- **Zeros double the count**; use `t >= x` and be careful with the base case.
- **Negatives need an offset** (shift by the sum of negatives) and a fresh row per item.
- **Combination Sum IV is different**: target outside, items inside, because order counts.
- The cost is **pseudo-polynomial**, O(n x K); say so.

---
[← Previous: Grid DP](../03-grid-dp/README.md) | [Next: Knapsack →](../05-knapsack/README.md) | [Back to Dynamic Programming Index](../README.md)
