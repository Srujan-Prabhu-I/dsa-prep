# Appendix A — The Java Template Library

> Every reusable piece of code in this handbook, in one place, in the form each chapter derived and verified. **This is a rebuild list, not a copy-paste kit.** The curriculum §1 benchmark asks you to rewrite every template here from a blank file, compiling on the first try, within the target time.

**How to use it:**
* Each template gives its **chapter** (for the derivation and the proof), **when** to use it, and a **rebuild target** time.
* Once a week in Phase 6, pick five templates at random and rebuild them in a plain editor (Ch 39 §8). Log any that miss the target, and add them to the re-solve schedule.
* Templates marked *(S)* come from S-tier chapters.

**Imports for everything below:** `import java.io.*; import java.util.*;`

---

## Contents

| # | Template | Ch | Target |
| :-- | :--- | :-: | :-: |
| A1 | OA skeleton + FastReader + output | 4 | 4 min |
| A2 | Big-stack thread | 4 | 1 min |
| A3 | Stress harness | 5 | 5 min |
| A4 | Boundary binary search, binary search on the answer | 10 | 3 min |
| A5 | Sorting with comparators | 4, 9 | 2 min |
| A6 | Prefix sums + hash counts; difference array | 8 | 3 min |
| A7 | Two pointers; variable sliding window | 11 | 4 min |
| A8 | Monotonic stack; monotonic deque | 12 | 4 min |
| A9 | Backtracking (subsets, permutations) | 6 | 4 min |
| A10 | Heap top-k; TreeMap multiset | 15 | 3 min |
| A11 | Trie | 20 | 4 min |
| A12 | Graph building, BFS, grid BFS, iterative DFS | 21 | 5 min |
| A13 | Topological sort (Kahn) | 22 | 4 min |
| A14 | Dijkstra; 0-1 BFS; Bellman–Ford; Floyd–Warshall | 23 | 8 min |
| A15 | DSU; Kruskal | 24 | 5 min |
| A16 | Bridges and articulation points | 25 | 6 min |
| A17 | DP skeletons: memo, table, bitmask, digit DP, LIS | 27–32 | 8 min |
| A18 | Math: gcd, power, binomials, sieve | 18 | 5 min |
| A19 | Fenwick; segment trees; sparse table *(S)* | 34 | 12 min |
| A20 | Prefix function, KMP, Z, rolling hash *(S)* | 35 | 10 min |
| A21 | Binary lifting LCA; Euler tour *(S)* | 36 | 10 min |

---

## A1 · OA skeleton, FastReader, output (Ch 4 §2–3)

```java
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        FastReader in = new FastReader(System.in);
        PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(System.out)));
        int t = in.nextInt();
        while (t-- > 0) {
            int n = in.nextInt();
            long[] a = new long[n];
            for (int i = 0; i < n; i++) a[i] = in.nextLong();
            out.println(solve(a));
        }
        out.flush();                                  // forget this and nothing is printed
    }

    static long solve(long[] a) { return 0; }

    static final class FastReader {
        private final BufferedReader reader;
        private StringTokenizer tokens;
        FastReader(InputStream stream) { reader = new BufferedReader(new InputStreamReader(stream), 1 << 16); }
        String next() throws IOException {
            while (tokens == null || !tokens.hasMoreTokens()) {
                String line = reader.readLine();
                if (line == null) return null;        // end of input
                tokens = new StringTokenizer(line);
            }
            return tokens.nextToken();
        }
        int nextInt() throws IOException { return Integer.parseInt(next()); }
        long nextLong() throws IOException { return Long.parseLong(next()); }
        double nextDouble() throws IOException { return Double.parseDouble(next()); }
        String nextLine() throws IOException { tokens = null; return reader.readLine(); }
    }
}
```

**When:** every stdin/stdout OA and CSES problem. Space-separated output: build it in a `StringBuilder`, never with `Arrays.toString`.

## A2 · Big-stack thread (Ch 4 §16)

```java
public static void main(String[] args) throws InterruptedException {
    Thread solver = new Thread(null, () -> {
        try { run(); } catch (IOException e) { throw new UncheckedIOException(e); }
    }, "solver", 1 << 26);                            // 64 MB of stack
    solver.start();
    solver.join();
}
```

**When:** a recursive DFS whose depth can reach 10⁵ in a program you control. On LeetCode, go iterative instead.

