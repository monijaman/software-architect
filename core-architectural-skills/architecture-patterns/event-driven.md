# 🎯 Event-Driven Architecture

**Event-Driven Architecture (EDA)** is a software design pattern where components (or services) communicate by producing and consuming "events."

Instead of services calling each other directly (like in REST APIs), they respond to events that describe **what happened** — not **what to do**.

## 🧭 What Makes EDA Different?

### Traditional | **🔁 Ordering Guarantees** | Ensuring events processed in correct sequence | Use event sequencing or design for commutative operations |

---

## 🧩 Common Tools & Technologies

### Event Brokers & Message Buses
| Tool | Best For | Key Features |
|------|----------|--------------|
| **Apache Kafka** | High-throughput streaming | Durability, partitioning, exactly-once processing |
| **RabbitMQ** | Traditional messaging | Flexible routing, multiple protocols |
| **AWS SNS/SQS** | Cloud-native AWS | Managed service, pay-per-use |
| **Google Pub/Sub** | GCP integration | Global distribution, exactly-once delivery |
| **NATS** | Lightweight, high-performance | Simple, fast, Kubernetes-native |

### Streaming & Processing Platforms
| Tool | Purpose | Use Case |
|------|---------|----------|
| **Apache Flink** | Real-time stream processing | Complex event processing, windowing |
| **Kafka Streams** | Lightweight stream processing | Simple transformations, aggregations |
| **Apache Spark Streaming** | Batch + stream processing | Unified batch/stream analytics |

### Event Storage & Schema Management
| Tool | Purpose | Features |
|------|---------|----------|
| **EventStoreDB** | Event sourcing database | Optimistic concurrency, projections |
| **DynamoDB Streams** | AWS event streaming | Serverless, integrated with Lambda |
| **Avro/JSON Schema** | Schema validation | Versioning, compatibility checks |
| **Protobuf** | Efficient serialization | Language-agnostic, compact |

### 🧠 EDA Patterns Overview

| Pattern | Description | When to Use |
|---------|-------------|-------------|
| **Event Notification** | "Something happened" - minimal info, consumer fetches details | Simple notifications, loose coupling |
| **Event-Carried State Transfer** | Event includes all data needed to act | High performance, offline processing |
| **Event Sourcing** | Store all state changes as events (replayable) | Audit requirements, temporal queries |
| **CQRS** | Separate command (write) and query (read) models | Complex domains, different read/write patterns |

---quest-Driven) Approach
When a user places an order in an e-commerce system:
- Order Service calls Inventory Service to reduce stock
- Then calls Payment Service to charge the customer
- Then calls Email Service to send confirmation
- **All services are tightly coupled** through direct API calls

### ⚡ Event-Driven Approach
In EDA, the Order Service simply publishes an event:
```json
{
  "event": "OrderPlaced",
  "orderId": 123,
  "userId": 10,
  "timestamp": "2025-10-08T10:30:00Z"
}
```

Then:
- **Inventory Service** listens for `OrderPlaced` → reduces stock
- **Payment Service** listens for `OrderPlaced` → charges customer
- **Email Service** listens for `OrderPlaced` → sends confirmation

**Each service reacts independently — no direct coupling!**

## 🔁 Event Flow Architecture

```
+---------------+        +-------------+        +---------------+
|  Order Service| -----> | Event Broker| -----> | Payment Service|
| (Publisher)   |        |  (e.g. Kafka)|       | (Subscriber)  |
+---------------+        +-------------+        +---------------+
                                    \
                                     \-------> | Inventory Service|
                                               | (Subscriber)    |
                                               +---------------+
```

## 🧩 Core Components

| Component | Role | Example |
|-----------|------|---------|
| **Event** | A record that something happened | `"UserRegistered"`, `"OrderPlaced"` |
| **Producer/Publisher** | Service that creates and publishes events | Order Service publishing `OrderPlaced` |
| **Consumer/Subscriber** | Service that listens and reacts to events | Inventory Service reducing stock |
| **Event Broker** | Middleware routing events (Kafka, RabbitMQ, AWS SNS/SQS) | Apache Kafka, RabbitMQ |

