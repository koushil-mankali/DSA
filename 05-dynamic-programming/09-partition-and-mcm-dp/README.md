# Partition and Matrix Chain (Interval) DP

> **Estimated Time:** 5 hr | **Difficulty:** Advanced
> **Prerequisites:** [01 - DP Intuition](../01-dp-intuition/README.md), [02 - One-D DP](../02-one-d-dp/README.md), [06 - String DP](../06-string-dp/README.md)

---

## What You'll Learn

- You can recognise the two partition shapes: **front-partition DP** (cut a prefix, `dp[i]`) and **interval DP** (`dp[i][j]`, try every split `k`).
- You can write the interval DP loop in the right order (by length, short to long) without getting stuck on "which cell is ready?".
- You can solve Matrix Chain Multiplication and print the bracket order, not only the cost.
- You can use the "think about the LAST thing" trick for Burst Balloons, Cut a Stick and Triangulation.
- You can solve 12 classic problems, including Catalan counting and Egg Dropping, and explain the binary-search speed-up.

## The Big Idea

Imagine a long rope that you must cut into pieces, and every cut costs you something that depends on the piece you are cutting. You do not know the best first cut. But you do know this: after the first cut you have two smaller ropes, and each smaller rope is the same problem again. So you try every possible first cut, solve both halves, and keep the cheapest total.

That is the whole chapter. A big range `[i..j]` is solved by splitting it at some point `k` into two smaller ranges, solving those, and paying a "joining cost". The state is a **range**, so the table is 2D: `dp[i][j]`.

There is a simpler cousin. Sometimes you only cut from the front: "the first `i` items are already done; where does the last piece start?" That is **front-partition DP**, with a 1D table `dp[i]`. It is the same idea with one end fixed at the start.

Why interviewers love it: it is hard to fake. You cannot guess it. You must define a range state, choose the order of filling, and handle the base case. The problems also look very different on the surface (matrices, balloons, sticks, strings, polygons), so spotting the shared pattern shows real skill.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "split the array/string into consecutive pieces, minimise/maximise something" | front-partition DP: `dp[i]` = best for first `i` items |
| "each piece has a cost/limit (length at most k, must be a palindrome)" | front-partition with an inner loop over the last piece |
| "best order to multiply / combine / burst / remove items in a range" | interval DP, `dp[i][j]`, split at `k` |
| "cost of an action depends on the neighbours that are still there" | think about the LAST action in the range, not the first |
| "number of different ways to put brackets / build trees" | interval DP counting (Catalan style) |
| "print two letters in a row, strings can be painted over" | interval DP on a string, match equal ends |
| "worst case over my choice, minimise" (minimax) | interval DP with `min over k of (cost + max(left, right))` |
| "n up to about 100 to 500, range [i..j]" | O(n^3) interval DP is expected |

Quick size check: `n` around 100 to 500 and a range state means O(n^3) is fine. `n` of 10^5 means this is not the pattern (or you need a special trick).

## Core Concepts / Template

### Shape 1: Front-partition DP (1D)

`dp[i]` = the best answer for the first `i` items. To fill it, guess where the **last piece** starts.

```js
// dp[0] = base (empty prefix). Answer is dp[n].
function frontPartition(arr, maxLen) {
  const n = arr.length;
  const dp = new Array(n + 1).fill(Infinity);
  dp[0] = 0;
  for (let i = 1; i <= n; i++) {
    for (let j = 0; j < i; j++) {            // last piece is arr[j..i-1]
      if (!isValidPiece(j, i - 1)) continue; // e.g. length <= k, or palindrome
      dp[i] = Math.min(dp[i], dp[j] + costOfPiece(j, i - 1));
    }
  }
  return dp[n];
}
```

Time is O(n * pieceChoices), usually O(n^2) or O(n * k).

### Shape 2: Interval DP (2D)

`dp[i][j]` = the best answer using only the items in range `[i..j]`. Try every split `k` inside the range.

```js
// Answer is dp[0][n-1].
function intervalDP(n) {
  const dp = Array.from({ length: n }, () => new Array(n).fill(0)); // length-1 base = 0
  for (let len = 2; len <= n; len++) {          // short ranges first
    for (let i = 0; i + len - 1 < n; i++) {
      const j = i + len - 1;
      dp[i][j] = Infinity;
      for (let k = i; k < j; k++) {             // split into [i..k] and [k+1..j]
        dp[i][j] = Math.min(dp[i][j], dp[i][k] + dp[k + 1][j] + joinCost(i, k, j));
      }
    }
  }
  return dp[0][n - 1];
}
```

Three loops, so time O(n^3), space O(n^2).

### Why loop by length?

`dp[i][j]` needs `dp[i][k]` and `dp[k+1][j]`. Both are strictly shorter ranges. If you loop `len = 2, 3, 4, ...`, every shorter range is already filled. (Another valid order: `i` from `n-1` down to 0, and `j` from `i` up to `n-1`. Both work; pick one and stay with it.)

