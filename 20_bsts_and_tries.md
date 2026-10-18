# Chapter 20 — BSTs & Tries

> **Part IV · Week 18** · Prerequisites: Ch 6 (backtracking, Word Search), Ch 15 §0.4 (what `TreeMap` is), Ch 17 (bits), Ch 19
> **Problems:** 6 worked (1 ★) · 9 practice · 3 stretch · **Time:** about 8 hours

Two trees that **organise data by its content**, so that the shape of the tree itself answers questions:
* A **binary search tree (BST)** orders keys so that every comparison discards a whole subtree. It's the structure behind `TreeMap`, and its invariant is a favourite interview topic, because the obvious check is wrong.
* A **trie** organises strings by their **prefixes**, so every word sharing a prefix shares a path. That makes "which words start with…" and "search many words at once" cheap. A **binary trie** does the same for the bits of numbers, and it's the standard tool for maximum-XOR problems.

---

## 0. From zero: binary search trees

### 0.1 The invariant

A **binary search tree** is a binary tree (Ch 19) where, **for every node, every value in its left subtree is smaller, and every value in its right subtree is larger.** (With duplicates, pick a convention, such as "equal goes right", and keep it.)

**The consequence that matters most: an inorder traversal of a BST visits the values in sorted order.** *Proof by induction on the tree:* inorder visits the whole left subtree (sorted, by induction, and all smaller than the root), then the root, then the whole right subtree (sorted, and all larger).

```text
                8               every value left of 8 is < 8:  {3, 1, 6, 4, 7}
             /     \            every value right of 8 is > 8: {10, 14, 13}
            3       10
           / \        \         inorder: 1 3 4 6 7 8 10 13 14  (sorted)
          1   6        14
             / \      /
            4   7   13
```

### 0.2 Operations

**Searching for 7** in the tree above compares 7 with one node per level and discards a whole subtree each time:

```text
at 8:  7 < 8 → go left   (the right subtree {10, 13, 14} can't contain 7)
at 3:  7 > 3 → go right  (the left subtree {1} can't contain 7)
at 6:  7 > 6 → go right
at 7:  found              4 comparisons = the depth of 7, plus one
```

**Inserting 5** follows the same path a search for 5 would (8 → left, 3 → right, 6 → left, 4 → right) and attaches 5 where that path falls off the tree, as the right child of 4.

* **Search** for x: at each node, x is either here, or (being smaller) only possibly in the left subtree, or (being larger) only possibly in the right one. Follow one branch. **O(h)**, where h is the height.
* **Insert** x: search for it; where the search falls off the tree (at a `null`), attach the new node. O(h). (Insert into a Binary Search Tree, in the practice set.)
* **Delete:** three cases, in Worked S3.
* **Minimum / maximum:** keep going left / right. **Successor** of a node with a right subtree: the minimum of that right subtree.

### 0.3 Why balance matters

Everything costs **O(h)**. If the tree is **balanced**, h ≈ log₂ n, and every operation is logarithmic. But insert the values in sorted order (1, 2, 3, …) and each new node becomes the right child of the last: the "tree" is a chain with **h = n**, and every operation is O(n), no better than a list:

```text
insert 4, 2, 6, 1, 3, 5, 7:          insert 1, 2, 3, 4, 5, 6, 7:
            4                         1
          /   \                        \
         2     6                        2
        / \   / \                        \
       1   3 5   7                        3 ...  → 7      height 6
    height 2: search ≤ 3 steps                            search up to 7 steps
```

The same values, in a different insertion order, give a completely different shape.

**Self-balancing trees** (AVL trees, red–black trees) fix this by restructuring locally with *rotations* after each insert or delete, keeping h = O(log n) guaranteed. Java's `TreeMap` and `TreeSet` are red–black trees. **Interviews almost never ask you to implement one.** They expect you to know *why* balance matters and to use `TreeMap` when you need a balanced tree. The interview questions about plain BSTs are about the **invariant** and the **traversal**.

**Common misconceptions:**

