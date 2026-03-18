# Arrays – Pattern-Wise DSA Notes

> **Language:** Java | **Level:** Interview-ready (L1–L3 patterns)

## Pattern Index

| # | Pattern | Time | Space | Key Problems |
|---|---|---|---|---|
| 1 | [Two Pointers](#1-two-pointers) | O(n) | O(1) | Pair Sum, Container with Water, 3Sum |
| 2 | [Sliding Window](#2-sliding-window) | O(n) | O(1)/O(k) | Max Subarray, Longest Substring |
| 3 | [Prefix Sum](#3-prefix-sum) | O(n) | O(n) | Range Sum, Subarray Sum Equals K |
| 4 | [Binary Search on Array](#4-binary-search-on-array) | O(log n) | O(1) | Search Rotated, Peak Element |
| 5 | [Kadane's Algorithm](#5-kadanes-algorithm) | O(n) | O(1) | Max Subarray, Max Circular Subarray |
| 6 | [Dutch National Flag / 3-way Partition](#6-dutch-national-flag--3-way-partition) | O(n) | O(1) | Sort Colors, Move Zeros |
| 7 | [Merge Intervals](#7-merge-intervals) | O(n log n) | O(n) | Merge Intervals, Insert Interval |
| 8 | [Cyclic Sort](#8-cyclic-sort) | O(n) | O(1) | Missing Number, Find Duplicates |
| 9 | [Matrix Traversal (2D Arrays)](#9-matrix-traversal-2d-arrays) | O(m×n) | O(1) | Spiral, Rotate, Set Zeroes |
| 10 | [Monotonic Stack on Array](#10-monotonic-stack-on-array) | O(n) | O(n) | Next Greater, Largest Rectangle |
| 11 | [Counting / Frequency Map](#11-counting--frequency-map) | O(n) | O(n) | Top K Elements, Majority Element |
| 12 | [Subarray / Subsequence Tricks](#12-subarray--subsequence-tricks) | O(n) | O(1) | Product Except Self, Buy/Sell Stock |

---

## 1. Two Pointers

**When to use:**
- Sorted array; find pair/triplet with target sum
- "Container" style: two boundaries shrinking toward each other
- Partitioning in-place

**Template:**
```java
int left = 0, right = arr.length - 1;
while (left < right) {
    int sum = arr[left] + arr[right];
    if (sum == target)      { /* found */ left++; right--; }
    else if (sum < target)  left++;
    else                    right--;
}
```

---

### Problem 1.1 – Two Sum II (Sorted Array)
**LeetCode 167** | Input sorted, find indices where `arr[i] + arr[j] == target`

```java
public int[] twoSum(int[] numbers, int target) {
    int l = 0, r = numbers.length - 1;
    while (l < r) {
        int sum = numbers[l] + numbers[r];
        if      (sum == target) return new int[]{l + 1, r + 1}; // 1-indexed
        else if (sum < target)  l++;
        else                    r--;
    }
    return new int[]{-1, -1};
}
// Time: O(n)  Space: O(1)
```

---

### Problem 1.2 – Container With Most Water
**LeetCode 11** | Find two lines forming max water container

```java
public int maxArea(int[] height) {
    int l = 0, r = height.length - 1, maxW = 0;
    while (l < r) {
        int water = Math.min(height[l], height[r]) * (r - l);
        maxW = Math.max(maxW, water);
        // move the shorter pointer (taller can only help if we reduce width)
        if (height[l] < height[r]) l++;
        else                       r--;
    }
    return maxW;
}
// Time: O(n)  Space: O(1)
```

---

### Problem 1.3 – 3Sum
**LeetCode 15** | Find all unique triplets summing to 0

```java
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> res = new ArrayList<>();
    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue; // skip duplicates
        int l = i + 1, r = nums.length - 1;
        while (l < r) {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum == 0) {
                res.add(List.of(nums[i], nums[l], nums[r]));
                while (l < r && nums[l] == nums[l + 1]) l++; // skip dup
                while (l < r && nums[r] == nums[r - 1]) r--; // skip dup
                l++; r--;
            } else if (sum < 0) l++;
            else                r--;
        }
    }
    return res;
}
// Time: O(n²)  Space: O(1) [ignoring output]
```

---

### Problem 1.4 – Trapping Rain Water
**LeetCode 42** | How much water is trapped between bars?

```java
public int trap(int[] height) {
    int l = 0, r = height.length - 1;
    int leftMax = 0, rightMax = 0, water = 0;
    while (l < r) {
        if (height[l] <= height[r]) {
            if (height[l] >= leftMax) leftMax = height[l];
            else water += leftMax - height[l];
            l++;
        } else {
            if (height[r] >= rightMax) rightMax = height[r];
            else water += rightMax - height[r];
            r--;
        }
    }
    return water;
}
// Key insight: water at position i = min(maxLeft, maxRight) - height[i]
// Two-pointer avoids needing two O(n) prefix/suffix arrays
// Time: O(n)  Space: O(1)
```

---

## 2. Sliding Window

**When to use:**
- Contiguous subarray/substring of fixed OR variable size
- Optimize a value (max/min/sum) over a window that slides forward

**Two types:**
- **Fixed window** – size k, slide one step at a time
- **Variable window** – expand right, shrink left when constraint violated

### Fixed Window Template
```java
int windowSum = 0, maxSum = 0;
// Build first window
for (int i = 0; i < k; i++) windowSum += arr[i];
maxSum = windowSum;
// Slide
for (int i = k; i < arr.length; i++) {
    windowSum += arr[i] - arr[i - k]; // add new, remove old
    maxSum = Math.max(maxSum, windowSum);
}
```

### Variable Window Template
```java
int l = 0, result = 0;
// state: e.g., current sum, freq map
for (int r = 0; r < arr.length; r++) {
    // expand: include arr[r] in window state
    while (/* constraint violated */) {
        // shrink: remove arr[l] from state
        l++;
    }
    result = Math.max(result, r - l + 1);
}
```

---

### Problem 2.1 – Maximum Average Subarray I (Fixed Window)
**LeetCode 643** | Contiguous subarray of length k with max average

```java
public double findMaxAverage(int[] nums, int k) {
    double sum = 0;
    for (int i = 0; i < k; i++) sum += nums[i];
    double maxSum = sum;
    for (int i = k; i < nums.length; i++) {
        sum += nums[i] - nums[i - k];
        maxSum = Math.max(maxSum, sum);
    }
    return maxSum / k;
}
// Time: O(n)  Space: O(1)
```

---

### Problem 2.2 – Longest Substring Without Repeating Characters (Variable Window)
**LeetCode 3**

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int l = 0, maxLen = 0;
    for (int r = 0; r < s.length(); r++) {
        char c = s.charAt(r);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= l) {
            l = lastSeen.get(c) + 1; // jump left past the duplicate
        }
        lastSeen.put(c, r);
        maxLen = Math.max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n)  Space: O(min(n, charset))
```

---

### Problem 2.3 – Minimum Size Subarray Sum (Variable Window)
**LeetCode 209** | Smallest subarray with sum ≥ target

```java
public int minSubArrayLen(int target, int[] nums) {
    int l = 0, sum = 0, minLen = Integer.MAX_VALUE;
    for (int r = 0; r < nums.length; r++) {
        sum += nums[r];
        while (sum >= target) {
            minLen = Math.min(minLen, r - l + 1);
            sum -= nums[l++]; // shrink from left
        }
    }
    return minLen == Integer.MAX_VALUE ? 0 : minLen;
}
// Time: O(n)  Space: O(1)
```

---

### Problem 2.4 – Longest Subarray with Ones After Deleting One Element
**LeetCode 1493** | At most one zero allowed (flip one 0 to extend)

```java
public int longestSubarray(int[] nums) {
    int l = 0, zeros = 0, maxLen = 0;
    for (int r = 0; r < nums.length; r++) {
        if (nums[r] == 0) zeros++;
        while (zeros > 1) {
            if (nums[l++] == 0) zeros--;
        }
        maxLen = Math.max(maxLen, r - l); // r - l (not +1) because we delete one
    }
    return maxLen;
}
// Time: O(n)  Space: O(1)
```

---

## 3. Prefix Sum

**When to use:**
- Range sum queries: sum from index `i` to `j` in O(1)
- Subarray sum = target (use HashMap for complement)
- Running balance / equilibrium problems

**Core idea:**
```
prefix[0] = 0
prefix[i] = arr[0] + arr[1] + ... + arr[i-1]
Sum(i..j) = prefix[j+1] - prefix[i]
```

```java
// Build prefix sum
int[] prefix = new int[nums.length + 1];
for (int i = 0; i < nums.length; i++)
    prefix[i + 1] = prefix[i] + nums[i];

// Query: sum from index l to r (inclusive, 0-indexed)
int rangeSum = prefix[r + 1] - prefix[l];
```

---

### Problem 3.1 – Subarray Sum Equals K
**LeetCode 560** | Count subarrays with sum = k

```java
public int subarraySum(int[] nums, int k) {
    // prefix[j] - prefix[i] == k  →  prefix[i] == prefix[j] - k
    Map<Integer, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0, 1); // empty prefix
    int sum = 0, count = 0;
    for (int num : nums) {
        sum += num;
        // how many past prefixes equal (sum - k)?
        count += prefixCount.getOrDefault(sum - k, 0);
        prefixCount.merge(sum, 1, Integer::sum);
    }
    return count;
}
// Time: O(n)  Space: O(n)
// KEY TRICK: HashMap of prefix → count; look for (currentSum - k)
```

---

### Problem 3.2 – Find Pivot Index
**LeetCode 724** | Index where left sum == right sum

```java
public int pivotIndex(int[] nums) {
    int total = 0;
    for (int n : nums) total += n;
    int leftSum = 0;
    for (int i = 0; i < nums.length; i++) {
        // rightSum = total - leftSum - nums[i]
        if (leftSum == total - leftSum - nums[i]) return i;
        leftSum += nums[i];
    }
    return -1;
}
// Time: O(n)  Space: O(1)
```

---

### Problem 3.3 – Product of Array Except Self
**LeetCode 238** | output[i] = product of all nums except nums[i]; no division

```java
public int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];

    // Pass 1: result[i] = product of all elements LEFT of i
    result[0] = 1;
    for (int i = 1; i < n; i++)
        result[i] = result[i - 1] * nums[i - 1];

    // Pass 2: multiply by product of all elements RIGHT of i
    int rightProduct = 1;
    for (int i = n - 1; i >= 0; i--) {
        result[i] *= rightProduct;
        rightProduct *= nums[i];
    }
    return result;
}
// Time: O(n)  Space: O(1) [output array not counted]
// KEY: Left pass stores left prefix; right pass accumulates suffix on the fly
```

---

## 4. Binary Search on Array

**When to use:**
- Sorted array (or monotonic condition)
- "Find minimum X satisfying condition Y"
- Rotated arrays, peak finding

**Standard Template – Find Leftmost Target:**
```java
int l = 0, r = arr.length; // r = arr.length for insertion point style
while (l < r) {
    int mid = l + (r - l) / 2; // avoid integer overflow
    if (arr[mid] >= target) r = mid;  // leftmost: keep looking left
    else                    l = mid + 1;
}
// l = first index where arr[l] >= target
```

---

### Problem 4.1 – Binary Search (Classic)
**LeetCode 704**

```java
public int search(int[] nums, int target) {
    int l = 0, r = nums.length - 1;
    while (l <= r) {
        int mid = l + (r - l) / 2;
        if      (nums[mid] == target) return mid;
        else if (nums[mid] < target)  l = mid + 1;
        else                          r = mid - 1;
    }
    return -1;
}
// Time: O(log n)  Space: O(1)
```

---

### Problem 4.2 – Search in Rotated Sorted Array
**LeetCode 33**

```java
public int search(int[] nums, int target) {
    int l = 0, r = nums.length - 1;
    while (l <= r) {
        int mid = l + (r - l) / 2;
        if (nums[mid] == target) return mid;

        // Determine which half is sorted
        if (nums[l] <= nums[mid]) {          // left half is sorted
            if (nums[l] <= target && target < nums[mid]) r = mid - 1;
            else l = mid + 1;
        } else {                              // right half is sorted
            if (nums[mid] < target && target <= nums[r]) l = mid + 1;
            else r = mid - 1;
        }
    }
    return -1;
}
// KEY: One half is always sorted; check if target is in sorted half first
// Time: O(log n)  Space: O(1)
```

---

### Problem 4.3 – Find Minimum in Rotated Sorted Array
**LeetCode 153**

```java
public int findMin(int[] nums) {
    int l = 0, r = nums.length - 1;
    while (l < r) {
        int mid = l + (r - l) / 2;
        if (nums[mid] > nums[r]) l = mid + 1; // min is in right half
        else                     r = mid;      // mid could be min
    }
    return nums[l];
}
// Time: O(log n)  Space: O(1)
```

---

### Problem 4.4 – Find Peak Element
**LeetCode 162** | Peak: nums[i] > neighbors (treat boundary as -∞)

```java
public int findPeakElement(int[] nums) {
    int l = 0, r = nums.length - 1;
    while (l < r) {
        int mid = l + (r - l) / 2;
        if (nums[mid] > nums[mid + 1]) r = mid; // peak on left side
        else                           l = mid + 1; // climbing; go right
    }
    return l;
}
// Time: O(log n)  Space: O(1)
```

---

### Problem 4.5 – Koko Eating Bananas (Binary Search on Answer)
**LeetCode 875** | Classic "search on the answer space" pattern

```java
public int minEatingSpeed(int[] piles, int h) {
    int l = 1, r = Arrays.stream(piles).max().getAsInt();
    while (l < r) {
        int mid = l + (r - l) / 2;
        if (canFinish(piles, mid, h)) r = mid; // valid; try smaller
        else                          l = mid + 1;
    }
    return l;
}

private boolean canFinish(int[] piles, int speed, int h) {
    int hours = 0;
    for (int p : piles) hours += (p + speed - 1) / speed; // ceil division
    return hours <= h;
}
// Pattern: binary search on answer (minimize speed where canFinish is true)
// Time: O(n log(maxPile))  Space: O(1)
```

---

## 5. Kadane's Algorithm

**When to use:**
- Maximum/minimum subarray (contiguous) sum
- "At least one element" constraint

**Core Idea:**
```
At each index, either:
  → extend the current subarray (add current element)
  → start fresh (current element alone is better)
dp[i] = max(nums[i], dp[i-1] + nums[i])
```

---

### Problem 5.1 – Maximum Subarray
**LeetCode 53**

```java
public int maxSubArray(int[] nums) {
    int maxSum = nums[0], curSum = nums[0];
    for (int i = 1; i < nums.length; i++) {
        curSum = Math.max(nums[i], curSum + nums[i]);
        maxSum = Math.max(maxSum, curSum);
    }
    return maxSum;
}
// Time: O(n)  Space: O(1)
```

---

### Problem 5.2 – Maximum Circular Subarray Sum
**LeetCode 918** | Array is circular (wraps around)

```java
public int maxSubarraySumCircular(int[] nums) {
    int totalSum = 0;
    int maxSum = nums[0], curMax = 0;
    int minSum = nums[0], curMin = 0;

    for (int num : nums) {
        curMax = Math.max(curMax + num, num);
        maxSum = Math.max(maxSum, curMax);
        curMin = Math.min(curMin + num, num);
        minSum = Math.min(minSum, curMin);
        totalSum += num;
    }
    // Circular max = total - minSubarray (wraps around)
    // But if all negative, maxSum is the answer (entire array = minSum = total → 0, invalid)
    return maxSum > 0 ? Math.max(maxSum, totalSum - minSum) : maxSum;
}
// KEY: circular subarray = totalSum - (minimum subarray)
// Time: O(n)  Space: O(1)
```

---

## 6. Dutch National Flag / 3-Way Partition

**When to use:**
- Sort array with 3 distinct values in O(n), O(1) space
- Partition around a pivot (QuickSort variant)
- Move zeros / segregate elements

**Template (3 pointers: low, mid, high):**
```java
int low = 0, mid = 0, high = arr.length - 1;
while (mid <= high) {
    if      (arr[mid] == 0) swap(arr, low++, mid++); // put 0 at low
    else if (arr[mid] == 1) mid++;                   // 1 in place
    else                    swap(arr, mid, high--);  // put 2 at high (don't mid++)
}
```

---

### Problem 6.1 – Sort Colors (0, 1, 2)
**LeetCode 75**

```java
public void sortColors(int[] nums) {
    int low = 0, mid = 0, high = nums.length - 1;
    while (mid <= high) {
        switch (nums[mid]) {
            case 0 -> { swap(nums, low++, mid++); }
            case 1 -> { mid++; }
            case 2 -> { swap(nums, mid, high--); } // no mid++ – swapped elem unchecked
        }
    }
}
private void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
// Time: O(n)  Space: O(1)
```

---

### Problem 6.2 – Move Zeroes
**LeetCode 283** | Move all zeros to end, preserve relative order of non-zeros

```java
public void moveZeroes(int[] nums) {
    int insertPos = 0;
    for (int num : nums) {                 // pass 1: compact non-zeros
        if (num != 0) nums[insertPos++] = num;
    }
    while (insertPos < nums.length) {      // pass 2: fill rest with 0
        nums[insertPos++] = 0;
    }
}
// Time: O(n)  Space: O(1)
```

---

## 7. Merge Intervals

**When to use:**
- Overlapping ranges that need merging
- Scheduling problems
- Insert new interval into sorted interval list

**Key check:** intervals `[a,b]` and `[c,d]` overlap if `c <= b`

---

### Problem 7.1 – Merge Intervals
**LeetCode 56**

```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // sort by start
    List<int[]> merged = new ArrayList<>();
    int[] current = intervals[0];
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] <= current[1]) {
            // overlap: extend current interval's end
            current[1] = Math.max(current[1], intervals[i][1]);
        } else {
            merged.add(current);
            current = intervals[i];
        }
    }
    merged.add(current);
    return merged.toArray(new int[0][]);
}
// Time: O(n log n)  Space: O(n)
```

---

### Problem 7.2 – Insert Interval
**LeetCode 57** | Insert new interval into sorted non-overlapping list

```java
public int[][] insert(int[][] intervals, int[] newInterval) {
    List<int[]> result = new ArrayList<>();
    int i = 0, n = intervals.length;

    // 1. Add all intervals that END before new interval starts
    while (i < n && intervals[i][1] < newInterval[0])
        result.add(intervals[i++]);

    // 2. Merge all overlapping intervals with newInterval
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.add(newInterval);

    // 3. Add remaining non-overlapping intervals
    while (i < n) result.add(intervals[i++]);

    return result.toArray(new int[0][]);
}
// Time: O(n)  Space: O(n)
```

---

### Problem 7.3 – Non-Overlapping Intervals (Min Removals)
**LeetCode 435** | Minimum intervals to remove to make rest non-overlapping

```java
public int eraseOverlapIntervals(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[1] - b[1]); // sort by END (greedy)
    int count = 0, end = Integer.MIN_VALUE;
    for (int[] interval : intervals) {
        if (interval[0] >= end) {      // no overlap: keep it
            end = interval[1];
        } else {
            count++;                   // overlap: remove it
        }
    }
    return count;
}
// Greedy: always keep interval with earliest end (leaves most room)
// Time: O(n log n)  Space: O(1)
```

---

## 8. Cyclic Sort

**When to use:**
- Array contains numbers in range `[1, n]` or `[0, n]`
- Find missing/duplicate numbers in O(n) time, O(1) space
- "Place each number at its correct index"

**Template:**
```java
int i = 0;
while (i < nums.length) {
    int correct = nums[i] - 1; // where nums[i] should be (0-indexed)
    if (nums[i] != nums[correct]) {
        swap(nums, i, correct); // place nums[i] at index (nums[i]-1)
    } else {
        i++;
    }
}
```

---

### Problem 8.1 – Missing Number
**LeetCode 268** | Array [0,n], find the one missing number

```java
public int missingNumber(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        if (nums[i] < nums.length && nums[i] != nums[nums[i]])
            swap(nums, i, nums[i]);
        else i++;
    }
    for (int j = 0; j < nums.length; j++)
        if (nums[j] != j) return j;
    return nums.length;
}
// Alternatively (no mutation): XOR or math formula
public int missingNumber2(int[] nums) {
    int n = nums.length, expected = n * (n + 1) / 2;
    for (int num : nums) expected -= num;
    return expected;
}
// Time: O(n)  Space: O(1)
```

---

### Problem 8.2 – Find All Duplicates in an Array
**LeetCode 442** | Range [1,n]; each appears once or twice; find all twice

```java
public List<Integer> findDuplicates(int[] nums) {
    List<Integer> result = new ArrayList<>();
    // Mark visited by negating index value
    for (int i = 0; i < nums.length; i++) {
        int idx = Math.abs(nums[i]) - 1;
        if (nums[idx] < 0) result.add(idx + 1); // already visited
        else               nums[idx] = -nums[idx];
    }
    return result;
}
// Strategy: use sign of nums[idx] as visited flag (O(1) space, no cyclic sort)
// Time: O(n)  Space: O(1)
```

---

### Problem 8.3 – Find the Duplicate Number
**LeetCode 287** | Array [1,n] has n+1 elements; one duplicate; O(1) space

```java
public int findDuplicate(int[] nums) {
    // Floyd's cycle detection (array as linked list: index → nums[index])
    int slow = nums[0], fast = nums[0];
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);
    // Find entry point of cycle
    slow = nums[0];
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return slow;
}
// Time: O(n)  Space: O(1)  — must NOT modify array
```

---

## 9. Matrix Traversal (2D Arrays)

**Core operations:**

```java
// Directions for 4-directional traversal
int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}}; // right, left, down, up
// For 8-directional (including diagonals):
int[][] dirs8 = {{0,1},{0,-1},{1,0},{-1,0},{1,1},{1,-1},{-1,1},{-1,-1}};

