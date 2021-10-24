# Chapter 3. Storage and Retrieval

1. [Data Structures That Power Your Database](#Data-Structures-That-Power-Your-Database)
2. [Hash Indexes](#Hash-Indexes)
3. [SSTables and LSM-Trees](#SSTables-and-LSM-Trees)
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

<br/>

## SSTables and LSM Trees

Let's change out segment files: _sort by key_  
We call this format _Sorted String Table_, or _SSTable.  
We also require that each key only appears once within each merged segment file.  

**Advantages of SSTables**

1. Merging segments is simple and efficient - similar to _mergesort_
2. No longer need to keep an index of all the keys in memory to find particular word - it can be sparse  
3. Can group records into a block and compress it before writing it to disk  

### Constructing and maintaining SSTables

With tree data structures like red-black trees and AVL trees, you can insert keys in any order and read them back in sorted order.  

- Write data to an in-memory balanced tree data structure. This in-memory tree is sometimes called a _memtable_. 
- When the memtable gets bigger than some threshold - write it out to disk as and SSTable file.  
- To serve read request, first try to find the key in the memtable, most recent to the oldest one.  
- From time to time, run a merging and compaction process in the background.  

One problem from this scheme: if the databases crashes, the most recent writes are lost.  
In order to avoid that problem, we can keep a separate log on disk to which every write is immediately appended.  

### Making an LSM tree out of SSTables

Following algorithm is used in LevelDB and RocksDB, key-value storage engine libraries that are designed to be embedded into other applications.  
Originally this indexing structure was described under the name _Log-Structured Merge-Tree_(or LSM-Tree), building on earlier work on log-structured filesystems.  
Storage engines that are based on this principle of merging and compacting often called LSM storage engines.  

Lucene, an indexing engine for full-text used by Elasticsearch and Solr, uses a similar method for storing its _term dictionary_.  
This is implemented with a key-value structure where the key is a word (_a term_) and the value is the list of IDs of all the documents that contain the word (_the postings list).  

### Performance optimizations

In order to optimize this kind of access, storage engines often use additional _Bloom filters_.  
It's a memory-efficient data structure for approximating the contents of a set, it can tell you if a key does not appear in the database.  

The most common options to determine the order and timing of how SSTables are compacted and merged are _size-tiered_ and _leveled_ compaction.
- LevelDB, RocksDB use leveled compaction
- HBase uses size-tiered
- Cassandra support both

In level compaction, the key range is split up into smaller SSTables and older data is moved into separate "levels", 
which allows the compaction to proceed more incrementally and use less disk space.  

The basic idea of LSM-trees(keeping a cascade of SSTables that are merged in the background) is simple and effective.  
It works well when the dataset is much bigger than the available memory.  
Since data is stored in sorted order, you can efficiently perform range queries.

<br/>


