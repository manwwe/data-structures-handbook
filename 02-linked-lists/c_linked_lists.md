# Linked Lists in C

Read the [theory](README.md) first. This guide covers basic usage, then exercises with solutions.

## Represent and Use Nodes

C has no built-in linked-list container. Define a structure containing a value and a pointer to the next node. Use `NULL` for the end of the list. Here is a complete example using local nodes:

```c
#include <stdio.h>

int main(void) {
    struct node {
        int value;
        struct node *next;
    };
    struct node second = {20, NULL};
    struct node first = {10, &second};
    struct node *head = &first;
    head->value = 15;
    for (const struct node *current = head;
         current != NULL; current = current->next) {
        printf("%d\n", current->value); // 15, then 20
    }
    head = head->next; // Unlink the first node
    printf("%d\n", head->value); // 20
    return 0;
}
```

`&` gets an object's address, and `->` accesses a member through a pointer. These local nodes remain alive until `main` returns; do not call `free` on them. The exercises instead allocate nodes with `malloc` from `<stdlib.h>` so nodes can outlive the function that creates them. Check allocation for failure and release each allocated node exactly once with `free`.

A function that changes the caller's head receives `node_t **head`: pass `&head` so the function can replace that pointer. When freeing a list, save the next link before freeing the current node. Never use a pointer to a freed node.

## Basic Exercises

