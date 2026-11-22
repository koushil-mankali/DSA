# Randomization and Sampling

> **Estimated Time:** 3 hr | **Difficulty:** Intermediate
> **Prerequisites:** [03 - Hashing](../../01-core-linear/03-hashing/README.md), [07 - Prefix Sum](../../01-core-linear/07-prefix-sum/README.md), [08 - Binary Search](../../01-core-linear/08-binary-search/README.md), [04 - Math Basics](../../00-foundations/04-math-basics/README.md)

---

## What You'll Learn
- You can shuffle an array fairly with Fisher-Yates and explain why the "obvious" shuffle is biased.
- You can pick a random item from a stream of unknown length using reservoir sampling (one item or k items).
- You can pick items by weight using prefix sums and binary search.
- You can build a set with O(1) insert, delete and getRandom (with and without duplicates).
- You can turn a "bad" random source into a good one with rejection sampling (rand7 to rand10, points in a circle).
- You can use random pivots for quickselect to find the kth largest in expected O(n).

## The Big Idea

Think of a lucky draw. You put names in a hat, mix them, and pull one out. A fair draw means every name has the same chance. If the hat is badly mixed, or some names are written on bigger slips of paper, the draw is not fair, even if it "feels" random.

Randomization problems are about **fairness you can prove**. The code is usually short. The hard part is the argument: "each outcome has probability exactly 1/n". Interviewers love these problems because:
- A small bug (`j` in the wrong range) silently gives a biased answer, and the program still "works".
- You must reason with probability, not just with loops.
- They test data-structure thinking (swap with last, hash map plus array, prefix sums).

In JavaScript the only random source is `Math.random()`, which returns a float in `[0, 1)`. To get an integer in `[0, n)` use `Math.floor(Math.random() * n)`. Every solution below uses this helper:

```js
const ri = (lo, hi) => lo + Math.floor(Math.random() * (hi - lo + 1)); // inclusive
```

Three tools cover almost everything in this chapter:
1. **Uniform choice** from a known list: pick a random index.
2. **Remap and shrink**: swap the chosen item to the end and pretend the list is shorter (shuffle, O(1) delete, flip matrix, blacklist).
3. **Rejection**: generate a bigger uniform range, throw away the part you cannot use, try again.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "shuffle", "random permutation", "all permutations equally likely" | Fisher-Yates |
| "random item from a stream / linked list of unknown length", "cannot store everything" | Reservoir sampling |
| "pick index with probability proportional to weight" | Prefix sums + binary search |
| "insert, delete, getRandom in O(1)" | Array + hash map, swap with last |
| "implement randA using randB" | Rejection sampling, or build a bigger uniform range first |
| "random point inside a shape" | Rejection sampling, or a formula (watch the area) |
| "flip a random 0 to 1 without repeats", "random number not in blacklist" | Virtual array with a hash map (lazy Fisher-Yates) |
| "kth largest / smallest, average O(n)" | Quickselect with a random pivot |
| "is this item probably in the set, tiny memory, no deletes" | Bloom filter |

## Core Concepts / Template

### 1. Fisher-Yates shuffle

Walk from the end to the start. At position `i`, pick `j` uniformly from `0..i` (including `i`), and swap `a[i]` with `a[j]`. After that, position `i` is final.

```js
function fisherYates(a) {
  for (let i = a.length - 1; i > 0; i--) {
    const j = ri(0, i);
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
```

Why it is fair: there are `n` choices for the last slot, then `n-1` for the next, and so on. That is `n * (n-1) * ... * 1 = n!` equally likely paths, and each path gives a different permutation. So each permutation has probability `1/n!`.

**Why the naive shuffle is biased.** The tempting version picks `j` from the whole array every time:

```js
function naiveShuffle(a) {
  for (let i = 0; i < a.length; i++) {
    const j = ri(0, a.length - 1);
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
```

This makes `n` choices at each of `n` steps, so there are `n^n` equally likely paths. But there are only `n!` permutations. `n^n` is not a multiple of `n!` for `n >= 3`, so some permutations must get more paths than others.

For `n = 3`: `3^3 = 27` paths, 6 permutations. 27 / 6 = 4.5, so it is impossible to be even. In practice the counts look like this (out of 27 paths):

| Permutation | 012 | 021 | 102 | 120 | 201 | 210 |
|---|---|---|---|---|---|---|
| Naive paths (of 27) | 4 | 5 | 5 | 5 | 4 | 4 |
| Fisher-Yates paths (of 6) | 1 | 1 | 1 | 1 | 1 | 1 |

