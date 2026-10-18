# Chapter 13 — Linked Lists: Pointer Surgery

> **Part II · Week 11** · Prerequisites: Ch 1–12 (especially Ch 2 invariants, Ch 6 recursion)
> **Problems:** 6 worked (1 ★) · 8 practice · 2 stretch · **Time:** about 7 hours

Linked lists show up in interviews far more often than in real code, for one reason: they test whether you can **change references without losing anything**. The algorithms are short. The difficulty is purely in the order of assignments, the empty and one-node cases, and the head of the list. This chapter gives you three habits that remove almost all of those bugs (draw it, save before you overwrite, use a dummy node), and two genuinely clever ideas: fast/slow pointers, with their proof, and weaving a copy into the original.

---

## 0. From zero: what a linked list is

A **linked list** stores elements in separate **nodes**, scattered anywhere in memory. Each node holds a value and a **reference to the next node**; the last node's reference is `null`. You hold on to the list through its first node, the **head**.

```java
public class ListNode {          // LeetCode's definition
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}
```

```text
head
 │
 ▼
[3 | •]──▶[8 | •]──▶[5 | null]
```

Walking the list: `for (ListNode cur = head; cur != null; cur = cur.next) { … }`.

**For testing** (including the stress harness, Ch 5 §6), convert between arrays and lists:

```java
static ListNode build(int... values) {                 // [3, 8, 5] -> 3 -> 8 -> 5
    ListNode dummy = new ListNode(0), tail = dummy;
    for (int v : values) { tail.next = new ListNode(v); tail = tail.next; }
    return dummy.next;
}

static List<Integer> toList(ListNode head) {           // 3 -> 8 -> 5 -> [3, 8, 5], easy to compare with equals
    List<Integer> out = new ArrayList<>();
    for (ListNode cur = head; cur != null; cur = cur.next) out.add(cur.val);
    return out;
}
```

**Where the nodes really are.** Unlike an array, the nodes are separate objects at unrelated addresses. Only the `next` references put them in order:

```text
address 5200: [3 | next = 7400]     address 7400: [8 | next = 1100]     address 1100: [5 | next = null]
```

So there's no formula for "the address of element i": reaching it means following i references.

**Inserting after a node you hold** (insert 6 after the node `p` holding 8) changes two references, **in this order**:

```text
before:     [3]──▶[8]──▶[5]──▶null
                   p

step 1:     node.next = p.next               [6]──▶[5]        the new node points at 8's old successor
step 2:     p.next = node             [3]──▶[8]──▶[6]──▶[5]──▶null
```

Do step 2 first, and `p.next` no longer leads to `[5]`: the rest of the list is lost. **Deleting the node after `p`** is one assignment, `p.next = p.next.next`, which makes the list skip it:

```text
before:  [3]──▶[8]──▶[6]──▶[5]──▶null          after:  [3]──▶[8]─────────▶[5]──▶null
                p     victim                                 p     [6] is unreachable: garbage-collected
```

Both are O(1), but notice they need the node **before** the position. In a singly linked list you can't go backwards, which is why so many algorithms track `prev`.

**Costs, compared with an array:**

| Operation | Array | Linked list | Why |
| :--- | :-: | :-: | :--- |
| Read element i | O(1) | **O(i)** | No addresses to compute; you must walk |
| Insert or delete **at a node you already hold** | O(n) (shifting) | **O(1)** | Change one or two references |
| Insert or delete at the front | O(n) | O(1) | — |
| Memory per element | 4 bytes (`int[]`) | ~16–24 bytes | Each node is an object with a header and a reference |
| Scanning speed | fast | slower | Nodes are scattered, so the CPU cache helps less (Ch 5 §0.1) |

So a linked list wins only when you hold references to nodes and insert or delete *at* them. LRU caches (Ch 16) are the classic case. A **doubly linked list** adds a `prev` reference to each node, so a node can be removed knowing only the node itself, without walking to its predecessor:

```text
null◀──[3]⇄[8]⇄[5]──▶null          remove [8]:   node.prev.next = node.next;   node.next.prev = node.prev;
                                     result:      null◀──[3]⇄[5]──▶null
```

**Common misconceptions:**

