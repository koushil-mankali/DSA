# Data Structure Internals

> **Estimated Time:** 4 hr | **Difficulty:** Advanced
> **Prerequisites:** [03 - Hashing](../../01-core-linear/03-hashing/README.md), [05 - Heaps](../../02-intermediate/05-heaps/README.md), [04 - Binary Search Trees](../../03-trees/04-binary-search-trees/README.md), [05 - Trie](../../03-trees/05-trie/README.md), [06 - Union Find](../../04-graphs/06-union-find/README.md), [05 - Design Problems](../05-design-problems/README.md)

---

## What You'll Learn

- You can explain how a hash table works inside: hashing, collisions, load factor, resizing, and why lookup is O(1) on average.
- You can explain why appending to a dynamic array is amortized O(1).
- You can describe how a balanced BST (AVL, red-black) keeps its height at O(log n), and write an AVL insert.
- You can explain why databases use B+ trees, and when a skip list, Bloom filter or consistent hash ring is the right tool.
- You can answer "which structure for which operation" and defend the choice with trade-offs.

## The Big Idea

Think of a library. A **hash table** is a catalogue that tells you the exact shelf from the book title. A **balanced BST** is a library sorted on the shelves, so you can also ask "all books between D and F". A **B+ tree** is the same library, but each sign in the hallway lists 100 directions instead of 2, so you reach the right shelf in 4 or 5 signs. A **Bloom filter** is a guard at the door who says "definitely not in this library" or "maybe".

Most interviews do not ask you to use these structures. They ask "what happens when you call `map.set(k, v)`?" or "why not just use a BST for a database index?". The answer shows if you understand the cost behind the API. That is why interviewers love this topic: it separates people who memorised complexities from people who know where they come from.

## How to Spot It

| If the question says ... | think ... |
|---|---|
| "How does a hash map work internally?" | hash function, buckets, collisions, load factor, resize |
| "Why is insert O(1) amortized?" | doubling capacity, geometric sum |
| "Why not use a plain BST?" | sorted input makes a chain, height O(n); balance it |
| "Index on disk / database" | B-tree or B+ tree, few levels, wide nodes, page-sized reads |
| "Is this item possibly in a huge set, using little memory?" | Bloom filter (no false negatives, some false positives) |
| "Spread keys over servers, servers come and go" | consistent hashing with virtual nodes |
| "Sorted set that is easy to make concurrent" | skip list |
| "Prefix search, autocomplete" | trie (see the Trie chapter) |
| "Evict the least recently / frequently used" | hash map + linked list (LRU), frequency buckets (LFU) |
| "Which structure supports X?" | the comparison table at the end |

## Core Concepts

### 1. Hash tables

**Goal:** turn a key into an array index, so lookup is "compute index, look there".

Steps for `set(key, value)`:

1. Compute `h = hash(key)`, a big integer.
2. Reduce it to a bucket: `index = h % capacity` (many implementations use a power of two and a bit mask instead of `%`).
3. Look in that bucket. If the key is already there, replace its value. Otherwise add it.
4. If the table is too full, grow it and move everything (rehash).

A good hash function is **deterministic** (same key, same hash), **fast**, and **spreads keys evenly**. Equal keys must give equal hashes. The reverse is not true: different keys can share a hash.

#### Collisions

Two keys landing in the same bucket is a **collision**. It cannot be avoided: more possible keys than buckets (pigeonhole). Two classic fixes:

| | Chaining | Open addressing |
|---|---|---|
| Idea | each bucket holds a list of entries | one entry per slot; on clash, probe other slots |
| Probing | n/a | linear (next slot), quadratic, or double hashing |
| Delete | remove from list | needs a **tombstone** marker, or later lookups stop too early |
| Load factor | can go above 1 | must stay below 1 (slots run out) |
| Cache | pointer chasing | contiguous array, cache friendly |
| Weak spot | long chains if hash is bad | **clustering**: runs of full slots grow and slow probing |

