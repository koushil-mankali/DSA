# Binary Search

> **Estimated Time:** 6 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - Arrays](../01-arrays/README.md), [02 - Sorting](../02-sorting/README.md), [02 - Complexity Analysis](../../00-foundations/02-complexity-analysis/README.md)

---

## What You'll Learn
- You can write a binary search that never loops forever and never misses by one.
- You can use **lower bound** and **upper bound** to answer "first", "last", "count" and "insert position" questions.
- You can search rotated arrays, mountains, peaks and 2D matrices.
- You can spot a **binary search on the answer** problem and write the yes/no check for it.
- You can explain why binary search works on any monotonic yes/no question, not only on sorted arrays.

## The Big Idea
Think of guessing a number from 1 to 100 while a friend says "higher" or "lower". You do not guess 1, 2, 3. You guess 50, and every answer throws away half of the numbers. In at most 7 guesses you are done. A dictionary works the same way: you open the middle, look at the word, and go left or right.

Binary search keeps a range where the answer must live. You look at the middle, decide which half cannot contain the answer, and cut it away. The range halves each step, so the cost is **O(log n)**. For one billion items that is about 30 steps.

The real insight is that you do not need a sorted array. You need a **monotonic** question: a yes/no question where the answers look like `NO NO NO NO YES YES YES`. If you can write such a question, you can binary search it. The array can be sorted values, a rotated array, a matrix, or even a plain range of numbers like "speeds from 1 to 1000".

Interviewers love it because it is short, and tiny mistakes (off by one, infinite loop, overflow) show up fast. It also tests whether you can see a hidden monotonic pattern.

## How to Spot It
| If the problem says ... | think ... |
|---|---|
| "sorted array", "find target" | classic binary search |
| "first / last position", "count of x", "insert position" | lower bound / upper bound |
| "rotated sorted array" | one half is always sorted |
| "O(log n)" is required | binary search |
| "minimum of the maximum", "maximum of the minimum" | binary search on the answer |
| "smallest speed / capacity / days such that it is possible" | binary search on the answer |
| "split into k parts", "k workers", "k cows" | binary search on the answer + greedy check |
| "peak", "mountain" | compare `mid` with `mid + 1` |
| "sorted rows and columns" | 2D: flatten, or walk from a corner |
| "find the k-th ..." in sorted data | binary search on index or on value |

## Core Concepts / Template

### The classic template
Keep the range `[lo, hi]` inclusive. The loop runs while the range has at least one item. The full code is in Problem 1 below.

Why `lo + ((hi - lo) >> 1)`? In languages with 32-bit ints, `(lo + hi) / 2` can overflow. JavaScript numbers do not overflow here, but it is a good habit and interviewers notice.

Why `mid + 1` and `mid - 1`? We already checked `mid`. If we wrote `lo = mid`, the range might never shrink and we loop forever.

### Lower bound and upper bound (the template you will use most)
Here the range is half open: `[lo, hi)`, and `hi` starts at `n` because "the answer is n" (nothing found) is a valid answer.

- **lowerBound(x)**: the first index whose value is `>= x`.
- **upperBound(x)**: the first index whose value is `> x`.

```js
function lowerBound(nums, x) {
  let lo = 0, hi = nums.length;
  while (lo < hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (nums[mid] >= x) hi = mid;
    else lo = mid + 1;
  }
  return lo;
}

function upperBound(nums, x) {
  let lo = 0, hi = nums.length;
  while (lo < hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (nums[mid] > x) hi = mid;
    else lo = mid + 1;
  }
  return lo;
}
```

Notice the pattern: the check is `NO NO NO YES YES YES`, and we look for the first YES. When `ok(mid)` is true, `mid` might be the answer, so `hi = mid` (keep it). When false, `mid` is useless, so `lo = mid + 1`. The loop is `lo < hi`, so the range always shrinks.

```
nums = [1, 2, 2, 2, 5, 7]     x = 2
index   0  1  2  3  4  5
>= 2    N  Y  Y  Y  Y  Y   -> lowerBound = 1
>  2    N  N  N  N  Y  Y   -> upperBound = 4
count of 2 = 4 - 1 = 3
```

### Binary search on the answer (the general "first true" template)
Many problems ask "what is the smallest X so that something works?". If bigger X never makes it harder, then the answers form `NO NO YES YES YES`. Search X directly.

```js
function firstTrue(lo, hi, ok) {
  while (lo < hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (ok(mid)) hi = mid;
    else lo = mid + 1;
  }
  return lo;
}
```

Recipe:
1. Decide what the answer is (a speed, a capacity, a distance) and its smallest and largest possible values (`lo`, `hi`).
2. Write `ok(x)`: "is x enough?" Usually a simple greedy loop, O(n).
3. Call `firstTrue`. Total time: O(n log(range)).

For "largest X that works" (maximize the minimum), flip it: use `lo = mid` when ok, and round `mid` **up** with `Math.ceil`, or you will loop forever.

### Off-by-one safety checklist
| Choice | Use when | Loop | Moves |
|---|---|---|---|
| Closed `[lo, hi]` | find exact value, return inside loop | `lo <= hi` | `mid + 1`, `mid - 1` |
| Half open `[lo, hi)` | first true, lower bound | `lo < hi` | `hi = mid`, `lo = mid + 1` |
| Last true | largest x that works | `lo < hi` | `lo = mid` (needs ceil mid), `hi = mid - 1` |

