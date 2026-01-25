# B-Trees (The King of Indexes)

**What are B-Trees?**: B-Trees can be defined as a sorted <key-value> pair index, but with a different design altogether. This design helps in efficient key lookups.

**B-Tree Design**: In a B-Tree, <key-value> pairs are organized into pages or blocks. This is somewhat similar to how our disks work—if a disk needs to read one character, it still returns the whole page or block.

Similarly, each block contains n <key-value> pairs, where the value might be the actual value or a reference to another block on disk, and n represents the branching factor of the tree.

Normally, the value we are looking for is present at the leaf node of the B-tree. Traversing an N-level deep B-tree normally takes O(Log N) time, but there is a secret to achieving this: **we need to always keep the tree "Balanced"**.

## How Reads are performed in a B-Tree.

One Page is designated as the root of the B-Tree, and we start there and follow the references, until we reach the value.

**How it works (The Branching Factor)**

Because one node is as large as a disk block (say 4KB), it can hold a lot of information. Inside one 4KB page, you can store hundreds of references to other pages.
**Why this is a "Cheat Code" for speed:** If every node has 500 children, a tree with only 4 levels can store:

        500 X 500 X 500 X 500 = 62,500,000,000 records

Because the tree is "flat and wide," you only need to jump to 4 different spots on the disk to find one specific record among billions.

## How Updates work in B-Trees.

In B-Trees, we perform an in-place update. All we need to do is find the value that needs to be updated, seek to that block, update the value in place, and leave all other references unchanged.

## How Insertion works in B-Trees.

Insertion can be a heavy operation at times. Why? Consider a situation where our 4KB blocks are full—we will then have to split that block into two, insert the new value, and update the parent as well, since B-Trees need to remain balanced.

## Breaking B-Trees VIA RUM

1. B-Trees are read-optimized because of the flat and wide tree structure. In just a few hops, we can reach what we are looking for, so we have the **R**.
2. Memory-optimized, as we only have a single copy of the data. We perform in-place updates, which is memory-efficient, so we have the **M**.
3. We sacrifice the **U**, as we must perform in-place updates. Also, if the value becomes larger, we might have to split the pages, which becomes an overhead. So we sacrifice the **U**.

## Maintaining B-Trees

1. **WAL Importance**: Write-Ahead Logging returns with B-Tree as well. Why? Consider an update operation where we might end up changing multiple nodes. If a failure happens between updates, we might end up with corrupted data. To avoid this, WAL is required to be maintained.

2. **Concurrency Control**: Because we perform in-place updates, there can be scenarios where multiple threads try to read the same values that might be getting updated. We need to protect them using locks, something known as latches.

## Real-Life Use Cases of B-Trees

1. All relational databases, obviously.
2. File systems—our operating systems arrange files and folders in B-Tree indexes.
3. Transactional systems, where consistency is a priority, use B-trees extensively.
