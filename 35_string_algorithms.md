# Chapter 35 — String Algorithms

> **Part VII · Week 32** · **S-tier** · Prerequisites: Ch 4 (String costs), Ch 5 (strings), Ch 7 (hashing), Ch 10 (binary search on the answer), Ch 18 §4 (modular arithmetic)
> **Problems:** 7 worked (1 ★) · 7 practice · 2 stretch · **Time:** about 9 hours

Three tools, each answering a question the naive approach answers in O(n·m) or worse:
* the **prefix function**: where does a pattern occur, and how does a string overlap itself?
* the **Z-function**: how far does each suffix agree with the start of the string?
* **rolling hashes**: are these two substrings equal, in O(1)?

Most interview string problems are Ch 5, 7, 11 and 29 material. This chapter is for the ones that aren't: n up to 10⁵ with matching, repetition or substring equality at their core.

---

## 0. From zero: what the naive search wastes

**The problem.** Find the first occurrence of pattern p (length m) in text t (length n). The naive search tries every start and compares character by character: O(n·m). On `t = "aaaa…a"` and `p = "aa…ab"`, every start matches m − 1 characters before failing, so it really costs n·m.

**The waste.** After matching `p[0..k)` and failing at the next character, the naive search moves the start by one and **re-reads characters it has already seen**. But it already knows those characters: they are `p[0..k)`. So the question "where could the next match start?" depends only on the **pattern**, and can be precomputed.

**Borders.** A **border** of a string is a proper prefix that is also a suffix. `"abab"` has the borders `"ab"` and `""`. `"aabaa"` has `"aa"`, `"a"` and `""`. After matching `p[0..k)`, the next possible match start keeps the longest border of `p[0..k)` already matched. Any start in between would need a longer border to exist.

---

## 1. Attempt first ★

> **[Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) (30 minutes).** Return the first index where `needle` occurs in `haystack`, or −1. LeetCode's limits (≤ 10⁴) let the naive search pass; **design for n, m = 10⁶**. Start by computing, by hand, the longest border of every prefix of `"aabaaab"`.

<details><summary>The table to check against</summary>

| i | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| :-- | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| s[i] | a | a | b | a | a | a | b |
| longest border of s[0..i] | 0 | 1 | 0 | 1 | 2 | 2 | 3 |
</details>

---

## 2. The prefix function and KMP

### 2.1 Computing it in O(n)

> `pi[i]` = the length of the longest border of `s[0..i]` (the prefix ending at i).

**Key property: the borders of a string are a chain.** A border of a border is again a border: it's a prefix of a prefix and a suffix of a suffix. And every border of `s[0..i]` other than the longest is a border of the longest one. So **all** borders of `s[0..i]` are `pi[i]`, `pi[pi[i] − 1]`, `pi[pi[pi[i] − 1] − 1]`, …, down to 0.

**Extending by one character.** A border of `s[0..i]` of length k + 1 is a border of `s[0..i−1]` of length k, extended by `s[i] == s[k]`. So try the borders of `s[0..i−1]` from the longest, following the chain, and stop at the first k with `s[k] == s[i]`:

```java
static int[] prefixFunction(String s) {
    int n = s.length();
    int[] pi = new int[n];
    for (int i = 1; i < n; i++) {
        int k = pi[i - 1];                                  // the longest border of s[0..i-1]
        while (k > 0 && s.charAt(i) != s.charAt(k)) k = pi[k - 1];   // next shorter border
        if (s.charAt(i) == s.charAt(k)) k++;
        pi[i] = k;
    }
    return pi;
}
```

**Why O(n), although there's a loop inside a loop (amortised, Ch 3).** k goes up by at most 1 per i, and every iteration of the `while` loop lowers it by at least 1. It never goes below 0, so the total number of decreases is at most the total number of increases, ≤ n.

**What a fallback looks like in a search.** Pattern `ababc` has pi = `[0, 0, 1, 2, 0]`. Searching the text `abababc`:

```text
text:      a b a b a b c
pattern:   a b a b c                matched "abab", then text[4] = 'a' ≠ 'c'
                                    "abab" has longest border "ab" (pi[3] = 2)
pattern:       a b a b c            slide so that border lines up: k = 2. The text pointer stays put;
                                    text[4..6] = "abc" then matches p[2..4]: found at index 2
```

The naive search would restart at text index 1 and re-read characters. KMP never moves backwards in the text.

### Worked K1 · [Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) ★

**KMP (Knuth–Morris–Pratt)** runs the same loop over the **text**, with k = how much of the pattern currently matches. On a mismatch, fall back along the pattern's border chain instead of moving back in the text:

```java
public int strStr(String haystack, String needle) {
    int[] pi = prefixFunction(needle);
    int k = 0;                                               // needle[0..k) matches the text ending here
    for (int i = 0; i < haystack.length(); i++) {
        while (k > 0 && haystack.charAt(i) != needle.charAt(k)) k = pi[k - 1];
        if (haystack.charAt(i) == needle.charAt(k)) k++;
        if (k == needle.length()) return i - k + 1;
    }
    return -1;
}
```

**O(n + m)**, by the same amortised argument. **For all occurrences** (CSES String Matching, practice), don't return; count the match and set `k = pi[k − 1]` to continue. **Alternative, in prose:** compute the prefix function of `needle + '#' + haystack`. A position with value m ends a match, provided `#` appears in neither string.

---

## 3. Borders and periods

**Period.** p is a period of s if `s[i] == s[i + p]` for every valid i. **p is a period exactly when s has a border of length n − p**: both say that the prefix and the suffix of length n − p are equal. So the **smallest period is n − pi[n − 1]**.

### Worked K2a · [Repeated Substring Pattern](https://leetcode.com/problems/repeated-substring-pattern/)

> Can s be written as some string repeated two or more times?

s = t repeated n / |t| times exactly when |t| is a period that divides n. **Claim: s is a repetition iff its smallest period p = n − pi[n − 1] divides n and p < n.**

```java
public boolean repeatedSubstringPattern(String s) {
    int n = s.length(), p = n - prefixFunction(s)[n - 1];
    return p < n && n % p == 0;
}
```

<details><summary>Why checking only the smallest period is enough</summary>

"If" is immediate. For "only if": suppose s = t repeated, with |t| = d, d | n, d ≤ n / 2. Then p ≤ d, so p + d ≤ n. The **periodicity lemma** (Fine and Wilf) says that if p and d are both periods and p + d − gcd(p, d) ≤ n, then gcd(p, d) is a period too. It's ≤ p, and p is the smallest, so gcd(p, d) = p. That means p divides d, which divides n.
</details>

**Alternative, in prose:** s is a repetition iff s occurs in `(s + s)` at some index in 1..n − 1. Check with `(s + s).indexOf(s, 1) < n`, or with KMP.

### Worked K2b · [Longest Happy Prefix](https://leetcode.com/problems/longest-happy-prefix/)

> Return the longest proper prefix of s that is also a suffix.

That's the longest border, by definition: `s.substring(0, prefixFunction(s)[n - 1])`. **All** the borders, in decreasing order, come from following the chain (CSES Finding Borders, practice).

### Worked K3 · [Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/)

> Add the fewest characters to the **front** of s to make it a palindrome. n ≤ 5·10⁴.

**Reduction.** Whatever you add, the result's prefix must mirror its suffix. The best you can do is keep the **longest palindromic prefix** `s[0..L)` in the middle and put the reverse of the rest in front: `reverse(s[L..]) + s`.

**Finding L with the prefix function.** A prefix of s that equals the suffix of `rev = reverse(s)` of the same length is a prefix equal to its own reverse, which is a palindrome. So L is the longest border of `s + '#' + rev`. The `#` stops a border from running past the middle, so L ≤ n.

```java
public String shortestPalindrome(String s) {
    String rev = new StringBuilder(s).reverse().toString();
    int[] pi = prefixFunction(s + "#" + rev);
    int keep = pi[pi.length - 1];                       // the longest palindromic prefix
    return new StringBuilder(s.substring(keep)).reverse() + s;
}
```

