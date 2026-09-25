# Complexity Analysis in C++

Read the [theory](README.md) first, then work through the language examples and exercises.

## Analyze Code Step by Step

Identify the input size, count how often each operation runs, and separate time from auxiliary space. Consecutive loops add their costs; nested loops require counting the total inner work. State whether you are analyzing the best or worst case. Count recursive calls and the maximum number of simultaneously active calls separately.

The examples use C++17. Analyze the operations inside a loop as well as the loop count; a library call may perform work proportional to its input.

## Basic Exercises

These C++17 examples mirror the C exercises so you can compare the same algorithms. Include `<iostream>` for output and `<new>` for the allocation example. Treat each exercise as a separate program fragment; some reuse function names.

Assume valid array lengths and indices, and that integer calculations fit in `int`. Built-in arrays keep indexing and loop costs explicit. Output of a fixed-size integer is treated as constant work for these analyses.

### Exercise 1: Two Loops

What is the time complexity of this code?

```cpp
void my_func(int array[], int n) {
    int sum = 0;
    int product = 1;

    for (int i = 0; i < n; i++) {
        sum += array[i];
    }

    for (int i = 0; i < n; i++) {
        product *= array[i];
    }

    std::cout << "Sum: " << sum << ", Product: " << product << "\n";
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n)`**

The first loop goes through all `n` elements to calculate the sum. The second goes through them again to calculate the product.
Because the loops run one after the other, their costs add up:

```text
O(n) + O(n) = O(2n) = O(n)
```

**Time complexity:** Going through the same array twice still takes linear time. In Big-O notation, constant factors are ignored.

**Space complexity:** O(1) auxiliary space. The function uses a fixed number of variables, regardless of the array’s size.

</details>

---

### Exercise 2: Nested Loops

What is the time complexity of this code?

```cpp
void print_pairs(int array[], int n) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            std::cout << "Pair: array[" << i << "] = " << array[i] << ", array[" << j << "] = " << array[j] << "\n";
        }
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n²)`**

The outer loop runs `n` times. For each outer iteration, the inner loop goes through all `n` elements. This means the function prints `n × n` pairs.

For example, if `n = 3`, it prints `3 × 3 = 9` pairs, including pairs where both elements come from the same position.

**Time complexity:** `O(n²)`. Each of the `n` outer iterations does `n` iterations of constant work.

**Space complexity:** `O(1)` auxiliary space. The function prints each pair immediately without storing the pairs.

</details>

---

### Exercise 3: A Shrinking Inner Loop

What is the time complexity of this code?

```cpp
void print_pairs(int array[], int n) {
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            std::cout << "Pair: array[" << i << "] = " << array[i] << ", array[" << j << "] = " << array[j] << "\n";
        }
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n²)`**

The inner loop starts at `i + 1`, so each element is paired only with elements after it. Each pair of positions is visited once, and no position is paired with itself.

The inner loop runs `n - 1` times on the first outer iteration, then `n - 2` times, and so on, down to zero. The total number of printed pairs is:

```text
(n - 1) + (n - 2) + ... + 1 + 0 = n(n - 1) / 2
```

For example, if `n = 3`, the function prints `2 + 1 + 0 = 3` pairs. This counts executions of `std::cout`, not every loop check or increment.

**Time complexity:** `O(n²)`. Expanding the pair count gives `(n² - n) / 2`. Dropping the constant factor and lower-order term leaves quadratic growth, even though the inner loop gets shorter.

**Space complexity:** `O(1)` auxiliary space. The function prints each pair immediately without storing the pairs.

</details>

---

### Exercise 4: Two Arrays and a Condition

What is the time complexity of this code? The arrays can have different lengths.

```cpp
void print_unordered_pairs(int array_a[], int array_b[], int len_array_a, int len_array_b) {
    for (int i = 0; i < len_array_a; i++) {
        for (int j = 0; j < len_array_b; j++) {
            if (array_a[i] < array_b[j]) {
                std::cout << "Match: array_a[" << i << "] = " << array_a[i] << " < array_b[" << j << "] = " << array_b[j] << "\n";
            }
        }
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n × m)`**

