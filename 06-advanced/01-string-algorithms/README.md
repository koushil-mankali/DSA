# String Algorithms: KMP, Z, Rabin-Karp, Manacher

> **Estimated Time:** 5 hr | **Difficulty:** Advanced
> **Prerequisites:** [04 - Strings](../../01-core-linear/04-strings/README.md), [03 - Hashing](../../01-core-linear/03-hashing/README.md), [08 - Binary Search](../../01-core-linear/08-binary-search/README.md), [06 - String DP](../../05-dynamic-programming/06-string-dp/README.md)

---

## What You'll Learn

- You can build the KMP prefix function (LPS array) and explain why the search never re-compares a text character.
- You can compute the Z-array and use it for pattern search and prefix questions.
- You can write a Rabin-Karp rolling hash in JavaScript without overflow, and you know when hashes lie.
- You can run Manacher to find every palindrome in a string in O(n).
- You can recognise which tool fits which problem: pattern search, borders, repeats, palindromes, or "longest repeated thing".

## The Big Idea

Think of looking for a word in a long book. The slow way: put your finger on page 1, compare letter by letter, and when a letter fails, move your finger one letter forward and start the word over. You re-read the same letters again and again.

A smart reader remembers: "I already matched `abab` here. The word starts with `ab`, so the last two letters I read are already a start of a new try. I do not need to look at them again." That memory is the whole idea of KMP and Z. Both compute, **once**, how much of the pattern overlaps with itself. Then they use it to skip work.

Rabin-Karp takes a different road. Turn each window of text into a single number (a hash). Compare numbers instead of strings. Slide the window by one letter and update the number in O(1).

Manacher uses the mirror trick. Inside a big palindrome, the left half is a mirror of the right half. So what you learned on the left tells you about the right, and you skip most comparisons.

Why interviewers love it: `indexOf` is O(n*m) in the worst case on paper. A candidate who can say "I will precompute the border array and get O(n+m)" shows real depth. Many "hard" string problems (shortest palindrome, longest happy prefix, repeated pattern) are one trick away once you know the prefix function.

Vocabulary used in this chapter:

- **Prefix**: the start of a string (`ab` in `abc`). **Suffix**: the end (`bc`).
- **Border**: a string that is both a proper prefix and a proper suffix. `abcab` has border `ab`.
- **Proper**: shorter than the whole string.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| Find a pattern in text, all positions, count occurrences | KMP or Z on `pattern + '#' + text` |
| Longest prefix that is also a suffix, "happy prefix", "border" | Prefix function, answer is `lps[n-1]` |
| String is built by repeating a smaller block | Prefix function: `p = n - lps[n-1]`, check `n % p === 0` |
| Is B a rotation of A | Search B inside `A + A` |
| Make a palindrome by adding chars at the front | Longest palindromic prefix via KMP/Z on `s + '#' + reverse(s)` |
| Longest palindromic substring, count palindromic substrings, n up to 1e5 | Manacher (expand-around-center is O(n^2)) |
| Longest repeated substring, "appears at least twice" | Binary search on length + rolling hash |
| Compare many substrings for equality quickly | Prefix hashes, O(1) per comparison |
| Anagram windows, "same letters in a window" | Counts, or an order-free hash (sum of random values) |
| Many patterns at once, or all suffixes | Mention Aho-Corasick, suffix array/automaton (not coded here) |

## Core Concepts / Template

### 1. KMP and the prefix function (LPS)

`lps[i]` = length of the longest **proper** prefix of `s[0..i]` that is also a suffix of `s[0..i]`.

```
s    = a a b a a a b
lps  = 0 1 0 1 2 2 3
```

Read `lps[4] = 2`: the text `aabaa` starts with `aa` and ends with `aa`.

**Building it.** Keep `k` = current border length. For each new char `s[i]`:

1. While `k > 0` and `s[i] !== s[k]`, fall back: `k = lps[k-1]`. We try the next shorter border.
2. If `s[i] === s[k]`, grow the border: `k++`.
3. Save `lps[i] = k`.

```js
function prefixFunction(s) {
  const lps = new Array(s.length).fill(0);
  let k = 0;
  for (let i = 1; i < s.length; i++) {
    while (k > 0 && s[i] !== s[k]) k = lps[k - 1];
    if (s[i] === s[k]) k++;
    lps[i] = k;
  }
  return lps;
}
```

**Why it is O(n).** `k` goes up by at most 1 per step, so it goes up at most n times in total. Each fall-back lowers `k`. `k` never goes below 0. So total fall-backs are at most n. The `while` loop looks scary but costs O(n) in total (amortized).

**Why the search never re-compares.** To search, run the same loop over the text, using the pattern's `lps`. `k` = how many pattern chars match the text ending at `i`. On a mismatch we do not move back in the text. We only shrink `k` to the next border: the longest shorter prefix of the pattern that must still match, because it is a suffix of what we already matched. The text pointer `i` only moves forward. So the text is read once: O(n + m).

Trace: search `aab` in `aaab`.

| i | text[i] | k before | action | k after |
|---|---|---|---|---|
| 0 | a | 0 | match | 1 |
| 1 | a | 1 | match | 2 |
| 2 | a | 2 | `a` vs `b` mismatch, k = lps[1] = 1, then `a` matches | 2 |
| 3 | b | 2 | match | 3 = m, found at 3-3+1 = 1 |

At i = 2 we did not go back to index 1 in the text. The two `a`'s we had read were reused.

### 2. Z-algorithm

