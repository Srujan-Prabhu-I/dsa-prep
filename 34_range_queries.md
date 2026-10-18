# Chapter 34 — Range Queries: Fenwick, Segment Tree, Sparse Table

> **Part VII · Week 31** · **S-tier** · Prerequisites: Ch 8 (prefix sums), Ch 9 §5 (coordinate compression), Ch 10, Ch 17 (`x & -x`), Ch 19 (recursion on a binary tree), Ch 33 §6
> **Problems:** 6 worked (1 ★) · 8 practice · 3 stretch · **Time:** about 10 hours

Ch 8 answered "sum of `a[l..r]`" in O(1) after an O(n) precomputation. That works only while the array **never changes**. This chapter covers what to do when updates and queries are interleaved, which is where interviews put "design a structure" and OAs put their last problem.

---

## 0. From zero: why both obvious answers fail

You have an array of n numbers and q operations, each one of:
* **update(i, v):** set `a[i] = v`;
* **query(l, r):** return `a[l] + … + a[r]`.

| Store | update | query | q = n = 10⁵ mixed operations |
| :--- | :-: | :-: | :--- |
| the plain array | O(1) | O(n): add up the range | 10¹⁰ |
| prefix sums (Ch 8) | O(n): every later prefix changes | O(1) | 10¹⁰ |

The plain array stores blocks of size 1, so a query must touch many of them. The prefix array stores the blocks `[0, i]`, which are huge, so an update must touch many of them. **The fix is to store sums of blocks of many different sizes**, chosen so that:
1. every range is the union of **few** stored blocks (a fast query), and
2. every position lies in **few** stored blocks (a fast update).

Both structures in this chapter are ways of choosing those blocks. Each gives O(log n) for both operations.

---

## 1. Attempt first ★

