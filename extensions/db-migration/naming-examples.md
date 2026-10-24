# Naming conventions and forward/rollback pairs

## EF Core

Command: `dotnet ef migrations add AddRefundReasonToOrders`

- PascalCase, starts with a verb: `AddX`, `RemoveY`, `RenameZToW`, `AlterXColumn`.
- The scaffold generates `Migrations/<timestamp>_Name.cs` plus a `.Designer.cs`; the `Down()` method is the rollback. Always review `Down()` - the scaffolded version drops everything `Up()` added, which is wrong once you hand-edit a data step in.

Pair:

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.AddColumn<string>(
        name: "RefundReason", table: "Orders", nullable: true);
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.DropColumn(name: "RefundReason", table: "Orders");
}
```

## Prisma

Command: `npx prisma migrate dev --name add_refund_reason_to_orders`

- snake_case, verb first; the folder becomes `<timestamp>_add_refund_reason_to_orders/migration.sql`.
- Prisma does not generate down-migrations. Keep the rollback next to the forward file as `rollback.sql`, or document the reverse `migrate diff` command in the PR.

Forward (`migration.sql`):

```sql
ALTER TABLE "Orders" ADD COLUMN "RefundReason" TEXT;
```

Rollback (`rollback.sql`):

```sql
ALTER TABLE "Orders" DROP COLUMN "RefundReason";
```

## Flyway

Files: `V202610231430__add_refund_reason_to_orders.sql` and `U202610231430__add_refund_reason_to_orders.sql` (undo, Teams edition) or a documented rollback script.

- Version: `V` + `yyyyMMddHHmm` timestamp so ordering is obvious; a double underscore separates version from description; description is snake_case.
- Never reuse a version number and never edit an applied file; the checksum mismatch blocks future runs.

Forward:

```sql
ALTER TABLE orders ADD COLUMN refund_reason VARCHAR(200) NULL;
```

Undo / rollback:

```sql
ALTER TABLE orders DROP COLUMN refund_reason;
```

## Expand/contract example: rename without downtime

Wrong: one migration that renames `orders.customer_name` to `orders.buyer_name` while the old app version still runs.

Right, three migrations:

1. Expand: add `buyer_name`, make the app dual-write, keep `customer_name`.
2. Backfill (separate data migration, batched): copy `customer_name` into `buyer_name` in chunks of 5,000 rows, `WHERE buyer_name IS NULL` so reruns are safe.
3. Contract (only after the old code is fully rolled out): drop `customer_name`.
