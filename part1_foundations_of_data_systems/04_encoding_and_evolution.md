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

**Problems of Language Specific Formats:**

- The encoding is often tied to a particular programming language, and reading the data in another language is very difficult.
- security problem: The decoding process needs to be able to instantiate arbitrary classes in order to restore data in the same object types.
- Versioning data is often an afterthought.
- Efficiency(CPU) is also often an afterthought.

### JSON, XML, and Binary Variants

JSON, XML, and CSV are textual formats, and thus somewhat human-readable.

**Problems of Textual Formats:**

- A lot of ambiguity around the encoding of numbers(number and string, integers and floating-point numbers).
- JSON, XML have good support for Unicode character strings, and do not support binary strings.
   - There is a way to use Base64, but it's somewhat hacky and increases the data size by 33%.
- Optional schema support for XML and JSON: powerful but complicated to learn
- CSV does not have any schema, so application have to define the meaning of each row and column.

#### Binary encoding

Once you get into the terabytes, the choice of data format can have a big impact.  
JSON is less verbose than XML, but both still use a lot fo space compared to binary formats.  
There are profusion of binary encodings, but none of them are widely adopted.

### Thrift and Protocol Buffers

Apache Thrift and Protocol Buffers are binary encoding libraries.  
They require a schema for any data that is encoded.  
They come with a code generation tool that takes a schema definition.

Thrift has two different binary encoding formats, _Binary Protocol_ and _CompactProtocol_.

**Thrift BinaryProtocol**

![06_Thrift_BinaryProtocol](../resources/part1/06_Thrift_BinaryProtocol.png)

- Each field has a type annotation and length indication.
- No field names but _field tags_, those are numbers appear in the schema definition.

**Thrift CompactProtocol**

![07_Thrift_CompactProtocol](../resources/part1/07_Thrift_CompactProtocol.png)

- Pack the field type and tag number into a single byte.
- Use variable-length integers
- Do not use full eight bytes for numbers. (1 byte: -64~63, 2bytes: -8192~8191)

**Protocol Buffers**

![08_Protocol_Buffers](../resources/part1/08_Protocol_Buffers.png)

- Similar to Thrifts' CompactProtocol, but does a bit packing slightly different.

**Note!**

Difference between **required** field and **optional** field is
**required** enables a runtime check that fails if the field is not set, which can be useful for catching bugs.

#### Field tags and schema evolution

_schema evolution_: schemas inevitable need to change over time

Encoded record is just the concatenation of its encoded fields.  
Each field is identified by its tag number and annotated with a datatype.  
You can change the name of a field, but cannot change a field's tag that would make all existing encoded data invalid.  

**Add new fields**

- Old code: tries to read but could not recognize so just ignore that field
- New code: can always read old data
- Cannot make that field it required -> optional or have default value

**Removing new fields**

- just like adding new fields, with backward and forward compatibility concerns reversed
- Can only remove a field that is optional
- Can never use the same tag number again

#### Datatypes and schema evolution

Changing the datatype of a field may be possible - but there is a risk that values will lose precision or get truncated.  
For example, when you change a 32-bit integer into a 64-bit integer
- New code: parser can fill any missing bits with zeros
- Old code: 64-bit value won't fit in 32 bits, it will be truncated.  

**Protocol Buffers for list or array datatype**

Use the same field tag simply appears multiple times in record.  
- New code: reading old data with zero of one element. 
- Old code: reading new data sees only the last element of the list.  

**Thrift for list or array datatype**

Thrift has a dedicated list datatype, which is parameterized with the datatype of the list elements.  
It has advantage of supporting nested lists.

### Avro

![09_Avro](../resources/part1/09_Avro.png)

Avro also uses a schema to specify the structure of the data encoded.  
It has two schema languages: Avro IDL for human editing, based on JSON that is mores easily machine-readable.  

- No tag numbers in the schema.  
- Nothing to identify fields or their datatypes.  
- Simply consists of values concatenated together.  
- To parse binary data, check schema and use it to tell you the datatype of each field.  
- Binary data can only be decoded correctly if the data is using the _exact same schema_.  

#### The writer's schema and the reader's schema

- _writer's schema_: when an application wants to encode some data
- _reader's schema_: when an application wants to decode some data

Key idea of Avro: writer's schema and reader's schema _don't have to be the same_ - they only need to be compatible.  

- no problem with different order from writer's schema and reader's schema
- only in writer's schema -> ignored
- only in reader's schema -> filled in with default value declared in the reader's schema

#### Schema evolution rules

With Avro, 
- forward compatibility: new writer's schema + old reader's schema
- backward compatibility: new reader's schema + old writer's schema

**Characteristics**

- To maintain, you may only add or remove a field that has a default value. 
- To allow a field to be null, you have to use a _union type_.  
- Does not have optional and required markers
- Changing the datatype of a field is possible, Changing the name of a field is little tricky.  

#### But what is the writer's schema

**Avro Usage**

- _Large file with lots of records_
- _Database with inidvidually written records_
- _Sending records over a network connection_

#### Dynamically generated schemas

Advantage of Avro's approach: the schema does not contain any numbers.  
Avro is friendlier to _dynamically generated_ schemas.  

For Avro,
- If the database schema changes, you can just generate a new Avro schema from the updated database schema and export data in the new Avro schema. 
- Simply do the schema conversion every time it runs
- Fields are identified by name

For Thrift or Protocol Buffers, field tags would likely to be assigned by hand when schema changes - map database column names to field tags.

#### Code generation and dynamically typed languages

Thrift and Protocol Buffers rely on code generation - Java, C++, C# which allows efficient in-memory structures  
Avro - JavaScript, Ruby, or Pythons which is not much point in generating code, no compile-time type checker to satisfy  
Moreover, in case of dynamically generated schema, code generation is an unnecessary obstacle to getting to the data.  
Avro file is _self-describing_ since it includes all the necessary metadata.  

### The Merits of Schemas

Although textual data formats such as JSON, XML, and CSV are widespread, binary encodings have a number of nice properties:

- Much more compact than the various "binary JSON"
- The schema is required for decoding - you can ensure that it is up to date
- Allows you to check forward and backward compatibility of schema changes
- For statically typed programming languaegs users, the ability to generate code from the schema is useful, since it enables type checking at compile time.

<br/>

## Modes of Dataflow

### Dataflow Through Databases

### Dataflow Through Services: REST and RPC

### Message Passing Dataflow


