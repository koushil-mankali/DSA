# Knapsack Patterns

> **Estimated Time:** 5 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - DP Intuition](../01-dp-intuition/README.md), [04 - Subset and Subsequence DP](../04-subset-and-subsequence-dp/README.md)

---

## What You'll Learn
- You can tell 0/1, unbounded and bounded knapsack apart in one read of a problem.
- You can write a knapsack as memo, as a 2D table, and as a 1D array.
- You can explain why the 1D loop goes backward for 0/1 and forward for unbounded.
- You can pick the right loop order for "combinations" versus "permutations" counting.
- You can turn disguised problems (coin change, target sum, perfect squares) into knapsack.

## The Big Idea
Imagine you pack a backpack for a trip. Each item has a weight and a value. The bag can only hold so much weight. For every item you ask one question: "Do I take it or leave it?" You want the best total value that still fits.

That is the whole pattern. You have a list of items and a budget (weight, amount, count, members). For each item you choose to take it or not, and the budget shrinks when you take it. The state is `(item index, budget left)`.

There are three flavours, and the only difference is how many copies of an item you may use:

| Flavour | Copies per item | Example |
|---|---|---|
| 0/1 knapsack | 0 or 1 | Classic knapsack, Target Sum, Ones and Zeroes |
| Unbounded knapsack | any number | Coin Change, Rod Cutting, Perfect Squares |
| Bounded knapsack | up to `cnt[i]` | Items in stock with a limit |

Why interviewers love it: the pattern hides inside many stories (coins, strings, squares, schemes). They want to see if you notice "pick or skip with a budget", and if you understand the 1D loop direction. That direction question is the classic follow-up.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "each item can be used at most once", "choose a subset" | 0/1 knapsack, inner loop goes backward |
| "unlimited supply", "any number of times" | unbounded, inner loop goes forward |
| "fewest coins", "minimum number of ..." | knapsack with `min`, init `Infinity`, `dp[0] = 0` |
| "number of ways to make amount" | knapsack with `+`, init `dp[0] = 1` |
| "number of ways, order matters" (1,2 differs from 2,1) | amount in the outer loop (permutations) |
| "number of ways, order does not matter" | items in the outer loop (combinations) |
| "assign + or - to each number to reach target" | convert to subset sum |
| two budgets (zeros and ones, members and profit) | 2D budget, still 0/1 |
| "from each pile take a prefix" | group knapsack: one group per pile |
| each item has a count limit | bounded: binary splitting into 0/1 items |

## Core Concepts / Template

### The recurrence
Let `dp[i][c]` be the best answer using the first `i` items with budget `c`.

```
dp[i][c] = max( dp[i-1][c],                        // skip item i
                value[i] + dp[i-1][c - weight[i]] )   // take item i (only if weight[i] <= c)
```

For unbounded, taking an item does not use it up, so the second branch stays on row `i`:
`value[i] + dp[i][c - weight[i]]`.

### Three ways to code 0/1 knapsack

```js
// 1) Memo (top-down)
function knapMemo(wt, val, W) {
  const n = wt.length, memo = Array.from({ length: n }, () => Array(W + 1).fill(-1));
  const go = (i, cap) => {
    if (i === n) return 0;
    if (memo[i][cap] !== -1) return memo[i][cap];
    let best = go(i + 1, cap);                                   // skip
    if (wt[i] <= cap) best = Math.max(best, val[i] + go(i + 1, cap - wt[i])); // take
    return (memo[i][cap] = best);
  };
  return go(0, W);
}

// 2) Tabulation (2D)
function knapTab(wt, val, W) {
  const n = wt.length;
  const dp = Array.from({ length: n + 1 }, () => Array(W + 1).fill(0));
  for (let i = 1; i <= n; i++)
    for (let c = 0; c <= W; c++) {
      dp[i][c] = dp[i - 1][c];
      if (wt[i - 1] <= c) dp[i][c] = Math.max(dp[i][c], val[i - 1] + dp[i - 1][c - wt[i - 1]]);
    }
  return dp[n][W];
}

// 3) 1D, space O(W)
function knap1D(wt, val, W) {
  const dp = Array(W + 1).fill(0);
  for (let i = 0; i < wt.length; i++)
    for (let c = W; c >= wt[i]; c--)            // BACKWARD
      dp[c] = Math.max(dp[c], val[i] + dp[c - wt[i]]);
  return dp[W];
}
```

