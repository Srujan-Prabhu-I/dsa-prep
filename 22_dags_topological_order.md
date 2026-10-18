# Chapter 22 — DAGs & Topological Order

> **Part V · Week 20** · Prerequisites: Ch 21 (BFS, DFS, the three-state cycle check), Ch 9 (sorting)
> **Problems:** 6 worked (1 ★) · 6 practice · 2 stretch · **Time:** about 6 hours

Dependencies are everywhere: courses and prerequisites, build steps, tasks in a pipeline, package installs. They form a **directed graph**, and two questions come up again and again. **Is there a valid order to do everything?** And if there is, **what is it?** Both are answered by a *topological sort*. Once you have that order, a third, more powerful idea follows: **any quantity that depends only on predecessors can be computed in one pass along it**, which is your first taste of dynamic programming.

---

## 0. From zero: topological order

A **topological order** of a directed graph lists every vertex so that **every edge `u → v` goes forward**: u appears before v.

> **Theorem.** A topological order exists **if and only if** the graph has no directed cycle (it's a **DAG**, a directed acyclic graph).
>
> * *If there's a cycle,* no order works: the vertices of the cycle would each have to come before the next one, all the way round, which is impossible.
> * *If there's no cycle,* some vertex has **in-degree 0** (no incoming edges). Otherwise, start anywhere and keep stepping backwards along incoming edges. With finitely many vertices, you'd eventually revisit one, and that's a cycle. Put that vertex first, delete it, and the rest is still a DAG. Repeat.

That proof is an algorithm: **repeatedly take a vertex with no remaining prerequisites.**

```text
   0 ──▶ 1 ──▶ 3            in-degree:  0:0  1:1  2:1  3:2  4:1
   │           ▲
   └──▶ 2 ─────┘            take 0 (in-degree 0)    → 1 and 2 drop to 0        order: 0
        │                   take 1                  → 3 drops to 1             order: 0 1
        └──▶ 4              take 2                  → 3 drops to 0, 4 to 0     order: 0 1 2
                            take 3, then 4                                     order: 0 1 2 3 4
```

Every edge points left to right in the final order. `0 2 1 4 3` is also valid: a DAG usually has many topological orders. Add the edge `3 ──▶ 0` and the cycle 0 → 1 → 3 → 0 means 0, 1 and 3 never reach in-degree 0, so no order exists.

---

## 1. Attempt first ★

> **[Course Schedule](https://leetcode.com/problems/course-schedule/) (20 minutes).** There are `numCourses` courses and pairs `[a, b]` meaning "to take a, you must first take b". Can you finish all of them?

---

## 2. Kahn's algorithm

### Worked D1 · [Course Schedule I & II](https://leetcode.com/problems/course-schedule/)

Turn the proof into code. Keep each vertex's **in-degree**, the number of prerequisites it is still waiting for. Every vertex with in-degree 0 is **ready**. Take a ready vertex, output it, and decrement its successors' in-degrees. A successor whose in-degree reaches 0 becomes ready.

```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> after = new ArrayList<>();          // after.get(u) = the courses that need u first
    for (int i = 0; i < numCourses; i++) after.add(new ArrayList<>());
    int[] indegree = new int[numCourses];                   // prerequisites still outstanding
    for (int[] p : prerequisites) {                          // p = {course, prerequisite}: edge prerequisite -> course
        after.get(p[1]).add(p[0]);
        indegree[p[0]]++;
    }
    ArrayDeque<Integer> ready = new ArrayDeque<>();
    for (int c = 0; c < numCourses; c++) if (indegree[c] == 0) ready.offer(c);
    int[] order = new int[numCourses];
    int taken = 0;
    while (!ready.isEmpty()) {
        int u = ready.poll();
        order[taken++] = u;
        for (int v : after.get(u)) {
            if (--indegree[v] == 0) ready.offer(v);          // v's last prerequisite is now done
        }
    }
    return taken == numCourses ? order : new int[0];         // anything left over is stuck behind a cycle
}
// Course Schedule I: return findOrder(numCourses, prerequisites).length == numCourses;
```

**Why a shortfall proves a cycle.** If some vertices are never output, each of them still has an incoming edge from another leftover vertex (otherwise its in-degree would have reached 0). Walking backwards along those edges must eventually repeat a vertex: a cycle. So **Kahn's algorithm both sorts and detects cycles**, in **O(V + E)**.

**Getting the edge direction right** is the bug in half of all attempts. `[a, b]` means *b → a*, "b before a". Write the edge's meaning in a comment, as above.

### The DFS alternative: reverse finishing order

In Ch 21's three-state DFS, a vertex **finishes** only after everything reachable from it has finished. So if `u → v`, then v finishes before u. **Listing the vertices in reverse finishing order puts every u before its successors:**

```java
// state: 0 = unvisited, 1 = on the current path, 2 = finished. Returns false if a cycle is found.
static boolean visit(int u, List<List<Integer>> adj, int[] state, List<Integer> finished) {
    state[u] = 1;
    for (int v : adj.get(u)) {
        if (state[v] == 1) return false;                        // an edge back into the current path: a cycle
        if (state[v] == 0 && !visit(v, adj, state, finished)) return false;
    }
    state[u] = 2;
    finished.add(u);                                            // everything reachable from u is already in the list
    return true;
}
// Call visit from every unvisited vertex; the topological order is `finished` reversed.
```

Both run in O(V + E). Prefer **Kahn's** when you need levels, the lexicographically smallest order (use a `PriorityQueue` in place of the queue, Ch 15), or an iterative algorithm for deep graphs. Prefer **DFS** when you're already running a DFS for other reasons, such as the strongly connected components of Ch 25.

---

## 3. Finding the order from comparisons

### Worked D2 *(custom)* · Alien Alphabet

> Words from an alien language are given **sorted** in that language's alphabetical order. The alphabet is some ordering of the lowercase letters that appear. Return any letter order consistent with the list, or `""` if none exists. (This is LeetCode's premium "Alien Dictionary". Verify your solution with the Ch 5 harness: check that your order really sorts the list, and brute-force small alphabets to confirm the `""` cases.)
>
> `["wrt", "wrf", "er", "ett", "rftt"]` → `"wertf"` (one valid answer).

**What does a sorted list actually tell you?** Compare two **adjacent** words. Lexicographic order is decided by the **first position where they differ**: if `a[j] ≠ b[j]` there, then letter `a[j]` comes before `b[j]`. That's one edge. Everything after that position tells you nothing. And if one word is a **prefix** of the other, the shorter must come first. `"abc"` before `"ab"` is a contradiction, and no alphabet can fix it.

Comparing only adjacent pairs is enough, because sortedness is transitive: if every adjacent pair is in order, the whole list is. Then a topological sort of the letters gives the alphabet, and a cycle means the comparisons contradict each other.

```java
static String alienOrder(String[] words) {
    boolean[] present = new boolean[26];
    for (String w : words) for (char ch : w.toCharArray()) present[ch - 'a'] = true;
    List<Set<Integer>> after = new ArrayList<>();
    for (int i = 0; i < 26; i++) after.add(new HashSet<>());     // sets: the same edge may be implied many times
    int[] indegree = new int[26];
    for (int i = 0; i + 1 < words.length; i++) {
        String a = words[i], b = words[i + 1];
        int j = 0;
        while (j < a.length() && j < b.length() && a.charAt(j) == b.charAt(j)) j++;
        if (j == Math.min(a.length(), b.length())) {
            if (a.length() > b.length()) return "";               // "abc" before "ab": impossible
            continue;                                             // a is a prefix of b: no information
        }
        int u = a.charAt(j) - 'a', v = b.charAt(j) - 'a';         // the first difference: u comes before v
        if (after.get(u).add(v)) indegree[v]++;
    }
    ArrayDeque<Integer> ready = new ArrayDeque<>();
    int letters = 0;
    for (int c = 0; c < 26; c++) {
        if (!present[c]) continue;
        letters++;
        if (indegree[c] == 0) ready.offer(c);
    }
    StringBuilder order = new StringBuilder();
    while (!ready.isEmpty()) {
        int u = ready.poll();
        order.append((char) ('a' + u));
        for (int v : after.get(u)) if (--indegree[v] == 0) ready.offer(v);
    }
    return order.length() == letters ? order.toString() : "";    // a cycle: contradictory comparisons
}
```

Two details that trip people up: **store edges in a set**, so that a repeated edge doesn't increment an in-degree twice; and **include letters that never appear in any comparison**, since they still belong in the alphabet.

---

## 4. DP in topological order

**The principle:** in a topological order, **every predecessor of v comes before v**. So any value of v that depends only on its predecessors' values (a longest path, an earliest finish time, a number of paths) can be computed in **one pass**, with each value already final when it's needed. This is dynamic programming over a DAG. Ch 27 generalises it, and every DP you'll ever write evaluates its states in some topological order.

### Worked D3a · [Parallel Courses III](https://leetcode.com/problems/parallel-courses-iii/)

> Course i takes `time[i]` months. Relations `[a, b]` mean a must finish before b starts (1-indexed). Any number of courses can run in parallel. Return the minimum number of months to finish them all. The graph is a DAG.

A course can start once its **latest** prerequisite finishes:

> `finish[v] = time[v] + max(finish[u] over its prerequisites u)`, or just `time[v]` if it has none.

The answer is the maximum finish time. That's the **longest path** in the DAG, with weights on the vertices. Run the computation inside Kahn's loop:

```java
public int minimumTime(int n, int[][] relations, int[] time) {
    List<List<Integer>> after = new ArrayList<>();
    for (int i = 0; i < n; i++) after.add(new ArrayList<>());
    int[] indegree = new int[n];
    for (int[] r : relations) {
        after.get(r[0] - 1).add(r[1] - 1);              // 1-indexed input: convert once
        indegree[r[1] - 1]++;
    }
    int[] finish = new int[n];                           // earliest finish time of each course
    ArrayDeque<Integer> ready = new ArrayDeque<>();
    for (int c = 0; c < n; c++) {
        if (indegree[c] == 0) {
            finish[c] = time[c];
            ready.offer(c);
        }
    }
    int answer = 0;
    while (!ready.isEmpty()) {
        int u = ready.poll();                            // every prerequisite of u is done: finish[u] is final
        answer = Math.max(answer, finish[u]);
        for (int v : after.get(u)) {
            finish[v] = Math.max(finish[v], finish[u] + time[v]);
            if (--indegree[v] == 0) ready.offer(v);
        }
    }
    return answer;
}
```

O(V + E). **Longest paths are only easy on DAGs.** In a general graph, "longest simple path" is NP-hard, so getting the DAG structure first is what makes this possible.

### Worked D3b · [Longest Increasing Path in a Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/)

> Return the length of the longest path through a grid in which each step moves to an adjacent cell with a **strictly larger** value.

**Spot the DAG:** draw an edge from each cell to every strictly larger neighbour. Values strictly increase along every edge, so **no cycle is possible**. It's a DAG, and the answer is its longest path. **A topological order for free:** sorting cells by value is one, because every edge goes from smaller to larger. Process the cells from the **largest value down**, so every larger neighbour's answer is already known:

```java
public int longestIncreasingPath(int[][] matrix) {
    int rows = matrix.length, cols = matrix[0].length, n = rows * cols;
    Integer[] cells = new Integer[n];                  // cell id = r * cols + c (Ch 5 §1)
    for (int i = 0; i < n; i++) cells[i] = i;
    Arrays.sort(cells, (a, b) -> Integer.compare(matrix[b / cols][b % cols], matrix[a / cols][a % cols]));
    int[] longest = new int[n];                        // the longest increasing path starting at each cell
    int[] dr = {-1, 1, 0, 0}, dc = {0, 0, -1, 1};
    int best = 0;
    for (int id : cells) {                             // larger values first: their answers are final
        int r = id / cols, c = id % cols;
        longest[id] = 1;
        for (int d = 0; d < 4; d++) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols || matrix[nr][nc] <= matrix[r][c]) continue;
            longest[id] = Math.max(longest[id], 1 + longest[nr * cols + nc]);
        }
        best = Math.max(best, longest[id]);
    }
    return best;
}
```

O(RC log(RC)) for the sort. The common alternative is a DFS that **remembers each cell's answer the first time it's computed** (Ch 6 §0.3), which is O(RC) but recurses up to RC deep. The sorted version is iterative, so it's safe on a 200 × 200 grid.

---

## 5. Peeling an undirected tree from its leaves

### Worked D4 · [Minimum Height Trees](https://leetcode.com/problems/minimum-height-trees/)

> An undirected tree with n vertices. Choosing a root gives a rooted tree of some height. Return every root that gives the minimum height.

Running a BFS from every vertex is O(n²). **Reverse the view (Ch 1, move 5):** the best roots are the **centre** of the tree, the middle of its longest path. Find it by **peeling leaves layer by layer**, the undirected cousin of Kahn's algorithm. Each round removes every current leaf, and the vertices that become leaves form the next layer. The tree shrinks from all of its ends at the same speed, so **the last 1 or 2 vertices are the middle of every longest path**, which are the roots of minimum height.

```java
public List<Integer> findMinHeightTrees(int n, int[][] edges) {
    if (n == 1) return List.of(0);
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    int[] degree = new int[n];
    for (int[] e : edges) {
        adj.get(e[0]).add(e[1]);
        adj.get(e[1]).add(e[0]);
        degree[e[0]]++;
        degree[e[1]]++;
    }
    ArrayDeque<Integer> leaves = new ArrayDeque<>();
    for (int v = 0; v < n; v++) if (degree[v] == 1) leaves.offer(v);
    int remaining = n;
    while (remaining > 2) {                          // peel one whole layer per round
        int layer = leaves.size();
        remaining -= layer;
        for (int i = 0; i < layer; i++) {
            int leaf = leaves.poll();
            for (int v : adj.get(leaf)) {
                if (--degree[v] == 1) leaves.offer(v);   // v has just become a leaf
            }
        }
    }
    return new ArrayList<>(leaves);
}
```

O(n). Why at most two survivors: a longest path with an odd number of vertices has one middle vertex, and one with an even number has two.

---

## 6. Where topological sorting breaks

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| The graph has cycles, but you still need an order or a DP | No topological order exists | Collapse each strongly connected component into one vertex; the result is a DAG (Ch 25) |
| "Is the order unique?" | Many orders are usually valid | It's unique exactly when Kahn's queue never holds more than one vertex |
| Shortest paths with weights on a general graph | No DAG order to follow | Dijkstra / Bellman–Ford (Ch 23) |
| Longest *simple* path in a graph with cycles | NP-hard in general | Exploit special structure, or n ≤ 20 bitmask DP (Ch 32) |
| Edges given as `[course, prerequisite]` but read the wrong way | A reversed order, or wrong in-degrees | Comment each edge's meaning |

---

## 7. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| The lexicographically smallest valid order | Kahn's with a min-heap of ready vertices |
| The minimum number of semesters (with unlimited courses per semester) | Kahn's, level by level: the number of levels |
| At most k courses per semester | Bitmask DP (Ch 32, Parallel Courses II) |
| Count the paths from s to t in a DAG | `paths[v] = Σ paths[u]` in topological order (mod p) |
| Reachability between all pairs | Propagate ancestor sets in topological order (practice: All Ancestors / Course Schedule IV) |

---

## 8. Recognition

**Signals:**
* "Prerequisites", "dependencies", "must happen before", "build order", "install order" → a directed graph; **topological sort**.
* "Is it possible to finish all…?" → **cycle detection** via Kahn's count.
* "Earliest finish", "longest chain", "number of ways", along dependencies → **DP in topological order**.
* A grid or sequence where moves go strictly "upward" in some value → an **implicit DAG**; sort by that value.
* "Minimum height / centre of a tree" → **leaf peeling**.

**False friends:** undirected "dependencies" (they're just connectivity), and dependency graphs that contain cycles but need DP anyway (Ch 25).

---

## 9. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Reversed edge direction | Write "edge X → Y means X before Y" next to where the graph is built |
| A duplicate edge counted twice in an in-degree | Use a set of edges, or accept duplicates in both places consistently |
| Forgetting vertices that have no edges at all | Start the ready queue from *every* in-degree-0 vertex |
| Checking only that the queue emptied, not the output count | Compare `taken` with `numCourses` |
| Recursive DFS topological sort on 10⁵ vertices | Kahn's (iterative) |
| 1-indexed input | Convert once, at reading time |

---

## 10. Practice

1. **[Minimum Number of Vertices to Reach All Nodes](https://leetcode.com/problems/minimum-number-of-vertices-to-reach-all-nodes/)** *(Medium)*
2. **[Find All Possible Recipes from Given Supplies](https://leetcode.com/problems/find-all-possible-recipes-from-given-supplies/)** *(Medium)*
3. **[All Ancestors of a Node in a Directed Acyclic Graph](https://leetcode.com/problems/all-ancestors-of-a-node-in-a-directed-acyclic-graph/)** *(Medium)*
4. **[Course Schedule IV](https://leetcode.com/problems/course-schedule-iv/)** *(Medium)*
5. **[Loud and Rich](https://leetcode.com/problems/loud-and-rich/)** *(Medium)*
6. **[Build a Matrix With Conditions](https://leetcode.com/problems/build-a-matrix-with-conditions/)** *(Hard)*

**Stretch:** **[Largest Color Value in a Directed Graph](https://leetcode.com/problems/largest-color-value-in-a-directed-graph/)** *(Hard)* · **[Sort Items by Groups Respecting Dependencies](https://leetcode.com/problems/sort-items-by-groups-respecting-dependencies/)** *(Hard)*

<details><summary>Nudges</summary>

1. Minimum Vertices: every vertex with in-degree 0 must be chosen (nothing else reaches it), and those vertices together reach everything. Prove the second part.
2. Recipes: recipes and ingredients are both vertices; edge ingredient → recipe. Start Kahn's from the supplies.
3. All Ancestors: in topological order, each vertex's ancestor set = the union of (each parent + that parent's ancestors). Use sorted sets or bitsets.
4. Course Schedule IV: compute reachability for all pairs. Propagate in topological order, or use Floyd–Warshall on n ≤ 100 (Ch 23).
5. Loud and Rich: the edge "richer → poorer". In topological order, push down the quietest person found so far.
6. Build a Matrix: one topological sort for the row conditions and one for the column conditions, independently. Place number i at (rowPos[i], colPos[i]).
7. Stretch, Largest Color Value: `count[v][c]` = the most nodes of colour c on any path ending at v; propagate in Kahn's order. If not every node is processed, there's a cycle: return −1.
8. Stretch, Sort Items: two levels of topological sort. First sort the groups (give each ungrouped item its own group), then sort the items inside each group.
</details>

<details><summary>Key</summary>

Min Vertices → the in-degree-zero argument · Recipes → D1 on a mixed graph · All Ancestors → D1 propagating sets (D3 style) · Course Schedule IV → reachability via D1, or Floyd · Loud and Rich → D3 · Matrix With Conditions → two D1 runs · Largest Color Value → D3 with 26 counters + a cycle check · Sort Items → two-level D1
</details>

---

## 11. Self-test

<details><summary>1. Prove that every DAG has a vertex with in-degree 0.</summary>

If every vertex had an incoming edge, you could walk backwards along incoming edges forever. With finitely many vertices, the walk would repeat one, and that's a cycle, contradicting acyclicity.
</details>

<details><summary>2. Why does Kahn's algorithm leave vertices unprocessed exactly when there's a cycle?</summary>

Every vertex on a cycle has an incoming edge from the cycle, so its in-degree can never reach 0. Conversely, if vertices are left over, each has an incoming edge from another leftover vertex, and walking backwards finds a cycle.
</details>

<details><summary>3. Why does reverse finishing order (DFS) give a topological order?</summary>

For an edge u → v in a DAG, v finishes before u: either v is explored from u, or it had already finished. So in reverse finishing order, u comes before v.
</details>

<details><summary>4. In the alien alphabet, why compare only adjacent words, and only up to the first difference?</summary>

Lexicographic order is decided by the first differing position, and later positions don't constrain the alphabet. Adjacent pairs suffice because if every adjacent pair is in order, the whole list is.
</details>

<details><summary>5. Why can longest paths be computed efficiently on DAGs but not on general graphs?</summary>

In topological order, each vertex's predecessors are final before it, so one pass suffices. With cycles there's no such order, and longest simple path is NP-hard in general.
</details>

---

## 12. Exit gate

Move on when you can:

* prove the DAG ⟺ topological-order theorem, and explain how the proof becomes Kahn's algorithm;
* write Kahn's algorithm (with cycle detection) and the DFS reverse-finishing-order version from blank files;
* solve the alien alphabet from a blank file, including both contradiction cases;
* write a DP in topological order (Parallel Courses III) and explain why each value is final when it's used;
* solve 5 of the 6 practice problems.
