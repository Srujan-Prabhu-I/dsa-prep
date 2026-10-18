# Chapter 25 — Connectivity & Euler Paths

> **Part V · Week 22** · **S-tier**: hard OAs and harder onsite rounds · Prerequisites: Ch 4 §16 (deep recursion in OAs), Ch 15 (heaps), Ch 21 (DFS), Ch 22 (topological DP)
> **Problems:** 6 worked (1 ★) · 5 practice · 2 stretch · **Time:** about 8 hours

Ch 21 asked whether two vertices are connected. This chapter asks how **robustly** they're connected, and how to exploit structure that plain BFS ignores:
* **Bridges and articulation points:** the single edges and single vertices whose failure disconnects a network. This is the "single point of failure" question in its exact form.
* **Strongly connected components:** in a directed graph, groups in which everyone can reach everyone. Collapsing them turns *any* directed graph into a DAG, so Ch 22's topological DP applies.
* **Euler paths:** walks that use **every edge exactly once**, with a clean existence test and a short algorithm.
* **Functional graphs:** graphs where every vertex has exactly one outgoing edge. Their structure is simple enough to handle in O(n).

All of it rests on one tool: **information recorded during a DFS**.

---

## 0. From zero: DFS timestamps and low-links

Run a DFS on an **undirected** graph and record `tin[u]`, the order in which u is first visited. The edges the DFS follows to new vertices form the **DFS tree**. Every other edge is a **back edge**, and in an undirected DFS a back edge always joins a vertex to one of its **ancestors**. There are no "cross" edges between separate branches, because the DFS would have followed such an edge before leaving the first branch.

> **low[u]** = the smallest `tin` reachable from u's subtree using tree edges downward plus **at most one back edge**. It measures how high u's subtree can climb back up the tree without using the edge into u.

It's computed in the same DFS, bottom-up (Ch 19's "return up"):
* `low[u]` starts at `tin[u]`;
* for each back edge `u – w`, use `tin[w]`;
* for each tree child v, use `low[v]` (anything v's subtree can reach, u's subtree can reach).

**A worked example.** Two triangles joined by one edge, with the DFS starting at 0 and visiting neighbours in edge order:

```text
   0 ─── 1 ─── 3 ─── 4         edges: 0-1, 1-2, 2-0, 1-3, 3-4, 4-5, 5-3
    \   /       \   /          DFS tree: 0 → 1 → 2, and 1 → 3 → 4 → 5
     \ /         \ /           back edges: 2-0 and 5-3
      2           5

vertex   tin   low   why low
0        0     0     itself
1        1     0     its child 2 has low 0 (the back edge 2-0 climbs to 0)
2        2     0     back edge 2-0 reaches tin 0
3        3     3     its subtree {3, 4, 5} climbs only back to 3 (via 5-3)
4        4     3     its child 5 has low 3
5        5     3     back edge 5-3 reaches tin 3
```

* **Bridge 1-3:** `low[3] = 3 > tin[1] = 1`, so nothing below 3 climbs past 3. Every other tree edge has `low[child] ≤ tin[parent]`.
* **Articulation points 1 and 3:** the child 3 of 1 has `low[3] = 3 ≥ tin[1]`, and the child 4 of 3 has `low[4] = 3 ≥ tin[3]`. The root 0 has only one DFS child, so it isn't one.

Check this table against your own hand computation before reading the code.

---

## 1. Attempt first ★

