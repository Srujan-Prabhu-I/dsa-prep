# Chapter 5 — Arrays, Strings & Implementation

> **Part I · Week 3** · Prerequisites: Ch 1, Ch 4
> **Problems:** 6 worked (1 ★) · 16 practice · 1 stretch · **Time:** about 8 hours

Arrays and strings aren't a technique. They're the material almost every problem is made of. This chapter makes handling them **automatic**: traversing, building, parsing and transforming them in place, without fumbling. It also introduces the most important tool in the book after the derivation protocol: the **stress-testing harness**, which finds the bugs your eyes can't.

Three things you'll be able to do by the end:

1. **Build a dynamic array from scratch**, and explain exactly why `add` is O(1) on average.
2. **Write grid and string code that is correct the first time**, because every loop has a stated invariant and every boundary has a reason.
3. **Prove your solution matches a brute force on 100 000 random inputs**, in the five minutes after you finish coding.

---

## 0. From zero: how arrays work

### 0.1 The array

An array is **one contiguous block of memory** holding `n` values of the same type. Element `i` lives at

> address of `a[i]` = address of `a[0]` + `i` × (size of one element)

so reaching *any* index is a single multiplication and addition: **O(1)**, however big the array is. For `int[] a = {7, 2, 9, 4, 1}` (4 bytes per `int`), starting at address 1000:

```text
index:      0      1      2      3      4
          +------+------+------+------+------+
a ──────► |  7   |  2   |  9   |  4   |  1   |      a.length = 5
          +------+------+------+------+------+
address:  1000   1004   1008   1012   1016

a[3] is at 1000 + 3 × 4 = 1012: computed directly, never searched for.
```

The variable `a` doesn't hold the values. It holds a **reference** (the arrow) to the block. That's why passing an array to a method lets the method change your elements (Ch 4 §7).

**Inserting in the middle** is what's expensive. Inserting 5 at index 1 means opening a gap, so every later element moves one slot right, starting from the end:

```text
before:            [ 7 | 2 | 9 | 4 | 1 | _ ]       _ = spare slot
shift 1 → slot 5   [ 7 | 2 | 9 | 4 | 1 | 1 ]
shift 4 → slot 4   [ 7 | 2 | 9 | 4 | 4 | 1 ]
shift 9 → slot 3   [ 7 | 2 | 9 | 9 | 4 | 1 ]
shift 2 → slot 2   [ 7 | 2 | 2 | 9 | 4 | 1 ]
write 5 at 1       [ 7 | 5 | 2 | 9 | 4 | 1 ]       4 moves for n = 5: O(n) in the worst case
```

Shifting from the **end** backwards matters: going forwards would overwrite 9 with 2 before 9 had been moved. Direct addressing and shifting together explain the whole cost table:

| Operation | Cost | Why |
| :--- | :-: | :--- |
| Read or write `a[i]` | O(1) | One address calculation |
| Find a value (unsorted) | O(n) | Nothing tells you where it is, so you look everywhere |
| Insert or delete in the middle | O(n) | Everything after the position must shift by one |
| Insert or delete at the end | O(1) | Nothing shifts, *if there's spare room* (see §0.2) |

Java adds two things on top: every array knows its `length`, and every access is **bounds-checked**. `a[n]` throws `ArrayIndexOutOfBoundsException` instead of reading garbage memory, as C would.

**2D arrays in Java are arrays of arrays.** `int[][] g = new int[R][C]` is an array of R references, each pointing to a separate row of C ints. Each row is contiguous, but the rows are not contiguous with each other. For `new int[3][4]`:

```text
g ──► +---+
      | ●─┼──► [ 0 | 0 | 0 | 0 ]     g[0]      g[r][c]: follow g[r], then index c
      +---+
      | ●─┼──► [ 0 | 0 | 0 | 0 ]     g[1]      g.length    = 3 (rows)
      +---+
      | ●─┼──► [ 0 | 0 | 0 | 0 ]     g[2]      g[0].length = 4 (columns)
      +---+
```

Two consequences:
* Rows can have **different lengths** (jagged arrays), and `g[r].length` is the length of row `r`.
* **Traverse row by row** (`for r … for c …`), not column by column. Reading along a row uses memory the CPU has just loaded; jumping between rows doesn't. On a 4000 × 4000 grid the column-first loop can be several times slower, with the same Big-O.

### 0.2 The dynamic array: building one yourself

A plain array has a fixed size. `ArrayList` gives you an array that **grows**. Here's the whole trick:

