# Stack and Queue

> **Estimated Time:** 4 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - Arrays](../01-arrays/README.md), [04 - Strings](../04-strings/README.md), [09 - Linked List](../09-linked-list/README.md)

---

## What You'll Learn
- You can explain LIFO (stack) and FIFO (queue) and pick the right one in seconds.
- You can build a stack, a queue and a deque in JavaScript with plain arrays, and know when `shift()` is a trap.
- You can solve "matching brackets", "undo" and "nested expression" problems with a stack.
- You can use a queue for "process in order" and level-by-level BFS problems.
- You can parse and evaluate expressions (RPN, calculators, infix to postfix).

## The Big Idea
Think of a stack of plates in a cafeteria. You add a plate on top and take a plate from the top. The last plate you put down is the first one you pick up. That is a **stack**: Last In, First Out (LIFO). Now think of a line at a ticket counter. The person who came first is served first. That is a **queue**: First In, First Out (FIFO).

Why do these two matter? Many problems have a "most recent unfinished thing" (open bracket, last number, last page you visited). A stack gives you exactly that thing in O(1). Other problems have "oldest thing first" (requests in time order, nodes to visit in BFS). A queue gives you that in O(1).

Interviewers love these because:
- The idea is simple, but the problems hide it. "Is this string valid?" is secretly a stack problem.
- They test whether you can design a data structure out of another one (queue from two stacks).
- They are the base for harder topics: monotonic stack, BFS, expression parsing, DFS without recursion.

A **deque** (double-ended queue) lets you add and remove at both ends. It is the base for the sliding-window maximum trick (see [11 - Monotonic Stack and Queue](../11-monotonic-stack-queue/README.md)).

## How to Spot It
| If the problem says ... | think ... |
|---|---|
| "valid parentheses", "balanced", "matching pairs" | Stack of open brackets |
| "undo", "go back", "most recent", "nested" | Stack |
| "decode", "expression with brackets", "evaluate" | Stack (or two stacks: numbers and operators) |
| "remove adjacent duplicates", "collisions" | Stack, compare with the top |
| "remove k digits to make smallest" | Stack that pops while the top is worse |
| "next greater / next smaller element", "daily temperatures" | Monotonic stack (chapter 11) |
| "process in arrival order", "last 3000 ms", "first come first served" | Queue |
| "minimum number of steps/minutes", "spread level by level" | Queue + BFS |
| "implement X using Y" | Think about moving items between two structures |
| "max in every window of size k" | Deque (chapter 11) |

## Core Concepts / Template

### Stack in JavaScript
A JS array is a fine stack. `push` and `pop` work at the end and both are O(1).

```js
const st = [];
st.push(1); st.push(2);
st[st.length - 1]; // peek -> 2
st.pop();          // 2
st.length === 0;   // isEmpty
```

### Queue in JavaScript: the `shift()` trap
`arr.push(x)` is O(1), but `arr.shift()` has to move every other element, so it can be O(n). Engines have tricks that make small arrays fast, but you should not rely on that and you should not say "shift is O(1)" in an interview. The simple fix is a **head pointer**: do not remove, just move the pointer forward.

```js
const q = []; let head = 0;
q.push(5); q.push(7);        // enqueue
const front = q[head++];     // dequeue -> 5 (O(1))
const size = q.length - head;
const isEmpty = head === q.length;
```

The cost is memory: dequeued items stay in the array until the end. For a bounded problem that is fine. If you need a truly long-lived queue, use a circular buffer (problem 16) or a linked list.

### Deque
A deque needs removal at both ends. With an array, `pop` is O(1) but `shift` is not. In interviews you can write a small class with two indexes (or a doubly linked list). JavaScript has no built-in deque, so say so, and say you would use head and tail indexes.

```js
class Deque {
  constructor() { this.a = {}; this.h = 0; this.t = 0; } // [h, t) holds items
  pushBack(x) { this.a[this.t++] = x; }
  pushFront(x) { this.a[--this.h] = x; }
  popBack() { if (this.h === this.t) return undefined; const x = this.a[--this.t]; delete this.a[this.t]; return x; }
  popFront() { if (this.h === this.t) return undefined; const x = this.a[this.h]; delete this.a[this.h++]; return x; }
  get size() { return this.t - this.h; }
}
```

All four operations are O(1). (An object with integer keys is used so negative indexes work for `pushFront`.)

### The three stack patterns
1. **Match and cancel.** Push openers. When a closer arrives, it must match the top. (Parentheses, asteroids, adjacent duplicates.)
2. **Defer work.** Push things you cannot finish yet (a number waiting for its multiplication, a string waiting for its `k[...]`). Finish them when you see the closing signal.
3. **Keep a best-so-far order.** Pop while the top is worse than the new item. (Remove k digits, next greater element.)

### Step-by-step trace: stack template on `"{[]}"`
| char | action | stack after |
|---|---|---|
| `{` | push | `{` |
| `[` | push | `{ [` |
| `]` | top is `[`, match, pop | `{` |
| `}` | top is `{`, match, pop | empty |

Empty at the end means valid.

### Queue template (BFS, level by level)
```js
// Process one level at a time: freeze the size before the inner loop.
const q = [start]; let head = 0, steps = 0;
while (head < q.length) {
  const end = q.length;          // everything up to here is one level
  while (head < end) {
    const cur = q[head++];
    // for each neighbour not seen yet: q.push(neighbour)
  }
  steps++;
}
```

Problems 1 to 20 below use these templates. Daily Temperatures (next warmer day) is a stack problem too, but it is the first problem of the monotonic stack pattern, so it lives in [11 - Monotonic Stack and Queue](../11-monotonic-stack-queue/README.md).

