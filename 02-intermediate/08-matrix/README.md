# Matrix

> **Estimated Time:** 5 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - Arrays](../../01-core-linear/01-arrays/README.md), [07 - Prefix Sum](../../01-core-linear/07-prefix-sum/README.md), [08 - Binary Search](../../01-core-linear/08-binary-search/README.md)

---

## What You'll Learn

- You can index a 2D grid without mixing up rows and columns, and walk it with a `directions` array.
- You can reshape a matrix in place: transpose, rotate, set zeroes, game of life.
- You can traverse in tricky orders: spiral, diagonal, zigzag.
- You can search a sorted matrix with binary search or the "staircase" walk.
- You can use a matrix as its own scratch space to reach O(1) extra space.

## The Big Idea

Think of a cinema. Every seat has a row letter and a seat number. To find a friend you do not scan every seat. You go to a row, then walk along it. A matrix is the same: a table of values, found by `(row, col)`.

A matrix is just an array of arrays. In JavaScript, `m[i][j]` means row `i`, column `j`. `m.length` is the number of rows. `m[0].length` is the number of columns. Most matrix bugs are a swapped `i` and `j`, or a missing check for the edges.

Interviewers love matrix problems because they test care, not tricks. The idea is often easy. The hard part is the boundaries, the order of moves, and doing it in place. They also check if you can see that "sorted rows and columns" is a hint for a smart walk, not a full scan.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "Rotate / transpose / flip" | Swap `m[i][j]` with `m[j][i]`, then reverse rows |
| "Do it in place", "O(1) space" | Use the first row/column, or encode two states in one cell |
| "Spiral", "layer by layer" | Four shrinking borders: top, bottom, left, right |
| "Diagonal" | `i + j` is constant on an anti-diagonal; `i - j` is constant on a diagonal |
| "Rows sorted and columns sorted" | Start at a corner and walk (staircase), O(m + n) |
| "Whole matrix sorted row by row" | Treat it as one flat sorted array; binary search with `/` and `%` |
| "Kth smallest", "median" in a sorted matrix | Binary search on the answer value, count with a staircase |
| "8 neighbours", "4 neighbours" | A `directions` array and a bounds check |
| "Sum of a sub-rectangle, many queries" | 2D prefix sums |
| "Largest square / rectangle of 1s" | Grid DP on the bottom-right corner |

## Core Concepts / Template

**1. Indexing and sizes**

```js
const R = m.length;        // rows
const C = m[0].length;     // columns (check m.length > 0 first if empty is possible)
// flat index <-> 2D:  idx = i * C + j   and   i = Math.floor(idx / C), j = idx % C
```

Never write `new Array(3).fill([])` for a grid. `fill` copies the same reference, so every row is one shared array. Use:

```js
const grid = Array.from({ length: R }, () => new Array(C).fill(0));
```

**2. The directions array**

Instead of four copy-pasted blocks, loop over deltas.

```js
const DIRS4 = [[0, 1], [1, 0], [0, -1], [-1, 0]];            // right, down, left, up (clockwise)
const DIRS8 = [[-1,-1],[-1,0],[-1,1],[0,-1],[0,1],[1,-1],[1,0],[1,1]];

for (const [di, dj] of DIRS4) {
  const ni = i + di, nj = j + dj;
  if (ni < 0 || ni >= R || nj < 0 || nj >= C) continue;      // always check bounds first
  // use m[ni][nj]
}
```

The order right, down, left, up is clockwise. Turning right means `d = (d + 1) % 4`. That one line powers spiral walks.

**3. In-place tricks**

- **Use the first row and first column as flags** (Set Matrix Zeroes). Remember their own state in two booleans first.
- **Encode two states in one number** (Game of Life): bit 0 = old state, bit 1 = new state. Read the old state with `& 1`, finish with `>>= 1`.
- **Rotate = transpose + reverse each row.** Clockwise 90 degrees. For counter-clockwise, reverse each row first, then transpose (or transpose, then reverse the column order).

**4. Shrinking borders for layers**

```js
let top = 0, bot = R - 1, left = 0, right = C - 1;
while (top <= bot && left <= right) {
  // walk top row, right col, bottom row, left col
  // then top++, right--, bot--, left++
}
```

The two checks inside the loop (after the top row and right column are done) stop you from walking a single row or column twice. See Spiral Matrix.

**5. Diagonals**

```
 (0,0) (0,1) (0,2)        anti-diagonal s = i + j
 (1,0) (1,1) (1,2)        s=0: (0,0)
 (2,0) (2,1) (2,2)        s=1: (0,1) (1,0)
                          s=2: (0,2) (1,1) (2,0)
```

For diagonal `s`, `i` runs from `max(0, s - C + 1)` to `min(R - 1, s)`, and `j = s - i`.

**Short trace: transpose then reverse rows (rotate 90 clockwise)**

```
1 2 3      transpose      1 4 7      reverse rows      7 4 1
4 5 6   ------------->    2 5 8   --------------->     8 5 2
7 8 9                     3 6 9                        9 6 3
```

## Problems (easy -> hard)

### 1. Transpose Matrix — Easy
**Companies:** Amazon, Google
**Problem:** Return a new matrix where rows and columns are swapped. `[[1,2,3],[4,5,6]]` -> `[[1,4],[2,5],[3,6]]`.
**Intuition:** The value at row `i`, column `j` moves to row `j`, column `i`. The result has `C` rows and `R` columns, so it is a different shape if the matrix is not square.
**Approach 1 - Brute force:** Build a new `C x R` matrix and copy each cell. Time O(R*C), Space O(R*C).
**Approach 2 - Better:** For a square matrix you can swap `m[i][j]` and `m[j][i]` for `j > i` in place, Space O(1). A non-square matrix cannot be done in place this simply, so allocate a new one.
**Approach 3 - Optimal:** The copy is already optimal in time, because every cell must be touched. Trace on `[[1,2,3],[4,5,6]]`: `t[0][0]=1, t[1][0]=2, t[2][0]=3, t[0][1]=4, t[1][1]=5, t[2][1]=6`.

```js
function transpose(m) {
  const R = m.length, C = m[0].length;
  const t = Array.from({ length: C }, () => new Array(R));
  for (let i = 0; i < R; i++)
    for (let j = 0; j < C; j++) t[j][i] = m[i][j];
  return t;
}
```

**Complexity:** Time O(R*C), Space O(R*C) for the output.
**Watch out:** Swapping `i` and `j` the wrong way round, and using the in-place swap on a non-square matrix.

### 2. Toeplitz Matrix — Easy
**Companies:** Google, Amazon
**Problem:** Return true if every diagonal (top-left to bottom-right) has the same value everywhere. `[[1,2,3,4],[5,1,2,3],[9,5,1,2]]` -> `true`.
**Intuition:** A cell must equal the one up-left of it. If every cell equals its up-left neighbour, each diagonal is constant.
**Approach 1 - Brute force:** Collect each diagonal and check it. Time O(R*C), Space O(min(R,C)) or more.
**Approach 2 - Better:** Store the first value of each diagonal in a hash map keyed by `i - j`, compare as you go. Space O(R + C).
**Approach 3 - Optimal:** Just compare each cell with `m[i-1][j-1]`. Trace: `m[1][1]=1` vs `m[0][0]=1` ok; `m[1][2]=2` vs `m[0][1]=2` ok; `m[2][3]=2` vs `m[1][2]=2` ok.

```js
function isToeplitz(m) {
  for (let i = 1; i < m.length; i++)
    for (let j = 1; j < m[0].length; j++)
      if (m[i][j] !== m[i - 1][j - 1]) return false;
  return true;
}
```

**Complexity:** Time O(R*C), Space O(1).
**Watch out:** Start loops at 1 so you never read index -1. A single row always returns true.

### 3. Count Negative Numbers in a Sorted Matrix — Easy
**Companies:** Amazon, Google
**Problem:** Rows are sorted descending and columns are sorted descending. Count the negative numbers. `[[4,3,2,-1],[3,2,1,-1],[1,1,-1,-2],[-1,-1,-2,-3]]` -> `8`.
**Intuition:** Stand at the top-right. If the cell is negative, then everything below it in this column is negative too (columns descend). Count them all, then move left. If it is not negative, move down.
**Approach 1 - Brute force:** Check all cells. Time O(R*C), Space O(1).
**Approach 2 - Better:** Binary search each row for the first negative. Time O(R log C).
**Approach 3 - Optimal:** Staircase walk. The pointer only moves left or down, so at most R + C steps. Trace on the example: `(0,3)=-1` negative, add 4, go left; `(0,2)=2` go down; `(1,2)=1` down; `(2,2)=-1` add 2, left; `(2,1)=1` down; `(3,1)=-1` add 1, left; `(3,0)=-1` add 1, left; out. Total 4+2+1+1 = 8.

```js
function countNegatives(g) {
  let r = 0, c = g[0].length - 1, count = 0;
  while (r < g.length && c >= 0) {
    if (g[r][c] < 0) { count += g.length - r; c--; }
    else r++;
  }
  return count;
}
```

**Complexity:** Time O(R + C), Space O(1).
**Watch out:** `g.length - r` counts the cells from row `r` down to the last row. Do not count the column twice.

### 4. Pascal's Triangle — Easy
**Companies:** Amazon, Apple, Microsoft
**Problem:** Return the first `n` rows. Each inner number is the sum of the two numbers above it. `n = 5` -> `[[1],[1,1],[1,2,1],[1,3,3,1],[1,4,6,4,1]]`.
**Intuition:** Every row starts and ends with 1. Middle value `j` of row `i` equals `t[i-1][j-1] + t[i-1][j]`.
**Approach 1 - Brute force:** Compute each value with the formula C(i, j) using factorials. Works but can overflow and repeats work.
**Approach 2 - Better:** Build row by row from the previous row (below).
**Approach 3 - Optimal:** The row-by-row build is optimal for returning all rows. For one row only, you can compute `C(i, j+1) = C(i, j) * (i - j) / (j + 1)` in O(i). Trace for row 3 from row 2 `[1,2,1]`: `[1, 1+2, 2+1, 1] = [1,3,3,1]`.

```js
function generate(n) {
  const t = [];
  for (let i = 0; i < n; i++) {
    const row = new Array(i + 1).fill(1);
    for (let j = 1; j < i; j++) row[j] = t[i - 1][j - 1] + t[i - 1][j];
    t.push(row);
  }
  return t;
}
```

**Complexity:** Time O(n^2), Space O(n^2) for the output.
**Watch out:** Row `i` has `i + 1` entries. The loop `j < i` skips the two edge ones. Values grow fast, so large rows can exceed safe integer range.

### 5. Row With Maximum Ones — Easy
**Companies:** Amazon, Microsoft
**Problem:** Each row of a 0/1 matrix is sorted (zeros then ones). Return the index of the row with the most ones (the smallest index on a tie), or -1 if there are no ones. `[[0,0,1],[1,1,1],[0,1,1]]` -> `1`.
**Intuition:** Keep a column pointer at the left edge of the ones found so far. For each row, slide it left while the cell is 1. If it moves, this row beats every earlier row.
**Approach 1 - Brute force:** Count ones in every row. Time O(R*C).
**Approach 2 - Better:** Binary search the first 1 in each row. Time O(R log C).
**Approach 3 - Optimal:** One pointer `c` that never moves right. Trace on the example: row 0: `m[0][2]=1` so `c` becomes 1 and best=0; `m[0][1]=0` stops. Row 1: `m[1][1]=1` `c=0`, `m[1][0]=1` `c=-1`, best=1. Row 2: `c=-1`, no move. Answer 1.