```
dp table for n = 4 (only the upper triangle is used)

        j=0   j=1   j=2   j=3
i=0   [ base  len2  len3  ANS ]
i=1   [  -    base  len2  len3]
i=2   [  -     -    base  len2]
i=3   [  -     -     -    base]

Fill order: the diagonal (len 1), then the next diagonal (len 2), ... up to the corner (len n).
```

### The "last action" trick

Some problems (Burst Balloons, Cut a Stick, Triangulation) are hard if you think "what do I do FIRST?", because the first action changes the neighbours for everyone else. Flip it: pick the **last** action in the range. Then, before that last action, the left side and the right side cannot affect each other, so they become independent subproblems. The "joining cost" uses the fixed boundaries `i` and `j`.

### Step-by-step trace of the template

Take the mini problem: combine three numbers `[2, 3, 4]`, where combining a range costs the sum of the range, and the total cost is the sum of all combine costs. Find the cheapest order.

```
Base: dp[0][0]=dp[1][1]=dp[2][2]=0
len=2: dp[0][1] = 0+0 + (2+3) = 5
       dp[1][2] = 0+0 + (3+4) = 7
len=3: dp[0][2], try k=0: dp[0][0] + dp[1][2] + 9 = 0+7+9 = 16
                try k=1: dp[0][1] + dp[2][2] + 9 = 5+0+9 = 14   <- best
Answer = 14  (combine 2 and 3 first, then add 4)
```

---

## Problems (easy -> hard)

### 1. Unique Binary Search Trees — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Given `n`, count the structurally different BSTs that store the values `1..n`. Example: `n = 3` -> `5`.
**Intuition:** Pick a root `r`. Everything smaller (`r-1` values) goes left, everything bigger (`n-r` values) goes right. The left and right choices are independent, so multiply. Only the **number** of nodes matters, not the actual values.
**Approach 1 - Brute force:** build every tree recursively. The count grows like Catalan numbers (4^n / n^1.5), so this explodes quickly.
**Approach 2 - Better:** memoised recursion on `count(n)`. Same as below, top-down. Time O(n^2), Space O(n).
**Approach 3 - Optimal:** `dp[nodes] = sum over root of dp[root-1] * dp[nodes-root]`, with `dp[0] = 1` (the empty tree is one way). Time O(n^2) is fine for interviews. (A closed form exists: `C(n+1) = C(n) * 2(2n+1) / (n+2)`, which is O(n).)

```
n = 3:  root=1: dp[0]*dp[2] = 1*2 = 2
        root=2: dp[1]*dp[1] = 1*1 = 1
        root=3: dp[2]*dp[0] = 2*1 = 2
        dp[3] = 5
```

```js
function numTrees(n) {
  const dp = new Array(n + 1).fill(0);
  dp[0] = 1; // empty tree counts as one shape
  for (let nodes = 1; nodes <= n; nodes++)
    for (let root = 1; root <= nodes; root++)
      dp[nodes] += dp[root - 1] * dp[nodes - root];
  return dp[n];
}
```

**Complexity:** Time O(n^2), Space O(n).
**Watch out:** `dp[0]` must be 1, not 0, or every product becomes 0. The sequence 1, 1, 2, 5, 14, 42, ... is the **Catalan numbers**. The same numbers count balanced bracket strings, ways to triangulate a polygon, and many more. Mention that, it impresses.

### 2. Partition Array for Maximum Sum — Medium
**Companies:** Google, Amazon
**Problem:** Split the array into consecutive pieces of length at most `k`. Each piece is replaced by its maximum value repeated (piece length times). Return the largest possible total sum. Example: `arr = [1,15,7,9,2,5,10], k = 3` -> `84`.
**Intuition:** This is front-partition DP. Let `dp[i]` be the best sum for the first `i` items. The last piece has length 1 to `k`. While we grow the piece backwards, we keep its running maximum.
**Approach 1 - Brute force:** try every way to cut the array. Exponential.
**Approach 2 - Better:** recursion with memo on the start index. Same complexity as the optimal, just top-down.
**Approach 3 - Optimal:** `dp[i] = max over len in 1..k of dp[i-len] + max(last len items) * len`.

```
arr = [1,15,7,9,2,5,10], k = 3
dp = [0, 1, 30, 45, 54, 63, 72, 84]   (dp[0..7])
dp[2]=30                     (piece [1,15] -> 15*2)
dp[3]=45                     (piece [1,15,7] -> 15*3)
dp[4]=max(dp[3]+9, dp[2]+9*2, dp[1]+15*3)=max(54,48,46)=54
dp[7]=84   (best cut: [1,15,7] [9] [2,5,10] -> 45 + 9 + 30 = 84)
```

```js
function maxSumAfterPartitioning(arr, k) {
  const n = arr.length;
  const dp = new Array(n + 1).fill(0); // dp[i] = best for first i items
  for (let i = 1; i <= n; i++) {
    let mx = 0;
    for (let len = 1; len <= Math.min(k, i); len++) {
      mx = Math.max(mx, arr[i - len]); // running max of the last piece
      dp[i] = Math.max(dp[i], dp[i - len] + mx * len);
    }
  }
  return dp[n];
}
```

**Complexity:** Time O(n * k), Space O(n).
**Watch out:** the running max must be updated **before** you use it for the current `len`. Also `len` cannot exceed `i`.

