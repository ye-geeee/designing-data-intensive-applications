# Chapter 4. Encoding and Evolution

1. [Formats for Encoding Data](#Formats-for-Encoding-Data)
    - [Language Specific Formats](#Language-Specific-Formats)
    - [JSON, XML, and Binary Variants](#JSON,-XML,-and-Binary-Variants)
    - [Thrift and Protocol Buffers](#Thrift-and-Protocol-Buffers)
    - [Avro](#Avro)
    - [The Merits of Schemas](#The-Merits-of-Schemas)
2. [Modes of Dataflow](#Modes-of-Dataflow)
    - [Dataflow Through Databases](#Dataflow-Through-Databases)
    - [Dataflow Through Services: REST and RPC](#Dataflow-Through-Services:-REST-and-RPC)
    - [Message Passing Dataflow](#Message-Passing-Dataflow)

<br/>

We should aim to build systems that make it easy to adapt to change(_evolvability_).  
In most cases, the system requires data to change:  

- Relational databases: schema can be changed(i.e ALTER statement)
- schema-on-read: mixture of older and newer data

When a data format or schema changes, a corresponding change to application code often needs to happen.  

- server-side applications: _rolling upgrade_(_staged rollout_)
   - deploying the new version to a few nodes at a time, checking whether the new version is running smoothly, gradually working your way through all the nodes
- client-side applications: mercy of user... 

This means that old and new versions of the code, and data formats may potentially all coexist in the system at the same time.  
And we need to maintain compatibility in both directions:  

- _Backward compatibility_: Newer code can read data that was written by older code
- _Forward compatibiliry_: Older code can read data that was written by newer code

Let's have a look at a several formats for encoding data(JSON, XML, Protocol Buffers, Thrift, and Avro).  
And discuss how these formats are used for data storage and for communication: Representational State(REST), remote procedure calls (RPC), message-passing systems.  

<br/>

## Formats for Encoding Data

Programs usually work with data in at least two different representations:  

1. In memory, data kept in objects, structs, lists, arrays, hash tables, trees and so on. 
   These data  structures are optimized for efficient access and manipulation by the CPU.
2. When you want to write data to a file or send it over the network, you have to encode it as 
   some kind of self-contained sequence of bytes(ex. JSON).  
   
Thus, we need some kind of translation between the two representations.  

- in-memory to a byte sequence: _encoding(serialization, marshalling)_
- a byte sequence to in-memory: _decoding(parsing, deserialization, unmarshalling)_

### Language Specific Formats

Many programming languages come with build in support for encoding in-memory objects into byt sequences.  
(Java - java.io.Serializable, Ruby - Marchal etc)

However, even though they are very convenient, they have a number of deep problems.  

- The encoding is often tied to a particular programming language, and reading the data in another language is very difficult.
- security problem: The decoding process needs to be able to instantiate arbitrary classes in order to restore data in the same object types.
- Versioning data is often an afterthought. 
- Efficiency(CPU) is also often an afterthought. 

### JSON, XML, and Binary Variants

### Thrift and Protocol Buffers

### Avro

### The Merits of Schemas

<br/>

## Modes of Dataflow

### Dataflow Through Databases

### Dataflow Through Services: REST and RPC

### Message Passing Dataflow


