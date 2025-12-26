### Summary
Organizes code into horizontal layers (Controller → Service → Repository).
### When to Use
- CRUD-heavy business applications
- Enterprise systems with stable requirements
- Teams familiar with traditional enterprise patterns
### Pros
- Clear separation of concerns
- Easy onboarding for developers
- Strong tooling and framework support (e.g., Spring Boot)
### Cons
- Business logic can leak across layers
- Encourages anemic domain models
- Harder to adapt to complex domains
### Real-World Usage
- Banking and insurance systems
- Typical Spring Boot enterprise backends

---

![[Pasted image 20251226220343.png]]
![[Pasted image 20251226220348.png]]
### Common anti-patterns
- **Anemic Domain Model**: entities are just data holders; all logic sits in services.
- **Service-to-service spaghetti**: “Service A calls B calls C calls D” with unclear ownership.
- **Repository-as-service**: putting business logic into repositories, custom queries everywhere, or calling repositories directly from controllers.
- **Transaction boundaries in the wrong layer**: `@Transactional` on controllers or repositories in a way that hides real unit-of-work intent.
- **Leaky abstractions**: controllers assembling persistence objects; services exposing persistence details.
### Spring Boot best practices
- **Define responsibilities explicitly**:
    - Controllers: request validation + orchestration only (no business logic).
    - Services: business rules + transaction boundaries.
    - Repositories: persistence only (no domain decisions).
- **Prefer “use-case oriented services”**:
    - Instead of generic `UserService`, prefer `RegisterUserUseCase`, `ApproveOrderUseCase` (even inside layered).
- **Keep JPA entities persistence-focused**:
    - Use entity graphs/fetch joins intentionally; avoid accidental N+1 and lazy-loading in controllers.
- **Validation at the edge**:
    - Bean Validation in request DTOs; treat validation as an API concern, not a persistence concern.
---
## Layered architecture: database implications
Layered architecture doesn’t change the physical DB, but it strongly influences how you access it.
### Repository layer discipline
The most common database degradation in layered architectures is the repository layer becoming a “query dump.”
Guidelines:
- Keep repositories close to aggregate needs, not “global query libraries.”
- Favor explicit query methods for read patterns, but avoid scattering domain decisions into SQL.
### Read performance strategy
Layered systems often evolve toward:
- complex filtering
- admin/report screens
- analytics-style queries
Do not fight PostgreSQL
- Use proper indexes and query plans early.
- If reporting becomes heavy, consider read-optimized projections (see CQRS).
### Anti-patterns
- “Repository returns entity graphs for UI” (over-fetching)
- Controllers building dynamic queries ad-hoc
- Implicit data access patterns (lazy loading surprises)