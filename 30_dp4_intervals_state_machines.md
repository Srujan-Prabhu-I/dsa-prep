# Chapter 30 — DP IV: Intervals & State Machines

> **Part VI · Week 27** · Prerequisites: Ch 18 §7 (games), Ch 27, Ch 29 (substring states)
> **Problems:** 6 worked (1 ★) · 7 practice · 2 stretch · **Time:** about 9 hours

Two more shapes of DP state:
* **Intervals:** `dp[l][r]` describes a contiguous segment, and the transition picks a **split point** inside it. The hard part isn't the code, it's choosing the *right* decision to split on. Usually it's the **last** action taken in the segment, not the first, and §2 shows why.
* **State machines:** the problem is a sequence of steps, and at each step you're in one of a few **modes** (holding a stock or not, in a cooldown or not, k transactions used). The DP tracks the best value in each mode. Drawing the automaton *is* the derivation.

---

## 0. Interval DP: the template

> **State:** `dp[l][r]` = the answer for the segment `l..r`.
> **Transition:** choose a split point k inside it; the segment's answer combines `dp[l][k]`, `dp[k][r]` (or `dp[k+1][r]`) and the cost of the decision at k.
> **Order:** every segment depends on **shorter** segments, so iterate by **increasing length**, or by l descending with r ascending.
> **Cost:** n² segments × n split points = **O(n³)**, which is fine for n ≤ ~500.

**The one hard question:** *which decision makes the two sides independent?* If the two sub-segments still affect each other after the split, the recurrence is wrong.

---

## 1. Attempt first ★

