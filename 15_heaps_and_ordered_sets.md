# Chapter 15 — Heaps & Ordered Sets

> **Part II · Week 12** · Prerequisites: Ch 3, Ch 4 §20–22, Ch 9, Ch 13 (for merging lists)
> **Problems:** 7 worked (1 ★) · 8 practice · 3 stretch · **Time:** about 8 hours

Many problems repeatedly ask the same question of a changing collection: *what is the smallest (or largest) right now?* Sorting answers it once, but the collection keeps changing: new tasks arrive, finished ones leave, the window slides. Re-sorting every time is O(n log n) per question. A **heap** answers "the smallest" in O(1) and absorbs each change in O(log n). An **ordered set** (`TreeMap`/`TreeSet`) does more, adding arbitrary deletion and nearest-value queries, for a larger constant.

This chapter builds a heap from scratch, then covers the five patterns where interviews use heaps: the top-k, merging sorted sources, two heaps for a median, deletions in a sliding window, and greedy scheduling.

---

## 0. From zero: the binary heap

### 0.1 The idea

We want three operations to be fast: **insert**, **look at the minimum**, and **remove the minimum**. A sorted array gives O(1) minimum access but O(n) inserts. An unsorted array gives O(1) inserts but an O(n) minimum. A heap is the compromise in between: **just enough order to find the minimum, and no more.**

Picture the elements arranged as a **tree**, like a tournament bracket. Each position has up to two *children* below it and one *parent* above, and the top position is the *root*. (Ch 19 covers trees in general; here you only need parents and children.) A **min-heap** keeps one rule:

> **Heap property:** every element is ≤ its children. So the root is the minimum.

The tree is always kept **complete**: every level is full except possibly the last, which fills from the left. That shape needs no references at all. Store the elements level by level in an array, and the relationships become arithmetic:

```text
            index 0 (root)
           /              \
       index 1           index 2
       /     \           /     \
   index 3  index 4  index 5  index 6
```

> children of `i`: `2i + 1` and `2i + 2`   ·   parent of `i`: `(i − 1) / 2`

A complete tree with n elements has about log₂ n levels. That height is the cost of every update.

### 0.2 The operations

* **Insert:** put the new element in the first free slot (the end of the array), then **sift it up**. While it's smaller than its parent, swap the two. The only place the heap property can be broken is between the new element and its parent, and each swap moves that problem one level up. **O(log n).**
* **Remove the minimum:** the root is the answer. Move the **last** element into the root, shrink the array, then **sift it down**. While it's larger than its smaller child, swap with that child. **O(log n).**
* **Peek:** the root. **O(1).**

**The same heap, two views.** The array `[1, 3, 2, 7, 4, 5]` is a valid min-heap: read level by level, it's this tree.

```text
               1                 index:  0  1  2  3  4  5
            /     \              value:  1  3  2  7  4  5
           3       2
          / \     /              Check: 1 ≤ 3, 2;   3 ≤ 7, 4;   2 ≤ 5.
         7   4   5               Not sorted (3 comes before 2), but the root is the minimum.
```

**push(0):** place it in the first free slot (index 6, the child of index 2), then sift up.

```text
[1 3 2 7 4 5 0]    0 < parent 2 (index 2): swap    →  [1 3 0 7 4 5 2]
[1 3 0 7 4 5 2]    0 < parent 1 (index 0): swap    →  [0 3 1 7 4 5 2]
                   index 0 is the root: stop              2 swaps = the tree's height
```

**pop()** on that heap: the answer is the root, 0. Move the last element (2) into the root, then sift down.

```text
[2 3 1 7 4 5]      children 3, 1: the smaller is 1, and 2 > 1: swap    →  [1 3 2 7 4 5]
[1 3 2 7 4 5]      index 2's only child is 5, and 2 ≤ 5: stop
```

Each operation walks one root-to-leaf path at most, so it costs O(log n). Sift-down swaps with the **smaller** child: swapping with the larger one would put it above the smaller one, breaking the rule.

### 0.3 Heapify in O(n), not O(n log n)