### 3. Guess Number Higher or Lower II — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** I pick a number in `1..n`. You guess; if wrong you pay the guessed number and I tell you higher or lower. Find the minimum money that **guarantees** a win. Example: `n = 10` -> `16`.
**Intuition:** This is minimax. You choose the guess `k`. The adversary then picks the worse side (more expensive). So `cost(i, j) = min over k of ( k + max(cost(i, k-1), cost(k+1, j)) )`. An empty or single-number range costs 0.
**Approach 1 - Brute force:** plain recursion over ranges. Exponential because ranges repeat.
**Approach 2 - Better:** memoise `cost(i, j)`. Same as the optimal.
**Approach 3 - Optimal:** bottom-up interval DP by length.

```
n = 3:  range [1,3]
  guess 1: 1 + max(0, cost[2,3]=2) = 3
  guess 2: 2 + max(0, 0)           = 2   <- best
  guess 3: 3 + max(cost[1,2]=1, 0) = 4
  answer 2
```

```js
function getMoneyAmount(n) {
  const dp = Array.from({ length: n + 2 }, () => new Array(n + 2).fill(0));
  for (let len = 2; len <= n; len++) {
    for (let i = 1; i + len - 1 <= n; i++) {
      const j = i + len - 1;
      let best = Infinity;
      for (let k = i; k <= j; k++) {
        const left = k > i ? dp[i][k - 1] : 0;
        const right = k < j ? dp[k + 1][j] : 0;
        best = Math.min(best, k + Math.max(left, right)); // adversary picks the worse side
      }
      dp[i][j] = best;
    }
  }
  return dp[1][n];
}
```

**Complexity:** Time O(n^3), Space O(n^2).
**Watch out:** it is `min` of `max`, not a sum of both sides (you only walk into one side). Use 1-based values here, since the cost IS the number.

### 4. Minimum Score Triangulation of Polygon — Medium
**Companies:** Google, Amazon
**Problem:** A convex polygon has vertex values `values[0..n-1]`. Cut it into triangles using non-crossing diagonals. A triangle scores the product of its 3 vertex values. Minimise the total. Example: `[3,7,4,5]` -> `144`.
**Intuition:** Look at the edge between vertex `i` and vertex `j`. In any triangulation, that edge belongs to exactly one triangle, with a third vertex `k` between them. That triangle splits the polygon into two smaller polygons: `[i..k]` and `[k..j]`. Note both halves **share** vertex `k`.
**Approach 1 - Brute force:** enumerate all triangulations. The count is a Catalan number, so exponential.
**Approach 2 - Better:** memoised recursion on `(i, j)`.
**Approach 3 - Optimal:** `dp[i][j] = min over k in (i, j) of dp[i][k] + dp[k][j] + v[i]*v[k]*v[j]`. Ranges of length 2 (just an edge) cost 0.

```
values = [3,7,4,5]
dp[0][2] = 3*7*4 = 84     dp[1][3] = 7*4*5 = 140
dp[0][3]: k=1: dp[0][1] + dp[1][3] + 3*7*5 = 0+140+105 = 245
          k=2: dp[0][2] + dp[2][3] + 3*4*5 = 84+0+60   = 144  <- best
```

```js
function minScoreTriangulation(values) {
  const n = values.length;
  const dp = Array.from({ length: n }, () => new Array(n).fill(0));
  for (let len = 3; len <= n; len++) {
    for (let i = 0; i + len - 1 < n; i++) {
      const j = i + len - 1;
      dp[i][j] = Infinity;
      for (let k = i + 1; k < j; k++)
        dp[i][j] = Math.min(dp[i][j], dp[i][k] + dp[k][j] + values[i] * values[k] * values[j]);
    }
  }
  return dp[0][n - 1];
}
```

**Complexity:** Time O(n^3), Space O(n^2).
**Watch out:** the halves are `[i..k]` and `[k..j]`, **not** `[k+1..j]`, because the triangle uses vertex `k` for both sides. Also, this has the same shape as Matrix Chain Multiplication (see problem 6).

### 5. Minimum Cost Tree From Leaf Values — Medium
**Companies:** Google, Amazon
**Problem:** Given `arr` of leaf values (in order), build a binary tree where every non-leaf has exactly 2 children, and the leaves are `arr` in order. Each non-leaf value is (max leaf in its left subtree) * (max leaf in its right subtree). Minimise the sum of all non-leaf values. Example: `[6,2,4]` -> `32`.
**Intuition:** Split the leaves `[i..j]` at `k`. The left part has some max, the right part has some max. Cost is their product plus the best for each part. That is interval DP. There is also a slick greedy: the smallest leaf must be multiplied with one of its neighbours, and its smaller neighbour is the cheapest choice, so remove the smallest leaf repeatedly. A monotonic stack does that in O(n).
**Approach 1 - Brute force:** try every tree shape. Catalan many.
**Approach 2 - Better:** interval DP with precomputed range maximums, O(n^3).
**Approach 3 - Optimal:** monotonic decreasing stack. When the new value `x` is larger than the stack top `mid`, `mid` is a local minimum: remove it and pay `mid * min(neighbour on the stack, x)`.

