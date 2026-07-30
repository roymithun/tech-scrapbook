# Sentinel Values in DP: Top-Down vs Bottom-Up

**Problem context:** [Target Sum (LeetCode 494)](https://leetcode.com/problems/target-sum/) — given an array of numbers, assign `+` or `-` to each so they sum to a target, and count the number of ways.

This note compares how the memo/DP table is initialized in the **top-down (memoized recursion)** solution vs the **bottom-up (tabulation)** solution, and why they differ.

---

## Top-down (memoized recursion): needs a "not computed yet" sentinel

```kotlin
Array(n + 1) { arrayOfNulls<Int>(width) }        // null = not computed
// or
Array(n + 1) { IntArray(width) { -1 } }          // -1 = not computed (since counts are always >= 0)
```

**Why:** recursion visits states in an unpredictable, data-dependent order — driven by which branch gets explored first. So when you land on `memo[i][idx]`, you genuinely can't tell, just by looking, whether:

- this state was **already computed and the answer happens to be `0`** (zero valid ways), or
- this state **hasn't been visited yet**

Both look identical if you use a plain `0`-defaulting array — so you need a value that can *never* be a real answer to act as the "unvisited" flag. `null` (with `Int?`) is the clean way; `-1` works too since the actual answer (a count of ways) can never be negative.

---

## Bottom-up (tabulation): plain `0`-default array is safe

```kotlin
Array(n + 1) { IntArray(width) }                 // defaults to 0, no ambiguity
```

**Why this is safe here:** you're filling the table in a fixed, deterministic order — row by row, `i = 0, 1, 2, ..., n`. By the time you read any cell in row `i`, that entire row is **already fully and correctly computed** — there's no "haven't gotten to it yet" state to worry about, because you never read a cell before you're guaranteed to have written its final value. So `0` only ever means one thing: "zero ways to reach this sum" — a real, trustworthy answer, never a placeholder.

---

## The core distinction

| | Order of computation | Can `0` mean two different things? | Needs sentinel? |
|---|---|---|---|
| **Top-down** | Unpredictable (recursion-driven) | Yes — "not visited" vs "visited, answer is 0" | Yes — `null` or `-1` |
| **Bottom-up** | Fixed, sequential (row by row) | No — `0` always means "zero ways" | No — plain `0`-default is fine |

**Rule of thumb:** whenever a *read* could happen *before* the guaranteed *write* (top-down/recursive), you need a sentinel. Whenever computation order guarantees every read happens after its write (bottom-up/iterative), a plain default works.
