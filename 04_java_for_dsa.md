# Chapter 4 — Java for DSA: Essentials, Semantics & Collections

> **Part I · Weeks 1–2** (Part A in Week 1; Parts B and C in Week 2) · Prerequisites: basic Java (variables, loops, `if`, methods, classes)
> **Problems:** 3 worked · 25-item predict-the-output lab · 5 practice · 2 stretch · **Time:** about 8 hours

This chapter covers **every Java feature you need for DSA, and nothing you don't.** It also covers the exact places where Java *silently* gives a wrong answer: no exception, no warning, just a wrong number that fails one hidden test case out of fifty. Those bugs cost more OA points than wrong algorithms do. The goal is that, from Chapter 5 onwards, every mistake you make is a mistake of reasoning, never of language.

* **Part A — Essentials:** program shapes, input/output, types, arrays, strings, methods, generics.
* **Part B — Where Java goes wrong:** overflow, division, floating point, bits, equality, comparators, hash keys, recursion depth.
* **Part C — The collections contract:** what each collection costs, and the idioms you'll use in every chapter.

A note on versions: LeetCode runs a recent JDK. Some OA platforms still run Java 8, 11 or 17. The table in §26 lists the features to check before you rely on them.

---

# Part A — Essentials

## 1. Two program shapes

**LeetCode / interview shape.** You fill in a method of `class Solution`. Input arrives as parameters, and you return the answer. No `main`, no I/O.

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        // ...
        return new int[] {0, 1};
    }
}
```

**OA / CSES / HackerRank shape.** You write a complete program that reads **standard input** and writes **standard output**. The class is usually called `Main`, but some platforms require `Solution`. **Read the stub the platform gives you and keep its class name.**

```java
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        // read input, solve, print output
    }
}
```

`import java.util.*;` brings in every collection; `import java.io.*;` brings in readers and writers. LeetCode imports common packages for you, but writing the imports yourself never hurts.

## 2. Reading input

### 2.1 `Scanner`: simple, and slow

```java
Scanner sc = new Scanner(System.in);
int n = sc.nextInt();
long big = sc.nextLong();
String word = sc.next();        // one whitespace-separated token
String line = sc.nextLine();    // the rest of the current line
```

`Scanner` parses with regular expressions, so it's roughly 10× slower than the alternative. Up to about 10⁵ numbers it's fine; at 10⁶ it can time out on its own, before your algorithm even starts.

> **The `nextInt()` + `nextLine()` trap.** `nextInt()` reads the number but leaves the newline after it. The next `nextLine()` returns that leftover empty string. If you must mix them, call `sc.nextLine()` once to throw away the rest of the line.

### 2.2 `FastReader`: use this for every OA

Memorise this class. You'll rebuild it from a blank file as an exit-gate item.

```java
static final class FastReader {
    private final BufferedReader reader;
    private StringTokenizer tokens;

    FastReader(InputStream stream) {
        reader = new BufferedReader(new InputStreamReader(stream), 1 << 16);
    }

    /** Next whitespace-separated token, or null at end of input. */
    String next() throws IOException {
        while (tokens == null || !tokens.hasMoreTokens()) {
            String line = reader.readLine();
            if (line == null) return null;          // end of input
            tokens = new StringTokenizer(line);
        }
        return tokens.nextToken();
    }

    int nextInt() throws IOException { return Integer.parseInt(next()); }
    long nextLong() throws IOException { return Long.parseLong(next()); }
    double nextDouble() throws IOException { return Double.parseDouble(next()); }

    /** A whole raw line, spaces included. Discards any unread tokens on the current line. */
    String nextLine() throws IOException {
        tokens = null;
        return reader.readLine();
    }
}
```

It reads a line at a time and splits it into tokens, so it doesn't care whether numbers sit on one line or many. It also has **no** leftover-newline trap: after `nextInt()` reads the last number on a line, `nextLine()` reads the *next* line, as you'd expect.

### 2.3 The input formats you'll meet

```java
FastReader in = new FastReader(System.in);

// (a) n, then n numbers (on one line or several: it doesn't matter)
int n = in.nextInt();
int[] a = new int[n];
for (int i = 0; i < n; i++) a[i] = in.nextInt();

// (b) T test cases
int t = in.nextInt();
while (t-- > 0) {
    int m = in.nextInt();
    // read and solve one test
}

// (c) a matrix of R rows and C columns
int rows = in.nextInt(), cols = in.nextInt();
int[][] grid = new int[rows][cols];
for (int r = 0; r < rows; r++)
    for (int c = 0; c < cols; c++)
        grid[r][c] = in.nextInt();

// (d) a character grid, each row given as one word like "#..#."
char[][] board = new char[rows][];
for (int r = 0; r < rows; r++) board[r] = in.next().toCharArray();

// (e) a line that contains spaces (a sentence, a name)
String sentence = in.nextLine();

// (f) read numbers until the input ends
String token;
while ((token = in.next()) != null) {
    int x = Integer.parseInt(token);
    // ...
}

// (g) an edge list with 1-indexed vertices: convert to 0-indexed immediately
int vertices = in.nextInt(), edges = in.nextInt();
for (int i = 0; i < edges; i++) {
    int u = in.nextInt() - 1, v = in.nextInt() - 1;
    // add edge u-v
}

// (h) an array written as "[1, 2, 3]" on one line (some company OAs)
String raw = in.nextLine().trim();
String inside = raw.substring(1, raw.length() - 1).trim();     // "1, 2, 3"
int[] parsed = inside.isEmpty() ? new int[0]
        : Arrays.stream(inside.split(",")).map(String::trim).mapToInt(Integer::parseInt).toArray();
```

## 3. Writing output

`System.out.println` flushes often, and printing 10⁵ lines with it can cost a full second. Buffer instead: either collect output in a `StringBuilder` and print it once, or use a `PrintWriter`:

```java
PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(System.out)));
out.println(answer);
out.print(x); out.print(' '); out.println(y);
out.printf("%.6f%n", value);     // fixed decimals for real-valued answers
out.flush();                     // FORGET THIS AND NOTHING IS PRINTED
```

Printing an array the way judges expect (space-separated, not `[1, 2, 3]`):

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < a.length; i++) {
    if (i > 0) sb.append(' ');
    sb.append(a[i]);
}
out.println(sb);
```

`Arrays.toString(a)` prints `[1, 2, 3]`. That's useful for debugging and almost never the required output format.

### 3.1 The OA skeleton: rebuild this from memory

