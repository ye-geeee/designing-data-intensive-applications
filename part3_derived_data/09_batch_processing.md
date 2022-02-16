# Chapter 9. Batch Processing

1. [Batch Processing with Unix Tools](#Batch-Processing-with-Unix-Tools)
    - [Simple Log Analysis](#Simple-Log-Analysis)
    - [The Unix Philosophy](#The-Unix-Philosophy)
2. [MapReduce and Distributed Filesystems](#MapReduce-and-Distributed-Filesystems)
    - [MapReduce Job Execution](#MapReduce-Job-Execution)
    - [Reduce Side Joins and Grouping](#Reduce-Side-Joins-and-Grouping)
    - [Map Side Joins](#Map-Side-Joins)
    - [The Output of Batch Workflows](#The-Output-of-Batch-Workflows)
    - [Comparing Hadoop to Distributed Databases](#Comparing-Hadoop-to-Distributed-Databases)
3. [Beyond MapReduce](#Beyond-MapReduce)
    - [Materialization of Intermediate State](#Materialization-of-Intermediate-State)
    - [Graphs and Iterative Processing](#Graphs-and-Iterative-Processing)
    - [High Level APIs and Languages](#High-Level-APIs-and-Languages)

<br/>

Three different type fo systems

1. _Services (online systems)_
   - primary measure of performance : **response time**
   - availability is often very important
2. _Batch processing systems (offline systems)_
   - primary measure of performance : **throughput**
   - takes a large amount of input data, runs a _job_ to process it, and produces some output data
3. _Stream processing systems (near-real-time systems)_
   - _near-real-time_ or _nearline_ processing
   - operates on events shortly after they happen -> lower latency

Batch processing is an important building block in our quest to build reliable, scalable, and maintainable applications. 
In this chapter, we will look at MapReduce and several other batch processing algorithms and frameworks, 
and explore how they are used in modern data systems.  

## Batch Processing with Unix Tools

### Simple Log Analysis

```script
cat /var/log/nginx/access.log |
   awk '{print $7}'
   sort
   uniq -c
   sort -r -n
   head -n 5
```

Those command line is incredibly powerful.  
It will process gigabytes of log files in a matter of seconds, 
and you can easily modify the analysis to suit your needs.  

#### Chain of commands versus custom program

Custom program programmed with hash table is not as concise as the chain of Unix pipes, 
but it's fairly readable, and which of the two you prefer is partly a matter of taste.

There is a big difference in the execution flow, 
which becomes apparent if you run this analysis on a large file.  

#### Sorting versus in-memory aggregation

If the _working set_ of the job(amount of memory to which the job needs random access) is small enough, 
an in-memory hash table works fine.  

On the other hand, if the job's working set is larger than the available memory, 
the sorting approach has the advantage that it can make efficient use of disks.  

It's the same principle in "SSTables and LSM-Trees":  
chunks of data can be sorted in memory and written out to disk as segment files, 
and then multiple sorted segments can be merged into a larger sorted file.  

The **sort** utility in GNU Coreutils automatically handles larger-than-memory datasets by spilling to disk, 
and automatically parallelize sorting across multiple CPU cores.  
Therefore, the simple chain of Unix commands we saw earlier easily scales to large datasets, without running out of memory.  

### The Unix Philosophy

#### A uniform interface

#### Separation of logic and writing

#### Transparency and experimentation

## MapReduce and Distributed Filesystems

### MapReduce Job Execution

#### Distributed execution of MapReduce

#### MapReduce workflows

### Reduce Side Joins and Grouping

#### Example: analysis of user activity events

#### Sort-merge joins

#### Bringing related data together in the same place

#### GROUP BY

#### Handling skew

### Map Side Joins

#### Broadcast hash joins

#### Partitioned hash joins

#### Map-side merge joins

#### MapReduce workflows with map-side joins

### The Output of Batch Workflows

#### Building search indexes

#### Key-value stores as batch process output

#### Philosophy of batch process outputs

### Comparing Hadoop to Distributed Databases

#### Diversity of storage

#### Diversity of processing models

#### Designing for frequent faults

## Beyond MapReduce

### Materialization of Intermediate State

#### Dataflow engines

#### Fault tolerance

#### Discussion of materialization

### Graphs and Iterative Processing

#### The Pregel processing model

#### Fault tolerance

#### Parallel execution

### High Level APIs and Languages

#### The move toward declarative query languages

#### Specialization for different domains
