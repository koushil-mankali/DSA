# Sparse Table and Sqrt Decomposition

> **Estimated Time:** 3 hr | **Difficulty:** Advanced
> **Prerequisites:** [07 - Prefix Sum](../../01-core-linear/07-prefix-sum/README.md), [02 - Segment Tree and Fenwick](../02-segment-tree-and-fenwick/README.md), [03 - Paths, LCA and Tree Properties](../../03-trees/03-paths-lca-and-tree-properties/README.md)

---

## What You'll Learn
- You can build a sparse table in O(n log n) and answer min, max and gcd range queries in O(1).
- You can explain why the "two overlapping blocks" trick works for min, max and gcd but breaks for sum.
- You can use sqrt decomposition for range queries with point updates, and know when it beats a segment tree (when the merge rule is awkward).
- You can describe Mo's algorithm for offline range queries, like "how many distinct values in [l, r]".
- You can pick the right tool (prefix sums, sparse table, Fenwick, segment tree, sqrt) from the problem statement.

## The Big Idea

Think of a library with a very long shelf of books, each with a page count. A visitor asks, "What is the thinnest book between position 40 and position 97?" You could walk the shelf every time. Or, before the library opens, you write a cheat sheet: "thinnest book in each run of 1, 2, 4, 8, 16 ... books starting at each position". Now any question is answered by looking at two entries on the sheet. The sheet took effort to make, but the shelf never changes, so you only make it once.

That cheat sheet is a **sparse table**. It only works when the array is **static** (no updates). If books are added or swapped, you need something that can change, like a segment tree or a Fenwick tree. **Sqrt decomposition** is a simpler middle path: cut the shelf into chunks of about sqrt(n) books, and keep a note for each chunk.

Why interviewers love it:
- It tests whether you know **tools beyond the standard ones**. Many people only know "prefix sum" and "segment tree".
- The overlap trick (idempotent operations) is a neat idea that is easy to ask "why does this fail for sum?".
- LCA through Euler tour plus RMQ links trees and arrays, which is a favourite "connect two ideas" question.
- Mo's algorithm shows up when constraints are around 10^5 and the answer is hard to merge.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| Many queries, array never changes, min / max / gcd of a range | Sparse table (O(1) query) |
| Many queries, array never changes, range sum | Prefix sums (simpler and faster) |
| Range sum or count with point updates | Fenwick tree (shortest code) |
| Range query with updates and a tricky merge (min, max, gcd, custom) | Segment tree |
| Updates and queries, and you can not find a clean tree merge | Sqrt decomposition |
| LCA queries on a static tree, many of them | Euler tour + sparse table |
| Offline queries (you can read them all first), answer hard to merge, "distinct in range" | Mo's algorithm |
| "Count of values <= x in [l, r]" and queries can be reordered | Offline sort + Fenwick |
| Sliding window max for every window | Monotonic deque first, sparse table is the lazy alternative |

Which tool to use, side by side (n = array size, q = number of queries):

| Tool | Build | Query | Point update | Works for | Notes |
|---|---|---|---|---|---|
| Prefix sums | O(n) | O(1) | O(n) rebuild | sum, xor, count (needs an inverse) | Static only. Simplest. |
| Sparse table | O(n log n) | O(1) | O(n log n) rebuild | min, max, gcd, and, or (idempotent) | Static only. Extra memory n log n. |
| Fenwick tree | O(n) or O(n log n) | O(log n) | O(log n) | sum, count (needs an inverse) | Very short code. |
| Segment tree | O(n) | O(log n) | O(log n) | any associative merge | Most flexible. Lazy version does range updates. |
| Sqrt decomposition | O(n) | O(sqrt n) | O(1) for sum, O(sqrt n) for min/max | almost anything | Easy to write, slower than a tree. |
| Mo's algorithm | O((n + q) sqrt n) total | offline only | none | "add one / remove one" friendly questions | Needs all queries up front. |

Rule of thumb: **static + idempotent -> sparse table; static + sum -> prefix sums; updates -> Fenwick or segment tree.**

## Core Concepts / Template

### 1. Sparse table

Define `t[k][i]` = answer for the block of length `2^k` that starts at `i`, that is positions `i .. i + 2^k - 1`.

- Level 0 is the array itself (blocks of length 1).
- A block of length `2^k` is two blocks of length `2^(k-1)` glued together:
  `t[k][i] = op(t[k-1][i], t[k-1][i + 2^(k-1)])`.
- There are about log2(n) levels, each with up to n entries. Build time and memory: **O(n log n)**.