I checked this with a chi-square test in Node: with 200,000 runs the naive shuffle fails clearly and Fisher-Yates passes. The most common wrong version of Fisher-Yates is `ri(0, i - 1)` (excluding `i`). That is Sattolo's algorithm: it only produces cyclic permutations, so no element can stay in its place.

### 2. Reservoir sampling

**Problem it solves:** pick one item uniformly from a stream whose length you do not know, using O(1) memory.

**Rule (k = 1):** keep the first item. For the `i`-th item (1-indexed), replace the kept item with probability `1/i`.

**Proof it is fair.** Item `i` is chosen at step `i` with probability `1/i`. It then must survive steps `i+1 ... n`. At step `j` it survives with probability `(j-1)/j`. Multiply:

```
1/i * (i/(i+1)) * ((i+1)/(i+2)) * ... * ((n-1)/n) = 1/n
```

Everything cancels, so every item ends with probability `1/n`.

**Rule (k items):** fill the reservoir with the first `k`. For item `i > k`, pick `j` uniformly in `0..i-1`. If `j < k`, replace `reservoir[j]` with this item. Item `i` enters with probability `k/i`. An existing item is knocked out with probability `(k/i) * (1/k) = 1/i`. The same cancellation gives `k/n` for every item.

```js
function reservoirK(iterable, k) {
  const res = [];
  let n = 0;
  for (const x of iterable) {
    n++;
    if (res.length < k) res.push(x);
    else {
      const j = Math.floor(Math.random() * n); // 0..n-1
      if (j < k) res[j] = x;
    }
  }
  return res;
}
```

### 3. Weighted random pick

Make a prefix-sum array. For weights `[1, 3, 6]` the prefix sums are `[1, 4, 10]`. Think of a number line `1..10`: index 0 owns `{1}`, index 1 owns `{2,3,4}`, index 2 owns `{5..10}`. Pick a random number `t` in `1..10`, then binary search for the first prefix sum `>= t`.

### 4. Rejection sampling

If you can produce a uniform value over a big range but you need a smaller range that does not divide it evenly, **throw away** the leftover values and retry. Because you only discard, the values you keep are still uniform. Never use "modulo on everything": that keeps the bias.

### 5. Randomized quickselect

Like quicksort, but you only go into the one side that holds the answer. With a random pivot the expected work is `n + n/2 + n/4 + ... = O(n)`. Worst case is O(n^2) but a random pivot makes that vanishingly unlikely. A deterministic O(n) worst case exists (median of medians) but it is rarely asked in code.

### 6. Bloom filter (intro)

A Bloom filter answers "have I seen this?" with a small bit array and a few hash functions. To add an item, set `k` bits (one per hash). To query, check those `k` bits. If any bit is 0, the item is **definitely not** in the set. If all are 1, it is **probably** in the set (false positives are possible, false negatives are not). You cannot delete from a plain one. It is used as a cheap "no" filter in front of a slow lookup. Randomization shows up because the hashes behave like random choices. The internals and the false-positive formula are in [06 - Data Structure Internals](../../06-advanced/06-data-structure-internals/README.md).

## Problems (easy -> hard)

### 1. Shuffle an Array — Medium
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** Design a class with `shuffle()` (returns a random permutation, all equally likely) and `reset()` (returns the original array). Example: `[1,2,3]` -> `shuffle()` could give `[3,1,2]`; `reset()` gives `[1,2,3]`.
**Intuition:** Keep the original untouched. For each shuffle, copy it and run Fisher-Yates on the copy.
**Approach 1 - Brute force:** Repeatedly pick a random unused element and append it (remove it from a list each time). Fair, but each removal is O(n), so total O(n^2).
**Approach 2 - Better:** Attach a random number to each element and sort. Fair (with ties handled), but O(n log n).
**Approach 3 - Optimal:** Fisher-Yates on a copy. For `[1,2,3]`, `i=2`: pick `j` in 0..2, say 0, swap -> `[3,2,1]`. `i=1`: pick `j` in 0..1, say 1, no change. Done: `[3,2,1]`.

```js
function fisherYates(a) {
  for (let i = a.length - 1; i > 0; i--) {
    const j = ri(0, i);
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

class Shuffle {
  constructor(nums) { this.orig = nums.slice(); }
  reset() { return this.orig.slice(); }
  shuffle() { return fisherYates(this.orig.slice()); }
}
```

**Complexity:** Time O(n) per shuffle, Space O(n) for the copy.
**Watch out:** `j` must include `i` itself (an element may stay). Do not shuffle `this.orig` in place, or `reset()` is lost. Empty and one-element arrays work without special code.

