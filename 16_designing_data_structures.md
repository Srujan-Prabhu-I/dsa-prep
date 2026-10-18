# Chapter 16 — Designing Data Structures

> **Part II · Week 13** · Prerequisites: Ch 7 (hash maps), Ch 10 (binary search), Ch 12 (queues), Ch 13 (doubly linked lists), Ch 15 (heaps, ordered sets)
> **Problems:** 5 worked (1 ★) · 6 practice · 3 stretch · **Time:** about 8 hours

"Design a cache", "design a rate limiter", "design a leaderboard": these questions are common in FAANG and startup loops, and they sit closest to real backend work. No new algorithm is involved. What's tested is whether you can **combine structures you already know** so that *every* operation meets its required cost, and **keep those structures consistent** with each other as data changes.

This chapter teaches a method for doing that, then applies it to the five combinations that cover nearly every such question.

---

## 1. The design method

1. **List the operations and the cost each one must meet.** Write a table: operation → required complexity. "O(1) get and put" is the spec; everything else serves it.
2. **For each operation, name the structure that makes it cheap on its own.** Lookup by key → hash map. "The oldest / most recent" → a linked list or deque. "The smallest" → heap or ordered set. "Random element" → array. "The latest version at or before time t" → sorted list + binary search.
3. **Find the conflicts.** Usually no single structure serves every operation. A hash map has no order; a list has no fast lookup. **Combine them**, and decide what each one stores. Often one structure holds references *into* the other: the map points at list nodes, or at array positions.
4. **Write the invariant that ties them together.** For example: *"the map's keys are exactly the nodes in the list, and the list is ordered by recency."* This is the single most important line of the design.
5. **Write each operation as a sequence of steps that leaves the invariant true.** Then check each operation against the cost table.

In an interview, say steps 1–4 out loud before you write any code. Interviewers grade the invariant more than the syntax.

---

## 2. Attempt first ★

