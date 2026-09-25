# Hash Tables in C++

Read the [theory](README.md) first, then work through the language examples and exercises.

## Use `std::unordered_map`

Include `<unordered_map>` for a hash-based key-value container. Keys are unique, and iteration order is unspecified. The container manages allocation and cleanup.

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<int, int> scores;
    scores[10] = 80; // Insert
    scores[10] = 95; // Update
    auto found = scores.find(10);
    if (found != scores.end()) {
        std::cout << found->second << "\n"; // 95
    }
    std::cout << scores.size() << "\n"; // 1
    scores.erase(10);
    scores.clear();
}
```

`find` checks for a key without inserting it. `operator[]` inserts a missing key with a value-initialized value (zero for `int`), so use `find` for a read-only lookup. `at(key)` reads an existing key and throws `std::out_of_range` if absent. `erase(key)` returns how many entries were removed. `empty()` checks whether any entries remain.

Lookup, insertion, and removal take O(1) on average under suitable hashing, but O(n) in the worst case. Rehashing can invalidate iterators; obtain them again after operations that may rehash. For keys without associated values, use `std::unordered_set` from `<unordered_set>`.

## Basic Exercises

Implement an integer-key, integer-value map using separate chaining. Try each exercise before opening its solution. Combine the shared definition and solutions in order; later exercises name their helper dependencies.

```cpp
#include <cstddef>
#include <new>

constexpr unsigned int BUCKET_COUNT = 5;

struct entry_t {
    int key;
    int val;
    entry_t *next;
};