### 2. Random Pick Index — Medium
**Companies:** Amazon, Meta, Google
**Problem:** Given an array with duplicates, `pick(target)` returns a random index `i` where `nums[i] == target`, each matching index equally likely. Example: `[1,2,3,3,3]`, `pick(3)` returns 2, 3 or 4 with chance 1/3 each.
**Intuition:** It is reservoir sampling with k = 1, where the "stream" is only the matching positions.
**Approach 1 - Brute force:** Scan, collect all matching indices, return a random one. Time O(n) per pick, Space O(n) per call.
**Approach 2 - Better:** Pre-build a map `value -> list of indices`. Pick is O(1), but you store O(n) extra. Good when `pick` is called many times and memory is fine.
**Approach 3 - Optimal (memory):** Reservoir sampling. Scan once. On the `seen`-th match, replace the answer with probability `1/seen`. For `[1,2,3,3,3]`, target 3: match at index 2 (seen=1, keep with prob 1), at index 3 (seen=2, replace with prob 1/2), at index 4 (seen=3, replace with prob 1/3). Final chance of index 2: `1 * 1/2 * 2/3 = 1/3`. Same for the others.

```js
class RandomPickIndex {
  constructor(nums) { this.nums = nums; }
  pick(target) {
    let seen = 0, ans = -1;
    for (let i = 0; i < this.nums.length; i++) {
      if (this.nums[i] !== target) continue;
      seen++;
      if (Math.floor(Math.random() * seen) === 0) ans = i; // prob 1/seen
    }
    return ans;
  }
}
```

**Complexity:** Time O(n) per pick, Space O(1).
**Watch out:** Count only matches, not all elements. The check is `floor(random * seen) === 0`, which is true with probability exactly `1/seen`.

### 3. Linked List Random Node — Medium
**Companies:** Amazon, Google, Meta
**Problem:** Given a singly linked list, `getRandom()` returns a random node's value, each node equally likely. Example: `1 -> 2 -> 3`, each value has chance 1/3.
**Intuition:** You do not know the length and cannot jump to a position. Reservoir sampling needs one pass and no length.
**Approach 1 - Brute force:** Copy values into an array, then pick a random index. Time O(n) build, O(1) pick, Space O(n).
**Approach 2 - Better:** Count the length once, then walk to a random position each call. Time O(n) per call, Space O(1). It breaks if the list changes between calls.
**Approach 3 - Optimal:** Reservoir sampling (k = 1) on every call. It works even if the list grows, and it is the follow-up the interviewer wants. Trace on `1 -> 2 -> 3`: n=1 keep 1; n=2 replace with prob 1/2; n=3 replace with prob 1/3. Chance of 1 surviving: `1 * 1/2 * 2/3 = 1/3`.

```js
class LinkedListRandom {
  constructor(head) { this.head = head; }
  getRandom() {
    let ans, n = 0;
    for (let c = this.head; c; c = c.next) {
      n++;
      if (Math.floor(Math.random() * n) === 0) ans = c.val;
    }
    return ans;
  }
}
```

**Complexity:** Time O(n) per call, Space O(1).
**Watch out:** Use the count of nodes seen so far, not the final length. For "k random nodes" use the k-reservoir function from the Core Concepts section.

### 4. Implement Rand10() Using Rand7() — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** You are given `rand7()`, uniform on 1..7. Build `rand10()`, uniform on 1..10. Example: calling `rand10()` many times gives each of 1..10 about 10% of the time.
**Intuition:** Two calls to `rand7` can be combined into one number in 1..49, all equally likely: `(a-1)*7 + b`. 49 is not a multiple of 10, but 40 is. Keep 1..40 and retry on 41..49. Then map with `% 10`.
**Approach 1 - Brute force (wrong):** `rand7() + rand7()` is not uniform (sums near 8 are most common). `rand7() % 10` never gives 8, 9, 10. Both are wrong answers you may be tempted by.
**Approach 2 - Better:** Same idea, but reuse the discarded values 41..49 (they are uniform on 1..9). Combine them with another `rand7` to get a 63-value range, keep 60, and so on. It reduces the calls, but it is more code than most interviews need.
**Approach 3 - Optimal (simple):** Rejection sampling. Dry run: `a=3, b=5` gives `v = 2*7 + 5 = 19`, 19 <= 40, return `(19-1) % 10 + 1 = 9`. `a=7, b=7` gives `v = 49 > 40`, retry. Probability of accepting is 40/49, so the expected number of `rand7` pairs is 49/40 = 1.225.

