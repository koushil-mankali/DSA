# Bitmask DP

> **Estimated Time:** 4 hr | **Difficulty:** Advanced
> **Prerequisites:** [01 - DP Intuition](../01-dp-intuition/README.md), [04 - Subset and Subsequence DP](../04-subset-and-subsequence-dp/README.md), [09 - Partition and MCM DP](../09-partition-and-mcm-dp/README.md)

---

## What You'll Learn

- You can store a set of up to about 20 items in one integer and use that integer as a DP state.
- You can use the bit tricks (set, clear, test, lowest bit, popcount) without looking them up.
- You can loop over all submasks of a mask, and explain why the total work is 3^n and not 4^n.
- You can solve "assign / partition / visit everything" problems with `dp[mask]` and `dp[mask][last]`.
- You can spot from the constraints (`n <= 20`) that bitmask DP is the intended solution.

## The Big Idea

Think of a row of light switches on a wall. Each switch is on or off. If you write down which switches are on, you have described the whole room. A "set of used items" works the same way: item `i` is either used or not used. One integer can hold all those on/off answers, one per bit.

Many hard problems say: "pick an order", "split into groups", or "visit every node". Brute force tries every order or every split, which is `n!` or worse. But often the future only depends on WHICH items are already used, not in what order you used them. Two different orders that used the same set of items lead to the same future. So we store one answer per set. There are only `2^n` sets. For `n = 20` that is about one million, which is fine. `20!` is far too big.

Interviewers love this because it tests three things at once: do you read constraints, do you know bit operations, and can you find a state that forgets the order.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| `n <= 20` (or 16, 18, 15) | State is a subset of `n` items: `dp[mask]` |
| "every item is used exactly once", "visit all nodes" | `mask` = used / visited items |
| "split into `k` groups of equal sum" | `dp[mask]` = progress of the group being filled |
| "assign each person to one task" | `dp[mask]` = tasks taken so far, person = popcount(mask) |
| "shortest tour that visits all" | `dp[mask][last]` (TSP shape) |
| "minimum number of sets that cover all skills" | `mask` = skills covered, `dp[mask]` = fewest people |
| 40 hats but only 10 people | Mask over the SMALL side (people), loop over the big side |
| Game where each number can be picked once | `mask` = numbers already picked |
| Constraint `n <= 20` AND answer needs an order | Bitmask beats permutations: `2^n * n` versus `n!` |

Rule of thumb: `2^n * n` is fine up to `n = 20` (about 20 million steps). `3^n` (submask loops) is fine up to `n = 15` or `16` (3^15 is about 14 million).

## Core Concepts / Template

**Bit tricks you must know**

| Goal | Code | Note |
|---|---|---|
| Is item `i` in the set? | `mask & (1 << i)` | non-zero means yes |
| Add item `i` | `mask \| (1 << i)` | |
| Remove item `i` | `mask & ~(1 << i)` or `mask ^ (1 << i)` | `^` only safe if `i` is in the set |
| Full set of `n` items | `(1 << n) - 1` | all `n` low bits are 1 |
| Lowest set bit | `mask & -mask` | e.g. `0b1100 -> 0b0100` |
| Remove lowest set bit | `mask & (mask - 1)` | used to count bits |
| Count set bits (popcount) | loop `x &= x - 1` | each step kills one bit |
| Index of lowest bit | `31 - Math.clz32(mask & -mask)` | JS has no built-in ctz |

JavaScript note: bit operators work on 32-bit signed integers. For `n <= 20` that is always safe. Do not use `1 << 31` or higher as a normal bit.

**Template 1: dp over subsets (build up by adding one item)**

```js
function solve(n) {
  const dp = new Array(1 << n).fill(INF);   // INF = "impossible"
  dp[0] = base;                             // empty set
  for (let mask = 0; mask < (1 << n); mask++) {
    if (dp[mask] === INF) continue;
    for (let i = 0; i < n; i++) {
      if (mask & (1 << i)) continue;        // already used
      const next = mask | (1 << i);
      dp[next] = better(dp[next], cost(dp[mask], i));
    }
  }
  return dp[(1 << n) - 1];
}
```

Why this order works: `next` is always bigger than `mask` as a number, so when we reach `mask` all smaller masks that can lead to it are already done. Cost: `O(2^n * n)`.

**Template 2: dp[mask][last] (order matters at the end, like TSP)**