**Query [l, r]:** let `len = r - l + 1` and `k = floor(log2(len))`. The two blocks `[l, l + 2^k - 1]` and `[r - 2^k + 1, r]` each have length `2^k` and together cover `[l, r]`. They overlap in the middle, and that is fine **if the operation does not care about seeing a value twice**:

```
index:   0  1  2  3  4  5  6
array:   5  2  4  7  1  3  6        query [1, 5], len 5, k = 2 (block length 4)
block A:    [2  4  7  1]             t[2][1]
block B:       [4  7  1  3]          t[2][2]   (starts at 5 - 4 + 1 = 2)
overlap:       [4  7  1]
answer = min(t[2][1], t[2][2]) = min(1, 1) = 1
```

This property is called **idempotent**: `op(x, x) = x`. min, max, gcd, bitwise and, bitwise or are idempotent. **Sum is not**: `x + x = 2x`, so the overlap would be counted twice. For sum (or xor, product) the table can still be used, but you must split the range into non-overlapping power-of-two blocks, which costs O(log n) per query. That is no better than a Fenwick tree, so for sum use prefix sums instead.

Trace of the build for `[5, 2, 4, 7, 1, 3, 6]` with min:

| level k | block length | entries |
|---|---|---|
| 0 | 1 | 5 2 4 7 1 3 6 |
| 1 | 2 | 2 2 4 1 1 3 |
| 2 | 4 | 2 1 1 1 |

Query [1, 3]: len 3, k = 1, answer = min(t[1][1], t[1][3 - 2 + 1 = 2]) = min(2, 4) = 2.

**Getting floor(log2(len)) in JavaScript:** `31 - Math.clz32(len)`. It is O(1) and exact. Do not use `Math.log2` for this; floating point can be off by one near powers of two on some inputs.

```js
const lg = (n) => 31 - Math.clz32(n);

// op must be idempotent: min, max, gcd, ...
function buildSparse(a, op) {
  const n = a.length, K = lg(n) + 1;
  const t = [a.slice()];
  for (let k = 1; k < K; k++) {
    const prev = t[k - 1];
    const cur = new Array(n - (1 << k) + 1);
    for (let i = 0; i + (1 << k) <= n; i++) {
      cur[i] = op(prev[i], prev[i + (1 << (k - 1))]);
    }
    t.push(cur);
  }
  // query inclusive [l, r]; assumes 0 <= l <= r < n
  return (l, r) => {
    const k = lg(r - l + 1);
    return op(t[k][l], t[k][r - (1 << k) + 1]);
  };
}

const query = buildSparse([5, 2, 4, 7, 1, 3, 6], Math.min);
// query(1, 3) -> 2, query(1, 5) -> 1
```

Limits: the table is **static**. One change means a rebuild in O(n log n). For n = 10^5 the table holds about 1.7 million numbers, which is fine. For n = 10^6 it is about 20 million, which starts to hurt in JavaScript; use typed arrays (`Int32Array`) then.

### 2. Sqrt decomposition

Cut the array into blocks of size `B ~ sqrt(n)`. Keep one summary value per block (sum, min, ...).

- **Point update:** change the element, fix its block summary. For sum this is O(1) (add the difference). For min/max you may need to rescan the block: O(sqrt n).
- **Range query [l, r]:** take the partial block on the left element by element, the partial block on the right element by element, and the whole blocks in between using summaries. At most about 2 sqrt(n) single elements plus sqrt(n) block summaries: **O(sqrt n)**.

```
n = 9, B = 3
index:   0 1 2 | 3 4 5 | 6 7 8
blocks:  [ b0 ]  [ b1 ]  [ b2 ]
query [1, 7]:  1,2 (left part)  + b1 (whole)  + 6,7 (right part)
```

Why use it when a segment tree exists? The code is short, hard to get wrong, and the "summary" can be something a tree can not merge easily (for example, a sorted copy of the block, or a frequency table). The price is O(sqrt n) instead of O(log n). For n = 10^5, sqrt is about 316, so 10^5 queries cost about 3 x 10^7 steps: fine.

### 3. Mo's algorithm (brief)

Offline idea. You read all queries first. You keep a "current window" `[L, R]` and a data structure that supports **add one element** and **remove one element** in O(1). To answer a query you slide the window to its `[l, r]` by moving L and R one step at a time.

If you answer queries in the given order, L and R jump around and it costs O(n) per query. Mo's trick is to **sort the queries**: group them by `floor(l / B)` where `B ~ sqrt(n)`, and inside a group sort by `r`. Then:
- Within a group, R only moves forward: at most n per group, so n * (n / B) total.
- L stays inside one block of size B: at most B per query, so q * B total.
- With `B = sqrt(n)`, the total is **O((n + q) sqrt n)**.

