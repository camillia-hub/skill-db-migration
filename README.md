# DB Migration

A Camillia skill that authors and verifies database migrations across EF Core, Prisma, and Flyway: naming, forward/rollback pairs, a destructive-change safety checklist, and data-migration discipline.

## What the agent gains

- Stack detection and correct scaffolding commands for EF Core, Prisma, and Flyway
- Forward and rollback migrations written together, never as an afterthought
- A destructive-change checklist that forces expand/contract for renames and `NOT NULL` additions
- Local apply-then-rollback verification before anything is committed

## Install

Install from the Camillia Community Hub. After installation the skill lives in the workspace under `.workspace/extensions/db-migration/` and the agent applies it whenever you create or review a migration.

## Contents

- `extensions/db-migration/SKILL.md` - the procedure the agent follows
- `extensions/db-migration/naming-examples.md` - naming conventions and forward/rollback pairs per stack
- `extensions/db-migration/destructive-change-checklist.md` - operation classification table

## Validate

```
node scripts/validate.mjs
```

## License

MIT - see [LICENSE](LICENSE). Copyright 2026 Camillia contributors.
