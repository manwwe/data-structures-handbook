# Trees in C

Read the [theory](README.md) first, then work through the language examples and exercises.

## Use a Binary Search Tree

C has no standard tree container. The exercises define a node with a key and two child pointers. A root pointer identifies the tree; `NULL` represents an empty tree or missing child. Keys smaller than a node go left; larger keys go right.

The following usage example calls the helpers built in the exercises. To run it, put their shared definition and solutions in order before this `main` function, and include `<stdio.h>`.

```c
int main(void) {
    node_t *root = NULL;
    if (!tree_insert(&root, 20) || !tree_insert(&root, 10)) {
        tree_clear(&root);
        return 1;
    }
    print_inorder(root); // 10, then 20
    tree_remove(&root, 10);
    tree_clear(&root);
    return 0;
}
```

Pass `&root` when a function may replace the root pointer. Nodes allocated with `malloc` must eventually be released with `free`; the clear helper releases the whole tree. Do not share owning child pointers or reuse removed nodes. This tree is unbalanced, so search, insertion, and removal can take O(n).

## Basic Exercises

Build an unbalanced binary search tree with unique integer keys. Try each exercise before opening its solution. Combine the shared definition and solutions in order.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stddef.h>

typedef struct node {
    int key;
    struct node *left;
    struct node *right;
} node_t;
```

Start with `node_t *root = NULL;`. Every left-subtree key must be smaller than its parent key, and every right-subtree key larger. Nodes must form a tree with no cycles or shared children. The tree owns nodes allocated by `create_node`; do not make shallow owning copies.

Pass the address of a valid root variable to functions taking `node_t **root`. The root itself may be empty. Node pointers must be live or empty, and removed nodes must not be accessed again.

Let `n` be the node count and `h` the height in edges. Assume the count fits in the size type and that allocation and release of one node take constant time. Recursive solutions can require linear call-stack space for a chain-shaped tree.

### Exercise 1: Create a Node

Write `node_t *create_node(int key)` to allocate a leaf with both children empty. Return an empty pointer on allocation failure.

<details>
<summary>Show solution</summary>

```c
node_t *create_node(int key) {
    node_t *node = malloc(sizeof *node);
    if (node == NULL) return NULL;
    node->key = key;
    node->left = NULL;
    node->right = NULL;
    return node;
}
```

Initialize both child links before using the node. The tree owns allocated nodes and must eventually release them.

**Time complexity:** O(1).

**Auxiliary space:** O(1), including the new node.

</details>

---

### Exercise 2: Search for a Key

Write `node_t *tree_find(node_t *root, int key)` to return the matching node or an empty pointer if absent.

<details>
<summary>Show solution</summary>

```c
node_t *tree_find(node_t *root, int key) {
    while (root != NULL) {
        if (key == root->key) return root;
        root = key < root->key ? root->left : root->right;
    }
    return NULL;
}
```

Use the ordering rule to choose one child at each step. The returned pointer refers to an existing node, not a copy.

**Time complexity:** O(h + 1), or O(n) in the worst case.

**Auxiliary space:** O(1).

</details>

---

### Exercise 3: Insert a Key

Write `int tree_insert(node_t **root, int key)` using `create_node`. Return 1 for a new insertion, 0 for a duplicate, and -1 for allocation failure. Leave the tree unchanged in the last two cases. Call it with `tree_insert(&root, key)`.

<details>
<summary>Show solution</summary>

```c
int tree_insert(node_t **root, int key) {
    node_t **link = root;
    while (*link != NULL) {
        if (key == (*link)->key) return 0;
        link = key < (*link)->key ? &(*link)->left : &(*link)->right;
    }
    node_t *node = create_node(key);
    if (node == NULL) return -1;
    *link = node;
    return 1;
}
```

The pointer-to-pointer tracks the link that may change: either the root variable or a child field. Assign only after successful allocation. Inserting into an empty tree needs no separate case.

**Time complexity:** O(h + 1).

**Auxiliary space:** O(1), including the new node.

</details>

---

### Exercise 4: Find the Minimum

Write `const node_t *tree_min(const node_t *root)` to return the smallest-key node, or an empty pointer for an empty tree.

<details>
<summary>Show solution</summary>

```c
const node_t *tree_min(const node_t *root) {
    if (root == NULL) return NULL;
    while (root->left != NULL) root = root->left;
    return root;
}
```

The smallest key is at the leftmost node. Following right links instead would find the maximum.

**Time complexity:** O(h + 1).

**Auxiliary space:** O(1).

</details>

---

### Exercise 5: Print in Sorted Order

Write `void print_inorder(const node_t *root)` to print the keys in increasing order. An empty tree should print nothing.

<details>
<summary>Show solution</summary>

```c
void print_inorder(const node_t *root) {
    if (root == NULL) return;
    print_inorder(root->left);
    printf("%d\n", root->key);
    print_inorder(root->right);
}
```

Visit the left subtree, then this node, then the right subtree. BST ordering and unique keys make the output strictly increasing.

**Time complexity:** O(n).

**Auxiliary space:** O(h + 1) for recursive calls.

</details>

---

### Exercise 6: Count the Nodes

Write `size_t tree_count(const node_t *root)` without a stored node count. Return zero for an empty tree.

<details>
<summary>Show solution</summary>

```c
size_t tree_count(const node_t *root) {
    if (root == NULL) return 0;
    return 1 + tree_count(root->left) + tree_count(root->right);
}
```

Count this node and add the counts of both subtrees. Every node is visited once.

**Time complexity:** O(n).

**Auxiliary space:** O(h + 1) for recursive calls.

</details>

---

### Exercise 7: Compute the Height

Write `int tree_height(const node_t *root)`. Count edges: an empty tree has height -1 and a leaf has height 0. Assume the height fits in int.

<details>
<summary>Show solution</summary>

```c
int tree_height(const node_t *root) {
    if (root == NULL) return -1;
    int left_height = tree_height(root->left);
    int right_height = tree_height(root->right);
    return 1 + (left_height > right_height ? left_height : right_height);
}
```

Compute both child heights and take the larger one, adding the edge to that child. For a leaf, both child heights are -1, giving height 0.

**Time complexity:** O(n), because both subtrees are visited.

**Auxiliary space:** O(h + 1) for recursive calls.

</details>

---

### Exercise 8: Delete a Key

Write `int tree_remove(node_t **root, int key)` to remove a key and release one node. Return 1 on removal or 0 if absent. Handle zero, one, and two children, including deletion of the root.

<details>
<summary>Show solution</summary>

```c
int tree_remove(node_t **root, int key) {
    node_t **link = root;
    while (*link != NULL && (*link)->key != key) {
        link = key < (*link)->key ? &(*link)->left : &(*link)->right;
    }
    if (*link == NULL) return 0;
    node_t *node = *link;
    if (node->left != NULL && node->right != NULL) {
        node_t **successor_link = &node->right;
        while ((*successor_link)->left != NULL) {
            successor_link = &(*successor_link)->left;
        }
        node_t *successor = *successor_link;
        node->key = successor->key;
        *successor_link = successor->right;
        free(successor);
    } else {
        *link = node->left != NULL ? node->left : node->right;
        free(node);
    }
    return 1;
}
```

For zero or one child, replace the node with its existing child or an empty link. For two children, copy the right subtree's smallest key and unlink that successor, preserving its possible right child. Do not assume saved node pointers retain the same key after deletion: this algorithm changes the target key and frees the successor.

**Time complexity:** O(h + 1).

**Auxiliary space:** O(1), using iterative searches.

</details>

---

### Exercise 9: Clear the Tree

Write `void tree_clear(node_t **root)` to release all nodes and leave the root empty. It must work on an empty tree too.

<details>
<summary>Show solution</summary>

```c
void tree_clear(node_t **root) {
    if (*root == NULL) return;
    tree_clear(&(*root)->left);
    tree_clear(&(*root)->right);
    free(*root);
    *root = NULL;
}
```

Use postorder: release children before their parent. Setting each link to empty leaves the root safe to reuse.

**Time complexity:** O(n).

**Auxiliary space:** O(h + 1) for recursive calls.

</details>

## Check the Completed Tree

Insert 8, 3, 10, 1, 6, and 14. Inorder output should be 1, 3, 6, 8, 10, 14; count should be 6, height 2, and minimum 1.

Check empty-tree operations, duplicate insertion, absent keys, allocation failure, and deleting a leaf, a one-child node, and a two-child node. Test both a direct right-child successor and a deeper successor that has its own right child. Also delete the only node, clear twice, and reuse the tree after clearing.

[Back to Trees](README.md)
