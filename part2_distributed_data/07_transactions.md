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

Almost all relational databases, and some nonrelational databases, support transactions.  
Most of them follow the style of IBM System R introduced in 1975.

In the late 2000s, nonrelational databases started gaining popularity.  
Many of this new generation of databases abandoned transactions entirely, 
or redefined the word to describe a much weaker set of guarantees than had previously been understood.  

Like every other technical design choice, transactions have advantages and limitations.  
There are two view points of transactions,  
- any large-scale system would have to abandon transactions to maintain good performance and high availabilty
- transactional guarantees are an essential requirement for "serious application" with "valuable data"

### The Meaning of ACID

ACID, stands for _Atomicity, Consistency, Isolation, and Durability_.  
It was coined in 1983 by Theo Harder and Andreas Reuter to establish precise teminology for fault-tolerance mechanisms in databases.  
However, in practice, one database's implementation of ACID does not equal another's implementation.  
There is a lot of ambiguity around the meaning of _isolation_.  

#### Atomicity

In general, _atomic_ refers to something that cannot be broken down into smaller parts.  
ACID atomicity describes what happens if a client wants to make several writes, but a fault occurs after some of the writes have been processed.  
If the writes are grouped together into an atomic transaction, and the transaction cannot be completed(_committed_) due to a fault, 
than the transaction is _aborted_ and database must discard or undo any writes it has made.  

Atomicity simplifies this problem: if a transaction was aborted, the application can be sure that it didn't change anything, so it can safely be retired.  

#### Consistency

In the context of ACID, _consistency_ refers to an application-specific notion of the database being in a "good state.".  
Therefore, you have certain statements about your data(_invariants_) that must always be true.  

However, this idea of consistency depends on the application's notions of invariants, 
and it's the application's responsibility to define its transactions correctly so that they preserve consistency.  

Atomicity, isolation, and durability are properties of the database, whereas consistency (in the ACID sense) is a property of the application.  

#### Isolation

Most databases are accessed by several clients at the same time.  
If they are accessing the same database records, you can run into concurrency problems (race conditions).  

_Isolation_ is the sense of ACID means that concurrently executing transactions are isolated from each other: they cannot step on each other's toes.  
The classic database textbooks formalize isolation as _serializability_:  
each transaction can pretend that it is the only transaction running on the entire database.  

However, in practice, serializable isolation is rarely used, because it carries a performance penalty.  
Some popular databases, such as Oracle 11g, don't even implement it.  
There is an isolation level called "serializable", which actually implements something called _snapshot isolation_.

#### Durability

_Durability_ is the promise that once a transaction has committed successfully, 
any data it has written will not be forgotten, even if there is a hardware fault, or the database crashes.  

In a single-node database, durability means that the data has been written to nonvolatile storage.  
In a replicated database, durability many mean that data has been successfully copied to some number of nodes.  
In order to provide a durability guarantee, a database must wait until these writes or replications are complete before reporting a transaction as successfully committed.  

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