```js
// dp[mask][last] = best cost having visited `mask`, standing at `last`
for (let mask = 1; mask < (1 << n); mask++)
  for (let last = 0; last < n; last++) {
    if (!(mask & (1 << last))) continue;
    for (let nxt = 0; nxt < n; nxt++) {
      if (mask & (1 << nxt)) continue;
      // relax dp[mask | 1 << nxt][nxt] using dp[mask][last]
    }
  }
```

Cost: `O(2^n * n^2)` time, `O(2^n * n)` space.

**Template 3: iterate over all submasks**

```js
for (let sub = mask; sub > 0; sub = (sub - 1) & mask) {
  // sub is a non-empty subset of mask; mask ^ sub is the rest
}
// to include the empty subset, handle sub = 0 separately
```

Why `(sub - 1) & mask` works: subtracting 1 flips the lowest set bit to 0 and turns all the zeros below it into 1s. The `& mask` throws away the bits that are not in `mask`. The result is the next smaller submask. Short trace for `mask = 0b101`:

```
sub = 101  -> next = (101 - 1) & 101 = 100 & 101 = 100
sub = 100  -> next = (100 - 1) & 101 = 011 & 101 = 001
sub = 001  -> next = (001 - 1) & 101 = 000 & 101 = 000  (stop)
```

Total work over all masks: each item is in one of three states for a pair (mask, sub): not in mask, in mask but not in sub, in both. So the pairs number `3^n`.

**Precompute subset sums fast**

`sum[mask] = sum[mask without lowest bit] + value[lowest bit]` gives all `2^n` sums in `O(2^n)` instead of `O(2^n * n)`. You will see this in the cookies problem.

---

## Problems (easy -> hard)

### 1. Matchsticks to Square — Medium
**Companies:** Amazon, Google
**Problem:** Given matchstick lengths, use every stick exactly once to form a square. Return true if possible. Example: `[1,1,2,2,2] -> true` (sides 2, 2, 2, 1+1).
**Intuition:** Fill the sides one after another. If we remember which sticks are used, the current side's length is `sum(used) % side`. So the mask alone is enough state.
**Approach 1 - Brute force:** Try putting each stick on each of 4 sides with backtracking. Time O(4^n), Space O(n).
**Approach 2 - Better:** Sort descending and prune. Still exponential in the worst case, but fast in practice.
**Approach 3 - Optimal:** `dp[mask]` = length filled in the current side after using the sticks in `mask`, or `-1` if impossible. Adding a stick that overflows the side is not allowed. When a side hits exactly `side` we wrap to 0 with `% side`. Answer: `dp[full] === 0`.

Trace for `[1,1,2,2,2]`, `side = 2`:

| mask (sticks used) | dp |
|---|---|
| none | 0 |
| {1} | 1 |
| {1,1} | 0 (side done) |
| {1,1,2} | 0 (wraps from 2) |
| {1,1,2,2} | 0 |
| all five | 0 -> true |

```js
// 1. Matchsticks to Square
function makesquare(matchsticks) {
  const n = matchsticks.length;
  const total = matchsticks.reduce((a, b) => a + b, 0);
  if (n < 4 || total % 4 !== 0) return false;
  const side = total / 4;
  if (Math.max(...matchsticks) > side) return false;
  const full = (1 << n) - 1;
  const dp = new Array(1 << n).fill(-1); // dp[mask] = length of the unfinished side, -1 = unreachable
  dp[0] = 0;
  for (let mask = 0; mask <= full; mask++) {
    if (dp[mask] < 0) continue;
    for (let i = 0; i < n; i++) {
      if (mask & (1 << i)) continue;
      const cur = dp[mask] + matchsticks[i];
      if (cur > side) continue;
      dp[mask | (1 << i)] = cur % side;
    }
  }
  return dp[full] === 0;
}
```

**Complexity:** Time O(2^n * n), Space O(2^n).
**Watch out:** Check `total % 4`, `n < 4`, and the longest stick `> side` first. Use `% side`, not `=== side ? 0 : ...` with a missing case, so a full side resets cleanly.

### 2. Fair Distribution of Cookies — Medium
**Companies:** Amazon, Google
**Problem:** Give `n` bags of cookies to `k` kids, each bag whole to one kid. Minimize the largest total any one kid gets. Example: `[8,15,10,20,8], k=2 -> 31`.
**Intuition:** Treat kid by kid. After the first `j` kids, the bags they took form a set. The next kid takes a submask of the remaining bags. We want the smallest "worst kid" so far.
**Approach 1 - Brute force:** Assign each bag to one of `k` kids, `k^n` ways. Grows far too fast.
**Approach 2 - Better:** Backtracking with pruning (skip a kid whose load is already `>= best`, and skip equal empty kids).
**Approach 3 - Optimal:** `dp[j][mask]` = smallest possible maximum load if `j` kids share exactly the bags in `mask`. Transition: the last kid takes `sub`, the rest `mask ^ sub` go to the first `j-1` kids: `max(prev[mask ^ sub], sum[sub])`. We only need the previous layer, so keep one array.

