# Linked List

> **Estimated Time:** 6 hr | **Difficulty:** Intermediate
> **Prerequisites:** [03 - JavaScript for DSA](../../00-foundations/03-javascript-for-dsa/README.md), [05 - Two Pointers](../05-two-pointers/README.md)

---

## What You'll Learn
- You can explain how singly, doubly and circular lists are stored, and what each operation really costs.
- You can use a **dummy node** to remove special cases for the head.
- You can use **fast and slow pointers** to find the middle, detect a cycle and find where the cycle starts.
- You can reverse a whole list, part of a list, or every group of k nodes, by rewiring `next` pointers.
- You can solve the classic 22 interview problems for this topic in O(1) extra space where possible.

## The Big Idea
Think of a treasure hunt. Each clue tells you only where the next clue is. You cannot jump to clue number 7. You must follow clues 1 to 6 first. But if you want to insert a new clue between 3 and 4, you only change what clue 3 says. Nobody has to move.

That is a linked list. A node holds a value and a pointer (`next`). Compared to an array:

| Operation | Array | Singly linked list |
|---|---|---|
| Read by index | O(1) | O(n) |
| Insert/delete at a known node | O(n) (shift) | O(1) |
| Insert/delete at head | O(n) | O(1) |
| Memory | one block | one object per node + pointer |

Variants:
- **Singly:** `next` only. You can only walk forward.
- **Doubly:** `prev` and `next`. You can delete a node in O(1) if you hold the node itself (no need to find the previous one). Used in LRU caches.
- **Circular:** the last node points back to the first (or into the middle, which is a "cycle"). A loop that waits for `null` never ends here.

Why interviewers love it: there is almost no algorithm to memorize. The whole problem is careful pointer handling. It shows whether you can hold a small mental picture, handle `null`, and not lose a node while rewiring.

```
head -> [1] -> [2] -> [3] -> null
         val   val   val
```

JavaScript has no built-in linked list. In interviews you get this class (same helper style used in the foundations chapter). `fromArray` and `toArray` are only for testing.

```js
class ListNode {
  constructor(val = 0, next = null) { this.val = val; this.next = next; }
}
const fromArray = (a) => {
  const dummy = new ListNode(); let t = dummy;
  for (const v of a) { t.next = new ListNode(v); t = t.next; }
  return dummy.next;
};
const toArray = (head) => {
  const out = [];
  for (; head; head = head.next) out.push(head.val);
  return out;
};
```

## How to Spot It
| If the problem says ... | think ... |
|---|---|
| "middle of the list", "split in half" | fast/slow pointers |
| "cycle", "loop", "repeats", "find duplicate number" | Floyd fast/slow |
| "nth from the end" | two pointers with a gap of n |
| "reverse", "palindrome", "reorder" | reverse the second half in place |
| head may be removed or changed | dummy node |
| "merge sorted lists" | dummy node + tail pointer |
| "k lists" | min-heap of the k current heads |
| "O(1) extra space" | rewire pointers, no array copy, no hash set |
| "given only the node to delete" | copy the next node's value over this one |
| "copy a list with extra pointers" | interleave copies or a hash map old -> new |

## Core Concepts / Template

### 1. Dummy node
If the head can be deleted or replaced, the code for "head" differs from "middle". A dummy node placed before the head makes every node have a previous node. Return `dummy.next` at the end.

```js
function template(head) {
  const dummy = new ListNode(0, head);
  let prev = dummy;
  while (prev.next) {
    // look at prev.next, decide to keep or remove it
    prev = prev.next;
  }
  return dummy.next;
}
```

### 2. Fast and slow pointers
Slow moves 1 step, fast moves 2. When fast reaches the end, slow is in the middle. If there is a cycle, fast laps slow and they meet inside the loop.

```js
let slow = head, fast = head;
while (fast && fast.next) {   // check BOTH, or you crash on null.next
  slow = slow.next;
  fast = fast.next.next;
}
// slow is the middle (second middle if the length is even)
```

Trace on `1 -> 2 -> 3 -> 4 -> 5`:

| step | slow | fast |
|---|---|---|
| start | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | 5 |
| fast.next is null, stop | 3 | 5 |

### 3. Reversing pointers
Keep three names: `prev`, `cur`, `nx`. Save `nx` first, flip `cur.next`, then step forward. Never flip before saving, or the rest of the list is lost.

```
prev=null  cur=1->2->3
step 1: save nx=2, 1.next=null, prev=1, cur=2
step 2: save nx=3, 2.next=1,    prev=2, cur=3
step 3: save nx=null, 3.next=2, prev=3, cur=null   -> return prev (3->2->1)
```

### 4. Pointer rules that save you
- Draw the picture. Write the pointer changes in order before you code.
- Before every `node.next.next`, ask: can `node.next` be null?
- When you rewire, first save what you are about to overwrite.
- Cut cleanly: when you split a list, set the tail's `next = null`, or you get an accidental cycle.

