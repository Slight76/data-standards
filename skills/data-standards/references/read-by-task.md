# Read by task

Full task-to-document map for the data standards handbook. Each row names the documents to read, in order, and the rule prefixes you will cite. Links are relative to this file.

| Task | Read first | Then | Prefixes |
| --- | --- | --- | --- |
| Choose a storage engine or add a new store | [Database architecture](../../../docs/database-architecture.md) | [Logical and physical database design](../../../docs/design-standard.md) | DB |
| Assign ownership of a schema or store; register a data steward | [Database architecture](../../../docs/database-architecture.md) | | DB |
| Model a new aggregate: entities, keys, constraints, tenant key | [Logical and physical database design](../../../docs/design-standard.md) | [Database naming conventions](../../../docs/naming-conventions.md) | DB, NAME |
| Name tables, columns, indexes, constraints, schemas | [Database naming conventions](../../../docs/naming-conventions.md) | | NAME |
| Choose column types (timestamps, money, ids, enums, JSONB) | [Database naming conventions](../../../docs/naming-conventions.md) | [PostgreSQL anti-patterns](../../../docs/postgres-anti-patterns.md) | NAME, PGX, DB |
| Configure EF Core entity mapping, DbContext schema, naming plugin | [Database naming conventions](../../../docs/naming-conventions.md) | [Persistence, transactions, and query behavior](../../../docs/persistence-standard.md) | NAME, DATA |
| Add an EF Core migration | [Schema delivery and recovery](../../../docs/migration-recovery-standard.md) | [PostgreSQL anti-patterns](../../../docs/postgres-anti-patterns.md) | MIG, PGX, DB |
| Rename or drop a column, split a table (expand/contract) | [Schema delivery and recovery](../../../docs/migration-recovery-standard.md) | | MIG, DB |
| Backfill data in production | [Schema delivery and recovery](../../../docs/migration-recovery-standard.md) | [PostgreSQL anti-patterns](../../../docs/postgres-anti-patterns.md) | MIG, PGX |
| Set up the migration deployment step and database identities | [Schema delivery and recovery](../../../docs/migration-recovery-standard.md) | [Database architecture](../../../docs/database-architecture.md) | MIG, DB |
| Write a repository or query port | [Persistence, transactions, and query behavior](../../../docs/persistence-standard.md) | [PostgreSQL anti-patterns](../../../docs/postgres-anti-patterns.md) | DATA, PGX |
| Fix slow queries, N+1, missing indexes, pagination | [PostgreSQL anti-patterns](../../../docs/postgres-anti-patterns.md) | [Persistence, transactions, and query behavior](../../../docs/persistence-standard.md) | PGX, DATA |
| Design a transaction, choose isolation, handle concurrency conflicts | [Persistence, transactions, and query behavior](../../../docs/persistence-standard.md) | | DATA |
| Size connection pool, set timeouts | [Persistence, transactions, and query behavior](../../../docs/persistence-standard.md) | [PostgreSQL anti-patterns](../../../docs/postgres-anti-patterns.md) | DATA, PGX |
| Review a PR containing SQL, migrations, or EF Core changes | [PostgreSQL anti-patterns](../../../docs/postgres-anti-patterns.md) | [Database naming conventions](../../../docs/naming-conventions.md) | PGX, NAME, MIG |
| Introduce an in-process or distributed cache | [Cache selection, ownership, and invalidation](../../../docs/caching-standard.md) | | CACHE |
| Use Redis for sessions, locks, or idempotency keys | [Cache selection, ownership, and invalidation](../../../docs/caching-standard.md) | [Persistence, transactions, and query behavior](../../../docs/persistence-standard.md) | CACHE, DATA |
| Define RPO/RTO, retention, backup encryption for a store | [Database architecture](../../../docs/database-architecture.md) | [Schema delivery and recovery](../../../docs/migration-recovery-standard.md) | DB, DR |
| Configure backups; verify a dump | [Backup verification and restore runbook](../../../docs/backup-restore-runbook.md) | | DR, DB |
| Run a quarterly restore drill and record evidence | [Backup verification and restore runbook](../../../docs/backup-restore-runbook.md) | [Schema delivery and recovery](../../../docs/migration-recovery-standard.md) | DR, DB |
| Recover production after data loss or corruption | [Backup verification and restore runbook](../../../docs/backup-restore-runbook.md) | [Backup and disaster recovery (operations-standards)](https://github.com/Slight76/operations-standards/blob/main/docs/backup-and-disaster-recovery.md) | DR |
| Record an exception to any rule | [Exception template (standards-marketplace)](https://github.com/Slight76/standards-marketplace/blob/main/templates/exception.md) | | any |

Rule statements for every prefix are listed in [catalog-digest.md](catalog-digest.md).
