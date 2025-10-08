# 🎯 System Boundaries & Domain Decomposition

Draw the right boundaries to minimize coupling and maximize autonomy.

## Principles
- High Cohesion within a boundary; Loose Coupling between boundaries
- Team Alignment (Conway’s Law); boundaries mirror team ownership
- Data Ownership: Each boundary owns its data; avoid shared mutable DBs

## Techniques
- Domain-Driven Design (DDD): Identify Bounded Contexts (see ../architecture-patterns/domain-driven-design.md)
- Context Mapping: Customer/Supplier, Conformist, Shared Kernel, ACL
- Integration Styles: Sync (APIs) vs Async (events); choose per interaction
- Anti-Corruption Layer (ACL): Translate external models to protect your domain
- Strangler Fig: Incrementally carve services from a monolith

## Boundary Contracts
- Clear APIs and SLAs; explicit error modes and idempotency
- Versioning and backward compatibility strategies
- Event contracts for async integration

## Data & Consistency
- Transaction boundaries align with aggregates (DDD)
- Use eventual consistency with compensations where needed

## Checklist
- Map domain capabilities and identify seams
- Align teams to boundaries; avoid shared ownership
- Define contracts and governance for changes
