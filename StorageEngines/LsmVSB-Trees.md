# LSM Tree vs B-Trees

For B-Trees and LSM-Trees, I believe a direct comparison doesn't make sense. Both have their own pros and cons, and ideally we should test them and make them work according to our workloads.

## What LSM trees are good for:

1. LSM trees are generally good for writes because of sequential writes, eliminating the need for random I/O.
2. LSM trees offer predictable write latency. However, they trade write amplification (multiple writes during compaction) for reduced random I/O during inserts.

## What LSM trees struggle with:

Compaction is the necessary evil for LSM trees. Although compaction runs in the background, it competes for disk I/O and CPU to merge segment files. This can affect write throughput and reads as well, since a key might exist in multiple unmerged segment files, requiring multiple lookups. Bloom filters help mitigate this read cost, but the storage engine needs to be carefully configured.

## What B-Trees are good for

1. Faster reads—in a well-balanced B-Tree, a read typically requires 3–4 disk hops.
2. Deterministic reads and consistency of values, since data is always in one place, ensuring you always get the latest result for a key.

## What B-Trees struggle with:

1. Consistency: B-Trees don't "struggle" with consistency because they are bad at it; they struggle because they promise more. They promise that the data is always in the right place. To keep that promise, they have to use heavy tools like WALs and Latches.

2. Write amplification— B-Trees write values to the WAL, then to the page. When a page splits, additional writes occur, creating multiple I/O operations per insert.

In a nutshell while choosing the correct database, for starters we can consider this table:

| Workload                   | Choose   | Why?                                                           |
| -------------------------- | -------- | -------------------------------------------------------------- |
| Logging / Sensors / IoT    | LSM-Tree | Constant, high-volume writes; reads are rare.                  |
| E-commerce / User Profiles | B-Tree   | Data is read many times; updates are occasional.               |
| Financial Ledger           | B-Tree   | Requires strict "In-Place" consistency and fast range scans.   |
| Big Data Search            | LSM-Tree | High throughput ingestion; Bloom filters handle the read cost. |

No more thoughts at this moment.. :P
