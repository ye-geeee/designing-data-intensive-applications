# Chapter 5. Replication

1. [Leaders and Followers](#Leaders-and-Followers)
   - [Synchronous Versus Asynchronous Replication](#Synchronous-Versus-Asynchronous-Replication)
   - [Setting Up New Folowers](#Setting-Up-New-Followers)
   - [Handling Node Outages](#Handling-Node-Outages)
   - [Implementation of Replication Logs](#Implementation-of-Replication-Logs)
2. [Problems with Replication Log](#Problems-with-Replication-Log)
   - [Reading Your Own Writes](#Reading-Your-Own-Writes)
   - [Monotonic Reads](#Monotonic-Reads)
   - [Consistent Prefix Reads](#Consistent-Prefix-Reads)
   - [Solutions for Replication Lag](#Solutions-for-Replication-Lag)
3. [Multi-Leader Replication](#Multi-Leader-Replication)
   - [Use Cases for Multi-Leader Replication](#Use-Cases-for-Multi-Leader-Replication)
   - [Handling Write Conflicts](#Handling-Write-Conflicts)
   - [Multi-Leader Replication Topologies](#Multi-Leader-Replication-Topologies)
4. [Leaderless Replication](#Leaderless-Replication)
   - [Writing to the Database When a Node Is Down](#Writing-to-the-Database-When-a-Node-Is-Down)
   - [Limitations of Quorum Consistency](#Limitations-of-Quorum-Consistency)
   - [Sloppy Quorums and Hinted Handoff](#Sloppy-Quorums-and-Hinted-Handoff) 
   - [Detecting Concurrent Writes](#Detecting-Concurrent-Writes)

<br/>

Replication: keeping a copy of the same data on multiple machines that are connected via a network

- To keep data geographically close to your users
- To allow the system to continue working even if some of its parts have failed(and thus increase availability)
- To scale out the number of machines that can serve read queries (and thus increase read throughput)

**Topic**

- how to handle changes to replicated data
- popular algorithm for replicating changes: _single-leader, multi-leader, leaderless_
- trade-offs between algorithm
   - whether to use synchronous or asynchronous replication
   - how to handle failed replicas
   - eventual consistency
   - _read-your-writes_ and _monotonic reads_ guarantees

## Leaders and Followers

_replica_: each node that stores a copy of the database  
The most common solution for this is called: _leader-based replication (active/passive of master-slave replication)_  

1. One of the replicas is designated the _leader_ (for write)
2. The other replicas are known as _followers_ (for read) - whenever leader writes new data to its local storage, 
   it also sends the data change to all of its followers as part of _replication log_ or _change stream_.  
3. When a client wants to read from database, it can query either the leader or any of the followers, but only accepted by leader.

### Synchronous Versus Asynchronous Replication

Whether the replication happens _synchronously_ or _asynchronously_ is an important detail of a replicated system.  

![Sample of leader-based replication](../resources/part2/01_leader-based_replication.png)

Figure 5-2 is general case of replication.  
One for _synchronous_ and others for _asynchronous_.  

**Synchronous Replication**

- pros: follower is guaranteed to have an up-to-date copy of the data that is consistent with the leader.  
- cons: if the follower does not respond, writes cannot be processed and all the writes are blocked.  

**_Semi-Synchronous_**

- It is impractical for all followers to be synchronous
- _one_ of the followers is synchronous, and the others are asynchronous
- If the synchronous follower becomes unavailable or slow, one of the asynchronous followers is made synchronous
- up-to-date copy of data on at least two nodes

**Leader-based Replication with Fully Asynchronous**

- often case,  widely used
- write is not guaranteed to be durable
- leader can continue process writes, even if all of its followers have fallen behind
- _chain replication_ to provide good performance and availability implemented in Microsoft Azure Storage

### Setting Up New Followers

### Handling Node Outages

### Implementation of Replication Logs

## Problems with Replication Log

### Reading Your Own Writes

### Monotonic Reads

### Consistent Prefix Reads

### Solutions for Replication Lag

## Multi Leader Replication

### Use Cases for Multi Leader Replication

### Handling Write Conflicts

### Multi Leader Replication Topologies

## Leaderless Replication

### Writing to the Database When a Node Is Down

### Limitations of Quorum Consistency

### Sloppy Quorums and Hinted Handoff

### Detecting Concurrent Writes


