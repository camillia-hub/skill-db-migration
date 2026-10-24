# Destructive-change safety checklist

Classify every operation in the migration before writing it.

| Operation | Class | Required approach |
|-----------|-------|-------------------|
| Add nullable column | Safe | Ship directly |
| Add column with a default | Mostly safe | Check table size; on huge tables add nullable + backfill instead of one locking default |
| Add table | Safe | Ship directly |
| Add index | Safe with care | Create concurrently / online (`CONCURRENTLY` in Postgres, `ONLINE = ON` in SQL Server) to avoid table locks |
| Add `NOT NULL` to an existing column | Expand/contract | Add nullable, backfill, then add the constraint in a later migration |
| Rename column or table | Expand/contract | Expand, dual-write, backfill, contract; a raw rename breaks the old app version during a rolling deploy |
| Widen a column type (varchar 50 -> 200) | Usually safe | Check lock behavior on large tables |
| Narrow a column type | Destructive | Confirm existing data fits, get explicit user approval, note backup |
| Drop column or table | Destructive | Only in a contract migration after the code no longer reads it; explicit user confirmation required |
| Change primary key / clustered index | Destructive | Table rebuild; requires a maintenance-window note |
| Data backfill | Data migration | Separate migration, batched, idempotent |

Rules:

- Present this classification to the user before running anything destructive.
- Every destructive migration needs a rollback plan that is honest about data loss: a rollback restores the schema, not the dropped rows. Say so explicitly.
- Backfill batches: 1,000-10,000 rows per transaction, idempotent statements (`WHERE new_column IS NULL`) so a rerun is safe.
