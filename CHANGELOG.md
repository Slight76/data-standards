# Changelog

All notable changes to this handbook. Versioning follows SemVer; rule IDs are never renamed or reused.

## 1.0.0 - 2026-10-07

First release of `data-standards`, split out of `Slight76/architecture-standards` (v0.3.0, commit `c1bda3d`) as decided in [ADR-0001](adr/0001-adopt-data-standards.md).

### Moved from architecture-standards@c1bda3d

Rule IDs, statements, and historic ADR references are unchanged. Wording was adjusted from "enterprise" to "team" and links were rewritten for the new repository layout; frontmatter was added.

| New location | Former location | Rules |
| --- | --- | --- |
| `docs/database-architecture.md` | `database/architecture.md` | DB-001..DB-005 |
| `docs/design-standard.md` | `database/design-standard.md` | DB-006..DB-008 |
| `docs/migration-recovery-standard.md` | `database/migration-recovery-standard.md` | MIG-001, MIG-002, DR-001 |
| `docs/persistence-standard.md` | `backend/persistence-standard.md` | DATA-001..DATA-003 |
| `docs/caching-standard.md` | `backend/caching-standard.md` | CACHE-001, CACHE-002 |

### Added

- `docs/naming-conventions.md` with rules NAME-001..NAME-004 (snake_case, plural tables, named constraints, module schemas, timestamp columns, enums, EF Core mapping).
- `docs/postgres-anti-patterns.md` with rules PGX-001..PGX-003 (FK indexes, keyset pagination, non-locking migrations) and a review checklist.
- `docs/backup-restore-runbook.md` (pg_dump/pg_restore and Fly Postgres procedures, restore drill cadence, evidence; verifies DB-005 and DR-001).
- `catalog/catalog.json` scoped to this handbook with `externalDecisions` for ADR-0005, ADR-0016, ADR-0018, ADR-0019.
- `adr/0001-adopt-data-standards.md`.
- `skills/data-standards/SKILL.md` with `references/read-by-task.md` and `references/catalog-digest.md`.
- `plugin.json`, `README.md`, `AGENTS.md`, `CLAUDE.md`, MIT `LICENSE`, lint configuration, and the shared `docs.yml` workflow.
