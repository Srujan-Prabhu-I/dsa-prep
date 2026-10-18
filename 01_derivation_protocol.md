# Chapter 1 — The Derivation Protocol

> **Part 0 · Week 1** · Prerequisites: basic Java · Used by: every chapter
> **Problems:** 4 worked (1 ★) · 6 practice · 2 stretch · **Time:** about 6 hours

Most people who "know DSA" and still fail interviews fail in the same place. They read the problem, wait for a pattern to light up, and when nothing lights up, they freeze. This chapter replaces waiting with a **procedure**. It is the same procedure strong problem-solvers run, mostly without noticing, written out so you can run it on purpose.

You'll use it on every problem in this book. The four worked derivations each show one **move**. Later chapters turn those moves into named techniques (prefix-state hashing, monotonic stacks, dynamic programming, binary search on the answer), but you'll have *derived* each of them before you learn its name. That's the point: a technique you derived once, you can derive again when you've forgotten its name.

> **What you need for this chapter: arrays and loops. Nothing else.**
>
> The derivations use three simple tools. Each is explained in plain words where it first appears, and built from scratch in a later chapter:
>
> * **A hash map** is a lookup table. You store a value under a key and get it back by key, in roughly constant time however many keys are stored. Java calls it `HashMap`. Derivation A explains the two calls it uses; you'll build a hash map yourself in Ch 7.
> * **A stack** is a list where you only ever add or remove at the end. In Derivation B it's literally an `int[]` plus a `size` counter.
> * **Halving a range** means checking the middle of a range and keeping the half where the answer must be. Derivation D explains it from zero.
>
> Some tables in this chapter mention technique names you haven't met yet (DP, heap, segment tree, …). They're **signposts** showing where each idea leads, not prerequisites: read each one as "Ch N teaches this". Reread this chapter after Part II (Week 14). It will read very differently.

### Big-O in one minute (Ch 3 makes it precise)

This chapter measures solutions by **how the number of basic steps grows with the input size n**, written in *Big-O* notation:

* **Count the steps, then keep only the fastest-growing term and drop constant factors.** A loop over n elements that does 3 things each time takes about 3n steps, written **O(n)**. Two nested loops over n elements take about n² steps: **O(n²)**. One step that doesn't depend on n is **O(1)**.
* **log n** (always log base 2 in this book) is **the number of times you can halve n before reaching 1**: log 8 = 3, log 10⁶ ≈ 20, log 10⁹ ≈ 30. An algorithm that halves its range each step takes O(log n) steps. **O(n log n)** is n steps that each cost log n, about 2·10⁷ for n = 10⁶.
* **Why it matters:** about 10⁸ simple steps take a second in Java (Step 2 below). At n = 10⁵, O(n²) is 10¹⁰ steps and times out, while O(n log n) is about 1.7·10⁶ and is instant.
* **Amortised O(1)** means "a single step can occasionally be expensive, but n steps cost O(n) in total, so O(1) on average". You'll meet it in Derivation B.
* **Space** is measured the same way: an extra array of n elements is O(n) space.

That's all the notation this chapter uses. Ch 3 (Week 3) adds the precise definitions, the counting rules for tricky loops and recursion, and how to measure growth for real.

---

## 1. Attempt first ★

Before reading anything else, try this problem for **25 minutes**. Write down whatever you get, even if it's slow.

> **Subarray Sum Equals K.** Given an integer array `nums` and an integer `k`, return the number of contiguous, non-empty subarrays whose sum equals `k`.
>
> Constraints: `1 ≤ n ≤ 2·10⁴`, `−1000 ≤ nums[i] ≤ 1000`, `−10⁷ ≤ k ≤ 10⁷`.
>
> Example: `nums = [1, 2, 3], k = 3` → `2` (the subarrays `[1, 2]` and `[3]`).

We'll derive the solution in §3, using the protocol.

---

## 2. The protocol

Eleven steps in four phases. Each step is a question you answer **in writing**. A text file or paper is fine; your head is not.

| Phase | Step | The question you answer |
| :--- | :--- | :--- |
| **Understand** | 1. Restate | What exactly is being chosen, and what is being asked about it? |
| | 2. Budget | Given the constraints, what is the slowest complexity that still passes? |
| | 3. Examples | Do I get the stated answer by hand? What tiny and adversarial cases exist? |
| **Search** | 4. Brute force | What is the search space, and what does it cost to try everything? |
| | 5. Bottleneck | Which part of the brute force is too slow, and what precisely does it waste? |
| | 6. Structure | Which structural question removes that waste? |
| **Justify** | 7. Prove or break | Why is the new idea correct, and can I break it? |
| | 8. Re-check budget | With real numbers, does it fit? |
| **Execute** | 9. Implement | Invariants as comments first; types chosen deliberately. |
| | 10. Test | Hand cases, edge cases, then the stress harness. |
| | 11. Reflect | What was the key insight, and what signal should have led me to it? |

### Step 1 · Restate

Rewrite the problem in your own words as a precise object. Pin down three things.

**(a) What is being chosen.** This one fact tells you the size of the brute force:

| You choose… | Number of candidates | Example |
| :--- | :--- | :--- |
| one element | n | find the peak |
| a pair `i < j` | n(n−1)/2 ≈ n²/2 | Two Sum |
| a subarray / substring (contiguous) | n(n+1)/2 ≈ n²/2 | maximum subarray |
| a subsequence / subset | 2ⁿ | longest increasing subsequence |
| an ordering / permutation | n! | task scheduling |
| a split into contiguous groups | 2ⁿ⁻¹ | split array into k parts |
| a monotone grid path | C(r + c, r) | unique paths |
| a value of the answer in `[lo, hi]` | hi − lo + 1 | minimum speed, minimum capacity |

