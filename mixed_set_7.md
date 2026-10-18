# Mixed Set VII — after Chapter 37

> **Week 33** · 22 problems, unlabeled, Hard-heavy, drawn from Ch 1–37 · **Time:** about 14 hours

The last Mixed Set, and the closest to a hard OA or a FAANG final round. Same rules as before, with one addition: **before writing any code, write one line naming the bottleneck of the brute force** (Ch 1). Most of these problems are a known technique plus one twist, and the twist is almost always found by asking what the brute force does repeatedly.

Perform mode: Medium 25 minutes, Hard 45 minutes. **The target is ≥ 60% solved and ≥ 75% chosen correctly.** The bar for "chosen correctly" is higher than for "solved" on purpose: on unseen Hards, choosing the right approach is the skill that transfers to interviews, and it's graded even when the code runs out of time.

**Next:** Ch 39 and the mock OAs (Week 34). The exit on 20 Jun is measured against the benchmarks in curriculum §1, on problems you have never seen; this set is the last full rehearsal before them.

---

## The set

1. [Smallest Range II](https://leetcode.com/problems/smallest-range-ii/)
2. [Making File Names Unique](https://leetcode.com/problems/making-file-names-unique/)
3. [Swap For Longest Repeated Character Substring](https://leetcode.com/problems/swap-for-longest-repeated-character-substring/)
4. [Sum of Distances](https://leetcode.com/problems/sum-of-distances/)
5. [Find the Kth Smallest Sum of a Matrix With Sorted Rows](https://leetcode.com/problems/find-the-kth-smallest-sum-of-a-matrix-with-sorted-rows/)
6. [Maximum Number of Robots Within Budget](https://leetcode.com/problems/maximum-number-of-robots-within-budget/)
7. [Design Graph With Shortest Path Calculator](https://leetcode.com/problems/design-graph-with-shortest-path-calculator/)
8. [Number of Atoms](https://leetcode.com/problems/number-of-atoms/)
9. [Count of Range Sum](https://leetcode.com/problems/count-of-range-sum/)
10. [Number of Pairs Satisfying Inequality](https://leetcode.com/problems/number-of-pairs-satisfying-inequality/)
11. [Kth Smallest Product of Two Sorted Arrays](https://leetcode.com/problems/kth-smallest-product-of-two-sorted-arrays/)
12. [Number of Increasing Paths in a Grid](https://leetcode.com/problems/number-of-increasing-paths-in-a-grid/)
13. [Shortest Distance After Road Addition Queries II](https://leetcode.com/problems/shortest-distance-after-road-addition-queries-ii/)
14. [Brace Expansion II](https://leetcode.com/problems/brace-expansion-ii/)
15. [Split Array With Same Average](https://leetcode.com/problems/split-array-with-same-average/)
16. [Count Pairs Of Nodes](https://leetcode.com/problems/count-pairs-of-nodes/)
17. [Count Increasing Quadruplets](https://leetcode.com/problems/count-increasing-quadruplets/)
18. [Longest Substring of One Repeating Character](https://leetcode.com/problems/longest-substring-of-one-repeating-character/)
19. [Find Building Where Alice and Bob Can Meet](https://leetcode.com/problems/find-building-where-alice-and-bob-can-meet/)
20. [Minimum Number of Visited Cells in a Grid](https://leetcode.com/problems/minimum-number-of-visited-cells-in-a-grid/)
21. [Maximum Sum of Subsequence With Non-adjacent Elements](https://leetcode.com/problems/maximum-sum-of-subsequence-with-non-adjacent-elements/)
22. [Count the Number of Good Partitions](https://leetcode.com/problems/count-the-number-of-good-partitions/)

---

<details><summary>Key: open only after attempting</summary>

| # | Technique | Chapter | The insight in one line |
| :-: | :--- | :-: | :--- |
| 1 | Sort + split point (exchange proof) | 9, 2 | After sorting, some prefix gets +k and the rest −k; try every split, comparing only the four end values |
| 2 | Hash map with the next suffix to try | 7 | Remember, per base name, the smallest k not yet tried, so the suffix search never restarts from 1 |
| 3 | Run lengths + letter counts | 5, 11 | Extend a run by one if the letter occurs elsewhere; join two runs split by one character, capped by the letter's total count |
| 4 | Group by value + prefix sums of indices | 7, 8 | For index i in its group: (left count · i − left sum) + (right sum − right count · i), in `long` |
| 5 | Merge row by row, keeping the k smallest | 15 | Combining the k smallest sums so far with the next row keeps only k candidates (m ≤ 40, k ≤ 200); or binary search on the answer + a counting DFS |
| 6 | Sliding window + monotonic deque | 11, 12 | Cost = window max + size · window sum; the deque gives the max, so shrink from the left while over budget |
| 7 | Dijkstra per query (design) | 23, 16 | n ≤ 100 with few calls: rerun Dijkstra; or keep Floyd–Warshall and update it in O(n²) per new edge |
| 8 | Stack of count maps + parsing | 12, 5 | `(` pushes a new map, `)k` multiplies the top map by k and merges it down; a TreeMap gives sorted output |
| 9 | Prefix sums + merge-sort counting or Fenwick | 8, 9, 34 | Count pairs i < j with lower ≤ P[j] − P[i] ≤ upper; prefix sums need `long` |
| 10 | Rewrite the inequality + Fenwick (or merge sort) | 34, 9 | With d = nums1 − nums2, it's d[i] ≤ d[j] + diff: count earlier values ≤ a threshold |
| 11 | Binary search on the answer + signed counting | 10 | Count the pairs with product ≤ x per element of one array; a negative factor reverses the order of the other array |
| 12 | DAG DP by value order | 22, 31 | Paths only go uphill, so it's a DAG: each cell's count is 1 + the sum over its smaller neighbours (memoised DFS), mod 10⁹ + 7 |
| 13 | Ordered set of surviving nodes | 15 | The roads never cross, so a road i → j deletes every node strictly between them; the answer is the set's size − 1 |
| 14 | Recursive-descent parsing with set algebra | 6, 12 | A comma means union; juxtaposition means the product of sets; braces recurse |
| 15 | Meet in the middle on (count, sum), or DP over count | 37, 28 | A subset of size k with sum k · total / n must exist; shift each value by the mean so the target sum is 0 |
| 16 | Sorted degrees + two pointers − shared-edge correction | 11, 7 | Count the pairs with deg[a] + deg[b] > q, then subtract the ones where removing their shared edges drops them to ≤ q |
| 17 | Prefix count tables, O(n²) | 8 | Fix (j, k) with nums[k] < nums[j]: (i < j with nums[i] < nums[k]) × (l > k with nums[l] > nums[j]) |
| 18 | Segment tree with prefix / suffix runs | 34 | Each node keeps its best run, its prefix run and its suffix run; merge by joining the middle runs when the boundary characters match |
| 19 | Offline + monotonic stack + binary search | 12, 10, 34 | If neither can reach the other directly, the answer is the leftmost index after max(a, b) taller than both; answer the queries sorted by position |
| 20 | BFS with "next unvisited" sets | 21, 15, 24 | Per-row and per-column ordered sets (or DSU skip pointers) of unvisited cells, so each cell enters the BFS once |
| 21 | Segment tree with 2 × 2 DP nodes | 34, 27 | House Robber per node, keyed by whether its first and last elements are taken; point updates re-merge one path |
| 22 | Interval merging → number of blocks, then a power of 2 | 14, 18 | Each value's first-to-last span must lie in one block; with m merged blocks the answer is 2^(m − 1) mod 10⁹ + 7 |
</details>
