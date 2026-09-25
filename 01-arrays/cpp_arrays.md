# Arrays in C++

Read the [array theory](README.md) first. This guide uses C++17 and explains built-in arrays and `std::vector`, then provides exercises with solutions.

## Built-in Arrays: `int values[]`

A built-in array stores elements of the same type in contiguous memory. Its number of elements is fixed when it is created. No header is needed to declare one.

```cpp
#include <iostream>
#include <iterator> // std::size (C++17)

int main() {
    int scores[] = {10, 20, 30, 40}; // Size inferred: four integers
    int zeros[5] = {};               // Five integers initialized to zero

    scores[1] = 25;
    std::cout << scores[1] << "\n"; // 25
    std::cout << std::size(scores) << "\n"; // 4
    for (int score : scores) {
        std::cout << score << " ";
    }
    std::cout << "\n";
    std::cout << zeros[0] << "\n"; // 0
}
```

`int scores[]` declares an array; `scores[index]` accesses an element. Valid indices here are `0` through `3`. Built-in arrays do not check bounds: accessing an invalid index causes undefined behavior. A local declaration such as `int scores[4];` leaves its integers uninitialized, so assign values before reading them.

An explicitly specified built-in array size must be a positive compile-time constant in standard C++. Use a vector when the size comes from user input or needs to change.

### Logical Length and Capacity

You can use only part of a built-in array, but must track that logical length yourself:

```cpp
int values[5] = {10, 20}; // Remaining slots are initialized to zero
int length = 2;
const int capacity = 5;

if (length < capacity) {
    values[length] = 30;
    ++length;
}
```

The physical array still contains five elements; only the first three belong to our logical sequence. Inserting in the middle requires shifting values right. Deleting requires shifting left and reducing `length`. The exercises below implement these operations.

### Passing an Array to a Function

An array parameter such as `const int values[]` is treated as a pointer to its first element. It does not carry the size, so pass the length separately:

```cpp
int total(const int values[], int length) {
    int result = 0;
    for (int i = 0; i < length; ++i) {
        result += values[i];
    }
    return result;
}
```

`const` prevents this function from changing the elements. `std::size` works on the actual array in the earlier example, but not on this pointer parameter. A logical length of zero represents an empty sequence; do not declare a zero-length built-in array.

## Dynamic Arrays: `std::vector`