* Keep an internal array (the **capacity**) that is usually larger than the number of elements actually stored (the **size**).
* Appending writes into the spare room: O(1).
* When the room runs out, allocate a **bigger** array, copy everything across (O(n) for that one append), and continue.

Appending 1, 2, 3, 4, 5 to a list that starts with capacity 2 and doubles:

```text
add 1   [1 | _ ]                    size 1, capacity 2
add 2   [1 | 2 ]                    size 2, capacity 2   (full)
add 3   [1 | 2 | 3 | _ ]            no room: new array of 4, copy 2 elements, then write
add 4   [1 | 2 | 3 | 4 ]            size 4, capacity 4   (full)
add 5   [1 | 2 | 3 | 4 | 5 | _ | _ | _ ]   no room: new array of 8, copy 4, then write
```

Only the appends that hit a full array pay for a copy, and they get rarer as the array grows.

**How much bigger?** This decides everything. Suppose you grow by **doubling**, starting from capacity 1, and append n elements. The copies happen at sizes 1, 2, 4, 8, …, up to n, so the total number of elements ever copied is

> 1 + 2 + 4 + … + n < 2n

(a geometric sum, Ch 3 §2.2). So n appends cost O(n) in total, which is **amortised O(1) per append**, even though an individual append is sometimes O(n). If you instead grew by a *fixed* amount (say, 10 more slots each time), the copies would cost 10 + 20 + 30 + … ≈ n²/20, which is quadratic. **Growing by a constant factor is what makes it work.** Java's `ArrayList` uses 1.5×; the argument is the same.

Build it:

```java
import java.util.Arrays;
import java.util.NoSuchElementException;

final class IntList {
    private int[] data = new int[4];   // capacity = data.length
    private int size = 0;              // elements in use: data[0 .. size)

    void add(int x) {                                    // amortised O(1)
        if (size == data.length) data = Arrays.copyOf(data, data.length * 2);
        data[size++] = x;
    }

    int get(int i) { checkIndex(i); return data[i]; }   // O(1)
    void set(int i, int x) { checkIndex(i); data[i] = x; }
    int size() { return size; }

    int removeLast() {                                   // O(1)
        if (size == 0) throw new NoSuchElementException();
        return data[--size];
    }

    void insert(int i, int x) {                          // O(size - i): shift the tail right
        if (i < 0 || i > size) throw new IndexOutOfBoundsException("index " + i + ", size " + size);
        if (size == data.length) data = Arrays.copyOf(data, data.length * 2);
        System.arraycopy(data, i, data, i + 1, size - i);
        data[i] = x;
        size++;
    }

    int removeAt(int i) {                                // O(size - i): shift the tail left
        checkIndex(i);
        int removed = data[i];
        System.arraycopy(data, i + 1, data, i, size - i - 1);
        size--;
        return removed;
    }

    private void checkIndex(int i) {
        if (i < 0 || i >= size) throw new IndexOutOfBoundsException("index " + i + ", size " + size);
    }
}
```

`System.arraycopy(src, srcPos, dst, dstPos, length)` copies a block correctly even when the source and destination overlap, which is exactly the situation when shifting within one array.

**Verify it.** Run 100 000 random operations (add, get, set, removeLast, insert, removeAt) on your `IntList` and on a `java.util.ArrayList<Integer>` side by side, and compare after every step. This is the same pattern as the multiset lab in Ch 4, and §6 turns it into a general tool.

### 0.3 Strings are arrays you can't change

A Java `String` is an array of `char`s with one extra rule: it is **immutable**. Every "change" creates a new string.

```text
String s = "cat";        s ──► [ c | a | t ]
s = s + "s";             s ──► [ c | a | t | s ]      a NEW array; the old one is copied, then dropped
```

So `s += c` inside a loop copies the whole string each time: O(n²) for n characters. Build strings with a `StringBuilder` (a dynamic array of chars, exactly like `IntList`), or work on a `char[]` and convert once at the end (Ch 4 §6).

**Characters are small numbers.** `'a'` is 97, `'b'` is 98, …, so `c - 'a'` turns a lowercase letter into an index 0–25, which is how a `count[26]` array works:

```text
"banana"  →  count[ 'a'-'a' = 0 ] = 3,  count[ 'b'-'a' = 1 ] = 1,  count[ 'n'-'a' = 13 ] = 2
```

### 0.4 Common misconceptions

