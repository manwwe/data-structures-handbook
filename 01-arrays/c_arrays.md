# Arrays in C

Read the [array theory](README.md) first. This guide explains how to use built-in arrays in C, followed by basic exercises.

## Create, Access, and Traverse

A built-in array stores values of one type in contiguous memory. Its size cannot change after creation. No header is required to declare an array; `<stdio.h>` is used here for output.

```c
#include <stdio.h>

int main(void) {
    int scores[] = {10, 20, 30};
    int zeros[5] = {0}; /* All five elements are zero */
    scores[1] = 25;

    for (int i = 0; i < 3; ++i) {
        printf("%d\n", scores[i]);
    }
    printf("%d\n", zeros[0]);
    return 0;
}
```

Indices start at zero. For three elements, valid indices are `0`, `1`, and `2`. C does not check bounds; using an invalid index causes undefined behavior. Local arrays declared without initialization, such as `int scores[3];`, must be filled before their values are read.

## Length and Capacity

```c
int values[5] = {10, 20};
int length = 2;
const int capacity = 5;

if (length < capacity) {
    values[length] = 30;
    ++length;
}
```

The physical array has five elements, but we use only the first `length` elements as our sequence. Track this logical length yourself. To insert or delete in the middle while preserving order, shift the later elements. The exercises below show how.

## Pass an Array to a Function

An array parameter is treated as a pointer and does not carry the array's length. Pass the length separately. Use `const` when the function should only read the values:

```c
int total(const int values[], int length) {
    int result = 0;
    for (int i = 0; i < length; ++i) {
        result += values[i];
    }
    return result;
}
```

For an actual array, `sizeof(values) / sizeof(values[0])` gives its physical element count. It does not work on a pointer parameter or tell you the logical length of a partially used array. Represent an empty sequence with logical length zero, not a zero-length built-in array.

## Basic Exercises

These solutions are functions, not complete programs. Call them from `main` to try them out.

Try each exercise before opening its solution. Indices start at zero. Assume array lengths and capacities are valid and that integer calculations fit in `int`. Include `<stdio.h>` when using `printf`.

### Exercise 1: Access and Update

What does this code print? What is its time complexity?

```c
void update_score(void) {
    int scores[] = {10, 20, 30, 40};
    scores[1] = scores[3] + 5;
    printf("Updated score: %d\n", scores[1]);
}
```

<details>
<summary>Show solution</summary>

It prints `Updated score: 45`. Index `3` contains 40, and the assignment replaces the value at index `1` with 45.

**Time complexity:** `O(1)`. The code performs a fixed amount of work.

**Space complexity:** `O(1)`. The array has a fixed size of four elements.

</details>

---

### Exercise 2: Sum the Elements

Write `int sum_array(int array[], int length)` to return the sum of all elements. An empty array should produce zero.

Example: `[3, 5, 2]` should return `10`.

<details>
<summary>Show solution</summary>

```c
int sum_array(int array[], int length) {
    int sum = 0;
    for (int i = 0; i < length; i++) {
        sum += array[i];
    }
    return sum;
}
```

Start with zero and add each element. When `length` is zero, the loop never runs.

**Time complexity:** `O(n)`, where `n = length`. Each element is visited once.

**Auxiliary space:** `O(1)`. Only the sum and loop counter are needed.

</details>

---

### Exercise 3: Find the First Match

Write `int find_first(int array[], int length, int target)` to return the index of the first matching value, or `-1` if the value is absent.

Example: searching for `7` in `[4, 7, 2, 7]` should return `1`.

<details>
<summary>Show solution</summary>

```c
int find_first(int array[], int length, int target) {
    for (int i = 0; i < length; i++) {
        if (array[i] == target) {
            return i;
        }
    }
    return -1;
}
```

Go through the array from left to right. Returning immediately on a match ensures that later duplicates do not change the result.

