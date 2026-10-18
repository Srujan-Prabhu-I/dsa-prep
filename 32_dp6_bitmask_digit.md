# Chapter 32 — DP VI: Bitmask & Digit DP

> **Part VI · Week 28** · **S-tier** · Prerequisites: Ch 6 (backtracking over subsets), Ch 17 (masks as sets), Ch 18 (counting), Ch 27
> **Problems:** 5 worked (1 ★) + 1 revisit · 8 practice · 3 stretch · **Time:** about 9 hours

Two DP families whose states look unusual at first, but follow the same method as every other DP:
* **Bitmask DP:** when n ≤ ~20 and the brute force tries every **ordering** or **assignment** (n!), the future often depends only on **which** items have been used, not on the order they were used in. So the state is a **set**, stored as a mask: 2ⁿ states instead of n!. At n = 15 that's 3·10⁴ instead of 1.3·10¹².
* **Digit DP:** to count the integers in `[L, R]` with some property, when R is up to 10¹⁸, build the numbers **digit by digit**, keeping only the state the property needs. The answer is f(R) − f(L − 1).

---

## 1. Attempt first ★

> **[Minimum XOR Sum of Two Arrays](https://leetcode.com/problems/minimum-xor-sum-of-two-arrays/) (30 minutes).** Two arrays of length n ≤ 14. Rearrange `nums2` to minimise `Σ (nums1[i] XOR nums2[i])`. Trying all 14! ≈ 8.7·10¹⁰ arrangements is too slow.

---

## 2. dp over subsets

### Worked B1 · [Minimum XOR Sum of Two Arrays](https://leetcode.com/problems/minimum-xor-sum-of-two-arrays/) ★

**What does the future depend on?** Assign `nums1[0], nums1[1], …` to partners **in index order**. After the first k have been assigned, what's left to decide is only *which* `nums2` elements are still free. The order in which the used ones were taken doesn't matter for the future, so the state is the **set** of used `nums2` indices:

> `best[mask]` = the minimum cost of pairing `nums1[0 .. |mask|)` with exactly the `nums2` indices in mask.
> **Last decision:** `nums1[|mask| − 1]` was paired with some j in mask: `best[mask] = min over j in mask of best[mask without j] + (nums1[|mask|−1] XOR nums2[j])`.

```java
public int minimumXORSum(int[] nums1, int[] nums2) {
    int n = nums1.length;
    int[] best = new int[1 << n];
    Arrays.fill(best, Integer.MAX_VALUE);
    best[0] = 0;
    for (int mask = 1; mask < (1 << n); mask++) {          // a smaller mask is always a smaller number: valid order
        int i = Integer.bitCount(mask) - 1;                // nums1[i] is the last one placed
        for (int j = 0; j < n; j++) {
            if ((mask >> j & 1) == 0) continue;            // its partner j must be in mask
            best[mask] = Math.min(best[mask], best[mask ^ (1 << j)] + (nums1[i] ^ nums2[j]));
        }
    }
    return best[(1 << n) - 1];
}
```

**O(n · 2ⁿ) ≈ 2.3·10⁵** for n = 14. **Evaluation order:** removing a bit always gives a numerically smaller mask, so increasing numeric order is a valid topological order (Ch 27 step 4). This is the **assignment problem** for small n. Parallel Courses II and Smallest Sufficient Team (practice and stretch) use the same "set of things done so far" state.

---

## 3. Sets plus a "last" element: travelling-salesman style

When the cost of the next step depends on the **previous** item (the distance from the last city, the overlap with the last word), the set alone isn't enough. Add the last item: **`dp[mask][last]`**, giving 2ⁿ · n states and n transitions each, **O(n² · 2ⁿ)**, which is fine for n ≤ ~16–18.

### Worked B2 · [Find the Shortest Superstring](https://leetcode.com/problems/find-the-shortest-superstring/)

> Given up to 12 words (none contained in another), return the shortest string containing every word as a substring.

**Reduction.** Writing the words in some order and overlapping each with the next as much as possible, the length is (sum of the lengths) − (sum of the overlaps between consecutive words). So **maximise the total overlap** over orderings. That's a path visiting every word once, a travelling-salesman path, with `overlap[i][j]` as the edge weight.

> `best[mask][last]` = the maximum total overlap of an ordering of the words in mask that ends with `last`. Reconstruct with `prev[mask][last]`.

```java
public String shortestSuperstring(String[] words) {
    int n = words.length;
    int[][] overlap = new int[n][n];                // the longest suffix of words[i] that's a prefix of words[j]
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++) {
            if (i == j) continue;
            for (int len = Math.min(words[i].length(), words[j].length()); len > 0; len--) {
                if (words[i].endsWith(words[j].substring(0, len))) { overlap[i][j] = len; break; }
            }
        }
    int full = (1 << n) - 1;
    int[][] best = new int[1 << n][n], prev = new int[1 << n][n];
    for (int[] row : best) Arrays.fill(row, -1);                 // -1 = unreachable
    for (int i = 0; i < n; i++) best[1 << i][i] = 0;
    for (int mask = 1; mask <= full; mask++) {
        for (int last = 0; last < n; last++) {
            if (best[mask][last] < 0) continue;
            for (int next = 0; next < n; next++) {
                if ((mask >> next & 1) == 1) continue;
                int nm = mask | (1 << next), value = best[mask][last] + overlap[last][next];
                if (value > best[nm][next]) {
                    best[nm][next] = value;
                    prev[nm][next] = last;
                }
            }
        }
    }
    int last = 0;
    for (int i = 1; i < n; i++) if (best[full][i] > best[full][last]) last = i;
    int[] order = new int[n];                                    // walk the prev pointers backwards
    for (int mask = full, k = n - 1; k >= 0; k--) {
        order[k] = last;
        int p = prev[mask][last];
        mask ^= 1 << last;
        last = p;
    }
    StringBuilder sb = new StringBuilder(words[order[0]]);
    for (int k = 1; k < n; k++) sb.append(words[order[k]].substring(overlap[order[k - 1]][order[k]]));
    return sb.toString();
}
```

O(n² · 2ⁿ) ≈ 6·10⁵ for n = 12. **Reconstruction through `prev`** is what turns the optimal value into the optimal ordering (Ch 27 bridge step 6). CSES Hamiltonian Flights (practice) is the counting version of the same state: the number of paths from city 1 to city n through every city.

---

## 4. Revisiting backtracking with a mask

### Worked B3 · [Partition to K Equal Sum Subsets (↺ Ch 6)](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/)

Ch 6 solved this (stretch) with pruned backtracking, which is exponential in the worst case. **With a mask, it's a clean DP.** Fill the buckets **one after another**. Once you know *which* numbers have been used, their total is known, and therefore so is how full the current bucket is: `total(mask) mod target`. So the mask alone is the state:

> `fill[mask]` = how full the current bucket is after placing the numbers in mask (−1 if no valid placement uses exactly that set).

```java
public boolean canPartitionKSubsets(int[] nums, int k) {
    int total = 0;
    for (int x : nums) total += x;
    if (total % k != 0) return false;
    int target = total / k, n = nums.length;
    int[] fill = new int[1 << n];
    Arrays.fill(fill, -1);
    fill[0] = 0;
    for (int mask = 0; mask < (1 << n); mask++) {
        if (fill[mask] < 0) continue;                            // unreachable
        for (int i = 0; i < n; i++) {
            if ((mask >> i & 1) == 1 || fill[mask] + nums[i] > target) continue;   // already used, or overflows the bucket
            fill[mask | (1 << i)] = (fill[mask] + nums[i]) % target;               // a full bucket resets to 0
        }
    }
    return fill[(1 << n) - 1] == 0;
}
```

O(n · 2ⁿ), with no dependence on k or on the order of the numbers. **The principle:** when backtracking's sub-searches depend only on *which* items remain (Ch 6 §5's "overlapping sub-searches"), the remaining set is a DP state.

