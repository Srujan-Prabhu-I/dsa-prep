# Chapter 9 — Sorting as Structure

> **Part II · Week 6** · Prerequisites: Ch 1–8 (especially Ch 2 exchange arguments, Ch 6 recursion)
> **Problems:** 5 worked (1 ★) · 7 practice · 2 stretch · **Time:** about 8 hours

Sorting matters in two ways. First, as an **algorithm**: the ideas behind merge sort and quicksort (divide and conquer, partitioning, invariants) reappear across the whole book, and interviewers still ask you to implement them. Second, and more often, as a **tool**: sorting *creates structure* that the original order hid. Equal values sit together, the closest pairs become neighbours, greedy choices become provable, and two pointers and binary search become possible.

Ch 1, question 1, was *"does the order of the input matter?"* When the answer is no, sorting is usually the first move. This chapter shows what that move buys you, and what it costs.

---

## 0. From zero: what sorting is, and the simplest sorts

**Sorting** rearranges the elements into non-decreasing order. Two properties distinguish sorting algorithms beyond their speed:

* **Stable:** equal elements keep their original relative order. That matters when you sort records by one field after sorting them by another. Sort students by name, then *stably* by grade, and each grade stays alphabetical. Java's object sort (`Arrays.sort(Object[])`, `Collections.sort`) is stable. Primitive sorts don't need to be, because equal `int`s are indistinguishable. For example, sorting `(name, grade)` records by grade only:

```text
input:              (Ana, B) (Bo, A) (Cy, B) (Di, A)
stable sort:        (Bo, A) (Di, A) (Ana, B) (Cy, B)     Bo before Di and Ana before Cy, as in the input
an unstable sort
may produce:        (Di, A) (Bo, A) (Cy, B) (Ana, B)     also sorted by grade, but the ties are reordered
```
* **In place:** it uses only O(1) (or O(log n)) extra memory.

**Selection sort.** Repeatedly find the minimum of the unsorted part and swap it to the front. *Invariant:* `a[0, i)` holds the i smallest elements, in order. It's always Θ(n²). On `[5, 2, 4, 1]` (the bar separates the sorted part):

```text
i = 0: min of [5 2 4 1] is 1, swap it to the front   →  [1 | 2 4 5]
i = 1: min of [2 4 5] is 2, already in place         →  [1 2 | 4 5]
i = 2: min of [4 5] is 4, already in place           →  [1 2 4 | 5]
```

**Insertion sort.** Take each element in turn and slide it left into its place among the already-sorted prefix:

```java
static void insertionSort(int[] a) {
    for (int i = 1; i < a.length; i++) {
        int x = a[i], j = i - 1;       // invariant: a[0, i) is sorted
        while (j >= 0 && a[j] > x) {   // shift the larger elements one step right
            a[j + 1] = a[j];
            j--;
        }
        a[j + 1] = x;
    }
}
```

On `[5, 2, 4, 6, 1, 3]` (the bar separates the sorted prefix from the rest):

```text
start:           [5 | 2 4 6 1 3]
insert 2:        [2 5 | 4 6 1 3]      5 shifts right
insert 4:        [2 4 5 | 6 1 3]      5 shifts right
insert 6:        [2 4 5 6 | 1 3]      nothing shifts
insert 1:        [1 2 4 5 6 | 3]      four elements shift
insert 3:        [1 2 3 4 5 6]        4, 5, 6 shift
```

It's Θ(n²) in the worst case (reversed input). But each shift fixes exactly one **inversion** (a pair that's out of order), so its cost is O(n + number of inversions). On *nearly* sorted data, that's almost linear. It's also stable, and that combination is why real library sorts finish small sub-arrays with insertion sort.

**Why O(n log n) is the floor for comparison sorts.** A sort that only learns about the data by comparing pairs can be pictured as a decision tree: each comparison has two outcomes, and each leaf is one possible ordering of the input. There are n! possible orderings, so the tree needs at least n! leaves, and therefore depth at least log₂(n!) ≈ n log₂ n. **No comparison sort can beat Θ(n log n) in the worst case.** Counting sort (§4) "beats" it only by not comparing at all, using the values as indices instead.

