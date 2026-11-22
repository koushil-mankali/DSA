# Stock Problems and the State-Machine DP

> **Estimated Time:** 4 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - DP Intuition](../01-dp-intuition/README.md), [02 - 1D DP](../02-one-d-dp/README.md), [06 - String DP](../06-string-dp/README.md)

---

## What You'll Learn
- You can describe any stock problem with one state: (day, holding or not, transactions left).
- You can turn that state into a memo solution, then a table, then an O(1)-space loop.
- You can solve Stock I, II, III, IV, Cooldown and Transaction Fee by changing only one line of the same template.
- You can draw the "state machine" (free / holding / cooldown) and read the code straight from the arrows.
- You can solve Maximum Profit in Job Scheduling with DP plus binary search.

## The Big Idea

Think of a shopkeeper with a small cash box. Each morning she sees today's price for one item. She may buy one item (if her hands are empty), sell the one she holds, or do nothing. She wants the most money at the end. To decide today, she only needs to know two things: do I hold an item right now, and how many deals am I still allowed to make? She does not need to remember the whole history.

That is the whole chapter. The "state" is small:

- `i`: which day we are on.
- `h`: 0 if we hold nothing, 1 if we hold one share.
- `t`: how many transactions we may still start (only for the limited variants).

On each day we pick the better of:
- **Skip:** do nothing, move to the next day with the same `h`.
- **Act:** if `h = 0`, buy (pay `prices[i]`, now `h = 1`). If `h = 1`, sell (gain `prices[i]`, now `h = 0`).

Each stock variant only changes the small details of "act": a fee when selling, a one-day wait after selling, a cap on `t`.

Why interviewers love it: it looks like six different problems, but it is one idea. A candidate who sees the state machine solves all six. A candidate who memorises six tricks forgets them under pressure.

**Rule for every problem here:** you may hold at most one share at a time. You must sell before you buy again.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "prices[i] is the price on day i", "maximise profit" | Stock family. Define `(i, holding)`. |
| "buy once, sell once" | Stock I. One pass, track the minimum so far. |
| "as many transactions as you like" | Stock II. Sum every upward step, or the 2-state DP. |
| "at most k transactions" | Add `t` to the state. Stock III is k = 2, Stock IV is any k. |
| "cooldown one day after selling" | Selling jumps to day `i + 2` (or add a cooldown state). |
| "transaction fee" | Subtract the fee once per transaction (at sell, or at buy). |
| "jobs with start, end, profit; no overlap; maximise" | Sort + DP + binary search for the next compatible job. |
| "pick a set of non-overlapping intervals with weights" | Same as job scheduling (weighted interval scheduling). |

## Core Concepts / Template

### The state machine

With unlimited transactions there are only two boxes. Arrows are actions. Numbers on arrows are the money change.

```
          buy  (-price)
     +------------------------+
     |                        v
  [ FREE ] <---------------- [ HOLD ]
     ^  |    sell (+price)      |  |
     |__|                       |__|
    skip (0)                  skip (0)
```

Cooldown adds a third box. After a sale you cannot buy on the next day:

```
  FREE --buy--> HOLD --sell--> COOL --(next day, forced)--> FREE
```

Limited transactions multiply the boxes by `t`: FREE(t), HOLD(t). A buy moves from FREE(t) to HOLD(t) in the same count, a sell moves to FREE(t-1) (we count a transaction when it completes). Either way works if you are consistent. In this chapter the count drops on **sell**, so a transaction is "complete" only when the share is sold.

### One recursion for everything (memo form)

```js
// f(i, h, t) = best profit from day i onward
function f(i, h, t) {
  if (i === n || t === 0) return 0;
  const skip = f(i + 1, h, t);
  const act = h
    ? prices[i] + f(i + 1, 0, t - 1)   // sell: lose one transaction
    : -prices[i] + f(i + 1, 1, t);     // buy
  return Math.max(skip, act);
}
```

### How each variant changes it

