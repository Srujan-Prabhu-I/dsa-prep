# Mixed Set II — after Chapter 16

> **Week 14** · 30 problems, unlabeled, drawn from Ch 1–16 · **Time:** about 12 hours over the week

This set is where you find out whether you can **choose** a technique, not just apply one you've been told to use. The problems are deliberately in random order. Nothing here tells you which chapter a problem belongs to, and several look like one chapter but belong to another.

## How to work this set

For each problem, **before writing code**, spend 3–5 minutes and write down:

1. **Restate + budget:** the object being chosen, the constraint, and the target complexity (Ch 1 Steps 1–2).
2. **The bottleneck** of the brute force, named as one of the five kinds (Ch 1 §2, Step 5).
3. **Your top two candidate techniques**, each with the **structural reason** it might apply: "monotone feasibility", "dominance", "pair of prefixes", "sort by end", and so on.
4. **The one-sentence proof** of the one you choose.

Then solve it in Perform mode: Medium 25 minutes, Hard 45 minutes, no hints. Afterwards, check the key and log it:
* the **key insight** and the **signal**;
* a failure tag if you missed it (`IDEA` / `PROOF` / `IMPL` / `EDGE` / `TIME`);
* whether your top-two candidates contained the right technique.

**Scoring.** Track two numbers: **solved** (out of 30) and **chose correctly** (the right technique was in your top two, out of 30). The checkpoint is ≥ 70% on both. If "chose correctly" is low but "solved" is fine, the gap is in recognition. Reread the Recognition sections of the chapters you missed, and redo those problems in two weeks.

---

## The set

