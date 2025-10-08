# ⬡ Hexagonal Architecture (Ports & Adapters)

Hexagonal Architecture, also called Ports and Adapters (by Alistair Cockburn), isolates the application core from external technologies by communicating through well-defined ports implemented by adapters.

---

## Core Idea
- The domain and application logic live at the center.
- "Ports" are interfaces describing what the application needs or offers.
- "Adapters" implement those ports to connect to the outside (HTTP, DB, Message Bus, CLI, Tests).

```
        ┌───────────────────────────────────┐
        │           Adapters                │
        │  (Web, DB, CLI, Messaging, UI)   │
        └───────▲───────────────────▲───────┘
                │                   │
             Inbound              Outbound
              Port                  Port
                │                   │
            ┌───┴───────────────────┴───┐
            │        Application Core    │
            │  (Domain + Use Cases)      │
            └────────────────────────────┘
```

---

## Benefits
- Technology-agnostic core: swap frameworks with minimal impact.
- High testability: use in-memory adapters in tests.
- Clear boundaries: reduces coupling and accidental complexity.

## Trade-offs
- More interfaces and classes to manage.
- Requires consistent naming and dependency direction.

---

## Typical Elements
- Inbound Adapters: Controllers, GraphQL resolvers, message consumers, CLI.
- Inbound Ports: Use case interfaces invoked by inbound adapters.
- Outbound Ports: Interfaces the core uses (repositories, gateways).
- Outbound Adapters: Implementations using DBs, HTTP clients, queues, files.

---

## Example Slice (pseudo)
```
interface PlaceOrder { execute(cmd): Result }
interface OrdersRepository { save(order); findById(id) }

class PlaceOrderService implements PlaceOrder {
  constructor(private repo: OrdersRepository) {}
  execute(cmd) {
    const order = Order.create(cmd);
    this.repo.save(order);
    return { id: order.id };
  }
}

// Outbound adapter
class SqlOrdersRepository implements OrdersRepository { /* ... */ }
// Inbound adapter
class HttpController { constructor(private placeOrder: PlaceOrder) {} /* ... */ }
```

---

## When to Use
- Need to keep domain free of infrastructure.
- Multiple delivery mechanisms (HTTP, messaging, CLI) over the same core.
- Want fast, isolated tests.

## When to Avoid
- Very small apps where the abstraction cost outweighs benefits.

---

## Related Patterns
- Clean Architecture
- Onion Architecture
- Layered Architecture

---

## References
- Alistair Cockburn, "Hexagonal Architecture"
- Ports and Adapters articles and talks
