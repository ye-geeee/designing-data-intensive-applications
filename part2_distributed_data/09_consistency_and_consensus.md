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

### Relying on Linearizabilty

### Implementing Linearizable Systems

### The Cost of Linearizability

<br/>

## Ordering Guarantees

### Ordering and Causality

### Sequence Number Ordering