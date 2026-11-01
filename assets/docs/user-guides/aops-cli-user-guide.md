# AOPS CLI User Guide

_Release Notes:_ Covers the commands the installed AOPS CLI ships; examples use <slug> for your project slug.

## 1 Introduction

### 1.1 Overview

#### 1.1.1 Overview

AOPS CLI (`aops`) is a client of a running AOPS Server. It selects a Server
target, resolves project context, provides guarded sugar commands, and falls
back to the Server's live agent-tool catalog when needed. It never installs,
starts, migrates or backs up a Server: `aops-server` does that on the Server's
machine (see the AOPS Server User Guide, `aops-server-user-guide`).
The AOPS server is canonical for durable Projectman, Agentspace, and Docman
records; `.aops-cache/**` content in a repository is a refreshable, read-only working
cache.

This guide is long, so agents should not load it into context in full by
default. Start with `aops <family> --help`, locate relevant content with
`aops doc search --local --q "<keywords>"`, and read an exact section with
`aops view doc-page aops-cli-user-guide#<heading>`. If no sugar exists, inspect
the live contract with `aops agent tools --domain <domain> --summary --json`
followed by `aops agent schema --tool <tool-id> --summary --json`.

Keep these two command types distinct:

1. **Sugar commands** make routine work concise and guarded. The running public
   CLI program is always the authoritative command list; use
   `aops <command> --help` for details.
2. **Hosted agent tools** discover domain capabilities dynamically from the
   server. Start with `aops agent tools`, then use `aops agent schema` or
   `aops agent openapi` when needed. Never guess tool names or payloads.

The introduction, concepts, and operating discipline in this document are
authored manually. The top-level `Appendices` section contains separate
subsections generated automatically from CLI registrations. When a new sugar
command is added, run `aops docs user-guide` instead of maintaining a manual
command list.

## 2 Core model

### 2.1 Overview

#### 2.1.1 Overview

The AOPS server owns writes to Projectman, Agentspace, and Docman records. The
CLI provides guarded sugar commands for routine work and live agent-tool
discovery for less common operations. The `.aops-cache/**` tree in a repository is a
reproducible, read-only working cache; editing its Markdown files does not
update server state.

## 3 Shortest setup flow

### 3.1 Trusted-local

#### 3.1.1 Overview

On a computer where AOPS is installed, open the AOPS window, or run:

```sh
aops tui
```

The TUI is one System Check page. It runs the Server steps (database connection, setup, migration, start and stop) through `aops-server` and the client steps (target and agent assets) through `aops`. Choose SQLite for a file on this machine (Node.js 22.16 or newer), or enter an existing PostgreSQL connection and TLS privately. The Server URL defaults to `http://127.0.0.1:5900`. Fix appears only for an eligible repair; the assets row offers Check/Update; F3 opens the sanitized command history. Minimum terminal size is 96x30.

Without the TUI, set up the Server with `aops-server` on its machine first (see the AOPS Server User Guide, `aops-server-user-guide`), then connect this CLI to it:

```sh
aops setup init --path 4 --api-base-url http://127.0.0.1:5900 --target-name local --yes --json
aops setup init --path 4 --api-base-url http://127.0.0.1:5900 --target-name local --apply --yes --json
```

Path 4 accepts a loopback HTTP Server URL only. It probes the running Server, saves and selects the named target and installs the bundled agent assets. It never installs, starts, migrates or stops a Server and never reads the Server's database configuration.

Normal setup installs the bundled agent assets; it does not require or seed a separate server catalog. Secrets never belong in chat, argv or logs.

### 3.2 Existing Server

#### 3.2.1 Overview

A target names the Server this CLI talks to. Read `aops target --help`, then:

```sh
aops target add --name work --api-base-url http://127.0.0.1:5900 --apply
aops target use work --apply
aops target list --json
aops target doctor work --json
```

- `trusted-local` (the default) is a Server on a loopback HTTP address of this machine: `localhost`, `127.0.0.0/8` or `[::1]`. The Server trusts this machine; there is no sign-in.
- `south` is a Server whose users sign in through South: `--auth south --south-url <url> --tenant <uuid>`. Its Server and South addresses are HTTPS, or plain HTTP on exactly `127.0.0.1`, `localhost` or `[::1]`, each written as its origin. Saving any other address as trusted-local is refused with `aops_target_remote_requires_south`.
- A saved `south` target needs a sign-in on this machine, `aops auth login --target <name>` (section 8). Until then a command that selects it stops with `aops_auth_login_required:<name>` and contacts nothing. `aops target doctor` reads its compatibility and health without a credential.
- One target is active at a time; `aops target use` switches between, for example, a personal and a work Server. A command's `--api-base-url` selects a Server for that command only, under the same rules.

```sh
aops target add --name team --api-base-url http://127.0.0.1:15961 --auth south --south-url https://auth.example.com --tenant <tenant-uuid> --use --apply
aops auth login --target team
aops pm board list --project-id <project-uuid> --json
```

A south-oracle Server listens on loopback only in this release (AOPS Server User Guide, section 2.4). From another machine, an operator-managed SSH tunnel forwards a local loopback port to the Server's loopback port, as in the example; aops does not create the tunnel. A trusted-local Server can be reached the same way and saved as a trusted-local target. Never expose a trusted-local Server directly on a network.

With SQLite, every client uses the same Server over HTTP; clients never open or synchronize its database file.

## 4 `init`

### 4.1 Overview

#### 4.1.1 Overview

`aops init` creates `.aops/aops.config.json` and the required local metadata
skeleton in the current repository. It does not create hosted
Projectman/Docman records. Read `aops init --help` for current options and write
guards before applying it.

## 5 Server administration

### 5.1 Overview

#### 5.1.1 Overview

Installing the Server, saving its database connection, migrations, start and stop, updates, backups and restores run on the Server's machine with `aops-server`, not with this CLI:

```sh
aops-server env --help
aops-server setup init --help
aops-server migrate --help
aops-server status --json
aops-server db --help
```

The AOPS Server User Guide (`aops-server-user-guide`) describes each step, including the full SQLite → PostgreSQL transfer and backup and restore. From a client, `aops host health`, `aops host diagnostics` and `aops host database status` read the selected Server without changing it. Never put database URLs or passwords in command arguments.

## 6 Local and remote access

### 6.1 Overview

#### 6.1.1 Overview

Local access is trusted-local on a loopback address; it needs no sign-in and asks for no first administrator. A Server whose users sign in through South needs a `south` target (section 3.2) and a sign-in (section 8); without one the CLI does not contact it. Do not attempt local identity bootstrap or database administration as a setup repair.

## 7 User management

### 7.1 Overview

#### 7.1.1 Overview

The public CLI has no `aops user` family; `aops auth` signs this CLI in to a `south` target (section 8). Local access is trusted-local.

Applies to: the project roles release (P1–P2), since AOPS CLI 0.3.55 and AOPS Server 0.2.41. Earlier Server builds keep their existing behaviour.

On a south-oracle Server, South provides user identity, tenant membership and application authority; AOPS keeps projects and their members. Two independent things decide a project call: the capabilities South grants the user in the tenant, and the user's role in the target project (`admin`, `write` or `read`). An operation needs both: the capability it requires and, in that project, the role the capability asks for. A South role such as `admin` grants no project by itself, and a project's `ownerId` records who created it and grants nothing: access comes from the membership row.

- Projectman operations of a project need `projectman.read`, `projectman.write` or `projectman.admin`, and the role `read`, `write` or `admin` there.
- Project list and get show the projects the user is a member of; with `agentspace.project.admin`, every project of the tenant.
- The members of a project and its name and description need `agentspace.project.manage` and the role `admin` in the project. For members only, `agentspace.project.admin` stands in for the role (rescue), and such a write is recorded as rescue. It never opens Projectman content or the project's name.
- Creating a project needs `agentspace.project.create`. The creator becomes the project's first `admin` (section 16.3).
- Every other operation answers `403 aops_south_operation_closed`.

`aops project member` (section 16.2) manages members. The AOPS Server User Guide, section 2.5, has the capability and role tables and the refusals.

Use live discovery for the capabilities the selected Server actually exposes. Access changes require explicit authority.

## 8 Login and tokens

### 8.1 Overview

#### 8.1.1 Overview

A trusted-local target needs no sign-in. A `south` target signs in through South with a browser-approved device flow:

```sh
aops auth login --target team          # shows a South address and a code; approve them in the browser
aops auth status --target team --json  # the sign-in, and whether the Server confirms it
aops auth refresh --target team        # renews the access token
aops auth logout --target team         # signs out at South first, then removes the sign-in here
```

- `aops auth login` opens the South page when it can; `--no-browser` prints the address and the code only. Approve the code in your own South browser session, for the target's tenant.
- South approves the fixed `aops-cli` client for the `aops` application and the `aops-api` audience, with the scope `aops.api offline_access`; no option changes them. A south-oracle Server accepts only this client.
- The Server confirms the sign-in. After South approves it, `auth login` stores it and reads it back from the Server on the path every command uses: `/api/auth/me` answers, uncached, the user, the South tenant, the client, the application and the audience South verified for the request. A sign-in the Server does not confirm ends at South and is not kept (`aops_auth_session_binding_mismatch`, `aops_auth_session_rejected`, `aops_auth_session_readback_failed`). If South does not confirm that end, the command fails with `aops_auth_login_failed_remote_unconfirmed`; the sign-in may stay active at South until it expires, so sign the device out in your South account. If another sign-in was stored meanwhile by another process, it stays. `auth status` reports a Server answer that names another user, tenant or client as `rejected`.
- The sign-in names the target's tenant to South. South keeps a device client such as `aops-cli` per tenant, so the same client signs in to each tenant separately. A `south` target without a tenant stops before any request (`aops_auth_profile_tenant_required`): add the tenant to the target, then sign in. Sign-ins made before this keep renewing.
- The sign-in belongs to one `south` target and is bound to its South address, Server address and tenant. Every request carries `x-tenant-id` with that tenant; a request naming another tenant (`--tenant-id`) is refused before it is sent (`aops_auth_tenant_header_mismatch`). When the target's addresses or tenant change, the sign-in is not used (`aops_auth_target_changed`): sign out, then sign in again.
- It lasts 30 days by default. Whether it outlives the approving browser's sign-out is the `aops-cli` client's session setting at South, independent or browser-bound, as the South administrator registered the client. Signing the device out at South ends it; the next renewal is refused (`aops_auth_refresh_rejected`). Then run `aops auth logout`, then `aops auth login`.
- The Server keeps a positive South decision for at most 30 seconds, so it refuses a revoked sign-in, a suspended membership or a paused application within 30 seconds.
- While South is unreachable, a decision still in that cache is used until it expires. After that, or without one, the Server refuses with `503 south_oracle_unavailable`. A stale decision is never used, and nothing falls back to trusted-local. `aops auth status` reports an unreachable South and keeps the sign-in.
- Tokens stay in an encrypted local record whose key is in the OS vault (on Windows, the Credential Manager). On a POSIX host without a vault, `AOPS_CLI_CREDENTIAL_STORE=file` keeps an owner-only key file. Tokens are never printed.
- `aops auth logout --local-only --confirm` removes the sign-in here without asking South, for example while South is unreachable.

Never write tokens to JSON output, an issue, or documentation.

## 9 Diagnostics

### 9.1 Overview

#### 9.1.1 Overview

Start with the non-mutating client checks: `aops doctor`, `aops target doctor <name>`, `aops auth status` for a `south` target, and `aops host health`. On the Server's machine, `aops-server status`, `aops-server health` and `aops-server logs` show the Server's state. Do not reset, reinstall, or perform database work before identifying a fault. Verify the active target, port, and data root first.

## 10 Owner model and domain selection

### 10.1 Overview

#### 10.1.1 Overview

Use the owner surface when a sugar command exists. Otherwise, discover the live
tool with `aops agent tools --domain <domain> --summary --json` and read its
payload contract with `aops agent schema --tool <tool-id> --summary --json`.
Chat owns coordination, Projectman owns plans and reviews, Discuss owns
decisions, and Docman owns document truth.

## 11 Projectman sugar commands

### 11.1 Overview

#### 11.1.1 Overview

`aops pm` is the server-first write surface for board, task, sprint, microtask,
issue, feedback, and review-request records. Read the relevant subcommand's
`--help` output before making changes. Do not move plan or review truth into a
room message or local mirror.

## 13 `.aops` local cache & sync

### 13.1 Overview

#### 13.1.1 Project registry, `authoringMode`, and `localRoot`

