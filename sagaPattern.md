# Saga Pattern

## What problem does it solve?

In distributed systems / microservices architecture we often need to perform **business transactions that span multiple services**.

Examples:
- Create order → reserve inventory → charge customer → create shipment → send confirmation
- Book hotel + flight + car rental (travel booking)
- Transfer money between accounts in different banks

**Classic ACID transaction is not possible** because:
- different services = different databases
- 2-phase commit (XA/2PC) across services is almost never practical (performance, availability, technology heterogeneity, coordinator failure issues)

**Saga pattern** = way to achieve consistency across multiple services **without distributed locking or 2PC**.

## Core idea

A saga is a **sequence of local transactions** where:

1. Each step is executed by one service
2. Each step updates only that service’s own data (local ACID transaction)
3. If all steps succeed → business transaction completes
4. If any step fails → execute **compensating transactions** (reverse/undo actions) for all previously completed steps (in reverse order)

→ Goal = maintain **eventual consistency** instead of immediate global consistency

## Two main styles

### 1. Choreography (event-driven, decentralized)

Services communicate only via events.  
No central coordinator.

'''
Order Service             Inventory Service          Payment Service           Shipping Service
     │                             │                        │                        │
Create Order (local tx)           │                        │                        │
     │─ OrderCreated ──────────────                       │                        │
     │                             │─ Reserve stock ──────│                        │
     │                             │   (local tx)           │                        │
     │                             │   StockReserved ──────│                        │
     │                             │                        │─ Charge card ─────────│
     │                             │                        │   (local tx)            │
     │                             │                        │   PaymentFailed ────────┘
     │                             │─ PaymentFailed ───────┘                        │
     │────────────────────────────┘                        │                        │
Release stock (compensate)                                 │                        │
     │                             │                        │                        │
     └─ OrderCancelled ──────────────────────────────────────────────────────────────┘
     Cancel order (compensate)
'''mermaid
sequenceDiagram
    participant Order as Order Service
    participant Inventory as Inventory Service
    participant Payment as Payment Service
    participant Shipping as Shipping Service
    participant Broker as Event Broker<br>(Kafka / RabbitMQ / ...)

    Note over Order,Shipping: Happy path + failure + compensation

    Order->>Order: Create order<br>(local transaction)
    Order->>Broker: OrderCreated

    Broker->>Inventory: OrderCreated
    Inventory->>Inventory: Reserve items<br>(local transaction)
    Inventory->>Broker: InventoryReserved

    Broker->>Payment: InventoryReserved
    Payment->>Payment: Charge customer<br>(local transaction) → **fails**
    Payment->>Broker: PaymentFailed

    Broker->>Inventory: PaymentFailed
    Inventory->>Inventory: Release items (compensate)
    Inventory->>Broker: InventoryReleased   (optional event)

    Broker->>Order: PaymentFailed
    Order->>Order: Cancel order (compensate)
    Order->>Broker: OrderCancelled          (optional)

    alt Optional – Shipping never started
        Note right of Shipping: Shipping service never receives event
    end
'''
    

**Advantages**
- loosely coupled
- no single point of failure / bottleneck
- natural fit with event-driven architecture + outbox pattern

**Disadvantages**
- difficult to understand the full flow
- hard to monitor / trace long-running sagas
- compensation logic is spread across many services

### 2. Orchestration (central coordinator)

One dedicated component (orchestrator / saga executor) tells each service what to do.

     ┌──────────────────────┐
              │   Saga Orchestrator  │
              └──────────┬───────────┘
                         │
               CreateOrderCommand
                         ▼
              ┌──────────────────────┐
              │    Order Service     │
              └──────────┬───────────┘
                         │ OrderCreated
                         ▼
               ReserveInventoryCommand
                         ▼
              ┌──────────────────────┐
              │  Inventory Service   │
              └──────────┬───────────┘
                         │ InventoryReserved
                         ▼
                ChargePaymentCommand
                         ▼
              ┌──────────────────────┐
              │   Payment Service    │ ───┐ PaymentFailed
              └──────────┬───────────┘    │
                         │                │
               ReleaseInventoryCommand   │
                         ▼                │
              ┌──────────────────────┐    │
              │  Inventory Service   │───┘
              └──────────┬───────────┘
                         │
               CancelOrderCommand
                         ▼
              ┌──────────────────────┐
              │    Order Service     │
              └──────────────────────┘



**Advantages**
- easier to understand & visualize the entire process
- compensation logic lives in one place
- easier to add timeouts, retries, state persistence, observability
- better for complex flows with many steps / conditions

**Disadvantages**
- introduces a central component (needs to be highly available)
- tighter coupling to the orchestrator

## Important properties & requirements

| Aspect                     | Requirement / Characteristic                               |
|----------------------------|-------------------------------------------------------------------|
| Local transactions         | Must be ACID within one service                                   |
| Idempotency                | All operations (forward + compensate) should be idempotent       |
| Compensating actions       | Must be possible to reverse (or tolerate) each completed step     |
| Communication              | Usually events (choreography) or commands + replies (orchestration) |
| Consistency model          | Eventual consistency – no global isolation during the saga        |
| Visibility / monitoring    | Very important – especially in choreography                       |
| State persistence          | Orchestrator usually persists saga state durably                  |

## Common companions / patterns used together with Saga

- Transactional Outbox – reliable event publishing
- Idempotent Consumer – safe handling of duplicate events/commands
- Dead-letter queues / retry queues – for transient failures
- Correlation ID / Saga ID – to trace one saga across services

## When to use which style (rough guideline 2025–2026)

- **Simple to medium complexity** (3–7 steps), strong event-driven culture → **Choreography**
- **Many steps**, many conditions, business rules, timeouts, human tasks, need good observability → **Orchestration**
- Already using Temporal / Cadence / Step Functions / Conductor → **Orchestration** almost always wins

Popular orchestration tools (2025–2026):
- Temporal
- Cadence
- Netflix Conductor
- Camunda
- AWS Step Functions
- Apache Airflow (more batch-oriented but sometimes used)
- custom lightweight orchestrator

## Summary – one-liner definitions

- **Saga** = distributed transaction pattern using local transactions + compensating transactions
- **Choreography** = sagas coordinated via events (no boss)
- **Orchestration** = sagas coordinated by a central brain (orchestrator)

Happy building reliable distributed systems!














