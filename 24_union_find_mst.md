# Chapter 24 — Union-Find & Minimum Spanning Trees

> **Part V · Week 21** · Prerequisites: Ch 2 (exchange arguments), Ch 3 §3 (amortisation), Ch 9 (sorting), Ch 21
> **Problems:** 4 worked (1 ★) · 8 practice · 4 stretch · **Time:** about 7 hours

Some graph questions only ever **add** connections and then ask "are these two in the same group?". Running BFS for every question costs O(V + E) each time. **Union-find** (also called a *disjoint set union*, or DSU) answers each one in **almost O(1)**. It's among the simplest data structures in the book and among the most useful: dynamic connectivity, grouping by an equivalence, cycle detection, and the heart of **Kruskal's minimum spanning tree** algorithm.

---

## 0. From zero: deriving union-find

> **The problem:** n elements, each starting in its own group. Support `union(a, b)` (merge the groups of a and b) and `connected(a, b)` (are they in the same group?), mixed in any order.

**Attempt 1: a group label for each element.** `connected` compares two labels in O(1). But `union` must relabel every member of one group, which is O(n), and O(n²) over a long run of merges.

**Attempt 2: a forest of parent pointers.** Each group is a tree; each element points to a **parent**, and the **root** (the element that points to itself) is the group's representative. `find(x)` follows parents up to the root. `union` links one root under the other, in O(1) after the two finds. `connected(a, b)` asks whether `find(a) == find(b)`. Now unions are cheap, but a bad sequence of links can build a tall chain, making each `find` O(n).

**Fix 1: union by size.** Always attach the **smaller** tree under the root of the **larger**.

> **Claim:** every tree then has height ≤ log₂ n. *Proof:* a node's depth grows by 1 only when its tree is attached under a tree **at least as large**, so the size of the tree containing it at least **doubles**. A size can double at most log₂ n times before reaching n.

**Fix 2: path compression.** After `find(x)` reaches the root, point **every node on the path directly at the root**, so later finds from those nodes take one step.

**A trace** with n = 6 and union by size (ties: the second root goes under the first), showing the `parent` array and the forest it encodes:

```text
start            parent = [0 1 2 3 4 5]    six trees of one node: every element is its own root
union(0, 1)      parent = [0 0 2 3 4 5]    1 goes under 0
union(2, 3)      parent = [0 0 2 2 4 5]    3 goes under 2
union(0, 2)      parent = [0 0 0 2 4 5]    equal sizes (2 and 2): root 2 goes under root 0
union(4, 5)      parent = [0 0 0 2 4 4]

forest:        0           4          connected(3, 1)?  find(3) = 0 and find(1) = 0: yes
             /   \         |          connected(3, 5)?  find(3) = 0, find(5) = 4: no
            1     2        5
                  |
                  3
```

**`find(3)` with path compression:** it walks 3 → 2 → 0, finds the root 0, then points 3 directly at 0:

```text
before:  0 ← 2 ← 3            parent = [0 0 0 2 4 4]
after:   0 ← 2,  0 ← 3        parent = [0 0 0 0 4 4]     the next find(3) takes one step
```

**Without union by size,** a bad but legal linking rule builds a chain. With "first root goes under second root":

```text
union(0, 1): 0 → 1        union(1, 2): root 1 → 2        union(2, 3): root 2 → 3   …
result: 0 → 1 → 2 → 3 → … → n−1, height n − 1, so find(0) walks n − 1 steps
```

Union by size refuses this: the growing chain is always the larger tree, so the single new element goes under it, not the other way round.

**Together:** a sequence of m operations costs O(m · α(n)), where α is the *inverse Ackermann* function. It grows so slowly that α(n) ≤ 4 for any n you'll ever meet. In interviews, say "**amortised almost O(1)**".

```java
final class DSU {
    private final int[] parent, size;
    private int components;

    DSU(int n) {
        parent = new int[n];
        size = new int[n];
        components = n;
        for (int i = 0; i < n; i++) { parent[i] = i; size[i] = 1; }   // n groups of one
    }

    int find(int x) {
        int root = x;
        while (parent[root] != root) root = parent[root];
        while (parent[x] != root) {          // path compression: point every node on the path at the root
            int next = parent[x];
            parent[x] = root;
            x = next;
        }
        return root;
    }

    /** Merges the groups of a and b. Returns false if they were already in the same group. */
    boolean union(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;
        if (size[ra] < size[rb]) { int t = ra; ra = rb; rb = t; }   // union by size: smaller under larger
        parent[rb] = ra;
        size[ra] += size[rb];
        components--;
        return true;
    }

    boolean connected(int a, int b) { return find(a) == find(b); }
    int sizeOf(int x) { return size[find(x)]; }
    int components() { return components; }
}
```

