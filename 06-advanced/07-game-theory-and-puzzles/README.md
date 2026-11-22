# Game Theory and Puzzles

> **Estimated Time:** 5 hr | **Difficulty:** Intermediate
> **Prerequisites:** [01 - DP Intuition](../../05-dynamic-programming/01-dp-intuition/README.md), [10 - Bitmask DP](../../05-dynamic-programming/10-bitmask-dp/README.md), [04 - Number Theory](../04-number-theory/README.md)

---

## What You'll Learn
- You can label any small game position as **winning or losing** by working backwards from the end.
- You can turn a "two players, both play perfectly" problem into a **score difference DP** (minimax without the pain).
- You can spot the games where a **parity or modulo trick** gives the answer in O(1) (Nim, Divisor Game, Stone Game I).
- You can explain **Nim and Sprague-Grundy** in plain words and compute Grundy numbers in a few lines.
- You can solve the classic logic and math puzzles (bulbs, doors, water jugs, Josephus, Hanoi, egg drop, celebrity) and say why they work.

## The Big Idea

Think of playing chess against a friend who never blunders. You cannot hope for a mistake. The only question is: "if I move here, and then they play their best reply, and then I play my best reply, who ends up ahead?" You answer it by starting at the end of the game, where the result is obvious, and walking backwards.

That is the whole chapter. Game problems in interviews have these rules:

- Two players take turns.
- Both play **perfectly** (optimally).
- The game always ends.
- Nothing is random, and both can see everything ("perfect information").

For such games, every position is either **winning** (the player to move can force a win) or **losing** (whatever they do, the opponent can force a win). No draws are hard to handle; draws just become a third label or a score of 0.

The two rules that define everything:

```
A position is WINNING  if at least one move leads to a LOSING position (for the opponent).
A position is LOSING   if every move leads to a WINNING position (for the opponent), or there is no move.
```

The puzzle half of the chapter is the same spirit: a small observation (pairing, parity, divisors) collapses a scary simulation into one line. Interviewers love these problems because they test whether you can **find structure** instead of coding the first idea. They also have a clean "brute force, then pattern, then proof" story that is easy to talk through.

## How to Spot It

| If the problem says ... | think ... |
|---|---|
| "two players take turns, both play optimally" | winning/losing positions, minimax, memoization |
| "return true if the first player wins" | win/lose DP, or a parity pattern. Brute-force small n and look for the pattern |
| "maximize your score minus the opponent's" or "can the first player get at least as many" | **score difference** DP: `dp = my best (me - opponent)` |
| "players pick from the ends of an array" | interval DP over `[l, r]` |
| "each player takes 1..k from a pile" | modulo `(k+1)` pattern |
| "choose each number at most once" (small set, up to ~20) | bitmask of used numbers as memo key |
| "many piles, take any amount from one pile" | Nim, XOR of pile sizes |
| "toggle every i-th item" | an item is toggled once per **divisor**, so only perfect squares end up ON |
| "people removed in a circle, every k-th" | Josephus recurrence |
| "measure exactly T with jugs of size a and b" | gcd |
| "worst case over guesses, minimize the worst case" | minimax over a range (interval DP) |
| "find the one who is known by all and knows no one" | candidate elimination, one pass |

## Core Concepts / Template

### 1. Win/lose DP (single pile games)

Take a game where the state is just a number `i` (stones left). Compute `win[i]` from smaller states.

```js
// win[i] = can the player to move force a win with i stones left?
function winTable(n, moves) {
  const win = new Array(n + 1).fill(false); // win[0] = false: no move, you lost
  for (let i = 1; i <= n; i++)
    for (const m of moves)
      if (i - m >= 0 && !win[i - m]) { win[i] = true; break; }
  return win;
}
```

Trace for moves = {1,2,3}:

| i | moves lead to | win[i] |
|---|---|---|
| 0 | none | false |
| 1 | 0 (lose) | true |
| 2 | 1, 0 | true (can reach 0) |
| 3 | 2, 1, 0 | true |
| 4 | 3, 2, 1 (all win) | **false** |
| 5 | 4 (lose) | true |

Pattern: losing at every multiple of 4. That is the Nim Game answer in problem 1.

**Interview habit:** when you do not see a pattern, run this table for n = 1..15 and print it. The pattern usually jumps out. Then you can prove it.

### 2. Score difference DP (the minimax trick)

When players collect points, you do not need two tables (mine and theirs). Define:

```
dp(state) = (best score the player to move can get) - (what the opponent gets) from this state on
```

When I make a move worth `gain`, the opponent then plays the same game from the new state and gets `dp(next)` as THEIR advantage. So:

```
dp(state) = max over moves of ( gain(move) - dp(state after move) )
```

The minus sign is the entire trick. It folds "the opponent plays their best" into one formula. At the end: the first player wins if `dp(start) > 0`, ties if `= 0`.

```js
// generic shape
const go = (state) => {
  if (isEnd(state)) return 0;
  let best = -Infinity;
  for (const move of moves(state)) best = Math.max(best, gain(move) - go(next(state, move)));
  return best;
};
```

Why subtract and not add? Because `go(next)` is measured from the **opponent's** point of view. What is good for them is bad for me.

Some problems (Stone Game II) are easier with "how many stones can I collect from here" plus suffix sums: `me = remaining_total - opponent_best`. It is the same idea in disguise.

### 3. Nim and Sprague-Grundy in plain words

**Nim:** several piles. A move: pick one pile and remove any positive number of stones from it. The player who cannot move loses.

Surprising fact: compute the **XOR** of all pile sizes. If XOR is **nonzero**, the player to move wins. If it is **zero**, the player to move loses.

