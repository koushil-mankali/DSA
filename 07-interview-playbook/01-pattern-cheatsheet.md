# Pattern Cheatsheet: "See X, Think Y"

> Read this the night before an interview. Each row maps a clue in the problem statement to the technique and the chapter that teaches it.

---

## 1. Clues in the Input

| If the input / statement looks like... | Think | Chapter |
|---|---|---|
| Sorted array, find a pair / triplet | Two pointers | [Two Pointers](../01-core-linear/05-two-pointers/README.md) |
| Contiguous subarray / substring with a condition | Sliding window | [Sliding Window](../01-core-linear/06-sliding-window/README.md) |
| "Number of subarrays with sum K", negatives allowed | Prefix sum + hash map | [Prefix Sum](../01-core-linear/07-prefix-sum/README.md) |
| Sorted array, or "minimise the maximum" / "smallest X such that..." | Binary search (on index or on the answer) | [Binary Search](../01-core-linear/08-binary-search/README.md) |
| "Next greater / smaller element", histogram, temperatures | Monotonic stack | [Monotonic Stack](../01-core-linear/11-monotonic-stack-queue/README.md) |
| Matching brackets, undo, nested structure | Stack | [Stack and Queue](../01-core-linear/10-stack-and-queue/README.md) |
| Frequency, duplicates, "have I seen this?" | Hash map / set | [Hashing](../01-core-linear/03-hashing/README.md) |
| Linked list: middle, cycle, k-th from end | Fast/slow pointers, dummy node | [Linked List](../01-core-linear/09-linked-list/README.md) |
| Top K, K-th largest, running median, merge K lists | Heap | [Heaps](../02-intermediate/05-heaps/README.md) |
| Intervals: merge, overlap, rooms, schedule | Sort by start + sweep line | [Intervals](../02-intermediate/06-intervals-and-sweep-line/README.md) |
| "Single number", parity, subsets of a small set, XOR | Bit manipulation | [Bit Manipulation](../02-intermediate/07-bit-manipulation/README.md) |
| Grid / matrix traversal | BFS/DFS on the grid; DP over cells | [Grid and Islands](../04-graphs/02-grid-and-islands/README.md), [Matrix](../02-intermediate/08-matrix/README.md) |
| Prefix lookups, autocomplete, word dictionary | Trie | [Trie](../03-trees/05-trie/README.md) |
| Random pick, shuffle, stream sampling | Reservoir sampling, Fisher-Yates | [Randomization](../02-intermediate/09-randomization-and-sampling/README.md) |

## 2. Clues in the Question

| If the question says... | Think | Chapter |
|---|---|---|
| "All subsets / permutations / combinations", "generate all" | Backtracking | [Backtracking](../02-intermediate/02-backtracking/README.md) |
| "Maximum / minimum / count the ways", choices overlap | Dynamic programming | [DP Intuition](../05-dynamic-programming/01-dp-intuition/README.md) |
| "Minimum number of operations / fewest steps" in an unweighted setting | BFS | [Graph Basics](../04-graphs/01-graph-basics-bfs-dfs/README.md) |
| "Shortest path", weighted edges | Dijkstra (non-negative), Bellman-Ford (negative) | [Shortest Paths](../04-graphs/05-shortest-paths/README.md) |
| "Order of tasks with dependencies", "can all courses be finished" | Topological sort | [Topological Sort](../04-graphs/04-topological-sort/README.md) |
| "Are X and Y connected", "merge groups", "number of components" with merges | Union-Find | [Union-Find](../04-graphs/06-union-find/README.md) |
| "Connect everything at minimum cost" | MST | [MST](../04-graphs/07-minimum-spanning-tree/README.md) |
| Locally best choice looks safe (intervals, coins, jumps) | Greedy, but prove it | [Greedy](../02-intermediate/04-greedy/README.md) |
| "Lowest common ancestor", "path sum", "diameter" | Tree DFS returning a value | [Paths, LCA and Properties](../03-trees/03-paths-lca-and-tree-properties/README.md) |
| "Validate / search / k-th smallest" in a BST | Inorder = sorted | [BSTs](../03-trees/04-binary-search-trees/README.md) |
| Range sum / min with updates | Fenwick or segment tree | [Segment Tree and Fenwick](../06-advanced/02-segment-tree-and-fenwick/README.md) |
| Pattern search in a string, repeated prefix | KMP / Z / rolling hash | [String Algorithms](../06-advanced/01-string-algorithms/README.md) |
| "Design a ... with O(1) get/put" | Hash map + linked list / heap | [Design Problems](../06-advanced/05-design-problems/README.md) |
| Two players, optimal play | Minimax DP, parity | [Game Theory](../06-advanced/07-game-theory-and-puzzles/README.md) |
| Answers modulo 1e9+7, nCr | Modular arithmetic, BigInt | [Number Theory](../06-advanced/04-number-theory/README.md) |

## 3. Constraint to Complexity

| n up to | Target | Typical technique |
|---|---|---|
| 10-12 | O(n!) | Permutations, backtracking |
| 20-25 | O(2^n) | Subsets, bitmask DP |
| 500 | O(n^3) | Floyd-Warshall, interval DP |
| 5,000 | O(n^2) | 2D DP, pairs |
| 1e5 - 1e6 | O(n log n) or O(n) | Sort, heap, binary search, two pointers, hashing |
| 1e9 or more | O(log n) or O(1) | Math, binary search on answer |

These are rules of thumb for typical judge time limits (about 1e8 simple operations). Always read the real constraints.

## 4. DP Quick Map

| Shape of state | Family | Chapter |
|---|---|---|
| dp[i] depends on earlier i | 1D DP | [1D DP](../05-dynamic-programming/02-one-d-dp/README.md) |
| dp[i][j] on a grid | Grid DP | [Grid DP](../05-dynamic-programming/03-grid-dp/README.md) |
| take / not take against a target | Subset sum | [Subset DP](../05-dynamic-programming/04-subset-and-subsequence-dp/README.md) |
| items with weights and values | Knapsack | [Knapsack](../05-dynamic-programming/05-knapsack/README.md) |
| two strings | LCS / edit distance | [String DP](../05-dynamic-programming/06-string-dp/README.md) |
| buy / sell with states | Stocks | [Stocks](../05-dynamic-programming/07-stocks/README.md) |
| increasing subsequence | LIS | [LIS](../05-dynamic-programming/08-longest-increasing-subsequence/README.md) |
| split a range at k | Interval / MCM | [Partition and MCM](../05-dynamic-programming/09-partition-and-mcm-dp/README.md) |
| n <= 20, visited set | Bitmask | [Bitmask DP](../05-dynamic-programming/10-bitmask-dp/README.md) |
| tree or digits | Tree DP / digit DP | [Tree and Digit DP](../05-dynamic-programming/11-tree-and-digit-dp/README.md) |

## 5. Five Questions Before Coding

1. Can I brute force it, and what does that cost?
2. What is repeated work, and can a hash map, prefix array or memo remove it?
3. Is there order I can exploit (sorting, monotonic, BST)?
4. Does the answer have a yes/no test I can binary search on?
5. What are the edge cases: empty, one element, duplicates, negatives, overflow?

---

[← Back to Interview Playbook](./README.md) | [Next: Company Index →](./02-company-index.md)