Multi-project repos use `.aops/aops.config.json` as a project registry. The
hosted server is the source of truth for project identity and for all
Projectman/Agentspace records; the repo registry only records which local
directory mirrors a hosted project as a read-only cache:

```bash
aops-cli project link --slug <slug> --mode local --local-root .aops-cache/projects/<slug> --apply --json
aops-cli project link --slug demo --mode hosted-only --apply --json
aops-cli project links list --json
aops-cli project migrate-local-root --project-slug <slug> --local-root .aops-cache/projects/<slug> --dry-run --json
aops-cli project migrate-local-root --project-slug <slug> --local-root .aops-cache/projects/<slug> --apply --confirm --json
```

Contract:

1. `project link/links` manages only the repo project registry after verifying
   the hosted project exists and is not archived/deleted.
2. `authoringMode: local` means a local cache directory is materialized under
   `localRoot` (normally `.aops-cache/projects/<slug>`). Create/write/read still go to
   the hosted server; `localRoot` is a read-only mirror refreshed by
   `aops-cli sync pull` (and `aops-cli doc mirror pull` for docs), not a
   repo-first source tree.
3. `authoringMode: hosted-only` means no local cache directory is materialized.
   Reads and writes both use the hosted gateway directly.
4. `migrate-local-root` is repo-local cache relocation. Always run `--dry-run`;
   the real move requires `--apply --confirm` because old flat roots are
   archived.
5. hosted-only-vs-local is only a cache-presence decision: local mode keeps a
   refreshable read-only mirror on disk, hosted-only mode reads straight from
   the server. Neither makes the repo the source of truth.

#### 13.1.2 Partitioned sync

Project-partitioned `sync pull` refreshes the local cache using the same
project registry selector contract. There is no `sync push`: the hosted server
is canonical, so the cache is only ever pulled, never pushed back.

```bash
aops-cli sync status --project-slug <slug> --json
aops-cli sync pull --project-slug <slug> --apply --json
aops-cli sync pull --project-slug <slug> --only projectman --apply --json
aops-cli sync pull --project-slug <slug> --only agentspace --apply --json
aops-cli sync pull --hosted-project-slug <slug> --only hosted-skills --apply --json
aops-cli sync status --all-projects --json
aops-cli sync pull --all-projects --apply --json
```

Rules:

1. `sync --project-slug/--all-projects` refreshes the read-only cache of hosted
   Projectman and Agentspace records. It is separate from
   `--hosted-project-slug`, which refreshes the read-only hosted prompt/skill
   mirrors.
2. `--project-slug` resolves the linked project, then refreshes that project's
   `localRoot` cache when `authoringMode` is `local`.
3. `--all-projects` runs once per repo-config local project and reports
   project-level results without fail-fast. Hosted-only links have no cache to
   refresh on this path; local links without a usable `localRoot` are
   reported/skipped rather than treated as another project's cache.
4. Because the server is canonical, conflict/drift resolution is not part of the
   pull: a refresh simply overwrites the local cache with current server state.
5. `sync pull` and `sync bootstrap` accept repeatable `--only` flags with
   `projectman`, `agentspace`, `hosted-skills`, and `hosted-prompts`. Omitting
   `--only` preserves the complete refresh. Prefer a scoped synchronous pull
   when only one cache family changed so completion and errors remain visible.
6. Docman is intentionally not a sync partition. Refresh document mirrors with
   `aops-cli doc mirror pull`; `--only docman` is rejected rather than silently
   doing the wrong operation.

#### 13.1.3 Archive lifecycle

`aops-cli archive` prepares hosted Projectman graph cleanup from a local bundle.
It is deliberately a CLI composition over existing hosted Projectman surfaces,
not a new hosted archive domain.

```bash
aops-cli archive create --project-slug <slug> --apply --json
aops-cli archive verify --manifest .aops/archive/<slug>/<ts>/manifest.json --apply --json
aops-cli archive delete --manifest .aops/archive/<slug>/<ts>/manifest.json --json
aops-cli archive delete --manifest .aops/archive/<slug>/<ts>/manifest.json --apply --confirm --json
aops-cli archive decommission-check --manifest .aops/archive/<slug>/<ts>/manifest.json --json
```

Rules:

1. `archive create` downloads the hosted PM graph into
   `.aops/archive/<slug>/<timestamp>` and records `pendingDomains`; it does not
   delete anything.
2. `archive verify --apply` re-fetches hosted PM data, compares counts and
   checksums, then persists `verification.status: passed` into the manifest.
3. `archive delete` without `--apply` is a preview. Destructive delete requires
   a verified manifest plus `--apply --confirm`.
4. Delete order is children-before-parents so review requests, feedback,
   issues, microtasks, sprints, tasks, columns, and boards are removed in a
   dependency-safe sequence. The manifest records per-action deletion state for
   resumability.
5. `archive decommission-check` permits full project/scope decommission only
   when the manifest is verification-passed, `decommissionSafe` is true, and
   `pendingDomains` is empty. Current bundles can still list Agentspace memory,
   discussions, chat, and hosted prompt/skill/resource/artifact domains as
   pending until those owners have their own archive coverage.

#### 13.1.4 Agent-tool catalog verification

No new hosted `archive.*` tool is expected for this slice. The CLI verifies or
composes existing hosted surfaces:

1. `project link` verifies hosted projects through existing
   `agentspace.project.*` tools, then writes the repo registry.
2. hosted-only PM direct commands use existing `projectman.*` tools.
3. archive cleanup composes existing Projectman read/delete tools and records a
   local manifest.

Spot-check the catalog before writing raw hosted payloads:

```bash
aops-cli agent tools --domain agentspace --q project --summary --json
aops-cli agent tools --domain projectman --q delete --summary --json
```

### 13.2 AOPS Markdown view sugar

#### 13.2.1 Overview

The `view` command family is a read-only presentation layer. Default commands
read `.aops-cache/**` files from the read-only local cache. Canonical truth remains on
the hosted server, and `sync pull` refreshes the cache. Explicit hosted commands
(`hosted-projects`, `hosted-inventory`) call hosted list APIs for reads only;
they do not sync, write caches or indexes, or run domain mutations. Default
output is agent/TUI-friendly Markdown; `--json` returns the same read model in
a stable envelope.

Command set:

```bash
aops-cli view dashboard --style agent
aops-cli view projects
aops-cli view hosted-projects --style compact
aops-cli view hosted-inventory --hosted-project aops --style compact
aops-cli view boards
aops-cli view board <selector>
aops-cli view tasks
aops-cli view task <selector>
aops-cli view sprints
aops-cli view sprint <selector> --max-items 20
aops-cli view issues
aops-cli view feedback
aops-cli view memory
aops-cli view resume
aops-cli view discussions
aops-cli view discussion <selector>
aops-cli view experience
aops-cli view skills
aops-cli view prompts
aops-cli view docs
aops-cli view doc <selector>
aops-cli view doc-page <doc-selector>#<heading-selector>
aops-cli view digest --task <selector> --depth deep --max-bytes 32768
```

Selector resolution for all commands that accept a `<selector>` argument:

```bash
# full UUID
aops-cli view task <task-uuid>

# 8+ character ID prefix (the UUID's first 8 characters or the -<8>.md filename suffix)
aops-cli view task <uuid-prefix>

# slug
aops-cli view board ops

# exact title/name
aops-cli view sprint "AOPS CLI view follow-up"

# doc-page composite selector (document#heading)
aops-cli view doc-page <document-slug>#<section-slug>
```

An ambiguous selector fails and returns a candidate table or JSON payload. If
`<selector>` is missing or matches more than one record, choose the correct
target from the candidate list.

Common flags available on all view commands:

| Flag | Meaning | Default |
|------|-------|---------|
| `--json` | Return a stable JSON envelope instead of Markdown | false |
| `--style agent\|compact\|wide` | Markdown density and format profile | `agent` (ASCII, emoji-free, link-mode none) |
| `--link-mode none\|relative\|absolute` | Path-link behavior | `none` |
| `--max-items <n>` | Maximum rows per list or table | 25 |
| `--max-bytes <n>` | Total Markdown budget; hard cap 32768 | 32768 |
| `--project-id\|--project-name\|--project-slug <v>` | Select a project that is not active in repo config | active project |

Additional flags for hosted view commands:

| Flag | Meaning | Default |
|------|-------|---------|
| `--api-base-url <url>` | Hosted API base URL | env/default host |
| `--access-token <token>` | Hosted API access token | auth config/env |
| `--refresh-token <token>` | Hosted API refresh token | auth config/env |
| `--timeout-ms <ms>` | Hosted request timeout | client default |
| `--tenant-id <id>` | Agent gateway tenant header | - |
| `--locale`, `--fallback-locale` | Agent gateway locale headers | - |
| `--scope-id <id>` | Hosted scope override | repo/project context |
| `--scope-resolution explicit\|cascade` | Hosted asset scope resolution | `explicit` for inventory |
| `view hosted-inventory --hosted-project <selector>` | Narrow by hosted project id, slug, name, or 8+ character prefix | all fetched projects |

Footer contract shown beneath every view output:

```text
- source: <relative-path-or-directory>
- local-state: local|dirty|synced|conflict|deleted|-
- updatedAt: <iso>
- lastPushedAt: <iso?>
- lastPulledAt: <iso?>
- truncated: true|false
```

`local-state` semantics are computed through `effectiveLocalState`: a record
reported as `synced` is marked `dirty` when drift is detected between
`baseHash` and current content. Trust this computed value, not the raw
`syncState` field.

Practical guidance for `view digest`:

1. The default `--depth shallow` provides enough summary for agent context.
2. Use `--depth deep` for detailed inspection, but reduce the `--max-bytes`
   budget. A single-sprint deep digest can stay near 8 KB, as in the example.
3. If the footer reports `truncated: true`, retry with a lower `--max-items`
   value or a narrower selector.

Typical usage scenarios:

```bash
# Agent kickoff: read the active window with one command
aops-cli view dashboard --style agent

# Sprint resume: phase/microtask + linked memory + discussions
aops-cli view sprint <sprint-id>

# Pipe a context pack to Codex/Claude Desktop
aops-cli view digest --sprint <sprint-id> --depth deep | pbcopy

# Rendered terminal reading with mdcat/glow
aops-cli view board <board-slug> | glow -p

# JSON for scripts and automation
aops-cli view tasks --json | jq '.result.data[] | select(.localState == "dirty") | .label'

# Hosted project inventory: project table first, then docs/skills/prompts/resources groups
aops-cli view hosted-inventory --hosted-project aops --style compact

# Debug selector ambiguity
aops-cli view task Duplicate --json | jq '.error.candidates'
```

Session-state nudges:

1. `view dashboard --style agent` can scan `.aops-cache/agentspace/session-state/**`
   read-only and show a `Session State Nudges` section.
2. This section does not write memory. It only gives the agent runtime-hygiene
   signals such as "checkpoint overdue" or "write a summary."
3. When a nudge requires a response, use the owner command
   `aops-cli mem checkpoint` or `aops-cli mem summary`.

Filter flags for each listing command:

```bash
# Memory: durability/kind/subject/id
aops-cli view memory --durability sticky --kind rule
aops-cli view memory --subject sprint --id <sprint-id>
aops-cli view memory --subject task --id <task-id-prefix>
aops-cli view resume --subject project

# Projectman tasks: board + status (column name resolves)
aops-cli view tasks --board ops --status Done
aops-cli view tasks --board engineering --status Doing

# Projectman issues: status + severity + board/sprint/task
aops-cli view issues --status open
aops-cli view issues --severity high --status resolved
aops-cli view issues --board ops --sprint <sprint-id>

# Projectman feedback: status + board/sprint/task
aops-cli view feedback --status open --board ops

# Projectman sprints: board + status
aops-cli view sprints --board ops --status doing

# Discussions: status + participant
aops-cli view discussions --status concluding
aops-cli view discussions --agent claude

# Experience: type + area
aops-cli view experience --type technique
aops-cli view experience --area memory
```

Filter contract:

| Command | Filter flags | Meaning |
|-------|------------------|-------|
| `view memory`, `view resume` | `--durability`, `--kind`, `--subject`, `--id` | durability=short\|durable\|sticky; kind=kickoff\|resume\|closeout\|note\|rule\|...; subject=project\|board\|sprint\|task\|ktask\|utask\|issue\|feedback; id=full UUID or 8+ character prefix |
| `view tasks` | `--board`, `--status` | board=slug/name/id; status=column name or slug (Done, Todo, Doing, Backlog) |
| `view issues`, `view feedback` | `--status`, `--severity`, `--board`, `--sprint`, `--task` | status=frontmatter status; severity=low\|medium\|high\|critical; board/sprint/task=the relevant subject relation |
| `view sprints` | `--board`, `--status` | board=slug/name; status=todo\|doing\|completed\|paused\|... |
| `view discussions` | `--status`, `--agent` | status=active\|concluding\|concluded\|abandoned; agent=an agent id in participants |
| `view experience` | `--type`, `--area` | type=technique\|tool\|script\|problem-solution\|idea; area=a tag in areas[] |

