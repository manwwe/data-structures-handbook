# Heaps

A **binary heap** keeps a minimum or maximum value easy to reach. It combines two rules: a complete binary-tree shape and an ordering rule between parents and children.

A **min-heap** has each parent less than or equal to its children. Its root is therefore a minimum value:

```text
          2
        /   \
       5     3
      / \   /
     9   7 8
```

A **max-heap** reverses the comparison: each parent is greater than or equal to its children, so the root is a maximum. Duplicate values are allowed.

Here, “heap” means a data structure. It is separate from the use of “heap” to describe dynamically allocated program memory.

## Shape and Order

A binary heap is a **complete binary tree**: every level is full except possibly the last, which fills from left to right. This keeps its height logarithmic in the number of elements.

The ordering rule applies along parent-child links. It does not sort siblings or entire subtrees relative to one another. In the example, 5 is left of 3 even though 5 is larger.

A heap is not a binary search tree. In a BST, left-subtree keys are smaller than the node and right-subtree keys are larger. A heap only guarantees the parent-child relationship and exposes an extreme value at the root.

## Storing a Heap in an Array

Store nodes in level order, from left to right:

```text
Index:  0  1  2  3  4  5
Value: [2, 5, 3, 9, 7, 8]
```

For zero-based index `i`:

```text
Parent:      (i - 1) / 2    when i > 0, using integer division
Left child:  2 * i + 1      if this index is less than size
Right child: 2 * i + 2      if this index is less than size
```

The complete shape lets us calculate these relationships without storing pointers. The root is at index 0. The active elements occupy indices 0 through `size - 1` without gaps.

These formulas assume the index calculations fit in the integer type. The fixed small capacity in the exercises keeps them within range.

## Basic Operations

The following operations use a min-heap.

### Peek at the Minimum

Read index 0 without removing it. This takes **O(1)** time. An empty heap has no minimum, so report absence before reading the array.

### Insert and Sift Up

Append the new value at the next unused slot. This preserves the complete shape but may violate the ordering rule. While the value is smaller than its parent, swap them. This is called **sifting up**.

```text
Start:        [2, 5, 3, 9, 7, 8]
Append 1:     [2, 5, 3, 9, 7, 8, 1]
Swap with 3:  [2, 5, 1, 9, 7, 8, 3]
Swap with 2:  [1, 5, 2, 9, 7, 8, 3]
```

The new value moves along one path toward the root. **Insertion takes O(log n) worst-case time** when capacity is available. If the value already fits under its parent, no swap is needed.

A fixed-capacity heap rejects insertion when full. A dynamic-array heap may grow its storage; an individual insertion that reallocates can take `O(n)` time. With geometric growth, insertion remains `O(log n)` amortized including the sifting work.

### Remove the Minimum and Sift Down

Save the root value, move the last active element to the root, and reduce size. Then repeatedly swap the replacement with its smaller child until the ordering rule holds. This is called **sifting down**.

```text
Start:                 [1, 5, 2, 9, 7, 8, 3]
Save 1, move last:     [3, 5, 2, 9, 7, 8]
Swap with smaller 2:  [2, 5, 3, 9, 7, 8]
Return:                1
```

Choosing the smaller child matters: swapping with a larger child could leave the new parent greater than its other child.

**Removing the minimum takes O(log n) worst-case time**, without resizing. Removing the only element leaves an empty heap and needs no sifting. As with insertion, an array-shrinking policy can add occasional copying costs.

### Size, Empty Check, and Clear

Keeping a stored size makes reading size and checking emptiness **O(1)** operations. For a fixed array of plain integers, clearing can simply reset size to zero. The old values may remain in memory but are no longer active heap elements.

Do not use a valid value such as -1 to mean “empty.” Return a status separately from a peeked or removed value.

## Building a Heap

We could insert each value separately, giving `O(n log n)` worst-case construction time. A faster approach is **bottom-up heap construction**, often called heapify:

1. Copy or place the values in the array.
2. Start at the last non-leaf node, index `n / 2 - 1`.
3. Sift down each node, moving backward toward the root.

Leaves already satisfy the heap rule. Processing parents from the bottom ensures their child subtrees are heaps before repairing the parent.

**Bottom-up construction takes O(n) time.** Most nodes are near the bottom and can move only a short distance. Roughly half are leaves, a quarter can move at most one level, an eighth at most two levels, and so on. Adding those costs gives linear total work.

## Searching and Changing Values

A heap does not provide fast arbitrary-key lookup. Searching for a value can take **O(n)** time because the parent-child rule does not identify a single search path.

If an element's index is already known, changing its priority and restoring order takes `O(log n)`: sift up if it becomes smaller in a min-heap, or down if it becomes larger. Finding the index first can still take `O(n)`.

Applications with frequent priority changes may maintain a separate mapping from item IDs to heap indices. Every swap must update that mapping.

## Complexity Summary

Let `n` be the active element count and `c` the array capacity. Assume constant-time comparisons and fixed-size values. Logarithmic bounds below describe nontrivial heaps; empty and one-element cases take constant work.

| Operation | Time complexity |
| --- | --- |
| Peek at minimum | O(1) |
| Insert with available capacity | O(log n) worst case |
| Remove minimum without resizing | O(log n) worst case |
| Read stored size or check empty | O(1) |
| Search for an arbitrary value | O(n) worst case |
| Change priority at a known index | O(log n) worst case |
| Build bottom-up | O(n) |
| Clear a retained array of simple values | O(1) |

Array storage takes **O(c)** space. Iterative sift-up and sift-down use **O(1) auxiliary space**. Bottom-up heapify can also work in place with constant auxiliary space; copying the input into a separate heap still needs storage for that heap.

## Heaps and Priority Queues

A **priority queue** removes the item with the most important priority, rather than the oldest or newest item. A binary heap is a common implementation.

A min-heap can return the task with the earliest deadline or the smallest tentative distance. A max-heap can return the largest score. Values with equal priority are not automatically processed in arrival order; preserving that order requires a tie-breaking field, such as an insertion sequence number.

## Heap Sort

Build a max-heap, swap its root with the last active element, shorten the active heap, and sift down. Repeating this places the largest remaining element at the end each time.

This sorts an array in ascending order in **O(n log n)** worst-case time, with **O(1) auxiliary space** when implemented iteratively in place. Standard heap sort is not stable: equal-key items may change their relative order.

Building a min-heap and repeatedly removing the minimum also produces sorted output, but collecting that output in another array uses additional storage.

## When to Use a Heap

Use a heap when you repeatedly need the next minimum or maximum without keeping all elements fully sorted. Examples include scheduling, selecting the best few candidates, and graph algorithms that repeatedly choose the smallest tentative cost.

Use a queue for arrival order, a stack for newest-first order, or an ordered search tree when sorted traversal and range queries matter. A heap's main strength is efficient access to one extreme.

## Language Guides and Practice

- [Heaps in C: examples and exercises](c_heaps.md)
- [Heaps in C++: examples and exercises](cpp_heaps.md)

[Review Arrays](../01-arrays/README.md) · [Review Trees](../06-trees/README.md) · [Review Queues](../04-queues/README.md)