## A3 · Stress harness (Ch 5 §6)

```java
public static void main(String[] args) {
    long seed = args.length > 0 ? Long.parseLong(args[0]) : System.nanoTime();
    Random rng = new Random(seed);
    for (int iter = 1; iter <= 100_000; iter++) {
        int n = 1 + rng.nextInt(Math.min(10, 1 + iter / 5_000));
        int[] a = new int[n];
        for (int i = 0; i < n; i++) a[i] = rng.nextInt(7) - 3;        // tiny values: ties, zeros, negatives
        int expected = brute(a.clone()), actual = solve(a.clone());
        if (expected != actual) {
            System.out.println("seed " + seed + " input " + Arrays.toString(a) + " expected " + expected + " actual " + actual);
            return;
        }
    }
    System.out.println("all passed (seed " + seed + ")");
}
```

**When:** after every non-trivial solution, and whenever a hidden test fails (Ch 39 §6). For problems with several valid answers, replace the comparison with a checker (Ch 5 §6.3).

## A4 · Binary search (Ch 10)

```java
// Invariant: ok(lo) is false, ok(hi) is true. lo and hi may be virtual sentinels; ok is never called on them.
long lo = BAD, hi = GOOD;
while (hi - lo > 1) {
    long mid = lo + (hi - lo) / 2;
    if (ok(mid)) hi = mid; else lo = mid;
}
// hi = the first value with ok true; lo = the last value with ok false

// Lower bound: the first index with a[i] >= x (a.length if none). For "> x", change the comparison.
static int lowerBound(int[] a, int x) {
    int lo = -1, hi = a.length;
    while (hi - lo > 1) {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] >= x) hi = mid; else lo = mid;
    }
    return hi;
}
```

**When:** a sorted array (index search), or **feasibility monotone in the answer** (search on the answer: capacity, time, distance). For "the largest value that works", swap the roles: `ok(lo)` true, `ok(hi)` false, return `lo`.

## A5 · Sorting with comparators (Ch 4 §14, Ch 9)

```java
Arrays.sort(intervals, (x, y) -> Integer.compare(x[0], y[0]));               // int[][] by column 0
Arrays.sort(intervals, (x, y) -> x[0] != y[0] ? Integer.compare(x[0], y[0])
                                              : Integer.compare(y[1], x[1])); // then column 1 descending
Integer[] order = new Integer[n];                                             // sort indices by a key
for (int i = 0; i < n; i++) order[i] = i;
Arrays.sort(order, (x, y) -> Integer.compare(key[x], key[y]));
```

**Never** write `(x, y) -> x - y`: it overflows (Ch 4 §14). `Arrays.sort(int[])` takes no comparator; sort `Integer[]` or `int[][]` instead.

## A6 · Prefix sums + hash counts; difference array (Ch 8)

```java
// The number of subarrays with sum exactly k (works with negatives).
static int subarraySum(int[] nums, int k) {
    Map<Long, Integer> seen = new HashMap<>();
    seen.put(0L, 1);                                   // the empty prefix
    long prefix = 0;
    int count = 0;
    for (int x : nums) {
        prefix += x;
        count += seen.getOrDefault(prefix - k, 0);     // earlier prefixes P with prefix - P = k
        seen.merge(prefix, 1, Integer::sum);
    }
    return count;
}

// Difference array: many range additions, read once at the end.
long[] diff = new long[n + 1];
// add v to a[l..r]:  diff[l] += v; diff[r + 1] -= v;
// then a[i] = diff[0] + ... + diff[i] (a running sum)
```

**When:** a subarray property that is a function of two prefix states (sum, remainder, XOR, parity mask).

## A7 · Two pointers; variable sliding window (Ch 11)

```java
// The longest window satisfying a condition that shrinking can only restore (here: at most k distinct values).
static int longestAtMostKDistinct(int[] a, int k) {
    Map<Integer, Integer> count = new HashMap<>();
    int best = 0;
    for (int left = 0, right = 0; right < a.length; right++) {
        count.merge(a[right], 1, Integer::sum);                  // grow
        while (count.size() > k) {                               // shrink until valid again
            if (count.merge(a[left], -1, Integer::sum) == 0) count.remove(a[left]);
            left++;
        }
        best = Math.max(best, right - left + 1);                 // the window [left, right] is valid
    }
    return best;
}
```

