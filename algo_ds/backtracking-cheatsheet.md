# Backtracking — Cheat Sheet

## What is Backtracking?

Backtracking is an algorithmic technique for solving problems incrementally —
building a solution piece by piece, and abandoning ("backtracking" from) a
partial solution as soon as you determine it can't possibly lead to a valid
complete solution.

Think of it as smart brute force: instead of generating every possible full
solution and then checking each one, you check validity *as you build*, so
you can prune whole branches of possibilities early.

The mental model: **explore → check → either continue deeper, or undo and
try the next option.**

It's essentially a **DFS (depth-first search) over a decision tree**, where
each node represents a partial solution, and children represent the choices
you can make next.

---

## How to Identify a Backtracking Problem

Look for these signals:

1. **You need to find all (or one) solutions/combinations/arrangements**
   that satisfy some constraint.
   - "Find all subsets", "all permutations", "all ways to place N queens",
     "all paths"
2. **The problem is built from a sequence of choices**, and each choice
   narrows the remaining options.
3. **There's a constraint that can invalidate a partial solution early** (so
   you don't need to build the whole thing to know it's wrong) — e.g., a
   queen attacking another, a Sudoku cell conflict, exceeding a target sum.
4. **Keywords**: "all possible ways," "combinations," "permutations,"
   "partition," "arrangements," "count the number of ways," "is it possible
   to," "generate all valid..."
5. Contrast with **DP (dynamic programming)**: if subproblems overlap and
   you only need an optimal value/count (not all actual solutions), DP might
   be better/faster. Backtracking is usually for **enumeration** or when
   constraints are too irregular for DP.

Classic backtracking problems: N-Queens, Sudoku solver, permutations /
subsets / combinations, word search in a grid, generating valid
parentheses, combination sum, graph coloring, Hamiltonian path, crossword
filling.

---

## Basic Steps to Build a Solution

There's a fairly universal template:

```
function backtrack(state):
    if state is a complete solution:
        record/return it
        return

    for choice in get_possible_choices(state):
        if is_valid(choice, state):        # pruning check
            make_choice(state, choice)      # move forward
            backtrack(state)                # recurse
            undo_choice(state, choice)       # <-- the "backtrack" step
```

Step-by-step thinking process:

1. **Define the state** — what represents a partial solution (e.g., a
   partial permutation, a board configuration, an index into an array).
2. **Define the choices** at each step — what can you try next?
3. **Define the base case** — when is the state a complete, valid solution
   (or clearly invalid, or clearly out of bounds)?
4. **Define the pruning/validity check** — the constraint check that lets
   you cut off invalid branches *early*, before wasting time going deeper.
5. **Make the choice → recurse → undo the choice.** The "undo" step
   (removing the last placed queen, removing the last chosen number from a
   subset, etc.) is what makes it *backtracking* rather than plain
   recursion.

---

## Common Variations

- **Enumerate all solutions** (e.g., all permutations) — no pruning
  shortcut, just collect every complete path.
- **Find one solution / feasibility check** (e.g., Sudoku, N-Queens "does a
  solution exist") — stop as soon as one valid complete solution is found
  (return `true`/solution immediately).
- **Count the number of solutions** — same traversal, but just increment a
  counter at the base case instead of storing results.
- **Optimal solution (backtracking + pruning bounds)** — sometimes merged
  with **branch and bound**: you track a "best so far" and prune branches
  that can't beat it (common in optimization problems like TSP).
- **Constraint satisfaction problems (CSP)** — Sudoku, graph coloring,
  crossword — where you also often add smarter heuristics like **choosing
  the most constrained variable next** or **forward checking** (looking
  ahead to eliminate impossible future choices, not just checking the
  current one).
- **Backtracking with memoization** — sometimes you can cache partial
  results if subproblems repeat, blending into DP (careful: only works if
  the "partial state" is reusable and doesn't have result-order
  dependence).

---

## Cheat Sheet / Common Approach

**Template shape (choose → explore → un-choose):**

```python
def backtrack(path, options):
    if is_solution(path):
        results.append(path[:])   # copy! important for lists/arrays
        return
    for option in options:
        if not is_valid(path, option):
            continue
        path.append(option)                 # choose
        backtrack(path, remaining(options, option))  # explore
        path.pop()                          # un-choose
```

**Checklist when approaching a new problem:**

| Question | Why it matters |
|---|---|
| What does a "state" look like? | Defines your recursion parameter |
| What are the choices at each state? | Defines your loop/branches |
| What makes a state complete? | Base case |
| What makes a choice invalid? | Where you prune |
| Do you need all solutions, one, or a count? | Changes what happens at base case & whether you return early |
| Can you prune earlier/smarter? | Big performance factor — e.g. sort input, use sets for "used" tracking, add bounds |
| Are you copying state correctly before storing? | Common bug: storing a reference to a mutable list that gets mutated later |

**Common pitfalls:**

- Forgetting to **undo** a choice (state leaks between branches).
- Storing a reference instead of a copy of the current path.
- Not pruning early enough (technically correct but very slow — check
  `is_valid` before recursing, not after).
- Confusing backtracking with plain DFS/recursion — the "undo" step and
  pruning are what distinguish it.

**Complexity intuition:** Backtracking is typically exponential in the
worst case (you're still exploring a tree of possibilities), but good
pruning can make it fast in practice — sometimes dramatically so. The value
of a good pruning condition is often the entire difference between a
solution that finishes instantly and one that never finishes.