Building a heap from n elements with n inserts costs O(n log n). Faster: put everything in the array, then **sift down every position from the last parent back to the root**. It's correct because, by the time position i is processed, both of its children are already roots of valid heaps, and sifting i down merges them into one.

**Why it's O(n).** A sift-down costs at most the height *below* the node. Half the nodes are leaves (cost 0), a quarter have height 1, an eighth height 2, and so on. The total is n · (0/2 + 1/4 + 2/8 + 3/16 + …) ≤ n. Most nodes are near the bottom, where sifting is cheap.

### Worked P0 *(custom)* · Build a binary min-heap

```java
final class MinHeap {
    private int[] a = new int[16];
    private int size = 0;                      // the heap is a[0, size)

    void push(int x) {
        if (size == a.length) a = Arrays.copyOf(a, size * 2);
        a[size] = x;
        siftUp(size++);
    }

    int peek() {
        if (size == 0) throw new NoSuchElementException();
        return a[0];
    }

    int pop() {
        if (size == 0) throw new NoSuchElementException();
        int min = a[0];
        a[0] = a[--size];                      // move the last element to the root,
        siftDown(0);                           // and let it sink to its place
        return min;
    }

    int size() { return size; }

    private void siftUp(int i) {
        while (i > 0) {
            int parent = (i - 1) / 2;
            if (a[parent] <= a[i]) break;      // the heap property holds: done
            swap(i, parent);
            i = parent;
        }
    }

    private void siftDown(int i) {
        while (true) {
            int left = 2 * i + 1, right = left + 1, smallest = i;
            if (left < size && a[left] < a[smallest]) smallest = left;
            if (right < size && a[right] < a[smallest]) smallest = right;
            if (smallest == i) break;          // both children are larger: done
            swap(i, smallest);
            i = smallest;
        }
    }

    private void swap(int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }

    static MinHeap heapify(int[] values) {     // O(n)
        MinHeap h = new MinHeap();
        h.a = Arrays.copyOf(values, Math.max(16, values.length));
        h.size = values.length;
        for (int i = h.size / 2 - 1; i >= 0; i--) h.siftDown(i);   // every parent, bottom-up
        return h;
    }
}
```

Verify it against `PriorityQueue<Integer>` with 100 000 random push/pop/peek operations, using the Ch 5 harness pattern.

**Heap sort** falls out for free: heapify (O(n)), then pop n times (O(n log n)). Done in place with a max-heap, it sorts in O(n log n) worst case with O(1) extra space. That's merge sort's guarantee without its buffer, at the cost of stability and cache-friendliness.

**Java's `PriorityQueue`** is exactly this structure, generic over a comparator (Ch 4 §20). Remember its limits:
* `remove(x)` and `contains(x)` are O(n) scans;
* iteration order isn't sorted;
* it has no "decrease the priority of x" operation. Insert a new entry and ignore the stale one (lazy deletion, §4).

**Common misconceptions:**

| Belief | Reality |
| :--- | :--- |
| "A heap is a sorted array" | Only the root is guaranteed; siblings and cousins are in no particular order |
| "Printing a `PriorityQueue` shows its elements in order" | It prints the array order. Pop repeatedly to get sorted order |
| "`new PriorityQueue<>()` gives the largest first" | It's a **min**-heap. For a max-heap, use `Collections.reverseOrder()` or a comparator |
| "Building a heap from n elements is O(n log n)" | n pushes are; heapify (§0.3) is O(n) |
| "Changing an element's priority in place is fine" | The heap doesn't notice, and its order breaks. Remove and re-insert, or use lazy deletion |

### 0.4 Ordered sets: what `TreeMap` is underneath

A `TreeMap` / `TreeSet` is a **balanced binary search tree**. Its elements are arranged so that walking the tree in a particular order visits them sorted, and its height is kept at O(log n) by automatic rebalancing. Ch 20 builds binary search trees and explains why balance matters. For now, the contract is what counts:

| Need | Heap (`PriorityQueue`) | Ordered set (`TreeMap` / `TreeSet`) |
| :--- | :--- | :--- |
| Minimum *or* maximum | O(1) peek (one end only) | O(log n), **both ends** |
| Insert, remove the min | O(log n) | O(log n) |
| Remove an **arbitrary** element | O(n) | **O(log n)** |
| Nearest value (`floor`, `ceiling`) | ✗ | O(log n) |
| Duplicates | ✓ | Need counts (multiset), or a tie-breaking comparator (Ch 4 §22) |
| Constant factor | small | larger |

**Rule of thumb:** use a heap when you only ever need the extreme value. Use an ordered set when you also delete arbitrary elements, need both ends, or ask for neighbours.

---

## 1. Attempt first ★

> **[Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) (15 minutes).** Design a class that receives numbers one at a time and, after each one, returns the k-th largest number seen so far. Sorting everything after every number is too slow.

---

## 2. A bounded heap for the top k

### Worked P1 · [Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/)

The k-th largest of everything seen so far is the **smallest of the k largest**. So keep exactly the k largest values in a **min-heap**. Its root is the answer, and when a new value arrives, the root is the one to evict.

```java
class KthLargest {
    private final PriorityQueue<Integer> topK = new PriorityQueue<>();   // min-heap holding the k largest so far
    private final int k;

    public KthLargest(int k, int[] nums) {
        this.k = k;
        for (int x : nums) add(x);
    }

    public int add(int val) {
        topK.offer(val);
        if (topK.size() > k) topK.poll();     // the smallest of k + 1 values can never be the k-th largest again
        return topK.peek();
    }
}
```

> **Invariant:** `topK` holds the k largest values seen so far. (Once k + 1 values are present, the smallest of them is below k others, and new values only add more.)

O(log k) per value and O(k) space. That's far better than sorting when k is much smaller than n.

**The direction is counter-intuitive, so learn it as a rule:** for the k **largest**, use a **min**-heap of size k (evict the smallest); for the k **smallest**, use a **max**-heap of size k. For a *static* array, quickselect is O(n) expected (Ch 9 §3). The heap wins for **streams**, and when you need a worst-case guarantee.

---

## 3. Merging k sorted sources: the frontier

### Worked P2a · [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

> Merge k sorted linked lists (N nodes in total) into one sorted list.

The next node of the result is **the smallest of the k current heads**. Keep those heads in a heap. Pop the smallest, append it, and push its successor:

```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> heads = new PriorityQueue<>((x, y) -> Integer.compare(x.val, y.val));
    for (ListNode head : lists) if (head != null) heads.offer(head);
    ListNode dummy = new ListNode(0), tail = dummy;          // Ch 13's dummy-node habit
    while (!heads.isEmpty()) {
        ListNode smallest = heads.poll();                     // the smallest remaining node overall
        tail.next = smallest;
        tail = smallest;
        if (smallest.next != null) heads.offer(smallest.next);   // its successor joins the frontier
    }
    return dummy.next;
}
```

**O(N log k)**: each node passes through a heap of size at most k once. (Merging the lists in pairs, like merge sort's tree, also gives O(N log k).)

**The general idea is a frontier.** The heap holds, for each sorted source, its **next unused element**. The overall next element is always the smallest one in the frontier, because each source is sorted, so nothing behind its front can be smaller.

### Worked P2b · [Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/)

> Given two sorted arrays, return the k pairs `(nums1[i], nums2[j])` with the smallest sums.

There are n₁ · n₂ pairs, too many to generate. But look at them as a grid: row i holds the pairs `(i, 0), (i, 1), …`, and **each row is sorted by sum** because `nums2` is sorted. That's k-way merging again, with one source per row:
* Start the frontier with `(i, 0)` for each row. Only the first k rows matter: row k and below sit behind at least k pairs with smaller or equal sums.
* Pop the smallest pair, output it, and push the next pair in its row, `(i, j + 1)`.

```java
public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
    // Entries are {i, j}. Sums are within ±2·10^9, which still fits in int.
    PriorityQueue<int[]> frontier = new PriorityQueue<>((p, q) ->
            Integer.compare(nums1[p[0]] + nums2[p[1]], nums1[q[0]] + nums2[q[1]]));
    for (int i = 0; i < Math.min(k, nums1.length); i++) frontier.offer(new int[] {i, 0});
    List<List<Integer>> result = new ArrayList<>();
    while (result.size() < k && !frontier.isEmpty()) {
        int[] p = frontier.poll();
        result.add(List.of(nums1[p[0]], nums2[p[1]]));
        if (p[1] + 1 < nums2.length) frontier.offer(new int[] {p[0], p[1] + 1});   // the next pair in this row
    }
    return result;
}
```

That's O(k log k). The same frontier idea produces the n-th Ugly Number (practice), the k-th smallest in a sorted matrix (the heap alternative to Ch 10's binary search), and Dijkstra's algorithm (Ch 23), where the frontier holds the nodes whose distance is known to be achievable.

