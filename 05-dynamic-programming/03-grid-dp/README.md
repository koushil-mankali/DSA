# Grid DP

> **Estimated Time:** 6 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - DP Intuition](../01-dp-intuition/README.md), [02 - One-Dimensional DP](../02-one-d-dp/README.md)

---

## What You'll Learn
- You can define `dp[i][j]` clearly in one sentence and write the recurrence from it.
- You can tell fixed start/end problems from variable start/end problems, and pick the right direction (top-down or bottom-up on the grid).
- You can shrink a 2D table to one row (or two rows) when each cell only looks at the row above.
- You can add an extra state (last choice, second walker, move count) when a plain `dp[i][j]` is not enough.
- You can choose between memoized DFS, tabulation, and backtracking for grid problems.

## The Big Idea

Think about walking in a city made of square blocks. You can only walk right or down. To know how many ways reach a corner, you do not need to remember every route. You only need to know how many ways reach the block above it and the block to its left. Every route to the corner must come through one of those two blocks. So the answer for a block is built from the answers of its neighbours.

That is grid DP. Each cell stores the answer to a small question ("best cost to get here", "number of ways to get here", "biggest square ending here"). The answer for a cell comes from a few cells that were already solved. You fill the grid in an order that guarantees those neighbours are ready.

Interviewers love grid DP because it is the first place DP becomes visual. A weak candidate tries to enumerate paths. A strong candidate says what `dp[i][j]` means, writes the transition, names the base cases, and then trims the space. It also gives easy follow-ups (obstacles, a second walker, a different move set) that show whether you understood the idea or only memorised one answer.

### Fixed vs variable start and end

This is the main way to organise the whole chapter.

| Kind | Example | How to define `dp[i][j]` |
|---|---|---|
| Fixed start, fixed end | Unique Paths, Min Path Sum | Best answer for reaching `(i, j)` from the start. Answer is the end cell. |
| Fixed start, variable end | Maximal Square, Longest Increasing Path | Best answer for something that *ends* at `(i, j)`. Answer is max over all cells. |
| Variable start, fixed end | Triangle (bottom-up), Min Falling Path (reversed) | Best answer for going from `(i, j)` to the end. Answer is at the start. |
| Variable start, variable end | Falling Path Sum, Gold Mine | First row (or any cell) can be start, last row (or any cell) can be end. Answer is min/max over a whole row or grid. |
| Reverse direction needed | Dungeon Game | Forward DP fails because the state needed depends on the future. Fill from the end backwards. |

Rule of thumb: if "what I need now" depends on "what comes later", fill from the end. If "what I have now" depends on "what came before", fill from the start.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "Grid, move only right/down, count paths" | `dp[i][j] = dp[i-1][j] + dp[i][j-1]` |
| "Grid, minimum/maximum sum path" | Same shape, replace `+` with `min`/`max` and add the cell value |
| "Triangle" or "each row has one more element" | Bottom-up DP, one array, reuse the row below |
| "Pick one per row, neighbours must be adjacent column" | Falling path: look at 3 cells above |
| "Cannot pick the same thing two days in a row" | Add `last choice` to the state |
| "Largest square / rectangle of 1s" | `dp[i][j]` = side of square whose bottom-right corner is `(i, j)` |
| "Going there and coming back" or "two people at once" | Move both walkers together, state is `(step, pos1, pos2)` |
| "Health/energy must never drop below 1" | Fill from the end; forward DP will fail |
| "Moves in 4 directions, strictly increasing values" | Memoized DFS (cycles are impossible, so no visited array needed) |
| "Count ways to leave the grid in at most k moves" | DP over `(moves, i, j)` |
| "Visit each cell at most once, any start, any end, small grid" | Backtracking DFS, not DP |

## Core Concepts / Template

Five steps, every time:
1. **State:** write `dp[i][j] = ...` as a full sentence. If you cannot finish it, you do not have a DP yet.
2. **Transition:** where can I come *from* (pull) or go *to* (push)? Pull is easier to get right.
3. **Base cells:** first row, first column, blocked cells. Bugs hide here.
4. **Order:** row by row if every dependency is above or left; loop backwards if dependencies are below or right.
5. **Answer location:** the last cell, a max over all cells, or a min over the last row?

### Reusable template (pull style, fixed start and end)

```js
function gridDP(grid) {
  const m = grid.length, n = grid[0].length;
  const dp = Array.from({ length: m }, () => new Array(n).fill(0));
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      if (i === 0 && j === 0) { dp[i][j] = BASE; continue; }
      const fromUp = i > 0 ? dp[i - 1][j] : IDENTITY;   // IDENTITY: 0 for sum, Infinity for min
      const fromLeft = j > 0 ? dp[i][j - 1] : IDENTITY;
      dp[i][j] = COMBINE(fromUp, fromLeft) + grid[i][j];
    }
  }
  return dp[m - 1][n - 1];
}
```

### Space trick: one row is enough
If `dp[i][j]` uses only `dp[i-1][j]` (up) and `dp[i][j-1]` (left), keep one array `dp` of length `n`. Before you overwrite `dp[j]`, it still holds the value from the row above. And `dp[j-1]` is already the new value of this row. So `dp[j] = dp[j] + dp[j-1]` means "up + left". If you also need the diagonal (up-left), save it in a variable before overwriting.

### Trace on a small grid
Count right/down paths in a 3x3 grid. Each cell = ways to reach it.

```
1 1 1
1 2 3
1 3 6
```

Row 0 and column 0 are all 1 (only one straight way). The cell `(1,1)` is up (1) + left (1) = 2. The cell `(2,2)` is up (3) + left (3) = 6.

