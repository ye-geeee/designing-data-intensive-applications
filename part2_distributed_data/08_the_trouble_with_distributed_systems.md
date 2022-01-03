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

Two philosophies on how to build large-scale computing systems:

- _high-performance computing_(HPC) - Super computers with thousands of CPUs, works like a single-node
- _cloud computing_ - multi-tenant datacenters, commodity computers connected with an IP network, elastic/on-demand resource allocation, and metered billing

**Implementing internet systems** compare to supercomputers:

- _online_: they need to be able to serve users with low latency at any time. Making the service unavailable is not accepted. 
- Provide equivalent performance as supercomputers at lower cost to due to economies of scale, but also have higher failure rates.
  - supercomputers: specialized hardware, each node quite reliable, nodes communicate through shared memory and remote memory access(RDMA)
- Based on IP and Ethernet, arranged int Clostopologies to provide high bisection bandwidth.
  - supercomputers: use specialized network topologies such as multi-dimensional meshes and toruses
- The bigger a system gets, the more likely it is that one of its components is broken.
- If the system can tolerate failed nodes and still keep working as a whole, that is a very useful feature for operations and maintenance.  

If we want to make distributed systems, work, we must accept the possibility of partial failure 
and build fault-tolerance mechanisms into the software.  
Thus, we need to build a reliable system from unreliable components.  
It is important to consider a wide range of possible faults.  
In distributed systems, suspicion, pessimism, and paranoia pay off.  

### Building a Reliable System from Unreliable Components

Although the system can be more reliable than its underlying parts,  
there is always a limit to how much more reliable it can be.  

- Error-correcting allow digital data to be transmitted accurately across a communication channel(radio interference), 
  but can only deal with a small number of single-bits.  
- TCP provides a more reliable transport layer. It can hide packet loss, duplication, and reordering from you, 
  but cannot magically remove delays in network.

Although the more reliable higher-level system is not perfect,  
it's still useful because it takes care of the tricky low-level faults.  

## Unreliable Networks

Share-nothing has become the dominant approach for building internet services, for several reasons:

- comparatively cheap because it requires no special hardware
- high reliability through redundancy across multiple geographically distributed datacenters

The internet and most internal networks are _asynchronous packet networks_.  
Therefore, there are many things that could go wrong.

1. Your request lost
2. Your request in queue and delivered later
3. Remote node failed
4. Remote node temporarily stopped responding(ex. experiencing a long garbage collection pause) and responds later
5. Remote node have processed your request, but response lost
6. Remote node have processed your request, but response has been delayed

If you send a request to another node and don't receive response, it is _impossible_ to tell why.  
The usual way of handling this issue is a _timeout_.  
However, when a timeout occurs, you still don't know whether the remote node got your request or not.  

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