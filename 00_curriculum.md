# The DSA Handbook — Curriculum & Operating Manual

> **Goal.** Clear any DSA round, from college placement filters to the hardest FAANG and startup rounds, by building three capabilities:
>
> 1. **Derivation.** Invent a solution to an unseen problem: isolate the brute-force bottleneck, find the structural property that removes it (monotonicity, invariant, dominance, exchange), and prove it or kill it with a counterexample.
> 2. **Execution.** Write correct, overflow-safe Java from a blank editor inside the complexity budget, and verify it with your own stress-testing harness.
> 3. **Selection.** Pick the right technique for a problem with no topic tag, by reading constraints and structure rather than keywords.
>
> **Scope.** Absolute mastery of interview DSA, *not* competitive programming. Everything a strong interviewer or a hard OA can reasonably ask is in scope. Things like max-flow, HLD, suffix automata and FFT are covered only well enough to recognise that a problem is out of scope (Ch 37).
>
> **A closed problem set, not a grind.** The [problem plan](00_problem_plan.md) is the complete list: about 475 problems you solve yourself, plus one contest a week. DSA is meant to take about 2 hours a day, so most of your engineering time stays on backend work.
>
> **Deadline.** The week-by-week [schedule](00_schedule.md) runs from 19 Oct 2026 to 20 Jun 2027, the day before 4th year starts.

---

## 1. What "done" looks like

You are finished when all of these are true on **problems you have never seen**:

| Benchmark | Target |
| :--- | :--- |
| LeetCode Medium, unlabeled | ≥ 85% solved, ≤ 25 min each, no hints |
| LeetCode Hard, unlabeled | ≈ 50% solved, ≤ 45 min each, no hints |
| 90-min OA (3–4 problems, mixed) | ≥ 3 fully solved, partial on the rest |
| 45-min live mock | Brute → optimal derived out loud, clean code, self-tested, follow-up handled |
| Blank-file rebuild | Every template in Appendix A rewritten from memory, compiling first try |

Codeforces ratings are deliberately not a target. A CF 1600–1800 level of *reasoning* falls out naturally from this book, but the speed tricks and exotic data structures CP rewards are not the point.

---

## 2. How this book is different from a pattern catalogue

**Problems before theory.** Every technique chapter opens with a problem you attempt *before* reading the technique. Productive failure is the point: you will understand *why* the technique exists because you will have hit the exact wall it removes.

**Properties, not patterns.** Chapters are organised by the *reasoning tool* (monotonicity, dominance, prefix state, information flow in trees, optimal substructure) rather than by data type. "Two pointers" and "sliding window" are one chapter because they rest on one property. Binary search and sliding window are siblings, because both exploit monotonicity.

**Every technique has a "where it breaks" section.** A technique you cannot break is a technique you will misapply. You learn the counterexample that kills it and the tool that replaces it.

**Recognition comes from interleaving, not keyword tables.** Labeled chapter problems teach the tool. The **Mixed Sets** (one per Part, unlabeled, drawn from *every* chapter so far) teach you to choose. Ch 38 is the final recognition map, built from *structural signals* (e.g. "feasibility is monotone in the answer"), not from words in the statement.

**Java is treated as an instrument.** Overflow, comparator contracts, boxing, collection costs, and recursion depth are drilled up front (Ch 4), and a stress-testing harness (Ch 5) is used in every chapter afterwards.

---

## 3. Curriculum

Tier legend: **M** = must (every interview loop), **S** = should (hard OAs, Google/startup hard rounds), **R** = recognise only.

### Part 0 — The Method *(Weeks 1–2, in parallel with Part I; reread monthly)*

| Ch | Title | Tier | Core content |
| :-- | :--- | :-: | :--- |
| 01 | [The Derivation Protocol](01_derivation_protocol.md) | M | Constraint → budget; brute force as a search space; bottleneck isolation; the structural questions; 13 unstuck moves; four full derivations |
| 02 | [Proofs and Counterexamples](02_proofs_and_counterexamples.md) | M | Invariants, induction, exchange argument, stays-ahead, dominance, extremal arguments; systematic counterexample hunting by hand and by brute-force search |
| 03 | [Complexity, Honestly](03_complexity.md) | M | Dependent loops, amortisation, harmonic sums, recursion trees, states × transitions, 3ⁿ submasks, hidden Java costs, memory budgets |

### Part I — Foundations: Java, Arrays & Strings, Recursion