Include `<vector>` to use `std::vector<T>`, where `T` is the element type. A vector manages its storage and tracks its size and capacity automatically.

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> scores = {10, 20, 30};
    scores[1] = 25;
    scores.push_back(40);                   // [10, 25, 30, 40]
    scores.insert(scores.begin() + 1, 15); // [10, 15, 25, 30, 40]
    scores.erase(scores.begin() + 2);      // [10, 15, 30, 40]
    scores.pop_back();                     // [10, 15, 30]

    std::cout << scores.at(1) << "\n"; // 15
    std::cout << scores.size() << "\n"; // 3
    for (int score : scores) {
        std::cout << score << " ";
    }
    std::cout << "\n";
}
```

`begin()` returns an iterator, which identifies a position in the vector. Adding an offset identifies the position used by `insert` or `erase`. Insertion can use the end position; erasure must identify an existing element. Call `pop_back()` only when the vector is nonempty.

Like built-in arrays, `operator[]` requires a valid index and does not provide bounds checking in C++17. `at(index)` checks the index and throws `std::out_of_range` if it is invalid.

### Size, Capacity, and Growth

```cpp
std::vector<int> values; // Empty: size 0
values.reserve(10);     // Capacity at least 10; size is still 0
values.push_back(7);    // Size 1; values[0] now exists
values.resize(3);       // Size 3: [7, 0, 0]
```

`reserve` allocates room without adding elements. You cannot access reserved slots beyond `size()`. `resize` changes the number of elements; new integers are initialized to zero when no value is supplied. `std::vector<int> values(5, 9)` creates five integers, each equal to nine.

Appending takes O(1) amortized time, but an append that reallocates takes O(n). Insertion and erasure in the middle take O(n) in the worst case because later elements shift. The growth factor is implementation-dependent. Reallocation invalidates existing pointers, references, and iterators into the vector. Insertion without reallocation and erasure invalidate positions at or after the changed position; obtain positions again after modifying the vector.

### Passing a Vector to a Function

```cpp
int vector_total(const std::vector<int>& values) {
    int result = 0;
    for (int value : values) {
        result += value;
    }
    return result;
}
```

`&` passes the vector by reference, avoiding a copy. `const` makes access read-only. Use `std::vector<int>&` when a function should modify the caller's vector. The vector carries its size, so no separate length parameter is needed.

## Choosing Between Them

| Feature | Built-in array | `std::vector<int>` |
| --- | --- | --- |
| Storage | Contiguous integers | Contiguous integers |
| Size | Fixed | Can change at runtime |
| Length tracking | Track logical length separately when partially used | `size()` |
| Append, insert, delete | Manage capacity and shifts yourself | Member functions handle storage and shifts |
| Checked access | No built-in option | `at()` |
| Function arguments | Usually pointer plus length | Reference, with size available |

Use built-in arrays to learn how indexing and shifting work directly. Use a vector for a sequence whose size changes. C++ also offers `std::array` in `<array>` for fixed-size storage with container conveniences; it is separate from the built-in array syntax covered here.

## Built-in Array Exercises

Try each exercise before opening its solution. These are functions or snippets, not complete programs; call functions from `main` to run them. Include `<iostream>` for `std::cout` and `<vector>` for vector examples. Indices start at zero. Assume valid lengths and capacities unless an exercise asks you to check them, and assume sums fit in `int`.

### Exercise 1: Access and Update

What does this code print? What is its time complexity?

```cpp
void update_score(void) {
    int scores[] = {10, 20, 30, 40};
    scores[1] = scores[3] + 5;
    std::cout << "Updated score: " << scores[1] << "\n";
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

```cpp
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

```cpp
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

```cpp
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

```cpp
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

```cpp
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

```cpp
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

## Vector Exercises

### Exercise 8: Trace Size and Capacity

What are the contents and size after each operation? What can you say about capacity?

```cpp
std::vector<int> values;
values.reserve(4);
values.push_back(10);
values.push_back(20);
values.resize(3);
values.pop_back();
```

<details>
<summary>Show solution</summary>

| Operation | Contents | Size |
| --- | --- | --- |
| Construction | `[]` | 0 |
| `reserve(4)` | `[]` | 0 |
| `push_back(10)` | `[10]` | 1 |
| `push_back(20)` | `[10, 20]` | 2 |
| `resize(3)` | `[10, 20, 0]` | 3 |
| `pop_back()` | `[10, 20]` | 2 |

After `reserve(4)`, capacity is at least four and stays unchanged through these operations. Reserved space does not create elements, and removing the last element does not shrink capacity.

</details>

### Exercise 9: Keep the Even Values

Write `std::vector<int> keep_even(const std::vector<int>& values)` to return a new vector containing the even values in their original order. Leave the input unchanged.

Example: `[3, 8, 2, 7, 0]` produces `[8, 2, 0]`. Empty input produces an empty vector.

<details>
<summary>Show solution</summary>

```cpp
std::vector<int> keep_even(const std::vector<int>& values) {
    std::vector<int> result;
    for (int value : values) {
        if (value % 2 == 0) {
            result.push_back(value);
        }
    }
    return result;
}
```

Go through the input and append each even value. The const reference avoids copying or changing the input.

**Time complexity:** O(n) across the traversal and amortized appends.

**Space:** O(k) for the output, where `k` is the number of even values; O(1) auxiliary space apart from the output storage.

</details>

### Exercise 10: Erase the First Match

Write `bool erase_first(std::vector<int>& values, int target)` to remove only the first matching value, preserving order. Return `true` if a value was removed and `false` otherwise.

Example: removing `7` from `[4, 7, 2, 7]` leaves `[4, 2, 7]`. Empty input and missing targets leave the vector unchanged.

<details>
<summary>Show solution</summary>

```cpp
bool erase_first(std::vector<int>& values, int target) {
    for (auto position = values.begin(); position != values.end(); ++position) {
        if (*position == target) {
            values.erase(position);
            return true;
        }
    }
    return false;
}
```

`*position` reads the element at the iterator. Erasing shifts later elements and updates the size automatically. Return immediately after erasing so the invalidated iterator is never used again.

**Time complexity:** O(n) in the worst case for searching and shifting.

**Auxiliary space:** O(1).

</details>

[Back to Arrays](README.md)
