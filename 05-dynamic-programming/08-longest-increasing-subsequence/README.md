# Longest Increasing Subsequence

> **Estimated Time:** 4 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - DP Intuition](../01-dp-intuition/README.md), [02 - One-Dimensional DP](../02-one-d-dp/README.md), [04 - Subset and Subsequence DP](../04-subset-and-subsequence-dp/README.md)

---

## What You'll Learn

- You can solve Longest Increasing Subsequence (LIS) with the simple O(n^2) DP and explain every line.
- You can solve it in O(n log n) with the "tails" array and explain **why** that array works.
- You can rebuild and print the actual subsequence, not just its length.
- You can spot the LIS shape hiding inside problems like divisible subset, string chain, bitonic, envelopes and pair chain.
- You can count how many LIS exist, and build the "left and right" DP used by mountain and bitonic problems.

---

## The Big Idea

Think of a queue of people at a concert. You want to pick some of them, keeping their order in the line, so that each next person is taller than the one before. You may skip anyone. The most people you can pick this way is the LIS. You do not move anyone. You only choose who stays.

A **subsequence** keeps the original order but may skip items. It is not a **subarray**, which must be contiguous. So `[2,5,7]` is a subsequence of `[2,9,5,3,7]`.

The O(n^2) idea is simple. Let `dp[i]` be the length of the longest increasing subsequence that **ends exactly at index i**. To fill it, look at every earlier `j` with `a[j] < a[i]` and extend the best one.

The O(n log n) idea is a clever shortcut. Instead of storing the best length per index, keep for each length the **smallest possible last value**. Smaller tails are always better, because they are easier to extend. That array is sorted, so we can binary search it.

Interviewers love LIS because:

- It tests whether you can define a DP state ("ends at i") cleanly.
- It has two very different solutions, so they can ask "can you do better?".
- Many hard-looking problems are LIS in disguise (envelopes, string chain, divisible subset).

---

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "longest increasing subsequence" | LIS, O(n^2) or tails + binary search |
| "longest chain", "each item must be bigger/fit the previous" | sort first, then LIS |
| "divisible subset" | sort, then LIS with `%` as the "increasing" test |
| "nested boxes / envelopes in 2D" | sort one side, LIS on the other |
| "mountain", "bitonic", "up then down" | LIS from the left + LIS from the right, combine at the peak |
| "how many longest subsequences" | keep a `count` array next to `len` |
| "arithmetic subsequence" | DP on index plus the difference (a map per index) |
| "does an increasing triplet exist" | LIS of length 3, keep two smallest tails |
| "intervals/pairs, pick the most non-overlapping" | greedy by end time (a chain, but greedy is enough) |

---

## Core Concepts / Template

### Template 1: O(n^2) DP

`dp[i]` = length of the longest increasing subsequence that ends at `a[i]`.

```js
function lisN2(a) {
  const n = a.length;
  if (n === 0) return 0;
  const dp = Array(n).fill(1); // every element alone is a subsequence of length 1
  let best = 1;
  for (let i = 1; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (a[j] < a[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
    }
    best = Math.max(best, dp[i]); // answer is the max over all i, not dp[n-1]
  }
  return best;
}
```

Time O(n^2), Space O(n).

Trace on `[10, 9, 2, 5, 3, 7, 101, 18]`:

| i | a[i] | earlier smaller values | dp[i] |
|---|------|------------------------|-------|
| 0 | 10 | none | 1 |
| 1 | 9 | none | 1 |
| 2 | 2 | none | 1 |
| 3 | 5 | 2 (dp 1) | 2 |
| 4 | 3 | 2 (dp 1) | 2 |
| 5 | 7 | 2, 5, 3 (best dp 2) | 3 |
| 6 | 101 | all others (best dp 3) | 4 |
| 7 | 18 | 10, 9, 2, 5, 3, 7 (best dp 3) | 4 |

Answer: max of dp = 4.

### Template 2: O(n log n) with the tails array

Keep an array `tails`. `tails[k]` = the **smallest last value** of any increasing subsequence of length `k+1` seen so far.

For each number `x`:

1. Find the first position `p` where `tails[p] >= x` (lower bound).
2. Set `tails[p] = x`. If `p` is the end, this appends and the LIS grows by one.