`z[i]` = length of the longest substring starting at `i` that is also a prefix of `s`. By convention `z[0] = n`.

```
s = a a b x a a b
z = 7 1 0 0 3 1 0
```

Keep a window `[l, r)` = the rightmost prefix-match found so far. If `i` is inside it, `s[i..r)` is a copy of `s[i-l..r-l)`, so we start from `min(r - i, z[i - l])` and only extend by comparing. The window `r` never moves left, so total work is O(n).

```js
function zFunction(s) {
  const n = s.length, z = new Array(n).fill(0);
  let l = 0, r = 0;
  for (let i = 1; i < n; i++) {
    if (i < r) z[i] = Math.min(r - i, z[i - l]);
    while (i + z[i] < n && s[z[i]] === s[i + z[i]]) z[i]++;
    if (i + z[i] > r) { l = i; r = i + z[i]; }
  }
  if (n) z[0] = n;
  return z;
}
```

**Pattern search with Z:** build `z` of `pattern + '#' + text` (use a separator that is not in either string). Every index with `z[i] === pattern.length` is a match. The separator stops a match from running across the border.

**KMP or Z?** Both are O(n). Pick the one you can write without bugs. Rule of thumb: borders and "prefix = suffix" questions feel natural in KMP. "How far does the prefix match at each position" feels natural in Z. In an interview, learn one cold and recognise the other.

### 3. Rabin-Karp rolling hash

Treat a window as a number in base `B`: `hash("abc") = a*B^2 + b*B + c`, all taken modulo a prime `M`. Slide by one char:

```
new = (old - out * B^(m-1)) * B + in      (mod M)
```

If two hashes differ, strings differ for sure. If hashes are equal, strings are **probably** equal. Two different strings with the same hash is a **collision**.

**JavaScript and overflow.** JS numbers are doubles. Integers are exact only up to 2^53 (about 9e15). A product like `(1e9+7) * (1e9+7)` is about 1e18 and silently loses digits. Options:

- **Keep products under 2^53.** If the modulus is about 1e9 (under 2^30) and the base is under 2^20, then `hash * B` is under 2^50. Safe, no BigInt needed. Add a small char code and take `% M`. This is what we do below.
- **Use `mulmod`** (split one factor in two 16-bit halves) when both factors can be about 1e9, for example prefix-hash range queries `H[l] * P[len]`.
- **BigInt** is always safe but 10 to 50 times slower. Fine for a quick check, bad for n = 1e5 loops.
- **Never use `Math.pow` for big powers.** Build powers by repeated multiply-and-mod.

**Reducing collisions.**
- Use **two** moduli (for example 1e9+7 and 998244353). Treat the pair as one hash. Collision chance drops to about 1e-18 per comparison.
- Use a **random base** at startup so nobody can craft an anti-hash input.
- On a hash match, **verify** with a real string compare when it is cheap (single pattern). In the worst case you pay O(m) per false alarm, but false alarms are very rare.
- Do not use a base smaller than the alphabet size. Do not use `0` for any char value (so `a` and `aa` differ): use `charCode` or `ch - 96`.

```js
const M1 = 1000000007, M2 = 998244353;
const B = 257 + Math.floor(Math.random() * 1000000); // < 2^20, so h*B < 2^50

function rabinKarp(text, pat) {
  const n = text.length, m = pat.length, res = [];
  if (m === 0 || m > n) return res;
  let p1 = 1, p2 = 1, hp1 = 0, hp2 = 0, h1 = 0, h2 = 0; // p = B^(m-1)
  for (let i = 0; i < m; i++) {
    if (i < m - 1) { p1 = (p1 * B) % M1; p2 = (p2 * B) % M2; }
    hp1 = (hp1 * B + pat.charCodeAt(i)) % M1; hp2 = (hp2 * B + pat.charCodeAt(i)) % M2;
    h1 = (h1 * B + text.charCodeAt(i)) % M1; h2 = (h2 * B + text.charCodeAt(i)) % M2;
  }
  for (let i = 0; ; i++) {
    if (h1 === hp1 && h2 === hp2 && text.startsWith(pat, i)) res.push(i); // verify: hash can lie
    if (i + m >= n) break;
    const out = text.charCodeAt(i), inn = text.charCodeAt(i + m);
    h1 = ((h1 - out * p1 % M1 + M1) * B + inn) % M1; // + M1 keeps it non-negative
    h2 = ((h2 - out * p2 % M2 + M2) * B + inn) % M2;
  }
  return res;
}
```

Expected time O(n + m). Worst case O(n*m) only if collisions are many, which a double hash with a random base makes practically impossible.

### 4. Manacher's algorithm

Expand-around-center costs O(n^2) on `aaaa...a`. Manacher keeps the rightmost palindrome `[l, r]` found so far. For a new center `i` inside it, the mirror position `l + r - i` already has a known radius. We copy it (capped at the edge `r`) and only expand beyond.

Two arrays, so we do not need to insert `#` between characters:

- `d1[i]` = number of odd palindromes centered at `i` (also the radius counting the center). Longest is `2*d1[i] - 1`.
- `d2[i]` = number of even palindromes centered **between** `i-1` and `i`. Longest is `2*d2[i]`.

