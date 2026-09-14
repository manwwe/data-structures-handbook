# Big-O Notation

Big-O describes how the **time or space** used by an algorithm **grows** as the input size gets larger. It gives us a ceiling on how fast that growth can be as the input gets larger.

Big-O is commonly used to describe worst-case complexity, but Big-O itself does not necessarily mean worst case.

The most important things Big-O is not:

- **It isn't the exact running time.** It doesn't tell you how many seconds an algorithm will take.
- **It isn't the exact amount of work.** Two algorithms that grow at the same rate can still do different amounts of work.
- **It doesn't always tell you which algorithm is faster.** An algorithm that grows more slowly can still be slower for small inputs.

## Common Big-O Complexities

The following are some of the most common Big-O complexities. They show different ways the number of steps can grow as the input size grows.

### Constant Time Complexity — O(1)

The number of steps stays constant and does not depend on the input size.

**Example:**
If `n = 10 → 1`
If `n = 1,000 → 1`

### Logarithmic Time Complexity — O(log n)

The algorithm often reduces the input size by half at each step. The number of steps grows slowly as the input grows.

**Example:**
If `n = 8 → log₂(8) = 3`
If `n = 1,024 → log₂(1,024) = 10`

### Linear Time Complexity — O(n)

The algorithm goes through each element in the input. The number of steps grows at the same rate as the input size.

**Example:**
If `n = 10 → n = 10`
If `n = 100 → n = 100`

### Linearithmic Time Complexity — O(n log n)

The algorithm goes through all `n` elements while taking about `log n` steps for each one.

**Example:**
If `n = 8 → n log₂(n) = 8 × 3 = 24`
If `n = 16 → n log₂(n) = 16 × 4 = 64`

### Bilinear Time Complexity — O(n × m)

The algorithm goes through two inputs, comparing each element of the first input with each element of the second.

**Example:**
If `n = 3` and `m = 4 → n × m = 3 × 4 = 12`
If `n = 5` and `m = 10 → n × m = 5 × 10 = 50`

### Quadratic Time Complexity — O(n²)

The number of steps grows roughly with the input size squared. A common example is going through every possible pair of elements in the input.

**Example:**
If `n = 10 → n² = 100`
If `n = 100 → n² = 10,000`

### Cubic Time Complexity — O(n³)

The number of steps grows roughly with the input size cubed. A common example is going through every possible combination of three elements in the input.

**Example:**
If `n = 10 → n³ = 1,000`
If `n = 100 → n³ = 1,000,000`

### Exponential Time Complexity — O(2ⁿ)

The algorithm often goes through all possible subsets of the input elements. The number of steps doubles with each additional element in the input.

**Example:**
If `n = 5 → 2ⁿ = 2⁵ = 32`
If `n = 10 → 2ⁿ = 2¹⁰ = 1,024`

### Factorial Time Complexity — O(n!)

The algorithm often goes through all possible permutations of the input elements. The number of steps grows extremely quickly as the input size increases.

**Example:**
If `n = 5 → n! = 5! = 120`
If `n = 10 → n! = 10! = 3,628,800`

> **Note:** These examples illustrate the growth of each complexity. The results do not represent the exact number of operations performed by an algorithm.

## Practice Exersices

## Before the Exercises

Before starting with the exercises, let's go over a few important concepts to help us understand the solutions.

- **Time complexity** describes how the amount of work grows as the input gets larger. We usually count operations or steps, rather than seconds.

- **Space complexity** describes how the amount of memory needed grows as the input gets larger. Be clear about whether you count all memory used, including the input, or only the extra memory the algorithm needs. This extra memory is called **auxiliary space**.

- **Input size** is what variables like `n` represent. For an array, `n` usually means the number of elements. When two inputs can grow independently, use separate variables, such as `n` and `m`.

- **Simplifying Big-O** means dropping constant factors and lower-order terms to focus on how work or memory grows for large inputs. For example, `3n² + 5n + 10` becomes `O(n²)`. The squared term dominates as `n` grows, and its constant multiplier does not change the growth category.

- **Consecutive steps** have costs that add up. For example, an `O(n)` loop followed by an `O(n²)` loop gives `O(n + n²)`, which simplifies to `O(n²)`.

- **Nested loops** require counting how often the inner work runs overall. Two nested loops do not automatically mean `O(n²)`. If the outer loop runs `n` times and the inner loop runs a fixed 5 times per outer iteration, with constant work each time, the total is `5n`, or `O(n)`.

- **The case being analyzed** matters. The best case describes the least work for an input of a given size, and the worst case describes the most work. The average case describes the expected work under stated assumptions about the inputs. Always make clear which case a solution considers.
  Filter files

[Exercises in C](exercises/c.md)
[Exercises in C++](exercises/cpp.md)
[Exercises in Python](exercises/python.md)