Trace for `[8,15,10,20,8], k = 2`: layer 1 is `sum[mask]`. For `mask = all`, we try each `sub`. Taking `sub = {15,8}` (23) leaves `{8,10,20}` (38): max is 38. Taking `sub = {8,15,8}` (31) leaves `{10,20}` (30): max is 31. That is the best.

```js
// 2. Fair Distribution of Cookies
function distributeCookies(cookies, k) {
  const n = cookies.length;
  const size = 1 << n;
  const sum = new Array(size).fill(0);
  for (let m = 1; m < size; m++) {
    const low = m & -m;
    sum[m] = sum[m ^ low] + cookies[31 - Math.clz32(low)];
  }
  // dp[j][mask] = best (smallest) max load when first j kids share the bags in mask
  let prev = sum.slice(); // one kid takes everything in mask
  for (let j = 2; j <= k; j++) {
    const cur = new Array(size).fill(Infinity);
    for (let mask = 0; mask < size; mask++) {
      for (let sub = mask; ; sub = (sub - 1) & mask) {
        cur[mask] = Math.min(cur[mask], Math.max(prev[mask ^ sub], sum[sub]));
        if (sub === 0) break;
      }
    }
    prev = cur;
  }
  return prev[size - 1];
}
```

**Complexity:** Time O(k * 3^n), Space O(2^n).
**Watch out:** The submask loop must include `sub = 0` (a kid may get nothing in a sub-layer), which is why the loop uses `if (sub === 0) break` at the end. Use `Infinity` for unreached cells.

### 3. Partition to K Equal Sum Subsets — Medium
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** Can the array be split into `k` non-empty groups with equal sums? Example: `[4,3,2,3,5,2,1], k=4 -> true` (groups `[5]`, `[1,4]`, `[2,3]`, `[2,3]`).
**Intuition:** Same trick as matchsticks. Compute `target = total / k`. Fill groups one after another. After using the numbers in `mask`, the current group has `sum(mask) % target`.
**Approach 1 - Brute force:** For every number choose one of `k` groups. Time O(k^n).
**Approach 2 - Better:** Backtracking: fill one group to `target` at a time, with sorting and skipping duplicates. Fast on most tests, exponential worst case.
**Approach 3 - Optimal:** `dp[mask]` = current group sum, `-1` if the order is impossible. Adding `nums[i]` is allowed only if the group does not exceed `target`. Answer: `dp[full] === 0`.

Trace for `[1,2,3,4]`, `k = 2`, `target = 5`: `{1}=1`, `{1,2}=3`, `{1,2,3}` is 6 > 5 so blocked, but `{1,4}=5 -> 0`, then `{1,4,2}=2`, `{1,4,2,3}=5 -> 0`. True.

```js
// 3. Partition to K Equal Sum Subsets
function canPartitionKSubsets(nums, k) {
  const n = nums.length;
  const total = nums.reduce((a, b) => a + b, 0);
  if (total % k !== 0) return false;
  const target = total / k;
  if (Math.max(...nums) > target) return false;
  const dp = new Array(1 << n).fill(-1);
  dp[0] = 0;
  for (let mask = 0; mask < (1 << n); mask++) {
    if (dp[mask] < 0) continue;
    for (let i = 0; i < n; i++) {
      if (mask & (1 << i)) continue;
      if (dp[mask] + nums[i] > target) continue;
      dp[mask | (1 << i)] = (dp[mask] + nums[i]) % target;
    }
  }
  return dp[(1 << n) - 1] === 0;
}
```

**Complexity:** Time O(2^n * n), Space O(2^n).
**Watch out:** `total % k !== 0` and `max > target` must return false early. With `k = 1` the answer is true (target equals total).