| Variant | State | Buy (h = 0) | Sell (h = 1) | Answer |
|---|---|---|---|---|
| I: one transaction | `(i, h)` or t = 1 | `-p[i] + f(i+1, 1)` | `+p[i]` and stop | `f(0, 0)` |
| II: unlimited | `(i, h)` | `-p[i] + f(i+1, 1)` | `p[i] + f(i+1, 0)` | `f(0, 0)` |
| III: at most 2 | `(i, h, t)`, t = 2 | `-p[i] + f(i+1, 1, t)` | `p[i] + f(i+1, 0, t-1)` | `f(0, 0, 2)` |
| IV: at most k | `(i, h, t)` | same as III | same as III | `f(0, 0, k)` |
| Cooldown | `(i, h)` | `-p[i] + f(i+1, 1)` | `p[i] + f(i+2, 0)` | `f(0, 0)` |
| Fee | `(i, h)` | `-p[i] + f(i+1, 1)` | `p[i] - fee + f(i+1, 0)` | `f(0, 0)` |
| Job scheduling | `(i)` over sorted jobs | skip: `f(i+1)` | take: `profit[i] + f(next(i))` | `f(0)` |

### From memo to table to O(1)

1. **Memo:** add a cache keyed by the state. Same code, no repeated work.
2. **Table:** the recursion only looks at `i + 1` (or `i + 2`). So fill `dp[i]` from `i = n - 1` down to `0`.
3. **O(1) space:** `dp[i]` only needs `dp[i + 1]` (and `dp[i + 2]` for cooldown). Keep a few variables instead of the table.

The O(1) loops below use the **forward** view, which is easier to read: `hold` = best money while holding a share, `free` = best money while holding nothing. Both are "money so far". The backward table gives the same final answer.

### Reading the loop from the picture

For unlimited transactions, go day by day:

```
newFree = max(free, hold + p)   // stay free, or sell today
hold    = max(hold, free - p)   // stay holding, or buy today
```

Use `free` from before today when computing `hold` (store `newFree` in a temp). Buying and selling on the same day gives profit 0, so even a slip here is harmless for II, but it is a real bug for Cooldown and Fee. Be careful.

## Problems (easy -> hard)

### 1. Best Time to Buy and Sell Stock — Easy
**Companies:** Amazon, Google, Meta, Microsoft, Bloomberg
**Problem:** `prices[i]` is the price on day `i`. Buy once, then sell on a later day. Return the max profit (0 if no profit is possible). Example: `[7,1,5,3,6,4]` -> `5` (buy at 1, sell at 6).
**Intuition:** If you sell today, the best day to have bought is the day with the lowest price so far. So walk once, remember the lowest price seen, and check "today minus lowest" each day.
**Approach 1 - Brute force:** try every pair (buy day, later sell day). Time O(n^2), Space O(1).
**Approach 2 - State machine (memo/table):** this is the general template with `t = 1`: `f(i, h, t)` where a sale uses the one allowed transaction. It works, but it is more than this problem needs. Time O(n), Space O(n).
**Approach 3 - Optimal:** one pass with the running minimum. Time O(n), Space O(1).

Trace on `[7,1,5,3,6,4]`:

| day price | min so far | profit today | best |
|---|---|---|---|
| 7 | 7 | 0 | 0 |
| 1 | 1 | 0 | 0 |
| 5 | 1 | 4 | 4 |
| 3 | 1 | 2 | 4 |
| 6 | 1 | 5 | 5 |
| 4 | 1 | 3 | 5 |

