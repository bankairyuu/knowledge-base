### Summary
Separates read and write models explicitly.
### When to Use
- Read-heavy systems
- Complex business rules on writes
- Performance-sensitive querying
### Pros
- Optimized read and write paths
- Scales independently
- Clean separation of responsibilities
### Cons
- Increased complexity
- Eventual consistency
- More moving parts
### Real-World Usage
- Financial trading systems
- Audit-heavy domains
- Systems with complex reporting needs

---

![[Pasted image 20251226220857.png]]
![[Pasted image 20251226220904.png]]
### Common anti-patterns
- **CQRS everywhere**: splitting read/write for trivial CRUD, creating unnecessary complexity.
- **Inconsistent projections**:
    - Read models get stale with no repair/rebuild mechanism.
- **Leaking write model to reads**:
    - Still querying the transactional schema for “read side,” negating CQRS benefits.
- **Hard-coupled projections**:
    - Projection logic embedded in write transaction (synchronous projection updates) causing latency and coupling.
- **No strategy for eventual consistency**:
    - UI/clients assume immediate consistency and then behave incorrectly.
### Spring Boot best practices
- **Adopt selectively**:
    - Use CQRS only for aggregates with heavy read requirements, complex writes, or expensive query patterns.
- **Separate persistence for read models when justified**:
    - Potentially separate tables/schemas optimized for queries; keep rebuild scripts/jobs.
- **Async projection updates**:
    - Read models updated by consumers processing events; monitor lag.
- **API design accounts for eventual consistency**:
    - Return “accepted/processing” responses where appropriate; provide client patterns (polling, websocket updates, etc.).
---
## CQRS: database separation between reads and writes
### Core concept
Write model and read model are optimized independently:
- Write DB: normalized, constraints, transactional integrity
- Read DB: denormalized, indexed for query patterns, often different tables or even different stores
### Read model building
Read models are typically built from:
- events emitted from write side
- stream processing / consumer services

Key properties:
- read model can be rebuilt (replay events)
- eventual consistency is explicit
### When CQRS is justified
- heavy read load
- complex filtering/reporting
- expensive joins on transactional schema
- need for multiple read “views” (admin vs customer vs analytics)
### Anti-patterns
- CQRS for trivial CRUD (cost > benefit)
- Read models without rebuild capability
- Synchronous read model updates inside write transaction (negates decoupling)
- No client/UI strategy for eventual consistency