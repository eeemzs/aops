# AOPS Server User Guide

_Release Notes:_ Covers what the installed AOPS ships; examples use <slug> for your project slug.

## 1 Introduction

### 1.1 Two programs

#### 1.1.1 Overview

`aops-server` installs, migrates, runs, updates and backs up the AOPS Server on its own machine. The AOPS CLI (`aops`) is a client of a running Server: it selects a Server (a target) and runs domain commands. A Server whose users sign in through South runs in the `south-oracle` authority mode (section 2.4); its clients sign in with `aops auth login` (AOPS CLI User Guide, `aops-cli-user-guide`). The CLI never installs, starts or migrates a Server.

Every `aops-server` command has `--help`; the help is the live contract for flags. Add `--json` for machine-readable output. Lifecycle commands accept `--instance <name>` (default `default`) and `--data-root <path>`.

Keep database URLs and passwords out of command arguments, chat, logs and tracked files. `aops-server` reads them from the environment or from its private env file.

## 2 Install

### 2.1 Save the database connection

#### 2.1.1 Overview

The Server stores its database connection in a private env file, `~/.aops/aops.server.env` by default (or in the directory that `AOPS_CLI_CONFIG_PATH` selects). Supply the URL through the environment, never as an argument:

```sh
AOPS_PG_URL='<private-url>' aops-server env --test-connection --postgres-tls require --yes --json
AOPS_PG_URL='<private-url>' aops-server env --auth trusted-local --postgres-tls require --yes --json
AOPS_REPO_URL='file:///absolute/path/aops.sqlite' aops-server env --auth trusted-local --yes --json
```

`--test-connection` checks the connection without saving it, migrating or changing services. A SQLite test never creates the file. `--candidate` saves a separate connection profile without selecting it; the TUI uses it before Connect.

PostgreSQL stays operator-owned: aops never installs, starts, stops, provisions or removes a database service, database or role. The TLS policy is `require` (encrypts), `verify-full` (also verifies the certificate and host name) or an explicit `disable` (unencrypted).

### 2.2 Set up the installation

#### 2.2.1 Overview

`aops-server setup init` inspects the installation plan without changes; `--apply` records it:

```sh
aops-server setup init --path 1 --postgres-tls require --yes --json
aops-server setup init --path 1 --postgres-tls require --plan-id <preview-plan-id> --apply --yes --json
aops-server setup init --path sqlite --sqlite-file /absolute/path/aops.sqlite --yes --json
```

Path `1` uses the saved PostgreSQL connection; path `sqlite` uses a Server-local SQLite file. Read the exact `installerPlan.planId` from the preview and pass it to `--apply`; a changed plan fails before any change.

Setup records the installation and starts the Server only when its database is current. A database that needs a migration stops it before the start with `aops_server_migration_pending`; the installation stays recorded and stopped. Setup itself never migrates.

### 2.3 First start

#### 2.3.1 Overview

A new database needs its first migration. After setup:

```sh
aops-server migrate --json
aops-server migrate --apply --json
aops-server start --json
aops-server status --json
```

For a new SQLite file, add `--create-database` to `migrate --apply`; only this command creates the file. Clients then connect with `aops setup init --path 4 --api-base-url <url>`.

### 2.4 South sign-in (south-oracle)

#### 2.4.1 Overview

A Server whose users sign in through South runs in the `south-oracle` authority mode. South provides each request's identity, tenant membership and application authority; AOPS keeps project ownership and membership. The mode needs PostgreSQL: a SQLite installation stays trusted-local (`aops_native_sqlite_trusted_local_required`).

South registers the `aops` application, its `aops-api` resource server and the `aops-cli` device client (`oauth-device-public`, scope `aops.api offline_access`, no South operation, no entitlement), enabled for each tenant whose users sign in. The Server accepts only the `aops-cli` client; a token of another client of the application is refused (`403 south_client_forbidden`).

Add the South lines to the Server env file. They are operator-owned, and `aops-server env` keeps them:

```sh
AOPS_SOUTH_ORACLE_URL=https://auth.example.com
AOPS_SOUTH_ORACLE_AUTHORITY_MODE=projection-v2
AOPS_SOUTH_ORACLE_APPLICATION_SLUG=aops
AOPS_SOUTH_ORACLE_AUDIENCE=aops-api
```

The South address is HTTPS, or plain HTTP on exactly `127.0.0.1`, `localhost` or `[::1]`, written as its origin. Optional limits: `AOPS_SOUTH_ORACLE_TIMEOUT_MS` (at most 60000), `AOPS_SOUTH_ORACLE_POSITIVE_TTL_MS` (at most, and by default, 30000) and `AOPS_SOUTH_ORACLE_NEGATIVE_TTL_MS` (at most 10000). Then select the mode and set up the workspace Server:

```sh
aops-server env --auth south-oracle --yes --json
aops-server setup --postgres-tls require --preview --json
aops-server setup --postgres-tls require --apply --json
aops-server migrate --apply --json
aops-server start --json
```

- `env`, `setup` and every `start` check the lines. A missing, unknown or invalid line stops them: `aops_south_oracle_config_missing:<KEY>`, `aops_south_oracle_config_unknown:<KEY>`, `aops_south_oracle_config_invalid:<KEY>`, `aops_south_oracle_url_requires_https`, `aops_south_oracle_projection_v2_required`. The Server process receives exactly these keys.
- `setup` records the mode, and `start` stops when the env file and the installation disagree (`aops_native_auth_provider_state_mismatch`); to change the mode, change the env file and run `setup` again. `aops-server setup init` stays trusted-local in this release.
- The Server still listens on loopback only; a network address is refused (`aops_exposed_server_unsupported`).
- Each request needs one South decision from South's authority projection v2, for the request's bearer token and `x-tenant-id`; the Server may answer it from its cache. A positive decision is reused for at most 30 seconds, and never past the token's or the device binding's expiry, so a revoked sign-in, a suspended membership or a paused application is refused within 30 seconds. A refusal is cached for at most 10 seconds.
- A request without a tenant header is refused before South is asked (`403 south_tenant_hint_required`).
- South's tenant ids are uuids. In the project roles release the Server keeps a verified tenant only in that shape; a verified tenant in any other shape ends the request with `503 south_oracle_unavailable`.
- While South is unreachable, a positive decision still in the cache is used until it expires; a request without one gets `503 south_oracle_unavailable`. A stale decision is never used, and nothing falls back to trusted-local.
- In the project roles release (section 2.5), a South user reaches the health check, its own principal, the tool catalog and the project operations of section 2.5. Everything else answers `403 aops_south_operation_closed`. A South role grants no project by itself: project access comes from the caller's membership in the project (section 2.5).

### 2.5 Project roles (south-oracle)

#### 2.5.1 Overview

Applies to: the project roles release (P1–P2), since AOPS CLI 0.3.55 and AOPS Server 0.2.41. Earlier Server builds keep their existing behaviour.

In the `south-oracle` mode the Server decides every project call itself, from two independent inputs: the capabilities South grants the caller in the tenant, and the caller's role in the target project, which AOPS keeps.

**Capabilities.** The `aops` application declares seven capability keys:

| Capability | What it permits the caller to ask for |
|---|---|
| `agentspace.project.read` | the projects it may see (list, get) |
| `projectman.read` | Projectman reads in a project |
| `projectman.write` | Projectman writes in a project |
| `projectman.admin` | Projectman admin operations in a project |
| `agentspace.project.manage` | the membership and the name or description of a project |
| `agentspace.project.create` | project creation (section 2.5.3) |
| `agentspace.project.admin` | every project of the tenant: list and read them, and manage the membership of any of them |

A request's capabilities are those of the caller's South roles, limited to the application's capability catalog and to the `aops-cli` client ceiling; the ceiling holds all seven. Roles are South data. A typical set:

| Role | Capabilities |
|---|---|
| reader | `agentspace.project.read`, `projectman.read` |
| member | reader + `projectman.write` |
| creator | member + `projectman.admin`, `agentspace.project.manage`, `agentspace.project.create` |
| admin | creator + `agentspace.project.admin` |