### Memo DFS vs tabulation on grids
- **Tabulation** is best when the order is obvious (right/down moves).
- **Memoized DFS** is best when the order is not obvious (4-direction moves with a monotone rule, like Longest Increasing Path). The rule "next cell must be bigger" makes the dependency graph acyclic, so memo is safe without a visited set.
- **Backtracking** is needed when the same cell must not be reused *within one path* and the state cannot be summarised (Gold Mine). It is exponential, which is fine only when the grid is small.

## Problems (easy -> hard)

### 1. Unique Paths — Medium
**Companies:** Amazon, Google, Microsoft, Bloomberg
**Problem:** A robot starts at the top-left of an `m x n` grid and can move only right or down. Count the ways to reach the bottom-right. Example: `m=3, n=7` -> `28`.
**Intuition:** To arrive at a cell you must come from the cell above or the cell on the left. So ways(cell) = ways(up) + ways(left).
**Approach 1 - Brute force:** Recursion `f(i, j) = f(i-1, j) + f(i, j-1)`. Time O(2^(m+n)), Space O(m+n) stack.
**Approach 2 - Better:** Memoize the recursion (or a full 2D table). Time O(m*n), Space O(m*n).
**Approach 3 - Optimal:** Keep one row. Start with all 1s (first row). For each next row, `dp[j] += dp[j-1]`.

Trace for `m=3, n=3` (row after each pass): `[1,1,1]` -> `[1,2,3]` -> `[1,3,6]`. Answer `6`.

There is also a maths answer, `C(m+n-2, m-1)`. Mention it, but the DP is what interviewers expect first.

```js
function uniquePaths(m, n) {
  const dp = new Array(n).fill(1);
  for (let i = 1; i < m; i++)
    for (let j = 1; j < n; j++) dp[j] += dp[j - 1];
  return dp[n - 1];
}
```

**Complexity:** Time O(m*n), Space O(n).
**Watch out:** Start loops at 1, not 0, because row 0 and column 0 are already 1. For the combinatorics version, watch for overflow in factorials.

### 2. Unique Paths II (Obstacles) — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Same as above, but cells with `1` are blocked. Example: `[[0,0,0],[0,1,0],[0,0,0]]` -> `2`.
**Intuition:** A blocked cell has zero ways to be reached. That zero then flows naturally into its neighbours through the same sum.
**Approach 1 - Brute force:** Recursion that returns 0 on a blocked cell. Time O(2^(m+n)).
**Approach 2 - Better:** 2D table. Time O(m*n), Space O(m*n).
**Approach 3 - Optimal:** One row. For each cell: if blocked, `dp[j] = 0`; otherwise add the left neighbour `dp[j-1]` (the up value is already in `dp[j]`).

Trace on the example (row after each pass):
```
row 0: [1,1,1]
row 1: [1,0,1]   (middle is blocked -> 0; last = up 1 + left 0 = 1)
row 2: [1,1,2]
```
Answer `2`.

```js
function uniquePathsWithObstacles(g) {
  const m = g.length, n = g[0].length;
  const dp = new Array(n).fill(0);
  dp[0] = g[0][0] === 1 ? 0 : 1;
  for (let i = 0; i < m; i++)
    for (let j = 0; j < n; j++) {
      if (g[i][j] === 1) dp[j] = 0;
      else if (j > 0) dp[j] += dp[j - 1];
    }
  return dp[n - 1];
}
```

**Complexity:** Time O(m*n), Space O(n).
**Watch out:** If the start or the end is blocked, the answer is 0. Do not set the whole first row to 1; a blocker in row 0 must cut off everything to its right (the loop above handles this because `dp[j]` becomes 0 and then feeds the next cell).

### 3. Minimum Path Sum — Medium
**Companies:** Amazon, Google, Microsoft, Goldman Sachs
**Problem:** Move right/down from top-left to bottom-right and minimise the sum of cells on the path. Example: `[[1,3,1],[1,5,1],[4,2,1]]` -> `7` (path 1-3-1-1-1).
**Intuition:** The best cost to reach a cell is its own value plus the cheaper of "from above" and "from left".
**Approach 1 - Brute force:** Try all paths. Time O(2^(m+n)).
**Approach 2 - Better:** Memoized recursion or 2D table. Time O(m*n), Space O(m*n).
**Approach 3 - Optimal:** One row of costs. Initialise with `Infinity`, except `dp[0] = 0`, so the first cell naturally picks "cost 0 before me".

Trace (row after each pass):
```
row 0: [1, 4, 5]
row 1: [2, 7, 6]
row 2: [6, 8, 7]
```
Answer `7`.

```js
function minPathSum(g) {
  const m = g.length, n = g[0].length;
  const dp = new Array(n).fill(Infinity);
  dp[0] = 0;
  for (let i = 0; i < m; i++)
    for (let j = 0; j < n; j++) {
      if (j > 0) dp[j] = Math.min(dp[j], dp[j - 1]); // up vs left
      dp[j] += g[i][j];
    }
  return dp[n - 1];
}
```

**Complexity:** Time O(m*n), Space O(n).
**Watch out:** Greedy ("always step to the smaller neighbour") is wrong; a cheap step can lead into an expensive region. Also this version modifies nothing, so the input stays intact (the in-place version is faster in memory but destroys the input; say that trade-off).

### 4. Triangle — Medium
**Companies:** Amazon, Google, Microsoft, Bloomberg
**Problem:** From the top of a triangle, move to an adjacent number on the row below (same index or index+1). Find the minimum path sum to the bottom. Example: `[[2],[3,4],[6,5,7],[4,1,8,3]]` -> `11` (2+3+5+1).
**Intuition:** The start is fixed (the top) but the end is anywhere in the bottom row. Go **bottom-up**: then the end is "free" (all bottom cells start as themselves) and the start is the single top cell, which is the answer.
**Approach 1 - Brute force:** DFS trying both children. Time O(2^n).
**Approach 2 - Better:** Top-down memo `f(i, j)`, or a top-down table and then min of the last row. Time O(n^2), Space O(n^2).
**Approach 3 - Optimal:** Copy the last row. For each row above, going up, `dp[j] = t[i][j] + min(dp[j], dp[j+1])`. After the top row, `dp[0]` is the answer.

