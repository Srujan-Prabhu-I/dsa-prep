# Chapter 2 — Proofs and Counterexamples

> **Part 0 · Week 2** · Prerequisites: Ch 1, Ch 4 Part A, and Ch 4 §8 and §14 (lambdas and sorting with a comparator)
> **Problems:** 6 worked (1 ★) · 10 prove-or-refute claims · 4 practice · **Time:** about 6 hours

In easy rounds, people fail because they find no idea. In hard rounds, they fail because they **believe a wrong one**. They code a plausible rule, it passes the examples, and it fails hidden test 37. In a live interview it's worse, because the interviewer usually knows the counterexample and is waiting to see whether you'll find it.

This chapter gives you two complementary skills:

* **Proving:** a short argument, a few sentences rather than a formal paper, that would convince a sceptical interviewer who is actively looking for an input where you're wrong.
* **Breaking:** a systematic way to find the input that kills a wrong idea, by hand in two minutes or by machine in two seconds.

You'll use five proof tools throughout the book. Each one has a one-sentence form you can say out loud (§9).

---

## 1. Attempt first ★

Spend **10 minutes** on this before reading on.

> You have coins worth **1, 3 and 4** (unlimited supply). To pay an amount `A` with the fewest coins, a friend suggests: *"Always take the largest coin that still fits."*
>
> Is the friend right for every `A`? If yes, why? If not, find the smallest `A` where the rule fails.

We'll come back to it in §8.

---

## 2. What a proof has to do

A claim like "this algorithm is correct" is a statement about **every possible input**. Examples can only ever show it works on *those* inputs. A proof has to cover the rest. In practice that means:

1. **State the claim precisely.** Not "greedy works", but "for every valid input, the value returned equals the minimum number of coins".
2. **Give a reason that doesn't depend on the specific numbers.** "It worked on 1, 2, 3" isn't a reason. "Every element is pushed once and popped at most once" is.
3. **Check where the reason uses the problem's conditions.** If your argument never uses "the array is sorted" or "all values are positive", either the condition is unnecessary or your argument has a gap. The places where a proof *uses* a condition are exactly where the algorithm breaks when that condition is removed.

The tools below are reusable *shapes* of reason.

---

## 3. Tool 1: the loop invariant

**An invariant** is a statement about your variables that is true every time the loop reaches a particular point. Proving one has three parts:

| Part | Question |
| :--- | :--- |
| **Initialisation** | Is it true before the first iteration? |
| **Maintenance** | If it's true before an iteration, is it still true after it? |
| **Termination** | When the loop ends, does the invariant say the answer is correct? |

It's induction on the number of iterations, packaged for loops.

### Worked: [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

> Given a **sorted** array `nums` (`n ≥ 1`), remove duplicates in place so each value appears once, keep the relative order, and return the number `k` of distinct values. The first `k` positions must hold them.

```java
public int removeDuplicates(int[] nums) {
    // Invariant (at the top of each iteration): nums[0..write) holds exactly the distinct
    // values of nums[0..i), in order. So nums[write - 1] is the largest value seen so far.
    int write = 1;
    for (int i = 1; i < nums.length; i++) {
        if (nums[i] != nums[write - 1]) {
            nums[write++] = nums[i];
        }
    }
    return write;
}
```

* **Initialisation** (`i = 1`, `write = 1`): `nums[0..1)` is the single value `nums[0]`, which is exactly the distinct values of `nums[0..1)`. ✓
* **Maintenance:** the array is sorted, so `nums[i] ≥` every earlier value, and equal values sit next to each other. So `nums[i]` is a *new* distinct value exactly when it differs from the largest one kept so far, `nums[write − 1]`. If it's new, we append it. If not, we skip it. Either way, the invariant holds for `i + 1`. Also, writing at position `write` never destroys anything we still need, because `write ≤ i`: we only overwrite positions we've already read. ✓
* **Termination** (`i = n`): `nums[0..write)` holds the distinct values of the whole array, in order. ✓