```js
// memo (state machine with one transaction)
function maxProfit1Memo(prices) {
  const n = prices.length;
  const memo = Array.from({ length: n }, () => [[-1, -1], [-1, -1]]); // [i][h][t]
  function f(i, h, t) {
    if (i === n || t === 0) return 0;
    if (memo[i][h][t] !== -1) return memo[i][h][t];
    let ans = f(i + 1, h, t);
    ans = Math.max(ans, h ? prices[i] + f(i + 1, 0, t - 1) : -prices[i] + f(i + 1, 1, t));
    return (memo[i][h][t] = ans);
  }
  return f(0, 0, 1);
}

// tabulation (after selling, t = 0 so the future is worth 0)
function maxProfit1Tab(prices) {
  const n = prices.length;
  const dp = Array.from({ length: n + 1 }, () => [0, 0]);
  for (let i = n - 1; i >= 0; i--) {
    dp[i][1] = Math.max(dp[i + 1][1], prices[i]);
    dp[i][0] = Math.max(dp[i + 1][0], -prices[i] + dp[i + 1][1]);
  }
  return dp[0][0];
}

// optimal: O(1) space
function maxProfit1(prices) {
  let minPrice = Infinity, best = 0;
  for (const p of prices) {
    minPrice = Math.min(minPrice, p);
    best = Math.max(best, p - minPrice);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1) for the last version.
**Watch out:** if prices only fall, the answer is `0`, not negative. Empty array returns `0`. Do not sell before you buy: update the minimum first, then compute profit (same day gives 0, which is fine).

### 2. Best Time to Buy and Sell Stock II — Medium
**Companies:** Amazon, Google, Meta, Microsoft, Goldman Sachs
**Problem:** Same, but you may buy and sell as many times as you want (one share at a time). Example: `[7,1,5,3,6,4]` -> `7` (buy 1, sell 5; buy 3, sell 6).
**Intuition:** No cap means we can catch every single upward step. Two states are enough: `free` and `hold`. (A greedy "sum every positive difference" also works, but it does not extend to fee or cooldown, so learn the DP.)
**Approach 1 - Brute force:** try all buy/sell combinations. Exponential.
**Approach 2 - Better:** memo `f(i, h)` and table `dp[i][h]`. Time O(n), Space O(n).
**Approach 3 - Optimal:** two variables, `free` and `hold`. Time O(n), Space O(1).

Trace on `[7,1,5,3,6,4]` (`free` / `hold`, start `0` / `-inf`):

| price | newFree = max(free, hold+p) | hold = max(hold, free-p) |
|---|---|---|
| 7 | 0 | -7 |
| 1 | 0 | -1 |
| 5 | 4 | -1 |
| 3 | 4 | 1 |
| 6 | 7 | 1 |
| 4 | 7 | 3 |

Answer `free = 7`.

```js
// memo
function maxProfit2Memo(prices) {
  const n = prices.length;
  const memo = Array.from({ length: n }, () => [-1, -1]);
  function f(i, h) {
    if (i === n) return 0;
    if (memo[i][h] !== -1) return memo[i][h];
    const skip = f(i + 1, h);
    const act = h ? prices[i] + f(i + 1, 0) : -prices[i] + f(i + 1, 1);
    return (memo[i][h] = Math.max(skip, act));
  }
  return f(0, 0);
}

// tabulation
function maxProfit2Tab(prices) {
  const n = prices.length;
  const dp = Array.from({ length: n + 1 }, () => [0, 0]);
  for (let i = n - 1; i >= 0; i--) {
    dp[i][0] = Math.max(dp[i + 1][0], -prices[i] + dp[i + 1][1]);
    dp[i][1] = Math.max(dp[i + 1][1], prices[i] + dp[i + 1][0]);
  }
  return dp[0][0];
}

// optimal: O(1) space
function maxProfit2(prices) {
  let free = 0, hold = -Infinity;
  for (const p of prices) {
    const newFree = Math.max(free, hold + p);
    hold = Math.max(hold, free - p);
    free = newFree;
  }
  return free;
}
```

**Complexity:** Time O(n), Space O(1) for the last version (O(n) for memo and table).
**Watch out:** the memo recursion is `n` deep; for very large `n` it can overflow the JS stack. That is one reason the loop version is preferred. Start `hold` at `-Infinity` (you cannot hold before buying).

### 3. Best Time to Buy and Sell Stock with Cooldown — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Unlimited transactions, but after you sell you cannot buy the next day. Example: `[1,2,3,0,2]` -> `3` (buy 1, sell 2, rest, buy 0, sell 2).
**Intuition:** Selling on day `i` means the next chance to buy is day `i + 2`. In the recursion, a sale jumps by two days. In the machine, add a COOL box that you must pass through.
**Approach 1 - Brute force:** try all actions each day. Exponential.
**Approach 2 - Better:** memo `f(i, h)` with sell going to `f(i + 2, 0)`, then the table with `n + 2` rows. Time O(n), Space O(n).
**Approach 3 - Optimal:** three variables. `hold`: holding a share. `sold`: sold today. `rest`: free and allowed to buy. Time O(n), Space O(1).

Trace on `[1,2,3,0,2]` (start `hold = -inf, sold = 0, rest = 0`):

| price | sold = hold+p | hold = max(hold, rest-p) | rest = max(rest, prevSold) |
|---|---|---|---|
| 1 | -inf | -1 | 0 |
| 2 | 1 | -1 | 0 |
| 3 | 2 | -1 | 1 |
| 0 | -1 | 1 | 2 |
| 2 | 3 | 1 | 2 |

Answer `max(sold, rest) = 3`.

```js
// memo
function cooldownMemo(prices) {
  const n = prices.length;
  const memo = Array.from({ length: n }, () => [-1, -1]);
  function f(i, h) {
    if (i >= n) return 0;
    if (memo[i][h] !== -1) return memo[i][h];
    const skip = f(i + 1, h);
    const act = h ? prices[i] + f(i + 2, 0) : -prices[i] + f(i + 1, 1);
    return (memo[i][h] = Math.max(skip, act));
  }
  return f(0, 0);
}

// tabulation: two extra zero rows so dp[i + 2] is always valid
function cooldownTab(prices) {
  const n = prices.length;
  const dp = Array.from({ length: n + 2 }, () => [0, 0]);
  for (let i = n - 1; i >= 0; i--) {
    dp[i][0] = Math.max(dp[i + 1][0], -prices[i] + dp[i + 1][1]);
    dp[i][1] = Math.max(dp[i + 1][1], prices[i] + dp[i + 2][0]);
  }
  return dp[0][0];
}

// optimal: O(1) space
function cooldown(prices) {
  let hold = -Infinity, sold = 0, rest = 0;
  for (const p of prices) {
    const prevSold = sold;          // yesterday's "sold" becomes today's "rest"
    sold = hold + p;
    hold = Math.max(hold, rest - p); // buy only from rest, never from sold
    rest = Math.max(rest, prevSold);
  }
  return Math.max(sold, rest);
}
```

**Complexity:** Time O(n), Space O(1) for the last version.
**Watch out:** order of updates matters. Save `prevSold` before overwriting `sold`. `hold` must buy from `rest`, not from `max(rest, sold)`, or you break the cooldown. In the memo, base case is `i >= n` (not `===`) because of the `i + 2` jump.

### 4. Best Time to Buy and Sell Stock with Transaction Fee — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Unlimited transactions, but each completed transaction costs `fee`. Example: `prices = [1,3,2,8,4,9]`, `fee = 2` -> `8`.
**Intuition:** Same as Stock II. Charge the fee once per transaction. We choose to charge it on the sell.
**Approach 1 - Brute force:** all action sequences. Exponential.
**Approach 2 - Better:** memo and table. Time O(n), Space O(n).
**Approach 3 - Optimal:** the two-variable loop with `- fee` on the sell. Time O(n), Space O(1).

Trace on `[1,3,2,8,4,9]`, fee 2:

| price | newFree = max(free, hold+p-2) | hold = max(hold, free-p) |
|---|---|---|
| 1 | 0 | -1 |
| 3 | 0 | -1 |
| 2 | 0 | -1 |
| 8 | 5 | -1 |
| 4 | 5 | 1 |
| 9 | 8 | 1 |

Answer `8` (buy 1, sell 8 minus fee 2 = 5; buy 4, sell 9 minus fee 2 = 3; total 8).

```js
// memo
function feeMemo(prices, fee) {
  const n = prices.length;
  const memo = Array.from({ length: n }, () => [-1, -1]);
  function f(i, h) {
    if (i === n) return 0;
    if (memo[i][h] !== -1) return memo[i][h];
    const skip = f(i + 1, h);
    const act = h ? prices[i] - fee + f(i + 1, 0) : -prices[i] + f(i + 1, 1);
    return (memo[i][h] = Math.max(skip, act));
  }
  return f(0, 0);
}