### 4. Beautiful Arrangement — Medium
**Companies:** Amazon, Google
**Problem:** Count permutations of `1..n` where, at each position `i` (1-indexed), `perm[i] % i == 0` or `i % perm[i] == 0`. Example: `n=2 -> 2`, `n=3 -> 3`.
**Intuition:** Fill positions from 1 to `n`. Which position is next? It is `popcount(mask) + 1`. So the mask alone describes the state and we never store the position.
**Approach 1 - Brute force:** Generate all `n!` permutations and check. Time O(n! * n).
**Approach 2 - Better:** Backtracking that places a number only if it fits. Much faster than `n!` because most branches die early.
**Approach 3 - Optimal:** `dp[mask]` = number of ways to fill the first `popcount(mask)` positions with exactly the numbers in `mask`. Transition by adding any unused number that fits the next position.

```js
// 4. Beautiful Arrangement
function countArrangement(n) {
  const dp = new Array(1 << n).fill(0);
  dp[0] = 1;
  for (let mask = 0; mask < (1 << n); mask++) {
    if (dp[mask] === 0) continue;
    const pos = popcount(mask) + 1; // next position, 1-indexed
    for (let num = 1; num <= n; num++) {
      if (mask & (1 << (num - 1))) continue;
      if (num % pos === 0 || pos % num === 0) dp[mask | (1 << (num - 1))] += dp[mask];
    }
  }
  return dp[(1 << n) - 1];
}

function popcount(x) {
  let c = 0;
  while (x) { x &= x - 1; c++; }
  return c;
}
```

Check: `n=4 -> 8`, `n=15 -> 24679`.

**Complexity:** Time O(2^n * n), Space O(2^n).
**Watch out:** Numbers are 1-indexed, bits are 0-indexed: number `num` uses bit `num - 1`. Mixing these up is the classic bug.

### 5. Can I Win — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Two players take turns choosing an unused integer from `1..maxChoosableInteger`. The running total counts both players' picks. Whoever makes the total reach `desiredTotal` or more wins. Can the first player force a win? Example: `(10, 11) -> false`.
**Intuition:** The set of numbers already taken fixes the running total (the sum of the numbers in the mask). So the mask is a full game state. A state is winning if SOME move wins at once or leaves the opponent in a losing state.
**Approach 1 - Brute force:** Try every move recursively. Time O(n!).
**Approach 2 - Better:** Memoize on the mask. Many move orders reach the same mask.
**Approach 3 - Optimal:** Same as approach 2: memoize `win(mask)` in an array or Map. States: `2^n`. Quick checks first: `desiredTotal <= 0` is a win for the first player; if the sum of all numbers `n(n+1)/2 < desiredTotal`, nobody can reach it, so the first player cannot win.

Trace for `(4, 6)`: first player picks 3 (remaining 3). The second player can pick 4 and win at once, so 3 is bad. Picks 4 (remaining 2): opponent picks any of 1,2,3. Picking 2 or 3 wins at once, so 4 is bad. Picks 2 (remaining 4): opponent picks 4 and wins. Picks 1 (remaining 5): opponent needs 5 or more in one pick, impossible; opponent picks 2, 3 or 4, leaving 3, 2 or 1. If opponent picks 4, remaining 1, then we pick anything and win. If opponent picks 2 (remaining 3), we pick 3 or 4 and win. If opponent picks 3 (remaining 2), we pick 2 or 4 and win. So the answer is true.

```js
// 5. Can I Win
function canIWin(maxChoosableInteger, desiredTotal) {
  const n = maxChoosableInteger;
  if (desiredTotal <= 0) return true;
  if ((n * (n + 1)) / 2 < desiredTotal) return false;
  const memo = new Map(); // mask -> can the player to move win (remaining total is implied by mask)
  function win(mask, remain) {
    if (memo.has(mask)) return memo.get(mask);
    let res = false;
    for (let i = 1; i <= n && !res; i++) {
      if (mask & (1 << (i - 1))) continue;
      if (i >= remain || !win(mask | (1 << (i - 1)), remain - i)) res = true;
    }
    memo.set(mask, res);
    return res;
  }
  return win(0, desiredTotal);
}
```

**Complexity:** Time O(2^n * n), Space O(2^n).
**Watch out:** Do not forget the two early checks; without them `desiredTotal = 0` and unreachable totals give wrong answers. The remaining total is a function of the mask, so it needs no place in the memo key.

### 6. Minimum Number of Work Sessions to Finish the Tasks — Medium
**Companies:** Google, Amazon
**Problem:** Each task takes some hours. In one session you can work at most `sessionTime` hours and tasks cannot be split. Minimum number of sessions? Example: `tasks=[3,1,3,1,1], sessionTime=8 -> 2`.
**Intuition:** This is bin packing. The last session takes some subset of the remaining tasks that fits. So `dp[mask] = min over fitting sub of dp[mask ^ sub] + 1`.
**Approach 1 - Brute force:** Try every assignment of tasks to sessions. Time O(n^n).
**Approach 2 - Better:** Backtracking with a list of session loads, try to add each task to an existing session or open a new one, prune with the best so far.
**Approach 3 - Optimal:** Precompute `fits[sub]` (sum <= sessionTime), then run the submask DP.

