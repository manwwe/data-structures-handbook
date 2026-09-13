# Big-O Notation

Big-O describes how the **time or space** used by an algorithm **grows** as the input size gets larger. It is a ceiling on how fast it grows in the worst-case scenario.

The most important things Big-O is not:

- **It isn't the exact running time.** It doesn't tell you how many seconds an algorithm will take.
- **It isn't the exact amount of work.** Two algorithms that grow at the same rate can still do different amounts of work.
- **It doesn't always tell you which algorithm is faster.** An algorithm that grows more slowly can still be slower for small inputs.

## Common Big-O notation

### Constant Time Complexity $O(1)$:

The runtime of the algorithm is constant and does not depend on the input size because it takes the same amount of cost no matter how large the input is.

### Logarithmic Time Complexity $O(log\ n)$:

The algorithm often halves the input size at each step because $log_2\ n$ tells us how many times the input can be halved before reaching 1. In Big-O notation, we simply write $O(log\ n)$ because the base of the logarithm does not affect how fast it grows.

### Linear Time Complexity $O(n)$:

The algorithm goes through the input a constant number of times, accessing each element as it goes. As the input size grows, the amount of work grows at the same rate. This can be the best possible time complexity when every element needs to be accessed.

### Linearithmic Time Complexity $O(n\ log\ n)$:

The algorithm processes all $n$ elements, doing about $O(log\ n)$ work for each one. This complexity is common in efficient sorting algorithms and algorithms that perform $O(log\ n)$ operations on a data structure for each input element.

### Quadratic Time Complexity $O(n^2)$:

The runtime grows roughly with the input size squared. It often contains two nested loops, but it depends on how many times each loop runs. A common example is going through every possible pair of elements in the input.

### Cubic Time Complexity $O(n^3)$:

The runtime grows roughly with the input size cubed. It often contains three nested loops, but it depends on how many times each loop runs. A common example is going through every possible triple of elements in the input.

### Exponential Time Complexity $O(2^n)$:

### Factorial Time Complexity $O(n!)$:
