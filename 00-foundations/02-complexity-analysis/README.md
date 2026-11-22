# Complexity Analysis

> **Estimated Time:** 3 hr | **Difficulty:** Beginner
> **Prerequisites:** [01 - Problem Solving Framework](../01-problem-solving-framework/README.md)

---

## What You'll Learn

- You can explain Big-O, Big-Omega and Big-Theta in plain words and know which one interviewers mean.
- You can count the time and space cost of loops, nested loops and recursion.
- You can solve simple recurrences (like `T(n) = 2T(n/2) + n`) with the Master theorem idea.
- You can explain amortized cost using a dynamic array and a queue built from two stacks.
- You can read a code snippet and state its complexity in under a minute.

## The Big Idea

Think about two ways to find a name in a phone book. One person reads every page from the start. Another opens the middle, decides "before or after", and repeats. If the book has 100 pages, both are fast. If it has 10 million pages, the first person is stuck for days and the second is done in about 24 steps. We do not care about seconds on one laptop. We care about how the work **grows** when the input grows.

That is complexity analysis. We count how many basic steps a program does as a function of the input size `n`, and we keep only the part that dominates when `n` is huge. We do the same for memory.

Why interviewers love it:

- It is the shared language for comparing solutions ("brute force is O(n^2), I can do O(n log n)").
- Every problem has a size limit hidden in it. If `n = 10^5`, an O(n^2) solution will time out, and you should know that before you code.
- You are expected to state time and space for your final answer, every time, without being asked.

### Time versus space

- **Time complexity:** how the number of steps grows with `n`.
- **Space complexity:** how the extra memory grows with `n`. Usually we count **auxiliary space** (extra memory you allocate, including the recursion stack) and not the input itself. Say which one you mean if the interviewer is strict.

Often you can trade one for the other. A hash map spends O(n) space to cut a search from O(n) to O(1).

## How to Spot It

Use the input limit as a hint for the complexity you need. This is a rule of thumb for roughly 1 second of work (about 10^8 simple operations).

| If the problem says | Aim for | Typical technique |
|---|---|---|
| `n <= 10` or `n <= 12` | O(n!) or O(2^n * n) | brute-force permutations, bitmask |
| `n <= 20` to `25` | O(2^n) | backtracking, subsets |
| `n <= 500` | O(n^3) | triple loop, Floyd-Warshall |
| `n <= 5,000` | O(n^2) | 2D DP, double loop |
| `n <= 10^5` or `10^6` | O(n log n) or O(n) | sorting, heap, binary search, hash map, two pointers |
| `n <= 10^9` or `10^18` | O(log n) or O(1) | binary search on the answer, math |
| "find in sorted data" | O(log n) | binary search |
| "each step halves the input" | O(log n) | divide and conquer |
| "try all subsets" | O(2^n) | backtracking |
| "try all orderings" | O(n!) | permutations |

## Core Concepts / Template

### 1. Big-O, Big-Omega, Big-Theta

Say a function `f(n)` is the exact step count of your code.

| Symbol | Meaning in plain words | Think of it as |
|---|---|---|
| **O(g)** (Big-O) | `f` grows **no faster than** `g` (up to a constant). | an upper bound ("at most") |
| **Omega(g)** (Big-Omega) | `f` grows **at least as fast as** `g`. | a lower bound ("at least") |
| **Theta(g)** (Big-Theta) | `f` grows **exactly like** `g`: both O(g) and Omega(g). | a tight bound ("about") |

Example: `f(n) = 3n + 5`. It is O(n), Omega(n) and Theta(n). It is also technically O(n^2), because n^2 is a looser upper bound, but nobody says that.

In interviews people say "Big-O" and mean "the tightest bound I can honestly give for the case we are discussing". Do not confuse the symbol with the case. Big-O is a bound on a function. "Worst case" is which function we measure. You can have the Big-O of the best case, and the Omega of the worst case. Keep them separate in your head.

### 2. The rules for simplifying