### Why the direction matters in 1D
The 2D table needs only row `i-1` to build row `i`. So we keep one row and overwrite it. The question is: when we compute `dp[c]`, does `dp[c - w]` still hold the OLD row (item `i-1`) or the NEW row (item `i`)?

- **Backward (c from W down to w):** `dp[c - w]` is at a smaller index, so it has not been touched yet in this round. It is the old value. Each item is used at most once. This is 0/1.
- **Forward (c from w up to W):** `dp[c - w]` was already updated in this round. It may already include item `i`. So item `i` can be reused again and again. This is unbounded.

A tiny proof by example. One item, weight 2, value 3, budget 6.

| Loop | dp after the item | Meaning |
|---|---|---|
| Backward | `[0,0,3,3,3,3,3]` | item used once, best is 3 |
| Forward | `[0,0,3,3,6,6,9]` | item reused 3 times, best is 9 |

So the same code with only the loop direction flipped changes the problem from 0/1 to unbounded. Forward on a 0/1 problem is a classic bug: it silently gives too-big answers.

### Trace of the 1D 0/1 knapsack
Items (weight, value): (1,1), (3,4), (4,5), (5,7). Budget 7. Index = budget.

| After item | dp[0..7] |
|---|---|
| start | 0 0 0 0 0 0 0 0 |
| (1,1) | 0 1 1 1 1 1 1 1 |
| (3,4) | 0 1 1 4 5 5 5 5 |
| (4,5) | 0 1 1 4 5 6 6 9 |
| (5,7) | 0 1 1 4 5 7 8 9 |

Answer `dp[7] = 9` (take weights 3 and 4: value 4 + 5).

### Loop order for counting ways (combinations vs permutations)
Take coins `[1,2]` and amount 3.

- **Items outer, amount inner:** each coin is "decided" once, in a fixed order. `1+2` and `2+1` look the same. This counts **combinations**. Answer 2: `{1,1,1}`, `{1,2}`.
- **Amount outer, items inner:** at every amount we try every coin as the LAST coin. So `1,2` and `2,1` are different. This counts **permutations**. Answer 3: `1,1,1`, `1,2`, `2,1`.

```js
// combinations: coins outer
for (const c of coins) for (let a = c; a <= amount; a++) dp[a] += dp[a - c];
// permutations: amount outer
for (let a = 1; a <= amount; a++) for (const c of coins) if (c <= a) dp[a] += dp[a - c];
```

### Bounded knapsack
Item `i` has `cnt[i]` copies. Simple way: expand into `cnt[i]` separate 0/1 items. Faster way: **binary splitting**. Split `cnt` into packs of size 1, 2, 4, ..., and a remainder. Any number from 0 to `cnt` can be built from these packs, and each pack is used 0 or 1 times. That turns `cnt` items into about `log(cnt)` items.

## Problems (easy -> hard)

### 1. 0/1 Knapsack — Medium
**Companies:** -
**Problem:** Given weights, values and a capacity `W`, find the maximum total value of items whose total weight is at most `W`. Each item is used at most once. Example: wt=[1,3,4,5], val=[1,4,5,7], W=7 -> 9.
**Intuition:** For each item, either skip it (budget unchanged) or take it (budget drops by its weight, value goes up). The best of the two is the answer for that state.
**Approach 1 - Brute force:** Try all 2^n subsets. Time O(2^n), Space O(n).
**Approach 2 - Better:** Memoize `(i, cap)`. There are `n * W` states, each O(1). Time O(n*W), Space O(n*W). A 2D table does the same bottom-up.
**Approach 3 - Optimal:** Keep one row. Loop capacity backward so each item is used once. Trace is in the Core Concepts section above (answer 9).

```js
function knapsack01(wt, val, W) {
  const dp = Array(W + 1).fill(0);
  for (let i = 0; i < wt.length; i++)
    for (let c = W; c >= wt[i]; c--)
      dp[c] = Math.max(dp[c], val[i] + dp[c - wt[i]]);
  return dp[W];
}
```

**Complexity:** Time O(n*W), Space O(W).
**Watch out:** O(n*W) is "pseudo-polynomial": it depends on the size of the number `W`, not just the input length. Say this out loud, interviewers like it. Items heavier than `W` must be skipped (the loop bound `c >= wt[i]` does that).

