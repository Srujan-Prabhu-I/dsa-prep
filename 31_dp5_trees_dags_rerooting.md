# Chapter 31 — DP V: Trees, DAGs & Rerooting

> **Part VI · Week 27** · Prerequisites: Ch 19 (information flow, "return one, record another"), Ch 21 (adjacency lists, BFS), Ch 22 (DP on DAGs), Ch 27
> **Problems:** 4 worked (1 ★) · 7 practice · 2 stretch · **Time:** about 8 hours

Trees are the friendliest structure for DP. A subtree is a self-contained subproblem, and the children of a node are its "smaller instances". **Tree DP** = Ch 19's information flow + Ch 27's state: each node computes a small vector of values (one per state) from its children's vectors, in postorder. This chapter covers the three shapes interviews use (include/exclude, multi-state covering, and diameter-style combining), then **rerooting**, which computes the answer *for every node as the root* in O(n) instead of O(n²).

DP on general DAGs is Ch 22 §4. Everything there applies here too, because a rooted tree is a DAG.

---

## 0. From zero: tree DP on general trees, without deep recursion

Binary trees come as `TreeNode` objects, and recursion is natural (Ch 19). **General trees usually come as an edge list or a parent array**, with n up to 10⁵. Recursing on a path-shaped tree of that size overflows Java's stack (Ch 4 §16). The standard fix:

1. **Root the tree** (usually at vertex 0) with a BFS, recording each vertex's `parent` and the **BFS order**.
2. In BFS order, every vertex comes **after its parent**. So iterating the order **backwards** processes every child before its parent, which is all a postorder DP needs. Iterating it **forwards** processes parents first, which top-down passes need (§5).

```java
// Roots the tree at 0. Fills parent[] (parent[0] = -1) and returns the vertices in BFS order.
static int[] bfsOrder(List<List<Integer>> adj, int[] parent) {
    int n = adj.size();
    int[] order = new int[n];
    Arrays.fill(parent, -2);                 // -2 = not visited yet
    parent[0] = -1;
    order[0] = 0;
    for (int head = 0, tail = 1; head < tail; head++) {
        int u = order[head];
        for (int v : adj.get(u)) {
            if (parent[v] != -2) continue;   // the parent (or an already-seen vertex)
            parent[v] = u;
            order[tail++] = v;
        }
    }
    return order;
}
```

---

## 1. Attempt first ★

