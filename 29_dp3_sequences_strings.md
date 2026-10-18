# Chapter 29 — DP III: Sequences & Strings

> **Part VI · Week 26** · Prerequisites: Ch 10 (the binary-search template), Ch 27, Ch 28 (rolling arrays and loop direction)
> **Problems:** 7 worked (1 ★) · 10 practice · 3 stretch · **Time:** about 10 hours

String and sequence DP is the most heavily tested DP family in interviews, and it's highly regular. Nearly every problem uses one of three state shapes:

| State shape | Meaning | Problems |
| :--- | :--- | :--- |
| `dp[i]` = the best for a subsequence **ending at i** | One sequence, choosing a subsequence | LIS and its relatives |
| `dp[i][j]` = the answer for the **prefixes** `a[0..i)` and `b[0..j)` | Two sequences aligned against each other | LCS, edit distance, matching, interleaving |
| `dp[i][j]` = the answer for the **substring** `s[i..j]` | One sequence, using both ends | Palindromes (and Ch 30's interval DP) |

Once you've recognised the shape, the transition always asks the same question: **what happens to the last characters?**

---

## 1. Attempt first ★

> **[Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) (25 minutes).** Return the length of the longest strictly increasing subsequence. `[10, 9, 2, 5, 3, 7, 101, 18]` → 4 (`2, 3, 7, 18`). First get O(n²); then look for O(n log n).

---

## 2. LIS: from O(n²) to O(n log n)

### Worked Q1 · [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) ★

**The O(n²) DP** (Ch 1 Derivation C's move: define the answer by its last element). `len[i]` = the longest increasing subsequence **ending at index i**, which is 1 plus the best `len[j]` over earlier `j` with `nums[j] < nums[i]`. That's n states with O(n) transitions each.

**Toward O(n log n): a better state.** For n = 2.5·10⁵, n² is too slow. The bottleneck is re-scanning every earlier j. Ask the dominance question (Ch 1, Q5): among all increasing subsequences of the **same length**, which one is the most useful for the future? **The one with the smallest last element**, because anything that extends a larger tail also extends the smaller one. So keep only that:

> **`tails[k]`** = the smallest possible last element of an increasing subsequence of length k + 1, among the elements processed so far.

**Fact 1: `tails` is strictly increasing.** Take a subsequence of length k + 2 ending with `tails[k+1]`. Its first k + 1 elements form a subsequence of length k + 1 ending in something smaller than `tails[k+1]`. So `tails[k] < tails[k+1]`.

**Fact 2: how a new element x updates `tails`.** Let k be the **first** position with `tails[k] ≥ x`.
* `tails[k − 1] < x` (or k = 0), so x extends a subsequence of length k, giving one of length k + 1 that ends in x. Since x ≤ `tails[k]`, x becomes the new smallest tail for that length: **set `tails[k] = x`**. If no such k exists (x is larger than every tail), x extends the longest subsequence, so append it.
* No other entry changes. A shorter length would end in x, which is larger than its current tail. A longer length would need a predecessor tail below x, but those tails are all ≥ x.

So the answer is simply **the size of `tails`**, and finding k is the lower-bound search from Ch 10:

```java
public int lengthOfLIS(int[] nums) {
    int[] tails = new int[nums.length];          // tails[k] = smallest tail of an increasing subsequence of length k + 1
    int size = 0;                                 // tails[0, size) is strictly increasing
    for (int x : nums) {
        int lo = -1, hi = size;                   // Ch 10: the first k with tails[k] >= x (hi = size if none)
        while (hi - lo > 1) {
            int mid = lo + (hi - lo) / 2;
            if (tails[mid] >= x) hi = mid; else lo = mid;
        }
        tails[hi] = x;                            // a smaller tail for length hi + 1 (or a new, longer length)
        if (hi == size) size++;
    }
    return size;
}
```

**O(n log n).** Two cautions:
* **`tails` is not itself an LIS.** It mixes elements from different subsequences. To reconstruct one, store for each element the index of its predecessor (the element at `tails[k − 1]` when it was placed) and walk back from the end.
* **Non-decreasing** (equal values allowed) changes the search to the first `tails[k] > x`.

Russian Doll Envelopes (practice) is LIS on one dimension, after sorting the other cleverly. Number of LIS needs the O(n²) DP with counts.

---

## 3. Two sequences: what happens to the last characters?

For two strings `a` (length n) and `b` (length m), the state is a pair of **prefix lengths**: `dp[i][j]` describes `a[0..i)` and `b[0..j)`. The transition looks at the last characters, `a[i−1]` and `b[j−1]`.

### Worked Q2a · [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)

**State:** `lcs[i][j]` = the LCS length of `a[0..i)` and `b[0..j)`.
**Last characters:**
* **Equal** (`a[i−1] == b[j−1]`): some LCS matches these two characters to each other. *Exchange:* take any LCS. If it doesn't use both of them, re-match its last character to this pair; that's still a common subsequence of the same length. So `lcs[i][j] = lcs[i−1][j−1] + 1`.
* **Different:** they can't both end the LCS, so at least one of them is unused: `max(lcs[i−1][j], lcs[i][j−1])`.

```java
public int longestCommonSubsequence(String a, String b) {
    int n = a.length(), m = b.length();
    int[][] lcs = new int[n + 1][m + 1];
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            lcs[i][j] = (a.charAt(i - 1) == b.charAt(j - 1))
                    ? lcs[i - 1][j - 1] + 1                          // use both last characters
                    : Math.max(lcs[i - 1][j], lcs[i][j - 1]);       // drop one of them
    return lcs[n][m];
}
```

O(n · m). **Reconstruction** walks back from `(n, m)`, following whichever case produced each value:

```java
StringBuilder sb = new StringBuilder();
for (int i = n, j = m; i > 0 && j > 0; ) {
    if (a.charAt(i - 1) == b.charAt(j - 1)) { sb.append(a.charAt(i - 1)); i--; j--; }
    else if (lcs[i - 1][j] >= lcs[i][j - 1]) i--;
    else j--;
}
String common = sb.reverse().toString();
```

Many "different" problems are LCS in disguise. Uncrossed Lines *is* LCS, and Delete Operation for Two Strings is n + m − 2·LCS. Shortest Common Supersequence (stretch) is LCS plus reconstruction.

### Worked Q2b · [Edit Distance](https://leetcode.com/problems/edit-distance/)

> Return the fewest single-character insertions, deletions and replacements that turn `a` into `b`.

**State:** `ops[i][j]` = the fewest edits turning `a[0..i)` into `b[0..j)`.
**The last operation** that fixes the final characters is one of:
* the last characters are **already equal**: no edit, `ops[i−1][j−1]`;
* **replace** `a[i−1]` with `b[j−1]`: `1 + ops[i−1][j−1]`;
* **delete** `a[i−1]`: `1 + ops[i−1][j]`;
* **insert** `b[j−1]` at the end: `1 + ops[i][j−1]`.

**Base cases:** turning a prefix of length i into the empty string takes i deletions, and turning the empty string into a prefix of length j takes j insertions.

```java
public int minDistance(String a, String b) {
    int n = a.length(), m = b.length();
    int[][] ops = new int[n + 1][m + 1];
    for (int i = 0; i <= n; i++) ops[i][0] = i;                 // delete everything
    for (int j = 0; j <= m; j++) ops[0][j] = j;                 // insert everything
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            if (a.charAt(i - 1) == b.charAt(j - 1)) {
                ops[i][j] = ops[i - 1][j - 1];                  // the last characters already match
            } else {
                ops[i][j] = 1 + Math.min(ops[i - 1][j - 1],     // replace
                                Math.min(ops[i - 1][j],         // delete from a
                                         ops[i][j - 1]));       // insert into a
            }
        }
    }
    return ops[n][m];
}
```

**The filled table** for `a = "horse"`, `b = "ros"` (rows are prefixes of a, columns are prefixes of b):

```text
           ""   r    o    s
   ""       0    1    2    3        row 0: insert j characters
   h        1    1    2    3        (h, r): different → 1 + min(diag 0, up 1, left 1) = 1   (replace h→r)
   o        2    2    1    2        (o, o): equal     → copy the diagonal: ops[1][1] = 1
   r        3    2    2    2
   s        4    3    3    2        (s, s): equal     → diagonal ops[3][2] = 2
   e        5    4    4    3        answer ops[5][3] = 3
```

Each cell looks at three neighbours: **diagonal** (match or replace), **up** (delete from a), **left** (insert into a). Fill row by row, left to right, and every neighbour is ready. Walking back from the bottom-right corner along the cells that produced each value recovers the edits: replace h → r, keep o, delete r, keep s, delete e. That's 3 operations.

When the last characters are equal, taking `ops[i−1][j−1]` is always at least as good as the other options, because adjacent entries of the table differ by at most 1. Each row depends only on the row above it, so a rolling array (with one saved diagonal value) gives O(min(n, m)) space when that matters.

---

## 4. Palindromes: using both ends

### Worked Q3a · [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/)

A palindrome reads the same from both ends, so **grow it from its centre**. There are 2n − 1 possible centres: n characters (odd lengths) and n − 1 gaps between characters (even lengths). Expand outward while the ends match:

```java
public String longestPalindrome(String s) {
    int bestStart = 0, bestLen = 1;
    for (int center = 0; center < s.length(); center++) {
        for (int gap = 0; gap <= 1; gap++) {                     // gap 0: odd length; gap 1: even length
            int l = center, r = center + gap;
            while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
            int len = r - l - 1;                                 // the palindrome is s(l, r), both ends exclusive
            if (len > bestLen) { bestLen = len; bestStart = l + 1; }
        }
    }
    return s.substring(bestStart, bestStart + bestLen);
}
```

That's O(n²) time and O(1) space. The table version, `pal[i][j] = s[i] == s[j] && pal[i+1][j−1]`, is also O(n²), but it uses O(n²) memory. Its value is that it *records* which substrings are palindromes, which Palindrome Partitioning II (stretch) needs. (Manacher's algorithm does this in O(n), and is recognition-only for interviews.)

### Worked Q3b · [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/)

A **subsequence** can skip characters, so expanding from the centre no longer works. Use the substring state:

> `lps[i][j]` = the longest palindromic subsequence of `s[i..j]`. If the two ends match, they wrap a palindrome of the inside: `lps[i+1][j−1] + 2`. Otherwise one of the ends is unused: `max(lps[i+1][j], lps[i][j−1])`.

**Order:** every state depends on *shorter* substrings inside it, so iterate i from high to low, and j from low to high.

```java
public int longestPalindromeSubseq(String s) {
    int n = s.length();
    int[][] lps = new int[n][n];
    for (int i = n - 1; i >= 0; i--) {                     // [i+1, j-1], [i+1, j] and [i, j-1] are already computed
        lps[i][i] = 1;
        for (int j = i + 1; j < n; j++) {
            if (s.charAt(i) == s.charAt(j)) lps[i][j] = lps[i + 1][j - 1] + 2;
            else lps[i][j] = Math.max(lps[i + 1][j], lps[i][j - 1]);
        }
    }
    return lps[0][n - 1];
}
```

Equivalently, `lps(s) = LCS(s, reverse(s))`. Minimum Insertion Steps to Make a String Palindrome (practice) is `n − lps(s)`.

---

## 5. Matching with wildcards

### Worked Q4 · [Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/)

> Does the pattern `p` match the **whole** of `s`? In the pattern, `'.'` matches any single character, and `'*'` means **zero or more copies of the element before it**.

**State:** `match[i][j]` = `s[0..i)` matches `p[0..j)`.
**The last pattern element:**
* **An ordinary character or `.`:** it must match `s[i−1]`, and the prefixes before them must match: `match[i−1][j−1]`.
* **`x*`** (the last two pattern characters), two cases:
  * **zero copies of x:** ignore `x*` entirely, giving `match[i][j−2]`;
  * **one more copy:** if x matches `s[i−1]`, consume that character but keep `x*`, because it may match more: `match[i−1][j]`.

**Base case:** an empty pattern matches only the empty string. But a non-empty pattern like `a*b*` *does* match the empty string, so `match[0][j]` must handle that.

```java
public boolean isMatch(String s, String p) {
    int n = s.length(), m = p.length();
    boolean[][] match = new boolean[n + 1][m + 1];
    match[0][0] = true;
    for (int j = 2; j <= m; j++) match[0][j] = p.charAt(j - 1) == '*' && match[0][j - 2];   // "a*b*" matches ""
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            char pc = p.charAt(j - 1);
            if (pc == '*') {
                char x = p.charAt(j - 2);                                   // the element being repeated
                boolean zeroCopies = match[i][j - 2];
                boolean oneMore = (x == '.' || x == s.charAt(i - 1)) && match[i - 1][j];
                match[i][j] = zeroCopies || oneMore;
            } else {
                match[i][j] = (pc == '.' || pc == s.charAt(i - 1)) && match[i - 1][j - 1];
            }
        }
    }
    return match[n][m];
}
```

O(n · m). **The `match[i−1][j]` in "one more" is the key move:** it keeps `x*` available instead of consuming it, so one transition handles any number of copies. Wildcard Matching (stretch) has the same structure, with `*` matching any sequence on its own.

---

## 6. Counting subsequences

### Worked Q5 · [Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/)

> Count the subsequences of `s` that equal `t`.

**State:** `ways[i][j]` = the number of ways to form `t[0..j)` from `s[0..i)`.
**Last character of s:** either `s[i−1]` isn't used (`ways[i−1][j]`), or, if it equals `t[j−1]`, it's used as t's last character (`ways[i−1][j−1]`). Those are different choices of positions, so the two options never overlap, and adding them is correct.

**Rolling:** each row depends only on the previous one. Iterate j **downward**, like the 0/1 knapsack (Ch 28 §4), because each character of s may be used at most once:

```java
public int numDistinct(String s, String t) {
    int m = t.length();
    int[] ways = new int[m + 1];                 // ways[j] = ways to form t[0, j) from the prefix of s seen so far
    ways[0] = 1;                                 // the empty target: one way (use nothing)
    for (int i = 0; i < s.length(); i++) {
        for (int j = m; j >= 1; j--) {           // downward: ways[j - 1] still describes the earlier prefix of s
            if (s.charAt(i) == t.charAt(j - 1)) ways[j] += ways[j - 1];
        }
    }
    return ways[m];
}
```

O(|s| · |t|). The final answer is guaranteed to fit in 32 bits. Intermediate entries can overflow, which is harmless, for the same wrap-around reason as Ch 28 §6.

---

## 7. Where string DPs break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| n, m ≈ 10⁵ for two-string DP | n · m = 10¹⁰ | Special structure (LCS of permutations → LIS); hashing + binary search (Ch 35) |
| LIS with n = 10⁵ using O(n²) | Too slow | The `tails` method (§2) |
| Treating `tails` as the LIS itself | It mixes subsequences | Store predecessors to reconstruct |
| Regex: consuming `x*` in the "one more" case | Then it can match only one copy | Keep j the same: `match[i−1][j]` |
| Palindromic *substring* solved with the subsequence DP | Wrong answers (subsequences may skip characters) | Expand around centres |

---

## 8. Recognition

**Signals:**
* "**Subsequence**" (not contiguous) of one sequence, with an order condition → `dp[i]` ending at i; LIS-style.
* **Two strings** with "common", "edit", "transform", "delete to make equal", "interleave", "match" → `dp[i][j]` over prefixes.
* "**Palindrome**" → substring: expand around centres; subsequence: `dp[i][j]` over intervals.
* "**Wildcards**", "regex" → prefix DP with a case for each pattern element.
* "**How many ways** to form t from s" → a counting prefix DP, with downward rolling.
* A **nested chain** where both dimensions must increase (envelopes, boxes) → sort one dimension, then LIS on the other.

**False friends:** contiguous "substring" problems with a validity condition are usually windows (Ch 11), not DP.

---

## 9. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Mixing up `dp[i]` = "the first i characters" and "index i" | Prefix states have size n + 1; comment which convention you're using |
| Wrong base cases for empty prefixes (edit distance, regex) | Fill row 0 and column 0 explicitly |
| Rolling a two-string DP but losing the diagonal value | Save `dp[i−1][j−1]` in a variable before overwriting it |
| Upper bound versus lower bound in LIS (strict versus non-decreasing) | Strict: the first `≥ x`; non-decreasing: the first `> x` |
| Interval DP filled in the wrong order | Inner states first: i descending, j ascending (or by length) |

---

## 10. Practice

1. **[Uncrossed Lines](https://leetcode.com/problems/uncrossed-lines/)** *(Medium)*
2. **[Minimum ASCII Delete Sum for Two Strings](https://leetcode.com/problems/minimum-ascii-delete-sum-for-two-strings/)** *(Medium)*
3. **[Longest String Chain](https://leetcode.com/problems/longest-string-chain/)** *(Medium)*
4. **[Largest Divisible Subset](https://leetcode.com/problems/largest-divisible-subset/)** *(Medium)*
5. **[Number of Longest Increasing Subsequence](https://leetcode.com/problems/number-of-longest-increasing-subsequence/)** *(Medium)*
6. **[Minimum Insertion Steps to Make a String Palindrome](https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/)** *(Hard)*
7. **[Interleaving String](https://leetcode.com/problems/interleaving-string/)** *(Medium)*
8. **[Longest Ideal Subsequence](https://leetcode.com/problems/longest-ideal-subsequence/)** *(Medium)*
9. **[Russian Doll Envelopes](https://leetcode.com/problems/russian-doll-envelopes/)** *(Hard)*
10. **[Minimum Number of Removals to Make Mountain Array](https://leetcode.com/problems/minimum-number-of-removals-to-make-mountain-array/)** *(Hard)*

**Stretch:** **[Wildcard Matching](https://leetcode.com/problems/wildcard-matching/)** *(Hard)* · **[Palindrome Partitioning II](https://leetcode.com/problems/palindrome-partitioning-ii/)** *(Hard)* · **[Shortest Common Supersequence](https://leetcode.com/problems/shortest-common-supersequence/)** *(Hard)*

<details><summary>Nudges</summary>

1. Uncrossed Lines: draw the picture. Non-crossing matched lines are exactly a common subsequence.
2. ASCII Delete: like LCS, but the cost of dropping a character is its ASCII value; minimise.
3. String Chain: sort by length; `best[word]` = 1 + the best over predecessors (the word with one character removed), using a map.
4. Divisible Subset: sort; `len[i]` = 1 + the max `len[j]` over j with `nums[i] % nums[j] == 0`; reconstruct via parents. Why does sorting make divisibility chain transitively?
5. Number of LIS: the O(n²) DP with a (length, count) pair per index; for equal lengths, add the counts.
6. Insertion Steps: n − (longest palindromic subsequence).
7. Interleaving: `ok[i][j]` = s3's first i + j characters interleave `s1[0..i)` and `s2[0..j)`; the last character came from s1 or from s2.
8. Ideal Subsequence: `best[c]` = the longest valid subsequence ending with letter c; each new letter looks at the letters within k of it. O(26 · n).
9. Russian Doll: sort by width ascending, and for equal widths by height **descending**, then run LIS on the heights. Why the descending tie-break?
10. Mountain: the LIS ending at each i from the left and from the right; the best peak needs both lengths ≥ 2.
11. Stretch, Wildcard: `*` matches any sequence, so `match[i][j] = match[i][j−1]` (it matches nothing) `|| match[i−1][j]` (it absorbs one more character).
12. Stretch, Partitioning II: precompute the palindrome table; `cuts[i]` = the fewest cuts for the first i characters; the last piece is a palindrome `s[j..i)`.
13. Stretch, Supersequence: LCS, then walk back, emitting shared characters once and unshared ones from whichever side you step back on.
</details>

<details><summary>Key</summary>

Uncrossed → Q2 (it's LCS) · ASCII Delete → weighted Q2 · String Chain → Q1-style over sorted lengths · Divisible Subset → Q1 + reconstruction · Number of LIS → Q1 with counts · Insertion Steps → Q3 (n − LPS) · Interleaving → Q2 on a prefix grid · Ideal Subsequence → a DP over the last letter · Russian Doll → sort (w ↑, h ↓) + Q1 · Mountain → Q1 from both sides · Wildcard → Q4 · Partitioning II → Q3 table + a cut DP · Supersequence → Q2 + reconstruction
</details>

---

## 11. Self-test

<details><summary>1. What does tails[k] mean in the O(n log n) LIS, and why is it strictly increasing?</summary>

The smallest possible tail of an increasing subsequence of length k + 1. The first k + 1 elements of a length-(k + 2) subsequence ending at `tails[k+1]` form a length-(k + 1) subsequence with a smaller tail, so `tails[k] < tails[k+1]`.
</details>

<details><summary>2. In LCS, why is matching the last two characters safe when they're equal?</summary>

By exchange: any LCS can be rearranged so its last character is matched to that pair without changing its length. So `lcs[i−1][j−1] + 1` is optimal.
</details>

<details><summary>3. Why can't Longest Palindromic Subsequence be solved by expanding around centres?</summary>

A subsequence may skip characters. Expanding from the centre only finds contiguous palindromes.
</details>

<details><summary>4. In regex matching, why is the "one more copy" case match[i−1][j] rather than match[i−1][j−2]?</summary>

`x*` may match further copies, so it has to stay available. Consuming it would allow only one copy.
</details>

<details><summary>5. Why does Distinct Subsequences roll its array downward over j?</summary>

Each character of s may be used at most once per subsequence, so the update must read the value from before this character, exactly as in the 0/1 knapsack.
</details>

---

## 12. Exit gate

Move on when you can:

* prove the `tails` invariant and write the O(n log n) LIS (using the Ch 10 template) from a blank file;
* write LCS with reconstruction, and Edit Distance, from blank files, each in under 10 minutes;
* solve Longest Palindromic Substring by expansion, and Longest Palindromic Subsequence with the interval table, and say which one applies when;
* derive the regex transitions, including the empty-string base row;
* solve 8 of the 10 practice problems.
