# Design Problems (Design a Data Structure)

> **Estimated Time:** 6 hr | **Difficulty:** Advanced
> **Prerequisites:** [03 - Hashing](../../01-core-linear/03-hashing/README.md), [09 - Linked List](../../01-core-linear/09-linked-list/README.md), [10 - Stack and Queue](../../01-core-linear/10-stack-and-queue/README.md), [05 - Heaps](../../02-intermediate/05-heaps/README.md), [05 - Trie](../../03-trees/05-trie/README.md)

---

## What You'll Learn
- You can turn a vague "design a class" prompt into an API, a contract and a complexity budget per method.
- You can combine basic structures (hash map + list, array + map, heap + map) to hit O(1) or O(log n) per operation.
- You can build LRU, LFU, All-O(1), time-based stores, file systems and text editors from scratch.
- You can talk through a design question in a clear order, so the interviewer follows you.

## The Big Idea
Think of a kitchen. A fridge is great for finding things by label but bad for ordering them by age. A shelf is great for order but bad for finding. A good cook uses both and keeps a note of where everything is. A design problem is the same: one structure alone is rarely enough, so you glue two or three together, and each one covers the other's weakness.

Interviewers love these because they test several skills at once: choosing structures, keeping them in sync (the hard part), handling edge cases, and explaining trade-offs. There is usually no trick. There is a list of required operations with target complexities, and your job is to find the combination that meets all of them.

## How to Spot It
| If the problem says ... | think ... |
|---|---|
| "O(1) get and put, evict the oldest used" | hash map + doubly linked list (LRU) |
| "O(1) getRandom with insert and delete" | array + map of value to index, swap-with-last on delete |
| "get value at or before a timestamp" | per-key sorted array + binary search |
| "last N items / last 5 minutes" | queue (or circular buffer) with a running total |
| "paths, directories, prefixes" | trie, one node per path part or character |
| "top K / most recent K merged from many lists" | heap holding one pointer per list |
| "running median" | max-heap (low half) + min-heap (high half) |
| "undo / back / cursor in the middle" | two stacks, or array + pointer |
| "count of each key, min and max count in O(1)" | buckets by count in a linked list (All O(1)) |

## Core Concepts / Template

### The golden rule: invariants
Every design has one or two facts that must be true after every method. Write them down first. Example for LRU: "the map has exactly the keys in the list; the list is ordered newest to oldest". Each method must restore the invariant before it returns. Most bugs are a method that updates one structure and forgets the other.

### Template for the class
```js
class Design {
  constructor(/* capacity, config */) {
    // 1. the structures (state)
    // 2. a comment with the invariants
  }
  op1(args) { /* update every structure that holds this fact */ }
  op2(args) { /* read from the cheapest structure */ }
}
```

### Tools you will reuse
- **Sentinel nodes** (dummy head and tail) in a doubly linked list remove every "is it the first or last node?" branch.
- **Lazy deletion / lazy expiry**: do not clean up on a timer. Clean when the next call arrives (hit counter, token bucket).
- **Injectable clock**: pass `now` into time-based classes, so they are testable.
- A tiny heap, used by Twitter and Median Finder below (JS has no built-in heap). `less(a, b)` decides which item floats to the top.

```js
class Heap {
  constructor(less) { this.a = []; this.less = less; }
  get size() { return this.a.length; }
  peek() { return this.a[0]; }
  push(x) {
    const a = this.a; a.push(x);
    let i = a.length - 1;
    while (i > 0) { const p = (i - 1) >> 1; if (!this.less(a[i], a[p])) break; [a[i], a[p]] = [a[p], a[i]]; i = p; }
  }
  pop() {
    const a = this.a, top = a[0], last = a.pop();
    if (a.length) {
      a[0] = last;
      let i = 0;
      for (;;) {
        let s = i; const l = 2 * i + 1, r = l + 1;
        if (l < a.length && this.less(a[l], a[s])) s = l;
        if (r < a.length && this.less(a[r], a[s])) s = r;
        if (s === i) break;
        [a[i], a[s]] = [a[s], a[i]]; i = s;
      }
    }
    return top;
  }
}
```

### How to talk through a design question (mini-guide)
1. **Clarify the API.** Ask for method names, inputs, outputs, and what to return on a miss. Ask about sizes: how many keys, how many calls?
2. **Write the required complexity per method** on the board, e.g. "get O(1), put O(1)". This is your target.
3. **Start with the naive version** in one sentence and say why it fails the target.
4. **Pick structures by asking "what does each method need?"** The fast lookup by key needs a map. Ordering needs a list or heap. Random access needs an array.
5. **State the invariants** out loud. Then walk each method and show it keeps them.
6. **Dry-run a small example** with 3-4 calls, including an edge case (empty, full, duplicate key).
7. **Code the data first, methods second**; keep helper methods small (`_unlink`, `_addFront`).
8. **Finish with trade-offs and follow-ups**: thread safety, memory, resizing, persistence, distributed version.

## Problems (easy -> hard)

### 1. Min Stack — Easy
**Companies:** Amazon, Google, Microsoft, Bloomberg
**Problem:** A stack with `push`, `pop`, `top` and `getMin`, all O(1). Example: push -2, 0, -3; getMin -> -3; pop; top -> 0; getMin -> -2.
**Intuition:** The minimum only changes when you push or pop. So store, next to each value, "the minimum of everything at or below me". Popping removes both together.
**Approach 1 - Brute force:** Scan the stack for the min on each `getMin`. Time O(n), Space O(1).
**Approach 2 - Better:** A second stack of minimums. Same idea, two arrays.
**Approach 3 - Optimal:** One stack of pairs `[value, minSoFar]`. Trace: push -2 -> [-2,-2]; push 0 -> [0,-2]; push -3 -> [-3,-3]; pop -> top is [0,-2], so min is -2.