Multiple filters use AND semantics. If the result is empty, the table falls
back to `No matching records.`.

Owner-boundary rules for `view`:

1. Cache-reading view commands (`dashboard`, `boards`, `tasks`, `issues`,
   `feedback`, `memory`, `skills`, `prompts`, `docs`, `digest`, ...) read only
   `.aops-cache/**/*.md` files from the read-only local cache. They do not call hosted
   tools, sync, write caches, mutate state, or write to `~/.aops`.
2. Hosted view commands (`hosted-projects`, `hosted-inventory`) call only these
   hosted read/list tools: `agentspace.project.list-projects`,
   `docman.document.list`, `agentspace.skill.list-skills`,
   `agentspace.prompt.list-prompts`, `agentspace.resource.list-resources`.
   They do not mutate, sync, refresh mirrors, or write caches.
3. Cross-domain joins follow existing frontmatter/API fields (`subjectType`,
   `subjectId`, `boardId`, `sprintLocalId`, `pmContext.taskId`, and so on);
   they do not invent new domain semantics.
4. `view skills` and `view prompts` read `.aops-cache/hosted/**` mirror files.
   Canonical truth remains in hosted Agentspace/server state, and view does not
   refresh it.
5. `view docs` and `view doc-page` read the `.aops-cache/docman/**` mirror. The
   read-only mirror banner and `pulledAt` appear in the footer.
6. Projectman planning views read from the cache. Use `view boards`,
   `view tasks`, `view sprints`, `view issues`, and `view feedback` for PM
   tables.

Surfaces deferred to V2:

1. `view relations <selector>` cross-domain RelationResolver
2. `view skill/prompt/experience <selector>` detail inspect (V1 list-only)
3. `aops-cli ls` and `aops-cli show` aliases
4. `--out <path>` generated artifact writer (V1 stdout default)
5. A deeper cross-domain edge resolver for the hosted relation graph
6. A cache/index if performance requires it

Critical rules:
- Server wins: the hosted server is canonical. `sync pull` reflects server state
  into the read-only local cache. There is no `sync push` (removed in S4), and
  the repository is not the source of truth.
- UI/server changes arrive in the cache on the next `sync pull`. Manual cache
  edits are not canonical and are overwritten during refresh.
- Derived views are computed from the cache; run `sync pull` to keep it current.
- Reusable `prompt` and hosted `skill` shell/version truth remains in the
  server/DB. `sync pull` only creates read-only mirrors under `.aops-cache/hosted/**`.
- A repository may intentionally pull another project's hosted prompt/skill
  mirror with `--hosted-project-id|name|slug`.
- `sync pull` is a project-level server-to-cache refresh. Use
  `--hosted-project-id|name|slug` to refresh hosted prompt/skill mirrors.

Startup flow:

1. Run `aops-cli init` for a new repository.
2. Refresh the read-only hosted-state cache with `aops-cli sync pull --project-slug <slug> --apply --json`.
3. Read `.aops-cache/projectman/views/index.md` and `.aops-cache/agentspace/memory/index.md` for cached context.
4. When reusable prompt/skill context is needed, read `.aops-cache/hosted/index.md`, `.aops-cache/hosted/skills/index.md`, and `.aops-cache/hosted/prompts/index.md`.
5. Use `aops-cli pm ...` for PM authoring, `aops-cli mem ...` for memory, `aops-cli exp ...` for agent experience, and `aops-cli discuss ...` for the agent discussion workspace. Each writes directly to the hosted server.

Shutdown flow:

1. Update PM/memory records on the hosted server through owner commands (`aops-cli pm ...`, `aops-cli mem ...`).
2. Run `aops-cli sync pull --apply --json` if you want a current cache readback.

Memory/handoff distinction:
- `--write-memory` writes an opt-in memory side effect after the main PM
  mutation succeeds. The AI default should remain short memory.
- `pm handoff write` writes a kickoff/resume/decision/blocker/closeout/rule
  memory record outside the mutation.
- `pm handoff resume` reads a curated resume pack for an existing tracked PM
  subject; it does not create a subject record.
- Durable `note` and sticky `rule` records are operator-controlled. They are not
  written by default while an agent works and require an explicit request.

Phase note:
- `phase` is a first-class planning concept in Projectman, but there is no
  standalone `phase.*` CRUD operation family today.
- `phase` is the nested grouping/status layer of a sprint plan.
- Therefore, AOPS sugar retains the existing `pm sprint` + `pm utask` surface
  instead of adding `pm phase ...`.

### 13.3 Private control root and generated cache layout

## 15 Prompt sugar

### 15.1 Overview

#### 15.1.1 Overview

`aops prompt` manages reusable prompt shells and versions server-first. Use
`list|get|inspect|current` for reads, `create|update|version create` for writes,
and `version publish` for publication. `.aops-cache/hosted/prompts/**` is only a
read-only mirror.

## 16 Project sugar

### 16.1 Overview

#### 16.1.1 Overview

`aops project` separates the hosted project record from the repo-local project
registry link. `list|get` reads hosted records; `link` attaches only a verified
slug to the local registry. For local-root relocation, first run
`migrate-local-root --dry-run`, then use `--apply --confirm` only for the
reviewed target.

### 16.2 Project members

#### 16.2.1 Overview

Applies to: the project roles release (P1–P2), since AOPS CLI 0.3.55 and AOPS Server 0.2.41. Earlier Server builds keep their existing behaviour.

`aops project member` manages who belongs to a project and with which role: `admin`, `write` or `read`. The target project is `--project-id`, or `--project-slug` resolved against the Server, or by default the repo-bound project. The command sends only that project's headers, whatever the repo binding is.

```bash
aops project member list --project-slug <slug> --json
aops project member list --project-slug <slug> --role admin --user me --limit 50 --offset 0 --json
aops project member add --project-slug <slug> --user <user-id> --role write --preview --json
aops project member add --project-slug <slug> --user <user-id> --role write --apply --json
aops project member set-role --project-slug <slug> --id <membership-id> --role admin --apply --json
aops project member remove --project-slug <slug> --id <membership-id> --apply --confirm --json
aops project member inventory --needs-admin --json
```

Rules:

1. `--user me` is the principal the Server confirms for the current sign-in. Any other user is a user id (a UUID); on a `south-oracle` Server it is the user's South id. Ids are accepted in any letter case.
2. Writes need `--apply`, and `remove` also needs `--confirm`. `--preview` validates the call without writing.
3. `set-role` and `remove` read the membership first and refuse a row of another project (`project_member_project_mismatch`) before any write.
4. Every write prints the Server's readback: the row after `add` or `set-role`, the remaining members after `remove`. When callers demote or remove themselves, the write is applied, but the Server may refuse the read afterwards: `readback.ok` is then `false` with the refusal code, and the write is not undone.
5. `list` pages with `--limit` and `--offset`, in the order the members were added.
6. `inventory` is read-only: every project the caller can list, with its member and admin counts. `--needs-admin` keeps the projects without an admin.
7. The Server keeps at least one admin per project: demoting or removing the last one answers `project_member_last_admin`.
8. On a `south-oracle` Server the caller needs `agentspace.project.manage` and the `admin` role in the project, or `agentspace.project.admin` (section 7.1.1; the AOPS Server User Guide, section 2.5). `aops project get` and `aops project update` likewise send only the project they are about.

### 16.3 Project creation

#### 16.3.1 Overview

Applies to: the project roles release (P1–P2), since AOPS CLI 0.3.55 and AOPS Server 0.2.41. Earlier Server builds keep their existing behaviour. An earlier Server answers creation on a `south-oracle` target with `403 aops_south_operation_closed`.

`aops project create` creates a project. The Server writes the project, its scope and the creator's `admin` membership together, or nothing. The creator is the signed-in user (on a `south-oracle` Server the South user; on a trusted-local Server the local operator).

```bash
aops project create --name "Atlas" --slug atlas --description "Shared work" --preview --json
aops project create --name "Atlas" --slug atlas --description "Shared work" --apply --json
```

Rules:

1. A create is about no existing project: it sends no project headers, not even the repo-bound project's, and reads no project first. It takes no `--project-id`, `--project-name` or `--scope-id`. The CLI sends the input as `data` only; a Server refuses any other key beside it with `400 project_create_input_forbidden`.
2. Writes need `--apply`; `--preview` validates the call without writing.
3. On a `south-oracle` Server the input is `--name`, `--slug` and `--description` only. `--status`, `--visibility` and `--project-type` are refused with `400 south_project_field_forbidden`. Without `--slug` the Server derives one from the name: lower case, accents dropped, every run of other characters one dash. A name that gives no slug is refused with `400 south_project_slug_required`; pass `--slug`.
4. A slug is unique in the tenant. The CLI sends no idempotency key of its own; the slug protects against a double create.

#### 16.3.2 Results and exit codes

| Answer | Meaning | Exit code |
|---|---|---|
| `201` | Created. The JSON result is the project with `membership` (`userId`, `role: admin`). | 0 |
| `409 project_slug_conflict`, `recovered: true` | Already created by you with the same name and description, and you are still its `admin` (a retry after a lost answer). The CLI prints "Project already created; your admin membership is current." | 0 |
| `409 project_slug_conflict`, `recovered: false` | The slug is taken. `existing` (project id, name, your role there, whether you created it) is shown only when you are a member of that project. | 1 |
| any other refusal | As for every command. | 1 |

`--json` prints `result.recovered`, `result.sameRequest` and `result.existing` for a slug conflict. A conflict never changes a role: a creator who is no longer `admin` there gets `recovered: false`.

#### 16.3.3 `--idempotency-key`

The Server does not replay a create for an idempotency key; the slug decides. With a slug, a retry with the same key answers like any retry (section 16.3.2). Without a slug, a trusted-local retry creates a second project.

#### 16.3.4 Records

The Server keeps these records of a create (AOPS Server User Guide, section 2.5.4):

- **A created project:** exactly one activity record, written in the same transaction as the project: the creator, the slug, the Server's request id for the call and the authority mode. A project never exists without it.
- **A refused or failed create on a `south-oracle` Server:** the Server attempts one event in the tenant South verified, for a create that reached the tool invoke with a valid envelope and was not a preview. When the event cannot be written, the answer says so (`audit.refusal` is `null`) and the result does not change; the record is attempted, not guaranteed. A refusal before the invoke (sign-in, tenant, a closed call), a malformed envelope and a preview get no event.
- **A refused create on a trusted-local Server:** it keeps the gateway's own record only when the call names a project scope. `aops project create` names none, so its refusals have no such record.

The request id is the Server's own: the CLI sends none, and a retry is a new request. The CLI prints the project or the refusal; the records are read on the Server.

## 17 Durable memory and synopsis sugar

### 17.1 Overview

#### 17.1.1 Overview

Recommended agent memory path:

```bash
aops-cli mem brief --subject project --json
aops-cli mem checkpoint --content "Slice in progress." --task-id <task-id> --sprint-id <sprint-id> --apply --json
aops-cli mem summary --content "Session summary." --apply --json
```

Rules:

1. `mem brief` is the read-only startup pack used at session start or resume. It
   does not replace PM state and does not write memory.
2. `mem checkpoint` writes short rolling status at a meaningful milestone,
   decision, blocker, or handoff point. Do not use it for every chat line or
   small edit.
3. `mem summary` is for session end or an operator summary request. An ordinary
   summary remains short. A durable closeout is written only with
   `--closeout --durability durable --confirm`.
4. Memory should be an evidence pack containing the request/purpose,
   board/task/sprint/issue references, concrete outcome, validation/review
   evidence, open risks, and next action.

### 17.2 Choosing a coordination surface

#### 17.2.1 Overview

`discuss`, `chat`, and `pm review-request` do different jobs:

- `aops-cli discuss` holds decisions: turns, final stances, and conclude outputs (`aops-cli-discuss`).
- `aops-cli chat` carries coordination and wake messages in an encrypted ChatV3 room (`aops-cli-chat`). `aops-cli chatv3` is a compatibility alias of the same commands.
- `aops-cli pm review-request` holds reviews, results, and re-reviews when the session uses aops pm (`aops-cli-projectman`).

A second agent on the same computer can also work in a tmux session; then that session carries the conversation and no room is needed.

