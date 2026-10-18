# Chapter 27 — DP I: From Brute Recursion to Tables

> **Part VI · Week 24** · Prerequisites: Ch 1 (Derivation C), Ch 2 (exchange/cut-and-paste), Ch 6 (recursion, memoisation), Ch 22 (DP in topological order)
> **Problems:** 6 worked (1 ★) · 8 practice · 3 stretch · **Time:** about 9 hours

Dynamic programming has a reputation for being mysterious, because it's often taught as a zoo of recurrences to memorise. It isn't a zoo. It's **one method**, and you've already used it three times without the name: Longest Arithmetic Subsequence in Ch 1, Fibonacci in Ch 6, and topological DP in Ch 22. This chapter makes the method explicit, in six steps you write down for every DP problem, and shows how to go from a slow, obviously correct recursion to a fast table, one mechanical step at a time.

Chapters 28–33 apply this method to every family of DP problems. **This is the chapter to master before them**, and the one to reread when a DP problem resists.

---

## 0. What dynamic programming is

A problem is ready for DP when two things are true:

1. **Optimal substructure.** An optimal solution to the problem is built from optimal solutions to **smaller instances of the same problem**. You prove this with a *cut-and-paste* argument (Ch 1 Derivation C): if part of an optimal solution weren't optimal for its own subproblem, you could cut it out and paste in a better one, which would make the whole thing better, a contradiction.
2. **Overlapping subproblems.** The brute-force recursion solves the **same subproblems over and over**. That's the Fibonacci tree from Ch 6 §0.3.

**DP = solve each distinct subproblem once, remember the answer, and combine.** The cost is

> **(number of distinct subproblems) × (work to combine each one)**,

the rule from Ch 3 §4.

---

## 1. The DP method: six steps, in writing

| Step | Write down | House Robber (below) |
| :-- | :--- | :--- |
| **1. State** | What one subproblem is, **in English**, precisely: its parameters and the value it stands for | `best[i]` = the most money obtainable from the **first i houses** |
| **2. Transition** | What is the **last decision** that produced the answer for this state? List every option; each one leads to smaller states | Either house i − 1 is robbed or it isn't: `best[i] = max(best[i−1], best[i−2] + nums[i−1])` |
| **3. Base cases** | The states small enough to answer directly | `best[0] = 0`, `best[1] = nums[0]` |
| **4. Order** | An evaluation order in which every state comes after the states it uses: a topological order (Ch 22) | increasing i |
| **5. Answer** | Which state (or combination of states) is the final answer | `best[n]` |
| **6. Cost** | (number of states) × (transition cost); then space, and whether it can be rolled down | n × O(1) time; O(n) space, rollable to O(1) |

**Two checks on step 2, every time:**
* **Optimisation (min/max):** the options must **cover every possibility**. Missing one gives a wrong optimum.
* **Counting:** the options must cover every possibility *and* **not overlap**. Overlapping options count the same object twice, the most common counting-DP bug.

**Step 1 is where most DP attempts fail.** If the state doesn't contain enough information to decide the future, the transition can't be written. The fix is always to **add what's missing to the state** (§4).

---

## 2. The bridge: from recursion to a table

You don't have to invent a table out of thin air. Start from the brute-force recursion, which is easy to write and obviously correct, and transform it:

1. **Brute-force recursion**, written straight from the transition.
2. **Spot the overlap:** draw the recursion tree and notice the repeated calls.
3. **Memoise** (top-down): store each result the first time it's computed.
4. **Tabulate** (bottom-up): fill an array in the step-4 order instead of recursing.
5. **Roll** the array: keep only the states that later states still need.
6. **Reconstruct** the actual choices, not just the optimal value, by walking back through the table.

### Worked DP1a · [Climbing Stairs (the bridge in miniature)](https://leetcode.com/problems/climbing-stairs-the-bridge-in-miniature/)

> You climb n stairs, 1 or 2 steps at a time. How many distinct ways are there?

**State:** `ways(i)` = the number of ways to climb exactly i stairs. **Last decision:** the final move was 1 step (from i − 1) or 2 steps (from i − 2). These options are disjoint, so they can be added: `ways(i) = ways(i − 1) + ways(i − 2)`, with `ways(0) = ways(1) = 1`.