A change in South reaches the Server within the 30-second decision cache (section 2.4).

**Project roles.** Each member of a project has one role: `admin`, `write` or `read`. Only the membership decides; the project's `ownerId` records who created it and grants nothing in this mode.

| Capability the operation needs | Condition in the target project |
|---|---|
| `projectman.read` | role `read` or higher |
| `projectman.write` | role `write` or higher |
| `projectman.admin` | role `admin` |
| `agentspace.project.read` | a member, or `agentspace.project.admin` |
| `agentspace.project.manage` | membership calls: role `admin`, or `agentspace.project.admin`; name and description: role `admin` |

An operation that needs several capabilities needs all of them, and the highest role among them. Seeing a tool in the catalog is not a permission: a caller with `projectman.write` cannot write in a project where its role is `read`.

**The target project.** A project call names exactly one project, through the headers `x-project-id`, `x-project-name` or `x-scope-id`, or through `projectId`, `scopeId`, `project` or `projectName` in the query or the body. Membership calls name it in `filter.projectId` (list) or `data.projectId` (add), or through the membership row they address. Every reference must name the same project; otherwise the call is refused as if the project were not visible.

**What a South user reaches.**

- Projectman: the operations of the target project, under the table above.
- Projects: list and get return only the projects the caller is a member of (every project of the tenant with `agentspace.project.admin`), with `id`, `name`, `slug`, `createdAt` and the caller's role.

  ```text
  GET  /api/agentspace/projects
  GET  /api/agentspace/projects/<id>
  GET  /api/agentspace/operations/project/get-project?id=<id>
  ```

- Membership: list, get, add, change the role, remove. The same calls are available through `aops-cli` (`aops project member`, see the AOPS CLI User Guide) and through the tool gateway.

  ```text
  GET    /api/agentspace/project-members?filter={"projectId":"<id>"}&options={"limit":50,"offset":0}
  POST   /api/agentspace/project-members        {"data":{"projectId":"<id>","userId":"<uuid>","role":"write"}}
  GET    /api/agentspace/project-members/<membership-id>
  PATCH  /api/agentspace/project-members/<membership-id>   {"patch":{"role":"admin"}}
  DELETE /api/agentspace/project-members/<membership-id>
  ```

  `filter` and `options` are JSON objects in the query (URL-encoded). `filter.projectId` is required; `filter.userId` and `filter.role` narrow the list; rows come in creation order.

- Name and description of a project (role `admin`; `agentspace.project.admin` does not stand in for it):

  ```text
  PATCH /api/agentspace/operations/project/update-project   {"id":"<id>","patch":{"name":"...","description":"..."}}
  ```

- Project creation, through the tool invoke `agentspace.project.create` (section 2.5.3).

**Membership rules.**

- A role is exactly `admin`, `write` or `read`; a user id is a UUID. Anything else answers `400 invalid_input`.
- A role change touches only the role; the project and the user of a row never change.
- Adding a user twice answers `409 project_member_duplicate`.
- The last `admin` of a project cannot be demoted or removed: `409 project_member_last_admin`. An admin may always be added to a project without one.
- A membership write that `agentspace.project.admin` allows without the `admin` role (rescue) is allowed for membership only, never for Projectman content or the project's name.
- An applied membership write is recorded twice: an authorization record before it runs and an outcome record after it. A rescue write is marked as such.
  - When the authorization record cannot be written, nothing is written: `503 south_audit_unavailable`.
  - When the outcome record cannot be written after the write ran, the write stays applied, the response carries `audit.outcome: null`, and the Server logs the failure.
- A membership list that cannot be read answers `503 south_project_members_unavailable`, never an empty list.

**Refusals.**