```js
function rowWithMaxOnes(m) {
  let best = -1, c = m[0].length - 1;
  for (let i = 0; i < m.length; i++) {
    while (c >= 0 && m[i][c] === 1) { c--; best = i; }
  }
  return best;
}
```

**Complexity:** Time O(R + C), Space O(1).
**Watch out:** Only a strictly longer run of ones changes `best`, so ties keep the earlier row. This needs sorted rows; for unsorted rows count each row.

### 6. Rotate a Matrix by 180 Degrees — Easy
**Companies:** Amazon, Microsoft
**Problem:** Rotate an `R x C` matrix by 180 degrees in place. `[[1,2,3],[4,5,6],[7,8,9]]` -> `[[9,8,7],[6,5,4],[3,2,1]]`.
**Intuition:** A 180 degree turn sends `(i, j)` to `(R-1-i, C-1-j)`. It is a swap of two cells, so no transpose is needed. It is also the same as reversing every row and then reversing the order of the rows.
**Approach 1 - Brute force:** Build a new matrix and copy each cell. Space O(R*C).
**Approach 2 - Better:** Rotate 90 degrees twice. Works for squares, costs two passes.
**Approach 3 - Optimal:** Swap each cell with its mirror, only for the top half of rows. If `R` is odd, fix the middle row by reversing it. Trace on 3x3: swap `1<->9`, `2<->8`, `3<->7`, then the middle row `[4,5,6]` is reversed to `[6,5,4]`.

```js
function rotate180(m) {
  const R = m.length, C = m[0].length;
  for (let i = 0; i < Math.floor(R / 2); i++)
    for (let j = 0; j < C; j++)
      [m[i][j], m[R - 1 - i][C - 1 - j]] = [m[R - 1 - i][C - 1 - j], m[i][j]];
  if (R % 2 === 1) m[(R - 1) / 2].reverse();
  return m;
}
```

**Complexity:** Time O(R*C), Space O(1).
**Watch out:** Looping over all rows swaps every pair twice and undoes the work. Handle the odd middle row on its own.

### 7. Zigzag (Snake) Traversal — Easy
**Companies:** Amazon, Microsoft
**Problem:** Print a matrix row by row, left to right on even rows and right to left on odd rows. `[[1,2,3],[4,5,6],[7,8,9]]` -> `[1,2,3,6,5,4,7,8,9]`.
**Intuition:** The row index parity decides the direction. This is the "snake" pattern.
**Approach 1 - Brute force:** Build each row, reverse odd ones in a copy, then join. Extra space.
**Approach 2 - Better:** Same, but reverse a copy only for odd rows.
**Approach 3 - Optimal:** Read the odd rows backwards with a reverse index. Trace: row 0 gives `1 2 3`; row 1 read from the end gives `6 5 4`; row 2 gives `7 8 9`.

```js
function zigzagRows(m) {
  const out = [];
  m.forEach((row, i) => {
    if (i % 2 === 0) out.push(...row);
    else for (let j = row.length - 1; j >= 0; j--) out.push(row[j]);
  });
  return out;
}
```

**Complexity:** Time O(R*C), Space O(R*C) for the output.
**Watch out:** Do not confuse this with the diagonal zigzag in problem 11. Ask which one is meant. `push(...row)` can fail on huge rows; use a loop if rows have 100k+ items.

### 8. Set Matrix Zeroes — Medium
**Companies:** Amazon, Microsoft, Meta, Google
**Problem:** If a cell is 0, set its whole row and column to 0, in place. `[[1,1,1],[1,0,1],[1,1,1]]` -> `[[1,0,1],[0,0,0],[1,0,1]]`.
**Intuition:** You need to remember which rows and columns to clear. Instead of extra arrays, use the first row and first column of the matrix as the memory.
**Approach 1 - Brute force:** Copy the matrix; for each 0 in the copy, clear its row and column in the original. Time O(R*C*(R+C)), Space O(R*C).
**Approach 2 - Better:** Two arrays `rowZero[R]` and `colZero[C]`. Space O(R + C).
**Approach 3 - Optimal:** Use `m[i][0]` as the flag for row `i` and `m[0][j]` as the flag for column `j`. The first row and column hold flags too, so record in two booleans first whether they had a zero. Then scan the inside and mark. Then clear the inside using the flags. Last, clear the first row and column if their booleans say so.

Trace on `[[1,1,1],[1,0,1],[1,1,1]]`: no zero in row 0 or column 0 (booleans false). The zero at `(1,1)` sets `m[1][0]=0` and `m[0][1]=0`. Clear inside: `(1,1)`, `(1,2)` because `m[1][0]=0`, and `(2,1)` because `m[0][1]=0`. The first row and column stay as marked. Result matches.

```js
function setZeroes(m) {
  const R = m.length, C = m[0].length;
  let row0 = false, col0 = false;
  for (let j = 0; j < C; j++) if (m[0][j] === 0) row0 = true;
  for (let i = 0; i < R; i++) if (m[i][0] === 0) col0 = true;
  for (let i = 1; i < R; i++)
    for (let j = 1; j < C; j++)
      if (m[i][j] === 0) { m[i][0] = 0; m[0][j] = 0; }
  for (let i = 1; i < R; i++)
    for (let j = 1; j < C; j++)
      if (m[i][0] === 0 || m[0][j] === 0) m[i][j] = 0;
  if (row0) for (let j = 0; j < C; j++) m[0][j] = 0;
  if (col0) for (let i = 0; i < R; i++) m[i][0] = 0;
  return m;
}
```

**Complexity:** Time O(R*C), Space O(1).
**Watch out:** Clear the inside first and the first row/column last. If you clear row 0 early, you destroy the flags. Do not set zeros while scanning, or new zeros spread wrongly.