Pick one template and stay in it. Before you submit, test with arrays of size 0, 1 and 2. Size 2 is the one that catches infinite loops.

## Problems (easy -> hard)

### 1. Binary Search — Easy
**Companies:** Amazon, Microsoft, Apple
**Problem:** Given a sorted array of distinct integers and a target, return the index of the target or -1. Example: `[-1,0,3,5,9,12]`, target 9 -> 4.
**Intuition:** Compare the middle with the target. If the middle is too small, everything on its left is too small too, so drop it.
**Approach 1 - Brute force:** Scan from left to right. Time O(n), Space O(1).
**Approach 2 - Optimal:** Halve the range each step.

| lo | hi | mid | nums[mid] | action |
|---|---|---|---|---|
| 0 | 5 | 2 | 3 | 3 < 9, lo = 3 |
| 3 | 5 | 4 | 9 | found, return 4 |

```js
function search(nums, target) {
  let lo = 0, hi = nums.length - 1;
  while (lo <= hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (nums[mid] === target) return mid;
    if (nums[mid] < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;
}
```

**Complexity:** Time O(log n), Space O(1).
**Watch out:** Use `lo <= hi` in the closed template. With `lo < hi` you miss a one-item range.

### 2. Search Insert Position — Easy
**Companies:** Amazon, Google, Microsoft
**Problem:** Return the index of the target in a sorted array, or the index where it would be inserted to keep order. Example: `[1,3,5,6]`, target 5 -> 2; target 2 -> 1; target 7 -> 4.
**Intuition:** "Where would it go" is exactly "the first index with a value `>= target`". That is lower bound.
**Approach 1 - Brute force:** Scan until `nums[i] >= target`. Time O(n), Space O(1).
**Approach 2 - Optimal:** Lower bound. For `[1,3,5,6]` and 2: `>= 2` gives `N Y Y Y`, first Y is index 1.

```js
const searchInsert = (nums, target) => lowerBound(nums, target);
```

(`lowerBound` is the function from the template section.)

**Complexity:** Time O(log n), Space O(1).
**Watch out:** The answer can be `n` (insert at the end). That is why `hi` starts at `n`, not `n - 1`.

### 3. First and Last Position of Element in Sorted Array (and Count Occurrences) — Medium
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** In a sorted array with duplicates, return `[first, last]` index of the target, or `[-1, -1]`. Example: `[5,7,7,8,8,10]`, target 8 -> `[3,4]`. The count of occurrences is `last - first + 1`.
**Intuition:** First position is lower bound. Last position is upper bound minus one. Two binary searches.
**Approach 1 - Brute force:** Scan for first and last. Time O(n), Space O(1). Too slow when the array is huge with many duplicates.
**Approach 2 - Optimal:** `first = lowerBound(x)`. If `nums[first] !== x`, the target is missing. Else `last = upperBound(x) - 1`.

```
nums = [5,7,7,8,8,10], x = 8
lowerBound = 3 (first >= 8), upperBound = 5 (first > 8)
answer [3, 5-1] = [3, 4], count = 5 - 3 = 2
```

```js
function searchRange(nums, target) {
  const first = lowerBound(nums, target);
  if (first === nums.length || nums[first] !== target) return [-1, -1];
  return [first, upperBound(nums, target) - 1];
}
```

Count of occurrences is one line: `upperBound(nums, x) - lowerBound(nums, x)`.

**Complexity:** Time O(log n), Space O(1).
**Watch out:** Check `first === nums.length` before reading `nums[first]`.

### 4. Floor and Ceil in a Sorted Array — Easy
**Companies:** Amazon, Microsoft
**Problem:** Floor is the largest value `<= x`, ceil is the smallest value `>= x`. Return `[floor, ceil]`, using -1 if none exists. Example: `[3,4,4,7,8,10]`, x = 5 -> `[4,7]`.
**Intuition:** Ceil sits at lower bound. Floor sits just before upper bound.
**Approach 1 - Brute force:** Scan and track the best. Time O(n), Space O(1).
**Approach 2 - Optimal:** `ceil = nums[lowerBound(x)]`, `floor = nums[upperBound(x) - 1]`. For x = 5: lowerBound = 3 (value 7), upperBound = 3, so floor is `nums[2] = 4`.

```js
function floorCeil(nums, x) {
  const i = lowerBound(nums, x);       // first >= x
  const j = upperBound(nums, x) - 1;   // last <= x
  return [j >= 0 ? nums[j] : -1, i < nums.length ? nums[i] : -1];
}
```

**Complexity:** Time O(log n), Space O(1).
**Watch out:** Index -1 (floor does not exist) and index n (ceil does not exist) are both valid results, guard them.

### 5. Sqrt(x) — Easy
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** Return the integer part of the square root of `x` without using `sqrt`. Example: 8 -> 2.
**Intuition:** `m*m <= x` is `YES YES YES NO NO`. We want the last YES. This is binary search on the answer.
**Approach 1 - Brute force:** Try m = 0, 1, 2, ... until `m*m > x`. Time O(sqrt x), Space O(1).
**Approach 2 - Optimal:** Search m in `[0, x]` for the largest `m` with `m*m <= x`.

