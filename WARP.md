# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repository Overview

This is an Obsidian-based knowledge repository documenting software architecture patterns and database practices, with a focus on Spring Boot implementations. The content is organized as an interconnected knowledge graph using Obsidian's wiki-link syntax (`[[Page Name]]`).

## Repository Structure

### Core Directories
- `architecture/` - Architecture pattern documentation (Monolith, Microservice, Hexagonal/Clean, Layered, Event-Driven, CQRS)
- `database/` - Database-specific guidance (PostgreSQL, Liquibase, schema evolution)
- `architecture/attachments/` - Diagrams and images referenced in architecture documents
- `.obsidian/` - Obsidian configuration (do not modify unless specifically requested)

### Entry Points
- `Landing page.md` - Main navigation hub linking to key documents
- `architecture/Architecture decider.md` - Decision matrix for choosing architectures
- `database/Database brain starter.md` - Database concerns across architectures

## Working with This Repository

### Markdown Format
- Uses Obsidian wiki-link syntax: `[[Document Name]]` for internal links
- Uses `[[Document Name#Section]]` for section links
- Images are embedded with `![[image-name.png]]`
- Tables use standard markdown table syntax

### Document Navigation
When referencing documents:
- Use the exact document names as they appear in the filesystem
- Respect the wiki-link format for cross-references
- Maintain the hierarchical structure when adding new documents

### Content Structure
Each architecture pattern document follows this template:
- **Summary** - One-sentence description
- **When to Use** - Use cases and scenarios
- **Pros/Cons** - Trade-offs
- **Real-World Usage** - Examples
- **Common anti-patterns** - What to avoid
- **Spring Boot best practices** - Implementation guidance
- **Database implications** - Persistence concerns specific to this pattern

## Architecture Decision Framework

The repository provides a structured approach to architecture selection:

| Situation | Recommended Architecture |
|-----------|-------------------------|
| MVP / Prototype | Monolith |
| Standard Enterprise CRUD | Layered |
| Complex business logic | Hexagonal/Clean/Ports and Adapters |
| Large teams, large scale | Microservice |
| High throughput, async workflows | Event-Driven |
| Heavy reads, complex writes | CQRS |

## Key Cross-Cutting Concerns

### Database Practices
- **Ownership**: Always define clear data ownership boundaries
- **Migrations**: Use Liquibase with expand/contract pattern for zero-downtime deployments
- **Constraints**: Enforce invariants at the database level (NOT NULL, CHECK, UNIQUE, FK)
- **Transactions**: Place `@Transactional` at service/application layer, not controllers
- **OSIV**: Disable Open Session In View; fetch intentionally

### Spring Boot Standards
- **DTO boundaries**: Controllers ↔ DTO ↔ Mapper ↔ Domain/Service ↔ Repository
- **MapStruct**: Use for DTO-to-domain mapping
- **Testing**: Testcontainers for integration tests with PostgreSQL
- **Configuration**: Externalized config with profiles; secrets via environment variables
- **Observability**: Structured logs, correlation IDs, Micrometer/OpenTelemetry

## Common Anti-Patterns to Avoid

### Architectural
- Distributed monolith (services deployed together)
- Wrong service boundaries (technical layers vs business capabilities)
- Shared databases in microservices
- CQRS for simple CRUD
- Event soup without schema governance

### Database
- Entity-driven schema design
- Missing database constraints ("we validate in code")
- Long-running transactions causing lock contention
- N+1 queries hidden by OSIV
- No idempotency in event consumers

### Code Organization
- Anemic domain models
- "God" service classes
- Entity leakage to controllers
- Service-to-service spaghetti

## Editing Guidelines

### Adding New Content
1. Place documents in appropriate directories (`architecture/` or `database/`)
2. Follow the established template structure
3. Use wiki-links for cross-references
4. Update `Landing page.md` or relevant hub pages with links to new content
5. Store images in `architecture/attachments/` or `database/attachments/`

### Modifying Existing Content
- Preserve wiki-link references when renaming documents
- Maintain the template structure (Summary, When to Use, Pros/Cons, etc.)
- Keep anti-patterns and best practices sections distinct
- Ensure Spring Boot specifics are clearly marked

### Cross-References
When referencing other documents:
- Use specific sections where relevant: `[[Document#Section Name]]`
- Maintain bidirectional links where concepts are related
- Reference both general patterns and specific implementations

## Documentation Philosophy

This repository emphasizes:
- **Practical over theoretical**: Real-world patterns and anti-patterns
- **Spring Boot context**: Concrete implementation guidance
- **Database implications**: Every architecture's impact on data persistence
- **Decision support**: Clear guidance on when to use each pattern
- **Failure modes**: Documenting what goes wrong and why
