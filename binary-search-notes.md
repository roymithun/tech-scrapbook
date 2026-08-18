# Binary Search: `low <= high` vs `low < high`

## The question I kept confusing myself on
Why do some binary search problems use `while (low <= high)` and others use `while (low < high)`?

## The answer: it depends on whether `mid` can be fully eliminated

There are two patterns. Once I know which one I'm in, the loop condition follows automatically — it's not something to memorize separately.

---

### Pattern A — Exact match search
**Use `low <= high`**

This is for when I'm looking for a specific value or condition, and every check on `mid` either:
- finds the answer right there, or
- proves `mid` is definitely NOT the answer, so I can throw it away completely

Because `mid` gets fully discarded each time:
```kotlin
low = mid + 1
high = mid - 1
```

Both sides shrink *past* `mid`. So I need `low <= high` to make sure I still check the very last remaining candidate (when `low == high`).

I return the answer *during* the loop, or return "not found" after the loop ends.

```kotlin
while (low <= high) {
    val mid = (low + high) ushr 1
    when {
        nums[mid] == target -> return mid
        nums[mid] < target -> low = mid + 1
        else -> high = mid - 1
    }
}
return -1
```

---

### Pattern B — Boundary / convergence search
**Use `low < high`**

This is for when `mid` might *be* the answer, so I can't eliminate it outright. I can only narrow toward it.

Because `mid` stays in the range:
```kotlin
low = mid + 1   // still safe to exclude mid on this side
high = mid      // NOT mid - 1, because mid might be the answer
```

If I used `low <= high` here, once `low == high`, `mid` would equal `low` and `high` too. The `else` branch (`high = mid`) would just reassign `high` to itself — **infinite loop**.

So the loop must stop exactly when `low == high`, meaning: **`low < high`**.

At that point, `low` and `high` have converged onto the single correct index, and I return `nums[low]` (or just `low`) — no need to check anything, the loop invariant guarantees it's correct.

```kotlin
while (low < high) {
    val mid = (low + high) ushr 1
    if (nums[mid] > nums[high]) low = mid + 1
    else high = mid
}
return nums[low]
```

---

## How I decide which pattern to use — 3 questions

1. **Once I check `mid`, can I rule it out completely?**
   - Yes → Pattern A (`low <= high`)
   - No, it might still be the answer → Pattern B (`low < high`)

2. **What do I return?**
   - A value found *inside* the loop → Pattern A
   - `nums[low]` or `low` *after* the loop → Pattern B

3. **What kind of problem is this?**
   - "Find this exact target" → Pattern A
   - "Find the first true / boundary / minimum / peak" → Pattern B

## The tell in the code itself
- If both updates have an offset (`mid + 1` and `mid - 1`) → must be `low <= high`
- If either update has NO offset (`high = mid` or `low = mid`) → must be `low < high`, otherwise infinite loop risk

## Quick reference table

| Problem type | Updates | Condition | Return |
|---|---|---|---|
| Find exact target | `mid+1` / `mid-1` | `low <= high` | during loop, or `-1` |
| Min in rotated sorted array | `mid+1` / `mid` | `low < high` | `nums[low]` |
| First `True` in `[F,F,F,T,T,T]` | `mid+1` / `mid` | `low < high` | `low` |
| Find peak element | `mid+1` / `mid` | `low < high` | `low` |
| Last occurrence (rightmost) | `mid` / `mid-1` (use `mid = (low+high+1)/2`) | `low < high` | `low` |