Why (short version): from XOR = 0, any move changes exactly one pile, which must change the XOR to something nonzero. From XOR != 0, you can always find one pile to shrink so the XOR becomes 0. So the winner can always hand the opponent a zero position, and zero positions end at "all piles empty".

**Sprague-Grundy** says: any "impartial" game (both players have the same moves available) behaves like one Nim pile. That pile's size is the position's **Grundy number**:

```
grundy(position) = mex { grundy(next) for every next position }
mex = smallest non-negative integer NOT in the set
```

- Grundy 0 means losing for the player to move.
- If a game is a **sum of independent games** (you pick one game and move in it), XOR the Grundy numbers of the parts. Nonzero means first player wins.

```js
// grundy numbers for "remove one of `moves` stones from a single pile"
function grundy(n, moves) {
  const g = new Array(n + 1).fill(0);
  for (let i = 1; i <= n; i++) {
    const seen = new Set();
    for (const m of moves) if (i - m >= 0) seen.add(g[i - m]);
    let x = 0;
    while (seen.has(x)) x++; // mex
    g[i] = x;
  }
  return g;
}
const nimFirstWins = (piles) => piles.reduce((a, b) => a ^ b, 0) !== 0;
```

Examples (checked by running the code):

| game | grundy for n = 0..8 |
|---|---|
| remove 1, 2 or 3 | 0 1 2 3 0 1 2 3 0 |
| remove 1, 3 or 4 | 0 1 0 1 2 3 2 0 1 |

First row: Grundy = `n mod 4`, which matches the Nim Game pattern. Second row: losing positions (Grundy 0) are n = 0, 2, 7, 9...

You will rarely need Sprague-Grundy in a coding interview. You may be asked "what if there are two piles?" and the answer is "XOR".

### 4. Parity and symmetry tricks

- **Mirror strategy:** if the second player can always copy the first player's move on a symmetric board, the second player wins. (Coin on a round table: first player takes the center, then mirrors.)
- **Parity:** count something that flips each turn. Stone Game I: the first player can grab all odd-indexed or all even-indexed piles, choosing the better one.
- **Pairing:** group the options into pairs so any move by the opponent has a reply in the same pair.

### 5. Brute force, find the pattern, prove it

For many "return true if first wins" problems:

1. Write the slow win/lose DP (memo is fine).
2. Print results for small inputs.
3. Spot the pattern (mod 4, even/odd, always true).
4. Say the proof out loud, then submit the O(1) answer. If you cannot prove it, submit the DP. A correct slow answer beats a wrong fast one.

### 6. Brief: Cat and Mouse (graph games)

LeetCode "Cat and Mouse" puts a cat and a mouse on a graph. The state is `(mouseNode, catNode, whoseTurn)`. Some states are known results (mouse at the hole = mouse wins, cat catches mouse = cat wins). You cannot use plain recursion because the game can loop forever (draw). The standard solution does **retrograde analysis**: start from known end states and run a BFS backwards:

- A state is a win for the player to move if **any** move leads to a state already known as their win.
- A state is a loss for the player to move if **all** moves lead to states known as the opponent's win (keep a counter of unresolved moves per state).
- States never resolved are draws.

This is hard-rated. Know the idea (backward BFS with degree counters, like topological sort); do not expect to be asked it often.

### 7. Brief: prisoners and hats

Classic: 100 prisoners in a line, each wears a black or white hat, each sees the hats in front of them, and they call out their own color from the back. How many can be saved for sure? Answer: 99. The last person announces "black" if the number of black hats they see is **odd**, otherwise "white". That announcement is a **parity** message. Each next person counts the blacks ahead, compares to the parity said so far (minus the answers already given), and deduces their own hat. The first person has a 50% chance; everyone else is exact.

Lesson: when information is limited, use a single bit to carry **parity** of everything.

### 8. Probability puzzles: simulate to check your reasoning

When you are not sure about a probability, write a 5-line simulation. It does not prove anything, but it catches wrong reasoning quickly and interviewers like seeing it.

```js
const simulate = (trials, fn) => {
  let hits = 0;
  for (let i = 0; i < trials; i++) if (fn()) hits++;
  return hits / trials;
};

// Monty Hall: should we switch doors?
const monty = (doSwitch) => simulate(100000, () => {
  const car = Math.floor(Math.random() * 3);
  const pick = Math.floor(Math.random() * 3);
  let open;
  do { open = Math.floor(Math.random() * 3); } while (open === car || open === pick);
  const final = doSwitch ? 3 - pick - open : pick; // the one door left
  return final === car;
});
// monty(true) ~ 0.667, monty(false) ~ 0.333

// Birthday: 23 people, chance two share a birthday?
const birthday = simulate(100000, () => {
  const seen = new Set();
  for (let i = 0; i < 23; i++) seen.add(Math.floor(Math.random() * 365));
  return seen.size < 23;
});
// ~ 0.507
```

Why switching wins at 2/3: your first pick is right 1/3 of the time. The host's reveal never changes that. So the other door holds the remaining 2/3. Switching wins exactly when your first pick was wrong. I ran both simulations: about 0.667 for switching, 0.333 for staying, and about 0.507 for the birthday case.

Other classics to be ready for: expected number of coin flips to get two heads in a row (6), and "a stick broken at two random points forms a triangle with probability 1/4". Quote these only if you are sure, and offer to simulate.

## Problems (easy -> hard)