### 9. Rotate Image — Medium
**Companies:** Amazon, Microsoft, Apple, Google, Meta
**Problem:** Rotate an `n x n` matrix 90 degrees clockwise, in place. `[[1,2,3],[4,5,6],[7,8,9]]` -> `[[7,4,1],[8,5,2],[9,6,3]]`.
**Intuition:** A clockwise turn is a transpose followed by a mirror of each row. Both steps are easy in place.
**Approach 1 - Brute force:** New matrix with `res[j][n-1-i] = m[i][j]`. Space O(n^2). Not allowed when "in place" is required.
**Approach 2 - Better:** Rotate four cells at a time in a cycle, layer by layer. In place, but the index maths is easy to get wrong.
**Approach 3 - Optimal:** Transpose then reverse each row (see the trace in Core Concepts). Both are O(n^2) in place.

```js
function rotate(m) {
  const n = m.length;
  for (let i = 0; i < n; i++)
    for (let j = i + 1; j < n; j++) [m[i][j], m[j][i]] = [m[j][i], m[i][j]];
  for (const row of m) row.reverse();
  return m;
}
```

**Complexity:** Time O(n^2), Space O(1).
**Watch out:** Start the inner loop at `i + 1`, not 0, or you swap twice and get the original back. For counter-clockwise, reverse the rows first, then transpose.

### 10. Spiral Matrix — Medium
**Companies:** Amazon, Microsoft, Google, Meta, Apple
**Problem:** Return all elements in spiral order (right, down, left, up, then inward). `[[1,2,3],[4,5,6],[7,8,9]]` -> `[1,2,3,6,9,8,7,4,5]`.
**Intuition:** Peel the matrix like an onion. Four borders shrink after each side is walked.
**Approach 1 - Brute force:** Keep a `visited` matrix and turn right when the next cell is out or visited. Space O(R*C).
**Approach 2 - Better:** Same as above using the directions array and `d = (d + 1) % 4`.
**Approach 3 - Optimal:** Four borders `top, bot, left, right`, no visited array. After the top row and right column, check the borders again before the bottom row and left column, because a single leftover row or column would otherwise be walked twice.

Trace on 3x3: top row `1 2 3` (top=1); right column `6 9` (right=1); bottom row `8 7` (bot=1); left column `4` (left=1). Loop again: top row `5`. Then `top=2 > bot=1`, stop.

```js
function spiralOrder(m) {
  const out = [];
  let top = 0, bot = m.length - 1, left = 0, right = m[0].length - 1;
  while (top <= bot && left <= right) {
    for (let j = left; j <= right; j++) out.push(m[top][j]);
    top++;
    for (let i = top; i <= bot; i++) out.push(m[i][right]);
    right--;
    if (top <= bot) {
      for (let j = right; j >= left; j--) out.push(m[bot][j]);
      bot--;
    }
    if (left <= right) {
      for (let i = bot; i >= top; i--) out.push(m[i][left]);
      left++;
    }
  }
  return out;
}
```

**Complexity:** Time O(R*C), Space O(1) extra (output not counted).
**Watch out:** Without the two `if` checks, a 1-row or 1-column matrix repeats numbers. Test `[[1,2,3]]` and `[[1],[2],[3]]`.

### 11. Spiral Matrix II — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Given `n`, build an `n x n` matrix filled with `1..n*n` in spiral order. `n = 3` -> `[[1,2,3],[8,9,4],[7,6,5]]`.
**Intuition:** Walk the spiral with the directions array. Keep going straight until the next cell is outside or already filled, then turn right.
**Approach 1 - Brute force:** Fill by four borders like problem 10. Works, just longer.
**Approach 2 - Better:** The same border loop, writing numbers instead of reading.
**Approach 3 - Optimal:** One loop of `n*n` steps with a direction index. A cell is "filled" when it is not 0, so the matrix itself is the visited array. Trace for `n = 2`: write 1 at (0,0), next (0,1) free; write 2 at (0,1), next (0,2) out so turn down; write 3 at (1,1), next (2,1) out so turn left; write 4 at (1,0).

```js
function generateMatrix(n) {
  const m = Array.from({ length: n }, () => new Array(n).fill(0));
  const D = [[0, 1], [1, 0], [0, -1], [-1, 0]];
  let r = 0, c = 0, d = 0;
  for (let k = 1; k <= n * n; k++) {
    m[r][c] = k;
    const nr = r + D[d][0], nc = c + D[d][1];
    if (nr < 0 || nr >= n || nc < 0 || nc >= n || m[nr][nc] !== 0) d = (d + 1) % 4;
    r += D[d][0]; c += D[d][1];
  }
  return m;
}
```

**Complexity:** Time O(n^2), Space O(1) beyond the output.
**Watch out:** After the last number the code steps once more, but it only moves `r, c` and never writes, so it is safe. Check bounds before reading `m[nr][nc]`.

### 12. Diagonal Traverse — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Return elements in the zigzag diagonal order: up-right, then down-left, and so on. `[[1,2,3],[4,5,6],[7,8,9]]` -> `[1,2,4,7,5,3,6,8,9]`.
**Intuition:** All cells on one anti-diagonal share `i + j = s`. Collect each diagonal. Even `s` goes up-right (row decreasing), odd `s` goes down-left (row increasing). Walk each diagonal from the top row down, and reverse it when `s` is even.
**Approach 1 - Brute force:** Put each diagonal in a hash map keyed by `i + j`, then output with reversal on even keys. Space O(R*C).
**Approach 2 - Better:** Loop over `s` directly and collect one diagonal at a time (below). Space O(min(R,C)) for the temporary list.
**Approach 3 - Optimal:** Simulate with a direction flag and bounce at the borders. It is O(1) extra space but has many edge cases. The diagonal-by-diagonal version is cleaner and what I would write in an interview. Trace for 3x3: `s=0` -> `[1]`; `s=1` rows 0..1 -> `[2,4]`; `s=2` rows 0..2 -> `[3,5,7]`, even so reversed to `[7,5,3]`; `s=3` -> `[6,8]`; `s=4` -> `[9]`. Joined: `1,2,4,7,5,3,6,8,9`.