| Belief | Reality |
| :--- | :--- |
| "BST operations are O(log n)" | O(height). Only a **balanced** BST guarantees O(log n); a plain one can be O(n) |
| "Checking each node against its children validates a BST" | The bound comes from **all** ancestors (§2) |
| "A BST and a heap are the same kind of order" | A heap orders parent vs children (vertical); a BST orders left vs right (horizontal) |
| "Inorder successor is always the right child" | It's the **leftmost** node of the right subtree; with no right subtree, it's the nearest ancestor you reach by going up from a left child |

---

## 1. Attempt first ★

> **[Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) (20 minutes).** Return whether a binary tree is a valid BST, with strict inequalities. Before you code: is checking `left.val < node.val < right.val` at every node enough?

---

## 2. The invariant is about whole subtrees: passing bounds down

### Worked S1 · [Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)

**The tempting check, and why it's wrong.** "Every node is larger than its left child and smaller than its right child" passes this tree:

```text
        5
       / \
      4   6
         / \
        3   7
```

But 3 is in the **right subtree of 5**, so it must be greater than 5. The invariant constrains a node by **all of its ancestors**, not just its parent.

**The fix: pass the allowed interval down** (Ch 19's top-down flow). The root may be anything. A left child must stay below its parent's value, *and* inside whatever bounds the parent had. A right child must stay above it.

```java
public boolean isValidBST(TreeNode root) {
    return within(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

// Every value in node's subtree must lie strictly between lo and hi.
private boolean within(TreeNode node, long lo, long hi) {
    if (node == null) return true;
    if (node.val <= lo || node.val >= hi) return false;
    return within(node.left, lo, node.val) && within(node.right, node.val, hi);
}
```

**Why `long` bounds:** node values can be `Integer.MIN_VALUE` or `MAX_VALUE`. With `int` bounds, a tree containing `Integer.MAX_VALUE` would be wrongly rejected. Using `long` (or nullable `Integer` bounds) gives true "unbounded" sentinels.

**An equivalent check:** an inorder traversal must be **strictly increasing**, using §0.1's fact. Keep the previously visited value and compare. Both are O(n).

---

## 3. Inorder = sorted

### Worked S2 · [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)

> Return the k-th smallest value.

Inorder visits values in sorted order, so **stop at the k-th visit**. The iterative inorder traversal (Ch 19 practice) makes stopping early easy:

```java
public int kthSmallest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode node = root;
    while (true) {
        while (node != null) {             // go as far left as possible
            stack.push(node);
            node = node.left;
        }
        node = stack.pop();                // the next value in sorted order
        if (--k == 0) return node.val;
        node = node.right;
    }
}
```

That's O(h + k): no need to visit the whole tree.

**The follow-up interviewers ask:** *the BST changes often and k-th queries are frequent. Can you do better than O(h + k) per query?* **Augment each node with the size of its subtree** (kept up to date on insert and delete). Then you can steer directly to the k-th value:

```java
// size(x) = number of nodes in x's subtree (0 for null), maintained on every insert/delete
int kth(Node node, int k) {
    int leftSize = size(node.left);
    if (k <= leftSize) return kth(node.left, k);         // it's among the smaller values
    if (k == leftSize + 1) return node.val;              // exactly leftSize values are smaller
    return kth(node.right, k - leftSize - 1);            // skip the left subtree and this node
}
```

That's O(h) per query. Java's `TreeMap` doesn't offer rank queries. When you need "the k-th smallest" or "how many are below x" on a changing set, the standard interview answer is a **Fenwick tree over compressed values** (Ch 34).

---

## 4. Changing the structure

### Worked S3 · [Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/)

> Delete the node with value `key` (if present), keep it a valid BST, and return the root.

**Contract:** `deleteNode(node, key)` deletes key from node's subtree and **returns the subtree's new root**. Returning the root is what lets a parent reattach a changed subtree with `node.left = deleteNode(node.left, key)`. When the node to delete is found, there are three cases:

* **No children:** return `null`.
* **One child:** return that child, which splices it into the parent's place. Everything in that subtree still satisfies the parent's bounds.
* **Two children:** replace the node's value with its **successor**, the smallest value in its right subtree. The successor is larger than everything on the left (it came from the right side) and smaller than everything else on the right (it's the minimum there), so it fits exactly where the deleted value was. Then delete the successor from the right subtree. It has no left child, so that's one of the easy cases.

