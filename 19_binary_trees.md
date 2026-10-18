# Chapter 19 — Binary Trees: Information Flow

> **Part IV · Week 17** · Prerequisites: Ch 6 (recursion contracts), Ch 8 (prefix + hash map), Ch 12 (stacks, queues)
> **Problems:** 10 worked (1 ★) · 11 practice · 2 stretch · **Time:** about 9 hours

Trees are the most heavily tested structure in interviews after arrays and strings. The good news is that nearly every tree problem is the **same recursion** with a different answer to one question:

> **What does each node need to know from above (passed down as a parameter), and what must it report upward (returned to its parent)?**

Get that information flow right and the code is a few lines. Get it wrong and you end up with global variables patched together until the tests pass. This chapter builds trees from zero, then trains that one question through six patterns.

---

## 0. From zero: trees and traversals

### 0.1 Vocabulary

A **tree** is a set of **nodes** connected by parent–child links. There is one **root** at the top, every other node has exactly one **parent**, and there are no cycles. In a **binary tree**, each node has at most two **children**, called left and right.

| Term | Meaning |
| :--- | :--- |
| **Leaf** | A node with no children |
| **Subtree** of v | v together with all of its descendants. It's a tree in its own right |
| **Depth** of v | The number of edges from the root down to v (the root has depth 0) |
| **Height** of v | The number of edges on the longest path from v down to a leaf (a leaf has height 0) |
| **Balanced** | (Interview meaning) for every node, the heights of its two subtrees differ by at most 1 |
| **Complete** | Every level is full except possibly the last, which fills from the left (the shape of a heap, Ch 15) |

All of these on one tree:

```text
                 8              ← root, depth 0          height of 8 = 3 (the path 8 → 3 → 6 → 4)
              /     \
             3       10         ← depth 1                 the parent of 6 is 3; the children of 3 are 1 and 6
            / \        \
           1   6        14      ← depth 2                 leaves: 1, 4, 7, 13
              / \      /
             4   7   13         ← depth 3                 subtree of 3 = {3, 1, 6, 4, 7}
```

It is **not** balanced: at node 10, the left subtree is empty (height −1, by convention) and the right one has height 1. It's not complete either, because level 2 has a gap (10 has no left child).

**Why height matters.** The same 4 values can form a bushy tree or a path:

```text
balanced, height 2:          2              path-shaped, height 3:   1
                           /   \                                       \
                          1     3                                       2
                                 \                                       \
                                  4                                       3
                                                                           \
                                                                            4
```

A tree with n nodes has height between about log₂ n and n − 1. Every algorithm in this chapter costs O(n) time, but **recursion depth and stack space are O(height)**, and a path-shaped tree of 10⁵ nodes makes that 10⁵.

The defining fact: **a binary tree is either empty, or a root plus a left binary tree plus a right binary tree.** That recursive definition is why recursion fits trees so naturally. The recursive contract from Ch 6 (§1) is the main tool of this whole chapter.

```java
public class TreeNode {             // LeetCode's definition
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

### 0.2 Building a tree from LeetCode's format

LeetCode writes trees **level by level**, with `null` marking a missing child: `[3, 9, 20, null, null, 15, 7]` is root 3, with children 9 and 20, where 20 has children 15 and 7:

```text
array:  [3,   9, 20,   null, null,   15, 7]
         │    └─┬─┘    └────┬───┘    └─┬─┘
        root  3's kids  9's kids     20's kids

tree:        3
            / \
           9   20
              /  \
             15   7
