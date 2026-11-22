# Bit Manipulation

> **Estimated Time:** 5 hr | **Difficulty:** Intermediate
> **Prerequisites:** [03 - JavaScript for DSA](../../00-foundations/03-javascript-for-dsa/README.md), [04 - Math Basics](../../00-foundations/04-math-basics/README.md), [03 - Hashing](../../01-core-linear/03-hashing/README.md)

---

## What You'll Learn
- You can read and write numbers as bits, and use `&`, `|`, `^`, `~`, `<<`, `>>`, `>>>` without guessing.
- You can explain two's complement in plain words, and avoid the JavaScript 32-bit signed traps.
- You can use the core tricks: `n & (n - 1)`, `n & -n`, and set / clear / toggle / check a bit.
- You can solve the classic problems: Single Number I/II/III, Missing Number, Counting Bits, Reverse Bits, Sum of Two Integers, Maximum XOR.
- You can enumerate all subsets with a bitmask, and say when a bit trick is the right tool.

## The Big Idea

Think of a row of light switches on a wall. Each switch is ON or OFF. A number is just a row of switches: switch 0 is worth 1, switch 1 is worth 2, switch 2 is worth 4, and so on. Bit manipulation means flipping, checking and combining those switches directly. The CPU does this in one step, so it is very fast and uses almost no memory.

Why interviewers love it:
- A few lines can replace a hash map or a sort (`Single Number` in O(1) space).
- It tests whether you understand how numbers really live in memory.
- Many "clever" answers come from one idea: **XOR cancels equal pairs**.
- The same ideas power bitmask DP, Fenwick trees and hash functions, so it shows up later in the handbook.

### The six operators in one table