---

## 4. Two heaps for the middle, and lazy deletion

### Worked P3 · [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/)

> Support `addNum(x)` and `findMedian()` over a growing stream.

The median splits the data into a **smaller half** and a **larger half**. Keep each half in a heap, oriented so its *inner* end is on top:
* `low`: a **max**-heap of the smaller half, whose top is the largest of the small values;
* `high`: a **min**-heap of the larger half, whose top is the smallest of the large values.

> **Invariants:** every value in `low` ≤ every value in `high`, and `low` has the same number of elements as `high`, or exactly one more. The median is then `low`'s top (odd count), or the average of the two tops (even count).

```java
class MedianFinder {
    private final PriorityQueue<Integer> low = new PriorityQueue<>(Collections.reverseOrder());   // max-heap
    private final PriorityQueue<Integer> high = new PriorityQueue<>();                            // min-heap

    public void addNum(int num) {
        low.offer(num);
        high.offer(low.poll());               // move low's largest up: restores "low <= high"
        if (high.size() > low.size()) low.offer(high.poll());   // restore the size rule
    }

    public double findMedian() {
        return low.size() > high.size() ? low.peek() : (low.peek() + (double) high.peek()) / 2.0;
    }
}
```

O(log n) per insert and O(1) per query. The `(double)` cast keeps the sum of two large `int`s from overflowing.

### Worked P4 · [Sliding Window Median](https://leetcode.com/problems/sliding-window-median/)

> Return the median of every window of length k.

Now elements also **leave**, and they can leave from either half. A heap can't remove an arbitrary element efficiently. There are two standard fixes:
* **Lazy deletion.** Leave departed elements inside the heaps, record them in a "to delete" count map, and discard them only when they reach a heap's top. Keep separate counts of the *valid* sizes. It's fiddly, but it's the general pattern. Stock Price Fluctuation (practice) and Dijkstra (Ch 23) both use it.
* **Ordered sets** for the two halves, which support O(log n) arbitrary removal directly. This is simpler to get right in Java.

**Lazy deletion, written out.** A `PriorityQueue` plus a map of "copies still to discard". `remove(x)` only records the debt; `prune()` pays it when a deleted value reaches the top, which is the only place it could ever be noticed:

```java
// A min-heap with remove(x), all operations O(log n) amortised.
final class LazyHeap {
    private final PriorityQueue<Integer> heap = new PriorityQueue<>();
    private final Map<Integer, Integer> toDelete = new HashMap<>();   // value -> copies still to discard
    private int size = 0;                                             // valid elements only

    void add(int x)    { heap.offer(x); size++; }
    void remove(int x) { toDelete.merge(x, 1, Integer::sum); size--; } // x must currently be present
    int peek()         { prune(); return heap.peek(); }
    int poll()         { prune(); size--; return heap.poll(); }
    int size()         { return size; }

    private void prune() {                        // discard deleted copies sitting on top
        while (!heap.isEmpty()) {
            Integer top = heap.peek(), pending = toDelete.get(top);
            if (pending == null) return;          // the top is a valid element
            heap.poll();
            if (pending == 1) toDelete.remove(top); else toDelete.put(top, pending - 1);
        }
    }
}
```

