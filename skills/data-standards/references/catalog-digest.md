# Catalog digest

Every rule in [catalog/catalog.json](../../../catalog/catalog.json) (version 1.0.0) with its statement, verification, status, and governing decision. Generated from the catalog; regenerate rather than edit by hand.

| ID | Statement | Verification | Status | Decision | Document |
| --- | --- | --- | --- | --- | --- |
| DB-001 | Every persistent store and schema MUST have a named owning application/module and data steward. | Ownership register review | Proposed | ADR-0005 | [database-architecture.md](../../../docs/database-architecture.md) |
| DB-002 | Schema changes MUST use committed migrations with expand/contract compatibility across supported releases. | Migration rehearsal and mixed-version integration tests | Proposed | ADR-0005 | [database-architecture.md](../../../docs/database-architecture.md) |
| DB-003 | Relational integrity MUST use constraints where applicable; indexes MUST be justified by measured access patterns. | Constraint tests and query plan review | Proposed | ADR-0005 | [database-architecture.md](../../../docs/database-architecture.md) |
| DB-004 | Runtime database identities MUST have least privilege and MUST NOT perform production schema administration. | Grant inspection | Proposed | ADR-0005 | [database-architecture.md](../../../docs/database-architecture.md) |
| DB-005 | Production stores MUST define RPO/RTO, retention, backup encryption, and restore-test evidence. | Recovery exercise review | Proposed | ADR-0005 | [database-architecture.md](../../../docs/database-architecture.md) |
| DATA-001 | Persistence MUST parameterize queries, bound results, and preserve module-owned access. | Injection, query-count, and maximum-result integration tests | Proposed | ADR-0018 | [persistence-standard.md](../../../docs/persistence-standard.md) |
| DATA-002 | Multi-write business operations MUST have tested atomicity and concurrency behavior. | Rollback and competing-write tests on the actual engine | Proposed | ADR-0018 | [persistence-standard.md](../../../docs/persistence-standard.md) |
| DATA-003 | Critical queries MUST have measured plans, indexes, and connection/timeout budgets. | Representative query/load report without sensitive data | Proposed | ADR-0018 | [persistence-standard.md](../../../docs/persistence-standard.md) |
| DB-006 | Each store MUST document source-of-truth ownership, classification, lifecycle, and logical/physical design. | ER model, ownership and retention review | Proposed | ADR-0019 | [design-standard.md](../../../docs/design-standard.md) |
| DB-007 | Relational models MUST use deliberate naming/types, integrity constraints, and tenant-consistent keys. | Constraint, precision, deletion, and tenant-link tests | Proposed | ADR-0019 | [design-standard.md](../../../docs/design-standard.md) |
| DB-008 | Derived stores MUST document consistency, invalidation, and rebuild paths. | Source loss/cache flush/rebuild exercise | Proposed | ADR-0019 | [design-standard.md](../../../docs/design-standard.md) |
| MIG-001 | Production migrations MUST be reviewed, rehearsed, serialized, and executed by a separate deployment identity. | Fresh and upgrade migration runs plus grant checks | Proposed | ADR-0019 | [migration-recovery-standard.md](../../../docs/migration-recovery-standard.md) |
| MIG-002 | Destructive schema changes MUST preserve the supported compatibility window and define data-safe recovery. | Mixed-version and interrupted-backfill tests | Proposed | ADR-0019 | [migration-recovery-standard.md](../../../docs/migration-recovery-standard.md) |
| DR-001 | Production stores MUST demonstrate restoration against approved RPO/RTO using measured evidence. | Isolated restore report including keys, roles, and application validation | Proposed | ADR-0019 | [migration-recovery-standard.md](../../../docs/migration-recovery-standard.md) |
| CACHE-001 | Caches MUST define ownership, scoped keys, staleness, invalidation, and loss behavior. | Cross-tenant key, stale-fill race, eviction, and outage tests | Proposed | ADR-0016 | [caching-standard.md](../../../docs/caching-standard.md) |
| CACHE-002 | Correctness-critical state MUST NOT rely on best-effort cache durability. | Architecture review and restart/failover tests | Proposed | ADR-0016 | [caching-standard.md](../../../docs/caching-standard.md) |
| NAME-001 | Database identifiers MUST be lower snake_case ASCII without quoting; tables MUST be plural nouns and columns MUST follow the documented patterns. | Migration SQL review against this document | Proposed | ADR-0001 | [naming-conventions.md](../../../docs/naming-conventions.md) |
| NAME-002 | Every table MUST have an `id` primary key, `created_at`, and `updated_at` as `timestamptz`; date-only values MUST use `date`. | Schema inspection and entity configuration review | Proposed | ADR-0001 | [naming-conventions.md](../../../docs/naming-conventions.md) |
| NAME-003 | Every constraint and index MUST have an explicit name following the `pk_`/`fk_`/`uq_`/`ck_`/`ix_` patterns. | Generated migration diff shows no engine-generated names | Proposed | ADR-0001 | [naming-conventions.md](../../../docs/naming-conventions.md) |
| NAME-004 | Each module MUST own a named schema and its own migration history table; application tables MUST NOT live in `public`. | Schema listing and `DbContext` configuration review | Proposed | ADR-0001 | [naming-conventions.md](../../../docs/naming-conventions.md) |
| PGX-001 | Every foreign-key column MUST have a supporting index, or a recorded review note explaining why the access pattern never needs it. | Schema query listing FK columns without an index returns only documented cases | Proposed | ADR-0001 | [postgres-anti-patterns.md](../../../docs/postgres-anti-patterns.md) |
| PGX-002 | List endpoints and background readers MUST use keyset pagination and bounded page sizes; `OFFSET` pagination MUST NOT be used beyond the first few pages. | Query review and integration test with a page cursor | Proposed | ADR-0001 | [postgres-anti-patterns.md](../../../docs/postgres-anti-patterns.md) |
| PGX-003 | Migrations MUST NOT take long `ACCESS EXCLUSIVE` locks on tables with production traffic; indexes MUST be created concurrently and constraints validated separately. | Migration SQL review and lock-timeout setting in the deployment step | Proposed | ADR-0001 | [postgres-anti-patterns.md](../../../docs/postgres-anti-patterns.md) |

## Decisions

| Decision | Where | Status |
| --- | --- | --- |
| ADR-0001 | [adr/0001-adopt-data-standards.md](../../../adr/0001-adopt-data-standards.md) | Accepted |
| ADR-0005 | [Slight76/architecture-standards](https://github.com/Slight76/architecture-standards/blob/main/adr/0005-database.md) | Accepted |
| ADR-0016 | [Slight76/architecture-standards](https://github.com/Slight76/architecture-standards/blob/main/adr/0016-implementation-decisions.md) | Proposed |
| ADR-0018 | [Slight76/architecture-standards](https://github.com/Slight76/architecture-standards/blob/main/adr/0018-implementation-decisions.md) | Proposed |
| ADR-0019 | [Slight76/architecture-standards](https://github.com/Slight76/architecture-standards/blob/main/adr/0019-implementation-decisions.md) | Proposed |
