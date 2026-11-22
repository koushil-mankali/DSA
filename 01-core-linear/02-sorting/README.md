# Sorting

> **Estimated Time:** 4 hr | **Difficulty:** Intermediate
> **Prerequisites:** [00 - Complexity Analysis](../../00-foundations/02-complexity-analysis/README.md), [01 - Arrays](../01-arrays/README.md)

---

## What You'll Learn

- You can write selection, bubble, insertion, merge, quick and heap sort from memory and explain their cost.
- You can pick counting, radix or bucket sort when a comparison sort is not the best tool.
- You can say which sorts are stable and in-place, and why that matters.
- You can use sorting as a tool: custom comparators, "sort then scan", and quickselect.
- You can avoid the classic JavaScript `Array.prototype.sort` traps.

## The Big Idea

Think of a teacher who has to line up 30 children by height. She can pick the shortest child, move her to the front, then pick the next shortest, and so on (selection sort). Or she can split the class in two, let each half line up alone, and then join the two lines by always taking the shorter front child (merge sort). Or she can pick one child as a marker, send shorter kids left and taller kids right, and repeat for each side (quick sort). Same goal, very different work.

Sorting means putting items in an order so that later work becomes easy. Once data is sorted you can binary search it, find duplicates by looking at neighbours, merge overlapping intervals, or pair up values with two pointers. Many "hard looking" problems become "sort, then one pass".

Interviewers love sorting because:

- It tests whether you really understand time and space trade-offs, not just the name of an algorithm.
- Merge sort and quick sort are the two classic divide and conquer examples.
- Variations (count inversions, kth largest, custom order) test if you can adapt a known algorithm.
- It is also a trap for JavaScript users: the default `sort()` compares as strings.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "sort", "order", "arrange so that ..." | Sort first, then scan. Often O(n log n) is fine. |
| "merge two sorted ..." | Two pointers. Often fill from the back to save space. |
| Values are only 0, 1, 2 (or a tiny fixed set) | Counting or Dutch National Flag, O(n). |
| Array holds numbers in range 1..n (or 0..n) and you must find missing or duplicate | Cyclic sort (put each value at its own index). |
| "kth largest / smallest", "median" | Quickselect (average O(n)) or a heap. |
| "count pairs (i < j) where a[i] vs a[j]" | Merge sort and count while merging. |
| "arrange numbers/strings to form the biggest/smallest ..." | Custom comparator. |
| Intervals, meetings, "overlap" | Sort by start time, then sweep. |
| "sort by frequency", "group by then order" | Count with a map, then sort the groups. |
| Keys are small integers, or fixed-width numbers | Counting sort or radix sort. |
| "stable", "keep original order of equal items" | Merge sort, insertion sort, or JS built-in sort. |

## Core Concepts / Template

### Words you must know

- **Stable:** equal items keep their original relative order. This matters when you sort by one key and then another (sort by name, then by age: with a stable sort, people of the same age stay in name order).
- **In-place:** uses O(1) extra memory (not counting the recursion stack, which interviewers usually discuss separately).
- **Comparison sort:** decides order only by asking "is a before b?". Any comparison sort needs at least about n log n comparisons in the worst case. Counting, radix and bucket sort beat this because they do not compare; they use the shape of the keys.

### Comparison table

| Algorithm | Best | Average | Worst | Space | Stable | In-place | Note |
|---|---|---|---|---|---|---|---|
| Selection | O(n^2) | O(n^2) | O(n^2) | O(1) | No | Yes | Fewest swaps (at most n-1) |
| Bubble (with early exit) | O(n) | O(n^2) | O(n^2) | O(1) | Yes | Yes | Teaching only |
| Insertion | O(n) | O(n^2) | O(n^2) | O(1) | Yes | Yes | Great for tiny or nearly sorted input |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | No | Predictable, good for linked lists |
| Quick | O(n log n) | O(n log n) | O(n^2) | O(log n) stack | No | Yes | Fastest in practice with a random pivot |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | No | Yes | Worst case guaranteed, poor cache use |
| Counting | O(n + k) | O(n + k) | O(n + k) | O(n + k) | Yes (if built backwards) | No | k = value range |
| Radix (LSD) | O(d(n + b)) | O(d(n + b)) | O(d(n + b)) | O(n + b) | Yes | No | d = digits, b = base |
| Bucket | O(n + k) | O(n + k) | O(n^2) | O(n + k) | Yes (if buckets are) | No | Needs evenly spread input |
| Cyclic | O(n) | O(n) | O(n) | O(1) | n/a | Yes | Only for values in 1..n |

### When to use which

| Situation | Use |
|---|---|
| Normal interview problem, no special structure | Built-in sort with a comparator, O(n log n) |
| Need the k-th element only | Quickselect or a heap |
| Need stability and guaranteed O(n log n) | Merge sort |
| Memory is tight, worst case must be O(n log n) | Heap sort |
| Small n (under about 20), or nearly sorted | Insertion sort |
| Small integer keys | Counting sort |
| Many fixed-width integers or strings | Radix sort |
| Values are exactly 1..n permutation-like | Cyclic sort |
| Count pairs while sorting | Merge sort with a counter |