**(b) What is being asked.** Is it a yes/no question, a count, a minimum or maximum, a construction ("return one such…"), or the k-th of something? Counting and optimising lead to very different solutions, even on the same objects.

**(c) The fine print.** Can the answer be empty? Are there duplicates? Negatives or zeros? Is the input sorted? May I modify it? "Return any" or "return the lexicographically smallest"? 0-indexed or 1-indexed? Most wrong answers in OAs come from fine print, not from the algorithm.

### Step 2 · Budget

A rough rule for Java: **about 10⁸ simple operations on primitive arrays take about one second**. Operations on `HashMap`, boxed `Integer`s or objects are 10–50× slower each. So 10⁷ map operations is fine and 10⁸ is not.

The right-hand column names techniques you'll learn later. For now, the first two columns are what matter.

| Largest n | Slowest complexity that's safe | What usually fits *(signposts)* |
| :--- | :--- | :--- |
| ≤ 10–12 | O(n! · n) | all permutations |
| ≤ 20–25 | O(2ⁿ · n) | all subsets, bitmask DP (n ≤ 20) |
| ≤ 40 | O(2ⁿᐟ² · n) | meet in the middle |
| ≤ 500 | O(n³) | interval DP, Floyd–Warshall |
| ≤ 5 000 | O(n²) | 2D DP, all pairs |
| ≤ 2·10⁵ – 10⁶ | O(n log n) or O(n) | sorting, binary search, heaps, hashing, two pointers |
| ≤ 10⁷ – 10⁸ | O(n), small constant | one pass over a primitive array, sieve |
| ≥ 10⁹ *(as a value)* | O(log n), O(√n), O(1) | binary search over values, math |

Constraints hold more information than just n:

* **Several variables.** For n elements and q queries, the cost is preprocessing + q × (cost per query). For a grid, the size is R × C cells. For a graph, the cost involves both V and E.
* **"The sum of n over all test cases ≤ 2·10⁵."** The budget applies to the *sum*, so an O(n) solution per test is fine even with 10⁴ tests.
* **The range of values.** Values ≤ 10⁶ mean you can index an array by value. Values ≤ 10⁹ mean you can't. An answer in `[1, 10⁹]` takes about 30 binary-search steps.
* **Tiny parameters.** k ≤ 20, an alphabet of 26, at most 10 digits: each of these can become a loop dimension or a bitmask.
* **"Return the answer modulo 10⁹ + 7."** The answer is astronomically large, so you're *counting*, and you can't enumerate the things you count. Think DP or combinatorics.
* **A suspiciously small n is a hint.** n ≤ 2000 where you'd expect 10⁵ means O(n²) is the intended solution. Don't over-optimise.

### Step 3 · Examples

Work through three examples before you code:

1. **The given example.** Check that you get the stated answer *by hand*. If you can't, you've misunderstood the problem.
2. **A tiny one** (n = 1 to 3) that you solve completely.
3. **An adversarial one** aimed at the fine print: all equal, all negative, strictly decreasing, containing zeros, or one where the answer is "none".

These become your test cases in Step 10.

### Step 4 · Brute force

State it in this form: *the answer is the best / count / existence over the search space S, and evaluating one candidate costs c.* The total cost is |S| · c.

Don't skip this step, for two reasons:

* **The bottleneck you'll attack in Step 5 lives inside the brute force.** Without it, you're guessing at techniques.
* **It's your test oracle.** Ch 5 shows how to compare a slow-but-obviously-correct brute force against your fast solution on thousands of random inputs. That catches bugs you would never find by reading the code.

Often the first improvement is free. Evaluate candidates **incrementally**: the sum of `a[i..j]` is the sum of `a[i..j−1]` plus `a[j]`. That turns c from O(n) into O(1). Call it level-0 optimisation.

In an interview, state the brute force and its complexity in under a minute. It shows you understood the problem, and it gives you a fallback.

### Step 5 · Find the bottleneck

Point at the loop that's too expensive and say *precisely* what it wastes. Nearly every bottleneck is one of five kinds, and each kind has a family of fixes. You don't need to know the fixes yet. What matters now is learning to **name the kind of waste**; the last two columns tell you which chapter teaches each fix.

| Kind of waste | What it looks like | What removes it | Where |
| :--- | :--- | :--- | :--- |
| **Recomputation** | The inner loop recomputes something derivable from the previous iteration or a table | Incremental updates, prefix sums | Ch 8 |
| **Re-scanning** | For each item, scan a collection looking for a match, a max, or a count | A structure that answers that exact query fast: hash map, sorted order + binary search, heap, ordered set, segment tree | Ch 7, 9, 10, 15, 34 |
| **Hopeless candidates** | Evaluating candidates that provably can't be the answer | Dominance (stacks/deques), monotone pointers, binary search, greedy | Ch 10–12, 26 |
| **Overlapping subproblems** | The recursion solves the same subproblem many times | Memoisation, DP | Ch 27–33 |
| **Independent exponential choices** | Choices interact only through a small summary | DP whose state is that summary | Ch 27–33 |

### Step 6 · Ask the structural questions

This is the heart of the protocol. Ask these in order; most problems give way by question 4. A few questions mention tools from later chapters. Right now, the *question* matters more than the tool: answering it in plain words is enough to make progress.