| lo | hi | mid | mid*mid <= 8? | action |
|---|---|---|---|---|
| 0 | 8 | 4 | no | hi = 3 |
| 0 | 3 | 2 | yes | lo = 2 |
| 2 | 3 | 3 | no | hi = 2, stop, answer 2 |

```js
function mySqrt(x) {
  let lo = 0, hi = x; // answer = largest m with m*m <= x
  while (lo < hi) {
    const mid = lo + Math.ceil((hi - lo) / 2);
    if (mid * mid <= x) lo = mid;
    else hi = mid - 1;
  }
  return lo;
}
```

**Complexity:** Time O(log x), Space O(1).
**Watch out:** "Last true" needs `Math.ceil` for `mid`. With a floor, `lo = mid` never moves when `hi = lo + 1`: infinite loop.

### 6. Nth Root of an Integer — Medium
**Companies:** Amazon, Google
**Problem:** Given `n` and `m`, return the integer `r` with `r^n = m`, or -1 if none exists. Example: n = 3, m = 27 -> 3; n = 4, m = 69 -> -1.
**Intuition:** `mid^n` grows when `mid` grows, so compare it with `m` and move left or right. This is a sqrt problem with a power.
**Approach 1 - Brute force:** Try r = 1, 2, ... computing `r^n`. Time O(m * n), Space O(1).
**Approach 2 - Optimal:** Binary search r in `[1, m]`. Compute `mid^n` with an early exit once the product passes `m`, so the number never gets huge.

```js
function nthRoot(n, m) {
  let lo = 1, hi = m;
  while (lo <= hi) {
    const mid = lo + ((hi - lo) >> 1);
    let p = 1, cmp = 0;
    for (let i = 0; i < n; i++) {
      p *= mid;
      if (p > m) { cmp = 1; break; } // stop early, no overflow
    }
    if (cmp === 0) cmp = p === m ? 0 : -1;
    if (cmp === 0) return mid;
    if (cmp < 0) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;
}
```

**Complexity:** Time O(n log m), Space O(1).
**Watch out:** Do not call `Math.pow` and compare floats for big values. Multiply in a loop and stop early.

### 7. Search in Rotated Sorted Array — Medium
**Companies:** Amazon, Meta, Google, Microsoft, Apple
**Problem:** A sorted array of distinct values was rotated at an unknown pivot. Find the target's index or -1. Example: `[4,5,6,7,0,1,2]`, target 0 -> 4.
**Intuition:** Cut anywhere and at least one half is still sorted. Find the sorted half, check if the target lies inside it, and go there. Otherwise go to the other half.
**Approach 1 - Brute force:** Scan. Time O(n), Space O(1).
**Approach 2 - Better:** Find the pivot with a binary search, then binary search one side. Time O(log n), two passes.
**Approach 3 - Optimal:** One pass, as below.

| lo | hi | mid | sorted half | target 0 inside? | action |
|---|---|---|---|---|---|
| 0 | 6 | 3 (7) | left `[4..7]` | no | lo = 4 |
| 4 | 6 | 5 (1) | left `[0,1]` | yes (0 >= 0 and 0 < 1) | hi = 4 |
| 4 | 4 | 4 (0) | | found | return 4 |

```js
function searchRotated(nums, target) {
  let lo = 0, hi = nums.length - 1;
  while (lo <= hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (nums[mid] === target) return mid;
    if (nums[lo] <= nums[mid]) { // left half sorted
      if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;
      else lo = mid + 1;
    } else {                     // right half sorted
      if (nums[mid] < target && target <= nums[hi]) lo = mid + 1;
      else hi = mid - 1;
    }
  }
  return -1;
}
```

**Complexity:** Time O(log n), Space O(1).
**Watch out:** Use `nums[lo] <= nums[mid]` (with `=`) for "left is sorted". With two items, `lo === mid`.

### 8. Search in Rotated Sorted Array II (with duplicates) — Medium
**Companies:** Amazon, Microsoft, Meta
**Problem:** Same as before, but values may repeat. Return true or false. Example: `[2,5,6,0,0,1,2]`, target 0 -> true.
**Intuition:** Duplicates can hide which half is sorted: `[1,0,1,1,1]` has `nums[lo] == nums[mid] == nums[hi]`. We cannot tell, so shrink both ends by one and try again.
**Approach 1 - Brute force:** Scan. Time O(n).
**Approach 2 - Optimal:** Same as problem 7, plus one extra case: if the three values are equal, do `lo++; hi--`.

```js
function searchRotatedDup(nums, target) {
  let lo = 0, hi = nums.length - 1;
  while (lo <= hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (nums[mid] === target) return true;
    if (nums[lo] === nums[mid] && nums[mid] === nums[hi]) { lo++; hi--; continue; }
    if (nums[lo] <= nums[mid]) {
      if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;
      else lo = mid + 1;
    } else {
      if (nums[mid] < target && target <= nums[hi]) lo = mid + 1;
      else hi = mid - 1;
    }
  }
  return false;
}
```

**Complexity:** Time O(log n) on average, **O(n) worst case** (an array of all equal values), Space O(1).
**Watch out:** Say the worst case out loud. It is a known follow-up: "why is it O(n) now?" Because duplicates remove the ability to discard half.

