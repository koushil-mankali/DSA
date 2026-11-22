# Hashing (Hash Map and Hash Set)

> **Estimated Time:** 5 hr | **Difficulty:** Beginner to Intermediate
> **Prerequisites:** [02 - Complexity Analysis](../../00-foundations/02-complexity-analysis/README.md), [03 - JavaScript for DSA](../../00-foundations/03-javascript-for-dsa/README.md), [01 - Arrays](../01-arrays/README.md)

---

## What You'll Learn

- You can explain what a hash map and a hash set are, and why lookup is "O(1) on average".
- You can use `Map` and `Set` in JavaScript, and know when a plain object is a bad choice.
- You can turn a slow nested loop into a one-pass solution using "have I seen this before?" lookups.
- You can count things (frequency counting) and group things (by a key you design).
- You can solve the classic hashing interview problems: Two Sum, Group Anagrams, Top K Frequent, Longest Consecutive Sequence, Subarray Sum Equals K and more.

## The Big Idea

Think of a coat check at a theatre. You hand over your coat and get a numbered ticket. Later you show the ticket and the attendant walks straight to the right hook. They do not search every hook. The ticket number tells them where to go.

A hash map works the same way. You give it a **key** (like a name or a number). A **hash function** turns the key into a number, and that number points to a slot in an array. So "find this key" costs one jump, not a scan of everything.

- A **hash map** stores `key -> value` pairs. (`Map` in JavaScript.)
- A **hash set** stores only keys, no values. It answers "is this in here?". (`Set` in JavaScript.)

Why interviewers love it: the most common way to make a slow solution fast is to **trade memory for time**. Many O(n^2) brute-force answers become O(n) when you remember what you have already seen. Hashing is the main tool for that trade.

### Collisions, in plain words

Two different keys can land on the same slot. That is a **collision**. It is like two people with different coat tickets whose coats must share one hook. The map handles this by keeping a small list at that slot (or by probing for the next free slot) and checking the real keys inside it.

- If keys are spread out evenly, each slot holds about one item. Lookup is fast: O(1) **on average**.
- If a bad hash function or a bad input puts everything in one slot, lookup slowly degrades toward O(n). This is the **worst case**. It is rare in practice, and engines work hard to avoid it.
- When the table gets too full, it **resizes** (makes a bigger array and re-places everything). One insert may be slow, but the cost averages out. We call that "amortized O(1)".

In an interview, say "O(1) average for insert, lookup and delete". Do not say "always O(1)".

### Map vs Object in JavaScript

You can use a plain object `{}` as a counter, but `Map` is usually safer.

| Point | `Map` | Plain object `{}` |
|---|---|---|
| Key types | Any value (numbers, strings, arrays, objects) | Strings and Symbols only. Numbers become strings. |
| `1` vs `"1"` | Two different keys | The same key |
| Size | `map.size` | `Object.keys(o).length` |
| Order | Insertion order, always | Integer-like keys come first in ascending order, then others in insertion order |
| Inherited keys | None | `"constructor"`, `"toString"` exist via the prototype, so `"constructor" in {}` is `true` |
| Delete / add in loops | Built for it | Works, but slower and clumsier |

Two more facts worth knowing:

- Keys that are objects or arrays are compared **by reference**, not by content. `new Map([[[1], "x"]]).has([1])` is `false`. To use a "list" as a key, turn it into a string first, like `[1, 2].join(",")`.
- `Map` treats `NaN` as equal to `NaN`. That is handy.

Rule of thumb: use `Map` and `Set` by default. Use an object only for small, fixed, string-keyed data, or when an array of 26 counters is even better (see below).

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "Find two numbers that add to a target" | Map of value -> index, look for the complement |
| "Does any value appear twice?" / "is there a duplicate?" | `Set` |
| "How many times does each ... appear?" | Frequency map |
| "Are these two strings made of the same letters?" | Compare frequency counts |
| "Group items that are 'the same' in some way" | Map from a **canonical key** to a list |
| "Most frequent / top k" | Frequency map, then bucket or heap |
| "Longest run of consecutive numbers", and O(n) is required | Set, start only at run beginnings |
| "Count subarrays with sum k" | Prefix sum + map of earlier prefix sums |
| "One-to-one mapping between two sequences" | Two maps (forward and backward) |
| "Have I been in this state before?" (cycle) | `Set` of seen states |
| "Need an O(1) lookup while scanning" | Put already-seen things in a map |

## Core Concepts / Template

### Operations you must know

```js
// Map
const m = new Map();
m.set(key, value);        // insert or overwrite
m.get(key);               // value, or undefined
m.has(key);               // boolean
m.delete(key);            // remove
m.size;                   // number of entries
for (const [k, v] of m) {} // iterate in insertion order

// Set
const s = new Set([1, 2, 3]);
s.add(4); s.has(2); s.delete(1); s.size;
```

