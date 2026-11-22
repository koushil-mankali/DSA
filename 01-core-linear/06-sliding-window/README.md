# Sliding Window

> **Estimated Time:** 5 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - Arrays](../../01-core-linear/01-arrays/README.md), [03 - Hashing](../../01-core-linear/03-hashing/README.md), [04 - Strings](../../01-core-linear/04-strings/README.md), [05 - Two Pointers](../../01-core-linear/05-two-pointers/README.md)

---

## What You'll Learn
- You can tell a **fixed-size** window problem from a **variable-size** one.
- You can write the "expand right, shrink left" loop from memory and explain why it is O(n).
- You can count subarrays with **exactly K** of something using `atMost(K) - atMost(K - 1)`.
- You can keep window state in a hash map, a 26-slot array, a running sum, or a monotonic deque.
- You can solve the classic hard ones: Minimum Window Substring, Sliding Window Maximum, Concatenation of All Words.

## The Big Idea

Think of looking at a long train through a small window of your car. You do not re-read the whole train at each moment. When the train moves one step, one carriage leaves your view and one new carriage enters. You only update what changed.

A sliding window is a **contiguous range** `[left, right]` of an array or string. Instead of recomputing the answer for every range from scratch (O(n^2) or worse), you **move the edges one step at a time** and update a small summary (a sum, a count map, a max). Each element enters the window once and leaves once, so the total work is O(n).

Interviewers love it because the brute force is obvious (try all subarrays) and the optimization is a clean, teachable idea. It also checks if you can pick the right state to keep (sum? map? deque?) and handle edge cases.

Two shapes:

| Shape | Window size | Goal |
|-------|-------------|------|
| Fixed | Always `k` | Compute something for every window of size `k` |
| Variable | Grows and shrinks | Find the longest or shortest window that is valid |

```
Fixed (k = 3):    [2 1 5] 1 3 2  ->  2 [1 5 1] 3 2  ->  2 1 [5 1 3] 2 ...
Variable:         l and r both move right. r always moves. l moves only when the window is bad (or, for "shortest", while it is good).
```

## How to Spot It

| If the problem says ... | think ... |
|-------------------------|-----------|
| "subarray / substring of size k" | Fixed window |
| "longest substring / subarray with at most K ..." | Variable window, shrink when invalid |
| "shortest / minimum subarray with sum >= target" | Variable window, shrink while valid |
| "contiguous" + "all elements positive" | Sum windows work because adding makes the sum bigger |
| "number of subarrays with exactly K ..." | `atMost(K) - atMost(K - 1)` |
| "contains all characters of t" | Need-count map plus a "missing" counter |
| "permutation / anagram of p inside s" | Fixed window of length `len(p)` with letter counts |
| "maximum in every window of size k" | Monotonic deque |
| "at most K replacements / flips / zeros" | Window where `bad count <= K` |

**When it does NOT work:** if the array has negative numbers and the question is about a sum (for example "longest subarray with sum = K"), adding an element can make the sum smaller, so shrinking the left side is not safe. Use prefix sums and a hash map instead (see the prefix sum chapter).

## Core Concepts / Template

#### Template 1: Fixed window

Build the first window, then slide: add the new element on the right, remove the old one on the left.

```js
function fixedWindow(nums, k) {
  let state = 0;                         // sum, count map, etc.
  for (let i = 0; i < k; i++) state += nums[i];   // first window
  let best = state;
  for (let i = k; i < nums.length; i++) {
    state += nums[i] - nums[i - k];      // add right, drop left
    best = Math.max(best, state);
  }
  return best;
}
```

#### Template 2: Variable window, "longest valid"

`right` always moves forward. If adding `nums[right]` breaks the rule, move `left` forward until the window is valid again. Then record the length.

```js
function longestValid(nums) {
  let left = 0, best = 0;
  for (let right = 0; right < nums.length; right++) {
    // 1. add nums[right] to the state
    while (/* window is invalid */ false) {
      // 2. remove nums[left] from the state
      left++;
    }
    best = Math.max(best, right - left + 1);   // 3. window is valid here
  }
  return best;
}
```

#### Template 3: Variable window, "shortest valid"

Here you shrink **while the window is still valid**, and record the length inside the loop.

```js
function shortestValid(nums) {
  let left = 0, best = Infinity;
  for (let right = 0; right < nums.length; right++) {
    // add nums[right] to the state
    while (/* window is valid */ false) {
      best = Math.min(best, right - left + 1);
      // remove nums[left] from the state
      left++;
    }
  }
  return best === Infinity ? 0 : best;
}
```

Memory hook: **longest -> record after the while loop; shortest -> record inside the while loop.**

#### Why is it O(n) with a nested loop?

`left` only moves right and never goes back. So across the whole run, `left` moves at most n steps and `right` moves n steps. The inner `while` does not restart for each `right`. Total is at most 2n moves.

#### The "at most K" trick for exactly K

Counting subarrays with **exactly** K of something is hard directly, because the window is not monotonic: making it bigger can break "exactly K" and then fix it again. But "**at most** K" is monotonic: if a window has at most K, any smaller window inside it also has at most K. So the standard shrinking window works.