O(n). The naive check of every prefix for being a palindrome is O(n²), 2.5·10⁹ at the limit. **The general trick:** to compare one string's prefixes with another string's suffixes, run the prefix function on `first + separator + second`.

---

## 4. The Z-function

> `z[i]` = the length of the longest common prefix of s and `s[i..]`. By convention, `z[0] = n`.

**Computing it in O(n).** Keep the **Z-box** `[l, r)`: the match `s[l..r) == s[0..r − l)` that reaches furthest right. For a new i inside the box, `s[i..r)` is a copy of `s[i − l..r − l)`, so `z[i]` is at least `min(r − i, z[i − l])`. Start from there and extend by direct comparison. On `s = aabxaab`:

```text
index:   0 1 2 3 4 5 6
s:       a a b x a a b
z:       7 1 0 0 3 1 0

i = 4: compare directly: "aab" matches s[0..3), then s[7] is past the end  → z[4] = 3, box [l, r) = [4, 7)
i = 5: inside the box, s[5..7) is a copy of s[1..3), so z[5] ≥ min(r − i, z[1]) = min(2, 1) = 1;
       one comparison (s[1] = 'a' vs s[6] = 'b') fails               → z[5] = 1, no extension
```

The code:

```java
static int[] zFunction(String s) {
    int n = s.length();
    int[] z = new int[n];
    z[0] = n;
    for (int i = 1, l = 0, r = 0; i < n; i++) {
        if (i < r) z[i] = Math.min(r - i, z[i - l]);         // reuse the copy inside the box
        while (i + z[i] < n && s.charAt(z[i]) == s.charAt(i + z[i])) z[i]++;
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}
```

**Why O(n):** each successful comparison in the `while` loop moves r to the right. If `z[i − l] < r − i`, the first comparison fails at once (the copy says so). Otherwise the comparisons start at r. And r never exceeds n.

**Prefix function or Z?** They carry the same information and convert into each other. Choose by how the question is phrased. "Borders, the next state after a mismatch, streaming text" → the prefix function. "How far does the suffix at i agree with the start" → Z.

### Worked K4 · [Sum of Scores of Built Strings](https://leetcode.com/problems/sum-of-scores-of-built-strings/)

> s is built by prepending characters one at a time, so the strings built along the way are its **suffixes**. The score of each is its longest common prefix with s. Return the total score. n ≤ 10⁵.

The score of the suffix starting at i is z[i], by definition, so the answer is Σ z[i] with z[0] = n. **Use `long`**: for `"aaa…a"` the sum is n(n + 1) / 2 ≈ 5·10⁹.

```java
public long sumScores(String s) {
    long total = 0;
    for (int v : zFunction(s)) total += v;
    return total;
}
```

**Recognising it** is the whole problem. "The LCP of every suffix with the whole string" is the Z-function's definition. Its uses: all periods (`i + z[i] == n` means i is a period, CSES Finding Periods), pattern matching (`z` of `p + '#' + t`), and Minimum Time to Revert Word (practice).

---

## 5. Rolling hashes

**The idea.** Map each string to a number so that equal strings get equal numbers, and the number for a **substring** can be computed in O(1) after O(n) preparation. Comparing two substrings then costs O(1), not O(length).

**The polynomial hash.** Treat the string as a number in base B, modulo a prime M:

> `H(s) = (s[0]·B^(k−1) + s[1]·B^(k−2) + … + s[k−1]) mod M`

**Prefix hashes** `h[i]` = H(s[0..i)) satisfy `h[i + 1] = h[i]·B + s[i]`, like reading digits. Then the hash of `s[l..r)` is `h[r] − h[l]·B^(r−l)` (mod M). It's Ch 8's prefix-difference identity, with the shift by `B^(r−l)` lining up the place values.