| Belief | Reality |
| :--- | :--- |
| "`ArrayList.add` is O(1)" | Amortised O(1): one add can cost O(n), but n adds cost O(n) in total |
| "`list.remove(0)` is cheap" | O(n): everything shifts left. Use a deque for removals at the front (Ch 12) |
| "`b = a` copies the array" | It copies the **reference**: `a` and `b` now point to the same block. Use `a.clone()` or `Arrays.copyOf` |
| "`s1 == s2` compares strings" | It compares references. Use `s1.equals(s2)` (Ch 4 §13) |
| "`int[][] g = new int[3][]` makes a 3 × something grid" | It makes 3 **null** row references; each row must be created |

---

## 1. The rules of correct implementation

Most implementation bugs come from a handful of habits. These rules remove them, and every solution in this book follows them.

1. **State what each variable means, as a comment, before writing the loop.** "`write` = the number of elements kept so far" or "the unvisited cells are rows `[top, bottom]` × columns `[left, right]`". If you can't say what a variable means, you can't update it correctly.
2. **Pick one interval convention and keep it.** This book uses **half-open `[l, r)`** for ranges: `l` included, `r` excluded. The length is `r − l`, an empty range is `l == r`, and splitting `[l, r)` at `m` gives `[l, m)` and `[m, r)` with no overlap and no gap. Java's own APIs agree (`substring`, `copyOfRange`, `subList`). When a problem is naturally closed, like the four borders of a spiral, say so in the invariant.
3. **Check bounds before you access**, and in the right order: `r >= 0 && r < rows && grid[r][c] == '#'`. Java's `&&` stops at the first `false`, so the access never happens out of range.
4. **Use direction arrays instead of copy-pasting four blocks:**

   ```java
   static final int[] DR = {-1, 0, 1, 0};     // up, right, down, left (clockwise)
   static final int[] DC = {0, 1, 0, -1};
   for (int d = 0; d < 4; d++) {
       int nr = r + DR[d], nc = c + DC[d];
       if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
       // visit (nr, nc)
   }
   ```

   Turning right is `d = (d + 1) % 4`; turning left is `d = (d + 3) % 4`. Never write `(d − 1) % 4`: in Java that's `-1` when `d = 0` (Ch 4 §10).
5. **Keep parse, solve and print separate.** The `solve` method takes values and returns values. That's what lets you stress-test it (§6).
6. **Flattening a grid cell to one number:** `id = r * cols + c`, and back with `r = id / cols`, `c = id % cols`. Use `long` if `rows × cols` can exceed about 2·10⁹.

---

## 2. Building and parsing strings

### Worked I1 · [Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/)

> Given a string `s` of words separated by spaces (possibly with leading, trailing or repeated spaces), return the words in reverse order, separated by single spaces.
>
> `"  the sky  is blue "` → `"blue is sky the"`.

**The quick version** uses the library: trim the string, split on runs of whitespace, then join the words in reverse.

```java
public String reverseWords(String s) {
    String[] words = s.trim().split("\\s+");      // trim first, or a leading space gives an empty word
    StringBuilder sb = new StringBuilder();
    for (int i = words.length - 1; i >= 0; i--) {
        sb.append(words[i]);
        if (i > 0) sb.append(' ');
    }
    return sb.toString();
}
```

That's acceptable, but interviewers often follow up with "do it without `split`". **The manual scan** walks from the right, skips spaces, finds each word's boundaries, and appends it. The invariant: `i` is the position we're about to examine, and everything to its right has been handled.

```java
public String reverseWords(String s) {
    StringBuilder result = new StringBuilder();
    int i = s.length() - 1;
    while (i >= 0) {
        while (i >= 0 && s.charAt(i) == ' ') i--;     // skip spaces
        if (i < 0) break;
        int end = i + 1;                               // the word is s[start, end)
        while (i >= 0 && s.charAt(i) != ' ') i--;
        int start = i + 1;
        if (result.length() > 0) result.append(' ');
        result.append(s, start, end);                  // appends s[start, end) without a substring copy
    }
    return result.toString();
}
```

Both versions are O(n). The second makes every boundary explicit, and that skill is what parsing questions test.

**Variation (preview):** "the input is a `char[]`, reverse the words *in place*". Reverse the whole array, then reverse each word back. The words end up in reverse order and each one reads forwards again. Rotate Array in Mixed Set II uses the same three-reversal trick.

---

## 3. Traversing in a pattern

### Worked I2 · [Spiral Matrix](https://leetcode.com/problems/spiral-matrix/) ★

> **Attempt first (20 minutes).** Given an `m × n` matrix, return all its elements in clockwise spiral order, starting at the top-left. `[[1,2,3],[4,5,6],[7,8,9]]` → `[1,2,3,6,9,8,7,4,5]`.