> **[House Robber III](https://leetcode.com/problems/house-robber-iii/) (25 minutes).** The houses form a binary tree, and robbing two directly linked houses alerts the police. Return the most money you can rob.

---

## 2. Include or exclude: two values per node

### Worked R1 · [House Robber III](https://leetcode.com/problems/house-robber-iii/) ★

The Ch 27 state "`best[i]` for the first i houses" becomes "the best for **this subtree**". One value isn't enough, though. Whether a node can be robbed depends on whether its **children** were, so return **two** values:

> `robbed` = the best for the subtree **with** this node robbed (so both children must be skipped);
> `skipped` = the best **without** this node (each child is free to be robbed or not).

```java
public int rob(TreeNode root) {
    int[] best = robOrSkip(root);
    return Math.max(best[0], best[1]);
}

// Returns {best if this node is robbed, best if it isn't}.
private int[] robOrSkip(TreeNode node) {
    if (node == null) return new int[] {0, 0};
    int[] left = robOrSkip(node.left), right = robOrSkip(node.right);
    int robbed = node.val + left[1] + right[1];                                // children must be skipped
    int skipped = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);  // children are free
    return new int[] {robbed, skipped};
}
```

That's O(n). **This is the maximum-weight independent set on a tree**, a problem that's NP-hard on general graphs but linear on trees, because the subtrees of different children never interact. CSES Tree Matching (practice) is the same idea with "is this node matched to one of its children?" as the state.

---

## 3. More than two states: covering

### Worked R2 · [Binary Tree Cameras](https://leetcode.com/problems/binary-tree-cameras/)

> A camera at a node covers the node itself, its parent and its children. Return the fewest cameras that cover every node.

Each node ends in one of three states, from its parent's point of view:

| State | Meaning | What the parent must do |
| :--- | :--- | :--- |
| `HAS_CAMERA` | A camera here | Nothing; and the parent is covered |
| `COVERED` | No camera, but a child's camera covers it | Nothing |
| `NOT_COVERED` | Nobody covers it yet | **The parent must have a camera** |

**Decide each node from its children's states, as late as possible:**
* if any child is `NOT_COVERED`, this node **must** hold a camera;
* otherwise, if any child has a camera, this node is `COVERED`;
* otherwise, leave it `NOT_COVERED` and let its parent cover it.

**Why "as late as possible" is optimal (exchange):** a camera placed one level higher covers a superset of the useful nodes. Take an optimal solution with a camera on a leaf: moving it to the leaf's parent still covers the leaf, and it also covers the parent's parent and the leaf's siblings. So cameras on parents of uncovered nodes are never worse.

```java
private static final int NOT_COVERED = 0, COVERED = 1, HAS_CAMERA = 2;
private int cameras;

public int minCameraCover(TreeNode root) {
    cameras = 0;
    if (state(root) == NOT_COVERED) cameras++;          // there's nobody above to cover the root
    return cameras;
}

private int state(TreeNode node) {
    if (node == null) return COVERED;                    // an empty child needs nothing and offers nothing
    int left = state(node.left), right = state(node.right);
    if (left == NOT_COVERED || right == NOT_COVERED) {   // a child depends on us
        cameras++;
        return HAS_CAMERA;
    }
    if (left == HAS_CAMERA || right == HAS_CAMERA) return COVERED;
    return NOT_COVERED;                                   // leave it to our parent: higher cameras cover more
}
```

That's O(n). It's a tree DP whose transitions became deterministic because the exchange argument settled every choice in advance. When no such argument exists, keep all three states as *costs*, `best[node][state]`, and take minimums, which is the general version that Maximum Sum BST (practice) needs.

---

## 4. Combining children: diameter-style DP on general trees

### Worked R3 · [Longest Path With Different Adjacent Characters](https://leetcode.com/problems/longest-path-with-different-adjacent-characters/)

> A tree is given as a parent array, with a letter on each node. Return the length (in nodes) of the longest path on which no two adjacent nodes have the same letter.

This is Ch 19's diameter on a **general tree, with a condition**:
* **Return** (to the parent): the longest valid **downward** chain starting at v, which is 1 + the longest child chain whose child letter differs from v's.
* **Record:** the best path whose highest node is v, which is 1 + the **two** longest such child chains.

Keep the top two child values per node, and process children before parents in reverse BFS order:

```java
public int longestPath(int[] parent, String s) {
    int n = parent.length;
    List<List<Integer>> children = new ArrayList<>();
    for (int i = 0; i < n; i++) children.add(new ArrayList<>());
    for (int i = 1; i < n; i++) children.get(parent[i]).add(i);
    int[] order = new int[n];                              // BFS order from the root (node 0)
    for (int head = 0, tail = 1; head < tail; head++) for (int c : children.get(order[head])) order[tail++] = c;
    int[] down = new int[n];                               // longest valid downward chain from v, in nodes
    int answer = 1;
    for (int idx = n - 1; idx >= 0; idx--) {               // children before parents
        int v = order[idx], first = 0, second = 0;         // the two longest usable child chains
        for (int c : children.get(v)) {
            if (s.charAt(c) == s.charAt(v)) continue;      // the same letter can't be adjacent on the path
            if (down[c] > first) { second = first; first = down[c]; }
            else if (down[c] > second) second = down[c];
        }
        down[v] = 1 + first;                               // return: one arm
        answer = Math.max(answer, 1 + first + second);     // record: both arms through v
    }
    return answer;
}
```

That's O(n), with no recursion. **Keeping the top two child values** is a pattern you'll reuse whenever a path bends at a node.

---

## 5. Rerooting: the answer for every node as root

Some problems ask for a value **for every node, as if that node were the root**: the sum of distances, the farthest node, or the number of correct guesses. Recomputing a tree DP from each root costs O(n²). **Rerooting does it in O(n) with two passes:**

1. **Down pass** (children before parents): compute subtree values for one fixed root.
2. **Up pass** (parents before children): derive each child's whole-tree answer from its **parent's** whole-tree answer, adjusting for the one edge that changes direction.

### Worked R4 · [Sum of Distances in Tree](https://leetcode.com/problems/sum-of-distances-in-tree/)

> For every node, return the sum of its distances to all other nodes. (n ≤ 3·10⁴.)

**Down pass, rooted at 0:** `size[v]` = the number of nodes in v's subtree, and `sub[v]` = the sum of distances from v to the nodes **in its subtree**. Each child c contributes `sub[c] + size[c]`, because every node in c's subtree is one edge further from v than from c.

**The root's answer** is `sub[0]`, since its subtree is the whole tree.

**Up pass: moving the root from p to a child c.** The size[c] nodes in c's subtree each get **1 closer**; the other n − size[c] nodes each get **1 farther**. So

> `answer[c] = answer[p] − size[c] + (n − size[c])`.

```java
public int[] sumOfDistancesInTree(int n, int[][] edges) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int[] e : edges) { adj.get(e[0]).add(e[1]); adj.get(e[1]).add(e[0]); }
    int[] parent = new int[n];
    int[] order = bfsOrder(adj, parent);                   // §0, rooted at 0
    int[] size = new int[n], sub = new int[n];
    for (int i = n - 1; i >= 0; i--) {                     // down pass: children before parents
        int v = order[i];
        size[v] += 1;                                      // v itself (its children were already added)
        if (parent[v] >= 0) {
            size[parent[v]] += size[v];
            sub[parent[v]] += sub[v] + size[v];            // v's whole subtree is one edge further from the parent
        }
    }
    int[] answer = new int[n];
    answer[0] = sub[0];
    for (int i = 1; i < n; i++) {                          // up pass: parents before children
        int c = order[i], p = parent[c];
        answer[c] = answer[p] - size[c] + (n - size[c]);   // re-root across the edge p–c
    }
    return answer;
}
```

That's O(n). The answers are at most about n²/2 ≈ 4.5·10⁸, which fits in `int`. For larger n, use `long`.

**The general rerooting recipe.** To move the root across an edge p – c, you need "the answer for p's side *without* c's subtree". When values combine by sums, as here, you can **subtract** c's contribution. When they combine by **max** (the farthest node), you can't subtract a max, so keep each node's **top two** child values (as in §4), or prefix and suffix maxima over its children, and use the best value that didn't come from c. Count Number of Possible Root Nodes (practice) is the sum version; CSES Tree Distances I is the max version.

---

## 6. Where tree DP breaks

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Recursion on 10⁵-node trees | Stack overflow | The reverse-BFS-order pattern (§0), or a big thread stack in OAs |
| Treating the undirected adjacency of a tree as directed | Revisiting the parent loops forever | Skip the parent (parent array, or a visited flag) |
| "For every root" done as n separate DPs | O(n²) | Rerooting (§5) |
| Rerooting a max-combined value by subtraction | You can't subtract a max | Top-two child values, or prefix/suffix maxima |
| The structure isn't a tree (it has cycles) | Subtrees aren't independent | Graph techniques, or collapse the SCCs and use DAG DP (Ch 25) |

---

## 7. Recognition

**Signals:**
* A tree, plus "choose nodes with constraints between neighbours" (no two adjacent, cover all) → **include/exclude** or **multi-state** tree DP.
* "The longest / best **path** in a tree with a condition" → return one arm, record two arms.
* "**For every node** as the root / as the meeting point / as the capital" → **rerooting**.
* "Moves along edges toward a root" (fuel, coins) → subtree sizes per edge.

---

## 8. Implementation traps

| Trap | Fix |
| :--- | :--- |
| A single value per node when the parent's choice needs more | Return a vector of states |
| Forgetting the root's own special case (Cameras) | Handle it after the DP finishes |
| `+1`s in the wrong place when counting nodes versus edges | Comment whether lengths count nodes or edges |
| Rerooting formulas off by the size of the moved subtree | Re-derive the "closer/farther" counts with a three-node example |
| `int` overflow in distance sums for large n | `long` |

---

## 9. Practice

1. **[Number of Nodes in the Sub-Tree With the Same Label](https://leetcode.com/problems/number-of-nodes-in-the-sub-tree-with-the-same-label/)** *(Medium)*
2. **[Minimum Fuel Cost to Report to the Capital](https://leetcode.com/problems/minimum-fuel-cost-to-report-to-the-capital/)** *(Medium)*
3. **[Most Profitable Path in a Tree](https://leetcode.com/problems/most-profitable-path-in-a-tree/)** *(Medium)*
4. **[CSES Tree Matching](https://cses.fi/problemset/task/1130)** *(Medium; stdin/stdout)*
5. **[CSES Tree Distances I](https://cses.fi/problemset/task/1132)** *(Medium; stdin/stdout)*
6. **[Maximum Sum BST in Binary Tree](https://leetcode.com/problems/maximum-sum-bst-in-binary-tree/)** *(Hard)*
7. **[Count Number of Possible Root Nodes](https://leetcode.com/problems/count-number-of-possible-root-nodes/)** *(Hard)*

**Stretch:** **[Minimize the Total Price of the Trips](https://leetcode.com/problems/minimize-the-total-price-of-the-trips/)** *(Hard)* · **[Difference Between Maximum and Minimum Price Sum](https://leetcode.com/problems/difference-between-maximum-and-minimum-price-sum/)** *(Hard)*

<details><summary>Nudges</summary>

1. Same Label: each node returns a 26-count array for its subtree; its answer is the count of its own letter.
2. Fuel Cost: the people from v's subtree all cross the edge from v to its parent, needing ⌈size[v] / seats⌉ cars. Use `long`.
3. Profitable Path: compute Bob's arrival time at each node on his path to 0. Alice's DFS gains the full amount, half or nothing, depending on who arrives first.
4. Tree Matching: two states, `free[v]` (v unmatched) and `matched[v]` (v matched to one child): try matching each child and pick the best gain.
5. Tree Distances I: the farthest node from each vertex. Rerooting with the top-two downward depths, or the two diameter endpoints plus two BFS runs.
6. Max Sum BST: each node returns (is this a BST, min, max, sum) and records the best sum.
7. Possible Roots: count the correct guesses for root 0, then reroot. Moving the root across the edge p–c flips only the guesses about that edge.
8. Stretch, Total Price: count how many trips pass through each node (path counting with LCA or DFS), then House Robber III on the weights count · price, halving prices for "robbed" nodes.
9. Stretch, Price Sum: for each root, (the maximum path sum from it) − its own price. Rerooting with the top-two child values.
</details>

<details><summary>Key</summary>

Same Label → subtree counts per letter · Fuel Cost → subtree sizes per edge · Profitable Path → Bob's arrival times + DFS · Tree Matching → R1 · Tree Distances I → R4 (max version) or two BFS · Max Sum BST → R2-style tuple return · Possible Roots → R4 counting correct guesses · Total Price → path counts + R1 · Price Sum → R4 with top-two children
</details>

---

## 10. Self-test

<details><summary>1. Why can tree DP solve maximum independent set in O(n), when it's NP-hard on general graphs?</summary>

The subtrees of different children share no edges, so once a node's choice is fixed, its children's subproblems are independent and can be combined by simple sums.
</details>

<details><summary>2. Why is reverse BFS order a valid substitute for postorder?</summary>

In BFS order every vertex appears after its parent, so reading the order backwards visits every child before its parent, and that's the only thing the bottom-up DP needs.
</details>

<details><summary>3. Derive the rerooting formula for Sum of Distances.</summary>

Moving the root from p to c makes the size[c] nodes of c's subtree one edge closer and the other n − size[c] nodes one edge farther. So answer[c] = answer[p] − size[c] + (n − size[c]).
</details>

<details><summary>4. Why can't you reroot a "farthest distance" value by subtraction?</summary>

Max isn't invertible: removing the child that achieved the maximum doesn't tell you the next-best value. Keep the top two values (or prefix/suffix maxima).
</details>

<details><summary>5. Why place cameras as high as possible in Binary Tree Cameras?</summary>

By exchange: moving a camera from a leaf to its parent still covers the leaf and covers a superset of its other useful nodes, so it's never worse.
</details>

---

## 11. Exit gate

Move on when you can:

* root a general tree iteratively and run a bottom-up DP in reverse BFS order;
* write House Robber III and Binary Tree Cameras from blank files, with the state meanings stated;
* write the top-two-children path DP (Longest Path With Different Adjacent Characters);
* derive and code Sum of Distances by rerooting, and say how you'd handle a max-combined value;
* solve 6 of the 7 practice problems.