Trace (n = 4 rows):
```
start      [4, 1, 8, 3]
row [6,5,7] -> [6+min(4,1), 5+min(1,8), 7+min(8,3)] = [7, 6, 10]
row [3,4]   -> [3+min(7,6), 4+min(6,10)]             = [9, 10]
row [2]     -> [2+min(9,10)]                         = [11]
```

```js
function minimumTotal(t) {
  const dp = t[t.length - 1].slice(); // copy: do not destroy the input
  for (let i = t.length - 2; i >= 0; i--)
    for (let j = 0; j <= i; j++)
      dp[j] = t[i][j] + Math.min(dp[j], dp[j + 1]);
  return dp[0];
}
```

**Complexity:** Time O(n^2) (n rows, about n^2/2 cells), Space O(n).
**Watch out:** Row `i` has `i+1` cells, so loop `j <= i`. Going top-down forces you to handle the edges (`j-1` out of range) and take a min over the last row; bottom-up avoids all that.

### 5. Minimum Falling Path Sum — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** In an `n x n` matrix, pick one number per row; each pick must be in the same column or an adjacent column of the previous pick. Minimise the sum. Example: `[[2,1,3],[6,5,4],[7,8,9]]` -> `13`.
**Intuition:** Variable start (any cell of row 0) and variable end (any cell of the last row). Let `dp[i][j]` be the cheapest falling path that ends at `(i, j)`. It comes from up-left, up, or up-right. The answer is the min of the last row.
**Approach 1 - Brute force:** Three choices per row. Time O(3^n).
**Approach 2 - Better:** 2D table. Time O(n^2), Space O(n^2).
**Approach 3 - Optimal:** Keep only the previous row.

Trace:
```
row 0: [2, 1, 3]
row 1: [6+min(2,1), 5+min(2,1,3), 4+min(1,3)] = [7, 6, 5]
row 2: [7+min(7,6), 8+min(7,6,5), 9+min(6,5)] = [13, 13, 14]
answer = min = 13
```

```js
function minFallingPathSum(a) {
  const n = a.length;
  let prev = a[0].slice();
  for (let i = 1; i < n; i++) {
    const cur = new Array(n);
    for (let j = 0; j < n; j++) {
      let best = prev[j];
      if (j > 0) best = Math.min(best, prev[j - 1]);
      if (j < n - 1) best = Math.min(best, prev[j + 1]);
      cur[j] = a[i][j] + best;
    }
    prev = cur;
  }
  return Math.min(...prev);
}
```

**Complexity:** Time O(n^2), Space O(n).
**Watch out:** Values can be negative, so do not start `best` at 0. Always include the straight-up neighbour first (it always exists). With a 1x1 matrix the answer is the single value.

### 6. Ninja Training (DP with last-choice state) — Medium
**Companies:** Amazon, Flipkart, Microsoft
**Problem:** For `n` days you can do one of 3 activities each day, with a point value per day and activity. You may not do the same activity on two consecutive days. Maximise total points. Example: `[[1,2,5],[3,1,1],[3,3,3]]` -> `11`.
**Intuition:** This is a grid of `n x 3`, but the rule is about the *previous column choice*, not adjacency. So the state is "day and the activity I did today". Today's best for activity `k` = points today + best of yesterday over the *other* two activities.
**Approach 1 - Brute force:** Recursion over days with the last activity. Time O(3^n).
**Approach 2 - Better:** Memo on `(day, last)`. Time O(n * 3 * 3), Space O(n * 3).
**Approach 3 - Optimal:** Keep only yesterday's three values.

Trace:
```
day 0: [1, 2, 5]
day 1: k0 = 3 + max(2,5) = 8;  k1 = 1 + max(1,5) = 6;  k2 = 1 + max(1,2) = 3  -> [8, 6, 3]
day 2: k0 = 3 + max(6,3) = 9;  k1 = 3 + max(8,3) = 11; k2 = 3 + max(8,6) = 11 -> [9, 11, 11]
answer = 11
```

```js
function ninjaTraining(pts) {
  let dp = pts[0].slice(); // dp[k] = best total if today's task is k
  for (let i = 1; i < pts.length; i++) {
    const cur = [0, 0, 0];
    for (let k = 0; k < 3; k++) {
      let best = 0;
      for (let p = 0; p < 3; p++) if (p !== k) best = Math.max(best, dp[p]);
      cur[k] = pts[i][k] + best;
    }
    dp = cur;
  }
  return Math.max(...dp);
}
```

**Complexity:** Time O(n * 3 * 3) = O(n), Space O(1).
**Watch out:** Do not say "just skip the last used activity and take the max"; greedy fails. Day 0 has no restriction, so it is simply the first row. This "last choice" pattern is the same idea as House Painting / Paint House.

### 7. Maximal Square — Medium
**Companies:** Amazon, Google, Microsoft, Apple, Facebook
**Problem:** Given a binary matrix of `'0'` and `'1'`, find the area of the largest square made only of `'1'`. Example: the matrix below -> `4`.
```
1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
```
**Intuition:** Let `dp[i][j]` = side of the biggest all-1 square whose **bottom-right corner** is `(i, j)`. That square is limited by its three neighbours: up, left, up-left. If all three can support side `s`, this cell supports side `s+1`. So `dp[i][j] = 1 + min(up, left, diag)` when the cell is `'1'`, else `0`.
**Approach 1 - Brute force:** For each cell as top-left, grow the square and check all cells. Time O((m*n) * min(m,n)^2) or worse.
**Approach 2 - Better:** 2D DP table. Time O(m*n), Space O(m*n).
**Approach 3 - Optimal:** One row plus a variable for the old diagonal value.

