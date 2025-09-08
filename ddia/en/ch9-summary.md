# Chapter 9 Summary: The Trouble with Distributed Systems

## Core Theme
Understanding the fundamental challenges and failure modes in distributed systems to build robust, fault-tolerant applications.

## Faults and Partial Failures

### Single Computer vs Distributed Systems

**Single Computer**:
- Deterministic behavior (same input → same output)
- Either fully functional or entirely broken
- Hardware problems cause total system failure

**Distributed Systems**:
- Partial failures are common and nondeterministic
- Some parts work while others fail unpredictably
- Must handle messy physical reality

### Types of Partial Failures
- Network partitions and delays
- Node crashes and restarts
- Process pauses (GC, virtualization)
- Clock synchronization issues
- Hardware degradation

## Unreliable Networks

### Network Failure Modes
1. **Request lost** - packet dropped or cable unplugged
2. **Request queued** - network or recipient overloaded
3. **Remote node failed** - crashed or powered down
4. **Remote node paused** - GC pause, still alive but unresponsive
5. **Response lost** - processed but reply dropped
6. **Response delayed** - will arrive later

### TCP Limitations
- Provides "reliable" delivery within limits
- Detects and retransmits dropped packets
- Cannot distinguish between lost request vs lost acknowledgment
- Connection close doesn't guarantee data processing
- Still subject to network-level failures

### Network Faults in Practice
- 12 network faults per month in medium datacenter
- Fiber cuts by animals, humans, natural disasters
- Round-trip times up to several minutes possible
- Partial connectivity (A↔B, B↔C, but A↮C)
- Brief interruptions can have lasting effects

### Detecting Faults
**Limited Feedback**:
- TCP RST/FIN packets for closed ports
- Process crash notifications (HBase approach)
- Hardware-level link failure detection
- ICMP Destination Unreachable

**Timeout Challenges**:
- Long timeout = slow failure detection
- Short timeout = false positives
- No "correct" timeout value exists
- Must balance detection speed vs accuracy

### Network Congestion and Queueing
**Queueing Points**:
- Network switch queues (congestion)
- OS receive buffers (CPU busy)
- VM monitor buffers (virtualization pauses)
- TCP flow control (sender-side queueing)

**Variability Factors**:
- Shared resources in cloud environments
- "Noisy neighbors" consuming bandwidth
- Queue buildup near capacity limits
- Exponential backoff and adaptive timeouts help

### Synchronous vs Asynchronous Networks

**Circuit-Switched (Synchronous)**:
- Fixed bandwidth allocation (telephone networks)
- Bounded delay guarantees
- Good for constant bit rate (voice/video)
- Wastes capacity for bursty traffic

**Packet-Switched (Asynchronous)**:
- Dynamic bandwidth sharing
- Unbounded delays due to queueing
- Optimal for bursty data transfers
- Better resource utilization but variable latency

## Unreliable Clocks

### Types of Clocks

**Time-of-Day Clocks**:
- Wall-clock time (calendar date/time)
- Synchronized with NTP
- Can jump backward (NTP corrections, leap seconds)
- Unsuitable for measuring elapsed time

**Monotonic Clocks**:
- Measure duration/intervals
- Always move forward
- No meaning across different computers
- Good for timeouts and performance measurement

### Clock Synchronization Problems
- Quartz drift (up to 200 ppm)
- NTP synchronization errors
- Network delays affect accuracy
- Misconfigured NTP servers
- Leap seconds cause timing assumptions to break
- VM clock virtualization issues
- Deliberate clock manipulation

### Dangerous Clock Usage

**Timestamp Ordering**:
- Last-write-wins with clock timestamps
- Clock skew can cause causality violations
- Data loss from lagging clocks
- Need logical clocks for proper ordering

**Confidence Intervals**:
- Clock readings have uncertainty ranges
- Most systems don't expose uncertainty
- Google Spanner's TrueTime API provides [earliest, latest]
- Amazon ClockBound similar approach

**Global Snapshots**:
- Spanner uses synchronized clocks for MVCC
- Waits for confidence interval before commit
- Requires GPS/atomic clocks for accuracy
- Ensures non-overlapping transaction timestamps

### Process Pauses

**Pause Causes**:
- Garbage collection (stop-the-world)
- Virtual machine suspension/migration
- Operating system context switches
- Synchronous disk I/O operations
- Memory swapping/paging
- SIGSTOP signals

**Lease Example Problem**:
```javascript
// Dangerous: assumes no long pauses
if (lease.expiryTimeMillis - System.currentTimeMillis() < 10000) {
    lease = lease.renew();
}
if (lease.isValid()) {
    process(request); // Could execute with expired lease!
}
```

