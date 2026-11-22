# JavaScript for DSA

> **Estimated Time:** 3 hr | **Difficulty:** Beginner
> **Prerequisites:** [02 - Complexity Analysis](../02-complexity-analysis/README.md)

---

## What You'll Learn

- You can name the cost of every common array, string, Map and Set operation, and avoid the slow ones (`shift`, `splice`, `s += c` in hot loops).
- You can dodge the classic JavaScript traps: lexicographic `sort()`, shared rows in a 2D array, 32-bit bit operators, unsafe integers, `-7 % 3`.
- You can pick the right number type: `Number`, `BigInt` or a typed array.
- You can paste tested helper classes that JavaScript lacks: `Heap` (min and max), `Deque`, `ListNode`, `TreeNode` with builders, `UnionFind` and `Trie`.

## The Big Idea

Think of a kitchen in a rented house. It has a stove and a fridge, but no blender, no scale, no timer. A good cook does not stop cooking; they bring their own tools and know exactly which ones the house provides. JavaScript is that house. It gives you arrays, strings, `Map` and `Set`. It does not give you a heap, a deque, an ordered map or a union-find. Interviewers know this, so they expect you to either write the tool quickly or say clearly what you would use.

Second, JavaScript has habits that surprise people from C++ or Java: there is one number type (a 64-bit float), `sort()` compares as strings, and bit operators work on 32 bits. Each habit causes bugs that look like algorithm errors but are language errors. Knowing them saves you from losing 10 minutes on a "wrong answer" that was a one-character fix.

Interviewers love this topic because it is cheap to test and very telling. Someone who says "`shift` is O(n), so I will use a queue with a head index" has clearly run real code.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "k largest", "k smallest", "merge k lists", "next smallest event" | Heap. JS has none, paste `Heap`. |
| "sliding window maximum", "BFS on a large graph" | Deque or head-index queue. Not `shift()` in a loop. |
| "sort numbers" | Always pass `(a, b) => a - b`. |
| "grid", "matrix", "DP table" | `Array.from({ length: n }, () => new Array(m).fill(0))`. |
| "answer modulo 1e9+7" with products | `a * b` can exceed 2^53. Use `BigInt` or a safe mulmod. |
| "numbers up to 10^18", "factorial", "big Fibonacci" | `BigInt`. |
| "count bits", "XOR", "power of two" with values near 2^31 or more | 32-bit bit-operator rules; use `>>> 0` or `BigInt`. |
| "connected components", "is there a cycle in undirected graph", "merge accounts" | `UnionFind`. |
| "prefix", "autocomplete", "word search II" | `Trie`. |
| "build a tree from [1,2,3,null,4]" | `buildTree`. |
| "deep recursion on 10^5 nodes" | Convert to an explicit stack, or the call stack overflows. |
| "smallest of an empty set" (`Math.min()`) | Remember `Infinity` / `-Infinity` as identity values. |

## Core Concepts / Template

### 1. Array methods and their cost

An array in V8 is a resizable block of memory. Jumping to `a[i]` is instant. Moving items around is not.

| Operation | Cost | Note |
|---|---|---|
| `a[i]`, `a.length`, `a.push(x)`, `a.pop()` | O(1) (push is amortized) | Use as a stack. |
| `a.shift()`, `a.unshift(x)` | O(n) in theory | Every item moves. V8 has tricks for small arrays, but do not rely on them. Never use as a queue on big input. |
| `a.splice(i, k)` | O(n) | Moves the tail. |
| `a.slice(i, j)` | O(j - i) | Makes a copy. `a.slice()` is O(n). Copying in recursion is a hidden O(n) per call. |
| `a.indexOf(x)`, `includes`, `find`, `some`, `every` | O(n) | Use a `Set` for repeated lookups. |
| `a.concat(b)`, `[...a, ...b]` | O(n + m) | Building an array with repeated spread in a loop is O(n^2). |
| `a.reverse()`, `a.join()`, `map`, `filter`, `forEach`, `reduce` | O(n) | `reverse` mutates. |
| `a.sort(cmp)` | O(n log n) | Stable since ES2019. Mutates. |
| `a.fill(v)` | O(n) | |
| `a.at(-1)` | O(1) | Last item without `a[a.length - 1]`. |

