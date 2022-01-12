# Chapter 9. Consistency and Consensus

1. [Consistency Guarantees](#Consistency-Guarantees)
2. [Linearizability](#Linearizability)
    - [What Makes a System Linearizable?](#What-Makes-a-System-Linearizable?)
    - [Relying on Linearizabilty](#Relying-on-Linearizabilty)
    - [Implementing Linearizable Systems](#Implementing-Linearizable-Systems)
    - [The Cost of Linearizability](#The-Cost-of-Linearizability)
3. [Ordering Guarantees](#Ordering-Guarantees)
    - [Ordering and Causality](#Ordering-and-Causality)
    - [Sequence Number Ordering](#Sequence-Number-Ordering)

<br/>

In chapter 8, the simplest way of handling faults is to simply entire service fail, and show user and error message.  
If that solution is unacceptable, we need to find ways of _tolerating_ faults(packet loss, reordered, duplicated, arbitrarily delayed, nodes pause, crash).  

One of the most important abstractions for distributed systems is _consensus_: getting all the nodes to agree on something.  
In this chapter, we will look into algorithms to solve consensus and related problems.  
Also, we will get an overview of what is and isn't possible and those fundamental limits.  

<br/>

## Consistency Guarantees

Most replicated databases provide at least _eventual consistency_(also called as _convergence_):  
stop writing to the database and wait for some unspecified length of time, and eventually all read requests will return the same value.  
However, this is weak guarantee - it doesn't say anything about _when_ the replicas will converge.  

When working with a database that provides only weak guarantees, you need to be constantly aware of its limitations and bugs are subtle and hard to find by testing.  
In this chapter, we will explore stronger consistency models that data systems may choose to provide, but have worse performance or be less fault-tolerant that the systems with weawker guarantees.

<br/>

## Linearizability

In an eventually consistent database, if you ask two different replicas the same question at the same time, you may get two different answers.  
Wouldn't it be a lot simpler if the database could give the illusion that there is only one replica?

This is the idea behind _linearizability(= atomic consistency, strong consistency, immediate consistency, external consistency)_:  
a system appear as if there were only one copy of the data.  
With this guarantee, even though there may be multiple replicas in reality, the application does not need to worry about them.  
In other words, linearizability is a _recency guarantee_.  

### What Makes a System Linearizable?

**Not fully linearlizable Example**

![08_non-linearlizable_example](../resources/part2/08_non-linearlizable_example.png)

Any read operations that overlap in time with operation might return either 0 or 1.  
There operations are _concurrent_ with the write.  
However, this is not yet sufficient to fully describe linearizability:  
if reads that are concurrent with a write can return either the old or the new value.  

**Linearizable Example 1**

![09_linearlizable_example](../resources/part2/09_linearlizable_example.png)

If one client's read returns the new value, all subsequent reads must also return the new value, even if the write operation has not yet completed.  

**Linearlizable Example 2**

![10_linearlizable_example2](../resources/part2/10_linearlizable_example2.png)

#### Linearizability Versus Serializability

**Linearlizability**: 
recency guarantee on reads and writes of a register

**Serializability**:  
an isolation property of _transactions_, where every transaction may read and write multiple objects

A database may provide both serializability and linearlizability, and this combination is knows as _strict serializability_ or _strong one-copy serializability_.  
Implementations of serializability based on two-phase locking or actual serial execution are typically linearizable.  

However, serializable snapshot isolation is not linearizable;  
the whole point of a consistent snapshot is that it does not include writes that are most recent that the snapshot,  
and thus reads from the snapshot are not linearizable.  

### Relying on Linearizabilty

There is a few areas in which linearizability is an important requirement for making a system work correctly.  

#### Locking and leader election

A system that uses single-leader replication needs to ensure that there is indeed only one leader, not several(split brain).  
Apache ZooKeeper, etcd are often used to implement distributed locks and leader election.  
A linearizable storage service is the basic foundation for these coordination tasks(they use details using libraries like Apache Curator).  
Also, Oracle Real Application Clusters(RAC) uses a lock per disk page, with multiple nodes sharing access to the same disk storage system.

#### Constraints and uniqueness guarantees

If you want to enforce contraints and uniqueness guarantees as the data is written, you need linearizabilty.  
For example, a username or email address must uniquely identify one user, bank account balance never goes negative.  

Other kinds of constraints, such as foreign key or attribute constraints, can be implemented without requiring linearizability.  

#### Cross-channel timing dependencies

The linearizabiltiy violation was only noticed because there was an additional communication channel in the system.  

![11_cross_channel_communication_example](../resources/part2/11_cross_channel_communication_example.png)

There is a risk of a race condition:  
the message queue might be faster than the internal replication inside the storage service.  
When the resizer fetches the image, it might see an old version of the image.  

This problem arises because there are two different communication channels between the web server and the resizer: the file storage and message queue.

### Implementing Linearizable Systems

The most common approach to making a system fault-tolerant is to use replication.  
Let's check replication methods, and compare whether they can be made linearizable:  

- _Single-leader replication (potentially linearizable)_
  - If you make reads from the leader, or from synchronously updated followers, they have the _potential_ to be linearizzable.  
  - However, not every single-leader database is actually linearizable, either by design or due to concurrency bugs.
  - Also, you have to know for sure who the leader is.
  - With asynchronous replication, failoever may even lose commited writes, which violates both durability and linearizability.  
- _Consensus algorithms (linearizable)_
  - They can bear a resembalance to single-leader replication with measures to prevent split brain and stale replicas.
  - Zookeeper, etcd
- _Multi-leader replication (not linearizable)_
  - They concurrently process writes on multiple nodes and asynchronously replicate them to other nodes.
  - Therefore, they can produce conflicting writes.  
- _Leaderless replication (probably not linearizable)_  
  - They depend on how you define strong consistency. However, it is not quite true.
  - Conflict resolution based on time-of-day clocks in Cassandra, are nonlinearizable, because clock timestamps cannot be guaranteed.
  - Sloppy quorums also ruin any chance of lineariability.

#### Linearizability and quorums

It seems as though strict quorum reads and writes should be linearizable.  
However, when we have variable network delays, it is possible to have race conditions.  

Interestingly, it is possible to make Dynamo-style quorums linearizable at the cost of reduced performance;  
a reader must perform read repair, and a writer must read the latest state of a quorum of nodes before sending its writes.  

However, Riak does not perform synchronous read repair due to the performance penalty.  
Cassandra does wait for read repair to complete on quo‐ rum reads, 
but it loses linearizability if there are multiple concurrent writes to the same key, 
due to its use of last-write-wins conflict resolution.

Therefore, it is safest to assume that a leaderless system with Dynamo-style replication does not provide linearizability.  

### The Cost of Linearizability

Let's consider what happens if there is a network interruption between the two datacenters.  
With a multi-leader database, each datacenter can continue operating normally,  
but if a single-leader replication is used, the leader must be in one of the datacenters.  

If the application requires linearizable reads and writes, 
the network interruption causes the application to become unavailable in the datacenters that cannot contact the leader.  

#### The CAP theorem

Any linearizable database has this problem, no matter how it is implemented.  
The issue also isn't specific to multi-datacenter deployments, but can occur on any unreliable network, even within one datacenter.  
The trade-off is as follows:

- If your application _requires_ linearizability, they must wait until the network problem is fixed, or return an error.  
- If your application _does not require_ linearizability, the application can remain _available_ in the face of a network problem, but its behavior is not linearizable.  

Thus, applications that don't require linearizability can be more tolerant of network problems.  
This insight is popular known as the _CAP theorem_.  

At the time, CAP encouraged database engineers to explore a wider design space of distributed shared-nothing systems, which is suitable for implementing large-scale web services.  
CAP deserves credit for this culture shift-witness the explosion of new database technologies since the mid-2000s(known as NoSQL).  

The CAP theorem as formally defined is of very narrow scope: consistency model(linearizability), kind of fault(network partitions).  
Thus, although CAP has been historically influential, it has little value for designing systems.  

#### Linearizability and network delays

Although linearizability is a  useful guarantee, few systems are actually linearizable in practice.  
Even RAM on a modern multi-core CPU is not linearizable.  
If a thread running on one CPU core writes to a memory address, and a thread on another CPU core reads the same address shortly afterward, 
it is not guaranteed to read the value written by the first thread(unless a _memory barrier_ or _fence_ is used).  

The reason for dropping linearizability is _performance_, not fault tolerance.  
Many distributed databases do so primarily to increase performance, not so much for fault tolerance.  
A faster algorithm for linearizability does not exist, but weaker consistency models can be much faster, 
so this trade-off is important for latency-sensitive systems.  

<br/>

## Ordering Guarantees

### Ordering and Causality

There are several reasons why ordering keeps coming up, and one of the reasons is that it helps preserve _causality_.  
Causality imposes an ordering on events: cause some before effect; a message is sent before that message is received; 
the question comes before the answer.  
If a system obeys the ordering imposed by causality, we say that it is _causally consistent_.  

#### The causal order is not a total order

_total order_: allows any two elements to be compared  
  - _Linearizability_: every operation is atomic
_partially ordered_: in some cases one set is greater than another, but in other cases they are incomparable.  
  - _Causality_: two events are ordered if they are causally related, but they are incomparable if they are concurrent.  

Therefore, according to this definition, there are no concurrent operations in a linearizable datastore:  
there must be a single timeline along which all operations are totally ordered.  

#### Linearizability is stronger that causal consistency

The linearizability _implies_ causality: any system that is linearizable will preserve causality correctly.  
Making a system linearizable can harm its performance and availability, 
so some distributed data systems abandoned linearizability to achieve better performance.  

The good news is that a middle ground is possible.  
In many cases, systems that appear to require linearizability in fact only really require causal consistency, 
which can be implemented more efficiently.

#### Capturing causal dependencies

In order to maintain causality, you need to know which operation _happened before_ which other operation.  
In order to determine causal dependencies, we need some way of describing the "knowledge" of a node in the system(similar to detecting concurrent writes).  
In order to determine causal ordering, the databases needs to know which version of the data was read by the application.  

### Sequence Number Ordering

In many applications, clients read lots of data before writing something.  
Explicitly tracking all the data would mean a large overhead.  

There is a better way: we can use _sequence numbers_ or _timestamps_ to other events.  
In this case, we do not use problematic a time-of-day clock, we use _a logical clock_ instead.  

Such sequence numbers or timestamps are compact, and they provide a _total order_.  
With a single-leader replication, the replication log defines a total order of write operations that is consistent with causality.

#### Noncausal sequence number generators

If there is not a single leader, it is less clear how to generate sequence numbers for operations.
There are some methods in practice, 
and all perform better and are more scalable than pushing all operations through a single leader that increments a counter.  
However, they all have problem: the sequence numbers they generate are _not consistent with causality_.  

There are various methods in practice and problems:  

1. Each node can generate its own independent set of sequence numbers. (for two nodes, one uses odd number and the other uses even number)
  - problem: cannot accurately tell which one causally happened first
2. You can attach timestamp from a time-of-day clock to each operation.  
  - problem: clock skew
3. You can preallocate blocks of sequence numbers(one for 1~1000, another for 1001~2000).
  - problem: the sequence number is inconsistent with causality  

#### Lamport timestamps

There is a simple method for generating sequence numbers that is consistent with causality: _Lamport timestamp_

![12_Lamport_timestamps](../resources/part2/12_Lamport_timestamps.png)

Every node and every client keeps track of the _maximum_ counter value it has seen so far, 
and includes that maximum on every request.  

Lamport timestamps are sometimes confused with version vectors.  

- version vectors: distinguish whether two operations are concurrent or whether one is causally dependent on the other
- Lamport timestamps: enforce a total ordering

#### Timestamp ordering is not sufficient

Although Lamport timestamps define a total order of operations, they are not sufficient to solve many common problems in distributed systems.  
You can compare the timestamps to determine the winner, 
but it is not sufficient when a node has just received a request, and need to decide _right_ now whether the request should succeed or fail.  
The node does not know whether another node is concurrently in the process of same operation, and what timestamp that other node may assign to the operation.  
You would have to check with every other node to see what it is doing.  

The problem here is that the total order of operations only emerges after you have collected all of the operations.  
To conclude: in order to implement something like a uniqueness constraints for usernames, 
it's not sufficient to have a total ordering of operations - you also need to know when that order si finalized.  

