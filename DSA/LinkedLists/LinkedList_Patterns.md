# Linked Lists – Pattern-Wise DSA Notes

> **Language:** Java | **Level:** Interview-ready | **Focus:** Pattern recognition, pointer manipulation, edge cases

## Pattern Index

| # | Pattern | Key Problems |
|---|---|---|
| 1 | [Core Concepts & Node Setup](#1-core-concepts--node-setup) | Singly, Doubly, Circular LL basics |
| 2 | [Two Pointers (Fast & Slow)](#2-two-pointers-fast--slow) | Cycle detection, Middle, Kth from end |
| 3 | [Reversal](#3-reversal-pattern) | Reverse full list, Reverse sublist, K-group reversal |
| 4 | [Merge & Sort](#4-merge--sort) | Merge two sorted, Merge K sorted, Sort list |
| 5 | [Cycle Detection & Entry](#5-cycle-detection--entry-floyd) | Detect cycle, Find entry point |
| 6 | [Remove / Delete Nodes](#6-remove--delete-nodes) | Remove Nth from end, Remove duplicates, Delete given node |
| 7 | [Intersection & Palindrome](#7-intersection--palindrome) | Intersection of two lists, Palindrome check |
| 8 | [Reorder & Rearrange](#8-reorder--rearrange) | Reorder list, Odd-Even, Rotate list |
| 9 | [Deep Copy & Clone](#9-deep-copy--clone) | Copy list with random pointer |
| 10 | [LRU Cache (Design)](#10-lru-cache-design) | LRU Cache using HashMap + DLL |

---

## 1. Core Concepts & Node Setup

### Node Definitions

```java
// Singly Linked List Node
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

// Doubly Linked List Node
class DoublyNode {
    int val;
    DoublyNode prev, next;
    DoublyNode(int val) { this.val = val; }
}

// Node with random pointer (Clone problem)
class RandomNode {
    int val;
    RandomNode next, random;
    RandomNode(int val) { this.val = val; }
}
```

### Key Properties

```
Singly LL:   each node → next only; tail.next = null
Doubly LL:   each node ↔ prev and next; head.prev = null, tail.next = null
Circular LL: tail.next = head (no null terminator)

Advantages over arrays:
  → O(1) insert/delete at known position (no shifting)
  → Dynamic size (no pre-allocation)

Disadvantages:
  → O(n) access by index (no random access)
  → Extra memory per node (pointer storage)
  → Poor cache locality vs arrays
```

### Utility Methods (Build & Print)

```java
// Build linked list from array
ListNode buildList(int[] vals) {
    ListNode dummy = new ListNode(0);
    ListNode cur = dummy;
    for (int v : vals) { cur.next = new ListNode(v); cur = cur.next; }
    return dummy.next;
}

// Print linked list
void printList(ListNode head) {
    StringBuilder sb = new StringBuilder();
    while (head != null) { sb.append(head.val).append(" -> "); head = head.next; }
    System.out.println(sb.append("null"));
}

// Length of linked list
int length(ListNode head) {
    int len = 0;
    while (head != null) { len++; head = head.next; }
    return len;
}
```

### Dummy Head Pattern (Simplifies Edge Cases)

```java
// ALWAYS use dummy head when the result head could change
// e.g., delete head node, merge sorted lists, reverse
ListNode dummy = new ListNode(0);
dummy.next = head;
ListNode cur = dummy;
// ... manipulate cur.next ...
return dummy.next; // real head of result
```

---

## 2. Two Pointers (Fast & Slow)

**Core idea:** Two pointers move at different speeds through the list.
- Fast moves 2 steps, slow moves 1 step
- When fast reaches end → slow is at middle
- If they meet → cycle exists

---

### Problem 2.1 – Middle of Linked List
**LeetCode 876:**

```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
    // Odd length [1,2,3,4,5] → returns 3 (true middle)
    // Even length [1,2,3,4] → returns 3 (second middle — use slow.next for first)
}
// Time: O(n)  Space: O(1)
```

---

### Problem 2.2 – Kth Node From End
**LeetCode 19 (part):**

```java
// Move fast k steps ahead, then move both until fast hits null
ListNode kthFromEnd(ListNode head, int k) {
    ListNode fast = head, slow = head;
    for (int i = 0; i < k; i++) fast = fast.next; // fast is k ahead
    while (fast != null) { slow = slow.next; fast = fast.next; }
    return slow; // slow is now at kth from end
}
// Time: O(n)  Space: O(1)
```

---

### Problem 2.3 – Remove Nth Node From End
**LeetCode 19:**

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0, head);
    ListNode fast = dummy, slow = dummy;

    for (int i = 0; i <= n; i++) fast = fast.next; // fast is n+1 ahead of slow

    while (fast != null) { slow = slow.next; fast = fast.next; }
    slow.next = slow.next.next; // delete the nth node from end

    return dummy.next;
}
// Dummy head avoids edge case of deleting the actual head node
// Time: O(n)  Space: O(1)
```

---

### Problem 2.4 – Linked List Cycle Detection
**LeetCode 141:**

```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true; // they meet → cycle
    }
    return false;
}
// Floyd's Tortoise and Hare  Time: O(n)  Space: O(1)
// Why they meet: in a cycle of length c, fast gains 1 step per iteration → meets in O(c)
```

---

## 3. Reversal Pattern

**Core iterative reversal (3-pointer technique):**
```
prev → null
cur  → head
next → cur.next (save before breaking link)

Step: cur.next = prev → prev = cur → cur = next
```

```java
ListNode prev = null, cur = head;
while (cur != null) {
    ListNode next = cur.next; // save next
    cur.next = prev;          // reverse link
    prev = cur;               // advance prev
    cur = next;               // advance cur
}
return prev; // new head
```

---

### Problem 3.1 – Reverse Linked List
**LeetCode 206:**

```java
// Iterative
public ListNode reverseList(ListNode head) {
    ListNode prev = null, cur = head;
    while (cur != null) {
        ListNode next = cur.next;
        cur.next = prev;
        prev = cur;
        cur = next;
    }
    return prev;
}

// Recursive
public ListNode reverseListRec(ListNode head) {
    if (head == null || head.next == null) return head; // base case
    ListNode newHead = reverseListRec(head.next); // reverse rest
    head.next.next = head; // make next node point back to head
    head.next = null;      // head is now the tail
    return newHead;
}
// Time: O(n)  Space: O(1) iterative | O(n) recursive stack
```

---

### Problem 3.2 – Reverse Sublist (Between positions left and right)
**LeetCode 92:**

```java
public ListNode reverseBetween(ListNode head, int left, int right) {
    ListNode dummy = new ListNode(0, head);
    ListNode prev = dummy;

    // Step 1: Walk to node just before 'left'
    for (int i = 1; i < left; i++) prev = prev.next;

    // Step 2: Reverse from left to right
    ListNode cur = prev.next;
    for (int i = 0; i < right - left; i++) {
        ListNode next = cur.next;
        cur.next = next.next;       // remove next from list
        next.next = prev.next;      // insert next at front of sublist
        prev.next = next;
    }
    return dummy.next;
}
// "Head-insertion" technique: pull each node to front of reversed section
// Time: O(n)  Space: O(1)
```

---

### Problem 3.3 – Reverse Nodes in K-Group
**LeetCode 25:**

```java
public ListNode reverseKGroup(ListNode head, int k) {
    // Check if k nodes remain
    ListNode check = head;
    for (int i = 0; i < k; i++) {
        if (check == null) return head; // fewer than k remain → don't reverse
        check = check.next;
    }

    // Reverse k nodes
    ListNode prev = null, cur = head;
    for (int i = 0; i < k; i++) {
        ListNode next = cur.next;
        cur.next = prev;
        prev = cur;
        cur = next;
    }
    // head is now tail of reversed group; connect to next group
    head.next = reverseKGroup(cur, k);
    return prev; // prev is new head of this group
}
// Time: O(n)  Space: O(n/k) recursive stack
```

---

## 4. Merge & Sort

### Problem 4.1 – Merge Two Sorted Lists
**LeetCode 21:**

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode cur = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { cur.next = l1; l1 = l1.next; }
        else                  { cur.next = l2; l2 = l2.next; }
        cur = cur.next;
    }
    cur.next = (l1 != null) ? l1 : l2; // attach remaining
    return dummy.next;
}
// Time: O(m + n)  Space: O(1)
```

---

### Problem 4.2 – Merge K Sorted Lists
**LeetCode 23:** Merge k sorted linked lists

```java
// Approach 1: Min-Heap (Priority Queue)
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a.val));
    for (ListNode node : lists)
        if (node != null) pq.offer(node); // add heads

    ListNode dummy = new ListNode(0), cur = dummy;
    while (!pq.isEmpty()) {
        ListNode min = pq.poll();
        cur.next = min;
        cur = cur.next;
        if (min.next != null) pq.offer(min.next); // add next from same list
    }
    return dummy.next;
}
// Time: O(N log k) where N=total nodes, k=number of lists  Space: O(k) heap

// Approach 2: Divide & Conquer (Merge Sort style) — same time complexity
public ListNode mergeKListsDC(ListNode[] lists) {
    if (lists.length == 0) return null;
    return mergeRange(lists, 0, lists.length - 1);
}
ListNode mergeRange(ListNode[] lists, int lo, int hi) {
    if (lo == hi) return lists[lo];
    int mid = lo + (hi - lo) / 2;
    ListNode left  = mergeRange(lists, lo, mid);
    ListNode right = mergeRange(lists, mid + 1, hi);
    return mergeTwoLists(left, right);
}
// Time: O(N log k)  Space: O(log k) recursion stack
```

---

### Problem 4.3 – Sort List (Merge Sort on Linked List)
**LeetCode 148:** Sort linked list in O(n log n) with O(1) space

```java
public ListNode sortList(ListNode head) {
    if (head == null || head.next == null) return head;

    // Step 1: Find middle (slow/fast pointers)
    ListNode mid = getMid(head);
    ListNode right = mid.next;
    mid.next = null; // split into two halves

    // Step 2: Recursively sort each half
    ListNode leftSorted  = sortList(head);
    ListNode rightSorted = sortList(right);

    // Step 3: Merge
    return mergeTwoLists(leftSorted, rightSorted);
}

ListNode getMid(ListNode head) {
    ListNode slow = head, fast = head.next; // fast = head.next → left-biased middle
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
// Time: O(n log n)  Space: O(log n) recursion stack
// KEY: Preferred over QuickSort for linked lists (no random access needed for merge)
```

---

## 5. Cycle Detection & Entry (Floyd's)

### Problem 5.1 – Linked List Cycle II (Find Entry Point)
**LeetCode 142:**

```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;

    // Phase 1: Detect cycle
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) break; // meeting point inside cycle
    }
    if (fast == null || fast.next == null) return null; // no cycle

    // Phase 2: Find cycle entry
    // Math proof: distance from head to entry == distance from meeting point to entry
    slow = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next; // both move at speed 1
    }
    return slow; // cycle entry point
}
// Time: O(n)  Space: O(1)
```

**Why Phase 2 works (proof sketch):**
```
Let:
  F = distance from head to cycle entry
  C = cycle length
  k = distance from entry to meeting point

At meeting:
  Slow traveled: F + k
  Fast traveled: F + k + C (fast did one full extra loop)
  Fast = 2 × Slow → F + k + C = 2(F + k) → C - k = F

So distance from meeting point to entry (going forward) = C - k = F
→ Moving one pointer from head and one from meeting point,
  both at speed 1, they meet exactly at cycle entry.
```

---

## 6. Remove / Delete Nodes

### Problem 6.1 – Remove Duplicates from Sorted List
**LeetCode 83:** Keep one copy of each value

```java
public ListNode deleteDuplicates(ListNode head) {
    ListNode cur = head;
    while (cur != null && cur.next != null) {
        if (cur.val == cur.next.val) cur.next = cur.next.next; // skip duplicate
        else                         cur = cur.next;
    }
    return head;
}
// Time: O(n)  Space: O(1)
```

---

### Problem 6.2 – Remove All Duplicates (Keep None)
**LeetCode 82:** Remove ALL nodes with duplicate values

```java
public ListNode deleteDuplicatesAll(ListNode head) {
    ListNode dummy = new ListNode(0, head);
    ListNode prev = dummy;
    ListNode cur = head;

    while (cur != null) {
        boolean isDup = false;
        while (cur.next != null && cur.val == cur.next.val) {
            cur = cur.next; // skip all same-value nodes
            isDup = true;
        }
        if (isDup) prev.next = cur.next; // skip all with this value
        else       prev = prev.next;     // value is unique; keep it
        cur = cur.next;
    }
    return dummy.next;
}
// Time: O(n)  Space: O(1)
```

---

### Problem 6.3 – Delete Node (Given Only the Node, Not Head)
**LeetCode 237:**

```java
public void deleteNode(ListNode node) {
    // Can't traverse from head; copy next value into current, delete next
    node.val  = node.next.val;
    node.next = node.next.next;
}
// Works because node is guaranteed NOT to be the tail
// Time: O(1)  Space: O(1)
```

---

### Problem 6.4 – Remove Linked List Elements (Remove all val)
**LeetCode 203:**

```java
public ListNode removeElements(ListNode head, int val) {
    ListNode dummy = new ListNode(0, head);
    ListNode cur = dummy;
    while (cur.next != null) {
        if (cur.next.val == val) cur.next = cur.next.next; // skip
        else                     cur = cur.next;
    }
    return dummy.next;
}
// Dummy head handles removal of actual head node  Time: O(n)  Space: O(1)
```

---

## 7. Intersection & Palindrome

### Problem 7.1 – Intersection of Two Linked Lists
**LeetCode 160:** Find the node at which two lists intersect

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode a = headA, b = headB;
    // Both pointers travel lenA + lenB total — they meet at intersection (or null)
    while (a != b) {
        a = (a == null) ? headB : a.next;
        b = (b == null) ? headA : b.next;
    }
    return a;
}
// Key insight: after one full traversal, switch to other list's head
// Total distance each travels: lenA + lenB → aligned at intersection
// If no intersection, both reach null at same time
// Time: O(m + n)  Space: O(1)
```

---

### Problem 7.2 – Palindrome Linked List
**LeetCode 234:**

```java
public boolean isPalindrome(ListNode head) {
    // Step 1: Find middle
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    // Step 2: Reverse second half
    ListNode secondHalf = reverseList(slow);
    ListNode copy = secondHalf; // save to restore (optional)

    // Step 3: Compare both halves
    ListNode first = head;
    while (secondHalf != null) {
        if (first.val != secondHalf.val) return false;
        first = first.next;
        secondHalf = secondHalf.next;
    }
    return true;
    // NOTE: Restore by reversing secondHalf again if needed
}

ListNode reverseList(ListNode head) {
    ListNode prev = null, cur = head;
    while (cur != null) { ListNode next = cur.next; cur.next = prev; prev = cur; cur = next; }
    return prev;
}
// Time: O(n)  Space: O(1)
```

---

## 8. Reorder & Rearrange

### Problem 8.1 – Reorder List
**LeetCode 143:** L0→L1→…→Ln  becomes  L0→Ln→L1→Ln-1→…

```java
public void reorderList(ListNode head) {
    if (head == null || head.next == null) return;

    // Step 1: Find middle
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    // Step 2: Reverse second half
    ListNode second = reverseList(slow.next);
    slow.next = null; // cut first half

    // Step 3: Interleave
    ListNode first = head;
    while (second != null) {
        ListNode tmp1 = first.next, tmp2 = second.next;
        first.next  = second;
        second.next = tmp1;
        first  = tmp1;
        second = tmp2;
    }
}
// Combined: find middle + reverse half + merge  Time: O(n)  Space: O(1)
```

---

### Problem 8.2 – Odd Even Linked List
**LeetCode 328:** Group all odd-indexed nodes, then even-indexed nodes

```java
public ListNode oddEvenList(ListNode head) {
    if (head == null) return null;
    ListNode odd = head, even = head.next, evenHead = even;

    while (even != null && even.next != null) {
        odd.next  = even.next; odd  = odd.next;  // skip to next odd
        even.next = odd.next;  even = even.next; // skip to next even
    }
    odd.next = evenHead; // connect odd list tail to even list head
    return head;
}
// Indices are 1-based: odd = 1,3,5...  even = 2,4,6...
// Time: O(n)  Space: O(1)
```

---

### Problem 8.3 – Rotate List
**LeetCode 61:** Rotate list to the right by k places

```java
public ListNode rotateRight(ListNode head, int k) {
    if (head == null || head.next == null || k == 0) return head;

    // Step 1: Find length and make circular
    ListNode tail = head;
    int len = 1;
    while (tail.next != null) { tail = tail.next; len++; }
    tail.next = head; // make circular

    // Step 2: Find new tail (len - k%len - 1 steps from head)
    int steps = len - k % len - 1;
    ListNode newTail = head;
    for (int i = 0; i < steps; i++) newTail = newTail.next;

    // Step 3: Break circle
    ListNode newHead = newTail.next;
    newTail.next = null;
    return newHead;
}
// k % len handles k > len  Time: O(n)  Space: O(1)
```

---

### Problem 8.4 – Partition List
**LeetCode 86:** All nodes < x before all nodes ≥ x (preserve relative order)

```java
public ListNode partition(ListNode head, int x) {
    ListNode lessHead  = new ListNode(0); // dummy for < x list
    ListNode greaterHead = new ListNode(0); // dummy for >= x list
    ListNode less = lessHead, greater = greaterHead;

    while (head != null) {
        if (head.val < x) { less.next = head;    less    = less.next; }
        else              { greater.next = head; greater = greater.next; }
        head = head.next;
    }
    greater.next = null; // important: avoid cycle from original list
    less.next = greaterHead.next;
    return lessHead.next;
}
// Time: O(n)  Space: O(1)
```

---

## 9. Deep Copy & Clone

### Problem 9.1 – Copy List with Random Pointer
**LeetCode 138:** Clone list where each node has `next` and `random` pointer

```java
// Approach 1: HashMap (O(n) space)
public RandomNode copyRandomList(RandomNode head) {
    if (head == null) return null;
    Map<RandomNode, RandomNode> map = new HashMap<>();

    // Pass 1: create all clone nodes
    RandomNode cur = head;
    while (cur != null) { map.put(cur, new RandomNode(cur.val)); cur = cur.next; }

    // Pass 2: assign next and random pointers
    cur = head;
    while (cur != null) {
        map.get(cur).next   = map.get(cur.next);
        map.get(cur).random = map.get(cur.random);
        cur = cur.next;
    }
    return map.get(head);
}
// Time: O(n)  Space: O(n)

// Approach 2: Interleave (O(1) space)
public RandomNode copyRandomListO1(RandomNode head) {
    if (head == null) return null;

    // Step 1: Interleave clone nodes: A → A' → B → B' → ...
    RandomNode cur = head;
    while (cur != null) {
        RandomNode clone = new RandomNode(cur.val);
        clone.next = cur.next;
        cur.next   = clone;
        cur        = clone.next;
    }

    // Step 2: Set random pointers for clones
    cur = head;
    while (cur != null) {
        if (cur.random != null)
            cur.next.random = cur.random.next; // A'.random = A.random.next = B'
        cur = cur.next.next;
    }

    // Step 3: Separate the two lists
    RandomNode dummy = new RandomNode(0);
    RandomNode clone = dummy;
    cur = head;
    while (cur != null) {
        clone.next = cur.next;
        cur.next   = cur.next.next;
        clone      = clone.next;
        cur        = cur.next;
    }
    return dummy.next;
}
// Time: O(n)  Space: O(1)
```

---

## 10. LRU Cache (Design)

**LeetCode 146:** Implement `get(key)` and `put(key, value)` both in O(1).

**Design:** HashMap (key → DLL node) + Doubly Linked List (order by recency)
- Most recently used: near head
- Least recently used: near tail (evict here)

```java
class LRUCache {
    // Doubly linked list node
    class Node {
        int key, val;
        Node prev, next;
        Node(int k, int v) { key = k; val = v; }
    }

    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    // Sentinel head (MRU side) and tail (LRU side)
    private final Node head = new Node(0, 0);
    private final Node tail = new Node(0, 0);

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        Node node = map.get(key);
        moveToFront(node); // mark as recently used
        return node.val;
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) {
            Node node = map.get(key);
            node.val = value;
            moveToFront(node);
        } else {
            if (map.size() == capacity) {
                Node lru = tail.prev;   // LRU is just before tail
                remove(lru);
                map.remove(lru.key);
            }
            Node newNode = new Node(key, value);
            insertFront(newNode);
            map.put(key, newNode);
        }
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void insertFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }

    private void moveToFront(Node node) {
        remove(node);
        insertFront(node);
    }
}
// get: O(1)  put: O(1)  Space: O(capacity)
// Key: HashMap gives O(1) node lookup; DLL gives O(1) removal/insertion anywhere
```

---

## Edge Cases (Always Check!)

```
1. Empty list:       head == null
2. Single node:      head.next == null
3. Two nodes:        covers many edge cases for fast/slow pointers
4. Cycle exists:     affects traversal (always use visited/fast-slow)
5. k = 0 or k > n:  in rotation, reversal-k-group (mod by length)
6. All duplicates:   [1,1,1,1] → result might be null or single node
7. Even vs Odd len:  affects "middle" definition; test both
8. Removing head:    always use dummy head node to avoid null pointer
```

---

## Pointer Manipulation Cheat Sheet

```
Advance pointer:           cur = cur.next
Save before breaking link: ListNode next = cur.next
Reverse single link:       cur.next = prev
Insert node after prev:    newNode.next = prev.next; prev.next = newNode
Delete node after prev:    prev.next = prev.next.next
Link two lists (A→B):      tail_of_A.next = head_of_B
Make circular:             tail.next = head
Break circular:            node.next = null
Swap values (not nodes):   int tmp = a.val; a.val = b.val; b.val = tmp
```

---

## Pattern Decision Guide

```
Given a linked list problem:
│
├── Need to find middle / split list?           → Fast & Slow pointers
│
├── Need distance from end?                     → Two pointers (gap = k)
│
├── Detect / find entry of cycle?               → Floyd's algorithm
│
├── Reverse part or all of list?                → 3-pointer iterative reversal
│   ├── Reverse full?                           → Standard reverse
│   ├── Reverse between [L, R]?                → Head-insertion in sublist
│   └── Reverse in K-groups?                   → Recurse + reverse k nodes
│
├── Merge / combine lists?                      → Dummy head + merge loop
│   ├── Two sorted?                             → Two-pointer merge
│   └── K sorted?                               → Min-heap or divide & conquer
│
├── Sort linked list?                           → Merge Sort (find mid + merge)
│
├── Remove duplicates / elements?               → Dummy head + skip logic
│
├── Rearrange / reorder?                        → Find mid + reverse + interleave
│
├── Find intersection?                          → Two-pointer with list switching
│
├── Check palindrome?                           → Find mid + reverse half + compare
│
├── Clone with complex pointers?                → HashMap or interleave trick
│
└── Design O(1) get/put with eviction?          → HashMap + Doubly Linked List (LRU)
```

---

## Common Interview Questions

| Question | Answer |
|---|---|
| **Array vs Linked List?** | Array: O(1) random access, O(n) insert/delete (shifting). LL: O(n) access, O(1) insert/delete at known node. Arrays better for cache, LL for frequent middle insertions. |
| **Why dummy head node?** | Avoids special-casing deletion/insertion at head. Result always `dummy.next` regardless of edge cases. |
| **How to reverse a LL in O(1) space?** | 3-pointer iterative: `prev, cur, next`. Advance all three, reverse `cur.next = prev` at each step. |
| **Fast/slow pointer — why does it work?** | Fast gains 1 step per iteration over slow. In a cycle of length C, they meet after at most C iterations inside the cycle. |
| **Merge sort vs quicksort for LL?** | Merge sort preferred: splitting at middle is O(n), merging is O(1) space by relinking. QuickSort needs random access for efficient pivot selection. |
| **How to detect cycle without extra space?** | Floyd's: fast pointer (2 steps) meets slow pointer (1 step) inside cycle → O(1) space. |
| **Delete a node given only that node?** | Copy value from next node into current, delete next node. Cannot delete true tail this way. |
| **How does LRU cache achieve O(1) both operations?** | HashMap: O(1) key-to-node lookup. DLL: O(1) insert/remove with direct node reference. Together: all ops O(1). |

---

*DSA Linked Lists Notes | Pattern-wise | Java | Updated: April 2026*