Mutating or not (this causes bugs): `sort`, `reverse`, `splice`, `push`, `pop`, `shift`, `unshift`, `fill` change the array. `slice`, `concat`, `map`, `filter`, `flat` return a new one. Newer Node versions also have `toSorted`, `toReversed`; do not rely on them in an interview judge unless you know the version.

`reduce` with no start value on an empty array throws. Always give the start value: `arr.reduce((s, v) => s + v, 0)`.

`['10','10','10'].map(parseInt)` gives `[10, NaN, 2]`, because `map` passes the index as a second argument and `parseInt` reads it as the radix. Write `map(Number)` or `map(x => parseInt(x, 10))`.

### 2. Sort: the comparator pitfall

`sort()` with no argument turns items into strings and compares them letter by letter. So `10` comes before `9` (because `"1" < "9"`).

```js
[10, 9, 1, 2].sort();                 // [1, 10, 2, 9]   WRONG for numbers
[10, 9, 1, 2].sort((a, b) => a - b);  // [1, 2, 9, 10]   ascending
[10, 9, 1, 2].sort((a, b) => b - a);  // [10, 9, 2, 1]   descending
```

The comparator returns a negative number (a first), zero (tie), or a positive number (b first). Three more rules:

- Sort pairs by two keys: `pts.sort((p, q) => p[0] - q[0] || p[1] - q[1])`. The `||` falls through to the second key on a tie.
- `a - b` can break if values can be `Infinity` (gives `NaN`) or are huge floats. For BigInt, return `a < b ? -1 : a > b ? 1 : 0`, because `a - b` is a BigInt and `sort` needs a Number.
- Never write `(a, b) => a > b` (returns true/false, not a signed number). It looks right and fails randomly.

Strings: `['b','a','B'].sort()` gives `['B','a','b']` (uppercase letters have smaller codes). For lowercase-only interview input it is fine. For anything else, use `a.localeCompare(b)` or compare char codes yourself.

`undefined` values always go to the end of a sorted array, and the comparator is not called on them.

### 3. Map and Set

| Structure | Lookup / insert / delete | Order |
|---|---|---|
| `Map` | O(1) average | Insertion order |
| `Set` | O(1) average | Insertion order |
| `Object` used as map | O(1) average | Integer-like keys first (ascending), then strings in insertion order |

Prefer `Map` over a plain object for counting. Objects have inherited keys (`"constructor"`), turn every key into a string, and reorder integer-like keys. `Map` keeps the key type: `1` and `"1"` are different.

Keys are compared by identity for objects and arrays. `map.set([1,2], v)` then `map.get([1,2])` gives `undefined`. For a pair or tuple key, build a string: `` `${r},${c}` ``. For a pair of small ints you can also use `r * COLS + c`.

```js
const count = new Map();
for (const x of nums) count.set(x, (count.get(x) ?? 0) + 1); // ?? not ||, so 0 is kept
for (const [k, v] of count) { /* destructure each entry */ }
const unique = [...new Set(nums)];
```

`NaN` is special: `[NaN].indexOf(NaN)` is `-1`, but `[NaN].includes(NaN)` and `Set.has(NaN)` are true.