**Two details:** `find` is **iterative**, so a long chain (before compression) can't overflow the stack. And `union` **returns whether a merge happened**, which is how cycle detection and Kruskal's algorithm use it.

**Common misconceptions:**

| Belief | Reality |
| :--- | :--- |
| "`parent[x]` is x's group" | Only `find(x)`, the **root**, identifies the group. `parent[x]` may be any ancestor |
| "Compare `parent[a] == parent[b]` to test connectivity" | Compare `find(a) == find(b)` |
| "Union-find can split groups" | It can't. For deletions, reverse time (Ch 37 §4) or recompute |
| "`union(a, b)` makes a's root point to b" | It links the two **roots**, never the original elements |
| "Path compression alone is enough" | Alone, it's already O(log n) amortised; with union by size it's almost O(1). Use both |

**Component metadata.** Anything you can combine when two groups merge (size, minimum, maximum, sum, count of edges) is kept at the root and merged in `union`. CSES Road Construction (practice) needs the number of components and the largest size after each new road.

---

## 1. Attempt first ★

> **[Redundant Connection](https://leetcode.com/problems/redundant-connection/) (15 minutes).** A tree with n nodes had one extra edge added. Given the edges in order, return an edge whose removal leaves a tree. If several work, return the one that appears last.

---

## 2. Cycle detection

### Worked U1 · [Redundant Connection](https://leetcode.com/problems/redundant-connection/) ★

Add the edges one at a time. An edge whose endpoints are **already connected** closes a cycle, because there was already a path between them. The first such edge is the answer. It's the last edge of the (only) cycle in input order, and every other edge on that cycle came before it.

```java
public int[] findRedundantConnection(int[][] edges) {
    DSU dsu = new DSU(edges.length + 1);            // nodes are 1..n, and there are n edges
    for (int[] e : edges) {
        if (!dsu.union(e[0], e[1])) return e;       // already connected: this edge closes the cycle
    }
    return new int[0];                              // unreachable: one edge is always redundant
}
```

That's O(n · α(n)), against O(n²) for running a DFS after each edge. In undirected graphs, union-find is often the shortest way to detect a cycle. (In directed graphs it doesn't work. Use Ch 21 §7.)

---

## 3. Grouping by an equivalence

When a relation is **transitive** ("shares an email with", "is equal to", "is connected by a swap"), the groups it defines are exactly the components of union-find.

### Worked U2 · [Accounts Merge](https://leetcode.com/problems/accounts-merge/)

> Each account is `[name, email1, email2, …]`. Two accounts belong to the same person if they **share any email**. Merge them, and return each person's emails sorted.

**The model:** each account is an element. For each email, union the current account with the **first account seen with that email**. Accounts that share an email, even through a chain of other accounts, end up in one group.

```java
public List<List<String>> accountsMerge(List<List<String>> accounts) {
    int n = accounts.size();
    DSU dsu = new DSU(n);
    Map<String, Integer> ownerOf = new HashMap<>();                 // email -> the first account seen with it
    for (int i = 0; i < n; i++) {
        List<String> acc = accounts.get(i);
        for (int j = 1; j < acc.size(); j++) {
            Integer first = ownerOf.putIfAbsent(acc.get(j), i);
            if (first != null) dsu.union(i, first);                 // a shared email: the same person
        }
    }
    Map<Integer, TreeSet<String>> emailsOf = new HashMap<>();       // group root -> its emails, sorted
    for (Map.Entry<String, Integer> e : ownerOf.entrySet()) {
        emailsOf.computeIfAbsent(dsu.find(e.getValue()), k -> new TreeSet<>()).add(e.getKey());
    }
    List<List<String>> result = new ArrayList<>();
    for (Map.Entry<Integer, TreeSet<String>> e : emailsOf.entrySet()) {
        List<String> merged = new ArrayList<>();
        merged.add(accounts.get(e.getKey()).get(0));                // the name
        merged.addAll(e.getValue());
        result.add(merged);
    }
    return result;
}
```

O(total emails · log) for the sorting. The same idea, "union by a shared attribute", underlies Most Stones Removed (rows and columns become elements), Smallest String With Swaps (positions connected by allowed swaps) and Satisfiability of Equality Equations (practice).

---

## 4. Minimum spanning trees

A **spanning tree** of a connected, undirected, weighted graph is a subset of its edges that connects every vertex and has no cycle, which means exactly V − 1 edges. A **minimum spanning tree (MST)** is one with the smallest total weight: the cheapest way to connect everything.