```js
class MinStack {
  constructor() { this.s = []; }
  push(x) { const m = this.s.length ? Math.min(x, this.s[this.s.length - 1][1]) : x; this.s.push([x, m]); }
  pop() { this.s.pop(); }
  top() { return this.s[this.s.length - 1][0]; }
  getMin() { return this.s[this.s.length - 1][1]; }
}
```

**Complexity:** Time O(1) per op, Space O(n).
**Watch out:** Do not keep a single `min` variable. After popping the min you cannot recover the old one.

### 2. Design HashMap (Separate Chaining) — Easy
**Companies:** Amazon, Google, Apple
**Problem:** Build `put`, `get`, `remove` for integer keys without using a built-in map. Example: put(1,1); get(1) -> 1; get(3) -> -1.
**Intuition:** Hash the key to a bucket number. Each bucket is a small list ("chain") of the pairs that landed there. Collisions just make a chain longer. Keep chains short by doubling the bucket count when the load factor (items / buckets) passes about 0.75.
**Approach 1 - Brute force:** A flat list of pairs, linear scan. O(n) per op.
**Approach 2 - Better:** A direct array of size 10^6 (key is the index). O(1) but wastes memory.
**Approach 3 - Optimal:** Buckets with chaining plus resize. A HashSet is the same thing with no value.

```js
class MyHashMap {
  constructor() { this.cap = 8; this.size = 0; this.b = Array.from({ length: 8 }, () => []); }
  _i(k) { return k % this.cap; }
  put(k, v) {
    const bucket = this.b[this._i(k)];
    for (const e of bucket) if (e[0] === k) { e[1] = v; return; }
    bucket.push([k, v]);
    if (++this.size > this.cap * 0.75) this._grow();
  }
  get(k) { for (const e of this.b[this._i(k)]) if (e[0] === k) return e[1]; return -1; }
  remove(k) {
    const bucket = this.b[this._i(k)];
    const i = bucket.findIndex(e => e[0] === k);
    if (i >= 0) { bucket.splice(i, 1); this.size--; }
  }
  _grow() {
    const old = this.b;
    this.cap *= 2;
    this.b = Array.from({ length: this.cap }, () => []);
    for (const bucket of old) for (const [k, v] of bucket) this.b[this._i(k)].push([k, v]);
  }
}
```

**Complexity:** Time O(1) average per op (resize is amortized), O(n) worst case if all keys collide. Space O(n).
**Watch out:** On `put` of an existing key, update, do not append a duplicate. For string keys you need your own hash function (e.g. polynomial rolling); `k % cap` only works for non-negative integers.

### 3. Implement Queue using Stacks — Easy
**Companies:** Amazon, Microsoft, Bloomberg, Apple
**Problem:** Build a FIFO queue with only stack operations. Example: push 1, push 2, peek -> 1, pop -> 1.
**Intuition:** Reversing a stack once turns LIFO into FIFO. Keep an input stack and an output stack. Only pour input into output when output is empty.
**Approach 1 - Brute force:** On every `push`, move everything to a helper and back. Push O(n).
**Approach 3 - Optimal:** Lazy pour. Each element is moved at most once, so pop is amortized O(1).

```js
class MyQueue {
  constructor() { this.inp = []; this.out = []; }
  push(x) { this.inp.push(x); }
  _move() { if (!this.out.length) while (this.inp.length) this.out.push(this.inp.pop()); }
  pop() { this._move(); return this.out.pop(); }
  peek() { this._move(); return this.out[this.out.length - 1]; }
  empty() { return !this.inp.length && !this.out.length; }
}
```

**Complexity:** Time amortized O(1) per op, Space O(n).
**Watch out:** Pouring while `out` is not empty breaks the order. Both stacks must be checked in `empty`.

### 4. Design Circular Queue (and Deque) — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** A fixed-capacity queue with `enQueue`, `deQueue`, `Front`, `Rear`, `isEmpty`, `isFull`, all O(1), reusing freed slots.
**Intuition:** Treat the array as a ring. Keep `head` and a `count`. The next write goes to `(head + count) % k`. Using `count` (not just head and tail) avoids the "empty or full?" confusion when head equals tail.
**Approach 1 - Brute force:** `shift()` from a normal array, O(n) per dequeue.
**Approach 3 - Optimal:** Ring buffer. For a deque, also allow `head = (head - 1 + k) % k` for push-front and `(head + count - 1) % k` for the rear.

```js
class MyCircularQueue {
  constructor(k) { this.a = new Array(k); this.k = k; this.head = 0; this.n = 0; }
  enQueue(x) { if (this.isFull()) return false; this.a[(this.head + this.n) % this.k] = x; this.n++; return true; }
  deQueue() { if (this.isEmpty()) return false; this.head = (this.head + 1) % this.k; this.n--; return true; }
  Front() { return this.isEmpty() ? -1 : this.a[this.head]; }
  Rear() { return this.isEmpty() ? -1 : this.a[(this.head + this.n - 1) % this.k]; }
  isEmpty() { return this.n === 0; }
  isFull() { return this.n === this.k; }
}
```

**Complexity:** Time O(1) per op, Space O(k).
**Watch out:** Negative modulo in JS stays negative; add `k` before `%` when moving backward.