1. **Does the order of the input matter?** If not, sort it, or reduce it to counts.
2. **If I fix one thing, does the rest become easy?** Fix the right endpoint, the last element, the maximum, the middle element, or the value of the answer.
3. **Scanning left to right, what's the *least* I must remember about the past to handle the current element?** That memory is your *state*. If it's a lookup by value, use a hash map. If it's an aggregate, use a prefix. If it's the best of a changing set, use a heap, an ordered set or a monotone structure. If it's a small summary of past choices, use DP.
4. **Is something monotone?** If x works, does x + 1? If a window is valid, are its sub-windows valid? Do the pointers only ever move one way?
5. **Can some candidates be discarded forever?** That happens when another candidate is at least as good for every future question.
6. **Can I assume the optimal solution has a particular shape?** For example, "there is an optimal answer where the chosen jobs are sorted by deadline."
7. **Can I count what each element contributes**, instead of enumerating the structures it belongs to?
8. **Is it a graph in disguise?** States become nodes, moves become edges.
9. **Is the reversed or complementary problem easier?** Go backwards in time, start from the target, or count the bad cases.
10. **Is some parameter tiny?** n ≤ 20, k ≤ 10, or 26 letters can be a loop dimension or a bitmask.

### Step 7 · Prove it or break it

Before you code, write **one sentence** saying why the idea is correct. Then spend two minutes trying to break it with the adversarial examples from Step 3.

If you can't write the sentence, you don't have an algorithm yet, only a hunch. The usual shapes of that sentence are:

* **invariant:** "after processing i elements, X holds"
* **exchange:** "swapping any solution toward my choice never makes it worse"
* **monotonicity:** "if g works, every smaller g works"

Ch 2 teaches each of these properly.

### Step 8 · Re-check the budget with numbers

Plug in the real maxima:

* n log n at n = 2·10⁵ is about 3.6·10⁶: trivially fine.
* n√n at 10⁵ is about 3·10⁷: fine.
* n² at 10⁵ is 10¹⁰: no.

Check memory too. An `int[10⁷]` is 40 MB; a `HashMap` with 10⁶ entries is roughly 50–100 MB.

### Step 9 · Implement

Write the invariants as comments *first*, then the code that maintains them. Choose types deliberately: can any sum or product exceed 2.1·10⁹? Then use `long`. Prefer half-open ranges `[l, r)`. Name variables for what they mean (`countOfPrefix`, not `mp`). Ch 4 and Ch 5 cover the Java side.

### Step 10 · Test

First run the examples from Step 3. Then check:

* n = 1
* maximum n (does it finish in time?)
* values large enough to overflow
* the "no answer" case

When something fails, don't change code at random. Find the first moment where the program's state differs from your hand trace. For anything non-trivial, use the stress harness from Ch 5.

### Step 11 · Reflect (2 minutes)

Write two lines in your log:

* **Insight:** what property made it solvable, in one sentence.
* **Signal:** what in the statement or constraints *should* have pointed you there.

This step is how the book trains you to pick techniques for unlabeled problems. After a hundred problems, your signal lines *are* your recognition map.

> **In an interview,** Steps 1–8 happen out loud in about 10 minutes. Ch 40 turns them into natural dialogue.

---

## 3. Derivation A: [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

*Move: **fix one end, query the past.***

**Step 1, restate.** Count pairs `(l, r)` with `0 ≤ l ≤ r < n` such that `nums[l] + … + nums[r] = k`. We're choosing a subarray: about n²/2 candidates. We're asked for a count. Fine print: values can be negative or zero, and the subarray must be non-empty.

**Step 2, budget.** n = 2·10⁴, so n²/2 = 2·10⁸. That's borderline at best, and an interviewer will expect better. Target O(n) or O(n log n).

**Step 3, examples.** `[1, 2, 3], k = 3` gives 2. Tiny: `[5], k = 5` gives 1. Adversarial: `[0, 0, 0], k = 0` gives 6 (every one of the 6 subarrays sums to 0). Also `[1, −1], k = 0` gives 1.

**Step 4, brute force.** Try every `(l, r)` and sum it. With level-0 optimisation, the sum for `(l, r)` extends the sum for `(l, r − 1)`:

```java
// Brute force: O(n^2) time. Keep it as the test oracle.
int count = 0;
for (int l = 0; l < n; l++) {
    int sum = 0;
    for (int r = l; r < n; r++) {
        sum += nums[r];          // sum of nums[l..r], extended by one element
        if (sum == k) count++;
    }
}
```

**Step 5, bottleneck.** For each right end `r`, we scan every left end `l`. That's **re-scanning**.

**Step 6, structure.** Question 2: *fix the right end `r`.* Then question 3: *what do I need to know about the past?*

Define prefix sums `P[0] = 0` and `P[i] = nums[0] + … + nums[i−1]`. Then

> sum(`nums[l..r]`) = `P[r+1] − P[l]`, so the subarray works ⟺ **`P[l] = P[r+1] − k`**.

For a fixed `r`, the question "how many `l ∈ [0, r]` work?" becomes "**how many earlier prefix values equal one specific number?**"

We could answer that by scanning all the earlier prefixes, but that's the same O(n²) again. What we need is a table that maps *a prefix value* to *how many times it has appeared so far*, and answers a lookup instantly. That table is a **hash map**: it stores values under keys (here, key = prefix value, value = count) and finds any key in roughly constant time, however many keys it holds.

