# Divide and Conquer

> **Estimated Time:** 5 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - Recursion](../01-recursion/README.md), [02 - Sorting](../../01-core-linear/02-sorting/README.md), [08 - Binary Search](../../01-core-linear/08-binary-search/README.md)

---

## What You'll Learn

- You can explain the three steps (divide, conquer, combine) and write the recurrence for any D&C solution.
- You can use the Master Theorem idea to read off the running time of recurrences like `T(n) = 2T(n/2) + O(n)`.
- You can turn merge sort into a counting tool (inversions, reverse pairs, smaller numbers after self).
- You can solve "split in the middle" problems: max subarray, skyline, balanced BST, majority element.
- You can explain quickselect, fast power, closest pair and Karatsuba at interview level.

## The Big Idea

Imagine you must sort a pile of 1000 exam papers by roll number. Doing it alone is slow. So you split the pile in two, give each half to a friend, and they do the same until each pile has one paper. Then everyone merges their sorted piles back. Nobody did anything hard: splitting is easy, merging two sorted piles is easy, and the work adds up to a fast sort.

Divide and conquer (D&C) has three steps:

1. **Divide:** cut the problem into smaller copies of the same problem (usually halves).
2. **Conquer:** solve each small copy by recursion. A tiny copy (size 0 or 1) is solved directly. This is the base case.
3. **Combine:** build the answer for the big problem from the answers of the small ones.

All the cleverness lives in the **combine** step. Merge sort has a cheap divide and a smart combine. Quick sort has a smart divide (partition) and a free combine.

Why interviewers love it: it tests recursion, it tests whether you can compute complexity from a recurrence, and many "hard" problems (count inversions, skyline) are just merge sort with one extra line. If you can see the pattern, hard problems become routine.

D&C vs. dynamic programming: in D&C the subproblems usually do **not** overlap (the left half and the right half are different). In DP they overlap and we cache them. (A few problems below, like "Different Ways to Add Parentheses", sit in between and benefit from a cache.)

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "count pairs `i < j` with some relation between `a[i]` and `a[j]`" | merge sort, count while merging |
| "for each element, count smaller/larger elements to its right" | merge sort on indices |
| "maximum subarray / best segment, answer may cross the middle" | left answer, right answer, crossing answer |
| "compute `x^n`, `n` is huge" | halve the exponent |
| "k-th largest/smallest, no need to fully sort" | quickselect (average O(n)) |
| "merge k sorted things" | pair them up, merge in rounds (a tournament) |
| "all ways to parenthesize / split an expression" | try every split point, combine left and right results |
| "build a balanced tree from a sorted array" | middle element is the root |
| "combine silhouettes / intervals from the left and right half" | merge two sorted event lists |
| "input is sorted and you need a median or k-th across two arrays" | halve the search space (see binary search chapter) |

## Core Concepts / Template

### The template

```js
function solve(problem) {
  if (isTiny(problem)) return directAnswer(problem); // base case
  const [left, right] = divide(problem);             // usually split at the middle
  const a = solve(left);                             // conquer
  const b = solve(right);
  return combine(a, b);                              // the interesting part
}
```

For arrays, a handy shape is `rec(lo, hi)` on index ranges, so you never copy the array. Use `mid = (lo + hi) >> 1` (safe for array sizes in JS).

### Recurrences and the Master Theorem (the practical version)

A D&C algorithm that splits into `a` subproblems of size `n/b` and spends `f(n)` on divide + combine has this running time:

```
T(n) = a * T(n/b) + f(n)
```

Compare the work at the root (`f(n)`) with the work at the leaves (about `n^(log_b a)`):

| Case | Which part wins | Result | Example |
|---|---|---|---|
| Leaves win | `f(n)` is smaller than `n^(log_b a)` | `O(n^(log_b a))` | `T(n) = 2T(n/2) + O(1)` gives O(n); Karatsuba `3T(n/2) + O(n)` gives `O(n^1.585)` |
| Equal | same size | `O(f(n) log n)` | merge sort `2T(n/2) + O(n)` gives `O(n log n)` |
| Root wins | `f(n)` is larger | `O(f(n))` | quickselect average `T(n) = T(n/2) + O(n)` gives O(n) |

Quick table you should remember:

| Recurrence | Solution | Where |
|---|---|---|
| `T(n) = T(n/2) + O(1)` | O(log n) | binary search, fast power |
| `T(n) = T(n/2) + O(n)` | O(n) | quickselect (average) |
| `T(n) = 2T(n/2) + O(1)` | O(n) | tree recursion on a balanced tree, majority element would be this if combine were O(1) |
| `T(n) = 2T(n/2) + O(n)` | O(n log n) | merge sort, count inversions, max subarray, skyline |
| `T(n) = 2T(n/2) + O(n log n)` | O(n log^2 n) | merge step that also sorts again (a common slip) |
| `T(n) = T(n-1) + O(n)` | O(n^2) | quick sort with the worst pivot every time |

A sanity check without the theorem: draw the recursion tree. Merge sort has `log n` levels and each level touches all `n` elements, so `n log n`.

### Recap: merge sort

Divide at the middle, sort both halves, merge two sorted lists with two pointers. Always O(n log n), needs O(n) extra space, and is stable. Its merge step is the place where we will "hook in" counting logic later.