**Where the proof used "sorted":** in maintenance, to say that comparing against `nums[write − 1]` alone is enough. On unsorted input such as `[1, 2, 1]`, the second `1` differs from the last kept value `2` and is wrongly appended. The proof tells you precisely why.

### Worked: [why Boyer–Moore majority voting works](https://leetcode.com/problems/why-boyer-moore-majority-voting-works/)

Ch 1's practice set included **Majority Element**: an element occurs more than n/2 times, so find it in O(1) space. The standard solution is short, but it's believable only once you see the invariant:

```java
public int majorityElement(int[] nums) {
    int candidate = 0, count = 0;
    for (int x : nums) {
        if (count == 0) candidate = x;
        count += (x == candidate) ? 1 : -1;
    }
    return candidate;
}
```

**Invariant:** the elements processed so far can be split into **pairs of two different values**, plus `count` unpaired copies of `candidate`.

* *Initialisation:* nothing processed; zero pairs and `count = 0`. ✓
* *Maintenance:* if `x` equals the candidate (or `count` was 0 and `x` becomes the candidate), `x` joins the unpaired pile. Otherwise `x` pairs with one unpaired candidate copy, and that pair has two different values. ✓
* *Termination:* let `m` be the true majority, occurring more than n/2 times. Each pair contains *at most one* `m`, and there are `(n − count)/2 ≤ n/2` pairs. If `candidate ≠ m`, then every `m` is inside a pair, so `m` occurs at most n/2 times. That's a contradiction, so `candidate = m`. ✓

The proof used "more than n/2" exactly once, at the end. Without a guaranteed majority (input `[1, 2, 3]`), the algorithm still returns *something*, and that something can be wrong. The proof shows you'd need a second pass to verify it.

---

## 4. Tool 2: induction and the recursive contract

**Induction** proves a statement `P(n)` for every `n`: prove `P` for the smallest case (the *base*), then prove that `P` for smaller cases implies `P(n)` (the *step*). You've already seen it twice:

* Ch 1, Derivation D: "greedy's t-th ball is no further right than any valid placement's t-th ball" was proved for `t = 1`, then for `t + 1` from `t`.
* Every loop invariant above is induction on the iteration count.

**The recursive contract** is the same idea applied to recursive functions. To trust a recursive function, check two things:

1. its base cases return the right answer;
2. *assuming* every recursive call on a smaller input returns the right answer, this call combines those answers correctly.

You never trace the whole recursion in your head. You trust the contract on smaller inputs, exactly as induction trusts `P` for smaller `n`. Ch 6 makes this the main way you write recursion.

---

## 5. Tool 3: the exchange argument

Use this when you claim that some **greedy choice** or **sorted order** is optimal. The shape is always:

1. Take *any* optimal solution `O`.
2. If it differs from your greedy solution `G`, find a place where it does.
3. Show that changing `O` at that place, one step toward `G`, **doesn't make it worse**.
4. Repeat. `O` turns into `G` without ever getting worse, so `G` is optimal too.

For **ordering problems** there's a lighter version, the **adjacent swap**: show that whenever two *adjacent* items are out of your order, swapping them doesn't make things worse. Then a solution sorted by your rule is optimal. This needs your rule to be a consistent ordering; see Largest Number below.

### Worked: [Two City Scheduling](https://leetcode.com/problems/two-city-scheduling/)

> `2n` people. Person `i` costs `costs[i][0]` to fly to city A or `costs[i][1]` to city B. Send exactly `n` people to each city at minimum total cost. (`1 ≤ costs ≤ 1000`.)

A tempting rule is "send each person to their cheaper city", but that can put far more than n people in one city. Try a **reformulation** instead. Start with *everyone* going to B, which costs `Σ bᵢ`. Moving person `i` from B to A changes the total by `dᵢ = aᵢ − bᵢ`, which may be negative. We must move exactly `n` people, so

> total = `Σ bᵢ + Σ (dᵢ over the n people moved to A)`.