> **A tempting wrong turn.** A natural idea is to keep a "window" `[l, r]`: move `r` right one step at a time, and whenever the window's sum is too big, drop elements from the left by moving `l` right. (This idea is called a *sliding window*; Ch 11 covers it properly.) Try it on `nums = [1, −1], k = 0`. At `r = 0`, the sum is 1 > 0, so you shrink, and the window becomes empty. At `r = 1`, the window `[−1]` sums to −1. You count 0, but the answer is 1 (the whole array).
>
> The window logic assumes that shrinking always lowers the sum and growing always raises it. **Negative numbers break that monotonicity.** When every value is positive, the window does work, in O(1) space (Ch 11).

**Step 7, prove it.** Every subarray has exactly one right end, so counting, for each `r`, the subarrays ending at `r` counts each subarray exactly once. For a fixed `r`, the subarrays ending at `r` with sum `k` correspond one-to-one to the indices `l ∈ [0, r]` with `P[l] = P[r+1] − k`. The algorithm keeps this **invariant**: *when we process `r`, the map holds the counts of exactly `P[0], …, P[r]`.* So adding `count[P[r+1] − k]` adds exactly the right number.

The invariant also fixes the **order of operations**: **query first, then insert `P[r+1]`**. If you insert first, then when `k = 0` you'd match `l = r + 1`, the empty subarray, and overcount by n.

**Step 8, budget.** One pass with O(1) expected work per element: **O(n) time, O(n) space**.

**Step 9, implement.**

```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int subarraySum(int[] nums, int k) {
        // countOfPrefix.get(p) = how many l in [0, r] have P[l] == p, where P[0] = 0 and
        // P[i] = nums[0] + ... + nums[i-1].
        Map<Integer, Integer> countOfPrefix = new HashMap<>();
        countOfPrefix.put(0, 1);              // P[0] = 0: the empty prefix
        int prefix = 0;                       // |prefix| <= 2*10^4 * 1000 = 2*10^7, fits in int
        int count = 0;
        for (int x : nums) {
            prefix += x;                                          // prefix == P[r+1]
            count += countOfPrefix.getOrDefault(prefix - k, 0);   // left ends with P[l] == P[r+1] - k
            countOfPrefix.merge(prefix, 1, Integer::sum);         // only now does P[r+1] join the past
        }
        return count;
    }
}
```

The two hash-map calls used:
* `getOrDefault(key, 0)` returns the count stored under `key`, or 0 if the key has never been stored.
* `merge(key, 1, Integer::sum)` adds 1 to the count under `key`, starting from 1 if the key is new. (`Integer::sum` is Java shorthand for "the function that adds two numbers", `(a, b) -> a + b`; `merge` uses it to combine the old count with the 1. `Map<Integer, Integer>` means "a map from `Integer` keys to `Integer` values". Ch 4 §8 explains both.)

Ch 4 §21 lists the other idioms.

The type check is part of the solution. The constraints bound the prefix by 2·10⁷ and `prefix − k` by 3·10⁷, so `int` is safe. If values could reach 10⁹, you'd need `long` and a `Map<Long, Integer>`.

**Step 10, test.** Trace `[1, 2, 3], k = 3`:

| r | x | prefix = P[r+1] | look for P[r+1] − k | found | count | map after |
| :-: | :-: | :-: | :-: | :-: | :-: | :--- |
| 0 | 1 | 1 | −2 | 0 | 0 | {0:1, 1:1} |
| 1 | 2 | 3 | 0 | 1 | 1 | {0:1, 1:1, 3:1} |
| 2 | 3 | 6 | 3 | 1 | 2 | {0:1, 1:1, 3:1, 6:1} |

The answer is 2. ✓ Now trace `[0, 0, 0], k = 0` yourself; you should get 6.

**Variations** *(previews of what an interviewer may ask next; each is solved in the chapter shown)*:

| Change | Adaptation |
| :--- | :--- |
| *Longest* subarray with sum k | Store the **first** index where each prefix value appears, not a count (Ch 8, Contiguous Array) |
| Sum *divisible by* k | Key the map by `P mod k`, normalised to be non-negative (Ch 8) |
| All values positive | Sliding window, O(1) space (Ch 11) |
| A 2D matrix | Fix a pair of rows, collapse the columns into one array, and reuse this algorithm (Ch 8, stretch) |

**Step 11, reflect.**
* *Insight:* a subarray sum is the difference of two prefix sums. Fix the right end and count the matching left prefixes.
* *Signal:* counting subarrays with an additive (or XOR) property, where negatives rule out a sliding window.

---

## 4. Derivation B: [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/)

*Move: **discard dominated candidates.***

> Given daily `temperatures` (`1 ≤ n ≤ 10⁵`, `30 ≤ t ≤ 100`), return `answer[i]` = the number of days after day `i` until a warmer temperature, or 0 if there is none.
>
> Example: `[73, 74, 75, 71, 69, 72, 76, 73]` → `[1, 1, 4, 2, 1, 1, 0, 0]`.

**Steps 1–3.** For each `i`, find the smallest `j > i` with `t[j] > t[i]`. Budget: n = 10⁵, so n² = 10¹⁰ is out. Adversarial case: strictly decreasing temperatures, where nobody ever gets a warmer day.

**Step 4, brute force.** For each day, scan to the right until you find a warmer day. On strictly decreasing input every scan runs to the end, giving about n²/2 steps.

**Step 5, bottleneck.** Re-scanning, and the scans overlap heavily: day 3 and day 4 both walk over days 5, 6, ….