Let `n = len_array_a` and `m = len_array_b`. For each of the `n` elements in `array_a`, the inner loop checks all `m` elements in `array_b`. The condition is evaluated `n × m` times, whether or not any pairs are printed.

For example, if `n = 3` and `m = 4`, the function makes `3 × 4 = 12` element comparisons. It may print anywhere from 0 to 12 pairs, depending on the values.

**Time complexity:** `O(n × m)` in both the best and worst cases for nonempty arrays. The condition controls printing, but it does not skip any loop iterations.

Keep both variables because the array lengths can grow independently. If both lengths were `n`, this would become `O(n²)`.

**Space complexity:** `O(1)` auxiliary space. The function prints matching pairs immediately without storing them.

</details>

---

### Exercise 5: Accessing One Element

What are the time and space complexities of this code?

```cpp
void print_first(int array[], int n) {
    if (n > 0) {
        std::cout << "First element: " << array[0] << "\n";
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(1)`**

The function checks the length and prints at most one element. It does not go through the array.

**Time complexity:** O(1). The amount of work stays bounded as the array grows.

**Space complexity:** O(1). No extra storage grows with the input.

</details>

---

### Exercise 6: Skipping Every Other Element

What are the time and space complexities of this code?

```cpp
void print_every_other(int array[], int n) {
    for (int i = 0; i < n; i += 2) {
        std::cout << "Element at index " << i << ": " << array[i] << "\n";
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n)`**

The loop visits indices 0, 2, 4, and so on. For n = 6, it prints 3 elements. In general, it visits about half the array.

**Time complexity:** O(n). Dropping the constant factor from roughly n / 2 iterations leaves linear growth.

**Space complexity:** O(1). Only a fixed number of variables are needed.

</details>

---

### Exercise 7: Halving a Number

What are the time and space complexities of this code?

```cpp
void print_halves(int n) {
    while (n > 1) {
        std::cout << "Current value: " << n << "\n";
        n /= 2;
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(log n)`**

Assume n >= 1. Each iteration halves n using integer division. For n = 16, the printed values are 16, 8, 4, and 2: four iterations.

**Time complexity:** O(log n). The number of halvings grows logarithmically with the starting value of n.

**Space complexity:** O(1). The function updates one variable.

</details>

---

### Exercise 8: A Fixed Inner Loop

What are the time and space complexities of this code?

```cpp
void print_three_times(int array[], int n) {
    for (int i = 0; i < n; i++) {
        for (int repeat = 0; repeat < 3; repeat++) {
            std::cout << "Repeat " << (repeat + 1) << ", element: " << array[i] << "\n";
        }
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n)`**

The inner loop runs exactly 3 times for each element. There are 3n print calls, even though the code has two nested loops.

**Time complexity:** O(n). The inner loop has a fixed size, so dropping the factor of 3 leaves linear growth.

**Space complexity:** O(1). The counters use a fixed amount of memory.

</details>

---

### Exercise 9: A Halving Loop for Each Element

What are the time and space complexities of this code?

```cpp
void print_element_levels(int array[], int n) {
    for (int i = 0; i < n; i++) {
        for (int remaining = n; remaining > 1; remaining /= 2) {
            std::cout << "Element: " << array[i] << ", remaining: " << remaining << "\n";
        }
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n log n)`**

For each of the n elements, the inner loop starts over at n and repeatedly halves its counter. With n = 8, there are 8 × 3 = 24 print calls.

**Time complexity:** O(n log n). Multiply n outer iterations by logarithmic inner work.

**Space complexity:** O(1). The function reuses a fixed number of variables.

</details>

---

### Exercise 10: Going Through Two Arrays Separately

What are the time and space complexities of this code?

```cpp
void print_two_arrays(int array_a[], int len_array_a,
                      int array_b[], int len_array_b) {
    for (int i = 0; i < len_array_a; i++) {
        std::cout << "Array A element: " << array_a[i] << "\n";
    }
    for (int j = 0; j < len_array_b; j++) {
        std::cout << "Array B element: " << array_b[j] << "\n";
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n + m)`**

