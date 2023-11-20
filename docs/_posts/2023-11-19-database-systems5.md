---
layout: post
title:  "Database systems V： Data storage"
date:   2023-11-19 21:22 +1100
categories: database
layout: math
---
# Database systems V： Data storage

## Key points

- Record format
- Buffer management

## Disk-block access

- Smallest process unit is a block.
    > If a single record in a block is needed, the entire block is transferred.
- Data are transferred between disk and main memory in units of blocks.
- A relation is stored as a **file** on disk.
- A file is a sequence of blocks, where a block is a fixed-length storage unit.
- A block is also called a **page**.

## Buffer management

Manages traffic between disk and memory by maintaining a buffer pool in main memory.

**Buffer pool:**

Collection of page slots (frames) which can be filled with copies of disk block data.

> E.g., One page = 4096 Bytes = One block

![](/assets/images/buffer_pool.png){: width="500"}

### Block operations

For each frame, we need to know:
- whether it is currently in use
- whether it has been modified since loading (dirty bit)
- how many transactions are currently using it (pin count)
- (maybe) time-stamp for most recent access

**request_block**

- If block is already in buffer pool:
  - no need to read it again;
  - use the copy there (unless write-locked).
- If block is not in buffer pool yet:
  - need to read from hard disk into a free frame;
  - if no free frames, need to remove block using a buffer replacement policy.

**release_block**

- Indicates that block is no longer in use.
    > Good candidate for removal / replacing.
- Decrement pin count for specified page.
- No real effect until replacement required.

**write_block**

- Updates contents of page in pool.
- Set dirty bit on.
    > Note: Doesn’t actually write to disk, until been replaced, or forced to commit.

**force_block**

- "commits" by writing to disk.

## Cache performance

**Cache hits:** pages can be served by the cache.

**Cache misses:** pages have to be retrieved from the disk.

**Hit rate** = #cache hits / (#cache hits + #cache misses)

> Use cache hit rate to measure the performance of buffer policies.

## Buffer replacement policies

1. Least Recently Used (LRU)
   - release the frame that has not been used for the longest period.
   - intuitively appealing idea but can perform badly
2. Most Recently Used (MRU)
   - release the frame used most recently
3. First in First Out (FIFO)
   - need to maintain a queue of frames
   - enter tail of queue when read in
4. Random

**Sequential flooding**

- LRU: We need to get in/out every page.
    > This is called sequential flooding.
- MRU: performs the best in this case (repeated scan).

No replacement policy is guaranteed to be superior to the others. The choice often depends on specific applications and their requirements.

## Block / Page format

- **Block:** A block is a collection of slots.
- **Slot:** Each slot contains a record.
- **Record:** A record is identified by record_id: rid = \<page id, slot number\>.

## Record format

### Fixed-length

Each field has a fixed length as well as the number of fields.

![](/assets/images/fixed_length_record.png){: width="400"}

### Variable-length

Some field is of variable length.

Encoding schemes where attributes are *stored in order:*
- Option1: Prefix each field by length

  ![](/assets/images/prefix_var_length_record.png){: width="400"}
- Option 2: Terminate fields by delimiter
  > 33357462/Neil Young/Musician/0277/
- Option 3: Array of offsets

  ![](/assets/images/arr_offset_var_length_record.png){: width="400"}

Encoding schemes where attributes are *not stored in order:*

Fixed-length part followed by variable-length part.
- The fixed-length part is to tell where we can find the data if it is a variable-length data field.
- The variable-length part is to store the data.

Variable length attributes are represented by fixed size **(offset, length)** in the fixed-length part, and keep attribute values in the variable-length part.

Fixed length attributes store attribute values in the fixed-length part.

**Example:** a tuple of (ID, Name, DeptName, Salary) where the first three are variable length.

![](/assets/images/unordered_var_length_record.png){: width="600"}

## Slotted page

Introduce slot directory in footer.
- Pointer to free space
- (Length, Pointer to beginning of record)
- Grows in reverse order (from right to left)
- Record ID = location in slot table

![](/assets/images/slotted_page.png){: width="500"}

### Delete record

Set the corresponding slot directory pointer to null.

### Insert record

1. Place record in free space on page.
2. Create (length, pointer) pair in the next open slot in slot directory.
3. Update the free space pointer.

### Fragmentation

After several insertions and deletions, the free space is fragmented.

Modern RDBMS often do compaction when system is idle.

![](/assets/images/fragmented_free_space.png){: width="400"}

## Query optimisation

## Key points

- Index
- Query plan
- Join order selection

## Index

### B+ Tree

![](/assets/images/b_tree_index.png)

The leaves of the index contains a pointer to the data (single record).

You can build many such indexes on a file (different search keys) as the index is separated from the data.

The underlying file that contains the records may or not be sorted by key.

When unsorted, the arrows (i.e., the pointers to the data) cross each other, this is referred to as unclustered index option. (cf. clustered, on the right)

### Hash index

Index contains "buckets", each bucket contains the index data entries.

A hash function works on the search key and produces a number over the range of 0 ... M-1 (M is the number of buckets).
> e.g., h(K) = (a * K + b), where a, b are constant, K is the search key.

**Features:**

- Fast to search (i.e., no traversing of tree nodes)
- Best for equality searches, cannot support range searches.

![](/assets/images/hash_index.png){: width="400"}

### CREATE INDEX

Defining indexes (syntax):
```sql
CREATE INDEX index_name ON table_name (attr1, attr2, ... );
```
`CREATE INDEX` also allows us to specify
an access method (USING btree, hash, rtree, or gist)
```sql
CREATE INDEX idx_address_phone ON address USING hash (phone);
```

## Query Tuning: EXPLAIN

Select on indexed attribute: index scan

![](/assets/images/index_scan.png)

Select on non-indexed attribute: sequential scan

![](/assets/images/seq_scan.png)

## JOIN in PostgreSQL

| | Nested Loop Join | Hash Join | Merge Join |
| :-- | :-- | :-- | :-- |
| **Terms of use** | Any | Qui-join | Join sorted tables |
| **Sources** | CPU, Disk I/O | Memory, Temporary space | |
| **Specificities** | 1. Highly selective indexing (when there is a table with a smaller number of records, and the joining column is indexed in the second table).<br>2. Perform restricted searches (e.g. key loopup).| 1. When there is a lack of indexing or the indexing conditions are ambiguous.<br>2. When tables are large. | The most efficient join. Used when join columns are indexed in both tables. Excellent for very large tables. |
| **Disadvantages** | Not efficient when the index is missing, or the query conditions are not restrictive enough, or the number of records in the table is high. | Build a hash table in memory or a tempdb. Higher cost in terms of memory consumption and disk I/O utilization. Slower return of results the first time around.| |

## Transaction management

## Key points

- Transcation
- Concurrency control
- Recovery

## NoSQL

## Key points

- NoSQL concept
- Different data model
- Key-Value, Document, Column-family, Graph
