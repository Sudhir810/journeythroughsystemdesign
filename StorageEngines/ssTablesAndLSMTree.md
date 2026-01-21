# SSTables and LSM Trees

**What are SSTables?** Sorted string tables are the segment files we defined in the world's simplest database, but with keys stored in sorted order.

## What is LSM-Trees

There are four key actors in an LSM tree:

1. **MEMTABLE**: An in-memory data structure (like a Red-Black tree or AVL tree) that stores all keys in sorted order. All incoming writes are first added to the MEMTABLE.

2. **SSTable**: Segment files created when the MEMTABLE exceeds a threshold (e.g., a few megabytes). At that point, all MEMTABLE data is written to disk as SSTables.

3. **WAL (Write Ahead Logs)**: A separate log on disk to which every write is appended. This prevents data loss if the database crashes, allowing the MEMTABLE to be restored after recovery.

4. **Compaction**: Merging multiple sorted segment files together, similar to a merge sort operation. When a key appears in multiple segment files, the value from the most recent segment file is retained.

Combining these four actors creates an **LSM (Log-Structured Merge) tree**.

## How Reads Work

To find a value in an LSM tree, the search proceeds as follows:

1. Search in the MEMTABLE.
2. Search in the most recent SSTable.
3. Continue to older SSTables until found.

## How Writes Work

All four actors work together in the write process:

- New writes go to the MEMTABLE.
- When MEMTABLE exceeds threshold, it's flushed to disk as an SSTable.
- WAL ensures no data loss on crash.
- Compaction merges SSTables in the background.

## Why LSM Trees Solve Previous Issues

1. **Memory Efficiency**: Since keys are sorted in segment files, the hash index doesn't need to store all keys in memory—a sparse index is sufficient.
   Sparse Index Analogy
   Imagine a huge phonebook with 1 million names.

   Instead of creating an index for every single name (too much memory), you create a sparse index:

   Page 1: Names A–D (index entry: "A starts at byte 0")
   Page 2: Names E–H (index entry: "E starts at byte 4096")
   Page 3: Names I–L (index entry: "I starts at byte 8192")
   And so on...
   When you search for "George":

   Check the sparse index: "G is between E–H, start at byte 4096"
   Jump to byte 4096
   Scan forward linearly until you find "George"
   Why it works:
   You only stored 4 index entries (A, E, I, M...) instead of 1 million
   You still found "George" fast by jumping to the right page
   You scan only one small section, not the entire book

2. **Range Query Support**: With data stored in sorted order, range queries are now efficiently supported.

## Why are writes faster in LSM tress

1. **No updates in place**: Because the database never goes looking for an old record to overwrite it, it eliminates Random I/O. Disk heads don't have to seek, and SSDs don't have to perform complex read-modify-write cycles immediately.

2. **Memtable**: Writes to the MemTable (usually a Skip List or Red-Black Tree) indeed have a time complexity of $O(logN), where N is the number of elements currently in memory. Since the MemTable is kept relatively small (e.g., 64MB or 128MB), N is small making this operation incredibly fast—essentially "memory speed."

3. **Compaction (The good or Evil)**: Compaction is a background process that merges old files to keep Reads fast and save disk space.
   but compaction can use resources that could otherwise be used for writes. Compaction involves reading data from disk and writing it back again in a merged format, it creates Write Amplification. 1KB of user data might end up being written to disk 10–20 times over its lifetime due to multiple levels of compaction.If the background compaction can't keep up with the incoming writes, the database will actually throttle or "stall" your writes to prevent the system from being overwhelmed by too many un-merged files.

| Feature     | Impact on Writes | Why?                                                    |
| ----------- | ---------------- | ------------------------------------------------------- |
| Append-only | Positive         | Converts random I/O into sequential I/O.                |
| MemTable    | Positive         | Absorbs writes at memory speed (O(logN)).               |
| Compaction  | Negative         | Competes for I/O bandwidth; can cause "Write Stalls."   |
| WAL (Log)   | Neutral          | Adds a small overhead for durability but is sequential. |

**In short**, The design makes writes fast by deferring the hard work (sorting and merging) to the background. However, that background work (compaction) eventually has to be paid for in I/O cycles.

## RUM conjecture

I just want to do a sneak peek around RUM conjecture which RUM stands for Reads, Updates, Memory and every storage engine must balance these tradeoffs. We can optimise only two of the three costs mentioned.

For example in case of LSM-trees:
To keep Updates fast, you append data anywhere there is space.
To keep Memory usage low, avoid building a massive index that points to every single row's location. (Sparse Index)

**The Result**: When you want to read a specific key, the database doesn't know exactly where it is. It has to look through multiple files (SSTables) on disk.

**The Cost**: A single read might turn into 5, 10, or 50 disk I/Os. This is known as Read Amplification.

## LSM-Tree Optimisation

Compaction is a "necessary maintenance cost" that LSM trees pay to keep the other parts of the system fast. Here is how the LSM tree helps by managing this "evil" efficiently:

## 1. It Decouples "Writing" from "Sorting"

In an LSM tree, the "evil" compaction is **asynchronous**.

- **The "Help":** Your application gets a "Success" as soon as the data hits the MemTable and WAL. The hard work of merging files happens later in the background when the system might have idle I/O cycles.

## 2. Choosing the "Lesser Evil" (Compaction Strategies)

Modern LSM trees (like RocksDB or Cassandra) allow you to tune your compaction to match your workload. You can choose how much "evil" you want to tolerate:

| Strategy          | How it works                                                     | Impact on Writes                                | Impact on Reads                                            |
| ----------------- | ---------------------------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------- |
| **Tiered (STCS)** | Merges files of similar size into new files.                     | **Best for Writes:** Lower write amplification. | **Slowest:** Reads might check many files.                 |
| **Leveled (LCS)** | Keeps levels of fixed size; merges one file into the next level. | **High Work:** High write amplification.        | **Fastest:** Limits the number of files a read must check. |

## 4. Reducing the Read Burden (Bloom Filters)

**The "Help":** A Bloom Filter is a tiny bit of memory that can tell the system, _"This key definitely does NOT exist in this file."_ This allows the database to skip 99% of the files on disk without even touching them, which means we can afford to let compaction run more slowly or less frequently.

### Summary: The Strategic Debt

Think of an LSM tree like a credit card:

- **The Write** is the purchase (it's instant and easy).
- **Compaction** is the bill (you have to pay it eventually).
- **The Benefit** is that you can "buy" (write) a huge amount of data during a peak burst (like Black Friday) and "pay the bill" (compact) during the night when the system is quiet. A B-Tree forces you to pay in cash for every single item, which is safer but much slower during a rush.

## Real life use cases of LSM-Trees

1. Social Media Timelines (Facebook, Instagram)
   When you "Like" a post, share a photo, or write a comment, that data needs to be recorded instantly for billions of users.

The Database: RocksDB (developed by Facebook).

2. Search Engines (Google, Elasticsearch)
   While search engines use "Inverted Indexes" to find words, the way they store those indexes on disk often follows LSM principles.

The Technology: Lucene (the engine inside Elasticsearch and Solr).

Why LSM? As Google crawls the web, it needs to update trillions of "Posting Lists" (lists of which pages contain a certain word). LSM trees allow them to batch these massive updates and merge them in the background.

3. Messaging & Activity Logs
   Every time you send a message on a platform like Discord or Slack, it is appended to a log.

The Database: Often RocksDB or LevelDB.