**Common misconceptions:**

| Belief | Reality |
| :--- | :--- |
| "O(n log n) always beats O(n²)" | Asymptotically. On 10–20 elements, insertion sort is faster, which is why libraries use it for small pieces |
| "Sorting first is free" | It costs O(n log n). If the rest is O(n), the sort dominates, and an O(n) hashing solution may exist |
| "Sorting a `List<Integer>` costs the same as an `int[]`" | Same Big-O, but boxed values are several times slower (Ch 4 §13) |
| "The comparator `(a, b) -> a - b` is fine" | It overflows for large values of opposite sign; use `Integer.compare` (Ch 4 §14) |
| "Quicksort is O(n log n)" | **Expected** O(n log n) with a random pivot; a fixed pivot on sorted or all-equal input is Θ(n²) (§3) |

---

## 1. Merge sort: divide and conquer

**Derivation.** Insertion sort is slow because it fixes one inversion at a time. Suppose instead you had **two sorted halves**. You could combine them in linear time by repeatedly taking the smaller of the two front elements. That's the *merge*. And the halves can be sorted the same way, recursively. With the recursive contract from Ch 6 (§1): *"`mergeSort(lo, hi)` sorts `a[lo, hi)`"*.

* **Base case:** ranges of length 0 or 1 are already sorted.
* **Recursive case:** sort each half (trust the contract), then merge.
* **Cost:** T(n) = 2T(n/2) + O(n), so **Θ(n log n)** in every case (Ch 3 §4). It needs O(n) scratch space.

On `[5, 2, 4, 6, 1, 3]`, the calls split the array down to single elements, then the merges rebuild it on the way back up:

```text
split    [5 2 4 6 1 3]
         [5 2 4]          [6 1 3]
         [5]  [2 4]       [6]  [1 3]
              [2] [4]          [1] [3]
merge         [2 4]            [1 3]
         [2 4 5]          [1 3 6]
         [1 2 3 4 5 6]
```

There are about log₂ n levels, and the merges on each level touch every element once: that's the n log n. One merge, of `[2 4 5]` and `[1 3 6]`, compares the two front elements and takes the smaller each time:

```text
left [2 4 5]  right [1 3 6]   take 1 (right)   out [1]
left [2 4 5]  right [3 6]     take 2 (left)    out [1 2]
left [4 5]    right [3 6]     take 3 (right)   out [1 2 3]
left [4 5]    right [6]       take 4, then 5   out [1 2 3 4 5]
left []       right [6]       copy the rest    out [1 2 3 4 5 6]
```

### Worked P0 · [Sort an Array](https://leetcode.com/problems/sort-an-array/)

> Sort an array of up to 5·10⁴ integers in O(n log n), without library sorts.

```java
public int[] sortArray(int[] nums) {
    int[] a = nums.clone();
    mergeSort(a, new int[a.length], 0, a.length);     // one scratch buffer, shared by every call
    return a;
}

// Contract: sorts a[lo, hi).
private void mergeSort(int[] a, int[] buf, int lo, int hi) {
    if (hi - lo <= 1) return;
    int mid = lo + (hi - lo) / 2;
    mergeSort(a, buf, lo, mid);
    mergeSort(a, buf, mid, hi);
    merge(a, buf, lo, mid, hi);
}

// Merges the sorted runs a[lo, mid) and a[mid, hi) into sorted a[lo, hi).
private void merge(int[] a, int[] buf, int lo, int mid, int hi) {
    int i = lo, j = mid, k = lo;
    while (i < mid && j < hi) buf[k++] = (a[j] < a[i]) ? a[j++] : a[i++];   // ties take the LEFT: stable
    while (i < mid) buf[k++] = a[i++];
    while (j < hi) buf[k++] = a[j++];
    System.arraycopy(buf, lo, a, lo, hi - lo);
}
```