**Step 6, structure.** Flip the viewpoint. Instead of each day *searching* forward, let each new day **resolve** the earlier days that are still waiting. Process days left to right and keep the set W of days whose answer isn't known yet. When day `i` arrives, it is the answer for exactly the waiting days that are colder than `t[i]`. Those days were still waiting, so nothing between them and `i` was warmer, which makes `i` the *first* warmer day.

What does W look like? Question 5 (dominance) gives the key fact:

> **Invariant.** The waiting days, taken in index order, have **non-increasing** temperatures.
>
> *Proof.* Suppose `j < j'` are both waiting and `t[j] < t[j']`. When `j'` arrived, `j` was already waiting and colder, so `j'` would have resolved it. That's a contradiction.

Because of this, the waiting days colder than `t[i]` are always a run at the **end** of W, the most recent ones. So remove days from the end while the last waiting day is colder, then add `i` at the end. A list used only at one end like this is called a **stack**. Adding is a *push*, removing is a *pop*. Here it's simply an `int[]` with a `size` counter.

**Step 7, prove it.** Correctness follows from the two facts above. Notice why the stack is *non-increasing* rather than strictly decreasing: equal temperatures don't resolve each other (we need *strictly* warmer), so equal days sit side by side.

**Step 8, budget.** A single day can pop many others, so is this O(n²) in the worst case? No. **Each index is pushed once and popped at most once**, so the total number of stack operations over the whole run is at most 2n. That's O(n) total, or **amortised O(1) per day**. Ch 3 formalises this kind of argument.

**Step 9, implement.**

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] answer = new int[n];        // 0 by default = "no warmer day"
        int[] waiting = new int[n];       // stack of indices with no warmer day found yet;
        int size = 0;                     // their temperatures are non-increasing, bottom to top
        for (int i = 0; i < n; i++) {
            while (size > 0 && temperatures[waiting[size - 1]] < temperatures[i]) {
                int j = waiting[--size];  // day i is the first day warmer than day j
                answer[j] = i - j;
            }
            waiting[size++] = i;
        }
        return answer;
    }
}
```

An `int[]` with a `size` counter is the fastest stack in Java. Ch 12 compares it with `ArrayDeque`.

**A second solution, from question 10.** Temperatures only take 71 values (30 to 100). Scan right to left, keeping `next[t]` = the nearest index to the right with temperature exactly `t`. For day `i`, the answer comes from the minimum of `next[t]` over all `t > temperatures[i]`:

```java
int n = temperatures.length;
int[] answer = new int[n];
int[] next = new int[101];
java.util.Arrays.fill(next, Integer.MAX_VALUE);           // "never seen"
for (int i = n - 1; i >= 0; i--) {
    int nearest = Integer.MAX_VALUE;
    for (int t = temperatures[i] + 1; t <= 100; t++) nearest = Math.min(nearest, next[t]);
    answer[i] = (nearest == Integer.MAX_VALUE) ? 0 : nearest - i;
    next[temperatures[i]] = i;
}
```

That's O(71 · n) and also passes. The lesson is that **a tiny value range is itself a signal**. The stack solution is still the one to know, because it works when temperatures go up to 10⁹.

**Variations** *(previews; each is solved in the chapter shown)*:

| Change | Adaptation |
| :--- | :--- |
| Circular array (wrap around) | Iterate over `2n` indices using `i % n` (Ch 12) |
| *Previous* warmer day | Scan from the right, or read the answer off the stack top before pushing |
| Warmer *or equal* | Change `<` to `<=` |
| Sum of minimums over all subarrays | Each element's previous/next smaller boundaries give its contribution (Ch 12) |

**Step 11, reflect.**
* *Insight:* a waiting day is resolved by the first warmer day. Waiting days form a non-increasing stack, and each index is pushed and popped at most once.
* *Signal:* "for each element, the nearest element to its right (or left) that beats it."

---

## 5. Derivation C: [Longest Arithmetic Subsequence of Given Difference](https://leetcode.com/problems/longest-arithmetic-subsequence-of-given-difference/)

*Move: **define the answer by its last element.***

> Given `arr` (`1 ≤ n ≤ 10⁵`) and an integer `difference` (all values in `[−10⁴, 10⁴]`), return the length of the longest subsequence in which every pair of adjacent elements differs by exactly `difference`.
>
> Example: `arr = [1, 5, 7, 8, 5, 3, 4, 2, 1], difference = −2` → `4` (`7, 5, 3, 1`).

**Steps 1–4.** We're choosing a subsequence: 2ⁿ candidates. The brute force is hopeless at n = 10⁵.

**Step 6, structure.** Question 2: *fix the last element.* Define

> `best(i)` = the length of the longest valid subsequence that **ends exactly at index `i`**.

Now ask about the **last decision**: what comes just before `arr[i]`? Either nothing (length 1), or some `j < i` with `arr[j] = arr[i] − difference`. In the second case, the part ending at `j` should itself be as long as possible:

> `best(i) = 1 + max{ best(j) : j < i, arr[j] = arr[i] − difference }`, where the max of an empty set is 0. The answer is `max over i of best(i)`.

Why must the part ending at `j` be optimal? If some longer valid subsequence also ended at `j`, you could swap it in. It still ends with the value `arr[i] − difference`, so appending `arr[i]` stays valid, and the result is longer. This "cut and paste" argument is called *optimal substructure*; Ch 27 builds all of dynamic programming on it.

Evaluating the recurrence directly costs O(n²) = 10¹⁰. The new **bottleneck** is re-scanning: for each `i`, we look for the best `j` among earlier indices *with one specific value*.

Question 3: *what's the least I need from the past?* Only this: for each value `v`, the best length of a subsequence seen so far that ends with value `v`. That's a lookup by value, so use a hash map.

**Step 7, the one subtle point.** When the same value appears again, we **overwrite** its entry. Is that safe, or could we lose a better older value? It's safe, because entries never decrease. Here's why. The only write to key `x` stores `1 + entry(x − difference)` *as of now*. By induction, `entry(x − difference)` now is at least what it was at any earlier time, so the new value for `x` is at least the old one. (When `difference = 0`, the new value is exactly the old value plus one.)

**Step 9, implement.**

```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int longestSubsequence(int[] arr, int difference) {
        // bestEndingWith.get(v) = longest valid subsequence among the elements seen so far
        // whose last element equals v. Entries never decrease, so overwriting is safe.
        Map<Integer, Integer> bestEndingWith = new HashMap<>();
        int best = 0;
        for (int x : arr) {
            int length = bestEndingWith.getOrDefault(x - difference, 0) + 1;
            bestEndingWith.put(x, length);
            best = Math.max(best, length);
        }
        return best;
    }
}
```

**O(n) expected time, O(n) space.** Question 10 applies here too. Values lie in `[−10⁴, 10⁴]`, so `x − difference` lies in `[−2·10⁴, 2·10⁴]`, and an `int[40_001]` with an offset of 20 000 replaces the map. That's several times faster in Java, because it avoids hashing and boxing.

**Variations** *(previews; each is solved in the chapter shown)*:

| Change | Adaptation |
| :--- | :--- |
| Any common difference (not given) | `dp[i][d]`, O(n²), Mixed Set VI |
| *Count* such subsequences | Sum instead of max; watch for overflow, usually mod 10⁹ + 7 |
| Return the subsequence itself | Store the predecessor index for each `i`, then walk back |

**Step 11, reflect.**
* *Insight:* define the best answer ending at each element. The previous element is forced to have value `x − d`, so the past only needs to answer "best length ending with value v", which is a hash lookup.
* *Signal:* optimising over a subsequence where each element constrains the one before it.

---

## 6. Derivation D: [Magnetic Force Between Two Balls](https://leetcode.com/problems/magnetic-force-between-two-balls/)

*Move: **turn "optimise" into "decide".***

> There are `n` distinct `position`s on a line (`2 ≤ n ≤ 10⁵`, `1 ≤ position[i] ≤ 10⁹`) and `m` balls (`2 ≤ m ≤ n`). Place the balls at positions so that the **minimum** distance between any two balls is as **large** as possible. Return that distance.
>
> Example: `position = [1, 2, 3, 4, 7], m = 3` → `3` (balls at 1, 4 and 7).

**Steps 1–4.** We're choosing m of n positions: C(n, m) candidates, which is astronomical. We're asked for a maximum of a minimum.

**Step 6, structure.** Optimising is hard, but *checking* a proposed answer is easy. Question 9: change the question to

> `feasible(g)`: can I place all `m` balls so that every gap is at least `g`?

Question 4: *is it monotone?* Yes. If `feasible(g)` holds and `g' < g`, the same placement has every gap ≥ g', so `feasible(g')` holds too. Listing `g = 1, 2, 3, …`, the answers look like **T T T … T F F F**, and the answer to the problem is the **last T**.