The difficulty isn't the idea; it's the boundaries. Single rows, single columns, and the moment the spiral's last ring is one cell wide are where most attempts break. The fix is an invariant:

> **Invariant:** the unvisited cells are exactly rows `[top, bottom]` × columns `[left, right]`, both inclusive.

Each lap peels off one side of that rectangle and moves the matching border inwards. The two `if` guards handle the cases where peeling the top row or the right column has already used up the rectangle:

```java
public List<Integer> spiralOrder(int[][] matrix) {
    List<Integer> order = new ArrayList<>();
    int top = 0, bottom = matrix.length - 1, left = 0, right = matrix[0].length - 1;
    while (top <= bottom && left <= right) {
        for (int c = left; c <= right; c++) order.add(matrix[top][c]);      // top row, left to right
        top++;
        for (int r = top; r <= bottom; r++) order.add(matrix[r][right]);    // right column, downwards
        right--;
        if (top <= bottom) {                                                // a bottom row remains
            for (int c = right; c >= left; c--) order.add(matrix[bottom][c]);
            bottom--;
        }
        if (left <= right) {                                                // a left column remains
            for (int r = bottom; r >= top; r--) order.add(matrix[r][left]);
            left++;
        }
    }
    return order;
}
```

**Test it on the shapes that break spirals:** a 1×1 grid, a 1×4 single row, a 4×1 single column, a 2×3 and a 3×2. In the 1×4 case, without the first guard, the "bottom row" loop would re-read the top row backwards.

**The alternative:** walk with a direction array (§1, rule 4), and turn right whenever the next cell is outside the grid or already visited. It costs an extra `boolean[][]`, but it has no special cases. It's also the easiest way to *fill* a grid in spiral order, the common follow-up.

---

## 4. Index transforms

### Worked I3 · [Rotate Image](https://leetcode.com/problems/rotate-image/)

> Rotate an `n × n` matrix 90° clockwise, **in place**.

**Find the mapping first.** Clockwise rotation sends the cell at `(r, c)` to `(c, n − 1 − r)`: the top row becomes the right column. Writing that mapping directly in place means juggling cycles of four cells. A cleaner route is to **compose two simple flips**:

* **Transpose** (flip across the main diagonal): `(r, c) → (c, r)`.
* **Reverse each row** (flip left to right): `(r, c) → (r, n − 1 − c)`.

Applied in that order, they send `(r, c) → (c, r) → (c, n − 1 − r)`, which is exactly the rotation. ✓ Each flip is easy to do in place, because it's just a set of swaps.

```java
public void rotate(int[][] matrix) {
    int n = matrix.length;
    for (int r = 0; r < n; r++)                        // transpose: swap each pair once (c > r)
        for (int c = r + 1; c < n; c++) {
            int tmp = matrix[r][c];
            matrix[r][c] = matrix[c][r];
            matrix[c][r] = tmp;
        }
    for (int[] row : matrix)                           // reverse every row
        for (int l = 0, r = n - 1; l < r; l++, r--) {
            int tmp = row[l];
            row[l] = row[r];
            row[r] = tmp;
        }
}
```

The transpose loop starts at `c = r + 1`. Starting at 0 would swap every pair *twice* and undo itself.

**The family of transforms.** Every square-grid symmetry is a composition of a few flips. Derive each one with the same mapping method rather than memorising it:

| Transform | Mapping | In place |
| :--- | :--- | :--- |
| Rotate 90° clockwise | `(r, c) → (c, n−1−r)` | transpose, then reverse each row |
| Rotate 90° counter-clockwise | `(r, c) → (n−1−c, r)` | transpose, then reverse each column |
| Rotate 180° | `(r, c) → (n−1−r, n−1−c)` | reverse the row order, then reverse each row |

---

## 5. Updating everything at once: in-place state encoding

### Worked I4 · [Game of Life](https://leetcode.com/problems/game-of-life/)

> Each cell of a grid is live (1) or dead (0). Compute the next generation **in place**, where every cell updates *simultaneously* from its 8 neighbours:
> * a live cell with 2 or 3 live neighbours stays live; otherwise it dies;
> * a dead cell with exactly 3 live neighbours becomes live.

**The trap:** if you overwrite a cell with its new state, its neighbours, processed later, read the *new* value instead of the old one. The easy fix is a copy of the board (O(rows × cols) extra space). The in-place fix is to **store both states in the same cell** until everyone has read the old state:

> cell value = **current** + 2 × **next**. Then `value % 2` is the current state and `value / 2` is the next one.