The first term is fixed. To minimise the second, move the **n people with the smallest `dᵢ`**. So: sort by `aᵢ − bᵢ`, send the first half to A and the rest to B.

The same fact as an exchange argument: suppose an optimal solution sends `i` to A and `j` to B with `dᵢ > dⱼ`. Swap them, `i` to B and `j` to A. The total changes by `dⱼ − dᵢ < 0`, so it gets cheaper, contradicting optimality. So in an optimal solution, everyone sent to A has a `d` no larger than everyone sent to B, which is the sorted split.

```java
public int twoCitySchedCost(int[][] costs) {
    int[][] people = costs.clone();      // shallow copy is fine: we only reorder the rows
    // differences are within ±999, so subtracting inside Integer.compare can't overflow
    Arrays.sort(people, (p, q) -> Integer.compare(p[0] - p[1], q[0] - q[1]));
    int n = people.length / 2, total = 0;
    for (int i = 0; i < people.length; i++) {
        total += (i < n) ? people[i][0] : people[i][1];
    }
    return total;
}
```

### Worked: [Largest Number](https://leetcode.com/problems/largest-number/)

> Arrange non-negative integers so that, written side by side, they form the largest possible number. Return it as a string. Example: `[3, 30, 34, 5, 9]` → `"9534330"`.

Sorting numerically in descending order fails: `[30, 3]` would give `"303"`, but `"330"` is larger. Instead, compare *how two numbers concatenate*: **put `a` before `b` if `a + b > b + a`** as strings. (`"3" + "30" = "330"` beats `"30" + "3" = "303"`, so 3 goes first.)

**Adjacent swap.** In any arrangement, suppose block `a` sits right before block `b` and `b + a > a + b`. Swapping the two blocks leaves everything before and after unchanged, and the middle section, which has a fixed length, gets larger. So the whole number gets larger. An optimal arrangement therefore has no such adjacent pair, which means it's sorted by this rule.

**But is the rule a consistent ordering?** This is the step people skip. Comparators must be **transitive** (if `a` before `b` and `b` before `c`, then `a` before `c`), or sorting produces garbage, as Ch 4 §14 warned. Here's the proof. Write `|x|` for the number of digits of `x`. As numbers, `a + b` (concatenated) = `a · 10^|b| + b`. So

> `a + b > b + a` ⟺ `a · 10^|b| + b > b · 10^|a| + a` ⟺ `a · (10^|b| − 1) > b · (10^|a| − 1)` ⟺ `a / (10^|a| − 1) > b / (10^|b| − 1)`.

So the rule is just "sort by the real number `f(x) = x / (10^|x| − 1)`, descending". Sorting by any numeric key is automatically consistent and transitive. ✓

```java
public String largestNumber(int[] nums) {
    String[] parts = new String[nums.length];
    for (int i = 0; i < nums.length; i++) parts[i] = String.valueOf(nums[i]);
    // a before b  ⟺  a+b > b+a. Both concatenations have the same length,
    // so comparing them as strings compares them as numbers.
    Arrays.sort(parts, (a, b) -> (b + a).compareTo(a + b));
    if (parts[0].equals("0")) return "0";        // every number was 0: not "000"
    StringBuilder sb = new StringBuilder();
    for (String p : parts) sb.append(p);
    return sb.toString();
}
```

---

## 6. Tool 4: greedy stays ahead

Use this when a greedy algorithm builds its solution step by step. Define a **measure of progress**. Then show by induction that **after every step, greedy's progress is at least as good as any other solution's progress after the same number of steps**. If greedy is never behind, it can't finish behind.

Ch 1's Magnetic Force proof was this argument (greedy's t-th ball is never further right). Here is another.

### Worked: [Jump Game II](https://leetcode.com/problems/jump-game-ii/)

> `nums[i]` is the maximum jump length from index `i`; from `i` you may land anywhere in `[i + 1, i + nums[i]]`. Return the minimum number of jumps from index 0 to index `n − 1`. The end is guaranteed reachable.

**Progress measure:** `R(j)` = the furthest index reachable using at most `j` jumps.

