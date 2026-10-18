# Chapter 38 — The Recognition Map

> **Part VIII · Week 30** · Prerequisites: Ch 1 (the protocol), Ch 1–33 for the drill; Ch 34–37 entries are marked *(S)* and become usable after those chapters
> **Drill:** 40 custom statements, triage only · **Time:** about 5 hours, then 20 minutes a week until the exit

Every chapter ended with a recognition section for **its own** techniques. This chapter merges them into one map, organised the way problems actually arrive: **unlabeled**. It's built from three kinds of evidence, in the order you should read them:
1. **the constraints**, which bound what can possibly fit;
2. **the structure** of the problem, the properties of Ch 1 Step 6 (monotone, dominance, "the future depends only on…");
3. **the shape of the output and operations** (count mod p, all solutions, k-th, online queries).

Keywords come last, and only as a tiebreaker. "Subarray" fits half the chapters of this handbook; "monotone feasibility" fits one.

**This chapter is a reference, not a reading.** Do the drill (§7), then come back to the map every time a Mixed Set or contest problem goes wrong, and add the signal you missed to your log.

---

## 1. The first five minutes

This is Ch 1's protocol, Steps 1–6, compressed into a timed script. It's what you do **before** committing to any approach, in an OA or an interview.

| Minute | Do | Write down |
| :-: | :--- | :--- |
| 0–1 | Read the statement twice. Restate it in one sentence, in your own words, with the input and output types | The restatement |
| 1–2 | Read the constraints. Find the budget (Ch 1 Step 2) | "n ≤ __ → at most O(__)" |
| 2–3 | Solve the example by hand, plus one tiny case | The hand answer, and anything that surprised you |
| 3–4 | State the brute force and its cost, and **name its bottleneck** (Ch 1 Step 5: recomputation, re-scanning, hopeless candidates, overlapping subproblems, or independent exponential choices) | "Brute: __, O(__). Waste: __" |
| 4–5 | Run the structural questions (Ch 1 Step 6) against the bottleneck. Pick **two candidates**; for each, one sentence on why it would work | "Candidate 1: __ because __. Candidate 2: __" |

**Then decide.** If one candidate has a one-sentence correctness argument, take it. If neither does, try to break the more promising one with a small counterexample (Ch 2) before coding. **If nothing fires by minute 5,** go to §6.

---

## 2. Constraints → the candidate set

Ch 1 Step 2's table gives the slowest safe complexity. This one lists **what usually lives at each size**, which narrows the candidates before you've read the story.

| Constraint | Usually means | Chapters |
| :--- | :--- | :--- |
| n ≤ 10–12 | Permutations, backtracking over orders | 6 |
| n ≤ 20 | Subsets, bitmask DP (`dp[mask]`, `dp[mask][last]`) | 6, 17, 32 |
| n ≤ 40 | Meet in the middle *(S)* | 37 |
| n ≤ 100–500 | O(n³): interval DP, Floyd–Warshall, DP with three indices | 23, 30 |
| n ≤ 2000–5000 | O(n²): 2D DP, all pairs, "fix one endpoint, scan the other" | 28–30 |
| n ≤ 10⁵–10⁶ | O(n log n) or O(n): sort, binary search, hashing, two pointers, heaps, stacks, BFS/DFS, DSU, 1D DP | 7–16, 21–24, 27 |
| n, q ≤ 2·10⁵ with interleaved updates | O(log n) per operation: Fenwick, segment tree *(S)*, ordered sets | 15, 34 |
| Values ≤ 10⁶ | Index arrays by value; counting sort; sieve | 9, 18 |
| A value or answer up to 10⁹–10¹⁸ | Binary search on the answer, math, digit DP, matrix-free recurrences | 10, 18, 32 |
| k, the alphabet, or the number of distinct values tiny (≤ 26, ≤ 10) | An extra DP dimension, a mask, or a count array | 17, 28 |
| "Mod 10⁹ + 7" | Counting: DP or combinatorics, never enumeration | 18, 27–33 |
| A grid of up to 10⁶ cells | BFS/DFS/DP over cells; the grid is a graph | 21, 28 |

**Two traps.** A **suspiciously small n** (2000 where 10⁵ would be natural) means the intended solution is O(n²): don't spend 20 minutes on an O(n log n) idea. And **several parameters** multiply: n ≤ 10⁵ with k ≤ 100 allows O(n·k) = 10⁷.

---

## 3. Structural signals → techniques

The core of the map. Each row is a **property you can check**, not a word in the statement.

### 3.1 About order and monotonicity

