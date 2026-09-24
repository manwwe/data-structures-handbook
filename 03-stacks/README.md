# Stacks

A **stack** stores elements in **last in, first out (LIFO)** order. The most recently added element is the first one removed.

Think of a stack of plates: you add a plate on top and take the top plate off first. The end where these operations happen is called the **top**.

```text
Push 10, then 20, then 30:

         ┌────┐
top →    │ 30 │  ← removed first
         ├────┤
         │ 20 │
         ├────┤
         │ 10 │  ← removed last
         └────┘
```

A stack describes a set of operations and their behavior, not one particular memory layout. It can be built using an array or a linked list.

## Basic Operations

### Push

**Push** adds a new element at the top.

```text
Before:  [10, 20]       top = 20
Push 30
After:   [10, 20, 30]   top = 30
```

In these horizontal examples, the bottom is on the left and the top is on the right.

A bounded stack must check for available space before accepting another element. A growing stack may need to allocate memory, which can fail. A failed push should leave the existing elements unchanged.

### Pop

**Pop** removes the top element and returns its value.

```text
Before:  [10, 20, 30]
Pop:     returns 30
After:   [10, 20]
```

Popping from an empty stack has no valid value to return. The operation must report that failure instead of reading an invalid element.

### Peek

**Peek**, sometimes called **top**, reads the top value without removing it.

```text
Before:  [10, 20]
Peek:    returns 20
After:   [10, 20]
```

Like pop, peek must handle an empty stack.

### Check Whether the Stack Is Empty

**is_empty** tells us whether there are any elements to read or remove.

```text
[]        → true
[10, 20]  → false
```

A stack can also expose its **size**, the number of stored elements. Keeping a count makes reading the size an `O(1)` operation.

### Clear

**Clear** removes all elements, leaving an empty stack that can be reused.

For an array of simple values, resetting the size is enough to make the stack logically empty. The old bytes may remain in storage. For a linked stack with dynamically allocated nodes, clearing must also free every node.

## Following a Sequence of Operations

| Operation | Returned value | Stack afterward, bottom to top |
| --- | --- | --- |
| Start | — | `[]` |
| Push 10 | — | `[10]` |
| Push 20 | — | `[10, 20]` |
| Peek | 20 | `[10, 20]` |
| Pop | 20 | `[10]` |
| Push 30 | — | `[10, 30]` |
| Pop | 30 | `[10]` |
| Pop | 10 | `[]` |
| Pop | Failure: empty | `[]` |

The order depends on when values were pushed, not on their numerical size. A stack does not sort its contents.

## Implementing a Stack with an Array

Keep an array and a count of the elements in use. If the count is `size`, the top element is at index `size - 1`, provided the stack is not empty.

```text
Index:     0    1    2       3
Storage:  [10,  20,  unused, unused]
Size:      2
Capacity:  4
Top:       index 1
```

To push 30, write it at index `size`, then increase `size`. To pop, decrease `size` and read the value at that index. Both operations happen at the end, so no elements need to shift.

### Fixed Capacity

A fixed-capacity stack reserves a set number of slots. Push takes `O(1)` time but fails when `size == capacity`.

Trying to push into a full bounded stack is called **overflow**. Trying to pop from an empty stack is called **underflow**. These are conditions to handle, not reasons to access memory outside the array.

### Dynamic Capacity

A dynamic-array stack can allocate larger storage when full and copy the existing elements into it.

With geometric growth, such as doubling capacity, push takes **O(1) amortized time** over a sequence of pushes. A single push that copies `n` elements can take **O(n)**.

Pop takes `O(1)` when it does not resize the array. An implementation that shrinks storage may occasionally copy elements during a pop, so its cost depends on the resizing policy.

## Implementing a Stack with a Linked List

Use the head of a singly linked list as the top:

```text
top → [30 | next] → [20 | next] → [10 | NULL]
```

To push, allocate a node, link it to the current top, and make it the new top. To pop, save the top value, move top to the next node, and release the removed node.

Both operations change only a few links. They take `O(1)` under the usual model that treats allocation and release of one node as constant time; actual allocator costs can vary.

Using the head is important. Removing the tail of a singly linked list would require finding its predecessor, which takes `O(n)` time in the worst case.

A linked stack has no fixed array capacity, but it can still run out of memory. A push must check allocation success before updating the top.

## Complexity Summary

Let `n` be the number of elements and `c` the allocated array capacity. Assume constant-size values, a stored size, and a dynamic array that grows geometrically and does not shrink during pop.

| Operation | Fixed array | Dynamic array | Linked list |
| --- | --- | --- | --- |
| Push | O(1), fails if full | O(1) amortized; O(n) worst case | O(1), subject to allocation |
| Pop | O(1) | O(1) | O(1) |
| Peek | O(1) | O(1) | O(1) |
| Check empty | O(1) | O(1) | O(1) |
| Read stored size | O(1) | O(1) | O(1) |
| Clear | O(1) for simple values | O(1) if retaining storage and using simple values | O(n) to free nodes |
| Storage | O(c) | O(c) | O(n) |

An array that retains capacity after many pops can use much more space than its current length suggests. A linked stack needs a pointer and possible allocation overhead for each node.

Ordinary push, pop, and peek use `O(1)` auxiliary space, apart from storage for a newly pushed element. Array growth by allocating and copying temporarily needs additional storage proportional to the old capacity. Values that own other resources may require extra cleanup when removed.

## Reporting Failure

An integer stack can contain any integer, including `-1` or zero. Using one of those values alone to mean “empty” makes the result ambiguous.

One C approach is to return a success flag and write the value through an output pointer:

```text
Pop succeeds:  return success and store the removed value
Pop fails:     return failure and leave the output unchanged
```

Other interfaces may use exceptions or optional values. The important distinction is whether an operation succeeded, separately from the value it returned.

## When to Use a Stack

- **Undo history:** The most recent action is the first one undone. Supporting redo requires additional bookkeeping, often a second stack.
- **Function calls:** A call stack keeps track of active calls so a function can return to its caller. Deep recursion can exhaust this storage.
- **Matching parentheses:** Push opening symbols and match each closing symbol with the most recent unmatched opening symbol.
- **Backtracking:** Save choices so the most recent one can be taken back first.

Use a stack when processing the newest pending item first matches the problem. If the oldest item should be processed first, a queue is a better fit. Frequent access to arbitrary positions is outside the basic stack interface.

[Review Arrays](../01-arrays/README.md) · [Review Linked Lists](../02-linked-lists/README.md)
