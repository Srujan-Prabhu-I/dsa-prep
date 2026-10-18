# Chapter 23 — Shortest Paths

> **Part V · Weeks 20–21** · Prerequisites: Ch 15 (heaps, lazy deletion), Ch 21 (BFS and its layer proof), Ch 22 (DP in topological order), Ch 4 §2 (fast I/O for CSES)
> **Problems:** 7 worked (1 ★) · 8 practice · 3 stretch · **Time:** about 10 hours

BFS finds shortest paths when every edge costs 1 (Ch 21 §0.3). Real problems have **costs**: travel times, prices, risks. This chapter derives the shortest-path algorithms from that one starting point. **Dijkstra** is BFS with a heap, **0-1 BFS** is BFS with a deque, and **Bellman–Ford** and **Floyd–Warshall** are DP. You'll learn exactly which condition each one needs, and the counterexample showing what goes wrong without it. Choosing the right algorithm from the problem's constraints is half the score in these questions.

---

## 0. From zero: weighted paths and relaxation

The **cost** of a path is the sum of its edge weights. The **single-source shortest path** problem asks for the cheapest cost from one source s to every vertex.

Every algorithm in this chapter is built from one operation, **relaxation**:

> If going through u is cheaper than the best known way to reach v, i.e. `dist[u] + w(u, v) < dist[v]`, then set `dist[v] = dist[u] + w(u, v)`.

`dist[v]` starts at ∞ (and `dist[s] = 0`), and it only ever decreases. It is always the cost of *some* real path, so it never drops below the true shortest distance. **The algorithms differ only in the order in which they relax edges**, and in what that order guarantees.

**Why BFS isn't enough:** with weights, a path with more edges can be cheaper. With A→B costing 10, and A→C, C→B costing 1 each, BFS reaches B first through the direct, expensive edge.

---

## 1. Attempt first ★

