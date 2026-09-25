# Graph Exercises in C++

Build a simple, undirected, unweighted graph using an adjacency matrix. Vertices have IDs from 0 through count - 1. Try each exercise before opening its solution; combine the shared definition and functions in order.

```cpp
constexpr int MAX_VERTICES = 8;

struct graph_t {
    int edges[MAX_VERTICES][MAX_VERTICES];
    int count;
};
```

Pass valid graph pointers and initialize successfully before other operations. Output arrays must have at least count writable elements and must not overlap the graph. Keep the matrix symmetric with a zero diagonal by using the edge operations below.

Let `V` be the active vertex count. This example supports at most eight vertices. Complexity bounds describe scaling to larger capacities: storage for the embedded matrix is O(MAX_VERTICES²), and the traversal buffers reserve O(MAX_VERTICES) slots even when fewer vertices are active. With capacity proportional to V, these become O(V²) and O(V). Simple operations use O(1) auxiliary space; recursive DFS additionally uses up to O(V) call-stack space.

These C++17 exercises keep the matrix and traversal logic explicit instead of using a graph library.

### Exercise 1: Initialize a Graph

Write `int graph_init(graph_t *graph, int count)` to create vertices 0 through count - 1 with no edges. Return 1 on success, or 0 for a negative count or one above capacity, leaving the object unchanged on failure.

<details>
<summary>Show solution</summary>

```cpp
int graph_init(graph_t *graph, int count) {
    if (count < 0 || count > MAX_VERTICES) return 0;
    for (int u = 0; u < count; u++) {
        for (int v = 0; v < count; v++) graph->edges[u][v] = 0;
    }
    graph->count = count;
    return 1;
}
```

Only the active square is initialized. Operations must never read inactive rows or columns.

**Complexity:** O(V²), with constant work for an empty graph. Buffer-capacity costs follow the assumptions above.

</details>

---

### Exercise 2: Validate a Vertex

Write `int graph_valid_vertex(const graph_t *graph, int vertex)` to return 1 for a valid vertex and 0 otherwise.

<details>
<summary>Show solution</summary>

```cpp
int graph_valid_vertex(const graph_t *graph, int vertex) {
    return vertex >= 0 && vertex < graph->count;
}
```

Check both bounds before indexing the matrix.

**Complexity:** O(1). Buffer-capacity costs follow the assumptions above.

</details>

---

### Exercise 3: Add an Edge

Write `int graph_add_edge(graph_t *graph, int u, int v)` using the validation helper. Return 1 if a new undirected edge is added; return 0 for invalid endpoints, a self-loop, or an existing edge. Leave the graph unchanged on failure.

<details>
<summary>Show solution</summary>

```cpp
int graph_add_edge(graph_t *graph, int u, int v) {
    if (!graph_valid_vertex(graph, u) || !graph_valid_vertex(graph, v) || u == v) return 0;
    if (graph->edges[u][v]) return 0;
    graph->edges[u][v] = 1;
    graph->edges[v][u] = 1;
    return 1;
}
```

Update both directions to preserve symmetry. The graph is simple, so repeated edges and self-loops are rejected.

**Complexity:** O(1). Buffer-capacity costs follow the assumptions above.

</details>

---

### Exercise 4: Check an Edge

Write `int graph_has_edge(const graph_t *graph, int u, int v)`. Return 1 if an edge exists and 0 if absent or either endpoint is invalid.

<details>
<summary>Show solution</summary>

```cpp
int graph_has_edge(const graph_t *graph, int u, int v) {
    if (!graph_valid_vertex(graph, u) || !graph_valid_vertex(graph, v)) return 0;
    return graph->edges[u][v] != 0;
}
```

Validation avoids accessing outside the active matrix. The result deliberately treats invalid endpoints as no edge.

**Complexity:** O(1). Buffer-capacity costs follow the assumptions above.

</details>

---

### Exercise 5: Remove an Edge

Write `int graph_remove_edge(graph_t *graph, int u, int v)` using `graph_has_edge`. Return 1 for a removed edge or 0 if no valid edge exists.

<details>
<summary>Show solution</summary>

```cpp
int graph_remove_edge(graph_t *graph, int u, int v) {
    if (!graph_has_edge(graph, u, v)) return 0;
    graph->edges[u][v] = 0;
    graph->edges[v][u] = 0;
    return 1;
}
```

Remove both directions. The endpoint vertices remain available for other edges.

**Complexity:** O(1). Buffer-capacity costs follow the assumptions above.

</details>

---

### Exercise 6: Count Neighbors

Write `int graph_degree(const graph_t *graph, int vertex)` to return a vertex's degree, or -1 for an invalid vertex.

<details>
<summary>Show solution</summary>

```cpp
int graph_degree(const graph_t *graph, int vertex) {
    if (!graph_valid_vertex(graph, vertex)) return -1;
    int degree = 0;
    for (int v = 0; v < graph->count; v++) {
        if (graph->edges[vertex][v]) degree++;
    }
    return degree;
}
```