```
arr = [6,2,4]     stack starts [inf]
x=6: push            [inf,6]
x=2: push            [inf,6,2]
x=4: top 2 <= 4: pop 2, pay 2 * min(6,4) = 8   stack [inf,6]; push 4 -> [inf,6,4]
end: pop 4, pay 4*6 = 24    -> total 32
```

```js
function mctFromLeafValues(arr) {
  const stack = [Infinity]; // sentinel so a neighbour always exists
  let res = 0;
  for (const x of arr) {
    while (stack[stack.length - 1] <= x) {
      const mid = stack.pop();
      res += mid * Math.min(stack[stack.length - 1], x);
    }
    stack.push(x);
  }
  while (stack.length > 2) res += stack.pop() * stack[stack.length - 1];
  return res;
}
```

**Complexity:** Time O(n), Space O(n). (The interval DP version is O(n^3) time, O(n^2) space. In an interview, give the DP first, then mention the greedy as the improvement.)
**Watch out:** use `<=` in the while loop so equal values are also removed. The greedy is correct but hard to prove on the spot; if you cannot justify it, present the DP.

### 6. Matrix Chain Multiplication (with bracket printing) — Hard
**Companies:** Amazon, Microsoft, Google, Adobe
**Problem:** You have `n` matrices. Matrix `A_i` has size `arr[i-1] x arr[i]`. Multiplying a `p x q` matrix by a `q x r` matrix costs `p*q*r` scalar multiplications. Matrix multiplication is associative, so the order of brackets changes only the cost. Find the minimum cost and the bracket order. Example: `arr = [10,30,5,60]` (3 matrices) -> cost `4500`, order `((A1A2)A3)`.
**Intuition:** The last multiplication splits the chain `A_i..A_j` at some `k` into `(A_i..A_k)(A_{k+1}..A_j)`. The left result is `arr[i-1] x arr[k]`, the right is `arr[k] x arr[j]`, so the last step costs `arr[i-1]*arr[k]*arr[j]`. Try every `k`.
**Approach 1 - Brute force:** try every bracketing. Catalan many.
**Approach 2 - Better:** memoised recursion `f(i, j)`.
**Approach 3 - Optimal:** bottom-up interval DP. Store the best `k` for every cell in a `split` table, then rebuild the brackets recursively.

```
arr = [10,30,5,60]  -> A1 = 10x30, A2 = 30x5, A3 = 5x60
dp[1][2] = 10*30*5  = 1500
dp[2][3] = 30*5*60  = 9000
dp[1][3]: k=1: dp[1][1] + dp[2][3] + 10*30*60 = 0+9000+18000 = 27000
          k=2: dp[1][2] + dp[3][3] + 10*5*60  = 1500+0+3000  = 4500  <- best, split[1][3]=2
Rebuild: build(1,3): split 2 -> ( build(1,2) , build(3,3) ) = ((A1 A2) A3)
```

```js
// matrix i is arr[i-1] x arr[i]; returns min cost and the bracket order
function matrixChain(arr) {
  const n = arr.length - 1;
  if (n <= 0) return { cost: 0, order: '' };
  const dp = Array.from({ length: n + 1 }, () => new Array(n + 1).fill(0));
  const split = Array.from({ length: n + 1 }, () => new Array(n + 1).fill(0));
  for (let len = 2; len <= n; len++) {
    for (let i = 1; i + len - 1 <= n; i++) {
      const j = i + len - 1;
      dp[i][j] = Infinity;
      for (let k = i; k < j; k++) {
        const c = dp[i][k] + dp[k + 1][j] + arr[i - 1] * arr[k] * arr[j];
        if (c < dp[i][j]) { dp[i][j] = c; split[i][j] = k; }
      }
    }
  }
  const build = (i, j) =>
    i === j ? 'A' + i : '(' + build(i, split[i][j]) + build(split[i][j] + 1, j) + ')';
  return { cost: dp[1][n], order: build(1, n) };
}
```

**Complexity:** Time O(n^3), Space O(n^2).
**Watch out:** `arr` has `n + 1` numbers for `n` matrices. Off-by-one here is the most common bug. Single matrix means cost 0. The cost does not multiply the matrices, it only counts operations. A faster O(n log n) algorithm (Hu-Shing) exists but is never asked in interviews.

### 7. Palindrome Partitioning II — Hard
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** Cut string `s` so that every piece is a palindrome. Return the minimum number of cuts. Example: `"aab"` -> `1` (`"aa" | "b"`).
**Intuition:** Front-partition DP. `dp[i]` = fewest pieces for the first `i` chars. The last piece is `s[j..i-1]`; if it is a palindrome, `dp[i] = dp[j] + 1`. To check "is `s[j..i-1]` a palindrome" in O(1), precompute a table with the expand rule: `pal[i][j] = s[i]==s[j] && pal[i+1][j-1]`.
**Approach 1 - Brute force:** try all cut sets (2^(n-1)) and check each. Exponential.
**Approach 2 - Better:** DP, but check palindromes by scanning each piece. Time O(n^3).
**Approach 3 - Optimal:** precompute `pal`, then the partition DP. Using `dp[0] = -1` stores "cuts" directly: pieces minus one.

