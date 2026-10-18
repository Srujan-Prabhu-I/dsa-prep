# Chapter 8 — Prefix Sums & Prefix-State Counting

> **Part II · Weeks 5–6** · Prerequisites: Ch 1 (Derivation A), Ch 7
> **Problems:** 6 worked (1 ★) · 8 practice · 2 stretch · **Time:** about 7 hours

**Attempt first ★ (25 minutes):** *Subarray Sums Divisible by K.* Count the non-empty contiguous subarrays whose sum is divisible by `k`. `nums = [4, 5, 0, −2, −3, 1], k = 5` → 7. You already know Ch 1 Derivation A; see how far it takes you before reading §3.

---

This chapter is about one idea, used in five ways:

> **Precompute the answer for every prefix, then answer any question about a range by combining two prefixes.**

It removes the *recomputation* bottleneck from Ch 1 §2, where every range sum re-adds elements that an earlier range sum already added. And when you combine it with the hash map from Ch 7, it turns "count the subarrays with property X" from O(n²) into O(n). That second use is one of the most frequently tested ideas in interviews.

---

## 1. The core identity

Define the prefix array with **one more slot than the input**, using the half-open convention from Ch 5 §1:

> `P[0] = 0`, `P[i] = a[0] + a[1] + … + a[i − 1]`, so `P[i]` is the sum of `a[0, i)`.

Then the sum of any range `a[l, r)` is

> **sum(`a[l, r)`) = `P[r] − P[l]`**

because `P[r]` adds `a[0..r)`, and subtracting `P[l]` removes the part before `l`. One subtraction answers any range query in O(1), after an O(n) pass to build `P`.

```text
index i:        0    1    2    3    4    5
a[i]:           3    1    4    1    5
P[i]:      0    3    4    8    9   14            P[i+1] = P[i] + a[i]
           ↑ P[0]                    ↑ P[5]

sum of a[1, 4) = 1 + 4 + 1 = 6
             = P[4] − P[1] = 9 − 3 = 6            P[4] covers a[0..3], P[1] covers a[0]; the difference is a[1..3]
```

Each `P` value sits *between* elements: `P[i]` is "the total of everything to the left of position i". A range is the difference of the totals at its two ends.

**Why `n + 1` slots.** With `P[0] = 0`, the range starting at index 0 needs no special case: sum(`a[0, r)`) = `P[r] − P[0]`. Most prefix-sum bugs come from leaving out that leading zero.

**What makes it work.** Addition can be *undone* by subtraction. The same trick works for any operation that has an inverse:

| Operation | Combine two prefixes with | Works? |
| :--- | :--- | :-: |
| Sum | `P[r] − P[l]` | ✓ |
| Count of elements with some property | `C[r] − C[l]` | ✓ (it's a sum of 0s and 1s) |
| XOR | `X[r] ^ X[l]` (XOR undoes itself) | ✓ |
| Product (no zeros), modulo a prime | `P[r] × inverse(P[l])` | ✓ (Ch 18) |
| **Max / min** | — | **✗** |

Max has no inverse. On `[5, 1, 2]`, the prefix maxima are `5, 5, 5`, and nothing in them reveals that the maximum of `[1, 2]` is 2. Range max/min needs a different tool: a sparse table or a segment tree (Ch 34).

### Worked P1a · [Range Sum Query — Immutable](https://leetcode.com/problems/range-sum-query-immutable/)

> Given `nums`, answer many queries `sumRange(left, right)` (inclusive).

```java
class NumArray {
    private final long[] prefix;            // prefix[i] = nums[0] + ... + nums[i-1]

    public NumArray(int[] nums) {
        prefix = new long[nums.length + 1];
        for (int i = 0; i < nums.length; i++) prefix[i + 1] = prefix[i] + nums[i];
    }

    public int sumRange(int left, int right) {     // inclusive [left, right] = half-open [left, right + 1)
        return (int) (prefix[right + 1] - prefix[left]);
    }
}
```

O(n) to build and O(1) per query. `long` is the safe default for prefix sums: n values of up to 10⁹ overflow `int` quickly. (Here LeetCode's bounds happen to fit in `int`.) LeetCode gives the range as **inclusive** `[left, right]`, so convert it once, at the boundary, to half-open `[left, right + 1)`, and keep everything inside your code half-open.

### Worked P1b · [Range Sum Query 2D — Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/)

> Given a matrix, answer many queries for the sum of the rectangle with corners `(r1, c1)` and `(r2, c2)` (inclusive).

Let `P[r][c]` be the sum of the rectangle covering rows `[0, r)` and columns `[0, c)`. Two applications of **inclusion–exclusion** do everything:

**Building `P`:** the rectangle up to `(r + 1, c + 1)` is the rectangle above it, plus the rectangle to its left, minus their overlap (which was counted twice), plus the new cell.

**Querying:** the target rectangle is the big rectangle from the origin to the bottom-right corner, minus the strip above the target, minus the strip to its left, plus the top-left corner (which was subtracted twice).

```text
   ┌────────┬───────────┐
   │   A    │     B     │        target = (A + B + C + D) − (A + B) − (A + C) + A
   ├────────┼───────────┤               = P[r2+1][c2+1] − P[r1][c2+1] − P[r2+1][c1] + P[r1][c1]
   │   C    │  target D │
   └────────┴───────────┘
```

```java
class NumMatrix {
    private final int[][] prefix;    // prefix[r][c] = sum of rows [0, r) × columns [0, c)

    public NumMatrix(int[][] matrix) {
        int rows = matrix.length, cols = matrix[0].length;
        prefix = new int[rows + 1][cols + 1];
        for (int r = 0; r < rows; r++)
            for (int c = 0; c < cols; c++)
                prefix[r + 1][c + 1] = prefix[r][c + 1] + prefix[r + 1][c] - prefix[r][c] + matrix[r][c];
    }

    public int sumRegion(int r1, int c1, int r2, int c2) {
        return prefix[r2 + 1][c2 + 1] - prefix[r1][c2 + 1] - prefix[r2 + 1][c1] + prefix[r1][c1];
    }
}
```

The signs are the only hard part. Re-derive them from the picture each time rather than memorising the formula.

---

## 2. Prefix + hash map: counting subarrays by a property of two prefixes

Ch 1 Derivation A solved *Subarray Sum Equals K*. Here is that idea in its general form:

> A subarray `a[l, r)` corresponds to a **pair of prefixes** `(l, r)` with `l < r`. If the property you want depends only on `P[l]` and `P[r]`, then **fix `r` and ask how many earlier prefixes `l` match.** A hash map from prefix value to count (or to an index) answers that in O(1).

Everything in this section and the next two is an instance of that sentence. What changes is **what you store about each prefix** and **what "match" means**:

| Question | Match condition on prefixes | Store per prefix value |
| :--- | :--- | :--- |
| Count subarrays with sum k | `P[l] = P[r] − k` | how many times it occurred |
| Longest subarray with sum k | `P[l] = P[r] − k` | the **first** index it occurred at |
| Shortest subarray with sum k | `P[l] = P[r] − k` | the **last** index it occurred at |
| Count subarrays with sum divisible by k | `P[l] ≡ P[r] (mod k)` | count per remainder (§3) |
| Longest substring with all vowel counts even | parity states equal (§4) | first index per state |

### Worked P2 · [Contiguous Array](https://leetcode.com/problems/contiguous-array/)

> Given a binary array, return the length of the longest contiguous subarray with an equal number of 0s and 1s.

**Transform first.** Replace every 0 with −1. Then "equal numbers of 0s and 1s" means "**sum = 0**", which means two prefixes with **the same value**. To make the subarray *longest*, pair each `r` with the **earliest** `l` that has the same prefix value.

```java
public int findMaxLength(int[] nums) {
    Map<Integer, Integer> firstIndexOf = new HashMap<>();   // prefix value -> earliest i with P[i] == value
    firstIndexOf.put(0, 0);                                  // P[0] = 0
    int prefix = 0, best = 0;
    for (int i = 0; i < nums.length; i++) {
        prefix += (nums[i] == 1) ? 1 : -1;                   // prefix == P[i + 1]
        Integer first = firstIndexOf.putIfAbsent(prefix, i + 1);   // only the FIRST occurrence is kept
        if (first != null) best = Math.max(best, i + 1 - first);
    }
    return best;
}
```

`putIfAbsent` returns the existing value, or `null` after inserting. That one call does the "store only the first index" rule. Prefix values lie in `[−n, n]`, so an `int[2n + 1]` with an offset of n would replace the map and run faster.

**The reusable move here is the transform:** *rewrite the condition so that it becomes "a sum equals something".* Balanced counts become a zero sum; "more 1s than 0s" becomes a positive sum; "exactly k odd numbers" becomes a count of odd numbers equal to k. Nice Subarrays in the practice set is that last one.

---

## 3. Remainders: divisibility

### Worked P3 · [Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k/) ★

> Count the non-empty subarrays whose sum is divisible by `k`. (`n ≤ 3·10⁴`, `|nums[i]| ≤ 10⁴`, `2 ≤ k ≤ 10⁴`.)

**Derivation.** A subarray `a[l, r)` qualifies when `k` divides `P[r] − P[l]`. That is exactly when **`P[l]` and `P[r]` leave the same remainder mod k**:

> `k | (P[r] − P[l])` ⟺ `P[r] mod k = P[l] mod k`.

So the answer is the number of pairs `l < r` of prefixes with equal remainders. Fix `r` and add the number of earlier prefixes with the same remainder. There are only k possible remainders, so an `int[k]` replaces the hash map.

```java
public int subarraysDivByK(int[] nums, int k) {
    int[] countOfRemainder = new int[k];       // how many prefixes so far have each remainder
    countOfRemainder[0] = 1;                    // P[0] = 0
    int prefix = 0, count = 0;
    for (int x : nums) {
        prefix = Math.floorMod(prefix + x, k);  // keep only the remainder, always in [0, k)
        count += countOfRemainder[prefix];
        countOfRemainder[prefix]++;
    }
    return count;
}
```

Two details make or break this:
* **`Math.floorMod`, not `%`.** With negative numbers, `%` gives negative remainders (Ch 4 §10). Then −1 and 4 (mod 5), which are the *same* class, get counted separately, and `countOfRemainder[-1]` crashes anyway.
* **Storing only the remainder** keeps `prefix` small, so it can't overflow even when the true prefix sums are large.

The trace for `[4, 5, 0, −2, −3, 1], k = 5`: the prefix remainders are `0, 4, 4, 4, 2, 4, 0`. The remainder 4 occurs 4 times, giving C(4, 2) = 6 pairs, and the remainder 0 occurs twice, giving 1 pair. Total **7** ✓. That shows a second way to compute the same count: tally each remainder, then add c(c − 1)/2 for each.

---

## 4. Parity and XOR: when the "sum" is a set of flags

### Worked P4 · [Find the Longest Substring Containing Vowels in Even Counts](https://leetcode.com/problems/find-the-longest-substring-containing-vowels-in-even-counts/)

> Return the length of the longest substring in which each of `a, e, i, o, u` appears an even number of times.

**What property of a prefix decides this?** A substring `s[l, r)` has an even count of `a` exactly when the counts of `a` in the two prefixes `s[0, l)` and `s[0, r)` have the **same parity** (both even or both odd). So we need the five vowel parities to agree. The prefix "state" is **five yes/no flags**, one per vowel, and the substring qualifies when the two prefixes have **the same state**. It's the Contiguous Array idea again: "longest" means storing the first index of each state.

**Encoding five flags as one number.** Give vowel `v` the value 2ᵛ (a = 1, e = 2, i = 4, o = 8, u = 16). A state is the sum of the values of the vowels whose count is currently odd, a number in `[0, 32)`. Flipping vowel `v`'s flag means adding 2ᵛ if it was off and subtracting it if it was on. Java's **XOR** operator does that in one step: `state ^ (1 << v)` flips exactly that flag and leaves the others alone. Ch 17 covers XOR and bitmasks properly; this one use is all you need here.

```java
public int findTheLongestSubstring(String s) {
    int[] firstSeen = new int[32];            // state -> earliest prefix index with that state, or -1
    Arrays.fill(firstSeen, -1);
    firstSeen[0] = 0;                         // the empty prefix: every count is 0 (even)
    int state = 0, best = 0;
    for (int i = 0; i < s.length(); i++) {
        int v = "aeiou".indexOf(s.charAt(i));
        if (v >= 0) state ^= 1 << v;          // flip that vowel's parity flag
        if (firstSeen[state] < 0) firstSeen[state] = i + 1;
        else best = Math.max(best, i + 1 - firstSeen[state]);
    }
    return best;
}
```

**The general form, prefix XOR.** XOR is its own inverse (`x ^ x = 0`), so with `X[i]` = XOR of `a[0, i)`, the XOR of a range is `X[r] ^ X[l]`, the same as sums with subtraction. Count Triplets That Can Form Two Arrays of Equal XOR, in the practice set, is that identity plus pair counting.

---

## 5. Difference arrays: many range updates, read once at the end

The reverse problem: you're given many updates "**add v to every element of `a[l, r)`**" and must report the final array once at the end. Doing each update directly costs O(length), so O(n · q) in total.

**Idea: record only where each update starts and stops.** Keep `D` with `D[l] += v` and `D[r] −= v`. The final value at `i` is the **prefix sum** of `D` up to `i`. Every update with `l ≤ i` has added v, and every update that also has `r ≤ i` has taken it away again, so exactly the updates covering `i` remain. Each update is O(1), plus one O(n) pass at the end.

> A difference array is the **inverse** of a prefix sum: taking prefix sums of `D` gives back `a`. Updates that are hard on `a` (a whole range) are easy on `D` (two points).

### Worked P5 · [Corporate Flight Bookings](https://leetcode.com/problems/corporate-flight-bookings/)

> Flights are numbered 1…n. Each booking `[first, last, seats]` reserves `seats` on every flight from `first` to `last` inclusive. Return the total seats on each flight.

```java
public int[] corpFlightBookings(int[][] bookings, int n) {
    int[] diff = new int[n + 1];             // 0-indexed flights; diff[n] absorbs updates that end at the last flight
    for (int[] b : bookings) {
        diff[b[0] - 1] += b[2];              // inclusive [first, last] (1-indexed) = half-open [first-1, last)
        diff[b[1]] -= b[2];
    }
    int[] seats = new int[n];
    int running = 0;
    for (int i = 0; i < n; i++) {
        running += diff[i];                  // running = seats on flight i
        seats[i] = running;
    }
    return seats;
}
```

O(n + q). Converting 1-indexed inclusive input to 0-indexed half-open is the only place bugs live, so do it once, at the boundary, with a comment.

**In 2D,** the same idea marks four corners per rectangle update and then takes 2D prefix sums (Stamping the Grid in the stretch set). **As a sweep:** a difference array is "+v when an interval opens, −v when it closes". Ch 14 does exactly that over sorted events, without needing a full-length array.

**Online, while you sweep.** Sometimes the updates are *decided during* the left-to-right pass: at position i you decide to add v to the range `[i, i + len)`. A difference array still works, as long as every new update starts at or after the current position. Keep a running sum of what's active, and schedule the end:

```java
long[] diff = new long[n + 1];
long active = 0;                                // the total of the updates covering position i
for (int i = 0; i < n; i++) {
    active += diff[i];                          // updates that ended before i drop out here
    long value = base[i] + active;
    if (needsMore(value)) {                     // decide an update starting here
        long v = amountNeeded(value);
        active += v;                            // it covers i immediately…
        diff[Math.min(n, i + len)] -= v;        // …and stops covering at i + len
    }
}
```

This is what Minimum Number of K Consecutive Bit Flips (Mixed Set II) and greedy "cover each position as late as possible" checks need: O(1) per update instead of O(len).

---

## 6. Where prefix techniques break

| Situation | Why it breaks | Use instead |
| :--- | :--- | :--- |
| Point updates mixed with range queries | Every update changes O(n) prefix values | Fenwick tree / segment tree (Ch 34) |
| Range **max / min / gcd** | No inverse operation (§1) | Sparse table, segment tree (Ch 34) |
| Range updates *and* queries interleaved | The difference array only gives answers at the end | Fenwick on the difference array, or lazy segment tree (Ch 34) |
| All values positive, single target sum | Prefix + map works, but uses O(n) space | Sliding window, O(1) space (Ch 11) |
| "Shortest subarray with sum **≥** k", with negatives | The match is an inequality, not an equality, so a hash lookup can't find it | Prefix sums + monotonic deque (Ch 12) |

The last row is worth remembering. **Hashing finds prefixes *equal* to a target. It can't find prefixes that are merely *at most* or *at least* a target.** For inequalities you need order: sorting, binary search, or monotonic structures.

---

## 7. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| Count subarrays with sum in `[lo, hi]` | count(sum ≤ hi) − count(sum ≤ lo − 1). Each needs an order-aware structure: merge sort (Ch 9) or a Fenwick tree (Ch 34) |
| Count submatrices with sum k | Fix a pair of rows, collapse each column between them to one number, then run the 1D prefix-map count (stretch) |
| Largest sum of a subarray with length ≥ L | `P[r]` minus the minimum `P[l]` over `l ≤ r − L` (a running minimum) |
| A circular array | Duplicate the array, or reason about "total minus a middle part" |
| Product of every element except `a[i]` | Prefix products × suffix products (Ch 1 practice) |

---

## 8. Recognition

**Signals:**
* Many **range-sum** queries on a static array, or a sum over every subarray.
* "**Count** (or find the longest) **subarrays** whose sum / count / XOR / parity is exactly / divisible by …" → a pair of prefixes plus a hash map.
* The condition can be **rewritten** as a sum condition (0 → −1, odd → 1).
* Many **range updates**, with results read once at the end → difference array.

**False friends:**
* "Subarray with sum ≥ k" or "≤ k": an *inequality* (§6). With positives only, use a window (Ch 11); with negatives, a deque (Ch 12).
* "Maximum subarray sum": a prefix-based solution exists, but Kadane (Ch 1 practice, Ch 27) is simpler.

---

## 9. Implementation traps

| Trap | Fix |
| :--- | :--- |
| No `P[0] = 0`, so subarrays starting at index 0 are missed | Size `n + 1`, or seed the map with `{0 → 1}` / `{0 → index 0}` |
| Inserting the current prefix before querying | Query first (Ch 1 A) |
| Storing the latest index when you need the longest subarray | Longest → the first index; shortest → the last |
| `%` with negatives | `Math.floorMod` |
| Prefix sums overflow `int` | `long` |
| 2D inclusion–exclusion signs | Re-derive from the A/B/C/D picture |
| Off-by-one when converting 1-indexed inclusive input | Convert once, at the boundary, to 0-indexed half-open |

---

## 10. Practice

1. **[Matrix Block Sum](https://leetcode.com/problems/matrix-block-sum/)** *(Medium)*
2. **[Binary Subarrays With Sum](https://leetcode.com/problems/binary-subarrays-with-sum/)** *(Medium)*
3. **[Car Pooling](https://leetcode.com/problems/car-pooling/)** *(Medium)*
4. **[Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays/)** *(Medium)*
5. **[Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/)** *(Medium)*
6. **[Count Triplets That Can Form Two Arrays of Equal XOR](https://leetcode.com/problems/count-triplets-that-can-form-two-arrays-of-equal-xor/)** *(Medium)*
7. **[Make Sum Divisible by P](https://leetcode.com/problems/make-sum-divisible-by-p/)** *(Medium)*
8. **[Number of Wonderful Substrings](https://leetcode.com/problems/number-of-wonderful-substrings/)** *(Medium)*

**Stretch:** **[Number of Submatrices That Sum to Target](https://leetcode.com/problems/number-of-submatrices-that-sum-to-target/)** *(Hard)* · **[Stamping the Grid](https://leetcode.com/problems/stamping-the-grid/)** *(Hard)*

<details><summary>Nudges</summary>

1. Block Sum: each answer cell is a rectangle sum; clamp the rectangle to the grid.
2. Binary Subarrays: exactly the §2 table's first row.
3. Car Pooling: a difference array over positions; check that the running sum never exceeds capacity.
4. Nice Subarrays: make each element 1 if it's odd and 0 if it's even. Now it's "sum = k".
5. Continuous Subarray Sum: equal remainders, but the length must be at least 2, so store the *first index* of each remainder, not a count.
6. Equal XOR: `a == b` ⟺ the XOR of `arr[i..k]` is 0 ⟺ `X[i] == X[k + 1]`. Each such pair contributes `k − i` valid choices of j.
7. Divisible by P: let `need = total mod p`. You want the shortest subarray with sum ≡ need (mod p): store the *latest* index of each prefix remainder.
8. Wonderful: a 10-letter parity state. A substring qualifies if its two states are equal **or differ in exactly one flag**, so try all 10 single flips for each prefix. Count with `long`.
9. Stretch, Submatrices: fix the top and bottom rows, compute the column sums between them, and count 1D subarrays with sum = target. O(rows² · cols).
10. Stretch, Stamping: use 2D prefix sums to test "is this whole stamp-sized rectangle empty?", then a 2D difference array to mark every stamp you place, then check that every empty cell is covered.
</details>

<details><summary>Key</summary>

Block Sum → P1 (2D) · Binary Subarrays → P2 · Car Pooling → P5 · Nice Subarrays → P2 after the odd → 1 transform · Continuous Subarray Sum → P3 with the earliest index · Equal XOR → P4 prefix XOR + pair counting · Divisible by P → P3 with the latest index · Wonderful → P4, equal or one flag apart · Submatrices → P1 rows + P2 columns · Stamping → 2D P1 + 2D P5
</details>

---

## 11. Self-test

<details><summary>1. Why does the prefix array have n + 1 entries?</summary>

`P[0] = 0` represents the empty prefix, so ranges starting at index 0 follow the same formula `P[r] − P[0]`, and "the empty prefix" can be a matching partner (as in Derivation A).
</details>

<details><summary>2. Why can't prefix maxima answer range-maximum queries?</summary>

Max has no inverse. Knowing the maximum of `a[0, r)` and of `a[0, l)` tells you nothing about the maximum of `a[l, r)` when the overall maximum lies before `l`.
</details>

<details><summary>3. For "longest subarray with sum k", should you store the first or the last index of each prefix value? For "shortest"?</summary>

Longest: the first, so the left end is as far left as possible. Shortest: the last.
</details>

<details><summary>4. Why must the remainder be computed with floorMod in Subarray Sums Divisible by K?</summary>

With negative sums, `%` returns negative remainders, so two prefixes in the same residue class (like −1 and 4 mod 5) would be counted as different, and a negative index would crash.
</details>

<details><summary>5. You need the shortest subarray with sum ≥ k, and the array contains negatives. Why doesn't prefix + hash map work?</summary>

The hash map finds prefixes *equal* to a value, but here you need the latest prefix that is *at most* `P[r] − k`. That's an inequality, which needs an order-aware structure: the monotonic deque of Ch 12.
</details>

---

## 12. Exit gate

Move on when you can:

* derive the 1D and 2D prefix identities (including the inclusion–exclusion signs) from scratch;
* state the "pair of prefixes" principle and fill in the §2 table for a new condition;
* write Contiguous Array, Subarray Sums Divisible by K, the vowel-parity substring and Corporate Flight Bookings from blank files, each in under 10 minutes;
* say when a prefix technique breaks and what replaces it (§6);
* solve 7 of the 8 practice problems.