> **[Burst Balloons](https://leetcode.com/problems/burst-balloons/) (40 minutes).** Balloons `nums[i]` sit in a row. Bursting balloon i earns `left · nums[i] · right`, where left and right are its **current** neighbours (1 if there are none). Burst them all. Return the maximum total coins. (n ≤ 300.)

---

## 2. The "last action" trick

### Worked I1a · [Burst Balloons](https://leetcode.com/problems/burst-balloons/) ★

**The natural attempt fails.** "Choose the **first** balloon to burst, then solve both sides." After bursting k, the balloons on the left and right of k become **neighbours of each other**. A later burst on the left side can earn coins from a balloon on the right side. **The two sides aren't independent**, so `dp[l][k−1] + dp[k+1][r]` is wrong.

**Choose the *last* balloon instead.** In the open interval (l, r), with l and r themselves still standing, suppose balloon k is burst **last**. Then:
* every balloon between l and k is gone before k, and those bursts only ever see neighbours inside (l, k), with l and k as walls;
* likewise for (k, r);
* when k finally bursts, its neighbours are exactly **l and r**.

**The two sides are separated by k, which stays standing until the end.** So they're independent:

> `best[l][r] = max over l < k < r of ( best[l][k] + v[l] · v[k] · v[r] + best[k][r] )`,

with a virtual balloon of value 1 added at each end.

```java
public int maxCoins(int[] nums) {
    int n = nums.length;
    int[] v = new int[n + 2];                     // padded with a virtual balloon of value 1 at each end
    v[0] = v[n + 1] = 1;
    for (int i = 0; i < n; i++) v[i + 1] = nums[i];
    int[][] best = new int[n + 2][n + 2];         // best[l][r] = max coins from bursting everything strictly between l and r
    for (int len = 2; len <= n + 1; len++) {      // len = r - l: shorter intervals first
        for (int l = 0; l + len <= n + 1; l++) {
            int r = l + len;
            for (int k = l + 1; k < r; k++) {     // k bursts LAST in (l, r): its neighbours are then l and r
                best[l][r] = Math.max(best[l][r], best[l][k] + v[l] * v[k] * v[r] + best[k][r]);
            }
        }
    }
    return best[0][n + 1];
}
```

O(n³) = 2.7·10⁷ for n = 300. **The general lesson: when a decision changes the context for everything around it, split on the decision that happens last, when the context is fixed.**

### Worked I1b · [Minimum Cost to Cut a Stick](https://leetcode.com/problems/minimum-cost-to-cut-a-stick/)

> A stick of length n must be cut at the given positions. The cost of a cut is the **current length of the piece** being cut. Order the cuts to minimise the total cost.

Here the **first** cut works, and it's worth understanding why this differs from the balloons. Once you cut a piece at k, the two resulting pieces **never interact again**. They're separate sticks. So splitting on the first cut gives independent subproblems. Sort the cut positions and add the two ends:

> `cost[i][j]` = the cheapest way to make every cut strictly between `c[i]` and `c[j]` = `(c[j] − c[i]) + min over i < k < j of (cost[i][k] + cost[k][j])`.

```java
public int minCost(int n, int[] cuts) {
    int m = cuts.length;
    int[] c = new int[m + 2];                     // c[0] = 0 and c[m+1] = n are the stick's ends
    for (int i = 0; i < m; i++) c[i + 1] = cuts[i];
    c[m + 1] = n;
    Arrays.sort(c);
    int[][] cost = new int[m + 2][m + 2];         // adjacent positions (no cut between them) cost 0
    for (int len = 2; len <= m + 1; len++) {
        for (int i = 0; i + len <= m + 1; i++) {
            int j = i + len, best = Integer.MAX_VALUE;
            for (int k = i + 1; k < j; k++) best = Math.min(best, cost[i][k] + cost[k][j]);   // k is cut FIRST
            cost[i][j] = best + (c[j] - c[i]);    // that first cut is made on the whole piece c[i]..c[j]
        }
    }
    return cost[0][m + 1];
}
```

O(m³). **Burst Balloons versus Cut Stick:** a burst *joins* its neighbours, so split on the last one. A cut *separates* the pieces, so split on the first one. Ask which decision makes the two sides independent, and split on that one.

---

## 3. Games on intervals

### Worked I2 · [Predict the Winner](https://leetcode.com/problems/predict-the-winner/)

> Two players alternately take a number from either **end** of the array. Both play optimally. Does player 1 finish with at least as much as player 2?

Ch 18 §7 handled Stone Game with a strategy argument, but only for its special conditions. **The general tool is minimax DP.** It's simplest with a *difference*:

> `lead[l][r]` = (the score of the player about to move) − (the opponent's score), when `nums[l..r]` remains and both play optimally.

If the mover takes `nums[l]`, the opponent becomes the mover on `l+1..r`, and their lead there counts *against* the current mover:

> `lead[l][r] = max(nums[l] − lead[l+1][r], nums[r] − lead[l][r−1])`.

```java
public boolean predictTheWinner(int[] nums) {
    int n = nums.length;
    int[][] lead = new int[n][n];
    for (int i = 0; i < n; i++) lead[i][i] = nums[i];
    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            lead[l][r] = Math.max(nums[l] - lead[l + 1][r],     // take the left end; the opponent then leads on the rest
                                  nums[r] - lead[l][r - 1]);    // or the right end
        }
    }
    return lead[0][n - 1] >= 0;                   // a tie counts as a win for player 1
}
```

**Using the difference** means one table instead of two (one per player's total), and "optimal for the mover" is a single `max`. Stone Game VII and Guess Number Higher or Lower II (practice) are the same minimax-on-intervals idea.

---

## 4. Intervals where pieces can merge

### Worked I3 · [Strange Printer](https://leetcode.com/problems/strange-printer/)

> A printer prints a run of **one** letter over any contiguous range, overwriting what's there. Return the fewest prints that produce `s`.

**State:** `turns[l][r]` = the fewest prints for `s[l..r]`. Look at `s[l]`:
* **Print it on its own:** `1 + turns[l+1][r]`.
* **Share its print with a later copy of the same letter.** If `s[k] == s[l]` for some k in (l, r], one print of that letter can cover the whole of `l..k`. The inside `l+1..k−1` is printed on top of it independently, and `s[k..r]` is handled as usual, with s[k]'s print being that same shared print. So s[l] costs nothing extra: `turns[l+1][k−1] + turns[k][r]`.

```java
public int strangePrinter(String s) {
    int n = s.length();
    int[][] turns = new int[n + 1][n + 1];        // empty ranges (l > r) cost 0
    for (int l = n - 1; l >= 0; l--) {
        for (int r = l; r < n; r++) {
            turns[l][r] = turns[l + 1][r] + 1;    // print s[l] on its own
            for (int k = l + 1; k <= r; k++) {
                if (s.charAt(k) == s.charAt(l)) { // one print of this letter covers both l and k
                    turns[l][r] = Math.min(turns[l][r], turns[l + 1][k - 1] + turns[k][r]);
                }
            }
        }
    }
    return turns[0][n - 1];
}
```

O(n³). The pattern, **"the first element merges with a matching later element"**, is the core of the hard interval DPs. Remove Boxes (stretch) adds a third dimension to track how many matching boxes are attached.

---

## 5. State machines

When each step's best choice depends on a small **mode** (holding a stock or not, whether a cooldown is active, how many transactions you've used), draw the modes as an **automaton**. Its arrows are the legal moves, each with a cost. The DP is then **one value per mode per step**, and each value is the best over the arrows that lead into that mode.

### Worked A1a · [Best Time to Buy and Sell Stock with Cooldown](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

> Trade as many times as you like, holding at most one share at a time. After you sell, you can't buy on the next day. Maximise the profit.

```text
          buy (−p)                sell (+p)
   REST ───────────▶ HOLD ───────────────▶ SOLD
   ▲  ↺ (stay)         ↺ (stay)             │
   └────────────────────────────────────────┘
               (cooldown over: the next day you're in REST)
```

Three modes per day:
* `hold`: owning a share;
* `sold`: just sold today, so you must rest tomorrow;
* `rest`: owning nothing and free to buy.

The arrows *into* each mode give the transitions:

```java
public int maxProfit(int[] prices) {
    int hold = Integer.MIN_VALUE / 2;               // impossible before the first day; /2 so that adding is safe
    int sold = Integer.MIN_VALUE / 2;
    int rest = 0;
    for (int p : prices) {
        int prevHold = hold, prevSold = sold, prevRest = rest;
        hold = Math.max(prevHold, prevRest - p);    // keep holding, or buy today (only from REST, never right after a sale)
        sold = prevHold + p;                        // sell today
        rest = Math.max(prevRest, prevSold);        // stay out, or come out of the cooldown
    }
    return Math.max(sold, rest);                    // finishing while holding is never optimal
}
```

**Why it's correct:** the modes record everything the future depends on (whether you hold a share, and whether you're allowed to buy), so the best value in each mode really is a valid DP state. A transaction fee (practice) just subtracts the fee on one arrow.

### Worked A1b · [Best Time to Buy and Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)

> At most **k** transactions (a buy followed by a sell). Maximise the profit.

Now the mode must also count transactions: "after the j-th buy" and "after the j-th sell", for j = 1…k.

> `buy[j] = max(buy[j], sell[j−1] − p)`: the j-th buy happens after j − 1 complete transactions.
> `sell[j] = max(sell[j], buy[j] + p)`.

```java
public int maxProfit(int k, int[] prices) {
    int n = prices.length;
    if (k >= n / 2) {                               // there can't be more than n/2 useful transactions: unlimited
        int profit = 0;
        for (int i = 1; i < n; i++) profit += Math.max(0, prices[i] - prices[i - 1]);   // take every rise
        return profit;
    }
    int[] buy = new int[k + 1], sell = new int[k + 1];   // the best profit after the j-th buy / j-th sell
    Arrays.fill(buy, Integer.MIN_VALUE / 2);
    for (int p : prices) {
        for (int j = 1; j <= k; j++) {
            buy[j] = Math.max(buy[j], sell[j - 1] - p);
            sell[j] = Math.max(sell[j], buy[j] + p);
        }
    }
    return sell[k];
}
```

That's O(n · k). Updating `buy[j]` and `sell[j]` on the same day is harmless: buying and selling on one day adds nothing. The `k ≥ n/2` shortcut matters, because k can be huge while there are only n/2 possible rises. Stock III (practice) is k = 2.

---

## 6. Where these break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Splitting on the first action when it changes its neighbours' context | The subproblems aren't independent (Burst Balloons) | Split on the **last** action |
| n ≈ 5000 interval DP at O(n³) | 1.25·10¹¹ operations | Look for monotone split points (Knuth's optimisation, recognition only) or a different state |
| A state machine with a missing mode | Illegal moves get allowed (buying right after selling) | Draw the automaton first; one variable per node |
| `Integer.MIN_VALUE` as "impossible", then adding to it | Overflow wraps to a large positive value | `Integer.MIN_VALUE / 2`, or explicit guards |
| Merges of non-adjacent equal elements need more context | Two dimensions aren't enough | Add a count dimension (Remove Boxes) |

---

## 7. Recognition

**Interval DP signals:** "burst", "merge adjacent", "cut", "remove from either end", "triangulate", "the minimum cost to combine a whole range", or games taking from the ends. n is usually ≤ 500.

**State-machine DP signals:** "buy / sell / cooldown / fee / at most k transactions", "at most k absences", "no three consecutive …", or any rule like "after doing X, you can't do Y for a while". Each rule becomes a mode.

---

## 8. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Iterating intervals in the wrong order | By increasing length, or l descending and r ascending |
| Forgetting the virtual boundary elements (balloons, cut ends) | Pad, and comment why |
| Off-by-one between open and closed interval conventions | State "open (l, r)" or "closed [l, r]" next to the table |
| Overwriting a mode's value before the other modes have read the old one | Copy `prevHold`, `prevSold`, … first |
| Minimax computed as a plain sum | Use the *difference* recurrence, with a minus sign |

---

## 9. Practice

1. **[Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)** *(Medium)*
2. **[Best Time to Buy and Sell Stock III](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)** *(Hard)*
3. **[Minimum Swaps To Make Sequences Increasing](https://leetcode.com/problems/minimum-swaps-to-make-sequences-increasing/)** *(Hard)*
4. **[Minimum Score Triangulation of Polygon](https://leetcode.com/problems/minimum-score-triangulation-of-polygon/)** *(Medium)*
5. **[Stone Game VII](https://leetcode.com/problems/stone-game-vii/)** *(Medium)*
6. **[Guess Number Higher or Lower II](https://leetcode.com/problems/guess-number-higher-or-lower-ii/)** *(Medium)*
7. **[Student Attendance Record II](https://leetcode.com/problems/student-attendance-record-ii/)** *(Hard)*

**Stretch:** **[Minimum Cost to Merge Stones](https://leetcode.com/problems/minimum-cost-to-merge-stones/)** *(Hard)* · **[Remove Boxes](https://leetcode.com/problems/remove-boxes/)** *(Hard)*

<details><summary>Nudges</summary>

1. Fee: two modes (hold, cash); subtract the fee when selling.
2. Stock III: A1b with k = 2: four variables, buy1, sell1, buy2 and sell2.
3. Min Swaps: two modes per index, swapped or not; each transition checks that both arrays stay strictly increasing.
4. Triangulation: interval DP on the polygon's vertices; the triangle on the edge (l, r) uses some vertex k, costing v[l] · v[k] · v[r].
5. Stone Game VII: the I2 difference trick; the score for removing an end is the sum of what remains (use prefix sums).
6. Guess Number II: `cost[l][r]` = min over guesses k of (k + max(cost[l][k−1], cost[k+1][r])), since you must pay for the worst case.
7. Attendance II: the modes are (absences ∈ {0, 1}, trailing lates ∈ {0, 1, 2}): six states per day, counted mod 10⁹ + 7.
8. Stretch, Merge Stones: `dp[l][r][p]` = the cheapest way to turn `l..r` into p piles; merging k piles into one costs the range's sum. It's feasible only if (n − 1) % (k − 1) == 0.
9. Stretch, Remove Boxes: `dp[l][r][extra]`, where extra counts the boxes equal to `box[l]` attached on its left; either remove them all now, or merge `box[l]` with a matching box further right.
</details>

<details><summary>Key</summary>

Fee → A1 · Stock III → A1 with k = 2 · Min Swaps → A1 (swapped / not swapped at i) · Triangulation → I1 · Stone Game VII → I2 · Guess Number II → I2 minimax cost · Attendance II → an A1 counting automaton · Merge Stones → I1 with a pile-count dimension · Remove Boxes → I3 with an extra "attached" count
</details>

---

## 10. Self-test

<details><summary>1. Why does Burst Balloons split on the last balloon burst rather than the first?</summary>

Bursting a balloon makes its neighbours adjacent, so after a first burst at k the two sides interact. If k is burst last, it stands as a wall throughout, so the two sides are independent, and its neighbours at the end are exactly the interval's boundaries.
</details>

<details><summary>2. Why can Cut Stick split on the first cut?</summary>

A cut separates the stick into two pieces that never interact again, so the subproblems are independent immediately.
</details>

<details><summary>3. Explain the minus sign in lead[l][r] = max(nums[l] − lead[l+1][r], …).</summary>

After you take `nums[l]`, the opponent moves on `l+1..r`, and their lead there is exactly their advantage over you. Your overall lead is what you took minus that advantage.
</details>

<details><summary>4. What do the three modes in the cooldown problem record, and why is that enough?</summary>

Whether you hold a share, and whether you're barred from buying today (just sold). The future depends only on those facts, so the best value per mode is a valid DP state.
</details>

<details><summary>5. Why does Stock IV switch to the "take every rise" greedy when k ≥ n/2?</summary>

A useful transaction needs at least two days, so at most n/2 of them can happen. With k ≥ n/2 the limit never binds, and summing every positive daily change is optimal.
</details>

---

## 11. Exit gate

Move on when you can:

* explain the "last action" trick, and contrast Burst Balloons with Cut Stick;
* write Burst Balloons, Cut Stick and Predict the Winner from blank files in the right iteration order;
* draw the automaton for a stock variant and derive its transitions from the arrows;
* derive Strange Printer's merge transition;
* solve 6 of the 7 practice problems.
