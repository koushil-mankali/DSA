# Strings

> **Estimated Time:** 4 hr | **Difficulty:** Beginner
> **Prerequisites:** [03 - Hashing](../03-hashing/README.md)

---

## What You'll Learn

- You can explain why strings in JavaScript are immutable and how that changes the cost of your code.
- You can use counting arrays, two pointers and "expand from the centre" to solve most string questions.
- You can build strings the right way (array + `join`) and avoid hidden O(n²) loops.
- You can handle the edge cases interviewers love: empty string, spaces, signs, overflow, carry.
- You can solve the classic string set: anagram, palindromes, prefix, isomorphic, rotation, compression, roman numerals, big-number math, atoi, encode/decode.

## The Big Idea

Think of a string as a row of letter tiles glued to a board. You can read any tile fast, but you cannot swap one tile. To change a word you must make a new board and copy the tiles over. That is exactly how JavaScript strings work: they are **immutable**.

Most string problems are not hard ideas. They are array problems in disguise. A string is just an array of characters, so the same tools work: counting with a hash map or a 26-slot array, two pointers from both ends, and a left-to-right scan that remembers a little state. The extra difficulty is small details: spaces, upper vs lower case, signs, overflow, off-by-one.

Interviewers love strings because the problems are short to state, have no setup, and quickly show whether you are careful. A candidate who forgets the empty string or builds a string with `+=` in a loop on a big input shows habits that matter in real work.

Key facts for JavaScript:

| Fact | Why it matters |
|---|---|
| `s[i]` and `s.length` are O(1) | Safe to use in loops |
| Strings are immutable: `s[0] = 'x'` silently does nothing | To "edit", build a new string or work on an array of characters (`[...s]` or `s.split('')`) |
| `s.slice`, `s.substring`, `s + t`, `s.includes`, `s.indexOf` cost O(length) | A call inside a loop can make the whole thing O(n²) |
| `s.charCodeAt(i) - 97` gives 0..25 for `a..z` | Use it as an index into a counting array |
| Build output with `parts.push(x)` then `parts.join('')` | Safer for cost than many `+=` (modern engines optimise `+=` well, but `join` is never wrong) |
| `[...s]` iterates by Unicode code point, `s[i]` by UTF-16 unit | Matters only for emoji; interview problems almost always say "lowercase English letters" or ASCII |

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "same letters", "rearrange", "anagram" | Count letters (26-slot array or Map), or sort |
| "palindrome", "reads the same both ways" | Two pointers from both ends, or expand from a centre |
| "longest/count palindromic substring" | Expand around every centre (2n-1 centres) |
| "common prefix", "starts with" | Compare column by column, or shrink the prefix |
| "one-to-one mapping between characters" | Two maps (forward and backward) |
| "rotation of a string" | Search in `s + s` |
| "in place", "O(1) extra space" on characters | Read pointer and write pointer |
| "add / multiply numbers given as strings" | Digit by digit from the right with a carry |
| "convert to integer", "parse" | State machine: spaces, sign, digits, overflow |
| "send many strings as one string" | Length prefix, e.g. `4#word` |
| "longest substring with ... (no repeats, k distinct)" | Sliding window, see [06 - Sliding Window](../06-sliding-window/README.md) |
| "find pattern in text" | `indexOf` in practice; KMP / Z / rolling hash if asked to build it, see [01 - String Algorithms](../../06-advanced/01-string-algorithms/README.md) |
| "all valid parentheses of n pairs" | Backtracking, see [02 - Backtracking](../../02-intermediate/02-backtracking/README.md) |

## Core Concepts / Template

### Tool 1: Count letters

If the alphabet is small and fixed (26 letters), use an array of 26 numbers. It is faster and simpler than a Map.

```js
function countLetters(s) {
  const count = new Array(26).fill(0);
  for (let i = 0; i < s.length; i++) count[s.charCodeAt(i) - 97]++; // 'a' is 97
  return count;
}
```

For any characters (unicode, mixed case, digits) use a `Map`. Two strings are anagrams exactly when their count tables are equal.

### Tool 2: Two pointers from both ends

```js
function bothEnds(s) {
  let l = 0, r = s.length - 1;
  while (l < r) {
    // compare s[l] and s[r]; skip or stop as the problem needs
    l++; r--;
  }
}
```

Trace on `"racecar"`: (0,6) r=r, (1,5) a=a, (2,4) c=c, pointers meet at 3. Palindrome.

