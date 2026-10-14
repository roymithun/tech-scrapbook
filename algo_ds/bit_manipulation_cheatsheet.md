# Bit Manipulation Cheat Sheet (Kotlin)

Core operators, common tricks, and patterns for solving bit manipulation problems.

## Basic Operators

| Operator | Symbol | Example | Description |
|---|---|---|---|
| AND | `&` | `5 and 3` → `1` | 1 only if both bits are 1 |
| OR | `\|` | `5 or 3` → `7` | 1 if either bit is 1 |
| XOR | `^` | `5 xor 3` → `6` | 1 if bits differ |
| NOT | `~` (`.inv()`) | `5.inv()` → `-6` | Flips all bits |
| Left Shift | `<<` (`shl`) | `1 shl 3` → `8` | Multiply by 2^n |
| Right Shift (signed) | `>>` (`shr`) | `-8 shr 1` → `-4` | Floor division by 2, sign-extends (fills with copies of the sign bit) |
| Unsigned Right Shift | `>>>` (`ushr`) | `-8 ushr 1` → large positive | Divide by 2 treating bits as unsigned, fills with 0 |

In Kotlin these are infix functions, not symbols: `a and b`, `a or b`, `a xor b`, `a shl n`, `a shr n`, `a ushr n`, `a.inv()`.

## Fundamental Identities

```
a ^ a = 0            // XOR of a number with itself is 0
a ^ 0 = a             // XOR with 0 is identity
a & 0 = 0
a | 0 = a
a & (~a) = 0
a | (~a) = -1         // all bits set
a ^ (~0) = ~a         // XOR with all-1s flips bits
```

## Set / Toggle / Check / Clear — The Core Mnemonic

The four most common single-bit operations map directly onto four operators — worth memorizing as a group since they cover most bitmask manipulation:

| Operation | Operator | Kotlin |
|---|---|---|
| **Set** bit `i` | `\|` (OR) | `n or (1 shl i)` |
| **Toggle** bit `i` | `^` (XOR) | `n xor (1 shl i)` |
| **Check** bit `i` | `&` (AND) | `n and (1 shl i) != 0` |
| **Clear** bit `i` | `& ~` (AND-NOT) | `n and (1 shl i).inv()` |

Why each works: OR forces a `1` in regardless of the prior value ("set"); XOR flips whatever was there ("toggle"); AND isolates just that bit so the result is nonzero only if it was already `1` ("check"); AND with an inverted mask keeps every other bit untouched while forcing the target to `0` ("clear").

## Essential Tricks

| Goal | Expression | Notes |
|---|---|---|
| Clear lowest set bit | `n and (n - 1)` | Used in popcount, power-of-2 check |
| Isolate lowest set bit | `n and (-n)` | Two's complement trick |
| Clear lowest set bit... and beyond isn't needed | — | — |
| Check power of two | `n > 0 && (n and (n - 1)) == 0` | Only one bit set |
| Turn off rightmost 0 bit (fill with 1) | `n or (n + 1)` | Less common |
| Get rightmost set bit position | `Integer.numberOfTrailingZeros(n)` | Or loop with `& 1` |
| Count set bits (popcount) | `Integer.bitCount(n)` | Built-in in Kotlin/JVM |
| Swap two variables without temp | `a = a xor b; b = a xor b; a = a xor b` | Classic XOR swap |
| Check if two numbers have opposite signs | `(a xor b) < 0` | Sign bit differs |
| Get absolute value (no branch) | `val mask = n shr 31; (n xor mask) - mask` | 32-bit trick |
| Round up to next power of 2 | see snippet below | Bit-spreading |

## Built-in Kotlin/JVM Helpers

```kotlin
Integer.bitCount(n)              // number of 1 bits
Integer.numberOfLeadingZeros(n)  // leading zeros in 32-bit int
Integer.numberOfTrailingZeros(n) // trailing zeros
Integer.highestOneBit(n)         // isolates the highest set bit
Integer.lowestOneBit(n)          // isolates the lowest set bit (same as n and -n)
Integer.reverse(n)               // reverses all 32 bits
Integer.toBinaryString(n)        // string representation, useful for debugging
```

64-bit equivalents exist under `java.lang.Long` (`Long.bitCount`, etc.) for `Long` values in Kotlin.

## Snippets

