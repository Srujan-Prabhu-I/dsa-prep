# Chapter 3 — Complexity, Honestly

> **Part 0 · Week 3** · Prerequisites: Ch 1, Ch 4 (all three parts)
> **Problems:** 20 code fragments to analyse + a timing experiment · **Time:** about 4 hours

Ch 1 used complexity as a *budget*: the constraints say how slow you're allowed to be. This chapter teaches you to compute the cost of any code **accurately**: nested loops whose bounds depend on each other, loops that look quadratic but are linear, recursion, and the Java operations that hide a loop inside a single line. Then you'll measure the cost for real, so you trust the analysis.

---

## 1. What complexity measures

Complexity doesn't measure seconds. Seconds depend on the machine, the JVM and the judge's load. It measures **how the amount of work grows as the input grows.** Let `T(n)` be the number of basic steps on an input of size `n`.

| Notation | Meaning | Informally |
| :--- | :--- | :--- |
| `T(n) = O(f(n))` | There's a constant `c` with `T(n) ≤ c · f(n)` for all large `n` | "grows **no faster** than f" |
| `T(n) = Ω(f(n))` | `T(n) ≥ c · f(n)` for all large `n` | "grows **at least** as fast as f" |
| `T(n) = Θ(f(n))` | Both | "grows **exactly** like f" |

In interviews, people say "O" when they mean Θ, and that's fine. What matters:

* **State the worst case** unless the question says otherwise. Hash maps are the standard exception: their O(1) is *expected* (average), and you say so.
* **Constants are dropped in the notation, but not in reality.** 2n and 50n are both O(n), but in Java, 10⁷ `HashMap` operations take noticeably longer than 10⁷ array reads. Use O to decide what's *possible*, and constants to choose between possible solutions.
* **Name every variable.** "O(n + m)" for n vertices and m edges, or "O(n · L)" for n strings of length L. Collapsing everything into "n" hides the truth.

The growth ladder, with rough step counts at n = 10⁶:

| O(1) | O(log n) | O(√n) | O(n) | O(n log n) | O(n²) | O(2ⁿ) |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| 1 | 20 | 1 000 | 10⁶ | 2·10⁷ | 10¹² ✗ | ✗✗✗ |

---

## 2. Counting loops: count the innermost work

**Rule: don't count `for` keywords. Count how many times the innermost statement runs, summed over everything.**

```java
// (a) Independent nested loops: n · n = n²
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++) work();

// (b) Dependent ("triangular"): (n−1) + (n−2) + … + 0 = n(n−1)/2 = Θ(n²)
for (int i = 0; i < n; i++)
    for (int j = i + 1; j < n; j++) work();

// (c) Sequential loops ADD: n + m, not n · m
for (int i = 0; i < n; i++) work();
for (int j = 0; j < m; j++) work();

// (d) Halving: i takes the values n, n/2, n/4, …, 1, so about log₂ n iterations
for (int i = n; i > 0; i /= 2) work();

// (e) Doubling: the same count, log₂ n
for (int i = 1; i < n; i *= 2) work();

// (f) Square-root bound: i runs up to √n
for (int i = 1; (long) i * i <= n; i++) work();
```

The triangular case (b) is the one people get wrong in both directions. "The inner loop is shorter, so it must be faster" is true by a factor of 2, but it's still Θ(n²).

### 2.1 When the inner bound depends on the outer variable

```java
// (g) Harmonic: the inner loop runs n/i times
for (int i = 1; i <= n; i++)
    for (int j = i; j <= n; j += i) work();
```

The total is `n/1 + n/2 + n/3 + … + n/n = n · (1 + 1/2 + … + 1/n) ≈ n ln n`, which is **Θ(n log n)**, not n². This shape ("for every i, visit its multiples") appears in divisor counting and prime sieves (Ch 18). Remember the harmonic sum.

```java
// (h) Shrinking inner loops: n + n/2 + n/4 + … < 2n, so Θ(n), not n log n
for (int len = n; len > 0; len /= 2)
    for (int j = 0; j < len; j++) work();
```

