# Heap Exercises in C

Build a fixed-capacity binary min-heap of integers. Try each exercise before opening its solution. Combine the shared definition and solutions in order; helper dependencies are named in each prompt.

```c
#define HEAP_CAPACITY 16

typedef struct heap {
    int data[HEAP_CAPACITY];
    int size;
} heap_t;
```

All heap pointers must refer to valid objects initialized before other operations. Maintain `0 <= size <= HEAP_CAPACITY` and the min-heap rule for all public operations. Duplicates and negative values are valid. For output parameters, pass a separate writable integer outside the heap.

Let `n` be the active element count. Complexity bounds describe how work grows with that count when considering heaps of increasing capacity; this example uses capacity 16. Empty and one-element operations take constant time. The small capacity also keeps child-index calculations within int range.

### Exercise 1: Initialize a Heap

Write `void heap_init(heap_t *heap)` to initialize an empty heap. Unused array slots do not need to be zeroed.

<details>
<summary>Show solution</summary>

```c
void heap_init(heap_t *heap) {
    heap->size = 0;
}
```

Only active slots belong to the heap.

**Time complexity:** O(1).

**Auxiliary space:** `O(1)`. The algorithms are iterative and do not allocate an extra buffer. The heap's embedded array is its storage.

</details>

---

### Exercise 2: Read Size and Check Capacity

Write `heap_size`, `heap_is_empty`, and `heap_is_full`. The checks return 1 for true and 0 for false.

<details>
<summary>Show solution</summary>

```c
int heap_size(const heap_t *heap) {
    return heap->size;
}

int heap_is_empty(const heap_t *heap) {
    return heap->size == 0;
}

int heap_is_full(const heap_t *heap) {
    return heap->size == HEAP_CAPACITY;
}
```

Read the stored count; no traversal is necessary.

**Time complexity:** O(1) per operation.

**Auxiliary space:** `O(1)`. The algorithms are iterative and do not allocate an extra buffer. The heap's embedded array is its storage.

</details>

---

### Exercise 3: Sift Up

Write `void sift_up(heap_t *heap, int index)`. Assume index is valid and only the value at that index may be too small for its parent, as after appending a new value.

<details>
<summary>Show solution</summary>

```c
void sift_up(heap_t *heap, int index) {
    while (index > 0) {
        int parent = (index - 1) / 2;
        if (heap->data[parent] <= heap->data[index]) break;
        int temp = heap->data[parent];
        heap->data[parent] = heap->data[index];
        heap->data[index] = temp;
        index = parent;
    }
}
```

Check index > 0 before calculating a parent. Each swap moves one level closer to the root. Equal values need no swap.

**Time complexity:** O(log n) worst case.

**Auxiliary space:** `O(1)`. The algorithms are iterative and do not allocate an extra buffer. The heap's embedded array is its storage.

</details>

---

### Exercise 4: Insert a Value

Write `int heap_insert(heap_t *heap, int val)` using `heap_is_full` and `sift_up`. Return 1 on success or 0 if full, leaving the heap unchanged on failure.

<details>
<summary>Show solution</summary>

```c
int heap_insert(heap_t *heap, int val) {
    if (heap_is_full(heap)) return 0;
    int index = heap->size;
    heap->data[index] = val;
    heap->size++;
    sift_up(heap, index);
    return 1;
}
```

Appending preserves the complete shape. Sifting restores the parent-child ordering.

**Time complexity:** O(log n) worst case; O(1) for a full-heap rejection.

**Auxiliary space:** `O(1)`. The algorithms are iterative and do not allocate an extra buffer. The heap's embedded array is its storage.

</details>

---

### Exercise 5: Peek at the Minimum

Write `int heap_peek(const heap_t *heap, int *out_val)`. Return 1 and write the minimum on success, or 0 on an empty heap without changing the output. Use `heap_is_empty`.

<details>
<summary>Show solution</summary>

```c
int heap_peek(const heap_t *heap, int *out_val) {
    if (heap_is_empty(heap)) return 0;
    *out_val = heap->data[0];
    return 1;
}
```

The min-heap rule puts a minimum at the root. Peek does not remove it.

**Time complexity:** O(1).

