# Chapter 10 — Binary Search: The Boundary Invariant

> **Part II · Week 7** · Prerequisites: Ch 1 (Derivation D), Ch 2, Ch 9
> **Problems:** 7 worked (1 ★) · 10 practice · 3 stretch · **Time:** about 8 hours

Most people learn binary search as three or four loop variants (`<` or `<=`? `mid` or `mid + 1`? return `lo` or `hi`?) and then fight off-by-one bugs forever. This chapter replaces all of them with **one template and one invariant**. Every variant then becomes a question about the *problem*, not the loop: *what is the predicate, and where are its boundaries?*

The second half is the part interviews actually test: **binary search on the answer.** You turn "find the minimum capacity / speed / largest sum" into "is this value good enough?", the move from Ch 1 Derivation D, now made general.

---

## 1. Attempt first ★

> **[Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) (25 minutes).** Packages with `weights` (in this fixed order) must be shipped within `days` days. Each day, the ship loads packages in order until the next one would exceed its capacity. Return the **minimum capacity** that ships everything within `days`. (`n ≤ 5·10⁴`, `1 ≤ weights[i] ≤ 500`.)
>
> `weights = [1,2,3,4,5,6,7,8,9,10], days = 5` → 15.

---

## 2. From the brute force to the predicate

**Steps 1–4 of the protocol.** The answer is a capacity `c`. It's at least `max(w)`, because the heaviest package must fit, and at most `sum(w)`, which ships everything in one day. The brute force tries every `c` from the smallest upwards and simulates: O(n · sum), up to 5·10⁴ × 2.5·10⁷, hopeless.

**Step 5, the bottleneck:** the simulations for consecutive capacities are nearly identical, and we run hundreds of thousands of them. **Step 6, question 4:** *is something monotone?* Define

> `canShip(c)`: can everything be shipped within `days` days with capacity `c`?

* **Monotone:** if capacity `c` works, any `c′ > c` works too, because the same daily loads still fit. So over `c = max(w) … sum(w)` the answers look like **F F … F T T … T**, and we want the **first T**.
* **Checkable in O(n):** load greedily, putting each package on the current day until it doesn't fit, then starting a new day. This uses the fewest days possible: by the same stays-ahead argument as Ch 1 Derivation D, after each day the greedy loader has shipped at least as many packages as any other valid schedule. So `canShip(c)` ⟺ greedy uses ≤ `days` days.

So the problem is now *find the boundary of a monotone predicate*. With a halving search, that takes about log₂(2.5·10⁷) ≈ 25 checks of O(n) each.

---

## 3. The template

> **Invariant:** `ok(lo)` is **false** and `ok(hi)` is **true**. The boundary, the first true, lies in `(lo, hi]`.
>
> While `hi − lo > 1`, test `mid` strictly between them and move whichever end has the same answer as `mid`. When `hi − lo = 1`, **`hi` is the first true and `lo` is the last false.**

```java
long lo = /* a value where ok is false (or a virtual one just outside the range) */;
long hi = /* a value where ok is true  (or a virtual one just outside the range) */;
while (hi - lo > 1) {
    long mid = lo + (hi - lo) / 2;       // lo < mid < hi, always
    if (ok(mid)) hi = mid; else lo = mid;
}
// hi = first value with ok true, lo = last value with ok false
```

**Why this is always right:**
* **The invariant holds at the start** by your choice of `lo` and `hi`, and is **maintained** because each branch moves the end that matches `mid`'s answer.
* **It terminates.** While `hi − lo ≥ 2`, `mid` lies strictly between them, so whichever end moves, the gap shrinks. No `mid + 1`, no special cases, no infinite loops.
* **`ok` is never evaluated at `lo` or `hi`**, only at values strictly between them. So `lo` and `hi` can be **virtual sentinels** that don't exist in the array (`−1` and `n`), and you never index out of bounds.
* **At the end** `hi = lo + 1`, with false at `lo` and true at `hi`: that's the boundary.

Choosing the sentinels is the only real decision, and it's a question about the *problem*: "what value is certainly bad?", "what value is certainly good?".

**The mirror orientation.** For "the *largest* value that works" (T T … T F F …, as in Ch 1 Derivation D), keep `ok(lo)` true and `ok(hi)` false, and return `lo`. It's the same template with the roles swapped. Both are "binary search for the boundary between two answers".