```

Each non-null value consumes the next two array slots for its own children. That's why 9's two `null`s come before 20's children. In memory, every node is a separate `TreeNode` object holding its value and two references, `left` and `right`, which are `null` where there's no child, exactly like a linked list with two `next` references. To test locally, you need a builder. It's a small BFS: each node waits in a queue to receive its two children from the array.

### Worked T0a *(custom)* · Build a tree, and print all four traversals

```java
static TreeNode build(Integer[] values) {
    if (values.length == 0 || values[0] == null) return null;
    TreeNode root = new TreeNode(values[0]);
    Deque<TreeNode> waiting = new ArrayDeque<>();      // nodes whose children haven't been attached yet
    waiting.offer(root);
    int i = 1;
    while (i < values.length) {
        TreeNode parent = waiting.poll();
        if (values[i] != null) { parent.left = new TreeNode(values[i]); waiting.offer(parent.left); }
        i++;
        if (i < values.length && values[i] != null) { parent.right = new TreeNode(values[i]); waiting.offer(parent.right); }
        i++;
    }
    return root;
}
```

The **depth-first traversals** differ only in *when* the node itself is visited relative to its subtrees:

```java
static void preorder(TreeNode n, List<Integer> out)  { if (n == null) return; out.add(n.val); preorder(n.left, out);  preorder(n.right, out); }
static void inorder(TreeNode n, List<Integer> out)   { if (n == null) return; inorder(n.left, out);  out.add(n.val); inorder(n.right, out); }
static void postorder(TreeNode n, List<Integer> out) { if (n == null) return; postorder(n.left, out); postorder(n.right, out); out.add(n.val); }
```

| Order | Node visited | Natural use |
| :--- | :--- | :--- |
| **Preorder** | before its subtrees | Copying or serialising a tree (the parent must exist before its children) |
| **Inorder** | between its subtrees | Binary search trees: inorder gives sorted order (Ch 20) |
| **Postorder** | after its subtrees | Anything that needs the children's answers first: heights, sums, deletion |
| **Level order** | level by level, using a queue | Anything about depth or levels; shortest paths (Ch 21) |

On the tree from §0.2 (3; children 9 and 20; 20's children 15 and 7):

```text
preorder   (node, left, right):   3  9  20 15 7
inorder    (left, node, right):   9  3  15 20 7
postorder  (left, right, node):   9  15 7  20 3
level order:                      [3]  [9, 20]  [15, 7]
```

A way to check them by hand: walk round the outside of the tree, starting left of the root, keeping the tree on your right. Preorder lists each node when you pass its **left** side, inorder when you pass **underneath** it, and postorder when you pass its **right** side.

**Common misconceptions:**

| Belief | Reality |
| :--- | :--- |
| "A binary tree's height is O(log n)" | Only if it's balanced. In general, height ≤ n − 1 |
| "Inorder of any binary tree is sorted" | Only for a binary **search** tree (Ch 20) |
| "One traversal determines the tree" | Not in general: you need two, e.g. preorder + inorder (Construct Binary Tree, §4), or a format with nulls marked |
| "Depth and height are the same thing" | Depth is measured from the root down to the node; height from the node down to its deepest leaf |
| "Recursion on trees is always safe" | Depth = height, so a path-shaped tree of 10⁵ nodes overflows Java's stack (Ch 4 §16) |

**Iteratively,** with an explicit stack (Ch 6 §5 warned that deep recursion can overflow; a path-shaped tree of 10⁵ nodes does exactly that):

```java
static List<Integer> preorderIterative(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    if (root != null) stack.push(root);
    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        out.add(node.val);
        if (node.right != null) stack.push(node.right);   // pushed first, so popped after the whole left subtree
        if (node.left != null) stack.push(node.left);
    }
    return out;
}
```

Inorder done iteratively is a practice problem: walk left, pushing each node; pop, visit, then go right. **Every traversal visits each node once: O(n) time, O(height) space.**

### Worked T0b · [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)

> Return the node values level by level, as a list of lists.

A queue processes nodes in the order they were discovered: first the root, then its children, then theirs. The one trick is knowing where each level ends: **at the start of a level, the queue holds exactly that level's nodes**, so record the queue's size and process exactly that many.

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> levels = new ArrayList<>();
    Deque<TreeNode> queue = new ArrayDeque<>();
    if (root != null) queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();                    // the number of nodes in the current level
        List<Integer> level = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        levels.add(level);
    }
    return levels;
}
```

---

## 1. Information flow: down as parameters, up as returns

### Worked T1 · [Maximum Depth of Binary Tree, both ways](https://leetcode.com/problems/maximum-depth-of-binary-tree/)

**Bottom-up** (information flows **up**). Contract: *"`depth(node)` returns the number of nodes on the longest path from node down to a leaf"*. A node needs its children's answers, so it's a postorder computation.

```java
public int maxDepth(TreeNode node) {
    if (node == null) return 0;
    return 1 + Math.max(maxDepth(node.left), maxDepth(node.right));
}
```

**Top-down** (information flows **down**). Pass each node its own depth as a parameter, and record the best depth seen at any node:

```java
private int deepest = 0;
private void visit(TreeNode node, int depth) {       // depth = number of nodes from the root to here
    if (node == null) return;
    deepest = Math.max(deepest, depth);
    visit(node.left, depth + 1);
    visit(node.right, depth + 1);
}
// maxDepth(root): deepest = 0; visit(root, 1); return deepest;
```

**How to choose.** Ask what each node needs:
* information about its **ancestors** (the path so far, a running maximum, bounds) → **pass it down** as a parameter;
* information about its **descendants** (height, sum, whether the subtree is balanced) → **return it up**.

Many problems need both: Count Good Nodes (practice) passes "the largest value on the path so far" down. And when a node needs to return *more than one* thing, return a small object or array. Balanced Binary Tree (practice) returns the height, using −1 to mean "unbalanced".

---

## 2. Return one thing, record another

The most important pattern for hard tree problems. **The value a node *returns* to its parent is not always the answer to the problem.** It's whatever the *parent* needs. The answer is recorded separately, as a running best, at the node where it's naturally complete.

### Worked T2a · [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/)

> Return the number of edges on the longest path between any two nodes. The path need not pass through the root.

Every path has a single **highest node** (the one closest to the root). At that node, the path goes down the left side and down the right side. So:

* the longest path whose highest node is v has `height(v.left) + height(v.right)` edges, counting each side as "nodes on the way down";
* **return** each node's height (the parent needs it); **record** the best left + right seen anywhere.

```java
private int diameter = 0;
public int diameterOfBinaryTree(TreeNode root) {
    diameter = 0;
    height(root);
    return diameter;
}
// Returns the number of nodes on the longest downward path starting at node.
private int height(TreeNode node) {
    if (node == null) return 0;
    int left = height(node.left), right = height(node.right);
    diameter = Math.max(diameter, left + right);     // edges in the longest path whose highest node is this one
    return 1 + Math.max(left, right);
}
```

### Worked T2b · [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) ★

> **Attempt first (30 minutes).** Node values may be negative. A path is any sequence of nodes connected by edges, each node used at most once, with at least one node. Return the maximum sum of any path.

Apply the same split:
* **Return:** the best sum of a path that **starts at this node and goes down**, because that's what a parent can extend. It can continue along only *one* side; a path that used both sides would branch at this node, and the parent couldn't attach to it.
* **Record:** the best path whose **highest node is this one**: `val + (best down on the left) + (best down on the right)`.
* **Negative branches:** a branch with a negative best-downward sum is better left off, so clamp each side at 0.

```java
private int best;
public int maxPathSum(TreeNode root) {
    best = Integer.MIN_VALUE;                       // at least one node: the answer can be negative
    down(root);
    return best;
}
// Returns the best sum of a path that starts at node and goes downward (possibly just node).
private int down(TreeNode node) {
    if (node == null) return 0;
    int left = Math.max(0, down(node.left));        // a negative branch contributes nothing
    int right = Math.max(0, down(node.right));
    best = Math.max(best, node.val + left + right); // the best path whose highest node is this one
    return node.val + Math.max(left, right);        // a parent can use only one side
}
```

**Why this considers every path:** every path has exactly one highest node, and the recursion visits every node as a candidate highest node, with the best possible extensions on each side. At most 3·10⁴ nodes of absolute value ≤ 1000 means sums stay within `int`.

**The reset matters on LeetCode:** set `best` inside the public method. Instance fields are fresh per test case, but resetting them is the habit that keeps you safe if the same object is ever reused (Ch 4 §7).

---

## 3. Searching a tree: the lowest common ancestor

### Worked T3 · [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)

> Given two nodes p and q in a binary tree, return their **lowest common ancestor**: the deepest node that has both p and q in its subtree (a node counts as its own ancestor).

**Contract:** `lca(node)` returns
* `null` if neither p nor q is in node's subtree;
* p or q, if exactly one of them is in it;
* the LCA, if both are.

```java
public TreeNode lowestCommonAncestor(TreeNode node, TreeNode p, TreeNode q) {
    if (node == null || node == p || node == q) return node;    // found one (or nothing); no need to look below
    TreeNode left = lowestCommonAncestor(node.left, p, q);
    TreeNode right = lowestCommonAncestor(node.right, p, q);
    if (left != null && right != null) return node;              // one on each side: node is the LCA
    return (left != null) ? left : right;                         // everything found is on one side
}
```