| Belief | Reality |
| :--- | :--- |
| "Linked-list insertion is O(1)" | Only at a node you already hold. Finding position i first costs O(i) |
| "`java.util.LinkedList` is a good default list" | Almost never: it's slower than `ArrayList` for nearly everything. Use it (or better, `ArrayDeque`) only for end operations |
| "`cur = cur.next` changes the list" | It only moves your local reference. The list changes only when you assign to some node's `.next` |
| "Removing a node deletes it" | It becomes unreachable, and the garbage collector frees it; any reference you kept still works |
| "`a == b` for nodes compares values" | It compares identity: the same node object. Compare `a.val == b.val` for values |

### Worked P0 · [Design Linked List](https://leetcode.com/problems/design-linked-list/)

> Implement `get(index)`, `addAtHead`, `addAtTail`, `addAtIndex(index, val)` (when `index == size` append; when `index > size` do nothing) and `deleteAtIndex(index)`.

The single most useful habit: put a **dummy node** *before* the first real node. Then every real node, including the first, has a predecessor, and "insert at the front" and "delete the head" stop being special cases:

```text
dummy──▶[3]──▶[8]──▶[5]──▶null        insert at index 0 = insert after dummy; delete index 0 = skip dummy.next
```

```java
class MyLinkedList {
    private static final class Node {
        int val;
        Node next;
        Node(int val, Node next) { this.val = val; this.next = next; }
    }

    private final Node dummy = new Node(0, null);   // never removed; dummy.next is the real first node
    private int size = 0;

    // The node just before position `index` (index in [0, size]). For index 0, the dummy.
    private Node before(int index) {
        Node node = dummy;
        for (int i = 0; i < index; i++) node = node.next;
        return node;
    }

    public int get(int index) {
        if (index < 0 || index >= size) return -1;
        return before(index).next.val;
    }

    public void addAtHead(int val) { addAtIndex(0, val); }
    public void addAtTail(int val) { addAtIndex(size, val); }

    public void addAtIndex(int index, int val) {
        if (index < 0 || index > size) return;
        Node prev = before(index);
        prev.next = new Node(val, prev.next);       // the new node points at the old successor, then prev points at it
        size++;
    }

    public void deleteAtIndex(int index) {
        if (index < 0 || index >= size) return;
        Node prev = before(index);
        prev.next = prev.next.next;                  // bypass the node; Java's garbage collector frees it
        size--;
    }
}
```

Everything goes through one helper, `before(index)`, and the dummy makes `before(0)` valid. That's the whole design.

---

## 1. Three habits for pointer surgery

1. **Draw before and after.** Draw boxes and arrows for the "before" state and for the "after" state. Then write one assignment per arrow that changes. Most linked-list bugs are assignments in the wrong order.
2. **Save before you overwrite.** If you're about to change `cur.next`, and you still need the old `cur.next`, save it in a local variable first.
3. **Use a dummy head** whenever the head might change (insertions, deletions, merges, partitions), and return `dummy.next`.

Then **test the four cases** every linked-list function must survive: the empty list (`null`), one node, two nodes, and the case that touches the head or the tail.

---

## 2. Reversal

### Worked P1a · [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)

**Iteratively.** Walk the list once and turn each arrow around. Three references:

> **Invariant:** `prev` is the head of the already-reversed part, and `cur` is the head of the part not yet reversed.

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null, cur = head;
    while (cur != null) {
        ListNode next = cur.next;     // save it (habit 2): we're about to overwrite cur.next
        cur.next = prev;              // turn this arrow around
        prev = cur;                   // the reversed part now starts at cur
        cur = next;
    }
    return prev;                      // cur == null: everything is reversed
}
```

On `1 → 2 → 3`, at the top of each iteration:

```text
         reversed part        not yet reversed
start    prev = null          cur = [1]──▶[2]──▶[3]──▶null
iter 1   null◀──[1]           cur = [2]──▶[3]──▶null          (save next = 2, point 1 back, advance)
iter 2   null◀──[1]◀──[2]     cur = [3]──▶null
iter 3   null◀──[1]◀──[2]◀──[3]   cur = null: return prev = [3]
```

**Recursively, with the contract from Ch 6:** "`reverseList(head)` reverses the list starting at `head` and returns the new head".

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;   // 0 or 1 nodes: already reversed
    ListNode newHead = reverseList(head.next);             // trust it: the rest is reversed, ending at head.next
    head.next.next = head;                                 // that last node now points back at head
    head.next = null;                                      // head becomes the new tail
    return newHead;
}
```

The recursive version uses O(n) stack. On a 10⁵-node list it can overflow the Java stack (Ch 4 §16), so prefer the iterative one unless the list is short.

### Worked P1b · [Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/)

