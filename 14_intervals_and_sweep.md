# Chapter 14 — Intervals & Sweep Line

> **Part II · Week 11** · Prerequisites: Ch 2 (exchange arguments), Ch 4 §22 (`TreeMap`), Ch 8 (difference arrays), Ch 9, Ch 11
> **Problems:** 6 worked (1 ★) · 7 practice · 3 stretch · **Time:** about 7 hours

Meetings, bookings, time ranges, segments on a line: interval problems are everywhere in interviews, and especially in backend-flavoured ones (calendars, rate limits, resource allocation). Almost all of them come down to **one decision: what do you sort by, and why?** Sorting by start time and sorting by end time solve *different* problems. Picking the wrong one gives a plausible algorithm that fails. This chapter gives each choice its proof, then adds the **sweep line**: walk along the timeline processing "starts" and "ends" as events, which turns many interval questions into a counting problem.

---

## 0. From zero: intervals and overlap

An interval `[s, e]` is every point from `s` to `e`. In code it's an `int[] {s, e}`. **Two intervals overlap exactly when neither one ends before the other starts.** The easiest way to get that right is to write the *opposite* first: `a` and `b` don't overlap if `a` ends before `b` starts, or `b` ends before `a` starts. Negating that gives

> `a` and `b` overlap ⟺ `a.s ≤ b.e` **and** `b.s ≤ a.e`.

**Decide whether touching counts.** Do `[1, 4]` and `[4, 5]` overlap? It depends on the problem:
* For closed intervals of *points* (both share the point 4), use `≤` as above.
* For *time ranges* where one meeting ends exactly when the next begins, they don't conflict. Those are half-open intervals `[s, e)`, and the test becomes `a.s < b.e && b.s < a.e`.

Read the statement's examples to find out which one applies, and write it as a comment. **Most interval bugs are this one decision.**

---

## 1. Attempt first ★

