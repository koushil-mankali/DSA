# Intervals and Sweep Line

> **Estimated Time:** 5 hr | **Difficulty:** Intermediate
> **Prerequisites:** [02 - Sorting](../../01-core-linear/02-sorting/README.md), [05 - Two Pointers](../../01-core-linear/05-two-pointers/README.md), [07 - Prefix Sum](../../01-core-linear/07-prefix-sum/README.md), [04 - Greedy](../04-greedy/README.md), [05 - Heaps](../05-heaps/README.md)

---

## What You'll Learn
- You can decide in seconds whether two intervals overlap, touch, or are apart.
- You can merge, insert, and remove intervals after one sort.
- You can pick between "sort by start" and "sort by end" and say why.
- You can count how many things happen at the same time with a +1/-1 sweep line.
- You can design small calendar and range classes on top of sorted interval lists.

## The Big Idea

Think of a hotel front desk. Each booking is a pair: check-in day and check-out day. The manager wants to know: can two bookings share a room? How many rooms do we need on the busiest night? Which days have no guests? If the bookings are in random order, every question needs a lot of comparing. If you put them in date order, each question needs just one walk down the list.

That is the whole chapter. An **interval** is a pair `[start, end]`. Almost every interval problem has the same first move: **sort**. After sorting, only neighbours can interact, so one pass is enough. The cost is O(n log n) for the sort and O(n) for the pass.

There are two main tools:
1. **Sort and scan.** Sort by start (merging, inserting, calendars) or by end (greedy "keep as many as possible").
2. **Sweep line.** Turn each interval into two events: `+1` at the start, `-1` at the end. Walk through the events in time order and keep a running count. The count is "how many are active right now".

Interviewers love intervals because the code is short but the traps are many: touching ends, equal times, sort order, and empty input. It also tests whether you can reason about greedy choices.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "merge overlapping", "combine ranges" | sort by start, extend the last interval |
| "insert a new interval into a sorted list" | three phases: before, overlap, after |
| "minimum to remove so none overlap", "max meetings you can attend" | sort by END, greedy keep |
| "minimum arrows / points to hit all intervals" | sort by end, shoot at the end |
| "minimum rooms / platforms / how many at once" | sweep line or two sorted arrays |
| "two sorted lists of intervals" | two pointers, drop the one that ends first |
| "free time", "gaps" | merge first, then read the holes |
| "book if no conflict", "k-booking" | sorted list or a +1/-1 map |
| "cover a range with fewest pieces" | greedy jump: furthest reach so far |
| "for each query, best interval containing it" | sort queries too, offline with a heap |
| "silhouette", "height at each x" | sweep with a max-heap |

## Core Concepts / Template

### 1. Overlap test

Two intervals `a` and `b` overlap when each starts before the other ends:

```
a.start <= b.end  AND  b.start <= a.end      // closed [s, e]: touching counts
a.start <  b.end  AND  b.start <  a.end      // half-open [s, e): touching does not count
```

Always ask: "does `[1,2]` and `[2,3]` overlap?" Meeting problems say no (half-open). Merge Intervals says yes (closed, they merge into `[1,3]`). This single question decides `<` versus `<=` in every solution below.

```
a: |-----|
b:       |-----|        touching at one point
a: |-------|
b:    |--|              b inside a
a: |---|
b:          |---|       apart
```

### 2. Which sort?

| Goal | Sort by | Why |
|---|---|---|
| Merge, insert, calendars | start | neighbours in start order are the only ones that can touch |
| Max non-overlapping, min arrows | end | the one that ends first leaves the most room for the rest |
| Covered intervals | start asc, end desc | the biggest interval with a given start comes first |

### 3. Sort and merge template

```js
function merge(intervals) {
  const a = [...intervals].sort((x, y) => x[0] - y[0]);
  const out = [];
  for (const [s, e] of a) {
    const last = out[out.length - 1];
    if (last && s <= last[1]) last[1] = Math.max(last[1], e); // overlap: extend
    else out.push([s, e]);                                    // gap: new block
  }
  return out;
}
```

### 4. Sweep line template (count active intervals)

Split into starts and ends, sort both, and walk with two pointers.

```js
// half-open [s, e): an end at time t is processed before a start at time t
function maxActive(intervals) {
  const starts = intervals.map(x => x[0]).sort((a, b) => a - b);
  const ends = intervals.map(x => x[1]).sort((a, b) => a - b);
  let cur = 0, best = 0, j = 0;
  for (let i = 0; i < starts.length; i++) {
    while (ends[j] <= starts[i]) { j++; cur--; } // finished before this start
    cur++;
    best = Math.max(best, cur);
  }
  return best;
}
```

The other style builds one event list `[time, +1/-1]`, sorts it, and sums. When times tie, put `-1` first for half-open intervals and `+1` first for closed ones.

Trace for `[[1,4],[2,5],[3,6]]`:

| time | event | active |
|---|---|---|
| 1 | +1 | 1 |
| 2 | +1 | 2 |
| 3 | +1 | 3 (peak) |
| 4 | -1 | 2 |
| 5 | -1 | 1 |
| 6 | -1 | 0 |

When the coordinates are small (for example 0..1000), you can skip sorting and use a difference array, the same trick as in the prefix sum chapter.

## Problems (easy -> hard)

### 1. Meeting Rooms (Can Attend All) — Easy
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Given meeting times `[start, end)`, can one person attend all of them? `[[0,30],[5,10]]` -> `false`.
**Intuition:** After sorting by start, a clash can only happen between neighbours. If a meeting starts before the previous one ends, someone has to be in two places.
**Approach 1 - Brute force:** Compare every pair. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** Sort by start, compare each meeting with the one before it.