```
s = "aab"   pal: "a","a","b" yes; "aa" yes; "ab" no; "aab" no
dp[0] = -1
dp[1]: last piece "a"            -> dp[0]+1 = 0
dp[2]: last "a" -> dp[1]+1 = 1;  last "aa" -> dp[0]+1 = 0   -> 0
dp[3]: last "b" -> dp[2]+1 = 1;  "ab" no; "aab" no          -> 1
Answer 1
```

```js
function minCut(s) {
  const n = s.length;
  const pal = Array.from({ length: n }, () => new Array(n).fill(false));
  for (let i = n - 1; i >= 0; i--)
    for (let j = i; j < n; j++)
      pal[i][j] = s[i] === s[j] && (j - i < 2 || pal[i + 1][j - 1]);
  const dp = new Array(n + 1).fill(Infinity); // dp[i] = min cuts for first i chars
  dp[0] = -1; // zero chars need "-1 cuts", so one piece gives 0 cuts
  for (let i = 1; i <= n; i++)
    for (let j = 0; j < i; j++)
      if (pal[j][i - 1]) dp[i] = Math.min(dp[i], dp[j] + 1);
  return dp[n];
}
```

**Complexity:** Time O(n^2), Space O(n^2) (O(n) for `dp`, O(n^2) for `pal`).
**Watch out:** do not confuse this with Palindrome Partitioning I (list all partitions, backtracking). Here you only count. The `dp[0] = -1` trick is easy to get wrong; if it feels odd, use `dp[i]` = pieces and subtract 1 at the end. `n = 0` is not a valid input on LeetCode, but the code returns `-1` for it, so guard if you must.

### 8. Minimum Cost to Cut a Stick — Hard
**Companies:** Google, Amazon
**Problem:** A stick has length `n`. You must cut at the positions in `cuts`, in any order you like. One cut costs the **current length** of the piece you cut. Minimise the total cost. Example: `n = 7, cuts = [1,3,4,5]` -> `16`.
**Intuition:** Add the two ends `0` and `n` to the sorted cuts. Now `dp[i][j]` = best cost to fully cut the stick between marks `i` and `j`. Choose the **first** cut `k` in that range (it costs the whole length `c[j] - c[i]`). After it, the two sides are independent. (It is the same as picking the last cut in the reverse view; both work.)
**Approach 1 - Brute force:** try every order of cuts: `m!`.
**Approach 2 - Better:** memoised recursion on `(i, j)`.
**Approach 3 - Optimal:** interval DP over the sorted marks.

```
n=7, cuts sorted + ends: c = [0,1,3,4,5,7]
dp[0][5] = (7-0) + min over k in 1..4 of dp[0][k] + dp[k][5]
  k=1 (cut at 1): 0 + 12 = 12  -> 19
  k=2 (cut at 3): 3 +  6 =  9  -> 16   <- best
  k=3 (cut at 4): 7 +  3 = 10  -> 17
  k=4 (cut at 5): 10 + 0 = 10  -> 17
Answer 16: cut at 3 (cost 7), then 1 (cost 3), then 4 (cost 4), then 5 (cost 2)
```

```js
function minCost(n, cuts) {
  const c = [0, ...[...cuts].sort((a, b) => a - b), n];
  const m = c.length;
  const dp = Array.from({ length: m }, () => new Array(m).fill(0));
  for (let len = 2; len < m; len++) {            // len = distance between marks
    for (let i = 0; i + len < m; i++) {
      const j = i + len;
      dp[i][j] = Infinity;
      for (let k = i + 1; k < j; k++)            // first cut in this piece
        dp[i][j] = Math.min(dp[i][j], dp[i][k] + dp[k][j] + c[j] - c[i]);
    }
  }
  return dp[0][m - 1];
}
```

**Complexity:** Time O(m^3), Space O(m^2), where `m = cuts.length + 2`.
**Watch out:** sort the cuts, and do not sort in place if the caller still needs the array (copy first). The cost is the length of the piece `c[j] - c[i]`, not the position of the cut. The stick length `n` can be up to 10^6 but that does not matter, since only the number of cuts drives the DP.

### 9. Burst Balloons — Hard
**Companies:** Google, Amazon, Meta, Microsoft
**Problem:** Balloons have values `nums`. Bursting balloon `i` gives `nums[i-1] * nums[i] * nums[i+1]` coins (the neighbours at that moment; outside the array counts as 1). Burst them all to get the maximum coins. Example: `[3,1,5,8]` -> `167`.
**Intuition:** If you think "which balloon do I burst first?", the neighbours keep changing and the subproblems are tangled. Flip it. In the open range `(i, j)`, pick the balloon `k` that is burst **last**. At that moment, everything else in `(i, j)` is already gone, so its neighbours are exactly `a[i]` and `a[j]`. The left side `(i, k)` and right side `(k, j)` are now independent.
**Approach 1 - Brute force:** try every burst order: `n!`.
**Approach 2 - Better:** memoised recursion on `(i, j)`.
**Approach 3 - Optimal:** pad the array with 1 on both ends, then `dp[i][j] = max over k in (i,j) of dp[i][k] + dp[k][j] + a[i]*a[k]*a[j]`. Here `dp[i][j]` is for balloons **strictly between** `i` and `j`.