### Tool 3: Expand around a centre

A palindrome mirrors around its middle. The middle is a character (odd length) or the gap between two characters (even length). Pick every centre, grow outward while the two ends match.

```js
function expand(s, l, r) {
  while (l >= 0 && r < s.length && s[l] === s[r]) { l--; r++; }
  return r - l - 1; // length of the palindrome found
}
// for each i: expand(s, i, i) is odd, expand(s, i, i + 1) is even
```

Trace on `"babad"`, centre i=1 (`a`): l=1,r=1 match; l=0,r=2 `b`=`b` match; l=-1 stop. Length = 2 - (-1) - 1 = 3 -> `"bab"`.

### Tool 4: Read pointer and write pointer

Used for in-place edits. `i` reads every character, `w` writes the next output slot. `w <= i` always, so you never overwrite something you still need. See String Compression below.

### Tool 5: Digit-by-digit with a carry

Go from the right end, add digits, keep `carry`, write `sum % base`, carry `sum / base`. Same method as school arithmetic. Used for Add Binary and Multiply Strings.

### Small bonus: Minimum Add to Make Parentheses Valid

Not worth a full section, but it shows the "scan with a counter" idea. Keep `open` = unmatched `(`. When you see `)` with `open == 0`, you need one more `(` (count it). Answer = unmatched `(` left + `)` that had no partner.

```js
function minAddToMakeValid(s) {
  let open = 0, add = 0;
  for (const ch of s) {
    if (ch === '(') open++;
    else if (open > 0) open--;
    else add++;
  }
  return open + add;
}
// "())" -> 1, "(((" -> 3, ")(" -> 2.   Time O(n), Space O(1)
```

### Where the other classic string problems live

| Problem | Where to learn it |
|---|---|
| Longest Substring Without Repeating Characters | [06 - Sliding Window](../06-sliding-window/README.md) (window + set/map of last seen index) |
| Implement strStr / Find the Index of the First Occurrence | In an interview say `s.indexOf(p)`; if they want it built, brute force is O(n·m), the O(n+m) way is KMP in [01 - String Algorithms](../../06-advanced/01-string-algorithms/README.md) |
| Generate Parentheses | [02 - Backtracking](../../02-intermediate/02-backtracking/README.md) |
| Longest Common Subsequence, Edit Distance | [06 - String DP](../../05-dynamic-programming/06-string-dp/README.md) |
| Zigzag Conversion | Practice list below; idea: keep one array per row, walk down and up, append each character to the current row |
| Count and Say | Practice list below; idea: build term k from term k-1 by reading runs of equal digits ("1211" -> "one 1, one 2, two 1s" -> "111221") |

## Problems (easy -> hard)

### 1. Valid Anagram — Easy
**Companies:** Amazon, Google, Meta, Microsoft
**Problem:** Given two strings `s` and `t`, return true if `t` is a rearrangement of `s`. Example: `"anagram", "nagaram"` -> `true`; `"rat", "car"` -> `false`.
**Intuition:** Two words are anagrams if they have the same letters with the same counts. Order does not matter, so count instead of compare.
**Approach 1 - Brute force:** Sort both strings and compare. Time O(n log n), Space O(n).
**Approach 2 - Better:** Two count tables, compare them. Time O(n), Space O(1) for a fixed alphabet.
**Approach 3 - Optimal:** One table. Add 1 for each letter of `s`, subtract 1 for each letter of `t`. If all slots are 0, they match. Different lengths cannot match, so exit early.

Trace for `s="ab"`, `t="ba"`: i=0: a +1, b -1 (a=1, b=-1). i=1: b +1, a -1 (a=0, b=0). All zero -> true.

```js
function isAnagram(s, t) {
  if (s.length !== t.length) return false;
  const c = new Array(26).fill(0);
  for (let i = 0; i < s.length; i++) {
    c[s.charCodeAt(i) - 97]++;
    c[t.charCodeAt(i) - 97]--;
  }
  return c.every((x) => x === 0);
}
```

**Complexity:** Time O(n), Space O(1) (26 slots).
**Watch out:** If the follow-up says "unicode characters", switch to a `Map`. Check lengths first, otherwise a longer `t` could hide extra letters.

