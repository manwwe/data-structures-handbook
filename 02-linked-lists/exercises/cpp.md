# Linked-List Exercises in C++

These C++17 exercises implement links directly using raw pointers. The list owns nodes created with `new`, and removal releases them with `delete`. This makes ownership and link changes visible; standard containers and smart pointers can automate ownership in other designs.

`new (std::nothrow)` returns `nullptr` on allocation failure for these nodes, allowing the exercises to report failure with a status. Do not mix these nodes with `malloc`/`free`. Solutions build on earlier exercises and can be combined in order.

Try each exercise before opening its solution. Use this node definition throughout:

```cpp
#include <iostream>
#include <cstddef>
#include <new>

struct node_t {
    int val;
    node_t *next;
};
```

Use a singly linked list with no cycles, stored tail, or stored length. An empty list has `node_t *head = nullptr`. All nodes removed by these functions must have been dynamically allocated, for example with `create_node` from Exercise 1.

For functions receiving `node_t **head`, pass the address of a valid head variable, such as `&head`; the list itself may be empty. Other node pointers must refer to live nodes or be `nullptr`. After deleting a node, do not use any saved pointer to it.

The solutions build on earlier helpers where stated. Let `n` be the number of nodes. Complexity explanations treat allocation and release of a single node as constant-time operations. For each exercise, try empty, one-node, and longer lists where applicable.

### Exercise 1: Create a Node

Write `node_t *create_node(int val)` to allocate a node, store `val`, and set `next` to `nullptr`. Return `nullptr` if allocation fails.

Example: `create_node(10)` should produce `[10 | nullptr]`.

<details>
<summary>Show solution</summary>

```cpp
node_t *create_node(int val) {
    node_t *node = new (std::nothrow) node_t;
    if (node == nullptr) {
        return nullptr;
    }
    node->val = val;
    node->next = nullptr;
    return node;
}
```

Allocate enough memory for one node. Check the result before accessing it. The caller owns the returned node and must eventually release it with `delete`.

**Time complexity:** O(1).

**Auxiliary space:** O(1), including one newly allocated node.

</details>

---

### Exercise 2: Traverse the List

Write `void print_list(const node_t *head)` to print every value in order. Print `nullptr` at the end, including for an empty list.

Example: `10 → 20 → nullptr`.

<details>
<summary>Show solution</summary>

```cpp
void print_list(const node_t *head) {
    const node_t *current = head;
    while (current != nullptr) {
        std::cout << current->val << " -> ";
        current = current->next;
    }
    std::cout << "nullptr\n";
}
```

Follow each next pointer until the list ends. Moving a local pointer does not change the caller's head.

**Time complexity:** O(n).

**Auxiliary space:** O(1).

</details>

---

### Exercise 3: Count the Nodes

Write `std::size_t count_nodes(const node_t *head)` to return the number of nodes. An empty list should return `0`.

<details>
<summary>Show solution</summary>

```cpp
std::size_t count_nodes(const node_t *head) {
    std::size_t count = 0;
    while (head != nullptr) {
        count++;
        head = head->next;
    }
    return count;
}
```

Visit every node and increment a counter. `std::size_t` is an unsigned integer type commonly used for sizes and counts.

**Time complexity:** O(n).

**Auxiliary space:** O(1).

</details>

---

### Exercise 4: Find a Value

Write `node_t *find_node(node_t *head, int val)` to return the first matching node, or `nullptr` if no node matches.

Example: searching for `20` in `10 → 20 → 20 → nullptr` returns the first node containing 20.

<details>
<summary>Show solution</summary>

```cpp
node_t *find_node(node_t *head, int val) {
    while (head != nullptr) {
        if (head->val == val) {
            return head;
        }
        head = head->next;
    }
    return nullptr;
}
```

Stop at the first match. The returned pointer refers to an existing node; no copy is created.

**Time complexity:** O(n) worst case; O(1) if the head matches.

