# Appendix B — The Practice Log

> The log is what turns solved problems into skill. Curriculum §4 sets the rules: two lines after every problem, a failure tag when it went wrong, and re-solves at 3 days, 2 weeks and 6 weeks. This appendix gives the formats, worked examples, and the weekly routine that uses them.

**Where to keep it:** one plain-text or Markdown file per month (`log_2026-10.md`), or one spreadsheet with the columns of §1. Choose whichever you'll actually open every day. The format matters less than the habit. Don't build tooling for it; that's procrastination.

---

## 1. The problem entry

Write it within 2 minutes of finishing (or abandoning) a problem, while the memory is fresh.

```
## 2026-11-18 · Subarray Sum Equals K · Ch 8 · Learn · 22 min · solved
Insight: count earlier prefixes equal to prefix - k; the empty prefix counts once.
Signal: "number of subarrays with sum k" + negatives allowed -> no window; two-prefix property.
Re-solve: -
```

```
## 2026-12-09 · Minimum Window Substring · Ch 11 · Learn · 75 min · hinted
Insight: grow until valid, then shrink while still valid; track "letters still missing" as one counter.
Signal: "smallest window containing all of t" -> valid windows are closed under growing.
Tag: IMPL (shrank before recording the answer)
Re-solve: 2026-12-12, 2026-12-23, 2027-01-20
```

**The fields:**

| Field | What to write |
| :--- | :--- |
| Date · problem · chapter · mode · minutes · outcome | Mode is Learn, Perform, Interview or Contest (curriculum §4.1). Outcome is `solved`, `hinted`, `failed` or `partial` |
| **Insight** | One sentence: the **property** that made it solvable. Not "use a hash map", but "count earlier prefixes equal to prefix − k" |
| **Signal** | One sentence: what in the **statement or constraints** should point you there next time. This is the line that builds recognition |
| **Tag** | Only if not solved cleanly: `IDEA`, `PROOF`, `IMPL`, `EDGE` or `TIME`, plus a few words on what exactly went wrong |
| **Re-solve** | For every `hinted`, `failed` or `partial` entry: the three dates, 3 days, 2 weeks and 6 weeks later |

**Writing a good insight line.** Bad: "Sliding window." Better: "Sliding window works because a window containing all letters stays valid when it grows." Best: the better version **plus** the fact that makes it break ("…and negatives would break it, but counts can't be negative"). If you can't write the insight in one sentence, you haven't understood the solution yet. Re-derive it before moving on.

**Writing a good signal line.** It must be checkable on an unseen problem. "It's a window problem" isn't a signal. "'Smallest contiguous range containing all of X' plus non-negative counts" is.

---

## 2. Failure tags: what each one tells you to do

| Tag | It means | The fix | Where |
| :--- | :--- | :--- | :--- |
| `IDEA` | You didn't find the approach | Re-derive it with the protocol; add the signal to your Ch 38 notes; re-solve on schedule | Ch 1, Ch 38 |
| `PROOF` | You believed a wrong approach (usually a greedy) | Find the smallest counterexample and write it down next to the entry | Ch 2 |
| `IMPL` | The approach was right, the code wasn't | Note the exact bug; if it's a template, rebuild the template three times this week | Appendix A |
| `EDGE` | You missed a case (empty, equal values, overflow, a boundary) | Add the case to your personal audit list (start from Ch 39 §5) | Ch 5, Ch 39 |
| `TIME` | You ran out of time | Note which phase took too long: reading, idea, coding or debugging | Ch 39 §3, Ch 40 §2 |

**One tag per entry,** the earliest thing that went wrong. If you had the wrong idea and also a bug, it's `IDEA`: the bug is downstream.

---

## 3. The re-solve queue

Every `hinted`, `failed` or `partial` problem gets three re-solve dates. **Rules** (curriculum §4.3):
* Re-solve **from a blank file**, without looking at your old code or notes, in Perform mode (timed, no hints).
* **Pass** = solved within the time box. Then cross off that date.
* **Fail** = the clock restarts: three new dates counted from today, and a new tag on the entry.

**Finding what's due:** at the start of each session (the first 15 minutes, curriculum §4.5), search the log for today's date or earlier in the `Re-solve:` lines. In a spreadsheet, filter the next-due column to ≤ today.

**Keep the queue honest.** If more than 10 re-solves are overdue, stop new chapter work for a day and clear them. The queue is where the forgetting curve gets beaten; skipping it makes everything else leak.

