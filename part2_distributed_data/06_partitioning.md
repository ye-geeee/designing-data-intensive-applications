# Chapter 5. Replication

1. [Partitioning and Replication](#Partitioning-and-Replication)
2. [Partitioning of Key Value Data](#Partitioning-of-Key-Value-Data)
    - [Partitioning by Key Range](#Partitioning-by-Key-Range)
    - [Partitioning by Hash of Key](#Partitioning-by-Hash-of-Key)
    - [Skewed Workloads and Relieving Hot Spots](#Skewed-Workloads-and-Relieving-Hot-Spots)
3. [Partitioning and Secondary Indexes](#Partitioning-and-Secondary-Indexes)
   - [Partitioning Secondary Indexes by Document](#Partitioning-Secondary-Indexes-by-Document)
   - [Partitioning Secondary Indexes by Term](#Partitioning-Secondary-Indexes-by-Term)
4. [Rebalancing Partitions](#Rebalancing-Partitions)
   - [Strategies for Rebalancing](#Strategies-for-Rebalancing)
   - [Operations Automatic or Manual Rebalancing](#Operations-Automatic-or-Manual-Rebalancing)
5. [Request Routing](#Request-Routing)
   - [Parallel Query Execution](#Parallel-Query-Execution)

<br/>

The main reason for wanting to partition data is _scalability_.  
Different partition can be placed on different nodes in a shared-nothing cluster.  
Complex queries can potentially be parallelized across many nodes.  

In this chapter, we will look at  
1. different approaches for partitioning large datasets
2. observe how the indexing of data interacts with partitioning
3. rebalancing - necessary if you want to add or remove nodes
4. overview of how databases route requests to the right partitions and execute queries  

<br/> 

## Partitioning and Replication

Partitioning is usually combined with replication so that copies of each partition are stored on multiple nodees.  
It may still be stored on different nodes for fault tolerance.  

In a leader-follower replication model,  
each partition's leader is assigned to one node, and its followers are assigned to other nodes.  

![03_partitioning_and_replication](../resources/part2/03_partitioning_and_replication.png)

## Partitioning of Key Value Data

Our goal with partitioning is to spread the data, and the query load evenly across nodes.  

_skewed_: some partitions have more data or queries than others  
_hot spot_: A partition with disproportionately high load   

If you assign records randomly,  
the big disadvantage is when you're trying to read a particular item, you have no way of knowing which node it is on, so you have to query all nodes in parallel.  

If you have a simple key-value data model,  
since all alphabetically sorted by title, you can quickly find the one you're looking for.  

### Partitioning by Key Range

One way of partitioning is to assign a continuous range of keys to each partition.  
If you know the boundaries between the ranges, you can easily determine which partition contains a given key.  

The ranges of keys are not necessarily evenly spaced, because your data may not be evenly distributed.  
So, the partition boundaries need to adapt to the data.  

For example, consider an application that stores data from a network fof sensors, where the key is the timestamp of the measurement.  

**Advantage**

- range scans are easy, 
- you can treat the key as a concatenated index in order to fetch several related records in one query.  

**Disadvantage**

- certain access patterns can lead to hot spots 
   - if we write data from the sensors to the databases as the measurements happen
   - all the writes end up going to the same partition

To avoid this problem, you need to use something other than timestamp.  
For example, you could prefix each timestamp with the sensor name.  
Then, when you want to fetch the values of multiple sensors within a time range, you need to perform a separate range query for each sensor name.  

### Partitioning by Hash of Key

Because of risk of skew and hot spots, many distributed datastores use a hash function to determine the partition for a given key.  
A good hash function takes skewed data and makes it uniformly distributed.  

**Advantage** 

- _consistent hashing_ : The partition boundaries can be evenly spaced, or they can be chosen pseudorandomly  

**Disadvantage**

- lose a nice property of key-range partitioning 

In case of Cassandra to compromise between the two partitioning strategies,  
it uses a tables declared with a _compound primary key_ consisting of several columns.  
Only first part of that key is hashed to determine the partition, but the other columns are used a sa concatenated index for sorting the data.  

If it specifies a fixed value for the first column, it can perform an efficient range scan over the otehr columns of the key.  
The concatenated index approach enables an elegant data model for one-to-many relationships.

### Skewed Workloads and Relieving Hot Spots

Hashing a key to determine its partition can help reduce hot spots.  
However, it can't avoid them entirely: in extreme case where all reads and writes are for the same key,  
you still end up with all requests being routed to the same partition.  
Therefore, hashing the key doesn't help.  

Today, most data systems are not able to automatically compensate for such a highly skewed workload.  
So, it's the responsibility of the application to reduce the skew.  
A simple technique is to add a random number to the beginning or end of the key.  
However, having split the writes across different keys, any reads now have to do additional work,  
as they have to read the data from all keys and combine it.

## Partitioning and Secondary Indexes

### Partitioning Secondary Indexes by Document

### Partitioning Secondary Indexes by Term

## Rebalancing Partitions

### Strategies for Rebalancing

### Operations Automatic or Manual Rebalancing

## Request Routing

### Parallel Query Execution
