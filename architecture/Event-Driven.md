### Summary
Components communicate via events rather than direct calls.
### When to Use
- Asynchronous workflows
- High scalability requirements
- Loose coupling between components
### Pros
- Highly decoupled systems
- Scales well under load
- Natural fit for reactive systems
### Cons
- More difficult to reason about flow
- Eventual consistency
- Debugging and tracing complexity
### Real-World Usage
- Payment systems
- Order processing platforms
- Messaging-based systems using Kafka or similar technologies

---

![[Pasted image 20251226221055.png]]


![[Pasted image 20251226221104.png]]
### Common anti-patterns
- **Event soup**: inconsistent naming, no schema governance, unclear semantics.
- **Using events as RPC**:
    - Emitting an event and expecting immediate “response” semantics; causes timing/coupling issues.
- **No idempotency**:
    - Consumers cannot safely reprocess; duplicates cause data corruption.
- **No replay strategy**:
    - Events emitted without retention/versioning; impossible to rebuild projections or debug history.
- **Ignoring ordering and partitioning realities**:
    - Assuming global ordering; failing to partition by business key (e.g., orderId).
### Spring Boot best practices
- **Explicit event contracts**:
    - Version your events; define schemas (JSON Schema/Avro/Protobuf depending on stack).
- **Idempotent consumers**:
    - Store processed event IDs (or use upsert semantics) to tolerate duplicates.
- **Transactional Outbox pattern**:
    - For “write DB + publish event” reliability, persist events in the same transaction, then publish asynchronously.
- **Separate internal domain events vs integration events**:
    - Internal events for decoupling within a service; integration events for other services/bounded contexts.
- **Operational tooling**:
    - Dead-letter queues/topics, retry policies, and monitoring for consumer lag.
---
## Event-driven systems: database patterns for reliability
### The reliability problem
The classic failure:
- transaction commits to DB
- message publish fails  
    Result: DB says “done” but the event never leaves.
### Transactional Outbox Pattern
Solve it by:
1. In the same DB transaction:
    - update business tables
    - insert an `outbox_event` row (payload + metadata)
2. A background publisher reads outbox rows and publishes them.
3. Mark outbox rows as published.

Benefits:
- no dual-write race condition
- events are durable
- supports retries and replay
### Inbox / idempotency
Consumers must handle:
- duplicates
- retries
- out-of-order deliveries

Approach:
- store a processed-event marker (`inbox_event` table keyed by eventId)
- apply changes only if not previously processed
### Anti-patterns
- Publishing events directly inside the transaction without outbox
- Consumers not idempotent
- No dead-letter strategy; poison messages block progress
- No event versioning; changes break consumers