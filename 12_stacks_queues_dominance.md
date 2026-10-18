# Chapter 12 — Stacks, Queues & Dominance

> **Part II · Week 10** · Prerequisites: Ch 1 (Derivation B), Ch 2, Ch 3 §3, Ch 5, Ch 11
> **Problems:** 9 worked (1 ★) · 10 practice · 3 stretch · **Time:** about 9 hours

A stack is the simplest structure in this book, and one of the most powerful. Its power comes from a single idea you first met in Ch 1 Derivation B: **dominance**. When a new element arrives, some older elements become useless for every future question, so you throw them away. If they're always thrown away from the same end, the survivors form a stack (or a deque), kept in sorted order *for free*. This chapter turns that idea into a family of O(n) algorithms for "nearest greater/smaller", areas, contributions and window maxima. Interviewers love these, because the brute force is obvious and the O(n) solution needs a real insight.

---

## 0. From zero: stacks, queues and deques

| Structure | Rule | Operations (all O(1)) | Natural use |
| :--- | :--- | :--- | :--- |
| **Stack** | Last in, first out (LIFO) | `push`, `pop`, `peek` | The most recent unfinished thing: nesting, undo, recursion |
| **Queue** | First in, first out (FIFO) | `offer` (at the back), `poll` (from the front), `peek` | Processing in arrival order: BFS (Ch 21), scheduling |
| **Deque** | Both ends | Add/remove/peek at front and back | Windows (§5), 0-1 BFS (Ch 23) |

These are **interfaces**: they say which element comes out next, not how it's stored. The pictures, after the operations shown:

```text
STACK   push 1, push 2, push 3                 QUEUE   offer 1, offer 2, offer 3
          ┌───┐  ← top: pop() returns 3                front                  back
          │ 3 │                                  poll() ← [ 1 | 2 | 3 ] ← offer(4)
          │ 2 │                                  returns 1               joins here
          │ 1 │
          └───┘                                DEQUE   both ends work
                                               pollFirst ← [ 1 | 2 | 3 ] → pollLast
                                               offerFirst →             ← offerLast
```

A stack reverses the order things arrived in; a queue preserves it.

### 0.1 A stack from an array

A stack needs only an array and a count. The top is at `size − 1`.

```java
final class IntStack {
    private int[] data = new int[8];
    private int size = 0;

    void push(int x) {
        if (size == data.length) data = java.util.Arrays.copyOf(data, size * 2);   // grow like IntList (Ch 5)
        data[size++] = x;
    }
    int pop()  { if (size == 0) throw new IllegalStateException("empty"); return data[--size]; }
    int peek() { if (size == 0) throw new IllegalStateException("empty"); return data[size - 1]; }
    boolean isEmpty() { return size == 0; }
}
```