> **[Critical Connections in a Network](https://leetcode.com/problems/critical-connections-in-a-network/) (30 minutes).** n servers and undirected connections. A connection is **critical** if removing it makes some server unable to reach another. Return all critical connections. (n ≤ 10⁵.) Removing each edge and running a BFS is O(E · (V + E)). Find the structural test.

---

## 2. Bridges

### Worked C1 · [Critical Connections in a Network](https://leetcode.com/problems/critical-connections-in-a-network/) ★

> **Bridge test.** A tree edge `p – u` (with p the parent) is a **bridge** exactly when **`low[u] > tin[p]`**.
>
> *Why:* `low[u] > tin[p]` says that nothing in u's subtree has a back edge to p or anything above it. So the only way out of u's subtree is the edge `p – u`, and removing it cuts the subtree off. Conversely, if some back edge from the subtree reaches p or higher, that edge and the tree path together form a cycle through `p – u`, so removing `p – u` leaves an alternative route. A back edge can never be a bridge, since it always lies on a cycle.

```java
private int timer;
private int[] tin, low;
private List<List<int[]>> adj;                    // adj.get(u) holds {v, edgeId}
private List<List<Integer>> bridges;

public List<List<Integer>> criticalConnections(int n, List<List<Integer>> connections) {
    adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int id = 0; id < connections.size(); id++) {
        int a = connections.get(id).get(0), b = connections.get(id).get(1);
        adj.get(a).add(new int[] {b, id});
        adj.get(b).add(new int[] {a, id});
    }
    tin = new int[n];
    low = new int[n];
    Arrays.fill(tin, -1);                         // -1 = not visited yet
    timer = 0;
    bridges = new ArrayList<>();
    for (int u = 0; u < n; u++) if (tin[u] == -1) dfs(u, -1);
    return bridges;
}

private void dfs(int u, int parentEdge) {
    tin[u] = low[u] = timer++;
    for (int[] e : adj.get(u)) {
        int v = e[0], id = e[1];
        if (id == parentEdge) continue;           // skip only the edge we arrived on (keeps parallel edges correct)
        if (tin[v] != -1) {
            low[u] = Math.min(low[u], tin[v]);    // a back edge to an ancestor
        } else {
            dfs(v, id);
            low[u] = Math.min(low[u], low[v]);
            if (low[v] > tin[u]) bridges.add(List.of(u, v));   // v's subtree can't climb back to u or above
        }
    }
}
```

That's **O(V + E)**. Skip the parent **edge** by id, not the parent **vertex**: if there are two parallel edges between u and its parent, the second one is a genuine back edge, and neither edge is a bridge.

**Recursion depth** can reach V, which is 10⁵ here. LeetCode's environment usually copes. In an OA, run the solver in a thread with a big stack (Ch 4 §16), or rewrite the DFS with an explicit stack of (vertex, next-edge index).

---

## 3. Articulation points

### Worked C2 *(custom)* · Critical Routers

> Given a connected undirected network, return every router whose failure (removing the vertex and its edges) disconnects the rest. Verify your solution against the brute force "remove each vertex, then count components with BFS" on small random graphs.

> **Articulation-point test.**
> * A **non-root** vertex u is an articulation point exactly when some DFS child v has **`low[v] ≥ tin[u]`**: v's subtree can climb at most back to u itself, so removing u strands it.
> * The **root** is an articulation point exactly when it has **two or more DFS children**. The children's subtrees can't connect except through the root, because there are no cross edges.

Compare this with the bridge test: `≥` where bridges use `>`. Reaching exactly u is enough to survive the loss of the *edge* p – u, but not the loss of the *vertex* u.

```java
private void dfsAP(int u, int parentEdge, boolean[] isCut) {
    tin[u] = low[u] = timer++;
    int children = 0;
    for (int[] e : adj.get(u)) {
        int v = e[0], id = e[1];
        if (id == parentEdge) continue;
        if (tin[v] != -1) {
            low[u] = Math.min(low[u], tin[v]);
        } else {
            children++;
            dfsAP(v, id, isCut);
            low[u] = Math.min(low[u], low[v]);
            if (parentEdge != -1 && low[v] >= tin[u]) isCut[u] = true;   // non-root rule
        }
    }
    if (parentEdge == -1 && children >= 2) isCut[u] = true;               // root rule
}
```

Minimum Number of Days to Disconnect Island (practice) needs this idea in its exact form: the answer is always 0, 1 or 2, and it's 1 exactly when the island has an articulation cell.

---

## 4. Strongly connected components

In a **directed** graph, a **strongly connected component (SCC)** is a maximal set of vertices in which **every vertex can reach every other**. Contract each SCC to a single vertex and you get the **condensation**, which is always a **DAG**: a cycle between components would merge them into one.

### 4.1 Kosaraju's algorithm

1. Run a DFS on G and record the vertices in **finishing order**.
2. Reverse every edge. Run DFS on the reversed graph, **starting from vertices in decreasing finishing time**. Each new DFS tree is exactly one SCC.

*Why it works:* the vertex that finishes **last** in step 1 lies in a **source** component of the condensation, one that no other component can reach. In the **reversed** graph that component becomes a sink: a DFS from it can't escape, so it collects exactly that SCC. Removing it and repeating peels off the components one at a time. The order in which step 2 finds them is a **topological order of the condensation** (sources first), which is ready-made for DP.

### Worked C3a · [CSES Planets and Kingdoms](https://cses.fi/problemset/task/1683)

> n planets (≤ 10⁵) and m one-way teleporters (≤ 2·10⁵). Two planets are in the same kingdom when each can reach the other. Print the number of kingdoms, then a kingdom label for each planet. (stdin/stdout)

```java
import java.io.*;
import java.util.*;

public class Main {
    static List<List<Integer>> g, rg;
    static boolean[] seen;
    static List<Integer> finishOrder = new ArrayList<>();
    static int[] comp;

    public static void main(String[] args) throws Exception {
        Thread t = new Thread(null, () -> {                          // deep recursion: big stack (Ch 4 §16)
            try { run(); } catch (IOException e) { throw new UncheckedIOException(e); }
        }, "solver", 1 << 27);
        t.start();
        t.join();
    }

    static void run() throws IOException {
        FastReader in = new FastReader(System.in);                   // Ch 4 §2.2
        int n = in.nextInt(), m = in.nextInt();
        g = new ArrayList<>();
        rg = new ArrayList<>();
        for (int i = 0; i < n; i++) { g.add(new ArrayList<>()); rg.add(new ArrayList<>()); }
        for (int i = 0; i < m; i++) {
            int a = in.nextInt() - 1, b = in.nextInt() - 1;
            g.get(a).add(b);
            rg.get(b).add(a);                                        // the reversed graph
        }
        seen = new boolean[n];
        for (int v = 0; v < n; v++) if (!seen[v]) order(v);          // pass 1: finishing order on G
        comp = new int[n];
        Arrays.fill(comp, -1);
        int components = 0;
        for (int i = n - 1; i >= 0; i--) {                           // pass 2: decreasing finish time, on reversed G
            int v = finishOrder.get(i);
            if (comp[v] == -1) assign(v, components++);
        }
        StringBuilder out = new StringBuilder().append(components).append('\n');
        for (int v = 0; v < n; v++) out.append(comp[v] + 1).append(v + 1 < n ? ' ' : '\n');
        System.out.print(out);
    }

    static void order(int u) {
        seen[u] = true;
        for (int v : g.get(u)) if (!seen[v]) order(v);
        finishOrder.add(u);                                          // u finishes after everything it reaches
    }

    static void assign(int u, int c) {
        comp[u] = c;
        for (int v : rg.get(u)) if (comp[v] == -1) assign(v, c);
    }

    // static final class FastReader { ... }   (Ch 4 §2.2)
}
```

O(V + E). Tarjan's algorithm finds the same components in one pass using low-links. It's equally valid; know one of the two well.

### Worked C3b · [CSES Coin Collector](https://cses.fi/problemset/task/1686)

> Each room has some coins, and the tunnels are one-way. Starting anywhere and moving along tunnels, collect as many coins as possible. Coins can be collected once per room, and you may revisit rooms.

**Inside an SCC you can visit every room and come back**, so entering a component means collecting **all of its coins**. That makes this a **longest-path problem on the condensation DAG**, with weights on the vertices: Ch 22's topological DP. Kosaraju already lists the components in topological order (sources first), so process them **in reverse**, so that every component's successors are finished first:

```java
// after Kosaraju: comp[v] in [0, components), numbered in topological order (sources first)
long[] coinsIn = new long[components];
for (int v = 0; v < n; v++) coinsIn[comp[v]] += coins[v];
long[] best = new long[components];                 // the most coins on a path starting in each component
long answer = 0;
for (int c = components - 1; c >= 0; c--) best[c] = coinsIn[c];   // a path that stays in c
for (int c = components - 1; c >= 0; c--) {         // reverse topological order: successors are final
    for (int v : membersOf.get(c))
        for (int w : g.get(v))
            if (comp[w] != c) best[c] = Math.max(best[c], coinsIn[c] + best[comp[w]]);
    answer = Math.max(answer, best[c]);
}
```

(`membersOf` groups the vertices by component.) **Use this whenever a DP must run on a directed graph that has cycles:** collapse the SCCs, then run the DP on the DAG.

---

## 5. Euler paths: every edge exactly once

An **Euler path** uses every edge exactly once; an **Euler circuit** is one that also ends where it started.

> **Existence (directed graph).** Every vertex has in-degree = out-degree, except possibly one **start** with out = in + 1 and one **end** with in = out + 1. Also, every edge must be reachable from the start. For a circuit, every vertex is balanced. (Undirected: zero or two vertices of odd degree.)

**Hierholzer's algorithm.** From the start, walk along unused edges, deleting each one as you use it. When you get stuck at a vertex (it has no unused edges left), that vertex is **finished**: put it in the output. Keep doing this, backing up the walk, and the output **reversed** is an Euler path.

*Why getting stuck is safe:* by the degree conditions, you can only get stuck at the end of the remaining trail. Any loop of edges you skipped past is spliced in when the walk backs up to it.

### Worked C4 · [Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/)

> Tickets `[from, to]` form an itinerary starting at `"JFK"` that uses every ticket exactly once. If there are several, return the lexicographically smallest.

The airports are vertices and the tickets are edges. A valid itinerary is an Euler path from JFK. **For the lexicographically smallest one**, always take the smallest available destination first: keep each airport's outgoing tickets in a **min-heap** (Ch 15).

```java
public List<String> findItinerary(List<List<String>> tickets) {
    Map<String, PriorityQueue<String>> out = new HashMap<>();      // the smallest destination comes out first
    for (List<String> t : tickets) out.computeIfAbsent(t.get(0), k -> new PriorityQueue<>()).add(t.get(1));
    List<String> route = new ArrayList<>();
    visit("JFK", out, route);
    Collections.reverse(route);                                     // airports were added as they finished
    return route;
}

// Hierholzer: use up every remaining ticket out of `airport`, then record it as finished.
private void visit(String airport, Map<String, PriorityQueue<String>> out, List<String> route) {
    PriorityQueue<String> next = out.get(airport);
    while (next != null && !next.isEmpty()) visit(next.poll(), out, route);   // poll = delete the edge
    route.add(airport);
}
```

That's O(E log E). **The tempting wrong approach** is a greedy walk that just follows the smallest destination and stops when stuck. It can reach a dead end early and leave tickets unused. Hierholzer's *finish-then-reverse* is what splices those detours back in correctly.

**Euler versus Hamilton.** An Euler path visits every **edge** once and takes linear time. A **Hamiltonian** path visits every **vertex** once, which is NP-hard in general, so bitmask DP is needed for n ≤ 20 (Ch 32). Telling the two apart is a classic recognition test.

---

## 6. Functional graphs: one outgoing edge per vertex

When **every vertex has exactly one outgoing edge** (`next[i]`), the graph's shape is forced. Each weakly connected piece is **one cycle with trees hanging into it**, a "ρ" shape. Following `next` from any vertex eventually enters a cycle. You saw this with Floyd's algorithm in Ch 13 §3.

### Worked C5 · [Longest Cycle in a Graph](https://leetcode.com/problems/longest-cycle-in-a-graph/)

> `edges[i]` is the single destination of vertex i, or −1 if there is none. Return the length of the longest cycle, or −1.

Walk from each unvisited vertex, stamping every vertex with **which walk reached it** and **at which step**. A walk stops when it hits −1 or a vertex that's already stamped:
* if that vertex was stamped **by this walk**, you've gone round a cycle, and its length is (current step − that vertex's step);
* if it was stamped by an **earlier** walk, this walk has merged into explored territory, and any cycle there was already counted.

