# Graph & BFS Practice Problems — Categorized

| Category | Problem | Difficulty | Approach | Interview Frequency |
|---|---|---|---|---|
| **Grid connectivity** | Number of Islands | Medium | BFS or DFS | 🔥🔥🔥 Very common |
| | Max Area of Island | Medium | BFS or DFS | 🔥🔥 Common |
| | Pacific Atlantic Water Flow | Medium | BFS/DFS from borders inward | 🔥 Occasional |
| | Surrounded Regions | Medium | BFS/DFS from borders inward | 🔥 Occasional |
| | Number of Provinces | Medium | BFS/DFS or Union-Find | 🔥 Occasional |
| | Flood Fill | Easy | BFS or DFS | 🔥🔥 Common (often as warm-up) |
| | Island Perimeter | Easy | Simple traversal | Rare |
| | Number of Closed Islands | Medium | BFS/DFS from borders | Rare |
| | Number of Distinct Islands | Medium | BFS/DFS + shape encoding | Rare |
| **Grid shortest distance** | Walls and Gates | Medium | Multi-source BFS | 🔥🔥 Common |
| | Rotting Oranges | Medium | Multi-source BFS | 🔥🔥🔥 Very common |
| | 01 Matrix | Medium | Multi-source BFS | 🔥🔥 Common |
| | Shortest Path in Binary Matrix | Medium | Single-source BFS | 🔥 Occasional |
| | As Far from Land as Possible | Medium | Multi-source BFS | Rare |
| | Shortest Path to Get Food | Medium | Single-source BFS | Rare |
| **Graph structure/connectivity** | Clone Graph | Medium | BFS or DFS + visited map | 🔥🔥🔥 Very common |
| | Graph Valid Tree | Medium | BFS/DFS or Union-Find | 🔥🔥 Common |
| | Number of Connected Components in an Undirected Graph | Medium | BFS/DFS or Union-Find | 🔥🔥 Common |
| | Redundant Connection | Medium | Union-Find | 🔥 Occasional |
| | Redundant Connection II | Hard | Union-Find + edge cases | Rare |
| | Accounts Merge | Medium | Union-Find or DFS/BFS | 🔥🔥 Common (esp. at Facebook/Meta) |
| | Is Graph Bipartite? | Medium | BFS/DFS with 2-coloring | 🔥 Occasional |
| | Satisfiability of Equality Equations | Medium | Union-Find | Rare |
| **Topological ordering** | Course Schedule | Medium | Kahn's (BFS) or DFS | 🔥🔥🔥 Very common |
| | Course Schedule II | Medium | Kahn's (BFS) or DFS topo sort | 🔥🔥 Common |
| | Alien Dictionary | Hard (Premium) | Topological sort | 🔥🔥 Common (esp. at Google/Meta) |
| | Minimum Height Trees | Medium | BFS layer-trimming | 🔥 Occasional |
| | Sequence Reconstruction | Medium | Topological sort + uniqueness | Rare |
| **Implicit/state-space graph** | Word Ladder | Hard | Single-source BFS | 🔥🔥 Common |
| | Word Ladder II | Hard | BFS + DFS/backtrack | Rare (very hard for interview time limits) |
| | Open the Lock | Medium | Single-source BFS | 🔥 Occasional |
| | Minimum Genetic Mutation | Medium | Single-source BFS | Rare |
| | Sliding Puzzle | Hard | Single-source BFS | Rare |
| | Jump Game III | Medium | BFS/DFS | Rare |
| | Snakes and Ladders | Medium | Single-source BFS | Rare |

## Highest ROI — Priority Order

If short on time, tackle these first (covers every category):

1. Number of Islands
2. Rotting Oranges
3. Clone Graph
4. Course Schedule
5. Walls and Gates
6. 01 Matrix
7. Course Schedule II
8. Accounts Merge
9. Alien Dictionary (if targeting Google/Meta-tier companies)
10. Word Ladder

## Category Intuition

- **Grid connectivity** — counting or measuring "blobs" of connected cells, or checking what's reachable from a boundary. Distance doesn't matter, only reachability.
- **Grid shortest distance** — need the *minimum* number of steps/time, so BFS is mandatory (not DFS), usually seeded from multiple starting cells at once.
- **Graph structure/connectivity** — reasoning about the graph's shape itself: connected or not, cyclic or not, how many separate pieces. Union-Find often works as well as or better than BFS/DFS here.
- **Topological ordering** — directed dependency relationships; need either a valid sequence or to detect that one's impossible (a cycle).
- **Implicit/state-space graph** — hardest to spot, since the "graph" isn't handed to you directly; nodes/edges are generated on the fly from some other structure (words, board states, etc.).
