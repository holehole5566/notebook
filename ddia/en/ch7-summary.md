# Chapter 7 Summary: Sharding

## Core Theme
Splitting large datasets across multiple machines (shards/partitions) to achieve horizontal scalability when data volume or write throughput exceeds single-machine capacity.

## Pros and Cons of Sharding

### Benefits
- **Horizontal scaling**: Add more machines instead of bigger machines
- **Distribute load**: Spread data and queries across multiple nodes
- **Geographic distribution**: Place data closer to users

### Challenges
- **Complexity**: Partition key selection, rebalancing, routing
- **Cross-shard operations**: Joins and transactions become difficult
- **Hot spots**: Uneven load distribution (skewed workloads)

### Sharding for Multitenancy
- **Per-tenant shards**: Each customer gets separate shard
- **Resource isolation**: One tenant's load doesn't affect others
- **Compliance**: Data residency and privacy requirements
- **Backup/restore**: Per-tenant operations

## Sharding Strategies

### Key Range Sharding
**Concept**: Assign contiguous ranges of keys to each shard

**Advantages**:
- Range queries are efficient
- Keys stored in sorted order within shards
- Natural for time-series data

**Disadvantages**:
- Hot spots with sequential writes (e.g., timestamps)
- Manual boundary management
- Expensive shard splitting

**Examples**: HBase, MongoDB, CockroachDB

### Hash-Based Sharding

#### Hash Modulo N
- Simple: `shard = hash(key) % N`
- Problem: Adding/removing nodes requires massive data movement

#### Fixed Number of Shards
- Create many more shards than nodes (e.g., 1000 shards, 10 nodes)
- Assign multiple shards per node
- Rebalancing moves entire shards between nodes
- **Challenge**: Choosing right number of shards upfront

#### Hash Range Sharding
- Combine hashing with range assignment
- Each shard handles range of hash values
- Can split shards as needed
- **Examples**: DynamoDB, YugabyteDB

#### Consistent Hashing
- Minimize key movement when nodes added/removed
- **Algorithms**: Rendezvous hashing, Jump consistent hash
- **Cassandra/ScyllaDB**: Multiple ranges per node with random boundaries

### Handling Skewed Workloads

**Hot Key Problems**:
- Celebrity users in social media
- Popular products in e-commerce
- Trending topics

**Solutions**:
- **Key splitting**: Add random suffix to hot keys
- **Dedicated shards**: Isolate hot keys
- **Application-level**: Cache hot data
- **Heat management**: Automatic hot shard detection and splitting

## Request Routing

### Routing Approaches
1. **Client contacts any node**: Node forwards to correct shard
2. **Routing tier**: Dedicated load balancer with shard awareness  
3. **Smart clients**: Client knows shard assignments

### Service Discovery
- **ZooKeeper/etcd**: Centralized coordination service
- **Gossip protocols**: Peer-to-peer state dissemination
- **DNS**: For less frequently changing assignments

### Coordination Challenges
- **Split brain**: Multiple coordinators making conflicting decisions
- **Failover**: Handling coordinator failures
- **Consistency**: Ensuring all nodes have same view of assignments

## Secondary Indexes and Sharding

### Local Secondary Indexes
- **Document-partitioned**: Each shard maintains indexes for its data only
- **Scatter/gather queries**: Must query all shards for non-partition-key lookups
- **Examples**: MongoDB, Cassandra, Elasticsearch

### Global Secondary Indexes
- **Term-partitioned**: Index entries distributed across shards
- **Faster queries**: Direct lookup without scatter/gather
- **Slower writes**: Must update multiple shards
- **Consistency challenges**: Index updates may lag behind data

## Rebalancing Strategies

### Manual Rebalancing
- **Administrator-controlled**: Explicit shard movement decisions
- **Predictable**: No surprise performance impacts
- **Operational overhead**: Requires human intervention

### Automatic Rebalancing
- **System-controlled**: Automatic shard splitting and movement
- **Convenient**: Adapts to load changes automatically
- **Risks**: Can cause cascading failures during high load

### Rebalancing Triggers
- **Size-based**: Split when shard exceeds size threshold
- **Load-based**: Split when write/read throughput too high
- **Time-based**: Periodic rebalancing operations

## Operational Considerations

### Monitoring and Alerting
- **Shard size distribution**: Detect imbalanced shards
- **Query patterns**: Identify hot spots and inefficient queries
- **Cross-shard operations**: Monitor expensive scatter/gather queries

### Schema Design
- **Partition key selection**: Critical for even distribution
- **Denormalization**: Reduce cross-shard queries
- **Hierarchical data**: Keep related data in same shard

### Failure Handling
- **Shard unavailability**: Graceful degradation strategies
- **Partial failures**: Handle when some shards are down
- **Data consistency**: Maintain consistency during failures

## Key Insights

### Partition Key Selection
- **Even distribution**: Avoid hot spots
- **Query patterns**: Enable efficient access patterns
- **Stability**: Minimize future rebalancing needs

### Trade-offs
- **Complexity vs. scalability**: Sharding adds operational complexity
- **Consistency vs. performance**: Cross-shard operations are expensive
- **Flexibility vs. efficiency**: Generic solutions vs. workload-specific optimizations

### Best Practices
- **Start simple**: Use single node until you need sharding
- **Plan for growth**: Choose partition keys that scale
- **Monitor continuously**: Watch for hot spots and imbalances
- **Test rebalancing**: Ensure rebalancing works under load

### Common Pitfalls
- **Premature sharding**: Adding complexity before it's needed
- **Poor partition key choice**: Leading to hot spots
- **Ignoring cross-shard queries**: Making common operations expensive
- **Inadequate monitoring**: Not detecting problems early

Sharding is a powerful technique for scaling databases, but it requires careful planning and ongoing operational attention to be successful.