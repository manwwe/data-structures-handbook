# Stack Exercises in C++

These C++17 exercises build the stack operations directly on a fixed-capacity array. `constexpr` defines the capacity as a compile-time constant. The solutions can be combined in order; no standard stack container is needed to implement the operations.

Build a fixed-capacity integer stack one operation at a time. Try each exercise before opening its solution. The solutions share the following definition and use earlier helpers where stated.

```cpp
constexpr int STACK_CAPACITY = 5;

struct stack_t {
    int data[STACK_CAPACITY];
    int size;
};
```

The bottom is at index 0. When the stack is nonempty, the top is at index `size - 1`. Maintain `0 <= size <= STACK_CAPACITY`. All stack pointers must point to valid objects, initialized by Exercise 1 before other operations are called.

For peek and pop, pass a pointer to a separate writable integer, such as `&value`; it must not point inside the stack. Examples show values from bottom to top.

### Exercise 1: Initialize a Stack

Write `void stack_init(stack_t *stack)` to create an empty logical stack. It does not need to zero the unused array slots.

<details>
<summary>Show solution</summary>

```cpp
void stack_init(stack_t *stack) {
    stack->size = 0;
}
```

Only the size determines which slots are in use. Call this before using a new stack.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No additional array or dynamic allocation is needed.

</details>

---

### Exercise 2: Check Whether It Is Empty

Write `int stack_is_empty(const stack_t *stack)` to return 1 for an empty stack and 0 otherwise.

<details>
<summary>Show solution</summary>

```cpp
int stack_is_empty(const stack_t *stack) {
    return stack->size == 0;
}
```

An empty stack has no elements, so its size is zero.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No additional array or dynamic allocation is needed.

</details>

---

### Exercise 3: Check Whether It Is Full

Write `int stack_is_full(const stack_t *stack)` to return 1 when no more values fit and 0 otherwise.

<details>
<summary>Show solution</summary>

```cpp
int stack_is_full(const stack_t *stack) {
    return stack->size == STACK_CAPACITY;
}
```

This implementation has fixed capacity. A full stack must reject another push.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No additional array or dynamic allocation is needed.

</details>

---

### Exercise 4: Read the Size

Write `int stack_size(const stack_t *stack)` to return the current number of elements.

<details>
<summary>Show solution</summary>

```cpp
int stack_size(const stack_t *stack) {
    return stack->size;
}
```

Reading the stored count requires no traversal.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No additional array or dynamic allocation is needed.

</details>

---

### Exercise 5: Push a Value

Write `int stack_push(stack_t *stack, int val)` to add a value at the top. Return 1 on success or 0 if full. A failed push must leave the stack unchanged.

Example: pushing 30 onto `[10, 20]` produces `[10, 20, 30]`.

<details>
<summary>Show solution</summary>

```cpp
int stack_push(stack_t *stack, int val) {
    if (stack_is_full(stack)) {
        return 0;
    }
    stack->data[stack->size] = val;
    stack->size++;
    return 1;
}
```

Use the helper from Exercise 3. The next unused slot is at index size. Check capacity before writing and increase the count afterward.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No additional array or dynamic allocation is needed.

</details>

---

### Exercise 6: Peek at the Top

Write `int stack_peek(const stack_t *stack, int *out_val)` to read the top without removing it. Return 1 on success or 0 if empty. On failure, leave `*out_val` unchanged.

<details>
<summary>Show solution</summary>

```cpp
int stack_peek(const stack_t *stack, int *out_val) {
    if (stack_is_empty(stack)) {
        return 0;
    }
    *out_val = stack->data[stack->size - 1];
    return 1;
}
```

Use the helper from Exercise 2. The last occupied slot is size - 1. The size stays unchanged, so repeated peeks return the same value until the stack changes.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No additional array or dynamic allocation is needed.

</details>

---

### Exercise 7: Pop a Value

Write `int stack_pop(stack_t *stack, int *out_val)` to remove the top and return its value through `out_val`. Return 1 on success or 0 if empty. On failure, leave both the stack and output unchanged.

Example: popping `[10, 20, 30]` writes 30 to the output and leaves `[10, 20]`.

<details>
<summary>Show solution</summary>

```cpp
int stack_pop(stack_t *stack, int *out_val) {
    if (stack_is_empty(stack)) {
        return 0;
    }
    stack->size--;
    *out_val = stack->data[stack->size];
    return 1;
}
```

Use the helper from Exercise 2. Decreasing size makes the old top slot fall outside the logical stack. Its bytes need not be erased. A separate status allows values such as -1 to be stored normally.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No additional array or dynamic allocation is needed.

</details>

---

### Exercise 8: Clear the Stack

Write `void stack_clear(stack_t *stack)` to remove all logical elements while keeping the stack available for reuse.

<details>
<summary>Show solution</summary>

```cpp
void stack_clear(stack_t *stack) {
    stack->size = 0;
}
```

These are plain integers in an embedded array, so no nodes or separately owned values need freeing. Resetting the size makes old values inaccessible through the stack operations.

**Time complexity:** `O(1)`.

**Auxiliary space:** `O(1)`. No additional array or dynamic allocation is needed.

</details>

## Check the Completed Stack

Initialize a stack, then push 10, 20, and 30. Peek should return 30 without changing the size. Three pops should return 30, 20, and 10 in that order.

Also check an empty pop and peek, a full stack followed by a failed push, a stored value of -1, and reuse after clearing. Confirm that failed operations leave the existing state and output values unchanged.

[Back to Stacks](../README.md)