**Merge invariant:** `buf[lo, k)` holds the k smallest elements of the two runs, in order, and `i` and `j` point at the smallest elements not yet taken. Taking the smaller front element keeps that true. Taking the *left* element on ties is what makes merge sort stable.

Allocating one shared buffer, rather than a new array in each call, avoids the hidden O(n)-per-call cost from Ch 3 §6. Quicksort (§3) is the other classic answer to this problem.

---

## 2. Counting while merging

### Worked P1 *(custom)* · Count Inversions

> Count the pairs `i < j` with `a[i] > a[j]` (n up to 2·10⁵). `[3, 1, 2]` → 2 (the pairs (3,1) and (3,2)).

Checking every pair is O(n²). **Observation:** when merging two sorted runs, each time an element from the **right** run is taken while elements remain in the **left** run, it is smaller than *every* remaining left element. And each of those left elements sat *before* it in the original array. So that single step reveals `mid − i` inversions at once:

```java
static long countInversions(int[] a) {
    return sortAndCount(a.clone(), new int[a.length], 0, a.length);
}

// Sorts a[lo, hi) and returns the number of inversions inside it.
static long sortAndCount(int[] a, int[] buf, int lo, int hi) {
    if (hi - lo <= 1) return 0;
    int mid = lo + (hi - lo) / 2;
    long count = sortAndCount(a, buf, lo, mid) + sortAndCount(a, buf, mid, hi);
    int i = lo, j = mid, k = lo;
    while (i < mid && j < hi) {
        if (a[j] < a[i]) {
            count += mid - i;             // a[j] is smaller than all of a[i, mid), which came before it
            buf[k++] = a[j++];
        } else {
            buf[k++] = a[i++];
        }
    }
    while (i < mid) buf[k++] = a[i++];
    while (j < hi) buf[k++] = a[j++];
    System.arraycopy(buf, lo, a, lo, hi - lo);
    return count;
}
```

**Proof that every inversion is counted exactly once.** Take an inversion `(p, q)`, with `p < q` and `a[p] > a[q]`. There's exactly one recursion level where `p` and `q` are first split into different halves: `p` on the left, `q` on the right. Inside each half, sorting only reorders elements *within* that half, so `a[p]` is still in the left run and `a[q]` in the right. When `a[q]` is taken during the merge, `a[p]` hasn't been taken yet: ties go left, so only left elements ≤ `a[q]` were taken before it, and `a[p] > a[q]`. So this pair is counted at that step, and at no other level. ✓

**Use `long`:** up to n(n − 1)/2 ≈ 2·10¹⁰ inversions.

**The general pattern:** *count pairs `(i < j)` satisfying some relation between `a[i]` and `a[j]`*. Merge sort lets you count all pairs that cross a split in linear time, because both sides are sorted. Reverse Pairs (stretch) counts `a[i] > 2·a[j]` this way. Ch 34 solves the same family with a Fenwick tree.

---

## 3. Partitioning, then quicksort and quickselect

### Worked P2 · [Sort Colors](https://leetcode.com/problems/sort-colors/) ★

> **Attempt first (20 minutes).** An array contains only 0s, 1s and 2s. Sort it **in one pass, in place**, without counting.

Counting the 0s, 1s and 2s and rewriting the array takes two passes. The one-pass method maintains **four regions** with pointers between them. This is Dijkstra's *Dutch national flag* partition:

```text
   [ 0 0 0 | 1 1 1 | ? ? ? ? ? | 2 2 2 ]
   0      low     mid         high     n
```

> **Invariant:** `a[0, low)` are 0s, `a[low, mid)` are 1s, `a[mid, high)` are unexamined, and `a[high, n)` are 2s.

Look at `a[mid]` and shrink the unexamined region by one each step:

```java
public void sortColors(int[] nums) {
    int low = 0, mid = 0, high = nums.length;
    while (mid < high) {
        if (nums[mid] == 0) swap(nums, low++, mid++);   // the value arriving at mid is a 1 (or it's the same cell)
        else if (nums[mid] == 1) mid++;
        else swap(nums, mid, --high);                   // DON'T advance mid: the value swapped in is unexamined
    }
}

private void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
```

