---
name: aops-cli-core
version: 17
description: "Use when an AI agent needs AOPS CLI core operator playbook: help-first discovery, guard flags, project registry/project selection, partitioned sync, archive lifecycle, server-canonical vs local cache rules, hosted invoke fallback, doc-reading ladder, and schema fallback. Thin guide; .aops/docman/aops-guides/aops-cli-user-guide.md and command --help are authoritative."
metadata:
  supersedes: "v16"
  short-description: "AOPS CLI core operator thin discipline guide"
  tags:
    - cli
    - tooling
    - operators
    - discovery
    - guard-pattern
    - sync
    - hosted-mirror
    - help-first
    - schema-first
    - project-registry
    - archive
---

# AOPS CLI Core

`aops-cli` is the operator plane. It routes agents to domain owners; it is not the semantic owner of planning, memory, documents, files, or execution.

This is a thin discipline guide. Deep mechanics live in `.aops/docman/aops-guides/aops-cli-user-guide.md`; exact command shape lives in live `--help`. If this skill conflicts with either, `--help` and the user guide win.

## Use This Skill For

1. Help-first command discovery.
2. Guard flags: `--preview`, `--apply`, `--confirm`, `--idempotency-key`, `--json`, `--yes`.
3. Server-canonical truth vs local cache routing.
4. Raw hosted invoke fallback when sugar is absent.
5. Doc/guide reading ladder and schema fallback discipline.
6. Repo project registry routing: `aops-cli project link`, `aops-cli project links list`, `aops-cli project migrate-local-root`, and `pm --project-slug` project selection.
7. Partitioned sync (read/mirror): `sync --project-slug` and `sync --all-projects`.
8. Hosted PM archive lifecycle: `archive create`, `archive verify`, `archive delete`, and `archive decommission-check`.

Use the family skills for domain work: projectman for PM (including review-request/result and re-review), agentspace for memory/assets, discuss for decision/consensus, chat for hosted-room coordination/wake, docman for docs, fileman for snapshots, tasker for runner/tasker, view for read-only cockpit surfaces.

## Canonical Sources

1. `.aops/docman/aops-guides/aops-cli-user-guide.md`, read by section name/keyword, not by fragile section number.
2. `aops-cli --help`, `aops-cli <family> --help`, and nested `--help`.
3. Domain user guides: `domains/<domain>/USER_GUIDE.md` for semantic rules.
4. Hosted prompt/skill truth through `aops-cli prompt|skill ...`; `.aops/hosted/**` is only a mirror.
5. Hosted Docman guide truth through `aops-cli doc ...`; `.aops/docman/**` is only a mirror.
6. Repo project registry truth: `.aops/aops.config.json`, inspected with `aops-cli project links list --json`.

## Help-First Workflow

```bash
aops-cli --help
aops-cli <family> --help
aops-cli <family> <subcommand> --help
```

Decision chain:

1. Use sugar if help exposes it.
2. Inspect a compact hosted catalog: `aops-cli agent tools --domain <domain> --summary --json`.
3. Narrow noisy domains with `--q` and `--limit`: `aops-cli agent tools --domain <domain> --q <keyword> --limit 20 --summary --json`.
4. Inspect compact schema first: `aops-cli agent schema --tool <domain>.<operation> --summary --json`.
5. Invoke exact payload: `aops-cli agent invoke --tool <toolId> --input '@payload.json' --apply --json`.
6. Use `aops-cli api call` only as an explicit escape hatch.

## Agent Tool Catalog Ladder

Use compact discovery before dumping whole catalogs:

```bash
aops-cli agent tools --domain agentspace --summary --json
aops-cli agent tools --domain agentspace --q memory --limit 20 --summary --json
aops-cli agent schema --tool agentspace.memory-item.search-memory-items --summary --json
```

`agent tools --json` without `--summary` returns the full gateway payload and can be very large. `--summary` returns compact resource/tool rows plus next-step hints. `--examples` adds the first invoke example only when you explicitly need it; otherwise prefer `agent schema` for payload authoring.

## Guard Flags

| Flag | Meaning |
|---|---|
| `--preview` | validate/planning output without write effects when the command supports preview |
| `--apply` | execute a guarded write |
| `--confirm` | confirm destructive overwrite/delete/restore/cleanup |
| `--idempotency-key <id>` | make retries deterministic |
| `--json` | scriptable output |
| `--yes` | non-interactive/fail-fast; not a substitute for `--apply` or `--confirm` |