```js
function manacher(s) {
  const n = s.length, d1 = new Array(n).fill(0), d2 = new Array(n).fill(0);
  for (let i = 0, l = 0, r = -1; i < n; i++) {
    let k = i > r ? 1 : Math.min(d1[l + r - i], r - i + 1);
    while (i - k >= 0 && i + k < n && s[i - k] === s[i + k]) k++;
    d1[i] = k;
    if (i + k - 1 > r) { l = i - k + 1; r = i + k - 1; }
  }
  for (let i = 0, l = 0, r = -1; i < n; i++) {
    let k = i > r ? 0 : Math.min(d2[l + r - i + 1], r - i + 1);
    while (i - k - 1 >= 0 && i + k < n && s[i - k - 1] === s[i + k]) k++;
    d2[i] = k;
    if (i + k - 1 > r) { l = i - k; r = i + k - 1; }
  }
  return { d1, d2 };
}
```

Example, `s = "abaab"`:

```
index : 0 1 2 3 4
char  : a b a a b
d1    : 1 2 1 1 1     d1[1]=2 -> "b" and "aba"
d2    : 0 0 0 2 0     d2[3]=2 -> "aa" and "baab"
```

Why O(n): `r` only moves right. Every successful comparison in the `while` loop pushes `r` right, and copying from the mirror costs O(1). Total comparisons are O(n).

The number of palindromic substrings is `sum(d1) + sum(d2)`, because `d1[i]` and `d2[i]` count how many palindromes each center produces.

### 5. Brief mentions (not needed for most interviews)

- **Suffix array**: all suffixes of a string sorted. With an LCP array it answers "longest repeated substring", "count distinct substrings", and "k-th substring". Building takes O(n log n) with the doubling trick. Needs more code than an interview allows, so know what it does.
- **Suffix automaton**: the smallest DFA accepting all suffixes, built in O(n). Counts distinct substrings and does substring queries fast. Competitive programming territory.
- **Aho-Corasick**: a trie with KMP-style fail links, to search many patterns at once. A natural next step after [Trie](../../03-trees/05-trie/README.md) and KMP.

If asked "what else could solve this", name them in one sentence and explain why you did not use them.

## Problems (easy -> hard)

### 1. Find the Index of the First Occurrence in a String (strStr) — Easy
**Companies:** Amazon, Google, Microsoft, Meta
**Problem:** Return the first index where `needle` appears in `haystack`, or -1. Example: `"sadbutsad"`, `"sad"` -> `0`. An empty needle returns `0`.
**Intuition:** Compare letter by letter, but on a mismatch do not restart. Use the pattern's border table to jump to the longest prefix that is still alive.
**Approach 1 - Brute force:** Try every start index and compare up to m chars. Time O(n*m), Space O(1).
**Approach 2 - Better:** Rabin-Karp: compare window hashes, verify on match. Expected Time O(n+m), Space O(1).
**Approach 3 - Optimal:** KMP. Build `lps` of the needle. Walk the haystack once with `k` = matched length. When `k === m`, the match ends at `i`, so it starts at `i - m + 1`.

Trace for `needle = "aab"` on `"aaab"` is the table in Core Concepts: found at index 1.

```js
function prefixFunction(s) {
  const lps = new Array(s.length).fill(0);
  let k = 0;
  for (let i = 1; i < s.length; i++) {
    while (k > 0 && s[i] !== s[k]) k = lps[k - 1];
    if (s[i] === s[k]) k++;
    lps[i] = k;
  }
  return lps;
}

function strStr(hay, needle) {
  if (needle.length === 0) return 0;
  const lps = prefixFunction(needle);
  let k = 0;
  for (let i = 0; i < hay.length; i++) {
    while (k > 0 && hay[i] !== needle[k]) k = lps[k - 1];
    if (hay[i] === needle[k]) k++;
    if (k === needle.length) return i - k + 1; // first match, stop
  }
  return -1;
}
```

**Complexity:** Time O(n + m), Space O(m).
**Watch out:** The empty needle case. To find all matches (not just the first), after a hit set `k = lps[k-1]` instead of returning, so overlapping matches are found. In a real interview, also say that `haystack.indexOf(needle)` exists, and that you are showing the algorithm because the question is about it.

### 2. Rotate String — Easy
**Companies:** Amazon, Google
**Problem:** Can `s` become `goal` by moving some leading chars to the end, any number of times? Example: `"abcde"`, `"cdeab"` -> `true`.
**Intuition:** Write `s` twice: `abcdeabcde`. Every rotation of `s` is a window of length n inside it. So the question is just "does `goal` appear in `s + s`".
**Approach 1 - Brute force:** Try all n rotations and compare. Time O(n^2), Space O(n).
**Approach 2 - Better:** `(s + s).includes(goal)` after a length check. Simple. Cost depends on the engine's search.
**Approach 3 - Optimal:** Same idea with KMP so the worst case is a guaranteed O(n).