```java
// Step 1: brute recursion. O(φⁿ): it's the Fibonacci tree from Ch 6.
int ways(int i) { return i <= 1 ? 1 : ways(i - 1) + ways(i - 2); }

// Step 3: memoised. Each i is computed once: O(n). (0 means "not computed": every real answer is ≥ 1.)
int ways(int i, int[] memo) {
    if (i <= 1) return 1;
    if (memo[i] != 0) return memo[i];
    return memo[i] = ways(i - 1, memo) + ways(i - 2, memo);
}

// Steps 4–5: tabulated in increasing i, then rolled down to the two states each step needs.
public int climbStairs(int n) {
    int twoBack = 1, oneBack = 1;                // ways(i - 2) and ways(i - 1), starting at i = 2
    for (int i = 2; i <= n; i++) {
        int current = oneBack + twoBack;
        twoBack = oneBack;
        oneBack = current;
    }
    return oneBack;
}
```

### Worked DP1b · [House Robber](https://leetcode.com/problems/house-robber/) ★

> **Attempt first (20 minutes).** Houses in a row hold `nums[i] ≥ 0` money. You can't rob two adjacent houses. Return the maximum amount you can rob.

**Steps 1–3**, from the table in §1. **Proof of the transition (cut-and-paste):** take an optimal plan for the first i houses. If it doesn't rob house i − 1, it's a plan for the first i − 1 houses, and it must be an optimal one (otherwise swapping in a better plan would improve it). If it does rob house i − 1, it can't rob house i − 2, so the rest is a plan for the first i − 2 houses, and again it must be optimal. One of the two options gives the true optimum, and the `max` takes it.

**Bridge step 1, the brute force**, which tries both choices for the last house. O(2ⁿ):

```java
int best(int[] nums, int i) {
    if (i <= 0) return 0;
    return Math.max(best(nums, i - 1), best(nums, i - 2) + nums[i - 1]);
}
```

**Step 2:** `best(i − 2)` is reached from `best(i)` directly *and* through `best(i − 1)`, the same overlap as Fibonacci. **Steps 3–4:** memoise it, or tabulate directly in increasing i:

```java
public int rob(int[] nums) {
    int n = nums.length;
    int[] best = new int[n + 1];                 // best[i] = most money from the first i houses
    best[0] = 0;
    best[1] = nums[0];
    for (int i = 2; i <= n; i++) best[i] = Math.max(best[i - 1], best[i - 2] + nums[i - 1]);
    return best[n];
}
```

**Step 5, rolling:** `best[i]` needs only `best[i − 1]` and `best[i − 2]`, so two variables replace the array, as in Climbing Stairs, for O(1) space.

**Step 6, reconstruction:** to report *which* houses were robbed, keep the full table and walk backwards, asking at each i **which option produced `best[i]`**:

```java
List<Integer> robbed = new ArrayList<>();
for (int i = n; i >= 1; ) {
    boolean robLast = (i == 1) || best[i] == best[i - 2] + nums[i - 1];   // did robbing house i-1 achieve best[i]?
    if (robLast) {
        robbed.add(i - 1);
        i -= 2;
    } else {
        i -= 1;
    }
}
```

Reconstruction is why you sometimes *don't* roll the array: you need the full table (or stored choices) to walk back through. When several options tie, any of them gives a valid optimal answer.

---

## 3. Segmentations: what is the last piece?

When a problem cuts a sequence into pieces, the **last decision is the last piece**. The state is "the best (or count) for the first i elements", and the transition tries every possible last piece.

### Worked DP2a · [Decode Ways](https://leetcode.com/problems/decode-ways/)

> Letters A–Z are encoded as `"1"`–`"26"`. Count the ways to decode a digit string. `"226"` → 3 (`2 2 6`, `22 6`, `2 26`).

**State:** `ways[i]` = the number of decodings of the **first i characters**. **Last piece:** either one digit (`s[i−1]`, valid if it's not `'0'`) or two digits (`s[i−2..i−1]`, valid if it's 10–26). A last piece of length 1 and one of length 2 are different decodings, so the options **don't overlap**, and adding them counts correctly.

