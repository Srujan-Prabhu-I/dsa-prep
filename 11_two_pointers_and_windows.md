# Chapter 11 — Two Pointers & Sliding Windows

> **Part II · Week 8** · Prerequisites: Ch 1–10 (especially Ch 2 dominance, Ch 3 §3 amortisation)
> **Problems:** 8 worked (1 ★) · 10 practice · 3 stretch · **Time:** about 9 hours

"Two pointers" and "sliding window" are usually taught as two separate tricks with templates to memorise. They're really **one idea**:

> Instead of examining every pair `(i, j)`, keep two indices that **only ever move in one direction**. Every move **discards a set of candidates you can prove are useless.**

The pointers together move at most 2n times, so the work drops from O(n²) to O(n) (the amortisation argument from Ch 3 §3). That half is free. The other half, *why the discarded candidates really are useless*, is the part you must be able to prove, because it's also how you notice when the technique **doesn't** apply.

---

## 1. Attempt first ★

> **[Two Sum II – Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) (15 minutes).** `numbers` is sorted ascending. Return the 1-indexed positions of the two numbers that add up to `target`, using O(1) extra space. `[2, 7, 11, 15], target = 9` → `[1, 2]`.
>
> The hash map from Ch 7 solves it, but uses O(n) space. Find something better.

---

## 2. Opposite ends: eliminate a row or a column

### Worked P1a · [Two Sum II](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)

Picture every candidate pair `(i, j)` with `i < j` as a cell in a triangular grid. Start with the pointers at the two extremes, `l = 0` and `r = n − 1`, and compare `numbers[l] + numbers[r]` with the target:

* **Sum too big:** `numbers[r]` paired with *any* remaining partner `numbers[l′]`, `l ≤ l′ < r`, is at least as big, because the array is sorted. So **no remaining pair uses `r`**: discard it with `r--`. That's a whole column of the grid gone in one step.
* **Sum too small:** by the mirror argument, no remaining pair uses `l`, so `l++`. That's a whole row gone.

> **Invariant:** if an answer exists, both of its indices lie in `[l, r]`.

Each step keeps the invariant and shrinks `[l, r]` by one, so after at most n − 1 steps the answer is found (or proven absent). That's Ch 2's *dominance* tool, discarding an index that can't take part in any future answer, applied over and over.

```java
public int[] twoSum(int[] numbers, int target) {
    int l = 0, r = numbers.length - 1;
    while (l < r) {
        int sum = numbers[l] + numbers[r];
        if (sum == target) return new int[] {l + 1, r + 1};   // 1-indexed, as the problem asks
        if (sum > target) r--;      // numbers[r] is too big even with the smallest remaining partner
        else l++;                   // numbers[l] is too small even with the largest remaining partner
    }
    return new int[0];              // unreachable: an answer is guaranteed
}
```

O(n) time, O(1) space. **The proof used "sorted" in both branches.** On unsorted input, neither discard is justified, which is exactly why the hash map exists.

### Worked P1b · [Container With Most Water](https://leetcode.com/problems/container-with-most-water/)

> `height[i]` is a vertical line at position `i`. Choose two lines that, with the x-axis, hold the most water: area = `min(height[l], height[r]) · (r − l)`.

This array **isn't sorted**, yet two pointers still work, because the elimination argument rests on a different fact. Start at the widest pair `(0, n − 1)`. Suppose `height[l] ≤ height[r]`. Consider any other container that uses line `l` with a closer partner `r′ < r`:
* its width `r′ − l` is **smaller**, and
* its height `min(height[l], height[r′])` is **at most `height[l]`**.

So its area is at most the area we just measured. **Line `l` can never do better**, and we discard it. The symmetric argument discards `r` when it's the shorter one.

```java
public int maxArea(int[] height) {
    int l = 0, r = height.length - 1, best = 0;
    while (l < r) {
        best = Math.max(best, Math.min(height[l], height[r]) * (r - l));   // ≤ 10^4 · 10^5: fits in int
        if (height[l] < height[r]) l++;      // the shorter line can't beat this area with any closer partner
        else r--;
    }
    return best;
}
```

**What generalises:** two pointers need an *elimination argument*, and sortedness is just one way to get one. Here the argument is "the shorter side caps the height, and the width only shrinks".

---

## 3. Fix one, two-pointer the rest

### Worked P2 · [3Sum](https://leetcode.com/problems/3sum/)

> Return every distinct triple `[a, b, c]` from the array with `a + b + c = 0`.

The brute force tries all triples: O(n³). **Fix the first element** (Ch 1, question 2). What's left is "find pairs summing to `−a[i]`" in the rest of the array, which after sorting is Two Sum II. Total O(n²). **Distinct** triples come from the duplicate-skip idea of Ch 6: skip a value already used in the same role.

```java
public List<List<Integer>> threeSum(int[] nums) {
    int[] a = nums.clone();
    Arrays.sort(a);
    List<List<Integer>> result = new ArrayList<>();
    for (int i = 0; i < a.length - 2; i++) {
        if (i > 0 && a[i] == a[i - 1]) continue;   // the same first value would produce the same triples
        if (a[i] > 0) break;                       // sorted: all three values would be positive
        int l = i + 1, r = a.length - 1;
        while (l < r) {
            int sum = a[i] + a[l] + a[r];
            if (sum < 0) l++;
            else if (sum > 0) r--;
            else {
                result.add(List.of(a[i], a[l], a[r]));
                l++;
                r--;
                while (l < r && a[l] == a[l - 1]) l++;   // skip repeats of the second value
            }
        }
    }
    return result;
}
```

After a match, skipping repeats of the *second* value is enough. If `a[r]` also repeats, the new sum is then larger than 0 (a bigger second value with the same third value), so `r` moves on without recording anything.

**k-Sum in general:** fix k − 2 elements with nested loops (or recursion, Ch 6), and two-pointer the last two. That's O(n^(k−1)). 4Sum in the stretch set is k = 4. Use `long` for the sums.

---

## 4. Same direction: the reader and the writer

When you filter an array in place, use two pointers moving the **same** way: a **reader** that visits every element, and a **writer** that marks the end of the output kept so far. You met this in Ch 2 (Remove Duplicates) and Ch 5 (String Compression).

### Worked P3 · [Remove Duplicates from Sorted Array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/)

> In a sorted array, keep each value **at most twice**, in place. Return the new length.

```java
public int removeDuplicates(int[] nums) {
    int write = 0;              // nums[0, write) is the output so far: sorted, each value at most twice
    for (int x : nums) {
        if (write < 2 || x != nums[write - 2]) nums[write++] = x;
    }
    return write;
}
```

**Why comparing with `nums[write − 2]` is enough:** the output is sorted and `x` is at least everything in it. So `x == nums[write − 2]` means the last two kept values are both `x` already, and a third copy would break the rule. Change the 2 to k to allow k copies. The writer never overtakes the reader (`write ≤` the number of elements read), so nothing unread is ever overwritten.

---

## 5. Sliding windows: the property that makes them work

A **window** is a contiguous range `[l, r]`. Many problems ask for the longest or shortest window satisfying some condition, or for how many windows satisfy it. Checking every window costs O(n²) (or O(n³) if each check is O(n)). A sliding window does it in O(n), but **only when the condition has the right monotonicity**:

> **Kind A: shrinking keeps it valid.** If `[l, r]` is valid, so is every window inside it. (Example: "no repeated characters", "at most k distinct values", "sum ≤ S with non-negative values".)
>
> Then for each right end `r`, the valid left ends form a **range `[L(r), r]`**, and **`L(r)` never decreases** as `r` grows. The reason: if `[l, r + 1]` is valid, its sub-window `[l, r]` is valid too, so `L(r) ≤ L(r + 1)`.