| Need | Command | Skill |
|---|---|---|
| Decision topic, turns, and final stances | `aops-cli discuss start`, `aops-cli discuss turn`, `aops-cli discuss wait`, `aops-cli discuss conclude` | `aops-cli-discuss` |
| Join a room with your own saved session | `aops-cli chat join --code <room-code> --handle <you> --apply` | `aops-cli-chat` |
| Send a short coordination or wake message | `aops-cli chat send --session <session> --room <room> "<message>"` | `aops-cli-chat` |
| Read new messages from your saved cursor | `aops-cli chat read --session <session> --room <room> --after-seq <cursor>` | `aops-cli-chat` |
| Wait for new messages | `aops-cli chat listen --session <session> --room <room> --after-seq <cursor> --timeout-sec 55` | `aops-cli-chat` |
| Request a review, record a result, or re-review | `aops-cli pm review-request create`, `aops-cli pm review-request result` | `aops-cli-projectman` |

A room or tmux message only wakes the other agent. The Discuss transcript is the record of a decision; with aops pm on, the review request is the record of a review. The writer and the listener must use the same room: a mismatched room is the most common cause of lost messages. `chat listen` exits `0` when messages arrived and `22` on a quiet timeout; mark messages delivered and read only after you processed them (`aops-cli-chat` has the cursor rules).

Slug-first operator contract:

1. `discuss start --slug <slug>` writes the canonical slug to the topic. If omitted, it is derived from the title and returned as `topicSlug` in JSON.
2. Selectors treat the exact slug as the operator-facing default. Legacy folder names and short ids are debug fallbacks. Use slugs in operator-facing commands, handoffs, and messages; keep raw UUIDs out of normal traffic except for debugging or JSON.
3. If a legacy folder-name or implicit short-id match wins, the JSON envelope returns `cliDeprecationWarnings`. Treat these as a signal to move to slugs.
4. Use `--short-id <8char>` when an explicit debug selector is necessary; this explicit route emits no warning. If resolution is ambiguous and only one active record exists, `--prefer-active` selects it. Durable handoffs still store the slug.
5. A new agent first lists its own saved sessions (`aops-cli chat session list --json`) and reads the room from its saved cursor; with aops pm on, it also lists pending reviews (`aops-cli pm review-request list --status requested --json`).
6. `discuss conclude` must not leave `_TBD_` placeholders in the consensus, disagreement, or open-questions outputs. The agent that drives the topic owns them and finalizes them.

### 17.3 PM window, chat-room binding, and active window

#### 17.3.1 Overview

With aops pm on, two agents share one Projectman window. If a board kickoff already exists, reuse that board's active task and sprint; create a new task and sprint only when no active window exists or the operator opens one on purpose.

```bash
aops-cli pm board kickoff --board <board> --title "<task>" --goal "<goal>" --apply --json
# When the agents meet in a room, bind the board there; the room only carries coordination
aops-cli chat binding add --session <session> --room <room> --binding-type projectman.board --ref-id <board-id> --title "Active board" --json
```

Use `pm sprint` and `pm utask` for an explicit task or sprint selection. With aops pm off there is no PM window: the plan and the reviews stay in the conversation with the other agent.

### 17.4 Two-agent research and deliberation flow

#### 17.4.1 Overview

When the operator asks agents to discuss a decision, research together, or propose a plan together, run a Discuss topic with two real, independent agents. Do not write the plan alone and ask for a review afterwards. `aops-cli-discuss` has the full ritual; the other agent works in a tmux session or meets you in a room, as the session's kickoff chose.

Driving agent:

1. Start the topic with both agents: `aops-cli discuss start --slug <slug> --title "<decision>" --agent <you> --agent <other> --question "<question>" --apply --json`. Put the request, repository roots, constraints, and expected outputs in the first turn.
2. Ask the other agent, in its tmux session or in the room, for independent research: a current-state map, recommendations, tradeoffs, risks, and open questions, not a review of your draft.
3. Research on your own and write your stance with `discuss turn`. Wake the other agent where it works with a short message that names the topic and the turn.
4. Wait with `aops-cli discuss wait --topic <slug> --agent <you> --timeout-sec 540 --interval-sec 5 --json`: exit `0` is your turn, `20` an open operator question (return to the operator), `21` ready to conclude, `22` a timeout (wait again).
5. Read the other agent's full turn, not only a summary message, and answer with agreements, corrections, and pushback.
6. Continue until there are at least four substantive turns that are not final stances. Then each agent writes a `final-stance` turn.
7. Confirm from `aops-cli discuss status --topic <slug> --json` that no final stance is missing, then run `discuss conclude` and finalize the consensus, disagreement, and open questions with no `_TBD_`.
8. Implementation waits for the concluded topic and, where the repository requires it, for the operator's approval of the consensus. A topic is never concluded on a timeout: when the other agent does not answer, tell the operator and continue other work.

Other agent:

1. Acknowledge the request where you were asked.
2. Research the requested sources yourself; the driving agent's draft is not the only truth.
3. Write your research and stance with `discuss turn`, then wake the driving agent where it works with the topic and turn.
4. Answer evidence-backed pushback with another turn that states which points you accept, which you keep, and which you leave to the operator; finish with your own `final-stance` turn.

The output is the concluded decision, open operator questions, and the implementation order. Write it into Projectman (with aops pm on) or Docman only with explicit commands.

### 17.5 ChatV3 product-channel room context

#### 17.5.1 Overview

`aops-cli chat` is the ChatV3 command tree: encrypted channels and rooms, agent-owned local sessions, messages, cursors, presence, and bindings. `aops-cli chatv3` is a compatibility alias of the same commands; new work uses `aops-cli chat`. Member tokens and key material stay in the local session store of the agent that joined; use only your own saved session.

Common commands:

```bash
aops-cli chat listen --session <session> --room <room> --after-seq <last-seq> --timeout-sec 55 --json
aops-cli chat binding add --session <session> --room <room> --binding-type projectman.review-request --ref-id <rr-id> --title "Slice review" --json
aops-cli chat binding list --session <session> --room <room> --json
aops-cli chat room brief --session <session> --room <room> --for <agent> --json
aops-cli chat room summary --session <session> --room <room> --after-seq <last-seq> --json
```

The ChatV3 User Guide has the room-code, session, cursor, and receipt rules.

## 18 Resource sugar

### 18.1 Overview

#### 18.1.1 Overview

`aops-cli resource` is the hosted Agentspace surface for durable knowledge pointers. A resource describes where knowledge lives; it does not own the document body, snapshot bytes, or planning state.

Use it when an agent needs a reusable pointer such as a guide, rule, spec, link, reference, template, dataset, code note, or skill-related reference:

```bash
aops-cli resource create --name "Hexagen Guide" --resource-type document --uri "docman:aops/hexagen" --apply --json
aops-cli resource list --resource-type document --json
aops-cli resource get --id <resource-id> --json
aops-cli resource update --id <resource-id> --uri "https://example.test/spec" --apply --json
aops-cli resource delete --id <resource-id> --apply --confirm --json
```

Before raw hosted writes, check the live schema:

```bash
aops-cli agent schema --tool agentspace.resource.create --timeout-ms 120000 --json
```

If `agent schema` ever returns only a flexible `data` envelope for an Agentspace operation, fall back to the matching sugar help (`aops-cli resource create --help`) and inspect an existing list/get record before composing payloads. Do not guess nested field names from memory.

## 19 Artifact sugar

### 19.1 Overview

#### 19.1.1 Overview

`aops-cli artifact` is hosted metadata for generated or external artifacts. It stores the artifact shell and links to project-scoped refs; it is not the byte store. Use artifact records for pointers such as storage paths, report paths, exported archives, screenshots, or generated JSON.

Core flow:

```bash
aops-cli artifact create --artifact-type file --storage-path "s3://bucket/report.json" --apply --json
aops-cli artifact link --artifact-id <artifact-id> --ref-type resource --ref-id <resource-id> --apply --json
aops-cli artifact ref list --ref-type resource --ref-id <resource-id> --json
aops-cli artifact get --id <artifact-id> --json
aops-cli artifact delete --id <artifact-id> --apply --confirm --json
```

Keep artifact content small and pointer-shaped. If the artifact is a repo file, include the path and validation context in memory or PM; do not paste large file bodies into artifact metadata.

## 20 Skill sugar

### 20.1 Overview

#### 20.1.1 Overview

`aops-cli skill` owns hosted reusable skill shells and skill versions. `.aops-cache/hosted/skills/**` is only the read-only mirror; never edit it as canonical truth.

Authoring loop:

```bash
aops-cli skill list --hosted-project-slug <slug> --name "<skill-name>" --json
aops-cli skill inspect --id <skill-id> --json
aops-cli skill version list --skill-id <skill-id> --json
aops-cli skill version create --hosted-project-slug <slug> --skill-id <skill-id> --content '@./SKILL.md' --entry-file SKILL.md --skill-standard aops-skill-v1 --meta '@./meta.json' --apply --json
aops-cli skill version publish --hosted-project-slug <slug> --id <skill-version-id> --apply --json
aops-cli sync pull --apply --hosted-project-slug <slug> --json
```

When `--version` is omitted, the CLI resolves the next version from hosted versions. If a publish or create reports a version conflict, use `skill version list`; mirror frontmatter can lag by one version immediately after publish. Hosted authoring does not rewrite global runtime skills. Those change only through a separately prepared and installed public asset release.

## 21 Durable activity logs

### 21.1 Overview

#### 21.1.1 Overview

Durable activity logs are audit/readback evidence for hosted operations. Many hosted write sugars append best-effort activity records or structured server logs. Treat these logs as verification context, not planning truth.

Current operator rules:

1. Planning and execution state still belongs in Projectman.
2. Durable handoff and decisions still belong in Agentspace memory.
3. Activity logs are useful when proving that a hosted write, invoke, or flow ran with a concrete request/response.
4. If a dedicated `aops-cli activity ...` command is not present in your runtime, discover the hosted activity surfaces with `aops-cli agent tools --domain agentspace --json` or use the domain guide. Do not invent an activity command from old docs.

## 23 Docman sugar

### 23.1 Overview

#### 23.1.1 Overview

`aops-cli doc` is the hosted Docman surface for document groups, documents, versions, sections, pages, page versions, retrieval rows, publish output, and mirror pull.

Common flows:

```bash
aops-cli doc list --project-slug <slug> --json
aops-cli doc version list --document-id <doc-id> --json
aops-cli doc outline get --document-version-id <docver-id> --titles-only --depth 2 --json
aops-cli doc page draft-save --page-version-id <pagever-id> --document-link-id <document-section-link-id> --content '@./page.md' --apply --json
aops-cli doc set-current-version --document-id <doc-id> --version-id <docver-id> --publish-now --apply --json
aops-cli doc mirror pull --project-slug <slug> --group-uid <group-uid> --document-slug <document-slug> --out-dir ./.aops-cache/docman --apply --json
```

For a known page edit, use clone_all + targeted page/section CRUD. For a whole markdown refresh, use `doc import --from-markdown` with `--baseline`, `--guard-target`, and a `--dry-run` first.

Retrieval notes:

1. `doc scope search --local` searches the local mirror and can rank source or architecture files above the guide you expected.
2. `doc search --local` is document-granular and confirms presence; do not assume it returned a full section body.
3. To read a mirror section, use `aops-cli view doc-page <document>#<number-prefixed-slug>`, for example `aops-cli view doc-page release-checklist#2-before-the-release`.
4. `doc outline get --titles-only --depth <n>` is the cheapest structure probe for a known hosted version.

## 24 Common operator notes

### 24.1 Overview

#### 24.1.1 Overview

High-signal operator notes:

1. First cold hosted calls can take around 20 seconds; use `--timeout-ms 120000` for schema, doc, skill, or file smoke commands.
2. `--yes` means non-interactive/fail-fast. It is not a magic fix for validation errors.
3. In PowerShell, quote file pointers for multiline or quote-heavy content: `--content '@file'` or `--input '@file.json'`.
4. If sugar returns validation errors, stop retrying guessed flags. Read `<command> --help`; for raw invokes, read `agent schema`.
5. If command help and skill text disagree, command help wins.
6. If a user guide and skill text disagree, the user guide wins.

## 25 Recommended daily flow

### 25.1 Overview

#### 25.1.1 Overview

For a normal agent session:

```bash
aops-cli mem resume --subject project --json
aops-cli view dashboard --style agent
aops-cli <family> --help
# do the scoped work
aops-cli mem write --mode resume --subject project --durability short --content '@./checkpoint.md' --apply --json
```