```js
function mergeSort(a) {
  if (a.length <= 1) return a;
  const mid = a.length >> 1;
  const l = mergeSort(a.slice(0, mid)), r = mergeSort(a.slice(mid));
  const out = [];
  let i = 0, j = 0;
  while (i < l.length && j < r.length) out.push(l[i] <= r[j] ? l[i++] : r[j++]); // <= keeps it stable
  while (i < l.length) out.push(l[i++]);
  while (j < r.length) out.push(r[j++]);
  return out;
}
```

### Recap: quick sort and quickselect

Quick sort: pick a pivot, **partition** so smaller items go left and bigger go right, then recurse on both sides. The divide is the hard part and the combine is empty (the array is already in place). Average O(n log n), worst O(n^2) when pivots are bad (for example always the smallest). A random pivot makes the worst case very unlikely.

Quickselect: after partitioning, the pivot sits at its final sorted position `p`. If `p` is the index we want, done. Otherwise we only recurse into **one** side. That is why the average cost is `n + n/2 + n/4 + ... = O(n)`. Problem 5 has the full code.

### Closest pair of points (conceptual)

Given `n` points in the plane, find the two closest ones. Brute force checks all pairs: O(n^2). The D&C idea:

1. Sort points by x. Split with a vertical line into a left half and a right half.
2. Solve both halves. Let `d` be the smaller of the two best distances.
3. **Combine:** the best pair might cross the line. Only points within distance `d` of the line matter (a vertical strip). Sort the strip by y. For each point you only need to compare with the next few (at most 7) points in y order, because a `d x 2d` box can hold only a handful of points that are pairwise at least `d` apart.

That gives `T(n) = 2T(n/2) + O(n)` (or O(n log n) with a naive y-sort per level), so O(n log n) overall. You will rarely code this in an interview. You should be able to say "split by x, solve both, check the strip of width 2d, only a constant number of neighbours per point."

### Karatsuba multiplication (conceptual)

Multiplying two `n`-digit numbers the school way is O(n^2). Split each number into a high and low half: `x = a*10^m + b`, `y = c*10^m + d`. Then:

```
x*y = ac * 10^(2m) + (ad + bc) * 10^m + bd
```

That is 4 smaller multiplications, so `T(n) = 4T(n/2) + O(n) = O(n^2)`. No gain. Karatsuba's trick: get `ad + bc` from one multiplication:

```
ad + bc = (a + b)(c + d) - ac - bd
```

Now only 3 multiplications (`ac`, `bd`, `(a+b)(c+d)`), so `T(n) = 3T(n/2) + O(n) = O(n^(log2 3)) ~ O(n^1.585)`.

Tiny example, `12 * 34`: `a=1, b=2, c=3, d=4`. `ac=3`, `bd=8`, `(a+b)(c+d)=3*7=21`, so `ad+bc = 21-3-8 = 10`. Result: `3*100 + 10*10 + 8 = 408`. Correct.

Lesson: D&C can be made faster by reducing the number of recursive calls, not only the combine cost. Strassen's matrix multiplication is the same idea (7 products instead of 8).

### Trace: merge sort on `[5, 2, 4, 1]`

```
            [5 2 4 1]
           /         \
       [5 2]         [4 1]        divide
       /   \         /   \
     [5]   [2]     [4]   [1]      base cases
       \   /         \   /
       [2 5]         [1 4]        combine (merge)
           \         /
           [1 2 4 5]
```

## Problems (easy -> hard)

### 1. Majority Element (D&C version) — Easy
**Companies:** Amazon, Google, Microsoft, Apple
**Problem:** Find the element that appears more than `n/2` times (it always exists). Example: `[2,2,1,1,1,2,2]` -> `2`.
**Intuition:** If `x` is the majority of the whole array, then `x` must be the majority of at least one half. (If it were not a majority in either half, it would have at most `n/2` total.) So find each half's majority and decide between the two candidates.
**Approach 1 - Brute force:** count every element with a loop or a hash map. Time O(n) with a map (O(n^2) with nested loops), Space O(n) for the map.
**Approach 2 - Better:** Boyer-Moore voting (keep a candidate and a counter): Time O(n), Space O(1). Best in practice, but it is not D&C.
**Approach 3 - Optimal (for this chapter, D&C):** recurse on halves. If both halves agree, return that. Otherwise count each candidate in the whole range and return the one with the higher count.

Trace on `[2,2,1,1,1,2,2]` (lo=0, hi=6, mid=3): left `[2,2,1,1]` has candidates 2 and 1 (tie: count both, say 2 wins or 1 wins; either is a valid candidate) and right `[1,2,2]` gives 2. Candidates 1 and 2 are counted over the whole range: 1 appears 3 times, 2 appears 4 times. Answer 2.

```js
function majorityElement(nums) {
  function rec(lo, hi) {
    if (lo === hi) return nums[lo];
    const mid = (lo + hi) >> 1;
    const l = rec(lo, mid), r = rec(mid + 1, hi);
    if (l === r) return l;
    const count = (v) => { let c = 0; for (let i = lo; i <= hi; i++) if (nums[i] === v) c++; return c; };
    return count(l) > count(r) ? l : r;
  }
  return rec(0, nums.length - 1);
}
```

**Complexity:** Time O(n log n) (`T(n) = 2T(n/2) + O(n)`), Space O(log n) recursion stack.
**Watch out:** the combine step counts inside `[lo, hi]` only, not the whole array. A tie between candidates cannot happen when a true majority exists in this range, but the sub-ranges do not always have a majority; their returned value is only a candidate. That is fine because the final answer is guaranteed to exist.

