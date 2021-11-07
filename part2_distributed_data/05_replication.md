# Chapter 5. Replication

1. [Leaders and Followers](#Leaders-and-Followers)
   - [Synchronous Versus Asynchronous Replication](#Synchronous-Versus-Asynchronous-Replication)
   - [Setting Up New Folowers](#Setting-Up-New-Followers)
   - [Handling Node Outages](#Handling-Node-Outages)
   - [Implementation of Replication Logs](#Implementation-of-Replication-Logs)
2. [Problems with Replication Log](#Problems-with-Replication-Log)
   - [Reading Your Own Writes](#Reading-Your-Own-Writes)
   - [Monotonic Reads](#Monotonic-Reads)
   - [Consistent Prefix Reads](#Consistent-Prefix-Reads)
   - [Solutions for Replication Lag](#Solutions-for-Replication-Lag)
3. [Multi-Leader Replication](#Multi-Leader-Replication)
   - [Use Cases for Multi-Leader Replication](#Use-Cases-for-Multi-Leader-Replication)
   - [Handling Write Conflicts](#Handling-Write-Conflicts)
   - [Multi-Leader Replication Topologies](#Multi-Leader-Replication-Topologies)
4. [Leaderless Replication](#Leaderless-Replication)
   - [Writing to the Database When a Node Is Down](#Writing-to-the-Database-When-a-Node-Is-Down)
   - [Limitations of Quorum Consistency](#Limitations-of-Quorum-Consistency)
   - [Sloppy Quorums and Hinted Handoff](#Sloppy-Quorums-and-Hinted-Handoff) 
   - [Detecting Concurrent Writes](#Detecting-Concurrent-Writes)

<br/>

## Leaders and Followers

### Synchronous Versus Asynchronous Replication

### Setting Up New Followers

### Handling Node Outages

### Implementation of Replication Logs

## Problems with Replication Log

### Reading Your Own Writes

### Monotonic Reads

### Consistent Prefix Reads

### Solutions for Replication Lag

## Multi Leader Replication

### Use Cases for Multi Leader Replication

### Handling Write Conflicts

### Multi Leader Replication Topologies

## Leaderless Replication

### Writing to the Database When a Node Is Down

### Limitations of Quorum Consistency

### Sloppy Quorums and Hinted Handoff

### Detecting Concurrent Writes


