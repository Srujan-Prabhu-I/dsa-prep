# Chapter 7 — Hashing: Remembering the Past

> **Part II · Week 5** · Prerequisites: Ch 1–6
> **Problems:** 6 worked (1 ★) · 7 practice · 2 stretch · **Time:** about 7 hours

Ch 1 showed the most common bottleneck of all: **re-scanning**. For each element, you scan everything before it looking for a match. A hash table removes that scan. It remembers what you've seen and answers "have I seen *this*?", "how many times?" or "where?" in constant expected time. That one ability turns a large share of O(n²) brute forces into O(n) solutions.

This chapter first builds a hash table from scratch, so you know exactly what "O(1) expected" means and when it fails. Then it covers the five ways problems use one.

---

## 0. From zero: how a hash table works

### 0.1 The problem it solves

**The interface first.** A **map** (also called a dictionary) stores **key → value** pairs with unique keys and supports three operations: `put(key, value)` (insert, or overwrite an existing key), `get(key)` (the value, or "absent"), and `remove(key)`. A **set** is a map with no values: `add`, `contains`, `remove`. Neither promises any order. That's a *contract*. A hash table is one way to implement it; a balanced search tree (`TreeMap`, Ch 15) is another, with different costs and an ordering guarantee.

Suppose the keys are small integers, say 0 to 10⁶. Then an array does everything: `seen[x] = true`, `count[x]++`. Each operation is O(1), because the key *is* the index. This is **direct addressing**, and you've already used it as `int[26]` letter counts (Ch 4 §6). Whenever the key range allows, it's the fastest option.

But keys are often too spread out (any `int` up to 10⁹), or aren't numbers at all (strings, pairs). An array indexed by key would be far too large.

### 0.2 The idea: squeeze keys into a small array

Keep an array of `m` **buckets**, and use a **hash function** to turn any key into a bucket number between 0 and m − 1:

> bucket(key) = hash(key) mod m

Two different keys can land in the same bucket, a **collision**. That's unavoidable: with more possible keys than buckets, some must share (the pigeonhole principle). So each bucket holds a **small list** of the entries that hashed there. To find a key, compute its bucket and scan only that short list. This is called *chaining*.

**A trace.** Take m = 5 buckets and `hash(x) = x` for integers, so `bucket(x) = x mod 5`. Insert the keys 12, 7, 20, 17, 3:

```text
put 12 → 12 mod 5 = 2        put 7 → 7 mod 5 = 2 (collision: appended to bucket 2's chain)
put 20 → 20 mod 5 = 0        put 17 → 17 mod 5 = 2        put 3 → 3 mod 5 = 3

bucket 0 │ 20
bucket 1 │ (empty)
bucket 2 │ 12 → 7 → 17          get(17): go straight to bucket 2, then compare 12, 7, 17: found
bucket 3 │ 3                    get(8):  8 mod 5 = 3, compare 3: not equal, so absent
bucket 4 │ (empty)
```

`get` never looks at buckets 0, 1, 3 or 4 when the key belongs in bucket 2. The work is the length of **one** chain, not the number of keys. Bucket 2 is unusually long here because this "hash" is poor for these keys, which is exactly the failure described below.

**Why it's fast.** If the hash function spreads keys evenly, then n keys in m buckets give an average of n/m keys per bucket. That ratio is the **load factor** α. Keep α below a constant (Java uses 0.75), and every operation scans a constant-length list on average: **O(1) expected**.

**Keeping α small as the table fills:** when the size passes 0.75 m, allocate a larger bucket array and **re-insert everything** (a *rehash*), because every key's bucket changes when m changes. This is the dynamic-array growth trick from Ch 5 §0.2 again. The table roughly doubles each time, so the total rehashing work is O(n), and each insertion is amortised O(1). Growing the table above from 5 to 10 buckets moves keys to new places:

```text
m = 5:   bucket 2 │ 12 → 7 → 17          m = 10:  bucket 2 │ 12
                                                  bucket 7 │ 7 → 17
         bucket 0 │ 20                            bucket 0 │ 20
         bucket 3 │ 3                             bucket 3 │ 3
```

**When it isn't fast.** If many keys hash to the same bucket, that bucket's list gets long, and operations degrade towards O(n). That happens with a poor hash function, or with inputs deliberately built to collide. Java's `HashMap` limits the damage: a bucket that grows past 8 entries is converted into a balanced tree (Ch 15), so the worst case is O(log n). In interviews, say **"O(1) expected"** for hash operations.

