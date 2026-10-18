# Mixed Set IV — after Chapter 20

> **Week 18** · 22 problems, unlabeled, drawn from Ch 1–20 · **Time:** about 9 hours

Same rules as before. Before coding, write the restatement and budget, the bottleneck, the **top two candidates with their structural reasons**, and the proof sentence. For tree problems, also write **what flows down and what flows up** (Ch 19 §1). Perform mode: Medium 25 minutes, Hard 45 minutes. The target is ≥ 70% solved and ≥ 70% chosen correctly.

---

## The set

1. [Maximum Difference Between Node and Ancestor](https://leetcode.com/problems/maximum-difference-between-node-and-ancestor/)
2. [Convert BST to Greater Tree](https://leetcode.com/problems/convert-bst-to-greater-tree/)
3. [Find Duplicate Subtrees](https://leetcode.com/problems/find-duplicate-subtrees/)
4. [Even Odd Tree](https://leetcode.com/problems/even-odd-tree/)
5. [Delete Nodes And Return Forest](https://leetcode.com/problems/delete-nodes-and-return-forest/)
6. [Pseudo-Palindromic Paths in a Binary Tree](https://leetcode.com/problems/pseudo-palindromic-paths-in-a-binary-tree/)
7. [Lowest Common Ancestor of Deepest Leaves](https://leetcode.com/problems/lowest-common-ancestor-of-deepest-leaves/)
8. [Balance a Binary Search Tree](https://leetcode.com/problems/balance-a-binary-search-tree/)
9. [Kth Largest Sum in a Binary Tree](https://leetcode.com/problems/kth-largest-sum-in-a-binary-tree/)
10. [Create Binary Tree From Descriptions](https://leetcode.com/problems/create-binary-tree-from-descriptions/)
11. [Populating Next Right Pointers in Each Node II](https://leetcode.com/problems/populating-next-right-pointers-in-each-node-ii/)
12. [Smallest String Starting From Leaf](https://leetcode.com/problems/smallest-string-starting-from-leaf/)
13. [Count Nodes Equal to Average of Subtree](https://leetcode.com/problems/count-nodes-equal-to-average-of-subtree/)
14. [Validate Binary Tree Nodes](https://leetcode.com/problems/validate-binary-tree-nodes/)
15. [Longest Univalue Path](https://leetcode.com/problems/longest-univalue-path/)
16. [Cousins in Binary Tree II](https://leetcode.com/problems/cousins-in-binary-tree-ii/)
17. [Number of Good Leaf Nodes Pairs](https://leetcode.com/problems/number-of-good-leaf-nodes-pairs/)
18. [Sum of Prefix Scores of Strings](https://leetcode.com/problems/sum-of-prefix-scores-of-strings/)
19. [Stream of Characters](https://leetcode.com/problems/stream-of-characters/)
20. [Maximum Genetic Difference Query](https://leetcode.com/problems/maximum-genetic-difference-query/)
21. [Longest Subarray With Maximum Bitwise AND](https://leetcode.com/problems/longest-subarray-with-maximum-bitwise-and/)
22. [Minimum Operations to Halve Array Sum](https://leetcode.com/problems/minimum-operations-to-halve-array-sum/)

---

<details><summary>Key: open only after attempting</summary>

| # | Technique | Chapter | The insight in one line |
| :-: | :--- | :-: | :--- |
| 1 | Top-down min/max along the path | 19 | Pass the path's min and max down; the answer at a leaf is max − min |
| 2 | Reverse inorder | 20 | Visit right, node, left, keeping a running sum of the larger values |
| 3 | Serialisation + hashing | 19, 7 | Serialise each subtree bottom-up; a repeated string means a duplicate |
| 4 | Level-order BFS | 19 | Check parity and strict monotonicity within each level |
| 5 | Postorder with a "delete" set | 19 | Return null for deleted nodes; their surviving children become new roots |
| 6 | Path parity mask | 17, 19 | Toggle bit `val` down the path; at a leaf, the mask has at most one bit set |
| 7 | Bottom-up (depth, node) | 19 | Return the deepest depth and the LCA of the deepest leaves in each subtree |
| 8 | Inorder → rebuild | 20 | Inorder gives a sorted list; build from the middle (Sorted Array to BST) |
| 9 | Level sums + heap / sort | 19, 15 | Sum each level; return the k-th largest (`long`) |
| 10 | Hashing child flags to find the root | 7 | Build the nodes in a map; the root is the one that's never a child |
| 11 | O(1)-space level linking | 19 | Walk the current level through its `next` pointers while building the next level with a dummy |
| 12 | DFS with path strings | 19 | Build the leaf-to-root string at each leaf; keep the smallest |
| 13 | Bottom-up (sum, count) | 19 | Return the subtree sum and size; compare the average at each node |
| 14 | In-degrees + one root + connectivity | 19 (Ch 21 preview) | Exactly one node with in-degree 0, none with in-degree > 1, and every node reachable from it |
| 15 | Return vs record | 19 | Return the longest same-value arm down; record left + right arms through the node |
| 16 | Level sums | 19 | New value = level sum − (the node and its sibling's sum), computed one level ahead |
| 17 | Bottom-up distance counts | 19 | Return the counts of leaves at each distance ≤ d; pair counts across the two children |
| 18 | Trie with prefix counts | 20 | Store how many words pass through each node; a word's score is the sum along its path |
| 19 | Reversed trie | 20 | Insert reversed words; keep a buffer of recent characters and walk the trie backwards |
| 20 | Offline DFS + binary trie with deletion | 20 | Insert the path values going down, remove them going up, answer each node's queries at the node |
| 21 | AND only shrinks | 17 | The answer is the longest run of the array's maximum value |
| 22 | Greedy max-heap | 15 | Always halve the current largest value; prove it with an exchange argument |
</details>