```js
const rand7 = () => ri(1, 7);
function rand10() {
  while (true) {
    const v = (rand7() - 1) * 7 + rand7(); // 1..49 uniform
    if (v <= 40) return ((v - 1) % 10) + 1;
  }
}
```

**Complexity:** Time expected O(1) (about 2.45 calls to `rand7`), worst case unbounded but with probability shrinking fast. Space O(1).
**Watch out:** Build the 1..49 number as `(a-1)*7 + b`, not `a*7 + b` (that leaves a gap and is wrong). State the "unbounded worst case" yourself; the interviewer will ask.

### 5. Random Pick with Weight — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Given weights `w`, `pickIndex()` returns index `i` with probability `w[i] / sum(w)`. Example: `w = [1,3]` -> index 1 about 75% of the time.
**Intuition:** Lay the weights end to end on a number line. A random point lands in index `i`'s segment with chance proportional to its length. Find which segment with binary search.
**Approach 1 - Brute force:** Expand each index `w[i]` times into a big array and pick randomly. Space O(sum of weights), which can be huge.
**Approach 2 - Better:** Pick `t` in `1..total`, scan the weights linearly subtracting. O(n) per pick.
**Approach 3 - Optimal:** Prefix sums once, binary search per pick. For `w=[1,3,6]`, prefix `[1,4,10]`. If `t=5`, the first prefix `>= 5` is `10` at index 2. If `t=4`, the first prefix `>= 4` is `4` at index 1.

```js
class WeightedPick {
  constructor(w) {
    this.pre = [];
    let s = 0;
    for (const x of w) { s += x; this.pre.push(s); }
    this.total = s;
  }
  pickIndex() {
    const t = Math.floor(Math.random() * this.total) + 1; // 1..total
    let lo = 0, hi = this.pre.length - 1;
    while (lo < hi) {
      const mid = (lo + hi) >> 1;
      if (this.pre[mid] >= t) hi = mid; else lo = mid + 1;
    }
    return lo;
  }
}
```

**Complexity:** Time O(n) build, O(log n) per pick; Space O(n).
**Watch out:** `t` must be in `1..total` (not `0..total-1`) if you search for the first prefix `>= t`. Mixing the two conventions gives an off-by-one bias. Zero weights are never picked with this version, which is correct.

### 6. Insert Delete GetRandom O(1) — Medium
**Companies:** Amazon, Google, Meta, Microsoft, Uber
**Problem:** Design a set with `insert(v)`, `remove(v)` and `getRandom()`, all average O(1); `getRandom` returns each present element with equal chance. Example: insert 1, insert 2, remove 1, getRandom -> 2.
**Intuition:** An array gives O(1) random access. A hash map gives O(1) lookup. Array deletion is O(n) only because of shifting, so avoid shifting: **swap the deleted element with the last one**, then pop.
**Approach 1 - Brute force:** Array only. Insert is O(1) but search and delete are O(n).
**Approach 2 - Better:** Hash set only. O(1) insert and delete, but `getRandom` needs to convert to an array, O(n).
**Approach 3 - Optimal:** Array `arr` plus map `pos: value -> index`. Remove 2 from `[1,2,3]` (pos: 1->0, 2->1, 3->2): `i = 1`, `last = 3`. Write 3 into slot 1 -> `[1,3,3]`, set `pos[3] = 1`, pop -> `[1,3]`, delete `pos[2]`.

```js
class RandomizedSet {
  constructor() { this.arr = []; this.pos = new Map(); }
  insert(v) {
    if (this.pos.has(v)) return false;
    this.pos.set(v, this.arr.length);
    this.arr.push(v);
    return true;
  }
  remove(v) {
    if (!this.pos.has(v)) return false;
    const i = this.pos.get(v), last = this.arr[this.arr.length - 1];
    this.arr[i] = last;
    this.pos.set(last, i);
    this.arr.pop();
    this.pos.delete(v); // delete AFTER set: handles v === last
    return true;
  }
  getRandom() { return this.arr[Math.floor(Math.random() * this.arr.length)]; }
}
```

**Complexity:** Time O(1) average for all three, Space O(n).
**Watch out:** If you remove the last element itself, `last === v`. Update the map for `last` first and delete `v` afterwards, as the code does (the other order leaves a stale entry). Also check the set is not empty before `getRandom` if your interviewer asks.