**Collisions: this is the part interviewers probe.** Different strings can share a hash. With M ≈ 10⁹ and N substrings compared, the expected number of colliding pairs is about N² / (2M) (the birthday bound). At N = 10⁵ that's **about 5 colliding pairs**: a single 10⁹ modulus is **not** safe. Two defences, used together:
* **Two independent moduli** (double hashing): M ≈ 10¹⁸ in effect, so N² / M is about 10⁻⁸.
* **A random base**, chosen at run time, so no fixed test input can be built to collide.

When an exact answer is cheap to confirm, also **verify a hash match** by comparing the actual substrings.

```java
static final class Hasher {
    static final long M1 = 1_000_000_007L, M2 = 998_244_353L;
    static final long B = 911 + new Random().nextInt(100_000);       // random base, fixed per run
    final long[] h1, h2, p1, p2;                                     // prefix hashes and powers of B

    Hasher(String s) {
        int n = s.length();
        h1 = new long[n + 1]; h2 = new long[n + 1]; p1 = new long[n + 1]; p2 = new long[n + 1];
        p1[0] = p2[0] = 1;
        for (int i = 0; i < n; i++) {
            h1[i + 1] = (h1[i] * B + s.charAt(i)) % M1;
            h2[i + 1] = (h2[i] * B + s.charAt(i)) % M2;
            p1[i + 1] = p1[i] * B % M1;
            p2[i + 1] = p2[i] * B % M2;
        }
    }

    long get(int l, int r) {                                          // both hashes of s[l, r), packed in one long
        long a = ((h1[r] - h1[l] * p1[r - l]) % M1 + M1) % M1;
        long b = ((h2[r] - h2[l] * p2[r - l]) % M2 + M2) % M2;
        return a * M2 + b;
    }
}
```

**Overflow check (Ch 18 §4):** every product is of two values below about 10⁹, so it stays below 10¹⁸ < 9.2·10¹⁸. `a * M2 + b` is below 10¹⁸ too, and packs the pair into one `long` key for a `HashSet`.

### Worked K5a · [Repeated DNA Sequences](https://leetcode.com/problems/repeated-dna-sequences/)

> Return every 10-letter substring (over `ACGT`) that occurs more than once. n ≤ 10⁵.

The length is fixed and the alphabet has 4 letters, so each window has an **exact** 20-bit code: 2 bits per letter. That's a rolling hash with **no collisions**. Slide the window by shifting in 2 bits and masking off the oldest 2:

```java
public List<String> findRepeatedDnaSequences(String s) {
    int[] code = new int[128];
    code['C'] = 1; code['G'] = 2; code['T'] = 3;                     // 'A' stays 0
    Set<Integer> seen = new HashSet<>(), reported = new HashSet<>();
    List<String> answer = new ArrayList<>();
    int window = 0, mask = (1 << 20) - 1;
    for (int i = 0; i < s.length(); i++) {
        window = ((window << 2) | code[s.charAt(i)]) & mask;         // drop the letter that left
        if (i >= 9 && !seen.add(window) && reported.add(window)) answer.add(s.substring(i - 9, i + 1));
    }
    return answer;
}
```

O(n). **Alternative, in prose:** put each `substring(i, i + 10)` in a HashSet. That's O(10·n), and fine here. The rolling version is the one that scales to long windows.

### Worked K5b · [Longest Duplicate Substring](https://leetcode.com/problems/longest-duplicate-substring/)

> Return any longest substring that occurs at least twice (the occurrences may overlap). n ≤ 3·10⁴.

**Monotone in the length** (Ch 10): if some substring of length L occurs twice, so does its prefix of length L − 1. So binary search for the largest L that has a duplicate. **The check** for one L puts the hash of every length-L window in a set: O(n). In total, O(n log n).

