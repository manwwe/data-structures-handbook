# Exercises in C

### Exercise 1: Two Loops

What is the time complexity of this code?

```c
void myfunc(int array[], int n) {
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

**Answer: `O(n)`**

The first loop goes through all `n` elements to calculate the sum. The second goes through them again to calculate the product.
Because the loops run one after the other, their costs add up:

```text
O(n) + O(n) = O(2n) = O(n)
```

**Time complexity:** Going through the same array twice still takes linear time. In Big-O notation, constant factors are ignored.

**Space complexity:** O(1) auxiliary space. The function uses a fixed number of variables, regardless of the array’s size.

[Back to Big-O Notation](../README.md)