```
exactly(K) = atMost(K) - atMost(K - 1)
```

Why it works: `atMost(K)` counts windows with 0, 1, ..., K. `atMost(K - 1)` counts windows with 0, ..., K-1. The difference is the windows with exactly K.

How to count in `atMost(g)`: for each `right`, after shrinking, **every** start between `left` and `right` gives a valid subarray ending at `right`. That is `right - left + 1` subarrays.

```js
function exactlyK(nums, K, atMost) {      // atMost(nums, g) is your window function
  return atMost(nums, K) - atMost(nums, K - 1);
}
```

Always guard `g < 0` (return 0), or you will get wrong answers when K = 0.

#### Trace: longest valid window, "at most 2 distinct", s = "eceba"

| right | char | window | distinct | action | best |
|-------|------|--------|----------|--------|------|
| 0 | e | `e` | 1 | ok | 1 |
| 1 | c | `ec` | 2 | ok | 2 |
| 2 | e | `ece` | 2 | ok | 3 |
| 3 | b | `eceb` | 3 | shrink: drop e, then c -> `eb` | 3 |
| 4 | a | `eba` | 3 | shrink: drop e, then b -> `ba` | 3 |

Answer 3 (`ece`).

## Problems (easy -> hard)

### 1. Maximum Sum Subarray of Size K — Easy
**Companies:** Amazon, Microsoft, Google
**Problem:** Given an array and `k`, return the largest sum of any `k` consecutive elements. Example: `[2,1,5,1,3,2], k=3 -> 9` (the window `5,1,3`).
**Intuition:** When the window moves one step, the sum only changes by two numbers: the new one in, the old one out. No need to add `k` numbers again.
**Approach 1 - Brute force:** For each start, add `k` numbers. Time O(n*k), Space O(1).
**Approach 2 - Optimal:** Fixed window. Sum the first `k`, then slide.

| window | sum |
|--------|-----|
| `2 1 5` | 8 |
| `1 5 1` | 8 - 2 + 1 = 7 |
| `5 1 3` | 7 - 1 + 3 = 9 |
| `1 3 2` | 9 - 5 + 2 = 6 |