Let n = len_array_a and m = len_array_b. The loops run one after the other, visiting n elements and then m elements.

**Time complexity:** O(n + m). Add the costs. Keep both variables because the lengths can grow independently.

**Space complexity:** O(1). No extra storage grows with either array.

</details>

---

### Exercise 11: Stopping When a Match Is Found

What are the time and space complexities of this code?

```cpp
int find_value(int array[], int n, int target) {
    for (int i = 0; i < n; i++) {
        if (array[i] == target) {
            return i;
        }
    }
    return -1;
}
```

<details>
<summary>Show solution</summary>

**Answer: `Best case: O(1). Worst case: O(n).`**

For a nonempty array, the best case occurs when the first element matches. The worst case occurs when the target is absent or only the last element matches, so the function checks all n elements.

**Time complexity:** O(1) in the best case and O(n) in the worst case. An early return can change how much work actually runs.

**Space complexity:** O(1). The search uses a counter and returns a single index.

</details>

---

### Exercise 12: Three Nested Loops

What are the time and space complexities of this code?

```cpp
void print_triples(int array[], int n) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            for (int k = 0; k < n; k++) {
                std::cout << "Triple: " << array[i] << ", " << array[j] << ", " << array[k] << "\n";
            }
        }
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n³)`**

Each of the three loops runs n times for every iteration of the loop outside it. For n = 3, the function prints 3 × 3 × 3 = 27 triples.

**Time complexity:** O(n³). The print call runs n × n × n times.

**Space complexity:** O(1). Printing immediately avoids storing the triples.

</details>

---

### Exercise 13: Copying an Array

What are the time and space complexities of this code?

```cpp
int *copy_array(int array[], int n) {
    if (n <= 0) {
        return nullptr;
    }
    int *copy = new (std::nothrow) int[n];
    if (copy == nullptr) {
        return nullptr;
    }
    for (int i = 0; i < n; i++) {
        copy[i] = array[i];
    }
    return copy;
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n) time and O(n) output space.`**

Assume a positive length, a valid allocation size, and successful allocation. The loop copies each of the n elements once. Include `<new>` for `std::nothrow`; the caller must release the returned array with `delete[] copy` when finished. A non-throwing allocation returns `nullptr` when storage cannot be obtained; assume the requested array length is valid.

**Time complexity:** O(n) for the successful copy under the usual allocation cost model. The loop performs n assignments.

**Space complexity:** O(n) for the returned array. If output storage is excluded, auxiliary space is O(1). The input array is not copied onto the call stack.

</details>

---

### Exercise 14: A Recursive Countdown

What are the time and space complexities of this code?

```cpp
void print_countdown(int n) {
    if (n <= 0) {
        return;
    }
    std::cout << "Countdown: " << n << "\n";
    print_countdown(n - 1);
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n)`**

For n >= 0, each call reduces n by 1 until it reaches zero. There are n print calls and one final call that returns immediately.

**Time complexity:** O(n). Each call does constant work before making one smaller call.

**Space complexity:** O(n) auxiliary space for the call stack, assuming no tail-call optimization. Up to n + 1 calls are active at once; each stores a fixed amount of information.

</details>

---

### Exercise 15: Two Recursive Calls

What are the time and space complexities of this code?

```cpp
void print_branches(int n) {
    if (n <= 0) {
        std::cout << "Reached a leaf\n";
        return;
    }
    print_branches(n - 1);
    print_branches(n - 1);
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(2ⁿ)`**

For n >= 0, each non-base call makes two calls with n - 1. The number of calls at each depth is 1, 2, 4, and so on. With n = 3, there are 8 leaf prints and 15 calls in total. In general, the total call count is 2^(n + 1) - 1.

**Time complexity:** O(2ⁿ). The branching doubles the number of calls at each depth.

**Space complexity:** O(n) auxiliary space for the call stack, assuming no call optimization. The two branches run one after the other, so only one path of up to n + 1 calls is active at a time.

</details>

---

### Exercise 16: Recursive Fibonacci

