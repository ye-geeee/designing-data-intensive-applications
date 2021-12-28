# Chapter 8. The Trouble with Distributed Systems

1. [Faults and Partial Failures](#Faults-and-Partial-Failures)
    - [Cloud Computing and Supercomputing](#Cloud-Computing-and-Supercomputing)
    - [Building a Reliable System from Unreliable Components](#Building-a-Reliable-System-from-Unreliable-Components)
2. [Unreliable Networks](#Unreliable-Networks)
    - [Network Faults in Practice](#Network-Faults-in-Practice)
    - [Detecting Faults](#Detecting-Faults)
    - [Timeouts and Unbounded Delays](#Timeouts-and-Unbounded-Delays)
    - [Synchronous Versus Asynchronous Networks](#Synchronous-Versus-Asynchronous-Networks)
3. [Unreliable Clocks](#Unreliable-Clocks)
    - [Monotonic Versus Time of Day Clocks](#Monotonic-Versus-Time-of-Day-Clocks)
    - [Clock Synchronization and Accuracy](#Clock-Synchronization-and-Accuracy)
    - [Relying on Synchronized Clocks](#Relying-on-Synchronized-Clocks)
    - [Process Pauses](#Process-Pauses)
4. [Knowledge, Truth, and Lies](#Knowledge,-Truth,-and-Lies)
    - [The Truth Is Defined by the Majority](#The-Truth-Is-Defined-by-the-Majority)
    - [Byzantine Faults](#Byzantine-Faults)
    - [Process Pauses](#Process-Pauses)
    - [System Model and Reality](#System-Model-and-Reality)

<br/>

## Faults and Partial Failures

With a single computer, buggy software is mostly just a consequence of badly written software.  
When the hardware is working correctly, the same operation always produces the same result - _deterministic_. 

In distributed systems, we are no longer operating in an idealized system model.  
There may well be some parts of the system that are broken in some unpredictable way, 
even though other parts of the system are working find - _partial failure_.
The difficulty of partial failures are _nondeterministic_ - it may sometimes work and sometimes unpredictable fail.

### Cloud Computing and Supercomputing

### Building a Reliable System from Unreliable Components

## Unreliable Networks

### Network Faults in Practice

### Detecting Faults

### Timeouts and Unbounded Delays

### Synchronous Versus Asynchronous Networks

## Unreliable Clocks

### Monotonic Versus Time of Day Clocks

### Clock Synchronization and Accuracy

### Relying on Synchronized Clocks

### Process Pauses

## Knowledge, Truth, and Lies

### The Truth Is Defined by the Majority

### Byzantine Faults

### System Model and Reality