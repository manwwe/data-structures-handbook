# Queues in C++

Read the [theory](README.md) first. This guide covers basic usage, then exercises with solutions.

## Use `std::queue`

Include `<queue>` for `std::queue<T>`. It provides a FIFO interface and uses `std::deque` as its default underlying container. Storage is managed automatically.

```cpp
#include <iostream>
#include <queue>

int main() {
    std::queue<int> values;
    values.push(10);
    values.push(20);
    std::cout << values.size() << "\n"; // 2
    if (!values.empty()) {
        std::cout << values.back() << "\n"; // 20
        int value = values.front();
        values.pop();
        std::cout << value << "\n"; // 10
    }
    while (!values.empty()) {
        values.pop();
    }
}
```

`push()` enqueues at the rear. `front()` reads the oldest value and `back()` reads the newest. `pop()` removes the front and returns no value; read `front()` first if you need that value. Reading either end or popping requires a nonempty queue.

For this integer queue with the default container, push, pop, front, back, size, and empty take O(1) time. There is no `clear()` member; popping all elements takes O(n). The interface provides neither indexed access nor iteration. The exercises below build a bounded circular queue to show how its storage works.

## Basic Exercises

Build a fixed-capacity circular queue one operation at a time. Try each exercise before opening its solution. Combine the following definition and the solutions in order.

```cpp
constexpr int QUEUE_CAPACITY = 5;

struct queue_t {
    int data[QUEUE_CAPACITY];
    int front;
    int size;
};
```

Maintain `0 <= front < QUEUE_CAPACITY` and `0 <= size <= QUEUE_CAPACITY`. The capacity is positive. Initialize the queue before calling other operations. All queue pointers must refer to valid objects.

For peek and dequeue, pass a pointer to a separate writable integer, such as `&value`; it must not point inside the queue. Return status is separate from the stored value, so values such as -1 remain valid data. Examples show logical order from front to rear.

These C++17 exercises implement the operations directly instead of using `std::queue`. `constexpr` makes the capacity a compile-time constant.

### Exercise 1: Initialize a Queue

Write `void queue_init(queue_t *queue)` to initialize an empty queue. Do not clear the unused array slots.

<details>
<summary>Show solution</summary>

```cpp
void queue_init(queue_t *queue) {
    queue->front = 0;
    queue->size = 0;
}
```

The size determines which slots are in use. The initial front position is zero.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No extra buffer or dynamic allocation is needed.

</details>

---

### Exercise 2: Check Whether It Is Empty

Write `int queue_is_empty(const queue_t *queue)` to return 1 if empty and 0 otherwise.

<details>
<summary>Show solution</summary>

```cpp
int queue_is_empty(const queue_t *queue) {
    return queue->size == 0;
}
```

No stored elements means size is zero, regardless of where front points.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No extra buffer or dynamic allocation is needed.

</details>

---

### Exercise 3: Check Whether It Is Full

Write `int queue_is_full(const queue_t *queue)` to return 1 if all slots are occupied and 0 otherwise.

<details>
<summary>Show solution</summary>

```cpp
int queue_is_full(const queue_t *queue) {
    return queue->size == QUEUE_CAPACITY;
}
```

A stored count distinguishes full from empty without reserving an unused slot.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No extra buffer or dynamic allocation is needed.

</details>

---

### Exercise 4: Read the Size

Write `int queue_size(const queue_t *queue)` to return the number of stored elements.

<details>
<summary>Show solution</summary>

```cpp
int queue_size(const queue_t *queue) {
    return queue->size;
}
```

Read the stored count directly; no traversal is needed.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No extra buffer or dynamic allocation is needed.

</details>

---

### Exercise 5: Enqueue a Value

Write `int queue_enqueue(queue_t *queue, int val)` to add a value at the rear. Return 1 on success or 0 if full, leaving the queue unchanged on failure. Use `queue_is_full` from Exercise 3.

Example: enqueueing 30 into `[10, 20]` produces `[10, 20, 30]`.

<details>
<summary>Show solution</summary>

```cpp
int queue_enqueue(queue_t *queue, int val) {
    if (queue_is_full(queue)) {
        return 0;
    }
    int rear = (queue->front + queue->size) % QUEUE_CAPACITY;
    queue->data[rear] = val;
    queue->size++;
    return 1;
}
```

The next insertion slot is front plus size, wrapped within capacity. Check for a full queue before writing.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No extra buffer or dynamic allocation is needed.

</details>

---

### Exercise 6: Peek at the Front

Write `int queue_peek(const queue_t *queue, int *out_val)` to read the front without removing it. Return 1 on success or 0 if empty. On failure, leave the output unchanged. Use `queue_is_empty` from Exercise 2.

<details>
<summary>Show solution</summary>

```cpp
int queue_peek(const queue_t *queue, int *out_val) {
    if (queue_is_empty(queue)) {
        return 0;
    }
    *out_val = queue->data[queue->front];
    return 1;
}
```

Read the front slot only after checking that it belongs to a nonempty queue. Neither index nor size changes.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No extra buffer or dynamic allocation is needed.

</details>

---

### Exercise 7: Dequeue a Value

Write `int queue_dequeue(queue_t *queue, int *out_val)` to remove the oldest element. Return 1 and write its value on success. Return 0 if empty, leaving both the queue and output unchanged. Use `queue_is_empty`.

Example: dequeueing `[10, 20, 30]` returns 10 and leaves `[20, 30]`.

<details>
<summary>Show solution</summary>

```cpp
int queue_dequeue(queue_t *queue, int *out_val) {
    if (queue_is_empty(queue)) {
        return 0;
    }
    *out_val = queue->data[queue->front];
    queue->front = (queue->front + 1) % QUEUE_CAPACITY;
    queue->size--;
    return 1;
}
```

Advance the front instead of shifting elements. If this empties the queue, front may remain nonzero; the next enqueue will still use the correct slot.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No extra buffer or dynamic allocation is needed.

</details>

---

### Exercise 8: Clear the Queue

Write `void queue_clear(queue_t *queue)` to remove every logical element and make the queue reusable.

<details>
<summary>Show solution</summary>

```cpp
void queue_clear(queue_t *queue) {
    queue->front = 0;
    queue->size = 0;
}
```

The array contains plain integers, so resetting the bookkeeping is enough. Old bytes remain but no longer belong to the logical queue.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No extra buffer or dynamic allocation is needed.

</details>

---

### Exercise 9: Trace Wraparound

Starting with an empty queue of capacity 5, enqueue 10, 20, 30, 40, and 50. Dequeue twice, then enqueue 60 and 70.

What are the front index, size, physical array contents, and order of the next five dequeued values? Does another enqueue succeed before removing an element?

<details>
<summary>Show solution</summary>

After two dequeues, front is 2 and size is 3. The next insertion indices are `(2 + 3) % 5 = 0` and `(2 + 4) % 5 = 1`.

```text
Physical array: [60, 70, 30, 40, 50]
Front index:   2
Size:          5
Removal order: 30, 40, 50, 60, 70
```

Another enqueue fails because the queue is full. No existing values are overwritten. Each enqueue or dequeue takes `O(1)` time and auxiliary space.

</details>

## Check the Completed Queue

Check FIFO order, empty peek and dequeue, a failed enqueue on a full queue, storing -1, wraparound after repeated operations, and reuse after clearing. Verify that failures leave state and output unchanged.

[Back to Queues](README.md)