### Worked P3 · [Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/)

```java
public int shipWithinDays(int[] weights, int days) {
    int lo = 0, hi = 0;
    for (int w : weights) {
        lo = Math.max(lo, w);
        hi += w;                                   // at most 5·10^4 · 500 = 2.5·10^7: fits in int
    }
    lo -= 1;                     // max(w) - 1 can't carry the heaviest package: certainly false
    // Invariant: canShip(lo) is false, canShip(hi) is true (capacity = total ships it all in one day).
    while (hi - lo > 1) {
        int mid = lo + (hi - lo) / 2;
        if (canShip(weights, days, mid)) hi = mid; else lo = mid;
    }
    return hi;
}

// Greedy: fill each day as far as possible. Only called with capacity >= max(w).
private boolean canShip(int[] weights, int days, int capacity) {
    int daysUsed = 1, load = 0;
    for (int w : weights) {
        if (load + w > capacity) {                 // w doesn't fit today: start a new day
            daysUsed++;
            load = 0;
        }
        load += w;
    }
    return daysUsed <= days;
}
```

**O(n log(sum))**. Notice that `canShip` is only ever called with `mid > lo = max(w) − 1`, so every package fits on an empty ship and the greedy loop never gets stuck. **The sentinel choice guarantees the check's precondition.**

---

## 4. The classics, derived from the template

Every "standard" binary search is the template with a particular predicate on a sorted array. Use `lo = −1` (virtual false) and `hi = n` (virtual true):

| You want | Predicate `ok(i)` | Answer |
| :--- | :--- | :--- |
| First index with `a[i] ≥ x` (*lower bound*) | `a[i] >= x` | `hi` (`n` if none) |
| First index with `a[i] > x` (*upper bound*) | `a[i] > x` | `hi` |
| Last index with `a[i] ≤ x` | `a[i] > x` | `lo` (`−1` if none) |
| Number of elements equal to `x` | — | upper bound − lower bound |
| Is `x` present? | lower bound | `hi < n && a[hi] == x` |
| Insert position for `x` | lower bound | `hi` |

### Worked P1 · [Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

> Return `[first, last]`, the positions of `target` in a sorted array, or `[−1, −1]`, in O(log n).

```java
public int[] searchRange(int[] nums, int target) {
    int first = firstIndexWhere(nums, target, true);      // first i with nums[i] >= target
    if (first == nums.length || nums[first] != target) return new int[] {-1, -1};
    int afterLast = firstIndexWhere(nums, target, false); // first i with nums[i] > target
    return new int[] {first, afterLast - 1};
}

// First index i with nums[i] >= target (orEqual) or nums[i] > target (!orEqual); nums.length if none.
private int firstIndexWhere(int[] nums, int target, boolean orEqual) {
    int lo = -1, hi = nums.length;                        // virtual sentinels: false at -1, true at n
    while (hi - lo > 1) {
        int mid = lo + (hi - lo) / 2;
        boolean ok = orEqual ? nums[mid] >= target : nums[mid] > target;
        if (ok) hi = mid; else lo = mid;
    }
    return hi;
}
```

> **Don't use `Arrays.binarySearch` for boundaries.** With duplicates it returns *some* matching index, not the first; when the value is absent it returns `−(insertion point) − 1`. It's fine for "is it present?" on distinct values, and nothing else. `TreeMap.ceilingKey` / `floorKey` (Ch 4 §22) answer lower/upper-bound questions on a dynamic set.

---

## 5. Predicates that aren't "value ≥ x"

The template needs **some predicate on the index** that is false at `lo` and true at `hi`. It doesn't have to compare against a target value. The skill is **inventing that predicate**.

### Worked P2a · [Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)

> A sorted array of distinct values was rotated (`[0,1,2,4,5,6,7]` → `[4,5,6,7,0,1,2]`). Find the minimum in O(log n).

**Invent the predicate.** A rotated array is two increasing runs, and every value in the left run is larger than every value in the right run. The last element, `nums[n − 1]`, belongs to the right run and is its largest value. So

> `ok(i)`: `nums[i] ≤ nums[n − 1]`, which is true exactly on the right run: **F F F T T T**.

