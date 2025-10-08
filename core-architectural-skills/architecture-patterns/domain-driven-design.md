# 🎨 Domain-Driven Design (DDD)

Domain-Driven Design is an approach to building software that places the business domain at the center. It emphasizes deep collaboration between domain experts and developers, a shared language, and a model that drives the design and implementation.

---

## Core Principles
- Ubiquitous Language: A shared, precise vocabulary used by developers and domain experts across code, docs, and conversations.
- Model-Driven Design: The domain model (not the database) anchors design; code reflects the model closely.
- Strategic and Tactical Design: Use high-level strategic patterns to carve boundaries and tactical patterns to model within them.

---

## Strategic DDD

### Bounded Contexts
- A bounded context is an explicit boundary within which a model applies.
- Each bounded context has its own ubiquitous language, model, and schemas.
- Avoids “one-size-fits-all” models that grow muddled over time.

### Context Mapping
Describe relationships between bounded contexts:
- Partnership: Jointly owned model/integration; high collaboration.
- Shared Kernel: Share a small, carefully curated subset of the model.
- Customer/Supplier: Downstream (customer) relies on upstream (supplier); upstream priorities impact downstream.
- Conformist: Downstream conforms to upstream model (limited influence).
- Anticorruption Layer (ACL): Protect your model from an external model by translating at the boundary.

---

## Tactical DDD

### Building Blocks
- Entities: Identified by identity, not attributes; mutable lifecycle.
- Value Objects: Immutable, equality by value; encapsulate concepts and invariants.
- Aggregates: Consistency boundaries containing entities/value objects; transactions don’t cross aggregates.
- Domain Services: Domain operations that don’t naturally belong to an entity or value object.
- Repositories: Access aggregates by identity; return domain objects, not persistence models.
- Factories: Encapsulate complex creation logic for aggregates/entities.
- Domain Events: Capture something that happened in the domain; often published for side effects.

### Aggregate Rules of Thumb
- One aggregate per transaction.
- Keep aggregates small and cohesive around invariants.
- Reference other aggregates by ID, not object references.

---

## Layering and Architecture Fit
DDD pairs well with Clean/Hexagonal/Onion architectures:
- Keep the domain model independent of frameworks (Hexagonal ports, Clean dependency rule).
- Application layer orchestrates use cases; domain layer encodes business rules.
- Use ACLs at integration boundaries to protect the model.

Example structure:
```
src/
  domain/            # Entities, Value Objects, Domain Services, Events
  application/       # Use cases, command/query handlers, DTOs
  adapters/
    in/              # HTTP controllers, message consumers
    out/             # Repositories, external service gateways (ACLs)
  infrastructure/    # Frameworks, persistence, messaging
```

---

## When to Use DDD
- Complex, evolving business domains with rich rules and workflows.
- Long-lived systems where maintainability, language, and model clarity matter.
- Multiple teams need clear boundaries and autonomy.

## When to Avoid / Downscale
- Simple CRUD apps with minimal domain logic.
- Short-lived utilities.
- Teams without capacity to invest in modeling and collaboration.

---

## Benefits
- Aligned software and business concepts.
- High maintainability through clear boundaries and ubiquitous language.
- Flexibility to evolve models per bounded context without ripple effects.

## Trade-offs & Pitfalls
- Modeling investment and learning curve.
- Over-modeling simple domains (“DDD everywhere” anti-pattern).
- Leaky integrations without proper ACLs.
- Bloated aggregates and anemic domain models.

---

## Minimal Example (Pseudo)
```
// Domain
class Money { /* value object with currency, amount, operations */ }
class Order { /* aggregate root with invariants, behaviors */ }
interface Orders { save(order: Order); byId(id): Order }

// Application
class PlaceOrder {
  constructor(private orders: Orders) {}
  execute(cmd) {
    const order = Order.place(cmd.items, cmd.customerId);
    this.orders.save(order);
    return { id: order.id };
  }
}

// Adapters
class SqlOrders implements Orders { /* repository implementation */ }
class HttpPlaceOrderController { constructor(private useCase: PlaceOrder) {} }
```

---

## References
- Eric Evans, Domain-Driven Design: Tackling Complexity in the Heart of Software
- Vaughn Vernon, Implementing Domain-Driven Design; Domain-Driven Design Distilled
- Strategic and Tactical DDD essays and talks