**Count set bits manually (Brian Kernighan's algorithm):**
```kotlin
fun countBits(nIn: Int): Int {
    var n = nIn
    var count = 0
    while (n != 0) {
        n = n and (n - 1)  // clears lowest set bit
        count++
    }
    return count
}
```

**Check power of two:**
```kotlin
fun isPowerOfTwo(n: Int): Boolean = n > 0 && (n and (n - 1)) == 0
```

**Get all subsets of a set using bitmask:**
```kotlin
fun subsets(nums: IntArray): List<List<Int>> {
    val n = nums.size
    val result = mutableListOf<List<Int>>()
    for (mask in 0 until (1 shl n)) {
        val subset = mutableListOf<Int>()
        for (i in 0 until n) {
            if ((mask shr i) and 1 == 1) subset.add(nums[i])
        }
        result.add(subset)
    }
    return result
}
```

**Iterate over all submasks of a bitmask `m` (classic trick):**
```kotlin
var sub = m
while (sub > 0) {
    // process `sub`
    sub = (sub - 1) and m
}
// don't forget to process the empty submask (0) separately if needed
```

**Round up to next power of 2:**
```kotlin
fun nextPowerOfTwo(nIn: Int): Int {
    var n = nIn - 1
    n = n or (n shr 1)
    n = n or (n shr 2)
    n = n or (n shr 4)
    n = n or (n shr 8)
    n = n or (n shr 16)
    return n + 1
}
```

**Add two integers without `+`:**
```kotlin
fun add(aIn: Int, bIn: Int): Int {
    var a = aIn
    var b = bIn
    while (b != 0) {
        val carry = a and b
        a = a xor b
        b = carry shl 1
    }
    return a
}
```

**XOR-based single-number pattern (find the one non-duplicate):**
```kotlin
fun singleNumber(nums: IntArray): Int = nums.reduce { acc, n -> acc xor n }
```

## Mental Model Cheat Sheet

| If the problem involves... | Think... |
|---|---|
| "Every element appears twice except one" | XOR everything |
| "Every element appears K times except one" | Bit-counting mod K, or state machine |
| Subsets / combinations of a small set (n ≤ ~20) | Bitmask enumeration `0` to `2^n - 1` |
| "Minimum steps / shortest path across states" with small state space | Bitmask DP: state = (position, visited-mask) |
| Counting bits across a range | DP relation `bits[i] = bits[i >> 1] + (i & 1)` |
| Maximizing XOR of pairs | Binary trie, greedy from MSB to LSB |
| No `+`/`-` allowed | XOR = sum without carry, AND+shift = carry |
| Range AND of `[m, n]` | Right-shift both until equal (common prefix) |
| Check/set/clear specific bit | `shr`/`and`/`or`/`xor` with `1 shl i` |
| Power of two / four checks | `n and (n-1)` and/or bit position checks |

## `shr` Nuance: Floor Division, Not Truncation

`n shr 1` is often described as "divide by 2," but that's only exact when there's no remainder. For negative odd numbers, `shr` **floors toward negative infinity**, while Kotlin's `/` operator **truncates toward zero** — these can disagree by 1.

```kotlin
println(-7 / 2)     // -3   (truncating division)
println(-7 shr 1)    // -4  (floor division via sign-extending shift)
```

**Why:** `shr` works by shifting bits right and filling the new leading bits with copies of the sign bit (sign extension) — it has no concept of "rounding," it's purely a bit operation that happens to behave like floor division on two's complement numbers.

```
-7 (32-bit):   1111 ... 1111 1001
shr 1:         1111 ... 1111 1100   = -4
```

Rule of thumb: `shr` and `/` agree for positive numbers and evenly-divisible negatives, but can differ by exactly 1 for odd negatives.

## Sign Extension (padding across widths)

When converting a signed bit pattern from a smaller width (e.g., 32-bit `Int`) to a larger width (e.g., 64-bit `Long`), you must pad with **copies of the sign bit**, not zeros — this is what `shr` does internally, and it's also what you'd do manually to reinterpret a 32-bit two's-complement pattern as 64-bit.

```
32-bit -6:  1111 1111 1111 1111 1111 1111 1111 1010
64-bit -6:  1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1010
```
(32 extra `1`s prepended — padding with `0`s instead would turn it into a large positive number.)

```kotlin
val a: Int = 5
println(a.inv())        // -6   (32-bit)
println(a.inv().toLong()) // -6L (sign-extended to 64-bit automatically)
```

## Quick Reference: Sign & Overflow Notes (Kotlin/JVM)

- `Int` is 32-bit, `Long` is 64-bit, both signed two's complement — no unsigned primitive types by default.
- `shr` is **arithmetic** (sign-preserving); `ushr` is **logical** (zero-fills). Use `ushr` when treating a negative `Int` as a raw bit pattern.
- `n and -n` relies on two's complement representation of negative numbers — works the same in Kotlin as in Java/C++.
- Watch for shift-amount wraparound: shifting by ≥32 on an `Int` wraps modulo 32 (e.g., `1 shl 32 == 1 shl 0`).