### Template 1: frequency counting

Count how many times each item appears. The `|| 0` trick gives a starting value when the key is new.

```js
function countFreq(arr) {
  const freq = new Map();
  for (const x of arr) freq.set(x, (freq.get(x) || 0) + 1);
  return freq;
}
```

If the input is only lowercase letters `a-z`, a fixed array of 26 numbers is faster and simpler than a map:

```js
const counts = new Array(26).fill(0);
counts[ch.charCodeAt(0) - 97]++; // 'a' -> 0, 'b' -> 1, ...
```

Caution: `x.get(k) || 0` is fine for counts. It is a bug if a stored value can legitimately be `0` or `""` and you need to tell that apart from "missing". Use `has` in that case.

### Template 2: "have I seen it?" in one pass

Scan once. For each item, ask the map about what you need from the past. Then add the current item.

```js
function oneScan(arr) {
  const seen = new Map();            // value -> index (or count, or anything useful)
  for (let i = 0; i < arr.length; i++) {
    // 1. ask: does what I need already exist in `seen`?
    // 2. then record arr[i] in `seen`
  }
}
```

The order matters: **check first, then insert**. If you insert first, an element can match itself (a classic Two Sum bug).

### Template 3: group by a canonical key

Pick a key that is identical for all items you want grouped. Then push each item into `map.get(key)`.

```js
function groupBy(items, keyOf) {
  const groups = new Map();
  for (const it of items) {
    const k = keyOf(it);
    if (!groups.has(k)) groups.set(k, []);
    groups.get(k).push(it);
  }
  return [...groups.values()];
}
```

### Template 4: prefix sum + map

A subarray sum is `prefix[j] - prefix[i]`. So "sum equals k" means "a prefix equal to `current - k` happened earlier". Store how many times (or the first index where) each prefix occurred. Start the map with `{0 -> 1}` (or `{0 -> -1}` for indices) so subarrays that start at index 0 are counted.

### Step-by-step trace: Two Sum with a map

`nums = [2, 7, 11, 15]`, `target = 9`

| i | nums[i] | need = 9 - nums[i] | in seen? | seen after |
|---|---|---|---|---|
| 0 | 2 | 7 | no | {2:0} |
| 1 | 7 | 2 | yes, index 0 | return [0, 1] |

One pass, no inner loop.

## Problems (easy -> hard)

### 1. Contains Duplicate — Easy
**Companies:** Amazon, Apple, Microsoft
**Problem:** Return `true` if any value appears at least twice. Example: `[1,2,3,1] -> true`, `[1,2,3] -> false`.
**Intuition:** A `Set` remembers what you have seen. If you meet something already in it, you found a duplicate.
**Approach 1 - Brute force:** Compare every pair. Time O(n^2), Space O(1).
**Approach 2 - Better:** Sort, then check neighbours. Time O(n log n), Space O(1) or O(n) depending on the sort.
**Approach 3 - Optimal:** One pass with a `Set`. For `[1,2,3,1]`: add 1, add 2, add 3, then 1 is already there -> `true`.

