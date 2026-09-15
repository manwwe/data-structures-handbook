# Exercises in C

### Exercise 1: Two Loops

What is the time complexity of this code?

```c
void my_func(int array[], int n) {
    int sum = 0;
    int product = 1;

    for (int i = 0; i < n; i++) {
        sum += array[i];
    }

    for (int i = 0; i < n; i++) {
        product *= array[i];
    }

    printf("Sum: %d, Product: %d\n", sum, product);
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

### Exercise 2: Nested Loops

What is the time complexity of this code?

```c
void print_pairs(int array[], int n) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            printf("Pair: array[%d] = %d, array[%d] = %d\n", i, array[i], j, array[j]);
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

### Exercise 3: A Shrinking Inner Loop

What is the time complexity of this code?

```c
void print_pairs(int array[], int n) {
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            printf("Pair: array[%d] = %d, array[%d] = %d\n", i, array[i], j, array[j]);
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

For example, if `n = 3`, the function prints `2 + 1 + 0 = 3` pairs. This counts executions of `printf`, not every loop check or increment.

**Time complexity:** `O(n²)`. Expanding the pair count gives `(n² - n) / 2`. Dropping the constant factor and lower-order term leaves quadratic growth, even though the inner loop gets shorter.

**Space complexity:** `O(1)` auxiliary space. The function prints each pair immediately without storing the pairs.

</details>

### Exercise 4: Two Arrays and a Condition

What is the time complexity of this code? The arrays can have different lengths.

```c
void print_unordered_pairs(int array_a[], int array_b[], int len_array_a, int len_array_b) {
    for (int i = 0; i < len_array_a; i++) {
        for (int j = 0; j < len_array_b; j++) {
            if (array_a[i] < array_b[j]) {
                printf("Match: array_a[%d] = %d < array_b[%d] = %d\n", i, array_a[i], j, array_b[j]);
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

[Back to Big-O Notation](../README.md)
