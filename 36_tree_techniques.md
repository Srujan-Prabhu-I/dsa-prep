# Chapter 36 — Tree Techniques

> **Part VII · Week 32** · **S-tier** · Prerequisites: Ch 4 §16 (recursion depth), Ch 17 (binary representation), Ch 19 (Worked T3: LCA for one query), Ch 21 (BFS on adjacency lists), Ch 31 (BFS order on trees), Ch 34 (Fenwick)
> **Problems:** 5 worked (1 ★) · 5 practice · **Time:** about 7 hours

Ch 19 answered **one** question about a tree in O(n): the LCA of two nodes, a path sum, a subtree total. This chapter is about **many** questions (q up to 2·10⁵) on the same tree, where O(n) each is 4·10¹⁰. Two ideas cover almost everything interviews and OAs ask here:
* **Binary lifting:** precompute jumps of length 1, 2, 4, 8, … up the tree, so that any upward walk takes O(log n) jumps. That gives the k-th ancestor, the LCA, distances and path aggregates.
* **Euler tour flattening:** number the nodes in DFS order, so that **every subtree is a contiguous range**. Subtree queries become Ch 34's range queries.

**Recursion depth:** these trees can be paths of 2·10⁵ nodes, which overflows Java's default stack (Ch 4 §16). Every traversal here is iterative: BFS order (Ch 31) or an explicit stack.

---

## 0. From zero: jumping instead of walking

To find the k-th ancestor of v, the obvious method follows `parent` k times: O(k), up to O(n) on a path-shaped tree. Over 10⁵ queries that's 10¹⁰.

**Write k in binary (Ch 17).** k = 13 = 8 + 4 + 1, so "go up 13" is "go up 8, then 4, then 1". If every node knew its ancestors at distances **1, 2, 4, 8, …**, any k would take at most log₂ k jumps.

**Building the table.** Let `up[j][v]` be the 2ʲ-th ancestor of v. Going up 2ʲ is going up 2ʲ⁻¹ **twice**:

> `up[0][v] = parent[v]`,  `up[j][v] = up[j − 1][ up[j − 1][v] ]`

Each level is computed from the level below, so fill level by level: O(n log n) time and memory. With n = 2·10⁵ and 18 levels, that's 3.6·10⁶ ints, about 14 MB.

**Past the root.** Decide what a jump beyond the root returns. Use −1 when the question is "does the ancestor exist" (L1), or the root itself, so that jumps saturate there (L2, which makes the LCA code simpler).

**A worked table**, using the saturating convention (a jump past the root stays at the root 0):

```text
            0                 depth 0        v       0  1  2  3  4  5  6  7  8
          /   \                              depth   0  1  1  2  2  2  3  3  4
         1     2              depth 1        up[0]   0  0  0  1  2  2  3  5  6    parent
         |    / \                            up[1]   0  0  0  0  0  0  1  2  3    2 steps up
         3   4   5            depth 2        up[2]   0  0  0  0  0  0  0  0  0    4 steps up
         |       |
         6       7            depth 3        e.g. up[1][8] = up[0][ up[0][8] ] = up[0][6] = 3
         |
         8                    depth 4
```

**The 3rd ancestor of 8:** 3 = `011`₂, so take the jumps for bit 0 (1 step) and bit 1 (2 steps): `up[0][8] = 6`, then `up[1][6] = 1`. Check by walking: 8 → 6 → 3 → 1. ✓ Two jumps instead of three steps; for k near 10⁵, about 17 jumps instead of 10⁵ steps.

**Common misconceptions:**

| Belief | Reality |
| :--- | :--- |
| "The jumps must be taken largest first" | For the k-th ancestor, any order of the set bits works; only the LCA walk needs largest-first |
| "The table can be filled node by node in any order" | It must be filled **level by level**: `up[j]` needs all of `up[j − 1]` |
| "The LCA is found by lifting both nodes immediately" | First equalise the depths; otherwise "different ancestors" doesn't mean "below the LCA" |
| "Binary lifting handles updates to the tree" | The table is static; a changed parent means rebuilding |

