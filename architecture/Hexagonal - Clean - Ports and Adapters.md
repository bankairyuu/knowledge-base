### Summary
Domain-centric architecture isolating business logic from infrastructure.
### When to Use
- Complex business domains
- Long-lived systems
- High testability requirements
- Multiple input/output adapters (REST, messaging, batch)
### Pros
- Excellent testability
- Infrastructure-agnostic domain logic
- Clear dependency direction (inward)
### Cons
- Higher upfront complexity
- Requires architectural discipline
- Overkill for simple CRUD systems
### Real-World Usage
- Core financial platforms
- Systems where business rules are the primary asset
- Frequently used in DDD-oriented teams

---

![[Pasted image 20251226220649.png]]
![[Pasted image 20251226220705.png]]
### Common anti-pattern
- **Fake hexagon**: packages named `domain/application/infrastructure`, but dependencies still point outward-to-inward incorrectly    
- **Domain depends on Spring**: `@Service`, `@Component`, `@Transactional` in domain code; the domain becomes framework-coupled.
- **Over-abstraction**: too many ports for trivial CRUD, leading to “interface explosion.”
- **Anemic “domain” anyway**: you adopt the structure but keep business rules in adapters/services, defeating the purpose.
- **Adapters bypass the application layer**: controllers calling repositories directly (or calling infra services directly).
### Spring Boot best practices
- **Keep the domain plain**:
    - Domain model and domain services should be simple Java/Kotlin objects without Spring annotations.
- **Application layer as orchestrator**:
    - Application services (use cases) are Spring-managed beans; they depend on ports (interfaces).
- **Infrastructure implements ports**:
    - Repositories, external clients, event publishers are adapters implementing port interfaces.
- **Dependency rule enforcement**:
    - Use ArchUnit to ensure `domain` does not import `org.springframework.*` and `infrastructure` does not get referenced by `domain`.
- **Testing pyramid advantage**:
    - Fast unit tests for domain + use cases with mocks for ports (Mockito), and fewer integration tests for adapters.
---
## Hexagonal/Clean: database as an adapter, not a core dependency
### Key idea
In hexagonal architecture:
- The **domain/application** shouldn’t know PostgreSQL exists.
- PostgreSQL is an implementation detail behind a port (interface).
### Database modeling guidance
You still design a schema in PostgreSQL, but you aim to keep:
- persistence concerns in the adapter layer
- mapping between domain concepts and tables explicit

There are two common approaches:
**Approach A: Domain model ≈ Persistence model**
- Domain objects are close to JPA entities (but still framework-free).
- Adapter maps domain ↔ entity.
Pros: less duplication. 
Cons: domain may get shaped by persistence constraints.

**Approach B: Separate persistence model**
- Entities are persistence artifacts only.
- Domain model is richer and independent.
- Mapping layer is more explicit.

Pros: best domain purity and long-term flexibility.  
Cons: more mapping and development overhead.
### Database anti-patterns
- “Domain depends on JPA annotations” (your domain is no longer portable/testable)
- Ports that leak persistence semantics (e.g., “findByStatusAndCreatedAtBetween” as a port)
- Over-abstracted repositories where every query becomes a “port,” increasing complexity
### Best practice boundary
Ports should reflect use cases, not storage:
- Good: `LoadCustomerProfile(customerId)`
- Risky: `FindCustomerEntitiesByEmailDomain(domain)`