## Table of Contents
- [Core Concepts](#core-concepts)
- [Event Types](#event-types)
- [Event Patterns](#event-patterns)
- [Implementation Examples](#implementation-examples)
- [Benefits & Challenges](#benefits--challenges)
- [Best Practices](#best-practices)

---

## Core Concepts

### Event
An **event** is an immutable record that something significant happened in the system. Events describe **what occurred**, not **what to do**.

**Key Characteristics:**
- **Immutable**: Events represent facts that cannot be changed
- **Descriptive**: Clear indication of what happened (e.g., `OrderPlaced`, `UserRegistered`)
- **Complete**: Contains all context needed for consumers to react
- **Timestamped**: Includes when the event occurred

### Event Producer (Publisher)
The **producer** creates and publishes events when something noteworthy occurs. Producers don't know or care who consumes their events.

### Event Consumer (Subscriber)
The **consumer** listens for specific events and reacts accordingly. Multiple consumers can react to the same event independently.

### Event Broker (Message Bus)
The **broker** acts as middleware that:
- Receives events from producers
- Routes events to interested consumers
- Provides durability, scalability, and reliability
- Examples: Apache Kafka, RabbitMQ, AWS SNS/SQS, Google Pub/Sub

---

## Event Types

### Domain Events
**Domain Events** represent business-significant occurrences within your domain. They capture meaningful state changes that other parts of the system might be interested in.

```javascript
// Domain Event Example
class OrderPlacedEvent {
  constructor(orderId, customerId, items, totalAmount) {
    this.eventType = 'OrderPlaced';
    this.orderId = orderId;
    this.customerId = customerId;
    this.items = items;
    this.totalAmount = totalAmount;
    this.timestamp = new Date();
  }
}

// Publishing a domain event
class OrderService {
  async placeOrder(orderData) {
    const order = await this.repository.save(orderData);

    // Publish domain event
    const event = new OrderPlacedEvent(
      order.id,
      order.customerId,
      order.items,
      order.totalAmount
    );

    await this.eventBus.publish('order.placed', event);
    return order;
  }
}
```

### Integration Events
**Integration Events** are used for communication between different bounded contexts or services. They represent cross-service concerns.

```javascript
// Integration Event Example
class UserRegisteredIntegrationEvent {
  constructor(userId, email, name) {
    this.eventType = 'UserRegistered';
    this.userId = userId;
    this.email = email;
    this.name = name;
    this.timestamp = new Date();
  }
}
```

### System Events
**System Events** represent infrastructure-level occurrences like service startup, database connections, or health check failures.

---

## Event Patterns

### Event Sourcing
**Event Sourcing** stores the state of an entity as a sequence of events rather than storing the current state directly. The current state is reconstructed by replaying events.

```javascript
class EventSourcedAggregate {
  constructor() {
    this.events = [];
    this.version = 0;
  }

  apply(event) {
    this.events.push(event);
    this.version++;
    this.handleEvent(event);
  }

  handleEvent(event) {
    // Update aggregate state based on event
    switch(event.eventType) {
      case 'OrderCreated':
        this.status = 'created';
        this.items = event.items;
        break;
      case 'OrderPaid':
        this.status = 'paid';
        break;
    }
  }

  // Reconstruct state from events
  static fromEvents(events) {
    const aggregate = new EventSourcedAggregate();
    events.forEach(event => aggregate.apply(event));
    return aggregate;
  }
}
```

### CQRS (Command Query Responsibility Segregation)
**CQRS** separates read and write operations into different models. Commands change state, queries read state. Often combined with event sourcing.

```javascript
// Command Model (Write Side)
class OrderCommandHandler {
  async handle(command) {
    switch(command.type) {
      case 'CreateOrder':
        const order = new Order(command.orderId);
        order.create(command.items);
        await this.repository.save(order);
        break;
      case 'PayOrder':
        const existingOrder = await this.repository.get(command.orderId);
        existingOrder.pay(command.paymentInfo);
        await this.repository.save(existingOrder);
        break;
    }
  }
}

// Query Model (Read Side)
class OrderQueryHandler {
  async getOrderSummary(orderId) {
    return await this.readRepository.getOrderSummary(orderId);
  }

  async getOrderHistory(orderId) {
    return await this.readRepository.getOrderHistory(orderId);
  }
}
```

### Event Streaming
**Event Streaming** processes continuous streams of events in real-time. Events are processed as they arrive, enabling immediate reactions.

```javascript
// Apache Kafka Event Streaming with error handling
const kafka = require('kafka-node');

const consumer = new kafka.Consumer(
  new kafka.Client(),
  [{ topic: 'user-events', partition: 0 }],
  { autoCommit: false } // Manual commit for better control
);

consumer.on('message', async (message) => {
  try {
    const event = JSON.parse(message.value);

    // Validate event structure
    if (!event.type || !event.userId) {
      console.error('Invalid event structure:', event);
      return;
    }

    switch(event.type) {
      case 'UserCreated':
        await sendWelcomeEmail(event.userId);
        break;
      case 'UserLoggedIn':
        await updateLastLogin(event.userId);
        break;
      default:
        console.warn('Unknown event type:', event.type);
    }

    // Manual commit after successful processing
    consumer.commit((err, data) => {
      if (err) console.error('Commit error:', err);
    });

  } catch (error) {
    console.error('Error processing event:', error);
    // Send to dead letter queue or retry logic here
  }
});

// Handle consumer errors
consumer.on('error', (err) => {
  console.error('Consumer error:', err);
});
```

---

## Implementation Examples

### Simple Event Bus
```javascript
class SimpleEventBus {
  constructor() {
    this.handlers = new Map();
  }

  subscribe(eventType, handler) {
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, []);
    }
    this.handlers.get(eventType).push(handler);
  }

  async publish(eventType, event) {
    const handlers = this.handlers.get(eventType) || [];
    await Promise.all(
      handlers.map(handler => handler(event))
    );
  }
}

// Usage
const eventBus = new SimpleEventBus();

eventBus.subscribe('order.placed', async (event) => {
  console.log(`Order ${event.orderId} was placed`);
  await sendConfirmationEmail(event.customerId);
});

eventBus.subscribe('order.placed', async (event) => {
  await updateInventory(event.items);
});
```

### Event Store Implementation
```javascript
class EventStore {
  constructor() {
    this.events = new Map(); // aggregateId -> events[]
  }

  async saveEvents(aggregateId, events, expectedVersion) {
    const existingEvents = this.events.get(aggregateId) || [];

    if (existingEvents.length !== expectedVersion) {
      throw new ConcurrencyError('Version conflict');
    }

    this.events.set(aggregateId, [...existingEvents, ...events]);
  }

  async getEvents(aggregateId) {
    return this.events.get(aggregateId) || [];
  }
}
```

---

## 💡 When to Use Event-Driven Architecture

### ✅ **Perfect For EDA:**
- **High scalability requirements** - Components scale independently
- **Domain fits "things happening"** - Orders, messages, status changes, IoT events
- **Real-time updates needed** - Notifications, analytics, live dashboards
- **Microservices architecture** - Loose coupling between services
- **Eventual consistency is acceptable** - Business can tolerate temporary inconsistencies

### ❌ **Avoid EDA When:**
- **Strict ACID transactions required** across multiple services
- **Simplicity and traceability** are more important than decoupling
- **Small team/small system** - Coupling overhead outweighs benefits
- **Real-time responses needed** - Synchronous communication is faster
- **Complex business logic** requires coordinated state changes

### 🎯 **EDA Sweet Spot:**
```
Microservices + Real-time Processing + Scalable Systems
```

---

## 🧠 Benefits of Event-Driven Architecture

| Benefit | Description |
|---------|-------------|
| **🔗 Loose Coupling** | Services don't call each other directly; only exchange events through a broker |
| **📈 Scalability** | Components scale independently based on their event processing load |
| **🔧 Resilience** | If one service fails, others continue; events can be replayed or processed later |
| **🚀 Extensibility** | Add new subscribers without changing existing services or producers |
| **⚡ Real-time Processing** | Perfect for notifications, analytics, dashboards, and stream processing |
| **📝 Audit Trail** | Complete history of all business events for compliance and debugging |
| **🔄 Flexibility** | Easy to add new features by subscribing to existing events |

## ⚠️ Challenges & Trade-offs

| Challenge | Description | Mitigation Strategy |
|-----------|-------------|-------------------|
| **🔍 Debugging Complexity** | Logic spread across async events makes tracing harder | Use distributed tracing (Jaeger, Zipkin) and correlation IDs |
| **🔄 Eventual Consistency** | Strong consistency harder to maintain | Design for eventual consistency; use sagas for complex transactions |
| **📊 Monitoring Complexity** | Need to monitor event flows across services | Implement comprehensive observability with metrics and logs |
| **📋 Schema Evolution** | Events evolve; old consumers must still work | Version events and handle backward compatibility |
| **🔁 Duplicate Processing** | Events might be delivered multiple times | Make consumers idempotent |
| **📏 Ordering Guarantees** | Ensuring events processed in correct sequence | Use event sequencing or design for commutative operations |

---

## Best Practices

### Event Design
1. **Make Events Immutable**: Events represent facts that happened
2. **Use Descriptive Names**: Event names should clearly indicate what occurred
3. **Include Context**: Events should contain all necessary information
4. **Version Events**: Plan for event schema evolution
5. **Keep Events Small**: Avoid large payloads in events

### Event Versioning & Schema Evolution
```javascript
// Versioned Event Structure
class OrderPlacedEventV1 {
  constructor(orderId, customerId, items, totalAmount) {
    this.eventType = 'OrderPlaced';
    this.eventVersion = '1.0';
    this.orderId = orderId;
    this.customerId = customerId;
    this.items = items;
    this.totalAmount = totalAmount;
    this.timestamp = new Date();
  }
}

// Backward Compatible Event (V2 adds shippingAddress)
class OrderPlacedEventV2 {
  constructor(orderId, customerId, items, totalAmount, shippingAddress = null) {
    this.eventType = 'OrderPlaced';
    this.eventVersion = '2.0';
    this.orderId = orderId;
    this.customerId = customerId;
    this.items = items;
    this.totalAmount = totalAmount;
    this.shippingAddress = shippingAddress; // New optional field
    this.timestamp = new Date();
  }

  // Factory method for backward compatibility
  static fromV1(v1Event) {
    return new OrderPlacedEventV2(
      v1Event.orderId,
      v1Event.customerId,
      v1Event.items,
      v1Event.totalAmount,
      null // Default shipping address
    );
  }
}

// Event Consumer with Version Handling
class OrderEventConsumer {
  async handleEvent(rawEvent) {
    let event = rawEvent;

    // Handle version compatibility
    if (rawEvent.eventVersion === '1.0') {
      event = OrderPlacedEventV2.fromV1(rawEvent);
    }

    // Process with latest version
    await this.processOrderPlaced(event);
  }
}
```

### Implementation Guidelines
1. **Idempotent Consumers**: Handle duplicate events gracefully
2. **Dead Letter Queues**: Handle events that cannot be processed
3. **Monitoring**: Track event throughput and processing times
4. **Testing**: Test event-driven flows end-to-end
5. **Documentation**: Document event schemas and contracts

### Common Pitfalls to Avoid
- **Synchronous Event Processing**: Don't block producers waiting for consumers
- **Event Chains**: Avoid long chains of dependent events
- **Event Pollution**: Don't publish events for every minor state change
- **Tight Coupling**: Consumers shouldn't depend on producer implementation details
- **Missing Event Versioning**: Always version events for schema evolution

### Testing Event-Driven Systems
```javascript
// Testing event publishing
describe('OrderService', () => {
  let eventBusSpy;

  beforeEach(() => {
    eventBusSpy = { publish: jest.fn() };
  });

  it('should publish OrderPlaced event when order is created', async () => {
    const service = new OrderService(mockRepository, eventBusSpy);

    await service.placeOrder(orderData);

    expect(eventBusSpy.publish).toHaveBeenCalledWith(
      'order.placed',
      expect.objectContaining({
        eventType: 'OrderPlaced',
        orderId: expect.any(String)
      })
    );
  });
});

// Testing event consumption
describe('OrderEventConsumer', () => {
  it('should send confirmation email when order is placed', async () => {
    const emailServiceSpy = { sendConfirmation: jest.fn() };
    const consumer = new OrderEventConsumer(emailServiceSpy);

    const event = {
      eventType: 'OrderPlaced',
      orderId: '123',
      customerId: '456'
    };

    await consumer.handleEvent(event);

    expect(emailServiceSpy.sendConfirmation).toHaveBeenCalledWith('456');
  });
});
```

  });
});

---

## 🏁 Summary: Event-Driven Architecture Essentials

| Concept | Key Point |
|---------|-----------|
| **🎯 Core Idea** | Systems react to events instead of making direct calls |
| **🎯 Goal** | Loose coupling, scalability, and asynchronous communication |
| **⚖️ Trade-off** | Simpler integration, but harder debugging & consistency |
| **🎯 Best Used In** | Microservices, real-time apps, IoT, analytics pipelines |
| **🛠️ Essential Tools** | Kafka, RabbitMQ, AWS SNS/SQS, EventStoreDB |
| **📋 Key Patterns** | Event Sourcing, CQRS, Event-Carried State Transfer |
| **⚡ Sweet Spot** | High-scale systems where eventual consistency is acceptable |

**Remember**: EDA excels when your domain naturally fits "things happening" rather than "commands to execute." Start with event storming to identify your domain events, then design your architecture around them.

---

*Return to [Architecture Patterns Overview](./README.md) or [Core Architectural Skills](../README.md)*