Common default: reads need no guard, writes need `--apply`, destructive writes need `--apply --confirm`.

## Server-Canonical Truth And Local Caches

The hosted AOPS server is the source of truth for planning and Agentspace state. Create, write, and read all go through the hosted gateway:

1. Projectman (`aops-cli pm ...`) — server-canonical.
2. Agentspace memory (`aops-cli mem ...`) — server-canonical.
3. Discussion topics (`aops-cli discuss ...`) — server-canonical (hosted discuss authoring).
4. Experience (`aops-cli exp ...`) — server-canonical.

Read-only local caches (refreshed by `sync pull`, never an authoring source):

1. `.aops/projectman/**`, `.aops/agentspace/memory/items/**`, `.aops/agentspace/discussions/**`, `.aops/agentspace/collabs/**`: a local cache of server state for offline/read inspection.
2. `.aops/hosted/prompts/**` and `.aops/hosted/skills/**`: hosted prompt/skill mirrors; refresh after a hosted prompt/skill publish with `aops-cli sync pull --apply --hosted-project-slug aops --json`.
3. `.aops/docman/**`: Docman guide mirrors; refresh with `aops-cli doc mirror pull ... --apply --json`. `sync pull` does not refresh Docman guide mirrors.

Do not hand-edit a read-only cache or mirror as canonical truth; change hosted truth through the hosted CLI and refresh the cache.

## Project Registry And Project Selection

The repo project registry lets one repo bind more than one hosted project so `pm --project-slug <slug>` (and the other family commands) can target the intended hosted project. Authoring is always hosted; the registry only controls selection and where the read-only local cache lives.

```bash
aops-cli project link --slug aops --mode local --local-root .aops/projects/aops --apply --json
aops-cli project link --slug cockpit --mode hosted-only --apply --json
aops-cli project links list --json
aops-cli project migrate-local-root --project-slug aops --local-root .aops/projects/aops --dry-run --json
aops-cli project migrate-local-root --project-slug aops --local-root .aops/projects/aops --apply --confirm --json
```

Operational facts:

1. `project link` mutates only `.aops/aops.config.json` after verifying the hosted project exists and is not archived/deleted.
2. `local-root` names the repo-relative path where that project's read-only cache (PM/memory/discuss mirrors) is materialized by `sync pull`.
3. `project link --mode` records the link mode metadata; PM/memory/discuss authoring routes to the hosted server regardless, so inspect a project through hosted/project views or read its local cache.
4. `migrate-local-root` is repo-local and only moves the local cache root. Run `--dry-run` first; the real move requires `--apply --confirm` because the prior root is moved to an archive.
5. `ownerRepo` and `parentProjectSlug` are metadata on the registry link; they do not change domain ownership.

## Partitioned Sync

Sync is a read/mirror operation that refreshes the local cache from the hosted server; it is not a PM authoring path. Use project selection when a repo has more than one linked project:

```bash
aops-cli sync status --project-slug aops --json
aops-cli sync diff --project-slug aops --json
aops-cli sync pull --project-slug aops --apply --json
aops-cli sync status --all-projects --json
aops-cli sync pull --all-projects --apply --json
```

Shipped behavior:

1. `sync --project-slug <slug>` resolves the project link from `.aops/aops.config.json` and scopes the cache refresh to that project's `localRoot`.
2. `sync --all-projects` runs `status`, `diff`, `pull`, or `bootstrap` once per runnable local project, skips ambiguous local projects without `localRoot` when multiple local projects exist, and reports project-level errors without fail-fast.
3. `sync pull` and `sync bootstrap` are project-level mirror commands. Use `--hosted-project-id|--hosted-project-name|--hosted-project-slug` to refresh hosted prompt/skill mirrors for additional hosted projects.
4. To author PM records, use the hosted `pm --project-slug` CRUD path described in `aops-cli-projectman`; sync does not write server records.

## Archive Lifecycle

Archive is currently documented in `aops-cli-core` rather than a separate skill because the shipped surface is a core CLI cleanup lifecycle over existing Projectman/Agentspace tools. Split it into `aops-cli-archive` only if it grows into a broader domain with its own guide and workflows.

