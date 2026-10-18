# Chapter 40 — The Live Interview Playbook

> **Read §1–9 in Week 8, before your first mock; use §10–11 for every mock; reread everything in Weeks 34–35** · Prerequisites: Ch 1 (the protocol), Ch 2 (proofs and counterexamples); Ch 38 for the final mocks
> **Drill:** 8 recorded mock interviews, graded on the §10 rubric · **Time:** about 10 hours

A live coding interview tests the same skills as an OA, plus one more: **making your reasoning visible**. The interviewer is not only checking whether your code works. They are collecting evidence about how you understand a problem, how you get from a brute force to something better, whether you can justify it, and whether you write and test code like an engineer. Silent, perfect code gives them less evidence than a clearly reasoned solution with one bug you find yourself.

This chapter covers the 45 minutes: what to do in each part, what to say, and how to handle hints, getting stuck and follow-up questions.

---

## 1. What is being graded

Companies word their rubrics differently, but almost all of them assess these five things:

| Dimension | Strong evidence | Weak evidence |
| :--- | :--- | :--- |
| **Understanding** | Restates the problem; asks the questions that matter; works an example | Starts coding from a misreading |
| **Problem solving** | Brute force first, names the bottleneck, derives the better approach with a reason | Jumps to a memorised solution and can't explain it |
| **Correctness** | States why the approach works, and checks a counterexample | "I think it works" |
| **Code** | Clear names, small helpers, handles edges, idiomatic Java | One long method, cryptic variables, dead code |
| **Verification** | Traces the code on an example, tests edge cases, finds their own bugs | Declares it done; the interviewer finds the bug |

**Communication** runs through all five rows. It isn't a separate act: it's how the interviewer sees the other four.

---

## 2. The 45 minutes

A typical slot: about 5 minutes of introductions, 35 of problem, and 5 for your questions. With one problem, pace it like this:

| Minute | Phase | You're done when |
| :-: | :--- | :--- |
| 0–5 | Introductions | You've given a 30-second summary of yourself and what you're working on |
| 5–10 | **Understand:** restate, ask clarifying questions, work an example | The interviewer agrees with your restatement and your example's answer |
| 10–18 | **Approach:** brute force and its cost, the bottleneck, the better approach, its correctness sentence and complexity | The interviewer agrees to the approach ("sounds good, go ahead") |
| 18–33 | **Code** | It's complete, including the edge cases you listed |
| 33–38 | **Test:** trace the example, then the edge cases | You've found and fixed your own bugs |
| 38–40 | **Complexity and follow-ups** | You've stated the final time and space |
| 40–45 | Your questions | — |

**Two problems in one slot** (common for easier problems): halve the understand and approach phases for the first one, and don't let it pass 20 minutes.

**If the approach phase runs long,** say so and ask: "I have an O(n log n) idea I'm fairly sure about and an O(n) one I'd need more time for. Shall I code the first?" Interviewers almost always prefer working code for a good approach over no code for the best one.

---

## 3. Understanding: the questions that matter

Ask only the questions whose answers change your solution. The constraints usually decide the approach (Ch 38 §2), so start there:
* **Sizes:** "How large can n get?" "How many queries?"
* **Values:** "Can values be negative? Zero? How large?" (This decides between a sliding window and prefix sums, and between `int` and `long`.)
* **Duplicates:** "Can there be repeated values?" "Are the IDs unique?"
* **Shape:** "Is the input sorted?" "Is the graph connected? Directed?" "Can the tree be unbalanced?"
* **Output:** "If there are several answers, any one?" "What should I return if there's none?"
* **Mutation:** "May I modify the input?" "Does it need to be in place?"

Then **work one example by hand, out loud**, and confirm the answer. Choose a small one that isn't trivial, ideally different from the given example. This catches misreadings in 60 seconds.

**If the interviewer says "you decide",** pick the reasonable assumption, say it, and write it as a comment at the top of your code.

---

## 4. Approach: thinking aloud

**Narrate decisions, not keystrokes.** "I'll loop over i" is noise. "I need, for each element, the nearest smaller one to its left; a monotonic stack gives that in O(n) total" is evidence.