**Finding the last T without trying every g.** Trying g = 1, 2, 3, … one by one could take 10⁹ checks. Instead, keep two numbers: `lo`, a gap you *know* is feasible, and `hi`, a gap you *know* is infeasible. Check the gap `mid` halfway between them:
* if `mid` is feasible, the last T is at `mid` or later, so move `lo` up to `mid`;
* otherwise it's before `mid`, so move `hi` down to `mid`.

Each check halves the distance between `lo` and `hi`. After about 30 checks (2³⁰ ≈ 10⁹), they're next to each other, and `lo` is the answer. This halving idea is called **binary search**; Ch 10 makes it a general tool.

**Deciding `feasible(g)`.** Sort the positions. Place the first ball at the smallest position. Then repeatedly place the next ball at the **first position at least `g` beyond the previous ball**. Feasible means you placed at least `m` balls.

**Step 7, prove the greedy check (it "stays ahead").** Let the greedy positions be `x₁ < x₂ < …` and take *any* valid placement `y₁ < y₂ < … < y_m` with all gaps ≥ g. We claim `xₜ ≤ yₜ` for every `t` that greedy reaches.

* For `t = 1`: `x₁` is the smallest position, so `x₁ ≤ y₁`.
* If `xₜ ≤ yₜ`, then `y_{t+1} ≥ yₜ + g ≥ xₜ + g`. So `y_{t+1}` is a candidate for greedy's next ball. Greedy takes the *first* such position, so `x_{t+1} ≤ y_{t+1}`.

So whenever any valid placement of m balls exists, greedy also places at least m. Ch 2 names this argument and makes it a tool.

**Step 9, implement.** The binary search keeps an **invariant**: `lo` is always feasible and `hi` is always infeasible. Ch 10 shows why this one template replaces every off-by-one variant you've seen.