**Auxiliary space:** `O(1)`. The algorithms are iterative and do not allocate an extra buffer. The heap's embedded array is its storage.

</details>

---

### Exercise 6: Sift Down

Write `void sift_down(heap_t *heap, int index)`. Assume index is valid, its child subtrees are heaps, and only this node may be too large for its children.

<details>
<summary>Show solution</summary>

```c
void sift_down(heap_t *heap, int index) {
    while (1) {
        int left = 2 * index + 1;
        if (left >= heap->size) break;
        int right = left + 1;
        int smaller = left;
        if (right < heap->size && heap->data[right] < heap->data[left]) {
            smaller = right;
        }
        if (heap->data[index] <= heap->data[smaller]) break;
        int temp = heap->data[index];
        heap->data[index] = heap->data[smaller];
        heap->data[smaller] = temp;
        index = smaller;
    }
}
```

Check child indices before accessing them. Swap with the smaller existing child, then continue down that path. The final parent may have only a left child.

**Time complexity:** O(log n) worst case.

**Auxiliary space:** `O(1)`. The algorithms are iterative and do not allocate an extra buffer. The heap's embedded array is its storage.

</details>

---

### Exercise 7: Remove the Minimum

Write `int heap_pop(heap_t *heap, int *out_val)` using `heap_is_empty` and `sift_down`. Return 1 and the removed minimum on success, or 0 if empty, leaving state and output unchanged on failure.

<details>
<summary>Show solution</summary>

```c
int heap_pop(heap_t *heap, int *out_val) {
    if (heap_is_empty(heap)) return 0;
    *out_val = heap->data[0];
    heap->size--;
    if (heap->size > 0) {
        heap->data[0] = heap->data[heap->size];
        sift_down(heap, 0);
    }
    return 1;
}
```

Replace the root with the last active value so the array has no gaps. Removing the only element needs no replacement or sifting.

**Time complexity:** O(log n) worst case.

**Auxiliary space:** `O(1)`. The algorithms are iterative and do not allocate an extra buffer. The heap's embedded array is its storage.

</details>

---

### Exercise 8: Build from an Array

Write `int heap_build(heap_t *heap, const int values[], int length)` using `sift_down`. Replace the heap with the given values and build it bottom-up. Return 0 for a negative length or a length above capacity without modifying the heap; otherwise return 1. Assume values points to at least length readable integers and does not overlap the heap. For length zero it may be an empty pointer.

<details>
<summary>Show solution</summary>

```c
int heap_build(heap_t *heap, const int values[], int length) {
    if (length < 0 || length > HEAP_CAPACITY) return 0;
    for (int i = 0; i < length; i++) heap->data[i] = values[i];
    heap->size = length;
    for (int i = length / 2 - 1; i >= 0; i--) {
        sift_down(heap, i);
    }
    return 1;
}
```

Start at the last non-leaf and work back toward the root. A signed loop counter lets the loop stop at -1, including when length is zero. Most nodes move only a short distance, giving linear total work.

**Time complexity:** O(n), where n is the supplied length.

**Auxiliary space:** `O(1)`. The algorithms are iterative and do not allocate an extra buffer. The heap's embedded array is its storage.

</details>

---

### Exercise 9: Clear the Heap

Write `void heap_clear(heap_t *heap)` to remove all logical elements while keeping the heap reusable.

<details>
<summary>Show solution</summary>

```c
void heap_clear(heap_t *heap) {
    heap->size = 0;
}
```

Plain integers in an embedded array do not need resource cleanup. Reset size without erasing unused slots.

**Time complexity:** O(1).

**Auxiliary space:** `O(1)`. The algorithms are iterative and do not allocate an extra buffer. The heap's embedded array is its storage.

</details>

## Check the Completed Heap

Insert 5, 2, 8, 1, and 3. Peek should return 1 without changing size. Repeated pop calls should return 1, 2, 3, 5, and 8.

Check empty peek/pop, full insertion failure, duplicate and negative values, a node with only a left child, and reuse after clearing. Build from sorted, reverse-sorted, empty, and single-element arrays. After each mutation, verify that every parent is at most its children; the entire array need not be sorted.

[Back to Heaps](../README.md)