### 2. Unbounded Knapsack — Medium
**Companies:** -
**Problem:** Same as above, but each item can be used any number of times. Example: wt=[2,3], val=[3,5], W=7 -> 11 (two 2s and one 3: 3+3+5).
**Intuition:** Taking an item does not remove it from the shop. So after taking it, we stay on the same item. In 1D this means the loop runs forward.
**Approach 1 - Brute force:** Recursively try every item at every step. Exponential.
**Approach 2 - Better:** Memo on `(i, cap)` with the take branch staying on `i`. Time O(n*W), Space O(n*W).
**Approach 3 - Optimal:** 1D, forward loop. `dp[c]` may already contain item `i`, which is exactly what we want.

```js
function unboundedKnapsack(wt, val, W) {
  const dp = Array(W + 1).fill(0);
  for (let i = 0; i < wt.length; i++)
    for (let c = wt[i]; c <= W; c++)             // FORWARD
      dp[c] = Math.max(dp[c], val[i] + dp[c - wt[i]]);
  return dp[W];
}
```

**Complexity:** Time O(n*W), Space O(W).
**Watch out:** If every item is heavier than `W`, the answer is 0, not an error. Do not copy the backward loop from problem 1.

### 3. Rod Cutting — Medium
**Companies:** -
**Problem:** A rod has length `n`. `price[i]` is the price of a piece of length `i+1`. Cut the rod to get the maximum total price. Example: price=[1,5,8,9,10,17,17,20], n=8 -> 22 (pieces 2 and 6: 5+17).
**Intuition:** It is unbounded knapsack. Piece length is the "weight", price is the "value", rod length is the capacity. Any piece length can be reused.
**Approach 1 - Brute force:** Try every first cut, recurse on the rest. Time O(2^n).
**Approach 2 - Better:** Memoize on remaining length. Time O(n^2), Space O(n).
**Approach 3 - Optimal:** Bottom-up: `dp[len] = max over cut of price[cut-1] + dp[len-cut]`.

Trace for price=[1,5,8,9], n=4:

| len | best cut | dp[len] |
|---|---|---|
| 1 | 1 (1) | 1 |
| 2 | 2 (5) | 5 |
| 3 | 3 (8) | 8 |
| 4 | 2+2 (5+5) | 10 |

```js
function rodCut(price, n) {
  const dp = Array(n + 1).fill(0);
  for (let len = 1; len <= n; len++)
    for (let cut = 1; cut <= Math.min(len, price.length); cut++)
      dp[len] = Math.max(dp[len], price[cut - 1] + dp[len - cut]);
  return dp[n];
}
```

**Complexity:** Time O(n^2), Space O(n).
**Watch out:** The price list may be shorter than `n`; the `Math.min` guard handles that. Not cutting at all is allowed (`cut = len`).

### 4. Coin Change — Medium
**Companies:** Amazon, Google, Meta, Microsoft, Apple
**Problem:** Given coin values and an amount, return the fewest coins that make the amount, or -1 if impossible. Example: coins=[1,2,5], amount=11 -> 3 (5+5+1).
**Intuition:** Unbounded knapsack where we minimize the count. `dp[a]` = fewest coins for amount `a`. Try each coin as the last coin: `dp[a] = min(dp[a - coin] + 1)`.
**Approach 1 - Brute force:** Try all coin sequences. Exponential.
**Approach 2 - Better:** Greedy (always take the largest coin) is wrong. For coins [1,3,4] and 6, greedy gives 4+1+1 = 3 coins, but 3+3 = 2 coins.
**Approach 3 - Optimal:** Bottom-up DP. `dp[0] = 0`, everything else `Infinity` (unreachable).

Trace for coins=[1,2,5], amount=5: dp = [0, 1, 1, 2, 2, 1]. At 5, the coin 5 gives `dp[0]+1 = 1`.

```js
function coinChange(coins, amount) {
  const dp = Array(amount + 1).fill(Infinity);
  dp[0] = 0;
  for (let a = 1; a <= amount; a++)
    for (const c of coins)
      if (c <= a && dp[a - c] + 1 < dp[a]) dp[a] = dp[a - c] + 1;
  return dp[amount] === Infinity ? -1 : dp[amount];
}
```

**Complexity:** Time O(amount * coins), Space O(amount).
**Watch out:** Amount 0 returns 0. Use `Infinity`, not a small number like `amount + 1`, only if you are careful: `Infinity + 1` is still `Infinity`, which is safe. Never use greedy.