**Iterating over submasks** (Ch 17 §0.5) handles "split the remaining set into a group plus the rest" transitions in O(3ⁿ) total. Fair Distribution of Cookies and Minimum Number of Work Sessions (practice) can use it.

---

## 5. Digit DP: counting numbers by building them digit by digit

**The problem shape:** count the integers in `[0, N]` (with N up to 10¹⁸) that satisfy a property defined digit by digit: no repeated digits, digit sum in a range, no two consecutive 1-bits, and so on. Then

> **count in `[L, R]` = f(R) − f(L − 1).**

**Building a number ≤ N, digit by digit from the left,** you need to remember:
* **`tight`:** whether the prefix built so far **equals** N's prefix. If it does, the next digit can't exceed N's next digit. If it's already smaller, every later digit is free (0–9).
* **`started`:** whether a non-zero digit has been placed yet, so that leading zeros don't count as digits (they matter for "distinct digits" and the like).
* **The property's own state:** the set of digits used, the running digit sum, the previous digit, …

The number of states is (positions ≤ 19) × (property states) × 2 × 2, which is tiny. Memoise on `(pos, property)` for the free, started case, which is the only case reached many times.

### Worked D1 · [Number of Digit One](https://leetcode.com/problems/number-of-digit-one/)

> Count the total number of digit **1**s appearing in all integers from 0 to n.