```js
function lowerBound(t, x) { // first index with t[i] >= x
  let lo = 0, hi = t.length;
  while (lo < hi) {
    const m = (lo + hi) >> 1;
    if (t[m] < x) lo = m + 1;
    else hi = m;
  }
  return lo;
}

function lisFast(a) {
  const tails = [];
  for (const x of a) tails[lowerBound(tails, x)] = x;
  return tails.length;
}
```

Time O(n log n), Space O(n).

Trace on `[10, 9, 2, 5, 3, 7, 101, 18]`:

| x | tails after |
|---|-------------|
| 10 | [10] |
| 9 | [9] (replaces 10) |
| 2 | [2] |
| 5 | [2, 5] (append) |
| 3 | [2, 3] (replaces 5) |
| 7 | [2, 3, 7] (append) |
| 101 | [2, 3, 7, 101] (append) |
| 18 | [2, 3, 7, 18] (replaces 101) |

Length = 4.

### Why the tails array works

Two facts make it correct.

**Fact 1: tails is always sorted (strictly increasing).** Suppose `tails[k]` is the smallest end of a length `k+1` subsequence. Take that subsequence and drop its last element. You get a length `k` subsequence whose end is smaller than `tails[k]`. So `tails[k-1] < tails[k]`. Sorted means we can binary search.

**Fact 2: smaller tails are never worse.** Say two subsequences both have length 3, one ends in 7 and one ends in 18. Anything you can append to the one ending in 18 (a bigger number) you can also append to the one ending in 7. So we only need to remember the 7. That is why we **replace** a bigger tail with a smaller `x`: nothing is lost, and future numbers have more room.

When `x` arrives, the first tail `>= x` at position `p` means: there is a length `p` subsequence ending below `x` (that is `tails[p-1] < x`), so we can build a length `p+1` ending in `x`. And it is a smaller end than the old `tails[p]`. So we overwrite.

**Warning:** `tails` is NOT the LIS itself. In the trace the final `[2, 3, 7, 18]` happens to be valid, but in general the array mixes values from different subsequences. It gives the correct **length** only.

Use **lower bound** (`>= x`) for strictly increasing. Use upper bound (`> x`) if equal values are allowed (non-decreasing).

### Template 3: Print the LIS

Store a `prev[i]` pointer next to `dp[i]`. Remember which `j` gave the best extension. At the end, walk back from the best index.

```js
function printLis(a) {
  const n = a.length;
  if (n === 0) return [];
  const dp = Array(n).fill(1), prev = Array(n).fill(-1);
  let end = 0;
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (a[j] < a[i] && dp[j] + 1 > dp[i]) { dp[i] = dp[j] + 1; prev[i] = j; }
    }
    if (dp[i] > dp[end]) end = i;
  }
  const res = [];
  for (let i = end; i !== -1; i = prev[i]) res.push(a[i]);
  return res.reverse();
}
```

---

## Problems (easy -> hard)

### 1. Longest Increasing Subsequence — Medium
**Companies:** Amazon, Google, Microsoft, Meta, Bloomberg
**Problem:** Given an integer array, return the length of the longest strictly increasing subsequence. Example: `[10,9,2,5,3,7,101,18]` -> `4` (for example `[2,3,7,101]`).
**Intuition:** Every element ends some increasing run. The best run ending at `i` is one more than the best run ending at some earlier smaller element.
**Approach 1 - Brute force:** Try every subsequence (take or skip each element) and check it is increasing. Time O(2^n), Space O(n).
**Approach 2 - Better:** `dp[i]` = longest ending at `i`, check all `j < i`. Time O(n^2), Space O(n). Good enough when n is up to a few thousand.
**Approach 3 - Optimal:** The tails array with binary search (see Template 2). Each number does one binary search, so n log n. Dry run is in the Core Concepts section above.

