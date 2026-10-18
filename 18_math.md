# Chapter 18 — Math for Coding Rounds

> **Part III · Weeks 15–16** · Prerequisites: Ch 2 (proofs), Ch 3 (harmonic sums), Ch 4 §9–10 (overflow, `%`), Ch 10 (binary search), Ch 17 (bits)
> **Problems:** 9 worked (1 ★) · 14 practice · 3 stretch · **Time:** about 10 hours

Interview math is a small, well-defined toolkit, not a degree's worth of number theory: **primes, gcd, modular arithmetic, counting, and expectation**. Each tool is tested because it turns a hopeless enumeration into a formula or a fast loop. OAs, especially in Indian placement rounds, use them heavily: "return the count modulo 10⁹ + 7" appears constantly.

This chapter covers each tool with the one proof you need to trust it, and nothing beyond what problems actually use.

---

## 1. Attempt first ★

> **[Count Primes](https://leetcode.com/problems/count-primes/) (20 minutes).** Count the primes less than `n` (`n ≤ 5·10⁶`). Testing each number by trial division is too slow. Why, and what can you reuse between numbers?

---

## 2. Primes

**Testing one number.** n > 1 is prime if no d in `[2, √n]` divides it. Checking up to √n is enough: if n = a · b with a ≤ b, then a ≤ √n, so any composite n has a factor no larger than √n. That's O(√n) per number. Writing the loop as `d * d <= n`, with `long d` when n is large, avoids floating-point square roots.

### Worked N1 · [Count Primes](https://leetcode.com/problems/count-primes/)

Trial division for every number below 5·10⁶ costs about n√n ≈ 10¹⁰. **Reverse the work** (Ch 1, move 5): instead of asking "does anything divide i?", let each prime **cross out its multiples**. This is the *sieve of Eratosthenes*.

* When the loop reaches `i` and it hasn't been crossed out, no smaller prime divides it, so **i is prime**.
* Start crossing out at `i · i`: any smaller multiple `i · k` with k < i has a prime factor smaller than i, which already crossed it out.

```java
public int countPrimes(int n) {
    if (n < 3) return 0;
    boolean[] composite = new boolean[n];               // composite[i] for i in [0, n)
    int count = 0;
    for (int i = 2; i < n; i++) {
        if (composite[i]) continue;
        count++;                                         // i survived every smaller prime: it's prime
        for (long j = (long) i * i; j < n; j += i) composite[(int) j] = true;   // long: i*i can overflow
    }
    return count;
}
```

**Cost:** the crossing-out work is n/2 + n/3 + n/5 + …, summed over the primes, which is **O(n log log n)**, essentially linear. (Summing n/i over *all* i would give the harmonic n ln n of Ch 3. Restricting it to primes makes it smaller still.)

**Factorising many numbers: the smallest-prime-factor sieve.** Store, for each number, its **smallest prime factor**. Then factorising x is repeated division, O(log x), because each step at least halves x:

```java
int[] spf = new int[MAX + 1];                     // spf[x] = smallest prime factor of x
for (int i = 2; i <= MAX; i++) {
    if (spf[i] != 0) continue;                    // already has a smaller prime factor
    for (int j = i; j <= MAX; j += i) if (spf[j] == 0) spf[j] = i;
}
// factorise x:
while (x > 1) {
    int p = spf[x];                               // p is a prime factor of x
    x /= p;
}
```

Use it when there are many numbers up to about 10⁷. For a single number up to 10¹², trial division to √x (10⁶ steps) is fine.

---

## 3. GCD, LCM and Bézout

**Euclid's algorithm:** `gcd(a, b) = gcd(b, a mod b)`, and `gcd(a, 0) = a`.

*Why it's correct:* any d dividing both a and b also divides `a − q·b = a mod b`. Conversely, anything dividing b and `a mod b` divides a. So the two pairs have exactly the same common divisors, and therefore the same greatest one. *Why it's fast:* every two steps at least halve the larger number, so it takes O(log min(a, b)) steps.

```java
static long gcd(long a, long b) { return b == 0 ? a : gcd(b, a % b); }
static long lcm(long a, long b) { return a / gcd(a, b) * b; }     // divide FIRST: a * b can overflow
```

**Bézout's identity:** for any a, b there are integers x, y with `a·x + b·y = gcd(a, b)`. Moreover, the numbers expressible as `a·x + b·y` are **exactly the multiples of gcd(a, b)**.

### Worked N2 · [Water and Jug Problem](https://leetcode.com/problems/water-and-jug-problem/)

> Jugs of capacity x and y, and unlimited water. You may fill a jug, empty a jug, or pour one into the other until it's empty or full. Can you end up with exactly `target` litres in total?

**Invariant (Ch 1, move 9):** let g = gcd(x, y). **Each jug's content is always a multiple of g.** Filling sets a jug to x or y, emptying sets it to 0, and pouring moves an amount equal to one jug's content or the other's free space, and each of those is a multiple of g. So **the total is always a multiple of g**. Conversely, by Bézout, every multiple of the gcd up to x + y can be reached. (Repeatedly filling one jug and pouring it into the other walks through all of them.) And the total can never exceed x + y.

```java
public boolean canMeasureWater(int x, int y, int target) {
    if (target > x + y) return false;              // more than both jugs can hold
    return target % gcd(x, y) == 0;                 // every reachable total is a multiple of gcd(x, y)
}
```

O(log) instead of a breadth-first search over every (amount in x, amount in y) state, which would also work (Ch 21) but needs O(x · y).

---

## 4. Modular arithmetic

**Why "modulo 10⁹ + 7".** Counting answers grow astronomically, and the problem lets you return the remainder instead. 10⁹ + 7 is prime (which matters for division, below), and the product of two remainders is less than 1.0000001·10¹⁸, so it fits in `long`.

**The rules** (Ch 4 §10), with every value kept in `[0, MOD)`:
* addition: `(a + b) % MOD`;
* subtraction: `((a − b) % MOD + MOD) % MOD`;
* multiplication: `(a * b) % MOD`, with `long a, b`;
* **reduce after every single operation**, not just at the end.

**Division is different.** You can't divide remainders directly: 6/3 = 2, but 6 mod 5 = 1 and 1/3 isn't an integer. Instead, multiply by the **modular inverse** b⁻¹, the number with `b · b⁻¹ ≡ 1 (mod p)`. §5 shows how to compute it.

### Fast exponentiation

Computing `aᵉ mod m` by multiplying e times is hopeless when e is 10¹⁸. **Write e in binary.** aᵉ is the product of `a^(2ⁱ)` over the set bits i of e, and each `a^(2^(i+1))` is just `a^(2ⁱ)` squared. That's O(log e) multiplications:

```java
static long power(long base, long exp, long mod) {
    long result = 1 % mod;
    base %= mod;
    if (base < 0) base += mod;
    while (exp > 0) {
        if ((exp & 1) == 1) result = result * base % mod;   // bit i of exp is set: include base^(2^i)
        base = base * base % mod;                          // base^(2^i) -> base^(2^(i+1))
        exp >>= 1;
    }
    return result;
}
```

### Worked N3a · [Count Good Numbers](https://leetcode.com/problems/count-good-numbers/)

> A digit string of length n (up to 10¹⁵) is "good" if digits at even indices are even (0, 2, 4, 6, 8) and digits at odd indices are prime (2, 3, 5, 7). Count them modulo 10⁹ + 7.

**The rule of product:** the positions are independent, so multiply their choices. There are ⌈n/2⌉ even indices with 5 choices each and ⌊n/2⌋ odd indices with 4 choices each:

```java
public int countGoodNumbers(long n) {
    final long MOD = 1_000_000_007L;
    return (int) (power(5, (n + 1) / 2, MOD) * power(4, n / 2, MOD) % MOD);
}
```

### Worked N3b · [Super Pow](https://leetcode.com/problems/super-pow/)

> Compute `a^b mod 1337`, where b is a huge number given as an array of digits.

Read b's digits left to right. If b = 10q + d, then **a^b = (a^q)¹⁰ · a^d**. So keep `result = a^(the digits seen so far)` and update it one digit at a time:

```java
public int superPow(int a, int[] b) {
    long result = 1;
    for (int digit : b) result = power(result, 10, 1337) * power(a, digit, 1337) % 1337;
    return (int) result;
}
```

That's O(len(b)), with no huge number ever formed.

---

## 5. Counting

| Principle | Statement | Example |
| :--- | :--- | :--- |
| **Product** | Independent choices multiply | Good Numbers: 5 · 4 · 5 · 4 · … |
| **Sum** | Disjoint cases add | Paths ending with a right move + paths ending with a down move |
| **Permutations** | n distinct items in a row: n! | — |
| **Combinations** | Choose k of n, order irrelevant: C(n, k) = n! / (k! (n − k)!) | — |
| **Pascal's rule** | C(n, k) = C(n − 1, k − 1) + C(n − 1, k) | Item n is either chosen or not |
| **Stars and bars** | n identical items into k boxes: C(n + k − 1, k − 1) | Count Sorted Vowel Strings (Mixed Set III) |
| **Multinomial** | Arrangements of a word with letter counts c₁, c₂, …: n! / (c₁! c₂! …) | Count Anagrams (practice) |
| **Inclusion–exclusion** | \|A ∪ B\| = \|A\| + \|B\| − \|A ∩ B\| | §6 |
| **Catalan** | Balanced parenthesisations: C(2n, n) / (n + 1) | Generate Parentheses (Ch 6) |

**Stars and bars, briefly.** Line up n stars, then insert k − 1 bars to cut them into k groups (groups may be empty). Every arrangement of n stars and k − 1 bars in a row gives a distribution, and vice versa. There are n + k − 1 positions, and you choose which k − 1 of them are bars.

### Worked N4a · [Unique Paths (by formula)](https://leetcode.com/problems/unique-paths/)

> A robot moves only right or down across an m × n grid, from the top-left to the bottom-right. How many paths are there?

Every path is a sequence of exactly m − 1 "down" moves and n − 1 "right" moves, in some order. Choosing **which** of the m + n − 2 moves are downs determines the path, so there are **C(m + n − 2, m − 1)** paths. Compute it without huge factorials by building the binomial one factor at a time:

```java
public int uniquePaths(int m, int n) {
    long paths = 1;
    for (int i = 1; i <= m - 1; i++) {
        paths = paths * (n - 1 + i) / i;     // after this step paths == C(n - 1 + i, i): the division is exact
    }
    return (int) paths;
}
```

Ch 28 solves the same problem with DP. That's slower, but it adapts to obstacles, which no simple formula handles.

### Worked N4b *(custom)* · 10⁵ binomial queries modulo a prime

> Answer up to 10⁵ queries C(n, k) mod 10⁹ + 7, with n ≤ 10⁶.

`C(n, k) = n! · (k!)⁻¹ · ((n − k)!)⁻¹`, so precompute factorials and **inverse factorials** once. The inverses come from **Fermat's little theorem**: for a prime p and any b not divisible by p,

> b^(p−1) ≡ 1 (mod p), so **b⁻¹ ≡ b^(p−2)**.

*Why it's true, in two lines:* multiplying each of 1, 2, …, p − 1 by b gives the same numbers again mod p, just in a different order (if b·i ≡ b·j then i ≡ j, because p is prime). So the products are equal: b^(p−1) · (p − 1)! ≡ (p − 1)!. Divide both sides by (p − 1)!, which isn't a multiple of p.

Only *one* exponentiation is needed: compute `invFact[N]` with Fermat, then walk down using `1/(i − 1)! = i · (1/i!)`.

```java
final class Binomial {
    static final long MOD = 1_000_000_007L;
    private final long[] fact, invFact;

    Binomial(int maxN) {
        fact = new long[maxN + 1];
        invFact = new long[maxN + 1];
        fact[0] = 1;
        for (int i = 1; i <= maxN; i++) fact[i] = fact[i - 1] * i % MOD;
        invFact[maxN] = power(fact[maxN], MOD - 2, MOD);           // Fermat: x^(p - 2) = x^(-1) mod p
        for (int i = maxN; i > 0; i--) invFact[i - 1] = invFact[i] * i % MOD;
    }

    long choose(int n, int k) {
        if (k < 0 || k > n) return 0;
        return fact[n] * invFact[k] % MOD * invFact[n - k] % MOD;
    }
}
```

That's O(N + log p) to build and **O(1) per query**. **Fermat needs a prime modulus.** For a composite modulus, inverses exist only for values coprime to it, and they come from the *extended* Euclidean algorithm instead.

---

## 6. Inclusion–exclusion, plus binary search

### Worked N5 · [Nth Magical Number](https://leetcode.com/problems/nth-magical-number/)

> A positive integer is "magical" if it's divisible by a or by b. Return the n-th magical number modulo 10⁹ + 7 (n ≤ 10⁹, a, b ≤ 4·10⁴).

Generating magical numbers one by one takes up to 10⁹ steps. **Count instead.** How many magical numbers are ≤ x? Multiples of a, plus multiples of b, minus the ones counted twice, the multiples of both, which are the multiples of lcm(a, b):

> count(x) = ⌊x/a⌋ + ⌊x/b⌋ − ⌊x/lcm(a, b)⌋.

count(x) only grows as x grows, so the n-th magical number is **the first x with count(x) ≥ n**: Ch 10's template, searching on values with a counting predicate.

```java
public int nthMagicalNumber(int n, int a, int b) {
    long lcm = (long) a / gcd(a, b) * b;
    long lo = 0, hi = (long) n * Math.min(a, b);         // count(lo) = 0 < n; count(hi) >= n
    while (hi - lo > 1) {
        long mid = lo + (hi - lo) / 2;
        long count = mid / a + mid / b - mid / lcm;
        if (count >= n) hi = mid; else lo = mid;
    }
    return (int) (hi % 1_000_000_007L);
}
```

The answer can reach 4·10¹³, so everything is `long`, and the modulus is taken only at the very end. Ugly Number III (practice) is the same idea with three divisors, which needs all seven terms of inclusion–exclusion for three sets.

---

## 7. Games: optimal play

In a two-player game with perfect play:
* a position is **losing** for the player to move if *every* move leads to a winning position for the opponent;
* it's **winning** if *some* move leads to a losing position.

Computing that over all positions is DP (Ch 30 does it over intervals). Sometimes, though, a short argument replaces the whole computation.

### Worked N6 · [Stone Game](https://leetcode.com/problems/stone-game/)

> An even number of piles in a row, with an odd total. Players alternately take a whole pile from either end. Alice goes first. Does she win (end with more stones) under optimal play?

**A strategy argument.** Colour the piles alternately: positions 0, 2, 4, … white and 1, 3, 5, … black. Because the number of piles is even, the two ends of the full row have different colours. Alice can take a white end. That leaves both ends black, so Bob must take a black pile, which exposes a white end again. So Alice can collect **all the white piles**, or by the same argument all the black ones. One colour has a strictly larger sum (the total is odd, so they can't tie), and she picks that colour.

```java
public boolean stoneGame(int[] piles) {
    return true;     // Alice takes every pile of the colour with the larger sum
}
```

The lesson isn't the one-line answer. It's that **game problems sometimes have an invariant or strategy argument**, so look for one before writing the DP. When the conditions change (an odd number of piles, or "by how much does she win"), the DP of Ch 30 is needed. (For multi-pile take-away games, *Nim*, the first player wins exactly when the XOR of the pile sizes is non-zero. That's a recognition fact for Ch 37.)

---

## 8. Probability and expectation

* **Probability** of an event = favourable outcomes ÷ total outcomes, when all outcomes are equally likely.
* **Expected value** E[X] = Σ (value × its probability), the long-run average.
* **Linearity of expectation:** **E[X + Y] = E[X] + E[Y], always, even when X and Y depend on each other.** This is the most useful fact in the topic. Split a complicated count into a sum of simple yes/no **indicator** variables, find each one's probability, and add them up.
* **Repeated trials:** if each attempt succeeds with probability p, the expected number of attempts until the first success is 1/p.

### Worked N7 *(custom)* · Expected number of records

> In a uniformly random permutation of n distinct numbers, a **record** is an element larger than everything before it. What is the expected number of records?

Enumerating the n! permutations is hopeless. **Split it into indicators.** Let Xᵢ = 1 if position i is a record. The number of records is X₁ + … + Xₙ, so by linearity

> E[records] = Σ P(position i is a record).

Position i is a record exactly when it holds the **largest of the first i values**, and in a random permutation that largest value is equally likely to be at any of those i positions. So P = 1/i, and

> **E[records] = 1 + 1/2 + 1/3 + … + 1/n ≈ ln n.**

The Xᵢ are *not* independent, but linearity doesn't care. Two related problems use the same tools: Implement Rand10() Using Rand7() uses rejection sampling, with an expected number of rounds of 1/p; Random Pick with Weight uses prefix sums plus a binary search on a random number. Both are in the practice set.

```java
static double expectedRecords(int n) {
    double expected = 0;
    for (int i = 1; i <= n; i++) expected += 1.0 / i;    // P(position i is a record) = 1/i
    return expected;
}
```

---

### 8.1 Randomised algorithms you write yourself

Two short algorithms come up in phone screens, and both are "prove the probability, then code three lines".

**Fisher–Yates shuffle:** produce a uniformly random permutation of an array, where all n! orders are equally likely.

```java
static void shuffle(int[] a, Random rng) {
    for (int i = a.length - 1; i > 0; i--) {
        int j = rng.nextInt(i + 1);              // uniform in [0, i]: any remaining element can land at i
        int t = a[i]; a[i] = a[j]; a[j] = t;
    }
}
```

*Why it's uniform:* position n − 1 receives each of the n elements with probability 1/n. Then position n − 2 receives each of the remaining n − 1 with probability 1/(n − 1), and so on. Each specific permutation therefore has probability (1/n) · (1/(n−1)) · … · 1 = 1/n!. **The classic bug:** `j = rng.nextInt(n)` (any index, every time) gives nⁿ equally likely runs, and nⁿ isn't divisible by n! for n ≥ 3, so some orders come out more often than others.

**Reservoir sampling:** pick one element uniformly at random from a stream whose length you don't know in advance, using O(1) memory. **Keep the i-th element (counting from 1) with probability 1/i:**

```java
int chosen = 0, seen = 0;
for (int x : stream) {
    seen++;
    if (rng.nextInt(seen) == 0) chosen = x;      // probability 1/seen
}
```

*Why it's uniform:* the j-th element is chosen at step j with probability 1/j, and then survives every later step i with probability 1 − 1/i = (i − 1)/i. The product telescopes: (1/j) · (j/(j+1)) · ((j+1)/(j+2)) · … · ((n−1)/n) = **1/n**, the same for every j. To keep **k** elements, fill the reservoir with the first k, then replace a random slot with element i with probability k/i.

**Rejection sampling** (Rand10 from Rand7, in the practice set) is the third tool: generate a uniform value over a larger range, and retry if it falls outside a range that's a multiple of what you need.

## 9. Where these tools break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Division modulo a **composite** number | Fermat needs a prime modulus; the inverse may not even exist | Extended Euclid (inverses exist only for values coprime to the modulus) |
| Products of two numbers near 10¹⁸ | They overflow even `long` | `Math.multiplyHigh` or `BigInteger`, which interviews rarely need |
| C(n, k) for n around 10¹⁸ with a small prime | You can't build factorial tables | Lucas' theorem (recognition only) |
| A sieve up to 10⁹ | 1 GB of booleans | A segmented sieve (recognition only); or trial division for single numbers |
| Floating-point probabilities compared with `==` | Precision (Ch 4 §11) | Tolerances, or exact fractions modulo p |
| A game without a clean invariant | — | Minimax DP over states (Ch 30) |

---

## 10. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| Count the divisors of every number up to n | Harmonic sieve: each d adds 1 to all of its multiples, O(n log n) (CSES Counting Divisors, practice) |
| a^(b^c) modulo a prime | Fermat: reduce the exponent b^c modulo p − 1 (CSES Exponentiation II, practice) |
| Number of ways to walk exactly k steps and end at d | Choose how many steps go right: a binomial (practice) |
| Counting with constraints that interact | Combinatorics + DP (Ch 28) |
| Expected value with adaptive stopping | Probability DP (Ch 28 Knight Probability; stretch Soup Servings) |

---

## 11. Recognition

**Signals:**
* "**Modulo 10⁹ + 7**" → a counting problem; reduce after every step; division needs inverses.
* "**Divisible**", "multiples", "gcd / lcm", "**primes**" → Euclid, sieves, inclusion–exclusion.
* A **huge exponent** or a repeated operation applied 10¹⁸ times → fast exponentiation.
* "**Number of ways** to choose / arrange / distribute" with simple independent structure → product rule, binomials, stars and bars.
* "**Expected** value" or "probability" → linearity with indicators, or probability DP.
* "**Both players play optimally**" → look for a strategy or invariant argument, then minimax DP.
* "The **k-th** number with a property" where counting "how many ≤ x" is easy → binary search + inclusion–exclusion.

---

## 12. Implementation traps

| Trap | Fix |
| :--- | :--- |
| `a * b % MOD` with `int` operands | Make them `long` before multiplying |
| A negative result after subtraction | Add MOD before taking `%` |
| `lcm = a * b / gcd` overflowing | `a / gcd * b` |
| `i * i < n` overflowing in a sieve | `(long) i * i` |
| `Math.pow` for exact integer powers | Fast power with `long` |
| Taking the modulus too early inside a binary search | Search on true values; reduce only the final answer |
| Forgetting `C(n, k) = 0` when k > n or k < 0 | Guard it in `choose` |

---

## 13. Practice

1. **[Excel Sheet Column Title](https://leetcode.com/problems/excel-sheet-column-title/)** *(Easy)*
2. **[Greatest Common Divisor of Strings](https://leetcode.com/problems/greatest-common-divisor-of-strings/)** *(Easy)*
3. **[Factorial Trailing Zeroes](https://leetcode.com/problems/factorial-trailing-zeroes/)** *(Medium)*
4. **[Closest Prime Numbers in Range](https://leetcode.com/problems/closest-prime-numbers-in-range/)** *(Medium)*
5. **[Distinct Prime Factors of Product of Array](https://leetcode.com/problems/distinct-prime-factors-of-product-of-array/)** *(Medium)*
6. **[Random Pick with Weight](https://leetcode.com/problems/random-pick-with-weight/)** *(Medium)*
7. **[Implement Rand10() Using Rand7()](https://leetcode.com/problems/implement-rand10-using-rand7/)** *(Medium)*
8. **[Ugly Number III](https://leetcode.com/problems/ugly-number-iii/)** *(Medium)*
9. **[Number of Ways to Reach a Position After Exactly k Steps](https://leetcode.com/problems/number-of-ways-to-reach-a-position-after-exactly-k-steps/)** *(Medium)*
10. **[CSES Exponentiation II](https://cses.fi/problemset/task/1712)** *(Medium; stdin/stdout)*
11. **[CSES Counting Divisors](https://cses.fi/problemset/task/1713)** *(Medium; stdin/stdout)*
12. **[Count Anagrams](https://leetcode.com/problems/count-anagrams/)** *(Hard)*
13. **[Shuffle an Array](https://leetcode.com/problems/shuffle-an-array/)** *(Medium)*
14. **[Linked List Random Node](https://leetcode.com/problems/linked-list-random-node/)** *(Medium)*

**Stretch:** **[Poor Pigs](https://leetcode.com/problems/poor-pigs/)** *(Hard)* · **[Soup Servings](https://leetcode.com/problems/soup-servings/)** *(Medium)* · **[Count the Number of Ideal Arrays](https://leetcode.com/problems/count-the-number-of-ideal-arrays/)** *(Hard)*

<details><summary>Nudges</summary>

1. Excel: like base 26, but the digits run 1…26 with no zero. Subtract 1 before each division.
2. GCD of Strings: if `s + t != t + s`, there's no answer. Otherwise the answer is the prefix of length gcd(|s|, |t|).
3. Trailing Zeroes: each trailing zero needs a factor 10 = 2 · 5, and fives are the scarcer factor. Count n/5 + n/25 + n/125 + ….
4. Closest Primes: sieve up to `right`, then compare consecutive primes in the range.
5. Distinct Prime Factors: factorise each number (trial division, or an SPF sieve) into one set.
6. Random Pick: prefix sums of the weights; pick a random target in [1, total] and binary-search the first prefix ≥ target.
7. Rand10: two calls to rand7 give a uniform number in 1…49. Reject anything above 40 and map the rest to 1…10. What's the expected number of calls?
8. Ugly III: binary search with three-set inclusion–exclusion (pairwise lcms, and the lcm of all three). Use `long`.
9. Exactly k Steps: with r right-steps and k − r left-steps, the displacement is 2r − k. That fixes r, so the answer is C(k, r), or 0.
10. Exponentiation II: compute b^c mod (p − 1) first (Fermat), then a raised to that, mod p. Watch the case a ≡ 0.
11. Counting Divisors: a harmonic sieve over d up to 10⁶, then answer each query in O(1). Use fast I/O.
12. Count Anagrams: for each word, its multinomial (length! / ∏ letter counts!), using factorial and inverse-factorial tables; multiply over the words.
13. Shuffle an Array: keep a copy of the original for `reset()`; `shuffle()` is Fisher–Yates (§8.1) on a fresh copy.
14. Linked List Random Node: walk the list once with reservoir sampling (§8.1). The follow-up "the list is huge and its length unknown" is exactly why reservoir sampling exists.
13. Stretch, Poor Pigs: in T rounds, each pig has T + 1 distinguishable outcomes (it dies after round 1, 2, …, T, or survives). Find the smallest p with (T + 1)^p ≥ buckets.
14. Stretch, Soup Servings: a memoised probability recursion over (A remaining, B remaining) in units of 25 ml. For large n the answer is within 10⁻⁵ of 1, so return 1.
15. Stretch, Ideal Arrays: factorise each possible last value. Each prime's exponent e is spread over n positions by stars and bars, C(n + e − 1, e).
</details>

<details><summary>Key</summary>

Excel → base 26 with no zero digit · GCD of Strings → N2 on the lengths + a period argument · Trailing Zeroes → count factors of 5 · Closest Primes → N1 sieve · Distinct Prime Factors → N1 / trial division · Random Pick → prefix sums + binary search · Rand10 → rejection sampling, expected calls (N7) · Ugly III → N5 with three sets · Exactly k Steps → N4 binomial · Exponentiation II → Fermat: reduce the exponent mod p − 1 · Counting Divisors → harmonic sieve · Count Anagrams → multinomials with inverse factorials (N4b) · Shuffle an Array → Fisher–Yates (§8.1) · Linked List Random Node → reservoir sampling (§8.1) · Poor Pigs → counting distinguishable states · Soup Servings → probability recursion + a cutoff · Ideal Arrays → N1 + stars and bars
</details>

---

## 14. Self-test

<details><summary>1. Why may the sieve start crossing out multiples of p at p²?</summary>

Any smaller multiple p · k with k < p has a prime factor smaller than p, which has already crossed it out.
</details>

<details><summary>2. Why is Euclid's algorithm correct?</summary>

Every common divisor of a and b also divides a mod b = a − q·b, and conversely, so (a, b) and (b, a mod b) have the same common divisors, and hence the same gcd.
</details>

<details><summary>3. How do you divide by b modulo the prime 10⁹ + 7?</summary>

Multiply by b^(p − 2) mod p, which is b's inverse by Fermat's little theorem. b must not be a multiple of p.
</details>

<details><summary>4. Count the 5-letter strings over {a, b, c} that contain at least one 'a'.</summary>

The complement: 3⁵ − 2⁵ = 243 − 32 = 211.
</details>

<details><summary>5. Why does linearity of expectation work even for dependent indicator variables?</summary>

E[X + Y] = Σ (x + y) · P(x, y) = Σ x · P(x, y) + Σ y · P(x, y) = E[X] + E[Y]. The joint distribution sums out, and independence is never used.
</details>

---

## 15. Exit gate

Move on when you can:

* write the sieve, the SPF sieve, gcd/lcm and fast power from memory, with each one's correctness argument;
* build factorial and inverse-factorial tables and answer C(n, k) mod p in O(1), explaining Fermat;
* solve Nth Magical Number from a blank file and derive its count(x);
* use linearity of expectation on a fresh "expected number of …" question;
* solve 11 of the 14 practice problems, and prove the uniformity of Fisher–Yates and reservoir sampling.
