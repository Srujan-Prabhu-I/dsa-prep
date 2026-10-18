# Chapter 37 — Synthesis Techniques & the Out-of-Scope Guide

> **Part VII · Week 33** · **S-tier** · Prerequisites: Ch 3 (counting per element), Ch 10, Ch 12 §4 (contribution), Ch 17 (subset enumeration), Ch 24 (DSU, offline processing)
> **Problems:** 4 worked (1 ★) · 5 practice · 2 stretch · **Time:** about 8 hours

This chapter has two halves.

**Part A, four synthesis techniques.** These aren't data structures. They are ways of **changing the question** until it becomes one you already know how to answer:
* **Contribution:** a sum over all substructures becomes a sum over elements.
* **Meet in the middle:** 2ⁿ becomes 2 · 2^(n/2).
* **Reverse time and offline processing:** deletions become insertions; a hard query order becomes a convenient one.
* **"The answer is small" and √n reasoning:** a bound that looks quadratic is really much smaller.

**Part B, the out-of-scope guide.** Earlier chapters signposted a set of competitive-programming tools: flows, heavy-light decomposition, suffix structures, the convex hull trick and others. Part B gives each one **a recognition brief**: what it solves, the signal, and what to do if you meet it in an OA or interview. It doesn't teach them. That's a deliberate scope decision (the curriculum's scope note): their frequency in interviews is too low to justify the hours.

---

## 1. Attempt first ★

> **[Total Appeal of A String](https://leetcode.com/problems/total-appeal-of-a-string/) (30 minutes).** The appeal of a string is its number of **distinct** characters. Return the sum of the appeals of **all** substrings of s, with n ≤ 10⁵. There are about 5·10⁹ substrings, so you can't list them.

---

# Part A — Synthesis techniques

## 2. Contribution: count per element, not per structure

Ch 3 counted pairs of nested subsets "per element" (3ⁿ). Ch 12 §4 summed subarray minimums by asking each element how many subarrays it is the minimum of. **The general move:**

> Σ over structures S of f(S) = Σ over elements x of (x's share of f) × (the number of structures in which x has that share).

The second count is usually a product of "how many ways to choose the left end" × "how many ways to choose the right end".

**The one rule that keeps it correct: make each unit of f belong to exactly one element.** If a substring has 3 distinct letters, some 3 positions must be "responsible" for them, with no double counting. Choose a canonical owner: the **first** occurrence of each letter, or the **leftmost** minimum (Ch 12's strict / non-strict tie-break).

### Worked Y1a · [Total Appeal of A String](https://leetcode.com/problems/total-appeal-of-a-string/) ★

**The owner.** In a substring, let each distinct letter be owned by its **first** occurrence. Position i (letter c) owns c in the substrings `[l, r]` with:
* `l` in `(last[c], i]`, where `last[c]` is the previous position of c (or −1), so that no earlier c is inside: `i − last[c]` choices;
* `r` in `[i, n − 1]`: `n − i` choices.

```java
public long appealSum(String s) {
    int n = s.length();
    int[] last = new int[26];
    Arrays.fill(last, -1);
    long total = 0;
    for (int i = 0; i < n; i++) {
        int c = s.charAt(i) - 'a';
        total += (long) (i - last[c]) * (n - i);     // substrings in which i is c's first occurrence
        last[c] = i;
    }
    return total;
}
```

**O(n).** The `(long)` cast matters: the product reaches 2.5·10⁹. **The slow version** (for each start, extend the end and keep a count of distinct letters) is O(n²) and is the stress-test oracle.

### Worked Y1b · [Count Unique Characters of All Substrings of a Given String](https://leetcode.com/problems/count-unique-characters-of-all-substrings-of-a-given-string/)

> Sum, over all substrings, the number of characters that appear **exactly once** in it.

Now position i counts in a substring only if its letter appears nowhere else in it. So `l` is in `(prev[i], i]` and `r` in `[i, next[i])`, where prev and next are the neighbouring occurrences of the same letter:

```java
public int uniqueLetterString(String s) {
    int n = s.length();
    int[] prev = new int[n], next = new int[n], seen = new int[26];
    Arrays.fill(seen, -1);
    for (int i = 0; i < n; i++) { int c = s.charAt(i) - 'A'; prev[i] = seen[c]; seen[c] = i; }
    Arrays.fill(seen, n);
    for (int i = n - 1; i >= 0; i--) { int c = s.charAt(i) - 'A'; next[i] = seen[c]; seen[c] = i; }
    long total = 0;
    for (int i = 0; i < n; i++) total += (long) (i - prev[i]) * (next[i] - i);
    return (int) total;
}
```

**Compare the two.** Y1a's owner is "the first occurrence", so only the *left* boundary is constrained. Y1b's condition is "the only occurrence", so both sides are. **Changing the definition of ownership changes the problem**, and writing the owner down in words is the step that prevents double counting. Sum of Total Strength of Wizards (practice) combines this with Ch 12's monotonic stack.

---

## 3. Meet in the middle

**The shape.** A brute force over all 2ⁿ subsets with n ≈ 40 (2⁴⁰ ≈ 10¹²) is hopeless, and there's no DP state smaller than the subset itself (the values are huge, so knapsack over sums doesn't fit). But if the answer combines **two independent halves** (sum of the left part + sum of the right part), you can:
1. enumerate all 2^(n/2) choices of each half (about 10⁶ each);
2. sort one side;
3. for each choice on the other side, **binary search** (Ch 10) or two-pointer (Ch 11) for its best partner.

That's O(2^(n/2) · n) instead of O(2ⁿ).

### Worked Y2 · [Closest Subsequence Sum](https://leetcode.com/problems/closest-subsequence-sum/)

> n ≤ 40 integers (|value| ≤ 10⁷) and a goal (|goal| ≤ 10⁹). Choose a subsequence (possibly empty) whose sum is as close to the goal as possible; return the minimum |sum − goal|.

Split the array into two halves. A subsequence is a subset of the left half plus a subset of the right half, chosen independently. For a left sum a, the best right sum is the one closest to `goal − a`: the first right sum ≥ it, or the one just before.

```java
public int minAbsDifference(int[] nums, int goal) {
    int n = nums.length, half = n / 2;
    int[] left = subsetSums(nums, 0, half), right = subsetSums(nums, half, n);
    Arrays.sort(right);
    long best = Long.MAX_VALUE;
    for (int a : left) {
        long need = (long) goal - a;
        int lo = -1, hi = right.length;                 // Ch 10: hi ends as the first right sum >= need
        while (hi - lo > 1) {
            int mid = lo + (hi - lo) / 2;
            if (right[mid] >= need) hi = mid; else lo = mid;
        }
        if (hi < right.length) best = Math.min(best, right[hi] - need);
        if (hi > 0) best = Math.min(best, need - right[hi - 1]);
    }
    return (int) best;
}

// All 2^(to - from) subset sums of a[from, to), each built from a smaller mask (Ch 17).
private int[] subsetSums(int[] a, int from, int to) {
    int k = to - from;
    int[] sums = new int[1 << k];
    for (int mask = 1; mask < (1 << k); mask++)
        sums[mask] = sums[mask & (mask - 1)] + a[from + Integer.numberOfTrailingZeros(mask)];
    return sums;
}
```

**O(2^(n/2) · n):** about 2·10⁷ for the sort and the searches together. **Ranges:** each half's sums are at most 20 · 10⁷ = 2·10⁸ in absolute value, so they fit in `int`; but `goal − a` can reach 1.2·10⁹ and the differences more, so they're `long`. **Two pointers instead of binary search, in prose:** sort both sides, then walk one ascending and the other descending, as in Two Sum on sorted arrays (Ch 11).

**Recognition:** n between about 30 and 45, an exponential search, and an objective that **splits into two independent halves**. When the halves must agree on something (both contain exactly n/2 elements), group each side's sums by that count first (Partition Array Into Two Arrays, practice).

---

## 4. Reverse time, and offline processing in general

**Some operations are easy in one direction only.** A DSU (Ch 24) can merge groups but never split them. So when a problem **removes** things over time and asks about connectivity after each removal, **play it backwards**: start from the final state, and *add* the removed things back in reverse order. Each answer is read off at the right moment and stored at its original index.

### Worked Y3 · [Bricks Falling When Hit](https://leetcode.com/problems/bricks-falling-when-hit/)

> A grid of bricks (1) and empty cells (0). A brick is stable if it's in the top row, or 4-connected to a stable brick; every brick starts stable. Each hit erases one cell. After a hit, every brick that is no longer stable falls (and is erased). Return how many bricks fall after each hit. Up to 4·10⁴ hits on a 200 × 200 grid.

**Forwards, it's hard:** each hit may split a component, and DSU can't split. **Backwards:**
1. Apply **all** the hits first. What's left is the grid after the last hit, *before* counting falls. Build a DSU over it, with an extra **roof** node joined to every top-row brick.
2. Undo the hits from last to first. Restoring a brick joins it to its neighbours (and to the roof if it's in the top row). The bricks that become connected to the roof **because of this brick** are exactly the bricks that fell when it was hit, forwards. So the answer is the roof component's growth, minus 1 for the restored brick itself.

```java
public int[] hitBricks(int[][] grid, int[][] hits) {
    int rows = grid.length, cols = grid[0].length, roof = rows * cols;
    int[][] g = new int[rows][];
    for (int r = 0; r < rows; r++) g[r] = grid[r].clone();
    for (int[] h : hits) g[h[0]][h[1]]--;                   // brick 1 -> 0; empty 0 -> -1 (never a brick)

    DSU dsu = new DSU(rows * cols + 1);                      // Ch 24, with sizeOf
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            if (g[r][c] == 1) attach(g, dsu, r, c);

    int[] answer = new int[hits.length];
    for (int k = hits.length - 1; k >= 0; k--) {
        int r = hits[k][0], c = hits[k][1];
        if (++g[r][c] != 1) continue;                       // the hit was on an empty cell: nothing fell
        int before = dsu.sizeOf(roof);
        attach(g, dsu, r, c);
        answer[k] = Math.max(0, dsu.sizeOf(roof) - before - 1);
    }
    return answer;
}

private static final int[][] DIRS = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

private void attach(int[][] g, DSU dsu, int r, int c) {    // join brick (r, c) to its neighbours (and the roof)
    int cols = g[0].length, id = r * cols + c;
    if (r == 0) dsu.union(id, g.length * cols);
    for (int[] d : DIRS) {
        int nr = r + d[0], nc = c + d[1];
        if (nr >= 0 && nr < g.length && nc >= 0 && nc < cols && g[nr][nc] == 1) dsu.union(id, nr * cols + nc);
    }
}
```

**Why `max(0, …)`:** a restored brick that doesn't touch the roof component adds nothing to it. Forwards, the hit only erased that one brick; nothing *fell*. **Why decrementing handles hits on empty cells:** a cell that was 0 becomes −1 and goes back to 0 when undone, so it never counts as a brick. O((cells + hits) · α).

**Offline processing, the general idea** (Ch 24, Ch 34): when all queries are known in advance, answer them in whatever order is **convenient**, then write each answer back to its original index. The convenient orders that recur:
* **Reverse time:** deletions become insertions (Y3).
* **Sorted by a threshold:** "using only edges of weight ≤ limit" becomes a sweep that adds edges as the limit grows (Ch 24).
* **Sorted by right endpoint:** range queries become prefix queries over a sweep (Ch 34, Distinct Values).
* **Sorted by value**, with a monotonic stack or segment tree over the other coordinate (Maximum Sum Queries, stretch).

---

## 5. "The answer is small", and √n reasoning

These aren't algorithms. They are **bounds you prove** to show that an apparently slow loop is fast.

* **Few distinct values.** The bitwise OR of the subarrays ending at i takes at most about 30 distinct values (each new value adds at least one bit, Ch 17). The GCD of the subarrays ending at i takes O(log V) distinct values (each change at least halves it). So "for each end, keep the set of distinct values" is O(n log V), not O(n²).
* **Harmonic sums.** Looping over the multiples of every d ≤ n costs n/1 + n/2 + … + n/n = O(n log n) (the harmonic sum of Ch 3 §2.1, used for the sieves of Ch 18).
* **√ splits.** If item sizes sum to S, at most about √(2S) of the sizes are distinct, since 1 + 2 + … + k ≤ S. And with "heavy" items (more than √n occurrences) and "light" ones, there are at most √n heavy items, while the light ones are each cheap. One such split often turns O(n²) into O(n√n).
* **The answer is bounded by the input.** For example, the k-th smallest of something is at most k steps from a known start, or the number of operations is at most log n. When the answer is provably small, iterate over the **answer** instead of over the input.
* **Few reachable states.** Sometimes the state space looks huge, but an exchange argument shows that only a few states are worth visiting. *Example:* the fewest operations to bring x ≤ 10¹⁸ down to 1, where each operation subtracts 1, halves an even x, or divides a multiple of 3 by 3. Memoising f(x − 1) would visit up to 10¹⁸ values. But **subtracting 1 more than x mod 2 times and then halving is never better than halving first** (two subtractions before a halving equal one after it). So the only useful moves are "drop to the nearest multiple of 2, then halve" and "drop to the nearest multiple of 3, then divide":

  ```java
  Map<Long, Long> memo = new HashMap<>();
  long fewest(long x) {                       // the fewest operations to reach 1
      if (x <= 1) return 0;
      if (x <= 3) return 1;                   // 2 -> 1 and 3 -> 1 each take one step
      Long known = memo.get(x);
      if (known != null) return known;
      long best = Math.min(x % 2 + 1 + fewest(x / 2), x % 3 + 1 + fewest(x / 3));
      memo.put(x, best);
      return best;
  }
  ```

  Every value reached is ⌊x / (2ᵃ 3ᵇ)⌋ for some a, b ≤ log₂ x, so there are only O(log² x) states, about 1 000 for x = 10¹⁸, and the recursion is only about 60 calls deep. Prove which states suffice, then memoise only those.

**When you're stuck on a Hard problem, ask:** "Is there a quantity here that is secretly small?" Then prove it before relying on it.

---

# Part B — The out-of-scope guide

**How to use this guide.** If an OA problem matches one of these signals, first check the **constraints**: they often allow the in-scope fallback listed. If not, and it's an OA, take the partial credit (Ch 39) and move on. In a live interview, naming the tool and explaining the brute force plus the bottleneck is usually what's being assessed, since interviewers rarely expect these to be implemented.

| Tool | What it solves | The signal | In an OA / interview |
| :--- | :--- | :--- | :--- |
| **Max flow / min cut** | The most "stuff" through a capacitated network; the cheapest set of edges whose removal disconnects s from t | "Assign workers to jobs with capacities", "the minimum number of edges/vertices to cut so that…" | For **bipartite matching** with n ≤ ~500, Kuhn's augmenting-path DFS (~15 lines) is the one flow algorithm worth learning next. Small n: bitmask DP (Ch 32) |
| **Heavy-light decomposition** | Updates and queries on arbitrary **tree paths** | Path max/sum **with** updates, n, q ~ 10⁵ | Static: binary lifting (Ch 36). Subtree-only: Euler tour (Ch 36) |
| **Centroid decomposition** | Counting or optimising over **all pairs of nodes** in a tree | "The number of pairs at distance exactly k", n ~ 10⁵ | Small k: tree DP with depth counts (Ch 31). Small n: BFS from every node |
| **Suffix array / suffix automaton** | Order and counts of all substrings: the k-th distinct substring, the number of distinct substrings, the longest common substring of many strings | "Distinct substrings", "lexicographic k-th substring", n ~ 10⁵ | Hashing + binary search (Ch 35) answers most equality-only versions |
| **Aho–Corasick** | Many patterns searched in a text at once | Thousands of patterns, a long text | Short patterns: a trie walked from each start (Ch 20) |
| **Manacher** | All palindromic substrings in O(n) | n ~ 10⁶ palindromes | n ≤ ~5·10³: expand around centres (Ch 29). Larger: hashing + binary search on each centre's radius (Ch 35) |
| **FFT / NTT** | Convolution: the counts of every pairwise sum a + b, polynomial multiplication | "For every k, the number of pairs with a[i] + b[j] = k", values ~ 10⁵ | Values small: O(V²) counting. Otherwise rarely appears in interviews |
| **Convex hull trick / Li Chao tree** | DP transitions of the form `min over j of (dp[j] + b[j] · a[i])` | A DP whose transition is a linear function of i with slopes from j (Ch 33 §6) | n ≤ ~5·10³: the plain O(n²) DP |
| **Knuth / divide-and-conquer DP optimisation** | Interval or partition DPs whose best split point moves monotonically | O(n³) interval DP or O(k·n²) partition DP at n ~ 5·10³ | The plain DP (Ch 29–30) at the sizes interviews use |
| **Mo's algorithm** | Offline range queries where adding/removing one element updates the answer in O(1) | "Distinct count / mode in [l, r]", n, q ~ 10⁵, no updates | Sort by right end + Fenwick (Ch 34) covers "distinct count"; the mode is a true Mo's case |
| **Persistent / 2D segment trees, merge-sort trees** | Queries on past versions; rectangle queries; "k-th smallest in a range" | "Version", "k-th smallest in a[l..r]" online | Offline sweeps (Ch 34) or 2D prefix sums (static) |
| **Link-cut trees / dynamic connectivity** | Trees or graphs that gain **and** lose edges online | Online edge deletions | Offline: reverse time (§4) |
| **Sprague–Grundy / Nim** | Sums of independent games | Many piles, take-away moves | Recognition fact: in Nim, the first player wins iff the XOR of the pile sizes is non-zero. Single game: DP (Ch 30) |
| **Matrix exponentiation** | A linear recurrence (Fibonacci-like, or counting paths of exactly n steps) evaluated at n up to 10¹⁸ | "the n-th term mod p" with n ≈ 10¹⁸ and a small, fixed set of states | Write the recurrence as a k × k matrix and use Ch 18's fast power on matrices, O(k³ log n). If n ≤ 10⁷, the plain O(n · k) DP is enough |
| **Morris traversal** | Inorder traversal of a binary tree in O(1) extra space, by temporarily rewiring null right pointers | "inorder without recursion or a stack" as a follow-up | The stack-based iterative inorder (Ch 19) is the expected answer; name Morris as the O(1)-space option |
| **Computational geometry** (convex hull, segment intersection, sweeps on angles) | Points and shapes | "Smallest fence around all points", "do these segments cross" | Rare in interviews; for grid/axis-aligned shapes, intervals and sweeps (Ch 14) |

**A note for backend loops: concurrency questions.** Some backend interviews include a coding question about threads rather than algorithms: a bounded blocking queue, "print in order" across threads, a thread-safe rate limiter or cache. That's not DSA, so this handbook doesn't teach it, but your backend preparation should. Know `synchronized`, `wait`/`notify`, `ReentrantLock` with `Condition`, `Semaphore`, `ConcurrentHashMap` and `BlockingQueue`, and be able to explain a race condition and a deadlock. Ch 16's rate-limiter follow-ups show how a DSA design question turns into this discussion.

**What "out of scope" doesn't mean.** It doesn't mean "never study it". After this handbook, if your target company's OA has a history of a topic (contest-style OAs sometimes do), pick it up then. Each has standard references, and the prerequisites in this handbook are sufficient.

---

## 6. Recognition

**Signals:**
* "Sum over all substrings / subarrays / pairs / subsets of …": contribution (§2). Ask "which element owns each unit?"
* n ≈ 30–45 with an exponential search whose objective splits in two: meet in the middle (§3).
* Deletions, with a structure that only supports insertions: reverse time (§4).
* All queries are given up front: consider offline orders (§4).
* A loop that looks quadratic over a quantity that is secretly small: §5.

**False friends:**
* n ≤ 20: plain subset enumeration or bitmask DP (Ch 32), not meet in the middle.
* n ≈ 40 but the values are small: a knapsack DP over sums (Ch 28) is exact and simpler.
* "Sum over subarrays" with an easy incremental update: an O(n) sweep (Ch 8) might do it without any contribution argument.

---

## 7. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Two elements both claiming one unit of f (ties) | Fix the owner: first occurrence, or a strict left / non-strict right boundary |
| `int` products in contribution sums | Cast one factor to `long` before multiplying |
| Meet in the middle with `List<Integer>` of 2²⁰ sums | Use `int[]` (boxing costs ~16 bytes and a pointer each; Ch 4) |
| Reverse time: forgetting cells that were never there | Decrement-on-apply, increment-on-undo, and act only on transitions to "present" |
| Offline answers written in the sorted order | Carry the original index and write `answer[idx]` |

---

## 8. Practice

1. **[Shortest Subarray to be Removed to Make Array Sorted](https://leetcode.com/problems/shortest-subarray-to-be-removed-to-make-array-sorted/)** *(Medium)*
2. **[CSES Meet in the Middle](https://cses.fi/problemset/task/1628)** *(Medium; stdin/stdout)*
3. **[Count Subarrays With Median K](https://leetcode.com/problems/count-subarrays-with-median-k/)** *(Hard)*
4. **[Partition Array Into Two Arrays to Minimize Sum Difference](https://leetcode.com/problems/partition-array-into-two-arrays-to-minimize-sum-difference/)** *(Hard)*
5. **[Sum of Total Strength of Wizards](https://leetcode.com/problems/sum-of-total-strength-of-wizards/)** *(Hard)*

**Stretch:** **[Sum of Imbalance Numbers of All Subarrays](https://leetcode.com/problems/sum-of-imbalance-numbers-of-all-subarrays/)** *(Hard)* · **[Maximum Sum Queries](https://leetcode.com/problems/maximum-sum-queries/)** *(Hard)*

<details><summary>Nudges</summary>

1. Remove to Sort: the kept part is a sorted prefix `[0..i]` plus a sorted suffix `[j..n−1]`. Find the longest of each; then, for each prefix end, advance a pointer in the suffix to the first value ≥ it (two pointers), removing what lies between.
2. Meet in the Middle: count the subsets with sum exactly x, n ≤ 40. Enumerate both halves, sort them, and count matching pairs with two pointers, handling runs of equal sums. The count needs `long`, and so do the sums.
3. Median K: the values are distinct, 1..n. A subarray containing k has median k iff (count greater than k) − (count smaller than k) is 0 or 1. Record the balance of every left extension from k's index in a map; for each right extension with balance b, add count[−b] + count[1 − b].
4. Two Arrays Min Diff: each half has n of the 2n numbers. For the left half, group subset sums by how many elements they use (c), and for the right half by n − c. Sort each group, and binary search for the partner that brings the chosen sum closest to total / 2.
5. Wizards: Σ over subarrays of (min × sum). Each element is the min of subarrays with l in (L, i] and r in [i, R), with L and R from Ch 12's monotonic stack (strict on one side). Summing the subarray sums over those choices needs **prefix sums of prefix sums**: with P the prefix sums, it's (i − L) · Σ_{r=i..R−1} P[r + 1] − (R − i) · Σ_{l=L+1..i} P[l]. Work mod 10⁹ + 7.
6. Stretch, Imbalance: n ≤ 1000, so fix the start and extend the end, keeping a `seen[]` array. Adding a new value x changes the imbalance by +1, then −1 if x − 1 is present and −1 if x + 1 is present. A repeated value changes nothing. That's O(n²).
7. Stretch, Max Sum Queries: offline. Sort the points by nums1 descending and the queries by x descending. Add points as x falls. Then each query needs the max of nums1 + nums2 over the added points with nums2 ≥ y: a segment tree for max on compressed nums2 (Ch 34). A Fenwick tree over reversed ranks also works here, but only because each position's stored value can only increase (the misconception table in Ch 34 §2).
</details>

<details><summary>Key</summary>

Remove to Sort → keep a prefix and a suffix, two pointers · Meet in the Middle → Y2 with sorted halves · Median K → balance prefix + hashing around k's index · Two Arrays Min Diff → Y2 grouped by count · Wizards → Y1 with a monotonic stack + prefix of prefix sums · Imbalance → Y1 per element · Max Sum Queries → offline sort + monotonic stack / segment tree
</details>

---

## 9. Self-test

<details><summary>1. In Total Appeal, why does position i count (i − last[c]) · (n − i) substrings?</summary>

i owns letter c in the substrings where it's c's first occurrence: the start must be after the previous c and at most i, and the end can be anything from i on.
</details>

<details><summary>2. How do Total Appeal and Count Unique Characters differ in their owner definitions?</summary>

Appeal: the first occurrence owns the letter, so only the left boundary is limited. Unique: the letter counts only if it's the only occurrence, so both boundaries are limited by the neighbouring occurrences.
</details>

<details><summary>3. When does meet in the middle apply, and what does it cost?</summary>

When the objective combines two independent halves (a sum, say). Enumerate 2^(n/2) choices per half, sort one side and binary search from the other: O(2^(n/2) · n).
</details>

<details><summary>4. In Bricks Falling, why is the answer the roof component's growth minus one?</summary>

Restoring the brick reconnects exactly the bricks that, forwards, lost their support when it was hit. The −1 removes the restored brick itself, and it's 0 if the brick doesn't connect to the roof.
</details>

<details><summary>5. An OA problem asks for the number of distinct substrings of a string of length 10⁵. What do you do?</summary>

Recognise a suffix-structure problem (out of scope). Check whether the constraints allow hashing all substrings of a few lengths, or an O(n²) method for partial credit, and budget time accordingly (Ch 39).
</details>

---

## 10. Exit gate

Move on when you can:

* state the owner of each unit before writing any contribution formula, and solve Y1a and Y1b from blank files;
* write meet in the middle with the subset-sum doubling trick;
* explain and write the reverse-time DSU for Bricks Falling;
* name the in-scope fallback for each tool in the Part B table;
* solve 3 of the 5 practice problems.