**When:** the window's validity is monotone (a valid window's sub-windows are valid). **Exactly k** = atMost(k) − atMost(k − 1). For pairs in a sorted array, move whichever pointer can only improve the answer (Ch 11 §2).

## A8 · Monotonic stack; monotonic deque (Ch 12)

```java
// For each i, the index of the next strictly greater element to the right (-1 if none).
static int[] nextGreater(int[] a) {
    int n = a.length;
    int[] answer = new int[n], stack = new int[n];
    Arrays.fill(answer, -1);
    int size = 0;                                                // indices whose values are non-increasing
    for (int i = 0; i < n; i++) {
        while (size > 0 && a[stack[size - 1]] < a[i]) answer[stack[--size]] = i;
        stack[size++] = i;
    }
    return answer;
}

// The maximum of every window of size k.
static int[] windowMax(int[] a, int k) {
    int[] result = new int[a.length - k + 1];
    ArrayDeque<Integer> dq = new ArrayDeque<>();                 // indices, values strictly decreasing
    for (int i = 0; i < a.length; i++) {
        if (!dq.isEmpty() && dq.peekFirst() <= i - k) dq.pollFirst();       // expired
        while (!dq.isEmpty() && a[dq.peekLast()] <= a[i]) dq.pollLast();   // dominated
        dq.addLast(i);
        if (i >= k - 1) result[i - k + 1] = a[dq.peekFirst()];
    }
    return result;
}
```

**When:** nearest greater/smaller, spans, histogram areas, contributions (strict on one side, non-strict on the other for ties); window extremes; DP over the last k states (Ch 33).

## A9 · Backtracking (Ch 6)

```java
// Choose → explore → un-choose. All subsets of nums (each index: take it or skip it).
static void subsets(int[] nums, int i, List<Integer> chosen, List<List<Integer>> out) {
    if (i == nums.length) { out.add(new ArrayList<>(chosen)); return; }   // copy: chosen keeps changing
    chosen.add(nums[i]);
    subsets(nums, i + 1, chosen, out);
    chosen.remove(chosen.size() - 1);                            // un-choose
    subsets(nums, i + 1, chosen, out);
}

// All permutations, with a used[] array.
static void permute(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> out) {
    if (path.size() == nums.length) { out.add(new ArrayList<>(path)); return; }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        used[i] = true; path.add(nums[i]);
        permute(nums, used, path, out);
        used[i] = false; path.remove(path.size() - 1);
    }
}
```

**When:** "list every …", n ≤ ~20 for subsets, ≤ ~10 for permutations. Duplicates: sort first and skip `nums[i] == nums[i − 1]` when `i − 1` isn't used (Ch 6).

## A10 · Heap top-k; TreeMap multiset (Ch 15)

```java
// The k largest values: a min-heap of size k; its top is the k-th largest.
static int kthLargest(int[] a, int k) {
    PriorityQueue<Integer> heap = new PriorityQueue<>();
    for (int x : a) {
        heap.offer(x);
        if (heap.size() > k) heap.poll();                        // drop the smallest
    }
    return heap.peek();
}

// A multiset: value -> count.
TreeMap<Integer, Integer> ms = new TreeMap<>();
// add x:     ms.merge(x, 1, Integer::sum);
// remove x:  if (ms.merge(x, -1, Integer::sum) == 0) ms.remove(x);
// min / max: ms.firstKey(), ms.lastKey();  neighbours: ms.floorKey(x), ms.ceilingKey(x)
```

**When:** a heap for "the current best" with insertions and removals of the best only; the multiset when arbitrary elements are removed or neighbours are needed.

## A11 · Trie (Ch 20)

```java
class Trie {
    private static final class Node {
        final Node[] child = new Node[26];
        boolean isWord;
    }
    private final Node root = new Node();

    void insert(String word) {
        Node node = root;
        for (int i = 0; i < word.length(); i++) {
            int c = word.charAt(i) - 'a';
            if (node.child[c] == null) node.child[c] = new Node();
            node = node.child[c];
        }
        node.isWord = true;
    }

    boolean search(String word) { Node node = walk(word); return node != null && node.isWord; }
    boolean startsWith(String prefix) { return walk(prefix) != null; }

    private Node walk(String s) {
        Node node = root;
        for (int i = 0; i < s.length() && node != null; i++) node = node.child[s.charAt(i) - 'a'];
        return node;
    }
}
```