DP table for the example (answer = max side squared):
```
1 0 1 0 0
1 0 1 1 1
1 1 1 2 2
1 0 0 1 0      max side = 2  -> area 4
```

```js
function maximalSquare(mat) {
  const m = mat.length, n = mat[0].length;
  const dp = new Array(n + 1).fill(0); // extra column avoids j-1 checks
  let best = 0;
  for (let i = 1; i <= m; i++) {
    let diag = 0; // dp[i-1][j-1]
    for (let j = 1; j <= n; j++) {
      const up = dp[j];
      if (mat[i - 1][j - 1] === '1') {
        dp[j] = 1 + Math.min(dp[j], dp[j - 1], diag);
        best = Math.max(best, dp[j]);
      } else dp[j] = 0;
      diag = up; // becomes up-left for the next column
    }
  }
  return best * best;
}
```

**Complexity:** Time O(m*n), Space O(n).
**Watch out:** Return the **area** (side squared), not the side. Cells are the strings `'0'`/`'1'` in the LeetCode version, so compare to `'1'`. Reset `dp[j] = 0` on a `'0'` cell, otherwise stale values leak into the next row.

### 8. Count Square Submatrices with All Ones — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** Count all square submatrices (any size) that contain only 1s. Example: `[[1,0,1],[1,1,0],[1,1,0]]` -> `7`.
**Intuition:** It is the same `dp` as Maximal Square. If the biggest square ending at `(i, j)` has side `s`, then there are exactly `s` squares ending there (sizes 1 to s). So the answer is the **sum** of the whole dp table instead of the max.
**Approach 1 - Brute force:** Check every top-left and every size. Time O((m*n) * min(m,n)^3) naive.
**Approach 2 - Better:** Same DP, separate table. Space O(m*n).
**Approach 3 - Optimal:** Reuse the input matrix as the dp table (if you are allowed to modify it), and add to a running total.

Trace:
```
input     dp
1 0 1     1 0 1
1 1 0     1 1 0
1 1 0     1 2 0     sum = 2 + 2 + 3 = 7
```

```js
function countSquares(mat) {
  const m = mat.length, n = mat[0].length;
  const dp = mat.map((r) => r.slice()); // copy so we do not destroy the input
  let total = 0;
  for (let i = 0; i < m; i++)
    for (let j = 0; j < n; j++) {
      if (i > 0 && j > 0 && dp[i][j] === 1)
        dp[i][j] = 1 + Math.min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]);
      total += dp[i][j];
    }
  return total;
}
```

**Complexity:** Time O(m*n), Space O(m*n) (O(1) extra if modifying the input in place).
**Watch out:** First row and first column keep their original 0/1 value (a square of side 1 at most). Interviewers like this follow-up: "Maximal Square and Count Squares are the same table; only the final reduction differs."

### 9. Out of Boundary Paths — Medium
**Companies:** Google, Amazon
**Problem:** A ball is at `(startRow, startColumn)` in an `m x n` grid. It moves up/down/left/right, at most `maxMove` times. Count the paths that leave the grid. Return modulo `1e9+7`. Example: `m=2, n=2, maxMove=2, start=(0,0)` -> `6`.
**Intuition:** Here the state needs the number of moves left, so it is `dp[moves][i][j]`. Easiest to think "forward": keep the count of ways to be at each cell after `t` moves. When a move steps off the grid, add those ways to the answer.
**Approach 1 - Brute force:** DFS over 4 directions for `maxMove` steps. Time O(4^maxMove).
**Approach 2 - Better:** Memoize `(i, j, movesLeft)`. Time O(maxMove*m*n*4), Space O(maxMove*m*n).
**Approach 3 - Optimal:** Roll the grid by steps; only the current and next layer are stored.

Trace for the example (2x2, 2 moves):
```
after 0 moves: cell (0,0) has 1 way
step 1: from (0,0) two moves leave the grid (ans = 2); one goes to (0,1), one to (1,0)
step 2: (0,1) has 2 ways to leave (ans += 2); (1,0) has 2 ways to leave (ans += 2)
answer = 6
```

```js
function findPaths(m, n, maxMove, sr, sc) {
  const MOD = 1e9 + 7;
  let cur = Array.from({ length: m }, () => new Array(n).fill(0));
  cur[sr][sc] = 1;
  let ans = 0;
  for (let step = 0; step < maxMove; step++) {
    const nxt = Array.from({ length: m }, () => new Array(n).fill(0));
    for (let i = 0; i < m; i++)
      for (let j = 0; j < n; j++) {
        const c = cur[i][j];
        if (!c) continue;
        for (const [di, dj] of [[1, 0], [-1, 0], [0, 1], [0, -1]]) {
          const x = i + di, y = j + dj;
          if (x < 0 || y < 0 || x >= m || y >= n) ans = (ans + c) % MOD; // left the grid
          else nxt[x][y] = (nxt[x][y] + c) % MOD;
        }
      }
    cur = nxt;
  }
  return ans;
}
```

**Complexity:** Time O(maxMove * m * n), Space O(m * n).
**Watch out:** Apply the modulo on every addition. A path counts as soon as it leaves; it must not continue after leaving (we never put it back in `nxt`). With `maxMove = 0` the answer is 0.

