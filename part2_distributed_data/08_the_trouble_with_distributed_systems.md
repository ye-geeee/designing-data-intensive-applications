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

<br/>

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

Many systematic studies show that network problems can be surprisingly common.  
Public cloud services such as EC2 are notorious for having frequent transient network glitches.  
Surprisingly, faults include a network interface that sometimes drops all inbound packets but sends outbound packets successfully:  
just because a network link works in one direction doesn't guarantee it's also working in the opposite direction.  

Faults _can_ occur means that your software needs to be able to handle them.  
If the error handling of network faults is not defined and tested, the cluster could become deadlocked and permanently unable to serve requests, 
even when the network recovers.  

Handling network faults doesn't necessarily mean _tolerating_ them.  
The simple way could be to show error message to users while your network is experiencing problems.  
However, you do need to know how your software reacts to network problems and ensure that the system can recover from them.  

### Detecting Faults

Many systems need to automatically detect faulty nodes.

- A load balancer needs to stop sending requests to a node that is dead
- In a distributed database with single-leader replication, if the leader fails, one of the followers needs to be promoted to be the new leader

Followings are some specific circumstances you might get some feedback:

- TCP - If there is no process is listening on the destination port, the OS will helpfully close or refuse TCP connections by sending RST or FIN.
- HBase - If a node process is crashed, a script can notify other nodes about the crash so that another node can take over quickly without having to wait for a timeout to expire.
- If you have access to the management interface of the network switches, you can query them to detect link failures at a hardware level.
- Router - If an IP address you're trying to connect to is unreachable, the router reply to you with an ICMP Destination Unreachable packet.

If something has gone wrong, you may get an error response at some level of the stack,  
but in general you have to assume that you will get no response at all.  
you can retry a few times, wait for timeout to elapse, and eventually declare the node dead.

### Timeouts and Unbounded Delays

If a timeout is the only sure way of detecting a fault, then how long should the timeout be?  
There is unfortunately no simple answer.  

Unfortunately, most asynchronous networks have _unbounded delays_,  
and most server implementations cannot guarantee that they can handle requests withing some maximum time.  

#### Network congestion and queueing

The variability of packet delays on computer networks is most often due to queueing:

- several nodes simultaneously try to send packets to the same destination
- a packet reaches the destination machine, but all CPU cores are currently busy
- virtualized environments - os is often paused while another virtual machine uses a CPU core
- TCP performs _flow control(congestion avoidance, backpressure_. 
  TCP considers a packet is lost if there is no act within some timeout, and lost packets are automatically retransmitted. 

+ Batch workloads such as MapReduce can easily saturate network links.  

In such environments, you can only choose timeouts experimentally:  
measure the distribution of network round-trip times over an extended period, and over many machines, to determine the expected variability of delays.  
Even better, systems can continually measure response times and their variability(_jitter_), 
and automatically adjust timeouts according to the observed response time distribution.  

### Synchronous Versus Asynchronous Networks

#### Synchronous Network

- does not suffer from queueing
- maximum end-to-end latency of the network is fixed - _bounded delay_

#### Can we not simply make network delays predictable?

In case of circuit in a telephone, there is a fixed amount of reserved bandwidth which nobody can use.  
How about to give TCP a variable-sized block of data?  
If datacenter networks were circuit-switched networks, it would be possible to establish a guaranteed maximum round-trip time.  
However, Ethernet and IP the packet-switched protocols do not have the concept of circuit.  

Why do datacenter networks use packet switching?  
It's because to optimize _bursty traffic_.  
Requesting a web page, sending an email, or transferring a file doesn't have any particular bandwidth requirement 
- we just want it to complete as quickly as possible.  

Using circuits for bursty data transfers wastes network capacity and makes transfers unnecessarily slow.  
By contrast, TCP dynamically adapts the rate of data transfer to the available network capacity.  

With careful use of _quality of service_(QoS) and _admission control_, 
it is possible to emulate circuit switching on packet networks, or provide statistically bounded delay.  
However, currently it is not enabled in multi-datacenters and clouds.  
We have to assume that network congestion, queueing, and unbounded delays will happen.  

<br/>

## Unreliable Clocks

In a distributed system, time is a tricky business, because communication is not instantaneous:  
it takes time for a message to travel across the network from on machine to another.  

Moreover, each machine on the network has its own clock, which is an actual hardware device: usually a quartz crystal ocillator.  
It is possible to synchronize clocks to some degree: the most commonly used mechanism is the Network Time Protocol(NTP).  

### Monotonic Versus Time of Day Clocks

Modern computers have at least two different kinds of clocks:  
a _time-of-day clock_ and a _monotonic clock_.  
They serve different purposes.  

#### Time-of-day clocks

- returns the current data and time according to some calendar
- usually synchronized with NTP

#### Monotonic clocks

- suitable for measuring a duration
- check the difference between the start time and end time
- _absolute_ value of the clock is meaningless

On a server sith multiple CPU sockets, there may be a separate timer per CPU.  
OS compensates for discrepancy and try to present a monotonic view of the clock to application threads, 
even they are scheduled across different CPUs.  

If it detects that the computer's local quartz is moving faster or slower than the NTP server,  
NTP allows the clock rate to be speed up or slowed down by up to 0.05%,  
but NTP cannot cause the monotonic clock to jump forward or backward.

In a distributed system, 
using a monotonic clock is usually fine, because it doesn't assume any synchronization between 
different nodes' clocks and is not sensitive to slight inaccuracies of measurement.  

### Clock Synchronization and Accuracy

Monotonic clocks don't need synchronization, but time-of-day clocks need to be set according to NTP server or otehr external time source in order to be useful.  
Unfortunately, our methods aren't nearly as reliable or accurate as you might hope(Skip example...)

It is possible to achieve good clock accuracy if you care about it sufficiently to invest significant resources.  
Plus, such accuracy can be achieved using GPS receivers, the Precision Time Protocol(PTP), and careful deployment and monitoring.  

### Relying on Synchronized Clocks

Although clocks work quite well most of the time,  
robust software needs to be prepared to deal with incorrect clocks.  

If a machine's CPU is defective or its network is misconfigured, it will be quickly be noticed and fixed.  
However, if NTP client is misconfigured, clock gradually drifts further away from reality.  

Thus, if you use software that requires clocks to be synchronized, it is essential that you also carefully monitor the clock offsets between all the machines.  
Such monitoring ensures you before they can cause too much damage.  

#### Timestamps for ordering events

In both multi-leader replication and leaderless databases such as Cassandra, 
conflict resolution strategy called _last write wins_(LWW) is widely used.  
However, even though it is tempting to resolve conflicts by keeping the most "recent" value and discarding other, 
it's important to be aware that the definition of "recent" depends on a local time-of-day clock, which may waell be incorrect.  

So-called _logical clocks_, which are based on incrementing counters rather than an oscillating qurtz crystal, 
are a safer alternative for orering events.  

#### Clock readings have a confidence interval

It doesn't make sense to think of a clock reading as a point in time  
- it is more like a range of times, within a confidence interval.  

The uncertainty bound can be calculated based on your time source.  
If you have a GPS receiver or atomic (caesium) clock directly attached to your computer, 
the expected error range is reposted by the manufacturer.  

Unfortunately, most systems don't expose this uncertainty.  
An interesting exception in Google's _TrueTime_ API in Spanner, 
it explicitly reports the confidence interval_[earliest, latest]_ on the local clock.  

#### Synchronized clocks for global snapshots

If we could get the synchronization good enough,  
we can use timestamps from synchronized time-of-day clocks as transaction IDs.  
However, the problem, of course, is the uncertainty about clock accuracy.  

In case of Spanner,  
it uses the clock's confidence interval as reported by the TrueTime API.  
In order to ensure that transaction timestamps reflect causality, 
Spanner deliberately waits for the length of the confidence interval before committing a read-write transaction.  
In order to keep the wait time as short as possible, Spanner needs to keep the clock uncertainty as small as possible.  

### Process Pauses

<br/>

## Knowledge, Truth, and Lies

### The Truth Is Defined by the Majority

### Byzantine Faults

### System Model and Reality