During the first pass, everyone reads `% 2`, the current state, which is never disturbed. The second pass replaces each cell with `/ 2`:

```java
public void gameOfLife(int[][] board) {
    int rows = board.length, cols = board[0].length;
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            int live = 0;
            for (int dr = -1; dr <= 1; dr++)
                for (int dc = -1; dc <= 1; dc++) {
                    if (dr == 0 && dc == 0) continue;
                    int nr = r + dr, nc = c + dc;
                    if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) live += board[nr][nc] % 2;
                }
            boolean alive = board[r][c] % 2 == 1;
            if ((alive && (live == 2 || live == 3)) || (!alive && live == 3)) {
                board[r][c] += 2;                          // record "live next generation"
            }
        }
    }
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++) board[r][c] /= 2;  // next becomes current
}
```

The same idea, *packing extra information into values the algorithm still reads correctly*, solves Set Matrix Zeroes in the practice set (there, the first row and column become marker storage) and Find All Duplicates in Ch 7 (there, the sign of a number becomes a marker). Ch 17 shows the bitwise version of the same trick.

---

## 6. The stress-testing harness

You've already run two stress tests: the multiset in Ch 4 and the coin rule in Ch 2. Here is the general tool. It finds a bug in seconds that would take an hour to find by staring at the code, or that you'd never find, because it lives in a case you didn't think of.

### 6.1 The idea

You need three pieces:

1. **`solve`:** the fast solution you want to check.
2. **`brute`:** a slow solution that is **obviously correct**. Try every possibility; no cleverness. If it takes more than 5 minutes to write, it's too clever.
3. **`generate`:** random **small** inputs. "Small" matters for two reasons. It keeps the brute force fast, and it produces failing cases you can trace by hand in a minute.

Then loop: generate an input, run both, compare. Stop at the first disagreement and print the input.

**Make the random inputs nasty.** Use tiny value ranges (−3 to 3, not −10⁹ to 10⁹) so duplicates, zeros and ties appear constantly. That's where bugs live. And **start with tiny sizes and grow them slowly**, so the first failure you find is as small as possible.

### 6.2 The template

Copy this file for every problem. Only the three marked methods change.

```java
import java.util.*;

public class Stress {
    // (1) The solution under test.
    static int solve(int[] a) {
        return 0;   // paste your solution here
    }

    // (2) Slow but obviously correct.
    static int brute(int[] a) {
        return 0;   // try every possibility
    }

    // (3) Random small inputs: tiny values make duplicates, zeros and ties common.
    static int[] generate(Random rng, int maxN) {
        int n = 1 + rng.nextInt(maxN);
        int[] a = new int[n];
        for (int i = 0; i < n; i++) a[i] = rng.nextInt(7) - 3;     // values in [-3, 3]
        return a;
    }

    public static void main(String[] args) {
        long seed = args.length > 0 ? Long.parseLong(args[0]) : System.nanoTime();
        Random rng = new Random(seed);
        for (int iter = 1; iter <= 100_000; iter++) {
            int maxN = Math.min(10, 1 + iter / 5_000);   // sizes 1, then up to 2, … up to 10
            int[] a = generate(rng, maxN);
            int expected = brute(a.clone());              // clone: neither may see the other's changes
            int actual = solve(a.clone());
            if (expected != actual) {
                System.out.println("Counterexample after " + iter + " tests (seed " + seed + ")");
                System.out.println("  input:    " + Arrays.toString(a));
                System.out.println("  expected: " + expected);
                System.out.println("  actual:   " + actual);
                return;
            }
        }
        System.out.println("100000 random tests passed (seed " + seed + ")");
    }
}
```

Three details that matter:
* **The seed is printed.** Running `java Stress.java 12345` replays exactly the same tests, so you can confirm your fix.
* **Inputs are cloned** before each call. If `solve` sorts its input in place, the brute force must not see the sorted version, or the two aren't testing the same thing.
* **`main` doesn't `catch` exceptions.** If `solve` crashes with an index error, you *want* the stack trace, plus the input that caused it: print the input before calling `solve` if you suspect a crash.

### 6.3 When there's more than one correct answer: the checker

Some problems accept *any* valid answer: "return any peak", "return any valid ordering". Comparing with a brute-force answer would report false failures. Replace the comparison with a **checker** that validates the answer directly:

```java
static boolean isValid(int[] input, int answer) {
    // Example, "return the index of any peak": the index is in range and the element there
    // is at least as large as both neighbours that exist.
    int n = input.length;
    if (answer < 0 || answer >= n) return false;
    boolean leftOk = answer == 0 || input[answer] >= input[answer - 1];
    boolean rightOk = answer == n - 1 || input[answer] >= input[answer + 1];
    return leftOk && rightOk;
}
// in the loop:  if (!isValid(a, solve(a.clone()))) { print the input; return; }
```

If the problem also needs the *best* value (say, the minimum cost), check both: the answer is valid, and its cost equals the brute force's optimal cost.

### 6.3b Adapting the harness to any problem

The template's `int[] → int` shape is only an example. Change three things to match your problem: the **types** of `solve` and `brute`, the **comparison**, and the **generator**.

| Output type | Compare with |
| :--- | :--- |
| `int`, `long`, `boolean` | `expected != actual` |
| `int[]`, `long[]`, `char[]` | `!Arrays.equals(expected, actual)` |
| `int[][]`, `char[][]` | `!Arrays.deepEquals(expected, actual)` |
| `String`, `List<Integer>`, `List<String>` | `!expected.equals(actual)` (list `equals` compares element by element) |
| A list of answers in **any order** (all subsets, all paths) | Sort both first (below), then `equals` |
| `double` | `Math.abs(expected - actual) > 1e-9` |
| "Any valid answer" | A checker (§6.3) |

Sorting a `List<List<Integer>>` so that two answer lists can be compared regardless of order:

```java
static List<List<Integer>> canonical(List<List<Integer>> answers) {
    List<List<Integer>> copy = new ArrayList<>();
    for (List<Integer> a : answers) { List<Integer> s = new ArrayList<>(a); Collections.sort(s); copy.add(s); }  // skip the inner sort if order inside matters
    copy.sort((x, y) -> {                                            // compare element by element, then by length
        for (int i = 0; i < Math.min(x.size(), y.size()); i++) {
            int c = Integer.compare(x.get(i), y.get(i));
            if (c != 0) return c;
        }
        return Integer.compare(x.size(), y.size());
    });
    return copy;
}
```

**Generators for other inputs,** always small, with tiny value ranges:

```java
StringBuilder sb = new StringBuilder();                                     // a random string over {a, b, c}
for (int i = 1 + rng.nextInt(8); i > 0; i--) sb.append((char) ('a' + rng.nextInt(3)));
String s = sb.toString();

char[][] grid = new char[1 + rng.nextInt(4)][1 + rng.nextInt(4)];          // a random 0/1 grid
for (char[] row : grid) for (int c = 0; c < row.length; c++) row[c] = rng.nextBoolean() ? '1' : '0';

int[] parent = new int[n];                                                  // a random tree on n vertices:
parent[0] = -1;                                                             // each vertex hangs under an earlier one
for (int v = 1; v < n; v++) parent[v] = rng.nextInt(v);

List<int[]> edges = new ArrayList<>();                                      // a random graph: each pair with probability 1/3
for (int u = 0; u < n; u++) for (int v = u + 1; v < n; v++) if (rng.nextInt(3) == 0) edges.add(new int[] {u, v});
```

For linked lists and binary trees, generate an array and convert it with the builders in Ch 13 §0 and Ch 19 §0.2. For a "random tree" whose labels shouldn't follow the parent order, shuffle the labels afterwards.

### Worked I5 *(custom)* · The planted-bug hunt

Here's a solution with a bug in it. It passes all the "obvious" examples. Find the bug **with the harness, not by reading.**

> Return the **second-largest distinct** value in an array, or `Integer.MIN_VALUE` if there isn't one. `[3, 1, 2]` → 2, `[1, 2]` → 1, `[7]` → `Integer.MIN_VALUE`.

```java
static int solve(int[] a) {
    int first = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
    for (int x : a) {
        if (x > first) {
            second = first;
            first = x;
        } else if (x > second) {
            second = x;
        }
    }
    return second;
}

static int brute(int[] a) {
    int[] distinct = Arrays.stream(a).distinct().sorted().toArray();
    return distinct.length >= 2 ? distinct[distinct.length - 2] : Integer.MIN_VALUE;
}
```

Put both into the template and run it. Within a few dozen tests, it reports an input like **`[0, 0]`**: expected `Integer.MIN_VALUE`, actual `0`. Now the bug is obvious. A *second copy of the maximum* fails the `x > first` test and lands in `else if (x > second)`, so the maximum gets recorded as the second-largest. The fix is `else if (x > second && x != first)`. Rerun with the same seed to confirm.