### 10. Knight Probability in Chessboard — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** A knight starts at `(row, column)` on an `n x n` board and makes exactly `k` moves, each chosen uniformly from its 8 knight moves (moves off the board are allowed to be chosen, and then it is gone). Return the probability the knight is still on the board. Example: `n=3, k=2, (0,0)` -> `0.0625`.
**Intuition:** Same layered idea as Out of Boundary Paths. Keep the probability of being on each cell after `s` moves. Each step spreads `p/8` to each of the 8 targets that stay on the board. Probability lost off-board just disappears. The answer is the sum of the final layer.
**Approach 1 - Brute force:** Recursion over 8 moves, `k` deep. Time O(8^k).
**Approach 2 - Better:** Memo on `(row, col, movesLeft)`. Time O(k*n^2*8).
**Approach 3 - Optimal:** Roll layers.

Trace for the example:
```
after 0 moves: (0,0) = 1
after 1 move : (1,2) = 1/8, (2,1) = 1/8   (the other 6 moves leave the board)
after 2 moves: from (1,2): (2,0) and (0,0) stay in -> 1/8 * 1/8 each
               from (2,1): (0,2) and (0,0) stay in -> 1/8 * 1/8 each
total = 4 * 1/64 = 0.0625
```

```js
function knightProbability(n, k, r, c) {
  const moves = [[1,2],[2,1],[-1,2],[-2,1],[1,-2],[2,-1],[-1,-2],[-2,-1]];
  let cur = Array.from({ length: n }, () => new Array(n).fill(0));
  cur[r][c] = 1;
  for (let s = 0; s < k; s++) {
    const nxt = Array.from({ length: n }, () => new Array(n).fill(0));
    for (let i = 0; i < n; i++)
      for (let j = 0; j < n; j++) {
        if (!cur[i][j]) continue;
        for (const [di, dj] of moves) {
          const x = i + di, y = j + dj;
          if (x >= 0 && y >= 0 && x < n && y < n) nxt[x][y] += cur[i][j] / 8;
        }
      }
    cur = nxt;
  }
  let p = 0;
  for (const row of cur) for (const v of row) p += v;
  return p;
}
```

**Complexity:** Time O(k * n^2 * 8), Space O(n^2).
**Watch out:** Divide by 8, not by the number of legal moves; the knight picks among all 8 even if some leave. With `k = 0` the answer is 1. Floating-point answers: compare with a small tolerance in tests.

### 11. Longest Increasing Path in a Matrix (Memo DFS) — Hard
**Companies:** Google, Amazon, Microsoft, Facebook
**Problem:** Find the length of the longest path where each step goes up/down/left/right to a strictly larger value. Example: `[[9,9,4],[6,6,8],[2,1,1]]` -> `4` (1 -> 2 -> 6 -> 9).
**Intuition:** Moves go in 4 directions, so row-by-row filling does not work. But "strictly increasing" means you can never come back to a cell. So the grid is a DAG. Let `f(i, j)` = longest increasing path **starting** at `(i, j)`. It is `1 + max f(neighbour)` over larger neighbours. Memoize it.
**Approach 1 - Brute force:** DFS from each cell with no memory. Time exponential in the worst case.
**Approach 2 - Better (the standard answer):** Memoized DFS. Each cell is computed once. Time O(m*n), Space O(m*n).
**Approach 3 - Alternative:** Sort cells by value and fill `dp` from the largest to the smallest (O(mn log mn)), or run Kahn's topological sort on the "smaller -> larger" edges. Good to mention.

Trace on the example (`f` = longest path starting at each cell):
```
values:      f:
9 9 4        1 1 2
6 6 8        2 2 1
2 1 1        3 4 3
```
The cell `1` (bottom middle) has `f = 1 + f(2) = 4`, because `2` has `f = 3` (2 -> 6 -> 9). Answer `4`.

```js
function longestIncreasingPath(g) {
  const m = g.length, n = g[0].length;
  const memo = Array.from({ length: m }, () => new Array(n).fill(0)); // 0 = not computed
  const dirs = [[1, 0], [-1, 0], [0, 1], [0, -1]];
  function dfs(i, j) {
    if (memo[i][j]) return memo[i][j];
    let best = 1;
    for (const [di, dj] of dirs) {
      const x = i + di, y = j + dj;
      if (x >= 0 && y >= 0 && x < m && y < n && g[x][y] > g[i][j])
        best = Math.max(best, 1 + dfs(x, y));
    }
    return (memo[i][j] = best);
  }
  let ans = 0;
  for (let i = 0; i < m; i++)
    for (let j = 0; j < n; j++) ans = Math.max(ans, dfs(i, j));
  return ans;
}
```

**Complexity:** Time O(m*n), Space O(m*n) (memo plus recursion depth up to `m*n` in the worst case).
**Watch out:** No `visited` array is needed because of the strict `>`. If the problem allowed equal values, you would get cycles and this breaks. Very large grids can overflow the JS call stack; the topological-sort version avoids recursion.

### 12. Path with Maximum Gold (Backtracking vs DP) — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** Each cell holds some gold (0 = empty). Start at any non-empty cell, move in 4 directions, never visit a cell twice, never enter a 0 cell. Collect the maximum gold. Example: `[[0,6,0],[5,8,7],[0,9,0]]` -> `24` (9 -> 8 -> 7).
**Intuition:** Why not DP? The rule "do not revisit a cell **in this path**" means the future depends on exactly which cells you already used. That is a set, not a small number, so there is no tidy `dp[i][j]`. With the small limits in the problem (at most 25 gold cells), plain backtracking is the intended answer: mark the cell, explore, unmark.
**Approach 1 - Brute force (and intended):** DFS with backtracking from every non-empty start cell. Time O(k * 3^k) for `k` gold cells (at most 3 new directions after the first move).
**Approach 2 - Better:** None that is general. DP on `(cell, visited-set)` only helps via bitmask for tiny `k`, and gains little here.
**Approach 3 - Optimal:** Backtracking with pruning: start only from cells that are non-zero (and, as an optimisation, from cells with at most one non-zero neighbour first). Mark by setting the cell to 0 temporarily.

