# Chapter 6 — Recursion & Backtracking

> **Part I · Week 4** · Prerequisites: Ch 1–5
> **Problems:** 9 worked (1 ★) · 10 practice · 3 stretch · **Time:** about 9 hours

Recursion is the first idea in this book that people often "understand" by watching, and then can't produce on a blank page. The reason is almost always the same: they try to *trace* the recursion in their head, every call and every return, and drown. This chapter teaches the other way, the one that scales. You **state what a function promises**, check the smallest cases, and **trust the promise** for the smaller calls. That's induction from Ch 2, and once it clicks, recursion stops being mysterious.

Then we use recursion to **search**: try every possibility, but abandon a branch the moment it can't succeed. That's *backtracking*. It's how you solve "return all …" problems, puzzles like N-Queens and Sudoku, and every problem where n is small enough that trying everything is the intended answer.

Recursion is also the foundation for most of what follows: merge sort (Ch 9), linked lists (Ch 13), every tree chapter (Ch 19–20), depth-first search (Ch 21) and dynamic programming (Ch 27).

---

## 0. From zero: what happens when a function calls itself

### 0.1 Two parts, always

A recursive function solves a problem by solving **smaller versions of the same problem**. Every correct one has exactly two parts:

* **A base case:** an input small enough to answer directly, with no further calls.
* **A recursive case:** an answer built from the answers to one or more *strictly smaller* inputs.

```java
// The sum of a[i..n): everything from index i to the end.
static int sumFrom(int[] a, int i) {
    if (i == a.length) return 0;            // base case: an empty range sums to 0
    return a[i] + sumFrom(a, i + 1);        // recursive case: first element + sum of the rest
}
```

"Strictly smaller" is what guarantees the recursion **stops**. Here `i` grows by one each call and stops at `a.length`. A recursive case that doesn't shrink the input, or a missing base case, recurses until the stack overflows.

### 0.2 The call stack

Each call gets its own **frame**: its own copy of the parameters and local variables, plus a note of where to resume when it returns. Frames pile up on the **call stack** and come off in reverse order. For `sumFrom({4, 7, 1}, 0)`:

```text
call sumFrom(a, 0)                         stack: [i=0]
  call sumFrom(a, 1)                       stack: [i=0][i=1]
    call sumFrom(a, 2)                     stack: [i=0][i=1][i=2]
      call sumFrom(a, 3) → base case: 0    stack: [i=0][i=1][i=2][i=3]
    sumFrom(a, 2) returns 1 + 0 = 1        stack: [i=0][i=1][i=2]
  sumFrom(a, 1) returns 7 + 1 = 8          stack: [i=0][i=1]
sumFrom(a, 0) returns 4 + 8 = 12           stack: [i=0]
```

Two consequences:
* **Depth costs memory.** n nested calls use O(n) stack space, even if nothing else is allocated. Java's default stack holds roughly 10⁴–10⁵ frames (Ch 4 §16), so a recursion over 10⁵ elements can crash with `StackOverflowError`, while a loop wouldn't.
* **Each frame's variables are private.** That's why recursion is a natural fit for problems where every branch needs its own state.

### 0.3 The recursion tree, and how it can explode

When a call makes **more than one** recursive call, the calls form a tree (Ch 3 §4). The total work is the number of nodes in that tree times the work per node.

### Worked R0a · [Fibonacci Number](https://leetcode.com/problems/fibonacci-number/)

> `F(0) = 0`, `F(1) = 1`, `F(n) = F(n − 1) + F(n − 2)`. Return `F(n)` for `n ≤ 30`.

The definition translates directly:

```java
public int fib(int n) {
    if (n <= 1) return n;                 // base cases F(0) = 0, F(1) = 1
    return fib(n - 1) + fib(n - 2);
}
```

It's correct, but look at the tree for `fib(5)`:

```text
                     fib(5)
              /                \
          fib(4)              fib(3)
         /      \            /      \
     fib(3)    fib(2)    fib(2)    fib(1)
     /   \      /  \      /  \
 fib(2) fib(1) ...   ...  ...
```