### 2.2 Sums to know

| Sum | Value | Where it shows up |
| :--- | :--- | :--- |
| 1 + 2 + … + n | n(n+1)/2 = Θ(n²) | Triangular loops, pairs |
| 1 + 2 + 4 + … + 2ᵏ | 2ᵏ⁺¹ − 1 (less than twice the last term) | Doubling, halving, binary trees |
| n/1 + n/2 + … + n/n | ≈ n ln n | Loops over multiples |
| log 1 + log 2 + … + log n | Θ(n log n) | Inserting n items into a sorted structure |
| 1 + r + r² + … for r < 1 | < 1/(1 − r), a constant | Shrinking work (h) |

---

## 3. Amortised analysis: when one step is expensive but the total isn't

Sometimes a single iteration does a lot of work, yet the **total** over the whole run is small. **Amortised analysis** counts the total directly, instead of multiplying the worst single step by the number of steps.

### 3.1 Pointers that only move forward

```java
int j = 0;
for (int i = 0; i < n; i++) {
    while (j < n && condition(i, j)) j++;
    // ...
}
```

The `while` loop can run n times in a single iteration of `i`, so is this n²? No. **`j` only ever increases, and it stops at n.** Across the *whole* run, `j++` executes at most n times in total. Total work: O(n) for the `i` loop plus O(n) for all the `j` moves = **O(n)**. Every two-pointer and sliding-window algorithm (Ch 11) relies on this argument.

### 3.2 Each element is pushed once and popped once

In Ch 1, Derivation B, one day could pop many waiting days off the stack. The total is still bounded, because **each index is pushed exactly once and popped at most once**. So all the pops together number at most n, and the total work is O(n). The pattern to remember: *count how many times each element can be touched over its whole lifetime*, not how many times one step might touch elements.

### 3.3 The growing array

`ArrayList.add` normally takes O(1). When the internal array is full, Java allocates one that's 1.5× bigger and copies everything: O(n) for that one call. Over n adds, the copies cost about n + n/1.5 + n/1.5² + …, a geometric sum that's less than 3n. So the total is O(n), or **amortised O(1) per add**. You'll build exactly this in Ch 5.

> **Amortised is not the same as worst-case per operation.** A single `add` can still take O(n). That matters in real-time systems, but in algorithm budgets it's the total that counts.

---

## 4. Recursion: draw the tree

Ch 6 teaches you how to *write* recursion. Here we only count its cost. The method is always the same: **draw the recursion tree**, with one node per call, and add up the work done at every node.

**Total cost = (number of calls) × (work per call)**, when every call does about the same work. When the work differs by level, sum it level by level.

| Recurrence | Tree shape | Total |
| :--- | :--- | :--- |
| `T(n) = T(n − 1) + O(1)` | A chain of n calls | **Θ(n)** |
| `T(n) = T(n − 1) + O(n)` | A chain; work n, n−1, … | **Θ(n²)** |
| `T(n) = 2 · T(n − 1) + O(1)` | Each call makes 2 calls: 1, 2, 4, … 2ⁿ nodes | **Θ(2ⁿ)** |
| `T(n) = T(n/2) + O(1)` | A chain of log n calls | **Θ(log n)** |
| `T(n) = T(n/2) + O(n)` | Work n + n/2 + n/4 + … | **Θ(n)** |
| `T(n) = 2 · T(n/2) + O(1)` | 1 + 2 + 4 + … + n nodes | **Θ(n)** |
| `T(n) = 2 · T(n/2) + O(n)` | log n levels, each doing n work in total | **Θ(n log n)** |

The last row is how merge sort works (Ch 9): each level of the tree handles all n elements in total, and there are log₂ n levels.

**The general rule** (the "master theorem", in plain words): for `T(n) = a · T(n/b) + O(nᵈ)`, compare `a` with `bᵈ`:
* `a < bᵈ`: the top call dominates → **Θ(nᵈ)**
* `a = bᵈ`: every level does the same work → **Θ(nᵈ log n)**
* `a > bᵈ`: the leaves dominate → **Θ(n^(log_b a))**

