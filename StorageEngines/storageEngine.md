# Storage Engines

![Storage Engine Diagram](./images/storageEngine.jpg)

**What is this**: Storage engines are nothing but databases that we use for our applications blindly. Whether NoSQL or SQL relational databases, all come under storage engines.

**Why I need to know**: For interviews? That is one part, yes… but to make sure we make the best choice possible for our workload. For example, finding the best storage engine for write operations vs. read operations.

## Starting from the World's Simplest Database

A simple database is basically an append-only file consisting of key-value pairs of data.

**Writes** here are faster. Why? Because of sequential writes. Even for an updated value for a key, all we do is append to the end of the file. This helps in reducing random I/O; the disk head does not have to move to a correct location to update.

**Reads**: Obviously these are slower. The tradeoff? Because to search for the recent value, you will have to search for the key from the end of the file to the top of the file.

Now, a possible solution for the slow reads?

The solution is we will have to define an index.

**What is an index?** An index is an additional structure that contains some metadata of the original data which helps to fasten the lookup/data retrieval.

The simplest example would be a table of contents in a book that points us directly to where a topic or chapter can be found.

## The Hash Index

So far so good. We need to define a simple index for our log file similar to a table of contents, which is nothing but a Hash Based Index.

**What is a Hash Index**? A hash-based index is nothing but a key-value pair based on our database, where the key represents the data key and the value is the offset where the key can be located in the file.

**Why does this help**? Now, to look up a key in the file, we can directly check the key in the hash table and jump directly to the offset in the file.

## The Problem: Limited Memory

Okay, so now reads are faster. But… yes, all keys in the file must be present in the hash table. This means they must be present in memory, which means we have a problem here as we have limited memory RAM.

**Random example**: Imagine you are storing the location of every message ever sent in a massive chat app. You want to know where each message is on the disk so you can retrieve it instantly.

### The Constraints

- Total Messages: 100 Billion
- Key (MessageID): Let's use a standard UUID or a long integer. Even at its smallest, it's about 16 bytes
- Value (Disk Pointer): To point to a location in a multi-petabyte storage system, you need a 64-bit offset: 8 bytes
- Total per Key-Value pair: 24 bytes

### The Math: "Cost of RAM" Calculation

If you try to keep every single message’s location in a Hash Index (RAM):

100,000,000,000 X 24 bytes = 2,400,000,000,000 bytes

Convert to Terabytes: 2.18 TB RAM (approx)

## Range Queries

Another thing to consider: what if we need to search for all messages between 01/2026 to 12/2026? With a hash map in place, we can't do that easily since range queries aren't possible with hash maps. We'd have to iterate through every key and check if it falls within that range.

## Compaction

What happens when the data on disk gets huge? Data on disk may contain all keys with potential duplicates. We can address this through compaction.

**Compaction** is the process of merging all keys together, keeping only the latest value for each key. This results in smaller segment files. As compaction reduces segment sizes, we can merge several segments together as well.

This process can run entirely in the background without interrupting incoming writes or reads. When the file is compacted, the coming reads can be redirected to this file.

## Conclusion

I think this is pretty much it from simple databases and hash indexes. We have seen what they are, why they are, what they are good at, and their issues.

But if Hash Indexes hit a memory wall and can’t handle range queries, how do giant databases like Cassandra or BigTable handle billions of rows so easily? In the next part, we’ll discuss SSTables and LSM-Trees to find out.
