# Arrays

An **array** stores a sequence of elements in order. Each element has a numbered position called an **index**, which lets us access it directly.

For example, an array could store four scores:

```text
Index:    0    1    2    3
Value:  [10,  20,  30,  40]
```

In languages such as C, C++, and Python, indexing starts at zero. The first element is at index `0`, and the last element of an array with `n` elements is at index `n - 1`.

The order is the order of the elements in the sequence; an array does not have to be sorted.

## How Arrays Are Stored

An array stores its slots next to one another in memory. This is called **contiguous storage**. Each slot has the same size, so the position of an element can be calculated directly:

```text
Element address = starting address + index × slot size
```

Suppose the array starts at address 1000 and each slot takes 4 bytes:

```text
Index:       0       1       2       3
Address:   1000    1004    1008    1012
Value:      10      20      30      40
```

To access index `2`, we calculate `1000 + 2 × 4 = 1008`. We do not need to go through the earlier elements. This is why accessing an element by index takes **O(1)** time.

The address and slot size above are examples. In a C array, the slots hold values of the same type. Some array-based containers, such as Python lists, store references to objects instead; the objects themselves do not have to sit next to each other.

## Length and Capacity

**Length** is the number of elements currently in use. **Capacity** is the number of elements the allocated storage can hold.

```text
Storage:  [10, 20, 30, unused, unused]
Length:   3
Capacity: 5
```

The unused slots are available space, not part of the logical sequence. They do not necessarily contain zero or a special empty value.

When using a C array this way, the program must keep track of the logical length separately. The built-in array does not manage it automatically.

## Basic Operations

The examples below preserve the order of the elements. Let `n` be the current length, and assume each element takes constant time to read, write, or compare.

### Access an Element

Use an index to read a value:

```text
Array:             [10, 20, 30, 40]
Read index 2:      30
Array afterward:   [10, 20, 30, 40]
```

**Time complexity: O(1).** The index tells us where to find the element directly.

The index must be valid. For a nonempty array of length `n`, the element indices range from `0` through `n - 1`.

### Update an Element

Find the slot by index and replace its value:

```text
Before:            [10, 20, 30, 40]
Set index 1 to 25
After:             [10, 25, 30, 40]
```

**Time complexity: O(1).** Updating one slot does not require shifting the other elements.

### Traverse an Array

Traversal means going through the elements, usually one at a time:

```text
Array:  [10, 20, 30, 40]
Visit:   10 → 20 → 30 → 40
```

**Time complexity: O(n).** Visiting every element takes work proportional to the length, assuming constant work per visit.

### Search for a Value

If the array is unsorted, check elements until the value is found or the array ends:

```text
Array:       [30, 10, 40, 20]
Find 40:     check 30 → check 10 → check 40
Result:      index 2
```

**Time complexity: O(n) in the worst case.** The value might be last or absent. The best case is **O(1)** when the first element matches.

Searching for a value is different from accessing a known index. An index tells us where to look; a value does not.

For a sorted array, **binary search** can find a value in **O(log n)** time by repeatedly cutting the search range in half. Keeping the array sorted can make insertion more expensive because elements may need to shift.

### Insert an Element

To insert a value at an index, make room by shifting the following elements one position to the right. Start shifting from the end so existing values are not overwritten.

```text
Before:                [10, 20, 30, unused]
Insert 15 at index 1
Shift 30 right:        [10, 20, 30, 30]
Shift 20 right:        [10, 20, 20, 30]
Write 15:              [10, 15, 20, 30]
Length:                3 → 4
```

**Time complexity: O(n) in the worst case.** Inserting at the beginning shifts all existing elements. Inserting at index `i` shifts `n - i` elements.

Adding an element at the end is called **appending**. It takes **O(1)** time when there is spare capacity because nothing needs to shift.

If the storage is full, a fixed-size array cannot grow. The program must reject the insertion or move the elements into larger storage. A dynamic array manages growth for us.

### Delete an Element

To delete a value at an index while preserving order, shift the later elements one position to the left, then reduce the length:

```text
Before:                [10, 20, 30, 40]
Delete index 1
Shift 30 left:         [10, 30, 30, 40]
Shift 40 left:         [10, 30, 40, 40]
Reduce length:         [10, 30, 40, unused]
Length:                4 → 3
```

**Time complexity: O(n) in the worst case.** Deleting the first element shifts the rest. Deleting the last element takes **O(1)** if the storage is not resized.

Reducing the logical length does not necessarily erase the old last slot or reduce the allocated capacity.

If order does not matter, a different approach is to replace the element being deleted with the last element and reduce the length. Deleting by a known index this way takes **O(1)** without resizing, but changes the order.

## Fixed-Size and Dynamic Arrays

A **fixed-size array** has a size that cannot change after it is created. A built-in C array is an example. We can change its values, but adding more elements than its storage holds requires separate, larger storage.

A **dynamic array** manages a backing array and can allocate a larger one when needed. C++ `std::vector` and Python `list` are examples of dynamic-array containers.

A typical growth step looks like this:

```text
Before:      [10, 20, 30, 40]                    length 4, capacity 4
Append 50
Allocate:    [unused, unused, unused, unused, unused, unused, unused, unused]
Copy/write:  [10, 20, 30, 40, 50, unused, unused, unused]
After:                                          length 5, capacity 8
```

Doubling the capacity is one possible growth strategy, not a rule for every implementation.

An append that reallocates and copies `n` elements takes **O(n)** time. However, growing capacity by a fixed factor greater than one leaves room for many later appends. Across a long sequence of appends, the total copying work grows linearly with the number of elements added.

This gives **O(1) amortized time per append**. “Amortized” means spreading the total cost across a sequence of operations. It does not mean that every append takes constant time or that we are assuming random inputs.

## Complexity Summary

These costs assume constant-size elements, valid indices, and order-preserving operations unless stated otherwise.

| Operation | Time complexity | Reason |
| --- | --- | --- |
| Access by index | O(1) | Calculate the slot directly |
| Update by index | O(1) | Replace one value |
| Traverse all elements | O(n) | Visit every element |
| Search an unsorted array | O(n) worst case | May check every element |
| Binary search a sorted array | O(log n) worst case | Halve the search range |
| Append with spare capacity | O(1) | Write to the next unused slot |
| Append to a dynamic array | O(1) amortized; O(n) worst case | Some appends require growth and copying |
| Insert at an arbitrary index | O(n) worst case | Shift later elements, possibly grow storage |
| Delete at an arbitrary index | O(n) worst case | Shift later elements |
| Delete the last element without resizing | O(1) | Reduce the length |

An array with capacity `c` uses **O(c)** storage. When capacity stays proportional to length, this is **O(n)** storage. Spare slots still use memory.

Access, update, traversal, and shifting can use **O(1) auxiliary space**. Resizing by allocating and copying to a new buffer temporarily needs **O(n) additional storage** under the usual proportional-growth strategy.

## When to Use Arrays

Arrays are useful when you frequently access elements by index or go through them in order. Examples include a row of image pixels, a list of daily temperatures, or a collection of scores.

Their contiguous storage often helps traversal run efficiently because nearby elements are stored together.

The main tradeoff is changing the sequence near the beginning or middle: preserving order requires shifting elements. Searching an unsorted array can also require checking every value.

Choose an array when its access pattern fits the work. If frequent insertion, deletion, or lookup by a key dominates the task, other data structures may be a better fit.

## Practice Exercises

- [Exercises in C](exercises/c.md)
- [Exercises in C++](exercises/cpp.md)
- [Exercises in Python](exercises/python.md)

[Review Big-O Notation](../00-complexity-analysis/README.md)
