# Number Theory for Interviews

> **Estimated Time:** 6 hr | **Difficulty:** Advanced
> **Prerequisites:** [04 - Math Basics](../../00-foundations/04-math-basics/README.md), [08 - Binary Search](../../01-core-linear/08-binary-search/README.md)

---

## What You'll Learn
- You can do modular arithmetic in JavaScript without losing digits (`BigInt`, or a split-multiply trick on plain numbers).
- You can compute powers, inverses and `nCr mod p` fast, and know when each tool is allowed.
- You can sieve in ranges and factor many numbers quickly with a smallest-prime-factor table.
- You can count with inclusion-exclusion, and turn a "count pairs by GCD" problem into a loop over divisors.
- You can recognise the "answer modulo 1e9 + 7" family and pick the right formula or DP.

## The Big Idea

Think of a car odometer with a fixed number of digits. When it passes the maximum it wraps to zero, but you can still add trips and get the right last digits. Modular arithmetic is that odometer. The real answer may have thousands of digits, yet the question only asks for the part that fits on the dial.

The foundations chapter ([04 - Math Basics](../../00-foundations/04-math-basics/README.md)) already covered digits, primes, the plain sieve, GCD, fast power and a first look at `modPow`. This chapter builds the next layer: division under a modulus, counting formulas, and sieve tricks that make "for every number up to N" problems fast.

Why interviewers love it: the ideas are short, but one wrong step (an overflow, a negative remainder, a division that is not allowed) gives a wrong answer that looks fine on small tests. It shows whether you think carefully about limits.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "return the answer modulo 1e9 + 7" | wrap after every add and multiply; use `BigInt` for products |
| "compute a^b where b is huge" or "b is given as a digit array" | fast power, or process the digits one by one |
| "number of paths / arrangements / subsets of size k" | `nCr` with precomputed factorials and inverse factorials |
| "divide" inside a modular formula | multiply by the modular inverse (prime modulus) |
| "count primes between L and R", R up to 1e12 or so | segmented sieve (needs primes up to sqrt(R)) |
| "factor many numbers" or "distinct primes of each element" | smallest-prime-factor (SPF) sieve |
| "number of divisors for every number up to N" | divisor-count sieve, O(N log N) |
| "count numbers up to X divisible by A or B or C" | inclusion-exclusion with LCMs |
| "k-th number with property" and a count function exists | binary search on the answer + counting formula |
| "sum / count over all pairs of GCD" | count multiples per divisor, then subtract larger divisors |
| "number of balanced brackets / binary trees / triangulations" | Catalan number |
| "probability or expected value mod p" | fraction `p/q` becomes `p * inverse(q)` |

## Core Concepts / Template

### 1. Modular arithmetic and the 2^53 trap

All of these hold for any modulus `m`:

- `(a + b) % m`, `(a * b) % m` can be taken after reducing each side.
- Subtraction: `((a - b) % m + m) % m`, because JS `%` keeps the sign of the left side.
- Division is **not** allowed directly. See inverses below.

JavaScript numbers are 64-bit doubles. They store whole numbers exactly only up to `2^53 - 1` (about 9.007e15). With `m = 1e9 + 7`, two residues multiplied are near `1e18`, which is too big. Two fixes:

```js
const MOD = 1000000007n;                      // use BigInt for products

// Fix A: BigInt. Simple and safe. About 5-20x slower than numbers.
const mulA = (a, b) => (a * b) % MOD;

// Fix B: plain numbers, split b into a high and low 16-bit part.
// Works when m < 2^31 and 0 <= a, b < m. Each partial product stays below 2^53.
function mulMod(a, b, m) {
  return (((a * (b >>> 16)) % m) * 65536 + a * (b & 65535)) % m;
}
```

Why fix B works: `a * (b >>> 16)` is below `2^31 * 2^15 = 2^46`. After `% m` it is below `2^31`; times 65536 gives below `2^47`. And `a * (b & 65535)` is below `2^47`. The sum is below `2^48`, still exact.

Rule of thumb: sums and differences of residues are safe in plain numbers. Only products need care. In an interview, `BigInt` is the answer people expect; mention Fix B if they ask about speed.

### 2. Modular exponentiation

Square the base, halve the exponent, multiply into the result when the low bit is 1. O(log e). It is the same loop as `Pow(x, n)`, with `% m` after every multiply.

```js
function modPow(b, e, m) {            // all BigInt
  b = ((b % m) + m) % m;
  let r = 1n % m;                     // 1n % 1n is 0n: the modulus 1 edge case
  while (e > 0n) {
    if (e & 1n) r = (r * b) % m;
    b = (b * b) % m;
    e >>= 1n;
  }
  return r;
}
```

### 3. Modular inverse: two ways

The inverse of `a` mod `m` is the number `x` with `a * x = 1 (mod m)`. It lets you "divide": `a / b = a * inv(b)`.

**Fermat** (modulus must be prime, `a` not a multiple of it): `inv(a) = a^(p-2) mod p`. One line using `modPow`.

**Extended Euclid** (works for any modulus, if `gcd(a, m) = 1`): Euclid's algorithm also finds `x, y` with `a*x + m*y = gcd`. If the gcd is 1, then `a*x = 1 (mod m)`, so `x` is the inverse.

```js
const invFermat = (a, p) => modPow(a, p - 2n, p);

function extGcd(a, b) {                 // returns [g, x, y] with a*x + b*y = g
  if (b === 0n) return [a, 1n, 0n];
  const [g, x, y] = extGcd(b, a % b);
  return [g, y, x - (a / b) * y];
}
function invEuclid(a, m) {              // null when no inverse exists
  const [g, x] = extGcd(((a % m) + m) % m, m);
  return g !== 1n ? null : ((x % m) + m) % m;
}
```

Short trace, inverse of 3 mod 7:

| call | a, b | result [g, x, y] |
|---|---|---|
| extGcd(3, 7) | 3, 7 | calls extGcd(7, 3) |
| extGcd(7, 3) | 7, 3 | calls extGcd(3, 1) |
| extGcd(3, 1) | 3, 1 | calls extGcd(1, 0) = [1, 1, 0] |
| back in (3, 1) | | [1, 0, 1 - 3*0] = [1, 0, 1] |
| back in (7, 3) | | [1, 1, 0 - 2*1] = [1, 1, -2] |
| back in (3, 7) | | [1, -2, 1 - 0*(-2)] = [1, -2, 1] |

`x = -2`, and `(-2 % 7 + 7) % 7 = 5`. Check: `3 * 5 = 15 = 1 (mod 7)`. 

Use Fermat when the modulus is the usual prime. Use extended Euclid when the modulus might not be prime.

### 4. nCr mod p with factorials

`C(n, r) = n! / (r! * (n-r)!)`. Precompute `fact[i]` once, then `inv(fact[n])` once, and walk down: `invFact[i-1] = invFact[i] * i`. That gives each query in O(1) after O(n) setup, with only one `modPow` call.

```js
function makeFact(n) {                    // BigInt arrays, modulus MOD
  const f = [1n], inv = new Array(n + 1);
  for (let i = 1; i <= n; i++) f.push((f[i - 1] * BigInt(i)) % MOD);
  inv[n] = modPow(f[n], MOD - 2n, MOD);
  for (let i = n; i > 0; i--) inv[i - 1] = (inv[i] * BigInt(i)) % MOD;
  return { f, inv };
}
// n, r are plain numbers (array indexes); result is a BigInt
const nCr = (n, r, { f, inv }) =>
  r < 0 || r > n ? 0n : (((f[n] * inv[r]) % MOD) * inv[n - r]) % MOD;
```

This needs `n < p` (so no factorial is a multiple of `p`). With `p = 1e9 + 7` and `n` up to a few million, that holds. If `n >= p`, you need Lucas' theorem (rare in interviews; just name it).

### 5. Catalan numbers

Catalan numbers count many shapes: balanced bracket strings, binary search trees with n nodes, ways to triangulate a polygon, Dyck paths. The first values are `1, 1, 2, 5, 14, 42, 132`.

- `C(n) = C(2n, n) / (n + 1)` (closed form).
- `C(n) = sum over i of C(i) * C(n-1-i)` (the DP form, O(n^2)).

The DP form comes from splitting at a root: `i` nodes on the left, `n-1-i` on the right.

### 6. Sieves you will reuse

**Smallest prime factor (SPF) sieve.** For each number store its smallest prime factor. Then factoring `x` takes O(log x): repeatedly divide by `spf[x]`.

```js
function buildSpf(n) {
  const spf = new Int32Array(n + 1);
  for (let i = 2; i <= n; i++)
    if (spf[i] === 0)                       // i is prime
      for (let j = i; j <= n; j += i) if (spf[j] === 0) spf[j] = i;
  return spf;
}
```

**Euler's totient** `phi(n)` = how many numbers in `1..n` have gcd 1 with `n`. Formula: `phi(n) = n * product of (1 - 1/p)` over distinct primes `p` of `n`. A sieve builds all values:

```js
function totients(n) {
  const phi = Array.from({ length: n + 1 }, (_, i) => i);
  for (let i = 2; i <= n; i++)
    if (phi[i] === i)                        // untouched so far means prime
      for (let j = i; j <= n; j += i) phi[j] -= phi[j] / i;
  return phi;                                // phi[9]=6, phi[10]=4, phi[13]=12
}
```

Uses: Euler's theorem (`a^phi(m) = 1 mod m` when `gcd(a, m) = 1`), counting fractions in lowest terms, and reducing huge exponents.

**Segmented sieve.** To find primes in `[L, R]` when `R` is too large for an array of size `R`: sieve the small primes up to `sqrt(R)`, then cross out their multiples inside a window of size `R - L + 1`. See problem 3.

### 7. Chinese remainder theorem (brief)

If `x = r1 (mod m1)` and `x = r2 (mod m2)` and `m1, m2` are coprime, there is exactly one answer modulo `m1 * m2`. Find it with extended Euclid: `m1*u + m2*v = 1`, then `x = r1 + m1 * ((r2 - r1) * u mod m2)`.

```js
function crt(r1, m1, r2, m2) {            // BigInt; returns null if no solution
  const [g, u] = extGcd(m1, m2);
  if ((r2 - r1) % g !== 0n) return null;
  const l = (m1 / g) * m2;
  const t = (((r2 - r1) / g) * u) % (m2 / g);
  return (((r1 + m1 * t) % l) + l) % l;
}
// crt(2n,3n, 3n,5n) -> 8n   (8 % 3 = 2, 8 % 5 = 3)
```

Interviews rarely ask for the code. Know the statement: "coprime moduli let you solve each one separately and glue the answers".

### 8. Inclusion-exclusion

To count numbers up to `X` divisible by `a` or `b` or `c`, add the single counts, subtract the pair overlaps, add back the triple overlap. An overlap of "divisible by a and b" is "divisible by lcm(a, b)".

```
count(X) = X/a + X/b + X/c - X/lcm(a,b) - X/lcm(a,c) - X/lcm(b,c) + X/lcm(a,b,c)    (integer division)
```

