# Arrays in Python

Read the [array theory](README.md) first. Python's built-in `list` is a dynamic-array container. It stores references to objects and can grow or shrink. No import is needed.

## Create, Access, and Traverse

```python
scores = [10, 20, 30]
scores[1] = 25
print(scores[0])  # 10
print(len(scores))  # 3

for score in scores:
    print(score)
```

Indices start at zero. Negative indices count from the end: `scores[-1]` is the last element. Reading or assigning an index outside the list raises `IndexError`. An empty list is written as `[]`.

Lists can contain different types, but these examples use integers so they match the array examples in the other languages.

## Add and Remove Elements

```python
values = [10, 20, 30]
values.append(40)      # [10, 20, 30, 40]
values.insert(1, 15)   # [10, 15, 20, 30, 40]
del values[2]         # [10, 15, 30, 40]
last = values.pop()   # last is 40; values is [10, 15, 30]
```

`append` adds one element at the end. `insert` makes room at a position. `del` removes by index, and `pop` removes and returns an element (the last one by default). Call `pop()` only on a nonempty list. `remove(value)` deletes the first matching value and raises `ValueError` if it is absent.

`len(values)` gives the current length. Python manages capacity internally; there is no public list method to reserve capacity. Index access takes O(1), appending takes O(1) amortized time, and inserting or deleting near the beginning takes O(n) because later references shift.

## Pass a List to a Function

```python
def add_score(scores, score):
    scores.append(score)

values = [10, 20]
add_score(values, 30)
print(values)  # [10, 20, 30]
```

The function receives a reference to the same list, so changing its elements changes the caller's list. `other = values` also shares the list. Use `other = values.copy()` for a separate shallow copy: the list is new, but any nested objects are still shared.

## Basic Exercises

Try each exercise before opening its solution.

### Exercise 1: Access and Update

What does this print?

```python
scores = [10, 20, 30, 40]
scores[1] = scores[-1] + 5
print(scores[1])
```

<details>
<summary>Show solution</summary>

It prints `45`. The last value is `40`, and the assignment replaces index `1` with `45`. This takes O(1) time.

</details>

### Exercise 2: Go Through the Values

Write `sum_values(values)` using a loop. Return zero for an empty list. For `[3, 5, 2]`, return `10`.

<details>
<summary>Show solution</summary>

```python
def sum_values(values):
    total = 0
    for value in values:
        total += value
    return total
```

The loop visits every value: O(n) time and O(1) auxiliary space, assuming constant-size integer arithmetic. Python also provides the built-in `sum(values)` for this operation.

</details>

### Exercise 3: Add and Remove

Start with `[10, 30]`. Insert `20` between the two values, append `40`, and then remove the first element by index. What remains?

<details>
<summary>Show solution</summary>

```python
values = [10, 30]
values.insert(1, 20)
values.append(40)
del values[0]
print(values)  # [20, 30, 40]
```

Insertion and deletion shift later elements, taking O(n) in general. Appending takes O(1) amortized time.

</details>

[Back to Arrays](README.md)