| sorted | check | result |
|---|---|---|
| [0,30] | - | - |
| [5,10] | 5 < 30 | clash, false |

```js
function canAttendMeetings(intervals) {
  const a = [...intervals].sort((x, y) => x[0] - y[0]);
  for (let i = 1; i < a.length; i++) if (a[i][0] < a[i - 1][1]) return false;
  return true;
}
```

**Complexity:** Time O(n log n), Space O(n) for the copy (O(1) if you may sort in place).
**Watch out:** `[5,8]` and `[8,10]` do not clash, so the test is `<`, not `<=`. Empty input is `true`.

### 2. Merge Intervals — Medium
**Companies:** Amazon, Google, Meta, Microsoft, Bloomberg
**Problem:** Merge all overlapping intervals. `[[1,3],[2,6],[8,10],[15,18]]` -> `[[1,6],[8,10],[15,18]]`.
**Intuition:** Sorted by start, a block keeps growing as long as the next start is inside it. The first start past the block's end begins a new block.
**Approach 1 - Brute force:** Repeatedly find any overlapping pair and merge it until nothing changes. Time O(n^2) or worse, Space O(n).
**Approach 2 - Optimal:** Sort by start, keep the last merged block, extend or push (template above).

| next | last block | action |
|---|---|---|
| [1,3] | - | push [1,3] |
| [2,6] | [1,3] | 2 <= 3, end = 6 |
| [8,10] | [1,6] | 8 > 6, push |
| [15,18] | [8,10] | 15 > 10, push |

```js
function merge(intervals) {
  const a = [...intervals].sort((x, y) => x[0] - y[0]);
  const out = [];
  for (const [s, e] of a) {
    const last = out[out.length - 1];
    if (last && s <= last[1]) last[1] = Math.max(last[1], e);
    else out.push([s, e]);
  }
  return out;
}
```

**Complexity:** Time O(n log n), Space O(n) for the output.
**Watch out:** Use `Math.max` for the end: `[1,10]` followed by `[2,3]` must stay `[1,10]`. `[1,4]` and `[4,5]` merge (closed intervals).

### 3. Insert Interval — Medium
**Companies:** Google, Amazon, Meta, LinkedIn
**Problem:** The list is already sorted and disjoint. Insert one more interval and merge as needed. `[[1,3],[6,9]]` + `[2,5]` -> `[[1,5],[6,9]]`.
**Intuition:** The list splits into three parts around the new interval: those that end before it, those that overlap it, and those that start after it. Only the middle group changes, and it all collapses into one interval.
**Approach 1 - Brute force:** Push it in, sort, run merge. Time O(n log n), Space O(n).
**Approach 2 - Optimal:** One pass, three phases, no sort needed.

For `[[1,2],[3,5],[6,7],[8,10],[12,16]]` and `[4,8]`:

| phase | intervals | effect |
|---|---|---|
| before (end < 4) | [1,2] | copied |
| overlap (start <= 8) | [3,5], [6,7], [8,10] | new becomes [3,10] |
| after | [12,16] | copied |

```js
function insert(intervals, newInterval) {
  const out = [];
  let i = 0;
  const n = intervals.length;
  let [s, e] = newInterval;
  while (i < n && intervals[i][1] < s) out.push(intervals[i++]); // fully before
  while (i < n && intervals[i][0] <= e) { // overlaps: grow the new one
    s = Math.min(s, intervals[i][0]);
    e = Math.max(e, intervals[i][1]);
    i++;
  }
  out.push([s, e]);
  while (i < n) out.push(intervals[i++]); // fully after
  return out;
}
```

**Complexity:** Time O(n), Space O(n) for the output.
**Watch out:** Empty list, new interval before everything, new interval after everything. The result must still be sorted.

### 4. Non-overlapping Intervals — Medium
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Remove the fewest intervals so the rest do not overlap. `[[1,2],[2,3],[3,4],[1,3]]` -> `1` (remove `[1,3]`).
**Intuition:** "Remove the fewest" is the same as "keep the most". This is the classic activity selection problem. Among overlapping intervals, always keep the one that ends first, because it blocks the least of the future.
**Approach 1 - Brute force:** Try all subsets, check each. Time O(2^n * n), Space O(n).
**Approach 2 - Optimal (greedy):** Sort by end. Keep an interval if it starts at or after the last kept end. Answer is `n - kept`.

| sorted by end | lastEnd | action |
|---|---|---|
| [1,2] | -inf | keep, lastEnd 2 |
| [2,3] | 2 | 2 >= 2 keep, lastEnd 3 |
| [1,3] | 3 | 1 < 3 drop |
| [3,4] | 3 | keep |