### 9. Find Minimum in Rotated Sorted Array — Medium
**Companies:** Amazon, Meta, Microsoft, Google
**Problem:** Return the smallest value in a rotated sorted array of distinct values. Example: `[3,4,5,1,2]` -> 1.
**Intuition:** Compare `mid` with `hi`. If `nums[mid] > nums[hi]`, the drop (the minimum) is on the right of `mid`. Otherwise `mid` could itself be the minimum, so keep it.
**Approach 1 - Brute force:** Scan for the minimum. Time O(n).
**Approach 2 - Optimal:** Half-open "first true" search. Question: "is `nums[mid] <= nums[hi]`?".

| lo | hi | mid | nums[mid] vs nums[hi] | action |
|---|---|---|---|---|
| 0 | 4 | 2 (5) | 5 > 2 | lo = 3 |
| 3 | 4 | 3 (1) | 1 <= 2 | hi = 3, stop, answer 1 |

```js
function findMin(nums) {
  let lo = 0, hi = nums.length - 1;
  while (lo < hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (nums[mid] > nums[hi]) lo = mid + 1; // min is on the right
    else hi = mid;                          // mid may be the min
  }
  return nums[lo];
}
```

**Complexity:** Time O(log n), Space O(1).
**Watch out:** Compare with `hi`, not with `lo`. Comparing with `lo` fails when the array is not rotated. With duplicates, use `else if (nums[mid] === nums[hi]) hi--`.

### 10. Single Element in a Sorted Array — Medium
**Companies:** Amazon, Google, Meta
**Problem:** Every value appears twice except one. The array is sorted. Find the single one in O(log n). Example: `[1,1,2,3,3,4,4,8,8]` -> 2.
**Intuition:** Before the single element, pairs start at even indexes `(0,1), (2,3)`. After it, pairs start at odd indexes. So look at the pair at `mid` and decide which side is broken.
**Approach 1 - Brute force:** XOR all values. Time O(n), Space O(1).
**Approach 2 - Optimal:** Make `mid` even. If `nums[mid] === nums[mid+1]`, the pairs on the left are fine, so the single is to the right: `lo = mid + 2`. Else the single is at `mid` or to its left: `hi = mid`.

| lo | hi | mid (even) | pair ok? | action |
|---|---|---|---|---|
| 0 | 8 | 4 (3 vs 4) | no | hi = 4 |
| 0 | 4 | 2 (2 vs 3) | no | hi = 2 |
| 0 | 2 | 0 (1 vs 1) | yes | lo = 2, answer `nums[2] = 2` |

```js
function singleNonDuplicate(nums) {
  let lo = 0, hi = nums.length - 1;
  while (lo < hi) {
    let mid = lo + ((hi - lo) >> 1);
    if (mid % 2 === 1) mid--;              // make mid even
    if (nums[mid] === nums[mid + 1]) lo = mid + 2; // pairs intact on the left
    else hi = mid;
  }
  return nums[lo];
}
```

**Complexity:** Time O(log n), Space O(1).
**Watch out:** The array length is always odd, so `mid + 1` is safe when `mid` is even and `< hi`.

### 11. Find Peak Element — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** A peak is greater than its neighbors. Return the index of any peak. Neighbors are never equal and `nums[-1] = nums[n] = -infinity`. Example: `[1,2,1,3,5,6,4]` -> 1 or 5.
**Intuition:** If `nums[mid] < nums[mid+1]`, we are going uphill, so a peak must exist on the right (the climb ends somewhere, at worst at the end). Otherwise a peak exists at `mid` or to its left.
**Approach 1 - Brute force:** Scan. Time O(n).
**Approach 2 - Optimal:** Follow the uphill direction.

| lo | hi | mid | compare | action |
|---|---|---|---|---|
| 0 | 6 | 3 | 3 < 5 | lo = 4 |
| 4 | 6 | 5 | 6 > 4 | hi = 5 |
| 4 | 5 | 4 | 5 < 6 | lo = 5, answer 5 |

```js
function findPeakElement(nums) {
  let lo = 0, hi = nums.length - 1;
  while (lo < hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (nums[mid] < nums[mid + 1]) lo = mid + 1;
    else hi = mid;
  }
  return lo;
}
```

**Complexity:** Time O(log n), Space O(1).
**Watch out:** This works even though the array is not sorted. The proof is "uphill must end", not "sorted".

### 12. Koko Eating Bananas — Medium
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** There are piles of bananas and `h` hours. Each hour Koko picks one pile and eats up to `k` bananas from it. Find the smallest `k` that finishes in `h` hours. Example: `[3,6,7,11]`, h = 8 -> 4.
**Intuition:** A bigger `k` never makes it slower. So "can she finish with speed k?" is `NO NO YES YES`. Hours needed for speed `k` is the sum of `ceil(pile / k)`.
**Approach 1 - Brute force:** Try k = 1, 2, 3, ... Time O(n * max).
**Approach 2 - Optimal:** Binary search k in `[1, max(piles)]`. Speed `max(piles)` always works (one hour per pile).

| k | hours | <= 8? |
|---|---|---|
| 3 | 1+2+3+4 = 10 | no |
| 4 | 1+2+2+3 = 8 | yes |

```js
function minEatingSpeed(piles, h) {
  const ok = (k) => piles.reduce((s, p) => s + Math.ceil(p / k), 0) <= h;
  return firstTrue(1, Math.max(...piles), ok);
}
```

**Complexity:** Time O(n log max), Space O(1).
**Watch out:** Use `ceil`, not floor. Low bound is 1 (not 0, division by zero).

