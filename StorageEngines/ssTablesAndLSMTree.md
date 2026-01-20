# SSTables and LSM Trees

**What are SSTables?** Sorted string tables, the segment files that we defined in worlds simplest database, same files but with keys stored in sorted order.

**How does SSTables help?** Sorted string tables basically solves two issues we encountered with the segment files and hash index. - They help in Range queries - They reduce the number of keys to be stored in the memory in hash table.
HOWww ? lets see..

**How SSTables Work**

There are different actors behind

1. **MEMTABLE**: What is memtable here, this acts as a in in memory storage which basically store all the keys in sorted order so that for example Red-Black tree or AVL trees. All the incoming writes are first added to MEMTABLE.

2. **SSTable**: Segment files, these are created once our in memory data structure is bigger than a threshold for example a few megabytes then all the data is written to SSTables.

3. **WAL: Write Ahead Logs**: If the database crashes we might lose all the recent writes, to avoid this problem we maintain a separate log on disk, to which every write is appended, its only purpose is to restore memtable after a crash.

4. **Compaction**: As we discussed previously, compaction is basically merging multiple segment files together, here as well we will be merging sorted segment files together, this process here will be similar to mergesort. The situation where a key appears in several segment files we take the value of the most recent segment file for that particular key.

Combining all above 4 actors result in our picture which is LSM trees.

**How Reads work here?**: So in order to find one value in LSM tree first the value is searched in memtable, then in the most recent segment file all the way to the oldest segment file.

**How Write work here?**: All above actors are at play for the same....

What else ...

Right how **LSM TREES** here help in resolving previous two issues...

1. As the keys are sorted in order in segment files, the hash index that we defined previously does not require to store all the keys in the memory, a sparse index will do.

2. Since the data is stored in sorted order, we can perform range queries as well.