### 0.3 What Java needs from your keys

Java computes `key.hashCode()` to choose the bucket and `key.equals(other)` to find the match inside it. That only works if **equal keys always produce equal hash codes**, and `Integer`, `String` and records all guarantee it. That's why Ch 4 §15 warned:
* a class that overrides `equals` but not `hashCode` puts equal keys in different buckets, so lookups silently fail;
* arrays (`int[]`) use identity, so two arrays with equal contents are different keys;
* changing a key *after* inserting it strands it in the wrong bucket.

### 0.4 Common misconceptions

| Belief | Reality |
| :--- | :--- |
| "Hash operations are O(1), period" | O(1) **expected**, given a good hash and a bounded load factor; many collisions degrade them |
| "A HashMap keeps insertion order" | No order at all. `LinkedHashMap` keeps insertion order; `TreeMap` keeps sorted order (Ch 4 §23, Ch 15) |
| "Iterating over a HashMap is O(size)" | O(size + number of buckets); fine in practice, but a map that held many keys and then shrank still pays for its buckets |
| "An `int[26]` count array is a different technique" | It's a hash table whose hash is `c − 'a'`, with no collisions: direct addressing |
| "Two `int[]` with the same contents are the same key" | Arrays compare by identity; use a `String`, a `List<Integer>` or a record as the key |

### Worked P0 · [Design HashMap](https://leetcode.com/problems/design-hashmap/)

> Implement `put(key, value)`, `get(key)` (return −1 if absent) and `remove(key)` for integer keys, **without** using any built-in hash table.

A plain `int[1_000_001]` would pass, since LeetCode keeps keys within 10⁶. But the point is to build the real thing: buckets, chains, a load factor and resizing. Each bucket is an `ArrayList` of `{key, value}` pairs. (Java's own `HashMap` uses linked nodes for its chains; you'll meet those in Ch 13.)

```java
class MyHashMap {
    private List<List<int[]>> buckets = newBuckets(17);   // each entry is {key, value}
    private int size = 0;

    private static List<List<int[]>> newBuckets(int capacity) {
        List<List<int[]>> b = new ArrayList<>(capacity);
        for (int i = 0; i < capacity; i++) b.add(new ArrayList<>());
        return b;
    }

    // The bucket a key belongs to. For an object key this would use key.hashCode().
    private List<int[]> bucketFor(int key) {
        return buckets.get(Math.floorMod(Integer.hashCode(key), buckets.size()));
    }

    public void put(int key, int value) {
        List<int[]> bucket = bucketFor(key);
        for (int[] entry : bucket) {
            if (entry[0] == key) { entry[1] = value; return; }       // existing key: overwrite
        }
        bucket.add(new int[] {key, value});
        size++;
        if (size > buckets.size() * 3 / 4) resize();                // keep the load factor ≤ 0.75
    }

    public int get(int key) {
        for (int[] entry : bucketFor(key)) {
            if (entry[0] == key) return entry[1];
        }
        return -1;
    }

    public void remove(int key) {
        List<int[]> bucket = bucketFor(key);
        for (int i = 0; i < bucket.size(); i++) {
            if (bucket.get(i)[0] == key) {
                bucket.set(i, bucket.get(bucket.size() - 1));       // move the last entry into the gap,
                bucket.remove(bucket.size() - 1);                   // then drop the last slot: O(1)
                size--;
                return;
            }
        }
    }

    private void resize() {
        List<List<int[]>> old = buckets;
        buckets = newBuckets(old.size() * 2 + 1);   // odd sizes spread patterned keys (16, 32, 48, …)
        for (List<int[]> bucket : old) {
            for (int[] entry : bucket) bucketFor(entry[0]).add(entry);   // every key gets a new bucket
        }
    }
}
```

Three design points, each a reason the operations stay O(1):
* **`Math.floorMod`**, not `%`, because a negative key's `%` is negative (Ch 4 §10), which isn't a valid index.
* **Odd bucket counts.** With m = 16, the keys 16, 32, 48, … all have remainder 0 and share one bucket. An odd m spreads them out. (Java's `HashMap` uses powers of two and mixes the hash code's bits instead; the goal is the same.)
* **Swap-with-last removal.** Order inside a bucket doesn't matter, so deletion needs no shifting. You'll use the same trick in Ch 16 for a set with random access.