**Auxiliary space:** O(1).

</details>

---

### Exercise 5: Access by Index

Write `node_t *node_at(node_t *head, int index)` to return the node at a zero-based index. Return `nullptr` for a negative or out-of-range index.

Example: index `1` in `10 → 20 → 30 → nullptr` refers to the node containing 20.

<details>
<summary>Show solution</summary>

```cpp
node_t *node_at(node_t *head, int index) {
    if (index < 0) {
        return nullptr;
    }
    while (head != nullptr && index > 0) {
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

```cpp
int update_at(node_t *head, int index, int val) {
    node_t *node = node_at(head, index);
    if (node == nullptr) {
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

Example: adding 5 to `10 → 20 → nullptr` produces `5 → 10 → 20 → nullptr`.

<details>
<summary>Show solution</summary>

```cpp
int prepend(node_t **head, int val) {
    node_t *node = create_node(val);
    if (node == nullptr) {
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

```cpp
int append(node_t **head, int val) {
    node_t *node = create_node(val);
    if (node == nullptr) {
        return 0;
    }
    if (*head == nullptr) {
        *head = node;
        return 1;
    }
    node_t *current = *head;
    while (current->next != nullptr) {
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

Write `int insert_after(node_t *previous, int val)` to insert after `previous`. Return `0` if `previous` is `nullptr` or allocation fails; otherwise return `1`. Use `create_node`.

<details>
<summary>Show solution</summary>

```cpp
int insert_after(node_t *previous, int val) {
    if (previous == nullptr) {
        return 0;
    }
    node_t *node = create_node(val);
    if (node == nullptr) {
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

```cpp
int delete_first(node_t **head) {
    if (*head == nullptr) {
        return 0;
    }
    node_t *removed = *head;
    *head = removed->next;
    delete removed;
    return 1;
}
```

Move the head before freeing the removed node. Removing the only node sets the head to nullptr automatically.

**Time complexity:** O(1).

**Auxiliary space:** O(1).

</details>

---

### Exercise 11: Delete the Last Node

Write `int delete_last(node_t **head)` to remove and free the last node. Return `1` if a node was removed or `0` if the list was empty.

<details>
<summary>Show solution</summary>

```cpp
int delete_last(node_t **head) {
    if (*head == nullptr) {
        return 0;
    }
    if ((*head)->next == nullptr) {
        delete *head;
        *head = nullptr;
        return 1;
    }
    node_t *previous = *head;
    while (previous->next->next != nullptr) {
        previous = previous->next;
    }
    delete previous->next;
    previous->next = nullptr;
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

Example: deleting 20 from `10 → 20 → 20 → nullptr` leaves `10 → 20 → nullptr`.

<details>
<summary>Show solution</summary>

```cpp
int delete_value(node_t **head, int val) {
    node_t *previous = nullptr;
    node_t *current = *head;
    while (current != nullptr && current->val != val) {
        previous = current;
        current = current->next;
    }
    if (current == nullptr) {
        return 0;
    }
    if (previous == nullptr) {
        *head = current->next;
    } else {
        previous->next = current->next;
    }
    delete current;
    return 1;
}
```

Keep track of the preceding node while searching. A match at the head needs a head update; other matches need the predecessor's link updated.

**Time complexity:** O(n) worst case.

**Auxiliary space:** O(1).

</details>

---

### Exercise 13: Clear the List

Write `void clear_list(node_t **head)` to free all nodes and leave the head set to `nullptr`. Clearing an empty list should do nothing.

<details>
<summary>Show solution</summary>

```cpp
void clear_list(node_t **head) {
    while (*head != nullptr) {
        node_t *removed = *head;
        *head = removed->next;
        delete removed;
    }
}
```

Save access to the next node by updating the head before freeing the current node. Never read a node after it has been freed.

**Time complexity:** O(n).

**Auxiliary space:** O(1).

</details>

[Back to Linked Lists](../README.md)
