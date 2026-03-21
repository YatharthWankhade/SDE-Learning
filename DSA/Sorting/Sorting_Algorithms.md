# Sorting Algorithms – DSA Notes

> **Language:** Java | **Level:** Interview-ready | **Focus:** How it works, complexity, code, when to use, interview traps

## Algorithm Index

| Algorithm | Best | Average | Worst | Space | Stable? | Notes |
|---|---|---|---|---|---|---|
| [Bubble Sort](#1-bubble-sort) | O(n) | O(n²) | O(n²) | O(1) | ✅ | Educational only |
| [Selection Sort](#2-selection-sort) | O(n²) | O(n²) | O(n²) | O(1) | ❌ | Min swaps |
| [Insertion Sort](#3-insertion-sort) | O(n) | O(n²) | O(n²) | O(1) | ✅ | Best for nearly sorted |
| [Merge Sort](#4-merge-sort) | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ | Gold standard, divide & conquer |
| [Quick Sort](#5-quick-sort) | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ | Fastest in practice (cache) |
| [Heap Sort](#6-heap-sort) | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | In-place, guaranteed O(n log n) |
| [Counting Sort](#7-counting-sort) | O(n+k) | O(n+k) | O(n+k) | O(k) | ✅ | Non-comparison; range limited |
| [Radix Sort](#8-radix-sort) | O(nk) | O(nk) | O(nk) | O(n+k) | ✅ | Digit by digit stable sort |
| [Bucket Sort](#9-bucket-sort) | O(n+k) | O(n+k) | O(n²) | O(n) | ✅ | Uniform distribution |
| [Tim Sort](#10-timsort--javas-built-in) | O(n) | O(n log n) | O(n log n) | O(n) | ✅ | Java's Arrays.sort() for objects |

---

## Core Concepts

### Stability
A sort is **stable** if equal elements maintain their original relative order.
```
Input:  [(Alice, 30), (Bob, 25), (Charlie, 30)]
Stable sort by age:  [(Bob, 25), (Alice, 30), (Charlie, 30)]  ← Alice before Charlie
Unstable sort by age could give: [(Bob, 25), (Charlie, 30), (Alice, 30)]
```
**Matters when:** sorting objects by a secondary key that should preserve primary key order.

### Comparison vs Non-Comparison
- **Comparison sorts**: must compare elements → theoretical lower bound **O(n log n)**
- **Non-comparison sorts** (Counting, Radix, Bucket): exploit structure of data → can be O(n)

### In-Place vs Out-of-Place
- **In-place:** O(1) extra space (Bubble, Selection, Insertion, Heap, Quick)
- **Out-of-place:** allocates extra memory (Merge Sort: O(n))

---

## 1. Bubble Sort

**How it works:** Repeatedly swap adjacent elements if they're in the wrong order. Largest element "bubbles" to the end each pass.

```
Pass 1: [5, 3, 8, 1] → [3, 5, 1, 8]   (8 in final position)
Pass 2: [3, 5, 1, 8] → [3, 1, 5, 8]   (5 in final position)
Pass 3: [3, 1, 5, 8] → [1, 3, 5, 8]   done
```

```java
public void bubbleSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false; // optimization: early exit if already sorted
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int tmp = arr[j]; arr[j] = arr[j + 1]; arr[j + 1] = tmp;
                swapped = true;
            }
        }
        if (!swapped) break; // already sorted → O(n) best case
    }
}
// Best: O(n) – already sorted with early exit
// Avg/Worst: O(n²)  Space: O(1)  Stable: YES
```

**When to use:** Never in production. Useful to explain stability and early exit optimization in interviews.

**Interview trap:** "Why is it O(n²) even though inner loop shrinks?" → Total comparisons = (n-1) + (n-2) + ... + 1 = n(n-1)/2 = O(n²).

---

## 2. Selection Sort

**How it works:** Find the minimum element in the unsorted portion, swap it to the front. Repeat.

```
[5, 3, 8, 1] → find min(1), swap with index 0 → [1, 3, 8, 5]
[1, 3, 8, 5] → find min(3), already at index 1  → [1, 3, 8, 5]
[1, 3, 8, 5] → find min(5), swap with index 2  → [1, 3, 5, 8]
```

```java
public void selectionSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) minIdx = j;
        }
        int tmp = arr[i]; arr[i] = arr[minIdx]; arr[minIdx] = tmp;
    }
}
// Best/Avg/Worst: O(n²)  Space: O(1)  Stable: NO (long-range swaps break stability)
```

**Key property:** Makes at most **n-1 swaps** (minimum possible). Useful when write cost is very high (e.g., flash memory).

**Interview trap:** "Is Selection Sort stable?" → **No**. Classic counter-example: `[3a, 3b, 1]` → swaps `3a` with `1` → `[1, 3b, 3a]` — `3a` and `3b` order reversed.

---

## 3. Insertion Sort

**How it works:** Build a sorted portion left to right. For each new element, insert it into the correct position in the sorted portion by shifting elements right.

```
[5, 3, 8, 1]
i=1: key=3 → shift 5 right → [3, 5, 8, 1]
i=2: key=8 → no shift needed → [3, 5, 8, 1]
i=3: key=1 → shift 8,5,3 right → [1, 3, 5, 8]
```

```java
public void insertionSort(int[] arr) {
    for (int i = 1; i < arr.length; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j]; // shift right (no swap needed)
            j--;
        }
        arr[j + 1] = key; // insert in correct position
    }
}
// Best: O(n) – nearly sorted  Worst: O(n²)  Space: O(1)  Stable: YES
```

**When to use:**
- Nearly sorted arrays (only k swaps away → O(nk))
- Small subarrays (n < 10–20): low overhead beats O(n log n) algorithms
- **TimSort** (Java's built-in) uses insertion sort for small runs (< 32 elements)

**Interview trap:** "How many comparisons vs swaps?" → Swaps = O(n²), but insertion sort does **shifts** (not full swaps), making it faster in practice than Bubble Sort.

---

## 4. Merge Sort

**How it works:** Divide array in half recursively until size 1, then **merge** two sorted halves. Classic divide-and-conquer.

```
      [5, 3, 8, 1]
     /             \
  [5, 3]         [8, 1]
  /    \          /    \
[5]   [3]      [8]    [1]
  \    /          \    /
  [3, 5]         [1, 8]
     \               /
       [1, 3, 5, 8]
```

```java
public void mergeSort(int[] arr, int left, int right) {
    if (left >= right) return; // base case: single element
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}

private void merge(int[] arr, int left, int mid, int right) {
    // Copy both halves to temp arrays
    int[] L = Arrays.copyOfRange(arr, left, mid + 1);
    int[] R = Arrays.copyOfRange(arr, mid + 1, right + 1);

    int i = 0, j = 0, k = left;
    while (i < L.length && j < R.length) {
        if (L[i] <= R[j]) arr[k++] = L[i++]; // <= ensures stability
        else               arr[k++] = R[j++];
    }
    while (i < L.length) arr[k++] = L[i++];
    while (j < R.length) arr[k++] = R[j++];
}

// Call: mergeSort(arr, 0, arr.length - 1)
// Time: O(n log n) all cases  Space: O(n)  Stable: YES
```

### Merge Sort – Key Interview Points

```
Recurrence: T(n) = 2T(n/2) + O(n)  → O(n log n) by Master Theorem

Why Stable?
  When merging, we prefer LEFT array on ties (L[i] <= R[j])
  → equal elements from left half always come first

Why O(n) Space?
  Each merge call creates temp arrays of total size n
  Only one merge active at each recursion level → O(n) extra

Advantages:
  → Guaranteed O(n log n) even for worst case
  → Stable sort
  → Preferred for Linked Lists (no random access needed; O(1) merge by relinking)

Disadvantages:
  → O(n) extra memory
  → More cache misses than quicksort (non-local memory access)
  → Slower in practice than quicksort for arrays due to memory allocation overhead
```

### Problem: Count Inversions (Merge Sort Application)
An **inversion** is a pair `(i,j)` where `i < j` but `arr[i] > arr[j]`. Count during merge step.

```java
public long countInversions(int[] arr, int left, int right) {
    if (left >= right) return 0;
    long inv = 0;
    int mid = left + (right - left) / 2;
    inv += countInversions(arr, left, mid);
    inv += countInversions(arr, mid + 1, right);
    inv += mergeCount(arr, left, mid, right);
    return inv;
}

private long mergeCount(int[] arr, int left, int mid, int right) {
    int[] L = Arrays.copyOfRange(arr, left, mid + 1);
    int[] R = Arrays.copyOfRange(arr, mid + 1, right + 1);
    int i = 0, j = 0, k = left;
    long inv = 0;
    while (i < L.length && j < R.length) {
        if (L[i] <= R[j]) {
            arr[k++] = L[i++];
        } else {
            // L[i] > R[j]: ALL remaining elements in L are > R[j] (sorted)
            inv += (L.length - i); // count of inversions
            arr[k++] = R[j++];
        }
    }
    while (i < L.length) arr[k++] = L[i++];
    while (j < R.length) arr[k++] = R[j++];
    return inv;
}
// LeetCode 315 (Count Smaller After Self) uses same idea
// Time: O(n log n)  Space: O(n)
```

---

## 5. Quick Sort

**How it works:** Pick a **pivot**, partition array so all elements < pivot go left, all > pivot go right. Recursively sort the two partitions.

```
arr = [3, 6, 8, 10, 1, 2, 1], pivot = arr[last] = 1
Partition: [1, 1 | 1 | 8, 10, 6, 3, 2]  ← pivot at correct index
Recurse left: [1, 1]
Recurse right: [8, 10, 6, 3, 2] → further partition...
```

```java
public void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pivotIdx = partition(arr, low, high);
        quickSort(arr, low, pivotIdx - 1);
        quickSort(arr, pivotIdx + 1, high);
    }
}

// Lomuto Partition Scheme
private int partition(int[] arr, int low, int high) {
    int pivot = arr[high]; // choose last element as pivot
    int i = low - 1;       // i = boundary of elements < pivot
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            int tmp = arr[i]; arr[i] = arr[j]; arr[j] = tmp;
        }
    }
    // Place pivot in correct position
    int tmp = arr[i + 1]; arr[i + 1] = arr[high]; arr[high] = tmp;
    return i + 1;
}
// Call: quickSort(arr, 0, arr.length - 1)
```

### Hoare Partition (Faster in Practice)

```java
// Hoare: two pointers moving toward each other
private int hoarePartition(int[] arr, int low, int high) {
    int pivot = arr[low + (high - low) / 2]; // middle element
    int i = low - 1, j = high + 1;
    while (true) {
        do { i++; } while (arr[i] < pivot);
        do { j--; } while (arr[j] > pivot);
        if (i >= j) return j;
        int tmp = arr[i]; arr[i] = arr[j]; arr[j] = tmp;
    }
}
```

### Randomized QuickSort (Avoid O(n²) Worst Case)

```java
private final Random rand = new Random();

public void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        // Random pivot: swap random element with last
        int randIdx = low + rand.nextInt(high - low + 1);
        int tmp = arr[randIdx]; arr[randIdx] = arr[high]; arr[high] = tmp;

        int pivotIdx = partition(arr, low, high);
        quickSort(arr, low, pivotIdx - 1);
        quickSort(arr, pivotIdx + 1, high);
    }
}
// Expected O(n log n) with randomization
// Space: O(log n) average stack depth; O(n) worst case (can use tail-call optimization)
```

### 3-Way QuickSort (Dutch National Flag – handles duplicates)

```java
// Best for arrays with many duplicate elements
public void quickSort3Way(int[] arr, int low, int high) {
    if (low >= high) return;
    int pivot = arr[low];
    int lt = low, gt = high, i = low + 1;
    while (i <= gt) {
        if      (arr[i] < pivot) swap(arr, lt++, i++);
        else if (arr[i] > pivot) swap(arr, i, gt--);
        else                     i++;
    }
    // arr[low..lt-1] < pivot, arr[lt..gt] == pivot, arr[gt+1..high] > pivot
    quickSort3Way(arr, low, lt - 1);
    quickSort3Way(arr, gt + 1, high);
}
// O(n log n) average; O(n) if all elements equal; used in Java's Arrays.sort for primitives
```

### QuickSort – Key Interview Points

```
Why faster than Merge Sort in practice?
  → Better cache locality (works in-place, no memory allocation)
  → Smaller constant factors

Worst case O(n²):
  → Occurs when pivot always picks min or max (already sorted array + always pick first/last)
  → Fix: random pivot or median-of-three pivot

Not Stable:
  → Long-range swaps break relative order of equal elements

Stack overflow risk:
  → Worst-case recursion depth O(n) (degenerate pivot)
  → Fix: always recurse on smaller partition first (tail call optimization)

Java uses:
  → Dual-Pivot QuickSort (by Vladimir Yaroslavskiy) for primitive arrays
  → TimSort for object arrays (stable required)
```

---

## 6. Heap Sort

**How it works:** Build a max-heap, then repeatedly extract the max (swap root with last, reduce heap size, heapify).

```
Phase 1 (Build Max-Heap):  [1,3,5,8,2] → [8,3,5,1,2]
Phase 2 (Extract max):
  Swap root(8) with last → [2,3,5,1,8] → heapify → [5,3,2,1,8]
  Swap root(5) with last → [1,3,2,5,8] → heapify → [3,1,2,5,8]
  ... → [1,2,3,5,8]
```

```java
public void heapSort(int[] arr) {
    int n = arr.length;
    // Phase 1: Build max-heap (start from last non-leaf node)
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);

    // Phase 2: Extract elements from heap one by one
    for (int i = n - 1; i > 0; i--) {
        // Move current root (max) to end
        int tmp = arr[0]; arr[0] = arr[i]; arr[i] = tmp;
        heapify(arr, i, 0); // restore heap property for reduced heap
    }
}

// Sift down: ensure subtree rooted at index i is a max-heap
private void heapify(int[] arr, int n, int i) {
    int largest = i;
    int left    = 2 * i + 1;
    int right   = 2 * i + 2;

    if (left  < n && arr[left]  > arr[largest]) largest = left;
    if (right < n && arr[right] > arr[largest]) largest = right;

    if (largest != i) {
        int tmp = arr[i]; arr[i] = arr[largest]; arr[largest] = tmp;
        heapify(arr, n, largest); // recursively fix affected subtree
    }
}
// Time: O(n log n) always  Space: O(1)  Stable: NO
// Build heap: O(n) — not O(n log n)! (mathematical proof via geometric series)
```

### Heap Sort – Key Points

```
Build Heap is O(n) NOT O(n log n):
  → Nodes at height h take O(h) to heapify
  → Sum across all levels = O(n) (each level halves)
  → See: Σ(n/2^k * k) for k=1..logn = O(n)

Why rarely used in practice despite O(n log n) guaranteed?
  → Poor cache performance: heap access jumps around memory
  → Hidden constants larger than QuickSort
  → Not stable

When to use:
  → Memory-constrained; need guaranteed O(n log n) in-place
  → Building priority queues (core data structure application)

Heap facts (interview):
  → Min-heap: parent ≤ children
  → Max-heap: parent ≥ children
  → Left child of i: 2i+1   Right child: 2i+2   Parent of i: (i-1)/2
  → Insert: add at end, sift up → O(log n)
  → Extract min/max: swap root with last, sift down → O(log n)
  → Build from array: O(n) using bottom-up heapify
```

---

## 7. Counting Sort

**How it works:** Count occurrences of each value, use prefix sums to find correct positions.

```
Input: [4, 2, 2, 8, 3, 3, 1]  range: [1..8]
Count: [0, 1, 2, 2, 1, 0, 0, 0, 1]  (index = value)
Prefix:[0, 1, 3, 5, 6, 6, 6, 6, 7]  (cumulative)
Place each element at prefix[val]-1, decrement prefix[val]
Output:[1, 2, 2, 3, 3, 4, 8]
```

```java
public int[] countingSort(int[] arr) {
    if (arr.length == 0) return arr;
    int max = Arrays.stream(arr).max().getAsInt();
    int min = Arrays.stream(arr).min().getAsInt();
    int range = max - min + 1;

    int[] count = new int[range];
    for (int num : arr) count[num - min]++;       // count frequencies

    // Build prefix sum
    for (int i = 1; i < range; i++) count[i] += count[i - 1];

    // Place elements in output (iterate BACKWARD for stability)
    int[] output = new int[arr.length];
    for (int i = arr.length - 1; i >= 0; i--) {
        output[--count[arr[i] - min]] = arr[i];
    }
    return output;
}
// Time: O(n + k)  where k = range of values
// Space: O(n + k)  Stable: YES (backward iteration preserves order)
```

**When to use:**
- Integer keys in a **small known range** (e.g., sort by age 0–150, sort exam scores 0–100)
- First step in Radix Sort

**When NOT to use:**
- Large value range (k >> n → wastes space)
- Floating-point numbers
- Negative numbers without offset adjustment

---

## 8. Radix Sort

**How it works:** Sort digit by digit from least significant to most significant, using a stable sort (Counting Sort) at each digit position.

```
Original:  [170, 45, 75, 90, 802, 24, 2, 66]

Sort by 1s digit:  [170, 90, 802, 2, 24, 45, 75, 66]
Sort by 10s digit: [802, 2, 24, 45, 66, 170, 75, 90]
Sort by 100s digit:[2, 24, 45, 66, 75, 90, 170, 802]
```

```java
public void radixSort(int[] arr) {
    int max = Arrays.stream(arr).max().getAsInt();
    // Sort by each digit position (1s, 10s, 100s, ...)
    for (int exp = 1; max / exp > 0; exp *= 10)
        countingSortByDigit(arr, exp);
}

private void countingSortByDigit(int[] arr, int exp) {
    int n = arr.length;
    int[] output = new int[n];
    int[] count = new int[10]; // digits 0–9

    // Count occurrences of each digit
    for (int num : arr) count[(num / exp) % 10]++;

    // Prefix sum → positions
    for (int i = 1; i < 10; i++) count[i] += count[i - 1];

    // Build output backward (stability)
    for (int i = n - 1; i >= 0; i--) {
        int digit = (arr[i] / exp) % 10;
        output[--count[digit]] = arr[i];
    }
    System.arraycopy(output, 0, arr, 0, n);
}
// Time: O(d × (n + b)) where d=digits, b=base(10)  → effectively O(n) for fixed-width int
// Space: O(n + b)  Stable: YES
```

**When to use:**
- Fixed max number of digits (phone numbers, ZIP codes, fixed-length IDs)
- Large n with small d (e.g., sort a million 32-bit integers → d=10)

**When NOT to use:**
- Variable-length strings with large alphabet
- When comparisons are cheap but memory is constrained

---

## 9. Bucket Sort

**How it works:** Distribute elements into buckets (ranges), sort each bucket (usually Insertion Sort), concatenate.

```
Input: [0.78, 0.17, 0.39, 0.26, 0.72, 0.94, 0.21, 0.12]
n = 8 buckets, each covers [i/n, (i+1)/n)

Bucket 0: [0.12, 0.17]   → sorted
Bucket 1: [0.21, 0.26]   → sorted
Bucket 3: [0.39]
Bucket 7: [0.72, 0.78]
Bucket 9: [0.94]

Concatenate → [0.12, 0.17, 0.21, 0.26, 0.39, 0.72, 0.78, 0.94]
```

```java
public void bucketSort(float[] arr) {
    int n = arr.length;
    List<Float>[] buckets = new List[n];
    for (int i = 0; i < n; i++) buckets[i] = new ArrayList<>();

    // Distribute elements into buckets
    for (float num : arr) {
        int idx = (int)(num * n); // assumes input in [0, 1)
        buckets[Math.min(idx, n - 1)].add(num);
    }

    // Sort each bucket (insertion sort is efficient for small buckets)
    int k = 0;
    for (List<Float> bucket : buckets) {
        Collections.sort(bucket);
        for (float val : bucket) arr[k++] = val;
    }
}
// Time: O(n + k) average (uniform distribution)  Worst: O(n²) if all in one bucket
// Space: O(n + k)  Stable: YES (if stable sort used per bucket)
```

**When to use:**
- Uniformly distributed floating point numbers in `[0, 1)`
- Sorting by a score/ratio where distribution is known to be uniform

---

## 10. TimSort – Java's Built-In

```
TimSort = Merge Sort + Insertion Sort hybrid
Used in:
  - Java: Arrays.sort(Object[]), Collections.sort() → TimSort (stable)
  - Java: Arrays.sort(int[])                        → Dual-Pivot QuickSort (not stable)
  - Python: sorted() / list.sort()                   → TimSort

How it works:
  1. Scan array for natural "runs" (already sorted sequences)
  2. If a run is too short (< minRun ≈ 32–64), extend with insertion sort
  3. Push runs onto a stack
  4. Merge adjacent runs when stack invariant is violated

Why O(n) on nearly-sorted input:
  → Finds long natural runs → O(n/run_length) merges
  → If input already sorted: 1 run → 0 merges → O(n)

Time: O(n log n) worst, O(n) best  Space: O(n)  Stable: YES
```

```java
// In Java – just use built-in (TimSort for objects, Dual-Pivot QS for primitives)
int[] primitiveArr = {5, 3, 8, 1};
Arrays.sort(primitiveArr);          // Dual-Pivot QuickSort (not stable, fast)

Integer[] objectArr = {5, 3, 8, 1};
Arrays.sort(objectArr);             // TimSort (stable)

// Custom comparator (always TimSort for objects)
Arrays.sort(objectArr, Comparator.reverseOrder());

// Sort with lambda
String[] names = {"Charlie", "Alice", "Bob"};
Arrays.sort(names, (a, b) -> a.compareTo(b));

// Sort subarray
Arrays.sort(primitiveArr, 1, 3); // sort indices [1, 3) only

// Sort 2D array by first column, then second column
int[][] matrix = {{3,1},{1,2},{1,1}};
Arrays.sort(matrix, (a, b) -> a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]);
```

---

## Sorting in Interview Problems

### Problem: Sort Array by Parity
**LeetCode 905** | Even elements before odd (relative order within does not matter)

```java
public int[] sortArrayByParity(int[] nums) {
    int l = 0, r = nums.length - 1;
    while (l < r) {
        while (l < r && nums[l] % 2 == 0) l++;  // skip evens on left
        while (l < r && nums[r] % 2 != 0) r--;  // skip odds on right
        if (l < r) { int t = nums[l]; nums[l++] = nums[r]; nums[r--] = t; }
    }
    return nums;
}
// Two-pointer partition (Dutch Flag style)  Time: O(n)  Space: O(1)
```

---

### Problem: Relative Sort Array
**LeetCode 1122** | Sort arr1 by order defined in arr2; remaining elements ascending at end

```java
public int[] relativeSortArray(int[] arr1, int[] arr2) {
    Map<Integer, Integer> order = new HashMap<>();
    for (int i = 0; i < arr2.length; i++) order.put(arr2[i], i);

    Integer[] boxed = Arrays.stream(arr1).boxed().toArray(Integer[]::new);
    Arrays.sort(boxed, (a, b) -> {
        boolean aIn = order.containsKey(a), bIn = order.containsKey(b);
        if (aIn && bIn)  return order.get(a) - order.get(b);
        if (aIn)         return -1; // a comes first
        if (bIn)         return 1;  // b comes first
        return a - b;               // both not in arr2 → ascending
    });
    return Arrays.stream(boxed).mapToInt(Integer::intValue).toArray();
}
// Time: O(n log n)  Space: O(n)
```

---

### Problem: Largest Number
**LeetCode 179** | Arrange integers to form largest number

```java
public String largestNumber(int[] nums) {
    String[] strs = new String[nums.length];
    for (int i = 0; i < nums.length; i++) strs[i] = String.valueOf(nums[i]);

    // Custom comparator: prefer order that gives larger concatenation
    Arrays.sort(strs, (a, b) -> (b + a).compareTo(a + b));

    if (strs[0].equals("0")) return "0"; // edge case: all zeros
    return String.join("", strs);
}
// Key insight: compare "ab" vs "ba" as strings. If "ba" > "ab" → b should come first
// Time: O(n log n × L)  where L = avg number length  Space: O(n)
```

---

### Problem: Meeting Rooms II (Sort + MinHeap)
**LeetCode 253** | Minimum number of conference rooms required

```java
public int minMeetingRooms(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // sort by start time
    PriorityQueue<Integer> endTimes = new PriorityQueue<>(); // min-heap of end times

    for (int[] interval : intervals) {
        // If a room freed up before this meeting starts, reuse it
        if (!endTimes.isEmpty() && endTimes.peek() <= interval[0])
            endTimes.poll();
        endTimes.offer(interval[1]); // assign room (or new room if no reuse)
    }
    return endTimes.size(); // active rooms = number without freed-up end times
}
// Time: O(n log n)  Space: O(n)
```

---

### Problem: K Closest Points to Origin (QuickSelect – O(n) average)
**LeetCode 973** | Don't need full sort; find k closest points

```java
public int[][] kClosest(int[][] points, int k) {
    quickSelect(points, 0, points.length - 1, k);
    return Arrays.copyOf(points, k);
}

private void quickSelect(int[][] pts, int lo, int hi, int k) {
    if (lo >= hi) return;
    int pivotIdx = partition(pts, lo, hi);
    if (pivotIdx == k - 1) return;         // exactly k on left
    else if (pivotIdx < k - 1) quickSelect(pts, pivotIdx + 1, hi, k);
    else                       quickSelect(pts, lo, pivotIdx - 1, k);
}

private int partition(int[][] pts, int lo, int hi) {
    long pivotDist = dist(pts[hi]);
    int i = lo;
    for (int j = lo; j < hi; j++) {
        if (dist(pts[j]) <= pivotDist) {
            int[] tmp = pts[i]; pts[i] = pts[j]; pts[j] = tmp; i++;
        }
    }
    int[] tmp = pts[i]; pts[i] = pts[hi]; pts[hi] = tmp;
    return i;
}

private long dist(int[] p) { return (long)p[0]*p[0] + (long)p[1]*p[1]; }
// QuickSelect: O(n) average, O(n²) worst  Space: O(log n) stack
// Alternative: sort by distance O(n log n), or min-heap O(n log k)
```

---

## Algorithm Comparison & Decision Guide

```
Need Stable Sort?
  YES → Merge Sort, TimSort, Counting Sort, Radix Sort
  NO  → QuickSort, HeapSort, Selection Sort also available

Memory Constraint? (O(1) space required)
  → HeapSort (guaranteed O(n log n), in-place)
  → QuickSort (O(log n) stack; not strictly O(1) but close)

Nearly Sorted Input?
  → Insertion Sort (O(nk) for k-away sorted)
  → TimSort (exploits natural runs)

Small Array (n < 20)?
  → Insertion Sort (low overhead beats O(n log n) for small n)

Integer Data in Small Range?
  → Counting Sort (O(n + k))

Fixed-Width Integer Keys (e.g., 32-bit int)?
  → Radix Sort (O(n))

Uniformly Distributed Floats?
  → Bucket Sort (O(n) average)

Find K-th element without full sort?
  → QuickSelect (O(n) average)

Default recommendation for general-purpose sorting in Java?
  → Arrays.sort() — let JDK pick (DualPivotQS for primitives, TimSort for objects)
```

---

## Full Complexity Reference

| Algorithm | Best | Average | Worst | Space | Stable | In-Place | Notes |
|---|---|---|---|---|---|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ | w/ early exit flag |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | ❌ | ✅ | Min swaps (n-1) |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ | Great for small/nearly-sorted |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ | ❌ | Best for linked lists |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ | ✅ | Fastest in practice |
| Quick Sort 3-Way | O(n) | O(n log n) | O(n²) | O(log n) | ❌ | ✅ | Best for many duplicates |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | ✅ | Cache-unfriendly |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | ✅ | ❌ | Range must be small |
| Radix Sort | O(nk) | O(nk) | O(nk) | O(n+b) | ✅ | ❌ | d digits, base b |
| Bucket Sort | O(n+k) | O(n+k) | O(n²) | O(n) | ✅ | ❌ | Uniform distribution |
| TimSort | O(n) | O(n log n) | O(n log n) | O(n) | ✅ | ❌ | Java default (objects) |

---

## Common Interview Questions

| Question | Answer |
|---|---|
| **Lower bound for comparison-based sort?** | **Ω(n log n)** — decision tree has n! leaves; height ≥ log₂(n!) ≈ n log n (Stirling) |
| **Why is QuickSort preferred over MergeSort?** | Better cache locality (in-place), smaller constants, no memory allocation |
| **When does QuickSort degrade to O(n²)?** | Already sorted/reverse-sorted array with bad pivot choice (first/last element) |
| **How does Java sort primitives vs objects?** | Primitives: Dual-Pivot QuickSort (fast, not stable). Objects: TimSort (stable, required for Comparator contracts) |
| **Why is Merge Sort preferred for Linked Lists?** | No random access needed; merging two linked lists is O(1) extra space by relinking pointers |
| **What is QuickSelect?** | Partitions like QuickSort but only recurses on one side → O(n) average to find k-th element |
| **3-Way QuickSort vs standard?** | 3-way handles duplicates efficiently → O(n) when all elements equal vs O(n²) for standard |
| **Stable sort importance example?** | Sorting students by grade then by name: stable sort preserves name order within same grade |
| **Can you sort in O(n)?** | Yes with non-comparison sorts (Counting, Radix, Bucket) — only if input has special structure |
| **Counting sort on negative numbers?** | Offset by min value: `count[num - min]++`; range = max - min + 1 |

---

*DSA Sorting Notes | Java | Updated: March 2026*