Trace for `[1,2,3]`, `sessionTime = 3`: `fits` is true for `{1}, {2}, {3}, {1,2}`. `dp[{1,2}] = 1`, `dp[{3}] = 1`, `dp[all]`: take sub `{1,2}`, rest `{3}` costs 1, so 2.

```js
// 6. Minimum Number of Work Sessions
function minSessions(tasks, sessionTime) {
  const n = tasks.length;
  const size = 1 << n;
  const fits = new Array(size).fill(false);
  for (let m = 1; m < size; m++) {
    let s = 0;
    for (let i = 0; i < n; i++) if (m & (1 << i)) s += tasks[i];
    fits[m] = s <= sessionTime;
  }
  const dp = new Array(size).fill(Infinity);
  dp[0] = 0;
  for (let mask = 1; mask < size; mask++) {
    for (let sub = mask; sub > 0; sub = (sub - 1) & mask) {
      if (fits[sub]) dp[mask] = Math.min(dp[mask], dp[mask ^ sub] + 1);
    }
  }
  return dp[size - 1];
}
```

**Complexity:** Time O(3^n) (plus O(2^n * n) for `fits`), Space O(2^n).
**Watch out:** Loop `sub` from `mask` down to 1 (non-empty), otherwise `dp[mask]` could depend on itself. Each task must be at most `sessionTime` (the problem guarantees it).

### 7. Number of Ways to Wear Different Hats to Each Other — Hard
**Companies:** Google, Amazon
**Problem:** There are `n <= 10` people and 40 hat types. Each person has a list of hats they like. Count ways so that every person wears a different hat they like. Answer modulo `1e9+7`. Example: `[[3,4],[4,5],[5]] -> 1`.
**Intuition:** The usual idea is to give each person a hat, but then you need a mask over 40 hats, which is too big. Flip it: go through hats one by one and decide which person (if any) gets this hat. Now the mask is over the 10 people.
**Approach 1 - Brute force:** Each person picks any liked hat, skip bad duplicates. Time O(40^n).
**Approach 2 - Better:** Mask over hats: `2^40` states, too big.
**Approach 3 - Optimal:** `dp[mask]` = ways where the people in `mask` already wear a hat from hats `1..h`. For each hat `h`, for each mask, for each person `p` who likes `h` and is not in `mask`: `dp[mask | p] += dp[mask]`. Walk masks from high to low so one hat is not used twice.

Trace for `[[3,5,1],[3,5]]` (bit 0 = person 0, bit 1 = person 1; `dp` is "ways so far", hats may stay unused):
- Start: `dp[00] = 1`.
- Hat 1 (only person 0): `dp[01] = 1`.
- Hat 3 (both): `dp[11] = 1` (person 1 takes it, person 0 already has hat 1), `dp[01] = 2`, `dp[10] = 1`.
- Hat 5 (both): `dp[11] = 1 + dp[01] + dp[10] = 4`.

```js
// 7. Number of Ways to Wear Different Hats
function numberWays(hats) {
  const MOD = 1e9 + 7;
  const n = hats.length;
  const people = Array.from({ length: 41 }, () => []);
  hats.forEach((list, p) => list.forEach(h => people[h].push(p)));
  const full = (1 << n) - 1;
  const dp = new Array(1 << n).fill(0);
  dp[0] = 1;
  for (let h = 1; h <= 40; h++) {
    for (let mask = full; mask >= 0; mask--) { // downwards: each hat used at most once
      if (dp[mask] === 0) continue;
      for (const p of people[h]) {
        if (mask & (1 << p)) continue;
        const nm = mask | (1 << p);
        dp[nm] = (dp[nm] + dp[mask]) % MOD;
      }
    }
  }
  return dp[full];
}
```

**Complexity:** Time O(40 * 2^n * n), Space O(2^n).
**Watch out:** Iterating masks upward would let the same hat be given to two people in one pass. Iterate downward (this is the same reason 0/1 knapsack loops downward). Apply the modulus on every addition.