A small speed-up: sort `r` ascending in even groups and descending in odd groups, so R does not run back to the start every group.

Use it when: queries are offline, there are no updates, and you can add/remove one element while keeping the answer (distinct count, number of pairs, frequency-of-frequency). Do not use it when a simple structure (prefix sums, sparse table) already gives the answer.

### 4. Which ones to remember for the interview

If the interviewer asks "can you do better than a segment tree?", the honest answer is: for **static + min/max/gcd** a sparse table gives O(1) queries. For everything else the segment tree or Fenwick tree is the normal answer, and sqrt decomposition or Mo's is a fallback when the merge is hard.

## Problems (easy -> hard)

### 1. Static Range Max Queries — Easy
**Companies:** -
**Problem:** Given an array that never changes and many queries `[l, r]`, return the maximum of each range. Example: `a = [3, 9, 2, 7, 5]`, query `[1, 3]` -> `9`.
**Intuition:** The array is static and max is idempotent. Two overlapping power-of-two blocks cover any range, and seeing a value twice does not change a max.
**Approach 1 - Brute force:** scan the range for each query. Time O(n) per query, Space O(1).
**Approach 2 - Better:** a segment tree. Time O(log n) per query, Space O(n).
**Approach 3 - Optimal:** sparse table with `Math.max`. Build once, answer in O(1).

Trace on `[3, 9, 2, 7, 5]`, query `[1, 3]`: len 3, k = 1. Blocks are `[1, 2]` = max(9, 2) = 9 and `[2, 3]` = max(2, 7) = 7. Answer = max(9, 7) = 9.

```js
// uses buildSparse from Core Concepts
function staticMax(a, queries) {
  const q = buildSparse(a, Math.max);
  return queries.map(([l, r]) => q(l, r));
}
// staticMax([3, 9, 2, 7, 5], [[1, 3], [0, 0], [2, 4]]) -> [9, 3, 7]
```

**Complexity:** Time O(n log n + q), Space O(n log n).
**Watch out:** queries are inclusive on both ends here. If your input uses `[l, r)`, convert to `r - 1`. An empty range is not allowed by this code.

### 2. Range Minimum Query — Easy
**Companies:** -
**Problem:** Same as above but the minimum, and the array is given once with up to 10^5 elements and 10^5 queries. Example: `a = [5, 2, 4, 7, 1, 3, 6]`, query `[1, 3]` -> `2`, query `[1, 5]` -> `1`.
**Intuition:** This is the textbook sparse table problem (often called RMQ). The same template with `Math.min`.
**Approach 1 - Brute force:** scan each range. Time O(n) per query, Space O(1).
**Approach 2 - Better:** segment tree. O(log n) per query.
**Approach 3 - Optimal:** sparse table, O(1) per query. If the question wants the **index** of the minimum, store indices in the table and compare `a[i] <= a[j]` (see problem 9).

Trace (the build table is in Core Concepts): query `[1, 5]`, len 5, k = 2. Blocks `[1..4]` and `[2..5]`. `t[2][1] = 1`, `t[2][2] = 1`. Answer 1. Note both blocks contain the minimum 1 (at index 4): that is the overlap, and it is harmless.

```js
function rangeMin(a) {
  const q = buildSparse(a, Math.min);
  return q; // q(l, r)
}
const rmq = rangeMin([5, 2, 4, 7, 1, 3, 6]);
// rmq(1, 3) -> 2, rmq(1, 5) -> 1, rmq(6, 6) -> 6
```

**Complexity:** Time O(n log n) build, O(1) per query. Space O(n log n).
**Watch out:** if updates appear in the statement, a sparse table is the wrong tool. Say so and switch to a segment tree. Also, `Math.min(...bigArray)` is not a fix for anything here; it can overflow the call stack for huge arrays.

### 3. Range GCD Query — Medium
**Companies:** -
**Problem:** For a static array, answer queries "what is the gcd of `a[l..r]`?". Example: `a = [12, 18, 24, 36, 5]`, query `[0, 3]` -> `6`, query `[2, 4]` -> `1`.
**Intuition:** gcd is idempotent: `gcd(x, x) = x`. Also gcd is associative and commutative, so the order of combining does not matter. That is exactly what the overlap trick needs.
**Approach 1 - Brute force:** fold gcd over the range. O(n log(max)) per query.
**Approach 2 - Better:** segment tree. O(log n * log(max)) per query.
**Approach 3 - Optimal:** sparse table with gcd. Each query makes one gcd call.

