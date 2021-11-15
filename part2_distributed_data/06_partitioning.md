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

## Partitioning of Key Value Data

### Partitioning by Key Range

### Partitioning by Hash of Key

### Skewed Workloads and Relieving Hot Spots

## Partitioning and Secondary Indexes

### Partitioning Secondary Indexes by Document

### Partitioning Secondary Indexes by Term

## Rebalancing Partitions

### Strategies for Rebalancing

### Operations Automatic or Manual Rebalancing

## Request Routing

### Parallel Query Execution
