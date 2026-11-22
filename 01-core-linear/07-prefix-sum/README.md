# Prefix Sum and Difference Arrays

> **Estimated Time:** 4 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - Arrays](../01-arrays/README.md), [03 - Hashing](../03-hashing/README.md)

---

## What You'll Learn

- You can build a prefix sum array and answer any range-sum query in O(1).
- You can turn "subarray with property X" problems into "two prefix values that match" problems using a hash map.
- You can use a difference array to apply many range updates in O(n + q) instead of O(n * q).
- You can build 2D prefix sums and use the inclusion-exclusion rule for rectangle sums.
- You can handle the tricky parts: negative numbers, negative modulo in JavaScript, and the "empty prefix" starting value.

## The Big Idea

Think of a road with milestone markers. Each marker shows how far you are from the start. If someone asks "how far is it from marker 3 to marker 8?", you do not walk the road. You just subtract: marker 8 minus marker 3. The markers are your prefix sums.

A **prefix sum** array stores the running total up to each position. You pay O(n) once to build it. After that, the sum of any range is one subtraction. The **difference array** is the mirror image: instead of asking many range questions, you make many range updates. You record only the "edges" of each update, and one prefix sum at the end rebuilds the full array.

Why interviewers love it: it is a small idea that turns an O(n^2) brute force into O(n). It also hides in many disguises (counts, remainders, 0/1 balance, XOR, bookings, trips). If you see the disguise, the problem becomes easy.

```
nums = [3, 1, 4, 1, 5]
pre  = [0, 3, 4, 8, 9, 14]      pre[i] = sum of the first i numbers
sum(index 1..3) = pre[4] - pre[1] = 9 - 3 = 6      (1 + 4 + 1)
```

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "sum of elements from i to j", many queries, array does not change | Prefix sum array |
| "number of subarrays with sum equal to K" | Prefix sum + hash map of counts |
| "longest subarray with sum K" | Prefix sum + hash map of first index |
| "equal number of 0s and 1s" | Turn 0 into -1, then "sum = 0" |
| "subarray sum divisible by K" / "multiple of K" | Prefix sum modulo K, equal remainders |
| "add V to every element from L to R", many updates | Difference array |
| "bookings", "trips", "meetings overlap" (counts over a range) | Difference array (sweep) |
| "sum of a rectangle in a matrix", many queries | 2D prefix sum |
| "XOR of a range" | Prefix XOR (XOR undoes itself) |
| "product of everything except me", no division | Prefix product and suffix product |
| "left side sum equals right side sum" | Total minus running left sum |

## Core Concepts / Template

### 1. Prefix sum with a leading zero

Use an array one longer than the input. `pre[0] = 0` means "nothing taken yet". This removes the special case for ranges that start at index 0.

```js
function buildPrefix(nums) {
  const pre = [0];
  for (const x of nums) pre.push(pre[pre.length - 1] + x);
  return pre;
}
// sum of nums[l..r] inclusive = pre[r + 1] - pre[l]
```

### 2. Prefix sum + hash map (the "seen before" trick)

A subarray `nums[i..j]` has sum K exactly when `pre[j+1] - pre[i] = K`. So while walking, for each current prefix `cur`, you ask: "how many earlier prefixes equal `cur - K`?" A map answers that in O(1).

```js
function countWithMap(nums, k) {
  const seen = new Map([[0, 1]]); // the empty prefix: seen once
  let cur = 0, count = 0;
  for (const x of nums) {
    cur += x;
    count += seen.get(cur - k) || 0; // look BEFORE you store cur
    seen.set(cur, (seen.get(cur) || 0) + 1);
  }
  return count;
}
```

Variants: store the **count** to count subarrays, store the **first index** to find the longest subarray, and store the **remainder** instead of the raw sum for divisibility questions.

### 3. Difference array

To add `v` to `arr[l..r]`: do `diff[l] += v` and `diff[r+1] -= v`. After all updates, a running sum of `diff` gives the final array.

```js
function applyUpdates(n, updates) {
  const diff = new Array(n + 1).fill(0); // +1 so r + 1 never falls off
  for (const [l, r, v] of updates) { diff[l] += v; diff[r + 1] -= v; }
  const out = [];
  let run = 0;
  for (let i = 0; i < n; i++) { run += diff[i]; out.push(run); }
  return out;
}
```