// Bounds check
boolean inBounds(int r, int c, int rows, int cols) {
    return r >= 0 && r < rows && c >= 0 && c < cols;
}

// Transpose a matrix (swap rows and columns)
for (int i = 0; i < n; i++)
    for (int j = i + 1; j < n; j++)
        swap(matrix[i][j], matrix[j][i]);
```

---

### Problem 9.1 – Spiral Matrix
**LeetCode 54** | Traverse matrix in spiral order

```java
public List<Integer> spiralOrder(int[][] matrix) {
    List<Integer> res = new ArrayList<>();
    int top = 0, bottom = matrix.length - 1;
    int left = 0, right = matrix[0].length - 1;

    while (top <= bottom && left <= right) {
        for (int c = left;  c <= right;  c++) res.add(matrix[top][c]);    top++;
        for (int r = top;   r <= bottom; r++) res.add(matrix[r][right]);  right--;
        if (top <= bottom)
            for (int c = right; c >= left; c--) res.add(matrix[bottom][c]); bottom--;
        if (left <= right)
            for (int r = bottom; r >= top; r--) res.add(matrix[r][left]);  left++;
    }
    return res;
}
// Time: O(m×n)  Space: O(1)
```

---

### Problem 9.2 – Rotate Image 90° Clockwise
**LeetCode 48** | In-place rotation

```java
public void rotate(int[][] matrix) {
    int n = matrix.length;
    // Step 1: Transpose (swap matrix[i][j] and matrix[j][i])
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++) {
            int tmp = matrix[i][j];
            matrix[i][j] = matrix[j][i];
            matrix[j][i] = tmp;
        }
    // Step 2: Reverse each row
    for (int[] row : matrix) {
        int l = 0, r = n - 1;
        while (l < r) { int tmp = row[l]; row[l++] = row[r]; row[r--] = tmp; }
    }
}
// Clockwise 90° = Transpose + Reverse each row
// Counter-clockwise = Reverse each row + Transpose
// Time: O(n²)  Space: O(1)
```

---

### Problem 9.3 – Set Matrix Zeroes
**LeetCode 73** | If cell is 0, set its entire row and column to 0; O(1) space

```java
public void setZeroes(int[][] matrix) {
    int m = matrix.length, n = matrix[0].length;
    boolean firstRowZero = false, firstColZero = false;
    // Check if first row/col have zeros
    for (int j = 0; j < n; j++) if (matrix[0][j] == 0) firstRowZero = true;
    for (int i = 0; i < m; i++) if (matrix[i][0] == 0) firstColZero = true;
    // Use first row/col as markers
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            if (matrix[i][j] == 0) { matrix[i][0] = 0; matrix[0][j] = 0; }
    // Zero out cells based on markers
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            if (matrix[i][0] == 0 || matrix[0][j] == 0) matrix[i][j] = 0;
    if (firstRowZero) for (int j = 0; j < n; j++) matrix[0][j] = 0;
    if (firstColZero) for (int i = 0; i < m; i++) matrix[i][0] = 0;
}
// Time: O(m×n)  Space: O(1) – use first row/col as flag storage
```

---

## 10. Monotonic Stack on Array

**When to use:**
- Next Greater / Next Smaller element
- Largest Rectangle in Histogram
- Problems asking: "what is the nearest element larger/smaller than me?"

**Monotonic Decreasing Stack (Next Greater):**
```java
Deque<Integer> stack = new ArrayDeque<>(); // stores indices
int[] nextGreater = new int[n];
Arrays.fill(nextGreater, -1);
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && arr[stack.peek()] < arr[i]) {
        nextGreater[stack.pop()] = arr[i]; // i is the next greater for top
    }
    stack.push(i);
}
```

---

### Problem 10.1 – Daily Temperatures
**LeetCode 739** | For each day, how many days until a warmer temperature?

```java
public int[] dailyTemperatures(int[] temps) {
    int n = temps.length;
    int[] result = new int[n];
    Deque<Integer> stack = new ArrayDeque<>(); // indices; monotonic decreasing
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && temps[stack.peek()] < temps[i]) {
            int idx = stack.pop();
            result[idx] = i - idx;
        }
        stack.push(i);
    }
    return result;
}
// Time: O(n)  Space: O(n)
```

---

### Problem 10.2 – Largest Rectangle in Histogram
**LeetCode 84**

```java
public int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>(); // monotonic increasing
    int maxArea = 0, n = heights.length;
    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i]; // sentinel 0 to flush stack
        while (!stack.isEmpty() && heights[stack.peek()] > h) {
            int height = heights[stack.pop()];
            int width  = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
// KEY: pop when current bar is shorter; width = distance to previous smaller bar
// Time: O(n)  Space: O(n)
```

---

### Problem 10.3 – Next Greater Element I
**LeetCode 496** | Given nums1 ⊂ nums2, find next greater in nums2 for each num1 element

```java
public int[] nextGreaterElement(int[] nums1, int[] nums2) {
    Map<Integer, Integer> nextGreater = new HashMap<>();
    Deque<Integer> stack = new ArrayDeque<>();
    for (int num : nums2) {
        while (!stack.isEmpty() && stack.peek() < num)
            nextGreater.put(stack.pop(), num);
        stack.push(num);
    }
    int[] result = new int[nums1.length];
    for (int i = 0; i < nums1.length; i++)
        result[i] = nextGreater.getOrDefault(nums1[i], -1);
    return result;
}
// Time: O(n1 + n2)  Space: O(n2)
```

---

## 11. Counting / Frequency Map

**When to use:**
- Majority element (Boyer-Moore Voting)
- Top-K frequent elements
- Anagram / character frequency problems

---

### Problem 11.1 – Majority Element (Boyer-Moore Voting)
**LeetCode 169** | Appears > n/2 times; always exists

```java
public int majorityElement(int[] nums) {
    int candidate = nums[0], count = 1;
    for (int i = 1; i < nums.length; i++) {
        if (count == 0) { candidate = nums[i]; count = 1; }
        else if (nums[i] == candidate) count++;
        else count--;
    }
    return candidate;
}
// KEY: majority element "cancels out" all others and still has votes left
// Time: O(n)  Space: O(1)
```

---

### Problem 11.2 – Top K Frequent Elements
**LeetCode 347** | O(n log k) via min-heap; O(n) via bucket sort

```java
// Approach 1: Min-heap (O(n log k))
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);

    PriorityQueue<Integer> minHeap = new PriorityQueue<>(Comparator.comparingInt(freq::get));
    for (int num : freq.keySet()) {
        minHeap.offer(num);
        if (minHeap.size() > k) minHeap.poll(); // remove least frequent
    }
    return minHeap.stream().mapToInt(Integer::intValue).toArray();
}