```java
public String longestDupSubstring(String s) {
    int n = s.length(), lo = 0, hi = n, bestStart = 0;   // Ch 10, mirrored: length lo has a duplicate (0 trivially),
    Hasher h = new Hasher(s);                             // length hi = n doesn't (the whole string occurs once)
    while (hi - lo > 1) {
        int mid = lo + (hi - lo) / 2;
        int start = duplicateStart(h, n, mid);
        if (start >= 0) { lo = mid; bestStart = start; } else hi = mid;
    }
    return s.substring(bestStart, bestStart + lo);
}

private int duplicateStart(Hasher h, int n, int len) {      // the start of a repeated window of this length, or -1
    Set<Long> seen = new HashSet<>();
    for (int i = 0; i + len <= n; i++) if (!seen.add(h.get(i, i + len))) return i;
    return -1;
}
```

**Why `bestStart` matches the final `lo`:** `lo` changes only on a success, and `bestStart` is set at the same moment. **Why double hashing matters here:** each check compares up to 3·10⁴ windows. With one 10⁹ modulus, a false "duplicate" in some check is a real risk, and a false success would return a wrong string. **The exact alternative** (a suffix array with an LCP array) is out of scope (Ch 37).

---

## 6. Where these break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| **Many** patterns searched at once | KMP per pattern is O(total · n) | A trie of the patterns (Ch 20), checked from each start, if they're short. Aho–Corasick (KMP on a trie) is out of scope (Ch 37) |
| Questions about **all distinct substrings**, or the k-th one | Hashing gives equality, not order | A suffix array or automaton: out of scope (Ch 37) |
| Palindromes in O(n) | Expanding around centres is O(n²) | Manacher: recognition only (Ch 29) |
| Comparing substrings **lexicographically** | A hash doesn't give order | Binary search on the LCP using hashes (O(log n) per comparison) |
| A single small modulus, adversarial tests | Collisions | Two moduli + a random base, or verify matches |

---

## 7. Recognition

**Signals:**
* Pattern search with n·m too large: KMP or Z.
* "A prefix that is also a suffix", "the smallest repeating unit", "made of copies of": borders and periods.
* "The shortest addition to make a palindrome", "the longest palindromic prefix": the prefix function on `s + '#' + reverse(s)`.
* "The LCP of each suffix with the whole string", "after removing k characters, is the rest a prefix?": Z.
* "Compare many substrings for equality", "the longest repeated / common substring", "count distinct substrings of a kind": rolling hash, often with binary search on the length.

**False friends:**
* Longest common **subsequence**: DP (Ch 29), not string matching.
* Anagrams and permutations in a window: letter counts (Ch 7, Ch 11).
* n ≤ 1000: often an O(n²) DP or a direct check is simpler and exact.
* `String.indexOf` and `contains` are fine for one search in an interview, *if* you can state their worst case (O(n·m)) when asked.

---

## 8. Implementation traps

| Trap | Fix |
| :--- | :--- |
| `pi[k]` instead of `pi[k − 1]` when falling back | The border of the matched part `p[0..k)` is `pi[k − 1]` |
| A separator that can appear in the input | Choose a character outside the alphabet (`#`, `\u0000`) |
| Negative values after subtracting hashes | `((x % M) + M) % M` |
| A modulus above about 3·10⁹ | Products overflow `long`; stay near 10⁹ per modulus |
| `substring` inside a loop | It copies, O(length) each time (Ch 4): compare hashes or indices instead |
| Forgetting `z[0]` | Set `z[0] = n` explicitly if the problem counts the whole string |
| Summing Z values in `int` | Up to n(n + 1) / 2: `long` |

---

## 9. Practice

