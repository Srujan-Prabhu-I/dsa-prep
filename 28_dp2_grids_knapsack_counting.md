# Chapter 28 — DP II: Grids, Knapsack & Counting

> **Part VI · Week 25** · Prerequisites: Ch 27 (the six steps and the bridge), Ch 18 §5 (counting principles)
> **Problems:** 7 worked (1 ★) · 10 practice · 3 stretch · **Time:** about 9 hours

Three families make up a large share of all DP questions:
* **Grid paths:** the state is a cell, and the last move came from a neighbour.
* **Knapsack:** choose items under a capacity, one at a time.
* **Counting:** the number of ways, where the real difficulty is counting each object **exactly once**.

They share one subtle fact, which this chapter makes precise. **In a rolled-up DP, the direction and nesting of the loops *are* the recurrence.** Reverse one loop and "each item once" becomes "unlimited items". Swap the two loops and "combinations" becomes "ordered sequences". You'll be able to say *why*, not just remember which is which.

---

## 1. Attempt first ★

> **[Unique Paths II](https://leetcode.com/problems/unique-paths-ii/) (20 minutes).** A robot moves only right or down across a grid from the top-left to the bottom-right. Some cells are obstacles (1). Count the paths.

---

## 2. Grid DP

### Worked K1a · [Unique Paths II](https://leetcode.com/problems/unique-paths-ii/) ★

**State:** `ways[r][c]` = the number of paths from the start to cell (r, c). **Last move:** into (r, c) from above or from the left, two **different** paths, so add them. Obstacles have 0 ways. **Order:** row by row, left to right, so both sources are ready.

**Rolling to one row:** `ways[r][c]` needs only the cell above (from the previous row) and the cell to the left (from the current row). A single array works. Before `ways[c]` is updated, it still holds the row above; `ways[c − 1]` already holds the current row.

```java
public int uniquePathsWithObstacles(int[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    int[] ways = new int[cols];                    // before updating ways[c]: the row above; after: this row
    ways[0] = (grid[0][0] == 1) ? 0 : 1;
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == 1) ways[c] = 0;                  // no path enters an obstacle
            else if (c > 0) ways[c] += ways[c - 1];           // from above (old ways[c]) + from the left
        }
    }
    return ways[cols - 1];
}
```

O(R · C) time, O(C) space. Without obstacles, Ch 18 §5 gives the closed form C(R + C − 2, R − 1). DP is what still works once the grid gets obstacles, costs or other constraints.

### Worked K1b · [Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/)

> Find the right/down path from the top-left to the bottom-right with the smallest sum of cell values.

The same shape, with `min` in place of `+`. **State:** the cheapest path sum to (r, c). **Last move:** the cheaper of from-above and from-the-left, plus this cell.

```java
public int minPathSum(int[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    int[] best = new int[cols];                    // best[c] = the cheapest path sum to (current row, c)
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (r == 0 && c == 0) best[c] = grid[0][0];
            else if (r == 0) best[c] = best[c - 1] + grid[r][c];        // first row: only from the left
            else if (c == 0) best[c] = best[c] + grid[r][c];            // first column: only from above
            else best[c] = Math.min(best[c], best[c - 1]) + grid[r][c];
        }
    }
    return best[cols - 1];
}
```

Triangle and Maximal Square (practice) are grid DPs too. Maximal Square's transition takes the *minimum of three* neighbours, and its proof is a nice exercise.

---

## 3. When the forward direction doesn't work: go backwards

### Worked K2 · [Dungeon Game](https://leetcode.com/problems/dungeon-game/)

> A knight moves right or down from the top-left to the princess at the bottom-right. Each room changes his health (negative = damage, positive = potions), and his health must stay **≥ 1 at every moment**. Return the minimum starting health.

**Why the forward direction fails.** Suppose you try to record "the best health on arriving at (r, c)". There are *two* competing goals, high current health and a high minimum along the way. One path can arrive richer, but after dipping dangerously low earlier. A single forward number can't rank paths, so **optimal substructure fails for that state**.

**Reverse the question (Ch 1, move 5):** what's the **least health needed on *entering* (r, c)** to survive to the end? From (r, c) the knight goes right or down, and he'll choose whichever needs less. That's a clean single-criterion recurrence:

> `need[r][c] = max(1, min(need[r+1][c], need[r][c+1]) − dungeon[r][c])`.

The `max(1, …)` encodes "health never drops below 1", even when a big potion here would make the formula smaller.

```java
public int calculateMinimumHP(int[][] dungeon) {
    int rows = dungeon.length, cols = dungeon[0].length;
    int[][] need = new int[rows + 1][cols + 1];    // need[r][c] = least health on entering (r, c)
    for (int[] row : need) Arrays.fill(row, Integer.MAX_VALUE);   // off-grid: impossible
    need[rows][cols - 1] = need[rows - 1][cols] = 1;              // just past the princess, 1 health suffices
    for (int r = rows - 1; r >= 0; r--) {
        for (int c = cols - 1; c >= 0; c--) {
            int onward = Math.min(need[r + 1][c], need[r][c + 1]);     // the easier way onward
            need[r][c] = Math.max(1, onward - dungeon[r][c]);
        }
    }
    return need[0][0];
}
```

**When the answer at a state depends on the future rather than the past, define the state by what the future requires,** and fill from the end.

---

## 4. The 0/1 knapsack: each item at most once

**The general problem:** items with weights (and values); a capacity; **each item can be used at most once**. Pick a subset to maximise the value, or to hit a target exactly, or to count the ways.

> **State:** `dp[i][s]` = the answer using only the **first i items** with total weight (or sum) s.
> **Last decision:** item i − 1 is either left out (`dp[i−1][s]`) or taken (`dp[i−1][s − wᵢ]`, plus its value).

The cost is O(n · S), where S is the capacity. It's polynomial in S, so when S is up to about 10⁵–10⁶ it's fast. When S is 10⁹, it isn't (§8).

### Worked K3 · [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/)

> Can the array be split into two subsets with equal sums?

Equal halves means one subset sums to **exactly total/2** (an odd total is immediately impossible). That's subset-sum, a 0/1 knapsack:

**Rolling to one array: iterate the sum *downward*.** The update `reachable[s] |= reachable[s − x]` must read the value from **before** item x was considered, which is the previous row. Going downward, `s − x` is smaller than s, so it hasn't been overwritten yet in this pass. Going upward would read an already-updated value and use x **twice**.

```java
public boolean canPartition(int[] nums) {
    int total = 0;
    for (int x : nums) total += x;
    if (total % 2 != 0) return false;
    int target = total / 2;
    boolean[] reachable = new boolean[target + 1];   // reachable[s] = some subset of the items so far sums to s
    reachable[0] = true;                              // the empty subset
    for (int x : nums) {
        for (int s = target; s >= x; s--) {           // DOWNWARD: reachable[s - x] still describes the earlier items
            if (reachable[s - x]) reachable[s] = true;
        }
    }
    return reachable[target];
}
```

**See the difference** with the single item x = 1 and target 3, starting from `reachable = [T, F, F, F]`:

```text
downward (s = 3, 2, 1):   s=3 reads [2] = F;  s=2 reads [1] = F;  s=1 reads [0] = T → [T, T, F, F]   correct: {1} reaches 1 only
upward   (s = 1, 2, 3):   s=1 reads [0] = T → [1] = T;  s=2 reads [1] = T (just set!) → T;  s=3 → T
                          result [T, T, T, T]: it "reached" 2 and 3 by using the single 1 two and three times
```

Going upward is exactly the unbounded knapsack (§5), which is why both loops look the same apart from their direction.

O(n · total). Target Sum and Last Stone Weight II (practice) are subset-sum problems in disguise. Rewriting them as one is the main insight in both.

---

## 5. Unbounded knapsack: unlimited copies

### Worked K4 · [Coin Change](https://leetcode.com/problems/coin-change/)

> Return the fewest coins (each denomination unlimited) that make up `amount`, or −1.

Ch 2 and Ch 26 showed that the greedy rule fails ({1, 3, 4}, amount 6). **State:** `fewest[a]` = the fewest coins making exactly a. **Last coin:** some coin c ≤ a, after which you need `fewest[a − c]`. Coins are unlimited, so there's no "item index" in the state at all.

```java
public int coinChange(int[] coins, int amount) {
    final int INF = amount + 1;                      // more than any real answer: amount coins of value 1
    int[] fewest = new int[amount + 1];
    Arrays.fill(fewest, INF);
    fewest[0] = 0;
    for (int a = 1; a <= amount; a++) {
        for (int c : coins) {
            if (c <= a && fewest[a - c] + 1 < fewest[a]) fewest[a] = fewest[a - c] + 1;   // the last coin is c
        }
    }
    return fewest[amount] >= INF ? -1 : fewest[amount];
}
```

O(amount · coins). **INF = amount + 1** is safe: no answer needs more than `amount` coins, and INF + 1 can't overflow.

---

## 6. Loop order *is* the recurrence: combinations versus sequences

### Worked P4 · [Coin Change II](https://leetcode.com/problems/coin-change-ii/) versus [Combination Sum IV](https://leetcode.com/problems/combination-sum-iv/)

> **Coin Change II:** count the **combinations** of coins summing to `amount` (1 + 2 and 2 + 1 are the same).
> **Combination Sum IV:** count the **ordered sequences** summing to `target` (1 + 2 and 2 + 1 are different).

The code is identical except for **which loop is outside**:

```java
// Coin Change II: combinations. OUTER loop over coins.
public int change(int amount, int[] coins) {
    int[] ways = new int[amount + 1];        // after processing the coins seen so far: combinations of them making a
    ways[0] = 1;
    for (int c : coins) {
        for (int a = c; a <= amount; a++) ways[a] += ways[a - c];   // upward: coin c may be used again
    }
    return ways[amount];
}

// Combination Sum IV: ordered sequences. OUTER loop over totals.
public int combinationSum4(int[] nums, int target) {
    int[] ways = new int[target + 1];        // ways[t] = ordered sequences summing to t
    ways[0] = 1;
    for (int t = 1; t <= target; t++) {
        for (int x : nums) if (x <= t) ways[t] += ways[t - x];      // x is the LAST number in the sequence
    }
    return ways[target];
}
```

**Why, in terms of the six steps:**
* **Combinations.** The state is really `ways[i][a]` = combinations using only the **first i coin types**. The last decision is "use zero copies of coin i" or "use at least one (remove one copy)": `ways[i][a] = ways[i−1][a] + ways[i][a − cᵢ]`. Every combination is built in **coin order** (all of coin 1, then all of coin 2, …), so each multiset is counted exactly once. With coins outside, the array always holds the current "first i coin types".
* **Sequences.** The state is `ways[t]`, and the last decision is **the last number in the sequence**. Different last numbers are different sequences, so 1 + 2 and 2 + 1 are both counted. With totals outside, every number is tried at every total.

(In Combination Sum IV, intermediate entries for unreachable totals can overflow `int`. That's harmless: addition wraps consistently modulo 2³², and the final answer is guaranteed to fit.)

**Every rolled 1D knapsack, in one table:**

| Each item | Count | Outer loop | Inner loop | Inner direction |
| :--- | :--- | :--- | :--- | :--- |
| **at most once** (0/1) | combinations / feasibility / best value | items | capacity | **downward** |
| **unlimited** | combinations / best value | items | capacity | **upward** |
| **unlimited** | **ordered sequences** | capacity | items | (upward) |

---

## 7. Counting without double counting

A counting DP is correct only when **every object has exactly one "last decision"** in your transition. The standard ways to guarantee it:
* **Fix a canonical order.** Combinations are counted in coin order (K5); sets are built in index order (Ch 6's `start` parameter).
* **Make the last decision a property every object has exactly once:** "the last piece", "the last number", "the position of the first chosen element".
* **For objects that really are different, use a sum. For overlapping descriptions, use inclusion–exclusion (Ch 18) or switch to a disjoint description.**

**Check every counting DP on a tiny case by hand**, or against a brute force (Ch 5). Double counting doesn't crash anything. It just produces a wrong number.

---

## 8. Where knapsack DP breaks

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Capacity around 10⁹ | n · S is far too big | Swap the roles: `dp[value] = the minimum weight` if values are small; or meet in the middle for n ≤ 40 (Ch 37) |
| Iterating upward in a 0/1 knapsack | Items get reused | Downward |
| Negative weights or sums | Array indices go negative | Offset the index by the most negative possible sum (Target Sum) |
| "Each item at most kᵢ times" (bounded) | Neither 0/1 nor unbounded | Split each item into powers of two (1, 2, 4, …) and run 0/1 (binary splitting) |
| Two capacities (count *and* weight) | — | A 2D state `dp[c1][c2]` (Ones and Zeroes, practice) |

---

## 9. Recognition

**Signals:**
* **Grid** movement restricted to right/down (or another acyclic direction) → grid DP.
* An answer that depends on **future requirements** (the minimum health needed, the minimum initial resources) → **backward DP**.
* **Choose a subset** subject to a capacity, sum or budget → **0/1 knapsack**.
* **Unlimited** supply of each kind (coins, pieces, steps) → **unbounded knapsack**.
* "**Number of ways**": ask whether order matters, which decides **combinations (items outer)** versus **sequences (totals outer)**.
* Splitting into two groups with equal or closest sums → **subset sum to total/2**.

**False friends:** "minimum number of coins" with a canonical coin system can be greedy, but only with a proof, so default to DP. A grid with moves in all four directions isn't a DAG, so you need BFS/Dijkstra (Ch 21, Ch 23).

---

## 10. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Upward iteration in 0/1 knapsack | Downward (§4) |
| Loops nested the wrong way for combinations / sequences | Items outside for combinations, totals outside for sequences (§6) |
| INF + 1 overflow | INF = amount + 1, or `Integer.MAX_VALUE / 2` |
| A missing `dp[0] = 1` base case in counting | The empty selection counts once |
| A negative sum used as an index | An offset |
| Grid DP that reads a cell before it's computed | Check the loop order against the dependencies |
| Counting without the modulus, or `int` overflow | Reduce after every addition; `long` intermediate values |

---

## 11. Practice

1. **[Triangle](https://leetcode.com/problems/triangle/)** *(Medium)*
2. **[Maximal Square](https://leetcode.com/problems/maximal-square/)** *(Medium)*
3. **[Target Sum](https://leetcode.com/problems/target-sum/)** *(Medium)*
4. **[Last Stone Weight II](https://leetcode.com/problems/last-stone-weight-ii/)** *(Medium)*
5. **[Ones and Zeroes](https://leetcode.com/problems/ones-and-zeroes/)** *(Medium)*
6. **[Number of Dice Rolls With Target Sum](https://leetcode.com/problems/number-of-dice-rolls-with-target-sum/)** *(Medium)*
7. **[Domino and Tromino Tiling](https://leetcode.com/problems/domino-and-tromino-tiling/)** *(Medium)*
8. **[Knight Probability in Chessboard](https://leetcode.com/problems/knight-probability-in-chessboard/)** *(Medium)*
9. **[CSES Coin Combinations II](https://cses.fi/problemset/task/1636)** *(Medium; stdin/stdout)*
10. **[CSES Book Shop](https://cses.fi/problemset/task/1158)** *(Medium; stdin/stdout, tight time limit)*

**Stretch:** **[Cherry Pickup](https://leetcode.com/problems/cherry-pickup/)** *(Hard)* · **[Profitable Schemes](https://leetcode.com/problems/profitable-schemes/)** *(Hard)* · **[Number of Ways to Rearrange Sticks With K Sticks Visible](https://leetcode.com/problems/number-of-ways-to-rearrange-sticks-with-k-sticks-visible/)** *(Hard)*

<details><summary>Nudges</summary>

1. Triangle: bottom-up. Each cell plus the smaller of its two children; one array.
2. Maximal Square: `side[r][c]` = 1 + the min of the left, top and top-left sides, when the cell is 1. Why the minimum of all three?
3. Target Sum: with P = the "+" set and N = the "−" set, P − N = target and P + N = total, so P = (target + total) / 2. Count the subsets with sum P (0/1, downward). Check parity and range.
4. Last Stone II: the result is |S₁ − S₂| for some split of the stones into two groups. Find the subset sum closest to total/2.
5. Ones and Zeroes: a 0/1 knapsack with two capacities (zeros, ones); both loops downward.
6. Dice: `ways[i][s]`: the last die shows 1…k. A bounded sum, mod 10⁹ + 7.
7. Tromino: work out the recurrence from small n, f(n) = 2·f(n−1) + f(n−3), or track a partial-fill state per column.
8. Knight Probability: `p[step][r][c]`. Each step moves to 8 squares with probability 1/8 each; add up what stays on the board.
9. Coin Combinations II: K5 combinations, mod 10⁹ + 7, with fast I/O. (Coin Combinations I is the sequence version.)
10. Book Shop: a 0/1 knapsack, maximising pages within the budget. Roll to 1D downward; `int` arrays, fast I/O.
11. Stretch, Cherry Pickup: two walkers move together (there and back is equivalent to two forward walks); the state is (step, row₁, row₂), and a shared cell counts once.
12. Stretch, Profitable Schemes: `dp[members][profit]`, with profit capped at minProfit (anything above counts the same); 0/1 over crimes.
13. Stretch, Sticks Visible: place the **shortest** stick last. If it's at the front, it's visible (one way); otherwise it's hidden behind something (n − 1 positions). So f(n, k) = f(n−1, k−1) + (n − 1)·f(n−1, k).
</details>

<details><summary>Key</summary>

Triangle → K1 bottom-up · Maximal Square → K1 (min of three neighbours) · Target Sum → K3 counting after the algebra · Last Stone II → K3 reformulation (closest to sum/2) · Ones and Zeroes → 2D K3 · Dice → bounded counting · Tromino → a counting recurrence · Knight Probability → probability DP · Coin Combinations II → K5 · Book Shop → K3 with tight memory/time in Java · Cherry Pickup → two walkers in lockstep · Profitable Schemes → K3 with a capped dimension · Sticks Visible → insert the smallest stick last (counting)
</details>

---

## 12. Self-test

<details><summary>1. Why must 0/1 knapsack iterate the capacity downward when rolled to 1D?</summary>

The update must read `dp[s − w]` from *before* this item was considered. Going downward, `s − w` hasn't been overwritten yet in this pass; going upward, it may already include this item, which reuses it.
</details>

<details><summary>2. Why does putting coins in the outer loop count combinations rather than sequences?</summary>

The array always describes "combinations using the first i coin types", so every combination is built in coin order exactly once. With totals outside, the last number varies at every total, which counts every ordering.
</details>

<details><summary>3. Why does Dungeon Game run backwards?</summary>

The forward state would need two criteria at once (current health and the minimum along the path) and can't rank paths. "The least health needed on entering" depends only on the future, which is a single value, so optimal substructure holds.
</details>

<details><summary>4. How does Target Sum turn into subset sum?</summary>

With P = the sum of the "+" numbers and N = the sum of the "−" numbers: P − N = target and P + N = total, so P = (target + total)/2. Count the subsets with sum P.
</details>

<details><summary>5. When is the O(n · S) knapsack too slow, and what then?</summary>

When S is huge (10⁹). Index by value instead (`dp[value] = min weight`) if the values are small, or use meet in the middle for n ≤ 40.
</details>

---

## 13. Exit gate

Move on when you can:

* write grid DP rolled to one row, and Dungeon Game backwards, from blank files;
* explain, without notes, why 0/1 knapsack iterates downward and unbounded iterates upward;
* derive why the loop nesting decides combinations versus sequences (§6), and reproduce the summary table;
* turn a "split into two groups" problem into subset sum;
* solve 8 of the 10 practice problems.