| Ch | Title | Tier | Core content |
| :-- | :--- | :-: | :--- |
| 04 | [Java for DSA: Essentials, Semantics & Collections](04_java_for_dsa.md) | M | **Every language feature DSA needs:** LeetCode vs OA program shapes, fast input/output (all common input formats), types, arrays, strings, methods, generics, lambdas, records; then where Java silently goes wrong (overflow, division/mod, floating point, boxing, equality, comparators, hash keys, recursion depth); the collections performance contract and idioms |
| 05 | [Arrays, Strings & Implementation](05_arrays_strings_implementation.md) | M | **From zero:** how arrays and dynamic arrays work (build one); handling fluency: traversal, building/parsing strings, `char` arithmetic, in-place writes, matrix transforms, simulation; the stress harness (brute + generator + checker); I/O drills on CSES |
| 06 | [Recursion & Backtracking](06_recursion_and_backtracking.md) | M | **From zero:** base case, call stack, recursion trees; recursion as induction; subsets/permutations/combinations with duplicates; constraint search (N-Queens, Sudoku, word search); pruning; search-tree size estimates |

### Part II — Linear Structures: the interview core

| Ch | Title | Tier | Core content |
| :-- | :--- | :-: | :--- |
| 07 | [Hashing: Remembering the Past](07_hashing.md) | M | **From zero:** hash tables (build one with chaining); complement lookup, canonical keys/signatures, grouping, frequency tables, index-as-hash |
| 08 | [Prefix Sums & Prefix-State Counting](08_prefix_sums.md) | M | 1D/2D prefix sums, difference arrays, prefix + hashmap (sum = k, divisible by k, balanced, XOR, parity masks), prefix/suffix decomposition |
| 09 | [Sorting as Structure](09_sorting.md) | M | **From zero:** elementary sorts, stability, merge sort, quicksort, the n log n lower bound; sorting to create structure, comparator design + exchange proofs, inversions, quickselect, counting sort, coordinate compression |
| 10 | [Binary Search: The Boundary Invariant](10_binary_search.md) | M | One template for everything; boundaries; search on the answer; k-th smallest by counting; non-monotone cases; reals |
| 11 | [Two Pointers & Sliding Windows](11_two_pointers_and_windows.md) | M | Monotone pointer movement; pair-elimination proof; variable windows; count-of-windows; atMost(k) − atMost(k−1); the negative-number counterexample |
| 12 | [Stacks, Queues & Dominance](12_stacks_queues_dominance.md) | M | **From zero:** stack, queue, circular buffer, deque (build them); monotonic stack (next greater, histogram, contribution of minimums), monotonic deque (window max), expression parsing, greedy stack |
| 13 | [Linked Lists: Pointer Surgery](13_linked_lists.md) | M | **From zero:** singly/doubly linked lists (build one); dummy nodes, reversal (whole/k-group), fast/slow with Floyd's proof, merge/split, copy with random pointer |
| 14 | [Intervals & Sweep Line](14_intervals_and_sweep.md) | M | Sort by start vs by end and why; merge/insert/intersect; event sweep; min rooms; TreeMap sweeps |
| 15 | [Heaps & Ordered Sets](15_heaps_and_ordered_sets.md) | M | **From zero:** binary heap (build one, O(n) heapify, heap sort), balanced BSTs behind TreeMap; top-k, k-way merge, two-heap median, lazy deletion, heap-driven simulation |
| 16 | [Designing Data Structures](16_designing_data_structures.md) | M | LRU, LFU, AllOne, time-based KV, snapshot array, rate limiter, randomised set, iterators |
| — | **[Mixed Set II](mixed_set_2.md)** | — | 30 unlabeled problems from Ch 1–16 |

### Part III — Bits & Math

| Ch | Title | Tier | Core content |
| :-- | :--- | :-: | :--- |
| 17 | [Bit Manipulation](17_bit_manipulation.md) | M | Binary representation and two's complement from zero; XOR algebra, masks as sets, submask enumeration, per-bit contribution, Gray code, bitwise tricks Java gets wrong |
| 18 | [Math for Coding Rounds](18_math.md) | M/S | GCD/LCM, modular arithmetic, fast power, inverse, nCr tables, sieve + SPF, counting (stars & bars, inclusion–exclusion, Catalan), expectation by linearity, minimax games |
| — | **[Mixed Set III](mixed_set_3.md)** | — | 25 unlabeled, Ch 1–18 |

### Part IV — Trees

