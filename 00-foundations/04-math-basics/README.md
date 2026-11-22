# Math Basics for Interviews

> **Estimated Time:** 5 hr | **Difficulty:** Beginner
> **Prerequisites:** [03 - JavaScript for DSA](../03-javascript-for-dsa/README.md)

---

## What You'll Learn

- You can pull a number apart digit by digit with `% 10` and `Math.floor(n / 10)`, and rebuild it safely.
- You can find divisors, test primes and factor numbers in `O(sqrt n)`, and list many primes with a sieve.
- You can compute GCD with Euclid and powers with fast exponentiation (`O(log n)`).
- You can do modular arithmetic without overflow in JavaScript, using `BigInt` when needed.
- You can handle the classic "number tricks": trailing zeros of a factorial, Pascal's triangle, happy numbers, Excel column titles.

## The Big Idea

Think of a cash register. It never counts coins one by one when it can do the sum in a few steps. It splits the bill into hundreds, tens and ones. Math problems in interviews are the same: instead of looping a billion times, you find a rule that skips most of the work.

Most of these problems have an easy brute-force answer and a clever short answer. The clever answer usually comes from one of four ideas:

1. **Digits:** `n % 10` gives the last digit, `Math.floor(n / 10)` removes it.
2. **Pairs:** divisors come in pairs `(i, n / i)`, so you only look up to `sqrt(n)`.
3. **Halving:** to compute a huge power, square the base and halve the exponent. That is `O(log n)` instead of `O(n)`.
4. **Reduction:** `gcd(a, b) = gcd(b, a % b)` shrinks the numbers very fast.

Interviewers love these because the code is short, but there are many edge cases: zero, negatives, overflow, `n = 1`. They test whether you think before you type.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "digits of a number", "reverse", "palindrome number" | `% 10` and `/ 10` loop, no string needed |
| "divisors", "factors" | loop `i` up to `sqrt(n)`, take pairs |
| "is it prime" | trial division up to `sqrt(n)` |
| "count primes below n", n up to millions | Sieve of Eratosthenes |
| "prime factors", "largest prime factor" | divide out each factor, loop to `sqrt(n)` |
| "common multiple", "common divisor", "fractions" | Euclid GCD, `lcm = a / gcd * b` |
| `x^n` with huge `n` | fast exponentiation (binary exponentiation) |
| "square root without `Math.sqrt`" | binary search on the answer |
| "return answer modulo 1e9+7" | reduce after every multiply, `BigInt` if products pass 2^53 |
| "trailing zeros of n!" | count factors of 5 |
| "row of Pascal's triangle", "nCr" | each cell = sum of the two above, or multiplicative formula |
| "repeat until 1 or loops forever" | cycle detection (fast and slow pointers) |
| "column title", "base 26" | base conversion, but 1-based, so subtract 1 first |

## Core Concepts / Template

### 1. Digit extraction

```js
// visit every digit of n (n >= 0), last digit first
while (n > 0) {
  const digit = n % 10;      // last digit
  n = Math.floor(n / 10);    // drop it
}
```

Use `Math.floor` for non-negative numbers. In JavaScript `/` gives a decimal, not an integer division. For negatives use `Math.trunc` (it rounds toward zero, like C and Java).

### 2. Loop to the square root

If `i` divides `n`, so does `n / i`. One of the two is always `<= sqrt(n)`. So you never need to check past `sqrt(n)`.

```js
for (let i = 1; i * i <= n; i++) { /* i and n / i are a pair */ }
```

Write `i * i <= n`, not `i <= Math.sqrt(n)`. It avoids floating point surprises and is fast.

### 3. Safe integers in JavaScript

JavaScript numbers are 64-bit doubles. Integers are exact only up to `Number.MAX_SAFE_INTEGER = 2^53 - 1` (about 9.0e15). This matters a lot:

| Situation | What to do |
|---|---|
| Result stays below 2^53 | plain `number` is fine |
| Product of two numbers near 1e9+7 (about 1e18) | exceeds 2^53, use `BigInt` |
| Problem says "32-bit signed int" (range -2^31 .. 2^31-1) | JS will NOT overflow for you; compare with the limits yourself |
| Bit operators `\|`, `&`, `>>`, `<<` | they silently cut numbers to 32 bits, avoid them for big values |

### 4. Euclid and fast power: the two big shortcuts

- Euclid: `gcd(a, b) = gcd(b, a % b)`, stop when `b = 0`.
- Binary exponentiation: `x^n = (x^2)^(n/2)` when `n` is even; extra `x` when `n` is odd.

Both run in `O(log)` steps because the numbers at least halve every one or two steps.

### 5. Short trace: digits of 472

| step | n | digit = n % 10 | new n |
|---|---|---|---|
| 1 | 472 | 2 | 47 |
| 2 | 47 | 7 | 4 |
| 3 | 4 | 4 | 0 |