The three cases on the §0 tree:

```text
(a) delete 13, a leaf: just remove it.

      10                      10
        \                       \
         14         →            14
        /
      13

(b) delete 10, which has one child: its child's subtree takes its place.

        8                       8
      /   \                   /   \
     3     10       →        3     14
             \                    /
              14                13
             /
           13

(c) delete 3, which has two children: its successor is 4 (the leftmost node of 3's right subtree).
    Copy 4 into 3's node, then delete the old 4 from the right subtree (case (a) here).

        8                       8
       /                       /
      3                       4
     / \           →         / \
    1   6                   1   6
       / \                       \
      4   7                       7
```

```java
public TreeNode deleteNode(TreeNode root, int key) {
    if (root == null) return null;
    if (key < root.val) root.left = deleteNode(root.left, key);
    else if (key > root.val) root.right = deleteNode(root.right, key);
    else {
        if (root.left == null) return root.right;           // zero or one child
        if (root.right == null) return root.left;
        TreeNode successor = root.right;                     // two children: the minimum of the right subtree
        while (successor.left != null) successor = successor.left;
        root.val = successor.val;
        root.right = deleteNode(root.right, successor.val);  // it has no left child: an easy case
    }
    return root;
}
```

O(h).

---

## 5. Using the order to steer

### Worked S4 · [Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

> Return the LCA of nodes p and q in a BST.

Ch 19's LCA searched both subtrees in O(n). **A BST tells you which side to go.** If both values are smaller than the current node, the LCA is in the left subtree; if both are larger, it's on the right. Otherwise the two values **split** here, or one of them *is* this node, so this is the LCA.

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    TreeNode node = root;
    while (true) {
        if (p.val < node.val && q.val < node.val) node = node.left;
        else if (p.val > node.val && q.val > node.val) node = node.right;
        else return node;                       // they split here (or one of them is this node)
    }
}
```

O(h), with no recursion and no extra space. **Using the order to steer is the BST's whole point.** Two Sum IV and Construct BST from Preorder (practice) both rely on it.

---

## 6. Tries: trees of prefixes

### 6.1 From zero

A **trie** stores a set of strings so that **all strings sharing a prefix share a path from the root**. Each node represents one prefix. It has one child per possible next character (an array of 26 for lowercase letters), and a flag saying whether the prefix it represents is a complete word.

```text
words: "car", "cat", "do"

            (root)
           /      \
         c          d
         |          |
         a          o*        * = end of a word
        / \
      r*   t*
```

* **Insert, search and "starts with" all take O(L)** for a string of length L, *independent of how many words are stored*. A hash set can do "is this exact word present?" in O(L) too, but it **can't** answer "does any word start with this prefix?".
* **Memory:** up to one node per character inserted, each with 26 references. That's fine for dictionaries of moderate size. For large alphabets, use a `HashMap<Character, Node>` per node instead of an array.

### Worked S5a · [Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/)

```java
class Trie {
    private static final class Node {
        final Node[] child = new Node[26];
        boolean isWord;
    }
    private final Node root = new Node();

    public void insert(String word) {
        Node node = root;
        for (int i = 0; i < word.length(); i++) {
            int c = word.charAt(i) - 'a';
            if (node.child[c] == null) node.child[c] = new Node();
            node = node.child[c];
        }
        node.isWord = true;
    }

    public boolean search(String word) {
        Node node = walk(word);
        return node != null && node.isWord;
    }

    public boolean startsWith(String prefix) {
        return walk(prefix) != null;
    }

