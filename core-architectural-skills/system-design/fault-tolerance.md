# 🛡️ Fault Tolerance, Resilience, and High Availability

Design systems to survive failures gracefully and remain available.

## Key Concepts
- Reliability vs Availability: Reliability = correct over time; Availability = up and responsive.
- Resilience: Recover quickly from failures; design for graceful degradation.
- High Availability (HA): Minimize downtime using redundancy and failover.

## Patterns and Techniques
- Timeouts, Retries, Backoff, and Jitter
- Circuit Breaker and Bulkhead Isolation
- Idempotency and Exactly-Once-Effect (at-least-once delivery + idempotent handlers)
- Load Balancing and Health Checks
- Redundancy: N+1, Multi-AZ/Region; Active-Active vs Active-Passive
- Graceful Degradation and Feature Flags
- Rate Limiting, Throttling, and Backpressure
- Chaos Engineering to validate resilience

## HA Topologies
- Active-Passive: Hot/warm standby with failover
- Active-Active: All nodes serve traffic; needs conflict handling

## Data Considerations
- Replication: Synchronous vs Asynchronous
- Quorums: Read/Write quorum configs
- Backups and Point-in-Time Recovery (PITR)
- RTO/RPO: Recovery Time/Point Objectives

## Observability
- SLO/SLI/SLA definitions
- Health endpoints, synthetic probes
- Distributed tracing for failure diagnosis

## Trade-offs
- Cost of redundancy vs availability targets
- Consistency vs availability choices (see CAP)

## Checklist
- Define SLOs; design for failure
- Enforce timeouts + circuit breakers at all network calls
- Implement idempotent operations
- Regularly run game days/chaos tests