**When:** prefix lookups across many words; walking a grid or string in step with the dictionary. A binary trie (2 children, highest bit first) answers maximum-XOR questions (Ch 20).

## A12 · Graphs: building, BFS, grid BFS, iterative DFS (Ch 21)

```java
static List<List<Integer>> adjacencyList(int n, int[][] edges, boolean directed) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int[] e : edges) {
        adj.get(e[0]).add(e[1]);
        if (!directed) adj.get(e[1]).add(e[0]);
    }
    return adj;
}

static int[] bfs(List<List<Integer>> adj, int source) {
    int[] dist = new int[adj.size()];
    Arrays.fill(dist, -1);
    ArrayDeque<Integer> queue = new ArrayDeque<>();
    dist[source] = 0;                                            // mark when ENQUEUED
    queue.offer(source);
    while (!queue.isEmpty()) {
        int u = queue.poll();
        for (int v : adj.get(u)) {
            if (dist[v] != -1) continue;
            dist[v] = dist[u] + 1;
            queue.offer(v);
        }
    }
    return dist;
}

static final int[][] DIRS = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

// Steps from (sr, sc) to every open cell ('.') of a grid; -1 if unreachable.
static int[][] gridBfs(char[][] grid, int sr, int sc) {
    int rows = grid.length, cols = grid[0].length;
    int[][] dist = new int[rows][cols];
    for (int[] row : dist) Arrays.fill(row, -1);
    ArrayDeque<int[]> queue = new ArrayDeque<>();
    dist[sr][sc] = 0;
    queue.offer(new int[] {sr, sc});
    while (!queue.isEmpty()) {
        int[] cell = queue.poll();
        for (int[] d : DIRS) {
            int r = cell[0] + d[0], c = cell[1] + d[1];
            if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] != '.' || dist[r][c] != -1) continue;
            dist[r][c] = dist[cell[0]][cell[1]] + 1;
            queue.offer(new int[] {r, c});
        }
    }
    return dist;
}

static boolean[] reachable(List<List<Integer>> adj, int source) {   // DFS with an explicit stack
    boolean[] seen = new boolean[adj.size()];
    ArrayDeque<Integer> stack = new ArrayDeque<>();
    seen[source] = true;
    stack.push(source);
    while (!stack.isEmpty()) {
        int u = stack.pop();
        for (int v : adj.get(u)) if (!seen[v]) { seen[v] = true; stack.push(v); }
    }
    return seen;
}
```

**When:** BFS for fewest steps (unweighted) and multi-source spreading (enqueue every source at distance 0); DFS for reachability, components and flood fill. State-space BFS: the "vertex" is a tuple (cell, keys, steps left), encoded as an int or a record (Ch 21 §8).

## A13 · Topological sort, Kahn (Ch 22)

```java
// Edges u -> v mean "u before v". Returns an order, or an empty array if there's a cycle.
static int[] topoOrder(int n, List<List<Integer>> after) {
    int[] indegree = new int[n];
    for (int u = 0; u < n; u++) for (int v : after.get(u)) indegree[v]++;
    ArrayDeque<Integer> ready = new ArrayDeque<>();
    for (int u = 0; u < n; u++) if (indegree[u] == 0) ready.offer(u);
    int[] order = new int[n];
    int taken = 0;
    while (!ready.isEmpty()) {
        int u = ready.poll();
        order[taken++] = u;
        for (int v : after.get(u)) if (--indegree[v] == 0) ready.offer(v);
    }
    return taken == n ? order : new int[0];
}
```

**When:** dependencies, cycle detection in directed graphs, and any DP on a DAG (process in this order). Write the edge's meaning in a comment: `[a, b]` usually means b → a.

## A14 · Shortest paths (Ch 23)