### 7. Insert Delete GetRandom O(1) - Duplicates Allowed — Hard
**Companies:** Amazon, Google, Meta
**Problem:** Same as above, but duplicates are allowed. `insert` returns true if the value was not present before. `getRandom` picks proportionally to the count (two 1s and one 2: chance 2/3 for 1). `remove` deletes one copy.
**Intuition:** The array now holds copies. The map must store **all** indices of a value, in a set. Removal is the same swap-with-last trick, but now two index sets change.
**Approach 1 - Brute force:** Array only, linear search for removal, O(n).
**Approach 2 - Better:** Map value -> count plus rebuild an array for `getRandom`. O(n) random.
**Approach 3 - Optimal:** Array plus `Map<value, Set<index>>`. Remove value `v`: take any index `i` from `pos[v]` and remove it. Let `last` be the final array element at index `L`. If `i !== L`: write `last` into slot `i`, then in `pos[last]` delete `L` and add `i`. Pop the array. Trace: arr `[1,1,2]`, pos `{1:{0,1}, 2:{2}}`. Remove 1: `i=0`, `L=2`, last=2. arr -> `[2,1,2]`, `pos[2]`: delete 2, add 0 -> `{0}`. Pop -> `[2,1]`.

```js
class RandomizedCollection {
  constructor() { this.arr = []; this.pos = new Map(); } // val -> Set of indices
  insert(v) {
    const fresh = !this.pos.has(v) || this.pos.get(v).size === 0;
    if (!this.pos.has(v)) this.pos.set(v, new Set());
    this.pos.get(v).add(this.arr.length);
    this.arr.push(v);
    return fresh;
  }
  remove(v) {
    const s = this.pos.get(v);
    if (!s || s.size === 0) return false;
    const i = s.values().next().value;
    s.delete(i);
    const lastIdx = this.arr.length - 1, last = this.arr[lastIdx];
    if (i !== lastIdx) {
      this.arr[i] = last;
      const ls = this.pos.get(last);
      ls.delete(lastIdx);
      ls.add(i);
    }
    this.arr.pop();
    return true;
  }
  getRandom() { return this.arr[Math.floor(Math.random() * this.arr.length)]; }
}
```

**Complexity:** Time O(1) average, Space O(n).
**Watch out:** The case `i === L` (removing the copy that is already last) must skip the move step, otherwise you delete and re-add wrong indices. Test with a random stress test against a plain array model; I did this with 20,000 mixed operations.

### 8. Kth Largest Element in an Array — Medium
**Companies:** Amazon, Meta, Google, Microsoft, Apple
**Problem:** Return the kth largest value (counting duplicates). Example: `[3,2,1,5,6,4]`, k=2 -> 5.
**Intuition:** The kth largest sits at index `n-k` in sorted order. You do not need full sorting: partition around a pivot, and look at only the side that contains index `n-k`.
**Approach 1 - Brute force:** Sort, O(n log n).
**Approach 2 - Better:** Min-heap of size k, O(n log k), Space O(k). See [05 - Heaps](../05-heaps/README.md). It is the safe answer and works on streams.
**Approach 3 - Optimal:** Randomized quickselect with a 3-way partition (less, equal, greater). Target index for `[3,2,1,5,6,4]`, k=2 is 4. Pivot 4: less `[3,2,1]`, equal `[4]` at index 3, greater `[5,6]` at 4..5. Target 4 is in the greater part, so repeat on `[5,6]`: pivot 6 puts `5` below and `6` at index 5; target 4 is in the less part, which is `[5]`, so the answer is 5.

```js
function findKthLargest(nums, k) {
  const a = nums.slice(); // do not mutate caller's array
  const target = a.length - k; // index in sorted order
  let lo = 0, hi = a.length - 1;
  while (lo < hi) {
    const p = ri(lo, hi);
    [a[p], a[hi]] = [a[hi], a[p]];
    // 3-way partition: [lo..lt-1] < pivot, [lt..gt] == pivot, [gt+1..hi] > pivot
    const pivot = a[hi];
    let lt = lo, i = lo, gt = hi;
    while (i <= gt) {
      if (a[i] < pivot) { [a[i], a[lt]] = [a[lt], a[i]]; i++; lt++; }
      else if (a[i] > pivot) { [a[i], a[gt]] = [a[gt], a[i]]; gt--; }
      else i++;
    }
    if (target < lt) hi = lt - 1;
    else if (target > gt) lo = gt + 1;
    else return pivot;
  }
  return a[lo];
}
```

**Complexity:** Time expected O(n), worst O(n^2) (very unlikely with random pivots). Space O(n) for the copy, O(1) extra if you may modify the input.
**Watch out:** A 2-way partition goes quadratic on many equal values (I tested a 100,000-element array of all 7s); the 3-way partition avoids it. A fixed pivot (always last) is attackable by sorted input. Mention the heap alternative and its trade-off.