| Ch | Title | Tier | Core content |
| :-- | :--- | :-: | :--- |
| 19 | [Binary Trees: Information Flow](19_binary_trees.md) | M | **From zero:** terminology, representation, all four traversals (recursive, plus iterative preorder and inorder); top-down vs bottom-up, return tuples, "return vs record globally" (diameter, max path sum), LCA, construction, serialisation |
| 20 | [BSTs & Tries](20_bsts_and_tries.md) | M | **From zero:** BST operations, why balance matters; BST invariant as interval bounds, k-th/successor/delete; tries (build one), Word Search II, binary (XOR) trie |
| — | **[Mixed Set IV](mixed_set_4.md)** | — | 22 unlabeled, Ch 1–20 |

### Part V — Graphs

| Ch | Title | Tier | Core content |
| :-- | :--- | :-: | :--- |
| 21 | [Graph Modelling, BFS & DFS](21_graphs_bfs_dfs.md) | M | **From zero:** terminology, edge list / adjacency list / matrix, BFS and DFS from scratch; "what is a state?"; implicit graphs; grids; multi-source BFS; components; cycle detection; bipartite; BFS on augmented state |
| 22 | [DAGs & Topological Order](22_dags_topological_order.md) | M | Kahn and DFS order; cycle detection; DP over DAGs; lexicographically smallest order; alien dictionary |
| 23 | [Shortest Paths](23_shortest_paths.md) | M | BFS proof, 0-1 BFS, Dijkstra (proof + negative-edge counterexample), state-augmented Dijkstra, k-edge Bellman–Ford, Floyd–Warshall for small n |
| 24 | [Union-Find & Minimum Spanning Trees](24_union_find_mst.md) | M | **From zero:** DSU derived from naive relabelling; path compression + size; component metadata; offline reverse; cut property; Kruskal, Prim; minimax paths |
| 25 | [Connectivity & Euler Paths](25_connectivity_euler.md) | S | Bridges & articulation points (low-link), SCCs + condensation DAG, Hierholzer (Reconstruct Itinerary) |
| — | **[Mixed Set V](mixed_set_5.md)** | — | 25 unlabeled, Ch 1–25 |

### Part VI — Greedy & Dynamic Programming

| Ch | Title | Tier | Core content |
| :-- | :--- | :-: | :--- |
| 26 | [Greedy: Prove It or Kill It](26_greedy.md) | M | Exchange, stays-ahead, regret greedy with heaps, the greedy counterexample museum |
| 27 | [DP I: From Brute Recursion to Tables](27_dp1_method.md) | M | The DP method (state in English → last decision → base → order → answer → cost); memo → table → rolling array → reconstruction; linear DPs |
| 28 | [DP II: Grids, Knapsack & Counting](28_dp2_grids_knapsack_counting.md) | M | Grid paths, 0/1 vs unbounded knapsack and loop-order semantics, combinations vs permutations, counting without double-counting |
| 29 | [DP III: Sequences & Strings](29_dp3_sequences_strings.md) | M | LIS (n log n with proof), LCS + reconstruction, edit distance, palindromes, wildcard/regex, distinct subsequences |
| 30 | [DP IV: Intervals & State Machines](30_dp4_intervals_state_machines.md) | M/S | Interval DP ("last action" trick: Burst Balloons), stone merging; stock problems as automata |
| 31 | [DP V: Trees, DAGs & Rerooting](31_dp5_trees_dags_rerooting.md) | M/S | Include/exclude on trees, tree diameter as DP, rerooting (sum of distances) |
| 32 | [DP VI: Bitmask & Digit DP](32_dp6_bitmask_digit.md) | S | dp[mask][last], assignment, partition into k subsets; digit DP (pos, tight, started, state) |
| 33 | [DP VII: Faster Transitions](33_dp7_faster_transitions.md) | S | Prefix-sum transitions, monotonic-deque DP, binary search in DP (weighted interval scheduling), heap-optimised DP |
| — | **[Mixed Set VI](mixed_set_6.md)** | — | 30 unlabeled, Ch 1–33 |

### Part VII — The Advanced Toolkit

| Ch | Title | Tier | Core content |
| :-- | :--- | :-: | :--- |
| 34 | [Range Queries: Fenwick, Segment Tree, Sparse Table](34_range_queries.md) | S | **From zero:** why prefix sums fail under updates → Fenwick and segment tree derived and built; lazy propagation, walking the tree, offline sweeps (count smaller after self), RMQ |
| 35 | [String Algorithms](35_string_algorithms.md) | S | Prefix function/KMP, Z-function, rolling hash with collision safety, hash + binary search |
| 36 | [Tree Techniques](36_tree_techniques.md) | S | Binary lifting (k-th ancestor, LCA, path aggregates), Euler tour flattening |
| 37 | [Synthesis Techniques & the Out-of-Scope Guide](37_synthesis_and_out_of_scope.md) | S/R | Contribution technique, meet in the middle, offline/reverse time, "answer is small", sqrt intuition; recognition briefs for flow, HLD, centroid, suffix structures, FFT, CHT, Mo's |
| — | **[Mixed Set VII](mixed_set_7.md)** | — | 22 unlabeled, Ch 1–37, Hard-heavy |