Loop stops at `n = 0`. Three digits seen: 2, 7, 4.

## Problems (easy -> hard)

### 1. Count Digits — Easy
**Companies:** -
**Problem:** Given an integer `n`, return how many digits it has. Example: `12345 -> 5`, `0 -> 1`, `-908 -> 3`.
**Intuition:** Every `/ 10` removes one digit. Count how many times you can do it before reaching 0.
**Approach 1 - Brute force:** `String(Math.abs(n)).length`. Time O(d), Space O(d), where d is the digit count. Fine in real life, but interviewers often ask for no strings.
**Approach 2 - Optimal:** Loop with `Math.floor(n / 10)` and count. Or use `Math.floor(Math.log10(n)) + 1` for `n > 0` (a one-liner, but floating point can be off for huge values, so mention the risk).

| n | step | count |
|---|---|---|
| 908 | 908 -> 90 | 1 |
| 90 | 90 -> 9 | 2 |
| 9 | 9 -> 0 | 3 |

```js
function countDigits(n) {
  n = Math.abs(n);
  if (n === 0) return 1; // 0 has one digit, the loop would give 0
  let c = 0;
  while (n > 0) { c++; n = Math.floor(n / 10); }
  return c;
}
```

**Complexity:** Time O(log10 n), Space O(1).
**Watch out:** `0` must return 1. Negative numbers: take `Math.abs` first (otherwise the loop never runs).

### 2. Palindrome Number — Easy
**Companies:** Amazon, Google, Microsoft, Adobe
**Problem:** Return true if an integer reads the same forward and backward. Example: `121 -> true`, `-121 -> false`, `10 -> false`.
**Intuition:** You do not need to reverse the whole number. Reverse only the second half and compare it with the first half. When the reversed half grows bigger than what is left, you have passed the middle.
**Approach 1 - Brute force:** Convert to a string and compare with its reverse. Time O(d), Space O(d).
**Approach 2 - Better:** Reverse the whole number and compare. Works, but the reversed value can overflow in languages with 32-bit ints. In JS it is fine, but half-reversal is cleaner.
**Approach 3 - Optimal:** Build `half` from the last digits while `x > half`. At the end, an even-length number has `x === half`; an odd-length number has `x === floor(half / 10)` (the middle digit is dropped).

Trace for `12321`:

| x | half |
|---|---|
| 12321 | 0 |
| 1232 | 1 |
| 123 | 12 |
| 12 | 123 (stop, x < half) |

`x = 12`, `floor(123 / 10) = 12`. Equal, so true.

```js
function isPalindrome(x) {
  // negatives have a '-', numbers ending in 0 can't start with 0 (except 0 itself)
  if (x < 0 || (x % 10 === 0 && x !== 0)) return false;
  let half = 0;
  while (x > half) { half = half * 10 + (x % 10); x = Math.floor(x / 10); }
  return x === half || x === Math.floor(half / 10);
}
```

**Complexity:** Time O(log10 x), Space O(1).
**Watch out:** the `x % 10 === 0` guard. Without it, `10` becomes `x=1, half=1` and wrongly returns true.

### 3. Armstrong Number — Easy
**Companies:** -
**Problem:** A number is an Armstrong number if the sum of each digit raised to the power of the digit count equals the number. Example: `153 = 1^3 + 5^3 + 3^3 -> true`, `10 -> false`.
**Intuition:** Count digits first (that is the power `k`), then walk the digits again and add `digit^k`.
**Approach 1 - Brute force:** Use strings: `[...String(n)].reduce(...)`. Time O(d), Space O(d).
**Approach 2 - Optimal:** Get `k` with `String(n).length` (or the digit loop), then sum with `% 10`. Dry run on `153`: `k = 3`, digits 3, 5, 1 give `27 + 125 + 1 = 153`. Equal, so true.

```js
function isArmstrong(n) {
  const k = String(n).length;
  let sum = 0, t = n;
  while (t > 0) { sum += (t % 10) ** k; t = Math.floor(t / 10); }
  return sum === n;
}
```

**Complexity:** Time O(d), Space O(1) (the string is only used for its length).
**Watch out:** the power must be the digit count, not always 3. `9474 = 9^4 + 4^4 + 7^4 + 4^4`. Single digits (and 0) are Armstrong numbers by this definition.

### 4. Print All Divisors — Easy
**Companies:** -
**Problem:** Return all divisors of `n` in increasing order. Example: `36 -> [1, 2, 3, 4, 6, 9, 12, 18, 36]`.
**Intuition:** Divisors pair up: `1 x 36`, `2 x 18`, `3 x 12`, `4 x 9`, `6 x 6`. Find the small half by looping to `sqrt(n)`; the large half comes free as `n / i`.
**Approach 1 - Brute force:** Try every `i` from 1 to `n`. Time O(n), Space O(1) extra.
**Approach 2 - Optimal:** Loop `i * i <= n`. When `n % i === 0`, `i` is a small divisor and `n / i` a large one. Skip the duplicate when `i === n / i` (perfect squares). Collect large ones separately, then reverse them to keep order.