### 1. Nim Game — Easy
**Companies:** Google, Adobe, Amazon
**Problem:** A heap has `n` stones. You and a friend alternate removing 1 to 3 stones, you go first. Whoever takes the last stone wins. Return true if you can win. Example: n = 4 -> false, n = 5 -> true.
**Intuition:** If 4 stones are left on your turn, you lose: whatever you take (1, 2 or 3), your friend takes the rest to make a total of 4 in that round. So 4 is a trap number. From any n that is not a multiple of 4, take `n % 4` stones and hand your friend a multiple of 4. From a multiple of 4, you are the one handed the trap.
**Approach 1 - Brute force:** win/lose DP: `win[i] = some m in 1..3 with !win[i-m]`. Time O(n), Space O(n).
**Approach 2 - Better:** keep only the last 3 values. Time O(n), Space O(1).
**Approach 3 - Optimal:** pattern. Table from the template shows false at 0, 4, 8, ... so the answer is `n % 4 !== 0`.

| n | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| first wins | T | T | T | F | T | T | T | F |

```js
const canWinNim = (n) => n % 4 !== 0;
```

**Complexity:** Time O(1), Space O(1).
**Watch out:** the generalization is `n % (k + 1) !== 0` when you may take 1..k. Do not forget the n values up to 2^31 - 1; a DP would be too slow, which is why the pattern matters.

### 2. Divisor Game — Easy
**Companies:** Amazon, Google, Microsoft
**Problem:** Alice and Bob start with a number `n` on the board. On a turn, the player picks `x` with `0 < x < n` and `n % x == 0`, then replaces `n` with `n - x`. A player who cannot move loses. Alice starts. Return true if Alice wins. Example: n = 2 -> true, n = 3 -> false.
**Intuition:** At n = 1 the player to move loses. At n = 2 you can only pick 1 and leave 1, so you win. Think about parity. If `n` is odd, every divisor `x` is odd, so `n - x` is even. If `n` is even, you can always pick `x = 1` and give an odd number. So the even player always hands out odd numbers and the odd player always hands out even numbers. The one who holds 1 (odd) is the loser, so the odd player loses.
**Approach 1 - Brute force:** recursion or DP over all divisors. Time O(n sqrt n) with memo, Space O(n).
**Approach 2 - Optimal:** return `n % 2 === 0`.

Dry run:

| n | Alice's options | result |
|---|---|---|
| 1 | none | lose |
| 2 | take 1 -> 1 (Bob loses) | win |
| 3 | take 1 -> 2 (Bob wins) | lose |
| 4 | take 1 -> 3 (Bob loses) | win |

```js
const divisorGame = (n) => n % 2 === 0;
```

**Complexity:** Time O(1), Space O(1).
**Watch out:** the proof is the induction: even numbers win by moving to an odd (which loses), odd numbers lose because every move gives an even (which wins). I also checked this against a brute-force table up to n = 60.

### 3. Flip Game — Easy
**Companies:** Google
**Problem:** A string has `+` and `-`. A move flips two consecutive `++` to `--`. Return all strings reachable in one move. Example: "++++" -> ["--++", "+--+", "++--"].
**Intuition:** This is the "list the moves" half of Flip Game II. Scan once, whenever you see two `+` side by side, build the new string.
**Approach 1 - Optimal (single approach):** one scan with slicing. Time O(n^2) because each slice costs O(n) and there are up to n moves, Space O(n) per result.

```js
function generatePossibleNextMoves(s) {
  const res = [];
  for (let i = 0; i + 1 < s.length; i++)
    if (s[i] === '+' && s[i + 1] === '+')
      res.push(s.slice(0, i) + '--' + s.slice(i + 2));
  return res;
}
```

**Complexity:** Time O(n^2), Space O(n^2) for the output.
**Watch out:** loop bound `i + 1 < s.length`, so a string like "+" gives []. Order of output follows left to right.

### 4. Tower of Hanoi (Move Count) — Easy
**Companies:** -
**Problem:** Move `n` disks from peg A to peg C using peg B, one disk at a time, never putting a bigger disk on a smaller one. How many moves are needed at minimum, and list them. Example: n = 3 -> 7.
**Intuition:** To move the biggest disk, you must first park the other n-1 disks on the middle peg. Then move the big one. Then move the n-1 disks on top of it. So `T(n) = 2*T(n-1) + 1`, which solves to `2^n - 1`.
**Approach 1 - Recursion:** move n-1 disks aside, move the big one, move n-1 back on top. Lists every move. Time O(2^n), Space O(n).
**Approach 2 - Formula:** the count is `2 ** n - 1`, Time O(1).

Trace for n = 2 (A to C via B):

```
move 1 disk A->B (park small)
move 1 disk A->C (big goes home)
move 1 disk B->C (small on top)
```

```js
const hanoiMoves = (n) => 2 ** n - 1;

function hanoi(n, from, to, via, out = []) {
  if (n === 0) return out;
  hanoi(n - 1, from, via, to, out); // clear the way
  out.push(`${from}->${to}`);       // move the biggest
  hanoi(n - 1, via, to, from, out); // rebuild on top
  return out;
}
// hanoi(2,'A','C','B') -> ['A->B','A->C','B->C']
```

**Complexity:** Time O(2^n) to list moves (O(1) for the count), Space O(n) recursion depth.
**Watch out:** for n up to 53 the count fits exactly in a JS number; beyond that use BigInt. The recurrence is the standard example of an exponential recursion with a closed form.

### 5. Bulb Switcher (and 100 Doors) — Medium
**Companies:** Google, Microsoft, Amazon
**Problem:** There are `n` bulbs, all off. Round 1 toggles every bulb. Round 2 toggles every 2nd bulb, round 3 every 3rd, ... round n toggles the last bulb. How many bulbs are on after `n` rounds? Example: n = 3 -> 1. The "100 doors" puzzle is exactly the same with n = 100.
**Intuition:** Bulb `i` is toggled in round `d` exactly when `d` divides `i`. So it is toggled once per divisor. It ends ON only if it has an **odd** number of divisors. Divisors come in pairs `(d, i/d)`, except when `d = i/d`, i.e. `i` is a perfect square. So only perfect squares end on. Count of squares up to n is `floor(sqrt(n))`.
**Approach 1 - Brute force:** simulate all rounds. Time O(n log n) (harmonic sum), Space O(n).
**Approach 2 - Optimal:** `Math.floor(Math.sqrt(n))`.