**Claim 1 (no gaps):** every index `≤ R(j)` is reachable in at most `j` jumps. *Induction:* the indices reachable in `≤ j + 1` jumps are those reachable in `≤ j`, plus every landing spot `[i + 1, i + nums[i]]` from them. Each of those landing ranges starts right next to a reachable index, so if the reachable set has no gaps, it still has none after one more jump.

**Claim 2:** `R(j + 1) = max(i + nums[i])` over every `i ≤ R(j)`. This follows directly from Claim 1.

So the answer is the smallest `j` with `R(j) ≥ n − 1`. We compute the `R(j)` values level by level, scanning each index once:

```java
public int jump(int[] nums) {
    int n = nums.length;
    int jumps = 0;
    int reachable = 0;        // R(jumps): furthest index reachable with `jumps` jumps
    int nextReachable = 0;    // best i + nums[i] over the indices scanned so far
    for (int i = 0; i < n - 1; i++) {
        nextReachable = Math.max(nextReachable, i + nums[i]);
        if (i == reachable) {             // we've scanned every index reachable with `jumps` jumps
            jumps++;
            reachable = nextReachable;    // this is R(jumps + 1), by Claim 2
            if (reachable >= n - 1) break;
        }
    }
    return jumps;
}
```

A trace on `[2, 3, 1, 1, 4]` (n = 5), where `reachable` is R(jumps) and `nextReachable` is the best i + nums[i] seen so far:

| i | i + nums[i] | nextReachable | i == reachable? | jumps | reachable |
| :-: | :-: | :-: | :--- | :-: | :-: |
| 0 | 2 | 2 | yes: every index reachable with 0 jumps has been scanned | 1 | 2 |
| 1 | 4 | 4 | no | 1 | 2 |
| 2 | 3 | 4 | yes: every index reachable with 1 jump has been scanned | 2 | 4 ≥ n − 1: stop |

The answer is 2. Each "yes" row closes one level: the indices `0..reachable` are exactly those reachable with `jumps` jumps.

This is a greedy that *stays ahead*: after `j` jumps, nobody can be further right than `R(j)`. Contrast it with a greedy that **doesn't** work: *"always jump as far as you can."* On `[2, 3, 1, 1, 4]` that rule goes 0 → 2 → 3 → 4 (3 jumps), but 0 → 1 → 4 takes 2. Jumping far is not the same as *reaching* far.

---

## 7. Tool 5: dominance and "moving" arguments

**Dominance** (from Ch 1, Derivation B): candidate `j` can be thrown away forever because some other candidate `k` is **at least as good for every future question**. To prove it, name the future question and compare `j` and `k` on it.

**Moving argument:** to show the optimum is at some point `x*`, start from any other point `x` and show that **moving toward `x*` never increases the cost**.

### Worked *(custom)*: the median minimises the total distance

> Given numbers `a₁ … aₙ`, find `x` minimising `|x − a₁| + |x − a₂| + … + |x − aₙ|`.

Pick any `x`. Let `L` be the number of values `≤ x` and `R` the number of values `> x`. Move `x` a tiny step `δ` to the right, not far enough to pass any value. Every distance to a value on the right shrinks by `δ`, and every distance to a value on the left grows by `δ`. The cost changes by **`δ · (L − R)`**.

* If `L < R` (more values to the right), moving right *lowers* the cost.
* By the same argument in the other direction, if more values are on the left, moving left lowers it.

So the cost is lowest where neither move helps: **at a point with at least half the values on each side, which is the median**. For even n, any point between the two middle values works.

```java
static long minTotalDistance(int[] a) {
    int[] sorted = a.clone();
    Arrays.sort(sorted);
    int median = sorted[sorted.length / 2];
    long total = 0;
    for (int v : sorted) total += Math.abs((long) v - median);   // long: n · 2·10^9 can overflow int
    return total;
}
```

**The objective matters.** To minimise the sum of *squared* distances, the answer is the **mean**, not the median. On `[1, 2, 9]`, the median 2 gives total distance 8, while the mean 4 gives 10. Change the objective and you change the answer, so re-prove; don't reuse a remembered result.