### 13. Capacity to Ship Packages Within D Days — Medium
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** Packages must ship in the given order, in at most `D` days, each day a ship carries up to `cap`. Find the smallest `cap`. Example: weights 1..10, D = 5 -> 15.
**Intuition:** A bigger ship never needs more days. Check a capacity greedily: load packages until the next one does not fit, then start a new day.
**Approach 1 - Brute force:** Try every capacity from `max` upward. Time O(n * sum).
**Approach 2 - Optimal:** Binary search the capacity in `[max(weights), sum(weights)]`. The low bound is `max` because one package must fit.

| cap | days |
|---|---|
| 15 | `1..5`, `6..7`, `8`, `9`, `10` = 5 days, ok |
| 14 | `1..4`, `5..6`, `7`, `8`, `9`, `10` = 6 days, no |

```js
function shipWithinDays(weights, days) {
  const ok = (cap) => {
    let d = 1, cur = 0;
    for (const w of weights) {
      if (cur + w > cap) { d++; cur = 0; }
      cur += w;
    }
    return d <= days;
  };
  return firstTrue(Math.max(...weights), weights.reduce((a, b) => a + b, 0), ok);
}
```

**Complexity:** Time O(n log sum), Space O(1).
**Watch out:** Start `lo` at the largest single weight, not 1. A smaller capacity can never ship that package.

### 14. Minimum Number of Days to Make m Bouquets — Medium
**Companies:** Amazon, Google, Meta
**Problem:** `bloomDay[i]` is the day flower `i` blooms. A bouquet needs `k` **adjacent** bloomed flowers. Find the earliest day we can make `m` bouquets, or -1. Example: `[1,10,3,10,2]`, m = 3, k = 1 -> 3.
**Intuition:** More days means more bloomed flowers, so it is monotonic. For one day, scan and count runs of `k` bloomed flowers.
**Approach 1 - Brute force:** Try every day. Time O(n * range).
**Approach 2 - Optimal:** If `m * k > n`, return -1. Else binary search the day in `[min, max]` of `bloomDay`.

```js
function minDays(bloomDay, m, k) {
  if (m * k > bloomDay.length) return -1;
  const ok = (day) => {
    let bouquets = 0, run = 0;
    for (const b of bloomDay) {
      if (b <= day) { if (++run === k) { bouquets++; run = 0; } }
      else run = 0;
    }
    return bouquets >= m;
  };
  return firstTrue(Math.min(...bloomDay), Math.max(...bloomDay), ok);
}
```

**Complexity:** Time O(n log(max - min)), Space O(1).
**Watch out:** A gap resets the run (`run = 0`). Forgetting this makes non-adjacent flowers count together. Also `m * k` can be big, so check it first.

### 15. Find the Smallest Divisor Given a Threshold — Medium
**Companies:** Amazon, Google
**Problem:** Choose a divisor `d`, sum `ceil(nums[i] / d)`. Find the smallest `d` so the sum is `<= threshold`. Example: `[1,2,5,9]`, threshold 6 -> 5.
**Intuition:** Bigger `d` gives a smaller sum. It is the same shape as Koko.
**Approach 1 - Brute force:** Try d = 1, 2, ... Time O(n * max).
**Approach 2 - Optimal:** Binary search `d` in `[1, max(nums)]`.

```js
function smallestDivisor(nums, threshold) {
  const ok = (d) => nums.reduce((s, x) => s + Math.ceil(x / d), 0) <= threshold;
  return firstTrue(1, Math.max(...nums), ok);
}
```

**Complexity:** Time O(n log max), Space O(1).
**Watch out:** `d = max(nums)` always gives sum `n`, so the threshold must be at least `n` (the problem guarantees it).

### 16. Aggressive Cows (Magnetic Force Between Two Balls) — Medium
**Companies:** Google, Amazon, Flipkart
**Problem:** Place `c` cows in stalls at given positions so the **smallest distance** between any two cows is as large as possible. Return that distance. Example: stalls `[1,2,4,8,9]`, c = 3 -> 3. (LeetCode "Magnetic Force Between Two Balls" is the same problem.)
**Intuition:** "Maximize the minimum" is the classic sign. If distance `d` works, any smaller distance works too: `YES YES YES NO NO`. Check `d` greedily: put a cow in the first stall, then in the next stall that is at least `d` away.
**Approach 1 - Brute force:** Try every subset of `c` stalls. Time O(C(n, c)).
**Approach 2 - Better:** Try every `d` from 1 to the range, check each in O(n).
**Approach 3 - Optimal:** Sort, then binary search `d` in `[1, last - first]` for the last YES.

| d | cows placed | enough? |
|---|---|---|
| 3 | stalls 1, 4, 8 | yes (3 cows) |
| 4 | stalls 1, 8 | no (2 cows) |

```js
function aggressiveCows(stalls, cows) {
  const a = [...stalls].sort((x, y) => x - y);
  const ok = (d) => { // can we place all cows at least d apart?
    let count = 1, last = a[0];
    for (let i = 1; i < a.length; i++) {
      if (a[i] - last >= d) { count++; last = a[i]; }
    }
    return count >= cows;
  };
  let lo = 1, hi = a[a.length - 1] - a[0]; // largest d with ok(d)
  while (lo < hi) {
    const mid = lo + Math.ceil((hi - lo) / 2);
    if (ok(mid)) lo = mid;
    else hi = mid - 1;
  }
  return lo;
}
```