> **Kind B: growing keeps it valid.** If `[l, r]` is valid, so is every window containing it. (Example: "sum ≥ S with positive values", "contains every character of t".)
>
> Then for each `r`, the valid left ends form a range `[0, L(r)]`, and again `L(r)` never decreases.

In both kinds, **the left end only ever moves right**. That's the elimination argument: a left end that's been passed can never be useful for any later right end. So both pointers move forward, and the total work is O(n). This gives two templates:

```java
// Kind A: the LONGEST valid window. Grow r; shrink l until the window is valid again.
for (int l = 0, r = 0; r < n; r++) {
    add(a[r]);
    while (!valid()) remove(a[l++]);
    best = Math.max(best, r - l + 1);
}

// Kind B: the SHORTEST valid window. Grow r; while valid, record and shrink.
for (int l = 0, r = 0; r < n; r++) {
    add(a[r]);
    while (valid()) {
        best = Math.min(best, r - l + 1);
        remove(a[l++]);
    }
}
```

`add`, `remove` and `valid` must each be **O(1)**: a running sum, a count array, or a counter of how many requirements are still unmet. If the window needs its maximum or median, that's no longer O(1). See §8.

### Worked P4 · [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

> Return the length of the longest substring with no repeated character.

"No repeats" is Kind A: a sub-window of a repeat-free window is repeat-free. Keep a count per character. When adding `s[r]` creates a repeat, shrink from the left until that one character's count is back to 1:

```java
public int lengthOfLongestSubstring(String s) {
    int[] count = new int[128];               // ASCII
    int best = 0;
    for (int l = 0, r = 0; r < s.length(); r++) {
        char c = s.charAt(r);
        count[c]++;
        while (count[c] > 1) count[s.charAt(l++)]--;   // only c can be repeated, so shrink until it isn't
        best = Math.max(best, r - l + 1);
    }
    return best;
}
```

(A variant stores each character's **last index** and jumps `l` straight past the earlier occurrence. It's the same idea with a different bookkeeping.)

### Worked P5a · [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/)

> Given **positive** integers and a `target`, return the length of the shortest subarray with sum ≥ target, or 0 if there is none.

With positive values, "sum ≥ target" is Kind B: growing a window only increases its sum.

```java
public int minSubArrayLen(int target, int[] nums) {
    int best = Integer.MAX_VALUE, sum = 0;
    for (int l = 0, r = 0; r < nums.length; r++) {
        sum += nums[r];
        while (sum >= target) {               // valid: record it, then try a shorter one
            best = Math.min(best, r - l + 1);
            sum -= nums[l++];
        }
    }
    return best == Integer.MAX_VALUE ? 0 : best;
}
```

**The proof used "positive"** to know that growing only raises the sum. With negatives, this is the counterexample from Ch 1 Derivation A, and you need prefix sums with a monotonic deque (Ch 12).

### Worked P5b · [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)

> Return the shortest substring of `s` that contains every character of `t`, counting multiplicity. `s = "ADOBECODEBANC", t = "ABC"` → `"BANC"`.

"Contains all of `t`" is Kind B. What makes `valid()` O(1) is a single counter:

> `need[c]` = (copies of `c` in `t`) − (copies of `c` in the window). It's positive while `c` is still needed, and negative when the window has spare copies.
> `missing` = how many characters of `t` (with multiplicity) the window still lacks. The window is valid exactly when `missing == 0`.

```java
public String minWindow(String s, String t) {
    int[] need = new int[128];
    for (int i = 0; i < t.length(); i++) need[t.charAt(i)]++;
    int missing = t.length();
    int bestStart = 0, bestLen = Integer.MAX_VALUE;
    for (int l = 0, r = 0; r < s.length(); r++) {
        if (need[s.charAt(r)]-- > 0) missing--;          // s[r] filled a real need
        while (missing == 0) {                            // the window covers t: record, then shrink
            if (r - l + 1 < bestLen) { bestLen = r - l + 1; bestStart = l; }
            if (++need[s.charAt(l++)] > 0) missing++;    // dropping s[l] re-opened a need
        }
    }
    return bestLen == Integer.MAX_VALUE ? "" : s.substring(bestStart, bestStart + bestLen);
}
```

