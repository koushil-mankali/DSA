# Problem-Solving Framework

> **Estimated Time:** 2 hr | **Difficulty:** Beginner
> **Prerequisites:** None

---

## What You'll Learn

- You can follow a fixed 7-step method (clarify, examples, brute force, optimize, code, test, complexity) on any interview problem.
- You can read the input limits (n up to 1e5, and so on) and guess the target time complexity before you write code.
- You can think aloud in a way that helps the interviewer help you.
- You can run an edge-case checklist and a "stuck" checklist when your mind goes blank.
- You can pick a data structure by asking what operation the problem repeats.

## The Big Idea

Think of a doctor in an emergency room. She does not guess a cure at once. She asks questions, looks at the symptoms, tries the safe treatment first, and only then tries the clever one. She also says what she is doing, so the team can follow. A coding interview is the same. A calm, ordered method beats a flash of genius.

Most people fail interviews not because they lack knowledge, but because they jump into code too early, get lost, and then panic. A method gives your brain a next step when it is empty.

Why interviewers love it: they are not only grading the final code. They are grading how you work. Do you ask about unclear parts? Do you start with something correct? Do you improve it with a reason? Do you test your own work? A candidate who does these things is someone they want on their team, even if the final code has a small bug.

## How to Spot It

This chapter is about the process, so the "spot it" table is about signals in the problem text and limits that tell you where to go next.

| If the problem says / shows ... | think ... |
|---|---|
| "Find a pair / triple / subarray with property X" | Brute force is nested loops. Ask what you can remember (hash map) or sort. |
| "Sorted array" or "sorted input" | Binary search or two pointers. |
| "Contiguous subarray / substring" | Sliding window or prefix sums. |
| "All combinations / permutations / subsets" | Backtracking. The answer itself is exponential. |
| "Minimum / maximum / count ways" with overlapping choices | Dynamic programming (or greedy if a local choice is always safe). |
| "Shortest path / fewest steps" in an unweighted graph or grid | BFS. |
| "Top k / k-th largest / running median" | Heap. |
| "Connected / groups / merge sets" | Union-Find or DFS. |
| "Matching brackets / undo / nearest greater" | Stack. |
| Input limit n <= 20 | Exponential is allowed (bitmask, backtracking). |
| Input limit n up to 1e5 or 1e6 | You need near-linear: O(n) or O(n log n). |

## Core Concepts / Template

### The 7-step method

```
1. Clarify      Ask about input, output, limits, edge cases.
2. Examples     Make your own small example. Include a tricky one.
3. Brute force  State the obvious correct solution. Give its complexity.
4. Optimize     Find the wasted work. Name the trick or data structure.
5. Code         Write clean code. Small functions, good names.
6. Test         Trace your own example by hand. Then edge cases.
7. Complexity   State time and space. Mention trade-offs.
```

Let us look at each step.

**1. Clarify.** Never assume. Ask short questions:

- What is the input type and size? Can it be empty? Can it be null?
- Can values be negative, zero, or duplicated?
- Is the input sorted? Is it a stream?
- What do I return if there is no answer: -1, null, empty array?
- If many answers exist, may I return any one?
- Can I change the input, or must I keep it as it is?

Why: a wrong assumption wastes the whole interview. Also, the limits you hear here decide your target complexity (see the table below).

**2. Examples.** Take the given example, then invent one more with a trap in it (duplicates, negatives, one element). Say the expected answer. Doing this by hand often shows you the pattern. It also proves you understood the question.

**3. Brute force.** Always say the brute force first, even if you know it is slow. It shows you can solve the problem, gives you a safe fallback, and tells you what work is wasted. Say its complexity. Often the interviewer says "ok, can you do better?" and that is your cue.

**4. Optimize.** Ask: "Where am I repeating work?" or "What am I searching for again and again?" Typical moves:

- Repeated search -> hash map / set (turns O(n) lookup into O(1)).
- Repeated range sum -> prefix sums.
- Nested loops over sorted data -> two pointers or binary search.
- Repeated sub-problems -> memoization (DP).
- "Best so far" decisions -> greedy or heap.
- Trade space for time: extra memory is the most common way to remove a loop.