### 8. Count Assignments of Tasks to People — Medium
**Companies:** Google, Amazon
**Problem:** There are `n` people and `n` tasks. `can[p][t] = 1` if person `p` can do task `t`. Count the ways to give each person exactly one different task. Example: `[[1,1],[1,1]] -> 2`.
**Intuition:** Give tasks to people in order: person 0, then person 1, and so on. After `popcount(mask)` people are done, the mask holds the tasks taken. The next person is `popcount(mask)`.
**Approach 1 - Brute force:** Try all `n!` permutations and check. Time O(n! * n).
**Approach 2 - Better:** Backtracking with a `used` array. Same worst case, fewer dead branches.
**Approach 3 - Optimal:** `dp[mask]` = number of ways the first `popcount(mask)` people took exactly the tasks in `mask`. Add one task for the next person if allowed.

Trace for `[[1,1,0],[0,1,1],[1,0,1]]`: person 0 takes task 0 or 1. If task 0, person 1 takes task 1 (then person 2 takes task 2, ok) or task 2 (then person 2 needs task 1, not allowed). If person 0 takes task 1, person 1 must take task 2 and person 2 takes task 0. Two ways in total.

```js
// 8. Count assignments: each person gets exactly one distinct task they can do
function countAssignments(can) {
  const n = can.length;
  const dp = new Array(1 << n).fill(0); // dp[mask] = ways for first popcount(mask) people to take tasks in mask
  dp[0] = 1;
  for (let mask = 0; mask < (1 << n); mask++) {
    if (dp[mask] === 0) continue;
    const person = popcount(mask);
    if (person === n) continue;
    for (let t = 0; t < n; t++) {
      if (!(mask & (1 << t)) && can[person][t]) dp[mask | (1 << t)] += dp[mask];
    }
  }
  return dp[(1 << n) - 1];
}

function popcount(x) {
  let c = 0;
  while (x) { x &= x - 1; c++; }
  return c;
}
```

**Complexity:** Time O(2^n * n), Space O(2^n).
**Watch out:** Counts grow fast (`n!`). For large counts the problem will ask for a modulus; add `% MOD` at every addition, as in problem 7. This is also the "minimum cost assignment" skeleton: replace `+=` by `min` and add costs.

### 9. Shortest Path Visiting All Nodes — Hard
**Companies:** Google, Amazon
**Problem:** An undirected connected graph with `n <= 12` nodes. Find the length of the shortest walk that visits every node. You may start anywhere and revisit nodes and edges. Example: `[[1,2,3],[0],[0],[0]] -> 4`.
**Intuition:** A walk is described by (which nodes seen so far, where I stand). That is `(mask, node)`, only `2^n * n` states. Edges all cost 1, so BFS finds the shortest path.
**Approach 1 - Brute force:** DFS over all walks. Infinite without a bound; with a bound, exponential.
**Approach 2 - Better:** Floyd-Warshall for all pairs, then try all `n!` orders of nodes. Time O(n! * n).
**Approach 3 - Optimal:** Multi-source BFS. Start with every `(1 << i, i)` at distance 0. Expand to neighbours, OR in the neighbour bit. The first time the mask is full, return the step count. Mark `seen[mask][node]` so no state is visited twice.

Trace for the star `0-{1,2,3}`: start `(node 1)`, step 1: `(0, {1,0})`, step 2: `(2, {1,0,2})`, step 3: `(0, ...)`, step 4: `(3, full)`. Answer 4.

```js
// 9. Shortest Path Visiting All Nodes
function shortestPathLength(graph) {
  const n = graph.length;
  if (n === 1) return 0;
  const full = (1 << n) - 1;
  const seen = Array.from({ length: 1 << n }, () => new Array(n).fill(false));
  let queue = [];
  for (let i = 0; i < n; i++) { queue.push([i, 1 << i]); seen[1 << i][i] = true; }
  let steps = 0;
  while (queue.length) {
    const next = [];
    steps++;
    for (const [node, mask] of queue) {
      for (const nb of graph[node]) {
        const nm = mask | (1 << nb);
        if (nm === full) return steps;
        if (!seen[nm][nb]) { seen[nm][nb] = true; next.push([nb, nm]); }
      }
    }
    queue = next;
  }
  return -1;
}
```

**Complexity:** Time O(2^n * n * deg) which is at most O(2^n * n^2), Space O(2^n * n).
**Watch out:** Use BFS, not DFS, because edges are unweighted and nodes may be revisited. Return 0 for a single node. For weighted edges use Dijkstra or the TSP-style DP with Floyd-Warshall.

