# 🎯 Event-Driven Architecture

Event-driven architecture (EDA) is a design pattern where system components communicate through events rather than direct method calls. Events represent significant occurrences or state changes that other components can react to asynchronously.

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
An **event** is a record of something that happened in the system. Events are immutable facts that describe state changes or significant occurrences.

### Event Producer
An **event producer** (or publisher) generates and publishes events when something noteworthy occurs in the system.

### Event Consumer
An **event consumer** (or subscriber) reacts to events by performing actions or updating their own state.

### Event Broker
An **event broker** (or message broker) acts as an intermediary that receives events from producers and delivers them to interested consumers.

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
// Apache Kafka Event Streaming
const kafka = require('kafka-node');

const consumer = new kafka.Consumer(
  new kafka.Client(),
  [{ topic: 'user-events', partition: 0 }],
  { autoCommit: true }
);

consumer.on('message', async (message) => {
  const event = JSON.parse(message.value);

  switch(event.type) {
    case 'UserCreated':
      await sendWelcomeEmail(event.userId);
      break;
    case 'UserLoggedIn':
      await updateLastLogin(event.userId);
      break;
  }
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

## Benefits & Challenges

### Benefits
- **Loose Coupling**: Components don't need to know about each other
- **Scalability**: Easy to add new consumers without affecting producers
- **Flexibility**: New features can be added by subscribing to existing events
- **Audit Trail**: Complete history of what happened in the system
- **Real-time Processing**: Immediate reactions to events as they occur

### Challenges
- **Eventual Consistency**: System state is eventually consistent
- **Debugging Complexity**: Harder to trace execution flow
- **Event Schema Evolution**: Managing changes to event structures
- **Duplicate Processing**: Handling duplicate events gracefully
- **Ordering Guarantees**: Ensuring events are processed in correct order

---

## Best Practices

### Event Design
1. **Make Events Immutable**: Events represent facts that happened
2. **Use Descriptive Names**: Event names should clearly indicate what occurred
3. **Include Context**: Events should contain all necessary information
4. **Version Events**: Plan for event schema evolution
5. **Keep Events Small**: Avoid large payloads in events

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

---

*Return to [Architecture Patterns Overview](./README.md) or [Core Architectural Skills](../README.md)*