### 9. Generate Random Point in a Circle — Medium
**Companies:** Amazon, Google, Meta
**Problem:** Given a circle (radius `r`, center `(x, y)`), `randPoint()` returns a uniformly random point inside. Example: `r=1`, center `(0,0)` -> something like `[0.31, -0.62]`.
**Intuition:** "Uniform" means equal chance per unit of **area**, not per unit of radius. A ring at radius `ρ` has area proportional to `ρ`, so there are more points far from the center.
**Approach 1 - Brute force (wrong):** `radius = r * random()`, random angle. This crowds points near the center. The inner half-radius disc has only 25% of the area but would get 50% of the points.
**Approach 2 - Better:** Rejection sampling. Pick `(dx, dy)` uniformly in the square `[-r, r]^2`, accept if `dx^2 + dy^2 <= r^2`. The circle covers `pi/4 = 78.5%` of the square, so about 1.27 tries on average.
**Approach 3 - Optimal:** Use the inverse of the area: `radius = r * sqrt(random())`. The fraction of area within radius `ρ` is `(ρ/r)^2`; setting that equal to a uniform `u` gives `ρ = r * sqrt(u)`. I checked in Node that 25% of points fall within half the radius.

```js
class CirclePoint {
  constructor(r, x, y) { this.r = r; this.x = x; this.y = y; }
  randPoint() {
    const rad = this.r * Math.sqrt(Math.random()); // sqrt: area grows with r^2
    const ang = Math.random() * 2 * Math.PI;
    return [this.x + rad * Math.cos(ang), this.y + rad * Math.sin(ang)];
  }
}
```

The rejection version, also fine and often easier to explain:

```js
class CirclePointReject {
  constructor(r, x, y) { this.r = r; this.x = x; this.y = y; }
  randPoint() {
    while (true) {
      const dx = (Math.random() * 2 - 1) * this.r, dy = (Math.random() * 2 - 1) * this.r;
      if (dx * dx + dy * dy <= this.r * this.r) return [this.x + dx, this.y + dy];
    }
  }
}
```

**Complexity:** Time O(1) (rejection: expected O(1)), Space O(1).
**Watch out:** Forgetting `sqrt` is the classic bug. Do not forget to add the center `(x, y)`. Points exactly on the border are allowed in the standard problem.

### 10. Random Point in Non-overlapping Rectangles — Medium
**Companies:** Amazon, Google, Meta
**Problem:** Given non-overlapping axis-aligned rectangles `[x1, y1, x2, y2]` (corners inclusive, integer coordinates), `pick()` returns a random integer point that lies in some rectangle, all points equally likely. Example: `[[0,0,1,1],[5,5,5,7]]` has 4 + 3 = 7 integer points.
**Intuition:** First choose a rectangle **weighted by its number of points**, then choose a point uniformly inside it. That is exactly problem 5 (weighted pick) on top of a uniform pick.
**Approach 1 - Brute force:** List every integer point, pick one. Space O(total area), too big.
**Approach 2 - Better:** Choose a rectangle uniformly. Wrong: a small rectangle would get too many points.
**Approach 3 - Optimal:** Prefix sums of point counts `(x2-x1+1)*(y2-y1+1)`: here `[4, 7]`. Pick `t` in 1..7, binary search the rectangle, then pick `x` in `[x1,x2]` and `y` in `[y1,y2]` independently.

```js
// integer points in rectangles [x1,y1,x2,y2] (corners inclusive)
class RectPick {
  constructor(rects) {
    this.rects = rects;
    this.pre = [];
    let s = 0;
    for (const [x1, y1, x2, y2] of rects) {
      s += (x2 - x1 + 1) * (y2 - y1 + 1);
      this.pre.push(s);
    }
    this.total = s;
  }
  pick() {
    const t = Math.floor(Math.random() * this.total) + 1;
    let lo = 0, hi = this.pre.length - 1;
    while (lo < hi) {
      const mid = (lo + hi) >> 1;
      if (this.pre[mid] >= t) hi = mid; else lo = mid + 1;
    }
    const [x1, y1, x2, y2] = this.rects[lo];
    return [ri(x1, x2), ri(y1, y2)];
  }
}
```

**Complexity:** Time O(n) build, O(log n) per pick; Space O(n).
**Watch out:** Count points with `+1` on both sides (corners are inclusive). If the rectangles are given with real coordinates and "area" instead, drop the `+1`. Overlapping rectangles would break fairness, which is why the problem forbids it.