```js
function findDiagonalOrder(m) {
  const R = m.length, C = m[0].length, out = [];
  for (let s = 0; s <= R + C - 2; s++) {
    const tmp = [];
    for (let i = Math.max(0, s - C + 1); i <= Math.min(R - 1, s); i++) tmp.push(m[i][s - i]);
    if (s % 2 === 0) tmp.reverse();
    out.push(...tmp);
  }
  return out;
}
```

**Complexity:** Time O(R*C), Space O(min(R, C)) extra.
**Watch out:** The row range `max(0, s-C+1)..min(R-1, s)` keeps `j` inside the matrix. Test one row and one column.

### 13. Search a 2D Matrix — Medium
**Companies:** Amazon, Microsoft, Meta, Google
**Problem:** Each row is sorted, and the first number of each row is larger than the last of the row before. Return whether `target` exists. `target = 3` in `[[1,3,5,7],[10,11,16,20],[23,30,34,60]]` -> `true`.
**Intuition:** If you read the rows one after another, you get one long sorted array. Binary search it, and convert the middle index to `(row, col)`.
**Approach 1 - Brute force:** Scan every cell. Time O(R*C).
**Approach 2 - Better:** Binary search the right row by its first/last value, then binary search inside it. Time O(log R + log C).
**Approach 3 - Optimal:** One binary search over `0 .. R*C-1` with `row = floor(mid / C)`, `col = mid % C`. It is the same O(log(R*C)) and has less code. Trace for target 3: `lo=0, hi=11, mid=5 -> 11` too big, `hi=4`; `mid=2 -> 5` too big, `hi=1`; `mid=0 -> 1` too small, `lo=1`; `mid=1 -> 3` found.

```js
function searchMatrix(m, target) {
  const R = m.length, C = m[0].length;
  let lo = 0, hi = R * C - 1;
  while (lo <= hi) {
    const mid = (lo + hi) >> 1;
    const v = m[Math.floor(mid / C)][mid % C];
    if (v === target) return true;
    if (v < target) lo = mid + 1; else hi = mid - 1;
  }
  return false;
}
```

**Complexity:** Time O(log(R*C)), Space O(1).
**Watch out:** Use `C` (the column count) to divide, not `R`. Do not mix this with problem 14, where the rows are not chained.

### 14. Search a 2D Matrix II — Medium
**Companies:** Amazon, Google, Microsoft, Apple
**Problem:** Rows are sorted left to right and columns are sorted top to bottom, but rows are not chained. Find `target`. `target = 5` in the 5x5 sample -> `true`.
**Intuition:** Start at the top-right corner. It is the biggest in its row and the smallest in its column. If it is too big, the whole column below is bigger still, so drop the column. If it is too small, the whole row to its left is smaller still, so drop the row. Each step throws away a full row or column.
**Approach 1 - Brute force:** Scan all. Time O(R*C).
**Approach 2 - Better:** Binary search each row. Time O(R log C).
**Approach 3 - Optimal:** The staircase walk. Trace for target 5 in `[[1,4,7,11,15],[2,5,8,12,19],[3,6,9,16,22],...]`: start 15 > 5, go left; 11 > 5, left; 7 > 5, left; 4 < 5, go down; `m[1][1]=5` found.

```js
function searchMatrix2(m, target) {
  let r = 0, c = m[0].length - 1;
  while (r < m.length && c >= 0) {
    const v = m[r][c];
    if (v === target) return true;
    if (v > target) c--; else r++;
  }
  return false;
}
```

**Complexity:** Time O(R + C), Space O(1).
**Watch out:** Start at the top-right or bottom-left. Top-left and bottom-right do not work, because both directions then go the same way (both bigger or both smaller) and you cannot decide.

### 15. Valid Sudoku — Medium
**Companies:** Amazon, Google, Apple, Microsoft, Meta
**Problem:** Check that a partly filled 9x9 board has no repeated digit in any row, column, or 3x3 box. Empty cells are `'.'`. You do not need to solve it.
**Intuition:** One pass, three sets of "seen digits": one per row, per column, per box. The box number of `(i, j)` is `floor(i/3) * 3 + floor(j/3)`.
**Approach 1 - Brute force:** For every filled cell, rescan its row, column and box. Time O(81 * 27), still tiny, but messy.
**Approach 2 - Better:** Three separate passes, one for rows, one for columns, one for boxes.
**Approach 3 - Optimal:** One pass with 9 row sets, 9 column sets and 9 box sets. If a digit is already in any of its three sets, return false. The board is fixed size, so the time is constant. Trace: the second `5` in a board's first box would find `5` in `box[0]` and return false.

```js
function isValidSudoku(b) {
  const mk = () => Array.from({ length: 9 }, () => new Set());
  const rows = mk(), cols = mk(), box = mk();
  for (let i = 0; i < 9; i++)
    for (let j = 0; j < 9; j++) {
      const v = b[i][j];
      if (v === '.') continue;
      const k = Math.floor(i / 3) * 3 + Math.floor(j / 3);
      if (rows[i].has(v) || cols[j].has(v) || box[k].has(v)) return false;
      rows[i].add(v); cols[j].add(v); box[k].add(v);
    }
  return true;
}
```

**Complexity:** Time O(1) for a fixed 9x9 board (O(n^2) for an n x n board), Space O(1) (O(n^2) in general).
**Watch out:** Valid does not mean solvable. Only check the filled digits. Digits are strings, so compare strings with strings.