## Problems (easy -> hard)

All code below was run in Node with assert tests on the examples and on edge cases (empty list, one node, all equal values, cycles, even/odd length).

### 1. Reverse Linked List — Easy
**Companies:** Amazon, Microsoft, Apple, Google, Meta
**Problem:** Reverse a singly linked list and return the new head. Example: `1->2->3->4->5` -> `5->4->3->2->1`.
**Intuition:** Walk once. At each node, point `next` backward instead of forward. Remember the old next first.
**Approach 1 - Brute force:** Copy values into an array, write them back reversed. Time O(n), Space O(n).
**Approach 2 - Better:** Recursion: reverse the rest, then make the next node point back to me. Time O(n), Space O(n) call stack.
**Approach 3 - Optimal:** Iterative with three pointers (trace in Core Concepts). O(1) space.

```js
function reverseList(head) {
  let prev = null, cur = head;
  while (cur) {
    const nx = cur.next; // save
    cur.next = prev;     // flip
    prev = cur;          // move both
    cur = nx;
  }
  return prev;
}

// recursive version
function reverseListRec(head) {
  if (!head || !head.next) return head;
  const newHead = reverseListRec(head.next);
  head.next.next = head; // the node after me now points to me
  head.next = null;      // I become the tail (avoids a cycle)
  return newHead;
}
```

**Complexity:** Iterative Time O(n), Space O(1). Recursive Time O(n), Space O(n).
**Watch out:** In the recursive version forgetting `head.next = null` creates a 2-node cycle. Recursion depth can overflow the stack for very long lists.

### 2. Middle of the Linked List — Easy
**Companies:** Amazon, Microsoft, Apple
**Problem:** Return the middle node. If there are two middles, return the second. Example: `1->2->3->4->5->6` -> node `4`.
**Intuition:** Two runners, one twice as fast. When the fast one finishes, the slow one is halfway.
**Approach 1 - Brute force:** Count length n, then walk n/2 steps. Two passes. Time O(n), Space O(1).
**Approach 3 - Optimal:** Fast/slow in one pass. On `1..6`: slow 1,2,3,4 and fast 1,3,5,null -> slow = 4.