### 11. Flip Matrix — Medium
**Companies:** Google, Amazon, Meta
**Problem:** An `m x n` grid starts all 0. `flip()` picks a random cell that is still 0, sets it to 1 and returns `[row, col]`. `reset()` sets everything back to 0. Minimize calls to `random`.
**Intuition:** Treat the grid as one list of ids `0..m*n-1`. Each flip is "pick a random unused id". That is Fisher-Yates done lazily: swap the chosen id with the last unused id and shrink. We cannot store `m*n` ids (can be huge), so keep a hash map of only the swaps that happened.
**Approach 1 - Brute force:** Pick a random cell, retry if already 1. Slows down as the grid fills (coupon collector). Expected worst O(m*n) tries for the last cell.
**Approach 2 - Better:** Store a list of all free ids and swap-remove. O(1) per flip, but Space O(m*n) up front.
**Approach 3 - Optimal:** Map `virtual slot -> real id`, with default `slot -> slot`. Trace for a 1x3 grid (ids 0,1,2), `left=3`: flip with `r=1`: `left` becomes 2, answer id `1` (no entry), then `map[1] = (map[2] ?? 2) = 2`. Free ids are now slots 0..1 which hold ids `0, 2`. Next flip with `r=1`: `left` becomes 1, answer `map[1] = 2`. The remaining free id is 0. Each id came out exactly once.

```js
class FlipMatrix {
  constructor(m, n) { this.m = m; this.n = n; this.left = m * n; this.map = new Map(); }
  flip() {
    const r = Math.floor(Math.random() * this.left--); // r in [0, left-1] before decrement
    const cell = this.map.has(r) ? this.map.get(r) : r;
    // move the last still-free id into slot r
    this.map.set(r, this.map.has(this.left) ? this.map.get(this.left) : this.left);
    return [Math.floor(cell / this.n), cell % this.n];
  }
  reset() { this.left = this.m * this.n; this.map.clear(); }
}
```

**Complexity:** Time O(1) per flip and reset is O(k) (clearing the map of the k flips done), Space O(k) after k flips.
**Watch out:** Read `left` carefully: `this.left--` returns the old value, so `r` is drawn from `0..left-1` of the old count, then `this.left` is the new last slot. Row = `floor(id / n)`, col = `id % n`, with `n` = number of columns.

### 12. Random Pick with Blacklist — Hard
**Companies:** Google, Amazon, Meta
**Problem:** Given `n` and a list of blacklisted numbers in `[0, n)`, `pick()` returns a uniform random number in `[0, n)` that is not blacklisted. Minimize random calls. Example: `n=7`, blacklist `[2,3,5]` -> returns 0, 1, 4 or 6 with chance 1/4 each.
**Intuition:** There are exactly `size = n - len(blacklist)` valid numbers. Pick `r` uniformly in `[0, size)`. If `r` is fine, return it. If it is blacklisted, return a pre-assigned good number from the tail `[size, n)` instead. The tail has the same count of good numbers as there are blacklisted ones in `[0, size)`, so the mapping is one to one.
**Approach 1 - Brute force:** Rejection: pick in `[0, n)` and retry if blacklisted. Many random calls when most numbers are blacklisted, and unbounded in theory.
**Approach 2 - Better:** Build the sorted list of good numbers (Space O(n)). Too big when `n` is 10^9.
**Approach 3 - Optimal:** Remap. Trace `n=7`, blacklist `{2,3,5}`: `size = 4`. Blacklisted numbers below 4 are 2 and 3. Walk `free` over the tail starting at 4: 4 is good, so `2 -> 4`. 5 is blacklisted, skip; 6 is good, so `3 -> 6`. Now picks come from `{0, 1, 4, 6}`, each 1/4.

```js
class BlacklistPick {
  constructor(n, blacklist) {
    this.size = n - blacklist.length; // valid ids are 0..size-1 after remapping
    const bad = new Set(blacklist);
    this.map = new Map();
    let free = this.size; // scan the tail [size, n) for good numbers
    for (const b of blacklist) {
      if (b >= this.size) continue;
      while (bad.has(free)) free++;
      this.map.set(b, free++);
    }
  }
  pick() {
    const r = Math.floor(Math.random() * this.size);
    return this.map.has(r) ? this.map.get(r) : r;
  }
}
```

**Complexity:** Time O(B) build (B = blacklist size, one pass; the `free` pointer only moves forward), O(1) per pick; Space O(B).
**Watch out:** Skip blacklisted numbers `>= size` when building the map: they are never drawn. Advance `free` past blacklisted tail values (the `while`). Test small edge cases like `n=3, blacklist=[1]` and a blacklist that fills the entire tail.

