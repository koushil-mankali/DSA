# Mock Interview Checklist

> Use this when you practise with a friend, a timer or a mirror. A real coding round is 45 minutes; plan for one medium problem or two easy ones.

---

## Before You Start

- [ ] Timer set for 45 minutes. No notes, no autocomplete.
- [ ] Pick a problem you have not seen. Use the [Practice Lists](./README.md) at the end of each chapter.
- [ ] Say out loud that you will think aloud.

## The 45-Minute Shape

| Minutes | What you do | What to say |
|---|---|---|
| 0-5 | Clarify | Restate the problem. Ask about input size, duplicates, negatives, sortedness, empty input and the return when nothing is found. |
| 5-10 | Examples | Work one normal example and one edge case by hand. |
| 10-15 | Brute force | State the simple solution and its complexity, even if you will not code it. |
| 15-20 | Optimise | Name the bottleneck. Say which pattern fits, using the [Pattern Cheatsheet](./01-pattern-cheatsheet.md). Agree on the plan before coding. |
| 20-35 | Code | Write clean code with meaningful names. Keep talking. |
| 35-42 | Test | Dry-run your example line by line. Then the edge cases. |
| 42-45 | Wrap up | State time and space complexity. Offer one improvement. |

## While Coding

- [ ] Handle empty input first.
- [ ] Use small helper functions for repeated work.
- [ ] Do not silently change the plan. If you change it, say why.
- [ ] If you are stuck for 3 minutes, say what you tried and ask for a hint. Interviewers prefer that to silence.

## Edge-Case Checklist

- [ ] Empty array / string / tree / graph
- [ ] One element
- [ ] All elements equal
- [ ] Already sorted / reverse sorted
- [ ] Negative numbers and zeros
- [ ] Very large values (overflow: `Number.MAX_SAFE_INTEGER`, bit operations are 32-bit in JS)
- [ ] Duplicates
- [ ] Disconnected graph, cycles, self loops
- [ ] Off-by-one at the boundaries

## Common Follow-Up Questions

| Follow-up | Where to look |
|---|---|
| "Can you do it in O(1) extra space?" | [Two Pointers](../01-core-linear/05-two-pointers/README.md), [Arrays](../01-core-linear/01-arrays/README.md) |
| "What if the input is a stream?" | [Heaps](../02-intermediate/05-heaps/README.md), [Randomization](../02-intermediate/09-randomization-and-sampling/README.md) |
| "What if it does not fit in memory?" | [Sorting](../01-core-linear/02-sorting/README.md), [Data-Structure Internals](../06-advanced/06-data-structure-internals/README.md) |
| "Now make it iterative." | [Tree Traversals](../03-trees/01-binary-tree-traversals/README.md), [Recursion](../02-intermediate/01-recursion/README.md) |
| "What if there are many queries?" | [Prefix Sum](../01-core-linear/07-prefix-sum/README.md), [Segment Tree and Fenwick](../06-advanced/02-segment-tree-and-fenwick/README.md) |
| "Can you reduce DP space?" | [DP Intuition](../05-dynamic-programming/01-dp-intuition/README.md) |
| "Is greedy correct here?" | [Greedy](../02-intermediate/04-greedy/README.md) |

## After the Mock

Write down, within 10 minutes:

1. Where you lost time (clarifying, brute force, coding, debugging).
2. The pattern you missed, and the chapter that teaches it.
3. One bug you made and how you would catch it next time.
4. One thing you did well, so you repeat it.

Redo the problem from scratch 3 days later without looking.

---

[← Previous: Four-Week Plan](./03-four-week-plan.md) | [Back to Interview Playbook](./README.md)