```js
// uses strStr from problem 1
const rotateString = (s, goal) => s.length === goal.length && strStr(s + s, goal) !== -1;
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** The length check is required. Without it `s = "ab"`, `goal = "abab"` would pass (`goal` sits inside `"abab"`) even though lengths differ. Both strings empty returns true, which is correct.

### 3. Repeated Substring Pattern — Easy
**Companies:** Amazon, Google, Microsoft
**Problem:** Can `s` be made by repeating a shorter substring two or more times? `"abab"` -> true, `"aba"` -> false.
**Intuition:** If `s` is `block` repeated, then removing the first block leaves a string that is also a prefix of `s`. That means `s` has a border of length `n - blockLength`. The prefix function gives the longest border `L = lps[n-1]`, so the smallest possible block is `p = n - L`. It is a real repeat only if `p` divides `n` and `L > 0`.
**Approach 1 - Brute force:** Try every divisor length, build the repeat, compare. Time O(n * d(n)), Space O(n).
**Approach 2 - Better:** `(s + s).slice(1, -1).includes(s)`. Short and clever. Same idea as rotation.
**Approach 3 - Optimal:** Prefix function.

Trace `"abcabcabc"`: `lps = [0,0,0,1,2,3,4,5,6]`, `L = 6`, `p = 3`, `9 % 3 === 0` -> true. Trace `"abaab"`: `lps[4] = 2`, `p = 3`, `5 % 3 !== 0` -> false.

```js
function repeatedSubstringPattern(s) {
  const n = s.length, lps = prefixFunction(s), p = n - lps[n - 1];
  return lps[n - 1] > 0 && n % p === 0;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Without the `lps[n-1] > 0` check, `"a"` gives `p = 1` and `1 % 1 === 0`, which is wrongly true (a single block repeated once). Also `"aba"`: `lps = 1`, `p = 2`, `3 % 2 !== 0`, false. Having a border is not enough; the divisibility matters.

### 4. String Matching in an Array — Easy
**Companies:** Amazon, Google
**Problem:** Return every word that is a substring of some **other** word in the list. `["mass","as","hero","superhero"]` -> `["as","hero"]`.
**Intuition:** It is a many-pairs substring test. For each word, look for it inside every longer word.
**Approach 1 - Brute force:** For each pair, use `includes`. Time O(k^2 * L^2) worst case, Space O(1).
**Approach 2 - Better:** Same with KMP per pair so each test is O(L). Time O(k^2 * L).
**Approach 3 - Optimal:** For huge lists, join all words with separators and use Aho-Corasick or a suffix automaton. With `k <= 100` and `L <= 30` (the LeetCode limits) the pairwise KMP is enough, and honest.

```js
// uses strStr from problem 1
function stringMatching(words) {
  return words.filter((w, i) =>
    words.some((o, j) => i !== j && o.length > w.length && strStr(o, w) !== -1));
}
```

**Complexity:** Time O(k^2 * L), Space O(L) for the `lps` table.
**Watch out:** Two equal words (`["a","a"]`): neither counts, because the other is not longer. The `o.length > w.length` check handles that. LeetCode guarantees unique words, so no de-duplication is needed.

### 5. Longest Happy Prefix — Medium
**Companies:** Google, Amazon
**Problem:** Return the longest non-empty proper prefix of `s` that is also a suffix (not the whole string). `"level"` -> `"l"`, `"ababab"` -> `"abab"`, `"a"` -> `""`.
**Intuition:** That is exactly the definition of `lps[n-1]`. The problem is the prefix function in disguise.
**Approach 1 - Brute force:** For length n-1 down to 1, compare prefix and suffix. Time O(n^2), Space O(n).
**Approach 2 - Better:** Rolling hash of prefix and suffix at each length, O(n) expected. Needs care about collisions.
**Approach 3 - Optimal:** Prefix function, take the last value.

Trace `"ababab"`: `lps = [0,0,1,2,3,4]`, last = 4, answer `s.slice(0,4) = "abab"`. The prefix and suffix overlap (`abab` at 0 and at 2). That is allowed.

```js
// uses prefixFunction from problem 1
function longestPrefix(s) {
  return s.slice(0, prefixFunction(s).at(-1) || 0);
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** Empty string: `.at(-1)` is `undefined`, so `|| 0` saves it. Overlap is legal. "Proper" means the whole string is excluded, and the prefix function already guarantees that.

### 6. Count Occurrences of a Pattern (overlapping) — Medium
**Companies:** -
**Problem:** Count how many times `pat` appears in `text`, overlaps allowed. `"aaaa"`, `"aa"` -> `3`.
**Intuition:** With Z on `pat + sep + text`, each position whose Z value equals `m` is one match start. Overlaps come for free because every position is checked.
**Approach 1 - Brute force:** Check every start. Time O(n*m), Space O(1).
**Approach 2 - Better:** KMP search, after each hit set `k = lps[k-1]` and keep going.
**Approach 3 - Optimal:** Z-algorithm (shown here). Same O(n+m).

Trace `pat = "ab"`, `text = "abab"`: string `ab#abab` has `z = [7,0,0,2,0,2,0]`. Text starts at index 3. Positions 3 and 5 have `z = 2 = m`: two matches.

```js
function zFunction(s) {
  const n = s.length, z = new Array(n).fill(0);
  let l = 0, r = 0;
  for (let i = 1; i < n; i++) {
    if (i < r) z[i] = Math.min(r - i, z[i - l]);
    while (i + z[i] < n && s[z[i]] === s[i + z[i]]) z[i]++;
    if (i + z[i] > r) { l = i; r = i + z[i]; }
  }
  if (n) z[0] = n;
  return z;
}

function countOccurrences(text, pat) {
  if (!pat.length) return 0;
  const z = zFunction(pat + '\u0000' + text); // separator not in either string
  let c = 0;
  for (let i = pat.length + 1; i < z.length; i++) if (z[i] === pat.length) c++;
  return c;
}
```

**Complexity:** Time O(n + m), Space O(n + m).
**Watch out:** The separator must not occur in the inputs. Without it, a Z value could run past the pattern into the text and give wrong equality. Do not use `text.split(pat).length - 1`: it counts non-overlapping matches only (`"aaaa"` gives 2).

### 7. Find All Pattern Positions with Rabin-Karp — Medium
**Companies:** -
**Problem:** Return every start index where `pat` occurs in `text`. `"abracadabra"`, `"abra"` -> `[0, 7]`.
**Intuition:** Hash the pattern once. Slide a window hash over the text. Only when hashes agree, do a real comparison. It is the right tool when you need many window comparisons (or multiple equal-length patterns, by putting their hashes in a set).
**Approach 1 - Brute force:** Compare each window char by char. Time O(n*m).
**Approach 2 - Better:** KMP, guaranteed O(n+m).
**Approach 3 - Optimal (for the rolling-hash goal):** Double rolling hash with verification. Code is the `rabinKarp` function from Core Concepts.

Trace the roll for `text = "abcd"`, `m = 2` (use `a=1,b=2,c=3,d=4` and base 10 to see it): `ab = 12`, drop `a`: `12 - 1*10 = 2`, times 10 plus `c`: `23`. Then `34`. The window moves in O(1).

```js
// rabinKarp(text, pat) from Core Concepts, step 3
rabinKarp('abracadabra', 'abra'); // [0, 7]
```

**Complexity:** Time expected O(n + m), Space O(1) beyond the output.
**Watch out:** (1) `h - out * p` can be negative, add `M` before `% M`. (2) Do the multiplication so no product reaches 2^53 (base under 2^20, modulus under 2^30). (3) Single mod with a fixed base can be broken by tests; use two mods and a random base. (4) Always verify on a hash hit unless the problem tolerates a tiny error.

### 8. Find All Anagrams in a String (hash version) — Medium
**Companies:** Amazon, Meta, Google, Microsoft
**Problem:** Return all start indices of substrings of `s` that are anagrams of `p`. `"cbaebabacd"`, `"abc"` -> `[0, 6]`.
**Intuition:** An anagram window has the same letter counts as `p`, in any order. A normal polynomial hash cares about order, so it is the wrong tool. Use an **order-free** hash instead: give every letter a random big number and use the **sum** over the window. Adding and removing a letter is O(1). Same letters means same sum.
**Approach 1 - Brute force:** Sort every window and compare to sorted `p`. Time O(n * m log m).
**Approach 2 - Better:** Sliding window with a 26-slot count array, compare counts at each step. Time O(26 * n). This is the standard answer; see [Sliding Window](../../01-core-linear/06-sliding-window/README.md).
**Approach 3 - Optimal:** Window sum hash for O(1) per step, and confirm candidates with the counts so there is no false positive.

Trace `p = "abc"`, `s = "cbaeb"`. Window `cba` has sum `r(c)+r(b)+r(a)` = target. Next, add `e`, remove `c`: sum changes, so no match. Counts are only compared when sums are equal.

```js
function findAnagrams(s, p) {
  const m = p.length, res = [];
  if (m > s.length) return res;
  const rnd = Array.from({ length: 26 }, () => 1 + Math.floor(Math.random() * 2 ** 30));
  const val = (ch) => rnd[ch.charCodeAt(0) - 97];
  let target = 0, win = 0;
  const need = new Array(26).fill(0), have = new Array(26).fill(0);
  for (const ch of p) { target += val(ch); need[ch.charCodeAt(0) - 97]++; }
  const same = () => need.every((v, i) => v === have[i]);
  for (let i = 0; i < s.length; i++) {
    win += val(s[i]); have[s.charCodeAt(i) - 97]++;
    if (i >= m) { win -= val(s[i - m]); have[s.charCodeAt(i - m) - 97]--; }
    if (i >= m - 1 && win === target && same()) res.push(i - m + 1);
  }
  return res;
}
```

**Complexity:** Time O(n) expected (the `same()` call costs 26 but runs only on sum matches), Space O(1).
**Watch out:** Sums stay below 2^53 because each value is under 2^30 and a window has at most `m` letters (fine for `m` under about 8 million). Honest note: the pure count-array solution is already O(26n), so in an interview write that first. The hash version is a technique demo, and it shines when the alphabet is huge (words, not letters).

### 9. Shortest Palindrome — Hard
**Companies:** Google, Amazon, Meta
**Problem:** You may add characters only at the **front** of `s`. Return the shortest palindrome you can make. `"aacecaaa"` -> `"aaacecaaa"`, `"abcd"` -> `"dcbabcd"`.
**Intuition:** Find the longest palindromic **prefix** of `s`. Whatever is left after it, reverse it and put it in front. So the real task is "longest palindromic prefix".
**Approach 1 - Brute force:** For each length from n down, test if the prefix is a palindrome. Time O(n^2), Space O(n).
**Approach 2 - Better:** Rolling hash of prefix versus reverse of prefix, O(n) expected.
**Approach 3 - Optimal:** KMP trick. Build `t = s + '#' + reverse(s)`. The last `lps` value is the longest prefix of `s` that equals a suffix of `reverse(s)`. A suffix of `reverse(s)` is the reverse of a prefix of `s`. So prefix equals its own reverse: a palindromic prefix.

Trace `s = "aacecaaa"`. `t = aacecaaa#aaacecaa`. The last `lps` is 7, so the palindromic prefix is `aacecaa` (length 7). Remaining part is `"a"`. Reverse it, prepend: `"a" + "aacecaaa"` = `"aaacecaaa"`.

```js
// uses prefixFunction from problem 1
function shortestPalindrome(s) {
  const rev = [...s].reverse().join('');
  const lps = prefixFunction(s + '#' + rev);
  return [...s.slice(lps.at(-1))].reverse().join('') + s;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** The `#` separator is mandatory. Without it, the prefix function could return a match longer than `n` that crosses the join, or a wrong value (`"aaaa"` would break). Empty string returns `""`. A string that is already a palindrome gives `lps = n` and nothing is added.

### 10. Minimum Characters to Add in Front to Make a Palindrome — Medium
**Companies:** -
**Problem:** Return only the **count** of characters you must add at the front. `"aacecaaa"` -> `1`, `"abcd"` -> `3`. (Classic GeeksforGeeks / interview-sheet variant of problem 9.)
**Intuition:** Same as problem 9: the answer is `n - (longest palindromic prefix length)`. Here we solve it with the Z-array for variety.
**Approach 1 - Brute force:** Check each prefix for being a palindrome. O(n^2).
**Approach 2 - Better:** KMP as in problem 9: `n - lps.at(-1)` on `s + '#' + reverse(s)`.
**Approach 3 - Optimal:** Z-array on the same string. A prefix of length `k` of `s` is a palindrome exactly when it equals the end part of `reverse(s)` of length `k`. In `t = s + '#' + rev`, that suffix of `rev` starts at index `n + 1 + (n - k)`. So we need `z[n + 1 + (n - k)] === k`. Take the largest such `k`.

Trace `s = "abcd"`, `t = abcd#dcba`. `k = 4`: `z[5+0] = z[5]` (starts `d`) is 0, not 4. `k = 3`: `z[6]` is 0. `k = 2`: `z[7]` is 0. `k = 1`: `z[8]` = `a` matches `a` then end: `z = 1 = k`. Palindromic prefix length 1, answer `4 - 1 = 3`.

```js
// uses zFunction from problem 6
function minPrepend(s) {
  const n = s.length, z = zFunction(s + '#' + [...s].reverse().join(''));
  for (let k = n; k >= 1; k--) if (z[n + 1 + (n - k)] === k) return n - k;
  return n; // only for empty s (loop did not run)
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** `k = 1` always works for non-empty `s` (a single char is a palindrome), so the answer is never more than `n - 1`. For empty input the function returns 0 (n = 0). A wrong off-by-one in the index is the usual bug, so test with `"a"`, `"aa"`, `"ab"`.

### 11. Longest Palindromic Substring (Manacher) — Medium
**Companies:** Amazon, Microsoft, Google, Meta, Apple
**Problem:** Return the longest substring of `s` that reads the same both ways. `"babad"` -> `"bab"` (or `"aba"`), `"cbbd"` -> `"bb"`.
**Intuition:** Every palindrome has a center. Manacher computes the radius at every center in O(n) total by reusing the mirror image inside the biggest palindrome seen so far.
**Approach 1 - Brute force:** All substrings, check each. Time O(n^3), Space O(1).
**Approach 2 - Better:** Expand around each of the 2n-1 centers. Time O(n^2), Space O(1). Usually accepted in interviews; it is the first answer to give.
**Approach 3 - Optimal:** Manacher. Take the largest `2*d1[i] - 1` (odd) and `2*d2[i]` (even), and convert the center to a start index.

Trace `"cbbd"`: `d1 = [1,1,1,1]`, `d2 = [0,0,1,0]`: `d2[2] = 1` (center between the two b's gives `bb`). Even length `2*1 = 2`, start `i - d2[i] = 2 - 1 = 1`. Slice `s[1..3)` = `"bb"`.

```js
// uses manacher from Core Concepts, step 4
function longestPalindrome(s) {
  const { d1, d2 } = manacher(s);
  let best = 0, start = 0;
  for (let i = 0; i < s.length; i++) {
    if (2 * d1[i] - 1 > best) { best = 2 * d1[i] - 1; start = i - d1[i] + 1; }
    if (2 * d2[i] > best) { best = 2 * d2[i]; start = i - d2[i]; }
  }
  return s.slice(start, start + best);
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** The start index formulas differ for odd and even. Odd: `i - d1[i] + 1`. Even: `i - d2[i]`. Many palindromes tie; the answer is usually "any one", so check what the problem accepts. For `n` up to 1000 the O(n^2) expand is fine, and writing it correctly beats a shaky Manacher.

### 12. Palindromic Substrings (count) — Medium
**Companies:** Meta, Amazon, Google, Microsoft
**Problem:** Count all substrings of `s` that are palindromes, counting equal substrings at different positions separately. `"abc"` -> `3`, `"aaa"` -> `6`.
**Intuition:** `d1[i]` is how many odd palindromes are centered at `i`. `d2[i]` is how many even ones are centered at the gap before `i`. Add them up.
**Approach 1 - Brute force:** Check every substring. O(n^3).
**Approach 2 - Better:** Expand around centers, count each successful expansion. O(n^2). Also a DP table `dp[i][j]`, see [String DP](../../05-dynamic-programming/06-string-dp/README.md).
**Approach 3 - Optimal:** Manacher, sum of both arrays.

Trace `"aaa"`: `d1 = [1,2,1]` (sum 4), `d2 = [0,1,1]` (sum 2). Total 6: `a`,`a`,`a`,`aa`,`aa`,`aaa`.

```js
// uses manacher from Core Concepts, step 4
function countSubstrings(s) {
  const { d1, d2 } = manacher(s);
  let c = 0;
  for (let i = 0; i < s.length; i++) c += d1[i] + d2[i];
  return c;
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** The count can be as large as `n(n+1)/2`, about 5e9 for `n = 1e5`. That is fine for a JS double (exact below 9e15), but would overflow a 32-bit int in other languages. Say that out loud.

### 13. Longest Duplicate Substring — Hard
**Companies:** Google, Amazon
**Problem:** Return any longest substring of `s` that occurs **at least twice** (overlap allowed), or `""`. `"banana"` -> `"ana"`.
**Intuition:** If a duplicate of length `L` exists, then a duplicate of every shorter length exists too (cut a char off both copies). That is a **monotone** yes/no, so binary search the length. For a fixed `L`, use a rolling hash over all windows and look for two equal hashes.
**Approach 1 - Brute force:** For every length and start, put substrings in a set. Time O(n^3) or worse.
**Approach 2 - Better:** Suffix array + LCP array: the answer is the max LCP. Time O(n log n) to O(n log^2 n), more code.
**Approach 3 - Optimal (interview):** Binary search on `L` plus a double rolling hash. Time O(n log n) expected.

Trace `"banana"`: `L = 3` (middle of 1..5). Windows `ban, ana, nan, ana`: `ana` repeats at index 1 and 3, found, so try bigger. `L = 4`: `bana, anan, nana`, no repeat. `L = 5`: no. Best is `L = 3`, start 1 -> `"ana"`.

```js
// M1, M2, B from Core Concepts, step 3
function longestDupSubstring(s) {
  const n = s.length, c = Array.from(s, (ch) => ch.charCodeAt(0) - 96);
  const find = (L) => { // start index of a duplicate window of length L, or -1
    let p1 = 1, p2 = 1, h1 = 0, h2 = 0;
    for (let i = 0; i < L; i++) {
      if (i < L - 1) { p1 = (p1 * B) % M1; p2 = (p2 * B) % M2; }
      h1 = (h1 * B + c[i]) % M1; h2 = (h2 * B + c[i]) % M2;
    }
    const seen = new Map();
    for (let i = 0; ; i++) {
      const key = h1 + ',' + h2;
      if (seen.has(key)) {
        const j = seen.get(key);
        if (s.startsWith(s.slice(i, i + L), j)) return i; // verify real equality
      } else seen.set(key, i);
      if (i + L >= n) return -1;
      h1 = ((h1 - c[i] * p1 % M1 + M1) * B + c[i + L]) % M1;
      h2 = ((h2 - c[i] * p2 % M2 + M2) * B + c[i + L]) % M2;
    }
  };
  let lo = 1, hi = n - 1, bestStart = -1, bestLen = 0;
  while (lo <= hi) {
    const mid = (lo + hi) >> 1, at = find(mid);
    if (at !== -1) { bestStart = at; bestLen = mid; lo = mid + 1; } else hi = mid - 1;
  }
  return bestStart === -1 ? '' : s.slice(bestStart, bestStart + bestLen);
}
```

**Complexity:** Time O(n log n) expected, Space O(n) for the map (string keys).
**Watch out:** (1) Binary search range is `1..n-1`, never `n` (a duplicate can't be the full string). (2) Letters map to `1..26`, not `0..25`, or `a` hashes like an empty char. (3) Overlapping copies are allowed: `"aaaa"` gives `"aaa"`. (4) On a collision the map keeps only the first index, so in a very rare case a real duplicate can be missed. With two mods that risk is negligible; say so.

### 14. Distinct Echo Substrings — Hard
**Companies:** -
**Problem:** Count distinct non-empty substrings that can be written as `a + a` (some string twice in a row). `"abcabcabc"` -> `3` (`abcabc`, `bcabca`, `cabcab`), `"leetcodeleetcode"` -> `2`.
**Intuition:** Fix the half-length `L`. A position `i` starts an echo if `s[i..i+L) === s[i+L..i+2L)`. That is the same as `s[j] === s[j+L]` for `L` positions in a row. So keep a **run counter** of consecutive equal pairs. When the run reaches `L`, an echo ends here. Use a hash only to **deduplicate** the repeated halves.
**Approach 1 - Brute force:** For each start and each `L`, compare halves and put the whole substring in a Set. Time O(n^3), memory heavy.
**Approach 2 - Better:** Same with halves compared via hashes. O(n^2) pairs, each O(1) if you have prefix hashes.
**Approach 3 - Optimal:** Run counter finds echoes in O(1) each; prefix hash dedupes. Total O(n^2). (With a suffix array you can get lower, but not worth it here.)

Trace `"abab"`, `L = 2`: pairs `s[i] vs s[i+2]`: `a=a`, `b=b` -> run 1, then run 2 at `i = 1`. `run >= 2`, echo starts at `i - L + 1 = 0`, half is `ab`. Added. Answer 1 (`abab`).

```js
// M1, M2, B from Core Concepts, step 3
function mulmod(a, b, M) { // a, b < 2^30: split b to stay under 2^53
  return ((a * (b >>> 16)) % M * 65536 + a * (b & 65535)) % M;
}

function distinctEchoSubstrings(s) {
  const n = s.length, H1 = [0], H2 = [0], P1 = [1], P2 = [1];
  for (let i = 0; i < n; i++) {
    const ch = s.charCodeAt(i);
    H1.push((H1[i] * B + ch) % M1); H2.push((H2[i] * B + ch) % M2);
    P1.push(mulmod(P1[i], B, M1)); P2.push(mulmod(P2[i], B, M2));
  }
  const hash = (l, len) => [
    (H1[l + len] - mulmod(H1[l], P1[len], M1) + M1) % M1,
    (H2[l + len] - mulmod(H2[l], P2[len], M2) + M2) % M2,
  ].join(',');
  const seen = new Set();
  for (let L = 1; 2 * L <= n; L++) {
    let run = 0; // consecutive i with s[i] === s[i + L]
    for (let i = 0; i + L < n; i++) {
      run = s[i] === s[i + L] ? run + 1 : 0;
      if (run >= L) seen.add(L + ':' + hash(i - L + 1, L)); // echo starts at i-L+1
    }
  }
  return seen.size;
}
```

**Complexity:** Time O(n^2), Space O(n + number of distinct echoes).
**Watch out:** Here we need `mulmod` because a range hash multiplies two numbers near 1e9. Include `L` in the Set key, since halves of different lengths could (rarely) share a hash. Dedupe by the **half**, not by the full echo; both identify the same string. Keep `run >= L`, not `=== L`: runs can keep growing and each new end gives a new echo with a different start.

## Common Mistakes

- **"KMP's inner `while` makes it O(n*m)."** No. `k` rises by at most one per char, so total fall-backs are at most n. It is amortized O(n).
- **"Rabin-Karp hash equal means strings equal."** No. It means probably equal. Verify when you can, and use two moduli when you cannot.
- **"JS numbers are fine for any modulus."** Doubles are exact only to 2^53. `a * b` with both near 1e9 breaks silently. Keep products small, use `mulmod`, or use BigInt.
- **"A negative hash is fine."** In JS `%` keeps the sign. Add `M` before taking `% M` when you subtract.
- **Forgetting the separator in `pat + '#' + text`.** Matches and prefix values then run across the join. Always use a char not in the input.
- **"Manacher needs the `#` trick."** Not required. The `d1`/`d2` version handles odd and even separately and is easier to get right.
- **"Palindromes only have a center on a letter."** Even palindromes (`abba`) center between letters. Cover both.
- **Using `split(pat).length - 1` to count occurrences.** It skips overlaps.
- **Binary searching duplicate length without checking monotonicity.** It works here because shorter duplicates always exist. Say why.
- **Using a polynomial hash for anagrams.** It is order-sensitive. Use counts or a sum of per-letter random values.

## Interview Tips

- Start with the simple version: `indexOf` or expand-around-center. Say its cost, then say "I can bring this to linear". Interviewers often accept the simple answer for `n <= 1000`.
- When you write KMP, **say the invariant out loud**: "`lps[i]` is the longest proper prefix of `s[0..i]` that is also a suffix". Then the code is easy to explain.
- For hashing, volunteer the risks: collisions, overflow in JS, negative values after subtraction, random base. This signals real experience.
- Follow-ups you will hear: "What if there are many patterns?" (Aho-Corasick). "What if you need all repeated substrings?" (suffix array + LCP). "Can you do it with O(1) extra space?" (Z/KMP need O(m); two-pointer brute force is O(1) space but slower.) "What about Unicode?" (JS strings are UTF-16; `charCodeAt` splits emoji into two units, use `[...s]` when it matters.)
- Edge cases to mention: empty string or empty pattern, pattern longer than text, all same char (`aaaa`), no match, overlapping matches, single char, string that is already a palindrome.
- Test your code on a tiny example by hand: `aabaaab` for the prefix function (`0 1 0 1 2 2 3`), `abaab` for Manacher.
- If you forget Manacher in the interview, expand-around-center is acceptable. Do not half-write Manacher with bugs.

## Practice List

**Easy**
- Implement strStr (Find the Index of the First Occurrence in a String)
- Rotate String
- Repeated Substring Pattern
- Valid Palindrome
- Longest Common Prefix
- String Matching in an Array

**Medium**
- Longest Happy Prefix
- Longest Palindromic Substring
- Palindromic Substrings
- Find All Anagrams in a String
- Repeated String Match
- Sum of Scores of Built Strings
- Maximum Repeating Substring
- Longest Chunked Palindrome Decomposition
- Count Unique Palindromic Subsequences of Length 3

**Hard**
- Shortest Palindrome
- Longest Duplicate Substring
- Distinct Echo Substrings
- Longest Common Subpath
- Palindrome Pairs
- Count Different Palindromic Subsequences
- Find Substring With Given Hash Value
- Shortest Common Supersequence

## Key Takeaways

- **The prefix function (LPS) is a table of borders.** It is the engine behind KMP, repeated-pattern checks, happy prefix and shortest palindrome.
- **KMP never moves the text pointer back.** On a mismatch it only shrinks the matched length `k`, so the total is **O(n + m)** (amortized).
- **Z-array = how far the prefix matches at each position.** Search by running it on `pattern + sep + text`. Always use a separator.
- **Rolling hashes trade certainty for speed.** Verify matches, use two moduli, a random base, and keep products under 2^53 in JavaScript (or use `mulmod`/BigInt).
- **"Longest X that repeats" often means binary search on length plus a hash**, because shorter repeats always exist.
- **Manacher finds every palindrome radius in O(n)** with the mirror trick; the count of palindromic substrings is `sum(d1) + sum(d2)`.
- **Suffix arrays and automata are for bigger jobs.** Know their names and what they answer; reach for them only when asked.

---

[← Previous: Tree and Digit DP](../../05-dynamic-programming/11-tree-and-digit-dp/README.md) | [Next: Segment Tree and Fenwick →](../02-segment-tree-and-fenwick/README.md) | [Back to Advanced Index](../README.md)
