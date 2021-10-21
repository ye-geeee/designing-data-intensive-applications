# Chapter 3. Storage and Retrieval

1. [Data Structures That Power Your Database](#Data-Structures-That-Power-Your-Database)
2. [Hash Indexes](#Hash-Indexes)
3. [SSTables and LSM-Trees]
4. [B-Trees]
5. [Comparing B-Trees and LSM-Trees]
6. [Other Indexing Structures]
7. [Transaction Processing or Analytics?]
8. [Data Warehousing]
9. [Stars and Snowflakes: Schemas for Analytics]
10. [Column-Oriented Storage]
11. [Column Compression]
12. [Set Order in Column Storage]
13. [Writing to Column-Oriented Storage]
14. [Aggregation: Data Cubes and Materialized Views]

<br/>

To tune a storage engine to perform well on your workload, 
you need to have a rough idea of what the storage engine is doing under the hood.  

In particular, there is a big difference between storages engines  
1. which are optimized for transactional workloads
2. which are optimized for analytics.  

Let's examine two families of :star2: _log-structured_ storage engines, and :star2: _page-oriented_ storage engines such as B-trees.  

## Data Structures That Power Your Database

Many databases internally use a :star2: _log_ which is an append-only data file to deal with mores issues
(concurrency control, reclaiming disk space, handling errors, partially written records).  
In addition, in order to efficiently find the value for a particular key in the database, we need a difference data structure: :star2: _index_
Therefore, the general idea behind them is 1. to keep some additional metadata on the side and 2. if you want to search, you may need several difference indexes on different parts of the data.  

An index is an _additional structure_.  
Many data-bases allow you to add and remove indexes, and it should not affect the contents of the databases; it only affects performance.  
Any kind of indexes usually slows down writes, because the index need to be updated everytime when data is written.  

Therefore, :heavy_exclamation_mark: **well-chosen indexes speed up read queries, but every index slow down writes**.  
For this reason, databases don't usually index everything by default, you can choose the indexes that give your application the greatest benefit, 
without introducing more overhead than necessary.  

<br/>

## Hash Indexes

Let's start with indexes for key-value data which is similar to the _dictionary_ type.  
The simplest possible indexing strategy is this: keep an in-memory hash map where every key is mapped to a byte offset in the data file.  
This may sound simplistic, but it is a viable approach.  

In this case, [Bitcask](https://en.wikipedia.org/wiki/Bitcask) offers high-performance reads and writes.  
The values can be loaded from disk to memory with just one disk seek.  
A storage engine like Bitcask is well suited to situations where the value for each key is updated frequently.  

But, how do we avoid eventually running out of disk space?
A good solution is to break the log into segments of a certain size by closing a segment file.  
We can perform _compaction_ on these segments like below image.  

![03_compaction](../resources/part1/03_compaction.png)

Compaction makes segments much smaller.  
We can also merge several segments together at the same time, and it could be done in a background thread.  

Each segment now has its own in-memory hash table, mapping keys to file offsets.  
In order to find the key, we check from the latest.   
The merging process keeps the number of segments small, so lookups don't need to check many hash maps.  

**Issues that are important in a real implementation**  

- _File format_: binary format is faster than csv format
- _Deleting records_: append a special deletion record to the data file
- _Crash recovery_: in-memory hash maps are lost when restarted. use a snapshot of each segment's hash map on disk  
- _Partially written records_: include checksums allowing corrupted parts of the log to be detected and ignored
- _Concurrency control_: Data file segments are append-only and otherwise immutable

**Why append-only design is good**

- Appending and segment merging are sequential write operations - much faster than random writes  
- Concurrency and crash recovery are much simpler  

**Limitation of Hash Tables**

- must fit in memory
- range queries are not efficient