**Real-Time Systems**:
- Hard deadlines with guaranteed response times
- Require RTOS, restricted languages, extensive testing
- Very expensive, used only for safety-critical systems
- Most server systems choose cheap/unreliable over expensive/reliable

**GC Impact Mitigation**:
- Modern collectors (G1, ZGC, Shenandoah)
- Treat GC pauses as planned outages
- Use non-GC languages (Rust, Swift)
- Restart processes before full GC needed

## Knowledge, Truth, and Lies

### The Fundamental Problem
- Nodes can only make guesses about other nodes
- No shared memory, only unreliable message passing
- Cannot distinguish network vs node failures
- Must assume execution can pause at any time

### System Models

**Timing Assumptions**:
- **Synchronous**: Bounded delays, pauses, clock error
- **Partially Synchronous**: Usually bounded, occasionally unbounded
- **Asynchronous**: No timing assumptions at all

**Node Failure Models**:
- **Crash-Stop**: Node crashes and never recovers
- **Crash-Recovery**: Node crashes but may restart
- **Degraded Performance**: Slow nodes, partial functionality
- **Byzantine**: Arbitrary behavior, including malicious

### The Majority Rules
- Individual nodes cannot trust their own judgment
- Decisions require quorum (majority vote)
- Prevents split-brain scenarios
- Node declared dead must accept majority decision

### Distributed Locks and Leases

**Common Problems**:
- Process pauses after acquiring lease
- Network delays in lock messages
- Multiple nodes believing they hold same lock
- Data corruption from concurrent access

**Fencing Tokens Solution**:
- Lock service provides monotonically increasing token
- Storage service rejects requests with old tokens
- Protects against both zombies and delayed messages
- Works with multiple replicas using timestamp prefixes

### Byzantine Faults

**Definition**:
- Nodes may lie or behave arbitrarily
- Includes malicious behavior, not just crashes
- Named after Byzantine Generals Problem

**When Relevant**:
- Aerospace (radiation-induced corruption)
- Multi-party systems (cryptocurrencies)
- Mutually untrusting participants

**When Not Relevant**:
- Single-organization datacenters
- Trusted internal networks
- Cost usually prohibitive for server systems

**Weak Forms of Lying**:
- Network packet corruption (checksums help)
- Input validation and sanitization
- Multiple NTP servers for outlier detection

### Safety and Liveness Properties

**Safety Properties**:
- "Nothing bad happens"
- Can point to violation moment
- Must always hold, even during failures
- Examples: uniqueness, monotonic sequence

**Liveness Properties**:
- "Something good eventually happens"
- Often include word "eventually"
- May have caveats (majority nodes alive)
- Examples: availability, eventual consistency

### Formal Methods and Testing

**Model Checking**:
- Verify algorithms using specification languages (TLA+)
- Systematically explore state space
- Find non-obvious bugs in protocols
- Used by CockroachDB, TiDB, Kafka

**Fault Injection**:
- Deliberately inject failures into running systems
- Test system behavior under various fault conditions
- Chaos engineering (Netflix Chaos Monkey)
- Jepsen framework for distributed systems testing

**Deterministic Simulation Testing (DST)**:
- Test actual code with controlled nondeterminism
- Mock network, I/O, clocks for reproducibility
- Explore many execution paths automatically
- Used by FoundationDB, TigerBeetle

## Key Insights

### Fundamental Challenges
- **Partial Failures**: Core characteristic of distributed systems
- **No Shared State**: Only unreliable message passing
- **Timing Uncertainty**: Cannot rely on synchronized time
- **Detection Difficulty**: Hard to distinguish failure types

### Design Principles
- **Assume Failures**: Design for partial failures from start
- **Timeout Carefully**: Balance detection speed vs false positives
- **Use Quorums**: Avoid single points of decision
- **Fence Zombies**: Protect against delayed/paused processes

### Testing Strategies
- **Fault Injection**: Deliberately break things to test resilience
- **Model Checking**: Verify algorithms before implementation
- **Deterministic Testing**: Control nondeterminism for reproducibility
- **Chaos Engineering**: Test in production environments

### The Power of Determinism
- **Event Sourcing**: Deterministic replay of events
- **Workflow Engines**: Deterministic execution semantics
- **State Machine Replication**: Same operations on all replicas
- **Simulation Testing**: Controlled exploration of behaviors

Distributed systems are fundamentally different from single-machine programs. Success requires embracing the messy reality of partial failures, unreliable networks, and unsynchronized clocks while building robust systems that can tolerate these inevitable problems.