> **[Merge Intervals](https://leetcode.com/problems/merge-intervals/) (20 minutes).** Merge all overlapping intervals and return the result. `[[1,3],[2,6],[8,10],[15,18]]` → `[[1,6],[8,10],[15,18]]`. Here `[1,4]` and `[4,5]` *do* merge.

---

## 2. Sort by start: merging and coverage

### Worked P1 · [Merge Intervals](https://leetcode.com/problems/merge-intervals/)

Comparing every pair and merging repeatedly can take O(n²) or worse. **Sort by start.** Then a new interval can only overlap the **most recently merged** interval, never an earlier one:

> **Invariant:** after processing the first i intervals (sorted by start), `merged` holds disjoint intervals in increasing order, and only the **last** one can still grow.
>
> *Why earlier ones are final:* interval i + 1 starts at or after every earlier start. If it doesn't overlap the last merged interval, which ends at least as late as any earlier one, it can't overlap anything before that. And every later interval starts even further right.

```java
public int[][] merge(int[][] intervals) {
    int[][] sorted = intervals.clone();
    Arrays.sort(sorted, (a, b) -> Integer.compare(a[0], b[0]));
    List<int[]> merged = new ArrayList<>();
    for (int[] cur : sorted) {
        int[] last = merged.isEmpty() ? null : merged.get(merged.size() - 1);
        if (last != null && cur[0] <= last[1]) {          // overlaps (touching counts here): extend the last one
            last[1] = Math.max(last[1], cur[1]);          // max: cur might lie entirely inside last
        } else {
            merged.add(new int[] {cur[0], cur[1]});       // a copy, so the caller's input isn't modified
        }
    }
    return merged.toArray(new int[0][]);
}
```

O(n log n) for the sort, then O(n). The `Math.max` handles an interval nested inside the last one: `[1, 10]` followed by `[2, 3]` must stay `[1, 10]`.

**Sort by start whenever the question is about the *union* or *coverage* of intervals:** merging, inserting into a sorted list, total length covered, gaps between meetings, whether a range is completely covered.

---

## 3. Sort by end: choosing the most non-overlapping intervals

### Worked P2 · [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)

> Return the minimum number of intervals to remove so that the rest don't overlap. Touching (`[1,2]` and `[2,3]`) is allowed.

Removing as few as possible means **keeping as many as possible**, so find the largest set of pairwise non-overlapping intervals. Ch 2 §8.4 showed two plausible greedy rules that fail: earliest start (`[1,10]` blocks `[2,3]` and `[4,5]`) and shortest interval. The rule that works is:

> **Repeatedly keep the interval that ends earliest among those that don't overlap what's already kept.**

**Exchange proof.** Let `g` be the interval that ends earliest overall, and take any optimal selection O. Let `o` be the interval in O that ends earliest. Since `g` ends no later than `o`, swapping `o` for `g` can't create an overlap: everything else in O starts after `o` ends, so it also starts after `g` ends. The swapped set is still optimal and contains `g`. Repeat the argument on the intervals that start after `g` ends. ✓

**Why ending early is the right thing to optimise:** the end of the last kept interval is the only thing that limits future choices. Ending earlier leaves at least as much room.

```java
public int eraseOverlapIntervals(int[][] intervals) {
    int[][] sorted = intervals.clone();
    Arrays.sort(sorted, (a, b) -> Integer.compare(a[1], b[1]));    // by END
    int kept = 0;
    long lastEnd = Long.MIN_VALUE;                                   // nothing kept yet
    for (int[] cur : sorted) {
        if (cur[0] >= lastEnd) {         // starts at or after the last kept end: compatible (touching allowed)
            kept++;
            lastEnd = cur[1];
        }
    }
    return intervals.length - kept;
}
```

`lastEnd` starts as a `long` sentinel below every possible start, so the first interval is always kept without a special case.

**Sort by end whenever you choose a maximum set of compatible intervals, or the fewest points that hit every interval.** Minimum Number of Arrows to Burst Balloons (practice) is the same proof: shoot each arrow at the earliest end among the remaining balloons.

| Question | Sort by | Why |
| :--- | :--- | :--- |
| Merge, union, total coverage, gaps, insert | **start** | A new interval can only touch the most recent merged one |
| The most compatible intervals, the fewest hitting points | **end** | Ending early leaves the most room (exchange) |
| How many overlap at once; the fewest rooms or groups | **events** (§4) | It's a running count over time |
| The most *weight* of compatible intervals | — | Greedy fails; DP + binary search (Ch 33) |

---

## 4. The sweep line: counting overlaps over time

Imagine a vertical line sweeping left to right along the timeline. Each interval becomes two **events**: it becomes active at its start and inactive after its end. The number of **active** intervals at any moment is a running sum of +1s and −1s. That's a difference array (Ch 8 §5), evaluated only at event points, so huge coordinates don't matter.

### Worked P3 · [Divide Intervals Into Minimum Number of Groups](https://leetcode.com/problems/divide-intervals-into-minimum-number-of-groups/)

> Split closed intervals into the fewest groups so that no two intervals in a group intersect (sharing an endpoint counts as intersecting). This is the classic "minimum meeting rooms" problem.

**Lower bound.** If k intervals all contain some point x, they must be in k different groups. So we need at least `maxActive` groups, the largest number of intervals covering any single point.

**It's achievable.** Process the intervals by start time, and give each one any group whose last interval has already ended. A new group is opened only when every existing group is busy at this interval's start, which means the number of groups never exceeds the number of intervals active at that moment. So **the answer is exactly `maxActive`**.

Computing `maxActive` doesn't even need the groups. Sort the starts and the ends separately, then sweep:

```java
public int minGroups(int[][] intervals) {
    int n = intervals.length;
    int[] starts = new int[n], ends = new int[n];
    for (int i = 0; i < n; i++) { starts[i] = intervals[i][0]; ends[i] = intervals[i][1]; }
    Arrays.sort(starts);
    Arrays.sort(ends);
    int active = 0, best = 0;
    for (int i = 0, j = 0; i < n; i++) {
        while (ends[j] < starts[i]) {     // closed intervals: one ending exactly at starts[i] still overlaps it
            active--;
            j++;
        }
        active++;                         // interval number i (in start order) begins
        best = Math.max(best, active);
    }
    return best;
}
```

Sorting the starts and ends *independently* is safe, because the count of active intervals at time x only depends on how many starts are ≤ x and how many ends are < x, not on which end belongs to which start. For **half-open** time ranges, where a meeting ending at 5 frees its room for one starting at 5, change the condition to `ends[j] <= starts[i]`.

The same sweep answers "**how many intervals cover point x?**" for many query points (Number of Flowers in Full Bloom, practice): the answer is the number of starts ≤ x minus the number of ends < x, each found by binary search in the sorted arrays (Ch 10).

---

## 5. Two sorted lists

### Worked P4 · [Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/)

> Two lists of closed intervals, each sorted and internally disjoint. Return their intersections.

Use two pointers (Ch 11). The intersection of `a = A[i]` and `b = B[j]` is `[max(a.s, b.s), min(a.e, b.e)]`, if that's non-empty. Then **advance whichever interval ends first.** It can't intersect anything later in the other list, because everything there starts after the current interval of that list, which already reaches past this end. That's the elimination argument again.

```java
public int[][] intervalIntersection(int[][] first, int[][] second) {
    List<int[]> result = new ArrayList<>();
    int i = 0, j = 0;
    while (i < first.length && j < second.length) {
        int lo = Math.max(first[i][0], second[j][0]);
        int hi = Math.min(first[i][1], second[j][1]);
        if (lo <= hi) result.add(new int[] {lo, hi});     // closed: a single shared point counts
        if (first[i][1] < second[j][1]) i++; else j++;     // the one ending first is finished
    }
    return result.toArray(new int[0][]);
}
```

---

## 6. Intervals that arrive one at a time: an ordered map

When intervals arrive **online** (book this meeting, then answer the next request), you can't sort everything up front. Keep the accepted intervals in a `TreeMap` from start to end. The *neighbours* of a new interval are then one `floorEntry` and one `ceilingEntry` away (Ch 4 §22).

### Worked P5 · [My Calendar I](https://leetcode.com/problems/my-calendar-i/)

> Implement `book(start, end)` for half-open `[start, end)`: accept it and return true if it overlaps no existing booking; otherwise return false.

The accepted bookings never overlap each other, so a new booking can only conflict with **the booking starting just before it, or the one starting just after it**:

```java
class MyCalendar {
    private final TreeMap<Integer, Integer> booked = new TreeMap<>();   // start -> end, pairwise disjoint

    public boolean book(int start, int end) {
        Map.Entry<Integer, Integer> before = booked.floorEntry(start);    // the latest booking starting <= start
        if (before != null && before.getValue() > start) return false;    // it hasn't ended yet when we start
        Integer after = booked.ceilingKey(start);                         // the earliest booking starting >= start
        if (after != null && after < end) return false;                   // it starts before we end
        booked.put(start, end);
        return true;
    }
}
```

O(log n) per booking. Note the `Integer` and `null` checks (Ch 4 §22's null trap). For "**at most two** overlaps allowed" (My Calendar II, practice), keep a `TreeMap` of +1/−1 events instead, and sweep it to check the maximum count. That's §4 done online.

---

## 7. Sweeping with an active set

Sometimes you need more than a *count* of the active intervals: you need their maximum, minimum or some other summary. Keep the active intervals in an **ordered multiset** (Ch 4 §22), adding at each start and removing at each end.

### Worked P6 · [The Skyline Problem](https://leetcode.com/problems/the-skyline-problem/)

> Each building is `[left, right, height]`. Return the "key points" of the skyline: each `[x, h]` where the outline's height changes to h.

**Sweep the x-coordinates.** At each building edge the set of buildings covering the next stretch changes, and the skyline's height is the **maximum height among them**. Use a `TreeMap` from height to count as a multiset, so the maximum is `lastKey()`. At each x, apply **every** event at that x, *then* read the maximum. Record a key point whenever the maximum differs from the last one recorded.

```java
public List<List<Integer>> getSkyline(int[][] buildings) {
    List<int[]> events = new ArrayList<>();              // {x, height, +1 = starts here, -1 = ends here}
    for (int[] b : buildings) {
        events.add(new int[] {b[0], b[2], 1});
        events.add(new int[] {b[1], b[2], -1});
    }
    events.sort((p, q) -> Integer.compare(p[0], q[0]));
    TreeMap<Integer, Integer> active = new TreeMap<>();  // multiset of the heights of active buildings
    List<List<Integer>> skyline = new ArrayList<>();
    int prevHeight = 0;
    for (int i = 0; i < events.size(); ) {
        int x = events.get(i)[0];
        for (; i < events.size() && events.get(i)[0] == x; i++) {     // apply ALL events at this x first
            int[] e = events.get(i);
            if (e[2] == 1) active.merge(e[1], 1, Integer::sum);
            else if (active.merge(e[1], -1, Integer::sum) == 0) active.remove(e[1]);
        }
        int height = active.isEmpty() ? 0 : active.lastKey();
        if (height != prevHeight) {
            skyline.add(List.of(x, height));
            prevHeight = height;
        }
    }
    return skyline;
}
```

O(n log n). **Processing every event at the same x before reading the height** is what avoids the classic bugs: a spurious dip when one building ends exactly where another of the same height starts, or a duplicate point when two buildings start together. A priority queue with lazy deletion (Ch 15) does the same job. The ordered multiset is simpler, because it supports removing an arbitrary height directly.

---

## 8. Where interval techniques break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Maximise the total **weight** (or profit) of compatible intervals | Earliest-end greedy can drop one heavy interval to keep two light ones | DP over intervals sorted by end + binary search (Ch 33, Maximum Profit in Job Scheduling) |
| Coordinates up to 10⁹ with a difference array | An array that large doesn't fit | Sort the events, or compress coordinates (Ch 9 §5) |
| Intervals inserted and *removed* online, with coverage queries | Re-sorting is too slow | `TreeMap` merging on insert (stretch: Range Module); segment trees (Ch 34) |
| Touching endpoints handled inconsistently | Off-by-one in overlap tests | State closed or half-open in a comment, and use one test throughout |
| Intervals on a circle (time of day that wraps) | "Sorted" has no natural start | Split wrapping intervals in two, or double the timeline |

---

## 9. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| Insert one interval into a sorted, disjoint list | Three phases: the intervals before it, the ones merging with it, the ones after it (practice) |
| Free time common to everyone | Merge all the busy intervals, then report the gaps |
| A minimum number of intervals covering `[0, T]` | Greedy: from the current covered end, take the interval reaching furthest (Video Stitching, practice) |
| For each query point, the smallest interval containing it | Sort the queries and the intervals; keep the active intervals in an ordered set by length (stretch) |
| Rectangles instead of intervals (union area) | Sweep in x, keep the active y-coverage in a segment tree (Ch 34, Rectangle Area II) |

---

## 10. Recognition

**Signals:** meetings, bookings, time ranges, segments, "overlap", "conflict", "merge", "cover", "rooms", "platforms", "maximum concurrent".

**The decision to make first:** is it **union/coverage** (sort by start), **selection** (sort by end), **concurrency** (events), **online** (`TreeMap`), or **weighted selection** (DP)?

**False friends:** "maximum total value of non-overlapping intervals" looks like Non-overlapping Intervals, but it's DP (§8).

---

## 11. Implementation traps

| Trap | Fix |
| :--- | :--- |
| `<` versus `≤` for touching endpoints | Decide closed or half-open from the examples; comment it |
| Forgetting `Math.max` when merging a nested interval | `last[1] = Math.max(last[1], cur[1])` |
| A subtracting comparator on coordinates up to 10⁹ | `Integer.compare` (Ch 4 §14) |
| Reading the sweep state between two events at the same x | Apply every event at x first |
| Mutating the caller's interval arrays | Copy before changing them |
| Unboxing a `null` from `floorKey` / `ceilingKey` | Use `Integer` and check for `null` |

---

## 12. Practice

1. **[Insert Interval](https://leetcode.com/problems/insert-interval/)** *(Medium)*
2. **[Count Days Without Meetings](https://leetcode.com/problems/count-days-without-meetings/)** *(Medium)*
3. **[Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/)** *(Medium)*
4. **[Remove Covered Intervals](https://leetcode.com/problems/remove-covered-intervals/)** *(Medium)*
5. **[Video Stitching](https://leetcode.com/problems/video-stitching/)** *(Medium)*
6. **[My Calendar II](https://leetcode.com/problems/my-calendar-ii/)** *(Medium)*
7. **[Number of Flowers in Full Bloom](https://leetcode.com/problems/number-of-flowers-in-full-bloom/)** *(Hard)*

**Stretch:** **[Data Stream as Disjoint Intervals](https://leetcode.com/problems/data-stream-as-disjoint-intervals/)** *(Hard)* · **[Range Module](https://leetcode.com/problems/range-module/)** *(Hard)* · **[Minimum Interval to Include Each Query](https://leetcode.com/problems/minimum-interval-to-include-each-query/)** *(Hard)*

<details><summary>Nudges</summary>

1. Insert: copy the intervals ending before the new one; merge everything overlapping it (min start, max end); copy the rest.
2. Days Without Meetings: merge (§2), add up the covered days, subtract from the total.
3. Arrows: sort by end, and shoot at the end of the first un-burst balloon. Balloons touching at a point are burst by the same arrow.
4. Covered: sort by start ascending and, for equal starts, end *descending*. Then an interval is covered exactly when its end ≤ the largest end seen so far.
5. Video Stitching: from the current covered end, choose the clip starting at or before it that reaches furthest. This is greedy that stays ahead (Ch 2 §6), much like Jump Game II.
6. Calendar II: a `TreeMap` of +1/−1 events. Tentatively add the new booking, sweep, and remove it again if the count ever reaches 3.
7. Flowers: sort the starts and the ends; for person p, the answer is (starts ≤ p) − (ends < p), via two binary searches.
8. Stretch, Disjoint Intervals: a `TreeMap` from start to end; on `addNum`, merge with the neighbours from `floorEntry` and `higherEntry`.
9. Stretch, Range Module: a `TreeMap` of disjoint intervals; add means merge everything overlapping; remove means cut the overlapping ones, possibly splitting one into two.
10. Stretch, Min Interval per Query: sort the queries; sweep, adding the intervals that have started into a `TreeMap` keyed by (length, end), and drop the ones that have ended before the query.
</details>

<details><summary>Key</summary>

Insert → P1 linear · Days Without Meetings → P1 · Arrows → P2 · Covered → sort by (start ↑, end ↓) · Video Stitching → greedy cover that stays ahead · Calendar II → P3 events, online (P5) · Flowers → P3 + binary search on sorted starts and ends · Disjoint Intervals → P5 merging on insert · Range Module → P5 split/merge · Min Interval per Query → offline sorted queries + P6-style active set
</details>

---

## 13. Self-test

<details><summary>1. Write the overlap test for two closed intervals, and for two half-open ones.</summary>

Closed: `a.s <= b.e && b.s <= a.e`. Half-open: `a.s < b.e && b.s < a.e`.
</details>

<details><summary>2. Why sort by end, not by start, to keep the most non-overlapping intervals?</summary>

Only the end of the last kept interval constrains future choices, so ending earliest leaves the most room. The exchange argument swaps the optimum's earliest-ending interval for the overall earliest-ending one without creating a conflict. Sorting by start fails on `[1,10], [2,3], [4,5]`.
</details>

<details><summary>3. Why is the minimum number of meeting rooms equal to the maximum number of meetings active at once?</summary>

That many meetings overlap at some moment, so you need at least that many rooms. Assigning each meeting, in start order, to any free room never needs more rooms than the number of meetings active at that start.
</details>

<details><summary>4. In Interval List Intersections, why advance the interval that ends first?</summary>

Every later interval in the other list starts after the current one there, which already extends past this end, so the interval that ended first can't intersect anything else.
</details>

<details><summary>5. Why can greedy fail for maximum-weight interval selection?</summary>

Keeping the earliest-ending interval can block a single much heavier interval. Example: `[1,3]` weight 1, `[2,10]` weight 100. Greedy keeps `[1,3]` and gets 1, while the optimum takes `[2,10]` for 100.
</details>

---

## 14. Exit gate

Move on when you can:

* say, for any interval problem, whether it needs sort-by-start, sort-by-end, events, an ordered map or DP, and give the one-line reason;
* prove the earliest-end greedy and the rooms = maximum-overlap theorem out loud;
* write Merge Intervals, the two-array sweep, My Calendar I and the Skyline from blank files, each in under 12 minutes;
* solve 6 of the 7 practice problems.