For n = 100 the open doors are 1, 4, 9, 16, 25, 36, 49, 64, 81, 100 (verified by running the simulation).

```js
const bulbSwitch = (n) => Math.floor(Math.sqrt(n));

// the simulation, handy to show interviewers the pattern first
function hundredDoors(n = 100) {
  const open = new Array(n + 1).fill(false);
  for (let step = 1; step <= n; step++)
    for (let d = step; d <= n; d += step) open[d] = !open[d];
  const res = [];
  for (let d = 1; d <= n; d++) if (open[d]) res.push(d);
  return res;
}
```

**Complexity:** Time O(1) (or O(n log n) simulating), Space O(1).
**Watch out:** n = 0 returns 0. For huge n, `Math.sqrt` on doubles is fine up to 2^31 but double-check borderline perfect squares if you ever go near 2^53.

### 6. Water and Jug Problem — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** You have two jugs of capacities `x` and `y` liters and unlimited water. Can you end up with exactly `target` liters in total across the two jugs? Operations: fill a jug, empty a jug, pour one into the other until full or empty. Example: (3, 5, 4) -> true. (2, 6, 5) -> false.
**Intuition:** Every operation changes the total by a multiple of `x` or `y`. So the amount you can make is some `a*x + b*y` with integers `a, b` (they may be negative: that is pouring away). By number theory (Bezout), the set of such values is exactly the multiples of `gcd(x, y)`. You also cannot have more than `x + y` in total.
**Approach 1 - Brute force:** BFS over states `(a, b)` with 6 moves each. Time O(x*y), Space O(x*y).
**Approach 2 - Optimal:** `target <= x + y && target % gcd(x, y) === 0`.

Dry run (3, 5, 4): gcd = 1, 4 is a multiple of 1, 4 <= 8, so true. Dry run (2, 6, 5): gcd = 2, 5 % 2 = 1, false.

```js
function canMeasureWater(x, y, target) {
  if (x + y < target) return false;
  if (target === 0) return true;
  const gcd = (a, b) => (b === 0 ? a : gcd(b, a % b));
  return target % gcd(x, y) === 0;
}
```

**Complexity:** Time O(log(min(x, y))), Space O(log) for recursion.
**Watch out:** if one jug is 0, `gcd(0, y) = y`, which works (target must be a multiple of y or 0 and not exceed y). Check `target === 0` before gcd so a (0, 0) input does not divide by zero.

### 7. Find the Celebrity — Medium
**Companies:** Facebook, LinkedIn, Google, Microsoft
**Problem:** In a party of `n` people, a celebrity is known by everyone and knows nobody. You have a function `knows(a, b)`. Find the celebrity or return -1. Aim for O(n) calls. Example: if only person 2 satisfies the rule, return 2.
**Intuition:** One call to `knows(a, b)` removes one person. If `a` knows `b`, then `a` is not a celebrity. If `a` does not know `b`, then `b` is not a celebrity (everyone must know the celebrity). So walk through people keeping one candidate; each call kills one person. Then verify the candidate once.
**Approach 1 - Brute force:** check every person against everyone, O(n^2) calls.
**Approach 2 - Optimal:** pick candidate `c = 0`. For each `i`, if `knows(c, i)` then `c = i`. After the loop, verify `c`: for all `i != c`, `knows(c, i)` must be false and `knows(i, c)` must be true.

Trace (person 2 is the celebrity, n = 4): c=0; knows(0,1)? no, stay 0; knows(0,2)? yes, c=2; knows(2,3)? no, stay 2. Verify 2: nobody known by 2, all know 2. Return 2.

```js
function findCelebrity(n, knows) {
  let c = 0;
  for (let i = 1; i < n; i++) if (knows(c, i)) c = i;
  for (let i = 0; i < n; i++) {
    if (i === c) continue;
    if (knows(c, i) || !knows(i, c)) return -1; // verification is required
  }
  return c;
}
```

**Complexity:** Time O(n) calls (at most 3n), Space O(1).
**Watch out:** the first pass finds the only POSSIBLE candidate. If there is no celebrity, it still returns someone, so the verification pass is not optional. n = 1 returns 0.

### 8. Flip Game II — Medium
**Companies:** Google
**Problem:** Same flipping rule as problem 3, but now two players alternate and the one who cannot move loses. Return true if the first player can guarantee a win. Example: "++++" -> true (flip the middle, leaving "+--+", opponent has no move).
**Intuition:** Apply the winning rule directly. A string is winning if some move leads to a string that is losing for the opponent. Memoize on the string because many move orders reach the same string.
**Approach 1 - Brute force:** recursion without memo: exponential.
**Approach 2 - Optimal:** recursion with a Map from string to result. State count is at most 2^n, in practice far smaller. (A Grundy-number solution over the segments of consecutive `+` runs is polynomial: each run is an independent game, XOR their Grundy values. Good to mention as a follow-up.)

Dry run on "++++": try flip at index 0 -> "--++": opponent flips index 2 -> "----", we have no move, we lose that line, so "--++" is winning for the opponent. Try index 1 -> "+--+": opponent has no move, so "+--+" is losing for them. Hence "++++" is winning.

```js
function canWinFlip(s) {
  const memo = new Map();
  const go = (str) => {
    if (memo.has(str)) return memo.get(str);
    let win = false;
    for (let i = 0; i + 1 < str.length && !win; i++)
      if (str[i] === '+' && str[i + 1] === '+')
        if (!go(str.slice(0, i) + '--' + str.slice(i + 2))) win = true;
    memo.set(str, win);
    return win;
  };
  return go(s);
}
```

