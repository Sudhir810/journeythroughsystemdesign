# SSTables and LSM Trees

**What are SSTables?** Sorted string tables are the segment files we defined in the world's simplest database, but with keys stored in sorted order.

**How do SSTables help?** They solve two issues we encountered with segment files and hash indexes:

- They enable range queries
- They reduce the number of keys that must be stored in the hash table memory

## How SSTables Work

There are four key actors in an LSM tree:

1. **MEMTABLE**: An in-memory data structure (like a Red-Black tree or AVL tree) that stores all keys in sorted order. All incoming writes are first added to the MEMTABLE.

2. **SSTable**: Segment files created when the MEMTABLE exceeds a threshold (e.g., a few megabytes). At that point, all MEMTABLE data is written to disk as SSTables.

3. **WAL (Write Ahead Logs)**: A separate log on disk to which every write is appended. This prevents data loss if the database crashes, allowing the MEMTABLE to be restored after recovery.

4. **Compaction**: Merging multiple sorted segment files together, similar to a merge sort operation. When a key appears in multiple segment files, the value from the most recent segment file is retained.

Combining these four actors creates an **LSM (Log-Structured Merge) tree**.

## How Reads Work

To find a value in an LSM tree, the search proceeds as follows:

1. Search in the MEMTABLE
2. Search in the most recent SSTable
3. Continue to older SSTables until found

## How Writes Work

All four actors work together in the write process:

- New writes go to the MEMTABLE
- When MEMTABLE exceeds threshold, it's flushed to disk as an SSTable
- WAL ensures no data loss on crash
- Compaction merges SSTables in the background

## Why LSM Trees Solve Previous Issues

1. **Memory Efficiency**: Since keys are sorted in segment files, the hash index doesn't need to store all keys in memory—a sparse index is sufficient.

2. **Range Query Support**: With data stored in sorted order, range queries are now efficiently supported.