1. **Drop constants.** `5n` becomes O(n). `n/2` becomes O(n).
2. **Drop smaller terms.** `n^2 + n + 100` becomes O(n^2).
3. **Add** the costs of steps done one after another: `O(n) + O(m)` is `O(n + m)`. If `n` and `m` are different inputs, keep both letters.
4. **Multiply** the costs of steps that are nested: a loop of `n` that does `O(m)` work inside is `O(n * m)`.
5. **Different inputs, different letters.** Looping over `arr1` (size `a`) then over `arr2` (size `b)` is O(a + b), not O(n).
6. **Log base does not matter.** log2 n and log10 n differ by a constant, so we write O(log n).

### 3. Common growth table

How many steps for a given `n` (rounded, for feel):

| Name | Big-O | n = 10 | n = 1,000 | n = 1,000,000 | Typical example |
|---|---|---|---|---|---|
| Constant | O(1) | 1 | 1 | 1 | array index, hash lookup |
| Logarithmic | O(log n) | 3 | 10 | 20 | binary search |
| Linear | O(n) | 10 | 1,000 | 10^6 | one loop |
| Linearithmic | O(n log n) | 33 | 10,000 | 2 * 10^7 | merge sort, heap sort |
| Quadratic | O(n^2) | 100 | 10^6 | 10^12 (too slow) | double loop, bubble sort |
| Cubic | O(n^3) | 1,000 | 10^9 | far too slow | triple loop |
| Exponential | O(2^n) | 1,024 | astronomically large | impossible | all subsets, naive Fibonacci |
| Factorial | O(n!) | 3.6 million | impossible | impossible | all permutations |

Order from fast to slow: `1 < log n < sqrt(n) < n < n log n < n^2 < n^3 < 2^n < n!`.

### 4. Counting loops

**Single loop.** The body runs `n` times. Cost = `n * (cost of body)`.

```js
function sum(arr) {
  let total = 0;
  for (const x of arr) total += x; // n times, O(1) each
  return total;                    // Time O(n), Space O(1)
}
```

**Nested loops, independent.** Multiply.

```js
for (let i = 0; i < n; i++)
  for (let j = 0; j < m; j++) work(); // O(n * m)
```

**Nested loops, dependent.** Add up the real counts. When the inner loop starts at `i + 1`:

```
i = 0 -> n-1 steps
i = 1 -> n-2 steps
...
i = n-1 -> 0 steps
Total = (n-1) + (n-2) + ... + 0 = n(n-1)/2  -> O(n^2)
```

The "1 + 2 + ... + n = n(n+1)/2" sum shows up everywhere. It is O(n^2), not O(n).

**Loop that multiplies or divides.** If the counter doubles (`i *= 2`) or halves (`i = Math.floor(i / 2)`), the loop runs about `log2(n)` times. Reason: you ask "how many times can I double 1 before reaching n?" and the answer is log2 n.

**A geometric sum is cheap.** `n + n/2 + n/4 + ... + 1` is less than `2n`, so it is O(n). This catches many people: a loop that halves `n` each round with a linear inner pass is O(n) in total, not O(n log n).

### 5. Counting recursion

Draw the **call tree**. Then:

```
Total time  = (number of calls) * (work done per call, not counting the recursive calls)
Stack space = (deepest chain of calls) * (memory per frame)
```

Key point: the recursion **stack space is the depth of the tree, not the number of nodes**. Calls that already returned have freed their frames.

Three shapes you must recognize:

| Shape | Example | Calls | Depth | Time | Stack space |
|---|---|---|---|---|---|
| One call, shrink by 1 | `f(n) = f(n-1) + 1` | n | n | O(n) | O(n) |
| One call, halve | binary search | log n | log n | O(log n) | O(log n) |
| Two calls, shrink by 1 | naive `fib(n-1) + fib(n-2)` | about 2^n | n | O(2^n) | O(n) |

Naive Fibonacci is the classic trap: the time is exponential but the stack space is only O(n), because at any moment only one root-to-leaf path is on the stack.

```js
function fib(n) {              // Time O(2^n) (really O(phi^n), phi ~ 1.618), Space O(n)
  return n < 2 ? n : fib(n - 1) + fib(n - 2);
}
```

Calls for `fib(5)` (the tree repeats work, which is why memoization fixes it):

```
                fib(5)
              /        \
         fib(4)        fib(3)
        /     \        /    \
    fib(3)  fib(2)  fib(2) fib(1)
    ...
```

With memoization every `fib(k)` is computed once: Time O(n), Space O(n).

JavaScript note: Node's default stack allows roughly 10,000 to 12,000 frames for simple functions. A recursion of depth 10^5 can throw `RangeError: Maximum call stack size exceeded`. That is a real reason to prefer a loop or an explicit stack for deep inputs.

### 6. Recurrence relations and the Master theorem (plain language)

A recurrence writes the cost of a problem in terms of smaller copies of itself.

```
T(n) = a * T(n / b) + f(n)
```

Read it like this: "To solve size `n`, I make **a** smaller problems, each of size **n/b**, and I pay **f(n)** extra to split and combine."

| Algorithm | Recurrence | Why |
|---|---|---|
| Binary search | `T(n) = T(n/2) + O(1)` | 1 half, constant work |
| Merge sort | `T(n) = 2T(n/2) + O(n)` | 2 halves, linear merge |
| Tree traversal (balanced) | `T(n) = 2T(n/2) + O(1)` | 2 halves, constant work |
| Naive Fibonacci | `T(n) = T(n-1) + T(n-2) + O(1)` | not the master form (shrinks by subtraction) |
| Linear recursion | `T(n) = T(n-1) + O(1)` | one call, shrink by 1 |
| Quick sort (good pivot) | `T(n) = 2T(n/2) + O(n)` | same as merge sort |

**The Master theorem, in words.** Compare two forces:

- the **leaves**: the pile of tiny subproblems at the bottom, about `n^(log_b a)` of them;
- the **work per level at the top**: `f(n)`.

Then there are three outcomes (assuming `f(n)` is a normal polynomial like `n^k`):

| Who wins | Condition (with `f(n) = n^k`) | Result | Picture |
|---|---|---|---|
| **Leaves win** (bottom is heavy) | `k < log_b a` | `T(n) = Theta(n^(log_b a))` | work grows as you go down |
| **Tie** (every level costs the same) | `k = log_b a` | `T(n) = Theta(n^k * log n)` | `log n` levels, same cost each |
| **Top wins** (root is heavy) | `k > log_b a` | `T(n) = Theta(f(n))` | work shrinks as you go down |

(The "top wins" case also needs the combine work to shrink sensibly, which is true for ordinary polynomials.)

Worked examples:

- **Merge sort:** `a = 2, b = 2, f = n`. `log_2 2 = 1`, and `k = 1`. Tie, so `Theta(n log n)`.
- **Binary search:** `a = 1, b = 2, f = 1 = n^0`. `log_2 1 = 0`, and `k = 0`. Tie, so `Theta(log n)`.
- **Tree traversal:** `a = 2, b = 2, f = 1`. `log_2 2 = 1 > k = 0`. Leaves win, so `Theta(n)`. (You visit each node once. Makes sense.)
- **`T(n) = T(n/2) + n`:** `a = 1, b = 2, k = 1`. `log_2 1 = 0 < 1`. Top wins, so `Theta(n)`. This is the geometric sum `n + n/2 + n/4 + ...`.
- **`T(n) = 4T(n/2) + n`:** `log_2 4 = 2 > 1`. Leaves win, so `Theta(n^2)`.

**Cheat shortcut that works without the theorem:** draw the tree. Write the work per level. If the levels are equal, multiply by the number of levels (`log n`). If they grow geometrically, the last level dominates. If they shrink geometrically, the first level dominates.

**When the master theorem does not apply:** for `T(n) = T(n-1) + ...` or `T(n) = T(n-1) + T(n-2)`, the input shrinks by subtraction. Unroll instead:

```
T(n) = T(n-1) + n
     = T(n-2) + (n-1) + n
     = ...
     = 1 + 2 + ... + n = O(n^2)
```

### 7. Best, average and worst case

The same code can behave differently on different inputs.

| Algorithm | Best | Average | Worst | Worst-case input |
|---|---|---|---|---|
| Linear search | O(1) | O(n) | O(n) | target is last or missing |
| Binary search | O(1) | O(log n) | O(log n) | target is missing |
| Insertion sort | O(n) | O(n^2) | O(n^2) | reverse-sorted array |
| Quick sort (random pivot) | O(n log n) | O(n log n) | O(n^2) | pivot is always the min or max |
| Merge sort | O(n log n) | O(n log n) | O(n log n) | none, always the same |
| Hash map lookup | O(1) | O(1) | O(n) | all keys collide (rare) |

Unless you are told otherwise, **state the worst case**. Mention average or best when it changes the story (quick sort, hash maps).

### 8. Amortized analysis

Some operations are usually cheap and sometimes very expensive. **Amortized cost** is the average cost per operation over a **long sequence of operations**, even in the worst case. It is not about random inputs. It is a guarantee about the total.

Analogy: you pay a gym a yearly fee of 1200. Some months it feels like 1200 at once, but the "amortized" cost is 100 per month.

#### Dynamic array (`push`)

An array has a capacity. When it is full, we allocate one of **double** the size and copy everything.

```
capacity: 1  2  4  8  16 ...
a push that triggers a resize copies: 1, 2, 4, 8, 16 ... items
```

Push `n` items. The copy costs total `1 + 2 + 4 + ... + n/2 < n`. The normal pushes cost `n`. So the total is under `2n`, and the cost per push is `O(1)` **amortized**, even though one single push can cost O(n).

I counted this in Node for `n = 65,536` pushes with doubling: 65,535 total element copies, which is just under `n`.

```js
class DynArray {
  constructor() { this.cap = 1; this.n = 0; this.a = new Array(1); }
  push(x) {
    if (this.n === this.cap) {          // full: double and copy, O(n) but rare
      const b = new Array(this.cap *= 2);
      for (let i = 0; i < this.n; i++) b[i] = this.a[i];
      this.a = b;
    }
    this.a[this.n++] = x;               // normal case, O(1)
  }
}
```

Why doubling and not "grow by 10"? Growing by a fixed amount means a resize every 10 pushes, each copying about `n` items, which gives O(n^2) total. **Growing by a constant factor** is what makes it amortized O(1). (JavaScript arrays handle this for you, and `arr.push` is amortized O(1).)

#### Queue from two stacks

Build a queue with two stacks: push new items on `inbox`. To dequeue, if `outbox` is empty, pour all of `inbox` into `outbox` (this reverses the order), then pop from `outbox`.

```js
class Queue2Stacks {
  constructor() { this.inbox = []; this.outbox = []; }
  enqueue(x) { this.inbox.push(x); }            // O(1)
  dequeue() {
    if (!this.outbox.length)                     // only pour when outbox is empty
      while (this.inbox.length) this.outbox.push(this.inbox.pop());
    return this.outbox.pop();                    // undefined if the queue is empty
  }
}
```

One `dequeue` can cost O(n) (the pour). But **each element is moved from `inbox` to `outbox` at most once** in its life. So over `m` operations the total number of moves is at most `m`. Amortized cost per operation is O(1).

Two ways to prove amortized bounds (you only need the idea):

- **Aggregate method:** total cost of `m` operations divided by `m`. (What we did above.)
- **Accounting method:** charge each operation a bit extra "credit" that pays for the future expensive step. Each enqueue is charged 2: one for pushing, one saved to pay for the later move to `outbox`.

Careful: **amortized is not average-case.** Average-case needs a probability over random inputs. Amortized is a worst-case guarantee over a sequence. Also, an individual operation can still be slow, which matters for real-time systems.

### 9. Space complexity quick guide

| What you allocate | Space |
|---|---|
| A few variables | O(1) |
| A new array/string/set/map of size up to `n` | O(n) |
| A 2D table `n x m` | O(n * m) |
| Recursion of depth `d` | O(d) |
| `arr.slice()` or `[...arr]` inside a loop | O(n) per call, easy to miss |
| Sorting in place | O(1) to O(log n) extra (depends on the algorithm) |

Things people forget when counting space:

- The recursion stack counts. A "no extra memory" recursive DFS on a path-shaped tree still uses O(n) stack.
- Building strings with `+=` in a loop can cost O(n^2) total if each concatenation copies. Collecting into an array and calling `join` is O(n).
- `Array.prototype.sort` in V8 is not constant-space: do not claim O(1) space for a solution that sorts a copy. Say "O(n) for the copy, plus the sort's own working memory".
- Methods like `shift`, `unshift`, `splice`, `slice`, `concat`, `indexOf`, `includes` and the spread operator all hide an O(n) cost. Using `arr.shift()` as a queue in a loop is quadratic for big arrays.

### 10. A 60-second method for any snippet

1. Name the input size(s): `n`, `m`, `V`, `E`, string length `L`.
2. Find the loops and recursive calls. Is each one `n`, `log n`, or a dependent count?
3. **Multiply** nested parts, **add** sequential parts.
4. Look for hidden costs in library calls (`sort`, `slice`, `includes`, `join`, string concat, spread).
5. Keep the biggest term, drop constants.
6. Do the same for memory, and do not forget the recursion stack.
7. Say which case (worst by default).

## Worked Examples

### Example A: two loops, one after another

```js
function f(arr) {
  for (const x of arr) console.log(x);       // n
  for (let i = 0; i < arr.length; i++)
    for (let j = 0; j < arr.length; j++)
      console.log(arr[i], arr[j]);           // n * n
}
```

Cost: `n + n^2`. Drop the small term: **O(n^2)** time, **O(1)** space.

### Example B: an early exit

```js
function hasDup(arr) {
  for (let i = 0; i < arr.length; i++)
    for (let j = i + 1; j < arr.length; j++)
      if (arr[i] === arr[j]) return true;
  return false;
}
```

Best case O(1) (first two equal). Worst case `n(n-1)/2`, so **O(n^2)** time, O(1) space. We say O(n^2) because worst case is the default. A hash set makes it O(n) time, O(n) space: that is the trade.

### Example C: two different inputs

```js
function common(a, b) {          // a has size p, b has size q
  const set = new Set(a);        // O(p) time, O(p) space
  return b.filter(x => set.has(x)); // O(q) time, O(q) output space worst case
}
```

Time **O(p + q)**, space **O(p + q)**. Writing O(n) here would hide that there are two sizes.

### Example D: a loop whose bound depends on the loop variable

```js
for (let i = 1; i <= n; i *= 2)
  for (let j = 0; j < i; j++) work();
```

Inner counts: `1 + 2 + 4 + ... + n` which is below `2n`. Total **O(n)**, not O(n log n). The outer loop runs log n times, but most of the work is in the last few rounds.

## Exercises: "What is the complexity?"

Cover the answers and try each one. State time and space.

### Exercise 1

```js
function a(n) { let c = 0; for (let i = 0; i < n; i++) c++; return c; }
```

### Exercise 2

```js
function b(n) { for (let i = 0; i < n; i++) for (let j = 0; j < n; j++) work(); }
```

### Exercise 3

```js
function c(n) { for (let i = 0; i < n; i++) for (let j = i + 1; j < n; j++) work(); }
```

### Exercise 4

```js
function d(n) { for (let i = 1; i < n; i *= 2) work(); }
```

### Exercise 5

```js
function e(n) { for (let i = 0; i < n; i++) for (let j = 1; j < n; j *= 2) work(); }
```

### Exercise 6

```js
function f(a, b) { for (const x of a) work(x); for (const y of b) work(y); }
// a has size p, b has size q
```

### Exercise 7

```js
function g(n) { return n < 2 ? n : g(n - 1) + g(n - 2); }
```

### Exercise 8

```js
function h(n) { return n === 0 ? 0 : n + h(n - 1); }
```

### Exercise 9

```js
function i(arr, lo, hi, t) {
  if (lo > hi) return -1;
  const mid = (lo + hi) >> 1;
  if (arr[mid] === t) return mid;
  return arr[mid] < t ? i(arr, mid + 1, hi, t) : i(arr, lo, mid - 1, t);
}
```

### Exercise 10

```js
function j(n) { while (n > 0) { for (let k = 0; k < n; k++) work(); n = Math.floor(n / 2); } }
```

### Exercise 11

```js
function k(arr) { for (let i = 0; i < arr.length; i++) arr.slice().sort((x, y) => x - y); }
```

### Exercise 12

```js
function l(n) { const memo = new Map(); const f = x => x < 2 ? x : memo.get(x) ?? (memo.set(x, f(x - 1) + f(x - 2)), memo.get(x)); return f(n); }
```

### Answers

| # | Time | Space | Reason |
|---|---|---|---|
| 1 | O(n) | O(1) | one loop |
| 2 | O(n^2) | O(1) | two independent nested loops, `n * n` |
| 3 | O(n^2) | O(1) | `(n-1) + (n-2) + ... + 0 = n(n-1)/2` |
| 4 | O(log n) | O(1) | `i` doubles, so about `log2 n` rounds |
| 5 | O(n log n) | O(1) | outer `n`, inner `log n`, multiplied |
| 6 | O(p + q) | O(1) | sequential loops add, different inputs get different letters |
| 7 | O(2^n) (tight: O(phi^n), about 1.618^n) | O(n) | two calls shrinking by 1; the stack is only as deep as `n` |
| 8 | O(n) | O(n) | one call shrinking by 1; `n` frames on the stack |
| 9 | O(log n) | O(log n) | halves each call; the recursion depth is `log n` (a loop version is O(1) space) |
| 10 | O(n) | O(1) | `n + n/2 + n/4 + ...` is below `2n`, a geometric sum |
| 11 | O(n^2 log n) | O(n) | `n` times: copy `O(n)` plus sort `O(n log n)`, and each copy is freed after use, so space is O(n) |
| 12 | O(n) | O(n) | memoized Fibonacci: each `x` computed once; memo map and the stack are both `n` |

## Interview Questions and Answers

**Q1. What is the difference between O(n), Omega(n) and Theta(n)?**
O(n) means "at most linear", Omega(n) means "at least linear", Theta(n) means both, so it is exactly linear growth. In an interview "O" is used loosely to mean the tight bound.

**Q2. Quick sort is O(n^2) in the worst case. Why do we still use it?**
The worst case needs a very bad pivot every time (for example, an already-sorted array with the first element as pivot). With a random or median-of-three pivot, the expected time is O(n log n), the constant factors are small, and it sorts in place. So the average case is what matters in practice.

**Q3. Is `arr.push(x)` O(1)?**
Amortized O(1). Sometimes the underlying storage must grow and copy O(n) elements, but because the capacity grows by a constant factor, the total over `n` pushes is O(n).

**Q4. What is the time and space of recursive merge sort?**
Time `T(n) = 2T(n/2) + O(n) = O(n log n)` in all cases. Space is O(n) for the merge buffer plus O(log n) for the stack, so O(n).

**Q5. Why is the complexity of "for each element, call `includes` on an array" O(n^2)?**
`includes` scans the array, which is O(n), and we call it `n` times. Switching to a `Set` makes each check O(1) on average and the total O(n).

**Q6. What is the difference between amortized and average-case?**
Average-case uses a probability model over inputs. Amortized is a guarantee for any sequence of operations, so it holds in the worst case too. It does not say any single operation is cheap.

**Q7. Can an O(n) algorithm be slower than an O(n^2) one?**
Yes for small `n`, because Big-O hides constants and lower terms. 1000n beats n^2 only when `n > 1000`. Big-O is about how it scales, not about speed on a specific input.

## Common Mistakes

- **"Two nested loops is always O(n^2)."** Not always. Check the bounds. If the inner loop runs `i` times with `i` doubling, the total can be O(n). If the inner loop is `log n`, it is O(n log n).
- **"Recursion has no space cost."** Every active call holds a frame. Depth `d` costs O(d) space.
- **"Naive Fibonacci uses exponential space."** Time is exponential. Stack space is only O(n).
- **"I call `slice`, `shift`, `includes` or spread, so it is O(1)."** They are O(n) and they hide inside loops, turning O(n) code into O(n^2).
- **"Drop everything except `n`."** Two inputs need two letters: O(a + b), O(rows * cols), O(V + E).
- **"Amortized O(1) means every call is O(1)."** One call can be O(n). Only the total over many calls is bounded.
- **"O(2n) is worse than O(n)."** They are the same class. Constants are dropped. (They may differ in real speed, but not in Big-O.)
- **"Hash map is always O(1)."** It is O(1) on average. Worst case with many collisions is O(n). Hashing a long string key also costs O(length of key).
- **"Input size is `n` for numbers too."** For a number `N`, a loop up to `N` is O(N) in the value, but the input only has `log N` digits. Interviewers usually use the value, but be clear which one you mean (this matters in prime checking and in digit problems).
- **Forgetting the output size.** Generating all subsets takes at least O(2^n) because the output has that many items.

## Interview Tips

- **State your complexity unprompted** after you describe each approach: "This is O(n log n) time, O(n) space because of the extra array."
- **Say the case you are measuring.** "Worst case O(n^2), average O(n log n)."
- **Name your variables.** "Let `n` be the length of the array and `k` the window size, so the time is O(n), not O(n * k)."
- **Justify, do not just announce.** Show the loop count or the recurrence in one sentence: "Each element is pushed and popped at most once, so the total is O(n)."
- **Use the constraints** to pick the target: if `n = 10^5`, say "O(n^2) is about 10^10 steps, too slow, so I need O(n log n) or better."
- **Follow-ups to expect:** "Can you reduce the space?", "What if the input is sorted?", "What is the worst-case input for your solution?", "What is the complexity if the array has a million elements and fits in memory but not in cache?" (answer: the Big-O does not change, only the constants).
- **Edge cases to mention:** empty input, `n = 1`, all equal elements, already sorted or reverse sorted input, and very deep recursion (stack overflow in JavaScript).
- **Mention the trade-off** when you add memory to save time. It shows you understand both axes.

## Practice List

Practice by stating the complexity of each solution you write. Good problems to apply this on:

**Easy**
- Two Sum
- Contains Duplicate
- Valid Anagram
- Binary Search
- Fibonacci Number
- Climbing Stairs
- Power of Two

**Medium**
- Pow(x, n)
- Sort an Array (merge sort)
- Kth Largest Element in an Array
- Subsets
- Permutations
- Implement Queue using Stacks
- Min Stack
- Design Dynamic Array (implement your own `push`)

**Hard**
- Median of Two Sorted Arrays
- Merge k Sorted Lists
- Sliding Window Maximum
- N-Queens

## Key Takeaways

- **Big-O is about growth**, not exact seconds: drop constants and smaller terms, keep the dominant term.
- **Add sequential steps, multiply nested steps**, and use different letters for different inputs.
- **Recursion time = number of calls times work per call; recursion space = depth**, not the total number of calls.
- **Master theorem in one line:** compare the bottom (leaves) with the top (per-level work). Leaves win gives `n^(log_b a)`, a tie gives an extra `log n`, top wins gives `f(n)`.
- **Amortized O(1) is a guarantee over a sequence**: doubling a dynamic array and the two-stack queue both work because each element is moved a bounded number of times.
- **State the worst case by default**, and mention best or average when it changes the story (quick sort, hash maps).
- **Watch for hidden costs** in `slice`, `shift`, `includes`, spread, string concatenation and `sort`.

---

[← Previous: Problem Solving Framework](../01-problem-solving-framework/README.md) | [Next: JavaScript for DSA →](../03-javascript-for-dsa/README.md) | [Back to Foundations Index](../README.md)
