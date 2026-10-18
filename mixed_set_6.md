# Mixed Set VI — after Chapter 33

> **Week 30** · 30 problems, unlabeled, drawn from Ch 1–33 · **Time:** about 14 hours

Same rules as before. This set leans on DP, but **not every problem in it is DP**: deciding that a problem is greedy, sliding-window or plain counting is part of the test. For any DP you choose, the first thing you write is **the state in English** (Ch 27 step 1), then the transition, before any code. If the transition loops over many earlier states, look up its shape in Ch 33 §6. Perform mode: Medium 25 minutes, Hard 45 minutes. The target is ≥ 70% solved and ≥ 70% chosen correctly.

**After the set:** log every miss with its failure tag. IDEA misses on DP almost always mean the state was missing information (Ch 27 §1): write down what the state forgot.

---

## The set

1. [Unique Binary Search Trees](https://leetcode.com/problems/unique-binary-search-trees/)
2. [Maximum Points You Can Obtain from Cards](https://leetcode.com/problems/maximum-points-you-can-obtain-from-cards/)
3. [Longest Arithmetic Subsequence](https://leetcode.com/problems/longest-arithmetic-subsequence/)
4. [Knight Dialer](https://leetcode.com/problems/knight-dialer/)
5. [Minimum Falling Path Sum](https://leetcode.com/problems/minimum-falling-path-sum/)
6. [Best Team With No Conflicts](https://leetcode.com/problems/best-team-with-no-conflicts/)
7. [Maximum Length of Repeated Subarray](https://leetcode.com/problems/maximum-length-of-repeated-subarray/)
8. [Champagne Tower](https://leetcode.com/problems/champagne-tower/)
9. [Number of Ways to Paint N × 3 Grid](https://leetcode.com/problems/number-of-ways-to-paint-n-3-grid/)
10. [Out of Boundary Paths](https://leetcode.com/problems/out-of-boundary-paths/)
11. [Minimum Cost Tree From Leaf Values](https://leetcode.com/problems/minimum-cost-tree-from-leaf-values/)
12. [Maximum Alternating Subsequence Sum](https://leetcode.com/problems/maximum-alternating-subsequence-sum/)
13. [Count Vowels Permutation](https://leetcode.com/problems/count-vowels-permutation/)
14. [Jump Game V](https://leetcode.com/problems/jump-game-v/)
15. [Last Day Where You Can Still Cross](https://leetcode.com/problems/last-day-where-you-can-still-cross/)
16. [Maximum Earnings From Taxi](https://leetcode.com/problems/maximum-earnings-from-taxi/)
17. [Count All Valid Pickup and Delivery Options](https://leetcode.com/problems/count-all-valid-pickup-and-delivery-options/)
18. [Stone Game II](https://leetcode.com/problems/stone-game-ii/)
19. [Frog Jump](https://leetcode.com/problems/frog-jump/)
20. [Painting the Walls](https://leetcode.com/problems/painting-the-walls/)
21. [Count Palindromic Subsequences](https://leetcode.com/problems/count-palindromic-subsequences/)
22. [Arithmetic Slices II – Subsequence](https://leetcode.com/problems/arithmetic-slices-ii-subsequence/)
23. [Tallest Billboard](https://leetcode.com/problems/tallest-billboard/)
24. [Allocate Mailboxes](https://leetcode.com/problems/allocate-mailboxes/)
25. [Paint House III](https://leetcode.com/problems/paint-house-iii/)
26. [Number of Ways to Stay in the Same Place After Some Steps](https://leetcode.com/problems/number-of-ways-to-stay-in-the-same-place-after-some-steps/)
27. [Freedom Trail](https://leetcode.com/problems/freedom-trail/)
28. [Maximum Value of K Coins From Piles](https://leetcode.com/problems/maximum-value-of-k-coins-from-piles/)
29. [Minimum Total Distance Traveled](https://leetcode.com/problems/minimum-total-distance-traveled/)
30. [Minimum Number of Operations to Make Array Continuous](https://leetcode.com/problems/minimum-number-of-operations-to-make-array-continuous/)

---

<details><summary>Key: open only after attempting</summary>

| # | Technique | Chapter | The insight in one line |
| :-: | :--- | :-: | :--- |
| 1 | Split DP (Catalan numbers) | 20, 28 | Choose the root: `count[n] = Σ count[left size] · count[right size]` |
| 2 | Complement fixed window | 11 | Taking k from the ends leaves a window of n − k in the middle; minimise its sum |
| 3 | `len[i][diff]` | 29 | The longest arithmetic run ending at i with difference diff extends the one ending at some j < i; offset diff into an array |
| 4 | DP over keys × moves | 28 | `ways[step][key]` from the keys a knight can jump from; mod 10⁹ + 7 |
| 5 | Grid DP | 28 | Each cell adds the min of the three cells above it |
| 6 | Sort + LIS-style DP | 9, 29 | Sort by (age, score); then the best sum of a non-decreasing score subsequence |
| 7 | Longest common substring | 29 | `run[i][j] = run[i−1][j−1] + 1` when the elements match, else 0 |
| 8 | Row-by-row simulation DP | 28 | A glass with x > 1 passes (x − 1) / 2 to each glass below; cap the answer at 1 |
| 9 | Pattern-state counting | 28 | Rows are ABA-type or ABC-type; ABA → 3 ABA + 2 ABC, ABC → 2 ABA + 2 ABC |
| 10 | DP over moves | 28 | `ways[move][r][c]`; a step that leaves the grid adds to the answer |
| 11 | Interval DP (or monotonic-stack greedy) | 30, 12 | Split [i, j] at k: cost + max(left) · max(right); the stack version removes each local minimum with its smaller neighbour |
| 12 | Two-state DP | 30 | Best sum whose last taken element was added vs subtracted |
| 13 | Automaton counting | 28 | Five states, one per last vowel; each rule is an edge |
| 14 | DP in sorted order (a DAG) | 22, 31 | Jumps only go to lower bars: process bars by height, or memoised DFS |
| 15 | Binary search + BFS, or reverse-time DSU | 10, 21, 24 | Crossability is monotone in the day; or add land backwards until top joins bottom |
| 16 | Weighted interval DP | 33 | Ch 33 F1 with profit end − start + tip; or a DP over positions 1..n grouped by end |
| 17 | Counting recurrence | 18 | Insert the n-th pair into a valid order of n − 1 pairs: `ans[n] = ans[n−1] · n · (2n − 1)` |
| 18 | Game DP with suffix sums | 30 | `best[i][M]` = the most the mover gets from piles i..; it's the suffix sum minus the opponent's best |
| 19 | Memo on (stone, last jump) | 27, 7 | Map each stone position to its index; try jumps of last − 1, last, last + 1 |
| 20 | 0/1 knapsack on "walls covered" | 28 | A paid wall i covers 1 + time[i] walls (the free painter works meanwhile); min cost to cover ≥ n |
| 21 | Fix the middle + prefix/suffix pair counts | 8, 29 | For each middle, count pairs (a, b) before and (b, a) after: 100 digit pairs per side (Hard) |
| 22 | `count[i][diff]` with `long` diffs | 29 | Count subsequences of length ≥ 2 ending at i; adding one more element makes them length ≥ 3, which is when they count |
| 23 | DP on height difference | 28 | `tallest[diff]` = the largest taller side when the sides differ by diff; each rod is skipped, added to the taller, or added to the shorter |
| 24 | Partition DP + median cost | 9, 29 | Sort the houses; one mailbox at the median of each group; `best[k][i]` over the last group |
| 25 | `cost[i][colour][groups]` | 28, 30 | Track the last colour and the neighbourhoods so far; painted houses have a fixed colour |
| 26 | DP over (step, position) with a cap | 28 | You can never usefully go further than steps / 2 from the start, so cap the positions |
| 27 | DP over (key index, ring position) | 29 | For each key character, try every ring position holding it; the rotation cost is the circular distance |
| 28 | Group knapsack | 28 | Each pile is a group: take 0..t coins from its top, with the prefix sum as the value |
| 29 | Sort both + DP over (robot, factory) | 9, 29 | After sorting, each factory repairs a contiguous block of robots (exchange argument) |
| 30 | Sort + dedupe + sliding window | 9, 11 | **Not DP.** Keep the most values that fit in some range [v, v + n − 1]; the answer is n − that count |
</details>