> **A design smell the harness also exposes:** using `Integer.MIN_VALUE` to mean "none" breaks if the input can *contain* `Integer.MIN_VALUE`. A clearer API returns an `Integer` (`null` for none) or a `boolean` flag. Keep this in mind when you design your own function signatures.

### 6.4 Stress-testing programs that read standard input (optional)

For OA programs, keep `solve` separate from the I/O (Ch 4 §3.1) and stress-test `solve` directly with §6.2. That's the easiest route. If you need to test a whole stdin program, compile once and loop in a script. On **Windows PowerShell**, do every redirection through `cmd /c`: PowerShell 5.1's own `>` writes UTF-16 text, and its pipes add a byte-order mark, and either one breaks Java's input parsing.

```powershell
# stress.ps1: Gen.java prints a random input for the seed it's given as an argument.
javac Gen.java Sol.java Brute.java
for ($seed = 1; $seed -le 500; $seed++) {
    cmd /c "java Gen $seed > input.txt"
    $mine  = cmd /c "java Sol < input.txt"
    $truth = cmd /c "java Brute < input.txt"
    if (($mine -join "`n") -ne ($truth -join "`n")) {
        "Mismatch on seed $seed"; Get-Content input.txt; "sol:   $mine"; "brute: $truth"; break
    }
}
```

The bash equivalent, on Linux or macOS:

```bash
javac Gen.java Sol.java Brute.java
for seed in $(seq 1 500); do
  java Gen "$seed" > input.txt
  if ! diff <(java Sol < input.txt) <(java Brute < input.txt) > /dev/null; then
    echo "Mismatch on seed $seed"; cat input.txt; break
  fi
done
```

---

## 7. Implementation traps

| Trap | Example | Prevention |
| :--- | :--- | :--- |
| Off-by-one on a boundary | Spiral re-reading a row | Write the invariant; test 1×1, 1×n, n×1 |
| Reading a cell after overwriting it | Game of Life without encoding | Encode both states, or write into a copy |
| Swapping a pair twice | Transpose loop starting at `c = 0` | Visit each unordered pair once (`c > r`) |
| Negative modulo in a direction turn | `(d − 1) % 4` | `(d + 3) % 4` |
| Shared state between calls | `static` fields on LeetCode (Ch 4 §7) | Instance fields or locals |
| Mutated input in a test | `solve` sorts the array the brute force then reads | Clone the input for each call |
| An empty string from `split` | `" a b".split(" ")` | `trim()` and `split("\\s+")`, or a manual scan |
| Overflow in index arithmetic | `r * cols + c` with 10⁵ × 10⁵ | `long`, or keep `(r, c)` separate |

---

## 8. Practice

The CSES problems are full stdin/stdout programs, so use the OA skeleton from Ch 4 §3.1. Everything else is LeetCode. **Stress-test at least three of these with the harness**, including one you're confident is correct.