Trace on `[12, 18, 24, 36, 5]`, query `[0, 3]`: len 4, k = 2, so one block `[0..3]` is used twice. Level 1: `gcd(12, 18) = 6`, `gcd(18, 24) = 6`, `gcd(24, 36) = 12`, `gcd(36, 5) = 1`. Level 2 at 0 = `gcd(t[1][0], t[1][2]) = gcd(6, 12) = 6`. The answer is 6.

```js
const gcd = (a, b) => {
  while (b) [a, b] = [b, a % b];
  return a;
};

function rangeGcd(a) {
  return buildSparse(a, gcd); // q(l, r)
}
const g = rangeGcd([12, 18, 24, 36, 5]);
// g(0, 3) -> 6, g(2, 4) -> 1
```

**Complexity:** Time O(n log n log(max)) build, O(log(max)) per query (one gcd call). Space O(n log n).
**Watch out:** `gcd(0, x) = x` and `gcd(0, 0) = 0` with this code, which is the usual convention. Numbers must be non-negative integers; take `Math.abs` first if negatives are allowed. For lcm the trick does **not** work (lcm is idempotent too, but values blow up past 2^53).

### 4. Range Sum Query - Mutable (sqrt decomposition) — Medium
**Companies:** Amazon, Google, Meta
**Problem:** Support `update(i, value)` and `sumRange(l, r)` on an array. This is LeetCode "Range Sum Query - Mutable". Example: `[1, 3, 5]`, `sumRange(0, 2)` -> `9`; `update(1, 2)`; `sumRange(0, 2)` -> `8`.
**Intuition:** Keep a sum per block. An update changes one element, so only one block sum changes. A query adds whole block sums in the middle and loose elements at the two ends.
**Approach 1 - Brute force:** update O(1), query O(n) by scanning. Or keep prefix sums: query O(1), update O(n).
**Approach 2 - Better:** sqrt decomposition: update O(1), query O(sqrt n).
**Approach 3 - Optimal:** a Fenwick tree or segment tree: both operations O(log n). Shown in [the previous chapter](../02-segment-tree-and-fenwick/README.md). The sqrt version below is the simplest to write from memory.

Trace: `a = [1, 3, 5, 7, 9]`, `B = 2`, block sums `[4, 12, 9]`. `sumRange(1, 4)`: left block 0, right block 2, different. Left part: index 1 (to end of block 0) = 3. Whole blocks strictly between: block 1 = 12. Right part: indices 4..4 = 9. Total 24. Check: 3 + 5 + 7 + 9 = 24.

```js
class SqrtSum {
  constructor(a) {
    this.a = a.slice();
    this.B = Math.max(1, Math.floor(Math.sqrt(a.length)));
    this.s = new Array(Math.ceil(a.length / this.B)).fill(0);
    a.forEach((v, i) => (this.s[Math.floor(i / this.B)] += v));
  }
  update(i, v) {
    this.s[Math.floor(i / this.B)] += v - this.a[i]; // only the difference
    this.a[i] = v;
  }
  query(l, r) {
    const { a, s, B } = this;
    const bl = Math.floor(l / B), br = Math.floor(r / B);
    let res = 0;
    if (bl === br) {                       // inside one block
      for (let i = l; i <= r; i++) res += a[i];
      return res;
    }
    for (let i = l; i < (bl + 1) * B; i++) res += a[i];
    for (let b = bl + 1; b < br; b++) res += s[b];
    for (let i = br * B; i <= r; i++) res += a[i];
    return res;
  }
}
// const x = new SqrtSum([1, 3, 5]); x.query(0, 2) -> 9; x.update(1, 2); x.query(0, 2) -> 8
```

**Complexity:** Time O(n) build, O(1) update, O(sqrt n) query. Space O(n).
**Watch out:** the same-block case is the classic bug (the left loop would walk past `r`). Update must add `v - old`, not `v`. For min/max blocks, an update that raises the block's current min needs a rescan of that block.

### 5. Sliding Window Maximum via Sparse Table — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** For every window of size k in an array, return the maximum. This is LeetCode "Sliding Window Maximum" (also "maximum of all subarrays of size k"). Example: `[1, 3, -1, -3, 5, 3, 6, 7]`, k = 3 -> `[3, 3, 5, 5, 6, 7]`.
**Intuition:** Every window is just a range query, and the array is static. So a sparse table answers each window in O(1). This is a reference solution: the **monotonic deque** is the real interview answer (O(n) time, O(k) space), covered in [Monotonic Stack and Queue](../../01-core-linear/11-monotonic-stack-queue/README.md).
**Approach 1 - Brute force:** scan every window. Time O(n k), Space O(1).
**Approach 2 - Better:** sparse table, or a heap with lazy deletion. Time O(n log n), Space O(n log n).
**Approach 3 - Optimal:** monotonic deque. Time O(n), Space O(k). Mention it, then show the sparse table only if asked for an "other way".

