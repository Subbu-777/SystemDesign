# Transactional Outbox Pattern

**Also known as**: Outbox Pattern, Reliable Outbox, Transactional Outbox Pattern

**Main goal**  
Guarantee that business changes and outgoing events / messages are **published reliably and atomically** — without using distributed (2PC/XA) transactions.

## The classic dual-write problem it solves

Most business operations need to do **two things** that must succeed or fail together:

1. Update the application's own database (e.g. create/update an order, booking, payment, user balance…)
2. Publish an event/message so other services can react (e.g. `OrderCreated`, `PaymentCaptured`, `BookingConfirmed`)

Naive implementation (not safe):

```text
BEGIN TRANSACTION
  UPDATE orders SET status = 'PAID', updated_at = NOW() WHERE id = ?
  kafkaTemplate.send("order-events", orderId, OrderPaidEvent(...))
COMMIT
---

## Failure modes that break Failure:DB commits → message publish fails / broker is down → lost event
Message sent → DB rollback / crash → phantom event (downstream thinks something happened that didn't)

Distributed transactions (2PC) are usually not an option because:many brokers (Kafka before exactly-once producers, most queues) don't support them
performance penalty is very high
operational complexity explodes

Core idea of the Outbox patternInstead of publishing directly inside the business transaction:Write the business change and the event(s) you want to send inside the same local database transaction
Store the events in a special table called the outbox
A separate background process reads the outbox table → publishes the events → marks them sent (or deletes them)


Because both writes happen in one ACID transaction, you get atomicity for free.
CREATE TABLE outbox (
    id                BIGSERIAL PRIMARY KEY,
    aggregate_type    VARCHAR(50)     NOT NULL,     -- e.g. 'Order', 'Payment', 'User'
    aggregate_id      VARCHAR(100)    NOT NULL,     -- business entity id
    event_type        VARCHAR(100)    NOT NULL,     -- 'OrderCreated', 'PaymentCaptured'…
    payload           JSONB           NOT NULL,     -- serialized event
    metadata          JSONB,                        -- tracing, correlation id, etc.
    created_at        TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    status            VARCHAR(20)     NOT NULL DEFAULT 'PENDING',
    published_at      TIMESTAMPTZ,
    version           INT             NOT NULL DEFAULT 1,   -- optimistic locking (optional)
    UNIQUE (aggregate_type, aggregate_id, event_type, version) -- helps prevent accidental duplicates
);


CREATE INDEX idx_outbox_pending ON outbox (status, created_at) WHERE status = 'PENDING';

sequenceDiagram
    participant Client
    participant Service
    participant DB
    participant Relay as Outbox Relay / Poller / CDC
    participant Broker as Kafka / RabbitMQ / SQS

    Client->>+Service: Create order + pay
    Service->>+DB: BEGIN TRANSACTION
    Service->>DB: INSERT/UPDATE business data (orders, payments, …)
    Service->>DB: INSERT INTO outbox (event_type='OrderPaid', payload=…, status='PENDING')
    DB-->>Service: OK
    Service->>DB: COMMIT
    DB-->>-Service: committed
    Service-->>-Client: 202 Accepted

    note over DB,Relay: independent process – can be seconds later

    loop every few seconds or via CDC
        Relay->>DB: SELECT * FROM outbox WHERE status = 'PENDING' ORDER BY created_at LIMIT 100
        DB-->>Relay: rows
        Relay->>Broker: publish each event
        Broker-->>Relay: ack
        Relay->>DB: UPDATE outbox SET status='SENT', published_at=NOW() WHERE id IN (...)
        DB-->>Relay: OK
    end