### 5. Design Browser History — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** `visit(url)`, `back(steps)`, `forward(steps)`. Visiting a new page erases the forward pages. Back and forward stop at the ends.
**Intuition:** It is a list plus a "you are here" pointer. Visiting cuts the list at the pointer and appends.
**Approach 1 - Brute force:** Two stacks (back and forward), moving one page at a time: O(steps).
**Approach 3 - Optimal:** One array and a pointer. Setting `length = cur + 1` drops the forward part in O(1) in JS. Back and forward are just clamped arithmetic.

```js
class BrowserHistory {
  constructor(home) { this.p = [home]; this.cur = 0; }
  visit(url) { this.p.length = this.cur + 1; this.p.push(url); this.cur++; }
  back(steps) { this.cur = Math.max(0, this.cur - steps); return this.p[this.cur]; }
  forward(steps) { this.cur = Math.min(this.p.length - 1, this.cur + steps); return this.p[this.cur]; }
}
```

**Complexity:** Time O(1) per op (truncating `length` is cheap), Space O(n).
**Watch out:** Forgetting to truncate on `visit` keeps stale forward pages.

### 6. Moving Average from Data Stream — Easy
**Companies:** Google, Amazon, Meta
**Problem:** Given a window size, `next(val)` returns the average of the last `size` values. Example: size 3; next(1) -> 1, next(10) -> 5.5, next(3) -> 4.67, next(5) -> 6.
**Intuition:** Do not re-add the window. Keep a running sum; add the new value and subtract the one that just left.
**Approach 3 - Optimal:** Queue plus running sum. Trace (size 3): [1] sum 1 -> 1; [1,10] sum 11 -> 5.5; [1,10,3] sum 14 -> 4.67; add 5, drop 1: sum 18 -> 6.

```js
class MovingAverage {
  constructor(size) { this.size = size; this.q = []; this.head = 0; this.sum = 0; }
  next(v) {
    this.q.push(v); this.sum += v;
    if (this.q.length - this.head > this.size) this.sum -= this.q[this.head++];
    return this.sum / (this.q.length - this.head);
  }
}
```

**Complexity:** Time O(1) per call, Space O(size) if you also trim the array (use a ring buffer to be strict).
**Watch out:** `shift()` is O(n); use a head index. Divide by the current count, not `size`, while the window is still filling.

### 7. Logger Rate Limiter — Easy
**Companies:** Google, Amazon, Meta
**Problem:** `shouldPrintMessage(timestamp, message)` returns true only if the same message was not printed in the last 10 seconds.
**Intuition:** For each message remember the next time it is allowed. A map does it.
**Approach 3 - Optimal:** Map from message to `nextAllowed`. If `t < nextAllowed` reject, else set `nextAllowed = t + 10`. Trace: (1,"a") ok, next=11; (2,"a") rejected; (11,"a") ok.

```js
class Logger {
  constructor() { this.next = new Map(); }
  shouldPrintMessage(t, msg) {
    if ((this.next.get(msg) ?? 0) > t) return false;
    this.next.set(msg, t + 10);
    return true;
  }
}
```

**Complexity:** Time O(1), Space O(distinct messages).
**Watch out:** A rejected message must NOT extend its timer. Memory grows forever; a real system also evicts old entries (queue of expiries, or periodic cleanup).

### 8. Design Hit Counter — Medium
**Companies:** Google, Amazon, Dropbox
**Problem:** `hit(t)` records a hit; `getHits(t)` returns the number of hits in the last 300 seconds. Timestamps are non-decreasing.
**Intuition:** Hits arrive in time order, so old ones sit at the front of a queue. Drop the expired ones when asked.
**Approach 2 - Better:** Queue of timestamps (below). Space grows with the number of hits.
**Approach 3 - Optimal for heavy traffic:** 300 buckets indexed by `t % 300`, each storing the second it belongs to and a count. Space O(300) no matter the traffic.

```js
class HitCounter {
  constructor() { this.q = []; this.head = 0; }
  hit(t) { this.q.push(t); }
  getHits(t) {
    while (this.head < this.q.length && this.q[this.head] <= t - 300) this.head++;
    return this.q.length - this.head;
  }
}
```

**Complexity:** Time amortized O(1), Space O(hits in window) (bucket version: O(300)).
**Watch out:** The window is `(t-300, t]`, so a hit at `t-300` is expired.

### 9. Insert Delete GetRandom O(1) — Medium
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** A set with `insert`, `remove` and `getRandom` (uniform), all average O(1).
**Intuition:** A map gives O(1) lookup but no random pick. An array gives random pick but O(n) delete. Combine: array holds the values, map holds value -> index. To delete, overwrite the slot with the last element, then pop.
**Approach 1 - Brute force:** Array only; delete is O(n).
**Approach 3 - Optimal:** Trace: a=[1,2,3], pos={1:0,2:1,3:2}. remove(1): last=3, a[0]=3, pos[3]=0, pop -> a=[3,2].

```js
class RandomizedSet {
  constructor() { this.a = []; this.pos = new Map(); }
  insert(v) { if (this.pos.has(v)) return false; this.pos.set(v, this.a.length); this.a.push(v); return true; }
  remove(v) {
    if (!this.pos.has(v)) return false;
    const i = this.pos.get(v), last = this.a[this.a.length - 1];
    this.a[i] = last; this.pos.set(last, i); // order matters when v is last
    this.a.pop(); this.pos.delete(v);
    return true;
  }
  getRandom() { return this.a[Math.floor(Math.random() * this.a.length)]; }
}
```