### 10. Smallest Sufficient Team — Hard
**Companies:** Google, Amazon
**Problem:** Given required skills (at most 16) and people with skill lists, return the indices of the smallest team that covers all skills. Example: skills `[java, nodejs, reactjs]`, people `[[java],[nodejs],[nodejs,reactjs]] -> [0,2]`.
**Intuition:** The mask is the set of skills covered. For each person, we ask: if I add this person to a team that covers `mask`, I now cover `mask | person`. We want the fewest people for each covered set.
**Approach 1 - Brute force:** Try every subset of people. With 60 people that is `2^60`, impossible.
**Approach 2 - Better:** Backtracking on the first uncovered skill: try each person who has it. Works but can be slow.
**Approach 3 - Optimal:** `best[mask]` = smallest list of people that covers exactly (at least) `mask`. For each person, update all masks. The state count is `2^skills` (at most 65536), not `2^people`.

Trace for the example (bits written java, nodejs, reactjs from left to right): `best[000] = []`. Person 0 (`java`): `best[100] = [0]`. Person 1 (`nodejs`): `best[010] = [1]`, `best[110] = [0,1]`. Person 2 (`nodejs, reactjs`): `best[011] = [2]`, `best[111] = [0,2]` (from `best[100]`, size 2 beats `[0,1,2]`).

```js
// 10. Smallest Sufficient Team
function smallestSufficientTeam(reqSkills, people) {
  const idx = new Map(reqSkills.map((s, i) => [s, i]));
  const k = reqSkills.length;
  const full = (1 << k) - 1;
  const best = new Array(1 << k).fill(null); // best[mask] = smallest list of people covering mask
  best[0] = [];
  const masks = people.map(p => p.reduce((m, s) => m | (1 << idx.get(s)), 0));
  for (let i = 0; i < people.length; i++) {
    if (masks[i] === 0) continue;
    for (let mask = 0; mask <= full; mask++) {
      if (best[mask] === null) continue;
      const nm = mask | masks[i];
      if (nm === mask) continue;
      if (best[nm] === null || best[nm].length > best[mask].length + 1) best[nm] = [...best[mask], i];
    }
  }
  return best[full];
}
```

**Complexity:** Time O(people * 2^skills), Space O(2^skills * team size).
**Watch out:** Mask over the SMALL side (skills), never people. Skip people with an empty mask. Copying arrays with `[...best[mask], i]` costs `O(team)`; to save memory store a parent pointer instead.

### 11. Travelling Salesman Problem (TSP) — Hard
**Companies:** Google, Amazon, Microsoft
**Problem:** Given a distance matrix for `n` cities, find the shortest route that starts at city 0, visits every other city exactly once, and returns to city 0. Example: the 4-city matrix below gives 80.
**Intuition:** The best way to finish a tour depends only on (which cities are visited, where I am now). The order that got me there does not matter. That is `dp[mask][last]`.
**Approach 1 - Brute force:** Try all `(n-1)!` orders. For `n = 15` that is 87 billion, too slow. Time O(n!).
**Approach 2 - Better:** Branch and bound, prune routes already longer than the best. Good heuristic, same worst case.
**Approach 3 - Optimal (Held-Karp):** `dp[mask][last]` = cheapest path that starts at 0, visits exactly `mask`, and ends at `last`. Extend by one unvisited city. At the end close the loop back to 0.

Trace on distances `0-1:10, 0-2:15, 0-3:20, 1-2:35, 1-3:25, 2-3:30`:

| state `(mask, last)` | cost |
|---|---|
| `({0}, 0)` | 0 |
| `({0,1}, 1)` | 10 |
| `({0,1,3}, 3)` | 10 + 25 = 35 |
| `({0,1,3,2}, 2)` | 35 + 30 = 65 |
| close: back to 0 | 65 + 15 = 80 |

```js
// 11. TSP
function tsp(dist) {
  const n = dist.length;
  const full = 1 << n;
  const dp = Array.from({ length: full }, () => new Array(n).fill(Infinity));
  dp[1][0] = 0; // start at city 0
  for (let mask = 1; mask < full; mask += 2) { // only masks containing city 0
    for (let last = 0; last < n; last++) {
      const cur = dp[mask][last];
      if (cur === Infinity) continue;
      for (let nxt = 0; nxt < n; nxt++) {
        if (mask & (1 << nxt)) continue;
        const nm = mask | (1 << nxt);
        dp[nm][nxt] = Math.min(dp[nm][nxt], cur + dist[last][nxt]);
      }
    }
  }
  let ans = Infinity;
  for (let last = 1; last < n; last++) ans = Math.min(ans, dp[full - 1][last] + dist[last][0]);
  return n === 1 ? 0 : ans;
}
```

