---
schemaVersion: 1
entityType: "docman.document-mirror"
readOnly: true
source: "docman"
projectId: "b4e7db65-7956-4c92-8cc1-0f16ef908d41"
projectSlug: "aops"
scopeId: "b4e7db65-7956-4c92-8cc1-0f16ef908d41"
groupId: "eaea8cf5-e39e-4686-9167-73ff4fd0264b"
groupUid: "aops-guides"
documentId: "a20b7f5c-c8bc-4afa-ba34-32135e5c73b9"
documentVersionId: "9c1119ee-ad5d-4e8c-b5df-62baef7e7437"
documentVersion: 21
documentSlug: "aops-cli-user-guide"
title: "AOPS CLI User Guide"
target: "markdown"
pulledAt: "2026-08-23T17:00:59.727Z"
---
<!-- READ-ONLY MIRROR: update Docman/source docs, then run aops-cli doc mirror pull. -->

# AOPS CLI User Guide

_Release Notes:_ Translates the full composable guide section by section, preserves all commands and guards, and corrects hosted skill/prompt ownership to Agentspace.

## 1 Introduction

### 1.1 Overview

#### 1.1.1 Overview

AOPS CLI is the shared command-line entry point for managing a local or
self-hosted AOPS server. It resolves project context, provides guarded sugar
commands, and falls back to the server's live agent-tool catalog when needed.
The AOPS server is canonical for durable Projectman, Agentspace, and Docman
records; `.aops/**` content in a repository is a refreshable, read-only working
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
discovery for less common operations. The `.aops/**` tree in a repository is a
reproducible, read-only working cache; editing its Markdown files does not
update server state.

## 3 Shortest setup flow

### 3.1 Trusted-local

#### 3.1.1 Overview

```bash
aops-cli init
aops-cli setup server-env --auth-provider trusted-local
pnpm --filter @aops/aops-server run dev:prepare
pnpm --filter @aops/aops-server run dev
aops-cli host health
```

### 3.2 Interactive auth

#### 3.2.1 Overview

```bash
aops-cli init
aops-cli setup server-env --auth-provider interactive auth
pnpm --filter @aops/aops-server run dev:prepare
pnpm --filter @aops/aops-server run dev
aops-cli setup first-admin
aops-cli auth login
```

## 4 `init`

### 4.1 Overview

#### 4.1.1 Overview

`aops init` creates `.aops/aops.config.json` and the required local metadata
skeleton in the current repository. It does not create hosted
Projectman/Docman records. Read `aops init --help` for current options and write
guards before applying it.

## 5 `setup server-env`

### 5.1 Overview

#### 5.1.1 Overview

`aops setup server-env` creates or validates the dedicated PostgreSQL and
authentication environment for the npm-based AOPS Server. Never copy secret
values into command output or documentation. When connection verification is
needed, use the test and TLS options documented by live `--help`.

## 6 `setup first-admin`

### 6.1 Overview

#### 6.1.1 Overview

This heading is a compatibility note retained from the old interactive-auth
flow. The current CLI has no `aops setup first-admin` command. Trusted-local
setup does not require a first-admin step. For remote interactive targets, use
`aops auth login` and the live AuthV2 tool contract.

## 7 User management

### 7.1 Overview

#### 7.1.1 Overview

The current public CLI has no generic `aops user` sugar family. Do not guess a
command name for user or identity operations. First run `aops agent tools
--domain authv2 --summary --json`, then inspect the selected tool with
`aops agent schema`. Deletions, role changes, and access changes also require
explicit operator approval.

## 8 Login and tokens

### 8.1 Overview

#### 8.1.1 Overview

Use `aops auth login|status|logout` for named remote-session targets. Tokens are
target-bound and stored encrypted; never write them to JSON output, an issue,
or documentation. Do not create an unnecessary login flow for a trusted-local
target.

## 9 Diagnostics

### 9.1 Overview

#### 9.1.1 Overview

Start with the non-mutating `aops doctor`, `aops target doctor <name>`, and
`aops host health` surfaces. Do not reset, reinstall, or perform database work
before identifying a fault. Verify the active target, port, and data root
first.

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

## 12 Tasker and Runner routing

### 12.1 Overview

#### 12.1.1 Overview

The current CLI has no generic `aops tasker` or `aops runner` sugar family. Use
`aops pm` for human-readable project planning and live `aops agent tools
--domain tasker|runner` discovery for Tasker/Runner domain capabilities. Do not
guess tool or payload names from this guide.

## 13 `.aops` local cache & sync

### 13.1 Overview

#### 13.1.1 Project registry, `authoringMode`, and `localRoot`

Multi-project repos use `.aops/aops.config.json` as a project registry. The
hosted server is the source of truth for project identity and for all
Projectman/Agentspace records; the repo registry only records which local
directory mirrors a hosted project as a read-only cache:

```bash
aops-cli project link --slug aops --mode local --local-root .aops/projects/aops --apply --json
aops-cli project link --slug demo --mode hosted-only --apply --json
aops-cli project links list --json
aops-cli project migrate-local-root --project-slug aops --local-root .aops/projects/aops --dry-run --json
aops-cli project migrate-local-root --project-slug aops --local-root .aops/projects/aops --apply --confirm --json
```

Contract:

1. `project link/links` manages only the repo project registry after verifying
   the hosted project exists and is not archived/deleted.
2. `authoringMode: local` means a local cache directory is materialized under
   `localRoot` (normally `.aops/projects/<slug>`). Create/write/read still go to
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
aops-cli sync status --project-slug aops --json
aops-cli sync pull --project-slug aops --apply --json
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

#### 13.1.3 Archive lifecycle

