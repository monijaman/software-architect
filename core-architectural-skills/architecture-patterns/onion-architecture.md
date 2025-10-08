# 🧅 Onion Architecture

Onion Architecture emphasizes a domain-centric design with layers forming concentric circles. Each layer depends inward, ensuring the domain model stays independent from infrastructure and UI concerns.

---

## Layers
1. Domain Model (Core): Entities, value objects, domain services — pure business.
2. Application Services: Orchestrate use cases; define interfaces for persistence, messaging, etc.
3. Interface Adapters: DTO mapping, controllers, presenters, gateways.
4. Infrastructure (Outer): Databases, web frameworks, ORMs, external services.

```
┌───────────────────────────────┐
│       Infrastructure          │
├─────────── Interface ─────────┤
│          Adapters             │
├──────── Application ──────────┤
│           Services            │
├─────────── Domain ────────────┤
│            Model              │
└───────────────────────────────┘
```

Dependency direction: Outermost → Inward. The domain never depends on infrastructure.

---

## Benefits
- Strong domain focus and isolation.
- Easy to test core logic without external systems.
- Swappable infrastructure (DBs, frameworks).

## Trade-offs
- Additional layering and indirection.
- Needs discipline in enforcing boundaries and dependency rules.

---

## Comparison to Clean/Hexagonal
- Very similar goals and rules. Differences are mostly terminology and visualization.
- Hexagonal emphasizes ports/adapters; Onion emphasizes concentric dependency rings.

---

## Example Folder Structure
```
src/
  domain/
  application/
  adapters/
  infrastructure/
```

---

## When to Use
- Complex business domains with frequent infrastructure changes.
- Long-lived systems demanding maintainability and testability.

## When to Avoid
- Small apps or prototypes where the overhead isn't justified.

---

## References
- Jeffrey Palermo, Onion Architecture
- Community articles and talks