---

## 1. Attempt first ★

> **[Kth Ancestor of a Tree Node](https://leetcode.com/problems/kth-ancestor-of-a-tree-node/) (30 minutes).** n ≤ 5·10⁴ nodes given by `parent[]` (with `parent[0] = −1`), and up to 5·10⁴ calls to `getKthAncestor(node, k)`, which returns −1 if there's no such ancestor. Make each call O(log n).

---

## 2. Binary lifting

### Worked L1 · [Kth Ancestor of a Tree Node](https://leetcode.com/problems/kth-ancestor-of-a-tree-node/) ★

```java
class TreeAncestor {
    private final int[][] up;                         // up[j][v] = the 2^j-th ancestor of v, or -1

    public TreeAncestor(int n, int[] parent) {
        int levels = 32 - Integer.numberOfLeadingZeros(n);       // 2^levels > n > any depth
        up = new int[levels][];
        up[0] = parent.clone();
        for (int j = 1; j < levels; j++) {
            up[j] = new int[n];
            for (int v = 0; v < n; v++) {
                int half = up[j - 1][v];
                up[j][v] = half < 0 ? -1 : up[j - 1][half];
            }
        }
    }

    public int getKthAncestor(int node, int k) {
        if (k >= up[0].length) return -1;              // deeper than any node can be
        for (int j = 0; k > 0 && node >= 0; j++, k >>= 1)
            if ((k & 1) == 1) node = up[j][node];      // take the 2^j jump if bit j of k is set
        return node;
    }
}
```

**Why the order of jumps doesn't matter:** going up a then b lands in the same place as b then a; both are a + b steps up the same root path. **Why the table doesn't need any node order:** level j reads only level j − 1, which is complete. That's why `parent[]` can list nodes in any order. **O(n log n) build, O(log n) per query.**

**Alternative, in prose:** do one DFS that records each node's root path on a stack, and answer the query as `stack[depth − k]`. That works offline (all queries known in advance) in O(n + q).

### Worked L2a · [CSES Company Queries II (LCA)](https://cses.fi/problemset/task/1688)

> n employees (n, q ≤ 2·10⁵); employee 1 is the general director, and each other employee's boss is given. For each query (a, b), print their lowest common boss: the deepest employee who is a boss-ancestor of both (an employee counts as their own ancestor).

**From Ch 19:** the one-query LCA searched both subtrees, O(n). **With lifting:**
1. **Equalise the depths.** Lift the deeper node by the depth difference, using L1's method. If the two nodes are now equal, that node is the LCA (one was an ancestor of the other).
2. **Lift both together**, from the largest jump to the smallest, taking a jump only if it lands them on **different** nodes.
3. They end as two different children of the LCA, so the answer is their parent.

**Why step 2 works.** Let d be the number of steps from a (and b) up to the LCA, so d ≥ 1. A jump keeps them apart exactly when it is **shorter** than their current distance to the LCA. Trying the jumps from the largest down and taking each one that keeps them apart adds up to exactly d − 1: it's d − 1 written in binary, the same reasoning as binary search on bits. So they stop one step below the LCA.

**On the §0 tree, lca(8, 7):**

```text
step 1   depth 4 vs 3: lift 8 by 1 → 6.                      a = 6, b = 7 (both depth 3, different)
step 2   j = 2: up[2][6] = 0, up[2][7] = 0: same → skip      (a jump of 4 would overshoot)
         j = 1: up[1][6] = 1, up[1][7] = 2: different → take  a = 1, b = 2
         j = 0: up[0][1] = 0, up[0][2] = 0: same → skip
step 3   answer = up[0][a] = up[0][1] = 0                     8 and 7 meet only at the root
```

One class serves L2 and L3. It uses the **saturating** convention (the root's parent is the root):

```java
static final class Lca {
    final int[][] up;
    final int[] depth;

    Lca(int[] parent, int[] depth) {                   // parent[root] == root
        int n = parent.length, levels = 32 - Integer.numberOfLeadingZeros(n);
        this.depth = depth;
        up = new int[levels][];
        up[0] = parent;
        for (int j = 1; j < levels; j++) {
            up[j] = new int[n];
            for (int v = 0; v < n; v++) up[j][v] = up[j - 1][up[j - 1][v]];
        }
    }

    int lift(int v, int k) {                           // the k-th ancestor; k <= depth[v]
        for (int j = 0; k > 0; j++, k >>= 1) if ((k & 1) == 1) v = up[j][v];
        return v;
    }

    int lca(int a, int b) {
        if (depth[a] < depth[b]) { int t = a; a = b; b = t; }
        a = lift(a, depth[a] - depth[b]);              // step 1: same depth
        if (a == b) return a;
        for (int j = up.length - 1; j >= 0; j--)       // step 2: the largest jumps that keep them apart
            if (up[j][a] != up[j][b]) { a = up[j][a]; b = up[j][b]; }
        return up[0][a];                               // step 3
    }
}
```

**Parents and depths without recursion:** BFS from the root over an undirected adjacency list (Ch 31's BFS order): the node a vertex is first reached from is its parent, and its depth is one more than that parent's.

```java
// Fills parent[] and depth[] by BFS from node 0 (parent[0] = 0).
static void rootAt0(List<List<Integer>> adj, int[] parent, int[] depth) {
    int n = adj.size();
    int[] queue = new int[n];
    boolean[] seen = new boolean[n];
    seen[0] = true;
    for (int head = 0, tail = 1; head < tail; head++) {
        int u = queue[head];
        for (int v : adj.get(u)) {
            if (seen[v]) continue;
            seen[v] = true;
            parent[v] = u;
            depth[v] = depth[u] + 1;
            queue[tail++] = v;
        }
    }
}
```

For Company Queries II, add each (employee, boss) pair as an undirected edge, convert to 0-based, call `rootAt0`, build the `Lca`, and print `lca(a − 1, b − 1) + 1` for each query. Use FastReader and a StringBuilder (Ch 4).

### Worked L2b · [CSES Distance Queries](https://cses.fi/problemset/task/1135)

> A tree with n nodes (n, q ≤ 2·10⁵). For each query (a, b), print the number of edges on the path between them.

**The path from a to b goes up to their LCA, then down**, so

> `dist(a, b) = depth[a] + depth[b] − 2 · depth[lca(a, b)]`

The `depth[lca]` term is subtracted twice because the part of the root path above the LCA is shared by both nodes' depths.

```java
public static void main(String[] args) throws IOException {
    FastReader in = new FastReader(System.in);
    int n = in.nextInt(), q = in.nextInt();
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int i = 0; i < n - 1; i++) {
        int a = in.nextInt() - 1, b = in.nextInt() - 1;
        adj.get(a).add(b);
        adj.get(b).add(a);
    }
    int[] parent = new int[n], depth = new int[n];
    rootAt0(adj, parent, depth);
    Lca lca = new Lca(parent, depth);
    StringBuilder out = new StringBuilder();
    while (q-- > 0) {
        int a = in.nextInt() - 1, b = in.nextInt() - 1;
        out.append(depth[a] + depth[b] - 2 * depth[lca.lca(a, b)]).append('\n');
    }
    System.out.print(out);
}
```

**The same identity works for any invertible path aggregate**: keep the value from the root to each node, and combine `value[a] + value[b] − 2 · value[lca]`. Weighted distances are an example (add the LCA's own value back once if nodes, not edges, carry the weights). **Alternative LCA methods, in prose:** the Euler tour plus a sparse table (Ch 34 §7) answers each query in O(1) after an O(n log n) build. Tarjan's offline LCA uses DSU (Ch 24). Lifting is the easiest to write correctly.

### Worked L3 · [Minimum Edge Weight Equilibrium Queries in a Tree](https://leetcode.com/problems/minimum-edge-weight-equilibrium-queries-in-a-tree/)

> A tree with n ≤ 10⁴ nodes, edge weights in 1..26. For each query (a, b), return the fewest edge-weight changes that make every edge on the a–b path equal. q ≤ 2·10⁴.

**The answer for one path:** keep the most common weight and change everything else, so it's (path length) − (the count of the most common weight on the path). With only 26 weights, we need **the count of each weight on the path**.

**Counts are invertible**, so use the root-prefix identity of L2b with one counter per weight: `cnt[v][w]` = the number of weight-w edges between the root and v. Then the count on the a–b path is `cnt[a][w] + cnt[b][w] − 2·cnt[l][w]` with l = lca(a, b).

```java
public int[] minOperationsQueries(int n, int[][] edges, int[][] queries) {
    List<List<int[]>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int[] e : edges) {
        adj.get(e[0]).add(new int[] {e[1], e[2]});
        adj.get(e[1]).add(new int[] {e[0], e[2]});
    }
    int[] parent = new int[n], depth = new int[n], queue = new int[n];
    int[][] cnt = new int[n][27];                      // cnt[v][w]: weight-w edges on the root-to-v path
    boolean[] seen = new boolean[n];
    seen[0] = true;
    for (int head = 0, tail = 1; head < tail; head++) {
        int u = queue[head];
        for (int[] e : adj.get(u)) {
            int v = e[0];
            if (seen[v]) continue;
            seen[v] = true;
            parent[v] = u;
            depth[v] = depth[u] + 1;
            cnt[v] = cnt[u].clone();
            cnt[v][e[1]]++;
            queue[tail++] = v;
        }
    }
    Lca lca = new Lca(parent, depth);
    int[] answer = new int[queries.length];
    for (int i = 0; i < queries.length; i++) {
        int a = queries[i][0], b = queries[i][1], l = lca.lca(a, b), most = 0;
        for (int w = 1; w <= 26; w++) most = Math.max(most, cnt[a][w] + cnt[b][w] - 2 * cnt[l][w]);
        answer[i] = depth[a] + depth[b] - 2 * depth[l] - most;
    }
    return answer;
}
```

O((n + q)·(26 + log n)). **When the aggregate isn't invertible** (the **max** edge weight on a path, say), the root-prefix trick fails, because you can't subtract a max. Instead, **lift the aggregate alongside the jump**: `best[j][v]` = the max edge weight over the 2ʲ edges above v, with `best[j][v] = max(best[j−1][v], best[j−1][up[j−1][v]])`. Then collect the max over every jump the LCA walk takes, on both sides. That's the general "path aggregates by lifting" pattern.

---

## 3. Euler tour: subtrees become ranges

**The idea.** Number the nodes in **DFS preorder** (a node, then everything below it). A DFS finishes all of v's subtree before it leaves v, so the subtree of v receives **consecutive** numbers:

> `subtree(v)` = the nodes numbered `tin[v] .. tin[v] + size[v] − 1`

On the §0 tree, a preorder that visits children left to right numbers the nodes like this:

```text
preorder:   0  1  3  6  8  2  4  5  7
tin:        0  1  2  3  4  5  6  7  8          (tin[v] = v's position in this order)

subtree(1) = {1, 3, 6, 8}:  tin[1] = 1, size 4 → positions 1..4   ✓ contiguous
subtree(2) = {2, 4, 5, 7}:  tin[2] = 5, size 4 → positions 5..8   ✓ contiguous
subtree(5) = {5, 7}:        tin[5] = 7, size 2 → positions 7..8
```

Any DFS order works, including the stack-based one below, which visits children in reverse; only "all of v's subtree before leaving v" matters.

A subtree sum is then a range sum, and a node update is a point update: exactly Ch 34's Fenwick tree.

**Preorder without recursion.** Pop a node from an explicit stack, number it, and push its children. A node's descendants are all pushed above everything that was already on the stack, so they're all numbered before the stack returns below them: the subtree stays contiguous. Subtree sizes come from the reverse of that order, as in Ch 31.

### Worked L4 · [CSES Subtree Queries](https://cses.fi/problemset/task/1137)

> A rooted tree (root 1) with a value at each node; n, q ≤ 2·10⁵. Queries: `1 s x` sets node s's value to x; `2 s` prints the sum of the values in s's subtree.

```java
public static void main(String[] args) throws IOException {
    FastReader in = new FastReader(System.in);
    int n = in.nextInt(), q = in.nextInt();
    long[] value = new long[n];
    for (int i = 0; i < n; i++) value[i] = in.nextInt();
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int i = 0; i < n - 1; i++) {
        int a = in.nextInt() - 1, b = in.nextInt() - 1;
        adj.get(a).add(b);
        adj.get(b).add(a);
    }

    int[] tin = new int[n], size = new int[n], order = new int[n], parent = new int[n];
    int[] stack = new int[n];
    int top = 0, timer = 0;
    stack[top++] = 0;
    parent[0] = -1;
    while (top > 0) {                                  // iterative preorder
        int u = stack[--top];
        tin[u] = timer;
        order[timer++] = u;
        for (int v : adj.get(u)) if (v != parent[u]) { parent[v] = u; stack[top++] = v; }
    }
    for (int i = n - 1; i >= 0; i--) {                 // children are numbered after their parent
        int u = order[i];
        size[u] += 1;
        if (parent[u] >= 0) size[parent[u]] += size[u];
    }

    Fenwick sums = new Fenwick(n);                     // Ch 34 §3, 1-based: node u sits at tin[u] + 1
    for (int u = 0; u < n; u++) sums.add(tin[u] + 1, value[u]);
    StringBuilder out = new StringBuilder();
    while (q-- > 0) {
        if (in.nextInt() == 1) {
            int s = in.nextInt() - 1;
            long x = in.nextInt();
            sums.add(tin[s] + 1, x - value[s]);
            value[s] = x;
        } else {
            int s = in.nextInt() - 1;
            out.append(sums.prefix(tin[s] + size[s]) - sums.prefix(tin[s])).append('\n');
        }
    }
    System.out.print(out);
}
```

O((n + q) log n). **Each node is pushed at most once**, so the stack never holds more than n entries. **Sums need `long`:** 2·10⁵ values up to 10⁹.

**The other direction.** If an update applies to a whole subtree and the query asks about one node, it's a range update with a point query: a Fenwick tree over the **difference** array (Ch 34 §8). CSES Path Queries (practice) uses it: changing node s's value changes the root-path sum of every node in s's subtree.

---

## 4. Where these break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Updates **and** queries on arbitrary paths (path max with edge updates) | Lifting tables are static; the Euler tour flattens subtrees, not paths | Heavy-light decomposition: out of scope (Ch 37) |
| The tree itself changes (links and cuts) | Every table must be rebuilt | Link-cut trees: out of scope |
| "Count paths with property X" over all pairs | Pairs are Θ(n²) | Centroid decomposition: out of scope (Ch 37); for small properties, a tree DP (Ch 31) |
| Memory with n = 10⁶ | n log n ints ≈ 80 MB | The Euler tour + sparse table, or offline Tarjan LCA |
| Recursive DFS on a 2·10⁵-node path | `StackOverflowError` | BFS order or an explicit stack, or a big-stack thread (Ch 4 §16) |

---

## 5. Recognition

**Signals:**
* A **static** tree with **many** queries (10⁴–2·10⁵) about ancestors, LCAs, distances or path summaries: binary lifting.
* "The k-th ancestor", "after k steps", with k up to 10⁹ in a **functional graph** (each node has exactly one successor): lifting works on any graph where each node has one "next", not just trees.
* Subtree sums or counts **with updates**: Euler tour + Fenwick.
* "Mark every node on many paths, then report counts": a difference on the tree (+1 at a and b, −1 at the LCA and the LCA's parent), then subtree sums.

**False friends:**
* **One** LCA query: Ch 19's O(n) recursion is simpler.
* LCA in a **BST**: steer by values (Ch 20).
* Sums over all subtrees, **without updates**: one bottom-up pass (Ch 31).

---

## 6. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Too few lifting levels | `levels = 32 − numberOfLeadingZeros(n)` guarantees 2^levels > n |
| Mixing the −1 and the saturating conventions | Choose one per class; with −1, guard every `up[j][-1]` |
| Lifting both nodes before equalising depths | Equalise first, then lift together |
| Recursion on deep trees | Iterative BFS or an explicit stack |
| Euler range off by one | The subtree is `[tin, tin + size − 1]`; with a 1-based Fenwick, sum `prefix(tin + size) − prefix(tin)` |
| Cloning a 27-int array per node at n = 2·10⁵ | Fine at 10⁴ (L3); at larger n, store counts in one flat `int[n * 27]` |

---

## 7. Practice

1. **[CSES Company Queries I](https://cses.fi/problemset/task/1687)** *(Easy; stdin/stdout)*
2. **[CSES Planets Queries I](https://cses.fi/problemset/task/1750)** *(Medium; stdin/stdout)*
3. **[Cycle Length Queries in a Tree](https://leetcode.com/problems/cycle-length-queries-in-a-tree/)** *(Hard)*
4. **[CSES Counting Paths](https://cses.fi/problemset/task/1136)** *(Medium; stdin/stdout)*
5. **[CSES Path Queries](https://cses.fi/problemset/task/1138)** *(Medium; stdin/stdout)*

<details><summary>Nudges</summary>

1. Company Queries I: the k-th boss, or −1. That's L1 directly.
2. Planets Queries I: every planet has exactly one teleport target, so `up[j][v]` is well defined for **every** node, with no −1. k ≤ 10⁹ needs 30 levels.
3. Cycle Length: the tree is a complete binary tree labelled like a heap, so the parent of v is v / 2. No table needed: repeatedly halve the **larger** label until the two meet, counting the steps. The cycle is that path plus the new edge.
4. Counting Paths: for each path (a, b), add +1 at a and b, −1 at the LCA and −1 at the LCA's parent (if it exists). Then each node's answer is the sum over its subtree, computed in reverse BFS order.
5. Path Queries: the query is the sum on the root-to-s path. Changing node s by δ changes that sum for every node in s's subtree: a range add on `[tin, tin + size − 1]` and a point query at `tin` (a Fenwick tree over the difference array).
</details>

<details><summary>Key</summary>

Company I → L1 · Planets Queries I → L1 on a functional graph · Cycle Length → LCA in an implicit heap · Counting Paths → L2 + difference on tree · Path Queries → L4 with range update / point query
</details>

---

## 8. Self-test

<details><summary>1. Why is up[j][v] = up[j − 1][up[j − 1][v]] correct, and in what order must the table be filled?</summary>

Going up 2ʲ is going up 2ʲ⁻¹ twice. Fill it level by level: level j needs only level j − 1, so the node order doesn't matter.
</details>

<details><summary>2. In the LCA walk, why take a jump only if it lands the nodes on different vertices?</summary>

A jump that lands them on the same vertex might overshoot the LCA. Taking only the "still different" jumps, largest first, adds up to exactly d − 1, which leaves both one step below the LCA.
</details>

<details><summary>3. Why does dist(a, b) subtract 2 · depth[lca]?</summary>

depth[a] + depth[b] counts the shared part of the root path, from the root down to the LCA, twice. The real path never uses it.
</details>

<details><summary>4. Why can't the root-prefix trick give the max edge weight on a path, and what replaces it?</summary>

You can't "subtract" a max. Store the max over each jump, `best[j][v]`, alongside `up[j][v]`, and combine it over the jumps of the LCA walk.
</details>

<details><summary>5. Why is a subtree a contiguous range in DFS preorder?</summary>

A DFS numbers v and then everything below v before it numbers anything outside v's subtree.
</details>

---

## 9. Exit gate

Move on when you can:

* build a lifting table and answer k-th ancestor and LCA queries from blank files in 10 minutes;
* derive the distance formula and explain when the root-prefix trick applies and when lifting aggregates are needed;
* flatten a tree with an iterative DFS and answer subtree queries with a Fenwick tree;
* solve 4 of the 5 practice problems.