```java
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        FastReader in = new FastReader(System.in);
        PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(System.out)));

        int t = in.nextInt();
        while (t-- > 0) {
            int n = in.nextInt();
            long[] a = new long[n];
            for (int i = 0; i < n; i++) a[i] = in.nextLong();
            out.println(solve(a));
        }

        out.flush();
    }

    static long solve(long[] a) {
        long sum = 0;
        for (long x : a) sum += x;
        return sum;
    }

    // FastReader from §2.2 goes here, as a static nested class.
}
```

Keep `solve` separate from the I/O. It lets you test `solve` directly, and that is exactly what the stress harness in Ch 5 does.

### Worked J0 · [CSES Weird Algorithm](https://cses.fi/problemset/task/1068)

> Read `n` (`1 ≤ n ≤ 10⁶`). Repeat: if `n` is even, halve it; otherwise replace it with `3n + 1`. Print every value, including the first and the final `1`, on one line.

Two Java decisions decide whether this passes:

1. **The type.** The values climb far above `n`. Starting below 10⁶, the sequence peaks at about 5.7·10¹⁰, well past `int`'s 2.1·10⁹. `3 * n + 1` in `int` silently wraps negative, and the program then never reaches 1. **Use `long`.**
2. **The output.** The sequence can have hundreds of terms; build it in a `StringBuilder` and print once.

```java
import java.io.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
        long n = Long.parseLong(in.readLine().trim());
        StringBuilder out = new StringBuilder();
        out.append(n);
        while (n != 1) {
            n = (n % 2 == 0) ? n / 2 : 3 * n + 1;    // long arithmetic: no overflow below ~3·10^18
            out.append(' ').append(n);
        }
        System.out.println(out);
    }
}
```

## 4. Primitive types, literals and casts

| Type | Bits | Range | Use it for |
| :--- | :-: | :--- | :--- |
| `int` | 32 | −2³¹ … 2³¹−1 ≈ **±2.1·10⁹** | Default for indices, counts and values ≤ 10⁹ |
| `long` | 64 | −2⁶³ … 2⁶³−1 ≈ **±9.2·10¹⁸** | Sums, products, anything that can exceed 2·10⁹ |
| `double` | 64 | ~15–16 significant digits | Real-valued answers only (see §11) |
| `char` | 16 | 0 … 65 535 (unsigned) | Characters; it's an integer type underneath |
| `boolean` | — | `true` / `false` | Flags, visited arrays |
| `byte`, `short`, `float` | 8, 16, 32 | — | Almost never. (`byte[]` occasionally, to save memory.) |

**Facts to know cold:** `Integer.MAX_VALUE = 2 147 483 647`. 10⁹ · 10⁹ = 10¹⁸ fits in `long`; 10¹⁰ · 10⁹ does not. `Long.MAX_VALUE ≈ 9.22·10¹⁸`.

**Literals.** `1_000_000_007` (underscores are allowed in numbers). `5L` is a `long`. `1e9` is a **`double`**, so `int x = 1e9;` doesn't compile; write `(int) 1e9` or `1_000_000_000`. `'a'` is a `char`; `"a"` is a `String`.

**Casts.**
* Widening (`int → long → double`) happens automatically.
* Narrowing needs a cast: `(int) someLong` keeps only the **low 32 bits**, with no error.
* `(int) 3.99` is `3`, and `(int) −3.99` is `−3`: casting a `double` truncates **toward zero**.
* `(int) 1e10` saturates to `Integer.MAX_VALUE`.

**`char` arithmetic.** A `char` used in arithmetic becomes an `int`:

```java
char c = 'd';
int index = c - 'a';                 // 3: the letter's position, for int[26] tables
char next = (char) ('a' + 4);        // 'e': the cast back is required
int digit = '7' - '0';               // 7
char asDigit = (char) ('0' + 5);     // '5'
boolean isLower = c >= 'a' && c <= 'z';     // or Character.isLowerCase(c)
```

## 5. Arrays and `java.util.Arrays`

```java
int[] a = new int[n];                 // all 0; boolean[] → false; Object[] → null
int[] b = {3, 1, 2};
int len = a.length;                   // a FIELD on arrays; s.length() on Strings; list.size() on lists
int[][] grid = new int[rows][cols];   // rectangular
int[][] jagged = new int[rows][];     // rows assigned later, each with its own length
jagged[0] = new int[5];
```

| Call | Does | Note |
| :--- | :--- | :--- |
| `Arrays.fill(a, -1)` | Fill a 1D array | For 2D: `for (int[] row : grid) Arrays.fill(row, -1);` |
| `Arrays.sort(a)` | Sort ascending | `Arrays.sort(a, from, to)` sorts the half-open range `[from, to)` |
| `a.clone()`, `Arrays.copyOf(a, len)` | Copy | For 2D arrays, `clone()` is **shallow**: the rows are shared |
| `Arrays.copyOfRange(a, from, to)` | Copy a sub-range | Half-open `[from, to)` |
| `Arrays.equals(a, b)` | Compare contents | `a.equals(b)` compares *references* |
| `Arrays.toString(a)`, `Arrays.deepToString(g)` | Debug printing | — |
| `Arrays.stream(a).sum()` | Sum | Returns an **`int`** for `int[]`, so it can overflow. Use `Arrays.stream(a).asLongStream().sum()` |
| `Arrays.asList(...)` | Fixed-size list view | `Arrays.asList(intArray)` gives a `List<int[]>` of size **1**, not a list of numbers |

Converting between `int[]` and `List<Integer>`:

```java
List<Integer> list = new ArrayList<>();
for (int x : a) list.add(x);                                  // int[] → List
int[] back = list.stream().mapToInt(Integer::intValue).toArray();   // List → int[]
```

## 6. Strings, `StringBuilder` and `Character`

A `String` is **immutable**: every "modification" creates a new string.

| Call | Cost | Note |
| :--- | :-: | :--- |
| `s.length()`, `s.charAt(i)` | O(1) | — |
| `s.substring(b, e)` | O(e − b) | Half-open `[b, e)`; it **copies** the characters |
| `s.equals(t)`, `s.compareTo(t)` | O(length) | Never compare strings with `==` (§13) |
| `s.indexOf(x)`, `s.contains(x)`, `s.startsWith(p)` | O(n) or more | Avoid inside loops |
| `s.toCharArray()` | O(n) | Gives a mutable copy |
| `String.valueOf(chars)`, `new String(chars)` | O(n) | `char[]` → `String` |
| `String.valueOf(42)`, `Integer.parseInt("42")` | — | number ↔ text |
| `s.split(regex)` | O(n) | The argument is a **regular expression**: `split(".")` returns an empty array; use `split("\\.")`. Leading separators produce an empty first token. Trim first, or `split("\\s+")` |
| `String.join(",", parts)`, `s.trim()`, `s.repeat(k)` | — | `repeat` needs Java 11+ |