> Reverse every consecutive group of k nodes. A final group shorter than k stays as it is. O(1) extra space.

It's the same three-pointer reversal, applied to one group at a time, plus the bookkeeping to reconnect each group. For each group:
1. Find its k-th node. If there aren't k nodes left, stop.
2. Reverse the group, starting with `prev = groupNext` (the node after the group). The group's first node, which becomes its *last* after reversal, then points onward automatically.
3. Connect the node before the group to the group's new first node, and move on.

```java
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode groupPrev = dummy;                  // the node just before the next group
    while (true) {
        ListNode kth = groupPrev;
        for (int i = 0; i < k && kth != null; i++) kth = kth.next;
        if (kth == null) break;                  // fewer than k nodes left: leave them
        ListNode groupNext = kth.next;
        ListNode prev = groupNext, cur = groupPrev.next;
        while (cur != groupNext) {               // reverse exactly this group
            ListNode next = cur.next;
            cur.next = prev;
            prev = cur;
            cur = next;
        }
        ListNode oldFirst = groupPrev.next;      // was the group's first node; now its last
        groupPrev.next = kth;                    // kth is now the group's first node
        groupPrev = oldFirst;
    }
    return dummy.next;
}
```

Each node is visited a constant number of times: O(n) time, O(1) space. Draw a 5-node list with k = 2 and trace the first group by hand. That's the fastest way to make this code yours.

---

## 3. Fast and slow pointers

Two pointers walk the same list at different speeds: `slow` one step at a time, `fast` two steps.

* **The middle:** when `fast` reaches the end, `slow` is halfway. Loop while `fast != null && fast.next != null`. For an even length this stops `slow` at the *second* middle node; start `fast` at `head.next` to get the first one.
* **The k-th node from the end:** move `fast` k steps ahead first, then move both at the same speed. When `fast` reaches the end, `slow` is k from the end (Remove Nth Node From End, practice).

### Worked P2 · [Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/) ★

> **Attempt first (25 minutes).** A list may have a cycle: some node's `next` points back to an earlier node. Return the node where the cycle begins, or `null`, using **O(1) extra space**.

A `HashSet` of visited nodes finds the entry immediately, but uses O(n) space. **Floyd's algorithm** uses two pointers.

**Step 1: detect.** If there's no cycle, `fast` reaches `null`. If there is one, both pointers eventually enter it. From then on, `fast` gains exactly one node on `slow` every step, so the distance between them (measured around the cycle) shrinks by one each step. It hits zero within one lap: **they meet.**

**Step 2: find the entry.** Let
* `a` = the distance from the head to the cycle's entry,
* `b` = the distance from the entry to the meeting point (along the cycle),
* `c` = the cycle's length.

When they meet, `slow` has walked `a + b`, and `fast` has walked twice as far, `2(a + b)`. The extra distance `a + b` consists of whole laps of the cycle: **`a + b = k·c`** for some whole number k ≥ 1. So **`a = k·c − b`**. In words: walking `a` steps from the meeting point goes `c − b` steps to reach the entry, then `(k − 1)` whole laps, and ends **exactly at the entry**.

So put one pointer back at the head and leave the other at the meeting point, then move both one step at a time. After `a` steps, both are at the entry, so **they meet there**.

```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {                       // they've met inside the cycle
            ListNode p = head;
            while (p != slow) {                   // both walk `a` steps and meet at the entry
                p = p.next;
                slow = slow.next;
            }
            return p;
        }
    }
    return null;                                  // fast fell off the end: there's no cycle
}
```

**The same idea on an array.** Ch 10 found the duplicate in an array of n + 1 values in `[1, n]` in O(n log n). Treat each index `i` as a node pointing to index `nums[i]`. Index 0 is the head, since no value points to it. The duplicate value has two indices pointing at it, so it's where the path *enters* a cycle. Floyd finds it in **O(n) time and O(1) space**:

```java
public int findDuplicate(int[] nums) {
    int slow = nums[0], fast = nums[nums[0]];
    while (slow != fast) { slow = nums[slow]; fast = nums[nums[fast]]; }
    int p = 0;
    while (p != slow) { p = nums[p]; slow = nums[slow]; }
    return p;
}
```

Structures where every node has exactly one outgoing arrow are called *functional graphs*. Ch 25 treats them in general.

---

## 4. Merging and splicing with a dummy node

### Worked P3 · [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)

> Merge two sorted lists into one sorted list by relinking their nodes.