```java
public int numDecodings(String s) {
    int n = s.length();
    int[] ways = new int[n + 1];
    ways[0] = 1;                                   // the empty prefix: exactly one way (decode nothing)
    for (int i = 1; i <= n; i++) {
        if (s.charAt(i - 1) != '0') ways[i] += ways[i - 1];                     // last piece: one digit 1–9
        if (i >= 2) {
            int two = (s.charAt(i - 2) - '0') * 10 + (s.charAt(i - 1) - '0');
            if (two >= 10 && two <= 26) ways[i] += ways[i - 2];                  // last piece: 10–26
        }
    }
    return ways[n];
}
```

**`ways[0] = 1`** is the base case that makes counting work: "there is one way to decode nothing", so a valid first piece contributes exactly one decoding. Getting the empty base case right is half of every counting DP.

### Worked DP2b · [Word Break](https://leetcode.com/problems/word-break/)

> Can a string be split into a sequence of dictionary words?

**State:** `can[i]` = the first i characters can be split. **Last piece:** some word `s[j..i)`, with `can[j]` true. Only words up to the longest dictionary word's length are worth trying.

```java
public boolean wordBreak(String s, List<String> wordDict) {
    Set<String> words = new HashSet<>(wordDict);
    int maxLen = 0;
    for (String w : wordDict) maxLen = Math.max(maxLen, w.length());
    int n = s.length();
    boolean[] can = new boolean[n + 1];
    can[0] = true;                                   // the empty prefix splits trivially
    for (int i = 1; i <= n; i++) {
        for (int j = Math.max(0, i - maxLen); j < i && !can[i]; j++) {   // the last word is s[j, i)
            can[i] = can[j] && words.contains(s.substring(j, i));
        }
    }
    return can[n];
}
```

That's O(n · L · L) for word length L, counting the substring and its hash. Notice that this is exactly the "**overlapping sub-searches**" warning from Ch 6 §5: backtracking over splits re-asks "can the rest be split?" from the same position many times. DP asks each position once.

---

## 4. When one value per index isn't enough

### Worked DP3 · [Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/)

> Return the largest product of a contiguous subarray (values may be negative or zero).

The first try mirrors Kadane (Ch 1 practice): `maxEnding[i] = max(nums[i], maxEnding[i−1] · nums[i])`. **It's wrong**, and seeing why teaches step 1. On `[−2, 3, −4]`, the best product ending at 3 is 3, and the formula then gives max(−4, 3 · −4) = −4 for the last position. But the true best is (−2 · 3) · (−4) = 24, which extends the **smallest** product ending at 3, namely −6. Multiplying by a negative number turns the smallest value into the largest.

**Fix the state, not the transition: keep both the largest and the smallest product ending at each index.**

```java
public int maxProduct(int[] nums) {
    int maxEnding = nums[0], minEnding = nums[0], best = nums[0];   // extremes of products ending at i
    for (int i = 1; i < nums.length; i++) {
        int x = nums[i];
        int a = maxEnding * x, b = minEnding * x;                     // a negative x swaps their roles
        maxEnding = Math.max(x, Math.max(a, b));                      // or start a new subarray at x
        minEnding = Math.min(x, Math.min(a, b));
        best = Math.max(best, maxEnding);
    }
    return best;
}
```

**The general lesson:** if the optimal choice at step i depends on something the state doesn't record, here the sign, **add it to the state**. You'll do this constantly: "holding a stock or not" (Ch 30), "the last colour used", "whether the first house was robbed" (House Robber II, practice).

---

## 5. Reducing to a DP you know

### Worked DP4 · [Delete and Earn](https://leetcode.com/problems/delete-and-earn/)

> Choose a value x from `nums`, earn x, and delete every copy of x − 1 and x + 1. Repeat. Return the maximum you can earn.

It doesn't look like House Robber, until you **change what you iterate over** (Ch 1, move 7). Iterate over **values** instead of positions. Taking value v means taking *every* copy of it (once you take one, nothing stops you taking the rest), which earns `v · count(v)`, and it forbids v − 1 and v + 1, its neighbours on the number line. That's House Robber on the array `points[v] = v · count(v)`:

```java
public int deleteAndEarn(int[] nums) {
    int maxValue = 0;
    for (int x : nums) maxValue = Math.max(maxValue, x);
    int[] points = new int[maxValue + 1];          // points[v] = what taking every copy of v earns
    for (int x : nums) points[x] += x;
    int twoBack = 0, oneBack = 0;                  // House Robber over v = 0, 1, …, maxValue
    for (int v = 0; v <= maxValue; v++) {
        int current = Math.max(oneBack, twoBack + points[v]);
        twoBack = oneBack;
        oneBack = current;
    }
    return oneBack;
}
```

O(n + maxValue). **Before inventing a new DP, ask whether the problem is a known one in disguise.** The reduction is often the whole insight.

---

## 6. Top-down or bottom-up?

| | Memoisation (top-down) | Tabulation (bottom-up) |
| :--- | :--- | :--- |
| Writing it | Easiest: it's the brute force plus a cache | You must choose the evaluation order |
| States visited | Only the reachable ones (good when most states are unused) | All of them |
| Recursion depth | Can overflow on 10⁴–10⁵ states (Ch 4 §16) | None |
| Space rolling | Not possible | Easy, once the dependencies are clear |
| Speed in Java | Slower (calls, and boxing if you use a `HashMap` cache) | Faster (array loops) |

**A practical path:** derive the recurrence, write it top-down to check it on the examples, then convert it to bottom-up for speed and space. Interviewers accept either. Say which one you're writing, and why.

---

## 7. Where DP breaks

| Situation | Why | Instead |
| :--- | :--- | :--- |
| The state is missing information the future depends on | The transition can't be written correctly (Max Product with only the max) | Add the missing information to the state |
| No optimal substructure | Optimal sub-answers don't combine (the longest *simple* path in a graph with cycles) | Different structure, or exponential search |
| Overlapping options in a counting DP | Objects counted twice | Make the "last decision" options disjoint |
| The state space is too large | 10⁵ × 10⁵ states | Find a smaller state (often by fixing what the answer depends on), or speed up the transitions (Ch 33) |
| A greedy argument exists | DP works but is slower | Prove the greedy (Ch 26) |

---

## 8. Recognition

**Signals:**
* "**Number of ways**", "**how many**" sequences / decodings / paths.
* "**Minimum / maximum** cost / profit / length" when choices **interact** (greedy fails; Ch 26 §6).
* "Can it be split / partitioned / reached?", where the brute force tries every split.
* A sequence of decisions where the future depends only on a **small summary of the past**: the last element, a count, a flag.
* Brute force is exponential, but the number of **distinct situations** is polynomial.

**False friends:** "the shortest path with unit costs" is BFS (Ch 21). "The maximum of a sliding window" is a deque (Ch 12). A provable greedy beats DP.

---

## 9. Implementation traps

| Trap | Fix |
| :--- | :--- |
| A vague state ("dp[i] = the answer up to i") | Say *exactly* which prefix and which value: "the first i houses", "ending at index i" |
| Mixing "the first i" with "ending at index i" | Pick one per problem and comment it |
| Off-by-one between dp indices and array indices | For prefix states use size n + 1, with `dp[i]` about `a[0..i−1]` |
| A wrong empty base case in counting | `ways[0] = 1` (one way to do nothing) unless the problem forbids empty |
| Memoising with 0 as "unset" when 0 is a real answer | Use −1, `Integer`/`Long` with `null`, or a separate `boolean[]` |
| Rolling the array, then trying to reconstruct | Keep the full table, or store the choices |
| Missing the modulus, or overflow, in counting | Reduce after every addition; use `long` |

---

## 10. Practice

For **every** problem, write the six steps of §1 before coding.

