# Graphs – Pattern-Wise DSA Notes

> **Language:** Java | **Level:** Interview-ready | **Focus:** Representations, traversals, classic algorithms, pattern recognition

## Pattern Index

| # | Pattern | Algorithm / Technique | Key Problems |
|---|---|---|---|
| 1 | [Representations & Setup](#1-graph-representations--setup) | Adj List, Adj Matrix, Edge List | Building graphs |
| 2 | [BFS](#2-bfs--breadth-first-search) | Queue-based level traversal | Shortest path (unweighted), Islands |
| 3 | [DFS](#3-dfs--depth-first-search) | Stack / Recursion | Connected components, Path existence |
| 4 | [Cycle Detection](#4-cycle-detection) | DFS color / Union-Find | Detect cycle in directed/undirected |
| 5 | [Topological Sort](#5-topological-sort) | Kahn's BFS / DFS post-order | Course Schedule, Build Order |
| 6 | [Shortest Path (Weighted)](#6-shortest-path-weighted) | Dijkstra, Bellman-Ford, SPFA | Network delay, Cheapest flights |
| 7 | [Minimum Spanning Tree](#7-minimum-spanning-tree-mst) | Kruskal (Union-Find), Prim | Min cost to connect, Critical connections |
| 8 | [Union-Find (DSU)](#8-union-find--disjoint-set-union) | Path compression + rank | Number of components, Redundant edge |
| 9 | [Multi-source BFS / 0-1 BFS](#9-multi-source-bfs--0-1-bfs) | BFS from multiple sources | Rotting Oranges, 01 Matrix |
| 10 | [Bipartite Check](#10-bipartite-check) | 2-coloring via BFS/DFS | Is Graph Bipartite? |
| 11 | [Strongly Connected Components](#11-strongly-connected-components) | Kosaraju / Tarjan | SCC, Critical connections (bridges) |
| 12 | [Graph DP / Advanced](#12-graph-dp--advanced) | Floyd-Warshall, Bellman DP | All pairs SP, Cheapest k stops |

---

## 1. Graph Representations & Setup

### Core Terminology

```
Vertex (Node):     entity in the graph
Edge:              connection between two vertices
Directed (Digraph): edges have direction (A → B ≠ B → A)
Undirected:         edges are bidirectional (A — B)
Weighted:           edges have a cost/weight
DAG:               Directed Acyclic Graph (no cycles; used for topological sort)
Degree:            number of edges connected to a vertex
  In-degree:  edges coming IN (directed graphs)
  Out-degree: edges going OUT (directed graphs)
Connected:         path exists between every pair of vertices (undirected)
Strongly Connected: path exists between every pair in BOTH directions (directed)
```

### Adjacency List (Most Common in Interviews)

```java
// Unweighted undirected graph
int n = 6; // number of vertices
List<List<Integer>> adj = new ArrayList<>();
for (int i = 0; i < n; i++) adj.add(new ArrayList<>());

// Add edge (undirected)
void addEdge(int u, int v) {
    adj.get(u).add(v);
    adj.get(v).add(u);
}

// Weighted directed graph
List<List<int[]>> adjW = new ArrayList<>();  // int[] = {neighbor, weight}
for (int i = 0; i < n; i++) adjW.add(new ArrayList<>());

void addWeightedEdge(int u, int v, int w) {
    adjW.get(u).add(new int[]{v, w});
    // For undirected: adjW.get(v).add(new int[]{u, w});
}

// From edge list input (common in problems)
int[][] edges = {{0,1},{1,2},{2,3}};
for (int[] e : edges) addEdge(e[0], e[1]);

// Space: O(V + E)  — each edge stored once (directed) or twice (undirected)
```

### Adjacency Matrix

```java
int[][] matrix = new int[n][n]; // matrix[u][v] = 1 if edge u→v
// Weighted: matrix[u][v] = weight (0 or INF if no edge)
matrix[0][1] = 1;
matrix[1][2] = 1;

// Space: O(V²) — inefficient for sparse graphs; good for dense graphs
// O(1) edge lookup vs O(degree) for adjacency list
```

### Building from Common Input Formats

```java
// Format: n nodes, edges as pairs
// Input: n=5, edges=[[0,1],[1,2],[0,3]]
public List<List<Integer>> buildGraph(int n, int[][] edges) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int[] e : edges) {
        adj.get(e[0]).add(e[1]);
        adj.get(e[1]).add(e[0]); // omit for directed
    }
    return adj;
}

// Grid as implicit graph: (row, col) ↔ node
// Move in 4 directions
int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
for (int[] d : dirs) {
    int nr = r + d[0], nc = c + d[1];
    if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
        // valid neighbor at (nr, nc)
    }
}
```

---

## 2. BFS – Breadth-First Search

**How it works:** Explore level by level using a queue.
- Visits all neighbors at distance 1 before distance 2, etc.
- **Guarantees shortest path in unweighted graphs.**

**Template:**
```java
boolean[] visited = new boolean[n];
Queue<Integer> queue = new LinkedList<>();
queue.offer(start);
visited[start] = true;

while (!queue.isEmpty()) {
    int node = queue.poll();
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor]) {
            visited[neighbor] = true;
            queue.offer(neighbor);
        }
    }
}
// Time: O(V + E)  Space: O(V)
```

---

### Problem 2.1 – Number of Islands
**LeetCode 200:** Count connected components of '1's in a 2D grid

```java
public int numIslands(char[][] grid) {
    int rows = grid.length, cols = grid[0].length, count = 0;
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};

    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == '1') {
                count++;
                // BFS to sink the entire island
                Queue<int[]> q = new LinkedList<>();
                q.offer(new int[]{r, c});
                grid[r][c] = '0'; // mark visited by sinking
                while (!q.isEmpty()) {
                    int[] curr = q.poll();
                    for (int[] d : dirs) {
                        int nr = curr[0] + d[0], nc = curr[1] + d[1];
                        if (nr >= 0 && nr < rows && nc >= 0 && nc < cols
                                && grid[nr][nc] == '1') {
                            grid[nr][nc] = '0';
                            q.offer(new int[]{nr, nc});
                        }
                    }
                }
            }
        }
    }
    return count;
}
// Time: O(m × n)  Space: O(min(m,n)) — queue holds at most one diagonal's worth
```

---

### Problem 2.2 – Shortest Path in Unweighted Graph (BFS)
**LeetCode 1091 – Shortest Path in Binary Matrix**

```java
public int shortestPathBinaryMatrix(int[][] grid) {
    int n = grid.length;
    if (grid[0][0] == 1 || grid[n-1][n-1] == 1) return -1;
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0},{1,1},{1,-1},{-1,1},{-1,-1}};

    Queue<int[]> q = new LinkedList<>();
    q.offer(new int[]{0, 0, 1}); // {row, col, dist}
    grid[0][0] = 1; // mark visited

    while (!q.isEmpty()) {
        int[] cur = q.poll();
        if (cur[0] == n-1 && cur[1] == n-1) return cur[2];
        for (int[] d : dirs) {
            int nr = cur[0]+d[0], nc = cur[1]+d[1];
            if (nr >= 0 && nr < n && nc >= 0 && nc < n && grid[nr][nc] == 0) {
                grid[nr][nc] = 1; // mark visited
                q.offer(new int[]{nr, nc, cur[2]+1});
            }
        }
    }
    return -1;
}
// BFS guarantees first time we reach destination = shortest path
// Time: O(n²)  Space: O(n²)
```

---

### Problem 2.3 – Word Ladder
**LeetCode 127:** Min transformations from beginWord → endWord, one character at a time

```java
public int ladderLength(String begin, String end, List<String> wordList) {
    Set<String> wordSet = new HashSet<>(wordList);
    if (!wordSet.contains(end)) return 0;

    Queue<String> q = new LinkedList<>();
    q.offer(begin);
    int steps = 1;

    while (!q.isEmpty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            String word = q.poll();
            char[] arr = word.toCharArray();
            for (int j = 0; j < arr.length; j++) {
                char orig = arr[j];
                for (char ch = 'a'; ch <= 'z'; ch++) {
                    arr[j] = ch;
                    String next = new String(arr);
                    if (next.equals(end)) return steps + 1;
                    if (wordSet.contains(next)) {
                        wordSet.remove(next); // remove = mark visited
                        q.offer(next);
                    }
                }
                arr[j] = orig;
            }
        }
        steps++;
    }
    return 0;
}
// BFS level = transformation steps  Time: O(M² × N) where M=word length, N=wordList size
```

---

## 3. DFS – Depth-First Search

**How it works:** Explore as deep as possible along one path before backtracking.
- Uses recursion (implicit stack) or explicit stack
- Good for: path existence, connected components, cycle detection, topological sort

**Template (Recursive):**
```java
boolean[] visited = new boolean[n];

void dfs(int node) {
    visited[node] = true;
    // process node
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor]) {
            dfs(neighbor);
        }
    }
    // post-order processing here (useful for topo sort, SCC)
}

// Iterative DFS (avoids stack overflow for large graphs)
void dfsIterative(int start) {
    boolean[] visited = new boolean[n];
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(start);
    while (!stack.isEmpty()) {
        int node = stack.pop();
        if (visited[node]) continue;
        visited[node] = true;
        for (int neighbor : adj.get(node))
            if (!visited[neighbor]) stack.push(neighbor);
    }
}
// Time: O(V + E)  Space: O(V)
```

---

### Problem 3.1 – Number of Connected Components
**LeetCode 323:** Count connected components in undirected graph

```java
public int countComponents(int n, int[][] edges) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int[] e : edges) { adj.get(e[0]).add(e[1]); adj.get(e[1]).add(e[0]); }

    boolean[] visited = new boolean[n];
    int components = 0;
    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfs(i, adj, visited);
            components++;
        }
    }
    return components;
}

void dfs(int node, List<List<Integer>> adj, boolean[] visited) {
    visited[node] = true;
    for (int nb : adj.get(node))
        if (!visited[nb]) dfs(nb, adj, visited);
}
// Alternatively: Union-Find is O(α(n)) per operation (see Section 8)
// DFS: Time O(V+E)  Space O(V)
```

---

### Problem 3.2 – Clone Graph
**LeetCode 133:** Deep clone a graph

```java
Map<Node, Node> cloned = new HashMap<>();

public Node cloneGraph(Node node) {
    if (node == null) return null;
    if (cloned.containsKey(node)) return cloned.get(node);
    Node copy = new Node(node.val);
    cloned.put(node, copy);
    for (Node neighbor : node.neighbors)
        copy.neighbors.add(cloneGraph(neighbor));
    return copy;
}
// DFS with HashMap as visited + mapping original → clone
// Time: O(V + E)  Space: O(V)
```

---

### Problem 3.3 – Path Existence / All Paths
**LeetCode 797 – All Paths Source to Target (DAG)**

```java
List<List<Integer>> result = new ArrayList<>();

public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
    dfs(graph, 0, new ArrayList<>(List.of(0)));
    return result;
}

void dfs(int[][] graph, int node, List<Integer> path) {
    if (node == graph.length - 1) { result.add(new ArrayList<>(path)); return; }
    for (int next : graph[node]) {
        path.add(next);
        dfs(graph, next, path);
        path.remove(path.size() - 1); // backtrack
    }
}
// DAG: no need for visited array (no cycles). Uses backtracking.
// Time: O(2^V × V)  Space: O(V) stack
```

---

## 4. Cycle Detection

### Undirected Graph – DFS with Parent Tracking

```java
// Returns true if cycle exists
boolean hasCycleUndirected(List<List<Integer>> adj, int n) {
    boolean[] visited = new boolean[n];
    for (int i = 0; i < n; i++)
        if (!visited[i] && dfsUndirected(adj, i, -1, visited)) return true;
    return false;
}

boolean dfsUndirected(List<List<Integer>> adj, int node, int parent, boolean[] visited) {
    visited[node] = true;
    for (int nb : adj.get(node)) {
        if (!visited[nb]) {
            if (dfsUndirected(adj, nb, node, visited)) return true;
        } else if (nb != parent) return true; // back edge to non-parent = cycle
    }
    return false;
}
```

### Directed Graph – DFS with 3-Color Marking

```java
// 0 = WHITE (unvisited), 1 = GRAY (in current path), 2 = BLACK (done)
int[] color;

boolean hasCycleDirected(List<List<Integer>> adj, int n) {
    color = new int[n]; // all start WHITE
    for (int i = 0; i < n; i++)
        if (color[i] == 0 && dfsDir(adj, i)) return true;
    return false;
}

boolean dfsDir(List<List<Integer>> adj, int node) {
    color[node] = 1; // GRAY: in current DFS path
    for (int nb : adj.get(node)) {
        if (color[nb] == 1) return true;  // back edge to GRAY = cycle!
        if (color[nb] == 0 && dfsDir(adj, nb)) return true;
    }
    color[node] = 2; // BLACK: done
    return false;
}
// Time: O(V + E)  Space: O(V)
// KEY: Gray = currently on stack. If we see a gray node, we have a back edge = cycle.
```

---

### Problem 4.1 – Course Schedule
**LeetCode 207:** Can you finish all courses given prerequisites? (cycle in directed graph?)

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    for (int[] pre : prerequisites) adj.get(pre[1]).add(pre[0]); // pre[1] → pre[0]

    int[] color = new int[numCourses];
    for (int i = 0; i < numCourses; i++)
        if (color[i] == 0 && hasCycle(adj, i, color)) return false;
    return true;
}

boolean hasCycle(List<List<Integer>> adj, int node, int[] color) {
    color[node] = 1;
    for (int nb : adj.get(node)) {
        if (color[nb] == 1) return true;
        if (color[nb] == 0 && hasCycle(adj, nb, color)) return true;
    }
    color[node] = 2;
    return false;
}
```

---

## 5. Topological Sort

**When to use:** Ordering tasks with dependencies (DAG only — directed acyclic graph).
**Applications:** Build order, course scheduling, dependency resolution.

### Kahn's Algorithm (BFS, In-degree)

```java
public int[] topoSort(int n, List<List<Integer>> adj) {
    int[] inDegree = new int[n];
    for (int u = 0; u < n; u++)
        for (int v : adj.get(u)) inDegree[v]++;

    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < n; i++)
        if (inDegree[i] == 0) q.offer(i); // all nodes with no dependencies

    int[] order = new int[n];
    int idx = 0;
    while (!q.isEmpty()) {
        int node = q.poll();
        order[idx++] = node;
        for (int nb : adj.get(node)) {
            inDegree[nb]--;
            if (inDegree[nb] == 0) q.offer(nb);
        }
    }
    // idx < n means cycle exists (some nodes never reached in-degree 0)
    return idx == n ? order : new int[0];
}
// Time: O(V + E)  Space: O(V)
```

### DFS Post-Order (Stack)

```java
public int[] topoSortDFS(int n, List<List<Integer>> adj) {
    boolean[] visited = new boolean[n];
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < n; i++)
        if (!visited[i]) dfs(adj, i, visited, stack);

    int[] order = new int[n];
    for (int i = 0; i < n; i++) order[i] = stack.pop();
    return order;
}

void dfs(List<List<Integer>> adj, int node, boolean[] visited, Deque<Integer> stack) {
    visited[node] = true;
    for (int nb : adj.get(node))
        if (!visited[nb]) dfs(adj, nb, visited, stack);
    stack.push(node); // push AFTER all descendants → post-order
}
// Reverse of post-order = topological order
// Time: O(V + E)  Space: O(V)
```

---

### Problem 5.1 – Course Schedule II
**LeetCode 210:** Return topological order of courses

```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    int[] inDegree = new int[numCourses];
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    for (int[] p : prerequisites) {
        adj.get(p[1]).add(p[0]);
        inDegree[p[0]]++;
    }
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) if (inDegree[i] == 0) q.offer(i);

    int[] order = new int[numCourses];
    int idx = 0;
    while (!q.isEmpty()) {
        int course = q.poll();
        order[idx++] = course;
        for (int next : adj.get(course))
            if (--inDegree[next] == 0) q.offer(next);
    }
    return idx == numCourses ? order : new int[0];
}
```

---

### Problem 5.2 – Alien Dictionary
**LeetCode 269:** Derive character ordering from sorted alien words

```java
public String alienOrder(String[] words) {
    Map<Character, Set<Character>> adj = new HashMap<>();
    Map<Character, Integer> inDegree = new HashMap<>();
    for (String w : words) for (char c : w.toCharArray()) { inDegree.putIfAbsent(c, 0); adj.putIfAbsent(c, new HashSet<>()); }

    for (int i = 0; i < words.length - 1; i++) {
        String w1 = words[i], w2 = words[i + 1];
        int len = Math.min(w1.length(), w2.length());
        if (w1.length() > w2.length() && w1.startsWith(w2)) return ""; // invalid
        for (int j = 0; j < len; j++) {
            if (w1.charAt(j) != w2.charAt(j)) {
                char from = w1.charAt(j), to = w2.charAt(j);
                if (!adj.get(from).contains(to)) {
                    adj.get(from).add(to);
                    inDegree.merge(to, 1, Integer::sum);
                }
                break;
            }
        }
    }
    // Kahn's BFS
    Queue<Character> q = new LinkedList<>();
    inDegree.forEach((c, d) -> { if (d == 0) q.offer(c); });
    StringBuilder sb = new StringBuilder();
    while (!q.isEmpty()) {
        char c = q.poll(); sb.append(c);
        for (char nb : adj.get(c))
            if (inDegree.merge(nb, -1, Integer::sum) == 0) q.offer(nb);
    }
    return sb.length() == inDegree.size() ? sb.toString() : "";
}
```

---

## 6. Shortest Path (Weighted)

### Dijkstra's Algorithm – Single Source, Non-Negative Weights

```
Uses a min-heap (priority queue) to always process the nearest unvisited node.
Greedy: once a node is popped with distance d, d is its true shortest distance.
Does NOT work with negative weights.
```

```java
public int[] dijkstra(int src, int n, List<List<int[]>> adj) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    // PQ: {distance, node}
    PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
    pq.offer(new int[]{0, src});

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int d = curr[0], node = curr[1];

        if (d > dist[node]) continue; // stale entry — skip

        for (int[] edge : adj.get(node)) {
            int nb = edge[0], w = edge[1];
            if (dist[node] + w < dist[nb]) {
                dist[nb] = dist[node] + w;
                pq.offer(new int[]{dist[nb], nb});
            }
        }
    }
    return dist;
}
// Time: O((V + E) log V)  Space: O(V + E)
```

---

### Problem 6.1 – Network Delay Time
**LeetCode 743:** Time for signal to reach all nodes; -1 if unreachable

```java
public int networkDelayTime(int[][] times, int n, int k) {
    List<List<int[]>> adj = new ArrayList<>();
    for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());
    for (int[] t : times) adj.get(t[0]).add(new int[]{t[1], t[2]});

    int[] dist = dijkstra(k, n + 1, adj); // 1-indexed → use n+1 size

    int maxDist = 0;
    for (int i = 1; i <= n; i++) {
        if (dist[i] == Integer.MAX_VALUE) return -1;
        maxDist = Math.max(maxDist, dist[i]);
    }
    return maxDist;
}
```

---

### Problem 6.2 – Cheapest Flights Within K Stops
**LeetCode 787:** Dijkstra variant — states include (cost, node, stops_used)

```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    List<List<int[]>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (int[] f : flights) adj.get(f[0]).add(new int[]{f[1], f[2]});

    // PQ: {cost, node, stops_remaining}
    PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
    pq.offer(new int[]{0, src, k + 1}); // k+1 stops allowed

    int[] visited = new int[n]; // best stops remaining when visiting each node
    Arrays.fill(visited, -1);

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int cost = curr[0], node = curr[1], stops = curr[2];
        if (node == dst) return cost;
        if (stops == 0) continue;
        if (visited[node] >= stops) continue; // already visited with more stops left
        visited[node] = stops;
        for (int[] e : adj.get(node))
            pq.offer(new int[]{cost + e[1], e[0], stops - 1});
    }
    return -1;
}
```

---

### Bellman-Ford – Handles Negative Weights

```java
public int[] bellmanFord(int src, int n, int[][] edges) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    // Relax ALL edges V-1 times
    for (int i = 0; i < n - 1; i++) {
        for (int[] e : edges) { // e = {u, v, weight}
            int u = e[0], v = e[1], w = e[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v])
                dist[v] = dist[u] + w;
        }
    }
    // V-th relaxation: if still improves → negative cycle exists
    for (int[] e : edges)
        if (dist[e[0]] != Integer.MAX_VALUE && dist[e[0]] + e[2] < dist[e[1]])
            throw new RuntimeException("Negative cycle detected");

    return dist;
}
// Time: O(V × E)  Space: O(V)
// Use when: negative weights, detect negative cycles, Cheapest flights (DP variant)
```

---

## 7. Minimum Spanning Tree (MST)

**MST:** Subset of edges connecting all vertices with minimum total weight.
- Exactly V-1 edges
- No cycles
- All vertices connected

### Kruskal's Algorithm (Sort edges + Union-Find)

```java
public int kruskalMST(int n, int[][] edges) {
    // Sort edges by weight ascending
    Arrays.sort(edges, Comparator.comparingInt(e -> e[2]));
    UnionFind uf = new UnionFind(n);
    int totalCost = 0, edgesUsed = 0;

    for (int[] e : edges) {
        int u = e[0], v = e[1], w = e[2];
        if (uf.union(u, v)) { // only add edge if it connects two different components
            totalCost += w;
            edgesUsed++;
            if (edgesUsed == n - 1) break; // MST complete
        }
    }
    return edgesUsed == n - 1 ? totalCost : -1; // -1 if graph not connected
}
// Time: O(E log E)  Space: O(V)
```

---

### Problem 7.1 – Min Cost to Connect All Points
**LeetCode 1584:** Each node is a 2D point; cost = Manhattan distance

```java
public int minCostConnectPoints(int[][] points) {
    int n = points.length;
    // Build all edges (n² edges for complete graph)
    List<int[]> edges = new ArrayList<>();
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++)
            edges.add(new int[]{i, j,
                Math.abs(points[i][0]-points[j][0]) + Math.abs(points[i][1]-points[j][1])});

    edges.sort(Comparator.comparingInt(e -> e[2]));
    UnionFind uf = new UnionFind(n);
    int cost = 0, count = 0;
    for (int[] e : edges) {
        if (uf.union(e[0], e[1])) { cost += e[2]; if (++count == n-1) break; }
    }
    return cost;
}
// Prim's is better here: O(n² log n) without building all edges explicitly
```

---

### Problem 7.2 – Critical Connections (Bridges)
**LeetCode 1192:** Find all edges whose removal disconnects the graph (Tarjan's bridge algorithm)

```java
List<List<Integer>> bridges = new ArrayList<>();
int timer = 0;

