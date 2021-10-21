# Chapter 3. Storage and Retrieval

1. [Data Structures That Power Your Database](#Data-Structures-That-Power-Your-Database)
2. [Hash Indexes]
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

To tune a storage engine to perform well on your workload, 
you need to have a rough idea of what the storage engine is doing under the hood.  

In particular, there is a big difference between storages engines  
1. which are optimized for transactional workloads
2. which are optimized for analytics.  

Let's examine two families of :punch:_log-structured_ storage engines, and :punch:_page-oriented_ storage engines such as B-trees.  

## Data Structures That Power Your Database

Many databases internally use a :star2:_log_ which is an append-only data file to deal with mores issues
(concurrency control, reclaiming disk space, handling errors, partially written records).  
In addition, in order to efficiently find the value for a particular key in the database, we need a difference data structure: :star2:_index_
Therefore, the general idea behind them is 
1. to keep some additional metadata on the side and
2. if you want to search, you may need several difference indexes on different parts of the data.  

An index is an _additional structure_.  
Many data-bases allow you to add and remove indexes, and it should not affect the contents of the databases; it only affects performance.  
Any kind of indexes usually slows down writes, because the index need to be updated everytime when data is written.  

Therefore, well-chosen indexes speed up read queries, but every index slow down writes.  
For this reason, databases don't usually index everything by default, you can choose the indexes that give your application the greatest benefit, 
without introducing more overhead than necessary.  


