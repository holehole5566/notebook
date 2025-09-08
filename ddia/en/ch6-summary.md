# Chapter 6 Summary: Replication

## Core Theme
Keeping copies of the same data on multiple machines connected via network for latency reduction, availability, and read throughput scaling.

## Single-Leader Replication

### Architecture
- **Leader (Primary)**: Accepts all writes, processes them locally first
- **Followers (Replicas)**: Receive replication log from leader, apply changes in same order
- **Reads**: Can be served by leader or any follower
- **Writes**: Only accepted by leader

### Synchronous vs Asynchronous Replication
**Synchronous**:
- Leader waits for follower confirmation before reporting success
- Guarantees up-to-date copy on follower
- Risk: Any follower failure blocks all writes

**Asynchronous**:
- Leader sends changes but doesn't wait for confirmation
- Better availability but potential data loss on leader failure
- Most systems use semi-synchronous (one synchronous + others asynchronous)

### Setting Up New Followers
1. Take consistent snapshot of leader
2. Copy snapshot to new follower
3. Follower requests all changes since snapshot
4. Follower catches up and continues processing changes

### Handling Node Outages

**Follower Failure**:
- Catch-up recovery using local log
- Request missing changes from leader
- Performance impact during catch-up

**Leader Failure (Failover)**:
1. Detect leader failure (timeout-based)
2. Choose new leader (most up-to-date replica)
3. Reconfigure system to use new leader

**Failover Problems**:
- Data loss with asynchronous replication
- Split brain scenarios
- Timeout configuration challenges

### Replication Log Implementation

**Statement-Based**:
- Forward SQL statements to followers
- Problems: Non-deterministic functions, execution order

**Write-Ahead Log (WAL) Shipping**:
- Send low-level disk block changes
- Tight coupling to storage engine

**Logical (Row-Based)**:
- Sequence of row-level changes
- Decoupled from storage engine
- Enables change data capture

## Problems with Replication Lag

### Read-After-Write Consistency
**Problem**: User may not see their own writes when reading from stale follower

**Solutions**:
- Read user's own data from leader
- Track timestamp of last write
- Route user's requests to same region

### Monotonic Reads
**Problem**: User may see data "moving backward in time" when reading from different followers

**Solution**: Ensure user always reads from same replica (or more up-to-date ones)

### Consistent Prefix Reads
**Problem**: Seeing writes in wrong order (violates causality)

**Solution**: Ensure causally related writes go to same partition

## Multi-Leader Replication

### Use Cases
- **Multi-datacenter operation**: Leader in each datacenter
- **Offline clients**: Local database acts as leader when offline
- **Collaborative editing**: Each user acts as leader for their changes

### Conflict Resolution

**Conflict Avoidance**:
- Route writes for particular record to same leader
- Not always possible (datacenter failures, user mobility)

**Last Write Wins (LWW)**:
- Assign timestamp to each write
- Problems: Clock synchronization, data loss

**Manual Resolution**:
- Store conflicting versions
- Application resolves on read or write

**Automatic Resolution**:
- CRDTs (Conflict-free Replicated Data Types)
- Operational Transformation
- Custom merge functions

### Multi-Leader Topologies
- **All-to-all**: Every leader sends changes to every other
- **Circular**: Changes flow in circle
- **Star**: One central node forwards changes

**Problems**: Write ordering, infinite loops, fault tolerance

## Leaderless Replication

### Architecture
- No designated leader
- Client sends writes to multiple replicas
- Uses quorums for consistency

### Quorum Consistency
- **W + R > N** where:
  - W = write replicas
  - R = read replicas  
  - N = total replicas
- Ensures overlap between read and write sets

### Handling Node Failures

**Read Repair**:
- Client detects stale data during reads
- Sends updates to stale replicas

**Anti-Entropy Process**:
- Background process compares replicas
- Copies missing data

### Limitations
- Not linearizable even with quorums
- Network delays can cause issues
- Monitoring staleness is difficult

### Concurrent Write Detection

**Happens-Before Relationship**:
- Operation A happens before B if B knows about A
- Concurrent operations don't know about each other

**Version Vectors**:
- Track version numbers per replica
- Determine causality and detect conflicts

## Key Insights

### Replication Trade-offs
- **Synchronous**: Strong consistency but availability risk
- **Asynchronous**: High availability but potential data loss
- **Semi-synchronous**: Balanced approach

### Consistency Models
- **Eventual consistency**: All replicas will converge eventually
- **Strong consistency**: Immediate consistency across replicas
- **Causal consistency**: Preserve cause-and-effect relationships

### Conflict Resolution Strategies
- **Avoidance**: Prevent conflicts through routing
- **Detection**: Identify conflicts and resolve them
- **CRDTs**: Data structures that merge automatically

### Performance Considerations
- **Read scaling**: Add more followers
- **Write scaling**: Requires sharding or multi-leader
- **Latency**: Geographic distribution helps
- **Consistency vs. availability**: Fundamental trade-off

The choice of replication strategy depends on your specific requirements for consistency, availability, performance, and operational complexity.