### 4.1 The cut property: why greedy works

> **Cut property.** Split the vertices into any two non-empty groups S and V − S. **The lightest edge crossing between them belongs to some MST.**
>
> *Proof (exchange, Ch 2 §5).* Let e be that lightest crossing edge, and take an MST T without e. Adding e to T creates a cycle. That cycle crosses from S to V − S along e, so it must cross back along some other edge e′, and e′ weighs at least as much as e (e was the lightest crossing edge). Removing e′ breaks the cycle and leaves a spanning tree whose weight is no greater. So some MST contains e. ∎

Both classic algorithms just keep applying the cut property.

### 4.2 Kruskal: sort the edges, add the ones that join different components

Process edges from lightest to heaviest. **Add an edge if it joins two different components**, and skip it if it would form a cycle. Each added edge is the lightest edge leaving its component (every lighter edge has already been handled), so the cut property says it's safe.

```java
static long kruskal(int n, int[][] edges) {                 // edges: {u, v, w}
    int[][] sorted = edges.clone();
    Arrays.sort(sorted, (a, b) -> Integer.compare(a[2], b[2]));
    DSU dsu = new DSU(n);
    long total = 0;
    int used = 0;
    for (int[] e : sorted) {
        if (dsu.union(e[0], e[1])) {                         // joins two components: safe by the cut property
            total += e[2];
            if (++used == n - 1) break;                      // a spanning tree is complete
        }
    }
    return used == n - 1 ? total : -1;                       // fewer edges: the graph is disconnected
}
```

That's **O(E log E)**, dominated by the sort. It's the right choice for sparse graphs given as an edge list (CSES Road Reparation, practice).

### 4.3 Prim: grow one tree, always taking the cheapest edge out of it

Start from any vertex and repeatedly add **the cheapest edge leaving the current tree**. With S = the tree, that's the cut property directly. With a heap it's O(E log V), like Dijkstra. For a **dense** graph, where every pair is connected, a plain array of "cheapest edge into the tree" for each outside vertex gives **O(V²)** with no heap at all.

### Worked U3 · [Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/)

> n ≤ 1000 points; connecting two points costs their Manhattan distance. Return the minimum total cost to connect all of them.

Every pair is a possible edge: n² ≈ 10⁶ edges. Kruskal would sort them all (O(n² log n)). **Prim's O(n²) array version** is simpler and faster:

```java
public int minCostConnectPoints(int[][] points) {
    int n = points.length;
    int[] best = new int[n];                 // cheapest edge from each outside point into the tree so far
    Arrays.fill(best, Integer.MAX_VALUE);
    boolean[] inTree = new boolean[n];
    best[0] = 0;                             // start the tree at point 0
    long total = 0;
    for (int added = 0; added < n; added++) {
        int u = -1;
        for (int v = 0; v < n; v++) {        // the outside point with the cheapest connection
            if (!inTree[v] && (u == -1 || best[v] < best[u])) u = v;
        }
        inTree[u] = true;
        total += best[u];
        for (int v = 0; v < n; v++) {        // u's edges may give outside points cheaper connections
            if (inTree[v]) continue;
            int d = Math.abs(points[u][0] - points[v][0]) + Math.abs(points[u][1] - points[v][1]);
            if (d < best[v]) best[v] = d;
        }
    }
    return (int) total;
}
```

**MSTs also solve minimax paths.** The path between u and v *inside the MST* minimises the **largest edge** on any u–v path. So "the cheapest way to get from a to b where the cost is the worst step" can be answered with Kruskal: add edges in increasing order until a and b become connected. The weight of the last edge added is the answer. That's a third way to solve Swim in Rising Water and Path With Minimum Effort (Ch 23).

---

## 5. Offline queries: sort the questions too

### Worked U4 · [Checking Existence of Edge Length Limited Paths](https://leetcode.com/problems/checking-existence-of-edge-length-limited-paths/)

> An undirected graph with weighted edges, and queries `(p, q, limit)`: is there a path from p to q using only edges with weight **strictly less than** `limit`? (Up to 10⁵ edges and 10⁵ queries.)