### 2. Valid Palindrome (and Valid Palindrome II) — Easy
**Companies:** Meta, Amazon, Microsoft, Google
**Problem:** (I) Ignore everything that is not a letter or digit, ignore case, and say if the string is a palindrome. `"A man, a plan, a canal: Panama"` -> `true`. (II) Given a string, can you make it a palindrome by deleting at most one character? `"abca"` -> `true`.
**Intuition:** Compare the outer pair, then move inward. In II, the first time the outer pair differs you get exactly one chance: skip the left one or skip the right one, and the rest must be a clean palindrome.
**Approach 1 - Brute force:** (I) Build a cleaned, lowercase string, reverse it, compare. Time O(n), Space O(n).
**Approach 2 - Better:** (II) Try deleting every index and check each result. Time O(n²), Space O(n).
**Approach 3 - Optimal:** Two pointers, no extra string. In I, move a pointer past junk characters. In II, on the first mismatch test the two options with a helper.

Trace for II on `"abca"`: (0,3) a=a ok. (1,2) b vs c mismatch. Try skip left: check(2,2) -> true. Return true.

```js
function isPalindrome(s) {
  const ok = (ch) => /[a-z0-9]/i.test(ch);
  let l = 0, r = s.length - 1;
  while (l < r) {
    if (!ok(s[l])) { l++; continue; }
    if (!ok(s[r])) { r--; continue; }
    if (s[l].toLowerCase() !== s[r].toLowerCase()) return false;
    l++; r--;
  }
  return true;
}

function validPalindromeII(s) {
  const check = (l, r) => {
    while (l < r) if (s[l++] !== s[r--]) return false;
    return true;
  };
  let l = 0, r = s.length - 1;
  while (l < r) {
    if (s[l] !== s[r]) return check(l + 1, r) || check(l, r - 1);
    l++; r--;
  }
  return true;
}
```

**Complexity:** Time O(n), Space O(1) for both.
**Watch out:** An empty string or a string of only punctuation is a palindrome (`true`). Digits count as characters: `"0P"` is `false`. In II, do not try to skip a second time: after the first mismatch the helper must be a strict check.

### 3. Longest Common Prefix — Easy
**Companies:** Amazon, Google, Microsoft, Apple
**Problem:** Return the longest starting part shared by all strings in an array, or `""`. `["flower","flow","flight"]` -> `"fl"`.
**Intuition:** The answer can never be longer than the shortest word. Start with the first word as the guess, and cut it down whenever another word does not start with it.
**Approach 1 - Brute force:** Compare column by column: for index `j`, check that every word has the same character at `j`. Stop at the first mismatch. Time O(S) where S is the total number of characters, Space O(1).
**Approach 2 - Better:** Sort the array; only compare the first and last word (they are the most different). Time O(S + n log n) in practice.
**Approach 3 - Optimal:** Shrink a candidate prefix. For each word, while the word does not start with the candidate, drop its last letter.

Trace: prefix = `flower`. `flow`: starts with `flower`? no -> `flowe` -> `flow` yes. `flight`: `flow`? no -> `flo` -> `fl` yes. Result `fl`.

```js
function longestCommonPrefix(strs) {
  let prefix = strs[0];
  for (const s of strs) {
    while (!s.startsWith(prefix)) prefix = prefix.slice(0, -1);
  }
  return prefix;
}
```

**Complexity:** Time O(S) worst case (S = total characters), Space O(1) extra.
**Watch out:** The loop ends because `""` is a prefix of everything. If the array can be empty, return `""` first. A one-word array returns that word.

### 4. Isomorphic Strings — Easy
**Companies:** Amazon, LinkedIn, Google
**Problem:** Two strings are isomorphic if you can replace characters in `s` (each letter always becomes the same letter) to get `t`, and no two different letters map to the same one. `"egg","add"` -> `true`; `"foo","bar"` -> `false`; `"badc","baba"` -> `false`.
**Intuition:** The mapping must be a one-to-one match (a bijection). Checking only `s -> t` misses the case where two letters of `s` land on the same letter of `t`. So check both directions.
**Approach 1 - Brute force:** For each pair of positions, check that equal-in-`s` means equal-in-`t` and the reverse. Time O(n²), Space O(1).
**Approach 2 - Better:** Convert both strings into "first seen index" patterns and compare the patterns. Time O(n), Space O(n).
**Approach 3 - Optimal:** Keep two maps, `s->t` and `t->s`. At each index, if either map already has an entry that disagrees, return false; otherwise record both.

Trace `"badc","baba"`: b->b, a->a, d->b (ok in s->t) but t->s has b->b, now b would map back to d. Conflict -> false.