```
nums = [3,1,5,8]  -> a = [1,3,1,5,8,1]
Full range (0,5). Try balloon 8 (index 4) as the LAST one:
  dp[0][4] + dp[4][5] + a[0]*a[4]*a[5] = 159 + 0 + 1*8*1 = 167   <- best
  (dp[0][4] = 159: balloons 3,1,5 with 3 last: dp[1][4] + 1*3*8 = 135 + 24)
  (dp[1][4] = 135: balloons 1,5 with 5 last: dp[1][3] + 3*5*8 = 15 + 120)
Real order: burst 1, then 5, then 3, then 8
  1: 3*1*5 = 15,  5: 3*5*8 = 120,  3: 1*3*8 = 24,  8: 1*8*1 = 8   -> 167
```

```js
function maxCoins(nums) {
  const a = [1, ...nums, 1];
  const n = a.length;
  const dp = Array.from({ length: n }, () => new Array(n).fill(0));
  for (let len = 2; len < n; len++) {
    for (let i = 0; i + len < n; i++) {
      const j = i + len;
      for (let k = i + 1; k < j; k++) // k = the LAST balloon burst in (i, j)
        dp[i][j] = Math.max(dp[i][j], dp[i][k] + dp[k][j] + a[i] * a[k] * a[j]);
    }
  }
  return dp[0][n - 1];
}
```

**Complexity:** Time O(n^3), Space O(n^2).
**Watch out:** the boundaries `i` and `j` are **not** burst inside this state; they are the surviving neighbours. Start `len` at 2 (a range with at least one balloon between). Empty `nums` returns 0 with this code. Zero-value balloons can be removed first as an optimisation, but it is not required.

### 10. Boolean Parenthesization (Count Ways to Evaluate to True) — Hard
**Companies:** Amazon, Microsoft, Google
**Problem:** Given an expression like `T|F&T^T` with operators `&`, `|`, `^`, count how many ways to add brackets so that it evaluates to **true**. Example: `T^F|F` -> `2` (`(T^F)|F` and `T^(F|F)`). The result can be big; many versions ask for it modulo 1,000,000,007.
**Intuition:** The **last operator evaluated** splits the expression into left and right sub-expressions. For each split we need to know, for left and right, how many ways make them true and how many make them false. Then combine using the truth table of the operator.
**Approach 1 - Brute force:** recursion over every operator, no memo. Exponential.
**Approach 2 - Better:** memoised recursion on `(i, j, wantTrue)`.
**Approach 3 - Optimal:** two tables `T[i][j]` and `F[i][j]`, filled by length. For operator `op` with left `(lt, lf)` and right `(rt, rf)`: `total = (lt+lf)*(rt+rf)`.

| op | ways to be true | 
|---|---|
| `&` | `lt * rt` |
| `\|` | `total - lf * rf` |
| `^` | `lt * rf + lf * rt` |

False ways are `total - trueWays`.

```
T^F|F  symbols [T,F,F], ops [^,|]
len 2: [T^F]: true 1, false 0.   [F|F]: true 0, false 1
len 3: split at ^ : left=T (1,0), right=[F|F] (0,1): ^ true = 1*1 + 0*0 = 1
       split at | : left=[T^F] (1,0), right=F (0,1): | true = total(1*1) - lf*rf(0*1) = 1
       T = 2
```

```js
function countTrue(expr) {
  const MOD = 1000000007n; // BigInt: products of two ~1e9 numbers exceed 2^53
  const syms = [], ops = [];
  for (let i = 0; i < expr.length; i++) (i % 2 ? ops : syms).push(expr[i]);
  const n = syms.length;
  const T = Array.from({ length: n }, () => new Array(n).fill(0n));
  const F = Array.from({ length: n }, () => new Array(n).fill(0n));
  for (let i = 0; i < n; i++) {
    T[i][i] = syms[i] === 'T' ? 1n : 0n;
    F[i][i] = 1n - T[i][i];
  }
  for (let len = 2; len <= n; len++) {
    for (let i = 0; i + len - 1 < n; i++) {
      const j = i + len - 1;
      let t = 0n, f = 0n;
      for (let k = i; k < j; k++) { // ops[k] sits between syms[k] and syms[k+1]
        const lt = T[i][k], lf = F[i][k], rt = T[k + 1][j], rf = F[k + 1][j];
        const total = (lt + lf) * (rt + rf);
        let tr;
        if (ops[k] === '&') tr = lt * rt;
        else if (ops[k] === '|') tr = total - lf * rf;
        else tr = lt * rf + lf * rt;
        t += tr; f += total - tr;
      }
      T[i][j] = t % MOD; F[i][j] = f % MOD;
    }
  }
  return Number(T[0][n - 1]);
}
```