**Complexity:** Time O(1) average per op, Space O(n).
**Watch out:** When `v` is the last element, set the map entry first and delete it last (as above), or you resurrect a deleted key. The follow-up with duplicates stores a Set of indices per value.

### 10. LRU Cache — Medium
**Companies:** Amazon, Google, Meta, Microsoft, Apple
**Problem:** A cache with capacity `c`. `get(k)` returns the value or -1. `put(k, v)` inserts or updates; if over capacity, evict the least recently used key. Both O(1).
**Intuition:** A map finds a key in O(1) but knows nothing about order. A doubly linked list keeps order and can move a node to the front in O(1) if you hold the node. So the map stores key -> node. The front is "most recent", the back is "least recent".
**Approach 1 - Brute force:** Array of keys with timestamps; O(n) to find the oldest.
**Approach 2 - Shortcut:** JS `Map` keeps insertion order. Delete and re-insert on each access, evict `map.keys().next().value`. Works, but many interviewers want the manual list.
**Approach 3 - Optimal:** Map + doubly linked list with dummy head and tail. Trace (cap 2): put1, put2 -> list [2,1]; get(1) -> [1,2]; put3 evicts tail.prev = 2 -> [3,1].

```js
class LRUCache {
  constructor(cap) {
    this.cap = cap; this.m = new Map();
    this.head = {}; this.tail = {};
    this.head.next = this.tail; this.tail.prev = this.head;
  }
  _unlink(n) { n.prev.next = n.next; n.next.prev = n.prev; }
  _addFront(n) { n.next = this.head.next; n.prev = this.head; this.head.next.prev = n; this.head.next = n; }
  get(k) {
    const n = this.m.get(k);
    if (!n) return -1;
    this._unlink(n); this._addFront(n);
    return n.val;
  }
  put(k, v) {
    let n = this.m.get(k);
    if (n) { n.val = v; this._unlink(n); this._addFront(n); return; }
    if (this.m.size === this.cap) { const lru = this.tail.prev; this._unlink(lru); this.m.delete(lru.key); }
    n = { key: k, val: v };
    this.m.set(k, n); this._addFront(n);
  }
}
```

**Complexity:** Time O(1) per op, Space O(c).
**Watch out:** The node must store its key, so eviction can delete it from the map. A `put` on an existing key also counts as a use. Both `get` and `put` refresh recency.

### 11. Time Based Key-Value Store — Medium
**Companies:** Amazon, Google, Meta, Lyft
**Problem:** `set(key, value, timestamp)` and `get(key, timestamp)` returns the value with the largest timestamp `<=` the query, or `""`. Timestamps for `set` are increasing.
**Intuition:** For each key, the history is already sorted by time, so "latest at or before t" is a binary search.
**Approach 1 - Brute force:** Linear scan of the history. O(n).
**Approach 3 - Optimal:** Map from key to an array of `[time, value]`; binary search for the last time `<= t`.

```js
class TimeMap {
  constructor() { this.m = new Map(); }
  set(k, v, t) { if (!this.m.has(k)) this.m.set(k, []); this.m.get(k).push([t, v]); }
  get(k, t) {
    const a = this.m.get(k);
    if (!a) return '';
    let lo = 0, hi = a.length - 1, ans = '';
    while (lo <= hi) {
      const mid = (lo + hi) >> 1;
      if (a[mid][0] <= t) { ans = a[mid][1]; lo = mid + 1; } else hi = mid - 1;
    }
    return ans;
  }
}
```

**Complexity:** `set` O(1), `get` O(log n), Space O(total sets).
**Watch out:** Query earlier than the first timestamp returns `""`. If `set` timestamps were not sorted you would need an ordered structure instead.

### 12. Snapshot Array — Medium
**Companies:** Google, Amazon, Atlassian
**Problem:** An array of length n, all zeros. `set(i, v)`, `snap()` returns a snapshot id, `get(i, snapId)` reads index i as of that snapshot.
**Intuition:** Copying the array on every snapshot is wasteful. Store only changes: per index, a list of `[snapId, value]` written in increasing snap order. Reading is a binary search.
**Approach 1 - Brute force:** Copy the whole array per `snap`. O(n) time and space each.
**Approach 3 - Optimal:** Per-index history. If the index was already set during the current snapshot period, overwrite instead of appending.

```js
class SnapshotArray {
  constructor(n) { this.h = Array.from({ length: n }, () => [[0, 0]]); this.id = 0; }
  set(i, v) {
    const a = this.h[i], last = a[a.length - 1];
    if (last[0] === this.id) last[1] = v; else a.push([this.id, v]);
  }
  snap() { return this.id++; }
  get(i, snapId) {
    const a = this.h[i];
    let lo = 0, hi = a.length - 1;
    while (lo < hi) { const mid = (lo + hi + 1) >> 1; if (a[mid][0] <= snapId) lo = mid; else hi = mid - 1; }
    return a[lo][1];
  }
}
```

**Complexity:** `set` O(1), `snap` O(1), `get` O(log s) where s is the number of writes to that index. Space O(total writes).
**Watch out:** `snap()` returns the id then increments. The initial `[0, 0]` entry makes every index readable at every snapshot.

### 13. Design Underground System — Medium
**Companies:** Amazon, Google, Bloomberg, Meta
**Problem:** `checkIn(id, station, t)`, `checkOut(id, station, t)`, `getAverageTime(start, end)` for trips that actually happened from start to end.
**Intuition:** Two maps. One remembers who is currently travelling (id -> start station and time). The other keeps, for every pair of stations, a running total time and trip count. Store the sum and count, not the average, so updates stay O(1) and exact.
**Approach 3 - Optimal:** On checkout, remove the check-in, add `t - t0` to the pair's total and increment the count.

