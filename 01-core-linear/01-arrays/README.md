# Arrays

> **Estimated Time:** 6 hr | **Difficulty:** Beginner
> **Prerequisites:** [02 - Complexity Analysis](../../00-foundations/02-complexity-analysis/README.md), [03 - JavaScript for DSA](../../00-foundations/03-javascript-for-dsa/README.md)

---

## What You'll Learn
- You can explain why array access is O(1) but insert and delete in the middle are O(n).
- You can edit an array in place with a read pointer and a write pointer.
- You can swap, reverse and partition an array without extra memory.
- You can use Kadane's idea, running min/max, and prefix/suffix passes to turn O(n^2) into O(n).
- You can solve the classic array interview problems and say their time and space cost out loud.

## The Big Idea

Think of a row of lockers, numbered 0, 1, 2, ... All lockers are the same size and sit side by side. If someone says "open locker 57", you walk straight there. You do not check the first 56. That is an array: items sit next to each other in memory, so the computer jumps to any index in one step.

The price: if you want to squeeze a new item into the middle, everyone after it must shift one locker. That is O(n) work. So most array problems are about two questions: "how do I read the data in one pass?" and "how do I rearrange it without shifting things again and again?"

Interviewers love arrays because they are simple to state, yet they test the core skills: loops, indexes, edge cases (empty, one element, all equal, negatives), and whether you can cut brute force O(n^2) down to O(n) with one good idea.

| Operation | Cost | Why |
|---|---|---|
| Read / write `a[i]` | O(1) | direct jump |
| Push / pop at the end | O(1) amortized | no shifting |
| Insert / delete at index i | O(n) | later items shift |
| `unshift` / `shift` (front) | O(n) | everything shifts |
| Search unsorted | O(n) | must look at each |
| Search sorted | O(log n) | binary search |

JavaScript note: a JS array is a flexible list, not a fixed block. It grows by itself, and `arr.length` is always known. The thinking above still holds for interviews.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "in place", "O(1) extra space" | read/write pointers, swaps, reversal |
| "largest / smallest / second largest" | one pass, keep a few variables |
| "contiguous subarray with max sum" | Kadane's algorithm |
| "sorted array, remove duplicates" | write pointer compares with last kept item |
| "move all X to the end" | write pointer for the "keep" items |
| "values 0, 1, 2 only" | Dutch national flag (3 pointers) |
| "more than n/2 (or n/3) times" | Boyer-Moore voting |
| "numbers 0..n with one missing" | sum or XOR trick |
| "product / sum of everything except me" | prefix pass + suffix pass |
| "next greater arrangement" | next permutation (scan from the right) |
| "buy low, sell high once" | track minimum so far |

## Core Concepts / Template

**1. Two pointers on the same array (read / write).** The read pointer `r` visits every item. The write pointer `w` marks where the next "kept" item goes. Everything left of `w` is already final.

```js
function keepIf(a, keep) {
  let w = 0;
  for (let r = 0; r < a.length; r++) {
    if (keep(a[r], r)) a[w++] = a[r];
  }
  return w; // new length
}
```

**2. Swap and reverse.** Swapping two items: `[a[i], a[j]] = [a[j], a[i]]`. Reversing a range uses two ends moving inward:

```js
function reverseRange(a, l, r) {
  while (l < r) { [a[l], a[r]] = [a[r], a[l]]; l++; r--; }
}
```

**3. Running best.** Walk once, keep "best so far" and "state right now" (running min, running sum, running candidate).

**4. Prefix and suffix passes.** One left-to-right pass stores what is to the left of each index. One right-to-left pass adds what is to the right.

**Quick warm-ups (all one pass):**

```js
// Is the array sorted (non-decreasing)? Also: "sorted then rotated" check.
function isSorted(a) {
  for (let i = 1; i < a.length; i++) if (a[i - 1] > a[i]) return false;
  return true;
}
// Sorted and rotated: a circular array may have at most ONE drop.
function isSortedAndRotated(a) {
  let drops = 0;
  for (let i = 0; i < a.length; i++) if (a[i] > a[(i + 1) % a.length]) drops++;
  return drops <= 1;
}

// Single Number: every value appears twice except one. x ^ x = 0 and x ^ 0 = x.
const singleNumber = (a) => a.reduce((x, y) => x ^ y, 0);

// Leaders: an item is a leader if nothing to its right is bigger. Scan from the right.
function leaders(a) {
  const out = []; let max = -Infinity;
  for (let i = a.length - 1; i >= 0; i--) if (a[i] >= max) { out.push(a[i]); max = a[i]; }
  return out.reverse(); // [16,17,4,3,5,2] -> [17,5,2]
}
```