```js
function isIsomorphic(s, t) {
  if (s.length !== t.length) return false;
  const st = new Map(), ts = new Map();
  for (let i = 0; i < s.length; i++) {
    const a = s[i], b = t[i];
    if (st.has(a) && st.get(a) !== b) return false;
    if (ts.has(b) && ts.get(b) !== a) return false;
    st.set(a, b); ts.set(b, a);
  }
  return true;
}
```

**Complexity:** Time O(n), Space O(k) where k is the alphabet size (O(1) for ASCII).
**Watch out:** Using one map is the classic bug. The same trick solves "Word Pattern" (map letters to words).

### 5. Rotate String — Easy
**Companies:** Amazon, Google, Microsoft
**Problem:** A shift moves the first letter to the end. Can `s` become `goal` after some shifts? `"abcde","cdeab"` -> `true`; `"abcde","abced"` -> `false`.
**Intuition:** Write `s` twice: `abcdeabcde`. Every rotation of `s` appears inside that doubled string as a window of length n.
**Approach 1 - Brute force:** Rotate `s` one step at a time (n steps), compare each with `goal`. Time O(n²), Space O(n).
**Approach 2 - Better:** None needed.
**Approach 3 - Optimal:** Check equal lengths, then check that `goal` is a substring of `s + s`. With `includes` this is fine for interviews; with KMP the worst case is guaranteed O(n).

Trace: `s+s = "abcdeabcde"`; `"cdeab"` starts at index 2 -> true. `"abced"` is nowhere -> false.

```js
function rotateString(s, goal) {
  return s.length === goal.length && (s + s).includes(goal);
}
```

**Complexity:** Time O(n) with KMP (engine `includes` is usually fast but its worst case is not guaranteed linear), Space O(n).
**Watch out:** Forgetting the length check makes `"a"` and `"aa"` wrongly match. Mention the idea, because it comes back for "check if one array is a rotation of another".

### 6. Reverse Words in a String — Medium
**Companies:** Microsoft, Amazon, Meta, Bloomberg
**Problem:** Reverse the order of words. Words are separated by one or more spaces, and the input may have leading or trailing spaces. The output has single spaces only. `"  hello   world  "` -> `"world hello"`.
**Intuition:** Cut the string into words, ignore the empty gaps, reverse the list, join with one space. If asked to do it "by hand", scan from the right and cut each word out.
**Approach 1 - Brute force:** Split on a single space, filter out empty strings, reverse, join. Time O(n), Space O(n).
**Approach 2 - Better:** `s.trim().split(/\s+/).reverse().join(' ')`. Same cost, one line.
**Approach 3 - Optimal:** (what to show when "no split" is required) Walk `i` from the right: skip spaces, mark the word end, move left to the word start, push the slice. Only one pass and no empty pieces.

Trace `"  hi  you "`: i at the end skips the space, word `you` (slice) pushed; skip two spaces; word `hi` pushed; loop ends. Join -> `"you hi"`.

```js
function reverseWords(s) {
  return s.trim().split(/\s+/).reverse().join(' ');
}

function reverseWordsManual(s) {
  const words = [];
  let i = s.length - 1;
  while (i >= 0) {
    while (i >= 0 && s[i] === ' ') i--;
    if (i < 0) break;
    const end = i;
    while (i >= 0 && s[i] !== ' ') i--;
    words.push(s.slice(i + 1, end + 1));
  }
  return words.join(' ');
}
```

**Complexity:** Time O(n), Space O(n).
**Watch out:** `split(' ')` gives empty strings for repeated spaces. Also `"".trim().split(/\s+/)` gives `[""]`, which joins to `""`, so the empty input is still fine. In C/C++ the in-place version needs reverse-all-then-reverse-each-word; JavaScript cannot do that on a string, so say so and use the array approach.

### 7. String Compression — Medium
**Companies:** Amazon, Microsoft, Google, Meta
**Problem:** Compress an array of characters in place: each run of equal characters becomes the character followed by the run length (if length > 1; length 10+ is written as separate digits). Return the new length. `['a','a','b','b','c','c','c']` -> 6, array starts `a 2 b 2 c 3`.
**Intuition:** Reading is always at or ahead of writing, because the compressed form is never longer than the run. So one array can hold both.
**Approach 1 - Brute force:** Build a new string with `+=`, then copy back. Time O(n), Space O(n), but the task asks for in place.
**Approach 2 - Better:** None.
**Approach 3 - Optimal:** Read pointer `i` finds the end `j` of the current run. Write pointer `w` writes the char, then each digit of the run length when it is more than 1.