> **[LRU Cache](https://leetcode.com/problems/lru-cache/) (30 minutes).** Implement a cache with a fixed `capacity`: `get(key)` returns the value or −1, and `put(key, value)` inserts or updates. When it's full, evict the **least recently used** key. Both operations must be **O(1)**.

---

## 3. Hash map + doubly linked list: order with fast lookup

### Worked D1 · [LRU Cache](https://leetcode.com/problems/lru-cache/)

**Steps 1–3.** `get` and `put` need lookup by key: a hash map. Eviction needs "the least recently used", and every access must make a key "most recent". That's an *ordering by recency*, updated constantly:
* An array keeps order, but moving an element to the front is O(n).
* A singly linked list can't remove a node without walking to the node before it.
* A **doubly linked list** (Ch 13 §0) can remove any node it holds a reference to in O(1), and insert at the front in O(1).

So: **the map stores key → list node, and the list orders the nodes by recency.**

**Step 4, the invariant:** *the list (between two sentinels) contains exactly the nodes in the map, most recently used first.* The sentinel `head` and `tail` nodes are Ch 13's dummy-node habit applied at both ends. With them, insertion and removal never have to handle an empty list or a missing neighbour.

```java
class LRUCache {
    private static final class Node {
        final int key;                 // stored so eviction can delete the key from the map
        int value;
        Node prev, next;
        Node(int key, int value) { this.key = key; this.value = value; }
    }

    private final int capacity;
    private final Map<Integer, Node> byKey = new HashMap<>();
    private final Node head = new Node(0, 0), tail = new Node(0, 0);   // sentinels; head.next is the most recent

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        Node node = byKey.get(key);
        if (node == null) return -1;
        moveToFront(node);                       // it has just been used
        return node.value;
    }

    public void put(int key, int value) {
        Node node = byKey.get(key);
        if (node != null) {                      // update: change the value, refresh recency
            node.value = value;
            moveToFront(node);
            return;
        }
        if (byKey.size() == capacity) {          // full: evict the least recently used, just before tail
            Node lru = tail.prev;
            unlink(lru);
            byKey.remove(lru.key);
        }
        node = new Node(key, value);
        byKey.put(key, node);
        insertAfterHead(node);
    }

    private void moveToFront(Node node) { unlink(node); insertAfterHead(node); }

    private void unlink(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void insertAfterHead(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
}
```

Every operation does O(1) map work plus O(1) pointer changes. **Why each node stores its key:** eviction finds the victim through the *list*, but must also delete it from the *map*, and only the key can do that. Forgetting this is the most common LRU bug.

Java's `LinkedHashMap` in access order (Ch 4 §23) is this exact design, built in. Mention it, but expect to be asked to write the real thing.

---

## 4. Hash map + array: O(1) removal *and* random access

### Worked D2 · [Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o-1/)

> A set supporting `insert(val)`, `remove(val)` and `getRandom()` (every element equally likely), each in O(1) on average.

* `getRandom` needs **index access** to choose uniformly: an array (`ArrayList`) plus a random index.
* `remove(val)` needs to find `val` (a hash map from value → index) and delete it from the array without O(n) shifting.

**The trick: order inside the array doesn't matter.** To delete the element at position i, **move the last element into position i** and drop the last slot. That's the swap-remove you used inside hash buckets in Ch 7. Then update the moved element's index in the map.

> **Invariant:** `values` holds exactly the set's elements, and `indexOf.get(v)` is v's position in `values`, for every element v.

```java
class RandomizedSet {
    private final List<Integer> values = new ArrayList<>();          // the elements, in no particular order
    private final Map<Integer, Integer> indexOf = new HashMap<>();   // value -> its position in values
    private final Random rng = new Random();

    public boolean insert(int val) {
        if (indexOf.containsKey(val)) return false;
        indexOf.put(val, values.size());
        values.add(val);
        return true;
    }

    public boolean remove(int val) {
        Integer i = indexOf.remove(val);
        if (i == null) return false;
        int last = values.remove(values.size() - 1);    // O(1): removing from the end
        if (i < values.size()) {                         // val wasn't the last element:
            values.set(i, last);                         // move the last one into its slot
            indexOf.put(last, i);
        }
        return true;
    }

    public int getRandom() { return values.get(rng.nextInt(values.size())); }
}
```

**The edge case that breaks most first attempts:** removing the element that is *already last*. The code handles it by removing the last slot first, then moving the old last element only if a gap remains (`i < values.size()`). Trace `insert(1); remove(1)` to see why the order matters. Allowing duplicates (RandomizedCollection, stretch) changes the map to value → **set of positions**.

---

## 5. Frequency buckets: least frequently used

### Worked D3 · [LFU Cache](https://leetcode.com/problems/lfu-cache/)

> Like LRU, but evict the key with the **smallest use count**. Among ties, evict the **least recently used**. `get` and `put` in O(1).

A heap ordered by (count, last use) gives O(log n). For O(1), notice that **counts only ever increase by exactly 1**. So keep a **bucket per count**: count → the keys with that count, in recency order. A use moves a key from bucket f to bucket f + 1. To evict, take the oldest key in the **lowest non-empty bucket**. Tracking `minFreq` is easy:
* a *new* key has count 1, so `minFreq` becomes 1;
* a use empties bucket f only if f was the minimum, and then the new minimum is f + 1.

Each bucket needs O(1) add, O(1) removal of a specific key, and O(1) access to its oldest key. Java's `LinkedHashSet` gives all three: it's a hash set that also remembers insertion order, so iteration starts from the oldest element. (A hand-built doubly linked list per bucket, as in LRU, is the from-scratch version.)

> **Invariants:** each key is in exactly one bucket, the one for its current count, and buckets are ordered from oldest use to newest. `minFreq` is the smallest count with a non-empty bucket, whenever the cache isn't empty.

```java
class LFUCache {
    private final int capacity;
    private final Map<Integer, Integer> valueOf = new HashMap<>();
    private final Map<Integer, Integer> countOf = new HashMap<>();
    private final Map<Integer, LinkedHashSet<Integer>> keysWithCount = new HashMap<>();   // oldest use first
    private int minFreq = 0;

    public LFUCache(int capacity) { this.capacity = capacity; }

    public int get(int key) {
        if (!valueOf.containsKey(key)) return -1;
        touch(key);
        return valueOf.get(key);
    }

    public void put(int key, int value) {
        if (capacity == 0) return;
        if (valueOf.containsKey(key)) {
            valueOf.put(key, value);
            touch(key);
            return;
        }
        if (valueOf.size() == capacity) {                       // evict: lowest count, then the oldest use
            LinkedHashSet<Integer> lowest = keysWithCount.get(minFreq);
            int victim = lowest.iterator().next();
            lowest.remove(victim);
            valueOf.remove(victim);
            countOf.remove(victim);
        }
        valueOf.put(key, value);
        countOf.put(key, 1);
        keysWithCount.computeIfAbsent(1, c -> new LinkedHashSet<>()).add(key);
        minFreq = 1;                                            // a new key always has the lowest count
    }

    // One more use: move the key from bucket f to bucket f + 1, keeping minFreq right.
    private void touch(int key) {
        int f = countOf.get(key);
        LinkedHashSet<Integer> bucket = keysWithCount.get(f);
        bucket.remove(key);
        if (f == minFreq && bucket.isEmpty()) minFreq = f + 1;
        countOf.put(key, f + 1);
        keysWithCount.computeIfAbsent(f + 1, c -> new LinkedHashSet<>()).add(key);   // newest in its new bucket
    }
}
```

The order of operations in `put` matters. Evict *before* inserting the new key, or the new key (count 1) would be the eviction victim. All O(1) Medium (AllOne, stretch) uses the same bucket idea, with keys moving up *or down* one count at a time.

---

## 6. Versions over time: binary search on history

### Worked D4 · [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/)

> `set(key, value, timestamp)` and `get(key, timestamp)`. `get` returns the value set at the **largest timestamp ≤ the query**, or `""`. Timestamps passed to `set` are strictly increasing.

For each key, keep its history as a list of (timestamp, value), appended in order. Because timestamps only increase, **the list is already sorted**, with no sorting needed. `get` is then "the last version with time ≤ t": Ch 10's template, returning `lo`.

```java
class TimeMap {
    private record Version(int time, String value) {}
    private final Map<String, List<Version>> history = new HashMap<>();   // each list sorted by time (appended in order)

    public void set(String key, String value, int timestamp) {
        history.computeIfAbsent(key, k -> new ArrayList<>()).add(new Version(timestamp, value));
    }

    public String get(String key, int timestamp) {
        List<Version> versions = history.get(key);
        if (versions == null) return "";
        int lo = -1, hi = versions.size();          // the last index with time <= timestamp is lo
        while (hi - lo > 1) {
            int mid = lo + (hi - lo) / 2;
            if (versions.get(mid).time() > timestamp) hi = mid; else lo = mid;
        }
        return lo < 0 ? "" : versions.get(lo).value();
    }
}
```

O(1) per `set` and O(log v) per `get`. If timestamps could arrive **out of order**, a `TreeMap<Integer, String>` per key with `floorEntry` does the same job in O(log v) for both. Snapshot Array (practice) is the same structure with "snapshot ids" in place of timestamps. This is also exactly how *multi-version* databases answer "what was the value as of time t?".

---

## 7. Time windows: a queue per client

### Worked D5 *(custom)* · Sliding-Window Rate Limiter

> Implement `allow(user, now)`: allow a request if that user has had **fewer than `limit` allowed requests in the last `windowMs` milliseconds** (the window `(now − windowMs, now]`), and record it if allowed. Timestamps are non-decreasing.

**What has to be remembered:** for each user, the times of their recent allowed requests. Old times expire from the front, and new ones join at the back. That's a **queue per user** (Ch 12). Each timestamp enters the queue once and leaves once, so `allow` is **amortised O(1)**.

```java
final class SlidingWindowRateLimiter {
    private final int limit;
    private final long windowMs;
    private final Map<String, ArrayDeque<Long>> recent = new HashMap<>();   // per user: allowed request times, oldest first

    SlidingWindowRateLimiter(int limit, long windowMs) {
        this.limit = limit;
        this.windowMs = windowMs;
    }

    boolean allow(String user, long now) {
        ArrayDeque<Long> times = recent.computeIfAbsent(user, u -> new ArrayDeque<>());
        while (!times.isEmpty() && times.peekFirst() <= now - windowMs) times.pollFirst();   // expired
        if (times.size() >= limit) return false;
        times.offerLast(now);
        return true;
    }
}
```

**This is a backend design question in miniature.** The follow-ups an interviewer will ask:

| Follow-up | Answer |
| :--- | :--- |
| Memory per user is O(limit). Can it be O(1)? | The **sliding-window counter**: keep counts for the current and previous fixed windows, and weight the previous one by how much of it still overlaps. It's approximate, but O(1) |
| Bursts should be allowed up to a cap, with a steady refill | **Token bucket** (stretch): store `tokens` and `lastRefill`; on each request, refill lazily by `(now − lastRefill) × rate`, capped at the capacity |
| Users who stop sending requests stay in memory forever | Remove a user's entry when their queue empties, or expire idle users periodically |
| Many threads call `allow` at once | Thread-safe maps (`ConcurrentHashMap`) plus per-user locking. At scale, the state lives in a shared store (e.g. Redis) with atomic operations |

The algorithmic core stays the same; these are the trade-offs a backend interviewer is looking for.

---

## 8. Where designs break

| Symptom | Cause | Fix |
| :--- | :--- | :--- |
| Correct alone, wrong in sequence | One operation leaves the cross-structure invariant broken | Write the invariant first; check every operation restores it |
| An O(n) step hiding in an "O(1)" design | `list.remove(Object)`, `pq.remove(x)`, scanning a bucket | Store a direct reference or index instead |
| "O(1)" that relies on hashing | Expected, not worst case | Say "O(1) expected" (Ch 7 §0.2) |
| Unbounded memory growth | Entries never expire (rate limiters, caches without eviction) | An eviction or expiry policy is part of the design |
| `LinkedHashMap` / library shortcut | Often not accepted as "the design" | Know both versions; lead with the explicit one |

---

## 9. Recognition

**Signals:** "Design a class with operations A, B, C in O(…)"; caches; "most recent / least recent / most frequent"; "random element"; "as of time t"; "per user, in the last T seconds"; iterators over nested data.

**The structure menu:**

| Requirement | Structure |
| :--- | :--- |
| Lookup by key | Hash map |
| Order by recency, with O(1) moves | Doubly linked list (+ the map pointing at its nodes) |
| Uniform random element with O(1) removal | Array + swap-remove + a value → index map |
| Counts that change by ±1, with an O(1) min/max count | Buckets per count |
| The latest version ≤ t | Sorted history + binary search, or a `TreeMap` |
| The smallest or largest under arbitrary updates | Heap with lazy deletion, or an ordered set |
| Events in a recent time window | Queue / deque per key |
| A sorted view by a changing score | `TreeSet` with a tie-breaking comparator; remove, update, re-add |

---

## 10. Practice

1. **[Design Browser History](https://leetcode.com/problems/design-browser-history/)** *(Medium)*
2. **[Design Underground System](https://leetcode.com/problems/design-underground-system/)** *(Medium)*
3. **[Snapshot Array](https://leetcode.com/problems/snapshot-array/)** *(Medium)*
4. **[Flatten Nested List Iterator](https://leetcode.com/problems/flatten-nested-list-iterator/)** *(Medium)*
5. **[Design Twitter](https://leetcode.com/problems/design-twitter/)** *(Medium)*
6. **[Design a Food Rating System](https://leetcode.com/problems/design-a-food-rating-system/)** *(Medium)*

**Stretch:** **[All O\`one Data Structure](https://leetcode.com/problems/all-o-one-data-structure/)** *(Hard)* · **[Insert Delete GetRandom O(1) – Duplicates allowed](https://leetcode.com/problems/insert-delete-getrandom-o-1-duplicates-allowed/)** *(Hard)* · ***(custom)* Token-Bucket Rate Limiter**: `allow(now)` with a `capacity` and a refill `rate` per second. Verify it against a brute force that simulates the bucket one millisecond at a time.

<details><summary>Nudges</summary>

1. Browser History: an array of pages, a `current` index and a `last` index. Visiting overwrites from `current + 1` and resets `last`.
2. Underground: map id → (station, time) for check-ins; map "from→to" → (total time, trip count) for averages.
3. Snapshot Array: for each index, a list of (snapId, value) with only the changes. `get` is the last entry with snapId ≤ the query (D4).
4. Nested Iterator: a stack of iterators. Let `hasNext()` do the work: expand nested lists until the top holds an integer.
5. Twitter: per user, a list of (globalTime, tweetId); per user, the set of followees. The feed is a k-way merge (Ch 15 §3) of the followees' latest tweets, stopping at 10.
6. Food Ratings: map food → (cuisine, rating); map cuisine → `TreeSet` ordered by (rating descending, name ascending). To change a rating: remove, update, re-add.
7. Stretch, AllOne: a doubly linked list of buckets ordered by count, each holding a set of keys, plus a map key → its bucket. inc/dec moves a key to the neighbouring bucket, creating or deleting buckets as needed.
8. Stretch, RandomizedCollection: map value → set of indices. Swap-remove still works; update the moved element's index set.
9. Stretch, Token Bucket: `tokens = min(capacity, tokens + (now − last) × rate)`; allow if `tokens ≥ 1`, then subtract 1. Use `double`, or scale to integers.
</details>

<details><summary>Key</summary>

Browser History → array + pointer · Underground → two maps · Snapshot → D4 (versions + binary search) · Nested Iterator → stack of iterators · Twitter → k-way merge (Ch 15) · Food Ratings → map → TreeSet with a tie-break · AllOne → D3 buckets · RandomizedCollection → D2 with index sets · Token Bucket → lazy refill arithmetic
</details>

---

## 11. Self-test

<details><summary>1. Why does the LRU list need to be doubly linked?</summary>

To unlink a node in O(1) you need its predecessor, and a singly linked list only lets you reach that by walking from the head.
</details>

<details><summary>2. Why must each LRU node store its key?</summary>

Eviction finds the victim through the list, but must also remove it from the map, and only the key can do that.
</details>

<details><summary>3. In RandomizedSet.remove, why remove the last element before moving it into the gap?</summary>

If the removed value *was* the last element, there's no gap to fill. Removing the last slot first and checking `i < values.size()` handles that case without writing back a deleted value.
</details>

<details><summary>4. How does the LFU cache keep minFreq correct in O(1)?</summary>

Counts only grow by 1. A new key resets `minFreq` to 1, and a use empties bucket f only if f was the minimum, in which case the new minimum is f + 1.
</details>

<details><summary>5. Why is the sliding-window rate limiter amortised O(1) even though `allow` can expire many timestamps at once?</summary>

Each timestamp is added once and removed once over its whole lifetime, so the total number of removals never exceeds the number of additions.
</details>

---

## 12. Exit gate

Move on when you can:

* run the five-step design method out loud for a new design problem, including the cross-structure invariant;
* write LRU Cache and Insert Delete GetRandom O(1) from blank files, each in under 15 minutes, passing the edge cases first time;
* explain LFU's bucket design and the `minFreq` update rule;
* discuss the rate limiter's follow-ups: memory, bursts, expiry and concurrency;
* solve 5 of the 6 practice problems.
