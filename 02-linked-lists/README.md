# Linked Lists

A **linked list** stores a sequence of elements in separate objects called **nodes**. Each node holds a value and a link to the next node.

```text
head
 |
 v
[10 | next] → [20 | next] → [30 | NULL]
```

The **head** points to the first node. `NULL` means there is no next node, so the last node marks the end of the list. An empty list has `head = NULL`.

Unlike an array, the nodes do not need to sit next to one another in memory. Their links determine the order of the sequence.

## How Linked Lists Are Stored

Each node is stored somewhere in memory. A link holds the address of another node, rather than the value stored in that node.

```text
Node address:     1000              2400              1600
Node contents:  [10 | 2400]       [20 | 1600]       [30 | NULL]

Logical order:  10 → 20 → 30
```

These addresses are only an illustration. The second node can be far from the first, and the third can be at a lower address than the second.

In C, a node for a singly linked list can be represented like this:

```c
struct node {
    int value;
    struct node *next;
};
```

The `next` pointer connects this node to another node of the same type. In languages with managed references, the idea is similar even though memory management works differently.

## Types of Linked Lists

### Singly Linked Lists

Each node links only to the next node:

```text
head → [10] → [20] → [30] → NULL
```

Traversal goes forward. To reach a previous node, we generally have to start from the head and find it again, unless we already saved a reference to it.

### Doubly Linked Lists

Each node links to both its next and previous nodes:

```text
NULL ← [10] ⇄ [20] ⇄ [30] → NULL
        ↑             ↑
       head          tail
```

We can move in either direction. Removing a known node is easier because its previous node is available directly. The tradeoff is an extra pointer per node and more links to update.

### Circular Linked Lists

The last node links back to the first instead of ending at `NULL`:

```text
head → [10] → [20] → [30]
        ↑             |
        └─────────────┘
```

A circular list can be singly or doubly linked. Traversal needs a stopping rule, such as returning to the starting node. Waiting for `NULL` would not stop a traversal of this list.

## Head, Tail, and Length

A list can keep additional information to make some operations faster:

- **Head:** A reference to the first node.
- **Tail:** An optional reference to the last node. It makes appending to a singly linked list possible without traversing it.
- **Length:** An optional stored count of nodes. Reading it takes `O(1)` time, but insertion and deletion must keep it up to date.

Without a stored length, counting the nodes takes `O(n)` time. In an empty list with a tail reference, both head and tail are `NULL`.

## Basic Operations

The following examples use a non-circular **singly linked list**. Let `n` be the number of nodes. Assume reading a value, comparing values, and changing a link take constant time.

### Access an Element by Position

To reach index `2`, start at the head and follow two links:

```text
Index:    0      1      2
head →  [10] → [20] → [30] → NULL
         start   hop    hop
```

**Time complexity: O(n) in the worst case.** We cannot calculate a node's address from its index as we can with an array. Accessing index `i` takes `O(i + 1)` time; accessing the first node takes `O(1)`.

### Update a Value

Once we have a reference to a node, we can replace its value directly:

```text
Before:  head → [10] → [20] → [30] → NULL
Update the known middle node to 25
After:   head → [10] → [25] → [30] → NULL
```

**Time complexity: O(1) for a known node.** If we must first find the node by index or value, that search can take `O(n)`.

Changing a node's value does not change its links.

### Traverse or Search

Start at the head and follow `next` until the list ends. For a search, stop early if the value is found.

```text
Find 30:
head → [10] → [20] → [30] → NULL
        check   check   found
```

**Traversal: O(n)** when visiting all nodes with constant work per visit.

**Search: O(n) in the worst case**, because the value may be last or absent. The best case is `O(1)` if the head matches.

Sorting a linked list does not give it the same efficient binary search as an array. Reaching a middle position still requires following links.

### Insert at the Beginning

Create a new node, link it to the current head, then make it the new head:

```text
Before:  head → [20] → [30] → NULL

1. New node's next → old head
   [10] → [20] → [30] → NULL

2. Head → new node
   head → [10] → [20] → [30] → NULL
```

The order matters: preserve access to the old head before replacing it.

**Time complexity: O(1) for the link changes.** Existing nodes do not move. If the list was empty and we maintain a tail, the new node becomes both head and tail.

### Insert After a Known Node

To insert 25 after the node containing 20:

```text
Before:  [10] → [20] → [30] → NULL

1. New node's next → node containing 30
2. Node containing 20's next → new node

After:   [10] → [20] → [25] → [30] → NULL
```

**Time complexity: O(1) once the preceding node is known.** Finding that node first can take `O(n)`.

