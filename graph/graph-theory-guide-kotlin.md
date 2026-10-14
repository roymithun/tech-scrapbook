# Graph Theory: A Practical Guide (Kotlin Edition)

A working reference for the core graph algorithms that show up again and again in coding interviews — with explanations and Kotlin implementations for each.

## Contents

1. [Key Terminology](#1-key-terminology)
2. [Representing a Graph](#2-representing-a-graph)
3. [Traversal: BFS & DFS](#3-traversal-bfs--dfs)
4. [Topological Sort](#4-topological-sort)
5. [Shortest Path Algorithms](#5-shortest-path-algorithms)
6. [Union-Find (Disjoint Set)](#6-union-find-disjoint-set)
7. [Minimum Spanning Tree](#7-minimum-spanning-tree)
8. [Cycle Detection](#8-cycle-detection)
9. [Bipartite Graphs](#9-bipartite-graphs)
10. [Bridges: Tarjan's Algorithm](#10-bridges-tarjans-algorithm)
11. [Eulerian Paths: Hierholzer's Algorithm](#11-eulerian-paths-hierholzers-algorithm)
12. [Bitmask DP on Graphs](#12-bitmask-dp-on-graphs)

### Tier Key

Sections are grouped by how often they tend to appear in interviews (percentages are rough estimates):

- **Tier 1: Core Fundamentals (~70% of problems)** — sections 2, 3, 4, 6, 8, 9
- **Tier 2: Weighted Graphs & Shortest Paths (~20% of problems)** — sections 5, 7
- **Tier 3: Advanced Niche Topics (~10% of problems)** — sections 10, 11, 12

---

## 1. Key Terminology

- **Vertex (node)** — a single point in the graph.
- **Edge** — a connection between two vertices, which may be **directed** (one-way) or **undirected** (two-way).
- **Weighted graph** — edges carry a cost/distance; **unweighted** edges are all treated as cost 1.
- **Degree** — number of edges touching a vertex. Directed graphs split this into **in-degree** and **out-degree**.
- **Path** — a sequence of vertices connected by edges.
- **Cycle** — a path that starts and ends at the same vertex without repeating edges.
- **Connected graph** — every vertex is reachable from every other vertex (undirected case).
- **DAG** — a Directed Acyclic Graph: directed edges, no cycles. Required for topological sorting.

---

## 2. Representing a Graph

> **Tier 1: Core Fundamentals**

Two standard representations, each with different trade-offs:

- **Adjacency Matrix** — an `n x n` grid where `matrix[u][v] = 1` (or a weight) if an edge exists. Fast `O(1)` edge lookups, but `O(n²)` space — wasteful for sparse graphs.

- **Adjacency List** — for each vertex, store a list of its neighbors. `O(n + e)` space, and the standard choice for most interview problems.

```kotlin
// Adjacency Matrix
fun buildAdjacencyMatrix(n: Int, edges: List<IntArray>): Array<IntArray> {
    val matrix = Array(n) { IntArray(n) }
    for ((u, v) in edges.map { it[0] to it[1] }) {
        matrix[u][v] = 1
        matrix[v][u] = 1 // omit this line for a directed graph
    }
    return matrix
}

// Adjacency List
fun buildAdjacencyList(n: Int, edges: List<IntArray>): List<MutableList<Int>> {
    val adj = List(n) { mutableListOf<Int>() }
    for ((u, v) in edges.map { it[0] to it[1] }) {
        adj[u].add(v)
        adj[v].add(u) // omit this line for a directed graph
    }
    return adj
}
```

**Practice:**
- [LC 133: Clone Graph](https://leetcode.com/problems/clone-graph/) *(Medium)*
- [LC 797: All Paths From Source to Target](https://leetcode.com/problems/all-paths-from-source-to-target/) *(Medium)*
- [LC 207: Course Schedule](https://leetcode.com/problems/course-schedule/) *(Medium)*
- [LC 1557: Minimum Number of Vertices to Reach All Nodes](https://leetcode.com/problems/minimum-number-of-vertices-to-reach-all-nodes/) *(Medium)*
- [LC 1466: Reorder Routes to Make All Paths Lead to the City Zero](https://leetcode.com/problems/reorder-routes-to-make-all-paths-lead-to-the-city-zero/) *(Medium)*

---

## 3. Traversal: BFS & DFS

> **Tier 1: Core Fundamentals**

- **BFS (Breadth-First Search)** explores level by level using a queue. It's the natural choice for shortest paths on unweighted graphs.

- **DFS (Depth-First Search)** dives as deep as possible before backtracking, typically via recursion or an explicit stack. It's a natural fit for connectivity checks, cycle detection, and backtracking-style problems.

```kotlin
// BFS
fun bfs(start: Int, adj: List<List<Int>>): List<Int> {
    val visited = BooleanArray(adj.size)
    val order = mutableListOf<Int>()
    val queue: ArrayDeque<Int> = ArrayDeque()

    queue.add(start)
    visited[start] = true

    while (queue.isNotEmpty()) {
        val node = queue.removeFirst()
        order.add(node)

        for (neighbor in adj[node]) {
            if (!visited[neighbor]) {
                visited[neighbor] = true
                queue.add(neighbor)
            }
        }
    }
    return order
}

// DFS (recursive)
fun dfs(node: Int, adj: List<List<Int>>, visited: BooleanArray, order: MutableList<Int>) {
    visited[node] = true
    order.add(node)

    for (neighbor in adj[node]) {
        if (!visited[neighbor]) {
            dfs(neighbor, adj, visited, order)
        }
    }
}
```

**Practice:**
- [LC 200: Number of Islands](https://leetcode.com/problems/number-of-islands/) *(Medium)*
- [LC 994: Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) *(Medium)*
- [LC 103: Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/) *(Medium)*
- [LC 130: Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) *(Medium)*
- [LC 1306: Jump Game III](https://leetcode.com/problems/jump-game-iii/) *(Medium)*

---

## 4. Topological Sort

> **Tier 1: Core Fundamentals**

Only valid on a DAG. Produces a linear ordering of vertices such that for every directed edge `u -> v`, `u` comes before `v`. Two common approaches:

- **DFS-based**: run DFS, and push each node onto a stack once all of its neighbors are fully processed (post-order). Popping the stack gives the topological order.

- **Kahn's Algorithm (BFS-based)**: repeatedly remove vertices with in-degree 0, decrementing the in-degree of their neighbors as you go.

```kotlin
// DFS-based topological sort
fun topoSortDfs(n: Int, adj: List<List<Int>>): List<Int> {
    val visited = BooleanArray(n)
    val stack = ArrayDeque<Int>()

    fun visit(node: Int) {
        visited[node] = true
        for (neighbor in adj[node]) {
            if (!visited[neighbor]) visit(neighbor)
        }
        stack.addLast(node)
    }

    for (i in 0 until n) {
        if (!visited[i]) visit(i)
    }
    return stack.reversed()
}

// Kahn's Algorithm
fun topoSortKahn(n: Int, adj: List<List<Int>>): List<Int> {
    val inDegree = IntArray(n)
    for (node in 0 until n) {
        for (neighbor in adj[node]) inDegree[neighbor]++
    }

    val queue = ArrayDeque<Int>()
    for (i in 0 until n) {
        if (inDegree[i] == 0) queue.add(i)
    }

    val order = mutableListOf<Int>()
    while (queue.isNotEmpty()) {
        val node = queue.removeFirst()
        order.add(node)
        for (neighbor in adj[node]) {
            inDegree[neighbor]--
            if (inDegree[neighbor] == 0) queue.add(neighbor)
        }
    }
    // If order.size < n, the graph has a cycle
    return order
}
```

**Practice:**
- [LC 210: Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) *(Medium)*
- [LC 802: Find Eventual Safe States](https://leetcode.com/problems/find-eventual-safe-states/) *(Medium)*
- [LC 329: Longest Increasing Path in a Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/) *(Hard)*
- [LC 1136: Parallel Courses](https://leetcode.com/problems/parallel-courses/) *(Medium - Premium)*
- [LC 1203: Sort Items by Groups Respecting Dependencies](https://leetcode.com/problems/sort-items-by-groups-respecting-dependencies/) *(Hard)*

---

## 5. Shortest Path Algorithms

> **Tier 2: Weighted Graphs & Shortest Paths**

- **Dijkstra's Algorithm** — greedy, works on graphs with non-negative weights. Uses a min-heap keyed on current best distance.

- **Bellman-Ford** — handles negative edge weights and detects negative cycles, at the cost of `O(V·E)` time. It also has a useful round-based property: after exactly k rounds of relaxation, the shortest distance to any vertex is guaranteed correct for paths using at most k edges.

- **Floyd-Warshall** — computes all-pairs shortest paths via dynamic programming, `O(V³)`, useful when the graph is small and dense.

```kotlin
// Dijkstra's Algorithm
fun dijkstra(n: Int, adj: List<List<Pair<Int, Int>>>, start: Int): IntArray {
    val dist = IntArray(n) { Int.MAX_VALUE }
    dist[start] = 0
    val pq = java.util.PriorityQueue<Pair<Int, Int>>(compareBy { it.first }) // (distance, node)
    pq.add(0 to start)

    while (pq.isNotEmpty()) {
        val (d, u) = pq.poll()
        if (d > dist[u]) continue

        for ((v, weight) in adj[u]) {
            val newDist = dist[u] + weight
            if (newDist < dist[v]) {
                dist[v] = newDist
                pq.add(newDist to v)
            }
        }
    }
    return dist
}

// Bellman-Ford
fun bellmanFord(n: Int, edges: List<IntArray>, start: Int): IntArray {
    val dist = IntArray(n) { Int.MAX_VALUE }
    dist[start] = 0

    repeat(n - 1) {
        for (edge in edges) {
            val (u, v, weight) = edge
            if (dist[u] != Int.MAX_VALUE && dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight
            }
        }
    }
    return dist
}

// Floyd-Warshall
fun floydWarshall(n: Int, graph: Array<IntArray>): Array<IntArray> {
    val dist = Array(n) { i -> graph[i].copyOf() }

    for (k in 0 until n) {
        for (i in 0 until n) {
            for (j in 0 until n) {
                if (dist[i][k] != Int.MAX_VALUE && dist[k][j] != Int.MAX_VALUE &&
                    dist[i][k] + dist[k][j] < dist[i][j]
                ) {
                    dist[i][j] = dist[i][k] + dist[k][j]
                }
            }
        }
    }
    return dist
}
```

**Practice:**
- [LC 743: Network Delay Time](https://leetcode.com/problems/network-delay-time/) *(Medium)*
- [LC 787: Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) *(Medium)*
- [LC 1631: Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) *(Medium)*
- [LC 1514: Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/) *(Medium)*
- [LC 1129: Shortest Path with Alternating Colors](https://leetcode.com/problems/shortest-path-with-alternating-colors/) *(Medium)*

---

## 6. Union-Find (Disjoint Set)

> **Tier 1: Core Fundamentals**

A structure for tracking which elements belong to the same group under merges, supporting near-`O(1)` amortized `find` and `union` operations via **path compression** and **union by rank**. It's the backbone of Kruskal's MST algorithm and any problem about merging connected components.

```kotlin
class UnionFind(n: Int) {
    private val parent = IntArray(n) { it }
    private val rank = IntArray(n)

    fun find(x: Int): Int {
        if (parent[x] != x) {
            parent[x] = find(parent[x]) // path compression
        }
        return parent[x]
    }

    fun union(x: Int, y: Int) {
        val rootX = find(x)
        val rootY = find(y)
        if (rootX == rootY) return

        when {
            rank[rootX] > rank[rootY] -> parent[rootY] = rootX
            rank[rootX] < rank[rootY] -> parent[rootX] = rootY
            else -> {
                parent[rootY] = rootX
                rank[rootX]++
            }
        }
    }
}
```

**Practice:**
- [LC 684: Redundant Connection](https://leetcode.com/problems/redundant-connection/) *(Medium)*
- [LC 990: Satisfiability of Equality Equations](https://leetcode.com/problems/satisfiability-of-equality-equations/) *(Medium)*
- [LC 721: Accounts Merge](https://leetcode.com/problems/accounts-merge/) *(Medium)*
- [LC 1319: Number of Operations to Make Network Connected](https://leetcode.com/problems/number-of-operations-to-make-network-connected/) *(Medium)*
- [LC 128: Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) *(Medium)*

---

## 7. Minimum Spanning Tree

> **Tier 2: Weighted Graphs & Shortest Paths**

A minimum spanning tree connects all vertices with the minimum total edge weight and no cycles.

- **Prim's Algorithm** — grows the tree one vertex at a time, always adding the cheapest edge that connects a new vertex. Similar structure to Dijkstra's, using a min-heap.

- **Kruskal's Algorithm** — sorts all edges by weight and adds them greedily, skipping any edge that would create a cycle (checked via Union-Find).

```kotlin
// Prim's Algorithm
fun primMst(n: Int, adj: List<List<Pair<Int, Int>>>): Int {
    val inMst = BooleanArray(n)
    val pq = java.util.PriorityQueue<Pair<Int, Int>>(compareBy { it.first }) // (cost, node)
    pq.add(0 to 0)
    var totalCost = 0

    while (pq.isNotEmpty()) {
        val (cost, u) = pq.poll()
        if (inMst[u]) continue

        inMst[u] = true
        totalCost += cost

        for ((v, weight) in adj[u]) {
            if (!inMst[v]) pq.add(weight to v)
        }
    }
    return totalCost
}

// Kruskal's Algorithm
data class WeightedEdge(val u: Int, val v: Int, val weight: Int)

fun kruskalMst(n: Int, edges: List<WeightedEdge>): Int {
    val sorted = edges.sortedBy { it.weight }
    val uf = UnionFind(n)
    var totalCost = 0

    for (edge in sorted) {
        if (uf.find(edge.u) != uf.find(edge.v)) {
            uf.union(edge.u, edge.v)
            totalCost += edge.weight
        }
    }
    return totalCost
}
```

**Practice:**
- [LC 1584: Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) *(Medium)*
- [LC 1135: Connecting Cities With Minimum Cost](https://leetcode.com/problems/connecting-cities-with-minimum-cost/) *(Medium - Premium)*
- [LC 1631: Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) *(Medium)* *(solvable via MST-style approach)*
- [LC 1101: The Earliest Moment When Everyone Become Friends](https://leetcode.com/problems/the-earliest-moment-when-everyone-become-friends/) *(Medium - Premium)*
- [LC 1579: Remove Max Number of Edges to Keep Graph Fully Traversable](https://leetcode.com/problems/remove-max-number-of-edges-to-keep-graph-fully-traversable/) *(Hard)*

---

## 8. Cycle Detection

> **Tier 1: Core Fundamentals**

- **Undirected graph** — during DFS, if you reach an already-visited neighbor that isn't your immediate parent, there's a cycle.

- **Directed graph** — track a recursion stack alongside the visited set. If DFS reaches a node that's currently on the recursion stack, there's a cycle (as opposed to just being visited earlier via a different branch).

```kotlin
// Cycle detection — undirected graph
fun hasCycleUndirected(n: Int, adj: List<List<Int>>): Boolean {
    val visited = BooleanArray(n)

    fun dfs(node: Int, parent: Int): Boolean {
        visited[node] = true
        for (neighbor in adj[node]) {
            if (!visited[neighbor]) {
                if (dfs(neighbor, node)) return true
            } else if (neighbor != parent) {
                return true
            }
        }
        return false
    }

    for (i in 0 until n) {
        if (!visited[i] && dfs(i, -1)) return true
    }
    return false
}

// Cycle detection — directed graph
fun hasCycleDirected(n: Int, adj: List<List<Int>>): Boolean {
    val visited = BooleanArray(n)
    val onStack = BooleanArray(n)

    fun dfs(node: Int): Boolean {
        visited[node] = true
        onStack[node] = true

        for (neighbor in adj[node]) {
            if (!visited[neighbor]) {
                if (dfs(neighbor)) return true
            } else if (onStack[neighbor]) {
                return true
            }
        }
        onStack[node] = false
        return false
    }

    for (i in 0 until n) {
        if (!visited[i] && dfs(i)) return true
    }
    return false
}
```

**Practice:**
- [LC 785: Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/) *(Medium)*
- [LC 207: Course Schedule](https://leetcode.com/problems/course-schedule/) *(Medium)* *(directed cycle detection)*
- [LC 261: Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/) *(Medium - Premium)*
- [LC 802: Find Eventual Safe States](https://leetcode.com/problems/find-eventual-safe-states/) *(Medium)* *(directed cycle detection)*
- [LC 1462: Course Schedule IV](https://leetcode.com/problems/course-schedule-iv/) *(Medium)*

---

## 9. Bipartite Graphs

> **Tier 1: Core Fundamentals**

A graph is bipartite if its vertices can be split into two sets such that every edge connects a vertex from one set to the other — equivalently, if it can be 2-colored with no two adjacent vertices sharing a color. BFS coloring is a clean way to check this: color the start node, then alternate colors as you traverse; if a conflict ever appears, the graph isn't bipartite.

```kotlin
fun isBipartite(n: Int, adj: List<List<Int>>): Boolean {
    val color = IntArray(n) { -1 }

    for (start in 0 until n) {
        if (color[start] != -1) continue

        color[start] = 0
        val queue = ArrayDeque<Int>()
        queue.add(start)

        while (queue.isNotEmpty()) {
            val node = queue.removeFirst()
            for (neighbor in adj[node]) {
                if (color[neighbor] == -1) {
                    color[neighbor] = 1 - color[node]
                    queue.add(neighbor)
                } else if (color[neighbor] == color[node]) {
                    return false
                }
            }
        }
    }
    return true
}
```

**Practice:**
- [LC 785: Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/) *(Medium)*
- [LC 886: Possible Bipartition](https://leetcode.com/problems/possible-bipartition/) *(Medium)*
- [LC 1034: Coloring A Border](https://leetcode.com/problems/coloring-a-border/) *(Medium)*
- [LC 994: Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) *(Medium)* *(BFS-based)*
- [LC 1091: Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) *(Medium)*

---

## 10. Bridges: Tarjan's Algorithm

> **Tier 3: Advanced Niche Topics**

A **bridge** is an edge whose removal disconnects the graph — a single point of failure in a network. Tarjan's approach finds all of them in one DFS by tracking two numbers per vertex:

- `disc[u]` — the time at which DFS first discovered `u`.
- `low[u]` — the earliest discovery time reachable from `u`'s subtree using at most one back edge.

If a child `v` of `u` has `low[v] > disc[u]`, nothing in `v`'s subtree can reach `u` or above without going through the edge `u-v`, so that edge is a bridge. Runs in `O(V + E)`.

```kotlin
fun criticalConnections(n: Int, connections: List<List<Int>>): List<List<Int>> {
    val adj = List(n) { mutableListOf<Int>() }
    for ((u, v) in connections) {
        adj[u].add(v)
        adj[v].add(u)
    }

    val disc = IntArray(n) { -1 }
    val low = IntArray(n)
    val bridges = mutableListOf<List<Int>>()
    var timer = 0

    fun dfs(u: Int, parent: Int) {
        disc[u] = timer
        low[u] = timer
        timer++

        for (v in adj[u]) {
            if (v == parent) continue
            if (disc[v] == -1) {
                dfs(v, u)
                low[u] = minOf(low[u], low[v])
                if (low[v] > disc[u]) bridges.add(listOf(u, v))
            } else {
                low[u] = minOf(low[u], disc[v]) // back edge
            }
        }
    }

    for (i in 0 until n) {
        if (disc[i] == -1) dfs(i, -1)
    }
    return bridges
}
```

> Skipping the parent by vertex is fine when there are no parallel edges. If the graph can contain duplicate edges, skip by edge id instead.

**Practice:**
- [LC 1192: Critical Connections in a Network](https://leetcode.com/problems/critical-connections-in-a-network/) *(Hard)*
- [LC 1568: Minimum Number of Days to Disconnect Island](https://leetcode.com/problems/minimum-number-of-days-to-disconnect-island/) *(Hard - articulation points)*

---

## 11. Eulerian Paths: Hierholzer's Algorithm

> **Tier 3: Advanced Niche Topics**

An **Eulerian path** uses every *edge* exactly once (compare with a Hamiltonian path, which visits every *vertex* once). It exists in a directed graph when at most one vertex has `out - in = 1` (the start), at most one has `in - out = 1` (the end), and all others are balanced.

Hierholzer's algorithm builds the path with a DFS that consumes edges as it goes: keep following unused edges until stuck, then add the current vertex to the route *on the way back out*. The route comes out reversed, so insert at the front. Runs in `O(E)` (plus sorting cost if edges must be taken in order).

```kotlin
// Reconstruct a directed Eulerian path, taking the lexically smallest destination first
fun findItinerary(tickets: List<List<String>>): List<String> {
    val graph = HashMap<String, java.util.PriorityQueue<String>>()
    for ((from, to) in tickets) {
        graph.getOrPut(from) { java.util.PriorityQueue() }.add(to)
    }

    val route = ArrayDeque<String>()

    fun visit(airport: String) {
        val destinations = graph[airport]
        while (destinations != null && destinations.isNotEmpty()) {
            visit(destinations.poll()) // consume the edge, then recurse
        }
        route.addFirst(airport) // added only when stuck: post-order
    }

    visit("JFK")
    return route.toList()
}
```

**Practice:**
- [LC 332: Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/) *(Hard)*
- [LC 753: Cracking the Safe](https://leetcode.com/problems/cracking-the-safe/) *(Hard)*
- [LC 2097: Valid Arrangement of Pairs](https://leetcode.com/problems/valid-arrangement-of-pairs/) *(Hard)*

---

## 12. Bitmask DP on Graphs

> **Tier 3: Advanced Niche Topics**

When a problem asks you to visit *every* node (or choose among a small set of items) and `n` is tiny — typically `n <= 20` — encode "which nodes have been visited" as a bitmask. A state becomes `(current node, visited mask)`, giving at most `n * 2^n` states.

For unweighted edges, BFS over those states finds the shortest walk that visits everything. For weighted edges, the same state space drives the classic Travelling Salesman DP.

```kotlin
// Shortest walk that visits every node; it may start anywhere and revisit nodes
fun shortestPathLength(graph: Array<IntArray>): Int {
    val n = graph.size
    if (n == 1) return 0

    val full = (1 shl n) - 1
    val visited = Array(n) { BooleanArray(1 shl n) }
    val queue = ArrayDeque<IntArray>() // each entry: (node, mask)

    for (i in 0 until n) {
        queue.add(intArrayOf(i, 1 shl i))
        visited[i][1 shl i] = true
    }

    var steps = 0
    while (queue.isNotEmpty()) {
        steps++
        repeat(queue.size) {
            val (node, mask) = queue.removeFirst()
            for (next in graph[node]) {
                val newMask = mask or (1 shl next)
                if (newMask == full) return steps
                if (!visited[next][newMask]) {
                    visited[next][newMask] = true
                    queue.add(intArrayOf(next, newMask))
                }
            }
        }
    }
    return -1
}
```

**Practice:**
- [LC 847: Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/) *(Hard)*
- [LC 943: Find the Shortest Superstring](https://leetcode.com/problems/find-the-shortest-superstring/) *(Hard)*
- [LC 1494: Parallel Courses II](https://leetcode.com/problems/parallel-courses-ii/) *(Hard)*

---

*A compact reference covering representation, traversal, ordering, shortest paths, connectivity, spanning trees, coloring, and a few advanced topics (bridges, Eulerian paths, bitmask DP) — the graph toolkit that covers the large majority of graph-tagged interview problems.*
