# Complexity Analysis in Python

Read the [theory](README.md) first, then work through the language examples and exercises.

## Analyze Code Step by Step

Identify the input size, count how often each operation runs, and separate time from auxiliary space. Consecutive loops add their costs; nested loops require counting the total inner work. State whether you are analyzing the best or worst case. Count recursive calls and the maximum number of simultaneously active calls separately.

In Python, `len(a_list)` takes O(1), while copying or slicing an entire list takes O(n). A loop over a list visits every element unless it exits early. Use these operation costs to analyze the examples below.

## Basic Exercises

### Exercise 1: One Pass

What are the time and auxiliary-space costs? Assume fixed-size integer arithmetic.

```python
def total(values):
    result = 0
    for value in values:
        result += value
    return result
```

<details>
<summary>Show solution</summary>

O(n) time for `n` values and O(1) auxiliary space. The empty list returns zero.

</details>

### Exercise 2: Copy a List

What are the costs of this function?

```python
def copy_values(values):
    return values.copy()
```

<details>
<summary>Show solution</summary>

O(n) time and O(n) output storage. This is a shallow copy: nested objects are shared. One line of code can still do linear work.

</details>

### Exercise 3: Nested Loops

How many times does the counter increase?

```python
def count_pairs(values):
    count = 0
    for _ in values:
        for _ in values:
            count += 1
    return count
```

<details>
<summary>Show solution</summary>

For a list of length `n`, the counter increases `n²` times. Time is O(n²), with O(1) auxiliary space under the fixed-size arithmetic model. Python integers can grow, so that assumption matters for very large values.

</details>

[Back to Big-O Notation](README.md)