// tabulation
function feeTab(prices, fee) {
  const n = prices.length;
  const dp = Array.from({ length: n + 1 }, () => [0, 0]);
  for (let i = n - 1; i >= 0; i--) {
    dp[i][0] = Math.max(dp[i + 1][0], -prices[i] + dp[i + 1][1]);
    dp[i][1] = Math.max(dp[i + 1][1], prices[i] - fee + dp[i + 1][0]);
  }
  return dp[0][0];
}

// optimal: O(1) space
function maxProfitFee(prices, fee) {
  let free = 0, hold = -Infinity;
  for (const p of prices) {
    const newFree = Math.max(free, hold + p - fee);
    hold = Math.max(hold, free - p);
    free = newFree;
  }
  return free;
}
```

**Complexity:** Time O(n), Space O(1) for the last version.
**Watch out:** charge the fee exactly once per transaction, not on both buy and sell. Never end with a share held: the answer is `free`, not `hold`. A sale that earns less than the fee is never taken because `max` keeps `free`.

### 5. Best Time to Buy and Sell Stock III — Hard
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** At most two transactions. Example: `[3,3,5,0,0,3,1,4]` -> `6` (buy 0, sell 3; buy 1, sell 4).
**Intuition:** Add the counter `t` to the state. With `t = 2` there are only four stages in a row: bought once, sold once, bought twice, sold twice. Each stage's best value depends on the stage before it.
**Approach 1 - Brute force:** choose two non-overlapping intervals. Time O(n^4) naive, O(n^2) with care.
**Approach 2 - Better:** memo `f(i, h, t)` and table `dp[i][h][t]`. Time O(n * 2 * 2), Space O(n).
**Approach 3 - Optimal:** four variables, one per stage. Time O(n), Space O(1).

Trace on `[3,3,5,0,0,3,1,4]`:

| price | b1 (buy 1) | s1 (sell 1) | b2 (buy 2) | s2 (sell 2) |
|---|---|---|---|---|
| 3 | -3 | 0 | -3 | 0 |
| 3 | -3 | 0 | -3 | 0 |
| 5 | -3 | 2 | -3 | 2 |
| 0 | 0 | 2 | 2 | 2 |
| 0 | 0 | 2 | 2 | 2 |
| 3 | 0 | 3 | 2 | 5 |
| 1 | 0 | 3 | 2 | 5 |
| 4 | 0 | 4 | 2 | 6 |

```js
// memo
function maxProfit3Memo(prices) {
  const n = prices.length;
  const memo = Array.from({ length: n }, () => Array.from({ length: 2 }, () => [-1, -1, -1]));
  function f(i, h, t) {
    if (i === n || t === 0) return 0;
    if (memo[i][h][t] !== -1) return memo[i][h][t];
    const skip = f(i + 1, h, t);
    const act = h ? prices[i] + f(i + 1, 0, t - 1) : -prices[i] + f(i + 1, 1, t);
    return (memo[i][h][t] = Math.max(skip, act));
  }
  return f(0, 0, 2);
}

// tabulation: generalised in problem 6 (same loops with k = 2)

