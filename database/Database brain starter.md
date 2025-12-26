## Core database concerns that exist in every architecture
### Data ownership and responsibility
Regardless of architecture, database design fails most often because **ownership is unclear**:
- Who owns the schema?
- Who is allowed to write which tables?
- What is the “source of truth” for each business concept?
Guardrail: define **bounded ownership** (module/service/team). Even in a monolith, treat domains as schema “owners” conceptually.
### The schema is part of your API
Your database schema is a long-lived contract:
- It outlives many code deployments.
- It must be evolvable without downtime.
- It must support backfill, rollbacks (or forward-only strategies), and partial deployments.
Guardrail: adopt **expand/contract migrations** (detailed later) and avoid breaking changes without a safe rollout plan.
### Transactions: what you can and cannot guarantee
[[PostgreSQL]] gives strong transactional semantics per database, but:
- In monoliths: one DB transaction can cover multiple aggregates/tables.
- In microservices/event-driven: you cannot rely on distributed transactions; you trade for **eventual consistency**.
Guardrail: explicitly document consistency expectations per use case.
### Auditability and compliance
Even if you are not regulated, audit concerns appear quickly:
- Who changed what, when, and why?
- How do you reconstruct state after failures?
- How do you prove correctness?
Guardrail: at minimum use `created_at`, `updated_at`, `created_by`, `updated_by`, and consider an **audit log** strategy for critical changes.
## Architecture wise
- [[Monolith#Monolith database practices and failure modes|Monolith]]
- [[Hexagonal - Clean - Ports and Adapters#Hexagonal/Clean database as an adapter, not a core dependency|Hexagonal-Clean-Ports and Adapters]]
- [[Microservice#Microservices “Database per service” and what it really implies|Microservices]]
- [[Event-Driven#Event-driven systems database patterns for reliability|Event-driven]]
- [[CQRS#CQRS database separation between reads and writes|CQRS]]
---
## Practical Spring Boot recommendations by architecture (database-only)
### Monolith / Layered
- Single [[PostgreSQL]] + [[Liquibase]]
- Strong DB constraints
- Disable OSIV; fetch intentionally
- Integration tests with Testcontainers
- Clear module ownership even inside one schema
### Hexagonal / Clean
- Treat DB as an adapter
- Domain is framework-free
- Ports reflect use cases, not SQL
- Mapping explicit (MapStruct where appropriate, but persistence mapping may be hand-written)
### Microservices
- Database per service (or schema per service as a transitional step)
- No shared tables
- Data duplication via events/projections
- Observability for DB performance and migrations
- Strict contract versioning between services (DB is not your integration boundary)
### Event-driven / CQRS
- Transactional outbox + idempotent consumers   
- Read projections with rebuild capability
- Event schema/versioning governance
- Operational tooling: DLQs, retries, lag monitoring