**A one-character trap:**

```java
int twoCalls(int n) { return n <= 1 ? 1 : twoCalls(n - 1) + twoCalls(n - 1); }   // Θ(2ⁿ)
int oneCall(int n)  { return n <= 1 ? 1 : 2 * oneCall(n - 1); }                    // Θ(n)
```

They compute the same number. One makes two recursive calls per level, the other makes one. **Count the calls, not the arithmetic.**

**Branching searches.** A recursion that makes `b` choices per level, to depth `d`, has up to `bᵈ` leaves. Choosing "in or out" for each of n items gives 2ⁿ. Ordering n items gives n · (n−1) · … = n!. Ch 6 adds *pruning*, which cuts branches early, but the worst case is still the full tree unless you can prove otherwise.

**Memoised recursion** (a preview of Ch 27): if every distinct call is computed only once and then remembered, the cost is **(number of distinct calls) × (work per call)**. That single sentence is the complexity rule for all of dynamic programming.

---

## 5. Counting what you enumerate

Many brute forces are "try every X". Know how many X there are:

| Enumerating | Count | Notes |
| :--- | :--- | :--- |
| Pairs `i < j` | n(n−1)/2 | — |
| Subarrays | n(n+1)/2 | — |
| Subsets of n items | 2ⁿ | 2²⁰ ≈ 10⁶, 2³⁰ ≈ 10⁹ |
| Permutations of n items | n! | 10! ≈ 3.6·10⁶, 12! ≈ 4.8·10⁸ |
| Pairs of sets `B ⊆ A ⊆ {1..n}` | **3ⁿ** | 3¹⁵ ≈ 1.4·10⁷ |

The last row surprises people. There are 2ⁿ choices for A and up to 2ⁿ for B inside it, so you might guess 4ⁿ. But count per element instead: each element is **in neither set, in A only, or in both**. That's 3 options per element, so 3ⁿ pairs. Counting *per element* rather than *per structure* is a trick you'll use again for the contribution technique (Ch 12, Ch 37).

---

## 6. Hidden costs: one line, a whole loop

These single lines hide loops. Each one has turned an O(n) plan into an O(n²) solution in some OA:

| Line | Looks like | Actually | Fix |
| :--- | :-: | :-: | :--- |
| `s += c` (String) in a loop | O(1) | O(length) **each time** → O(n²) | `StringBuilder` |
| `s.substring(i, j)` | O(1) | O(j − i): it copies | Pass indices instead of substrings |
| `list.remove(0)`, `list.add(0, x)` on `ArrayList` | O(1) | O(n): shifts everything | `ArrayDeque` |
| `list.contains(x)`, `list.indexOf(x)` | O(1) | O(n) scan | `HashSet` / `HashMap` |
| `pq.remove(x)`, `pq.contains(x)` | O(log n) | O(n) scan | Lazy deletion (Ch 15) |
| `map.put(key, …)` with a String key | O(1) | O(key length) to hash | Usually fine; know it when keys are long |
| `treeMap.headMap(x).size()` | O(log n) | O(k) | Fenwick tree (Ch 34) |
| `Arrays.sort(...)` inside a loop | — | O(n log n) per iteration | Sort once, outside |
| `new int[n]` or `a.clone()` in each recursive call | — | O(n) per call | Share one array, undo changes (Ch 6) |
| `Integer` boxing in a hot loop | free | ~10× slower than `int` | Primitive arrays |

---

## 7. Space complexity

* **Auxiliary space** is the extra memory you allocate beyond the input. Interviewers usually mean this. The output you're required to return is normally not counted.
* **Recursion uses stack space.** Recursion depth D means O(D) space, even if you allocate nothing. A recursive DFS on a path of 10⁵ nodes uses O(10⁵) stack, and in Java that can overflow (Ch 4 §16).
* **Modifying the input in place** can give O(1) auxiliary space, but ask the interviewer first whether you're allowed to.

**Memory budgets in Java** (a typical limit is 256 MB):