public List<List<Integer>> criticalConnections(int n, List<List<Integer>> connections) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (List<Integer> c : connections) { adj.get(c.get(0)).add(c.get(1)); adj.get(c.get(1)).add(c.get(0)); }

    int[] disc = new int[n], low = new int[n];
    Arrays.fill(disc, -1);
    dfsBridge(adj, 0, -1, disc, low);
    return bridges;
}

void dfsBridge(List<List<Integer>> adj, int u, int parent, int[] disc, int[] low) {
    disc[u] = low[u] = timer++;
    for (int v : adj.get(u)) {
        if (disc[v] == -1) { // not visited
            dfsBridge(adj, v, u, disc, low);
            low[u] = Math.min(low[u], low[v]);
            if (low[v] > disc[u]) // v cannot reach u or above without u-v edge
                bridges.add(List.of(u, v));
        } else if (v != parent) {
            low[u] = Math.min(low[u], disc[v]); // back edge
        }
    }
}
// disc[u] = discovery time  low[u] = min disc reachable from subtree of u
// Bridge: low[v] > disc[u] means v's subtree cannot reach u without edge u-v
// Time: O(V + E)  Space: O(V)
```

---

## 8. Union-Find / Disjoint Set Union

**Core operations:**
- `find(x)`: find root/representative of x's component
- `union(x, y)`: merge two components
- With **path compression** + **union by rank**: near O(1) per operation (O(α(n)))

```java
class UnionFind {
    int[] parent, rank;