**Complexity:** Time O(2^n * n^2), Space O(2^n * n).
**Watch out:** Start from city 0 only (a tour is a cycle, so any start gives the same answer). The loop `mask += 2` visits only masks that contain city 0. Keep `Infinity` for unreachable states, and handle `n = 1`. For asymmetric distances this code is still correct.

---

## Common Mistakes

- **"Bitmask DP works for any n."** It does not. `2^n` doubles with every extra item; `n = 25` is already 33 million masks times `n`. If `n` is bigger, you probably need a different idea (greedy, meet in the middle, or a smaller mask side).
- **Off-by-one between items and bits.** Item `num` (1-indexed) uses bit `num - 1`. Write the conversion once and stick to it.
- **Masking the wrong side.** In hats and teams, put the mask on the SMALL side (people, skills), and loop over the large side (hats, people).
- **Iterating masks in the wrong direction.** For "add one item" transitions, increasing order is fine. For "each hat only once" (0/1 style) you must go downward.
- **Forgetting that `sub = 0` ends the submask loop.** The `for (sub = mask; sub > 0; ...)` form skips the empty subset. Handle it explicitly if you need it.
- **Mixing up `^` and `& ~`.** `mask ^ bit` flips; it adds the bit if it was not there. Use `& ~bit` when the bit might be absent.
- **Operator precedence.** `mask & 1 << i` parses as `mask & (1 << i)` in JS, but `mask & (1 << i) === 0` does not do what you think. Always put parentheses around the test.
- **Storing the order when it is not needed.** If the future depends only on the set, do not add extra dimensions.

## Interview Tips

- Say out loud: "`n` is at most 20, so I will use a bitmask over the items. States are `2^n`, each with `n` transitions."
- Explain WHY the order can be forgotten: "Two orders that use the same items have the same future, so I store one answer per set."
- State the cost before coding: `O(2^n * n)` for add-one transitions, `O(3^n)` for submask transitions, `O(2^n * n^2)` for TSP.
- Mention that the mask can often replace a second index (for example, the number of people already assigned is `popcount(mask)`).
- Follow-ups you will hear: "can you recover the actual path or team?" (store a parent array or a parent mask), "what if `n = 25`?" (meet in the middle or prune), "can you reduce memory?" (rolling layers in the cookies problem).
- Edge cases to mention: `n = 1`, empty graph or empty array, impossible cases (return `-1` / `false`), integer overflow from counts (use modulus or `BigInt`), and JS 32-bit limits on bit operators.
- Before the DP, offer the brute force in one sentence, then say why it is too slow. It shows you know what you are optimizing.

## Practice List

**Easy**
- Subsets (bitmask version)
- Number of 1 Bits
- Power of Two
- Sum of All Subset XOR Totals

**Medium**
- Partition Equal Subset Sum
- Maximum Product of Word Lengths
- Maximum Length of a Concatenated String with Unique Characters
- Campus Bikes II
- Minimum XOR Sum of Two Arrays
- Maximum Compatibility Score Sum
- Distribute Repeating Integers
- Number of Squareful Arrays

**Hard**
- Find the Shortest Superstring (the TSP shape)
- Find Minimum Time to Finish All Jobs
- Maximum AND Sum of Array
- Minimum Incompatibility
- Parallel Courses II
- Maximum Students Taking Exam
- Stickers to Spell Word
- Shortest Path to Get All Keys

## Key Takeaways

- **`n <= 20` is the hint**: a set of items fits in one integer, giving `2^n` states.
- **Forget the order, keep the set**: if the future depends only on which items are used, `dp[mask]` is enough.
- **Know the bit tricks**: test with `mask & (1 << i)`, add with `|`, lowest bit with `mask & -mask`, count bits with `x &= x - 1`.
- **Submask loops cost `3^n`**: use `sub = (sub - 1) & mask`, and only when `n` is about 15 or less.
- **TSP shape is `dp[mask][last]`** with `O(2^n * n^2)` time; BFS over `(mask, node)` solves unweighted "visit all nodes".
- **Put the mask on the small side** (people, skills), and loop over the big side (hats, candidates).
- **Popcount gives a free index**: the number of bits set tells you which person or position is next.

---

[← Previous: Partition and MCM DP](../09-partition-and-mcm-dp/README.md) | [Next: Tree and Digit DP →](../11-tree-and-digit-dp/README.md) | [Back to Dynamic Programming Index](../README.md)
