# Monotonic Stack and Monotonic Queue

> **Estimated Time:** 4 hr | **Difficulty:** Intermediate
> **Prerequisites:** [10 - Stack and Queue](../10-stack-and-queue/README.md), [01 - Arrays](../01-arrays/README.md), [06 - Sliding Window](../06-sliding-window/README.md)

---

## What You'll Learn
- You can explain what a monotonic stack is and why each element is pushed and popped only once.
- You can find the next/previous greater/smaller element for every index in O(n).
- You can use the "contribution" idea to sum something over all subarrays without looping over them.
- You can use a monotonic deque to get a sliding window maximum in O(n).
- You can pick the right loop shape: pop on `<`, pop on `<=`, scan left to right, or scan right to left.

## The Big Idea

Imagine you stand in a queue for a concert and you want to know "who is the first person in front of me who is taller than me?" Short people hidden behind a tall person can never be the answer for anyone further back. So once a tall person arrives, you can forget every shorter person before them. You only keep a list of people who still might be someone's answer.

That is a monotonic stack. It is a normal stack, but you keep its values in one order (always decreasing, or always increasing). Before you push a new value, you pop everything that breaks the order. The popped items have just found their answer: the new value.

Why it is fast: every element is pushed once and popped at most once. So the inner `while` loop looks scary but the total work is O(n), not O(n^2).

Why interviewers love it: the brute force is an obvious double loop. The stack version needs a real insight, and the same trick solves a big family of problems (histograms, rain water, subarray sums, greedy digit removal). If you know it well, many "hard" problems become template work.

The **monotonic queue (deque)** is the same idea for a moving window. You also remove from the front when an element leaves the window.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "next greater / smaller element" | monotonic stack, scan left to right |
| "how many days until a warmer day" | next greater, store the distance |
| "span", "how many previous elements are <= me" | previous greater, stack with counts |
| "largest rectangle", "widest range where I am the minimum" | previous smaller + next smaller |
| "sum over all subarrays of min or max" | contribution technique (count subarrays where each item is the min) |
| "remove k digits / letters to get the smallest result" | greedy monotonic stack |
| "collisions", "cars catching up" | stack where the top interacts with the new item |
| "max of every window of size k" | monotonic deque (decreasing) |
| "shortest subarray with sum at least K" and negatives exist | prefix sums + monotonic deque |
| "circular array" | loop `2n` times, use `i % n` |

## Core Concepts / Template

**Rule of thumb**

- To find the **next greater**, keep the stack **decreasing** (top is the smallest). A bigger new value pops the smaller ones, and it is their answer.
- To find the **next smaller**, keep the stack **increasing**.
- For **previous** greater/smaller: after you pop, whatever is left on top is the answer for the current element (before you push it).
- Store **indices** on the stack when you need distances or widths. Store **values** when you only need the value.

**Template: next greater element to the right**

```js
function nextGreater(a) {
  const ans = new Array(a.length).fill(-1);
  const st = []; // indices, values a[st[0]] >= a[st[1]] >= ... (decreasing)
  for (let i = 0; i < a.length; i++) {
    while (st.length && a[st[st.length - 1]] < a[i]) {
      ans[st.pop()] = a[i]; // a[i] is the answer for the popped index
    }
    st.push(i);
  }
  return ans; // indices left on the stack have no greater element: stay -1
}
```

Trace on `[2, 1, 5, 3, 6]`:

| i | a[i] | stack (values) before | popped | answers set | stack after |
|---|---|---|---|---|---|
| 0 | 2 | [] | - | - | [2] |
| 1 | 1 | [2] | - | - | [2,1] |
| 2 | 5 | [2,1] | 1, 2 | ans of 1 and 2 = 5 | [5] |
| 3 | 3 | [5] | - | - | [5,3] |
| 4 | 6 | [5,3] | 3, 5 | ans of 3 and 5 = 6 | [6] |

Result: `[5, 5, 6, 6, -1]`.

**The four variants (same skeleton)**

| Want | Stack order | Pop while | Answer is |
|---|---|---|---|
| next greater | decreasing | `top < cur` | `cur`, given to the popped item |
| next smaller | increasing | `top > cur` | `cur`, given to the popped item |
| previous greater | decreasing | `top <= cur` | the top left after popping |
| previous smaller | increasing | `top >= cur` | the top left after popping |

Whether you pop on `<` or `<=` decides how **ties** are handled. This matters a lot in the contribution problems (see problem 13).