O(|s| + |t|). Each character changes `missing` only when it crosses the boundary between "needed" and "spare", and that's what keeps every step O(1).

**The trace** on `s = "ADOBECODEBANC"`, `t = "ABC"` (only the steps where something happens):

```text
r   s[r]  effect                               missing   window and action
0   A     fills a need                         2
3   B     fills a need                         1
5   C     fills a need                         0         [0,5] "ADOBEC" valid: best = ADOBEC (6)
                                                         drop A: a need reopens → missing 1, l = 1
9   B     spare copy (B is no longer needed)   1
10  A     fills the reopened need              0         [1,10] valid, shrink: drop D, O, B (spare), E, all
                                                         still valid … [5,10] "CODEBA" (6), not shorter
                                                         drop C: a need reopens → missing 1, l = 6
12  C     fills it                             0         shrink [6,12] → [9,12] "BANC" (4): new best
                                                         drop B: reopens → missing 1, l = 10
end                                                      answer "BANC"
```

Notice the B at index 9: when it arrives, the window already holds a B (from index 3), so `need[B]` goes from 0 to −1, a spare copy, and `missing` doesn't change. Later, dropping the B at index 3 only brings `need[B]` back to 0, so the window stays valid and keeps shrinking.

---

## 6. Counting windows, and "exactly k"

For Kind A conditions, once `l` is positioned for a given `r`, **every** window `[l′, r]` with `l ≤ l′ ≤ r` is valid. So `r − l + 1` valid windows end at `r`. Adding that up over all `r` counts every valid window in O(n).

"**Exactly** k" conditions are *not* Kind A. A window with exactly k distinct values can have a sub-window with fewer. The fix is a subtraction:

> windows with **exactly** k = windows with **at most** k − windows with **at most** k − 1.

Each "at most" count *is* Kind A.

### Worked P6 · [Subarrays with K Different Integers](https://leetcode.com/problems/subarrays-with-k-different-integers/)

> Count the subarrays containing exactly `k` distinct values (values in `[1, n]`).

```java
public int subarraysWithKDistinct(int[] nums, int k) {
    return atMostDistinct(nums, k) - atMostDistinct(nums, k - 1);
}

// The number of subarrays with at most k distinct values (Kind A, so count r - l + 1 per r).
private int atMostDistinct(int[] nums, int k) {
    int[] count = new int[nums.length + 1];       // values are in [1, n]
    int distinct = 0, total = 0;
    for (int l = 0, r = 0; r < nums.length; r++) {
        if (count[nums[r]]++ == 0) distinct++;
        while (distinct > k) {
            if (--count[nums[l++]] == 0) distinct--;
        }
        total += r - l + 1;                       // every window [l', r] with l <= l' <= r is valid
    }
    return total;
}
```

`total` is at most n(n+1)/2 = 2·10⁸ for n = 2·10⁴, which fits in `int`. For larger n, use `long`. The same "exactly = atMost(k) − atMost(k − 1)" trick works for any condition where "at most k" is Kind A: exactly k odd numbers, sum exactly k with non-negative values, and so on.

---

## 7. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| **Fixed** window length k | Add `a[r]` and remove `a[r − k]` each step. No validity loop (Permutation in String) |
| Window that **wraps** around a circular array | Iterate over 2n indices, or use the complement: total minus a middle window |
| "Fewest elements removed from both ends" | The complement: the *longest middle window* with sum = total − x (Minimum Operations to Reduce X to Zero) |
| Window **maximum / minimum** | A monotonic deque (Ch 12) |
| Window **median** | Two heaps or ordered sets with lazy deletion (Ch 15) |
| Negative numbers, sum = k | Prefix sums + hash map (Ch 8) |
| Negative numbers, shortest sum ≥ k | Prefix sums + monotonic deque (Ch 12) |