### 5. Perfect Squares — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Find the fewest perfect squares (1, 4, 9, ...) that sum to `n`. Example: 12 -> 3 (4+4+4).
**Intuition:** It is Coin Change where the coins are the squares up to `n`. Unbounded, minimize count.
**Approach 1 - Brute force:** Recursion over all squares. Exponential.
**Approach 2 - Better:** Coin-change DP, loop amount outer, squares inner. Time O(n * sqrt(n)).
**Approach 3 - Optimal:** Same DP written as items outer, amount inner (below). For one number, number theory (Lagrange's four-square theorem) gives an O(sqrt(n)) answer, but the DP is what interviews expect.

```js
function numSquares(n) {
  const dp = Array(n + 1).fill(Infinity);
  dp[0] = 0;
  for (let s = 1; s * s <= n; s++)
    for (let a = s * s; a <= n; a++)
      dp[a] = Math.min(dp[a], dp[a - s * s] + 1);
  return dp[n];
}
```

**Complexity:** Time O(n * sqrt(n)), Space O(n).
**Watch out:** The answer always exists (1+1+...+1), and is at most 4. Do not forget `n` itself can be a square (answer 1).

### 6. Coin Change II — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Count the number of combinations of coins that make `amount`. Order does not matter. Example: amount=5, coins=[1,2,5] -> 4.
**Intuition:** Same unbounded knapsack, but we add instead of min. To avoid counting `1+2` and `2+1` twice, we put **coins in the outer loop**. Each coin is fully processed before the next.
**Approach 1 - Brute force:** Enumerate all multisets. Exponential.
**Approach 2 - Better:** Memo on `(coin index, remaining)`. Time O(n*amount), Space O(n*amount).
**Approach 3 - Optimal:** 1D with coins outer, amount forward.

Trace for coins=[1,2,5], amount=5 (index = amount):

| After coin | dp[0..5] |
|---|---|
| start | 1 0 0 0 0 0 |
| 1 | 1 1 1 1 1 1 |
| 2 | 1 1 2 2 3 3 |
| 5 | 1 1 2 2 3 4 |

```js
function change(amount, coins) {
  const dp = Array(amount + 1).fill(0);
  dp[0] = 1;                                    // one way to make 0: take nothing
  for (const c of coins)
    for (let a = c; a <= amount; a++) dp[a] += dp[a - c];
  return dp[amount];
}
```

**Complexity:** Time O(n * amount), Space O(amount).
**Watch out:** Swapping the two loops silently turns this into permutations (problem 7). Amount 0 gives 1, not 0. The answer can be large; in LeetCode it fits in 32 bits, but in other settings ask about overflow or a modulo.

### 7. Combination Sum IV — Medium
**Companies:** Amazon, Google, Meta
**Problem:** Count sequences of numbers from `nums` (reuse allowed) that add up to `target`. Different orders count as different. Example: nums=[1,2,3], target=4 -> 7. Despite the name, it counts permutations.
**Intuition:** Ask "what was the LAST number I added?" Any of the numbers can be last, so `dp[t] = sum of dp[t - x]`. That makes the **amount the outer loop**.
**Approach 1 - Brute force:** Recursion over all sequences. Exponential.
**Approach 2 - Better:** Memoize on remaining target. Time O(target * n).
**Approach 3 - Optimal:** Bottom-up, amount outer, items inner.

Trace for nums=[1,2,3], target=4: dp = [1, 1, 2, 4, 7]. For example `dp[3] = dp[2] + dp[1] + dp[0] = 2 + 1 + 1 = 4`.

```js
function combinationSum4(nums, target) {
  const dp = Array(target + 1).fill(0);
  dp[0] = 1;
  for (let a = 1; a <= target; a++)
    for (const x of nums) if (x <= a) dp[a] += dp[a - x];
  return dp[target];
}
```

**Complexity:** Time O(target * n), Space O(target).
**Watch out:** Compare with problem 6: the only difference is the loop order. Follow-up: if negative numbers were allowed, infinite sequences appear (`+1, -1, +1, ...`), so you would need a length limit.

### 8. Target Sum — Medium
**Companies:** Facebook, Google, Amazon
**Problem:** Put `+` or `-` in front of every number. Count the ways the result equals `target`. Example: nums=[1,1,1,1,1], target=3 -> 5.
**Intuition:** Let `P` be the sum of numbers with `+`, `N` the sum with `-`. Then `P - N = target` and `P + N = total`. Add them: `P = (total + target) / 2`. So the question becomes: "How many subsets sum to `P`?" That is 0/1 knapsack counting.
**Approach 1 - Brute force:** Try 2^n sign choices. Time O(2^n).
**Approach 2 - Better:** Memo on `(index, running sum)`. Time O(n * total), Space O(n * total).
**Approach 3 - Optimal:** Subset-sum counting in 1D, backward loop.

Trace for nums=[1,1,1,1,1], target=3: total=5, `P = 4`. Count subsets with sum 4: choose 4 of 5 ones = 5.

```js
function findTargetSumWays(nums, target) {
  const total = nums.reduce((a, b) => a + b, 0);
  const diff = total + target;
  if (Math.abs(target) > total || diff % 2) return 0; // impossible
  const s = diff / 2, dp = Array(s + 1).fill(0);
  dp[0] = 1;
  for (const x of nums)
    for (let j = s; j >= x; j--) dp[j] += dp[j - x];
  return dp[s];
}
```

**Complexity:** Time O(n * s), Space O(s) where `s = (total + target) / 2`.
**Watch out:** Check `|target| > total` and odd `total + target` first, or you may build a negative-size array. Zeros in `nums` double the count (a `0` can be `+0` or `-0`); the backward loop `j >= x` with `x = 0` handles it, and `[0,0,0]` with target 0 gives 8.

### 9. Ones and Zeroes — Medium
**Companies:** Google, Amazon
**Problem:** Given binary strings and limits `m` zeros and `n` ones, find the largest subset of strings that uses at most `m` zeros and `n` ones in total. Example: ["10","0001","111001","1","0"], m=5, n=3 -> 4.
**Intuition:** It is 0/1 knapsack with TWO budgets. Each string is an item that costs `(zeros, ones)` and is worth 1.
**Approach 1 - Brute force:** All subsets. Time O(2^k).
**Approach 2 - Better:** Memo on `(string index, m left, n left)`. Time O(k*m*n), Space O(k*m*n).
**Approach 3 - Optimal:** One 2D array `dp[i][j]`, updated per string, both loops backward.

Trace idea: for string "10" (1 zero, 1 one), `dp[i][j] = max(dp[i][j], dp[i-1][j-1] + 1)`.

```js
function findMaxForm(strs, m, n) {
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0));
  for (const s of strs) {
    let z = 0, o = 0;
    for (const ch of s) ch === '0' ? z++ : o++;
    for (let i = m; i >= z; i--)
      for (let j = n; j >= o; j--)
        dp[i][j] = Math.max(dp[i][j], dp[i - z][j - o] + 1);
  }
  return dp[m][n];
}
```

**Complexity:** Time O(k * m * n) plus O(total string length) to count digits, Space O(m * n).
**Watch out:** Both budget loops must go backward. Count zeros and ones per string before the loops, not inside them.

### 10. Maximum Value of K Coins From Piles — Hard
**Companies:** Google
**Problem:** There are piles of coins. In one move you take the top coin of any pile. Do exactly `k` moves and maximize the total value. Example: piles=[[1,100,3],[7,8,9]], k=2 -> 101.
**Intuition:** This is a **group knapsack**. Each pile is a group. From a group you choose how many coins `t` to take (0 to size), and the value is the prefix sum of the top `t` coins. You pick exactly one choice per group, like an item with several options.
**Approach 1 - Brute force:** Try all ways to split `k` among piles. Exponential.
**Approach 2 - Better:** Memo on `(pile index, coins left)`, trying every `t`. Time O(k * total coins), Space O(piles * k).
**Approach 3 - Optimal:** 1D over coins used. For each pile, `j` goes backward (0/1 across groups), and an inner loop tries `t`.

Trace for piles=[[1,100,3],[7,8,9]], k=2. Prefix of pile 1: [0,1,101,104]. After pile 1: dp = [0,1,101]. For pile 2 (prefix [0,7,15,24]): `dp[2] = max(101, dp[1]+7 = 8, dp[0]+15 = 15) = 101`.

```js
function maxValueOfCoins(piles, k) {
  const dp = Array(k + 1).fill(0);
  for (const pile of piles) {
    const pre = [0];
    for (const x of pile) pre.push(pre[pre.length - 1] + x);
    for (let j = k; j >= 1; j--)               // backward: one choice per pile
      for (let t = 1; t <= Math.min(j, pile.length); t++)
        dp[j] = Math.max(dp[j], dp[j - t] + pre[t]);
  }
  return dp[k];
}
```

**Complexity:** Time O(k * S) where `S` is the total number of coins, Space O(k).
**Watch out:** Only top coins are reachable, so you must take a prefix. Do not sort coins. Backward `j` is what stops you from taking from the same pile twice. Here `dp[j]` means "best using at most `j` coins" (it starts at 0 everywhere), and since all values are positive, using all `k` is always at least as good.

### 11. Profitable Schemes — Hard
**Companies:** Google
**Problem:** There are `n` members. Crime `i` needs `group[i]` members and gives `profit[i]`. Count the subsets of crimes with profit at least `minProfit` using at most `n` members in total. Return the answer modulo 1e9+7. Example: n=5, minProfit=3, group=[2,2], profit=[2,3] -> 2.
**Intuition:** 0/1 knapsack with two dimensions: members (a budget, at most) and profit (a goal, at least). The trick: **cap the profit at `minProfit`**. Any profit above the goal is just "enough", so we do not need more states.
**Approach 1 - Brute force:** All subsets, check both conditions. Time O(2^k).
**Approach 2 - Better:** Memo on `(crime, members used, profit capped)`. Time O(k*n*minProfit), Space O(k*n*minProfit).
**Approach 3 - Optimal:** 2D array `dp[m][p]` = number of schemes using exactly `m` members with capped profit exactly `p`. For each crime, loop `m` downward and push to `min(minProfit, q + profit)`.

Trace for n=5, minProfit=3, group=[2,2], profit=[2,3]:
- start: `dp[0][0] = 1`
- after crime 0 (g=2, p=2): `dp[2][2] = 1`
- after crime 1 (g=2, p=3): from `dp[0][0]` -> `dp[2][3] += 1`; from `dp[2][2]` -> `dp[4][3] += 1` (profit 5 capped to 3)
- sum `dp[m][3]` over `m`: 1 + 1 = 2.

```js
function profitableSchemes(n, minProfit, group, profit) {
  const MOD = 1e9 + 7;
  const dp = Array.from({ length: n + 1 }, () => Array(minProfit + 1).fill(0));
  dp[0][0] = 1;
  for (let k = 0; k < group.length; k++) {
    const g = group[k], pr = profit[k];
    for (let m = n; m >= g; m--)               // downward: row m-g is still the old one
      for (let q = 0; q <= minProfit; q++) {
        const np = Math.min(minProfit, q + pr);
        dp[m][np] = (dp[m][np] + dp[m - g][q]) % MOD;
      }
  }
  let ans = 0;
  for (let m = 0; m <= n; m++) ans = (ans + dp[m][minProfit]) % MOD;
  return ans;
}
```

**Complexity:** Time O(k * n * minProfit), Space O(n * minProfit).
**Watch out:** The empty scheme counts when `minProfit = 0` (it is the `dp[0][0]` start). This relies on `group[i] >= 1`; with a group of size 0 the in-place update would reuse the same row. Take the modulo on every addition.

### 12. Bounded Knapsack with Item Counts — Hard
**Companies:** -
**Problem:** Item `i` has weight `wt[i]`, value `val[i]` and at most `cnt[i]` copies. Maximize the value within capacity `W`. Example: wt=[2,3], val=[3,4], cnt=[2,1], W=7 -> 10 (two items of weight 2, and the weight-3 item: 3+3+4).
**Intuition:** Bounded sits between 0/1 and unbounded. Splitting `cnt` into packs of 1, 2, 4, ... and a remainder lets us build every count from 0 to `cnt`. Each pack is then a plain 0/1 item.
**Approach 1 - Brute force:** Try every count for every item. Exponential.
**Approach 2 - Better:** Expand each item into `cnt[i]` copies and run 0/1. Time O(W * sum of cnt).
**Approach 3 - Optimal:** Binary splitting. Time O(W * sum of log(cnt)).

Trace for `cnt = 13`: packs are 1, 2, 4, then remainder 6 (13 - 7). Check: any number 0..13 is a sum of distinct packs, and nothing above 13 is possible (1+2+4+6 = 13).

```js
function boundedKnapsack(wt, val, cnt, W) {
  const dp = Array(W + 1).fill(0);
  for (let i = 0; i < wt.length; i++) {
    let left = cnt[i];
    for (let take = 1; left > 0; take *= 2) {
      const t = Math.min(take, left);          // last pack is the remainder
      left -= t;
      const w = t * wt[i], v = t * val[i];     // one pack = one 0/1 item
      for (let c = W; c >= w; c--) dp[c] = Math.max(dp[c], dp[c - w] + v);
    }
  }
  return dp[W];
}
```

**Complexity:** Time O(W * sum of log(cnt[i])), Space O(W).
**Watch out:** The remainder pack must be `min(take, left)`, not another power of two, or you could take more copies than allowed. Loop must be backward since packs are 0/1. If `cnt` is huge (like 1e9), an O(W) monotonic-queue trick exists, but it is beyond a normal interview.

## Common Mistakes
- **"The inner loop direction does not matter."** It decides 0/1 vs unbounded. Backward = each item once. Forward = items reusable.
- **"Coin Change II and Combination Sum IV are the same loops."** Items outer counts combinations. Amount outer counts permutations.
- **"Greedy works for coin change."** It fails for coins like [1,3,4]. Use DP.
- **"`dp` starts as all zeros."** For min problems start with `Infinity` and `dp[0] = 0`. For counting start with `dp[0] = 1`.
- **"Unreachable states are fine to read."** `Infinity + 1` is safe in JS, but `-1` as a sentinel will give wrong mins. Pick one clear sentinel.
- **"Target Sum needs a 2D sum table with negatives."** Convert to subset sum with `(total + target) / 2` and check parity.
- **"The knapsack is polynomial."** It is O(n*W), pseudo-polynomial. The problem itself is NP-hard in general.
- **"In-place 2D (Ones and Zeroes) can loop forward."** Both dimensions must go backward, or an item gets reused.

## Interview Tips
- Say the recurrence first: "take it or skip it, budget goes down when I take it." Then pick memo, then optimize to 1D.
- State the three questions out loud: can I reuse items, does order matter, am I minimizing, maximizing or counting.
- When you go to 1D, say: "I iterate backward so `dp[c - w]` still holds the previous item's row." That sentence shows real understanding.
- For counting, explain the loop order with the `[1,2]` and amount 3 example (2 versus 3 answers).
- Follow-ups you may get: recover which items were chosen (keep the 2D table and walk back), reduce space (1D), handle huge capacity (meet in the middle or switch the DP over values instead of weights), handle item counts (binary splitting).
- Edge cases to mention: capacity 0, empty items, item heavier than capacity, amount 0, impossible target, large counts needing a modulo.
- To rebuild the chosen items you need the 2D table, because the 1D array overwrites history.

## Practice List
**Easy**
- Climbing Stairs (as a unbounded count with steps 1 and 2)
- Min Cost Climbing Stairs

**Medium**
- Partition Equal Subset Sum
- Last Stone Weight II
- Coin Change
- Coin Change II
- Combination Sum IV
- Perfect Squares
- Target Sum
- Ones and Zeroes
- Word Break
- Minimum Cost For Tickets
- Number of Dice Rolls With Target Sum
- Partition to K Equal Sum Subsets
- Knapsack with Duplicate Items (GfG: Unbounded Knapsack)

**Hard**
- Maximum Value of K Coins From Piles
- Profitable Schemes
- Minimum Difference in Sums After Removal of Elements
- Tallest Billboard
- Number of Ways to Form a Target String Given a Dictionary

## Key Takeaways
- **Knapsack = pick or skip with a shrinking budget.** State is (item, budget left).
- **0/1 means backward in 1D, unbounded means forward.** Direction decides whether `dp[c - w]` is old or new.
- **Min problems start at Infinity with dp[0] = 0; counting starts at dp[0] = 1.**
- **Items outer = combinations, amount outer = permutations.** Same recurrence, different loop order.
- **Target Sum is subset sum in disguise:** `P = (total + target) / 2`.
- **Two budgets, two backward loops** (Ones and Zeroes); a "goal" budget can be capped (Profitable Schemes).
- **Group knapsack** (piles) takes one choice per group; **bounded knapsack** uses binary splitting.

---
[← Previous: Subset and Subsequence DP](../04-subset-and-subsequence-dp/README.md) | [Next: String DP →](../06-string-dp/README.md) | [Back to Dynamic Programming Index](../README.md)