| Data | Size |
| :--- | :--- |
| `int[10⁷]` | 40 MB |
| `long[10⁷]` | 80 MB |
| `boolean[10⁷]` | 10 MB (1 byte each) |
| `int[n][m]` | 4nm bytes, plus about 16 bytes of overhead per row |
| One `Integer` in a collection | ~16 bytes for the object + 4–8 for the reference |
| One `HashMap` entry | ~40–50 bytes |
| `ArrayList<Integer>` with 10⁶ elements | ~20 MB, versus 4 MB for `int[10⁶]` |

So a `HashMap` with 10⁷ entries (~500 MB) doesn't fit, whatever its time complexity.

---

## 8. Measure it: the doubling experiment

Analysis predicts growth; measurement confirms it. Run the code for n, 2n, 4n, … and look at the **ratio** between consecutive times:

| Ratio when n doubles | Complexity |
| :-: | :--- |
| ≈ 1 | O(1) or O(log n) |
| ≈ 2 | O(n) |
| a little over 2 (2.1–2.3) | O(n log n) |
| ≈ 4 | O(n²) |
| ≈ 8 | O(n³) |

```java
public class Doubling {
    // Put the code fragment you want to measure here. Return something computed from the
    // work, so the JIT compiler can't decide the loop is useless and delete it.
    static long work(int n) {
        long sink = 0;
        for (int i = 1; i <= n; i++)
            for (int j = i; j <= n; j += i) sink += j;      // fragment (g): harmonic
        return sink;
    }

    public static void main(String[] args) {
        work(1 << 16);                                   // warm-up: let the JIT compile work() first
        long previous = 0;
        for (int n = 1 << 16; n <= 1 << 23; n <<= 1) {
            long start = System.nanoTime();
            long sink = work(n);
            long elapsed = System.nanoTime() - start;
            double ratio = previous == 0 ? 0.0 : (double) elapsed / previous;
            System.out.printf("n = %9d   %9.2f ms   ratio %5.2f   (sink %d)%n",
                    n, elapsed / 1e6, ratio, sink);
            previous = elapsed;
        }
    }
}
```

