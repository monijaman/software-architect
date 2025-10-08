# 📨 Message Queues & Event Systems

Use messaging to decouple services, smooth load, and enable async workflows.

## Concepts
- Queue vs Topic (pub/sub); Point-to-Point vs Broadcast
- Consumer Groups and Scaling; Ordering guarantees (key-based)
- Delivery Semantics: At-most-once, At-least-once, Exactly-once (effects)
- Dead Letter Queues (DLQ), Retry policies, Poison messages

## Patterns
- Commands vs Events; Event-Carried State Transfer
- Outbox Pattern for atomic write + publish
- Saga/Process Manager for long-running workflows
- Deduplication and Idempotent consumers

## Broker Choices
- Kafka: Partitioned log, high throughput, retention, stream processing
- RabbitMQ: AMQP broker, routing keys, flexible topologies
- Cloud-native: SQS/SNS, Pub/Sub, Event Hubs, Service Bus

## Design Guidance
- Choose keys for ordering and locality
- Backpressure and flow control; consumer lag monitoring
- Schema management (Avro/Protobuf/JSON + schema registry)
- Security: ACLs, encryption, network isolation

## Checklist
- Define DLQ policy and retry/backoff
- Enforce idempotency and observability (correlation IDs)
- Monitor end-to-end latency and throughput
