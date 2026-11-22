# Two Pointers

> **Estimated Time:** 4 hr | **Difficulty:** Beginner to Intermediate
> **Prerequisites:** [01 - Arrays](../01-arrays/README.md), [02 - Sorting](../02-sorting/README.md), [04 - Strings](../04-strings/README.md)

---

## What You'll Learn

- You can tell the three two-pointer shapes apart: opposite ends, read/write (same direction), and fast/slow.
- You can explain why moving one pointer is safe, and prove it in one or two sentences.
- You can turn O(n^2) pair and triple searches on sorted data into O(n) or O(n^2) with a pointer pair.
- You can do in-place array edits (remove, dedupe, partition) in one pass with O(1) extra space.
- You can handle duplicates in 3Sum and 4Sum without a Set.

## The Big Idea

Imagine two people searching a long hallway of numbered doors. One starts at the front, one at the back. Each time they meet by the middle they have checked every door, but nobody walked back and forth. Now imagine a teacher handing out papers: one finger points at the next paper to read, another finger points at the next free spot in the pile you keep. Two fingers, one walk.

Two pointers means: use two indexes instead of one, and move them by a rule so that every step throws away part of the search space for good. A brute force pair search checks all n*(n-1)/2 pairs. With a rule that says "this pointer can never be part of an answer again", each step removes at least one index, so you do at most n steps.

Interviewers love it because it tests whether you can see structure (sorted order, symmetry, a stable region behind a pointer) and justify a greedy move. The code is short, but the proof of "why is it safe to move" is the real question.

There are three shapes:

```
1. Opposite ends         [ L -> ..........  <- R ]     pairs, palindromes, containers
2. Read / write (same)   [ w  r ------------->    ]     filter, dedupe, partition in place
3. Fast / slow           [ s  f ------------->    ]     cycles, middle of a list (see Linked List chapter)
```

A fourth idea, the sliding window, is two same-direction pointers that mark a range. It gets its own chapter: [06 - Sliding Window](../06-sliding-window/README.md). Problem 17 here is a preview.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "sorted array", "find a pair with sum X" | opposite ends, move the side that fixes the sum |
| "palindrome", "reverse", "symmetric" | opposite ends, compare and move both |
| "in place", "O(1) extra space", "return the new length" | read/write pointers |
| "remove / move all X to the end" | read/write pointers (stable partition) |
| "triplet / quadruplet that sums to X" | sort, fix one or two, two pointers on the rest |
| "maximize area / minimize something between two ends" | opposite ends, move the weaker side |
| "merge two sorted arrays" | two readers, or two backward pointers if writing in place |
| "sort values from a tiny set (0/1/2)" | three pointers (Dutch flag) |
| "smallest range that contains ..." | window with two same-direction pointers |
| "cycle in a list", "middle of a list" | fast and slow |

Not a fit: unsorted data where you need pair sums and may not sort (indexes matter). Use a hash map ([03 - Hashing](../03-hashing/README.md)) there.

## Core Concepts / Template

### Shape 1: opposite ends

Works when moving a pointer gives you a clear signal. Sorted sum: if the sum is too small, the left value is too small to pair with anything remaining, so drop it.

```js
function oppositeEnds(a, target) {
  let l = 0, r = a.length - 1;
  while (l < r) {                     // l < r: do not pair an element with itself
    const s = a[l] + a[r];
    if (s === target) return [l, r];
    if (s < target) l++;              // need bigger: only moving l up can help
    else r--;                         // need smaller: only moving r down can help
  }
  return null;
}
```

Why it is safe (the proof to say out loud): suppose `a[l] + a[r] < target`. For this `l`, the largest partner left is `a[r]`. Even that is too small, so `l` can pair with nothing in `(l, r]`. Drop `l`. The symmetric argument drops `r` when the sum is too big.

Trace for `[1,3,4,6,9]`, target 10:

| l | r | a[l] + a[r] | action |
|---|---|---|---|
| 0 | 4 | 1 + 9 = 10 | found |

Trace for target 7: `1+9=10` too big, r-- ; `1+6=7` found.

### Shape 2: read / write pointers

`r` scans every element. `w` marks where the next kept element goes. Everything before `w` is the finished answer.

```js
function keepIf(a, keep) {
  let w = 0;
  for (let r = 0; r < a.length; r++) {
    if (keep(a[r], r)) a[w++] = a[r];   // copy a keeper to the write spot
  }
  return w;                             // new length; a[0..w) is the result
}
```

Invariant to state: "`a[0..w)` holds the answer for everything read so far". Trace, remove 3 from `[3,2,2,3]`:

| r | a[r] | keep? | w after | array prefix |
|---|---|---|---|---|
| 0 | 3 | no | 0 | [] |
| 1 | 2 | yes | 1 | [2] |
| 2 | 2 | yes | 2 | [2,2] |
| 3 | 3 | no | 2 | [2,2] |

