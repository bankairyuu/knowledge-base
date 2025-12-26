## PostgreSQL operational concerns (often ignored until production)
### Indexing strategy
- Every foreign key column typically needs an index (for joins and deletes).
- Create composite indexes that match your common query predicates.
- Use `EXPLAIN (ANALYZE, BUFFERS)` early in performance troubleshooting.
### Locking and contention
- Long transactions are the silent killer (locks + vacuum issues).
- Bulk updates need batching.
- Be careful with `SELECT ... FOR UPDATE` patterns.
### Vacuum and bloat
- Hot tables with frequent updates/deletes can bloat.
- Ensure autovacuum is healthy; monitor dead tuples.
### Backups and restore testing
- A backup you never restored is not a plan.
- Periodically run restore drills (even if just to a local environment).