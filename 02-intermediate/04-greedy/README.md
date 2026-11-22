# Greedy Algorithms

> **Estimated Time:** 5 hr | **Difficulty:** Intermediate
> **Prerequisites:** [02 - Sorting](../../01-core-linear/02-sorting/README.md), [05 - Two Pointers](../../01-core-linear/05-two-pointers/README.md)

---

## What You'll Learn
- You can explain the **greedy-choice property** and why "best now" sometimes gives "best overall".
- You can argue a greedy solution is correct with an **exchange argument**, in plain words.
- You can spot when greedy FAILS (the coin change counterexample) and switch to DP.
- You can solve the classic greedy interview set: intervals, jump game, gas station, candy, scheduling, Huffman.
- You can pick the right sort key (end time, ratio, profit, height) for each problem.

## The Big Idea

Imagine you pack a suitcase at the airport and the bag is almost full. You grab the item with the best "value per kilo" first, then the next best, until the bag is full. You never take an item out again. That is greedy: at each step take the choice that looks best right now, and never go back.

Greedy works only when local best choices add up to a global best answer. Two things must be true:

1. **Greedy-choice property:** some optimal solution contains the greedy first choice.
2. **Optimal substructure:** after you make that choice, what is left is the same kind of problem, smaller.

Why do interviewers love it? Greedy code is usually short (a sort and a loop), but finding the right rule is hard, and you must justify it. A candidate who can say "here is why this rule is safe" stands out. A candidate who just guesses a rule and hopes is easy to catch with one counterexample.

### Exchange argument (in plain words)

This is the standard way to prove a greedy rule. Take any best solution. If it does not use your greedy choice, swap one of its choices for your greedy choice. Show the swap does not make the answer worse. Repeat the swap and you turn that solution into the greedy one without losing anything. So greedy is at least as good as the best.

Example: "pick the meeting that ends earliest". Suppose an optimal schedule starts with a meeting that ends later. Replace it with the earliest-ending meeting. That one ends sooner, so it clashes with nothing the old one did not clash with. The count stays the same. So the earliest-ending choice is safe.

### When greedy FAILS: coin change

Coins are `[1, 3, 4]`, amount `6`. Greedy takes the biggest coin first: `4 + 1 + 1` = 3 coins. The best is `3 + 3` = 2 coins. Greedy took 4 early, and that choice blocked the better answer. There is no exchange that fixes it.

```js
// Greedy coin change (largest first). Correct only for "nice" coin systems.
function greedyCoins(coins, amount) { // coins sorted ascending
  let count = 0;
  for (let i = coins.length - 1; i >= 0; i--) {
    count += Math.floor(amount / coins[i]);
    amount %= coins[i];
  }
  return amount === 0 ? count : -1;
}
greedyCoins([1, 3, 4], 6);       // 3  (wrong, best is 2)
greedyCoins([1, 5, 10, 25], 63); // 6  (right: 25+25+10+1+1+1)
```

Real-world coin systems (US, Indian) are built so greedy works. An arbitrary list of coins is not. If a problem says "any coins" and asks for the minimum count, use DP (see [01 - DP Intuition](../../05-dynamic-programming/01-dp-intuition/README.md)).

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "maximum number of non-overlapping / meetings / activities" | sort by **end time**, take if it fits |
| "minimum removals so intervals do not overlap" | same as above, answer = total - kept |
| "minimum arrows / points to cover all intervals" | sort by end, shoot at the end |
| "can you reach the end", "minimum jumps" | track the **farthest reach** |
| "circular route, start where?" | running tank, reset when it goes negative |
| "each person needs more than a neighbour with higher rating" | two passes, left then right |
| "pick items by value per weight, fractions allowed" | sort by ratio |
| "deadlines and profits, one slot each" | sort by profit, place at latest free slot |
| "combine smallest two repeatedly" | min-heap (Huffman) |
| "you may take any coins, find minimum" and the list looks random | greedy may FAIL, try DP |
| "sort by two keys, then insert" | sort one key descending, other ascending |

Quick test before you commit: try to invent a tiny counterexample (3 to 5 elements). If you cannot break your rule after a few tries, and you can say the exchange argument in one sentence, go.

## Core Concepts / Template

Most greedy solutions have the same shape.

```js
function greedyTemplate(items) {
  // 1. Sort by the key that makes the "best next choice" obvious.
  const sorted = [...items].sort((a, b) => keyOf(a) - keyOf(b));
  // 2. Walk once. Keep a tiny state (last end, current reach, tank ...).
  let answer = 0, state = initialState;
  for (const item of sorted) {
    // 3. If the item is allowed, take it and update state. Never undo.
    if (fits(item, state)) { answer++; state = update(state, item); }
  }
  return answer;
}
```

Variants you will see:

| Pattern | State you keep | Typical problems |
|---|---|---|
| Sort + scan | last chosen end / capacity left | meetings, arrows, knapsack |
| Running reach | farthest index you can touch | jump games |
| Reset on failure | start index, tank | gas station |
| Two passes | left array, then right array | candy |
| Heap | smallest items so far | Huffman, task scheduling |
| Counters instead of sort | counts of 5s, 10s | lemonade change |

Step-by-step trace of "sort by end, take if it fits" on meetings `(1,2) (3,4) (0,6) (5,7) (8,9) (5,9)`:

| Sorted by end | start > lastEnd? | Action | lastEnd |
|---|---|---|---|
| (1,2) | 1 > -inf | take | 2 |
| (3,4) | 3 > 2 | take | 4 |
| (0,6) | 0 > 4 no | skip | 4 |
| (5,7) | 5 > 4 | take | 7 |
| (8,9) | 8 > 7 | take | 9 |
| (5,9) | 5 > 9 no | skip | 9 |

Answer: 4 meetings. Always ask "sorted by what?" first. That one choice is the whole problem.

## Problems (easy -> hard)

### 1. Assign Cookies — Easy
**Companies:** Amazon, Google
**Problem:** Each child has a greed `g[i]` (minimum cookie size they accept). Each cookie has a size `s[j]`. Give each child at most one cookie. Maximise happy children. Example: `g=[1,2,3], s=[1,1]` -> `1`.
**Intuition:** Never waste a big cookie on a child who is happy with a small one. Give the smallest cookie that satisfies the least greedy child.
**Approach 1 - Brute force:** try every assignment. Time O(n!), Space O(n).
**Approach 2 - Optimal:** sort both. Walk cookies from small to large. If the current cookie satisfies the current child, move to the next child. Time O(n log n).

Trace for `g=[1,2,3], s=[1,1]`: cookie 1 feeds child 1 (greed 1). Next cookie 1 < greed 2, skip. Result 1.

```js
function findContentChildren(g, s) {
  g = [...g].sort((a, b) => a - b);
  s = [...s].sort((a, b) => a - b);
  let child = 0;
  for (const cookie of s) {
    if (child < g.length && cookie >= g[child]) child++;
  }
  return child;
}
```

**Complexity:** Time O(n log n + m log m), Space O(1) extra.
**Watch out:** the loop is over cookies, and the child pointer only moves on success. Empty arrays return 0.

### 2. Lemonade Change — Easy
**Companies:** Amazon, Google
**Problem:** Customers pay with 5, 10 or 20 dollar bills for a 5 dollar lemonade, in order. Can you always give correct change? Example: `[5,5,5,10,20]` -> `true`.
**Intuition:** Only the counts of 5s and 10s matter. For a 20, prefer giving back `10 + 5` because 5s are more useful (they can change both 10s and 20s). Keep the flexible bills.
**Approach 1 - Brute force:** try all change combinations (backtracking). Exponential.
**Approach 2 - Optimal:** two counters and the rule above. One pass.

Trace `[5,5,5,10,20]`: five=3 -> 10 comes: five=2, ten=1 -> 20 comes: use ten and five: five=1, ten=0 -> true.