A script that follows Ch 1's protocol:
1. **"The brute force is…"** One sentence, plus its complexity. "Try every pair, O(n²), which at n = 10⁵ is 10¹⁰: too slow."
2. **"The waste is…"** Name the bottleneck. "For each right end, I rescan all left ends looking for a matching prefix sum."
3. **"So I want…"** The query the bottleneck needs answered fast. "A way to ask: have I seen prefix sum P − k before, and how many times? A hash map of counts does it in O(1)."
4. **"It's correct because…"** One sentence. For greedy, the exchange argument; for DP, the state and the last decision; for binary search, the monotone property.
5. **"That gives O(…) time and O(…) space."** Then: **"Shall I code it?"**

**Ask for agreement before coding.** It's the cheapest moment to discover that the interviewer expected something else, or that you've missed a constraint.

**If you know the problem,** say so honestly, or at least don't fake a discovery. Then derive it properly anyway: the reasoning is what's being graded, and a memorised answer with no reasoning scores poorly.

---

## 5. Code

* **Structure first.** Write the method signature and the main loop, then fill in the helpers. Name the helpers by what they compute (`firstGreater`, `isFeasible`).
* **Names that carry meaning:** `left`, `windowSum`, `best`, `prefixCount`. Not `a`, `t`, `tmp2`.
* **Edge cases** you listed in §3: handle them at the top, or explain why the general code already handles them.
* **Idiomatic Java:** `ArrayDeque` for stacks and queues, `Map.getOrDefault` and `merge`, `Integer.compare` in comparators (Ch 4). Interviewers notice `Stack` and `a - b` comparators.
* **Keep talking, briefly.** Say what each block is for as you start it: "Now the shrinking step: while the window is invalid, move left."
* **Don't optimise prematurely.** Clean O(n log n) beats a tangled O(n) unless O(n) was asked for.
* **If you realise mid-code that the approach is wrong,** stop and say so. Explain the counterexample, and go back to the approach phase. Discovering it yourself is good evidence; hiding it isn't.

---

## 6. Test: find your own bugs

When the code is written, say "Let me test it", and do it properly:
1. **Trace the example** through the code, line by line, tracking the variables in a comment or out loud. Don't trace what you *meant*; trace what you *wrote*.
2. **Edge cases:** an empty or single-element input, all equal values, the extreme values, the "no answer" case.
3. **A case aimed at your logic:** the boundary of the window, the tie in the comparator, the first and last iteration of the loop.

**Fix bugs calmly and say what was wrong.** "The window shrinks one step too late because I check before adding; I'll move the check." Interviewers grade bug-finding highly, especially when it's *you* who finds it.

---

## 7. Hints

A hint is the interviewer helping you produce evidence. Using it well is a skill and is scored; ignoring it is scored too, negatively.

| The hint sounds like | It means | Respond by |
| :--- | :--- | :--- |
| "What's the time complexity of that?" | It's too slow, or you haven't said | Computing it with the real n; if it's too slow, naming the bottleneck |
| "What if the input were sorted?" | Sorting (or order) is the key | Asking whether sorting is allowed, and what it gives you |
| "Do you need to look at every element again?" | There's repeated work | Naming what's recomputed, and storing it |
| "What about this case: …?" | There's a bug or a wrong approach | Running that case through your approach or code, out loud |
| "Is there a data structure that…?" | They have one in mind | Listing the operations you need, then the structure that supports them |

**How to take a hint:** repeat it in your own words, connect it to your current approach ("so if it's sorted, the pairs that work form a contiguous range, and I can use two pointers"), and continue. Don't argue with it, and don't stall waiting for a bigger one.

---

## 8. When you're stuck

Silence is the worst outcome: it gives no evidence at all. When stuck:
1. **Say where you are.** "I have the O(n²) solution. I'm trying to avoid rescanning the left part, but I don't see what to store yet."
2. **Use the unstuck moves aloud** (Ch 1 §7, Ch 38 §6): small cases, fix something, reverse the problem, change the representation.
3. **Offer the fallback.** "I could code the O(n²) version now and optimise after. Would that be useful?"
4. **Ask a targeted question,** not "can I have a hint?" but "is the key to process the elements in some particular order?"

---

## 9. Follow-up questions

