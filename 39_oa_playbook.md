# Chapter 39 — The OA Playbook

> **Read §1–6 in Week 5, before your first contest; reread everything in Weeks 34–35 for the mock OAs** · Prerequisites: Ch 4 (program shapes, FastReader), Ch 5 §6 (stress testing); Ch 38 for the mock OAs
> **Drill:** 6 timed mock OAs with post-mortems · **Time:** about 12 hours, spread over two weeks

An online assessment (OA) is the first filter at most companies, and at college placements it's often the only coding round before the interviews. It differs from an interview in three ways that change how you should play it:
* **Nobody watches you think.** Only the tests that pass count. A brilliant idea with a bug scores zero; a correct brute force can score half.
* **You can't see the failing tests.** You get "wrong answer on hidden test 7" at best, sometimes just a final score.
* **The clock covers everything.** Reading, choosing, coding and debugging all come out of the same 60–120 minutes.

This chapter is the procedure for those conditions. It doesn't teach new techniques; Ch 1–38 did that. It teaches you to convert what you know into **points** under time pressure.

**Integrity:** OAs are usually proctored, and their rules state what's allowed (a local IDE, documentation, nothing else). Follow them exactly. Everything in this chapter works within the strictest version of those rules.

---

## 1. Before the OA

**Know the format.** Check, or ask the recruiter or seniors who took it last year:
* the platform, and its **Java version**;
* the number of problems and the total time;
* **the program shape** (Ch 4 §1): a function stub you fill in, or a full program reading standard input. Some platforms give a stub with the input already parsed;
* **how it's scored**: by the fraction of tests passed (the most common), with correctness and performance reported separately, or with time spent as a factor. The instructions page states it; read it.

**Prepare the environment.**
* **Practise in a plain editor**, without autocomplete, for the last two weeks. Some platforms' editors only highlight syntax.
* Know your **imports** by heart: `java.util.*` and `java.io.*` cover everything in this handbook.
* Have **FastReader** (Ch 4 §2.2) memorised; rewrite it once a week. For stub-style platforms, it's not needed.
* Decide in advance what you'll do if the platform allows a local IDE: test there, then paste back. If it doesn't, the plain-editor drills (§8) are what make that comfortable.

**The night before:** no new topics. Re-read your log's signal lines for your three weakest chapters.

---

## 2. The first five minutes: triage

**Read every problem before coding any.** Spend about a minute on each: the statement, the constraints, and the example. For each one, write down:
* a **difficulty guess**: E (you see the solution now), M (you see a direction), H (no idea yet);
* the **budget** from the constraints (Ch 38 §2);
* a first candidate technique, if any.

**Then choose the order:**
1. **E problems first**, quickest first. Banking points early takes the pressure off everything after.
2. Then **M problems**, in order of how confident your candidate is.
3. **H problems last.** On most OAs the last problem is the hardest, but not always; trust your triage, not the numbering.

**Don't fall in love with one problem.** If a problem has taken its planned time (§3) and isn't close, bank its partial credit (§4) and move on. Come back if time remains.

---

## 3. Time checkpoints

Plan the clock before you start, and write the checkpoints down.

**A 90-minute OA with 4 problems** (typical: E, M, M, H):

| By minute | You should have |
| :-: | :--- |
| 5 | Read all four; order chosen |
| 15 | The E problem accepted |
| 40 | The first M problem accepted, or a brute force submitted for it |
| 65 | The second M problem accepted, or a brute force submitted |
| 80 | A brute force or partial solution for the H problem submitted |
| 80–90 | The audit (§5) on anything not fully passing; improvements to the best partial |

**A 60-minute OA with 2 problems** (typical: M, H): 3 minutes to read both, about 25 on the M, about 25 on the H, and 5 for the audit and resubmission.

**Falling behind a checkpoint** means: stop optimising, submit what you have, and move to the next problem. The checkpoints exist to stop a single problem from eating the whole OA, which is the most common way strong candidates fail.

---

## 4. Partial credit: always submit something that passes some tests

Hidden test suites almost always include **small** cases. A correct brute force passes those.

**The rule:** if the optimal solution isn't clear within about 10 minutes, **write the brute force, submit it**, and then think about the optimal one with points already banked. The brute force also becomes your stress-test oracle (Ch 5 §6) if you do find the optimal solution.

**Ways to get more than the brute force's share:**
* **Special-case the structure.** Is the input sorted in some tests? Are all values equal, or is k = 1? A special case you can solve fast may cover a block of tests.
* **Split by size.** `if (n <= 2000) bruteForce() else heuristicOrGreedy()`. The large tests then have *some* chance.
* **Prune the brute force** (Ch 6): ordering choices, memoising obvious repeats, early exits. That often doubles the n it can handle.
* **A greedy you can't prove.** In an interview, an unproven greedy is a risk. In an OA, after the brute force is submitted, it's a free attempt. Only submit it if it agrees with the brute force on random small inputs (a 3-minute stress test).

**What not to do:** leave a problem blank because the optimal solution was out of reach, or spend 30 minutes on a submission that can only reach the same score as a 5-minute brute force.

---

## 5. The pre-submit audit

Run this list **before every submission**. It takes about 60 seconds and catches the bugs that hidden tests are designed to find.