## Problems (easy -> hard)

### 1. Valid Parentheses — Easy
**Companies:** Amazon, Google, Meta, Microsoft, Bloomberg
**Problem:** Given a string of `()[]{}`, say if every bracket is closed by the same type in the right order. `"()[]{}"` -> true, `"(]"` -> false.
**Intuition:** The bracket that opened last must close first. That is exactly "last in, first out".
**Approach 1 - Brute force:** Keep deleting `()`, `[]`, `{}` from the string until nothing changes. Time O(n^2), Space O(n).
**Approach 2 - Optimal:** Scan once. Push openers. For a closer, pop and check that it matches. At the end the stack must be empty.

| char | stack |
|---|---|
| `(` | `(` |
| `[` | `( [` |
| `]` | `(` (matched) |
| `)` | empty |

```js
function isValid(s) {
  const pair = { ')': '(', ']': '[', '}': '{' };
  const st = [];
  for (const c of s) {
    if (c in pair) { if (st.pop() !== pair[c]) return false; }
    else st.push(c);
  }
  return st.length === 0;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** A closer on an empty stack: `pop()` gives `undefined`, which never equals a bracket, so it returns false. Also check the stack is empty at the end (`"(("`). Odd length can exit early.

### 2. Implement Queue using Stacks — Easy
**Companies:** Amazon, Microsoft, Bloomberg, Apple
**Problem:** Build a FIFO queue with only stack operations: `push`, `pop`, `peek`, `empty`.
**Intuition:** Reversing a stack once turns LIFO into FIFO. Keep two stacks: one to receive, one to serve. Only refill the serving stack when it is empty.
**Approach 1 - Brute force:** On every `push`, move everything to a helper stack, add the item, move everything back. `push` is O(n), others O(1).
**Approach 2 - Optimal (lazy two stacks):** `push` goes to `inS`. For `pop`/`peek`, if `outS` is empty, pour all of `inS` into `outS` (this reverses the order). Then use the top of `outS`. Each item moves at most twice in its life, so the cost is **amortized O(1)**.

Trace: push 1, push 2, peek, push 3, pop, pop.
| op | inS | outS | result |
|---|---|---|---|
| push 1, push 2 | [1,2] | [] | |
| peek | [] | [2,1] | 1 |
| push 3 | [3] | [2,1] | |
| pop | [3] | [2] | 1 |
| pop | [3] | [] | 2 |

```js
class MyQueue {
  constructor() { this.inS = []; this.outS = []; }
  push(x) { this.inS.push(x); }
  _move() { if (!this.outS.length) while (this.inS.length) this.outS.push(this.inS.pop()); }
  pop() { this._move(); return this.outS.pop(); }
  peek() { this._move(); return this.outS[this.outS.length - 1]; }
  empty() { return !this.inS.length && !this.outS.length; }
}
```

**Complexity:** Time O(1) amortized per operation (worst single `pop` is O(n)), Space O(n).
**Watch out:** Never pour `inS` into `outS` while `outS` still has items; that breaks the order. Say "amortized" out loud.

### 3. Implement Stack using Queues — Easy
**Companies:** Amazon, Microsoft, Bloomberg
**Problem:** Build a LIFO stack with only queue operations.
**Intuition:** After adding a new item at the back, rotate the older items behind it. Then the newest item is at the front, which is the stack's top.
**Approach 1 - Two queues:** Move everything to a second queue on each push. Time O(n) push.
**Approach 2 - Optimal (one queue):** `push(x)`: add `x`, then take the `size - 1` older items from the front and add them to the back. `pop` and `top` just read the front. Push is O(n), pop and top are O(1). (You can also make push O(1) and pop O(n); one of them must pay.)

Trace: push 1, push 2, push 3 (queue front on the left).
| push | after add | after rotate |
|---|---|---|
| 1 | [1] | [1] |
| 2 | [1,2] | [2,1] |
| 3 | [2,1,3] | [3,2,1] |

```js
class MyStack {
  constructor() { this.q = []; this.head = 0; }
  push(x) {
    this.q.push(x);
    for (let i = this.q.length - this.head - 1; i > 0; i--) this.q.push(this.q[this.head++]);
  }
  pop() { return this.q[this.head++]; }
  top() { return this.q[this.head]; }
  empty() { return this.head === this.q.length; }
}
```

**Complexity:** Time push O(n), pop/top O(1), Space O(n).
**Watch out:** `head` is the queue pointer from the template. Rotating means `size - 1` moves, not `size`. Being honest: with one queue you cannot make both push and pop O(1).

### 4. Backspace String Compare — Easy
**Companies:** Google, Amazon, Meta, Microsoft
**Problem:** `#` means backspace. Check whether two typed strings give the same text. `"ab#c"` and `"ad#c"` -> true (both `"ac"`).
**Intuition:** Typing is a stack: letters push, `#` pops (and popping an empty stack is allowed).
**Approach 1 - Brute force / stack:** Build the final text for each string with a stack and compare. Time O(n + m), Space O(n + m).
**Approach 2 - Optimal:** Two pointers from the end of both strings. Going backwards, a `#` tells you how many letters to skip, so you never need to store anything. Find the next real letter in each string and compare. Space O(1).

Trace for `s = "ab#c"` backwards: `c` is real. Next, `#` means skip 1, so `b` is skipped, then `a` is real. Real letters: `c`, `a`. Same for `"ad#c"`.