Without the subtraction, numbers divisible by two of them would be counted twice.

### 9. Probability and expected value basics

- Probability = good outcomes / all outcomes (when all are equally likely).
- **Linearity of expectation:** `E[X + Y] = E[X] + E[Y]`, even if X and Y depend on each other. Example: the expected number of fixed points in a random permutation of n items is `n * (1/n) = 1`, because each position is a fixed point with chance `1/n`. No need to enumerate permutations.
- Expected value with a loop-until-success: if each try succeeds with chance `p`, the expected number of tries is `1/p`.
- When the answer must be "mod 1e9 + 7": a probability `p/q` is returned as `p * inv(q) mod 1e9+7`. Example: `1/2` becomes `500000004`.

---

## Problems (easy -> hard)

### 1. Bulb Switcher — Easy
**Companies:** Google, Amazon, Microsoft
**Problem:** There are `n` bulbs, all off. In round `i` you toggle every `i`-th bulb (rounds 1 to n). How many bulbs are on at the end? Example: `n = 3` -> `1`.
**Intuition:** Bulb `k` is toggled once for each divisor of `k`. It ends on only if `k` has an odd number of divisors. Divisors come in pairs `(d, k/d)`, so the count is odd only when one pair repeats, i.e. `k` is a perfect square.
**Approach 1 - Brute force:** Simulate all rounds. Time O(n log n) with stepping, Space O(n).
**Approach 2 - Optimal:** Count perfect squares up to `n`: `floor(sqrt(n))`.

| n | squares up to n | answer |
|---|---|---|
| 3 | 1 | 1 |
| 10 | 1, 4, 9 | 3 |

```js
function bulbSwitch(n) {
  return Math.floor(Math.sqrt(n));
}
// bulbSwitch(3) -> 1, bulbSwitch(0) -> 0, bulbSwitch(1e9) -> 31622
```

**Complexity:** Time O(1), Space O(1).
**Watch out:** `Math.sqrt` is exact enough for `n` up to 2^31. For values near 2^53, fix the result with a check like `while (r*r > n) r--`.

### 2. Power Mod (Pow(x, n) mod m) — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** Given `a`, `b` (can be huge) and `m`, return `a^b mod m`. Example: `a = 2, b = 10, m = 1000` -> `24`.
**Intuition:** Never build `a^b`. Square the base and halve the exponent, wrapping with `% m` each step. The exponent has about `log2(b)` bits, so about that many steps.
**Approach 1 - Brute force:** Multiply `b` times. Time O(b), too slow for `b` near 1e18.
**Approach 2 - Optimal:** Binary exponentiation from the Core section, with `BigInt`.

Trace for `2^10 mod 1000` (10 = binary 1010):

| bit | base before | result after |
|---|---|---|
| 0 | 2 | 1 |
| 1 | 4 | 4 |
| 0 | 16 | 4 |
| 1 | 256 | 4 * 256 = 1024 % 1000 = 24 |

```js
function powMod(a, b, m) {              // accepts numbers or strings, returns a number
  return Number(modPow(BigInt(a), BigInt(b), BigInt(m)));
}
// powMod(2, 10, 1000) -> 24, powMod(2, 0, 1) -> 0, powMod(5, 3, 13) -> 8
// powMod(2, "1000000000000000000", 1e9 + 7) works because the exponent is a string
```

**Complexity:** Time O(log b), Space O(1).
**Watch out:** `b` above 2^53 must arrive as a string or `BigInt`, not a number. `m = 1` must return 0 (that is why `r` starts as `1n % m`). Negative base needs the first normalising line. If asked to reduce a huge exponent: `a^b = a^(b mod phi(m))` is valid only when `gcd(a, m) = 1`.

### 3. Count Primes in a Range (Segmented Sieve) — Medium
**Companies:** Amazon, Google
**Problem:** Given `L <= R` with `R` up to about 1e12 and `R - L` up to about 1e6, count the primes in `[L, R]`. Example: `[1, 10]` -> `4` (2, 3, 5, 7).
**Intuition:** Every composite number `x <= R` has a prime factor at most `sqrt(R)`. Find those small primes with a normal sieve. Then, for each, cross out its multiples inside the window only. Memory is `sqrt(R) + (R - L)`, not `R`.
**Approach 1 - Brute force:** Test each number by trial division. Time O((R-L) * sqrt(R)).
**Approach 2 - Optimal:** Segmented sieve.

Trace for `[10, 30]`: small primes up to `sqrt(30) = 5` are 2, 3, 5. Cross out multiples inside the window: for 2 start at 10, for 3 start at 12, for 5 start at 10. Survivors: 11, 13, 17, 19, 23, 29 -> 6.

```js
function countPrimesInRange(L, R) {
  if (R < 2) return 0;
  L = Math.max(L, 2);                              // 0 and 1 are not prime
  const lim = Math.floor(Math.sqrt(R));
  const small = new Uint8Array(lim + 1), primes = [];
  for (let i = 2; i <= lim; i++) {
    if (!small[i]) {
      primes.push(i);
      for (let j = i * i; j <= lim; j += i) small[j] = 1;
    }
  }
  const comp = new Uint8Array(R - L + 1);          // comp[j - L] = 1 means composite
  for (const p of primes) {
    // start at the first multiple >= L, and never below p*p so p itself survives
    let s = Math.max(p * p, Math.ceil(L / p) * p);
    for (let j = s; j <= R; j += p) comp[j - L] = 1;
  }
  let c = 0;
  for (const x of comp) if (!x) c++;
  return c;
}
// countPrimesInRange(1, 10) -> 4, countPrimesInRange(10, 30) -> 6
```

