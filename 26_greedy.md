# Chapter 26 — Greedy: Prove It or Kill It

> **Part VI · Week 23** · Prerequisites: Ch 2 (all of it), Ch 9, Ch 14 (sort by end), Ch 15 (heaps)
> **Problems:** 5 worked (1 ★) + an 8-exhibit counterexample museum · 8 practice · 3 stretch · **Time:** about 8 hours

A greedy algorithm builds an answer by repeatedly making the choice that looks best **right now**, and never reconsiders it. When it's right, it's usually the shortest and fastest solution available. When it's wrong, it's still plausible, passes the examples, and fails hidden tests. That's why greedy is the most dangerous technique in interviews.

You already have the tools for both outcomes: exchange arguments, stays-ahead and counterexample hunting (Ch 2). This chapter turns them into a **protocol**, adds the one idea that rescues many greedy rules that almost work (**regret**: undoing an earlier choice with a heap), and ends with a museum of greedy rules that look right and aren't.

---

## 0. The greedy protocol

1. **State the rule precisely.** "Sort by ___, then take each item if ___." A vague rule can't be proved or broken.
2. **Attack it for two minutes** with Ch 2 §8's checklist: tiny cases, ties, one huge item, and above all **an input where the rule's first choice is wrong**.
3. **Prove it**, usually in one of two ways:
   * **Exchange:** any optimal solution can be changed, step by step, into the greedy one without getting worse.
   * **Stays ahead:** after every step, greedy's progress is at least as good as any other solution's.
4. **Stress-test it** against a brute force (Ch 5 §6). A greedy rule you can't prove *must* pass a stress test before you trust it.

If step 2 finds a counterexample, the rule is dead. Ask why it died. Usually it's because **an early choice constrains later choices in a way the rule can't see**, and that's the signal for DP (Ch 27).

---

## 1. Attempt first ★