    UnionFind(int n) {
        parent = new int[n];
        rank   = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    int find(int x) {
        if (parent[x] != x)
            parent[x] = find(parent[x]); // path compression
        return parent[x];
    }

    boolean union(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return false; // already same component
        // Union by rank: attach smaller tree under larger
        if      (rank[px] < rank[py]) parent[px] = py;
        else if (rank[px] > rank[py]) parent[py] = px;
        else                          { parent[py] = px; rank[px]++; }
        return true;
    }

    boolean connected(int x, int y) { return find(x) == find(y); }
}
// Time: O(α(n)) per op — practically O(1)  Space: O(n)
```

---

### Problem 8.1 – Redundant Connection
**LeetCode 684:** Find the edge that creates a cycle in an undirected graph

```java
public int[] findRedundantConnection(int[][] edges) {
    UnionFind uf = new UnionFind(edges.length + 1);
    for (int[] e : edges) {
        if (!uf.union(e[0], e[1])) return e; // already connected → this edge is redundant
    }
    return new int[0];
}
// First edge that connects two already-connected nodes creates cycle
// Time: O(n α(n))  Space: O(n)
```

---

### Problem 8.2 – Number of Provinces
**LeetCode 547:** Connected components via adjacency matrix

```java
public int findCircleNum(int[][] isConnected) {
    int n = isConnected.length;
    UnionFind uf = new UnionFind(n);
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++)
            if (isConnected[i][j] == 1) uf.union(i, j);