This is the merge step of merge sort (Ch 9 §1), on nodes instead of an array. Keep a `tail` pointer to the last node of the merged list. Its dummy start means the first append isn't a special case.

```java
public ListNode mergeTwoLists(ListNode a, ListNode b) {
    ListNode dummy = new ListNode(0), tail = dummy;    // tail = last node of the merged list so far
    while (a != null && b != null) {
        if (b.val < a.val) { tail.next = b; b = b.next; }
        else               { tail.next = a; a = a.next; }   // ties take a: stable
        tail = tail.next;
    }
    tail.next = (a != null) ? a : b;                   // attach whatever remains, in one step
    return dummy.next;
}
```

O(n + m) time and O(1) extra space: no new nodes are created. With this and "find the middle" (§3), **merge sort works directly on a linked list** in O(n log n) (Sort List, practice). Splitting at the middle replaces array indexing. Partition List (practice) uses *two* dummies, one per output list, and joins them at the end.

---

## 5. Cloning a structure that points into itself

### Worked P4 · [Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/)

> Each node has `next` and also `random`, which points to any node in the list or to `null`. Make a deep copy.

**The obstacle:** when you create the copy of node A, the copy of `A.random` might not exist yet.
* **With a map** (old node → its copy): first create every copy, then set each copy's `next` and `random` by looking them up. That's O(n) time and O(n) extra space, and perfectly fine in an interview.
* **Follow-up, O(1) extra space:** *weave* each copy directly after its original, `A → A′ → B → B′ → …`. Now "the copy of X" is simply `X.next`, so the map isn't needed:

```java
public Node copyRandomList(Node head) {
    // 1. Weave: insert each node's copy right after it.
    for (Node cur = head; cur != null; cur = cur.next.next) {
        Node copy = new Node(cur.val);
        copy.next = cur.next;
        cur.next = copy;
    }
    // 2. The copy of cur.random is cur.random.next.
    for (Node cur = head; cur != null; cur = cur.next.next) {
        cur.next.random = (cur.random == null) ? null : cur.random.next;
    }
    // 3. Unweave the copies into their own list, restoring the original.
    Node dummy = new Node(0), copyTail = dummy;
    for (Node cur = head; cur != null; cur = cur.next) {
        Node copy = cur.next;
        cur.next = copy.next;          // restore the original link
        copyTail.next = copy;
        copyTail = copy;
    }
    return dummy.next;
}
```

**The reusable idea:** when you need a mapping from old to new, see whether **the structure itself can store it temporarily**. It's the in-place encoding of Ch 5 §5 and Ch 7 §4, applied to references.

---

## 6. Where linked lists break

| Situation | Problem | Instead |
| :--- | :--- | :--- |
| Index access, binary search | O(i) to reach position i | An array / `ArrayList` |
| Scanning speed matters | Scattered nodes defeat the CPU cache | An array |
| A recursive algorithm on a long list | O(n) stack, so `StackOverflowError` near 10⁵ nodes | Iterative pointer loops |
| `java.util.LinkedList` as a general list | `get(i)` is O(i), and every element costs a node | `ArrayList`, or `ArrayDeque` for queues |

---

## 7. Variations *(previews)*

| Change | Adaptation |
| :--- | :--- |
| Reverse only positions `left..right` | Walk to the node before `left`, reverse that segment, reconnect (stretch) |
| Merge k sorted lists | A heap of the k list heads (Ch 15) |
| Remove a node given only that node (not the last one) | Copy the next node's value into it, then bypass the next node |
| A doubly linked list with O(1) removal | The core of the LRU cache (Ch 16) |
| Detect a cycle in a sequence defined by a function | Floyd works on any `x → f(x)` sequence (§3) |

---

## 8. Recognition

**Signals:**
* The input *is* a linked list. Then think: dummy node? reversal? fast/slow? merge?
* "In O(1) extra space" on a list → pointer manipulation instead of copying to an array.
* "Cycle", "loop", "the starting point of a repetition", including in arrays that define a function `i → a[i]` → Floyd.
* "The middle", "the k-th from the end", "palindrome list" → fast/slow (plus reversal of the second half).
* "Deep copy" of a structure with extra references → a map, or weaving.

**False friends:** a problem that merely *stores* data in a sequence. If it needs index access or sorting, copy the values into an array.

---

## 9. Implementation traps

