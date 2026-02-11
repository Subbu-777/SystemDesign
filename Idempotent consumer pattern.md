# Idempotent Consumer Pattern with Conditional Writes

The pattern used in your payment/queue system — where a **single persistent record** (for an order, transaction, or payment intent) is:

- Created **exactly once** via a conditional write
- Then only **conditionally updated** forward through states (e.g. "AUTHORIZED" → "CAPTURED" only if current status matches expectation)
- Kept "ongoing" in DynamoDB (or similar) until it reaches a final state (SETTLED / FAILED / REFUNDED)

… is most commonly known as:

## **Idempotent Consumer** pattern  
(also frequently called **Idempotent Processing** or **Idempotency via Conditional Writes** in DynamoDB/AWS contexts)

### Sequence Diagram (Mermaid)

```mermaid
sequenceDiagram
    participant Client
    participant API as API Server
    participant DB as DynamoDB
    participant Queue as Message Queue
    participant Worker as Async Worker

    Client->>API: POST /payments (idempotencyKey or orderId)
    API->>DB: Conditional PutItem<br/>(attribute_not_exists(pk))
    alt First attempt (condition true)
        DB-->>API: Success - Item created
        API->>Queue: Enqueue processing job
        API-->>Client: 202 Accepted / 200 OK + status
    else Retry / duplicate
        DB-->>API: Condition check failed
        API->>DB: GetItem (read current state)
        DB-->>API: Existing item + status
        API-->>Client: 200 OK with current status (or 409 Conflict etc.)
    end

    Note over Queue,Worker: Async processing

    loop Worker consumes message
        Worker->>DB: Conditional UpdateItem<br/>(currentStatus = :expected AND version = :v)
        alt Valid transition
            DB-->>Worker: Success - state advanced
            alt Final state reached?
                Worker->>DB: Update to SETTLED/FAILED
                Worker-->>Queue: (optional) success ack
            else Intermediate
                Worker-->>Queue: Re-enqueue if needed / ack
            end
        else Invalid / already done
            DB-->>Worker: Condition check failed
            Worker-->>Queue: Ack (skip safely)
        end
    end

    Client->>API: GET /payments/{orderId} (polling)
    API->>DB: GetItem
    DB-->>API: Current status
    API-->>Client: Response with status
# Idempotent Consumer Pattern with Conditional Writes

The pattern used in your payment/queue system — where a **single persistent record** (for an order, transaction, or payment intent) is:

- Created **exactly once** via a conditional write
- Then only **conditionally updated** forward through states (e.g. "AUTHORIZED" → "CAPTURED" only if current status matches expectation)
- Kept "ongoing" in DynamoDB (or similar) until it reaches a final state (SETTLED / FAILED / REFUNDED)

… is most commonly known as:

## **Idempotent Consumer** pattern  
(also frequently called **Idempotent Processing** or **Idempotency via Conditional Writes** in DynamoDB/AWS contexts)

### Core Mechanism

1. **Client → Server (synchronous entry point)**  
   - Client sends request with idempotency key (UUID) or business key (order-id / txn-id).  
   - Server performs **conditional create** in DynamoDB:  
     ```text
     PutItem / TransactWriteItems
     ConditionExpression: "attribute_not_exists(#pk)"
     ```
   - **If condition passes** → new record created (first attempt only) → enqueue async work → return 202/200 immediately.  
   - **If condition fails** (already exists) → read existing state → return current status or stored response.  
     → **No new record**, no re-processing.

2. **Async queue / worker side**  
   - Workers consume messages from queue (SQS, Kafka, etc.).  
   - Before advancing state, perform **conditional update** on the **same single record**:  
     ```text
     UpdateItem
     ConditionExpression: "currentStatus = :expected"   // e.g. :expected = "AUTHORIZED"
     ```
   - If condition fails → skip safely (already processed or in different state).  
   - Result: forward-only state transitions, safe against retries/redeliveries.

### Why This Pattern Works at Scale

- Achieves **exactly-once effect** despite at-least-once delivery semantics  
- Prevents double charges, duplicate orders, or lost updates  
- Single item per logical entity → no record explosion  
- DynamoDB conditional writes are strongly consistent and very low latency  
- Used heavily by Grab, Gojek, Stripe integrations, Amazon e-commerce peaks, etc.

### Common Alternative / Related Names

- **Idempotency Key pattern** (Stripe terminology)  
  → Client provides key → server stores response/state keyed by it  
- **Deduplication with Conditional Writes** (AWS / DynamoDB blogs)  
- **Optimistic Concurrency Control** (via status/version in condition)  
- **Exactly-once processing** (the semantic outcome)  
- **Transactional Outbox** variant (if using append-only event log instead of mutating one row)

### In Production Naming

Most engineering teams at scale don't invent fancy names — they simply say:

> "We use conditional puts/updates on the order/payment item for idempotency"  
> or  
> "Idempotent consumer pattern with DynamoDB conditional writes"

### Recommendation for Documentation

When explaining this internally or to new team members, the clearest and most widely recognized name is:

**Idempotent Consumer pattern**  
(with DynamoDB conditional writes for enforcement)

This name is standard in microservices & event-driven architecture literature (Chris Richardson, AWS guides, etc.) and immediately conveys the intent: safe, retry-tolerant processing without side-effect duplication.

Let me know if you'd like code examples (Python + boto3), sequence diagrams, or comparison tables added to this markdown!
