## Liquibase and “safe schema evolution” (expand/contract)
This is one of the most practical areas where architecture and database meet.
### Expand/contract in practice
Goal: deploy schema and code changes safely with minimal downtime.
**Phase 1: Expand**
- Add new columns/tables nullable
- Add new indexes concurrently where possible
- Add triggers or dual writes if needed
- Keep old fields working

**Phase 2: Deploy application**
- Code writes to both old and new fields (or prefers new but tolerates old)
- Code reads using new field, fallback to old if null

**Phase 3: Migrate data**
- Backfill old → new (batch job)
- Validate counts and consistency

**Phase 4: Contract**
- Remove old columns
- Make new columns `NOT NULL`
- Add strict constraints once data is clean
### Common Liquibase pitfalls
- Large table migrations executed in one migration step without batching
- Adding non-nullable columns without defaults on large tables (locks/rewrites)
- Index creation without considering lock impact
- Coupling migrations tightly to a single code version (breaks rolling deployments)