```js
function lemonadeChange(bills) {
  let five = 0, ten = 0;
  for (const b of bills) {
    if (b === 5) five++;
    else if (b === 10) {
      if (five === 0) return false;
      five--; ten++;
    } else {
      if (ten > 0 && five > 0) { ten--; five--; } // keep 5s, they are more useful
      else if (five >= 3) five -= 3;
      else return false;
    }
  }
  return true;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** if you give `5+5+5` for a 20 while you also hold a 10, you can fail later. `[5,5,10,10,20]` returns false.

### 3. Minimum Number of Coins (Indian Coins) — Easy
**Companies:** -
**Problem:** Coins are `1, 2, 5, 10, 20, 50, 100, 200, 500, 2000`. Pay `V` with the fewest coins. Example: `70` -> `[50, 20]`.
**Intuition:** This coin system is built so the biggest coin that fits is always safe. Exchange argument: two small coins can always be replaced by one bigger coin of the same value (for example 20+20+10 -> 50), so the best answer never skips a big coin that fits.
**Approach 1 - Brute force:** try all combinations. Exponential.
**Approach 2 - Better:** DP over amounts. Time O(V * coins), Space O(V). Works for any coins.
**Approach 3 - Optimal:** go from the biggest coin down, take it as often as it fits.

```js
const INDIAN = [1, 2, 5, 10, 20, 50, 100, 200, 500, 2000];
function minCoins(amount) {
  const used = [];
  for (let i = INDIAN.length - 1; i >= 0; i--) {
    while (amount >= INDIAN[i]) { amount -= INDIAN[i]; used.push(INDIAN[i]); }
  }
  return used;
}
// minCoins(121) -> [100, 20, 1]
```

**Complexity:** Time O(V / 2000 + coins), Space O(answer size).
**Watch out:** this only works because of the coin set. Say that out loud, then mention the `[1,3,4]` counterexample. For arbitrary coins use DP.

### 4. Fractional Knapsack — Medium
**Companies:** Amazon, Microsoft
**Problem:** Items have value and weight. The bag has capacity `W`. You may take part of an item. Maximise total value. Example: items `(60,10) (100,20) (120,30)`, `W=50` -> `240`.
**Intuition:** Because you can split items, the best thing is the item with the highest value per kilo. Fill the bag with those first. (In 0/1 knapsack you cannot split, and greedy FAILS there. That is a DP problem.)
**Approach 1 - Brute force:** none that is practical, since fractions make the choices infinite.
**Approach 2 - Optimal:** sort by `value/weight` descending. Take whole items while they fit, then a fraction of the next one.

Trace: ratios 6, 5, 4. Take all of item 1 (60, cap 40). Take all of item 2 (160, cap 20). Take 20/30 of item 3: `120 * 2/3 = 80`. Total 240.

```js
function fractionalKnapsack(items, capacity) { // items: [value, weight]
  const sorted = [...items].sort((a, b) => b[0] / b[1] - a[0] / a[1]);
  let total = 0;
  for (const [value, weight] of sorted) {
    if (capacity === 0) break;
    const take = Math.min(weight, capacity);
    total += value * (take / weight);
    capacity -= take;
  }
  return total;
}
```

**Complexity:** Time O(n log n), Space O(n) for the sorted copy.
**Watch out:** sort by the ratio, not by value or weight alone. The answer can be a fraction, so floating point output is normal.

### 5. N Meetings in One Room (Activity Selection) — Easy
**Companies:** Amazon, Microsoft, Flipkart
**Problem:** Given start and end times, find the maximum number of meetings one room can host. Meetings cannot overlap (a meeting may not start at the exact end time of the previous one in the classic TUF version). Example: start `[1,3,0,5,8,5]`, end `[2,4,6,7,9,9]` -> `4`.
**Intuition:** Finishing early leaves the most room for the rest. Pick the meeting that ends first, then the next one that starts after it.
**Approach 1 - Brute force:** try all subsets, keep the valid ones. Time O(2^n).
**Approach 2 - Better:** DP on sorted-by-start intervals. Time O(n log n) with binary search, more code.
**Approach 3 - Optimal:** sort indices by end time. Take a meeting if its start is greater than the last taken end.

```js
function maxMeetings(start, end) {
  const order = start.map((_, i) => i).sort((a, b) => end[a] - end[b]);
  let count = 0, lastEnd = -Infinity;
  for (const i of order) {
    if (start[i] > lastEnd) { count++; lastEnd = end[i]; }
  }
  return count;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** sorting by start time or by shortest length is WRONG (a long early meeting or a short meeting in the middle can block two others). Check whether touching meetings (`end == next start`) are allowed. Here they clash, so use `>`. For LeetCode-style problems where touching is fine, use `>=`.

### 6. Shortest Job First (Average Waiting Time) — Easy
**Companies:** -
**Problem:** Jobs with burst times all arrive at time 0 and run one at a time. Order them to minimise the average waiting time. Example: `[4,3,7,2,1,2]` -> `29/6` (about 4.83).
**Intuition:** A job's duration is paid by every job behind it. Put short jobs first so the long ones delay fewer people. Exchange argument: if a long job sits right before a short one, swapping them lowers the short one's wait by more than it raises the long one's.
**Approach 1 - Brute force:** try all orders. Time O(n!).
**Approach 2 - Optimal:** sort ascending, accumulate waits.

Trace sorted `[1,2,2,3,4,7]`: waits `0,1,3,5,8,12` -> sum 29 -> 29/6.

```js
function avgWaitingTime(burst) {
  const sorted = [...burst].sort((a, b) => a - b);
  let wait = 0, total = 0;
  for (const t of sorted) { total += wait; wait += t; }
  return total / sorted.length;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** this is the non-preemptive, all-arrive-at-0 version. If jobs arrive at different times you need a min-heap on the jobs that have arrived (see [05 - Heaps](../05-heaps/README.md)). Empty input would divide by zero, so guard it if asked.

### 7. Jump Game — Medium
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** `nums[i]` is the maximum jump length from index `i`. Can you reach the last index starting from index 0? Example: `[2,3,1,1,4]` -> `true`, `[3,2,1,0,4]` -> `false`.
**Intuition:** You do not need the exact path. Just track the farthest index you can reach. If you ever stand on an index beyond that reach, you are stuck.
**Approach 1 - Brute force:** recursion trying every jump length. Time O(2^n).
**Approach 2 - Better:** DP / memo "can reach from i". Time O(n^2), Space O(n).
**Approach 3 - Optimal:** one pass with `reach`.

Trace `[3,2,1,0,4]`: reach after i=0: 3; i=1: 3; i=2: 3; i=3: 3; i=4: 4 > reach 3 -> false.

```js
function canJump(nums) {
  let reach = 0;
  for (let i = 0; i < nums.length; i++) {
    if (i > reach) return false;
    reach = Math.max(reach, i + nums[i]);
  }
  return true;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** `[0]` is true (already at the end). `[0,1]` is false. Check `i > reach` BEFORE updating reach.

### 8. Jump Game II — Medium
**Companies:** Amazon, Google, Meta
**Problem:** Same array, and the end is always reachable. Return the minimum number of jumps. Example: `[2,3,1,1,4]` -> `2`.
**Intuition:** Think in waves, like BFS. Wave 1 is everything reachable with 1 jump. While scanning a wave, track the farthest you could get with one more jump. When the wave ends, you must jump, and the next wave ends at that farthest point.
**Approach 1 - Brute force:** recursion. Exponential.
**Approach 2 - Better:** DP `dp[i] = min jumps`, Time O(n^2).
**Approach 3 - Optimal:** greedy waves, one pass.

Trace `[2,3,1,1,4]`:

| i | farthest | i == curEnd? | jumps | curEnd |
|---|---|---|---|---|
| 0 | 2 | yes | 1 | 2 |
| 1 | 4 | no | 1 | 2 |
| 2 | 4 | yes | 2 | 4 |
| 3 | 4 | no | 2 | 4 |

```js
function jump(nums) {
  let jumps = 0, curEnd = 0, farthest = 0;
  for (let i = 0; i < nums.length - 1; i++) { // stop before the last index
    farthest = Math.max(farthest, i + nums[i]);
    if (i === curEnd) { jumps++; curEnd = farthest; }
  }
  return jumps;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** loop to `length - 1`, not `length`, or you count one jump too many. `[0]` returns 0.

### 9. Minimum Number of Platforms — Medium
**Companies:** Amazon, Microsoft, Flipkart
**Problem:** Given arrival and departure times of trains, find the minimum platforms so no train waits. Example: arrivals `[900,940,950,1100,1500,1800]`, departures `[910,1200,1120,1130,1900,2000]` -> `3`.
**Intuition:** You only need the maximum number of trains at the station at the same moment. Arrivals and departures can be sorted separately because we only count how many are inside, not which train is which.
**Approach 1 - Brute force:** for each train, count how many others overlap it. Time O(n^2).
**Approach 2 - Optimal:** sort both lists, two pointers. If the next arrival is at or before the next departure, a platform is needed. Else a train leaves.

Trace (sorted departures `910,1120,1130,1200,1900,2000`): 900 arrives (1), 940 > 910 so one leaves (0), then 940 (1), 950 (2), 1100 (3, best), then departures free them, 1500 (1), 1800 (2). Best = 3.

```js
function minPlatforms(arr, dep) {
  const a = [...arr].sort((x, y) => x - y);
  const d = [...dep].sort((x, y) => x - y);
  let i = 0, j = 0, cur = 0, best = 0;
  while (i < a.length) {
    if (a[i] <= d[j]) { cur++; i++; best = Math.max(best, cur); } // same time: old train still there
    else { cur--; j++; }
  }
  return best;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** the `<=` matters. If a train arrives exactly when another departs, the classic problem needs 2 platforms. Confirm this rule with the interviewer. The pointer `j` never runs out before `i` does, because every departure is not earlier than its own arrival.

### 10. Non-overlapping Intervals — Medium
**Companies:** Amazon, Meta, Google
**Problem:** Return the minimum number of intervals to remove so the rest do not overlap. Touching ends are fine. Example: `[[1,2],[2,3],[3,4],[1,3]]` -> `1`.
**Intuition:** Removing the fewest is the same as keeping the most. Keeping the most is the meeting-room problem (problem 5). Answer = total - kept.
**Approach 1 - Brute force:** try all subsets. O(2^n).
**Approach 2 - Better:** sort by start, and on overlap remove the one with the larger end. Also O(n log n).
**Approach 3 - Optimal:** sort by end, keep an interval if its start >= last kept end.

Trace sorted by end `[1,2],[2,3],[1,3],[3,4]`: keep [1,2] (end 2), keep [2,3] (end 3), [1,3] start 1 < 3 skip, keep [3,4]. Kept 3, removed 1.

```js
function eraseOverlapIntervals(intervals) {
  const sorted = [...intervals].sort((a, b) => a[1] - b[1]);
  let kept = 0, lastEnd = -Infinity;
  for (const [s, e] of sorted) {
    if (s >= lastEnd) { kept++; lastEnd = e; }
  }
  return intervals.length - kept;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** touching intervals (`[1,2]`,`[2,3]`) do NOT overlap here, so use `>=`. Do not sort by start and keep the first one you see; a long early interval can ruin it.

### 11. Minimum Number of Arrows to Burst Balloons — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Balloons are horizontal ranges `[xstart, xend]`. One arrow shot at x bursts all balloons with `xstart <= x <= xend`. Minimum arrows? Example: `[[10,16],[2,8],[1,6],[7,12]]` -> `2`.
**Intuition:** Sort by end. Shoot at the end of the first balloon: that spot catches the most balloons that start before it. All balloons whose start is at or before that spot are popped. The next balloon that starts after it needs a new arrow.
**Approach 1 - Brute force:** try arrow positions at every endpoint, set cover. Exponential.
**Approach 2 - Optimal:** sort by end, count how many times a balloon starts strictly after the last arrow.

Trace sorted `[1,6],[2,8],[7,12],[10,16]`: shoot at 6 (arrows 1) pops [1,6],[2,8]. [7,12] starts 7 > 6, shoot at 12 (arrows 2), pops [10,16] (10 <= 12). Answer 2.

```js
function findMinArrowShots(points) {
  const sorted = [...points].sort((a, b) => a[1] - b[1]);
  let arrows = 0, shot = -Infinity;
  for (const [s, e] of sorted) {
    if (s > shot) { arrows++; shot = e; }
  }
  return arrows;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** here touching DOES overlap (`[1,2]`,`[2,3]` share x=2), so use `s > shot`, the opposite of problem 10. In JS, `a[1] - b[1]` is safe even for 32-bit extremes (numbers are doubles), unlike in C++/Java.

### 12. Partition Labels — Medium
**Companies:** Amazon, Meta, Google
**Problem:** Split a string into as many parts as possible so each letter appears in at most one part. Return the part sizes. Example: `"ababcbacadefegdehijhklij"` -> `[9,7,8]`.
**Intuition:** If a part contains letter `a`, it must reach the LAST occurrence of `a`. Extend the part end whenever a new letter in it has a later last occurrence. When the scan position reaches the part end, cut.
**Approach 1 - Brute force:** for each cut, check all letters on both sides. Time O(n^2).
**Approach 2 - Optimal:** precompute last index per letter, then one scan.

Trace: first letter `a` last at 8, so end=8. Scanning raises end to 8 (b last 5, c last 7, a 8). At i=8 cut: size 9. Next part starts at 9 (`d`), ends at 15, size 7. Last part size 8.

```js
function partitionLabels(s) {
  const last = {};
  for (let i = 0; i < s.length; i++) last[s[i]] = i;
  const sizes = [];
  let start = 0, end = 0;
  for (let i = 0; i < s.length; i++) {
    end = Math.max(end, last[s[i]]);
    if (i === end) { sizes.push(end - start + 1); start = i + 1; }
  }
  return sizes;
}
```

**Complexity:** Time O(n), Space O(1) (26 letters).
**Watch out:** the cut test is `i === end` after updating `end`. A string with one repeated letter at both ends (`"eccbbbbdec"`) is a single part of size 10.

### 13. Valid Parenthesis String — Medium
**Companies:** Meta, Google, Amazon, Microsoft
**Problem:** String of `(`, `)` and `*`. `*` can be `(`, `)` or empty. Is the string valid? Example: `"(*))"` -> `true`.
**Intuition:** We do not know what each `*` is, so track a range: `lo` = fewest possible open brackets, `hi` = most possible. `*` can lower `lo` (as `)`) or raise `hi` (as `(`). If `hi` ever goes negative, there are too many `)` even in the best case. `lo` is never allowed below 0 (a `*` can just be empty). At the end, `lo` must be 0.
**Approach 1 - Brute force:** try all 3 choices per `*`. Time O(3^k).
**Approach 2 - Better:** DP over (index, open count) or two stacks of indices. Time O(n^2) or O(n).
**Approach 3 - Optimal:** the `lo/hi` range, one pass.

Trace `"(*))"`:

| char | lo | hi |
|---|---|---|
| `(` | 1 | 1 |
| `*` | 0 | 2 |
| `)` | 0 (was -1, clamped) | 1 |
| `)` | 0 (clamped) | 0 |

End `lo == 0` -> true.

```js
function checkValidString(s) {
  let lo = 0, hi = 0;
  for (const ch of s) {
    if (ch === '(') { lo++; hi++; }
    else if (ch === ')') { lo--; hi--; }
    else { lo--; hi++; }
    if (hi < 0) return false;
    if (lo < 0) lo = 0; // a '*' can be empty instead
  }
  return lo === 0;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** `"*("` is false even though counts look fine: `hi` stays positive but `lo` ends at 1. Order matters, which is why we check `hi < 0` at every step and `lo === 0` at the end.

### 14. Gas Station — Medium
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** Stations on a circle. `gas[i]` fuel at station `i`, `cost[i]` to drive to the next. Return the start index for a full loop, or `-1`. Example: `gas=[1,2,3,4,5], cost=[3,4,5,1,2]` -> `3`.
**Intuition:** Two facts. (1) If total gas < total cost, no start works. (2) If you start at `s` and run out at station `i`, then no station between `s` and `i` works either (each of them arrives at `i` with even less fuel than you did). So the next candidate is `i + 1`. If total gas >= total cost, the answer exists and the last candidate is it.
**Approach 1 - Brute force:** simulate from every start. Time O(n^2).
**Approach 2 - Optimal:** one pass with a running tank and a total.

Trace: `diff = gas - cost = [-2,-2,-2,3,3]`. Tank goes negative at 0, 1, 2, so start moves to 1, 2, 3. From 3: tank 3, then 6. Total = 0 >= 0 -> answer 3.

```js
function canCompleteCircuit(gas, cost) {
  let total = 0, tank = 0, start = 0;
  for (let i = 0; i < gas.length; i++) {
    const diff = gas[i] - cost[i];
    total += diff; tank += diff;
    if (tank < 0) { start = i + 1; tank = 0; }
  }
  return total >= 0 ? start : -1;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** do not reset `total`, only `tank`. With total >= 0, `start` can be `n` only if the last diff made tank negative, which would force total negative too, so it never happens.

### 15. Candy — Hard
**Companies:** Amazon, Google, Microsoft
**Problem:** Children stand in a line with ratings. Each gets at least 1 candy, and a child with a higher rating than a neighbour must get more candy than that neighbour. Minimum total candy? Example: `[1,0,2]` -> `5`.
**Intuition:** Two rules (left neighbour, right neighbour) are hard together but easy one at a time. Do a left-to-right pass for the left rule, then a right-to-left pass for the right rule, keeping the max.
**Approach 1 - Brute force:** repeat "fix violations" until none are left. Can take O(n^2).
**Approach 2 - Optimal:** two passes. (A one-pass O(1)-space slope counting version exists, but is trickier to get right in an interview.)

Trace `[1,0,2]`:

| pass | result |
|---|---|
| start | `[1,1,1]` |
| left to right | `[1,1,2]` (2 > 0) |
| right to left | `[2,1,2]` (1 > 0 so max(1, 1+1)) |

Sum = 5.

```js
function candy(ratings) {
  const n = ratings.length;
  const c = new Array(n).fill(1);
  for (let i = 1; i < n; i++) if (ratings[i] > ratings[i - 1]) c[i] = c[i - 1] + 1;
  for (let i = n - 2; i >= 0; i--) if (ratings[i] > ratings[i + 1]) c[i] = Math.max(c[i], c[i + 1] + 1);
  return c.reduce((a, b) => a + b, 0);
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** equal ratings need no relation (`[1,2,2]` -> `[1,2,1]` = 4). In the second pass you must take the max, or you destroy the work of the first pass.

### 16. Job Sequencing with Deadlines — Medium
**Companies:** Amazon, Flipkart, Microsoft
**Problem:** Each job takes 1 unit of time, has a deadline and a profit. One job at a time. Maximise profit. Return `[jobsDone, totalProfit]`. Example: `[[100,2],[19,1],[27,2],[25,1],[15,3]]` (profit, deadline) -> `[3, 142]`.
**Intuition:** Try the most profitable job first. Put it in the latest free slot at or before its deadline. That leaves earlier slots open for jobs with tight deadlines. If no slot is free, drop the job.
**Approach 1 - Brute force:** try all subsets and check feasibility. O(2^n * n).
**Approach 2 - Better:** the slot search above is O(n * maxDeadline) worst case.
**Approach 3 - Optimal:** same greedy, but find the free slot with Union-Find ("latest free slot at or before d"), giving near O(n log n). Mention this as a follow-up; the simple version is usually enough.

Trace sorted by profit: 100 (d=2) -> slot 2. 27 (d=2) -> slot 2 taken, slot 1 free -> slot 1. 25 (d=1) -> none. 19 (d=1) -> none. 15 (d=3) -> slot 3. Count 3, profit 142.

```js
function jobSequencing(jobs) { // jobs: [profit, deadline]
  const sorted = [...jobs].sort((a, b) => b[0] - a[0]);
  const maxD = Math.max(0, ...jobs.map(j => j[1]));
  const slot = new Array(maxD + 1).fill(false);
  let count = 0, profit = 0;
  for (const [p, d] of sorted) {
    for (let t = d; t >= 1; t--) { // latest free slot, keep early ones open
      if (!slot[t]) { slot[t] = true; count++; profit += p; break; }
    }
  }
  return [count, profit];
}
```

**Complexity:** Time O(n log n + n * maxD), Space O(maxD).
**Watch out:** placing a job in the EARLIEST free slot wastes room for tight jobs. `Math.max(...bigArray)` can overflow the call stack for very large inputs; use a loop then. Deadlines can be larger than `n`, so cap them if memory matters.

### 17. Merge Triplets to Form Target Triplet — Medium
**Companies:** Google, Amazon
**Problem:** You may pick triplets and merge them by taking the max of each position. Can you reach `target`? Example: triplets `[[2,5,3],[1,8,4],[1,7,5]]`, target `[2,7,5]` -> `true`.
**Intuition:** Merging by max can only increase values. So any triplet with a value bigger than the target in any position is poison: never use it. Among the safe triplets, you just need each target value to appear at the right position in at least one of them.
**Approach 1 - Brute force:** try all subsets. O(2^n).
**Approach 2 - Optimal:** filter safe triplets, mark which positions hit the target exactly.

Trace: `[2,5,3]` safe (all <= target), hits position 0. `[1,8,4]` has 8 > 7, skip. `[1,7,5]` safe, hits positions 1 and 2. All three hit -> true.

```js
function mergeTriplets(triplets, target) {
  const got = [false, false, false];
  for (const t of triplets) {
    if (t[0] > target[0] || t[1] > target[1] || t[2] > target[2]) continue;
    for (let k = 0; k < 3; k++) if (t[k] === target[k]) got[k] = true;
  }
  return got.every(Boolean);
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** do not merge actual triplets; you never need to. Reaching a value in one position from a bad triplet is not allowed even if the other positions look useful.

### 18. Hand of Straights — Medium
**Companies:** Google, Amazon, Meta
**Problem:** Can you split the cards into groups of `groupSize` consecutive numbers? Example: `hand=[1,2,3,6,2,3,4,7,8], groupSize=3` -> `true`.
**Intuition:** The smallest card must start a group (nothing smaller is left to put in front of it). So repeatedly take the smallest card and build `x, x+1, ..., x+k-1`. If any card is missing, fail.
**Approach 1 - Brute force:** sort and repeatedly search and remove cards. Time O(n^2).
**Approach 2 - Optimal:** count map. Walk the sorted distinct values. For a value with count `need > 0`, subtract `need` from each of the next `groupSize` values (starting a `need` groups at once).

Trace: counts `1:1, 2:2, 3:2, 4:1, 6:1, 7:1, 8:1`. At 1: need 1, take 1,2,3 -> `2:1, 3:1`. At 2: need 1, take 2,3,4 -> ok. At 3 and 4: counts 0. At 6: take 6,7,8. All good.

```js
function isNStraightHand(hand, groupSize) {
  if (hand.length % groupSize !== 0) return false;
  const count = new Map();
  for (const c of hand) count.set(c, (count.get(c) || 0) + 1);
  for (const c of [...count.keys()].sort((a, b) => a - b)) {
    const need = count.get(c);
    if (need === 0) continue;
    for (let k = 0; k < groupSize; k++) {
      const have = count.get(c + k) || 0;
      if (have < need) return false;
      count.set(c + k, have - need);
    }
  }
  return true;
}
```

**Complexity:** Time O(d log d + d * groupSize) for d distinct values, Space O(d).
**Watch out:** check `length % groupSize` first. Missing keys in the map count as 0. This is the same problem as "Divide Array in Sets of K Consecutive Numbers".

### 19. Task Scheduler — Medium
**Companies:** Meta, Amazon, Google, Microsoft
**Problem:** Tasks are letters. The same letter needs `n` idle-or-other slots between two runs. Return the minimum time (slots) to finish all. Example: `AAABBB`, `n=2` -> `8`.
**Intuition:** The most frequent task is the bottleneck. Lay it out as frames of size `n+1`: `A _ _ | A _ _ | A`. Fill the gaps with other tasks. The last frame only needs as many slots as there are tasks with the max frequency. If there are more tasks than the frames offer, there is no idle time at all and the answer is just the number of tasks.
**Approach 1 - Brute force / simulation:** each step run the available task with the biggest remaining count, using a max-heap and a cooldown queue. Time O(T log 26), Space O(26). Works, and is a good fallback (see [05 - Heaps](../05-heaps/README.md)).
**Approach 2 - Optimal (formula):** `max( tasks, (maxFreq - 1) * (n + 1) + countOfMaxFreq )`.

Trace `AAABBB`, n=2: maxFreq 3, two letters with freq 3. `(3-1)*3 + 2 = 8`. Schedule: `A B _ A B _ A B`. Total tasks 6 < 8, so 8.

```js
function leastInterval(tasks, n) {
  const freq = new Array(26).fill(0);
  for (const t of tasks) freq[t.charCodeAt(0) - 65]++;
  const max = Math.max(...freq);
  const numMax = freq.filter(f => f === max).length;
  return Math.max(tasks.length, (max - 1) * (n + 1) + numMax);
}
```

**Complexity:** Time O(T), Space O(1).
**Watch out:** the `Math.max(tasks.length, ...)` part is needed when there are many distinct tasks (no idle time). With `n = 0` the formula gives `tasks.length` too.

### 20. Queue Reconstruction by Height — Medium
**Companies:** Google, Amazon, Meta
**Problem:** `people[i] = [h, k]`: height `h` and exactly `k` people in front with height `>= h`. Rebuild the queue. Example: `[[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]` -> `[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]`.
**Intuition:** Place the tallest first. Shorter people never affect a taller person's `k` (a taller person only counts people at least as tall). So when a person with height `h` is inserted, everyone already in the list is `>= h`. Their `k` is simply the index to insert at.
**Approach 1 - Brute force:** try every permutation. O(n!).
**Approach 2 - Optimal:** sort by height descending, and for ties by `k` ascending. Insert each person at index `k`.

Trace: sorted `[7,0],[7,1],[6,1],[5,0],[5,2],[4,4]`. Insert one by one:

```
[7,0]
[7,0] [7,1]
[7,0] [6,1] [7,1]
[5,0] [7,0] [6,1] [7,1]
[5,0] [7,0] [5,2] [6,1] [7,1]
[5,0] [7,0] [5,2] [6,1] [4,4] [7,1]
```

```js
function reconstructQueue(people) {
  const sorted = [...people].sort((a, b) => b[0] - a[0] || a[1] - b[1]);
  const res = [];
  for (const p of sorted) res.splice(p[1], 0, p);
  return res;
}
```

**Complexity:** Time O(n^2) because `splice` shifts elements, Space O(n).
**Watch out:** the tie-break `k` ascending is important. Using descending breaks the insertion. A Fenwick tree or segment tree can bring it to O(n log n), mention it as a follow-up.

### 21. Huffman Coding — Hard
**Companies:** Amazon, Google, Microsoft
**Problem:** Given characters and their frequencies, build prefix-free binary codes with the smallest total encoded length. Example: `{a:5, b:9, c:12, d:13, e:16, f:45}` -> total bits `224`, and `f` gets a 1-bit code.
**Intuition:** Rare characters should get long codes, common ones short codes. Build a tree from the bottom: always merge the two LEAST frequent nodes into one parent. Repeat until one tree is left. Exchange argument: the two rarest symbols can always sit as sibling leaves at the deepest level.
**Approach 1 - Brute force:** try all prefix-free trees. Exponential.
**Approach 2 - Optimal:** a min-heap. Pop two, push their sum. Walk the final tree: left edge `0`, right edge `1`.

Trace merges: `5+9=14`, `12+13=25`, `14+16=30`, `25+30=55`, `45+55=100`. Depths: f=1, c,d,e=3, a,b=4. Bits: `45*1 + (12+13+16)*3 + (5+9)*4 = 45 + 123 + 56 = 224`.

```js
function huffman(freqs) { // freqs: {char: count}  -> {char: "0101"}
  const heap = []; // tiny binary min-heap on node.f
  const push = (x) => {
    heap.push(x);
    let i = heap.length - 1;
    while (i > 0) {
      const p = (i - 1) >> 1;
      if (heap[p].f <= heap[i].f) break;
      [heap[p], heap[i]] = [heap[i], heap[p]]; i = p;
    }
  };
  const pop = () => {
    const top = heap[0], last = heap.pop();
    if (heap.length) {
      heap[0] = last;
      let i = 0;
      for (;;) {
        let m = i; const l = 2 * i + 1, r = l + 1;
        if (l < heap.length && heap[l].f < heap[m].f) m = l;
        if (r < heap.length && heap[r].f < heap[m].f) m = r;
        if (m === i) break;
        [heap[m], heap[i]] = [heap[i], heap[m]]; i = m;
      }
    }
    return top;
  };
  for (const ch of Object.keys(freqs)) push({ f: freqs[ch], ch });
  if (heap.length === 1) return { [heap[0].ch]: '0' }; // single symbol still needs 1 bit
  while (heap.length > 1) {
    const a = pop(), b = pop();
    push({ f: a.f + b.f, left: a, right: b });
  }
  const codes = {};
  (function walk(node, code) {
    if (node.ch !== undefined) { codes[node.ch] = code; return; }
    walk(node.left, code + '0'); walk(node.right, code + '1');
  })(heap[0], '');
  return codes;
}
```

**Complexity:** Time O(k log k) for k distinct symbols, Space O(k).
**Watch out:** JavaScript has no built-in heap, so say you would use one from a library or write it as above. Exact codes can differ when frequencies tie, but the TOTAL cost is always the same. A one-symbol input needs a special case.

## Common Mistakes
- **"Greedy always works if the first idea looks right."** No. Always try a tiny counterexample (coin change `[1,3,4]`, 0/1 knapsack).
- **Sorting by the wrong key.** Meetings sorted by start or by length fail. Sort by END time.
- **Mixing `>` and `>=` for touching intervals.** Non-overlapping intervals allow touching (`>=`). Arrows and balloons do not (`>`). Read the statement.
- **Forgetting the proof.** "It passes the samples" is not an answer. Say the exchange argument in one sentence.
- **Updating state before checking it** (Jump Game: check `i > reach` first).
- **Using greedy for 0/1 knapsack or general coin change.** Those need DP.
- **Mutating the input array with `sort`.** `sort` changes the array in place. Copy first with `[...arr]` if the caller still needs it.
- **Subtraction comparators with huge values in other languages.** In JS doubles it is safe, but mention you would use a safe comparison in Java or C++.

## Interview Tips
- Say it out loud: "I will sort by X because choosing the item that finishes first leaves the most room for the rest."
- State the proof shape: "Take any optimal answer. If it differs from mine at the first step, I can swap in my choice without making it worse."
- Offer a brute force first (even just in words), then show the greedy as the improvement.
- Ask about touching boundaries, ties, empty input, and whether the answer must be the set or only the count.
- Follow-ups you will hear: "Can you prove it?", "What if coins are `[1,3,4]`?" (DP), "What if intervals have weights?" (weighted interval scheduling is DP), "Can you do it in O(1) space?" (Candy, Jump Game).
- Edge cases to mention: single element, all equal elements, already sorted input, huge values, and the total being exactly zero (gas station).
- If greedy fails mid-way on an example, do not panic: say "this suggests I need DP" and pivot. Showing that you can detect failure is a plus.

## Practice List
**Easy**
- Best Time to Buy and Sell Stock II
- Assign Cookies
- Lemonade Change
- Maximum Subarray (Kadane)
- Is Subsequence
- Maximum Units on a Truck

**Medium**
- Jump Game, Jump Game II
- Gas Station
- Non-overlapping Intervals
- Minimum Number of Arrows to Burst Balloons
- Partition Labels
- Valid Parenthesis String
- Merge Triplets to Form Target Triplet
- Hand of Straights
- Task Scheduler
- Queue Reconstruction by Height
- Boats to Save People
- Reorganize String
- Two City Scheduling
- Minimum Add to Make Parentheses Valid
- Remove K Digits
- Car Fleet
- Meeting Rooms II
- Course Schedule III
- Divide Array in Sets of K Consecutive Numbers

**Hard**
- Candy
- Minimum Number of Refueling Stops
- IPO
- Smallest Range Covering Elements from K Lists
- Create Maximum Number
- Patching Array

## Key Takeaways
- **Greedy = best local choice, never undone.** It works only if the greedy-choice property and optimal substructure hold.
- **Prove with an exchange argument:** swap an optimal answer's choice for yours without losing value.
- **The sort key is the whole problem.** End time for intervals, ratio for knapsack, profit for jobs, height-then-k for queues.
- **Always hunt for a counterexample.** Coin change with `[1,3,4]` and 0/1 knapsack are the standard failures; those need DP.
- **Reach / tank / counter state** replaces DP in jump games, gas station and lemonade change, giving O(n) time and O(1) space.
- **Boundaries matter:** `>` versus `>=` decides touching intervals (arrows vs non-overlapping).
- **Heap-based greedy** (Huffman, task scheduling, SJF with arrivals) is the bridge to the next chapter.

---
[← Previous: Divide and Conquer](../03-divide-and-conquer/README.md) | [Next: Heaps →](../05-heaps/README.md) | [Back to Intermediate Index](../README.md)