---

## 8. Breaking ideas: counterexample hunting

### 8.1 By hand: the two-minute checklist

Before you commit to an idea, run it on inputs of these shapes. Most wrong ideas die on one of them:

| Shape | Why it breaks things |
| :--- | :--- |
| n = 1, 2, 3 | Off-by-one errors, empty loops, "the second-best doesn't exist" |
| All values equal | Ties, `<` versus `≤`, duplicate handling |
| Zeros, negatives | Monotonicity assumptions (sums no longer grow as you add elements) |
| Sorted ascending / descending | Worst cases for "scan forward" ideas |
| One huge element among small ones | Rules like "largest first" or "average" |
| Values just around a threshold | Boundary conditions such as `≥ k` versus `> k` |
| An input designed to make *your rule's first choice* bad | Takes your greedy's first move and makes it the wrong one |

The last row is the most powerful. Ask: *"What input would make my rule's very first choice a mistake?"*

### 8.2 The attempt-first problem, solved

Apply the checklist to the coin rule with coins `{1, 3, 4}`. Tabulate small amounts:

| A | Greedy (largest first) | Coins | Best possible | Coins |
| :-: | :--- | :-: | :--- | :-: |
| 1–5 | 1 · 1+1 · 3 · 4 · 4+1 | 1, 2, 1, 1, 2 | the same | same |
| **6** | **4 + 1 + 1** | **3** | **3 + 3** | **2** |

**A = 6** breaks it. Greedy's first choice, taking the 4, is the mistake: it leaves 2, which needs two 1-coins. The general fix is dynamic programming (Ch 28). The habit to keep is that you *checked small cases before trusting the rule*.

### 8.3 By machine: brute-force counterexample search

When the inputs are small enough to enumerate, let the computer hunt. Write the suspicious rule and a brute force that is obviously correct, then compare them on every small input:

```java
public class CoinCounterexample {
    // The suspicious rule: largest coin first. Coins must be listed in descending order.
    static int greedy(int amount, int[] coinsDescending) {
        int used = 0;
        for (int coin : coinsDescending) {
            used += amount / coin;
            amount %= coin;
        }
        return amount == 0 ? used : -1;
    }

    // Obviously correct for three coin types: try every count of the first two coins.
    static int bruteForce(int amount, int c1, int c2, int c3) {
        int best = Integer.MAX_VALUE;
        for (int x = 0; x * c1 <= amount; x++) {
            for (int y = 0; x * c1 + y * c2 <= amount; y++) {
                int rest = amount - x * c1 - y * c2;
                if (rest % c3 == 0) best = Math.min(best, x + y + rest / c3);
            }
        }
        return best == Integer.MAX_VALUE ? -1 : best;
    }

    public static void main(String[] args) {
        int[] coins = {4, 3, 1};
        for (int amount = 1; amount <= 50; amount++) {
            int g = greedy(amount, coins);
            int b = bruteForce(amount, 4, 3, 1);
            if (g != b) {
                System.out.println("Counterexample: amount " + amount
                        + ", greedy uses " + g + ", optimal uses " + b);
                return;
            }
        }
        System.out.println("No counterexample up to 50");
    }
}
```

This prints `Counterexample: amount 6, greedy uses 3, optimal uses 2`.

Two facts about machine search:
* **A single mismatch is a disproof.** You're done: the rule is wrong.
* **No mismatch is evidence, not proof.** "It held up to 50" doesn't mean it holds at 51. Greedy really is optimal for coins `{1, 5, 10, 25}`, but you'd still need a proof to be sure.

Ch 5 turns this into a general **stress-testing harness** that you'll use on every problem.

### 8.4 A small gallery of plausible, wrong rules

Each of these looks right and has a tiny counterexample. Memorise the counterexample, not just the fact that the rule fails. The full museum is in Ch 26.