**Each case keeps the invariant:**
* a 0 swaps with the first 1 (at `low`) and both pointers advance;
* a 1 just extends the 1s region;
* a 2 swaps with the last unexamined cell, and that unexamined value now sits at `mid`, so `mid` must *not* advance.

When `mid == high`, nothing is unexamined. One pass, O(1) space.

### Quicksort: partition, then recurse

Swap the fixed values 0, 1, 2 for "**less than / equal to / greater than a pivot**", and the same loop becomes the partition step of **quicksort**:

```java
private final Random rng = new Random();

// Contract: sorts a[lo, hi).
private void quickSort(int[] a, int lo, int hi) {
    if (hi - lo <= 1) return;
    int pivot = a[lo + rng.nextInt(hi - lo)];           // random pivot: no input is reliably bad
    int lt = lo, i = lo, gt = hi;                        // [lo,lt) < pivot, [lt,i) = pivot, [gt,hi) > pivot
    while (i < gt) {
        if (a[i] < pivot) swap(a, lt++, i++);
        else if (a[i] > pivot) swap(a, i, --gt);
        else i++;
    }
    quickSort(a, lo, lt);                                // the "= pivot" block [lt, gt) is already in place
    quickSort(a, gt, hi);
}
```

One partition of `[5, 2, 4, 6, 1, 3]` with pivot 4 (`lt`, `i` and `gt` as in the code; `|` marks the region boundaries):

```text
[5 2 4 6 1 3]  lt=0 i=0 gt=6   5 > 4: swap with a[5], gt=5     [3 2 4 6 1 | 5]
[3 2 4 6 1 5]  i=0             3 < 4: swap a[0] with itself    [3 | 2 4 6 1 | 5]     lt=1 i=1
[3 2 4 6 1 5]  i=1             2 < 4                           [3 2 | 4 6 1 | 5]     lt=2 i=2
[3 2 4 6 1 5]  i=2             4 = 4: i=3
[3 2 4 6 1 5]  i=3             6 > 4: swap with a[4], gt=4     [3 2 | 4 | 1 | 6 5]
[3 2 4 1 6 5]  i=3             1 < 4: swap with a[lt=2]        [3 2 1 | 4 | 6 5]     lt=3 i=4 = gt: done
```

Now `[3 2 1]` < 4 ≤ `[4]` < `[6 5]`, and quicksort recurses on the two outer blocks only.

* **Why a random pivot?** If the pivot is always the first element, an already-sorted input splits into sizes 0 and n − 1 at every level, which is Θ(n²). A random pivot lands in the middle half of the values half the time, so the expected cost is **O(n log n)** for *every* input.
* **Why three regions instead of two?** With a two-way partition, an array of all-equal values splits as 0 and n − 1 every time, which is Θ(n²) again. The "equal" block takes all the duplicates out in one pass. LeetCode's Sort an Array includes exactly such tests.
* **Compared with merge sort:** quicksort sorts in place (O(log n) expected stack) and is fast in practice, but it isn't stable and has a (very unlikely) O(n²) worst case.

### Worked P3 · [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)

> Return the k-th largest element in O(n) expected time.

Sorting costs O(n log n), but we only need **one** position: in ascending order, the answer sits at index `n − k`. After one partition, we know which of the three blocks contains that index, so we **recurse into only that block**. That's *quickselect*:

```java
public int findKthLargest(int[] nums, int k) {
    int[] a = nums.clone();
    Random rng = new Random();
    int target = a.length - k;                  // the answer's index in ascending order
    int lo = 0, hi = a.length;                  // invariant: index `target` lies in [lo, hi)
    while (true) {
        int pivot = a[lo + rng.nextInt(hi - lo)];
        int lt = lo, i = lo, gt = hi;
        while (i < gt) {
            if (a[i] < pivot) swap(a, lt++, i++);
            else if (a[i] > pivot) swap(a, i, --gt);
            else i++;
        }
        if (target < lt) hi = lt;               // the answer is among the smaller values
        else if (target >= gt) lo = gt;         // ... among the larger values
        else return pivot;                      // it's in the block equal to the pivot
    }
}
```