| Answer | Meaning |
|---|---|
| `403 south_project_not_visible` | The project is not visible to the caller, or the call's project references disagree. Whether it exists is not revealed. |
| `403 south_project_role_insufficient` | The project is visible, but the caller's role is too low; `required` and `role` name both. |
| `403 policy_forbidden` | A capability the operation needs is missing. |
| `403 south_tenant_forbidden` | The call names another tenant. |
| `403 south_project_scope_conflict` | The call names more than one project. |
| `400 south_project_field_forbidden` | An input field the operation does not take; ids, tenant, scope, owner and author fields always come from the Server. |
| `400 south_project_scope_required` | A membership call without its project. |
| `403 aops_south_operation_closed` | The operation is not open in the `south-oracle` mode. |

**trusted-local.** Nothing changes: the local operator holds every capability, and project visibility follows ownership and membership as before.

#### 2.5.2 Role conversion

This release keeps project roles as `admin`, `write` and `read`, and the database holds that vocabulary with a constraint. `aops-server migrate --apply` converts stored roles once: `owner` → `admin`, `editor` → `write`, `viewer` → `read`. It takes the pre-migration backup first (section 3.3). A stored role outside both vocabularies stops the migration with nothing changed; correct that row, then run the migration again.

#### 2.5.3 Project creation

Applies to: the project roles release (P1–P2), since AOPS CLI 0.3.55 and AOPS Server 0.2.41. Earlier Server builds keep their existing behaviour.

**One write.** A create writes the project's scope, the project, the creator's `admin` membership and the create's activity record in one transaction, on PostgreSQL and on SQLite: all of them or none. The creator is the request's principal; its user id becomes the project's `ownerId` and its membership.

**South (`south-oracle`).** Creation is open only through the tool invoke `agentspace.project.create`; `POST` on the REST route answers `403 aops_south_operation_closed`. The call goes through the gateway like any other route:

1. The invoke gives the create a request id of the Server's own and checks only the input's shape. The input holds `data` and nothing else: any other key beside it, a context key such as `locale` or `tenantId` included, answers `400 project_create_input_forbidden`, in both authority modes and for a preview as well. `data` may hold `name` (required), `slug` and `description`; any other field answers `400 south_project_field_forbidden`. A missing slug is derived from the name (lower case, accents dropped, every run of other characters one dash, none at either end). An empty result answers `400 south_project_slug_required`.
2. Domain admission, then the route policy: the caller needs `agentspace.project.create`. No project role is asked for: the project does not exist yet.
3. `apply: true` is required (`409 apply_required` otherwise). A preview validates without writing.
4. The domain creates, or answers a slug conflict.

**Slug conflict.** A slug is unique in the tenant (`project_slug_tenant_unique`), and that index alone decides between two creates of one slug. A create whose slug is taken writes nothing and answers `409` with `errorCode` `agentspace_operation_failed.project_slug_conflict` and `details`:

| Field | Value |
|---|---|
| `existing` | Only when the caller has a membership in the project holding the slug: `projectId`, `name`, `callerRole` (the current role) and `createdByCaller` (the project's `ownerId` is the caller; information only). |
| `sameRequest` | The request's name and description equal the project's. |
| `recovered` | `createdByCaller` and `sameRequest`, and `callerRole` is `admin`: the caller's own create already happened (a retry after a lost answer). A role is never raised again. |

Two creates of one slug at once give one `201`; the other gets the answer a later retry gets. The gateway does not replay `project.create` for an idempotency key (route policy `safety.idempotency: domain`).

**trusted-local.** The same flow and the same one write; the local operator becomes the project's `admin`. The input holds only `data` here too (`400 project_create_input_forbidden`). Without a slug nothing is checked, as before.

#### 2.5.4 Records

The success record is atomic. A refusal record is attempted, and when it cannot be written the answer says so.

**Request id.** The Server gives every create invoke a new uuid, in both authority modes. It travels beside the input, never in it: a caller cannot set it, and request headers such as `x-request-id` are not used. A successful answer carries it as `audit.requestId`. A retry is a new request with a new id.

**A successful create (both modes).** The create's transaction writes one row in `activity-items`:

| Column | Value |
|---|---|
| `action`, `status` | `project.create`, `success` |
| `sourceKind`, `sourceId` | `system`, `agentspace.project.create` |
| `scopeId`, `projectId` | the new project |
| `payload` | `operation`, `actorUserId`, `slug`, `requestId`, `authorityMode` (`south-oracle` or `trusted-local`); nothing else |

On a trusted-local Server the gateway writes no second record for a successful create. A refused trusted-local create keeps the gateway's own record when the call names a project scope.

**A refused or failed South create (`south-oracle` only).** One event in `sys_event_stores`, in the tenant South verified, for each create that reached the invoke route with a valid envelope, was not a preview and did not succeed:

| Field | Value |
|---|---|
| `eventType` | `aops.project.create.refused` for a definite refusal (`400` input, `403` route policy, `404` tool or admission, `409 apply_required`, `409` slug conflict); `aops.project.create.failed` for a `5xx` or an unknown outcome, for which no claim of zero effect is made |
| `aggregateId` | the request id |
| `eventData` | `operation`, `actorUserId`, `authorityMode`, `status`, `errorCode`; `slug` only when the input's slug is in slug form; `projectId` only when the answer already told the caller (`details.existing`) |

The answer carries the event id as `audit.refusal`. When the event cannot be written, `audit.refusal` is `null` and the Server logs the event type, the status and a reason, never an input value; the HTTP answer does not change. Refusals before the route (sign-in, tenant, closed calls), invalid envelopes and previews are not recorded here.

**Reading the records.** Read-only, bounded by tenant, ordered and limited. PostgreSQL:

```sql
-- The record of one request: the request id is audit.requestId or the event's aggregateId.
SELECT "eventId", "eventType", "aggregateId", "eventData", "occurredAt"
FROM sys_event_stores
WHERE "tenantId" = $1
  AND "eventType" IN ('aops.project.create.refused', 'aops.project.create.failed')
  AND "aggregateId" = $2
ORDER BY "occurredAt" DESC
LIMIT 20;

-- The create record of one project.
SELECT "createdAt", "status", "payload"
FROM "activity-items"
WHERE "tenantId" = $1 AND "projectId" = $2 AND "action" = 'project.create'
ORDER BY "createdAt" DESC
LIMIT 20;
```

SQLite (`eventData` and `payload` are JSON text; `occurredAt` and `createdAt` are epoch milliseconds):

```sql
SELECT "eventId", "eventType", "aggregateId", "eventData", "occurredAt"
FROM sys_event_stores
WHERE "tenantId" = ?1
  AND "eventType" IN ('aops.project.create.refused', 'aops.project.create.failed')
  AND "aggregateId" = ?2
ORDER BY "occurredAt" DESC
LIMIT 20;

SELECT "createdAt", "status", json_extract("payload", '$.requestId') AS "requestId", "payload"
FROM "activity-items"
WHERE "tenantId" = ?1 AND "projectId" = ?2 AND "action" = 'project.create'
ORDER BY "createdAt" DESC
LIMIT 20;
```

No new South capability reads these records; they are read on the Server's database.

## 3 Migrations

### 3.1 One command changes the schema

#### 3.1.1 Overview

Only `aops-server migrate --apply` changes the database schema. `start`, `restart`, `update`, `setup` and `setup init` verify the database and never migrate it. When the database needs a migration, they stop with `aops_server_migration_pending`, list the pending migrations and name the next action, `aops-server migrate --apply`. The Server does not start and the database is unchanged.

`rollback app` never migrates either. It returns to the prior application's schema instead, restoring the update's pre-migration backup when the update's migration ran (see 5.2).

### 3.2 Preview

#### 3.2.1 Overview

`aops-server migrate [--json]` reads the plan without changes. Its `summary` gives the action (`verify-only`, `migrate` or `record-pending`), the pending migration count and tags, and `next` when a migration is pending. `aops-server status` shows the same live summary.

### 3.3 Apply

#### 3.3.1 Overview

`aops-server migrate --apply` runs while the Server is stopped:

```sh
aops-server stop --json
aops-server migrate --apply --json
aops-server start --json
```

It takes the instance operation lock, refuses a running Server (`aops_server_migrate_requires_stopped_server`) and then:

1. takes the backup the plan needs: a pre-migration backup for a nonempty SQLite file; for PostgreSQL, a managed owned-set snapshot or a recorded external snapshot attestation when the plan requires one;
2. applies the plan;
3. verifies it with a fresh read-only plan, which must find nothing left to do;
4. writes the receipt and leaves the Server stopped.

The result names the backup (`backup`, or `null` when the plan needed none, such as an empty PostgreSQL baseline) and, for PostgreSQL, the applied tags (`appliedMigrationTags`). When the migration completes a pending application update, its update record keeps the backup that undoes it.

### 3.4 A missing SQLite file

#### 3.4.1 Overview

A missing SQLite file is created only for a new installation, with `aops-server migrate --apply --create-database`. Without the flag the command stops with `aops_server_migrate_database_missing`: a lost file of an existing installation is never replaced by an empty one.

## 4 Run

### 4.1 Lifecycle

#### 4.1.1 Overview

```sh
aops-server start --json
aops-server stop --json
aops-server restart --json
aops-server status --json
aops-server health --json
aops-server logs --json
aops-server doctor --json
```

`start` and `restart` detach by default. `status` reports the installation, runtime state, liveness and the live migration summary; its exit code follows the runtime state. `health` checks a running Server without changes. `doctor` checks this machine without changes.

The API and the Cockpit share one origin, `http://127.0.0.1:5900` by default; open `/ui` on that origin.

## 5 Update and rollback

### 5.1 Update

#### 5.1.1 Overview

An installed AOPS updates itself. When the Server starts, and once a day while it runs, it checks for a newer version and downloads it. The next start from the Startup entry or the desktop shortcut installs it before the Server starts; the TUI's **Update now** installs a downloaded version at once. Running a newer setup yourself updates the same way.

An update whose version needs a data update takes a backup first, applies the data update, and starts the new version. A version that does not start healthy goes back to the earlier one with that backup.

`aops-server autoupdate status` shows whether updates are on. `aops-server autoupdate off` stops the checks, the downloads, and the automatic installs; `aops-server autoupdate on` turns them on again.

### 5.2 Rollback

#### 5.2.1 Overview

When an updated version does not start healthy, AOPS returns to the earlier version by itself, as 5.1 describes.

To go back to the earlier version by hand, stop the Server and run `aops-server rollback app`; `aops-server rollback app --help` lists what it needs. When the update's data update already ran, the rollback restores the backup taken before it, which loses the data written since; it then asks for `--confirm-data-rewind` and changes nothing without it. Rollback refuses a running Server, and it stops before it changes the installation or the database when the database does not match the earlier version.

## 6 Backup and restore

### 6.1 Backups

#### 6.1.1 Overview

```sh
aops-server backup --json
aops-server backup list --json
aops-server db backup --help
aops-server db restore --help
```

A backup takes and verifies all aops data: a first-party PostgreSQL bundle or a consistent SQLite snapshot. A restore replaces all aops-owned data, needs a stopped Server and explicit `--confirm-data-rewind` approval, and takes a mandatory undo backup of the target first.

`aops-server db list` and `aops-server db which` show the local backups that a restore of the selected backend accepts. PostgreSQL and SQLite backups are not interchangeable: a native backup restores into its own backend. Prefer an explicit `--backup` path over a guessed newest backup.

### 6.2 PostgreSQL backup and restore

#### 6.2.1 Overview

Run `aops-server db backup --json` on the Server machine. Keep the returned backup directory and its `receiptPath` together. The Server reads all aops-owned data directly; no separate PostgreSQL client executables are needed.

To restore, stop the Server, then run `aops-server db restore --backup <backup-directory> --confirm-data-rewind --json`. The target must already have the current compatible aops schema and the same PostgreSQL major version as the backup. All aops data is replaced, not merged; unrelated tables are preserved. A verified undo is taken first. Verify the result before starting the Server again. `aops-server restore` follows the same rules.

A restore does not downgrade migrations. Recovering an interrupted application update is the separate exact-update operation in section 5.2 (`aops-server restore db --help`); do not substitute a normal restore or repeat an uncertain operation blindly. `aops-server host database` native dumps are a separate explicit administration tool, not the aops backup method or an automatic fallback.

### 6.3 SQLite to PostgreSQL transfer

#### 6.3.1 Overview

The only cross-backend direction is a full SQLite → PostgreSQL transfer. Run these commands on the relevant Server machine; `--instance` and `--data-root` select its local installation.

1. Stop the source Server so that no writes arrive after the snapshot. With that SQLite installation selected, run `aops-server db backup --portable --json`. It keeps a verified native SQLite snapshot and returns a portable `backup.bundleDirectory` and `backup.receiptPath`.
2. Keep the containing transfer directory intact: `bundle/` plus its sibling `receipt.json`. It holds every project's data and the ChatV3 keyring; keep it private. To move machines, copy the whole directory privately to the target Server machine. Never copy a live SQLite file alone.
3. Set up the operator-provided PostgreSQL target separately with its private connection and TLS (section 2) and bring its schema current with `aops-server migrate --apply` (section 3); the transfer itself never migrates. Existing aops data in the target is replaced, not merged. There is no project or domain selection.
4. Stop the target Server. With that PostgreSQL installation selected, run `aops-server db restore --backup <transfer-directory>/bundle --confirm-data-rewind --json`. It validates the complete schema, values and target constraints, takes a verified target undo, then replaces all aops-owned rows in one transaction. It never uses CASCADE, rewrites source data, repairs invalid references, switches profiles or starts the Server; foreign data stays outside the replacement.
5. Inspect the result: success includes `services: stopped`, `connectionChanged: false`, row counts, weak-orphan counts and `undo.bundleDirectory`. Start the Server explicitly and verify PM, memory, documents and encrypted chat. Keep the source and the undo until that readback succeeds.

Portable bundles are not eligible for SQLite. Undo uses the same command on the same compatible PostgreSQL target: stop its Server and run `aops-server db restore --backup <undo.bundleDirectory> --confirm-data-rewind --json`. A new verified undo is always taken; there is no option to skip it. Older PostgreSQL dump files are kept but not accepted by `aops-server db restore` or `aops-server restore`; latest selection skips them.

A missing or incompatible schema, invalid JSON or types, UUID case-twin conflicts and hard foreign-key orphans are rejected before replacement; existing weak orphans are preserved and counted. On failure, read the safe error and recovery result: `targetState: unchanged` means nothing was replaced; `unknown` (for example a lost COMMIT acknowledgement) is neither success nor a proven rollback. Keep the Server stopped and recover with the reported verified undo instead of repeating the command blindly.

There is no PostgreSQL portable export, PostgreSQL → SQLite transfer, append or upsert, selected-project import, or automatic backend switch. Backup and restore stay outside the TUI.

## 7 TUI

### 7.1 System Check

#### 7.1.1 Overview

The TUI runs the Server's setup, migration and service commands through `aops-server` and the client steps through `aops`. Set up and Connect record the selection and, when the database needs it, run `aops-server migrate --apply` and start the Server under the same confirmation. The Migrations Fix stops the Server, migrates and starts it again. A selected target that is not this machine's Server (its exact local origin) is shown as that Server; no local repair is offered for it.

On keyboard layouts that type `@`, `#` or `{` with AltGr, paste such values (Ctrl+V) into the TUI's connection fields.

## 8 Troubleshooting

### 8.1 Diagnostics

#### 8.1.1 Overview

- `aops_server_migration_pending`: the database needs a migration; run `aops-server migrate --apply` with the Server stopped, then start it.
- `aops_server_migrate_requires_stopped_server`: run `aops-server stop` first.
- `aops_server_migrate_database_missing`: check the configured SQLite path. Use `--create-database` only for a new installation.
- An interrupted operation: inspect `aops-server operation-status` and `aops-server lock-status` before `reconcile-operation` or `recover-lock`.

Read the full diagnostic with `--json`: `code`, `summary`, `details` and `nextActions`. Never reset a database to clear an error.
