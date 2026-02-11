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

service and downstream consumers

Do not use it (or combine with other approaches) when you need:Strong global ordering across aggregates
Sub-100 ms end-to-end event delivery (use direct publishing + retries + circuit breakers instead)

Popular references (2025–2026):microservices.io – Transactional Outbox
Confluent – The Transactional Outbox Pattern
AWS – Implementing the Transactional Outbox Pattern
Debezium documentation – Outbox Event Router

