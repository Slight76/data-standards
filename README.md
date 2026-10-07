# data-standards

Team data standards for developers and AI agents: database design, naming, PostgreSQL, migrations, persistence, caching, backup and restore. The reference stack is PostgreSQL with .NET and EF Core, hosted on Fly.io.

Part of the Slight76 standards handbooks indexed at [standards-marketplace](https://github.com/Slight76/standards-marketplace). Written for a small team and its AI agents.

## Documents

| Document | Covers | Rule prefixes |
| --- | --- | --- |
| [Database architecture](docs/database-architecture.md) | Store ownership, logical/physical design scope, migrations, least-privilege identities, RPO/RTO and restore evidence | DB |
| [Logical and physical database design](docs/design-standard.md) | Storage selection, naming and types, constraints, tenant and lifecycle design, derived stores, design evidence | DB |
| [Database naming conventions](docs/naming-conventions.md) | snake_case identifiers, plural tables, column and constraint/index name patterns, timestamps, enums, schemas, EF Core mapping | NAME |
| [PostgreSQL anti-patterns and preferred alternatives](docs/postgres-anti-patterns.md) | Review checklist: column types, missing FK indexes, soft delete, JSONB, N+1, SELECT *, IN lists, OFFSET pagination, long transactions, locking migrations | PGX |
| [Schema delivery and recovery](docs/migration-recovery-standard.md) | Migration workflow, expand/contract, rollback versus recovery, backup strategy and restore exercises, runbook steps | MIG, DR |
| [Persistence, transactions, and query behavior](docs/persistence-standard.md) | EF Core defaults, transactions and isolation, concurrency tokens, query discipline, parameterization, integration tests on the real engine | DATA |
| [Cache selection, ownership, and invalidation](docs/caching-standard.md) | When to add a cache, key scoping, invalidation and stampede control, failure behavior, Redis for non-cache duties | CACHE |
| [Backup verification and restore runbook](docs/backup-restore-runbook.md) | pg_dump/pg_restore and Fly Postgres commands, quarterly restore drill, cadence, evidence record, production restore steps | (verifies DB, DR) |

## Read by task

See [skills/data-standards/SKILL.md](skills/data-standards/SKILL.md).

## Install as an agent skill

| Agent | Command |
| --- | --- |
| Copilot CLI | `copilot plugin marketplace add Slight76/standards-marketplace` then `copilot plugin install data-standards@slight76-standards` |
| GitHub CLI (any agent) | `gh skill install Slight76/data-standards data-standards --scope user --pin v1.0.0` |
| Claude Code | `/plugin marketplace add Slight76/standards-marketplace` then `/plugin install data-standards@slight76-standards` |

## Layout

| Path | Purpose |
| --- | --- |
| `docs/` | Standards documents (frontmatter, applies-when, rule table) |
| `catalog/catalog.json` | Machine-readable rules; `externalDecisions` points at historic ADRs |
| `adr/` | Decisions local to this handbook |
| `skills/data-standards/` | Agent skill and references |
| `CHANGELOG.md` | Release notes, including where each moved document came from |

Validation: `py ../standards-marketplace/tooling/validate.py --root .`. License: [MIT](LICENSE).