## Common Mistakes
- **"Pick `j` from the whole array each time is a fine shuffle."** It is biased (`n^n` paths cannot split evenly into `n!`). Use `j` in `0..i`.
- **"Sorting by `Math.random() - 0.5` shuffles."** Comparators must be consistent. This produces biased and engine-dependent results. Use Fisher-Yates.
- **"`rand7() % 10` or `rand7() + rand7()` gives rand10."** Neither is uniform. Build a uniform range first, then reject.
- **"`Math.floor(Math.random() * n) + 1` is the same as `ri(0, n)`."** Mind inclusive and exclusive ends. Write one helper and test the edges.
- **"Random radius gives a random point in a circle."** It over-samples the center. Use `sqrt`.
- **"Removing from the array by `splice` is fine, it is only one line."** `splice` is O(n). Swap with last, then `pop`.
- **"Reservoir: replace with probability 1/k."** It is `1/i` (the index so far) for k = 1, and `k/i` for k items.
- **"Quickselect with a 2-way partition is O(n)."** Many duplicates push it to O(n^2). Use 3-way or a random pivot plus 3-way.
- **"My program printed a random-looking answer, so it is fair."** One run proves nothing. Test with many runs and compare counts to the expected ones.

## Interview Tips
- **Say the probability proof out loud.** For reservoir: "item i is picked with 1/i and survives with i/n, so 1/n." For Fisher-Yates: "n! equally likely paths."
- **Name the random source first.** "I assume a function that returns an integer in `[0, n)`; in JS that is `Math.floor(Math.random() * n)`."
- **Ask about repeated calls.** Many calls favour pre-processing (prefix sums, maps). Few calls favour O(1) memory (reservoir).
- **Mention the true cost of rejection.** Expected constant time, unbounded worst case. Give the acceptance probability (40/49, pi/4).
- **Test fairness, do not trust your eyes.** Say you would run it 100,000 times and compare counts to the expected counts (a chi-square style check). It shows engineering maturity.
- **Edge cases to mention:** empty or single-element input, `k >= n`, zero weights, all values equal (quickselect), removing the last element (O(1) set), full grid (flip), blacklist covering the whole tail.
- **Follow-ups you may get:** "stream with unknown length" (reservoir), "k items" (k-reservoir), "weights change over time" (Fenwick tree for prefix sums; see [02 - Segment Tree and Fenwick](../../06-advanced/02-segment-tree-and-fenwick/README.md)), "worst-case O(n) kth element" (median of medians), "distributed sampling" (sample per machine, then combine with weights), "cryptographic randomness" (`Math.random` is not secure; use `crypto.randomInt` in Node).
- **Bloom filter one-liner:** "No false negatives, tunable false positives, no deletes, very small memory."

## Practice List
**Easy**
- Random Pick Index (try the map version first)
- Shuffle the Array (the "interleave" problem, different from shuffle but good warm-up)
- Kth Largest Element in a Stream

**Medium**
- Shuffle an Array
- Linked List Random Node
- Implement Rand10() Using Rand7()
- Random Pick with Weight
- Insert Delete GetRandom O(1)
- Kth Largest Element in an Array
- Generate Random Point in a Circle
- Random Point in Non-overlapping Rectangles
- Flip Matrix
- Top K Frequent Elements (quickselect variant)
- K Closest Points to Origin (quickselect variant)

**Hard**
- Insert Delete GetRandom O(1) - Duplicates Allowed
- Random Pick with Blacklist
- Median of Two Sorted Arrays (selection idea)
- Sliding Window Median (heap or ordered structure, related to selection)

## Key Takeaways
- **Fisher-Yates with `j` in `0..i`** gives every permutation exactly `1/n!`; the naive "random index every time" is biased.
- **Reservoir sampling** picks uniformly from a stream with O(1) memory: replace with probability `1/i` (k = 1) or `k/i` (k items).
- **Prefix sums plus binary search** turn weights into a random pick in O(log n).
- **Swap with last, then pop** makes array deletion O(1); pair it with a hash map of positions for O(1) set operations with `getRandom`.
- **Rejection sampling** keeps uniformity because you only discard; build a bigger uniform range first (`(a-1)*7 + b`).
- **Uniform in area means `sqrt`** for the radius; fairness is about the measure you care about.
- **Random pivot plus 3-way partition** gives expected O(n) quickselect; always say the worst case out loud.

---
[← Previous: Matrix](../08-matrix/README.md) | [Next: Binary Tree Traversals →](../../03-trees/01-binary-tree-traversals/README.md) | [Back to Intermediate Index](../README.md)