**Why returning early at p (or q) is right:** if the other node is somewhere below p, then p *is* the LCA. If it's elsewhere, some ancestor will receive p from one side and the other node from the other side, and return itself. This relies on both nodes being in the tree, which the problem guarantees. O(n). In a BST you can steer instead of searching both sides (Ch 20), and for many LCA queries on one tree there's binary lifting (Ch 36).

---

## 4. Building trees from traversals, and serialising them

### Worked T4a · [Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

> Rebuild a tree (with distinct values) from its preorder and inorder sequences.

* **Preorder starts with the root.**
* **In the inorder sequence, the root splits** the values into its left subtree (before it) and its right subtree (after it). So the left subtree has `rootIndexInInorder − inStart` nodes.
* The same two facts hold for each subtree, so recurse.

Use a map from value to inorder index so each lookup is O(1), and a single moving pointer into preorder:

```java
private int pre;                                       // the next preorder index to use
private Map<Integer, Integer> inorderIndex;

public TreeNode buildTree(int[] preorder, int[] inorder) {
    pre = 0;
    inorderIndex = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) inorderIndex.put(inorder[i], i);
    return build(preorder, 0, inorder.length);
}

// Builds the subtree whose values occupy inorder[lo, hi).
private TreeNode build(int[] preorder, int lo, int hi) {
    if (lo == hi) return null;
    TreeNode root = new TreeNode(preorder[pre++]);     // preorder visits the root first
    int mid = inorderIndex.get(root.val);
    root.left = build(preorder, lo, mid);              // inorder[lo, mid) is the left subtree, built next in preorder
    root.right = build(preorder, mid + 1, hi);
    return root;
}
```

Why can a single moving `pre` pointer work? Preorder lists the entire left subtree before the right subtree, and the recursion builds the left subtree before the right one, consuming preorder in exactly that order. **O(n).**

### Worked T4b · [Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)

> Convert a tree to a string and back.

Preorder alone doesn't determine a tree, because many shapes share it. **Preorder with explicit null markers does.** Every node is followed by its complete left subtree, then its complete right subtree, and the markers say exactly where each one ends.

```java
public String serialize(TreeNode root) {
    StringBuilder sb = new StringBuilder();
    write(root, sb);
    return sb.toString();
}
private void write(TreeNode node, StringBuilder sb) {
    if (node == null) { sb.append("#,"); return; }
    sb.append(node.val).append(',');
    write(node.left, sb);
    write(node.right, sb);
}

public TreeNode deserialize(String data) {
    Deque<String> tokens = new ArrayDeque<>(Arrays.asList(data.split(",")));
    return read(tokens);
}
// Reads one complete subtree from the front of tokens: the mirror image of write().
private TreeNode read(Deque<String> tokens) {
    String t = tokens.poll();
    if (t.equals("#")) return null;
    TreeNode node = new TreeNode(Integer.parseInt(t));
    node.left = read(tokens);
    node.right = read(tokens);
    return node;
}
```

`deserialize` mirrors `serialize` exactly, which is the easiest way to keep them consistent. Both are O(n). (`split(",")` drops the trailing empty string, as Ch 4 §6 noted.)

---

## 5. State along the root-to-node path

### Worked T5 · [Path Sum III](https://leetcode.com/problems/path-sum-iii/)

> Count the paths that go **downward** (parent to child, starting and ending anywhere) and sum to `targetSum`.

A downward path ending at node v is a contiguous piece of the **root-to-v path**. That's Ch 8's "pair of prefixes", with the array replaced by the current root-to-node path. Keep a hash map of the prefix sums *on the current path*:
* entering a node: count the earlier prefixes equal to `current − target`, then add `current` to the map;
* **leaving a node: remove it again**. This is backtracking (Ch 6 §2), so sibling subtrees don't see each other's prefixes.

```java
public int pathSum(TreeNode root, int targetSum) {
    Map<Long, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0L, 1);                                    // the empty prefix, above the root
    return count(root, 0L, targetSum, prefixCount);
}

// Paths ending in node's subtree; `above` is the sum from the root down to node's parent.
private int count(TreeNode node, long above, int target, Map<Long, Integer> prefixCount) {
    if (node == null) return 0;
    long current = above + node.val;                           // values up to 1e9 deep down a path: long
    int paths = prefixCount.getOrDefault(current - target, 0); // paths ending exactly here
    prefixCount.merge(current, 1, Integer::sum);
    paths += count(node.left, current, target, prefixCount) + count(node.right, current, target, prefixCount);
    prefixCount.merge(current, -1, Integer::sum);              // undo: we're leaving this path
    return paths;
}
```