**Why O(n) expected:** each round costs the current range size and (in expectation) shrinks the range by a constant factor, so the total is a geometric sum, n + 3n/4 + (3/4)²n + … = O(n). The worst case is O(n²), but with a random pivot it's vanishingly unlikely. A heap of size k gives O(n log k) worst case instead (Ch 15).

---

## 4. Not comparing at all: counting and bucket sort

When values are **small integers** in `[0, V)`, count them and write them back out in order:

```java
static int[] countingSort(int[] a, int maxValue) {      // every value is in [0, maxValue]
    int[] count = new int[maxValue + 1];
    for (int x : a) count[x]++;
    int[] sorted = new int[a.length];
    int k = 0;
    for (int v = 0; v <= maxValue; v++)
        while (count[v]-- > 0) sorted[k++] = v;
    return sorted;
}
```

That's **O(n + V)**, which beats n log n when V is about n or smaller. It doesn't contradict §0's lower bound, because it never compares two elements. (To sort *records* stably by a small key, turn the counts into starting positions with a prefix sum (Ch 8), then place each record at its key's next free position. Repeating that digit by digit is *radix sort*.)

**Bucket sort** generalises this: group items into buckets by some range of values, then handle each bucket. The bucket index can be **anything bounded**, not just the value itself.

### Worked P4 · [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)

> Return the k most frequent elements, in better than O(n log n).

Count the frequencies with a map (Ch 7). Sorting the distinct values by frequency is O(d log d). But **every frequency is between 1 and n**, which is a small bounded range, so use the frequency as a bucket index:

```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> count = new HashMap<>();
    for (int x : nums) count.merge(x, 1, Integer::sum);
    List<List<Integer>> withFrequency = new ArrayList<>();        // index f -> the values occurring f times
    for (int f = 0; f <= nums.length; f++) withFrequency.add(new ArrayList<>());
    for (Map.Entry<Integer, Integer> e : count.entrySet()) withFrequency.get(e.getValue()).add(e.getKey());
    int[] result = new int[k];
    int filled = 0;
    for (int f = nums.length; f >= 1 && filled < k; f--)         // from the highest frequency down
        for (int x : withFrequency.get(f)) {
            if (filled == k) break;
            result[filled++] = x;
        }
    return result;
}
```

**O(n).** The move to remember: **when the sort key is bounded by n, index buckets by it instead of comparing.** H-Index in the practice set uses the same trick: citations above n can be capped at n.

---

## 5. Sorting as a tool: what sorted order gives you

Most sorting in interviews isn't "implement a sort". It's *sort, and then the problem becomes easy*. Here is what sorted order guarantees, each with a one-line reason:

| After sorting… | Why | Enables |
| :--- | :--- | :--- |
| **Equal values are adjacent** | They're all equal, so nothing else can sit between them | Deduplication, grouping, run counting |
| **The closest pair of values is adjacent** | For `i < j` that aren't neighbours, `a[j] − a[i] ≥ a[i+1] − a[i]` | Minimum difference in O(n) after the sort |
| **The k smallest form a prefix** | By definition | Greedy choices of "cheapest first" |
| **Smallest-with-smallest matching is optimal** for many pairing problems | Exchange argument (Ch 2 §5) | Assign Cookies, Boats, Advantage Shuffle |
| **Monotone pointer moves become valid** | Moving right only ever increases values | Two pointers (Ch 11) |
| **"Is x present?" and "how many are ≤ x?" become O(log n)** | Halving a sorted range | Binary search (Ch 10) |

**Designing the order.** Often the right sort key isn't a raw field but **something derived from the problem**:
* Sort cars by position, then compare their *arrival times* (Car Fleet).
* Sort intervals by *end* rather than start (Ch 14).
* Sort numbers by a custom comparison like Largest Number's `a + b` vs `b + a` (Ch 2 §5). Any such order needs both the exchange argument *and* a consistency (transitivity) check.