1. [Find Pivot Index](https://leetcode.com/problems/find-pivot-index/)
2. [Maximum Width Ramp](https://leetcode.com/problems/maximum-width-ramp/)
3. [Grumpy Bookstore Owner](https://leetcode.com/problems/grumpy-bookstore-owner/)
4. [Minimize the Maximum Difference of Pairs](https://leetcode.com/problems/minimize-the-maximum-difference-of-pairs/)
5. [K-diff Pairs in an Array](https://leetcode.com/problems/k-diff-pairs-in-an-array/)
6. [Rotate Array](https://leetcode.com/problems/rotate-array/)
7. [Partition Labels](https://leetcode.com/problems/partition-labels/)
8. [Frequency of the Most Frequent Element](https://leetcode.com/problems/frequency-of-the-most-frequent-element/)
9. [Remove All Adjacent Duplicates in String II](https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string-ii/)
10. [Count Number of Teams](https://leetcode.com/problems/count-number-of-teams/)
11. [Online Election](https://leetcode.com/problems/online-election/)
12. [Longest Mountain in Array](https://leetcode.com/problems/longest-mountain-in-array/)
13. [Maximum Sum of Distinct Subarrays With Length K](https://leetcode.com/problems/maximum-sum-of-distinct-subarrays-with-length-k/)
14. [Find Right Interval](https://leetcode.com/problems/find-right-interval/)
15. [Majority Element II](https://leetcode.com/problems/majority-element-ii/)
16. [Number of Subsequences That Satisfy the Given Sum Condition](https://leetcode.com/problems/number-of-subsequences-that-satisfy-the-given-sum-condition/)
17. [Hand of Straights](https://leetcode.com/problems/hand-of-straights/)
18. [Maximum Value at a Given Index in a Bounded Array](https://leetcode.com/problems/maximum-value-at-a-given-index-in-a-bounded-array/)
19. [Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/)
20. [Minimum Moves to Equal Array Elements II](https://leetcode.com/problems/minimum-moves-to-equal-array-elements-ii/)
21. [Maximum Frequency Stack](https://leetcode.com/problems/maximum-frequency-stack/)
22. [Minimum Swaps to Group All 1's Together II](https://leetcode.com/problems/minimum-swaps-to-group-all-1s-together-ii/)
23. [Number of Visible People in a Queue](https://leetcode.com/problems/number-of-visible-people-in-a-queue/)
24. [Range Frequency Queries](https://leetcode.com/problems/range-frequency-queries/)
25. [Longest Subarray of 1's After Deleting One Element](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/)
26. [House Robber IV](https://leetcode.com/problems/house-robber-iv/)
27. [Minimum Absolute Sum Difference](https://leetcode.com/problems/minimum-absolute-sum-difference/)
28. [Maximum Erasure Value](https://leetcode.com/problems/maximum-erasure-value/)
29. [Minimum Number of K Consecutive Bit Flips](https://leetcode.com/problems/minimum-number-of-k-consecutive-bit-flips/)
30. [Sum of Subsequence Widths](https://leetcode.com/problems/sum-of-subsequence-widths/)

---

<details><summary>Key: open only after attempting all 30 (or after each problem's timebox)</summary>

| # | Technique | Chapter | The insight in one line |
| :-: | :--- | :-: | :--- |
| 1 | Prefix sums | 8 | Left sum = prefix; right sum = total − prefix − the element |
| 2 | Monotonic stack | 12 | A decreasing stack of left candidates (only new minimums can start a widest ramp); scan from the right, popping matches |
| 3 | Fixed sliding window | 11 | Maximise the *extra* customers won inside a window of length `minutes` |
| 4 | Binary search on the answer + greedy | 10 | Is a maximum difference d feasible? Sort, then greedily pair adjacent elements |
| 5 | Hashing | 7 | For each distinct x, look up x + k (special case k = 0: count values appearing ≥ 2 times) |
| 6 | Three reversals | 5 | Reverse all, then reverse the first k and the rest (k %= n) |
| 7 | Greedy with last occurrences | 7, 14 | A part can end once it reaches the furthest last occurrence of every letter inside it |
| 8 | Sort + sliding window | 9, 11 | After sorting, the window is valid when `max · len − sum ≤ k` (use `long`) |
| 9 | Stack of (char, count) | 12 | Push runs; pop when a count reaches k |
| 10 | Fix the middle, count both sides | 1 | For each middle soldier, (smaller on the left × larger on the right) + (larger on the left × smaller on the right) |
| 11 | Precompute leaders + binary search | 10 | The leader after each vote, then the last vote time ≤ t |
| 12 | One pass / up–down run counts | 5 | Track the lengths of the current ascent and descent |
| 13 | Fixed window + hash counts | 7, 11 | Slide the window with counts; it's valid when every value is distinct |
| 14 | Sort starts + binary search | 10, 14 | For each interval, the lower bound of its end among the sorted starts |
| 15 | Boyer–Moore with two candidates | 2 | At most two values exceed n/3; cancel triples, then verify |
| 16 | Sort + two pointers + powers of 2 | 9, 11 | For each min, the largest max that fits; there are 2^(r − l) subsequences with that min (mod) |
| 17 | TreeMap greedy from the smallest | 15 | The smallest card must start a group; consume consecutive values |
| 18 | Binary search on the answer + arithmetic series | 10 | Is the peak value v feasible? Sum the two arithmetic slopes, clipped at 1 (use `long`) |
| 19 | Fixed window with frequency counts | 11 | Compare 26 counts as the window slides |
| 20 | Median minimises total distance | 2 | Move everything to the median |
| 21 | Frequency → stack map | 16 | Bucket per frequency; each bucket is a stack; track the maximum frequency |
| 22 | Circular fixed window | 11 | Window size = total ones; find the window with the most ones over the doubled array |
| 23 | Monotonic stack | 12 | Scan from the right with a decreasing stack; each person sees those popped, plus one more |
| 24 | Hash value → sorted index list + binary search | 7, 10 | Count of a value in [l, r] = upper bound − lower bound in its index list |
| 25 | Window with at most one zero | 11 | Kind A window; the answer is the window length − 1 |
| 26 | Binary search on the answer + greedy check | 10 | Can we rob k houses using only houses ≤ c? Greedily take a house, then skip the next |
| 27 | Sort a copy + binary search for the best replacement | 9, 10 | For each i, the closest value to `nums2[i]` in sorted `nums1` (mod at the end) |
| 28 | Window with a set | 11 | The longest window of distinct values, maximising its sum |
| 29 | Greedy + difference array | 8 | Flip at the first 0 you meet; a difference array tracks the flips still active |
| 30 | Sort + contribution | 9, 12 | After sorting, a[i] is the max in 2^i subsequences and the min in 2^(n−1−i) |
</details>
