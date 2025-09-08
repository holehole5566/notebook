# Chapter 10 Summary: Consistency and Consensus

## Core Theme
Understanding linearizability, logical clocks, and consensus algorithms to build strongly consistent, fault-tolerant distributed systems.

## Linearizability

### Definition and Properties
**Linearizability** (atomic consistency, strong consistency):
- Makes replicated system appear as single copy of data
- All operations appear atomic and instantaneous
- **Recency guarantee**: reads return most recent written value
- If operation A completes before B starts, B must see A's effects

### Key Characteristics
- **Single register semantics**: Operations on individual objects
- **Real-time ordering**: Respects actual time ordering of operations
- **Atomic point**: Each operation takes effect at some point between start/end
- **Sequential consistency**: All nodes see same order of operations

### Linearizability vs Serializability
**Serializability**:
- Transaction isolation property
- Multiple objects per transaction
- Allows stale reads
- Order can differ from actual execution order

**Linearizability**:
- Single object guarantee
- Recency requirement
- Must respect real-time ordering
- No transaction grouping

**Strict Serializability**: Combines both properties

### Use Cases for Linearizability

**Locking and Leader Election**:
- Distributed locks require linearizable compare-and-set
- Leader election needs atomic decision
- Prevents split-brain scenarios
- ZooKeeper, etcd provide linearizable coordination

**Uniqueness Constraints**:
- Username/email uniqueness
- Bank account balance constraints
- Inventory management
- Seat booking systems

**Cross-Channel Timing Dependencies**:
- Multiple communication channels (database + message queue)
- Race conditions between channels
- Example: file upload + transcoding notification

### Implementing Linearizable Systems

**Single-Leader Replication** (potentially linearizable):
- Leader has primary copy
- Requires knowing true leader
- Split-brain violates linearizability
- Asynchronous replication loses durability

**Consensus Algorithms** (linearizable):
- Automatic leader election
- Prevent split-brain
- ZooKeeper (Zab), etcd (Raft)
- Must check leadership for reads

**Multi-Leader Replication** (not linearizable):
- Concurrent writes on multiple nodes
- Conflicting writes require resolution
- Asynchronous replication

**Leaderless Replication** (probably not linearizable):
- Quorum reads/writes insufficient
- Clock-based conflict resolution fails
- Variable network delays cause races
- Read repair needed for linearizability

### Linearizability and Quorums
- Quorums (w + r > n) don't guarantee linearizability
- Network delays create race conditions
- Synchronous read repair required
- Performance penalty for linearizability

### Cost of Linearizability

**CAP Theorem**:
- **CP**: Consistent under partitions (unavailable during splits)
- **AP**: Available under partitions (not linearizable)
- Network partitions force choice between consistency and availability
- CAP has limited practical value

**Performance Impact**:
- Linearizability is slow even without faults
- Response time proportional to network delay uncertainty
- Multi-core CPUs sacrifice linearizability for performance
- Weaker consistency models much faster

## ID Generators and Logical Clocks

### Single-Node ID Generator Problems
- Single point of failure
- Geographic latency
- Throughput bottleneck
- Need distributed alternatives

### Distributed ID Generation Approaches

**Sharded ID Assignment**:
- Multiple nodes generate different ranges
- Loses ordering property
- Example: even/odd number assignment

**Preallocated Blocks**:
- Nodes claim ID ranges
- Still loses correct ordering
- Reduces coordination overhead

**Random UUIDs**:
- No coordination required
- 128-bit identifiers
- Random ordering (no temporal information)
- Very low collision probability

**Wall-Clock Timestamps**:
- Timestamp in most significant bits
- Additional uniqueness bits
- Clock skew affects ordering
- Examples: Snowflake, ULIDs, ObjectIDs

### Logical Clocks

**Purpose**:
- Count events, not time
- Provide causal ordering
- Compact and unique timestamps
- Total ordering of events

**Requirements**:
- Compact size
- Unique timestamps
- Total ordering
- Causal consistency

### Lamport Timestamps

**Algorithm**:
- Each node has (counter, node_id) pair
- Increment counter for each operation
- Update counter when seeing higher timestamp
- Compare by counter first, then node ID

**Properties**:
- Causally consistent ordering
- Total ordering of all events
- Compact representation
- No physical time relation

**Limitations**:
- No physical time information
- Concurrent events get arbitrary ordering
- Counters can diverge significantly

### Hybrid Logical Clocks

**Combines**:
- Physical time (seconds/microseconds)
- Logical ordering guarantees
- Monotonic advancement

**Benefits**:
- Approximate physical time
- Causal consistency
- Bounded drift from physical time
- Used by CockroachDB

### Vector Clocks vs Logical Clocks
- **Vector clocks**: Detect concurrency, larger timestamps
- **Logical clocks**: Total ordering, compact timestamps
- **Trade-off**: Concurrency detection vs space efficiency

### Linearizable ID Generators

**Problem with Logical Clocks**:
- Don't provide linearizability
- Can't ensure global ordering without communication
- Example: privacy settings race condition