```java
public int longestCycle(int[] edges) {
    int n = edges.length;
    int[] walk = new int[n];          // which walk first reached each vertex (0 = none yet)
    int[] step = new int[n];          // the step at which that walk reached it
    int best = -1;
    for (int start = 0; start < n; start++) {
        if (walk[start] != 0) continue;
        int id = start + 1, u = start, s = 0;
        while (u != -1 && walk[u] == 0) {           // the path is unique: just follow it
            walk[u] = id;
            step[u] = s++;
            u = edges[u];
        }
        if (u != -1 && walk[u] == id) best = Math.max(best, s - step[u]);   // came back to this walk: a cycle
    }
    return best;
}
```

Each vertex is stamped once, so this is **O(n)**. Count Visited Nodes (practice) adds the tails: a vertex's answer is its distance to the cycle plus the cycle's length. Planets Queries (Ch 36) jumps k steps at a time using binary lifting.

---

## 7. Where these break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Skipping the parent *vertex* instead of the parent *edge* | Parallel edges are misreported as bridges | Skip by edge id |
| Recursive low-link or Kosaraju on 10⁵+ vertices in Java | Stack overflow | A big thread stack (OA), or an iterative DFS |
| Bridges in **directed** graphs | The concept differs (you'd want strong connectivity) | SCCs / dominators (beyond scope) |
| A greedy walk for an Euler path | It gets stuck with edges left over | Hierholzer |
| "Visit every **vertex** once" | NP-hard (Hamiltonian) | Bitmask DP for n ≤ 20 (Ch 32) |

---

## 8. Recognition

**Signals:**
* "Critical connection", "single point of failure", "removing one edge / node disconnects" → **bridges / articulation points**.
* "Mutually reachable", "groups where everyone reaches everyone", or a DP on a directed graph **with cycles** → **SCCs + condensation**.
* "Use every ticket / edge / domino exactly once", "a sequence containing every combination" (de Bruijn) → **Euler path**.
* "Each node has exactly one next", "follow pointers", "the first repeated state" → **functional graph**.

---

## 9. Practice

1. **[CSES Flight Routes Check](https://cses.fi/problemset/task/1682)** *(Medium; stdin/stdout)*
2. **[CSES Mail Delivery](https://cses.fi/problemset/task/1691)** *(Hard; stdin/stdout)*
3. **[Count Visited Nodes in a Directed Graph](https://leetcode.com/problems/count-visited-nodes-in-a-directed-graph/)** *(Hard)*
4. **[Valid Arrangement of Pairs](https://leetcode.com/problems/valid-arrangement-of-pairs/)** *(Hard)*
5. **[Minimum Number of Days to Disconnect Island](https://leetcode.com/problems/minimum-number-of-days-to-disconnect-island/)** *(Hard)*

**Stretch:** **[Cracking the Safe](https://leetcode.com/problems/cracking-the-safe/)** *(Hard)* · **[Maximum Employees to Be Invited to a Meeting](https://leetcode.com/problems/maximum-employees-to-be-invited-to-a-meeting/)** *(Hard)*

<details><summary>Nudges</summary>

1. Flight Routes Check: everyone can reach everyone ⟺ vertex 1 reaches everything in G and in reversed G. Print a counterexample pair otherwise.
2. Mail Delivery: an Euler circuit in an undirected graph. Check that every degree is even and that all edges are reachable; use Hierholzer with "used edge" flags and an iterative stack.
3. Count Visited: find each cycle's vertices and length (§6); for tail vertices, answer = 1 + answer of next, computed in reverse visiting order.
4. Valid Arrangement: the pairs are edges; start at the vertex with out = in + 1 (or anywhere if all are balanced); Hierholzer.
5. Disconnect Island: if it's already 0 or ≥ 2 islands, the answer is 0; if one cell's removal disconnects it (an articulation cell, or brute force over the cells), 1; otherwise 2.
6. Stretch, Cracking the Safe: the vertices are (n − 1)-digit strings, and each edge appends a digit. An Euler circuit visits every n-digit combination exactly once.
7. Stretch, Max Employees: in a functional graph, the answer is the larger of (the longest cycle of length ≥ 3) and (the sum over all 2-cycles of 2 + the longest chains hanging into each end).
</details>

<details><summary>Key</summary>

Flight Routes Check → reachability on G and reversed G · Mail Delivery → C4 circuit + degree check · Count Visited → C5 cycle lengths + tail DP · Valid Arrangement → C4 on a directed graph · Disconnect Island → answer ≤ 2; C2 or brute force · Cracking the Safe → C4 on a de Bruijn graph · Max Employees → C5 (2-cycles + chains versus the longest cycle)
</details>

---

## 10. Self-test

<details><summary>1. State the bridge and articulation-point tests, and explain the difference between > and ≥.</summary>

Bridge: tree edge p–u with `low[u] > tin[p]`. Articulation point (non-root): a child v with `low[v] ≥ tin[u]`; the root: two or more children. Reaching exactly u saves the subtree when the *edge* p–u is removed, but not when the *vertex* u is removed.
</details>

<details><summary>2. Why is the condensation of a directed graph always a DAG?</summary>

A cycle between components would let every vertex in them reach every other, which would make them one component.
</details>

<details><summary>3. Why does Kosaraju's second pass start from the vertex that finished last?</summary>

That vertex lies in a source component of the condensation. In the reversed graph, that component becomes a sink, so a DFS from it collects exactly that SCC and nothing more.
</details>

<details><summary>4. Why does a greedy walk fail to find Euler paths, while Hierholzer succeeds?</summary>

A greedy walk can reach a dead end while edges remain elsewhere. Hierholzer records a vertex only when it's stuck there, and backtracks to splice the remaining loops into the path.
</details>

<details><summary>5. In a functional graph, why does each walk either find a new cycle or merge into an old one?</summary>

Each vertex has exactly one successor, so the walk is forced. It either revisits one of its own vertices (a new cycle) or reaches a vertex an earlier walk already stamped, which leads only into explored structure.
</details>

---

## 11. Exit gate

Move on when you can:

* compute `tin` and `low` by hand on a small graph, and state both cut tests with their proofs;
* write Critical Connections and the articulation-point DFS from blank files, and pass the brute-force comparison;
* write Kosaraju, and use the condensation for a DP (Coin Collector);
* write Hierholzer (Reconstruct Itinerary), and state the Euler existence conditions;
* solve 4 of the 5 practice problems.