    Set<Integer> roots = new HashSet<>();
    for (int i = 0; i < n; i++) roots.add(uf.find(i));
    return roots.size();
}
```

---

## 9. Multi-Source BFS / 0-1 BFS

### Multi-Source BFS
**Start BFS from multiple sources simultaneously**. All sources begin at distance 0.

---

### Problem 9.1 – Rotting Oranges
**LeetCode 994:** All rotten oranges rot adjacent fresh ones each minute

```java
public int orangesRotting(int[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    Queue<int[]> q = new LinkedList<>();
    int fresh = 0;
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};

    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == 2) q.offer(new int[]{r, c}); // all rotten = sources
            else if (grid[r][c] == 1) fresh++;
        }

    int minutes = 0;
    while (!q.isEmpty() && fresh > 0) {
        minutes++;
        int size = q.size();
        for (int i = 0; i < size; i++) {
            int[] cur = q.poll();
            for (int[] d : dirs) {
                int nr = cur[0]+d[0], nc = cur[1]+d[1];
                if (nr>=0 && nr<rows && nc>=0 && nc<cols && grid[nr][nc]==1) {
                    grid[nr][nc] = 2;
                    fresh--;
                    q.offer(new int[]{nr, nc});
                }
            }
        }
    }
    return fresh == 0 ? minutes : -1;
}
// Time: O(m × n)  Space: O(m × n)
```

---

### Problem 9.2 – 01 Matrix (Distance to Nearest 0)
**LeetCode 542:** Multi-source BFS from all 0s

```java
public int[][] updateMatrix(int[][] mat) {
    int m = mat.length, n = mat[0].length;
    int[][] dist = new int[m][n];
    Queue<int[]> q = new LinkedList<>();
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};

    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++) {
            if (mat[i][j] == 0) q.offer(new int[]{i, j});
            else dist[i][j] = Integer.MAX_VALUE;
        }

    while (!q.isEmpty()) {
        int[] cur = q.poll();
        for (int[] d : dirs) {
            int nr = cur[0]+d[0], nc = cur[1]+d[1];
            if (nr>=0&&nr<m&&nc>=0&&nc<n && dist[nr][nc] > dist[cur[0]][cur[1]] + 1) {
                dist[nr][nc] = dist[cur[0]][cur[1]] + 1;
                q.offer(new int[]{nr, nc});
            }
        }
    }
    return dist;
}
// Time: O(m × n)  Space: O(m × n)
```

### 0-1 BFS (Deque)
When edges have weights of only 0 or 1 — use **Deque**: weight-0 edges add to **front**, weight-1 add to **back**.

```java
public int minCost01BFS(int[][] grid) {
    // grid: 0 = no cost to move in arrow direction, 1 = must change (cost 1)
    int m = grid.length, n = grid[0].length;
    int[][] dist = new int[m][n];
    for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
    dist[0][0] = 0;
    Deque<int[]> dq = new ArrayDeque<>();
    dq.offerFirst(new int[]{0, 0});
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};

    while (!dq.isEmpty()) {
        int[] cur = dq.pollFirst();
        int r = cur[0], c = cur[1];
        for (int d = 0; d < 4; d++) {
            int nr = r + dirs[d][0], nc = c + dirs[d][1];
            if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
            int cost = (grid[r][c] == d) ? 0 : 1; // 0 if arrow matches direction
            if (dist[r][c] + cost < dist[nr][nc]) {
                dist[nr][nc] = dist[r][c] + cost;
                if (cost == 0) dq.offerFirst(new int[]{nr, nc});
                else           dq.offerLast(new int[]{nr, nc});
            }
        }
    }
    return dist[m-1][n-1];
}
// Time: O(V + E)  Space: O(V) — faster than Dijkstra for 0-1 weights
```

---

## 10. Bipartite Check

A graph is **bipartite** if vertices can be split into two sets such that every edge crosses between sets (2-colorable; no odd-length cycles).

```java
public boolean isBipartite(int[][] graph) {
    int n = graph.length;
    int[] color = new int[n]; // 0=uncolored, 1=red, -1=blue
    for (int i = 0; i < n; i++) {
        if (color[i] == 0 && !bfsColor(graph, i, color)) return false;
    }
    return true;
}