**Complexity:** Time roughly O(n * number_of_states), up to exponential; fine for n <= 60 on real tests. Space O(number_of_states * n).
**Watch out:** no available move means false (you lose), which is the natural base case: the loop finds nothing and `win` stays false.

### 9. Predict the Winner (Coin Game, Pick From the Ends) — Medium
**Companies:** Google, Amazon, Microsoft, Facebook
**Problem:** Two players take turns picking a number from either end of an array. Player 1 starts. Each adds picks to their score. Return true if player 1 can get a score at least as large as player 2. Example: [1,5,2] -> false, [1,5,233,7] -> true.
**Intuition:** Use the score difference trick. Let `dp[l][r]` = (best you can win by) from the window `l..r` when it is your turn. Take left: you get `nums[l]`, then the opponent plays `l+1..r` and gets `dp[l+1][r]` ahead of you. So `nums[l] - dp[l+1][r]`. Same on the right.
**Approach 1 - Brute force:** recursion over both choices: O(2^n).
**Approach 2 - Better:** memoized interval DP: O(n^2) time and space.
**Approach 3 - Optimal (space):** since `dp[l][r]` depends only on `dp[l+1][r]` and `dp[l][r-1]`, keep a single array indexed by `l`, increasing window length.

Trace for [1,5,2]:

| window length | dp[0] | dp[1] | dp[2] | how |
|---|---|---|---|---|
| 1 | 1 | 5 | 2 | single pick |
| 2 | max(1-5, 5-1) = 4 | max(5-2, 2-5) = 3 | - | |
| 3 | max(1-dp[1], 2-dp[0]) = max(1-3, 2-4) = -2 | | | |

`dp[0] = -2 < 0`, so return false.

```js
function predictTheWinner(nums) {
  const n = nums.length;
  const dp = [...nums]; // dp[i] = best diff for the window starting at i, current length
  for (let len = 2; len <= n; len++)
    for (let i = 0; i + len - 1 < n; i++)
      // dp[i+1] is the window (i+1..j), dp[i] (old) is the window (i..j-1)
      dp[i] = Math.max(nums[i] - dp[i + 1], nums[i + len - 1] - dp[i]);
  return dp[0] >= 0;
}
```

**Complexity:** Time O(n^2), Space O(n).
**Watch out:** tie goes to player 1, so the test is `>= 0`. Updating `dp[i]` in increasing `i` is safe because `dp[i+1]` still holds the previous length when we read it.

### 10. Stone Game (I) — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** An **even** number of piles with a total that is **odd**. Alice and Bob alternately take a whole pile from either end, Alice first. Most stones wins. Return true if Alice wins. Example: [5,3,4,5] -> true.
**Intuition:** Index the piles 0..n-1. Alice can decide to take all even-indexed piles or all odd-indexed piles. She moves first: say she takes pile 0 (even index). Now Bob can only see the ends 1 or n-1, both odd-indexed. Whatever Bob takes, the ends are even-indexed again for Alice. So she gets either all evens or all odds, her choice. Since the total is odd, one group is strictly bigger. Alice picks that group. So the answer is always **true**.
**Approach 1 - Brute force / DP:** reuse problem 9. Time O(n^2).
**Approach 2 - Optimal:** `return true`.

```js
const stoneGame = () => true; // even number of piles, odd total: no tie possible
```

**Complexity:** Time O(1), Space O(1).
**Watch out:** this relies on the guarantees (even count, odd total, so no ties). If they change, fall back to the DP from problem 9. With an even count and an even total, Alice can still always get at least half (I tested that with the DP on random arrays), but the result may be a tie, so "wins" would need a different question.

### 11. The Josephus Problem (Find the Winner of the Circular Game) — Medium
**Companies:** Amazon, Google, Microsoft
**Problem:** `n` friends stand in a circle, numbered 1..n. Starting at friend 1, count `k` friends (including the start); that friend leaves. Counting restarts from the next friend. Last friend is the winner. Return the winner. Example: n = 5, k = 2 -> 3. n = 6, k = 5 -> 1.
**Intuition:** Think about the winner's position as 0-indexed. With 1 person, the winner is position 0. With `i` people, the first person removed is at position `(k-1) % i`. After that, we are left with `i-1` people, restarted from the one after the removed. A position `p` in the smaller circle corresponds to position `(p + k) % i` in the bigger circle (shift by `k`). So `f(i) = (f(i-1) + k) % i`.
**Approach 1 - Brute force:** simulate with an array and `splice`: Time O(n^2), Space O(n).
**Approach 2 - Optimal:** the recurrence in a loop, then add 1 for 1-indexed numbering.

Trace n = 5, k = 2 (0-indexed result `pos`):

| i | pos = (pos + k) % i |
|---|---|
| 1 | 0 |
| 2 | (0+2)%2 = 0 |
| 3 | (0+2)%3 = 2 |
| 4 | (2+2)%4 = 0 |
| 5 | (0+2)%5 = 2 |

Answer `2 + 1 = 3`.