```java
import java.util.Arrays;

class Solution {
    public int maxDistance(int[] position, int m) {
        int[] p = position.clone();          // don't silently reorder the caller's array
        Arrays.sort(p);
        // Invariant: canPlace(lo) is true, canPlace(hi) is false.
        int lo = 1;                          // positions are distinct and m <= n, so gap 1 always works
        int hi = p[p.length - 1] - p[0] + 1; // with m >= 2, a gap wider than the whole span is impossible
        while (hi - lo > 1) {
            int mid = lo + (hi - lo) / 2;    // strictly between lo and hi
            if (canPlace(p, m, mid)) lo = mid; else hi = mid;
        }
        return lo;                           // the largest feasible gap
    }

    // Greedy: each ball goes at the leftmost position at least `gap` past the previous ball.
    private boolean canPlace(int[] p, int m, int gap) {
        int placed = 1;
        int last = p[0];
        for (int i = 1; i < p.length && placed < m; i++) {
            if (p[i] - last >= gap) {
                placed++;
                last = p[i];
            }
        }
        return placed >= m;
    }
}
```

**Step 10, test.** For sorted `[1, 2, 3, 4, 7]`, `m = 3`, start with `lo = 1`, `hi = 7`:

| lo | hi | mid | greedy placement | feasible? |
| :-: | :-: | :-: | :--- | :-: |
| 1 | 7 | 4 | 1, then the first ≥ 5 is 7: only 2 balls | F → `hi = 4` |
| 1 | 4 | 2 | 1, 3, 7 | T → `lo = 2` |
| 2 | 4 | 3 | 1, 4, 7 | T → `lo = 3` |
| 3 | 4 | — | `hi − lo = 1`, stop | answer **3** ✓ |

**Step 8, budget.** Sorting is O(n log n), plus about 30 greedy passes of O(n) each: **O(n log n + n log(span))**. Every quantity stays below 10⁹, so `int` is safe.

**Variations** *(previews; each is solved in the chapter shown)*:

| Change | Adaptation |
| :--- | :--- |
| *Minimise the maximum* (ship packages in D days, split an array into k parts) | The mirror image: feasibility is F…F T…T, so find the first T (Ch 10) |
| Real-valued positions | Binary search on `double` with a fixed number of iterations (Ch 10) |
| Return the placement | Rerun the greedy at the final answer |

**Step 11, reflect.**
* *Insight:* "maximise the minimum" becomes "is gap g achievable?". Feasibility is monotone in g, and a leftmost greedy decides it.
* *Signal:* maximise a minimum or minimise a maximum, where *checking* a candidate answer is easy.

---

## 7. When you're stuck: 13 unstuck moves

When 10 minutes pass with no idea, don't stare. Work through these **in writing**, one at a time:

| # | Move | Example of it working |
| :-: | :--- | :--- |
| 1 | **Shrink.** Solve n = 1, 2, 3 by hand and tabulate the answers | The answers form a pattern or a recurrence |
| 2 | **Brute-force and print.** Run your brute force on small inputs and stare at the *optimal solutions*, not just the numbers | You notice the optimum is always sorted, or always uses the maximum |
| 3 | **Fix something.** The right endpoint, the last element, the maximum, the middle, the answer value | Derivations A, C and D |
| 4 | **Sort, or process in a better order.** By value, by end time, by deadline | Interval scheduling sorted by end time (Ch 14) |
| 5 | **Reverse.** Time (deletions become additions), direction (right to left), or roles (from target to source) | "Remove edges one by one" becomes "add them in reverse" with union-find (Ch 24) |
| 6 | **Complement.** Count the bad ones; "fewest removed" = n − "most kept" | Minimum Operations to Reduce X to Zero becomes the longest middle window (Ch 11) |
| 7 | **Change what you iterate over.** Values instead of indices, answers instead of decisions, each element's contribution instead of each structure | Sum of subarray minimums (Ch 12) |
| 8 | **Relax a constraint.** No negatives, distinct values, k = 1, 1D instead of 2D. Solve that, then see what breaks when you put the constraint back | Positives-only subarray sum → window; then negatives → prefix map (Derivation A) |
| 9 | **Look for an invariant.** What never changes under the allowed operations? Parity, sum, gcd, the multiset of values | Many "can you reach this state?" problems are parity checks |
| 10 | **Look for monotonicity.** If x works, does x + 1? | Derivation D |
| 11 | **Make it a graph.** States as nodes, moves as edges; fewest moves → BFS | Open the Lock (Ch 21) |
| 12 | **Massage an optimal solution.** "Take any optimal answer; I can change it into one with property P without making it worse." | Exchange arguments (Ch 2, Ch 26) |
| 13 | **Bound the answer.** Is it always ≤ 2? ≤ log n? ≤ 26? Small answers can be tried one by one | Minimum Number of Days to Disconnect Island: the answer is 0, 1 or 2 (Ch 25) |

If all 13 fail, take a hint, but first write two sentences: what you tried, and why each attempt failed. Writing those often unsticks you on its own. When it doesn't, it turns the hint into a lesson rather than a spoiler.

---

## 8. Where the protocol fails

The protocol fails in predictable ways. Each one has a fix:

* **Pattern-guessing.** You jump straight to "is this a heap problem?" without Steps 4–6. You're fitting a solution to a problem you haven't taken apart. *Fix:* write down the brute force and its bottleneck before naming any technique.
* **An unproven greedy.** You skip Step 7, code a plausible rule, and spend 40 minutes debugging something that was never correct. *Fix:* the one-sentence justification plus two minutes of trying to break it. Ch 2 has a gallery of greedy rules that look right and aren't.
* **Over-optimising.** n ≤ 2000 and O(n²) is intended, but you burn the interview on an O(n log n) idea. *Fix:* Step 2 comes before Step 5 for a reason.
* **Working in your head.** You go round in circles and lose track of which ideas you've already rejected. *Fix:* paper.
* **Never reflecting.** The same failure repeats across problems. *Fix:* the two log lines, every time.