### Part VIII — Performance

| Ch | Title | Tier | Core content |
| :-- | :--- | :-: | :--- |
| 38 | [The Recognition Map](38_recognition_map.md) | M | Constraint → candidate set; structural signal → candidate; false friends; the "first 5 minutes" script |
| 39 | [The OA Playbook](39_oa_playbook.md) | M | Triage, partial credit, time checkpoints, pre-submit audit, plain-editor drills |
| 40 | [The Live Interview Playbook](40_interview_playbook.md) | M | 45-minute pacing, thinking aloud, absorbing hints, follow-up variations, self-recorded mocks |

### Appendices

| | Title |
| :-- | :--- |
| A | [Java Template Library](appendix_a_templates.md): every rebuild target in one file |
| B | [Practice Log & Re-solve Schedule](appendix_b_practice_log.md) |

Links point to each chapter's file name; chapters are written in order, to the template in §5. The exact worked, practice and stretch problems for every chapter are in the [problem plan](00_problem_plan.md), and the dates are in the [schedule](00_schedule.md).

### 3.1 Starting from zero: the data-structure coverage map

**Prerequisite:** basic Java programming (variables, loops, `if`, methods, arrays, simple classes). **No DSA knowledge is assumed.** Every data structure is introduced from zero before a technique relies on it (Ch 4 shows the Java collection APIs early, as tools): what it is, how it works inside, the cost of each operation and *why*, and then **you implement it yourself** and check it against Java's version with the stress harness. Only after that does the chapter move on to techniques.

| Structure | Introduced from zero in | You build it yourself | Java's version you then use |
| :--- | :--- | :--- | :--- |
| Array, dynamic array | Ch 5 | Growable `IntList` | `int[]`, `ArrayList` |
| String, string builder | Ch 4–5 | Manual `char[]` building | `String`, `StringBuilder` |
| Recursion / call stack | Ch 6 | Recursion trees by hand | — |
| Hash table (set/map) | Ch 7 | Chaining hash map with resizing | `HashMap`, `HashSet` |
| Sorting algorithms | Ch 9 | Insertion, merge, quick, counting sort | `Arrays.sort`, `Collections.sort` |
| Stack, queue, deque, circular buffer | Ch 12 | Array-backed stack, circular queue | `ArrayDeque` |
| Linked list (singly, doubly) | Ch 13 | Full linked list | (rarely `LinkedList`) |
| Binary heap / priority queue | Ch 15 | Min-heap with sift-up/down, heapify | `PriorityQueue` |
| Balanced BST / ordered set | Ch 15 (concept), Ch 20 (BST ops) | Plain BST insert/search/delete | `TreeMap`, `TreeSet` |
| Composite designs (LRU, LFU…) | Ch 16 | All of them | — |
| Binary tree | Ch 19 | Tree from array + all traversals | — |
| Trie, binary trie | Ch 20 | Both | — |
| Graph (edge list, adjacency list/matrix) | Ch 21 | All three representations, BFS, DFS | — |
| Disjoint set union | Ch 24 | Full DSU | — |
| Fenwick tree, segment tree, sparse table | Ch 34 | All three | — |

---

## 4. The practice system (lean on purpose)

### 4.1 Three modes

| Mode | When | Rules |
| :--- | :--- | :--- |
| **Learn** | Chapter problems (labeled) | Timebox: Easy 15 min, Medium 40 min, Hard 75 min. When stuck, apply the unstuck moves (Ch 1 §7) *in writing* before reading any hint. |
| **Perform** | Mixed Sets, random unseen problems | No tags, no hints. Timebox: Medium 25 min, Hard 45 min. Submit-or-abandon at the limit; then study the solution (see "When you fail" below). |
| **Interview** | 1 mock every 2 weeks from Part II onward; weekly in Part VIII | 45 min, talk aloud, record yourself or use a partner. Plain editor, no autocomplete. |
| **Contest** | 1 LeetCode weekly contest per week from Part II onward (live or virtual) | This is your stream of unseen, unlabeled problems; a closed set can't supply those once you've seen it. Log contest problems like any other. |