The first T is where the right run starts, and that's the minimum. (If the array wasn't rotated at all, every index is T and the first is 0. That's still correct.)

```java
public int findMin(int[] nums) {
    int n = nums.length, lo = -1, hi = n - 1;      // ok(n - 1) is trivially true
    while (hi - lo > 1) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] <= nums[n - 1]) hi = mid; else lo = mid;
    }
    return nums[hi];
}
```

*Search in Rotated Sorted Array* (practice) follows directly: find the minimum's index, which splits the array into two sorted runs, then run a lower-bound search on whichever run could contain the target.

### Worked P2b · [Find Peak Element: when the predicate isn't monotone](https://leetcode.com/problems/find-peak-element/)

> `nums[i] ≠ nums[i + 1]` for all `i`; imagine `nums[−1] = nums[n] = −∞`. Return the index of any **peak** (an element greater than both neighbours) in O(log n).

Try the predicate `ok(i)`: "`nums[i] > nums[i + 1]`", i.e. "the array goes down after `i`". It **isn't monotone**: on `[1, 3, 2, 4, 1]` it reads F T F T T.

But look at what the template's invariant actually gives you. At the end, `ok(lo)` is false and `ok(hi = lo + 1)` is true:
* `ok(lo)` false means `nums[lo] < nums[lo + 1] = nums[hi]`, so the array goes **up into** `hi`. (If `lo = −1`, that's the virtual −∞.)
* `ok(hi)` true means `nums[hi] > nums[hi + 1]`, so it goes **down after** `hi`. (If `hi = n − 1`, that's the other virtual −∞.)

**So `hi` is a peak.** The invariant holds at the start (`lo = −1` is virtually false, and `hi = n − 1` is virtually true), and each step keeps it.

```java
public int findPeakElement(int[] nums) {
    int lo = -1, hi = nums.length - 1;    // false at lo: up into lo + 1; true at hi: down after hi
    while (hi - lo > 1) {
        int mid = lo + (hi - lo) / 2;      // mid < hi <= n - 1, so nums[mid + 1] exists
        if (nums[mid] > nums[mid + 1]) hi = mid; else lo = mid;
    }
    return hi;
}
```

> **The deep fact about binary search.** It doesn't need monotonicity. **It finds *a* boundary between a false and a true whenever you can keep a false at one end and a true at the other**, much like the intermediate value theorem. Monotonicity only guarantees that the boundary is *unique*, which matters when you want "the first" or "the minimum". When any boundary will do (any peak), the invariant alone is enough.

---

## 6. Binary search on the answer: the method

Every "search on the answer" problem follows the same five steps. Write them down before coding:

1. **The answer space and sentinels.** What range can the answer possibly be in? Pick a value that's certainly bad and one that's certainly good.
2. **The predicate and its direction.** "Is `x` big enough?" (F…F T…T, return `hi`) or "is `x` small enough?" (T…T F…F, return `lo`).
3. **The monotonicity proof**, in one sentence: "if `x` works, `x + 1` works, because …". If you can't write it, the method doesn't apply.
4. **The check**, which is usually a greedy pass (with its own stays-ahead proof) or a counting pass.
5. **The cost:** log₂(range) × the cost of the check. The range can be 10¹⁸, which is only about 60 steps.

The practice set has five of these: Koko (speed), bouquets (days), Split Array (largest sum), the train speed problem, and the multiplication table. They differ only in steps 1, 2 and 4.

---

## 7. Searching values by counting

Sometimes the question is "find the k-th smallest" (or "find the duplicate") in a set that is too big, or too awkward, to sort. Search over **values** instead of positions, with a **counting** predicate.

### Worked P4a · [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

> `nums` has n + 1 integers, each in `[1, n]`. One value appears more than once (possibly many times). Find it **without modifying the array, using O(1) extra space**.

The array can't be sorted (no modifying) and can't be hashed (no extra space). Search over the *value* instead:

> `ok(x)`: "more than `x` elements are ≤ `x`".

**Monotonicity proof.** Let `d` be the duplicate, appearing `k ≥ 2` times. There are n + 1 elements and n + 1 − (k − 1) = n − k + 2 distinct values, so exactly k − 2 values of `[1, n]` are missing.
* For `x < d`, every value ≤ x appears at most once, so at most x elements are ≤ x, and `ok(x)` is **false**.
* For `x ≥ d`, the values ≤ x that are present number at least x − (k − 2), and `d` contributes k − 1 *extra* copies, so the count is at least x − (k − 2) + (k − 1) = x + 1 > x, and `ok(x)` is **true**.

So the first true is exactly `d`.

```java
public int findDuplicate(int[] nums) {
    int lo = 0, hi = nums.length - 1;      // values are in [1, n] with n = nums.length - 1; ok(n) is true
    while (hi - lo > 1) {
        int mid = lo + (hi - lo) / 2;
        int atMost = 0;
        for (int x : nums) if (x <= mid) atMost++;
        if (atMost > mid) hi = mid; else lo = mid;
    }
    return hi;
}
```

That's O(n log n) time and O(1) space. (Ch 13 gives an O(n) solution by viewing the array as a linked structure with a cycle.)

### Worked P4b · [Kth Smallest Element in a Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/)

> In an n × n matrix whose rows and columns are each sorted ascending, find the k-th smallest element.

> `ok(v)`: "at least k cells are ≤ v". It's monotone in `v` (raising `v` only adds cells), and the first true is the answer.

**Why the answer is always an actual matrix value:** if the first true `v` weren't in the matrix, then exactly the same cells would be ≤ `v − 1`, so `ok(v − 1)` would be true too, contradicting "first".

**Counting the cells ≤ v in O(n), without binary-searching every row.** Start at the bottom-left corner. If that cell is ≤ v, then (because columns are sorted) the whole column above it is ≤ v as well: count them all and move right. Otherwise, that row is too big from this column onwards: move up. Each step moves right or up, so there are at most 2n steps.

```java
public int kthSmallest(int[][] matrix, int k) {
    int n = matrix.length;
    // values are in [-1e9, 1e9], so hi - lo <= 2e9 + 1 still fits in int
    int lo = matrix[0][0] - 1, hi = matrix[n - 1][n - 1];      // ok(lo) false, ok(hi) true
    while (hi - lo > 1) {
        int mid = lo + (hi - lo) / 2;
        if (countAtMost(matrix, mid) >= k) hi = mid; else lo = mid;
    }
    return hi;
}

// Cells <= v, counted by walking a staircase from the bottom-left corner.
private int countAtMost(int[][] matrix, int v) {
    int n = matrix.length, count = 0, r = n - 1, c = 0;
    while (r >= 0 && c < n) {
        if (matrix[r][c] <= v) {
            count += r + 1;            // all of column c, rows 0..r, are <= v
            c++;
        } else {
            r--;                       // row r is > v from column c onwards
        }
    }
    return count;
}
```

**O(n log(range)).** The same shape solves Kth Smallest Number in Multiplication Table and Find K-th Smallest Pair Distance (practice and stretch): **search over the value; count how many are ≤ it.**

---

## 8. Searching over real numbers

### Worked P5 *(custom)* · Cube Root

> Return the real cube root of `x` (`|x| ≤ 10⁹`) to within 10⁻⁹.

`ok(m)`: "m³ ≥ x" is monotone in `m`. On reals there is no "adjacent" pair, so instead of looping until `hi − lo` is tiny, **run a fixed number of iterations**:

```java
static double cubeRoot(double x) {
    double lo = Math.min(-1.0, x), hi = Math.max(1.0, x);    // the root lies between them
    for (int iter = 0; iter < 100; iter++) {                  // each step halves the interval
        double mid = (lo + hi) / 2;
        if (mid * mid * mid >= x) hi = mid; else lo = mid;
    }
    return hi;
}
```

**Why a fixed count instead of `while (hi − lo > 1e-9)`?** For large magnitudes, two adjacent `double`s can be further apart than 10⁻⁹, so `hi − lo` might *never* get that small, and the loop never ends. 100 halvings shrink any starting interval by 2¹⁰⁰, which is far more precision than a `double` holds.

**Prefer integers when possible.** If the answer is really an integer, or the input has a fixed number of decimal places (Minimum Speed to Arrive on Time gives hours with two decimals), multiply everything by 100 and binary-search over integers. You avoid precision bugs entirely.

---

## 9. Where binary search breaks

| Situation | Why | What to do |
| :--- | :--- | :--- |
| You need *the first* boundary, but the predicate isn't monotone | There may be several F→T boundaries, and the template finds one of them | Prove monotonicity, or find a different predicate |
| **Duplicates in a rotated array** (Search in Rotated Sorted Array II) | When `nums[mid] == nums[n − 1]`, you can't tell which run `mid` is in. For `[1,1,1,0,1,1]` versus `[1,1,1,1,0,1]`, the probes look identical, so in the worst case every element must be inspected: **Ω(n)** | Shrink by one element when the comparison is ambiguous, and accept O(n) worst case |
| An "**exactly** k" predicate | "Exactly" isn't monotone | Search with "at least k", then check |
| The check is too slow | log(range) × check must fit the budget | A faster check (counting with a pointer instead of a scan) |
| A wrong sentinel | If `hi` isn't truly feasible, the answer comes back as `hi` silently | Prove both sentinels; return a "no answer" value when `hi` is virtual |
| Overflow inside the check | Sums of up to n × 10⁹ | `long` in the check |

The rotated-with-duplicates row is a common interview follow-up. The expected answer is "it degrades to O(n) in the worst case, and here's the input that shows why."

---

## 10. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| Return the actual shipping schedule | Run the greedy once more at the final capacity |
| Many queries with different `days` | Each query is its own binary search, or precompute the answer for every `days` value |
| The median of two sorted arrays | Binary search on how many elements to take from the first array (stretch) |
| Find the boundary in an infinite or unknown-size array | Double `hi` until `ok(hi)` is true (*exponential search*), then search normally |
| "Binary search" on a tree, by powers of two | Binary lifting (Ch 36) |
| The minimum of a unimodal function | Binary search on the sign of `f(x + 1) − f(x)` |

---

## 11. Recognition

**Signals:**
* A **sorted** array, or a structure that is sorted piecewise (rotated, a sorted matrix).
* An explicit **O(log n)** requirement.
* "**Minimum** X such that …" or "**maximum** X such that …", where checking a single X is easy. That includes "minimise the maximum" and "maximise the minimum".
* The **k-th smallest** of a large implicit set (all pairs, a multiplication table) where counting "how many are ≤ v" is easy.
* An answer range that's huge (up to 10⁹–10¹⁸) while the check is O(n).

**False friends:**
* "Minimise the **sum** / total cost" is usually DP or greedy. Feasibility of "total ≤ X" is monotone, but checking it is as hard as the original problem.
* "**Exactly** k …": not monotone as stated (§9).
* "Find any index with property P": only works if you can maintain a false at one end and a true at the other (§5).

---

## 12. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Mixing loop styles (`lo <= hi`, `lo < hi`, `mid + 1`, …) | One template: `while (hi − lo > 1)`, and move the end that matches |
| `(lo + hi) / 2` overflowing | `lo + (hi − lo) / 2` when both are ≥ 0. When the range spans negatives and positives, `hi − lo` itself can overflow `int` (and so can the loop test `hi − lo > 1`): use `long lo, hi`, as in the §3 template |
| Evaluating `ok` at a sentinel | Impossible with the template; `mid` is always strictly inside |
| A sentinel that isn't actually feasible / infeasible | Prove it, or make it virtual and handle "none" |
| `Arrays.binarySearch` for first/last occurrence | Write the bound search |
| `while (hi − lo > eps)` on doubles | A fixed number of iterations |
| `int` overflow in the check (`sum of hours`) | `long` |

---

## 13. Practice

1. **[Search Insert Position](https://leetcode.com/problems/search-insert-position/)** *(Easy)*
2. **[Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/)** *(Medium)*
3. **[Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array/)** *(Medium)*
4. **[Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)** *(Medium)*
5. **[Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/)** *(Medium)*
6. **[Minimum Number of Days to Make m Bouquets](https://leetcode.com/problems/minimum-number-of-days-to-make-m-bouquets/)** *(Medium)*
7. **[Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/)** *(Medium)*
8. **[Minimum Speed to Arrive on Time](https://leetcode.com/problems/minimum-speed-to-arrive-on-time/)** *(Medium)*
9. **[Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/)** *(Hard)*
10. **[Kth Smallest Number in Multiplication Table](https://leetcode.com/problems/kth-smallest-number-in-multiplication-table/)** *(Hard)*

**Stretch:** **[Find K-th Smallest Pair Distance](https://leetcode.com/problems/find-k-th-smallest-pair-distance/)** *(Hard)* · **[Maximum Running Time of N Computers](https://leetcode.com/problems/maximum-running-time-of-n-computers/)** *(Hard)* · **[Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)** *(Hard)*

<details><summary>Nudges</summary>

1. Insert Position: the lower bound (§4).
2. 2D Matrix: the rows laid end to end form one sorted array; index `i` is cell `(i / cols, i % cols)`.
3. Single Element: before the single element, pairs start at even indices; after it, at odd ones. Predicate on even `i`: `nums[i] != nums[i + 1]`.
4. Rotated: find the minimum's index (§5), then search the run that could contain the target.
5. Koko: "can she finish at speed s?" Hours per pile is `ceil(p / s)`. Sum the hours in `long`.
6. Bouquets: "can we make m bouquets by day d?" Scan for runs of bloomed flowers. If m · k > n it's impossible: return −1 before searching.
7. Rotated II: when `nums[mid] == nums[hi]`, you can't decide, so shrink `hi` by one. Explain why the worst case is O(n).
8. Train speed: the time is a sum of `ceil` values plus the exact last leg. Compare against `hour` exactly, e.g. scale by 100. What upper bound on the speed is safe?
9. Split Array: Ship Packages in disguise. The parts must be contiguous and there must be exactly k of them; why is "at most k" equivalent?
10. Multiplication Table: count the entries ≤ v row by row: `min(v / i, n)` in row i.
11. Stretch, Pair Distance: sort, then count the pairs with distance ≤ d using a pointer that only moves forward (Ch 3 §3).
12. Stretch, Running Time: "can all n computers run for T minutes?" Each battery contributes `min(battery, T)`. Feasible iff the sum ≥ n · T. Use `long`.
13. Stretch, Median: binary-search on how many elements `i` to take from the shorter array, so that the left halves of both arrays together hold half the elements, and every left element is ≤ every right element.
</details>

<details><summary>Key</summary>

Insert Position → P1 · 2D Matrix → P1 on a flattened index · Single Element → P2 (pair-parity predicate) · Rotated → P2 then P1 · Koko → P3 · Bouquets → P3 · Rotated II → where P2 breaks (§9) · Train Speed → P3 with exact arithmetic (§8) · Split Array → P3 + greedy check · Multiplication Table → P4 · Pair Distance → P4 + a forward-only pointer count · Running Time → P3 with a clever check · Median → P2 on a partition index
</details>

---

## 14. Self-test

<details><summary>1. State the template's invariant, and explain why `ok` is never called at `lo` or `hi`.</summary>

`ok(lo)` is false and `ok(hi)` is true. While `hi − lo ≥ 2`, `mid = lo + (hi − lo)/2` lies strictly between them, and `ok` is only ever called on `mid`.
</details>

<details><summary>2. You want the last index with a[i] ≤ x. Which predicate, and which end do you return?</summary>

The predicate `a[i] > x` (F…F T…T) with sentinels −1 and n. Return `lo`, the last false, which is −1 if none.
</details>

<details><summary>3. Why does Find Peak Element work even though its predicate isn't monotone?</summary>

The template maintains "false at `lo`, true at `hi`", and any adjacent (false, true) pair is a peak. Monotonicity is only needed to make the boundary unique.
</details>

<details><summary>4. Why must "minimum capacity" check functions use long sums in some problems?</summary>

The check sums up to n values of up to 10⁹ (or counts of hours), which can exceed 2.1·10⁹.
</details>

<details><summary>5. Give an input showing that a rotated array with duplicates can't always be searched in O(log n).</summary>

`[1,1,1,1,0,1,1]`, searching for 0: every probe sees 1 except at the single 0, which could be anywhere, so an algorithm must inspect the positions one by one in the worst case.
</details>

---

## 15. Exit gate

Move on when you can:

* write the template from memory and prove its invariant, termination and sentinel safety;
* derive lower bound, upper bound, rotated minimum and peak finding from the template, without looking anything up;
* run the five-step method (§6) on a new "minimum X such that" problem, *including* the monotonicity sentence;
* solve 8 of the 10 practice problems, each with one template and no off-by-one fixes after the first run.