**Sorting loses the original positions.** If you'll need them, sort **indices** by value (`Integer[] idx` with comparator `(x, y) -> Integer.compare(a[x], a[y])`), or sort `int[] {value, index}` pairs.

### Coordinate compression

When values are huge (up to 10⁹) but **only their relative order matters**, replace each value by its **rank** among the distinct values. Afterwards the values fit in `[0, d)`, so they can index arrays, which is what Fenwick and segment trees need (Ch 34):

```java
int[] distinct = Arrays.stream(a).distinct().sorted().toArray();
Map<Integer, Integer> rankOf = new HashMap<>();
for (int r = 0; r < distinct.length; r++) rankOf.put(distinct[r], r);
// rankOf.get(x) is x's 0-based rank; ties share a rank, and order is preserved
```

Rank Transform of an Array in the practice set is exactly this, using 1-based ranks.

---

## 6. Where sorting breaks

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| You need only the k-th element, or the top k | Sorting does unnecessary work | Quickselect, O(n) expected (§3), or a heap, O(n log k) (Ch 15) |
| The data arrives as a stream, and you need the current minimum or median | You can't re-sort after every arrival | Heaps (Ch 15) |
| Values are small integers | n log n is unnecessary | Counting / bucket sort, O(n + V) |
| The answer depends on the **original order** (contiguous subarrays, "the next greater element") | Sorting destroys exactly that information | Keep the order: prefix sums (Ch 8), stacks (Ch 12), windows (Ch 11) |
| The comparator isn't consistent | Garbage order or an exception (Ch 4 §14) | Prove transitivity, or sort by a numeric key |
| Many duplicates in quicksort | A two-way partition degrades to Θ(n²) | The three-way partition (§3) |

The fourth row is the classic beginner mistake: sorting an array in a problem about **subarrays**. Before sorting, ask Ch 1's question 1 out loud: *does the order matter?*

---

## 7. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| Count pairs `i < j` with `a[i] > 2·a[j]` | Count during the merge with a second pointer (stretch, Reverse Pairs) |
| Count smaller elements to the right of each element | Merge sort on indices (Ch 34 also does it with a Fenwick tree) |
| Sort a linked list | Merge sort; splitting uses fast/slow pointers (Ch 13) |
| Sort huge files that don't fit in memory | External merge sort: sort chunks, then do a k-way merge (Ch 15) |
| Maximum gap between neighbours after sorting, in O(n) | Pigeonhole buckets (stretch, Maximum Gap) |

---

## 8. Recognition

**Signals:**
* The order of the input **doesn't matter** to the answer, but some ordering would make the structure visible.
* Pairing or matching two groups ("assign", "match", "pair up").
* "The k-th smallest / largest", "the top k".
* Minimum difference or closest values.
* Counting pairs `i < j` with a relation between their values (inversions and relatives).
* Values huge, but only their comparisons matter → coordinate compression.

**False friends:**
* **Subarray / substring** problems: sorting destroys contiguity (§6).
* "The next greater element" and "days until warmer": they depend on position order, so use a stack (Ch 12).

---

## 9. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Advancing `mid` after swapping with `high` in the three-way partition | The swapped-in value is unexamined; leave `mid` where it is |
| A fixed pivot (first or last element) | A random pivot |
| A two-way partition with many duplicates | The three-way partition |
| Allocating a new array in every merge call | One shared buffer |
| `int` inversion counts | `long` |
| A subtracting comparator | `Integer.compare` (Ch 4 §14) |
| Sorting the caller's array when they still need it | `clone()` first, or sort indices |
| Using ties-right in merge when stability matters | Ties take the left element |

---

## 10. Practice