```js
function eraseOverlapIntervals(intervals) {
  const a = [...intervals].sort((x, y) => x[1] - y[1]);
  let kept = 0, lastEnd = -Infinity;
  for (const [s, e] of a) {
    if (s >= lastEnd) { kept++; lastEnd = e; }
  }
  return a.length - kept;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** Sorting by start fails: `[1,100]` would be kept and block many small ones. Touching ends (`[1,2]`, `[2,3]`) are fine here. Use `-Infinity` as the start value since times can be negative.

### 5. Minimum Number of Arrows to Burst Balloons — Medium
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** Each balloon is a horizontal range `[xs, xe]`. An arrow shot straight up at `x` bursts every balloon with `xs <= x <= xe`. Find the fewest arrows. `[[10,16],[2,8],[1,6],[7,12]]` -> `2`.
**Intuition:** It is the same greedy as problem 4, with one change: touching counts as overlap. Shoot at the end of the balloon that ends first. That arrow hits every balloon that starts at or before that point. Then move on to the first balloon it missed.
**Approach 1 - Brute force:** Try every x from the endpoints and search for the smallest cover. Exponential.
**Approach 2 - Optimal:** Sort by end. Start a new arrow when a balloon starts after the current arrow position.

| sorted by end | arrow at | action |
|---|---|---|
| [1,6] | none | new arrow at 6 |
| [2,8] | 6 | 2 <= 6, hit |
| [7,12] | 6 | 7 > 6, new arrow at 12 |
| [10,16] | 12 | 10 <= 12, hit |

```js
function findMinArrowShots(points) {
  const a = [...points].sort((x, y) => x[1] - y[1]);
  let arrows = 0, pos = -Infinity;
  for (const [s, e] of a) {
    if (s > pos) { arrows++; pos = e; } // compare, never subtract
  }
  return arrows;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** Sort with `x[1] - y[1]` is safe in JS, but in Java or C++ the subtraction overflows for values near the 32-bit limits (this problem has such tests). Say that you would use a comparison there. Answer here equals "max number of mutually overlapping groups", which is why it matches problem 4 with `>` instead of `>=`.

### 6. Remove Covered Intervals — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** Count the intervals that remain after removing every interval fully inside another one. `[[1,4],[3,6],[2,8]]` -> `2` (`[3,6]` is inside `[2,8]`).
**Intuition:** Sort by start. If an interval ends at or before the largest end seen so far, an earlier interval (with start <= its start) already covers it. Ties on start: put the longer one first, so the shorter is seen as covered.
**Approach 1 - Brute force:** Check every pair. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** Sort by start asc, end desc. Track `maxEnd`. If `end > maxEnd`, the interval survives.

| sorted | maxEnd | survives? |
|---|---|---|
| [1,4] | -inf | yes, maxEnd 4 |
| [2,8] | 4 | yes, maxEnd 8 |
| [3,6] | 8 | 6 <= 8, covered |

```js
function removeCoveredIntervals(intervals) {
  const a = [...intervals].sort((x, y) => x[0] - y[0] || y[1] - x[1]);
  let count = 0, maxEnd = -Infinity;
  for (const [, e] of a) {
    if (e > maxEnd) { count++; maxEnd = e; }
  }
  return count;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** Without the "end descending" tie-break, `[1,2]` then `[1,4]` would count both. Equal start and end intervals: the second one is covered (`e > maxEnd` is false).

### 7. Interval List Intersections — Medium
**Companies:** Meta, Google, Amazon, Uber
**Problem:** Two lists of sorted, disjoint intervals. Return all intersections. `A=[[0,2],[5,10]]`, `B=[[1,5],[8,12]]` -> `[[1,2],[5,5],[8,10]]`.
**Intuition:** It is a merge step from merge sort. Look at the front of both lists. Their overlap, if any, is `[max(starts), min(ends)]`. Then throw away the interval that ends first, because it cannot meet anything later in the other list.
**Approach 1 - Brute force:** Compare every pair. Time O(n*m), Space O(1) extra.
**Approach 2 - Optimal:** Two pointers.

| A[i] | B[j] | overlap | advance |
|---|---|---|---|
| [0,2] | [1,5] | [1,2] | A (2 < 5) |
| [5,10] | [1,5] | [5,5] | B (5 < 10) |
| [5,10] | [8,12] | [8,10] | A |

```js
function intervalIntersection(A, B) {
  const out = [];
  let i = 0, j = 0;
  while (i < A.length && j < B.length) {
    const lo = Math.max(A[i][0], B[j][0]);
    const hi = Math.min(A[i][1], B[j][1]);
    if (lo <= hi) out.push([lo, hi]);
    if (A[i][1] < B[j][1]) i++; else j++; // drop the one that ends first
  }
  return out;
}
```

**Complexity:** Time O(n + m), Space O(1) beyond the output.
**Watch out:** `lo <= hi` allows a single point like `[5,5]` (the problem uses closed intervals). If one list is empty the loop never runs and the answer is `[]`.

### 8. Maximum Overlap (Most Crowded Time) — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Given `[start, end)` visits, find the largest number of people present at once and a time when it happens. `[[1,4],[2,5],[3,6]]` -> count 3 at time 3.
**Intuition:** The crowd only grows at a start. So check the count right after each start. A sweep gives the count in one pass.
**Approach 1 - Brute force:** For each start time, count how many intervals contain it. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** Sorted starts and sorted ends with two pointers (the template). Before counting a new start, remove everyone whose end is `<=` that start.

See the trace table in the template section: the peak is 3 at time 3.

```js
// half-open [s, e): a meeting ending at t does not overlap one starting at t
function maxOverlap(intervals) {
  const starts = intervals.map(x => x[0]).sort((a, b) => a - b);
  const ends = intervals.map(x => x[1]).sort((a, b) => a - b);
  let cur = 0, best = 0, bestTime = null, j = 0;
  for (let i = 0; i < starts.length; i++) {
    while (ends[j] <= starts[i]) { j++; cur--; }
    cur++;
    if (cur > best) { best = cur; bestTime = starts[i]; }
  }
  return { count: best, time: bestTime };
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** The "Maximum Population Year" problem is the same thing on years (the end year is exclusive). If the problem has inclusive ends, use `ends[j] < starts[i]`. Empty input returns count 0.

### 9. Minimum Number of Platforms — Medium
**Companies:** Amazon, Microsoft, Goldman Sachs, Flipkart
**Problem:** Trains have arrival and departure times (a classic on GeeksforGeeks and TUF). Find the fewest platforms so no train waits. `arr=[900,940,950,1100,1500,1800]`, `dep=[910,1200,1120,1130,1900,2000]` -> `3`.
**Intuition:** It is "maximum overlap" with the arrays already split. The answer is the highest number of trains in the station at one moment. Arrival and departure arrays do not have to be paired, because only the counts matter.
**Approach 1 - Brute force:** For each train, count trains present at its arrival. Time O(n^2), Space O(1).
**Approach 2 - Optimal:** Sort both arrays, two pointers.

| arrival | departures passed | in station |
|---|---|---|
| 900 | none (910 > 900) | 1 |
| 940 | 910 | 1 |
| 950 | none | 2 |
| 1100 | none (1120 > 1100) | 3 |
| 1500 | 1120, 1130, 1200 | 1 |

```js
// a train leaving at t still holds its platform when another arrives at t
function minPlatforms(arr, dep) {
  const a = [...arr].sort((x, y) => x - y);
  const d = [...dep].sort((x, y) => x - y);
  let need = 0, best = 0, j = 0;
  for (let i = 0; i < a.length; i++) {
    while (d[j] < a[i]) { j++; need--; }
    need++;
    best = Math.max(best, need);
  }
  return best;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** Here equal times count as overlap (`<`), the opposite of meeting rooms. Always confirm the tie rule with the interviewer. If a departure comes before the matching arrival in the input, the data is invalid; mention you assume `arr[i] <= dep[i]`.

### 10. Meeting Rooms II — Medium
**Companies:** Amazon, Google, Meta, Microsoft, Bloomberg
**Problem:** Find the minimum number of rooms for all meetings `[start, end)`. `[[0,30],[5,10],[15,20]]` -> `2`.
**Intuition:** A new room is needed only if every room in use is still busy when a meeting starts. The earliest-ending running meeting tells you whether a room is free.
**Approach 1 - Min-heap of end times:** Sort by start. The heap holds the end times of rooms in use. If the earliest end is `<=` the new start, pop it (reuse the room). Push the new end. Heap size at the end is the answer. Time O(n log n), Space O(n).
**Approach 2 - Optimal (two sorted arrays):** No heap. Sort starts and ends separately. A pointer on ends tracks the earliest room to free up.

| start | earliest end | action | rooms |
|---|---|---|---|
| 0 | 10 | 0 < 10, new room | 1 |
| 5 | 10 | 5 < 10, new room | 2 |
| 15 | 10 | 15 >= 10, reuse, move to end 20 | 2 |

```js
function minMeetingRooms(intervals) {
  const starts = intervals.map(x => x[0]).sort((a, b) => a - b);
  const ends = intervals.map(x => x[1]).sort((a, b) => a - b);
  let rooms = 0, j = 0;
  for (const s of starts) {
    if (s >= ends[j]) j++; // a room freed up: reuse it
    else rooms++;
  }
  return rooms;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** It works because we only count rooms, not which meeting sits in which room. If asked to assign rooms, use the heap version. `[1,5],[5,8]` needs one room (`>=`). Empty input gives 0 (the loop does not run).

### 11. Car Pooling — Medium
**Companies:** Google, Amazon, Microsoft, Uber
**Problem:** Trips are `[passengers, from, to]`. The car has `capacity` seats. Can it do every trip? `[[2,1,5],[3,3,7]]`, capacity 4 -> `false`; capacity 5 -> `true`.
**Intuition:** This is a weighted sweep line. At `from` the load goes up by `passengers`, at `to` it goes down. If the load ever exceeds capacity, answer `false`.
**Approach 1 - Brute force:** For each stop position, add up passengers of trips covering it. Time O(n * range), Space O(1).
**Approach 2 - Events:** Sort `[time, delta]` pairs. On ties, drop-offs (negative) come first, since passengers leave before new ones board. Time O(n log n).
**Approach 3 - Difference array:** The problem limits stops to 1000, so a `diff` array of size 1001 plus a running sum gives O(n + 1000). Same idea, no sort.

| time | delta | load (cap 4) |
|---|---|---|
| 1 | +2 | 2 |
| 3 | +3 | 5 > 4, false |

```js
function carPooling(trips, capacity) {
  const ev = [];
  for (const [p, from, to] of trips) { ev.push([from, p]); ev.push([to, -p]); }
  ev.sort((a, b) => a[0] - b[0] || a[1] - b[1]); // drop-offs (negative) first on ties
  let load = 0;
  for (const [, d] of ev) {
    load += d;
    if (load > capacity) return false;
  }
  return true;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** The tie rule. With `[3,2,7],[3,7,9]` the first group leaves at 7 as the second boards, so the load stays 3. Wrong tie order gives 6.

### 12. Video Stitching — Medium
**Companies:** Google, Amazon, Uber
**Problem:** Clips `[start, end]` can be cut and joined. Find the fewest clips that cover `[0, time]`, or `-1`. `clips=[[0,2],[4,6],[8,10],[1,9],[1,5],[5,9]]`, `time=10` -> `3` (`[0,2]`, `[1,9]`, `[8,10]`).
**Intuition:** It is the "Jump Game II" idea. From the current covered point, you may use any clip that starts at or before it. Always take the one that reaches furthest. When you hit the end of the current reach, you must take a new clip.
**Approach 1 - Brute force / DP:** `dp[t]` = fewest clips to cover `[0,t]`, try every clip for every t. Time O(time * n), Space O(time).
**Approach 2 - Optimal (greedy):** Precompute `reach[s]` = furthest end among clips starting at `s`. Walk positions; at each boundary, jump to the furthest reach.

For the example: reach[0]=2, reach[1]=9, reach[4]=6, reach[5]=9, reach[8]=10.

| i | far | curEnd | action |
|---|---|---|---|
| 0 | 2 | 0 -> 2 | i == curEnd, count 1 |
| 1 | 9 | 2 | - |
| 2 | 9 | 2 -> 9 | count 2 |
| 9 | 10 | 9 -> 10 | count 3, done |

```js
function videoStitching(clips, time) {
  const reach = new Array(time + 1).fill(0);
  for (const [s, e] of clips) if (s <= time) reach[s] = Math.max(reach[s], e);
  let count = 0, curEnd = 0, far = 0;
  for (let i = 0; i < time; i++) {
    far = Math.max(far, reach[i]);
    if (i === curEnd) {
      if (far <= i) return -1; // gap: nothing extends past here
      count++;
      curEnd = far;
    }
  }
  return curEnd >= time ? count : -1;
}
```

**Complexity:** Time O(n + time), Space O(time).
**Watch out:** A gap such as `[[0,1],[1,2]]` with `time=5` returns `-1`. No clip starting at 0 also returns `-1` (`far <= i` at i=0). `time=0` returns 0.

### 13. Employee Free Time — Hard
**Companies:** Google, Amazon, Meta, Airbnb, Uber
**Problem:** Each employee has a sorted list of busy intervals. Return the finite intervals when everybody is free. `[[[1,2],[5,6]],[[1,3]],[[4,10]]]` -> `[[3,4]]`.
**Intuition:** Everyone is free exactly where nobody is busy. Throw all busy intervals into one pile and sort by start. Walk through while tracking the furthest end so far. A gap before the next start is free time. (The LeetCode version uses `Interval` objects with `.start` and `.end`; here they are `[s, e]` pairs.)
**Approach 1 - Merge all:** Flatten, sort, merge, read the holes. Time O(N log N) for N total intervals, Space O(N).
**Approach 2 - Better:** A min-heap holding the next interval of each employee, since each list is already sorted. Time O(N log k) for k employees. Same sweep afterwards. Mention it, since many interviewers ask for it.

Trace on the example, sorted: `[1,2] [1,3] [4,10] [5,6]`.

| interval | end so far | action |
|---|---|---|
| [1,2] | 2 | - |
| [1,3] | 3 | - |
| [4,10] | 3 | 4 > 3, free [3,4]; end 10 |
| [5,6] | 10 | - |

```js
// schedule[i] = list of [start, end] for employee i
function employeeFreeTime(schedule) {
  const all = schedule.flat().sort((x, y) => x[0] - y[0]);
  const free = [];
  let end = all.length ? all[0][1] : 0;
  for (const [s, e] of all) {
    if (s > end) free.push([end, s]);
    end = Math.max(end, e);
  }
  return free;
}
```

**Complexity:** Time O(N log N), Space O(N).
**Watch out:** Use `Math.max` for `end`, because `[4,10]` followed by `[5,6]` must not shrink it. Time before the first busy slot and after the last is not "free" (it is infinite), so it is excluded.

### 14. My Calendar I — Medium
**Companies:** Google, Amazon, Meta, Microsoft
**Problem:** `book(start, end)` returns `true` and saves the booking if it does not overlap any earlier one (half-open `[start, end)`), else `false`.
**Intuition:** A new booking conflicts with an existing one when `start < e && s < end`. Touching is fine.
**Approach 1 - List scan:** Check against every saved booking. `book` is O(n), total O(n^2).
**Approach 2 - Better:** Keep bookings sorted (a balanced tree or sorted array with binary search). Find the neighbours of `start` and check only those two. O(log n) per search. JavaScript has no built-in tree map, so say what you would use (a sorted array with binary search gives O(log n) search and O(n) insert).

```js
class MyCalendar {
  constructor() { this.books = []; }
  book(start, end) {
    for (const [s, e] of this.books) if (start < e && s < end) return false;
    this.books.push([start, end]);
    return true;
  }
}
```

**Complexity:** Time O(n) per booking, Space O(n).
**Watch out:** `book(10,20)` then `book(20,30)` is allowed. Do not store a rejected booking.

### 15. My Calendar II — Medium
**Companies:** Google, Amazon, Meta
**Problem:** Same as I, but a double booking is allowed. Reject a booking only if it would create a triple booking.
**Intuition:** Keep two lists: all bookings, and the regions that are already double booked. A new booking is bad if it touches any double-booked region. If it is fine, its overlaps with existing single bookings become new double-booked regions.
**Approach 1 - Two lists (shown):** O(n) per booking.
**Approach 2 - Sweep with a map:** Store +1/-1 at start/end, scan in sorted order and reject if the count hits 3. O(n log n) per booking. Simpler to get right under pressure, slower.

Trace: book(10,20) ok. book(50,60) ok. book(10,40): overlaps [10,20], so doubles = [[10,20]]. book(5,15): hits double [10,20], false.

```js
class MyCalendarTwo {
  constructor() { this.books = []; this.overlaps = []; }
  book(start, end) {
    for (const [s, e] of this.overlaps) if (start < e && s < end) return false; // would be a triple
    for (const [s, e] of this.books) {
      if (start < e && s < end) this.overlaps.push([Math.max(s, start), Math.min(e, end)]);
    }
    this.books.push([start, end]);
    return true;
  }
}
```

**Complexity:** Time O(n) per booking, Space O(n).
**Watch out:** Check the double-booked list first, and only add new overlaps after you know the booking is accepted. If you add them first, a rejected booking leaves ghost overlaps behind.

### 16. My Calendar III — Hard
**Companies:** Google, Amazon, Meta
**Problem:** After every `book(start, end)`, return the largest k such that some time is booked k times.
**Intuition:** This is the sweep line in its purest form. Keep a map `time -> change`. Add +1 at start, -1 at end. To answer, sum the changes in time order and take the peak.
**Approach 1 - Sorted map (shown):** Sort keys each call. Time O(n log n) per booking.
**Approach 2 - Optimal:** A dynamic segment tree with lazy range-add and a global max gives O(log C) per booking, where C is the coordinate range. See the segment tree chapter: [02 - Segment Tree and Fenwick](../../06-advanced/02-segment-tree-and-fenwick/README.md). A balanced tree map with prefix maximums is the other common answer.

Trace: book(10,20) -> 1. book(50,60) -> 1. book(10,40): at 10 the count is 2 -> 2. book(5,15): at 10 the count is 3 -> 3.

```js
class MyCalendarThree {
  constructor() { this.diff = new Map(); }
  book(start, end) {
    this.diff.set(start, (this.diff.get(start) || 0) + 1);
    this.diff.set(end, (this.diff.get(end) || 0) - 1);
    let cur = 0, best = 0;
    for (const t of [...this.diff.keys()].sort((a, b) => a - b)) {
      cur += this.diff.get(t);
      best = Math.max(best, cur);
    }
    return best;
  }
}
```

**Complexity:** Time O(n log n) per booking, Space O(n).
**Watch out:** Sort keys numerically (`(a, b) => a - b`), because the default sort compares strings and puts 100 before 20.

### 17. Data Stream as Disjoint Intervals — Hard
**Companies:** Google, Amazon, Meta
**Problem:** Numbers arrive one at a time. After each one, `getIntervals()` returns the numbers seen so far as sorted disjoint ranges. Add 1, 3, 7, 2, 6 -> `[[1,3],[6,7]]`.
**Intuition:** Keep the ranges in a sorted list. A new number `v` is either already inside a range, extends one, bridges two (the previous ends at `v-1` and the next starts at `v+1`), or is alone.
**Approach 1 - Store all numbers:** Keep a sorted set and build ranges on `getIntervals`. add O(log n), get O(n).
**Approach 2 - Optimal:** Keep ranges directly. Binary search for where `v` goes, then check the left and right neighbour (shown). Search is O(log n); `splice` makes insert O(n) in an array. A tree map would give O(log n) overall.

| add | action | state |
|---|---|---|
| 1 | new | [1,1] |
| 3 | new | [1,1] [3,3] |
| 7 | new | [1,1] [3,3] [7,7] |
| 2 | bridges [1,1] and [3,3] | [1,3] [7,7] |
| 6 | joins next start 7 | [1,3] [6,7] |

```js
class SummaryRanges {
  constructor() { this.iv = []; } // sorted, disjoint, closed [s, e]
  addNum(v) {
    const iv = this.iv;
    let lo = 0, hi = iv.length; // first interval whose start > v
    while (lo < hi) {
      const m = (lo + hi) >> 1;
      if (iv[m][0] > v) hi = m; else lo = m + 1;
    }
    const prev = lo > 0 ? iv[lo - 1] : null;
    const next = lo < iv.length ? iv[lo] : null;
    if (prev && prev[1] >= v) return; // already inside
    const joinPrev = prev && prev[1] + 1 === v;
    const joinNext = next && next[0] === v + 1;
    if (joinPrev && joinNext) { prev[1] = next[1]; iv.splice(lo, 1); }
    else if (joinPrev) prev[1] = v;
    else if (joinNext) next[0] = v;
    else iv.splice(lo, 0, [v, v]);
  }
  getIntervals() { return this.iv.map(x => [...x]); }
}
```

**Complexity:** Time O(log n) search + O(n) splice per add, O(n) per get. Space O(n).
**Watch out:** Duplicates must do nothing. Return copies from `getIntervals` so callers cannot change your state. Follow-up: "what if there are many merges and few distinct ranges?" The range count stays small, so the splice cost is small too.

### 18. Range Module (concept) — Hard
**Companies:** Google, Amazon, Meta
**Problem:** Design a class that tracks half-open ranges: `addRange(l, r)`, `removeRange(l, r)`, `queryRange(l, r)` (true if every number in `[l, r)` is tracked).
**Intuition:** It is Insert Interval (problem 3) plus a "cut a hole" operation. Keep a sorted list of disjoint ranges. Add merges, remove splits, query checks that one stored range contains the query fully (a covered query cannot span two stored ranges, because touching ranges are always merged).
**Approach 1 - Array of booleans:** Works only for tiny coordinates. The real limit is 10^9, so no.
**Approach 2 - Sorted disjoint list (shown):** Simple, O(n) per operation. In an interview, start here and say it out loud.
**Approach 3 - Optimal:** Binary search the list, or use a balanced tree, for O(log n + k) per operation (k is the ranges touched). A dynamic segment tree with lazy assignment is the other known solution.

Trace: add [10,20). remove [14,16) -> [10,14) [16,20). query [13,15) -> false. add [14,16) -> merges to [10,20).

```js
class RangeModule { // half-open [l, r), sorted disjoint list, O(n) per op
  constructor() { this.iv = []; }
  addRange(l, r) {
    const out = [];
    let i = 0;
    const iv = this.iv;
    while (i < iv.length && iv[i][1] < l) out.push(iv[i++]);
    while (i < iv.length && iv[i][0] <= r) { l = Math.min(l, iv[i][0]); r = Math.max(r, iv[i][1]); i++; }
    out.push([l, r]);
    while (i < iv.length) out.push(iv[i++]);
    this.iv = out;
  }
  queryRange(l, r) {
    return this.iv.some(([s, e]) => s <= l && r <= e);
  }
  removeRange(l, r) {
    const out = [];
    for (const [s, e] of this.iv) {
      if (e <= l || s >= r) out.push([s, e]);
      else {
        if (s < l) out.push([s, l]);
        if (e > r) out.push([r, e]);
      }
    }
    this.iv = out;
  }
}
```

**Complexity:** Time O(n) per call with this version, Space O(n).
**Watch out:** In `addRange`, `iv[i][1] < l` (strict) so that `[1,3)` and `[3,5)` merge into `[1,5)`. In `removeRange`, a stored range can split into two pieces.

### 19. Minimum Interval to Include Each Query — Hard
**Companies:** Google, Amazon, Meta
**Problem:** For each query `q`, find the size (`end - start + 1`) of the smallest interval with `start <= q <= end`, or `-1`. `intervals=[[1,4],[2,4],[3,6],[4,4]]`, `queries=[2,3,4,5]` -> `[3,3,1,4]`.
**Intuition:** Answering queries in random order is slow. If you sort the queries, you can add intervals once as the query point moves right, and drop them once they are behind it. A min-heap keyed by size gives the smallest active interval. This trick is called offline processing: you reorder the questions, then put the answers back at their original index.
**Approach 1 - Brute force:** For each query scan all intervals. Time O(n * q), Space O(1).
**Approach 2 - Optimal:** Sort intervals by start, sort query indexes by value. For each query: push all intervals with `start <= q` into the heap as `[size, end]`; pop while the top has `end < q`; the top is the answer. Expired intervals deeper in the heap are fine, they will be removed when they reach the top.

Trace for query 3 (after query 2): heap holds sizes 4 ([1,4]) and 3 ([2,4]). Push [3,6] (size 4). Top is size 3, end 4 >= 3. Answer 3.

```js
class Heap { // tiny binary heap; less(a, b) must return a boolean
  constructor(less) { this.a = []; this.less = less; }
  get size() { return this.a.length; }
  top() { return this.a[0]; }
  push(x) {
    const a = this.a; a.push(x);
    let i = a.length - 1;
    while (i > 0) {
      const p = (i - 1) >> 1;
      if (!this.less(a[i], a[p])) break;
      [a[i], a[p]] = [a[p], a[i]]; i = p;
    }
  }
  pop() {
    const a = this.a, top = a[0], last = a.pop();
    if (a.length) {
      a[0] = last;
      let i = 0;
      for (;;) {
        let m = i; const l = 2 * i + 1, r = l + 1;
        if (l < a.length && this.less(a[l], a[m])) m = l;
        if (r < a.length && this.less(a[r], a[m])) m = r;
        if (m === i) break;
        [a[i], a[m]] = [a[m], a[i]]; i = m;
      }
    }
    return top;
  }
}

function minInterval(intervals, queries) {
  const iv = [...intervals].sort((a, b) => a[0] - b[0]);
  const order = queries.map((q, i) => i).sort((a, b) => queries[a] - queries[b]);
  const heap = new Heap((x, y) => x[0] < y[0] || (x[0] === y[0] && x[1] < y[1])); // [size, end]
  const res = new Array(queries.length).fill(-1);
  let i = 0;
  for (const qi of order) {
    const q = queries[qi];
    while (i < iv.length && iv[i][0] <= q) { heap.push([iv[i][1] - iv[i][0] + 1, iv[i][1]]); i++; }
    while (heap.size && heap.top()[1] < q) heap.pop(); // expired
    if (heap.size) res[qi] = heap.top()[0];
  }
  return res;
}
```

**Complexity:** Time O((n + q) log (n + q)), Space O(n + q).
**Watch out:** Store answers by the original query index. Duplicate queries are fine. A heap `less` that returns a number instead of a boolean is a real bug (it passes for the wrong reason, `-3` is truthy). This is the version of the heap used by JavaScript interviews since there is no built-in; see [05 - Heaps](../05-heaps/README.md).

### 20. The Skyline Problem — Hard
**Companies:** Google, Amazon, Meta, Microsoft, Uber
**Problem:** Buildings are `[left, right, height]`. Return the key points `[x, height]` of the outline where the height changes. `[[2,9,10],[3,7,15],[5,12,12],[15,20,10],[19,24,8]]` -> `[[2,10],[3,15],[7,12],[12,0],[15,10],[20,8],[24,0]]`.
**Intuition:** At any x, the skyline is the tallest building covering x. Sweep left to right. At each building edge, ask "what is the tallest active building now?" If that height differs from the last output height, record a key point. A max-heap gives the tallest. Buildings that already ended are removed lazily: only when they reach the top.
**Approach 1 - Brute force:** Compress coordinates, for each x take the max over all buildings. Time O(n^2), Space O(n).
**Approach 2 - Optimal:** Events plus a max-heap.
  - Start event: `[l, -h, r]`. End event: `[r, 0, 0]`.
  - Sort by x, then by the second field. This puts taller starts first, and starts before ends at the same x. That avoids false dips between touching buildings.
  - Heap holds `[height, right]`, seeded with a ground `[0, Infinity]`.

Trace on `[[2,9,10],[3,7,15]]` (part):

| x | event | heap top after cleanup | output |
|---|---|---|---|
| 2 | start h10 | 10 | [2,10] |
| 3 | start h15 | 15 | [3,15] |
| 7 | end | 15 expired (right 7 <= 7), top 10 | [7,10] |
| 9 | end | 10 expired, top 0 | [9,0] |

```js
// uses the Heap class from problem 19
function getSkyline(buildings) {
  const ev = [];
  for (const [l, r, h] of buildings) { ev.push([l, -h, r]); ev.push([r, 0, 0]); }
  // same x: taller starts first; starts (-h) before ends (0)
  ev.sort((a, b) => a[0] - b[0] || a[1] - b[1]);
  const heap = new Heap((x, y) => x[0] > y[0]); // max-heap of [height, rightEdge]
  heap.push([0, Infinity]);
  const out = [];
  let prev = 0;
  for (const [x, negH, r] of ev) {
    if (negH < 0) heap.push([-negH, r]);
    while (heap.top()[1] <= x) heap.pop(); // lazy delete finished buildings
    const h = heap.top()[0];
    if (h !== prev) { out.push([x, h]); prev = h; }
  }
  return out;
}
```

**Complexity:** Time O(n log n), Space O(n).
**Watch out:** Two buildings starting at the same x must output only the taller one. The same-x ordering above and the `h !== prev` check handle it. Touching buildings `[0,2,3],[2,5,3]` must give `[[0,3],[5,0]]`, with no dip at 2. Empty input gives `[]`. The last point always has height 0.

## Common Mistakes
- **"Sort by start always works."** For "keep the most intervals" and "fewest arrows" you must sort by **end**. Sorting by start lets one huge interval block many small ones.
- **"Overlap always means `<=`."** It depends on the problem. Meetings and calendars are half-open (touching is fine). Merge Intervals and arrows are closed (touching counts). Decide this first.
- **"The new end is just the current end."** When merging, use `Math.max(lastEnd, end)`, since the next interval can sit fully inside the block.
- **"Order of events at the same time does not matter."** It does. Car pooling and meeting rooms process ends first; closed-interval problems process starts first. Wrong order gives off-by-one counts.
- **"JS default sort is fine for numbers."** `[10, 9, 1].sort()` gives `[1, 10, 9]`. Always pass `(a, b) => a - b`.
- **"Sorting a copy is not needed."** `intervals.sort` changes the caller's array. Copy first if the input should not change, or say you are mutating on purpose.
- **"A heap comparator can return a number."** A custom heap that expects a boolean breaks silently if you give it `x - y`.

## Interview Tips
- **Say the first move:** "I will sort by start so only neighbours can interact." Then state your overlap rule (closed or half-open) before writing code.
- **Ask:** Are intervals sorted? Can they be empty or have `start == end`? Can there be duplicates? Are the ends inclusive?
- **Choose greedy by what you optimize:** "maximize count kept" -> sort by end; "merge everything" -> sort by start.
- **Sweep line pitch:** "Each interval becomes a +1 at the start and a -1 at the end. The running sum is the number active. The peak is the answer." This one sentence solves rooms, platforms, car pooling, and calendar III.
- **Follow-ups you will hear:** "Can you avoid the sort?" (small coordinates: difference array.) "What if the data streams in?" (sorted structure or tree map.) "What if it does not fit in memory?" (external sort, or sweep over chunks by time.) "Which room does each meeting get?" (min-heap of rooms.)
- **Edge cases to say out loud:** empty input, single interval, all identical, one interval containing all others, touching ends, negative times, huge values (compare instead of subtract in typed languages).
- **State complexity:** sorting dominates, O(n log n). The scan is O(n).

## Practice List
**Easy**
- Meeting Rooms
- Summary Ranges
- Maximum Population Year
- Teemo Attacking
- Missing Ranges
- Find Right Interval (as a sorted search warm-up)

**Medium**
- Merge Intervals, Insert Interval, Non-overlapping Intervals
- Minimum Number of Arrows to Burst Balloons
- Meeting Rooms II
- Interval List Intersections
- Remove Covered Intervals
- Car Pooling
- Video Stitching
- My Calendar I, My Calendar II
- Partition Labels
- Add Bold Tag in String / Bold Words
- Divide Intervals Into Minimum Number of Groups
- Count Days Without Meetings
- Meeting Scheduler
- Corporate Flight Bookings
- Maximum Number of Events That Can Be Attended

**Hard**
- Employee Free Time
- My Calendar III
- The Skyline Problem
- Data Stream as Disjoint Intervals
- Range Module
- Minimum Interval to Include Each Query
- Maximum Number of Events That Can Be Attended II
- Rectangle Area II
- Falling Squares
- Perfect Rectangle

## Key Takeaways
- **Sort first.** After sorting by start, only neighbours can interact, so one pass is enough. Total cost is O(n log n).
- **Decide closed or half-open before coding.** It sets every `<` versus `<=` and the tie order of events.
- **Sort by end for greedy selection.** Keeping the interval that finishes first leaves the most room. This solves non-overlapping intervals and balloons.
- **Merge with `Math.max` on the end.** A later interval can lie fully inside the current block.
- **Sweep line = +1 at start, -1 at end.** The running sum is the number active; its peak answers rooms, platforms, car pooling, and calendars.
- **Two sorted lists means two pointers.** Drop the interval that ends first; it cannot match anything later.
- **Sort the queries too (offline).** With sorted queries and a heap, one sweep answers all of them.

---
[← Previous: Heaps](../05-heaps/README.md) | [Next: Bit Manipulation →](../07-bit-manipulation/README.md) | [Back to Intermediate Index](../README.md)