Use `view` for read-only context, then mutate through the owner family: `pm` for planning, `mem` for durable context, `doc` for documents, `skill`/`prompt` for hosted reusable assets.

Do not run closeout commands unless the operator explicitly approves closeout. Ordinary stop points should write resume/handoff memory.

## 26 Installer model

### 26.1 Overview

#### 26.1.1 Overview

Runtime skills are loaded from the user-level Codex and Claude homes, not from repository mirrors. AOPS installs complete public skill directories from one versioned asset release. It does not install pointer files and runtime use does not depend on an active repository checkout.

Routine use:

```bash
aops assets status --json                 # installed and bundled versions; no network
aops assets status --check-latest --json  # also the latest published release and what an update would do
aops assets install --target all --json   # the release bundled with AOPS
aops assets update --target all --json    # the latest published release, when it is newer
aops assets rollback --target all --json  # the previous release; no network
```

`assets update` reads the published `assets-<version>` releases of the public `eeemzs/aops` repository. It installs the highest one only when it is newer than the installed release, not older than the bundled one, and when this `aops` meets the release's `minCliVersion`; otherwise it changes nothing and says why. Before it installs, it checks that the release's tag, manifest, and archive belong together and match their sizes and SHA-256 digests, and it stops a check or download that takes longer than 45 seconds. The same version with other bytes is an error, whatever its source. `aops assets update --from-release <directory>` installs a reviewed local release instead.

`install`, `update`, and `rollback` replace the skills of the installed and the new release in each runtime they update. A skill that only an older release kept in `~/.aops/assets` carried is removed where the runtime still holds that release's exact files; a skill of your own under that name stays. Unrelated user skills remain untouched, and a failure restores the prior working files. One install, update, or rollback runs at a time, and a release that needs a newer `aops` is refused before anything changes, whatever its source. `install` and `update --from-release` also take `--expect-current <version>` (`none`: no release installed yet): when another release is installed by then, they change nothing and stop with `aops_assets_current_changed:<installed version>`. A lock left by an `aops` that no longer runs is taken over. `aops_assets_install_lock_stuck` means a lock whose owner cannot be read, or a takeover that was itself interrupted: once no `aops` command runs, remove `~/.aops/assets/install.lock` and `~/.aops/assets/install.lock.takeover`, whichever exists, and try again. After the new install succeeds, the retired `~/.aops/agent-assets` store is removed. A symlink or non-directory at that exact legacy path fails before new asset writes.

`~/.aops/assets` stores downloaded releases, current/previous state, and backups. Public releases contain skills, user guides, working disciplines, and roles. The release contract intentionally has no gateway, pointer, receipt, signature, commit identity, or machine identity layer.

Start a new Codex or Claude Code session after an install or update so that it loads the new skills.

## 27 Help-first model

### 27.1 Overview

#### 27.1.1 Overview

AOPS command discovery is help-first:

```bash
aops-cli --help
aops-cli <family> --help
aops-cli <family> <subcommand> --help
```

Decision chain:

1. Use sugar help for routine CLI work.
2. Use `aops-cli agent tools --domain <domain> --json` to find hosted operations when sugar is missing.
3. Use `aops-cli agent schema --tool <domain>.<operation> --json` before raw payload authoring.
4. Use `aops-cli agent invoke --tool <id> --input '@payload.json' --apply --json` only when sugar is absent or broken.
5. Use `aops-cli api call` only as an explicit low-level escape hatch.

For a host-admitted domain command, optional `cli:v1` metadata can provide a path, aliases, help, schema-bound field flags and defaults. An exact native command leaf wins; valid admitted metadata is next; otherwise the generated command spelling remains available, including for an unknown metadata version. Inspect the live input with `aops agent schema --tool <tool-id>` before a raw invoke. Metadata does not grant access or replace the host's apply/confirm gate. The app CLI owns transport, local files and output.

## 28 Guard-flag convention

### 28.1 Overview

#### 28.1.1 Overview

Guard flags are consistent across AOPS sugar:

| Flag | Meaning |
|---|---|
| `--preview` | Validate and describe the operation without mutation. |
| `--apply` | Execute a guarded write. |
| `--confirm` | Confirm destructive actions such as delete, restore overwrite, cleanup, or reset. |
| `--idempotency-key <key>` | Make write retries deterministic when the command supports it. |
| `--json` | Return scriptable structured output. |
| `--yes` | Non-interactive/fail-fast mode for prompts and missing choices. |

Read commands need no guard. Normal writes require `--apply`. Destructive writes require `--apply --confirm`. `--preview` without `--apply` should not mutate; if a command mutates during preview, file an issue.

## 29 Hosted source of truth and mirror cache

### 29.1 Overview

#### 29.1.1 Overview

The hosted server is the source of truth. The `.aops-cache/**` tree is a read-only
local cache; do not treat it as canonical or hand-edit it as truth.

Server-canonical, mirrored as read-only caches:

1. `.aops-cache/projectman/**` caches Projectman boards, tasks, sprints, issues, feedback, and views. Refresh with `aops-cli sync pull --project-slug <slug> --apply --json`.
2. `.aops-cache/agentspace/memory/items/**` caches Agentspace memory. Refresh with `aops-cli sync pull ...`.
3. `.aops-cache/agentspace/discussions/**` caches discuss topics/transcripts (discuss authoring is hosted). Refresh with `aops-cli sync pull ...`.
4. `.aops-cache/hosted/prompts/**` and `.aops-cache/hosted/skills/**` mirror hosted Agentspace prompt/skill current versions. Refresh with `aops-cli sync pull --apply --hosted-project-slug <slug> --json`.
5. `.aops-cache/docman/**` mirrors hosted Docman documents. Refresh with `aops-cli doc mirror pull ...`.

To change content, write through the owner surface (`aops-cli pm ...`, `aops-cli mem ...`, `aops-cli discuss ...`, `aops-cli doc ...`), then refresh the cache.

## 30 Guides and document mirrors

### 30.1 Overview

#### 30.1.1 Overview

The AOPS user guides are installed with the agent assets; `aops assets status --json` reports their folder as `userGuidesRoot`. Read them there; they need no server.

Your project's own documents are Docman-owned, not `sync pull`-owned. Mirror them with:

```bash
aops-cli doc mirror pull --project-slug <slug> --apply --json
aops-cli doc mirror pull --project-slug <slug> --document-slug <document-slug> --out-dir ./.aops-cache/docman --apply --json
```

Use `sync pull` for hosted prompts/skills, and `doc mirror pull` for documents.

## 31 Cross-cutting anti-patterns

### 31.1 Overview

#### 31.1.1 Overview

Avoid these:

1. Guessing flags after a validation error instead of reading `--help`.
2. Writing raw `agent invoke` payloads without `agent schema`.
3. Treating `aops-cli` as semantic owner for planning, memory, docs, files, or domain business state.
4. Hand-editing `.aops-cache/hosted/**` or `.aops-cache/docman/**` mirrors.
5. Assuming `sync pull` refreshes Docman guides.
6. Running full Docman import for a one-page edit when CRUD ids are known.
7. Writing closeout memory for an ordinary checkpoint.

## 32 Skill and user-guide search discipline

### 32.1 Overview

#### 32.1.1 Overview

Use the smallest useful read:

```bash
aops-cli <family> --help
aops-cli doc scope search --project-slug <slug> --q "<keywords>" --local --json
aops-cli doc outline get --document-version-id <docver-id> --titles-only --depth 2 --json
aops-cli view doc-page <document-slug>#<number-prefixed-section-slug>
```

Search rules:

1. Search by document title + section name + keywords, not bare section numbers.
2. `doc scope search` is broad; verify the `documentSlug` and `mirrorPath` before trusting a hit.
3. `doc search --local` is not a section body reader by itself.
4. `view doc-page <document>#<number-prefixed-slug>` is the ergonomic local section reader.
5. If a skill is thin, follow its canonical guide pointer rather than expecting full mechanics in the skill body.

## 33 AGENTS.md prompt-template bootstrap

### 33.1 Overview

#### 33.1.1 Overview

`aops-cli agents-md` manages generated AGENTS.md prompt-template blocks. Keep project-specific rules outside the managed block.

```bash
aops-cli agents-md preview --collab
aops-cli agents-md update --collab --apply
aops-cli agents-md reset --apply --confirm
```

Use this when a repo needs the standard AOPS task execution or collaborative work protocol reminders. Do not manually edit the managed block unless recovering from a broken generated state.

## 34 Agent runtime prompt/skill bootstrap

### 34.1 Overview

#### 34.1.1 Overview

AOPS installs its reviewed agent asset bundle (skills, user guides, working disciplines and roles) for Codex and Claude. Check it, or install it again, before asking an agent to use the AOPS skills:

```bash
aops assets status --json
aops assets install --target all --json
```

Destinations:

1. Codex skills: `~/.codex/skills/<skill>/SKILL.md`
2. Claude skills: `~/.claude/skills/<skill>/SKILL.md`
3. Local bundle store, user guides, disciplines, and roles: `~/.aops/assets/releases/<version>/files/**`; `aops assets status --json` reports the user guides folder as `userGuidesRoot`.

`assets install` uses the bundle shipped with AOPS; `assets update` installs a newer published release (26.1). To repair from another reviewed local bundle directory, use:

```bash
aops assets update --from-release <reviewed-local-directory> --target all --json
```

The Skills row of the AOPS window (TUI) runs these same commands and does not own a second installer.

## 35 `start` kickoff composer

### 35.1 Overview

#### 35.1.1 Overview

`aops-cli start` composes the hosted "AOPS Collaborative Startup" starter
prompt from kickoff answers. The command's live help is canonical:

```bash
aops-cli start --help
```

Two usage modes:

1. Operator TTY: `aops-cli start` asks questions interactively and writes the
   prompt to stdout or to a file with `--out <file>`.
2. Agent interview: `aops-cli start --json` returns questions in
   `result.missing` with an `askOperator` marker. The agent supplies known
   answers as flags and asks the operator only for `askOperator` items. Roles
   are operator-only.

Default `--json` output is compact. Instead of returning the prompt body inline,
it points to it with `result.promptRef`; `--out tmp/start.md` keeps a long prompt
in a file. Request an inline prompt with `--full-output` only when necessary.

Frequently used commands:

```bash
aops-cli start --mode solo --board <board> --task "<task>" --json --out tmp/start.md
aops-cli start --mode chat-room --board <board> --discipline build-review-chat --json
aops-cli start --reminder --task "<current task>" --area <area> --limit 3 --json
```

Important fields in a ready result:

1. `result.promptRef.path` / `result.promptRef.sha256` - prompt file and hash.
2. `result.memoryBrief` - read-only local-cache memory startup pack; `--no-memory-brief` skips it.
3. `result.sessionGuidance` - layered runtime rules, discipline guardrails, accepted playbooks, ranked experience briefs.

`start --reminder` is not a full kickoff: it asks no questions, does not
serialize the full starter prompt, and writes no PM, memory, or hosted state. It
provides a bounded read-only answer to mid-session questions such as "where
were we, and which rules, playbooks, or experience items apply?"

## 36 Implementation plan

### 36.1 Overview

#### 36.1.1 Overview

An implementation plan is a Projectman sprint facade, and the plan id is the
sprint id. It does not replace Projectman task, sprint or review truth; it is
the same record seen as a plan.

```bash
aops-cli plan create --task <task-id> --name "<plan name>" --goal "<goal>" --apply --json
aops-cli plan get --id <sprint-id> --json
aops-cli plan update --id <sprint-id> --phases-json '@tmp/phases.json' --apply --json
```

Rules:

1. Tasks, sprint phases/microtasks, issues, feedback, and review requests and
   results belong to Projectman.
2. Use `aops-cli pm utask ...` for single-item microtask edits; the plan facade
   is not a second table.
3. Bind an accepted discuss consensus to the plan with a reference, and get the
   plan reviewed before implementation.

## 37 Playbook and experience consult

### 37.1 Overview

#### 37.1.1 Overview

Playbooks and experience are not bulk-loaded at startup. Read a bounded pack
first:

```bash
aops-cli start --reminder --task "<current task>" --area <area> --limit 3 --json
aops-cli view experience --area <area>
aops-cli skill current --id <skill-id> --summary --json
```

Read `result.sessionGuidance` in three layers:

1. L1 runtime pointers: AGENTS.md, ChatV3/channel rules, command refs.
2. L2 discipline guardrails: id/title/phase/enforcement/evidence summary.
3. L3 accepted playbook briefs + ranked experience briefs.

The default experience limit is 3 and the hard maximum is 5. If a brief is
irrelevant, do not read the full body. If it is relevant, use the
skill/prompt/current summary, memory/experience detail, or document-reading
ladder for a targeted read.