| Trap | Fix |
| :--- | :--- |
| Losing the rest of the list by overwriting `next` too early | Save `next` first (habit 2) |
| Special-casing the head, and getting it wrong | A dummy node (habit 3) |
| `fast.next.next` throwing `NullPointerException` | Loop while `fast != null && fast.next != null` |
| Returning `head` after the head has changed | Return `dummy.next` (or the new head) |
| Leaving a cycle in the result (forgetting `tail.next = null`) | Draw the after-state; check where the last node points |
| Comparing nodes with `.val` when identity is meant (cycle detection) | Compare references with `==` |
| Recursion on long lists | Iterate |

---

## 10. Practice

1. **[Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)** *(Medium)*
2. **[Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists/)** *(Easy)*
3. **[Partition List](https://leetcode.com/problems/partition-list/)** *(Medium)*
4. **[Rotate List](https://leetcode.com/problems/rotate-list/)** *(Medium)*
5. **[Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)** *(Medium)*
6. **[Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)** *(Easy; O(1) extra space)*
7. **[Reorder List](https://leetcode.com/problems/reorder-list/)** *(Medium)*
8. **[Sort List](https://leetcode.com/problems/sort-list/)** *(Medium; O(n log n))*

**Stretch:** **[Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/)** *(Medium)* · **[Flatten a Multilevel Doubly Linked List](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/)** *(Medium)*

<details><summary>Nudges</summary>

1. Remove Nth: a dummy, plus a `fast` pointer that starts n + 1 nodes ahead of `slow`. When `fast` hits `null`, `slow` is just before the node to remove.
2. Intersection: walk A then B with one pointer, and B then A with another. Both travel the same total distance, so they meet at the intersection, or both reach `null` together.
3. Partition: two dummies, for the "less than x" list and the "the rest" list. Join them, and end the second with `null`.
4. Rotate: find the length L and the tail. k %= L. Make the list circular, then cut it L − k nodes from the start.
5. Add Two Numbers: walk both lists with a carry, building the result after a dummy.
6. Palindrome: find the middle (§3), reverse the second half (§2), compare the halves, and (politely) reverse it back.
7. Reorder: middle + reverse the second half + merge the two halves alternately.
8. Sort List: split at the middle, sort each half (Ch 6 contract), merge (§4).
9. Stretch, Reverse II: a dummy; walk to the node before `left`; reverse `right − left + 1` nodes; reconnect both ends.
10. Stretch, Flatten: when a node has a child, splice the flattened child list between the node and its next. A stack (or recursion) handles nesting.
</details>

<details><summary>Key</summary>

Remove Nth → P2 gap pointers + dummy · Intersection → the equal-total-distance argument · Partition → P3 with two dummies · Rotate → length + re-link · Add Two Numbers → P3 with a carry · Palindrome → P2 middle + P1 reversal · Reorder → P2 + P1 + P3 · Sort List → merge sort with a P2 split · Reverse II → P1 on a segment · Flatten → stack / recursion splicing
</details>

---

## 11. Self-test

<details><summary>1. Why does a dummy node simplify insertions and deletions?</summary>

Every real node, including the first, then has a predecessor, so changing the head is the same code as changing any other node.
</details>

<details><summary>2. In Floyd's algorithm, why must the two pointers meet if there's a cycle?</summary>

Once both are in the cycle, `fast` gains exactly one node on `slow` per step, so the gap between them drops by one each step and reaches zero within one lap.
</details>

<details><summary>3. Derive why resetting one pointer to the head finds the cycle's entry.</summary>

When they meet, `slow` has walked `a + b` and `fast` `2(a + b)`. The difference `a + b` is a multiple of the cycle length c, so `a = kc − b`. Walking `a` steps from the meeting point reaches the entry, and so does walking `a` steps from the head.
</details>

<details><summary>4. Why might recursive list reversal crash where the iterative one doesn't?</summary>

It makes one nested call per node, so it needs O(n) stack depth. Java's default stack overflows somewhere around 10⁴–10⁵ frames.
</details>

<details><summary>5. In the weaving clone, what does `cur.random.next` refer to after step 1?</summary>

The copy of `cur.random`, because every original node's copy sits immediately after it.
</details>

---

## 12. Exit gate

Move on when you can:

* build `MyLinkedList` with a dummy node from a blank file;
* write iterative reversal, k-group reversal and Merge Two Sorted Lists, each in under 8 minutes, passing the empty, one-node and two-node cases first time;
* prove Floyd's entry-finding step from the `a`, `b`, `c` picture;
* write the weaving copy and explain each of its three phases;
* solve 7 of the 8 practice problems.