    // The node for string s, or null if no stored word has s as a prefix.
    private Node walk(String s) {
        Node node = root;
        for (int i = 0; i < s.length() && node != null; i++) node = node.child[s.charAt(i) - 'a'];
        return node;
    }
}
```

### Worked S5b · [Word Search II](https://leetcode.com/problems/word-search-ii/)

> Given a letter grid and a list of up to 3·10⁴ words, return every word that can be traced in the grid (Ch 6 Word Search's rules).

Running Ch 6's Word Search once per word repeats the same grid exploration thousands of times: words sharing a prefix re-explore the same paths. **Search for all the words at once.** Put them in a trie, and run *one* backtracking DFS from each cell, **walking the trie in step with the path**. When the current path's prefix has no trie node, **no word can continue this way**, and the whole branch is pruned for every word at once.

```java
public List<String> findWords(char[][] board, String[] words) {
    Node root = new Node();
    for (String w : words) {                         // build the trie; each word's last node stores the word
        Node node = root;
        for (char ch : w.toCharArray()) {
            int c = ch - 'a';
            if (node.child[c] == null) node.child[c] = new Node();
            node = node.child[c];
        }
        node.word = w;
    }
    List<String> found = new ArrayList<>();
    for (int r = 0; r < board.length; r++)
        for (int c = 0; c < board[0].length; c++) dfs(board, r, c, root, found);
    return found;
}

private static final class Node {
    final Node[] child = new Node[26];
    String word;                                     // non-null exactly when a word ends here
}