```js
function backspaceCompare(s, t) {
  const build = (x) => { const st = []; for (const c of x) c === '#' ? st.pop() : st.push(c); return st.join(''); };
  return build(s) === build(t);
}

// O(1) space follow-up
function backspaceCompareO1(s, t) {
  const next = (x, i) => { let skip = 0; while (i >= 0) {
    if (x[i] === '#') { skip++; i--; } else if (skip > 0) { skip--; i--; } else break; } return i; };
  let i = s.length - 1, j = t.length - 1;
  while (true) {
    i = next(s, i); j = next(t, j);
    if (i < 0 || j < 0) return i < 0 && j < 0;
    if (s[i] !== t[j]) return false;
    i--; j--;
  }
}
```

**Complexity:** Stack version Time O(n + m), Space O(n + m). Two-pointer version Time O(n + m), Space O(1).
**Watch out:** `"a##c"` vs `"#a#c"` -> true (backspace on empty text does nothing). In the pointer version, when one string runs out, the other must run out too.

### 5. Number of Recent Calls — Easy
**Companies:** Amazon, Google, Microsoft
**Problem:** `ping(t)` records a request at time `t` (always increasing) and returns how many requests happened in `[t - 3000, t]`. Calls: 1, 100, 3001, 3002 -> 1, 2, 3, 3.
**Intuition:** Requests arrive in time order, and old ones expire in the same order. That is a queue: add at the back, drop expired ones from the front.
**Approach 1 - Brute force:** Store all times, count those in range every call. Time O(n) per ping.
**Approach 2 - Optimal:** Queue with a head pointer. After adding `t`, move `head` past every time `< t - 3000`. The answer is `length - head`. Each time is added once and removed once, so O(1) amortized.

Trace: ping(3001): queue `[1,100,3001]`, cutoff `3001 - 3000 = 1`; `1` is not `< 1`, so keep it. Answer 3. ping(3002): cutoff 2, drop `1`. Answer 3.

```js
class RecentCounter {
  constructor() { this.q = []; this.h = 0; }
  ping(t) { this.q.push(t); while (this.q[this.h] < t - 3000) this.h++; return this.q.length - this.h; }
}
```

**Complexity:** Time O(1) amortized per ping, Space O(w) in a real queue (here the array keeps old items; trim it if memory matters).
**Watch out:** The range is inclusive on both ends, so only strictly older times are dropped.

### 6. Generate Binary Numbers 1 to N — Easy
**Companies:** Amazon, Microsoft, Adobe
**Problem:** Return the first `n` positive integers as binary strings. `n = 5` -> `["1","10","11","100","101"]`.
**Intuition:** Every binary number `x` leads to two next numbers: `x0` and `x1`. Taking them in that order, level by level, gives the numbers in increasing order. That is BFS on an imaginary tree, so use a queue.
**Approach 1 - Brute force:** Convert each integer with `toString(2)`. Time O(n log n), Space O(n). Fine in practice, but the queue shows the pattern.
**Approach 2 - Optimal:** Start the queue with `"1"`. Pop `cur`, output it, push `cur + "0"` and `cur + "1"`.

Trace:
| pop | output so far | queue after |
|---|---|---|
| 1 | 1 | 10, 11 |
| 10 | 1, 10 | 11, 100, 101 |
| 11 | 1, 10, 11 | 100, 101, 110, 111 |

```js
function generateBinary(n) {
  const out = [], q = ['1']; let h = 0;
  while (out.length < n) {
    const cur = q[h++]; out.push(cur);
    q.push(cur + '0', cur + '1');
  }
  return out;
}
```

**Complexity:** Time O(n log n) counting string building (each string has up to log n characters), Space O(n log n).
**Watch out:** Stop when you have `n` outputs, not when the queue is empty (it never is).

### 7. Remove Outermost Parentheses — Easy
**Companies:** Amazon, Google, Microsoft
**Problem:** A valid string splits into "primitive" pieces (a balanced piece that cannot be split further). Remove the outer pair of each piece. `"(()())(())"` -> `"()()()"`.
**Intuition:** The outer `(` of a piece is the one that takes depth from 0 to 1. The outer `)` is the one that brings depth back to 0. You do not need a real stack, a counter is enough.
**Approach 1 - Stack:** Push on `(`, pop on `)`, and mark pieces. More code than needed. Time O(n), Space O(n).
**Approach 2 - Optimal (depth counter):** For `(`: keep it only if depth is already above 0, then depth++. For `)`: depth-- first, then keep it only if depth is still above 0.

Trace on `(()())`:
| char | depth before | keep? | depth after |
|---|---|---|---|
| `(` | 0 | no | 1 |
| `(` | 1 | yes | 2 |
| `)` | 2 | yes | 1 |
| `(` | 1 | yes | 2 |
| `)` | 2 | yes | 1 |
| `)` | 1 | no | 0 |

```js
function removeOuterParentheses(s) {
  let depth = 0, res = '';
  for (const c of s) {
    if (c === '(') { if (depth > 0) res += c; depth++; }
    else { depth--; if (depth > 0) res += c; }
  }
  return res;
}
```

**Complexity:** Time O(n), Space O(1) extra (output aside; building by `+=` is fine in JS, or use an array and `join`).
**Watch out:** Order matters: for `)` decrement before checking, for `(` check before incrementing. This "counter instead of stack" trick works whenever there is only one bracket type.

### 8. Min Stack — Medium
**Companies:** Amazon, Google, Meta, Microsoft, Bloomberg
**Problem:** A stack with `push`, `pop`, `top` and `getMin`, all O(1).
**Intuition:** The minimum at any moment depends only on what is below. So when you push an item, remember "the minimum of everything up to here". When you pop, that memory disappears together with the item.
**Approach 1 - Brute force:** `getMin` scans the stack. O(n).
**Approach 2 - Optimal:** Store `[value, minSoFar]` pairs. `minSoFar = min(value, previous minSoFar)`. (A second stack of minimums works the same way.)