O(n) instead of O(n²). Prefix sums need `long`: node values up to 10⁹ along a path of a thousand nodes overflow `int`.

---

## 6. Levels

### Worked T6 · [Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)

> Return the values visible from the right side: the last node of each level.

The level-order loop of T0b records each level's last node. Alternatively, run a DFS that visits **right before left** and records the first node seen at each new depth:

```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> view = new ArrayList<>();
    see(root, 0, view);
    return view;
}
private void see(TreeNode node, int depth, List<Integer> view) {
    if (node == null) return;
    if (depth == view.size()) view.add(node.val);     // the first node reached at this depth is the rightmost
    see(node.right, depth + 1, view);
    see(node.left, depth + 1, view);
}
```

Zigzag order, maximum width and vertical order (all in the practice set) are level-order variations. Maximum Width numbers the nodes like a heap (children of i at 2i and 2i + 1), which overflows on deep trees unless you **renumber relative to each level's first index**.

---

## 7. Where tree recursion breaks

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| A skewed tree with 10⁵ nodes | The recursion depth overflows the stack | An explicit stack (§0.2), or a bigger thread stack in OAs (Ch 4 §16) |
| A node needs information from its **parent's side**, not just its subtree | One pass can't see it | Two passes (down, then up), i.e. *rerooting* (Ch 31); or convert the tree to a graph |
| Distances that go up *and* down ("all nodes at distance k from a target") | Parent links aren't stored | Record each node's parent, then BFS as if it were a graph (Ch 21; practice) |
| Global fields carried across calls | Wrong answers on reuse | Reset them in the public method, or return tuples |
| Many LCA queries on the same tree | O(n) per query | Binary lifting, O(log n) per query (Ch 36) |

---

## 8. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| A path must go from the root to a leaf | Top-down running sum; check at leaves (Path Sum II, with backtracking) |
| The best path where adjacent nodes differ | T2 shape, with a condition on extending (Ch 31) |
| Maximum independent set ("rob" non-adjacent nodes) | Return two values per node: with it and without it (Ch 31) |
| A binary search tree | Use the order to steer (Ch 20) |
| Build from postorder + inorder | The root is postorder's last element; build the right subtree first |

---

## 9. Recognition

**The first question for every tree problem:** *what does a node need from its children (return value), and what from its ancestors (parameter)?*

**Signals:**
* **Height, depth, balance, subtree sums** → bottom-up returns.
* **The path from the root so far** (a maximum, a sum, a string) → top-down parameters.
* **"Any path"** (diameter, maximum path sum, longest anything) → return the downward best, record the best through the node.
* **"Levels", "nearest", "row by row", "view"** → level-order BFS.
* **Two nodes and their meeting point** → LCA.
* **"Encode / decode / rebuild"** → preorder with null markers, or traversal pairs.

---

## 10. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Returning the recorded answer instead of what the parent needs | Write both down: "returns ___", "records ___" |
| Forgetting that a path can't branch | Return only one side; record both sides |
| Clamping a branch at 0 when the problem forbids empty paths | `best` starts at `MIN_VALUE`; clamp only the *extensions*, never the node itself |
| Not undoing path state (Path Sum III) | Remove on the way out |
| Not resetting fields between calls | Reset in the public entry point |
| `int` prefix sums along long paths | `long` |
| Recursion depth on degenerate trees | Iterate, or bound the depth |

---

## 11. Practice