// optimal: O(1) space, four stage variables
function maxProfit3(prices) {
  let b1 = -Infinity, s1 = 0, b2 = -Infinity, s2 = 0;
  for (const p of prices) {
    b1 = Math.max(b1, -p);       // best money after first buy
    s1 = Math.max(s1, b1 + p);   // after first sell
    b2 = Math.max(b2, s1 - p);   // after second buy (uses profit of first sell)
    s2 = Math.max(s2, b2 + p);   // after second sell
  }
  return s2;
}
```

**Complexity:** Time O(n), Space O(1) for the last version.
**Watch out:** the four updates run in order, each using the value just updated. That means "buy and sell on the same day", which adds 0 and is harmless. Initialise buys to `-Infinity` (or `-prices[0]`), never `0`.

### 6. Best Time to Buy and Sell Stock IV — Hard
**Companies:** Amazon, Google, Meta, Uber
**Problem:** At most `k` transactions. Example: `k = 2, prices = [3,2,6,5,0,3]` -> `7` (buy 2, sell 6; buy 0, sell 3).
**Intuition:** Stock III with `k` stages instead of 2. Keep arrays `hold[j]` and `free[j]` for `j` transactions. Shortcut: if `k >= n / 2`, you can never use all transactions (each needs at least two days), so the answer equals Stock II.
**Approach 1 - Brute force:** all action sequences. Exponential.
**Approach 2 - Better:** memo `f(i, h, t)`, then table `dp[i][h][t]`. Time O(n * k), Space O(n * k).
**Approach 3 - Optimal:** keep only the `k + 1` values for "today". Time O(n * k), Space O(k).

Trace on `k = 2, [3,2,6,5,0,3]` (`hold[1], free[1], hold[2], free[2]`):

| price | hold[1] | free[1] | hold[2] | free[2] |
|---|---|---|---|---|
| 3 | -3 | 0 | -3 | 0 |
| 2 | -2 | 0 | -2 | 0 |
| 6 | -2 | 4 | -2 | 4 |
| 5 | -2 | 4 | -1 | 4 |
| 0 | 0 | 4 | 4 | 4 |
| 3 | 0 | 4 | 4 | 7 |

(Rows show values after the day. Within a day, `j` goes from `k` down to `1`, so each uses the values from the previous day.)

```js
// memo
function maxProfitKMemo(k, prices) {
  const n = prices.length;
  const memo = Array.from({ length: n }, () => Array.from({ length: 2 }, () => new Array(k + 1).fill(-1)));
  function f(i, h, t) {
    if (i === n || t === 0) return 0;
    if (memo[i][h][t] !== -1) return memo[i][h][t];
    const skip = f(i + 1, h, t);
    const act = h ? prices[i] + f(i + 1, 0, t - 1) : -prices[i] + f(i + 1, 1, t);
    return (memo[i][h][t] = Math.max(skip, act));
  }
  return f(0, 0, k);
}

// tabulation: row n and column t = 0 stay zero
function maxProfitKTab(k, prices) {
  const n = prices.length;
  const dp = Array.from({ length: n + 1 }, () => Array.from({ length: 2 }, () => new Array(k + 1).fill(0)));
  for (let i = n - 1; i >= 0; i--) {
    for (let t = 1; t <= k; t++) {
      dp[i][0][t] = Math.max(dp[i + 1][0][t], -prices[i] + dp[i + 1][1][t]);
      dp[i][1][t] = Math.max(dp[i + 1][1][t], prices[i] + dp[i + 1][0][t - 1]);
    }
  }
  return dp[0][0][k];
}

// optimal: O(k) space
function maxProfitK(k, prices) {
  const n = prices.length;
  if (k >= n / 2) return maxProfit2(prices); // from problem 2: unlimited
  const hold = new Array(k + 1).fill(-Infinity); // hold[j]: holding, j-th buy done
  const free = new Array(k + 1).fill(0);         // free[j]: j sells done
  for (const p of prices) {
    for (let j = k; j >= 1; j--) {
      free[j] = Math.max(free[j], hold[j] + p);
      hold[j] = Math.max(hold[j], free[j - 1] - p);
    }
  }
  return free[k];
}
```

**Complexity:** Time O(n * k), Space O(k) for the last version.
**Watch out:** without the `k >= n / 2` shortcut, a huge `k` (like 10^9) allocates a giant array and times out. `k = 0` or an empty array returns `0`. Stock III is just this with `k = 2`.

### 7. Maximum Profit in Job Scheduling — Hard
**Companies:** Google, Amazon, Uber, Airbnb
**Problem:** Jobs have `startTime[i]`, `endTime[i]`, `profit[i]`. Pick non-overlapping jobs for the max total profit. A job that ends at time `X` does not overlap a job that starts at `X`. Example: start `[1,2,3,3]`, end `[3,4,5,6]`, profit `[50,10,40,70]` -> `120` (jobs 1 and 4).
**Intuition:** It is the same "take or skip" choice as the stock DP, but the "state" is a job index. Sort jobs by start. For job `i`: skip it (go to `i + 1`), or take it and jump to the first job whose start is `>= end[i]`. Binary search finds that jump fast.
**Approach 1 - Brute force:** try all subsets, check overlaps. Time O(2^n * n).
**Approach 2 - Better:** memo/table with a linear scan for the next job. Time O(n^2), Space O(n).
**Approach 3 - Optimal:** same DP with binary search for `next`. Time O(n log n), Space O(n).

Trace of the forward version on the example. Sort by end time: `(1,3,50) (2,4,10) (3,5,40) (3,6,70)`. Keep `ends = [0]`, `best = [0]`.

| job | latest slot with end <= start | profit if taken | action |
|---|---|---|---|
| (1,3,50) | ends[0]=0 -> best 0 | 50 | push: ends `[0,3]`, best `[0,50]` |
| (2,4,10) | ends[0]=0 -> best 0 | 10 | 10 <= 50, skip |
| (3,5,40) | ends[1]=3 -> best 50 | 90 | push: ends `[0,3,5]`, best `[0,50,90]` |
| (3,6,70) | ends[1]=3 -> best 50 | 120 | push; answer `120` |

```js
// first index in [lo, hi) with arr[idx] >= x   (returns hi if none)
function lowerBound(arr, x, lo, hi) {
  while (lo < hi) {
    const mid = (lo + hi) >> 1;
    if (arr[mid] >= x) hi = mid; else lo = mid + 1;
  }
  return lo;
}