```js
function middleNode(head) {
  let slow = head, fast = head;
  while (fast && fast.next) { slow = slow.next; fast = fast.next.next; }
  return slow;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** For the first middle in an even list (needed when splitting for sort), start `fast = head.next`.

### 3. Merge Two Sorted Lists — Easy
**Companies:** Amazon, Microsoft, Google, Meta, Apple
**Problem:** Merge two sorted lists into one sorted list by relinking the nodes. Example: `1->2->4` and `1->3->4` -> `1->1->2->3->4->4`.
**Intuition:** Like merging two sorted piles of cards: always take the smaller top card. A dummy node avoids the "who is the first node" special case.
**Approach 1 - Brute force:** Put all values in an array, sort, rebuild. Time O((n+m) log(n+m)), Space O(n+m).
**Approach 3 - Optimal:** Tail pointer. Compare heads, attach the smaller, advance. When one list ends, attach the other whole.

```js
function mergeTwoLists(a, b) {
  const dummy = new ListNode();
  let tail = dummy;
  while (a && b) {
    if (a.val <= b.val) { tail.next = a; a = a.next; } // <= keeps it stable
    else { tail.next = b; b = b.next; }
    tail = tail.next;
  }
  tail.next = a || b; // attach the leftover in O(1)
  return dummy.next;
}
```

**Complexity:** Time O(n + m), Space O(1).
**Watch out:** Forgetting to attach the leftover list. Either list can be null.

### 4. Delete Node in a Linked List — Easy
**Companies:** Amazon, Microsoft, Apple
**Problem:** You are given only the node to delete (never the tail), not the head. Delete it. Example: `4->5->1->9`, node `5` -> `4->1->9`.
**Intuition:** You cannot reach the previous node. So do not delete this node: become the next node. Copy its value, then skip it.
**Approach 3 - Optimal:** `node.val = node.next.val; node.next = node.next.next`.

```js
function deleteNode(node) {
  node.val = node.next.val;
  node.next = node.next.next;
}
```

**Complexity:** Time O(1), Space O(1).
**Watch out:** Does not work for the tail (that is why the problem forbids it). It changes node identity, so it is a trick, not a general delete.

### 5. Remove Duplicates from Sorted List — Easy
**Companies:** Amazon, Microsoft, Meta
**Problem:** Keep one copy of each value in a sorted list. Example: `1->1->2->3->3` -> `1->2->3`.
**Intuition:** Duplicates sit next to each other. If my next has my value, skip it.
**Approach 3 - Optimal:** Only advance when the next value differs.

```js
function deleteDuplicates(head) {
  let cur = head;
  while (cur && cur.next) {
    if (cur.val === cur.next.val) cur.next = cur.next.next; // skip, stay here
    else cur = cur.next;
  }
  return head;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Do not advance after a skip: `1->1->1` needs several skips from the same node. The head never changes here, so no dummy needed.

### 6. Linked List Cycle — Easy
**Companies:** Amazon, Microsoft, Google, Meta, Bloomberg
**Problem:** Return true if following `next` ever loops back. Example: `3->2->0->-4` with `-4` pointing to `2` -> true.
**Intuition:** On a circular track, a faster runner eventually laps a slower one. On a straight road, the fast one just finishes.
**Approach 1 - Brute force:** Store visited nodes in a Set; if you see one twice, there is a cycle. Time O(n), Space O(n).
**Approach 3 - Optimal:** Floyd's tortoise and hare. Fast gains 1 node per step on slow, so inside a loop of length L they meet within L steps.

```js
function hasCycle(head) {
  let slow = head, fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) return true; // compare nodes, not values
  }
  return false;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Compare node references, not `val` (values repeat).

### 7. Intersection of Two Linked Lists — Easy
**Companies:** Amazon, Microsoft, Meta, Google, Apple
**Problem:** Two lists may share a tail (same nodes, not equal values). Return the first shared node or null. Example: `A = 4->1->8->4->5`, `B = 5->6->1->8->4->5`, shared from `8` -> node `8`.
**Intuition:** Let each pointer walk its own list, then switch to the other list's head. Both then travel (lenA + lenB) in total, so they line up at the shared part, or both hit null together.
**Approach 1 - Brute force:** For each node in A, scan all of B. Time O(n*m), Space O(1).
**Approach 2 - Better:** Put A's nodes in a Set, scan B. Time O(n+m), Space O(n).
**Approach 3 - Optimal:** Two pointers switching heads.

Trace with A = `a1 a2 c1 c2`, B = `b1 c1 c2`:

| step | p | q |
|---|---|---|
| 0 | a1 | b1 |
| 1 | a2 | c1 |
| 2 | c1 | c2 |
| 3 | c2 | null |
| 4 | null | a1 |
| 5 | b1 | a2 |
| 6 | c1 | c1 -> equal, return c1 |

```js
function getIntersectionNode(a, b) {
  let p = a, q = b;
  while (p !== q) {
    p = p ? p.next : b; // switch after falling off the end
    q = q ? q.next : a;
  }
  return p; // shared node, or null if none (both reach null together)
}
```

**Complexity:** Time O(n + m), Space O(1).
**Watch out:** Switch when the pointer becomes null (after one null step), not before; that null step is what makes disjoint lists end cleanly.

### 8. Palindrome Linked List — Easy
**Companies:** Amazon, Microsoft, Meta, Apple
**Problem:** Is the list the same forwards and backwards? Example: `1->2->2->1` -> true.
**Intuition:** A singly list cannot be read backward. So reverse the second half in place, then walk the first half and the reversed second half together and compare.
**Approach 1 - Brute force:** Copy to an array and check with two pointers. Time O(n), Space O(n).
**Approach 3 - Optimal:** Find the middle, reverse from the middle, compare, then (politely) reverse back.

Trace `1->2->2->1`: slow ends at second `2`. Reverse from there: `1->2`. Compare `1,2` with `1,2`. Match.

```js
function isPalindrome(head) {
  let slow = head, fast = head;
  while (fast && fast.next) { slow = slow.next; fast = fast.next.next; }
  const half = reverseList(slow); // reverseList from problem 1
  let l = head, r = half, ok = true;
  while (r) {
    if (l.val !== r.val) { ok = false; break; }
    l = l.next; r = r.next;
  }
  reverseList(half); // restore the input
  return ok;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** For odd length, the middle goes in the second half; comparing only while `r` exists skips it correctly. Mention that you restore the list (interviewers like that).

### 9. Remove Nth Node From End of List — Medium
**Companies:** Amazon, Meta, Microsoft, Google, Apple
**Problem:** Remove the nth node from the end in one pass. Example: `1->2->3->4->5`, n=2 -> `1->2->3->5`.
**Intuition:** Keep a gap of n nodes between two pointers. When the front one reaches the end, the back one stands right before the node to delete.
**Approach 1 - Brute force:** Count length, then walk `len - n` steps. Two passes. Time O(n), Space O(1).
**Approach 3 - Optimal:** Dummy node, `fast` moves n steps ahead, then both move until `fast.next` is null.

Trace `1->2->3->4->5`, n=2, dummy `D`: fast goes D->1->2. Then move both: (fast 3, slow 1), (4, 2), (5, 3). fast.next is null, slow = 3. Delete `slow.next` (4).

```js
function removeNthFromEnd(head, n) {
  const dummy = new ListNode(0, head);
  let fast = dummy, slow = dummy;
  for (let i = 0; i < n; i++) fast = fast.next;
  while (fast.next) { fast = fast.next; slow = slow.next; }
  slow.next = slow.next.next;
  return dummy.next;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Removing the head (n equals length) works only because of the dummy node.

### 10. Linked List Cycle II (start and length of the cycle) — Medium
**Companies:** Amazon, Microsoft, Meta, Google, Bloomberg
**Problem:** Return the node where the cycle begins, or null. Also be ready to give the cycle length.
**Intuition (the math):** Let `a` = distance head to cycle start, `L` = cycle length, `b` = how far into the cycle slow is when they meet. Slow walked `a + b`, fast walked `2(a + b)`. Fast walked `a + b` more than slow, which is a whole number of laps, so `a + b = k*L`. So `a = k*L - b`: from the meeting point, going `a` more steps lands exactly on the cycle start. So: one pointer from head, one from the meeting point, move both 1 step at a time; they meet at the start.
**Approach 1 - Brute force:** Set of visited nodes; first repeated node is the start. Time O(n), Space O(n).
**Approach 3 - Optimal:** Floyd phase 1 (meet), phase 2 (reset one pointer to head).

```js
function detectCycle(head) {
  let slow = head, fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) {
      let p = head;
      while (p !== slow) { p = p.next; slow = slow.next; }
      return p; // cycle start
    }
  }
  return null;
}

