# Chapter 21 — Graph Modelling, BFS & DFS

> **Part V · Week 19** · Prerequisites: Ch 5 (grids, direction arrays), Ch 6 (recursion), Ch 7 (hashing), Ch 12 (queues, stacks), Ch 19
> **Problems:** 9 worked (1 ★) · 11 practice · 4 stretch · **Time:** about 10 hours

A graph is the most general structure in this book: **things, and connections between them.** Cities and roads, people and friendships, grid cells and their neighbours, puzzle states and legal moves. The two algorithms in this chapter, breadth-first search and depth-first search, are short. In graph problems, though, the algorithm is rarely the hard part. **The hard part is seeing the graph**: deciding what a node is and what an edge is, when the problem talks about locks, words or obstacles. This chapter builds the algorithms from zero, then trains the modelling.

---

## 0. From zero: graphs

### 0.1 Vocabulary

A **graph** is a set of **vertices** (nodes) V and a set of **edges** E, each edge connecting two vertices.

| Term | Meaning |
| :--- | :--- |
| **Undirected / directed** | An edge goes both ways (a friendship), or one way, `u → v` (following someone) |
| **Weighted** | Each edge carries a number (a length, a cost). Unweighted means every edge counts as 1 |
| **Degree** | The number of edges at a vertex. In directed graphs, **in-degree** and **out-degree** separately |
| **Path** | A sequence of vertices, each joined to the next by an edge |
| **Cycle** | A path that returns to its start without reusing an edge |
| **Connected component** | A maximal set of vertices that can all reach each other (undirected) |
| **Tree** | A connected graph with no cycles. It has exactly V − 1 edges. Ch 19's trees are rooted trees |
| **DAG** | A directed acyclic graph: directed, with no cycles (Ch 22) |

A graph with V vertices has at most V(V − 1)/2 undirected edges. Most interview graphs are **sparse**: E is close to V, not V².

An undirected graph with V = 7 and E = 6, used through this section:

```text
   0 ─── 1            5 ─── 6         components: {0, 1, 2, 3, 4} and {5, 6}
   │     │                            degree(1) = 3 (neighbours 0, 2, 3)
   │     │                            cycle: 0 – 1 – 2 – 0
   2 ────┘                            path from 0 to 4: 0 – 1 – 3 – 4
         1 ─── 3 ─── 4                {5, 6} on its own is a tree (connected, no cycle)
```