There is no built-in ordered map (a sorted tree map like Java's `TreeMap`). If a problem needs "floor key" or "ceiling key" (calendar problems, `SortedList`), say so: keep a sorted array and use binary search (insert is O(n)), or write a balanced tree if time allows.

### 4. Strings are immutable

```js
let s = "abc";
s[0] = "z";   // silently ignored (throws in strict mode); s is still "abc"
```

To change a string, split to an array, edit, and `join`:

```js
const chars = s.split("");   // or [...s]
chars[0] = "z";
const t = chars.join("");
```

Cost notes: `s += c` in a loop works fine in V8 in practice (it uses internal ropes), but it is not guaranteed by the language. The safe habit is to push pieces into an array and `join("")` once. `s.slice(i, j)` and `s.substring(i, j)` are O(length of the piece) in the worst case; a loop that slices every time is O(n^2). `s.length` is O(1). Comparing two strings with `===` is O(length).

Character tricks you will use daily:

```js
"a".charCodeAt(0);                  // 97
String.fromCharCode(97 + 2);        // "c"
s.charCodeAt(i) - 97;               // index 0..25 for lowercase letters
new Array(26).fill(0);              // letter counts, faster than a Map
```

Emoji and some scripts take two UTF-16 units: `"😀".length` is 2, `[..."😀"].length` is 1. Interview inputs are usually ASCII; mention it only if the problem says Unicode.

### 5. Number precision, MAX_SAFE_INTEGER, BigInt

Every `Number` is a 64-bit float. Integers are exact only up to `Number.MAX_SAFE_INTEGER = 9007199254740991` (2^53 - 1). After that, neighbours collapse: `2**53 === 2**53 + 1` is `true`.

```js
0.1 + 0.2 === 0.3                       // false
Math.abs(0.1 + 0.2 - 0.3) < 1e-9        // true: compare floats with a tolerance
```

Use integer math whenever you can. Where it bites:

- Sums and products near 10^16 or more.
- `(a * b) % MOD` with `MOD = 1e9 + 7`: `a * b` can reach about 10^18, past 2^53, and the answer is silently wrong. With `a = b = 999999999`, plain `(a * b) % MOD` gives 63, but the true answer is 64.
- Factorials and Fibonacci past about F(78).

`BigInt` is exact for any size. Add `n` to a literal, or call `BigInt(x)`.

```js
const MOD = 1_000_000_007n;
const mulmod = (a, b) => Number((BigInt(a) * BigInt(b)) % MOD); // safe, slower
5n / 2n       // 2n   (division truncates toward zero)
-7n / 2n      // -3n
1n + 1        // TypeError: cannot mix BigInt and Number
```

Rules: do not mix with `Number` without converting; no `Math.*` on BigInt; compare with `<` fine, `===` only against another BigInt. BigInt is much slower, so use it only where needed (usually the multiplication step).

### 6. Integer division, modulo, and 32-bit bit operations

There is no integer division operator. `7 / 2` is `3.5`.

| Want | Write | Notes |
|---|---|---|
| floor of a / b | `Math.floor(a / b)` | `Math.floor(-7 / 2)` is `-4` |
| truncate toward zero | `Math.trunc(a / b)` | `-3`, like C++ and Java |
| quick truncate | `(a / b) \| 0` | Only if the result fits in 32 bits |
| positive modulo | `((a % m) + m) % m` | `-7 % 3` is `-1` in JS, the sign follows the left side |

Bit operators (`& | ^ ~ << >>`) first convert the number to a **signed 32-bit** integer. `>>>` gives an unsigned 32-bit result.

```js
2 ** 31 | 0       // -2147483648   (wraps)
2 ** 32 | 0       // 0             (high bits are thrown away)
1 << 31           // -2147483648
1 << 32           // 1             (shift count is taken mod 32)
(1 << 31) >>> 0   // 2147483648    (>>> 0 turns it back into unsigned)
-1 >>> 0          // 4294967295
-8 >> 1           // -4            (keeps the sign)
-8 >>> 28         // 15            (fills with zeros)
```

So bit tricks work only for values below 2^31 (or 2^32 with `>>> 0`). For a mask over 32 or more items, use `BigInt` (`1n << 40n`), or a `Uint32Array` and split the index. For a bitmask DP with 20 items, plain numbers are fine.

### 7. Typed arrays

`Int32Array`, `Uint8Array`, `Float64Array` and friends are fixed-size, zero-filled, and hold one numeric type. They use less memory and are faster than normal arrays for big tables (sieve, DP, counts). Values wrap or clamp silently:

```js
const a = new Int32Array(3);  a[0] = 2 ** 31;   // a[0] is -2147483648 (wraps)
const b = new Uint8Array(2);  b[0] = 300;       // 44 (300 mod 256)
const c = new Uint8ClampedArray(1); c[0] = 300; // 255 (clamped)
```

They have no `push`. They do have `fill`, `sort` (numeric by default, unlike plain arrays), `slice` and `set`. Use `new Uint8Array(n + 1).fill(1)` for a sieve. For a DP that needs values above 2^31, use `Float64Array` (exact to 2^53) or a normal array.

### 8. Infinity, NaN, destructuring

```js
Math.max()               // -Infinity   (identity for max)
Math.min()               // Infinity    (identity for min)
let best = Infinity;     // start for "find the minimum"
Infinity - Infinity      // NaN
NaN === NaN              // false; use Number.isNaN(x)
```

`Math.max(...arr)` spreads items as function arguments. With about 10^5 items or more it can throw a `RangeError`. Use a loop or `reduce` for big arrays.

Destructuring makes code short:

```js
let x = 1, y = 2;
[x, y] = [y, x];                 // swap, no temp (put a ; on the line before!)
const [first, ...rest] = [1, 2, 3];
for (const [i, v] of arr.entries()) {}
for (const [a, b] of edges) {}   // edges = [[0,1],[1,2]]
```

A swap line starting with `[` after a line without a semicolon gets glued to the previous line. Keep semicolons.

### 9. 2D arrays and the shared-reference bug

```js
const bad = new Array(3).fill([]);   // ONE array, three references
bad[0].push(1);
// bad is [[1], [1], [1]]

const good = Array.from({ length: 3 }, () => []);   // fresh array per row
good[0].push(1);
// good is [[1], [], []]

const grid = Array.from({ length: R }, () => new Array(C).fill(0)); // R x C of zeros
```

`fill` is safe for numbers, strings and booleans (copied by value) and unsafe for objects and arrays (copied by reference). The same trap exists for `new Array(3).fill({})`. Shallow copy of a grid: `grid.map(row => [...row])` (a plain `grid.slice()` still shares the rows).

### 10. Recursion depth

JavaScript has no tail-call optimisation in Node. Each call uses a stack frame. On the default Node setting you get about 10,000 frames before `RangeError: Maximum call stack size exceeded` (I measured about 10,400 for a trivial function; heavier frames give fewer, and the number differs by Node version and by judge). Practical meaning:

- A DFS on a skewed tree or a linked-list recursion with 10^5 nodes can crash. Use an explicit stack (an array with `push` and `pop`) and a loop.
- Memoised DP written top-down with depth 10^5 can crash; write it bottom-up.
- Binary search, merge sort, balanced-tree recursion (depth about log n) are fine.

Say this out loud in the interview: "Depth could reach n, so in production I would make this iterative."

### 11. Reusable toolkit (tested, paste what you need)

JavaScript has no heap, deque, ordered map or union-find. These snippets are plain classes with no imports. Each one was checked in Node against a brute-force model (heap against `sort`, deque against array `shift`/`unshift`, tree and list builders by round trip, union-find on 200,000 nodes). Later chapters refer to this section.

**Heap (min-heap or max-heap by comparator).** `cmp(a, b) < 0` means `a` leaves first. `push` and `pop` are O(log n), `peek` is O(1).

```js
// cmp(a, b) < 0 means a comes out first. Default = min-heap on numbers.
class Heap {
  constructor(cmp = (a, b) => a - b, items = []) {
    this.cmp = cmp;
    this.a = [];
    for (const x of items) this.push(x); // O(n log n); fine for interviews
  }
  get size() { return this.a.length; }
  peek() { return this.a[0]; }
  push(x) {
    const a = this.a;
    a.push(x);
    let i = a.length - 1;
    while (i > 0) {
      const p = (i - 1) >> 1;
      if (this.cmp(a[i], a[p]) >= 0) break;
      [a[i], a[p]] = [a[p], a[i]];
      i = p;
    }
  }
  pop() {
    const a = this.a;
    if (a.length === 0) return undefined;
    const top = a[0];
    const last = a.pop();
    if (a.length > 0) {
      a[0] = last;
      let i = 0;
      for (;;) {
        const l = 2 * i + 1, r = l + 1;
        let m = i;
        if (l < a.length && this.cmp(a[l], a[m]) < 0) m = l;
        if (r < a.length && this.cmp(a[r], a[m]) < 0) m = r;
        if (m === i) break;
        [a[i], a[m]] = [a[m], a[i]];
        i = m;
      }
    }
    return top;
  }
}
const MinHeap = (items) => new Heap((a, b) => a - b, items);
const MaxHeap = (items) => new Heap((a, b) => b - a, items);
```

```js
// usage
const h = MinHeap([5, 1, 3]);   h.push(0);  h.pop();  // 0
const mx = MaxHeap();           mx.push(2); mx.push(9);  mx.pop(); // 9
const pq = new Heap((a, b) => a[0] - b[0]);  // pairs [dist, node] for Dijkstra
```

For "k largest", keep a min-heap of size k (pop when size passes k). Ties and objects: compare by a numeric field in the comparator.

**Deque (O(1) at both ends).** Use for BFS, sliding window maximum, 0-1 BFS. A BFS queue that only needs `push` and `pop-front` can also be a plain array with a head index (`let h = 0; while (h < q.length) { const x = q[h++]; ... }`), which is simpler.

```js
// Circular buffer: O(1) push/pop at both ends. array.shift() is O(n).
class Deque {
  constructor() { this.buf = new Array(8); this.head = 0; this.n = 0; }
  get size() { return this.n; }
  _grow() {
    const nb = new Array(this.buf.length * 2);
    for (let i = 0; i < this.n; i++) nb[i] = this.buf[(this.head + i) % this.buf.length];
    this.buf = nb;
    this.head = 0;
  }
  pushBack(x) {
    if (this.n === this.buf.length) this._grow();
    this.buf[(this.head + this.n) % this.buf.length] = x;
    this.n++;
  }
  pushFront(x) {
    if (this.n === this.buf.length) this._grow();
    this.head = (this.head - 1 + this.buf.length) % this.buf.length;
    this.buf[this.head] = x;
    this.n++;
  }
  popFront() {
    if (this.n === 0) return undefined;
    const x = this.buf[this.head];
    this.buf[this.head] = undefined; // let GC reclaim
    this.head = (this.head + 1) % this.buf.length;
    this.n--;
    return x;
  }
  popBack() {
    if (this.n === 0) return undefined;
    const i = (this.head + this.n - 1) % this.buf.length;
    const x = this.buf[i];
    this.buf[i] = undefined;
    this.n--;
    return x;
  }
  peekFront() { return this.n ? this.buf[this.head] : undefined; }
  peekBack() { return this.n ? this.buf[(this.head + this.n - 1) % this.buf.length] : undefined; }
}
```

**Linked list node and builders.** LeetCode gives `ListNode`; in a local test file you must define it.

```js
class ListNode {
  constructor(val = 0, next = null) { this.val = val; this.next = next; }
}
// [1,2,3] -> 1->2->3
function buildList(arr) {
  const dummy = new ListNode();
  let t = dummy;
  for (const v of arr) { t.next = new ListNode(v); t = t.next; }
  return dummy.next;
}
function listToArray(head) {
  const out = [];
  for (let c = head; c; c = c.next) out.push(c.val);
  return out;
}
```

**Binary tree node and builders.** `buildTree([1,2,3,null,4])` reads the LeetCode level-order format (null means missing child, and a null has no children listed after it).

```js
class TreeNode {
  constructor(val = 0, left = null, right = null) {
    this.val = val; this.left = left; this.right = right;
  }
}
// LeetCode level-order format: [1,2,3,null,4] (null = no node)
function buildTree(arr) {
  if (!arr.length || arr[0] == null) return null;
  const root = new TreeNode(arr[0]);
  const q = [root];
  let i = 1, h = 0; // h = read index; avoids O(n) shift()
  while (h < q.length && i < arr.length) {
    const node = q[h++];
    if (i < arr.length && arr[i] != null) { node.left = new TreeNode(arr[i]); q.push(node.left); }
    i++;
    if (i < arr.length && arr[i] != null) { node.right = new TreeNode(arr[i]); q.push(node.right); }
    i++;
  }
  return root;
}
function treeToArray(root) {
  if (!root) return [];
  const out = [], q = [root];
  for (let h = 0; h < q.length; h++) {
    const n = q[h];
    if (n) { out.push(n.val); q.push(n.left, n.right); } else out.push(null);
  }
  while (out.length && out[out.length - 1] === null) out.pop();
  return out;
}
```

**Union-Find (disjoint set), with path compression and union by size.** Nearly O(1) per operation (inverse Ackermann). `union` returns `false` when the two were already joined, which is how you detect a cycle in an undirected graph.

```js
class UnionFind {
  constructor(n) {
    this.parent = Array.from({ length: n }, (_, i) => i);
    this.size = new Array(n).fill(1);
    this.count = n; // number of components
  }
  find(x) { // iterative path compression (no recursion-depth risk)
    let r = x;
    while (this.parent[r] !== r) r = this.parent[r];
    while (this.parent[x] !== r) { const nx = this.parent[x]; this.parent[x] = r; x = nx; }
    return r;
  }
  union(a, b) { // returns false if already connected (a cycle edge)
    let ra = this.find(a), rb = this.find(b);
    if (ra === rb) return false;
    if (this.size[ra] < this.size[rb]) [ra, rb] = [rb, ra];
    this.parent[rb] = ra;
    this.size[ra] += this.size[rb];
    this.count--;
    return true;
  }
  connected(a, b) { return this.find(a) === this.find(b); }
}
```

**Trie.** `Map` children work for any alphabet. For lowercase only, `new Array(26).fill(null)` is faster.

```js
class TrieNode {
  constructor() { this.next = new Map(); this.end = false; }
}
class Trie {
  constructor() { this.root = new TrieNode(); }
  insert(w) {
    let n = this.root;
    for (const ch of w) {
      if (!n.next.has(ch)) n.next.set(ch, new TrieNode());
      n = n.next.get(ch);
    }
    n.end = true;
  }
  _walk(s) {
    let n = this.root;
    for (const ch of s) { n = n.next.get(ch); if (!n) return null; }
    return n;
  }
  search(w) { const n = this._walk(w); return !!n && n.end; }
  startsWith(p) { return this._walk(p) !== null; }
}
```

## Worked Examples

Three short problems that use the toolkit. Each was run in Node against examples and edge cases.

### Example 1: Top K Frequent Elements (uses Heap)

Count with a `Map`, then keep a min-heap of size k keyed by count. When the heap grows past k, pop the weakest. What remains are the k most frequent. Time O(n log k), space O(n).

```js
// Top K frequent elements with a min-heap of size k: O(n log k)
function topKFrequent(nums, k) {
  const freq = new Map();
  for (const x of nums) freq.set(x, (freq.get(x) ?? 0) + 1);
  const h = new Heap((a, b) => a[1] - b[1]); // [value, count], smallest count on top
  for (const e of freq) {
    h.push(e);
    if (h.size > k) h.pop(); // drop the weakest
  }
  return h.a.map(e => e[0]);
}
```

Trace for `[1,1,1,2,2,3]`, k = 2: counts are 1:3, 2:2, 3:1. Push 1 (heap size 1), push 2 (size 2), push 3 (size 3, pop the smallest count, which is 3). Heap holds 1 and 2. The order of the returned items is not guaranteed.

### Example 2: Sliding Window Maximum (uses Deque)

Keep indices in the deque so their values are decreasing. The front is the max of the window. A new value removes all smaller values from the back (they can never be the max again). Time O(n), space O(k).

```js
// Sliding window maximum: deque holds indices, values decreasing. O(n)
function maxSlidingWindow(nums, k) {
  const dq = new Deque(), out = [];
  for (let i = 0; i < nums.length; i++) {
    while (dq.size && nums[dq.peekBack()] <= nums[i]) dq.popBack();
    dq.pushBack(i);
    if (dq.peekFront() <= i - k) dq.popFront(); // index left the window
    if (i >= k - 1) out.push(nums[dq.peekFront()]);
  }
  return out;
}
```

Trace for `[1,3,-1,-3,5,3,6,7]`, k = 3: deque (as values) after i=0: [1]; i=1: [3] (1 removed); i=2: [3,-1], output 3; i=3: [3,-1,-3], output 3; i=4: 5 clears everything, [5], and front index 1 left the window, output 5; and so on. Result `[3,3,5,5,6,7]`. Using `shift()` here would be fine for small input but is O(n) per call.

### Example 3: Number of Connected Components (uses UnionFind)

Start with n components. Each successful `union` merges two, so the count drops by 1. Time about O(n + E), space O(n).

```js
// Number of connected components: n nodes, undirected edges
function countComponents(n, edges) {
  const uf = new UnionFind(n);
  for (const [a, b] of edges) uf.union(a, b);
  return uf.count;
}
```

Trace for n = 5, edges `[0,1],[1,2],[3,4]`: count 5, then 4, then 3, then 2. Answer 2.

## Interview Questions

**1. Why does `[10, 9, 1].sort()` return `[1, 10, 9]`, and how do you fix it?**
Default `sort` converts items to strings and compares character by character, so `"10" < "9"`. Pass a comparator that returns a number: `(a, b) => a - b`.

**2. Why is `queue.shift()` a problem in BFS, and what do you do?**
`shift` moves every remaining item, so it is O(n) in theory and a BFS over 10^5 or more nodes can time out. Use a head index on a plain array, or the `Deque` above.

**3. `grid[0][0] = 1` changed every row. What happened?**
The grid was built with `new Array(R).fill(new Array(C).fill(0))`. `fill` copies the reference, not the array, so all rows are one array. Build with `Array.from({ length: R }, () => new Array(C).fill(0))`.

**4. How would you compute `(a * b) % 1_000_000_007` safely?**
`a * b` can pass 2^53 and lose digits. Convert to `BigInt`, multiply, take the modulo, then convert back: `Number((BigInt(a) * BigInt(b)) % 1000000007n)`. If only additions are needed, plain numbers are fine because sums of two values below 1e9+7 stay small.

**5. What does `1 << 31` give, and why does it matter?**
`-2147483648`. Bit operators use signed 32-bit integers, so bit 31 is the sign bit. Use `>>> 0` to read it as unsigned, or use `BigInt` for wider masks.

**6. JavaScript has no heap. What do you do in an interview?**
Say that the language lacks one, then write a short binary heap with a comparator (about 30 lines): sift-up on `push`, sift-down on `pop`. If the interviewer allows, ask whether a library is allowed. Know the complexity: push and pop O(log n), peek O(1), building from n items with repeated push O(n log n).

**7. A recursive DFS crashes on a 100,000-node chain. Why, and how do you fix it?**
Each call uses stack space and Node allows about 10,000 frames by default. Rewrite with an explicit stack and a loop.

## Common Mistakes

- **"`sort()` sorts numbers."** It sorts text. Always pass `(a, b) => a - b` for numbers.
- **"`new Array(n).fill([])` makes n empty rows."** It makes n references to one row. Use `Array.from({ length: n }, () => [])`.
- **"`a / b` is integer division."** It is float division. Use `Math.floor` or `Math.trunc`, and pick the one that matches the problem for negative numbers.
- **"`-7 % 3` is 2."** It is `-1`. Use `((a % m) + m) % m` for a non-negative remainder.
- **"Numbers are exact up to 10^18."** Exact only to about 9 x 10^15. Use `BigInt` beyond that.
- **"Bit operators work on big numbers."** They cut to 32 bits. `2**32 | 0` is `0`.
- **"`x || 0` is a safe default."** `||` also replaces a real `0` or empty string. Use `x ?? 0` when 0 is a valid value.
- **"A `Map` key can be an array."** Only by identity. Use a string key such as `` `${r},${c}` ``.
- **"`arr.length = 0` and `arr = []` are the same."** The first clears the array that other variables also point to; the second makes a new one and leaves other references untouched.
- **"`for...in` loops over array items."** It loops over index strings (and inherited keys). Use `for...of`.
- **"`Math.max(...bigArray)` is fine."** A huge spread can overflow the call stack arguments limit. Use a loop.

## Interview Tips

- Say your tool choices out loud: "JS has no heap, so I will write a small one with a comparator", "I will use a head index instead of `shift`".
- State the cost of the built-ins you call: `includes` is O(n), `slice` copies, `sort` is O(n log n).
- Ask about limits early: if values can exceed 2^53 (about 9 x 10^15) or products appear with a modulus, you need `BigInt` or a safe mulmod.
- Mention 32-bit limits when you use `<<`, `|`, `~` or `>>`.
- Test edge cases aloud: empty array, one element, negative numbers, duplicates, `0` as a valid value, all-equal values, `null` children in trees.
- Follow-ups interviewers ask: "Can you do it without extra space?", "What if the array is sorted?", "What is the worst case of your hash map?" (all keys collide, O(n); mention it, do not dwell), "How would you handle recursion depth?"
- Write helper classes before the main logic and keep them compact. If the interviewer says a built-in is allowed (some companies provide a `PriorityQueue`), use it and move on.
- Prefer `const`/`let`, `for...of`, and clear names. Avoid clever one-liners you cannot explain.

## Practice List

**Easy:** Two Sum, Valid Anagram, Contains Duplicate, Reverse String, Majority Element, Squares of a Sorted Array, Kth Largest Element in a Stream, Last Stone Weight

**Medium:** Sort Colors, Top K Frequent Elements, K Closest Points to Origin, Sliding Window Maximum (as a deque drill), Number of Connected Components in an Undirected Graph, Implement Trie (Prefix Tree), Rotate Image, Set Matrix Zeroes, Pow(x, n), Evaluate Reverse Polish Notation (integer division), Insert Delete GetRandom O(1)

**Hard:** Find Median from Data Stream, Merge k Sorted Lists, Word Search II, Sliding Window Median, My Calendar III (ordered-map thinking)

## Key Takeaways

- **Know the cost of built-ins**: `push`/`pop`/index are O(1); `shift`, `unshift`, `splice`, `indexOf`, `includes` are O(n); `sort` is O(n log n).
- **Always pass a numeric comparator to `sort`**; the default is lexicographic.
- **Build 2D arrays with `Array.from`**, never `fill([])`, or every row is the same array.
- **Numbers are 64-bit floats**: exact only to 2^53 - 1. Use `BigInt` for big products, factorials and modulo math.
- **Bit operators are 32-bit signed**; use `>>> 0` for unsigned, `BigInt` for wide masks. Integer division needs `Math.floor` or `Math.trunc`.
- **JavaScript has no heap, deque, ordered map or union-find**: keep the tested classes from this chapter ready.
- **Recursion depth is about 10,000 frames**: go iterative for deep trees, lists and graphs.

---

[← Previous: Complexity Analysis](../02-complexity-analysis/README.md) | [Next: Math Basics →](../04-math-basics/README.md) | [Back to Foundations Index](../README.md)