// Approach 2: Bucket sort (O(n)) — if frequency values bounded by n
public int[] topKFrequentBucket(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);

    List<Integer>[] buckets = new List[nums.length + 1]; // index = frequency
    for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
        int f = e.getValue();
        if (buckets[f] == null) buckets[f] = new ArrayList<>();
        buckets[f].add(e.getKey());
    }
    int[] result = new int[k];
    int ri = 0;
    for (int f = buckets.length - 1; f >= 0 && ri < k; f--)
        if (buckets[f] != null)
            for (int num : buckets[f]) { result[ri++] = num; if (ri == k) break; }
    return result;
}
```

---

### Problem 11.3 – Contains Duplicate II
**LeetCode 219** | Any two equal elements within k distance

```java
public boolean containsNearbyDuplicate(int[] nums, int k) {
    Map<Integer, Integer> lastIndex = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        if (lastIndex.containsKey(nums[i]) && i - lastIndex.get(nums[i]) <= k)
            return true;
        lastIndex.put(nums[i], i);
    }
    return false;
}
// Time: O(n)  Space: O(min(n, k))
```

---

## 12. Subarray / Subsequence Tricks

### Problem 12.1 – Best Time to Buy and Sell Stock
**LeetCode 121** | One transaction; maximize profit

```java
public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE, maxProfit = 0;
    for (int price : prices) {
        minPrice = Math.min(minPrice, price);
        maxProfit = Math.max(maxProfit, price - minPrice);
    }
    return maxProfit;
}
// Time: O(n)  Space: O(1)
```

---

### Problem 12.2 – Maximum Product Subarray
**LeetCode 152** | Tricky because negatives can flip sign

```java
public int maxProduct(int[] nums) {
    int maxProd = nums[0], minProd = nums[0], result = nums[0];
    for (int i = 1; i < nums.length; i++) {
        // negative flips max↔min; so consider all three candidates
        int candidates[] = {nums[i], maxProd * nums[i], minProd * nums[i]};
        maxProd = Arrays.stream(candidates).max().getAsInt();
        minProd = Arrays.stream(candidates).min().getAsInt();
        result = Math.max(result, maxProd);
    }
    return result;
}
// KEY: Track both max AND min product (negative × negative = big positive)
// Time: O(n)  Space: O(1)
```

---

### Problem 12.3 – Longest Consecutive Sequence
**LeetCode 128** | O(n) without sorting

```java
public int longestConsecutive(int[] nums) {
    Set<Integer> numSet = new HashSet<>();
    for (int n : nums) numSet.add(n);
    int maxLen = 0;
    for (int n : numSet) {
        if (!numSet.contains(n - 1)) { // only start from sequence beginning
            int len = 1;
            while (numSet.contains(n + len)) len++;
            maxLen = Math.max(maxLen, len);
        }
    }
    return maxLen;
}
// KEY: Only start counting from numbers that have no left neighbor
// Time: O(n)  Space: O(n)
```

---

## Quick Reference – Pattern Decision Tree

```
Array problem?
│
├── Sorted array + find pair/triplet? ──────────────────────→ Two Pointers
│
├── Subarray sum / frequency? ──────────────────────────────→ Prefix Sum + HashMap
│
├── Contiguous subarray, max/min/count? ────────────────────→ Sliding Window
│
├── Sorted array, find element in O(log n)? ────────────────→ Binary Search
│   └── Rotated? Peak? Minimize answer?  ──────────────────→ Binary Search variant
│
├── Max/min sum of contiguous subarray? ────────────────────→ Kadane's Algorithm
│
├── 3 categories, sort in-place? ───────────────────────────→ Dutch National Flag
│
├── Overlapping ranges? Scheduling? ──────────────────────  → Merge Intervals
│
├── Numbers in range [1..n], find dup/missing? ─────────────→ Cyclic Sort
│
├── 2D grid traversal / rotation? ─────────────────────────→ Matrix Patterns
│
├── Next greater/smaller, largest rectangle? ───────────────→ Monotonic Stack
│
├── Frequency / most common? ───────────────────────────────→ HashMap / Bucket Sort
│
└── Stock prices / product tricks? ─────────────────────────→ Greedy + Running State
```

---

*DSA Array Notes – Pattern-wise | Java | Updated: March 2026*