After the main problem, many interviewers change one condition. These are the common ones, with where the answer lives:

| Follow-up | Usually leads to |
| :--- | :--- |
| "What if the input is a **stream**?" | Online structures: a heap, a running state, a sliding window; no sorting the whole input (Ch 11, 15) |
| "What if there are **many queries**?" | Precompute: prefix sums, sorting plus binary search, a Fenwick tree or a lifting table (Ch 8, 10, 34, 36) |
| "What if memory is **O(1)**?" | Two pointers, in-place tricks, bit manipulation (Ch 5, 11, 17) |
| "What if it **doesn't fit in memory**?" | Process in chunks: external sort and merge (k-way merge with a heap, Ch 15), hashing into buckets |
| "**k** instead of 2?" | Generalise: k pointers become a heap; pairs become a DP dimension |
| "Return **all** solutions / the **count** / **one**" | Backtracking / DP / early exit (Ch 6, 27) |
| "What if values can be **negative**?" | Windows stop working: prefix sums, deques, a different invariant (Ch 8, 12) |
| "What if the graph has **weights** / **negative** weights?" | Dijkstra / Bellman–Ford (Ch 23) |
| "How would you **test** this?" | The brute force + random comparison of Ch 5 §6 |
| "What if this ran as a **service** under heavy load?" | Precomputation and caching (an LRU cache, Ch 16), splitting reads from writes, what consistency you need |

The last row is where your backend engineering strengths help. Answer at the level of data structures and complexity first, then mention the system aspects briefly, unless the interviewer wants to go deeper.

---

## 10. The mock rubric

Score each recorded mock 1–4 on each dimension, from the recording, not from memory.

| Dimension | 1 | 2 | 3 | 4 |
| :--- | :--- | :--- | :--- | :--- |
| **Understanding** | Coded from a misreading | Restated, but missed a key constraint | Restated, asked the key questions | Also worked a non-trivial example before the approach |
| **Problem solving** | No approach | A solution without a derivation | Brute → bottleneck → better approach | Same, plus explained why simpler alternatives fail |
| **Correctness argument** | None | "It works on the example" | A one-sentence justification | A justification plus a counterexample checked against it |
| **Code** | Didn't finish | Finished, messy, needed big fixes | Clean, minor fixes | Clean, idiomatic, edge cases handled first time |
| **Verification** | None | Ran the example only | Traced example + edge cases | Found their own bug, or showed convincingly there wasn't one |
| **Communication** | Long silences | Narrated keystrokes | Narrated decisions | Narrated decisions, used hints well, managed time |

**Target:** 3 or more on every dimension in at least six of the eight mocks, with no 1s in the last four.

---

## 11. The eight recorded mocks

**Format.** 45 minutes each, one or two per week in Phase 6 (and every two weeks before that, per the curriculum). Record your screen and voice. Use a plain editor (curriculum §4.1, Interview mode).

**Choosing problems.** A partner picks them, or you take them from your **unseen contest backlog** (problems from contests you haven't done, so you don't recognise them). Aim for one Easy-Medium warm-up and three Mediums across the first four mocks, and Medium-Hard with follow-ups in the last four.

**With a partner:** they play the interviewer, give hints only when you're stuck for 3 minutes or more, and ask one follow-up from §9. **Alone:** read the problem with the timer already running, and speak as if someone were listening. It feels awkward for the first two mocks and normal after that.

**Review** (30 minutes, within a day): watch the recording at 1.5× speed, score the §10 rubric, and write in your log:
* the one dimension that scored lowest, and the moment in the recording that shows why;
* one concrete change for the next mock ("ask about negatives before the approach", "trace the code I wrote, not the code I meant").

**Trends to act on:** long silences → practise the §4 script on old problems, aloud. Bugs found by the "interviewer" → the §6 routine isn't being run. Running out of time → the approach phase is too long, so offer the fallback earlier.

---

## 12. Exit gate

Done when:

* you can run the 45-minute structure without looking at this chapter, including asking for agreement before coding;
* your last four mocks score 3 or more on every rubric dimension;
* you can answer each §9 follow-up for at least one problem from each Part of this handbook;
* the curriculum §1 benchmark holds: brute force → optimal derived out loud, clean code, self-tested, follow-up handled.
