# Matrix Traversal in C++

A matrix organizes values into rows and columns. Traversal means visiting its cells in a deliberate order while keeping both coordinates within bounds.

![Matrix Traversal overview](assets/matrix-traversal-overview.svg)

This lesson uses C++17. Review [Arrays](../../01-arrays/cpp_arrays.md) for the underlying structure or technique.

## When to Use It

Use matrix traversal for tables, images, boards, and grids. Start with row-by-row traversal before adding more complicated paths.

A `std::vector<std::vector<int>>` can have rows of different lengths. This lesson requires a rectangular matrix and rejects unequal row lengths. A rectangular vector of vectors still stores each row separately; it is not necessarily one contiguous block.

## Worked Example

For `[[1, 2, 3], [4, 5, 6]]`:

```text
            column
           0  1  2
row 0      1  2  3
row 1      4  5  6

Row-major visit order: 1 → 2 → 3 → 4 → 5 → 6
```

Use the outer loop for rows and the inner loop for columns. Cell `(1, 2)` contains 6. Its valid orthogonal neighbors are `(0, 2)` and `(1, 1)`; diagonals are excluded.

For movement problems, check a neighbor's coordinates before accessing it. A top-row cell has no upward neighbor, and a left-column cell has no leftward neighbor.

## C++ Implementation

```cpp
#include <cstddef>
#include <iostream>
#include <stdexcept>
#include <vector>

using Matrix = std::vector<std::vector<int>>;

std::size_t column_count(const Matrix& matrix) {
    if (matrix.empty()) return 0;
    const auto columns = matrix.front().size();
    for (const auto& row : matrix) {
        if (row.size() != columns) throw std::invalid_argument("Ragged matrix");
    }
    return columns;
}

long long matrix_sum(const Matrix& matrix) {
    const auto columns = column_count(matrix);
    long long total = 0;
    for (std::size_t row = 0; row < matrix.size(); ++row) {
        for (std::size_t col = 0; col < columns; ++col) {
            total += matrix[row][col];
        }
    }
    return total;
}

int main() {
    const Matrix matrix = {{1, 2, 3}, {4, 5, 6}};
    std::cout << matrix_sum(matrix) << "\n"; // 21
}
```

## Implementation Details

An empty matrix and a matrix whose rows are all empty both sum to zero. Rectangular validation happens before processing any cell. `long long` stores the sum; assume intermediate totals fit.

For unsigned indices, test `row > 0` before computing `row - 1`, and `col > 0` before computing `col - 1`. For moving down or right, check the new position against the row or column count. Keep `(row, column)` order consistent.

## Complexity

O(R + R × C) time including validation, and O(1) auxiliary space. For positive column count this simplifies to O(R × C). The O(R) term still matters when every row is empty.

## Common Mistakes

- Reading matrix[0] before checking empty input.
- Using the row count as the column count on a rectangular matrix.
- Assuming vector-of-vector rows always have equal lengths.
- Subtracting from an unsigned zero index before checking the boundary.

## Exercises

Try each exercise before opening its solution. Reuse the definitions and headers above; place exercise helpers before `main`.

### Exercise 1: Visit by Columns

What is the column-major order for `[[1, 2, 3], [4, 5, 6]]`?

<details>
<summary>Show solution</summary>

`1, 4, 2, 5, 3, 6`. Put the column loop outside and the row loop inside. Every cell is still visited once.

</details>

### Exercise 2: Row Totals

Write a helper returning one total per row, keeping the rectangular-input contract.

<details>
<summary>Show solution</summary>

```cpp
std::vector<long long> row_sums(const Matrix& matrix) {
    column_count(matrix);
    std::vector<long long> result;
    result.reserve(matrix.size());
    for (const auto& row : matrix) {
        long long total = 0;
        for (int value : row) total += value;
        result.push_back(total);
    }
    return result;
}
```

The example returns `[6, 15]`. O(R + R × C) time and O(R) output storage; each running total uses constant space.

</details>

### Exercise 3: Boundary Neighbors

List the orthogonal neighbors of `(0, 0)` in a 2-by-3 matrix. What about a 1-by-1 matrix?

<details>
<summary>Show solution</summary>

In the 2-by-3 matrix: `(1, 0)` and `(0, 1)`. In a 1-by-1 matrix there are none. Never wrap across a boundary.

</details>

[Review Arrays](../../01-arrays/cpp_arrays.md)