**5. Code.** Write after the interviewer agrees with your plan. Use clear names (`left`, `right`, `seen`, not `a`, `b`, `c`). Keep helper logic in small functions. Do not erase and restart; fix in place.

**6. Test.** Do not say "done" and wait. Walk your own example through the code line by line, with real values. Then try the edge cases. This is where you catch off-by-one errors yourself, which looks very good.

**7. Complexity.** Say time and space with the reason ("one pass, each element is touched once, so O(n); the map can hold n entries, so O(n) space"). Mention what could be improved if the limits changed.

### How to think aloud

Silence is the worst thing in an interview. The interviewer cannot help you if they cannot see your thinking. Use short sentences with a purpose:

| Moment | What to say |
|---|---|
| Start | "Let me restate the problem to be sure I understand." |
| Unclear part | "Can the array contain negatives? That changes the approach." |
| Brute force | "The simple way is two loops, O(n^2). It is correct, but slow for n = 1e5." |
| Idea | "I am repeating a search here. If I store what I have seen in a map, I can look it up in O(1)." |
| Before coding | "Does this plan sound okay before I code it?" |
| While coding | "I use `left` and `right` as window edges. Now I move `right`." |
| Bug found | "Wait, this fails when the array is empty. I will add a guard." |
| Stuck | "I am stuck on how to avoid the second loop. Let me try a small example by hand." |

Do not narrate every keystroke. Narrate decisions and reasons.

### Constraints hint the target complexity

A typical judge runs about 1e8 simple operations in a second (a rough rule of thumb, not exact; JavaScript is a bit slower than C++). So look at n and ask: "How many operations can I afford?"

| Input size n | Target time | What it usually means |
|---|---|---|
| n <= 10 | O(n!) or O(n^2 * 2^n) | Try all permutations. |
| n <= 20 | O(2^n) or O(n * 2^n) | Subsets, bitmask, backtracking. |
| n <= 40 | O(2^(n/2)) | Meet in the middle (rare). |
| n <= 100 | O(n^3) | Triple loops, Floyd-Warshall. |
| n <= 1,000 to 5,000 | O(n^2) | Nested loops, 2D DP. |
| n <= 1e5 | O(n log n) or O(n) | Sort, heap, binary search, two pointers, hash map. |
| n <= 1e6 | O(n) or O(n log n) | One pass, sort. |
| n <= 1e7 or more | O(n) with a small constant, or O(log n) / O(1) | Math formula, binary search on the answer. |
| n up to 1e18 | O(log n) | Binary search, fast exponent, math. |

Other hints:

- A value limit matters too. If values are small (say <= 1e5), you can use an array of counts instead of a map (counting sort, frequency array).
- "k is small" (like k <= 20) often means you can afford an extra factor of k or 2^k.
- If the problem asks for the answer modulo 1e9+7, the answer can be huge, which hints at counting / DP.
- If the interviewer gives no limits, ask for them. If they say "assume it is big", aim for O(n) or O(n log n).

Use the table to rule things out. If n = 1e5 and your idea is O(n^2) = 1e10, it will time out. Go back to step 4.

### Edge-case checklist

Run this list in your head before and after coding.

| Area | Check |
|---|---|
| Size | Empty input, one element, two elements. |
| Values | Zero, negatives, all equal, all different, very large (overflow is rare in JS numbers up to 2^53, but sums of big values can lose precision). |
| Order | Already sorted, reverse sorted, random. |
| Duplicates | Repeated values, repeated pairs. Can I use the same element twice? |
| Strings | Empty string, one character, all same character, upper vs lower case, spaces, unicode. |
| Numbers | Integer vs float, division by zero, negative modulo (in JS, `-7 % 3` is `-1`). |
| Linked list / tree | Null head, single node, a list with a cycle, a skewed tree (like a linked list), a very deep tree (stack depth). |
| Graph | Disconnected parts, self-loop, parallel edges, cycles, a single node. |
| Answer | No answer exists. Many answers exist. Answer is at the border (first or last index). |
| Loops | Off-by-one: `<` vs `<=`, `mid` rounding, `right = n - 1` vs `n`. |