| Plausible rule | Counterexample | What's actually right |
| :--- | :--- | :--- |
| Coins: take the largest coin that fits | `{1, 3, 4}`, amount 6 | DP (Ch 28) |
| Fill a capacity-10 knapsack by best value/weight ratio | Items (w 6, v 7), (w 5, v 5), (w 5, v 5): the best ratio is the (6, 7) item, which leaves room for nothing else → value 7; optimal is the two 5s → 10 | 0/1 knapsack DP (Ch 28) |
| Choose the most non-overlapping intervals by **earliest start** | `[1,10], [2,3], [4,5]`: picks `[1,10]` → 1; optimal 2 | Sort by **end** (Ch 14) |
| …by **shortest interval** | `[1,5], [4,7], [6,10]`: picks `[4,7]` → 1; optimal 2 | Sort by end (Ch 14) |
| Largest number: sort numerically descending | `[30, 3]` → `"303"` instead of `"330"` | The concatenation comparator (§5) |
| Jump game: always jump as far as possible | `[2, 3, 1, 1, 4]` → 3 jumps; optimal 2 | Stays-ahead levels (§6) |
| Minimise total distance with the **mean** | `[1, 2, 9]` → 10; the median gives 8 | Median (§7) |

---

## 9. The proof sentence: say it out loud

In an interview, nobody wants a formal proof. They want **one or two sentences that show you know *why***. The templates:

| Tool | Sentence |
| :--- | :--- |
| Invariant | "After each iteration, ___ holds. When the loop ends, that's exactly the answer." |
| Induction / recursion | "Assuming the recursive calls are correct for smaller inputs, combining them this way is correct, and the base cases are right." |
| Exchange | "Any optimal solution can be turned into mine one swap at a time without getting worse." |
| Stays ahead | "After k steps, mine has progressed at least as far as any other solution after k steps." |
| Monotone | "If x is feasible, so is every x′ ≥ x, because ___." |
| Dominance | "Candidate j can never be the answer again, because candidate k is at least as good for every future query." |
| Counterexample | "This rule fails on ___: it picks ___ first, but the optimum is ___." |

---

## 10. Practice

### 10.1 Prove or refute: 10 claims

For each claim, decide whether it's true or false. If it's true, write the one-sentence proof and name the tool. If it's false, give the **smallest** counterexample you can. Five minutes each.

1. In any array, the maximum subarray sum is either the sum of the whole array or a single element.
2. If every element is positive, then for each right end `r`, the smallest `l` with `sum(a[l..r]) ≤ S` never moves left as `r` increases.
3. To minimise the total waiting time of `n` customers served one at a time, serve them in increasing order of service time.
4. To attend the most non-overlapping meetings, repeatedly pick the meeting that starts earliest among those that don't overlap what you've chosen.
5. In a sorted array, if `a[l] + a[r] > target`, then no pair `(l′, r)` with `l ≤ l′ < r` sums to `target`.
6. In a sorted array with `n ≥ 1`, the number of distinct values equals 1 plus the number of indices `i ≥ 1` with `a[i] ≠ a[i − 1]`.
7. If an element occurs more than n/2 times, then after sorting it is always at index `n / 2`.
8. To make all elements equal using +1/−1 moves, moving everything to the mean (rounded) minimises the number of moves.
9. To split an array of positive numbers into the fewest contiguous parts with each part's sum ≤ C, greedily make each part as long as possible from the left.
10. Choosing the k largest elements maximises their sum, and also maximises their product.

<details><summary>Answers</summary>

