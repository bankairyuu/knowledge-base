## The most common / important architectures are:
1. [[Monolith]]
2. [[Layered]]
3. [[Hexagonal - Clean - Ports and Adapters]]
4. [[Microservice]]
5. [[Event-Driven]]
6. [[CQRS]]
## Skeleton
| Situation                        | Recommended                                |
| -------------------------------- | ------------------------------------------ |
| MVP / Prototype                  | [[Monolith]]                               |
| Standard Enterprise CRUD         | [[Layered]]                                |
| Complex business logic           | [[Hexagonal - Clean - Ports and Adapters]] |
| Large teams, large scale         | [[Microservice]]                           |
| High throughput, async workflows | [[Event-Driven]]                           |
| Heavy reads, complex writes      | [[CQRS]]                                   |
## Spring Boot Implementation Mapping (Cross-Cutting Practices)
These apply regardless of architecture, but they become non-negotiable as complexity increases:
## Boundary hygiene
- Controllers only handle HTTP concerns (validation, auth context, request/response mapping).
- MapStruct for DTO ↔ internal model mapping.
- Never return JPA entities from controllers.
## Transaction and consistency
- `@Transactional` belongs in the application/service layer.
- Avoid lazy-loading outside of service boundaries; do not rely on Open Session In View.
## Configuration and environments
- Profiles + externalized config (`application-*.properties/yaml`), secrets via environment variables.
- Separate config per environment; do not branch logic per environment inside code.
## Testing strategy
- Domain/use-case tests: fast unit tests (Mockito).
- Adapter tests: integration tests using Testcontainers for PostgreSQL and messaging where applicable.
- Contract tests become valuable once microservices/event-driven are in place.
## Observability
- Structured logs, correlation IDs, metrics, traces (Micrometer/OpenTelemetry).
- Health checks and readiness probes for containerized deployments.