```js
function divisors(n) {
  const small = [], large = [];
  for (let i = 1; i * i <= n; i++) {
    if (n % i === 0) {
      small.push(i);
      if (i !== n / i) large.push(n / i);
    }
  }
  return small.concat(large.reverse());
}
```

**Complexity:** Time O(sqrt n), Space O(number of divisors).
**Watch out:** for `36`, `i = 6` must be added once, not twice. A number has at most a few thousand divisors for `n` up to 1e9, so the output is small.

### 5. Check Prime — Easy
**Companies:** Amazon, Microsoft
**Problem:** Return true if `n` is prime (only divisors are 1 and itself). Example: `97 -> true`, `91 -> false` (7 x 13), `1 -> false`.
**Intuition:** If `n` has a divisor other than 1 and `n`, then it has one `<= sqrt(n)`. So test only up to there.
**Approach 1 - Brute force:** Test `2 .. n-1`. Time O(n).
**Approach 2 - Better:** Test `2 .. sqrt(n)`. Time O(sqrt n).
**Approach 3 - Optimal (practical):** Every prime above 3 has the form `6k +- 1`. Remove 2 and 3 first, then test `i` and `i + 2` for `i = 5, 11, 17, ...`. About 3 times fewer checks, same big-O.

```js
function isPrime(n) {
  if (n < 2) return false;
  if (n < 4) return true;               // 2 and 3
  if (n % 2 === 0 || n % 3 === 0) return false;
  for (let i = 5; i * i <= n; i += 6) {
    if (n % i === 0 || n % (i + 2) === 0) return false;
  }
  return true;
}
```

**Complexity:** Time O(sqrt n), Space O(1).
**Watch out:** 0 and 1 are not prime. 2 is the only even prime. Use this when you test a few numbers; use a sieve when you need many (next problem).

### 6. Count Primes (Sieve of Eratosthenes) — Medium
**Companies:** Amazon, Microsoft, Apple, Google
**Problem:** Count primes strictly less than `n`. Example: `n = 10 -> 4` (2, 3, 5, 7).
**Intuition:** Instead of testing each number, cross out the multiples of each prime. Whatever is never crossed out is prime. It is like a kid crossing out table 2, then table 3, then table 5 on a hundred-chart.
**Approach 1 - Brute force:** Run `isPrime` on each number below `n`. Time O(n sqrt n).
**Approach 2 - Optimal:** Sieve. Keep an array of marks. For each `i` from 2 that is not marked, count it and mark `i*i, i*i + i, ...`. Start at `i*i` because smaller multiples like `2i, 3i` were already marked by smaller primes.

Trace for `n = 20` (marks added, in order):

| prime i | marks |
|---|---|
| 2 | 4 6 8 10 12 14 16 18 |
| 3 | 9 15 |
| 5 | 25 is out of range, nothing |

Unmarked from 2: 2, 3, 5, 7, 11, 13, 17, 19 = 8 primes.

```js
function countPrimes(n) { // primes strictly less than n
  if (n < 3) return 0;
  const composite = new Uint8Array(n); // compact, starts all 0
  let count = 0;
  for (let i = 2; i < n; i++) {
    if (composite[i]) continue;
    count++;
    for (let j = i * i; j < n; j += i) composite[j] = 1;
  }
  return count;
}
```

**Complexity:** Time O(n log log n), Space O(n).
**Watch out:** "less than n", not "up to n". `i * i` can pass 2^53 only for `i` near 1e8, far beyond what memory allows, so it is safe here. For `n` around 1e7 this runs in well under a second in Node.

### 7. Prime Factorization — Medium
**Companies:** -
**Problem:** Return the prime factors of `n` with their powers. Example: `360 -> [[2,3],[3,2],[5,1]]` because `360 = 2^3 x 3^2 x 5`.
**Intuition:** Divide out the smallest factor again and again. After removing all 2s, all 3s and so on, what is left (if above 1) must be one big prime.
**Approach 1 - Brute force:** For every `i` up to `n`, test `i` for primality and divisibility. Slow.
**Approach 2 - Optimal:** Loop `p` from 2 while `p * p <= n`. If `p` divides `n`, divide fully and count the power. Because we always remove small factors first, `p` is always prime when it divides. After the loop, if `n > 1`, it is a leftover prime.

Trace for `360`:

| p | n before | action | n after |
|---|---|---|---|
| 2 | 360 | /2 x3 | 45 |
| 3 | 45 | /3 x2 | 5 |
| 4 | 5 | 16 > 5, stop | 5 |

Leftover `5 > 1`, so add `[5, 1]`.

```js
function primeFactors(n) { // returns [[prime, exponent], ...]
  const res = [];
  for (let p = 2; p * p <= n; p++) {
    if (n % p === 0) {
      let e = 0;
      while (n % p === 0) { n /= p; e++; }
      res.push([p, e]);
    }
  }
  if (n > 1) res.push([n, 1]); // one prime bigger than sqrt(original n)
  return res;
}
```

**Complexity:** Time O(sqrt n), Space O(log n) for the output.
**Watch out:** forgetting the leftover prime (`n = 14` gives factor 2 and leftover 7). Note `p * p <= n` uses the shrinking `n`, which is a nice speedup. For many queries on numbers up to 1e6, precompute a smallest-prime-factor array with a sieve (see [04 - Number Theory](../../06-advanced/04-number-theory/README.md)).

### 8. GCD and LCM (Euclid) — Easy
**Companies:** Amazon, Google, Microsoft
**Problem:** Return the greatest common divisor and the least common multiple of two non-negative integers. Example: `gcd(12, 18) = 6`, `lcm(4, 6) = 12`.
**Intuition:** Any number that divides both `a` and `b` also divides `a % b` (the remainder). So swap the problem for smaller numbers until the remainder is 0.
**Approach 1 - Brute force:** Try `min(a, b)` down to 1, return the first number that divides both. Time O(min(a, b)).
**Approach 2 - Optimal:** Euclid. Time O(log(min(a, b))). LCM follows from `a * b = gcd * lcm`.

Trace `gcd(48, 18)`:

| a | b | a % b |
|---|---|---|
| 48 | 18 | 12 |
| 18 | 12 | 6 |
| 12 | 6 | 0 |
| 6 | 0 | done, answer 6 |

```js
function gcd(a, b) { while (b !== 0) [a, b] = [b, a % b]; return a; }
// divide first, then multiply: smaller intermediate value, less overflow risk
function lcm(a, b) { return a === 0 || b === 0 ? 0 : (a / gcd(a, b)) * b; }
```

**Complexity:** Time O(log(min(a, b))), Space O(1).
**Watch out:** `gcd(a, 0) = a`. Compute `a / gcd * b`, not `a * b / gcd`, so the product does not grow too big. For many numbers, fold: `reduce(gcd)`. A small bonus: `gcd` also solves "make two jugs measure z liters" and "simplify a fraction".

### 9. Reverse Integer (Overflow) — Medium
**Companies:** Amazon, Google, Apple, Microsoft, Adobe
**Problem:** Reverse the digits of a signed 32-bit integer. If the result is outside `[-2^31, 2^31 - 1]`, return 0. Example: `123 -> 321`, `-123 -> -321`, `120 -> 21`, `1534236469 -> 0`.
**Intuition:** Pop the last digit and push it onto the answer. The only danger is the answer getting too big. The original problem assumes you cannot store 64-bit values, so interviewers test whether you check overflow before it happens.
**Approach 1 - Brute force:** Convert to string, reverse, parse, then range-check. Time O(d), Space O(d).
**Approach 2 - Optimal:** Digit loop. In JavaScript, `%` keeps the sign (`-123 % 10 = -3`) and `Math.trunc` rounds toward zero, so negatives work with no special case. Because JS uses doubles, `rev * 10 + d` is exact here (each step we stop as soon as `rev` leaves the 32-bit range, so it never gets near 2^53). So we can check after each step.

In C++ or Java you would check before multiplying: `rev > MAX / 10 || (rev === MAX / 10 && d > 7)`. Say this out loud; it shows you know the real trick.

Trace for `1534236469` (digits popped: 9, 6, 4, 6, 3, 2, 4, 3, 5, 1):

| popped | rev |
|---|---|
| 9 | 9 |
| 6 | 96 |
| ... | ... |
| 5 | 964632435 |
| 1 | 9646324351 > 2147483647, return 0 |

```js
function reverse(x) {
  const MAX = 2 ** 31 - 1, MIN = -(2 ** 31);
  let rev = 0;
  while (x !== 0) {
    const d = x % 10;        // keeps the sign of x
    x = Math.trunc(x / 10);
    rev = rev * 10 + d;
    if (rev > MAX || rev < MIN) return 0;
  }
  return rev;
}
```

**Complexity:** Time O(log10 |x|), Space O(1).
**Watch out:** `Math.floor` on a negative number rounds the wrong way (`-1.2 -> -2`), use `Math.trunc`. `-2147483648` reversed is `-8463847412`, out of range, so 0. Trailing zeros vanish (`120 -> 21`) automatically.

