# Hash Tables

A **hash table** stores values under keys so we can look up a value without searching every entry.

For example, a table might associate student IDs with scores:

```text
Key (student ID)    Value (score)
12                 85
27                 91
34                 76
```

Looking up key 27 returns 91. A key identifies an entry; a value is the information stored under it. In a map, each key has one associated value, so inserting an existing key updates that value.

A hash table can implement a **map**, which stores key–value pairs, or a **set**, which stores keys to answer membership questions.

## From Key to Bucket

A **hash function** turns a key into a hash value. The table uses that value to select a position in an array of **buckets**.

A simple teaching example for nonnegative integer keys is:

```text
bucket index = key % bucket_count
```

With 5 buckets:

```text
12 % 5 = 2
27 % 5 = 2
34 % 5 = 4
```

The `%` operator gives the remainder. This rule is useful for understanding the mechanics, but a real hash function should distribute the expected keys well. Keys with repeated numerical patterns can make this simple rule perform poorly.

A hash is not a unique ID. Different keys can produce the same hash value or bucket index. The table must compare actual keys before deciding that it has found a match.

Equal keys must produce equal hashes, and a stored key must not change in a way that changes its hash or equality behavior. Otherwise, a later lookup may search the wrong bucket.

## Collisions

A **collision** occurs when different keys map to the same bucket. In the example above, both 12 and 27 map to bucket 2.

Collisions are normal. They do not mean that one entry should overwrite the other. There are two common ways to handle them.

### Separate Chaining

Each bucket points to a collection of entries, often a linked list:

```text
Bucket 0: empty
Bucket 1: empty
Bucket 2: [27:91] → [12:85] → NULL
Bucket 3: empty
Bucket 4: [34:76] → NULL
```

To find key 12, choose bucket 2, then compare keys along its chain. Key 27 is different, so keep going until key 12 is found or the chain ends.

The bucket array has a fixed number of positions until resized, but each chain can contain multiple entries. A chaining table can hold more entries than buckets.

### Open Addressing

Entries are stored directly in the array. When a slot is occupied by a different key, follow a **probing** rule to find another slot.

With linear probing, try the next slot and wrap around when necessary:

```text
Insert key 12: home slot 2 is empty → store at 2
Insert key 27: home slot 2 is occupied → try 3
```

Lookup must follow the same probing rule. Deletion cannot simply mark a slot as never used: doing so could make a lookup stop before reaching a displaced entry. One approach uses a **tombstone**, a marker meaning “deleted, but keep searching.”

Open addressing needs available slots or a resize policy. As the table fills, probing usually becomes more expensive. A probe sequence must also be bounded so a missing-key lookup cannot loop forever in a full table.

## Basic Operations with Separate Chaining

The following examples use integer keys and values, 5 buckets, and linked lists for collisions.

### Insert a New Key

To insert `(17, 88)`:

1. Calculate `17 % 5 = 2`.
2. Search bucket 2 to check whether key 17 already exists.
3. If absent, allocate an entry and link it into the chain.

```text
Before: bucket 2 → [27:91] → [12:85] → NULL
After:  bucket 2 → [17:88] → [27:91] → [12:85] → NULL
```

Checking for an existing key prevents duplicate entries for the same key. If allocation fails, leave the table unchanged.

### Update an Existing Key

To store value 95 under key 27, search bucket 2 and replace the matching entry's value:

```text
Before: [17:88] → [27:91] → [12:85]
After:  [17:88] → [27:95] → [12:85]
```

The number of entries stays the same. A single operation called **put** or **insert-or-update** commonly handles both insertion and replacement.

### Look Up a Key

Calculate the bucket, then compare keys in that chain. Return the matching value if found, or report absence when the chain ends.

A value such as zero or -1 might be valid data. Keep the success status separate from the returned value so a missing key is unambiguous.

### Delete a Key

Find the entry and its predecessor, bypass it in the chain, then release it. Removing the first entry requires changing the bucket's head pointer.

```text
Delete key 27:
Before: [17:88] → [27:95] → [12:85] → NULL
After:  [17:88] ─────────→ [12:85] → NULL
```

The other colliding keys must remain reachable. If the key is absent, leave the table unchanged.

### Size and Clear

A stored entry count makes reading the size an `O(1)` operation. Increase it only for a new key, not an update; decrease it only after successful deletion.

Clearing a chained table releases every entry and resets every bucket to empty. With `n` entries and `b` buckets, this takes `O(n + b)` time.

## Load Factor

The **load factor** measures how many entries there are relative to buckets:

```text
load factor α = entry_count / bucket_count
```

With 10 entries and 5 buckets, the load factor is 2. For chaining, that means an average of 2 entries per bucket, though the actual distribution might be uneven.

A low load factor does not guarantee short chains: a poor hash function can put all entries in one bucket. For open addressing, the load factor cannot exceed 1, and useful performance generally requires spare slots.

## Resizing and Rehashing

To grow a chained table, allocate a larger bucket array and redistribute entries using the new bucket count. This is called **rehashing**.

```text
Key 27 with 5 buckets:   27 % 5 = 2
Key 27 with 10 buckets:  27 % 10 = 7
```

Simply copying bucket heads to the same indices would put entries in the wrong places for future lookups.

A geometric growth policy with a suitable load-factor threshold can keep expected operation costs small. A single resizing insertion can take linear work, while the expected cost per insertion across a sequence can remain amortized `O(1)`.

This depends on good key distribution and an appropriate resizing policy; it is not a guarantee for every input.

## Complexity Summary

For separate chaining, let `n` be the entry count, `b` the bucket count, and `α = n / b`. Assume constant-time hashing and equality checks, and constant-time allocation or release of one entry.

| Operation | Expected cost with well-distributed hashes | Worst case |
| --- | --- | --- |
| Lookup or membership check | O(1 + α) | O(n) |
| Insert or update, without resizing | O(1 + α) | O(n) |
| Delete | O(1 + α) | O(n) |
| Read stored size | O(1) | O(1) |
| Initialize bucket array | O(b) | O(b) |
| Clear | O(n + b) | O(n + b) |

If resizing keeps `α` bounded, expected lookup and deletion are `O(1)`, and insertion can be expected amortized `O(1)`. If all keys collide, searching the chain is still `O(n)`.

A chained table uses **O(n + b)** storage. Ordinary lookup, insertion, and deletion need `O(1)` auxiliary space beyond a newly allocated entry. Rehashing also needs storage for the new bucket array.

For strings, hashing typically takes time proportional to key length, and equality checks can also inspect multiple characters. The constant-time-key assumption should not be applied to arbitrarily long strings.

## Hash Tables and Other Structures

| Structure | Typical strength | Tradeoff |
| --- | --- | --- |
| Array | Direct access by numeric index | Searching for a value may require traversal |
| Linked list | Link changes at known nodes | Finding a key usually requires traversal |
| Hash table | Fast expected lookup by key | Collisions, memory overhead, no inherent sorted order |

Do not rely on hash-table iteration order unless the particular container explicitly guarantees it. Hash tables are also not naturally suited to finding all keys in a sorted range.

## When to Use Hash Tables

Use them for counting occurrences, checking whether a value has been seen, or associating identifiers with records. A set can detect duplicates; a map can associate each word with its frequency.

They are a good fit when key-based access matters more than ordering. If you need sorted traversal or range queries, an ordered structure may be more appropriate.

[Review Arrays](../01-arrays/README.md) · [Review Linked Lists](../02-linked-lists/README.md)