struct table_t {
    entry_t *buckets[BUCKET_COUNT];
    std::size_t size;
};
```

Use valid table pointers, initialize before other operations, and clear the table when finished. Do not make shallow copies of a live table: its bucket pointers own the allocated entries. Output pointers must refer to separate writable integers outside the table and its entries. Assume the entry count fits in the size type.

Let `n` be the entry count, `b` the bucket count, and `α = n / b`. This teaching table has five buckets and **does not resize**, so average chain lengths grow as entries accumulate. Expected constant-time performance cannot be assumed for arbitrarily large inputs. Allocation and release of one entry are treated as constant-time operations.

These C++17 examples use raw pointers to make chaining explicit. Allocate with `new (std::nothrow)`, which returns `nullptr` on allocation failure for these entries, and release with `delete`.

### Exercise 1: Initialize the Table

Write `void table_init(table_t *table)` to set every bucket to empty and the size to zero. Use it only on a new table or one that has already been cleared.

<details>
<summary>Show solution</summary>

```cpp
void table_init(table_t *table) {
    for (unsigned int i = 0; i < BUCKET_COUNT; i++) {
        table->buckets[i] = nullptr;
    }
    table->size = 0;
}
```

Initializing a live table this way would lose its entry pointers without releasing them. Clear it first when reusing allocated entries.

**Time complexity:** O(b), where b is the bucket count.

**Auxiliary space:** `O(1)`, including at most one newly allocated entry for insertion.

</details>

---

### Exercise 2: Choose a Bucket

Write `unsigned int bucket_index(int key)` to return an index from 0 through `BUCKET_COUNT - 1`, including for negative keys. Convert the key to `unsigned int` before taking the remainder.

<details>
<summary>Show solution</summary>

```cpp
unsigned int bucket_index(int key) {
    return static_cast<unsigned int>(key) % BUCKET_COUNT;
}
```

Unsigned conversion is defined for negative values and avoids a negative array index. Do not use abs(key), which cannot represent the positive counterpart of the most negative int. This simple teaching hash is not intended to distribute all key patterns well.

**Time complexity:** O(1).

**Auxiliary space:** `O(1)`, including at most one newly allocated entry for insertion.

</details>

---

### Exercise 3: Look Up a Value

Write `int table_get(const table_t *table, int key, int *out_val)` to find a key. Return 1 and store its value on success, or return 0 and leave the output unchanged if absent. Use `bucket_index`.

<details>
<summary>Show solution</summary>

```cpp
int table_get(const table_t *table, int key, int *out_val) {
    unsigned int index = bucket_index(key);
    const entry_t *entry = table->buckets[index];
    while (entry != nullptr) {
        if (entry->key == key) {
            *out_val = entry->val;
            return 1;
        }
        entry = entry->next;
    }
    return 0;
}
```

Different keys can share a bucket, so compare each actual key. A value of -1 remains valid data because status is returned separately.

**Time complexity:** O(1 + α) expected with well-distributed keys; O(n) worst case.

**Auxiliary space:** `O(1)`, including at most one newly allocated entry for insertion.

</details>

---

### Exercise 4: Insert or Update

Write `int table_put(table_t *table, int key, int val)`. Update an existing key or create a new entry at the front of its bucket. Return 1 on success or 0 on allocation failure. Leave the table unchanged on failure. Use `bucket_index`.

Example: put (12, 85), then (17, 90), then (12, 95). The table should contain two entries, with key 12 mapped to 95.

<details>
<summary>Show solution</summary>

```cpp
int table_put(table_t *table, int key, int val) {
    unsigned int index = bucket_index(key);
    for (entry_t *entry = table->buckets[index]; entry != nullptr; entry = entry->next) {
        if (entry->key == key) {
            entry->val = val;
            return 1;
        }
    }
    entry_t *entry = new (std::nothrow) entry_t;
    if (entry == nullptr) {
        return 0;
    }
    entry->key = key;
    entry->val = val;
    entry->next = table->buckets[index];
    table->buckets[index] = entry;
    table->size++;
    return 1;
}
```

Search before allocating so updates neither create duplicate keys nor change the entry count. Finish allocating and initializing the new entry before linking it into the table.

**Time complexity:** O(1 + α) expected with well-distributed keys; O(n) worst case.

**Auxiliary space:** `O(1)`, including at most one newly allocated entry for insertion.

</details>

---

### Exercise 5: Check Membership

Write `int table_contains(const table_t *table, int key)` using `table_get` from Exercise 3. Return 1 if present or 0 if absent.

<details>
<summary>Show solution</summary>

```cpp
int table_contains(const table_t *table, int key) {
    int val;
    return table_get(table, key, &val);
}
```

Membership depends on whether lookup succeeds, not on whether the stored value is nonzero. The local output is never read on failure.

**Time complexity:** O(1 + α) expected with well-distributed keys; O(n) worst case.

**Auxiliary space:** `O(1)`, including at most one newly allocated entry for insertion.

</details>

---

### Exercise 6: Delete a Key

Write `int table_remove(table_t *table, int key)` to unlink and release a matching entry. Return 1 on removal or 0 if absent. Other entries in the bucket must remain reachable. Use `bucket_index`.

<details>
<summary>Show solution</summary>

```cpp
int table_remove(table_t *table, int key) {
    unsigned int index = bucket_index(key);
    entry_t *previous = nullptr;
    entry_t *entry = table->buckets[index];
    while (entry != nullptr && entry->key != key) {
        previous = entry;
        entry = entry->next;
    }
    if (entry == nullptr) {
        return 0;
    }
    if (previous == nullptr) {
        table->buckets[index] = entry->next;
    } else {
        previous->next = entry->next;
    }
    delete entry;
    table->size--;
    return 1;
}
```

A match at the bucket head needs a bucket update. Other matches need a predecessor link update. Decrease size only after finding and removing an entry.

**Time complexity:** O(1 + α) expected with well-distributed keys; O(n) worst case.

**Auxiliary space:** `O(1)`, including at most one newly allocated entry for insertion.

</details>

---

### Exercise 7: Read the Size

Write `std::size_t table_size(const table_t *table)` to return the stored entry count.

<details>
<summary>Show solution</summary>

```cpp
std::size_t table_size(const table_t *table) {
    return table->size;
}
```

The count increases for new keys and decreases for successful removals. Updating a value does not change it.

**Time complexity:** O(1).

**Auxiliary space:** `O(1)`, including at most one newly allocated entry for insertion.

</details>

---

### Exercise 8: Clear the Table

Write `void table_clear(table_t *table)` to release every entry, reset bucket heads, and set size to zero. It should also work on an empty initialized table.

<details>
<summary>Show solution</summary>

```cpp
void table_clear(table_t *table) {
    for (unsigned int i = 0; i < BUCKET_COUNT; i++) {
        entry_t *entry = table->buckets[i];
        while (entry != nullptr) {
            entry_t *next = entry->next;
            delete entry;
            entry = next;
        }
        table->buckets[i] = nullptr;
    }
    table->size = 0;
}
```

Save the next pointer before releasing the current entry. The embedded bucket array stays available for reuse.

**Time complexity:** O(n + b), visiting every entry and bucket.

**Auxiliary space:** `O(1)`, including at most one newly allocated entry for insertion.

</details>

---

### Exercise 9: Trace a Collision Chain

Starting from an empty table, put (12, 85), (17, 90), and (22, 70). Then update key 17 to 95 and remove key 17. What does bucket 2 contain, and what is the size?

<details>
<summary>Show solution</summary>

All three keys have remainder 2 when divided by 5. New entries go at the front:

```text
After inserts:  [22:70] → [17:90] → [12:85]
After update:   [22:70] → [17:95] → [12:85]
After removal:  [22:70] ─────────→ [12:85]
Final size:     2
```

Updating does not change size. Removing the middle entry reconnects its predecessor to its successor. Each search can inspect the whole chain, giving O(n) worst-case time and O(1) auxiliary space.

</details>

## Check the Completed Table

Check colliding keys, updates without size changes, head/middle/tail removal within a chain, missing keys, negative keys, stored values of zero and -1, and reuse after clearing. A failed allocation must leave existing entries and size unchanged; updating an existing key should not allocate.

[Back to Hash Tables](README.md)
