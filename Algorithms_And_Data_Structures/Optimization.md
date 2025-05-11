# 📌 Memoization

## 🧠 What is Memoization?

**Memoization** is an optimization technique used primarily to speed up **function calls** by storing the results of expensive function executions and returning the cached result when the **same inputs** occur again.

It's a specific form of **caching** applied to function results.

---

## 🔍 Why Use Memoization?

- 🚀 Improves performance for functions with **repeated calls and deterministic output** (same input → same output).
- 💡 Especially helpful in **recursive** or **computationally expensive** algorithms (e.g., Fibonacci sequence, dynamic programming problems).

---

## 🛠️ How It Works

1. A function is wrapped so that before performing its computation, it checks if it has already seen the input.
2. If yes, it returns the stored result.
3. If not, it computes the result, stores it, and then returns it.

---

## ✅ Example in Kotlin

```kotlin
fun <T, R> memoize(function: (T) -> R): (T) -> R {
    val cache = mutableMapOf<T, R>()
    return { input: T ->
        cache.getOrPut(input) { function(input) }
    }
}

// Example usage with Fibonacci
val slowFib: (Int) -> Long = { n ->
    if (n <= 1) n.toLong() else slowFib(n - 1) + slowFib(n - 2)
}

val fastFib = memoize(slowFib)

fun main() {
    println(fastFib(40)) // Much faster
}