Trace: windows start at `0..5`. Window at 2 is `[-1, -3, 5]`, len 3, k = 1, blocks `[2..3]` = max(-1, -3) = -1 and `[3..4]` = max(-3, 5) = 5. Answer 5.

```js
// uses buildSparse
function maxSlidingWindow(nums, k) {
  const q = buildSparse(nums, Math.max);
  const out = [];
  for (let i = 0; i + k <= nums.length; i++) out.push(q(i, i + k - 1));
  return out;
}
// maxSlidingWindow([1, 3, -1, -3, 5, 3, 6, 7], 3) -> [3, 3, 5, 5, 6, 7]
```

**Complexity:** Time O(n log n), Space O(n log n).
**Watch out:** the memory is the downside; with n = 10^5 it is fine, but the deque uses far less. If k > n, the loop returns an empty array (check what the problem expects). Do not claim this beats the deque.

### 6. Lowest Common Ancestor via Euler Tour + Sparse Table — Hard
**Companies:** Google, Amazon, Meta
**Problem:** For a rooted tree with n nodes, answer many queries "what is the lowest common ancestor of u and v?". Example: edges `0-1, 0-2, 1-3, 1-4`, root 0. LCA(3, 4) = 1, LCA(3, 2) = 0, LCA(1, 3) = 1.
**Intuition:** Walk the tree depth first and write down a node **every time you arrive at it, including when you come back up from a child**. This is the Euler tour. Between the first visit of u and the first visit of v, the walk goes up to the LCA and down again. So the LCA is the **node with the smallest depth** in that slice of the tour. That is a range minimum query, and the tree is static, so we use a sparse table.
**Approach 1 - Brute force:** walk up from both nodes using parent pointers and depths. Time O(depth) per query, can be O(n).
**Approach 2 - Better:** binary lifting. O(n log n) build, O(log n) per query. See [Paths, LCA and Tree Properties](../../03-trees/03-paths-lca-and-tree-properties/README.md).
**Approach 3 - Optimal:** Euler tour + sparse table. O(n log n) build, **O(1)** per query.

Trace on the example tree (0 has children 1, 2; 1 has children 3, 4):

```
euler : 0  1  3  1  4  1  0  2  0
index : 0  1  2  3  4  5  6  7  8
depth : 0  1  2  1  2  1  0  1  0
first : 0->0, 1->1, 3->2, 4->4, 2->7
LCA(3, 4): slice [2..4] = 3 1 4, smallest depth is node 1  -> 1
LCA(3, 2): slice [2..7] = 3 1 4 1 0 2, smallest depth is node 0 -> 0
```

The tour has 2n - 1 entries. The table stores **node ids** and compares them by depth.

```js
// uses buildSparse; iterative DFS so a long chain does not overflow the call stack
function makeLCA(n, edges, root = 0) {
  const adj = Array.from({ length: n }, () => []);
  for (const [u, v] of edges) { adj[u].push(v); adj[v].push(u); }
  const euler = [], depth = new Array(n).fill(0), first = new Array(n).fill(-1);
  const stack = [[root, -1, 0]];            // [node, parent, next child index]
  first[root] = 0;
  euler.push(root);
  while (stack.length) {
    const top = stack[stack.length - 1];
    const [u, p] = top;
    if (top[2] < adj[u].length) {
      const v = adj[u][top[2]++];
      if (v === p) continue;
      depth[v] = depth[u] + 1;
      first[v] = euler.length;
      euler.push(v);
      stack.push([v, u, 0]);
    } else {
      stack.pop();
      if (stack.length) euler.push(stack[stack.length - 1][0]); // back at the parent
    }
  }
  const q = buildSparse(euler, (x, y) => (depth[x] <= depth[y] ? x : y));
  return (u, v) => {
    let l = first[u], r = first[v];
    if (l > r) [l, r] = [r, l];
    return q(l, r);
  };
}
// const lca = makeLCA(5, [[0,1],[0,2],[1,3],[1,4]]);
// lca(3, 4) -> 1, lca(3, 2) -> 0, lca(1, 3) -> 1, lca(2, 2) -> 2
```