| If you can show… | Then try | Ch |
| :--- | :--- | :-: |
| The input's order doesn't affect the answer | Sort it, or reduce it to counts | 7, 9 |
| "If x works, every larger x works" (feasibility is monotone in the answer) | Binary search on the answer + a feasibility check | 10 |
| A function of the index is monotone (sorted data, prefix sums of positives) | Binary search on the index | 10 |
| Growing a window only makes it "worse" (or only "better") | Two pointers / a variable sliding window | 11 |
| Both pointers can be made to move in one direction only | Two pointers on sorted data | 11 |
| The best predecessor sits in a sliding window | Monotonic deque | 12, 33 |

### 3.2 About what to remember

| If you can show… | Then try | Ch |
| :--- | :--- | :-: |
| Each element needs a lookup **by value** among earlier elements | Hash map / set | 7 |
| A subarray property is a function of two prefix states | Prefix sums (or XOR, or remainders) + hash map | 8 |
| Each element needs the nearest greater/smaller element | Monotonic stack | 12 |
| Some candidates are beaten forever by a later one (dominance) | Stack / deque that discards them | 12 |
| You repeatedly need the current best of a changing set | Heap; an ordered set if you also delete arbitrary elements | 15 |
| The structure must support several operations within a time bound | Combine structures; each covers one operation (design) | 16 |
| The only memory needed about past choices is a small summary | DP with that summary as the state | 27 |

### 3.3 About choices

| If you can show… | Then try | Ch |
| :--- | :--- | :-: |
| An exchange argument: swapping two adjacent choices into some order never hurts | Greedy in that order | 2, 26 |
| A locally best choice "stays ahead" of any other solution | Greedy, proved by staying ahead | 2, 26 |
| The future depends only on (index, small parameter) | DP over those | 27–30 |
| The last decision splits the problem into a left part and a right part | Interval / partition DP | 30 |
| The future depends only on **which** items are used, n ≤ 20 | Bitmask DP | 32 |
| The search must list every solution | Backtracking with pruning | 6 |
| A DP is correct but its transition loops over a range of earlier states | Speed up the transition by its shape | 33 |

### 3.4 About relations between things

| If you can show… | Then try | Ch |
| :--- | :--- | :-: |
| There are states and moves between them (even if no graph is mentioned) | A graph: BFS for unweighted steps | 21 |
| The moves have weights | Dijkstra (non-negative), 0-1 BFS, Bellman–Ford (negative) | 23 |
| "A before B" constraints | Topological order, DP on the DAG | 22 |
| Things merge into groups, and only "same group?" matters | DSU | 24 |
| Connect everything as cheaply as possible | MST | 24 |
| Removing one element disconnects something | Bridges / articulation points | 25 |
| Use every edge exactly once | Euler path | 25 |
| A hierarchy with questions answered from the children | Tree DP / post-order | 19, 31 |
| Many questions about ancestors or paths in a static tree *(S)* | Binary lifting, Euler tour | 36 |

### 3.5 About counting and sums over structures

| If you can show… | Then try | Ch |
| :--- | :--- | :-: |
| The answer is a sum over all subarrays/substrings/pairs | Contribution: count per element (with an owner rule) | 12, 37 |
| You're counting arrangements with a closed form | Combinatorics, inclusion–exclusion, mod inverse | 18 |
| You're counting integers in [L, R] by a digit property | Digit DP | 32 |
| Pairs i < j with a relation between their values | Sort / merge-sort counting; Fenwick over values *(S)* | 9, 34 |

### 3.6 About strings

| If you can show… | Then try | Ch |
| :--- | :--- | :-: |
| Two sequences are aligned character by character | 2D DP (LCS, edit distance) | 29 |
| Words are looked up by **prefix** | Trie | 20 |
| Exact pattern search at 10⁵–10⁶ *(S)* | KMP / Z | 35 |
| Many substrings compared for equality *(S)* | Rolling hash, often with binary search on length | 35 |

---

## 4. Output and operation signals

| The problem asks for… | Think first of | Ch |
| :--- | :--- | :-: |
| The minimum number of **steps / moves** | BFS over states | 21 |
| The minimum **cost** with weighted moves | Dijkstra over states | 23 |
| The count of ways, mod p | DP or combinatorics | 18, 27 |
| Every solution (all subsets, all paths) | Backtracking | 6 |
| The k-th smallest / largest | Heap of size k; quickselect; binary search on the value + counting | 9, 10, 15 |
| The lexicographically smallest result built by deletions | Greedy with a stack | 12 |
| The maximum of a minimum (or minimum of a maximum) | Binary search on the answer | 10 |
| "Is it possible…" with huge n | An invariant, parity, or a math argument | 2, 18 |
| Answers after **each** operation, online | A maintained structure (heap, DSU, ordered set, segment tree) | 15, 24, 34 |
| Answers to queries that can be reordered | Offline: sort the queries | 24, 34, 37 |
| A class with methods and complexity requirements | Design: compose structures | 16 |

