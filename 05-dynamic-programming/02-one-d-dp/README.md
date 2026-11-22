# One-Dimensional DP

> **Estimated Time:** 6 hr | **Difficulty:** Intermediate
> **Prerequisites:** [05 - DP Intuition](../01-dp-intuition/README.md)

---

## What You'll Learn
- You can turn a "best answer for the first i items" question into a 1D `dp` array.
- You can write a recurrence, pick base cases, and shrink the array to one or two variables.
- You can tell the three 1D shapes apart: **take or skip**, **try every last step**, and **running best ending here**.
- You can solve about twenty classic 1D DP interview problems, from House Robber to Word Break.

## The Big Idea
Think of climbing a staircase. To know how many ways reach step 10, you do not re-walk the whole staircase. You already know the answers for steps 9 and 8, and you combine them. One-dimensional DP is exactly this: one number (an index, an amount, a length) describes your "state", and each answer is built from smaller answers you already stored.

Why bother? Plain recursion solves the same small question again and again, and the time explodes (often `O(2^n)`). A 1D array stores each answer once, so the time drops to `O(n)` or `O(n * k)`.

Interviewers love 1D DP because it is the entry point to all DP. If you can define the state, write the recurrence and explain the base case out loud, the 2D and knapsack problems become variations of the same skill.

## How to Spot It
| If the problem says ... | think ... |
|---|---|
| "maximum / minimum total, you cannot pick two neighbours" | take-or-skip DP (House Robber) |
| "fewest items / minimum coins to reach a target" | `dp[target] = 1 + min(dp[target - item])` |
| "number of ways to reach / build X" | `dp[x] = sum of dp[smaller states]` |
| "best subarray ending at i" | Kadane: `cur = max(x, cur + x)` |
| "can the string be split / decoded" | `dp[i]` = is the prefix of length i valid |
| "can you reach the end, minimum jumps" | reachability DP, or greedy as a shortcut |
| "answer for n depends on n-1, n-2, n-3" | rolling variables, `O(1)` space |

## Core Concepts / Template

Four steps, every time:
1. **State:** what does `dp[i]` mean in one sentence? (Write it down. Most bugs start here.)
2. **Recurrence:** how does `dp[i]` come from earlier cells?
3. **Base case:** which cells are known without a formula (`dp[0]`)?
4. **Answer:** which cell holds the final result (`dp[n]`, or the max over all cells)?

Three shapes cover almost everything in this chapter.

**Shape A: take or skip (linear choice).**
```js
// best[i] = max(best[i-1] /* skip i */, best[i-2] + val[i] /* take i */)
function takeOrSkip(vals) {
  let prev = 0, cur = 0;              // best[i-2], best[i-1]
  for (const v of vals) [prev, cur] = [cur, Math.max(cur, prev + v)];
  return cur;
}
```

**Shape B: try every last step (unbounded choices).**
```js
// dp[a] = best way to make amount a; try each last coin c
function lastStep(coins, amount) {
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0;                          // base: nothing needed for amount 0
  for (let a = 1; a <= amount; a++)
    for (const c of coins) if (c <= a) dp[a] = Math.min(dp[a], dp[a - c] + 1);
  return dp[amount];
}
```

**Shape C: best ending here (subarray / substring).** Keep `cur` = best value of a run that must end at `i`, and a separate `best` = max of all `cur`. This is Kadane.

Tiny trace of Shape A on `[2, 7, 9, 3, 1]` (House Robber):

| item | prev | cur (after) |
|---|---|---|
| start | 0 | 0 |
| 2 | 0 | 2 |
| 7 | 2 | 7 |
| 9 | 7 | 11 |
| 3 | 11 | 11 |
| 1 | 11 | 12 |

**Rolling variables.** If `dp[i]` only needs the last 1 to 3 cells, keep just those variables. Space goes from `O(n)` to `O(1)`. Write the array version first, then shrink it.

## Problems (easy -> hard)

### 1. House Robber — Easy
**Companies:** Amazon, Google, Microsoft, Cisco
**Problem:** Houses in a row hold money. You cannot rob two neighbours. Return the maximum money. Example: `[2,7,9,3,1]` -> `12`.
**Intuition:** At each house you only decide "rob it or skip it". If you rob house i, the previous house is off limits, so you add to the best from two houses back.
**Approach 1 - Brute force:** Recursion with two branches per house. Time O(2^n), Space O(n).
**Approach 2 - Better:** Memoised recursion or a `dp` array: `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`. Time O(n), Space O(n).
**Approach 3 - Optimal:** Only the last two values matter, so keep two variables. Trace is the table in Core Concepts: `0,2,7,11,11,12`.

