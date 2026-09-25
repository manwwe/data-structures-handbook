# Trees

A **tree** represents relationships between elements as a hierarchy. In a rooted tree, one node is the **root**, and every other node has exactly one parent. Following child links never takes us back to an earlier node: there are no cycles.

```text
         A            ← root
        / \
       B   C          ← children of A
      / \   \
     D   E   F        ← leaves
```

Trees can represent nested folders, document structure, or decision paths. Their shape describes relationships, not necessarily sorted data.

## Basic Terminology

- **Parent and child:** A node directly above another node is its parent; the lower node is its child.
- **Siblings:** Nodes with the same parent, such as B and C.
- **Leaf:** A node with no children.
- **Edge:** A connection between a parent and a child.
- **Subtree:** A node together with all its descendants.
- **Depth:** The number of edges from the root to a node. The root has depth 0.
- **Height:** The number of edges on the longest downward path to a leaf. A leaf has height 0.

The example tree has height 2. For recursive formulas in this chapter, an empty tree has height -1. Other resources may count nodes rather than edges, so check the convention when comparing formulas.

A nonempty tree with `n` nodes has `n - 1` edges. There is exactly one path between any two nodes when connections can be followed in either direction.

## Binary Trees

A **binary tree** gives each node at most two children, called left and right:

```text
         8
        / \
       3   10
      / \    \
     1   6    14
```

A child can be absent. A binary tree does not, by itself, require any ordering of values.

Some useful shape descriptions are:

- **Full:** Every node has either zero or two children.
- **Complete:** Every level except possibly the last is filled, and the last level fills from left to right.
- **Perfect:** Every internal node has two children and all leaves have the same depth.

These terms describe shape; they do not imply a search ordering.

## Binary Search Trees

A **binary search tree (BST)** adds an ordering rule. In this chapter, keys are unique:

```text
Every key in the left subtree < node's key
Every key in the right subtree > node's key
```

The rule applies to entire subtrees, not only immediate children. In the tree above, every descendant on the left of 8 must be less than 8.

Other designs support duplicates using a count or a documented placement policy. Our exercises reject duplicate insertions without changing the tree.

## Searching

To find 6 in the example BST:

```text
6 < 8 → go left
6 > 3 → go right
6 = 6 → found
```

Each comparison chooses one subtree. Reaching a missing child means the key is absent.

**Time complexity: O(h + 1)** for height `h`, including the root visit. This is commonly written as `O(h)` for nontrivial trees. A balanced tree has logarithmic height; a chain-shaped tree can have linear height.

Searching an arbitrary binary tree without the BST ordering can require visiting all `n` nodes.

## Insertion

Follow the search path until reaching an empty child position, then link in a new node:

```text
Insert 4:

Before:       8             After:        8
             / \                        / \
            3   10                     3   10
             \                          \
              6                          6
                                        /
                                       4
```

A new key becomes a leaf. An empty tree receives a new root. If allocation fails, leave the existing tree unchanged.

Insertion takes `O(h + 1)` time. Ordinary BST insertion does not guarantee balance.

## Minimum and Maximum

In a nonempty BST, the smallest key is at the leftmost node. The largest is at the rightmost node.

```text
Minimum: repeatedly follow left
Maximum: repeatedly follow right
```

Both take `O(h + 1)` time. An empty tree has neither a minimum nor a maximum, so report absence rather than using a possible key as an error value.

## Deletion

First find the node. Then handle its number of children.

### No Children

Unlink the leaf from its parent and release it. If it was the root, the tree becomes empty.

### One Child

Link the parent directly to the child, then release the removed node:

```text
Before:  8 → right 10 → right 14
After:   8 → right 14
```

When deleting the root, its child becomes the new root.

### Two Children

Find the **inorder successor**, the smallest key in the node's right subtree. Copy that key into the node, then remove the successor from its old position.

```text
Before:       8             Delete 8:       10
             / \                          /  \
            3   10                       3    14
                  \
                   14
```

The successor has no left child, so its removal is a zero-child or one-child case. If nodes store key–value pairs, move the associated value with the key.

Deletion takes `O(h + 1)` time. The basic procedure preserves BST ordering but does not restore balance.

## Traversals

A **traversal** visits every node. Different visit orders serve different purposes.

For this tree:

```text
        8
       / \
      3   10
     / \
    1   6
```

| Traversal | Rule | Result |
| --- | --- | --- |
| Preorder | Node, left, right | 8, 3, 1, 6, 10 |
| Inorder | Left, node, right | 1, 3, 6, 8, 10 |
| Postorder | Left, right, node | 1, 6, 3, 10, 8 |
| Level order | Level by level, left to right | 8, 3, 10, 1, 6 |

Inorder traversal produces sorted keys **for a BST**. It does not sort an arbitrary binary tree.

Preorder can record a hierarchy, though reconstructing a general binary tree also requires structural information. Postorder is useful for releasing nodes because children are handled before their parent. Level order uses a queue to hold nodes waiting to be visited.

Every full traversal takes `O(n)` time with constant work per node. Recursive depth-first traversal uses `O(h + 1)` call-stack space. Level order uses `O(w)` queue space, where `w` is the largest number of nodes at a level.

## Height and Balance

Insertion order can drastically change an ordinary BST:

```text
Insert 1, 2, 3, 4:        Insert 3, 2, 4, 1:

1                                3
 \                              / \
  2                            2   4
   \                          /
    3                        1
     \
      4
```

The first tree behaves like a linked list when searching along its long path. The second has shorter paths.

Self-balancing BSTs, such as AVL and red-black trees, maintain rules that keep height `O(log n)`. They can rotate local groups of nodes while preserving the sorted inorder sequence. Their balancing rules are beyond the basic BST exercises.

## Complexity Summary

Assume constant-time key comparisons and allocation or release of one node. Here `n` is the node count and `h` the height.

| Operation | Balanced height | Worst-case unbalanced BST |
| --- | --- | --- |
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |
| Minimum or maximum | O(log n) | O(n) |
| Count nodes without a stored count | O(n) | O(n) |
| Compute height by traversal | O(n) | O(n) |
| Traverse or clear all nodes | O(n) | O(n) |

A pointer-based tree uses `O(n)` storage. Iterative search needs `O(1)` auxiliary space. Recursive operations can need `O(h + 1)` call-stack space, which becomes `O(n)` for a chain. Deep recursion can exhaust the call stack even when enough memory remains for the nodes themselves.

Computing height visits both subtrees; it is not a single-path search. A stored node count makes reading size `O(1)`, provided mutations update it correctly.

## When to Use Trees

Use general trees for hierarchical relationships. Use an ordered search tree when you need key lookup together with sorted traversal, minimum/maximum queries, or range queries.

A hash table often offers faster expected exact-key lookup, but does not inherently keep keys sorted. An array gives direct indexed access, which a basic pointer-based BST does not provide efficiently without additional metadata.

A heap is another tree-based structure with a different ordering rule, designed to expose a minimum or maximum efficiently. It is not a BST.

## Language Guides and Practice

- [Trees in C: examples and exercises](c_trees.md)
- [Trees in C++: examples and exercises](cpp_trees.md)

[Review Linked Lists](../02-linked-lists/README.md) · [Review Queues](../04-queues/README.md) · [Review Hash Tables](../05-hash-tables/README.md)