1. **[Rank Transform of an Array](https://leetcode.com/problems/rank-transform-of-an-array/)** *(Easy)*
2. **[K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)** *(Medium)*
3. **[H-Index](https://leetcode.com/problems/h-index/)** *(Medium)*
4. **[Minimum Increment to Make Array Unique](https://leetcode.com/problems/minimum-increment-to-make-array-unique/)** *(Medium)*
5. **[Array of Doubled Pairs](https://leetcode.com/problems/array-of-doubled-pairs/)** *(Medium)*
6. **[Car Fleet](https://leetcode.com/problems/car-fleet/)** *(Medium)*
7. **[Advantage Shuffle](https://leetcode.com/problems/advantage-shuffle/)** *(Medium)*

**Stretch:** **[Reverse Pairs](https://leetcode.com/problems/reverse-pairs/)** *(Hard)* · **[Maximum Gap](https://leetcode.com/problems/maximum-gap/)** *(Medium, with an O(n) requirement)*

<details><summary>Nudges</summary>

1. Rank Transform: §5's compression, with ranks starting at 1.
2. K Closest: sort by squared distance (no square roots needed; use `long` if the coordinates are large), or quickselect on that key for O(n) expected.
3. H-Index: the answer is at most n. Count the papers by citations, capped at n, then scan from the top, accumulating how many papers have at least h citations.
4. Min Increment: sort, then each element must be at least the previous one plus 1. Prove that processing in sorted order is optimal.
5. Doubled Pairs: process the values in order of absolute value; each x must be matched with a 2x that's still unused. Why does absolute value fix the negatives?
6. Car Fleet: sort by starting position, closest to the target first. Compute each car's arrival time. A car that would arrive no later than the fleet ahead of it joins that fleet.
7. Advantage Shuffle: sort A. For B's values from largest to smallest, beat each with A's largest remaining value if possible; otherwise sacrifice A's smallest. Prove it by exchange.
8. Stretch, Reverse Pairs: in each merge step, before merging, count `a[i] > 2·a[j]` across the halves with a pointer that only moves forward. Use `long` for `2·a[j]`.
9. Stretch, Maximum Gap: with n values between min and max, some gap is at least (max − min)/(n − 1). Make buckets of that width: the maximum gap never falls inside a bucket, so only each bucket's min and max matter.
</details>

<details><summary>Key</summary>

Rank Transform → coordinate compression (§5) · K Closest → selection (P3) or a sort by derived key · H-Index → bounded bucket counting (P4) · Min Increment → sort + greedy (§5) · Doubled Pairs → sort by absolute value + counts (§5) · Car Fleet → sort by position, compare arrival times (§5) · Advantage Shuffle → sort + exchange argument (§5) · Reverse Pairs → counting during the merge (P1) · Maximum Gap → bucket + pigeonhole (P4)
</details>

---

## 11. Self-test

<details><summary>1. Why can no comparison sort beat Θ(n log n) in the worst case?</summary>

Its comparisons form a binary decision tree that must distinguish all n! orderings, so its depth is at least log₂(n!) ≈ n log₂ n.
</details>

<details><summary>2. In the three-way partition, why does mid advance after swapping a 0 with low, but not after swapping a 2 with high?</summary>

The value arriving from `low` comes from the region already known to be 1s (or it's the same cell), so it's already classified. The value arriving from `high` comes from the unexamined region and still needs to be looked at.
</details>

<details><summary>3. Where exactly does the inversion count `mid − i` come from?</summary>

When a right-run element is taken, the left elements still waiting, `a[i, mid)`, are all larger than it (ties went left), and they all came before it in the original array. Each one forms an inversion with it.
</details>

<details><summary>4. Quickselect is O(n) expected. Why isn't it O(n log n) like quicksort?</summary>

It recurses into only *one* side, so the work shrinks geometrically (n + 3n/4 + …) instead of every level doing n work in total.
</details>

<details><summary>5. You're asked for the longest subarray with sum ≤ k. Should you sort?</summary>

No. Sorting destroys the contiguity that defines a subarray.
</details>

---

## 12. Exit gate

Move on when you can:

* write insertion sort, merge sort, the three-way partition, quicksort and quickselect from blank files, and state the invariant of each;
* prove the inversion-counting step, and explain the n log n lower bound;
* list four kinds of structure that sorted order creates, each with its one-line reason (§5);
* solve 6 of the 7 practice problems.