Trace `aabbccc`: i=0, run a x2 -> write `a`,`2` (w=2). i=2, run b x2 -> `b`,`2` (w=4). i=4, run c x3 -> `c`,`3` (w=6). Return 6.

```js
function compress(chars) {
  let w = 0, i = 0;
  while (i < chars.length) {
    const ch = chars[i];
    let j = i;
    while (j < chars.length && chars[j] === ch) j++;
    chars[w++] = ch;
    if (j - i > 1) for (const d of String(j - i)) chars[w++] = d;
    i = j;
  }
  chars.length = w; // LeetCode only reads the returned length; trimming is optional
  return w;
}
```

**Complexity:** Time O(n), Space O(1) extra.
**Watch out:** Run length 12 must be written as `'1','2'`, two cells, not one. Single characters get no `1`. Do not write the count before you know the run end.

### 8. Roman to Integer / Integer to Roman — Medium
**Companies:** Amazon, Microsoft, Google, Apple
**Problem:** Convert between Roman numerals and integers (1 to 3999). `"MCMXCIV"` <-> `1994`.
**Intuition:** Roman numerals are normally added left to right (`VIII` = 5+1+1+1). The only exception: a small symbol before a bigger one is subtracted (`IV` = 4). For the other direction, always take the biggest symbol that still fits. Put the six subtractive forms (`CM`, `CD`, `XC`, `XL`, `IX`, `IV`) in the table so the greedy method handles them.
**Approach 1 - Brute force:** Roman to int: scan and check special two-letter patterns with a lookup. Works but is clumsy.
**Approach 2 - Better:** Int to Roman by splitting into thousands, hundreds, tens, ones and looking up each digit in 4 tables.
**Approach 3 - Optimal:** (a) Roman to int: if the current value is smaller than the next one, subtract it, else add it. (b) Int to Roman: greedy over a table of 13 values from big to small.

Trace roman to int `MCMXCIV`: M +1000, C (<M) -100, M +1000, X (<C) -10, C +100, I (<V) -1, V +5 = 1994.
Trace int to roman 1994: 1000 -> M (994 left), 900 -> CM (94), 90 -> XC (4), 4 -> IV. Result `MCMXCIV`.

```js
const VAL = { I: 1, V: 5, X: 10, L: 50, C: 100, D: 500, M: 1000 };

function romanToInt(s) {
  let total = 0;
  for (let i = 0; i < s.length; i++) {
    const v = VAL[s[i]];
    total += i + 1 < s.length && v < VAL[s[i + 1]] ? -v : v;
  }
  return total;
}

function intToRoman(num) {
  const vals = [1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1];
  const syms = ['M', 'CM', 'D', 'CD', 'C', 'XC', 'L', 'XL', 'X', 'IX', 'V', 'IV', 'I'];
  let out = '';
  for (let i = 0; i < vals.length; i++) {
    while (num >= vals[i]) { out += syms[i]; num -= vals[i]; }
  }
  return out;
}
```

**Complexity:** Time O(n) for roman to int; O(1) for int to roman (the output has at most 15 symbols for numbers up to 3999). Space O(1).
**Watch out:** Do not forget the last character has no "next" one. In int to roman, leaving out the `900/400/90/40/9/4` entries gives `VIIII` style answers.

### 9. Add Binary / Multiply Strings — Medium
**Companies:** Meta, Amazon, Microsoft, Google
**Problem:** (a) Add two binary strings: `"11" + "1"` -> `"100"`. (b) Multiply two non-negative integers given as decimal strings without using BigInt or converting the whole string: `"123" * "456"` -> `"56088"`.
**Intuition:** Do it the school way, digit by digit with a carry. For multiplication, the digit at `a[i]` times the digit at `b[j]` always lands in positions `i+j` (carry) and `i+j+1` (digit) of the result, because a number with `m` digits times one with `n` digits has at most `m+n` digits.
**Approach 1 - Brute force:** Convert to numbers and use `BigInt` (`BigInt(a) * BigInt(b)`). Short, but interviewers usually forbid it. Time depends on the engine, Space O(n).
**Approach 2 - Better:** (b) Multiply `a` by each digit of `b`, shift, and add the partial results with Add Strings. Time O(m·n), Space O(m+n).
**Approach 3 - Optimal:** (a) Walk both strings from the right with a carry. (b) Fill one result array of length `m+n` as shown.