> **[Network Delay Time](https://leetcode.com/problems/network-delay-time/) (25 minutes).** A signal is sent from node k across directed edges `times[i] = [u, v, w]` (w = travel time ≥ 0). Return the time it takes for all n nodes to receive it, or −1 if some never do.

---

## 2. Dijkstra: BFS ordered by distance

**Derivation.** BFS works because it processes vertices in order of distance. With weights, keep that idea, but order by **cost**: repeatedly take the unfinished vertex with the **smallest tentative distance**, declare it final, and relax its outgoing edges. "Repeatedly take the smallest" is exactly what a **heap** is for (Ch 15). The heap is the *frontier* of Ch 15 §3.

> **Claim (requires every weight ≥ 0):** when u has the smallest tentative distance among unfinished vertices, `dist[u]` is its true shortest distance.
>
> *Proof.* Take any path from s to u. It starts inside the finished set, so it must leave the finished set somewhere, along an edge into some unfinished vertex x. The part of the path up to x costs at least `dist[x]` (x was relaxed from a finished vertex), and `dist[x] ≥ dist[u]` because u was the smallest. **The rest of the path, from x to u, costs ≥ 0, because weights are non-negative.** So the whole path costs at least `dist[u]`. ∎

**Lazy deletion** (Ch 15 §4). Java's `PriorityQueue` can't lower a key in place. So push a new entry each time `dist[v]` improves, and when an entry pops whose distance is bigger than the current `dist[v]`, it's **stale**: skip it.

**A trace** from A (heap entries are `(distance, vertex)`):

```text
      A ──4──▶ B ──1──▶ D
      │        ▲        ▲
      1        2        5
      ▼        │        │
      C ───────┴────────┘          edges: A→B 4, A→C 1, C→B 2, B→D 1, C→D 5

pop      action                                   dist (A B C D)     heap afterwards
(0,A)    relax B = 4, C = 1                       0  4  1  ∞         (1,C) (4,B)
(1,C)    B: 1+2 = 3 < 4 → 3;  D: 1+5 = 6          0  3  1  6         (3,B) (4,B) (6,D)
(3,B)    D: 3+1 = 4 < 6 → 4                       0  3  1  4         (4,B) (4,D) (6,D)
(4,B)    stale: 4 > dist[B] = 3, skip
(4,D)    final; no outgoing edges
(6,D)    stale, skip                              final: A 0, B 3, C 1, D 4
```

B was first reached at cost 4 but finalised at 3, through C. That's exactly the case BFS gets wrong, and the stale `(4,B)` entry is the lazy-deletion leftover.

### Worked SP1 · [Network Delay Time](https://leetcode.com/problems/network-delay-time/) ★

```java
public int networkDelayTime(int[][] times, int n, int k) {
    List<List<int[]>> adj = new ArrayList<>();                 // adj.get(u) holds {v, w}
    for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());
    for (int[] t : times) adj.get(t[0]).add(new int[] {t[1], t[2]});   // nodes are 1..n
    int[] dist = new int[n + 1];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));   // {distance, node}
    pq.offer(new int[] {0, k});
    while (!pq.isEmpty()) {
        int[] top = pq.poll();
        int d = top[0], u = top[1];
        if (d > dist[u]) continue;                 // stale: u was already finalised with a smaller distance
        for (int[] e : adj.get(u)) {
            int v = e[0], nd = d + e[1];
            if (nd < dist[v]) {
                dist[v] = nd;
                pq.offer(new int[] {nd, v});        // the old entry for v stays, and is skipped later
            }
        }
    }
    int worst = 0;
    for (int v = 1; v <= n; v++) worst = Math.max(worst, dist[v]);
    return worst == Integer.MAX_VALUE ? -1 : worst;
}
```

**O((V + E) log V)**: each edge pushes at most one entry. Two habits come from here. Use **`long` distances** whenever a path can sum many large weights (CSES uses weights up to 10⁹). And **never mark a vertex visited when it's pushed**, as BFS does. In Dijkstra, a vertex is final only when it's **popped**, because a cheaper route may still be found after the first push.

**Where Dijkstra breaks: negative edges.** Edges A→B = 2, A→C = 3, C→B = −2. Dijkstra pops B with distance 2 and finalises it, but A→C→B costs 1. The proof's last step, "the rest of the path costs ≥ 0", is exactly what failed. **Negative edge → Bellman–Ford (§6).**

---

## 3. When the cost isn't a sum

### Worked SP2 · [Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/)

> Moving between adjacent cells costs the absolute height difference. A path's **effort** is its **largest single step**. Return the minimum effort from the top-left to the bottom-right.

The path cost is now a **maximum**, not a sum. Does Dijkstra still work? Re-read the proof: it needs only that **extending a path never makes it cheaper**. `max(effort so far, next step) ≥ effort so far` holds, so Dijkstra applies, with `max` in place of `+`.

```java
public int minimumEffortPath(int[][] heights) {
    int rows = heights.length, cols = heights[0].length;
    int[][] effort = new int[rows][cols];
    for (int[] row : effort) Arrays.fill(row, Integer.MAX_VALUE);
    effort[0][0] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));   // {effort, r, c}
    pq.offer(new int[] {0, 0, 0});
    int[] dr = {-1, 1, 0, 0}, dc = {0, 0, -1, 1};
    while (!pq.isEmpty()) {
        int[] top = pq.poll();
        int e = top[0], r = top[1], c = top[2];
        if (e > effort[r][c]) continue;
        if (r == rows - 1 && c == cols - 1) return e;          // popped means final: stop early
        for (int d = 0; d < 4; d++) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
            int ne = Math.max(e, Math.abs(heights[nr][nc] - heights[r][c]));   // a path's cost is its largest step
            if (ne < effort[nr][nc]) {
                effort[nr][nc] = ne;
                pq.offer(new int[] {ne, nr, nc});
            }
        }
    }
    return 0;
}
```

**The general rule:** Dijkstra works for any path cost that **never decreases when you extend the path**: sums of non-negative weights, maximums, products of probabilities ≤ 1 (maximised, in Path with Maximum Probability, practice). Two other approaches give the same answer: binary search on the effort with a BFS check (Ch 10), and adding edges in increasing weight order until the corners connect (Ch 24).

---

## 4. Weights 0 and 1: the deque

### Worked SP3 · [Minimum Cost to Make at Least One Valid Path in a Grid](https://leetcode.com/problems/minimum-cost-to-make-at-least-one-valid-path-in-a-grid/)

> Each cell has an arrow (right, left, down, up). Following the arrow is free; moving in any other direction costs 1 (you change the arrow). Return the minimum cost to get from the top-left to the bottom-right.

Every edge costs **0 or 1**. Dijkstra works, but a deque does better. **0-1 BFS:** a vertex reached through a 0-edge has the *same* distance as the current one, so push it to the **front**. One reached through a 1-edge has distance + 1, so push it to the **back**. The deque then only ever holds two consecutive distance values, in order, so it behaves like Dijkstra's heap with O(1) operations.

```java
public int minCost(int[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    int[] dr = {0, 0, 1, -1}, dc = {1, -1, 0, 0};          // index d = arrow value - 1: right, left, down, up
    int[][] cost = new int[rows][cols];
    for (int[] row : cost) Arrays.fill(row, Integer.MAX_VALUE);
    cost[0][0] = 0;
    ArrayDeque<int[]> deque = new ArrayDeque<>();
    deque.offerFirst(new int[] {0, 0});
    while (!deque.isEmpty()) {
        int[] cell = deque.pollFirst();
        int r = cell[0], c = cell[1];
        for (int d = 0; d < 4; d++) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
            int w = (grid[r][c] == d + 1) ? 0 : 1;           // following the arrow is free
            if (cost[r][c] + w < cost[nr][nc]) {
                cost[nr][nc] = cost[r][c] + w;
                if (w == 0) deque.offerFirst(new int[] {nr, nc});   // the same distance: front
                else deque.offerLast(new int[] {nr, nc});           // distance + 1: back
            }
        }
    }
    return cost[rows - 1][cols - 1];
}
```

That's **O(V + E)**. A cell's cost can improve at most once after it's first set (from d + 1 to d), so each cell enters the deque at most twice. Minimum Obstacle Removal (practice) is the same algorithm.

---

## 5. Dijkstra on augmented state

Ch 21 §8 added information to BFS states. Do the same here: **the vertex becomes (node, extra state)**, and Dijkstra runs on that bigger graph.

### Worked SP4 · [CSES Flight Discount](https://cses.fi/problemset/task/1195)

> n cities (≤ 10⁵), m directed flights (≤ 2·10⁵) with prices up to 10⁹. You have **one coupon** that halves one flight's price (rounded down). Find the cheapest route from city 1 to city n. (Read from standard input; print one number.)

**What must a state record?** Whether the coupon has been used. So the state is `(city, used ∈ {0, 1})`, with three kinds of edge for each flight u → v of price c:
* (u, 0) → (v, 0) at cost c, keeping the coupon;
* (u, 0) → (v, 1) at cost ⌊c/2⌋, spending it;
* (u, 1) → (v, 1) at cost c, the coupon already gone.

```java
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        FastReader in = new FastReader(System.in);                   // Ch 4 §2.2
        int n = in.nextInt(), m = in.nextInt();
        List<List<int[]>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        for (int i = 0; i < m; i++) {
            int a = in.nextInt() - 1, b = in.nextInt() - 1, c = in.nextInt();
            adj.get(a).add(new int[] {b, c});
        }
        long[][] dist = new long[2][n];                              // dist[used][city]; sums reach 10^14: long
        for (long[] row : dist) Arrays.fill(row, Long.MAX_VALUE);
        dist[0][0] = 0;
        PriorityQueue<long[]> pq = new PriorityQueue<>((x, y) -> Long.compare(x[0], y[0]));   // {cost, city, used}
        pq.offer(new long[] {0, 0, 0});
        while (!pq.isEmpty()) {
            long[] top = pq.poll();
            long d = top[0];
            int u = (int) top[1], used = (int) top[2];
            if (d > dist[used][u]) continue;
            for (int[] e : adj.get(u)) {
                relax(dist, pq, used, e[0], d + e[1]);               // pay full price
                if (used == 0) relax(dist, pq, 1, e[0], d + e[1] / 2);   // spend the coupon here
            }
        }
        System.out.println(dist[1][n - 1]);   // spending the coupon somewhere never costs more
    }

    private static void relax(long[][] dist, PriorityQueue<long[]> pq, int used, int v, long cost) {
        if (cost < dist[used][v]) {
            dist[used][v] = cost;
            pq.offer(new long[] {cost, v, used});
        }
    }

    // static final class FastReader { ... }   (Ch 4 §2.2)
}
```

The state space is 2n, so the cost is still O((V + E) log V). **The same move handles** "at most k free edges" (states × (k + 1)), "arrive by time T" (state = (node, time), Minimum Cost to Reach Destination in Time, practice), and "the parity of the number of steps".

---

## 6. Bellman–Ford: negative edges, and limits on the number of edges

**Derivation, as a DP over the number of edges.** Let `dᵢ[v]` be the cheapest cost to reach v **using at most i edges**. A cheapest path with at most i + 1 edges is either one with at most i edges, or one with at most i edges to some u, followed by the edge u → v. So **one pass relaxing every edge** takes you from round i to round i + 1.

> **After round i, `dist[v]` ≤ the cheapest path to v with at most i edges.** A shortest path never needs more than V − 1 edges, unless a **negative cycle** lets you keep lowering the cost by going round it. So:
> * after **V − 1 rounds**, every distance is final;
> * if a **V-th round** still improves something, a negative cycle is reachable.

That's O(V · E), much slower than Dijkstra, but it's correct with negative edges, and it can **count edges**.

```java
// Plain Bellman–Ford with negative-cycle detection (CSES High Score, practice, builds on this).
long[] dist = new long[n];
Arrays.fill(dist, INF);
dist[source] = 0;
for (int round = 1; round < n; round++)
    for (int[] e : edges)                                       // e = {u, v, w}
        if (dist[e[0]] != INF && dist[e[0]] + e[2] < dist[e[1]]) dist[e[1]] = dist[e[0]] + e[2];
// one more round: any further improvement means a negative cycle is reachable from source
```

### Worked SP5 · [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/)

> The cheapest price from `src` to `dst` with **at most k stops** (k + 1 flights), or −1.

Dijkstra minimises cost but ignores the number of edges, and the cheapest route might have too many stops. Bellman–Ford's rounds **are** edge counts, so run exactly k + 1 of them. Each round must relax **only from the previous round's values**, though. Updating in place lets one round chain several flights, which silently breaks the edge limit.

```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    final int INF = Integer.MAX_VALUE;
    int[] dist = new int[n];                       // cheapest cost using at most `round` flights
    Arrays.fill(dist, INF);
    dist[src] = 0;
    for (int round = 1; round <= k + 1; round++) {
        int[] next = dist.clone();                 // read from the last round, write into a copy
        for (int[] f : flights) {
            int u = f[0], v = f[1], w = f[2];
            if (dist[u] != INF && dist[u] + w < next[v]) next[v] = dist[u] + w;
        }
        dist = next;
    }
    return dist[dst] == INF ? -1 : dist[dst];
}
```

That's O(k · E). The `dist[u] != INF` guard prevents `INF + w` from overflowing into a negative number.

---

## 7. Counting the shortest paths

### Worked SP6 · [Number of Ways to Arrive at Destination](https://leetcode.com/problems/number-of-ways-to-arrive-at-destination/)

> An undirected graph with travel times (≥ 1). Count the **shortest** paths from 0 to n − 1, modulo 10⁹ + 7.

Run Dijkstra and keep `ways[v]` alongside `dist[v]`:
* a **strictly shorter** route to v replaces everything: `ways[v] = ways[u]`;
* an **equally short** route adds: `ways[v] += ways[u]`.

**Why the counts are right when they're used:** with weights ≥ 1, every predecessor of v on a shortest path has a strictly smaller distance, so it's popped, and its `ways` is final, before v is popped. That's the topological-order DP of Ch 22, on the "shortest-path DAG".

```java
public int countPaths(int n, int[][] roads) {
    final long MOD = 1_000_000_007L;
    List<List<int[]>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int[] r : roads) {
        adj.get(r[0]).add(new int[] {r[1], r[2]});
        adj.get(r[1]).add(new int[] {r[0], r[2]});
    }
    long[] dist = new long[n];                        // times up to 10^9 across many roads: long
    long[] ways = new long[n];
    Arrays.fill(dist, Long.MAX_VALUE);
    dist[0] = 0;
    ways[0] = 1;
    PriorityQueue<long[]> pq = new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));
    pq.offer(new long[] {0, 0});
    while (!pq.isEmpty()) {
        long[] top = pq.poll();
        long d = top[0];
        int u = (int) top[1];
        if (d > dist[u]) continue;
        for (int[] e : adj.get(u)) {
            int v = e[0];
            long nd = d + e[1];
            if (nd < dist[v]) {                       // strictly shorter: it replaces all earlier routes
                dist[v] = nd;
                ways[v] = ways[u];
                pq.offer(new long[] {nd, v});
            } else if (nd == dist[v]) {               // another shortest route
                ways[v] = (ways[v] + ways[u]) % MOD;
            }
        }
    }
    return (int) ways[n - 1];
}
```

With **zero-weight** edges this breaks: a predecessor and v could share a distance, and v might be processed first. That's why the weights-≥-1 constraint matters here.

---

## 8. All pairs: Floyd–Warshall

**Derivation, as a DP over the intermediate vertices.** Let `dₖ[i][j]` be the cheapest path from i to j whose **intermediate** vertices all come from `{0, …, k}`. Such a path either avoids vertex k, costing `dₖ₋₁[i][j]`, or passes through it exactly once, costing `dₖ₋₁[i][k] + dₖ₋₁[k][j]`. So:

```java
for (int k = 0; k < n; k++)                // k MUST be the outermost loop
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            if (dist[i][k] + dist[k][j] < dist[i][j]) dist[i][j] = dist[i][k] + dist[k][j];
```

**Why k is outermost:** the recurrence needs every pair's answer for the set `{0, …, k − 1}` before it allows k. Putting k inside computes something else, and gets wrong answers. That's O(V³), so use it for **V ≤ about 400**. It handles negative edges (but not negative cycles: if `dist[i][i] < 0` afterwards, one exists) and gives every pair at once.

### Worked SP7 · [Find the City With the Smallest Number of Neighbors at a Threshold Distance](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/)

> n ≤ 100 cities, weighted undirected roads. For each city, count the cities reachable within `distanceThreshold`. Return the city with the fewest such neighbours (ties: the largest index).

```java
public int findTheCity(int n, int[][] edges, int distanceThreshold) {
    final int INF = 1_000_000_000;                    // INF + INF still fits in int
    int[][] dist = new int[n][n];
    for (int[] row : dist) Arrays.fill(row, INF);
    for (int i = 0; i < n; i++) dist[i][i] = 0;
    for (int[] e : edges) {
        dist[e[0]][e[1]] = Math.min(dist[e[0]][e[1]], e[2]);
        dist[e[1]][e[0]] = Math.min(dist[e[1]][e[0]], e[2]);
    }
    for (int k = 0; k < n; k++)
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                if (dist[i][k] + dist[k][j] < dist[i][j]) dist[i][j] = dist[i][k] + dist[k][j];
    int bestCity = -1, fewest = Integer.MAX_VALUE;
    for (int i = 0; i < n; i++) {
        int reachable = 0;
        for (int j = 0; j < n; j++) if (i != j && dist[i][j] <= distanceThreshold) reachable++;
        if (reachable <= fewest) {                    // <= so that ties go to the larger index
            fewest = reachable;
            bestCity = i;
        }
    }
    return bestCity;
}
```

**Choosing INF.** It must be big enough to mean "unreachable", yet small enough that INF + INF doesn't overflow. 10⁹ in `int` (or `Long.MAX_VALUE / 4` in `long`) satisfies both, which saves an `if` in the innermost loop.

---

## 9. Which algorithm?

| Edge weights | Question | Algorithm | Cost |
| :--- | :--- | :--- | :--- |
| All equal (unweighted) | Single source | **BFS** (Ch 21) | O(V + E) |
| Only 0 and 1 | Single source | **0-1 BFS** | O(V + E) |
| Non-negative | Single source | **Dijkstra** | O((V + E) log V) |
| Any, on a **DAG** | Single source | **Topological DP** (Ch 22) | O(V + E) |
| Negative allowed, or **at most k edges** | Single source | **Bellman–Ford** | O(V · E) or O(k · E) |
| Any (no negative cycles), V ≤ ~400 | **All pairs** | **Floyd–Warshall** | O(V³) |
| Non-negative, all pairs, V large and graph sparse | All pairs | Dijkstra from every vertex | O(V (V + E) log V) |

---

## 10. Where shortest-path algorithms break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Dijkstra with negative edges | A finalised vertex can later get cheaper (§2) | Bellman–Ford |
| A negative cycle on the way to the target | No shortest path exists (it's −∞) | Detect it (the V-th round), and say so |
| Bellman–Ford with a stop limit, updated in place | Several edges get used in one round | Relax from a copy of the last round |
| Dijkstra marking vertices visited when pushed | Locks in a too-early distance | Finalise on pop; skip stale entries |
| `int` distances on large weights, `INF + w` overflow | Negative "distances" | `long`; guard with `dist[u] != INF` |
| Floyd with k as an inner loop | Wrong answers | k outermost |

---

## 11. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| The **second** shortest path / time | Keep the two best distinct values per vertex (stretch) |
| Edges you're required to use / a meeting point | Run Dijkstra from several sources, including on the reversed graph, then combine (stretch, Minimum Weighted Subgraph) |
| Which edges lie on *some* shortest path | Dijkstra from both ends: an edge (u, v, w) qualifies iff `d₁[u] + w + d₂[v] = d₁[target]` (practice) |
| The cheapest path minimising its largest edge | A minimax path: Dijkstra with `max`, or MST (Ch 24) |
| Waiting is allowed at vertices with time windows | Dijkstra over time, with waiting rules (stretch) |

---

## 12. Recognition

**Signals:**
* "Cheapest / fastest / minimum cost / minimum time" with **varying costs** → weighted shortest path; choose with §9.
* Costs that are only **0 or 1** ("free if …, otherwise 1") → **0-1 BFS**.
* "**At most k** stops / edges / changes" → Bellman–Ford rounds, or a state including the count.
* "Coupons", "fuel", "time windows", "the parity of steps" → **augmented state**.
* "Minimise the **maximum** edge on a path" → Dijkstra with `max` (or MST, Ch 24).
* "**Number of** shortest paths" → Dijkstra + counting.
* "**Every pair** of cities", with n ≤ 400 → Floyd–Warshall.

**False friends:** "the shortest path visiting **every** vertex" isn't a shortest-path problem. It's a travelling-salesman problem, and bitmask DP handles it for n ≤ 20 (Ch 32).

---

## 13. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Comparing `long` distances with a subtracting comparator | `Long.compare` |
| A missing stale-entry check | `if (d > dist[u]) continue;` |
| Using `Integer.MAX_VALUE` as INF and then adding to it | Guard the addition, or use INF = 10⁹ / `Long.MAX_VALUE / 4` |
| Undirected edges added once | Add both directions |
| 1-indexed nodes | Size arrays n + 1, or subtract 1 at input |
| Returning the moment the target is *pushed* | Return when it's *popped* (only then is it final) |

---

## 14. Practice

1. **[CSES Shortest Routes I](https://cses.fi/problemset/task/1671)** *(Medium; stdin/stdout)*
2. **[Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/)** *(Medium)*
3. **[Minimum Obstacle Removal to Reach Corner](https://leetcode.com/problems/minimum-obstacle-removal-to-reach-corner/)** *(Hard)*
4. **[Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/)** *(Hard)*
5. **[CSES Shortest Routes II](https://cses.fi/problemset/task/1672)** *(Medium; stdin/stdout)*
6. **[CSES High Score](https://cses.fi/problemset/task/1673)** *(Hard; stdin/stdout)*
7. **[Find Edges in Shortest Paths](https://leetcode.com/problems/find-edges-in-shortest-paths/)** *(Hard)*
8. **[Minimum Cost to Reach Destination in Time](https://leetcode.com/problems/minimum-cost-to-reach-destination-in-time/)** *(Hard)*

**Stretch:** **[Minimum Weighted Subgraph With the Required Paths](https://leetcode.com/problems/minimum-weighted-subgraph-with-the-required-paths/)** *(Hard)* · **[Second Minimum Time to Reach Destination](https://leetcode.com/problems/second-minimum-time-to-reach-destination/)** *(Hard)* · **[Minimum Time to Visit a Cell In a Grid](https://leetcode.com/problems/minimum-time-to-visit-a-cell-in-a-grid/)** *(Hard)*

<details><summary>Nudges</summary>

1. Shortest Routes I: SP1 with `long` distances and fast I/O; print all n distances.
2. Max Probability: maximise a product of probabilities ≤ 1. Extending a path never increases it, so use Dijkstra with a max-heap.
3. Obstacle Removal: entering an obstacle costs 1 and an empty cell costs 0, so 0-1 BFS.
4. Swim: a path's cost is the highest cell on it (SP2). Or binary search on the time with a BFS check.
5. Shortest Routes II: Floyd–Warshall with `long`, n ≤ 500; queries print −1 when unreachable.
6. High Score: maximise the score = negate the weights and minimise. A negative cycle matters only if it's reachable from 1 **and** can reach n (check both directions).
7. Edges in Shortest Paths: Dijkstra from 0 and from n − 1; check `d₀[u] + w + dₙ[v] == d₀[n − 1]` in either direction.
8. Destination in Time: the state is (city, time spent), with cost = fees. Dijkstra by fee, pruning states over maxTime. Or a DP over time.
9. Stretch, Weighted Subgraph: Dijkstra from src1, from src2, and from dest on the *reversed* graph; minimise over meeting points m of `d₁[m] + d₂[m] + d₃[m]`.
10. Stretch, Second Minimum: BFS keeping the first two distinct arrival times per vertex; compute the travel time with the traffic-signal waits.
11. Stretch, Visit a Cell: Dijkstra; if you arrive too early, bounce back and forth, so the wait depends on parity. It's impossible only if both first moves are blocked.
</details>

<details><summary>Key</summary>

Routes I → SP1 with `long` and fast I/O · Max Probability → SP1 maximising a product · Obstacle Removal → SP3 · Swim → SP2 (or binary search + BFS, or DSU) · Routes II → SP7 · High Score → Bellman–Ford + two-way reachability for negative cycles · Edges in Shortest Paths → Dijkstra from both ends · Destination in Time → SP4 (city, time) · Weighted Subgraph → three Dijkstras, one reversed · Second Minimum → BFS keeping two distances · Visit a Cell → SP1 with waiting parity
</details>

---

## 15. Self-test

<details><summary>1. Where exactly does Dijkstra's proof use non-negative weights?</summary>

In the last step: the remainder of any alternative path, from the first unfinished vertex x to u, must cost ≥ 0, so the whole path costs at least `dist[x] ≥ dist[u]`.
</details>

<details><summary>2. Give a three-vertex graph where Dijkstra fails.</summary>

A→B = 2, A→C = 3, C→B = −2. Dijkstra finalises B at 2; the true distance is 1.
</details>

<details><summary>3. Why must Cheapest Flights relax from a copy of the previous round?</summary>

Updating in place lets a single round use an edge relaxed earlier in the same round, which chains several flights and breaks the "at most i edges after round i" guarantee.
</details>

<details><summary>4. Why must k be the outermost loop in Floyd–Warshall?</summary>

The recurrence for allowing intermediate vertex k needs every pair's answer using only intermediates {0, …, k − 1}. Only when k is outermost are all of those complete.
</details>

<details><summary>5. Why does 0-1 BFS push 0-edges to the front?</summary>

A vertex reached through a 0-edge has the same distance as the current one, so it belongs with the vertices being processed now. Pushing it to the front keeps the deque sorted by distance.
</details>

---

## 16. Exit gate

Move on when you can:

* prove Dijkstra's greedy step and give the negative-edge counterexample;
* write Dijkstra with lazy deletion, 0-1 BFS, k-round Bellman–Ford and Floyd–Warshall from blank files, each in under 10 minutes;
* choose the right algorithm for a new problem from §9 alone;
* model a coupon / fuel / time constraint as augmented state;
* solve 6 of the 8 practice problems.
