# Stream Processing

- [Transmitting Event Streams](#Transmitting-Event-Streams)
    - [Message Systems](#Message-Systems)
    - [Partitioned Logs](#Partitioned-Logs)
- [Databases and Streams](#Databases-and-Streams)
    - [Keeping Systems in Sync](#Keeping-Systems-in-Sync)
    - [Change Data Capture](#Change-Data-Capture)
    - [Event Sourcing](#Event-Sourcing)
    - [State, Streams, and Immutability](#State,-Streams,-and-Immutability)
- [Processing Streams](#Processing-Streams)
    - [Uses of Stream Processing](#Uses-of-Stream-Processing)
    - [Reasoning About Time](#Reasoning-About-Time)
    - [Stream Joins](#Stream-Joins)

To reduce delay of batch processing, we can run the processing more frequently.  
This is the idea behind _stream processing_.

In this chapter we will look at _event streams_ as a data management mechanism:  
unbounded, incrementally processed counterpart to the batch data.

**Topic**

- how streams are represented, stored, and transmitted over a network
- relationship between stream sand databases
- approaches and tools for processing those streams continually, and way that they can be used to build applications

<br/>

## Transmitting Event Streams

In a stream processing context, a record is more commonly knows as an _event_.  
An event is generated once by a _producer_(also knows as a _publisher_ or _sender_), 
and then potentially processed by multiple _consumers_(_subscribers_ or _recipients_).  
Plus, related events are usually grouped together into a _topic_ or _stream_.  

In principle, a file or database is sufficient to connect producers and consumers:  
a producer writes every event that is generates to the datastore, 
and each consumer periodically polls the datastore to check for events that have appeared since it last ran.  

However, when moving toward continual processing with low delays, polling is expensive.  
So, it is better for consumers to be notified when new events appear.

### Message Systems

#### Direct Messaging from producers to consumers

#### Message brokers

#### Message brokers compared to databases

#### Multiple consumers

#### Acknowledgments and redelivery

### Partitioned Logs

#### Using logs for message storage

#### Logs compared to traditional messaging

#### Consumer offsets

#### Disk space usage

#### When consumers cannot keep up with producers

#### Replaying ole messages

## Databases and Streams

### Keeping Systems in Sync

### Change Data Capture

#### Implementing change data capture

#### Initial snapshot

#### Log compaction

#### API support for change streams

### Event Sourcing

#### Deriving current state from the event log

#### Commands and events

### State, Streams, and Immutability

#### Advantages of immutable events

#### Deriving several views from the same event log

#### Concurrency control

#### Limitations of immutability

## Processing Streams

### Uses of Stream Processing

#### Complex event processing

#### Stream analytics

#### Maintaining materialized views

#### Search on streams

#### Message passing and RPC

### Reasoning About Time

#### Event time versus processing time

#### Knowing when you're ready

#### Whose clock are you using, anyway?

#### Types of windows

### Stream Joins

#### Stream-stream join (window join)

#### Stream-table join (stream enrichment)

#### Table-table join (materialized view mainenance)

#### Time-dependence of joins

### Fault Tolerance

#### Microbatching and checkpointing

#### Atomic commit revisited

#### Idempotence

#### Rebuilding state after a failure