```js
function josephus(n, k) {
  let pos = 0; // 0-indexed winner of a circle of size 1
  for (let i = 2; i <= n; i++) pos = (pos + k) % i;
  return pos + 1;
}

// simulation, good for checking
function josephusSim(n, k) {
  const a = Array.from({ length: n }, (_, i) => i + 1);
  let idx = 0;
  while (a.length > 1) { idx = (idx + k - 1) % a.length; a.splice(idx, 1); }
  return a[0];
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** the recurrence is 0-indexed; forgetting the final `+ 1` is the classic bug. I tested both functions against each other for n up to 30 and k up to 8. For k = 2 there is a closed form: write n in binary, move the leading 1 to the end (e.g. n = 5 = 101 -> 011 = 3), good to mention.

### 12. Can I Win — Medium
**Companies:** Google, Amazon, Microsoft
**Problem:** Players take turns picking an unused integer from 1..`maxChoosable` and add it to a running total. The player who makes the total reach or pass `desiredTotal` wins. Return whether the first player can force a win. Example: (10, 11) -> false.
**Intuition:** The state is "which numbers are already used". The running total is determined by the used set (it is the sum of used numbers), so the used set alone is enough as memo key. With `maxChoosable <= 20`, a bitmask has at most 2^20 states. This is the bitmask DP from [Bitmask DP](../../05-dynamic-programming/10-bitmask-dp/README.md) applied to a game.
**Approach 1 - Brute force:** try every sequence: O(n!).
**Approach 2 - Optimal:** memoize on the bitmask. At a state, I win if there is a number `k` such that `k >= remaining` (I finish now) or the state after picking `k` is losing for the opponent.

Edge checks before the DP:
- `desiredTotal <= 0`: the first player wins immediately.
- If the sum of all numbers `n(n+1)/2 < desiredTotal`, nobody can reach it: return false.

```js
function canIWin(maxChoosable, desired) {
  if (desired <= 0) return true;
  if ((maxChoosable * (maxChoosable + 1)) / 2 < desired) return false;
  const memo = new Map(); // used-bitmask -> can the player to move win?
  const go = (used, rem) => {
    if (memo.has(used)) return memo.get(used);
    let win = false;
    for (let k = 1; k <= maxChoosable && !win; k++) {
      const bit = 1 << (k - 1);
      if (used & bit) continue;
      if (k >= rem || !go(used | bit, rem - k)) win = true;
    }
    memo.set(used, win);
    return win;
  };
  return go(0, desired);
}
```

**Complexity:** Time O(2^n * n), Space O(2^n).
**Watch out:** the memo key is only the bitmask, which is valid because `rem` is a function of `used`. Do the "sum too small" check first, or you will treat an impossible game as a loss only after exploring everything.

### 13. Guess Number Higher or Lower II — Medium
**Companies:** Google, Amazon, Facebook
**Problem:** I pick a number in 1..n. You guess; a wrong guess of `x` costs `x` dollars and I tell you higher or lower. Return the minimum money that **guarantees** you win whatever number I picked. Example: n = 10 -> 16.
**Intuition:** This is minimax: you choose the guess to minimize the cost, the adversary (the secret number) chooses the side to maximize your cost. On range `[l, r]`, if you guess `x`, you pay `x` plus the worse of the two remaining ranges `[l, x-1]` and `[x+1, r]`. Take the best `x`.
**Approach 1 - Brute force:** recursion on every range: exponential.
**Approach 2 - Optimal:** interval DP by increasing length.

```
dp[l][r] = min over x in [l..r] of ( x + max(dp[l][x-1], dp[x+1][r]) )
dp for a range of length 1 = 0 (you guess right immediately)
```

Small checks: n = 2 -> 1 (guess 1; if wrong, the number is 2, so you pay 1). n = 3 -> 2 (guess 2; if wrong you pay 2 and then know the answer).

```js
function getMoneyAmount(n) {
  const dp = Array.from({ length: n + 2 }, () => new Array(n + 2).fill(0));
  for (let len = 2; len <= n; len++)
    for (let l = 1; l + len - 1 <= n; l++) {
      const r = l + len - 1;
      let best = Infinity;
      for (let x = l; x <= r; x++) {
        const left = x > l ? dp[l][x - 1] : 0;
        const right = x < r ? dp[x + 1][r] : 0;
        best = Math.min(best, x + Math.max(left, right));
      }
      dp[l][r] = best;
    }
  return dp[1][n];
}
```

**Complexity:** Time O(n^3), Space O(n^2).
**Watch out:** it is NOT binary search; the cost depends on the value guessed, so the best guess is usually above the middle. The guarantee is about the worst case, not the average. Expected values: n = 1 -> 0, n = 2 -> 1, n = 3 -> 2, n = 10 -> 16 (all tested).

### 14. Stone Game II — Medium
**Companies:** Google, Amazon
**Problem:** Piles in a row. A player may take the first `X` piles where `1 <= X <= 2M`, then `M = max(M, X)`. Start with `M = 1`, Alice first. Both want the most stones. Return the max stones Alice can get. Example: [2,7,9,4,4] -> 10.
**Intuition:** State is `(i, M)`: the first `i` piles are gone and the current limit is `M`. Use suffix sums. If I take `x` piles, the opponent moves from `(i + x, max(M, x))` and collects `go(i+x, M')`; I then get everything left in the suffix minus what the opponent gets. This is the "total minus opponent's best" form of the same trick.
**Approach 1 - Brute force:** recursion, exponential.
**Approach 2 - Optimal:** memoize on `(i, M)`: about n * n states, each trying up to `2M` options.
Shortcut: if `i + 2M >= n`, take everything: `suf[i]`.

```js
function stoneGameII(piles) {
  const n = piles.length;
  const suf = new Array(n + 1).fill(0);
  for (let i = n - 1; i >= 0; i--) suf[i] = suf[i + 1] + piles[i];
  const memo = new Map();
  const go = (i, M) => {
    if (i >= n) return 0;
    if (i + 2 * M >= n) return suf[i]; // can grab it all
    const key = i * (n + 1) + M;
    if (memo.has(key)) return memo.get(key);
    let best = 0;
    for (let x = 1; x <= 2 * M; x++)
      best = Math.max(best, suf[i] - go(i + x, Math.max(M, x)));
    memo.set(key, best);
    return best;
  };
  return go(0, 1);
}
```

Trace for [2,7,9,4,4] (total 26): Alice takes 1 pile (2). Bob, with M = 1, takes 2 piles (7 and 9) = 16, so M becomes 2. Alice can now take up to 4 piles and grabs the rest (4 + 4 = 8). Alice total: 2 + 8 = 10. Taking 2 piles first would let Bob (M = 2) grab all 3 remaining piles (9 + 4 + 4 = 17), leaving Alice 9. The code tries every option and returns 10.

**Complexity:** Time O(n^3) worst case (n^2 states times up to n choices), Space O(n^2).
**Watch out:** the memo key must contain both `i` and `M`. Do not forget the "take all" shortcut: it also keeps the recursion shallow. M never needs to exceed n.

### 15. Stone Game IV — Hard
**Companies:** Google, Amazon
**Problem:** A pile has `n` stones. On your turn you must remove a **non-zero perfect square** number of stones. The player who cannot move loses. Alice starts. Return true if Alice wins. Example: n = 1 -> true, n = 2 -> false, n = 4 -> true.
**Intuition:** It is the plain win/lose table with moves `{1, 4, 9, 16, ...}`. `win[i]` is true if some square `k*k <= i` leads to a losing `i - k*k`. There is no simple modulo pattern here, so DP it is (this is the one to show that not every game collapses to a trick).
**Approach 1 - Brute force:** recursion without memo: exponential.
**Approach 2 - Optimal:** bottom-up table. Time O(n sqrt n), Space O(n).

| i | squares tried | result |
|---|---|---|
| 1 | 1 -> 0 (lose) | win |
| 2 | 1 -> 1 (win) | lose |
| 3 | 1 -> 2 (lose) | win |
| 4 | 1 -> 3 (win); 4 -> 0 (lose) | win |
| 7 | 1 -> 6 (win), 4 -> 3 (win) | lose |

```js
function winnerSquareGame(n) {
  const win = new Array(n + 1).fill(false);
  for (let i = 1; i <= n; i++)
    for (let k = 1; k * k <= i; k++)
      if (!win[i - k * k]) { win[i] = true; break; } // found a move to a losing state
  return win[n];
}
```

**Complexity:** Time O(n sqrt n), Space O(n).
**Watch out:** the early `break` is only valid because one winning move is enough. Do not fall into the trap of guessing a pattern from a few values (1 true, 2 false, 3 true, 4 true...), there is none that helps.

### 16. Stone Game III — Hard
**Companies:** Google, Amazon
**Problem:** Stones with values (may be negative) are in a row. On a turn a player takes 1, 2 or 3 stones from the **front**. Highest total wins. Return "Alice", "Bob" or "Tie". Example: [1,2,3,7] -> "Bob".
**Intuition:** Score difference DP over positions. `dp[i]` = best (mine - theirs) when it is my turn and stones `i..n-1` remain. If I take `j+1` stones, I gain their sum, and the opponent faces `dp[i+j+1]`.
**Approach 1 - Brute force:** recursion with three choices: O(3^n).
**Approach 2 - Optimal:** bottom-up, `dp[n] = 0`.

```
dp[i] = max over j = 0..2 of ( sum(v[i..i+j]) - dp[i+j+1] )
```

Trace for [1,2,3,7]:

| i | options | dp[i] |
|---|---|---|
| 4 | end | 0 |
| 3 | take 7 | 7 |
| 2 | 3-7 = -4 ; 3+7-0 = 10 | 10 |
| 1 | 2-10 ; 5-7 ; 12-0 | 12 |
| 0 | 1-12 = -11 ; 3-10 = -7 ; 6-7 = -1 | -1 |

`dp[0] = -1 < 0`, so Bob wins.

```js
function stoneGameIII(v) {
  const n = v.length;
  const dp = new Array(n + 1).fill(0);
  for (let i = n - 1; i >= 0; i--) {
    let best = -Infinity, sum = 0;
    for (let j = 0; j < 3 && i + j < n; j++) {
      sum += v[i + j];
      best = Math.max(best, sum - dp[i + j + 1]);
    }
    dp[i] = best;
  }
  return dp[0] > 0 ? 'Alice' : dp[0] < 0 ? 'Bob' : 'Tie';
}
```

**Complexity:** Time O(n), Space O(n) (O(1) with 3 rolling values).
**Watch out:** start `best` at `-Infinity`, not 0, since all-negative piles force you to take something. With [-1,-2,-3] the answer is "Tie" (tested). You must take at least one stone, so the loop always runs once.

### 17. Egg Dropping (Super Egg Drop) — Hard
**Companies:** Google, Amazon, Microsoft, Goldman Sachs
**Problem:** You have `k` identical eggs and a building of `n` floors. There is a threshold floor `f`: eggs thrown from floors above `f` break, from `f` or below do not. A broken egg is gone, an unbroken one can be reused. What is the minimum number of drops that guarantees finding `f`? Example: k = 2, n = 6 -> 3.
**Intuition:** Flip the question. Instead of "how many moves for n floors", ask "with `m` moves and `e` eggs, how many floors can I fully handle?" Call it `f(m, e)`. Drop one egg from some floor. If it breaks, I have `m-1` moves and `e-1` eggs to search the floors below: `f(m-1, e-1)`. If it survives, I search the floors above with `m-1` moves and `e` eggs: `f(m-1, e)`. Plus the floor I dropped from itself. So:

```
f(m, e) = f(m-1, e-1) + f(m-1, e) + 1
```

Keep adding moves until `f(m, k) >= n`.
**Approach 1 - Brute force:** `dp[k][n]` with the min over every first floor: O(k n^2).
**Approach 2 - Better:** binary search on the floor inside that DP: O(k n log n).
**Approach 3 - Optimal:** the moves-based DP above. It takes about O(k log n) steps when `k` is small, and at most `n` moves in the worst case (k = 1).

Trace for k = 2, n = 6 (f[e] per move, updated from high e to low e):

| moves m | f[1] | f[2] |
|---|---|---|
| 0 | 0 | 0 |
| 1 | 1 | 1 |
| 2 | 2 | 3 |
| 3 | 3 | 6 |

At m = 3, `f[2] = 6 >= 6`, so the answer is 3.

```js
function superEggDrop(k, n) {
  const f = new Array(k + 1).fill(0); // f[e] = floors coverable with e eggs and m moves
  let m = 0;
  while (f[k] < n) {
    m++;
    for (let e = k; e >= 1; e--) f[e] = f[e] + f[e - 1] + 1; // high to low: uses old values
  }
  return m;
}
```

**Complexity:** Time O(k * answer), which is O(k log n) for larger k and O(n) for k = 1; Space O(k).
**Watch out:** update `e` from high to low, or you read already-updated values. Tested: (1,2) -> 2, (2,6) -> 3, (3,14) -> 4, (2,100) -> 14, (1,100) -> 100. With one egg you are forced to go floor by floor: `n` drops.

## Common Mistakes
- **"I will just add my score and subtract theirs at the end."** Track only the difference; it removes one dimension from the state. And do not forget the minus sign on the opponent's result.
- **"The opponent plays randomly, so take the average."** Wrong. Assume the best possible opponent. Optimal play is the contract of every one of these problems.
- **Forgetting the base case "no move means you lose".** For Nim and Divisor Game, the empty position must be a LOSS for the player to move.
- **Guessing a modulo pattern from two data points.** Print at least 10-15 values of the win table and confirm. Stone Game IV looks patterned and is not.
- **Bad memo keys.** Can I Win works with just the bitmask because the total is derived from it. For Stone Game II you need `(i, M)`. A missing component gives wrong answers that look plausible.
- **Mixing 0-indexed and 1-indexed in Josephus.** The recurrence is 0-indexed; add 1 only at the end.
- **Skipping verification in Find the Celebrity.** The candidate pass only yields a possibility.
- **Treating "guarantee" problems as expected-value problems.** Guess Number II and Egg Drop ask for the worst case: min over your choices of the max over the adversary's choices.
- **Believing simulations prove probabilities.** They are a sanity check; the proof is the counting argument.

## Interview Tips
- Start by saying the definition out loud: "A position is winning if some move reaches a losing position for the opponent."
- Offer the brute-force recursion first, then memoize, then say "let me print the table for small n to look for a pattern". This shows method, not luck.
- For score games, write down `dp[state] = best (me - opponent)` and the one-line recurrence with the minus sign.
- State the assumption: both players are optimal, perfect information, no randomness.
- Edge cases to mention: n = 0 or 1, empty array, all negative values (Stone Game III), a target of 0 (Can I Win, Water Jug), ties (>= vs >).
- Follow-ups you may get:
  - "What if there are several piles?" -> Nim, XOR of the piles (and Sprague-Grundy for non-standard moves).
  - "What if you may take 1 to k stones?" -> `n % (k+1)`.
  - "Can you return the actual winning move?" -> keep the move that achieved the max or the first move that reaches a losing state.
  - "What if the game can loop?" -> retrograde analysis (Cat and Mouse).
  - "Prove it." -> induction on the position: show every move from a "losing" position goes to a "winning" one, and from a "winning" one there is a move to a "losing" one.
- For puzzles, explain the structure before the code: "toggled once per divisor, divisors pair up, so squares are the odd ones out."
- If the pattern is unproven and you must pick, ship the DP.

## Practice List
**Easy**
- Nim Game
- Divisor Game
- Flip Game
- Bulb Switcher (the squares answer)
- Find the Town Judge (similar elimination idea to the celebrity)

**Medium**
- Predict the Winner
- Stone Game
- Stone Game II
- Flip Game II
- Can I Win
- Guess Number Higher or Lower II
- Water and Jug Problem
- Find the Celebrity
- Find the Winner of the Circular Game
- Bulb Switcher II
- Super Egg Drop (also appears as Hard)
- Coins in a Line (LintCode)

**Hard**
- Stone Game III
- Stone Game IV
- Stone Game V
- Stone Game VI
- Stone Game VII
- Stone Game VIII
- Stone Game IX
- Cat and Mouse
- Cat and Mouse II

## Key Takeaways
- A position is **winning if any move reaches a losing position**; losing if every move reaches a winning one.
- For scoring games, use the **score difference DP**: `dp = max(gain - dp(next))`. The minus sign handles the optimal opponent.
- **Brute force small cases, print the table, find the pattern, then prove it.** Nim (`n % 4`), Divisor Game (even), Stone Game I (always true) all fall this way.
- **Nim is XOR**; Sprague-Grundy extends it to any impartial game with `mex` of the next positions' Grundy numbers.
- A **bitmask of used items** is the memo key for "pick each number once" games like Can I Win.
- Many puzzles hide a counting fact: **divisors pair up** (bulbs, doors), **gcd** controls reachable amounts (jugs), `2^n - 1` for Hanoi, a shifting recurrence for Josephus.
- Worst-case guarantees (Guess Number II, Egg Drop) are **min over your choice of max over the adversary**; flipping the question ("how many floors can m moves cover?") often makes it cheap.

---
[← Previous: Data Structure Internals](../06-data-structure-internals/README.md) | [Next: Interview Playbook →](../../07-interview-playbook/README.md) | [Back to Advanced Index](../README.md)
