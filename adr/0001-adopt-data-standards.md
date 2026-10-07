# ADR-0001: Adopt the data standards handbook

Status: Accepted

Date: 2026-10-07

Owner: @Slight76

## Context

The former single `architecture-standards` repository (v0.3.0) mixed every domain into one catalog and skill. Database, persistence, caching, migration, and recovery guidance was spread across `database/` and `backend/` folders, shared one rule catalog with frontend and platform rules, and could only be installed as one oversized agent skill. The team also lacked concrete, PostgreSQL-specific guidance (naming, anti-patterns, a restore runbook) that an agent could apply without interpretation.

## Decision

Create `data-standards` as the home for data standards, versioned independently from the other handbooks and installable as an agent skill. Documents moved here keep their rule IDs and historical ADR references; new documents are first drafts with `status: proposed`.

Net-new documents introduced by this handbook and governed by this ADR: `docs/naming-conventions.md` (NAME-001..NAME-004), `docs/postgres-anti-patterns.md` (PGX-001..PGX-003), and `docs/backup-restore-runbook.md` (no new rules; verification procedure for DB-005 and DR-001). Tables are plural, identifiers are snake_case, every constraint is named, and each module owns a schema.

## Alternatives

- Keep the domain inside `architecture-standards`: rejected; one repository was too broad to read or install selectively.
- Rewrite all rules from scratch: rejected; existing rules are kept verbatim to preserve consumer baselines.
- Place the backup runbook only in `operations-standards`: rejected; the database-specific commands belong next to the migration and recovery policy, with the platform-wide DR plan linking here.

## Consequences

Consumers pin this repository in `architecture-baseline.json` (`standards[]`). Historic decisions remain in `architecture-standards/adr/` and are declared in `catalog/catalog.json` under `externalDecisions`. New NAME and PGX rules are `Proposed` until a consuming repository adopts the baseline.

## Traceability

Rule prefixes: CACHE, DATA, DB, DR, MIG, NAME, PGX. Related: standards-marketplace ADR-0001; architecture-standards ADR-0028..0031. Historic decisions: ADR-0005, ADR-0016, ADR-0018, ADR-0019 (external).

## Verification

`validate.py` passes; `docs.yml` green on `main`; the skill installs through the standards marketplace.

## Approval

@Slight76, 2026-10-07, plan approved in the split planning session.