**Complexity:** Time O(n log n + n log range), Space O(1) beyond the sorted copy.
**Watch out:** "Last true" needs `Math.ceil` for `mid`. Also sort first.

### 17. Split Array Largest Sum (Allocate Books, Painter's Partition) — Hard
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Split an array into `k` continuous parts to minimize the largest part sum. Example: `[7,2,5,10,8]`, k = 2 -> 18 (`[7,2,5]` and `[10,8]`).
**Intuition:** Guess the largest allowed sum `cap`. Greedily fill each part until the next item would exceed it. If you need `<= k` parts, `cap` works. Bigger `cap` needs fewer parts, so it is monotonic. **Allocate Books** (students are parts, pages are values) and **Painter's Partition** (painters are parts, board lengths are values) are the exact same problem with new words.
**Approach 1 - Brute force:** Try all ways to cut, or DP. Time O(n^2 * k) with DP.
**Approach 2 - Optimal:** Binary search `cap` in `[max(nums), sum(nums)]`. It is the same function as ship capacity (problem 13).

| cap | parts (greedy) | <= 2? |
|---|---|---|
| 17 | `[7,2,5]`, `[10]`, `[8]` = 3 | no |
| 18 | `[7,2,5]`, `[10,8]` = 2 | yes |

```js
function splitArray(nums, k) {
  const ok = (cap) => {
    let parts = 1, cur = 0;
    for (const x of nums) {
      if (cur + x > cap) { parts++; cur = 0; }
      cur += x;
    }
    return parts <= k;
  };
  return firstTrue(Math.max(...nums), nums.reduce((a, b) => a + b, 0), ok);
}
```

**Complexity:** Time O(n log sum), Space O(1).
**Watch out:** In Allocate Books, if `k > n` return -1 (some student gets no book). Parts must be contiguous, that is why greedy is correct.

### 18. Kth Missing Positive Number — Easy
**Companies:** Meta, Amazon, Google
**Problem:** A sorted array of distinct positive integers is given. Find the `k`-th positive integer that is missing from it. Example: `[2,3,4,7,11]`, k = 5 -> 9.
**Intuition:** At index `i`, the count of missing numbers before `arr[i]` is `arr[i] - (i + 1)`. That count never decreases, so it is monotonic. Find the first index where missing `>= k`. Then the answer sits `k` steps after the `lo` items that come before it.
**Approach 1 - Brute force:** Walk numbers 1, 2, 3, ... and count the missing ones. Time O(n + k).
**Approach 2 - Optimal:** Binary search on that count.

| index | arr[i] | missing before |
|---|---|---|
| 0 | 2 | 1 |
| 1 | 3 | 1 |
| 2 | 4 | 1 |
| 3 | 7 | 3 |
| 4 | 11 | 6 |

For k = 5 the first index with missing `>= 5` is 4, so `lo = 4` and the answer is `4 + 5 = 9`.

```js
function findKthPositive(arr, k) {
  // missing(i) = arr[i] - (i+1). find first i with missing >= k
  let lo = 0, hi = arr.length;
  while (lo < hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (arr[mid] - (mid + 1) >= k) hi = mid;
    else lo = mid + 1;
  }
  return lo + k; // lo elements are before the answer
}
```

**Complexity:** Time O(log n), Space O(1).
**Watch out:** If no index qualifies, `lo = n` and the formula still gives `n + k`. No special case needed.

### 19. Median of Two Sorted Arrays — Hard
**Companies:** Google, Amazon, Meta, Microsoft, Apple
**Problem:** Given two sorted arrays, return the median of all values in O(log(min(m, n))). Example: `[1,3]`, `[2]` -> 2; `[1,2]`, `[3,4]` -> 2.5.
**Intuition:** Do not merge. Split both arrays so that the left side has exactly half of all items and every left item is `<=` every right item. If you pick `i` items from A, you must take `half - i` from B. Binary search `i` until the borders cross correctly.
**Approach 1 - Brute force:** Merge, take the middle. Time O(m + n), Space O(m + n).
**Approach 2 - Better:** Merge with two pointers only up to the middle. Time O(m + n), Space O(1).
**Approach 3 - Optimal:** Binary search the cut `i` in the smaller array. A split is correct when `A[i-1] <= B[j]` and `B[j-1] <= A[i]`. If `A[i-1] > B[j]`, we took too many from A: `hi = i - 1`. Else take more: `lo = i + 1`.

```
A = [1,3]  B = [2]   half = 2
i=1, j=1:  A left = [1], A right = [3];  B left = [2], B right = []
1 <= inf  and  2 <= 3  -> valid. odd total, median = max(1, 2) = 2
```

```js
function findMedianSortedArrays(A, B) {
  if (A.length > B.length) return findMedianSortedArrays(B, A);
  const m = A.length, n = B.length, half = (m + n + 1) >> 1;
  let lo = 0, hi = m;
  while (lo <= hi) {
    const i = lo + ((hi - lo) >> 1), j = half - i;
    const aL = i === 0 ? -Infinity : A[i - 1], aR = i === m ? Infinity : A[i];
    const bL = j === 0 ? -Infinity : B[j - 1], bR = j === n ? Infinity : B[j];
    if (aL <= bR && bL <= aR) {
      const left = Math.max(aL, bL);
      if ((m + n) % 2 === 1) return left;
      return (left + Math.min(aR, bR)) / 2;
    }
    if (aL > bR) hi = i - 1;
    else lo = i + 1;
  }
}
```