Trace: push -2, push 0, push -3, getMin, pop, top, getMin.
| op | stack of [value, min] | result |
|---|---|---|
| push -2 | [-2,-2] | |
| push 0 | [-2,-2] [0,-2] | |
| push -3 | [-2,-2] [0,-2] [-3,-3] | |
| getMin | | -3 |
| pop | [-2,-2] [0,-2] | |
| top / getMin | | 0 / -2 |

```js
class MinStack {
  constructor() { this.st = []; } // pairs [value, minSoFar]
  push(x) { const m = this.st.length ? Math.min(x, this.st[this.st.length - 1][1]) : x; this.st.push([x, m]); }
  pop() { this.st.pop(); }
  top() { return this.st[this.st.length - 1][0]; }
  getMin() { return this.st[this.st.length - 1][1]; }
}
```

**Complexity:** Time O(1) per operation, Space O(n).
**Watch out:** Do not keep a single `min` variable: after popping the minimum you would not know the next one. Follow-up: a version with O(1) extra space stores `value - min` encoded differences. It is clever but fragile; mention it, do not start with it.

### 9. Evaluate Reverse Polish Notation — Medium
**Companies:** Amazon, Google, LinkedIn, Microsoft, Bloomberg
**Problem:** Evaluate a postfix expression such as `["2","1","+","3","*"]` -> `(2 + 1) * 3 = 9`. Division truncates toward zero.
**Intuition:** In postfix, an operator always acts on the two most recent numbers. "Most recent" means stack.
**Approach 1 - Brute force:** Repeatedly find the first operator in the list, compute with the two items before it, replace the three items by the result. Time O(n^2).
**Approach 2 - Optimal:** Push numbers. For an operator, pop `b` first, then `a`, push `a op b`.

Trace on `["4","13","5","/","+"]`:
| token | stack |
|---|---|
| 4 | 4 |
| 13 | 4 13 |
| 5 | 4 13 5 |
| / | 4 2 (13/5 truncated) |
| + | 6 |

```js
function evalRPN(tokens) {
  const st = [];
  for (const t of tokens) {
    if (t === '+' || t === '-' || t === '*' || t === '/') {
      const b = st.pop(), a = st.pop();
      st.push(t === '+' ? a + b : t === '-' ? a - b : t === '*' ? a * b : Math.trunc(a / b));
    } else st.push(Number(t));
  }
  return st[0];
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Pop order: the first pop is the right operand `b`. Wrong order breaks `-` and `/`. Use `Math.trunc`, not `Math.floor` (`-7/2` must be `-3`, not `-4`). Negative numbers like `"-11"` are tokens, so check for the four operator strings, not the first character.

### 10. Simplify Path — Medium
**Companies:** Meta, Google, Microsoft, Amazon
**Problem:** Convert a Unix absolute path to its canonical form. `"/a/./b/../../c/"` -> `"/c"`.
**Intuition:** Each folder name goes deeper, `..` goes back one level (undo), `.` and empty parts do nothing. Going back is a pop.
**Approach 1 - Brute force:** Repeatedly rewrite `//` and `/./` and `/x/../` with string replace until nothing changes. Messy and slow.
**Approach 2 - Optimal:** Split on `/`. Skip `""` and `"."`. For `".."` pop (if not empty). Otherwise push. Join with `/` and add a leading `/`.

Trace on `/a/./b/../../c/` -> parts `a . b .. .. c`:
| part | stack |
|---|---|
| a | a |
| . | a |
| b | a b |
| .. | a |
| .. | (empty) |
| c | c |