### 16. Game of Life — Medium
**Companies:** Amazon, Google, Microsoft, Dropbox
**Problem:** Each cell is live (1) or dead (0). All cells update at once using the 8 neighbours: a live cell with 2 or 3 live neighbours lives, otherwise dies; a dead cell with exactly 3 live neighbours becomes live. Update the board in place.
**Intuition:** The trouble is that updating one cell changes the data its neighbours need. Keep the old state in bit 0 and write the new state in bit 1. Neighbour counts read only bit 0. When all is done, shift right by one to keep the new state.
**Approach 1 - Brute force:** Copy the board, read from the copy, write to the original. Space O(R*C).
**Approach 2 - Better:** Use special marker values like 2 ("was live, now dead") and -1; this is O(1) space but harder to read.
**Approach 3 - Optimal:** Two-bit encoding. State values: `0` = dead->dead, `1` = live->dead, `2` = dead->live, `3` = live->live. Reading old state: `v & 1`. Trace for `[[1,1],[1,0]]`: cell (0,0) has 2 live neighbours, live, so becomes 3. Cell (0,1) has 2, becomes 3. Cell (1,0) has 2, becomes 3. Dead cell (1,1) has 3 live neighbours, becomes 2. After `>>= 1`: `[[1,1],[1,1]]`.

```js
function gameOfLife(b) {
  const R = b.length, C = b[0].length;
  for (let i = 0; i < R; i++)
    for (let j = 0; j < C; j++) {
      let n = 0;
      for (let di = -1; di <= 1; di++)
        for (let dj = -1; dj <= 1; dj++) {
          if (!di && !dj) continue;
          const x = i + di, y = j + dj;
          if (x >= 0 && x < R && y >= 0 && y < C) n += b[x][y] & 1; // old state only
        }
      if (b[i][j] === 1 && (n === 2 || n === 3)) b[i][j] = 3;
      else if (b[i][j] === 0 && n === 3) b[i][j] = 2;
    }
  for (let i = 0; i < R; i++)
    for (let j = 0; j < C; j++) b[i][j] >>= 1;
  return b;
}
```

**Complexity:** Time O(R*C*8), Space O(1).
**Watch out:** Always read neighbours with `& 1`, never the raw value. The "infinite board" follow-up cannot be done with a fixed grid; store only live cells in a set.

### 17. Maximal Square — Medium
**Companies:** Amazon, Google, Apple, Microsoft, Meta
**Problem:** In a matrix of `'0'` and `'1'`, find the area of the largest square of all 1s. The sample `[["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]` -> `4`.
**Intuition:** Let `dp[i][j]` be the side of the biggest square whose bottom-right corner is `(i, j)`. That square can only be as big as the smallest of the three squares touching it from up, left and up-left, plus one.
**Approach 1 - Brute force:** For each cell as top-left, grow the side while the square is all ones. Time O((R*C) * min(R,C)^2) or so.
**Approach 2 - Better:** Same but check growth with 2D prefix sums. Time O(R*C*min(R,C)).
**Approach 3 - Optimal:** The DP: `dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])` when the cell is `'1'`, else 0. Answer is `max(dp)^2`. A padded row and column of zeros avoids edge checks. Trace on the sample: rows 2-3, columns 2-3 hold a 2x2 block of ones. Its top-left, top-right and bottom-left cells have dp 1, so the bottom-right gets `1 + min(1,1,1) = 2`. Best side 2, area 4.

```js
function maximalSquare(m) {
  const R = m.length, C = m[0].length;
  const dp = Array.from({ length: R + 1 }, () => new Array(C + 1).fill(0));
  let best = 0;
  for (let i = 1; i <= R; i++)
    for (let j = 1; j <= C; j++)
      if (m[i - 1][j - 1] === '1') {
        dp[i][j] = 1 + Math.min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]);
        best = Math.max(best, dp[i][j]);
      }
  return best * best;
}
```

**Complexity:** Time O(R*C), Space O(R*C) (can be cut to O(C) with a rolling row).
**Watch out:** Return the area, not the side. Cells are strings `'1'`, not numbers. More on this style in the grid DP chapter: [03 - Grid DP](../../05-dynamic-programming/03-grid-dp/README.md).

### 18. Kth Smallest Element in a Sorted Matrix — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** In an `n x n` matrix where every row and column is sorted ascending, return the `k`-th smallest value (counting duplicates). `[[1,5,9],[10,11,13],[12,13,15]], k = 8` -> `13`.
**Intuition:** Rows are not chained, so you cannot index the k-th directly. But you can ask "how many values are `<= x`?" quickly with the staircase walk. Binary search for the smallest `x` whose count is at least `k`. The answer is always a value in the matrix.
**Approach 1 - Brute force:** Flatten and sort. Time O(n^2 log n), Space O(n^2).
**Approach 2 - Better:** A min-heap seeded with the first column; pop `k` times, pushing the right neighbour each time. Time O(k log n). See [05 - Heaps](../05-heaps/README.md).
**Approach 3 - Optimal:** Binary search on the value range `[m[0][0], m[n-1][n-1]]`. Time O(n log(max - min)). Trace on the sample: range 1..15, `mid=8`: values `<=8` are 1,5 so count 2 < 8, `lo=9`; `mid=12`: 1,5,9,10,11,12 gives 6 < 8, `lo=13`; `mid=14`: count is 8 (also 13, 13) so `hi=14`; `mid=13`: count 8 >= 8, `hi=13`. Answer 13.

```js
function kthSmallest(m, k) {
  const n = m.length;
  let lo = m[0][0], hi = m[n - 1][n - 1];
  const countLE = (x) => {
    let c = 0, j = n - 1;
    for (let i = 0; i < n; i++) {
      while (j >= 0 && m[i][j] > x) j--;
      c += j + 1;
    }
    return c;
  };
  while (lo < hi) {
    const mid = Math.floor((lo + hi) / 2);
    if (countLE(mid) >= k) hi = mid; else lo = mid + 1;
  }
  return lo;
}
```

**Complexity:** Time O(n * log(max - min)), Space O(1).
**Watch out:** Use `hi = mid` (not `mid - 1`) so you keep a candidate. Use `Math.floor` because `mid` can be negative or fractional. The pointer `j` in `countLE` only moves left, that is why it is O(n).