```js
function containsDuplicate(nums) {
  const seen = new Set();
  for (const x of nums) {
    if (seen.has(x)) return true;
    seen.add(x);
  }
  return false;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Empty array returns `false`. Sorting in place changes the caller's array; the set does not.

### 2. Valid Anagram — Easy
**Companies:** Amazon, Meta, Microsoft
**Problem:** Are two strings made of exactly the same letters with the same counts? Example: `"anagram", "nagaram" -> true`; `"rat", "car" -> false`.
**Intuition:** Anagrams have identical letter counts. Count up for one string, count down for the other. If everything ends at 0, they match.
**Approach 1 - Brute force:** Sort both strings and compare. Time O(n log n), Space O(n).
**Approach 2 - Better:** Two frequency maps, then compare. Time O(n), Space O(1) for a fixed alphabet.
**Approach 3 - Optimal:** One array of 26 counters. +1 for `s`, -1 for `t`. Check all zero. For `"rat"` vs `"car"`: r+1, a+1, t+1, then c-1, a-1 (back to 0), r-1 (back to 0). `t` stays at 1 and `c` is -1 -> not all zero -> `false`.

```js
function isAnagram(s, t) {
  if (s.length !== t.length) return false; // quick exit, also keeps the loop safe
  const c = new Array(26).fill(0);
  for (let i = 0; i < s.length; i++) {
    c[s.charCodeAt(i) - 97]++;
    c[t.charCodeAt(i) - 97]--;
  }
  return c.every((x) => x === 0);
}
```

**Complexity:** Time O(n), Space O(1) (26 counters).
**Watch out:** This code assumes lowercase `a-z`. For Unicode input (the usual follow-up), use a `Map` instead of the 26-array. Check lengths first.

### 3. Two Sum — Easy
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Return the indices of the two numbers that add to `target`. Exactly one answer exists and you may not use the same element twice. Example: `[2,7,11,15], 9 -> [0,1]`.
**Intuition:** For each number `x`, you only need to know whether `target - x` appeared earlier. That is a lookup, so use a map.
**Approach 1 - Brute force:** Try all pairs. Time O(n^2), Space O(1).
**Approach 2 - Better:** Sort and use two pointers. Time O(n log n). You lose original indices unless you store them, so it is clumsy here.
**Approach 3 - Optimal:** Map of `value -> index`. Check the complement **before** inserting the current value. For `[3,2,4], 6`: i=0 need 3, not seen, store 3. i=1 need 4, store 2. i=2 need 2, found at index 1 -> `[1,2]`.

```js
function twoSum(nums, target) {
  const seen = new Map(); // value -> index
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
  return [];
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** `[3,3], 6` works because you check before inserting. Insert-first would pair `3` with itself in `[3,2,4], 6` and give `[0,0]`. Use `seen.has`, not `if (seen.get(need))`, because index `0` is falsy.

### 4. First Unique Character in a String — Easy
**Companies:** Amazon, Meta, Bloomberg
**Problem:** Return the index of the first character that appears only once, or `-1`. Example: `"loveleetcode" -> 2`.
**Intuition:** You cannot know if a letter is unique until you have seen the whole string. So count first, then scan again in original order.
**Approach 1 - Brute force:** For each char, scan the rest of the string. Time O(n^2), Space O(1).
**Approach 2 - Better:** (the optimal one below is already simple)
**Approach 3 - Optimal:** Pass 1 builds counts. Pass 2 returns the first index whose count is 1. For `"loveleetcode"`: counts l2 o2 v1 e4 t1 c1 d1. Scanning: l(2), o(2), v(1) at index 2 -> return 2.

```js
function firstUniqChar(s) {
  const c = new Map();
  for (const ch of s) c.set(ch, (c.get(ch) || 0) + 1);
  for (let i = 0; i < s.length; i++) if (c.get(s[i]) === 1) return i;
  return -1;
}
```

**Complexity:** Time O(n), Space O(k) where k is the alphabet size (O(1) for a fixed alphabet).
**Watch out:** Scan the **string** in the second pass, not the map. Map order is first-seen order of keys, and this also works, but scanning the string is clearer and never wrong. Empty string returns `-1`.

### 5. Isomorphic Strings — Easy
**Companies:** -
**Problem:** Can you replace characters in `s` to get `t`, where each character maps to exactly one other character, and no two characters map to the same one? Example: `"egg","add" -> true`; `"badc","baba" -> false`.
**Intuition:** The mapping must be one-to-one in **both** directions. One map is not enough: `"badc"` to `"baba"` has `b->b, a->a, d->b, c->a`. Each source char has one target, but two sources share a target.
**Approach 1 - Brute force:** Compare the "first occurrence index pattern" of both strings by building those arrays. Works, same complexity.
**Approach 2 - Better:** (same idea as optimal)
**Approach 3 - Optimal:** Keep `s->t` and `t->s` maps. At each position, if either map already has an entry that disagrees, return `false`. For `"foo","bar"`: f->b, o->a, then o must map to r but already maps to a -> `false`.

```js
function isIsomorphic(s, t) {
  if (s.length !== t.length) return false;
  const st = new Map(), ts = new Map();
  for (let i = 0; i < s.length; i++) {
    const a = s[i], b = t[i];
    if (st.has(a) && st.get(a) !== b) return false;
    if (ts.has(b) && ts.get(b) !== a) return false;
    st.set(a, b);
    ts.set(b, a);
  }
  return true;
}
```

**Complexity:** Time O(n), Space O(k) for the alphabet.
**Watch out:** Forgetting the reverse map. Different lengths must return `false` early.

### 6. Word Pattern — Easy
**Companies:** -
**Problem:** Does a string of words follow a pattern of letters, with a one-to-one match between letters and words? Example: `"abba", "dog cat cat dog" -> true`; `"abba", "dog dog dog dog" -> false`.
**Intuition:** This is Isomorphic Strings again, with words in place of the second string's characters.
**Approach 1 - Brute force:** Not needed.
**Approach 2 - Better:** Not needed.
**Approach 3 - Optimal:** Split into words, check that the counts match, then use the same two-map check. For `"abba"` vs `"dog dog dog dog"`: a->dog, b->dog is rejected because `dog` already maps back to `a`.

```js
function wordPattern(pattern, s) {
  const w = s.split(" ");
  if (w.length !== pattern.length) return false;
  const pw = new Map(), wp = new Map();
  for (let i = 0; i < w.length; i++) {
    const p = pattern[i];
    if (pw.has(p) && pw.get(p) !== w[i]) return false;
    if (wp.has(w[i]) && wp.get(w[i]) !== p) return false;
    pw.set(p, w[i]);
    wp.set(w[i], p);
  }
  return true;
}
```

**Complexity:** Time O(n + total word length), Space O(n).
**Watch out:** Check `w.length !== pattern.length` first. Do not compare words with `==` on a joined string; compare each word.

### 7. Intersection of Two Arrays II — Easy
**Companies:** Amazon, Meta
**Problem:** Return the elements common to both arrays, counting duplicates (each value appears as many times as it shows in both). Example: `[1,2,2,1], [2,2] -> [2,2]`.
**Intuition:** Count one array. Walk the other one, and each time you meet a value with a count above 0, take it and decrement.
**Approach 1 - Brute force:** For each item in `a`, search `b` and mark the match used. Time O(n*m), Space O(m).
**Approach 2 - Better:** Sort both and use two pointers. Time O(n log n + m log m), Space O(1) extra. Good if the arrays are already sorted or too big for memory.
**Approach 3 - Optimal:** Frequency map of the smaller array, then scan the other. For `[4,9,5]` and `[9,4,9,8,4]`: counts {4:1,9:1,5:1}. Scan: 9 take (9 left 0), 4 take, 9 count is 0 skip, 8 not there, 4 count is 0 skip -> `[9,4]`.

```js
function intersect(a, b) {
  const c = new Map();
  for (const x of a) c.set(x, (c.get(x) || 0) + 1);
  const out = [];
  for (const x of b) {
    if ((c.get(x) || 0) > 0) {
      out.push(x);
      c.set(x, c.get(x) - 1);
    }
  }
  return out;
}
```

**Complexity:** Time O(n + m), Space O(min(n, m)) if you count the smaller array (as written it counts `a`, so swap if `a` is bigger).
**Watch out:** Output order is not specified. For the "Intersection of Two Arrays I" version (unique values only) just use two sets.

### 8. Happy Number — Easy
**Companies:** Amazon, Microsoft
**Problem:** Replace a number with the sum of the squares of its digits, again and again. It is "happy" if it reaches 1. Otherwise it loops forever. Example: `19 -> true` (1+81=82, 64+4=68, 36+64=100, 1).
**Intuition:** It either hits 1 or enters a cycle. A `Set` of seen values detects the cycle: if the next value is already in the set, you are looping.
**Approach 1 - Brute force:** Run a fixed large number of steps and see if you reach 1. Works but is a guess.
**Approach 2 - Better:** `Set` of seen numbers (below). Time and space are tiny because values stay small.
**Approach 3 - Optimal:** Floyd's slow/fast pointers detect the cycle with O(1) space (see the Linked List chapter for the idea). Trace for `2`: 2, 4, 16, 37, 58, 89, 145, 42, 20, 4 -> 4 seen again -> `false`.

```js
function isHappy(n) {
  const seen = new Set();
  while (n !== 1 && !seen.has(n)) {
    seen.add(n);
    let s = 0;
    while (n > 0) {
      const d = n % 10;
      s += d * d;
      n = Math.floor(n / 10);
    }
    n = s;
  }
  return n === 1;
}
```

**Complexity:** Time O(log n) per step with only a few steps (the sequence stays small after the first step), Space O(log n) for the set.
**Watch out:** Use `Math.floor(n / 10)`, not `n / 10` (that gives decimals in JS). The loop ends either at 1 or at a repeat.

### 9. Group Anagrams — Medium
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** Group strings that are anagrams of each other. Example: `["eat","tea","tan","ate","nat","bat"] -> [["bat"],["nat","tan"],["ate","eat","tea"]]` (any group order).
**Intuition:** You need one key that is the same for every anagram. Either the sorted letters, or the 26 letter counts.
**Approach 1 - Brute force:** Compare every string with every group using Valid Anagram. Time O(n^2 * k).
**Approach 2 - Better:** Key = sorted string. `"eat"`, `"tea"`, `"ate"` all become `"aet"`. Time O(n * k log k), Space O(n * k).
**Approach 3 - Optimal:** Key = the 26 counts joined into a string. No sorting. For `"eat"`: the counts array has 1 at a, e, t, so the key is `1#0#0#0#1#...`. Same for `"tea"` and `"ate"`. So they land in the same bucket.

```js
function groupAnagrams(strs) {
  const m = new Map();
  for (const s of strs) {
    const c = new Array(26).fill(0);
    for (let i = 0; i < s.length; i++) c[s.charCodeAt(i) - 97]++;
    const key = c.join("#"); // arrays are compared by reference, so make it a string
    if (!m.has(key)) m.set(key, []);
    m.get(key).push(s);
  }
  return [...m.values()];
}
```

**Complexity:** Time O(n * k) where k is the longest string length, Space O(n * k).
**Watch out:** Use a separator when joining counts. Without it, counts `1,11` and `11,1` both become `"111"`. `[""]` returns `[[""]]`. Never use the array itself as a map key.

### 10. Top K Frequent Elements — Medium
**Companies:** Amazon, Meta, Google
**Problem:** Return the `k` most frequent values. Example: `[1,1,1,2,2,3], k=2 -> [1,2]`.
**Intuition:** Count first. Then, since a count can never exceed `n`, put each value into a **bucket indexed by its count** and read buckets from the highest count down. No sorting needed.
**Approach 1 - Brute force:** Count, then sort the entries by count. Time O(n log n).
**Approach 2 - Better:** Min-heap of size `k` over the counts. Time O(n log k). (Heaps are covered in the Heaps chapter.)
**Approach 3 - Optimal:** Bucket sort. For `[1,1,1,2,2,3]`: counts {1:3, 2:2, 3:1}. Buckets: index 3 -> [1], index 2 -> [2], index 1 -> [3]. Read from index 6 down: skip empties, take 1, take 2, stop at `k=2`.

```js
function topKFrequent(nums, k) {
  const c = new Map();
  for (const x of nums) c.set(x, (c.get(x) || 0) + 1);
  const buckets = Array.from({ length: nums.length + 1 }, () => []);
  for (const [v, f] of c) buckets[f].push(v);
  const out = [];
  for (let f = buckets.length - 1; f > 0 && out.length < k; f--) {
    for (const v of buckets[f]) {
      out.push(v);
      if (out.length === k) break;
    }
  }
  return out;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Allocate `n + 1` buckets (a value can appear `n` times). Use `Array.from({length}, () => [])`, not `new Array(n).fill([])`, which shares one array everywhere. Mention the heap version when the interviewer asks "what if the data is a stream?".

### 11. Longest Consecutive Sequence — Medium
**Companies:** Google, Amazon, Meta
**Problem:** Given an unsorted array, find the length of the longest run of consecutive integers (like 1,2,3,4). It must run in O(n). Example: `[100,4,200,1,3,2] -> 4`.
**Intuition:** Sorting would give O(n log n). Instead, put everything in a set. Only start counting from a number that is the **beginning** of a run, meaning `x - 1` is not in the set. Then walk up with `x+1, x+2, ...`.
**Approach 1 - Brute force:** For each number, walk up and check membership with a linear scan. Time O(n^3) worst case.
**Approach 2 - Better:** Sort, then count runs (skip duplicates). Time O(n log n), Space O(1) or O(n).
**Approach 3 - Optimal:** Set + start only at run beginnings. For `[100,4,200,1,3,2]`: 100 starts (99 absent), run length 1. 4 has 3 present, skip. 200 starts, length 1. 1 starts (0 absent): 1,2,3,4 -> length 4. 3 and 2 skip.

```js
function longestConsecutive(nums) {
  const s = new Set(nums);
  let best = 0;
  for (const x of s) {
    if (s.has(x - 1)) continue; // not the start of a run
    let y = x;
    while (s.has(y + 1)) y++;
    best = Math.max(best, y - x + 1);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(n). The inner `while` runs only from run starts, so each number is walked once in total.
**Watch out:** Without the `has(x - 1)` skip, a long run is re-walked from every element and the time becomes O(n^2). Loop over the **set**, not the array, so duplicates do not waste time. Empty input returns 0.

### 12. Subarray Sum Equals K — Medium
**Companies:** Meta, Amazon, Google
**Problem:** Count the contiguous subarrays whose sum is `k`. Numbers may be negative. Example: `[1,1,1], k=2 -> 2`.
**Intuition:** Let `sum` be the running total up to now. A subarray ending here has sum `k` exactly when some earlier prefix equals `sum - k`. So keep a map of "prefix sum -> how many times seen".
**Approach 1 - Brute force:** Try every start and end. Time O(n^2) (or O(n^3) if you re-sum).
**Approach 2 - Better:** Sliding window does **not** work, because negatives break the "grow or shrink" logic. (It works only when all numbers are positive.)
**Approach 3 - Optimal:** Prefix sum + map. Seed with `{0: 1}` for the empty prefix. For `[1,1,1], k=2`:

| x | sum | sum-k | map count of (sum-k) | total | map after |
|---|---|---|---|---|---|
| start | 0 | | | 0 | {0:1} |
| 1 | 1 | -1 | 0 | 0 | {0:1, 1:1} |
| 1 | 2 | 0 | 1 | 1 | {0:1, 1:1, 2:1} |
| 1 | 3 | 1 | 1 | 2 | {..., 3:1} |

```js
function subarraySum(nums, k) {
  const m = new Map([[0, 1]]); // empty prefix, so subarrays starting at index 0 count
  let sum = 0, cnt = 0;
  for (const x of nums) {
    sum += x;
    cnt += m.get(sum - k) || 0; // look up BEFORE adding the current sum
    m.set(sum, (m.get(sum) || 0) + 1);
  }
  return cnt;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Forgetting the `{0: 1}` seed. Updating the map before the lookup (wrong when `k = 0`). `[1,-1,0]` with `k=0` gives 3: `[1,-1]`, `[0]`, `[1,-1,0]`.

### 13. Longest Subarray with Sum K (and Zero-Sum) — Medium
**Companies:** -
**Problem:** Find the **length** of the longest subarray with sum `k`. Negatives allowed. Example: `[1,-1,5,-2,3], k=3 -> 4` (`[1,-1,5,-2]`). The zero-sum version is the same with `k = 0`: `[15,-2,2,-8,1,7,10,23] -> 5` (`[-2,2,-8,1,7]`).
**Intuition:** Same prefix idea as problem 12, but now you want the **longest**, so store the **first** index where each prefix sum appeared. The earliest start gives the longest length.
**Approach 1 - Brute force:** All start/end pairs. Time O(n^2), Space O(1).
**Approach 2 - Better:** Sliding window only for non-negative numbers. Does not work with negatives.
**Approach 3 - Optimal:** Map `prefix -> first index`, seeded with `{0: -1}`. At index `i`, if `sum - k` is in the map, a candidate length is `i - first[sum - k]`. Only store a prefix if it is **new** (keep the earliest). For `[1,-1,5,-2,3], k=3`: sums 1,0,5,3,6. At i=3, sum=3, so `sum - k = 0`. Prefix 0 first appears at index -1 (the seed), so length = 3 - (-1) = 4.

```js
function longestSubarraySumK(nums, k) {
  const first = new Map([[0, -1]]); // prefix sum -> earliest index
  let sum = 0, best = 0;
  for (let i = 0; i < nums.length; i++) {
    sum += nums[i];
    if (first.has(sum - k)) best = Math.max(best, i - first.get(sum - k));
    if (!first.has(sum)) first.set(sum, i); // keep the earliest, never overwrite
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Overwriting an existing prefix with a later index shortens answers. For `[0,0,0]` with `k=0` the answer is 3, which depends on the `-1` seed and on keeping the first index. Return 0 when nothing matches.

### 14. 4Sum II (Count A+B+C+D = 0) — Medium
**Companies:** -
**Problem:** Given four arrays of equal length `n`, count the tuples `(i, j, k, l)` with `A[i] + B[j] + C[k] + D[l] == 0`. Example: `A=[1,2], B=[-2,-1], C=[-1,2], D=[0,2] -> 2`.
**Intuition:** Four loops is O(n^4). Split in two halves: all sums of `A+B` go into a map with counts. Then for every `C+D`, look up its negative. That is "Two Sum" on pair sums.
**Approach 1 - Brute force:** Four nested loops. Time O(n^4).
**Approach 2 - Better:** Three loops plus a set for D. Time O(n^3).
**Approach 3 - Optimal:** Meet in the middle. Two double loops, one map. For the example, `A+B` sums: -1, 0, 0, 1 -> map {-1:1, 0:2, 1:1}. Then `C+D`: -1+0=-1 (need 1, count 1), -1+2=1 (need -1, count 1), 2+0=2 (need -2, 0), 2+2=4 (need -4, 0). Total 2.

```js
function fourSumCount(A, B, C, D) {
  const m = new Map();
  for (const a of A) for (const b of B) m.set(a + b, (m.get(a + b) || 0) + 1);
  let count = 0;
  for (const c of C) for (const d of D) count += m.get(-(c + d)) || 0;
  return count;
}
```

**Complexity:** Time O(n^2), Space O(n^2).
**Watch out:** It counts **index tuples**, not unique value tuples, so duplicates count separately. That is why the map stores counts. Do not confuse with classic 4Sum (one array, unique quadruplets), which uses sorting and two pointers.

### 15. Count Distinct Elements in Every Window of Size k — Medium
**Companies:** -
**Problem:** For every contiguous window of size `k`, report how many distinct values it has. Example: `[1,2,1,3,4,2,3], k=4 -> [3,4,4,3]`.
**Intuition:** Slide the window by one step. Add the new element to a frequency map. Remove the old element, and delete its key when its count hits 0. Then `map.size` is the answer for the window.
**Approach 1 - Brute force:** For every window, build a set. Time O(n * k).
**Approach 2 - Better:** (the sliding map below is the standard one)
**Approach 3 - Optimal:** Maintain one frequency map as the window slides. For `[1,2,1,3,4,2,3]`, k=4: window `[1,2,1,3]` -> map {1:2, 2:1, 3:1}, size 3. Slide: add 4, remove the old 1 -> {1:1,2:1,3:1,4:1}, size 4. Slide: add 2, remove the old 2 (count 2 -> 1) -> size 4. Slide: add 3, remove the last 1 (count 0, delete key) -> {2:1,3:2,4:1}, size 3. Result `[3,4,4,3]`.

```js
function distinctInWindows(nums, k) {
  const c = new Map(), out = [];
  for (let i = 0; i < nums.length; i++) {
    c.set(nums[i], (c.get(nums[i]) || 0) + 1);
    if (i >= k) {
      const old = nums[i - k]; // element leaving the window
      if (c.get(old) === 1) c.delete(old); // delete at 0, or size is wrong
      else c.set(old, c.get(old) - 1);
    }
    if (i >= k - 1) out.push(c.size);
  }
  return out;
}
```

**Complexity:** Time O(n), Space O(k).
**Watch out:** Forgetting to delete a key at count 0 makes `size` too big. If `k > n` there are no windows, so the output is `[]`. More on windows in the Sliding Window chapter.

### 16. Copy List with Random Pointer — Medium
**Companies:** Amazon, Meta, Microsoft, Google
**Problem:** A linked list node has `next` and `random` (which can point to any node or `null`). Make a deep copy. Example: a 3-node list where `random` jumps around -> a new 3-node list with the same shape and no shared nodes.
**Intuition:** The hard part is that `random` may point to a node you have not copied yet. A map from **old node -> new node** solves it: first make all copies, then wire the pointers by looking each target up.
**Approach 1 - Brute force:** For each node, walk the list to find the index of its random target, then rebuild. Time O(n^2).
**Approach 2 - Better:** Two passes with a map (below). Space O(n).
**Approach 3 - Optimal:** In-place weaving: insert each copy right after its original, set `copy.random = orig.random.next`, then split the lists. Time O(n), Space O(1) extra. Know it exists; the map version is usually accepted and is easier to get right. Trace of the map version: pass 1 creates `A'`, `B'`, `C'` and stores `A->A'`, `B->B'`, `C->C'`. Pass 2: `A'.next = map.get(B) = B'`, `A'.random = map.get(C) = C'`, and so on.

```js
class Node {
  constructor(val) { this.val = val; this.next = null; this.random = null; }
}

function copyRandomList(head) {
  if (!head) return null;
  const m = new Map(); // old node -> new node (nodes are keys, compared by reference)
  for (let p = head; p; p = p.next) m.set(p, new Node(p.val));
  for (let p = head; p; p = p.next) {
    m.get(p).next = m.get(p.next) || null;     // m.get(null) is undefined -> null
    m.get(p).random = m.get(p.random) || null;
  }
  return m.get(head);
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Here using objects as keys is the point: a `Map` keyed by node reference works, a plain `{}` would turn nodes into the string `"[object Object]"`. Handle `null` for both `next` and `random`. Empty list returns `null`.

### 17. 3Sum — hash vs two pointers (mention only) — Medium
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** Find all unique triplets in an array that sum to 0. Example: `[-1,0,1,2,-1,-4] -> [[-1,-1,2],[-1,0,1]]`.
**Intuition:** Fix one number `a`, then you need Two Sum for `-a` in the rest.
**Approach 1 - Brute force:** Three loops plus a set to remove duplicate triplets. Time O(n^3).
**Approach 2 - Better:** Fix `a`, then use a hash set for the Two Sum part. Time O(n^2), Space O(n). The pain is **de-duplicating** the triplets, which needs extra care.
**Approach 3 - Optimal:** Sort, then fix `a` and move two pointers inward. Time O(n^2), Space O(1) extra. Skipping equal neighbours gives unique triplets for free. This is the version interviewers expect. Full code and trace are in the [Two Pointers chapter](../05-two-pointers/README.md).

```js
// Hash-flavoured idea (sketch, not the expected final answer):
// for each i: run Two Sum on the rest with a Set for -nums[i],
// then store triplets as a sorted "a,b,c" string in a Set to drop duplicates.
```

**Complexity:** Time O(n^2) for both the hash and the two-pointer way.
**Watch out:** Say out loud why the sorted two-pointer solution wins: no extra memory, and duplicates are easy to skip. The hash version is a good fallback when sorting is not allowed.

## Common Mistakes

- **"A Map and an object are the same."** They are not. Object keys become strings (so `1` and `"1"` collide), and inherited names like `constructor` exist. Use `Map` and `Set`.
- **"I can use an array as a key."** Arrays and objects are compared by reference. `m.has([1])` is false even after `m.set([1], ...)`. Convert to a string like `arr.join("#")` first.
- **"Lookups are always O(1)."** It is O(1) on **average**. Worst case with many collisions is O(n). Say "average".
- **"Insert first, then check."** In Two Sum this lets an element match itself. Check first, insert after.
- **`if (map.get(k))` to test presence.** A stored `0` is falsy. Use `map.has(k)` when `0` or `""` can be a real value.
- **Forgetting the `{0: 1}` (or `{0: -1}`) seed** in prefix-sum problems. Subarrays that start at index 0 are lost.
- **Using sliding window when negatives exist.** Window logic needs non-negative numbers. Use prefix sum + map instead.
- **Joining counts with no separator** when building an anagram key. `[1, 11]` and `[11, 1]` become the same string.
- **Modifying a Map while looping** over it carelessly. Deleting is safe in `for...of`, but adding keys can make the loop visit them too.

## Interview Tips

- Start by saying the brute force and its cost. Then say: "I can lower the time by storing what I have seen in a hash map, at the cost of O(n) space." That sentence shows the trade-off you are making.
- Say what the **key** is and what the **value** is before coding (for example "key is the value, value is its index"). Interviewers like clear data design.
- Always state "average O(1)" for map operations. If asked about the worst case, explain collisions in one sentence.
- Edge cases to mention: empty input, one element, all duplicates, negative numbers, zero, very large `n`, Unicode strings (for anagrams).
- Common follow-ups:
  - "What if the array is already sorted?" -> two pointers with O(1) space.
  - "What if memory is tight?" -> sorting, or two pointers, or Bloom-filter style ideas for "probably seen" checks.
  - "What if the data is a stream?" -> keep a running map, and for top-k use a min-heap of size `k`.
  - "What if keys are not hashable / are arrays?" -> build a string key.
  - "Can you do it in O(1) space?" -> usually sort first, or use the problem's special structure (like Floyd's cycle detection for Happy Number).
- Mention that JS `Map` keeps insertion order. That makes the output deterministic and is why `[...m.values()]` works.

### Quick interview questions

1. **How does a hash map find a key quickly?** A hash function turns the key into an array index. You jump there and compare keys. A good hash spreads keys so each slot has few items.
2. **What happens on a collision?** Two keys share a slot. The map keeps both (in a small list, or by probing for another slot) and compares the real keys. Lookups stay near O(1) as long as slots stay lightly loaded.
3. **Map or object for counting?** `Map`. Keys keep their type, there are no inherited keys, and `size` is built in.

## Practice List

**Easy**
- Two Sum
- Contains Duplicate
- Valid Anagram
- Ransom Note
- First Unique Character in a String
- Isomorphic Strings
- Word Pattern
- Intersection of Two Arrays
- Intersection of Two Arrays II
- Happy Number
- Majority Element
- Single Number
- Find the Difference
- Jewels and Stones
- Contains Duplicate II

**Medium**
- Group Anagrams
- Top K Frequent Elements
- Top K Frequent Words
- Longest Consecutive Sequence
- Subarray Sum Equals K
- Continuous Subarray Sum
- Subarray Sums Divisible by K
- Contiguous Array
- 4Sum II
- Sort Characters By Frequency
- Find All Anagrams in a String
- Encode and Decode Strings
- Design HashMap
- Copy List with Random Pointer
- Brick Wall
- Longest Substring Without Repeating Characters

**Hard**
- Substring with Concatenation of All Words
- Minimum Window Substring
- Max Points on a Line
- First Missing Positive
- Longest Substring with At Most K Distinct Characters
- Count of Range Sum

## Key Takeaways

- A hash map or set gives **average O(1)** insert, lookup and delete. It trades **memory for time**.
- Collisions are two keys on one slot. Good hashing and resizing keep them rare; the worst case is O(n), so say "average".
- In JavaScript prefer **`Map` and `Set`**. Objects turn keys into strings and carry inherited names. Arrays and objects as keys compare by **reference**.
- **Check first, then insert** in "have I seen the complement?" problems.
- For **grouping**, design a **canonical key** (sorted letters, a counts string) that is equal for all items that belong together.
- For **subarray sums with negatives**, use **prefix sum + map**. Seed with `{0: 1}` for counting and `{0: -1}` for longest, and store the **first** index for longest.
- Hash lookups work as an alternative to two pointers (like 3Sum), but **sorting + two pointers** often saves memory and makes de-duplication easy.

---

[← Previous: Sorting](../02-sorting/README.md) | [Next: Strings →](../04-strings/README.md) | [Back to Core Linear Index](../README.md)
