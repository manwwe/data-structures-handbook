# Queues

A **queue** stores elements in **first in, first out (FIFO)** order. The first element added is the first one removed, like people waiting in line.

Elements enter at the **rear** and leave at the **front**.

```text
Remove here                           Add here
    ↓                                     ↓
front → [10] → [20] → [30] ← rear
```

If we add 10, 20, and 30 in that order, removing them returns 10, then 20, then 30. Their values do not determine the order; their arrival does.

A queue describes behavior, not a specific storage layout. Arrays and linked lists can both implement it.

## Basic Operations

### Enqueue

**Enqueue** adds an element at the rear.

```text
Before:  [10, 20]       front = 10
Enqueue 30
After:   [10, 20, 30]   front = 10
```

A bounded queue rejects an enqueue when full. A growing queue may need memory allocation, which can fail. A failed operation should leave existing elements unchanged.

### Dequeue

**Dequeue** removes the front element. In this chapter it also returns the removed value; some interfaces require reading the front separately before removal.

```text
Before:  [10, 20, 30]
Dequeue: returns 10
After:   [20, 30]
```

An empty queue has no element to remove. Report failure before trying to read its storage.

### Peek

**Peek**, or **front**, reads the next value to be removed without removing it.

```text
Before:  [20, 30]
Peek:    returns 20
After:   [20, 30]
```

Peek also needs to handle an empty queue.

### Check Empty, Read Size, and Clear

**is_empty** checks whether the queue has zero elements. Keeping a stored count makes reading the **size** an `O(1)` operation.

**Clear** removes all logical elements. For a fixed array of integers, resetting the bookkeeping is enough. A linked queue must also release its allocated nodes.

## Following a Sequence

| Operation | Returned value | Queue afterward, front to rear |
| --- | --- | --- |
| Start | — | `[]` |
| Enqueue 10 | — | `[10]` |
| Enqueue 20 | — | `[10, 20]` |
| Peek | 10 | `[10, 20]` |
| Dequeue | 10 | `[20]` |
| Enqueue 30 | — | `[20, 30]` |
| Dequeue | 20 | `[30]` |
| Dequeue | 30 | `[]` |
| Dequeue | Failure: empty | `[]` |

A stack would remove the newest element first. A queue removes the oldest pending element first.

## Implementing a Queue with an Array

One approach removes index 0 and shifts every later element left. That preserves order, but each dequeue can take `O(n)` time.

A **circular array**, also called a **ring buffer**, avoids shifting. It keeps track of where the front is and reuses freed slots by wrapping around to the start of the array.

### Front, Size, and Capacity

Use three pieces of information:

- **front:** The index of the next element to remove when nonempty.
- **size:** The number of stored elements.
- **capacity:** The number of allocated slots, which must be positive.

The next insertion index is calculated as:

```text
rear insertion index = (front + size) % capacity
```

The `%` operator gives the remainder. For capacity 5, index 5 wraps to 0, and index 6 wraps to 1.

This chapter uses the rear position to mean the **next insertion slot**. Other implementations store the last occupied index instead; either convention works if used consistently.

### A Wraparound Example

Start with a full array, then dequeue 10 and 20:

```text
Index:    0       1       2    3    4
Storage: [unused, unused, 30,  40,  50]
Front:   2
Size:    3
```

Enqueue 60 at `(2 + 3) % 5 = 0`, then enqueue 70 at `(2 + 4) % 5 = 1`:

```text
Index:    0   1   2   3   4
Storage: [60, 70, 30, 40, 50]
Front:   2
Size:    5

Logical order: 30 → 40 → 50 → 60 → 70
```

The logical order differs from reading physical slots from left to right. No existing elements moved.

To dequeue, read the front slot, set `front = (front + 1) % capacity`, and decrease size. To enqueue, check space, write at the insertion index, and increase size.

### Empty and Full

With a stored count, the states are unambiguous:

```text
Empty: size == 0
Full:  size == capacity
```

The insertion index can equal the front index in either state, so indices alone would not distinguish them under this convention. The count allows us to use every allocated slot.

Unused slots may still contain old values. Size determines whether those values belong to the queue.

### Growing a Circular Array

A dynamic queue can allocate larger storage when full. Copy the elements in **logical order**, starting at the front and wrapping as needed, then set the new front to zero.

Geometric growth, such as doubling capacity, gives `O(1)` amortized enqueue time. One enqueue that copies `n` elements takes `O(n)`. Amortized means spreading the total cost across a sequence of operations; it does not make every individual enqueue constant time.

## Implementing a Queue with a Linked List

Maintain a head pointer for the front and a tail pointer for the rear:

```text
head → [10 | next] → [20 | next] → [30 | NULL]
                                     ↑
                                    tail
```

To enqueue, allocate a node whose next pointer is null, link it after the tail, and update the tail. In an empty queue, the new node becomes both head and tail.

To dequeue, save the head value, move head to its successor, and release the removed node. If that removes the final node, set tail to null too. Otherwise, tail would point to released memory.

Both operations take `O(1)` under the usual assumption that allocating or releasing one node takes constant time. Actual allocator costs can vary. Without a tail pointer, finding the insertion point takes `O(n)`.

A linked queue grows one node at a time, but memory allocation can still fail. Check allocation before changing existing links. Release removed nodes when using manual memory management; containers with automatic ownership handle this cleanup for you.

## Complexity Summary

Let `n` be the number of elements and `c` the array capacity. Assume constant-size values, a stored count, a linked queue with head and tail, and no shrinking on dequeue.

| Operation | Fixed circular array | Growing circular array | Linked list |
| --- | --- | --- | --- |
| Enqueue | O(1), fails if full | O(1) amortized; O(n) worst case | O(1), subject to allocation |
| Dequeue | O(1) | O(1) | O(1) |
| Peek | O(1) | O(1) | O(1) |
| Check empty | O(1) | O(1) | O(1) |
| Read stored size | O(1) | O(1) | O(1) |
| Clear | O(1) for simple values | O(1) for simple values when retaining storage | O(n) to release nodes |
| Storage | O(c) | O(c) | O(n) |

A circular array reserves all its capacity, even when mostly empty. A linked queue needs a link and possible allocation overhead per node.

Ordinary enqueue and dequeue use `O(1)` auxiliary space beyond storage for a newly added element. Growth by allocating and copying temporarily needs extra space proportional to the old capacity. Values that own resources may require additional cleanup on removal.

## Reporting Failure

Do not use a valid queue value such as -1 alone to mean “empty.” A queue may legitimately store that value.

An interface can return a success flag and write the dequeued or peeked value through an output parameter. On failure, leave the queue and output unchanged. Other designs use optional values or exceptions.

## When to Use a Queue

- **Waiting work:** Process pending jobs in arrival order.
- **Breadth-first search:** Visit a graph or tree one level at a time.
- **Buffers:** Hold data until a consumer is ready to process it.
- **Simulations:** Represent arrivals waiting for service.

FIFO describes removal order, not necessarily completion order when multiple workers run concurrently. A basic queue implementation also needs additional coordination before being shared safely between threads.

Use a stack when the newest item should be processed first. Use a priority queue when urgency, rather than arrival order, should decide what comes next.

## Language Guides and Practice

- [Queues in C: usage and exercises](c_queues.md)
- [Queues in C++: usage and exercises](cpp_queues.md)

[Review Arrays](../01-arrays/README.md) · [Review Linked Lists](../02-linked-lists/README.md) · [Review Stacks](../03-stacks/README.md)