### The simple O(n^2) sorts

```js
const swap = (a, i, j) => { [a[i], a[j]] = [a[j], a[i]]; };

function selectionSort(a) {
  for (let i = 0; i < a.length - 1; i++) {
    let min = i;
    for (let j = i + 1; j < a.length; j++) if (a[j] < a[min]) min = j;
    if (min !== i) swap(a, i, min);
  }
  return a;
}

function bubbleSort(a) {
  for (let end = a.length - 1; end > 0; end--) {
    let swapped = false;
    for (let j = 0; j < end; j++) {
      if (a[j] > a[j + 1]) { swap(a, j, j + 1); swapped = true; }
    }
    if (!swapped) break; // already sorted: O(n) best case
  }
  return a;
}

function insertionSort(a) {
  for (let i = 1; i < a.length; i++) {
    const x = a[i];
    let j = i - 1;
    while (j >= 0 && a[j] > x) { a[j + 1] = a[j]; j--; } // strict > keeps it stable
    a[j + 1] = x;
  }
  return a;
}
```

Why each one works:

- **Selection:** the left part is always the smallest `i` items in final position. Each round finds the minimum of the rest. It swaps over long distances, so it is not stable ([2a, 2b, 1] becomes [1, 2b, 2a]).
- **Bubble:** big values "bubble" to the right end. After each pass the largest unsorted value is in place. The `swapped` flag gives O(n) on sorted input.
- **Insertion:** like sorting playing cards in your hand. Take the next card and slide it left past bigger cards. Fast when the input is nearly sorted, which is why real libraries use it for small chunks.

### Merge sort

Idea: split in half, sort each half, merge two sorted halves. The merge is the real work.

```js
function mergeSort(a) {
  if (a.length <= 1) return a;
  const mid = a.length >> 1;
  const left = mergeSort(a.slice(0, mid));
  const right = mergeSort(a.slice(mid));
  const out = [];
  let i = 0, j = 0;
  while (i < left.length && j < right.length) {
    out.push(left[i] <= right[j] ? left[i++] : right[j++]); // <= keeps it stable
  }
  while (i < left.length) out.push(left[i++]);
  while (j < right.length) out.push(right[j++]);
  return out;
}
```

Example: `[5,2,4,1]` splits to `[5,2]` and `[4,1]`, which sort to `[2,5]` and `[1,4]`, which merge to `[1,2,4,5]`.

Why O(n log n): there are log n levels of splitting, and each level does O(n) work in total merging. The `<=` in the merge takes the left item on ties, so equal items keep their order (stable). Space is O(n) for the temporary arrays.

### Quick sort (Lomuto, Hoare, random pivot)

Idea: pick a pivot, rearrange so smaller items are on its left and bigger on its right, then the pivot is already in its final place. Recurse on both sides.

**Lomuto partition** is simple. Keep a boundary `i`; everything before `i` is smaller than the pivot.

```js
function lomutoPartition(a, lo, hi) {
  const r = lo + Math.floor(Math.random() * (hi - lo + 1)); // random pivot
  swap(a, r, hi);
  const pivot = a[hi];
  let i = lo; // a[lo..i-1] are < pivot
  for (let j = lo; j < hi; j++) if (a[j] < pivot) swap(a, i++, j);
  swap(a, i, hi);
  return i;
}

function quickSortLomuto(a, lo = 0, hi = a.length - 1) {
  if (lo >= hi) return a;
  const p = lomutoPartition(a, lo, hi);
  quickSortLomuto(a, lo, p - 1);
  quickSortLomuto(a, p + 1, hi);
  return a;
}
```

Trace of partition on `[3, 7, 1, 5]`, pivot 5 (moved to the end first, so the array is `[3, 7, 1, 5]`):

```
j=0: 3 < 5 -> swap with a[i=0], i=1       [3,7,1,5]
j=1: 7 < 5 no                              [3,7,1,5]
j=2: 1 < 5 -> swap a[1],a[2], i=2         [3,1,7,5]
end: swap a[i=2] with pivot                [3,1,5,7]   pivot index 2
```

**Hoare partition** uses two pointers moving toward each other and does fewer swaps. It is trickier: the returned index `j` is a split point, not the pivot's final position, so you recurse on `[lo..j]` and `[j+1..hi]`.

```js
function hoareSort(a, lo = 0, hi = a.length - 1) {
  if (lo >= hi) return a;
  const pivot = a[lo + Math.floor(Math.random() * (hi - lo + 1))];
  let i = lo - 1, j = hi + 1;
  while (true) {
    do i++; while (a[i] < pivot);
    do j--; while (a[j] > pivot);
    if (i >= j) break;
    swap(a, i, j);
  }
  hoareSort(a, lo, j);     // note: j, not j - 1
  hoareSort(a, j + 1, hi);
  return a;
}
```