> **[Range Sum Query – Mutable](https://leetcode.com/problems/range-sum-query-mutable/) (30 minutes, before reading §2).** Implement `update(index, val)` and `sumRange(left, right)` for n, q ≤ 3·10⁴. Design something with **O(√n)** per operation by storing the sums of blocks of size about √n. How does a query use the blocks? How does an update keep them correct? Then ask what would make *both* operations cheaper.

<details><summary>The √n design</summary>

Cut the array into blocks of size b ≈ √n and store each block's sum. **Update:** change `a[i]` and adjust its one block sum, O(1). **Query:** add up the partial blocks at the two ends element by element (≤ 2b elements), plus the whole blocks in between (≤ n / b blocks): O(b + n/b) = O(√n) when b = √n. One block size is a compromise. Blocks of *every* power-of-two size remove it, which is the segment tree.
</details>

---

## 2. The segment tree, from zero

**The blocks.** The root stores the sum of the whole array `[0, n − 1]`. Each block `[l, r]` with l < r is split at `mid = (l + r) / 2` into two children `[l, mid]` and `[mid + 1, r]`, and each child is split again, down to single elements. It's a binary tree (Ch 19) in which every node knows the sum of its range. Each level halves the ranges, so there are about **log₂ n levels**.

For `a = [5, 8, 6, 3, 2, 7, 2, 6]`, each node shows its range and its stored sum:

```text
                              [0,7] 39
                    /                          \
             [0,3] 22                          [4,7] 17
            /         \                       /         \
      [0,1] 13      [2,3] 9             [4,5] 9        [6,7] 8
       /    \        /    \              /    \         /    \
    [0]5   [1]8   [2]6   [3]3         [4]2   [5]7    [6]2   [7]6
```

**query(1, 6)** = a[1] + … + a[6] = 28 uses just four stored blocks, `[1]`, `[2,3]`, `[4,5]` and `[6]`: 8 + 9 + 9 + 2 = 28. The recursion skips everything outside [1, 6] and stops descending at every node lying fully inside it.

**update(5, 1)** (set a[5] from 7 to 1) changes only the nodes whose range contains 5, one per level: `[5]` 7 → 1, `[4,5]` 9 → 3, `[4,7]` 17 → 11, `[0,7]` 39 → 33.

**Storage without pointers.** Number the root 1 and give node i the children 2i and 2i + 1, the heap layout of Ch 15 shifted to start at 1. An array of size **4n** always suffices: the tree has at most 2·(the next power of two ≥ n) nodes, and that power of two is less than 2n.

```java
long[] sum = new long[4 * n];

void build(int node, int l, int r, int[] a) {
    if (l == r) { sum[node] = a[l]; return; }
    int mid = (l + r) / 2;
    build(2 * node, l, mid, a);
    build(2 * node + 1, mid + 1, r, a);
    sum[node] = sum[2 * node] + sum[2 * node + 1];
}

// The sum of a[ql..qr], from the node covering [l, r].
long query(int node, int l, int r, int ql, int qr) {
    if (qr < l || r < ql) return 0;                      // disjoint: contributes nothing
    if (ql <= l && r <= qr) return sum[node];            // fully inside: use the stored block
    int mid = (l + r) / 2;
    return query(2 * node, l, mid, ql, qr) + query(2 * node + 1, mid + 1, r, ql, qr);
}

// Sets a[pos] = v.
void update(int node, int l, int r, int pos, long v) {
    if (l == r) { sum[node] = v; return; }
    int mid = (l + r) / 2;
    if (pos <= mid) update(2 * node, l, mid, pos, v);
    else update(2 * node + 1, mid + 1, r, pos, v);
    sum[node] = sum[2 * node] + sum[2 * node + 1];       // repair on the way back up
}
```

Call these as `build(1, 0, n - 1, a)`, `query(1, 0, n - 1, l, r)` and `update(1, 0, n - 1, i, v)`.

**Why update is O(log n):** position i lies in exactly one node per level, the path from its leaf to the root, and only those sums change.

**Why query is O(log n).** Call a node *partial* if its range overlaps `[ql, qr]` without lying inside it. Only partial nodes recurse. **Claim: each level has at most 2 partial nodes.** A partial node's range contains `ql` or `qr` on its boundary side: if it contained neither endpoint, the query range would either cover it or miss it entirely. At one level the ranges are disjoint, so only the node containing ql and the node containing qr can be partial. So at most 2 nodes per level recurse, each making 2 calls, giving at most 4 visits per level and O(log n) in total.

**Any associative combine works.** Replace `+` with `min`, `max`, `gcd` or a custom merge of two records (§2.1 below), with the matching "identity" for disjoint ranges (0 for sum, +∞ for min). This generality is the segment tree's advantage over the next structure.

**Common misconceptions:**

| Belief | Reality |
| :--- | :--- |
| "A segment tree needs 2n slots" | The recursive layout needs up to 4n; the iterative one (§6) uses exactly 2n |
| "A query visits O(log n) *ranges*, so O(log n) leaves" | It visits up to about 4 nodes per level, and stops at fully covered nodes: no leaves needed |
| "A Fenwick tree stores prefix sums" | It stores **block** sums of varying lengths; prefixes are assembled from ≤ log n of them |
| "Fenwick trees can answer range minimum" | Only **prefix** queries, and only when updates move values one way: prefix max when values only increase, prefix min when they only decrease. Arbitrary ranges need subtraction, which min and max lack |
| "A sparse table handles updates" | It's static: one update can change O(n log n) entries |

### 2.1 Nodes that store a record: the merge pattern

Sometimes one number per node isn't enough to combine two halves. **Example: the maximum subarray sum inside `a[l..r]`, with point updates.** The best subarray of a range lies entirely in the left half, entirely in the right half, or **crosses the middle**. A crossing one is (the best suffix of the left half) + (the best prefix of the right half). So each node stores four values, and the parent computes its own four from its children's:

```java
record Info(long sum, long pre, long suf, long best) {     // pre/suf: the best non-empty prefix/suffix sum
    static Info leaf(long v) { return new Info(v, v, v, v); }

    static Info merge(Info l, Info r) {                     // l covers the LEFT part: the order matters
        if (l == null) return r;                            // null = the empty range, the identity
        if (r == null) return l;
        return new Info(l.sum + r.sum,
                        Math.max(l.pre, l.sum + r.pre),     // a prefix stays left, or takes all of left plus a right prefix
                        Math.max(r.suf, r.sum + l.suf),
                        Math.max(Math.max(l.best, r.best), l.suf + r.pre));   // left, right, or crossing the middle
    }
}

Info[] tree = new Info[4 * n];
// build and update are §2's code, with   tree[node] = Info.leaf(a[l])                    at a leaf
//                                  and   tree[node] = Info.merge(tree[2*node], tree[2*node+1])   above it.
Info query(int node, int l, int r, int ql, int qr) {
    if (qr < l || r < ql) return null;                     // disjoint: the identity
    if (ql <= l && r <= qr) return tree[node];
    int mid = (l + r) / 2;
    return Info.merge(query(2 * node, l, mid, ql, qr),     // left result first, then right
                      query(2 * node + 1, mid + 1, r, ql, qr));
}
// answer for [ql, qr]:  query(1, 0, n - 1, ql, qr).best()
```

The recipe for any record node:
1. Write down **what the parent must know** that its children's single answers don't tell it (here, the best prefix and suffix).
2. Write `merge(left, right)` so that it is **associative**, and remember that it may **not be commutative**: always pass the left part first.
3. Choose an **identity** for empty ranges (`null` here) that `merge` passes through unchanged.

Mixed Set VII uses this twice. Problem #18 stores runs of equal characters (prefix run, suffix run, best run, and the end characters). Problem #21 stores a 2 × 2 table: the best sum given whether the range's first and last elements are taken.

---

## 3. The Fenwick tree (binary indexed tree), from zero

A Fenwick tree stores **n numbers**, not 4n, and its code is four lines. It answers **prefix** queries, and a range `[l, r]` becomes `prefix(r) − prefix(l − 1)`. That subtraction is why it handles **sums** (and XOR), but not min or max, for arbitrary ranges.

**The blocks.** Use 1-based positions. Position i stores the sum of the block that **ends at i** and has length **lowbit(i)**, the value of i's lowest set bit (`i & -i`, Ch 17):

> `tree[i] = a[i − lowbit(i) + 1] + … + a[i]`

For example, 12 = 1100₂ has lowbit 4, so `tree[12]` holds a[9..12]. 13 = 1101₂ has lowbit 1, so it holds only a[13]. And 8 = 1000₂ holds a[1..8].

**Prefix query.** `prefix(i)` = the block ending at i, plus `prefix(i − lowbit(i))`. Subtracting the lowest set bit removes one 1-bit, so there are **at most log₂ n steps**. For example, prefix(13) = tree[13] + tree[12] + tree[8], covering a[13], a[9..12] and a[1..8].

The blocks for n = 8, one row per position i, with each block's length equal to lowbit(i):

```text
position:      1   2   3   4   5   6   7   8
tree[1]       [1]                                  lowbit 1
tree[2]       [1 ─ 2]                              lowbit 2
tree[3]               [3]                          lowbit 1
tree[4]       [1 ───────── 4]                      lowbit 4
tree[5]                       [5]
tree[6]                       [5 ─ 6]
tree[7]                               [7]
tree[8]       [1 ───────────────────────── 8]      lowbit 8

prefix(7) = tree[7] + tree[6] + tree[4]           7 → 6 → 4 → 0  (subtract lowbit each step)
add(3, δ) updates tree[3], tree[4], tree[8]       3 → 4 → 8 → 16 > n  (add lowbit each step)
```

Every prefix is a union of at most log₂ n blocks (going down), and every position lies in at most log₂ n blocks (going up).

**Update.** Which blocks contain position i? They are exactly **i, i + lowbit(i), then add the lowbit again, …**, until the index passes n. Each step clears at least one bit at the bottom and moves the lowest set bit up, so there are at most log₂ n steps.

<details><summary>Proof that the update chain is exactly the blocks containing i</summary>

Block j ≥ i contains i iff `j − lowbit(j) < i`. Take j > i and let b be the highest bit where they differ; j has 1 there, i has 0. If j had any set bit below b, then `j − lowbit(j)` would still have bit b, so it would exceed i, and the block wouldn't contain i. So j = (i's bits above b), then a 1 at b, then all zeros, and then `j − lowbit(j)` is i with every bit below b cleared (bit b of i is already 0), which is < i exactly when i has a set bit below b. So the containing blocks are i itself plus one block for each 0-bit of i above its lowest set bit. Adding lowbit(i) to i carries through the run of 1s starting at the lowest set bit and lands on the first such 0-bit. Repeating gives each such block in increasing order.
</details>