**Complexity:** Time O(log(min(m, n))), Space O(1).
**Watch out:** Binary search the **shorter** array, or `j` can go negative. Use `-Infinity` / `Infinity` for borders that fall outside an array.

### 20. Search a 2D Matrix — Medium
**Companies:** Amazon, Microsoft, Meta, Google
**Problem:** Each row is sorted, and the first value of a row is greater than the last value of the previous row. Does the target exist? Example: `[[1,3,5,7],[10,11,16,20],[23,30,34,60]]`, target 3 -> true.
**Intuition:** Read the matrix row by row and it is one long sorted array. Pretend it is flat: index `mid` is row `floor(mid / C)`, column `mid % C`.
**Approach 1 - Brute force:** Scan every cell. Time O(R * C).
**Approach 2 - Better:** Binary search the row, then binary search inside it. Time O(log R + log C).
**Approach 3 - Optimal:** One binary search on `R * C` virtual items. Same big-O, simpler code.

```js
function searchMatrix(matrix, target) {
  const R = matrix.length, C = matrix[0].length;
  let lo = 0, hi = R * C - 1;
  while (lo <= hi) {
    const mid = lo + ((hi - lo) >> 1);
    const v = matrix[Math.floor(mid / C)][mid % C];
    if (v === target) return true;
    if (v < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return false;
}
```

**Complexity:** Time O(log(R * C)), Space O(1).
**Watch out:** Divide by the number of **columns**, not rows.

### 21. Search a 2D Matrix II — Medium
**Companies:** Amazon, Google, Microsoft, Apple
**Problem:** Rows are sorted left to right and columns are sorted top to bottom (but rows are not chained). Does the target exist? Example: the 5x5 matrix `[[1,4,7,11,15],[2,5,8,12,19],[3,6,9,16,22],[10,13,14,17,24],[18,21,23,26,30]]`, target 5 -> true.
**Intuition:** Start at the **top-right** corner. If the value is too big, the whole column below it is bigger too, so go left. If too small, the whole row to its left is smaller too, so go down. Each step kills a row or a column. (This is a "staircase" search, not a classic halving, but it is expected in this topic.)
**Approach 1 - Brute force:** Scan. Time O(R * C).
**Approach 2 - Better:** Binary search each row. Time O(R log C).
**Approach 3 - Optimal:** Staircase walk.

| cell | value | vs 5 | move |
|---|---|---|---|
| (0,4) | 15 | bigger | left |
| (0,3) | 11 | bigger | left |
| (0,2) | 7 | bigger | left |
| (0,1) | 4 | smaller | down |
| (1,1) | 5 | equal | found |

```js
function searchMatrixII(matrix, target) {
  let r = 0, c = matrix[0].length - 1; // start top-right
  while (r < matrix.length && c >= 0) {
    const v = matrix[r][c];
    if (v === target) return true;
    if (v > target) c--;
    else r++;
  }
  return false;
}
```

**Complexity:** Time O(R + C), Space O(1).
**Watch out:** Start at top-right or bottom-left. Top-left does not work: both moves go up in value.

### 22. Row with Maximum Number of 1s — Easy
**Companies:** Amazon, Flipkart, Microsoft
**Problem:** Each row of a binary matrix is sorted (zeros then ones). Return the index of the row with the most 1s (the smallest index if tied), or -1 if there are no ones. Example: `[[0,1,1],[0,0,1],[1,1,1]]` -> 2.
**Intuition:** In a sorted 0/1 row, the number of ones is `length - (index of first 1)`. Finding the first 1 is a lower bound.
**Approach 1 - Brute force:** Count ones in every row. Time O(R * C).
**Approach 2 - Optimal:** Lower bound of 1 in each row. Time O(R log C). (A staircase walk from the top-right gives O(R + C).)

```js
function rowWithMax1s(matrix) {
  let best = -1, bestCount = 0;
  for (let r = 0; r < matrix.length; r++) {
    const row = matrix[r];
    const ones = row.length - lowerBound(row, 1); // rows are sorted 0..0 1..1
    if (ones > bestCount) { bestCount = ones; best = r; }
  }
  return best;
}
```

**Complexity:** Time O(R log C), Space O(1).
**Watch out:** Update only on a strictly greater count, so ties keep the smaller row index.

### 23. Find in Mountain Array — Hard
**Companies:** Google, Amazon, Meta
**Problem:** A mountain array rises strictly then falls strictly. You can only call `get(i)` and `length()`, and `get` is limited to about 100 calls. Return the smallest index with the target, or -1. Example: `[1,2,3,4,5,3,1]`, target 3 -> 2.
**Intuition:** Three small binary searches. First find the peak. Then search the rising side (ascending). Then, only if needed, the falling side (descending). Search the rising side first, because it holds the smaller index.
**Approach 1 - Brute force:** Scan with `get`. Uses O(n) calls, which breaks the call limit.
**Approach 2 - Optimal:** Peak search (problem 11), then two classic searches, one reversed.