```js
// optimal, tested
function lengthOfLIS(a) {
  const tails = [];
  for (const x of a) {
    let lo = 0, hi = tails.length;
    while (lo < hi) {
      const m = (lo + hi) >> 1;
      if (tails[m] < x) lo = m + 1;
      else hi = m;
    }
    tails[lo] = x; // lo === length means append
  }
  return tails.length;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** Using `<=` instead of `<` in the binary search lets equal values extend the sequence, which solves the "non-decreasing" variant, not this one. For `[7,7,7]` the answer must be 1. Also, do not return `dp[n-1]`; the best run may end anywhere.

### 2. Print the Longest Increasing Subsequence — Medium
**Companies:** Amazon, Microsoft, Adobe
**Problem:** Return one actual longest increasing subsequence, not just its length. Example: `[10,9,2,5,3,7,101,18]` -> `[2,5,7,101]` (any valid LIS is fine unless told otherwise).
**Intuition:** A DP value tells you "how long". A `prev` pointer tells you "where I came from". Follow the pointers back from the best end.
**Approach 1 - Brute force:** Enumerate all subsequences, keep the longest increasing one. Time O(2^n), Space O(n).
**Approach 2 - Better:** Use the O(n^2) DP with a `prev` array (below). Time O(n^2), Space O(n).
**Approach 3 - Optimal:** O(n log n) is possible too. Store, for each tails slot, the **index** of the element, and for each element the index of the tail just before its slot at the moment it was placed. Then walk back from the last slot. It is correct but trickier, and interviewers usually accept the O(n^2) version for printing.

Trace of `prev` on `[10,9,2,5,3,7,101,18]`:

| i | a[i] | dp | prev |
|---|------|----|------|
| 3 | 5 | 2 | 2 (value 2) |
| 4 | 3 | 2 | 2 (value 2) |
| 5 | 7 | 3 | 3 (value 5) |
| 6 | 101 | 4 | 5 (value 7) |

Best end is index 6 (first index with dp 4). Walk: 101 <- 7 <- 5 <- 2. Reverse: `[2,5,7,101]`.

```js
// tested
function printLIS(a) {
  const n = a.length;
  if (n === 0) return [];
  const dp = Array(n).fill(1), prev = Array(n).fill(-1);
  let end = 0;
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (a[j] < a[i] && dp[j] + 1 > dp[i]) { dp[i] = dp[j] + 1; prev[i] = j; }
    }
    if (dp[i] > dp[end]) end = i;
  }
  const res = [];
  for (let i = end; i !== -1; i = prev[i]) res.push(a[i]);
  return res.reverse();
}
```

**Complexity:** Time O(n^2), Space O(n).
**Watch out:** Do not print the `tails` array and call it the LIS. Also remember to reverse, since you walk backwards. When many LIS exist, different tie rules give different valid answers; ask if a specific one (such as lexicographically smallest) is needed.

### 3. Increasing Triplet Subsequence — Medium
**Companies:** Amazon, Meta, Microsoft
**Problem:** Return `true` if there exist indices `i < j < k` with `a[i] < a[j] < a[k]`. Example: `[2,1,5,0,4,6]` -> `true`.
**Intuition:** This is "is the LIS at least 3?". The tails array never needs more than 2 slots. Keep `first` (smallest tail of length 1) and `second` (smallest tail of length 2). If anything beats `second`, we have three.
**Approach 1 - Brute force:** Three nested loops. Time O(n^3), Space O(1).
**Approach 2 - Better:** For each middle `j`, check a smaller number exists on the left and a bigger one on the right using prefix min and suffix max arrays. Time O(n), Space O(n).
**Approach 3 - Optimal:** Two variables, the tails idea with a cap of 2.

Trace on `[2,1,5,0,4,6]`:

| x | first | second | note |
|---|-------|--------|------|
| 2 | 2 | inf | |
| 1 | 1 | inf | smaller start |
| 5 | 1 | 5 | length-2 chain 1,5 |
| 0 | 0 | 5 | |
| 4 | 0 | 4 | better length-2 tail |
| 6 | | | 6 > second, return true |

```js
// tested
function increasingTriplet(a) {
  let first = Infinity, second = Infinity;
  for (const x of a) {
    if (x <= first) first = x;
    else if (x <= second) second = x;
    else return true;
  }
  return false;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Use `<=` so equal values do not count as increasing (`[1,1,1]` must be `false`). When `first` is replaced after `second` was set (like `0` above), `first` no longer sits before `second` in the array. That is fine: `second` still proves a smaller value existed earlier, and we only need a yes/no.

### 4. Maximum Length of Pair Chain — Medium
**Companies:** Amazon, Google
**Problem:** Given pairs `[left, right]` with `left < right`, a pair `q` can follow `p` if `p[1] < q[0]`. Return the longest chain you can form (any order of picking). Example: `[[1,2],[2,3],[3,4]]` -> `2`.
**Intuition:** Order is free, so sort. After sorting, it is LIS-like. But there is a greedy that is even better: it is the "activity selection" problem. Always take the pair that ends earliest, because it leaves the most room.
**Approach 1 - Brute force:** Try all subsets of pairs. Time O(2^n), Space O(n).
**Approach 2 - Better:** Sort by start, then LIS-style DP where `j` can precede `i` if `pairs[j][1] < pairs[i][0]`. Time O(n^2), Space O(n).
**Approach 3 - Optimal:** Sort by end. Walk through. If the pair starts after the current chain end, take it and update the end. Greedy works because picking the earliest end can never hurt.

Trace on `[[1,2],[7,8],[4,5]]` sorted by end: `[1,2],[4,5],[7,8]`.

| pair | start > end? | chain | end |
|------|--------------|-------|-----|
| [1,2] | 1 > -inf | 1 | 2 |
| [4,5] | 4 > 2 | 2 | 5 |
| [7,8] | 7 > 5 | 3 | 8 |

```js
// tested
function findLongestChain(pairs) {
  pairs = [...pairs].sort((a, b) => a[1] - b[1]); // copy, do not mutate input
  let end = -Infinity, count = 0;
  for (const [s, e] of pairs) {
    if (s > end) { count++; end = e; }
  }
  return count;
}
```

**Complexity:** Time O(n log n), Space O(n) for the copy (O(1) extra if mutation is allowed).
**Watch out:** The rule is strict (`s > end`). Touching pairs like `[1,2],[2,3]` cannot chain. Sort by **end**, not start, for the greedy; sorting by start breaks it.

### 5. Number of Longest Increasing Subsequences — Medium
**Companies:** Amazon, Google, Meta
**Problem:** Return how many longest strictly increasing subsequences exist. Example: `[1,3,5,4,7]` -> `2` (`1,3,5,7` and `1,3,4,7`).
**Intuition:** Track two numbers per index: `len[i]` (longest ending here) and `cnt[i]` (how many ways to get that longest length). When a longer path is found, replace the count. When an equal-length path is found, add the counts.
**Approach 1 - Brute force:** Generate all subsequences, find the max length, count them. Time O(2^n), Space O(n).
**Approach 2 - Better:** The O(n^2) `len` and `cnt` DP below. Time O(n^2), Space O(n).
**Approach 3 - Optimal:** O(n log n) exists using a Fenwick tree or a layered tails structure, but it is rarely asked. Say that you know it exists, and give the O(n^2).

Trace on `[1,3,5,4,7]`:

| i | a[i] | len | cnt | why |
|---|------|-----|-----|-----|
| 0 | 1 | 1 | 1 | |
| 1 | 3 | 2 | 1 | from 1 |
| 2 | 5 | 3 | 1 | from 3 |
| 3 | 4 | 3 | 1 | from 3 |
| 4 | 7 | 4 | 2 | from 5 (cnt 1) plus from 4 (cnt 1) |

Longest = 4. Sum `cnt` over indices with `len == 4` -> 2.

```js
// tested
function findNumberOfLIS(a) {
  const n = a.length;
  if (n === 0) return 0;
  const len = Array(n).fill(1), cnt = Array(n).fill(1);
  let best = 1;
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (a[j] < a[i]) {
        if (len[j] + 1 > len[i]) { len[i] = len[j] + 1; cnt[i] = cnt[j]; }
        else if (len[j] + 1 === len[i]) cnt[i] += cnt[j];
      }
    }
    best = Math.max(best, len[i]);
  }
  let res = 0;
  for (let i = 0; i < n; i++) if (len[i] === best) res += cnt[i];
  return res;
}
```

**Complexity:** Time O(n^2), Space O(n).
**Watch out:** Sum the counts over **all** indices that reach the best length, not just the last one. All-equal arrays like `[2,2,2,2,2]` give `5` (each single element is an LIS of length 1). On the real LeetCode problem counts fit in a normal number; in other settings mention overflow or BigInt.

### 6. Largest Divisible Subset — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** Given distinct positive integers, return the largest subset where every pair `(x, y)` satisfies `x % y == 0` or `y % x == 0`. Example: `[1,2,4,8]` -> `[1,2,4,8]`; `[1,2,3]` -> `[1,2]` (or `[1,3]`).
**Intuition:** After sorting, if `a[j]` divides `a[i]` and `a[j-prev]` divides `a[j]`, then divisibility is transitive, so only the **last** link matters. It becomes LIS where "increasing" means "divides".
**Approach 1 - Brute force:** Check all subsets. Time O(2^n * n^2), Space O(n).
**Approach 2 - Better:** Sort, recursion with memo on (index, previous). Time O(n^2), Space O(n^2).
**Approach 3 - Optimal:** Sort, then the print-LIS DP with `a[i] % a[j] === 0` as the test, and `prev` pointers to rebuild.

Trace on `[1,2,4,8]` (sorted): dp = 1, 2, 3, 4, with prev chain 8 -> 4 -> 2 -> 1.

```js
// tested
function largestDivisibleSubset(nums) {
  const n = nums.length;
  if (n === 0) return [];
  const a = [...nums].sort((x, y) => x - y);
  const dp = Array(n).fill(1), prev = Array(n).fill(-1);
  let end = 0;
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (a[i] % a[j] === 0 && dp[j] + 1 > dp[i]) { dp[i] = dp[j] + 1; prev[i] = j; }
    }
    if (dp[i] > dp[end]) end = i;
  }
  const res = [];
  for (let i = end; i !== -1; i = prev[i]) res.push(a[i]);
  return res.reverse();
}
```

**Complexity:** Time O(n^2), Space O(n).
**Watch out:** You must sort first, or the "only the last link matters" argument fails. Checking only the immediate previous element is enough because divisibility is transitive.

### 7. Longest String Chain — Medium
**Companies:** Google, Amazon, Meta
**Problem:** A word `a` is a predecessor of `b` if you can insert exactly one letter into `a` to get `b`. Return the longest chain `w1 -> w2 -> ...` from the list. Example: `["a","b","ba","bca","bda","bdca"]` -> `4` (`a -> ba -> bda -> bdca`).
**Intuition:** Sort words by length. For each word, try deleting each letter to get its possible predecessors, and look them up in a map of best chain lengths. It is LIS where "can precede" means "one letter shorter and a subsequence".
**Approach 1 - Brute force:** For each pair, test if one is a predecessor, then DP over pairs. Time O(n^2 * L), Space O(n).
**Approach 2 - Better:** Same pair DP with a quick predecessor check using two pointers. Still O(n^2 * L).
**Approach 3 - Optimal:** Map `word -> best chain ending at word`. For each word delete each char (L options, each slice costs O(L)). Time O(n * L^2) plus sorting.

Trace on `["a","b","ba","bca","bda","bdca"]` sorted by length:

| word | predecessors found | chain |
|------|--------------------|-------|
| a | none | 1 |
| b | none | 1 |
| ba | "a"(1), "b"(1) | 2 |
| bca | "ca"(none), "ba"(2), "bc"(none) | 3 |
| bda | "ba"(2) | 3 |
| bdca | "dca","bca"(3),"bda"(3),"bdc" | 4 |

```js
// tested
function longestStrChain(words) {
  const sorted = [...words].sort((a, b) => a.length - b.length);
  const dp = new Map();
  let best = 0;
  for (const w of sorted) {
    let cur = 1;
    for (let i = 0; i < w.length; i++) {
      const p = w.slice(0, i) + w.slice(i + 1);
      if (dp.has(p)) cur = Math.max(cur, dp.get(p) + 1);
    }
    dp.set(w, cur);
    best = Math.max(best, cur);
  }
  return best;
}
```

**Complexity:** Time O(n log n + n * L^2), Space O(n * L).
**Watch out:** Sorting by length is what guarantees predecessors are processed first. Single-letter words have the empty string as a predecessor, but it is never in the map, so they correctly stay at 1. Return 0 for an empty list (the code does, since `best` starts at 0).

### 8. Longest Bitonic Subsequence — Medium
**Companies:** Amazon, Microsoft, Samsung
**Problem:** Find the longest subsequence that first strictly increases and then strictly decreases. Pure increasing or pure decreasing counts too (the classic GFG version). Example: `[1,11,2,10,4,5,2,1]` -> `6` (`1,2,10,4,2,1`).
**Intuition:** Pick a peak `i`. The best bitonic sequence with that peak is the LIS **ending** at `i` plus the longest decreasing run **starting** at `i`, minus 1 because the peak is counted twice.
**Approach 1 - Brute force:** Try all subsequences. Time O(2^n), Space O(n).
**Approach 2 - Better:** For each peak, recompute left and right with loops. Time O(n^3).
**Approach 3 - Optimal:** Compute `L[i]` with the LIS DP from the left, and `R[i]` by running the same DP on the reversed array, then flipping back. Answer is the max of `L[i] + R[i] - 1`.

Trace on `[1,11,2,10,4,5,2,1]`:

| i | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|---|
| a | 1 | 11 | 2 | 10 | 4 | 5 | 2 | 1 |
| L | 1 | 2 | 2 | 3 | 3 | 4 | 2 | 1 |
| R | 1 | 5 | 2 | 4 | 3 | 3 | 2 | 1 |
| L+R-1 | 1 | 6 | 3 | 6 | 5 | 6 | 3 | 1 |

Max = 6.

```js
// tested
function lisEndingAt(a) {
  const dp = Array(a.length).fill(1);
  for (let i = 0; i < a.length; i++)
    for (let j = 0; j < i; j++)
      if (a[j] < a[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
  return dp;
}

function longestBitonic(a) {
  if (a.length === 0) return 0;
  const L = lisEndingAt(a);
  // longest decreasing starting at i == LIS ending at i of the reversed array
  const R = lisEndingAt([...a].reverse()).reverse();
  let best = 0;
  for (let i = 0; i < a.length; i++) best = Math.max(best, L[i] + R[i] - 1);
  return best;
}
```

**Complexity:** Time O(n^2), Space O(n). (Swap the helper for the tails method to get O(n log n) when you only need lengths per index; that needs a small tweak, so mention it only if asked.)
**Watch out:** Forgetting the `- 1`. Also, whether a pure increasing run is allowed depends on the problem statement. This version allows it. The mountain problem below does not.

### 9. Longest Arithmetic Subsequence — Medium
**Companies:** Google, Amazon, Meta
**Problem:** Return the length of the longest subsequence where consecutive differences are all equal. Example: `[9,4,7,2,10]` -> `3` (`4,7,10`).
**Intuition:** One index is not enough state. A chain ending at `i` also depends on its **difference**. So `dp[i][d]` = length of the longest arithmetic subsequence ending at `i` with difference `d`.
**Approach 1 - Brute force:** Try every pair as the first two items and extend greedily; or all subsequences. Time O(n^3) for the pair approach.
**Approach 2 - Better:** Same DP using a 2D array indexed by difference offset. Works if values are small (up to about 500 on LeetCode), costs O(n * range) memory.
**Approach 3 - Optimal:** One `Map` per index. For each pair `j < i`, `d = a[i] - a[j]`, then `dp[i][d] = dp[j][d] + 1` (default length 1 for `j`, so a pair gives 2).

Trace on `[9,4,7,2,10]`:

| pair (j,i) | d | dp[i][d] |
|------------|---|----------|
| (4,7) | 3 | 2 |
| (7,10) | 3 | dp[7][3] + 1 = 3 |
| (4,10) | 6 | 2 |

Max = 3.

```js
// tested
function longestArithSeqLength(a) {
  const n = a.length;
  if (n <= 2) return n;
  const dp = a.map(() => new Map());
  let best = 2;
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < i; j++) {
      const d = a[i] - a[j];
      const len = (dp[j].get(d) || 1) + 1;
      if (len > (dp[i].get(d) || 0)) dp[i].set(d, len);
      best = Math.max(best, len);
    }
  }
  return best;
}
```

**Complexity:** Time O(n^2), Space O(n^2) in the worst case.
**Watch out:** Differences can be negative or zero, so use a `Map`, not an array index. Any two elements already form a sequence of length 2, hence the default of 1 for `dp[j]` and the answer floor of 2. For each `i` we take the best over all `j` with the same `d`, which the `if` check handles.

### 10. Minimum Number of Removals to Make Mountain Array — Hard
**Companies:** Amazon, Google, Microsoft
**Problem:** A mountain array has length at least 3, with a peak strictly in the middle: values strictly increase up to the peak, then strictly decrease. Return the fewest removals to turn the given array into a mountain. Example: `[2,1,1,5,6,2,3,1]` -> `3` (keep `1,5,6,3,1`, length 5, so remove 3).
**Intuition:** Removing the fewest means keeping the **longest** mountain subsequence. That is the bitonic problem, with one rule: the peak needs at least one element on each side.
**Approach 1 - Brute force:** Try all subsets, check mountain. Time O(2^n * n).
**Approach 2 - Better:** For each peak, loop to find left and right best. Time O(n^3).
**Approach 3 - Optimal:** Compute `L[i]` and `R[i]` as in the bitonic problem. Only consider peaks with `L[i] > 1` **and** `R[i] > 1`. Answer is `n - max(L[i] + R[i] - 1)`.

Trace on `[2,1,1,5,6,2,3,1]`:

| i | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|---|
| a | 2 | 1 | 1 | 5 | 6 | 2 | 3 | 1 |
| L | 1 | 1 | 1 | 2 | 3 | 2 | 3 | 1 |
| R | 2 | 1 | 1 | 3 | 3 | 2 | 2 | 1 |

Valid peaks need `L>1` and `R>1`. Peak at index 3 (value 5): `2 + 3 - 1 = 4`. Peak at index 4 (value 6): `3 + 3 - 1 = 5`. Peak at index 6 (value 3): `3 + 2 - 1 = 4`. Best = 5, so remove `8 - 5 = 3`.

```js
// tested
function minimumMountainRemovals(a) {
  const n = a.length;
  const L = lisEndingAt(a);
  const R = lisEndingAt([...a].reverse()).reverse();
  let keep = 0;
  for (let i = 1; i < n - 1; i++) {
    if (L[i] > 1 && R[i] > 1) keep = Math.max(keep, L[i] + R[i] - 1);
  }
  return n - keep;
}
// lisEndingAt is the helper from problem 8
```

**Complexity:** Time O(n^2), Space O(n).
**Watch out:** Without the `L>1 && R>1` check you would count a pure ramp as a mountain, which is not allowed. The statement guarantees a mountain is always possible (n >= 3), so `keep` is at least 3.

### 11. Russian Doll Envelopes — Hard
**Companies:** Google, Amazon, Meta
**Problem:** Each envelope is `[w, h]`. One fits inside another only if **both** width and height are strictly smaller. Return the maximum number you can nest. Example: `[[5,4],[6,4],[6,7],[2,3]]` -> `3` (`[2,3] -> [5,4] -> [6,7]`).
**Intuition:** Two dimensions are hard. Sort by width to take care of one. Then you need strictly increasing heights, which is LIS on heights. One trap: envelopes with the **same width** must not nest. Sort equal widths by height **descending** so that LIS can never pick two of them (their heights go down, not up).
**Approach 1 - Brute force:** Try all subsets and check nesting. Time O(2^n).
**Approach 2 - Better:** Sort by width, then the O(n^2) LIS DP on heights with a width check. Time O(n^2), Space O(n). Too slow for n = 100,000.
**Approach 3 - Optimal:** Sort by `w` ascending, then `h` descending. Run tails + binary search on `h`.

Trace on `[[5,4],[6,4],[6,7],[2,3]]`. Sorted: `[2,3],[5,4],[6,7],[6,4]`. Heights: `3,4,7,4`.

| h | tails after |
|---|-------------|
| 3 | [3] |
| 4 | [3, 4] |
| 7 | [3, 4, 7] |
| 4 | [3, 4, 7] (4 replaces itself) |

Answer 3.

```js
// tested
function maxEnvelopes(envelopes) {
  const e = [...envelopes].sort((a, b) => a[0] - b[0] || b[1] - a[1]);
  const tails = [];
  for (const [, h] of e) {
    let lo = 0, hi = tails.length;
    while (lo < hi) {
      const m = (lo + hi) >> 1;
      if (tails[m] < h) lo = m + 1;
      else hi = m;
    }
    tails[lo] = h;
  }
  return tails.length;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** Sorting heights ascending for equal widths is the classic bug: `[[1,1],[1,1],[1,1]]` would wrongly nest. Descending order plus the strict lower bound fixes it. Do not use the O(n^2) DP on big inputs; it will time out.

---

## Common Mistakes

- **"A subsequence must be contiguous."** No. That is a subarray. A subsequence only keeps order.
- **"The answer is `dp[n-1]`."** No. `dp[i]` means "ends at i". The answer is the **max** over all `i`.
- **"The tails array is the LIS."** No. It only gives the correct length. To print the LIS you need parent pointers.
- **"Use `<=` in the binary search to be safe."** That changes the problem to non-decreasing. For strictly increasing use lower bound (`tails[m] < x` moves right).
- **"Greedy: just take the next bigger number."** It fails: `[1, 100, 2, 3, 4]` would give `1,100`, while the LIS is `1,2,3,4`.
- **Not sorting before a chain problem.** Divisible subset, string chain, pair chain and envelopes all need a sort first so earlier items can only precede later ones.
- **Equal widths in envelopes.** Sort equal widths by height descending, or you will nest envelopes that are not strictly bigger.
- **Forgetting `- 1` in bitonic.** The peak is in both `L[i]` and `R[i]`.

---

## Interview Tips

- Start by saying the state out loud: "`dp[i]` is the longest increasing subsequence **ending at index i**". It shows you understand why the answer is a max over all i.
- Give the O(n^2) solution first, then say "we can speed this up with binary search by keeping the smallest tail for each length". Then explain Fact 1 (sorted) and Fact 2 (smaller tails are better). This is the usual follow-up.
- If asked "print the sequence", add a `prev` array. If asked "print the sequence in O(n log n)", mention storing indices in the tails slots plus parent pointers.
- Ask: strictly increasing or non-decreasing? Distinct values? Equal values change the binary search from lower bound to upper bound.
- Edge cases: empty array, one element, all equal, strictly decreasing (answer 1), already sorted (answer n).
- For "Is this really LIS?" problems, say the reduction clearly: "after sorting by X, I need a strictly increasing sequence of Y".
- Know the link to LCS: the LIS of an array equals the LCS of the array and its sorted, de-duplicated copy. This is a nice trivia point, but the LIS code is faster.
- Complexity talk: O(n^2) is fine to about 5,000 elements; for 100,000 you need O(n log n).

---

## Practice List

**Easy**
- Fibonacci-style warmups from the 1D DP chapter (to confirm the DP habit)
- Monotonic Array (check increasing or decreasing)
- Longest Continuous Increasing Subsequence

**Medium**
- Longest Increasing Subsequence (both ways)
- Number of Longest Increasing Subsequence
- Largest Divisible Subset
- Longest String Chain
- Maximum Length of Pair Chain
- Longest Arithmetic Subsequence
- Increasing Triplet Subsequence
- Longest Bitonic Subsequence
- Wiggle Subsequence
- Longest Arithmetic Subsequence of Given Difference
- Minimum Number of Removals to Make Mountain Array

**Hard**
- Russian Doll Envelopes
- Longest Increasing Path in a Matrix
- Make Array Strictly Increasing
- Maximum Height by Stacking Cuboids
- Longest Increasing Subsequence II (with a max step, needs a segment tree)

---

## Key Takeaways

- **`dp[i]` means "best run ending at i"**, so the final answer is the max over all i, not the last cell.
- **The O(n^2) DP** checks every earlier smaller element. It is simple and extends easily to print, count and chain variants.
- **The tails array stores the smallest end for each length.** It stays sorted, and smaller ends are never worse, so binary search gives O(n log n).
- **Tails is not the sequence.** It gives the length only; use `prev` pointers to print.
- **Strict vs non-strict** is just lower bound vs upper bound (and `<` vs `<=` in the DP).
- **Sort first** for chain problems (divisible, string chain, pair chain, envelopes); for envelopes sort equal widths by height descending.
- **Mountain and bitonic** = LIS from the left + LIS from the right, combined at each peak with `L + R - 1`.

---

[← Previous: Stocks](../07-stocks/README.md) | [Next: Partition and MCM DP →](../09-partition-and-mcm-dp/README.md) | [Back to Dynamic Programming Index](../README.md)
