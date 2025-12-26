### Summary
System decomposed into independently deployable services.
### When to Use
- Large teams and organizations
- Independently evolving domains
- Need for independent scaling and deployment
### Pros
- Independent scaling and deployment
- Strong team autonomy
- Fault isolation
### Cons
- High operational complexity
- Distributed systems challenges (latency, consistency)
- Requires mature DevOps and monitoring
### Real-World Usage
- Netflix
- Amazon
- Large SaaS platforms with many teams

---

![[Pasted image 20251226220752.png]]
![[Pasted image 20251226220759.png]]
### Common anti-patterns

- **Distributed monolith**: services must be deployed together; tight runtime coupling via synchronous calls.
- **Shared database**: multiple services writing the same schema (nearly always a long-term failure mode).
- **Wrong service boundaries**:
    - Splitting by technical layers (auth-service, user-service, email-service) instead of by business capabilities.
- **Chatty services**:
    - Many synchronous calls per request; high latency and fragile dependencies.
- **“Microservices first” without ops maturity**:
    - No tracing, poor observability, manual deployments, no SLOs; diagnosing issues becomes expensive.
### Spring Boot best practices
- **Design boundaries around business capabilities**:
    - Each service owns its schema (and migrations) and has clear APIs/contracts.
- **Resilience patterns**:
    - Timeouts, retries with backoff, circuit breakers (Resilience4j), bulkheads where needed.
- **Observability by default**:
    - Micrometer + OpenTelemetry tracing; structured logging with correlation IDs.
- **Contract-first thinking**:
    - Versioned APIs, backward compatibility, consumer-driven contracts where appropriate.
- **Prefer async for cross-service workflows**:
    - Use events or queues to reduce synchronous coupling; keep sync calls for strictly necessary query-like interactions.
---
## Microservices: “Database per service” and what it really implies
### The foundational rule
**Each microservice owns its database (or schema) and is the only writer.**  
No shared tables, no cross-service foreign keys.

This is the core that preserves independence.
### Why shared databases fail
Shared DBs cause:
- deployment coupling (schema changes break other services)
- runtime coupling (unexpected locks/queries)
- implicit dependencies that aren’t documented as APIs
- governance problems (who can change what?)
### Data duplication is normal
If Service A needs data from Service B:
- It should not read B’s tables.
- It should have its own data (copied, projected) via:
    - events (preferred)
    - explicit API calls and caching (situational)
### Referential integrity moves upward
Since cross-service FKs are impossible, integrity becomes:
- local: enforced by DB constraints within a service boundary
- global: enforced via workflows, events, and compensations

Example:
- Orders service references `customerId` as a scalar.
- It cannot FK to Customers DB.
- It ensures customer existence via validation at command time and/or event-driven synchronization.
### Migration strategy becomes operational
With multiple services:
- each has its own Liquibase migration lifecycle
- each deploy must be compatible with running versions of other services
- you need versioning and backward compatibility in contracts
### Anti-patterns
- Shared database (most common)
- “Reporting service” directly querying all service DBs (better to build projections)
- Tight synchronous dependencies with transactional expectations (“call service B in the same transaction”)
- Attempting distributed ACID via hacks instead of embracing eventual consistency and sagas