### 4. 2D prefix sum

`p[i+1][j+1]` = sum of the rectangle from (0,0) to (i,j). Build with inclusion-exclusion: add top and left, subtract the overlap that was counted twice, add the cell.

```
rect(r1,c1,r2,c2) = p[r2+1][c2+1] - p[r1][c2+1] - p[r2+1][c1] + p[r1][c1]
```

### Step-by-step trace (prefix + map, `nums = [1, 2, 3]`, `k = 3`)

| Step | x | cur | look up cur-k | found | count | map after |
|---|---|---|---|---|---|---|
| start | | 0 | | | 0 | {0:1} |
| 1 | 1 | 1 | -2 | 0 | 0 | {0:1, 1:1} |
| 2 | 2 | 3 | 0 | 1 | 1 | {0:1, 1:1, 3:1} |
| 3 | 3 | 6 | 3 | 1 | 2 | {0:1, 1:1, 3:1, 6:1} |

The two matches are `[1,2]` and `[3]`. Answer: 2.

## Problems (easy -> hard)

### 1. Running Sum of 1D Array — Easy
**Companies:** Amazon, Google, Apple
**Problem:** Return an array where `out[i]` is the sum of `nums[0..i]`. Example: `[1,2,3,4]` -> `[1,3,6,10]`.
**Intuition:** This is the prefix sum itself. Each answer is the previous answer plus the new number.
**Approach 1 - Brute force:** For each i, add up `nums[0..i]` again. Time O(n^2), Space O(n).
**Approach 2 - Optimal:** Keep one running total and push it each step.

| i | nums[i] | running total |
|---|---|---|
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | 6 |
| 3 | 4 | 10 |

```js
function runningSum(nums) {
  const out = [];
  let sum = 0;
  for (const x of nums) { sum += x; out.push(sum); }
  return out;
}
```

**Complexity:** Time O(n), Space O(n) for the output (O(1) extra).
**Watch out:** Empty input returns `[]`. Do not forget to build a new array if the caller expects the input unchanged.

### 2. Range Sum Query - Immutable — Easy
**Companies:** Amazon, Google, Microsoft
**Problem:** Build a class from an array. Then answer many `sumRange(l, r)` calls (inclusive). Example: `[-2,0,3,-5,2,-1]`, `sumRange(0,2)` -> `1`.
**Intuition:** Pay once to precompute prefix sums, then each query is one subtraction.
**Approach 1 - Brute force:** Loop from l to r on every query. Time O(n) per query, Space O(1).
**Approach 2 - Optimal:** Prefix array with a leading 0. Answer is `pre[r+1] - pre[l]`.

Trace for `sumRange(2,5)`: `pre = [0,-2,-2,1,-4,-2,-3]`. `pre[6] - pre[2] = -3 - (-2) = -1`. Check by hand: `3 + -5 + 2 + -1 = -1`.

```js
class NumArray {
  constructor(nums) {
    this.pre = [0]; // pre[i] = sum of first i numbers
    for (const x of nums) this.pre.push(this.pre[this.pre.length - 1] + x);
  }
  sumRange(l, r) { return this.pre[r + 1] - this.pre[l]; }
}
```

**Complexity:** Time O(n) to build, O(1) per query. Space O(n).
**Watch out:** Use `pre[r + 1] - pre[l]`, not `pre[r] - pre[l]`. The leading zero is what makes `l = 0` work with no `if`.

### 3. Find Pivot Index — Easy
**Companies:** Amazon, Meta, Microsoft
**Problem:** Find the leftmost index where the sum on its left equals the sum on its right (the index itself is not counted). Return -1 if none. Example: `[1,7,3,6,5,6]` -> `3`.
**Intuition:** You do not need two arrays. If you know the total and the left sum, the right sum is `total - left - nums[i]`.
**Approach 1 - Brute force:** For each i, sum the left and the right. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** One pass for the total, one pass keeping the left sum.

Trace (`total = 28`):

| i | nums[i] | left | right = 28 - left - nums[i] | equal? |
|---|---|---|---|---|
| 0 | 1 | 0 | 27 | no |
| 1 | 7 | 1 | 20 | no |
| 2 | 3 | 8 | 17 | no |
| 3 | 6 | 11 | 11 | yes, return 3 |