// cycle length: after they meet, walk once around
function cycleLength(head) {
  let slow = head, fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) {
      let n = 1;
      for (let p = slow.next; p !== slow; p = p.next) n++;
      return n;
    }
  }
  return 0; // no cycle
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Both phases move one step at a time in phase 2. The same trick finds the duplicate in "Find the Duplicate Number" (treat `nums[i]` as a `next` pointer).

### 11. Add Two Numbers — Medium
**Companies:** Amazon, Microsoft, Google, Meta, Apple
**Problem:** Two non-negative numbers are stored as lists with digits in reverse order (ones digit first). Return their sum as a list. Example: `2->4->3` + `5->6->4` -> `7->0->8` (342 + 465 = 807).
**Intuition:** It is school addition. Reverse order is a gift: you start at the ones digit, which is the head. Keep a carry.
**Approach 1 - Brute force:** Convert to numbers and add. Breaks: lists can be longer than a double can hold exactly (use BigInt, which works but misses the point).
**Approach 3 - Optimal:** Walk both lists together. Loop while either list or the carry remains.

```js
function addTwoNumbers(a, b) {
  const dummy = new ListNode();
  let tail = dummy, carry = 0;
  while (a || b || carry) {
    const sum = (a ? a.val : 0) + (b ? b.val : 0) + carry;
    tail.next = new ListNode(sum % 10);
    carry = Math.floor(sum / 10);
    tail = tail.next;
    a = a && a.next;
    b = b && b.next;
  }
  return dummy.next;
}
```

**Complexity:** Time O(max(n, m)), Space O(1) extra (output list not counted).
**Watch out:** A final carry makes a new node: `9->9->9` + `1` = `0->0->0->1`. Follow-up (digits in forward order): reverse both first, or use two stacks.

### 12. Odd Even Linked List — Medium
**Companies:** Amazon, Microsoft, Meta
**Problem:** Group nodes at odd positions first, then even positions, keeping the relative order. This is about positions, not values. Example: `1->2->3->4->5` -> `1->3->5->2->4`.
**Intuition:** Build two chains while walking: an odd chain and an even chain. Then glue even after odd.
**Approach 3 - Optimal:** Keep `odd`, `even` and remember `evenHead`.