The array and `size` after each operation (`_` = a slot not in use; its old contents don't matter):

```text
push 4   [4 _ _ _ _ _ _ _]  size 1
push 9   [4 9 _ _ _ _ _ _]  size 2
push 2   [4 9 2 _ _ _ _ _]  size 3      peek() = data[2] = 2
pop      [4 9 _ _ _ _ _ _]  size 2      returns 2; nothing is erased, size just moves down
push 7   [4 9 7 _ _ _ _ _]  size 3      overwrites the old slot
```

When you know the maximum size in advance (at most n indices, as in most problems here), skip the class and use a raw `int[] stack` plus `int size`, as Ch 1 did. It's the fastest stack in Java. Otherwise use `ArrayDeque` (Ch 4 §19); never the legacy `Stack` class.

### Worked P0 · [Design Circular Queue](https://leetcode.com/problems/design-circular-queue/)

> Implement a queue with fixed capacity k: `enQueue`, `deQueue`, `Front`, `Rear`, `isEmpty`, `isFull`.

A queue on a plain array has a problem. Removing from the front would shift everything (O(n)), and if you don't shift, the used region drifts right and runs off the end. The fix is to **let the queue wrap around**. Keep the index of the front element (`head`) and the number of elements (`size`). Element `j` of the queue lives at `(head + j) % capacity`. With capacity 4:

```text
offer A, B, C         [A B C _]   head 0, size 3        queue: A B C
poll, poll            [_ _ C _]   head 2, size 1        queue: C
offer D               [_ _ C D]   head 2, size 2        D goes to (2 + 1) % 4 = 3
offer E               [E _ C D]   head 2, size 3        E goes to (2 + 2) % 4 = 0: wrapped round
                                                         queue, front to back: C D E
```

Think of the array as a ring: the slot after index 3 is index 0.

```java
class MyCircularQueue {
    private final int[] data;
    private int head = 0, size = 0;     // the queue is data[head], data[head + 1], ... wrapping at the end

    public MyCircularQueue(int k) { data = new int[k]; }

    public boolean enQueue(int value) {
        if (size == data.length) return false;
        data[(head + size) % data.length] = value;     // the slot just past the back
        size++;
        return true;
    }

    public boolean deQueue() {
        if (size == 0) return false;
        head = (head + 1) % data.length;
        size--;
        return true;
    }

    public int Front() { return size == 0 ? -1 : data[head]; }
    public int Rear()  { return size == 0 ? -1 : data[(head + size - 1) % data.length]; }
    public boolean isEmpty() { return size == 0; }
    public boolean isFull()  { return size == data.length; }
}
```

Storing `size` rather than a separate tail index removes the classic ambiguity, where `head == tail` could mean either empty or full. `ArrayDeque` is exactly this circular buffer, plus growth. Implement Queue using Stacks (practice) shows another way to build FIFO from LIFO, with amortised O(1) operations.

**Common misconceptions:**

| Belief | Reality |
| :--- | :--- |
| "Java's `Stack` class is the stack to use" | It's a legacy synchronised class. Use `ArrayDeque` (`push`/`pop`/`peek`) or a raw array |
| "`ArrayDeque` iterates bottom to top when used as a stack" | `push` adds at the **front**, so iteration goes from the top down |
| "`queue.remove(x)` is cheap" | Removing from the middle of a queue or deque is O(n); only the ends are O(1) |
| "A queue needs a linked list" | A circular array (above) is faster and simpler; `ArrayDeque` is one |
| "`ArrayDeque` can hold `null`" | It throws on `null`; `poll` returning `null` means "empty" |

---

## 1. Matching and nesting: the most recent unfinished thing

A stack answers "**what is the most recent thing that's still unfinished?**" That's the shape of every nesting problem: brackets, tags, function calls, expressions.

### Worked P1a · [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)

> Is a string of `()[]{}` balanced and properly nested?

Each closing bracket must match the **most recent unmatched** opening one, which is exactly the top of a stack.

```java
public boolean isValid(String s) {
    Deque<Character> open = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') open.push(c);
        else {
            if (open.isEmpty()) return false;              // a closer with nothing to close
            char top = open.pop();
            if ((c == ')' && top != '(') || (c == ']' && top != '[') || (c == '}' && top != '{')) return false;
        }
    }
    return open.isEmpty();                                 // nothing left unclosed
}
```

### Worked P1b · [Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/)

> Evaluate a string like `"3+2*2"` or `" 3/2 "` containing non-negative integers, `+ − * /` and spaces (no parentheses). Division truncates toward zero.

**The structure:** the expression is a **sum of terms**, where each term is a product or quotient chain (`3 + 2*2` = `3 + (2·2)`). Multiplication and division act on the **most recent term**, and addition and subtraction start a new one. So keep a stack of terms. On `+` or `−`, push `±num`. On `*` or `/`, replace the top term. At the end, add the stack up.

```java
public int calculate(String s) {
    Deque<Integer> terms = new ArrayDeque<>();     // the expression's value = the sum of these terms
    int num = 0;
    char op = '+';                                 // the operator that came BEFORE the number being read
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (Character.isDigit(c)) num = num * 10 + (c - '0');
        boolean numberEnds = (!Character.isDigit(c) && c != ' ') || i == s.length() - 1;
        if (numberEnds) {
            switch (op) {
                case '+': terms.push(num); break;
                case '-': terms.push(-num); break;
                case '*': terms.push(terms.pop() * num); break;
                default:  terms.push(terms.pop() / num); break;   // Java truncates toward zero, as required
            }
            op = c;
            num = 0;
        }
    }
    int result = 0;
    for (int t : terms) result += t;
    return result;
}
```

Only the top of the stack is ever modified, so you could keep just the running total plus the last term instead, for O(1) space. With **parentheses** (Basic Calculator, stretch), each `(` pushes the current state and each `)` pops it back, which is the same "most recent unfinished thing" idea applied to nesting levels.

---

## 2. Nearest greater / smaller: the monotonic stack

Ch 1 Derivation B (Daily Temperatures) found, for every day, the *next* warmer day. The argument generalises to four questions that come up constantly:

> For each element, find the **nearest** element to its **left/right** that is **greater/smaller**.

**The dominance argument, once more.** Scan left to right and keep the elements still *waiting* for their answer (here, the next greater one). When `x` arrives, it **is** the answer for every waiting element smaller than `x`. Waiting elements always have non-increasing values: a smaller one sitting after a larger one would already have been resolved. So the ones `x` resolves are a run at the **top** of the stack. Pop them, then push `x`. Each index is pushed once and popped at most once, so the whole scan is **O(n)**.

**A trace** of "next greater to the right" on `a = [2, 1, 5, 3, 4]`. The stack holds indices; their values are shown in brackets:

```text
i=0  a=2   stack empty: push 0                        stack: 0[2]
i=1  a=1   top 2 > 1: nothing resolves; push 1        stack: 0[2] 1[1]
i=2  a=5   pop 1 (1 < 5): answer[1] = 5
           pop 0 (2 < 5): answer[0] = 5; push 2       stack: 2[5]
i=3  a=3   5 > 3: push 3                              stack: 2[5] 3[3]
i=4  a=4   pop 3 (3 < 4): answer[3] = 4; 5 > 4: push  stack: 2[5] 4[4]
end        indices 2 and 4 never resolve: answer -1

answer = [5, 5, -1, 4, -1]
```

Notice that the values on the stack always read non-increasing from bottom to top, which is why the resolved elements are always a run at the top.

| Question | Scan | Pop while the top is … than the new element | After popping, the top is … |
| :--- | :--- | :--- | :--- |
| Next greater (to the right) | left → right | smaller | — (popped elements get the answer `i`) |
| Next smaller | left → right | greater | — |
| Previous greater (to the left) | left → right | smaller **or equal** | the previous greater, if any |
| Previous smaller | left → right | greater or equal | the previous smaller, if any |

**Strict or non-strict** ("greater" versus "greater or equal") changes the answer only when values repeat. Decide it explicitly every time; §4 shows a problem where getting it wrong double-counts.

### Worked P2 · [Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/)

> In a **circular** array, return for each element the next greater element (wrapping around), or −1.

The only change from Daily Temperatures: elements near the end must be able to "see" the beginning. **Scan twice**, using `i % n`, and push each index only during the first lap.

```java
public int[] nextGreaterElements(int[] nums) {
    int n = nums.length;
    int[] answer = new int[n];
    Arrays.fill(answer, -1);
    int[] stack = new int[n];          // indices still waiting; their values are non-increasing
    int size = 0;
    for (int i = 0; i < 2 * n; i++) {
        int x = nums[i % n];
        while (size > 0 && nums[stack[size - 1]] < x) answer[stack[--size]] = x;
        if (i < n) stack[size++] = i;   // the second lap only resolves; it doesn't add waiters
    }
    return answer;
}
```

Always **store indices** on the stack, not values. The index gives you the value *and* the position, which you need for distances, widths and contributions.

---

## 3. Areas: each bar's extent is set by its nearest smaller neighbours

### Worked P3 · [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) ★

> **Attempt first (30 minutes).** `heights[i]` is the height of a bar of width 1. Return the area of the largest rectangle that fits under the histogram. `[2, 1, 5, 6, 2, 3]` → 10 (the bars 5 and 6, height 5, width 2).

**Brute force:** for every pair `(l, r)`, the area is `min(heights[l..r]) × (r − l + 1)`: O(n²) with a running minimum. **Change what you iterate over** (Ch 1 move 7): instead of iterating over ranges, iterate over **which bar is the shortest in the rectangle**. If bar `i` is the shortest, the rectangle has height `heights[i]` and stretches as far as it can in both directions while the bars are at least as tall. That's up to the **previous smaller** bar on the left and the **next smaller** bar on the right:

> area(i) = `heights[i] × (nextSmaller(i) − previousSmaller(i) − 1)`. The best rectangle is the maximum over all `i`.

Both boundaries come from §2's table, and a single stack pass gives both at once. When bar `i` pops bar `j` (because `heights[i] < heights[j]`), `i` is `j`'s next smaller bar. After the pop, the new top is `j`'s previous smaller bar.

```java
public int largestRectangleArea(int[] heights) {
    int n = heights.length, best = 0;
    int[] stack = new int[n + 1];      // indices whose heights increase from bottom to top
    int size = 0;
    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i];              // a virtual height-0 bar at the end flushes the stack
        while (size > 0 && heights[stack[size - 1]] >= h) {
            int height = heights[stack[--size]];        // this bar is the shortest in its rectangle
            int left = (size == 0) ? -1 : stack[size - 1];   // nearest shorter bar on the left (or -1)
            best = Math.max(best, height * (i - left - 1)); // i is the first bar on the right that isn't taller
        }
        stack[size++] = i;
    }
    return best;                                        // ≤ 10^4 · 10^5: fits in int
}
```

**The trace** on `[2, 1, 5, 6, 2, 3]` (the stack holds indices; a pop computes the rectangle for the popped bar):

```text
i  h   pops                                                         stack after   best
0  2   —                                                            [0]           0
1  1   bar 0 (h 2): left = −1, width = 1 − (−1) − 1 = 1, area 2      [1]           2
2  5   —                                                            [1, 2]        2
3  6   —                                                            [1, 2, 3]     2
4  2   bar 3 (h 6): left = 2, width = 1, area 6                      [1, 4]        6
       bar 2 (h 5): left = 1, width = 4 − 1 − 1 = 2, area 10                       10
5  3   —                                                            [1, 4, 5]     10
6  0   bar 5 (h 3): left = 4, width 1, area 3   (virtual bar flushes)
       bar 4 (h 2): left = 1, width 4, area 8
       bar 1 (h 1): left = −1, width 6, area 6                       [6]           10
```

Each pop happens at the first bar to the right that is *not taller* (i), and the new stack top is the nearest *shorter* bar to the left. So the popped bar's rectangle is exactly `(left, i)`, both ends exclusive.

**Equal heights.** The loop pops on `>=`, so a bar is popped by an *equal* bar to its right, and its computed width stops short. That's harmless: the **last** bar in a run of equal heights is popped by a genuinely shorter bar (or the virtual 0), and it computes the full width. The maximum is still found. The general lesson: **with duplicates, make sure that at least one representative computes the true answer and that no one computes something too large.**

Maximal Rectangle (practice) applies this row by row to a binary matrix: each row turns into a histogram of how many consecutive 1s stand above each cell.

---

## 4. Contribution: summing over every subarray without listing them

### Worked P4 · [Sum of Subarray Minimums](https://leetcode.com/problems/sum-of-subarray-minimums/)

> Return the sum of `min(subarray)` over **all** contiguous subarrays, modulo 10⁹ + 7. (`n ≤ 3·10⁴`.)

There are about n²/2 subarrays. Even with a running minimum that's O(n²), roughly 4.5·10⁸. **Turn the sum around** (Ch 1 move 7, and the per-element counting of Ch 3 §5): instead of "for each subarray, its minimum", ask **"for each element, in how many subarrays is it the minimum?"**. Then

> answer = Σ over i of `arr[i] × (number of subarrays whose minimum is arr[i])`.

Element `i` is the minimum of exactly those subarrays that contain `i` and stay strictly between its nearest smaller neighbours: `left[i] < l ≤ i ≤ r < right[i]`. That's `(i − left[i]) × (right[i] − i)` choices of `(l, r)`.

**The duplicate trap.** With `[2, 2]`, the subarray `[2, 2]` has two minimum positions. If both boundaries are "strictly smaller", *both* elements claim it and it's counted twice. **Break ties asymmetrically:** `left` = previous element **< arr[i]**, `right` = next element **≤ arr[i]**. Then element `i` claims exactly the subarrays in which it is the **rightmost** minimum. Every subarray has exactly one rightmost minimum, so every subarray is counted exactly once. ✓

```java
public int sumSubarrayMins(int[] arr) {
    final long MOD = 1_000_000_007L;
    int n = arr.length;
    int[] left = new int[n], right = new int[n], stack = new int[n];
    int size = 0;
    for (int i = 0; i < n; i++) {                   // left[i]: previous index with value < arr[i], else -1
        while (size > 0 && arr[stack[size - 1]] >= arr[i]) size--;
        left[i] = (size == 0) ? -1 : stack[size - 1];
        stack[size++] = i;
    }
    size = 0;
    for (int i = n - 1; i >= 0; i--) {              // right[i]: next index with value <= arr[i], else n
        while (size > 0 && arr[stack[size - 1]] > arr[i]) size--;
        right[i] = (size == 0) ? n : stack[size - 1];
        stack[size++] = i;
    }
    long total = 0;
    for (int i = 0; i < n; i++) {
        long subarrays = (long) (i - left[i]) * (right[i] - i);    // those where arr[i] is the rightmost minimum
        total = (total + subarrays % MOD * arr[i]) % MOD;
    }
    return (int) total;
}
```

**This technique is called contribution counting**, and it's one of the highest-value ideas for hard problems. *Sum of something over all subarrays* (or all pairs, or all subsets) becomes *the sum over elements of (value × how many structures it's responsible for)*. Sum of Subarray Ranges (practice) is this for max − min. Ch 37 returns to it with strings.

