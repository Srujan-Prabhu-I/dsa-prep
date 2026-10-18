# Chapter 33 — DP VII: Faster Transitions

> **Part VI · Week 28** · **S-tier** · Prerequisites: Ch 8 (prefix sums), Ch 10 (boundary binary search), Ch 12 (monotonic deque), Ch 18 (modular arithmetic), Ch 27–29
> **Problems:** 5 worked (1 ★) · 5 practice · 2 stretch · **Time:** about 7 hours

Ch 27 step 6 says a DP costs **(number of states) × (cost of one transition)**. Every DP so far had cheap transitions: two or three options, or a short loop. This chapter is about the DPs whose state count is fine but whose transition is **a loop over many earlier states**, giving O(n²) or O(n·k) when only O(n log n) passes.

The method doesn't change. You still write the state and the transition first, and you write the **slow** version first. Then you look at *which* earlier states the transition reads, and notice they form a **shape**:

| The transition reads… | …and takes their | Make it fast with | Section |
| :--- | :--- | :--- | :--- |
| a **prefix** of the states, in some sorted order | max (and the max only grows) | sort + binary search for the boundary | §2 |
| a **sliding window** of the last k states | max or min | a monotonic deque (Ch 12 §5) | §3 |
| a **window** of states | count or sum | prefix sums over the states (Ch 8) | §4–5 |

Each speed-up replaces "loop over the range" with "ask a structure about the range". **Keep the slow version:** it's the brute force for your stress test (Ch 5 §6).

---

## 1. Attempt first ★