1. **[Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/)** *(Easy)*
2. **[Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/)** *(Easy)*
3. **[Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)** *(Easy; iterative)*
4. **[Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/)** *(Medium)*
5. **[Count Good Nodes in Binary Tree](https://leetcode.com/problems/count-good-nodes-in-binary-tree/)** *(Medium)*
6. **[Path Sum II](https://leetcode.com/problems/path-sum-ii/)** *(Medium)*
7. **[Flatten Binary Tree to Linked List](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/)** *(Medium)*
8. **[Maximum Width of Binary Tree](https://leetcode.com/problems/maximum-width-of-binary-tree/)** *(Medium)*
9. **[Distribute Coins in Binary Tree](https://leetcode.com/problems/distribute-coins-in-binary-tree/)** *(Medium)*
10. **[All Nodes Distance K in Binary Tree](https://leetcode.com/problems/all-nodes-distance-k-in-binary-tree/)** *(Medium)*
11. **[Vertical Order Traversal of a Binary Tree](https://leetcode.com/problems/vertical-order-traversal-of-a-binary-tree/)** *(Hard)*

**Stretch:** **[Step-By-Step Directions From a Binary Tree Node to Another](https://leetcode.com/problems/step-by-step-directions-from-a-binary-tree-node-to-another/)** *(Medium)* · **[Recover a Tree From Preorder Traversal](https://leetcode.com/problems/recover-a-tree-from-preorder-traversal/)** *(Hard)*

<details><summary>Nudges</summary>

1. Balanced: return the height, or −1 if any subtree is unbalanced, so one pass does both jobs.
2. Subtree: for each node of the big tree, check "same tree". Or serialise both trees with null markers and use a substring search.
3. Iterative Inorder: push left children all the way down; pop, visit, then move to the right child.
4. Zigzag: level order, reversing every other level (or adding to the front of a deque).
5. Good Nodes: pass down the maximum value on the path so far.
6. Path Sum II: top-down sum plus a path list, with backtracking; record a copy at each matching leaf.
7. Flatten: process right, then left, then the node, keeping a `prev` pointer (reverse preorder). Or splice each left subtree between the node and its right subtree.
8. Width: level order with heap-style indices; subtract each level's first index before computing children's indices.
9. Coins: return each subtree's excess (coins − nodes). Every coin crossing an edge costs one move, so add |excess| for each child edge.
10. Distance K: record parent pointers (one DFS), then BFS from the target, treating the tree as an undirected graph.
11. Vertical Order: traverse recording (column, row, value), then sort by column, then row, then value.
12. Stretch, Directions: find the root-to-start and root-to-dest paths as strings of L/R; strip their common prefix; replace what's left of the start path with U.
13. Stretch, Recover: parse (depth, value) pairs; keep a stack of the current path, popping until its size equals the depth.
</details>

<details><summary>Key</summary>

Balanced → T2 (height, or −1) · Subtree → T3-style search / serialisation · Iterative Inorder → explicit stack · Zigzag → T6 · Good Nodes → T1 top-down maximum · Path Sum II → T5 with backtracking · Flatten → reverse preorder or T2 · Max Width → T6 with normalised indices · Distribute Coins → T2 (excess flows through edges) · Distance K → tree to graph + BFS · Vertical Order → traversal + sort · Directions → T3 + path strings · Recover → stack by depth
</details>

---

## 12. Self-test

<details><summary>1. Which traversal do you use to compute subtree heights, and why?</summary>

Postorder, because a node's height needs both children's heights first.
</details>

<details><summary>2. In Maximum Path Sum, why does the function return only one side but record both?</summary>

A path that a parent extends can't branch at this node, so it continues down one side only. The path whose highest node *is* this one can use both sides, and that's the candidate to record.
</details>

<details><summary>3. Why does the LCA recursion return a node as soon as it equals p or q?</summary>

If the other node is below it, it's the LCA. If not, an ancestor will receive it from one side and the other node from the other side, and that ancestor returns itself.
</details>

<details><summary>4. Why does preorder with null markers determine a tree uniquely, when plain preorder doesn't?</summary>

Each node is followed by its complete left subtree and then its complete right subtree, and the null markers show exactly where each one ends, so the parse is forced.
</details>

<details><summary>5. In Path Sum III, what goes wrong if you don't decrement the prefix count when leaving a node?</summary>

Prefixes from one branch stay in the map while the other branch is explored, so you count "paths" that jump between sibling subtrees.
</details>

---

## 13. Exit gate

Move on when you can:

* build a tree from LeetCode's array format and produce all four traversals, both recursively and iteratively;
* for any tree problem, state what flows down and what flows up *before* coding;
* write Maximum Path Sum, LCA, Construct from Preorder/Inorder and Serialize/Deserialize from blank files, each in under 12 minutes;
* explain the "return one, record another" split, with the diameter as your example;
* solve 9 of the 11 practice problems.