---

## 5. Sliding window maximum: the monotonic deque

### Worked P5 · [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)

> Return the maximum of every window of length k. (`n ≤ 10⁵`.)

Rescanning each window is O(nk). The window's maximum isn't O(1) to maintain with a sum-style update (Ch 11 §8), because removing the maximum means you have to *find* the next one. **Dominance:** when a new element `x` enters, every older element that is `≤ x` can **never** be a window maximum again. `x` is at least as large, *and* it stays in the window longer. So keep only the undominated elements, in index order. Their values then strictly decrease, so the front is the maximum.

The difference from §2: elements also leave from the **front** when they fall out of the window. Removal at both ends needs a **deque**.

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    int[] result = new int[n - k + 1];
    int[] deque = new int[n];          // indices, front to back, with strictly decreasing values
    int head = 0, tail = 0;            // the deque is deque[head, tail); each index enters once, so no wrapping
    for (int i = 0; i < n; i++) {
        while (tail > head && nums[deque[tail - 1]] <= nums[i]) tail--;   // dominated by nums[i]: discard
        deque[tail++] = i;
        if (deque[head] <= i - k) head++;                                  // the front has left the window
        if (i >= k - 1) result[i - k + 1] = nums[deque[head]];            // the front is the maximum
    }
    return result;
}
```

**Invariant:** the deque holds, in index order, exactly the elements of the current window that have no larger-or-equal element after them in the window. The largest of these is the first. Each index enters and leaves at most once: **O(n)**.

**Where the deque idea goes next:** with prefix sums, it solves *Shortest Subarray with Sum at Least K* even with negative numbers (stretch; the case Ch 11's window couldn't handle). In DP, it speeds up transitions of the form "best of the last k states" (Ch 33).

---

## 6. Greedy with a stack: the smallest result in lexicographic order

### Worked P6 · [Remove K Digits](https://leetcode.com/problems/remove-k-digits/)

> Remove exactly `k` digits from the number string `num` so that the result is as small as possible. Return it without leading zeros (`"0"` if empty). `num = "1432219", k = 3` → `"1219"`.

**Removing one digit.** The leftmost digits matter most. Scan from the left: as long as the digits are non-decreasing, removing any of them would pull a larger-or-equal digit forward, which doesn't help. At the **first place where a digit is greater than the one after it** (a "peak"), removing that digit makes the number smaller at the earliest possible position. *Exchange argument:* any other single removal leaves that larger digit in place, at or before a position where this removal has a smaller digit. If there's no peak (the digits are non-decreasing), remove the last digit.

**Removing k digits** means applying that rule k times (the greedy stays optimal at each step), and a stack does all k applications in one left-to-right pass. When a smaller digit arrives, pop the larger digits before it while removals remain.

```java
public String removeKdigits(String num, int k) {
    StringBuilder kept = new StringBuilder();        // used as a stack of kept digits
    for (int i = 0; i < num.length(); i++) {
        char d = num.charAt(i);
        while (k > 0 && kept.length() > 0 && kept.charAt(kept.length() - 1) > d) {
            kept.deleteCharAt(kept.length() - 1);    // a larger digit just before a smaller one: remove it
            k--;
        }
        kept.append(d);
    }
    kept.setLength(kept.length() - k);               // leftover removals: kept is non-decreasing, so cut the tail
    int start = 0;
    while (start < kept.length() - 1 && kept.charAt(start) == '0') start++;   // strip leading zeros, keep one digit
    return kept.length() == 0 ? "0" : kept.substring(start);
}
```

The same stack-greedy shape solves Remove Duplicate Letters (practice). The extra rule there: only pop a letter if it **appears again later**, otherwise you'd lose it entirely.

---

## 7. Where stack techniques break

| Situation | Why | Instead |
| :--- | :--- | :--- |
| Queries for an **arbitrary** range's min/max (not nearest, not a sliding window) | The stack only answers "nearest" questions in scan order | Sparse table, segment tree (Ch 34) |
| A window whose left end can move **backwards**, or jump around | The deque relies on both ends moving forward | Ordered multiset (Ch 15), segment tree (Ch 34) |
| The **k-th** nearest greater, or a count of the greater elements to the right | Popped elements are gone forever | Ordered structures / a Fenwick tree (Ch 34) |
| The window **median** | Not decided by dominance | Two heaps with lazy deletion (Ch 15) |
| Contribution counting with **ties** | Both tied elements claim the same subarray | Asymmetric strictness (§4) |

---

## 8. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| Next greater in another array (Next Greater Element I) | Run §2 on the second array; map value → answer |
| Number of people each person can see (Number of Visible People in a Queue) | A decreasing stack, scanning from the right (Mixed Set II) |
| Furthest `j > i` with `a[j] ≥ a[i]` (Maximum Width Ramp) | A decreasing stack of candidates, then a scan from the right (Mixed Set II) |
| Window minimum | The mirror deque (pop while `≥`) |
| Sum of subarray *maximums* | Mirror the comparisons in §4 |

---

## 9. Recognition

**Signals:**
* Matching, nesting, undo, "the most recent unfinished …" → a **stack**.
* "For each element, the **nearest / next / previous greater or smaller**", "how many days until …", "span", "visible", "the first element that blocks …" → a **monotonic stack**.
* An area or width **limited by the minimum** of a range (histograms, rectangles) → nearest-smaller boundaries.
* "**Sum over all subarrays** of min / max / range" → **contribution**, with boundaries from a monotonic stack.
* The maximum or minimum of **every window** of fixed size → a **monotonic deque**.
* "The **smallest number / lexicographically smallest string** after removing …" → a **greedy stack**.

**False friends:**
* "Maximum of an arbitrary range, many queries": that needs a range structure (Ch 34).
* "k-th largest in a window": heaps or ordered sets (Ch 15).

---

## 10. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Pushing values instead of indices | Push indices; read values through them |
| Inconsistent strictness with duplicates | Choose `<` versus `≤` deliberately; use asymmetric strictness for contributions |
| Forgetting the elements left on the stack at the end | A sentinel (the height-0 bar), or a final flush loop |
| The deque front isn't expired | `if (deque[head] <= i − k) head++` before reading the front |
| Popping an empty stack | Check `size > 0` (or `!isEmpty()`) inside every `while` |
| Area or contribution overflow | `long` products; reduce modulo where required |
| `Stack` / `LinkedList` | `int[]` + size, or `ArrayDeque` |

---

## 11. Practice

1. **[Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/)** *(Easy; amortised O(1))*
2. **[Min Stack](https://leetcode.com/problems/min-stack/)** *(Medium)*
3. **[Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/)** *(Medium)*
4. **[Asteroid Collision](https://leetcode.com/problems/asteroid-collision/)** *(Medium)*
5. **[Online Stock Span](https://leetcode.com/problems/online-stock-span/)** *(Medium)*
6. **[Decode String](https://leetcode.com/problems/decode-string/)** *(Medium)*
7. **[Sum of Subarray Ranges](https://leetcode.com/problems/sum-of-subarray-ranges/)** *(Medium)*
8. **[132 Pattern](https://leetcode.com/problems/132-pattern/)** *(Medium)*
9. **[Remove Duplicate Letters](https://leetcode.com/problems/remove-duplicate-letters/)** *(Medium)*
10. **[Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/)** *(Hard)*

**Stretch:** **[Shortest Subarray with Sum at Least K](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/)** *(Hard)* · **[Basic Calculator](https://leetcode.com/problems/basic-calculator/)** *(Hard)* · **[Longest Valid Parentheses](https://leetcode.com/problems/longest-valid-parentheses/)** *(Hard)*

<details><summary>Nudges</summary>

1. Queue via Stacks: an "in" stack and an "out" stack. Move everything from in to out only when out is empty. Why is each element moved at most once?
2. Min Stack: keep a second stack holding the minimum *at each depth*.
3. RPN: numbers push; an operator pops two (careful with the order for `−` and `/`) and pushes the result.
4. Asteroids: a stack of survivors. A left-moving asteroid fights the right-moving tops until one side is destroyed.
5. Stock Span: the previous *greater* price (§2), with the stack kept across calls. Store (price, span) pairs.
6. Decode String: on `[`, push the current string and the repeat count; on `]`, pop and repeat.
7. Subarray Ranges: Σ max − Σ min, each by contribution (§4). Use `long`.
8. 132: scan from the right, keeping a decreasing stack. Values popped become candidates for the "2"; you need some earlier element smaller than the best such candidate.
9. Duplicate Letters: like Remove K Digits, but pop a letter only if it appears later, and skip letters that are already in the stack.
10. Maximal Rectangle: build a histogram for each row (the run of 1s above each cell) and run P3 on it.
11. Stretch, Shortest Subarray ≥ K: over prefix sums `P`, keep a deque of increasing `P` values. Pop from the front while `P[i] − P[front] ≥ k` (recording an answer); pop from the back while `P[back] ≥ P[i]` (dominated).
12. Stretch, Basic Calculator: a stack of the signs in effect at each nesting level.
13. Stretch, Longest Valid: keep a stack of indices, with the index of the last unmatched `)` at the bottom as the base.
</details>

<details><summary>Key</summary>

Queue via Stacks → P0 (amortised transfer) · Min Stack → auxiliary stack of minimums · RPN → P1 · Asteroids → stack simulation · Stock Span → P2 (previous greater) · Decode String → P1 nested · Subarray Ranges → P4 for both max and min · 132 → P2 scanning from the right · Remove Duplicate Letters → P6 + last-occurrence guard · Maximal Rectangle → P3 per row · Shortest Subarray ≥ K → P5 over prefix sums · Basic Calculator → P1 with a sign stack · Longest Valid → P1 with an index stack
</details>

---

## 12. Self-test

<details><summary>1. Why is a monotonic-stack scan O(n) even though one step can pop many elements?</summary>

Each index is pushed once and popped at most once, so the total number of pops over the whole scan is at most n (Ch 3 §3).
</details>

<details><summary>2. In the histogram, why is the rectangle for bar i bounded by its previous and next smaller bars?</summary>

If bar i is the shortest bar in the rectangle, the rectangle can extend over every bar at least as tall as it, and it must stop at the first strictly shorter bar on each side.
</details>

<details><summary>3. Why does Sum of Subarray Minimums use "< on the left, ≤ on the right"?</summary>

So that each subarray is credited to exactly one element: its rightmost minimum. With strict comparisons on both sides, tied minimums would each claim the subarray.
</details>

<details><summary>4. In the sliding window maximum, why can an element be discarded as soon as a larger-or-equal element arrives?</summary>

The newcomer is at least as large and leaves the window later, so in every future window containing the old element, the newcomer is there too and is at least as big.
</details>

<details><summary>5. Why must leftover removals in Remove K Digits come from the end?</summary>

After the pass, the kept digits are non-decreasing, so there's no peak left, and by the one-removal rule the best single removal is the last digit. Repeating that k times removes the tail.
</details>

---

## 13. Exit gate

Move on when you can:

* build an array stack and a circular queue from scratch, and say why `size` beats a separate tail index;
* fill in §2's four-row table (scan direction, pop condition, strictness) without notes;
* write Largest Rectangle in Histogram, Sum of Subarray Minimums and Sliding Window Maximum from blank files, each in under 12 minutes, and explain how each handles duplicates;
* explain the dominance argument behind each monotonic structure in one sentence;
* solve 8 of the 10 practice problems.