### 2. Convert Sorted Array to Binary Search Tree — Easy
**Companies:** Amazon, Facebook, Microsoft
**Problem:** Build a height-balanced BST from a sorted array. Example: `[-10,-3,0,5,9]` -> a tree with root `0` (other valid answers exist).
**Intuition:** In a BST the root splits smaller values from larger ones. In a sorted array the middle element does exactly that. Choose the middle as root, then build the left subtree from the left part and the right subtree from the right part. Halving each time keeps the tree balanced.
**Approach 1 - Brute force:** insert the numbers one by one into a plain BST. A sorted input makes a chain (height n). Time O(n^2), Space O(n).
**Approach 2 - Better:** none worth it.
**Approach 3 - Optimal:** pick the middle, recurse on `[lo, mid-1]` and `[mid+1, hi]`.

Trace on `[1,2,3,4,5]`: mid=3 is root. Left range `[1,2]`: mid=1 is root, its right child is 2. Right range `[4,5]`: mid=4 is root, its right child is 5.

```js
function TreeNode(val, left = null, right = null) { this.val = val; this.left = left; this.right = right; }

function sortedArrayToBST(nums) {
  const build = (lo, hi) => {
    if (lo > hi) return null;
    const mid = (lo + hi) >> 1;
    return new TreeNode(nums[mid], build(lo, mid - 1), build(mid + 1, hi));
  };
  return build(0, nums.length - 1);
}
```

**Complexity:** Time O(n), Space O(log n) for the stack (plus O(n) for the tree itself).
**Watch out:** for an even count there are two middles. Either choice is accepted, but be consistent. Do not use `nums.slice` in each call: it adds O(n log n) copying.

### 3. Pow(x, n) — Medium
**Companies:** Facebook, Google, LinkedIn, Microsoft
**Problem:** Compute `x` raised to the integer power `n` (negative `n` allowed). Example: `2, 10` -> `1024`; `2, -2` -> `0.25`.
**Intuition:** `x^10 = (x^5)^2` and `x^5 = (x^2)^2 * x`. Each step halves the exponent, so we need only about `log n` multiplications. Compute `x^(n/2)` once and reuse it (do not call twice).
**Approach 1 - Brute force:** multiply `x` by itself `n` times. Time O(n), Space O(1).
**Approach 2 - Better:** none.
**Approach 3 - Optimal:** `half = pow(x, floor(n/2))`. If `n` is even the answer is `half*half`, else `half*half*x`. For negative `n`, compute for `-n` and invert.

Trace `pow(2, 10)`: pow(2,5) -> pow(2,2) -> pow(2,1) -> pow(2,0)=1. Back up: pow(2,1)=1*1*2=2, pow(2,2)=2*2=4, pow(2,5)=4*4*2=32, pow(2,10)=32*32=1024.

```js
function myPow(x, n) {
  if (n < 0) return 1 / myPow(x, -n); // JS numbers are doubles, so -n is safe even for -2^31
  if (n === 0) return 1;
  const half = myPow(x, Math.floor(n / 2));
  return n % 2 === 0 ? half * half : half * half * x;
}
```

**Complexity:** Time O(log n), Space O(log n) stack (O(1) if written as a loop on the bits of `n`).
**Watch out:** in languages with 32-bit ints, `-n` overflows for `n = -2^31`; in JS it does not, but say it out loud. Never write `pow(x, n/2) * pow(x, n/2)`: that makes two calls and silently turns O(log n) into O(n). In JS use `Math.floor(n / 2)` since `/` is not integer division. Also `x = 0` with negative `n` gives `Infinity`; mention it.

### 4. Maximum Subarray (D&C version) — Medium
**Companies:** Amazon, Microsoft, Apple, LinkedIn, Google
**Problem:** Find the contiguous subarray with the largest sum. Example: `[-2,1,-3,4,-1,2,1,-5,4]` -> `6` (subarray `[4,-1,2,1]`).
**Intuition:** Split at the middle. The best subarray is fully in the left half, fully in the right half, or **crosses the middle**. The first two are recursive. For the crossing one: the best sum ending at `mid` going left, plus the best sum starting at `mid+1` going right.
**Approach 1 - Brute force:** try all `(i, j)` pairs with a running sum. Time O(n^2), Space O(1).
**Approach 2 - Better:** Kadane's algorithm: `cur = max(x, cur + x)`. Time O(n), Space O(1). This is what you would write in a real interview.
**Approach 3 - Optimal (D&C, O(n log n)):** the three-way max described above. Interviewers sometimes ask this as the "follow-up" to Kadane's.

Trace on `[-2, 1, -3, 4]` (mid index 1): left `[-2,1]` best is 1. Right `[-3,4]` best is 4. Crossing: going left from index 1: `1`, then `1-2=-1`, best 1. Going right from index 2: `-3`, then `-3+4=1`, best 1. Cross = 2. Answer max(1, 4, 2) = 4.

```js
function maxSubArray(nums) {
  function rec(lo, hi) {
    if (lo === hi) return nums[lo];
    const mid = (lo + hi) >> 1;
    let s = 0, bestLeft = -Infinity;
    for (let i = mid; i >= lo; i--) { s += nums[i]; bestLeft = Math.max(bestLeft, s); }
    s = 0;
    let bestRight = -Infinity;
    for (let i = mid + 1; i <= hi; i++) { s += nums[i]; bestRight = Math.max(bestRight, s); }
    return Math.max(rec(lo, mid), rec(mid + 1, hi), bestLeft + bestRight);
  }
  return rec(0, nums.length - 1);
}
```