**Complexity:** Time O(n log n) build, O(1) per query. Space O(n log n) (the tour has 2n - 1 entries).
**Watch out:** push the parent again after returning from **every** child, or the slice will miss the path through the LCA. Use first occurrence, and swap so `l <= r`. The combine function takes the shallower node, and it is idempotent (`op(x, x) = x`), so the overlap trick is valid. A recursive DFS can blow the stack on a 10^5-long chain, hence the explicit stack.

### 7. Count Distinct Values in a Range (Mo's Algorithm) — Hard
**Companies:** -
**Problem:** Given an array and many queries `[l, r]`, return how many different values appear in each range. Example: `a = [1, 2, 1, 3, 2]`, query `[0, 2]` -> `2`, query `[1, 4]` -> `3`.
**Intuition:** A distinct count can not be merged from two halves (the halves may share values), so a segment tree does not fit directly. But "add one element" and "remove one element" are trivial with a count array. Mo's order makes the window move only O((n + q) sqrt n) steps in total.
**Approach 1 - Brute force:** put the range in a Set for each query. Time O(n) per query, Space O(n).
**Approach 2 - Better:** offline with a Fenwick tree (sort by `r`, keep only the last occurrence of each value). Time O((n + q) log n). This is faster, and worth mentioning.
**Approach 3 - Optimal for the "Mo" topic:** Mo's algorithm. Time O((n + q) sqrt n).

Trace on `a = [1, 2, 1, 3, 2]`, `B = 2`. Queries: Q0 = `[0, 2]`, Q1 = `[1, 4]`. Q0 is in block 0, Q1 in block 0 (`floor(1/2) = 0`), so they sort by `r`: Q0, then Q1.

| step | window [L, R] | counts | distinct |
|---|---|---|---|
| start | [0, -1] (empty) | - | 0 |
| add 0, 1, 2 | [0, 2] | 1:2, 2:1 | 2 (answer Q0) |
| add 3, 4 | [0, 4] | 1:2, 2:2, 3:1 | 3 |
| remove 0 | [1, 4] | 1:1, 2:2, 3:1 | 3 (answer Q1) |

```js
function distinctInRanges(a, queries) {
  const n = a.length, B = Math.max(1, Math.floor(Math.sqrt(n)));
  const ids = new Map();                       // compress values to 0..m-1
  const c = a.map((v) => {
    if (!ids.has(v)) ids.set(v, ids.size);
    return ids.get(v);
  });
  const cnt = new Array(ids.size).fill(0);
  const order = queries.map((_, i) => i).sort((x, y) => {
    const bx = Math.floor(queries[x][0] / B), by = Math.floor(queries[y][0] / B);
    if (bx !== by) return bx - by;
    return bx % 2 === 0 ? queries[x][1] - queries[y][1] : queries[y][1] - queries[x][1];
  });
  const ans = new Array(queries.length);
  let L = 0, R = -1, distinct = 0;
  const add = (i) => { if (cnt[c[i]]++ === 0) distinct++; };
  const rem = (i) => { if (--cnt[c[i]] === 0) distinct--; };
  for (const qi of order) {
    const [l, r] = queries[qi];
    while (L > l) add(--L);                    // expand first, shrink later:
    while (R < r) add(++R);                    // the window never goes "negative"
    while (L < l) rem(L++);
    while (R > r) rem(R--);
    ans[qi] = distinct;
  }
  return ans;
}
// distinctInRanges([1, 2, 1, 3, 2], [[0, 2], [1, 4]]) -> [2, 3]
```

**Complexity:** Time O((n + q) sqrt n), Space O(n + q).
**Watch out:** the order of the four `while` loops matters: expand before you shrink, or the window can briefly have L > R + 1 and counts go negative. Return answers in the original query order (we store by `qi`). Compress values first so `cnt` is a small array. The built-in sort is O(q log q), which is small next to the sqrt term.

### 8. Count of Elements <= x in a Range (Offline) — Hard
**Companies:** -
**Problem:** Given an array and queries `[l, r, x]`, return how many elements in `a[l..r]` are `<= x`. Example: `a = [5, 1, 4, 2, 3]`, query `[1, 3, 3]` -> `2` (the values 1, 4, 2 give 1 and 2), query `[0, 4, 5]` -> `5`.
**Intuition:** If we process queries in increasing `x`, we can insert array elements in increasing value order. When we reach a query, **exactly the elements with value <= x are inserted**. Now the question is just "how many inserted positions are in `[l, r]`?", which is a Fenwick tree range count. Sorting the queries is allowed because the problem is offline.
**Approach 1 - Brute force:** scan the range per query. Time O(n) per query.
**Approach 2 - Better:** sqrt decomposition where each block keeps a sorted copy; binary search in each full block. Time O(sqrt n log n) per query.
**Approach 3 - Optimal:** offline sort + Fenwick tree. Time O((n + q) log n). (The online version is a merge sort tree or persistent segment tree; just name them.)

