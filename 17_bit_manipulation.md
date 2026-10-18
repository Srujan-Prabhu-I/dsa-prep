# Chapter 17 — Bit Manipulation

> **Part III · Week 15** · Prerequisites: Ch 3 §5 (counting subsets), Ch 4 §12 (Java's shift rules), Ch 7, Ch 12 §4 (contribution)
> **Problems:** 6 worked (1 ★) · 8 practice · 3 stretch · **Time:** about 6 hours

An `int` is 32 yes/no switches. Bit manipulation means working on those switches directly. That gives you three things: **compact sets** (a set of up to 64 small items fits in one `long`, and union or intersection is one instruction), **algebraic tricks** (XOR cancels pairs; `x & −x` isolates the lowest set bit), and a powerful way of thinking, which is to **handle each bit position independently**. The last one is the important one, because many problems about sums, ANDs, ORs or XORs of numbers fall apart into 32 easy problems once you look at one bit at a time.

---

## 0. From zero: how integers are stored

### 0.1 Binary

A non-negative integer is a sum of powers of two. Bit i (counting from 0 at the right, the *least significant bit*) says whether 2ⁱ is included:

> 13 = 8 + 4 + 1 = `1101`₂: bit 3 = 1, bit 2 = 1, bit 1 = 0, bit 0 = 1.

```text
bit position:   7    6    5    4    3    2    1    0
place value:  128   64   32   16    8    4    2    1
13         =    0    0    0    0    1    1    0    1      8 + 4 + 1
```

**Converting to binary:** divide by 2 repeatedly and read the remainders from the bottom up (13 → 6 r1 → 3 r0 → 1 r1 → 0 r1, giving `1101`). In Java, `Integer.toBinaryString(13)` shows it.

An `int` has 32 bits (0–31); a `long` has 64.

### 0.2 Negative numbers: two's complement

Java stores negative numbers in **two's complement**: **−x = ~x + 1**, where `~x` flips every bit. Why that particular rule? Because x + ~x is all ones, which is −1 in this system, so x + (~x + 1) = 0, which is exactly what −x must satisfy. In 8 bits, for brevity (an `int` works the same with 32):

```text
 5        = 0000 0101
~5        = 1111 1010        flip every bit
~5 + 1    = 1111 1011        = −5
check:      0000 0101 + 1111 1011 = 1 0000 0000 → the 9th bit falls off, leaving 0000 0000 = 0  ✓
```

Two consequences:
* **Bit 31 is the sign bit.** It's 1 for every negative number.
* `int` ranges from −2³¹ to 2³¹ − 1, so **`Integer.MIN_VALUE` has no positive twin** (Ch 4 §9). Its negation is itself.

### 0.3 The operators

| Operator | Meaning, bit by bit | Example (`a = 1100`, `b = 1010`) |
| :--- | :--- | :--- |
| `a & b` | 1 if **both** are 1 | `1000` |
| `a \| b` | 1 if **either** is 1 | `1110` |
| `a ^ b` | 1 if they **differ** (XOR) | `0110` |
| `~a` | flip every bit | `…0011` |
| `a << k` | shift left k places = multiply by 2ᵏ (fills with 0) | `1100 << 1 = 11000` |
| `a >> k` | shift right, copying the sign bit = divide by 2ᵏ, rounding *down* | — |
| `a >>> k` | shift right, filling with 0 (treats the bits as unsigned) | — |

Java-specific rules from Ch 4 §12 still apply. Shift distances are taken mod 32 (`1 << 32 == 1`), so use `1L << k` for k ≥ 31. Comparisons bind tighter than `&`, so **parenthesise every bitwise expression**: `(x & 1) == 0`.

### 0.4 The idioms

| Goal | Code | Why it works |
| :--- | :--- | :--- |
| Is bit i set? | `((x >> i) & 1) == 1` | Move bit i to position 0, then mask |
| Set / clear / toggle bit i | `x \| (1 << i)` · `x & ~(1 << i)` · `x ^ (1 << i)` | — |
| **Lowest set bit** | `x & -x` | `−x = ~x + 1`: flipping makes the lowest 1 a 0 with 1s below it, and adding 1 carries back up to exactly that bit. Nothing else survives the `&` |
| **Clear the lowest set bit** | `x & (x - 1)` | `x − 1` turns the lowest 1 into 0 and the 0s below it into 1s |
| Is x a power of two? | `x > 0 && (x & (x - 1)) == 0` | A power of two has exactly one set bit |
| Count set bits | `Integer.bitCount(x)` | Or repeat `x &= x − 1` until 0 |

The two "lowest set bit" idioms on x = 12, in 8 bits:

```text
x        = 0000 1100          lowest set bit: bit 2 (value 4)
-x       = 1111 0100          ~x = 1111 0011, plus 1
x & -x   = 0000 0100  = 4     only the lowest set bit is 1 in both

x - 1    = 0000 1011          the lowest 1 became 0, the 0s below it became 1s
x & (x-1)= 0000 1000  = 8     the lowest set bit is gone
```

### 0.5 Masks as sets

A number can **represent a set** of items 0…n−1, with bit i set meaning item i is in the set. With items {A=0, B=1, C=2, D=3}:

```text
          D C B A
mask 11 = 1 0 1 1   = {A, B, D}
mask  6 = 0 1 1 0   = {B, C}
11 & 6  = 0 0 1 0   = {B}            intersection
11 | 6  = 1 1 1 1   = {A, B, C, D}   union
```

Then:

| Set operation | Bit operation |
| :--- | :--- |
| union, intersection | `a \| b`, `a & b` |
| difference a − b | `a & ~b` |
| is a ⊆ b? | `(a & b) == a` |
| add / remove item i | `a \| (1 << i)`, `a & ~(1 << i)` |
| every subset of n items | `for (int mask = 0; mask < (1 << n); mask++)` |
| every **sub**set of mask m | `for (int s = m; s > 0; s = (s - 1) & m)`, plus the empty set |

The submask loop works because `(s − 1) & m` is the next smaller number whose bits all lie inside m. For m = `1011` (11):

```text
s = 1011 (11)  →  s - 1 = 1010, & m = 1010 (10)  →  s - 1 = 1001, & m = 1001 (9)
  →  s - 1 = 1000, & m = 1000 (8)  →  0111 & m = 0011 (3)  →  0010 & m = 0010 (2)
  →  0001 & m = 0001 (1)  →  0000: stop          submasks: 11, 10, 9, 8, 3, 2, 1 (and 0)
```

Subtracting 1 borrows through the low bits, and `& m` throws away every bit outside m, so the loop steps through exactly the 2³ = 8 subsets of m's three bits, in decreasing order. Over all masks m, the submask loops run **3ⁿ** times in total (Ch 3 §5: each item is outside m, in m but not in s, or in both). Bitmask DP (Ch 32) is built on these loops.

**XOR algebra.** These three facts power half the tricks in this chapter:

> `x ^ x = 0`, `x ^ 0 = x`, and XOR is commutative and associative.

So XOR-ing a list **cancels every value that appears an even number of times**, in any order.

**Common misconceptions:**

| Belief | Reality |
| :--- | :--- |
| "`x >> 1` is the same as `x / 2`" | For negatives, `>>` rounds down (−7 >> 1 = −4) while `/` rounds toward zero (−7 / 2 = −3) |
| "`1 << 31` is 2³¹" | It's `Integer.MIN_VALUE` (negative). `1 << 32` is 1, not 0. Use `1L << k` |
| "`x & 1 == 0` tests for even" | It parses as `x & (1 == 0)`, a compile error. Write `(x & 1) == 0` |
| "`~x` is −x" | `~x = −x − 1` |
| "Bit tricks are always faster" | Clarity first; the JIT already optimises `x % 2` and `x * 2` |

---

## 1. Attempt first ★

> **[Single Number](https://leetcode.com/problems/single-number/) (10 minutes).** Every element appears twice except one. Find it in O(n) time and **O(1) extra space**.

---

## 2. XOR cancellation

### Worked X1 · [Single Number](https://leetcode.com/problems/single-number/)

XOR everything together. Each pair cancels to 0 (`x ^ x = 0`), order doesn't matter (associativity and commutativity), and 0 ^ single = single.

```java
public int singleNumber(int[] nums) {
    int result = 0;
    for (int x : nums) result ^= x;     // pairs cancel; what remains is the single element
    return result;
}
```

The same algebra finds the **Missing Number** (practice): XOR together all the indices 0…n and all the values, and every present value cancels with its index.

### Worked X2 · [Single Number III](https://leetcode.com/problems/single-number-iii/)

> Every element appears twice except **two**, `a` and `b`. Return both, in O(n) time and O(1) space.

XOR-ing everything gives `x = a ^ b`, which is not enough to separate them. But `a ≠ b`, so **x has at least one set bit**, a position where a and b differ. Take one, say the lowest: `x & −x`. Now **split the whole array by that bit**:
* `a` and `b` land in different groups, because they differ at that bit;
* both copies of every other value land in the *same* group, because they're equal.

So each group is a Single Number problem with exactly one survivor.

```java
public int[] singleNumber(int[] nums) {
    int xor = 0;
    for (int x : nums) xor ^= x;                 // = a ^ b, which is non-zero
    int bit = xor & -xor;                         // a position where a and b differ
    int a = 0, b = 0;
    for (int x : nums) {
        if ((x & bit) == 0) a ^= x;               // each group: pairs cancel, one single survives
        else b ^= x;
    }
    return new int[] {a, b};
}
```

When `xor == Integer.MIN_VALUE`, `xor & −xor` is still `MIN_VALUE` (bit 31), which is a valid distinguishing bit, so the code needs no special case.

---

## 3. Lowest-bit tricks for counting

### Worked X3 · [Counting Bits](https://leetcode.com/problems/counting-bits/)

> Return the number of 1 bits for every integer 0…n, in O(n) total.

Calling `bitCount` n times is O(n log n) in principle. Instead, **reuse a smaller answer**. Clearing the lowest set bit of i gives a smaller number with exactly one fewer 1:

```java
public int[] countBits(int n) {
    int[] bits = new int[n + 1];
    for (int i = 1; i <= n; i++) bits[i] = bits[i & (i - 1)] + 1;   // i & (i-1) < i: already computed
    return bits;
}
```

(`bits[i >> 1] + (i & 1)` works too: drop the last bit, then add it back.) This is your first small dynamic program: an answer built from a smaller, already-known answer. Ch 27 builds on it.

---

## 4. One bit at a time: per-bit contribution

Many problems combine numbers with XOR, AND or OR, all of which act on **each bit position independently**. So solve the problem for one bit position, then add the answers up, weighting bit b by 2ᵇ. This is Ch 12's contribution technique, applied to bit positions.

### Worked X4 · [Total Hamming Distance](https://leetcode.com/problems/total-hamming-distance/)

> The Hamming distance between two numbers is the number of bit positions where they differ. Return the sum over **all pairs** in the array. (n ≤ 10⁴.)

All pairs is O(32 · n²), about 3·10⁹. **One bit at a time:** at position b, suppose c numbers have that bit set and n − c don't. A pair differs at b exactly when it takes one number from each group, so position b contributes **c · (n − c)**.

```java
public int totalHammingDistance(int[] nums) {
    int n = nums.length, total = 0;
    for (int bit = 0; bit < 32; bit++) {
        int ones = 0;
        for (int x : nums) ones += (x >> bit) & 1;
        total += ones * (n - ones);              // pairs that differ at this bit; ≤ 32 · 2.5·10^7 fits in int
    }
    return total;
}
```

O(32 · n). Largest Combination With Bitwise AND Greater Than Zero and Single Number II (both in practice) are the same move: count per bit.

---

## 5. Greedy from the highest bit

**Why greedy works on bits:** bit b is worth 2ᵇ, which is **more than all lower bits combined** (2ᵇ⁻¹ + … + 1 = 2ᵇ − 1). So to maximise a number, make the highest bit 1 if at all possible, *whatever* that costs the lower bits. Then decide the next bit, and so on.

### Worked X5 · [Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/)

> Return the maximum of `a ^ b` over all pairs (values in `[0, 2³¹)`).

Go bit by bit from the top, keeping the best answer prefix found so far. At each bit, ask: **is there a pair whose XOR starts with (the answer so far, followed by 1)?** That's Two Sum with XOR in place of addition:

> `p ^ q = want` ⟺ `q = p ^ want`, so put every number's top bits (its *prefix*) in a set, and for each prefix p, look up `p ^ want`.

```java
public int findMaximumXOR(int[] nums) {
    int answer = 0;                                    // the best XOR's top bits decided so far
    Set<Integer> prefixes = new HashSet<>();
    for (int bit = 30; bit >= 0; bit--) {              // values are non-negative: bit 31 is always 0
        prefixes.clear();
        for (int x : nums) prefixes.add(x >> bit);     // each number's top (31 - bit) bits
        int want = (answer << 1) | 1;                  // try to make the next bit 1
        answer <<= 1;                                  // ...otherwise it's 0
        for (int p : prefixes) {
            if (prefixes.contains(p ^ want)) {         // some pair's prefixes XOR to want
                answer = want;
                break;
            }
        }
    }
    return answer;
}
```

**Why it's correct:** by the domination argument above, the maximum XOR's top bits must be the largest *achievable* prefix at every stage, and the set lookup tests achievability exactly. That's O(31 · n). Ch 20 gives a faster-in-practice version using a **binary trie**, which walks each number's bits down a tree, preferring the opposite bit at each step.

---

## 6. Common prefixes

### Worked X6 · [Bitwise AND of Numbers Range](https://leetcode.com/problems/bitwise-and-of-numbers-range/)

> Return `left & (left + 1) & … & right` (0 ≤ left ≤ right < 2³¹).

Looping over the whole range is up to 2³¹ steps. **Observation:** if `left < right`, the range contains two consecutive numbers, one even and one odd, so **bit 0 of the AND is 0**. Drop bit 0 from every number in the range, and you get exactly the range `[left >> 1, right >> 1]`. So

> AND(left..right) = AND((left >> 1)..(right >> 1)) << 1, whenever left < right.

Repeat until `left == right`. What remains is the **common binary prefix** of `left` and `right`, followed by zeros.

```java
public int rangeBitwiseAnd(int left, int right) {
    int shift = 0;
    while (left < right) {          // the range still contains an even and an odd number
        left >>= 1;
        right >>= 1;
        shift++;
    }
    return left << shift;           // the common prefix, followed by zeros
}
```

At most 31 iterations.

---

## 7. Where bit tricks break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| A set of more than 64 items | It won't fit in a `long` | `boolean[]`, or `java.util.BitSet` |
| Enumerating 2ⁿ masks for n > ~25 | 3·10⁷+ masks times the work per mask | Prune (Ch 6), meet in the middle (Ch 37), or a different idea |
| Negative numbers with `>>` | The sign bit is copied in, so loops "until 0" never end | `>>>`, or loop exactly 32 times |
| `1 << k` with k ≥ 31 | Overflow, or wrap-around of the shift distance | `1L << k` |
| "Every element appears k times except one" with k odd (k = 3) | XOR cancels only *pairs* | Count each bit mod k (Single Number II) |

---

## 8. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| Maximum XOR of a number with any element ≤ m, for many queries | Offline: sort the queries, insert into a binary trie as you go (Ch 20, stretch) |
| XOR of a subarray | Prefix XOR (Ch 8 §4) |
| Count pairs with XOR in a range | Binary trie with counts (Ch 20) |
| The minimum over subsets of n ≤ 20 items | Bitmask DP (Ch 32) |
| A set of small letters | A 26-bit mask: Maximum Product of Word Lengths (practice) |

---

## 9. Recognition

**Signals:**
* "Appears twice / an even number of times except …" → **XOR**.
* "Bitwise AND / OR / XOR" in the statement → **per-bit thinking**.
* "Maximum XOR" → **greedy from the high bit** (set lookup or trie).
* **Small sets** (≤ 20–25 items, or 26 letters) to compare, combine or enumerate → **masks**.
* "Without using + or −", or "power of two" → bit identities.

**False friends:** "the sum of all pairs' XORs", if you try it pair by pair. Per-bit counting turns n² into 32n.

---

## 10. Implementation traps

| Trap | Fix |
| :--- | :--- |
| `x & 1 == 0` (precedence) | `(x & 1) == 0` |
| `1 << 40` | `1L << 40` |
| `>>` on a negative number in a loop | `>>>`, or a fixed 32 iterations |
| Treating `x & -x` as broken for `MIN_VALUE` | It correctly gives bit 31 |
| Forgetting the empty set in the submask loop | Handle `s = 0` separately |
| `Integer.toBinaryString` for negatives | It prints all 32 bits, which is correct but long |

---

## 11. Practice

1. **[Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/)** *(Easy)*
2. **[Missing Number](https://leetcode.com/problems/missing-number/)** *(Easy)*
3. **[Reverse Bits](https://leetcode.com/problems/reverse-bits/)** *(Easy)*
4. **[Single Number II](https://leetcode.com/problems/single-number-ii/)** *(Medium)*
5. **[Minimum Flips to Make a OR b Equal to c](https://leetcode.com/problems/minimum-flips-to-make-a-or-b-equal-to-c/)** *(Medium)*
6. **[Maximum Product of Word Lengths](https://leetcode.com/problems/maximum-product-of-word-lengths/)** *(Medium)*
7. **[Gray Code](https://leetcode.com/problems/gray-code/)** *(Medium)*
8. **[Largest Combination With Bitwise AND Greater Than Zero](https://leetcode.com/problems/largest-combination-with-bitwise-and-greater-than-zero/)** *(Medium)*

**Stretch:** **[Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/)** *(Medium)* · **[Decode XORed Permutation](https://leetcode.com/problems/decode-xored-permutation/)** *(Medium)* · **[Bitwise ORs of Subarrays](https://leetcode.com/problems/bitwise-ors-of-subarrays/)** *(Medium)*

<details><summary>Nudges</summary>

1. 1 Bits: `x &= x − 1` until 0, counting. (The input is treated as unsigned; that's fine.)
2. Missing Number: XOR all the indices 0…n with all the values.
3. Reverse Bits: 32 times, shift the result left and add the input's lowest bit; shift the input right with `>>>`.
4. Single II: for each bit, count the 1s mod 3. The leftover bits form the answer. (Bit 31 included: the result can be negative.)
5. Min Flips: per bit. If c's bit is 1, you need a|b's bit to be 1 (1 flip if both a and b are 0). If it's 0, flip every 1 bit in a and b.
6. Word Lengths: a 26-bit mask per word. Two words share no letters ⟺ `(m1 & m2) == 0`.
7. Gray Code: `i ^ (i >> 1)`. Why do consecutive values differ in exactly one bit?
8. AND Combination: a combination has a non-zero AND ⟺ every member shares some bit. So the answer is the largest per-bit count of 1s.
9. Stretch, Sum: `a ^ b` is the sum without carries and `(a & b) << 1` is the carries. Repeat until there are no carries.
10. Stretch, Decode XORed: the XOR of 1…n is known. XOR-ing every second encoded value gives the XOR of all elements except the first.
11. Stretch, ORs of Subarrays: the set of OR values of subarrays ending at i has at most 32 distinct values (each one adds bits). Build it from the set for i − 1.
</details>

<details><summary>Key</summary>

1 Bits → X3 (clear the lowest bit) · Missing → X1 · Reverse Bits → unsigned shifts (`>>>`) · Single II → X4 (per-bit count mod 3) · Min Flips → per bit · Word Lengths → letter masks (§0.5) · Gray Code → `i ^ (i >> 1)`; consecutive values differ in one bit · AND Combination → X4 (count per bit) · Sum of Two Integers → carry loop · Decode XORed → X1 with the XOR of 1..n · ORs of Subarrays → the set of ORs ending at i has ≤ 32 values
</details>

---

## 12. Self-test

<details><summary>1. Why does x & −x isolate the lowest set bit?</summary>

−x = ~x + 1. Flipping makes the lowest 1 a 0 with all 1s below it; adding 1 carries back up to exactly that position. Above it, x and −x are complements, so `&` clears everything except that bit.
</details>

<details><summary>2. Why does XOR-ing an array find the single unpaired element?</summary>

XOR is associative and commutative, so pairs can be grouped together, and each pair gives x ^ x = 0. What's left is 0 ^ single = single.
</details>

<details><summary>3. In Single Number III, why are a and b guaranteed to fall into different groups?</summary>

The splitting bit is set in a ^ b, so a and b differ at that position.
</details>

<details><summary>4. Why is it safe to decide the maximum XOR greedily from the highest bit?</summary>

Bit b is worth 2ᵇ, which exceeds the sum of all lower bits (2ᵇ − 1). So a 1 at bit b beats every possible combination below it.
</details>

<details><summary>5. How many iterations do the submask loops make in total over all masks of n bits?</summary>

3ⁿ: each element is outside the mask, inside the mask but outside the submask, or inside both.
</details>

---

## 13. Exit gate

Move on when you can:

* convert numbers to binary and two's complement by hand, and explain why −x = ~x + 1;
* write every idiom in §0.4 and §0.5 from memory, with the reason for `x & −x` and `x & (x − 1)`;
* solve Single Number III and Total Hamming Distance from blank files and explain the split / per-bit argument;
* prove the greedy-from-the-high-bit rule;
* solve 7 of the 8 practice problems.