1. **[Pascal's Triangle](https://leetcode.com/problems/pascals-triangle/)** *(Easy)*
2. **[Valid Anagram](https://leetcode.com/problems/valid-anagram/)** *(Easy)*
3. **[Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/)** *(Easy)*
4. **[CSES Increasing Array](https://cses.fi/problemset/task/1094)** *(Easy)*
5. **[Add Binary](https://leetcode.com/problems/add-binary/)** *(Easy)*
6. **[CSES Permutations](https://cses.fi/problemset/task/1070)** *(Easy)*
7. **[String Compression](https://leetcode.com/problems/string-compression/)** *(Medium)*
8. **[Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/)** *(Medium; follow-up: O(1) extra space)*
9. **[Valid Sudoku](https://leetcode.com/problems/valid-sudoku/)** *(Medium)*
10. **[Diagonal Traverse](https://leetcode.com/problems/diagonal-traverse/)** *(Medium)*
11. **[Zigzag Conversion](https://leetcode.com/problems/zigzag-conversion/)** *(Medium)*
12. **[Robot Bounded In Circle](https://leetcode.com/problems/robot-bounded-in-circle/)** *(Medium)*
13. **[Walking Robot Simulation](https://leetcode.com/problems/walking-robot-simulation/)** *(Medium)*
14. **[CSES Number Spiral](https://cses.fi/problemset/task/1071)** *(Medium)*
15. **[CSES Coin Piles](https://cses.fi/problemset/task/1754)** *(Medium)*
16. **[Multiply Strings](https://leetcode.com/problems/multiply-strings/)** *(Medium)*

**Stretch:** **[Text Justification](https://leetcode.com/problems/text-justification/)** *(Hard)*

<details><summary>Nudges</summary>

1. Pascal: row `i` has `i + 1` entries; each inner entry is the sum of the two above it.
2. Anagram: one `int[26]`. Add for one string, subtract for the other, then check every count is zero.
3. LCP: compare column by column across all strings; stop at the first mismatch or the end of the shortest string.
4. Increasing Array: walk left to right keeping the running maximum; each element must be raised to it. The total can exceed `int`.
5. Add Binary: walk both strings from the right with a carry; build in a `StringBuilder`, then reverse.
6. Permutations: there's no answer for n = 2 or 3. For larger n, try all the even numbers, then all the odd ones.
7. Compression: a reader index and a writer index over the same array. Write each character, then its count's digits if the count is more than 1.
8. Set Zeroes: use the first row and first column as marker storage. Remember whether they themselves need zeroing *before* you use them.
9. Sudoku: the 3×3 box index of `(r, c)` is `(r / 3) * 3 + c / 3`.
10. Diagonal Traverse: all cells on one diagonal share `r + c`. Alternate the direction on each diagonal.
11. Zigzag: the row pattern repeats with period `2 · numRows − 2`. Or simulate with a direction that bounces between the first and last rows.
12. Robot Bounded: after running the instructions once, the robot is bounded if it's back at the origin *or* facing a different direction. Why does that mean it returns within 4 runs?
13. Walking Robot: put the obstacles in a `HashSet` of packed keys or records; move one step at a time.
14. Number Spiral: the answer depends on `max(row, col)` and on whether that value is even or odd. Find the formula from small cases. Use `long`.
15. Coin Piles: removing 2 from one pile and 1 from the other. What must hold for `a + b`, and for `max(a, b)` compared with `min(a, b)`?
16. Multiply Strings: digit `i` of one number times digit `j` of the other lands in position `i + j + 1` of an `int[m + n]`.
17. Stretch, Text Justification: greedily fill each line, then share the extra spaces out, giving the leftmost gaps the remainder. The last line is a special case.
</details>

<details><summary>Key</summary>

Pascal → 2D jagged building (§0.1) · Anagram → `int[26]` + `c − 'a'` (Ch 4 §4) · LCP → column scan with early exit · Increasing Array → running max, `long` total (fast I/O) · Add Binary → right-to-left carry with `StringBuilder` (§2) · Permutations → construction + stdout · Compression → reader/writer in place (§1, rule 1) · Set Zeroes → in-place markers (§5) · Sudoku → box-index transform (§4) · Diagonal Traverse → index sums (§3) · Zigzag → period arithmetic or a bouncing direction (§1, rule 4) · Robot Bounded → direction array + "after 4 runs" invariant · Walking Robot → direction array + hashed obstacles · Number Spiral → formula from small cases, `long` · Coin Piles → invariants: `(a + b) % 3 == 0` and `2·min ≥ max` · Multiply Strings → digit-position arithmetic · Text Justification → careful simulation with a stated invariant per line
</details>

---

## 9. Self-test

<details><summary>1. Why must a dynamic array grow by a constant factor rather than a constant amount?</summary>

With a factor, the copies form a geometric series that sums to O(n), so appends are amortised O(1). With a fixed increment k, the copies cost k + 2k + 3k + … = Θ(n²/k), so appends are amortised O(n/k).
</details>

<details><summary>2. Which two inputs break a spiral traversal that has no guards?</summary>

A single row (1 × n) and a single column (n × 1). Without the guards, the loop walks back over cells it has already visited.
</details>

<details><summary>3. In the in-place Game of Life, what does a cell value of 2 mean?</summary>

Currently dead (2 % 2 = 0) and live next generation (2 / 2 = 1).
</details>

<details><summary>4. Your stress test passes 100 000 cases with values in [−10⁹, 10⁹]. What's wrong with that test?</summary>

With such a wide range, duplicates, ties and zeros almost never occur, so the cases most likely to break your code are barely tested. Use tiny value ranges.
</details>

<details><summary>5. When do you need a checker instead of a brute force to compare against?</summary>

When more than one output is correct ("return any …"). The checker validates the property directly; if optimality matters too, it also compares the cost with the brute force's optimum.
</details>

---

## 10. Exit gate

Move on when you can:

* write `IntList` from a blank file and show it matching `ArrayList` on 100 000 random operations;
* write Spiral Matrix and Rotate Image from blank files in under 10 minutes each, passing the 1×1, 1×n and n×1 cases first time;
* write the stress harness (§6.2) from memory and use it to find the planted bug in under 5 minutes;
* solve 13 of the 16 practice problems, and have stress-tested at least three of them.