### When you are stuck

Being stuck is normal. Interviewers expect it. What matters is how you move. Try these in order:

1. **Re-read the problem.** One word you skipped ("sorted", "distinct", "contiguous") may be the key.
2. **Work a small example by hand.** Do not think about code. Ask "how would I do this on paper?" Then write down the steps you did.
3. **Solve a simpler version.** Remove a condition (no duplicates, k = 1, only positives). Solve that. Then add the condition back.
4. **Write the brute force.** Even a slow answer is progress. Then ask where it repeats work.
5. **Look at the constraints.** The limit tells you the shape of the answer (table above).
6. **Try a known tool.** Sort it? Put it in a hash map? Use two pointers? Use a stack? Go backwards? Draw it as a graph?
7. **Think about the answer first.** "What would the final answer look like? What is the smallest piece of it?" This is how recursion and DP start.
8. **Ask for a hint, politely and specifically.** "I think the problem has repeated work in the inner loop. Is there a structure I should consider?" That is much better than "I do not know."

### How to pick the data structure

Do not start from the structure. Start from the operation your algorithm repeats, then pick the structure that makes that operation cheap.

| The operation I repeat | Data structure | Cost |
|---|---|---|
| "Have I seen this value?" / count of values | Hash set / hash map | O(1) average |
| "What is the smallest / largest right now?" and remove it | Heap (priority queue) | O(log n) |
| "Latest in, first out" (undo, brackets, nearest greater) | Stack | O(1) |
| "First in, first out" (BFS, level order) | Queue | O(1) |
| "Sliding window max / min" | Monotonic deque | O(1) amortized |
| "Sum of a range, many times" | Prefix sum array | O(1) per query |
| "Is x in a sorted collection?" / "first position >= x" | Sorted array + binary search | O(log n) |
| "Are these two items in the same group?" | Union-Find | almost O(1) |
| "Words that share a prefix" | Trie | O(length) |
| "Neighbours of a node" | Adjacency list | O(degree) |
| "Insert / delete / find in the middle, keep order" | Balanced tree (in JS you must build or simulate it) | O(log n) |
| "Cache with order of use" | Map + doubly linked list (LRU) | O(1) |

A quick rule: if you find yourself writing a loop inside a loop just to search, replace the inner loop with a map, a set, a sorted array, or a heap.

## Worked Examples

Three problems solved from scratch with the full method. Watch the thinking, not only the code.

### Worked Example 1: Two Sum (the "remember what you saw" move)

**Step 1, clarify.** Problem: given an array `nums` and a number `target`, return the indices of two different elements that add up to `target`.

Questions I would ask:

- Is there exactly one answer, or could there be none? (Assume exactly one; return `[]` if none.)
- Can I use the same element twice? (No, two different indices.)
- Is the array sorted? (No.) Can values be negative or duplicated? (Yes to both.)
- How large is n? (Say up to 1e4 or 1e5.)

**Step 2, examples.**

- `[2, 7, 11, 15]`, target 9 -> `[0, 1]`.
- Tricky: `[3, 3]`, target 6 -> `[0, 1]`. This tests duplicates. Another: `[3, 2, 4]`, target 6 -> `[1, 2]` (a trap: 3 + 3 would reuse the same element).

**Step 3, brute force.** Try every pair.

```js
function twoSumBrute(nums, target) {
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === target) return [i, j];
    }
  }
  return [];
}
```

Time O(n^2), space O(1). With n = 1e5 that is about 5e9 pair checks. Too slow.

**Step 4, optimize.** Where is the wasted work? For each `nums[i]`, the inner loop searches the rest of the array for `target - nums[i]`. That is a repeated search. A hash map answers "have I seen this value, and where?" in O(1). So: walk the array once; for each number compute what partner I need; if the partner is already in the map, done; otherwise store this number.

Why store after checking? So an element never pairs with itself.

**Step 5, code.**

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