Why a random pivot: if you always pick the first or last item, a sorted array makes every split 1 vs n-1, and the cost becomes O(n^2). A random pivot makes that very unlikely, so the expected time is O(n log n). Both versions above degrade on many equal keys (Lomuto sends all equals to one side). A 3-way partition (less, equal, greater) fixes this; the Dutch flag in problem 3 is exactly that idea.

### Heap sort

Build a max-heap in the array itself, then repeatedly swap the max to the end and shrink the heap.

```js
function heapSort(a) {
  const n = a.length;
  const siftDown = (i, size) => {
    while (true) {
      let big = i;
      const l = 2 * i + 1, r = l + 1;
      if (l < size && a[l] > a[big]) big = l;
      if (r < size && a[r] > a[big]) big = r;
      if (big === i) return;
      swap(a, i, big);
      i = big;
    }
  };
  for (let i = (n >> 1) - 1; i >= 0; i--) siftDown(i, n); // build max-heap, O(n)
  for (let end = n - 1; end > 0; end--) { swap(a, 0, end); siftDown(0, end); }
  return a;
}
```

Building the heap bottom-up is O(n). Then n times `siftDown` costs O(log n) each. It is in-place and worst-case O(n log n), but not stable, and it jumps around memory so it is usually slower than quick sort.

### Counting, radix and bucket sort (no comparisons)

```js
function countingSort(a, max) { // non-negative integers, values <= max
  const count = new Array(max + 1).fill(0);
  for (const x of a) count[x]++;
  for (let v = 1; v <= max; v++) count[v] += count[v - 1]; // prefix: end position
  const out = new Array(a.length);
  for (let i = a.length - 1; i >= 0; i--) out[--count[a[i]]] = a[i]; // backwards = stable
  return out;
}

function radixSort(a) { // non-negative integers, LSD base 10
  if (a.length === 0) return a;
  const max = Math.max(...a);
  for (let exp = 1; Math.floor(max / exp) > 0; exp *= 10) {
    const count = new Array(10).fill(0);
    const digit = (x) => Math.floor(x / exp) % 10;
    for (const x of a) count[digit(x)]++;
    for (let d = 1; d < 10; d++) count[d] += count[d - 1];
    const out = new Array(a.length);
    for (let i = a.length - 1; i >= 0; i--) out[--count[digit(a[i])]] = a[i];
    a = out;
  }
  return a;
}

function bucketSort(a) { // numbers in [0, 1)
  const n = a.length;
  const buckets = Array.from({ length: n }, () => []);
  for (const x of a) buckets[Math.floor(x * n)].push(x);
  return buckets.flatMap((b) => insertionSort(b));
}
```

- **Counting:** count how many times each value appears, turn the counts into end positions with a prefix sum, then place items. Going backwards while placing is what makes it stable. This stability is the reason radix sort works.
- **Radix (LSD):** sort by the last digit, then the next digit, and so on, each pass using a stable counting sort. After the final digit, the whole array is sorted. Cost O(d(n + b)).
- **Bucket:** spread numbers into n buckets by value, sort each small bucket, join. Fast only if the data is spread evenly; if everything lands in one bucket, you get the cost of the inner sort (here O(n^2) with insertion sort).

### Cyclic sort

If the array holds the numbers 1..n, then value `v` belongs at index `v - 1`. Look at `a[i]`: if it is not in its home, swap it into its home. If it is home (or its home already holds the same value), move on. Each swap puts one value home forever, so there are at most n swaps: O(n) time, O(1) space.

```js
function cyclicSort(a) { // a is a permutation of 1..n
  let i = 0;
  while (i < a.length) {
    const j = a[i] - 1; // where a[i] belongs
    if (a[i] !== a[j]) swap(a, i, j);
    else i++;
  }
  return a;
}
```

Trace on `[3, 1, 2]`:

```
i=0: a[0]=3 belongs at index 2, swap -> [2,1,3]
i=0: a[0]=2 belongs at index 1, swap -> [1,2,3]
i=0: a[0]=1 is home -> i=1, then i=2, done
```

This is the engine behind "find the missing / duplicate number" problems below.

### JavaScript sort pitfalls

| Pitfall | Example | Fix |
|---|---|---|
| Default sort compares as **strings** | `[10, 9, 1].sort()` gives `[1, 10, 9]` | `nums.sort((a, b) => a - b)` |
| Negative numbers also break | `[-1, -2].sort()` stays `[-1, -2]` (the string "-1" < "-2") | Use the numeric comparator |
| `sort` mutates the array and returns the same array | `const b = a.sort(...)` changes `a` too | Copy first: `[...a].sort(...)` (or `a.toSorted(...)` in modern Node) |
| Comparator must return a number, not a boolean | `sort((a, b) => a > b)` is a bug (true/false become 1/0, never negative) | Return `a - b`, or -1 / 0 / 1 |
| `a - b` can overflow or give NaN | With `Infinity - Infinity`, or mixed `undefined` | Compare with `<` and `>` and return -1/0/1 |
| `undefined` goes to the end, whatever the comparator | `[undefined, 3, 1].sort()` gives `[1, 3, undefined]` | Remove or replace them first |