```js
function oddEvenList(head) {
  if (!head) return head;
  let odd = head, even = head.next;
  const evenHead = even;
  while (even && even.next) {
    odd.next = even.next;  odd = odd.next;
    even.next = odd.next;  even = even.next;
  }
  odd.next = evenHead;
  return head;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** The loop condition `even && even.next` covers both odd and even lengths. Forgetting `evenHead` loses the even chain.

### 13. Remove Duplicates from Sorted List II — Medium
**Companies:** Amazon, Microsoft, Meta, Google
**Problem:** Remove every value that appears more than once (delete all copies). Example: `1->2->3->3->4->4->5` -> `1->2->5`.
**Intuition:** The head itself might be a duplicate, so use a dummy. `prev` always points at the last node we are sure to keep. If the current value repeats, skip the entire run.
**Approach 3 - Optimal:** One pass with `prev` and `head` as the current node.

```js
function deleteDuplicatesII(head) {
  const dummy = new ListNode(0, head);
  let prev = dummy;
  while (head) {
    if (head.next && head.val === head.next.val) {
      const v = head.val;
      while (head && head.val === v) head = head.next; // skip whole run
      prev.next = head;
    } else {
      prev = head;
      head = head.next;
    }
  }
  return dummy.next;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Do not move `prev` when you skip a run: the node after the run may be a duplicate run too. Compare with problem 5 (keeps one copy).

### 14. Rotate List — Medium
**Companies:** Amazon, Microsoft, Google, Meta
**Problem:** Rotate the list to the right by k places. Example: `1->2->3->4->5`, k=2 -> `4->5->1->2->3`.
**Intuition:** Rotating by k means the last k nodes move to the front. Join the tail to the head (a circle), then cut the circle at the right place.
**Approach 1 - Brute force:** Rotate by 1, k times. Time O(n*k).
**Approach 3 - Optimal:** Find length and tail. `k %= len`. The new tail is at index `len - k - 1`. Cut there.

Trace `1..5`, k=2: len=5, new tail = node 3, new head = 4. Cut `3->null`, `5->1`. Result `4->5->1->2->3`.

```js
function rotateRight(head, k) {
  if (!head || !head.next || k === 0) return head;
  let len = 1, tail = head;
  while (tail.next) { tail = tail.next; len++; }
  k %= len;
  if (k === 0) return head;
  let p = head;
  for (let i = 0; i < len - k - 1; i++) p = p.next;
  const newHead = p.next;
  p.next = null;
  tail.next = head;
  return newHead;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** k can be far larger than the length. Always use `k % len`, and return early when it is 0.

### 15. Reverse Linked List II (between m and n) — Medium
**Companies:** Amazon, Microsoft, Meta, Google
**Problem:** Reverse only the nodes from position `left` to `right` (1-indexed). Example: `1->2->3->4->5`, left=2, right=4 -> `1->4->3->2->5`.
**Intuition:** Stand just before the segment (`pre`). The first node of the segment (`cur`) stays in place but slides to the back of the segment. Repeatedly take the node after `cur` and insert it right after `pre`.
**Approach 1 - Brute force:** Copy the segment values into an array, reverse them, write back. Time O(n), Space O(n).
**Approach 3 - Optimal:** Head insertion, one pass, O(1) space.

Trace `1->2->3->4->5`, left=2, right=4, `pre`=1, `cur`=2:
- move 3 to after pre: `1->3->2->4->5`
- move 4 to after pre: `1->4->3->2->5`

```js
function reverseBetween(head, left, right) {
  const dummy = new ListNode(0, head);
  let pre = dummy;
  for (let i = 1; i < left; i++) pre = pre.next;
  const cur = pre.next; // stays; it ends up as the segment's tail
  for (let i = 0; i < right - left; i++) {
    const nx = cur.next;
    cur.next = nx.next;     // unlink nx
    nx.next = pre.next;     // put nx at the front of the segment
    pre.next = nx;
  }
  return dummy.next;
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** `left = 1` changes the head: the dummy handles it. `left === right` does nothing.

### 16. Reorder List — Medium
**Companies:** Amazon, Microsoft, Meta, Google
**Problem:** Reorder `L0->L1->...->Ln` into `L0->Ln->L1->Ln-1->...` in place. Example: `1->2->3->4` -> `1->4->2->3`.
**Intuition:** This is three earlier tools glued together: find the middle, reverse the second half, merge by alternating.
**Approach 1 - Brute force:** Put nodes in an array, then rewire with two indices. Time O(n), Space O(n).
**Approach 3 - Optimal:** Middle, reverse, interleave.

Trace `1->2->3->4->5`: first half `1->2->3`, second half reversed `5->4`. Interleave: `1->5->2->4->3`.

```js
function reorderList(head) {
  if (!head || !head.next) return;
  let slow = head, fast = head;
  while (fast.next && fast.next.next) { slow = slow.next; fast = fast.next.next; }
  let b = reverseList(slow.next); // second half, reversed
  slow.next = null;               // cut the first half
  let a = head;
  while (b) {
    const an = a.next, bn = b.next;
    a.next = b;
    b.next = an;
    a = an;
    b = bn;
  }
}
```

**Complexity:** Time O(n), Space O(1).
**Watch out:** Cutting with `slow.next = null` is required. This loop makes the first half the longer one for odd lengths, which is what the problem wants. The function returns nothing; it edits in place.

### 17. Sort List — Medium
**Companies:** Amazon, Microsoft, Meta, Google, Bloomberg
**Problem:** Sort a linked list in ascending order. Follow-up: O(n log n) time. Example: `4->2->1->3` -> `1->2->3->4`.
**Intuition:** Quick sort and heap sort need random access. Merge sort does not: splitting is "find the middle" and merging is problem 3.
**Approach 1 - Brute force:** Copy to an array, sort, rebuild. Time O(n log n), Space O(n).
**Approach 3 - Optimal:** Top-down merge sort. `fast` starts one step ahead so `slow` ends at the first middle, which guarantees both halves are smaller than the whole (no infinite recursion on 2 nodes).

```js
function sortList(head) {
  if (!head || !head.next) return head;
  let slow = head, fast = head.next;
  while (fast && fast.next) { slow = slow.next; fast = fast.next.next; }
  const right = slow.next;
  slow.next = null;                 // split
  return mergeTwoLists(sortList(head), sortList(right)); // from problem 3
}
```

**Complexity:** Time O(n log n), Space O(log n) recursion stack. A bottom-up merge sort gets O(1) space but is much longer to write; mention it, do not code it unless asked.
**Watch out:** If `fast` starts at `head`, a 2-node list splits into 2 and 0 nodes and recurses forever.

### 18. Flatten a Multilevel Doubly Linked List — Medium
**Companies:** Amazon, Microsoft, Meta, Google, Bloomberg
**Problem:** Each node has `prev`, `next` and an optional `child` list. Flatten into one list: a child list goes right after its parent, before the parent's old next. All `child` pointers become null. Example: `1-2-3-4-5-6` with `3.child = 7-8-9` and `8.child = 11-12` -> `1,2,3,7,8,11,12,9,4,5,6`.
**Intuition:** Walk the list. At a node with a child, remember the old `next` on a stack, plug the child in. When a chain ends, pop the saved next and attach it.
**Approach 1 - Brute force:** Recursive DFS that returns the tail of each flattened child. Time O(n), Space O(depth).
**Approach 3 - Optimal:** Iterative with an explicit stack (shown). Fixing `prev` pointers is half of the job.

```js
function flatten(head) {
  const stack = [];
  let cur = head;
  while (cur) {
    if (cur.child) {
      if (cur.next) stack.push(cur.next); // come back here later
      cur.next = cur.child;
      cur.child.prev = cur;
      cur.child = null;
    } else if (!cur.next && stack.length) {
      const nx = stack.pop();
      cur.next = nx;
      nx.prev = cur;
    }
    cur = cur.next;
  }
  return head;
}
```

**Complexity:** Time O(n), Space O(depth of nesting) for the stack.
**Watch out:** Set `child = null` and fix `prev` on every join. The node class has four fields: `val, prev, next, child`.

### 19. Copy List with Random Pointer — Medium
**Companies:** Amazon, Microsoft, Meta, Google, Bloomberg
**Problem:** Each node has `next` and `random` (any node or null). Make a deep copy (all new nodes). Example: a 5-node list where random pointers jump around -> a new list with the same shape and no shared nodes.
**Intuition:** The hard part is: "which new node matches this old node?" Either store the mapping in a Map, or weave each copy right after its original so the mapping is just `old.next`.
**Approach 1 - Brute force / Better:** Two passes with a Map old -> new. Pass 1 creates nodes; pass 2 sets `next` and `random` via the map. Time O(n), Space O(n).
**Approach 3 - Optimal:** Weave copies in, set randoms, then unweave.

Weave: `A -> A' -> B -> B' -> C -> C'`. Random: `A'.random = A.random.next`. Unweave: restore `A->B->C` and extract `A'->B'->C'`.

```js
class RNode {
  constructor(val, next = null, random = null) { this.val = val; this.next = next; this.random = random; }
}

function copyRandomList(head) {
  if (!head) return null;
  for (let c = head; c; c = c.next.next) c.next = new RNode(c.val, c.next); // 1. weave
  for (let c = head; c; c = c.next.next) {                                  // 2. randoms
    if (c.random) c.next.random = c.random.next;
  }
  const newHead = head.next;
  for (let c = head; c; c = c.next) {                                       // 3. unweave
    const copy = c.next;
    c.next = copy.next;
    copy.next = copy.next && copy.next.next;
  }
  return newHead;
}
```

**Complexity:** Time O(n), Space O(1) extra (the copy is the output).
**Watch out:** Set randoms before unweaving, never after. Restore the original list (step 3 does). In an interview, the Map version is fine to code first; offer the weave as the O(1) improvement.

### 20. Doubly Linked List: Insert and Delete — Medium
**Companies:** -
**Problem:** Build a doubly linked list with O(1) insert after a node, add at front or back, and delete a given node. This is the engine inside an LRU cache and a browser history.
**Intuition:** Use two sentinel nodes, `head` and `tail`, that never hold data. Then every real node has both a `prev` and a `next`, and there are no null checks at the ends.
**Approach 3 - Optimal:** Four pointer writes per operation. Order matters: set the new node's pointers first, then update the neighbours.

Insert `n` after `x` (with `y = x.next`): `n.prev = x; n.next = y; y.prev = n; x.next = n`.
Delete `n`: `n.prev.next = n.next; n.next.prev = n.prev`.

```js
class DNode {
  constructor(val, prev = null, next = null, child = null) {
    this.val = val; this.prev = prev; this.next = next; this.child = child;
  }
}

class DoublyList {
  constructor() {
    this.head = new DNode(null); // sentinels
    this.tail = new DNode(null);
    this.head.next = this.tail;
    this.tail.prev = this.head;
    this.size = 0;
  }
  insertAfter(node, val) {
    const n = new DNode(val, node, node.next);
    node.next.prev = n;
    node.next = n;
    this.size++;
    return n;
  }
  addFirst(v) { return this.insertAfter(this.head, v); }
  addLast(v) { return this.insertAfter(this.tail.prev, v); }
  remove(node) { // O(1): needs the node, not the value
    node.prev.next = node.next;
    node.next.prev = node.prev;
    this.size--;
  }
  toArray() {
    const out = [];
    for (let c = this.head.next; c !== this.tail; c = c.next) out.push(c.val);
    return out;
  }
}
```

**Complexity:** Time O(1) per insert/delete given the node, Space O(1) per node (two pointers each).
**Watch out:** Never remove a sentinel. Finding a node by value is still O(n). A doubly list costs one extra pointer per node, which is why singly lists win when you only go forward.

### 21. Reverse Nodes in k-Group — Hard
**Companies:** Amazon, Microsoft, Google, Meta, Apple
**Problem:** Reverse the nodes k at a time. If fewer than k remain at the end, leave them as is. Example: `1->2->3->4->5`, k=2 -> `2->1->4->3->5`; k=3 -> `3->2->1->4->5`.
**Intuition:** Repeat a simple job: check that k nodes exist, reverse exactly those k, and reconnect to the groups on both sides. Reversing a segment is problem 1, with a small twist: the first node of the segment should point to whatever follows the segment.
**Approach 1 - Brute force:** Dump to an array, reverse chunks, rebuild. Time O(n), Space O(n).
**Approach 3 - Optimal:** For each group, find the k-th node (`kth`). If none, stop. Reverse the group with `prev` starting at `nextGroup` (so the old first node ends up pointing to the next group automatically). Connect the previous group's tail to `kth`.

Trace `1->2->3->4->5`, k=2: group (1,2): kth=2, nextGroup=3. Reverse with prev=3: `2->1->3`. The dummy now points to 2; `prevGroup` = 1. Group (3,4): kth=4, nextGroup=5, becomes `4->3->5`, and `1.next = 4`. Group (5): only one node left, stop. Result `2->1->4->3->5`.

```js
function reverseKGroup(head, k) {
  const dummy = new ListNode(0, head);
  let prevGroup = dummy;
  while (true) {
    let kth = prevGroup;
    for (let i = 0; i < k && kth; i++) kth = kth.next;
    if (!kth) break;                       // fewer than k left
    const nextGroup = kth.next;
    let prev = nextGroup, cur = prevGroup.next;
    while (cur !== nextGroup) {            // reverse exactly k nodes
      const nx = cur.next;
      cur.next = prev;
      prev = cur;
      cur = nx;
    }
    const first = prevGroup.next;          // old first = new tail of this group
    prevGroup.next = kth;                  // kth = new head of this group
    prevGroup = first;
  }
  return dummy.next;
}
```

**Complexity:** Time O(n) (each node touched about twice), Space O(1).
**Watch out:** Always check that k nodes exist before reversing, otherwise the leftover gets reversed by mistake. `k = 1` returns the list unchanged. Save `first` before relinking `prevGroup.next`.

### 22. Merge k Sorted Lists — Hard
**Companies:** Amazon, Google, Meta, Microsoft, Uber
**Problem:** Merge k sorted lists into one sorted list. Example: `[1->4->5, 1->3->4, 2->6]` -> `1->1->2->3->4->4->5->6`.
**Intuition:** At every step, the next output node is the smallest among the k current heads. A min-heap gives that smallest in O(log k).
**Approach 1 - Brute force:** Merge lists one after another into a result. Total Time O(k * N) in the worst case (the result keeps growing). Space O(1).
**Approach 2 - Better:** Divide and conquer: merge lists in pairs (round 1: k/2 merges, round 2: k/4, ...). Time O(N log k), Space O(log k) for recursion, or O(1) done iteratively with the problem 3 merge.
**Approach 3 - Optimal (heap):** Put each list's head in a min-heap. Pop the smallest, append it, push its `next`. Heap size never exceeds k. JavaScript has no built-in heap, so a small one is included; in an interview, say "I will assume a min-heap" and write it only if asked.

Trace `[1->4->5, 1->3->4, 2->6]`: heap {1a, 1b, 2}. Pop 1a, push 4 -> output 1. Pop 1b, push 3 -> output 1,1. Pop 2, push 6 -> 1,1,2. Pop 3, push 4 -> 1,1,2,3. And so on.

```js
class MinHeap { // compares node.val
  constructor() { this.a = []; }
  get size() { return this.a.length; }
  push(x) {
    const a = this.a;
    a.push(x);
    let i = a.length - 1;
    while (i > 0) {
      const p = (i - 1) >> 1;
      if (a[p].val <= a[i].val) break;
      [a[p], a[i]] = [a[i], a[p]];
      i = p;
    }
  }
  pop() {
    const a = this.a, top = a[0], last = a.pop();
    if (a.length) {
      a[0] = last;
      let i = 0;
      for (;;) {
        const l = 2 * i + 1, r = l + 1;
        let m = i;
        if (l < a.length && a[l].val < a[m].val) m = l;
        if (r < a.length && a[r].val < a[m].val) m = r;
        if (m === i) break;
        [a[m], a[i]] = [a[i], a[m]];
        i = m;
      }
    }
    return top;
  }
}

function mergeKLists(lists) {
  const heap = new MinHeap();
  for (const l of lists) if (l) heap.push(l); // skip empty lists
  const dummy = new ListNode();
  let tail = dummy;
  while (heap.size) {
    const n = heap.pop();
    tail.next = n;
    tail = n;
    if (n.next) heap.push(n.next);
  }
  return dummy.next;
}
```

**Complexity:** Time O(N log k) where N is the total number of nodes, Space O(k) for the heap.
**Watch out:** The input can be `[]` or contain `null` lists. Push the node, not the value, so you can follow `next`.

## Common Mistakes
- **"I can change the head without a dummy."** Then every delete/insert at the front needs a special case, and you will miss one. Use a dummy whenever the head can change.
- **"`fast.next.next` is safe."** It crashes when `fast.next` is null. Check `fast && fast.next` first.
- **"Flip the pointer, then move on."** After `cur.next = prev` you lost the rest. Save `nx = cur.next` first.
- **"I split the list, so it is two lists."** Only if the first half's tail has `next = null`. Otherwise it still leads into the second half (or forms a cycle).
- **"Equal values mean the same node."** Cycle and intersection checks compare node references (`===`), not `val`.
- **"Recursion is fine for any list."** A list with 100,000 nodes can overflow the JS call stack. Prefer iteration when n can be large.
- **"Losing the old head is harmless."** After rewiring, always return the correct new head (`dummy.next`, or the last `prev`).

## Interview Tips
- Say the plan first: "I will use a dummy node, a fast and a slow pointer, then reverse the second half." Draw 4-5 nodes and trace.
- Ask: singly or doubly? Can there be a cycle? Is the list sorted? Can I modify the input? Is the node count bounded?
- Start with the hash-set / array version if you are stuck, then say "I can get O(1) space with pointer rewiring."
- Test aloud on these: empty list, one node, two nodes, odd and even length, all equal values, head removed, tail removed, cycle that includes the head.
- Typical follow-ups:
  - "Do it in one pass." (nth from end, middle)
  - "Do it in O(1) space." (palindrome, copy random, cycle start)
  - "Do it recursively, then iteratively." (reverse)
  - "Why does Floyd's work?" (say the `a + b = k*L` argument from problem 10)
  - "Why a doubly linked list for LRU?" (O(1) delete when you hold the node)
- Mention when you restore the input (palindrome) or when you mutate it (reorder, sort). Interviewers care about side effects.

## Practice List
**Easy**
- Remove Linked List Elements
- Convert Binary Number in a Linked List to Integer
- Merge Nodes in Between Zeros
- Delete the Middle Node of a Linked List
- Design Linked List

**Medium**
- Swap Nodes in Pairs
- Partition List
- Add Two Numbers II
- Find the Duplicate Number
- Swap Nodes in a Linked List (swap kth from start and end)
- Split Linked List in Parts
- Maximum Twin Sum of a Linked List
- LRU Cache
- Insert into a Sorted Circular Linked List
- Next Greater Node in Linked List

**Hard**
- Merge k Sorted Lists (try the divide and conquer version)
- Reverse Nodes in k-Group (try the recursive version)
- LFU Cache
- All O`one Data Structure
- Sort List with O(1) space (bottom-up merge sort)

## Key Takeaways
- A linked list gives **O(1) insert/delete at a known node** but **O(n) access by position**.
- Use a **dummy node** whenever the head might change.
- **Fast/slow pointers** find the middle, detect a cycle and (with a second phase from the head) find the cycle start.
- **Reverse by saving `next` first**, then flipping; three pointers `prev, cur, nx`.
- Many problems are combinations: **middle + reverse + merge** solves palindrome, reorder and sort.
- Cut lists explicitly with `next = null`, and compare **nodes, not values**.
- A **min-heap of heads** (or pairwise merging) merges k lists in O(N log k).

---
[← Previous: Binary Search](../08-binary-search/README.md) | [Next: Stack and Queue →](../10-stack-and-queue/README.md) | [Back to Core Linear Index](../README.md)