```js
class UndergroundSystem {
  constructor() { this.in = new Map(); this.stat = new Map(); }
  checkIn(id, station, t) { this.in.set(id, [station, t]); }
  checkOut(id, station, t) {
    const [s, t0] = this.in.get(id);
    this.in.delete(id);
    const key = s + '->' + station, st = this.stat.get(key) ?? [0, 0];
    st[0] += t - t0; st[1]++;
    this.stat.set(key, st);
  }
  getAverageTime(a, b) { const [sum, n] = this.stat.get(a + '->' + b); return sum / n; }
}
```

**Complexity:** Time O(1) per op, Space O(active riders + station pairs).
**Watch out:** Direction matters: A->B and B->A are different keys. A separator in the key avoids ambiguity ("ab"+"c" vs "a"+"bc").

### 14. Peeking Iterator and Flatten Nested List Iterator — Medium
**Companies:** Google, Meta, Amazon, Apple
**Problem:** (a) Add `peek()` to an iterator that only has `next()` and `hasNext()`. (b) Given a nested list such as `[[1,1],2,[1,1]]`, iterate its integers in order: `1,1,2,1,1`.
**Intuition:** (a) Always keep one element read ahead. (b) A stack of "list + position" is your DFS path. `hasNext` digs until the top of the stack points at an integer; `next` returns it.
**Approach 1 - Brute force (b):** Flatten everything in the constructor. O(n) space up front, and it breaks for huge or lazy input.
**Approach 3 - Optimal:** Lazy stack. Trace `[[1],2]`: stack [[root,0]]; item 0 is a list -> advance root to 1, push [[1],0]; item is integer 1 -> hasNext true; `next` returns 1, position 1; next `hasNext` pops the finished list, finds 2.

```js
class PeekingIterator {
  constructor(iter) { this.it = iter; this.has = iter.hasNext(); this.nxt = this.has ? iter.next() : undefined; }
  peek() { return this.nxt; }
  next() {
    const r = this.nxt;
    this.has = this.it.hasNext();
    this.nxt = this.has ? this.it.next() : undefined;
    return r;
  }
  hasNext() { return this.has; }
}
class NestedIterator { // JS arrays stand in for NestedInteger lists
  constructor(list) { this.st = [[list, 0]]; }
  hasNext() {
    while (this.st.length) {
      const top = this.st[this.st.length - 1];
      if (top[1] === top[0].length) { this.st.pop(); continue; }
      const x = top[0][top[1]];
      if (typeof x === 'number') return true;
      top[1]++; this.st.push([x, 0]);
    }
    return false;
  }
  next() { this.hasNext(); const top = this.st[this.st.length - 1]; return top[0][top[1]++]; }
}
```

**Complexity:** Peeking: O(1) per op. Nested: amortized O(1) per integer, Space O(nesting depth).
**Watch out:** Empty inner lists like `[[], [[]]]` must be skipped inside `hasNext`. `next()` should call `hasNext()` itself in case the caller did not.

### 15. Design In-Memory File System — Hard
**Companies:** Amazon, Google, Dropbox, Microsoft
**Problem:** `ls(path)` (a file path returns just its name; a directory returns sorted children), `mkdir(path)` (creates missing parents), `addContentToFile(path, text)` (creates if missing, else appends), `readContentFromFile(path)`.
**Intuition:** Paths are a tree, so use a trie where each node is a path part. A node that holds `content !== null` is a file, otherwise a directory.
**Approach 3 - Optimal:** One `_walk(path, create)` helper does every traversal. Trace: `mkdir /a/b` walks `a`, `b`, creating nodes. `addContent /a/b/f` creates `f` and sets its content.

```js
class FileSystem {
  constructor() { this.root = { kids: new Map(), content: null }; }
  _walk(path, create) {
    let n = this.root;
    for (const p of path.split('/').filter(Boolean)) {
      if (!n.kids.has(p)) { if (!create) return null; n.kids.set(p, { kids: new Map(), content: null }); }
      n = n.kids.get(p);
    }
    return n;
  }
  ls(path) {
    const n = this._walk(path, false);
    if (n.content !== null) return [path.split('/').pop()];
    return [...n.kids.keys()].sort();
  }
  mkdir(path) { this._walk(path, true); }
  addContentToFile(path, text) { const n = this._walk(path, true); n.content = (n.content ?? '') + text; }
  readContentFromFile(path) { return this._walk(path, false).content; }
}
```

**Complexity:** Time O(L) per op for path length L (plus O(k log k) sorting in `ls`), Space O(total path parts + content).
**Watch out:** `"/".split('/')` gives empty strings; filter them. An empty file has content `''` (not `null`), which is why we compare to `null`. Real systems also need delete, move and permissions: mention them as extensions.

### 16. Design Twitter — Medium
**Companies:** Amazon, Google, Meta, Twitter
**Problem:** `postTweet`, `follow`, `unfollow`, and `getNewsFeed(user)` returning the 10 most recent tweet ids from the user and the people they follow, newest first.
**Intuition:** Each user's tweets are already sorted by time. The feed is "merge K sorted lists, take 10". Use a max-heap that holds just the newest unread tweet of each followee. Pop the newest, push that user's next older tweet.
**Approach 1 - Brute force:** Collect every tweet of everyone and sort. O(T log T).
**Approach 3 - Optimal:** K-way merge, stopping after 10 pops. Uses the `Heap` class from Core Concepts.