## 38 Checkpoint cadence

### 38.1 Overview

#### 38.1.1 Overview

Checkpoint memory is a resume evidence pack, not a transcript. Do not write one
for every chat line or small edit. Write it at a meaningful milestone,
decision, blocker, RR/RRR result, import/publish slice, or session handoff.

```bash
aops-cli mem checkpoint --subject sprint --id <sprint-id> \
  --content '@tmp/checkpoint.md' \
  --task-id <task-id> --sprint-id <sprint-id> \
  --source-ref "projectman.review-request:<rr-id>" \
  --validation-state "PASS: tests/typecheck/smoke" \
  --next-action "<next action>" \
  --apply --json
```

A good checkpoint contains:

1. Request/purpose and PM surface references.
2. Concrete outcome and current status.
3. Validation/review evidence.
4. Open risks/blockers.
5. Next action and next-read references.

Durable closeout memory requires operator approval; an ordinary checkpoint
remains a short resume/carry-forward record.

## 39 Appendices

### 39.1 Overview

#### 39.1.1 Overview

This top-level section contains reference appendices generated deterministically
from source code and live registration surfaces. Editorial guide sections are
maintained manually; do not edit the marker blocks below by hand.

<!-- aops-generated:cli-command-catalog:start -->

### 39.2 Generated CLI command catalog

#### 39.2.1 Overview

> This section is generated from the public Commander program registered by `buildAopsProgram()`. Do not edit it by hand; regenerate it with `aops docs user-guide`.

#### 39.2.2 Command families

| Command | Purpose |
| --- | --- |
| `aops activity` | Agentspace activity ledger read sugar over the hosted AOPS gateway |
| `aops agent` | Primary operator plane for federated tool discovery and invoke |
| `aops agent-profile` | Author and read hosted Agentspace agent profiles (server-first) |
| `aops agents-md` | Manage AOPS prompt-template bootstrap blocks in AGENTS.md |
| `aops api` | Direct domain API escape hatch (/api/{domain}/...) |
| `aops archive` | AOPS archive bundle commands for hosted Projectman graph cleanup preparation |
| `aops artifact` | Agentspace artifact sugar commands over the hosted AOPS gateway |
| `aops assets` | Install, update, and roll back the AOPS skills, guides, disciplines, and roles: bundled, local, or the latest published release |
| `aops auth` | Sign in to a south target through South |
| `aops bootstrap` | Read the host bootstrap contract |
| `aops chat` | ChatV3 encrypted channels, rooms, agent sessions, coordination, and foreground listening |
| `aops chatv3` | Compatibility command tree for canonical `aops chat`. ChatV3 encrypted channels, rooms, agent sessions, coordination, and foreground listening |
| `aops checkpoint` | Server-first session checkpoint facade over hosted Agentspace memory |
| `aops codex` | Experimental local agent-runtime integrations for Codex Desktop |
| `aops config-root` | Print the brand-specific user config root |
| `aops discuss` | Manage hosted (server-first) Agentspace discussion topics |
| `aops doc` | Docman authoring, retrieval, and publish sugar over the hosted AOPS plane |
| `aops docs` | Generate AOPS documentation sections from live CLI registrations |
| `aops doctor` | Run mutation-free checks of this CLI and the selected server target |
| `aops exp` (aliases: experience) | Author and read hosted Agentspace experience items (server-first) |
| `aops health` | Read host health |
| `aops host` | Read-only health, diagnostics and database status of the selected AOPS Server |
| `aops init` | Initialize repo-local AOPS metadata (.aops) |
| `aops mem` (aliases: memory) | Agentspace server-first memory commands (hosted memory-item ops are the source of truth; the local .aops memory tree is never written) |
| `aops plan` | Sprint-backed Projectman implementation plan commands |
| `aops playbook` | List reviewed playbooks projected from hosted Agentspace memory rules/constraints (server-first) |
| `aops pm` | Projectman authoring commands (hosted server-first) |
| `aops project` | Agentspace project sugar commands over the hosted AOPS gateway |
| `aops prompt` | Agentspace prompt and prompt-version sugar commands over the hosted AOPS gateway |
| `aops resource` | Agentspace resource sugar commands over the hosted AOPS gateway |
| `aops setup` | Connect this CLI to an AOPS Server and install the agent assets |
| `aops skill` | Agentspace skill and skill-version sugar commands over the hosted AOPS gateway |
| `aops start` | Compose the AOPS Collaborative Startup kickoff prompt from explicit flags |
| `aops sync` | Server-first sync commands: refresh the read-only local cache of Projectman/Agentspace state plus hosted prompt/skill mirrors |
| `aops target` | Manage named AOPS Server targets |
| `aops tui` | Open the AOPS terminal UI; it installs the bundled agent assets first |
| `aops version` | Show immutable CLI identity and optional selected target context |
| `aops view` | Read-only AOPS markdown/JSON presentation views |

#### 39.2.3 `aops activity` commands

| Command | Purpose |
| --- | --- |
| `aops activity get` | Get an activity ledger item by id |
| `aops activity list` | List activity ledger items |

#### 39.2.4 `aops agent` commands

| Command | Purpose |
| --- | --- |
| `aops agent invoke` | Invoke a tool via the canonical operator plane (/api/agent/tools/{toolId}/invoke) |
| `aops agent openapi` | Generate OpenAPI from the canonical federated tool catalog |
| `aops agent schema` | Print the live JSON Schema for one tool's input contract — use this before authoring --input payloads |
| `aops agent tools` | List federated tools from the canonical operator plane (/api/agent/tools) |

#### 39.2.5 `aops agent-profile` commands

| Command | Purpose |
| --- | --- |
| `aops agent-profile create` | Create a hosted Agentspace agent profile |
| `aops agent-profile delete` | Delete one hosted agent profile |
| `aops agent-profile get` | Get one hosted agent profile by id |
| `aops agent-profile list` | List hosted agent profiles within the current owner scope |
| `aops agent-profile update` | Patch one hosted agent profile |

#### 39.2.6 `aops agents-md` commands

| Command | Purpose |
| --- | --- |
| `aops agents-md preview` | Print merged AGENTS.md content without writing |
| `aops agents-md reset` | Remove the managed AOPS AGENTS.md prompt-template block |
| `aops agents-md update` | Insert or replace the managed AOPS AGENTS.md prompt-template block |

#### 39.2.7 `aops api` commands

| Command | Purpose |
| --- | --- |
| `aops api call` | Call direct domain route via dynamic host router (escape hatch, not primary tool plane) |

#### 39.2.8 `aops archive` commands

| Command | Purpose |
| --- | --- |
| `aops archive create` | Download a hosted PM graph into .aops/archive/<slug>/<timestamp> |
| `aops archive decommission-check` | Check whether an archive manifest permits full project/scope decommission |
| `aops archive delete` | Preview or apply destructive hosted PM graph deletion from a verified archive manifest |
| `aops archive verify` | Verify an archive bundle by re-fetching hosted PM graph data and comparing counts/checksums |

#### 39.2.9 `aops artifact` commands

| Command | Purpose |
| --- | --- |
| `aops artifact create` | Create a scope-owned artifact shell |
| `aops artifact delete` | Delete an artifact |
| `aops artifact get` | Get an artifact by id |
| `aops artifact link` | Link an artifact to a project-scoped ref |
| `aops artifact ref` | Artifact ref lookup commands |
| `aops artifact ref list` | List artifacts linked to a ref |

#### 39.2.10 `aops assets` commands

| Command | Purpose |
| --- | --- |
| `aops assets install` | Install the bundled or selected AOPS asset release |
| `aops assets rollback` | Restore the previous AOPS asset release and projected skills |
| `aops assets status` | Show locally installed AOPS asset release state; --check-latest also reads the latest published release |
| `aops assets update` | Install the latest assets release published at eeemzs/aops when it is newer (network), or a local release with --from-release |

#### 39.2.11 `aops auth` commands

| Command | Purpose |
| --- | --- |
| `aops auth login` | Sign in through South in a browser and keep the sign-in for this target |
| `aops auth logout` | Sign out at South, then remove the sign-in here |
| `aops auth refresh` | Renew the access token of the sign-in |
| `aops auth status` | Show the sign-in of the target and whether its Server accepts it |

#### 39.2.12 `aops chat` commands

| Command | Purpose |
| --- | --- |
| `aops chat binding` | ChatV3 loose channel/room reference bindings |
| `aops chat binding add` | Attach a loose external reference to the active room or channel |
| `aops chat binding list` | List loose references for the active room by default |
| `aops chat binding remove` | Remove a loose ChatV3 binding |
| `aops chat channel` | ChatV3 channel lifecycle helpers |
| `aops chat channel create` | Create a ChatV3 channel and print the invite once; invite contains secrets |
| `aops chat channel delete` | Hard-delete one ChatV3 channel after confirm-slug guard |
| `aops chat channel purge-before` | Preview or apply admin cleanup for channels created before an ISO cutoff |
| `aops chat channels` | List minimal channel and room labels; listing grants no content access |
| `aops chat join` | Run `aops chat join --help` for the current command contract. |
| `aops chat leave` | Run `aops chat leave --help` for the current command contract. |
| `aops chat listen` | Run `aops chat listen --help` for the current command contract. |
| `aops chat member` | ChatV3 channel member roster |
| `aops chat member list` | Run `aops chat member list --help` for the current command contract. |
| `aops chat member remove` | Remove a member from a ChatV3 channel (owner/operator), or with --room kick them from one room (room creator or owner/operator) |
| `aops chat member restore` | Restore a removed member (channel-level, owner/operator), or with --room re-add them to one room |
| `aops chat presence` | ChatV3 room presence |
| `aops chat presence list` | Run `aops chat presence list --help` for the current command contract. |
| `aops chat presence set` | Run `aops chat presence set --help` for the current command contract. |
| `aops chat read` | Run `aops chat read --help` for the current command contract. |
| `aops chat room` | ChatV3 room orientation helpers |
| `aops chat room brief` | Build a paste-ready room brief from guidance, bindings, members, presence, and cursor refs |
| `aops chat room code` | Time-limited room-only access codes (manager only) |
| `aops chat room code create` | Create a time-limited room code and display the secret once |
| `aops chat room code list` | List outstanding code metadata, never the secret code |
| `aops chat room code revoke` | Revoke one room code without removing existing members |
| `aops chat room create` | Create a room in the session channel (creator becomes its first participant) |
| `aops chat room join` | Join a room roster (disjoin later with "room leave"); rejected after a room-level removal |
| `aops chat room leave` | Leave a room roster (disjoin); the channel membership stays intact |
| `aops chat room list` | List active rooms of the session channel |
| `aops chat room members` | List the room-scoped participant roster (active participants by default) |
| `aops chat room summary` | Build a room summary pack with source messages for agent-composed memory digest |
| `aops chat send` | Run `aops chat send --help` for the current command contract. |
| `aops chat session` | Local ChatV3 agent session records |
| `aops chat session forget` | Run `aops chat session forget --help` for the current command contract. |
| `aops chat session get` | Run `aops chat session get --help` for the current command contract. |
| `aops chat session list` | Run `aops chat session list --help` for the current command contract. |
| `aops chat wake-watch` | EXPERIMENTAL foreground ChatV3-to-local-Codex wake watcher |

#### 39.2.13 `aops chatv3` commands

> Compatibility command tree for canonical `aops chat`. It is retained for existing automation; prefer `aops chat` for new workflows.