### 19. Median of a Row-Wise Sorted Matrix — Hard
**Companies:** Amazon, Google, Microsoft
**Problem:** Each row of an `R x C` matrix is sorted ascending. `R * C` is odd. Find the median of all values without flattening and sorting. `[[1,3,5],[2,6,9],[3,6,9]]` -> `5`.
**Intuition:** The median is the value `x` such that more than half of the numbers are `<= x`, and it is the smallest such `x`. Binary search `x`. To count numbers `<= x`, binary search (upper bound) in each row. Columns are not sorted, so the staircase from problem 18 does not apply.
**Approach 1 - Brute force:** Flatten and sort. Time O(R*C log(R*C)), Space O(R*C).
**Approach 2 - Better:** K-way merge with a heap until the middle element. Time O((R*C) log R).
**Approach 3 - Optimal:** Binary search on the value, count with a per-row upper bound. Let `need = floor(R*C / 2) + 1`. Find the smallest `x` with `count(<= x) >= need`. Trace on the sample (9 values, need 5): range 1..9. `mid=5`: row counts 3+1+1 = 5 >= 5, `hi=5`. `mid=3`: counts 2+1+1 = 4 < 5, `lo=4`. `mid=4`: 4 < 5, `lo=5`. Answer 5.

```js
function upperBound(a, x) {            // count of items <= x in a sorted row
  let lo = 0, hi = a.length;
  while (lo < hi) {
    const mid = (lo + hi) >> 1;
    if (a[mid] <= x) lo = mid + 1; else hi = mid;
  }
  return lo;
}

function medianRowWise(m) {
  const R = m.length, C = m[0].length;
  let lo = Infinity, hi = -Infinity;
  for (const row of m) { lo = Math.min(lo, row[0]); hi = Math.max(hi, row[C - 1]); }
  const need = Math.floor((R * C) / 2) + 1;
  while (lo < hi) {
    const mid = Math.floor((lo + hi) / 2);
    let c = 0;
    for (const row of m) c += upperBound(row, mid);
    if (c < need) lo = mid + 1; else hi = mid;
  }
  return lo;
}
```

**Complexity:** Time O(R * log C * log(max - min)), Space O(1).
**Watch out:** This assumes an odd count. For an even count, say so and ask what the median should be (usually the average of two middle values, found with two searches). Do not use `/ 2 | 0` on negatives; use `Math.floor`.

### 20. Range Sum Query 2D (Matrix Prefix Sums) — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Build a structure from a matrix, then answer many queries: the sum of the rectangle from `(r1, c1)` to `(r2, c2)`.
**Intuition:** Same idea as 1D prefix sums, in two dimensions. Let `p[i][j]` be the sum of the rectangle from `(0,0)` to `(i-1, j-1)`. Any rectangle is "big minus two strips plus the overlap that was subtracted twice". The full theory is in [07 - Prefix Sum](../../01-core-linear/07-prefix-sum/README.md); here is the matrix version.
**Approach 1 - Brute force:** Add up the rectangle on each query. Time O(R*C) per query.
**Approach 2 - Better:** Keep a 1D prefix per row, sum one slice per row. Time O(R) per query.
**Approach 3 - Optimal:** 2D prefix table, O(1) per query. Build: `p[i+1][j+1] = m[i][j] + p[i][j+1] + p[i+1][j] - p[i][j]`. Query: `p[r2+1][c2+1] - p[r1][c2+1] - p[r2+1][c1] + p[r1][c1]`. Sample: querying `(2,1)` to `(4,3)` on the LeetCode sample gives `8`.

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

**Complexity:** Build O(R*C), each query O(1), Space O(R*C).
**Watch out:** The table has one extra row and column of zeros so that `r1 = 0` or `c1 = 0` needs no special case. Add on the overlap, do not forget it.

### 21. Number of Islands (grid DFS/BFS) — Medium
**Companies:** Amazon, Google, Meta, Microsoft, Apple
**Problem:** Count groups of connected `'1'` cells (4 directions) in a grid. Full treatment is in [02 - Grid and Islands](../../04-graphs/02-grid-and-islands/README.md); this is the matrix-flavoured short version.
**Intuition:** Scan the grid. When you meet an unvisited `'1'`, count one island and flood it so it is never counted again.
**Approach 1 - Brute force:** Not needed; flood fill is the natural solution.
**Approach 2 - Better:** DFS with a separate `visited` matrix. Space O(R*C).
**Approach 3 - Optimal:** DFS that overwrites visited cells with `'0'`, using the directions array. Trace on `[["1","1","0"],["0","0","0"],["1","0","1"]]`: island at (0,0) floods (0,1); island at (2,0); island at (2,2). Count 3.

```js
function numIslands(g) {
  const R = g.length, C = g[0].length;
  let count = 0;
  const dfs = (i, j) => {
    if (i < 0 || j < 0 || i >= R || j >= C || g[i][j] !== '1') return;
    g[i][j] = '0';
    for (const [di, dj] of [[1, 0], [-1, 0], [0, 1], [0, -1]]) dfs(i + di, j + dj);
  };
  for (let i = 0; i < R; i++)
    for (let j = 0; j < C; j++)
      if (g[i][j] === '1') { count++; dfs(i, j); }
  return count;
}
```

**Complexity:** Time O(R*C), Space O(R*C) worst case for recursion depth.
**Watch out:** It changes the input; say so, or restore it. Deep recursion on a big grid can overflow the stack in JS, so use BFS with a queue for huge inputs.