**Complexity:** Time O(n log n), Space O(log n).
**Watch out:** start the best values at `-Infinity`, not 0, or an all-negative array returns a wrong answer (the subarray must be non-empty). The crossing part must include at least one element from each side.

### 5. Kth Largest Element in an Array (Quickselect) — Medium
**Companies:** Amazon, Facebook, Google, Microsoft, LinkedIn
**Problem:** Return the k-th largest value (not the k-th distinct). Example: `[3,2,1,5,6,4], k=2` -> `5`.
**Intuition:** Sorting works but does too much. We only need one element in its right place. Partition around a pivot: the pivot lands at its final sorted index. If that is the index we want, stop. If not, we know which side holds our answer and drop the other side.
**Approach 1 - Brute force:** sort and pick `n-k`. Time O(n log n), Space O(1) to O(n).
**Approach 2 - Better:** a min-heap of size `k`. Time O(n log k), Space O(k). Safe and predictable (see the heaps chapter).
**Approach 3 - Optimal:** quickselect with a random pivot. Average O(n), worst O(n^2) (very unlikely with random pivots).

The k-th largest is at sorted index `n - k`. Trace on `[3,2,1,5,6,4]`, k=2, target index 4. Pick pivot 4: after partition `[3,2,1,4,6,5]`, pivot at index 3. 3 < 4, so go right: range `[6,5]`, pivot 5 -> `[5,6]`, pivot at index 4 = target. Answer 5.

```js
function findKthLargest(nums, k) {
  const a = nums.slice(); // do not mutate the caller's array
  const target = a.length - k;
  let lo = 0, hi = a.length - 1;
  while (lo < hi) {
    const p = lo + Math.floor(Math.random() * (hi - lo + 1)); // random pivot
    [a[p], a[hi]] = [a[hi], a[p]];                              // park pivot at the end
    let s = lo;                                                 // s = next slot for a "smaller" item
    for (let i = lo; i < hi; i++) if (a[i] < a[hi]) { [a[i], a[s]] = [a[s], a[i]]; s++; }
    [a[s], a[hi]] = [a[hi], a[s]];                              // pivot to its final place
    if (s === target) return a[s];
    if (s < target) lo = s + 1; else hi = s - 1;
  }
  return a[lo];
}
```

**Complexity:** Time O(n) average, O(n^2) worst. Space O(1) beyond the copy.
**Watch out:** with many equal values and the strict `<` partition above, runs of duplicates still make progress because the pivot itself is fixed in place each round. Without a random pivot, a sorted input is the classic worst case. Quickselect does not give a stable worst-case bound; if asked for one, mention "median of medians" (O(n) worst, rarely coded) or the heap.

### 6. Convert Sorted List to Binary Search Tree — Medium
**Companies:** Amazon, Facebook, Microsoft
**Problem:** Same as problem 2, but the input is a sorted singly linked list. Example: `-10 -> -3 -> 0 -> 5 -> 9` -> a balanced BST.
**Intuition:** A linked list has no O(1) middle access. Finding the middle each time (slow/fast pointers) works but costs O(n) per level. The trick: build the tree **in-order**. An in-order walk of a BST visits values in sorted order, which is the order of the list. So recurse left first, then take the next list node as the root, then recurse right. The list pointer moves forward as we build.
**Approach 1 - Brute force:** copy the list into an array, then reuse problem 2. Time O(n), Space O(n) extra.
**Approach 2 - Better:** find the middle with slow/fast pointers, cut the list, recurse. Time O(n log n), Space O(log n).
**Approach 3 - Optimal:** count `n`, then build with an index range and a shared pointer `cur`. Time O(n).

Trace on `1 -> 2 -> 3` (n=3, range 0..2, mid=1): build(0,0) takes node 1 (cur moves to 2). Then root takes node 2 (cur moves to 3). Then build(2,2) takes node 3. Result: 2 with children 1 and 3.

```js
function ListNode(val, next = null) { this.val = val; this.next = next; }
function TreeNode(val, left = null, right = null) { this.val = val; this.left = left; this.right = right; }

function sortedListToBST(head) {
  let n = 0;
  for (let p = head; p; p = p.next) n++;
  let cur = head; // advances in in-order, so no random access is needed
  const build = (lo, hi) => {
    if (lo > hi) return null;
    const mid = (lo + hi) >> 1;
    const left = build(lo, mid - 1);       // must run first: consumes the smaller values
    const node = new TreeNode(cur.val);
    cur = cur.next;
    node.left = left;
    node.right = build(mid + 1, hi);
    return node;
  };
  return build(0, n - 1);
}
```

**Complexity:** Time O(n), Space O(log n) stack.
**Watch out:** the order of operations is the whole trick. If you create the node before building the left subtree, the values land in the wrong places. Handle the empty list (`n = 0` returns `null`).

### 7. Different Ways to Add Parentheses — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** Given an expression of digits and `+ - *`, return the results of all possible ways to group it with parentheses. Example: `"2-1-1"` -> `[0, 2]` (`(2-1)-1 = 0` and `2-(1-1) = 2`).
**Intuition:** Each way of grouping decides which operator is evaluated **last**. Pick every operator as the "last" one. The text on its left and right are smaller expressions: solve them recursively, then combine every left result with every right result using that operator.
**Approach 1 - Brute force:** the recursion itself, with no cache. Time grows like the Catalan numbers (exponential).
**Approach 2 - Better:** the same recursion plus a memo keyed by the substring, because the same sub-expression appears many times.
**Approach 3 - Optimal:** there is no polynomial answer, since the output itself can be exponentially large (Catalan). Memoized D&C is the standard solution.