> **[Maximum Profit in Job Scheduling](https://leetcode.com/problems/maximum-profit-in-job-scheduling/) (40 minutes).** n ≤ 5·10⁴ jobs, each with `startTime`, `endTime` and `profit`. Choose jobs that don't overlap (a job may start exactly when the previous one ends) to maximise the total profit. Write the O(n²) DP first, then make it fast.

---

## 2. Predecessors that form a sorted prefix

### Worked F1 · [Maximum Profit in Job Scheduling](https://leetcode.com/problems/maximum-profit-in-job-scheduling/) ★

**Why greedy fails.** Ch 14's "earliest end first" maximises the *number* of jobs, not their total profit. One long, very profitable job beats three short cheap ones (Ch 26 §6, museum entry 3), so we need a DP.

**The slow DP.** Sort the jobs by end time. Consider the jobs in that order and ask Ch 27's question, *what is the last decision?* For the i-th job: either it isn't taken, or it is, and then the jobs before it must all end by its start.

> `best[i]` = the maximum profit using only the first i jobs (in end order).
> `best[i] = max(best[i − 1], profit of job i−1 + best[j])`, where the first j jobs are exactly those ending at or before job i−1's start.

**Why "the first j jobs" is a prefix.** The jobs are sorted by end time, so the jobs ending at or before a given time are an initial segment of the order. The slow version finds j by scanning: O(n) per job, O(n²) in total, 2.5·10⁹ steps.

**Two facts make it fast:**
1. `best` is **non-decreasing** (more jobs available can't lower the best profit). So the best compatible predecessor is simply the **largest** j that is compatible, with no need to compare candidates.
2. "Largest j with `end[j − 1] ≤ start`" is a **boundary** in a sorted array: the first index whose end exceeds `start`, which is Ch 10's binary search.

```java
public int jobScheduling(int[] startTime, int[] endTime, int[] profit) {
    int n = startTime.length;
    Integer[] order = new Integer[n];
    for (int i = 0; i < n; i++) order[i] = i;
    Arrays.sort(order, (a, b) -> Integer.compare(endTime[a], endTime[b]));
    int[] ends = new int[n];
    for (int i = 0; i < n; i++) ends[i] = endTime[order[i]];

    int[] best = new int[n + 1];                        // best[i]: the first i jobs in end order
    for (int i = 1; i <= n; i++) {
        int job = order[i - 1];
        int j = firstGreater(ends, startTime[job]);     // jobs 0..j-1 end at or before this start
        best[i] = Math.max(best[i - 1], profit[job] + best[j]);
    }
    return best[n];
}

// The first index whose value is > x (n if none): Ch 10's boundary search.
private int firstGreater(int[] a, int x) {
    int lo = -1, hi = a.length;                       // virtual sentinels: false at -1, true at n
    while (hi - lo > 1) {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] > x) hi = mid; else lo = mid;
    }
    return hi;
}
```

**O(n log n).** Job `i − 1` itself never counts as its own predecessor: its end is greater than its start, so `j ≤ i − 1`. Total profit is at most 5·10⁴ · 10⁴ = 5·10⁸, which fits in an `int`. Check this every time; Events II (practice) doesn't change the bound, but other variants do.

**Alternative, in prose.** Sort by *start* and fill from the right: `best[i] = max(best[i + 1], profit[i] + best[first job starting ≥ end[i]])`. Or keep a `TreeMap<endTime, bestProfit>` and use `floorEntry` (Ch 15). All three are the same idea: the predecessors form a sorted prefix, and a monotone `best` makes the boundary the answer.

**"Strict or not?"** This problem allows a job to start at the exact moment the previous one ends, so the boundary is `end ≤ start` (first *greater*). In Two Best Non-Overlapping Events and Events II (practice) the end time is **inclusive**, so the boundary becomes `end < start` (first *greater or equal*). One character, and the most common bug in this family.

---

## 3. Predecessors in a sliding window: max or min

### Worked F2a · [Jump Game VI](https://leetcode.com/problems/jump-game-vi/)

> From index i you can jump to any of i + 1 … i + k. Your score is the sum of `nums` over the indices you land on, including the first and the last. Return the maximum score to reach index n − 1. n, k ≤ 10⁵.

**The slow DP.** `score[i] = nums[i] + max(score[i − k .. i − 1])`. That's O(n·k) = 10¹⁰.

**The shape:** the range is the **last k states**, a window that slides right by one each step. "Max of every sliding window" is exactly Ch 12 §5's monotonic deque. The only difference from Sliding Window Maximum is that the values entering the window are the DP's own results, computed one step earlier.

**Why the deque is still correct (dominance, Ch 12 §5).** If j < j′ and `score[j] ≤ score[j′]`, then j can never be the window max again: j′ is at least as good *and* leaves the window later. So the deque holds indices with **strictly decreasing** scores, and the front is the max.

```java
public int maxResult(int[] nums, int k) {
    int n = nums.length;
    int[] score = new int[n];
    score[0] = nums[0];
    ArrayDeque<Integer> window = new ArrayDeque<>();   // indices; scores strictly decreasing front to back
    window.add(0);
    for (int i = 1; i < n; i++) {
        if (window.peekFirst() < i - k) window.pollFirst();               // left the window [i-k, i-1]
        score[i] = nums[i] + score[window.peekFirst()];
        while (!window.isEmpty() && score[window.peekLast()] <= score[i]) window.pollLast();   // dominated
        window.addLast(i);
    }
    return score[n - 1];
}
```

**O(n):** each index enters and leaves the deque once. **Order inside the loop matters:** expire the front, *then* read the max, *then* push i. Pushing before reading would let i be its own predecessor. Expiring one index per step is enough here because exactly one index leaves the window each step.

### Worked F2b · [Constrained Subsequence Sum](https://leetcode.com/problems/constrained-subsequence-sum/)

> Choose a non-empty subsequence of `nums` such that any two consecutive chosen indices are at most k apart. Maximise its sum. Values may be negative.

**The state.** `best[i]` = the maximum sum of a valid subsequence that **ends by choosing i**. Its previous chosen index is within `i − k .. i − 1`, or there is none (i starts the subsequence):

> `best[i] = nums[i] + max(0, max best[i − k .. i − 1])`

The answer is `max(best[i])` over all i, because the subsequence can end anywhere. This is Jump Game VI with two changes: the `max(0, …)` lets a subsequence start fresh (Kadane's idea, Ch 1 practice), and the answer is a max over all ends.

```java
public int constrainedSubsetSum(int[] nums, int k) {
    int n = nums.length, answer = Integer.MIN_VALUE;
    int[] best = new int[n];
    ArrayDeque<Integer> window = new ArrayDeque<>();
    for (int i = 0; i < n; i++) {
        if (!window.isEmpty() && window.peekFirst() < i - k) window.pollFirst();
        best[i] = nums[i] + (window.isEmpty() ? 0 : Math.max(0, best[window.peekFirst()]));
        answer = Math.max(answer, best[i]);
        while (!window.isEmpty() && best[window.peekLast()] <= best[i]) window.pollLast();
        window.addLast(i);
    }
    return answer;
}
```

**Sums fit in `int`:** at most 10⁵ · 10⁴ = 10⁹. With a larger bound, this is where `long` is needed.

**Min instead of max** (minimum cost to reach i using a jump of at most k) is the same deque with the comparison flipped. Delivering Boxes (stretch) applies it to a cost with an extra prefix-sum term.

---

## 4. Predecessors in a window: count or sum

### Worked F3 · [Jump Game VII](https://leetcode.com/problems/jump-game-vii/)

> A binary string s with `s[0] == '0'`. From index i you may jump to any j with `i + minJump ≤ j ≤ i + maxJump` and `s[j] == '0'`. Can you reach the last index? n ≤ 10⁵.

**The slow DP.** `reach[j]` = true if `s[j] == '0'` and some `reach[i]` is true for i in `[j − maxJump, j − minJump]`. Scanning the window costs O(maxJump − minJump) per index, 10¹⁰ in the worst case.

**The shape:** we don't need the max, just **whether any** state in a window is true. "Any" is "count > 0", and the count of trues in a window is a difference of **prefix counts** over the `reach` array itself, built as we go. The window ends at `j − minJump < j`, so its prefix count is already known when we reach j.

```java
public boolean canReach(String s, int minJump, int maxJump) {
    int n = s.length();
    boolean[] reach = new boolean[n];
    int[] reachedBefore = new int[n + 1];          // reachedBefore[x] = how many of 0..x-1 are reachable
    reach[0] = true;
    reachedBefore[1] = 1;
    for (int j = 1; j < n; j++) {
        int lo = j - maxJump, hi = j - minJump;    // i must lie in [lo, hi]
        if (s.charAt(j) == '0' && hi >= 0) {
            reach[j] = reachedBefore[hi + 1] - reachedBefore[Math.max(lo, 0)] > 0;
        }
        reachedBefore[j + 1] = reachedBefore[j] + (reach[j] ? 1 : 0);
    }
    return reach[n - 1];
}
```

**O(n).** **Alternatives, in prose:** keep a running count of reachable indices in the window, adding the one that enters and subtracting the one that leaves (the sliding version of the same idea). Or BFS (Ch 21) that remembers the farthest index already scanned, so no index is examined twice.

---

## 5. Sums over the previous row

### Worked F4 · [K Inverse Pairs Array](https://leetcode.com/problems/k-inverse-pairs-array/)

> How many permutations of 1..n have exactly k inverse pairs (pairs i < j with a[i] > a[j])? n, k ≤ 1000, answer mod 10⁹ + 7.

**The state, by building the permutation up.** Take a permutation of 1..i−1 and **insert i**, the largest value. Wherever it goes, it forms an inverse pair with exactly the elements placed **after** it, and with nothing else. Putting it with p elements after it (p = 0 … i − 1) adds exactly p pairs. Every permutation of 1..i arises exactly once this way (remove i to recover the smaller permutation), so the options cover everything and don't overlap, as counting DP requires (Ch 27 §1).

> `ways[i][j]` = the number of permutations of 1..i with exactly j inverse pairs.
> `ways[i][j] = Σ ways[i − 1][j − p]` for p = 0 … min(j, i − 1).

**The slow version** is states (n·k = 10⁶) × transition (up to n) = 10⁹. **The shape:** the sum reads a **contiguous range of the previous row**, `ways[i − 1][j − i + 1 .. j]`. So build the previous row's prefix sums once, and each entry costs O(1):

```java
public int kInversePairs(int n, int k) {
    final int MOD = 1_000_000_007;
    long[] prev = new long[k + 1];                 // row i = 1: only the identity, with 0 pairs
    prev[0] = 1;
    for (int i = 2; i <= n; i++) {
        long[] prefix = new long[k + 2];           // prefix[x] = prev[0] + … + prev[x - 1]
        for (int x = 0; x <= k; x++) prefix[x + 1] = (prefix[x] + prev[x]) % MOD;
        long[] cur = new long[k + 1];
        for (int j = 0; j <= k; j++) {
            int from = Math.max(0, j - (i - 1));   // the range prev[from .. j]
            cur[j] = (prefix[j + 1] - prefix[from] + MOD) % MOD;
        }
        prev = cur;
    }
    return (int) prev[k];
}
```

**O(n·k) = 10⁶.** **`+ MOD` before `% MOD`:** both prefix values are already reduced, so their difference can be negative (Ch 18 §4). **Alternative, in prose:** the sliding form `cur[j] = cur[j − 1] + prev[j] − prev[j − i]` (when j − i ≥ 0) is the same sum maintained incrementally, without a prefix array.

---

## 6. The general move, and what lies beyond

1. **Write the slow DP** and test it. The speed-up never changes the answer, only the cost, so the slow version is your oracle.
2. **Look at the transition's range** of earlier states, and what is taken over it.
3. **Match the shape:**

| Shape of the range | Taken over it | Tool | Where |
| :--- | :--- | :--- | :--- |
| a prefix in sorted order, `best` monotone | max | binary search for the boundary | §2 |
| a prefix in sorted order, `best` not monotone | max | a running prefix max, then binary search | Two Best Events (practice) |
| the last k indices | max / min | monotonic deque | §3 |
| every state of the previous row **except one index** (e.g. "any column but this one") | max / min | keep the previous row's best and second best, plus the index of the best; use the second when the excluded index is the best one | the top-two trick of Ch 31 §5 |
| any window | count / sum | prefix sums over the DP values | §4–5 |
| a set of states that enter and leave **out of order** (each state is valid only until its own deadline) | max / min | a heap with lazy deletion: pop the top while it has expired (Ch 15); or a `TreeMap` multiset | Jump Game VI also passes this way, in O(n log n) |
| the states whose **value** (not index) is < x | max / count | a Fenwick or segment tree indexed by value: **Ch 34 preview** | LIS-style DPs |
| states read through a linear function, `dp[j] + a[j]·x` | min / max | convex hull trick: **outside this handbook's scope** (Ch 37 guide) | rare in interviews |

**The last two rows are signposts, not material for this chapter.** Ch 34 teaches the trees. The convex hull trick and the "Knuth" and "divide and conquer" DP optimisations are competitive-programming tools; Ch 37 explains how to recognise them and why interviews almost never require them.

---

## 7. Where these break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| `best` isn't monotone over the sorted order | The latest compatible predecessor may not be the best | Store a running prefix max, then binary search |
| The window's ends don't both move forward | The deque's "leaves later" argument fails | A segment tree (Ch 34) or a `TreeMap` multiset (Ch 15) |
| The constraint is on a **value**, e.g. "previous element smaller" | The range isn't contiguous in index order | Sort by value, or index a Fenwick tree by value (Ch 34) |
| A max over a window *with deletions in the middle* | A deque only removes from the ends | A `TreeMap` multiset (Ch 15) |
| Sums mod p, then a difference | Negative intermediate | `(a − b + MOD) % MOD` |

---

## 8. Recognition

**Signals:**
* You've written a correct DP, and its cost is O(n²) or O(n·k) with n up to 10⁵ (or n·k·n with n, k up to 10³).
* The transition reads "every earlier state with **end ≤ my start**", or "the previous **k** states", or "states **between** i − a and i − b".
* Weighted intervals: start, end and a weight. Unweighted, it's greedy (Ch 14); weighted, it's F1.
* "At most k apart", "jump up to k", "within the last k days".
* A count over permutations or sequences with a range of contributions per new element (F4's "insert the largest" view).

**False friends:**
* **Unweighted** interval selection (Non-overlapping Intervals, Ch 14): greedy by end time, no DP.
* Window max where values only increase: a running max is enough.
* "Jump at most nums[i] steps" (Jump Game I/II): the jump range varies per index *forward*, and a greedy farthest-reach (Ch 2 §6) beats any DP.

---

## 9. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Strict vs non-strict compatibility (`end ≤ start` vs `end < start`) | Decide from the statement; it picks first-greater vs first-greater-or-equal |
| Pushing i into the deque before reading the max | Expire, read, then push |
| Storing **values** in the deque | Store **indices**; you need them to expire the front |
| Off-by-one in prefix arrays | `prefix[x]` = the sum of the first x entries; the range `[lo, hi]` is `prefix[hi + 1] − prefix[lo]` |
| Clamping a window start below 0 | `Math.max(lo, 0)`, and skip windows with `hi < 0` |
| Sorting job indices with `int[]` and a comparator | Use `Integer[]` (Ch 4), or sort `int[][]` rows |

---

## 10. Practice

1. **[Two Best Non-Overlapping Events](https://leetcode.com/problems/two-best-non-overlapping-events/)** *(Medium)*
2. **[Maximum Number of Events That Can Be Attended II](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended-ii/)** *(Hard)*
3. **[Maximum Sum of 3 Non-Overlapping Subarrays](https://leetcode.com/problems/maximum-sum-of-3-non-overlapping-subarrays/)** *(Hard)*
4. **[Number of Ways to Form a Target String Given a Dictionary](https://leetcode.com/problems/number-of-ways-to-form-a-target-string-given-a-dictionary/)** *(Hard)*
5. **[Max Sum of Rectangle No Larger Than K](https://leetcode.com/problems/max-sum-of-rectangle-no-larger-than-k/)** *(Hard)*

**Stretch:** **[Delivering Boxes from Storage to Ports](https://leetcode.com/problems/delivering-boxes-from-storage-to-ports/)** *(Hard)* · **[Number of Music Playlists](https://leetcode.com/problems/number-of-music-playlists/)** *(Hard)*

<details><summary>Nudges</summary>

1. Two Best Events: at most two events, and the end time is inclusive. Sort by start time, build a **suffix max** of values, and for each event binary search for the first event starting after its end.
2. Events II: F1 with a count. `best[i][c]` = the most value using the first i events (by end) and at most c of them. n·k ≤ 10⁶, so the extra dimension is affordable. End times are inclusive.
3. 3 Subarrays: compute every window sum of length k (Ch 8). Then `bestLeft[i]` = the index of the best window starting at or before i, and `bestRight[i]` the same from the right. Try every middle window. Ties must pick the lexicographically smallest indices, so compare with `>` on the left and `>=` on the right.
4. Target String: all words have the same length, so only the **column** matters. Precount `cnt[col][letter]`. `ways[i][col]` = the ways to form the first i characters of target using columns before col. Mod 10⁹ + 7, with `long`.
5. Rectangle ≤ K: fix the top and bottom rows, and compress the band into one array of column sums. Now find the largest subarray sum ≤ k: for each prefix sum P, you want the smallest earlier prefix ≥ P − k, which is a `TreeSet.ceiling` (Ch 15). Loop over the smaller dimension.
6. Stretch, Delivering Boxes: `trips[i]` = the minimum trips to deliver the first i boxes. The last trip carries boxes j..i−1, limited by count and weight (a window whose left edge only moves right). Its cost is 2 + (the number of port changes inside it), which is a prefix-sum difference. Minimise `trips[j] − changes[j]` over the window with a deque.
7. Stretch, Music Playlists: `ways[i][j]` = playlists of length i using exactly j distinct songs. The i-th song is either new (n − (j − 1) choices) or a replay of one of the j songs not played in the last k slots (max(j − k, 0) choices).
</details>

<details><summary>Key</summary>

Two Best Events → sort + suffix max + binary search · Events II → F1 with k · 3 Subarrays → best-left / best-right prefix arrays + reconstruction · Target String → precount letters per column, then DP · Rectangle ≤ K → fix the row pair + TreeSet of prefix sums · Delivering Boxes → F2 with prefix costs · Music Playlists → counting DP with a reuse constraint
</details>

---

## 11. Self-test

<details><summary>1. In Job Scheduling, why is the largest compatible predecessor j always the best one?</summary>

`best` is non-decreasing in the number of jobs available, and every j′ < j is also compatible, with `best[j′] ≤ best[j]`.
</details>

<details><summary>2. Why can an index that is dominated in the deque be thrown away for good?</summary>

A later index with a score at least as large stays in the window at least as long, so the dominated index can never be the window's max again.
</details>

<details><summary>3. In Jump Game VII, why is the prefix count for the window already available when you reach j?</summary>

The window ends at j − minJump, and minJump ≥ 1, so every index in it is less than j and already decided.
</details>

<details><summary>4. Why does inserting i into a permutation of 1..i−1 add exactly p inverse pairs, where p is the number of elements after it?</summary>

i is larger than everything, so it forms a pair with each element after it and with none before it. The other pairs are unchanged.
</details>

<details><summary>5. A DP's transition is "max over earlier states whose value is smaller than mine". Which tool, and why doesn't the deque work?</summary>

A Fenwick or segment tree indexed by value (Ch 34). The qualifying states aren't a window in index order, so there's no "leaves later" order for a deque to exploit.
</details>

---

## 12. Exit gate

Move on when you can:

* write Job Scheduling, Jump Game VI and K Inverse Pairs from blank files, starting from the slow DP each time;
* name the shape of a transition's range and the tool that matches it, using the §6 table;
* get the strict/non-strict boundary right for both kinds of interval statement;
* stress-test a fast DP against its slow version;
* solve 4 of the 5 practice problems.