---

## 8. Where two pointers break

| Situation | Why | Instead |
| :--- | :--- | :--- |
| **Negative values** with a sum condition | Growing a window can *lower* its sum, so neither Kind A nor Kind B holds (Ch 1 A: `[1, −1], k = 0`) | Prefix + hash (Ch 8), prefix + deque (Ch 12) |
| "Exactly k" | Not closed under shrinking | atMost(k) − atMost(k − 1) (§6) |
| A window statistic that isn't O(1) to update | Max, min, median, "k-th largest" | Deque (Ch 12), heaps / ordered sets (Ch 15) |
| A pair problem on **unsorted** data with no elimination argument | Discarding `l` or `r` isn't justified | Sort first (if indices don't matter), or hash (Ch 7) |
| Subsequences (not contiguous) | Windows are contiguous by definition | DP (Ch 27, 29) or greedy |

**The one-question test.** Before writing any window, ask: *"If `[l, r]` is valid, is `[l + 1, r]` valid?"* (Kind A), or *"is `[l, r + 1]` valid?"* (Kind B). If you can't answer yes to one of them with a reason, the window will be wrong on some input.

---

## 9. Recognition

**Signals:**
* **Sorted** input, and a question about pairs or triples with a sum or difference.
* "**Longest / shortest contiguous** subarray or substring such that …", where the condition is Kind A or Kind B.
* "**Count** the subarrays such that …", with a Kind A condition, or "exactly" (subtract two "at most" counts).
* In-place filtering or compaction (reader/writer).
* Two sorted sequences to merge or compare.

**False friends:**
* "Subarray sum = k" with **negatives**: prefix sums (Ch 8).
* "Longest **subsequence**": not contiguous, so DP.
* Maximum in every window: a deque (Ch 12), not a plain window.

---

## 10. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Using a window when values can be negative | Check Kind A / Kind B first (§8) |
| Recording a Kind B answer *after* shrinking | Record inside the `while (valid())` loop, before removing |
| Recounting the whole window to test validity (O(n) per step) | Maintain counts or a `missing` counter incrementally |
| Skipping duplicates in 3Sum before recording the match | Record first, then skip |
| `l` passing `r` in a fixed-pattern loop | For Kind A the window can become empty (`l = r + 1`); make sure `valid()` handles that |
| `int` counts of windows when n is large | `long` |
| `Map<Character, Integer>` for ASCII windows | `int[128]` |

---

## 11. Practice