// `parent` is the trie node for the letters of the path so far (excluding board[r][c]).
private void dfs(char[][] board, int r, int c, Node parent, List<String> found) {
    if (r < 0 || r >= board.length || c < 0 || c >= board[0].length) return;
    char ch = board[r][c];
    if (ch == '#') return;                           // already on the current path
    Node node = parent.child[ch - 'a'];
    if (node == null) return;                        // no word continues with this letter: prune
    if (node.word != null) {
        found.add(node.word);
        node.word = null;                            // report each word only once
    }
    board[r][c] = '#';                               // choose (Ch 6 §2)
    dfs(board, r + 1, c, node, found);
    dfs(board, r - 1, c, node, found);
    dfs(board, r, c + 1, node, found);
    dfs(board, r, c - 1, node, found);
    board[r][c] = ch;                                // un-choose
}
```

Setting `node.word = null` after reporting prevents duplicates, and it's also a cheap pruning step. (Removing trie nodes that have run out of words prunes even more, and is a common follow-up.)

---

## 7. Binary tries: maximum XOR

### Worked S6 · [Maximum XOR of Two Numbers in an Array (↺ Ch 17)](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array-ch-17/)

Ch 17 solved this greedily from the high bit, using a hash set of prefixes for each bit. A **binary trie** stores each number's bits from the highest to the lowest, so each number is a root-to-leaf path with two children (0 and 1) at every level. Then, for a given x, **walk down choosing the *opposite* bit whenever that child exists**. Each opposite choice makes that XOR bit 1, and higher bits dominate (Ch 17 §5), so taking the opposite branch whenever possible gives the best partner.

With 3-bit numbers {2 = `010`, 5 = `101`, 7 = `111`}, and the query x = 2 = `010`:

```text
                 root                bit 2: x has 0, want 1 → go right (exists)
              0/      \1             bit 1: x has 1, want 0 → go left  (exists: 5's path)
              •         •            bit 0: x has 0, want 1 → go right (exists)
             1|       0/  \1         reached 5 = 101:  2 XOR 5 = 010 XOR 101 = 111 = 7
              •       •     •
             0|      1|     |1       Going towards 7 instead would give 010 XOR 111 = 101 = 5.
              2       5     7
```

```java
public int findMaximumXOR(int[] nums) {
    int[][] child = new int[2][nums.length * 31 + 1];    // child[b][node]: two big arrays, not millions of small ones
    int nodes = 1, best = 0;                              // node 0 is the root; 0 also means "no child"
    for (int x : nums) {
        int node = 0;                                     // insert x, from bit 30 down
        for (int bit = 30; bit >= 0; bit--) {
            int b = (x >> bit) & 1;
            if (child[b][node] == 0) child[b][node] = nodes++;
            node = child[b][node];
        }
        node = 0;                                         // query: the best partner for x so far
        int xor = 0;
        for (int bit = 30; bit >= 0; bit--) {
            int b = (x >> bit) & 1;
            if (child[1 - b][node] != 0) {                // the opposite bit exists: this XOR bit becomes 1
                xor |= 1 << bit;
                node = child[1 - b][node];
            } else {
                node = child[b][node];
            }
        }
        best = Math.max(best, xor);
    }
    return best;
}
```

That's O(31 · n). Storing the trie in arrays rather than `Node` objects is much faster and lighter in Java when there are millions of nodes, **but only with the dimensions this way round.** `new int[2][size]` is two large arrays. `new int[size][2]` would be `size` separate two-element arrays, each a heap object with its own header (Ch 3 §7): for n = 2·10⁵ that's 6·10⁶ objects and well over 100 MB, worse than the `Node` version. With **counts** stored in each node, a binary trie also supports deletion, and counting the pairs with XOR below a bound. With queries **sorted offline**, it answers "maximum XOR with an element ≤ m" (stretch).

---

## 8. Where these break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Inserting sorted data into a plain BST | Degenerates into a chain: O(n) per operation | `TreeMap` (balanced) |
| You need rank, or "count below x", on a changing set | Plain BSTs and `TreeMap` don't track subtree sizes | An augmented BST, or a Fenwick tree over compressed values (Ch 34) |
| A trie over a huge alphabet, or long strings | 26 references per node becomes a lot of memory | `HashMap` children, or a hash set of strings when prefixes aren't needed |
| Removing words from a trie | Shared prefixes mustn't be freed | Store pass-through counts; decrement on removal |
| Validating with parent–child comparisons only | Misses violations against ancestors | Pass the bounds down (§2) |

---

## 9. Recognition

**Signals:**
* **"BST" in the statement** → use the order: steer, use inorder = sorted, pass bounds down.
* **"k-th smallest", "successor", "closest value"** in a BST → inorder, or steering.
* **Prefixes**: autocomplete, "starts with", "replace with the shortest root", "search suggestions" → **trie**.
* **Many words searched at once** (in a grid, a stream, a sentence) → **trie + one search**.
* **"Maximum XOR"**, or XOR thresholds over pairs → **binary trie**.

**False friends:** "a sorted dynamic set" with only floor/ceiling queries just needs `TreeMap` (Ch 4 §22), not a hand-written BST.

---

## 10. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Validating only against the parent | Pass `(lo, hi)` bounds down |
| `int` bounds when values reach `Integer.MAX_VALUE` | `long` bounds, or nullable bounds |
| Losing the subtree after deleting | Assign the result: `node.left = deleteNode(node.left, key)` |
| A trie over `char − 'a'` with uppercase or other characters in the input | Check the alphabet; size the array accordingly |
| Reporting the same word twice in Word Search II | Clear the stored word after reporting it |
| Using `Node` objects for a binary trie with 10⁶+ nodes | Array-backed nodes |

---

## 11. Practice

1. **[Insert into a Binary Search Tree](https://leetcode.com/problems/insert-into-a-binary-search-tree/)** *(Medium)*
2. **[Convert Sorted Array to Binary Search Tree](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/)** *(Easy)*
3. **[Two Sum IV – Input is a BST](https://leetcode.com/problems/two-sum-iv-input-is-a-bst/)** *(Easy)*
4. **[Binary Search Tree Iterator](https://leetcode.com/problems/binary-search-tree-iterator/)** *(Medium)*
5. **[Construct Binary Search Tree from Preorder Traversal](https://leetcode.com/problems/construct-binary-search-tree-from-preorder-traversal/)** *(Medium)*
6. **[Recover Binary Search Tree](https://leetcode.com/problems/recover-binary-search-tree/)** *(Medium)*
7. **[Replace Words](https://leetcode.com/problems/replace-words/)** *(Medium)*
8. **[Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/)** *(Medium)*
9. **[Search Suggestions System](https://leetcode.com/problems/search-suggestions-system/)** *(Medium)*

**Stretch:** **[Maximum XOR With an Element From Array](https://leetcode.com/problems/maximum-xor-with-an-element-from-array/)** *(Hard)* · **[Concatenated Words](https://leetcode.com/problems/concatenated-words/)** *(Hard)* · **[Palindrome Pairs](https://leetcode.com/problems/palindrome-pairs/)** *(Hard)*

<details><summary>Nudges</summary>

1. Insert: search for the value; attach a new node where the search reaches `null`.
2. Sorted Array to BST: the middle element is the root, and each half recursively becomes a subtree. Why is the result balanced?
3. Two Sum IV: inorder gives a sorted list; then two pointers (Ch 11). Or a hash set during any traversal.
4. BST Iterator: the iterative inorder stack, spread across calls. Why is `next()` amortised O(1)?
5. From Preorder: pass the bounds down (S1) and consume preorder with a moving index; each value either fits the current bounds or belongs higher up.
6. Recover: inorder of a BST with two values swapped has one or two "descents". Swap the first descent's larger value with the last descent's smaller value.
7. Replace Words: put the roots in a trie; for each word, walk until you hit the first root that ends a word.
8. Add and Search: a trie; for `.`, try every child (backtracking).
9. Suggestions: sort the products, then binary-search each prefix's position (Ch 10) and take the next three that match. Or a trie storing up to three words per node.
10. Stretch, Max XOR With Element: sort the queries by their limit m and the numbers ascending; insert numbers ≤ m into a binary trie, then query.
11. Stretch, Concatenated: sort the words by length; for each word, check whether it splits into earlier (shorter) words, using a trie plus a memoised recursion over positions (Ch 6 §0.3's "remember each answer").
12. Stretch, Palindrome Pairs: for each word, consider every split into a prefix and a suffix. If one part is a palindrome, look up the reverse of the other part, using a map or a trie of the reversed words.
</details>

<details><summary>Key</summary>

Insert → S4 steering · Sorted Array → middle as root (balance by construction) · Two Sum IV → S2 + two pointers or hashing · BST Iterator → S2 with a stack, amortised O(1) · From Preorder → S1 bounds · Recover → S2, find the two inversions · Replace Words → S5 · Add and Search → S5 + wildcard DFS · Suggestions → sort + binary search, or S5 · Max XOR With Element → offline sort + S6 · Concatenated → S5 + memoised splitting · Palindrome Pairs → S5 / hashing on reversed words
</details>

---

## 12. Self-test

<details><summary>1. Why is an inorder traversal of a BST sorted?</summary>

By induction: it visits the whole left subtree (sorted, and all smaller than the root), then the root, then the whole right subtree (sorted, and all larger).
</details>

<details><summary>2. Give a tree that passes the parent–child check but isn't a BST.</summary>

Root 5, right child 6, and 6's left child 3. Every parent–child pair looks fine, but 3 is in 5's right subtree and is smaller than 5.
</details>

<details><summary>3. When deleting a node with two children, why is its successor a valid replacement?</summary>

The successor comes from the right subtree, so it's larger than everything on the left. It's the minimum of the right subtree, so it's smaller than everything else there. It fits exactly where the deleted value was.
</details>

<details><summary>4. What can a trie do that a HashSet of strings can't do efficiently?</summary>

Prefix queries: "does any word start with p?", "list the words with prefix p", "find the shortest stored prefix of this word". All of them take O(|p|).
</details>

<details><summary>5. In the binary trie, why does taking the opposite bit whenever possible give the maximum XOR?</summary>

Each opposite choice sets that XOR bit to 1, and a higher bit is worth more than all lower bits combined (Ch 17 §5). So the greedy choice at each level, from the top down, is optimal.
</details>

---

## 13. Exit gate

Move on when you can:

* state the BST invariant precisely, and explain why the parent–child check is wrong;
* write Validate BST (with bounds), Delete Node and the iterative k-th smallest from blank files;
* implement a trie and Word Search II from blank files, each in under 15 minutes;
* write the array-backed binary trie for maximum XOR;
* solve 7 of the 9 practice problems.
