# The Problem Plan: a closed set, done deeply

> This is the **complete** problem list for the handbook. Nothing outside it is required, except contests (see §1.3) and company-specific practice in the 2–3 weeks before each interview.

---

## 1. How the set is built and why it is enough

### 1.1 Four kinds of problems

| Kind | Where | How many | What you do |
| :--- | :--- | :-: | :--- |
| **Worked** | Inside each chapter | ~220 | The chapter derives these step by step. Problems marked ★ are **attempt-first**: try them (timeboxed) *before* reading the section. For the rest, stop after the statement and spend 5 minutes on the brute force, its bottleneck and one candidate idea before reading on (curriculum §4.1). |
| **Practice** | End of each chapter | ~285 | Solve these yourself, from a blank file. They're listed **without** pattern labels, easiest first; the pattern key is hidden. |
| **Stretch** | End of each chapter | ~90 | Optional on the first pass. Do them on the second pass, or if the chapter is a weak area. |
| **Mixed Sets** | End of each Part | ~155 | Unlabeled problems drawn from *every chapter so far*. These are the only problems that train selection on a closed set. Never peek at the key until you've committed to an approach. |

Problems you solve **independently** come to roughly **475**: about 35 ★ problems, ~285 practice problems and ~155 mixed-set problems. With stretch it's roughly **565**. That's about half a typical grinder's count. The difference is that every problem gets the full treatment: protocol, proof, insight log, and re-solves of your misses.

### 1.2 Time budget (honest numbers)

| Track | Independent solves | Total hours (incl. reading, re-solves, contests) | At 2 h/day, 6 days/week |
| :--- | :-: | :-: | :-: |
| **Full** (all chapters, no stretch) | ~475 | ~500 h | ~9–10 months |
| **Core** (skip S-tier chapters 25, 32–37 and Mixed Set VII) | ~400 | ~420 h | ~8 months |
| **Compression** (placement in < 3 months; see curriculum §4.5) | ~250 | ~240 h | ~3 months at 3 h/day |

This leaves the larger share of your engineering time for backend work. If your interview is sooner than your track allows, use the compression track. Don't cut the re-solves; they are where the retention comes from.

### 1.3 Contests are part of the plan

From Part II onward, take **one LeetCode weekly contest per week** (live or virtual). That gives you about 4 unseen, unlabeled problems each week, around 150 over the plan. Log them like any other problem. If your contest solve rate stops improving in one area, go back to that chapter's stretch problems. Do that rather than looking for more problem lists.

### 1.4 Notation

* ★ = attempt before reading the section.
* ↺ = a problem from an earlier chapter, revisited with a new technique. It isn't counted twice.
* *(custom)* = a problem written for this book to isolate one idea. It ships with a brute-force reference so you can verify your solution with the stress harness (Ch 5).
* `CSES` = cses.fi problem set; everything else is LeetCode (all non-premium).
* `P1, P2…` = the chapter's patterns. Practice keys map each problem to a pattern.

### 1.5 Where did "Arrays" and "Strings" go?

They are *data types*, not techniques, so each array/string technique lives in the chapter of the property it exploits:

| Classic "arrays/strings" topic | Lives in |
| :--- | :--- |
| Kadane, best time to buy/sell, product except self, Boyer–Moore majority | Ch 1 (derivation moves), Ch 27 (Kadane as DP) |
| Handling fluency: traversal, in-place writes, rotation, spiral, matrix transforms, string building/parsing, `char` arithmetic | **Ch 5 · Arrays, Strings & Implementation** |
| Anagrams, frequency arrays, index-as-hash (first missing positive) | Ch 7 |
| Dutch national flag, partitioning | Ch 9 |
| Reversal, palindromes, two-pointer string scans | Ch 11 |
| Expand-around-center, palindromic DP | Ch 29 |
| KMP, Z, rolling hash | Ch 35 |

---

## 2. Part 0 — The Method

### Ch 01 · The Derivation Protocol  *(W 4 · P 6 · S 2)*

The patterns here are **derivation moves**, which later chapters turn into named techniques.

