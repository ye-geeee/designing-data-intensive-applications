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

<br/>

## Linearizability

### What Makes a System Linearizable?

### Relying on Linearizabilty

### Implementing Linearizable Systems

### The Cost of Linearizability

<br/>

## Ordering Guarantees

### Ordering and Causality

### Sequence Number Ordering