Trace on `a = [5, 1, 4, 2, 3]`. Elements sorted by value: index 1 (1), 3 (2), 4 (3), 2 (4), 0 (5). Queries sorted by x: Qb = `[1, 3, 3]`, then Qa = `[0, 4, 5]`.

| processing | inserted positions | count in range |
|---|---|---|
| Qb, x = 3: insert values 1, 2, 3 -> positions 1, 3, 4 | {1, 3, 4} | positions in [1, 3]: 1 and 3 -> 2 |
| Qa, x = 5: also insert 4, 5 -> positions 2, 0 | {0, 1, 2, 3, 4} | positions in [0, 4]: 5 |

```js
function countLE(a, queries) {
  const n = a.length, bit = new Array(n + 1).fill(0);
  const upd = (i) => { for (i++; i <= n; i += i & -i) bit[i]++; };
  const pre = (i) => {                       // count of inserted positions in [0..i]
    let s = 0;
    for (i++; i > 0; i -= i & -i) s += bit[i];
    return s;
  };
  const idx = a.map((_, i) => i).sort((x, y) => a[x] - a[y]);
  const qs = queries.map((_, i) => i).sort((x, y) => queries[x][2] - queries[y][2]);
  const ans = new Array(queries.length);
  let p = 0;
  for (const qi of qs) {
    const [l, r, x] = queries[qi];
    while (p < n && a[idx[p]] <= x) upd(idx[p++]);
    ans[qi] = pre(r) - (l ? pre(l - 1) : 0);
  }
  return ans;
}
// countLE([5, 1, 4, 2, 3], [[1, 3, 3], [0, 4, 5]]) -> [2, 5]
```

**Complexity:** Time O((n + q) log n), Space O(n + q).
**Watch out:** use `<=` when advancing the pointer (not `<`). The comparator `a - b` is needed because JavaScript's default sort compares as strings. Store answers by original query index. If queries must be answered online (one by one as they arrive), this does not work; use a merge sort tree.

### 9. Largest Rectangle in Histogram (reference: sparse table + divide) — Hard
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Given bar heights, return the area of the largest rectangle that fits inside the histogram. This is LeetCode "Largest Rectangle in Histogram". Example: `[2, 1, 5, 6, 2, 3]` -> `10` (the bars 5 and 6, width 2, height 5).
**Intuition:** Look at the **lowest bar** in a range. A rectangle using the whole range can be at most `lowest height * width`. Any better rectangle must avoid that lowest bar, so it lies fully on the left or fully on the right. Recurse on both sides. Finding the lowest bar of a range quickly is exactly a range-minimum-index query. The real interview answer is the monotonic stack (see [Monotonic Stack and Queue](../../01-core-linear/11-monotonic-stack-queue/README.md)); this version is a reference to show the RMQ link.
**Approach 1 - Brute force:** for each start, extend right and track the minimum. Time O(n^2), Space O(1).
**Approach 2 - Better:** divide and conquer with a sparse table of argmin. Time O(n log n) build + O(n) recursion calls with O(1) each.
**Approach 3 - Optimal:** monotonic stack, O(n) time, O(n) space. Use this in the interview.

Trace on `[2, 1, 5, 6, 2, 3]`: whole range `[0..5]`, lowest bar is index 1 (height 1): area 1 * 6 = 6. Left `[0..0]`: 2. Right `[2..5]`: lowest is index 4 (height 2): area 2 * 4 = 8. Its left `[2..3]`: lowest is index 2 (height 5): area 5 * 2 = 10. Best so far 10. The rest give smaller values. Answer 10.

```js
// uses buildSparse; table stores indices, compared by height
function largestRectangleArea(h) {
  const n = h.length;
  if (!n) return 0;
  const q = buildSparse(h.map((_, i) => i), (i, j) => (h[i] <= h[j] ? i : j));
  let best = 0;
  const st = [[0, n - 1]];                  // explicit stack: no deep recursion on sorted input
  while (st.length) {
    const [l, r] = st.pop();
    if (l > r) continue;
    const m = q(l, r);
    best = Math.max(best, h[m] * (r - l + 1));
    st.push([l, m - 1], [m + 1, r]);
  }
  return best;
}
// largestRectangleArea([2, 1, 5, 6, 2, 3]) -> 10, largestRectangleArea([2, 4]) -> 4
```