// memo: sort by start; next = first later job starting at or after my end
function jobMemo(startTime, endTime, profit) {
  const jobs = startTime.map((s, i) => [s, endTime[i], profit[i]]).sort((a, b) => a[0] - b[0]);
  const n = jobs.length, starts = jobs.map(j => j[0]);
  const memo = new Array(n + 1).fill(-1);
  function f(i) {
    if (i >= n) return 0;
    if (memo[i] !== -1) return memo[i];
    const next = lowerBound(starts, jobs[i][1], i + 1, n);
    return (memo[i] = Math.max(f(i + 1), jobs[i][2] + f(next)));
  }
  return f(0);
}

// tabulation
function jobTab(startTime, endTime, profit) {
  const jobs = startTime.map((s, i) => [s, endTime[i], profit[i]]).sort((a, b) => a[0] - b[0]);
  const n = jobs.length, starts = jobs.map(j => j[0]);
  const dp = new Array(n + 1).fill(0);
  for (let i = n - 1; i >= 0; i--) {
    const next = lowerBound(starts, jobs[i][1], i + 1, n);
    dp[i] = Math.max(dp[i + 1], jobs[i][2] + dp[next]);
  }
  return dp[0];
}

// optimal: sort by end; ends/best are two parallel increasing lists
function jobScheduling(startTime, endTime, profit) {
  const jobs = startTime.map((s, i) => [s, endTime[i], profit[i]]).sort((a, b) => a[1] - b[1]);
  const ends = [0], best = [0]; // best[j] = max profit using jobs finishing by ends[j]
  for (const [s, e, p] of jobs) {
    const j = lowerBound(ends, s + 1, 0, ends.length) - 1; // last slot with end <= s
    const cand = best[j] + p;
    if (cand > best[best.length - 1]) { ends.push(e); best.push(cand); }
  }
  return best[best.length - 1];
}
```

**Complexity:** Time O(n log n) (sorting + n binary searches), Space O(n). There is no O(1)-space version: you need the table (or the two lists) to look up earlier answers.
**Watch out:** "end equals start" is allowed, so search for `start >= end` (not `>`). In the forward version search for the last end `<= s`, which is `lowerBound(ends, s + 1) - 1` for integer times. Sorting only by start but then binary searching on ends (or the reverse) is a classic bug. Profit values can add up large, but JavaScript numbers are safe here (up to 2^53).

## Common Mistakes
- **"I need a separate trick for each variant."** No. Same state, different transition. Learn the table at the top.
- **"Count the transaction on the buy."** You can, but be consistent. Counting on the sell (as here) means a half-finished transaction never uses up your quota wrongly.
- **"Start `hold` at 0."** Wrong. You cannot hold something you have not bought. Use `-Infinity` (or `-prices[0]`).
- **"Return `hold` at the end."** Wrong. Ending with a share is never better than selling it. Return the `free` / last sell value.
- **"Updating variables in place is fine."** Not when the new value uses the old value of another variable (cooldown, `k` loop order). Use a temp, or loop `j` from `k` down.
- **"Greedy sum of positive differences solves everything."** Only Stock II. It breaks with a fee, cooldown or a cap.
- **"Sort jobs by start but look up by end."** Keep the two orders straight: the memo form sorts by start, the forward form sorts by end.
- **"Large `k` is fine."** Cap it: if `k >= n / 2`, treat as unlimited.

## Interview Tips
- Say the state out loud first: "At each day I know if I hold a share, and how many transactions are left." Then write the recurrence. The code follows.
- Start with the memo recursion (easy to get right), then say "the recursion only reads day `i + 1`, so I can fill a table backward, then keep just a few variables."
- Name the base cases: `i == n` returns 0, `t == 0` returns 0.
- For Stock I, jump straight to the single pass. Mention the DP view only if asked about follow-ups.
- Follow-ups you will hear: "return the actual buy/sell days" (store the day where the min or the choice was made, or walk the table), "what if you may hold many shares" (a different problem, knapsack-like), "what if the fee is on the buy" (move `- fee` to the buy, answer unchanged), "what if the cooldown is `c` days" (jump to `i + c + 1`).
- Edge cases to mention: empty array, one price, strictly decreasing prices (answer 0), `k = 0`, very large `k`, fee larger than any gain, all jobs overlapping, jobs that touch (`end == start`).
- Mention recursion depth for large `n` and that the loop version avoids it.

## Practice List

**Easy**
- Best Time to Buy and Sell Stock
- Best Time to Buy and Sell Stock II
- Maximum Subarray (the same "best ending here" idea)

**Medium**
- Best Time to Buy and Sell Stock with Cooldown
- Best Time to Buy and Sell Stock with Transaction Fee
- Best Time to Buy and Sell Stock II (solve with greedy, then DP)
- Maximum Profit in Job Scheduling
- House Robber (the take-or-skip pattern)
- Maximum Product Subarray

**Hard**
- Best Time to Buy and Sell Stock III
- Best Time to Buy and Sell Stock IV
- Maximum Earnings From Taxi
- Two City Scheduling (a related choose-one-of-two idea)

## Key Takeaways
- **One state fits all:** (day, holding, transactions left).
- **Each variant changes one transition:** fee on sell, `i + 2` on sell for cooldown, `t - 1` on sell for a cap.
- **Memo -> table -> O(1):** the recursion only looks one (or two) days ahead, so keep a few variables.
- **Stock I is a special case** you can solve with a running minimum in one pass.
- **Cap `k` at `n / 2`** and Stock IV becomes Stock II.
- **Return the "free" state,** never the "holding" state, and start `hold` at `-Infinity`.
- **Job scheduling is the same take/skip DP,** with sorting and binary search to find the next compatible job.

### Summary table

| Variant | State | Transition (O(1) loop) | Time | Space |
|---|---|---|---|---|
| I (one) | min so far | `best = max(best, p - min)` | O(n) | O(1) |
| II (unlimited) | free, hold | `free = max(free, hold+p)`; `hold = max(hold, free-p)` | O(n) | O(1) |
| Cooldown | hold, sold, rest | `sold = hold+p`; `hold = max(hold, rest-p)`; `rest = max(rest, prevSold)` | O(n) | O(1) |
| Fee | free, hold | `free = max(free, hold+p-fee)`; `hold = max(hold, free-p)` | O(n) | O(1) |
| III (k = 2) | b1, s1, b2, s2 | `b1=max(b1,-p)`; `s1=max(s1,b1+p)`; `b2=max(b2,s1-p)`; `s2=max(s2,b2+p)` | O(n) | O(1) |
| IV (any k) | hold[j], free[j] | for `j = k..1`: `free[j]=max(free[j],hold[j]+p)`; `hold[j]=max(hold[j],free[j-1]-p)` | O(nk) | O(k) |
| Job scheduling | job index | `dp[i] = max(dp[i+1], profit[i] + dp[next(i)])` | O(n log n) | O(n) |

---
[← Previous: String DP](../06-string-dp/README.md) | [Next: Longest Increasing Subsequence →](../08-longest-increasing-subsequence/README.md) | [Back to Dynamic Programming Index](../README.md)
