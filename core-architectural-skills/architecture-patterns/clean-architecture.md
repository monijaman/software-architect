# 🧼 Clean Architecture

Clean Architecture organizes software so that business rules are independent of frameworks, databases, and UI. The dependency rule states that source code dependencies point inward toward the core (entities and use cases).

---

## Key Concepts
- Entities: Enterprise-wide business rules, the most stable layer.
- Use Cases (Application Business Rules): Orchestrate application-specific behavior using entities.
- Interface Adapters: Translate data between the outside world (web, DB, UI) and the use cases/entities.
- Frameworks & Drivers: External details like the web framework, database, messaging system, UI.

### Dependency Rule
Only inner layers may be known by outer layers. Inner layers must not know about outer layers or their frameworks.

```
Frameworks & Drivers (outermost)
        ↓
  Interface Adapters
        ↓
       Use Cases
        ↓
       Entities (innermost)
```

---

## Why Use It
- Testability: Core business logic testable without web/database.
- Maintainability: Isolate volatile details; change frameworks with minimal impact.
- Flexibility: Multiple UIs or data sources can plug into the same core.

## Trade-offs
- Added indirection/boilerplate (DTOs, boundaries).
- Requires discipline in package/module boundaries.
- Overkill for very small/simple apps.

---

## Typical Structure (example)
```
src/
  domain/           # Entities, value objects, domain services
  use-cases/        # Application services / interactors
  adapters/
    in/             # Controllers, CLI, GraphQL resolvers (driving)
    out/            # DB, message brokers, external APIs (driven)
  infrastructure/   # Frameworks, HTTP, ORM, DI wiring
```

---

## Minimal Flow Example (language-agnostic)
- Controller (adapter-in) calls a UseCase input port.
- UseCase uses a Repository port (interface) to load/save entities.
- A DB adapter implements the Repository port.

```
Controller → UseCase(InputPort) → Repository(Port) → DBAdapter
```

---

## When to Use
- Medium to large codebases with evolving frameworks/IO.
- Complex domain logic that deserves isolation and heavy testing.
- Need to support multiple interfaces (REST, CLI, jobs) over the same core.

## When to Avoid
- Tiny scripts or CRUD utilities where layering adds little value.

---

## Related Patterns
- Hexagonal Architecture (Ports & Adapters)
- Onion Architecture
- Layered Architecture

---

## References
- Robert C. Martin (Uncle Bob), Clean Architecture (book, 2017)
- Blog: The Clean Architecture (2012)