The simplest digit counting needs no DP at all. **Count per position**, which is the contribution idea from Ch 12 §4. At the position worth p (1, 10, 100, …), split n into `high` (the digits above), `cur` (this digit) and `low` (the digits below):
* every complete cycle of the higher part contributes **p** ones at this position: `high · p`;
* in the final, incomplete cycle: if cur > 1, the whole block of p ones appears; if cur == 1, only `low + 1` of them (the block is cut short at n).

```java
public int countDigitOne(int n) {
    long count = 0;
    for (long p = 1; p <= n; p *= 10) {          // the position worth p
        long high = n / (p * 10), cur = (n / p) % 10, low = n % p;
        count += high * p;                       // complete cycles of the higher digits
        if (cur > 1) count += p;                 // the current cycle passes this position's whole block of ones
        else if (cur == 1) count += low + 1;     // the block is cut short at n
    }
    return (int) count;
}
```

O(number of digits). `p` is a `long`, so `p * 10` can't overflow near the top of the `int` range.

### Worked D2 · [Numbers At Most N Given Digit Set](https://leetcode.com/problems/numbers-at-most-n-given-digit-set/)

> Using only the digits in `digitSet` (any number of times), how many **positive** integers ≤ n can you write?

This is the `tight` flag worked by hand. With k allowed digits:
* **shorter numbers** (fewer digits than n) are all smaller than n: kˡ of each length l;
* **same length:** walk along n's digits while staying tight. At each position, every allowed digit *smaller* than n's digit gives k^(remaining positions) numbers, all of them free afterwards. If n's digit itself is allowed, stay tight and continue; if not, stop. If you stay tight all the way to the end, n itself counts.

```java
public int atMostNGivenDigitSet(String[] digitSet, int n) {
    String s = String.valueOf(n);
    int len = s.length(), k = digitSet.length;
    long count = 0, power = 1;
    for (int l = 1; l < len; l++) {              // every shorter positive number
        power *= k;
        count += power;
    }
    for (int i = 0; i < len; i++) {              // same length: walk n's digits while tight
        int nd = s.charAt(i) - '0';
        long rest = 1;
        for (int j = i + 1; j < len; j++) rest *= k;     // free choices for the positions after i
        boolean canStayTight = false;
        for (String d : digitSet) {
            int dv = d.charAt(0) - '0';
            if (dv < nd) count += rest;          // smaller here: no longer tight, everything after is free
            else if (dv == nd) canStayTight = true;
        }
        if (!canStayTight) return (int) count;
    }
    return (int) (count + 1);                    // tight to the end: n itself can be written
}
```

### Worked D3 · [Count Special Integers](https://leetcode.com/problems/count-special-integers/)

> Count the integers in `[1, n]` whose digits are all **distinct**.

Now the property needs real state, **the set of digits used so far**, so write the general memoised digit DP:

```java
private char[] digits;
private int[][] memo;                            // memo[pos][mask], used only when free (not tight) and started

public int countSpecialNumbers(int n) {
    digits = String.valueOf(n).toCharArray();
    memo = new int[digits.length][1 << 10];
    for (int[] row : memo) Arrays.fill(row, -1);
    return count(0, 0, true, false);
}

// The number of ways to fill positions pos.. given: the digits used so far (mask), whether the prefix
// equals n's prefix (tight), and whether a non-zero digit has been placed yet (started).
private int count(int pos, int mask, boolean tight, boolean started) {
    if (pos == digits.length) return started ? 1 : 0;       // all leading zeros = the number 0, which doesn't count
    if (!tight && started && memo[pos][mask] != -1) return memo[pos][mask];
    int limit = tight ? digits[pos] - '0' : 9;
    int ways = 0;
    for (int d = 0; d <= limit; d++) {
        boolean nextTight = tight && d == limit;
        if (!started && d == 0) {
            ways += count(pos + 1, mask, nextTight, false);   // still a leading zero: no digit used
        } else if ((mask >> d & 1) == 0) {
            ways += count(pos + 1, mask | (1 << d), nextTight, true);
        }
    }
    if (!tight && started) memo[pos][mask] = ways;
    return ways;
}
```