```js
class Twitter {
  constructor() { this.time = 0; this.tweets = new Map(); this.follows = new Map(); }
  postTweet(u, id) {
    if (!this.tweets.has(u)) this.tweets.set(u, []);
    this.tweets.get(u).push([this.time++, id]);
  }
  follow(a, b) { if (!this.follows.has(a)) this.follows.set(a, new Set()); this.follows.get(a).add(b); }
  unfollow(a, b) { this.follows.get(a)?.delete(b); }
  getNewsFeed(u) {
    const users = new Set(this.follows.get(u) ?? []); users.add(u);
    const h = new Heap((x, y) => x[0] > y[0]); // max-heap on time
    for (const v of users) {
      const t = this.tweets.get(v);
      if (t && t.length) h.push([t[t.length - 1][0], t[t.length - 1][1], v, t.length - 1]);
    }
    const res = [];
    while (res.length < 10 && h.size) {
      const [, id, v, i] = h.pop();
      res.push(id);
      if (i > 0) { const t = this.tweets.get(v)[i - 1]; h.push([t[0], t[1], v, i - 1]); }
    }
    return res;
  }
}
```

**Complexity:** Post/follow/unfollow O(1); feed O(F + 10 log F) for F followees. Space O(total tweets + follows).
**Watch out:** A user sees their own tweets. A global counter (not wall clock) gives a strict order. Follow-up: at scale, "push" fan-out (write to followers' feeds) vs "pull" (merge on read); celebrities make push expensive.

### 17. Design Text Editor — Hard
**Companies:** Google, Amazon, Meta
**Problem:** `addText(s)` at the cursor, `deleteText(k)` deletes up to k chars left of the cursor and returns how many were deleted, `cursorLeft(k)` / `cursorRight(k)` move the cursor and return up to 10 characters left of it.
**Intuition:** The cursor splits the text in two. Text left of it is a stack (top = nearest char); text right of it is another stack. Moving the cursor pops from one and pushes to the other. Everything happens at the cursor, so stack tops are enough.
**Approach 1 - Brute force:** One string; every edit rebuilds it. O(n) per op.
**Approach 3 - Optimal:** Two stacks. Trace: "leetcode" -> delete 4 -> "leet"; add "practice" -> "leetpractice"; cursorRight(3) with nothing on the right does nothing.

```js
class TextEditor {
  constructor() { this.L = []; this.R = []; }
  addText(s) { for (const c of s) this.L.push(c); }
  deleteText(k) { const d = Math.min(k, this.L.length); this.L.length -= d; return d; }
  _left() { return this.L.slice(-10).join(''); }
  cursorLeft(k) { while (k-- > 0 && this.L.length) this.R.push(this.L.pop()); return this._left(); }
  cursorRight(k) { while (k-- > 0 && this.R.length) this.L.push(this.R.pop()); return this._left(); }
}
```

**Complexity:** Each op O(length of text involved, i.e. k or |s|), Space O(n).
**Watch out:** Clamp k to what exists. The right stack stores characters in reverse, which is exactly the order you want when popping them back. The follow-up with huge text uses a rope or gap buffer.

### 18. Design Search Autocomplete System — Hard
**Companies:** Amazon, Google, Meta, Uber
**Problem:** Given sentences and counts, `input(c)` is called one character at a time and returns the top 3 sentences with the current prefix, ordered by count descending, then ASCII order. `'#'` ends the sentence, saves it (count +1) and resets.
**Intuition:** A trie follows the typed prefix one step per character. Each node remembers the sentences that pass through it with their counts, so a query is just "read the node, sort, take 3".
**Approach 1 - Brute force:** Filter all sentences by prefix on every character. O(N * L).
**Approach 3 - Optimal (simple):** Trie with a `Map(sentence -> count)` at each node. Keep a pointer to the current node so each character is one step. For huge data, store only the top 3 per node and update on insert.

```js
class AutocompleteSystem {
  constructor(sentences, times) {
    this.root = { kids: new Map(), cnt: new Map() };
    this.cur = this.root; this.buf = '';
    sentences.forEach((s, i) => this._add(s, times[i]));
  }
  _add(s, c) {
    let n = this.root;
    for (const ch of s) {
      if (!n.kids.has(ch)) n.kids.set(ch, { kids: new Map(), cnt: new Map() });
      n = n.kids.get(ch);
      n.cnt.set(s, (n.cnt.get(s) ?? 0) + c);
    }
  }
  input(c) {
    if (c === '#') { this._add(this.buf, 1); this.buf = ''; this.cur = this.root; return []; }
    this.buf += c;
    this.cur = this.cur && this.cur.kids.get(c);
    if (!this.cur) return [];
    return [...this.cur.cnt].sort((a, b) => b[1] - a[1] || (a[0] < b[0] ? -1 : 1)).slice(0, 3).map(e => e[0]);
  }
}
```

**Complexity:** Insert O(L) per sentence; each `input` O(m log m) for m sentences under that prefix (O(1) if you cache top 3). Space O(total characters of sentences x depth).
**Watch out:** Once the prefix leaves the trie, `cur` becomes `undefined`; keep it that way until `'#'` (hence `this.cur &&`). Tie-break by string order, not by insertion order.

### 19. Find Median from Data Stream — Hard
**Companies:** Amazon, Google, Meta, Microsoft, Apple
**Problem:** `addNum(x)` and `findMedian()`. Example: add 1, 2 -> 1.5; add 3 -> 2.
**Intuition:** The median sits between the smaller half and the larger half. Keep the smaller half in a max-heap (its top is the biggest of the small ones) and the larger half in a min-heap. Keep their sizes equal (or low has one extra).
**Approach 1 - Brute force:** Sort on every query. O(n log n).
**Approach 2 - Better:** Insert into a sorted array (binary search + splice). O(n) insert.
**Approach 3 - Optimal:** Two heaps. Add: push to low, move low's top to high, then if high is bigger, move its top back. Trace: add 1 -> low [1], high [] -> median 1. Add 2 -> low [1], high [2] -> median 1.5. Add 3 -> low [2,1], high [3] -> median 2 (the extra one lives in low).

```js
class MedianFinder { // uses Heap from Core Concepts
  constructor() { this.lo = new Heap((a, b) => a > b); this.hi = new Heap((a, b) => a < b); }
  addNum(x) {
    this.lo.push(x); this.hi.push(this.lo.pop());
    if (this.hi.size > this.lo.size) this.lo.push(this.hi.pop());
  }
  findMedian() { return this.lo.size > this.hi.size ? this.lo.peek() : (this.lo.peek() + this.hi.peek()) / 2; }
}
```

**Complexity:** `addNum` O(log n), `findMedian` O(1), Space O(n).
**Watch out:** Always push through the other heap first; that guarantees every low value is `<=` every high value. Follow-up: if all numbers are in 0..100, use a counting array; if 99% are in 0..100, count them and keep outliers separately.

### 20. LFU Cache — Hard
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Like LRU, but evict the least frequently used key; on a tie evict the least recently used among them. `get` and `put` in O(1).
**Intuition:** Group keys by use count. Each group is ordered by recency. Track `min`, the lowest count in use. A touch moves a key from group f to group f+1. Eviction takes the oldest key of group `min`.
**Approach 1 - Brute force:** Scan all keys for the lowest (count, time). O(n) per eviction.
**Approach 2 - Better:** A heap keyed by (count, time). O(log n).
**Approach 3 - Optimal:** Three maps: `kv` (key -> value), `freq` (key -> count), `buckets` (count -> Set of keys). A JS `Set` iterates in insertion order, so its first element is the oldest. Trace (cap 2): put1, put2, get1 (1 has freq 2); put3 evicts the oldest in bucket 1 which is key 2.

```js
class LFUCache {
  constructor(cap) { this.cap = cap; this.kv = new Map(); this.freq = new Map(); this.buckets = new Map(); this.min = 0; }
  _touch(k) {
    const f = this.freq.get(k), b = this.buckets.get(f);
    b.delete(k);
    if (!b.size) { this.buckets.delete(f); if (this.min === f) this.min++; }
    this.freq.set(k, f + 1);
    if (!this.buckets.has(f + 1)) this.buckets.set(f + 1, new Set());
    this.buckets.get(f + 1).add(k);
  }
  get(k) { if (!this.kv.has(k)) return -1; this._touch(k); return this.kv.get(k); }
  put(k, v) {
    if (this.cap === 0) return;
    if (this.kv.has(k)) { this.kv.set(k, v); this._touch(k); return; }
    if (this.kv.size === this.cap) {
      const b = this.buckets.get(this.min);
      const old = b.values().next().value; // oldest in the lowest-frequency bucket
      b.delete(old); if (!b.size) this.buckets.delete(this.min);
      this.kv.delete(old); this.freq.delete(old);
    }
    this.kv.set(k, v); this.freq.set(k, 1); this.min = 1;
    if (!this.buckets.has(1)) this.buckets.set(1, new Set());
    this.buckets.get(1).add(k);
  }
}
```

**Complexity:** Time O(1) per op, Space O(c).
**Watch out:** Capacity 0. New key always resets `min = 1` (after evicting). Updating an existing key's value also counts as a use. In languages without ordered sets, each bucket is a doubly linked list.

### 21. All O(1) Data Structure — Hard
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** `inc(key)`, `dec(key)` (removes the key at 0), `getMaxKey()`, `getMinKey()`, all O(1). Return `""` if empty.
**Intuition:** Same bucket idea as LFU, but counts move by exactly +1 or -1, so the buckets can live in a doubly linked list sorted by count. Min is next to the head, max is next to the tail. A key's bucket is found through a map.
**Approach 1 - Brute force:** Map of counts; scan for min and max. O(n).
**Approach 3 - Optimal:** Linked list of `{cnt, keys}` nodes. `inc` moves the key to the next node if its count is exactly cnt+1, else creates a node. Remove a node when it empties.

```js
class AllOne {
  constructor() {
    this.head = { cnt: 0, keys: new Set() }; this.tail = { cnt: Infinity, keys: new Set() };
    this.head.next = this.tail; this.tail.prev = this.head; this.where = new Map();
  }
  _after(node, cnt) { // new bucket right after node
    const b = { cnt, keys: new Set(), prev: node, next: node.next };
    node.next.prev = b; node.next = b; return b;
  }
  _drop(b) { if (!b.keys.size && b !== this.head && b !== this.tail) { b.prev.next = b.next; b.next.prev = b.prev; } }
  inc(key) {
    const cur = this.where.get(key) ?? this.head;
    const nxt = cur.next.cnt === cur.cnt + 1 ? cur.next : this._after(cur, cur.cnt + 1);
    nxt.keys.add(key); this.where.set(key, nxt);
    cur.keys.delete(key); this._drop(cur);
  }
  dec(key) {
    const cur = this.where.get(key);
    if (cur.cnt === 1) this.where.delete(key);
    else {
      const prv = cur.prev.cnt === cur.cnt - 1 ? cur.prev : this._after(cur.prev, cur.cnt - 1);
      prv.keys.add(key); this.where.set(key, prv);
    }
    cur.keys.delete(key); this._drop(cur);
  }
  getMaxKey() { return this.tail.prev === this.head ? '' : this.tail.prev.keys.values().next().value; }
  getMinKey() { return this.head.next === this.tail ? '' : this.head.next.keys.values().next().value; }
}
```

**Complexity:** Time O(1) per op, Space O(n).
**Watch out:** A new key starts from the head sentinel (count 0). Never delete the sentinels; the check in `_drop` protects them. Link the new node before removing the old one so neighbours stay valid.

### 22. Rate Limiter (Token Bucket) — Medium
**Companies:** Amazon, Google, Stripe, Uber
**Problem:** Allow at most `rate` requests per second on average but permit bursts up to `capacity`. `allow()` returns true or false.
**Intuition:** A bucket holds tokens, refilled at a steady rate up to a cap. Each request spends a token; no token, no entry. You do not need a timer: when a request arrives, compute how many tokens would have been added since the last request.
**Approach 1 - Brute force:** Log each request time and count those in the last second. O(n) space and time.
**Approach 2 - Better:** Fixed window counter. O(1) but allows 2x bursts at window edges.
**Approach 3 - Optimal:** Token bucket with lazy refill. Inject the clock so tests are deterministic.

```js
class TokenBucket {
  constructor(capacity, refillPerSec, now = () => Date.now()) {
    this.cap = capacity; this.rate = refillPerSec; this.now = now;
    this.tokens = capacity; this.last = now();
  }
  allow(cost = 1) {
    const t = this.now();
    this.tokens = Math.min(this.cap, this.tokens + ((t - this.last) / 1000) * this.rate);
    this.last = t;
    if (this.tokens < cost) return false;
    this.tokens -= cost;
    return true;
  }
}
```

**Complexity:** Time O(1) per call, Space O(1) per limited key (use a map from user id to bucket for per-user limits).
**Watch out:** Cap the tokens, or an idle client builds up an endless burst. Rejected calls still update `last`, which is fine because tokens were already added. Compare with the sliding-window log (exact, more memory) and leaky bucket (smooth output rate).

## Common Mistakes
- **"One structure is enough."** -> Most designs need two kept in sync. Write the invariant and check it in every method.
- **Forgetting to update both structures** (e.g. evicting from the list but not the map in LRU). -> Put paired updates into one helper.
- **Linked list without sentinels.** -> Dummy head and tail remove half of the edge cases.
- **Storing averages or derived values.** -> Store sums and counts; derive on read.
- **Using `array.shift()` for queues.** -> It is O(n). Use a head index or ring buffer.
- **Starting to code before asking about the API and sizes.** -> Clarify first; the right structure depends on the numbers.
- **Ignoring ties and boundaries** (LFU tie by recency, window edge `t - 300`). -> Re-read the statement for tie-break rules.

## Interview Tips
- Say the complexity target before any code: "get and put must both be O(1), so I need a map for lookup and something ordered for eviction."
- Name the invariant: "the map and the list always hold the same keys."
- Dry-run a short sequence out loud, including a full-cache or empty case.
- Edge cases to mention: capacity 0 or 1, missing key, duplicate insert, removing the last element, queries before any data, very large timestamps.
- Follow-ups you will hear: make it thread-safe (a lock around each method, or a concurrent map; mention contention), make it distributed (consistent hashing, replication), add TTL expiry (lazy on read plus a periodic sweep), reduce memory, support persistence.
- If you forget a library structure (heap, ordered map), say you will write a small one or assume it exists, and ask. Do not stall silently.
- Prefer clean helper methods over clever one-liners. Readable invariants win over micro-optimizations.

## Practice List
**Easy:** Implement Stack using Queues, Design HashSet, Design Parking System, Number of Recent Calls, Kth Largest Element in a Stream, Design Compressed String Iterator
**Medium:** Design Circular Deque, Design Phone Directory, Design Add and Search Words Data Structure, Implement Trie, Design Authentication Manager, Design Tic-Tac-Toe, Random Pick with Weight, Insert Delete GetRandom O(1) - Duplicates allowed, Design a Leaderboard, Design Skiplist, Zigzag Iterator, Design Snake Game, Max Stack, Design Most Recently Used Queue, Product of the Last K Numbers
**Hard:** LFU Cache (revisit), Design Search Autocomplete System (revisit), Maximum Frequency Stack, Design Excel Sum Formula, Design Bounded Blocking Queue, Stream of Characters, Range Module, My Calendar III

## Key Takeaways
- **Combine structures**: each one covers the other's weakness (map for lookup, list or heap for order, array for random access).
- **Write the complexity target for each method first**; it decides which combination is allowed.
- **Name the invariant** and make every method restore it; most bugs are two structures drifting apart.
- **Sentinels, lazy cleanup and injectable clocks** remove edge cases and make code testable.
- **Store sums and counts, not derived values**, and **store changes, not copies** (snapshot array).
- **LRU = map + doubly linked list; LFU/All-O(1) = buckets by count**; learn these two patterns cold.
- **Talk in a fixed order**: clarify API, target complexity, naive idea, structures, invariants, dry run, trade-offs.

---
[← Previous: Number Theory](../04-number-theory/README.md) | [Next: Data Structure Internals →](../06-data-structure-internals/README.md) | [Back to Advanced Index](../README.md)