Trace `"2*3-4"`: split at `*`: left `[2]`, right `"3-4" = [-1]` -> `2 * -1 = -2`. Split at `-`: left `"2*3" = [6]`, right `[4]` -> `6 - 4 = 2`. Result `[-2, 2]`.

```js
function diffWaysToCompute(expr) {
  const memo = new Map();
  const go = (s) => {
    if (memo.has(s)) return memo.get(s);
    const res = [];
    for (let i = 0; i < s.length; i++) {
      const c = s[i];
      if (c === '+' || c === '-' || c === '*') {
        for (const a of go(s.slice(0, i)))
          for (const b of go(s.slice(i + 1)))
            res.push(c === '+' ? a + b : c === '-' ? a - b : a * b);
      }
    }
    if (res.length === 0) res.push(Number(s)); // no operator found: it is just a number
    memo.set(s, res);
    return res;
  };
  return go(expr);
}
```

**Complexity:** Time and Space roughly O(Catalan(m)) where `m` is the number of operators (output size dominates).
**Watch out:** numbers can have several digits (like `"15"`), so test "no operator" and parse the whole substring, not one character. The order of results is not fixed; sort before comparing in tests. Never mutate a memoized array.

### 8. Count Inversions — Medium
**Companies:** Amazon, Microsoft, Flipkart, Adobe
**Problem:** Count pairs `(i, j)` with `i < j` and `a[i] > a[j]`. It measures how unsorted an array is. Example: `[2,4,1,3,5]` -> `3` (pairs `(2,1)`, `(4,1)`, `(4,3)`).
**Intuition:** Run merge sort. When merging two sorted halves `L` and `R`, if we pick an element from `R` while `L` still has `m` elements left, then that `R` element is smaller than **all** `m` remaining elements of `L`. That is `m` inversions counted in one step, without checking each one.
**Approach 1 - Brute force:** two nested loops. Time O(n^2), Space O(1).
**Approach 2 - Better:** a Fenwick tree over compressed values (see the segment tree and Fenwick chapter). Time O(n log n).
**Approach 3 - Optimal (in spirit):** merge sort with counting. Time O(n log n), Space O(n).

Trace the merge of `L=[2,4]` and `R=[1,3]`:

| Compare | Pick | Reason | Inversions added |
|---|---|---|---|
| 2 vs 1 | 1 (from R) | R smaller, L has 2 left | +2 |
| 2 vs 3 | 2 (from L) | - | 0 |
| 4 vs 3 | 3 (from R) | R smaller, L has 1 left | +1 |
| rest | 4, then 5 | - | 0 |

Total inside this merge: 3. The other levels add 0.

```js
function countInversions(arr) {
  const a = arr.slice(), tmp = new Array(a.length);
  function sort(lo, hi) { // sorts a[lo..hi) and returns inversions inside it
    if (hi - lo < 2) return 0;
    const mid = (lo + hi) >> 1;
    let cnt = sort(lo, mid) + sort(mid, hi);
    let i = lo, j = mid, k = lo;
    while (i < mid && j < hi) {
      if (a[i] <= a[j]) tmp[k++] = a[i++];          // <= : equal values are NOT an inversion
      else { cnt += mid - i; tmp[k++] = a[j++]; }   // a[i..mid) are all greater than a[j]
    }
    while (i < mid) tmp[k++] = a[i++];
    while (j < hi) tmp[k++] = a[j++];
    for (let x = lo; x < hi; x++) a[x] = tmp[x];
    return cnt;
  }
  return sort(0, a.length);
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** use `<=` when comparing so equal elements stay on the left and are not counted. The answer can reach about `n^2/2`: fine for JS doubles up to roughly 9e15 (so n up to about 1e8), but say "use a 64-bit integer" in languages with 32-bit ints.

### 9. Merge k Sorted Lists — Hard
**Companies:** Amazon, Google, Facebook, Microsoft, Uber
**Problem:** Merge `k` sorted linked lists into one sorted list. Example: `[[1,4,5],[1,3,4],[2,6]]` -> `[1,1,2,3,4,4,5,6]`.
**Intuition:** Merging two lists is easy. To merge `k`, do it like a knockout tournament: pair lists up, merge each pair, then pair the results, until one list remains. Each element takes part in only `log k` merges.
**Approach 1 - Brute force:** merge lists one after another into an accumulator. The accumulator grows, so total work is O(N*k) where `N` is the total number of nodes.
**Approach 2 - Better:** a min-heap holding the head of each list. Pop the smallest, push its next. Time O(N log k), Space O(k). (Heaps chapter.)
**Approach 3 - Optimal (D&C):** recursive pairing. Time O(N log k), Space O(log k) for the stack, and no heap needed.

Trace with lists `A, B, C, D`: round 1 merges `(A,B)` and `(C,D)`. Round 2 merges the two results. That is 2 rounds = `log2(4)`, and each round touches all nodes once.

```js
function ListNode(val, next = null) { this.val = val; this.next = next; }