boolean bfsColor(int[][] graph, int start, int[] color) {
    Queue<Integer> q = new LinkedList<>();
    q.offer(start);
    color[start] = 1;
    while (!q.isEmpty()) {
        int node = q.poll();
        for (int nb : graph[node]) {
            if (color[nb] == 0) {
                color[nb] = -color[node]; // alternate color
                q.offer(nb);
            } else if (color[nb] == color[node]) {
                return false; // same color on both ends → not bipartite
            }
        }
    }
    return true;
}
// Bipartite ⟺ No odd-length cycles ⟺ 2-colorable
// Time: O(V + E)  Space: O(V)
```

---

## 11. Strongly Connected Components

**SCC:** Maximal set of vertices where every vertex is reachable from every other.

### Kosaraju's Algorithm (2-pass DFS)

```java
void kosaraju(List<List<Integer>> adj, int n) {
    boolean[] visited = new boolean[n];
    Deque<Integer> stack = new ArrayDeque<>();

    // Pass 1: DFS on original graph; push to stack in finish order
    for (int i = 0; i < n; i++)
        if (!visited[i]) dfs1(adj, i, visited, stack);

    // Build reverse graph
    List<List<Integer>> radj = new ArrayList<>();
    for (int i = 0; i < n; i++) radj.add(new ArrayList<>());
    for (int u = 0; u < n; u++)
        for (int v : adj.get(u)) radj.get(v).add(u);

    // Pass 2: DFS on REVERSE graph in finish-time order; each DFS = one SCC
    Arrays.fill(visited, false);
    int sccCount = 0;
    while (!stack.isEmpty()) {
        int node = stack.pop();
        if (!visited[node]) {
            dfs2(radj, node, visited);
            sccCount++;
        }
    }
    System.out.println("SCCs: " + sccCount);
}

