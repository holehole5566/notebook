# Chapter 2: Back-of-the-Envelope Estimation

## Key Concepts

**Back-of-the-envelope estimation** is used in system design interviews to estimate system capacity and performance requirements using thought experiments and common performance numbers.

## Core Knowledge Areas

### Power of Two
- Essential for calculating data volumes in distributed systems
- Basic units: byte (8 bits), ASCII character (1 byte)

### Latency Numbers
- Memory is fast, disk is slow
- Avoid disk seeks when possible
- Simple compression algorithms are fast
- Compress data before internet transmission
- Inter-region data center communication takes time

### Availability Numbers
- High availability measured as percentage (99%-100%)
- SLA (Service Level Agreement) defines uptime commitments
- Major cloud providers target 99.9%+ availability
- "Nines" correlation: more nines = less downtime

## Example: Twitter Estimation

### Assumptions
- 300M monthly active users, 50% daily usage
- 2 tweets/day average, 10% contain media
- 5-year data retention

### Results
- Daily active users: 150M
- Tweet QPS: ~3,500 (peak: ~7,000)
- Media storage: 30TB/day, ~55PB over 5 years

## Best Practices

- Use rounding and approximation
- Document assumptions
- Label units clearly
- Practice common estimations (QPS, storage, servers)
- Focus on problem-solving process over exact results