---

## 9. Recognition: the four moves

| Move | Signal in the problem | Becomes |
| :--- | :--- | :--- |
| Fix one end, query the past | Count or find the longest subarrays with an additive or XOR property | Prefix-state hashing (Ch 8) |
| Discard dominated candidates | "For each element, the nearest one to the left/right that is bigger/smaller" | Monotonic stack / deque (Ch 12) |
| Define the answer by its last element | Optimise over subsequences or sequences of choices, where each choice depends on a few earlier ones | Dynamic programming (Ch 27–33) |
| Turn optimise into decide | "Maximise the minimum", "minimise the maximum", "the smallest x such that…", with an easy check | Binary search on the answer (Ch 10) |

**False friends:**
* "Subarray" + negative numbers looks like sliding window, but isn't (Derivation A).
* "Maximise the minimum" where feasibility *isn't* monotone in the answer is rare, but check the monotonicity argument every time instead of assuming it.

---

## 10. Practice

**How to practise this chapter.** For each problem, write Steps 1–8 as a comment block at the top of your file *before* writing code: restate, budget, brute force, bottleneck, the structural question that worked, and your one-sentence proof. Timebox Easy problems at 15 minutes and Medium at 40. Then do Step 11 in your log.

The problems are easiest first. None needs a technique beyond this chapter, only the moves.

1. **[Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)** *(Easy)*
2. **[Majority Element](https://leetcode.com/problems/majority-element/)** *(Easy; follow-up: O(1) extra space)*
3. **[Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)** *(Medium)*
4. **[Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/)** *(Medium; no division allowed)*
5. **[Jump Game](https://leetcode.com/problems/jump-game/)** *(Medium)*
6. **[Gas Station](https://leetcode.com/problems/gas-station/)** *(Medium)*

**Stretch:** **[Next Permutation](https://leetcode.com/problems/next-permutation/)** *(Medium)* · **[Candy](https://leetcode.com/problems/candy/)** *(Hard)*

<details><summary>Nudges: open one only after 15 minutes stuck</summary>

1. Fix the *sell* day. What do you need to know about the days before it?
2. Pair up and cancel two different elements. What survives? Prove that the majority element survives.
3. Fix the right end. What is the best subarray *ending here*, in terms of the best ending one step earlier?
4. The product of everything except `i` = (product of everything to the left) × (product of everything to the right).
5. Scan left to right and track the furthest reachable index. When does the scan fail?
6. If the total gas is at least the total cost, a valid start exists. Where does the running tank balance hit its minimum?
7. Stretch, Next Permutation: what's the smallest change you can make near the *right* end of the array?
8. Stretch, Candy: there are two one-sided rules (left neighbour, right neighbour). Can you satisfy each one in its own pass?
</details>

<details><summary>Key</summary>

Buy/Sell → fix the sell day, keep the minimum of the past (move 1 of §9) · Majority → cancellation invariant (Boyer–Moore; proof in Ch 2) · Max Subarray → best ending here (move 3) · Product Except Self → split around i into a prefix and a suffix · Jump Game → reach invariant · Gas Station → total ≥ 0 plus "start after the prefix minimum" · Next Permutation → find the rightmost ascent, swap it with the next larger element, reverse the suffix · Candy → two passes plus a max
</details>

---

## 11. Self-test

Commit to an answer before opening each one.

<details><summary>1. n ≤ 3000. Which of O(n³), O(n² log n), O(n²) fit?</summary>

n² = 9·10⁶: yes. n² log n ≈ 1.1·10⁸: borderline, probably fine with primitive arrays. n³ = 2.7·10¹⁰: no.
</details>

<details><summary>2. n, q ≤ 10⁵, and each query scans the whole array. OK?</summary>

No: 10¹⁰ operations. You need preprocessing plus O(log n) or O(1) per query.
</details>

<details><summary>3. In Derivation A, what goes wrong if you insert the current prefix before querying?</summary>

When k = 0, every r matches l = r + 1, the empty subarray, so the answer is inflated by n. For other k it happens to work, which makes it a bug your examples might never catch.
</details>

<details><summary>4. In Derivation B, why is the stack non-increasing rather than strictly decreasing?</summary>

Equal temperatures don't resolve each other, since the answer needs a strictly warmer day. So equal days stay side by side on the stack.
</details>

<details><summary>5. In Derivation D, state the monotonicity in one sentence and say why it's true.</summary>

If gap g is achievable then every smaller gap is too, because the same placement has all gaps ≥ g > g'.
</details>

<details><summary>6. A problem says "return the number of valid sequences modulo 10⁹ + 7", with n ≤ 1000. What does the modulus tell you?</summary>

You're counting something astronomically large, so you can't enumerate it. Expect a DP (or combinatorics), probably O(n²) given n ≤ 1000.
</details>

<details><summary>7. Classify the bottleneck: "for each i, compute the maximum of a[i..i+k−1] by scanning the window".</summary>

Re-scanning. Consecutive windows share k − 1 elements, and most of those elements can never be a future maximum (dominance). That leads to the monotonic deque (Ch 12).
</details>

---

## 12. Exit gate

Move on when you can:

* **Rebuild** Derivations A, B and D from a blank file, in under 10 minutes each, compiling first try.
* **Explain** each of the four moves, *and* the proof behind its derivation, out loud in under 2 minutes, including the counterexample that kills the sliding window in Derivation A.
* **Solve** at least 5 of the 6 practice problems without opening the nudges.
