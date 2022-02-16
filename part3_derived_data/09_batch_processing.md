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

## Batch Processing with Unix Tools

### Simple Log Analysis

#### Chain of commands versus custom program

#### Sorting versus in-memory aggregation

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