void dfs1(List<List<Integer>> adj, int u, boolean[] vis, Deque<Integer> stack) {
    vis[u] = true;
    for (int v : adj.get(u)) if (!vis[v]) dfs1(adj, v, vis, stack);
    stack.push(u); // push after all descendants
}

void dfs2(List<List<Integer>> adj, int u, boolean[] vis) {
    vis[u] = true;
    for (int v : adj.get(u)) if (!vis[v]) dfs2(adj, v, vis);
}
// Time: O(V + E)  Space: O(V + E) for reversed graph
```

---

## 12. Graph DP / Advanced

### Floyd-Warshall – All Pairs Shortest Path

```java
public int[][] floydWarshall(int n, int[][] edges) {
    int INF = Integer.MAX_VALUE / 2;
    int[][] dist = new int[n][n];
    for (int[] row : dist) Arrays.fill(row, INF);
    for (int i = 0; i < n; i++) dist[i][i] = 0;
    for (int[] e : edges) { dist[e[0]][e[1]] = e[2]; /* for undirected: dist[e[1]][e[0]] = e[2]; */ }

    // Try every intermediate vertex k
    for (int k = 0; k < n; k++)
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                if (dist[i][k] + dist[k][j] < dist[i][j])
                    dist[i][j] = dist[i][k] + dist[k][j];
    // Negative cycle: dist[i][i] < 0 for some i
    return dist;
}
// Time: O(V³)  Space: O(V²)
// Use when: small V (≤ 500); all-pairs; transitive closure
```

---

### Problem 12.1 – Cheapest Flights with K Stops (Bellman-Ford / DP)

```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    // Bellman-Ford: relax edges at most k+1 times (k stops = k+1 edges)
    int[] cost = new int[n];
    Arrays.fill(cost, Integer.MAX_VALUE);
    cost[src] = 0;

    for (int i = 0; i <= k; i++) {
        int[] temp = Arrays.copyOf(cost, n); // snapshot BEFORE this round
        for (int[] f : flights) {
            int u = f[0], v = f[1], w = f[2];
            if (cost[u] != Integer.MAX_VALUE && cost[u] + w < temp[v])
                temp[v] = cost[u] + w;
        }
        cost = temp;
    }
    return cost[dst] == Integer.MAX_VALUE ? -1 : cost[dst];
}
// KEY: copy before each round to prevent using more than k+1 edges in one round
// Time: O(k × E)  Space: O(V)
```

---

## Graph Algorithm Decision Guide

```
Type of problem?
│
├── Traverse / Explore:
│   ├── Shortest path (unweighted)?     → BFS
│   ├── Any path / all paths / DFS tree? → DFS
│   └── Connected components?           → DFS or Union-Find
│
├── Cycle detection:
│   ├── Undirected graph?               → DFS with parent tracking
│   └── Directed graph?                 → DFS 3-color (WHITE/GRAY/BLACK)
│
├── Task ordering / dependencies?       → Topological Sort (Kahn's BFS or DFS post-order)
│
├── Shortest Path:
│   ├── Unweighted?                     → BFS (O(V+E))
│   ├── Non-negative weights?           → Dijkstra (O((V+E) log V))
│   ├── Negative weights / neg cycle?   → Bellman-Ford (O(VE))
│   ├── 0 or 1 weights only?            → 0-1 BFS / Deque (O(V+E))
│   └── All pairs?                      → Floyd-Warshall (O(V³))
│
├── Minimum Spanning Tree:
│   ├── Sparse graph?                   → Kruskal (sort edges + Union-Find)
│   └── Dense graph?                    → Prim (adj matrix version O(V²))
│
├── Group / component membership?       → Union-Find (DSU)
│
├── Multiple starting sources?          → Multi-Source BFS
│
├── 2-coloring / scheduling?            → Bipartite check
│
├── Find bridges / articulation points? → Tarjan (disc[], low[])
│
└── Strongly connected components?      → Kosaraju (2-pass DFS) or Tarjan
```

---

## Complexity Reference

| Algorithm | Time | Space | Works on |
|---|---|---|---|
| BFS | O(V + E) | O(V) | Unweighted; undir/dir |
| DFS | O(V + E) | O(V) | Any graph |
| Dijkstra | O((V+E) log V) | O(V) | Non-negative weights |
| Bellman-Ford | O(V × E) | O(V) | Any weights; neg cycle detect |
| Floyd-Warshall | O(V³) | O(V²) | All-pairs; any weights |
| 0-1 BFS | O(V + E) | O(V) | Weights 0 or 1 only |
| Kruskal MST | O(E log E) | O(V) | Undirected weighted |
| Prim MST | O((V+E) log V) | O(V) | Undirected weighted |
| Topological Sort | O(V + E) | O(V) | DAG only |
| Union-Find | O(α(n)) | O(V) | Component queries |
| Kosaraju SCC | O(V + E) | O(V + E) | Directed graph |
| Tarjan SCC/Bridge | O(V + E) | O(V) | Directed/Undirected |

---

## Common Interview Questions

| Question | Answer |
|---|---|
| **BFS vs DFS?** | BFS: shortest path in unweighted graphs; level-order; uses queue. DFS: path existence, cycles, topo sort; uses stack/recursion. BFS better for shortest path; DFS less memory for deep-narrow graphs. |
| **Why Dijkstra fails with negative weights?** | Greedy assumption: once a node is popped, its distance is final. Negative edge can later offer a shorter path to an already-settled node → use Bellman-Ford instead. |
| **Union-Find vs DFS for components?** | Union-Find: O(α(n)) per query, good for dynamic edge additions. DFS: O(V+E) one-time; simpler for static graphs. |
| **What is a DAG?** | Directed Acyclic Graph. Required for topological sort. Any DFS on a DAG produces no back edges. |
| **What is a bridge/articulation point?** | Bridge: edge whose removal disconnects the graph. Articulation point: vertex whose removal disconnects. Found using Tarjan's algorithm with disc[] and low[]. |
| **Topological sort uniqueness?** | Unique only if the DAG is a single chain. Multiple valid orderings exist when multiple nodes have in-degree 0 simultaneously. |
| **When to use Union-Find vs BFS/DFS?** | Union-Find: detecting cycles (Kruskal), dynamic connectivity, Redundant Connection. BFS/DFS: finding actual paths, component structure, when you need traversal order. |
| **What does low[] mean in Tarjan's?** | `low[u]` = minimum discovery time reachable from subtree of u via back edges. If `low[v] > disc[u]`, edge u-v is a bridge (v cannot reach u without this edge). |

---

*DSA Graph Notes | Pattern-wise | Java | Updated: March 2026*
