# Chapter 2 Summary: Defining Nonfunctional Requirements

## Core Theme
Beyond functionality, systems must meet nonfunctional requirements: performance, reliability, scalability, and maintainability.

## Case Study: Social Network Home Timelines

**Problem**: Displaying recent posts from followed users at scale (500M posts/day, 5,700 posts/sec average)

**Approach 1 - Query on Read**: 
- Join posts, follows, users tables on each timeline request
- Problem: 2M timeline queries/sec → 400M post lookups/sec

**Approach 2 - Materialized Timelines**:
- Pre-compute timelines, insert posts into followers' timelines (fan-out)
- Trade-off: More work on write (1M timeline writes/sec) vs faster reads
- Challenge: Celebrity users with millions of followers

## Performance Metrics

### Response Time vs Throughput
- **Response Time**: Time from request to response (user-facing)
- **Throughput**: Requests/data per second (determines resource needs)
- **Relationship**: Response time increases as throughput approaches capacity (queueing)

### Measuring Performance
- **Mean**: Useful for throughput estimation, poor for user experience
- **Percentiles**: Better for understanding user experience
  - **Median (p50)**: Half of requests faster than this
  - **p95, p99, p999**: Tail latencies affecting valuable customers
- **Tail Latency Amplification**: One slow backend call slows entire request

## Reliability and Fault Tolerance

### Key Concepts
- **Fault**: Component stops working (disk failure, machine crash)
- **Failure**: System stops providing required service
- **Fault Tolerance**: Continue service despite faults
- **Single Point of Failure (SPOF)**: Component whose failure causes system failure

### Types of Faults
**Hardware Faults**:
- 2-5% disk failure rate/year
- 0.5-1% SSD failure rate/year
- Memory corruption, CPU errors
- Solution: Redundancy, distributed systems

**Software Faults**:
- Highly correlated (same bug affects all nodes)
- Harder to anticipate than hardware faults
- Examples: Leap second bugs, resource exhaustion, cascading failures

**Human Errors**:
- Leading cause of outages (configuration changes)
- Solution: Blameless postmortems, better tooling, gradual rollouts

## Scalability

### Scaling Approaches
- **Vertical Scaling (Scale Up)**: More powerful single machine
- **Horizontal Scaling (Scale Out)**: Distributed system with multiple nodes
- **Shared-Nothing Architecture**: Each node has own CPU, RAM, disk

### Scalability Principles
- Break systems into independent components
- No "magic scaling sauce" - architecture must fit specific load patterns
- Plan for one order of magnitude growth at a time
- Keep things simple when possible

## Maintainability

### Three Key Aspects

**Operability**:
- Make routine tasks easy for operations teams
- Good monitoring, documentation, predictable behavior
- Self-healing where appropriate, manual control when needed

**Simplicity**:
- Manage complexity through good abstractions
- Avoid "big ball of mud" architecture
- Essential vs accidental complexity

**Evolvability**:
- Make changes easy as requirements evolve
- Minimize irreversible actions
- Loosely-coupled systems easier to modify

## Key Insights
- Performance optimization requires understanding your specific load patterns
- Fault tolerance is about graceful degradation, not preventing all faults
- Scalability is not one-dimensional - must consider specific growth patterns
- Maintainability requires upfront investment but pays long-term dividends