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

A common approach for notifying consumers about new events is to use a _messaging system_.  
The basic model of this approach is Unix pipe or TCP communication, which connect exactly one sender with one recipient.  
The messaging system allows multiple producer nodes to send messages to the same topic and allows multiple consumer nodes to receive messages in a topic.  

Within _publish/subscribe_ model, different systems take a wide range of approaches, and there is no one right answer for all purposes:  

1. _What happens if the producers send messages faster than the consumers can process them?_
   - a. drop messages
   - b. buffer messages in a queue - must understand what happened as that queue grows
   - c. apply _backpressure_(blocking the producer from sending more messages)
2. _What happens if nodes crash or temporarily go offline - are any messages lost?_
   - durability may require some combination of writing to disk and/or replication
    
Whether message loss is acceptable depends very much on the application.  
However, beware that if many messages are dropped, it may not be immediately apparent that the metrics are incorrect.  
A nice property of the batch processing systems is that they provide a strong reliability guarantee:  
failed tasks are automatically retried, and partial output from failed tasks is automatically discarded.  

#### Direct Messaging from producers to consumers

- UDP multicast is widely used in the financial industry for streams such as stocks market feeds, where low latency is iomportant.  
- ZeroMQ(brokerless messaging libraries), and nanomsg implements publish/subscribe over TCP or IP multicast.  
- StatsD and Brubeck use UDP messaging for collectin metrics from all machines on the network and monitoring them.  
- If the consumer exposes a service on the network, producers can make a direct HTTP or RPC request.  

With this approach, they generally require the application code to be aware of the possibility of message loss.  
If a consumer is offline, it may miss messages that were sent while it is unreachable.  

#### Message brokers

A widely used alternativ is to send messages via a _message broker_, 
which is essentially a kind of database that is optimized for handling message streams.  

By centralizing the data in the broker, these systems can more easily tolerate clients that come and go, 
and the question of durability is moved to the broker instead.  

A consequence of queueing is also that consumers are generally _asynchronous_:  
when a producer sends a message, it normally only waits for the broker to confirm.  

#### Message brokers compared to databases

1. Deletion
- database: keep data until it is explicitly deleted
- message brokers: delete message when is has been delivered to its consumers -> not suitable for long-term data storage
2. Queue size
- message brokers: assume that their work set is fairly small, so if consumers are slow there and individual message takes longer to process, the overall throughput may degrade
3. Secondary Index
- database: support secondary index
- message brokers: support using a subset of topics matching some pattern
4. Arbitrary queries
- database: based on a point-in-time snapshot of the data, if the data changed, the first client does not find out that its prior result is now outdated
- message brokers: do not support arbitrary queries, but they do notify clients when data changes

#### Multiple consumers

When multiple consumers read messages in the same topic, two main patterns of messaging are used:  

_Load balancing_
- Each message is delivered to _one_ of the consumers
- able to add consumers to parallelize the processing
_ in JMS, it is called a _shared subscription_

_Fan-out_
- Each message is delivered to _all_ of the consumers
- in JMS, it is supported by topic subscriptions, and exchange bindings in AMQP

#### Acknowledgments and redelivery

In order to ensure that the message is not lost by crash, message brokers use _acknowledgments_:  
a client must explicitly tell the broker when it has finished processing a message so that the broker can remove it from the queue.  

If there is no acknowledgment, the broker delivers the message again to another consumer.  
When combined with load balancing, this redelivery behavior has an interesting effect on the ordering of messages.  

Even if the message broker otherwise tries to preserve the order of messages, 
the combination of load balancing with redelivery inevitably leads to message being reordered.  
To avoid this issue, you can use a separate queue per consumer.  

<br/> 

### Partitioned Logs

Sending a packet over a network or making a request to a network service is normally a transient operation that leaves no permanent trace.  
You cannot run the same consumer again and expect to get the same result.  

Why can we not have a hybrid, combining the durable storage approach of databases with the low-latency notification facilities of messaing?  
This is the idea behind _log-based message brokers_.  

#### Using logs for message storage

A log is simply an append-only sequence of records on disk.  
The same structure can be used to implement a message broker:  
a producer sends a message by appending it to the end of the log, 
and a consumer receives message by reading the log sequentially.  

In order to scale to higher throughput than a single disk can offer, the log can be _partitioned_.  
Different partitions can then be hosted on different machines, making each partition a separate log that can be read and written independently of other partitions.

With each partition, the broker assigns a monotonically increasing sequence number, or _offset_, to every message.  
There is no ordering guarantee across different partitions.  

Apache Kafka, Amazon Kinesis Streams, and Twitter's DistributedLog are log-based message brokers that work like this.  
Google Cloud Pub/Sub is similar but exposes a JMS-style API rather than a log abstraction.  

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