| Command | Purpose |
| --- | --- |
| `aops chatv3 binding` | ChatV3 loose channel/room reference bindings |
| `aops chatv3 binding add` | Attach a loose external reference to the active room or channel |
| `aops chatv3 binding list` | List loose references for the active room by default |
| `aops chatv3 binding remove` | Remove a loose ChatV3 binding |
| `aops chatv3 channel` | ChatV3 channel lifecycle helpers |
| `aops chatv3 channel create` | Create a ChatV3 channel and print the invite once; invite contains secrets |
| `aops chatv3 channel delete` | Hard-delete one ChatV3 channel after confirm-slug guard |
| `aops chatv3 channel purge-before` | Preview or apply admin cleanup for channels created before an ISO cutoff |
| `aops chatv3 channels` | List minimal channel and room labels; listing grants no content access |
| `aops chatv3 join` | Run `aops chatv3 join --help` for the current command contract. |
| `aops chatv3 leave` | Run `aops chatv3 leave --help` for the current command contract. |
| `aops chatv3 listen` | Run `aops chatv3 listen --help` for the current command contract. |
| `aops chatv3 member` | ChatV3 channel member roster |
| `aops chatv3 member list` | Run `aops chatv3 member list --help` for the current command contract. |
| `aops chatv3 member remove` | Remove a member from a ChatV3 channel (owner/operator), or with --room kick them from one room (room creator or owner/operator) |
| `aops chatv3 member restore` | Restore a removed member (channel-level, owner/operator), or with --room re-add them to one room |
| `aops chatv3 presence` | ChatV3 room presence |
| `aops chatv3 presence list` | Run `aops chatv3 presence list --help` for the current command contract. |
| `aops chatv3 presence set` | Run `aops chatv3 presence set --help` for the current command contract. |
| `aops chatv3 read` | Run `aops chatv3 read --help` for the current command contract. |
| `aops chatv3 room` | ChatV3 room orientation helpers |
| `aops chatv3 room brief` | Build a paste-ready room brief from guidance, bindings, members, presence, and cursor refs |
| `aops chatv3 room code` | Time-limited room-only access codes (manager only) |
| `aops chatv3 room code create` | Create a time-limited room code and display the secret once |
| `aops chatv3 room code list` | List outstanding code metadata, never the secret code |
| `aops chatv3 room code revoke` | Revoke one room code without removing existing members |
| `aops chatv3 room create` | Create a room in the session channel (creator becomes its first participant) |
| `aops chatv3 room join` | Join a room roster (disjoin later with "room leave"); rejected after a room-level removal |
| `aops chatv3 room leave` | Leave a room roster (disjoin); the channel membership stays intact |
| `aops chatv3 room list` | List active rooms of the session channel |
| `aops chatv3 room members` | List the room-scoped participant roster (active participants by default) |
| `aops chatv3 room summary` | Build a room summary pack with source messages for agent-composed memory digest |
| `aops chatv3 send` | Run `aops chatv3 send --help` for the current command contract. |
| `aops chatv3 session` | Local ChatV3 agent session records |
| `aops chatv3 session forget` | Run `aops chatv3 session forget --help` for the current command contract. |
| `aops chatv3 session get` | Run `aops chatv3 session get --help` for the current command contract. |
| `aops chatv3 session list` | Run `aops chatv3 session list --help` for the current command contract. |
| `aops chatv3 wake-watch` | EXPERIMENTAL foreground ChatV3-to-local-Codex wake watcher |

#### 39.2.14 `aops checkpoint` commands

| Command | Purpose |
| --- | --- |
| `aops checkpoint create` | Write a structured short session checkpoint |
| `aops checkpoint get` | Get a checkpoint by id, or the current checkpoint for a subject |
| `aops checkpoint list` | List checkpoint timeline, including superseded records |

#### 39.2.15 `aops codex` commands

| Command | Purpose |
| --- | --- |
| `aops codex register` | Register a verified local Codex task for one ChatV3 agent member |
| `aops codex registrations` | List local ChatV3-to-Codex registrations with session ids redacted |
| `aops codex unregister` | Remove one local ChatV3 member wake registration |
| `aops codex wake` (aliases: wakeup) | Wake the exact local Codex Desktop session through internal IPC |

#### 39.2.16 `aops discuss` commands

| Command | Purpose |
| --- | --- |
| `aops discuss abandon` | Mark a hosted discussion topic as abandoned |
| `aops discuss conclude` | Conclude a hosted discussion topic (server enforces min-turns + final-stance readiness) |
| `aops discuss digest` | Build a non-interpretive hosted discussion context pack |
| `aops discuss follow-up` | Create a hosted child topic referencing an existing discussion (parent not mutated) |
| `aops discuss fork` | Create a hosted fork topic branching from an existing discussion (parent not mutated) |
| `aops discuss get` | Get a hosted discussion topic with its turns and outputs |
| `aops discuss lineage` | Show the child-topic lineage tree for one discussion root (built from hosted list calls) |
| `aops discuss list` | List hosted discussion topics |
| `aops discuss loop-prompt` | Print a persistent agent loop prompt for hosted discussion automation |
| `aops discuss output` | Set a hosted discussion output (conclude-flow output authoring) |
| `aops discuss start` | Create a hosted discussion topic |
| `aops discuss status` | Read hosted discussion state mapped to lifecycleState/nextTurn/openQuestions |
| `aops discuss turn` | Append one turn to a hosted discussion topic (turn-order enforced server-side) |
| `aops discuss wait` | Poll hosted status until an agent may write, an operator block appears, or the topic is ready/done |

#### 39.2.17 `aops doc` commands

| Command | Purpose |
| --- | --- |
| `aops doc answer` | Read a citation-first deterministic answer pack for a saved document version |
| `aops doc create` | Create a Docman document through the canonical flow surface |
| `aops doc delete` | Safely delete one Docman document graph after exact title confirmation |
| `aops doc get` | Get a Docman document by id through the hosted gateway |
| `aops doc group` | Docman document-group commands |
| `aops doc group create` | Create a document group |
| `aops doc group delete` | Delete a document group |
| `aops doc group get` | Get a document group by id |
| `aops doc group list` | List document groups |
| `aops doc group update` | Update a document group |
| `aops doc import` | Import structured Docman content from source files |
| `aops doc index` | Docman retrieval index commands |
| `aops doc index build` | Build or refresh the persisted retrieval index for a saved document version |
| `aops doc link` | Docman structure linking commands |
| `aops doc link page` | Link or unlink pages in a section |
| `aops doc link section` | Link or unlink sections in a document outline |
| `aops doc link section delete` | Alias for doc section unlink |
| `aops doc list` | List Docman documents through the hosted gateway |
| `aops doc mirror` | Docman v2 structured and v3 source-file mirror helpers |
| `aops doc mirror pull` | Materialize structured v2 .md mirrors and exact v3 source files with adjacent sidecars |
| `aops doc mirror push` | Import root markdown files or guarded-push one v3 source file with its adjacent sidecar |
| `aops doc order` | Docman outline ordering commands |
| `aops doc order pages` | Update section-page-link ordering through the canonical flow surface |
| `aops doc order sections` | Update document-section-link ordering through the canonical flow surface |
| `aops doc outline` | Docman normalized outline inspection |
| `aops doc outline get` | Read a section-centric normalized outline for a saved document version |
| `aops doc page` | Docman page and page-version commands |
| `aops doc page copy` | Copy a page into a section through the canonical flow surface |
| `aops doc page create` | Create a page with its first draft and optionally link it into a document tree |
| `aops doc page draft-save` | Create or update a page-version draft through the canonical flow surface |
| `aops doc page get` | Get a page by id |
| `aops doc page list` | List pages |
| `aops doc page membership-unlink` | Delete a reusable section-page-link without deleting the page |
| `aops doc page move` | Move a section-page-link to another section |
| `aops doc page unlink` | Unlink a page node created in the document outline without deleting the page |
| `aops doc page update` | Update Docman page metadata through the canonical flow surface |
| `aops doc page-version` | Docman page-version commands |
| `aops doc page-version get` | Get a page version by id |
| `aops doc page-version list` | List page versions |
| `aops doc page-version update` | Update page-version metadata; content changes must use doc page draft-save |
| `aops doc publish` | Materialize saved document content to markdown, html, or exact source |
| `aops doc scope` | Docman scope-owned retrieval commands |
| `aops doc scope answer` | Read a citation-first answer pack across latest document versions in one scope |
| `aops doc scope search` | Search persisted Docman retrieval rows across latest document versions in one scope |
| `aops doc search` | Search persisted Docman retrieval rows for a saved document version |
| `aops doc section` | Docman section commands |
| `aops doc section copy` | Copy a section into a document version through the canonical flow surface |
| `aops doc section create` | Create a section and optionally link it into a document outline |
| `aops doc section get` | Get a section by id |
| `aops doc section list` | List sections |
| `aops doc section unlink` | Unlink a section from a document outline without deleting the section |
| `aops doc section update` | Update a Docman section through the canonical flow surface |
| `aops doc set-current-version` | Flip a document’s canonical current version atomically (peer-clear + optional publish + publishedAt). Dispatches docman.document-version.set-current. |
| `aops doc source` | Fetch exact composed source for a saved document fragment |
| `aops doc source-file` | Immutable Docman text-file snapshot commands |
| `aops doc source-file save` | Save an exact UTF-8 source file as a new immutable text-file document version |
| `aops doc summary` | Docman persisted summary commands |
| `aops doc summary build` | Build or refresh persisted summaries for a saved document version; index rows are ensured first |
| `aops doc update` | Update a Docman document through the canonical flow surface |
| `aops doc version` | Docman document-version commands |
| `aops doc version create` | Create a document version through the canonical flow surface |
| `aops doc version get` | Get a document version by id |
| `aops doc version list` | List document versions |
| `aops doc version update` | Update document-version header metadata (status / title / summary / release-notes / label). To switch the canonical current version, use `aops-cli doc set-current-version` instead. |

#### 39.2.18 `aops docs` commands

| Command | Purpose |
| --- | --- |
| `aops docs user-guide` | Generate or refresh the dynamic appendix sections in the AOPS CLI user guide |

#### 39.2.20 `aops exp` commands

| Command | Purpose |
| --- | --- |
| `aops exp capture` | Capture a hosted experience item |
| `aops exp delete` | Delete one hosted experience item |
| `aops exp get` | Get one hosted experience item by id |
| `aops exp list` | List hosted experience items within the current owner scope |
| `aops exp promote` | Promote a hosted experience item into a durable memory item (or a playbook with --as-playbook) |
| `aops exp search` | Lexically rank hosted experience items (server-truth rows ranked client-side) |
| `aops exp update` | Patch one hosted experience item |

#### 39.2.21 `aops host` commands

| Command | Purpose |
| --- | --- |
| `aops host database` (aliases: db) | Inspect the selected Server database status |
| `aops host database status` | Load hosted database status (/api/aops/settings/database/status) |
| `aops host diagnostics` (aliases: plugins) | Inspect host runtime diagnostics (/api/host-admin/plugins) |
| `aops host health` | Smoke the runtime health endpoint (GET /api/health) |
| `aops host hello` | Smoke the hello endpoint (GET /api/hello) |

#### 39.2.23 `aops mem` commands

| Command | Purpose |
| --- | --- |
| `aops mem brief` | Build a read-only bootstrap brief from synopsis, resume, sticky guidance, and refs |
| `aops mem checkpoint` | Write a short rolling checkpoint; defaults to project resume memory |
| `aops mem compact` | Build a reviewable compact pack from old short memory and optionally write a summary |
| `aops mem delete` | Advanced: delete a memory item by id |
| `aops mem doc` | Advanced: read Docman through recommended memory refs |
| `aops mem doc answer` | Run Docman answer-pack against a recommended memory ref |
| `aops mem doc publish` | Materialize Docman output from a recommended memory ref |
| `aops mem doc refs` | List Docman-capable recommended refs from a curated resume pack |
| `aops mem doc source` | Fetch exact Docman source from a recommended memory ref |
| `aops mem get` | Get a memory item by id |
| `aops mem list` | List memory items within the current owner scope |
| `aops mem prune` | Prune old hosted short memory items with a dry-run first |
| `aops mem resume` | Recommended: build a curated resume pack from hosted memory |
| `aops mem search` | Recommended: search memory with subject-aware retrieval ranking |
| `aops mem summary` | Write a short session summary; durable closeout requires explicit --closeout --durability durable --confirm |
| `aops mem synopsis` | Build a deterministic project/subject synopsis from hosted memory |
| `aops mem update` | Advanced: update an existing memory item |
| `aops mem write` | Advanced: write an Agentspace memory item with standardized metadata |

#### 39.2.24 `aops plan` commands

| Command | Purpose |
| --- | --- |
| `aops plan create` | Create a sprint-backed implementation plan, optionally bound to a kanban task |
| `aops plan get` | Get a sprint-backed implementation plan by selector |
| `aops plan list` | List sprint-backed implementation plans |
| `aops plan update` | Patch a sprint-backed implementation plan, including phases and microtasks |

#### 39.2.25 `aops playbook` commands

| Command | Purpose |
| --- | --- |
| `aops playbook get` | Get one hosted playbook projection |
| `aops playbook list` | List hosted playbook projections |
| `aops playbook project-set` | Load accepted sticky/durable project-scope playbooks for start/resume guidance |
| `aops playbook promote` | Promote reviewed experience into a playbook memory rule/constraint |
| `aops playbook show` | Render one hosted playbook as agent-readable markdown |

#### 39.2.26 `aops pm` commands