| # | Move | Worked |
| :-- | :--- | :--- |
| M1 | Fix one end, query the past | [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) ★ |
| M2 | Discard dominated candidates | [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) |
| M3 | Define the answer by its last element | [Longest Arithmetic Subsequence of Given Difference](https://leetcode.com/problems/longest-arithmetic-subsequence-of-given-difference/) |
| M4 | Turn "optimise" into "decide" | [Magnetic Force Between Two Balls](https://leetcode.com/problems/magnetic-force-between-two-balls/) |

**Practice:** [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) · [Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) · [Majority Element](https://leetcode.com/problems/majority-element/) · [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) · [Jump Game](https://leetcode.com/problems/jump-game/) · [Gas Station](https://leetcode.com/problems/gas-station/)
**Stretch:** [Next Permutation](https://leetcode.com/problems/next-permutation/) · [Candy](https://leetcode.com/problems/candy/)

<details><summary>Key</summary>

Buy/Sell → M1 (fix sell day, keep min of past) · Max Subarray → M3 (best ending here) · Majority → cancellation invariant (Ch 2) · Product Except Self → split around i (prefix × suffix) · Jump Game → M3 / reach invariant · Gas Station → prefix minimum + proof · Next Permutation → "what is the smallest change at the right end" · Candy → two one-sided constraints, two passes
</details>

### Ch 02 · Proofs and Counterexamples  *(W 6 · P 4 + 10 claims)*

| # | Tool | Worked |
| :-- | :--- | :--- |
| T1 | Loop invariant | [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) |
| T2 | Exchange argument | [Largest Number](https://leetcode.com/problems/largest-number/) · [Two City Scheduling](https://leetcode.com/problems/two-city-scheduling/) |
| T3 | Greedy stays ahead | [Jump Game II](https://leetcode.com/problems/jump-game-ii/) |
| T4 | Counterexample hunting (by hand + brute-force search) | [*(custom)* Greedy coin change on {1, 3, 4}](https://leetcode.com/problems/greedy-coin-change-on-1-3-4/) |
| T5 | Extremal / "moving" argument | *(custom)* Why the median minimises Σ\|x − aᵢ\| |

**Practice:** 10 prove-or-refute claims *(in chapter)* · [Assign Cookies](https://leetcode.com/problems/assign-cookies/) · [Boats to Save People](https://leetcode.com/problems/boats-to-save-people/) · [Minimum Deletions to Make Character Frequencies Unique](https://leetcode.com/problems/minimum-deletions-to-make-character-frequencies-unique/) · [Queue Reconstruction by Height](https://leetcode.com/problems/queue-reconstruction-by-height/)

<details><summary>Key</summary>

Assign Cookies → T2 · Boats → T2 (heaviest with lightest) · Min Deletions → T5 (process frequencies high to low) · Queue Reconstruction → T2 + insertion invariant (tallest first)
</details>

### Ch 03 · Complexity, Honestly  *(drills only)*

**Practice:** 20 code fragments to analyse on paper, then confirm with a doubling experiment *(in chapter)*. No LeetCode problems; every later chapter exercises this.

---

## 3. Part I — Foundations: Java, Arrays & Strings, Recursion

Every chapter that introduces a data structure opens with a **from-zero pattern (P0/I0/R0/T0/G0)**: what the structure is, how it works inside, the cost of each operation and why, and **you build it yourself** before using Java's version. From Ch 5 on, you never *rely* on a structure you couldn't implement. (Ch 4 introduces the Java collection APIs early, as tools, before their chapters build them.) See the coverage map in curriculum §3.1.

### Ch 04 · Java for DSA: Essentials, Semantics & Collections  *(W 3 + lab · P 5 · S 2)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| J0 | Language essentials: program shapes (LeetCode vs OA), fast input/output, types, arrays, strings, methods, generics | [CSES Weird Algorithm](https://cses.fi/problemset/task/1068) |
| J1 | Overflow, widening, exact arithmetic | [Reverse Integer](https://leetcode.com/problems/reverse-integer/) |
| J2 | Comparator contracts and tie-breaking + collection idioms | [Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words/) |
| — | Predict-the-output lab: 25 snippets covering overflow, `%`, boxing, `==`, comparators, strings, collections | *(custom, in chapter)* |

**Practice:** [Sqrt(x)](https://leetcode.com/problems/sqrtx/) · [Sort Characters By Frequency](https://leetcode.com/problems/sort-characters-by-frequency/) · [Pow(x, n)](https://leetcode.com/problems/powx-n/) · [String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/) · *(custom)* TreeMap multiset lab
**Stretch:** [Divide Two Integers](https://leetcode.com/problems/divide-two-integers/) · [Fraction to Recurring Decimal](https://leetcode.com/problems/fraction-to-recurring-decimal/)

<details><summary>Key</summary>

Sqrt → `mid * mid` overflow (J1) · Sort by Frequency → J2 · Pow → `-n` overflows at `Integer.MIN_VALUE` (J1) · atoi → J1 clamping · Multiset lab → J2 idioms · Divide → J1 edge `MIN_VALUE / -1` · Fraction → J1 + hashing remainders
</details>

### Ch 05 · Arrays, Strings & Implementation (+ Stress Testing)  *(W 6 · P 16 · S 1)*

This is the home for array, string and matrix **handling fluency**: traversal, building, parsing, in-place transforms, `char` arithmetic and `StringBuilder` discipline. The techniques that happen to run on arrays live in their property chapters (§1.5).

| # | Pattern | Worked |
| :-- | :--- | :--- |
| I0 | **From zero:** contiguous memory, O(1) indexing, why insert-in-middle is O(n), dynamic arrays and amortised growth, 2D arrays in Java | [*(custom)* Build a growable `IntList`, verified against `ArrayList`](https://leetcode.com/problems/build-a-growable-intlist-verified-against-arraylist/) |
| I1 | Building and parsing strings (tokens, `StringBuilder`, `char[]`, `c - 'a'`) | [Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/) |
| I2 | Layer/boundary traversal | [Spiral Matrix](https://leetcode.com/problems/spiral-matrix/) ★ |
| I3 | Index transforms (rotate, transpose, reflect) | [Rotate Image](https://leetcode.com/problems/rotate-image/) |
| I4 | In-place state encoding / reader–writer writes | [Game of Life](https://leetcode.com/problems/game-of-life/) |
| I5 | Stress testing a planted bug | [*(custom)* Planted-bug hunt](https://leetcode.com/problems/planted-bug-hunt/) |

**Practice:** [Pascal's Triangle](https://leetcode.com/problems/pascals-triangle/) · [Valid Anagram](https://leetcode.com/problems/valid-anagram/) · [Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/) · [String Compression](https://leetcode.com/problems/string-compression/) · [CSES Increasing Array](https://cses.fi/problemset/task/1094) · [CSES Permutations](https://cses.fi/problemset/task/1070) · [Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/) · [Valid Sudoku](https://leetcode.com/problems/valid-sudoku/) · [Diagonal Traverse](https://leetcode.com/problems/diagonal-traverse/) · [Zigzag Conversion](https://leetcode.com/problems/zigzag-conversion/) · [Robot Bounded In Circle](https://leetcode.com/problems/robot-bounded-in-circle/) · [Walking Robot Simulation](https://leetcode.com/problems/walking-robot-simulation/) · [CSES Number Spiral](https://cses.fi/problemset/task/1071) · [CSES Coin Piles](https://cses.fi/problemset/task/1754) · [Multiply Strings](https://leetcode.com/problems/multiply-strings/) · [Add Binary](https://leetcode.com/problems/add-binary/)
**Stretch:** [Text Justification](https://leetcode.com/problems/text-justification/)

<details><summary>Key</summary>

Pascal → 2D jagged building · Valid Anagram → `int[26]` with `c - 'a'` (I1) · Longest Common Prefix → column scan with early exit · String Compression → I4 reader/writer in place · CSES problems → stdin/stdout + fast I/O drill (Number Spiral and Coin Piles also need overflow care) · Set Matrix Zeroes → I4 (first row/column as markers) · Valid Sudoku → box-index transform (I3) · Diagonal Traverse → I2 / index sums · Zigzag → I3 (period arithmetic) · Robot Bounded → direction vectors + invariant after 4 loops · Walking Robot → direction vectors + hashed obstacles · Multiply Strings → digit-position arithmetic · Add Binary → right-to-left carry with `StringBuilder` · Text Justification → careful simulation
</details>

### Ch 06 · Recursion & Backtracking  *(W 9 · P 10 · S 3)*

It sits here because Sorting (merge sort), Linked Lists and every tree chapter depend on it.

| # | Pattern | Worked |
| :-- | :--- | :--- |
| R0 | **From zero:** base case + recursive case, the call stack, drawing the recursion tree, why naive recursion can blow up | [Fibonacci Number](https://leetcode.com/problems/fibonacci-number/) · [*(custom)* Tower of Hanoi](https://leetcode.com/problems/tower-of-hanoi/) |
| B1 | Recursion as induction (trust the contract) | [K-th Symbol in Grammar](https://leetcode.com/problems/k-th-symbol-in-grammar/) |
| B2 | Include/exclude (subsets) | [Subsets](https://leetcode.com/problems/subsets/) ★ |
| B3 | Permutations with duplicates | [Permutations II](https://leetcode.com/problems/permutations-ii/) |
| B4 | Combinations with constraints and duplicates | [Combination Sum II](https://leetcode.com/problems/combination-sum-ii/) |
| B5 | Constraint search with pruning | [N-Queens](https://leetcode.com/problems/n-queens/) |
| B6 | Grid search with undo | [Word Search](https://leetcode.com/problems/word-search/) |
| B7 | Generating valid structures | [Generate Parentheses](https://leetcode.com/problems/generate-parentheses/) |

**Practice:** [Find the Winner of the Circular Game](https://leetcode.com/problems/find-the-winner-of-the-circular-game/) · [Subsets II](https://leetcode.com/problems/subsets-ii/) · [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) · [Combination Sum](https://leetcode.com/problems/combination-sum/) · [Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/) · [Restore IP Addresses](https://leetcode.com/problems/restore-ip-addresses/) · [Different Ways to Add Parentheses](https://leetcode.com/problems/different-ways-to-add-parentheses/) · [Maximum Length of a Concatenated String with Unique Characters](https://leetcode.com/problems/maximum-length-of-a-concatenated-string-with-unique-characters/) · [Matchsticks to Square](https://leetcode.com/problems/matchsticks-to-square/) · [Sudoku Solver](https://leetcode.com/problems/sudoku-solver/)
**Stretch:** [Expression Add Operators](https://leetcode.com/problems/expression-add-operators/) · [Partition to K Equal Sum Subsets](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/) · [Remove Invalid Parentheses](https://leetcode.com/problems/remove-invalid-parentheses/)

<details><summary>Key</summary>

Circular Game → Josephus recurrence (B1) · Subsets II → B2 + skip rule · Phone → B2 product · Combination Sum → B4 reuse allowed · Palindrome Partitioning → B4 over cut points · IP → B5 with length bounds · Add Parentheses → B1 divide & conquer · Concatenated Unique → B2 with a 26-letter `boolean[]` (bitmask version in Ch 17) · Matchsticks → B5 (sort descending, skip equal sides) · Sudoku → B5 with `boolean` tables per row, column and box · Add Operators → B7 carrying the last term for `*` · K Equal Subsets → B5 pruning (bitmask DP in Ch 32) · Invalid Parentheses → B7 with min-removal bound
</details>

---

## 4. Part II — Linear Structures

### Ch 07 · Hashing: Remembering the Past  *(W 6 · P 7 · S 2)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| P0 | **From zero:** hash functions, buckets, chaining, load factor, resizing, why lookups are O(1) *expected*, `equals`/`hashCode` | [Design HashMap](https://leetcode.com/problems/design-hashmap/) *(build it with chaining)* |
| P1 | Complement lookup | [Two Sum](https://leetcode.com/problems/two-sum/) |
| P2 | Canonical signature → grouping | [Group Anagrams](https://leetcode.com/problems/group-anagrams/) |
| P3 | Set membership, start-of-run detection | [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) ★ |
| P4 | The array as its own hash table | [First Missing Positive](https://leetcode.com/problems/first-missing-positive/) |
| P5 | Count by a derived key | [Brick Wall](https://leetcode.com/problems/brick-wall/) |

**Practice:** [Contains Duplicate II](https://leetcode.com/problems/contains-duplicate-ii/) · [Isomorphic Strings](https://leetcode.com/problems/isomorphic-strings/) · [Find All Duplicates in an Array](https://leetcode.com/problems/find-all-duplicates-in-an-array/) · [Determine if Two Strings Are Close](https://leetcode.com/problems/determine-if-two-strings-are-close/) · [Count Number of Bad Pairs](https://leetcode.com/problems/count-number-of-bad-pairs/) · [4Sum II](https://leetcode.com/problems/4sum-ii/) · [Longest Palindrome by Concatenating Two Letter Words](https://leetcode.com/problems/longest-palindrome-by-concatenating-two-letter-words/)
**Stretch:** [Max Points on a Line](https://leetcode.com/problems/max-points-on-a-line/) · [Minimum Area Rectangle](https://leetcode.com/problems/minimum-area-rectangle/)

<details><summary>Key</summary>

Contains Dup II → P1 with last index · Isomorphic → P2 (bijection both ways) · All Duplicates → P4 (sign marking) · Close Strings → P2 (key set + sorted frequency multiset) · Bad Pairs → P5 (key `a[i] − i`, count the complement) · 4Sum II → P1 on pair sums · Two-Letter Words → P1 + reverse-pair counting · Max Points → P2 (reduced slope via gcd) · Min Area Rectangle → P3 (look up the opposite corners)
</details>

### Ch 08 · Prefix Sums & Prefix-State Counting  *(W 6 · P 8 · S 2)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| P1 | Static range sums, 1D and 2D | [Range Sum Query – Immutable](https://leetcode.com/problems/range-sum-query-immutable/) · [Range Sum Query 2D – Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/) |
| P2 | Prefix + hashmap: count / longest | [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) · [Contiguous Array](https://leetcode.com/problems/contiguous-array/) |
| P3 | Prefix modulo k | [Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k/) ★ |
| P4 | Prefix XOR / parity bitmask | [Find the Longest Substring Containing Vowels in Even Counts](https://leetcode.com/problems/find-the-longest-substring-containing-vowels-in-even-counts/) |
| P5 | Difference arrays (offline range updates) | [Corporate Flight Bookings](https://leetcode.com/problems/corporate-flight-bookings/) |

**Practice:** [Matrix Block Sum](https://leetcode.com/problems/matrix-block-sum/) · [Binary Subarrays With Sum](https://leetcode.com/problems/binary-subarrays-with-sum/) · [Car Pooling](https://leetcode.com/problems/car-pooling/) · [Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays/) · [Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/) · [Count Triplets That Can Form Two Arrays of Equal XOR](https://leetcode.com/problems/count-triplets-that-can-form-two-arrays-of-equal-xor/) · [Make Sum Divisible by P](https://leetcode.com/problems/make-sum-divisible-by-p/) · [Number of Wonderful Substrings](https://leetcode.com/problems/number-of-wonderful-substrings/)
**Stretch:** [Number of Submatrices That Sum to Target](https://leetcode.com/problems/number-of-submatrices-that-sum-to-target/) · [Stamping the Grid](https://leetcode.com/problems/stamping-the-grid/)

<details><summary>Key</summary>

Matrix Block Sum → P1 (2D) · Binary Subarrays → P2 · Car Pooling → P5 · Nice Subarrays → P2 (prefix count of odds) · Continuous Subarray Sum → P3 (earliest index per remainder) · Equal XOR → P4 (equal prefix XORs, count index pairs) · Make Sum Divisible by P → P3 (latest index per remainder) · Wonderful Substrings → P4 (mask equal or off by one bit) · Submatrices → P1 rows + P2 columns · Stamping → 2D P1 + 2D P5
</details>

### Ch 09 · Sorting as Structure  *(W 5 · P 7 · S 2)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| P0 | **From zero:** what sorting guarantees, stability, elementary sorts (insertion/selection with their invariants), why comparison sorting is Ω(n log n) | [Sort an Array *(implement insertion sort, then merge sort, then quicksort, and compare)*](https://leetcode.com/problems/sort-an-array-implement-insertion-sort-then-merge-sort-then-quicksort-and-compare/) |
| P1 | Merge sort as divide & conquer; counting while merging | [*(custom)* Count Inversions](https://leetcode.com/problems/count-inversions/) |
| P2 | Partitioning (Dutch national flag) | [Sort Colors](https://leetcode.com/problems/sort-colors/) ★ |
| P3 | Selection (quickselect) | [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) |
| P4 | Counting / bucket sort | [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) |
| P5 | Sort to create structure, then greedy/scan | [*(in practice)*](https://leetcode.com/problems/in-practice/) |
| P6 | Coordinate compression | [*(in practice)*](https://leetcode.com/problems/in-practice/) |

**Practice:** [Rank Transform of an Array](https://leetcode.com/problems/rank-transform-of-an-array/) · [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) · [H-Index](https://leetcode.com/problems/h-index/) · [Minimum Increment to Make Array Unique](https://leetcode.com/problems/minimum-increment-to-make-array-unique/) · [Array of Doubled Pairs](https://leetcode.com/problems/array-of-doubled-pairs/) · [Car Fleet](https://leetcode.com/problems/car-fleet/) · [Advantage Shuffle](https://leetcode.com/problems/advantage-shuffle/)
**Stretch:** [Reverse Pairs](https://leetcode.com/problems/reverse-pairs/) · [Maximum Gap](https://leetcode.com/problems/maximum-gap/)

<details><summary>Key</summary>

Rank Transform → P6 · K Closest → P3 · H-Index → P4 (counts capped at n) · Min Increment → P5 · Doubled Pairs → P5 (process by absolute value) · Car Fleet → P5 (sort by position, compare arrival times) · Advantage Shuffle → P5 + exchange argument · Reverse Pairs → P1 · Maximum Gap → P4 + pigeonhole proof
</details>

### Ch 10 · Binary Search: The Boundary Invariant  *(W 7 · P 10 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| P1 | The boundary template; lower/upper bound | [Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) |
| P2 | Predicates on index that aren't "value ≥ x" | [Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) · [Find Peak Element](https://leetcode.com/problems/find-peak-element/) |
| P3 | Search on the answer (monotone feasibility) | [Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) ★ |
| P4 | Search on value by counting | [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) · [Kth Smallest Element in a Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/) |
| P5 | Real-valued search; precision and limits | [*(custom)* Cube root to 10⁻⁹](https://leetcode.com/problems/cube-root-to-10/) |

**Practice:** [Search Insert Position](https://leetcode.com/problems/search-insert-position/) · [Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/) · [Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array/) · [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) · [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) · [Minimum Number of Days to Make m Bouquets](https://leetcode.com/problems/minimum-number-of-days-to-make-m-bouquets/) · [Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/) · [Minimum Speed to Arrive on Time](https://leetcode.com/problems/minimum-speed-to-arrive-on-time/) · [Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) · [Kth Smallest Number in Multiplication Table](https://leetcode.com/problems/kth-smallest-number-in-multiplication-table/)
**Stretch:** [Find K-th Smallest Pair Distance](https://leetcode.com/problems/find-k-th-smallest-pair-distance/) · [Maximum Running Time of N Computers](https://leetcode.com/problems/maximum-running-time-of-n-computers/) · [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)

<details><summary>Key</summary>

Insert Position → P1 · 2D Matrix → P1 on flattened index · Single Element → P2 (pair-parity predicate) · Rotated → P2 · Koko → P3 · Bouquets → P3 · Rotated II → where P2 breaks (duplicates force O(n) worst case) · Min Speed → P3 with exact arithmetic · Split Array → P3 + greedy check · Multiplication Table → P4 · Pair Distance → P4 + two-pointer count · Running Time → P3 with a clever check · Median → P2 on a partition index
</details>

### Ch 11 · Two Pointers & Sliding Windows  *(W 8 · P 10 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| P1 | Opposite-end elimination | [Two Sum II – Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) ★ · [Container With Most Water](https://leetcode.com/problems/container-with-most-water/) |
| P2 | Sort, fix one, two-pointer the rest | [3Sum](https://leetcode.com/problems/3sum/) |
| P3 | Reader/writer compaction | [Remove Duplicates from Sorted Array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/) |
| P4 | Window, longest valid (fixed or variable) | [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) |
| P5 | Window, shortest valid | [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) · [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) |
| P6 | Counting windows; atMost(k) − atMost(k−1) | [Subarrays with K Different Integers](https://leetcode.com/problems/subarrays-with-k-different-integers/) |

**Practice:** [Move Zeroes](https://leetcode.com/problems/move-zeroes/) · [Squares of a Sorted Array](https://leetcode.com/problems/squares-of-a-sorted-array/) · [3Sum Closest](https://leetcode.com/problems/3sum-closest/) · [Permutation in String](https://leetcode.com/problems/permutation-in-string/) · [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) · [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) · [Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/) · [Minimum Operations to Reduce X to Zero](https://leetcode.com/problems/minimum-operations-to-reduce-x-to-zero/) · [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) · [Count Subarrays Where Max Element Appears at Least K Times](https://leetcode.com/problems/count-subarrays-where-max-element-appears-at-least-k-times/)
**Stretch:** [4Sum](https://leetcode.com/problems/4sum/) · [Substring with Concatenation of All Words](https://leetcode.com/problems/substring-with-concatenation-of-all-words/) · [Count Subarrays With Fixed Bounds](https://leetcode.com/problems/count-subarrays-with-fixed-bounds/)

<details><summary>Key</summary>

Move Zeroes → P3 · Squares → P1 (merge from both ends) · 3Sum Closest → P2 · Permutation in String → P4 fixed window · Max Ones III → P4 · Char Replacement → P4 (why max-frequency may go stale) · Product < K → P6 · Reduce X → complement: longest middle window (P4) · Rain Water → P1 with a two-sided max invariant · Max Element ≥ K Times → P6 · 4Sum → P2 generalised to k-sum · Concatenation → P4 with word-sized strides · Fixed Bounds → P6 tracking last positions
</details>

### Ch 12 · Stacks, Queues & Dominance  *(W 9 · P 10 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| P0 | **From zero:** stack (LIFO) and queue (FIFO); array-backed stack; circular-buffer queue; deque; why `ArrayDeque` beats `Stack`/`LinkedList` | [*(custom)* Array-backed stack](https://leetcode.com/problems/array-backed-stack/) · [Design Circular Queue](https://leetcode.com/problems/design-circular-queue/) |
| P1 | Matching and parsing | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) · [Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/) |
| P2 | Next greater / smaller | [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) · [Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/) |
| P3 | Area via nearest-smaller boundaries | [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) ★ |
| P4 | Contribution via boundaries (with duplicates) | [Sum of Subarray Minimums](https://leetcode.com/problems/sum-of-subarray-minimums/) |
| P5 | Monotonic deque | [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) |
| P6 | Greedy stack for lexicographic order | [Remove K Digits](https://leetcode.com/problems/remove-k-digits/) |

**Practice:** [Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/) · [Min Stack](https://leetcode.com/problems/min-stack/) · [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) · [Asteroid Collision](https://leetcode.com/problems/asteroid-collision/) · [Online Stock Span](https://leetcode.com/problems/online-stock-span/) · [Decode String](https://leetcode.com/problems/decode-string/) · [Sum of Subarray Ranges](https://leetcode.com/problems/sum-of-subarray-ranges/) · [132 Pattern](https://leetcode.com/problems/132-pattern/) · [Remove Duplicate Letters](https://leetcode.com/problems/remove-duplicate-letters/) · [Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/)
**Stretch:** [Shortest Subarray with Sum at Least K](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/) · [Basic Calculator](https://leetcode.com/problems/basic-calculator/) · [Longest Valid Parentheses](https://leetcode.com/problems/longest-valid-parentheses/)

<details><summary>Key</summary>

Queue via Stacks → P0, amortised transfer · Min Stack → auxiliary stack of minimums · RPN → P1 · Asteroids → stack simulation · Stock Span → P2 (previous greater) · Decode String → P1 nested · Subarray Ranges → P4 for max and min · 132 → P2 scanning from the right · Remove Duplicate Letters → P6 + "last occurrence" guard · Maximal Rectangle → P3 per row · Shortest Subarray ≥ K → P5 over prefix sums · Basic Calculator → P1 with sign stack · Longest Valid → P1 with index stack
</details>

### Ch 13 · Linked Lists: Pointer Surgery  *(W 6 · P 8 · S 2)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| P0 | **From zero:** nodes and references, singly vs doubly linked, insert/delete at head/middle/tail, arrays vs lists (cost and cache behaviour) | [Design Linked List](https://leetcode.com/problems/design-linked-list/) |
| P1 | Reversal | [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) · [Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/) |
| P2 | Fast/slow pointers (with Floyd's proof) | [Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/) ★ |
| P3 | Dummy-node splicing and merging | [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) |
| P4 | Cloning with a map / interleaving | [Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/) |

**Practice:** [Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) · [Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists/) · [Partition List](https://leetcode.com/problems/partition-list/) · [Rotate List](https://leetcode.com/problems/rotate-list/) · [Add Two Numbers](https://leetcode.com/problems/add-two-numbers/) · [Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/) · [Reorder List](https://leetcode.com/problems/reorder-list/) · [Sort List](https://leetcode.com/problems/sort-list/)
**Stretch:** [Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/) · [Flatten a Multilevel Doubly Linked List](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/)

<details><summary>Key</summary>

Remove Nth → P2 gap pointers · Intersection → equal-length walk proof · Partition → P3 with two dummies · Rotate → length + re-link · Add Two Numbers → P3 with carry · Palindrome → P2 middle + P1 · Reorder → P2 + P1 + P3 · Sort List → merge sort with P2 split · Reverse II → P1 on a segment · Flatten → stack / recursion splicing
</details>

### Ch 14 · Intervals & Sweep Line  *(W 6 · P 7 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| P1 | Merge (sort by start) | [Merge Intervals](https://leetcode.com/problems/merge-intervals/) ★ |
| P2 | Max non-overlapping (sort by end) | [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) |
| P3 | Max overlap / min resources (event sweep) | [Divide Intervals Into Minimum Number of Groups](https://leetcode.com/problems/divide-intervals-into-minimum-number-of-groups/) |
| P4 | Two sorted lists, two pointers | [Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/) |
| P5 | Ordered-map bookings | [My Calendar I](https://leetcode.com/problems/my-calendar-i/) |
| P6 | Sweep with an active set (ordered multiset) | [The Skyline Problem](https://leetcode.com/problems/the-skyline-problem/) |

**Practice:** [Insert Interval](https://leetcode.com/problems/insert-interval/) · [Count Days Without Meetings](https://leetcode.com/problems/count-days-without-meetings/) · [Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) · [Remove Covered Intervals](https://leetcode.com/problems/remove-covered-intervals/) · [Video Stitching](https://leetcode.com/problems/video-stitching/) · [My Calendar II](https://leetcode.com/problems/my-calendar-ii/) · [Number of Flowers in Full Bloom](https://leetcode.com/problems/number-of-flowers-in-full-bloom/)
**Stretch:** [Data Stream as Disjoint Intervals](https://leetcode.com/problems/data-stream-as-disjoint-intervals/) · [Range Module](https://leetcode.com/problems/range-module/) · [Minimum Interval to Include Each Query](https://leetcode.com/problems/minimum-interval-to-include-each-query/)

<details><summary>Key</summary>

Insert → P1 linear · Days Without Meetings → P1 · Arrows → P2 · Covered → sort by (start ↑, end ↓) · Video Stitching → greedy cover, stays ahead · Calendar II → P3 / P5 · Flowers → P3 + binary search on sorted starts and ends · Disjoint Intervals → P5 merge on insert · Range Module → P5 split/merge · Min Interval per Query → offline sorted queries + an ordered set of active intervals
</details>

### Ch 15 · Heaps & Ordered Sets  *(W 7 · P 8 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| P0 | **From zero:** the heap property, array layout (children at 2i+1, 2i+2), sift-up/sift-down, O(n) heapify proof, heap sort; what a balanced BST is and why `TreeMap` is O(log n) | [*(custom)* Build a binary min-heap, verified against `PriorityQueue`](https://leetcode.com/problems/build-a-binary-min-heap-verified-against-priorityqueue/) |
| P1 | Bounded heap for top-k / k-th | [Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) ★ |
| P2 | k-way merge / frontier expansion | [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) · [Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/) |
| P3 | Two heaps | [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) |
| P4 | Lazy deletion / ordered multiset | [Sliding Window Median](https://leetcode.com/problems/sliding-window-median/) |
| P5 | Greedy scheduling by counts | [Task Scheduler](https://leetcode.com/problems/task-scheduler/) |

**Practice:** [Ugly Number II](https://leetcode.com/problems/ugly-number-ii/) · [Reorganize String](https://leetcode.com/problems/reorganize-string/) · [Single-Threaded CPU](https://leetcode.com/problems/single-threaded-cpu/) · [Process Tasks Using Servers](https://leetcode.com/problems/process-tasks-using-servers/) · [Furthest Building You Can Reach](https://leetcode.com/problems/furthest-building-you-can-reach/) · [Maximum Subsequence Score](https://leetcode.com/problems/maximum-subsequence-score/) · [Stock Price Fluctuation](https://leetcode.com/problems/stock-price-fluctuation/) · [Design a Number Container System](https://leetcode.com/problems/design-a-number-container-system/)
**Stretch:** [IPO](https://leetcode.com/problems/ipo/) · [Smallest Range Covering Elements from K Lists](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/) · [Meeting Rooms III](https://leetcode.com/problems/meeting-rooms-iii/)

<details><summary>Key</summary>

Ugly II → P2 · Reorganize → P5 · Single-Threaded CPU → heap-driven simulation · Servers → two heaps over time · Furthest Building → regret heap (keep ladders for the largest climbs) · Subsequence Score → sort + P1 · Stock Price → P4 · Number Container → map of TreeSets · IPO → sort + heap of affordable · Smallest Range → P2 with max tracking · Meeting Rooms III → two heaps simulation
</details>

### Ch 16 · Designing Data Structures  *(W 5 · P 6 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| D1 | Hash map + doubly linked list | [LRU Cache](https://leetcode.com/problems/lru-cache/) ★ |
| D2 | Hash map + array with swap-remove | [Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o-1/) |
| D3 | Frequency buckets | [LFU Cache](https://leetcode.com/problems/lfu-cache/) |
| D4 | Versioned values + binary search | [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) |
| D5 | Time windows with queues | [*(custom)* Sliding-Window Rate Limiter](https://leetcode.com/problems/sliding-window-rate-limiter/) |

**Practice:** [Design Browser History](https://leetcode.com/problems/design-browser-history/) · [Design Underground System](https://leetcode.com/problems/design-underground-system/) · [Snapshot Array](https://leetcode.com/problems/snapshot-array/) · [Flatten Nested List Iterator](https://leetcode.com/problems/flatten-nested-list-iterator/) · [Design Twitter](https://leetcode.com/problems/design-twitter/) · [Design a Food Rating System](https://leetcode.com/problems/design-a-food-rating-system/)
**Stretch:** [All O\`one Data Structure](https://leetcode.com/problems/all-o-one-data-structure/) · [Insert Delete GetRandom O(1) – Duplicates allowed](https://leetcode.com/problems/insert-delete-getrandom-o-1-duplicates-allowed/) · *(custom)* Token-Bucket Rate Limiter

<details><summary>Key</summary>

Browser History → array + pointer · Underground → two maps · Snapshot → D4 · Nested Iterator → stack of iterators · Twitter → k-way merge (Ch 15) · Food Ratings → map → TreeSet · AllOne → D3 · RandomizedCollection → D2 with index sets · Token Bucket → lazy refill arithmetic
</details>

---

### Mixed Set II — after Ch 16  *(30 problems, unlabeled)*

Mixed sets are ordered randomly on purpose. For each problem, write down your candidate techniques and why **before** you code.

1. Find Pivot Index
2. Maximum Width Ramp
3. Grumpy Bookstore Owner
4. Minimize the Maximum Difference of Pairs
5. K-diff Pairs in an Array
6. Rotate Array
7. Partition Labels
8. Frequency of the Most Frequent Element
9. Remove All Adjacent Duplicates in String II
10. Count Number of Teams
11. Online Election
12. Longest Mountain in Array
13. Maximum Sum of Distinct Subarrays With Length K
14. Find Right Interval
15. Majority Element II
16. Number of Subsequences That Satisfy the Given Sum Condition
17. Hand of Straights
18. Maximum Value at a Given Index in a Bounded Array
19. Find All Anagrams in a String
20. Minimum Moves to Equal Array Elements II
21. Maximum Frequency Stack
22. Minimum Swaps to Group All 1's Together II
23. Number of Visible People in a Queue
24. Range Frequency Queries
25. Longest Subarray of 1's After Deleting One Element
26. House Robber IV
27. Minimum Absolute Sum Difference
28. Maximum Erasure Value
29. Minimum Number of K Consecutive Bit Flips
30. Sum of Subsequence Widths

<details><summary>Key</summary>

1 prefix sums · 2 monotonic stack (decreasing candidates) + scan from right · 3 fixed window on "extra satisfied" · 4 binary search on answer + greedy adjacent pairing · 5 hashing · 6 triple reversal · 7 greedy on last occurrence · 8 sort + window · 9 stack of (char, count) · 10 fix the middle, count both sides · 11 precompute leaders + binary search on time · 12 single pass / up–down counts · 13 window + hash · 14 sort starts + binary search · 15 Boyer–Moore with two candidates · 16 sort + two pointers + powers of two · 17 TreeMap greedy from the smallest · 18 binary search on answer + arithmetic series · 19 fixed window frequency · 20 median (Ch 2 proof) · 21 frequency → stack map · 22 circular fixed window · 23 monotonic stack · 24 hash value → sorted index list + binary search · 25 window with ≤ 1 zero · 26 binary search on answer + greedy check · 27 sort copy + binary search for the best replacement · 28 window + set · 29 greedy + difference array · 30 sort + contribution of each element as max and min
</details>

---

## 5. Part III — Bits & Math

### Ch 17 · Bit Manipulation  *(W 6 · P 8 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| X1 | XOR cancellation | [Single Number](https://leetcode.com/problems/single-number/) ★ |
| X2 | Split by a distinguishing bit | [Single Number III](https://leetcode.com/problems/single-number-iii/) |
| X3 | Lowest-bit tricks | [Counting Bits](https://leetcode.com/problems/counting-bits/) |
| X4 | Per-bit contribution | [Total Hamming Distance](https://leetcode.com/problems/total-hamming-distance/) |
| X5 | Greedy from the high bit | [Maximum XOR of Two Numbers in an Array *(prefix-set version)*](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array-prefix-set-version/) |
| X6 | Common-prefix reasoning | [Bitwise AND of Numbers Range](https://leetcode.com/problems/bitwise-and-of-numbers-range/) |

**Practice:** [Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/) · [Missing Number](https://leetcode.com/problems/missing-number/) · [Reverse Bits](https://leetcode.com/problems/reverse-bits/) · [Single Number II](https://leetcode.com/problems/single-number-ii/) · [Minimum Flips to Make a OR b Equal to c](https://leetcode.com/problems/minimum-flips-to-make-a-or-b-equal-to-c/) · [Maximum Product of Word Lengths](https://leetcode.com/problems/maximum-product-of-word-lengths/) · [Gray Code](https://leetcode.com/problems/gray-code/) · [Largest Combination With Bitwise AND Greater Than Zero](https://leetcode.com/problems/largest-combination-with-bitwise-and-greater-than-zero/)
**Stretch:** [Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/) · [Decode XORed Permutation](https://leetcode.com/problems/decode-xored-permutation/) · [Bitwise ORs of Subarrays](https://leetcode.com/problems/bitwise-ors-of-subarrays/)

<details><summary>Key</summary>

1 Bits → X3 · Missing → X1 · Reverse Bits → unsigned shifts (`>>>`) · Single II → per-bit count mod 3 (X4) · Min Flips → per bit · Word Lengths → letter masks · Gray Code → `i ^ (i >> 1)` and why · Largest AND Combination → X4 (count per bit) · Sum of Two Integers → carry loop, Java sign · Decode XORed → X1 with the total XOR of 1..n · ORs of Subarrays → the set of ORs ending at i has ≤ 32 values
</details>

### Ch 18 · Math for Coding Rounds  *(W 9 · P 14 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| N1 | Sieve, smallest prime factor, factorisation | [Count Primes](https://leetcode.com/problems/count-primes/) ★ |
| N2 | GCD, LCM, Bézout | [Water and Jug Problem](https://leetcode.com/problems/water-and-jug-problem/) |
| N3 | Modular fast power | [Count Good Numbers](https://leetcode.com/problems/count-good-numbers/) · [Super Pow](https://leetcode.com/problems/super-pow/) |
| N4 | nCr with factorial / inverse tables | [Unique Paths *(closed form)*](https://leetcode.com/problems/unique-paths-closed-form/) · [*(custom)* 10⁵ nCr-mod-p queries](https://leetcode.com/problems/10-ncr-mod-p-queries/) |
| N5 | Inclusion–exclusion + binary search | [Nth Magical Number](https://leetcode.com/problems/nth-magical-number/) |
| N6 | Games: invariants and minimax | [Stone Game](https://leetcode.com/problems/stone-game/) |
| N7 | Linearity of expectation | [*(custom)* Expected number of records in a random permutation](https://leetcode.com/problems/expected-number-of-records-in-a-random-permutation/) |

**Practice:** [Excel Sheet Column Title](https://leetcode.com/problems/excel-sheet-column-title/) · [Greatest Common Divisor of Strings](https://leetcode.com/problems/greatest-common-divisor-of-strings/) · [Factorial Trailing Zeroes](https://leetcode.com/problems/factorial-trailing-zeroes/) · [Closest Prime Numbers in Range](https://leetcode.com/problems/closest-prime-numbers-in-range/) · [Distinct Prime Factors of Product of Array](https://leetcode.com/problems/distinct-prime-factors-of-product-of-array/) · [Random Pick with Weight](https://leetcode.com/problems/random-pick-with-weight/) · [Implement Rand10() Using Rand7()](https://leetcode.com/problems/implement-rand10-using-rand7/) · [Ugly Number III](https://leetcode.com/problems/ugly-number-iii/) · [Number of Ways to Reach a Position After Exactly k Steps](https://leetcode.com/problems/number-of-ways-to-reach-a-position-after-exactly-k-steps/) · [CSES Exponentiation II](https://cses.fi/problemset/task/1712) · [CSES Counting Divisors](https://cses.fi/problemset/task/1713) · [Count Anagrams](https://leetcode.com/problems/count-anagrams/) · [Shuffle an Array](https://leetcode.com/problems/shuffle-an-array/) · [Linked List Random Node](https://leetcode.com/problems/linked-list-random-node/)
**Stretch:** [Poor Pigs](https://leetcode.com/problems/poor-pigs/) · [Soup Servings](https://leetcode.com/problems/soup-servings/) · [Count the Number of Ideal Arrays](https://leetcode.com/problems/count-the-number-of-ideal-arrays/)

<details><summary>Key</summary>

Excel → bijective base 26 · GCD of Strings → N2 on lengths + period proof · Trailing Zeroes → count factors of 5 · Closest Primes → N1 · Distinct Prime Factors → N1 / trial division · Random Pick → prefix sums + binary search · Rand10 → rejection sampling, expected calls (N7) · Ugly III → N5 with lcm of 3 · Exactly k Steps → N4 · Exponentiation II → Fermat: reduce the exponent mod p − 1 · Counting Divisors → divisor sieve (harmonic) · Count Anagrams → multinomial with inverses · Shuffle an Array → Fisher–Yates · Linked List Random Node → reservoir sampling · Poor Pigs → counting states per pig · Soup Servings → probability DP + cutoff · Ideal Arrays → N1 + stars and bars
</details>

---

### Mixed Set III — after Ch 18  *(25 problems, unlabeled)*

1. Permutation Sequence
2. Shortest Unsorted Continuous Subarray
3. Combination Sum III
4. XOR Queries of a Subarray
5. Count Sorted Vowel Strings
6. Find Kth Bit in Nth Binary String
7. Longest Turbulent Subarray
8. Beautiful Arrangement
9. Four Divisors
10. Sum of All Subset XOR Totals
11. Find Original Array From Doubled Array
12. Splitting a String Into Descending Consecutive Values
13. Count Numbers with Unique Digits
14. Smallest Subarrays With Maximum Bitwise OR
15. Number of Ways to Split a String
16. The Number of Beautiful Subsets
17. Minimum Number of Operations to Make Array XOR Equal to K
18. Mirror Reflection
19. Minimum Array End
20. Maximum Score Words Formed by Letters
21. Kth Smallest Instructions
22. Number of Sets of K Non-Overlapping Line Segments
23. Minimum Number of Swaps to Make the String Balanced
24. Maximum Number of Removable Characters
25. Number of Substrings Containing All Three Characters

<details><summary>Key</summary>

1 factorial number system · 2 sort-compare or two scans with running max/min · 3 backtracking · 4 prefix XOR · 5 stars and bars · 6 recursion on halves · 7 single pass / window · 8 backtracking with pruning · 9 trial division · 10 per-bit contribution (OR of all × 2ⁿ⁻¹) · 11 sort + hashmap counts · 12 backtracking · 13 combinatorics by length · 14 per-bit last position, scanning right-to-left · 15 counting ones + combinatorics · 16 backtracking or grouping by residue · 17 XOR of all, then popcount of the difference · 18 lcm / unfolding · 19 fill the zero bits of x with the bits of n − 1 · 20 backtracking over subsets · 21 greedy + nCr · 22 stars and bars bijection · 23 greedy stack counting · 24 binary search on answer + subsequence check · 25 window with counts (count "at least one of each")
</details>

---

## 6. Part IV — Trees

### Ch 19 · Binary Trees: Information Flow  *(W 10 · P 11 · S 2)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| T0 | **From zero:** nodes, root/leaf/height/depth, the four traversals (pre/in/post/level), recursive and iterative; building a tree from LeetCode's array format | [*(custom)* Build a tree from a level-order array and print all four traversals](https://leetcode.com/problems/build-a-tree-from-a-level-order-array-and-print-all-four-traversals/) · [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) |
| T1 | Top-down parameters vs bottom-up returns | [Maximum Depth of Binary Tree *(both ways)*](https://leetcode.com/problems/maximum-depth-of-binary-tree-both-ways/) |
| T2 | Return one thing, record another globally | [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) · [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) ★ |
| T3 | Search and LCA | [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) |
| T4 | Construction and serialisation | [Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) · [Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) |
| T5 | Root-to-node path state | [Path Sum III](https://leetcode.com/problems/path-sum-iii/) |
| T6 | Level-order BFS | [Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/) |

**Practice:** [Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/) · [Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/) · [Binary Tree Inorder Traversal *(iterative)*](https://leetcode.com/problems/binary-tree-inorder-traversal-iterative/) · [Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/) · [Count Good Nodes in Binary Tree](https://leetcode.com/problems/count-good-nodes-in-binary-tree/) · [Path Sum II](https://leetcode.com/problems/path-sum-ii/) · [Flatten Binary Tree to Linked List](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/) · [Maximum Width of Binary Tree](https://leetcode.com/problems/maximum-width-of-binary-tree/) · [Distribute Coins in Binary Tree](https://leetcode.com/problems/distribute-coins-in-binary-tree/) · [All Nodes Distance K in Binary Tree](https://leetcode.com/problems/all-nodes-distance-k-in-binary-tree/) · [Vertical Order Traversal of a Binary Tree](https://leetcode.com/problems/vertical-order-traversal-of-a-binary-tree/)
**Stretch:** [Step-By-Step Directions From a Binary Tree Node to Another](https://leetcode.com/problems/step-by-step-directions-from-a-binary-tree-node-to-another/) · [Recover a Tree From Preorder Traversal](https://leetcode.com/problems/recover-a-tree-from-preorder-traversal/)

<details><summary>Key</summary>

Balanced → T2 (height or −1) · Subtree → T3 / serialisation · Iterative Inorder → explicit stack · Zigzag → T6 · Good Nodes → T1 top-down max · Path Sum II → T5 with backtracking · Flatten → reverse preorder or T2 · Max Width → T6 with indices (normalise per level to avoid overflow) · Distribute Coins → T2 (excess flows through each edge) · Distance K → tree to graph + BFS · Vertical Order → traversal + sort · Directions → T3 + path strings · Recover → stack by depth
</details>

### Ch 20 · BSTs & Tries  *(W 6 · P 9 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| S0 | **From zero:** the BST property, search/insert/delete, why an unbalanced BST degrades to O(n), what balancing (AVL / red-black) buys conceptually; what a trie node holds | [*(covered by S1–S5 below; no extra problem)*](https://leetcode.com/problems/covered-by-s1-s5-below-no-extra-problem/) |
| S1 | The BST invariant as interval bounds | [Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) ★ |
| S2 | Inorder = sorted (+ order-statistic follow-up) | [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) |
| S3 | Structural edits | [Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/) |
| S4 | BST-guided search | [Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) |
| S5 | Tries | [Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/) · [Word Search II](https://leetcode.com/problems/word-search-ii/) |
| S6 | Binary (XOR) trie | [Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) |

**Practice:** [Insert into a Binary Search Tree](https://leetcode.com/problems/insert-into-a-binary-search-tree/) · [Convert Sorted Array to Binary Search Tree](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/) · [Two Sum IV – Input is a BST](https://leetcode.com/problems/two-sum-iv-input-is-a-bst/) · [Binary Search Tree Iterator](https://leetcode.com/problems/binary-search-tree-iterator/) · [Construct Binary Search Tree from Preorder Traversal](https://leetcode.com/problems/construct-binary-search-tree-from-preorder-traversal/) · [Recover Binary Search Tree](https://leetcode.com/problems/recover-binary-search-tree/) · [Replace Words](https://leetcode.com/problems/replace-words/) · [Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/) · [Search Suggestions System](https://leetcode.com/problems/search-suggestions-system/)
**Stretch:** [Maximum XOR With an Element From Array](https://leetcode.com/problems/maximum-xor-with-an-element-from-array/) · [Concatenated Words](https://leetcode.com/problems/concatenated-words/) · [Palindrome Pairs](https://leetcode.com/problems/palindrome-pairs/)

<details><summary>Key</summary>

Insert → S4 · Sorted Array → middle as root (balance proof) · Two Sum IV → S2 + two pointers or hashing · BST Iterator → S2 with stack, amortised O(1) · From Preorder → S1 bounds · Recover → S2, find the two inversions · Replace Words → S5 · Add and Search → S5 + wildcard DFS · Suggestions → S5 or sort + binary search · Max XOR With Element → offline sort + S6 · Concatenated → S5 + DP · Palindrome Pairs → S5 on reversed words
</details>

---

### Mixed Set IV — after Ch 20  *(22 problems, unlabeled)*

1. Maximum Difference Between Node and Ancestor
2. Convert BST to Greater Tree
3. Find Duplicate Subtrees
4. Even Odd Tree
5. Delete Nodes And Return Forest
6. Pseudo-Palindromic Paths in a Binary Tree
7. Lowest Common Ancestor of Deepest Leaves
8. Balance a Binary Search Tree
9. Kth Largest Sum in a Binary Tree
10. Create Binary Tree From Descriptions
11. Populating Next Right Pointers in Each Node II
12. Smallest String Starting From Leaf
13. Count Nodes Equal to Average of Subtree
14. Validate Binary Tree Nodes
15. Longest Univalue Path
16. Cousins in Binary Tree II
17. Number of Good Leaf Nodes Pairs
18. Sum of Prefix Scores of Strings
19. Stream of Characters
20. Maximum Genetic Difference Query
21. Longest Subarray With Maximum Bitwise AND
22. Minimum Operations to Halve Array Sum

<details><summary>Key</summary>

1 top-down min/max · 2 reverse inorder · 3 serialisation + hashing · 4 level BFS · 5 postorder with a "parent deleted" flag · 6 path parity mask · 7 bottom-up (depth, node) · 8 inorder → rebuild · 9 level sums + heap/sort · 10 hashing child flags to find the root · 11 O(1) space level linking · 12 DFS with path string compare · 13 bottom-up (sum, count) · 14 in-degrees + one root + connectivity · 15 return-vs-record (Ch 19 T2) · 16 level sums · 17 bottom-up distance counts · 18 trie with prefix counts · 19 reversed trie · 20 offline DFS + XOR trie with deletions · 21 the AND only drops, so take the longest run of the max · 22 greedy max-heap
</details>

---

## 7. Part V — Graphs

### Ch 21 · Graph Modelling, BFS & DFS  *(W 9 · P 11 · S 4)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| G0 | **From zero:** vertices/edges, directed/undirected/weighted, degree, paths, cycles, connectivity; edge list vs adjacency list vs matrix (memory and cost); BFS and DFS from scratch, recursive and iterative | [*(custom)* Build all three representations from an edge list](https://leetcode.com/problems/build-all-three-representations-from-an-edge-list/) · [Find if Path Exists in Graph](https://leetcode.com/problems/find-if-path-exists-in-graph/) |
| G1 | Grid flood fill / components | [Number of Islands](https://leetcode.com/problems/number-of-islands/) ★ |
| G2 | Multi-source BFS | [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) |
| G3 | Implicit state graphs | [Open the Lock](https://leetcode.com/problems/open-the-lock/) · [Word Ladder](https://leetcode.com/problems/word-ladder/) |
| G4 | Graph copy | [Clone Graph](https://leetcode.com/problems/clone-graph/) |
| G5 | Two-colouring | [Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/) |
| G6 | BFS on augmented state | [Shortest Path in a Grid with Obstacles Elimination](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/) |

**Practice:** [Max Area of Island](https://leetcode.com/problems/max-area-of-island/) · [Number of Provinces](https://leetcode.com/problems/number-of-provinces/) · [Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) · [01 Matrix](https://leetcode.com/problems/01-matrix/) · [Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) · [Find Eventual Safe States](https://leetcode.com/problems/find-eventual-safe-states/) · [Shortest Bridge](https://leetcode.com/problems/shortest-bridge/) · [Snakes and Ladders](https://leetcode.com/problems/snakes-and-ladders/) · [Jump Game IV](https://leetcode.com/problems/jump-game-iv/) · [Detonate the Maximum Bombs](https://leetcode.com/problems/detonate-the-maximum-bombs/) · [Sliding Puzzle](https://leetcode.com/problems/sliding-puzzle/)
**Stretch:** [Bus Routes](https://leetcode.com/problems/bus-routes/) · [Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/) · [Making A Large Island](https://leetcode.com/problems/making-a-large-island/) · [Shortest Path to Get All Keys](https://leetcode.com/problems/shortest-path-to-get-all-keys/)

<details><summary>Key</summary>

Max Area → G1 · Provinces → G1 on a matrix · Surrounded → reverse: flood from the border · 01 Matrix → G2 · Pacific Atlantic → reverse flood from both oceans · Safe States → 3-colour DFS or reverse topological sort · Shortest Bridge → G1 then G2 · Snakes → G3 with index ↔ cell mapping · Jump Game IV → BFS with value buckets cleared after use · Bombs → directed reachability from each node · Sliding Puzzle → G3 on board strings · Bus Routes → BFS over routes, not stops · Visiting All Nodes → G6 (node, mask) · Large Island → component labels + neighbour sets · All Keys → G6 (cell, keys mask)
</details>

### Ch 22 · DAGs & Topological Order  *(W 6 · P 6 · S 2)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| D1 | Kahn's algorithm / cycle detection | [Course Schedule](https://leetcode.com/problems/course-schedule/) ★ · [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) |
| D2 | Order from pairwise comparisons | [*(custom)* Alien Alphabet *(LeetCode's "Alien Dictionary" is premium)*](https://leetcode.com/problems/alien-alphabet-leetcodes-alien-dictionary-is-premium/) |
| D3 | DP in topological order | [Parallel Courses III](https://leetcode.com/problems/parallel-courses-iii/) · [Longest Increasing Path in a Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/) |
| D4 | Leaf peeling | [Minimum Height Trees](https://leetcode.com/problems/minimum-height-trees/) |

**Practice:** [Minimum Number of Vertices to Reach All Nodes](https://leetcode.com/problems/minimum-number-of-vertices-to-reach-all-nodes/) · [Find All Possible Recipes from Given Supplies](https://leetcode.com/problems/find-all-possible-recipes-from-given-supplies/) · [All Ancestors of a Node in a Directed Acyclic Graph](https://leetcode.com/problems/all-ancestors-of-a-node-in-a-directed-acyclic-graph/) · [Course Schedule IV](https://leetcode.com/problems/course-schedule-iv/) · [Loud and Rich](https://leetcode.com/problems/loud-and-rich/) · [Build a Matrix With Conditions](https://leetcode.com/problems/build-a-matrix-with-conditions/)
**Stretch:** [Largest Color Value in a Directed Graph](https://leetcode.com/problems/largest-color-value-in-a-directed-graph/) · [Sort Items by Groups Respecting Dependencies](https://leetcode.com/problems/sort-items-by-groups-respecting-dependencies/)

<details><summary>Key</summary>

Min Vertices → in-degree zero proof · Recipes → D1 on a mixed graph · All Ancestors → D1 propagating sets · Course Schedule IV → transitive closure via D1 or Floyd · Loud and Rich → D3 · Matrix With Conditions → two D1 runs · Largest Color Value → D3 with 26 counters + cycle check · Sort Items → two-level D1
</details>

### Ch 23 · Shortest Paths  *(W 7 · P 8 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| SP1 | Dijkstra (proof + negative-edge counterexample) | [Network Delay Time](https://leetcode.com/problems/network-delay-time/) ★ |
| SP2 | Dijkstra with a non-additive path cost | [Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) |
| SP3 | 0-1 BFS | [Minimum Cost to Make at Least One Valid Path in a Grid](https://leetcode.com/problems/minimum-cost-to-make-at-least-one-valid-path-in-a-grid/) |
| SP4 | Dijkstra on augmented state | [CSES Flight Discount](https://cses.fi/problemset/task/1195) |
| SP5 | Bellman–Ford with an edge limit | [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) |
| SP6 | Counting shortest paths | [Number of Ways to Arrive at Destination](https://leetcode.com/problems/number-of-ways-to-arrive-at-destination/) |
| SP7 | All-pairs (Floyd–Warshall) | [Find the City With the Smallest Number of Neighbors at a Threshold Distance](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/) |

**Practice:** [CSES Shortest Routes I](https://cses.fi/problemset/task/1671) · [Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/) · [Minimum Obstacle Removal to Reach Corner](https://leetcode.com/problems/minimum-obstacle-removal-to-reach-corner/) · [Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) · [CSES Shortest Routes II](https://cses.fi/problemset/task/1672) · [CSES High Score](https://cses.fi/problemset/task/1673) · [Find Edges in Shortest Paths](https://leetcode.com/problems/find-edges-in-shortest-paths/) · [Minimum Cost to Reach Destination in Time](https://leetcode.com/problems/minimum-cost-to-reach-destination-in-time/)
**Stretch:** [Minimum Weighted Subgraph With the Required Paths](https://leetcode.com/problems/minimum-weighted-subgraph-with-the-required-paths/) · [Second Minimum Time to Reach Destination](https://leetcode.com/problems/second-minimum-time-to-reach-destination/) · [Minimum Time to Visit a Cell In a Grid](https://leetcode.com/problems/minimum-time-to-visit-a-cell-in-a-grid/)

<details><summary>Key</summary>

Routes I → SP1 with fast I/O and `long` distances · Max Probability → SP1 maximising a product · Obstacle Removal → SP3 · Swim → SP2 (or binary search + BFS, or DSU) · Routes II → SP7 · High Score → Bellman–Ford + reachability check of negative cycles · Edges in Shortest Paths → Dijkstra from both ends · Destination in Time → SP4 (node, time) · Weighted Subgraph → three Dijkstras, one on the reversed graph · Second Minimum → BFS keeping two distances · Visit a Cell → SP1 with waiting parity
</details>

### Ch 24 · Union-Find & Minimum Spanning Trees  *(W 4 · P 8 · S 4)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| U0 | **From zero:** derive DSU from naive relabelling → union by size → path compression; why it is "almost O(1)" | [*(covered by U1)*](https://leetcode.com/problems/covered-by-u1/) |
| U1 | DSU basics; cycle detection | [Redundant Connection](https://leetcode.com/problems/redundant-connection/) ★ |
| U2 | Union by a shared attribute | [Accounts Merge](https://leetcode.com/problems/accounts-merge/) |
| U3 | MST: cut property; Kruskal; Prim on dense graphs | [Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) |
| U4 | Offline sort + DSU | [Checking Existence of Edge Length Limited Paths](https://leetcode.com/problems/checking-existence-of-edge-length-limited-paths/) |

**Practice:** [Number of Operations to Make Network Connected](https://leetcode.com/problems/number-of-operations-to-make-network-connected/) · [Satisfiability of Equality Equations](https://leetcode.com/problems/satisfiability-of-equality-equations/) · [CSES Road Construction](https://cses.fi/problemset/task/1676) · [Most Stones Removed with Same Row or Column](https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/) · [Smallest String With Swaps](https://leetcode.com/problems/smallest-string-with-swaps/) · [Lexicographically Smallest Equivalent String](https://leetcode.com/problems/lexicographically-smallest-equivalent-string/) · [Evaluate Division](https://leetcode.com/problems/evaluate-division/) · [CSES Road Reparation](https://cses.fi/problemset/task/1675)
**Stretch:** [Remove Max Number of Edges to Keep Graph Fully Traversable](https://leetcode.com/problems/remove-max-number-of-edges-to-keep-graph-fully-traversable/) · [Number of Good Paths](https://leetcode.com/problems/number-of-good-paths/) · [Find Critical and Pseudo-Critical Edges in Minimum Spanning Tree](https://leetcode.com/problems/find-critical-and-pseudo-critical-edges-in-minimum-spanning-tree/) · [Largest Component Size by Common Factor](https://leetcode.com/problems/largest-component-size-by-common-factor/)

<details><summary>Key</summary>

Network Connected → count components; edges ≥ n − 1 · Equality Equations → U1 (unions first, then check ≠) · Road Construction → U1 with size and count metadata · Stones → U2 (rows and columns as nodes) · Swaps → U2 + sort each component · Equivalent String → U1 with min-representative · Evaluate Division → weighted DSU or BFS · Road Reparation → U3 Kruskal · Fully Traversable → two DSUs, shared edges first · Good Paths → U4 by value · Critical Edges → U3 with exclusion/forcing · Common Factor → U2 over prime factors
</details>

### Ch 25 · Connectivity & Euler Paths  *(W 6 · P 5 · S 2)* — S-tier

| # | Pattern | Worked |
| :-- | :--- | :--- |
| C1 | Bridges (low-link) | [Critical Connections in a Network](https://leetcode.com/problems/critical-connections-in-a-network/) ★ |
| C2 | Articulation points | [*(custom)* Critical Routers, verified against remove-and-BFS brute force](https://leetcode.com/problems/critical-routers-verified-against-remove-and-bfs-brute-force/) |
| C3 | SCCs and the condensation DAG | [CSES Planets and Kingdoms](https://cses.fi/problemset/task/1683) · [CSES Coin Collector](https://cses.fi/problemset/task/1686) |
| C4 | Euler paths (Hierholzer) | [Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/) |
| C5 | Functional graphs (out-degree 1) | [Longest Cycle in a Graph](https://leetcode.com/problems/longest-cycle-in-a-graph/) |

**Practice:** [CSES Flight Routes Check](https://cses.fi/problemset/task/1682) · [CSES Mail Delivery](https://cses.fi/problemset/task/1691) · [Count Visited Nodes in a Directed Graph](https://leetcode.com/problems/count-visited-nodes-in-a-directed-graph/) · [Valid Arrangement of Pairs](https://leetcode.com/problems/valid-arrangement-of-pairs/) · [Minimum Number of Days to Disconnect Island](https://leetcode.com/problems/minimum-number-of-days-to-disconnect-island/)
**Stretch:** [Cracking the Safe](https://leetcode.com/problems/cracking-the-safe/) · [Maximum Employees to Be Invited to a Meeting](https://leetcode.com/problems/maximum-employees-to-be-invited-to-a-meeting/)

<details><summary>Key</summary>

Flight Routes Check → reachability on G and reversed G · Mail Delivery → C4 circuit with degree check · Count Visited → C5 cycle lengths + tail DP · Valid Arrangement → C4 on a directed graph · Disconnect Island → answer ≤ 2; C2 or brute force by removing one cell · Cracking the Safe → C4 on a de Bruijn graph · Max Employees → C5 (2-cycles + longest chains vs the largest cycle)
</details>

---

### Mixed Set V — after Ch 25  *(25 problems, unlabeled)*

1. Number of Closed Islands
2. Reorder Routes to Make All Paths Lead to the City Zero
3. Minimize Hamming Distance After Swap Operations
4. Time Needed to Inform All Employees
5. Count Sub Islands
6. Shortest Path with Alternating Colors
7. Maximal Network Rank
8. Find the Safest Path in a Grid
9. Count Unreachable Pairs of Nodes in an Undirected Graph
10. Path With Maximum Gold
11. Minimum Score of a Path Between Two Cities
12. Regions Cut By Slashes
13. Number of Restricted Paths From First to Last Node
14. Coloring A Border
15. Maximum Total Importance of Roads
16. Count the Number of Complete Components
17. Escape the Spreading Fire
18. Trapping Rain Water II
19. Process Restricted Friend Requests
20. Minimum Moves to Move a Box to Their Target Location
21. Divide Nodes Into the Maximum Number of Groups
22. Cut Off Trees for Golf Event
23. Minimum Cost Walk in Weighted Graph
24. Maximum Number of Points From Grid Queries
25. Minimum Rounds to Complete All Tasks

<details><summary>Key</summary>

1 flood fill from the border, then count · 2 DFS with edge direction flags · 3 DSU + multiset compare per component · 4 tree DFS max path · 5 flood fill with a containment check · 6 BFS on (node, last colour) · 7 degrees + adjacency check · 8 multi-source BFS + binary search or max-min Dijkstra · 9 components + counting · 10 backtracking · 11 component min edge (DSU or BFS) · 12 upscale to 3×3 or DSU on triangles · 13 Dijkstra + DAG DP by distance · 14 flood fill + border rule · 15 greedy: degree sort · 16 components + edge count check · 17 binary search on wait + BFS (or two BFS) · 18 heap BFS from the boundary · 19 DSU with tentative unions · 20 BFS on (box, player) with an inner reachability BFS · 21 bipartite check + BFS eccentricity per component · 22 sort trees + repeated BFS · 23 DSU with component AND · 24 offline sorted queries + heap expansion · 25 counting + ceil(f/3), f = 1 impossible
</details>

---

## 8. Part VI — Greedy & Dynamic Programming

### Ch 26 · Greedy: Prove It or Kill It  *(W 5 + museum · P 8 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| G1 | Sort by deadline + regret heap | [Course Schedule III](https://leetcode.com/problems/course-schedule-iii/) ★ |
| G2 | Heap greedy sweeping over time | [Maximum Number of Events That Can Be Attended](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended/) |
| G3 | Deferred decisions (regret) | [Minimum Number of Refueling Stops](https://leetcode.com/problems/minimum-number-of-refueling-stops/) |
| G4 | Interval covering (stays ahead) | [Minimum Number of Taps to Open to Water a Garden](https://leetcode.com/problems/minimum-number-of-taps-to-open-to-water-a-garden/) |
| G5 | Where greedy ends and DP begins | [Wiggle Subsequence](https://leetcode.com/problems/wiggle-subsequence/) · [*(custom)* the Greedy Counterexample Museum (8 exhibits)](https://leetcode.com/problems/the-greedy-counterexample-museum-8-exhibits/) |

**Practice:** [Increasing Triplet Subsequence](https://leetcode.com/problems/increasing-triplet-subsequence/) · [Score After Flipping Matrix](https://leetcode.com/problems/score-after-flipping-matrix/) · [Bag of Tokens](https://leetcode.com/problems/bag-of-tokens/) · [Maximum Swap](https://leetcode.com/problems/maximum-swap/) · [Monotone Increasing Digits](https://leetcode.com/problems/monotone-increasing-digits/) · [Broken Calculator](https://leetcode.com/problems/broken-calculator/) · [Valid Parenthesis String](https://leetcode.com/problems/valid-parenthesis-string/) · [Eliminate Maximum Number of Monsters](https://leetcode.com/problems/eliminate-maximum-number-of-monsters/)
**Stretch:** [Patching Array](https://leetcode.com/problems/patching-array/) · [Minimum Cost to Hire K Workers](https://leetcode.com/problems/minimum-cost-to-hire-k-workers/) · [Put Marbles in Bags](https://leetcode.com/problems/put-marbles-in-bags/)

<details><summary>Key</summary>

Increasing Triplet → two best-so-far thresholds, stays ahead · Flipping Matrix → greedy by bit weight · Bag of Tokens → sort + two pointers, exchange · Max Swap → last occurrence of each digit · Monotone Digits → fix the first drop, then fill 9s · Broken Calculator → work backwards · Parenthesis String → range of open counts · Monsters → sort by arrival time · Patching → stays ahead on the reachable prefix · Hire K Workers → sort by ratio + heap · Marbles → contribution of each cut, sort pair sums
</details>

### Ch 27 · DP I: From Brute Recursion to Tables  *(W 6 · P 8 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| DP1 | The bridge: recursion → memo → table → rolling → reconstruct | [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/) · [House Robber](https://leetcode.com/problems/house-robber/) ★ |
| DP2 | Segmentations and decodings | [Decode Ways](https://leetcode.com/problems/decode-ways/) · [Word Break](https://leetcode.com/problems/word-break/) |
| DP3 | Several states per index | [Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/) |
| DP4 | Reduce to a known DP | [Delete and Earn](https://leetcode.com/problems/delete-and-earn/) |

**Practice:** [House Robber II](https://leetcode.com/problems/house-robber-ii/) · [Count Ways To Build Good Strings](https://leetcode.com/problems/count-ways-to-build-good-strings/) · [Solving Questions With Brainpower](https://leetcode.com/problems/solving-questions-with-brainpower/) · [Minimum Cost For Tickets](https://leetcode.com/problems/minimum-cost-for-tickets/) · [Perfect Squares](https://leetcode.com/problems/perfect-squares/) · [Partition Array for Maximum Sum](https://leetcode.com/problems/partition-array-for-maximum-sum/) · [Maximum Sum Circular Subarray](https://leetcode.com/problems/maximum-sum-circular-subarray/) · [Extra Characters in a String](https://leetcode.com/problems/extra-characters-in-a-string/)
**Stretch:** [Filling Bookcase Shelves](https://leetcode.com/problems/filling-bookcase-shelves/) · [Word Break II](https://leetcode.com/problems/word-break-ii/) · [Minimum Difficulty of a Job Schedule](https://leetcode.com/problems/minimum-difficulty-of-a-job-schedule/)

<details><summary>Key</summary>

Robber II → two runs of DP1 · Good Strings → DP1 counting · Brainpower → DP1 backwards · Tickets → DP1 over days with three choices · Perfect Squares → DP1 (unbounded choices) · Partition for Max Sum → DP2 over the last segment · Circular → Kadane on max and min (DP3) · Extra Characters → DP2 with a set or trie · Bookcase → DP2 over the last shelf · Word Break II → DP2 + reconstruction of all answers · Job Schedule → DP over (day, index), last segment max
</details>

### Ch 28 · DP II: Grids, Knapsack & Counting  *(W 7 · P 10 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| K1 | Grid DP | [Unique Paths II](https://leetcode.com/problems/unique-paths-ii/) ★ · [Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/) |
| K2 | Backward DP when forward fails | [Dungeon Game](https://leetcode.com/problems/dungeon-game/) |
| K3 | 0/1 knapsack | [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) |
| K4 | Unbounded knapsack | [Coin Change](https://leetcode.com/problems/coin-change/) |
| K5 | Combinations vs ordered sequences (loop order) | [Coin Change II](https://leetcode.com/problems/coin-change-ii/) · [Combination Sum IV](https://leetcode.com/problems/combination-sum-iv/) |

**Practice:** [Triangle](https://leetcode.com/problems/triangle/) · [Maximal Square](https://leetcode.com/problems/maximal-square/) · [Target Sum](https://leetcode.com/problems/target-sum/) · [Last Stone Weight II](https://leetcode.com/problems/last-stone-weight-ii/) · [Ones and Zeroes](https://leetcode.com/problems/ones-and-zeroes/) · [Number of Dice Rolls With Target Sum](https://leetcode.com/problems/number-of-dice-rolls-with-target-sum/) · [Domino and Tromino Tiling](https://leetcode.com/problems/domino-and-tromino-tiling/) · [Knight Probability in Chessboard](https://leetcode.com/problems/knight-probability-in-chessboard/) · [CSES Coin Combinations II](https://cses.fi/problemset/task/1636) · [CSES Book Shop](https://cses.fi/problemset/task/1158)
**Stretch:** [Cherry Pickup](https://leetcode.com/problems/cherry-pickup/) · [Profitable Schemes](https://leetcode.com/problems/profitable-schemes/) · [Number of Ways to Rearrange Sticks With K Sticks Visible](https://leetcode.com/problems/number-of-ways-to-rearrange-sticks-with-k-sticks-visible/)

<details><summary>Key</summary>

Triangle → K1 bottom-up · Maximal Square → K1 (min of three neighbours) · Target Sum → K3 counting with an offset or algebra · Last Stone II → K3 reformulation (closest to sum/2) · Ones and Zeroes → 2D K3 · Dice → bounded counting · Tromino → state counting with a recurrence · Knight Probability → probability DP · Coin Combinations II → K5 · Book Shop → K3 with tight memory/time in Java · Cherry Pickup → two walkers in lockstep · Profitable Schemes → K3 with a capped dimension · Sticks Visible → insert the smallest/largest last (counting)
</details>

### Ch 29 · DP III: Sequences & Strings  *(W 7 · P 10 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| Q1 | LIS, O(n²) → O(n log n) with proof | [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) ★ |
| Q2 | Two-sequence alignment | [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/) · [Edit Distance](https://leetcode.com/problems/edit-distance/) |
| Q3 | Palindromes | [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/) · [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/) |
| Q4 | Pattern matching | [Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/) |
| Q5 | Counting subsequences | [Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/) |

**Practice:** [Uncrossed Lines](https://leetcode.com/problems/uncrossed-lines/) · [Minimum ASCII Delete Sum for Two Strings](https://leetcode.com/problems/minimum-ascii-delete-sum-for-two-strings/) · [Longest String Chain](https://leetcode.com/problems/longest-string-chain/) · [Largest Divisible Subset](https://leetcode.com/problems/largest-divisible-subset/) · [Number of Longest Increasing Subsequence](https://leetcode.com/problems/number-of-longest-increasing-subsequence/) · [Minimum Insertion Steps to Make a String Palindrome](https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/) · [Interleaving String](https://leetcode.com/problems/interleaving-string/) · [Longest Ideal Subsequence](https://leetcode.com/problems/longest-ideal-subsequence/) · [Russian Doll Envelopes](https://leetcode.com/problems/russian-doll-envelopes/) · [Minimum Number of Removals to Make Mountain Array](https://leetcode.com/problems/minimum-number-of-removals-to-make-mountain-array/)
**Stretch:** [Wildcard Matching](https://leetcode.com/problems/wildcard-matching/) · [Palindrome Partitioning II](https://leetcode.com/problems/palindrome-partitioning-ii/) · [Shortest Common Supersequence](https://leetcode.com/problems/shortest-common-supersequence/)

<details><summary>Key</summary>

Uncrossed → Q2 (it's LCS) · ASCII Delete → Q2 weighted · String Chain → Q1-style over sorted lengths · Divisible Subset → Q1 + reconstruction · Number of LIS → Q1 with counts · Insertion Steps → Q3 (n − LPS) · Interleaving → Q2 on a prefix grid · Ideal Subsequence → DP over the last letter · Russian Doll → sort (w ↑, h ↓) + Q1 · Mountain → Q1 from both sides · Wildcard → Q4 · Partitioning II → Q3 table + cut DP · Supersequence → Q2 + reconstruction
</details>

### Ch 30 · DP IV: Intervals & State Machines  *(W 6 · P 7 · S 2)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| I1 | Interval DP with the "last action" trick | [Burst Balloons](https://leetcode.com/problems/burst-balloons/) ★ · [Minimum Cost to Cut a Stick](https://leetcode.com/problems/minimum-cost-to-cut-a-stick/) |
| I2 | Interval minimax games | [Predict the Winner](https://leetcode.com/problems/predict-the-winner/) |
| I3 | Interval DP with merges | [Strange Printer](https://leetcode.com/problems/strange-printer/) |
| A1 | Explicit state machines | [Best Time to Buy and Sell Stock with Cooldown](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/) · [Best Time to Buy and Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/) |

**Practice:** [Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/) · [Best Time to Buy and Sell Stock III](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/) · [Minimum Swaps To Make Sequences Increasing](https://leetcode.com/problems/minimum-swaps-to-make-sequences-increasing/) · [Minimum Score Triangulation of Polygon](https://leetcode.com/problems/minimum-score-triangulation-of-polygon/) · [Stone Game VII](https://leetcode.com/problems/stone-game-vii/) · [Guess Number Higher or Lower II](https://leetcode.com/problems/guess-number-higher-or-lower-ii/) · [Student Attendance Record II](https://leetcode.com/problems/student-attendance-record-ii/)
**Stretch:** [Minimum Cost to Merge Stones](https://leetcode.com/problems/minimum-cost-to-merge-stones/) · [Remove Boxes](https://leetcode.com/problems/remove-boxes/)

<details><summary>Key</summary>

Fee → A1 · Stock III → A1 with k = 2 · Min Swaps → A1 (swapped / not swapped at i) · Triangulation → I1 · Stone Game VII → I2 · Guess Number II → I2 minimax cost · Attendance II → A1 counting over (absences, trailing lates) · Merge Stones → I1 with a pile-count dimension · Remove Boxes → I3 with an extra "attached" count
</details>

### Ch 31 · DP V: Trees, DAGs & Rerooting  *(W 4 · P 7 · S 2)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| R1 | Include/exclude on trees | [House Robber III](https://leetcode.com/problems/house-robber-iii/) ★ |
| R2 | Multi-state tree DP | [Binary Tree Cameras](https://leetcode.com/problems/binary-tree-cameras/) |
| R3 | Diameter-style combination | [Longest Path With Different Adjacent Characters](https://leetcode.com/problems/longest-path-with-different-adjacent-characters/) |
| R4 | Rerooting | [Sum of Distances in Tree](https://leetcode.com/problems/sum-of-distances-in-tree/) |

**Practice:** [Number of Nodes in the Sub-Tree With the Same Label](https://leetcode.com/problems/number-of-nodes-in-the-sub-tree-with-the-same-label/) · [Minimum Fuel Cost to Report to the Capital](https://leetcode.com/problems/minimum-fuel-cost-to-report-to-the-capital/) · [Most Profitable Path in a Tree](https://leetcode.com/problems/most-profitable-path-in-a-tree/) · [CSES Tree Matching](https://cses.fi/problemset/task/1130) · [CSES Tree Distances I](https://cses.fi/problemset/task/1132) · [Maximum Sum BST in Binary Tree](https://leetcode.com/problems/maximum-sum-bst-in-binary-tree/) · [Count Number of Possible Root Nodes](https://leetcode.com/problems/count-number-of-possible-root-nodes/)
**Stretch:** [Minimize the Total Price of the Trips](https://leetcode.com/problems/minimize-the-total-price-of-the-trips/) · [Difference Between Maximum and Minimum Price Sum](https://leetcode.com/problems/difference-between-maximum-and-minimum-price-sum/)

<details><summary>Key</summary>

Same Label → subtree counts per letter · Fuel Cost → subtree sizes per edge · Profitable Path → Bob's arrival times + DFS · Tree Matching → R1 · Tree Distances I → R4 (or two BFS via diameter) · Max Sum BST → R2 returning (isBST, min, max, sum) · Possible Root Nodes → R4 counting correct guesses · Total Price → path counts + R1 · Price Sum → R4 with top-two children
</details>

### Ch 32 · DP VI: Bitmask & Digit DP  *(W 5 · P 8 · S 3)* — S-tier

| # | Pattern | Worked |
| :-- | :--- | :--- |
| B1 | dp[mask]: assignment | [Minimum XOR Sum of Two Arrays](https://leetcode.com/problems/minimum-xor-sum-of-two-arrays/) ★ |
| B2 | dp[mask][last] + reconstruction | [Find the Shortest Superstring](https://leetcode.com/problems/find-the-shortest-superstring/) |
| B3 | Subset partition over masks | [Partition to K Equal Sum Subsets](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/) |
| D1 | Digit counting by position | [Number of Digit One](https://leetcode.com/problems/number-of-digit-one/) |
| D2 | Digit DP (pos, tight) | [Numbers At Most N Given Digit Set](https://leetcode.com/problems/numbers-at-most-n-given-digit-set/) |
| D3 | Digit DP with carried state | [Count Special Integers](https://leetcode.com/problems/count-special-integers/) |

**Practice:** [Fair Distribution of Cookies](https://leetcode.com/problems/fair-distribution-of-cookies/) · [CSES Hamiltonian Flights](https://cses.fi/problemset/task/1690) · [Minimum Number of Work Sessions to Finish the Tasks](https://leetcode.com/problems/minimum-number-of-work-sessions-to-finish-the-tasks/) · [CSES Elevator Rides](https://cses.fi/problemset/task/1653) · [Smallest Sufficient Team](https://leetcode.com/problems/smallest-sufficient-team/) · [Non-negative Integers without Consecutive Ones](https://leetcode.com/problems/non-negative-integers-without-consecutive-ones/) · [CSES Counting Numbers](https://cses.fi/problemset/task/2220) · [Count of Integers](https://leetcode.com/problems/count-of-integers/)
**Stretch:** [Maximum Students Taking Exam](https://leetcode.com/problems/maximum-students-taking-exam/) · [Parallel Courses II](https://leetcode.com/problems/parallel-courses-ii/) · [Number of Ways to Wear Different Hats to Each Other](https://leetcode.com/problems/number-of-ways-to-wear-different-hats-to-each-other/)

<details><summary>Key</summary>

Cookies → submask DP or backtracking · Hamiltonian Flights → B2 counting (tight in Java) · Work Sessions → B3-style (sessions, remaining time) · Elevator Rides → B3 (rides, last load) · Sufficient Team → dp over skill masks + reconstruction · No Consecutive Ones → D2 on binary digits · Counting Numbers → D3 (previous digit, started) · Count of Integers → D3 (digit sum), f(R) − f(L − 1) · Exam → row-mask DP · Parallel Courses II → B1 over taken courses · Hats → iterate hats, mask over people
</details>

### Ch 33 · DP VII: Faster Transitions  *(W 5 · P 5 · S 2)* — S-tier

| # | Pattern | Worked |
| :-- | :--- | :--- |
| F1 | Binary search over sorted predecessors | [Maximum Profit in Job Scheduling](https://leetcode.com/problems/maximum-profit-in-job-scheduling/) ★ |
| F2 | Monotonic-deque window maximum | [Jump Game VI](https://leetcode.com/problems/jump-game-vi/) · [Constrained Subsequence Sum](https://leetcode.com/problems/constrained-subsequence-sum/) |
| F3 | Prefix-count transitions | [Jump Game VII](https://leetcode.com/problems/jump-game-vii/) |
| F4 | Prefix sums over the previous row | [K Inverse Pairs Array](https://leetcode.com/problems/k-inverse-pairs-array/) |

**Practice:** [Two Best Non-Overlapping Events](https://leetcode.com/problems/two-best-non-overlapping-events/) · [Maximum Number of Events That Can Be Attended II](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended-ii/) · [Maximum Sum of 3 Non-Overlapping Subarrays](https://leetcode.com/problems/maximum-sum-of-3-non-overlapping-subarrays/) · [Number of Ways to Form a Target String Given a Dictionary](https://leetcode.com/problems/number-of-ways-to-form-a-target-string-given-a-dictionary/) · [Max Sum of Rectangle No Larger Than K](https://leetcode.com/problems/max-sum-of-rectangle-no-larger-than-k/)
**Stretch:** [Delivering Boxes from Storage to Ports](https://leetcode.com/problems/delivering-boxes-from-storage-to-ports/) · [Number of Music Playlists](https://leetcode.com/problems/number-of-music-playlists/)

<details><summary>Key</summary>

Two Best Events → sort + suffix max + binary search · Events II → F1 with k · 3 Subarrays → best-left / best-right prefix arrays + reconstruction · Target String → precount letters per column, then DP · Rectangle ≤ K → fix the row pair + TreeSet of prefix sums · Delivering Boxes → F2 with prefix costs · Music Playlists → counting DP with a reuse constraint
</details>

---

### Mixed Set VI — after Ch 33  *(30 problems, unlabeled)*

1. Unique Binary Search Trees
2. Maximum Points You Can Obtain from Cards
3. Longest Arithmetic Subsequence
4. Knight Dialer
5. Minimum Falling Path Sum
6. Best Team With No Conflicts
7. Maximum Length of Repeated Subarray
8. Champagne Tower
9. Number of Ways to Paint N × 3 Grid
10. Out of Boundary Paths
11. Minimum Cost Tree From Leaf Values
12. Maximum Alternating Subsequence Sum
13. Count Vowels Permutation
14. Jump Game V
15. Last Day Where You Can Still Cross
16. Maximum Earnings From Taxi
17. Count All Valid Pickup and Delivery Options
18. Stone Game II
19. Frog Jump
20. Painting the Walls
21. Count Palindromic Subsequences
22. Arithmetic Slices II – Subsequence
23. Tallest Billboard
24. Allocate Mailboxes
25. Paint House III
26. Number of Ways to Stay in the Same Place After Some Steps
27. Freedom Trail
28. Maximum Value of K Coins From Piles
29. Minimum Total Distance Traveled
30. Minimum Number of Operations to Make Array Continuous

<details><summary>Key</summary>

1 Catalan / split DP · 2 complement fixed window · 3 dp[i][diff] with maps · 4 state DP over keys · 5 grid DP · 6 sort + LIS-style DP · 7 2D DP (or hash + binary search) · 8 row simulation DP · 9 pattern-state counting · 10 DP over moves · 11 interval DP, or greedy with a monotonic stack · 12 two-state DP · 13 automaton counting · 14 DP in sorted order / memo DFS · 15 binary search + BFS, or reverse-time DSU · 16 sort + binary search DP · 17 combinatorics or DP · 18 suffix-sum game DP · 19 dp[stone] set of jumps · 20 knapsack on "time covers walls" · 21 count middle + prefix/suffix pair counts (Hard) · 22 dp[i][diff] counting with `long` · 23 DP on height difference · 24 partition DP + median cost · 25 dp[i][colour][groups] · 26 DP with position capped at steps/2 · 27 DP over (key index, ring position) · 28 group knapsack · 29 sort both + DP over (robot, factory) · 30 sort + dedupe + window
</details>

---

## 9. Part VII — The Advanced Toolkit *(S-tier)*

### Ch 34 · Range Queries: Fenwick, Segment Tree, Sparse Table  *(W 6 · P 8 · S 3)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| Q1 | Fenwick: point update, prefix query | [Range Sum Query – Mutable](https://leetcode.com/problems/range-sum-query-mutable/) ★ |
| Q2 | Fenwick over compressed values | [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) |
| Q3 | Walking down a segment tree | [CSES Hotel Queries](https://cses.fi/problemset/task/1143) |
| Q4 | Lazy propagation + compression | [Falling Squares](https://leetcode.com/problems/falling-squares/) |
| Q5 | Segment tree as a DP accelerator | [Longest Increasing Subsequence II](https://leetcode.com/problems/longest-increasing-subsequence-ii/) |
| Q6 | Sparse table for idempotent queries | [CSES Static Range Minimum Queries](https://cses.fi/problemset/task/1647) |

**Practice:** [CSES Dynamic Range Sum Queries](https://cses.fi/problemset/task/1648) · [CSES Range Update Queries](https://cses.fi/problemset/task/1651) · [CSES List Removals](https://cses.fi/problemset/task/1749) · [My Calendar III](https://leetcode.com/problems/my-calendar-iii/) · [CSES Salary Queries](https://cses.fi/problemset/task/1144) · [CSES Distinct Values Queries](https://cses.fi/problemset/task/1734) · [Create Sorted Array through Instructions](https://leetcode.com/problems/create-sorted-array-through-instructions/) · [Count Good Triplets in an Array](https://leetcode.com/problems/count-good-triplets-in-an-array/)
**Stretch:** [Booking Concert Tickets in Groups](https://leetcode.com/problems/booking-concert-tickets-in-groups/) · [Handling Sum Queries After Update](https://leetcode.com/problems/handling-sum-queries-after-update/) · [Rectangle Area II](https://leetcode.com/problems/rectangle-area-ii/)

<details><summary>Key</summary>

Dynamic Range Sum → Q1 · Range Update → Fenwick on a difference array · List Removals → Q3 (k-th present element) · Calendar III → Q4 or sweep · Salary Queries → offline compression + Q1 · Distinct Values → offline by right endpoint + Q1 · Sorted Array Instructions → Q2 · Good Triplets → position mapping + Q2 from both sides · Concert Tickets → Q3 with max and sum · Sum Queries → Q4 with range flip · Rectangle Area II → sweep + coverage segment tree
</details>

### Ch 35 · String Algorithms  *(W 7 · P 7 · S 2)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| K1 | Prefix function / KMP | [Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) ★ |
| K2 | Borders and periods | [Repeated Substring Pattern](https://leetcode.com/problems/repeated-substring-pattern/) · [Longest Happy Prefix](https://leetcode.com/problems/longest-happy-prefix/) |
| K3 | Prefix function on a combined string | [Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/) |
| K4 | Z-function | [Sum of Scores of Built Strings](https://leetcode.com/problems/sum-of-scores-of-built-strings/) |
| K5 | Rolling hash (+ binary search on length) | [Repeated DNA Sequences](https://leetcode.com/problems/repeated-dna-sequences/) · [Longest Duplicate Substring](https://leetcode.com/problems/longest-duplicate-substring/) |

**Practice:** [Rotate String](https://leetcode.com/problems/rotate-string/) · [Repeated String Match](https://leetcode.com/problems/repeated-string-match/) · [CSES String Matching](https://cses.fi/problemset/task/1753) · [CSES Finding Borders](https://cses.fi/problemset/task/1732) · [CSES Finding Periods](https://cses.fi/problemset/task/1733) · [Minimum Time to Revert Word to Initial State II](https://leetcode.com/problems/minimum-time-to-revert-word-to-initial-state-ii/) · [Distinct Echo Substrings](https://leetcode.com/problems/distinct-echo-substrings/)
**Stretch:** [Longest Common Subpath](https://leetcode.com/problems/longest-common-subpath/) · [Count Prefix and Suffix Pairs II](https://leetcode.com/problems/count-prefix-and-suffix-pairs-ii/)

<details><summary>Key</summary>

Rotate → K1 on s + s · Repeated Match → K1 with a bounded repeat count · String Matching → K1 / K4 · Borders → K2 chain · Periods → K4 (or K2) · Revert Word → K4 at multiples of k · Echo → K5 or LCP table · Common Subpath → K5 + binary search across paths · Prefix-Suffix Pairs → K4 + trie of (prefix, suffix) pairs
</details>

### Ch 36 · Tree Techniques  *(W 5 · P 5)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| L1 | Binary lifting | [Kth Ancestor of a Tree Node](https://leetcode.com/problems/kth-ancestor-of-a-tree-node/) ★ |
| L2 | LCA by lifting | [CSES Company Queries II](https://cses.fi/problemset/task/1688) · [CSES Distance Queries](https://cses.fi/problemset/task/1135) |
| L3 | Path aggregates by lifting | [Minimum Edge Weight Equilibrium Queries in a Tree](https://leetcode.com/problems/minimum-edge-weight-equilibrium-queries-in-a-tree/) |
| L4 | Euler tour + Fenwick | [CSES Subtree Queries](https://cses.fi/problemset/task/1137) |

**Practice:** [CSES Company Queries I](https://cses.fi/problemset/task/1687) · [CSES Planets Queries I](https://cses.fi/problemset/task/1750) · [Cycle Length Queries in a Tree](https://leetcode.com/problems/cycle-length-queries-in-a-tree/) · [CSES Counting Paths](https://cses.fi/problemset/task/1136) · [CSES Path Queries](https://cses.fi/problemset/task/1138)

<details><summary>Key</summary>

Company I → L1 · Planets Queries I → L1 on a functional graph · Cycle Length → LCA in an implicit heap · Counting Paths → L2 + difference on tree · Path Queries → L4 with range update / point query
</details>

### Ch 37 · Synthesis Techniques & the Out-of-Scope Guide  *(W 4 · P 5 · S 2)*

| # | Pattern | Worked |
| :-- | :--- | :--- |
| Y1 | Contribution counting | [Total Appeal of A String](https://leetcode.com/problems/total-appeal-of-a-string/) ★ · [Count Unique Characters of All Substrings of a Given String](https://leetcode.com/problems/count-unique-characters-of-all-substrings-of-a-given-string/) |
| Y2 | Meet in the middle | [Closest Subsequence Sum](https://leetcode.com/problems/closest-subsequence-sum/) |
| Y3 | Reverse time | [Bricks Falling When Hit](https://leetcode.com/problems/bricks-falling-when-hit/) |
| — | Recognition briefs: max-flow/min-cut, HLD, centroid decomposition, suffix array/automaton, FFT, convex hull trick, Mo's algorithm | [*(in chapter; no problems)*](https://leetcode.com/problems/in-chapter-no-problems/) |

**Practice:** [Shortest Subarray to be Removed to Make Array Sorted](https://leetcode.com/problems/shortest-subarray-to-be-removed-to-make-array-sorted/) · [CSES Meet in the Middle](https://cses.fi/problemset/task/1628) · [Count Subarrays With Median K](https://leetcode.com/problems/count-subarrays-with-median-k/) · [Partition Array Into Two Arrays to Minimize Sum Difference](https://leetcode.com/problems/partition-array-into-two-arrays-to-minimize-sum-difference/) · [Sum of Total Strength of Wizards](https://leetcode.com/problems/sum-of-total-strength-of-wizards/)
**Stretch:** [Sum of Imbalance Numbers of All Subarrays](https://leetcode.com/problems/sum-of-imbalance-numbers-of-all-subarrays/) · [Maximum Sum Queries](https://leetcode.com/problems/maximum-sum-queries/)

<details><summary>Key</summary>

Remove to Sort → keep a prefix and a suffix, two pointers · Meet in the Middle → Y2 with sorted halves · Median K → balance prefix + hashing around k's index · Two Arrays Min Diff → Y2 grouped by count · Wizards → Y1 with a monotonic stack + prefix of prefix sums · Imbalance → Y1 per element · Max Sum Queries → offline sort + monotonic stack / segment tree
</details>

---

### Mixed Set VII — after Ch 37  *(22 problems, unlabeled, Hard-heavy)*

1. Smallest Range II
2. Making File Names Unique
3. Swap For Longest Repeated Character Substring
4. Sum of Distances
5. Find the Kth Smallest Sum of a Matrix With Sorted Rows
6. Maximum Number of Robots Within Budget
7. Design Graph With Shortest Path Calculator
8. Number of Atoms
9. Count of Range Sum
10. Number of Pairs Satisfying Inequality
11. Kth Smallest Product of Two Sorted Arrays
12. Number of Increasing Paths in a Grid
13. Shortest Distance After Road Addition Queries II
14. Brace Expansion II
15. Split Array With Same Average
16. Count Pairs Of Nodes
17. Count Increasing Quadruplets
18. Longest Substring of One Repeating Character
19. Find Building Where Alice and Bob Can Meet
20. Minimum Number of Visited Cells in a Grid
21. Maximum Sum of Subsequence With Non-adjacent Elements
22. Count the Number of Good Partitions

<details><summary>Key</summary>

1 sort + split point proof · 2 hashing with next suffix · 3 run lengths + counts · 4 group by value, prefix sums of indices · 5 heap merge row by row, or binary search + DFS count · 6 window + monotonic deque · 7 Dijkstra per query (design) · 8 stack + parsing · 9 prefix sums + merge-sort count or BIT · 10 rearrange inequality + merge-sort count or BIT · 11 binary search on answer + counting with signs · 12 DAG DP by value order · 13 ordered set of surviving nodes · 14 recursive parsing with set algebra · 15 meet in the middle or DP over (count, sum) · 16 degree sort + two pointers − shared-edge correction · 17 prefix count tables (O(n²)) · 18 segment tree with prefix/suffix runs · 19 offline + monotonic stack + binary search · 20 BFS with "next unvisited" sets or segment trees · 21 segment tree with 2×2 DP nodes · 22 last occurrences → number of blocks, power of 2
</details>

---

## 10. Part VIII — Performance

| Ch | Drill set |
| :-- | :--- |
| 38 · The Recognition Map | 40 problem statements *(custom)* to triage in 3 minutes each: constraints → budget → candidate set → strongest signal. You do **not** solve them; you're graded on whether the right technique is in your top two. |
| 39 · The OA Playbook | 6 timed mock OAs, each one unseen LeetCode virtual contest (90 min, 4 problems), followed by a post-mortem against the playbook's checklists. |
| 40 · The Live Interview Playbook | 8 recorded mock interviews. Problems are chosen by a partner, or picked from your contest backlog of unseen problems so you don't recognise them. Graded on the chapter's rubric. |

---

## 11. Totals

| Part | Worked | Practice | Stretch | Mixed |
| :--- | :-: | :-: | :-: | :-: |
| 0 · Method | 10 | 10 (+ 30 drills) | 2 | — |
| I · Foundations: Java, Arrays & Strings, Recursion | 18 | 31 (+ 25-item lab) | 6 | — |
| II · Linear | 65 | 81 | 26 | [30](https://leetcode.com/problems/30/) |
| III · Bits & Math | 15 | 22 | 6 | [25](https://leetcode.com/problems/25/) |
| IV · Trees | 16 | 20 | 5 | [22](https://leetcode.com/problems/22/) |
| V · Graphs | 32 | 38 | 15 | [25](https://leetcode.com/problems/25/) |
| VI · Greedy & DP | 45 | 63 | 21 | [30](https://leetcode.com/problems/30/) |
| VII · Advanced | 22 | 25 | 7 | [22](https://leetcode.com/problems/22/) |
| **Total** | **223** | **290** | **88** | [**154**](https://leetcode.com/problems/154/) |

Independent solves = ~36 ★ + 288 practice + 154 mixed ≈ **480**.

Plus ~150 contest problems spread over the plan, and the Part VIII drills.