**Solutions**:
- Single-node ID generator with replication
- Batch allocation for efficiency
- Google Spanner approach with uncertainty intervals
- Requires consensus for fault tolerance

## Consensus

### The Consensus Problem
**Examples**:
- Leader election
- Atomic compare-and-set
- Distributed locks
- Uniqueness constraints

**Why Difficult**:
- Single node solutions don't scale
- Fault tolerance requirements
- Network partitions and delays
- Split-brain prevention

### Consensus Properties

**Uniform Agreement**: No two nodes decide differently
**Integrity**: Can't change decision once made
**Validity**: Decided value was proposed by some node
**Termination**: Non-crashed nodes eventually decide

**Fault Tolerance**: Requires majority of nodes operational

### Forms of Consensus

#### Single-Value Consensus
- Multiple nodes propose values
- Algorithm decides on one value
- Used for leader election, locks
- Equivalent to atomic compare-and-set

#### Shared Logs (Total Order Broadcast)
**Properties**:
- **Eventual append**: Requested values eventually appear
- **Reliable delivery**: All nodes see all entries
- **Append-only**: Immutable ordering
- **Agreement**: Same sequence on all nodes
- **Validity**: Only proposed values appear

**Implementation**:
- Separate consensus per log slot
- Retry failed proposals in later slots
- Sequential delivery of decided entries

#### Atomic Commitment
**Two-Phase Commit Problems**:
- Coordinator single point of failure
- Blocking if coordinator fails
- All participants must vote yes

**Consensus-Based Solution**:
- All nodes exchange votes
- Propose "commit" only if all voted yes
- Consensus decides final outcome
- Fault-tolerant coordinator

### Consensus Algorithms

**Common Algorithms**:
- **Paxos**: Single-value consensus, Multi-Paxos for logs
- **Raft**: Shared log with leader election
- **Viewstamped Replication**: Primary-backup with view changes
- **Zab**: ZooKeeper's atomic broadcast

**Common Structure**:
1. **Leader Election**: Quorum votes for leader with epoch number
2. **Proposal Voting**: Quorum votes on each log entry
3. **Epoch Numbers**: Higher epochs override lower ones
4. **Quorum Overlap**: Election and proposal quorums must intersect

### Consensus in Practice

#### State Machine Replication
- Log entries represent state changes
- All replicas apply same changes in same order
- Deterministic execution ensures consistency
- Used for event sourcing, serializable transactions

#### From Single-Leader to Consensus
- Traditional failover requires manual intervention
- Consensus provides automatic failover
- Prevents split-brain with epoch numbers
- Two rounds of voting: leader election + proposals

#### Subtleties and Challenges
- New leader must honor old leader's committed entries
- Different approaches: Raft vs Paxos leader requirements
- Linearizable reads need quorum confirmation
- Reconfiguration for adding/removing nodes

### Pros and Cons of Consensus

**Benefits**:
- Automatic failover
- No data loss
- No split-brain
- Strong consistency guarantees

**Costs**:
- Requires strict majority (3 nodes for 1 failure)
- Every operation needs quorum communication
- Can't improve throughput by adding nodes
- Sensitive to network delays and timeouts
- Geographic distribution challenges

### Coordination Services

**Examples**: ZooKeeper, etcd, Consul

**Features**:
- **Locks and Leases**: Atomic compare-and-set operations
- **Fencing Support**: Monotonic tokens prevent zombie processes
- **Failure Detection**: Session timeouts and heartbeats
- **Change Notifications**: Subscribe to key changes
- **Small Data**: Memory-resident with disk persistence

**Use Cases**:
- **Leader Election**: Single leader among multiple nodes
- **Work Allocation**: Assign shards to nodes
- **Service Discovery**: Find service endpoints
- **Configuration Management**: Distribute settings

**Design Principles**:
- Fixed small cluster (3-5 nodes)
- Slow-changing data (minutes/hours)
- High availability for reads (observers)
- Strong consistency for critical operations

## Key Insights

### Fundamental Trade-offs
- **Consistency vs Availability**: CAP theorem limitations
- **Performance vs Consistency**: Linearizability is expensive
- **Simplicity vs Fault Tolerance**: Single node vs distributed

### Design Principles
- **Use Consensus Sparingly**: Only when linearizability needed
- **Separate Concerns**: Coordination service + application data
- **Batch Operations**: Reduce consensus overhead
- **Timeout Tuning**: Balance detection speed vs false positives

### Practical Considerations
- **Geographic Distribution**: Consensus across regions is slow
- **Network Sensitivity**: Algorithms vary in robustness
- **Testing Importance**: Formal verification and fault injection
- **Alternative Approaches**: Weaker consistency when appropriate

### The Power of Consensus
- **Equivalence**: Many problems reduce to consensus
- **Foundation**: Enables higher-level distributed abstractions
- **Theory Meets Practice**: Proven algorithms in production systems
- **Ongoing Research**: Improving robustness and performance