Verify it like `IntList`: 100 000 random operations against `java.util.HashMap`, compared after every step.

---

## 1. Remember the past: complement lookup

### Worked P1 · [Two Sum](https://leetcode.com/problems/two-sum/)

> Given `nums` and `target`, return indices `i ≠ j` with `nums[i] + nums[j] = target`. Exactly one answer exists. (`n ≤ 10⁴`, values and target in `[−10⁹, 10⁹]`.)

**Brute force:** every pair, O(n²). **Bottleneck:** for each `j`, we re-scan the earlier elements looking for **one specific value**, `target − nums[j]`. Ch 1, question 3: *what must I remember about the past?* Just the values seen so far, each with an index, in a structure that answers "is this value present?" instantly.

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> indexOf = new HashMap<>();   // value -> an index where it appears, in nums[0..i)
    for (int i = 0; i < nums.length; i++) {
        Integer j = indexOf.get(target - nums[i]);    // the complement, if we've seen it
        if (j != null) return new int[] {j, i};
        indexOf.put(nums[i], i);                       // only now does nums[i] join the past
    }
    return new int[0];                                 // unreachable: an answer is guaranteed
}
```

* **Query before insert,** exactly as in Ch 1 Derivation A. If you insert first, `nums[i]` can pair with itself (target 6, value 3).
* **Duplicates work:** for `[3, 3]`, target 6, the first 3 is stored, and the second finds it.
* **Overflow check:** `target − nums[i]` lies in `[−2·10⁹, 2·10⁹]`, which fits in `int` (limit ±2.147·10⁹), barely. If the bounds were larger, use `long`.

**O(n) expected time, O(n) space.** An alternative is to sort and walk two pointers inwards (Ch 11): O(n log n) time and O(1) extra space, but you lose the original indices unless you sort (value, index) pairs.

---

## 2. Group equivalent things: canonical keys

### Worked P2 · [Group Anagrams](https://leetcode.com/problems/group-anagrams/)

> Group strings that are anagrams of each other (the same letters, rearranged). `["eat","tea","tan","ate","nat","bat"]` → `[["eat","tea","ate"], ["tan","nat"], ["bat"]]`.

Comparing every pair is O(n² · k) for n strings of length k. Better: compute a **canonical key** for each string, a value that is *the same for all anagrams and different for non-anagrams*, then group by key in a map.

**Key 1: the sorted string.** All anagrams sort to the same letters: `"eat" → "aet"`. O(k log k) per string.

```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        groups.computeIfAbsent(new String(chars), k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(groups.values());
}
```

**Key 2: the letter counts.** O(k) per string, faster when strings are long:

```java
private static String signature(String s) {
    int[] count = new int[26];
    for (int i = 0; i < s.length(); i++) count[s.charAt(i) - 'a']++;
    StringBuilder key = new StringBuilder();
    for (int c : count) key.append(c).append('#');   // '#' separates the counts
    return key.toString();
}
```

**A key must work in both directions.** Equivalent objects must get *equal* keys, and non-equivalent objects must get *different* keys. Drop the `'#'` separator and the second direction fails. "1 a, 11 b's" produces `"111000…"`, and so does "11 a's, 1 b". Two strings that aren't anagrams now share a key. **Every time you invent a key, check both directions.**

Other canonical keys you'll meet:

| Equivalence | Canonical key |
| :--- | :--- |
| Anagrams | Sorted string, or letter counts with separators |
| Strings with the same pattern (`"egg"` ~ `"add"`) | Each character replaced by the index where it first appeared: `[0, 1, 1]` |
| Points on the same line through a fixed point | The direction `(dy, dx)` reduced by its gcd, with a fixed sign |
| Rows/columns equal after shifting by a constant | The differences between consecutive elements |
| A pair `(a, b)` where order doesn't matter | `(min, max)` |

---

## 3. Fast membership: start-of-run detection

### Worked P3 · [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) ★

> **Attempt first (25 minutes).** Given an unsorted array, return the length of the longest run of consecutive integers that all appear in it, **in O(n) time**. `[100, 4, 200, 1, 3, 2]` → 4 (the run 1, 2, 3, 4).

**Sorting** solves it in O(n log n): sort, then scan for runs. The O(n) requirement rules that out, so ask the Ch 1 questions. *Is the order of the input important?* No. *What do we need to know?* Only **which values are present**, so put them in a `HashSet`.

Now, from each value `x`, walk upwards (`x + 1`, `x + 2`, …) while values are present. Done naively, that's O(n²): for the run 1…n, you'd walk from 1, then again from 2, then from 3, ….

**The fix is to walk only from the *start* of a run.** A value `x` starts a run exactly when `x − 1` is absent. Each run then gets walked once, from its start, so the total walking over all runs is at most n steps.

```java
public int longestConsecutive(int[] nums) {
    Set<Integer> present = new HashSet<>();
    for (int x : nums) present.add(x);
    int best = 0;
    for (int x : present) {                       // iterate the SET, so duplicates are already gone
        if (present.contains(x - 1)) continue;    // x is not the start of a run
        int length = 1;
        while (present.contains(x + length)) length++;
        best = Math.max(best, length);
    }
    return best;
}
```

**Why O(n):** each value is the starting point for at most one walk, and each value is stepped over by at most one walk (the one for its own run). That's the amortisation argument from Ch 3 §3.

> **Trap: iterating over `nums` instead of `present`.** With input `[1, 1, 1, …, 1, 2, 3, …, k]`, every one of the duplicate 1s is "the start of a run" and walks the whole run again, which is O(n²). Iterating over the set removes the duplicates first.

The values are within ±10⁹ here, so `x − 1` and `x + length` can't overflow. If keys could be `Integer.MIN_VALUE`, `x − 1` would wrap to `MAX_VALUE`.

---

## 4. When the values are the indices: the array as its own hash table

### Worked P4 · [First Missing Positive](https://leetcode.com/problems/first-missing-positive/)

> Return the smallest positive integer missing from an unsorted array, in **O(n) time and O(1) extra space**. `[3, 4, −1, 1]` → 2; `[7, 8, 9]` → 1.

A `HashSet` gives O(n) time but uses O(n) space. To get O(1) space, find the structure first.

**Observation:** with n elements, the answer is always in `[1, n + 1]`. At best the array holds exactly 1…n, making the answer n + 1; any gap makes it smaller. So only the values 1…n matter, and **there are exactly n slots to record them in: the array itself**. Put value `v` in its "home" slot, index `v − 1`. Afterwards, the first slot `i` that doesn't hold `i + 1` gives the answer.

```java
public int firstMissingPositive(int[] nums) {
    int n = nums.length;
    for (int i = 0; i < n; i++) {
        // While nums[i] is a value in [1, n] whose home slot doesn't hold it yet, swap it home.
        // The "home doesn't hold it" check also stops the loop on duplicates.
        while (nums[i] >= 1 && nums[i] <= n && nums[nums[i] - 1] != nums[i]) {
            int v = nums[i];
            nums[i] = nums[v - 1];
            nums[v - 1] = v;
        }
    }
    for (int i = 0; i < n; i++) {
        if (nums[i] != i + 1) return i + 1;
    }
    return n + 1;
}
```

**Why the nested `while` is still O(n):** every swap puts at least one value into its home slot, where it stays forever. There are only n slots, so there are at most n swaps over the *whole* run.

**The second way to use the array as storage: sign marks.** If every value is in `[1, n]` and you're allowed to modify the array, mark "value `v` has been seen" by making `nums[v − 1]` negative, and read the original value back with `Math.abs`. Find All Duplicates, in the practice set, uses this.

---

## 5. Count by a derived key

### Worked P5 · [Brick Wall](https://leetcode.com/problems/brick-wall/)

> A wall is a list of rows; each row lists its bricks' widths, and every row has the same total width. Draw one vertical line from top to bottom that crosses the **fewest** bricks. A line along a brick edge doesn't cross that brick, and you can't draw along the two outer edges. Return the number of bricks crossed.

Trying every horizontal position is hopeless: widths can sum to 2·10⁹. **Reframe it:** a line at position x crosses every row *except* the rows with a brick edge at x. So minimising crossings means **finding the position where the most rows have an edge**. Each row's edges are its running sums, excluding the last (that's the wall's outer edge). Count them in a map:

```java
public int leastBricks(List<List<Integer>> wall) {
    Map<Integer, Integer> edgesAt = new HashMap<>();   // position -> number of rows with an edge there
    int mostEdges = 0;
    for (List<Integer> row : wall) {
        int position = 0;
        for (int i = 0; i < row.size() - 1; i++) {     // skip the last brick: its edge is the border
            position += row.get(i);
            mostEdges = Math.max(mostEdges, edgesAt.merge(position, 1, Integer::sum));
        }
    }
    return wall.size() - mostEdges;
}
```

O(total number of bricks). **The pattern: invent the key that the question is really about** (here, an edge position), and count by it. Count Number of Bad Pairs in the practice set uses the key `nums[i] − i`.

---

## 6. Where hashing breaks

| Situation | Why a hash table fails | Use instead |
| :--- | :--- | :--- |
| You need **order**: nearest key, k-th smallest, everything in a range | Hashing scatters keys deliberately, so it knows nothing about order | Sorting + binary search (Ch 9–10), `TreeMap` (Ch 4 §22, Ch 15) |
| Keys are small integers | A map works, but costs 10–50× more than an array | Direct addressing: `int[]`, `boolean[]` |
| Memory is tight with 10⁷ keys | ~40–50 bytes per entry (Ch 3 §7) | Arrays, or sort the data and scan it |
| Keys are `double` | 0.1 + 0.2 ≠ 0.3, so "equal" keys hash differently | Canonical integer forms (reduced fractions, scaled integers) |
| Adversarial input | Crafted keys can share buckets | Accept O(log n) per operation via Java's tree buckets; on hackable judges, randomise the hash |

---

## 7. Variations *(previews; each is solved in the chapter shown)*

| Change | Adaptation |
| :--- | :--- |
| Two Sum, but count *all* pairs | Store counts instead of indices, adding `count[target − x]` before inserting x |
| Two Sum on a sorted array, O(1) space | Two pointers from both ends (Ch 11) |
| Subarray with sum k | Hash the *prefix sums* (Ch 8; Ch 1 Derivation A) |
| Longest Consecutive with a stream of insertions | Union-find over neighbouring values (Ch 24), or a `TreeMap` of intervals (Ch 14) |
| Group strings by a more complex equivalence | Design the canonical key; check both directions |

---

## 8. Recognition

**Signals:**
* "Have I seen…?", "does a pair / complement exist?", "find the duplicate".
* "Group the items that are equivalent under…".
* Counting by some property ("how many rows have an edge at x", "how many pairs share `nums[i] − i`").
* An O(n) requirement on a problem that would be easy after sorting.
* Values limited to `[1, n]` with an O(1)-space requirement: the array is its own table.

**False friends:**
* "The closest value", "the k-th smallest", "how many are less than x": hashing has no order. Use sorting or ordered sets.
* "Subarray sum equals k" *is* hashing, but of prefix sums, which is a different key from the elements themselves (Ch 8).

---

## 9. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Inserting before querying, so an element pairs with itself | Query, then insert (Two Sum, Ch 1 A) |
| A canonical key that merges different groups | Separators between counts; check both directions |
| Iterating over the input instead of the set (Longest Consecutive) | Iterate over the set |
| `map.get(k)` unboxed into `int` when the key is absent | `getOrDefault`, or check for `null` |
| `map.get(a) == map.get(b)` with counts above 127 | `.equals` or `intValue()` (Ch 4 §13) |
| Using `int[]` as a key | `String`, `List<Integer>`, a record, or a packed `long` (Ch 4 §15) |
| A `Map<Character, Integer>` for lowercase letters | `int[26]` |

---

## 10. Practice

1. **[Contains Duplicate II](https://leetcode.com/problems/contains-duplicate-ii/)** *(Easy)*
2. **[Isomorphic Strings](https://leetcode.com/problems/isomorphic-strings/)** *(Easy)*
3. **[Find All Duplicates in an Array](https://leetcode.com/problems/find-all-duplicates-in-an-array/)** *(Medium; O(1) extra space)*
4. **[Determine if Two Strings Are Close](https://leetcode.com/problems/determine-if-two-strings-are-close/)** *(Medium)*
5. **[Count Number of Bad Pairs](https://leetcode.com/problems/count-number-of-bad-pairs/)** *(Medium)*
6. **[4Sum II](https://leetcode.com/problems/4sum-ii/)** *(Medium)*
7. **[Longest Palindrome by Concatenating Two Letter Words](https://leetcode.com/problems/longest-palindrome-by-concatenating-two-letter-words/)** *(Medium)*

**Stretch:** **[Max Points on a Line](https://leetcode.com/problems/max-points-on-a-line/)** *(Hard)* · **[Minimum Area Rectangle](https://leetcode.com/problems/minimum-area-rectangle/)** *(Medium)*

<details><summary>Nudges</summary>

1. Contains Duplicate II: remember the *last* index where each value appeared.
2. Isomorphic: the mapping must be one-to-one in **both** directions. Use two maps, or the "index of first appearance" key from §2.
3. All Duplicates: values are in `[1, n]`. Mark "seen v" by negating `nums[v − 1]`. A value whose slot is already negative is a duplicate.
4. Close Strings: which properties survive both operations (swapping two characters, and relabelling every copy of one letter as another)? The set of letters used, and the *multiset* of counts.
5. Bad Pairs: count the *good* pairs (`j − i = nums[j] − nums[i]` ⟺ `nums[i] − i = nums[j] − j`) and subtract them from all pairs. Use `long`.
6. 4Sum II: n ≤ 200, so n⁴ is far too slow. Count every sum `a + b` in a map, then look up `−(c + d)`: n² + n².
7. Two-Letter Words: pair each word with its reverse. Words like `"gg"` pair with themselves, and at most one leftover one can sit in the middle.
8. Stretch, Max Points: fix one point and hash the *direction* to every other point, reduced by `gcd` with a fixed sign. `gcd(a, b) = gcd(b, a % b)`, with `gcd(a, 0) = |a|`.
9. Stretch, Min Area Rectangle: put every point in a set. For every pair of points that could be opposite corners (different x *and* different y), check whether the other two corners exist.
</details>

<details><summary>Key</summary>

Contains Dup II → P1 with the last index · Isomorphic → P2 (bijection both ways) · All Duplicates → P4 (sign marks) · Close Strings → P2 (set of letters + sorted count multiset) · Bad Pairs → P5 (key `nums[i] − i`, count the complement) · 4Sum II → P1 on pair sums · Two-Letter Words → P1 + reverse-pair counting · Max Points → P2 (reduced slope) · Min Area Rectangle → P3 (look up the opposite corners)
</details>

---

## 11. Self-test

<details><summary>1. What is the load factor, and why does Java resize at 0.75?</summary>

The load factor is entries ÷ buckets, which is the average chain length. Keeping it below a constant keeps every operation O(1) expected. 0.75 balances memory against chain length; resizing roughly doubles the table, so the rehashing cost is amortised O(1) per insert.
</details>

<details><summary>2. Why is a hash map O(1) "expected" and not O(1) worst case?</summary>

The bound depends on keys spreading evenly over the buckets. Keys that collide (by bad luck, a poor hash function or adversarial input) can make one bucket long. Java's tree buckets cap the worst case at O(log n).
</details>

<details><summary>3. Give a canonical key for "two strings follow the same pattern" ("egg" and "add" yes, "foo" and "bar" no).</summary>

Replace each character by the index of its first appearance: "egg" → [0, 1, 1] and "add" → [0, 1, 1], while "foo" → [0, 1, 1] but "bar" → [0, 1, 2].
</details>

<details><summary>4. In First Missing Positive, why is the answer always in [1, n + 1]?</summary>

The n elements can occupy at most n of the values 1…n + 1, so at least one of those n + 1 values is missing.
</details>

<details><summary>5. You need, for each element, the number of earlier elements smaller than it. Hash map?</summary>

No. "Smaller than" needs order, which hashing destroys. That's a job for a sorted structure: merge sort (Ch 9) or a Fenwick tree (Ch 34).
</details>

---

## 12. Exit gate

Move on when you can:

* write `MyHashMap` (buckets, load factor, resize) from a blank file and match `java.util.HashMap` on 100 000 random operations;
* explain why its operations are O(1) expected, and name two ways that guarantee breaks;
* write Two Sum, Group Anagrams (both keys), Longest Consecutive Sequence and First Missing Positive from blank files, each in under 10 minutes;
* for any canonical key you design, argue both directions;
* solve 6 of the 7 practice problems.