**Time complexity:** `O(n)` in the worst case and `O(1)` when the first element matches.

**Auxiliary space:** `O(1)`.

</details>

---

### Exercise 4: Append with Spare Capacity

Write `int append_value(int array[], int length, int capacity, int value)` to append a value when space is available. Return the new length, or `-1` if the array is full.

Assume `0 <= length <= capacity` and storage for `capacity` elements. The caller must save the returned length when the operation succeeds.

Example: `[10, 20, unused]`, length `2`, capacity `3`, and value `30` should become `[10, 20, 30]` and return `3`.

<details>
<summary>Show solution</summary>

```c
int append_value(int array[], int length, int capacity, int value) {
    if (length == capacity) {
        return -1;
    }
    array[length] = value;
    return length + 1;
}
```

The next available slot is at index `length`. Checking capacity first prevents writing beyond the allocated storage. Changing the local length would not update the caller's variable, so the function returns the new length.

**Time complexity:** `O(1)`. No elements shift and no storage is allocated.

**Auxiliary space:** `O(1)`.

</details>

---

### Exercise 5: Insert at an Index

Write `int insert_at(int array[], int length, int capacity, int index, int value)` to insert a value while preserving order. Return the new length, or `-1` if the array is full or the index is invalid. Leave the array unchanged on failure.

Assume `0 <= length <= capacity` and storage for `capacity` elements. Valid insertion indices are `0` through `length`, including appending at the end.

Example: inserting `15` at index `1` in `[10, 20, 30, unused]` should produce `[10, 15, 20, 30]`.

<details>
<summary>Show solution</summary>

```c
int insert_at(int array[], int length, int capacity, int index, int value) {
    if (length == capacity || index < 0 || index > length) {
        return -1;
    }
    for (int i = length; i > index; i--) {
        array[i] = array[i - 1];
    }
    array[index] = value;
    return length + 1;
}
```

Shift from right to left so that each value is copied before its slot is overwritten. Then write the new value into the gap. The caller must save the returned length on success.

**Time complexity:** `O(n)` in the worst case. Inserting at the beginning shifts all `n` elements; inserting at the end takes `O(1)`.

**Auxiliary space:** `O(1)`.

</details>

---

### Exercise 6: Delete at an Index

Write `int delete_at(int array[], int length, int index)` to remove an element while preserving order. Return the new length, or `-1` for an invalid index. Leave the array unchanged on failure.

Example: deleting index `1` from `[10, 20, 30, 40]` should leave the logical array `[10, 30, 40]`.

<details>
<summary>Show solution</summary>

```c
int delete_at(int array[], int length, int index) {
    if (index < 0 || index >= length) {
        return -1;
    }
    for (int i = index; i < length - 1; i++) {
        array[i] = array[i + 1];
    }
    return length - 1;
}
```

Shift the later elements left. The old final slot still exists, but it is outside the new logical length. The caller must save the returned length on success.

**Time complexity:** `O(n)` in the worst case. Deleting the last element takes `O(1)`.

**Auxiliary space:** `O(1)`. Capacity does not change.

</details>

---

### Exercise 7: Reverse in Place

Write `void reverse_array(int array[], int length)` to reverse the original array without creating another array.

Example: `[1, 2, 3, 4, 5]` should become `[5, 4, 3, 2, 1]`. Empty and single-element arrays should remain unchanged.

<details>
<summary>Show solution</summary>

```c
void reverse_array(int array[], int length) {
    int left = 0;
    int right = length - 1;
    while (left < right) {
        int temp = array[left];
        array[left] = array[right];
        array[right] = temp;
        left++;
        right--;
    }
}
```

Swap the two outer elements, then move inward. Stop when the indices meet or cross.

**Time complexity:** `O(n)`. The function performs `floor(n / 2)` swaps.

**Auxiliary space:** `O(1)`. It uses two indices and one temporary value.

</details>

[Back to Arrays](README.md)