### 10. Pow(x, n) (Fast Exponentiation) — Medium
**Companies:** Amazon, Google, Meta, Microsoft, LinkedIn
**Problem:** Compute `x` raised to the integer power `n` (`n` can be negative). Example: `2^10 -> 1024`, `2^-2 -> 0.25`.
**Intuition:** `2^10 = (2^2)^5`, and `2^5 = 2 * (2^2)^2`. Each step halves `n`. That is only about 31 steps for `n` near 2^31, not 2 billion.
**Approach 1 - Brute force:** Multiply `x` by itself `n` times. Time O(n). Too slow and, with recursion, a stack overflow.
**Approach 2 - Optimal:** Iterative binary exponentiation. Look at `n` in binary: for each 1 bit, multiply the answer by the current square.
- If `n` is odd: `res *= x`.
- Always: `x *= x`, `n = floor(n / 2)`.
- For negative `n`: use `1 / x` and `-n`.

Trace `x = 2, n = 10` (binary 1010):

| n | n odd? | res | x after squaring |
|---|---|---|---|
| 10 | no | 1 | 4 |
| 5 | yes | 4 | 16 |
| 2 | no | 4 | 256 |
| 1 | yes | 1024 | 65536 |

```js
function myPow(x, n) {
  if (n < 0) { x = 1 / x; n = -n; } // JS numbers are doubles, so -(-2^31) is safe
  let res = 1;
  while (n > 0) {
    if (n % 2 === 1) res *= x;
    x *= x;
    n = Math.floor(n / 2);
  }
  return res;
}
```

**Complexity:** Time O(log n), Space O(1) (a recursive version uses O(log n) stack).
**Watch out:** in languages with 32-bit ints, `-n` overflows when `n = -2^31`. In JS it does not, but mention it. `x = 0` with a negative `n` is undefined (infinite); the problem usually avoids it. Do not use `n >> 1` for large `n` because bit operators cut to 32 bits.

### 11. Sqrt(x) (Binary Search) — Easy
**Companies:** Amazon, Google, Meta, Microsoft, Apple
**Problem:** Return the integer part of the square root of a non-negative integer, without `Math.sqrt` or `**0.5`. Example: `8 -> 2`, `16 -> 4`.
**Intuition:** The answer lies in `[0, x]`. `mid * mid` is increasing with `mid`, so it is a sorted yes/no question: "is `mid * mid <= x`?" Keep the biggest `mid` that says yes.
**Approach 1 - Brute force:** Try `i = 1, 2, 3 ...` until `i * i > x`. Time O(sqrt x).
**Approach 2 - Optimal:** Binary search over the answer. Time O(log x).

Trace for `x = 8` (`lo = 0, hi = 8`):

| lo | hi | mid | mid^2 | action |
|---|---|---|---|---|
| 0 | 8 | 4 | 16 | too big, hi = 3 |
| 0 | 3 | 1 | 1 | ok, ans = 1, lo = 2 |
| 2 | 3 | 2 | 4 | ok, ans = 2, lo = 3 |
| 3 | 3 | 3 | 9 | too big, hi = 2 |

Stop; answer 2.

```js
function mySqrt(x) {
  let lo = 0, hi = x, ans = 0;
  while (lo <= hi) {
    const mid = lo + Math.floor((hi - lo) / 2);
    if (mid * mid <= x) { ans = mid; lo = mid + 1; } // mid works, try bigger
    else hi = mid - 1;
  }
  return ans;
}
```

**Complexity:** Time O(log x), Space O(1).
**Watch out:** in Java or C++, `mid * mid` overflows for `x` near 2^31, so people write `mid <= x / mid`. In JS, `mid * mid` is a double, so there is no wraparound; for huge `mid` it may be rounded, but it is then far above `x`, so the comparison is still right. Mention the overflow issue anyway. See [08 - Binary Search](../../01-core-linear/08-binary-search/README.md) for "binary search on the answer".

### 12. Modular Arithmetic Basics (Fast Power and Inverse) — Medium
**Companies:** -
**Problem:** Many problems say "return the answer modulo 1e9 + 7". Compute `(base^exp) mod m` fast, and divide modulo a prime. Example: `modPow(2, 10, 1000) = 24`.
**Intuition:** Think of a clock. After 12 you wrap to 1. Adding, subtracting and multiplying all work on the wrapped numbers, so you can wrap after every step and keep numbers small:
- `(a + b) % m = ((a % m) + (b % m)) % m`
- `(a * b) % m = ((a % m) * (b % m)) % m`
- Subtraction: `((a - b) % m + m) % m` (JS `%` can return a negative number).
- Division is different: `a / b` mod `p` (prime `p`) is `a * b^(p-2) mod p`. This is the modular inverse, by Fermat's little theorem.