**Template: monotonic deque for a window**

```js
// window max: keep indices whose values are decreasing
// 1) pop from the back while the back is not bigger than the new value
// 2) drop the front if it left the window
// 3) the front is the max
```

The deque is the "stack that also lets the oldest item leave". In JavaScript, `Array.shift()` is O(n) in the worst case, so use a `head` pointer instead.

## Problems (easy -> hard)

### 1. Next Greater Element I — Easy
**Companies:** Amazon, Google, Bloomberg
**Problem:** `nums1` is a subset of `nums2`. For each value of `nums1`, find the first bigger value to its right in `nums2`, or -1. Example: `nums1=[4,1,2]`, `nums2=[1,3,4,2]` -> `[-1,3,-1]`.
**Intuition:** Compute "next greater" for every value of `nums2` once with a stack, save it in a map, then look up each query.
**Approach 1 - Brute force:** for each query find it in `nums2` and scan right. Time O(n*m), Space O(1).
**Approach 2 - Optimal:** decreasing stack over `nums2`. When a bigger `x` arrives, every smaller value on the stack gets `x` as its answer. Dry run on `[1,3,4,2]`: push 1; 3 pops 1 (1 -> 3), push 3; 4 pops 3 (3 -> 4), push 4; push 2. Leftover 4 and 2 -> -1. Queries 4, 1, 2 give -1, 3, -1.

```js
function nextGreaterElementI(nums1, nums2) {
  const next = new Map(), st = [];
  for (const x of nums2) {
    while (st.length && st[st.length - 1] < x) next.set(st.pop(), x);
    st.push(x);
  }
  return nums1.map(x => next.has(x) ? next.get(x) : -1);
}
```

**Complexity:** Time O(n + m), Space O(n).
**Watch out:** the values are unique in this problem, so a value-keyed map is safe. With duplicates, key by index instead.

### 2. Previous Smaller Element — Easy
**Companies:** Amazon, Flipkart, Microsoft
**Problem:** For each element, find the nearest element to its left that is strictly smaller, or -1 (classic GFG problem). Example: `[4,5,2,10,8]` -> `[-1,4,-1,2,2]`.
**Intuition:** Keep an increasing stack. Anything on the stack that is `>=` the new value can never be the "previous smaller" for a later element (the new value is closer and not bigger). So pop them. What is left on top is the answer.
**Approach 1 - Brute force:** scan left for each index. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** increasing stack, answer = top after popping. Dry run: 4 -> empty, -1, push; 5 -> top 4, push; 2 -> pops 5 and 4, empty, -1, push; 10 -> top 2; 8 -> pops 10, top 2.

