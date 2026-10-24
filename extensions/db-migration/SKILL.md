---
name: db-migration
description: Use when creating, modifying, or reviewing a database schema or data migration in EF Core, Prisma, or Flyway, including rollback pairs and destructive-change checks.
---

# Database Migration

Author paired forward/rollback migrations and verify them before the user commits.

## When to use / When not to use

Use this skill when:

- The user asks to add a column, table, index, or constraint, or any other schema change.
- The user asks to backfill or transform stored data.
- A pull request touches migration files and needs review.

Do not use this skill when:

- The change is purely in application code with no schema or data change.
- The project uses a migration tool other than EF Core, Prisma, or Flyway. Follow that tool's documentation instead of guessing.

## Inputs

- The migration stack, detected from the repo:
  - EF Core: a `.csproj` referencing `Microsoft.EntityFrameworkCore`, a `DbContext` subclass, a `Migrations/` folder.
  - Prisma: `prisma/schema.prisma`.
  - Flyway: `flyway.conf` or `flyway.toml`, or a `sql/` or `migrations/` folder with `V*__*.sql` files.
- The model change the user wants.
- The two most recent existing migrations, to continue the naming convention and the version sequence.
- Whether the target databases already contain production data; this decides how strict the safety checklist is.

## Procedure

1. Detect the stack (see Inputs) and read the two most recent migrations to learn local conventions.
2. Name the migration per [naming-examples.md](naming-examples.md):
   - EF Core: `dotnet ef migrations add AddRefundReasonToOrders` (PascalCase, verb first).
   - Prisma: `npx prisma migrate dev --name add_refund_reason_to_orders` (snake_case).
   - Flyway: `V202610231430__add_refund_reason_to_orders.sql` (`V` + timestamp + `__` + snake_case); the rollback goes in a paired undo file `U202610231430__add_refund_reason_to_orders.sql` or a documented rollback script.
3. Write the forward migration and the rollback migration together. Every forward step needs a matching undo step in reverse order; concrete pairs per stack are in [naming-examples.md](naming-examples.md).
4. Run every operation through the checklist in [destructive-change-checklist.md](destructive-change-checklist.md) and classify it:
   - Safe: additive only (new nullable column, new table, index created concurrently).
   - Needs expand/contract: renames, type changes, adding `NOT NULL`. Ship as two or three migrations (expand, backfill, contract) instead of one destructive step.
   - Destructive: drops, narrowing types, changing primary keys. Require explicit user confirmation and a data-backup note in the PR description.
5. Apply data-migration discipline:
   - Backfills live in their own migration, separate from the schema change.
   - Batch large updates (1,000-10,000 rows per transaction) and make statements idempotent so a rerun is safe.
   - Migrations must not import application code or business logic; use plain SQL or the migration API only.
6. Verify locally before handing back:
   - EF Core: `dotnet ef database update`, then `dotnet ef database update <previous-migration>` to prove rollback, then update again.
   - Prisma: `npx prisma migrate dev` on a scratch database; inspect the generated SQL with `npx prisma migrate diff` when the diff looks wrong.
   - Flyway: `flyway migrate`, then `flyway undo` (Teams edition) or apply the paired rollback script on a scratch database.
7. Report what was created, the verification result, and any destructive operations the user must confirm.

## Output format

- New migration files in the stack's standard location. Never edit a migration that was already applied; create a new one.
- A summary listing: forward file, rollback file or section, checklist result (safe / expand-contract / destructive), and the exact verification commands run with their outcome.

## Guardrails

- Never edit a migration that has already been applied to a shared database; create a new migration that corrects it.
- Never drop data without explicit user confirmation; present the checklist result first and wait for approval.
- Keep migrations deterministic: no current timestamps, random values, or environment-dependent branches in the migration body unless the user asked.
- Workspace rules and the repository's existing naming and folder conventions take precedence over the defaults above.
- Run migrations only against a local or scratch database, never against a production connection string from a dev machine.
