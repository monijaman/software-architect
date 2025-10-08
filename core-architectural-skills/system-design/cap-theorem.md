# ⚖️ CAP Theorem & Trade-offs

In any distributed system that can experience partitions, you can only fully provide at most two of: Consistency (C), Availability (A), and Partition Tolerance (P). Partition tolerance is non-negotiable, so practical systems choose between stronger C or stronger A during partitions.

## Definitions
- Consistency: Every read receives the most recent write (linearizability).
- Availability: Every request receives a non-error response (not necessarily latest data).
- Partition Tolerance: System continues operation despite network partitions.

## CP vs AP Examples
- CP: Prefer consistency over availability under partition (e.g., majority-quorum stores, ZooKeeper).
- AP: Prefer availability with eventual consistency (e.g., Dynamo-style systems, Cassandra with suitable settings).

## Beyond CAP: PACELC
- If Partition (P): choose between Availability (A) or Consistency (C)
- Else (no partition): choose between Latency (L) or Consistency (C)

## Consistency Models
- Strong, Sequential, Causal, Read-your-writes, Eventual
- Tunable consistency via quorum reads/writes

## Design Guidance
- Identify domain operations needing strong consistency vs acceptable staleness
- Use idempotency and compensating actions where eventual consistency is used
- Consider user experience impacts (loading states, conflict UI)

## Checklist
- Map business invariants to required consistency
- Choose quorum/replication configs accordingly
- Instrument and test partition scenarios