```js
function previousSmaller(a) {
  const ans = new Array(a.length).fill(-1), st = [];
  for (let i = 0; i < a.length; i++) {
    while (st.length && st[st.length - 1] >= a[i]) st.pop();
    if (st.length) ans[i] = st[st.length - 1];
    st.push(a[i]);
  }
  return ans;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** this version returns values. If you need distances (for widths), store indices instead.

### 3. Daily Temperatures — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** For each day, how many days until a warmer one? 0 if none. Example: `[73,74,75,71,69,72,76,73]` -> `[1,1,4,2,1,1,0,0]`.
**Intuition:** It is "next greater", but you store the distance `i - j` instead of the value. So the stack must hold indices.
**Approach 1 - Brute force:** for each day, scan right until warmer. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** decreasing stack of indices. Dry run on `[73,74,75,71]`: i=0 push; i=1 (74) pops 0, ans[0]=1; i=2 (75) pops 1, ans[1]=1; i=3 (71) pushes. Stack `[2,3]` stay 0 until a warmer day appears.

```js
function dailyTemperatures(t) {
  const ans = new Array(t.length).fill(0), st = []; // indices, temps decreasing
  for (let i = 0; i < t.length; i++) {
    while (st.length && t[st[st.length - 1]] < t[i]) {
      const j = st.pop();
      ans[j] = i - j;
    }
    st.push(i);
  }
  return ans;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** pop on strictly `<`. An equal temperature is not "warmer".

### 4. Next Greater Element II (Circular) — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Same as problem 1 but the array is circular, so after the last element you wrap to the first. Example: `[1,2,1]` -> `[2,-1,2]`.
**Intuition:** A circle is the array written twice. Walk indices `0..2n-1`, read `nums[i % n]`.
**Approach 1 - Brute force:** for each index scan up to n-1 steps forward with wrap. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** same stack, two passes. Only push in the first pass (so each index is on the stack once). In the second pass we only resolve the leftovers. Dry run `[1,2,1]`: pass 1 pushes idx0; 2 pops idx0 (ans 2), push idx1; push idx2. Pass 2: value 1 pops nothing; value 2 pops idx2 (ans 2); idx1 (value 2) has no bigger value anywhere, stays -1.

```js
function nextGreaterElementsCircular(nums) {
  const n = nums.length, ans = new Array(n).fill(-1), st = [];
  for (let i = 0; i < 2 * n; i++) {
    const x = nums[i % n];
    while (st.length && nums[st[st.length - 1]] < x) ans[st.pop()] = x;
    if (i < n) st.push(i);
  }
  return ans;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** do not push in the second pass, or you waste work and may double count. The max element correctly stays -1.

### 5. Online Stock Span — Medium
**Companies:** Amazon, Google, Microsoft, Bloomberg
**Problem:** Design `next(price)`. It returns the span: how many consecutive days (including today, going back) the price was `<=` today's price. Example prices `100,80,60,70,60,75,85` -> `1,1,1,2,1,4,6`.
**Intuition:** It is "previous greater" in an online setting. Instead of indices, store `[price, span]`. When a bigger price arrives, it swallows the spans of the smaller ones it pops.
**Approach 1 - Brute force:** each call walks back through the history. Time O(n) per call, Space O(n).
**Approach 2 - Optimal:** decreasing stack of `[price, span]`. Dry run: 70 comes after 100,80,60. It pops 60 (span 1), so span = 1+1 = 2. Then 75 pops 60 (span 1) and 70 (span 2): span = 1+1+2 = 4. Stack is now `[100,1],[80,1],[75,4]`. Then 85 pops 75 and 80 but stops at 100: span = 1+4+1 = 6.

```js
class StockSpanner {
  constructor() { this.st = []; } // [price, span]
  next(price) {
    let span = 1;
    while (this.st.length && this.st[this.st.length - 1][0] <= price) span += this.st.pop()[1];
    this.st.push([price, span]);
    return span;
  }
}
```

**Complexity:** Time amortized O(1) per call, Space O(n).
**Watch out:** pop on `<=` (equal prices count in the span). Store the span, not just the price; this is what keeps the work amortized O(1).

### 6. Remove K Digits — Medium
**Companies:** Amazon, Google, Microsoft, Facebook
**Problem:** Given a number as a string, remove exactly `k` digits to make the smallest possible number. Example: `"1432219", k=3` -> `"1219"`.
**Intuition:** Compare digits from the left. If a digit is bigger than the one after it, removing it makes the number smaller right away (a smaller digit moves into that high place). So keep an increasing stack and pop when the new digit is smaller.
**Approach 1 - Brute force:** try all ways to choose which `k` digits to remove. Exponential.
**Approach 2 - Optimal:** increasing stack, pop while `k > 0` and `top > digit`. Dry run on `1432219`, k=3: push 1; push 4; 3 pops 4 (k=2), push 3; 2 pops 3 (k=1), push 2; next 2 pushes (stack `1,2,2`); 1 pops one 2 (k=0), push 1 (stack `1,2,1`); 9 pushes. Answer `1219`.

```js
function removeKdigits(num, k) {
  const st = [];
  for (const c of num) {
    while (k > 0 && st.length && st[st.length - 1] > c) { st.pop(); k--; }
    st.push(c);
  }
  while (k-- > 0) st.pop();
  let i = 0;
  while (i < st.length - 1 && st[i] === '0') i++;
  return st.slice(i).join('') || '0';
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** (a) if `k` is left over (digits were already increasing, like `"123"`), pop from the end. (b) Strip leading zeros but keep at least one digit. (c) An empty result is `"0"`.

### 7. Remove Duplicate Letters — Medium
**Companies:** Amazon, Google, Microsoft, Facebook
**Problem:** Remove duplicate letters so every letter appears once, and the result is the smallest in alphabetical order among all choices. The same problem is "Smallest Subsequence of Distinct Characters". Example: `"cbacdcbc"` -> `"acdb"`.
**Intuition:** Same as remove-k-digits, with a rule: you may pop a letter only if it appears again later (so you can still place it). Also skip a letter that is already in the stack.
**Approach 1 - Brute force:** generate all subsequences with each letter once and take the smallest. Exponential.
**Approach 2 - Optimal:** precompute the last index of each letter. Increasing stack + `seen` set. For letter `c` not yet seen, pop the top while `top > c` and `top` appears later. Dry run `bcabc`: b push; c push; a: pops c (c appears later), pops b (appears later), push a; b push; c push. Result `abc`.

```js
function removeDuplicateLetters(s) {
  const last = {}, seen = new Set(), st = [];
  for (let i = 0; i < s.length; i++) last[s[i]] = i;
  for (let i = 0; i < s.length; i++) {
    const c = s[i];
    if (seen.has(c)) continue;
    while (st.length && st[st.length - 1] > c && last[st[st.length - 1]] > i) seen.delete(st.pop());
    st.push(c); seen.add(c);
  }
  return st.join('');
}
```

**Complexity:** Time O(n), Space O(1) for a fixed alphabet (O(n) in general).
**Watch out:** if the letter is already in the stack, skip it. Do not pop it from the middle. Remember to remove popped letters from `seen`.

### 8. Asteroid Collision — Medium
**Companies:** Amazon, Google, Meta, Uber
**Problem:** Each number is an asteroid: size = absolute value, positive moves right, negative moves left, same speed. When two meet, the smaller explodes (both if equal). Return what is left. Example: `[10,2,-5]` -> `[10]`.
**Intuition:** A collision only happens when the stack top moves right (positive) and the new one moves left (negative). Every other pair never meets.
**Approach 1 - Brute force:** repeat scanning for adjacent colliding pairs until none remain. Time O(n^2).
**Approach 2 - Optimal:** stack. For a new negative asteroid, while the top is positive: compare. If the top is smaller, pop it and continue. If equal, pop and the new one dies too. If the top is bigger, the new one dies. Dry run `[10,2,-5]`: push 10, push 2; -5 meets 2 (2 < 5, pop), meets 10 (10 > 5, -5 dies). Result `[10]`.

```js
function asteroidCollision(a) {
  const st = [];
  for (const x of a) {
    let alive = true;
    while (alive && x < 0 && st.length && st[st.length - 1] > 0) {
      const top = st[st.length - 1];
      if (top < -x) st.pop();
      else { if (top === -x) st.pop(); alive = false; }
    }
    if (alive) st.push(x);
  }
  return st;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** negative asteroids at the bottom of the stack survive (nothing to hit). The equal-size case destroys both. Use a flag for "new asteroid alive".

### 9. Car Fleet — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** Cars drive to `target` on a one-lane road and cannot pass; a faster car behind slows to the speed of the car ahead and they form a fleet. Count fleets that arrive. Example: `target=12, position=[10,8,0,5,3], speed=[2,4,1,1,3]` -> `3`.
**Intuition:** Compute each car's arrival time if it were alone. Go from the car closest to the target backwards. If a car behind needs more time than the one ahead, it never catches up and starts a new fleet. If it needs less or equal time, it joins the fleet ahead.
**Approach 1 - Brute force:** simulate time steps. Hard to bound and slow.
**Approach 2 - Optimal:** sort by position descending, keep the slowest time so far (this is the stack top in a stack version). Dry run: times by position 10,8,5,3,0 are 1, 1, 7, 3, 12. Walk: 1 (new fleet, max=1), 1 (joins), 7 (new, max=7), 3 (joins), 12 (new). Total 3.

```js
function carFleet(target, position, speed) {
  const cars = position.map((p, i) => [p, (target - p) / speed[i]]).sort((a, b) => b[0] - a[0]);
  let fleets = 0, slowest = 0;
  for (const [, t] of cars) if (t > slowest) { fleets++; slowest = t; }
  return fleets;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** compare times with `>` (strict). Equal times arrive together, so they are one fleet. Floating division is fine here because equal ratios divide the same way for these inputs, but mention it if asked.

### 10. Sliding Window Maximum — Hard
**Companies:** Amazon, Google, Meta, Microsoft, Uber
**Problem:** For every window of size `k`, output the maximum. Example: `[1,3,-1,-3,5,3,6,7], k=3` -> `[3,3,5,5,6,7]`.
**Intuition:** Keep a deque of indices with decreasing values. Smaller old values behind a new bigger value will never be a maximum again (the bigger one lives longer), so remove them. The front is always the maximum. Remove the front when it slides out.
**Approach 1 - Brute force:** scan each window. Time O(n*k), Space O(1).
**Approach 2 - Better:** a max heap with lazy deletion. Time O(n log n).
**Approach 3 - Optimal:** monotonic deque. Dry run on `[1,3,-1,-3,5]`, k=3: i=0: deque [0]. i=1 (3): pops 0, deque [1]. i=2 (-1): deque [1,2], output 3. i=3 (-3): deque [1,2,3], output 3. i=4 (5): front idx 1 is out of window (1 <= 4-3), drop; pops 3, 2; deque [4], output 5.

```js
function maxSlidingWindow(nums, k) {
  const dq = [], out = []; // indices; values decreasing
  let head = 0; // avoid O(n) shift
  for (let i = 0; i < nums.length; i++) {
    while (dq.length > head && nums[dq[dq.length - 1]] <= nums[i]) dq.pop();
    dq.push(i);
    if (dq[head] <= i - k) head++;
    if (i >= k - 1) out.push(nums[dq[head]]);
  }
  return out;
}
```

**Complexity:** Time O(n), Space O(k).
**Watch out:** the deque stores indices (so you can check expiry). Pop the back on `<=` to keep it small. Using `Array.shift()` can be slow, so the code moves a `head` pointer.

### 11. Largest Rectangle in Histogram — Hard
**Companies:** Amazon, Google, Microsoft, Meta, Bloomberg
**Problem:** Bars of width 1 with given heights. Find the area of the biggest rectangle inside. Example: `[2,1,5,6,2,3]` -> `10` (bars 5 and 6, height 5, width 2).
**Intuition:** The best rectangle has some bar as its shortest bar. For bar `i` with height `h`, the rectangle stretches left and right until it meets a shorter bar. So you need the **previous smaller** and **next smaller** for each bar. Width = `next - prev - 1`.
**Approach 1 - Brute force:** for each pair `(i, j)` track the minimum. Time O(n^2), Space O(1).
**Approach 2 - Better:** compute previous-smaller and next-smaller arrays in two passes, then area per bar. Time O(n), Space O(n).
**Approach 3 - Optimal (one pass):** increasing stack of indices. When a shorter bar `i` arrives, it is the "next smaller" for the popped bar. The bar just below on the stack is its "previous smaller". Compute the area right then. A sentinel height 0 at the end flushes the stack. Dry run `[2,1,5,6,2,3]`: push idx 0; i=1 (h=1) pops idx 0: height 2, left=-1, width 1, area 2; push 1; push 2; push 3; i=4 (2) pops 3 (h6, left=2, width 4-2-1=1, area 6), pops 2 (h5, left=1, width 2, area 10), push 4; i=5 push 5; i=6 (0) pops 5 (h3, left=4, width 1, area 3), pops 4 (h2, left=1, width 4, area 8), pops 1 (h1, left=-1, width 6, area 6). Max 10.

```js
function largestRectangleArea(h) {
  const st = []; let best = 0;
  for (let i = 0; i <= h.length; i++) {
    const cur = i === h.length ? 0 : h[i]; // sentinel flushes the stack
    while (st.length && h[st[st.length - 1]] >= cur) {
      const height = h[st.pop()];
      const left = st.length ? st[st.length - 1] : -1;
      best = Math.max(best, height * (i - left - 1));
    }
    st.push(i);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** pop on `>=` is fine here (an equal bar will compute the full width when it pops later). Do not forget the final flush, or the bars still on the stack are lost.

### 12. Maximal Rectangle — Hard
**Companies:** Google, Amazon, Microsoft, Meta
**Problem:** A binary matrix of `"0"` and `"1"` characters. Find the area of the largest rectangle that has only 1s. Example: the 4x5 grid in the code test below -> `6`.
**Intuition:** Treat each row as the floor of a histogram. For each column, the bar height is the number of consecutive 1s going up from this row. Then it is problem 11, once per row.
**Approach 1 - Brute force:** try every rectangle and check. Time O((rows*cols)^2) or worse, Space O(1).
**Approach 2 - Optimal:** update heights row by row (`h[j] = h[j] + 1` if cell is 1, else 0), call the histogram function. Dry run on the first two rows `10100 / 10111`: heights after row 1 are `[1,0,1,0,0]`, best 1; after row 2 they are `[2,0,2,1,1]`, best 3 (columns 2 to 4 at height 1). Later rows raise it to 6.

```js
function maximalRectangle(matrix) {
  if (!matrix.length) return 0;
  const h = new Array(matrix[0].length).fill(0); let best = 0;
  for (const row of matrix) {
    for (let j = 0; j < row.length; j++) h[j] = row[j] === '1' ? h[j] + 1 : 0;
    best = Math.max(best, largestRectangleArea(h));
  }
  return best;
}
```

**Complexity:** Time O(rows*cols), Space O(cols).
**Watch out:** the input holds strings `"1"`/`"0"`, not numbers. Reset the height to 0 on a 0 cell. Empty matrix returns 0.

### 13. Sum of Subarray Minimums — Medium
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** Sum the minimum of every contiguous subarray, modulo 1e9+7. Example: `[3,1,2,4]` -> `17`.
**Intuition (contribution technique):** Do not list subarrays. Ask: "for each element, in how many subarrays is it the minimum?" If `arr[i]` is the min of subarrays that start within `L` choices on the left and end within `R` choices on the right, it is the min of `L * R` subarrays, so it adds `arr[i] * L * R`. `L = i - prevSmaller`, `R = nextSmaller - i`.
**Approach 1 - Brute force:** all subarrays with a running min. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** one monotonic stack pass. When index `mid` is popped by index `i`, `i` is its next smaller and the new top is its previous smaller. Dry run `[3,1,2,4]`: 1 pops 3 (L=1,R=1, adds 3). End flush: pop 4 (L=1,R=1: 4), pop 2 (L=1,R=2: 4), pop 1 (L=2,R=3: 6). Total 3+4+4+6 = 17.

```js
function sumSubarrayMins(arr) {
  const MOD = 1_000_000_007n, n = arr.length, st = [];
  let total = 0n;
  for (let i = 0; i <= n; i++) {
    const cur = i === n ? -Infinity : arr[i];
    // pop on >=: left side strict-less, right side less-or-equal => ties counted once
    while (st.length && arr[st[st.length - 1]] >= cur) {
      const mid = st.pop();
      const left = st.length ? st[st.length - 1] : -1;
      total += BigInt(arr[mid]) * BigInt(mid - left) * BigInt(i - mid);
    }
    st.push(i);
  }
  return Number(total % MOD);
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** **ties.** With equal values, a subarray has more than one min element, and you must credit it to exactly one of them. Here one side is strict and the other is non-strict: popping on `>=` means the previous side is "strictly smaller" and the next side is "smaller or equal". The sum can exceed 2^53, so the code uses `BigInt` and takes the modulo at the end. (In other languages, take the modulo as you go.)

### 14. Sum of Subarray Ranges — Medium
**Companies:** Amazon, Google, Meta
**Problem:** The range of a subarray is `max - min`. Return the sum of ranges of all subarrays. This covers "sum of subarray maximums" too, since the answer is (sum of maximums) - (sum of minimums). Example: `[1,2,3]` -> `4`.
**Intuition:** Run the contribution technique twice: once for maximums, once for minimums. Subtract.
**Approach 1 - Brute force:** all subarrays, track min and max. Time O(n^2), Space O(1) (fine for n up to 1000, which is this problem's limit).
**Approach 2 - Optimal:** two monotonic stack passes. Dry run `[1,2,3]`: sum of maxs = 1+2+3+2+3+3 = 14, sum of mins = 1+2+3+1+2+1 = 10, difference 4.

```js
function subArrayRanges(nums) {
  const n = nums.length;
  const f = (cmp) => { // sum of subarray extremes, cmp(a,b) true if a should be popped by b
    let total = 0; const st = [];
    for (let i = 0; i <= n; i++) {
      while (st.length && (i === n || cmp(nums[st[st.length - 1]], nums[i]))) {
        const mid = st.pop();
        const left = st.length ? st[st.length - 1] : -1;
        total += nums[mid] * (mid - left) * (i - mid);
      }
      st.push(i);
    }
    return total;
  };
  return f((top, cur) => top <= cur) - f((top, cur) => top >= cur);
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** the tie rule again: for the max pass, pop on `<=` (top <= cur); for the min pass, pop on `>=`. Each pass has one strict side and one non-strict side. The same helper handles both by swapping the comparison.

### 15. Trapping Rain Water — Hard
**Companies:** Amazon, Google, Meta, Microsoft, Goldman Sachs
**Problem:** Given bar heights, how much rain water is trapped between them? Example: `[0,1,0,2,1,0,1,3,2,1,2,1]` -> `6`.
**Intuition:** Water sits in a "valley": a bottom bar with a taller bar on both sides. Keep a decreasing stack. When a taller bar arrives, the popped bar is the bottom of a valley, the new stack top is the left wall, and the new bar is the right wall. Fill that layer horizontally.
**Approach 1 - Brute force:** for each bar, find max on the left and max on the right; water = `min(L, R) - h`. Time O(n^2), Space O(1).
**Approach 2 - Better:** precompute prefix max and suffix max arrays. Time O(n), Space O(n). A two-pointer version gets Space O(1) (see [05 - Two Pointers](../05-two-pointers/README.md)).
**Approach 3 - Stack:** water in a layer = `(min(leftWall, rightWall) - bottom) * width`, where width = `i - left - 1`. Dry run `[3,0,2]`: push 0, push 1; i=2 (h=2) pops bottom=1 (h=0), left=0 (h=3), water = (min(3,2) - 0) * (2-0-1) = 2.

```js
function trap(h) {
  const st = []; let water = 0; // indices, heights decreasing
  for (let i = 0; i < h.length; i++) {
    while (st.length && h[st[st.length - 1]] < h[i]) {
      const bottom = st.pop();
      if (!st.length) break;
      const left = st[st.length - 1];
      water += (Math.min(h[left], h[i]) - h[bottom]) * (i - left - 1);
    }
    st.push(i);
  }
  return water;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** if the stack is empty after popping the bottom, there is no left wall: stop (the code uses `break`). The stack method adds water layer by layer (horizontal), while the prefix-max method adds it column by column (vertical). Both give the same total.

### 16. 132 Pattern — Medium
**Companies:** Google, Amazon, Meta
**Problem:** Is there a triple `i < j < k` with `nums[i] < nums[k] < nums[j]`? Example: `[3,1,4,2]` -> `true` (1, 4, 2).
**Intuition:** Scan from the right. Keep a decreasing stack of candidate "3"s. When a bigger number arrives, the popped values are smaller than it; the largest of those popped is the best "2" (call it `third`). If you later see a number smaller than `third`, that is the "1" and you are done.
**Approach 1 - Brute force:** three loops. Time O(n^3). A better brute force fixes the middle and keeps the prefix min: O(n^2).
**Approach 2 - Optimal:** right to left, stack + `third`. Dry run `[3,1,4,2]`: i=3 (2): stack [2]. i=2 (4): pops 2, third=2, stack [4]. i=1 (1): 1 < third (2): return true.

```js
function find132pattern(nums) {
  const st = []; let third = -Infinity; // third = best "2" with a bigger "3" before it
  for (let i = nums.length - 1; i >= 0; i--) {
    if (nums[i] < third) return true;
    while (st.length && st[st.length - 1] < nums[i]) third = st.pop();
    st.push(nums[i]);
  }
  return false;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** check `nums[i] < third` before pushing. `third` always has a bigger value ("3") still on the stack to its left in scan order (to its right in the array), which is why the order is valid.

### 17. Shortest Subarray with Sum at Least K — Hard
**Companies:** Google, Amazon, Meta
**Problem:** Find the length of the shortest non-empty subarray with sum `>= k`. The array may contain negative numbers. Return -1 if none. Example: `[2,-1,2], k=3` -> `3`.
**Intuition:** Normal sliding window fails because of negatives (adding a number can make the sum smaller). Use prefix sums `pre`. You want the closest `i < j` with `pre[j] - pre[i] >= k`. Keep a deque of indices with **increasing** prefix sums: (1) if the front satisfies the sum, record the length and pop it (any later `j` would only give a longer subarray with this `i`); (2) before pushing `j`, pop the back while `pre[back] >= pre[j]` (a smaller, later start is always better).
**Approach 1 - Brute force:** all pairs of prefix sums. Time O(n^2), Space O(n).
**Approach 2 - Optimal:** monotonic deque over prefix sums. Dry run `[2,-1,2], k=3`: pre = `[0,2,1,3]`. j=0: dq [0]. j=1: 2-0=2 < 3; push, dq [0,1]. j=2: pre=1; 1-0 < 3; back pre 2 >= 1 pops 1; dq [0,2]. j=3: pre=3; 3-0=3 >= 3, length 3, pop 0; 3-1=2 < 3. Answer 3.

```js
function shortestSubarray(nums, k) {
  const n = nums.length, pre = new Array(n + 1).fill(0);
  for (let i = 0; i < n; i++) pre[i + 1] = pre[i] + nums[i];
  const dq = []; let head = 0, best = Infinity;
  for (let j = 0; j <= n; j++) {
    while (dq.length > head && pre[j] - pre[dq[head]] >= k) { best = Math.min(best, j - dq[head]); head++; }
    while (dq.length > head && pre[dq[dq.length - 1]] >= pre[j]) dq.pop();
    dq.push(j);
  }
  return best === Infinity ? -1 : best;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** it is a prefix-sum deque, not a plain window. Use a `head` index, not `shift()`. If all numbers are positive, a two-pointer window is enough.

## Common Mistakes
- **"The inner `while` makes it O(n^2)."** No. Each element is pushed once and popped at most once, so the total is O(n).
- **Pushing values when you need distance.** If the answer uses `i - j` or a width, store indices on the stack.
- **Wrong pop condition on ties.** `<` vs `<=` changes the answer in span, histogram and contribution problems. Decide what a tie means before coding.
- **Forgetting leftovers.** Items still on the stack at the end have no next greater/smaller. Set them to -1 / 0, or flush with a sentinel (as in the histogram).
- **Using `shift()` for a deque in JS.** It can be O(n). Use a head pointer or a linked structure.
- **Remove-k-digits: forgetting leading zeros and leftover `k`.** `"10200"`, k=1 gives `"200"`; `"123"`, k=2 needs popping from the end.
- **Circular arrays: pushing in both passes.** Only push in the first pass.

## Interview Tips
- Say the brute force first (scan right for each element, O(n^2)), then say "elements that can never be an answer again can be thrown away, so I will keep a monotonic stack".
- State the invariant out loud: "the stack is decreasing from bottom to top, so the top is the nearest candidate".
- Say why it is O(n): each index is pushed once, popped once.
- Before coding, say whether the stack holds values or indices and why.
- Mention the tie rule and how you chose `<` vs `<=`.
- Follow-ups you may get: do it for a circular array; do it online (stock span); get both previous and next smaller in one pass; solve rain water with two pointers in O(1) space; solve sliding window max with a heap and compare.
- Edge cases to mention: empty input, one element, all equal values, strictly increasing and strictly decreasing arrays, negative numbers, `k >= n`, `k = 0` (remove-k-digits), result with leading zeros.

## Practice List
**Easy**
- Next Greater Element I
- Baseball Game
- Final Prices With a Special Discount in a Shop
- Remove All Adjacent Duplicates In String
- Make The String Great

**Medium**
- Daily Temperatures
- Next Greater Element II
- Online Stock Span
- Remove K Digits
- Remove Duplicate Letters
- Smallest Subsequence of Distinct Characters
- Asteroid Collision
- Car Fleet
- 132 Pattern
- Sum of Subarray Minimums
- Sum of Subarray Ranges
- Online Election
- Next Greater Node In Linked List
- Number of Visible People in a Queue
- Longest Well-Performing Interval
- Maximum Width Ramp
- Minimum Cost Tree From Leaf Values
- Sum of Total Strength of Wizards

**Hard**
- Sliding Window Maximum
- Largest Rectangle in Histogram
- Maximal Rectangle
- Trapping Rain Water
- Shortest Subarray with Sum at Least K
- Constrained Subsequence Sum
- Jump Game VI
- Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit

## Key Takeaways
- A **monotonic stack** keeps candidates in sorted order and throws away the ones that can never be an answer.
- It is **O(n) overall** because each element is pushed once and popped at most once.
- Next greater = decreasing stack; next smaller = increasing stack. For "previous", read the top **after** popping.
- Store **indices** when you need distances or widths; values when you only need the value.
- The **contribution technique** (count subarrays where each element is the min/max) turns O(n^2) subarray sums into one stack pass; be careful with **ties**.
- A **monotonic deque** is the same idea for sliding windows: pop the back to keep order, pop the front when it expires.
- For greedy "smallest result" problems (remove k digits, remove duplicate letters), **pop a bigger earlier item if a smaller one arrives and you can afford to drop it**.

---
[← Previous: Stack and Queue](../10-stack-and-queue/README.md) | [Next: Recursion →](../../02-intermediate/01-recursion/README.md) | [Back to Core Linear Index](../README.md)
