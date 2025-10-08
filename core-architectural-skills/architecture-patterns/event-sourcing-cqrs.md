# 📝 Event Sourcing & CQRS

Event Sourcing and CQRS are complementary patterns often used together to model complex domains, improve scalability, and enable powerful auditing and temporal queries.

- Event Sourcing: Persist all state changes as a sequence of immutable domain events; rebuild state by replaying events.
- CQRS (Command Query Responsibility Segregation): Split the write model (commands) from the read model (queries) for independent scaling and optimized data models.

They’re orthogonal: you can do CQRS without Event Sourcing, and Event Sourcing without CQRS—but many systems combine them.

---

## Core Building Blocks

- Commands: Intent to change state (imperative): PlaceOrder, ApproveInvoice.
- Command Handlers: Validate and execute commands (often at the Application layer).
- Aggregates: Enforce invariants; in event-sourced systems, they apply and emit events.
- Events: Facts about what happened: OrderPlaced, InvoiceApproved (immutable, append-only).
- Event Store: Append-only log storing events per aggregate stream; supports optimistic concurrency via versioning.
- Projections (Read Models): Materialized views built by consuming event streams to answer queries efficiently.
- Queries: Read-only operations, served from read models.

---

## Typical Flow

1) Client sends a Command to the write side.
2) Command Handler loads the Aggregate (by replaying its events or from a snapshot), validates invariants, and decides new Events.
3) New Events are appended to the Event Store with an expected version (optimistic concurrency check).
4) Projectors consume Events and update Read Models (databases, caches, search indexes).
5) Clients query the Read Models for views.

```
Client → Command → CommandHandler → Aggregate → Events → Event Store
                                             ↓
                                         Projectors → Read Models → Query
```

---

## Event Sourcing Details

- State Reconstruction: Aggregate state = fold(all events) or snapshot + events since snapshot.
- Snapshots: Periodic state snapshots reduce replay cost; store snapshot version.
- Concurrency: Use expectedVersion when appending; if mismatch, reload/retry.
- Idempotency: Ensure handlers and projectors handle duplicate deliveries gracefully.
- Ordering: Per-aggregate order matters; global ordering only if needed.

### Evolving Events
- Schema Versioning: Add fields backward-compatibly; prefer additive changes.
- Upcasting: Upgrade old events on read to the latest schema.
- Deprecation: Migrate read models; keep old handlers until migration completes.

---

## CQRS Details

- Write Model: Optimized for invariant enforcement and state changes (aggregates, command validation).
- Read Model(s): One or many models optimized for query patterns (SQL, NoSQL, search engines), updated asynchronously.
- Consistency: Typically eventual between write and read sides; design UX accordingly.
- Scaling: Scale reads and writes independently; denormalize read models for speed.

---

## Process Coordination

- Sagas / Process Managers: Coordinate multi-aggregate or long-running workflows by reacting to events and issuing commands.
- Outbox Pattern: Ensure reliable event publication with the write side using the same transaction (or transactional outbox + background publisher).
- At-Least-Once Delivery: Design projectors to be idempotent; use checkpoints to resume.

---

## Pros and Cons

Benefits:
- Full audit log and time-travel (rebuild state at any time).
- Powerful debugging and analytics on domain behavior.
- Highly scalable reads with tailor-made projections.
- Natural fit for complex domains with rich workflows.

Trade-offs:
- Higher complexity (event design, versioning, projections, sagas).
- Eventual consistency between write and read sides.
- Storage growth and maintenance (snapshots, archiving, compaction).
- Steeper learning curve for teams and operations.

---

## When to Use
- Strong audit, compliance, or temporal query requirements.
- Complex aggregates and workflows; need to track how state evolved.
- Read-heavy systems with diverse query shapes.

When to Avoid or Downscale:
- Simple CRUD applications without audit/temporal needs.
- Small teams or short timelines where complexity isn’t justified.
- Where strict immediate consistency is required across many aggregates.

---

## Minimal Example (Pseudo)

```
// Events
record OrderPlaced(orderId, customerId, items)
record ItemAdded(orderId, item)

// Aggregate
class OrderAggregate {
  state = { items: [], placed: false }
  apply(e) { /* fold events into state */ }
  place(cmd) {
    if (this.state.placed) throw Error('Already placed')
    return [ new OrderPlaced(cmd.id, cmd.customerId, cmd.items) ]
  }
  addItem(cmd) {
    if (this.state.placed) throw Error('Closed')
    return [ new ItemAdded(cmd.id, cmd.item) ]
  }
}

// Command handler (write side)
function handlePlaceOrder(command, eventStore) {
  const stream = `order-${command.id}`
  const events = eventStore.readStream(stream)
  const order = new OrderAggregate()
  events.forEach(e => order.apply(e))
  const newEvents = order.place(command)
  eventStore.append(stream, newEvents, expectedVersion=events.length)
}

// Projector (read side)
function project(events, readDb) {
  for (const e of events) {
    switch (e.type) {
      case 'OrderPlaced': readDb.orders.insert({ id: e.orderId, customerId: e.customerId, items: e.items, status: 'PLACED' }); break
      case 'ItemAdded': readDb.orderItems.insert({ orderId: e.orderId, item: e.item }); break
    }
  }
}
```

---

## Implementation Notes

- Event Store Options: EventStoreDB, PostgreSQL (append-only tables), Kafka (with keys + compaction), DynamoDB streams, etc.
- Projections: Build read models with Kafka consumers, EventStoreDB subscriptions, CDC, or message brokers.
- Observability: Correlation IDs, causation IDs, and event metadata improve debugging.
- Testing: Property-based tests for aggregates; contract tests for projectors.

---

## References
- Greg Young, “Introduction to Event Sourcing” and “CQRS Documents”
- Udi Dahan, Sagas/Process Managers
- EventStoreDB docs; Martin Fowler articles on Event Sourcing and CQRS