```js
function maxSumSubarray(nums, k) {
  if (k <= 0 || k > nums.length) return 0;
  let sum = 0;
  for (let i = 0; i < k; i++) sum += nums[i];
  let best = sum;
  for (let i = k; i < nums.length; i++) {
    sum += nums[i] - nums[i - k];
    best = Math.max(best, sum);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Do not start `best` at 0; with all negative numbers the answer is negative. Handle `k > n`.

### 2. Longest Substring Without Repeating Characters — Medium
**Companies:** Amazon, Google, Meta, Microsoft, Bloomberg
**Problem:** Return the length of the longest substring with all distinct characters. Example: `"abcabcbb" -> 3` (`abc`).
**Intuition:** Keep a window with no repeats. When a new character is already inside, jump `left` to just after its previous position. You do not need to move one step at a time.
**Approach 1 - Brute force:** Check every substring with a set. Time O(n^2) to O(n^3), Space O(min(n, alphabet)).
**Approach 2 - Better:** Window with a set, shrink one by one while the char is a duplicate. Time O(n).
**Approach 3 - Optimal:** Store the **last index** of each character. If `last[c] >= left`, set `left = last[c] + 1`.

Trace on `"abba"`:

| right | c | last[c] before | left | window | best |
|-------|---|----------------|------|--------|------|
| 0 | a | - | 0 | `a` | 1 |
| 1 | b | - | 0 | `ab` | 2 |
| 2 | b | 1 (>= 0) | 2 | `b` | 2 |
| 3 | a | 0 (< left 2, ignore) | 2 | `ba` | 2 |

```js
function lengthOfLongestSubstring(s) {
  const last = new Map();
  let left = 0, best = 0;
  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    if (last.has(c) && last.get(c) >= left) left = last.get(c) + 1;
    last.set(c, right);
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(min(n, alphabet size)).
**Watch out:** The check `last.get(c) >= left` is required. In `"abba"`, the old `a` at index 0 is outside the window, and without the check `left` would go backwards.

### 3. Minimum Size Subarray Sum — Medium
**Companies:** Amazon, Meta, Microsoft, Google
**Problem:** Given positive integers and a `target`, return the length of the shortest subarray with sum `>= target`, or 0. Example: `target=7, [2,3,1,2,4,3] -> 2` (`4,3`).
**Intuition:** All numbers are positive, so adding grows the sum and removing shrinks it. Grow until the sum is enough, then shrink as much as you can while it stays enough. This is the "shortest valid" template.
**Approach 1 - Brute force:** Try all subarrays. Time O(n^2), Space O(1).
**Approach 2 - Better:** Prefix sums plus binary search for each start. Time O(n log n), Space O(n).
**Approach 3 - Optimal:** Variable window, shrink while valid.

Trace on `[2,3,1,2,4,3]`, target 7:

| right | sum | shrink steps | best |
|-------|-----|--------------|------|
| 0..2 | 2, 5, 6 | none (< 7) | inf |
| 3 | 8 | record 4, drop 2 -> 6 | 4 |
| 4 | 10 | record 4, drop 3 -> 7, record 3 (`1 2 4`), drop 1 -> 6 | 3 |
| 5 | 9 | record 3 (`2 4 3`), drop 2 -> 7, record 2 (`4 3`), drop 4 -> 3 | 2 |

```js
function minSubArrayLen(target, nums) {
  let left = 0, sum = 0, best = Infinity;
  for (let right = 0; right < nums.length; right++) {
    sum += nums[right];
    while (sum >= target) {
      best = Math.min(best, right - left + 1);
      sum -= nums[left++];
    }
  }
  return best === Infinity ? 0 : best;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** This only works for non-negative numbers. Return 0, not Infinity, when nothing fits.

### 4. Max Consecutive Ones III — Medium
**Companies:** Meta, Amazon, Google, Microsoft
**Problem:** Binary array `nums` and `k`. You may flip at most `k` zeros to ones. Return the longest run of ones. Example: `[1,1,1,0,0,0,1,1,1,1,0], k=2 -> 6`.
**Intuition:** Rephrase: find the longest window that contains **at most k zeros**. Flipping is just a story. The window is valid while `zeros <= k`.
**Approach 1 - Brute force:** For every start, extend while zeros <= k. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** Variable window, count zeros. If `zeros > k`, move `left` until a zero leaves.

Trace (k = 2) on `[1,1,1,0,0,0,1,1,1,1,0]`:

| right | zeros | left | length |
|-------|-------|------|--------|
| 0-2 | 0 | 0 | 3 |
| 3 | 1 | 0 | 4 |
| 4 | 2 | 0 | 5 |
| 5 | 3, shrink until the zero at 3 leaves | 4 | 2 |
| 6-9 | 2 | 4 | up to 6 (indices 4..9) |
| 10 | 3, shrink until the zero at 4 leaves | 5 | 6 (indices 5..10) |

Best is 6.

```js
function longestOnes(nums, k) {
  let left = 0, zeros = 0, best = 0;
  for (let right = 0; right < nums.length; right++) {
    if (nums[right] === 0) zeros++;
    while (zeros > k) {
      if (nums[left] === 0) zeros--;
      left++;
    }
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** `k = 0` and all zeros gives 0. The window can become empty (`left = right + 1`), so length 0 is fine.

### 5. Longest Repeating Character Replacement — Medium
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** String of uppercase letters, up to `k` replacements. Return the longest substring that can be made of one repeated letter. Example: `"AABABBA", k=1 -> 4`.
**Intuition:** In a window, keep the most frequent letter and replace the rest. Replacements needed = `window length - maxFreq`. The window is valid if that is `<= k`.
**Approach 1 - Brute force:** For each substring count frequencies. Time O(n^2 * 26), Space O(26).
**Approach 2 - Better:** Window, and recompute max frequency (26 slots) when checking validity. Time O(26n).
**Approach 3 - Optimal:** Keep `maxFreq` as a "historical max" and **never decrease it**. The window never shrinks, it only slides (when invalid, move `left` once).

Why the stale `maxFreq` is OK: the answer only grows if we find a window with a **larger** real max frequency. A stale high value just keeps the window size from shrinking, it cannot produce a wrong larger answer, because the window only grows when `maxFreq` truly increases.

Trace `"AABABBA"`, k = 1:

| right | char | counts | maxFreq | len - maxFreq | action | window len |
|-------|------|--------|---------|---------------|--------|------------|
| 0 | A | A1 | 1 | 0 | - | 1 |
| 1 | A | A2 | 2 | 0 | - | 2 |
| 2 | B | A2 B1 | 2 | 1 | - | 3 |
| 3 | A | A3 B1 | 3 | 1 | - | 4 |
| 4 | B | A3 B2 | 3 | 2 > 1 | left++ | 4 |
| 5 | B | A2 B3 (after the last drop) | 3 | 2 > 1 | left++ | 4 |

```js
function characterReplacement(s, k) {
  const cnt = new Array(26).fill(0);
  let left = 0, maxFreq = 0, best = 0;
  for (let right = 0; right < s.length; right++) {
    const i = s.charCodeAt(right) - 65;
    maxFreq = Math.max(maxFreq, ++cnt[i]);
    if (right - left + 1 - maxFreq > k) {   // shrink by one only
      cnt[s.charCodeAt(left) - 65]--;
      left++;
    }
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(26) = O(1).
**Watch out:** Use `if`, not `while`, with the stale-maxFreq trick. If you want the plain version, use `while` and recompute the max each time (slower but easier to prove).

### 6. Fruit Into Baskets — Medium
**Companies:** Google, Amazon, Meta
**Problem:** `fruits[i]` is a tree type. Two baskets, each holds one type, unlimited count. Start anywhere, walk right picking one fruit from every tree, stop when you meet a third type. Return the max fruits. Example: `[1,2,3,2,2] -> 4`.
**Intuition:** This is a disguised "longest subarray with at most 2 distinct values".
**Approach 1 - Brute force:** For every start, walk until a third type. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** Window with a count map. When the map has 3 types, shrink from the left until one type disappears.

Trace `[1,2,3,2,2]`:

| right | window | map | action |
|-------|--------|-----|--------|
| 0 | `1` | {1:1} | len 1 |
| 1 | `1 2` | {1:1,2:1} | len 2 |
| 2 | `1 2 3` | 3 types | drop 1 -> `2 3`, len 2 |
| 3 | `2 3 2` | {2:2,3:1} | len 3 |
| 4 | `2 3 2 2` | {2:3,3:1} | len 4 |

```js
function totalFruit(fruits) {
  const cnt = new Map();
  let left = 0, best = 0;
  for (let right = 0; right < fruits.length; right++) {
    cnt.set(fruits[right], (cnt.get(fruits[right]) || 0) + 1);
    while (cnt.size > 2) {
      const f = fruits[left++];
      cnt.set(f, cnt.get(f) - 1);
      if (cnt.get(f) === 0) cnt.delete(f);   // must delete, or size never drops
    }
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1) (at most 3 keys).
**Watch out:** Delete the key when its count hits 0. Checking `cnt.size` with zero-count keys still in the map is the classic bug.

### 7. Longest Substring with At Most K Distinct Characters — Medium
**Companies:** Google, Amazon, Meta, Microsoft, Uber
**Problem:** Return the length of the longest substring with at most `k` distinct characters. Example: `"eceba", k=2 -> 3`.
**Intuition:** Same as problem 6 with `2` replaced by `k`. This is the **general form** of the "at most K distinct" window. (On LeetCode this is a premium problem, search "Longest Substring with At Most K Distinct Characters".)
**Approach 1 - Brute force:** All substrings with a set. Time O(n^2), Space O(k).
**Approach 2 - Optimal:** Variable window with a count map. Shrink while `map.size > k`. Trace is shown in the Core Concepts section above.

```js
function lengthOfLongestSubstringKDistinct(s, k) {
  if (k === 0) return 0;
  const cnt = new Map();
  let left = 0, best = 0;
  for (let right = 0; right < s.length; right++) {
    cnt.set(s[right], (cnt.get(s[right]) || 0) + 1);
    while (cnt.size > k) {
      const c = s[left++];
      cnt.set(c, cnt.get(c) - 1);
      if (cnt.get(c) === 0) cnt.delete(c);
    }
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(k).
**Watch out:** Return 0 early for `k = 0`, and for an empty string. Delete zero-count keys, or `map.size` lies.

### 8. Binary Subarrays With Sum — Medium
**Companies:** Google, Amazon, Meta
**Problem:** Binary array `nums` and `goal`. Count subarrays whose sum equals `goal`. Example: `[1,0,1,0,1], goal=2 -> 4`.
**Intuition:** "Exactly goal" is not monotonic, but "at most goal" is (all numbers are 0 or 1, so the sum never goes down when you grow). Use `atMost(goal) - atMost(goal - 1)`.
**Approach 1 - Brute force:** All subarrays with running sum. Time O(n^2), Space O(1).
**Approach 2 - Better:** Prefix sum + hash map: count previous prefixes equal to `prefix - goal`. Time O(n), Space O(n).
**Approach 3 - Optimal:** Two sliding windows through `atMost`. Space O(1).

Trace of `atMost(g)` for `[1,0,1,0,1]`:

| right | sum (after shrink, g=2) | left | add (right-left+1) | g=1 add |
|-------|-------------------------|------|--------------------|---------|
| 0 | 1 | 0 | 1 | 1 |
| 1 | 1 | 0 | 2 | 2 |
| 2 | 2 | 0 | 3 | left->1, sum 1, add 2 |
| 3 | 2 | 0 | 4 | add 3 |
| 4 | 3 -> shrink to 2, left=1 | 1 | 4 | sum 2->1 left=3, add 2 |

`atMost(2)` = 1+2+3+4+4 = 14. `atMost(1)` = 1+2+2+3+2 = 10. Answer 4.

```js
function numSubarraysWithSum(nums, goal) {
  const atMost = (g) => {
    if (g < 0) return 0;               // goal = 0 makes g - 1 = -1
    let left = 0, sum = 0, count = 0;
    for (let right = 0; right < nums.length; right++) {
      sum += nums[right];
      while (sum > g) sum -= nums[left++];
      count += right - left + 1;
    }
    return count;
  };
  return atMost(goal) - atMost(goal - 1);
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** `goal = 0` needs the `g < 0` guard. Otherwise `while (sum > -1)` shrinks forever past the array start.

### 9. Count Number of Nice Subarrays — Medium
**Companies:** Google, Amazon, Meta
**Problem:** Count subarrays that contain exactly `k` odd numbers. Example: `[1,1,2,1,1], k=3 -> 2`.
**Intuition:** Turn each number into 1 (odd) or 0 (even). Now it is exactly problem 8 with `goal = k`.
**Approach 1 - Brute force:** All subarrays, count odds. Time O(n^2), Space O(1).
**Approach 2 - Better:** Prefix count of odds in a hash map. Time O(n), Space O(n).
**Approach 3 - Optimal:** `atMost(k) - atMost(k - 1)` with an odd counter.

```js
function numberOfSubarrays(nums, k) {
  const atMost = (g) => {
    if (g < 0) return 0;
    let left = 0, odd = 0, count = 0;
    for (let right = 0; right < nums.length; right++) {
      odd += nums[right] & 1;
      while (odd > g) odd -= nums[left++] & 1;
      count += right - left + 1;
    }
    return count;
  };
  return atMost(k) - atMost(k - 1);
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** `x & 1` works for non-negative integers. For possibly negative numbers in JS, `-3 & 1` is `1` (two's complement), so it is still fine, but `x % 2` would give `-1`. Prefer `&`.

### 10. Number of Substrings Containing All Three Characters — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** String of only `a`, `b`, `c`. Count substrings that contain at least one of each. Example: `"abcabc" -> 10`.
**Intuition:** Once a window `[left, right]` has all three letters, every longer window with the same `left` (extending to the end) also has them. So for each valid `left`, add `n - right` substrings. Or, simpler: for each `right`, the valid starts are all indices up to `min(lastA, lastB, lastC)`, which is `min + 1` substrings.
**Approach 1 - Brute force:** All substrings with a set. Time O(n^2), Space O(1).
**Approach 2 - Better:** Window with counts: when all three are present, add `n - right`, then shrink.
**Approach 3 - Optimal:** Track the last seen index of each letter. At each `right`, add `min(last) + 1`. One pass, no inner loop.

Trace `"abcabc"` (last = [a,b,c], starts at [-1,-1,-1]):

| i | char | last | min+1 | total |
|---|------|------|-------|-------|
| 0 | a | [0,-1,-1] | 0 | 0 |
| 1 | b | [0,1,-1] | 0 | 0 |
| 2 | c | [0,1,2] | 1 | 1 |
| 3 | a | [3,1,2] | 2 | 3 |
| 4 | b | [3,4,2] | 3 | 6 |
| 5 | c | [3,4,5] | 4 | 10 |

```js
function numberOfSubstrings(s) {
  const last = [-1, -1, -1];
  let total = 0;
  for (let i = 0; i < s.length; i++) {
    last[s.charCodeAt(i) - 97] = i;
    total += Math.min(...last) + 1;      // starts 0..min(last) are all valid
  }
  return total;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** If a letter has not appeared, `last` is -1, so `min + 1 = 0`. Good. The result can be up to about n^2 / 2, which fits easily in a JS number for LeetCode limits.

### 11. Permutation in String — Medium
**Companies:** Microsoft, Amazon, Google, Meta
**Problem:** Return true if `s2` contains a permutation of `s1` as a substring. Example: `s1="ab", s2="eidbaooo" -> true` (`ba`).
**Intuition:** A permutation has the same letters with the same counts. So look at every window of length `len(s1)` in `s2` and compare letter counts. It is a fixed window.
**Approach 1 - Brute force:** Sort each window and compare. Time O(n * m log m), Space O(m).
**Approach 2 - Better:** Fixed window, compare two 26-arrays every step. Time O(26n).
**Approach 3 - Optimal:** Keep a `mismatch` counter: how many of the 26 letters have different counts. Each slide changes at most two letters, so update `mismatch` in O(1). Window matches when `mismatch === 0`.

```js
function checkInclusion(s1, s2) {
  const m = s1.length;
  if (m > s2.length) return false;
  const need = new Array(26).fill(0), win = new Array(26).fill(0);
  for (const c of s1) need[c.charCodeAt(0) - 97]++;
  let mismatch = 0;
  for (let i = 0; i < 26; i++) if (need[i] !== 0) mismatch++;   // win is all zeros
  const update = (i, d) => {
    if (win[i] === need[i]) mismatch++;      // was equal, about to change
    win[i] += d;
    if (win[i] === need[i]) mismatch--;      // became equal
  };
  for (let r = 0; r < s2.length; r++) {
    update(s2.charCodeAt(r) - 97, 1);
    if (r >= m) update(s2.charCodeAt(r - m) - 97, -1);   // drop the left char
    if (mismatch === 0) return true;
  }
  return false;
}
```

**Complexity:** Time O(n + 26), Space O(26).
**Watch out:** The letter that enters and the letter that leaves can be the same letter. The `update` function handles that because it processes them one after another. Handle `len(s1) > len(s2)` first.

### 12. Find All Anagrams in a String — Medium
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** Return all start indices in `s` where an anagram of `p` begins. Example: `s="cbaebabacd", p="abc" -> [0,6]`.
**Intuition:** Same as problem 11, but collect every match instead of stopping at the first.
**Approach 1 - Brute force:** Sort each window. Time O(n * m log m).
**Approach 2 - Optimal:** Fixed window with the `mismatch` counter.

Trace idea: window of 3 at index 0 is `cba` (counts equal to `abc`, record 0). At index 6 it is `bac`, record 6. Everything between has a different count.

```js
function findAnagrams(s, p) {
  const m = p.length, res = [];
  if (m > s.length) return res;
  const need = new Array(26).fill(0), win = new Array(26).fill(0);
  let mismatch = 0;
  for (const c of p) need[c.charCodeAt(0) - 97]++;
  for (let i = 0; i < 26; i++) if (need[i]) mismatch++;
  const update = (i, d) => {
    if (win[i] === need[i]) mismatch++;
    win[i] += d;
    if (win[i] === need[i]) mismatch--;
  };
  for (let r = 0; r < s.length; r++) {
    update(s.charCodeAt(r) - 97, 1);
    if (r >= m) update(s.charCodeAt(r - m) - 97, -1);
    if (mismatch === 0) res.push(r - m + 1);     // start index of the window
  }
  return res;
}
```

**Complexity:** Time O(n), Space O(26) plus output.
**Watch out:** Overlapping matches are allowed (`"abab", "ab" -> [0,1,2]`). Return the **start** index, not the end.

### 13. Subarrays With K Different Integers — Hard
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Count subarrays with **exactly** `k` distinct integers. Example: `[1,2,1,2,3], k=2 -> 7`.
**Intuition:** The poster problem for the "at most K" trick. Exactly-K windows are not monotonic, so count `atMost(k) - atMost(k - 1)`.
**Approach 1 - Brute force:** For each start, grow with a set and count when size equals k. Time O(n^2), Space O(n).
**Approach 2 - Better:** One window plus a second "left2" pointer to the leftmost start of a valid window (needs care). Same O(n) but easy to get wrong.
**Approach 3 - Optimal:** Two calls of `atMost`.

Counts for `[1,2,1,2,3]`: `atMost(2)` = 1 + 2 + 3 + 4 + 2 = 12 (for right = 4, window shrinks to `[2,3]`). `atMost(1)` = 1+1+1+1+1 = 5. Answer 12 - 5 = 7.

```js
function subarraysWithKDistinct(nums, k) {
  const atMost = (g) => {
    const cnt = new Map();
    let left = 0, count = 0;
    for (let right = 0; right < nums.length; right++) {
      cnt.set(nums[right], (cnt.get(nums[right]) || 0) + 1);
      while (cnt.size > g) {
        const x = nums[left++];
        cnt.set(x, cnt.get(x) - 1);
        if (cnt.get(x) === 0) cnt.delete(x);
      }
      count += right - left + 1;        // every start in [left, right] works
    }
    return count;
  };
  return atMost(k) - atMost(k - 1);
}
```

**Complexity:** Time O(n), Space O(k).
**Watch out:** With `k = 1`, `atMost(0)` shrinks to an empty window each time, adds 0. That works because `right - left + 1 = 0` after `left = right + 1`. Delete zero-count keys.

### 14. Minimum Window Substring — Hard
**Companies:** Meta, Amazon, Google, Microsoft, Uber
**Problem:** Given `s` and `t`, return the smallest substring of `s` containing every character of `t` (with multiplicity). Return `""` if none. Example: `s="ADOBECODEBANC", t="ABC" -> "BANC"`.
**Intuition:** Expand `right` until the window covers `t`. Then shrink `left` as far as it stays covering. That is the shortest window ending near this `right`. Remember the best, then continue.
**Approach 1 - Brute force:** Check every substring against `t`. Time O(n^2 * m), Space O(m).
**Approach 2 - Better:** Window with need map, but compare maps every step. Time O(n * 52).
**Approach 3 - Optimal:** A `need` map (can go negative for extra letters) and a `missing` counter: how many characters from `t` are still not covered. The window is valid when `missing === 0`. Each add or remove updates `missing` in O(1).

Trace `s="ADOBECODEBANC"`, `t="ABC"` (need A1 B1 C1, missing 3):

| right | char | missing | action | best |
|-------|------|---------|--------|------|
| 0 A | A | 2 | - | - |
| 3 B | B | 1 | - | - |
| 5 C | C | 0 | valid `ADOBEC` (6). shrink: drop A -> missing 1 | `ADOBEC` |
| 9 B | B (extra) | 1 | - | - |
| 10 A | A | 0 | valid `DOBECODEBA`; shrink D, O, B (extra), E, C... stop when C leaves | ... |
| 12 C | C | 0 | valid `CODEBANC`; shrink to `BANC` (4) | `BANC` |

```js
function minWindow(s, t) {
  if (t.length > s.length) return "";
  const need = new Map();
  for (const c of t) need.set(c, (need.get(c) || 0) + 1);
  let missing = t.length, left = 0, bestL = 0, bestLen = Infinity;
  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    if (need.has(c)) {
      if (need.get(c) > 0) missing--;        // this char was still needed
      need.set(c, need.get(c) - 1);          // may go negative (extra copy)
    }
    while (missing === 0) {                  // valid: try to shrink
      if (right - left + 1 < bestLen) { bestLen = right - left + 1; bestL = left; }
      const d = s[left++];
      if (need.has(d)) {
        need.set(d, need.get(d) + 1);
        if (need.get(d) > 0) missing++;      // we just lost a needed char
      }
    }
  }
  return bestLen === Infinity ? "" : s.slice(bestL, bestL + bestLen);
}
```

**Complexity:** Time O(n + m), Space O(m) (distinct chars in `t`).
**Watch out:** Duplicates in `t` (`t = "aa"`). Only decrement `missing` when `need[c] > 0` before the decrement. Characters not in `t` must not touch the map. Record the best **inside** the while loop.

### 15. Sliding Window Maximum — Hard
**Companies:** Amazon, Google, Meta, Microsoft, Uber
**Problem:** Return the maximum of every window of size `k`. Example: `[1,3,-1,-3,5,3,6,7], k=3 -> [3,3,5,5,6,7]`.
**Intuition:** The max cannot be maintained with one number, because when the max leaves the window you need the next biggest. Keep a **deque of indices** whose values are in decreasing order. The front is always the max. A smaller element behind a bigger newer one can never be the max again (the newer one stays longer), so drop it.
**Approach 1 - Brute force:** Scan each window. Time O(n*k), Space O(1).
**Approach 2 - Better:** Max-heap with lazy deletion (pop top while its index is outside the window). Time O(n log n), Space O(n).
**Approach 3 - Optimal:** Monotonic deque. Each index is pushed once and popped once.

Rules at each `i`: (1) pop from the back while `nums[back] <= nums[i]`; (2) push `i`; (3) if the front index is `<= i - k`, drop it; (4) if `i >= k - 1`, the answer is `nums[front]`.

Trace on `[1,3,-1,-3,5,3,6,7]`, k = 3 (deque shows values):

| i | num | deque (values) | output |
|---|-----|----------------|--------|
| 0 | 1 | [1] | - |
| 1 | 3 | [3] (1 popped) | - |
| 2 | -1 | [3,-1] | 3 |
| 3 | -3 | [3,-1,-3] | 3 |
| 4 | 5 | [5] (everything smaller popped, front 3 also gone) | 5 |
| 5 | 3 | [5,3] | 5 |
| 6 | 6 | [6] | 6 |
| 7 | 7 | [7] | 7 |

```js
function maxSlidingWindow(nums, k) {
  const dq = [];          // indices; values decreasing front -> back
  let head = 0;           // array + head pointer avoids slow shift()
  const res = [];
  for (let i = 0; i < nums.length; i++) {
    while (dq.length > head && nums[dq[dq.length - 1]] <= nums[i]) dq.pop();
    dq.push(i);
    if (dq[head] <= i - k) head++;           // front left the window
    if (i >= k - 1) res.push(nums[dq[head]]);
  }
  return res;
}
```

**Complexity:** Time O(n), Space O(k) for the deque (the array here may hold up to n indices because we only advance `head`; use a real deque if memory matters).
**Watch out:** Store **indices**, not values, so you can tell when the front expired. Do not use `Array.shift()` in a hot loop: it is O(n). Pop on `<=` (not `<`) to keep the deque strictly decreasing and small.

### 16. Substring with Concatenation of All Words — Hard
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Given `s` and an array `words` (all the same length), return start indices of substrings that are a concatenation of every word exactly once, in any order. Example: `s="barfoothefoobarman", words=["foo","bar"] -> [0,9]`.
**Intuition:** It is the anagram problem, but the "letters" are whole words. Since all words have length `w`, run a window that moves **`w` characters at a time**. Do this for each of the `w` possible starting offsets (0 to `w-1`) so every alignment is covered.
**Approach 1 - Brute force:** For each start, cut `n` pieces of length `w` and compare counts. Time O(len(s) * n * w), Space O(n*w).
**Approach 2 - Optimal:** For each offset, slide a window of words. Keep a count map of words in the window. If the new word is unknown, clear the window and restart after it. If its count is too high, shrink from the left, word by word, until fine. When the window holds exactly `n` words, record `left`.

Trace on `s="barfoothefoobarman"`, words `foo, bar`, offset 0 (words of 3: `bar foo the foo bar man`):

| step | word | window words | action |
|------|------|--------------|--------|
| r=0 | bar | bar | - |
| r=3 | foo | bar foo | count 2 = n, record 0, drop `bar` |
| r=6 | the | (unknown) | clear, left=9 |
| r=9 | foo | foo | - |
| r=12 | bar | foo bar | record 9, drop foo |
| r=15 | man | unknown | clear |

```js
function findSubstring(s, words) {
  const res = [];
  if (!words.length) return res;
  const w = words[0].length, n = words.length;
  if (s.length < w * n) return res;
  const need = new Map();
  for (const x of words) need.set(x, (need.get(x) || 0) + 1);
  for (let off = 0; off < w; off++) {
    const win = new Map();
    let left = off, used = 0;
    for (let r = off; r + w <= s.length; r += w) {
      const word = s.slice(r, r + w);
      if (!need.has(word)) { win.clear(); used = 0; left = r + w; continue; }
      win.set(word, (win.get(word) || 0) + 1);
      used++;
      while (win.get(word) > need.get(word)) {      // too many copies: shrink
        const lw = s.slice(left, left + w);
        win.set(lw, win.get(lw) - 1);
        used--;
        left += w;
      }
      if (used === n) {                              // exact match
        res.push(left);
        const lw = s.slice(left, left + w);          // slide by one word
        win.set(lw, win.get(lw) - 1);
        used--;
        left += w;
      }
    }
  }
  return res;
}
```

**Complexity:** Time O(len(s) * w) (each offset does O(len(s)/w) steps, each slice costs O(w)), Space O(n * w).
**Watch out:** Duplicate words in `words` (use counts, not a set). The result order is by offset, not sorted; LeetCode accepts any order. Do not forget the offset loop, or you miss matches that do not start at multiples of `w`.

## Common Mistakes

- **"Sliding window works on any array."** It needs a monotonic rule: growing the window moves a quantity one way. With negative numbers and a sum target, it breaks. Use prefix sums.
- **Using `if` where `while` is needed.** After adding one element you may need to remove several from the left. (The one exception is the stale-`maxFreq` trick in problem 5.)
- **Recording the answer in the wrong place.** Longest: after the shrink loop. Shortest: inside the shrink loop.
- **Forgetting to delete zero-count map keys** when you use `map.size` as the distinct count.
- **Off-by-one in length.** The window `[left, right]` has `right - left + 1` elements.
- **Not guarding `atMost(-1)`.** For `K = 0`, return 0 for negative limits.
- **Thinking exactly K can be done with one plain window.** It cannot; use `atMost(K) - atMost(K - 1)`.
- **Using `Array.shift()` as a queue** inside the loop. It is O(n) per call. Use indices or a head pointer.
- **Not saying that O(n) holds although there are two loops.** Explain that `left` never moves back.

## Interview Tips

- Say it first: "The subarray is contiguous, so I can use a sliding window. I need a rule that tells me when the window is valid."
- Name the state out loud: "I keep a count map / a running sum / a deque, and update it when an element enters or leaves."
- State the invariant: "After the shrink loop, the window `[left, right]` is valid."
- Explain the complexity: "Each element enters once and leaves once, so it is O(n), even with the inner while."
- Start with brute force (all subarrays, O(n^2)), then show where the repeated work is.
- Follow-ups interviewers ask:
  - "What if there are negative numbers?" -> prefix sum + hash map.
  - "What if the input is a stream?" -> window works online; keep only the state, not the array.
  - "Can you return the window itself, not just the length?" -> remember `bestL`.
  - "Exactly K instead of at most K?" -> subtract two at-most calls.
  - "What if the alphabet is Unicode?" -> use a Map instead of a 26-array.
- Edge cases to mention: empty input, `k > n`, `k = 0`, all elements equal, all elements invalid, the answer is the whole array, duplicates in the target (`t = "aa"`).

## Practice List

**Easy**
- Maximum Average Subarray I
- Contains Duplicate II
- Maximum Number of Vowels in a Substring of Given Length
- Minimum Recolors to Get K Consecutive Black Blocks

**Medium**
- Longest Substring with At Most Two Distinct Characters
- Longest Subarray of 1's After Deleting One Element
- Max Consecutive Ones II
- Maximum Points You Can Obtain from Cards
- Replace the Substring for Balanced String
- Get Equal Substrings Within Budget
- Subarray Product Less Than K
- Frequency of the Most Frequent Element
- Count Subarrays With Score Less Than K
- Maximum Erasure Value
- Grumpy Bookstore Owner
- Longest Turbulent Subarray
- Minimum Swaps to Group All 1's Together
- Count Vowel Substrings of a String

**Hard**
- Minimum Window Subsequence
- Shortest Subarray with Sum at Least K
- Sliding Window Median
- Count Subarrays Where Max Element Appears at Least K Times
- Minimum Number of K Consecutive Bit Flips
- Constrained Subsequence Sum

## Key Takeaways

- A **sliding window** is a contiguous range `[left, right]` where each element enters once and leaves once, giving **O(n)** time.
- **Fixed window:** add the new element, remove the old one, and update the answer.
- **Variable window:** `right` always moves; `left` moves to restore validity. **Longest -> record after shrinking; shortest -> record while shrinking.**
- **Exactly K = atMost(K) - atMost(K - 1)**, because "at most" is monotonic and "exactly" is not.
- Choose state to match the question: a **sum**, a **count map**, a **26-letter array with a mismatch counter**, or a **monotonic deque of indices** for window max.
- Sliding window needs a **monotonic validity rule**; with negative numbers and sum targets, switch to **prefix sums + hash map**.
- Guard the usual traps: **delete zero-count keys, handle `K = 0`, and use `right - left + 1`** for the length.

---

[← Previous: Two Pointers](../../01-core-linear/05-two-pointers/README.md) | [Next: Prefix Sum →](../../01-core-linear/07-prefix-sum/README.md) | [Back to Core Linear Index](../README.md)