---

## 4. Contest, mock OA and mock interview entries

**Weekly contest:**

```
## 2026-12-13 · Weekly Contest 4xx · 3/4 solved · rank 2140
Q1 · 4 min · solved
Q2 · 11 min · solved
Q3 · 34 min · solved · Insight: ... · Signal: ...
Q4 · not solved · Tag: IDEA · upsolved next day · Insight: ... · Signal: ... · Re-solve: dates
Takeaway: spent 15 min on Q3's wrong greedy before the counterexample; run the 3-element check earlier.
```

**Mock OA** (Ch 39 §9): the per-problem table of triage, times, result and tag, then one line on the checkpoints ("met / missed at minute __ because __").

**Mock interview** (Ch 40 §11): the rubric scores (Understanding, Problem solving, Correctness, Code, Verification, Communication, each 1–4), the lowest-scoring moment with its timestamp in the recording, and one concrete change for next time.

---

## 5. The weekly review (Day 7, 20 minutes)

1. **Read the week's insight and signal lines,** all of them. This is the most valuable 10 minutes of the week: it's spaced repetition for recognition.
2. **Count the tags.** Two or more of the same tag is a pattern:
   * `IDEA` clustered in one chapter → redo that chapter's self-test and two practice problems;
   * `IMPL` → rebuild the relevant templates;
   * `EDGE` → extend your audit list;
   * `TIME` → do one session in strict Perform mode with checkpoints.
3. **Check the queue:** how many re-solves are due next week? Plan them first.
4. **One line for the week:** what improved, and what to change.

```
## Week of 2026-12-07 · review
Problems: 14 (11 solved, 2 hinted, 1 failed). Contest: 3/4.
Tags: IDEA ×2 (both Ch 11, "window validity not monotone" missed), IMPL ×1.
Action: redo Ch 11 self-test; add "check monotone validity before using a window" to the audit list.
Re-solves due next week: 4.
```

---

## 6. Progress tracker

Tick each item when its **exit gate** is met, not when the chapter has been read. Copy this table into your first log file.

| Part | Items |
| :--- | :--- |
| 0 · Method | ☐ Ch 1 · ☐ Ch 2 · ☐ Ch 3 |
| I · Foundations | ☐ Ch 4 · ☐ Ch 5 · ☐ Ch 6 |
| II · Linear | ☐ Ch 7 · ☐ Ch 8 · ☐ Ch 9 · ☐ Ch 10 · ☐ Ch 11 · ☐ Ch 12 · ☐ Ch 13 · ☐ Ch 14 · ☐ Ch 15 · ☐ Ch 16 · ☐ Mixed Set II |
| III · Bits & Math | ☐ Ch 17 · ☐ Ch 18 · ☐ Mixed Set III |
| IV · Trees | ☐ Ch 19 · ☐ Ch 20 · ☐ Mixed Set IV |
| V · Graphs | ☐ Ch 21 · ☐ Ch 22 · ☐ Ch 23 · ☐ Ch 24 · ☐ Ch 25 · ☐ Mixed Set V |
| VI · Greedy & DP | ☐ Ch 26 · ☐ Ch 27 · ☐ Ch 28 · ☐ Ch 29 · ☐ Ch 30 · ☐ Ch 31 · ☐ Ch 32 · ☐ Ch 33 · ☐ Mixed Set VI |
| VII · Advanced | ☐ Ch 34 · ☐ Ch 35 · ☐ Ch 36 · ☐ Ch 37 · ☐ Mixed Set VII |
| VIII · Performance | ☐ Ch 38 drill ≥ 34/40 · ☐ Mock OAs (4 of 6 at target) · ☐ Mock interviews (last 4 at ≥ 3 everywhere) · ☐ Appendix A rebuild, all templates |

**Checkpoints** (00_schedule.md): ☐ 24 Jan (Mixed Set II ≥ 70%) · ☐ 21 Mar (Mixed Set V ≥ 70%, contests 3/4 most weeks) · ☐ 20 Jun (curriculum §1 benchmarks).

---

## 7. What not to log

* **Full solutions.** The code lives in your solutions folder; the log holds the *reasons*.
* **Easy problems solved in a few minutes with nothing learned.** A one-line entry (`date · problem · solved · 6 min`) is enough.
* **Feelings without actions.** "Bad day" is not an entry. "Spent 40 min on Q2 because I didn't check for negatives" is.