```java
// Dijkstra, non-negative weights. adj.get(u) holds {v, w}.
static long[] dijkstra(List<List<int[]>> adj, int source) {
    long[] dist = new long[adj.size()];
    Arrays.fill(dist, Long.MAX_VALUE);
    dist[source] = 0;
    PriorityQueue<long[]> pq = new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));   // {dist, node}
    pq.offer(new long[] {0, source});
    while (!pq.isEmpty()) {
        long[] top = pq.poll();
        long d = top[0];
        int u = (int) top[1];
        if (d > dist[u]) continue;                               // stale entry
        for (int[] e : adj.get(u)) {
            long nd = d + e[1];
            if (nd < dist[e[0]]) { dist[e[0]] = nd; pq.offer(new long[] {nd, e[0]}); }
        }
    }
    return dist;
}

// 0-1 BFS: weights 0 or 1. A 0-edge pushes to the front, a 1-edge to the back.
static int[] zeroOneBfs(List<List<int[]>> adj, int source) {
    int[] dist = new int[adj.size()];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[source] = 0;
    ArrayDeque<Integer> dq = new ArrayDeque<>();
    dq.offerFirst(source);
    while (!dq.isEmpty()) {
        int u = dq.pollFirst();
        for (int[] e : adj.get(u)) {
            int v = e[0], w = e[1];
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                if (w == 0) dq.offerFirst(v); else dq.offerLast(v);
            }
        }
    }
    return dist;
}

// Bellman–Ford: negative edges; edges {u, v, w}. Returns null if a negative cycle is reachable.
static long[] bellmanFord(int n, int[][] edges, int source) {
    final long INF = Long.MAX_VALUE / 4;
    long[] dist = new long[n];
    Arrays.fill(dist, INF);
    dist[source] = 0;
    for (int round = 1; round < n; round++)
        for (int[] e : edges)
            if (dist[e[0]] != INF && dist[e[0]] + e[2] < dist[e[1]]) dist[e[1]] = dist[e[0]] + e[2];
    for (int[] e : edges)
        if (dist[e[0]] != INF && dist[e[0]] + e[2] < dist[e[1]]) return null;    // still improving
    return dist;
}

// Floyd–Warshall: all pairs, V <= ~400. dist[i][i] = 0, missing edges = INF (with INF + INF safe).
for (int k = 0; k < n; k++)                                      // k outermost
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            if (dist[i][k] + dist[k][j] < dist[i][j]) dist[i][j] = dist[i][k] + dist[k][j];
```

**When:** Dijkstra for non-negative weights (and for max/min path costs, Ch 23 §3); 0-1 BFS for 0/1 weights; Bellman–Ford for negative edges or an edge-count limit (relax from a copy of the previous round, Ch 23 §6); Floyd–Warshall for all pairs at small V. Augmented states: `dist[state][node]` (Ch 23 §5).

## A15 · DSU; Kruskal (Ch 24)

```java
final class DSU {
    private final int[] parent, size;
    DSU(int n) {
        parent = new int[n];
        size = new int[n];
        for (int i = 0; i < n; i++) { parent[i] = i; size[i] = 1; }
    }
    int find(int x) {
        int root = x;
        while (parent[root] != root) root = parent[root];
        while (parent[x] != root) { int next = parent[x]; parent[x] = root; x = next; }   // path compression
        return root;
    }
    boolean union(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;
        if (size[ra] < size[rb]) { int t = ra; ra = rb; rb = t; }                         // union by size
        parent[rb] = ra;
        size[ra] += size[rb];
        return true;
    }
    int sizeOf(int x) { return size[find(x)]; }
}

static long kruskal(int n, int[][] edges) {                     // edges {u, v, w}; -1 if disconnected
    int[][] sorted = edges.clone();
    Arrays.sort(sorted, (a, b) -> Integer.compare(a[2], b[2]));
    DSU dsu = new DSU(n);
    long total = 0;
    int used = 0;
    for (int[] e : sorted) {
        if (dsu.union(e[0], e[1])) {
            total += e[2];
            if (++used == n - 1) break;
        }
    }
    return used == n - 1 ? total : -1;
}
```

**When:** dynamic connectivity with merges only, grouping by equivalence, cycle detection while adding undirected edges, MST. Deletions: reverse time (Ch 37 §4).

## A16 · Bridges and articulation points (Ch 25)