---

## 5. False friends: the common misroutes

These are the confusions the Mixed Sets reveal most often. When your first candidate is on the left, check the condition before committing.

| You think… | …but check | If so, it's actually |
| :--- | :--- | :--- |
| Sliding window | Are there **negative** numbers? | Prefix sums + hash map, or prefix + monotonic deque |
| Greedy | Does a 3-element counterexample break it? (Ch 2) | DP |
| DP | Does an exchange argument prove a fixed order? | Greedy (faster and simpler) |
| Binary search on the answer | Is feasibility **really** monotone? | Something else; test x and x + 1 by hand |
| BFS | Are the edge weights different? | Dijkstra or 0-1 BFS |
| Dijkstra | Are there negative edges? | Bellman–Ford, or a DAG DP |
| DFS for "shortest" | Is it unweighted? | BFS (DFS doesn't give shortest paths) |
| Heap | Do you need to delete arbitrary elements? | Ordered set (`TreeMap`) |
| Backtracking over subsets | Do sub-searches overlap? | DP (memoise the remaining set or the summary) |
| Bitmask DP | Is n really ≤ 20? | Meet in the middle (≤ 40), or another state |
| Segment tree | Are there any updates at all? | Prefix sums or a sparse table |
| Segment tree | Is the window sliding monotonically? | Monotonic deque |
| Union-find | Are edges **removed**? | Reverse time, or BFS per query |
| An O(n²) DP over pairs | Must the chosen pairs increase in **both** keys? | Sort by one key (ties in reverse), then LIS on the other, as in Russian Doll Envelopes (Ch 29) |
| Interval DP | Is it unweighted interval selection? | Greedy by end time |
| String algorithms | Is it a subsequence, not a substring? | DP |

---

## 6. When nothing fires

You've spent five minutes and no candidate has a correctness sentence. Don't stare. Use Ch 1 §7's unstuck moves **in writing**. These six resolve most cases; try them in this order:
1. **Solve smaller cases by hand** (n = 1, 2, 3, 4) and look for a pattern in the answers.
2. **Fix something:** the answer's value, the last element, the maximum, the middle.
3. **Relax a constraint:** solve it without the hard condition, then add it back.
4. **Reverse it:** go backwards in time, start from the target, or count the complement.
5. **Change the representation:** a graph of states, a sorted order, prefix sums, a different index.
6. **Write the brute force anyway**, then ask which part of it repeats work.

In an OA, if 10 minutes pass with nothing: implement the brute force for partial credit (Ch 39) and move on. In an interview, say what you've tried and ask for a nudge; using a hint well scores better than silence (Ch 40).

---

## 7. The drill: 40 statements

**Rules.** For each statement: 3 minutes, a timer, no solving. Write the budget, your **top two** candidate techniques, and the signal that chose them. Then check the key. Score 1 if the key's technique is in your top two. **Target: 34 / 40.** Redo the misses a week later with the statements shuffled.

Statements marked *(S)* use S-tier chapters; skip them if you haven't done those chapters yet.

1. n ≤ 2·10⁵ **positive** integers and a target S. Find the length of the shortest contiguous segment with sum ≥ S.
2. n ≤ 10⁵ integers (possibly negative) and k. Count the contiguous segments whose sum is divisible by k.
3. n ≤ 10⁵ packages with weights must ship in the given order within D days, one or more packages per day. Find the smallest daily capacity that works.
4. A string of up to 10⁶ characters from `()[]`. Find the length of the longest substring that is correctly bracketed.
5. n ≤ 10⁵ buildings in a row. For each, find the nearest building to its left that is at least as tall.
6. n ≤ 10⁵ broadcasts, each a half-open time interval [s, e). Find the largest number on air at any one instant.
7. Numbers arrive one at a time (up to 10⁵). After each arrival, report the median of everything so far.
8. Design a structure supporting insert(x), remove(x) and getRandom() (uniform over the current elements), all in O(1) on average.
9. A grid of up to 10⁶ cells has exactly two islands. Find the fewest water cells to turn into land to connect them.
10. n ≤ 10⁵ tasks with prerequisite pairs. Output an order in which all tasks can be done, or report that none exists.
11. A directed graph with n ≤ 10⁵ nodes, m ≤ 2·10⁵ edges, non-negative costs. You may halve the cost of one edge of your choice, once. Find the cheapest cost from node 1 to node n.
12. n ≤ 10⁵ cities and m ≤ 2·10⁵ roads built one at a time. After each road, print the number of separate regions.
13. A network of n ≤ 10⁵ computers and m ≤ 2·10⁵ cables. List every cable whose failure would split the network.
14. n ≤ 10⁵ one-hour jobs, each with a deadline (in hours) and a profit. Choose jobs to maximise the total profit of those finished on time.
15. Count the ways to climb n ≤ 10⁵ stairs using steps of 1, 3 or 4, mod 10⁹ + 7.
16. A 500 × 500 grid with blocked cells. Count the paths from the top-left to the bottom-right corner moving only right or down, mod 10⁹ + 7.
17. Two strings of length ≤ 2000. Find the fewest single-character deletions (from either string) that make them equal.
18. n ≤ 300 piles in a row. Merging two **adjacent** piles costs their total size. Find the minimum total cost to merge everything into one pile.
19. A tree with n ≤ 10⁵ weighted nodes. Choose nodes, no two of them adjacent, to maximise the total weight.
20. n ≤ 16 workers, n jobs and a cost matrix. Assign one job per worker to minimise the total cost.
21. Count the integers in [L, R], R ≤ 10¹⁸, whose digits never decrease from left to right.
22. n ≤ 10⁵ positions in a row with values (some negative). Start at position 0 and end at position n − 1, jumping forward at most k positions at a time. Maximise the sum of the values you land on.
23. *(S)* An array of n ≤ 10⁵ numbers and q ≤ 10⁵ operations, each "add v to every element in [l, r]" or "report the minimum of [l, r]".
24. *(S)* A text of length 10⁶ and a pattern of length 10⁶. Count the occurrences of the pattern in the text.
25. *(S)* A tree with n ≤ 2·10⁵ nodes and q ≤ 2·10⁵ queries, each asking for the number of edges between two given nodes.
26. *(S)* n ≤ 36 numbers, each up to 10¹². Count the subsets whose sum is exactly T.
27. n ≤ 10⁵ numbers. Find the sum, over all contiguous segments, of (maximum − minimum).
28. n ≤ 10⁵ pairs (a, b) with a < b. Using pairs in any order, build the longest chain in which each pair's a is greater than the previous pair's b.
29. A number x ≤ 10¹⁸. Each operation subtracts 1, halves x if it's even, or divides it by 3 if it's divisible by 3. Find the fewest operations to reach 1.
30. n ≤ 10⁵ words of total length ≤ 10⁶. Count the pairs (i, j), i ≠ j, where word i is a prefix of word j.
31. n ≤ 10⁵ integers up to 10⁹. Find the largest XOR of any two of them.
32. n ≤ 10⁵ intervals. Remove as few as possible so that the rest don't overlap.
33. An n × m matrix (n · m ≤ 10⁵) whose rows and columns are each sorted. Find its k-th smallest value.
34. n ≤ 10⁵ fuel stations around a circular road, each with an amount of fuel and the fuel needed to reach the next. Find a station from which a full circuit is possible, or report none.
35. A singly linked list of n ≤ 10⁵ nodes. Reverse it in consecutive groups of k, leaving a final short group as it is.
36. A string of length ≤ 10⁵. Find the longest substring with at most 2 distinct characters.
37. A number n ≤ 10¹². Count its divisors.
38. n ≤ 10⁵ integers. Count the pairs i < j with a[j] − a[i] = j − i.
39. List every string of length n ≤ 12 over {a, b, c} with no two equal adjacent letters and exactly k letters 'a'.
40. A log of n ≤ 10⁵ timestamped deposits and withdrawals, then q ≤ 10⁵ queries "the earliest time the balance is ≤ −t".

<details><summary>Key: open only after the full 40</summary>

| # | Technique (top choice) | Also accepted | Signal | Ch |
| :-: | :--- | :--- | :--- | :-: |
| 1 | Variable sliding window | Prefix sums + binary search | Positive values: growing only increases the sum | 11 |
| 2 | Prefix remainders + hash counts | — | Segment property = two prefix states; negatives rule out windows | 8 |
| 3 | Binary search on the capacity | — | Feasibility is monotone in the capacity | 10 |
| 4 | Stack of indices | DP | Nesting: match each closer to the most recent unmatched opener | 12 |
| 5 | Monotonic stack | — | Nearest element to the left with a comparison; shorter ones are dominated | 12 |
| 6 | Sweep over sorted events (ends before starts) | Heap of end times | Concurrency of intervals | 14 |
| 7 | Two heaps | Ordered multiset | Online median | 15 |
| 8 | Hash map (value → index) + array, swap-remove | — | O(1) for three operations: one structure each | 16 |
| 9 | Multi-source BFS from one island | 0-1 BFS | Fewest cells = unweighted steps from a set | 21 |
| 10 | Topological sort (Kahn) | DFS with colours | "Before" constraints; a cycle means impossible | 22 |
| 11 | Dijkstra on (node, used) | — | A one-time option becomes a state bit | 23 |
| 12 | DSU with a component counter | — | Only merging, and "how many groups" | 24 |
| 13 | Bridges (low-link DFS) | — | "Removing one edge disconnects" | 25 |
| 14 | Greedy by deadline + min-heap of chosen profits | Sort by profit + DSU of free slots | Exchange argument; regret | 26 |
| 15 | 1D DP | — | Count ways, last step is 1, 3 or 4 | 27 |
| 16 | Grid DP | — | Paths that only go right/down: each cell from two neighbours | 28 |
| 17 | LCS DP | — | Deletions only: keep the longest common subsequence | 29 |
| 18 | Interval DP | — | The last merge splits the row into a left and a right part | 30 |
| 19 | Tree DP (take / skip) | — | Choices interact only with neighbours in the tree | 31 |
| 20 | Bitmask DP (assignment) | — | n ≤ 16; the future depends only on which jobs are used | 32 |
| 21 | Digit DP (previous digit, tight) | Combinatorics (stars and bars) | Count integers up to 10¹⁸ by a digit property | 32 |
| 22 | DP + monotonic deque | — | Max over the last k states | 33 |
| 23 | Lazy segment tree (add, min) | — | Range updates + range queries, interleaved | 34 |
| 24 | KMP | Z-function | Exact matching at 10⁶ | 35 |
| 25 | LCA by binary lifting + depths | Euler tour + sparse table | Many path questions on a static tree | 36 |
| 26 | Meet in the middle | — | n ≈ 36, huge values (no knapsack) | 37 |
| 27 | Contribution + monotonic stack | — | Sum over all segments; each element owns the segments where it's the max (min) | 12, 37 |
| 28 | Greedy by end (b) | Sort + LIS-style DP | Unweighted chain of intervals = interval scheduling | 14, 26 |
| 29 | Memoised recursion over the few useful states | — | Naive memoisation on x − 1 explores up to 10¹⁸ values. But subtracting 1 more than x mod 2 times before a halving never helps (exchange), so f(x) = min(x mod 2 + 1 + f(⌊x/2⌋), x mod 3 + 1 + f(⌊x/3⌋)), and only the O(log² x) values ⌊x / (2ᵃ3ᵇ)⌋ are ever reached | 27, 37 §5 |
| 30 | Trie with end counts | Sort + hashing | Prefix relations among words | 20 |
| 31 | Binary trie, greedy by bit | Bit-by-bit with a prefix set | Maximise XOR: decide the highest bit first | 17, 20 |
| 32 | Greedy by end time | — | Keep the most compatible intervals: earliest end first | 14 |
| 33 | Binary search on the value + staircase count | Heap of row heads | k-th in a sorted structure; counting ≤ x is O(n + m) | 10, 15 |
| 34 | Greedy: restart after a negative prefix | — | If you fail at j starting from i, no start between i and j works | 2, 26 |
| 35 | Pointer surgery | — | Linked list rewiring | 13 |
| 36 | Variable sliding window + counts | — | "At most k distinct": shrinking only helps | 11 |
| 37 | Trial division to √n | — | One number up to 10¹² | 18 |
| 38 | Transform to a[i] − i, hash counts | — | The relation becomes equality of one derived value | 7 |
| 39 | Backtracking | — | "List every" at n ≤ 12 | 6 |
| 40 | Prefix minimum of the balance + binary search | — | Prefix minima are monotone non-increasing over time | 8, 10 |
</details>

---

## 8. Exit gate

Done when you can:

* run the five-minute script on any unseen problem and produce two candidates with reasons;
* score ≥ 34 / 40 on the drill, and ≥ 36 on the shuffled redo a week later;
* name the check that separates each false-friend pair in §5;
* add every recognition miss from your contests and Mixed Sets to your log as one "signal" line.