| Command | Purpose |
| --- | --- |
| `aops pm board` | Projectman kanban board commands |
| `aops pm board archive` | Soft-archive a Projectman board by setting archivedAt on the board record |
| `aops pm board bootstrap` | Board-slug bootstrap registry commands |
| `aops pm board bootstrap get` | Read the canonical board bootstrap registry for a board slug/id/name |
| `aops pm board bootstrap set` | Create or update the board bootstrap registry |
| `aops pm board closeout` | Operator-approved only: atomically close the active board window, write closeout memory, move task to Done, and clear bootstrap active refs |
| `aops pm board create` | Create a Projectman board bootstrap |
| `aops pm board delete` | Delete a Projectman board |
| `aops pm board get` | Get a Projectman board by id, exact name, or slug |
| `aops pm board kickoff` | Create or reuse the active task+sprint window for a board and refresh its bootstrap registry |
| `aops pm board list` | List Projectman boards |
| `aops pm board resume` | Resolve board bootstrap, active window, and resume context for a board slug/id/name |
| `aops pm board unarchive` | Restore a soft-archived Projectman board by clearing archivedAt |
| `aops pm column` | Projectman board column commands |
| `aops pm column add` | Create a column and place it on a board in one step |
| `aops pm column list` | List a board's columns in board order |
| `aops pm column remove` | Remove a column from a board; refused while it still holds tasks |
| `aops pm column reorder` | Set the board column order; name every column exactly once |
| `aops pm feedback` | Projectman feedback commands |
| `aops pm feedback create` | Create a Projectman feedback item |
| `aops pm feedback delete` | Delete a Projectman feedback item |
| `aops pm feedback get` | Get a Projectman feedback item by id |
| `aops pm feedback list` | List Projectman feedback items |
| `aops pm feedback update` | Patch a Projectman feedback item |
| `aops pm handoff` | Projectman-aware memory resume and write commands |
| `aops pm handoff resume` | Build a subject-aware resume pack for a Projectman entity |
| `aops pm handoff write` | Write subject-aware Projectman-linked memory for kickoff/resume/decision/blocker/closeout/rule |
| `aops pm issue` | Projectman issue commands |
| `aops pm issue create` | Create a Projectman issue |
| `aops pm issue delete` | Delete a Projectman issue |
| `aops pm issue get` | Get a Projectman issue by id |
| `aops pm issue list` | List Projectman issues |
| `aops pm issue update` | Patch a Projectman issue |
| `aops pm ktask` (aliases: task) | Projectman kanban task commands |
| `aops pm ktask create` | Create a Projectman kanban task |
| `aops pm ktask delete` | Delete a Projectman kanban task |
| `aops pm ktask get` | Get a Projectman kanban task by selector |
| `aops pm ktask list` | List Projectman kanban tasks |
| `aops pm ktask set-status` | Move a kanban task to the workflow column matching the requested status |
| `aops pm resume` | Composed paste-ready resume brief: active sprint windows, review queue for an agent, open issues/feedback, and durable memory |
| `aops pm review-request` (aliases: rr) | Projectman review request commands; canonical RR/RRR owner |
| `aops pm review-request create` | Create a Projectman review request; --parent opens a child re-review gate |
| `aops pm review-request delete` | Delete a Projectman review request |
| `aops pm review-request get` | Get a Projectman review request by id |
| `aops pm review-request list` | List Projectman review requests |
| `aops pm review-request result` (aliases: add-result) | Append an immutable review result (RRR) to a Projectman review request |
| `aops pm review-request update` | Patch Projectman review-request metadata |
| `aops pm sprint` | Projectman sprint commands |
| `aops pm sprint archive` | Soft-archive a Projectman sprint by setting archivedAt on the sprint record |
| `aops pm sprint create` | Create a Projectman sprint execution document, optionally bound to a kanban task |
| `aops pm sprint delete` | Delete a Projectman sprint execution document |
| `aops pm sprint get` | Get a Projectman sprint execution document by selector |
| `aops pm sprint list` | List Projectman sprint execution documents |
| `aops pm sprint set-status` | Set all microtask statuses in a sprint to the target status, deriving the sprint-level status |
| `aops pm sprint unarchive` | Restore a soft-archived Projectman sprint by clearing archivedAt |
| `aops pm sprint update-plan` | Patch a sprint plan, including phases and microtasks |
| `aops pm status` | Read-only Projectman status audit commands |
| `aops pm status audit` | Read-only audit for kanban task and sprint completion drift |
| `aops pm status reconcile` | Preview or explicitly apply one guarded PM status reconciliation |
| `aops pm utask` | Projectman sprint-bound microtask commands |
| `aops pm utask create` | Create a sprint-bound utask |
| `aops pm utask delete` | Delete a sprint-bound utask |
| `aops pm utask set-status` | Update only the lifecycle status of a sprint-bound utask |
| `aops pm utask update` | Patch a sprint-bound utask without replacing the full sprint plan |

#### 39.2.27 `aops project` commands

| Command | Purpose |
| --- | --- |
| `aops project create` | Create a project; you become its first admin |
| `aops project delete` | Delete a project |
| `aops project get` | Get a project by id |
| `aops project link` | Link a hosted project slug into the repo project registry |
| `aops project links` | List repo project registry links |
| `aops project links list` | List project links from .aops/aops.config.json |
| `aops project list` | List hosted projects |
| `aops project member` | Project membership: list, add, set-role, remove and inventory |
| `aops project member add` | Add a member to a project and print the server readback |
| `aops project member inventory` | Read-only: every listable project with its member and admin counts |
| `aops project member list` | List the members of a project |
| `aops project member remove` | Remove a project member and print the server readback |
| `aops project member set-role` | Change the role of a project member and print the server readback |
| `aops project migrate-local-root` | Plan or apply flat .aops projectman/agentspace migration into .aops-cache/projects/<slug> |
| `aops project update` | Update a project |

#### 39.2.28 `aops prompt` commands

| Command | Purpose |
| --- | --- |
| `aops prompt create` | Create a scope-owned prompt shell |
| `aops prompt current` | Resolve the current prompt version for a prompt shell |
| `aops prompt delete` | Delete a prompt shell |
| `aops prompt get` | Get a prompt shell by id |
| `aops prompt inspect` | Load the prompt shell, versions, and current version together |
| `aops prompt list` | List reusable prompt shells |
| `aops prompt update` | Update a prompt shell |
| `aops prompt version` | Prompt-version authoring and lifecycle commands |
| `aops prompt version create` | Create a new prompt version |
| `aops prompt version delete` | Delete a prompt version |
| `aops prompt version get` | Get a prompt version by id |
| `aops prompt version list` | List prompt versions |
| `aops prompt version publish` | Publish a prompt version and sync the prompt current version |
| `aops prompt version update` | Update an existing prompt version |

#### 39.2.29 `aops resource` commands

| Command | Purpose |
| --- | --- |
| `aops resource create` | Create a scope-owned resource |
| `aops resource delete` | Delete a resource |
| `aops resource get` | Get a resource by id |
| `aops resource list` | List reusable resource shells |
| `aops resource update` | Update a resource |

#### 39.2.31 `aops setup` commands

| Command | Purpose |
| --- | --- |
| `aops setup ai` | Print a safe, copy-ready AOPS installation prompt for a terminal AI agent |
| `aops setup guide` | Print the packaged agent-readable AOPS installation skill |
| `aops setup init` | Connect this CLI to an existing AOPS Server (path 4) and install the agent assets |

#### 39.2.32 `aops skill` commands

| Command | Purpose |
| --- | --- |
| `aops skill ask` | Recommend hosted skills using one deterministic metadata search (no LLM or body loading) |
| `aops skill create` | Create a scope-owned skill shell |
| `aops skill current` | Resolve the current skill version for a skill shell |
| `aops skill delete` | Delete a skill shell |
| `aops skill get` | Get a skill shell by id |
| `aops skill inspect` | Load the skill shell, versions, and current version together |
| `aops skill list` | List reusable skill shells |
| `aops skill search` | Find current published hosted skills by bounded deterministic metadata ranking |
| `aops skill update` | Update a skill shell |
| `aops skill version` | Skill-version authoring and lifecycle commands |
| `aops skill version create` | Create a new skill version |
| `aops skill version delete` | Delete a skill version |
| `aops skill version get` | Get a skill version by id |
| `aops skill version list` | List skill versions |
| `aops skill version publish` | Publish a skill version and sync the skill current version |
| `aops skill version update` | Update an existing skill version |

#### 39.2.33 `aops sync` commands

| Command | Purpose |
| --- | --- |
| `aops sync bootstrap` | Seed the read-only local cache from hosted server state and refresh read-only prompt/skill mirrors |
| `aops sync diff` | Show local records that are not synced |
| `aops sync pull` | Pull hosted Projectman/Agentspace state into the read-only local cache and refresh read-only prompt/skill mirrors |
| `aops sync rebuild-views` | Rebuild derived local cache markdown views and hosted read-only indexes |
| `aops sync resolve` | Resolve a local cache conflict by adopting the hosted/cache version (--prefer remote) |
| `aops sync sidecar` | Run a localhost-bound cockpit sidecar that exposes read-only local cache status/diff on the client machine |
| `aops sync status` | Show local cache sync state |

#### 39.2.34 `aops target` commands

| Command | Purpose |
| --- | --- |
| `aops target add` | Run `aops target add --help` for the current command contract. |
| `aops target doctor` | Run `aops target doctor --help` for the current command contract. |
| `aops target list` | Run `aops target list --help` for the current command contract. |
| `aops target remove` | Run `aops target remove --help` for the current command contract. |
| `aops target show` | Run `aops target show --help` for the current command contract. |
| `aops target use` | Run `aops target use --help` for the current command contract. |

#### 39.2.36 `aops view` commands

| Command | Purpose |
| --- | --- |
| `aops view board` | Show a Projectman board with tasks and sprints |
| `aops view boards` | List Projectman boards from .aops-cache/projectman |
| `aops view dashboard` | Show repo-first dashboard across local AOPS workspaces |
| `aops view digest` | Compose a bounded repo-first context pack |
| `aops view discussion` | Show a discussion topic with turns and outputs |
| `aops view discussions` | List local discussion topics |
| `aops view doc` | Show a Docman mirror document |
| `aops view doc-page` | Show a heading slice from Docman mirror markdown |
| `aops view docs` | List read-only Docman document mirrors |
| `aops view experience` | List local Agentspace experience items |
| `aops view feedback` | List Projectman feedback |
| `aops view hosted-inventory` | Show hosted project inventory grouped by docs, skills, prompts, and resources |
| `aops view hosted-projects` | List hosted Agentspace projects |
| `aops view issues` | List Projectman issues |
| `aops view memory` | List local Agentspace memory items |
| `aops view project` | Show current or selected repo-bound project |
| `aops view projects` | List repo-bound projects and local counts |
| `aops view prompts` | List read-only hosted prompt mirrors |
| `aops view resume` | Show local resume/kickoff/closeout memory items |
| `aops view review-requests` (aliases: review-request) | List Projectman review requests |
| `aops view skills` | List read-only hosted skill mirrors |
| `aops view sprint` | Show a Projectman sprint with plan and relations |
| `aops view sprints` | List Projectman sprints |
| `aops view task` | Show a Projectman kanban task with relations |
| `aops view tasks` | List Projectman kanban tasks |

<!-- aops-generated:cli-command-catalog:end -->

<!-- aops-generated:capability-discovery:start -->

### 39.3 Generated capability discovery guide

#### 39.3.1 Overview

> AOPS does not copy a fixed hosted-tool list into this guide. The server catalog is discovered live, so newly registered domain capabilities appear without rewriting prose.

#### 39.3.2 Source-of-truth model

1. **DCM (Domain Capability Manifest)** is the canonical domain capability source.
2. Host routes, the agent catalog, OpenAPI, JSON Schema, and domain CLI projections are derived views of that capability source.
3. **HRM (Host Registration Manifest)** tells the host how a domain is registered; it is not a second capability source.
4. AOPS sugar commands come from the public Commander program and are cataloged separately above.

#### 39.3.3 Live discovery commands

| Command | Purpose |
| --- | --- |
| `aops agent tools` | List federated tools from the canonical operator plane (/api/agent/tools) |
| `aops agent schema` | Print the live JSON Schema for one tool's input contract — use this before authoring --input payloads |
| `aops agent openapi` | Generate OpenAPI from the canonical federated tool catalog |
| `aops agent invoke` | Invoke a tool via the canonical operator plane (/api/agent/tools/{toolId}/invoke) |

Use the smallest useful read:

```bash
aops agent tools --domain <domain> --summary --json
aops agent schema --tool <tool-id> --summary --json
aops agent openapi --domain <domain> --out ./openapi.json
```

<!-- aops-generated:capability-discovery:end -->