```js
function rob(nums) {
  let prev = 0, cur = 0;                       // best up to i-2, best up to i-1
  for (const x of nums) [prev, cur] = [cur, Math.max(cur, prev + x)];
  return cur;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Empty input should return 0 (it does here). Do not "alternate" houses blindly: `[2,1,1,2]` is best by robbing the two ends, `4`.

### 2. Maximum Subarray — Easy
**Companies:** Amazon, Microsoft, Google, Meta, Apple
**Problem:** Find the contiguous subarray with the largest sum. Example: `[-2,1,-3,4,-1,2,1,-5,4]` -> `6` (subarray `[4,-1,2,1]`).
**Intuition:** (Kadane as DP.) Let `dp[i]` be the best sum of a subarray that ends exactly at i. Either it starts fresh at i, or it extends the best one ending at i-1. A negative running sum only hurts, so we drop it.
**Approach 1 - Brute force:** Try every start and end. Time O(n^2) with a running sum, Space O(1).
**Approach 2 - Better:** Divide and conquer. Time O(n log n).
**Approach 3 - Optimal:** `dp[i] = max(nums[i], dp[i-1] + nums[i])`, answer is the max over all `dp[i]`.

| x | -2 | 1 | -3 | 4 | -1 | 2 | 1 | -5 | 4 |
|---|---|---|---|---|---|---|---|---|---|
| cur | -2 | 1 | -2 | 4 | 3 | 5 | 6 | 1 | 5 |
| best | -2 | 1 | 1 | 4 | 4 | 5 | 6 | 6 | 6 |

```js
function maxSubArray(nums) {
  let cur = nums[0], best = nums[0];           // start from nums[0], not 0
  for (let i = 1; i < nums.length; i++) {
    cur = Math.max(nums[i], cur + nums[i]);    // restart or extend
    best = Math.max(best, cur);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** If all numbers are negative, the answer is the largest single number, not 0. Initialising `best = 0` is the classic bug.

### 3. Jump Game — Medium
**Companies:** Amazon, Microsoft, Google, Meta
**Problem:** `nums[i]` is the max jump length from i. Can you reach the last index? `[2,3,1,1,4]` -> `true`, `[3,2,1,0,4]` -> `false`.
**Intuition:** As DP, "index i is reachable" depends on earlier reachable indices. But you do not need the whole table: just track the farthest index reachable so far. If you ever stand on an index beyond it, you are stuck.
**Approach 1 - Brute force:** Recursion trying every jump size. Exponential.
**Approach 2 - Better:** DP table `reach[i]`: for each reachable i mark `i+1..i+nums[i]`. Time O(n^2), Space O(n).
**Approach 3 - Optimal:** Keep `far`. For `[3,2,1,0,4]`: far goes 3,3,3,3, then at i=4 we have `4 > 3`, so return false.

```js
function canJump(nums) {
  let far = 0;                                 // farthest reachable index
  for (let i = 0; i < nums.length; i++) {
    if (i > far) return false;                 // gap we cannot cross
    far = Math.max(far, i + nums[i]);
  }
  return true;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** `[0]` is `true` (already at the end). A zero in the middle is only a problem if you cannot jump past it.

### 4. Perfect Squares — Medium
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** Fewest perfect squares (1, 4, 9, ...) that add up to n. Example: `12` -> `3` (`4+4+4`).
**Intuition:** This is Coin Change where the coins are the squares. The last square used is some `s*s`, so `dp[i] = 1 + min(dp[i - s*s])`.
**Approach 1 - Brute force:** Recursion over every square. Exponential.
**Approach 2 - Better:** Bottom-up DP below, Time O(n sqrt n), Space O(n).
**Approach 3 - Optimal:** Lagrange's four-square theorem says the answer is at most 4, which allows an `O(sqrt n)` math solution. It is a trick; in an interview the DP is the expected answer.

Trace for n=12: `dp[1..12] = 1,2,3,1,2,3,4,2,1,2,3,3`. For 12: `min(dp[11]+1, dp[8]+1, dp[3]+1) = min(4,3,4) = 3`.

```js
function numSquares(n) {
  const dp = new Array(n + 1).fill(Infinity);
  dp[0] = 0;
  for (let i = 1; i <= n; i++)
    for (let s = 1; s * s <= i; s++) dp[i] = Math.min(dp[i], dp[i - s * s] + 1);
  return dp[n];
}
```

**Complexity:** Time O(n sqrt n), Space O(n).
**Watch out:** `dp[0] = 0` is the base. Without it every cell stays Infinity.

### 5. Coin Change — Medium
**Companies:** Amazon, Google, Microsoft, Meta, Apple
**Problem:** Given coin values (unlimited supply) and an amount, return the fewest coins that make it, or `-1`. Example: `[1,2,5], 11` -> `3` (`5+5+1`).
**Intuition:** Greedy (always take the biggest coin) fails, for example coins `[1,3,4]`, amount 6 (greedy gives 4+1+1, best is 3+3). So try every coin as the last coin: `dp[a] = 1 + min(dp[a - c])`.
**Approach 1 - Brute force:** Recursion, try each coin at each step. Exponential.
**Approach 2 - Better:** Top-down with memo. Time O(amount * coins).
**Approach 3 - Optimal:** Bottom-up table, same time, no recursion depth problem.

| a | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| dp | 0 | 1 | 1 | 2 | 2 | 1 | 2 | 2 | 3 | 3 | 2 | 3 |

```js
function coinChange(coins, amount) {
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0;
  for (let a = 1; a <= amount; a++)
    for (const c of coins) if (c <= a) dp[a] = Math.min(dp[a], dp[a - c] + 1);
  return dp[amount] === Infinity ? -1 : dp[amount];
}
```

**Complexity:** Time O(amount * n), Space O(amount).
**Watch out:** `amount = 0` returns 0. Use `Infinity` (not `-1`) as the "impossible" marker inside the table, so `min` ignores it; convert to `-1` only at the end.

### 6. Decode Ways — Medium
**Companies:** Meta, Amazon, Google, Microsoft, Uber
**Problem:** `A=1, B=2, ..., Z=26`. Count how many ways a digit string can be decoded. Example: `"226"` -> `3` (`BZ`, `VF`, `BBF`).
**Intuition:** The last letter came from either the last one digit (1-9) or the last two digits (10-26). So `dp[i] = (one digit valid ? dp[i-1] : 0) + (two digits valid ? dp[i-2] : 0)`. Same shape as climbing stairs, with validity checks.
**Approach 1 - Brute force:** Recursion splitting 1 or 2 digits. Exponential.
**Approach 2 - Better:** `dp` array of size n+1. Time O(n), Space O(n).
**Approach 3 - Optimal:** Two rolling variables.

Trace `"226"`: start `prev=1, cur=1`. At `'2'` (pair 22 valid): `cur = 1 + 1 = 2`. At `'6'` (pair 26 valid): `cur = 2 + 1 = 3`.

```js
function numDecodings(s) {
  let prev = 1, cur = s[0] === '0' ? 0 : 1;     // ways for prefix i-2 and i-1
  for (let i = 1; i < s.length; i++) {
    let next = 0;
    if (s[i] !== '0') next += cur;              // '1'..'9' alone
    const two = Number(s.slice(i - 1, i + 1));
    if (two >= 10 && two <= 26) next += prev;   // '10'..'26' together
    prev = cur; cur = next;
  }
  return cur;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Zeros. `"06"` is 0 ways, `"10"` is 1 way, `"100"` is 0 ways. A pair like `"07"` is not valid (needs `>= 10`).

### 7. House Robber II — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Same as House Robber, but the houses form a circle, so the first and last are neighbours. Example: `[2,3,2]` -> `3`.
**Intuition:** First and last cannot both be robbed. So solve two straight-line problems: skip the last house, or skip the first house. Take the better.
**Approach 1 - Brute force:** Try every subset with no two neighbours (circular). Exponential.
**Approach 2 - Better:** Not needed.
**Approach 3 - Optimal:** `max(rob(nums[0..n-2]), rob(nums[1..n-1]))`. For `[2,3,2]`: `max(rob([2,3]), rob([3,2])) = 3`.

```js
function rob(nums) {                           // linear version from problem 1
  let prev = 0, cur = 0;
  for (const x of nums) [prev, cur] = [cur, Math.max(cur, prev + x)];
  return cur;
}
function robII(nums) {
  if (nums.length === 1) return nums[0];       // slices would both be empty
  return Math.max(rob(nums.slice(1)), rob(nums.slice(0, -1)));
}
```

**Complexity:** Time O(n), Space O(n) for the slices (O(1) if you pass index bounds instead).
**Watch out:** The single-house case. Without the guard you return 0.

### 8. Word Break — Medium
**Companies:** Amazon, Meta, Google, Microsoft, Apple
**Problem:** Can string `s` be split into words from a dictionary (words reusable)? Example: `"leetcode", ["leet","code"]` -> `true`.
**Intuition:** `dp[i]` = can the first i characters be split. It is true if some earlier cut `j` has `dp[j]` true and `s[j..i)` is a word.
**Approach 1 - Brute force:** Recursion trying every prefix. Exponential on inputs like `"aaaa...b"`.
**Approach 2 - Better:** Memoised recursion on the start index.
**Approach 3 - Optimal:** Bottom-up with a Set for O(1) word lookup.

Trace `"leetcode"`: `dp[0]=T`. `dp[4]=T` (`leet`). `dp[8]=T` because `dp[4]` is true and `code` is a word.

```js
function wordBreak(s, wordDict) {
  const set = new Set(wordDict);
  const dp = new Array(s.length + 1).fill(false);
  dp[0] = true;                                // empty prefix is splittable
  for (let i = 1; i <= s.length; i++)
    for (let j = 0; j < i; j++)
      if (dp[j] && set.has(s.slice(j, i))) { dp[i] = true; break; }
  return dp[s.length];
}
```

**Complexity:** Time O(n^2 * L) where slicing and hashing a substring costs up to O(L) (L is the substring length, at most n); often quoted as O(n^3) worst case. Space O(n + total dictionary size).
**Watch out:** `"catsandog"` with `["cats","dog","sand","and","cat"]` is `false`: greedy matching `cats` first leads to a dead end, which is why we check every cut. You can speed up by only trying `j` where `i - j <= maxWordLength`.

### 9. Maximum Product Subarray — Medium
**Companies:** Amazon, Google, Microsoft, LinkedIn
**Problem:** Find the contiguous subarray with the largest product. Example: `[2,3,-2,4]` -> `6`.
**Intuition:** Like Kadane, but a big negative number times another negative becomes a big positive. So track both the largest and the smallest product ending here; the smallest may become the largest next step.
**Approach 1 - Brute force:** Every subarray. Time O(n^2).
**Approach 2 - Better:** Prefix and suffix products (reset after a zero). Time O(n).
**Approach 3 - Optimal:** Keep `mx` and `mn` ending at i: `mx = max(x, mx*x, mn*x)`, `mn = min(x, mx*x, mn*x)` (using the old values).

| x | 2 | 3 | -2 | 4 |
|---|---|---|---|---|
| mx | 2 | 6 | -2 | 4 |
| mn | 2 | 3 | -12 | -48 |
| best | 2 | 6 | 6 | 6 |

```js
function maxProduct(nums) {
  let mx = nums[0], mn = nums[0], best = nums[0];
  for (let i = 1; i < nums.length; i++) {
    const x = nums[i], a = mx * x, b = mn * x;  // compute both before overwriting
    mx = Math.max(x, a, b);
    mn = Math.min(x, a, b);
    best = Math.max(best, mx);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Overwriting `mx` before using it for `mn` (that is why `a` and `b` are saved first). A zero resets both through the `x` option. `[-2,0,-1]` gives `0`.

### 10. Palindromic Substrings — Medium
**Companies:** Meta, Amazon, Google, Microsoft
**Problem:** Count all substrings (by position) that are palindromes. `"aaa"` -> `6`.
**Intuition:** `dp[i][j]` = is `s[i..j]` a palindrome. It is true when `s[i] === s[j]` and the inside `s[i+1..j-1]` is a palindrome (or the inside is empty or one char). This uses a 2D table because the state is a pair (start, end); it is the DP version of the problem.
**Approach 1 - Brute force:** Check every substring. Time O(n^3).
**Approach 2 - Better:** The table below, Time O(n^2), Space O(n^2).
**Approach 3 - Optimal:** Expand around each of the `2n-1` centres: Time O(n^2), Space O(1). Mention it in the interview; it is shorter on memory.

Fill order matters: `dp[i][j]` needs `dp[i+1][j-1]`, so loop `i` from the end down to 0.

```js
function countSubstrings(s) {
  const n = s.length;
  const dp = Array.from({ length: n }, () => new Array(n).fill(false));
  let count = 0;
  for (let i = n - 1; i >= 0; i--)
    for (let j = i; j < n; j++)
      if (s[i] === s[j] && (j - i < 2 || dp[i + 1][j - 1])) { dp[i][j] = true; count++; }
  return count;
}
```

**Complexity:** Time O(n^2), Space O(n^2).
**Watch out:** Length 1 and 2 substrings have no real "inside", hence the `j - i < 2` shortcut. Count by position: `"aaa"` has 6, not 3 distinct.

### 11. Longest Palindromic Substring — Medium
**Companies:** Amazon, Microsoft, Google, Meta, Apple
**Problem:** Return the longest palindromic substring. `"babad"` -> `"bab"` (or `"aba"`), `"cbbd"` -> `"bb"`.
**Intuition:** The same table as problem 10. Whenever a cell becomes true, compare its length with the best seen and remember the start.
**Approach 1 - Brute force:** Check every substring. Time O(n^3).
**Approach 2 - Better:** The DP table, Time O(n^2), Space O(n^2).
**Approach 3 - Optimal:** Expand around centres in O(n^2) time and O(1) space; Manacher's algorithm reaches O(n) (see the string algorithms chapter).

```js
function longestPalindrome(s) {
  const n = s.length;
  const dp = Array.from({ length: n }, () => new Array(n).fill(false));
  let start = 0, len = 1;
  for (let i = n - 1; i >= 0; i--)
    for (let j = i; j < n; j++)
      if (s[i] === s[j] && (j - i < 2 || dp[i + 1][j - 1])) {
        dp[i][j] = true;
        if (j - i + 1 > len) { len = j - i + 1; start = i; }
      }
  return s.slice(start, start + len);
}
```

**Complexity:** Time O(n^2), Space O(n^2).
**Watch out:** Both odd (`"aba"`) and even (`"abba"`) palindromes; the table handles both. Several answers can be valid, so tests must accept any longest one.

### 12. Integer Break — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Split n (n >= 2) into at least two positive integers; return the maximum product of the parts. `10` -> `36` (`3*3*4`).
**Intuition:** Cut off a first piece `j`. The rest `i-j` is either left whole (`j*(i-j)`) or broken further (`j * dp[i-j]`). Take the max over all `j`.
**Approach 1 - Brute force:** Try all compositions. Exponential.
**Approach 2 - Better:** Memoised recursion.
**Approach 3 - Optimal:** Bottom-up DP O(n^2). A math shortcut also exists: use as many 3s as possible (with a 4 or a 2 at the end). The DP is safer to explain.

```js
function integerBreak(n) {
  const dp = new Array(n + 1).fill(0);
  dp[1] = 1;
  for (let i = 2; i <= n; i++)
    for (let j = 1; j < i; j++)
      dp[i] = Math.max(dp[i], j * (i - j), j * dp[i - j]);
  return dp[n];
}
```

**Complexity:** Time O(n^2), Space O(n).
**Watch out:** `dp[i]` is the best product of breaking i into two or more parts, so `dp[1] = 1` is only a helper value (1 cannot be split). The `j*(i-j)` term covers "do not break the rest further"; the `j*dp[i-j]` term covers "break it further".

### 13. Ugly Number II — Medium
**Companies:** Amazon, Google, Microsoft, Apple
**Problem:** Ugly numbers only have prime factors 2, 3, 5 (1 counts). Return the nth one. `n = 10` -> `12`.
**Intuition:** Every ugly number is an earlier ugly number times 2, 3 or 5. Keep three pointers; each points to the next ugly number still waiting to be multiplied by its factor. The smallest of the three products is the next ugly number.
**Approach 1 - Brute force:** Test 1, 2, 3, ... for "ugly" by dividing. Slow for big n.
**Approach 2 - Better:** Min-heap plus a Set to remove duplicates. Time O(n log n).
**Approach 3 - Optimal:** Three pointers.

Trace: list `1`; candidates `2,3,5` -> add 2 (pointer for 2 moves); candidates `4,3,5` -> add 3; `4,6,5` -> add 4; `6,6,5` -> add 5; `6,6,10` -> add 6 and move **both** the 2 and 3 pointers.

```js
function nthUglyNumber(n) {
  const u = [1];
  let i2 = 0, i3 = 0, i5 = 0;
  while (u.length < n) {
    const a = u[i2] * 2, b = u[i3] * 3, c = u[i5] * 5;
    const m = Math.min(a, b, c);
    u.push(m);
    if (m === a) i2++;                          // not else-if: ties must advance all
    if (m === b) i3++;
    if (m === c) i5++;
  }
  return u[n - 1];
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Use three separate `if`s, not `else if`, or duplicates like 6 appear twice.

### 14. Delete and Earn — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Picking value `v` earns `v` points and deletes every `v-1` and `v+1` from the array. Maximise points. `[2,2,3,3,3,4]` -> `9`.
**Intuition:** Group equal values: `sum[v] = v * count(v)`. Picking `v` means you cannot pick `v-1` or `v+1`. That is House Robber on the array `sum[]`.
**Approach 1 - Brute force:** Try every order of picks. Exponential.
**Approach 2 - Better:** Not needed.
**Approach 3 - Optimal:** Bucket sums, then House Robber over values `1..max`. Here `sum = [_,0,4,9,4]`: results `0,4,9,9`, so `9`.

```js
function deleteAndEarn(nums) {
  const max = Math.max(...nums);
  const sum = new Array(max + 1).fill(0);
  for (const x of nums) sum[x] += x;
  let prev = 0, cur = 0;
  for (let v = 1; v <= max; v++) [prev, cur] = [cur, Math.max(cur, prev + sum[v])];
  return cur;
}
```

**Complexity:** Time O(n + max), Space O(max).
**Watch out:** If values are huge and sparse (like `10^9`), sort the distinct values instead of using a bucket array. `Math.max(...nums)` can overflow the stack on very large arrays; use a loop then.

### 15. Paint Fence — Medium
**Companies:** Google, Amazon, Meta
**Problem:** Paint `n` posts with `k` colours so that no three consecutive posts share a colour. Count the ways. `n=3, k=2` -> `6`.
**Intuition:** Track two counts for the last post: `same` (same colour as the one before) and `diff` (different). A new post can be `diff` after anything, in `k-1` ways; it can be `same` only if the last pair was `diff`.
**Approach 1 - Brute force:** Generate every colouring. Time O(k^n).
**Approach 2 - Better:** `total[i] = (k-1) * (total[i-1] + total[i-2])`. O(n) with an array.
**Approach 3 - Optimal:** Two variables.

Trace `n=3, k=2`: post 2: `same=2, diff=2`. Post 3: `newSame = diff = 2`, `newDiff = (2+2)*1 = 4`. Total `6`.

```js
function numWays(n, k) {
  if (n === 1) return k;
  let same = k, diff = k * (k - 1);             // state after post 2
  for (let i = 3; i <= n; i++) {
    const ns = diff, nd = (same + diff) * (k - 1);
    same = ns; diff = nd;
  }
  return same + diff;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** `n = 1` returns `k`. With `k = 1` and `n >= 3` the answer is 0 (it falls out of the formula).

### 16. Best Sightseeing Pair — Medium
**Companies:** Google, Amazon, Meta
**Problem:** Score of a pair `i < j` is `values[i] + values[j] + i - j`. Find the maximum score. `[8,1,5,2,6]` -> `11` (`i=0, j=2`).
**Intuition:** Split the formula: `(values[i] + i) + (values[j] - j)`. For each j, you want the best `values[i] + i` seen so far. That "best so far" is a one-variable DP.
**Approach 1 - Brute force:** All pairs. Time O(n^2).
**Approach 2 - Better:** Prefix array of `values[i] + i`. O(n) space.
**Approach 3 - Optimal:** One running variable `bestLeft`.

| j | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| bestLeft before j | 8 | 8 | 8 | 8 |
| score | 8 | 11 | 7 | 10 |

```js
function maxScoreSightseeing(values) {
  let bestLeft = values[0], best = -Infinity;   // bestLeft = max(values[i] + i)
  for (let j = 1; j < values.length; j++) {
    best = Math.max(best, bestLeft + values[j] - j);
    bestLeft = Math.max(bestLeft, values[j] + j);   // update after scoring j
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Update `bestLeft` **after** using it, so `i` is always less than `j`.

### 17. Maximum Sum Circular Subarray — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** The array is circular, so a subarray may wrap around the end. Return the maximum sum. `[5,-3,5]` -> `10` (`5 + 5` wrapping).
**Intuition:** The best subarray is either a normal one (Kadane max), or a wrapping one. A wrapping one is "everything minus a middle chunk", and to maximise it you remove the **minimum** subarray. So the answer is `max(maxKadane, total - minKadane)`.
**Approach 1 - Brute force:** Try every start and length over the doubled array. Time O(n^2).
**Approach 2 - Better:** Prefix sums with a monotonic deque over the doubled array. Time O(n).
**Approach 3 - Optimal:** Run max-Kadane and min-Kadane together.

Trace `[5,-3,5]`: `maxKadane = 7`, `total = 7`, `minKadane = -3`, wrap = `7 - (-3) = 10`. Answer `10`.

```js
function maxSubarraySumCircular(nums) {
  let total = 0, curMax = 0, curMin = 0, mx = -Infinity, mn = Infinity;
  for (const x of nums) {
    total += x;
    curMax = Math.max(x, curMax + x); mx = Math.max(mx, curMax);
    curMin = Math.min(x, curMin + x); mn = Math.min(mn, curMin);
  }
  return mx < 0 ? mx : Math.max(mx, total - mn);   // all negative: wrap would be empty
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** If every number is negative, `total - mn` is 0 (an empty subarray, not allowed). The `mx < 0` guard handles it: `[-3,-2,-3]` -> `-2`.

### 18. Minimum Cost For Tickets — Medium
**Companies:** Google, Amazon, Uber
**Problem:** You travel on given days of the year. Tickets cover 1, 7 or 30 days and have costs `costs[0..2]`. Return the minimum total cost. `days=[1,4,6,7,8,20], costs=[2,7,15]` -> `11`.
**Intuition:** `dp[d]` = min cost to cover all travel days up to day d. If you do not travel on d, it equals `dp[d-1]`. If you do, the last ticket you buy ends on d and covers d, d-6 or d-29 back.
**Approach 1 - Brute force:** Recursion: at each travel day pick one of three tickets. Exponential.
**Approach 2 - Better:** Memo on the travel-day index.
**Approach 3 - Optimal:** Day-by-day table up to the last day.

Trace (partial): `dp[1]=2`, `dp[4]=4`, `dp[6]=6`, `dp[7]=min(8, dp[0]+7)=7`, `dp[8]=9`, `dp[20]=min(11, 16, 15)=11`.

```js
function mincostTickets(days, costs) {
  const travel = new Set(days), last = days[days.length - 1];
  const dp = new Array(last + 1).fill(0);
  for (let d = 1; d <= last; d++) {
    if (!travel.has(d)) { dp[d] = dp[d - 1]; continue; }
    dp[d] = Math.min(
      dp[d - 1] + costs[0],
      dp[Math.max(0, d - 7)] + costs[1],         // clamp: day numbers start at 1
      dp[Math.max(0, d - 30)] + costs[2]);
  }
  return dp[last];
}
```

**Complexity:** Time O(last day), Space O(last day).
**Watch out:** Forgetting to clamp `d - 7` below 0. A 7-day pass bought on day d covers `d..d+6`, so looking back it covers `d-6..d`, and the previous state is `dp[d-7]`.

### 19. Coin Change II — Medium
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** Count the **combinations** of coins (unlimited supply) that sum to the amount. `5, [1,2,5]` -> `4`.
**Intuition:** Order must not matter: `2+1+1+1` and `1+2+1+1` are the same. The trick is the loop order. Put the **coins on the outer loop**: you finish using coin 1 before considering coin 2, so each combination is counted in one fixed order only.
**Approach 1 - Brute force:** Recursion over coins and remaining amount, exponential (memoised it becomes O(amount * coins) with a 2D table).
**Approach 2 - Better:** The 2D table `dp[coinIndex][amount]`.
**Approach 3 - Optimal:** Collapse to 1D with `dp[a] += dp[a - c]`.

Trace (amount 5): start `[1,0,0,0,0,0]`. After coin 1: `[1,1,1,1,1,1]`. After coin 2: `[1,1,2,2,3,3]`. After coin 5: `dp[5] = 3 + dp[0] = 4`.

```js
function change(amount, coins) {
  const dp = new Array(amount + 1).fill(0);
  dp[0] = 1;                                    // one way to make 0: take nothing
  for (const c of coins)                        // coins OUTER = combinations
    for (let a = c; a <= amount; a++) dp[a] += dp[a - c];
  return dp[amount];
}
```

**Complexity:** Time O(amount * n), Space O(amount).
**Watch out:** Swapping the loops (amount outer, coins inner) counts **permutations** (ordered sequences), which is a different problem (Combination Sum IV). Also `change(0, [7])` is `1`.

### 20. Jump Game II — Medium
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** You can always reach the end. Return the minimum number of jumps. `[2,3,1,1,4]` -> `2`.
**Intuition:** `dp[j]` = fewest jumps to reach j. From index i you can reach `i+1..i+nums[i]`, each with `dp[i] + 1` jumps.
**Approach 1 - Brute force:** Recursion over all jump sizes. Exponential.
**Approach 2 - Better:** The DP below, Time O(n^2), Space O(n).
**Approach 3 - Optimal:** Greedy BFS-by-levels in O(n): track the current jump's end and the farthest reach, and count a jump when you pass the end. Greedy is covered in its own chapter; the DP is the safe first answer.

Trace `[2,3,1,1,4]`: i=0 sets `dp[1]=dp[2]=1`. i=1 sets `dp[3]=dp[4]=2`. Answer `dp[4] = 2`.

```js
function jump(nums) {
  const n = nums.length, dp = new Array(n).fill(Infinity);
  dp[0] = 0;
  for (let i = 0; i < n; i++)
    for (let j = i + 1; j <= Math.min(n - 1, i + nums[i]); j++)
      dp[j] = Math.min(dp[j], dp[i] + 1);
  return dp[n - 1];
}
```

**Complexity:** Time O(n^2), Space O(n).
**Watch out:** `[0]` (one element) needs 0 jumps. The O(n^2) version may time out on the largest LeetCode inputs; mention the greedy upgrade.

### 21. Domino and Tromino Tiling — Medium
**Companies:** Google, Amazon, Meta
**Problem:** Count the ways to tile a `2 x n` board with 2x1 dominoes and L-shaped trominoes (rotations allowed), modulo `10^9 + 7`. `n=3` -> `5`.
**Intuition:** Work out the ending of the board. A vertical domino leaves `n-1`; two horizontal dominoes leave `n-2`. Trominoes force longer "interlocking" shapes, and each longer shape can be built in exactly 2 mirror ways, which gives the clean recurrence `dp[n] = 2*dp[n-1] + dp[n-3]`.
**Approach 1 - Brute force:** Recursion on a profile (which cells of the last column are filled). Works but is harder to code.
**Approach 2 - Better:** Two-state DP: full column vs. a column with one extra cell sticking out.
**Approach 3 - Optimal:** The closed recurrence below with `dp[0]=1, dp[1]=1, dp[2]=2`.

Trace: `dp[3] = 2*2 + 1 = 5`, `dp[4] = 2*5 + 1 = 11`.

```js
function numTilings(n) {
  const M = 1000000007;
  const dp = [1, 1, 2];
  for (let i = 3; i <= n; i++) dp[i] = (2 * dp[i - 1] + dp[i - 3]) % M;
  return dp[n];
}
```

**Complexity:** Time O(n), Space O(n) (O(1) with three rolling variables).
**Watch out:** Apply `% M` every step. `2 * dp[i-1]` is below `2.1 * 10^9`, which is safe in a JS double. This recurrence is hard to derive cold; if you forget it, derive the two-state version instead.

## Common Mistakes
- **"Greedy works for Coin Change."** It does not for general coins (`[1,3,4]`, amount 6). Use DP.
- **"Start Kadane with 0."** All-negative arrays then return 0. Start from `nums[0]`.
- **"Loop order does not matter in Coin Change II."** Coins outer counts combinations; amount outer counts permutations.
- **"Skip the base case, the loop will handle it."** `dp[0]` must be set (0 for min problems, 1 for counting).
- **"`dp[i]` is just 'the answer'."** Define it in words first, for example "best sum of a subarray ending **at** i". Kadane's `dp[i]` is not the final answer; the max over all `dp[i]` is.
- **"Rolling variables update in any order."** Save old values before overwriting (Max Product, Paint Fence).
- **"Zeros are harmless in Decode Ways."** A `0` can only be the second digit of `10` or `20`.

## Interview Tips
- Say the state in one sentence before any code: "`dp[i]` is the minimum number of coins to make amount i."
- Say the recurrence, the base case and the answer cell. Then the complexity, then code.
- Write the array version first, then say "this only reads the last two cells, so I can use two variables".
- Follow-ups you will hear: "Can you do it in O(1) space?", "Return the actual coins / subarray, not just the count" (store a parent pointer or the start index), "What if the coins are huge and the amount is small / vice versa?"
- Edge cases to name: empty input, one element, all negatives, zeros, amount 0, impossible targets (`-1`), modulo overflow.
- If you see the exponential recursion first, say so: "this repeats subproblems, so I will memoise" is exactly the signal interviewers want.

## Practice List
**Easy:** Climbing Stairs, Min Cost Climbing Stairs, N-th Tribonacci Number, Fibonacci Number, Counting Bits, Is Subsequence, Divisor Game
**Medium:** Combination Sum IV, Longest Increasing Subsequence, Maximum Length of Repeated Subarray, Unique Binary Search Trees, Count Sorted Vowel Strings, Number of Ways to Stay in the Same Place After Some Steps, Arithmetic Slices, Knight Dialer, Maximum Alternating Subsequence Sum, Longest Turbulent Subarray, Solving Questions With Brainpower, Count Ways To Build Good Strings
**Hard:** Decode Ways II, Word Break II, Maximum Sum of 3 Non-Overlapping Subarrays, Jump Game IV, Number of Ways to Form a Target String Given a Dictionary

## Key Takeaways
- **State first:** write what `dp[i]` means in one sentence before any code.
- **Three shapes:** take-or-skip, try every last step, best-ending-here (Kadane).
- **Base cases decide correctness:** `dp[0] = 0` for minimums, `dp[0] = 1` for counting.
- **Loop order encodes the question:** coins outer = combinations, amount outer = permutations.
- **Roll the array** into one or two variables when `dp[i]` only needs the last 1 to 3 cells.
- **Reductions pay off:** Delete and Earn is House Robber, Perfect Squares is Coin Change, Circular Kadane is two Kadanes.
- **Greedy is not DP in disguise:** greedy fails on Coin Change; use it only when you can prove it (Jump Game).

---
[← Previous: DP Intuition](../01-dp-intuition/README.md) | [Next: Grid DP →](../03-grid-dp/README.md) | [Back to Dynamic Programming Index](../README.md)