Trace on the example, starting at 9: go to 8 (total 17), then from 8 try 5 (22), 7 (24), 6 (23). Best is 24. Starting at 5 gives 5+8+9 = 22 or 5+8+7 = 20. Starting at 6 gives 6+8+9 = 23.

```js
function getMaximumGold(g) {
  const m = g.length, n = g[0].length;
  function dfs(i, j) {
    if (i < 0 || j < 0 || i >= m || j >= n || g[i][j] === 0) return 0;
    const v = g[i][j];
    g[i][j] = 0; // mark visited
    const best = Math.max(dfs(i + 1, j), dfs(i - 1, j), dfs(i, j + 1), dfs(i, j - 1));
    g[i][j] = v; // backtrack: undo the mark
    return v + best;
  }
  let ans = 0;
  for (let i = 0; i < m; i++)
    for (let j = 0; j < n; j++) ans = Math.max(ans, dfs(i, j));
  return ans;
}
```

**Complexity:** Time O(k * 3^k) worst case, Space O(k) recursion.
**Watch out:** You must **restore** the cell after exploring, otherwise other starts see a wrong grid. Do not memoize by cell alone: the best value from a cell depends on what was already visited, so memoizing gives wrong answers. State this clearly; it shows you know when DP does not apply.

### 13. Number of Paths with Max Score — Hard
**Companies:** Google, Amazon
**Problem:** An `n x n` board has `'S'` at bottom-right, `'E'` at top-left, `'X'` for obstacles, and digits `1..9` as points. From `S`, move up, left, or diagonally up-left to reach `E`. Return `[maxScore, numberOfWaysWithThatScore % 1e9+7]`, or `[0, 0]` if no path. Example: `["E23","2X2","12S"]` -> `[7, 1]`.
**Intuition:** Each cell keeps **two** numbers: the best score from `S` to here, and how many paths reach that best. When combining three neighbours, take the highest score; if several tie, add their path counts. This "value + count" pair is a pattern worth remembering (it also appears in Number of Longest Increasing Subsequences).
**Approach 1 - Brute force:** DFS all paths and count. Time exponential.
**Approach 2 - Better:** Memoized DFS carrying `(score, ways)`. Time O(n^2).
**Approach 3 - Optimal:** Bottom-up tables `score` and `ways`, starting from `S` and moving towards `E`. Use `-1` for "unreachable".

Trace (score/ways per cell, filled from `S`):
```
S = 0/1
(2,1) '2' -> 2/1      (2,0) '1' -> 3/1      (1,2) '2' -> 2/1      (1,1) 'X' -> blocked
(1,0) '2' -> 3+2 = 5/1   (0,2) '3' -> 2+3 = 5/1   (0,1) '2' -> best neighbour 5, so 7/1
(0,0) 'E' -> best neighbour 7 -> 7/1
answer [7, 1]
```
Each cell looks at the cell below, to its right, and diagonally below-right, because the code fills from the bottom-right corner.

```js
function pathsWithMaxScore(board) {
  const MOD = 1e9 + 7, n = board.length;
  const score = Array.from({ length: n }, () => new Array(n).fill(-1)); // -1 = unreachable
  const ways = Array.from({ length: n }, () => new Array(n).fill(0));
  score[n - 1][n - 1] = 0;
  ways[n - 1][n - 1] = 1;
  for (let i = n - 1; i >= 0; i--)
    for (let j = n - 1; j >= 0; j--) {
      const ch = board[i][j];
      if ((i === n - 1 && j === n - 1) || ch === 'X') continue;
      let best = -1, cnt = 0;
      for (const [x, y] of [[i + 1, j], [i, j + 1], [i + 1, j + 1]]) {
        if (x >= n || y >= n || score[x][y] < 0) continue;
        if (score[x][y] > best) { best = score[x][y]; cnt = ways[x][y]; }
        else if (score[x][y] === best) cnt = (cnt + ways[x][y]) % MOD;
      }
      if (best < 0) continue; // no way to reach this cell from S
      score[i][j] = best + (ch === 'E' ? 0 : Number(ch));
      ways[i][j] = cnt;
    }
  return score[0][0] < 0 ? [0, 0] : [score[0][0], ways[0][0]];
}
```

**Complexity:** Time O(n^2), Space O(n^2).
**Watch out:** `S` itself has score 0, which is a real score, so use `-1` (not `0`) for "unreachable". `E` adds no points. Take `% MOD` on every count addition. When a cell gets a **strictly better** score, replace the count, do not add to it.

### 14. Dungeon Game — Hard
**Companies:** Amazon, Google, Microsoft, Apple
**Problem:** A knight starts at the top-left of a grid and must reach the bottom-right (the princess). Each cell changes health (negative = damage, positive = potion). Health must stay at least 1 at every moment. Find the minimum starting health. Example: `[[-2,-3,3],[-5,-10,1],[10,30,-5]]` -> `7`.
**Intuition:** Forward DP breaks: "the most health I have so far" and "the least health I needed" pull in different directions, and a big potion later does not help if you die before it. Flip the question: let `dp[i][j]` = **minimum health needed when entering** `(i, j)` to still reach the end alive. Then `dp[i][j] = max(1, min(dp[down], dp[right]) - cell)`. The `max(1, ...)` keeps health at least 1 (a big potion cannot lower the need below 1).
**Approach 1 - Brute force:** DFS tracking health and the minimum needed. Exponential.
**Approach 2 - Better:** Memoized DFS from the end side. Time O(m*n).
**Approach 3 - Optimal:** Bottom-up from the princess, one row.

