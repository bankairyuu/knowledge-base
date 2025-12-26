### Summary
A single deployable unit containing UI, business logic, and data access.
### When to Use
- Early-stage products and prototypes
- Small teams
- Simple domains
- Low operational maturity
### Pros
- Simple to develop, test, and deploy
- Minimal infrastructure overhead
- Easy debugging and local development
### Cons
- Tight coupling across modules
- Scaling is coarse-grained
- Risk of becoming an unmaintainable “big ball of mud”
### Real-World Usage
- Early versions of systems at companies like Basecamp
- Internal tools, MVPs, proof-of-concepts

---

![[Pasted image 20251226220127.png]]
![[Pasted image 20251226220136.png]]

### Common anti-patterns
- **Big Ball of Mud**: no meaningful module boundaries; everything depends on everything.
- **“God” service classes**: one service owns large portions of business logic with poor cohesion.
- **Shared mutable state**: static singletons, global caches, hidden side effects.
- **Entity leakage to API**: exposing JPA entities directly in controllers; tight coupling to persistence model.
- **Feature branching inside code**: environment/customer-specific behavior with `if/else` scattered throughout.
### Spring Boot best practice
- **Modularize inside the monolith**:
    - Use package/module boundaries by domain (e.g., `billing`, `booking`, `blog`) rather than by technical layer only.
    - Enforce boundaries with tools (ArchUnit tests) and build modules if needed (multi-module Maven).
- **DTOs at the edges**:
    - Controller ↔ DTO ↔ Mapper ↔ Domain/Service ↔ Repository.
    - Use MapStruct for stable mapping; avoid returning entities.
- **Clear transaction boundaries**:
    - Transactions at the service/application layer, not in controllers.
- **Configuration discipline**:
    - Profiles, externalized config, secrets via environment variables; avoid “config spread” across code.

---
## Monolith: database practices and failure modes

### Typical topology
- Single Spring Boot app
- Single PostgreSQL database
- Single Liquibase migration set
### Best practices
**a) Schema design aligned to domain modules**  
Even if you keep one physical DB, structure it:
- Schemas per domain (`billing.*`, `booking.*`, `blog.*`) or at least table prefixes.
- Ownership rules: “Only the billing module writes billing tables.”
**b) Avoid entity-driven schema design**  
JPA entities are not a schema design tool. Let the schema reflect:
- invariants
- constraints
- relationships
- query requirements
Then map entities to that.

**c) Enforce invariants in the database**  
Use:
- `NOT NULL`
- `CHECK` constraints
- `UNIQUE` constraints
- `FOREIGN KEY` constraints
- exclusion constraints when needed (e.g., booking overlap patterns)
This prevents “invalid states” when bugs occur.
### Common anti-patterns (database-specific)
- One giant schema with inconsistent naming and no ownership boundaries
- No constraints (“we validate in code”) → corruption appears over time
- Overusing cascading deletes/updates without clear lifecycle rules
- Long-running transactions (especially with heavy reads + writes) → lock contention
- N+1 query issues hidden by OSIV (Open Session In View)
### Recommended Spring Boot posture
- Disable OSIV and handle fetch strategies consciously.
- Put `@Transactional` at the service/use-case layer.
- Use Liquibase for deterministic schema evolution.
- Use integration tests with Testcontainers for all persistence-critical flows.