**Approach 1 - Brute force:** Multiply `exp` times, taking `% m` each time. Time O(exp).
**Approach 2 - Optimal:** Fast exponentiation (as in Pow(x, n)) with `% m` after each multiply. Time O(log exp).

The JavaScript trap: with `m = 1e9 + 7`, a product of two residues is up to about 1e18, which is above 2^53 (about 9e15). A normal `number` silently loses digits. Use `BigInt` for the multiply.

```js
function modPow(base, exp, mod) { // all BigInt
  base = ((base % mod) + mod) % mod; // make non-negative
  let res = 1n;
  while (exp > 0n) {
    if (exp & 1n) res = (res * base) % mod;
    base = (base * base) % mod;
    exp >>= 1n;
  }
  return res;
}
// inverse of a modulo a PRIME p
const modInverse = (a, p) => modPow(a, p - 2n, p);
// modPow(2n, 10n, 1000n) -> 24n
// (modInverse(3n, 1000000007n) * 3n) % 1000000007n -> 1n
```

**Complexity:** Time O(log exp), Space O(1).
**Watch out:** `BigInt` and `number` cannot be mixed (`2n * 3` throws). The inverse trick works only when the modulus is prime and `a` is not a multiple of it. A cheaper trick for sums: if each term is below `m`, a plain `number` addition is fine, only multiplication needs `BigInt`. `BigInt` is slower, so use it only where a product can pass 2^53.

### 13. Factorial Trailing Zeroes — Medium
**Companies:** Amazon, Bloomberg, Google
**Problem:** Return how many zeros are at the end of `n!`. Example: `5! = 120 -> 1`, `25! -> 6`.
**Intuition:** A trailing zero comes from a factor `10 = 2 x 5`. There are always more 2s than 5s in `n!`, so the count of 5s decides. Numbers 5, 10, 15 ... give one 5 each; 25, 50 ... give a second one; 125 gives a third.
**Approach 1 - Brute force:** Compute `n!` and count zeros. Needs `BigInt`, huge. Time very large.
**Approach 2 - Better:** For each `i` in `1..n`, count how many times 5 divides it. Time O(n log n).
**Approach 3 - Optimal:** Sum `floor(n/5) + floor(n/25) + floor(n/125) + ...`. Repeatedly do `n = floor(n / 5)` and add.

Trace `n = 100`:

| n after / 5 | added | total |
|---|---|---|
| 20 | 20 | 20 |
| 4 | 4 | 24 |
| 0 | stop | 24 |

```js
function trailingZeroes(n) {
  let c = 0;
  while (n >= 5) { n = Math.floor(n / 5); c += n; }
  return c;
}
```

**Complexity:** Time O(log5 n), Space O(1).
**Watch out:** do not stop after `n / 5`; 25 contributes two 5s. Same idea answers "highest power of prime `p` dividing `n!`" (Legendre's formula).

### 14. Pascal's Triangle — Easy
**Companies:** Amazon, Google, Apple, Adobe, Microsoft
**Problem:** Return the first `numRows` rows of Pascal's triangle. Example: `5 -> [[1],[1,1],[1,2,1],[1,3,3,1],[1,4,6,4,1]]`.
**Intuition:** Every inner number is the sum of the two numbers above it. The edges are always 1. The value at row `r`, column `c` (0-based) is `C(r, c)`, "r choose c".
**Approach 1 - Brute force:** Compute each cell as `C(r, c)` with factorials. Costly and risks overflow.
**Approach 2 - Optimal:** Build row by row from the previous row.

```
row 0:       1
row 1:      1 1
row 2:     1 2 1
row 3:    1 3 3 1
row 4:   1 4 6 4 1
```

Row 4 inner values: `1+3 = 4`, `3+3 = 6`, `3+1 = 4`.

```js
function generate(numRows) {
  const rows = [];
  for (let i = 0; i < numRows; i++) {
    const row = new Array(i + 1).fill(1);              // edges are 1
    for (let j = 1; j < i; j++) row[j] = rows[i - 1][j - 1] + rows[i - 1][j];
    rows.push(row);
  }
  return rows;
}

// one cell, or one value C(n, r), without building the triangle
function nCr(n, r) {
  r = Math.min(r, n - r);
  let res = 1;
  for (let i = 1; i <= r; i++) res = (res * (n - r + i)) / i; // stays an integer each step
  return res;
}
```

**Complexity:** Time O(numRows^2), Space O(numRows^2) for the output. `nCr`: Time O(r), Space O(1).
**Watch out:** rows 0 and 1 have no inner cells, the inner loop must not run. In `nCr`, multiply before dividing; the running product of `i` consecutive numbers is always divisible by `i`, so no fraction appears. Common follow-ups: "return only row k" (keep one array, update right to left) and "find the value at (r, c)" (`nCr(r - 1, c - 1)` for 1-based).

### 15. Happy Number — Easy
**Companies:** Amazon, Google, Apple, Airbnb
**Problem:** Replace a number by the sum of the squares of its digits, repeat. It is happy if it reaches 1; otherwise it loops forever. Example: `19 -> 1^2 + 9^2 = 82 -> 68 -> 100 -> 1`, true. `2 -> 4 -> 16 -> 37 -> 58 -> 89 -> 145 -> 42 -> 20 -> 4 ...`, false.
**Intuition:** The sequence either hits 1 or falls into a cycle. A big number quickly shrinks (a 10-digit number maps to at most `10 x 81 = 810`), so the path cannot grow forever. "Does it repeat?" is exactly the linked-list cycle question.
**Approach 1 - Brute force / Better:** Keep a `Set` of seen values; if you see a value twice, return false. Time O(log n) per step with a small number of steps, Space O(steps).
**Approach 2 - Optimal:** Floyd's cycle detection: a slow pointer moves one step, a fast pointer two. If `fast` reaches 1 the answer is true; if they meet at some other value, there is a cycle. No extra memory.

Trace for `n = 2`. The chain is 2, 4, 16, 37, 58, 89, 145, 42, 20, 4, ... (slow moves 1 step, fast moves 2):

| round | slow | fast |
|---|---|---|
| start | 2 | 4 |
| 1 | 4 | 37 |
| 2 | 16 | 89 |
| 3 | 37 | 42 |
| 4 | 58 | 4 |
| 5 | 89 | 37 |
| 6 | 145 | 89 |
| 7 | 42 | 42 |

They meet at 42, not at 1, so there is a cycle and the answer is false.

```js
function isHappy(n) {
  const next = (x) => {
    let s = 0;
    while (x > 0) { const d = x % 10; s += d * d; x = Math.floor(x / 10); }
    return s;
  };
  let slow = n, fast = next(n);
  while (fast !== 1 && slow !== fast) { slow = next(slow); fast = next(next(fast)); }
  return fast === 1;
}
```

**Complexity:** Time O(log n) (the sequence collapses to small numbers fast), Space O(1).
**Watch out:** `n = 1` is happy at once: `fast = next(1) = 1`. Start `fast` one step ahead so the first comparison is not trivially equal. See [09 - Linked List](../../01-core-linear/09-linked-list/README.md) for Floyd's cycle detection.

### 16. Excel Sheet Column Title — Medium
**Companies:** Amazon, Microsoft, Google, Facebook
**Problem:** Convert a positive integer to its Excel column title. `1 -> A`, `26 -> Z`, `27 -> AA`, `28 -> AB`, `701 -> ZY`, `703 -> AAA`.
**Intuition:** It is base 26, but with a twist. There is no zero digit: the digits are A..Z = 1..26. Normal base conversion needs digits 0..25. Fix it by subtracting 1 before every step, so A..Z become 0..25. The reverse problem (title to number) is a plain `ans = ans * 26 + (letter - 'A' + 1)`.
**Approach 1 - Brute force:** Not natural here; one could count up from "A" using a string increment. Slow for big `n`.
**Approach 2 - Optimal:** Repeat: `n--`, take `n % 26` as the letter, prepend it, `n = floor(n / 26)`.

Trace for `n = 701`:

| n | n-1 | letter = (n-1) % 26 | next n = floor((n-1) / 26) |
|---|---|---|---|
| 701 | 700 | 24 = Y | 26 |
| 26 | 25 | 25 = Z | 0 |

Letters prepended: Y then Z gives "ZY".

```js
function convertToTitle(n) {
  let s = '';
  while (n > 0) {
    n--; // 1-based -> 0-based so A=0 ... Z=25
    s = String.fromCharCode(65 + (n % 26)) + s; // 65 is 'A'
    n = Math.floor(n / 26);
  }
  return s;
}
```

**Complexity:** Time O(log26 n), Space O(log26 n).
**Watch out:** forgetting `n--` makes 26 come out as "A@" or similar garbage. Prepending to a string in a loop is fine here since the title has at most 7 letters for 32-bit input.

## Common Mistakes

- **"`/` is integer division."** In JavaScript `7 / 2 = 3.5`. Use `Math.floor` (non-negative) or `Math.trunc` (any sign).
- **"`%` is always non-negative."** `-7 % 5` is `-2` in JS. For modular arithmetic use `((a % m) + m) % m`.
- **"JS integers overflow at 2^31."** They do not. JS numbers stay exact up to 2^53 - 1. If the problem says 32-bit, you must check the range yourself. And beyond 2^53 you need `BigInt`.
- **"Bit shifts are a fast divide by 2."** `n >> 1` and `n | 0` convert to 32-bit first and give wrong results for big values. Use `Math.floor(n / 2)`.
- **"Loop `i <= Math.sqrt(n)`."** It works but invites floating point doubts. Prefer `i * i <= n`.
- **"1 is prime."** It is not. Also remember 0, negatives and 2 as special cases.
- **"The sieve starts at `2i`."** It can start at `i * i`; smaller multiples were already crossed out. Starting at `2i` is correct but slower.
- **"`a * b / gcd` is fine for LCM."** The product can be huge. Divide first: `a / gcd * b`.
- **"Use `BigInt` everywhere to be safe."** It is slower and cannot mix with `number`. Use it only where a product can pass 2^53.
- **"Excel titles are plain base 26."** There is no zero digit, so subtract 1 before taking `% 26`.

## Interview Tips

- Before coding, ask: range of `n`? Can it be negative or zero? Is overflow a concern (32-bit or 64-bit)? These questions are half of what the interviewer is grading.
- Say the brute force first, then the idea that beats it: "pairs of divisors", "halving the exponent", "Euclid shrinks the numbers".
- Always state the complexity as a function of the value, then as a function of bits. Trial division is `O(sqrt n)` in the value but exponential in the number of bits. Interviewers like hearing that nuance.
- Edge cases to mention out loud: `0`, `1`, negative numbers, perfect squares (divisor pairs), `n = 2^31 - 1`, `INT_MIN`, and trailing zeros when reversing.
- Follow-ups you should expect:
  - "Do it without converting to a string." (digit loop)
  - "Do it in `O(log n)`." (fast power, Euclid)
  - "What if the number is too big for 64 bits?" (use `BigInt`, or digit arrays)
  - "How would you find primes in a range `[L, R]` with `R` up to 1e12?" (segmented sieve: sieve primes up to `sqrt(R)`, then cross out in the range)
  - "Why is a sieve `O(n log log n)`?" (the sum of `n / p` over primes `p` is about `n log log n`)
- Interview questions with short answers:
  1. *Why do divisors come in pairs, and why stop at `sqrt n`?* If `i * j = n` and both exceed `sqrt n`, their product exceeds `n`. So one of each pair is at most `sqrt n`.
  2. *Why is `gcd(a, b) = gcd(b, a % b)`?* Write `a = q * b + r`. Any common divisor of `a` and `b` divides `r = a - q * b`, and any common divisor of `b` and `r` divides `a`. The sets of common divisors are the same.
  3. *Why does fast power take `O(log n)`?* The exponent halves each loop, so it takes about `log2 n` loops (at most about 31 for 32-bit).
  4. *How do you compute `(a / b) mod p`?* You cannot divide directly. Multiply by the modular inverse `b^(p-2) mod p`, valid when `p` is prime and `b` is not a multiple of `p`.
  5. *Why does a trailing zero need only counting 5s?* Each zero needs a 2 and a 5, and 2s are far more common in `n!`.

## Practice List

**Easy**
- Palindrome Number
- Plus One
- Add Digits
- Power of Two
- Power of Three
- Fizz Buzz
- Count Primes
- Happy Number
- Excel Sheet Column Number
- Valid Perfect Square
- Pascal's Triangle II
- Ugly Number
- Self Dividing Numbers

**Medium**
- Reverse Integer
- Pow(x, n)
- Factorial Trailing Zeroes
- Excel Sheet Column Title
- Super Pow
- Fraction to Recurring Decimal
- Integer to Roman
- Count Numbers with Unique Digits
- Water and Jug Problem
- Ugly Number II
- Minimum Number of Operations to Make Array Empty
- Smallest Value After Replacing With Sum of Prime Factors

**Hard**
- Largest Palindrome Product
- Max Points on a Line
- Number of Digit One
- Preimage Size of Factorial Zeroes Function
- Prime Palindrome

## Key Takeaways

- **Digits come from `% 10` and `Math.floor(x / 10)`**; use `Math.trunc` for negatives, never plain `/`.
- **Loop to `sqrt(n)`** for divisors, primality and factorization; divisors and factors come in pairs.
- **Sieve of Eratosthenes** counts or lists primes up to `n` in `O(n log log n)`; start crossing out at `i * i`.
- **Euclid's `gcd(a, b) = gcd(b, a % b)`** is `O(log)`; compute LCM as `a / gcd * b`.
- **Halve the exponent** for fast power in `O(log n)`; the same loop works with a modulus.
- **JS numbers are exact only up to 2^53 - 1**: use `BigInt` for big modular products, and check 32-bit ranges yourself.
- **Count factors of 5** for trailing zeros; **subtract 1 first** for Excel titles; **cycle detection** solves Happy Number.

---

[← Previous: 03 - JavaScript for DSA](../03-javascript-for-dsa/README.md) | [Next: 01 - Arrays →](../../01-core-linear/01-arrays/README.md) | [Back to Foundations Index](../README.md)