**Step 6, test by hand** on `[3, 2, 4]`, target 6:

| i | nums[i] | need | seen before | action |
|---|---|---|---|---|
| 0 | 3 | 3 | {} | not found; seen = {3:0} |
| 1 | 2 | 4 | {3:0} | not found; seen = {3:0, 2:1} |
| 2 | 4 | 2 | {3:0, 2:1} | found 2 at index 1 -> return [1, 2] |

The trap case works, because the 3 was stored after the check. Edge cases: `[3, 3]` target 6: at i=1, need = 3 and the map has 3 at index 0, so the answer is `[0, 1]`. Empty array returns `[]`. A single element returns `[]`.

**Step 7, complexity.** Time O(n), one pass with O(1) map operations on average. Space O(n) for the map. We traded memory for speed. If the interviewer says "no extra space", the follow-up is: sort a copy with indices and use two pointers, which costs O(n log n) time.

### Worked Example 2: Longest Substring Without Repeating Characters (the "window" move)

**Step 1, clarify.** Given a string `s`, return the length of the longest substring (contiguous!) with no repeated character.

Questions: Is it a substring (contiguous) or subsequence? (Substring.) Which characters: letters only, or any ASCII? (Any.) Empty string? (Return 0.) Case sensitive? (Yes.)

**Step 2, examples.**

- `"abcabcbb"` -> 3 (`"abc"`).
- `"bbbbb"` -> 1.
- `"pwwkew"` -> 3 (`"wke"`; note `"pwke"` is a subsequence, not a substring).
- Tricky: `"abba"` -> 2. Remember this one: it breaks a common wrong version.

**Step 3, brute force.** For every start index `i`, extend `j` while no repeat, using a set. Stop at the first repeat.

```js
function brute(s) {
  let best = 0;
  for (let i = 0; i < s.length; i++) {
    const set = new Set();
    for (let j = i; j < s.length; j++) {
      if (set.has(s[j])) break;
      set.add(s[j]);
      best = Math.max(best, j - i + 1);
    }
  }
  return best;
}
```

Time O(n^2) (worst case with a small alphabet it is O(n * alphabet), but in general say O(n^2)), space O(min(n, alphabet)).

**Step 4, optimize.** The wasted work: when starting at `i + 1` we rebuild a window we mostly already built. Idea: keep a window `[left, right]` that is always valid (no repeats). Move `right` forward one step at a time. If `s[right]` already sits inside the window, move `left` just past that old copy. Remember the last index of every character so we can jump `left` in one move.

Careful point (the `"abba"` trap): the stored last index may be outside the current window (to the left of `left`). Then it must be ignored. So the rule is: `left = max(left, last[c] + 1)`.

**Step 5, code.**