### Shape 3: fast / slow

Both start at the head but fast moves two steps and slow moves one. If they meet there is a cycle; when fast hits the end, slow is at the middle. It is mostly used on linked lists, so the full treatment is in [09 - Linked List](../09-linked-list/README.md). On arrays it appears in "Find the Duplicate Number" (treat `i -> nums[i]` as a linked list).

### Shape 4 (preview): the window

Both pointers move right. `r` grows the window, `l` shrinks it when a rule breaks. Problem 17 shows it.

### Sorting first

If order of the original array does not matter and you need pairs, sorting costs O(n log n) and buys you the opposite-ends argument. If the answer needs original indexes (Two Sum I), do not sort, or sort pairs of (value, index).

## Problems (easy -> hard)

### 1. Valid Palindrome — Easy
**Companies:** Meta, Microsoft, Amazon, Apple
**Problem:** After lowercasing and ignoring non-alphanumeric characters, is the string a palindrome? Example: `"A man, a plan, a canal: Panama"` -> `true`.
**Intuition:** A palindrome reads the same from both ends. Compare the outermost characters, then step inward.
**Approach 1 - Brute force:** Build a cleaned string, reverse it, compare. Time O(n), Space O(n).
**Approach 2 - Optimal:** Two pointers on the original string. Skip characters that are not letters or digits, compare the rest. Space O(1).

Trace on `"a,b a"` (cleaned `aba`):

| l | r | s[l] | s[r] | action |
|---|---|---|---|---|
| 0 | 4 | a | a | equal, move both |
| 1 | 3 | , | (skip) | l -> 2 |
| 2 | 3 | b | (space, skip) | r -> 2, compare b with b, equal, done |