```js
function simplifyPath(path) {
  const st = [];
  for (const p of path.split('/')) {
    if (p === '' || p === '.') continue;
    if (p === '..') st.pop(); else st.push(p);
  }
  return '/' + st.join('/');
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** `..` at the root stays at the root (`"/../"` -> `"/"`, popping an empty array is harmless). `"..."` is a real folder name, not special. An empty result must give `"/"`, which the leading `'/' + ''` does.

### 11. Minimum Remove to Make Valid Parentheses — Medium
**Companies:** Meta, Amazon, Microsoft, Google
**Problem:** Remove the fewest `(` or `)` so the string is valid. Letters stay. `"lee(t(c)o)de)"` -> `"lee(t(c)o)de"`.
**Intuition:** A `)` with no open `(` before it is surely bad. A `(` still waiting at the end is surely bad too. The stack holds the indexes of unmatched `(`.
**Approach 1 - Brute force:** Try removing every subset of brackets and check validity. Time O(2^n * n).
**Approach 2 - Optimal:** One pass. `(` -> push its index. `)` -> if the stack has an index, pop (matched); else mark this `)` for deletion. After the pass, every index left in the stack is an unmatched `(`; mark those too. Build the string without marked positions.

Trace on `a)b(c`:
| i | char | action |
|---|---|---|
| 1 | `)` | stack empty, delete it |
| 3 | `(` | push 3 |
| end | | stack has 3, delete it |

Result: `abc`.

```js
function minRemoveToMakeValid(s) {
  const a = s.split(''), st = [];
  for (let i = 0; i < a.length; i++) {
    if (a[i] === '(') st.push(i);
    else if (a[i] === ')') { if (st.length) st.pop(); else a[i] = ''; }
  }
  for (const i of st) a[i] = '';
  return a.join('');
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Store indexes, not characters, because you need to know where to delete. `"))(("` -> `""`. Two-pass alternative: left to right remove bad `)`, then right to left remove bad `(`, using counters only.

### 12. Asteroid Collision — Medium
**Companies:** Amazon, Meta, Google, Uber
**Problem:** Each number is an asteroid; the sign is the direction, the size is the absolute value. Same direction never collide. Opposite directions that meet: the smaller explodes, equal both explode. `[5,10,-5]` -> `[5,10]`.
**Intuition:** Only a right-moving asteroid on the stack followed by a left-moving new one can collide. The stack holds survivors so far, and the new asteroid fights the top again and again.
**Approach 1 - Brute force:** Repeatedly scan for an adjacent `(+, -)` pair, resolve it, restart. Time O(n^2).
**Approach 2 - Optimal:** For each asteroid `a`: while `a` is negative, the top is positive, and `a` is still alive, compare. Top smaller: pop it and keep fighting. Equal: pop and `a` dies. Top bigger: `a` dies. If `a` survives, push it.

Trace on `[10, 2, -5]`:
| a | fights | stack |
|---|---|---|
| 10 | none | 10 |
| 2 | none (positive) | 10 2 |
| -5 | beats 2 (pop), loses to 10 | 10 |

```js
function asteroidCollision(ast) {
  const st = [];
  for (const a of ast) {
    let alive = true;
    while (alive && a < 0 && st.length && st[st.length - 1] > 0) {
      const top = st[st.length - 1];
      if (top < -a) st.pop();
      else { if (top === -a) st.pop(); alive = false; }
    }
    if (alive) st.push(a);
  }
  return st;
}
```

**Complexity:** Time O(n) (each asteroid is pushed and popped at most once), Space O(n).
**Watch out:** `[-2,-1,1,2]` -> no collision at all (moving apart). `[1,-2,-2,-2]` -> `[-2,-2,-2]`: after the first `-2` clears the `1`, the others see an empty stack and just join. The equal case must kill both.

### 13. Remove K Digits — Medium
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** Remove exactly `k` digits from a number string to get the smallest possible number. `"1432219"`, k = 3 -> `"1219"`.
**Intuition:** To make a number small, the leftmost digits matter most. If a digit is bigger than the one right after it, deleting it helps. So scan left to right, and whenever the previous kept digit is bigger than the current one, delete the previous one.
**Approach 1 - Brute force:** Try all ways to choose `k` digits to remove. Time O(C(n,k) * n).
**Approach 2 - Optimal (greedy stack):** While `k > 0` and the top is greater than the current digit, pop and `k--`. Push the digit. At the end, if `k` is left, remove from the back (the stack is non-decreasing, so the end is the largest). Strip leading zeros; empty means `"0"`.

Trace on `1432219`, k = 3:
| digit | action | stack | k |
|---|---|---|---|
| 1 | push | 1 | 3 |
| 4 | push | 1 4 | 3 |
| 3 | pop 4 | 1 3 | 2 |
| 2 | pop 3 | 1 2 | 1 |
| 2 | push | 1 2 2 | 1 |
| 1 | pop 2 | 1 2 1 | 0 |
| 9 | push | 1 2 1 9 | 0 |

Answer `1219`.

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
**Watch out:** Three edge cases: leftover `k` (`"112"`, k = 1 -> `"11"`), leading zeros (`"10200"`, k = 1 -> `"200"`), and removing everything (`"10"`, k = 2 -> `"0"`). Do not use `Number(...)`; the string can be longer than a safe integer. This is a monotonic stack (see chapter 11).

### 14. Decode String — Medium
**Companies:** Google, Amazon, Meta, Microsoft, Bloomberg
**Problem:** Decode `k[encoded]` meaning "repeat `encoded` k times", with nesting. `"3[a2[c]]"` -> `"accaccacc"`.
**Intuition:** When you hit `[`, you start a new inner piece, so you must remember what you had built and the repeat count. When you hit `]`, finish the inner piece and glue it back. Saving "what to come back to" is stack work.
**Approach 1 - Recursion:** Parse, and on `[` recurse to build the inner string. Same logic; the call stack is the stack. Time O(output), Space O(depth).
**Approach 2 - Optimal (explicit stacks):** Keep `cur` (string being built) and `n` (number being read; multi-digit!). On `[`: push `n` and `cur`, reset both. On `]`: pop count `c` and previous string `p`; `cur = p + cur.repeat(c)`.

Trace on `3[a2[c]]`:
| char | n | cur | stacks |
|---|---|---|---|
| `3` | 3 | "" | |
| `[` | 0 | "" | nums [3], strs [""] |
| `a` | 0 | "a" | |
| `2` | 2 | "a" | |
| `[` | 0 | "" | nums [3,2], strs ["", "a"] |
| `c` | 0 | "c" | |
| `]` | | "a" + "cc" = "acc" | nums [3], strs [""] |
| `]` | | "" + "acc"x3 | empty |

```js
function decodeString(s) {
  const nums = [], strs = []; let cur = '', n = 0;
  for (const c of s) {
    if (c >= '0' && c <= '9') n = n * 10 + Number(c);
    else if (c === '[') { nums.push(n); strs.push(cur); n = 0; cur = ''; }
    else if (c === ']') { cur = strs.pop() + cur.repeat(nums.pop()); }
    else cur += c;
  }
  return cur;
}
```

**Complexity:** Time O(size of the output), Space O(depth + output).
**Watch out:** Numbers can have many digits (`"10[a]"`), so build `n = n * 10 + digit`. Reset `n` after using it. Letters outside any bracket (`"2[abc]3[cd]ef"`) go straight to `cur`.

### 15. Design Browser History — Medium
**Companies:** Google, Amazon, Meta, Microsoft
**Problem:** Start at a homepage. `visit(url)` goes to a new page and clears the forward history. `back(steps)` and `forward(steps)` move as far as they can and return the page.
**Intuition:** This is two stacks (back and forward) in disguise. One array plus a "current index" does the same job with less code: back moves the index left, forward moves it right, visit cuts everything to the right of the index.
**Approach 1 - Two stacks:** Back stack and forward stack. `back(k)` moves up to `k` pages from one to the other. Time O(k) per move.
**Approach 2 - Optimal (array + index):** `back`/`forward` just clamp the index: O(1). `visit` truncates the array with `length = i + 1` then pushes.

Trace: home, visit A, visit B, back(1), visit C.
| op | array | index | result |
|---|---|---|---|
| start | [home] | 0 | |
| visit A, B | [home,A,B] | 2 | |
| back(1) | [home,A,B] | 1 | A |
| visit C | [home,A,C] | 2 | (B is gone) |

```js
class BrowserHistory {
  constructor(home) { this.h = [home]; this.i = 0; }
  visit(url) { this.h.length = this.i + 1; this.h.push(url); this.i++; }
  back(steps) { this.i = Math.max(0, this.i - steps); return this.h[this.i]; }
  forward(steps) { this.i = Math.min(this.h.length - 1, this.i + steps); return this.h[this.i]; }
}
```

**Complexity:** Time O(1) for back/forward, O(1) amortized for visit (assigning `length` is cheap), Space O(n).
**Watch out:** Clamp with `Math.max(0, ...)` and `Math.min(len - 1, ...)`. A visit after going back must drop the forward pages.

### 16. Design Circular Queue — Medium
**Companies:** Amazon, Google, Microsoft, Apple
**Problem:** A fixed-capacity queue with `enQueue`, `deQueue`, `Front`, `Rear`, `isEmpty`, `isFull`, all O(1), reusing freed space.
**Intuition:** A plain array queue wastes the space at the front after dequeues. Treat the array as a ring: when an index reaches the end, wrap to 0 using `% k`.
**Approach 1 - Brute force:** Shift items on dequeue. O(n) dequeue.
**Approach 2 - Optimal:** Keep `head` (index of the front) and `size`. The next free slot is `(head + size) % k` and the rear is `(head + size - 1) % k`. Tracking `size` removes the classic "is head == tail empty or full?" confusion.

Trace with k = 3: enQueue 1, 2, 3, deQueue, enQueue 4.
| op | array | head | size |
|---|---|---|---|
| enQueue 1,2,3 | [1,2,3] | 0 | 3 |
| deQueue | [1,2,3] | 1 | 2 |
| enQueue 4 | [4,2,3] (slot 0 reused) | 1 | 3 |

Front = 2, Rear = 4.

```js
class MyCircularQueue {
  constructor(k) { this.a = new Array(k); this.k = k; this.head = 0; this.size = 0; }
  enQueue(v) { if (this.isFull()) return false; this.a[(this.head + this.size) % this.k] = v; this.size++; return true; }
  deQueue() { if (this.isEmpty()) return false; this.head = (this.head + 1) % this.k; this.size--; return true; }
  Front() { return this.isEmpty() ? -1 : this.a[this.head]; }
  Rear() { return this.isEmpty() ? -1 : this.a[(this.head + this.size - 1) % this.k]; }
  isEmpty() { return this.size === 0; }
  isFull() { return this.size === this.k; }
}
```

**Complexity:** Time O(1) per operation, Space O(k).
**Watch out:** `Front` and `Rear` on an empty queue return `-1`, not `undefined`. This is the idea behind ring buffers used for logging, streaming and the deque in many libraries.

### 17. Rotting Oranges — Medium
**Companies:** Amazon, Microsoft, Google, Meta
**Problem:** Grid cells: 0 empty, 1 fresh, 2 rotten. Each minute, fresh oranges next to a rotten one (4 directions) become rotten. Return minutes until none are fresh, or `-1` if impossible.
**Intuition:** All rotten oranges start spreading at the same time. "Spread one step per minute from many sources" is multi-source BFS: put every rotten orange in the queue at the start, then process level by level. One level = one minute.
**Approach 1 - Brute force (simulation):** Each minute, scan the whole grid and rot the neighbours. Repeat until no change. Time O((R*C)^2) in the worst case.
**Approach 2 - Optimal (multi-source BFS):** Queue all `2` cells and count fresh ones. Process level by level (freeze the level size). Each newly rotten orange is marked as `2` right away so it is not added twice, and `fresh--`. Answer is the number of levels that rotted something, or `-1` if `fresh > 0` at the end.

Trace on `[[2,1,1],[1,1,0],[0,1,1]]`:
| minute | newly rotten cells | fresh left |
|---|---|---|
| start | (0,0) | 6 |
| 1 | (0,1), (1,0) | 4 |
| 2 | (0,2), (1,1) | 2 |
| 3 | (2,1) | 1 |
| 4 | (2,2) | 0 |

Answer 4.

```js
function orangesRotting(grid) {
  const R = grid.length, C = grid[0].length, g = grid.map(r => r.slice());
  const q = []; let fresh = 0, h = 0;
  for (let r = 0; r < R; r++) for (let c = 0; c < C; c++) {
    if (g[r][c] === 2) q.push([r, c]); else if (g[r][c] === 1) fresh++;
  }
  let minutes = 0;
  while (h < q.length && fresh > 0) {
    const end = q.length; // one level = one minute
    while (h < end) {
      const [r, c] = q[h++];
      for (const [dr, dc] of [[1,0],[-1,0],[0,1],[0,-1]]) {
        const nr = r + dr, nc = c + dc;
        if (nr >= 0 && nr < R && nc >= 0 && nc < C && g[nr][nc] === 1) { g[nr][nc] = 2; fresh--; q.push([nr, nc]); }
      }
    }
    minutes++;
  }
  return fresh === 0 ? minutes : -1;
}
```

**Complexity:** Time O(R*C), Space O(R*C).
**Watch out:** The loop stops when `fresh` hits 0, so the last level is not counted as an extra minute. A grid with no fresh oranges returns 0. A grid with fresh but no rotten returns `-1`. Copying the grid avoids changing the caller's input. Grid and BFS in depth: [Grid and Islands](../../04-graphs/02-grid-and-islands/README.md).

### 18. Basic Calculator II — Medium
**Companies:** Amazon, Meta, Google, Microsoft, Uber
**Problem:** Evaluate a string with non-negative integers, `+ - * /` and spaces, no brackets. Integer division truncates. `" 3+5 / 2 "` -> `5`.
**Intuition:** `*` and `/` bind tighter, so finish them immediately, but delay `+` and `-` by storing the signed numbers and summing at the end. The stack holds the terms of the sum.
**Approach 1 - Two stacks:** One for numbers, one for operators, with precedence rules (the general shunting-yard idea, see problem 19). More code.
**Approach 2 - Optimal:** Remember the operator that came *before* the current number (`op`). When the number ends (at the next operator or the end of string), apply `op`: `+` push `num`; `-` push `-num`; `*` replace the top with `top * num`; `/` replace the top with `trunc(top / num)`. Finally sum the stack.

Trace on `3+5/2` (op starts as `+`):
| number done | op | stack |
|---|---|---|
| 3 | + | 3 |
| 5 | + | 3 5 |
| 2 | / | 3 2 (5/2 truncated) |

Sum = 5.

```js
function calculateII(s) {
  const st = []; let num = 0, op = '+';
  for (let i = 0; i <= s.length; i++) {
    const c = s[i];
    if (c >= '0' && c <= '9') { num = num * 10 + Number(c); continue; }
    if (c === ' ') continue;
    if (op === '+') st.push(num);
    else if (op === '-') st.push(-num);
    else if (op === '*') st.push(st.pop() * num);
    else st.push(Math.trunc(st.pop() / num));
    op = c; num = 0;
  }
  return st.reduce((a, b) => a + b, 0);
}
```

**Complexity:** Time O(n), Space O(n) (can be O(1) by keeping only `last` and `sum`).
**Watch out:** The loop runs to `i <= s.length` so the last number is flushed (the `undefined` char acts as an end marker). Skip spaces without flushing. Use `Math.trunc`: `14-3/2` is `13`. Digits can be multi-digit.

### 19. Infix to Postfix Conversion — Medium
**Companies:** Amazon, Microsoft, Adobe, Zoho
**Problem:** Convert an infix expression such as `a+b*(c^d-e)` to postfix (operators after operands, no brackets). `a+b*c` -> `abc*+`.
**Intuition:** Operands go straight to the output. Operators wait on a stack. A new operator first pushes out the waiting operators that must run before it (higher precedence, or equal precedence and left-to-right). Brackets make a "wall" on the stack.
**Approach 1 - Brute force:** Build the expression tree by recursive descent and print it in postfix. Works, more code.
**Approach 2 - Optimal (shunting-yard):**
- operand -> output
- `(` -> push
- `)` -> pop to output until `(`, discard `(`
- operator `o` -> while the top is not `(` and has higher precedence than `o` (or equal and `o` is left-associative), pop to output; then push `o`
- end -> pop everything to output

Trace on `a+b*c`:
| char | output | stack |
|---|---|---|
| a | a | |
| + | a | + |
| b | ab | + |
| * | ab | + * (`*` binds tighter, do not pop `+`) |
| c | abc | + * |
| end | abc*+ | |

```js
function infixToPostfix(expr) {
  const prec = { '+': 1, '-': 1, '*': 2, '/': 2, '^': 3 };
  const out = [], st = [];
  for (const c of expr) {
    if (c === ' ') continue;
    if (/[A-Za-z0-9]/.test(c)) out.push(c);
    else if (c === '(') st.push(c);
    else if (c === ')') { while (st[st.length - 1] !== '(') out.push(st.pop()); st.pop(); }
    else {
      while (st.length && st[st.length - 1] !== '(' &&
        (prec[st[st.length - 1]] > prec[c] || (prec[st[st.length - 1]] === prec[c] && c !== '^'))) out.push(st.pop());
      st.push(c);
    }
  }
  while (st.length) out.push(st.pop());
  return out.join('');
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** `a-b-c` -> `ab-c-` (equal precedence, left-associative, so pop). `a^b^c` -> `abc^^` (`^` is right-associative, so do not pop on equal). Assumes valid input and single-character operands; say so. To evaluate the result, feed it to problem 9's stack.

### 20. Basic Calculator — Hard
**Companies:** Google, Amazon, Meta, Microsoft, Uber
**Problem:** Evaluate a string with non-negative integers, `+`, `-`, brackets and spaces (unary minus allowed, as in `-(2+3)`). `"(1+(4+5+2)-3)+(6+8)"` -> `23`.
**Intuition:** Without `*` and `/`, there is no precedence problem. Only brackets are hard. A `-` before a bracket flips everything inside. So keep a running `res` and the current `sign`. On `(`, save `res` and `sign` on the stack and start fresh. On `)`, combine the inner result with the saved sign and the saved result.
**Approach 1 - Recursion:** Evaluate the content inside brackets with a recursive call that returns the value and the new index. Same logic.
**Approach 2 - Optimal:** Scan once.
- digit: build `num`
- `+` / `-`: `res += sign * num`, reset `num`, set the new `sign`
- `(`: push `res`, then push `sign`; reset `res = 0`, `sign = 1`
- `)`: `res += sign * num`, reset `num`; then `res = savedSign * res + savedRes`
- end: `res + sign * num`

Trace on `2-(5-6)`:
| char | res | num | sign | stack |
|---|---|---|---|---|
| `2` | 0 | 2 | 1 | |
| `-` | 2 | 0 | -1 | |
| `(` | 0 | 0 | 1 | [2, -1] |
| `5` | 0 | 5 | 1 | |
| `-` | 5 | 0 | -1 | |
| `6` | 5 | 6 | -1 | |
| `)` | 5 - 6 = -1 -> -1 * -1 + 2 = 3 | 0 | -1 | [] |

Answer 3 (the sign variable is stale after `)` but it is reset by the next operator; at the end of the string `num` is 0 so it adds nothing).

```js
function calculateI(s) {
  const st = []; let res = 0, num = 0, sign = 1;
  for (const c of s) {
    if (c >= '0' && c <= '9') num = num * 10 + Number(c);
    else if (c === '+' || c === '-') { res += sign * num; num = 0; sign = c === '+' ? 1 : -1; }
    else if (c === '(') { st.push(res, sign); res = 0; sign = 1; }
    else if (c === ')') { res += sign * num; num = 0; res = st.pop() * res + st.pop(); }
  }
  return res + sign * num;
}
```

**Complexity:** Time O(n), Space O(n) for nesting depth.
**Watch out:** Pop order on `)`: you pushed `res` then `sign`, so the first pop gives `sign`. Flush the pending `num` before combining at `)`. Unary minus works because `-` at the start just adds `0`, or `res` is `0` after `(`. Basic Calculator III (adds `* /` and brackets) combines this with problem 18; try it next.

## Common Mistakes
- **"`shift()` is O(1) so a queue is free."** It may cost O(n). Use a head pointer or a proper deque.
- **Popping an empty stack.** `pop()` on `[]` returns `undefined` in JS; it does not throw. That can hide a bug, or help you (valid parentheses). Know which one you are relying on.
- **Forgetting the final check.** Valid parentheses must end with an empty stack. Remove k digits must handle leftover `k`.
- **Wrong operand order.** In RPN, the first pop is the right operand. `a - b` is not `b - a`.
- **`Math.floor` for division.** The problems say truncate toward zero. Use `Math.trunc`.
- **Mutating the queue while looping a level.** Freeze `const end = q.length` first, or the levels blur together.
- **Using a stack when only a counter is needed.** One bracket type means a depth counter is enough.
- **Marking visited when you pop, not when you push (BFS).** The same cell gets queued many times. Mark when you push.

## Interview Tips
- Say the structure first: "Most recent unfinished item, so a stack." or "Oldest first, so a queue."
- Say what the stack holds: indexes, characters, numbers, or pairs. Choosing indexes when you need positions is often the key.
- Ask: "Can the input be empty? Is it always valid? Can numbers be negative or multi-digit? Is division truncating?"
- Be ready for the "no extra space" follow-up (backspace compare, calculator) and the "iterative instead of recursive" follow-up (decode string, DFS).
- For "implement X using Y", state the cost trade-off out loud and use the word **amortized**.
- Recursion is a hidden stack. Any recursive solution can be turned into an explicit stack, which also avoids stack overflow on deep input (JS has a limited call stack depth).
- Mention that JavaScript has no built-in deque or queue; you would write a small class or use a library.
- For monotonic stack problems (next greater element, daily temperatures, largest rectangle), say "this is a monotonic stack" and see chapter 11.

## Practice List
**Easy**
- Baseball Game
- Make The String Great
- Remove All Adjacent Duplicates In String
- Next Greater Element I
- Crawler Log Folder
- Time Needed to Buy Tickets
- Number of Students Unable to Eat Lunch
- Final Prices With a Special Discount in a Shop

**Medium**
- Daily Temperatures (see chapter 11)
- Online Stock Span (see chapter 11)
- Next Greater Element II
- Remove All Adjacent Duplicates in String II
- Score of Parentheses
- Validate Stack Sequences
- Flatten Nested List Iterator
- Design Hit Counter
- Design Circular Deque
- Exclusive Time of Functions
- Remove Duplicate Letters
- Basic Calculator III (premium)
- Walls and Gates
- 132 Pattern

**Hard**
- Largest Rectangle in Histogram (see chapter 11)
- Maximal Rectangle
- Sliding Window Maximum (see chapter 11)
- Longest Valid Parentheses
- Trapping Rain Water
- Max Stack
- Shortest Subarray with Sum at Least K
- Number of Visible People in a Queue

## Key Takeaways
- **Stack = LIFO**, queue = FIFO. Use a stack for "most recent unfinished thing", a queue for "oldest first".
- In JavaScript, **`push`/`pop` are O(1)** but **`shift` may be O(n)**: use a head pointer for queues.
- **Valid brackets, decode, calculators, simplify path** all follow the same shape: push when something opens, resolve when it closes.
- **Two stacks make a queue** with amortized O(1) operations; one queue can make a stack, but one operation must cost O(n).
- **Store extra info with each item** (min so far, index, `[value, min]` pairs) to answer queries in O(1).
- **Multi-source BFS** starts with every source in the queue at once and counts levels as time.
- **Operator precedence** is handled by delaying `+ -` (stack of terms) or by shunting-yard; remember associativity for `^`.

---
[← Previous: Linked List](../09-linked-list/README.md) | [Next: Monotonic Stack and Queue →](../11-monotonic-stack-queue/README.md) | [Back to Core Linear Index](../README.md)