**Complexity:** Time O(sqrt(R) log log R + (R-L) log log R), Space O(sqrt(R) + (R-L)).
**Watch out:** forgetting `p*p` as the lower start crosses out the prime `p` itself when it lies in the window. Forgetting `L = max(L, 2)` counts 1 as prime. Products like `p * p` stay exact for `R` up to about 1e15.

### 4. Number of Divisors via Sieve — Medium
**Companies:** -
**Problem:** For every `x` in `1..N`, find how many divisors `x` has. Example: `N = 12` -> `d[12] = 6` (1, 2, 3, 4, 6, 12).
**Intuition:** Flip the question. Instead of asking "which numbers divide x?", ask "which numbers does `i` divide?". Each `i` divides `i, 2i, 3i, ...`, so add 1 to each of those.
**Approach 1 - Brute force:** For each `x`, try all `d` up to `sqrt(x)`. Time O(N sqrt(N)).
**Approach 2 - Optimal:** Harmonic-sum sieve. The total work is `N/1 + N/2 + ... + N/N`, about `N ln N`.

```js
function divisorCounts(n) {
  const d = new Int32Array(n + 1);
  for (let i = 1; i <= n; i++)
    for (let j = i; j <= n; j += i) d[j]++;
  return d;
}
// divisorCounts(36)[12] -> 6, [36] -> 9, [13] -> 2, [1] -> 1
```

**Complexity:** Time O(N log N), Space O(N).
**Watch out:** for a single number do plain `sqrt` trial division instead. Using the SPF table, if `x = p1^a1 * p2^a2 ...` then the divisor count is `(a1+1)(a2+1)...`; that is O(N) total for all `x` with a linear sieve, but the harmonic version is easier to write correctly.

### 5. Distinct Prime Factors of Product of Array — Medium
**Companies:** Amazon, Google
**Problem:** Given an array of positive integers, return how many distinct primes divide the product of all elements. Example: `[2,4,3,7,10,6]` -> `4` (primes 2, 3, 5, 7).
**Intuition:** The product is huge, but you never need it. A prime divides the product exactly when it divides some element. So factor each element and put its primes in a set.
**Approach 1 - Brute force:** Trial-divide each element by every number up to its square root. Time O(n sqrt(M)).
**Approach 2 - Optimal:** Build an SPF table up to the max value, then factor each element in O(log x) steps.

Trace for `x = 10`: `spf[10] = 2` -> add 2, strip all 2s -> 5. `spf[5] = 5` -> add 5, strip -> 1. Set now has {2, 5}.

```js
function distinctPrimeFactors(nums) {
  const spf = buildSpf(Math.max(...nums));   // from the Core section
  const seen = new Set();
  for (let x of nums) {
    while (x > 1) {
      const p = spf[x];
      seen.add(p);
      while (x % p === 0) x = x / p;         // strip every copy of p
    }
  }
  return seen.size;
}
// distinctPrimeFactors([2,4,3,7,10,6]) -> 4, ([2,4,8,16]) -> 1, ([1,1]) -> 0
```

**Complexity:** Time O(M log log M + n log M), Space O(M), where `M` is the max element.
**Watch out:** `Math.max(...nums)` can overflow the call stack for arrays of around 100k+ items; use a loop or `reduce` there. Value 1 has no prime factors.

### 6. nCr mod 1e9+7 — Medium
**Companies:** Amazon, Google
**Problem:** Given `n` and `r` (up to about 1e6), return `C(n, r) mod 1e9 + 7`. Example: `C(5, 2)` -> `10`.
**Intuition:** `C(n, r)` is a ratio of huge factorials, so you cannot divide after wrapping. But the modulus is prime, so division becomes multiplication by an inverse. Precompute factorials and inverse factorials once; each query is two multiplies.
**Approach 1 - Brute force:** Pascal's triangle with wraparound. Time O(n * r), Space O(r).
**Approach 2 - Optimal:** Factorial tables from the Core section, one `modPow`.

```js
function nCrMod(n, r) {
  if (r < 0 || r > n) return 0;
  const { f, inv } = makeFact(n);
  return Number((((f[n] * inv[r]) % MOD) * inv[n - r]) % MOD);
}
// nCrMod(5, 2) -> 10, nCrMod(10, 0) -> 1, nCrMod(3, 5) -> 0, nCrMod(1000, 500) -> 159835829
```

**Complexity:** Time O(n) setup, then O(1) per query. Space O(n).
**Watch out:** `r > n` must return 0, not crash. For many queries build the table once, to the largest `n`. The method fails when `n >= p`.

### 7. Unique Paths (Combinatorics) — Medium
**Companies:** Amazon, Google, Microsoft, Bloomberg
**Problem:** A robot starts at the top-left of an `m x n` grid and can only move right or down. Count the ways to reach the bottom-right. Example: `m = 3, n = 7` -> `28`.
**Intuition:** Every path has exactly `m-1` down moves and `n-1` right moves, in some order. So you pick which of the `m+n-2` steps are the down moves: `C(m+n-2, m-1)`.
**Approach 1 - Brute force:** Recursion on (row, col). Time O(2^(m+n)).
**Approach 2 - Better:** DP grid. Time O(mn), Space O(n).
**Approach 3 - Optimal:** The binomial formula. With `m = 3, n = 7`: `C(8, 2) = 8*7/2 = 28`.