```java
// adj.get(u) holds {v, edgeId}. tin[] filled with -1 before the first call; call dfs(root, -1).
int timer;
int[] tin, low;

void dfs(int u, int parentEdge, List<List<int[]>> adj, List<int[]> bridges, boolean[] isCut) {
    tin[u] = low[u] = timer++;
    int children = 0;
    for (int[] e : adj.get(u)) {
        int v = e[0], id = e[1];
        if (id == parentEdge) continue;                          // skip the arrival EDGE, not the vertex
        if (tin[v] != -1) {
            low[u] = Math.min(low[u], tin[v]);                   // back edge
        } else {
            children++;
            dfs(v, id, adj, bridges, isCut);
            low[u] = Math.min(low[u], low[v]);
            if (low[v] > tin[u]) bridges.add(new int[] {u, v});
            if (parentEdge != -1 && low[v] >= tin[u]) isCut[u] = true;
        }
    }
    if (parentEdge == -1 && children >= 2) isCut[u] = true;     // the root rule
}
```

**When:** "removing one edge / vertex disconnects". Recursion depth can reach V: use A2 in an OA.

## A17 · DP skeletons (Ch 27–32)

```java
// Top-down: memo array, sentinel for "not computed".
long[] memo;                                                     // Arrays.fill(memo, -1) before the first call
long best(int i) {
    if (i >= n) return 0;                                        // base case
    if (memo[i] != -1) return memo[i];
    long result = /* combine best(smaller states) over the last decision */ 0;
    return memo[i] = result;
}

// Bottom-up, rolled to two rows: dp[i][j] needs only row i - 1.
long[] prev = new long[m + 1], cur = new long[m + 1];
for (int i = 1; i <= n; i++) {
    for (int j = 0; j <= m; j++) cur[j] = /* from prev[...] and cur[j - 1] */ 0;
    long[] t = prev; prev = cur; cur = t;
}

// Bitmask DP: the set of used items is the state; smaller masks first.
int[] dp = new int[1 << n];
Arrays.fill(dp, INF);
dp[0] = 0;
for (int mask = 0; mask < (1 << n); mask++) {
    if (dp[mask] == INF) continue;
    for (int j = 0; j < n; j++)
        if ((mask >> j & 1) == 0) dp[mask | 1 << j] = Math.min(dp[mask | 1 << j], dp[mask] + cost(mask, j));
}

// Digit DP: count numbers in [0, N] by a digit property. Memoise only when !tight && started.
long count(int pos, int state, boolean tight, boolean started) {
    if (pos == digits.length) return started && accept(state) ? 1 : 0;
    if (!tight && started && memo[pos][state] != -1) return memo[pos][state];
    int limit = tight ? digits[pos] - '0' : 9;
    long ways = 0;
    for (int d = 0; d <= limit; d++) {
        boolean nextTight = tight && d == limit;
        if (!started && d == 0) ways += count(pos + 1, state, nextTight, false);
        else ways += count(pos + 1, next(state, d), nextTight, true);
    }
    if (!tight && started) memo[pos][state] = ways;
    return ways;
}

// LIS in O(n log n): tails[k] = the smallest tail of an increasing subsequence of length k + 1.
static int lengthOfLIS(int[] nums) {
    int[] tails = new int[nums.length];
    int size = 0;
    for (int x : nums) {
        int lo = -1, hi = size;                                  // the first k with tails[k] >= x
        while (hi - lo > 1) {
            int mid = lo + (hi - lo) / 2;
            if (tails[mid] >= x) hi = mid; else lo = mid;
        }
        tails[hi] = x;
        if (hi == size) size++;
    }
    return size;
}
```

**When:** write the six steps of Ch 27 §1 first (state in English, transition, base, order, answer, cost); the skeleton is only the container. Counting DPs: `% MOD` at every addition. Transitions that loop over a range of states: Ch 33's shapes.

## A18 · Math (Ch 18)

```java
static long gcd(long a, long b) { return b == 0 ? a : gcd(b, a % b); }
static long lcm(long a, long b) { return a / gcd(a, b) * b; }    // divide first

static long power(long base, long exp, long mod) {
    long result = 1 % mod;
    base %= mod;
    if (base < 0) base += mod;
    while (exp > 0) {
        if ((exp & 1) == 1) result = result * base % mod;
        base = base * base % mod;
        exp >>= 1;
    }
    return result;
}

final class Binomial {                                           // C(n, k) mod a prime, O(1) per query
    static final long MOD = 1_000_000_007L;
    private final long[] fact, invFact;
    Binomial(int maxN) {
        fact = new long[maxN + 1];
        invFact = new long[maxN + 1];
        fact[0] = 1;
        for (int i = 1; i <= maxN; i++) fact[i] = fact[i - 1] * i % MOD;
        invFact[maxN] = power(fact[maxN], MOD - 2, MOD);          // Fermat
        for (int i = maxN; i > 0; i--) invFact[i - 1] = invFact[i] * i % MOD;
    }
    long choose(int n, int k) {
        if (k < 0 || k > n) return 0;
        return fact[n] * invFact[k] % MOD * invFact[n - k] % MOD;
    }
}

static boolean[] sieve(int n) {                                  // isPrime[0..n]
    boolean[] isPrime = new boolean[n + 1];
    Arrays.fill(isPrime, 2, n + 1, true);
    for (int i = 2; (long) i * i <= n; i++)
        if (isPrime[i]) for (int j = i * i; j <= n; j += i) isPrime[j] = false;
    return isPrime;
}
```