Trace add binary `"11" + "1"`: right digits 1+1 = 2 -> write 0, carry 1. Next 1+0+1 = 2 -> write 0, carry 1. Both strings used up but carry is 1 -> write 1. Reverse `001` -> `"100"`.

Trace multiply `"12" * "3"`: res = [0,0,0]. i=1,j=0: 2*3=6 -> res[2]=6. i=0,j=0: 1*3 + res[1](0) = 3 -> res[1]=3. res = [0,3,6]. Skip the leading zero -> `"36"`.

```js
function addBinary(a, b) {
  let i = a.length - 1, j = b.length - 1, carry = 0;
  const out = [];
  while (i >= 0 || j >= 0 || carry) {
    const sum = (i >= 0 ? +a[i--] : 0) + (j >= 0 ? +b[j--] : 0) + carry;
    out.push(sum % 2);
    carry = sum >> 1;
  }
  return out.reverse().join('');
}

function multiply(a, b) {
  if (a === '0' || b === '0') return '0';
  const res = new Array(a.length + b.length).fill(0);
  for (let i = a.length - 1; i >= 0; i--) {
    for (let j = b.length - 1; j >= 0; j--) {
      const sum = (a.charCodeAt(i) - 48) * (b.charCodeAt(j) - 48) + res[i + j + 1];
      res[i + j + 1] = sum % 10;
      res[i + j] += Math.floor(sum / 10);
    }
  }
  let k = 0;
  while (res[k] === 0) k++; // strip leading zeros
  return res.slice(k).join('');
}
```

**Complexity:** Add Binary: Time O(max(m, n)), Space O(max(m, n)). Multiply: Time O(m·n), Space O(m+n).
**Watch out:** The final carry in addition (the `|| carry` in the loop). In multiplication, handle `"0"` first, or the "strip zeros" loop runs off the array. `res[i+j]` may temporarily exceed 9; it is fixed on a later step because the next iteration reads it as `res[i+j+1]`. The very first slot never exceeds 9 for valid inputs.