**Complexity:** Time O(n^3), Space O(n^2), with `n` the number of symbols.
**Watch out:** in JavaScript, multiplying two numbers near 10^9 breaks past 2^53 and silently gives wrong answers, so use `BigInt` (or a safe multiply) when you take a modulus. Keep `T` and `F` both: "false" counts are needed to compute "true" for `|` and `^`. A negative modulo result is also a risk; with `BigInt` and only additions/subtractions that stay positive here, it is safe.

### 11. Strange Printer — Hard
**Companies:** Google, Amazon
**Problem:** A printer prints a run of one repeated character per turn, and each new print can overwrite earlier ones. Find the minimum turns to print string `s`. Example: `"aba"` -> `2` (print `aaa`, then `b` in the middle).
**Intuition:** First squash runs of equal letters (`"aaabbb"` becomes `"ab"`); they never cost extra. Now look at the last char `s[j]`. Either it gets its own print (`dp[i][j-1] + 1`), or it was printed together with an earlier equal char `s[k]`: one long stroke covers both, and the part between them `(k+1..j-1)` is painted on top afterwards. That costs `dp[i][k] + dp[k+1][j-1]`.
**Approach 1 - Brute force:** try all print sequences. Exponential.
**Approach 2 - Better:** memoised recursion on `(i, j)`, same transitions.
**Approach 3 - Optimal:** interval DP, `i` from the end down, `j` from `i` up.

```
s = "aba" (no runs to squash)
dp[0][0]=dp[1][1]=dp[2][2]=1
dp[0][1] ("ab") = dp[0][0] + 1 = 2
dp[1][2] ("ba") = 2
dp[0][2]: own print: dp[0][1]+1 = 3
          k=0 (s[0]=='a'==s[2]): dp[0][0] + dp[1][1] = 1 + 1 = 2   <- best
Answer 2
```

```js
function strangePrinter(s) {
  const t = s.replace(/(.)\1+/g, '$1'); // a run of equal letters costs one print
  const n = t.length;
  if (n === 0) return 0;
  const dp = Array.from({ length: n }, () => new Array(n).fill(0));
  for (let i = n - 1; i >= 0; i--) {
    dp[i][i] = 1;
    for (let j = i + 1; j < n; j++) {
      dp[i][j] = dp[i][j - 1] + 1; // t[j] gets its own print
      for (let k = i; k < j; k++) {
        if (t[k] === t[j]) {
          const mid = k + 1 <= j - 1 ? dp[k + 1][j - 1] : 0;
          dp[i][j] = Math.min(dp[i][j], dp[i][k] + mid); // t[j] rides along with t[k]
        }
      }
    }
  }
  return dp[0][n - 1];
}
```

**Complexity:** Time O(n^3), Space O(n^2).
**Watch out:** when `k + 1 > j - 1` (neighbours), the middle part is empty and costs 0; reading `dp[k+1][j-1]` there would use a wrong cell. Squashing runs is an optimisation, but it also makes the `+1` own-print case consistent.

### 12. Egg Dropping Puzzle (Super Egg Drop) — Hard
**Companies:** Google, Amazon, Microsoft, Adobe
**Problem:** You have `e` identical eggs and a building with `f` floors. There is a critical floor: eggs dropped from above it break, from at or below it survive. A surviving egg can be reused; a broken egg cannot. Find the minimum number of drops needed to **guarantee** you find the critical floor. Example: `e = 2, f = 6` -> `3`.
**Intuition:** Drop an egg from floor `x`. If it breaks, you have `e-1` eggs and the `x-1` floors below. If it survives, you have `e` eggs and the `f-x` floors above. The adversary picks the worse case, you pick the best `x`: `dp[e][f] = 1 + min over x of max(dp[e-1][x-1], dp[e][f-x])`. This is minimax again.
**Approach 1 - Brute force:** plain recursion on this formula. Exponential.
**Approach 2 - Better:** the 2D DP above with a loop over `x`. Time O(e * f^2). `dp[e-1][x-1]` grows with `x`, and `dp[e][f-x]` shrinks with `x`, so the best `x` is where they cross. That means you can binary-search `x`, giving O(e * f * log f).
**Approach 3 - Optimal:** flip the question. Let `dp[m][e]` = the maximum number of floors you can check with `m` drops and `e` eggs. One drop: if the egg breaks you still cover `dp[m-1][e-1]` floors below; if it survives, `dp[m-1][e]` floors above; plus the floor you dropped from. So `dp[m][e] = dp[m-1][e-1] + dp[m-1][e] + 1`. Increase `m` until `dp[m][e] >= f`. Time O(e * answer), and the answer is at most `f`; for many eggs it is around log2(f).

```
e = 2, f = 6.  Keep a row over eggs, start [0,0,0] (index = eggs used).
m=1: [0, 1, 1]       (1 drop checks 1 floor)
m=2: [0, 2, 3]       (e=2: 1 + 1 + 1 = 3 floors)
m=3: [0, 3, 6]       (e=2: 3 + 2 + 1 = 6)  >= 6 floors  -> answer 3
```