(Vertex 1 appears twice in the drawing only for layout; it's one vertex with edges to 0, 2 and 3.) Written as an edge list: `{0,1} {0,2} {1,2} {1,3} {3,4} {5,6}`. **Directed** edges are drawn as arrows, `u ──▶ v`, and can be followed only in that direction.

### 0.2 Three ways to store a graph

| Representation | Memory | "Is u–v an edge?" | List u's neighbours | Use when |
| :--- | :-: | :-: | :-: | :--- |
| **Edge list** `int[][] edges` | O(E) | O(E) | O(E) | It's how input arrives; Kruskal sorts it (Ch 24) |
| **Adjacency matrix** `boolean[V][V]` | O(V²) | **O(1)** | O(V) | V ≤ ~2000 and dense graphs; Floyd–Warshall (Ch 23) |
| **Adjacency list** `List<List<Integer>>` | **O(V + E)** | O(degree) | **O(degree)** | **The default** for BFS/DFS/everything else |

The same graph (vertices 0–4 only, to keep it small) in each form:

```text
edge list            adjacency matrix              adjacency list
{0,1}                    0  1  2  3  4             0: [1, 2]
{0,2}                0 [ .  1  1  .  . ]           1: [0, 2, 3]
{1,2}                1 [ 1  .  1  1  . ]           2: [0, 1]
{1,3}                2 [ 1  1  .  .  . ]           3: [1, 4]
{3,4}                3 [ .  1  .  .  1 ]           4: [3]
                     4 [ .  .  .  1  . ]
```

Each undirected edge appears **twice** in the matrix (symmetric) and twice in the lists (once at each end). That's why the lists hold 2E entries in total, and the O(V + E) memory.

### Worked G0a *(custom)* · Build all three representations from an edge list

```java
static List<List<Integer>> adjacencyList(int n, int[][] edges, boolean directed) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int[] e : edges) {
        adj.get(e[0]).add(e[1]);
        if (!directed) adj.get(e[1]).add(e[0]);     // an undirected edge appears in both lists
    }
    return adj;
}

static boolean[][] adjacencyMatrix(int n, int[][] edges, boolean directed) {
    boolean[][] m = new boolean[n][n];
    for (int[] e : edges) {
        m[e[0]][e[1]] = true;
        if (!directed) m[e[1]][e[0]] = true;
    }
    return m;
}
```

(Weighted graphs store `int[] {neighbour, weight}` in the lists. Input is often 1-indexed, so subtract 1 on reading, as in Ch 4 §2.3.)

### 0.3 Breadth-first search: layer by layer

BFS explores outward from a source: first everything 1 edge away, then 2 edges away, and so on. It uses a **queue** (Ch 12): discovered vertices wait in arrival order.

```java
static int[] bfs(List<List<Integer>> adj, int source) {
    int[] dist = new int[adj.size()];
    Arrays.fill(dist, -1);                       // -1 = not reached yet
    ArrayDeque<Integer> queue = new ArrayDeque<>();
    dist[source] = 0;                            // mark a vertex when it is ENQUEUED
    queue.offer(source);
    while (!queue.isEmpty()) {
        int u = queue.poll();
        for (int v : adj.get(u)) {
            if (dist[v] != -1) continue;         // already discovered
            dist[v] = dist[u] + 1;
            queue.offer(v);
        }
    }
    return dist;
}
```

**A trace** from vertex 0 on the graph above (neighbours taken in list order):

```text
dequeue   newly discovered (dist)        queue afterwards
start     0 (0)                          [0]
0         1 (1), 2 (1)                   [1, 2]
1         3 (2)      (0, 2 already seen) [2, 3]
2         —          (0, 1 already seen) [3]
3         4 (3)                          [4]
4         —                              []

dist = [0, 1, 1, 2, 3, -1, -1]           5 and 6 are in another component: never reached
```

The queue always holds one distance "layer" followed by the start of the next, so vertices are finalised in rings around the source:

```text
layer 0: {0}    layer 1: {1, 2}    layer 2: {3}    layer 3: {4}
```

**Why BFS finds shortest paths** (in unweighted graphs). *Claim:* vertices leave the queue in non-decreasing order of `dist`, and `dist[v]` is the true distance. *Induction:* the queue only ever holds vertices at distance d followed by vertices at distance d + 1. A vertex at distance d + 1 is first discovered from some vertex at distance d, which is dequeued before any vertex at distance d + 1. So the first discovery comes through a shortest path. **Weights break this:** a two-edge path can be cheaper than a one-edge path, which is why Ch 23 exists.

**Mark when enqueued, not when dequeued.** If you mark on dequeue, a vertex can be enqueued once *per neighbour* before it's processed. In a dense graph that's O(E) queue entries for *each* vertex, and the run blows up.

**Cost:** each vertex is enqueued once, and each adjacency list is scanned once: **O(V + E)**.

### 0.4 Depth-first search: go deep, then back up

DFS follows one path as far as it can, then backtracks (Ch 6). It's the natural tool for **reachability, components, cycles and orderings**:

```java
static void dfs(int u, List<List<Integer>> adj, boolean[] seen) {
    seen[u] = true;
    for (int v : adj.get(u)) if (!seen[v]) dfs(v, adj, seen);
}
```

**Recursion depth equals the path length**, which can be V. On a path-shaped graph of 10⁵ vertices, that overflows Java's stack (Ch 4 §16). Here is an **iterative** version with an explicit stack. It reaches the same set of vertices, although in a different order:

```java
static boolean[] reachable(List<List<Integer>> adj, int source) {
    boolean[] seen = new boolean[adj.size()];
    ArrayDeque<Integer> stack = new ArrayDeque<>();
    seen[source] = true;
    stack.push(source);
    while (!stack.isEmpty()) {
        int u = stack.pop();
        for (int v : adj.get(u)) {
            if (!seen[v]) { seen[v] = true; stack.push(v); }
        }
    }
    return seen;
}
```

When the *order* matters (finishing order in Ch 22, low-link values in Ch 25), use the recursive version, or an explicit stack of (vertex, next-neighbour index). Like BFS, DFS is **O(V + E)**.

**The recursive DFS from 0** on the graph above, indented by recursion depth:

```text
dfs(0)                     neighbours [1, 2]
  dfs(1)                   neighbours [0, 2, 3]: 0 seen
    dfs(2)                 neighbours [0, 1]: both seen → return
    dfs(3)                 neighbours [1, 4]: 1 seen
      dfs(4)               neighbours [3]: seen → return
  (back in dfs(0): 2 already seen)
visit order: 0, 1, 2, 3, 4          BFS order was 0, 1, 2, 3, 4 too, but by layers, not by depth
```

DFS goes as deep as it can along one branch before backing up, which is why it finds *a* path but not necessarily the shortest. BFS explores in rings, which is why it finds the shortest.

**Common misconceptions:**

| Belief | Reality |
| :--- | :--- |
| "DFS finds shortest paths" | Only BFS does, in unweighted graphs; DFS finds *some* path |
| "BFS and DFS are O(V²)" | O(V + E) with adjacency lists; O(V²) only with a matrix |
| "Marking a vertex visited when it's dequeued is the same" | It lets a vertex be enqueued many times. Mark on enqueue (§0.3) |
| "A graph is connected if BFS from one vertex finishes" | Only if that BFS reached every vertex; otherwise count components by restarting from unvisited vertices |
| "An undirected edge is stored once" | Store it at **both** ends, or half the neighbours are missing |

### Worked G0b · [Find if Path Exists in Graph](https://leetcode.com/problems/find-if-path-exists-in-graph/)

> An undirected graph with n ≤ 2·10⁵ vertices. Is there a path from `source` to `destination`?

Build the adjacency list, then check reachability with the iterative search (the recursive one could overflow at this size). **O(V + E).** Ch 24 answers the same question with union-find, which is even simpler when edges arrive one at a time.

---

## 1. Attempt first ★

> **[Number of Islands](https://leetcode.com/problems/number-of-islands/) (20 minutes).** A grid of `'1'` (land) and `'0'` (water). Count the islands: groups of land cells connected horizontally or vertically.

---

## 2. Grids are graphs

A grid is a graph with **cells as vertices** and **edges between adjacent cells**. You never build the adjacency list. The direction arrays from Ch 5 §1 generate each cell's neighbours on the fly. An R × C grid has V = RC and E ≈ 2RC.

```text
grid (1 = land):           the same land cells as a graph:
  1 1 0 0                    (0,0)──(0,1)
  1 0 0 1                      │
  0 0 1 1                    (1,0)            (1,3)
                                                │
                                      (2,2)──(2,3)
                           two components = two islands
```

Cell `(r, c)` is the vertex; its neighbours are `(r ± 1, c)` and `(r, c ± 1)`, when they're inside the grid and passable. Diagonals count only if the problem says so.

### Worked G1 · [Number of Islands](https://leetcode.com/problems/number-of-islands/) ★

An island is a **connected component** of land. Scan every cell; each land cell not yet visited **starts a new island**, and a flood fill from it visits the whole island.

```java
public int numIslands(char[][] grid) {
    int rows = grid.length, cols = grid[0].length, islands = 0;
    int[] dr = {-1, 1, 0, 0}, dc = {0, 0, -1, 1};
    ArrayDeque<int[]> queue = new ArrayDeque<>();
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] != '1') continue;
            islands++;                                   // unvisited land: a new island
            grid[r][c] = '0';                            // mark it visited by sinking it
            queue.offer(new int[] {r, c});
            while (!queue.isEmpty()) {                   // BFS flood fill: iterative, so no stack overflow
                int[] cell = queue.poll();
                for (int d = 0; d < 4; d++) {
                    int nr = cell[0] + dr[d], nc = cell[1] + dc[d];
                    if (nr < 0 || nr >= rows || nc < 0 || nc >= cols || grid[nr][nc] != '1') continue;
                    grid[nr][nc] = '0';
                    queue.offer(new int[] {nr, nc});
                }
            }
        }
    }
    return islands;
}
```

O(R · C). Sinking cells **modifies the input**. In an interview, ask whether that's allowed, or use a separate `boolean[][] seen`. A recursive DFS flood fill is shorter, but a 300 × 300 all-land grid makes it 9·10⁴ calls deep, which is a stack-overflow risk.

**Reverse thinking (Ch 1, move 5)** is common with grids. "Which cells can reach the border / both oceans?" becomes "**flood from the border / each ocean**, and see what gets reached". Surrounded Regions and Pacific Atlantic (practice) both work this way.

---

## 3. Many starting points at once: multi-source BFS

### Worked G2 · [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)

> Each minute, every rotten orange (2) rots its fresh neighbours (1). Return the minutes until no fresh orange remains, or −1 if that never happens.

Rot spreads from **all rotten oranges simultaneously**. Put **every source in the queue at distance 0**. That's the same as one BFS from a virtual "super-source" joined to all of them, so the layer-by-layer proof of §0.3 still holds. Each BFS layer is one minute.

```java
public int orangesRotting(int[][] grid) {
    int rows = grid.length, cols = grid[0].length, fresh = 0;
    ArrayDeque<int[]> queue = new ArrayDeque<>();
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == 2) queue.offer(new int[] {r, c});    // every source starts at minute 0
            else if (grid[r][c] == 1) fresh++;
        }
    int[] dr = {-1, 1, 0, 0}, dc = {0, 0, -1, 1};
    int minutes = 0;
    while (!queue.isEmpty() && fresh > 0) {
        for (int size = queue.size(); size > 0; size--) {           // exactly one layer = one minute
            int[] cell = queue.poll();
            for (int d = 0; d < 4; d++) {
                int nr = cell[0] + dr[d], nc = cell[1] + dc[d];
                if (nr < 0 || nr >= rows || nc < 0 || nc >= cols || grid[nr][nc] != 1) continue;
                grid[nr][nc] = 2;
                fresh--;
                queue.offer(new int[] {nr, nc});
            }
        }
        minutes++;
    }
    return fresh == 0 ? minutes : -1;
}
```

Multi-source BFS answers "**distance to the nearest of many sources**" for every cell in one O(RC) pass, instead of one BFS per source. 01 Matrix and As Far from Land as Possible are both this.

---

## 4. Modelling: what is a state?

When a problem talks about moves, operations or transformations, **the vertices are the possible *states* and the edges are the legal *moves*.** "The minimum number of moves" is then a BFS distance. The key question:

> **What must a state record so that the future depends only on the state, not on how you got there?**

### Worked G3a · [Open the Lock](https://leetcode.com/problems/open-the-lock/)

> A lock has 4 wheels, each showing 0–9, starting at `"0000"`. One move turns one wheel by ±1 (with wrap-around). Some combinations are dead ends that can't be entered. Return the minimum number of moves to reach `target`, or −1.

**The model:** the states are the 10⁴ combinations. Each state has 8 neighbours (4 wheels × 2 directions). Dead ends are removed vertices. That's a BFS on 10⁴ vertices, which is tiny. Encode each state as an integer 0…9999 so `int[]` arrays can replace string sets.

```java
public int openLock(String[] deadends, String target) {
    boolean[] blocked = new boolean[10000];
    for (String d : deadends) blocked[Integer.parseInt(d)] = true;
    if (blocked[0]) return -1;
    int goal = Integer.parseInt(target);
    int[] dist = new int[10000];
    Arrays.fill(dist, -1);
    dist[0] = 0;
    ArrayDeque<Integer> queue = new ArrayDeque<>(List.of(0));
    int[] place = {1, 10, 100, 1000};                       // the value of each wheel's position
    while (!queue.isEmpty()) {
        int state = queue.poll();
        if (state == goal) return dist[state];
        for (int p : place) {
            int digit = state / p % 10;
            for (int delta : new int[] {1, 9}) {            // +1, and −1 written as +9 (mod 10)
                int next = state + ((digit + delta) % 10 - digit) * p;
                if (blocked[next] || dist[next] != -1) continue;
                dist[next] = dist[state] + 1;
                queue.offer(next);
            }
        }
    }
    return -1;
}
```

### Worked G3b · [Word Ladder](https://leetcode.com/problems/word-ladder/)

> Transform `beginWord` into `endWord`, changing one letter at a time, where every intermediate word must be in `wordList`. Return the number of words in the shortest sequence, or 0.

**The model:** the words are vertices, and two words are joined if they differ in exactly one letter. **How you generate the edges matters.** Comparing every pair of words costs n² · L, which is 2.5·10⁹ for 5000 words. Instead, for each word, **try all 26 letters at each position** and look the result up in a hash set: 26 · L candidates per word.

```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> unvisited = new HashSet<>(wordList);         // removing a word = marking it visited
    if (!unvisited.contains(endWord)) return 0;
    unvisited.remove(beginWord);
    ArrayDeque<String> queue = new ArrayDeque<>(List.of(beginWord));
    for (int length = 1; !queue.isEmpty(); length++) {      // length = words in the sequence so far
        for (int size = queue.size(); size > 0; size--) {
            char[] word = queue.poll().toCharArray();
            for (int i = 0; i < word.length; i++) {
                char original = word[i];
                for (char ch = 'a'; ch <= 'z'; ch++) {
                    if (ch == original) continue;
                    word[i] = ch;
                    String next = new String(word);
                    if (!unvisited.remove(next)) continue;   // not in the list, or already reached
                    if (next.equals(endWord)) return length + 1;
                    queue.offer(next);
                }
                word[i] = original;
            }
        }
    }
    return 0;
}
```

O(n · L · 26 · L), where the last L is the cost of building and hashing each string. **Generating neighbours from the state's own structure rather than by comparing pairs** is the general lesson for implicit graphs. (*Bidirectional BFS*, searching from both ends until the frontiers meet, cuts the work further. It's a common follow-up.)

---

## 5. Copying a graph

### Worked G4 · [Clone Graph](https://leetcode.com/problems/clone-graph/)

> Deep-copy a connected undirected graph, given one of its nodes. Each node has a value and a list of neighbours.

It's Ch 13's "copy with a map" applied to a graph, which may contain cycles. Keep a map from each original node to its copy. **Create a node's copy the first time the search meets it**, so the map doubles as the visited set and the cycles are handled automatically.

```java
public Node cloneGraph(Node node) {
    if (node == null) return null;
    Map<Node, Node> copyOf = new HashMap<>();
    copyOf.put(node, new Node(node.val));
    ArrayDeque<Node> queue = new ArrayDeque<>(List.of(node));
    while (!queue.isEmpty()) {
        Node u = queue.poll();
        for (Node v : u.neighbors) {
            if (!copyOf.containsKey(v)) {          // first meeting: create the copy, explore v later
                copyOf.put(v, new Node(v.val));
                queue.offer(v);
            }
            copyOf.get(u).neighbors.add(copyOf.get(v));   // copy the edge u-v (each direction once)
        }
    }
    return copyOf.get(node);
}
```

---

## 6. Two-colouring: bipartite graphs

### Worked G5 · [Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/)

> Can the vertices be split into two groups so that every edge goes **between** the groups?

Colour greedily. Give the start colour A; its neighbours must be B, their neighbours A, and so on. That's BFS. The colouring is **forced**: once one vertex in a component has a colour, every other vertex's colour follows. So if the BFS ever finds an edge between two vertices of the same colour, no valid colouring exists. Otherwise the colouring it built is valid. (Equivalently, a graph is bipartite exactly when it has **no odd-length cycle**.)

```java
public boolean isBipartite(int[][] graph) {
    int n = graph.length;
    int[] colour = new int[n];                     // 0 = uncoloured; 1 and -1 are the two groups
    for (int start = 0; start < n; start++) {
        if (colour[start] != 0) continue;           // the graph may be disconnected: colour each component
        colour[start] = 1;
        ArrayDeque<Integer> queue = new ArrayDeque<>(List.of(start));
        while (!queue.isEmpty()) {
            int u = queue.poll();
            for (int v : graph[u]) {
                if (colour[v] == 0) {
                    colour[v] = -colour[u];
                    queue.offer(v);
                } else if (colour[v] == colour[u]) {
                    return false;                   // an edge inside one group
                }
            }
        }
    }
    return true;
}
```

**Restarting from every uncoloured vertex** handles disconnected graphs. Forgetting that is the classic bug.

---

## 7. Detecting cycles

* **Undirected graphs:** during a DFS, an edge to an already-visited vertex **other than the one you came from** closes a cycle. (Or use union-find: an edge joining two vertices already in the same component is a cycle, Ch 24.)
* **Directed graphs:** "already visited" isn't enough, because reaching a finished vertex again along a different branch isn't a cycle. Use **three states**: unvisited, *on the current DFS path*, and finished. **An edge into a vertex on the current path is a cycle**, since that vertex can reach you and you can reach it.

```java
// state: 0 = unvisited, 1 = on the current DFS path, 2 = finished
static boolean hasCycle(int u, List<List<Integer>> adj, int[] state) {
    state[u] = 1;
    for (int v : adj.get(u)) {
        if (state[v] == 1) return true;                       // back into the current path
        if (state[v] == 0 && hasCycle(v, adj, state)) return true;
    }
    state[u] = 2;                                            // u and everything below it are cycle-free
    return false;
}
```

Call it from every unvisited vertex. Ch 22 turns the finishing order of this same DFS into a topological sort. Find Eventual Safe States (practice) is exactly "which vertices can't reach a cycle?".

---

## 8. BFS on augmented state

### Worked G6 · [Shortest Path in a Grid with Obstacles Elimination](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/)

> Walk from the top-left to the bottom-right corner of a grid of 0s (empty) and 1s (obstacles), and you may remove up to `k` obstacles. Return the minimum number of steps, or −1. (Grid ≤ 40 × 40, k ≤ 1600.)

A plain grid BFS fails, because **arriving at a cell with more eliminations left is a genuinely different situation** from arriving with fewer, even at the same distance. Apply §4's question: *what must the state record?* The answer is **(row, column, eliminations left)**. BFS over those states:

```java
public int shortestPath(int[][] grid, int k) {
    int rows = grid.length, cols = grid[0].length;
    if (k >= rows + cols - 2) return rows + cols - 2;          // enough to walk straight there, through anything
    boolean[][][] seen = new boolean[rows][cols][k + 1];      // state = (row, col, eliminations left)
    ArrayDeque<int[]> queue = new ArrayDeque<>();
    queue.offer(new int[] {0, 0, k});
    seen[0][0][k] = true;
    int[] dr = {-1, 1, 0, 0}, dc = {0, 0, -1, 1};
    for (int steps = 0; !queue.isEmpty(); steps++) {
        for (int size = queue.size(); size > 0; size--) {
            int[] s = queue.poll();
            if (s[0] == rows - 1 && s[1] == cols - 1) return steps;
            for (int d = 0; d < 4; d++) {
                int nr = s[0] + dr[d], nc = s[1] + dc[d];
                if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                int left = s[2] - grid[nr][nc];                  // stepping on an obstacle uses one elimination
                if (left < 0 || seen[nr][nc][left]) continue;
                seen[nr][nc][left] = true;
                queue.offer(new int[] {nr, nc, left});
            }
        }
    }
    return -1;
}
```

The number of states is R · C · (k + 1), and each has 4 edges. The early exit for large k keeps the k dimension small in practice, because the shortest possible path, R + C − 2 steps, needs at most that many eliminations. **"Add the extra information to the state" is the general technique**: keys collected (a bitmask, Ch 17), fuel left, the parity of a step count, the last move made. Ch 23 does the same with weighted edges.

---

## 9. Where BFS and DFS break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Edges have **weights** | BFS's layers no longer equal distances | Dijkstra; 0-1 BFS for weights in {0, 1} (Ch 23) |
| Marking visited on dequeue | Vertices are enqueued many times; time blows up | Mark on enqueue |
| Deep recursion (10⁵ vertices) | `StackOverflowError` | Iterative BFS/DFS |
| A state space that's too big | (row, col, keys, …) multiplies quickly | Shrink the state; prune dominated states; bidirectional BFS |
| Directed cycle detection with a single visited flag | False cycles on cross edges | Three states (§7) |
| "Connected after many edge additions" | A BFS per query | Union-find (Ch 24) |

---

## 10. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| Shortest path with costs | Dijkstra (Ch 23) |
| An order that respects dependencies | Topological sort (Ch 22) |
| Edges whose removal disconnects the graph | Bridges via low-link (Ch 25) |
| Visiting every vertex once (all nodes, n ≤ 12) | BFS over (vertex, visited-set mask) (stretch) |
| Mutual reachability in directed graphs | Strongly connected components (Ch 25) |

---

## 11. Recognition

**The first question:** *what are the vertices, and what are the edges?* Write the model down before choosing an algorithm.

**Signals:**
* Grids, mazes, boards → cells are vertices.
* "**Minimum number of moves / steps / operations / transformations**", each costing 1 → **BFS** over states.
* "**Connected**", "islands", "regions", "provinces", "groups" → components (BFS/DFS, or DSU).
* "Can reach", "escape", "from the border" → reachability, often **reversed**.
* "Nearest of many sources" → **multi-source BFS**.
* "Two groups / teams / colours, no conflict on any edge" → **bipartite**.
* "Dependencies", "prerequisites", "deadlock", "loop" → directed cycles / DAGs (Ch 22).

**False friends:** "shortest path" with weights isn't BFS (Ch 23). "Fewest operations" where operations have different costs isn't BFS either.

---

## 12. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Marking visited when dequeued | Mark when enqueued |
| Forgetting disconnected parts (bipartite, counting components) | Loop over every vertex as a possible start |
| Recursive flood fill on a large grid | BFS or an explicit stack |
| `int[]` states in a `HashSet` | Encode as `int`/`long` (Ch 4 §15), or use a `boolean[][][]` |
| 1-indexed input used directly as indices | Subtract 1 when reading |
| Undirected edges added in one direction only | Add both directions |
| Level counting off by one | Count in whole layers, using `queue.size()` |

---

## 13. Practice

1. **[Max Area of Island](https://leetcode.com/problems/max-area-of-island/)** *(Medium)*
2. **[Number of Provinces](https://leetcode.com/problems/number-of-provinces/)** *(Medium)*
3. **[Surrounded Regions](https://leetcode.com/problems/surrounded-regions/)** *(Medium)*
4. **[01 Matrix](https://leetcode.com/problems/01-matrix/)** *(Medium)*
5. **[Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/)** *(Medium)*
6. **[Find Eventual Safe States](https://leetcode.com/problems/find-eventual-safe-states/)** *(Medium)*
7. **[Shortest Bridge](https://leetcode.com/problems/shortest-bridge/)** *(Medium)*
8. **[Snakes and Ladders](https://leetcode.com/problems/snakes-and-ladders/)** *(Medium)*
9. **[Jump Game IV](https://leetcode.com/problems/jump-game-iv/)** *(Hard)*
10. **[Detonate the Maximum Bombs](https://leetcode.com/problems/detonate-the-maximum-bombs/)** *(Medium)*
11. **[Sliding Puzzle](https://leetcode.com/problems/sliding-puzzle/)** *(Hard)*

**Stretch:** **[Bus Routes](https://leetcode.com/problems/bus-routes/)** *(Hard)* · **[Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/)** *(Hard)* · **[Making A Large Island](https://leetcode.com/problems/making-a-large-island/)** *(Hard)* · **[Shortest Path to Get All Keys](https://leetcode.com/problems/shortest-path-to-get-all-keys/)** *(Hard)*

<details><summary>Nudges</summary>

1. Max Area: G1, returning each component's size.
2. Provinces: an adjacency matrix; count the components with DFS/BFS.
3. Surrounded: flood from the border's `O`s and mark them safe; flip every other `O`.
4. 01 Matrix: multi-source BFS from all the 0s.
5. Pacific Atlantic: two reverse floods, from each ocean's border, moving to cells of **equal or greater** height; take the intersection.
6. Safe States: a vertex is safe exactly when it can't reach a cycle. Use the three-state DFS and keep the vertices that finish with state 2. (Or a reverse topological sort, Ch 22.)
7. Shortest Bridge: find one island (flood fill), then multi-source BFS from all its cells until you reach the other island.
8. Snakes and Ladders: squares 1…n² are the vertices; the edges are dice rolls 1–6, following any snake or ladder. The hard part is converting a square number to (row, col) on the boustrophedon board.
9. Jump Game IV: also jump between indices with equal values. Group indices by value, and **clear each group after using it once**, or the run is O(n²).
10. Bombs: a directed edge i → j if j lies within i's radius (compare squared distances in `long`). BFS from each bomb.
11. Sliding Puzzle: the state is the board as a 6-character string; the moves swap 0 with a neighbour. BFS over at most 720 states.
12. Stretch, Bus Routes: BFS over *routes*, not stops. Two routes are connected if they share a stop; map each stop to its routes.
13. Stretch, Visiting All Nodes: the state is (current node, visited mask). Multi-source BFS starting from every node.
14. Stretch, Large Island: label each island with an id and its size, then for each 0 cell, add up the sizes of the *distinct* neighbouring islands, plus 1.
15. Stretch, All Keys: the state is (row, col, keys mask). Locks need their key's bit.
</details>

<details><summary>Key</summary>

Max Area → G1 · Provinces → G1 on a matrix · Surrounded → reverse flood from the border · 01 Matrix → G2 · Pacific Atlantic → reverse flood from both oceans · Safe States → three-state DFS (§7) / reverse topological order · Shortest Bridge → G1 then G2 · Snakes → G3 with index ↔ cell mapping · Jump Game IV → BFS with value groups cleared after use · Bombs → directed reachability from each bomb · Sliding Puzzle → G3 on board strings · Bus Routes → BFS over routes · Visiting All Nodes → G6 (node, mask) · Large Island → component labels + distinct neighbours · All Keys → G6 (cell, keys mask)
</details>

---

## 14. Self-test

<details><summary>1. Why does BFS give shortest paths in unweighted graphs but not in weighted ones?</summary>

BFS processes vertices in order of edge count. With unit weights, edge count *is* distance. With weights, a path with more edges can be cheaper.
</details>

<details><summary>2. Why mark vertices as visited when they're enqueued rather than when they're dequeued?</summary>

Otherwise a vertex can be enqueued once for each neighbour before it's processed, which can mean O(E) queue entries per vertex.
</details>

<details><summary>3. In directed cycle detection, why doesn't a single "visited" flag work?</summary>

Reaching an already-*finished* vertex through another branch (a cross edge) isn't a cycle. Only an edge into a vertex on the *current* path is, which is why you need the "on path" state.
</details>

<details><summary>4. In the obstacle-elimination BFS, why must the remaining eliminations be part of the state?</summary>

Reaching a cell with more eliminations left gives strictly more options afterwards, so the same cell with different counts is a different situation. A plain cell-visited check would discard better states.
</details>

<details><summary>5. What's the cost of generating Word Ladder neighbours by comparing all pairs of words, versus trying 26 letters per position?</summary>

Pairs: O(n² · L). Letter substitution: O(n · 26 · L), plus the cost of building and hashing each candidate. For thousands of words, the second is far cheaper.
</details>

---

## 15. Exit gate

Move on when you can:

* build all three representations from an edge list, and write BFS (with distances) and DFS (recursive and iterative) from blank files;
* prove BFS's layer property, and explain mark-on-enqueue;
* model a new "minimum moves" problem as a state graph, stating what a state must contain;
* write Number of Islands, Rotting Oranges, Is Graph Bipartite? and the three-state cycle check, each in under 10 minutes;
* solve 9 of the 11 practice problems.