```java
class Fenwick {
    private final long[] tree;                   // tree[i] = sum of a(i - lowbit(i), i], 1-based

    Fenwick(int n) { tree = new long[n + 1]; }

    void add(int i, long delta) {                // a[i] += delta
        for (; i < tree.length; i += i & -i) tree[i] += delta;
    }

    long prefix(int i) {                         // a[1] + ... + a[i]; prefix(0) = 0
        long s = 0;
        for (; i > 0; i -= i & -i) s += tree[i];
        return s;
    }
}
```

### Worked Q1 · [Range Sum Query – Mutable](https://leetcode.com/problems/range-sum-query-mutable/) ★

`update` sets a value, but the Fenwick tree adds, so add the **difference** and remember the current values:

```java
class NumArray {
    private final int[] nums;
    private final Fenwick fenwick;

    public NumArray(int[] nums) {
        this.nums = nums.clone();
        fenwick = new Fenwick(nums.length);
        for (int i = 0; i < nums.length; i++) fenwick.add(i + 1, nums[i]);    // shift to 1-based
    }

    public void update(int index, int val) {
        fenwick.add(index + 1, val - nums[index]);
        nums[index] = val;
    }

    public int sumRange(int left, int right) {
        return (int) (fenwick.prefix(right + 1) - fenwick.prefix(left));
    }
}
```

O(log n) per operation and O(n log n) to build. **The O(n) build, in prose:** fill `tree[i] = a[i]`, then for each i in increasing order, add `tree[i]` into `tree[i + lowbit(i)]` (if that index is ≤ n). The §2 segment tree solves this problem equally well; the Fenwick tree is shorter, which matters with a 45-minute clock.