Some implementations use chaining and switch a very long chain into a balanced tree (Java's `HashMap` does this). Python's `dict` uses open addressing.

#### Load factor and resizing

**Load factor** = `entries / buckets`. With chaining it is the average chain length. Keep it small (a common threshold is around 0.75) and each lookup touches only a few entries. When it crosses the threshold, allocate double the buckets and **rehash every key** (the index depends on capacity, so every entry moves). One resize costs O(n), but it happens rarely, so the cost spreads out: amortized O(1) per insert.

#### Why O(1) average, and when it is not

If the hash spreads keys evenly and load factor is bounded by a constant, the expected chain length is a constant. So lookup, insert and delete are O(1) **expected**. Worst case is O(n) when all keys collide (bad hash, or an attacker choosing keys that collide on purpose). Real systems mitigate the attack with randomized hash seeds. Also hashing a key is itself O(length of key), so for long strings "O(1)" hides a factor of the key length.

Hash tables give **no order** and no "next larger key". If you need range queries or sorted iteration, you need a tree.

#### Chaining table in JS

```js
class ChainMap {
  constructor(cap = 4) { this.buckets = Array.from({ length: cap }, () => []); this.size = 0; }
  static hash(key) {                       // string hash, 32-bit, djb2-style
    let h = 5381;
    for (const ch of String(key)) h = (Math.imul(h, 33) + ch.charCodeAt(0)) >>> 0;
    return h;
  }
  _bucket(key) { return this.buckets[ChainMap.hash(key) % this.buckets.length]; }
  get(key) { const e = this._bucket(key).find(e => e[0] === key); return e && e[1]; }
  set(key, val) {
    const b = this._bucket(key), e = b.find(e => e[0] === key);
    if (e) { e[1] = val; return; }
    b.push([key, val]); this.size++;
    if (this.size / this.buckets.length > 0.75) this._resize();   // load factor
  }
  delete(key) {
    const b = this._bucket(key), i = b.findIndex(e => e[0] === key);
    if (i < 0) return false;
    b.splice(i, 1); this.size--; return true;
  }
  _resize() {
    const old = this.buckets;
    this.buckets = Array.from({ length: old.length * 2 }, () => []);
    for (const b of old) for (const [k, v] of b) this._bucket(k).push([k, v]);  // rehash all
  }
}
```

Trace with capacity 4: after 3 inserts the load is 3/4 = 0.75, which is not above the threshold. The 4th insert makes it 4/4 = 1, so we resize to 8 buckets and rehash all 4 keys. Each later resize doubles again.

#### JavaScript `Map` and `Set`

- Keys can be any value. Comparison is **SameValueZero**: `NaN` equals `NaN`, and `0` equals `-0`. Objects are compared by **reference**, so two equal-looking `{}` are different keys.
- Iteration order is **insertion order**. Re-setting an existing key keeps its original position.
- The spec only requires that access time is sublinear on average, so the real structure is an engine detail. Engines use a hash table that also remembers insertion order.
- `Object` as a map is different: string and symbol keys only, integer-like keys iterate first in ascending order, and it carries prototype baggage. Prefer `Map` for dynamic key sets and frequent add/delete.
- Mutating a key object after using it as a key is fine for `Map` (identity, not content). Using a joined string like `` `${x},${y}` `` as a key is the usual trick for pairs, and it costs the hashing of that string.

### 2. Dynamic arrays and amortized analysis

A fixed array cannot grow. A dynamic array (JS `Array`, Java `ArrayList`, Python `list`) keeps spare capacity. `push` writes into spare room, O(1). When full, it allocates a bigger block (growth by a constant factor, such as 1.5 or 2, depending on the engine), copies everything, and continues.

**Why amortized O(1) with doubling:** starting at capacity 1 and pushing n items, copies happen at sizes 1, 2, 4, 8, ... The total copy work is `1 + 2 + 4 + ... + n/2 < n`. Add the n plain writes and the total is under `2n`, so the average cost per push is a constant. Test: pushing 1024 items from capacity 1 does exactly 1023 copies.

**Amortized is not average-case.** It is a guarantee over any sequence of operations: one single push can cost O(n), but n pushes cost O(n) in total. If the growth were `+10` each time (arithmetic instead of geometric), the total would be O(n^2).

Other facts: random access `a[i]` is O(1); insert or delete in the middle is O(n) (shifting). Shrinking is optional. JS engines also store arrays in different internal shapes (packed integers, doubles, generic, or sparse dictionary mode), so a `new Array(1e6)` with holes can behave very differently from a filled one. That is an engine detail, do not quote exact numbers.

### 3. Binary heap internals (recap)

A heap stores a complete binary tree **in an array**: children of index `i` are `2i+1` and `2i+2`, parent is `(i-1) >> 1`. `push` adds at the end and **sifts up**. `pop` swaps the root with the last element, removes it, and **sifts down**. Both walk one root-to-leaf path: O(log n). Peek is O(1). Building a heap from n items with sift-down from the middle is O(n), not O(n log n), because most nodes are near the bottom and move very little. Search for an arbitrary value is O(n): a heap is only half-sorted. Full details and problems are in [05 - Heaps](../../02-intermediate/05-heaps/README.md).

### 4. Why balanced BSTs

A BST gives O(h) search, insert, delete where `h` is the height. Insert 1, 2, 3, ..., n in order and the tree becomes a chain: `h = n`, everything is O(n). A **self-balancing** BST repairs the shape after each change so `h = O(log n)`. The repair tool is the **rotation**: a local, constant-time rearrangement that keeps the in-order sequence (so it is still a valid BST) and changes the heights.

```
Right rotation at y            Left rotation at x
      y            x                x                 y
     / \          / \              / \               / \
    x   C   ->   A   y            A   y     ->      x   C
   / \              / \              / \           / \
  A   B            B   C            B   C         A   B
```

#### AVL tree

Rule: for every node, `|height(left) - height(right)| <= 1`. After an insert, walk back up, update heights, and check the **balance factor** `h(left) - h(right)`. If it hits +2 or -2, fix with one of four cases:

| Case | Where the new key went | Fix |
|---|---|---|
| Left-Left | left child's left subtree | rotate right at node |
| Left-Right | left child's right subtree | rotate left at left child, then right at node |
| Right-Right | right child's right subtree | rotate left at node |
| Right-Left | right child's left subtree | rotate right at right child, then left at node |

An AVL tree's height is at most about 1.44 log2(n), so lookups are very fast. It rotates more often than red-black trees on updates.

```js
class Node { constructor(k) { this.key = k; this.left = null; this.right = null; this.h = 1; } }
const h = n => (n ? n.h : 0);
const upd = n => { n.h = 1 + Math.max(h(n.left), h(n.right)); };
const bal = n => h(n.left) - h(n.right);
function rotR(y) { const x = y.left; y.left = x.right; x.right = y; upd(y); upd(x); return x; }
function rotL(x) { const y = x.right; x.right = y.left; y.left = x; upd(x); upd(y); return y; }
function avlInsert(n, k) {
  if (!n) return new Node(k);
  if (k < n.key) n.left = avlInsert(n.left, k);
  else if (k > n.key) n.right = avlInsert(n.right, k);
  else return n;
  upd(n);
  const b = bal(n);
  if (b > 1) { if (k > n.left.key) n.left = rotL(n.left); return rotR(n); }     // LL or LR
  if (b < -1) { if (k < n.right.key) n.right = rotR(n.right); return rotL(n); } // RR or RL
  return n;
}
```

Trace: insert 10, 20, 15. After 10, 20: right-leaning. Insert 15 goes to the left of 20, so node 10 has balance -2 and the new key is below the **left** of its right child: Right-Left case. Rotate right at 20 (15 becomes the right child of 10, with 20 under it), then rotate left at 10. Root is 15 with children 10 and 20. Tested: inserting 1..1023 in sorted order gives height exactly 10 and root 512.

**Delete** is the same idea: remove like a normal BST, then rebalance on the way up. It can need rotations at several levels (O(log n) total).

#### Red-black tree (idea only)

Each node is red or black, with these rules: root is black; red nodes have no red child; every path from a node down to its null leaves has the same number of black nodes. Those rules force the longest path to be at most twice the shortest, so height is O(log n) (at most 2 log2(n+1)). Insert and delete use recoloring plus at most a few rotations. It is **less strictly balanced** than AVL (slightly slower lookups) but does less work per update. It is the usual choice in library ordered maps (for example Java's `TreeMap`). In an interview you almost never code it. Know the rules and the trade-off with AVL.

JS has no built-in ordered map or tree. If an interview needs one, say so and write the minimal structure you need, or use a heap, sorted array plus binary search, or sorted-key tricks.

### 5. B-tree and B+ tree: why databases use them

A disk (or SSD) read fetches a whole **page** (a block of several KB), and it is thousands of times slower than memory. The cost to minimise is the **number of page reads**, not comparisons. A binary tree with a million keys has about 20 levels, so about 20 reads. A **B-tree** node holds many keys (one page worth) and has many children. With a fanout of 100, 100^4 = 100 million, so about 5 levels reach billions of keys. And the top levels stay cached in memory, so a lookup is often 1 or 2 real disk reads.

- **B-tree:** each node has between about `t-1` and `2t-1` sorted keys. Every leaf is at the same depth. Insert into a full node **splits** it in two and pushes the middle key up. Delete may **merge** or borrow from siblings. Height stays O(log_t n).
- **B+ tree:** the common variant in databases. Internal nodes hold **only keys for routing**; all the actual records (or pointers) live in the **leaves**, and leaves are linked in a list. Benefits: internal nodes are small, so fanout is higher; range scans walk the leaf list; every lookup costs the same number of reads.

```
        [ 30 | 60 ]                 <- internal: routing keys only
       /     |     \
 [10 20] -> [30 40 50] -> [60 70 80]   <- leaves, linked, hold the data
```

Why not a hash index? A hash index is great for `WHERE id = 5`, but cannot do `WHERE id BETWEEN 5 AND 50`, `ORDER BY`, or prefix matches. A B+ tree does all of these. Why not a balanced BST? Too tall, and neighbouring nodes are scattered across pages. Note LSM trees (used by several write-heavy stores) are another family: they favour fast writes and use Bloom filters to skip files.

### 6. Skip list

A skip list is a sorted linked list with **express lanes**. Level 0 holds all keys. Each node is also promoted to level 1 with probability 1/2, to level 2 with probability 1/4, and so on. Search starts at the top lane, moves right while the next key is smaller, then drops down a level. Expected O(log n) for search, insert, delete, with expected O(n) space (about 2n pointers).

```
L2:  H ----------------> 17 ------------------> nil
L1:  H -------> 9 ------> 17 -------> 25 -----> nil
L0:  H -> 3 -> 9 -> 12 -> 17 -> 20 -> 25 -> 31 -> nil
```

Why use it over a balanced tree? It is much simpler to code (no rotations), and easier to make concurrent because an insert changes only a few local pointers. Redis sorted sets use a skip list (together with a hash table). Downside: probabilistic guarantees, more pointers per key, worse cache behaviour than arrays.

```js
class SkipList {
  constructor(maxLevel = 16, rnd = Math.random) {
    this.max = maxLevel; this.rnd = rnd;
    this.head = { key: -Infinity, next: new Array(maxLevel).fill(null) };
  }
  _level() { let l = 1; while (l < this.max && this.rnd() < 0.5) l++; return l; }
  _path(key) {                                  // last node < key on every level
    const upd = new Array(this.max); let cur = this.head;
    for (let i = this.max - 1; i >= 0; i--) {
      while (cur.next[i] && cur.next[i].key < key) cur = cur.next[i];
      upd[i] = cur;
    }
    return upd;
  }
  has(key) { const c = this._path(key)[0].next[0]; return !!c && c.key === key; }
  add(key) {
    const upd = this._path(key), c = upd[0].next[0];
    if (c && c.key === key) return false;
    const lvl = this._level(), node = { key, next: new Array(lvl).fill(null) };
    for (let i = 0; i < lvl; i++) { node.next[i] = upd[i].next[i]; upd[i].next[i] = node; }
    return true;
  }
  remove(key) {
    const upd = this._path(key), c = upd[0].next[0];
    if (!c || c.key !== key) return false;
    for (let i = 0; i < c.next.length; i++) upd[i].next[i] = c.next[i];
    return true;
  }
}
```

Tested with 2000 keys: all found, level 0 is sorted and has no duplicates, removals work. The `rnd` argument lets a test inject a fixed random source.

### 7. Bloom filter

A **Bloom filter** is a bit array of size `m` plus `k` hash functions. `add(x)` sets `k` bits. `mightHave(x)` checks those `k` bits: if any is 0, the answer is **definitely not present**; if all are 1, it is **probably present**.

- **No false negatives.** An added item is always reported present.
- **False positives are possible**: other items may have set exactly those bits.
- Cannot delete (a bit may belong to many items) and cannot list items. (A "counting" variant exists.)
- Uses a few bits per item, no matter how large the item is.

**False-positive math.** After inserting `n` items, the chance that a given bit is still 0 is `(1 - 1/m)^(kn)`, about `e^(-kn/m)`. A false positive needs all `k` probed bits to be 1:

```
p  ~  (1 - e^(-kn/m))^k
best k   = (m/n) * ln 2
m        = -n * ln(p) / (ln 2)^2        (bits needed for target rate p)
```

For `p = 1%`, that is about 9.6 bits per item and `k` about 7. We checked this in code: `n = 10000` gives `m = 95851`, `k = 7`, and a test with 100000 absent keys saw about 0.98% false positives.

```js
class Bloom {
  constructor(m, k) { this.m = m; this.k = k; this.bits = new Uint8Array(m); }
  _idx(s) {                                     // double hashing: h1 + i*h2
    let a = 2166136261, b = 5381;
    for (let i = 0; i < s.length; i++) { a = Math.imul(a ^ s.charCodeAt(i), 16777619) >>> 0; b = (Math.imul(b, 33) + s.charCodeAt(i)) >>> 0; }
    b |= 1;
    return Array.from({ length: this.k }, (_, i) => (a + Math.imul(i, b)) >>> 0).map(x => x % this.m);
  }
  add(s) { for (const i of this._idx(s)) this.bits[i] = 1; }
  mightHave(s) { return this._idx(s).every(i => this.bits[i]); }
}
const bloomParams = (n, p) => {
  const m = Math.ceil(-n * Math.log(p) / Math.LN2 ** 2);
  return { m, k: Math.round((m / n) * Math.LN2) };
};
```

Typical uses: "skip the disk read if the key is surely absent" (LSM-tree stores), a browser or crawler checking "have I seen this URL", a cache guarding against lookups for keys that do not exist. Note the filter is sized from the expected `n`; if you insert far more, the false-positive rate climbs.

### 8. Consistent hashing

**Problem:** spread keys over `N` cache servers. The naive way, `server = hash(key) % N`, breaks when `N` changes: almost every key maps to a new server, so the whole cache is cold. In a test with 10000 keys going from 4 to 5 servers, about 80% of keys moved with `%`.

**Idea:** put both servers and keys on a **ring** (positions 0 to 2^32-1). A key belongs to the **first server clockwise** from its position. Adding a server only takes keys from its neighbour on the ring; removing one hands its keys to the next. On average only about `1/N` of keys move.

**Virtual nodes:** with one point per server, the ring arcs are uneven, so load is lopsided. Give each server many points (`server#0`, `server#1`, ...). Arcs average out, and a failed server's load spreads over many others, not onto one neighbour.

```js
function hash32(s) { // FNV-1a then a mixer so similar strings spread
  let x = 2166136261;
  for (let i = 0; i < s.length; i++) x = Math.imul(x ^ s.charCodeAt(i), 16777619);
  x ^= x >>> 16; x = Math.imul(x, 0x85ebca6b); x ^= x >>> 13; x = Math.imul(x, 0xc2b2ae35); x ^= x >>> 16;
  return x >>> 0;
}
class Ring {
  constructor(vnodes = 100) { this.v = vnodes; this.ring = []; }   // sorted [pos, node]
  add(node) {
    for (let i = 0; i < this.v; i++) this.ring.push([hash32(node + '#' + i), node]);
    this.ring.sort((a, b) => a[0] - b[0]);
  }
  remove(node) { this.ring = this.ring.filter(([, n]) => n !== node); }
  get(key) {
    if (!this.ring.length) return undefined;
    const p = hash32(key);
    let lo = 0, hi = this.ring.length;               // first position >= p
    while (lo < hi) { const mid = (lo + hi) >> 1; if (this.ring[mid][0] < p) lo = mid + 1; else hi = mid; }
    return this.ring[lo % this.ring.length][1];      // wrap around
  }
}
```

Tested: 4 servers x 100 virtual nodes, 10000 keys, each server got between 15% and 35% of the load. Adding a fifth server moved about 25% of keys (and every moved key went to the new server), compared with about 80% for modulo. Removing it restores the old mapping exactly. Lookup is a binary search over sorted positions, O(log (N*v)). Used in distributed caches and Dynamo-style databases.

### 9. LRU vs LFU eviction

Both are cache policies built from simpler structures; the full coding problems are in [05 - Design Problems](../05-design-problems/README.md).

| | LRU (least recently used) | LFU (least frequently used) |
|---|---|---|
| Evicts | the item untouched for longest | the item with fewest accesses (ties: oldest) |
| Structure | hash map + doubly linked list (move to front on use) | hash map + a map of frequency -> ordered list, plus a `minFreq` pointer |
| get / put | O(1) | O(1) |
| Strength | adapts fast to changing access patterns, simple | keeps long-term popular items |
| Weakness | one big scan can flush the cache | old once-popular items stick around; needs aging |

In JS, a `Map` keeps insertion order, so a quick LRU is: on access `delete` then `set` the key again, and evict `map.keys().next().value`.

### 10. Trie vs hash map

| | Hash map of strings | Trie |
|---|---|---|
| Exact lookup | O(L) to hash the key, O(1) probes | O(L) |
| Prefix query ("all words starting with ab") | scan everything | walk L nodes, then collect |
| Sorted iteration | no | yes (children in order) |
| Memory | one entry per word | shared prefixes help, but each node carries child pointers; can be larger |
| Worst case | O(n) with bad hashing | no collisions |

Pick a trie for prefix search, autocomplete, longest-prefix match, word-grid search. Pick a hash map for plain exact lookups. See [05 - Trie](../../03-trees/05-trie/README.md).

### 11. Union-Find recap

Union-Find (disjoint set union) keeps a forest where each set is a tree with a root representative. `find` follows parents to the root; `union` links one root under another. Two tricks make it nearly constant: **path compression** (flatten paths during `find`) and **union by size or rank** (attach the smaller tree). Together the cost per operation is O(alpha(n)), the inverse Ackermann function, which is at most 4 or 5 for any input that fits in a computer. It answers "are a and b connected?" and "how many components?" but **cannot split** sets. See [06 - Union Find](../../04-graphs/06-union-find/README.md).

```js
function makeUF(n) {
  const parent = Array.from({ length: n }, (_, i) => i), size = new Array(n).fill(1);
  const find = (x) => { while (parent[x] !== x) { parent[x] = parent[parent[x]]; x = parent[x]; } return x; }; // path halving
  const union = (a, b) => {
    a = find(a); b = find(b);
    if (a === b) return false;
    if (size[a] < size[b]) [a, b] = [b, a];   // attach small under big
    parent[b] = a; size[a] += size[b];
    return true;
  };
  return { find, union };
}
```

### 12. Ropes and gap buffers (just know the idea)

Text editors cannot afford O(n) shifting for every keystroke in a big file.

- **Gap buffer:** an array with a movable empty gap at the cursor. Typing fills the gap in O(1); moving the cursor shifts the gap. Great when edits are local.
- **Rope:** a balanced tree whose leaves hold small string pieces. Concatenate and split are O(log n) instead of copying the whole string. Good for very large texts with edits anywhere.

JS strings are immutable. Repeated `s += x` in a loop is a known trap, so collect pieces in an array and `join` once.

### Which structure for which operation

Costs below are the usual textbook ones (average for hash, worst for the balanced structures). "n/a" means the structure does not support it efficiently.

| Need | Array (unsorted) | Sorted array | Hash table | Balanced BST | Heap | Trie | B+ tree |
|---|---|---|---|---|---|---|---|
| Lookup by key | O(n) | O(log n) | O(1) avg | O(log n) | O(n) | O(L) | O(log n) |
| Insert | O(1) amortized at end | O(n) | O(1) avg | O(log n) | O(log n) | O(L) | O(log n) |
| Delete by key | O(n) | O(n) | O(1) avg | O(log n) | O(n) (O(log n) with index map) | O(L) | O(log n) |
| Min / max | O(n) | O(1) | O(n) | O(log n) | O(1) | O(L) walk | O(log n) |
| Sorted iteration | no | O(n) | no | O(n) | no | O(total size) | O(n) via leaf list |
| Range query | O(n) | O(log n + k) | no | O(log n + k) | no | prefix only | O(log n + k) |
| Predecessor / successor | O(n) | O(log n) | no | O(log n) | no | O(L) | O(log n) |
| Prefix search | O(n L) | O(log n + k) | no | O(log n + k) | no | O(L + k) | O(log n + k) |

`L` = key length, `k` = number of results.

## Worked Interview Questions

### Q1. What happens inside `map.set(key, value)`?

**Answer:** The key is hashed to an integer. The integer is reduced to a bucket index (modulo capacity, or a mask for power-of-two sizes). The bucket is searched for an equal key (equality check, since different keys can share a hash). If found, the value is replaced; if not, a new entry is added. Then the load factor `size / capacity` is checked, and if it passed the threshold the table doubles and every entry is rehashed into the new array. All of this is O(1) expected.

### Q2. Chaining vs open addressing: how do you choose?

**Answer:** Chaining is simple, handles a load factor above 1, and deletion is easy, but it chases pointers and allocates per entry. Open addressing keeps everything in one array so it is cache friendly and compact, but it needs a load factor well below 1, suffers clustering (especially linear probing), and deletion needs tombstones. I would pick chaining for simplicity or when deletes are common, open addressing when memory layout and speed matter and the load factor is controlled.

### Q3. Why is hash table lookup O(1) and not O(n)?

**Answer:** It is O(1) **expected**, under two assumptions: the hash function spreads keys roughly evenly, and resizing keeps the load factor below a constant. Then each bucket holds a constant number of entries on average. The worst case is O(n) if everything collides. I would also mention that hashing a string costs O(length), and that systems use random seeds to resist collision attacks.

### Q4. Why is `push` on a dynamic array amortized O(1) even though a resize costs O(n)?

**Answer:** Resizes happen at sizes 1, 2, 4, ..., so the copy costs form a geometric series that sums to less than the final size. Over n pushes the total work is under 2n, so each push costs a constant on average. It only works because growth is multiplicative. Growing by a fixed amount each time would give O(n^2) total.

### Q5. A BST gives O(log n) operations, so why do we need AVL or red-black trees?

**Answer:** O(log n) holds only when the tree is balanced. Inserting sorted data into a plain BST makes a chain with height n, so operations become O(n). Self-balancing trees keep the height O(log n) with rotations, which are O(1) local changes that preserve the in-order order. AVL is stricter (faster lookups, more rotations); red-black is looser (fewer rotations on update).

### Q6. Why do databases use B+ trees instead of binary trees or hash tables?

**Answer:** Disk reads are page-sized and slow, so the goal is to minimise page reads. A B+ tree node fills a page and holds many keys, so the tree is very shallow (a handful of levels for billions of rows, with the top levels cached). Leaves are linked, so range scans and `ORDER BY` are sequential. Keeping data only in leaves keeps internal nodes small, giving higher fanout. A hash index cannot do range or ordered queries; a binary tree is too deep and causes many page reads.

### Q7. What is a Bloom filter and what can go wrong?

**Answer:** A bit array with `k` hash functions. Adding sets `k` bits; a query says "no" if any bit is 0 and "maybe" if all are 1. It never gives false negatives, but gives false positives with probability about `(1 - e^(-kn/m))^k`. About 9.6 bits per item gives roughly 1% false positives. It cannot delete or enumerate, and it degrades if you insert more than the planned `n`. I would use it to avoid expensive lookups for keys that are not there.

### Q8. Why does `hash(key) % N` fail when servers are added, and how does consistent hashing fix it?

**Answer:** Changing `N` changes the result for most keys, so most cached data is suddenly on the "wrong" server. Consistent hashing places servers and keys on a ring and assigns each key to the next server clockwise. Adding or removing a server only moves the keys in that server's arc, about `1/N` of all keys. Virtual nodes give each server many ring positions, which evens out load and spreads a failed server's keys over many servers.

### Q9. How do you implement LRU and LFU in O(1)?

**Answer:** LRU: a hash map from key to a node in a doubly linked list; on access move the node to the front, on overflow remove the tail. LFU: a hash map from key to node, plus a map from frequency to a linked list of keys with that frequency, plus the current minimum frequency. On access, move the key from list `f` to list `f+1` and update the minimum if list `f` became empty and was the minimum. Evict from the minimum-frequency list, oldest first.

### Q10. When would you choose a skip list over a balanced tree?

**Answer:** When simplicity and concurrency matter. A skip list has no rotations, and an insert changes only a few pointers, so fine-grained locking or lock-free versions are easier. The cost is probabilistic (not worst-case) O(log n) and more memory for the pointers. Redis sorted sets are a well-known example.

### Q11. Trie or hash map for a dictionary of words?

**Answer:** For exact membership only, a hash map is simpler and usually faster. If I need prefix queries, autocomplete, or sorted order, a trie, because prefix search becomes a walk of the prefix length and does not scan all words. The trade-off is memory: each trie node has child pointers.

### Q12. Why does `Map` iterate in insertion order, and how is it different from using an object?

**Answer:** The language specifies it, so the engine pairs its hash table with an order record. `Map` accepts any key type with SameValueZero equality (so `NaN` is a usable key), and has a reliable `size`. Plain objects only allow string and symbol keys, order integer-like keys first, and inherit from a prototype, so `Map` is the safer dictionary.

## Common Mistakes

- **"Hash tables are O(1)."** It is O(1) expected, with assumptions (good hash, bounded load factor). Say "average", and mention the worst case.
- **"Amortized means average over random inputs."** It means a worst-case guarantee over a sequence of operations, with no randomness.
- **"A BST is always O(log n)."** Only if balanced. Sorted insertion creates a chain.
- **"Bloom filters can say 'yes' for sure."** Only "no" is certain. "Yes" means "maybe".
- **"Consistent hashing needs no rebalancing."** A small fraction of keys still move; it is just about `1/N` instead of nearly all.
- **"B-tree and B+ tree are the same."** B+ keeps data only in leaves and links the leaves; that is why range scans are cheap.
- **"Rotations change the BST order."** They keep the in-order sequence; they only change the shape.
- **Using a mutable-looking object as a Map key and expecting content equality.** `Map` compares objects by reference.
- **Forgetting to rehash on resize.** The index depends on capacity, so entries cannot just be copied to the same index.

## Interview Tips

- Open with the analogy and the main trade-off in one sentence, then add details if asked. For a hash table: "array plus hash function, collisions handled by chaining or probing, resize at a load factor, so O(1) average."
- Always say **average vs worst case** for hash structures, **expected** for skip lists and Bloom filters, **amortized** for dynamic arrays.
- For "design X" follow-ups, name the structure and the operation it makes cheap, then the one it makes expensive.
- Know the follow-ups: How do you delete in open addressing (tombstones)? What if the hash is bad (degrades to O(n))? What happens during a resize (O(n) pause, can be done incrementally)? How do you pick a Bloom filter size (formula above)? How do you handle a hot key or a dead node in a ring (virtual nodes, replication)?
- Do not claim exact internals of a specific engine (V8's array growth factor, Java's tree threshold details) unless you are sure. Say "implementation detail, typically X".
- Mention JS has no built-in tree or heap; in a real interview, say which structure you would write and keep it minimal.
- Edge cases to mention: empty structure, duplicate keys (update vs ignore), resize at exactly the threshold, negative or huge hash values (use `>>> 0` or `Math.imul` in JS to stay in 32 bits), wrap-around on the ring.

## Practice List

Names only, LeetCode-style, for the structures in this chapter.

**Easy:** Design HashSet, Design HashMap, Design Underground System (hash map usage), Min Stack.

**Medium:** LRU Cache, Implement Trie (Prefix Tree), Insert Delete GetRandom O(1), Time Based Key-Value Store, Design Twitter.

**Hard:** LFU Cache, Design Skiplist, All O(1) Data Structure, Count of Smaller Numbers After Self (balanced tree / BIT), Sliding Window Median (two heaps or ordered structure), Design Search Autocomplete System, Number of Islands II (union-find).

## Key Takeaways

- A **hash table** is an array plus a hash function; collisions are handled by chaining or probing, and **resizing at a load factor** keeps operations O(1) on average.
- **Amortized O(1)** push comes from **multiplicative growth**: copy costs form a geometric series under 2n.
- **Balanced BSTs** keep height O(log n) with **rotations**; AVL is stricter, red-black does fewer updates.
- **B+ trees** minimise **page reads** with wide nodes and linked leaves, which is why databases use them for range-friendly indexes.
- **Bloom filters** trade exactness for space: **no false negatives, some false positives** (about 1% at roughly 9.6 bits per item).
- **Consistent hashing** with virtual nodes moves only about **1/N of keys** when a server joins or leaves.
- Pick the structure by the **operation you need cheap**: exact lookup (hash), ordered or range (tree / B+ tree), min or max (heap), prefix (trie), connectivity (union-find).

---

[← Previous: Design Problems](../05-design-problems/README.md) | [Next: Game Theory and Puzzles →](../07-game-theory-and-puzzles/README.md) | [Back to Advanced Index](../README.md)
