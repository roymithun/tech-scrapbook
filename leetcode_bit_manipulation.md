# LeetCode: Bit Manipulation Problems

A curated list of LeetCode problems where bitwise techniques provide the practical, efficient solution — grouped by core technique.

## XOR Tricks (Cancellation, Pairing)

| # | Problem | Core Idea |
|---|---------|-----------|
| 136 | Single Number | XOR cancels out duplicate pairs |
| 137 | Single Number II | Bit-counting mod 3, or 2-bit state machine |
| 260 | Single Number III | XOR all, split by lowest differing bit |
| 268 | Missing Number | XOR indices with values |
| 389 | Find the Difference | XOR two strings together |
| 421 | Maximum XOR of Two Numbers in an Array | Bit-trie or greedy prefix building |
| 1720 | Decode XORed Array | Reconstruct via running XOR |
| 1734 | Decode XORed Permutation | XOR of all + XOR of odd-indexed elements |

## Counting Set Bits

| # | Problem | Core Idea |
|---|---------|-----------|
| 191 | Number of 1 Bits | `n & (n-1)` clears lowest set bit |
| 338 | Counting Bits | DP: `bits[i] = bits[i >> 1] + (i & 1)` |
| 461 | Hamming Distance | XOR then count set bits |
| 476 | Number Complement | Flip bits within value's bit-length |
| 477 | Total Hamming Distance | Count 0/1 split per bit position across array |
| 1356 | Sort Integers by The Number of 1 Bits | Popcount as sort key |

## Arithmetic Without +/- (Bitwise Simulation)

| # | Problem | Core Idea |
|---|---------|-----------|
| 371 | Sum of Two Integers | XOR = sum w/o carry, AND+shift = carry, loop |
| 29 | Divide Two Integers | Repeated doubling via left shift |
| 67 | Add Binary | Bitwise addition with carry |

## Subset / Bitmask Enumeration

| # | Problem | Core Idea |
|---|---------|-----------|
| 78 | Subsets | Iterate `0` to `2^n - 1`; each bit = include/exclude |
| 90 | Subsets II | Same, with duplicate-skipping logic |
| 320 | Generalized Abbreviation | Bitmask over which letters to abbreviate |
| 1601 | Max Number of Achievable Transfer Requests | Bitmask over requests, check balance |
| 1994 | The Number of Good Subsets | Bitmask over prime factors (as state) |
| 1125 | Smallest Sufficient Team | Bitmask set-cover DP |
| 1178 | Number of Valid Words for Each Puzzle | Represent words as 26-bit letter masks |
| 2044 | Count Number of Maximum Bitwise-OR Subsets | Brute force subset OR via bitmask |

## Bitmask Dynamic Programming (State = Subset)

| # | Problem | Core Idea |
|---|---------|-----------|
| 698 | Partition to K Equal Sum Subsets | Bitmask of used elements |
| 847 | Shortest Path Visiting All Nodes | BFS state = (node, visited-mask) |
| 864 | Shortest Path to Get All Keys | BFS state = (position, key-mask) |
| 1349 | Maximum Students Taking Exam | Row bitmask DP with seat-compatibility checks |
| 1655 | Distribute Repeating Integers | Bitmask DP over customer subsets |
| 1723 | Find Minimum Time to Finish All Jobs | Bitmask DP / subset-sum partition |
| 1799 | Maximize Score After N Operations | Bitmask DP over used pairs |
| 1986 | Min Number of Work Sessions to Finish Tasks | Bitmask DP over task subsets |
| 2151 | Maximum Good People Based on Statements | Small n, brute-force bitmask validation |
| 2172 | Maximum AND Sum of Array | Bitmask DP with base-3 slot encoding |

## Power-of-Two / Single-Bit Checks

| # | Problem | Core Idea |
|---|---------|-----------|
| 231 | Power of Two | `n > 0 && (n & (n-1)) == 0` |
| 342 | Power of Four | Single bit set, and at an even position |
| 868 | Binary Gap | Track distance between consecutive set bits |
| 1009 | Complement of Base 10 Integer | Flip bits within bit-length |

## Range / Prefix Bit Tricks

| # | Problem | Core Idea |
|---|---------|-----------|
| 201 | Bitwise AND of Numbers Range | Right-shift both bounds to common prefix |
| 1310 | XOR Queries of a Subarray | Prefix XOR array |
| 1442 | Count Triplets That Can Form Two Arrays of Equal XOR | Prefix XOR equality |
| 1521 | Find a Value of the Mysterious Function Closest to Target | AND is monotonic non-increasing; dedupe with running set |
| 1803 | Count Pairs With XOR in a Range | Bit-trie with range counting |

## Trie + Bitwise (XOR Maximization Family)

| # | Problem | Core Idea |
|---|---------|-----------|
| 421 | Maximum XOR of Two Numbers in an Array | Build binary trie, greedily maximize per bit |
| 1707 | Maximum XOR With an Element From Array | Offline trie + queries sorted by limit |

---

### Suggested Learning Path

Start roughly in this order to build intuition progressively:

**136 → 191 → 338 → 78 → 231 → 260 → 371 → 421 → 847**

This moves from basic XOR/AND properties → bit counting → subset enumeration → bitmask DP, covering most patterns that show up in interviews.