```js
function isPalindrome(s) {
  let l = 0, r = s.length - 1;
  const ok = (c) => /[a-z0-9]/i.test(c);
  while (l < r) {
    while (l < r && !ok(s[l])) l++;
    while (l < r && !ok(s[r])) r--;
    if (s[l].toLowerCase() !== s[r].toLowerCase()) return false;
    l++; r--;
  }
  return true;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** The inner skips need `l < r` too, or an all-punctuation string like `" "` runs past the ends. Digits count as characters: `"0P"` is false.

### 2. Reverse String — Easy
**Companies:** Microsoft, Amazon, Apple
**Problem:** Reverse an array of characters in place. Example: `["h","e","l","l","o"]` -> `["o","l","l","e","h"]`.
**Intuition:** Swap the two ends, move inward. The middle needs no swap.
**Approach 1 - Brute force:** Copy into a new reversed array. Space O(n).
**Approach 2 - Optimal:** Swap `s[l]` and `s[r]` while `l < r`. Space O(1).

```js
function reverseString(s) {
  let l = 0, r = s.length - 1;
  while (l < r) {
    [s[l], s[r]] = [s[r], s[l]];
    l++; r--;
  }
  return s;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Use `l < r`, not `l <= r` (harmless here, but sloppy). Strings in JavaScript are immutable, so you can only do this on an array: `s.split("")`.

### 3. Remove Element — Easy
**Companies:** Google, Amazon, Microsoft
**Problem:** Remove every occurrence of `val` in place and return the new length. The order of the first `k` elements may be anything, but the usual solution keeps it. Example: `[3,2,2,3]`, `val=3` -> `2` (prefix `[2,2]`).
**Intuition:** Walk through the array. Copy each element that is not `val` to the next free spot.
**Approach 1 - Brute force:** Shift the whole tail left on every match. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** Read/write pointers (the template above).

```js
function removeElement(nums, val) {
  let w = 0;
  for (let r = 0; r < nums.length; r++) {
    if (nums[r] !== val) nums[w++] = nums[r];
  }
  return w;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Return the length, not the array. If few elements are removed and the order is free, you can also swap the last element into the hole and shrink `n`: fewer writes.

### 4. Move Zeroes — Easy
**Companies:** Meta, Amazon, Google, Microsoft, Apple
**Problem:** Move all `0`s to the end while keeping the order of non-zero elements. In place. Example: `[0,1,0,3,12]` -> `[1,3,12,0,0]`.
**Intuition:** It is Remove Element plus "fill the rest with zeros". Swapping does both at once.
**Approach 1 - Brute force:** Copy non-zeros to a new array, pad zeros. Space O(n).
**Approach 2 - Optimal:** When `nums[r]` is non-zero, swap it with `nums[w]` and advance `w`. Everything before `w` is non-zero, everything between `w` and `r` is zero.

| r | nums after step | w |
|---|---|---|
| 0 (0) | [0,1,0,3,12] | 0 |
| 1 (1) | [1,0,0,3,12] | 1 |
| 2 (0) | [1,0,0,3,12] | 1 |
| 3 (3) | [1,3,0,0,12] | 2 |
| 4 (12) | [1,3,12,0,0] | 3 |

```js
function moveZeroes(nums) {
  let w = 0;
  for (let r = 0; r < nums.length; r++) {
    if (nums[r] !== 0) {
      [nums[w], nums[r]] = [nums[r], nums[w]];
      w++;
    }
  }
  return nums;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** When `w === r` the swap is a no-op, which is fine. Do not use `sort`: it is slower and may break the order.

### 5. Merge Sorted Array — Easy
**Companies:** Meta, Microsoft, Amazon, Google
**Problem:** `nums1` has `m` real values followed by `n` empty slots. `nums2` has `n` values. Both are sorted. Merge `nums2` into `nums1` in place. Example: `[1,2,3,0,0,0]`, `[2,5,6]` -> `[1,2,2,3,5,6]`.
**Intuition:** If you merge from the front you overwrite values you still need. The back of `nums1` is empty, so fill it from the back using the largest values first.
**Approach 1 - Brute force:** Append `nums2` and sort. Time O((m+n) log(m+n)).
**Approach 2 - Extra copy:** Copy `nums1`'s real part, merge forward. Space O(m).
**Approach 3 - Optimal:** Three pointers: `i` at the last real element of `nums1`, `j` at the last of `nums2`, `k` at the last slot. Place the larger of the two at `k`.

Trace: `i=2 (3)`, `j=2 (6)`, `k=5`.

| k | compare | placed | result |
|---|---|---|---|
| 5 | 3 vs 6 | 6 | [1,2,3,0,0,6] |
| 4 | 3 vs 5 | 5 | [1,2,3,0,5,6] |
| 3 | 3 vs 2 | 3 | [1,2,3,3,5,6] |
| 2 | 2 vs 2 | 2 (from nums2) | [1,2,2,3,5,6] |
| rest | j < 0 | stop | done |

```js
function merge(a, m, b, n) {
  let i = m - 1, j = n - 1, k = m + n - 1;
  while (j >= 0) {                       // leftover a[0..i] is already in place
    a[k--] = i >= 0 && a[i] > b[j] ? a[i--] : b[j--];
  }
  return a;
}
```

**Complexity:** Time O(m + n), Space O(1).
**Watch out:** Loop on `j >= 0`, not on `i`. If `nums2` runs out first, the rest of `nums1` is already correct. If `m = 0`, `i = -1` and the guard `i >= 0` saves you.

### 6. Two Sum II (Sorted Input) — Medium
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** Sorted array, find two numbers that add up to `target`. Return their 1-based indexes. Example: `[2,7,11,15]`, `9` -> `[1,2]`. Exactly one answer exists. Use O(1) extra space.
**Intuition:** Sorted order tells you which way to move. Too small: need a bigger left value. Too big: need a smaller right value.
**Approach 1 - Brute force:** All pairs. Time O(n^2).
**Approach 2 - Better:** Hash map of seen values. Time O(n), Space O(n) (breaks the space rule).
**Approach 3 - Optimal:** Opposite ends (the template).

Trace for `[2,7,11,15]`, 9: `2+15=17` too big, r--; `2+11=13` too big, r--; `2+7=9` found.

```js
function twoSumII(nums, target) {
  let l = 0, r = nums.length - 1;
  while (l < r) {
    const s = nums[l] + nums[r];
    if (s === target) return [l + 1, r + 1];
    if (s < target) l++; else r--;
  }
  return [];
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Return 1-based indexes. Negative numbers are fine, the argument only needs sorted order.

### 7. Squares of a Sorted Array — Easy
**Companies:** Meta, Amazon, Google
**Problem:** Given a sorted array (may contain negatives), return the squares, sorted. Example: `[-4,-1,0,3,10]` -> `[0,1,9,16,100]`.
**Intuition:** The biggest square is at one of the two ends (a big negative or a big positive). Pick the larger end, put it at the back of the answer, move that pointer in.
**Approach 1 - Brute force:** Square everything, sort. Time O(n log n).
**Approach 2 - Optimal:** Opposite ends, fill the result from the back.

Trace for `[-4,-1,0,3,10]`:

| k | compare abs | placed | result (back to front) |
|---|---|---|---|
| 4 | 4 vs 10 | 100 | [_,_,_,_,100] |
| 3 | 4 vs 3 | 16 | [_,_,_,16,100] |
| 2 | 1 vs 3 | 9 | [_,_,9,16,100] |
| 1 | 1 vs 0 | 1 | [_,1,9,16,100] |
| 0 | 0 vs 0 | 0 | [0,1,9,16,100] |

```js
function sortedSquares(nums) {
  const n = nums.length, res = new Array(n);
  let l = 0, r = n - 1;
  for (let k = n - 1; k >= 0; k--) {
    if (Math.abs(nums[l]) > Math.abs(nums[r])) { res[k] = nums[l] ** 2; l++; }
    else { res[k] = nums[r] ** 2; r--; }
  }
  return res;
}
```

**Complexity:** Time O(n), Space O(n) for the output (O(1) extra).
**Watch out:** When `l === r` both branches work. Fill from the back, not the front, or the order is wrong.

### 8. Remove Duplicates from Sorted Array II — Medium
**Companies:** Meta, Microsoft, Amazon, Google
**Problem:** Sorted array. Keep at most two copies of each value, in place, return the new length. Example: `[1,1,1,2,2,3]` -> `5` (prefix `[1,1,2,2,3]`).
**Intuition:** A value is allowed if it differs from the element two slots behind the write position. That element is the second-last kept one; if they are equal, we already have two copies.
**Approach 1 - Brute force:** Count and splice. O(n^2).
**Approach 2 - Optimal:** Read/write. Always keep the first two; after that keep `x` only if `x !== nums[w-2]`.

| x | w | nums[w-2] | keep? | prefix |
|---|---|---|---|---|
| 1 | 0 | - | yes (w<2) | [1] |
| 1 | 1 | - | yes | [1,1] |
| 1 | 2 | 1 | no | [1,1] |
| 2 | 2 | 1 | yes | [1,1,2] |
| 2 | 3 | 1 | yes | [1,1,2,2] |
| 3 | 4 | 2 | yes | [1,1,2,2,3] |

```js
function removeDuplicatesII(nums) {
  let w = 0;
  for (const x of nums) {
    if (w < 2 || x !== nums[w - 2]) nums[w++] = x;
  }
  return w;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Compare with `nums[w-2]` (the written array), not `nums[r-2]` (the original), though for sorted input both usually agree. Change `2` to `k` for "at most k copies". With `k = 1` this is Remove Duplicates I.

### 9. Sort Colors (Dutch National Flag) — Medium
**Companies:** Meta, Amazon, Microsoft, Google, Apple
**Problem:** Array of 0s, 1s and 2s. Sort in place, one pass, no library sort. Example: `[2,0,2,1,1,0]` -> `[0,0,1,1,2,2]`.
**Intuition:** Keep three zones: 0s on the left (before `lo`), 1s in the middle (before `mid`), 2s on the right (after `hi`). `mid` scans the unknown zone.
**Approach 1 - Brute force:** Count each value, rewrite. Two passes, O(n), O(1).
**Approach 2 - Optimal:** One pass, three pointers.

```
[ 0 0 0 | 1 1 1 | ? ? ? ? | 2 2 ]
          lo     mid      hi
```

- `nums[mid] === 0`: swap with `lo`, both `lo++` and `mid++` (what we got from `lo` is a known 1).
- `nums[mid] === 1`: `mid++`.
- `nums[mid] === 2`: swap with `hi`, `hi--` only (the swapped-in value is unknown).

Trace for `[2,0,1]`: mid sees 2, swap with hi -> `[1,0,2]`, hi=1. mid sees 1 -> mid=1. mid sees 0, swap with lo -> `[0,1,2]`.

```js
function sortColors(nums) {
  let lo = 0, mid = 0, hi = nums.length - 1;
  while (mid <= hi) {
    if (nums[mid] === 0) { [nums[lo], nums[mid]] = [nums[mid], nums[lo]]; lo++; mid++; }
    else if (nums[mid] === 1) mid++;
    else { [nums[mid], nums[hi]] = [nums[hi], nums[mid]]; hi--; }
  }
  return nums;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Loop while `mid <= hi`, not `<`. After swapping with `hi`, do not advance `mid`.

### 10. Boats to Save People — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Each boat carries at most 2 people and a weight `limit`. Find the minimum number of boats. Example: `[3,2,2,1]`, `limit=3` -> `3`.
**Intuition:** The heaviest person needs a boat anyway. Try to pair them with the lightest. If even the lightest cannot fit, the heaviest goes alone. Sorting plus greedy plus opposite ends.
**Approach 1 - Brute force:** Try all pairings. Exponential.
**Approach 2 - Optimal:** Sort. `l` at lightest, `r` at heaviest. Always `r--` (heaviest boards). If `p[l] + p[r] <= limit`, the lightest shares, so `l++`. Count one boat each step.

Trace for sorted `[1,2,2,3]`, limit 3:

| l | r | sum | boat takes | boats |
|---|---|---|---|---|
| 0 (1) | 3 (3) | 4 > 3 | 3 alone | 1 |
| 0 (1) | 2 (2) | 3 ok | 2 and 1 | 2 |
| 1 (2) | 1 (2) | 4 / single | 2 alone | 3 |

```js
function numRescueBoats(people, limit) {
  const p = [...people].sort((a, b) => a - b);   // numeric sort, not default
  let l = 0, r = p.length - 1, boats = 0;
  while (l <= r) {
    if (p[l] + p[r] <= limit) l++;
    r--; boats++;
  }
  return boats;
}
```

**Complexity:** Time O(n log n), Space O(n) for the copy (O(1) if you may sort in place; O(n) counting sort is possible when weights are bounded).
**Watch out:** `l <= r` so the last single person counts. When `l === r`, the sum is `2*p[l]`; if it is too heavy they ride alone, if it fits, `l++` and `r--` still counts exactly one boat, which is correct.

### 11. Partition Labels — Medium
**Companies:** Amazon, Meta, Microsoft, Google
**Problem:** Split a string into as many parts as possible so each letter appears in at most one part. Return the part sizes. Example: `"ababcbacadefegdehijhklij"` -> `[9,7,8]`.
**Intuition:** If a part contains letter `c`, it must reach `c`'s last position. So extend the part end to the farthest last-occurrence of everything seen. When the scan index reaches that end, cut.
**Approach 1 - Brute force:** For each start, grow the end and re-check all letters. O(n^2).
**Approach 2 - Optimal:** Precompute `last[c]`. One scan with `start` and `end` (a read pointer and a moving boundary).

Trace on `"ababcbaca|defegde|hijhklij"`: starting `end = last[a] = 8`. Scanning to index 8 pushes `end` to at most 8 (b's last is 5, c's last is 7). At `i = 8`, cut size 9. Repeat.

```js
function partitionLabels(s) {
  const last = {};
  for (let i = 0; i < s.length; i++) last[s[i]] = i;
  const res = [];
  let start = 0, end = 0;
  for (let i = 0; i < s.length; i++) {
    end = Math.max(end, last[s[i]]);
    if (i === end) { res.push(end - start + 1); start = i + 1; }
  }
  return res;
}
```

**Complexity:** Time O(n), Space O(1) (at most 26 letters).
**Watch out:** Use `Math.max`; a later letter can have an earlier last occurrence. Size is `end - start + 1`.

### 12. Container With Most Water — Medium
**Companies:** Amazon, Google, Meta, Microsoft, Apple
**Problem:** `height[i]` is a vertical line at `x = i`. Pick two lines to hold the most water. Area = `min(h[l], h[r]) * (r - l)`. Example: `[1,8,6,2,5,4,8,3,7]` -> `49`.
**Intuition:** Start with the widest container. Width can only shrink from here, so the only way to do better is a taller short side. The short side limits the height, so move the short side. Moving the tall side can never help: width drops and the height is still capped by the same short side.
**Approach 1 - Brute force:** All pairs. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** Opposite ends. Update best, move the shorter line inward.

Trace (first steps): `l=0 (1), r=8 (7)`: area `1*8=8`, move `l`. `l=1 (8), r=8 (7)`: area `7*7=49`, move `r` (7 < 8). Later pairs give smaller areas, best stays 49.

```js
function maxArea(h) {
  let l = 0, r = h.length - 1, best = 0;
  while (l < r) {
    best = Math.max(best, Math.min(h[l], h[r]) * (r - l));
    if (h[l] < h[r]) l++; else r--;
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Water level is the shorter line, not the average. Do not confuse with Trapping Rain Water (problem 16): that one has walls in between that matter.

### 13. 3Sum — Medium
**Companies:** Amazon, Meta, Google, Microsoft, Apple
**Problem:** Find all unique triplets `[a,b,c]` with `a + b + c = 0`. Example: `[-1,0,1,2,-1,-4]` -> `[[-1,-1,2],[-1,0,1]]`.
**Intuition:** Fix one number `a`. Then you need two numbers summing to `-a`, which is Two Sum II on the rest. Sorting lets you skip duplicates by comparing neighbors.
**Approach 1 - Brute force:** Three loops plus a Set of sorted triples. Time O(n^3).
**Approach 2 - Better:** Fix `i`, hash set for the pair. Time O(n^2), Space O(n), duplicates are fiddly.
**Approach 3 - Optimal:** Sort. For each `i` (skip equal to previous), run opposite-end search on `(i, n)`. After a hit, move `l` past equal values.

Sorted: `[-4,-1,-1,0,1,2]`.

| i | l | r | sum | action |
|---|---|---|---|---|
| 0 (-4) | 1 | 5 | -4-1+2=-3 | l++ ... never reaches 0, none |
| 1 (-1) | 2 | 5 | -1-1+2=0 | record [-1,-1,2], l=3, r=4 |
| 1 (-1) | 3 | 4 | -1+0+1=0 | record [-1,0,1] |
| 2 (-1) | - | - | same as previous i | skip |
| 3 (0) | 4 | 5 | 0+1+2 = 3 > 0 | no triplet |

```js
function threeSum(nums) {
  const a = [...nums].sort((x, y) => x - y), res = [];
  for (let i = 0; i < a.length - 2; i++) {
    if (a[i] > 0) break;                     // all later values are positive
    if (i > 0 && a[i] === a[i - 1]) continue; // skip duplicate first value
    let l = i + 1, r = a.length - 1;
    while (l < r) {
      const s = a[i] + a[l] + a[r];
      if (s < 0) l++;
      else if (s > 0) r--;
      else {
        res.push([a[i], a[l], a[r]]);
        l++; r--;
        while (l < r && a[l] === a[l - 1]) l++; // skip duplicate second value
      }
    }
  }
  return res;
}
```

**Complexity:** Time O(n^2) (sort O(n log n) is dominated), Space O(1) beyond the output and the sorted copy.
**Watch out:** Compare with the previous element (`a[i-1]`), not the next, or you skip `[-1,-1,2]`. Default `.sort()` in JS sorts as strings: always pass a numeric comparator. Duplicate skipping on `r` is not required once `l` is skipped, because the sum will then change and `r` moves anyway.

### 14. 3Sum Closest — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Find three numbers whose sum is closest to `target`; return that sum. Example: `[-1,2,1,-4]`, `1` -> `2`.
**Intuition:** Same skeleton as 3Sum. Instead of an exact hit, track the sum with the smallest distance to the target and still move pointers by "too small / too big".
**Approach 1 - Brute force:** Three loops. O(n^3).
**Approach 2 - Optimal:** Sort, fix `i`, opposite ends on the rest. Update `best` whenever `|s - target|` improves. Exit early on an exact match.

Sorted `[-4,-1,1,2]`, target 1: `i=0`: `-4-1+2=-3`, then `-4+1+2=-1`. `i=1`: `-1+1+2=2` distance 1, best = 2.

```js
function threeSumClosest(nums, target) {
  const a = [...nums].sort((x, y) => x - y);
  let best = a[0] + a[1] + a[2];
  for (let i = 0; i < a.length - 2; i++) {
    let l = i + 1, r = a.length - 1;
    while (l < r) {
      const s = a[i] + a[l] + a[r];
      if (Math.abs(s - target) < Math.abs(best - target)) best = s;
      if (s === target) return s;
      if (s < target) l++; else r--;
    }
  }
  return best;
}
```

**Complexity:** Time O(n^2), Space O(1) beyond the sorted copy.
**Watch out:** Initialize `best` with a real triple sum, not `Infinity` or `0` (0 may be closer than any real sum). No duplicate skipping is needed for correctness, only for speed. Requires `n >= 3`.

### 15. 4Sum — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Find all unique quadruplets that add up to `target`. Example: `[1,0,-1,0,-2,2]`, `0` -> `[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]`.
**Intuition:** One more loop around 3Sum. The general rule: k-Sum = (k-2) nested loops + two pointers, O(n^(k-1)).
**Approach 1 - Brute force:** Four loops. O(n^4).
**Approach 2 - Better:** Pair-sum hash map. O(n^2) average but unique handling is painful.
**Approach 3 - Optimal:** Sort, two fixed indexes `i`, `j`, then opposite ends. Skip duplicates at all three levels.

```js
function fourSum(nums, target) {
  const a = [...nums].sort((x, y) => x - y), res = [], n = a.length;
  for (let i = 0; i < n - 3; i++) {
    if (i > 0 && a[i] === a[i - 1]) continue;
    for (let j = i + 1; j < n - 2; j++) {
      if (j > i + 1 && a[j] === a[j - 1]) continue;
      let l = j + 1, r = n - 1;
      while (l < r) {
        const s = a[i] + a[j] + a[l] + a[r];   // JS numbers: no overflow worry at LeetCode limits
        if (s < target) l++;
        else if (s > target) r--;
        else {
          res.push([a[i], a[j], a[l], a[r]]);
          l++; r--;
          while (l < r && a[l] === a[l - 1]) l++;
        }
      }
    }
  }
  return res;
}
```

**Complexity:** Time O(n^3), Space O(1) beyond the output.
**Watch out:** The inner skip is `j > i + 1`, not `j > 0`; otherwise you skip valid `j` right after `i`. Do not prune with `a[i] > target` when negatives exist. In Java/C++ the sum can overflow 32 bits; in JS it cannot at these limits.

### 16. Trapping Rain Water — Hard
**Companies:** Amazon, Google, Meta, Microsoft, Apple
**Problem:** `height[i]` are bar heights of width 1. How much water is held after rain? Example: `[0,1,0,2,1,0,1,3,2,1,2,1]` -> `6`.
**Intuition:** Water above bar `i` = `min(maxLeft(i), maxRight(i)) - height[i]` (never negative). Everything is about the tallest wall on each side. The three solutions are three ways to get those maxima.

**Approach 1 - Brute force:** For every bar scan left and right for the max. Time O(n^2), Space O(1).
**Approach 2 - Prefix/suffix arrays:** Precompute `L[i]` = max of `h[0..i]` and `R[i]` = max of `h[i..n-1]`. Sum `min(L[i], R[i]) - h[i]`. Time O(n), Space O(n).
**Approach 3 - Stack:** Keep indexes of decreasing heights. When a taller bar `i` arrives, the popped bar is a "valley bottom"; the new stack top is its left wall and `i` is its right wall. Fill that layer: width `i - left - 1`, height `min(h[left], h[i]) - h[bottom]`. Time O(n), Space O(n). Uses the idea from [11 - Monotonic Stack and Queue](../11-monotonic-stack-queue/README.md).
**Approach 4 - Two pointers (optimal):** Keep `leftMax` and `rightMax`. If `h[l] < h[r]`, the left side is the limiting one: `rightMax >= h[r] > h[l]`, so the water at `l` depends only on `leftMax`. Add `leftMax - h[l]` and move `l`. Else do the mirror on `r`. Space O(1).

Trace for `[4,2,0,3,2,5]` (answer 9):

| l | r | h[l] vs h[r] | side | leftMax / rightMax | water added | total |
|---|---|---|---|---|---|---|
| 0 | 5 | 4 < 5 | left | 4 / 0 | 0 | 0 |
| 1 | 5 | 2 < 5 | left | 4 / 0 | 2 | 2 |
| 2 | 5 | 0 < 5 | left | 4 / 0 | 4 | 6 |
| 3 | 5 | 3 < 5 | left | 4 / 0 | 1 | 7 |
| 4 | 5 | 2 < 5 | left | 4 / 0 | 2 | 9 |
| 5 | 5 | loop ends | | | | 9 |

```js
// Approach 2: prefix / suffix maxima
function trapPrefix(h) {
  const n = h.length;
  if (!n) return 0;
  const L = new Array(n), R = new Array(n);
  L[0] = h[0];
  for (let i = 1; i < n; i++) L[i] = Math.max(L[i - 1], h[i]);
  R[n - 1] = h[n - 1];
  for (let i = n - 2; i >= 0; i--) R[i] = Math.max(R[i + 1], h[i]);
  let sum = 0;
  for (let i = 0; i < n; i++) sum += Math.min(L[i], R[i]) - h[i];
  return sum;
}

// Approach 3: monotonic stack, fills water layer by layer
function trapStack(h) {
  const st = []; let sum = 0;
  for (let i = 0; i < h.length; i++) {
    while (st.length && h[i] > h[st[st.length - 1]]) {
      const bottom = st.pop();
      if (!st.length) break;                  // no left wall
      const left = st[st.length - 1];
      sum += (Math.min(h[left], h[i]) - h[bottom]) * (i - left - 1);
    }
    st.push(i);
  }
  return sum;
}

// Approach 4: two pointers (optimal)
function trap(h) {
  let l = 0, r = h.length - 1, lm = 0, rm = 0, sum = 0;
  while (l < r) {
    if (h[l] < h[r]) { lm = Math.max(lm, h[l]); sum += lm - h[l]; l++; }
    else { rm = Math.max(rm, h[r]); sum += rm - h[r]; r--; }
  }
  return sum;
}
```

**Complexity:** Two pointers: Time O(n), Space O(1). Prefix and stack: Time O(n), Space O(n).
**Watch out:** Update the max before adding, so the term is never negative. In the stack version, pop first, then check that a left wall exists. Empty or length-1 inputs give 0.

### 17. Minimum Window Substring — Hard
**Companies:** Meta, Amazon, Google, Microsoft, Uber
**Problem:** Smallest substring of `s` that contains every character of `t` (with multiplicity). Return `""` if none. Example: `"ADOBECODEBANC"`, `"ABC"` -> `"BANC"`.
**Intuition:** This is a window made of two same-direction pointers. Grow `r` until the window is valid. Then shrink `l` as far as it stays valid, recording the best. Each index enters and leaves at most once. The full sliding-window theory is in [06 - Sliding Window](../06-sliding-window/README.md); here it shows that "two pointers" does not always mean "opposite ends".
**Approach 1 - Brute force:** Check every substring. Time O(n^2 * m) or worse.
**Approach 2 - Optimal:** `need[c]` = how many of `c` are still wanted (can go negative when the window has extra). `missing` = count of required characters not yet covered. When `missing === 0` the window is valid.

Trace on `"ADOBECODEBANC"`, `t = "ABC"` (key moments):

| step | window | state | action |
|---|---|---|---|
| r reaches `C` (idx 5) | `ADOBEC` | missing 0 | best = ADOBEC (6), shrink l |
| l moves past `A` | `DOBEC` | missing 1 | grow r |
| r reaches `A` (idx 10) | `DOBECODEBA` | missing 0 | shrink, window `CODEBA` (6) |
| r reaches `C` (idx 12), shrink | `BANC` | missing 0 | best = BANC (4) |

```js
function minWindow(s, t) {
  if (t.length > s.length) return '';
  const need = new Map();
  for (const c of t) need.set(c, (need.get(c) || 0) + 1);
  let missing = t.length, l = 0, bestL = 0, bestLen = Infinity;
  for (let r = 0; r < s.length; r++) {
    if ((need.get(s[r]) || 0) > 0) missing--;      // this char was still needed
    need.set(s[r], (need.get(s[r]) || 0) - 1);
    while (missing === 0) {                         // valid: try to shrink
      if (r - l + 1 < bestLen) { bestLen = r - l + 1; bestL = l; }
      need.set(s[l], need.get(s[l]) + 1);
      if (need.get(s[l]) > 0) missing++;            // we just lost a needed char
      l++;
    }
  }
  return bestLen === Infinity ? '' : s.slice(bestL, bestL + bestLen);
}
```

**Complexity:** Time O(n + m), Space O(k) for k distinct characters.
**Watch out:** `t` may have repeated letters (`"aa"`), so count, do not use a Set. Only decrement `missing` when the count was positive before; extra copies must not count. Record the best before moving `l`.

## Common Mistakes

- **"Two pointers works on any array."** It needs a reason the move is safe: sorted order, symmetry, or an invariant behind the pointer. If you cannot state the reason, you probably need a hash map or a window.
- **"Sorting is free."** It costs O(n log n) and destroys original indexes. Say it out loud, and use `(a, b) => a - b`: JavaScript's default `sort()` compares as strings, so `[10, 9, 1]` sorts as `[1, 10, 9]`.
- **"`l <= r` and `l < r` are interchangeable."** For pair problems, `l === r` would pair an element with itself. For Boats and Sort Colors you need `<=` because one element is left to handle. Decide per problem.
- **Skipping duplicates by looking ahead.** Compare with the previous value (`a[i] === a[i-1]`). Looking ahead drops valid answers like `[-1,-1,2]`.
- **Forgetting to advance both pointers after a hit** in 3Sum or 4Sum: infinite loop.
- **Writing forward when merging in place.** Merging from the front overwrites unread data. Write from the back.
- **Returning the array instead of the length** in the in-place removal problems.
- **Mixing up Container With Most Water and Trapping Rain Water.** Container ignores bars in between; Trapping Rain Water uses them.

## Interview Tips

- Start with the brute force and its complexity, then say "the array is sorted (or symmetric), so I can discard one end each step". That sentence is what interviewers wait for.
- Say the invariant: "everything before `w` is the answer so far", or "every pair involving an index outside `[l, r]` has been ruled out".
- Ask: Is the input sorted? Can I modify it? Can there be duplicates? Is the answer a value, indexes, or a list? Should the answer be unique?
- Edge cases to name: empty array, one element, all equal, all zeros, negatives, `n < 3` for 3Sum, `n < 4` for 4Sum, a string with no alphanumeric characters.
- Typical follow-ups: "do it without sorting" (hash map, more space), "what if the input is a stream" (hash set instead), "extend to k-Sum" (recursion: reduce to 2Sum, O(n^(k-1))), "at most k duplicates" (compare with `nums[w-k]`), "do it in one pass" (Dutch flag instead of counting).
- For Trapping Rain Water, walk the ladder: brute force, then prefix/suffix, then two pointers. Explain that two pointers is the prefix version with the arrays removed.
- State complexity including the sort: 3Sum is O(n^2) because the sort O(n log n) is smaller.

## Practice List

**Easy**
- Two Sum II (variants with 0-based output)
- Is Subsequence
- Remove Duplicates from Sorted Array
- Backspace String Compare
- Reverse Vowels of a String
- Valid Palindrome II
- Linked List Cycle (fast/slow)

**Medium**
- Find the Duplicate Number
- Rotate Array
- Next Permutation
- Subarray Product Less Than K
- Max Number of K-Sum Pairs
- Number of Subsequences That Satisfy the Given Sum Condition
- Longest Repeating Character Replacement (window)
- 4Sum II (hash map)

**Hard**
- Median of Two Sorted Arrays
- Sliding Window Maximum
- Substring with Concatenation of All Words
- Smallest Range Covering Elements from K Lists

## Key Takeaways

- **Two pointers replaces a nested loop** when each step can throw away an index forever.
- **Opposite ends needs a reason**: sorted order, symmetry, or a limiting side (the shorter wall in Container With Most Water).
- **Read/write pointers** edit arrays in place: `w` marks the finished prefix, `r` scans.
- **k-Sum = sort + (k-2) loops + two pointers**; skip duplicates by comparing with the previous element.
- **Merge in place from the back**; the empty space is at the end.
- **Trapping Rain Water ladder**: brute force, prefix/suffix, stack, two pointers; the last drops space to O(1).
- **Sort with a numeric comparator** in JavaScript, and remember sorting loses indexes.

---

[← Previous: Strings](../04-strings/README.md) | [Next: Sliding Window →](../06-sliding-window/README.md) | [Back to Core Linear Index](../README.md)
