# LeetCode: Bit Manipulation Problems

A curated list of LeetCode problems where bitwise techniques provide the practical, efficient solution — grouped by core technique.

## XOR Tricks (Cancellation, Pairing)

| # | Problem | Core Idea |
|---|---------|-----------|
| [136](https://leetcode.com/problems/single-number/) | Single Number | XOR cancels out duplicate pairs |
| [137](https://leetcode.com/problems/single-number-ii/) | Single Number II | Bit-counting mod 3, or 2-bit state machine |
| [260](https://leetcode.com/problems/single-number-iii/) | Single Number III | XOR all, split by lowest differing bit |
| [268](https://leetcode.com/problems/missing-number/) | Missing Number | XOR indices with values |
| [389](https://leetcode.com/problems/find-the-difference/) | Find the Difference | XOR two strings together |
| [421](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) | Maximum XOR of Two Numbers in an Array | Bit-trie or greedy prefix building |
| [1720](https://leetcode.com/problems/decode-xored-array/) | Decode XORed Array | Reconstruct via running XOR |
| [1734](https://leetcode.com/problems/decode-xored-permutation/) | Decode XORed Permutation | XOR of all + XOR of odd-indexed elements |

## Counting Set Bits

| # | Problem | Core Idea |
|---|---------|-----------|
| [191](https://leetcode.com/problems/number-of-1-bits/) | Number of 1 Bits | `n & (n-1)` clears lowest set bit |
| [338](https://leetcode.com/problems/counting-bits/) | Counting Bits | DP: `bits[i] = bits[i >> 1] + (i & 1)` |
| [461](https://leetcode.com/problems/hamming-distance/) | Hamming Distance | XOR then count set bits |
| [476](https://leetcode.com/problems/number-complement/) | Number Complement | Flip bits within value's bit-length |
| [477](https://leetcode.com/problems/total-hamming-distance/) | Total Hamming Distance | Count 0/1 split per bit position across array |
| [1356](https://leetcode.com/problems/sort-integers-by-the-number-of-1-bits/) | Sort Integers by The Number of 1 Bits | Popcount as sort key |

## Arithmetic Without +/- (Bitwise Simulation)

| # | Problem | Core Idea |
|---|---------|-----------|
| [371](https://leetcode.com/problems/sum-of-two-integers/) | Sum of Two Integers | XOR = sum w/o carry, AND+shift = carry, loop |
| [29](https://leetcode.com/problems/divide-two-integers/) | Divide Two Integers | Repeated doubling via left shift |
| [67](https://leetcode.com/problems/add-binary/) | Add Binary | Bitwise addition with carry |

## Subset / Bitmask Enumeration

| # | Problem | Core Idea |
|---|---------|-----------|
| [78](https://leetcode.com/problems/subsets/) | Subsets | Iterate `0` to `2^n - 1`; each bit = include/exclude |
| [90](https://leetcode.com/problems/subsets-ii/) | Subsets II | Same, with duplicate-skipping logic |
| [320](https://leetcode.com/problems/generalized-abbreviation/) | Generalized Abbreviation | Bitmask over which letters to abbreviate |
| [1601](https://leetcode.com/problems/maximum-number-of-achievable-transfer-requests/) | Max Number of Achievable Transfer Requests | Bitmask over requests, check balance |
| [1994](https://leetcode.com/problems/the-number-of-good-subsets/) | The Number of Good Subsets | Bitmask over prime factors (as state) |
| [1125](https://leetcode.com/problems/smallest-sufficient-team/) | Smallest Sufficient Team | Bitmask set-cover DP |
| [1178](https://leetcode.com/problems/number-of-valid-words-for-each-puzzle/) | Number of Valid Words for Each Puzzle | Represent words as 26-bit letter masks |
| [2044](https://leetcode.com/problems/count-number-of-maximum-bitwise-or-subsets/) | Count Number of Maximum Bitwise-OR Subsets | Brute force subset OR via bitmask |

## Bitmask Dynamic Programming (State = Subset)

| # | Problem | Core Idea |
|---|---------|-----------|
| [698](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/) | Partition to K Equal Sum Subsets | Bitmask of used elements |
| [847](https://leetcode.com/problems/shortest-path-visiting-all-nodes/) | Shortest Path Visiting All Nodes | BFS state = (node, visited-mask) |
| [864](https://leetcode.com/problems/shortest-path-to-get-all-keys/) | Shortest Path to Get All Keys | BFS state = (position, key-mask) |
| [1349](https://leetcode.com/problems/maximum-students-taking-exam/) | Maximum Students Taking Exam | Row bitmask DP with seat-compatibility checks |
| [1655](https://leetcode.com/problems/distribute-repeating-integers/) | Distribute Repeating Integers | Bitmask DP over customer subsets |
| [1723](https://leetcode.com/problems/find-minimum-time-to-finish-all-jobs/) | Find Minimum Time to Finish All Jobs | Bitmask DP / subset-sum partition |
| [1799](https://leetcode.com/problems/maximize-score-after-n-operations/) | Maximize Score After N Operations | Bitmask DP over used pairs |
| [1986](https://leetcode.com/problems/minimum-number-of-work-sessions-to-finish-the-tasks/) | Min Number of Work Sessions to Finish Tasks | Bitmask DP over task subsets |
| [2151](https://leetcode.com/problems/maximum-good-people-based-on-statements/) | Maximum Good People Based on Statements | Small n, brute-force bitmask validation |
| [2172](https://leetcode.com/problems/maximum-and-sum-of-array/) | Maximum AND Sum of Array | Bitmask DP with base-3 slot encoding |

## Power-of-Two / Single-Bit Checks

| # | Problem | Core Idea |
|---|---------|-----------|
| [231](https://leetcode.com/problems/power-of-two/) | Power of Two | `n > 0 && (n & (n-1)) == 0` |
| [342](https://leetcode.com/problems/power-of-four/) | Power of Four | Single bit set, and at an even position |
| [868](https://leetcode.com/problems/binary-gap/) | Binary Gap | Track distance between consecutive set bits |
| [1009](https://leetcode.com/problems/complement-of-base-10-integer/) | Complement of Base 10 Integer | Flip bits within bit-length |

## Range / Prefix Bit Tricks

| # | Problem | Core Idea |
|---|---------|-----------|
| [201](https://leetcode.com/problems/bitwise-and-of-numbers-range/) | Bitwise AND of Numbers Range | Right-shift both bounds to common prefix |
| [1310](https://leetcode.com/problems/xor-queries-of-a-subarray/) | XOR Queries of a Subarray | Prefix XOR array |
| [1442](https://leetcode.com/problems/count-triplets-that-can-form-two-arrays-of-equal-xor/) | Count Triplets That Can Form Two Arrays of Equal XOR | Prefix XOR equality |
| [1521](https://leetcode.com/problems/find-a-value-of-the-mysterious-function-closest-to-target/) | Find a Value of the Mysterious Function Closest to Target | AND is monotonic non-increasing; dedupe with running set |
| [1803](https://leetcode.com/problems/count-pairs-with-xor-in-a-range/) | Count Pairs With XOR in a Range | Bit-trie with range counting |

## Trie + Bitwise (XOR Maximization Family)

| # | Problem | Core Idea |
|---|---------|-----------|
| [421](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) | Maximum XOR of Two Numbers in an Array | Build binary trie, greedily maximize per bit |
| [1707](https://leetcode.com/problems/maximum-xor-with-an-element-from-array/) | Maximum XOR With an Element From Array | Offline trie + queries sorted by limit |

---

### Suggested Learning Path

Start roughly in this order to build intuition progressively:

**[136](https://leetcode.com/problems/single-number/) → [191](https://leetcode.com/problems/number-of-1-bits/) → [338](https://leetcode.com/problems/counting-bits/) → [78](https://leetcode.com/problems/subsets/) → [231](https://leetcode.com/problems/power-of-two/) → [260](https://leetcode.com/problems/single-number-iii/) → [371](https://leetcode.com/problems/sum-of-two-integers/) → [421](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) → [847](https://leetcode.com/problems/shortest-path-visiting-all-nodes/)**

This moves from basic XOR/AND properties → bit counting → subset enumeration → bitmask DP, covering most patterns that show up in interviews.