What are the time and space complexities of this code? Assume `n >= 0` and that the result fits in `int`.

```cpp
int fibonacci(int n) {
    if (n <= 1) {
        return n;
    }
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(2ⁿ)` time and `O(n)` auxiliary space.**

Each non-base call asks for the two previous Fibonacci numbers. Many of these calculations repeat: for example, `fibonacci(5)` calculates `fibonacci(3)` through both branches.

**Time complexity:** `O(2ⁿ)` is a valid upper bound because each call makes at most two smaller calls and the depth grows with `n`. The branches have different depths, so this is not an exact call count. A tighter growth bound is `Θ(φⁿ)`, where `φ` is about 1.618; `Θ` describes a matching upper and lower growth bound.

**Space complexity:** `O(n)` for the call stack. The deepest path reduces `n` by 1 each time. The calls do not all stay active at once.

</details>

---

### Exercise 17: Recursively Summing an Array

What are the time and space complexities of this code? Assume `n >= 0` and that the sums fit in `int`.

```cpp
int sum_array(int array[], int n) {
    if (n == 0) {
        return 0;
    }
    return array[n - 1] + sum_array(array, n - 1);
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n)` time and `O(n)` auxiliary space.**

Each call handles the last element of the remaining portion and asks the next call to sum the earlier elements. For `[2, 4, 6]`, the additions work out to `6 + 4 + 2 + 0 = 12`.

**Time complexity:** `O(n)`. There are `n + 1` calls, including the base case. Each call does constant work outside the recursive call.

**Space complexity:** `O(n)` for the call stack under the usual model without recursion optimization. The pending calls wait for the smaller sum before adding their element. Passing the array does not copy it; the extra memory comes from the calls.

</details>

---

### Exercise 18: Printing Permutations

What are the time and space complexities of this code? Start with `print_permutations(array, 0, n)`. Assume `n >= 1` and distinct array values.

```cpp
void swap_values(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

void print_permutations(int array[], int start, int n) {
    if (start == n) {
        std::cout << "Permutation:";
        for (int i = 0; i < n; i++) {
            std::cout << " " << array[i];
        }
        std::cout << "\n";
        return;
    }

    for (int i = start; i < n; i++) {
        swap_values(&array[start], &array[i]);
        print_permutations(array, start + 1, n);
        swap_values(&array[start], &array[i]);
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n × n!)` time and `O(n)` auxiliary space.**

The function picks a value for the current position, then fills the remaining positions recursively. After each recursive call, it swaps back to restore the array before trying the next choice. This is called **backtracking**.

There are `n` choices for the first position, `n - 1` for the second, and so on. Multiplying gives `n!` permutations. For `[1, 2, 3]`, the function prints 6 permutations, each containing 3 values.

**Time complexity:** `O(n × n!)`. There are `n!` permutations, and printing each one takes `O(n)` work. The printing cost matters: this full function is not just `O(n!)`.

**Space complexity:** `O(n)` for the recursive call stack. The function rearranges the existing array and restores it afterward. It prints each permutation immediately, so it does not store all `n!` permutations.

</details>

---

### Exercise 19: Reversing an Array

What are the time and space complexities of this code? Assume `n >= 0`.

```cpp
void reverse_array(int array[], int n) {
    int left = 0;
    int right = n - 1;

    while (left < right) {
        int temp = array[left];
        array[left] = array[right];
        array[right] = temp;
        left++;
        right--;
    }
}
```

<details>
<summary>Show solution</summary>

**Answer: `O(n)` time and `O(1)` auxiliary space.**

The function swaps the first and last elements, then moves both indices toward the middle. For `[1, 2, 3, 4, 5]`, it swaps 1 with 5 and 2 with 4. The middle element stays in place.

**Time complexity:** `O(n)`. There are `floor(n / 2)` swaps, where `floor` means rounding down. Each swap takes constant work, and dropping the factor of one-half leaves linear growth.

**Space complexity:** `O(1)`. The function changes the original array using only two indices and a temporary variable. This is called reversing the array **in place**.

</details>

[Back to Big-O Notation](README.md)