```bash
aops-cli archive create --project-slug aops --apply --json
aops-cli archive verify --manifest .aops/archive/aops/<ts>/manifest.json --apply --json
aops-cli archive delete --manifest .aops/archive/aops/<ts>/manifest.json --json
aops-cli archive delete --manifest .aops/archive/aops/<ts>/manifest.json --apply --confirm --json
aops-cli archive decommission-check --manifest .aops/archive/aops/<ts>/manifest.json --json
```

Shipped behavior:

1. `archive create` downloads the hosted Projectman graph to `.aops/archive/<slug>/<timestamp>` and writes a local bundle. It does not delete hosted records.
2. New manifests are intentionally partial: `partial: true`, `decommissionSafe: false`, and non-empty `pendingDomains` for `agentspace.memory`, `agentspace.discussions`, `agentspace.chat`, and hosted reusable assets. This blocks full project/scope decommission until other domains are handled.
3. `archive verify` re-fetches the hosted PM graph and compares local files to remote counts/checksums. With `--apply`, it persists `verification.status: passed` into the manifest.
4. `archive delete` requires a verified manifest. Without `--apply`, it is a preview. Destructive hosted deletion requires `--apply --confirm`.
5. Delete order is children-before-parents: review requests, feedback, issues, microtasks, sprints, tasks, board-column links, columns, boards. The manifest records per-action deletion state so reruns skip already `deleted` or `missing` actions.
6. `archive decommission-check` allows full project/scope decommission only when `decommissionSafe === true` and `pendingDomains` is empty. The current create flow leaves this blocked by design.
7. Archive uses existing hosted tools (`agentspace.project.get-by-id`, Projectman list/get/delete tools). There is no new hosted archive tool in the agent catalog.

## Doc Reading Ladder

Use this ladder when a skill points to docs:

```bash
# broad project guide search, local mirror when available
aops-cli doc scope search --project-slug aops --q "<keyword>" --local --json

# search within a known saved document version
aops-cli doc search --document-version-id <docver-id> --q "<keyword>" --local --json

# hosted structure/id probe; can still be large on big docs
aops-cli doc outline get --document-version-id <docver-id> --titles-only --depth 0 --json

# actual mirrored section body
aops-cli view doc-page <document-slug>#<section-slug> --max-bytes 6000
```

`doc scope search` is broad and can rank source docs above guide docs. `doc search --local` is retrieval/search, not guaranteed full section body. `doc outline get --titles-only` drops page bodies but still returns link/section/page metadata, so use it for ids rather than as a compact table of contents. `view doc-page` selectors use `<document-slug-or-uid>#<exact-heading-or-heading-slug>`; partial keywords and group paths do not resolve. There is no `aops-cli docman ... --slug` command.

## Schema Fallback Ladder

Before direct hosted payloads:

```bash
aops-cli agent schema --tool <domain>.<operation> --summary --json
```

`--summary` flattens nested fields into compact `path`, `type`, `required`, and enum rows, plus next-step hints. Use full `--json` only when you need the complete JSON Schema.

If the compact summary marks `opaque: true`, or if full schema only exposes a flexible envelope such as `{ "data": { "additionalProperties": true } }`, use this order:

1. Matching sugar `--help`, if sugar exists.
2. Existing read shape from list/get/detail commands.
3. Domain `USER_GUIDE.md` and architecture docs.
4. `aops-cli agent openapi --domain <domain> --json` or the HTTP detail endpoint `GET /api/agent/tools/{toolId}`.
5. Source Zod/schema only when authoring direct invoke or debugging a wrapper bug.

If sugar returns 400/validation errors, stop retrying random flag variants. Compare sugar payload with `agent schema --summary` and then full schema/OpenAPI as needed; use raw `agent invoke` as a temporary bypass only after the contract is known, or file a wrapper bug.

## Anti-Patterns

1. Guessing flags or payload fields from memory.
2. Treating `aops-cli` as the domain owner.
3. Hand-editing `.aops/hosted/**` or `.aops/docman/**` mirrors, or the `.aops/projectman/**` / `.aops/agentspace/**` local cache, as canonical truth.
4. Assuming `sync pull` refreshes Docman guide mirrors.
5. Treating `--yes` as the missing write guard.
6. Reading whole guides when a targeted search plus `view doc-page` is enough.
7. Treating `sync` as a way to write server records; sync only refreshes the read-only local cache, while PM/memory/discuss authoring is hosted (`pm|mem|discuss --project-slug`).
8. Using `archive delete` before `archive verify --apply`, or treating `decommission-check` as passed while `pendingDomains` is non-empty.