**When you fail a problem.** The book gives nudges and a one-line key, not full solutions. If those aren't enough, the sanctioned next step is the problem's LeetCode editorial (or the CSES discussion), used like this:
1. Read only until you see the key idea, then close it.
2. Re-derive the rest yourself, with the protocol's Steps 4–8, in writing.
3. Code it from a blank file, and stress-test it (Ch 5 §6).
4. Log it with a failure tag, and add it to the re-solve schedule.

Copying a solution teaches nothing; re-deriving it from the idea is the lesson.

**Worked problems.** For every worked problem without a ★, stop after reading its statement and spend 5 minutes writing the brute force, its bottleneck and one candidate idea before reading on.

### 4.2 After every problem (2 minutes, non-negotiable)

Write two lines in your log (Appendix B):

1. **Key insight**, in one sentence: what property made it solvable?
2. **Signal**: what in the problem *should* have pointed you there?

If you failed, also tag **why**: `IDEA` (didn't find the approach), `PROOF` (believed a wrong approach), `IMPL` (approach right, code wrong), `EDGE` (missed a case), `TIME` (too slow to finish). Five tags only; more categories are bureaucracy.

### 4.3 Re-solve schedule

Every failed or hinted problem is re-solved **from a blank file**, without notes, after **3 days**, **2 weeks**, and **6 weeks**. If you fail a re-solve, the clock restarts. This is the single highest-leverage habit in the book.

### 4.4 Chapter exit gate

Move on only when you can:

1. **Rebuild** the chapter's core code from a blank file inside the stated time, compiling first try.
2. **Explain** the key property and its proof out loud in under 2 minutes, *including* the counterexample that breaks it.
3. **Solve** the chapter's practice set with ≥ 80% success unaided. The chapter's real test arrives later, in the next Mixed Set, where nothing tells you which chapter a problem is from.

### 4.5 Weekly rhythm (≈ 2 h/day on 6 days, plus the contest)

| Block | Time | Content |
| :--- | :--- | :--- |
| Re-solves | 15 min | Whatever the schedule says is due |
| Learn | 45 min | Read the chapter; attempt ★ problems before their sections |
| Practice | 50 min | Chapter practice problems from a blank file |
| Log | 10 min | Insight + signal lines |
| Contest day | 1.5 h + 30 min review | Weekly contest, then upsolve one problem you missed |

**Compression track (placement season in < 3 months, ~3 h/day):** Ch 1–6 → 7–13, 15 → 19–20, 21–24 → 26–29 → 38–40, with Mixed Sets II, IV, VI and practice problems only (no stretch). Skip all S-tier chapters. Do not skip Ch 2 or Ch 27; they carry everything else.

---

## 5. Chapter template (every technique chapter follows this)

| § | Section | Purpose |
| :-- | :--- | :--- |
| 0 | **From zero** *(chapters that introduce a structure)* | What the structure is, how it works inside, the cost of each operation and why; you implement it, then compare with Java's version. |
| 1 | **Attempt first** | A problem you try *before* reading on (timeboxed). |
| 2 | **Brute force → bottleneck** | The naive search space, its cost, and the precise repeated or wasted work. |
| 3 | **The key property** | The structural fact that removes the bottleneck, stated precisely and **proved**. |
| 4 | **Algorithm & Java** | Clean, overflow-safe code with invariants as comments and complexity stated. |
| 5 | **Where it breaks** | Concrete inputs that kill the technique, and what replaces it. |
| 6 | **Variations** | Constraint mutations an interviewer will throw (bigger n, online, reconstruct, count instead of optimise…). |
| 7 | **Recognition** | Structural signals that suggest the property, plus false friends that look similar but aren't. |
| 8 | **Implementation traps** | The specific bugs this technique invites. |
| 9 | **Practice ladder** | Core (unlabeled), stretch, nudges and a key in `<details>`. |
| 10–12 | **Self-test, traps, exit gate** | Hidden-answer questions, the bugs this technique invites, and the gate to move on (sections can appear in a slightly different order per chapter). |

---

## 6. Conventions

* **Java 17+** (LeetCode and most OA platforms support it; records need 16+). Where an OA platform is stuck on Java 8, the chapter says so.
* **Intervals are half-open `[l, r)`** unless stated otherwise. Prefix arrays have length `n + 1` with `pre[0] = 0`.
* **Complexities** are stated as worst-case unless marked *amortised* or *expected*.
* **Problem references** are LeetCode titles unless marked `CSES` (cses.fi/problemset). Search the exact title on the site; titles are unique.
* **Answers to self-tests** are inside collapsed `<details>` blocks. Commit to an answer before you open them.