### 10. Longest Palindromic Substring (and Count Palindromic Substrings) — Medium
**Companies:** Amazon, Microsoft, Google, Meta, Bloomberg
**Problem:** Return the longest palindromic substring of `s`. `"babad"` -> `"bab"` (or `"aba"`). Follow-up: count how many palindromic substrings exist. `"aaa"` -> 6.
**Intuition:** Every palindrome has a centre. There are `2n - 1` centres (n letters plus n-1 gaps). From each centre, grow while the ends match. That is simple and needs no table.
**Approach 1 - Brute force:** Check every substring: O(n²) substrings, O(n) check. Time O(n³), Space O(1).
**Approach 2 - Better:** Dynamic programming table `dp[i][j]` = is `s[i..j]` a palindrome (true if the ends match and the inside is a palindrome). Time O(n²), Space O(n²). See [06 - String DP](../../05-dynamic-programming/06-string-dp/README.md).
**Approach 3 - Optimal for interviews:** Expand around each centre. Time O(n²), Space O(1). (Manacher's algorithm does it in O(n) by reusing mirror information from earlier centres, but it is rarely expected. Name it, say what it gives, and move on. It is in [01 - String Algorithms](../../06-advanced/01-string-algorithms/README.md).)

Trace `"cbbd"`: centre i=1 odd: `b` only (length 1). Centre i=1 even (1,2): `b=b` -> grow (0,3): `c` vs `d` stop -> length 2. Best is `"bb"` starting at `i - ((2-1)>>1) = 1`.

```js
function longestPalindrome(s) {
  let start = 0, best = 0;
  const expand = (l, r) => {
    while (l >= 0 && r < s.length && s[l] === s[r]) { l--; r++; }
    return r - l - 1;
  };
  for (let i = 0; i < s.length; i++) {
    const len = Math.max(expand(i, i), expand(i, i + 1));
    if (len > best) { best = len; start = i - ((len - 1) >> 1); }
  }
  return s.slice(start, start + best);
}

// Count Palindromic Substrings: same loop, count every successful step
function countSubstrings(s) {
  let count = 0;
  const expand = (l, r) => {
    while (l >= 0 && r < s.length && s[l] === s[r]) { count++; l--; r++; }
  };
  for (let i = 0; i < s.length; i++) { expand(i, i); expand(i, i + 1); }
  return count;
}
```

**Complexity:** Time O(n²), Space O(1) (output slice aside).
**Watch out:** Forgetting the even centres (`"abba"` fails). The start index formula for both odd and even lengths is `i - ((len - 1) >> 1)`. Counting counts substrings by position, so `"aaa"` has 6 (three `a`, two `aa`, one `aaa`), not 3.

### 11. String to Integer (atoi) — Medium
**Companies:** Amazon, Microsoft, Meta, Google
**Problem:** Parse a string to a 32-bit signed integer: skip leading spaces, read an optional `+` or `-`, read digits until the first non-digit, clamp to `[-2^31, 2^31 - 1]`. `"   -042"` -> `-42`; `"words 987"` -> `0`; `"-91283472332"` -> `-2147483648`.
**Intuition:** It is a tiny state machine: spaces, then sign, then digits, then stop. No clever idea, just no missed rule.
**Approach 1 - Brute force:** Use `parseInt` and clamp. Fine in real life, but the interviewer wants the manual loop. Time O(n), Space O(1).
**Approach 2 - Better:** None.
**Approach 3 - Optimal:** Manual scan. Because JavaScript numbers are safe up to 2^53, you can compare to the limit after each digit (the digits keep the number small because we stop the moment it hits the limit). In languages with 32-bit ints, check `n > (MAX - digit) / 10` before multiplying.

Trace `"  -42x"`: skip 2 spaces. Sign `-` -> sign = -1. Digit 4 -> n=4. Digit 2 -> n=42. `x` stops. Return -42.

```js
function myAtoi(s) {
  const MAX = 2 ** 31 - 1, MIN = -(2 ** 31);
  let i = 0, sign = 1, n = 0;
  while (s[i] === ' ') i++;
  if (s[i] === '-' || s[i] === '+') sign = s[i++] === '-' ? -1 : 1;
  while (i < s.length && s[i] >= '0' && s[i] <= '9') {
    n = n * 10 + (s.charCodeAt(i++) - 48);
    if (sign * n >= MAX) return MAX;
    if (sign * n <= MIN) return MIN;
  }
  return sign * n;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Only a space is skipped at the start (per the usual statement), a sign is read once, `"+-12"` gives 0, `""` and `"+"` give 0. The early return on overflow also stops a huge digit string from becoming `Infinity`. Not covered: decimals or exponents; they are not part of this problem.

### 12. Encode and Decode Strings — Medium
**Companies:** Google, Meta, Amazon, LinkedIn
**Problem:** Design `encode(list)` -> one string and `decode(string)` -> the same list. The strings may contain any character, including the ones you would use as a separator, and can be empty. `["lint","code"]` -> `"4#lint4#code"` -> back.
**Intuition:** A delimiter alone breaks when the data contains the delimiter. Instead, put the **length** before each string. The decoder reads the length, then jumps exactly that many characters, so the content is never inspected.
**Approach 1 - Brute force:** Join with a rare separator (like `\u0001`). Works until the data contains it. Not safe.
**Approach 2 - Better:** Escape the separator inside the data (like CSV quotes). Correct but fiddly.
**Approach 3 - Optimal:** Length prefix + `#`: `len#text`. Decode: find the next `#` from position `i`, read the number before it, slice that many characters, move on.

Trace `["3#a",""]`: encoded `3#3#a0#`. Decode: i=0, `#` at 1, len=3 -> slice `3#a`, i=5. `#` at 6, len=0 -> `""`, i=7. Done.

```js
function encode(strs) {
  return strs.map((s) => s.length + '#' + s).join('');
}

function decode(str) {
  const out = [];
  let i = 0;
  while (i < str.length) {
    const h = str.indexOf('#', i);          // first '#' after i is always the length's end
    const len = Number(str.slice(i, h));
    out.push(str.slice(h + 1, h + 1 + len));
    i = h + 1 + len;
  }
  return out;
}
```

**Complexity:** Time O(total characters), Space O(total characters).
**Watch out:** Empty list encodes to `""` and decodes to `[]` (works). Empty strings encode to `0#`. The first `#` after `i` is always the length marker, since the length is only digits, so a `#` inside the data never confuses it. Do not use `split('#')`.

## Common Mistakes

- **"`s[0] = 'X'` changes the string."** It silently does nothing. Convert to an array, edit, `join('')`.
- **"`+=` in a loop is always fine."** On big inputs, or with `slice`/`substring` inside the loop, you can hide O(n²). Collect pieces in an array and `join` once, and do not slice in a loop when an index will do.
- **"A map `s -> t` is enough for isomorphic."** You need the reverse map too, or two letters can share one target.
- **"Palindrome means compare with the reverse."** That works but costs extra memory. Two pointers use O(1).
- **"Even-length palindromes are a special case I can skip."** Half of all centres are gaps between letters.
- **"Comparing characters with `<` is the same as comparing digits."** `'9' > '10'` is true as strings. Convert with `charCodeAt(i) - 48` before doing math.
- **"`split(' ')` handles multiple spaces."** It leaves empty strings. Use `/\s+/` after `trim()`.
- **"Uppercase and lowercase share the counting slots."** `charCodeAt - 97` gives negative or large indexes for `A-Z`. Lower-case first, or use a 128/256-slot table.

## Interview Tips

- **Ask first:** only lowercase letters or ASCII? Unicode? Can the string be empty? Is case important? In place or new string allowed?
- **Say the cost of string operations out loud:** "`slice` copies, so I will use indexes."
- **Edge cases to name:** empty string, one character, all same characters, leading/trailing spaces, sign characters, overflow, carry at the end (`"1" + "1"`), strings with `#` or your delimiter in them.
- **Pick the right tool:** counting array for letters, two pointers for symmetry, expand-around-centre for palindromes, sliding window for "longest substring such that ...", digit-by-digit with carry for big numbers.
- **Typical follow-ups:** "Do it without extra space." "What if the strings are huge or come from a stream?" (count tables work on streams; two-pointer palindromes do not). "What if characters are Unicode?" (use `Map`, mention code points vs UTF-16 units). "Can you do longest palindromic substring in O(n)?" (Manacher). "Faster substring search?" (KMP, Rabin-Karp).
- **Do not over-engineer:** in a real interview using `indexOf`, `includes`, `split`, `join` is normal. Say what you use, and say you can write it by hand if they ask.
- **Test aloud** with a tiny example plus the empty string before you say you are done.

## Practice List

**Easy**
- Reverse String
- Reverse Vowels of a String
- Valid Palindrome
- Valid Anagram
- Isomorphic Strings
- Word Pattern
- Roman to Integer
- Add Binary
- Add Strings
- Find the Index of the First Occurrence in a String (strStr)
- Longest Common Prefix
- Rotate String
- Reverse Words in a String III
- Count and Say (read runs, build the next term; start from "1")
- Length of Last Word

**Medium**
- Reverse Words in a String
- String Compression
- Integer to Roman
- Multiply Strings
- Longest Palindromic Substring
- Palindromic Substrings
- Group Anagrams
- Find All Anagrams in a String
- Permutation in String
- String to Integer (atoi)
- Zigzag Conversion (one list per row, bounce down and up)
- Encode and Decode Strings
- Minimum Add to Make Parentheses Valid
- Longest Substring Without Repeating Characters
- Generate Parentheses
- Valid Palindrome II
- Decode String
- Custom Sort String

**Hard**
- Minimum Window Substring
- Longest Valid Parentheses
- Shortest Palindrome
- Palindrome Pairs
- Edit Distance
- Longest Palindromic Subsequence
- Text Justification
- Basic Calculator

## Key Takeaways

- **Strings are immutable in JavaScript**: edit through an array, build with `push` + `join`, and watch hidden O(n) calls like `slice` in loops.
- **Counting with a 26-slot array** solves anagram-type questions in O(n) time and O(1) space.
- **Two pointers from both ends** is the default for palindrome checks; Valid Palindrome II gets exactly one skip at the first mismatch.
- **Expand around 2n-1 centres** for longest palindromic substring and palindromic substring count; Manacher exists for O(n) but is rarely required.
- **A bijection needs two maps** (isomorphic strings), and a rotation test is `goal` inside `s + s` after a length check.
- **Digit-by-digit with a carry** handles Add Binary and Multiply Strings; the product digit pair `(i, j)` writes to `i+j` and `i+j+1`.
- **Length-prefix encoding** is the safe way to pack many strings into one: never trust a delimiter alone.

---

[← Previous: Hashing](../03-hashing/README.md) | [Next: Two Pointers →](../05-two-pointers/README.md) | [Back to Core Linear Index](../README.md)