1. **False.** `[−1, 2, 3, −1]`: the best is `[2, 3]` = 5, while the whole array is 3 and the best single element is 3.
2. **True (monotone).** With positive values, a window that is too big stays too big as `r` grows. So the smallest valid `l` for `r + 1` can't be less than the one for `r`. (Ch 11 builds the sliding window on this.)
3. **True (adjacent swap).** If a longer job sits right before a shorter one, swapping them cuts the second customer's wait by the difference and changes nothing else.
4. **False.** `[1, 10], [2, 3], [4, 5]`: earliest start takes `[1, 10]` and gets 1 meeting; the optimum is 2.
5. **True (dominance).** Sorted means `a[l′] ≥ a[l]`, so `a[l′] + a[r] ≥ a[l] + a[r] > target`. The value `a[r]` can be discarded. (This is the core of two pointers, Ch 11.)
6. **True (invariant).** Each run of equal values is announced by exactly one index where the value changes, except the first run, which is counted by the "1".
7. **True.** A block of more than n/2 equal values in a sorted array must cover the middle index.
8. **False.** `[1, 2, 9]`: the mean 4 needs 3 + 2 + 5 = 10 moves; the median 2 needs 1 + 0 + 7 = 8.
9. **True (stays ahead).** After each part, greedy's cut is at least as far right as any valid split's corresponding cut. That's the same argument as Ch 1, Derivation D.
10. **Sum true, product false.** `[−5, −4, 1, 2]`, k = 2: the largest two give 1 · 2 = 2, but (−5)(−4) = 20.
</details>

### 10.2 Problems

For each one, write the proof sentence (§9) *before* coding.

1. **[Assign Cookies](https://leetcode.com/problems/assign-cookies/)** *(Easy)*
2. **[Boats to Save People](https://leetcode.com/problems/boats-to-save-people/)** *(Medium)*
3. **[Minimum Deletions to Make Character Frequencies Unique](https://leetcode.com/problems/minimum-deletions-to-make-character-frequencies-unique/)** *(Medium)*
4. **[Queue Reconstruction by Height](https://leetcode.com/problems/queue-reconstruction-by-height/)** *(Medium)*

<details><summary>Nudges</summary>

1. Sort both arrays. Should the smallest cookie go to the least greedy child it can satisfy? Prove it by exchange.
2. Sort. The heaviest person must ride with somebody or alone. Who is the best partner to try, and why does pairing them never hurt?
3. Count the frequencies. Process them from highest to lowest; each must be strictly lower than the last one you kept.
4. Place the tallest people first. Why does inserting a shorter person later never disturb the counts of the taller people already placed?
</details>

<details><summary>Key</summary>

Assign Cookies → exchange (smallest cookie to the least greedy child it satisfies) · Boats → exchange (pair the heaviest with the lightest if they fit; otherwise the heaviest rides alone) · Min Deletions → "moving" argument, processing from the highest frequency down · Queue Reconstruction → exchange + insertion invariant (sort by height descending, then k ascending; insert at index k)
</details>

---

## 11. Self-test

<details><summary>1. Name the three parts of an invariant proof.</summary>

Initialisation, maintenance, termination.
</details>

<details><summary>2. Your greedy passes 10 000 random tests against a brute force. Is it proven correct?</summary>

No. That's strong evidence, but only a proof covers all inputs. One failing test, though, *is* a disproof.
</details>

<details><summary>3. Why must you prove that the Largest Number comparator is transitive?</summary>

Sorting assumes a consistent order. A non-transitive comparator can produce wrong orders, or make Java throw "Comparison method violates its general contract!". The proof shows the rule is really "sort by a numeric key", which is always consistent.
</details>

<details><summary>4. In the Boyer–Moore proof, where exactly is "more than n/2" used?</summary>

At termination: the pairs number at most n/2, and each holds at most one copy of the majority element. So if the candidate weren't the majority, the majority would occur at most n/2 times.
</details>

<details><summary>5. Give the shortest counterexample to "always jump as far as possible" in Jump Game II.</summary>

`[2, 3, 1, 1, 4]`: farthest-first takes 3 jumps; the optimum is 2 (0 → 1 → 4).
</details>

---

## 12. Exit gate

Move on when you can:

* write the invariant proof for Remove Duplicates and the Boyer–Moore proof from memory;
* give the exchange argument for Two City Scheduling and the transitivity argument for Largest Number in under 2 minutes each, out loud;
* answer 9 of the 10 claims correctly, with a minimal counterexample for each false one;
* run the machine counterexample search (§8.3) on a coin system of your own choosing.