function mergeKLists(lists) {
  const merge2 = (a, b) => {
    const dummy = new ListNode(0);
    let t = dummy;
    while (a && b) {
      if (a.val <= b.val) { t.next = a; a = a.next; } else { t.next = b; b = b.next; }
      t = t.next;
    }
    t.next = a || b; // attach whatever is left
    return dummy.next;
  };
  const rec = (lo, hi) => { // inclusive range of list indices
    if (lo > hi) return null;
    if (lo === hi) return lists[lo];
    const mid = (lo + hi) >> 1;
    return merge2(rec(lo, mid), rec(mid + 1, hi));
  };
  return rec(0, lists.length - 1);
}
```

**Complexity:** Time O(N log k), Space O(log k).
**Watch out:** `lists` can be empty or contain `null` entries. The merge relinks existing nodes (it destroys the input lists); say so, or copy nodes if inputs must stay intact. Do not merge into one growing accumulator: that is the O(N*k) trap.

### 10. Count of Smaller Numbers After Self — Hard
**Companies:** Google, Amazon, Microsoft, Bloomberg
**Problem:** For each `i`, count how many `j > i` have `nums[j] < nums[i]`. Example: `[5,2,6,1]` -> `[2,1,1,0]`.
**Intuition:** This is inversion counting, but we need the count **per element**, not a total. Merge sort moves values around, so we sort **indices** and keep `nums` untouched. While merging, when we place a left-half index, the number of right-half items already placed is exactly how many smaller values (from the right side) it has. Add that to its answer.
**Approach 1 - Brute force:** for every `i`, scan to the right. Time O(n^2), Space O(1).
**Approach 2 - Better:** Fenwick tree over compressed values, scanning from the right. Time O(n log n), Space O(n).
**Approach 3 - Optimal (here):** merge sort on indices, Time O(n log n), Space O(n).

Trace on `[5,2,6,1]`: the left half `[5,2]` becomes sorted indices `[1(2), 0(5)]` and the right half `[6,1]` becomes `[3(1), 2(6)]`. Merge:

| Next left | Next right | Action | Right placed so far |
|---|---|---|---|
| 2 (idx1) | 1 (idx3) | 1 < 2, place right | 1 |
| 2 (idx1) | 6 (idx2) | place left idx1: `res[1] += 1` | 1 |
| 5 (idx0) | 6 (idx2) | place left idx0: `res[0] += 1` | 1 |
| - | 6 (idx2) | place right | 2 |

Earlier the inner merges added `res[0] += 1` (5 > 2) and `res[2] += 1` (6 > 1). Final: `[2, 1, 1, 0]`.

```js
function countSmaller(nums) {
  const n = nums.length, res = new Array(n).fill(0);
  const idx = Array.from({ length: n }, (_, i) => i); // we sort indices, not values
  function sort(lo, hi) { // [lo, hi)
    if (hi - lo < 2) return;
    const mid = (lo + hi) >> 1;
    sort(lo, mid); sort(mid, hi);
    const tmp = [];
    let i = lo, j = mid;
    while (i < mid || j < hi) {
      // take from the right only if it is STRICTLY smaller (equal values are not "smaller")
      if (j < hi && (i >= mid || nums[idx[j]] < nums[idx[i]])) tmp.push(idx[j++]);
      else { res[idx[i]] += j - mid; tmp.push(idx[i++]); } // j - mid right items are smaller than it
    }
    for (let x = 0; x < tmp.length; x++) idx[lo + x] = tmp[x];
  }
  sort(0, n);
  return res;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** if you sort values instead of indices you lose track of who is who. On ties, place the **left** item first so equal values on the right are not counted as smaller. An element only counts right-half items, which is correct because the left half already handled its own side in deeper levels.

### 11. Reverse Pairs — Hard
**Companies:** Google, Amazon, Microsoft, Uber
**Problem:** Count pairs `i < j` with `nums[i] > 2 * nums[j]`. Example: `[1,3,2,3,1]` -> `2`.
**Intuition:** It is the inversion count with a "stronger" condition. The catch: the condition is not the same as the sort order, so we cannot count during the merge itself. Do it in **two passes** inside each level: first count using two pointers on the two sorted halves, then do an ordinary merge.
**Approach 1 - Brute force:** all pairs. Time O(n^2), Space O(1).
**Approach 2 - Better:** Fenwick tree with coordinate compression of `nums` and `2*nums`. Time O(n log n).
**Approach 3 - Optimal (here):** merge sort with a separate counting sweep. Time O(n log n), Space O(n).

Why the pointer only moves forward: the left half is sorted ascending, so as `a[i]` grows, the set of `j` with `a[i] > 2*a[j]` only grows (the right half is sorted too). Trace for left `[1,3]`, right `[1,2]`: i=1: `1 > 2*1`? no, `j` stays at mid, add 0. i=3: `3 > 2` yes, j moves 1 step; `3 > 4`? no. add 1. Total 1.

```js
function reversePairs(nums) {
  const a = nums.slice(), tmp = new Array(a.length);
  function sort(lo, hi) { // sorts a[lo..hi), returns the pair count
    if (hi - lo < 2) return 0;
    const mid = (lo + hi) >> 1;
    let cnt = sort(lo, mid) + sort(mid, hi);
    let j = mid; // j only moves forward because the left half is sorted
    for (let i = lo; i < mid; i++) {
      while (j < hi && a[i] > 2 * a[j]) j++;
      cnt += j - mid;
    }
    let i = lo, k = lo;
    j = mid;
    while (i < mid && j < hi) tmp[k++] = a[i] <= a[j] ? a[i++] : a[j++];
    while (i < mid) tmp[k++] = a[i++];
    while (j < hi) tmp[k++] = a[j++];
    for (let x = lo; x < hi; x++) a[x] = tmp[x];
    return cnt;
  }
  return sort(0, a.length);
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** do the counting pass **before** merging; after merging the halves are mixed. Write `a[i] > 2 * a[j]`, not `a[i]/2 > a[j]` (integer-division bugs in other languages). In languages with 32-bit ints, `2 * a[j]` overflows: use a 64-bit type. Not an issue in JS doubles.

### 12. The Skyline Problem — Hard
**Companies:** Google, Amazon, Microsoft, Uber, Facebook
**Problem:** Given buildings `[left, right, height]`, return the "key points" (x, height) of the outline when viewed from far away. Example: `[[2,9,10],[3,7,15],[5,12,12],[15,20,10],[19,24,8]]` -> `[[2,10],[3,15],[7,12],[12,0],[15,10],[20,8],[24,0]]`.
**Intuition:** One building has a trivial skyline: `[[left, h], [right, 0]]`. Merge two skylines like merging two sorted lists by x. While scanning, remember the current height of each skyline. The merged height at any x is the **max** of the two. Emit a point only when that max changes.
**Approach 1 - Brute force:** collect all x coordinates and, for each, scan all buildings for the tallest one covering it. Time O(n^2), Space O(n).
**Approach 2 - Better:** sweep line with a max-heap of active building heights. Time O(n log n), Space O(n). (See the intervals and sweep line chapter.)
**Approach 3 - Optimal (here, D&C):** split the building list in halves, solve each, merge skylines. `T(n) = 2T(n/2) + O(n)`, so O(n log n).

Trace merging `A = [[1,3],[4,0]]` and `B = [[2,5],[3,0]]`:

| Step | x taken | A height | B height | max | Output |
|---|---|---|---|---|---|
| 1 | 1 (A) | 3 | 0 | 3 | [1,3] |
| 2 | 2 (B) | 3 | 5 | 5 | [2,5] |
| 3 | 3 (B) | 3 | 0 | 3 | [3,3] |
| 4 | 4 (A) | 0 | 0 | 0 | [4,0] |

```js
function getSkyline(buildings) {
  function rec(lo, hi) { // inclusive
    if (lo === hi) { const [l, r, h] = buildings[lo]; return [[l, h], [r, 0]]; }
    const mid = (lo + hi) >> 1;
    return mergeSky(rec(lo, mid), rec(mid + 1, hi));
  }
  function mergeSky(A, B) {
    const out = [];
    let i = 0, j = 0, ha = 0, hb = 0;
    while (i < A.length || j < B.length) {
      let x;
      if (j >= B.length || (i < A.length && A[i][0] <= B[j][0])) {
        x = A[i][0]; ha = A[i][1]; i++;
        if (j < B.length && B[j][0] === x) { hb = B[j][1]; j++; } // same x in both: consume both
      } else {
        x = B[j][0]; hb = B[j][1]; j++;
      }
      const h = Math.max(ha, hb);
      if (out.length === 0 || out[out.length - 1][1] !== h) {
        if (out.length && out[out.length - 1][0] === x) out[out.length - 1][1] = h; // same x: overwrite
        else out.push([x, h]);
      }
    }
    return out;
  }
  return buildings.length ? rec(0, buildings.length - 1) : [];
}
```

**Complexity:** Time O(n log n), Space O(n) (output plus O(log n) stack).
**Watch out:** only add a point when the height **changes** (no duplicate heights in a row). When both skylines have a point at the same x, consume both before taking the max. If the output already has a point at this x, overwrite its height instead of pushing a second point. The buildings arrive sorted by `left`; the D&C merge does not rely on that, but the original problem guarantees it.

### 13. Median of Two Sorted Arrays — Hard
**Companies:** Google, Amazon, Microsoft, Apple, Facebook
**Problem:** Two sorted arrays of sizes `m` and `n`. Return the median of the combined data in `O(log(m+n))` time. Example: `[1,2]` and `[3,4]` -> `2.5`.
**Intuition:** The median splits all numbers into a left half and a right half of equal size. If we choose how many items `i` come from `A`'s left side, then `j = half - i` come from `B`'s left side. The split is correct when `A[i-1] <= B[j]` and `B[j-1] <= A[i]`. Binary search on `i` in the **smaller** array. This is "halve the search space", the same D&C spirit as binary search, covered fully in [08 - Binary Search](../../01-core-linear/08-binary-search/README.md).
**Approach 1 - Brute force:** merge both arrays and pick the middle. Time O(m+n), Space O(m+n).
**Approach 2 - Better:** two pointers walking to the middle without building the merged array. Time O(m+n), Space O(1).
**Approach 3 - Optimal:** binary search on the partition of the smaller array.

Trace `A=[1,2]`, `B=[3,4]` (`half = 2`): try `i=1, j=1`: `A[0]=1 <= B[1]=4` ok, but `B[0]=3 > A[1]=2` fails, so move `i` right. Try `i=2, j=0`: `A[1]=2 <= B[0]=3` ok, and `bL = -Infinity <= aR = Infinity` ok. Total is even: `(max(2, -Inf) + min(Inf, 3)) / 2 = 2.5`.

```js
function findMedianSortedArrays(A, B) {
  if (A.length > B.length) [A, B] = [B, A]; // search in the smaller array
  const m = A.length, n = B.length, half = (m + n + 1) >> 1;
  let lo = 0, hi = m;
  while (lo <= hi) {
    const i = (lo + hi) >> 1, j = half - i;
    const aL = i > 0 ? A[i - 1] : -Infinity, aR = i < m ? A[i] : Infinity;
    const bL = j > 0 ? B[j - 1] : -Infinity, bR = j < n ? B[j] : Infinity;
    if (aL > bR) hi = i - 1;        // took too many from A
    else if (bL > aR) lo = i + 1;   // took too few from A
    else return (m + n) % 2 ? Math.max(aL, bL) : (Math.max(aL, bL) + Math.min(aR, bR)) / 2;
  }
}
```

**Complexity:** Time O(log(min(m, n))), Space O(1).
**Watch out:** use `+/-Infinity` sentinels for empty sides instead of special cases. Always search the smaller array, otherwise `j` can go negative. For an odd total, the median is the max of the left sides because `half` uses `(m+n+1)>>1`.

## Common Mistakes

- **"D&C always means O(n log n)."** No. Binary search is O(log n), quickselect is O(n) on average, Karatsuba is `O(n^1.585)`. Write the recurrence and solve it.
- **Calling the recursive function twice for the same value** (like `pow(x, n/2) * pow(x, n/2)`). Store the result once. Otherwise you lose the whole benefit.
- **Slicing arrays in every call.** `slice` costs O(n), which can silently add a log factor or more. Pass index ranges `(lo, hi)` instead.
- **Forgetting the base case or using `(lo + hi) / 2` without flooring.** In JS `/` gives a fraction. Use `>> 1` or `Math.floor`.
- **Counting in the wrong moment while merging.** For inversions, count when taking from the right half. For reverse pairs, count in a separate pass **before** the merge. Mixing these up gives wrong totals.
- **Wrong tie rules.** Strict versus non-strict comparison decides whether equal elements count. Test with duplicates like `[2,2,2]`.
- **Believing quick sort is always fast.** A fixed pivot on sorted input gives O(n^2) and deep recursion. Use a random pivot.
- **Using D&C where there is heavy overlap.** If the same subproblem appears again and again (Fibonacci style), D&C is exponential; add a cache (DP).

## Interview Tips

- **Say the three steps out loud:** "I divide at the middle, solve both halves, and combine like this." Then state the recurrence and its solution before coding.
- **State the base case first.** Usually `lo === hi` or `lo > hi`.
- **Offer the simple answer first, then the D&C one.** For max subarray, say "Kadane is O(n); the D&C version is O(n log n) but generalises to range queries" (that is the idea behind segment trees).
- **Common follow-ups:**
  - Can you do it iteratively (bottom-up merge sort)? Yes: merge runs of size 1, 2, 4, ... which avoids recursion.
  - Space? Merge sort needs O(n) for the temp array; quick sort is in-place with O(log n) average stack.
  - Is it stable? Merge sort yes (with `<=`); quick sort no.
  - Why is quickselect O(n) on average? `n + n/2 + n/4 + ...` is about `2n`.
  - Can the count be done another way? Fenwick tree or segment tree after coordinate compression.
- **Edge cases to mention:** empty input, single element, all equal values, all negative numbers, already sorted/reverse sorted input, very large totals (overflow in other languages), and `null` or empty lists for list problems.
- **When you see "count pairs" on an array**, say "merge sort counting" right away. It is the trigger phrase interviewers hope to hear.

## Practice List

**Easy**
- Sort an Array (merge sort / quick sort)
- Maximum Depth of Binary Tree (D&C on a tree)
- Merge Two Sorted Lists
- Sqrt(x) (binary search, halving)
- Search in a Rotated Sorted Array (see binary search chapter)

**Medium**
- Sort List (merge sort on a linked list)
- Sort Colors (partition idea)
- Kth Smallest Element in a Sorted Matrix
- Top K Frequent Elements (quickselect variant)
- Unique Binary Search Trees II (all trees, D&C plus recursion)
- Construct Binary Tree from Preorder and Inorder Traversal
- Search a 2D Matrix II
- Super Pow
- Beautiful Array
- Count Inversions (GFG practice problem)

**Hard**
- Count of Range Sum
- Maximum Sum Circular Subarray (variants of max subarray)
- Merge k Sorted Lists (revisit with a heap)
- Closest Pair of Points (SPOJ / GFG classic)
- Number of Longest Increasing Subsequence (D&C/segment tree idea)
- The Skyline Problem (sweep-line + heap version)
- Median of Two Sorted Arrays (revisit the partition proof)
- Burst Balloons (interval D&C with a cache; see DP)

## Key Takeaways

- **Divide, conquer, combine:** the combine step is where the real thinking happens.
- **Write the recurrence** `T(n) = a*T(n/b) + f(n)`; `2T(n/2) + O(n)` means O(n log n), `T(n/2) + O(1)` means O(log n), `T(n/2) + O(n)` means O(n).
- **"Count pairs" on an array usually means merge sort with counting**; count before or during the merge depending on whether the pair condition matches the sort order.
- **Pass index ranges, not slices**, to keep each level at O(n) work.
- **Compute a repeated subresult once** (fast power) and reuse it; reducing the number of recursive calls is how Karatsuba beats O(n^2).
- **Quickselect gives O(n) average** by recursing into one side only; use a random pivot and mention the heap as the safe alternative.
- **D&C subproblems do not overlap; DP subproblems do.** If you see repeats, add a memo.

---

[← Previous: Backtracking](../02-backtracking/README.md) | [Next: Greedy →](../04-greedy/README.md) | [Back to Intermediate Index](../README.md)
