# Chapter 8 Summary: Transactions

## Core Theme
Transactions provide atomicity, consistency, isolation, and durability (ACID) to simplify error handling and concurrency control in database systems.

## What Is a Transaction?

### ACID Properties

**Atomicity**:
- All writes in a transaction succeed or all fail (abort/rollback)
- Enables safe retry on failure
- Better term might be "abortability"

**Consistency**:
- Application-specific invariants must be preserved
- Database enforces declared constraints
- Application responsible for complex invariants

**Isolation**:
- Concurrent transactions don't interfere with each other
- Ideally serializable (same result as serial execution)
- Often implemented with weaker guarantees for performance

**Durability**:
- Committed data survives crashes and power failures
- Single-node: write-ahead log + fsync
- Distributed: replication across nodes

### Single vs Multi-Object Operations

**Single-Object**:
- Atomicity via crash recovery logs
- Isolation via object-level locks
- Atomic operations (increment, compare-and-set)

**Multi-Object**:
- Required for foreign keys, denormalized data, secondary indexes
- Grouped by client connection (BEGIN/COMMIT)
- More complex error handling without transactions

## Weak Isolation Levels

### Read Committed

**Guarantees**:
- No dirty reads (only see committed data)
- No dirty writes (only overwrite committed data)

**Implementation**:
- Row-level locks prevent dirty writes
- MVCC prevents dirty reads (old/new value approach)

**Problems**:
- Read skew (nonrepeatable reads)
- Lost updates still possible

### Snapshot Isolation

**Concept**:
- Each transaction sees consistent snapshot from transaction start
- Prevents read skew and phantom reads in read-only queries

**MVCC Implementation**:
- Multiple versions of each row stored
- Transaction IDs track which version to read
- Visibility rules determine what each transaction sees

**Visibility Rules**:
1. Ignore writes from in-progress transactions at start
2. Ignore writes from later transactions
3. Ignore writes from aborted transactions
4. All other writes are visible

**Benefits**:
- Readers never block writers, writers never block readers
- Long-running read queries don't affect writes
- Consistent backups and analytics

### Lost Updates Prevention

**Atomic Operations**:
- Database-provided increment, update operations
- Best solution when applicable

**Explicit Locking**:
- SELECT FOR UPDATE to lock rows
- Application controls read-modify-write cycle

**Automatic Detection**:
- Database detects lost updates and aborts transaction
- Available in PostgreSQL, Oracle, SQL Server

**Compare-and-Set**:
- Update only if value unchanged since read
- Optimistic locking approach

### Write Skew and Phantoms

**Write Skew Pattern**:
1. Read data to check precondition
2. Make decision based on read result
3. Write to database (different objects than read)

**Examples**:
- Doctor on-call scheduling
- Meeting room booking
- Username claiming
- Double-spending prevention

**Phantom Problem**:
- Write changes result of search query
- SELECT FOR UPDATE can't lock non-existent rows
- Requires serializable isolation or explicit locks

**Materializing Conflicts**:
- Create lock objects for phantom scenarios
- Last resort solution

## Serializability

### Actual Serial Execution

**Requirements**:
- All data fits in memory
- Short, fast transactions
- Single-threaded execution

**Stored Procedures**:
- Eliminate network round-trips
- Submit entire transaction at once
- Modern implementations use general-purpose languages

**Sharding**:
- Scale across CPU cores
- Cross-shard transactions much slower
- Requires careful data partitioning

### Two-Phase Locking (2PL)

**Principle**:
- Shared locks for reads, exclusive locks for writes
- Hold all locks until transaction end
- Readers block writers, writers block readers

**Implementation**:
- Shared mode: multiple readers allowed
- Exclusive mode: single writer only
- Lock upgrade from shared to exclusive
- Automatic deadlock detection

**Performance Issues**:
- Reduced concurrency
- Unstable latencies
- Deadlock overhead

**Predicate Locks**:
- Lock based on search conditions
- Prevent phantom reads
- Usually approximated with index-range locks

### Serializable Snapshot Isolation (SSI)

**Approach**:
- Optimistic concurrency control
- Allow transactions to proceed
- Detect conflicts at commit time

**Conflict Detection**:
1. **Stale MVCC reads**: Transaction reads data that was modified by committed transaction
2. **Writes affecting prior reads**: Transaction modifies data that another transaction read

**Performance**:
- Better than 2PL (no blocking)
- Scalable across multiple machines
- Abort rate affects overall performance

## Distributed Transactions

### Two-Phase Commit (2PC)

**Participants**:
- Coordinator (transaction manager)
- Participants (database nodes)

**Protocol**:
1. **Prepare phase**: Coordinator asks all participants to prepare
2. **Commit phase**: If all vote yes, coordinator sends commit; otherwise abort

**Coordinator Failure**:
- Participants left "in doubt"
- Must wait for coordinator recovery
- Locks held during uncertainty

### XA Transactions

**Purpose**:
- Heterogeneous distributed transactions
- Standard API for 2PC across different systems

**Problems**:
- Single point of failure (coordinator)
- Application code as bottleneck
- Lowest common denominator limitations
- Orphaned transactions

### Database-Internal Distributed Transactions

**Advantages**:
- Replicated coordinator with automatic failover
- Direct communication between nodes
- Optimized protocols for specific system
- Consensus algorithms for fault tolerance

**Exactly-Once Processing**:
- Alternative: idempotent processing with message IDs
- Store message ID in database transaction
- Check for duplicate before processing

## Key Insights

### Isolation Trade-offs
- **Serializable**: Strongest guarantees, performance cost
- **Snapshot**: Good balance for most applications
- **Read Committed**: Minimal guarantees, best performance

### Concurrency Control Approaches
- **Pessimistic**: Block on potential conflicts (2PL)
- **Optimistic**: Detect conflicts at commit (SSI)
- **Serial**: Eliminate concurrency entirely

### Distributed Transaction Challenges
- **Heterogeneous**: Complex, fragile, poor performance
- **Homogeneous**: Viable with proper design
- **Alternatives**: Idempotence, eventual consistency

### Implementation Strategies
- **MVCC**: Multiple versions for snapshot isolation
- **Locking**: Prevent conflicts through coordination
- **Timestamps**: Order operations across nodes

The choice of transaction approach depends on consistency requirements, performance needs, and system architecture. Modern systems increasingly favor optimistic approaches like SSI or carefully designed eventual consistency over traditional pessimistic locking.