### Worked Q2 · [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/)

> For each i, count the j > i with `nums[j] < nums[i]`. n ≤ 10⁵.

**Turn "after" into "already seen":** scan from right to left. When you reach i, the elements after it are exactly those already scanned. **Index the Fenwick tree by value:** position v holds how many scanned elements equal v. Then "how many seen elements are smaller than nums[i]" is `prefix(rank − 1)`.

Values can be negative or huge, so first replace each value by its **rank** among the distinct values (coordinate compression, Ch 9 §5). Only the comparisons matter, and ranks preserve them.

```java
public List<Integer> countSmaller(int[] nums) {
    int n = nums.length;
    int[] values = nums.clone();
    Arrays.sort(values);
    int m = 0;                                            // dedupe in place: values[0..m) distinct
    for (int x : values) if (m == 0 || values[m - 1] != x) values[m++] = x;

    Fenwick seen = new Fenwick(m);
    Integer[] answer = new Integer[n];
    for (int i = n - 1; i >= 0; i--) {
        int rank = Arrays.binarySearch(values, 0, m, nums[i]) + 1;     // 1-based rank
        answer[i] = (int) seen.prefix(rank - 1);          // seen values strictly smaller
        seen.add(rank, 1);
    }
    return Arrays.asList(answer);
}
```

