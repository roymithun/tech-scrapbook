# 2D DP Pattern Classification

A reference table for classifying 2D dynamic programming problems by what the two dimensions represent, along with the recurrence shape, loop order, and example LeetCode problems for each pattern.

| Pattern | Dimensions mean | Recurrence shape | Loop order | LeetCode problems |
|---|---|---|---|---|
| **Interval DP** | `left`, `right` = two ends of a range in one sequence | Split range at `k`, combine `solve(left,k)` + `solve(k,right)` | By increasing range length | 1. 312 - Burst Balloons<br>2. 1130 - Minimum Cost Tree From Leaf Values<br>3. 1039 - Minimum Score Triangulation of Polygon<br>4. 516 - Longest Palindromic Subsequence<br>5. 1547 - Minimum Cost to Cut a Stick<br>6. 375 - Guess Number Higher or Lower II<br>7. 664 - Strange Printer |
| **Two-sequence / alignment DP** | `i`, `j` = prefix positions in two different sequences | Step back one position in one or both sequences, compare current chars/elements | Row by row or column by column | 1. 1143 - Longest Common Subsequence<br>2. 72 - Edit Distance<br>3. 10 - Regular Expression Matching<br>4. 44 - Wildcard Matching<br>5. 97 - Interleaving String<br>6. 115 - Distinct Subsequences<br>7. 583 - Delete Operation for Two Strings<br>8. 712 - Minimum ASCII Delete Sum for Two Strings |
| **Knapsack-family DP** | `i` = item index, `capacity` = remaining budget | Include item `i` or skip it | By item, then by capacity | 1. 416 - Partition Equal Subset Sum<br>2. 494 - Target Sum<br>3. 322 - Coin Change<br>4. 518 - Coin Change II<br>5. 474 - Ones and Zeroes<br>6. 1049 - Last Stone Weight II<br>7. 279 - Perfect Squares |
| **State-machine DP** | `i` = position, `state` = current mode (small fixed set) | Transition between states at each position | By position, updating all states | 1. 121 - Best Time to Buy/Sell Stock<br>2. 122 - Best Time to Buy/Sell Stock II<br>3. 123 - Best Time to Buy/Sell Stock III<br>4. 188 - Best Time to Buy/Sell Stock IV<br>5. 309 - Best Time to Buy/Sell Stock w/ Cooldown<br>6. 714 - Best Time to Buy/Sell Stock w/ Transaction Fee<br>7. 198 - House Robber<br>8. 213 - House Robber II<br>9. 276 - Paint Fence |
| **Grid/path DP** | `row`, `col` = literal grid cell | Combine from cell(s) above/left (or all 4 neighbors) | Row by row or diagonal by diagonal | 1. 62 - Unique Paths<br>2. 63 - Unique Paths II<br>3. 64 - Minimum Path Sum<br>4. 174 - Dungeon Game<br>5. 120 - Triangle<br>6. 931 - Minimum Falling Path Sum<br>7. 221 - Maximal Square |

## Fast diagnostic

When you hit a new problem, ask what the two indices *mean*:

- Two ends of one range → **Interval DP**
- Positions in two different things → **Two-sequence / alignment DP**
- Position + budget → **Knapsack-family DP**
- Position + mode → **State-machine DP**
- Literal grid cell → **Grid/path DP**

That answer usually tells you the recurrence and loop order before writing any code.

---

*Note: this list was compiled from memory rather than a fresh lookup — worth spot-checking problem numbers against LeetCode directly before relying on it for practice, since a small number of older problems have had numbering changes over time.*
