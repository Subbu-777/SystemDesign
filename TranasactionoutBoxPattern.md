# Transactional Outbox Pattern

The **Transactional Outbox Pattern** (also called **Outbox Pattern** or **Transactional Outbox**) is a widely used architectural pattern in distributed systems and microservices to solve the **dual-write problem** while ensuring reliable, atomic event/message publishing without distributed transactions (2PC/XA).

It is one of the foundational patterns in event-driven architectures, popularized by Chris Richardson (microservices.io) and adopted by companies using Kafka, RabbitMQ, AWS, Azure, etc.

## Problem It Solves: The Dual-Write Problem

In a microservice, a business operation often needs to:

1. Update persistent state (e.g., insert/update an Order in the database)
2. Publish an event/message to a broker (e.g., "OrderCreated" to Kafka, SQS, RabbitMQ) so other services can react

If you do these as two separate steps:

```text
BEGIN TRANSACTION
  UPDATE orders SET status = 'CREATED' WHERE id = ?
  PUBLISH "OrderCreated" to Kafka
COMMIT
```

A failure (crash, network issue, broker down) after the DB commit but before publish → inconsistency
→ Downstream services never get notified, leading to lost events, stale views, or manual reconciliation.Distributed transactions (2PC) are complex, hurt performance, and are not supported by many modern brokers (e.g., Kafka before transactional producers).Core IdeaInstead of publishing directly to the broker inside the transaction:Write the business change and the outbound message(s) atomically inside the same local database transaction.
Store outbound messages in a dedicated Outbox table (or collection in NoSQL).
A separate, independent process (poller, relay, CDC connector) reads the Outbox and publishes the messages to the broker → marks them as sent or deletes them.

If the transaction commits → both business data + outbox message are saved → eventual publish is guaranteed.

sequenceDiagram
    participant Client
    participant Service as Business Service
    participant DB as Database
    participant OutboxPoller as Outbox Relay / Poller / CDC
    participant Broker as Message Broker (Kafka/RabbitMQ)

    Client->>Service: POST /orders (create order)
    activate Service

    Service->>DB: BEGIN TRANSACTION
    Service->>DB: INSERT/UPDATE business entity (Order)
    Service->>DB: INSERT into outbox_table (event payload, type, aggregate_id, created_at, status='PENDING')
    DB-->>Service: Success
    Service->>DB: COMMIT
    deactivate Service

    Service-->>Client: 202 Accepted / 200 OK

    Note over DB,OutboxPoller: Later / continuously...

    loop Polling or CDC
        OutboxPoller->>DB: SELECT * FROM outbox WHERE status = 'PENDING' ORDER BY created_at LIMIT ...
        DB-->>OutboxPoller: Pending messages
        alt Messages found
            OutboxPoller->>Broker: Publish event (e.g. OrderCreated JSON)
            Broker-->>OutboxPoller: Ack
            OutboxPoller->>DB: UPDATE outbox SET status = 'SENT' / DELETE row
        end
    end


# Transactional Outbox Pattern – Key Components

| Component              | Purpose / Responsibility                                                                 | Typical Implementation / Examples                          | Important Notes / Requirements                          |
|------------------------|------------------------------------------------------------------------------------------|------------------------------------------------------------|----------------------------------------------------------|
| **Business Logic**     | Performs the core domain operation (create/update entity)                                | Service layer, controller, use-case/handler                | Must run inside a transaction                            |
| **Database Transaction** | Atomically commits both business change + outbox message insertion                      | ACID transaction (SQL: BEGIN/COMMIT, DynamoDB Transact*)   | Single local transaction – no 2PC                        |
| **Outbox Table / Collection** | Persistent queue for events that need to be published                                   | Relational: `outbox` table<br>NoSQL: dedicated item/stream | Fields: id, event_type, aggregate_id, payload, status, created_at |
| **Event Payload**      | Serialized representation of the domain event                                            | JSON / JSONB / Avro / Protobuf                             | Should include event ID, version, timestamp, metadata   |
| **Relay / Publisher / Poller** | Reads unsent events from outbox and sends them to the message broker                    | - Scheduled job / cron<br>- Debezium Outbox Router<br>- DynamoDB Streams → Lambda<br>- Kafka Connect | Runs independently, can retry, should be highly available |
| **Message Broker**     | Durable, distributed destination for events                                             | Kafka, RabbitMQ, AWS SQS/SNS, Azure Service Bus, Redis Streams | Must support at-least-once delivery                     |
| **Consumer(s)**        | Downstream services that react to the events                                             | Other microservices, read models, notifications, etc.      | **Must be idempotent** (critical requirement)            |
| **Status / Tracking**  | Tracks whether an outbox entry has been published (PENDING → SENT / FAILED)             | Column: `status`, `published_at`, `attempts`               | Optional: dead-letter logic for permanent failures       |
| **Cleanup Mechanism**  | Prevents outbox table from growing indefinitely                                         | TTL, scheduled DELETE, archive job                         | Usually delete after SENT + X days                       |

### Quick Summary – Core Guarantee

Everything inside the **business transaction** is atomic → either both the state change **and** the outbox message are saved, or neither is.

The relay takes care of the **eventual** publishing → giving you reliable at-least-once delivery without coupling the critical path to the broker.

Pros & ConsAdvantagesAtomicity without 2PC / distributed transactions
Works with any DB that supports ACID transactions
Reliable "at-least-once" delivery to broker
Retries are easy (poller can retry failed publishes)
No direct dependency on broker availability during business transaction
CDC-based variants (Debezium) → zero polling overhead

DisadvantagesAdditional table → schema pollution (mitigated by separate schema or JSONB)
Polling adds latency (seconds) — CDC (Debezium, DynamoDB Streams) removes this
Outbox can grow large if relay is down → needs cleanup/TTL
Eventual consistency between service DB and downstream consumers
Requires idempotent consumers (but most event-driven systems need this anyway)

Common Implementation VariantsPolling Outbox — Simple scheduled job queries status = 'PENDING'
Change Data Capture (CDC) — Debezium / Maxwell tails binlog → publishes automatically (popular with Postgres + Kafka)
Transaction Log Tailing — Similar to CDC, used by SeatGeek, many large-scale systems
NoSQL variants — Single-item transactions (DynamoDB TransactWriteItems) or multi-document (Cosmos DB)


If it rolls back → nothing is published → perfect consistency.