1. **[Rotate String](https://leetcode.com/problems/rotate-string/)** *(Easy)*
2. **[Repeated String Match](https://leetcode.com/problems/repeated-string-match/)** *(Medium)*
3. **[CSES String Matching](https://cses.fi/problemset/task/1753)** *(Easy; stdin/stdout)*
4. **[CSES Finding Borders](https://cses.fi/problemset/task/1732)** *(Easy; stdin/stdout)*
5. **[CSES Finding Periods](https://cses.fi/problemset/task/1733)** *(Medium; stdin/stdout)*
6. **[Minimum Time to Revert Word to Initial State II](https://leetcode.com/problems/minimum-time-to-revert-word-to-initial-state-ii/)** *(Hard)*
7. **[Distinct Echo Substrings](https://leetcode.com/problems/distinct-echo-substrings/)** *(Hard)*

**Stretch:** **[Longest Common Subpath](https://leetcode.com/problems/longest-common-subpath/)** *(Hard)* · **[Count Prefix and Suffix Pairs II](https://leetcode.com/problems/count-prefix-and-suffix-pairs-ii/)** *(Hard)*

<details><summary>Nudges</summary>

1. Rotate String: every rotation of s is a substring of s + s. Check the lengths match, then search.
2. Repeated String Match: b can only fit in ⌈|b| / |a|⌉ or one more copies of a; try both with KMP.
3. String Matching: K1 counting every match, continuing with `k = pi[k − 1]`.
4. Finding Borders: follow the chain from `pi[n − 1]` and print the lengths in increasing order.
5. Finding Periods: p is a period iff `p + z[p] == n` (the suffix at p agrees with the start until the end), and n itself always is one.
6. Revert Word: after t seconds, the part of the original still in place is the suffix starting at t·k, and it must be a prefix of the word: `z[t·k] ≥ n − t·k`. Take the first such t; otherwise ⌈n / k⌉.
7. Echo Substrings: n ≤ 2000, so O(n²) pairs (start, half-length) are fine. Compare the two halves by hash in O(1), and put the hash of each echo in a set to count distinct ones.
8. Stretch, Common Subpath: binary search the length. For a length L, collect the set of hashes of length-L windows in each path, and intersect the sets. The values are city numbers up to 10⁵, so hash ints, not chars. Double hashing is essential here.
9. Stretch, Prefix-Suffix Pairs: insert each word into a trie keyed by the **pair** (`w[k]`, `w[len − 1 − k]`). Walking word j's pairs passes through exactly the earlier words that are both its prefix and its suffix: count the words ending at each node on the way.
</details>

<details><summary>Key</summary>

Rotate → K1 on s + s · Repeated Match → K1 with a bounded repeat count · String Matching → K1 / K4 · Borders → K2 chain · Periods → K4 (or K2) · Revert Word → K4 at multiples of k · Echo → K5 or LCP table · Common Subpath → K5 + binary search across paths · Prefix-Suffix Pairs → K4 + trie of (prefix, suffix) pairs
</details>

---

## 10. Self-test

<details><summary>1. Why are all the borders of a string found by following pi from the longest one?</summary>

Any border shorter than the longest is a prefix and a suffix of the longest border, so it's a border of it. Repeating the argument gives the whole chain.
</details>

<details><summary>2. Why is the prefix-function loop O(n) despite its inner while loop?</summary>

k rises by at most 1 per step and each while iteration lowers it, never below 0, so the total number of while iterations is at most n.
</details>

<details><summary>3. What is the smallest period of s in terms of pi, and why?</summary>

n − pi[n − 1]: p is a period iff s has a border of length n − p, so the longest border gives the smallest period.
</details>

<details><summary>4. In Shortest Palindrome, what does the '#' prevent?</summary>

Without it, a border of s + reverse(s) could be longer than s, mixing characters from both halves. With it, the border can't cross the separator, so its length is at most n.
</details>

<details><summary>5. Why is a single 10⁹ modulus unsafe for 10⁵ substrings?</summary>

By the birthday bound, about N² / (2M) ≈ 5 pairs of different substrings would share a hash. Two moduli push the effective M to about 10¹⁸.
</details>

<details><summary>6. Why can Longest Duplicate Substring binary search on the length?</summary>

A repeated substring of length L has a repeated prefix of length L − 1, so "a duplicate of length L exists" is monotone in L.
</details>

---

## 11. Exit gate

Move on when you can:

* write the prefix function and KMP from blank files in 6 minutes, and prove the O(n) bound;
* derive the smallest period from pi and use the `s + '#' + t` trick;
* write the Z-function and explain the Z-box;
* write a double rolling hash, and explain the collision risk with numbers;
* solve 5 of the 7 practice problems.