Trace (needed health, filled from bottom-right):
```
 7  5  2
 6 11  5
 1  1  6
```
Princess cell: `max(1, 1 - (-5)) = 6`. Cell `(2,1)`: `min(inf, 6) - 30 < 1` -> `1`. Cell `(1,1)`: `min(1, 5) - (-10) = 11`. Top-left: `min(6, 5) - (-2) = 7`.

```js
function calculateMinimumHP(d) {
  const m = d.length, n = d[0].length;
  const dp = new Array(n + 1).fill(Infinity);
  dp[n - 1] = 1; // virtual cell below the exit: you need 1 health after the last cell
  for (let i = m - 1; i >= 0; i--)
    for (let j = n - 1; j >= 0; j--) {
      const need = Math.min(dp[j], dp[j + 1]) - d[i][j]; // dp[j] = below, dp[j+1] = right
      dp[j] = Math.max(1, need);
    }
  return dp[0];
}
```

**Complexity:** Time O(m*n), Space O(n).
**Watch out:** The `max(1, ...)` is the whole trick; forgetting it gives zero or negative health. Direction matters: filling from the start with "min health so far" is not enough, since it ignores what damage is still ahead. The bottom row and right column only have one neighbour; the `Infinity` padding handles that.

### 15. Cherry Pickup (I) — Hard
**Companies:** Google, Amazon, Meta, Uber
**Problem:** Grid cells are `0` (empty), `1` (cherry), `-1` (thorn, blocked). Walk from `(0,0)` to `(n-1,n-1)` moving right/down, then walk back to `(0,0)` moving left/up. Picking a cherry empties the cell. Maximise cherries. Return 0 if no valid path exists. Example: `[[0,1,-1],[1,0,-1],[1,1,1]]` -> `5`.
**Intuition:** Walking back is the same as another walk from `(0,0)` to the end (just reversed). So we have **two walkers** going down-right at the same time. If both are always on the same "diagonal" (same number of steps `t`), then walker 1 at row `r1` has column `t - r1`, and walker 2 at row `r2` has column `t - r2`. The state is `(t, r1, r2)`: 3D DP. If both stand on the same cell, count its cherry only once.
**Approach 1 - Brute force:** Find the best path greedily and then the best path again. This is **wrong** (greedy first path can block a better combined answer). The honest brute force is trying all pairs of paths, exponential.
**Approach 2 - Better:** Memoized recursion `f(r1, c1, r2)` (with `c2 = r1 + c1 - r2`). Time O(n^3), Space O(n^3).
**Approach 3 - Optimal:** Layer by `t` and keep only the previous layer: a 2D table over `(r1, r2)`. Time O(n^3), Space O(n^2).

Each walker came from up or left, so there are 4 predecessor pairs: `(r1, r2)`, `(r1-1, r2)`, `(r1, r2-1)`, `(r1-1, r2-1)` in the previous layer.

Trace idea on the example: walker A goes down, down, right, right and picks `(1,0), (2,0), (2,1), (2,2)` = 4 cherries. Walker B (the way back, reversed) goes right, down, down, right and picks `(0,1)` = 1 more (the rest are already taken or empty). Total `5`.

```js
function cherryPickup(grid) {
  const n = grid.length;
  // dp[r1][r2] = best cherries with both walkers t steps from the start
  let dp = Array.from({ length: n }, () => new Array(n).fill(-Infinity));
  dp[0][0] = grid[0][0];
  for (let t = 1; t <= 2 * (n - 1); t++) {
    const nd = Array.from({ length: n }, () => new Array(n).fill(-Infinity));
    for (let r1 = Math.max(0, t - (n - 1)); r1 <= Math.min(n - 1, t); r1++) {
      const c1 = t - r1;
      if (grid[r1][c1] === -1) continue;
      for (let r2 = r1; r2 <= Math.min(n - 1, t); r2++) { // r2 >= r1: symmetry
        const c2 = t - r2;
        if (grid[r2][c2] === -1) continue;
        let best = -Infinity;
        for (const a of [0, 1]) for (const b of [0, 1]) {
          const p1 = r1 - a, p2 = r2 - b; // a/b = 1 means "came from above"
          if (p1 >= 0 && p2 >= 0) best = Math.max(best, dp[p1][p2]);
        }
        if (best === -Infinity) continue;
        let gain = grid[r1][c1];
        if (r1 !== r2) gain += grid[r2][c2]; // same cell: count once
        nd[r1][r2] = best + gain;
      }
    }
    dp = nd;
  }
  return Math.max(0, dp[n - 1][n - 1]);
}
```

**Complexity:** Time O(n^3), Space O(n^2).
**Watch out:** Greedy (take the best path, remove its cherries, take the best again) is a classic wrong answer. Do not forget the `-Infinity` for unreachable states, and `Math.max(0, ...)` at the end for "no path".

### 16. Cherry Pickup II — Hard
**Companies:** Google, Amazon, Meta
**Problem:** Two robots start at the top row: robot A at column 0, robot B at the last column. Each step both move down one row, each shifting its column by -1, 0, or +1. They collect cherries from the cells they visit (a shared cell counts once). Maximise the total at the last row. Example: `[[3,1,1],[2,5,1],[1,5,5],[2,1,1]]` -> `24`.
**Intuition:** Both robots are always in the same row, so the row is the "time". The state is `(row, colA, colB)`. This is the same "two walkers" idea as Cherry Pickup I, but easier because the rows are in sync by definition.
**Approach 1 - Brute force:** 9 combined moves per row. Time O(9^m).
**Approach 2 - Better:** Memoized recursion `(row, a, b)`. Time O(m * n^2 * 9), Space O(m*n^2).
**Approach 3 - Optimal:** Only the previous row of `n x n` states is needed.