**Never build a string with `+=` in a loop.** Each `+=` copies the whole string so far, so n appends cost O(n²). Use `StringBuilder`:

```java
StringBuilder sb = new StringBuilder();
sb.append("x").append(42).append(',');      // amortised O(1) per append
sb.setCharAt(0, 'y');                        // O(1)
sb.deleteCharAt(sb.length() - 1);            // O(1) at the end; O(n) elsewhere
sb.insert(0, 'z');                           // O(n): shifts everything
sb.reverse();                                // O(n)
String result = sb.toString();
```

**Common string idioms:**

```java
// sort the characters of a string
char[] chars = s.toCharArray();
Arrays.sort(chars);
String sorted = new String(chars);

// frequency of lowercase letters (much faster than Map<Character, Integer>)
int[] freq = new int[26];
for (int i = 0; i < s.length(); i++) freq[s.charAt(i) - 'a']++;

// classify characters
Character.isDigit(c); Character.isLetter(c); Character.isLetterOrDigit(c);
Character.isUpperCase(c); Character.toLowerCase(c);
```

## 7. Methods, references and state

**Java passes everything by value, but for objects and arrays the value *is a reference*.** A method can change the *contents* of an array you pass it, and the caller sees the change. If the method *reassigns* its parameter, the caller sees nothing:

```java
static void touch(int[] arr) {
    arr[0] = 99;          // visible to the caller: same array
    arr = new int[5];     // invisible to the caller: only the local copy of the reference changes
}
```

**Returning several values.** Return a small array (`new int[] {lo, hi}`), or a **record** (§8). Avoid out-parameters.

**Helper state.** For recursive helpers, either pass state as parameters or keep it in **instance fields** of `Solution`.

> **LeetCode trap:** LeetCode runs all test cases in one JVM. Instance fields are fresh for each test, because a new `Solution` is created each time. **`static` fields keep their values between tests.** A `static` counter or memo table that you never reset produces a wrong answer on test 2 even though test 1 passes. Don't use `static` mutable state in `Solution`.

**Nested helper classes** go inside your class as `static class Node { … }`, or inside `Solution` on LeetCode.

## 8. Generics, wrappers, lambdas and records

**Generics: what the angle brackets mean.** `List<Integer>` reads "a list whose elements are `Integer`s". The type in brackets is a **type parameter**: the same `ArrayList` class works for any element type, and the compiler checks that you only put in (and take out) that type. `Map<String, List<Integer>>` maps each `String` key to a list of `Integer`s. On the right-hand side, `new ArrayList<>()` uses the **diamond** `<>`, and Java fills in the type from the left-hand side. You'll rarely write your own generic classes; you'll *use* them in every chapter.

**Collections hold objects, not primitives.** `List<Integer>`, not `List<int>`. Java converts between `int` and `Integer` automatically (*autoboxing*), but it isn't free:
* Each `Integer` is a separate heap object of about 16 bytes, plus a reference. An `ArrayList<Integer>` of 10⁶ elements uses about 20 MB; an `int[]` uses 4 MB.
* Unboxing a `null` throws `NullPointerException`: `int x = map.get(key);` crashes when the key is missing.

**Lambdas and method references** are how you pass comparators and functions:

```java
list.sort((x, y) -> Integer.compare(x, y));
map.merge(key, 1, Integer::sum);
groups.computeIfAbsent(k, unused -> new ArrayList<>());
```

**Streams, for reading only.** A few lines in this book use a *stream pipeline*, such as `Arrays.stream(a).distinct().sorted().toArray()`. Read it left to right as a chain of steps over the elements: turn the array into a stream, drop the duplicates, sort, and collect the result back into an array. Each pipeline is equivalent to a short loop, and the loop is usually faster in Java, so the book uses streams only in brute forces and one-off conversions. You never need to write one.

**Iterators.** Every collection can hand you an `Iterator<E>`, with `hasNext()` (is there another element?) and `next()` (return it and advance). The for-each loop uses one behind the scenes. Some design problems ask you to *implement* `Iterator<Integer>`: write a class with those two methods, and keep enough state to know what comes next (Ch 16 practice).

**`var`** (Java 10+) infers a local variable's type: `var map = new HashMap<String, List<Integer>>();`. Use it when the type is long and obvious.

**Records** (Java 16+) are immutable data classes with `equals`, `hashCode` and `toString` generated for you. That makes them perfect map keys and return values:

```java
record Cell(int row, int col) {}

Set<Cell> visited = new HashSet<>();
visited.add(new Cell(2, 3));
boolean seen = visited.contains(new Cell(2, 3));   // true: compared by value
```

---

# Part B — Where Java silently goes wrong

## 9. Integer overflow

`int` and `long` arithmetic **wraps around** on overflow, with no exception. `Integer.MAX_VALUE + 1 == Integer.MIN_VALUE`.

**The classic bug: the overflow happens before the widening.**

```java
int a = 1_000_000, b = 1_000_000;
long wrong = a * b;          // -727379968: the multiplication happens in int, THEN it's widened
long right = (long) a * b;   // 1000000000000: one operand is long, so the multiplication is long
```

Rules that prevent nearly every overflow bug:

1. **Before writing any sum or product, bound it.** n · max|value| > 2·10⁹ means the accumulator must be `long`. Ten seconds of arithmetic.
2. **Cast before the operation, not after:** `(long) a * b`, never `(long) (a * b)`.
3. **Middle of a range:** `lo + (hi - lo) / 2`, because `(lo + hi) / 2` can overflow when both are large.
4. **Sentinels:** use "infinity" values that survive addition. `Integer.MAX_VALUE / 2` or `1_000_000_000` for `int`, `Long.MAX_VALUE / 4` for `long`. `Integer.MAX_VALUE + edgeWeight` wraps to a large negative number and corrupts a shortest-path computation.
5. **`Integer.MIN_VALUE` has no positive twin.** `Math.abs(Integer.MIN_VALUE)` and `-Integer.MIN_VALUE` both return `Integer.MIN_VALUE` (still negative). `Integer.MIN_VALUE / -1` is also `Integer.MIN_VALUE`, with no exception. Widen to `long` before negating any value that could be `MIN_VALUE`.
6. **Products that might overflow even `long`:** `Math.multiplyExact(a, b)` and `Math.addExact(a, b)` throw `ArithmeticException` instead of wrapping. They're useful in a stress test to confirm your bounds. For a guard like "is `a * b > limit`?" with positive values, test `a > limit / b` instead.

## 10. Division, remainder and modular arithmetic

Java's integer division **truncates toward zero**, and `%` takes the **sign of the dividend**:

| Expression | Result | Floor version |
| :--- | :-: | :--- |
| `7 / 2`, `7 % 2` | `3`, `1` | — |
| `-7 / 2`, `-7 % 2` | `-3`, `-1` | `Math.floorDiv(-7, 2) = -4`, `Math.floorMod(-7, 2) = 1` |
| `7 % -2` | `1` | — |

**Consequences:**
* `x % 2 == 1` is **false for negative odd `x`** (it's `-1`). Test oddness with `x % 2 != 0`, or `(x & 1) == 1`.
* A remainder used as an array index, or a hash bucket, must be made non-negative: `Math.floorMod(x, m)` or `((x % m) + m) % m`.

**Ceiling division** for `a ≥ 0, b > 0`: `(a + b - 1) / b` (this can overflow when `a` is near the type's max), or the overflow-free `a / b + (a % b == 0 ? 0 : 1)`. Java 18+ has `Math.ceilDiv(a, b)`, which also handles negatives.

**Modular arithmetic** (counting problems that say "modulo 10⁹ + 7"):

```java
static final int MOD = 1_000_000_007;

long sum  = (a + b) % MOD;                  // a, b already in [0, MOD)
long diff = ((a - b) % MOD + MOD) % MOD;     // subtraction can go negative: add MOD back
long prod = (a * b) % MOD;                   // a, b must be long: the product reaches ~10^18
```

Reduce after **every** operation, not just at the end. Ch 18 adds fast exponentiation and modular division.

## 11. Floating point

A `double` has about 15–16 significant decimal digits and can't represent most decimals exactly. `0.1 + 0.2 == 0.3` is **false**.

* **Never compare doubles with `==`.** Use `Math.abs(x - y) < 1e-9`. For large magnitudes, use a relative tolerance.
* **Avoid doubles when integers will do.** To compare fractions `a/b < c/d` with `b, d > 0`, compare `(long) a * d < (long) c * b`. For slopes, store the reduced pair `(dy / g, dx / g)` (Ch 7, Max Points on a Line).
* **`Math.pow` returns a `double`.** Don't use it for integer powers like `Math.pow(10, 18)`, because large results lose precision. Multiply in a loop, or use fast power (Ch 18).
* **Integer square root:** `Math.sqrt` can be off by one for large inputs. Correct it:

  ```java
  long r = (long) Math.sqrt((double) x);          // x up to ~10^18
  while (r * r > x) r--;
  while ((r + 1) * (r + 1) <= x) r++;
  ```
* `Math.floor` and `Math.ceil` return `double`; `Math.round` returns `long` and rounds half up (`Math.round(-2.5) == -2`).

## 12. Bits and shifts: the Java-specific parts

Ch 17 covers bit manipulation properly. These are the parts where Java itself surprises people:

* **Shift distances are taken mod 32 for `int`** (mod 64 for `long`). `1 << 32 == 1`, and `1 << 31` is negative. For masks wider than 30 bits, write `1L << k`.
* **`>>` copies the sign bit; `>>>` shifts in zeros.** For negative numbers, `-8 >> 1 == -4`, but `-8 >>> 1` is a large positive number.
* **Precedence:** `+` and `-` bind tighter than `<<`, so `1 << n - 1` means `1 << (n - 1)`. Comparisons bind tighter than `&`, so `x & 1 == 0` doesn't even compile. **Parenthesise every bitwise expression:** `(x & 1) == 0`.
* Useful built-ins: `Integer.bitCount(x)`, `Integer.numberOfTrailingZeros(x)`, `Integer.highestOneBit(x)`, `Integer.toBinaryString(x)`, plus the `Long` versions.

## 13. Equality and boxing

`==` on objects compares **references** (is it the same object?), not values.

* **The `Integer` cache.** Java reuses `Integer` objects for −128…127, so `Integer a = 127, b = 127; a == b` is `true`, but with 128 it's `false`. Code that compares boxed values with `==` passes small tests and fails large ones. **The classic:** `map.get(x) == map.get(y)` where the values are counts over 127.
* Compare boxed values with `.equals`, or unbox first: `map.get(x).intValue() == map.get(y)`, or `Objects.equals(p, q)` (null-safe).
* **Strings:** `s == t` can appear to work, because string *literals* are shared. It fails as soon as either string is computed (`substring`, `StringBuilder.toString()`, input). Always use `s.equals(t)`.
* **`Long` versus `Integer` keys.** In a `Map<Long, V>`, `map.containsKey(1)` is **false**: the literal `1` boxes to `Integer`, and `Integer.equals(Long)` is false. Write `1L`, or cast: `containsKey((long) x)`.

## 14. Comparators

A comparator returns a negative number, zero or a positive number. It must be **consistent**: never both `a < b` and `b < a`, it must be transitive, and `compare(a, a) == 0`. If it isn't, sorting can produce garbage or throw *"Comparison method violates its general contract!"*

**Never compare by subtracting.** `(a, b) -> a - b` overflows when `a` and `b` have opposite signs and large magnitudes, which reverses the order. Always use:

```java
(a, b) -> Integer.compare(a, b)        // or Long.compare, Double.compare, a.compareTo(b)
```

**Building comparators:**

```java
// people: int[] {age, height}; sort by age ascending, then height descending
Arrays.sort(people, (p, q) -> p[0] != q[0] ? Integer.compare(p[0], q[0])
                                           : Integer.compare(q[1], p[1]));

// the same with combinators. The explicit (int[] p) type is REQUIRED when you chain
// .then...: without a target type, Java infers p as Object and the chain fails to compile.
Arrays.sort(people, Comparator.comparingInt((int[] p) -> p[0])
                              .thenComparing((int[] p) -> p[1], Comparator.reverseOrder()));

// strings by length, then alphabetically
words.sort(Comparator.comparingInt(String::length).thenComparing(Comparator.naturalOrder()));
```

`.reversed()` applies to the **whole chain** built so far, not just the last key, which is a common surprise. When in doubt, write the lambda out in full.

**What can be sorted with a comparator:**
* `int[]` **cannot**; `Arrays.sort(int[], comparator)` doesn't exist.
* For descending `int[]`, sort ascending and reverse it in place. Or use `Integer[]`, or `int[][]`, both of which accept comparators.
* Sorting objects (`Integer[]`, `int[][]`, lists) is **stable** (equal elements keep their order); sorting primitives isn't, but equal primitives are indistinguishable anyway.

> **A footnote on primitive sorts.** `Arrays.sort` on any primitive array (`int[]`, `long[]`, …) is a dual-pivot quicksort. On Java 8 and 11, crafted inputs can drive it to O(n²); since JDK 14 it falls back to heapsort when the recursion gets too deep, so its worst case is O(n log n). This matters only on judges where other contestants can "hack" your solution with such inputs (Codeforces); interview and OA platforms don't do this. On an old JDK, if you ever need protection, shuffle the array first, or sort an `Integer[]` copy, which uses TimSort (O(n log n) worst case, but slower because of boxing). Sorting a `long[]` copy does **not** help: it's the same quicksort.

## 15. Hash keys

`HashMap` and `HashSet` find keys using `hashCode()` and `equals()`. So a key must have *value* semantics, and it must **not change while it's in the map**.

| Key type | Works? | Notes |
| :--- | :-: | :--- |
| `Integer`, `Long`, `String`, `Character` | ✓ | Standard |
| `record Cell(int r, int c)` | ✓ | Best choice for composite keys (Java 16+) |
| `List<Integer>` | ✓ | Correct, but slow to hash; never mutate it while it's a key |
| **`int[]`, `char[]`** | ✗ | Arrays use *identity*: two arrays with equal contents are different keys. Convert to `String` (`new String(chars)`) or `List` first |
| A packed `long` | ✓ | Fastest. For a pair of `int`s: `((long) a << 32) \| (b & 0xffffffffL)`. For grid cells: `(long) row * cols + col` |
| `row + "," + col` | ✓ | Simple but slow; fine for small inputs |

A custom class used as a key **must override both `equals` and `hashCode`** (records do it for you). If you override only `equals`, lookups fail silently.

## 16. Recursion depth

Each call uses stack memory. The default thread stack fits roughly **10⁴ to 10⁵ frames**, depending on how many local variables each frame holds. Past that, Java throws `StackOverflowError`. A DFS on a path-shaped graph with 10⁵ nodes, or a recursion over a 10⁵-character string, is at risk.

* **On LeetCode** you can't change the stack size. Depth around 10⁴ is safe; for depth near 10⁵, use an iterative version with an explicit stack (Ch 6 and Ch 21 show how).
* **On OA/CSES** you control `main`, so run the solver in a thread with a big stack:

```java
public static void main(String[] args) throws InterruptedException {
    Thread solver = new Thread(null, () -> {
        try {
            run();                                    // your real main logic
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    }, "solver", 1 << 26);                            // 64 MB of stack
    solver.start();
    solver.join();
}
```

### Worked J1 · [Reverse Integer](https://leetcode.com/problems/reverse-integer/)

> Given a signed 32-bit integer `x`, return `x` with its digits reversed. If the result falls outside the 32-bit range, return 0. *Interview twist: assume you cannot use 64-bit integers.*

**With `long` allowed,** accumulate in a `long`, then check the range once at the end:

```java
public int reverse(int x) {
    long result = 0;
    while (x != 0) {
        result = result * 10 + x % 10;     // x % 10 has the sign of x (§10), so negatives just work
        x /= 10;
    }
    return (result < Integer.MIN_VALUE || result > Integer.MAX_VALUE) ? 0 : (int) result;
}
```

**Without `long`,** check *before* each step that `result * 10 + digit` will stay in range. `Integer.MAX_VALUE = 2147483647`, so `MAX / 10 = 214748364` with last digit 7. `Integer.MIN_VALUE = -2147483648`, so `MIN / 10 = -214748364` (truncated toward zero) with last digit −8.

```java
public int reverse(int x) {
    int result = 0;
    while (x != 0) {
        int digit = x % 10;            // in [-9, 9], same sign as x
        x /= 10;
        if (result > Integer.MAX_VALUE / 10
                || (result == Integer.MAX_VALUE / 10 && digit > 7)) return 0;
        if (result < Integer.MIN_VALUE / 10
                || (result == Integer.MIN_VALUE / 10 && digit < -8)) return 0;
        result = result * 10 + digit;  // proven in range by the two checks
    }
    return result;
}
```

The technique here, **guarding the next operation instead of detecting overflow afterwards**, also solves String to Integer (atoi) and Divide Two Integers in the practice set.

---

# Part C — The collections contract

## 17. What everything costs

| Structure | Fast operations | Slow operations, and traps |
| :--- | :--- | :--- |
| `ArrayList<E>` | `get(i)`, `set(i, x)`: O(1) · `add(x)` at the end: amortised O(1) | `add(i, x)`, `remove(i)`: O(n − i) · `contains`, `indexOf`, `remove(Object)`: O(n) |
| `ArrayDeque<E>` | add/remove/peek at **both** ends: O(1) | No index access · `contains`: O(n) · **rejects `null`** |
| `PriorityQueue<E>` | `offer`, `poll`: O(log n) · `peek`: O(1) · building from a collection: O(n) | `remove(Object)`, `contains`: **O(n)** · iteration order is **not** sorted |
| `HashMap<K,V>` / `HashSet<E>` | `get`, `put`, `remove`, `containsKey`: O(1) expected | Iteration order is unspecified · each entry costs ~40–50 bytes |
| `LinkedHashMap<K,V>` | Same as `HashMap` | Iterates in insertion (or access) order |
| `TreeMap<K,V>` / `TreeSet<E>` | `get`, `put`, `remove`, `floor`/`ceiling`/`lower`/`higher`, `first`/`last`, `pollFirst`/`pollLast`: O(log n) · iteration is sorted | `headMap(x).size()` is **O(k), not O(log n)**: counting elements below x needs a Fenwick tree (Ch 34) |
| `String` | `charAt`, `length`: O(1) | `substring`, `+`, `equals`, `hashCode` (first call): O(length) |
| `StringBuilder` | `append`, `setCharAt`, delete at the end: O(1) amortised | `insert`, `deleteCharAt(i)` in the middle: O(n) |

**What *not* to use:** `Stack` (a legacy synchronized `Vector`; use `ArrayDeque`), `LinkedList` as a list or queue (one heap node per element, slow; use `ArrayList`/`ArrayDeque`), `Vector`, `Hashtable`.

## 18. Lists

```java
List<Integer> list = new ArrayList<>();
list.add(5);                     // append
list.get(0); list.set(0, 7);
list.remove(list.size() - 1);    // remove the last element: O(1)
list.remove(Integer.valueOf(7)); // remove by VALUE; list.remove(7) removes INDEX 7
Collections.sort(list);          // or list.sort(null) for natural order
Collections.reverse(list);
List<List<Integer>> result = new ArrayList<>();
result.add(new ArrayList<>(current));    // COPY when saving a list you'll keep modifying (backtracking!)
```

`List.of(1, 2, 3)` is **immutable**: `add` throws `UnsupportedOperationException`. `Arrays.asList(...)` is fixed-size: `set` works, `add` throws. Wrap either in `new ArrayList<>(...)` if you need to modify it.

## 19. `ArrayDeque`: stack, queue and deque

```java
// as a STACK (LIFO): all operations at the head
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1); stack.push(2);
int top = stack.peek();          // 2
int popped = stack.pop();        // 2

// as a QUEUE (FIFO): add at the tail, remove at the head
Deque<Integer> queue = new ArrayDeque<>();
queue.offer(1); queue.offer(2);
int front = queue.peek();        // 1
int served = queue.poll();       // 1

// as a DEQUE: explicit ends
Deque<Integer> dq = new ArrayDeque<>();
dq.offerFirst(1); dq.offerLast(2);
dq.peekFirst(); dq.peekLast(); dq.pollFirst(); dq.pollLast();
```

`peek()` and `poll()` return `null` when the deque is empty; `pop()` throws. `int x = queue.poll();` on an empty queue therefore throws `NullPointerException`. Check `isEmpty()` first.

## 20. `PriorityQueue`

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();                          // smallest first
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());  // largest first

// arrays as entries, e.g. {distance, node} for Dijkstra (distances can be long)
PriorityQueue<long[]> pq = new PriorityQueue<>((x, y) -> Long.compare(x[0], y[0]));
pq.offer(new long[] {0, source});
long[] best = pq.poll();
```

* **Changing an element after inserting it corrupts the heap.** If its priority changes, insert a new entry and skip stale ones when you poll them ("lazy deletion", Ch 15 and Ch 23).
* `remove(x)` and `contains(x)` are linear scans, so don't use them in a loop.
* Printing or iterating a heap shows its internal array order, not sorted order. To get the elements sorted, `poll` repeatedly.

## 21. `HashMap` and `HashSet` idioms

```java
Map<String, Integer> count = new HashMap<>();
count.merge(word, 1, Integer::sum);                      // count[word]++  (inserts 1 if absent)
int c = count.getOrDefault(word, 0);                     // read with a default
if (count.merge(word, -1, Integer::sum) == 0) count.remove(word);   // decrement; drop zeros

Map<Integer, List<Integer>> groups = new HashMap<>();
groups.computeIfAbsent(key, k -> new ArrayList<>()).add(value);     // map of lists

for (Map.Entry<String, Integer> e : count.entrySet()) {  // iterate keys and values together
    String key = e.getKey();
    int value = e.getValue();
}
count.entrySet().removeIf(e -> e.getValue() == 0);       // remove while iterating, safely

Set<Integer> seen = new HashSet<>();
if (!seen.add(x)) { /* x was already present: add() returns false */ }
```

When keys are small integers or characters, a plain array (`int[26]`, `int[n + 1]`, `boolean[]`) beats a map by a large constant factor. Prefer it whenever the key range allows.

## 22. `TreeMap` and `TreeSet`: ordered maps and sets

These keep keys **sorted** and answer "nearest key" questions in O(log n). They're the most underused tool in interview Java.

| `TreeMap` method | Returns | `TreeSet` equivalent |
| :--- | :--- | :--- |
| `firstKey()`, `lastKey()` | Smallest / largest key (throws if empty) | `first()`, `last()` |
| `floorKey(x)` | Largest key **≤ x**, or `null` | `floor(x)` |
| `ceilingKey(x)` | Smallest key **≥ x**, or `null` | `ceiling(x)` |
| `lowerKey(x)` | Largest key **< x**, or `null` | `lower(x)` |
| `higherKey(x)` | Smallest key **> x**, or `null` | `higher(x)` |
| `floorEntry(x)` etc. | The same, as a key–value entry | — |
| `pollFirstEntry()`, `pollLastEntry()` | Remove and return the min / max entry (`null` if empty) | `pollFirst()`, `pollLast()` |
| `headMap(x)`, `tailMap(x)`, `subMap(a, b)` | Live views of keys `< x`, `≥ x`, `[a, b)` | `headSet`, `tailSet`, `subSet` |
| `descendingMap()` | Reverse-order view | `descendingSet()` |

> **Null trap.** The navigation methods return `Integer`, which can be `null`. `int k = map.floorKey(x);` throws `NullPointerException` when nothing is ≤ x. Assign to `Integer` and check for `null`.

**A multiset** (a sorted collection that allows duplicates) is a `TreeMap` from value to count:

```java
TreeMap<Integer, Integer> bag = new TreeMap<>();
bag.merge(x, 1, Integer::sum);                              // add one copy of x
if (bag.merge(x, -1, Integer::sum) == 0) bag.remove(x);    // remove one copy (x MUST be present,
                                                           //  or this inserts a count of -1)
int smallest = bag.firstKey(), largest = bag.lastKey();
```

> **Comparator-equality trap.** A `TreeSet` treats two elements as *the same* whenever its comparator returns 0. A `TreeSet<int[]>` ordered only by `a[0]` silently **drops** every element whose `a[0]` is already present. Break ties with a unique field, such as an index: `(p, q) -> p[0] != q[0] ? Integer.compare(p[0], q[0]) : Integer.compare(p[1], q[1])`.

## 23. `LinkedHashMap`, `Collections` and immutable collections

* `new LinkedHashMap<>()` iterates in insertion order. `new LinkedHashMap<>(16, 0.75f, true)` iterates in *access* order, and overriding `removeEldestEntry` turns it into a ten-line LRU cache. Interviews usually want you to build that cache yourself (Ch 16), but it's worth knowing this exists.
* `Collections`: `sort`, `reverse`, `swap(list, i, j)`, `max`, `min`, `frequency(list, x)`, `nCopies(k, x)`, `emptyList()`.
* `List.of`, `Set.of`, `Map.of` (up to 10 pairs) create immutable collections, handy for constants and tests.

## 24. Changing a collection while iterating over it

Adding or removing elements inside a for-each loop over the same collection throws `ConcurrentModificationException`. Instead:

* use `iterator.remove()`, or `collection.removeIf(...)`; or
* collect the changes in a separate list and apply them after the loop.

Calling `map.put(k, newValue)` for a key that **already exists** during iteration is fine, because it isn't a structural change. Adding a *new* key isn't.

### Worked J2 · [Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words/)

> Given `words` and `k`, return the `k` most frequent words, sorted by frequency (highest first). Break ties alphabetically.

This tests three things from this chapter: counting with `merge`, a **two-key comparator** written correctly, and choosing between sorting everything and keeping a bounded heap.

**Version 1: sort all distinct words.** O(n + d log d) for d distinct words.

```java
public List<String> topKFrequent(String[] words, int k) {
    Map<String, Integer> count = new HashMap<>();
    for (String w : words) count.merge(w, 1, Integer::sum);

    List<String> distinct = new ArrayList<>(count.keySet());
    distinct.sort((a, b) -> {
        int byFrequency = Integer.compare(count.get(b), count.get(a));   // higher frequency first
        return byFrequency != 0 ? byFrequency : a.compareTo(b);          // then alphabetical
    });
    return new ArrayList<>(distinct.subList(0, k));   // subList is a view; copy it
}
```

**Version 2 *(a preview of Ch 15 §2: skim it now, master it there)*: a heap of size k.** O(n + d log k). This is the follow-up interviewers ask for. The heap's top must be the **worst** of the current k, so the comparator is the *reverse* of the answer order: lower frequency is worse, and for equal frequency the alphabetically *later* word is worse.

```java
public List<String> topKFrequent(String[] words, int k) {
    Map<String, Integer> count = new HashMap<>();
    for (String w : words) count.merge(w, 1, Integer::sum);

    PriorityQueue<String> heap = new PriorityQueue<>((a, b) -> {
        int byFrequency = Integer.compare(count.get(a), count.get(b));  // less frequent = worse = on top
        return byFrequency != 0 ? byFrequency : b.compareTo(a);         // alphabetically later = worse
    });
    for (String w : count.keySet()) {
        heap.offer(w);
        if (heap.size() > k) heap.poll();      // evict the current worst
    }
    List<String> result = new ArrayList<>();
    while (!heap.isEmpty()) result.add(heap.poll());   // comes out worst-first
    Collections.reverse(result);
    return result;
}
```

Getting the tie-break direction wrong in Version 2 is the single most common bug in this problem. **Test it with a tie:** `["a", "b", "a", "b", "c"], k = 2` must give `["a", "b"]`.

---

## 25. Predict-the-output lab

For each snippet, write down the exact output (or "compile error" / the exception) **before** running it. Score yourself; the exit gate requires 22/25. Assume `import java.util.*;`.

```java
/* 1 */  int a = 1_000_000; long b = a * a; System.out.println(b);
/* 2 */  long c = (long) a * a; System.out.println(c);
/* 3 */  System.out.println((-7 / 2) + " " + (-7 % 2));
/* 4 */  System.out.println(Math.floorMod(-7, 3) + " " + Math.floorDiv(-7, 3));
/* 5 */  System.out.println(Math.abs(Integer.MIN_VALUE));
/* 6 */  System.out.println((1 << 31) + " " + (1 << 32) + " " + (1L << 32));
/* 7 */  System.out.println(1 << 3 - 1);
/* 8 */  System.out.println('a' + 'b');
/* 9 */  System.out.println("" + 'a' + 'b');
/* 10 */ System.out.println(1 + 2 + "3" + 4 + 5);
/* 11 */ char ch = 'a'; ch += 2; System.out.println(ch);   // and: what does  ch = ch + 2;  do?
/* 12 */ System.out.println(0.1 + 0.2 == 0.3);
/* 13 */ System.out.println((int) -3.7 + " " + (int) 1e10);
/* 14 */ Integer x = 127, y = 127, p = 128, q = 128; System.out.println((x == y) + " " + (p == q));
/* 15 */ String s = "hi", t = new String("hi"); System.out.println((s == t) + " " + s.equals(t));
/* 16 */ Map<Long, String> m = new HashMap<>(); m.put(1L, "one"); System.out.println(m.containsKey(1));
/* 17 */ List<Integer> list = new ArrayList<>(List.of(5, 1, 3)); list.remove(1); System.out.println(list);
/* 18 */ int[] arr = {3, 1, 2}; System.out.println(Arrays.asList(arr).size());
/* 19 */ int[] a1 = {1, 2}; int[] b1 = a1; b1[0] = 9; System.out.println(a1[0]);
/* 20 */ int[][] g = {{1}, {2}}; int[][] h = g.clone(); h[0][0] = 7; System.out.println(g[0][0]);
/* 21 */ PriorityQueue<Integer> pq = new PriorityQueue<>(List.of(3, 1, 2)); System.out.println(pq.peek() + " " + pq);
/* 22 */ TreeMap<Integer, String> tm = new TreeMap<>(Map.of(10, "x", 20, "y"));
         System.out.println(tm.floorKey(15) + " " + tm.ceilingKey(25));   // and: int k = tm.floorKey(5); ?
/* 23 */ System.out.println("a,b,,c,,".split(",").length + " " + " a b".split(" ").length);
/* 24 */ int[] big = {1_500_000_000, 1_500_000_000}; System.out.println(Arrays.stream(big).sum());
/* 25 */ Integer[] v = {Integer.MIN_VALUE, 1}; Arrays.sort(v, (u, w) -> u - w); System.out.println(Arrays.toString(v));
```

<details><summary>Answers</summary>

| # | Output | Why |
| :-: | :--- | :--- |
| 1 | `-727379968` | `a * a` is computed in `int` and wraps; then it's widened (§9) |
| 2 | `1000000000000` | The cast happens first, so the multiplication is `long` |
| 3 | `-3 -1` | Truncation toward zero; `%` takes the dividend's sign (§10) |
| 4 | `2 -3` | Floor semantics |
| 5 | `-2147483648` | `MIN_VALUE` has no positive twin (§9) |
| 6 | `-2147483648 1 4294967296` | Shift distance mod 32 for `int` (§12) |
| 7 | `4` | `-` binds tighter than `<<`: `1 << 2` |
| 8 | `195` | `char + char` is `int` arithmetic (97 + 98) |
| 9 | `ab` | Starting with `""` makes it string concatenation, left to right |
| 10 | `3345` | `1 + 2 = 3` first, then concatenation |
| 11 | `c`; `ch = ch + 2` is a **compile error** | Compound assignment casts implicitly; plain `+` produces an `int` |
| 12 | `false` | Binary floating point (§11) |
| 13 | `-3 2147483647` | Truncation toward zero; a too-large double saturates |
| 14 | `true false` | `Integer` cache −128…127 (§13) |
| 15 | `false true` | `==` compares references |
| 16 | `false` | `1` boxes to `Integer`; an `Integer` never equals a `Long` (§13) |
| 17 | `[5, 3]` | `remove(int)` removes by **index** (§18) |
| 18 | `1` | `Arrays.asList(int[])` is a `List<int[]>` holding one element (§5) |
| 19 | `9` | Same array, two references (§7) |
| 20 | `7` | `clone()` of a 2D array is shallow: the rows are shared (§5) |
| 21 | `1 [1, 3, 2]` | `peek` gives the minimum; printing shows heap order, not sorted order (§20) |
| 22 | `10 null`; then `NullPointerException` | `ceilingKey(25)` has no answer; unboxing `null` throws (§22) |
| 23 | `4 3` | Trailing empty strings are dropped; a leading separator gives an empty first token (§6) |
| 24 | `-1294967296` | `IntStream.sum()` returns an `int` and wraps (§5) |
| 25 | `[1, -2147483648]` | The subtracting comparator overflows, so `MIN_VALUE` sorts as the larger one (§14) |
</details>

---

## 26. Version check for OA platforms

| Feature | Needs Java |
| :--- | :-: |
| `List.of`, `Set.of`, `Map.of` | 9 |
| `var` | 10 |
| `String.repeat`, `strip`, `isBlank` | 11 |
| `switch` with `->` arrows | 14 |
| `record` | 16 |
| `Math.ceilDiv` | 18 |

If an OA platform shows Java 8, avoid everything in this table. Use a small `static final class` with your own `equals`/`hashCode` instead of a record.

---

## 27. Practice

Easiest first. Each problem exercises a specific trap from this chapter, and the key names it.

1. **[Sqrt(x)](https://leetcode.com/problems/sqrtx/)** *(Easy)*
2. **[Sort Characters By Frequency](https://leetcode.com/problems/sort-characters-by-frequency/)** *(Medium)*
3. **[Pow(x, n)](https://leetcode.com/problems/powx-n/)** *(Medium)*
4. **[String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/)** *(Medium)*
5. ***(custom)* TreeMap multiset lab** (spec below)

**Stretch:** **[Divide Two Integers](https://leetcode.com/problems/divide-two-integers/)** *(Medium)* · **[Fraction to Recurring Decimal](https://leetcode.com/problems/fraction-to-recurring-decimal/)** *(Medium)*

### The multiset lab

Implement this class with every operation in O(log n), except `size()`, which must be O(1). Use a `TreeMap<Integer, Integer>` from value to count.

```java
final class IntMultiset {
    void add(int x)              // add one copy of x
    boolean removeOne(int x)     // remove one copy; return false if x isn't present
    int count(int x)             // copies of x
    int size()                   // total copies of everything
    Integer min()                // smallest element, or null if empty
    Integer max()                // largest element, or null if empty
    Integer floor(int x)         // largest element <= x, or null
    Integer higher(int x)        // smallest element > x, or null
}
```

Then check it against this obviously-correct slow version with 100 000 random operations. This is your first stress test; Ch 5 generalises it.

```java
final class BruteMultiset {
    private final List<Integer> items = new ArrayList<>();
    void add(int x) { items.add(x); }
    boolean removeOne(int x) { return items.remove(Integer.valueOf(x)); }
    int count(int x) { int c = 0; for (int v : items) if (v == x) c++; return c; }
    int size() { return items.size(); }
    Integer min() { return items.isEmpty() ? null : Collections.min(items); }
    Integer max() { return items.isEmpty() ? null : Collections.max(items); }
    Integer floor(int x) {
        Integer best = null;
        for (int v : items) if (v <= x && (best == null || v > best)) best = v;
        return best;
    }
    Integer higher(int x) {
        Integer best = null;
        for (int v : items) if (v > x && (best == null || v < best)) best = v;
        return best;
    }
}

// in main:
Random rng = new Random(42);
IntMultiset fast = new IntMultiset();
BruteMultiset slow = new BruteMultiset();
for (int step = 0; step < 100_000; step++) {
    int op = rng.nextInt(8);
    int x = rng.nextInt(21);                   // small value range, so duplicates are common
    Object expected;
    Object actual;
    switch (op) {
        case 0: fast.add(x); slow.add(x); expected = actual = "ok"; break;
        case 1: expected = slow.removeOne(x); actual = fast.removeOne(x); break;
        case 2: expected = slow.count(x); actual = fast.count(x); break;
        case 3: expected = slow.size(); actual = fast.size(); break;
        case 4: expected = slow.min(); actual = fast.min(); break;
        case 5: expected = slow.max(); actual = fast.max(); break;
        case 6: expected = slow.floor(x); actual = fast.floor(x); break;
        default: expected = slow.higher(x); actual = fast.higher(x); break;
    }
    if (!Objects.equals(expected, actual)) {
        System.out.println("step " + step + ", op " + op + ", x " + x
                + ": expected " + expected + ", got " + actual);
        return;
    }
}
System.out.println("all 100000 operations agree");
```

<details><summary>Nudges</summary>

1. Sqrt: binary search on the answer. What happens to `mid * mid` when `mid` is about 46 341?
2. Sort by Frequency: count, then sort the *distinct* characters by count with a correct comparator, then build the result with `StringBuilder`.
3. Pow: multiplying |n| times is too slow (|n| can be 2·10⁹). Halve the exponent instead: xⁿ = (x · x)^(n/2) when n is even, and x · (x · x)^(n/2) when n is odd, so about log₂ |n| ≈ 31 steps (Ch 18 §4 proves and generalises this "fast power"). For negative n, compute x^|n| and take 1 / result. What happens when you negate `n = Integer.MIN_VALUE`?
4. atoi: skip spaces, read an optional sign, read digits, and clamp *before* overflow, as in Worked J1.
5. Multiset: the null trap and the "remove a value that isn't present" trap are exactly what the stress test is there to catch.
6. Stretch, Divide: the only overflowing case is `MIN_VALUE / -1`. Then subtract doubled multiples of the divisor (in `long`, or working in negatives).
7. Stretch, Fraction: use `long` (why? `Math.abs(Integer.MIN_VALUE)`). Store each remainder's position in the output so you can tell where the repetition starts.
</details>

<details><summary>Key</summary>

Sqrt → J1 overflow: use `long mid` or compare `mid <= x / mid` · Sort by Frequency → J2 comparator + `StringBuilder` · Pow → J1: copy `n` into a `long` before negating · atoi → J1 guarded accumulation · Multiset lab → J2 TreeMap idioms (§22) · Divide → J1 edge case `MIN_VALUE / -1` · Fraction → J1 `long` + hashing remainders (§21)
</details>

---

## 28. Exit gate

Move on when you can:

* score **≥ 22/25** on the lab without running anything;
* write the **OA skeleton with `FastReader` and `PrintWriter`** (§2.2, §3.1) from a blank file in under 5 minutes, compiling first try;
* explain, without notes, why each of these is wrong: `long r = a * b;` · `(a, b) -> a - b` · `map.get(x) == map.get(y)` · `int k = treeMap.floorKey(x);` · `x % 2 == 1` for odd · `static` memo tables on LeetCode;
* show your multiset passing 100 000 random operations against the brute force.