```js
function pivotIndex(nums) {
  let total = 0;
  for (const x of nums) total += x;
  let left = 0;
  for (let i = 0; i < nums.length; i++) {
    if (left === total - left - nums[i]) return i;
    left += nums[i];
  }
  return -1;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Index 0 is valid when the right side sums to 0 (left is the empty sum, 0). Example `[2,1,-1]` -> `0`. Check `left` BEFORE you add `nums[i]` to it.

### 4. XOR Queries of a Subarray — Medium
**Companies:** Google, Amazon
**Problem:** Given `arr` and queries `[l, r]`, return the XOR of `arr[l..r]` for each query. Example: `arr=[1,3,4,8]`, query `[0,1]` -> `2`.
**Intuition:** XOR is its own undo: `a ^ a = 0`. So the prefix idea works with XOR in place of plus. `pre[r+1] ^ pre[l]` cancels the first l numbers and leaves the range.
**Approach 1 - Brute force:** XOR the range for each query. Time O(n) per query.
**Approach 2 - Optimal:** Prefix XOR array, O(1) per query.

Trace: `pre = [0,1,2,6,14]`. Query `[1,2]`: `pre[3] ^ pre[1] = 6 ^ 1 = 7`. Check: `3 ^ 4 = 7`.

```js
function xorQueries(arr, queries) {
  const pre = [0];
  for (const x of arr) pre.push(pre[pre.length - 1] ^ x);
  return queries.map(([l, r]) => pre[r + 1] ^ pre[l]);
}
```

**Complexity:** Time O(n + q), Space O(n).
**Watch out:** The leading 0 is the XOR identity (`x ^ 0 = x`). Same leading-zero trick as sums.

### 5. Product of Array Except Self — Medium
**Companies:** Amazon, Meta, Microsoft, Apple, Google
**Problem:** Return `out` where `out[i]` is the product of all numbers except `nums[i]`. No division allowed, O(n) time. Example: `[1,2,3,4]` -> `[24,12,8,6]`.
**Intuition:** The answer for i is (product of everything left of i) times (product of everything right of i). That is a prefix product and a suffix product.
**Approach 1 - Brute force:** For each i, multiply all others. Time O(n^2), Space O(1) extra.
**Approach 2 - Better:** Two full arrays, `left[]` and `right[]`, then multiply. Time O(n), Space O(n) extra.
**Approach 3 - Optimal:** Store the left products in the output array. Then walk from the right with one variable holding the right product, multiplying it in.

Trace on `[1,2,3,4]`:

| i | left product (pass 1) | right product (pass 2) | out[i] |
|---|---|---|---|
| 0 | 1 | 24 | 24 |
| 1 | 1 | 12 | 12 |
| 2 | 2 | 4 | 8 |
| 3 | 6 | 1 | 6 |

```js
function productExceptSelf(nums) {
  const n = nums.length;
  const out = new Array(n).fill(1);
  let left = 1;
  for (let i = 0; i < n; i++) { out[i] = left; left *= nums[i]; }
  let right = 1;
  for (let i = n - 1; i >= 0; i--) { out[i] *= right; right *= nums[i]; }
  return out;
}
```

**Complexity:** Time O(n), Space O(1) extra (the output does not count).
**Watch out:** Zeros work with no special case here, which is the reason this beats the "total / nums[i]" idea (that one breaks on zero). In JavaScript a result can be `-0` when a negative meets a zero. It prints as `0` and most judges accept it, but `Object.is(-0, 0)` is false. Add `+ 0` if your tests compare strictly.

### 6. Range Sum Query 2D - Immutable — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Build from a matrix. Answer many `sumRegion(r1, c1, r2, c2)` calls for the inclusive rectangle. Example: in the classic 5x5 sample, `sumRegion(2,1,4,3)` -> `8`.
**Intuition:** Let `p[i][j]` be the sum of the rectangle from the top-left corner to just before (i, j). A query rectangle is a big rectangle, minus the strip above, minus the strip on the left, plus the corner that was removed twice.

```
+-----------+------+
|     A     |  B   |     want D
+-----------+------+     D = whole - (A + B) - (A + C) + A
|     C     |  D   |       = whole - B_strip - C_strip + A
+-----------+------+
```

**Approach 1 - Brute force:** Loop over the rectangle per query. Time O(R * C) per query.
**Approach 2 - Better:** One 1D prefix array per row, so a query costs O(rows). Time O(R) per query.
**Approach 3 - Optimal:** 2D prefix array with an extra top row and left column of zeros.

```js
class NumMatrix {
  constructor(m) {
    const R = m.length, C = m[0].length;
    this.p = Array.from({ length: R + 1 }, () => new Array(C + 1).fill(0));
    for (let i = 0; i < R; i++)
      for (let j = 0; j < C; j++)
        this.p[i + 1][j + 1] = m[i][j] + this.p[i][j + 1] + this.p[i + 1][j] - this.p[i][j];
  }
  sumRegion(r1, c1, r2, c2) {
    const p = this.p;
    return p[r2 + 1][c2 + 1] - p[r1][c2 + 1] - p[r2 + 1][c1] + p[r1][c1];
  }
}
```

**Complexity:** Time O(R * C) to build, O(1) per query. Space O(R * C).
**Watch out:** The signs: `- top - left + corner`. Forgetting the `+ corner` is the number one bug. Use the extra row and column of zeros so `r1 = 0` or `c1 = 0` need no checks.

### 7. Subarray Sum Equals K — Medium
**Companies:** Amazon, Meta, Google, Microsoft, Bloomberg
**Problem:** Count the continuous subarrays whose sum is exactly K. Numbers can be negative. Example: `[1,1,1]`, `k=2` -> `2`.
**Intuition:** If prefix at j minus prefix at i equals K, then the subarray between them sums to K. So for each j, count earlier prefixes equal to `cur - K`.
**Approach 1 - Brute force:** Try every start and end. Time O(n^2), Space O(1).
**Approach 2 - Why not sliding window?** Sliding window needs all numbers non-negative so the sum only grows when you extend. With negatives it breaks. That is the signal to use prefix + map.
**Approach 3 - Optimal:** Map of prefix value -> how many times seen. Start with `{0: 1}`.

Trace on `[1,2,3]`, `k=3` is in Core Concepts. Another: `[1,-1,0]`, `k=0`:

| x | cur | look for cur-0 | found | count | map after |
|---|---|---|---|---|---|
| | 0 | | | 0 | {0:1} |
| 1 | 1 | 1 | 0 | 0 | {0:1,1:1} |
| -1 | 0 | 0 | 1 | 1 | {0:2,1:1} |
| 0 | 0 | 0 | 2 | 3 | {0:3,1:1} |

```js
function subarraySum(nums, k) {
  const seen = new Map([[0, 1]]);
  let sum = 0, count = 0;
  for (const x of nums) {
    sum += x;
    count += seen.get(sum - k) || 0;
    seen.set(sum, (seen.get(sum) || 0) + 1);
  }
  return count;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** (1) Seed the map with `0 -> 1`, or you miss subarrays that start at index 0. (2) Look up `cur - k` BEFORE inserting `cur`; otherwise with `k = 0` you count an empty subarray.

### 8. Contiguous Array — Medium
**Companies:** Meta, Amazon, Google
**Problem:** Find the longest contiguous subarray with the same number of 0s and 1s. Example: `[0,0,1,0,0,0,1,1]` -> `6`.
**Intuition:** Change each 0 into -1. Now "equal 0s and 1s" means "sum is 0". Two equal prefix values mean the part between them sums to 0.
**Approach 1 - Brute force:** Count 0s and 1s for every pair. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** Keep a running balance (+1 for 1, -1 for 0). Store the FIRST index where each balance appeared. If the balance appears again at i, the length is `i - firstIndex`.

Trace on `[0,1,0]`:

| i | x | balance | first seen at | length |
|---|---|---|---|---|
| start | | 0 | -1 | |
| 0 | 0 | -1 | store 0 | |
| 1 | 1 | 0 | -1 | 1 - (-1) = 2 |
| 2 | 0 | -1 | 0 | 2 - 0 = 2 |

```js
function findMaxLength(nums) {
  const first = new Map([[0, -1]]);
  let bal = 0, best = 0;
  for (let i = 0; i < nums.length; i++) {
    bal += nums[i] === 1 ? 1 : -1;
    if (first.has(bal)) best = Math.max(best, i - first.get(bal));
    else first.set(bal, i);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Seed `0 -> -1` (the empty prefix sits before index 0). Store only the first index. Never overwrite it, or the subarray shrinks.

### 9. Subarray Sums Divisible by K — Medium
**Companies:** Amazon, Meta, Google
**Problem:** Count subarrays whose sum is divisible by K. Example: `[4,5,0,-2,-3,1]`, `k=5` -> `7`.
**Intuition:** If two prefix sums have the same remainder mod K, the part between them is a multiple of K. So count pairs of equal remainders.
**Approach 1 - Brute force:** All pairs, sum each. Time O(n^2).
**Approach 2 - Optimal:** Count how many times each remainder has appeared (array of size K). For each new prefix remainder r, add `cnt[r]` to the answer, then `cnt[r]++`. Start with `cnt[0] = 1`.

Trace on `[4,5,0,-2,-3,1]`, k=5 (remainder in 0..4):

| x | prefix | rem | cnt[rem] before | answer |
|---|---|---|---|---|
| | 0 | 0 | | 0 (cnt[0]=1) |
| 4 | 4 | 4 | 0 | 0 |
| 5 | 9 | 4 | 1 | 1 |
| 0 | 9 | 4 | 2 | 3 |
| -2 | 7 | 2 | 0 | 3 |
| -3 | 4 | 4 | 3 | 6 |
| 1 | 5 | 0 | 1 | 7 |

```js
function subarraysDivByK(nums, k) {
  const cnt = new Array(k).fill(0);
  cnt[0] = 1;
  let sum = 0, ans = 0;
  for (const x of nums) {
    sum = (((sum + x) % k) + k) % k; // JS % can be negative
    ans += cnt[sum];
    cnt[sum]++;
  }
  return ans;
}
```

**Complexity:** Time O(n), Space O(K).
**Watch out:** In JavaScript `-3 % 5` is `-3`, not `2`. Fix with `((x % k) + k) % k`. Without it, negative inputs give wrong answers.

### 10. Continuous Subarray Sum — Medium
**Companies:** Meta, Amazon, Google
**Problem:** Is there a subarray of length at least 2 whose sum is a multiple of K (0 counts as a multiple)? Example: `[23,2,4,6,7]`, `k=6` -> `true` (`[2,4]`).
**Intuition:** Same remainder idea as problem 9, but now you only need to know IF, and the length must be at least 2. So store the FIRST index of each remainder and check the gap.
**Approach 1 - Brute force:** All pairs. Time O(n^2).
**Approach 2 - Optimal:** Map remainder -> first index, seeded with `0 -> -1`. When a remainder repeats, return true if `i - first >= 2`. If the gap is 1, keep the old (earlier) index.

Trace on `[23,2,4,6,7]`, k=6:

| i | x | rem of prefix | seen? | gap | result |
|---|---|---|---|---|---|
| | | 0 | store -1 | | |
| 0 | 23 | 5 | no, store 0 | | |
| 1 | 2 | 1 | no, store 1 | | |
| 2 | 4 | 5 | yes at 0 | 2 | true |

```js
function checkSubarraySum(nums, k) {
  const first = new Map([[0, -1]]);
  let rem = 0;
  for (let i = 0; i < nums.length; i++) {
    rem = (rem + nums[i]) % k;
    if (first.has(rem)) { if (i - first.get(rem) >= 2) return true; }
    else first.set(rem, i);
  }
  return false;
}
```

**Complexity:** Time O(n), Space O(min(n, K)).
**Watch out:** This version assumes `k > 0` and non-negative numbers, as the LeetCode statement does (`0 <= nums[i]`, `1 <= k`). For negatives or negative k, normalize the remainder like problem 9. `[0,0]` with any k is `true`; `[0]` alone is `false` (length 1).

### 11. Range Addition — Medium
**Companies:** Google, Amazon
**Problem:** Start with `length` zeros. Apply updates `[start, end, inc]` (inclusive) and return the final array. Example: `length=5`, updates `[[1,3,2],[2,4,3],[0,2,-2]]` -> `[-2,0,3,5,3]`. (This is a LeetCode premium problem.)
**Intuition:** Do not touch every cell. Mark only where the add starts (`+inc`) and where it stops (`-inc` just after the end). A running sum spreads the effect.
**Approach 1 - Brute force:** Loop from start to end per update. Time O(n * q).
**Approach 2 - Optimal:** Difference array, then one prefix sum.

Trace of the difference array (size 6):

| update | diff after |
|---|---|
| start | [0,0,0,0,0,0] |
| [1,3,2] | [0,2,0,0,-2,0] |
| [2,4,3] | [0,2,3,0,-2,-3] |
| [0,2,-2] | [-2,2,3,2,-2,-3] |

Running sum of the first 5 entries: `-2, 0, 3, 5, 3`.

```js
function getModifiedArray(length, updates) {
  const diff = new Array(length + 1).fill(0);
  for (const [l, r, v] of updates) { diff[l] += v; diff[r + 1] -= v; }
  const out = [];
  let run = 0;
  for (let i = 0; i < length; i++) { run += diff[i]; out.push(run); }
  return out;
}
```

**Complexity:** Time O(n + q), Space O(n).
**Watch out:** Make `diff` one longer so `end + 1` is a valid index. Use `-=` and `+=`, never `=`, because updates overlap.

### 12. Corporate Flight Bookings — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** There are `n` flights numbered 1..n. Each booking `[first, last, seats]` reserves seats on flights first..last. Return seats reserved per flight. Example: `[[1,2,10],[2,3,20],[2,5,25]]`, `n=5` -> `[10,55,45,25,25]`.
**Intuition:** It is Range Addition with 1-based flight numbers.
**Approach 1 - Brute force:** For each booking, add to every flight in range. Time O(n * q).
**Approach 2 - Optimal:** Difference array, then sweep.

Trace of the difference array (index 1..5, then 6 as spare): after all bookings: `diff[1]=10`, `diff[2]=20+25=45`, `diff[3]=-10`, `diff[4]=-20`, `diff[6]=-25`. The running sum is `10, 55, 45, 25, 25`.

```js
function corpFlightBookings(bookings, n) {
  const diff = new Array(n + 2).fill(0); // 1-indexed flights
  for (const [f, l, seats] of bookings) { diff[f] += seats; diff[l + 1] -= seats; }
  const out = [];
  let run = 0;
  for (let i = 1; i <= n; i++) { run += diff[i]; out.push(run); }
  return out;
}
```

**Complexity:** Time O(n + q), Space O(n).
**Watch out:** 1-based indexes. Size `n + 2` so `last + 1` (up to `n + 1`) fits.

### 13. Car Pooling — Medium
**Companies:** Amazon, Uber, Google, Lyft
**Problem:** A car has `capacity` seats and only drives east. Each trip is `[passengers, from, to]`. Can it carry all trips without ever exceeding capacity? Example: `[[2,1,5],[3,3,7]]`, `capacity=4` -> `false`.
**Intuition:** At each stop, passengers change: `+p` at `from`, `-p` at `to`. The running sum is the number of people in the car. If it ever goes above capacity, fail.
**Approach 1 - Brute force:** For each trip, add its passengers to every stop it covers, then check the max. Time O(n * 1000).
**Approach 2 - Optimal:** Difference array over stop positions (LeetCode limits stops to 0..1000), then sweep and check.
**Approach 3 - Any range of stops:** Sort the events by stop and process drop-offs first. Time O(n log n). Use this if the positions are huge.

Trace for `[[2,1,5],[3,3,7]]`, capacity 4: `diff[1]=+2`, `diff[3]=+3`, `diff[5]=-2`, `diff[7]=-3`. Running load: stop 1 -> 2, stop 3 -> 5. `5 > 4`, return false.

```js
function carPooling(trips, capacity) {
  const diff = new Array(1002).fill(0);
  for (const [p, from, to] of trips) { diff[from] += p; diff[to] -= p; } // drop-off before pickup at same stop
  let load = 0;
  for (const d of diff) { load += d; if (load > capacity) return false; }
  return true;
}
```

**Complexity:** Time O(n + S), Space O(S), where S = 1001 stops.
**Watch out:** The passengers leave at `to`, so they are NOT in the car at `to` (that is why we subtract at `to`, not `to + 1`). A pickup and drop-off at the same stop must not overlap. Check the load after each full position, which this sweep does.

### 14. Maximum Subarray (via prefix sums) — Medium
**Companies:** Amazon, Microsoft, Google, Apple, Meta
**Problem:** Find the largest sum of any non-empty contiguous subarray. Example: `[-2,1,-3,4,-1,2,1,-5,4]` -> `6` (`[4,-1,2,1]`).
**Intuition:** The sum of `nums[i..j]` is `pre[j+1] - pre[i]`. To make it as big as possible for a fixed j, subtract the SMALLEST earlier prefix. This is the same as Kadane's algorithm, seen through prefix sums.
**Approach 1 - Brute force:** All pairs. Time O(n^2).
**Approach 2 - Optimal:** Walk once, keep the running prefix and the minimum prefix seen so far (including the empty prefix 0).

Trace on `[-2,1,-3,4]`:

| x | prefix | min prefix before | prefix - min | best |
|---|---|---|---|---|
| -2 | -2 | 0 | -2 | -2 |
| 1 | -1 | -2 | 1 | 1 |
| -3 | -4 | -2 | -2 | 1 |
| 4 | 0 | -4 | 4 | 4 |

```js
function maxSubArray(nums) {
  let pre = 0, minPre = 0, best = -Infinity;
  for (const x of nums) {
    pre += x;
    best = Math.max(best, pre - minPre);
    minPre = Math.min(minPre, pre);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Start `best` at `-Infinity`, not 0, or an all-negative array returns 0 (wrong, the subarray must be non-empty). Update `best` BEFORE you update `minPre`, so the subarray is never empty.

### 15. Count of Range Sum — Hard
**Companies:** Google, Amazon, Meta
**Problem:** Count subarrays whose sum lies in `[lower, upper]`. Example: `[-2,5,-1]`, `lower=-2`, `upper=2` -> `3`.
**Intuition:** With prefix sums, you want pairs `i < j` with `lower <= pre[j] - pre[i] <= upper`. A hash map cannot do a range of values. Instead, sort while counting (merge sort), because after sorting the left half, the valid `i` values form a window that moves only forward.
**Approach 1 - Brute force:** All pairs. Time O(n^2), Space O(n).
**Approach 2 - Optimal:** Merge sort on the prefix array. Before merging, for each `j` in the right half, two pointers `a` and `b` find the left-half items with `pre[j] - upper <= pre[i] <= pre[j] - lower`. Add `b - a`. (A Fenwick tree or balanced tree also works; see the Segment Tree and Fenwick chapter.)

Trace on `[-2,5,-1]`: prefixes `[0,-2,3,2]`, the valid pairs `(i<j)` are: `(0,-2)` has diff -2 ok; `(0,3)` diff 3 no; `(0,2)` diff 2 ok; `(-2,3)` diff 5 no; `(-2,2)` diff 4 no; `(3,2)` diff -1 ok. Total 3.

```js
function countRangeSum(nums, lower, upper) {
  const pre = [0];
  for (const x of nums) pre.push(pre[pre.length - 1] + x);
  let count = 0;
  function sort(lo, hi) { // sorts pre[lo..hi)
    if (hi - lo <= 1) return;
    const mid = (lo + hi) >> 1;
    sort(lo, mid); sort(mid, hi);
    let a = lo, b = lo;
    for (let j = mid; j < hi; j++) {
      while (a < mid && pre[a] < pre[j] - upper) a++;
      while (b < mid && pre[b] <= pre[j] - lower) b++;
      count += b - a;
    }
    const merged = pre.slice(lo, hi).sort((x, y) => x - y); // shortcut: O(n log^2 n) total, fine here
    for (let i = 0; i < merged.length; i++) pre[lo + i] = merged[i];
  }
  sort(0, pre.length);
  return count;
}
```

**Complexity:** Time O(n log^2 n) as written (the merge uses `sort` for brevity); swap in a two-pointer merge for O(n log n). Space O(n).
**Watch out:** Count BEFORE merging (while both halves are still separately sorted and `i < j` holds by position). Use the prefix array with the leading 0, or subarrays that start at index 0 are missed. This is rare in interviews; know the idea more than the code.

## Common Mistakes

- **"Prefix sum has n entries."** It needs n + 1 (with the leading 0). Without it, every query that starts at index 0 needs a special case, and many bugs hide there.
- **"I can use sliding window for sum = K."** Only if all numbers are non-negative. With negatives use prefix + map.
- **"Insert the prefix into the map, then look up."** Look up first, then insert. Otherwise K = 0 counts empty subarrays.
- **"I forgot the empty prefix in the map."** Seed with `0 -> 1` (counts) or `0 -> -1` (first index). Otherwise subarrays that start at index 0 are missed.
- **"`%` gives a non-negative result."** In JavaScript it keeps the sign of the left side. Use `((x % k) + k) % k`.
- **"For longest, store the latest index."** Store the FIRST index. Overwriting shrinks the answer.
- **"Difference array needs size n."** Size n + 1, because the `-v` goes at `r + 1`.
- **"2D prefix: just add the strips."** You must subtract the overlapping corner once (inclusion-exclusion).
- **"Prefix sum works if the array changes."** It is for static arrays. If there are point updates, use a Fenwick tree (see [02 - Segment Tree and Fenwick](../../06-advanced/02-segment-tree-and-fenwick/README.md)).
- **"Numbers will fit."** Sums of many large numbers can pass `Number.MAX_SAFE_INTEGER` (about 9e15). Ask about limits; use `BigInt` only if needed.

## Interview Tips

- Say the pattern out loud: "I will precompute prefix sums so each range sum is one subtraction." Then state the leading-zero convention.
- For subarray counting, say: "A subarray sum is a difference of two prefix sums, so I count earlier prefixes equal to `cur - K` with a hash map."
- Always ask: can numbers be negative? Can they be zero? Can K be zero or negative? These decide between sliding window and prefix + map.
- Follow-ups you may hear:
  - "What if the array changes between queries?" -> Fenwick tree or segment tree.
  - "Longest instead of count?" -> store first index, not count.
  - "Do it in O(1) extra space." -> product except self, pivot index.
  - "Many range updates, one read at the end?" -> difference array.
  - "Many updates AND many queries mixed?" -> Fenwick tree with range update, or segment tree with lazy propagation.
- Mention that the difference array and the prefix sum are inverses: one is "differences", the other is "running totals".
- Edge cases to name: empty array, single element, all negative, all zeros, K = 0, queries at index 0, and the last index.
- Keep a tiny example on the whiteboard and trace the map. It catches off-by-one mistakes fast.

## Practice List

**Easy**
- Running Sum of 1D Array
- Range Sum Query - Immutable
- Find Pivot Index
- Find the Middle Index in Array
- Find the Highest Altitude
- Left and Right Sum Differences
- Minimum Value to Get Positive Step by Step Sum
- Sum of All Odd Length Subarrays

**Medium**
- Subarray Sum Equals K
- Contiguous Array
- Continuous Subarray Sum
- Subarray Sums Divisible by K
- Product of Array Except Self
- Range Sum Query 2D - Immutable
- Corporate Flight Bookings
- Car Pooling
- Range Addition
- XOR Queries of a Subarray
- Maximum Size Subarray Sum Equals k
- Binary Subarrays With Sum
- Count Number of Nice Subarrays
- Make Sum Divisible by P
- Maximum Subarray
- Maximum Sum of Two Non-Overlapping Subarrays
- Number of Submatrices That Sum to Target (2D prefix + map)
- Shortest Unsorted Continuous Subarray
- Find the Longest Substring Containing Vowels in Even Counts (prefix of bitmask)
- Maximum Sum Circular Subarray

**Hard**
- Count of Range Sum
- Max Sum of Rectangle No Larger Than K
- Shortest Subarray with Sum at Least K (prefix + monotonic deque)
- Maximum Sum of 3 Non-Overlapping Subarrays
- Count of Smaller Numbers After Self (same merge-sort idea)

## Key Takeaways

- A **prefix array with a leading 0** makes any range sum `pre[r + 1] - pre[l]`, with no special cases.
- **Subarray sum = difference of two prefix sums.** So "count subarrays with sum K" becomes "count earlier prefixes equal to `cur - K`".
- Seed the map with the **empty prefix** (`0 -> 1` for counts, `0 -> -1` for first index), and **look up before you insert**.
- **Negative numbers** rule out sliding window; prefix + hash map still works.
- For divisibility, track the **prefix remainder**, and fix negative remainders in JavaScript with `((x % k) + k) % k`.
- A **difference array** turns many range updates into two point updates each; one running sum rebuilds the result.
- **2D prefix sums** use inclusion-exclusion: add, subtract two strips, add the corner back.
- Prefix sums are for **static** data. When updates mix with queries, move to a Fenwick or segment tree.

---

[← Previous: Sliding Window](../06-sliding-window/README.md) | [Next: Binary Search →](../08-binary-search/README.md) | [Back to Core Linear Index](../README.md)