All of the above were run against examples (`[3,4,5,1,2]` is sorted-and-rotated, `[2,1,3,4]` is not; `[4,1,2,1,2]` -> 4).

## Problems (easy -> hard)

### 1. Second Largest Element — Easy
**Companies:** Amazon, Microsoft, Adobe
**Problem:** Return the second largest distinct value in an array, or `null` if there is none. Example: `[12, 35, 1, 10, 34, 1]` -> `34`.
**Intuition:** Keep two boxes, `first` and `second`. When a new number beats `first`, the old `first` slides down into `second`. No sorting needed.
**Approach 1 - Brute force:** Sort, then walk from the end to find a value different from the max. Time O(n log n), Space O(1) extra if sorting in place.
**Approach 2 - Better:** Two passes: find the max, then find the biggest value that is smaller than max. Time O(n), Space O(1).
**Approach 3 - Optimal:** One pass with two variables.

| x | first | second |
|---|---|---|
| start | -inf | -inf |
| 12 | 12 | -inf |
| 35 | 35 | 12 |
| 1 | 35 | 12 |
| 10 | 35 | 12 |
| 34 | 35 | 34 |
| 1 | 35 | 34 |

```js
function secondLargest(a) {
  let first = -Infinity, second = -Infinity;
  for (const x of a) {
    if (x > first) { second = first; first = x; }
    else if (x < first && x > second) second = x; // x < first skips duplicates of the max
  }
  return second === -Infinity ? null : second;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** `[5,5,5]` has no second largest (answer `null`). Negative numbers work because we start from `-Infinity`, not 0. Initializing with `a[0]` and `a[1]` is a common source of bugs.

### 2. Remove Duplicates from Sorted Array — Easy
**Companies:** Amazon, Microsoft, Google, Meta
**Problem:** Given a sorted array, keep one copy of each value at the front, in place, and return how many unique values there are. Example: `[0,0,1,1,1,2,2,3,3,4]` -> `5`, array starts `[0,1,2,3,4,...]`.
**Intuition:** Because the array is sorted, duplicates sit side by side. A new value is just "different from the last one I kept".
**Approach 1 - Brute force:** Put values into a `Set`, copy back. Time O(n), Space O(n). Breaks the "in place" rule.
**Approach 2 - Better:** Delete duplicates with `splice`. Each splice shifts items, so Time O(n^2), Space O(1).
**Approach 3 - Optimal:** Read/write pointers. `w` is the next free slot; `a[w-1]` is the last kept value.

Trace on `[0,0,1,1,2]`:
```
r=1: a[1]=0 equals a[w-1]=0 -> skip
r=2: a[2]=1 differs         -> a[1]=1, w=2
r=3: a[3]=1 equals a[1]=1   -> skip
r=4: a[4]=2 differs         -> a[2]=2, w=3   => first 3 items: [0,1,2]
```

```js
function removeDuplicates(a) {
  if (a.length === 0) return 0;
  let w = 1;
  for (let r = 1; r < a.length; r++) {
    if (a[r] !== a[w - 1]) a[w++] = a[r];
  }
  return w;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Empty array. Start `w` and `r` at 1, since the first item is always kept. This only works because the input is sorted. For "allow at most 2 copies" compare with `a[w-2]` instead.

### 3. Move Zeroes — Easy
**Companies:** Meta, Amazon, Microsoft, Apple, Google
**Problem:** Move all 0s to the end, keep the order of non-zero items, in place. Example: `[0,1,0,3,12]` -> `[1,3,12,0,0]`.
**Intuition:** Same read/write idea. Each non-zero item is swapped to the write spot. Zeros get pushed back automatically.
**Approach 1 - Brute force:** Copy non-zeros into a new array, then pad with zeros. Time O(n), Space O(n).
**Approach 2 - Better:** Write non-zeros forward, then fill the rest with 0. Time O(n), Space O(1), two loops.
**Approach 3 - Optimal:** One loop, swap non-zero items into `w`.

Trace on `[0,1,0,3,12]`:
```
r=0: 0, skip                     w=0
r=1: 1 -> swap a[0],a[1] = [1,0,0,3,12]  w=1
r=2: 0, skip
r=3: 3 -> swap a[1],a[3] = [1,3,0,0,12]  w=2
r=4: 12-> swap a[2],a[4] = [1,3,12,0,0]  w=3
```

```js
function moveZeroes(a) {
  let w = 0;
  for (let r = 0; r < a.length; r++) {
    if (a[r] !== 0) { [a[w], a[r]] = [a[r], a[w]]; w++; }
  }
  return a;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** When `w === r` the swap is a harmless self-swap. If the interviewer asks to minimize writes, add `if (w !== r)` before the swap. The same loop (with `=== 1`) also solves "Max Consecutive Ones" style counting.

### 4. Rotate Array — Medium
**Companies:** Amazon, Microsoft, Google, Adobe
**Problem:** Rotate the array to the right by `k` steps, in place. Example: `[1,2,3,4,5,6,7]`, k=3 -> `[5,6,7,1,2,3,4]`.
**Intuition:** Rotating right by k means the last k items jump to the front. Reversing pieces gives exactly that: reverse everything, then reverse the two parts back.
**Approach 1 - Brute force:** Rotate by one step, k times. Each step shifts n items. Time O(n*k), Space O(1). Too slow for big k.
**Approach 2 - Better:** Extra array: item at `i` goes to `(i + k) % n`. Time O(n), Space O(n).
```js
function rotateExtra(a, k) {
  const n = a.length, out = new Array(n);
  for (let i = 0; i < n; i++) out[(i + k) % n] = a[i];
  return out;
}
```
**Approach 3 - Optimal:** Three reversals.

Trace on `[1,2,3,4,5,6,7]`, k=3:
```
reverse all        -> [7,6,5,4,3,2,1]
reverse first k=3  -> [5,6,7,4,3,2,1]
reverse the rest   -> [5,6,7,1,2,3,4]
```

```js
function rotate(a, k) {
  const n = a.length; k %= n; // k can be bigger than n
  const rev = (l, r) => { while (l < r) { [a[l], a[r]] = [a[r], a[l]]; l++; r--; } };
  rev(0, n - 1); rev(0, k - 1); rev(k, n - 1);
  return a;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Do `k %= n` first. For an empty array `n = 0` gives `NaN`; guard with `if (n === 0) return a`. For a LEFT rotation by k, rotate right by `n - k`. A fourth way, the "cycle / juggling" method, also gives O(1) space but is harder to write bug-free; the reversal trick is the safe choice in an interview.

### 5. Missing Number — Easy
**Companies:** Amazon, Microsoft, Apple, Meta
**Problem:** An array holds `n` distinct numbers from `0..n`. One number is missing. Find it. Example: `[3,0,1]` -> `2`.
**Intuition:** If the full set `0..n` were present, adding them all gives a known total. The gap between the expected total and the real total is the missing number. XOR does the same without overflow worry.
**Approach 1 - Brute force:** For each v in `0..n`, scan the array. Time O(n^2), Space O(1).
**Approach 2 - Better:** Put everything in a `Set`, find the one not there. Time O(n), Space O(n). Or sort and look for the gap, O(n log n).
**Approach 3 - Optimal:** XOR all indexes and all values together. Pairs cancel (`x ^ x = 0`), the missing value stays.

Trace on `[3,0,1]` (n=3), start `x = 3`:
```
i=0: x ^= 0 ^ 3 -> 0
i=1: x ^= 1 ^ 0 -> 1
i=2: x ^= 2 ^ 1 -> 2   answer 2
```

```js
function missingNumber(a) {
  let x = a.length;
  for (let i = 0; i < a.length; i++) x ^= i ^ a[i];
  return x;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Start `x` at `n`, because indexes only go up to `n-1` while values go up to `n`. The sum formula `n*(n+1)/2 - sum` also works and is easy to explain; in JS numbers are safe up to 2^53, so overflow is not an issue here (it is in C++/Java).

### 6. Maximum Subarray (Kadane's Algorithm) — Medium
**Companies:** Amazon, Microsoft, Google, Apple, Meta
**Problem:** Find the contiguous subarray with the largest sum (and, as a follow-up, print it). Example: `[-2,1,-3,4,-1,2,1,-5,4]` -> `6` from `[4,-1,2,1]`.
**Intuition:** At each item, ask: "is the sum of the run that ends just before me helping or hurting?" If it is negative, drop it and start fresh at me. A negative prefix can never make a later sum bigger.
**Approach 1 - Brute force:** Try every start and end, sum each. Time O(n^3), or O(n^2) with a running sum. Space O(1).
**Approach 2 - Better:** Divide and conquer (best in left half, right half, or crossing the middle). Time O(n log n). Rarely asked, but good to know it exists.
**Approach 3 - Optimal:** Kadane. Track `cur` (best sum ending here), `best`, and the start index of the current run.

Trace on `[-2,1,-3,4,-1,2,1,-5,4]`:

| i | a[i] | cur (after) | start | best |
|---|---|---|---|---|
| 0 | -2 | -2 | 0 | -2 |
| 1 | 1 | 1 (old cur < 0, restart) | 1 | 1 |
| 2 | -3 | -2 | 1 | 1 |
| 3 | 4 | 4 (restart) | 3 | 4 |
| 4 | -1 | 3 | 3 | 4 |
| 5 | 2 | 5 | 3 | 5 |
| 6 | 1 | 6 | 3 | 6 (ends at 6) |
| 7 | -5 | 1 | 3 | 6 |
| 8 | 4 | 5 | 3 | 6 |

```js
function maxSubArray(a) {
  let best = a[0], cur = a[0], start = 0, bs = 0, be = 0;
  for (let i = 1; i < a.length; i++) {
    if (cur < 0) { cur = a[i]; start = i; } else cur += a[i];
    if (cur > best) { best = cur; bs = start; be = i; }
  }
  return { best, sub: a.slice(bs, be + 1) };
}
```

**Complexity:** Time O(n), Space O(1) (the printed slice aside).
**Watch out:** Do NOT initialize `best = 0`; an all-negative array like `[-3,-1,-2]` must return `-1`, not 0. Ask whether an empty subarray is allowed. Compare with `cur > best` (strict) to keep the first of equal-sum answers.

### 7. Best Time to Buy and Sell Stock — Easy
**Companies:** Amazon, Meta, Google, Microsoft, Bloomberg
**Problem:** `prices[i]` is the price on day i. Buy once, sell later, maximize profit (0 if no profit). Example: `[7,1,5,3,6,4]` -> `5` (buy at 1, sell at 6).
**Intuition:** For each day, the best you can do if you sell today is `today - cheapest price so far`. Remember the cheapest price as you walk.
**Approach 1 - Brute force:** All pairs (buy, sell). Time O(n^2), Space O(1).
**Approach 2 - Better:** Precompute the max price to the right of each day, then one pass. Time O(n), Space O(n).
**Approach 3 - Optimal:** One pass, one variable for the minimum.

| day price | lowest so far | profit if sold today | best |
|---|---|---|---|
| 7 | 7 | 0 | 0 |
| 1 | 1 | 0 | 0 |
| 5 | 1 | 4 | 4 |
| 3 | 1 | 2 | 4 |
| 6 | 1 | 5 | 5 |
| 4 | 1 | 3 | 5 |

```js
function maxProfit(prices) {
  let lo = Infinity, best = 0;
  for (const p of prices) {
    lo = Math.min(lo, p);
    best = Math.max(best, p - lo);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Update the minimum before computing profit (selling the same day you buy gives 0, which is fine). Falling prices must return 0, never a negative number. Multiple buys/sells and "with cooldown" are different problems; see the Dynamic Programming stocks chapter.

### 8. Sort Colors (Dutch National Flag) — Medium
**Companies:** Amazon, Microsoft, Meta, Apple, Google
**Problem:** The array has only 0, 1 and 2. Sort it in place in one pass without a library sort. Example: `[2,0,2,1,1,0]` -> `[0,0,1,1,2,2]`.
**Intuition:** Split the array into four zones: `[0s | 1s | unknown | 2s]`. Pointers `lo`, `mid`, `hi` mark the borders. Look at `a[mid]` and send it to the right zone. The unknown zone shrinks every step.
**Approach 1 - Brute force:** Library sort. Time O(n log n), Space depends. Fine for real code, not what the interviewer wants.
**Approach 2 - Better:** Counting: count 0s, 1s, 2s, then overwrite. Time O(n), Space O(1), two passes.
**Approach 3 - Optimal:** Three pointers, one pass.

Rules: `a[mid] === 0` -> swap with `lo`, `lo++`, `mid++`. `a[mid] === 1` -> `mid++`. `a[mid] === 2` -> swap with `hi`, `hi--` (do NOT move `mid`, the swapped-in item is unchecked).

Trace on `[2,0,2,1,1,0]`:
```
lo=0 mid=0 hi=5 a[0]=2 -> swap with hi  [0,0,2,1,1,2] hi=4
a[0]=0 -> lo=1 mid=1
a[1]=0 -> lo=2 mid=2
a[2]=2 -> swap with hi  [0,0,1,1,2,2] hi=3
a[2]=1 -> mid=3
a[3]=1 -> mid=4 > hi, stop
```

```js
function sortColors(a) {
  let lo = 0, mid = 0, hi = a.length - 1;
  while (mid <= hi) {
    if (a[mid] === 0) { [a[lo], a[mid]] = [a[mid], a[lo]]; lo++; mid++; }
    else if (a[mid] === 1) mid++;
    else { [a[mid], a[hi]] = [a[hi], a[mid]]; hi--; } // do not advance mid here
  }
  return a;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** The loop condition is `mid <= hi`, not `<`. After swapping with `hi`, `mid` must stay. After swapping with `lo`, `mid` can move on, because everything left of `mid` and right of `lo` is a 1 that we already saw.

### 9. Majority Element (Boyer-Moore Voting) — Easy
**Companies:** Amazon, Google, Microsoft, Adobe, Meta
**Problem:** Find the value that appears more than `n/2` times (it is guaranteed to exist). Example: `[2,2,1,1,1,2,2]` -> `2`.
**Intuition:** Imagine a vote where each different pair cancels out one vote for each side. The true majority has more than half the votes, so it cannot be fully cancelled. Whoever is left standing is the winner.
**Approach 1 - Brute force:** Count each value with a nested loop. Time O(n^2), Space O(1).
**Approach 2 - Better:** Hash map of counts (or sort and take the middle item). Time O(n), Space O(n) for the map; O(n log n) for sorting.
**Approach 3 - Optimal:** Boyer-Moore. Keep a candidate and a counter.

Trace on `[2,2,1,1,1,2,2]`:

| x | candidate | count |
|---|---|---|
| 2 | 2 | 1 |
| 2 | 2 | 2 |
| 1 | 2 | 1 |
| 1 | 2 | 0 |
| 1 | 1 (count was 0, switch) | 1 |
| 2 | 1 | 0 |
| 2 | 2 (switch) | 1 |

```js
function majorityElement(a) {
  let cand = null, cnt = 0;
  for (const x of a) {
    if (cnt === 0) cand = x;
    cnt += x === cand ? 1 : -1;
  }
  return cand; // add a verify pass if a majority is NOT guaranteed
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** The candidate is only right if a majority truly exists. If not guaranteed, count the candidate in a second pass. Follow-up, **Majority Element II** (more than `n/3`): at most TWO values can qualify, so keep two candidates and two counters, then verify both:

```js
function majorityII(a) {
  let c1 = null, c2 = null, n1 = 0, n2 = 0;
  for (const x of a) {
    if (x === c1) n1++;
    else if (x === c2) n2++;
    else if (n1 === 0) { c1 = x; n1 = 1; }
    else if (n2 === 0) { c2 = x; n2 = 1; }
    else { n1--; n2--; }
  }
  const t = Math.floor(a.length / 3), res = [];
  for (const c of [c1, c2]) {
    if (c !== null && a.filter(x => x === c).length > t && !res.includes(c)) res.push(c);
  }
  return res; // [3,2,3] -> [3]
}
```
The order of the `if` checks matters: match existing candidates first, then take an empty slot, and only then cancel.

### 10. Rearrange Array Elements by Sign — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** The array has equal numbers of positives and negatives. Reorder so signs alternate, starting with positive, and each sign group keeps its original order. Example: `[3,1,-2,-5,2,-4]` -> `[3,-2,1,-5,2,-4]`.
**Intuition:** Positives belong at even indexes (0, 2, 4...), negatives at odd indexes (1, 3, 5...). Keep one write cursor for each sign and jump by 2.
**Approach 1 - Brute force:** Split into a positives list and a negatives list, then interleave. Time O(n), Space O(n). Clear and fine.
**Approach 2 - Better:** Same idea, but place each item directly into the answer array with two cursors (below). One pass instead of two.
**Approach 3 - Optimal:** For this exact version (order must be kept) O(n) extra space is the accepted answer. In-place with order kept needs rotations and costs O(n^2), so say that out loud rather than chase it. If the counts are NOT equal (follow-up), fill both cursors as far as possible, then append the leftover of the longer sign at the end.

Trace on `[3,1,-2,-5,2,-4]` (p=0, n=1):
```
3  -> out[0]=3  p=2
1  -> out[2]=1  p=4
-2 -> out[1]=-2 n=3
-5 -> out[3]=-5 n=5
2  -> out[4]=2  p=6
-4 -> out[5]=-4 n=7     out=[3,-2,1,-5,2,-4]
```

```js
function rearrangeBySign(a) {
  const out = new Array(a.length);
  let p = 0, n = 1;
  for (const x of a) {
    if (x > 0) { out[p] = x; p += 2; }
    else { out[n] = x; n += 2; }
  }
  return out;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Ask what to do with zero and whether counts are equal. The "in place" version is a trap if the order must stay stable.

### 11. Next Permutation — Medium
**Companies:** Google, Amazon, Meta, Microsoft, Bloomberg
**Problem:** Rearrange the array into the next lexicographically larger arrangement. If it is already the largest, wrap to the smallest (sorted ascending). In place. Example: `[1,2,3]` -> `[1,3,2]`, `[3,2,1]` -> `[1,2,3]`.
**Intuition:** Think of a car odometer. To get the next number you change the rightmost digit that CAN grow, make it grow by the smallest possible step, then reset everything after it to the smallest order. The tail that is in non-increasing order is already "maxed out", so we look just before it.
**Approach 1 - Brute force:** Generate all permutations in order, find yours, return the next. Time O(n! * n), Space O(n!). Only to show you understand the problem.
**Approach 2 - Better:** None worth presenting.
**Approach 3 - Optimal:** Three steps.
1. From the right, find the first index `i` with `a[i] < a[i+1]` (the "pivot"). If none, the array is the largest; skip to step 3.
2. From the right, find the first `j` with `a[j] > a[i]`, swap `a[i]` and `a[j]`.
3. Reverse everything after `i` (the tail was descending, now it is the smallest ascending order).

Trace on `[1,3,5,4,2]`:
```
step 1: pivot i=1 (3 < 5); tail [5,4,2] is descending
step 2: first from right bigger than 3 is 4 (j=3) -> swap -> [1,4,5,3,2]
step 3: reverse after i -> [1,4,2,3,5]     (next permutation after 13542)
```

```js
function nextPermutation(a) {
  let i = a.length - 2;
  while (i >= 0 && a[i] >= a[i + 1]) i--; // >= handles duplicates
  if (i >= 0) {
    let j = a.length - 1;
    while (a[j] <= a[i]) j--;
    [a[i], a[j]] = [a[j], a[i]];
  }
  for (let l = i + 1, r = a.length - 1; l < r; l++, r--) [a[l], a[r]] = [a[r], a[l]];
  return a;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Use `>=` and `<=` so duplicates (`[1,1,5]` -> `[1,5,1]`) work. When no pivot exists, `i = -1` and the final loop reverses the whole array, which is exactly the wrap-around case. For the "previous permutation" follow-up, flip the comparisons.

### 12. Product of Array Except Self — Medium
**Companies:** Amazon, Meta, Microsoft, Apple, Google
**Problem:** Return an array where `out[i]` is the product of all items except `a[i]`, without using division, in O(n). Example: `[1,2,3,4]` -> `[24,12,8,6]`.
**Intuition:** The product without me = (product of everything on my left) x (product of everything on my right). Compute left products in one pass, right products in another.
**Approach 1 - Brute force:** For each i, multiply all others. Time O(n^2), Space O(1) extra.
**Approach 2 - Better:** Two arrays, `left[]` and `right[]`, then multiply. Time O(n), Space O(n). Division (total product / a[i]) also fails when a zero is present.
**Approach 3 - Optimal:** Store left products in the output array itself, then sweep from the right with one running variable.

Trace on `[1,2,3,4]`:
```
left pass:  out = [1, 1, 2, 6]       (out[i] = product of items before i)
right pass: right=1: out[3]=6*1=6,  right=4
            out[2]=2*4=8,           right=12
            out[1]=1*12=12,         right=24
            out[0]=1*24=24          out = [24,12,8,6]
```

```js
function productExceptSelf(a) {
  const n = a.length, out = new Array(n).fill(1);
  let left = 1;
  for (let i = 0; i < n; i++) { out[i] = left; left *= a[i]; }
  let right = 1;
  for (let i = n - 1; i >= 0; i--) { out[i] *= right; right *= a[i]; }
  return out;
}
```

**Complexity:** Time O(n), Space O(1) extra (the output array does not count).
**Watch out:** Zeros work naturally here. JS gotcha: `0 * -3` is `-0`, so an answer can print as `-0`. `assert.deepStrictEqual` and `Object.is` can tell it apart from `0`; add `+ 0` (as in `x + 0`) if exact zeros matter. Very large products can pass 2^53 and lose precision; mention BigInt or the problem's stated bound.

## Common Mistakes
- **"Sorting always works."** It costs O(n log n) and often breaks the "in place" or "O(n)" requirement. Ask what the target is first.
- **Using `shift()` / `unshift()` / `splice()` inside a loop.** Each is O(n), so the loop becomes O(n^2). Use pointers.
- **Initializing a max with 0.** Arrays can be all negative. Start with `-Infinity` or `a[0]`.
- **Initializing `best = 0` in Kadane.** Wrong for all-negative input. Start from `a[0]`.
- **Forgetting `k %= n` in rotate,** or dividing by a zero length.
- **Off by one in the write pointer.** `w` is the COUNT of kept items and also the next free index. Be clear about which one you return.
- **Modifying the array while a `for...of` or `forEach` runs over it.** Use index loops for in-place work.
- **Assuming the array is non-empty.** Say the edge case out loud, then guard it.
- **Comparing arrays with `==` or `===` in tests.** Compare element by element (or use `assert.deepStrictEqual`).

## Interview Tips
- Start by restating the problem with one example, then ask: sorted or not? duplicates? negatives? size limits? in place? can I modify the input?
- Say the brute force first with its cost, then say which part is wasted work. This shows the path to the optimal answer.
- Name the technique: "this is read/write pointers", "this is Kadane", "this is Boyer-Moore". It tells the interviewer you have seen the pattern.
- Dry-run your code on a tiny example, plus the edge cases: empty, one item, all equal, all negative, already sorted, k larger than n.
- Common follow-ups: "print the subarray" (Kadane with indexes), "what if a majority is not guaranteed?" (verify pass), "what if the array is huge and does not fit in memory?" (streaming works for Boyer-Moore, Kadane, running min), "do it with fewer writes", "rotate left instead of right".
- State complexity at the end for both time AND space, and say whether the output array counts.

## Practice List

**Easy**
- Largest Element in Array
- Check if Array Is Sorted and Rotated
- Single Number
- Max Consecutive Ones
- Merge Sorted Array
- Contains Duplicate
- Plus One
- Find All Numbers Disappeared in an Array

**Medium**
- Set Matrix Zeroes
- Rotate Image
- Spiral Matrix
- Longest Consecutive Sequence
- Find the Duplicate Number
- Majority Element II
- Maximum Product Subarray
- Leaders in an Array
- Pascal's Triangle
- Subarray Sum Equals K
- Next Greater Element (array version)

**Hard**
- First Missing Positive
- Trapping Rain Water
- Merge Intervals
- Count of Smaller Numbers After Self
- Reverse Pairs
- Max Value of Equation

## Key Takeaways
- Array access is **O(1) by index**, but inserting or deleting in the middle is **O(n)**. Avoid shifting inside loops.
- **Read/write pointers** edit an array in place in O(n): remove duplicates, move zeroes, filter.
- **Reversal** is the tool for rotation: reverse all, then reverse the two parts.
- **Kadane's rule:** drop the running sum when it goes negative. Never start the best at 0.
- **Boyer-Moore voting** finds a majority in O(1) space; verify the candidate when a majority is not guaranteed.
- **Prefix + suffix passes** replace "everything except me" without division.
- **Next permutation** = find the pivot from the right, swap with the next bigger item, reverse the tail.
- Always test **empty, single item, all equal, negatives** before saying "done".

---
[← Previous: Math Basics](../../00-foundations/04-math-basics/README.md) | [Next: Sorting →](../02-sorting/README.md) | [Back to Core Linear Structures Index](../README.md)