A BFS per query is O(Q · (V + E)) ≈ 10¹⁰. But a query with a larger limit allows **every edge a smaller limit allows, and more**. So **answer the queries in increasing order of limit** (*offline*: you're allowed to reorder them, as long as each answer goes back to its original position). Sweep through the edges sorted by weight, unioning each one whose weight is below the current limit, then check `connected(p, q)`.

```java
public boolean[] distanceLimitedPathsExist(int n, int[][] edgeList, int[][] queries) {
    int[][] edges = edgeList.clone();
    Arrays.sort(edges, (a, b) -> Integer.compare(a[2], b[2]));
    Integer[] order = new Integer[queries.length];                  // query indices, by increasing limit
    for (int i = 0; i < order.length; i++) order[i] = i;
    Arrays.sort(order, (a, b) -> Integer.compare(queries[a][2], queries[b][2]));
    DSU dsu = new DSU(n);
    boolean[] answer = new boolean[queries.length];
    int e = 0;
    for (int qi : order) {
        int limit = queries[qi][2];
        while (e < edges.length && edges[e][2] < limit) {           // every edge allowed by this limit
            dsu.union(edges[e][0], edges[e][1]);
            e++;
        }
        answer[qi] = dsu.connected(queries[qi][0], queries[qi][1]); // stored at the query's ORIGINAL index
    }
    return answer;
}
```

That's O((E + Q) log + (E + Q) α(n)). **Offline processing** (sort the queries along with the data, sweep once, and write each answer back to its original position) reappears in Ch 34 and Ch 37. Number of Good Paths (stretch) is the same sweep, ordered by node value.

**Deletions** are the one thing union-find can't do: groups can merge but never split. When a problem *removes* edges over time, **reverse time**. Start from the final graph and add the removed edges back in reverse order (Ch 37, Bricks Falling When Hit).

---

## 6. Where union-find breaks

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Edges are **removed** | Groups can't be split | Process in reverse, as additions; or recompute per query |
| You need the actual **path** between a and b | DSU only knows "same group" | BFS/DFS (Ch 21), or the MST's tree path |
| **Directed** reachability | "Same group" is symmetric; reachability isn't | BFS/DFS, SCCs (Ch 25) |
| Relations with **weights** (a/b = 2.5) | A plain DSU loses the ratios | A *weighted* DSU: store each node's ratio to its parent, multiplied along `find` (Evaluate Division) |
| No union by size or path compression | Chains lead to O(n) finds | Always use both |

---

## 7. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| Count the edges needed to connect everything | Number of components − 1, provided there are enough edges (practice) |
| Union-find on a grid | Element id = `r * cols + c` (Ch 5 §1) |
| Two edge types, each person must see a connected graph | Two DSUs; use the shared edges first (stretch) |
| Which MST edges are essential / optional | Re-run Kruskal excluding each edge, or forcing it in (stretch) |
| Connectivity by shared prime factors | Union each number with its prime factors (Ch 18 SPF; stretch) |

---

## 8. Recognition

**Signals:**
* "Connected", "groups", "merge", "union", "same group", "friend circles", "**equivalence**" (a transitive relation).
* Edges or relations **arrive over time**, followed by "are a and b connected (yet)?".
* "Does adding this edge create a cycle?" (undirected).
* "**Minimum cost to connect** all …", "build roads / cables / pipes" → **MST**.
* Queries with **thresholds** (≤ limit, ≥ value) → sort the queries and sweep with a DSU.

**False friends:** directed dependencies (Ch 22), and shortest paths (Ch 23). A DSU knows *whether* two vertices are connected, not *how far apart* they are.

---

## 9. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Recursive `find` on long chains | Iterative (as above), or rely on union by size |
| `parent[a] = b` instead of linking the *roots* | Always `find` both first |
| Forgetting `components--` | Decrement inside a successful `union` only |
| Answering offline queries in sorted order but returning them in that order | Write each answer at the query's original index |
| Kruskal without sorting by weight (or with a subtracting comparator) | `Integer.compare` on the weight |
| `int` MST totals with large weights | `long` |

---

## 10. Practice

1. **[Number of Operations to Make Network Connected](https://leetcode.com/problems/number-of-operations-to-make-network-connected/)** *(Medium)*
2. **[Satisfiability of Equality Equations](https://leetcode.com/problems/satisfiability-of-equality-equations/)** *(Medium)*
3. **[CSES Road Construction](https://cses.fi/problemset/task/1676)** *(Medium; stdin/stdout)*
4. **[Most Stones Removed with Same Row or Column](https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/)** *(Medium)*
5. **[Smallest String With Swaps](https://leetcode.com/problems/smallest-string-with-swaps/)** *(Medium)*
6. **[Lexicographically Smallest Equivalent String](https://leetcode.com/problems/lexicographically-smallest-equivalent-string/)** *(Medium)*
7. **[Evaluate Division](https://leetcode.com/problems/evaluate-division/)** *(Medium)*
8. **[CSES Road Reparation](https://cses.fi/problemset/task/1675)** *(Medium; stdin/stdout)*

**Stretch:** **[Remove Max Number of Edges to Keep Graph Fully Traversable](https://leetcode.com/problems/remove-max-number-of-edges-to-keep-graph-fully-traversable/)** *(Hard)* · **[Number of Good Paths](https://leetcode.com/problems/number-of-good-paths/)** *(Hard)* · **[Find Critical and Pseudo-Critical Edges in Minimum Spanning Tree](https://leetcode.com/problems/find-critical-and-pseudo-critical-edges-in-minimum-spanning-tree/)** *(Hard)* · **[Largest Component Size by Common Factor](https://leetcode.com/problems/largest-component-size-by-common-factor/)** *(Hard)*

<details><summary>Nudges</summary>

1. Network Connected: with fewer than n − 1 cables it's impossible. Otherwise the answer is (components − 1).
2. Equality Equations: union all the `==` pairs first, then check that no `!=` pair ends up in the same group.
3. Road Construction: a DSU tracking the component count and the largest component size; print both after each road (fast I/O).
4. Stones: union each stone's row with its column (offset the column ids). The answer is stones − (number of groups).
5. Swaps: positions connected by swaps form groups; within each group, sort the characters and place them back in sorted position order.
6. Equivalent String: a DSU over 26 letters where the root is always the smallest letter (link the larger root under the smaller).
7. Evaluate Division: a graph with weighted edges a → b (value) and b → a (1/value), then DFS/BFS multiplying along the path. Or a weighted DSU.
8. Road Reparation: Kruskal; print `IMPOSSIBLE` if fewer than n − 1 edges get used.
9. Stretch, Fully Traversable: two DSUs (Alice, Bob); add the type-3 edges to both first, then the type-1 and type-2 edges; count the edges that were never needed.
10. Stretch, Good Paths: process the nodes by increasing value, unioning with already-processed neighbours; within each new group, count pairs of nodes with the current value.
11. Stretch, Critical Edges: for each edge, compare the MST weight without it (if it rises, the edge is critical) and with it forced in (if it's unchanged, the edge is pseudo-critical).
12. Stretch, Common Factor: union each number with each of its prime factors (using an SPF sieve, Ch 18); the answer is the largest group.
</details>

<details><summary>Key</summary>

Network Connected → count components; check edges ≥ n − 1 · Equality Equations → U1 (unions first, then the ≠ checks) · Road Construction → U0 with size and count metadata · Stones → U2 (rows and columns as elements) · Swaps → U2 + sort each group · Equivalent String → U0 with the smallest letter as root · Evaluate Division → weighted DSU or BFS · Road Reparation → Kruskal · Fully Traversable → two DSUs, shared edges first · Good Paths → U4 by value · Critical Edges → Kruskal with exclusion/forcing · Common Factor → U2 over prime factors
</details>

---

## 11. Self-test

<details><summary>1. Why does union by size keep every tree's height at most log₂ n?</summary>

A node's depth grows only when its tree is attached under one at least as large, which at least doubles the size of the tree containing it. A size can double at most log₂ n times.
</details>

<details><summary>2. State the cut property and sketch its proof.</summary>

The lightest edge crossing any cut belongs to some MST. Add it to an MST that lacks it: the resulting cycle crosses the cut again along an edge at least as heavy. Remove that edge, and you have a spanning tree that's no heavier.
</details>

<details><summary>3. Why is Prim's O(V²) array version better than Kruskal for 1000 points with all pairs connected?</summary>

There are about 5·10⁵ edges. Kruskal must generate and sort all of them, while array-based Prim does V rounds of O(V) work with no sorting and no heap.
</details>

<details><summary>4. Why can the limit queries be answered in sorted order?</summary>

A larger limit allows a superset of the edges a smaller limit allows, so the DSU state only ever grows as the limit increases. Sort the queries, sweep the edges once, and store each answer at its original index.
</details>

<details><summary>5. Why can't union-find support edge deletions directly?</summary>

Merging loses the information about which edges created a group, so there's no way to split it again. Problems with deletions are solved by reversing time, so deletions become additions.
</details>

---

## 12. Exit gate

Move on when you can:

* derive union-find from the naive versions, and prove the log-height bound;
* write the DSU class (iterative `find`, union by size, component count) from a blank file in under 5 minutes;
* state and prove the cut property, and explain why Kruskal and Prim are both applications of it;
* solve an offline threshold-query problem by sorting the queries;
* solve 7 of the 8 practice problems.
