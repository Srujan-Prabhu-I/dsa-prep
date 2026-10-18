# Mixed Set III — after Chapter 18

> **Week 16** · 25 problems, unlabeled, drawn from Ch 1–18 · **Time:** about 10 hours

Same rules as Mixed Set II. Before coding each problem, write the restatement and budget, the brute-force bottleneck, your **top two candidate techniques with their structural reasons**, and the proof sentence. Perform mode: Medium 25 minutes, Hard 45 minutes. Score **solved** and **chose correctly**; the target is ≥ 70% on both.

About a fifth of this set reviews Part II. The rest leans on recursion, bits and math, but nothing tells you which is which.

---

## The set

1. [Permutation Sequence](https://leetcode.com/problems/permutation-sequence/)
2. [Shortest Unsorted Continuous Subarray](https://leetcode.com/problems/shortest-unsorted-continuous-subarray/)
3. [Combination Sum III](https://leetcode.com/problems/combination-sum-iii/)
4. [XOR Queries of a Subarray](https://leetcode.com/problems/xor-queries-of-a-subarray/)
5. [Count Sorted Vowel Strings](https://leetcode.com/problems/count-sorted-vowel-strings/)
6. [Find Kth Bit in Nth Binary String](https://leetcode.com/problems/find-kth-bit-in-nth-binary-string/)
7. [Longest Turbulent Subarray](https://leetcode.com/problems/longest-turbulent-subarray/)
8. [Beautiful Arrangement](https://leetcode.com/problems/beautiful-arrangement/)
9. [Four Divisors](https://leetcode.com/problems/four-divisors/)
10. [Sum of All Subset XOR Totals](https://leetcode.com/problems/sum-of-all-subset-xor-totals/)
11. [Find Original Array From Doubled Array](https://leetcode.com/problems/find-original-array-from-doubled-array/)
12. [Splitting a String Into Descending Consecutive Values](https://leetcode.com/problems/splitting-a-string-into-descending-consecutive-values/)
13. [Count Numbers with Unique Digits](https://leetcode.com/problems/count-numbers-with-unique-digits/)
14. [Smallest Subarrays With Maximum Bitwise OR](https://leetcode.com/problems/smallest-subarrays-with-maximum-bitwise-or/)
15. [Number of Ways to Split a String](https://leetcode.com/problems/number-of-ways-to-split-a-string/)
16. [The Number of Beautiful Subsets](https://leetcode.com/problems/the-number-of-beautiful-subsets/)
17. [Minimum Number of Operations to Make Array XOR Equal to K](https://leetcode.com/problems/minimum-number-of-operations-to-make-array-xor-equal-to-k/)
18. [Mirror Reflection](https://leetcode.com/problems/mirror-reflection/)
19. [Minimum Array End](https://leetcode.com/problems/minimum-array-end/)
20. [Maximum Score Words Formed by Letters](https://leetcode.com/problems/maximum-score-words-formed-by-letters/)
21. [Kth Smallest Instructions](https://leetcode.com/problems/kth-smallest-instructions/)
22. [Number of Sets of K Non-Overlapping Line Segments](https://leetcode.com/problems/number-of-sets-of-k-non-overlapping-line-segments/)
23. [Minimum Number of Swaps to Make the String Balanced](https://leetcode.com/problems/minimum-number-of-swaps-to-make-the-string-balanced/)
24. [Maximum Number of Removable Characters](https://leetcode.com/problems/maximum-number-of-removable-characters/)
25. [Number of Substrings Containing All Three Characters](https://leetcode.com/problems/number-of-substrings-containing-all-three-characters/)

---

<details><summary>Key: open only after attempting</summary>

| # | Technique | Chapter | The insight in one line |
| :-: | :--- | :-: | :--- |
| 1 | Factorial number system | 18 | The first digit is fixed by (k − 1) / (n − 1)!; recurse on the rest |
| 2 | Two scans with running max/min (or sort and compare) | 1, 9 | The right boundary is the last index below the running max from the left; mirror it for the left boundary |
| 3 | Backtracking with pruning | 6 | Choose from 1–9 in increasing order; prune when the sum exceeds n |
| 4 | Prefix XOR | 8 | xor(l..r) = X[r + 1] ^ X[l] |
| 5 | Stars and bars | 18 | Sorted strings of length n over 5 vowels = C(n + 4, 4) |
| 6 | Recursion on halves (the contract) | 6 | Sₙ = Sₙ₋₁ + "1" + reverse(invert(Sₙ₋₁)); recurse into the half that holds k |
| 7 | One pass with run lengths / window | 5, 11 | Extend while the comparison sign alternates; reset otherwise |
| 8 | Backtracking with pruning | 6 | Place positions from the end (fewer valid choices) using a `used` array |
| 9 | Trial division | 18 | Exactly four divisors means p·q (distinct primes) or p³ |
| 10 | Per-bit contribution | 17 | The answer is (OR of all) · 2ⁿ⁻¹ |
| 11 | Sort + hash counts | 7, 9 | Process from the smallest; each x must consume a 2x |
| 12 | Backtracking over split points | 6 | The first piece fixes the rest; parse in `long` (or `BigInteger` caution) |
| 13 | Combinatorics by length | 18 | 9 · 9 · 8 · … for each length, plus 1 for zero |
| 14 | Per-bit last positions, right to left | 17 | For each i, the reach is the furthest "next index containing bit b" over all bits b |
| 15 | Counting ones + combinatorics | 18 | With no ones, C(n − 1, 2); otherwise the product of the gap choices at the two cut points |
| 16 | Backtracking (or grouping by residue) | 6, 18 | Skip a number if x − k or x + k is already chosen; or count per residue class mod k |
| 17 | XOR + popcount | 17 | Flip the bits where (XOR of all) differs from k: `bitCount(total ^ k)` |
| 18 | LCM / unfolding the reflections | 18 | Unfold the room; the parities of the multipliers in lcm(p, q) decide the receptor |
| 19 | Bit filling | 17 | Spread the bits of n − 1 into the zero positions of x |
| 20 | Backtracking over subsets | 6 | Try every subset of the words (≤ 14) against the letter counts |
| 21 | Greedy + nCr | 18 | At each step, count the paths that start with 'H'; compare with k |
| 22 | Stars and bars (a bijection) | 18 | C(n + k − 1, 2k), by mapping segments to chosen points |
| 23 | Greedy stack counting | 12 | Unmatched `]` count m; the answer is ⌈m / 2⌉ |
| 24 | Binary search on the answer + subsequence check | 10 | If removing k characters keeps p as a subsequence, removing fewer does too |
| 25 | Window counting (Kind B) | 11 | For each r, every l ≤ the smallest valid left end works; add them up |
</details>