1. **[House Robber II](https://leetcode.com/problems/house-robber-ii/)** *(Medium)*
2. **[Count Ways To Build Good Strings](https://leetcode.com/problems/count-ways-to-build-good-strings/)** *(Medium)*
3. **[Solving Questions With Brainpower](https://leetcode.com/problems/solving-questions-with-brainpower/)** *(Medium)*
4. **[Minimum Cost For Tickets](https://leetcode.com/problems/minimum-cost-for-tickets/)** *(Medium)*
5. **[Perfect Squares](https://leetcode.com/problems/perfect-squares/)** *(Medium)*
6. **[Partition Array for Maximum Sum](https://leetcode.com/problems/partition-array-for-maximum-sum/)** *(Medium)*
7. **[Maximum Sum Circular Subarray](https://leetcode.com/problems/maximum-sum-circular-subarray/)** *(Medium)*
8. **[Extra Characters in a String](https://leetcode.com/problems/extra-characters-in-a-string/)** *(Medium)*

**Stretch:** **[Filling Bookcase Shelves](https://leetcode.com/problems/filling-bookcase-shelves/)** *(Medium)* · **[Word Break II](https://leetcode.com/problems/word-break-ii/)** *(Hard)* · **[Minimum Difficulty of a Job Schedule](https://leetcode.com/problems/minimum-difficulty-of-a-job-schedule/)** *(Hard)*

<details><summary>Nudges</summary>

1. Robber II: the houses form a circle, so the first and last can't both be robbed. Run House Robber twice: without the last house, and without the first.
2. Good Strings: `ways[len]` = strings of exactly that length; the last append was `zero` zeros or `one` ones. Sum over the allowed lengths, mod 10⁹ + 7.
3. Brainpower: go *backwards*. `best[i]` = max(skip question i: `best[i + 1]`; solve it: `points + best[i + brainpower + 1]`). Use `long`.
4. Tickets: `cost[d]` = the cheapest cover for days ≤ d; on a travel day, the last ticket bought was a 1-, 7- or 30-day pass.
5. Perfect Squares: `dp[n]` = the fewest squares summing to n; the last square used is some k² ≤ n.
6. Partition for Max Sum: the last segment ends at i and has length 1…k; its value is length × its maximum.
7. Circular: the answer is either the ordinary Kadane maximum, or total − the minimum subarray (the wrapping case). If every number is negative, return the Kadane maximum.
8. Extra Characters: `dp[i]` = the fewest extra characters in the first i; either character i − 1 is extra, or some dictionary word ends at i.
9. Stretch, Bookcase: the last shelf holds books j…i−1 (total width ≤ the shelf width); its height is their tallest book.
10. Stretch, Word Break II: Word Break's table, plus reconstruction of *all* answers (memoise the list of sentences per start index).
11. Stretch, Job Schedule: `dp[d][i]` = the best way to do the first i jobs in d days; the last day takes jobs j…i−1, costing their maximum.
</details>

<details><summary>Key</summary>

Robber II → two runs of DP1 · Good Strings → DP1 counting · Brainpower → DP1 run backwards · Tickets → DP1 over days with three options · Perfect Squares → DP1 with many choices · Partition for Max Sum → DP2 (last segment) · Circular → Kadane on max and min (DP3) · Extra Characters → DP2 with a set or trie · Bookcase → DP2 (last shelf) · Word Break II → DP2 + reconstruction · Job Schedule → a 2D last-segment DP
</details>

---

## 11. Self-test

<details><summary>1. What are the six steps of the DP method?</summary>

The state (in English), the transition from the last decision, base cases, evaluation order, the answer's location, and the cost (states × transition).
</details>

<details><summary>2. Why does counting DP need the "last decision" options to be disjoint?</summary>

Each object must be counted exactly once. If two options can produce the same object, it gets counted twice.
</details>

<details><summary>3. In Max Product Subarray, why does keeping only the maximum fail?</summary>

Multiplying by a negative number turns the smallest product into the largest. On `[−2, 3, −4]`, the optimum extends −6, the minimum ending at 3, not the maximum, 3.
</details>

<details><summary>4. Why might you not roll a DP array down to O(1) space?</summary>

To reconstruct the actual choices, you walk back through the table, so you need it whole (or a stored record of each choice).
</details>

<details><summary>5. What's the cost formula for a DP, and what is it for Word Break?</summary>

(Number of states) × (transition cost). Word Break: n states, each trying up to L last words of length up to L, so O(n · L²) including the substring hashing.
</details>

---

## 12. Exit gate

Move on when you can:

* write the six steps for a new DP problem *before* any code, including the cut-and-paste argument for the transition;
* take House Robber through all six bridge stages, including reconstruction, from a blank file;
* explain the empty base case in Decode Ways, and the two-value state in Max Product;
* spot a reduction to a known DP (as in Delete and Earn);
* solve 7 of the 8 practice problems, each with the six steps written first.