1. **[Move Zeroes](https://leetcode.com/problems/move-zeroes/)** *(Easy)*
2. **[Squares of a Sorted Array](https://leetcode.com/problems/squares-of-a-sorted-array/)** *(Easy)*
3. **[3Sum Closest](https://leetcode.com/problems/3sum-closest/)** *(Medium)*
4. **[Permutation in String](https://leetcode.com/problems/permutation-in-string/)** *(Medium)*
5. **[Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/)** *(Medium)*
6. **[Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/)** *(Medium)*
7. **[Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/)** *(Medium)*
8. **[Minimum Operations to Reduce X to Zero](https://leetcode.com/problems/minimum-operations-to-reduce-x-to-zero/)** *(Medium)*
9. **[Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)** *(Hard)*
10. **[Count Subarrays Where Max Element Appears at Least K Times](https://leetcode.com/problems/count-subarrays-where-max-element-appears-at-least-k-times/)** *(Medium)*

**Stretch:** **[4Sum](https://leetcode.com/problems/4sum/)** *(Medium)* · **[Substring with Concatenation of All Words](https://leetcode.com/problems/substring-with-concatenation-of-all-words/)** *(Hard)* · **[Count Subarrays With Fixed Bounds](https://leetcode.com/problems/count-subarrays-with-fixed-bounds/)** *(Hard)*

<details><summary>Nudges</summary>

1. Move Zeroes: reader/writer. Write the non-zeros, then fill the rest with zeros. Or swap as you go.
2. Squares: the largest square is at one of the two ends. Fill the output from the back.
3. 3Sum Closest: 3Sum without the duplicate handling; track the closest sum seen so far.
4. Permutation in String: a fixed window of length |s1|; compare 26 counts, or maintain a "matching letters" counter.
5. Max Ones III: Kind A, "at most k zeros in the window".
6. Char Replacement: valid ⟺ `length − maxCount ≤ k`. Why does it still work if `maxCount` is never decreased when the window shrinks?
7. Product < K: Kind A with positive values. Count `r − l + 1` per `r`. When k ≤ 1 the answer is 0.
8. Reduce X: removing from both ends = keeping a middle window. Find the longest window with sum = total − x. Values are positive.
9. Rain Water: the water above i is `min(maxLeft, maxRight) − height[i]`. With two pointers, the side with the smaller maximum is already decided.
10. Max ≥ k Times: let M be the global maximum. Kind B ("contains M at least k times"): for each r, every l ≤ L(r) works. Use `long`.
11. Stretch, 4Sum: fix two indices, two-pointer the rest, skip duplicates at every level, and use `long` sums.
12. Stretch, Concatenation: all the words have the same length w. Run w separate windows, one per starting offset, each stepping by whole words.
13. Stretch, Fixed Bounds: for each r, track the last positions of minK, of maxK, and of any out-of-range value. The count for r is `max(0, min(lastMin, lastMax) − lastBad)`.
</details>

<details><summary>Key</summary>

Move Zeroes → P3 · Squares → P1 (fill from both ends) · 3Sum Closest → P2 · Permutation in String → fixed window · Max Ones III → P4 · Char Replacement → P4 (a stale max is safe because the answer only grows) · Product < K → P6 counting · Reduce X → complement + P4 · Rain Water → P1 with a two-sided max invariant · Max ≥ k Times → P6 (Kind B counting) · 4Sum → P2 as k-sum · Concatenation → P4 with word-sized strides · Fixed Bounds → P6 tracking last positions
</details>

---

## 12. Self-test

<details><summary>1. In Two Sum II, why can r be discarded when the sum is too big?</summary>

Every remaining partner `numbers[l′]` (with `l ≤ l′ < r`) is at least `numbers[l]`, because the array is sorted, so every remaining pair with `r` is also too big.
</details>

<details><summary>2. State the property that makes a "longest valid window" algorithm correct.</summary>

Kind A: if `[l, r]` is valid then every sub-window is valid. So the smallest valid left end `L(r)` never decreases as `r` grows, and `l` only needs to move forward.
</details>

<details><summary>3. Why doesn't "subarray sum ≥ target" work with a window when values can be negative?</summary>

Adding an element can lower the sum, so growing a window doesn't preserve validity. Neither Kind A nor Kind B holds, and moving `l` forward can skip the answer.
</details>

<details><summary>4. Why is "exactly k distinct" computed as atMost(k) − atMost(k − 1)?</summary>

"Exactly k" isn't closed under shrinking, but "at most k" is (Kind A). The windows with exactly k distinct values are exactly those with at most k, minus those with at most k − 1.
</details>

<details><summary>5. In Container With Most Water, why move the shorter line?</summary>

Any container using the shorter line with a closer partner has a smaller width and a height capped by that shorter line, so it can't beat the area just measured.
</details>

---

## 13. Exit gate

Move on when you can:

* prove the elimination step for Two Sum II and Container With Most Water out loud;
* classify any window condition as Kind A, Kind B or neither, with a one-line reason;
* write the two window templates, Minimum Window Substring and the atMost-difference count from blank files in under 10 minutes each;
* name the replacement technique for each failure case in §8;
* solve 8 of the 10 practice problems.