```js
function uniquePaths(m, n) {               // exact answer, fits a double by the problem limits
  const k = Math.min(m, n) - 1, total = m + n - 2;
  let r = 1;
  for (let i = 1; i <= k; i++) r = (r * (total - k + i)) / i; // stays an integer each step
  return Math.round(r);
}
// uniquePaths(3, 7) -> 28, uniquePaths(3, 2) -> 3, uniquePaths(1, 1) -> 1

function uniquePathsMod(m, n) {            // same count, modulo 1e9 + 7, for big grids
  const { f, inv } = makeFact(m + n - 2);
  return Number((((f[m + n - 2] * inv[m - 1]) % MOD) * inv[n - 1]) % MOD);
}
```

**Complexity:** Time O(min(m, n)) for the loop, or O(m + n) for the mod version. Space O(1) or O(m + n).
**Watch out:** in the loop, multiply first and then divide; the running value after step `i` equals `C(total - k + i, i)`, which is always an integer. Dividing first gives fractions. If the original problem adds obstacles, the formula no longer works and you need the DP.

### 8. Catalan Numbers (Unique Binary Search Trees) — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** How many structurally different BSTs store the values `1..n`? Example: `n = 3` -> `5`.
**Intuition:** Pick a root `i`. Values below `i` form the left subtree, values above form the right. The two sides are independent, so multiply their counts and sum over all roots. That recurrence is exactly the Catalan numbers.
**Approach 1 - Brute force:** Recursion without caching. Time exponential.
**Approach 2 - Better:** DP with `d[i] = sum of d[j] * d[i-1-j]`. Time O(n^2), Space O(n).
**Approach 3 - Optimal:** Closed form `C(2n, n) / (n + 1) = (2n)! / (n! (n+1)!)` under a modulus.

Trace of the DP: `d[0] = 1`, `d[1] = 1`, `d[2] = d0*d1 + d1*d0 = 2`, `d[3] = d0*d2 + d1*d1 + d2*d0 = 5`.

```js
function numTreesDP(n) {                     // exact, fine for n <= 19 (LeetCode range)
  const d = [1];
  for (let i = 1; i <= n; i++) {
    d[i] = 0;
    for (let j = 0; j < i; j++) d[i] += d[j] * d[i - 1 - j];
  }
  return d[n];
}

function catalanMod(n) {                     // modulo 1e9 + 7, n up to millions
  const { f, inv } = makeFact(2 * n + 1);
  return Number((((f[2 * n] * inv[n]) % MOD) * inv[n + 1]) % MOD);
}
// numTreesDP(3) -> 5, numTreesDP(5) -> 42; catalanMod(3) -> 5, catalanMod(0) -> 1
```

**Complexity:** DP: Time O(n^2), Space O(n). Closed form: Time O(n), Space O(n).
**Watch out:** the table must reach `2n + 1`, not just `2n`, because the formula uses `inv[n+1]` (it fails for `n = 0` otherwise). Catalan grows fast, so the exact DP overflows doubles past about `n = 30`. The count of valid strings in Generate Parentheses is also Catalan(n).

### 9. Number of Ways to Paint N x 3 Grid — Hard
**Companies:** Google, Amazon
**Problem:** A grid has `n` rows and 3 columns. Paint each cell red, yellow or green so no two neighbours (side by side or above/below) share a colour. Count the ways, mod 1e9 + 7. Example: `n = 1` -> `12`, `n = 2` -> `54`.
**Intuition:** One row has two shapes: `ABA` (first and last the same, 6 ways) and `ABC` (all different, 6 ways). The next row depends only on the shape of the current one. Work out how many next-row shapes each can produce:
- From `ABA`: 3 next rows of type `ABA`, 2 of type `ABC`.
- From `ABC`: 2 next rows of type `ABA`, 2 of type `ABC`.

**Approach 1 - Brute force:** Try all colourings row by row. Time O(12^n).
**Approach 2 - Optimal:** Keep two counts `a` (rows of shape ABA) and `b` (shape ABC): `a' = 3a + 2b`, `b' = 2a + 2b`.

| n | a (ABA) | b (ABC) | total |
|---|---|---|---|
| 1 | 6 | 6 | 12 |
| 2 | 3*6+2*6 = 30 | 2*6+2*6 = 24 | 54 |