**When:** modular counting, fast exponentiation, binomials for counting paths and arrangements, primes up to 10⁷. Subtraction mod p: `(a - b + MOD) % MOD`.

## A19 · Range queries *(S)* (Ch 34)

```java
class Fenwick {                                                  // 1-based; point add, prefix sum
    private final long[] tree;
    Fenwick(int n) { tree = new long[n + 1]; }
    void add(int i, long delta) { for (; i < tree.length; i += i & -i) tree[i] += delta; }
    long prefix(int i) { long s = 0; for (; i > 0; i -= i & -i) s += tree[i]; return s; }
}

// Recursive segment tree with lazy "add to range", answering range minimum. Call with (1, 0, n - 1, ...).
long[] min = new long[4 * n], add = new long[4 * n];
void push(int node) {
    if (add[node] == 0) return;
    for (int c = 2 * node; c <= 2 * node + 1; c++) { min[c] += add[node]; add[c] += add[node]; }
    add[node] = 0;
}
void update(int node, int l, int r, int ql, int qr, long v) {
    if (qr < l || r < ql) return;
    if (ql <= l && r <= qr) { min[node] += v; add[node] += v; return; }
    push(node);
    int mid = (l + r) / 2;
    update(2 * node, l, mid, ql, qr, v);
    update(2 * node + 1, mid + 1, r, ql, qr, v);
    min[node] = Math.min(min[2 * node], min[2 * node + 1]);
}
long query(int node, int l, int r, int ql, int qr) {
    if (qr < l || r < ql) return Long.MAX_VALUE;
    if (ql <= l && r <= qr) return min[node];
    push(node);
    int mid = (l + r) / 2;
    return Math.min(query(2 * node, l, mid, ql, qr), query(2 * node + 1, mid + 1, r, ql, qr));
}

// Iterative segment tree: point update, range query on [lo, hi), commutative combine (max here).
int[] tree = new int[2 * size];                                  // leaves at tree[size + i]
void set(int i, int v) { for (tree[i += size] = v; i > 1; i /= 2) tree[i / 2] = Math.max(tree[i], tree[i ^ 1]); }
int maxOn(int lo, int hi) {
    int res = Integer.MIN_VALUE;
    for (lo += size, hi += size; lo < hi; lo /= 2, hi /= 2) {
        if ((lo & 1) == 1) res = Math.max(res, tree[lo++]);
        if ((hi & 1) == 1) res = Math.max(res, tree[--hi]);
    }
    return res;
}

// Sparse table: static range minimum in O(1).
int levels = 32 - Integer.numberOfLeadingZeros(n);
int[][] sp = new int[levels][];
sp[0] = a.clone();
for (int j = 1; j < levels; j++) {
    sp[j] = new int[n - (1 << j) + 1];
    for (int i = 0; i < sp[j].length; i++) sp[j][i] = Math.min(sp[j - 1][i], sp[j - 1][i + (1 << (j - 1))]);
}
// min of a[l..r]: int j = 31 - Integer.numberOfLeadingZeros(r - l + 1);
//                 Math.min(sp[j][l], sp[j][r - (1 << j) + 1])
```

**When:** the Ch 34 §8 table. The iterative tree starts filled with the combine's identity (here `Integer.MIN_VALUE`, or 0 when every value is ≥ 0).

## A20 · Strings *(S)* (Ch 35)

```java
static int[] prefixFunction(String s) {
    int n = s.length();
    int[] pi = new int[n];
    for (int i = 1; i < n; i++) {
        int k = pi[i - 1];
        while (k > 0 && s.charAt(i) != s.charAt(k)) k = pi[k - 1];
        if (s.charAt(i) == s.charAt(k)) k++;
        pi[i] = k;
    }
    return pi;
}

static int countOccurrences(String text, String pat) {          // KMP
    int[] pi = prefixFunction(pat);
    int k = 0, count = 0;
    for (int i = 0; i < text.length(); i++) {
        while (k > 0 && text.charAt(i) != pat.charAt(k)) k = pi[k - 1];
        if (text.charAt(i) == pat.charAt(k)) k++;
        if (k == pat.length()) { count++; k = pi[k - 1]; }
    }
    return count;
}

static int[] zFunction(String s) {
    int n = s.length();
    int[] z = new int[n];
    z[0] = n;
    for (int i = 1, l = 0, r = 0; i < n; i++) {
        if (i < r) z[i] = Math.min(r - i, z[i - l]);
        while (i + z[i] < n && s.charAt(z[i]) == s.charAt(i + z[i])) z[i]++;
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}

static final class Hasher {                                     // double polynomial hash, random base
    static final long M1 = 1_000_000_007L, M2 = 998_244_353L;
    static final long B = 911 + new Random().nextInt(100_000);
    final long[] h1, h2, p1, p2;
    Hasher(String s) {
        int n = s.length();
        h1 = new long[n + 1]; h2 = new long[n + 1]; p1 = new long[n + 1]; p2 = new long[n + 1];
        p1[0] = p2[0] = 1;
        for (int i = 0; i < n; i++) {
            h1[i + 1] = (h1[i] * B + s.charAt(i)) % M1;
            h2[i + 1] = (h2[i] * B + s.charAt(i)) % M2;
            p1[i + 1] = p1[i] * B % M1;
            p2[i + 1] = p2[i] * B % M2;
        }
    }
    long get(int l, int r) {                                     // s[l, r), both hashes in one long
        long a = ((h1[r] - h1[l] * p1[r - l]) % M1 + M1) % M1;
        long b = ((h2[r] - h2[l] * p2[r - l]) % M2 + M2) % M2;
        return a * M2 + b;
    }
}
```

**When:** borders, periods and matching (prefix function); "LCP of each suffix with the whole string" (Z); substring equality at scale (hash, with binary search on the length).

## A21 · Trees *(S)* (Ch 36)

```java
static final class Lca {                                         // parent[root] == root; depth from a BFS
    final int[][] up;
    final int[] depth;
    Lca(int[] parent, int[] depth) {
        int n = parent.length, levels = 32 - Integer.numberOfLeadingZeros(n);
        this.depth = depth;
        up = new int[levels][];
        up[0] = parent;
        for (int j = 1; j < levels; j++) {
            up[j] = new int[n];
            for (int v = 0; v < n; v++) up[j][v] = up[j - 1][up[j - 1][v]];
        }
    }
    int lift(int v, int k) { for (int j = 0; k > 0; j++, k >>= 1) if ((k & 1) == 1) v = up[j][v]; return v; }
    int lca(int a, int b) {
        if (depth[a] < depth[b]) { int t = a; a = b; b = t; }
        a = lift(a, depth[a] - depth[b]);
        if (a == b) return a;
        for (int j = up.length - 1; j >= 0; j--) if (up[j][a] != up[j][b]) { a = up[j][a]; b = up[j][b]; }
        return up[0][a];
    }
}
// dist(a, b) = depth[a] + depth[b] - 2 * depth[lca(a, b)]

// Euler tour (iterative preorder): subtree(u) = positions [tin[u], tin[u] + size[u] - 1].
int[] tin = new int[n], size = new int[n], order = new int[n], parent = new int[n], stack = new int[n];
int top = 0, timer = 0;
stack[top++] = root;
parent[root] = -1;
while (top > 0) {
    int u = stack[--top];
    tin[u] = timer;
    order[timer++] = u;
    for (int v : adj.get(u)) if (v != parent[u]) { parent[v] = u; stack[top++] = v; }
}
for (int i = n - 1; i >= 0; i--) {
    int u = order[i];
    size[u] += 1;
    if (parent[u] >= 0) size[parent[u]] += size[u];
}
```

**When:** many ancestor, LCA, distance or path queries on a static tree; subtree queries with updates (Euler tour + A19's Fenwick).