### 22. Word Search (grid backtracking) — Medium
**Companies:** Amazon, Microsoft, Google, Meta
**Problem:** Does the word exist in the grid by walking to adjacent cells, using each cell at most once per path? Full treatment lives in [02 - Backtracking](../02-backtracking/README.md); here is the matrix angle.
**Intuition:** Try every cell as a start. Mark the cell as used, try the four neighbours for the next letter, then unmark it ("undo") so other paths can use it.
**Approach 1 - Brute force:** Generate all paths and compare. Far too slow.
**Approach 2 - Better:** DFS with a separate `visited` matrix.
**Approach 3 - Optimal:** DFS that marks the cell by overwriting it with `'#'` and restores it afterwards. It prunes as soon as a letter does not match. Trace for `"ABCCED"` on the standard sample: A(0,0) -> B(0,1) -> C(0,2) -> C(1,2) -> E(2,2) -> D(2,1). Found.

```js
function exist(b, word) {
  const R = b.length, C = b[0].length;
  const dfs = (i, j, k) => {
    if (k === word.length) return true;
    if (i < 0 || j < 0 || i >= R || j >= C || b[i][j] !== word[k]) return false;
    const ch = b[i][j];
    b[i][j] = '#';
    const ok = dfs(i + 1, j, k + 1) || dfs(i - 1, j, k + 1) || dfs(i, j + 1, k + 1) || dfs(i, j - 1, k + 1);
    b[i][j] = ch;                                   // undo
    return ok;
  };
  for (let i = 0; i < R; i++)
    for (let j = 0; j < C; j++) if (dfs(i, j, 0)) return true;
  return false;
}
```

**Complexity:** Time O(R*C * 3^L) worst case for word length `L`, Space O(L) recursion.
**Watch out:** Forgetting to restore the cell breaks other branches. Check the length condition before the bounds check, or the last letter fails.

## Common Mistakes

- **"`new Array(R).fill(new Array(C).fill(0))` makes a grid."** It makes `R` copies of one row. Writing to one cell changes every row. Use `Array.from({ length: R }, () => new Array(C).fill(0))`.
- **"The matrix is square."** Many problems give `R != C`. Do not use one `n` for both. Transpose of a non-square matrix has a new shape.
- **"Swap in the transpose over the whole grid."** Loop `j` from `i + 1`. Otherwise every pair swaps twice and nothing changes.
- **"Set zeroes as you scan."** New zeros then spread to new rows and columns. Mark first, clear after.
- **"Spiral needs no border checks after the top row."** On a single row or column the walk repeats cells. Check `top <= bot` and `left <= right` again before the other two sides.
- **"Row is `i`, column is `j` everywhere."** In some problems (and in math "x, y") the order is reversed. Read the signature. Using `m[x][y]` with `x` as column is a classic bug.
- **"Search II also works from the top-left."** From the top-left both moves make the value bigger, so you cannot decide. Start at the top-right or bottom-left.
- **"A bounds check can come after the read."** `m[-1]` is `undefined` and `m[-1][0]` throws. Always check indices first.

## Interview Tips

- **Say the shape first.** "I will assume the matrix is `R x C` and not empty. Is it always square?" Ask about empty input.
- **Say the indexing out loud.** "`m[i][j]` is row `i`, column `j`." It prevents the swapped-index bug and shows care.
- **State the trade-off for in place.** "I can do this with an extra matrix in O(R*C) space, or in place in O(1). Which do you prefer?" Then code the one they pick.
- **Name your in-place trick.** "I will use the first row and column as flags, and two booleans to remember their own state."
- **Follow-ups you should expect:** rotate counter-clockwise or 180 degrees; do spiral for a non-square matrix; Game of Life on an infinite board; Set Zeroes with O(R + C) space first and then O(1); kth smallest with a heap versus binary search; search a matrix when rows are sorted but not chained.
- **Edge cases to mention:** 1x1, a single row, a single column, all zeros, duplicates, negative numbers, `R != C`, empty matrix.
- **Test with a tiny example by hand.** A 2x2 or 3x3 trace catches boundary mistakes quickly.
- **Do not mutate input without saying so.** If the problem does not allow mutation (for example Number of Islands), copy or restore it.

## Practice List

**Easy**
- Flipping an Image
- Reshape the Matrix
- Matrix Diagonal Sum
- Lucky Numbers in a Matrix
- Island Perimeter
- Flood Fill

**Medium**
- Rotate Image (counter-clockwise variant)
- Search a 2D Matrix (rows not chained variant)
- Sort the Matrix Diagonally
- Range Sum Query 2D - Immutable
- Maximal Rectangle
- Surrounded Regions
- Pacific Atlantic Water Flow
- Rotting Oranges
- 01 Matrix
- Longest Increasing Path in a Matrix

**Hard**
- Largest Rectangle in Histogram (building block for Maximal Rectangle)
- Number of Submatrices That Sum to Target
- Max Sum of Rectangle No Larger Than K
- Sudoku Solver
- N-Queens
- Word Search II
- Median of Two Sorted Arrays
- Shortest Path in a Grid with Obstacles Elimination

## Key Takeaways

- **`m[i][j]` is row `i`, column `j`**; `m.length` is rows and `m[0].length` is columns. Say it out loud to avoid swaps.
- **Use a `directions` array** and check bounds before reading, instead of copy-pasting four neighbour blocks.
- **Rotate 90 clockwise = transpose, then reverse each row.** 180 degrees is a mirror swap of `(i, j)` with `(R-1-i, C-1-j)`.
- **Shrinking borders** (`top, bot, left, right`) handle spiral order. Re-check the borders to avoid repeats on thin matrices.
- **In place means encoding state inside the matrix:** first row/column as flags, or two bits per cell.
- **Sorted rows and columns:** the top-right **staircase walk** gives O(R + C). A fully chained matrix is one flat sorted array for binary search.
- **Binary search on the answer value** solves kth smallest and median in a sorted matrix, with a counting helper inside.
- **2D prefix sums** give O(1) rectangle sums: add the big corner, subtract two strips, add back the overlap.

---

[← Previous: Bit Manipulation](../07-bit-manipulation/README.md) | [Next: Randomization and Sampling →](../09-randomization-and-sampling/README.md) | [Back to Intermediate Index](../README.md)