**Why it's correct:** a deleted copy that's still buried can't affect `peek` or `poll`; only the top matters, and `prune` cleans the top before anyone reads it. **Why it's fast:** each value enters the heap once and leaves it once (either as a real `poll` or in `prune`), so the total work is O(log n) per operation, amortised. **Keep `size` yourself:** `heap.size()` still counts the buried, deleted copies. Dijkstra's "skip stale entries" (Ch 23) is the same idea without the map: a stale entry is recognised by comparing it with `dist[]`.

With ordered sets, store **indices**, and compare them by value, breaking ties by index. Then equal values stay distinct elements (Ch 4 §22's comparator-equality trap).

```java
public double[] medianSlidingWindow(int[] nums, int k) {
    Comparator<Integer> byValue = (i, j) -> nums[i] != nums[j] ? Integer.compare(nums[i], nums[j])
                                                               : Integer.compare(i, j);
    TreeSet<Integer> low = new TreeSet<>(byValue), high = new TreeSet<>(byValue);   // indices
    double[] result = new double[nums.length - k + 1];
    for (int i = 0; i < nums.length; i++) {
        if (i >= k) {                                         // the element leaving the window
            int out = i - k;
            if (!low.remove(out)) high.remove(out);
        }
        low.add(i);
        high.add(low.pollLast());                             // keep every value in low <= every value in high
        while (high.size() > low.size()) low.add(high.pollFirst());   // low has the same size, or one more
        if (i >= k - 1) {
            result[i - k + 1] = (k % 2 == 1) ? nums[low.last()]
                    : ((double) nums[low.last()] + nums[high.first()]) / 2.0;
        }
    }
    return result;
}
```

O(n log k). Work through the four cases (k even or odd, the departing element in `low` or in `high`) to convince yourself that the single `while` always restores the size rule.

---

## 5. Greedy scheduling: counts first, then the heap

### Worked P5 · [Task Scheduler](https://leetcode.com/problems/task-scheduler/)

> Tasks (letters) must each run for one unit of time. Two runs of the same task need at least `n` idle-or-other units between them. Return the minimum total time.

A simulation with a max-heap of remaining counts works: each round, run the n + 1 most frequent remaining tasks. But the structure gives a formula, and its proof is the real lesson.

Let `f` be the highest frequency and `m` the number of tasks that have it.
* **Lower bound.** The most frequent task runs f times, with at least n units between consecutive runs. That takes at least `(f − 1)(n + 1) + 1` units. Each of the other m − 1 tasks with frequency f also needs a slot after the last "frame" begins, so the time is at least **`(f − 1)(n + 1) + m`**. It's also at least the **number of tasks**, since each takes a unit.
* **Achievable.** Lay out f frames of n + 1 slots (the last frame has only m). Put the most frequent tasks first in each frame, then fill the remaining slots *column by column* with the other tasks, most frequent first. Copies of a task in consecutive frames sit at least a frame apart, so the cooldown holds. If there are more tasks than slots, widen the frames. Then no slot is ever idle, and the time is exactly the number of tasks.

```java
public int leastInterval(char[] tasks, int n) {
    int[] count = new int[26];
    for (char t : tasks) count[t - 'A']++;
    int maxFreq = 0, tasksWithMax = 0;
    for (int c : count) {
        if (c > maxFreq) { maxFreq = c; tasksWithMax = 1; }
        else if (c == maxFreq) tasksWithMax++;
    }
    int framed = (maxFreq - 1) * (n + 1) + tasksWithMax;   // forced by the most frequent tasks
    return Math.max(framed, tasks.length);                  // overflow case: no idling needed at all
}
```

**When there's no formula, simulate with a heap.** Reorganize String (practice) is the n = 1 version that asks for the actual arrangement: repeatedly place the most frequent remaining letter that differs from the last one placed. Single-Threaded CPU and Process Tasks Using Servers (practice) are heap-driven simulations over time: one heap orders the *available* jobs, and another orders the *busy* resources by when they become free.

---

## 6. Where heaps break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Delete or update an arbitrary element | O(n) in `PriorityQueue` | Lazy deletion (§4), or an ordered set |
| Need the **k-th** element, not the top | A heap only exposes its root | Two heaps (median), an ordered set, or a Fenwick tree over values (Ch 34) |
| Need nearest values (floor / ceiling) | A heap has no order beyond the root | `TreeMap` / `TreeSet` |
| Static top-k | The heap costs O(n log k) | Quickselect O(n) (Ch 9), or buckets if the keys are small (Ch 9 §4) |
| A priority changes after insertion | Corrupts the heap silently (Ch 4 §20) | Reinsert and skip stale entries |

---

## 7. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| A k-way merge of huge files | External sort: sort chunks, then a heap of the chunk heads (Ch 9 §7) |
| The smallest range containing an element from each of k lists | A frontier heap, plus a running maximum (stretch) |
| Choose up to k projects to maximise capital | Sort by cost; a max-heap of the affordable profits (stretch, IPO) |
| Rooms that get reassigned as meetings end | Two heaps: free rooms by number, busy rooms by end time (stretch, Meeting Rooms III) |
| Shortest paths with weights | Dijkstra: the frontier heap with lazy deletion (Ch 23) |
| "Undo" a bad earlier choice | A regret heap: keep the past choices and swap out the worst (Ch 26; Furthest Building, practice) |

---

## 8. Recognition

**Signals:**
* "The **k-th** largest / smallest", "the **top k**", "the k closest", especially in a **stream**.
* **Repeatedly take** the smallest or largest remaining item, while new items keep arriving.
* **Merge** several sorted sequences, or generate items in sorted order from several "rows".
* The **median** (or another quantile) of a stream or window.
* **Simulations over time**: "process the job that finishes / arrives first", "the server that frees up first".
* An extreme value plus **arbitrary removal** or neighbours → an ordered set instead.

**False friends:** "the k-th smallest of a static array" (quickselect), and "the top k by frequency" with small counts (buckets, Ch 9).

---

## 9. Implementation traps

| Trap | Fix |
| :--- | :--- |
| A min-heap where a max-heap was needed, or vice versa | Say the eviction rule out loud: "the k largest → evict the smallest → min-heap" |
| A subtracting comparator (`(a, b) -> a - b`) | `Integer.compare` / `Long.compare` |
| Mutating an element that is already in the heap | Reinsert it; skip stale entries |
| `int x = pq.poll()` on an empty heap | Check `isEmpty()` first |
| A `TreeSet` comparator without a tie-break | Equal values vanish. Break ties by index |
| Overflowing an average of two `int`s | Cast to `double` (or `long`) before adding |
| Expecting `PriorityQueue` iteration or `toString()` to be sorted | Poll repeatedly |

---

## 10. Practice

1. **[Ugly Number II](https://leetcode.com/problems/ugly-number-ii/)** *(Medium)*
2. **[Reorganize String](https://leetcode.com/problems/reorganize-string/)** *(Medium)*
3. **[Single-Threaded CPU](https://leetcode.com/problems/single-threaded-cpu/)** *(Medium)*
4. **[Process Tasks Using Servers](https://leetcode.com/problems/process-tasks-using-servers/)** *(Medium)*
5. **[Furthest Building You Can Reach](https://leetcode.com/problems/furthest-building-you-can-reach/)** *(Medium)*
6. **[Maximum Subsequence Score](https://leetcode.com/problems/maximum-subsequence-score/)** *(Medium)*
7. **[Stock Price Fluctuation](https://leetcode.com/problems/stock-price-fluctuation/)** *(Medium)*
8. **[Design a Number Container System](https://leetcode.com/problems/design-a-number-container-system/)** *(Medium)*

**Stretch:** **[IPO](https://leetcode.com/problems/ipo/)** *(Hard)* · **[Smallest Range Covering Elements from K Lists](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/)** *(Hard)* · **[Meeting Rooms III](https://leetcode.com/problems/meeting-rooms-iii/)** *(Hard)*

<details><summary>Nudges</summary>

1. Ugly II: every ugly number is 2×, 3× or 5× a smaller one. Use a min-heap of candidates plus a set to skip duplicates (or three pointers into the list built so far). Use `long`.
2. Reorganize: it's impossible if some letter appears more than ⌈n/2⌉ times. Otherwise place the most frequent letter that differs from the previous one; a max-heap of (count, letter).
3. CPU: sort the tasks by arrival time. When the CPU is free, push every task that has arrived into a heap ordered by (processing time, index). If none has arrived, jump ahead in time.
4. Servers: two heaps. Free servers ordered by (weight, index); busy servers ordered by (free time, weight, index). Use `long` time.
5. Furthest Building: use ladders on the k largest climbs, bricks on the rest. Keep a min-heap of the climbs given ladders; when there are too many, convert the smallest to bricks.
6. Subsequence Score: sort the pairs by `nums2` descending. With each `nums2` as the minimum, you want the largest k `nums1` values seen so far: a min-heap of size k plus a running sum (`long`).
7. Stock Price: a map from timestamp to price (corrections overwrite), plus max and min heaps of (price, timestamp). Discard heap tops whose price no longer matches the map (lazy deletion).
8. Number Container: a map index → number, plus a map number → `TreeSet` of indices. Changing an index removes it from the old number's set.
9. Stretch, IPO: sort the projects by capital. Push every affordable project's profit into a max-heap and take the largest k times.
10. Stretch, Smallest Range: a frontier with one element per list, plus the current maximum. The range is [heap min, max]. Advance the list the minimum came from, and stop when any list runs out.
11. Stretch, Meeting Rooms III: sort the meetings by start. Free rooms by number; busy rooms by (end time, room). A meeting that must wait is delayed until the earliest room frees up, and keeps its duration.
</details>

<details><summary>Key</summary>

Ugly II → P2 frontier · Reorganize → P5 · Single-Threaded CPU → heap-driven simulation · Servers → two heaps over time · Furthest Building → a regret heap (keep ladders for the largest climbs) · Subsequence Score → sort + P1 · Stock Price → P4 lazy deletion · Number Container → map of TreeSets (§0.4) · IPO → sort + a heap of the affordable projects · Smallest Range → P2 with max tracking · Meeting Rooms III → two-heap simulation
</details>

---

## 11. Self-test

<details><summary>1. In an array-based heap, where are the children and the parent of index i?</summary>

The children are at 2i + 1 and 2i + 2; the parent is at (i − 1) / 2.
</details>

<details><summary>2. Why is bottom-up heapify O(n) rather than O(n log n)?</summary>

The cost of a sift-down is the node's height, and most nodes are near the bottom. Summing n/2^(h+1) nodes × h over all heights h gives at most n.
</details>

<details><summary>3. To track the k largest values in a stream, which heap do you use, and why?</summary>

A min-heap of size k. Its root is the smallest of the k largest, which is both the k-th largest and the one to evict when a bigger value arrives.
</details>

<details><summary>4. State the two invariants of the two-heap median.</summary>

Every value in the smaller-half max-heap is ≤ every value in the larger-half min-heap, and the max-heap has the same number of elements as the min-heap, or one more.
</details>

<details><summary>5. Why does the sliding-window median use TreeSets of indices rather than PriorityQueues of values?</summary>

Elements must be removed when they leave the window, and that's O(n) in a `PriorityQueue` but O(log k) in a `TreeSet`. Indices with a value-then-index comparator keep equal values as separate elements.
</details>

---

## 12. Exit gate

Move on when you can:

* build `MinHeap` (push, pop, peek, heapify) from a blank file, match `PriorityQueue` on random operations, and prove that heapify is O(n);
* choose between a heap and an ordered set for a given need, with a reason (§0.4);
* write Kth Largest in a Stream, Merge k Sorted Lists, Find Median from Data Stream and Sliding Window Median from blank files, each in under 12 minutes;
* prove the Task Scheduler formula's lower bound and explain how it's achieved;
* solve 7 of the 8 practice problems.