Count nonzero entries in the active row. Since self-loops are excluded, each entry corresponds to one incident edge.

**Complexity:** O(V). Buffer-capacity costs follow the assumptions above.

</details>

---

### Exercise 7: Find Distances with BFS

Write `int graph_bfs(const graph_t *graph, int start, int distance[])`. The output array must have at least count slots. Return 0 for an invalid start without changing output. Otherwise return 1, recording the fewest edges from start to each reachable vertex and -1 for unreachable vertices.

<details>
<summary>Show solution</summary>

```cpp
int graph_bfs(const graph_t *graph, int start, int distance[]) {
    if (!graph_valid_vertex(graph, start)) return 0;
    for (int v = 0; v < graph->count; v++) distance[v] = -1;
    int queue[MAX_VERTICES];
    int front = 0;
    int rear = 0;
    distance[start] = 0;
    queue[rear++] = start;
    while (front < rear) {
        int u = queue[front++];
        for (int v = 0; v < graph->count; v++) {
            if (graph->edges[u][v] && distance[v] == -1) {
                distance[v] = distance[u] + 1;
                queue[rear++] = v;
            }
        }
    }
    return 1;
}
```

The distance array doubles as visited state. Set a distance before enqueueing, so each vertex enters the queue at most once. Therefore this one-pass queue needs no wraparound and at most V slots.

**Complexity:** O(V²) worst case for the matrix; O(V) auxiliary space conceptually. Buffer-capacity costs follow the assumptions above.

</details>

---

### Exercise 8: Visit Reachable Vertices with DFS

Write `void dfs_visit(const graph_t *graph, int vertex, int visited[])` and `int graph_dfs(const graph_t *graph, int start, int visited[])`. The public function returns 0 for an invalid start without changing output. Otherwise, clear visited, mark all reachable vertices, and return 1. The helper assumes a valid vertex and initialized visited state.

<details>
<summary>Show solution</summary>

```cpp
void dfs_visit(const graph_t *graph, int vertex, int visited[]) {
    visited[vertex] = 1;
    for (int v = 0; v < graph->count; v++) {
        if (graph->edges[vertex][v] && !visited[v]) {
            dfs_visit(graph, v, visited);
        }
    }
}

int graph_dfs(const graph_t *graph, int start, int visited[]) {
    if (!graph_valid_vertex(graph, start)) return 0;
    for (int v = 0; v < graph->count; v++) visited[v] = 0;
    dfs_visit(graph, start, visited);
    return 1;
}
```

Mark before following neighbors to avoid cycling back into an active call. The recursion explores one path before returning to try another.

**Complexity:** O(V²) worst case for the matrix; O(V) recursive stack space. Buffer-capacity costs follow the assumptions above.

</details>

---

### Exercise 9: Count Connected Components

Write `int graph_components(const graph_t *graph)` using `dfs_visit`. Return zero for an empty graph. Count isolated vertices as individual components.

<details>
<summary>Show solution</summary>

```cpp
int graph_components(const graph_t *graph) {
    int visited[MAX_VERTICES] = {0};
    int components = 0;
    for (int v = 0; v < graph->count; v++) {
        if (!visited[v]) {
            components++;
            dfs_visit(graph, v, visited);
        }
    }
    return components;
}
```

Keep one visited array across all searches. Each unvisited start belongs to a component no earlier search reached. This reasoning uses undirected edges.

**Complexity:** O(V²) for the matrix; O(V) visited and recursion space. Buffer-capacity costs follow the assumptions above.

</details>

---

### Exercise 10: Clear All Edges

Write `void graph_clear_edges(graph_t *graph)` to remove every edge while retaining the vertex set.

<details>
<summary>Show solution</summary>

```cpp
void graph_clear_edges(graph_t *graph) {
    for (int u = 0; u < graph->count; u++) {
        for (int v = 0; v < graph->count; v++) graph->edges[u][v] = 0;
    }
}
```

The result has V isolated vertices, not zero vertices. For an empty graph, no slots are accessed.

**Complexity:** O(V²), with constant work for an empty graph. Buffer-capacity costs follow the assumptions above.

</details>

## Check the Completed Graph

Initialize five vertices and add edges (0, 1), (0, 2), (1, 2), and (1, 3). Vertex 4 is isolated. BFS from 0 should produce `[0, 1, 1, 2, -1]`; DFS should mark `[1, 1, 1, 1, 0]`; the component count should be 2.

Check an empty graph, one isolated vertex, cycles, invalid indices, duplicate edges, self-loop rejection, symmetric removal, and a path reaching all vertices. After clearing edges, five active vertices should form five components. Failed initialization and invalid traversal starts must leave existing state or outputs unchanged.

[Back to Graphs](../README.md)