```js
function findInMountainArray(target, m) {
  const n = m.length();
  let lo = 0, hi = n - 1;
  while (lo < hi) { // peak
    const mid = lo + ((hi - lo) >> 1);
    if (m.get(mid) < m.get(mid + 1)) lo = mid + 1;
    else hi = mid;
  }
  const peak = lo;
  lo = 0; hi = peak;
  while (lo <= hi) { // ascending side
    const mid = lo + ((hi - lo) >> 1), v = m.get(mid);
    if (v === target) return mid;
    if (v < target) lo = mid + 1;
    else hi = mid - 1;
  }
  lo = peak + 1; hi = n - 1;
  while (lo <= hi) { // descending side
    const mid = lo + ((hi - lo) >> 1), v = m.get(mid);
    if (v === target) return mid;
    if (v > target) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;
}
```

**Complexity:** Time O(log n), Space O(1). About `3 * log n` calls to `get`.
**Watch out:** On the falling side, the comparison flips: if `v > target`, go right. Also the peak belongs to the ascending search.

## Common Mistakes
- **"`mid = (lo + hi) / 2` is fine in JavaScript."** It gives a fraction. Use `lo + ((hi - lo) >> 1)` or `Math.floor`. In other languages the plain sum can also overflow.
- **"I can write `lo = mid`."** In a `lo <= hi` loop that can loop forever. Always move past `mid` unless you are using the half-open "first true" form (`hi = mid`) or "last true" with ceil.
- **"Floor `mid` works for max-min problems."** For "last true" with `lo = mid`, a floor mid gets stuck when `hi = lo + 1`. Round up.
- **"Binary search needs a sorted array."** It needs a monotonic yes/no check. Peak search and answer-space search have no sorted input.
- **"The answer range is 0 to n."** Pick bounds from the problem. Koko starts at 1, capacity starts at `max(weights)`, and lower bound can return `n`.
- **"Rotated search with duplicates is still O(log n)."** Worst case is O(n), because equal ends hide the sorted half.
- **"My check function can be rough."** It must be exactly monotonic. If a bigger value can fail while a smaller one passes, binary search is the wrong tool.
- **Forgetting to test sizes 0, 1, 2.** Most bugs hide there.

## Interview Tips
- **Say the invariant out loud:** "the answer is always inside `[lo, hi]`" (or "first true is in `[lo, hi]`"). It proves correctness and calms you down.
- **State the monotonic question first:** "if speed 5 works, then speed 6 works", then write the check.
- **Choose and name your template:** closed `[lo, hi]` for exact search, half open `[lo, hi)` for first true.
- **Edge cases to mention:** empty array, one item, target smaller than all or larger than all, all items equal, duplicates, answer at the very end.
- **Follow-ups you will hear:** "What if there are duplicates?" (lower/upper bound, rotated II becomes O(n) worst case.) "What if the array is huge or unknown size?" (grow `hi` by doubling until `nums[hi] >= target`, then search.) "Can you do it iteratively?" (yes, always prefer iterative, O(1) space.) "Why O(log n)?" (the range halves each step.)
- **Tie to other patterns:** the greedy check inside answer-space problems is often a two-pointer or counting loop, see [Greedy](../../02-intermediate/04-greedy/README.md). Sorting first (aggressive cows) is covered in [Sorting](../02-sorting/README.md).
- **Floating point answers:** loop a fixed number of times (e.g. 100) instead of `while (lo < hi)`.

## Practice List
**Easy**
- Guess Number Higher or Lower
- Valid Perfect Square
- Arranging Coins
- First Bad Version
- Peak Index in a Mountain Array
- Count Negative Numbers in a Sorted Matrix
- Special Array With X Elements Greater Than or Equal X

**Medium**
- Find First and Last Position of Element in Sorted Array
- Time Based Key-Value Store
- Find K Closest Elements
- Maximum Value at a Given Index in a Bounded Array
- Minimum Time to Complete Trips
- Magnetic Force Between Two Balls
- Find the Duplicate Number (binary search on value)
- Capacity To Ship Packages Within D Days
- Search in a Sorted Array of Unknown Size
- Kth Smallest Element in a Sorted Matrix
- H-Index II
- Maximum Running Time of N Computers

**Hard**
- Median of Two Sorted Arrays
- Split Array Largest Sum
- Find in Mountain Array
- Kth Smallest Number in Multiplication Table
- Minimize Max Distance to Gas Station
- Find K-th Smallest Pair Distance
- Koko Eating Bananas variants with constraints (Minimized Maximum of Products Distributed to Any Store)

## Key Takeaways
- **Binary search needs a monotonic yes/no question**, not necessarily a sorted array.
- **Lower bound (first `>= x`) and upper bound (first `> x`)** solve first, last, count, insert position, floor and ceil.
- **Pick one template and stick to it**: closed `[lo, hi]` with `mid +- 1`, or half open `[lo, hi)` with `hi = mid`.
- **For "maximize the minimum", round mid up** and use `lo = mid`, or the loop never ends.
- **Binary search on the answer** = choose the range, write a greedy `ok(x)`, find the first true. Time is O(n log range).
- **Rotated arrays: one half is always sorted.** Duplicates push the worst case to O(n).
- **Test sizes 0, 1 and 2**, and say your invariant out loud.

---
[← Previous: 07 - Prefix Sum](../07-prefix-sum/README.md) | [Next: 09 - Linked List →](../09-linked-list/README.md) | [Back to Core Linear Index](../README.md)