| Check | The bug it catches |
| :--- | :--- |
| **Overflow:** products, sums of many values, counts of pairs. Is anything near 2·10⁹? | `int` wraparound (Ch 4) |
| **Edge sizes:** n = 0, n = 1, all elements equal, the maximum n | Empty loops, index −1, off-by-one at the end |
| **Negative values, zeros, duplicates** | Wrong comparisons; a sliding window that assumed positives |
| **Division and mod:** integer division rounding, `%` of a negative number, mod after subtraction | `(a − b) % M < 0` (Ch 18) |
| **The output format:** spaces vs newlines, trailing spaces, upper/lower case, "−1" vs "NO" | A correct answer marked wrong |
| **Multiple test cases:** is every static array and counter **reset** between tests? | State leaking from test 1 into test 2 |
| **Reading:** is the whole input consumed, in the right order? | Misaligned reads after a skipped line |
| **Complexity at the maximum n:** recompute it with the real numbers (Ch 1 Step 8) | Time limit exceeded on the big tests |
| **Recursion depth** at the maximum n | `StackOverflowError` on a path-shaped input (Ch 4 §16) |
| **Debug output removed** | Extra lines in the output |

---

## 6. When a hidden test fails

You can't see the test, so **generate one**. This is where Ch 5's stress harness pays for itself:
1. Write a brute force (you may already have one from §4).
2. Write a random generator for **small** inputs (n ≤ 8, values ≤ 10).
3. Compare the two in a loop, and print the first input where they differ.

In a local IDE, this takes 5–7 minutes and finds almost every wrong-answer bug. Without one, work through the audit (§5) again, then hand-run the tiny cases (n = 1, 2) and an adversarial case from Ch 1 Step 3.

**Reading the verdict:**

| Verdict | Usual causes |
| :--- | :--- |
| Wrong answer on early tests | A misread statement: reread it, slowly, against your restatement |
| Wrong answer on late tests only | Overflow, an edge case at large n, or a greedy that fails on some structure |
| Time limit on late tests | Wrong complexity, `HashMap` with boxed keys in a hot loop, slow input or output (use FastReader and a StringBuilder), string concatenation in a loop |
| Runtime error | An index out of bounds at an edge size, recursion depth, or a null from a map lookup |
| Memory limit | An n × n table where n is 10⁵: roll the DP (Ch 27), or store less |

---

## 7. Platform notes

Platforms differ in their editors and scoring, and change both over time, so treat these as things to **check**, not facts to rely on:
* **Stub vs full program.** Some give a function to complete with the parsing done for you; others give an empty `main`. Check whether you may change the class name or the method signature (usually not).
* **Custom input.** Most let you run your code on input you type. Use it for the tiny and edge cases before submitting.
* **Submission limits and penalties.** Some count every submission, some only the last. Read the instructions.
* **Scoring.** By tests passed, by correctness and performance separately, or including time. If performance is scored separately, a correct but slow solution still earns its correctness share, which is another reason to submit the brute force first.
* **Language version.** Older Java versions lack some newer methods and syntax (records, `var`, `List.of`). If the platform's Java is old, avoid them; Ch 4 §26 lists what to check.

---

## 8. Plain-editor drills

Once a week during Phase 6, in an editor with no autocomplete and a 10-minute timer each, write from memory and run:
* FastReader plus a program that reads T test cases of an array and prints their sums;
* BFS on a grid, Dijkstra with a priority queue, and DSU;
* binary search on the answer, with its feasibility function;
* a Fenwick tree *(S)*, and a 1D DP with reconstruction.

Each must compile on the first try. The Appendix A templates are the reference versions.

---

## 9. The mock OAs

**Six mock OAs**, two or three a week in Weeks 34–35. Each one is an **unseen LeetCode virtual contest**: 90 minutes, 4 problems, plain-editor rules, with no breaks, no notes and no hints, exactly like the real thing. Pick contests you haven't seen; your contest backlog (curriculum §4.1) is the source.

**The post-mortem** (30 minutes, straight after; write it in your log):

| For each problem | Record |
| :--- | :--- |
| Triage | Your difficulty guess vs reality |
| Time | Minutes to read, to the idea, to the first submission, to acceptance |
| Result | Accepted / partial (what fraction) / not attempted |
| Failure tag | IDEA, PROOF, IMPL, EDGE or TIME (curriculum §4.2) |
| The fix | Which section of this chapter, or which Ch 38 signal, would have saved the most time |

**Then, for the whole OA:** were the checkpoints met? If not, which problem broke them, and why? Upsolve every problem you didn't get, from a blank file, and add it to the re-solve schedule.

**What to look for across the six:**
* **IDEA tags** on M problems: back to Ch 38's drill, and the chapters behind the misses.
* **IMPL / EDGE tags:** the audit isn't being run, or a template isn't solid. Drill it.
* **TIME tags:** the triage or the checkpoints aren't being respected.

**The target** is curriculum §1's benchmark: **at least 3 of 4 fully solved, with partial credit on the rest**, in four of the six mocks.

---

## 10. Exit gate

Done when you can:

* triage four problems in 5 minutes and set checkpoints for the remaining time;
* produce a submitted brute force within 10 minutes for any problem you understand;
* run the §5 audit from memory;
* set up a stress test from a blank file in under 7 minutes;
* meet the mock-OA target in four of the six mocks.
