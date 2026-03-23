# Dynamic Programming – Pattern-Wise DSA Notes

> **Language:** Java | **Level:** Interview-ready | **Focus:** Pattern recognition, state design, transitions, optimization

## Pattern Index

| # | Pattern | Key Problems |
|---|---|---|
| 1 | [Core Concepts & Framework](#1-core-concepts--framework) | Fibonacci, Climbing Stairs |
| 2 | [0/1 Knapsack](#2-01-knapsack-pattern) | Knapsack, Subset Sum, Partition Equal Subset |
| 3 | [Unbounded Knapsack](#3-unbounded-knapsack-pattern) | Coin Change, Rod Cutting, Max Ribbon Cut |
| 4 | [1D DP – Linear](#4-1d-dp--linear) | House Robber, Jump Game, Decode Ways |
| 5 | [2D DP – Grid](#5-2d-dp--grid) | Unique Paths, Min Path Sum, Dungeon Game |
| 6 | [Longest Common Subsequence](#6-longest-common-subsequence-pattern) | LCS, Edit Distance, Shortest Common Supersequence |
| 7 | [Longest Increasing Subsequence](#7-longest-increasing-subsequence-pattern) | LIS, Russian Dolls, Max Envelopes |
| 8 | [String DP](#8-string-dp) | Palindrome Substrings, Palindrome Partitioning, Wildcard Match |
| 9 | [Interval DP](#9-interval-dp) | Matrix Chain Multiplication, Burst Balloons, Stone Merge |
| 10 | [Tree DP](#10-tree-dp) | Diameter, Max Path Sum, House Robber III |
| 11 | [DP on Subsets / Bitmask](#11-dp-on-subsets--bitmask) | Traveling Salesman, Minimum XOR Sum |
| 12 | [DP + Stock Problems](#12-dp--stock-problems) | Best Time to Buy and Sell (all 6 variants) |

---

## 1. Core Concepts & Framework

### What is DP?

```
DP = Breaking a problem into OVERLAPPING SUBPROBLEMS
     + storing solutions to avoid recomputation (MEMOIZATION / TABULATION)

Two conditions for DP:
  1. Optimal Substructure: optimal solution built from optimal sub-solutions
  2. Overlapping Subproblems: same subproblems recomputed repeatedly

DP ≠ Greedy:   Greedy picks local best; DP considers ALL choices
DP ≠ Divide & Conquer: D&C subproblems don't overlap (Merge Sort)
```

### Two Approaches

```
TOP-DOWN (Memoization):
  Write recursion naturally → add a cache (Map or array)
  + Intuitive to write; only computes needed states
  - Recursion overhead; risk of stack overflow for large n

BOTTOM-UP (Tabulation):
  Fill a DP table iteratively from smallest subproblem
  + No recursion; better cache performance; space-optimal with rolling array
  - Need to figure out correct iteration order
```

### DP Framework (Apply to Every Problem)

```
Step 1: Define the state
        dp[i] = answer for subproblem of size/index i
        dp[i][j] = answer for subproblem with two parameters

Step 2: Write the recurrence (transition)
        HOW does dp[i] depend on smaller states?

Step 3: Identify base cases
        Smallest subproblem with a known answer

Step 4: Determine iteration order (for tabulation)
        Ensure dp[i-1], dp[i-2] etc. are computed before dp[i]

Step 5: Return the correct cell
        Often dp[n], dp[n][m], or max over range
```

### Fibonacci – Both Approaches

```java
// TOP-DOWN (Memoization)
int[] memo = new int[n + 1];
Arrays.fill(memo, -1);

int fib(int n) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    return memo[n] = fib(n - 1) + fib(n - 2);
}
// Time: O(n)  Space: O(n)

// BOTTOM-UP (Tabulation)
int fib(int n) {
    if (n <= 1) return n;
    int[] dp = new int[n + 1];
    dp[0] = 0; dp[1] = 1;
    for (int i = 2; i <= n; i++) dp[i] = dp[i-1] + dp[i-2];
    return dp[n];
}
// Time: O(n)  Space: O(n)

// SPACE OPTIMIZED (only need last 2 values)
int fib(int n) {
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
// Time: O(n)  Space: O(1)
```

---

## 2. 0/1 Knapsack Pattern

**Core idea:** For each item, make a binary choice: **take it** or **skip it**.
"0/1" = each item used at most once.

**State:** `dp[i][w]` = max value using first `i` items with weight capacity `w`

**Transition:**
```
dp[i][w] = max(
    dp[i-1][w],                         // skip item i
    dp[i-1][w - weight[i]] + value[i]   // take item i (if w >= weight[i])
)
```

---

### Problem 2.1 – 0/1 Knapsack
Classic: n items each with weight and value; capacity W; maximize value

```java
public int knapsack(int[] weights, int[] values, int W) {
    int n = weights.length;
    int[][] dp = new int[n + 1][W + 1];

    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            dp[i][w] = dp[i-1][w]; // skip item i
            if (weights[i-1] <= w) {
                dp[i][w] = Math.max(dp[i][w],
                    dp[i-1][w - weights[i-1]] + values[i-1]); // take item i
            }
        }
    }
    return dp[n][W];
}
// Time: O(n × W)  Space: O(n × W)

// SPACE OPTIMIZED to O(W) — iterate w in REVERSE (so we use "previous row" values)
public int knapsack1D(int[] weights, int[] values, int W) {
    int n = weights.length;
    int[] dp = new int[W + 1];
    for (int i = 0; i < n; i++) {
        for (int w = W; w >= weights[i]; w--) { // REVERSE to prevent reuse
            dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
        }
    }
    return dp[W];
}
// Time: O(n × W)  Space: O(W)
// KEY: reverse iteration prevents using item i more than once
```

---

### Problem 2.2 – Subset Sum
**LeetCode-style:** Can we pick a subset of nums that sums exactly to target?

```java
public boolean canPartition(int[] nums, int target) {
    boolean[] dp = new boolean[target + 1];
    dp[0] = true; // empty subset = sum 0

    for (int num : nums) {
        for (int j = target; j >= num; j--) { // reverse: each num used once
            dp[j] = dp[j] || dp[j - num];
        }
    }
    return dp[target];
}
// Time: O(n × target)  Space: O(target)
```

---

### Problem 2.3 – Partition Equal Subset Sum
**LeetCode 416:** Split array into two equal-sum subsets

```java
public boolean canPartition(int[] nums) {
    int total = Arrays.stream(nums).sum();
    if (total % 2 != 0) return false; // odd sum → impossible
    int target = total / 2;

    boolean[] dp = new boolean[target + 1];
    dp[0] = true;
    for (int num : nums) {
        for (int j = target; j >= num; j--) {
            dp[j] = dp[j] || dp[j - num];
        }
    }
    return dp[target];
}
// Time: O(n × sum)  Space: O(sum)
```

---

### Problem 2.4 – Target Sum
**LeetCode 494:** Assign + or − to each number, count ways to reach target

```java
// Math reduction: assign + to set P, − to set N
// P - N = target,  P + N = sum  →  P = (sum + target) / 2
// Count subsets with sum = (sum + target) / 2
public int findTargetSumWays(int[] nums, int target) {
    int sum = Arrays.stream(nums).sum();
    if ((sum + target) % 2 != 0 || Math.abs(target) > sum) return 0;
    int s = (sum + target) / 2;

    int[] dp = new int[s + 1];
    dp[0] = 1; // one way to get sum 0: empty subset
    for (int num : nums) {
        for (int j = s; j >= num; j--) {
            dp[j] += dp[j - num]; // count ways
        }
    }
    return dp[s];
}
// Time: O(n × s)  Space: O(s)
```

---

### Problem 2.5 – 0/1 Knapsack Variant: Last Stone Weight II
**LeetCode 1049:** Minimize |sum(S1) - sum(S2)| by splitting stones into two groups

```java
public int lastStoneWeightII(int[] stones) {
    int sum = Arrays.stream(stones).sum();
    int target = sum / 2; // maximize one group's sum ≤ total/2
    boolean[] dp = new boolean[target + 1];
    dp[0] = true;
    for (int stone : stones) {
        for (int j = target; j >= stone; j--) {
            dp[j] = dp[j] || dp[j - stone];
        }
    }
    for (int j = target; j >= 0; j--) {
        if (dp[j]) return sum - 2 * j; // minimize difference
    }
    return sum;
}
```

---

## 3. Unbounded Knapsack Pattern

**Core idea:** Each item can be used **unlimited times**.

**Key difference from 0/1:** Iterate weight capacity **forward** (not reverse), allowing item reuse.

**Transition:**
```
dp[w] = max(dp[w], dp[w - weight[i]] + value[i])
         ↑ skip item      ↑ REUSE from same row (unbounded)
```

---

### Problem 3.1 – Coin Change (Min Coins)
**LeetCode 322:** Fewest coins to make amount

```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1); // "infinity" = impossible
    dp[0] = 0;
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
// Time: O(amount × coins)  Space: O(amount)
// State: dp[i] = min coins to make amount i
```

---

### Problem 3.2 – Coin Change II (Count Ways)
**LeetCode 518:** Number of combinations to make amount (order doesn't matter)

```java
public int change(int amount, int[] coins) {
    int[] dp = new int[amount + 1];
    dp[0] = 1; // one way to make 0: use no coins
    for (int coin : coins) {           // outer = items
        for (int j = coin; j <= amount; j++) { // inner = capacity (forward)
            dp[j] += dp[j - coin];
        }
    }
    return dp[amount];
}
// Time: O(amount × coins)  Space: O(amount)
// KEY: outer loop on COINS avoids counting permutations as different combinations
// Swap loop order → counts permutations (e.g., Combination Sum IV LeetCode 377)
```

---

### Problem 3.3 – Combination Sum IV (Count Permutations)
**LeetCode 377:** Order matters (different sequences count separately)

```java
public int combinationSum4(int[] nums, int target) {
    int[] dp = new int[target + 1];
    dp[0] = 1;
    for (int j = 1; j <= target; j++) {  // outer = target (permutations)
        for (int num : nums) {
            if (num <= j) dp[j] += dp[j - num];
        }
    }
    return dp[target];
}
// Swapping loop order vs Coin Change II is the key insight for combinations vs permutations
```

---

### Problem 3.4 – Perfect Squares
**LeetCode 279:** Minimum number of perfect squares summing to n

```java
public int numSquares(int n) {
    int[] dp = new int[n + 1];
    Arrays.fill(dp, n + 1);
    dp[0] = 0;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j * j <= i; j++) { // try all perfect squares ≤ i
            dp[i] = Math.min(dp[i], dp[i - j * j] + 1);
        }
    }
    return dp[n];
}
// Time: O(n√n)  Space: O(n)
```

---

## 4. 1D DP – Linear

Problems where `dp[i]` depends on a few previous states in a 1D array.

---

### Problem 4.1 – Climbing Stairs
**LeetCode 70:** n steps, can climb 1 or 2 at a time

```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    int prev2 = 1, prev1 = 2;
    for (int i = 3; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1; prev1 = curr;
    }
    return prev1;
}
// Identical to Fibonacci  Time: O(n)  Space: O(1)
// Generalize: k steps → dp[i] = sum(dp[i-1], dp[i-2], ..., dp[i-k])
```

---

### Problem 4.2 – House Robber
**LeetCode 198:** Can't rob adjacent houses; maximize total

```java
public int rob(int[] nums) {
    if (nums.length == 1) return nums[0];
    int prev2 = nums[0], prev1 = Math.max(nums[0], nums[1]);
    for (int i = 2; i < nums.length; i++) {
        int curr = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1; prev1 = curr;
    }
    return prev1;
}
// State: dp[i] = max money robbing up to house i
// dp[i] = max(dp[i-1], dp[i-2] + nums[i])
// Time: O(n)  Space: O(1)
```

---

### Problem 4.3 – House Robber II (Circular)
**LeetCode 213:** Houses in a circle (first and last are adjacent)

```java
public int rob(int[] nums) {
    if (nums.length == 1) return nums[0];
    // Rob either [0..n-2] or [1..n-1]; take the max
    return Math.max(
        robLinear(nums, 0, nums.length - 2),
        robLinear(nums, 1, nums.length - 1)
    );
}

private int robLinear(int[] nums, int start, int end) {
    int prev2 = 0, prev1 = 0;
    for (int i = start; i <= end; i++) {
        int curr = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1; prev1 = curr;
    }
    return prev1;
}
// Time: O(n)  Space: O(1)
```

---

### Problem 4.4 – Decode Ways
**LeetCode 91:** Count ways to decode a digit string into letters (A=1...Z=26)

```java
public int numDecodings(String s) {
    int n = s.length();
    // dp[i] = ways to decode s[0..i-1]
    int[] dp = new int[n + 1];
    dp[0] = 1;                     // empty string: 1 way
    dp[1] = s.charAt(0) != '0' ? 1 : 0;

    for (int i = 2; i <= n; i++) {
        int oneDigit = Integer.parseInt(s.substring(i - 1, i));
        int twoDigit = Integer.parseInt(s.substring(i - 2, i));
        if (oneDigit >= 1)                    dp[i] += dp[i - 1]; // single decode
        if (twoDigit >= 10 && twoDigit <= 26) dp[i] += dp[i - 2]; // double decode
    }
    return dp[n];
}
// Time: O(n)  Space: O(n) → optimizable to O(1) with two vars
```

---

### Problem 4.5 – Jump Game II (Min Jumps)
**LeetCode 45:** Minimum jumps to reach last index

```java
// Greedy DP: track furthest reachable at each "level"
public int jump(int[] nums) {
    int jumps = 0, currentEnd = 0, farthest = 0;
    for (int i = 0; i < nums.length - 1; i++) {
        farthest = Math.max(farthest, i + nums[i]);
        if (i == currentEnd) { // must take a jump
            jumps++;
            currentEnd = farthest;
        }
    }
    return jumps;
}
// Time: O(n)  Space: O(1)
// Greedy: at each "window" [prev+1..currentEnd], pick the jump that goes furthest
```

---

### Problem 4.6 – Min Cost Climbing Stairs
**LeetCode 746:** Minimum cost to reach the top (can step 1 or 2)

```java
public int minCostClimbingStairs(int[] cost) {
    int n = cost.length;
    // dp[i] = min cost to climb FROM step i
    int prev2 = cost[0], prev1 = cost[1];
    for (int i = 2; i < n; i++) {
        int curr = cost[i] + Math.min(prev1, prev2);
        prev2 = prev1; prev1 = curr;
    }
    return Math.min(prev1, prev2); // can start at 0 or 1
}
// Time: O(n)  Space: O(1)
```

---

## 5. 2D DP – Grid

**State:** `dp[i][j]` = answer at cell (i, j), typically with top-left as base.

**Common transitions:**
```
From top:    dp[i][j] ← dp[i-1][j]
From left:   dp[i][j] ← dp[i][j-1]
From diag:   dp[i][j] ← dp[i-1][j-1]
```

---

### Problem 5.1 – Unique Paths
**LeetCode 62:** Robot from top-left to bottom-right, only moves right/down

```java
public int uniquePaths(int m, int n) {
    int[] dp = new int[n];
    Arrays.fill(dp, 1); // first row: only 1 way (all right)
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[j] += dp[j - 1]; // from top (dp[j]) + from left (dp[j-1])
        }
    }
    return dp[n - 1];
}
// Time: O(m × n)  Space: O(n)
// Math solution: C(m+n-2, m-1) — combination
```

---

### Problem 5.2 – Minimum Path Sum
**LeetCode 64:** Grid with costs; min cost from top-left to bottom-right

```java
public int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    // Modify grid in-place (or use dp array)
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (i == 0 && j == 0) continue;
            else if (i == 0) grid[i][j] += grid[i][j - 1];
            else if (j == 0) grid[i][j] += grid[i - 1][j];
            else grid[i][j] += Math.min(grid[i-1][j], grid[i][j-1]);
        }
    }
    return grid[m - 1][n - 1];
}
// Time: O(m × n)  Space: O(1) in-place
```

---

### Problem 5.3 – Triangle (Min Path Sum, Variable Width)
**LeetCode 120:** Min path from top to bottom of triangle

```java
public int minimumTotal(List<List<Integer>> triangle) {
    int n = triangle.size();
    int[] dp = new int[n];
    // Start from bottom row
    for (int i = 0; i < n; i++) dp[i] = triangle.get(n - 1).get(i);
    // Process bottom-up
    for (int i = n - 2; i >= 0; i--) {
        for (int j = 0; j <= i; j++) {
            dp[j] = triangle.get(i).get(j) + Math.min(dp[j], dp[j + 1]);
        }
    }
    return dp[0];
}
// Time: O(n²)  Space: O(n) — bottom-up avoids needing 2D array
```

---

### Problem 5.4 – Dungeon Game
**LeetCode 174:** Knight needs min health to rescue princess; must maintain health > 0

```java
public int calculateMinimumHP(int[][] dungeon) {
    int m = dungeon.length, n = dungeon[0].length;
    int[][] dp = new int[m + 1][n + 1];
    // dp[i][j] = min health needed entering cell (i,j)
    // Fill borders with infinity; bottom-right exit needs at least 1
    for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE);
    dp[m][n - 1] = dp[m - 1][n] = 1; // sentinels at borders

    for (int i = m - 1; i >= 0; i--) {
        for (int j = n - 1; j >= 0; j--) {
            int minNext = Math.min(dp[i + 1][j], dp[i][j + 1]);
            dp[i][j] = Math.max(1, minNext - dungeon[i][j]);
            // must be at least 1 (can't enter with 0 or negative health)
        }
    }
    return dp[0][0];
}
// Time: O(m × n)  Space: O(m × n) → optimizable to O(n)
// KEY: Work backward from destination; forward DP has circularity issues
```

---

## 6. Longest Common Subsequence Pattern

**State:** `dp[i][j]` = LCS length of `s1[0..i-1]` and `s2[0..j-1]`

**Transition:**
```
if s1[i-1] == s2[j-1]:  dp[i][j] = dp[i-1][j-1] + 1
else:                    dp[i][j] = max(dp[i-1][j], dp[i][j-1])
```

---

### Problem 6.1 – Longest Common Subsequence
**LeetCode 1143:**

```java
public int longestCommonSubsequence(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i-1) == s2.charAt(j-1))
                dp[i][j] = dp[i-1][j-1] + 1;
            else
                dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
        }
    }
    return dp[m][n];
}
// Time: O(m × n)  Space: O(m × n) → O(n) with rolling array
```

---

### Problem 6.2 – Edit Distance (Levenshtein)
**LeetCode 72:** Min operations (insert, delete, replace) to convert word1 → word2

```java
public int minDistance(String w1, String w2) {
    int m = w1.length(), n = w2.length();
    // dp[i][j] = min ops to convert w1[0..i-1] → w2[0..j-1]
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) dp[i][0] = i; // delete all of w1
    for (int j = 0; j <= n; j++) dp[0][j] = j; // insert all of w2

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (w1.charAt(i-1) == w2.charAt(j-1))
                dp[i][j] = dp[i-1][j-1]; // no operation needed
            else
                dp[i][j] = 1 + Math.min(dp[i-1][j-1],    // replace
                               Math.min(dp[i-1][j],        // delete from w1
                                        dp[i][j-1]));      // insert into w1
        }
    }
    return dp[m][n];
}
// Time: O(m × n)  Space: O(m × n)
```

---

### Problem 6.3 – Shortest Common Supersequence
**LeetCode 1092:** Shortest string containing both s1 and s2 as subsequences

```java
public String shortestCommonSupersequence(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m + 1][n + 1]; // LCS table
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i-1) == s2.charAt(j-1)) dp[i][j] = dp[i-1][j-1] + 1;
            else dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
        }

    // Reconstruct SCS by tracing LCS table
    StringBuilder sb = new StringBuilder();
    int i = m, j = n;
    while (i > 0 && j > 0) {
        if (s1.charAt(i-1) == s2.charAt(j-1)) { sb.append(s1.charAt(i-1)); i--; j--; }
        else if (dp[i-1][j] > dp[i][j-1])     { sb.append(s1.charAt(i-1)); i--; }
        else                                    { sb.append(s2.charAt(j-1)); j--; }
    }
    while (i > 0) sb.append(s1.charAt(i-- - 1));
    while (j > 0) sb.append(s2.charAt(j-- - 1));
    return sb.reverse().toString();
}
// SCS length = m + n - LCS(s1, s2)
```

---

## 7. Longest Increasing Subsequence Pattern

**State:** `dp[i]` = length of LIS ending at index i

**Transition:** `dp[i] = max(dp[j] + 1)` for all `j < i` where `arr[j] < arr[i]`

---

### Problem 7.1 – LIS (O(n²) DP)
**LeetCode 300:**

```java
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];
    Arrays.fill(dp, 1); // each element alone = LIS of length 1
    int maxLen = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i])
                dp[i] = Math.max(dp[i], dp[j] + 1);
        }
        maxLen = Math.max(maxLen, dp[i]);
    }
    return maxLen;
}
// Time: O(n²)  Space: O(n)
```

---

### Problem 7.2 – LIS (O(n log n) via Binary Search)

```java
public int lengthOfLIS(int[] nums) {
    // tails[i] = smallest tail element of increasing subsequence of length i+1
    List<Integer> tails = new ArrayList<>();
    for (int num : nums) {
        // Binary search for first tail >= num (lower_bound)
        int lo = 0, hi = tails.size();
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (tails.get(mid) < num) lo = mid + 1;
            else                       hi = mid;
        }
        if (lo == tails.size()) tails.add(num);    // extend LIS
        else                    tails.set(lo, num); // replace to keep tails small
    }
    return tails.size();
}
// Time: O(n log n)  Space: O(n)
// tails array is always sorted; binary search finds insertion point
```

---

### Problem 7.3 – Russian Doll Envelopes (2D LIS)
**LeetCode 354:** Envelope fits inside another if BOTH width AND height are strictly greater

```java
public int maxEnvelopes(int[][] envelopes) {
    // Sort by width ASC; if same width, sort by height DESC
    // → prevents picking multiple same-width envelopes
    Arrays.sort(envelopes, (a, b) ->
        a[0] != b[0] ? a[0] - b[0] : b[1] - a[1]);

    // LIS on heights only (O(n log n))
    int[] tails = new int[envelopes.length];
    int size = 0;
    for (int[] env : envelopes) {
        int h = env[1];
        int lo = 0, hi = size;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (tails[mid] < h) lo = mid + 1;
            else                hi = mid;
        }
        tails[lo] = h;
        if (lo == size) size++;
    }
    return size;
}
// Time: O(n log n)  Space: O(n)
// Key trick: DESC sort by height within same width → LIS can't pick two same-width
```

---

## 8. String DP

### Problem 8.1 – Longest Palindromic Substring
**LeetCode 5:** Expand around center approach (not DP but related; DP below)

```java
// DP approach: dp[i][j] = true if s[i..j] is palindrome
public String longestPalindrome(String s) {
    int n = s.length(), start = 0, maxLen = 1;
    boolean[][] dp = new boolean[n][n];
    for (int i = 0; i < n; i++) dp[i][i] = true; // single chars

    // Check substrings of length 2 to n
    for (int len = 2; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            if (s.charAt(i) == s.charAt(j)) {
                dp[i][j] = (len == 2) || dp[i+1][j-1];
                if (dp[i][j] && len > maxLen) { maxLen = len; start = i; }
            }
        }
    }
    return s.substring(start, start + maxLen);
}
// Time: O(n²)  Space: O(n²)
// Expand-around-center: O(n²) time, O(1) space (preferred in interview)
```

---

### Problem 8.2 – Palindrome Partitioning II (Min Cuts)
**LeetCode 132:** Minimum cuts to partition string so each part is a palindrome

```java
public int minCut(String s) {
    int n = s.length();
    // isPalin[i][j] = is s[i..j] palindrome
    boolean[][] isPalin = new boolean[n][n];
    for (int i = n - 1; i >= 0; i--) {
        for (int j = i; j < n; j++) {
            isPalin[i][j] = s.charAt(i) == s.charAt(j)
                && (j - i <= 2 || isPalin[i+1][j-1]);
        }
    }
    // cut[i] = min cuts for s[0..i]
    int[] cut = new int[n];
    for (int i = 0; i < n; i++) {
        if (isPalin[0][i]) { cut[i] = 0; continue; } // whole prefix is palindrome
        cut[i] = i; // max cuts = i (each char separately)
        for (int j = 1; j <= i; j++) {
            if (isPalin[j][i])
                cut[i] = Math.min(cut[i], cut[j-1] + 1);
        }
    }
    return cut[n - 1];
}
// Time: O(n²)  Space: O(n²)
```

---

### Problem 8.3 – Wildcard Matching
**LeetCode 44:** `?` matches any single char; `*` matches any sequence (including empty)

```java
public boolean isMatch(String s, String p) {
    int m = s.length(), n = p.length();
    // dp[i][j] = does s[0..i-1] match p[0..j-1]
    boolean[][] dp = new boolean[m + 1][n + 1];
    dp[0][0] = true;
    // '*' can match empty: dp[0][j] = true if p[0..j-1] is all '*'
    for (int j = 1; j <= n; j++) dp[0][j] = dp[0][j-1] && p.charAt(j-1) == '*';

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            char pc = p.charAt(j - 1);
            if (pc == '*') {
                dp[i][j] = dp[i-1][j]   // '*' matches one more char of s
                         || dp[i][j-1]; // '*' matches empty (skip '*')
            } else {
                dp[i][j] = dp[i-1][j-1]
                    && (pc == '?' || pc == s.charAt(i-1));
            }
        }
    }
    return dp[m][n];
}
// Time: O(m × n)  Space: O(m × n)
```

---

## 9. Interval DP

**State:** `dp[i][j]` = optimal answer for subproblem on interval `[i, j]`

**Transition:** Try every split point `k` in `[i, j-1]`:
```
dp[i][j] = optimal over k of (dp[i][k] op dp[k+1][j] + cost(i,j,k))
```

**Iteration order:** Must process smaller intervals before larger ones.

---

### Problem 9.1 – Burst Balloons
**LeetCode 312:** Burst all balloons to maximize coins; bursting costs `nums[left] * nums[i] * nums[right]`

```java
public int maxCoins(int[] nums) {
    int n = nums.length;
    // Add boundary balloons of value 1
    int[] arr = new int[n + 2];
    arr[0] = arr[n + 1] = 1;
    for (int i = 0; i < n; i++) arr[i + 1] = nums[i];
    int m = n + 2;

    // dp[i][j] = max coins bursting all balloons between i and j (exclusive)
    int[][] dp = new int[m][m];

    for (int len = 2; len < m; len++) {         // length of interval
        for (int i = 0; i < m - len; i++) {
            int j = i + len;
            for (int k = i + 1; k < j; k++) {  // k = LAST balloon burst in (i,j)
                dp[i][j] = Math.max(dp[i][j],
                    dp[i][k] + arr[i] * arr[k] * arr[j] + dp[k][j]);
            }
        }
    }
    return dp[0][m - 1];
}
// KEY INSIGHT: Think of k as LAST balloon to burst (not first)
//   → known boundaries arr[i] and arr[j] still exist when k is burst
// Time: O(n³)  Space: O(n²)
```

---

### Problem 9.2 – Stone Merge (Min Cost to Merge)
**LeetCode 1000 / Classic:** Min cost to merge k consecutive piles into one

```java
// Merge exactly 2 piles at a time variant (classic):
public int mergeStones(int[] stones) {
    int n = stones.length;
    int[] prefix = new int[n + 1];
    for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + stones[i];

    int[][] dp = new int[n][n]; // dp[i][j] = min cost to merge stones[i..j]
    for (int len = 2; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            dp[i][j] = Integer.MAX_VALUE;
            for (int k = i; k < j; k++) {
                dp[i][j] = Math.min(dp[i][j],
                    dp[i][k] + dp[k+1][j] + prefix[j+1] - prefix[i]);
            }
        }
    }
    return dp[0][n - 1];
}
// Time: O(n³)  Space: O(n²)
```

---

## 10. Tree DP

DP on trees — compute answers bottom-up via DFS post-order.

**Pattern:** Solve for subtrees first, combine at each node.

---

### Problem 10.1 – Diameter of Binary Tree
**LeetCode 543:** Longest path between any two nodes (may not pass through root)

```java
int maxDiameter = 0;

public int diameterOfBinaryTree(TreeNode root) {
    height(root);
    return maxDiameter;
}

private int height(TreeNode node) {
    if (node == null) return 0;
    int left  = height(node.left);
    int right = height(node.right);
    maxDiameter = Math.max(maxDiameter, left + right); // diameter through this node
    return 1 + Math.max(left, right);                  // return height up
}
// Time: O(n)  Space: O(h) stack
```

---

### Problem 10.2 – Binary Tree Maximum Path Sum
**LeetCode 124:** Path can start/end at any node; path cannot fork

```java
int maxSum = Integer.MIN_VALUE;

public int maxPathSum(TreeNode root) {
    gainFrom(root);
    return maxSum;
}

private int gainFrom(TreeNode node) {
    if (node == null) return 0;
    int leftGain  = Math.max(0, gainFrom(node.left));  // ignore negative paths
    int rightGain = Math.max(0, gainFrom(node.right));
    maxSum = Math.max(maxSum, node.val + leftGain + rightGain); // path through node
    return node.val + Math.max(leftGain, rightGain);            // return one branch
}
// Time: O(n)  Space: O(h)
// KEY: Return ONLY one branch upward (can't take both — that would fork the path)
```

---

### Problem 10.3 – House Robber III (Tree)
**LeetCode 337:** Rob houses on binary tree; can't rob parent and child together

```java
public int rob(TreeNode root) {
    int[] res = dfs(root);
    return Math.max(res[0], res[1]);
}

// returns [maxRobWithoutRoot, maxRobWithRoot]
private int[] dfs(TreeNode node) {
    if (node == null) return new int[]{0, 0};
    int[] left  = dfs(node.left);
    int[] right = dfs(node.right);

    int withoutRoot = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);
    int withRoot    = node.val + left[0] + right[0]; // children NOT robbed

    return new int[]{withoutRoot, withRoot};
}
// Time: O(n)  Space: O(h)
```

---

## 11. DP on Subsets / Bitmask

**When to use:** Small n (≤ 20); need to track which subset has been visited/used.

**State:** `dp[mask]` or `dp[mask][i]` — mask encodes which elements included.

---

### Problem 11.1 – Traveling Salesman Problem (TSP)
Classic: Visit all cities exactly once, return to start; minimize total distance.

```java
public int tsp(int[][] dist) {
    int n = dist.length;
    int FULL = (1 << n) - 1;
    // dp[mask][i] = min cost to visit cities in mask, ending at city i
    int[][] dp = new int[1 << n][n];
    for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE / 2);
    dp[1][0] = 0; // start at city 0, only city 0 visited

    for (int mask = 1; mask <= FULL; mask++) {
        for (int u = 0; u < n; u++) {
            if ((mask & (1 << u)) == 0) continue; // u not in mask
            if (dp[mask][u] == Integer.MAX_VALUE / 2) continue;
            for (int v = 0; v < n; v++) {
                if ((mask & (1 << v)) != 0) continue; // v already visited
                int newMask = mask | (1 << v);
                dp[newMask][v] = Math.min(dp[newMask][v], dp[mask][u] + dist[u][v]);
            }
        }
    }
    int ans = Integer.MAX_VALUE;
    for (int u = 1; u < n; u++)
        ans = Math.min(ans, dp[FULL][u] + dist[u][0]); // return to 0
    return ans;
}
// Time: O(n² × 2^n)  Space: O(n × 2^n)
```

---

### Problem 11.2 – Partition to K Equal Sum Subsets
**LeetCode 698:**

```java
public boolean canPartitionKSubsets(int[] nums, int k) {
    int sum = Arrays.stream(nums).sum();
    if (sum % k != 0) return false;
    int target = sum / k;
    Arrays.sort(nums);
    if (nums[nums.length - 1] > target) return false;

    int n = nums.length;
    int[] dp = new int[1 << n]; // dp[mask] = current sum of the "active" bucket
    Arrays.fill(dp, -1);
    dp[0] = 0;

    for (int mask = 0; mask < (1 << n); mask++) {
        if (dp[mask] == -1) continue;
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) continue; // already used
            int newMask = mask | (1 << i);
            if (dp[mask] % target + nums[i] <= target) {
                dp[newMask] = dp[mask] + nums[i];
            }
        }
    }
    return dp[(1 << n) - 1] == sum;
}
// Time: O(n × 2^n)  Space: O(2^n)
```

---

## 12. DP + Stock Problems

All 6 stock problems follow the same state machine pattern.

**State:** `dp[day][held][transactions_used]`

---

### Problem 12.1 – Buy and Sell Stock I (One Transaction)
**LeetCode 121:**

```java
public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE, maxProfit = 0;
    for (int p : prices) {
        minPrice   = Math.min(minPrice, p);
        maxProfit  = Math.max(maxProfit, p - minPrice);
    }
    return maxProfit;
}
```

---

### Problem 12.2 – Buy and Sell Stock II (Unlimited Transactions)
**LeetCode 122:** Collect every upward movement

```java
public int maxProfit(int[] prices) {
    int profit = 0;
    for (int i = 1; i < prices.length; i++)
        profit += Math.max(0, prices[i] - prices[i - 1]);
    return profit;
}
// Greedy: take every positive daily difference
```

---

### Problem 12.3 – Buy and Sell Stock III (At Most 2 Transactions)
**LeetCode 123:**

```java
public int maxProfit(int[] prices) {
    int buy1 = Integer.MIN_VALUE, sell1 = 0;
    int buy2 = Integer.MIN_VALUE, sell2 = 0;
    for (int p : prices) {
        buy1  = Math.max(buy1,  -p);           // best profit after buying once
        sell1 = Math.max(sell1, buy1 + p);     // best profit after selling once
        buy2  = Math.max(buy2,  sell1 - p);    // best profit after buying twice
        sell2 = Math.max(sell2, buy2 + p);     // best profit after selling twice
    }
    return sell2;
}
// State machine: 4 states per day  Time: O(n)  Space: O(1)
```

---

### Problem 12.4 – Buy and Sell Stock IV (At Most K Transactions)
**LeetCode 188:**

```java
public int maxProfit(int k, int[] prices) {
    int n = prices.length;
    if (k >= n / 2) { // unlimited transactions (same as problem II)
        int profit = 0;
        for (int i = 1; i < n; i++) profit += Math.max(0, prices[i] - prices[i-1]);
        return profit;
    }
    int[] buy = new int[k + 1], sell = new int[k + 1];
    Arrays.fill(buy, Integer.MIN_VALUE);
    for (int p : prices) {
        for (int t = k; t >= 1; t--) {
            sell[t] = Math.max(sell[t], buy[t] + p);
            buy[t]  = Math.max(buy[t], sell[t-1] - p);
        }
    }
    return sell[k];
}
// Time: O(n × k)  Space: O(k)
```

---

### Problem 12.5 – Buy and Sell Stock with Cooldown
**LeetCode 309:** After selling, must wait 1 day before buying again

```java
public int maxProfit(int[] prices) {
    int held = Integer.MIN_VALUE, sold = 0, rest = 0;
    for (int p : prices) {
        int prevSold = sold;
        held = Math.max(held, rest - p);  // buy: came from rest state
        sold = held + p;                  // sell today
        rest = Math.max(rest, prevSold);  // cooldown or continue resting
    }
    return Math.max(sold, rest);
}
// States: held (own stock), sold (just sold, cooldown next), rest (no stock, can buy)
// Time: O(n)  Space: O(1)
```

---

### Problem 12.6 – Buy and Sell Stock with Transaction Fee
**LeetCode 714:** Pay fee on each sell

```java
public int maxProfit(int[] prices, int fee) {
    int held = -prices[0], free = 0;
    for (int i = 1; i < prices.length; i++) {
        held = Math.max(held, free - prices[i]);       // buy
        free = Math.max(free, held + prices[i] - fee); // sell and pay fee
    }
    return free;
}
// Time: O(n)  Space: O(1)
```

---

## DP Decision Guide

```
Does the problem ask for:
│
├── Count of ways / combinations?  → DP (not greedy)
│   └── Items used at most once?   → 0/1 Knapsack (reverse inner loop)
│   └── Items reused unlimited?    → Unbounded Knapsack (forward inner loop)
│   └── Order matters?             → Permutation variant (target as outer loop)
│
├── Max/Min value over choices?    → DP
│   └── On a path (grid)?          → 2D Grid DP
│   └── On sequence?              → 1D DP / LIS / Kadane's
│   └── On intervals?             → Interval DP (O(n³))
│   └── On tree?                  → Tree DP (DFS + return array)
│
├── Is it feasible? (yes/no)       → DP with boolean table
│   └── Subset sum, partition?     → 0/1 Knapsack boolean variant
│
├── Two sequences compared?        → LCS family (dp[i][j])
│   └── Edit cost?                 → Edit Distance
│   └── Matching with wildcards?   → Wildcard / Regex DP
│
├── Palindrome problems?           → Expand-around-center OR 2D palindrome DP
│
├── State machine (stock / transactions)?  → Track states (hold, free, cooldown)
│
└── Small n (≤ 20) + subset?       → Bitmask DP (2^n states)
```

---

## Common Interview Questions

| Question | Answer |
|---|---|
| **Memoization vs Tabulation?** | Top-down memoization = recursive + cache; simple to write. Bottom-up tabulation = iterative; better space optimization with rolling array, better cache performance. |
| **How to reduce 2D DP to 1D?** | When `dp[i][j]` only depends on previous row `dp[i-1][...]`. Use two 1D arrays or iterate carefully. For knapsack, reverse inner loop for 0/1, forward for unbounded. |
| **When does greedy work instead of DP?** | When the greedy choice is provably optimal at each step (no need to consider all choices). Interval scheduling, fractional knapsack. 0/1 knapsack requires DP. |
| **Difference: Combinations vs Permutations in coin change?** | Combinations (order independent): items as outer loop. Permutations (order matters): target as outer loop. |
| **How to count inversions?** | Merge sort — count during merge step when right element placed before remaining left elements. |
| **LIS in O(n log n)?** | Maintain `tails[]` array (always sorted): for each element, binary search for first tail ≥ element and replace. |
| **Why interval DP is O(n³)?** | O(n²) subproblems (all intervals [i,j]) × O(n) split points k each. |
| **Bitmask DP when?** | n ≤ 20; need to track exact set of items used (e.g., TSP, assignment problems). |

---

*DSA Dynamic Programming Notes | Pattern-wise | Java | Updated: March 2026*