```js
function lengthOfLongestSubstring(s) {
  const last = new Map(); // char -> most recent index
  let left = 0;
  let best = 0;
  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    // only react if the old copy is inside the window
    if (last.has(c) && last.get(c) >= left) left = last.get(c) + 1;
    last.set(c, right);
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Step 6, test by hand** on `"abba"`:

| right | c | last[c] | left after | window | best |
|---|---|---|---|---|---|
| 0 | a | none | 0 | "a" | 1 |
| 1 | b | none | 0 | "ab" | 2 |
| 2 | b | 1 (>= 0) | 2 | "b" | 2 |
| 3 | a | 0 (< 2, outside window, ignore) | 2 | "ba" | 2 |

Answer 2. If we had written `left = last[c] + 1` without the check, at right=3 `left` would jump back to 1, and the window `"bba"` would be wrong. Edge cases: `""` returns 0; one character returns 1; all-distinct string returns n.

**Step 7, complexity.** Time O(n): `right` moves n times, `left` never moves backward. Space O(min(n, alphabet size)) for the map. This is the sliding window pattern; see [Sliding Window](../../01-core-linear/06-sliding-window/README.md).

### Worked Example 3: Subarray Sum Equals K (what to do when a pattern fails)

This one shows the "stuck" recovery in action.

**Step 1, clarify.** Given an integer array `nums` and an integer `k`, return the number of contiguous subarrays whose sum equals `k`.

Questions: Can there be negatives? (Yes. This is the key question.) Zeros? (Yes.) Count all, even overlapping ones? (Yes.) n? (Up to about 2e4 on the usual version, but treat it as 1e5 to be safe.)

**Step 2, examples.**

- `[1, 1, 1]`, k = 2 -> 2 (indices 0..1 and 1..2).
- `[1, 2, 3]`, k = 3 -> 2 (`[1,2]` and `[3]`).
- Tricky: `[1, -1, 0]`, k = 0 -> 3 (`[1,-1]`, `[1,-1,0]`, `[0]`).

**Step 3, brute force.** For each start `i`, keep a running sum going right; count whenever it equals `k`.

```js
function subarraySumBrute(nums, k) {
  let count = 0;
  for (let i = 0; i < nums.length; i++) {
    let sum = 0;
    for (let j = i; j < nums.length; j++) {
      sum += nums[j];
      if (sum === k) count++;
    }
  }
  return count;
}
```

Time O(n^2), space O(1). Already better than checking each subarray with a third loop (O(n^3)).

**Step 4, optimize. First attempt fails.** "Contiguous subarray" smells like a sliding window: grow right, shrink left when the sum is too big. But a shrinking window only works if adding an element always increases the sum. With negatives, a bigger window can have a smaller sum, so we cannot decide when to shrink. The window idea is dead. This is the moment to say aloud: "Sliding window needs non-negative numbers. Not valid here."

**Recover.** Use the "stuck" list: look for a known tool for range sums, which is prefix sums. Let `P[i]` be the sum of the first `i` elements. The sum of `nums[a..b]` is `P[b+1] - P[a]`. We want that to be `k`, so for each ending point we want to count earlier prefixes equal to `P_now - k`. "Count earlier values equal to X" is exactly what a hash map does.

So: walk once, keep the running sum, keep a map `prefix value -> how many times seen`. Seed the map with `{0: 1}` so that a subarray starting at index 0 is counted (the empty prefix).

**Step 5, code.**

```js
function subarraySum(nums, k) {
  const count = new Map([[0, 1]]); // prefix sum -> times seen (empty prefix = 0)
  let sum = 0;
  let ans = 0;
  for (const x of nums) {
    sum += x;
    ans += count.get(sum - k) || 0; // earlier prefixes that make a subarray ending here
    count.set(sum, (count.get(sum) || 0) + 1);
  }
  return ans;
}
```

**Step 6, test by hand** on `[1, -1, 0]`, k = 0:

| x | sum | looking for sum - k | found | ans | map after |
|---|---|---|---|---|---|
| start | 0 | - | - | 0 | {0:1} |
| 1 | 1 | 1 | 0 | 0 | {0:1, 1:1} |
| -1 | 0 | 0 | 1 | 1 | {0:2, 1:1} |
| 0 | 0 | 0 | 2 | 3 | {0:3, 1:1} |

Answer 3, matches. The order matters: look up first, then store the current sum. If we stored first and `k = 0`, we would count an empty subarray. Edge cases: empty array gives 0; `k` larger than any possible sum gives 0.

**Step 7, complexity.** Time O(n) average, space O(n). The lesson is bigger than the problem: when one pattern fails because of a hidden condition (negatives), say why it fails, then change the tool, not the problem.

## Interview Questions and Answers

**Q1. Why should I state the brute force even if it is obviously too slow?**
It proves you understand the problem, gives a correct fallback if you run out of time, and shows where the repeated work is. Interviewers often use it as the starting point for the hint "can you do better?"

**Q2. n = 2 * 10^5. What complexity do I aim for, and what does that rule out?**
Aim for O(n log n) or O(n). O(n^2) would be about 4 * 10^10 operations, which is far too slow. So I would think of sorting, a heap, binary search, two pointers, a hash map, or a single pass.

**Q3. You cannot find the optimal solution. What do you do?**
Do not go silent. Write the correct brute force, say its complexity, then look for the repeated work, using the stuck checklist (small example, simpler version, constraints, known tool). If I still cannot find it, I ask for a specific hint. A working slow solution is worth more than no solution.

**Q4. When is a hash map the wrong choice?**
When I need ordered data (smallest, largest, range queries): use sorting, a heap, or a tree. When memory is tight: use sorting plus two pointers. When keys are small integers: a plain array is faster and simpler. Also, the hash map's O(1) is average case; the worst case can be worse.

**Q5. How do I test code without running it, in an interview?**
Pick a small example and trace the variables line by line in a table, as in the walk-throughs above. Then trace an empty input, a single element, and one tricky case. State the result of each trace out loud.

## Common Mistakes

- **"Coding fast shows I am good."** Wrong. Coding before the plan is agreed is the top reason for restarts. Spend the first 5 to 10 minutes on steps 1 to 4.
- **"I should skip the brute force to save time."** The brute force takes one minute and gives you a safe answer. Skipping it leaves you with nothing if the clever idea does not come.
- **"Edge cases are for the end."** Ask about them in step 1, then check them again in step 6. Empty input and one element crash many solutions.
- **"The constraints are just decoration."** They are the biggest hint you get. Ignoring n = 1e5 and writing O(n^2) means a time-out.
- **"If my pattern is almost right, force it."** Sliding window with negatives is the classic example. If a hidden condition breaks the pattern, change the tool.
- **"Silent thinking looks smart."** It looks stuck. Narrate decisions.
- **"I wrote the code, so I am done."** Not until you trace it. Most off-by-one bugs show up in a 2-minute dry run.
- **Mutating the input without asking.** Sorting the original array may break an answer that needs original indices. Ask or copy first.

## Interview Tips

- Open with: "Let me restate the problem and ask a few questions." It buys time and shows maturity.
- Always give the brute force and its complexity first, then say "the repeated work is ... so I can use ...".
- Get a "yes, that plan sounds fine" before you type code.
- Use meaningful names and say them out loud. Avoid clever one-liners that you cannot explain.
- When you find a bug, say it calmly and fix it. Finding your own bug is a plus.
- Say time and space at the end without being asked. Add "the trade-off is extra memory for speed."
- Common follow-ups: "Can you do it in O(1) extra space?", "What if the input does not fit in memory?", "What if the data is a stream?", "What if there are duplicates / negatives?", "How would you test this?"
- Edge cases to mention every time: empty, one element, duplicates, negatives, already sorted, no answer, very large input.
- If the interviewer gives a hint, take it, say thanks, and use it. Do not defend your old idea.
- Time yourself in practice: about 5 min to clarify and plan, 15 to 20 min to code, 5 min to test, in a 45-minute round.

## Practice List

Use the 7 steps on each. Write down the brute force and the constraint-based target before you code.

**Easy:** Two Sum, Contains Duplicate, Valid Anagram, Best Time to Buy and Sell Stock, Valid Parentheses, Maximum Subarray, Move Zeroes.

**Medium:** Longest Substring Without Repeating Characters, Subarray Sum Equals K, 3Sum, Group Anagrams, Top K Frequent Elements, Product of Array Except Self, Container With Most Water, Longest Consecutive Sequence.

**Hard:** Trapping Rain Water, Minimum Window Substring, Sliding Window Maximum, Median of Two Sorted Arrays, First Missing Positive.

## Key Takeaways

- **Follow the 7 steps in order:** clarify, examples, brute force, optimize, code, test, complexity.
- **Always state the brute force first.** It is your safe answer and it shows where the repeated work is.
- **Read the constraints:** n up to 1e5 means O(n log n) or better; n up to 20 allows exponential.
- **Think aloud with decisions and reasons,** not silence and not every keystroke.
- **Pick the data structure from the operation you repeat** (lookup, min/max, last-in, range sum), not from habit.
- **When stuck:** re-read, small example by hand, simpler version, brute force, known tool, then ask for a specific hint.
- **Test your own code by tracing it** with a normal case and an edge case before you say "done".

---

[Next: Complexity Analysis →](../02-complexity-analysis/README.md) | [Back to Foundations Index](../README.md)