Try each exercise before opening its solution. Use this node definition throughout:

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct node {
    int val;
    struct node *next;
} node_t;
```

Use a singly linked list with no cycles, stored tail, or stored length. An empty list has `node_t *head = NULL`. All nodes removed by these functions must have been dynamically allocated, for example with `create_node` from Exercise 1.

For functions receiving `node_t **head`, pass the address of a valid head variable, such as `&head`; the list itself may be empty. Other node pointers must refer to live nodes or be `NULL`. After deleting a node, do not use any saved pointer to it.

The solutions build on earlier helpers where stated. Let `n` be the number of nodes. Complexity explanations treat allocation and release of a single node as constant-time operations. For each exercise, try empty, one-node, and longer lists where applicable.

### Exercise 1: Create a Node

Write `node_t *create_node(int val)` to allocate a node, store `val`, and set `next` to `NULL`. Return `NULL` if allocation fails.

Example: `create_node(10)` should produce `[10 | NULL]`.

<details>
<summary>Show solution</summary>

```c
node_t *create_node(int val) {
    node_t *node = malloc(sizeof *node);
    if (node == NULL) {
        return NULL;
    }
    node->val = val;
    node->next = NULL;
    return node;
}
```

Allocate enough memory for one node. Check the result before accessing it. The caller owns the returned node and must eventually free it.

**Time complexity:** O(1).

**Auxiliary space:** O(1), including one newly allocated node.

</details>

---

### Exercise 2: Traverse the List

Write `void print_list(const node_t *head)` to print every value in order. Print `NULL` at the end, including for an empty list.

Example: `10 → 20 → NULL`.

<details>
<summary>Show solution</summary>

```c
void print_list(const node_t *head) {
    const node_t *current = head;
    while (current != NULL) {
        printf("%d -> ", current->val);
        current = current->next;
    }
    printf("NULL\n");
}
```

Follow each next pointer until the list ends. Moving a local pointer does not change the caller's head.

**Time complexity:** O(n).

**Auxiliary space:** O(1).

</details>

---

### Exercise 3: Count the Nodes

Write `size_t count_nodes(const node_t *head)` to return the number of nodes. An empty list should return `0`.

<details>
<summary>Show solution</summary>

```c
size_t count_nodes(const node_t *head) {
    size_t count = 0;
    while (head != NULL) {
        count++;
        head = head->next;
    }
    return count;
}
```

Visit every node and increment a counter. `size_t` is an unsigned integer type commonly used for sizes and counts.

**Time complexity:** O(n).

**Auxiliary space:** O(1).

</details>

---

### Exercise 4: Find a Value

Write `node_t *find_node(node_t *head, int val)` to return the first matching node, or `NULL` if no node matches.

Example: searching for `20` in `10 → 20 → 20 → NULL` returns the first node containing 20.

<details>
<summary>Show solution</summary>

```c
node_t *find_node(node_t *head, int val) {
    while (head != NULL) {
        if (head->val == val) {
            return head;
        }
        head = head->next;
    }
    return NULL;
}
```

Stop at the first match. The returned pointer refers to an existing node; no copy is created.

**Time complexity:** O(n) worst case; O(1) if the head matches.

**Auxiliary space:** O(1).

</details>

---

### Exercise 5: Access by Index

Write `node_t *node_at(node_t *head, int index)` to return the node at a zero-based index. Return `NULL` for a negative or out-of-range index.

Example: index `1` in `10 → 20 → 30 → NULL` refers to the node containing 20.

<details>
<summary>Show solution</summary>

```c
node_t *node_at(node_t *head, int index) {
    if (index < 0) {
        return NULL;
    }
    while (head != NULL && index > 0) {
        head = head->next;
        index--;
    }
    return head;
}
```

Follow one link per position. Unlike an array, a linked list cannot jump directly to an index.

**Time complexity:** O(n) worst case.

**Auxiliary space:** O(1).

</details>

---

### Exercise 6: Update a Value

Write `int update_at(node_t *head, int index, int val)` to replace the value at an index. Return `1` on success or `0` for an invalid index. Use `node_at` from Exercise 5.

<details>
<summary>Show solution</summary>

```c
int update_at(node_t *head, int index, int val) {
    node_t *node = node_at(head, index);
    if (node == NULL) {
        return 0;
    }
    node->val = val;
    return 1;
}
```

Finding the node takes traversal. Once it is found, replacing its value takes constant time and does not change any links.

**Time complexity:** O(n) worst case.

**Auxiliary space:** O(1).

</details>

---

### Exercise 7: Insert at the Beginning

Write `int prepend(node_t **head, int val)` to add a node before the current head. Return `1` on success or `0` on allocation failure. Leave the list unchanged on failure. Use `create_node`.

Example: adding 5 to `10 → 20 → NULL` produces `5 → 10 → 20 → NULL`.

<details>
<summary>Show solution</summary>

```c
int prepend(node_t **head, int val) {
    node_t *node = create_node(val);
    if (node == NULL) {
        return 0;
    }
    node->next = *head;
    *head = node;
    return 1;
}
```

A pointer to the head pointer lets the function change the caller's head. Call it with `prepend(&head, 5)`. Link the new node to the old head before replacing it. The same steps work for an empty list.

**Time complexity:** O(1).

**Auxiliary space:** O(1), including the new node.

</details>

---

### Exercise 8: Insert at the End

Write `int append(node_t **head, int val)` without a stored tail pointer. Return `1` on success or `0` on allocation failure, leaving the list unchanged on failure. Use `create_node`.

<details>
<summary>Show solution</summary>

```c
int append(node_t **head, int val) {
    node_t *node = create_node(val);
    if (node == NULL) {
        return 0;
    }
    if (*head == NULL) {
        *head = node;
        return 1;
    }
    node_t *current = *head;
    while (current->next != NULL) {
        current = current->next;
    }
    current->next = node;
    return 1;
}
```

For an empty list, the new node becomes the head. Otherwise, walk to the last node and change its next pointer.

**Time complexity:** O(n) worst case; O(1) for an empty list.

**Auxiliary space:** O(1), including the new node.

</details>

---

### Exercise 9: Insert After a Known Node

Write `int insert_after(node_t *previous, int val)` to insert after `previous`. Return `0` if `previous` is `NULL` or allocation fails; otherwise return `1`. Use `create_node`.

<details>
<summary>Show solution</summary>

```c
int insert_after(node_t *previous, int val) {
    if (previous == NULL) {
        return 0;
    }
    node_t *node = create_node(val);
    if (node == NULL) {
        return 0;
    }
    node->next = previous->next;
    previous->next = node;
    return 1;
}
```

Preserve the old successor in the new node before changing the preceding node's link. This works after the last node too.

**Time complexity:** O(1), because the preceding node is already known.

**Auxiliary space:** O(1), including the new node.

</details>

---

### Exercise 10: Delete the First Node

Write `int delete_first(node_t **head)` to remove and free the first node. Return `1` if a node was removed or `0` if the list was empty.

<details>
<summary>Show solution</summary>

```c
int delete_first(node_t **head) {
    if (*head == NULL) {
        return 0;
    }
    node_t *removed = *head;
    *head = removed->next;
    free(removed);
    return 1;
}
```

Move the head before freeing the removed node. Removing the only node sets the head to NULL automatically.

**Time complexity:** O(1).

**Auxiliary space:** O(1).

</details>

---

### Exercise 11: Delete the Last Node

Write `int delete_last(node_t **head)` to remove and free the last node. Return `1` if a node was removed or `0` if the list was empty.

<details>
<summary>Show solution</summary>

```c
int delete_last(node_t **head) {
    if (*head == NULL) {
        return 0;
    }
    if ((*head)->next == NULL) {
        free(*head);
        *head = NULL;
        return 1;
    }
    node_t *previous = *head;
    while (previous->next->next != NULL) {
        previous = previous->next;
    }
    free(previous->next);
    previous->next = NULL;
    return 1;
}
```

Handle empty and one-node lists first. For longer lists, stop at the second-to-last node so its next link can be cleared.

**Time complexity:** O(n) worst case.

**Auxiliary space:** O(1).

</details>

---

### Exercise 12: Delete by Value

Write `int delete_value(node_t **head, int val)` to remove and free only the first matching node. Return `1` on removal or `0` if no match exists.

Example: deleting 20 from `10 → 20 → 20 → NULL` leaves `10 → 20 → NULL`.

<details>
<summary>Show solution</summary>

```c
int delete_value(node_t **head, int val) {
    node_t *previous = NULL;
    node_t *current = *head;
    while (current != NULL && current->val != val) {
        previous = current;
        current = current->next;
    }
    if (current == NULL) {
        return 0;
    }
    if (previous == NULL) {
        *head = current->next;
    } else {
        previous->next = current->next;
    }
    free(current);
    return 1;
}
```

Keep track of the preceding node while searching. A match at the head needs a head update; other matches need the predecessor's link updated.

**Time complexity:** O(n) worst case.

**Auxiliary space:** O(1).

</details>

---

### Exercise 13: Clear the List

Write `void clear_list(node_t **head)` to free all nodes and leave the head set to `NULL`. Clearing an empty list should do nothing.

<details>
<summary>Show solution</summary>

```c
void clear_list(node_t **head) {
    while (*head != NULL) {
        node_t *removed = *head;
        *head = removed->next;
        free(removed);
    }
}
```

Save access to the next node by updating the head before freeing the current node. Never read a node after it has been freed.

**Time complexity:** O(n).

**Auxiliary space:** O(1).

</details>

[Back to Linked Lists](README.md)