O(n log n). **Alternative, in prose:** merge-sort counting (Ch 9's inversion count), tracking original indices, so each element learns how many smaller elements crossed from its right. **The general shape** is "for each element, count the earlier (or later) elements with a smaller (or larger) value": a sweep in index order plus a Fenwick tree over values. Create Sorted Array, Good Triplets and Reverse Pairs-style problems all follow it.

---

## 4. Walking down the tree: "the first position where…"

### Worked Q3 · [CSES Hotel Queries](https://cses.fi/problemset/task/1143)

> n hotels, hotel i has `h[i]` free rooms. m groups arrive in order; group j needs `r[j]` rooms and takes the **first** hotel (lowest index) with enough free rooms, or none. Print each group's hotel (1-based), or 0. n, m ≤ 2·10⁵.

Scanning for the first hotel with enough rooms costs O(n) per group. **Store the maximum free rooms in each node.** At a node, if the left child's max is enough, the first fitting hotel is in the left half; otherwise it's in the right half. Descend, never backtracking, and decrease the leaf you arrive at. **One root-to-leaf path: O(log n).**

```java
static int[] max;                                       // max[node] = most free rooms in the node's range

static void build(int node, int l, int r, int[] h) {
    if (l == r) { max[node] = h[l]; return; }
    int mid = (l + r) / 2;
    build(2 * node, l, mid, h);
    build(2 * node + 1, mid + 1, r, h);
    max[node] = Math.max(max[2 * node], max[2 * node + 1]);
}

// Gives `need` rooms in the leftmost hotel of [l, r] that has enough; returns its index, or -1.
static int assign(int node, int l, int r, int need) {
    if (max[node] < need) return -1;                     // no hotel in this range fits
    if (l == r) { max[node] -= need; return l; }
    int mid = (l + r) / 2;
    int at = max[2 * node] >= need ? assign(2 * node, l, mid, need)
                                   : assign(2 * node + 1, mid + 1, r, need);
    max[node] = Math.max(max[2 * node], max[2 * node + 1]);
    return at;
}

public static void main(String[] args) throws IOException {
    FastReader in = new FastReader(System.in);           // Ch 4 §2.2
    int n = in.nextInt(), m = in.nextInt();
    int[] h = new int[n];
    for (int i = 0; i < n; i++) h[i] = in.nextInt();
    max = new int[4 * n];
    build(1, 0, n - 1, h);
    StringBuilder out = new StringBuilder();
    for (int j = 0; j < m; j++) out.append(assign(1, 0, n - 1, in.nextInt()) + 1).append(' ');   // -1 + 1 = 0
    System.out.println(out.toString().trim());
}
```

**The walk works for any question of the form "the first index where a monotone condition on prefix aggregates holds"**: the k-th present element (List Removals: store counts and go left if the left count ≥ k, else subtract it and go right), the first prefix whose sum reaches x, the first element ≥ x within a range. A Fenwick tree can also walk ("binary lifting on the Fenwick tree"), but the segment tree version is easier to get right.

---

## 5. Range updates: lazy propagation

A range update like "set every value in `[l, r]` to v" would touch every leaf in the range: O(n). **The lazy idea:** when an update fully covers a node, change that node's stored answer *and leave a note* ("everything below me is set to v") instead of descending. Only when a later operation needs to go *inside* that node do you **push** the note down to its two children. Each operation still visits O(log n) nodes, by the same "2 partial nodes per level" argument as §2.

### Worked Q4 · [Falling Squares](https://leetcode.com/problems/falling-squares/)

> Squares fall one by one: square `[left, side]` covers x in `[left, left + side)` and lands on the highest square already below that range (or the ground). After each square, report the tallest stack so far. n ≤ 1000, coordinates up to 10⁸.

**Each square is "range max, then range assign":** its bottom is the max height over its range, its top is that plus `side`, and the whole range becomes that top.

**Compression with half-open ranges.** Coordinates up to 10⁸ won't fit as leaves, but only the square edges matter. Sort the distinct edges `x₀ < x₁ < … < x_m`. Each leaf is an **elementary segment** `[x_k, x_{k+1})`, and a square covering `[left, left + side)` covers exactly the segments from `index(left)` to `index(left + side) − 1`. Using segments rather than points is what makes two squares that only *touch* at an edge correctly not stack.

```java
private int[] max, assign;                  // assign[node] = a pending "set my whole range to this" (0 = none)

public List<Integer> fallingSquares(int[][] positions) {
    TreeSet<Integer> edges = new TreeSet<>();
    for (int[] p : positions) { edges.add(p[0]); edges.add(p[0] + p[1]); }
    List<Integer> xs = new ArrayList<>(edges);
    int m = xs.size() - 1;                  // segments 0..m-1; segment k is [xs[k], xs[k+1])
    max = new int[4 * m];
    assign = new int[4 * m];
    List<Integer> answer = new ArrayList<>();
    int tallest = 0;
    for (int[] p : positions) {
        int l = Collections.binarySearch(xs, p[0]);
        int r = Collections.binarySearch(xs, p[0] + p[1]) - 1;
        int top = query(1, 0, m - 1, l, r) + p[1];
        set(1, 0, m - 1, l, r, top);
        tallest = Math.max(tallest, top);
        answer.add(tallest);
    }
    return answer;
}

private void push(int node) {               // hand a pending assignment to both children
    if (assign[node] == 0) return;
    for (int child = 2 * node; child <= 2 * node + 1; child++) {
        max[child] = assign[node];
        assign[child] = assign[node];
    }
    assign[node] = 0;
}

private int query(int node, int l, int r, int ql, int qr) {
    if (qr < l || r < ql) return 0;
    if (ql <= l && r <= qr) return max[node];
    push(node);                             // about to look inside: children must be up to date
    int mid = (l + r) / 2;
    return Math.max(query(2 * node, l, mid, ql, qr), query(2 * node + 1, mid + 1, r, ql, qr));
}

private void set(int node, int l, int r, int ql, int qr, int v) {
    if (qr < l || r < ql) return;
    if (ql <= l && r <= qr) { max[node] = v; assign[node] = v; return; }   // covered: note it, stop
    push(node);
    int mid = (l + r) / 2;
    set(2 * node, l, mid, ql, qr, v);
    set(2 * node + 1, mid + 1, r, ql, qr, v);
    max[node] = Math.max(max[2 * node], max[2 * node + 1]);
}
```

**0 can mean "no note"** only because every assigned height is ≥ 1; otherwise keep a separate boolean array. `push` runs only at partial nodes, which always have children, so no index goes past 4m.

**A small trace** with 4 segments (leaves 0–3), all heights 0:

```text
set([0, 1], 3):   root [0,3] is partial → push (no note) → recurse
                  [0,1] is fully covered → max = 3, note "assign 3"; STOP (leaves 0 and 1 untouched)
                  [2,3] is disjoint → nothing
                  root max = max(3, 0) = 3
query([1, 2]):    root partial → push (no note) → both children
                  [0,1] partial → PUSH its note: leaves 0 and 1 get max 3 and the note; [0,1] note cleared
                      leaf 1 inside → 3;  leaf 0 disjoint → 0
                  [2,3] partial → leaf 2 inside → 0;  leaf 3 disjoint → 0
                  answer max(3, 0) = 3
```

The update touched only 2 nodes below the root; the leaves learned about it only when a query needed them.

**Honest note on constraints:** with n ≤ 1000, an O(n²) loop over earlier squares passes too. We use this problem because it's the cleanest illustration of lazy propagation, which you need when n is 10⁵ (Handling Sum Queries After Update, stretch). **Other lazy notes:** "add v to the range" (`max[child] += v`, notes add up), and "flip every bit" (count of ones becomes length − count; two flips cancel). In every case, define (1) what the note does to a node's stored value, and (2) how two notes combine.

---

## 6. A segment tree inside a DP

### Worked Q5 · [Longest Increasing Subsequence II](https://leetcode.com/problems/longest-increasing-subsequence-ii/)

> The longest strictly increasing subsequence in which adjacent chosen elements differ by **at most k**. n, values, k ≤ 10⁵.

**The DP** (Ch 29's LIS state, keyed by value instead of index): `len[v]` = the longest valid subsequence seen so far that ends with value v. Processing `nums` left to right, an element v extends the best subsequence ending with a value in `[v − k, v − 1]`:

> `len[v] = 1 + max(len[v − k .. v − 1])`

This is Ch 33 §6's row "max over states whose **value** is in a range". The range is contiguous in *value*, so index a segment tree by value: a max query on `[v − k, v − 1]`, then a point update at v. O(n log V).

**The compact (iterative) segment tree.** For point update + range query with a commutative combine (sum, min, max), there's a shorter, recursion-free form. Put the leaves at `tree[size .. 2·size)`; node i's parent is i / 2. A query on the half-open range `[lo, hi)` climbs from both ends: if `lo` is a right child, its parent's range sticks out to the left, so take `lo` itself and step right; symmetrically for `hi`.

```java
public int lengthOfLIS(int[] nums, int k) {
    int size = 100_001;                              // a leaf for every value 0..100000
    int[] tree = new int[2 * size];                  // leaves at tree[size + v]
    int best = 0;
    for (int v : nums) {
        int len = 1 + maxOn(tree, size, Math.max(0, v - k), v);    // values in [v - k, v)
        best = Math.max(best, len);
        int i = size + v;
        tree[i] = Math.max(tree[i], len);
        for (i /= 2; i >= 1; i /= 2) tree[i] = Math.max(tree[2 * i], tree[2 * i + 1]);
    }
    return best;
}

private int maxOn(int[] tree, int size, int lo, int hi) {           // max over values [lo, hi)
    int res = 0;
    for (lo += size, hi += size; lo < hi; lo /= 2, hi /= 2) {
        if ((lo & 1) == 1) res = Math.max(res, tree[lo++]);
        if ((hi & 1) == 1) res = Math.max(res, tree[--hi]);
    }
    return res;
}
```

**Why not the deque of Ch 33?** The window `[v − k, v − 1]` is in value space, and consecutive elements jump around in it, so there's no sliding order. **Why not plain LIS with binary search (Ch 29)?** That trick keeps only the smallest tail for each length, which forgets the values the difference limit needs.

---

## 7. Static ranges: the sparse table

### Worked Q6 · [CSES Static Range Minimum Queries](https://cses.fi/problemset/task/1647)

> n, q ≤ 2·10⁵. The array never changes; answer q queries "min of `a[l..r]`".

Without updates, there's a stronger trade. **Precompute the min of every block whose length is a power of two:** `sp[j][i]` = min of `a[i .. i + 2ʲ)`, built level by level from two halves. That's n log n values.

**The query in O(1):** for a range of length L, let 2ʲ be the largest power of two ≤ L. The two blocks of length 2ʲ starting at l and ending at r **cover the range, overlapping in the middle**. For **min**, counting an element twice does no harm (min(x, x) = x; such operations are called *idempotent*), so the answer is the min of the two blocks.

```text
a:          index 0  1  2  3  4  5  6  7  8  9
query [1, 8]: length 8 → j = 3, block length 2³ = 8:
            block from l = 1:   [1 ─────────────── 8]
            block ending at r:  [1 ─────────────── 8]       one block covers it exactly
query [2, 8]: length 7 → j = 2, block length 4:
            block from l = 2:   [2 ──── 5]
            block ending at 8:           [5 ──── 8]          overlap at 5: harmless for min
```

```java
int levels = 32 - Integer.numberOfLeadingZeros(n);         // 2^(levels-1) <= n
int[][] sp = new int[levels][];
sp[0] = a.clone();
for (int j = 1; j < levels; j++) {
    int half = 1 << (j - 1);
    sp[j] = new int[n - (1 << j) + 1];
    for (int i = 0; i < sp[j].length; i++) sp[j][i] = Math.min(sp[j - 1][i], sp[j - 1][i + half]);
}

// min of a[l..r], 0-based inclusive
int j = 31 - Integer.numberOfLeadingZeros(r - l + 1);      // floor(log2(length))
int answer = Math.min(sp[j][l], sp[j][r - (1 << j) + 1]);
```

(Read with FastReader, convert the 1-based queries, and collect the output in a StringBuilder, as in Q3.) **O(n log n) build, O(1) query.** **Sums aren't idempotent**: the overlap would be counted twice. Static sums need only prefix sums anyway. Sparse tables suit min, max, gcd, AND and OR.

---

## 8. Choosing the structure

| Need | Use |
| :--- | :--- |
| Static array, range **sum** | Prefix sums (Ch 8) |
| Static array, range **min / max / gcd** | Sparse table |
| Point updates + range **sum** (or XOR) | Fenwick |
| Point updates + range **min / max / any associative merge** | Segment tree (the iterative form if the combine is commutative) |
| "The first index where …" | Segment tree walk |
| **Range** updates + point queries (sums) | Fenwick over the **difference array** (Ch 8 §5) |
| Range updates + range queries | Lazy segment tree |
| "Count earlier elements smaller than me" | Sweep + Fenwick over compressed values |
| Queries that can be answered in any order | **Offline:** sort the queries (by right end, by value), then sweep with a Fenwick tree |

**Offline processing** deserves a line of its own. When all queries are known in advance, answering them in a *convenient* order often turns a hard query into a prefix query. CSES Distinct Values (practice) is the model case: sort the queries by right end, sweep r, and keep a 1 only at the **last** occurrence of each value seen so far. Then "distinct values in `[a, r]`" is a range sum.

---

## 9. Where these break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Coordinates up to 10⁹ with **online** queries | Can't compress before seeing them | A dynamic segment tree (create children on first touch), or a `TreeMap` (Ch 15) if the operation allows |
| 2D range queries | One dimension isn't enough | 2D prefix sums (static), a 2D Fenwick for small grids; beyond that, out of scope (Ch 37) |
| Queries on tree **paths** | Paths aren't contiguous ranges | Binary lifting (Ch 36); heavy-light decomposition is out of scope (Ch 37) |
| Queries on **subtrees** | — | The Euler tour makes subtrees contiguous (Ch 36) |
| A non-associative combine | Blocks can't be merged | Rethink the node record (store prefix/suffix/best, Mixed Set VII) |

---

## 10. Recognition

**Signals:**
* Updates and queries **interleaved**, n and q up to 10⁵–2·10⁵. If there are no updates, the answer is prefix sums or a sparse table.
* "For each element, how many **earlier / later** elements are smaller / larger" (counting pairs): a sweep plus a Fenwick tree over values. Merge-sort counting is the alternative.
* "The **first** position / the **k-th** remaining element": a segment tree walk.
* A DP whose transition is a max over a **value range** (§6).
* "Book / assign / remove and report": a design problem whose state is an array under range changes.

**False friends:**
* Sliding window max/min: a monotonic deque (Ch 12) is simpler and O(n).
* The k-th smallest of a stream with no deletions: two heaps (Ch 15).
* Range additions **all before** any query: a difference array (Ch 8), no tree.
* "Count the pairs i < j with a[i] > a[j]" once, for the whole array: merge-sort counting (Ch 9) needs no extra structure.

---

## 11. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Fenwick index 0 | `i += i & -i` never moves from 0: an infinite loop. Shift to 1-based |
| A segment tree array of size 2n with the recursive layout | Use 4n |
| Forgetting `push` before descending in a lazy tree | Push in **every** function that goes below a node, query included |
| Range sums overflowing `int` | `long` nodes |
| Compressing only the array values, not the query values (offline) | Collect every value that will ever be looked up, then compress |
| Closed vs half-open ranges mixed up (especially after compression) | Pick one per structure and write it in the comment; Q4 uses segments, Q5 uses `[lo, hi)` |
| `Math.log` for the sparse-table level | `31 - Integer.numberOfLeadingZeros(len)`: exact, no floating point |

---

## 12. Practice

1. **[CSES Dynamic Range Sum Queries](https://cses.fi/problemset/task/1648)** *(Easy; stdin/stdout)*
2. **[CSES Range Update Queries](https://cses.fi/problemset/task/1651)** *(Easy; stdin/stdout)*
3. **[CSES List Removals](https://cses.fi/problemset/task/1749)** *(Medium; stdin/stdout)*
4. **[My Calendar III](https://leetcode.com/problems/my-calendar-iii/)** *(Hard)*
5. **[CSES Salary Queries](https://cses.fi/problemset/task/1144)** *(Medium; stdin/stdout)*
6. **[CSES Distinct Values Queries](https://cses.fi/problemset/task/1734)** *(Medium; stdin/stdout)*
7. **[Create Sorted Array through Instructions](https://leetcode.com/problems/create-sorted-array-through-instructions/)** *(Hard)*
8. **[Count Good Triplets in an Array](https://leetcode.com/problems/count-good-triplets-in-an-array/)** *(Hard)*

**Stretch:** **[Booking Concert Tickets in Groups](https://leetcode.com/problems/booking-concert-tickets-in-groups/)** *(Hard)* · **[Handling Sum Queries After Update](https://leetcode.com/problems/handling-sum-queries-after-update/)** *(Hard)* · **[Rectangle Area II](https://leetcode.com/problems/rectangle-area-ii/)** *(Hard)*

<details><summary>Nudges</summary>

1. Dynamic Range Sum: Q1 exactly, with `long` sums and 1-based input.
2. Range Update: "add u to a[a..b]" is two point updates on the **difference array**: +u at a, −u at b + 1. The value at k is then the prefix sum up to k.
3. List Removals: keep a 1 at every position still present. The k-th remaining element is a walk (go left if the left count ≥ k, else subtract it and go right). Then set that position to 0.
4. Calendar III: each booking adds +1 on `[start, end)`; report the max over all points. With ≤ 400 bookings, the TreeMap sweep of Ch 14 passes. The scalable version is a lazy segment tree with range add + max over compressed (or dynamically created) coordinates.
5. Salary Queries: values up to 10⁹, so read **all** operations first and compress every value that appears (initial, updated, and query bounds). Then a Fenwick tree of counts per value: a change is −1 at the old value and +1 at the new one.
6. Distinct Values: offline. Sort the queries by right end. Sweep r: if a[r] was seen at position p, put 0 at p, and put 1 at r. A query [a, r] is then the sum over [a, r].
7. Sorted Array Instructions: for each value, cost = min(the count of smaller values inserted so far, the count of larger ones). Both come from one Fenwick tree over values (larger = inserted − prefix(v)). Mod 10⁹ + 7.
8. Good Triplets: map each value to its position in nums2, and rewrite nums1 as those positions. A good triplet is now an increasing triplet of positions. For each middle element, multiply (smaller to its left) × (larger to its right), both from Fenwick trees. Use `long`.
9. Stretch, Concert Tickets: per row, store the seats left. Use a segment tree with max (for `gather`: the first row ≤ maxRow with ≥ k free seats, a walk) and sum (for `scatter`: is the total over rows ≤ maxRow ≥ k?). `scatter` then fills rows from the first non-empty one, with a pointer that only moves forward.
10. Stretch, Sum Queries: a lazy segment tree over nums1 storing the count of ones, with a **flip** note. Type-2 queries add p · (count of ones) to a running `long` total.
11. Stretch, Rectangle Area II: sweep over x. Each rectangle is +1 on its y-range at its left edge and −1 at its right edge. A segment tree over compressed y stores the **covered length** (a node is fully covered if its count > 0; otherwise its length is the sum of its children's). Area += covered length × Δx, mod 10⁹ + 7.
</details>

<details><summary>Key</summary>

Dynamic Range Sum → Q1 · Range Update → Fenwick on a difference array · List Removals → Q3 (k-th present element) · Calendar III → Q4 or sweep · Salary Queries → offline compression + Q1 · Distinct Values → offline by right endpoint + Q1 · Sorted Array Instructions → Q2 · Good Triplets → position mapping + Q2 from both sides · Concert Tickets → Q3 with max and sum · Sum Queries → Q4 with range flip · Rectangle Area II → sweep + coverage segment tree
</details>

---

## 13. Self-test

<details><summary>1. Why does a segment tree query visit only O(log n) nodes?</summary>

At each level, only the nodes containing ql or qr can partially overlap the query. Every other node is fully inside (answered from its stored value) or disjoint. So at most 2 nodes per level recurse.
</details>

<details><summary>2. Which block does Fenwick position 24 store, and why can't a Fenwick tree answer arbitrary range minimums?</summary>

24 = 11000₂, lowbit 8: a[17..24]. A range query is prefix(r) − prefix(l − 1), and min has no inverse to "subtract" the prefix before l.
</details>

<details><summary>3. In Count of Smaller Numbers After Self, why scan from the right, and why compress?</summary>

From the right, the elements after i are exactly those already inserted. Compression maps arbitrary values to ranks 1..m so they can index the tree, without changing any comparison.
</details>

<details><summary>4. What does a lazy note mean, and when must it be pushed?</summary>

"This update applies to my whole range, and my children haven't seen it yet." Push it before any operation descends into the node's children.
</details>

<details><summary>5. Why does the sparse table answer min in O(1) but not sum?</summary>

The two power-of-two blocks overlap. Min is idempotent, so counting the overlap twice is harmless; a sum would count it twice.
</details>

<details><summary>6. LIS II has a window [v − k, v − 1]. Why doesn't Ch 33's deque apply?</summary>

The window is in value space, and successive elements' windows move arbitrarily, not monotonically, so "leaves the window later" has no meaning.
</details>

---

## 14. Exit gate

Move on when you can:

* explain the block idea behind both structures and prove the O(log n) bounds;
* write the Fenwick tree in 2 minutes and the recursive segment tree (build, query, update) in 8, from blank files;
* add a walk and a lazy note to the segment tree;
* build a sparse table and query it with `numberOfLeadingZeros`;
* choose among prefix sums, sparse table, Fenwick and segment tree from the §8 table;
* solve 6 of the 8 practice problems.