**Complexity:** Time O(n log n), Space O(n log n).
**Watch out:** each range creates two children and every bar is chosen as the minimum exactly once, so there are at most n useful ranges. Recursion would go n deep on a sorted array, so we use a manual stack. For ties either index is fine because the sub-ranges still cover everything.

## Common Mistakes
- **"A sparse table works for sum."** The overlap would count values twice. For sums use prefix sums (static) or Fenwick (updates). Only idempotent operations (min, max, gcd, and, or) get O(1).
- **"A sparse table supports updates."** One change affects many table entries. It is static: rebuild in O(n log n), or use a segment tree.
- **Using `Math.floor(Math.log2(len))`.** Floating point can be wrong at exact powers of two in some environments. Use `31 - Math.clz32(len)`.
- **Second block starting at the wrong place.** It is `r - 2^k + 1`, not `l + 2^k`. The two blocks must end exactly at `r`.
- **Building level k past the array end.** Only create entries for `i + 2^k <= n`, otherwise you read `undefined`.
- **Sqrt decomposition: forgetting the single-block case.** When `l` and `r` are in the same block, the left-partial loop must stop at `r`.
- **Mo's: wrong order of pointer moves**, or returning answers in sorted order instead of the input order.
- **Mo's: using it when updates exist.** Plain Mo's has no updates (there is an extension with time as a third dimension, but that is rarely asked).
- **Euler tour: forgetting to record the parent again after each child.** The tour must contain the path between any two nodes.
- **Choosing a heavy tool for a light problem.** Static sum queries need prefix sums, nothing more.

## Interview Tips
- Say the **constraints reasoning** out loud: "The array is static and the operation is min, which is idempotent, so a sparse table gives O(1) queries after O(n log n) preprocessing."
- If asked "why not sum?", answer in one line: "The two blocks overlap, and sum would count the overlap twice. Min, max and gcd do not care."
- Always compare: prefix sums (static, sum), sparse table (static, idempotent), Fenwick (updates, sum), segment tree (updates, any merge), sqrt (when the merge is awkward). The table in "How to Spot It" is the answer.
- Mention the memory cost: n log n. For n = 10^6 you want typed arrays or a different tool.
- For LCA, mention the alternatives (binary lifting, O(log n) per query and less memory) and that Euler + sparse table trades memory for O(1) queries.
- Follow-ups you may get: "what if there are updates?" (segment tree), "can you reduce the memory?" (block decomposition, the O(n) RMQ idea, or just a segment tree), "online or offline?" (decides Mo's and offline Fenwick).
- Edge cases to state: `l == r`, a single-element array, `n = 1` (only level 0 exists), gcd with zeros, negative numbers for max/min (do not initialise with 0).
- Do not volunteer Mo's unless the problem fits. If you do, say its complexity: O((n + q) sqrt n).

## Practice List
**Easy**
- Range Sum Query - Immutable
- Min Stack

**Medium**
- Range Sum Query - Mutable
- Range Sum Query 2D - Immutable
- Sliding Window Maximum
- Lowest Common Ancestor of a Binary Tree (compare with the Euler approach)
- Count of Smaller Numbers After Self
- Maximum of All Subarrays of Size K (TUF title)
- Range Minimum Query (SPOJ / CSES "Static Range Minimum Queries")

**Hard**
- Largest Rectangle in Histogram
- Count of Range Sum
- Maximal Rectangle
- Number of distinct values in a range (SPOJ DQUERY, Mo's)
- Range Sum Query 2D - Mutable
- The Skyline Problem
- Kth Ancestor of a Tree Node (compare with binary lifting)

## Key Takeaways
- A **sparse table** gives O(1) range queries after O(n log n) building, but only for a **static** array.
- It works only for **idempotent** operations (min, max, gcd, and, or) because the two blocks **overlap**.
- **Sum can not use the overlap trick**: use prefix sums for static sums and Fenwick for updates.
- **Sqrt decomposition** splits the array into blocks of size about sqrt(n): O(sqrt n) per query, easy to write, handy when a tree merge is awkward.
- **Mo's algorithm** answers offline queries by sorting them by block of `l`, then `r`, giving O((n + q) sqrt n) with simple add/remove steps.
- **LCA = range minimum of depth** over the Euler tour, so the sparse table gives O(1) LCA queries.
- **Pick the tool from the table**: static + idempotent, sparse table; updates, Fenwick or segment tree; offline and hard to merge, Mo's or sort + Fenwick.

---

[← Previous: Segment Tree and Fenwick](../02-segment-tree-and-fenwick/README.md) | [Next: Number Theory →](../04-number-theory/README.md) | [Back to Advanced Index](../README.md)
