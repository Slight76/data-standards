---
title: "Database naming conventions"
status: proposed
version: 1.0.0
owner: "@Slight76"
---
# Database naming conventions

Baseline: 1.0.0. Applies when: a solution creates or changes PostgreSQL schemas, tables, columns, indexes, or constraints, including through EF Core migrations

Decision: [ADR-0001](../adr/0001-adopt-data-standards.md). Rules become binding when this baseline is adopted; examples explain the policy and do not establish business requirements.

This document fixes the names so that nobody has to decide them again. It refines the naming paragraph in [Logical and physical database design](design-standard.md) (DB-007) into patterns an agent can apply mechanically and a reviewer can check by eye. The reference engine is PostgreSQL; the reference ORM is EF Core.

## Case and characters

- Use lower `snake_case` for every identifier: schemas, tables, columns, indexes, constraints, sequences, functions, views.
- Use ASCII letters, digits, and underscores only. Start with a letter. Never rely on quoting to preserve case; an identifier that needs quotes is wrong.
- Stay within 63 bytes (the PostgreSQL limit). Abbreviate the *entity* part of a generated name before shortening the prefix or suffix, and record the abbreviation in the entity configuration.
- Avoid reserved words (`user`, `order`, `group`, `type`). Prefer `app_users`, `purchase_orders`, `user_groups`.
- No Hungarian prefixes (`tbl_`, `col_`), no type suffixes (`name_str`), no spaces, no camelCase.

## Tables: plural

Tables are **plural nouns**: `orders`, `order_lines`, `warehouses`, `stock_levels`. A table is a set of rows, and `SELECT * FROM orders WHERE ...` reads as prose. Choosing plural also matches the existing design standard, the EF Core default convention (`DbSet<Order> Orders` maps to `orders` under the snake-case naming plugin), and the names already in production. Consistency matters more than the grammar argument, so do not mix: a repository that adopted singular names before this baseline records a documented exception rather than converting half its tables.

Join tables are named after both sides in alphabetical order, plural: `products_tags`, `roles_users`. When the relationship carries its own data and identity, give it a real name (`enrollments`, not `courses_students`).

## Columns

| Kind | Pattern | Example |
| --- | --- | --- |
| Primary key | `id` | `orders.id` |
| Foreign key | `<singular_entity>_id` | `order_lines.order_id` |
| Self-reference / role-specific FK | `<role>_<entity>_id` | `shipments.origin_warehouse_id` |
| Boolean | `is_<adjective>` or `has_<noun>` | `is_active`, `has_attachments` |
| Instant (point in time) | `<event>_at`, type `timestamptz` | `created_at`, `shipped_at`, `deleted_at` |
| Calendar date | `<event>_on`, type `date` | `due_on`, `invoiced_on` |
| Count / quantity | `<noun>_count`, `<noun>_quantity` | `retry_count`, `reserved_quantity` |
| Money | `<noun>_amount` numeric + `<noun>_currency` char(3) | `total_amount`, `total_currency` |
| Concurrency token | `row_version` (`xmin` or bigint) | `stock_levels.row_version` |
| Tenant key | `tenant_id` | every tenant-scoped table |

Every table has `id`, `created_at timestamptz NOT NULL DEFAULT now()`, and `updated_at timestamptz NOT NULL`. `updated_at` is set by the application on every write (EF Core `SaveChanges` interceptor), not by a trigger, so that the value is testable and visible in the change-tracker. Do not add `created_by`/`updated_by` columns by reflex; add them when an audit requirement names them.

## Constraints and indexes

Name every constraint and index explicitly; never accept engine-generated names, because they differ between a fresh database and an upgraded one and make migrations unreadable.

| Object | Pattern | Example |
| --- | --- | --- |
| Primary key | `pk_<table>` | `pk_orders` |
| Foreign key | `fk_<table>_<referenced_table>` (add `_<role>` when two FKs target the same table) | `fk_order_lines_orders`, `fk_shipments_warehouses_origin` |
| Unique constraint | `uq_<table>_<col>[_<col>]` | `uq_stock_levels_tenant_id_sku` |
| Check constraint | `ck_<table>_<rule>` | `ck_stock_levels_quantity_non_negative` |
| Index | `ix_<table>_<col>[_<col>]` | `ix_order_lines_order_id` |
| Partial index | `ix_<table>_<col>_<condition>` | `ix_orders_tenant_id_open` |
| Sequence (when not identity) | `seq_<table>_<col>` | `seq_invoices_number` |

Column order in the name matches column order in the definition. Every foreign key column gets an `ix_` index unless a review records why the access pattern never needs it (see [Postgres anti-patterns](postgres-anti-patterns.md)).

## Schemas

Each owning module gets its own schema named after the module in snake_case: `ordering`, `inventory`, `identity`. Do not put application tables in `public`; reserve it for extensions. EF Core migration history lives in the owning schema (`ordering.__ef_migrations_history`) so that two modules sharing an instance never fight over one history table. Cross-schema foreign keys are a smell that the module boundary is wrong; see the design standard.

## Enums

Prefer a `text` column plus a `CHECK` constraint listing the allowed values, or a small lookup table when values carry data or change at runtime. Native PostgreSQL `ENUM` types are allowed only when the value set is genuinely closed (ISO codes, weekday); adding a value to a native enum is not transactional in older versions and renaming one requires a rewrite. Store the enum member name (`shipped`), never the C# integer ordinal; a reordered C# enum silently corrupts data otherwise. Configure this in EF Core with `HasConversion<string>()` and a `HasMaxLength` wide enough for the longest member.

## EF Core mapping notes

- Use `EFCore.NamingConventions` with `UseSnakeCaseNamingConvention()` so that C# PascalCase maps automatically; keep C# names as domain language and never bend them to SQL.
- Put every explicit override in an `IEntityTypeConfiguration<T>` under Infrastructure; no data annotations on domain types.
- Name constraints in configuration (`HasConstraintName`, `HasDatabaseName`) following the tables above so that generated migrations are stable and reviewable.
- Set `HasDefaultSchema("<module>")` and `MigrationsHistoryTable("__ef_migrations_history", "<module>")` on the context.
- Map `DateTimeOffset`/UTC `DateTime` to `timestamptz`, `DateOnly` to `date`, `decimal` with explicit `HasPrecision(p, s)`.
- Review the generated migration SQL (`dotnet ef migrations script`) for names that violate this document before opening the PR; the reviewer checks names, not the C# diff.

## Rules and required evidence

| ID | Requirement | Verification |
| --- | --- | --- |
| NAME-001 | Database identifiers MUST be lower snake_case ASCII without quoting; tables MUST be plural nouns and columns MUST follow the documented patterns. | Migration SQL review against this document |
| NAME-002 | Every table MUST have an `id` primary key, `created_at`, and `updated_at` as `timestamptz`; date-only values MUST use `date`. | Schema inspection and entity configuration review |
| NAME-003 | Every constraint and index MUST have an explicit name following the `pk_`/`fk_`/`uq_`/`ck_`/`ix_` patterns. | Generated migration diff shows no engine-generated names |
| NAME-004 | Each module MUST own a named schema and its own migration history table; application tables MUST NOT live in `public`. | Schema listing and `DbContext` configuration review |

## Exceptions

Use the [exception record](https://github.com/Slight76/standards-marketplace/blob/main/templates/exception.md) for a departure. Record affected rules, scope, compensating controls, approval evidence, expiry, and migration path. Agents must not silently replace defaults.