**Why memoise only the free, started case:** the tight states lie on a single path (they track N's own prefix), and the not-yet-started states are just as few. In the free, started case, the count depends only on `(pos, mask)`. That's 10 positions × 1024 masks × 10 digits ≈ 10⁵ steps. **Change the property state to change the problem:** the digit sum gives Count of Integers (practice); the previous digit gives "no two adjacent equal digits"; a remainder gives "divisible by k".

---

## 6. Where these break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| n ≈ 25+ for 2ⁿ · n | 8·10⁸+ | Meet in the middle (Ch 37), or exploit structure |
| `dp[mask][last]` at n = 20 | 2²⁰ · 20 · 20 ≈ 4·10⁸ | Usually OK in C++, tight in Java: use `int[]` arrays and a cache-friendly loop order |
| Digit DP that forgets the `started` flag | Leading zeros counted as digits (e.g. "distinct digits" rejects 7 as 007) | Track `started` |
| Memoising tight states together with free ones | Wrong counts | Memoise only when not tight (or include `tight` in the key) |
| f(L − 1) when L = 0 | Calling with −1 | Handle L = 0 separately, or count from 1 |

---

## 7. Recognition

**Bitmask DP signals:** n ≤ 20 (sometimes ≤ 16 when there's a `last` dimension); "assign each … to exactly one …"; "visit every …"; "order everything / choose a permutation"; "split into groups"; a brute force over **permutations** whose future depends only on the **set** used.

**Digit DP signals:** "count the integers in `[L, R]`" with **R up to 10⁹–10¹⁸**, and a property of the **digits** (distinct, sum, adjacent constraints, how many times a digit appears, the binary representation).

**False friends:** n ≤ 20 with a simple sum objective is often just backtracking or meet in the middle. Counting numbers with an *arithmetic* property, like "multiple of 3 or 5", is usually inclusion–exclusion (Ch 18).

---

## 8. Implementation traps

| Trap | Fix |
| :--- | :--- |
| `1 << n` in loops when n = 31+ | It won't arise for n ≤ 20; use `1L << n` beyond 30 |
| Iterating masks in an order that reads unfinished states | Increasing numeric order, when transitions only *add* bits |
| `Integer.MAX_VALUE` + cost overflowing | Skip unreachable states, or use INF = 10⁹ |
| `Math.pow` for exact counts | Multiply in a loop with `long` |
| Reconstruction pointers not stored | Keep `prev[mask][last]` alongside the values |

---

## 9. Practice

1. **[Fair Distribution of Cookies](https://leetcode.com/problems/fair-distribution-of-cookies/)** *(Medium)*
2. **[CSES Hamiltonian Flights](https://cses.fi/problemset/task/1690)** *(Hard; stdin/stdout)*
3. **[Minimum Number of Work Sessions to Finish the Tasks](https://leetcode.com/problems/minimum-number-of-work-sessions-to-finish-the-tasks/)** *(Medium)*
4. **[CSES Elevator Rides](https://cses.fi/problemset/task/1653)** *(Hard; stdin/stdout)*
5. **[Smallest Sufficient Team](https://leetcode.com/problems/smallest-sufficient-team/)** *(Hard)*
6. **[Non-negative Integers without Consecutive Ones](https://leetcode.com/problems/non-negative-integers-without-consecutive-ones/)** *(Hard)*
7. **[CSES Counting Numbers](https://cses.fi/problemset/task/2220)** *(Hard; stdin/stdout)*
8. **[Count of Integers](https://leetcode.com/problems/count-of-integers/)** *(Hard)*

**Stretch:** **[Maximum Students Taking Exam](https://leetcode.com/problems/maximum-students-taking-exam/)** *(Hard)* · **[Parallel Courses II](https://leetcode.com/problems/parallel-courses-ii/)** *(Hard)* · **[Number of Ways to Wear Different Hats to Each Other](https://leetcode.com/problems/number-of-ways-to-wear-different-hats-to-each-other/)** *(Hard)*

<details><summary>Nudges</summary>

1. Cookies: `dp[k][mask]` = the best unfairness distributing the cookies in mask to k children; the last child gets some submask. Backtracking with pruning also passes (n ≤ 8).
2. Hamiltonian Flights: `ways[mask][last]` for paths starting at city 1, mod 10⁹ + 7. Only extend masks that contain city 1; n = 20 is tight in Java, so use int arrays and the bit loops of Ch 17.
3. Work Sessions: like Elevator Rides, with `best[mask] = (sessions, time used in the last session)`, compared lexicographically.
4. Elevator Rides: `best[mask] = (rides, weight in the current ride)`; add a person to the current ride if they fit, otherwise start a new one.
5. Sufficient Team: the state is the *skill* mask (≤ 16 skills); for each person, update `best[skills | personSkills]`. Reconstruct the team.
6. No Consecutive Ones: a digit DP over the binary digits, with the previous bit as state (the count follows the Fibonacci numbers).
7. Counting Numbers: a digit DP with state (previous digit, started); f(b) − f(a − 1); a = 0 needs care.
8. Count of Integers: a digit DP on strings with the digit sum as state (capped at max_sum); f(num2) − f(num1) + (num1 itself valid ? 1 : 0), mod 10⁹ + 7.
9. Stretch, Exam: a row-by-row DP over seat masks; a mask is valid if it has no adjacent seats and no broken seats; it conflicts with the previous row's mask on diagonals.
10. Stretch, Parallel Courses II: `dp[mask]` = the minimum semesters to finish mask; next semester, take a submask of size ≤ k of the courses whose prerequisites are all in mask.
11. Stretch, Hats: there are 40 hats but only ≤ 10 people. Iterate over hats, with a mask over **people**: `dp[hat][peopleMask]`.
</details>

<details><summary>Key</summary>

Cookies → submask DP or backtracking · Hamiltonian Flights → B2 counting · Work Sessions → B3-style (sessions, remaining time) · Elevator Rides → B3 (rides, last load) · Sufficient Team → a DP over skill masks + reconstruction · No Consecutive Ones → D2-style on binary digits · Counting Numbers → D3 (previous digit, started) · Count of Integers → D3 (digit sum), f(R) − f(L − 1) · Exam → row-mask DP · Parallel Courses II → B1 over the courses taken · Hats → iterate hats, mask over people
</details>

---

## 10. Self-test

<details><summary>1. Why can Minimum XOR Sum use a mask instead of a permutation as its state?</summary>

`nums1` is assigned in index order, so after k assignments, only *which* `nums2` elements are used affects the future, not the order in which they were taken.
</details>

<details><summary>2. When do you need dp[mask][last] instead of dp[mask]?</summary>

When the cost of the next step depends on the previous item (a distance, or an overlap from the last element), not just on which items have been used.
</details>

<details><summary>3. In Partition to K Equal Sum Subsets, why does the mask alone determine the current bucket's fill?</summary>

Buckets are filled one after another, so the used numbers' total is known from the mask, and the current bucket's fill is that total mod target.
</details>

<details><summary>4. What do the tight and started flags mean in digit DP?</summary>

`tight`: the prefix equals N's prefix, so the next digit is bounded by N's digit. `started`: a non-zero digit has been placed, so leading zeros don't count as digits.
</details>

<details><summary>5. Why are only the non-tight, started states memoised?</summary>

Those are the states reached many times, and their counts depend only on `(pos, property)`. Tight and unstarted states occur along very few paths.
</details>

---

## 11. Exit gate

Move on when you can:

* justify a mask state ("the future depends only on which items are used") and write `dp[mask]` and `dp[mask][last]` from blank files;
* reconstruct an optimal ordering from `prev` pointers;
* write the general digit DP with `tight` and `started`, and use f(R) − f(L − 1);
* derive the per-position digit-one formula;
* solve 6 of the 8 practice problems.
