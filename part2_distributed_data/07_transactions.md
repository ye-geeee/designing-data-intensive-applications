# Chapter 7. Transactions

1. [The Slippery Concept of a Transaction](#The-Slippery-Concept-of-a-Transaction)
    - [The Meaning of ACID](#The-Meaning-of-ACID)
    - [Single Object and Multi Object Operations](#Single-Object-and-Multi-Object-Operations)
2. [Weak Isolation Levels](#Weak-Isolation-Levels)
    - [Read Committed](#Read-Committed)
    - [Snapshot Isolation and Repeatable Read](#Snapshot-Isolation-and-Repeatable-Read)
    - [Preventing Lost Updates](#Preventing-Lost-Updates)
    - [Write Skew and Phantoms](#Write-Skew-and-Phantoms)
3. [Serializability](#Serializability)
    - [Actual Serial Execution](#Actual-Serial-Execution)
    - [Two Phase Locking (2PL)](#Two-Phase-Locking-(2PL))
    - [Serializable Snapshot Isolation (SSI)](#Serializable-Snapshot-Isolation-(SSI))

<br/>

In the harsh reality of data systems, many things can go wrong.  
For decades, _transactions_ have been the mechanism of choice for simplifying these issues.

**_Transactions_**: a way for an application to group several reads and writes together into a logical unit.  

Transactions are not a law of nature; they were created with a purpose, namely to _simplify the programming model_ for applications accessing a database. 
By using transactions, the application is free to ignore certain potential error scenarios and concurrency issues, 
because the database takes care of them instead(_safety guarantees_).  

We will go deep in the area of concurrency control, discussing various kinds of race conditions 
that can occur and how databases implement isolation levels such as _read committed, snapshot isolation, and serializabiltiy_. 

## The Slippery Concept of a Transaction

### The Meaning of ACID

#### Atomicity

#### Consistency

#### Isolation

#### Durability

### Single Object and Multi Object Operations

#### Single-object writes

#### The need for multi-object transactions

#### Handling errors and aborts

## Weak Isolation Levels

### Read Committed

#### No dirty reads

#### No dirty writes

#### Implementing read committed

### Snapshot Isolation and Repeatable Read

#### Implementing snapshot isolation

#### Visibility rules for observing a consistent snapshot

#### Indexes and snapshot isolation

#### Repeatable read and naming confusion

### Preventing Lost Updates

#### Atomic write operations

#### Explicit locking

#### Automatically detecting lost updates

#### Compare-and-set

#### Conflict resolution and replication

### Write Skew and Phantoms

#### Characterizing write skew

#### More examples of write skew

#### Phantoms causing write skew

#### Materializing conflicts

## Serializability

### Actual Serial Execution

#### Encapsulating transactions in stored procedures

#### Pros and cons of stored procedures

#### Partitioning

#### Summary of serial execution

### Two Phase Locking (2PL)

#### Implementation of two-phase locking 

#### Performance of two-phase locking

#### Predicate locks

#### Index-range locks

### Serializable Snapshot Isolation (SSI)

#### Pessimistic versus optimistic concurrency control 

#### Decisions based on an outdated premise

#### Detecting stale MVCC reads

#### Detecting writes that affect prior reads

#### Performance of serializable snapshot isolation