| Op | Name | Rule (per bit) | Example (4-bit) |
|----|------|----------------|-----------------|
| `a & b` | AND | 1 only if both are 1 | `1100 & 1010 = 1000` |
| `a \| b` | OR | 1 if at least one is 1 | `1100 \| 1010 = 1110` |
| `a ^ b` | XOR | 1 if the bits differ | `1100 ^ 1010 = 0110` |
| `~a` | NOT | flip every bit | `~0101 = 1010` (see two's complement) |
| `a << k` | left shift | move bits left, fill 0 | `0011 << 2 = 1100` (times 2^k) |
| `a >> k` | arithmetic right shift | move bits right, copy the sign bit | `-8 >> 1 = -4` |
| `a >>> k` | logical right shift | move bits right, fill 0 | `-1 >>> 0 = 4294967295` |

XOR laws you will use all the time:
- `x ^ x = 0` (a number cancels itself)
- `x ^ 0 = x`
- `a ^ b = b ^ a`, and order does not matter (so a whole array can be XORed in any order)

### Two's complement in plain words

Computers store negative numbers so that addition still works with no special case. Rule: to get `-x`, **flip all bits of x, then add 1**.

```
 5  = 00000000 00000000 00000000 00000101
flip= 11111111 11111111 11111111 11111010
+1  = 11111111 11111111 11111111 11111011   <- this is -5
```

- The top bit (bit 31) is the **sign bit**: 1 means negative.
- `-1` is all ones. `~x` equals `-x - 1` (so `~5 = -6`).
- Because of this, `n & -n` isolates the lowest set bit (explained below).

### JavaScript traps (read this twice)

JavaScript numbers are 64-bit floats, but **every bitwise operator first converts its inputs to a signed 32-bit integer**, works on that, and returns a signed 32-bit integer.

| Expression | Result | Why |
|------------|--------|-----|
| `1 << 31` | `-2147483648` | bit 31 is the sign bit |
| `2 ** 32 \| 0` | `0` | bits above 32 are thrown away |
| `-5 >> 1` | `-3` | copies the sign bit, rounds toward minus infinity |
| `-5 >>> 1` | `2147483645` | zero fill, the result looks huge |
| `-5 >>> 0` | `4294967291` | the trick to see a 32-bit value as unsigned |
| `1 << 32` | `1` | the shift count is taken mod 32 |

Rules of thumb:
- Need an **unsigned** 32-bit answer (Reverse Bits)? End with `>>> 0`.
- Need to loop "while n is not zero" on a possibly negative number? Use `>>>` to shift, not `>>` (otherwise a negative number never reaches 0 and the loop never ends).
- Values beyond 32 bits (like `2^40`)? Normal bit operators silently break. Use **BigInt** (`1n << 40n`) or use arithmetic (`Math.floor(x / 2)`, `x % 2`). BigInt cannot be mixed with plain numbers: `5n & 1` throws a TypeError.
- Safe range for plain bit ops: integers in `[-2^31, 2^31 - 1]`.

## How to Spot It

| If the problem says ... | think ... |
|-------------------------|-----------|
| "every number appears twice except one" | XOR everything |
| "appears three times except one" | count each bit position mod 3 |
| "two numbers appear once" | XOR all, split by a differing bit |
| "do not use `+`, `-`, `*`, `/`" | XOR is add-without-carry, AND shifted is the carry |
| "power of two", "count set bits" | `n & (n - 1)` clears the lowest set bit |
| "constant extra space, find the missing / odd one" | XOR instead of a hash set |
| "all subsets" with `n <= ~20` | loop `mask` from `0` to `2^n - 1` |
| "maximum XOR of a pair" | binary trie, go greedy from the top bit |
| "range of numbers, bitwise AND" | common prefix of the two ends |
| "flip / differing bits between two numbers" | `popcount(a ^ b)` |
| "answer is built bit by bit", "per-bit contribution" | handle each of the 32 bit positions separately |

## Core Concepts / Template

### The five everyday tricks

```js
const isBitSet  = (n, i) => ((n >> i) & 1) === 1; // check bit i
const setBit    = (n, i) => n | (1 << i);         // force bit i to 1
const clearBit  = (n, i) => n & ~(1 << i);        // force bit i to 0
const toggleBit = (n, i) => n ^ (1 << i);         // flip bit i

// Brian Kernighan: clear the lowest set bit.
// n - 1 flips the lowest 1 and all the 0s below it, so the AND wipes exactly that 1.
const clearLowest = (n) => n & (n - 1);

// Isolate the lowest set bit. -n = ~n + 1, so only that bit survives in both.
const lowestBit = (n) => n & -n;
```

Why `n & (n - 1)` works (n = 1011 -> 11):

```
n     = 1011
n - 1 = 1010   (lowest 1 became 0, nothing below it)
n & . = 1010   <- lowest set bit gone

n     = 1000   (8)
n - 1 = 0111   (the 1 became 0, the 0s below became 1)
n & . = 0000   <- one bit only, so the result is 0 (power of two test)
```

Why `n & -n` works (n = 12 = 01100):

```
 n = 01100
-n = 10100   (flip -> 10011, add 1 -> 10100)
&  = 00100   <- only the lowest set bit
```

### Masks you should know

| Goal | Code |
|------|------|
| lowest `k` bits all ones | `(1 << k) - 1` (careful: `k = 32` breaks) |
| is `n` odd | `(n & 1) === 1` |
| `n / 2` (floor, for non-negative) | `n >> 1` |
| `n * 2^k` | `n << k` (watch for overflow past bit 31) |
| turn on all of bits 0..n-1 | `(1 << n) - 1` |
| iterate set bits of `mask` | `while (m) { const low = m & -m; ...; m ^= low; }` |

### Template: loop over the 32 bits

Many "per-bit" problems share this shape: look at each bit position alone, solve a tiny counting problem, glue answers back.

```js
for (let b = 0; b < 32; b++) {
  let ones = 0;
  for (const x of nums) ones += (x >> b) & 1;
  // use `ones` and `nums.length - ones` here
}
```

### Template: all subsets with a bitmask

```js
for (let mask = 0; mask < (1 << n); mask++) {
  for (let i = 0; i < n; i++) {
    if ((mask >> i) & 1) { /* element i is in this subset */ }
  }
}
```

Trace for `n = 3`, items `[a, b, c]`: mask `101` (5) means "take a and c".

## Problems (easy -> hard)

### 1. Check / Set / Clear / Toggle the i-th Bit — Easy
**Companies:** -
**Problem:** Given `n` and `i`, tell whether bit `i` (0 is the rightmost) is 1, then write the helpers to set, clear and toggle it. Example: `n = 5 (101), i = 1` -> bit is 0; setting it gives `7`.
**Intuition:** To look at one switch, build a mask with only that switch ON (`1 << i`) and use it with AND, OR, or XOR.
**Approach 1 - Brute force:** Convert to a binary string and index into it. Works but slow and messy with negatives. Time O(log n), Space O(log n).
**Approach 2 - Optimal:** Use the four one-liners. `check` shifts the bit down to position 0 and ANDs with 1. `set` ORs the mask. `clear` ANDs with the inverted mask. `toggle` XORs the mask.

| Op | n = 101, i = 1 | result |
|----|----------------|--------|
| check | `(101 >> 1) & 1 = 10 & 1` | 0 |
| set | `101 \| 010` | 111 = 7 |
| clear (i = 2) | `101 & ~100 = 101 & ...011` | 001 = 1 |
| toggle (i = 0) | `101 ^ 001` | 100 = 4 |

```js
const isBitSet  = (n, i) => ((n >> i) & 1) === 1;
const setBit    = (n, i) => n | (1 << i);
const clearBit  = (n, i) => n & ~(1 << i);
const toggleBit = (n, i) => n ^ (1 << i);
```

**Complexity:** Time O(1), Space O(1).
**Watch out:** `i` must be in `0..30` for safe positive masks; `1 << 31` is negative and `1 << 32` wraps to `1`.

### 2. Number of 1 Bits — Easy
**Companies:** Amazon, Apple, Microsoft, Google
**Problem:** Count how many bits are 1 in a 32-bit integer (the "Hamming weight" or popcount). Example: `11` (1011) -> `3`.
**Intuition:** Instead of visiting all 32 positions, jump from one set bit to the next. Each `n & (n - 1)` removes exactly one set bit, so the number of steps equals the number of set bits.
**Approach 1 - Brute force:** Check each of 32 bits with `(n >> i) & 1`. Time O(32), Space O(1).
**Approach 2 - Optimal (Kernighan):** Loop `n &= n - 1` and count the steps.

| step | n (binary) | count |
|------|-----------|-------|
| start | 1011 | 0 |
| 1 | 1010 | 1 |
| 2 | 1000 | 2 |
| 3 | 0000 | 3 |

```js
function hammingWeight(n) {
  let count = 0;
  while (n !== 0) { // `!== 0` not `> 0`: works for negatives too
    n &= n - 1;
    count++;
  }
  return count;
}
```

**Complexity:** Time O(k) where k is the number of set bits (at most 32), Space O(1).
**Watch out:** If the input is given as a negative number (the 32-bit pattern), `n > 0` would skip the loop. Test `-1` -> 32. Do not use `n >> 1` loops on negatives: they never reach 0.

### 3. Power of Two — Easy
**Companies:** Amazon, Google, Microsoft
**Problem:** Return true if `n` is `2^k` for some `k >= 0`. Example: `16` -> true, `6` -> false.
**Intuition:** A power of two has exactly one set bit. `n & (n - 1)` removes the lowest set bit, so the result is 0 only when there was just one.
**Approach 1 - Brute force:** Keep dividing by 2 while even; check you end at 1. Time O(log n), Space O(1).
**Approach 2 - Optimal:** `n > 0 && (n & (n - 1)) === 0`.

Trace: `16 = 10000`, `15 = 01111`, AND = `0` -> true. `6 = 110`, `5 = 101`, AND = `100` -> false.

```js
const isPowerOfTwo = (n) => n > 0 && (n & (n - 1)) === 0;
```

**Complexity:** Time O(1), Space O(1).
**Watch out:** `n = 0` gives `0 & -1 = 0` (looks like a hit) and negatives can pass for `-2^31`. The `n > 0` guard is required. Variant: power of four also needs `(n & 0x55555555) !== 0`.

### 4. Swap Two Numbers Without a Temp — Easy
**Companies:** -
**Problem:** Swap `a` and `b` without a third variable. Example: `(3, 9)` -> `(9, 3)`.
**Intuition:** XOR is its own undo. If `x = a ^ b`, then `x ^ a = b` and `x ^ b = a`.
**Approach 1 - Brute force:** Use a temp variable (clearest, and what you should write in real code).
**Approach 2 - Optimal:** Three XORs.

| step | a | b |
|------|---|---|
| start | 3 (0011) | 9 (1001) |
| `a ^= b` | 10 (1010) | 9 |
| `b ^= a` | 10 | 3 (0011) |
| `a ^= b` | 9 (1001) | 3 |

```js
function swap(a, b) {
  a ^= b;
  b ^= a;
  a ^= b;
  return [a, b];
}
```

**Complexity:** Time O(1), Space O(1).
**Watch out:** If `a` and `b` are the **same memory cell** (swapping `arr[i]` with `arr[i]` when `i === j`), the first XOR zeroes it and you lose the value. Guard with `if (i !== j)`. In modern JS, `[a, b] = [b, a]` is simpler; say the XOR way is a party trick, not production style. Works only on 32-bit integers.

### 5. Missing Number — Easy
**Companies:** Amazon, Microsoft, Apple, Meta
**Problem:** `nums` holds `n` distinct numbers from `0..n`. One is missing; find it. Example: `[3, 0, 1]` -> `2`.
**Intuition:** If we XOR every index `0..n` with every value in the array, each present number appears twice and cancels. Only the missing one survives.
**Approach 1 - Brute force:** Sort and scan for the first gap. Time O(n log n), Space O(1).
**Approach 2 - Better:** Sum formula: `n(n+1)/2 - sum(nums)`. Time O(n), Space O(1). Fine here (sums stay small), but can overflow in languages with fixed ints.
**Approach 3 - Optimal:** Start with `x = n` and XOR in `i ^ nums[i]` for each index.

Trace `[3, 0, 1]`, n = 3:

| i | nums[i] | x after |
|---|---------|---------|
| start | | 3 |
| 0 | 3 | 3 ^ 0 ^ 3 = 0 |
| 1 | 0 | 0 ^ 1 ^ 0 = 1 |
| 2 | 1 | 1 ^ 2 ^ 1 = 2 |

```js
function missingNumber(nums) {
  let x = nums.length; // the index n has no matching slot
  for (let i = 0; i < nums.length; i++) x ^= i ^ nums[i];
  return x;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Forgetting to seed with `n` (the array is one shorter than the range). Test `[0]` -> 1 and `[1]` -> 0.

### 6. Hamming Distance — Easy
**Companies:** Meta, Amazon
**Problem:** Count the bit positions where `x` and `y` differ. Example: `1` (001) and `4` (100) -> `2`.
**Intuition:** XOR gives a 1 exactly where two bits differ. So the answer is the popcount of `x ^ y`.
**Approach 1 - Brute force:** Compare bit by bit in a loop of 32. Time O(32), Space O(1).
**Approach 2 - Optimal:** Reuse the popcount from problem 2.

```js
const hammingDistance = (x, y) => hammingWeight(x ^ y); // hammingWeight from problem 2
```

**Complexity:** Time O(k), Space O(1).
**Watch out:** It is the same one-liner as Minimum Bit Flips (next problem); recognise it as a disguise.

### 7. Minimum Bit Flips to Convert Number — Easy
**Companies:** Amazon, Google
**Problem:** Smallest number of single-bit flips to turn `start` into `goal`. Example: `10` (1010) -> `7` (0111): `3`.
**Intuition:** A flip changes one bit. You must flip every position where they differ, and no others. That count is `popcount(start ^ goal)`.
**Approach 1 - Brute force:** BFS over numbers, flipping one bit at a time. Far too slow.
**Approach 2 - Optimal:** XOR then count.

Trace: `1010 ^ 0111 = 1101` -> three ones -> 3.

```js
const minBitFlips = (start, goal) => hammingWeight(start ^ goal);
```

**Complexity:** Time O(k), Space O(1).
**Watch out:** None really; the trap is overthinking it.

### 8. Single Number — Easy
**Companies:** Amazon, Google, Meta, Apple, Microsoft
**Problem:** Every element appears twice except one. Find it in O(n) time and O(1) space. Example: `[4, 1, 2, 1, 2]` -> `4`.
**Intuition:** XOR cancels equal pairs and order does not matter. After XORing everything, only the single one is left.
**Approach 1 - Brute force:** For each element, count how often it appears. Time O(n^2), Space O(1).
**Approach 2 - Better:** Hash map of counts (or a set that adds then removes). Time O(n), Space O(n). Does not meet the space limit.
**Approach 3 - Optimal:** XOR all.

Trace: `4 ^ 1 = 5`, `5 ^ 2 = 7`, `7 ^ 1 = 6`, `6 ^ 2 = 4`.

```js
function singleNumber(nums) {
  let x = 0;
  for (const v of nums) x ^= v;
  return x;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Works for negatives too (XOR works on the 32-bit pattern). It only works because every other number appears an **even** count.

### 9. Counting Bits — Easy
**Companies:** Amazon, Google, Apple
**Problem:** For every `i` in `0..n` return the number of 1 bits in `i`, as an array. Example: `n = 5` -> `[0, 1, 1, 2, 1, 2]`.
**Intuition:** `i >> 1` is `i` with its last bit removed, and we already know its answer. So `bits[i] = bits[i >> 1] + (i & 1)`. This is a tiny DP.
**Approach 1 - Brute force:** Run popcount for every `i`. Time O(n log n), Space O(1) extra.
**Approach 2 - Optimal:** DP with the recurrence above. (Another version: `bits[i] = bits[i & (i - 1)] + 1`.)

| i | binary | i >> 1 | bits[i >> 1] | i & 1 | bits[i] |
|---|--------|--------|--------------|-------|---------|
| 1 | 001 | 0 | 0 | 1 | 1 |
| 2 | 010 | 1 | 1 | 0 | 1 |
| 3 | 011 | 1 | 1 | 1 | 2 |
| 4 | 100 | 2 | 1 | 0 | 1 |
| 5 | 101 | 2 | 1 | 1 | 2 |

```js
function countBits(n) {
  const bits = new Array(n + 1).fill(0);
  for (let i = 1; i <= n; i++) bits[i] = bits[i >> 1] + (i & 1);
  return bits;
}
```

**Complexity:** Time O(n), Space O(n) for the output (no extra).
**Watch out:** The array must have `n + 1` slots. `bits[0]` stays 0.

### 10. Reverse Bits — Easy
**Companies:** Apple, Amazon, Microsoft
**Problem:** Reverse the 32 bits of an unsigned integer. Example: `43261596` (`00000010100101000001111010011100`) -> `964176192` (`00111001011110000010100101000000`).
**Intuition:** Peel bits off the right of `n` one by one, and push each onto the right of a result that grows leftward. After 32 rounds the order is mirrored.
**Approach 1 - Brute force:** Make a 32-char binary string, reverse it, parse it. Time O(32), Space O(32).
**Approach 2 - Optimal:** Loop 32 times: `result = (result << 1) | (n & 1); n >>>= 1`.

Trace on 4 bits `1011` (to show the idea): result `0 -> 1 -> 11 -> 110 -> 1101` = reversed `1101`.

```js
function reverseBits(n) {
  let result = 0;
  for (let i = 0; i < 32; i++) {
    result = (result << 1) | (n & 1);
    n >>>= 1; // logical shift so negative inputs also drain
  }
  return result >>> 0; // show as unsigned
}
```

**Complexity:** Time O(32) = O(1), Space O(1).
**Watch out:** Without `>>> 0` at the end, a result with bit 31 set prints as a negative number. Always loop exactly 32 times (leading zeros matter). Follow-up "called many times": cache the reverse of each byte and combine four lookups.

### 11. XOR of 1..n and XOR of a Range — Medium
**Companies:** -
**Problem:** Compute `1 ^ 2 ^ ... ^ n` in O(1), then use it to get `l ^ (l+1) ^ ... ^ r`. Example: `n = 7` -> `0`; range `3..9` -> `3 ^ 4 ^ ... ^ 9`.
**Intuition:** Print the prefix XORs and a pattern shows up with period 4.

| n | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| XOR 1..n | 1 | 3 | 0 | 4 | 1 | 7 | 0 | 8 |

By `n % 4`: `0 -> n`, `1 -> 1`, `2 -> n + 1`, `3 -> 0`. For a range use the prefix idea: `xor(l..r) = f(r) ^ f(l - 1)`, because the shared prefix cancels.
**Approach 1 - Brute force:** Loop from `l` to `r`. Time O(r - l).
**Approach 2 - Optimal:** The four-case formula. Time O(1).

```js
function xorUpTo(n) { // XOR of 0..n
  switch (n % 4) {
    case 0: return n;
    case 1: return 1;
    case 2: return n + 1;
    default: return 0;
  }
}
const xorRange = (l, r) => xorUpTo(r) ^ xorUpTo(l - 1);
```

**Complexity:** Time O(1), Space O(1).
**Watch out:** `l = 0` makes `xorUpTo(-1)`; `-1 % 4` is `-1` in JS and falls to `default` -> 0, which is the right answer by luck. Guard it if you want to be clear. Values past 2^31 need BigInt or care with `^` (it truncates to 32 bits).

### 12. Sum of Two Integers — Medium
**Companies:** Meta, Amazon, Google
**Problem:** Add `a` and `b` without using `+` or `-`. Example: `1, 2` -> `3`.
**Intuition:** Add the way you do on paper. XOR is the sum of each column ignoring the carry. `(a & b) << 1` is the carry, moved one column left. Repeat until no carry is left.
**Approach 1 - Brute force:** Increment/decrement one at a time (needs `+` or `-`, so not allowed; mention only).
**Approach 2 - Optimal:** Loop: `carry = (a & b) << 1; a ^= b; b = carry;` until `b === 0`.

Trace `a = 5 (101), b = 3 (011)`:

| round | a ^ b | (a & b) << 1 |
|-------|-------|--------------|
| 1 | 110 | 010 |
| 2 | 100 | 100 |
| 3 | 000 | 1000 |
| 4 | 1000 = 8 | 0 -> stop |

```js
function getSum(a, b) {
  while (b !== 0) {
    const carry = (a & b) << 1;
    a ^= b;
    b = carry;
  }
  return a;
}
```

**Complexity:** Time O(32), Space O(1).
**Watch out:** In JS the 32-bit signed wrap makes negatives work for free (`-1 + 1 = 0`, `-5 + -7 = -12`); C++/Java users add masks, you do not. Results outside the 32-bit range wrap: `2147483647 + 1` gives `-2147483648`. Say that out loud.

### 13. Subsets via Bitmask — Medium
**Companies:** Amazon, Meta, Microsoft, Google
**Problem:** Return all subsets of an array of distinct numbers. Example: `[1, 2]` -> `[[], [1], [2], [1, 2]]`.
**Intuition:** Each element is in or out: that is one bit. A number from `0` to `2^n - 1` is a complete in/out choice for all `n` elements.
**Approach 1 - Brute force / standard:** Backtracking (see [02 - Backtracking](../02-backtracking/README.md)). Time O(n * 2^n).
**Approach 2 - Optimal (iterative):** Loop over masks and read the bits.

| mask | bits (c b a) | subset of [a, b, c] |
|------|--------------|---------------------|
| 0 | 000 | [] |
| 1 | 001 | [a] |
| 3 | 011 | [a, b] |
| 5 | 101 | [a, c] |
| 7 | 111 | [a, b, c] |

```js
function subsets(nums) {
  const n = nums.length, out = [];
  for (let mask = 0; mask < (1 << n); mask++) {
    const s = [];
    for (let i = 0; i < n; i++) if ((mask >> i) & 1) s.push(nums[i]);
    out.push(s);
  }
  return out;
}
```

**Complexity:** Time O(n * 2^n), Space O(n * 2^n) for the output.
**Watch out:** Only for `n <= 30` in JS (`1 << 31` is negative and the loop would never run). Real inputs are `n <= 20` anyway. For duplicates you need sorting plus a skip rule, which masks do not give you naturally. Same idea powers [bitmask DP](../../05-dynamic-programming/10-bitmask-dp/README.md).

### 14. Single Number II — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Every element appears three times except one, which appears once. Find it in O(n) time, O(1) space. Example: `[2, 2, 3, 2]` -> `3`.
**Intuition:** XOR no longer cancels triples. But look at one bit position at a time: the count of ones there is `3k` (from the triples) or `3k + 1` (if the single number has that bit). So `count % 3` is the single number's bit.
**Approach 1 - Brute force:** Hash map of counts. Time O(n), Space O(n).
**Approach 2 - Better (per-bit count):** For each of the 32 bits, count ones across `nums`, take `% 3`, set that bit in the answer. Time O(32 n), Space O(1).
**Approach 3 - Optimal (two-register state machine):** Each bit has three states (seen 0, 1, or 2 times mod 3). Two variables `ones` and `twos` hold those two state bits for all 32 positions at once. `ones` collects bits seen once, `twos` bits seen twice; a third sighting clears both.

Trace `[2, 2, 3, 2]`:

| x | ones | twos |
|---|------|------|
| 2 (10) | 10 | 00 |
| 2 | 00 | 10 |
| 3 (11) | 01 | 00 |
| 2 | 11 | 00 |

Answer `ones = 3`.

```js
function singleNumberII(nums) {
  let ones = 0, twos = 0;
  for (const x of nums) {
    ones = (ones ^ x) & ~twos; // add x to "seen once" unless already "seen twice"
    twos = (twos ^ x) & ~ones; // add x to "seen twice" unless it just became "once"
  }
  return ones;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Order of the two lines matters (`twos` uses the **new** `ones`). The per-bit counting version is easier to explain on a whiteboard and easier to get right; start there if the state machine feels hard. Negatives work because we use all 32 bits.

### 15. Single Number III — Medium
**Companies:** Amazon, Meta, Google
**Problem:** Exactly two numbers appear once, all others appear twice. Return the two. Example: `[1, 2, 1, 3, 2, 5]` -> `[3, 5]`.
**Intuition:** XOR everything: the pairs vanish, leaving `a ^ b`. Since `a != b`, that value has at least one 1 bit, and at that bit `a` and `b` differ. Use that bit to split the array into two groups, one holding `a` and the other `b`. Duplicates land in the same group as each other, so they still cancel. Then XOR each group.
**Approach 1 - Brute force / Better:** Hash map of counts. Time O(n), Space O(n).
**Approach 2 - Optimal:** XOR, isolate a differing bit with `x & -x`, split, XOR each group.

Trace: `xr = 3 ^ 5 = 110`. `low = 110 & -110 = 010` (value 2).
- Bit 1 set: `2, 3, 2` -> XOR = `3`.
- Bit 1 clear: `1, 1, 5` -> XOR = `5`.

```js
function singleNumberIII(nums) {
  let xr = 0;
  for (const v of nums) xr ^= v;
  const low = xr & -xr; // any set bit of xr works; the lowest is easiest
  let a = 0, b = 0;
  for (const v of nums) {
    if (v & low) a ^= v;
    else b ^= v;
  }
  return [a, b];
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** `xr & -xr` breaks for `xr = -2147483648` in other languages (overflow), but in JS it is fine. Test `[0, 1]` and `[-1, 0]`: a value of 0 never enters the `if` branch for `a`, which is fine because XOR with 0 changes nothing. The order of the two answers is not fixed.

### 16. Bitwise AND of Numbers Range — Medium
**Companies:** Amazon, Google
**Problem:** Return the AND of all integers in `[left, right]`. Example: `5..7` -> `4`.
**Intuition:** Counting up from `left` to `right` flips the low bits through every pattern, so any bit that changes anywhere in the range ANDs to 0. Only the **common prefix** of the two ends (in binary) survives, followed by zeros.
**Approach 1 - Brute force:** AND them all. Time O(right - left): too slow (up to 2^31).
**Approach 2 - Optimal:** Shift both right until they are equal; count the shifts; shift the result back left.

Trace `5 (101)` and `7 (111)`:

| step | left | right | shifts |
|------|------|-------|--------|
| start | 101 | 111 | 0 |
| 1 | 10 | 11 | 1 |
| 2 | 1 | 1 | 2 (equal, stop) |

Result: `1 << 2 = 4`.

(Alternative: `while (right > left) right &= right - 1; return right;`)

```js
function rangeBitwiseAnd(left, right) {
  let shift = 0;
  while (left !== right) {
    left >>= 1;
    right >>= 1;
    shift++;
  }
  return left << shift;
}
```

**Complexity:** Time O(32), Space O(1).
**Watch out:** `left = 0` gives 0 (the loop shifts until both are 0). Inputs up to `2^31 - 1` fit; `left << shift` cannot overflow because the prefix came from the same number.

### 17. Gray Code — Medium
**Companies:** Amazon, Google
**Problem:** Return a sequence of all `2^n` numbers from `0` to `2^n - 1` where neighbours differ in exactly one bit (and it starts at 0). Example: `n = 2` -> `[0, 1, 3, 2]`.
**Intuition:** There is a closed formula: the `i`-th Gray code is `i ^ (i >> 1)`. Moving from `i` to `i + 1` changes the binary number's trailing bits, but XOR with its own shift turns that into a single flipped bit.
**Approach 1 - Brute force:** Backtrack: from the current number, flip any one bit that leads to an unseen number. Time O(n * 2^n) with a visited set.
**Approach 2 - Better (reflect and prefix):** Gray(n) = Gray(n-1), then Gray(n-1) reversed with the new top bit set. Time O(2^n).
**Approach 3 - Optimal:** The formula.

| i | binary | i >> 1 | gray |
|---|--------|--------|------|
| 0 | 00 | 00 | 00 |
| 1 | 01 | 00 | 01 |
| 2 | 10 | 01 | 11 |
| 3 | 11 | 01 | 10 |

```js
function grayCode(n) {
  const out = [];
  for (let i = 0; i < (1 << n); i++) out.push(i ^ (i >> 1));
  return out;
}
```

**Complexity:** Time O(2^n), Space O(2^n) for the output.
**Watch out:** Many valid answers exist; judges accept any valid sequence. The last and first values also differ by one bit (a cycle). Tested for `n = 1, 3, 5`.

### 18. Divide Two Integers — Medium
**Companies:** Meta, Amazon, Google, Microsoft
**Problem:** Divide `dividend` by `divisor` without `*`, `/` or `%`, truncating toward zero. Clamp to the 32-bit range. Example: `10, 3` -> `3`; `7, -3` -> `-2`.
**Intuition:** Long division in binary. Any quotient is a sum of powers of two. Try the biggest shift first: if `divisor << i` still fits in what is left, subtract it and add `1 << i` to the answer.
**Approach 1 - Brute force:** Subtract the divisor repeatedly. Time O(dividend / divisor): up to 2^31 steps, too slow.
**Approach 2 - Optimal:** Work with positive magnitudes and go from bit 31 down to 0.

Trace `10 / 3`:

| i | `3 << i` | fits in remaining? | remaining | quotient |
|---|----------|--------------------|-----------|----------|
| 2 | 12 | no | 10 | 0 |
| 1 | 6 | yes | 4 | 2 |
| 0 | 3 | yes | 1 | 3 |

```js
function divide(dividend, divisor) {
  const MAX = 2147483647, MIN = -2147483648;
  if (dividend === MIN && divisor === -1) return MAX; // the only overflow case
  const negative = (dividend < 0) !== (divisor < 0);
  // BigInt: `<<` on plain numbers wraps at 32 bits and |MIN| = 2^31 does not fit
  let a = BigInt(Math.abs(dividend));
  const b = BigInt(Math.abs(divisor));
  let q = 0n;
  for (let i = 31n; i >= 0n; i--) {
    if ((b << i) <= a) {
      a -= b << i;
      q += 1n << i;
    }
  }
  return Number(negative ? -q : q);
}
```

**Complexity:** Time O(32), Space O(1).
**Watch out:** `MIN / -1` overflows (answer would be `2^31`), so clamp it. `Math.abs(MIN)` is fine in JS (it is a float) but would overflow in C++/Java; mention that. Truncation toward zero, not floor: `7 / -3 = -2`. Divisor of `0` is not allowed by the problem; say you would check it. We tested 500 random pairs against `Math.trunc(a / b)`.

### 19. Total Hamming Distance — Medium
**Companies:** Meta, Amazon
**Problem:** Sum of Hamming distances over all pairs in `nums`. Example: `[4, 14, 2]` -> `6`.
**Intuition:** Do not compare pairs. Look at one bit position at a time. If `ones` numbers have a 1 there and `zeros = n - ones` have a 0, every (1, 0) pair differs at that bit: `ones * zeros` pairs. Add over all 32 bits.
**Approach 1 - Brute force:** Check every pair with `popcount(a ^ b)`. Time O(n^2 * 32), Space O(1).
**Approach 2 - Optimal:** Per-bit counting.

Trace `[4 (0100), 14 (1110), 2 (0010)]`:

| bit | ones | zeros | pairs |
|-----|------|-------|-------|
| 0 | 0 | 3 | 0 |
| 1 | 2 | 1 | 2 |
| 2 | 2 | 1 | 2 |
| 3 | 1 | 2 | 2 |

Total `6`.

```js
function totalHammingDistance(nums) {
  let total = 0;
  for (let b = 0; b < 32; b++) {
    let ones = 0;
    for (const v of nums) ones += (v >> b) & 1;
    total += ones * (nums.length - ones);
  }
  return total;
}
```

**Complexity:** Time O(32 n), Space O(1).
**Watch out:** The answer can exceed 2^31 for big `n`, but a plain JS number holds it exactly up to 2^53, so no BigInt is needed. Do not use `|` or `<<` on the running total.

### 20. XOR Queries of a Subarray — Medium
**Companies:** Google, Amazon
**Problem:** Given `arr` and queries `[l, r]`, return `arr[l] ^ ... ^ arr[r]` for each. Example: `arr = [1, 3, 4, 8]`, query `[0, 1]` -> `2`, `[1, 2]` -> `7`.
**Intuition:** Same trick as prefix sums (see [07 - Prefix Sum](../../01-core-linear/07-prefix-sum/README.md)). XOR is its own inverse, so `prefix[r + 1] ^ prefix[l]` cancels everything before `l`.
**Approach 1 - Brute force:** XOR each range directly. Time O(n q).
**Approach 2 - Optimal:** Build prefix XORs once, answer each query in O(1).

Trace `arr = [1, 3, 4, 8]`: `prefix = [0, 1, 2, 6, 14]`. Query `[1, 2]`: `prefix[3] ^ prefix[1] = 6 ^ 1 = 7`.

```js
function xorQueries(arr, queries) {
  const p = [0];
  for (const v of arr) p.push(p[p.length - 1] ^ v);
  return queries.map(([l, r]) => p[r + 1] ^ p[l]);
}
```

**Complexity:** Time O(n + q), Space O(n).
**Watch out:** Off-by-one: prefix has `n + 1` entries, query uses `r + 1` and `l`. Same idea gives "count subarrays with XOR equal to K" (hash map of prefix XORs).

### 21. Maximum XOR of Two Numbers in an Array — Medium
**Companies:** Google, Amazon, Meta
**Problem:** Find the largest `nums[i] ^ nums[j]`. Example: `[3, 10, 5, 25, 2, 8]` -> `28` (`5 ^ 25`).
**Intuition:** To maximise XOR, you want the **highest** bit to differ first, then the next, and so on. Put numbers in a binary trie (bit 30 down to bit 0). For each number walk the trie, and at each level take the opposite bit if that branch exists.
**Approach 1 - Brute force:** Try every pair. Time O(n^2), Space O(1).
**Approach 2 - Better:** Build the answer bit by bit from the top, using a set of prefixes and checking `candidate ^ prefix` exists. Time O(32 n), Space O(n).
**Approach 3 - Optimal:** Binary trie, insert and query in one pass.

Trace for `5 (00101)` vs `25 (11001)`: bits 4, 3, 2 differ (5 has 0, 0, 1; 25 has 1, 1, 0) and bits 1, 0 match, so `5 ^ 25 = 11100 = 28`. The trie walk finds this by always preferring the opposite bit from the top.

```js
function findMaximumXOR(nums) {
  const root = [null, null]; // each node: [child for bit 0, child for bit 1]
  const insert = (x) => {
    let node = root;
    for (let i = 30; i >= 0; i--) { // bit 31 is the sign bit; inputs are non-negative
      const bit = (x >> i) & 1;
      if (!node[bit]) node[bit] = [null, null];
      node = node[bit];
    }
  };
  const query = (x) => {
    let node = root, best = 0;
    for (let i = 30; i >= 0; i--) {
      const bit = (x >> i) & 1, want = bit ^ 1;
      if (node[want]) { best |= 1 << i; node = node[want]; }
      else node = node[bit];
    }
    return best;
  };
  let answer = 0;
  insert(nums[0]);
  for (let i = 1; i < nums.length; i++) {
    answer = Math.max(answer, query(nums[i]));
    insert(nums[i]);
  }
  return answer;
}
```

**Complexity:** Time O(31 n), Space O(31 n) worst case for the trie.
**Watch out:** A single-element array returns 0 (the loop skips). Pairing a number with itself is never needed (XOR 0). The trie is the same structure as in [05 - Trie](../../03-trees/05-trie/README.md), with an alphabet of size 2. The "Maximum XOR With an Element From Array" variant adds sorted offline queries on top of this trie.

## Common Mistakes
- **"`>>` and `>>>` are the same."** `>>` copies the sign bit; `>>>` fills with zeros. Use `>>>` for unsigned work and for draining a possibly negative number.
- **"JS ints are 64-bit, so `1 << 40` works."** Bit ops use 32 bits and shift counts wrap mod 32: `1 << 40` is `256`. Use BigInt (`1n << 40n`) for large masks.
- **"`n & (n - 1) === 0` means power of two."** Operator precedence: `===` binds tighter than `&`. Write `(n & (n - 1)) === 0` and add `n > 0`.
- **"A result of `-2147483648` is a bug."** It may be a correct 32-bit signed value. Add `>>> 0` only when the problem says "unsigned".
- **"XOR trick works for any duplicates."** It needs an even count of the others. For three copies use per-bit counting mod 3.
- **"`x ^= y` swap is fine on array slots."** If both indices are the same, the value becomes 0.
- **"Bitwise ops mix with BigInt."** `5n & 1` throws. Convert with `BigInt(x)` or `Number(b)` on purpose.
- **"`~n` is `-n`."** It is `-n - 1`. `~0` is `-1`.
- **"Always faster than normal code."** On modern engines the gain is small; bits win on space and on elegance, not always on time. Do not obscure code when a plain version is clear.

## Interview Tips
- Say the **idea in one sentence first**: "XOR cancels pairs, so XOR-ing everything leaves the single one". Then code.
- Start with the simple version (hash map, sort, sum formula), state its cost, then say "we can get O(1) space with bits". This shows you know the trade-off.
- Mention the language limits: "JavaScript bitwise ops work on signed 32-bit integers, so I will use `>>> 0` where I need unsigned, and BigInt if values exceed 32 bits."
- Draw the binary of the example (4-8 bits). A tiny table beats a long speech.
- Edge cases to mention: `0`, negative numbers, `n = 1`, the single-element array, `INT_MIN`, and powers of two.
- Common follow-ups: "Do it without extra space", "What if the input is a stream", "What if numbers can be 64-bit" (BigInt or split into two 32-bit halves), "Call it a million times" (lookup tables for bytes), "Why does `n & -n` work" (two's complement).
- Know popcount tools: the loop `n &= n - 1`, a lookup table, and the idea that hardware has a popcount instruction (JS does not expose it directly; `Math.clz32` counts leading zeros).
- If asked for a clever trick you do not remember, derive it from the operators: small examples in binary show the pattern.

## Practice List

**Easy**
- Number of 1 Bits
- Power of Two
- Power of Four
- Missing Number
- Single Number
- Hamming Distance
- Counting Bits
- Reverse Bits
- Binary Number with Alternating Bits
- Number Complement
- Add Binary
- Minimum Bit Flips to Convert Number
- Find the Difference
- Sum of Two Integers (as a warm-up variant)

**Medium**
- Single Number II
- Single Number III
- Bitwise AND of Numbers Range
- Subsets
- Gray Code
- Divide Two Integers
- Total Hamming Distance
- XOR Queries of a Subarray
- Maximum XOR of Two Numbers in an Array
- Sum of Two Integers
- Count Triplets That Can Form Two Arrays of Equal XOR
- Decode XORed Array
- UTF-8 Validation
- Maximum Product of Word Lengths
- Count Subarrays with XOR Equal to K (TUF classic)

**Hard**
- Maximum XOR With an Element From Array
- Minimum One Bit Operations to Make Integers Zero
- Smallest Sufficient Team (bitmask DP)
- Shortest Path Visiting All Nodes (bitmask BFS)
- Number of Valid Words for Each Puzzle
- Partition to K Equal Sum Subsets (bitmask DP version)

## Key Takeaways
- **XOR cancels equal pairs**: `x ^ x = 0`, `x ^ 0 = x`, order does not matter. This one fact solves Single Number, Missing Number and Hamming-type problems.
- **`n & (n - 1)` clears the lowest set bit**: use it for popcount and the power-of-two test.
- **`n & -n` isolates the lowest set bit**: it is the key to splitting groups in Single Number III.
- **Two's complement** means `-n = ~n + 1`; the top bit is the sign.
- **JavaScript bit operators are 32-bit signed**: use `>>> 0` for unsigned, `>>>` to drain negatives, and BigInt past 32 bits.
- **Think per bit position** when pairs are hard: Total Hamming Distance and Single Number II treat each of the 32 columns separately.
- **A bitmask is a set**: `0..2^n - 1` enumerates all subsets, and the same idea leads to bitmask DP.

---
[← Previous: Intervals and Sweep Line](../06-intervals-and-sweep-line/README.md) | [Next: Matrix →](../08-matrix/README.md) | [Back to Intermediate Index](../README.md)
