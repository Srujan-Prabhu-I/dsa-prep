# Mixed Set V — after Chapter 25

> **Week 22** · 25 problems, unlabeled, drawn from Ch 1–25 · **Time:** about 11 hours

Same rules as before. For graph-flavoured problems, the first line you write must be **the model**: what the vertices are, what the edges are, and whether edges are weighted or directed. Then choose the algorithm (Ch 23 §9 for paths). Perform mode: Medium 25 minutes, Hard 45 minutes. The target is ≥ 70% solved and ≥ 70% chosen correctly.

**Checkpoint (21 Mar):** this set at ≥ 70%, and contests at 3 of 4 most weeks.

---

## The set

1. [Number of Closed Islands](https://leetcode.com/problems/number-of-closed-islands/)
2. [Reorder Routes to Make All Paths Lead to the City Zero](https://leetcode.com/problems/reorder-routes-to-make-all-paths-lead-to-the-city-zero/)
3. [Minimize Hamming Distance After Swap Operations](https://leetcode.com/problems/minimize-hamming-distance-after-swap-operations/)
4. [Time Needed to Inform All Employees](https://leetcode.com/problems/time-needed-to-inform-all-employees/)
5. [Count Sub Islands](https://leetcode.com/problems/count-sub-islands/)
6. [Shortest Path with Alternating Colors](https://leetcode.com/problems/shortest-path-with-alternating-colors/)
7. [Maximal Network Rank](https://leetcode.com/problems/maximal-network-rank/)
8. [Find the Safest Path in a Grid](https://leetcode.com/problems/find-the-safest-path-in-a-grid/)
9. [Count Unreachable Pairs of Nodes in an Undirected Graph](https://leetcode.com/problems/count-unreachable-pairs-of-nodes-in-an-undirected-graph/)
10. [Path With Maximum Gold](https://leetcode.com/problems/path-with-maximum-gold/)
11. [Minimum Score of a Path Between Two Cities](https://leetcode.com/problems/minimum-score-of-a-path-between-two-cities/)
12. [Regions Cut By Slashes](https://leetcode.com/problems/regions-cut-by-slashes/)
13. [Number of Restricted Paths From First to Last Node](https://leetcode.com/problems/number-of-restricted-paths-from-first-to-last-node/)
14. [Coloring A Border](https://leetcode.com/problems/coloring-a-border/)
15. [Maximum Total Importance of Roads](https://leetcode.com/problems/maximum-total-importance-of-roads/)
16. [Count the Number of Complete Components](https://leetcode.com/problems/count-the-number-of-complete-components/)
17. [Escape the Spreading Fire](https://leetcode.com/problems/escape-the-spreading-fire/)
18. [Trapping Rain Water II](https://leetcode.com/problems/trapping-rain-water-ii/)
19. [Process Restricted Friend Requests](https://leetcode.com/problems/process-restricted-friend-requests/)
20. [Minimum Moves to Move a Box to Their Target Location](https://leetcode.com/problems/minimum-moves-to-move-a-box-to-their-target-location/)
21. [Divide Nodes Into the Maximum Number of Groups](https://leetcode.com/problems/divide-nodes-into-the-maximum-number-of-groups/)
22. [Cut Off Trees for Golf Event](https://leetcode.com/problems/cut-off-trees-for-golf-event/)
23. [Minimum Cost Walk in Weighted Graph](https://leetcode.com/problems/minimum-cost-walk-in-weighted-graph/)
24. [Maximum Number of Points From Grid Queries](https://leetcode.com/problems/maximum-number-of-points-from-grid-queries/)
25. [Minimum Rounds to Complete All Tasks](https://leetcode.com/problems/minimum-rounds-to-complete-all-tasks/)

---

<details><summary>Key: open only after attempting</summary>

| # | Technique | Chapter | The insight in one line |
| :-: | :--- | :-: | :--- |
| 1 | Flood fill from the border, then count | 21 | Sink every land component touching the border first; count what's left |
| 2 | DFS with edge-direction flags | 21 | Traverse as undirected from 0; count the edges that point away from 0 |
| 3 | DSU + multiset compare per group | 24, 7 | Positions in one swap-group can be arranged freely; match the value counts per group |
| 4 | Tree DFS, maximum path cost | 19, 21 | Time to reach a node = the parent's time + the parent's inform time; take the max |
| 5 | Flood fill with a containment check | 21 | A grid-2 island counts if none of its cells is water in grid 1 |
| 6 | BFS on (node, last colour) | 21 | Augmented state: the colour of the last edge used |
| 7 | Degrees + an adjacency check | 21 | deg[a] + deg[b] − (1 if a–b is an edge); try all pairs |
| 8 | Multi-source BFS + binary search (or max-min Dijkstra) | 21, 10, 23 | Distance from every cell to the nearest thief, then maximise the minimum along a path |
| 9 | Components + counting | 21, 24 | Σ size · (n − size) / 2, using `long` |
| 10 | Backtracking | 6 | DFS from every gold cell, undoing as you return |
| 11 | Component minimum edge | 21, 24 | You can reuse roads, so the answer is the smallest edge in 1's component |
| 12 | Upscale to 3×3, or DSU on triangles | 21, 24 | Draw the slashes on a 3× grid and count regions |
| 13 | Dijkstra + DAG DP by distance | 23, 22 | Restricted edges go strictly down in distance-to-n: count paths in that order |
| 14 | Flood fill + border rule | 21 | A component cell is on the border if a neighbour is outside or differs |
| 15 | Greedy: sort by degree | 2, 9 | The highest degree gets the highest value (exchange argument) |
| 16 | Components + edge count check | 21, 24 | A component of size k is complete iff it has k(k − 1)/2 edges |
| 17 | Binary search on the wait + BFS (or two BFS) | 10, 21 | Fire arrival times by multi-source BFS; can you stay ahead of them after waiting w? |
| 18 | Heap BFS from the boundary | 15, 23 | Process the lowest boundary cell first; water level = the max seen so far |
| 19 | DSU with tentative unions | 24 | Accept a request only if it wouldn't merge any restricted pair |
| 20 | BFS on (box, player) with an inner reachability BFS | 21 | The state is the box position + the player's side; a push costs 1 |
| 21 | Bipartite check + BFS depth per component | 21 | Not bipartite → −1; otherwise the sum over components of the max BFS depth |
| 22 | Sort trees + repeated BFS | 9, 21 | Visit the trees in height order; sum the BFS distances between them |
| 23 | DSU with component AND | 24, 17 | Within a component you can walk every edge, so the answer is the AND of all its edges |
| 24 | Offline sorted queries + heap expansion | 15, 24 | Expand from (0, 0) by increasing cell value; answer queries in sorted order |
| 25 | Counting + ⌈f / 3⌉ | 7, 18 | Impossible if some count is 1; otherwise ⌈f / 3⌉ rounds per task |
</details>