For the harmonic fragment, expect ratios a little above 2 (n log n). Swap in a triangular double loop and they jump to about 4 (use smaller n, around 2¹⁰–2¹⁵, or you'll wait a long time). Small n give noisy timings; trust the ratios from the larger sizes.

> **Why the warm-up?** Java first interprets your code and then compiles the hot parts to machine code (the "JIT"). The first timing would include that compilation and look misleadingly slow.

---

## 9. Practice: 20 fragments

For each fragment, give a tight Θ bound in terms of the named variables, with a one-line reason. Do all 20 on paper first, then check. Afterwards, measure three of them (#5, #9, #11) with the doubling experiment.

```java
/* 1 */  for (int i = 0; i < n; i++) for (int j = 0; j < n; j++) work();
/* 2 */  for (int i = 0; i < n; i++) for (int j = i; j < n; j++) work();
/* 3 */  for (int i = 1; i < n; i *= 2) work();
/* 4 */  for (int i = 0; i < n; i++) for (int j = 1; j < n; j *= 2) work();
/* 5 */  for (int i = 1; i <= n; i++) for (int j = i; j <= n; j += i) work();
/* 6 */  for (int i = 1; (long) i * i <= n; i++) work();
/* 7 */  int j = 0; for (int i = 0; i < n; i++) { while (j < n && a[j] <= a[i] + k) j++; }
/* 8 */  for (int len = n; len > 0; len /= 2) for (int t = 0; t < len; t++) work();
/* 9 */  String s = ""; for (int i = 0; i < n; i++) s += 'a';
/* 10 */ StringBuilder sb = new StringBuilder(); for (int i = 0; i < n; i++) sb.append('a');
/* 11 */ for (int i = 0; i < n; i++) list.remove(0);          // list is an ArrayList of size n
/* 12 */ for (int i = 0; i < n; i++) deque.pollFirst();        // deque is an ArrayDeque of size n
/* 13 */ int f(int n) { return n <= 1 ? 1 : f(n - 1) + f(n - 1); }     // and: g(n) = 2 * g(n - 1)?
/* 14 */ void h(int n) { if (n <= 1) return; h(n / 2); h(n / 2); for (int i = 0; i < n; i++) work(); }
/* 15 */ void p(int n) { if (n <= 1) return; for (int i = 0; i < n; i++) work(); p(n / 2); }
/* 16 */ for (int i = 0; i < n; i++) { int[] copy = a.clone(); Arrays.sort(copy); }
/* 17 */ // for every subset of an n-element array, add up the elements in that subset
/* 18 */ // for every pair of sets B ⊆ A ⊆ {1, …, n}, do O(1) work
/* 19 */ // BFS over an R × C grid: every cell enters the queue at most once and checks 4 neighbours
/* 20 */ for (String w : words) for (int i = 0; i < w.length(); i++) work();  // total length L
```

<details><summary>Answers</summary>

| # | Bound | Reason |
| :-: | :--- | :--- |
| 1 | Θ(n²) | Independent nested loops |
| 2 | Θ(n²) | Triangular: n(n+1)/2 |
| 3 | Θ(log n) | Doubling |
| 4 | Θ(n log n) | n iterations of a doubling loop |
| 5 | Θ(n log n) | Harmonic sum: n/1 + n/2 + … ≈ n ln n (§2.1) |
| 6 | Θ(√n) | `i` runs while i² ≤ n |
| 7 | Θ(n) | `j` only increases and stops at n; amortised (§3.1) |
| 8 | Θ(n) | n + n/2 + … < 2n (§2.1) |
| 9 | Θ(n²) | Each `+=` copies the whole string so far (§6) |
| 10 | Θ(n) | Amortised O(1) append |
| 11 | Θ(n²) | Each `remove(0)` shifts the remaining elements (§6) |
| 12 | Θ(n) | O(1) at the head of an `ArrayDeque` |
| 13 | Θ(2ⁿ) for f; Θ(n) for g | Two calls per level versus one (§4) |
| 14 | Θ(n log n) | `T(n) = 2T(n/2) + n` |
| 15 | Θ(n) | `T(n) = T(n/2) + n`: n + n/2 + … < 2n |
| 16 | Θ(n² log n) | An O(n log n) sort, n times |
| 17 | Θ(n · 2ⁿ) | 2ⁿ subsets, up to n elements to add for each |
| 18 | Θ(3ⁿ) | 3 choices per element (§5) |
| 19 | Θ(R · C) | Each cell is processed once, with constant work per cell |
| 20 | Θ(L) | The total work is the total length, whatever the number of words |
</details>

---

## 10. Self-test

<details><summary>1. n = 10⁵. Your algorithm is O(n√n). Will it pass in about 1 second in Java?</summary>

n√n ≈ 10⁵ × 316 ≈ 3·10⁷ simple steps: yes, comfortably, as long as those steps are array operations rather than map operations.
</details>

<details><summary>2. A loop has an inner `while` that sometimes runs n times. When is the whole thing still O(n)?</summary>

When the inner loop's variable only moves in one direction and is bounded, so its total movement over the whole run is at most n (§3.1).
</details>

<details><summary>3. Why is "amortised O(1)" not the same as "O(1)"?</summary>

A single operation can still be expensive (an `ArrayList` resize is O(n)). Only the average over a long sequence of operations is O(1).
</details>

<details><summary>4. A recursion makes 3 calls on n/3 of the input and does O(n) work itself. What's the total?</summary>

a = 3, b = 3, d = 1, so a = bᵈ: Θ(n log n).
</details>

<details><summary>5. You need a map with 2·10⁷ integer keys and the memory limit is 256 MB. Problem?</summary>

Yes. At ~40–50 bytes per entry that's about 1 GB. Use a primitive array indexed by key if the key range allows, or rethink the algorithm.
</details>

---

## 11. Exit gate

Move on when you can:

* get at least 18 of the 20 fragments right on paper;
* explain amortisation with the forward-only pointer and the push-once/pop-once arguments, without notes;
* run the doubling experiment and match the measured ratios to your predictions for three fragments.