Trace on the example: the best pair is robot A on columns `0,0,1,0` (3+2+5+2 = 12) and robot B on columns `2,1,2,2` (1+5+5+1 = 12). Total `24`. In an interview, do not search by hand: define the state, list the 9 predecessors, and run one row.

```js
function cherryPickupII(grid) {
  const m = grid.length, n = grid[0].length;
  let dp = Array.from({ length: n }, () => new Array(n).fill(-Infinity));
  dp[0][n - 1] = grid[0][0] + (n > 1 ? grid[0][n - 1] : 0);
  for (let i = 1; i < m; i++) {
    const nd = Array.from({ length: n }, () => new Array(n).fill(-Infinity));
    for (let a = 0; a < n; a++)
      for (let b = 0; b < n; b++) {
        let best = -Infinity;
        for (let da = -1; da <= 1; da++)
          for (let db = -1; db <= 1; db++) {
            const pa = a + da, pb = b + db; // columns in the previous row
            if (pa >= 0 && pa < n && pb >= 0 && pb < n) best = Math.max(best, dp[pa][pb]);
          }
        if (best === -Infinity) continue;
        nd[a][b] = best + grid[i][a] + (a === b ? 0 : grid[i][b]);
      }
    dp = nd;
  }
  let ans = 0;
  for (const r of dp) for (const v of r) ans = Math.max(ans, v);
  return ans;
}
```

**Complexity:** Time O(m * n^2 * 9), Space O(n^2).
**Watch out:** When both robots stand on the same cell, count it once. With `n = 1` both robots start on the same cell, which is why the base case adds the second cherry only when `n > 1`. The answer is the max over all end columns, not a fixed corner.

## Common Mistakes
- **"Greedy works: just step to the smaller neighbour."** It does not. A cheap step can lead to an expensive region. Use DP.
- **"Forward DP always works."** Dungeon Game needs the reverse direction, because the state is "health needed from here".
- **Wrong base cells.** The first row and first column have only one source. Pad the table (an extra row/column of 0 or `Infinity`) or write the checks carefully.
- **Using `0` for "unreachable" when 0 is a real value.** Use `-1`, `-Infinity`, or `Infinity` depending on the problem.
- **Overwriting the diagonal in a 1-row DP.** If the recurrence needs `dp[i-1][j-1]`, save it in a variable before you overwrite `dp[j]`.
- **Memoizing backtracking by cell alone.** In Gold Mine the value from a cell depends on the visited set, so a per-cell memo is wrong.
- **Adding a `visited` array to Longest Increasing Path.** Not needed (strictly increasing means no cycles) and it can even break the memo.
- **Forgetting modulo on every addition** in counting problems, and counting a shared cell twice in two-walker problems.
- **Row/column mix-ups.** `grid.length` is rows (`m`), `grid[0].length` is columns (`n`). In Cherry Pickup I, `c = t - r`.

## Interview Tips
- Begin with: "Let `dp[i][j]` be ..." and finish the sentence before writing code.
- Say the order: "I fill row by row because each cell depends only on up and left." For Dungeon: "I go backward because the need depends on the future."
- Offer the layers: recursion -> memo -> table -> rolling array. Interviewers usually want the table and then the space cut.
- Ask early: can I modify the input? Is the grid empty? Can the start/end cell be blocked? Are values negative?
- Edge cases to mention: 1xN and Nx1 grids, blocked start or end, all zeros, negative values (Minimum Falling Path), `k = 0` moves, a path that does not exist.
- Common follow-ups:
  - "Print the path, not just the cost." Keep the full table and walk back from the end choosing the neighbour that produced the value.
  - "Now allow moves in 4 directions." Plain DP breaks if cycles are possible; you need Dijkstra (non-negative costs) or a different method.
  - "What if the grid is huge and sparse?" Combinatorics for Unique Paths, or only store the reachable frontier.
  - "Can you do it in O(n) space?" Yes, for most problems here (one row, or two rolling layers).
  - "Cherry Pickup: why not run the best path twice?" Because the first greedy path can steal cherries that a better pair of paths would share or avoid.
- State complexity as `O(m*n)` for time and say which dimension the space reduces to.

## Practice List

**Easy**
- Pascal's Triangle
- Min Cost Climbing Stairs
- Climbing Stairs

**Medium**
- Unique Paths III
- Minimum Falling Path Sum II
- Paint House
- Maximal Rectangle
- Largest Plus Sign
- Number of Increasing Paths in a Grid
- Minimum Cost to Make at Least One Valid Path in a Grid
- Matrix Block Sum
- Range Sum Query 2D - Immutable

**Hard**
- Paths in Matrix Whose Sum Is Divisible by K
- Number of Ways to Paint N x 3 Grid
- Maximum Number of Points with Cost

## Key Takeaways
- **Define `dp[i][j]` in one sentence first.** If you cannot, you are not ready for the recurrence.
- **Fixed start/end gives the answer in one cell; variable start/end gives a min/max over a row or the whole grid.**
- **Fill in the direction of the dependencies.** Forward for "from the start", backward for "need from here" (Dungeon Game, Triangle bottom-up).
- **Most grid DPs only need the row above, so one array is enough.** Save the diagonal in a variable when you need it.
- **Add the missing piece of memory to the state:** the last choice (Ninja Training), the step count (Out of Boundary Paths), or the second walker (Cherry Pickup).
- **Maximal Square and Count Squares share one table;** the answer is the max versus the sum of it.
- **Use memoized DFS for 4-direction monotone moves and backtracking when the visited set matters.** Do not force a DP where the state is not small.

---
[← Previous: One-Dimensional DP](../02-one-d-dp/README.md) | [Next: Subset and Subsequence DP →](../04-subset-and-subsequence-dp/README.md) | [Back to Dynamic Programming Index](../README.md)