Good news: since ES2019, `Array.prototype.sort` is required to be **stable** (V8 uses TimSort, a merge/insertion hybrid). Cost is O(n log n). You can rely on that in Node.

## Problems (easy -> hard)

### 1. Merge Sorted Array — Easy
**Companies:** Amazon, Meta, Microsoft, Google
**Problem:** `nums1` has length `m + n`; its first `m` items are sorted and the last `n` are zeros (empty space). `nums2` has `n` sorted items. Merge `nums2` into `nums1` in place. Example: `[1,2,3,0,0,0]`, `[2,5,6]` -> `[1,2,2,3,5,6]`.
**Intuition:** If you fill from the front you overwrite values you still need. Fill from the back instead: the biggest remaining value goes to the last free slot.
**Approach 1 - Brute force:** copy `nums2` into the tail and sort all. Time O((m+n) log(m+n)), Space O(1) to O(n).
**Approach 2 - Better:** merge into a new array, then copy back. Time O(m+n), Space O(m+n).
**Approach 3 - Optimal:** three pointers `i` (end of nums1's real items), `j` (end of nums2), `k` (end of nums1). Take the larger of the two, write at `k`, move back.

Trace for `[1,2,3,_,_,_]` and `[2,5,6]`:

| step | compare | write at k | nums1 |
|---|---|---|---|
| 1 | 3 vs 6 | 6 at k=5 | [1,2,3,0,0,6] |
| 2 | 3 vs 5 | 5 at k=4 | [1,2,3,0,5,6] |
| 3 | 3 vs 2 | 3 at k=3 | [1,2,3,3,5,6] |
| 4 | 2 vs 2 | 2 (from nums2, tie) at k=2 | [1,2,2,3,5,6] |
| done | j < 0 | remaining nums1 items already in place | [1,2,2,3,5,6] |

```js
function merge(nums1, m, nums2, n) {
  let i = m - 1, j = n - 1, k = m + n - 1;
  while (j >= 0) nums1[k--] = i >= 0 && nums1[i] > nums2[j] ? nums1[i--] : nums2[j--];
}
```

**Complexity:** Time O(m + n), Space O(1).
**Watch out:** loop on `j >= 0`, not on `i`. When `nums2` is used up, the rest of `nums1` is already correct. If `m = 0`, only `nums2` items are copied.

### 2. Missing Number — Easy
**Companies:** Amazon, Microsoft, Apple, Meta
**Problem:** `nums` holds `n` distinct numbers from `0..n`. Return the one missing. Example: `[3,0,1]` -> `2`.
**Intuition:** There are `n + 1` possible values but `n` slots. Put each value `v` into slot `v`. The slot that ends up with the wrong value (or the value `n`, which has no slot) shows the missing number.
**Approach 1 - Brute force:** sort, then find the first index where `nums[i] !== i`. Time O(n log n), Space O(1).
**Approach 2 - Better:** sum formula `n(n+1)/2 - sum(nums)` (or XOR of indices and values). Time O(n), Space O(1). Very good, and honest to say it is the best in practice.
**Approach 3 - Optimal (cyclic sort):** swap each value to its own index, skipping `v = n`. Then the first index with `a[k] !== k` is the answer, else `n`.

Trace on `[3,0,1]` (n=3):

```
i=0: 3 is >= n, skip, i=1
i=1: 0 belongs at 0, swap -> [0,3,1]
i=1: 3 skip, i=2
i=2: 1 belongs at 1, swap -> [0,1,3]
i=2: 3 skip, i=3. Scan: a[0]=0, a[1]=1, a[2]=3 != 2 -> answer 2
```

```js
function missingNumber(nums) { // cyclic sort; works on a copy
  const a = [...nums], n = a.length;
  let i = 0;
  while (i < n) {
    const j = a[i];
    if (j < n && a[j] !== j) [a[i], a[j]] = [a[j], a[i]];
    else i++;
  }
  for (let k = 0; k < n; k++) if (a[k] !== k) return k;
  return n;
}
```

**Complexity:** Time O(n), Space O(n) because of the copy (O(1) if you may change the input).
**Watch out:** the guard `j < n` stops an out-of-range index. If nothing is wrong inside, the answer is `n`. If the interviewer says "do not modify the input", prefer the sum or XOR trick.

### 3. Sort Colors — Medium
**Companies:** Amazon, Microsoft, Meta, Apple, Google
**Problem:** Sort an array of 0s, 1s and 2s in place, in one pass, without using the library sort. Example: `[2,0,2,1,1,0]` -> `[0,0,1,1,2,2]`.
**Intuition:** Three zones: 0s on the left, 2s on the right, 1s in the middle. Keep three pointers and grow the zones as you scan (Dutch National Flag by Dijkstra).
**Approach 1 - Brute force:** `sort()`. Time O(n log n).
**Approach 2 - Better:** count 0s, 1s, 2s, then overwrite. Two passes, Time O(n), Space O(1).
**Approach 3 - Optimal:** one pass with `lo`, `mid`, `hi`. `a[0..lo-1]` are 0, `a[lo..mid-1]` are 1, `a[hi+1..]` are 2, and `a[mid..hi]` is unknown.

Trace on `[2,0,1]`:

```
lo=0 mid=0 hi=2  a[mid]=2 -> swap with hi: [1,0,2], hi=1   (do NOT move mid)
lo=0 mid=0 hi=1  a[mid]=1 -> mid=1
lo=0 mid=1 hi=1  a[mid]=0 -> swap with lo: [0,1,2], lo=1, mid=2
mid > hi, stop
```

```js
function sortColors(nums) {
  let lo = 0, mid = 0, hi = nums.length - 1;
  while (mid <= hi) {
    if (nums[mid] === 0) { [nums[lo], nums[mid]] = [nums[mid], nums[lo]]; lo++; mid++; }
    else if (nums[mid] === 2) { [nums[mid], nums[hi]] = [nums[hi], nums[mid]]; hi--; }
    else mid++;
  }
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** after swapping with `hi`, do not advance `mid`; the swapped-in value is not checked yet. After swapping with `lo`, you may advance `mid` because what came from `lo` is always a 1.

### 4. Sort Characters By Frequency — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Sort a string so the most frequent characters come first. Example: `"tree"` -> `"eetr"` (or `"eert"`).
**Intuition:** Count first, then order the groups by count, and write each group out as a block.
**Approach 1 - Brute force:** for each character, count it by scanning, then sort. Time O(n^2).
**Approach 2 - Better:** map of counts, sort the entries by count descending, build the string. Time O(n + u log u) where u = number of distinct characters.
**Approach 3 - Optimal:** bucket sort by frequency. Make `n + 1` buckets where bucket `c` holds characters that appear `c` times, then read from the highest bucket to the lowest. Time O(n). In an interview, Approach 2 is normally accepted; mention bucket sort as the upgrade.

```js
function frequencySort(s) {
  const freq = new Map();
  for (const ch of s) freq.set(ch, (freq.get(ch) ?? 0) + 1);
  return [...freq].sort((a, b) => b[1] - a[1]).map(([ch, c]) => ch.repeat(c)).join('');
}
```

Dry run on `"tree"`: counts `t:1, r:1, e:2` -> sorted entries `e:2, t:1, r:1` -> `"ee" + "t" + "r"` = `"eetr"`.

**Complexity:** Time O(n + u log u), Space O(n).
**Watch out:** characters with the same count may come in any order unless the problem says otherwise. Uppercase and lowercase are different characters. 

### 5. Merge Intervals — Medium
**Companies:** Amazon, Google, Meta, Microsoft, Bloomberg
**Problem:** Merge all overlapping intervals. Example: `[[1,3],[2,6],[8,10],[15,18]]` -> `[[1,6],[8,10],[15,18]]`. This is the base of every "meeting" question (can one person attend all meetings? how many rooms?).
**Intuition:** If intervals are sorted by start, an interval can only overlap the one just before it (the merged result's last interval). So one pass is enough.
**Approach 1 - Brute force:** repeatedly look for any overlapping pair and merge until none. Time O(n^2) or worse.
**Approach 3 - Optimal:** sort by start, then keep a result list. If the next start is <= the last end, extend the last end with `max`; else push a new interval.

Trace on `[[1,3],[2,6],[8,10]]`:

```
out=[[1,3]]
[2,6]: 2 <= 3 -> end = max(3,6)=6   out=[[1,6]]
[8,10]: 8 > 6 -> push               out=[[1,6],[8,10]]
```

```js
function mergeIntervals(intervals) {
  const sorted = [...intervals].sort((a, b) => a[0] - b[0]);
  const out = [];
  for (const [s, e] of sorted) {
    const last = out[out.length - 1];
    if (last && s <= last[1]) last[1] = Math.max(last[1], e);
    else out.push([s, e]);
  }
  return out;
}
```

**Complexity:** Time O(n log n), Space O(n) for the output (and the sorted copy).
**Watch out:** use `max` for the end (`[1,4]` and `[2,3]` must give `[1,4]`). Touching intervals like `[1,4]` and `[4,5]` merge here (`<=`); check with the interviewer whether touching counts. Sorting the input mutates it, so copy if needed.

Meeting-sort basics that follow from the same idea: **Meeting Rooms** (can attend all) is "sort by start, return false if any `start < previous end`". **Meeting Rooms II** (min rooms) is "sort starts and ends separately and sweep" or "sort by start with a min-heap of end times". See the intervals chapter for the full treatment.

### 6. Largest Number — Medium
**Companies:** Amazon, Google, Microsoft, Apple
**Problem:** Arrange non-negative integers to form the largest possible number, returned as a string. Example: `[3,30,34,5,9]` -> `"9534330"`.
**Intuition:** Plain numeric or alphabetical order fails (`3` vs `30`). Instead decide order by asking: "which glued result is bigger, `a+b` or `b+a`?" That is a valid total order, so sorting with it works.
**Approach 1 - Brute force:** try all permutations. Time O(n! * n).
**Approach 3 - Optimal:** convert to strings, sort with comparator on `a+b` vs `b+a`, join. If the first item is `"0"` the answer is `"0"`.

Dry run for `3` and `30`: `"330"` vs `"303"`. `"330"` is bigger, so `3` comes before `30`.

```js
function largestNumber(nums) {
  const strs = nums.map(String).sort((a, b) => (a + b > b + a ? -1 : a + b < b + a ? 1 : 0));
  return strs[0] === '0' ? '0' : strs.join('');
}
```

**Complexity:** Time O(n log n * L) where L is the digit length (string comparison), Space O(n * L).
**Watch out:** `[0, 0]` must give `"0"`, not `"00"`. Do not use `a - b` or the default sort here. The comparator must be consistent (transitive); gluing strings is.

### 7. Find the Duplicate Number — Medium
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** `nums` has `n + 1` integers, each in `1..n`, with exactly one value repeated (maybe many times). Return it. Example: `[1,3,4,2,2]` -> `2`.
**Intuition:** Value `v` belongs at index `v - 1`. While putting values home, the duplicate is the one whose home is already taken by an equal value.
**Approach 1 - Brute force:** compare every pair. Time O(n^2), Space O(1).
**Approach 2 - Better:** sort and look at neighbours (O(n log n)), or a `Set` (O(n) time, O(n) space).
**Approach 3 - Optimal:** cyclic sort, O(n) time and O(1) space, but it **changes the input**. If the problem forbids that, use Floyd's cycle detection (treat `i -> nums[i]` as a linked list) or binary search on the value range; those leave the array alone.

Trace on `[3,1,3,4,2]`:

```
i=0: a[0]=3 belongs at 2, a[2]=3 equals it -> duplicate is 3
```

```js
function findDuplicate(nums) { // cyclic sort, mutates nums
  let i = 0;
  while (i < nums.length) {
    const j = nums[i] - 1;
    if (nums[i] === i + 1) i++;
    else if (nums[j] === nums[i]) return nums[i];
    else [nums[i], nums[j]] = [nums[j], nums[i]];
  }
  return -1;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** the check for "home already holds the same value" must come before swapping, or you loop forever on duplicates. This needs values in `1..n`; with values `0..n` the home index changes.

### 8. Kth Largest Element in an Array — Medium
**Companies:** Amazon, Meta, Google, Microsoft, Apple
**Problem:** Return the k-th largest element (not the k-th distinct). Example: `[3,2,1,5,6,4]`, `k = 2` -> `5`.
**Intuition:** You do not need the whole array sorted, only the element that would sit at index `n - k`. Quick sort's partition tells you the final position of the pivot. Then keep only the side that contains the target (quickselect).
**Approach 1 - Brute force:** sort and index. Time O(n log n), Space O(1) to O(n).
**Approach 2 - Better:** min-heap of size k. Time O(n log k), Space O(k). Good when data streams in or k is small.
**Approach 3 - Optimal:** quickselect with a random pivot. Average O(n), worst O(n^2) (very unlikely with a random pivot). Space O(1) iterative.

Trace on `[3,2,1,5,6,4]`, k=2 -> target index 4. Say pivot 4: partition gives `[3,2,1,4,6,5]`, pivot at index 3. Target 4 > 3, so search `[6,5]` (indices 4..5). Pivot 5: partition gives `[5,6]`, pivot index 4 = target, answer `5`.

```js
function findKthLargest(nums, k) {
  const a = [...nums];
  const target = a.length - k; // index in sorted order
  let lo = 0, hi = a.length - 1;
  while (lo < hi) {
    const r = lo + Math.floor(Math.random() * (hi - lo + 1));
    [a[r], a[hi]] = [a[hi], a[r]];
    const pivot = a[hi];
    let i = lo;
    for (let j = lo; j < hi; j++) if (a[j] < pivot) { [a[i], a[j]] = [a[j], a[i]]; i++; }
    [a[i], a[hi]] = [a[hi], a[i]];
    if (i === target) return a[i];
    if (i < target) lo = i + 1; else hi = i - 1;
  }
  return a[lo];
}
```

**Complexity:** Time O(n) average, O(n^2) worst; Space O(n) because of the copy (O(1) if you may modify the input).
**Watch out:** target index is `n - k` for k-th largest (`k - 1` for k-th smallest). A fixed pivot gets O(n^2) on sorted input; that is why the pivot is random. Duplicates are fine because the partition uses a strict `<` and we only compare indices.

### 9. Count Inversions — Medium
**Companies:** Amazon, Microsoft, Google, Flipkart
**Problem:** An inversion is a pair `i < j` with `a[i] > a[j]`. Count them. Example: `[2,4,1,3,5]` -> `3` (the pairs `(2,1)`, `(4,1)`, `(4,3)`). It measures "how far from sorted" an array is.
**Intuition:** During merge sort's merge step, both halves are sorted. If the left item is bigger than the right item, then it and every remaining item in the left half are all bigger than that right item. One comparison counts many inversions at once.
**Approach 1 - Brute force:** check all pairs. Time O(n^2), Space O(1).
**Approach 2 - Better:** a Fenwick tree (BIT) over compressed values. Time O(n log n), Space O(n). See the segment tree and Fenwick chapter.
**Approach 3 - Optimal (simple to code):** merge sort and add `mid - i` whenever you take from the right half.

Trace on `[2,4,1,3,5]` (left `[2,4]`, right `[1,3,5]` after sorting each side):

```
take 1 from right (2 > 1): count += 2 (both 2 and 4 are > 1)  -> 2
take 2 from left
take 3 from right (4 > 3): count += 1 (only 4 left)           -> 3
take 4, then 5 -> done. Total 3
```

```js
function countInversions(nums) {
  const a = [...nums];
  const tmp = new Array(a.length);
  const sort = (lo, hi) => { // sorts a[lo..hi), returns inversions
    if (hi - lo < 2) return 0;
    const mid = (lo + hi) >> 1;
    let cnt = sort(lo, mid) + sort(mid, hi);
    let i = lo, j = mid, k = lo;
    while (i < mid && j < hi) {
      if (a[i] <= a[j]) tmp[k++] = a[i++];
      else { cnt += mid - i; tmp[k++] = a[j++]; } // a[i..mid) are all > a[j]
    }
    while (i < mid) tmp[k++] = a[i++];
    while (j < hi) tmp[k++] = a[j++];
    for (let x = lo; x < hi; x++) a[x] = tmp[x];
    return cnt;
  };
  return sort(0, a.length);
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** use `<=` when taking from the left so equal values are not counted (an inversion needs strictly greater). Counting must happen only in the branch where you take from the right half. With up to about 10^5 items the count can reach about 5 x 10^9, which a JS number holds exactly (below 2^53).

### 10. Reverse Pairs — Hard
**Companies:** Google, Amazon, Microsoft, Flipkart
**Problem:** Count pairs `i < j` with `nums[i] > 2 * nums[j]`. Example: `[1,3,2,3,1]` -> `2`.
**Intuition:** Same as count inversions, but the condition is not the same as the sort order, so you cannot count inside the merge. Instead, before merging, count with a second pointer over the two sorted halves, then merge normally.
**Approach 1 - Brute force:** all pairs. Time O(n^2).
**Approach 2 - Better:** Fenwick tree over compressed values. Time O(n log n), Space O(n).
**Approach 3 - Optimal:** merge sort with a counting pass. For each `i` in the left half, move `j` in the right half forward while `a[i] > 2 * a[j]`. Because the left half is sorted ascending, `j` never needs to go back, so the pass is O(n).

Trace on `[2,4,3,5,1]`. Top merge: left `[2,4]`, right sorted `[1,3,5]`.

```
i=2: 2 > 2*1? no                      -> 0
i=4: 4 > 2 yes (j=1), 4 > 6 no        -> +1   (pair 4,1)
right half [3,5,1]: its own merges find (5,1) and (3,1) -> +2
total 3
```

```js
function reversePairs(nums) {
  const a = [...nums];
  const tmp = new Array(a.length);
  const sort = (lo, hi) => {
    if (hi - lo < 2) return 0;
    const mid = (lo + hi) >> 1;
    let cnt = sort(lo, mid) + sort(mid, hi);
    let j = mid; // count BEFORE merging; both halves are sorted
    for (let i = lo; i < mid; i++) {
      while (j < hi && a[i] > 2 * a[j]) j++;
      cnt += j - mid;
    }
    let p = lo, q = mid, k = lo;
    while (p < mid && q < hi) tmp[k++] = a[p] <= a[q] ? a[p++] : a[q++];
    while (p < mid) tmp[k++] = a[p++];
    while (q < hi) tmp[k++] = a[q++];
    for (let x = lo; x < hi; x++) a[x] = tmp[x];
    return cnt;
  };
  return sort(0, a.length);
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** do the counting pass BEFORE the merge, while both halves are still sorted separately. Use `2 * a[j]`, not `a[j] << 1` (the shift turns the number into a 32-bit int and overflows for values near 2^31). The `j` pointer must not reset inside the `i` loop.

### 11. First Missing Positive — Hard
**Companies:** Amazon, Google, Microsoft, Meta, Apple
**Problem:** Find the smallest positive integer not in an unsorted array, in O(n) time and O(1) extra space. Example: `[3,4,-1,1]` -> `2`.
**Intuition:** The answer is always in `1..n+1`, since only `n` numbers exist. So ignore anything outside `1..n` and use the array itself as a "seen" table with cyclic sort: put `v` at index `v - 1`.
**Approach 1 - Brute force:** for `x = 1, 2, 3 ...` scan the array. Time O(n^2).
**Approach 2 - Better:** put everything in a Set, then count up from 1. Time O(n), Space O(n). Or sort then scan, O(n log n).
**Approach 3 - Optimal:** cyclic sort ignoring out-of-range values and duplicates, then the first index with `a[i] !== i + 1` gives `i + 1`; else `n + 1`.

Trace on `[3,4,-1,1]`:

```
i=0: 3 -> index 2, swap -> [-1,4,3,1]
i=0: -1 out of range, i=1
i=1: 4 -> index 3, swap -> [-1,1,3,4]
i=1: 1 -> index 0, swap -> [1,-1,3,4]
i=1: -1 skip, i=2: 3 home, i=3: 4 home
scan: a[1] = -1 != 2 -> answer 2
```

```js
function firstMissingPositive(nums) {
  const a = nums, n = a.length;
  for (let i = 0; i < n; ) {
    const v = a[i];
    if (v >= 1 && v <= n && a[v - 1] !== v) [a[i], a[v - 1]] = [a[v - 1], a[i]];
    else i++;
  }
  for (let i = 0; i < n; i++) if (a[i] !== i + 1) return i + 1;
  return n + 1;
}
```

**Complexity:** Time O(n), Space O(1) (changes the input).
**Watch out:** the condition `a[v - 1] !== v` is what handles duplicates; without it `[1,1]` loops forever. Zero and negatives are skipped. All of `1..n` present means the answer is `n + 1`.

## Common Mistakes

- **"`sort()` sorts numbers."** It sorts by string. Always pass `(a, b) => a - b` for numbers.
- **"The comparator can return true/false."** It must return a negative, zero or positive number.
- **"Quick sort is always O(n log n)."** Its worst case is O(n^2). A random pivot (or median of three) makes it unlikely; say that out loud.
- **"Merge sort is in-place."** Array merge sort needs O(n) extra space. Only linked-list merge sort avoids it.
- **"Sorting is free."** Adding a sort makes the whole solution at least O(n log n). If the problem wants O(n), look for counting, hashing or cyclic sort.
- **"Hoare returns the pivot position."** It returns a split point; recurse on `[lo..j]` and `[j+1..hi]`, never `j - 1` (that can loop forever).
- **"A custom comparator can be anything."** It must be consistent: if `a < b` and `b < c` then `a < c`. Otherwise results are undefined.

## Interview Tips

- Start with "I will sort first, which costs O(n log n)". Then ask whether that is acceptable or whether O(n) is required. This shows you know the cost.
- Ask: are there duplicates? Is the range small? Is the data nearly sorted? May I modify the input? The answers pick the algorithm.
- Be ready to write merge sort and quick sort fully from memory, and the merge step alone.
- Common follow-ups: "Is it stable?", "What is the worst case of quick sort and how do you avoid it?", "How would you sort a file too big for memory?" (external merge sort: sort chunks, merge them), "How do you sort a linked list?" (merge sort), "Why is the lower bound n log n?" (n! possible orders, each comparison halves at best, so about log2(n!) comparisons).
- Edge cases to say aloud: empty, one item, all equal, sorted, reverse sorted, negatives, duplicates.
- For kth element, offer three options (sort, heap, quickselect) and state the trade-offs.
- For "count pairs" problems, say "merge sort counts pairs while merging, because both halves are already sorted".

## Practice List

**Easy:** Sort an Array (own implementation), Squares of a Sorted Array, Valid Anagram, Meeting Rooms, Sort Array By Parity, Relative Sort Array, Find All Numbers Disappeared in an Array

**Medium:** Meeting Rooms II, Insert Interval, Top K Frequent Elements, K Closest Points to Origin, Sort List, Find All Duplicates in an Array, Set Mismatch, Maximum Gap, Custom Sort String, Minimum Number of Arrows to Burst Balloons, Count of Smaller Numbers After Self

**Hard:** Count of Range Sum, Median of Two Sorted Arrays, Max Chunks To Make Sorted II, Merge k Sorted Lists

## Key Takeaways

- **Default `sort()` is lexicographic**; always pass a numeric comparator and return a number.
- **Merge sort** is the safe choice: O(n log n) always, stable, O(n) space, and its merge step can count pairs.
- **Quick sort** is fastest in practice but worst case O(n^2); a **random pivot** makes that very unlikely, and **quickselect** finds the kth element in average O(n).
- **Stable and in-place are different questions**: insertion, merge, bubble are stable; selection, quick, heap are not.
- **Counting, radix and bucket sort** beat n log n by using the key's shape, but need small ranges or even spread.
- **Cyclic sort** solves "missing / duplicate in 1..n" in O(n) time and O(1) space by putting each value at its own index.
- **A custom comparator** turns many problems into "sort, then one pass" (largest number, intervals, relative order).

---

[← Previous: Arrays](../01-arrays/README.md) | [Next: Hashing →](../03-hashing/README.md) | [Back to Core Linear Index](../README.md)