`fib(3)` is computed twice and `fib(2)` three times. The number of calls grows like 1.6ⁿ: computing `fib(30)` makes 2 692 537 calls to find a number we could build in 30 steps. The waste is **the same subproblem solved again and again**, which is the "overlapping subproblems" bottleneck from Ch 1 §2. Two fixes:

* **Remember each answer the first time it's computed** (*memoisation*), so every `fib(k)` is computed once: O(n). That's the doorway to dynamic programming, Ch 27.
* **Build upwards with a loop**, keeping only the last two values:

```java
public int fib(int n) {
    if (n == 0) return 0;
    int prev = 0, cur = 1;                // F(i - 1) and F(i), starting at i = 1
    for (int i = 2; i <= n; i++) {
        int next = prev + cur;
        prev = cur;
        cur = next;
    }
    return cur;
}
```

**The lesson:** recursion is a way of *thinking*, and it doesn't have to be the way of *running*. When the tree repeats work, fix it.

### Worked R0b *(custom)* · Tower of Hanoi

> Three pegs. `n` disks of different sizes sit on peg A, largest at the bottom. Move them all to peg C. Rules: move one disk at a time, and never put a larger disk on a smaller one. Output the moves.

Trying to simulate this in your head fails quickly. Instead, think about **the largest disk**. It has to move from A to C at some point. At that moment, the other n − 1 disks can't be on A (they'd be on top of it) or on C (it's landing there), so they must all be on B. So the solution *must* look like this:

1. move the top n − 1 disks from A to B (using C as the spare);
2. move the largest disk from A to C;
3. move the n − 1 disks from B to C (using A as the spare).

Steps 1 and 3 are the **same problem with n − 1 disks**. We don't need to know how they work, only that a function exists that does them:

```java
// Moves n disks from peg `from` to peg `to`, using `via` as the spare, appending each move.
static void hanoi(int n, char from, char to, char via, List<String> moves) {
    if (n == 0) return;                      // nothing to move
    hanoi(n - 1, from, via, to, moves);      // clear the smaller disks onto the spare
    moves.add(from + "->" + to);             // move the largest disk
    hanoi(n - 1, via, to, from, moves);      // bring the smaller disks back on top of it
}
```

**How many moves?** With M(n) moves for n disks, M(0) = 0 and M(n) = 2·M(n − 1) + 1. By induction, M(n) = 2ⁿ − 1: true for n = 0, and if M(n − 1) = 2ⁿ⁻¹ − 1, then M(n) = 2(2ⁿ⁻¹ − 1) + 1 = 2ⁿ − 1. ✓ The argument above also shows no solution can do better, since the largest disk forces this structure.

Notice what you **didn't** do: trace the calls. You reasoned about one level, and trusted the function for the smaller problem. That's the whole skill.

---

## 1. Recursion as induction: the contract

To write a recursive function, write its **contract** first, in one sentence: *"`f(x)` returns ___ for input x."* Then:

1. **Base case.** Make `f` correct on the smallest inputs.
2. **Recursive case.** *Assume* `f` already honours its contract on every strictly smaller input. Using those answers, build the answer for `x`.
3. **Progress.** Check that every recursive call really is on a smaller input.

If all three hold, `f` is correct by induction (Ch 2 §4). You never trace it. If the code is wrong, the bug is in exactly one of these three places: the contract is vague, a base case is wrong, or the combine step is wrong.

### Worked B1 · [K-th Symbol in Grammar](https://leetcode.com/problems/k-th-symbol-in-grammar/)

> Row 1 is `0`. Each next row replaces every `0` with `01` and every `1` with `10`. Rows: `0`, `01`, `0110`, `01101001`, …. Return the `k`-th symbol (1-indexed) of row `n`, for `n ≤ 30`.

Row 30 has 2²⁹ ≈ 5·10⁸ symbols, so building it is out. Write the contract instead: **`kth(n, k)` returns the k-th symbol of row n.**

* **Base:** `kth(1, 1) = 0`.
* **Recursive case:** where does symbol k of row n come from? Each symbol of row n − 1 *produces two* symbols of row n. Symbol k comes from **parent position `(k + 1) / 2`** of row n − 1. If k is odd, it's the first symbol of the pair, which equals the parent (`0 → 01`, `1 → 10`). If k is even, it's the second, which is the parent flipped.
* **Progress:** n decreases every call.

```java
public int kthGrammar(int n, int k) {
    if (n == 1) return 0;
    int parent = kthGrammar(n - 1, (k + 1) / 2);
    return (k % 2 == 1) ? parent : 1 - parent;
}
```

That's O(n) time with O(n) stack depth, which is 30. Trusting the contract turned a problem about a 500-million-symbol string into five lines.

**How to choose "the smaller problem".** These are the usual ways to shrink an input:

| Shrink by | Example |
| :--- | :--- |
| One element (first or last) | `sumFrom(a, i + 1)`, process `n − 1` items |
| Half | Binary search (Ch 10), merge sort (Ch 9) |
| One level of structure | A tree's left and right subtrees (Ch 19) |
| One decision | "Use item i or not", "which column for this row's queen" (§2 onwards) |

---

## 2. Backtracking: searching a tree of decisions

Many problems ask you to **find or list every way** to make a sequence of decisions: which elements to include, what order to put them in, where to place each queen. The decisions form a **tree**. Each level is one decision, each branch is one choice, and each leaf is a complete candidate. Backtracking walks that tree depth-first using recursion:

```java
void backtrack(State state) {
    if (state is complete) {
        record a COPY of the state;   // the state keeps changing after this
        return;
    }
    for (Choice choice : choices available now) {
        if (choice breaks a rule) continue;   // prune: don't walk into a dead branch
        apply(choice);                        // choose
        backtrack(state);                     // explore
        undo(choice);                         // un-choose: put the state back exactly as it was
    }
}
```

**Why "undo" instead of passing a fresh copy down each branch?** A copy per call costs O(n) each time (Ch 3 §6). Instead, one shared state is changed on the way down and restored on the way up, so every branch sees exactly the state it expects. The rule that makes this work:

> **Invariant:** when `backtrack` returns, the state is exactly what it was when it was called.

Break that, by forgetting an undo or undoing the wrong thing, and branches start seeing each other's choices.

### Worked B2 · [Subsets](https://leetcode.com/problems/subsets/) ★

> **Attempt first (20 minutes).** Given an array of distinct integers, return all possible subsets (the power set). `[1, 2, 3]` → `[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]` in any order.

**The decision tree:** for each element in turn, decide *in* or *out*. There are n levels and 2 branches each, so 2ⁿ leaves, each one a subset. For `[1, 2, 3]`, with `chosen` shown at every node:

```text
level 0: decide 1                      []
                              out /         \ in
level 1: decide 2           []                 [1]
                       out /   \ in        out /   \ in
level 2: decide 3     []        [2]       [1]       [1,2]
                     /  \      /   \     /   \      /    \
leaves:            []   [3]  [2] [2,3] [1] [1,3] [1,2] [1,2,3]
```

The depth-first walk visits the leaves left to right. Going **down** an "in" edge is `chosen.add`, and coming back **up** it is the undo, `chosen.remove`. So at any moment, `chosen` holds exactly the choices on the path from the root to the current node.

```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    decide(nums, 0, new ArrayList<>(), result);
    return result;
}

// Decide in/out for nums[i..]; `chosen` holds the elements chosen from nums[0..i).
private void decide(int[] nums, int i, List<Integer> chosen, List<List<Integer>> result) {
    if (i == nums.length) {
        result.add(new ArrayList<>(chosen));          // a leaf: record a copy
        return;
    }
    decide(nums, i + 1, chosen, result);              // nums[i] out
    chosen.add(nums[i]);                              // nums[i] in
    decide(nums, i + 1, chosen, result);
    chosen.remove(chosen.size() - 1);                 // undo
}
```

**A second shape of the same search.** Here every node is a subset, not only the leaves, and the loop picks *the next element to add*. It only looks forward from `start`, so each set is produced once, in increasing index order:

```java
private void extend(int[] nums, int start, List<Integer> chosen, List<List<Integer>> result) {
    result.add(new ArrayList<>(chosen));              // every node is a subset
    for (int i = start; i < nums.length; i++) {
        chosen.add(nums[i]);
        extend(nums, i + 1, chosen, result);
        chosen.remove(chosen.size() - 1);
    }
}
```

The second shape is the one to know. "Choose the next element from `start` onwards" is the skeleton of every combination problem.

**Cost.** There are 2ⁿ subsets and copying each costs up to n, so **O(n · 2ⁿ)**. The output is that big, so no algorithm can do better. At n = 20 that's about 2·10⁷: fine. At n = 30 it isn't.

### Worked B3 · [Permutations II](https://leetcode.com/problems/permutations-ii/)

> Given numbers that **may contain duplicates**, return all *distinct* permutations. `[1, 1, 2]` → `[1,1,2], [1,2,1], [2,1,1]`.

For distinct numbers: at each position, try every unused element. Track "unused" with `boolean[] used`. That gives n · (n − 1) · … = n! leaves.

With duplicates, `[1₁, 1₂, 2]` and `[1₂, 1₁, 2]` are the same permutation, so the plain search outputs it twice. **The fix: give equal values a fixed order.** Sort the array so equal values sit next to each other, and **only allow a copy to be used if the copy before it is already in use**. That forces equal values to be placed left to right, so each distinct arrangement is built exactly one way.

```java
public List<List<Integer>> permuteUnique(int[] nums) {
    int[] sorted = nums.clone();
    Arrays.sort(sorted);                                       // equal values become adjacent
    List<List<Integer>> result = new ArrayList<>();
    permute(sorted, new boolean[sorted.length], new ArrayList<>(), result);
    return result;
}

private void permute(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> result) {
    if (path.size() == nums.length) {
        result.add(new ArrayList<>(path));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        // Equal values must be placed left to right: skip nums[i] while the equal value
        // just before it is still unused.
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;
        used[i] = true;
        path.add(nums[i]);
        permute(nums, used, path, result);
        path.remove(path.size() - 1);
        used[i] = false;
    }
}
```

**Why it's exactly right.** Take any distinct arrangement. Label the copies of each value 1, 2, 3, … in the order they appear *in that arrangement*. The rule allows exactly that labelling (first copy first, second copy second) and forbids every other labelling of the same arrangement. So each distinct permutation is produced once. ✓

### Worked B4 · [Combination Sum II](https://leetcode.com/problems/combination-sum-ii/)

> Given `candidates` (positive, **possibly with duplicates**) and a `target`, return all distinct combinations that sum to `target`, using each candidate at most once.

Use the "choose the next element from `start`" shape from Subsets, plus two rules:

1. **Skip a value already tried at this level.** If `c[i] == c[i − 1]` and `i > start`, choosing `c[i]` as the *next* element gives exactly the same combinations as choosing `c[i − 1]` did a moment ago, since everything after them is identical. So skip it. (`i > start` matters: using equal values at *different* levels is allowed, e.g. `[1, 1, 6]` for target 8.)
2. **Prune.** The array is sorted, so once `c[i] > remaining`, every later value is too big as well: `break`, not `continue`.

```java
public List<List<Integer>> combinationSum2(int[] candidates, int target) {
    int[] c = candidates.clone();
    Arrays.sort(c);
    List<List<Integer>> result = new ArrayList<>();
    combine(c, 0, target, new ArrayList<>(), result);
    return result;
}

// Choose the next element from c[start..]; `remaining` is how much is still needed.
private void combine(int[] c, int start, int remaining, List<Integer> path, List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(path));
        return;
    }
    for (int i = start; i < c.length; i++) {
        if (i > start && c[i] == c[i - 1]) continue;   // same value as the previous choice at this level
        if (c[i] > remaining) break;                    // sorted, so nothing later fits either
        path.add(c[i]);
        combine(c, i + 1, remaining - c[i], path, result);   // i + 1: each element used at most once
        path.remove(path.size() - 1);
    }
}
```

Change `i + 1` to `i` in the recursive call and an element can be reused, which is Combination Sum in the practice set. That one character is the whole difference.

---

## 3. Constraint search and pruning

When the rules of the problem rule out most branches early, backtracking explores only a small part of the full tree. **Pruning** means checking the rules *as soon as* a choice is made, not when the candidate is complete.

### Worked B5 · [N-Queens](https://leetcode.com/problems/n-queens/)

> Place n queens on an n × n board so that no two attack each other (same row, column or diagonal). Return every arrangement, as boards of `'Q'` and `'.'`.

**Decisions:** one queen per row is forced, so the decision at level `r` is *"which column for row r's queen?"*.

**Fast rule checks:** keep three boolean tables. A queen at `(r, c)` occupies its column `c`, its "down-right" diagonal (all cells there share `r − c`), and its "down-left" anti-diagonal (all share `r + c`). `r − c` ranges over `−(n−1) … n−1`, so add `n − 1` to make it a valid index.

```java
public List<List<String>> solveNQueens(int n) {
    List<List<String>> result = new ArrayList<>();
    place(0, n, new int[n], new boolean[n], new boolean[2 * n - 1], new boolean[2 * n - 1], result);
    return result;
}

// queenCol[r] = column of the queen already placed in row r, for rows 0..row-1.
private void place(int row, int n, int[] queenCol, boolean[] colUsed, boolean[] diagUsed,
                   boolean[] antiUsed, List<List<String>> result) {
    if (row == n) {
        result.add(render(queenCol, n));
        return;
    }
    for (int c = 0; c < n; c++) {
        int d = row - c + n - 1, a = row + c;
        if (colUsed[c] || diagUsed[d] || antiUsed[a]) continue;    // prune immediately
        queenCol[row] = c;
        colUsed[c] = diagUsed[d] = antiUsed[a] = true;
        place(row + 1, n, queenCol, colUsed, diagUsed, antiUsed, result);
        colUsed[c] = diagUsed[d] = antiUsed[a] = false;             // undo
    }
}

private List<String> render(int[] queenCol, int n) {
    List<String> board = new ArrayList<>();
    for (int r = 0; r < n; r++) {
        char[] row = new char[n];
        Arrays.fill(row, '.');
        row[queenCol[r]] = 'Q';
        board.add(new String(row));
    }
    return board;
}
```

**How much does pruning save?** Without any checks, there are nⁿ ways to put one queen in each row: 16 777 216 for n = 8. Using each column once cuts that to 8! = 40 320. With diagonal pruning, the search for n = 8 visits about 2 000 nodes and finds 92 solutions.

### Worked B6 · [Word Search](https://leetcode.com/problems/word-search/)

> Given a grid of letters and a word, return whether the word can be traced through horizontally or vertically adjacent cells, using each cell at most once.

**Contract:** `matchFrom(r, c, i)` returns whether `word[i..]` can be traced starting at cell `(r, c)`, without reusing cells on the current path. Mark a cell as used by **temporarily overwriting it** with a character that can't appear in the word, then restore it.

```java
public boolean exist(char[][] board, String word) {
    for (int r = 0; r < board.length; r++)
        for (int c = 0; c < board[0].length; c++)
            if (matchFrom(board, r, c, word, 0)) return true;
    return false;
}

private boolean matchFrom(char[][] board, int r, int c, String word, int i) {
    if (i == word.length()) return true;                               // everything matched
    if (r < 0 || r >= board.length || c < 0 || c >= board[0].length) return false;
    if (board[r][c] != word.charAt(i)) return false;                    // also rejects '#'
    char saved = board[r][c];
    board[r][c] = '#';                                                  // in use on this path
    boolean found = matchFrom(board, r + 1, c, word, i + 1) || matchFrom(board, r - 1, c, word, i + 1)
                 || matchFrom(board, r, c + 1, word, i + 1) || matchFrom(board, r, c - 1, word, i + 1);
    board[r][c] = saved;                                                // undo
    return found;
}
```

The `||` stops at the first success. The restore line still runs, because it comes after the whole expression has been evaluated, so the board is always left unchanged.

**Cost.** For each of the R · C start cells, the path branches at most 3 ways at each step (it can't go back), so the worst case is O(R · C · 3^L) for a word of length L. Two cheap prunings matter in practice: give up at once if the board doesn't contain enough of some letter, and search from whichever end of the word has the rarer letter.

### Worked B7 · [Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)

> Return every well-formed string of n pairs of parentheses. n = 3 → `((())), (()()), (())(), ()(()), ()()()`.

The brute force generates all 2²ⁿ strings of `(` and `)` and checks each one. Instead, **only generate valid prefixes**. A prefix can still be completed into a valid string exactly when it has **no more `)` than `(` at every point, and at most n `(` in total**. So:
* add `(` if fewer than n have been used;
* add `)` only if it would still close something (`close < open`).

```java
public List<String> generateParenthesis(int n) {
    List<String> result = new ArrayList<>();
    build(n, 0, 0, new StringBuilder(), result);
    return result;
}

// sb holds a valid prefix containing `open` '(' and `close` ')'.
private void build(int n, int open, int close, StringBuilder sb, List<String> result) {
    if (sb.length() == 2 * n) {
        result.add(sb.toString());
        return;
    }
    if (open < n) {
        sb.append('(');
        build(n, open + 1, close, sb, result);
        sb.deleteCharAt(sb.length() - 1);
    }
    if (close < open) {
        sb.append(')');
        build(n, open, close + 1, sb, result);
        sb.deleteCharAt(sb.length() - 1);
    }
}
```

Every branch leads to at least one valid answer, so there are **no dead ends**: the work is proportional to the output. The number of answers is the n-th *Catalan number* (Ch 18), about 4ⁿ / n^1.5: 16 796 for n = 10.

---

## 4. Estimating the size of a search

Before writing a backtracking solution, **estimate the tree** and check it against the budget (Ch 1 §2):

| Search | Leaves (worst case) | Safe up to about |
| :--- | :--- | :--- |
| Subsets (in/out per element) | 2ⁿ | n ≈ 20–25 |
| Permutations | n! | n ≈ 10–11 |
| k-element combinations | C(n, k) | depends on k: C(30, 5) ≈ 1.4·10⁵ is fine, C(30, 15) ≈ 1.6·10⁸ is not |
| Grid path of length L | R · C · 3^(L−1) | L ≈ 10–15 |
| Placing items into k groups | kⁿ | small n and k |

Then multiply by the work done at each leaf (often copying an answer: O(n)).

**Pruning changes the average, not the worst case** unless you can prove it does. So the safe reasoning is: *"the full tree fits the budget, and pruning makes it faster still."* When the full tree doesn't fit and you can't prove the pruning is enough, you need a different idea. That's the next section.

---

## 5. Where backtracking breaks

| Symptom | Example | What replaces it |
| :--- | :--- | :--- |
| The same sub-search repeats | Fibonacci; "can the rest of the string be split into dictionary words?" asked again and again from the same position | **Memoisation / DP** (Ch 27): remember each sub-answer |
| You need a *count* or an *optimum*, not a list, and n is large | Number of ways to climb 1000 stairs | **DP** (Ch 27–28): counting doesn't require enumerating |
| n is around 40 for subsets | Closest subset sum with 40 numbers: 2⁴⁰ ≈ 10¹² | **Meet in the middle** (Ch 37): 2 × 2²⁰ |
| The recursion is too deep | DFS on a path-shaped input of 10⁵ nodes | An **explicit stack** (Ch 19, Ch 21), or a bigger thread stack in an OA (Ch 4 §16) |

The first row is the important one. **If the same arguments reach your recursive function many times, it's a DP problem in disguise.** You'll see that turned into a method in Ch 27.

---

## 6. Variations *(previews; each is solved in the chapter shown)*

| Change | Adaptation |
| :--- | :--- |
| Count the answers instead of listing them | Return counts from the recursion instead of recording paths. If the count is huge or sub-searches repeat, use DP (Ch 27) |
| Only ask whether an answer exists | Return `boolean` and stop at the first success, as Word Search does |
| Subsets of exactly size k | Stop when `chosen.size() == k`; prune when there aren't enough elements left |
| Elements may be reused | Recurse with `i` instead of `i + 1` (Combination Sum) |
| Many words to search in one grid | A trie shares common prefixes between words (Ch 20, Word Search II) |
| The best arrangement, with n ≤ 20 | Often bitmask DP over subsets (Ch 32) |

---

## 7. Recognition

**Signals that backtracking is intended:**
* The problem asks for **all** solutions ("return all possible…"), so the output itself is exponential and any algorithm must enumerate.
* The **constraints are tiny**: n ≤ 10 for orderings, n ≤ 20 for subsets, a 9 × 9 board, a word of length ≤ 15.
* The answer is a **sequence of decisions under rules** that you can check partway through (queens, Sudoku, words in a grid).

**False friends:**
* *"Count the number of ways"* with large n looks like enumeration, but it's DP (Ch 27–28).
* *"Minimum / maximum over subsets"* with n ≤ 20 can be backtracking, but it's often bitmask DP (Ch 32), which is more reliable when sub-searches repeat.
* A recursion whose arguments repeat (§5) is DP waiting to happen.

---

## 8. Implementation traps

| Trap | Symptom | Fix |
| :--- | :--- | :--- |
| Recording the live path instead of a copy | Every answer in the result is the same (usually empty) list | `result.add(new ArrayList<>(path))` |
| A missing or misplaced undo | Branches see each other's choices; answers too long or wrong | Every `apply` has a matching `undo` right after the recursive call |
| `continue` instead of `break` when pruning on sorted input (or the reverse) | Too slow, or answers missing | `break` only when *every* later choice also fails |
| Duplicate skip with `i > 0` instead of `i > start` | Valid combinations like `[1, 1, 6]` disappear | Skip only repeats *at the same level* |
| Building strings with `+` | O(L) per step, plus garbage | `StringBuilder` with `append` and `deleteCharAt` / `setLength` |
| No base case, or one the input never reaches | `StackOverflowError` | Write the base case first; check every call shrinks the input |
| Modifying the input array without restoring it | Later code (or the caller) sees a damaged board | Restore every change (Word Search) |

---

## 9. Practice

For each problem, write the **contract** (what the recursive function returns or records) and the **decision at each level** before coding. Stress-test at least two against a slower enumeration, using the Ch 5 harness: compare the *sorted* lists of answers.

1. **[Find the Winner of the Circular Game](https://leetcode.com/problems/find-the-winner-of-the-circular-game/)** *(Medium; follow-up: a recurrence instead of simulation)*
2. **[Subsets II](https://leetcode.com/problems/subsets-ii/)** *(Medium)*
3. **[Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)** *(Medium)*
4. **[Combination Sum](https://leetcode.com/problems/combination-sum/)** *(Medium)*
5. **[Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)** *(Medium)*
6. **[Restore IP Addresses](https://leetcode.com/problems/restore-ip-addresses/)** *(Medium)*
7. **[Different Ways to Add Parentheses](https://leetcode.com/problems/different-ways-to-add-parentheses/)** *(Medium)*
8. **[Maximum Length of a Concatenated String with Unique Characters](https://leetcode.com/problems/maximum-length-of-a-concatenated-string-with-unique-characters/)** *(Medium)*
9. **[Matchsticks to Square](https://leetcode.com/problems/matchsticks-to-square/)** *(Medium)*
10. **[Sudoku Solver](https://leetcode.com/problems/sudoku-solver/)** *(Hard)*

**Stretch:** **[Expression Add Operators](https://leetcode.com/problems/expression-add-operators/)** *(Hard)* · **[Partition to K Equal Sum Subsets](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/)** *(Medium, but pruning-heavy)* · **[Remove Invalid Parentheses](https://leetcode.com/problems/remove-invalid-parentheses/)** *(Hard)*

<details><summary>Nudges</summary>

1. Circular Game: number the friends from 0. If you know the winner's position among n − 1 people, where is that position in the original circle of n? (Contract: `winner(n, k)` returns the 0-indexed position of the winner.)
2. Subsets II: the "choose the next element from `start`" shape, plus the same-level skip rule from Combination Sum II.
3. Phone: one level per digit; the choices are that digit's letters.
4. Combination Sum: Combination Sum II with reuse allowed, and no duplicates in the input.
5. Palindrome Partitioning: the decision at each level is *where the next piece ends*. Only recurse when that piece is a palindrome.
6. IP: exactly 4 levels. Each piece is 1–3 digits, ≤ 255, and has no leading zero unless it is exactly "0". Prune when the remaining length can't fit the remaining pieces.
7. Add Parentheses: contract `ways(expression)` = the list of all values. Split at every operator; combine every left value with every right value.
8. Concatenated: decide in/out for each string; skip any string that repeats its own letters, and any that clashes with the letters used so far (a `boolean[26]`).
9. Matchsticks: four sides, each needing total/4. Sort descending, try each stick on each side, and skip a side whose current length equals an earlier side's (it would give the same search).
10. Sudoku: fill empty cells one by one; keep `boolean` tables for the digits used in each row, column and 3×3 box (Ch 5 §8 gives the box index).
11. Stretch, Add Operators: carry the current value *and* the last term added, because `*` has to undo that term: `value − last + last × next`.
12. Stretch, K Equal Subsets: like Matchsticks, with k buckets. Sorting descending and skipping equal-sum buckets are what make it pass.
13. Stretch, Invalid Parentheses: first count how many `(` and `)` must be removed, then backtrack removing exactly that many, skipping consecutive equal characters.
</details>

<details><summary>Key</summary>

Circular Game → B1 (Josephus recurrence `(winner(n − 1, k) + k) % n`) · Subsets II → B2 + skip rule · Phone → B2 product of choices · Combination Sum → B4 with reuse · Palindrome Partitioning → B4 over cut positions · IP → B5 with length pruning · Add Parentheses → B1 divide and conquer · Concatenated Unique → B2 with a 26-letter `boolean[]` · Matchsticks → B5 (sort descending, skip equal sides) · Sudoku → B5 with `boolean` tables · Add Operators → B7 carrying the last term · K Equal Subsets → B5 pruning (bitmask DP in Ch 32) · Invalid Parentheses → B7 with a minimum-removal bound
</details>

---

## 10. Self-test

<details><summary>1. What three things must you check for a recursive function to be correct?</summary>

That its base cases are right; that, assuming the recursive calls honour the contract, the combine step is right; and that every call is on a strictly smaller input, so it terminates.
</details>

<details><summary>2. Why does `result.add(path)` (without a copy) produce a list of identical, usually empty, lists?</summary>

Every entry is a reference to the *same* `path` object, which keeps changing and ends up back where it started (usually empty) after all the undos.
</details>

<details><summary>3. In Combination Sum II, why is the duplicate check `i > start` and not `i > 0`?</summary>

Equal values may be used at *different* levels (`[1, 1, 6]`). The repeat to avoid is choosing the same value twice *as the same next element*, which only happens at the same level, i.e. for `i > start`.
</details>

<details><summary>4. A backtracking search over subsets of n = 35 elements, with pruning, is your plan. What do you say?</summary>

2³⁵ ≈ 3.4·10¹⁰ in the worst case is far over budget, and pruning isn't guaranteed to help. Look for overlapping sub-searches (DP) or split the set in half (meet in the middle, Ch 37).
</details>

<details><summary>5. How many moves does Tower of Hanoi need for 20 disks, and why can't it be fewer?</summary>

2²⁰ − 1 = 1 048 575. The largest disk must move at some point, and at that moment all the others must be on the third peg. That forces the recursive structure, and with it M(n) = 2·M(n − 1) + 1.
</details>

---

## 11. Exit gate

Move on when you can:

* write the **contract** for any recursive function you write, before writing its body;
* write Subsets (both shapes), Permutations II and Combination Sum II from blank files in under 10 minutes each, and explain each duplicate-skip rule;
* write N-Queens with the three boolean tables from memory;
* estimate a search's size before coding it, and say which alternative to use when it doesn't fit (§5);
* solve 8 of the 10 practice problems, with at least two stress-tested.