> **[Course Schedule III](https://leetcode.com/problems/course-schedule-iii/) (30 minutes).** Each course is `[duration, lastDay]`: it takes `duration` consecutive days and must finish by day `lastDay`. You take courses one at a time, starting from day 1. Return the maximum number of courses you can take.

---

## 2. Sort by deadline, then regret

### Worked G1 · [Course Schedule III](https://leetcode.com/problems/course-schedule-iii/) ★

**Step A: the order within a chosen set.** Suppose you've decided *which* courses to take. If any order meets every deadline, then **earliest deadline first** does too. *Exchange:* if two adjacent courses are out of deadline order, swapping them doesn't hurt. Both still finish by the later finish time, and that time now falls under the later deadline. So sort by deadline, and only the *choice* of courses remains.

**Step B: which courses.** Scan in deadline order, adding each course. If the total time now exceeds the current deadline, the set is infeasible, and **one course must go. Remove the longest course taken so far** (possibly the new one).

> **Invariant:** after processing the first i courses (in deadline order), the chosen set is feasible, **as large as possible** among those i courses, and has the **smallest total time** among all sets of that size.
>
> *Why removing the longest keeps it true:* if adding course i breaks feasibility, then by the invariant no set of size (count + 1) is feasible within the first i courses. So the best possible size stays the same, and among the sets of that size, dropping the longest course leaves the smallest total time, which is the most slack for every later course.

That "drop the worst past choice" step is **regret**. A max-heap of the durations taken makes it O(log n):

```java
public int scheduleCourse(int[][] courses) {
    int[][] byDeadline = courses.clone();
    Arrays.sort(byDeadline, (a, b) -> Integer.compare(a[1], b[1]));   // any feasible set can run in deadline order
    PriorityQueue<Integer> taken = new PriorityQueue<>(Collections.reverseOrder());   // durations, longest on top
    int time = 0;                                   // total duration of the courses taken so far
    for (int[] c : byDeadline) {
        taken.offer(c[0]);
        time += c[0];
        if (time > c[1]) time -= taken.poll();      // past the deadline: drop the longest course so far
    }
    return taken.size();
}
```

O(n log n). Without the regret step, "earliest deadline first, take whatever fits" fails on `[(5, 5), (1, 6), (1, 6), (1, 6)]`. It takes the 5-day course and then fits only one of the short ones, for 2, while the three short ones together give 3. That's exhibit 6 in §6.

---

## 3. Greedy over time with a heap

### Worked G2 · [Maximum Number of Events That Can Be Attended](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended/)

> Each event `[start, end]` lasts for some days; you can attend it on **any one day** in that range, and you can attend at most one event per day. Return the maximum number of events attended.

**Rule:** each day, among the events available that day (already started, not yet ended), attend **the one that ends soonest**.

**Exchange proof.** Suppose an optimal schedule attends some other available event f on day d, while e is the available event ending soonest. If e is attended later, swap the days of e and f. f ends no sooner than e does, so f can take e's later day, and e fits today. If e is never attended, just put e in f's place on day d. Neither change reduces the count.

A min-heap holds the end days of events that have started. Each day, drop the expired events, then attend the top one:

```java
public int maxEvents(int[][] events) {
    int[][] byStart = events.clone();
    Arrays.sort(byStart, (a, b) -> Integer.compare(a[0], b[0]));
    PriorityQueue<Integer> openEnds = new PriorityQueue<>();        // end days of events that have started
    int i = 0, attended = 0, n = byStart.length;
    for (int day = 1; i < n || !openEnds.isEmpty(); day++) {
        if (openEnds.isEmpty() && byStart[i][0] > day) day = byStart[i][0];   // skip days with nothing to do
        while (i < n && byStart[i][0] == day) openEnds.offer(byStart[i++][1]);
        while (!openEnds.isEmpty() && openEnds.peek() < day) openEnds.poll();   // these have already ended
        if (!openEnds.isEmpty()) {
            openEnds.poll();                                         // attend the one ending soonest
            attended++;
        }
    }
    return attended;
}
```

O((n + D) log n), where D is the number of days actually visited. Compare with Ch 14's interval scheduling: there, each chosen interval occupies its whole range, so you sort by end. Here each event needs only **one day**, so you decide day by day, using the heap for the deadline order.

---

## 4. Deferring decisions: the regret heap

### Worked G3 · [Minimum Number of Refueling Stops](https://leetcode.com/problems/minimum-number-of-refueling-stops/)

> A car starts with `startFuel` and drives toward `target`, using one unit of fuel per unit of distance. `stations[i] = [position, fuel]`, sorted by position. Return the minimum number of refuelling stops needed to reach the target, or −1.

Deciding at each station whether to stop is hard, because you don't yet know what the next stations offer. **Defer the decision.** Drive as far as the current fuel allows. Every station you pass goes into a max-heap of "fuel I could have taken". When you can't reach any further, **pretend, retroactively, that you stopped at the passed station with the most fuel**.

**Stays ahead.** Greedy's k-th stop takes the largest fuel available among the stations reachable using k − 1 stops, so after k stops it reaches at least as far as any strategy with k stops. Hence it needs the fewest stops.

```java
public int minRefuelStops(int target, int startFuel, int[][] stations) {
    PriorityQueue<Integer> passed = new PriorityQueue<>(Collections.reverseOrder());   // fuel at stations driven past
    long reach = startFuel;                      // the furthest point reachable with the stops chosen so far
    int stops = 0, i = 0;
    while (reach < target) {
        while (i < stations.length && stations[i][0] <= reach) passed.offer(stations[i++][1]);
        if (passed.isEmpty()) return -1;         // stuck, with nothing left to refuel from
        reach += passed.poll();                  // retroactively stop at the passed station with the most fuel
        stops++;
    }
    return stops;
}
```

`reach` is a `long` because the total fuel can exceed `int`. Furthest Building You Can Reach (Ch 15 practice) is the same pattern: give ladders to the largest climbs so far, and convert the smallest one back to bricks when you run out.

---

## 5. Covering a range: stays ahead

### Worked G4 · [Minimum Number of Taps to Open to Water a Garden](https://leetcode.com/problems/minimum-number-of-taps-to-open-to-water-a-garden/)

> The garden is the segment `[0, n]`. Tap i, at position i, waters `[i − ranges[i], i + ranges[i]]`. Return the fewest taps that water the whole garden, or −1.

**Reduce it to Jump Game II** (Ch 2 §6). For each left position l, record the **furthest right end** of any tap whose interval starts at l. Then scan from left to right, keeping the furthest point reachable from everything seen so far. When the scan reaches the end of the current coverage, you **must** open another tap, so open the one that reaches furthest.

```java
public int minTaps(int n, int[] ranges) {
    int[] furthest = new int[n + 1];             // furthest[l] = the furthest right end among taps starting at l
    for (int i = 0; i <= n; i++) {
        int left = Math.max(0, i - ranges[i]), right = Math.min(n, i + ranges[i]);
        furthest[left] = Math.max(furthest[left], right);
    }
    int taps = 0, covered = 0, nextCovered = 0;  // [0, covered] is watered by the taps chosen so far
    for (int x = 0; x < n; x++) {
        nextCovered = Math.max(nextCovered, furthest[x]);   // the best reach among taps starting at or before x
        if (x == covered) {                      // coverage ends here: we must open another tap
            if (nextCovered <= x) return -1;     // nothing reaches past x: a gap that can't be watered
            taps++;
            covered = nextCovered;
        }
    }
    return taps;
}
```

O(n). **Stays ahead:** after k taps, greedy's coverage extends at least as far as any other choice of k taps. Video Stitching (Ch 14 practice) is the same problem with clips.

---

## 6. Where greedy ends and DP begins

### Worked G5a · [Wiggle Subsequence](https://leetcode.com/problems/wiggle-subsequence/)

> Return the length of the longest subsequence whose consecutive differences strictly alternate in sign.

**Rule:** count the **turning points**: the first element, then every element where the direction of change flips (ignoring equal neighbours).

*Why it's optimal:* any wiggle subsequence takes at most one element from each maximal increasing or decreasing run of the array, since two elements from the same run can't alternate. Taking the extreme element of each run (each peak and valley) is always at least as good (exchange), and that gives exactly 1 + the number of direction changes.

```java
public int wiggleMaxLength(int[] nums) {
    int length = 1, lastDirection = 0;               // the sign of the last counted move; 0 = none yet
    for (int i = 1; i < nums.length; i++) {
        int direction = Integer.compare(nums[i], nums[i - 1]);
        if (direction != 0 && direction != lastDirection) {   // a turn (or the first real move)
            length++;
            lastDirection = direction;
        }
    }
    return length;
}
```

It also has an O(n) DP with two states, the longest wiggle ending with an up move and with a down move (Ch 27). Greedy wins here because the structure (one element per run) makes local choices safe. **When local choices interact**, as in the exhibits below, they aren't safe, and DP is required.

### Worked G5b *(custom)* · The Greedy Counterexample Museum

For each exhibit, cover the right-hand columns, try to break the rule yourself, and only then look. The goal is to build the reflex of attacking every greedy rule before trusting it.

| # | Plausible rule | Smallest counterexample | What's actually right |
| :-: | :--- | :--- | :--- |
| 1 | Coin change: always take the largest coin that fits | Coins {1, 3, 4}, amount 6: greedy 4+1+1 (3 coins); optimal 3+3 (2) | DP over amounts (Ch 28) |
| 2 | 0/1 knapsack: take items by value/weight ratio | Capacity 10; items (w 6, v 7), (5, 5), (5, 5): greedy gets 7; optimal 10 | 0/1 knapsack DP (Ch 28) |
| 3 | Weighted intervals: choose by earliest end | `[1,3]` weight 1, `[2,10]` weight 100: greedy 1; optimal 100 | DP + binary search (Ch 33) |
| 4 | Jump Game II: always jump as far as possible | `[2, 3, 1, 1, 4]`: 3 jumps; optimal 2 | Stays-ahead levels (Ch 2 §6) |
| 5 | Longest path in a DAG: always follow the heaviest edge | A→B 10, B→D 1, A→C 1, C→D 100: greedy 11; optimal 101 | Topological DP (Ch 22) |
| 6 | Deadlines: earliest deadline first, never dropping a course | `[(5,5), (1,6), (1,6), (1,6)]`: greedy 2; optimal 3 | The regret heap (G1) |
| 7 | Merging sticks (cost = the sum of the pair): merge the two largest | `[1, 2, 3]`: largest-first costs 5 + 6 = 11; smallest-first costs 3 + 6 = 9 | Always merge the two **smallest** (a heap; Huffman's argument) |
| 8 | Gas station: start where there's the most gas | gas `[1,2,3,4,5]`, cost `[3,4,5,1,2]`: starting at index 4 fails; index 3 works | Start right after the prefix minimum (Ch 1 practice) |

**What the failures share:** in exhibits 1, 2, 3 and 5, an early choice **uses up something** (capacity, time, the remaining amount) that a later choice needed, and the local rule can't see that. When you find such an interaction, stop looking for a better greedy rule and start defining DP states.

---

## 7. Recognition

**Signals that greedy is plausible:**
* An **ordering** decides everything ("process by deadline / end / ratio"), and an exchange argument looks natural.
* A **covering** or **reaching** problem ("fewest taps / jumps / arrows / stops") → stays ahead.
* A **one-pass decision with an undo option** → a regret heap.
* **Lexicographically smallest / largest** results → a greedy stack (Ch 12 §6) or digit-by-digit choices.

**Signals against greedy:**
* "Maximise the total **value** / **weight**" with capacity or overlap constraints → usually DP.
* "**Count** the ways": greedy builds one solution, not a count.
* You found a counterexample, however strange-looking.

---

## 8. Implementation traps

| Trap | Fix |
| :--- | :--- |
| A greedy rule you never tried to break | Two minutes of the Ch 2 §8 checklist, plus a stress test |
| Sorting by the wrong key (start versus end, ratio versus difference) | Prove the key with an exchange argument before coding |
| A regret heap with the wrong orientation | "I'll want to undo the *worst* past choice, so the worst must be on top" |
| Forgetting that the new item might itself be the one to drop | Push it first, then pop (G1) |
| Overflow in accumulated totals (fuel, time) | `long` |

---

## 9. Practice

1. **[Increasing Triplet Subsequence](https://leetcode.com/problems/increasing-triplet-subsequence/)** *(Medium)*
2. **[Score After Flipping Matrix](https://leetcode.com/problems/score-after-flipping-matrix/)** *(Medium)*
3. **[Bag of Tokens](https://leetcode.com/problems/bag-of-tokens/)** *(Medium)*
4. **[Maximum Swap](https://leetcode.com/problems/maximum-swap/)** *(Medium)*
5. **[Monotone Increasing Digits](https://leetcode.com/problems/monotone-increasing-digits/)** *(Medium)*
6. **[Broken Calculator](https://leetcode.com/problems/broken-calculator/)** *(Medium)*
7. **[Valid Parenthesis String](https://leetcode.com/problems/valid-parenthesis-string/)** *(Medium)*
8. **[Eliminate Maximum Number of Monsters](https://leetcode.com/problems/eliminate-maximum-number-of-monsters/)** *(Medium)*

**Stretch:** **[Patching Array](https://leetcode.com/problems/patching-array/)** *(Hard)* · **[Minimum Cost to Hire K Workers](https://leetcode.com/problems/minimum-cost-to-hire-k-workers/)** *(Hard)* · **[Put Marbles in Bags](https://leetcode.com/problems/put-marbles-in-bags/)** *(Hard)*

<details><summary>Nudges</summary>

1. Increasing Triplet: keep the smallest value seen, and the smallest value that has something smaller before it. Why is it safe to keep lowering both?
2. Flipping Matrix: first make every row start with 1 (the top bit dominates every other bit). Then flip each column if that gives it more 1s than 0s.
3. Bag of Tokens: sort; spend the smallest token to gain a point, and when out of power, cash a point in for the largest token. Track the best score.
4. Max Swap: record each digit's last position; for each position from the left, swap with the largest digit that appears later (taking its last occurrence).
5. Monotone Digits: find the first place where a digit drops, decrement there (moving left past equal digits), then fill everything after with 9s.
6. Broken Calculator: work **backwards** from the target: halve if even, otherwise add 1. Why is the reverse greedy optimal?
7. Parenthesis String: track the range [lo, hi] of possible open counts; `*` widens it. Keep lo ≥ 0; fail if hi < 0.
8. Monsters: compute each monster's arrival time (rounded up), sort, and shoot one per minute; you lose when a monster arrives before its turn.
9. Stretch, Patching: if [1, reach) is coverable, adding a number x ≤ reach extends coverage to [1, reach + x). When the next number is too big, patch with `reach` itself. Stays ahead.
10. Stretch, Hire K Workers: sort by wage/quality ratio. For each ratio as the cap, take the k smallest qualities among the workers so far (a max-heap of qualities).
11. Stretch, Marbles: every split adds `w[i] + w[i+1]` for the cut point i. Sort those pair sums, then take the (k − 1) largest minus the (k − 1) smallest.
</details>

<details><summary>Key</summary>

Increasing Triplet → two best-so-far thresholds (stays ahead) · Flipping Matrix → greedy by bit weight · Bag of Tokens → sort + two pointers, exchange · Max Swap → last occurrence of each digit · Monotone Digits → fix the first drop, then fill 9s · Broken Calculator → work backwards · Parenthesis String → a range of open counts · Monsters → sort by arrival time · Patching → stays ahead on the reachable prefix · Hire K Workers → sort by ratio + heap · Marbles → contribution of each cut, sorted pair sums
</details>

---

## 10. Self-test

<details><summary>1. In Course Schedule III, why is it enough to consider courses in deadline order?</summary>

By exchange: if two adjacent courses are out of deadline order, swapping them keeps both within their deadlines. So any feasible set is also feasible in deadline order.
</details>

<details><summary>2. Why does Course Schedule III drop the longest course, rather than the newest?</summary>

Both choices leave the same count, but dropping the longest leaves the smallest total time, which gives every later course the most slack (the invariant).
</details>

<details><summary>3. What does the refuelling greedy's "retroactive stop" mean, and why is it allowed?</summary>

When you can't go further, you choose, from all the stations you already passed, the one with the most fuel, as if you had stopped there. The fuel would have been available at that point anyway, so it's always legal. Choosing the largest maximises how far you get for each stop.
</details>

<details><summary>4. Give the smallest counterexample to "merge the two largest sticks first".</summary>

`[1, 2, 3]`: largest-first costs 5 + 6 = 11; smallest-first costs 3 + 6 = 9.
</details>

<details><summary>5. What's the common reason the museum's greedy rules fail?</summary>

An early choice consumes a resource or constrains later choices in a way the local rule can't see. That interaction is what DP handles.
</details>

---

## 11. Exit gate

Move on when you can:

* run the four-step greedy protocol on a new rule, and either prove it or produce a counterexample;
* explain Course Schedule III's invariant and the regret step, and write it from a blank file;
* write the refuelling and taps greedies with their stays-ahead arguments;
* reproduce all eight exhibits of the museum from memory: the rule, the counterexample, and the fix;
* solve 7 of the 8 practice problems.
