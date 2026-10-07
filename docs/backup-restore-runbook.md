---
title: "Backup verification and restore runbook"
status: proposed
version: 1.0.0
owner: "@Slight76"
---
# Backup verification and restore runbook

Baseline: 1.0.0. Applies when: a team operates a production PostgreSQL database, on Fly.io Postgres or any managed/self-hosted PostgreSQL

Decision: [ADR-0001](../adr/0001-adopt-data-standards.md). Rules become binding when this baseline is adopted; examples explain the policy and do not establish business requirements.

This runbook is the hands-on companion to [Schema delivery and recovery](migration-recovery-standard.md). That document states the policy (DB-005, DR-001, MIG-002); this one lists the commands, the cadence, and the evidence. Platform-wide disaster-recovery planning (which services, in which order, who declares an incident) lives in the operations handbook: [Backup and disaster recovery](https://github.com/Slight76/operations-standards/blob/main/docs/backup-and-disaster-recovery.md). No new rules are introduced here.

## What "backed up" means for us

A store is backed up when all of the following are true:

1. A **physical/continuous** mechanism exists (Fly Postgres snapshots plus WAL archiving where the plan provides it, or the managed provider's point-in-time recovery) and its retention is known.
2. A **logical dump** (`pg_dump` custom format) is taken on a schedule, stored off the database host (object storage in a different account/region), encrypted at rest, and retained per the data classification.
3. Roles, extensions, and configuration needed to recreate the database are captured (`pg_dumpall --globals-only`, the Fly app config, the list of extensions).
4. A **restore drill** has been completed within the cadence below and its evidence recorded.

A green backup job is not evidence; a measured restore is.

## Backup commands

Run these from a maintenance machine or a scheduled Fly Machine, never from the application runtime identity (DB-004). Use a dedicated `backup` role with `pg_read_all_data`.

```bash
# Fly Postgres: open a tunnel first (or run inside the private network).
fly proxy 15432:5432 -a <pg-app>

# Logical dump, custom format, compressed, one file per database.
pg_dump --format=custom --compress=6 --no-owner --no-privileges \
  --file="<db>_$(date -u +%Y%m%dT%H%M%SZ).dump" \
  "postgres://backup@localhost:15432/<db>?sslmode=require"

# Globals (roles, memberships) once per cluster.
pg_dumpall --globals-only --no-role-passwords \
  "postgres://backup@localhost:15432/postgres?sslmode=require" > globals.sql

# Verify the archive is readable before uploading.
pg_restore --list "<db>_<stamp>.dump" | head
```

Upload with a checksum (`sha256sum`) and record the object key, size, and checksum in the backup log. Fly Postgres volume snapshots are managed by Fly (`fly volumes snapshots list <vol-id>`); note the snapshot id and timestamp alongside the dump for the same date.

## Restore drill, step by step

The drill restores the latest backup into an **isolated** target, verifies it, and measures time. Nothing here touches production.

| Step | Command / action | Record |
| --- | --- | --- |
| 1. Provision target | `fly postgres create --name <db>-drill --region <region> --vm-size shared-cpu-1x --volume-size <n>` or a local Docker PostgreSQL of the **same major version** | Target name, version, start time |
| 2. Fetch backup | Download the dump and `globals.sql`; verify `sha256sum` | Backup stamp, checksum match |
| 3. Restore globals | `psql -f globals.sql <target>` (ignore "role already exists" for `postgres`) | Roles created |
| 4. Create database and extensions | `createdb <db>`; `psql -c 'CREATE EXTENSION IF NOT EXISTS ...'` for each extension in the list | Extensions present |
| 5. Restore data | `pg_restore --dbname=<target>/<db> --jobs=4 --no-owner --no-privileges --exit-on-error <dump>` | Wall-clock duration, exit code |
| 6. Schema check | Compare `pg_dump --schema-only` of production and target (`diff`), or confirm the EF Core migrations history table lists the expected last migration | Last migration id matches |
| 7. Row-count check | `SELECT relname, n_live_tup FROM pg_stat_user_tables ORDER BY 1` on both; differences must be explained by writes after the backup time | Table counts within tolerance |
| 8. Application check | Point a staging instance of the application at the target with read-only credentials; run the smoke test suite | Smoke tests green |
| 9. Measure | RTO = time from step 1 to step 8 passing. RPO = production `now()` at backup start minus the newest `updated_at` in the restored data | Measured RTO and RPO vs. targets |
| 10. Tear down | `fly apps destroy <db>-drill`; delete downloaded dumps | Confirmation |

If any step fails, the drill fails. Open an issue, fix the cause (missing extension, oversized dump, wrong version), and rerun. A drill that needed an undocumented manual step is a partial failure: add the step here.

## Point-in-time recovery (when available)

Fly Postgres and most managed providers can restore a snapshot to a new cluster; point-in-time recovery depends on WAL archiving being enabled for the plan. Once a quarter, confirm that the provider path works by restoring the most recent snapshot to a new cluster (`fly postgres create --fork-from <pg-app>` on Fly) and running steps 6 to 10. Record whether the restore point was a snapshot time or an arbitrary timestamp; this determines the real RPO.

## Cadence

| Activity | Frequency | Owner |
| --- | --- | --- |
| Logical dump | Daily (hourly for stores with RPO under 24 h) | Scheduled job |
| Dump readability check (`pg_restore --list`) | Every dump | Scheduled job |
| Full restore drill (steps 1 to 10) | Quarterly, and after any major version upgrade, extension change, or provider change | Store owner (DB-001) |
| Snapshot/fork restore check | Quarterly | Store owner |
| Review of retention and off-site copy | Semi-annually | @Slight76 |

Drills may be scheduled together across stores, but each store gets its own evidence record.

## Evidence to record

Keep one Markdown file per drill under the consuming repository's `docs/evidence/restore-drills/<yyyy-mm-dd>-<db>.md` (or the location the baseline names) containing:

- Store, backup stamp, checksum, PostgreSQL version of source and target.
- Who ran it, start and end time, measured RTO and RPO, the approved targets (DB-005), and whether they were met.
- The last migration id, row-count comparison summary, and smoke-test result link.
- Every deviation from this runbook and the follow-up issue.

Reference the latest record from `implementation-evidence.json` under DR-001. Evidence older than the cadence interval means DR-001 is `failed`, not `not_run`.

## Production restore (real incident)

Follow the incident process in the operations handbook first; this section covers only the database steps and assumes incident authority has been assigned.

1. **Stop the bleeding.** Revoke the application's write grant or scale the app to zero so no new writes land on a corrupt store. Preserve the current state (take a snapshot or dump) before changing anything, even if it is broken; it may contain the only copy of recent writes.
2. **Choose the recovery point.** Latest dump, snapshot, or point-in-time. Write down the chosen time and the expected data loss.
3. **Restore into a new database or cluster**, never over the damaged one, using steps 3 to 7 of the drill. Keep the damaged store for forensics until the incident closes.
4. **Reconcile.** Identify writes after the recovery point from logs, outbox tables, or the preserved damaged store; replay or record them as lost. Document what cannot be recovered.
5. **Switch.** Update the application's connection secret (`fly secrets set`), run the smoke tests, restore write grants, and scale up. Monitor error rates and query latency for the first hour.
6. **Close.** Record actual RTO/RPO against targets, update this runbook with anything learned, and schedule the next drill within thirty days.

Sources: [PostgreSQL backup and restore](https://www.postgresql.org/docs/current/backup.html), [pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html), [pg_restore](https://www.postgresql.org/docs/current/app-pgrestore.html), [Fly Postgres backup and restore](https://fly.io/docs/postgres/managing/backup-and-restore/). The cadence and evidence requirements are our policy.

## Rules referenced

This runbook carries no rules of its own. It is the verification procedure for DB-005 and DR-001 in [Database architecture](database-architecture.md) and [Schema delivery and recovery](migration-recovery-standard.md).

## Exceptions

Use the [exception record](https://github.com/Slight76/standards-marketplace/blob/main/templates/exception.md) for a departure. Record affected rules, scope, compensating controls, approval evidence, expiry, and migration path. Agents must not silently replace defaults.
