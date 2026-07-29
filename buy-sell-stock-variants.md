# Best Time to Buy and Sell Stock — Variants & Techniques

## What "N-state DP" Means

At the end of each day, your money is in exactly one of a few **states**. You track the *best possible profit* for being in each state, and update all states day-by-day.

The simplest case has 2 states:
- **`hold`** = best profit if you currently own a share
- **`cash`** = best profit if you currently own no share

Each day, each state can only have gotten there from specific previous states:
- `hold` today = either you already held it yesterday, OR you just bought today (from `cash` yesterday, minus price)
- `cash` today = either you already had no share yesterday, OR you just sold today (from `hold` yesterday, plus price)

So it's just: **"what are the finite ways I could be in this situation today, and which was best?"** More states just means more constraints to track (e.g., "holding, on my 2nd transaction" is a different state from "holding, on my 1st transaction"). That's all "k-state" or "variable-state" DP means — no new idea, just more buckets.

---

## Summary Table

| # | Variant (Problem Statement) | States needed | Best technique | Time / Space |
|---|---------|---------------|-----------------|---------------|
| 1 | One transaction *(You're given an array of daily prices; you may buy once and sell once, buy must occur before sell — find the max profit)* | none (just running min) | Greedy min-tracking | O(n) / O(1) |
| 2 | Unlimited transactions *(Same array, but you may buy and sell as many times as you like — never holding more than one share at a time — find the max total profit)* | 2 (hold, cash) | Greedy sum of gains | O(n) / O(1) |
| 3 | At most 2 transactions *(Same as above, but you're limited to at most two buy-sell pairs total)* | 4 (buy1, sell1, buy2, sell2) | Fixed-variable DP | O(n) / O(1) |
| 4 | At most k transactions *(Generalization of #3 — you're given an integer k and limited to at most k buy-sell pairs)* | 2k (variable) | DP table over k | O(n·k) / O(k) |
| 5 | Cooldown after sell *(Unlimited transactions like #2, but after selling you must wait one full day before you're allowed to buy again)* | 3 (hold, sold, rest) | State-machine DP | O(n) / O(1) |
| 6 | Transaction fee *(Unlimited transactions like #2, but each completed sale incurs a fixed fee, subtracted from that trade's profit)* | 2 (hold, cash) | State-machine DP | O(n) / O(1) |

---

## Technique Definitions

### 1. Running-min tracking (a.k.a. "one-pass greedy" for problem 1)

**What it is:** You scan once, left to right, carrying forward exactly one piece of memory (the lowest price seen so far). At each step you ask "if I sold today, having bought at the cheapest point so far, what's my profit?"

**Why it's not really DP:** There's no table of subproblem answers being built from smaller ones — just one running variable getting updated. It's DP-*adjacent* only in the loose sense that you're reusing a previously computed value.

**How to recognize when this applies:** The decision at each step depends on a single simple aggregate (min-so-far, max-so-far) — not on multiple possible "states" you could be in.

### 2. Greedy (a.k.a. "sum of positive deltas," for problem 2)

**What it is:** At each step you make the locally-best choice, and — this is the key claim a greedy strategy always requires — you can prove local choices sum to the global optimum, with no need to remember alternatives.

**Why it works here specifically:** Since you can buy/sell unlimited times, capturing *every* uphill move (day i to day i+1 where price rises) is always optimal — there's no scenario where skipping a small gain now gets you a bigger gain later. That proof is what makes it "greedy" rather than "I just picked something that seemed reasonable."

**How to recognize when this applies:** Ask "is there ever a reason to NOT take a locally good move?" If no — greedy. If yes (e.g., limited transactions — taking a move now might cost you a better move later) — you need DP.

### 3, 5, 6. State-machine DP (the umbrella term for the rest)

**What it is:** You define a small fixed set of **states** — situations you could be in at the end of day `i` — and for each state you track the best profit achievable to reach that state. Every day, you recompute each state's value using only the previous day's state values.

**The mechanical recipe, every time:**
1. List the states (e.g., "holding a share", "not holding, just sold", "not holding, resting").
2. For each state, ask: "what state(s) yesterday could transition into this state today, and what's the cost/gain of that transition?"
3. Write one line of code per state, taking the max over its incoming transitions.
4. Loop over days, updating all state variables **simultaneously** (using the previous day's values, not values already updated this iteration — that's why `prevSold` gets saved before overwriting `sold` in problem 5).

**Why "2-state" vs "3-state" vs "4-state" — the number is just how many buckets you need:**
- Problem 6 (fee): 2 buckets — `hold`, `cash`. No extra constraint, so 2 is enough.
- Problem 5 (cooldown): 3 buckets — because "just sold" and "resting/eligible to buy" behave differently (you can't buy the day right after selling), so they can't share a bucket.
- Problem 3 (at most 2 transactions): 4 buckets — because "holding on my 1st purchase" and "holding on my 2nd purchase" must be tracked separately, or you'd lose track of how many transactions you've spent.

**This is the actual distinguishing test:** if two situations ever need *different* future treatment (different allowed next moves, or different transaction budget remaining), they're different states. If they'd always be treated identically going forward, merge them into one state.

### 4. DP table over (day, transactions-used) — same idea, just made variable-sized

This is state-machine DP where the number of states isn't fixed by the problem's shape but by input `k`. Instead of hand-naming 4 variables (`buy1, sell1, buy2, sell2`), you use an array indexed by "how many transactions used," because `k` isn't known until runtime.

### Quick Disambiguation Cheatsheet

| If you hear... | It means... |
|---|---|
| "Greedy" | No memory of alternatives needed; locally best = globally best, provably |
| "Running min/max" | One scalar carried forward; not proper DP |
| "State-machine DP" / "k-state DP" | Fixed small set of named situations, each with its own tracked best-value, updated via transitions each step |
| "DP table" | Same as above, but states are indexed by a variable (like `k`) rather than hand-named |

---

## Kotlin Implementations

### 1. One Transaction
```kotlin
fun maxProfit(prices: IntArray): Int {
    var minPrice = Int.MAX_VALUE
    var best = 0
    for (p in prices) {
        minPrice = minOf(minPrice, p)
        best = maxOf(best, p - minPrice)
    }
    return best
}
```

### 2. Unlimited Transactions
```kotlin
fun maxProfit(prices: IntArray): Int {
    var profit = 0
    for (i in 1 until prices.size) {
        if (prices[i] > prices[i - 1]) profit += prices[i] - prices[i - 1]
    }
    return profit
}
```

### 3. At Most 2 Transactions
```kotlin
fun maxProfit(prices: IntArray): Int {
    var buy1 = Int.MIN_VALUE; var sell1 = 0
    var buy2 = Int.MIN_VALUE; var sell2 = 0
    for (p in prices) {
        buy1 = maxOf(buy1, -p)
        sell1 = maxOf(sell1, buy1 + p)
        buy2 = maxOf(buy2, sell1 - p)
        sell2 = maxOf(sell2, buy2 + p)
    }
    return sell2
}
```

### 4. At Most k Transactions
```kotlin
fun maxProfit(k: Int, prices: IntArray): Int {
    val n = prices.size
    if (n == 0) return 0
    if (k >= n / 2) { // unlimited shortcut
        var profit = 0
        for (i in 1 until n) if (prices[i] > prices[i-1]) profit += prices[i] - prices[i-1]
        return profit
    }
    val buy = IntArray(k + 1) { Int.MIN_VALUE }
    val sell = IntArray(k + 1)
    for (p in prices) {
        for (j in 1..k) {
            buy[j] = maxOf(buy[j], sell[j - 1] - p)
            sell[j] = maxOf(sell[j], buy[j] + p)
        }
    }
    return sell[k]
}
```

### 5. With Cooldown
```kotlin
fun maxProfit(prices: IntArray): Int {
    var hold = Int.MIN_VALUE
    var sold = 0
    var rest = 0
    for (p in prices) {
        val prevSold = sold
        sold = hold + p
        hold = maxOf(hold, rest - p)
        rest = maxOf(rest, prevSold)
    }
    return maxOf(sold, rest)
}
```

### 6. With Transaction Fee
```kotlin
fun maxProfit(prices: IntArray, fee: Int): Int {
    var cash = 0
    var hold = -prices[0]
    for (i in 1 until prices.size) {
        cash = maxOf(cash, hold + prices[i] - fee)
        hold = maxOf(hold, cash - prices[i])
    }
    return cash
}
```

---

## Rule of Thumb for Choosing an Approach

- If transactions are capped at a small constant (1 or 2) → hand-roll the fixed-variable DP (O(1) space), it's cleaner than invoking the general framework.
- If `k` is a variable input → full DP table, with the `k ≥ n/2` greedy shortcut.
- If there's a structural constraint (cooldown, fee) → extend the 2-state DP with an extra state or extra term.
- This is never actually a real sliding-window (two-pointer shrink/grow) technique — that's a common misconception since problem 1 "looks" like a window problem.

# Technique Definitions — Best Time to Buy and Sell Stock

## 1. Running-min tracking (a.k.a. "one-pass greedy" for problem 1)

**What it is:** You scan once, left to right, carrying forward exactly one piece of memory (the lowest price seen so far). At each step you ask "if I sold today, having bought at the cheapest point so far, what's my profit?"

**Why it's not really DP:** There's no table of subproblem answers being built from smaller ones — just one running variable getting updated. It's DP-*adjacent* only in the loose sense that you're reusing a previously computed value.

**How to recognize when this applies:** The decision at each step depends on a single simple aggregate (min-so-far, max-so-far) — not on multiple possible "states" you could be in.

## 2. Greedy (a.k.a. "sum of positive deltas," for problem 2)

**What it is:** At each step you make the locally-best choice, and — this is the key claim a greedy strategy always requires — you can prove local choices sum to the global optimum, with no need to remember alternatives.

**Why it works here specifically:** Since you can buy/sell unlimited times, capturing *every* uphill move (day i to day i+1 where price rises) is always optimal — there's no scenario where skipping a small gain now gets you a bigger gain later. That proof is what makes it "greedy" rather than "I just picked something that seemed reasonable."

**How to recognize when this applies:** Ask "is there ever a reason to NOT take a locally good move?" If no — greedy. If yes (e.g., limited transactions — taking a move now might cost you a better move later) — you need DP.

## 3, 5, 6. State-machine DP (the umbrella term for the rest)

**What it is:** You define a small fixed set of **states** — situations you could be in at the end of day `i` — and for each state you track the best profit achievable to reach that state. Every day, you recompute each state's value using only the previous day's state values.

**The mechanical recipe, every time:**
1. List the states (e.g., "holding a share", "not holding, just sold", "not holding, resting").
2. For each state, ask: "what state(s) yesterday could transition into this state today, and what's the cost/gain of that transition?"
3. Write one line of code per state, taking the max over its incoming transitions.
4. Loop over days, updating all state variables **simultaneously** (using the previous day's values, not values already updated this iteration — that's why `prevSold` gets saved before overwriting `sold` in problem 5).

**Why "2-state" vs "3-state" vs "4-state" — the number is just how many buckets you need:**
- Problem 6 (fee): 2 buckets — `hold`, `cash`. No extra constraint, so 2 is enough.
- Problem 5 (cooldown): 3 buckets — because "just sold" and "resting/eligible to buy" behave differently (you can't buy the day right after selling), so they can't share a bucket.
- Problem 3 (at most 2 transactions): 4 buckets — because "holding on my 1st purchase" and "holding on my 2nd purchase" must be tracked separately, or you'd lose track of how many transactions you've spent.

**This is the actual distinguishing test:** if two situations ever need *different* future treatment (different allowed next moves, or different transaction budget remaining), they're different states. If they'd always be treated identically going forward, merge them into one state.

## 4. DP table over (day, transactions-used) — same idea, just made variable-sized

This is state-machine DP where the number of states isn't fixed by the problem's shape but by input `k`. Instead of hand-naming 4 variables (`buy1, sell1, buy2, sell2`), you use an array indexed by "how many transactions used," because `k` isn't known until runtime.

## Quick Disambiguation Cheatsheet

| If you hear... | It means... |
|---|---|
| "Greedy" | No memory of alternatives needed; locally best = globally best, provably |
| "Running min/max" | One scalar carried forward; not proper DP |
| "State-machine DP" / "k-state DP" | Fixed small set of named situations, each with its own tracked best-value, updated via transitions each step |
| "DP table" | Same as above, but states are indexed by a variable (like `k`) rather than hand-named |