```js
function numOfWays(n) {
  let a = 6n, b = 6n;
  for (let i = 1; i < n; i++) {
    [a, b] = [(3n * a + 2n * b) % MOD, (2n * a + 2n * b) % MOD];
  }
  return Number((a + b) % MOD);
}
// numOfWays(1) -> 12, numOfWays(2) -> 54, numOfWays(7) -> 106494, numOfWays(5000) -> 30228214
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** use the old `a` and `b` for both new values (the destructuring does this). Plain numbers would be safe here, since each product is below 1e10, but keep `BigInt` or `%` after each step so values never grow. A matrix power version gives O(log n) if asked.

### 10. Super Pow — Medium
**Companies:** Google, Amazon, Facebook
**Problem:** Compute `a^b mod 1337`, where `b` is a huge number given as an array of digits. Example: `a = 2, b = [1,0]` -> `1024`, since `2^10 = 1024` and `1024 mod 1337 = 1024`.
**Intuition:** Read the digits left to right. If the prefix so far is `P` and the next digit is `d`, the new exponent is `10*P + d`. So `a^(10P + d) = (a^P)^10 * a^d`. Keep the running result `r = a^P mod 1337` and update it.
**Approach 1 - Brute force:** Convert `b` to a number and run fast power. Fails when `b` has thousands of digits.
**Approach 2 - Optimal:** The digit-by-digit update, using fast power for the `^10` and `^d`.

Trace for `a = 2, b = [1, 0]`:

| digit | r before | r after = r^10 * 2^d |
|---|---|---|
| 1 | 1 | 1 * 2 = 2 |
| 0 | 2 | 2^10 * 1 = 1024 |

```js
function superPow(a, b) {
  const M = 1337;
  let r = 1;
  for (const d of b) {
    r = (powMod(r, 10, M) * powMod(a, d, M)) % M;
  }
  return r;
}
// superPow(2, [3]) -> 8, superPow(2, [1,0]) -> 1024,
// superPow(1, [4,3,3,8,5,2]) -> 1, superPow(2147483647, [2,0,0]) -> 1198
```

**Complexity:** Time O(len(b) * log 10), about O(len(b)), Space O(1).
**Watch out:** 1337 is not prime (7 * 191), so do not reach for the Fermat reduction of the exponent. Products here are below 1337^2, so plain numbers are fine. `powMod` is from problem 2 and returns a number.

### 11. Ugly Number III — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** An ugly number here is divisible by `a`, `b` or `c`. Return the `n`-th smallest one. Example: `n = 3, a = 2, b = 3, c = 5` -> `4` (the sequence is 2, 3, 4, 5, 6, ...).
**Intuition:** It is hard to jump to the `n`-th item, but easy to count how many ugly numbers are `<= x` using inclusion-exclusion. That count only goes up as `x` grows, so binary search for the smallest `x` whose count is at least `n`.
**Approach 1 - Brute force:** Walk up from 1 counting. Time O(answer), too slow (answer up to 2e9).
**Approach 2 - Optimal:** Binary search on `x` in `[1, 2e9]` with the counting formula from Core section 8.

Trace for `n = 3` with `(2, 3, 5)`: `count(4) = 2 + 1 + 0 - 0 - 0 - 0 + 0 = 3 >= 3`, and `count(3) = 1 + 1 = 2 < 3`. The answer is 4.

```js
function nthUglyNumber(n, a, b, c) {
  const gcd = (x, y) => { while (y) [x, y] = [y, x % y]; return x; };
  const lcm = (x, y) => (x / gcd(x, y)) * y;        // divide first to stay small
  const ab = lcm(a, b), ac = lcm(a, c), bc = lcm(b, c), abc = lcm(ab, c);
  const count = (x) =>
    Math.floor(x / a) + Math.floor(x / b) + Math.floor(x / c)
    - Math.floor(x / ab) - Math.floor(x / ac) - Math.floor(x / bc)
    + Math.floor(x / abc);
  let lo = 1, hi = 2e9;
  while (lo < hi) {
    const mid = Math.floor((lo + hi) / 2);
    if (count(mid) >= n) hi = mid; else lo = mid + 1;
  }
  return lo;
}
// nthUglyNumber(3,2,3,5) -> 4, (4,2,3,4) -> 6, (5,2,11,13) -> 10
// (1000000000, 2, 217983653, 336916467) -> 1999999984
```

**Complexity:** Time O(log(2e9)), about 31 steps, Space O(1).
**Watch out:** LCMs of three numbers near 1e9 reach about 1e27, which is not exact in a double. That is fine here: any LCM above `hi` makes its `Math.floor(x / l)` equal 0, and the rounding error cannot turn a huge value into a small one. If you want a strict guarantee, use `BigInt` for the LCMs. Use `>=` and shrink `hi`, so the search lands on a number that is itself ugly.

### 12. Fraction to Recurring Decimal — Medium
**Companies:** Google, Amazon, Facebook, Microsoft
**Problem:** Given two integers (numerator, denominator), return the decimal as a string, putting any repeating part in parentheses. Example: `4 / 333` -> `"0.(012)"`, `1 / 6` -> `"0.1(6)"`.
**Intuition:** Do long division by hand. After each digit you have a remainder. If a remainder repeats, the digits repeat from where it first appeared. There are fewer than `denominator` possible remainders, so it must end or repeat.
**Approach 1 - Brute force:** Generate many digits and look for a cycle by eye. Not reliable.
**Approach 2 - Optimal:** Long division with a map from remainder to the position in the output.

Trace for `1 / 6`:

| step | remainder | action | digits |
|---|---|---|---|
| start | 1 | integer part 0, remember 1 -> pos 0 | |
| 1 | 10 | digit 1, new remainder 4, remember 4 -> pos 1 | 1 |
| 2 | 40 | digit 6, new remainder 4, seen at pos 1 | 16 |

Cycle starts at pos 1, so the result is `0.1(6)`.

```js
function fractionToDecimal(nu, de) {
  if (nu === 0) return "0";
  let s = (nu < 0) !== (de < 0) ? "-" : "";
  const n = Math.abs(nu), d = Math.abs(de);       // fine in a double (up to 2^31)
  s += Math.floor(n / d);
  let r = n % d;
  if (r === 0) return s;
  s += ".";
  const seen = new Map(), digits = [];
  while (r !== 0) {
    if (seen.has(r)) {
      const i = seen.get(r);
      return s + digits.slice(0, i).join("") + "(" + digits.slice(i).join("") + ")";
    }
    seen.set(r, digits.length);
    r *= 10;
    digits.push(Math.floor(r / d));
    r %= d;
  }
  return s + digits.join("");
}
// (1,2) -> "0.5"  (2,1) -> "2"  (4,333) -> "0.(012)"  (1,6) -> "0.1(6)"
// (-50,8) -> "-6.25"  (0,-5) -> "0"  (-2147483648,-1) -> "2147483648"
```

**Complexity:** Time O(d) in the worst case (at most `d` distinct remainders), Space O(d).
**Watch out:** the sign when only one side is negative; `0` over a negative number must print `"0"`, not `"-0"`. In Java `Math.abs(Integer.MIN_VALUE)` overflows, but JS is safe. Remember to store the remainder, not the digit, in the map; two different remainders can give the same digit.

### 13. Count Anagrams — Hard
**Companies:** Google, Amazon
**Problem:** A string has words separated by single spaces. Count the distinct strings you get by rearranging the letters inside each word (words stay in place), mod 1e9 + 7. Example: `"too hot"` -> `18`.
**Intuition:** Each word is independent, so multiply the answers. For one word of length `L`, the number of distinct arrangements is `L! / (c1! * c2! * ...)` where `ci` are the letter counts. This is the "arrangements with repeated letters" formula. Division becomes multiplication by inverse factorials.
**Approach 1 - Brute force:** Generate all permutations and dedupe. Time O(L!).
**Approach 2 - Optimal:** Factorial and inverse-factorial tables up to the longest word.

Trace for `"too hot"`: `too` -> `3! / 2! = 3`. `hot` -> `3! = 6`. Product 18.

```js
function countAnagrams(s) {
  const words = s.split(" ");
  let mx = 0;
  for (const w of words) mx = Math.max(mx, w.length);
  const { f, inv } = makeFact(mx);
  let ans = 1n;
  for (const w of words) {
    const cnt = {};
    for (const ch of w) cnt[ch] = (cnt[ch] || 0) + 1;
    let v = f[w.length];
    for (const k in cnt) v = (v * inv[cnt[k]]) % MOD;
    ans = (ans * v) % MOD;
  }
  return Number(ans);
}
// countAnagrams("too hot") -> 18, ("aa") -> 1, ("a") -> 1
```

**Complexity:** Time O(total length + longest word), Space O(longest word).
**Watch out:** compute the table size from the longest word, not the whole string. Never divide exact factorials (they are huge); always use the inverse tables. The answer for each word is a count of distinct strings, so duplicate letters must be divided out.

### 14. Number of Ways to Form a Target String Given a Dictionary — Hard
**Companies:** Google, Amazon
**Problem:** All words have equal length `m`. Build `target` by choosing characters from the columns, left to right: to take the next target letter from column `j`, you must then only use columns after `j`. Within a column you may pick any word having that letter. Count the ways, mod 1e9 + 7. Example: `words = ["acca","bbbb","caca"], target = "aba"` -> `6`.
**Intuition:** Only the column counts matter: `cnt[col][letter]` = how many words have that letter in that column. Then it is a DP over columns and target positions. The point of this problem here is the modular DP habit: wrap after each multiply and add.
**Approach 1 - Brute force:** Recursion over (column, target index). Exponential.
**Approach 2 - Optimal:** 1D DP, where `dp[j]` = ways to build the first `j` letters using the columns seen so far. For each column, go `j` from high to low so a column is used at most once:
`dp[j] += dp[j-1] * cnt[col][target[j-1]]`.

Trace idea for the example: column 0 letters are a, b, c; column 1 are c, b, a; column 2 are c, b, c; column 3 are a, b, a. Starting `dp = [1,0,0,0]`, the final `dp[3]` is 6.

```js
function numWays(words, target) {
  const m = words[0].length, n = target.length;
  if (n > m) return 0;                              // not enough columns
  const cnt = Array.from({ length: m }, () => new Array(26).fill(0));
  for (const w of words)
    for (let i = 0; i < m; i++) cnt[i][w.charCodeAt(i) - 97]++;
  const dp = new Array(n + 1).fill(0n);
  dp[0] = 1n;
  for (let i = 0; i < m; i++) {
    for (let j = Math.min(n, i + 1); j >= 1; j--) {  // high to low: one use per column
      const c = cnt[i][target.charCodeAt(j - 1) - 97];
      if (c) dp[j] = (dp[j] + dp[j - 1] * BigInt(c)) % MOD;
    }
  }
  return Number(dp[n]);
}
// numWays(["acca","bbbb","caca"], "aba") -> 6, (["abba","baab"], "bab") -> 4, (["a"], "aa") -> 0
```

**Complexity:** Time O(m * n + total word length), Space O(26 * m + n).
**Watch out:** iterating `j` upward would let one column serve two target letters. This is a reference for how to write "DP with mod"; see [05 - Dynamic Programming](../../05-dynamic-programming/01-dp-intuition/README.md) for the general DP method. `dp[j-1] * c` can pass 2^53, hence `BigInt`.

### 15. Sum of GCD of All Pairs — Hard
**Companies:** -
**Problem:** Given an array of positive integers, return the sum of `gcd(a[i], a[j])` over all pairs `i < j`. Example: `[2, 4, 6]` -> `6` (gcd 2, 2, 2).
**Intuition:** Pairs are too many to test (n^2). Group them by the value of the gcd instead. For each `d`, let `m` be how many elements are multiples of `d`. Then `C(m, 2)` pairs have a gcd that is a multiple of `d`. To keep only pairs whose gcd is exactly `d`, subtract the exact counts already found for `2d, 3d, ...`. Process `d` from large to small.
**Approach 1 - Brute force:** Two loops with Euclid. Time O(n^2 log M).
**Approach 2 - Optimal:** Frequency array + divisor loops. Time O(M log M).

Trace for `[2, 4, 6]` (`cnt[2] = cnt[4] = cnt[6] = 1`):

| d | multiples present (m) | C(m,2) | minus exact for larger multiples | exact(d) | adds d * exact |
|---|---|---|---|---|---|
| 6, 5, 4, 3 | 1 or 0 | 0 | 0 | 0 | 0 |
| 2 | 3 | 3 | exact(4) + exact(6) = 0 | 3 | 6 |
| 1 | 3 | 3 | exact(2) + ... = 3 | 0 | 0 |

Total 6.

```js
function sumGcdPairs(a) {
  let mx = 0;
  for (const x of a) mx = Math.max(mx, x);
  const cnt = new Int32Array(mx + 1);
  for (const x of a) cnt[x]++;
  const exact = new Array(mx + 1).fill(0n);        // BigInt: pair counts can pass 2^53
  let ans = 0n;
  for (let d = mx; d >= 1; d--) {
    let m = 0;
    for (let k = d; k <= mx; k += d) m += cnt[k];  // elements divisible by d
    let v = (BigInt(m) * BigInt(m - 1)) / 2n;      // pairs with gcd a multiple of d
    for (let k = 2 * d; k <= mx; k += d) v -= exact[k];
    exact[d] = v;
    ans += v * BigInt(d);
  }
  return ans;                                      // BigInt
}
// sumGcdPairs([2,4,6]) -> 6n, sumGcdPairs([5,5]) -> 5n
```

**Complexity:** Time O(M log M), Space O(M), where `M` is the maximum value.
**Watch out:** `m - 1` is negative when `m = 0`; the product `0 * -1` is still 0, so it is safe. Return a `BigInt` because the sum can be large.

---

## Common Mistakes
- **"Taking `% m` once at the end is fine."** Numbers blow past 2^53 long before. Wrap after every multiply.
- **"`a * b % m` is safe in plain numbers."** Not when `a` and `b` are near 1e9. Use `BigInt` or the split-multiply helper.
- **"`(a - b) % m` is the right subtraction."** It can be negative in JS. Use `((a - b) % m + m) % m`.
- **"I can divide with `/` and then take `% m`."** Division of wrapped values is wrong. Multiply by the modular inverse.
- **"Fermat inverse works for any modulus."** Only for a prime modulus. For other moduli use extended Euclid, and only when `gcd(a, m) = 1`.
- **"Mix `BigInt` and numbers freely."** `2n * 3` throws a `TypeError`. Convert once at the edges.
- **"A segmented sieve starts at the first multiple."** It must start at `max(p*p, first multiple >= L)`, otherwise primes inside the window get crossed out.
- **"Inclusion-exclusion: just subtract overlaps."** The signs alternate: add singles, subtract pairs, add the triple.

## Interview Tips
- Say the modulus out loud and why you pick `BigInt`: "a product of two residues can reach 1e18, and doubles are exact only to about 9e15."
- Name the tool before coding: "division under a prime modulus, so I use the inverse via Fermat" or "I need `nCr` many times, so I precompute factorials and inverse factorials."
- For counting problems, state the closed form first (paths = `C(m+n-2, m-1)`), then discuss how to compute it safely.
- Follow-ups you will hear: "What if the modulus is not prime?" (extended Euclid, or CRT over prime-power factors). "What if `n >= p`?" (Lucas' theorem). "Can you do it without `BigInt`?" (yes: the 16-bit split multiply from Core section 1).
- Binary-search-on-answer works whenever you can count "how many values are `<= x`" and the count is monotone. Ugly Number III is the standard example.
- Edge cases to mention: `n = 0` or `1`, `r > n`, `m = 1`, negative inputs, zero in a denominator, an array with all ones, repeated values.
- Say the cost of a precompute: "O(n) for the tables, then O(1) per query."

## Practice List
**Easy**
- Power of Two / Three / Four
- Count Primes
- Ugly Number
- Fizz Buzz
- Greatest Common Divisor of Strings
- Find Greatest Common Divisor of Array

**Medium**
- Pow(x, n)
- Super Ugly Number
- Ugly Number II
- Count Good Numbers
- Unique Paths
- Number of Ways to Paint N x 3 Grid
- Count Anagrams
- Distinct Prime Factors of Product of Array
- Four Divisors
- Subarray Sums Divisible by K
- Fraction to Recurring Decimal
- Nth Magical Number (binary search + LCM)

**Hard**
- Count Different Palindromic Subsequences (mod DP)
- Number of Ways to Form a Target String Given a Dictionary
- Count Array Pairs Divisible by K
- Number of Ways to Reorder Array to Get Same BST
- Number of Sets of K Non-Overlapping Line Segments (combinatorics + mod)

## Key Takeaways
- **Wrap after every operation**, and use `BigInt` (or a split multiply) for products, since doubles are exact only to 2^53.
- **Division means multiplication by an inverse.** Fermat for a prime modulus, extended Euclid when the modulus is composite and `gcd = 1`.
- **Precompute factorials and inverse factorials** once; then every `nCr` is O(1).
- **Counting formulas beat simulation:** paths are `C(m+n-2, m-1)`, bulbs on are `floor(sqrt(n))`, BSTs are Catalan numbers.
- **SPF sieve plus a segmented sieve** cover "factor many numbers" and "primes in a huge range".
- **Inclusion-exclusion plus binary search** solves "k-th number divisible by one of several values".
- **Group by divisor** (count multiples, subtract larger ones) to avoid checking all pairs.

---

[← Previous: Sparse Table and Sqrt Decomposition](../03-sparse-table-and-sqrt/README.md) | [Next: Design Problems →](../05-design-problems/README.md) | [Back to Advanced Index](../README.md)