Unlike array insertion, this does not shift later elements. If the insertion is after the last node, update the tail if one is stored.

### Append at the End

With a tail reference, link the old tail to the new node and move the tail reference:

```text
Before:  head → [10] → [20] → NULL
                        ↑
                       tail

After:   head → [10] → [20] → [30] → NULL
                               ↑
                              tail
```

**Time complexity: O(1) with a tail reference**, assuming constant-time node allocation. Without a tail, finding the last node takes `O(n)`.

When appending to an empty list, set both head and tail to the new node.

### Delete the First Node

Save the old head, move head to its next node, then release the removed node if the language requires it:

```text
Before:  head → [10] → [20] → [30] → NULL
After:           head → [20] → [30] → NULL
```

**Time complexity: O(1).** If there was only one node, the new head is `NULL`; any stored tail must also become `NULL`.

An empty list has no first node to remove, so check for that case before accessing the head.

### Delete After a Known Node

To remove 20, use the preceding node, which contains 10:

```text
Before:  [10] → [20] → [30] → NULL

1. Save a reference to the node containing 20
2. Set node 10's next to node 20's next
3. Release the removed node when required

After:   [10] ───────→ [30] → NULL
```

**Time complexity: O(1) when the preceding node is known.** Searching for that node can take `O(n)`. If the removed node was the tail, the preceding node becomes the new tail.

Having only the node to remove is not generally enough for constant-time deletion from a singly linked list: we need to update the previous node's link. A doubly linked list stores that previous link directly.

### Delete the Last Node

In a singly linked list, we need to find the node before the tail so it can become the new last node.

**Time complexity: O(n) in the worst case, even with a tail reference.** The tail does not tell us where its predecessor is.

In a doubly linked list with a tail reference, deleting the last node takes `O(1)` because the previous link is available.

## Complexity Summary

These are worst-case costs for non-circular lists. The doubly linked list below maintains both head and tail. Node allocation and release are treated as constant-time operations for this comparison; actual allocator costs can vary.

| Operation | Singly linked list | Doubly linked list with head and tail |
| --- | --- | --- |
| Access by index | O(n) | O(n) |
| Update a known node's value | O(1) | O(1) |
| Traverse or search | O(n) | O(n) |
| Insert at the beginning | O(1) | O(1) |
| Insert after a known node | O(1) | O(1) |
| Append | O(1) with tail; O(n) without | O(1) |
| Delete the first node | O(1) | O(1) |
| Delete a known node | O(1) with predecessor or at head; O(n) otherwise | O(1) |
| Delete the last node | O(n) | O(1) |
| Read length | O(1) if stored; O(n) otherwise | O(1) if stored; O(n) otherwise |

The key distinction is **finding a node versus changing its links**. A few link updates may take `O(1)`, while locating the place to make them can take `O(n)`.

## Space and Memory Management

A linked list uses **O(n)** storage for constant-size values. Each node also needs one or more links, and separate allocations may add overhead.

Iterative traversal uses **O(1) auxiliary space**: a current-node reference is enough. A recursive traversal can use **O(n) auxiliary space** on the call stack.

In C, dynamically allocated nodes must eventually be freed. Save any link you still need before freeing a node, and never read through a pointer to freed memory. Allocating a node can fail, so check the allocation before writing to it or linking it into the list.

To free a whole list, save the current node's next pointer, free the current node, and then move to the saved next node. Repeat until the list is empty.

## Arrays and Linked Lists

| Property | Array | Linked list |
| --- | --- | --- |
| Storage layout | Contiguous slots | Nodes can be scattered |
| Access by index | O(1) | O(n) worst case |
| Insert near the beginning | Usually shifts elements | Changes links once the location is known |
| Extra storage | May reserve unused capacity | Needs links and often allocation overhead per node |
| Traversal | Nearby slots often help memory access | Following scattered nodes can be slower |
| Growth | May require larger storage and copying | Can allocate and link another node |

A linked list does not automatically make insertion or deletion faster. If every operation starts by searching for an index, finding the position may dominate the work.

## When to Use Linked Lists

Linked lists fit tasks where operations happen at the ends or at nodes we already hold references to. A singly linked list can support a stack at its head or a queue with head and tail references. A doubly linked list is useful when moving both forward and backward or removing known nodes frequently.

Arrays are often a better fit for frequent indexed access and compact, efficient traversal. Choose based on the operations the program actually performs, rather than the fact that a linked list can grow one node at a time.

[Review Arrays](../01-arrays/README.md) · [Review Big-O Notation](../00-complexity-analysis/README.md)