`aops-cli archive` prepares hosted Projectman graph cleanup from a local bundle.
It is deliberately a CLI composition over existing hosted Projectman surfaces,
not a new hosted archive domain.

```bash
aops-cli archive create --project-slug aops --apply --json
aops-cli archive verify --manifest .aops/archive/aops/<ts>/manifest.json --apply --json
aops-cli archive delete --manifest .aops/archive/aops/<ts>/manifest.json --json
aops-cli archive delete --manifest .aops/archive/aops/<ts>/manifest.json --apply --confirm --json
aops-cli archive decommission-check --manifest .aops/archive/aops/<ts>/manifest.json --json
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
read `.aops/**` files from the read-only local cache. Canonical truth remains on
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
aops-cli view task 0ea46e18-d717-454d-8244-90ad388c4a80

# 8+ character ID prefix (the UUID's first 8 characters or the -<8>.md filename suffix)
aops-cli view task 0ea46e18

# slug
aops-cli view board ops

# exact title/name
aops-cli view sprint "AOPS CLI view follow-up"

# doc-page composite selector (document#heading)
aops-cli view doc-page tooling-cli-host-plugin-system#runtime-config
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

1. `view dashboard --style agent` can scan `.aops/agentspace/session-state/**`
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
   `.aops/**/*.md` files from the read-only local cache. They do not call hosted
   tools, sync, write caches, mutate state, or write to `~/.aops`.
2. Hosted view commands (`hosted-projects`, `hosted-inventory`) call only these
   hosted read/list tools: `agentspace.project.list-projects`,
   `docman.document.list`, `agentspace.skill.list-skills`,
   `agentspace.prompt.list-prompts`, `agentspace.resource.list-resources`.
   They do not mutate, sync, refresh mirrors, or write caches.
3. Cross-domain joins follow existing frontmatter/API fields (`subjectType`,
   `subjectId`, `boardId`, `sprintLocalId`, `pmContext.taskId`, and so on);
   they do not invent new domain semantics.
4. `view skills` and `view prompts` read `.aops/hosted/**` mirror files.
   Canonical truth remains in hosted Agentspace/server state, and view does not
   refresh it.
5. `view docs` and `view doc-page` read the `.aops/docman/**` mirror. The
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
  server/DB. `sync pull` only creates read-only mirrors under `.aops/hosted/**`.
- A repository may intentionally pull another project's hosted prompt/skill
  mirror with `--hosted-project-id|name|slug`.
- `sync pull` is a project-level server-to-cache refresh. Use
  `--hosted-project-id|name|slug` to refresh hosted prompt/skill mirrors.

Startup flow:

1. Run `aops-cli init` for a new repository.
2. Refresh the read-only hosted-state cache with `aops-cli sync pull --project-slug aops --apply --json`.
3. Read `.aops/projectman/views/index.md` and `.aops/agentspace/memory/index.md` for cached context.
4. When reusable prompt/skill context is needed, read `.aops/hosted/index.md`, `.aops/hosted/skills/index.md`, and `.aops/hosted/prompts/index.md`.
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

## 14 Runner sugar

### 14.1 Overview

#### 14.1.1 Overview

Runner work uses hosted, schema-discovered execution surfaces; do not assume a
top-level `aops runner` command exists. Find available tools with
`aops agent tools --domain runner --summary --json`, read the schema, and obtain
separate explicit approval for flows with effects.

## 15 Prompt sugar

### 15.1 Overview

#### 15.1.1 Overview

`aops prompt` manages reusable prompt shells and versions server-first. Use
`list|get|inspect|current` for reads, `create|update|version create` for writes,
and `version publish` for publication. `.aops/hosted/prompts/**` is only a
read-only mirror.

## 16 Project sugar

### 16.1 Overview

#### 16.1.1 Overview

`aops project` separates the hosted project record from the repo-local project
registry link. `list|get` reads hosted records; `link` attaches only a verified
slug to the local registry. For local-root relocation, first run
`migrate-local-root --dry-run`, then use `--apply --confirm` only for the
reviewed target.

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

`discuss`, `chat`, and `pm review-request` are distinct coordination surfaces:
use `discuss` for decisions/consensus, hosted chat rooms (`chat`) for
coordination/wake signals, and Projectman (`pm review-request`) for reviews. A
mismatched writer and listener is the most common cause of silently lost
traffic; the coordination message and listener must use the same hosted chat
room. See the `aops-cli-chat` skill.

| Need                                                   | Command                        | Skill            |
|--------------------------------------------------------|--------------------------------|------------------|
| Structured decision transcript and final decisions    | `aops-cli discuss start`        | `aops-cli-discuss` |
| Decision loop for agent order and lifecycle           | `aops-cli discuss wait`, `aops-cli discuss turn`, `aops-cli discuss conclude` | `aops-cli-discuss` |
| Multi-agent coordination/wake room                    | `aops-cli chat room create`, `aops-cli chat message send` | `aops-cli-chat` |
| Discover an agent's pending room/message work         | `aops-cli chat inbox --for <agent>` | `aops-cli-chat` |
| Listen to room traffic or read unread messages        | `aops-cli chat listen`, `aops-cli chat catchup` | `aops-cli-chat` |
| Request a review, record a result, or re-review       | `aops-cli pm review-request create`, `aops-cli pm review-request result` | `aops-cli-projectman` |

Coordination (wake signals, room messages, and listeners) lives in hosted chat
rooms; the decision ritual lives in `discuss`. Write a structured decision or
stance to the transcript with `discuss turn`/`conclude`. To wake the other
agent, announce that work with a short `chat message send` in the linked hosted
chat room. The room message is a wake signal; the Discuss transcript is the
canonical record. Clarify invitation and listener expectations in the chat
room. For details, see `aops-cli-chat` (room lifecycle, members, and
`chat send/listen/catchup`) and `aops-cli-discuss` (decision ritual and
`discuss wait` exit codes). Review flow (RR/RRR, re-review, and material issues)
belongs to `aops-cli-projectman`.

Slug-first operator contract:

1. `discuss start --slug <slug>` writes the canonical slug to topic frontmatter.
   If omitted, it is derived from the title and returned as `topicSlug` in
   JSON. Hosted chat rooms also carry a slug through
   `chat room create --slug <slug>`.
2. Selectors now treat the exact slug as the operator-facing default. Legacy
   folder names and short ids are debug/legacy fallbacks. Use slugs in
   operator-facing commands, handoffs, and chat pings; keep raw UUIDs out of
   normal traffic except for debugging or JSON.
3. If a legacy folder-name or implicit short-id match wins, the JSON envelope
   returns `cliDeprecationWarnings`. Except during debugging, treat these as a
   signal to migrate to slugs.
4. Use `--short-id <8char>` when an explicit debug selector is necessary. This
   explicit route emits no warning; an implicit short id emits
   `cliDeprecationWarnings`. If slug/folder/short-id resolution is ambiguous and
   only one active record exists, `--prefer-active` selects it. Durable
   handoffs should still store the slug. For Discuss commands, `--short-id` is
   the topic selector.
5. Bootstrap a new agent by running `aops-cli chat inbox --for <agent> --json`,
   then read unread messages in the linked room with
   `chat catchup --for <agent> --apply --json`. Use
   `chat listen --for <agent> --max-loops 1 --json` for an active wake and
   `aops-cli pm review-request list --json` for pending reviews.
6. `discuss conclude` must not leave `_TBD_` placeholders in `consensus.md`, the
   agent final stance, `disagreement.md`, or `open-questions.md`. The agent that
   started the topic owns the output and reviews or enriches these files before
   finalize/closeout.

### 17.3 PM window, chat-room binding, and active window

#### 17.3.1 Overview

Use one Projectman window for two-agent execution. If a board kickoff already
exists, reuse that board's active task/sprint by default; create a new task and
sprint only when no active window exists. The repo-first `collab pm-bind`
command is retired. Manage the PM window directly with `pm`, and coordinate in
a hosted chat room bound to the board.

```bash
aops-cli pm board kickoff --board ops --title "AOPS PM tooling triage" --goal "..." --apply --json
# Bind the coordination room to the board (wake/flow stays in hosted chat)
aops-cli chat room create --slug ops-room --title "Ops" --created-by <agent> --apply --json
aops-cli chat binding add --room-id <room-id> --binding-type projectman.board --binding-id <board-id> --label "Active board" --created-by <agent> --apply --json
```

When explicit task/sprint selection is required, use the relevant
`pm sprint`/`pm utask` commands. If the operator intentionally opens a new
window, create a new `pm board kickoff`/sprint. Otherwise, prefer active board
references instead of creating a duplicate kickoff window.

### 17.4 Two-agent research and deliberation flow

#### 17.4.1 Overview

When the operator asks agents to "discuss," "talk to Claude and propose a
plan," or "research together," the primary agent should not write a plan alone
and request review afterward. By default, both parties produce independent
context and then converge through a Discuss topic for decisions plus a hosted
chat room for wake signals. Reviews belong in Projectman.

Primary-agent flow:

1. Open a decision topic carrying the objective and agents with
   `aops-cli discuss start --slug <slug>`, and open a linked coordination room
   with `aops-cli chat room create`.
2. Put the operator request, repository roots, constraints, and expected
   deliverables in the topic's first turn or as a context message in the room.
3. Give the other agent a precise directive through a room message or
   `pm review-request`: explicitly request independent research, not merely a
   review of your draft. Expected output should include a current-state map,
   recommendations, tradeoffs, risks, and open questions.
4. Announce the directive in the room with `chat message send`. If the other
   agent must be started manually, tell the operator which room/topic it should
   listen to.
5. Research independently in parallel and write your stance to the topic with
   `discuss turn`.
6. Announce the decision turn with a short `chat message send` containing the
   topic id/turn and the review questions for the other agent.
7. Wait for the decision loop with
   `aops-cli discuss wait --id <topic> --for <agent> --timeout-sec 540 --interval-sec 5 --json`,
   and wait for a coordination wake with
   `aops-cli chat listen --for <agent> --max-loops 1 --json`.
8. When the response arrives, read the other agent's full `discuss turn` or
   scratch file, not only the room TL;DR. Write another `discuss turn` for
   agreements, corrections, pushback, and points that require an operator
   decision; then ping the room.
9. If disagreement is material, run one more bounded loop. Do not write a final
   plan after one room response on architectural or high-impact topics.
10. Before implementation, complete at least two real-time turns and record the
    result as a final stance/consensus with `discuss conclude`. Direct
    implementation followed by review is allowed only when the operator
    explicitly requests an override/urgent mode, and the override is recorded.
11. The output is the locked decisions (`discuss conclude`), open operator
    questions, POC order, and owner/domain boundaries. Write Projectman or
    Docman records only through explicit commands.

Other-agent flow:

1. Accept the directive and write a room note saying that research is in
   progress and a result will follow; advance the read cursor with
   `chat catchup --apply`.
2. Research the requested sources independently; do not treat the primary
   agent's draft as the sole truth.
3. Write your research/stance with `discuss turn` and provide a short room
   summary with the topic id/turn.
4. If the primary agent provides evidence-backed pushback, write another
   `discuss turn` that states which points you accept, which you maintain, and
   which you leave to the operator.

Minimum quality gate: both agents contribute at least one independent
context/research turn, or the primary agent explicitly records the bounded
timeout and tells the operator that work is proceeding without the other
response.

### 17.5 ChatV3 product-channel room context

#### 17.5.1 Overview

`aops-cli chatv3` is not a hosted `aops-cli chat` room; it is the encrypted
product-channel/session CLI. Invite/session/member tokens and room epoch-key
context operate through the local ChatV3 session store. Use `aops-cli chat` for
hosted AOPS coordination and `aops-cli chatv3` for active product-room tracking.

Common commands:

```bash
aops-cli chatv3 listen --session codex --room general --after-seq <last-seq> --timeout-sec 60 --json
aops-cli chatv3 binding add --session codex --room general --binding-type projectman.review-request --ref-id <rr-id> --title "Slice review" --json
aops-cli chatv3 binding list --session codex --room general --json
aops-cli chatv3 room brief --session codex --room general --for claude --json
aops-cli chatv3 room summary --session codex --room general --after-seq <last-seq> --json
```

Rules:

1. `listen` exits with `0` for a new message and `22` for timeout; read/listen
   output carries `latestSeq` and `caughtUp`.
2. `binding add/list/remove` stores loose references; it does not alter
   PM/RR/Docman/Discuss truth.
3. `room brief` is a read-only onboarding pack containing guidance, members,
   presence, bindings, cursor references, and recommended next reads.
4. `room summary` is an agent-composed narrative digest pack. It provides
   `sourceRef.type=chatv3.room`, a sequence range, `nextReadRefs`,
   summarization-only `sourceMessages`, and a `memoryWrite` recipe with a
   `NARRATIVE-DIGEST` slot.
5. `sourceMessages` is only summarization input and is not copied verbatim into
   memory. The agent first creates an abstractive narrative digest, then writes
   the digest, references, and sequence range through an explicit
   `mem checkpoint` or `mem summary`.

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

`aops-cli artifact` is hosted metadata for generated or external artifacts. It stores the artifact shell and links to project-scoped refs; it is not the byte store. Use Fileman for filesystem snapshots and artifact records for pointers such as storage paths, report paths, exported archives, screenshots, or generated JSON.

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

`aops-cli skill` owns hosted reusable skill shells and skill versions. `.aops/hosted/skills/**` is only the read-only mirror; never edit it as canonical truth.

Authoring loop:

```bash
aops-cli skill list --hosted-project-slug aops --name "aops-cli-fileman" --json
aops-cli skill inspect --id <skill-id> --json
aops-cli skill version list --skill-id <skill-id> --json
aops-cli skill version create --hosted-project-slug aops --skill-id <skill-id> --content '@./SKILL.md' --entry-file SKILL.md --skill-standard aops-skill-v1 --meta '@./meta.json' --apply --json
aops-cli skill version publish --hosted-project-slug aops --id <skill-version-id> --apply --json
aops-cli sync pull --apply --hosted-project-slug aops --json
pnpm skills:claude-codex:sync
```

When `--version` is omitted, the CLI resolves the next version from hosted versions. If a publish or create reports a version conflict, use `skill version list`; mirror frontmatter can lag by one version immediately after publish.

## 21 Durable activity logs

### 21.1 Overview

#### 21.1.1 Overview

Durable activity logs are audit/readback evidence for hosted operations. Many hosted write sugars append best-effort activity records or structured server logs. Treat these logs as verification context, not planning truth.

Current operator rules:

1. Planning and execution state still belongs in Projectman.
2. Durable handoff and decisions still belong in Agentspace memory.
3. Activity logs are useful when proving that a hosted write, invoke, or flow ran with a concrete request/response.
4. If a dedicated `aops-cli activity ...` command is not present in your runtime, discover the hosted activity surfaces with `aops-cli agent tools --domain agentspace --json` or use the domain guide. Do not invent an activity command from old docs.

## 22 Hosted Fileman usage

### 22.1 Overview

#### 22.1.1 Overview

Fileman is hosted-canonical: there is no `.aops/fileman/**` repo-first mirror. It tracks directory roots, immutable snapshots, file history, diffs, restore/copy/zip handoff, and cleanup.

Actual CLI tree:

```bash
aops-cli file target track --root-path <directory> --apply --json
aops-cli file target list --json
aops-cli file target get --id <target-id> --json
aops-cli file snapshot create --target-id <target-id> --apply --json
aops-cli file snapshot list --target-id <target-id> --json
aops-cli file snapshot diff --from-snapshot-id <id> --to-snapshot-id <id> --json
aops-cli file snapshot restore --snapshot-id <id> --apply --confirm --json
aops-cli file snapshot copy --snapshot-id <id> --output-path <dir> --apply --json
aops-cli file snapshot zip --snapshot-id <id> --output-path <zip> --apply --json
aops-cli file file history --target-id <target-id> --path <relative-path> --json
aops-cli file clean --target-id <target-id> --dry-run --apply --json
```

`target track` expects a directory. To snapshot one file, track its parent directory and address the file by relative path through `aops-cli file file history|get|restore`.

For unfamiliar usage, run the nested help first:

```bash
aops-cli file --help
aops-cli file target track --help
aops-cli file snapshot diff --help
aops-cli file file history --help
aops-cli file clean --help
```

Run `file clean --dry-run` before destructive cleanup and then re-run with `--apply --confirm` only after reviewing the target list.

## 23 Docman sugar

### 23.1 Overview

#### 23.1.1 Overview

`aops-cli doc` is the hosted Docman surface for document groups, documents, versions, sections, pages, page versions, retrieval rows, publish output, and mirror pull.

Common flows:

```bash
aops-cli doc list --project-slug aops --json
aops-cli doc version list --document-id <doc-id> --json
aops-cli doc outline get --document-version-id <docver-id> --titles-only --depth 2 --json
aops-cli doc page draft-save --page-version-id <pagever-id> --document-link-id <document-section-link-id> --content '@./page.md' --apply --json
aops-cli doc set-current-version --document-id <doc-id> --version-id <docver-id> --publish-now --apply --json
aops-cli doc mirror pull --project-slug aops --group-uid aops-guides --document-slug aops-cli-user-guide --out-dir ./.aops/docman --apply --json
```

For a known page edit, use clone_all + targeted page/section CRUD. For a whole markdown refresh, use `doc import --from-markdown` with `--baseline`, `--guard-target`, and a `--dry-run` first.

Retrieval notes:

1. `doc scope search --local` searches the local mirror and can rank source or architecture files above the guide you expected.
2. `doc search --local` is document-granular and confirms presence; do not assume it returned a full section body.
3. To read a mirror section, use `aops-cli view doc-page <document>#<number-prefixed-slug>`, for example `aops-cli view doc-page aops-cli-user-guide#27-guard-flag-konvansiyonu`.
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

Use `view` for read-only context, then mutate through the owner family: `pm` for planning, `mem` for durable context, `doc` for documents, `file` for snapshots, `skill`/`prompt` for hosted reusable assets.

Do not run closeout commands unless the operator explicitly approves closeout. Ordinary stop points should write resume/handoff memory.

## 26 Installer model

### 26.1 Overview

#### 26.1.1 Overview

Runtime skills/prompts are loaded from user-level Codex/Claude homes, not directly from repo mirrors. AOPS installs pointer files that re-read `.aops/hosted/**` from the active repo.

Typical refresh:

```bash
aops-cli sync pull --apply --hosted-project-slug aops --json
aops-cli setup agent-assets --apply --target both --asset both
pnpm skills:claude-codex:check
```

After pointer template changes, restart Codex or Claude Code. Hosted content changes usually only need `sync pull`, because the pointer reads the repo mirror at trigger time.

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

The hosted server is the source of truth. The `.aops/**` tree is a read-only
local cache; do not treat it as canonical or hand-edit it as truth.

Server-canonical, mirrored as read-only caches:

1. `.aops/projectman/**` caches Projectman boards, tasks, sprints, issues, feedback, and views. Refresh with `aops-cli sync pull --project-slug aops --apply --json`.
2. `.aops/agentspace/memory/items/**` caches Agentspace memory. Refresh with `aops-cli sync pull ...`.
3. `.aops/agentspace/discussions/**` caches discuss topics/transcripts (discuss authoring is hosted). Refresh with `aops-cli sync pull ...`.
4. `.aops/hosted/prompts/**` and `.aops/hosted/skills/**` mirror hosted Agentspace prompt/skill current versions. Refresh with `aops-cli sync pull --apply --hosted-project-slug aops --json`.
5. `.aops/docman/**` mirrors hosted Docman documents. Refresh with `aops-cli doc mirror pull ...`.

To change content, write through the owner surface (`aops-cli pm ...`, `aops-cli mem ...`, `aops-cli discuss ...`, `aops-cli doc ...`), then refresh the cache.

## 30 Hosted guide mirror bootstrap

### 30.1 Overview

#### 30.1.1 Overview

Guide mirrors are Docman-owned, not `sync pull`-owned. Refresh AOPS operator guides with:

```bash
aops-cli doc mirror pull --project-slug aops --group-uid aops-guides --document-slug aops-cli-user-guide --document-slug aops-agent-assets-bootstrap --out-dir ./.aops/docman --apply --json
aops-cli doc mirror pull --project-slug aops --group-uid domain-guides --document-slug agentspace-user-guide --out-dir ./.aops/docman --apply --json
```

Use `sync pull` for hosted prompts/skills, and `doc mirror pull` for guides/documents.

## 31 Cross-cutting anti-patterns

### 31.1 Overview

#### 31.1.1 Overview

Avoid these:

1. Guessing flags after a validation error instead of reading `--help`.
2. Writing raw `agent invoke` payloads without `agent schema`.
3. Treating `aops-cli` as semantic owner for planning, memory, docs, files, or domain business state.
4. Hand-editing `.aops/hosted/**` or `.aops/docman/**` mirrors.
5. Assuming `sync pull` refreshes Docman guides.
6. Running full Docman import for a one-page edit when CRUD ids are known.
7. Writing closeout memory for an ordinary checkpoint.
8. Leaving test Fileman targets/snapshots after smoke tests; run `file clean` and remove temp roots.

## 32 Skill and user-guide search discipline

### 32.1 Overview

#### 32.1.1 Overview

Use the smallest useful read:

```bash
aops-cli <family> --help
aops-cli doc scope search --project-slug aops --q "<keywords>" --local --json
aops-cli doc outline get --document-version-id <docver-id> --titles-only --depth 2 --json
aops-cli view doc-page aops-cli-user-guide#27-guard-flag-konvansiyonu
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

Before asking Codex or Claude to use AOPS hosted skills, refresh mirrors and install pointers:

```bash
aops-cli sync pull --apply --hosted-project-slug aops --json
aops-cli setup agent-assets --apply --target both --asset both
pnpm skills:claude-codex:check
```

Destinations:

1. Codex skills: `~/.codex/skills/<skill>/SKILL.md`
2. Codex prompts: `~/.codex/prompts/<prompt>.md`
3. Claude skills: `~/.claude/skills/<skill>/SKILL.md`
4. Claude commands: `~/.claude/commands/<prompt>.md` when supported by the local Claude Code build.

If a pointer triggers but `.aops/hosted/skills/<name>.md` is missing in the active repo, refresh hosted mirrors for the canonical project slug first. If the skill belongs to another domain repo, switch to that repo or pull that hosted project mirror intentionally.

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
aops-cli start --resume <mission-id> --mode solo --board <board> --json
aops-cli start --reminder --task "<current task>" --area <area> --limit 3 --json
```

Important fields in a ready result:

1. `result.promptRef.path` / `result.promptRef.sha256` - prompt file and hash.
2. `result.memoryBrief` - read-only local-cache memory startup pack; `--no-memory-brief` skips it.
3. `result.sessionGuidance` - layered runtime rules, discipline guardrails, accepted playbooks, ranked experience briefs.
4. `result.mission.policyJson` - free-form policy seed for mission create/update.

`start --reminder` is not a full kickoff: it asks no questions, does not
serialize the full starter prompt, and writes no PM, memory, or hosted state. It
provides a bounded read-only answer to mid-session questions such as "where
were we, and which rules, playbooks, or experience items apply?"

## 36 Mission and implementation plan

### 36.1 Overview

#### 36.1.1 Overview

Mission is an Agentspace-owned session anchor; it does not replace Projectman
task/sprint/RR truth. An implementation plan is a Projectman sprint facade, and
the plan id is the sprint id.

```bash
aops-cli mission create --objective "<objective>" --policy-json '<json>' --apply --json
aops-cli mission list --summary --json
aops-cli mission get --id <mission-id> --json
aops-cli mission update --id <mission-id> --active-plan <sprint-id> --apply --json
aops-cli mission resume --id <mission-id> --depth light --limit 8 --json

aops-cli plan create --task <task-id> --name "<plan name>" --goal "<goal>" --apply --json
aops-cli plan get --id <sprint-id> --json
aops-cli plan update --id <sprint-id> --phases-json '@tmp/phases.json' --apply --json
```

Rules:

1. Mission stores intent, status, policy, and the active-plan reference.
2. Tasks, sprint phases/microtasks, issues, feedback, and RR/RRR records belong
   to Projectman.
3. Mission resume is compact and token-bounded; use `mission resume --full` for
   the raw skeleton only when necessary.
4. `start --resume <mission-id>` combines the starter prompt with the same
   compact mission pack.
5. Use `aops-cli pm utask ...` for single-item microtask edits; the plan facade
   is not a second table.

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

> This section is generated from the public Commander program registered by `buildCommunityProgram()`. Do not edit it by hand; regenerate it with `aops docs user-guide`.

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
| `aops assets` | Install and update public AOPS skills from one simple release archive |
| `aops auth` | Authenticate the CLI to a named remote-session target |
| `aops bootstrap` | Read the host bootstrap contract |
| `aops chat` | ChatV3 encrypted channels, rooms, agent sessions, coordination, and foreground listening |
| `aops chatv3` | ChatV3 encrypted channels, rooms, agent sessions, coordination, and foreground listening |
| `aops checkpoint` | Server-first session checkpoint facade over hosted Agentspace memory |
| `aops cockpit` | Open and independently operate AOPS Cockpit on its own loopback port |
| `aops codex` | Experimental local agent-runtime integrations for Codex Desktop |
| `aops config-root` | Print the brand-specific user config root |
| `aops db` | Operator database surface: full PostgreSQL backup, listing and restore |
| `aops discuss` | Manage hosted (server-first) Agentspace discussion topics |
| `aops doc` | Docman authoring, retrieval, and publish sugar over the hosted AOPS plane |
| `aops docs` | Generate AOPS documentation sections from live CLI registrations |
| `aops doctor` | Run mutation-free, profile-aware CLI/target/local-instance checks |
| `aops exp` (aliases: experience) | Author and read hosted Agentspace experience items (server-first) |
| `aops health` | Read host health |
| `aops host` | Host inventory, registration, diagnostics, and admin commands |
| `aops init` | Initialize repo-local AOPS Community metadata (.aops) |
| `aops license` | Inspect the public trusted-local license profile; commercial activation is unavailable |
| `aops mem` (aliases: memory) | Agentspace server-first memory commands (hosted memory-item ops are the source of truth; the local .aops memory tree is never written) |
| `aops plan` | Sprint-backed Projectman implementation plan commands |
| `aops playbook` | List reviewed playbooks projected from hosted Agentspace memory rules/constraints (server-first) |
| `aops pm` | Projectman authoring commands (hosted server-first) |
| `aops project` | Agentspace project sugar commands over the hosted AOPS gateway |
| `aops prompt` | Agentspace prompt and prompt-version sugar commands over the hosted AOPS gateway |
| `aops resource` | Agentspace resource sugar commands over the hosted AOPS gateway |
| `aops server` | Configure and operate a named AOPS Community server instance |
| `aops setup` | Inspect and bootstrap an AOPS Community installation |
| `aops skill` | Agentspace skill and skill-version sugar commands over the hosted AOPS gateway |
| `aops start` | Compose the AOPS Collaborative Startup kickoff prompt from explicit flags |
| `aops sync` | Server-first sync commands: refresh the read-only local cache of Projectman/Agentspace state plus hosted prompt/skill mirrors |
| `aops target` | Manage named local or remote AOPS server targets |
| `aops update` | Check, prepare, and apply a guarded global AOPS npm update |
| `aops version` | Show immutable CLI identity and optional target/runtime context |
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
| `aops assets install` | Install the bundled or selected AOPS skill release |
| `aops assets rollback` | Restore the previous release into global skill directories |
| `aops assets status` | Show locally installed AOPS skill release state |
| `aops assets update` | Update from a local release or the aops-assets GitHub release |

#### 39.2.11 `aops auth` commands

| Command | Purpose |
| --- | --- |
| `aops auth legacy-export` | Export historical local identity/tenant/role metadata without credentials |
| `aops auth login` | Login with browser approval and store target-bound encrypted tokens |
| `aops auth logout` | Revoke the remote session, then remove encrypted credentials for one target |
| `aops auth status` | Verify the selected target and current authenticated principal |

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
| `aops chat channels` | List channels owned by or joined by the verified AuthV2 principal |
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
| `aops chatv3 channels` | List channels owned by or joined by the verified AuthV2 principal |
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

#### 39.2.15 `aops cockpit` commands

| Command | Purpose |
| --- | --- |
| `aops cockpit health` | Check the Cockpit host health without mutation |
| `aops cockpit logs` | Show recent Cockpit logs |
| `aops cockpit open` | Start stopped local services as needed and open Cockpit |
| `aops cockpit restart` | Restart only the Cockpit process |
| `aops cockpit start` | Start only the Cockpit process; do not start the AOPS Server |
| `aops cockpit status` | Show Cockpit process status without mutation |
| `aops cockpit stop` | Stop only the Cockpit process; do not stop the AOPS Server |

#### 39.2.16 `aops codex` commands

| Command | Purpose |
| --- | --- |
| `aops codex register` | Register a verified local Codex task for one ChatV3 agent member |
| `aops codex registrations` | List local ChatV3-to-Codex registrations with session ids redacted |
| `aops codex unregister` | Remove one local ChatV3 member wake registration |
| `aops codex wake` (aliases: wakeup) | Wake the exact local Codex Desktop session through internal IPC |

#### 39.2.17 `aops db` commands

| Command | Purpose |
| --- | --- |
| `aops db backup` | Take one full PostgreSQL backup with pg_dump and verify it |
| `aops db list` | List backups and say which ones a restore will accept |
| `aops db restore` | Restore one full backup: --latest or --backup <path> |
| `aops db which` | Print the backup that --latest would choose, without restoring |

#### 39.2.18 `aops discuss` commands

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

#### 39.2.19 `aops doc` commands

| Command | Purpose |
| --- | --- |
| `aops doc answer` | Read a citation-first deterministic answer pack for a saved document version |
| `aops doc create` | Create a Docman document through the canonical flow surface |
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

#### 39.2.20 `aops docs` commands

| Command | Purpose |
| --- | --- |
| `aops docs user-guide` | Generate or refresh the dynamic appendix sections in the AOPS CLI user guide |

#### 39.2.21 `aops exp` commands

| Command | Purpose |
| --- | --- |
| `aops exp capture` | Capture a hosted experience item |
| `aops exp delete` | Delete one hosted experience item |
| `aops exp get` | Get one hosted experience item by id |
| `aops exp list` | List hosted experience items within the current owner scope |
| `aops exp promote` | Promote a hosted experience item into a durable memory item (or a playbook with --as-playbook) |
| `aops exp search` | Lexically rank hosted experience items (server-truth rows ranked client-side) |
| `aops exp update` | Patch one hosted experience item |

#### 39.2.22 `aops host` commands

| Command | Purpose |
| --- | --- |
| `aops host config` | Inspect or update the local host-owned env config |
| `aops host config set` | Write repo target and/or host log level into the local host env file |
| `aops host config show` | Show repo target and host log level from the local host env file |
| `aops host database` (aliases: db) | Inspect or mutate the active host database via AOPS db-admin routes |
| `aops host database backup` | Export or restore hosted database backups |
| `aops host database backup export` | Export a hosted database backup through the AOPS backup route |
| `aops host database backup restore` | Restore a hosted database backup through the AOPS restore route |
| `aops host database dump` | Run PG-native pg_dump / pg_restore against the configured PostgreSQL repo URL |
| `aops host database dump export` | Write a native PostgreSQL custom dump file with pg_dump |
| `aops host database dump restore` | Restore a native PostgreSQL custom dump file with pg_restore |
| `aops host database reset` | Reset hosted database tables and optionally recreate schema |
| `aops host database status` | Load hosted database status (/api/aops/settings/database/status) |
| `aops host diagnostics` (aliases: plugins) | Inspect host runtime diagnostics (/api/host-admin/plugins) |
| `aops host explain-registration` | Explain the stored registration manifest and projected host-config fragment |
| `aops host health` | Smoke the runtime health endpoint (GET /api/health) |
| `aops host hello` | Smoke the hello endpoint (GET /api/hello) |
| `aops host register` | Return commercial_profile_required; external domains are disabled in trusted-local |
| `aops host registrations` | List installed host registrations in the operator registry |
| `aops host unregister` | Remove an installed host registration |

#### 39.2.23 `aops license` commands

| Command | Purpose |
| --- | --- |
| `aops license activate` | Return commercial_profile_required without accepting commercial evidence |
| `aops license status` | Show the free built-in domain profile |

#### 39.2.24 `aops mem` commands

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

#### 39.2.25 `aops plan` commands

| Command | Purpose |
| --- | --- |
| `aops plan create` | Create a sprint-backed implementation plan, optionally bound to a kanban task |
| `aops plan get` | Get a sprint-backed implementation plan by selector |
| `aops plan list` | List sprint-backed implementation plans |
| `aops plan update` | Patch a sprint-backed implementation plan, including phases and microtasks |

#### 39.2.26 `aops playbook` commands

| Command | Purpose |
| --- | --- |
| `aops playbook get` | Get one hosted playbook projection |
| `aops playbook list` | List hosted playbook projections |
| `aops playbook project-set` | Load accepted sticky/durable project-scope playbooks for start/resume guidance |
| `aops playbook promote` | Promote reviewed experience into a playbook memory rule/constraint |
| `aops playbook show` | Render one hosted playbook as agent-readable markdown |

#### 39.2.27 `aops pm` commands

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

#### 39.2.28 `aops project` commands

| Command | Purpose |
| --- | --- |
| `aops project create` | Create a project |
| `aops project delete` | Delete a project |
| `aops project get` | Get a project by id |
| `aops project link` | Link a hosted project slug into the repo project registry |
| `aops project links` | List repo project registry links |
| `aops project links list` | List project links from .aops/aops.config.json |
| `aops project list` | List hosted projects |
| `aops project migrate-local-root` | Plan or apply flat .aops projectman/agentspace migration into .aops/projects/<slug> |
| `aops project update` | Update a project |

#### 39.2.29 `aops prompt` commands

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

#### 39.2.30 `aops resource` commands

| Command | Purpose |
| --- | --- |
| `aops resource create` | Create a scope-owned resource |
| `aops resource delete` | Delete a resource |
| `aops resource get` | Get a resource by id |
| `aops resource list` | List reusable resource shells |
| `aops resource update` | Update a resource |

#### 39.2.31 `aops server` commands

| Command | Purpose |
| --- | --- |
| `aops server attest-external-snapshot` | Optionally record an operator-owned external PostgreSQL snapshot for one risky migration plan |
| `aops server backup` | Create and verify a custom-format PostgreSQL backup plus receipt |
| `aops server backup list` | List existing backups and say which ones restore will accept |
| `aops server backup-readiness` | Report whether this host can take the backup an update depends on, without mutation |
| `aops server health` | Check the installed server health without mutation |
| `aops server lock-status` | Inspect the exact same-instance operation lock without changing it |
| `aops server logs` | Show recent server logs |
| `aops server migration-plan` | Read the exact pending native database migration plan without mutation |
| `aops server operation-status` | Inspect one exact durable operation journal without changing it |
| `aops server reconcile-operation` | Reconcile one exact interrupted operation under its instance lock |
| `aops server reconcile-recovery` | Explicitly finish or abandon one exact hard-killed recovery attempt |
| `aops server recover` | Recover one exact failed or unhealthy update from its verified backup |
| `aops server recover-lock` | Recover one exact stale lock generation proven dead or PID-reused |
| `aops server reset` | Remove local installation state; managed PostgreSQL is deleted only when explicitly requested |
| `aops server restart` | Restart the installed server |
| `aops server restore` | Restore database data; dispatches on the install profile |
| `aops server restore db` | Restore only PostgreSQL for one exact native application update |
| `aops server rollback` | Rollback application code; legacy flat form remains OCI-only until P4 |
| `aops server rollback app` | Rollback only native application code; never restore or rewind PostgreSQL |
| `aops server setup` | Configure and start an npm-package, source-checkout, or OCI server profile |
| `aops server start` (aliases: up) | Start the installed native npm/source host or OCI stack |
| `aops server status` | Show install and runtime status |
| `aops server stop` (aliases: down) | Stop the server without deleting data |
| `aops server update` | Update an installed native application or OCI stack |

#### 39.2.32 `aops setup` commands

| Command | Purpose |
| --- | --- |
| `aops setup ai` | Print a safe, copy-ready AOPS installation prompt for a terminal AI agent |
| `aops setup catalog` | Inspect or reconcile the inert signed AOPS official server catalog in the fixed aops-official-catalog scope (server contract: agentspace.official-catalog.reconcile) |
| `aops setup catalog reconcile` | Preview or apply an append-only reconcile from one verified Community release |
| `aops setup catalog rollback` | Restore a prior reserved current-version map without deleting history |
| `aops setup catalog status` | Inspect only the reserved official catalog scope |
| `aops setup docker` | Preview or apply the recommended Docker-hosted AOPS Server setup |
| `aops setup guide` | Print the packaged agent-readable AOPS installation skill |
| `aops setup init` | Inspect or apply an explicit AOPS installation path |
| `aops setup server-env` | Create or validate the private PostgreSQL/auth env for the npm server |

#### 39.2.33 `aops skill` commands

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

#### 39.2.34 `aops sync` commands

| Command | Purpose |
| --- | --- |
| `aops sync bootstrap` | Seed the read-only local cache from hosted server state and refresh read-only prompt/skill mirrors |
| `aops sync diff` | Show local records that are not synced |
| `aops sync pull` | Pull hosted Projectman/Agentspace state into the read-only local cache and refresh read-only prompt/skill mirrors |
| `aops sync rebuild-views` | Rebuild derived local cache markdown views and hosted read-only indexes |
| `aops sync resolve` | Resolve a local cache conflict by adopting the hosted/cache version (--prefer remote) |
| `aops sync sidecar` | Run a localhost-bound cockpit sidecar that exposes read-only local cache status/diff on the client machine |
| `aops sync status` | Show local cache sync state |

#### 39.2.35 `aops target` commands

| Command | Purpose |
| --- | --- |
| `aops target add` | Run `aops target add --help` for the current command contract. |
| `aops target doctor` | Run `aops target doctor --help` for the current command contract. |
| `aops target list` | Run `aops target list --help` for the current command contract. |
| `aops target remove` | Run `aops target remove --help` for the current command contract. |
| `aops target show` | Run `aops target show --help` for the current command contract. |
| `aops target use` | Run `aops target use --help` for the current command contract. |

#### 39.2.36 `aops update` commands

| Command | Purpose |
| --- | --- |
| `aops update apply` | Apply one exact prepared plan with automatic rollback and health verification |
| `aops update bridge` | Apply an old-CLI plan from an externally launched exact public candidate |
| `aops update check` | Compare the installed global AOPS closure with the public npm latest tag |
| `aops update prepare` | Download and verify exact candidate and rollback tarballs without changing the installation |
| `aops update recover` | Finish one stranded exact plan at its target closure after a failed apply |

#### 39.2.37 `aops view` commands

| Command | Purpose |
| --- | --- |
| `aops view board` | Show a Projectman board with tasks and sprints |
| `aops view boards` | List Projectman boards from .aops/projectman |
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