```js
// dp[e] = max floors checkable with the current number of moves and e eggs
function superEggDrop(eggs, floors) {
  const dp = new Array(eggs + 1).fill(0);
  let moves = 0;
  while (dp[eggs] < floors) {
    moves++;
    // go from high e to low e so dp[e-1] is still from the previous move
    for (let e = eggs; e >= 1; e--) dp[e] = dp[e] + dp[e - 1] + 1;
  }
  return moves;
}
```

**Complexity:** Time O(eggs * moves), Space O(eggs). With `floors = 0` it returns 0.
**Watch out:** the inner loop must go from large `e` down to small, or you overwrite `dp[e-1]` before using its old value. With 1 egg the answer is `f` (linear scan from the bottom). With enough eggs it becomes binary search, log2(f) drops. Do not claim the O(e f^2) version passes the big LeetCode limits (it does not); say you would use the binary search on `x` or the moves-based version.

---

## Common Mistakes

- **"I will think about the first action."** For balloons, sticks and triangles the first action changes everyone's neighbours. Choose the last action in the range instead, so the two sides are independent.
- **Wrong loop order.** Looping `i` from 0 and `j` from `i` upward reads cells that are not filled yet. Loop by increasing length, or `i` descending with `j` ascending.
- **Mixing up the split `[i..k] + [k+1..j]` with `[i..k] + [k..j]`.** Chain multiplication and string cuts use `k` and `k+1`. Triangulation and the "marks" in Cut a Stick share the middle point `k`.
- **Base case off by one.** Single item costs 0 for most interval problems, 1 for Strange Printer, and `dp[0] = 1` for Catalan counting. Think about what an empty or single range means each time.
- **Summing both sides in minimax problems.** In Guess Number II and Egg Drop, the adversary picks one side. Use `max` of the two sides, not `+`.
- **Using plain numbers for big modulo products in JavaScript.** `a * b % MOD` is wrong once `a * b` passes 2^53. Use `BigInt`.
- **"Partition DP always needs a 2D table."** If one end is fixed at the start (cut a prefix), a 1D `dp[i]` is enough.

## Interview Tips

- Say the state out loud first: "`dp[i][j]` is the best answer for the items from `i` to `j`." Then say the transition and the fill order.
- Name the shape: "this is interval DP, O(n^3)". Interviewers like to hear that you expect three nested loops, and why (n^2 states, n splits each).
- For "burst / cut / triangulate", explain the last-action trick in one sentence. This is the part most candidates miss.
- Mention the link to Catalan numbers: the number of bracketings of `n` matrices, triangulations of an `(n+2)`-gon, and BSTs with `n` nodes are the same sequence. This is why brute force is hopeless.
- For Matrix Chain, offer to print the order. Keep a `split` table and rebuild recursively. It is a common follow-up.
- Egg Dropping follow-ups: "can you do better than O(e f^2)?" Answer: binary search on the drop floor (monotone cross point), or flip the DP to "max floors for `m` moves".
- Edge cases to mention: empty input, one item, all items equal, very small `k`, `k` larger than `n`, a single matrix, and the numeric overflow of results (use BigInt or a modulus if asked).
- Space: you can usually not shrink an interval DP to 1D (every cell reads from many others). Say so honestly. Front-partition DP, however, is naturally O(n).

## Practice List

**Easy / warm-up**
- Climbing Stairs
- Min Cost Climbing Stairs
- Palindromic Substrings
- Longest Palindromic Substring

**Medium**
- Unique Binary Search Trees II
- Partition Array for Maximum Sum
- Word Break
- Palindrome Partitioning
- Minimum Score Triangulation of Polygon
- Minimum Cost Tree From Leaf Values
- Guess Number Higher or Lower II
- Different Ways to Add Parentheses
- Remove Boxes
- Stone Game (interval version)
- Minimum Difficulty of a Job Schedule
- Split Array Largest Sum

**Hard**
- Burst Balloons
- Minimum Cost to Cut a Stick
- Palindrome Partitioning II
- Palindrome Partitioning III
- Strange Printer
- Super Egg Drop
- Matrix Chain Multiplication (GeeksforGeeks)
- Boolean Parenthesization (GeeksforGeeks)
- Count Different Palindromic Subsequences
- Minimum Cost to Merge Stones (the Merge Stones problem)

## Key Takeaways

- **Interval DP** means the state is a range `dp[i][j]`; split at every `k` and combine the two smaller ranges.
- **Fill by increasing length** so every smaller range is ready before you need it.
- **Think about the last action** (last balloon, last cut, last multiplication) so the two sides become independent.
- **Front-partition DP** (`dp[i]`, last piece length) is the 1D version; use it when pieces are cut from the front with a validity rule.
- **Time is usually O(n^3), space O(n^2)**; accept that and give the size check ("n up to a few hundred").
- **Minimax problems** (Guess Number II, Egg Drop) take `min` over your choice and `max` over the adversary's two outcomes.
- **Catalan numbers** count brackets, triangulations and BSTs; Egg Drop speeds up with binary search or by flipping the DP to "max floors for `m` moves".

---

[← Previous: Longest Increasing Subsequence](../08-longest-increasing-subsequence/README.md) | [Next: Bitmask DP →](../10-bitmask-dp/README.md) | [Back to Dynamic Programming Index](../README.md)
