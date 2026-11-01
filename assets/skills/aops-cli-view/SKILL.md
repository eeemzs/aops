---
name: aops-cli-view
version: 13
description: "Read-only views of plans, memory and documents: the AOPS CLI presentation/cockpit operator playbook for read-only local-cache markdown/JSON views and hosted project inventory, the selector contract, the filter matrix, output controls, and the view ownership boundary. Thin discipline guide; canonical mechanics live in the installed AOPS CLI User Guide (AOPS markdown view sugar section) and command --help."
metadata:
  supersedes: "v12"
  short-description: "AOPS CLI local-cache view / cockpit / presentation guide"
  tags:
    - cli
    - view
    - cockpit
    - presentation
    - local-cache
    - hosted
    - read-only
    - markdown
    - json
---

# AOPS CLI View

`aops-cli view ...` is a **read-only presentation cockpit**. Most commands are local-cache and read the markdown files that `aops sync pull` keeps under `.aops-cache/**`. Explicit hosted commands (`hosted-projects`, `hosted-inventory`) call hosted list APIs for inspection only. View **never** syncs, refreshes mirrors, writes cache/index files, or runs domain mutations. Default output is agent/TUI compatible markdown; `--json` returns the same read-model as a stable envelope.

This skill is intentionally not a domain skill: presentation is a CLI-internal cockpit concern. **Canonical view content lives in the installed AOPS CLI User Guide** (AOPS markdown view sugar section); skill text is workflow + filter/selector reference.

## When to use this skill

1. Local-cache dashboards (project cockpit, board, sprint, task, issue, feedback, memory, resume, discussions)
2. Hosted project inventory inspection (`hosted-projects`, `hosted-inventory`)
3. Skill/prompt/doc mirror views from `.aops-cache/hosted/**`/`.aops-cache/docman/**`
4. Focused context pack via `digest` (sprint/task/board, depth shallow|deep)
5. Filter and selector contract for read-only views

## Use another skill for

1. CLI guard flags, sync, hosted mirror mechanics: `aops-cli-core`.
2. Planning mutations (board/sprint/task/issue/feedback create/update): `aops-cli-projectman`.
3. Memory/agent-profile mutations: `aops-cli-agentspace`; discuss decision/consensus topics: `aops-cli-discuss`; chat coordination/wake: `aops-cli-chat`.
4. Document graph CRUD/search/publish: `aops-cli-docman`.

## Canonical sources (authoritative)

When this skill is silent, ambiguous, or out of date, **defer to these**:

1. The installed AOPS CLI User Guide — `aops-cli-user-guide.md` in the folder `aops assets status --json` reports as `userGuidesRoot`; it includes the "AOPS markdown view sugar" section under "`.aops` local cache and sync".
2. `aops-cli view --help` and nested family help (per subcommand) — flag-level detail.
3. Section-focused reading: search the installed guide for keywords such as "view sugar", "filter" or "digest" instead of reading it whole; reference sections by document title + section name, not bare numbers.
4. Local-cache views read the `.aops-cache/**` mirror; hosted views read hosted list APIs. **Canonical truth still lives in the originating domain** (Projectman, Agentspace, Docman, hosted server).

## Discovery: which view for what

| Need | Command |
|------|---------|
| Project cockpit | `aops-cli view dashboard` |
| Repo-bound projects | `aops-cli view projects` |
| Hosted Agentspace project list | `aops-cli view hosted-projects` |
| Per-project hosted inventory (docs/skills/prompts/resources) | `aops-cli view hosted-inventory --hosted-project <slug>` |
| Local-cache PM list views | `aops-cli view boards/tasks/sprints/issues/feedback` |
| Focused window detail | `aops-cli view sprint\|task\|board <selector>` |
| Agentspace context | `aops-cli view memory/resume/experience/discussions` |
| Hosted skill/prompt mirror | `aops-cli view skills/prompts` |
| Docman mirror | `aops-cli view docs/doc/doc-page` |
| Focused context pack (handoff) | `aops-cli view digest --sprint\|--task\|--board <selector> --depth shallow\|deep` |

## Selector contract (essential)

1. Local-cache selectors accept full UUID, 8+ char prefix, slug, or exact title.
2. Hosted project selectors accept hosted project id, scope id, slug, name, or 8+ char prefix via `--hosted-project`.
3. Slug match is case-insensitive; exact title/name match is whitespace-trimmed.
4. Ambiguous selectors raise an error listing candidates so the operator can disambiguate.

## Filter matrix (essential)

| Komut | Filter flag'leri | Anlam |
|-------|------------------|-------|
| `view memory`, `view resume` | `--durability`, `--kind`, `--subject`, `--id` | durability=short\|durable\|sticky; kind=kickoff\|resume\|closeout\|note\|rule\|...; subject=project\|board\|sprint\|task\|ktask\|utask\|issue\|feedback; id=full UUID veya 8+ char prefix |
| `view tasks` | `--board`, `--status` | board=slug/name/id; status=column adı veya slug |
| `view issues`, `view feedback` | `--status`, `--severity`, `--board`, `--sprint`, `--task` | status=frontmatter; severity=low\|medium\|high\|critical |
| `view sprints` | `--board`, `--status` | board=slug/name; status=todo\|doing\|completed\|paused |
| `view discussions` | `--status`, `--agent` | status=active\|concluding\|concluded\|abandoned; agent=participants |
| `view experience` | `--type`, `--area` | type=technique\|tool\|script\|problem-solution\|idea; area=tag |
| `view hosted-inventory` | `--hosted-project`, `--scope-resolution` | hostedProject=id/slug/name/prefix; scopeResolution=explicit\|cascade |

Filters AND together. Empty result emits `No matching records.` fallback.

## Output controls

| Flag | Effect |
|------|--------|
| `--style agent\|compact\|wide` | layout density |
| `--json` | scriptable envelope (`surface: "local-cache-view"` or `"hosted-view"`) |
| `--max-items <n>` | cap returned/rendered records |
| `--max-bytes <n>` | hard-capped at 32768 |
| `--link-mode none\|relative\|absolute` | path link rendering; default is `none` |

Local-cache JSON envelope footer: `local-state` (synced/dirty/conflict/local/deleted), not raw `syncState`. Hosted JSON envelope: `surface: "hosted-view"`.

Hosted commands also accept common auth flags: `--api-base-url`, `--access-token`, `--refresh-token`, `--timeout-ms`, `--tenant-id`, `--locale`, `--fallback-locale`, `--scope-id`, `--scope-resolution`. Local-cache command help may display these shared flags, but they are only used by explicit hosted view commands.

## Common workflows

### Workflow 1: Daily cockpit

```bash
aops-cli view dashboard --style agent
aops-cli view boards
aops-cli view tasks --board <slug> --status Doing
aops-cli view issues --status open --severity high
aops-cli view feedback --status open
```

### Workflow 2: Focused subject inspection

```bash
aops-cli view sprint <selector> --max-items 20
aops-cli view task <selector>
aops-cli view memory --subject sprint --id <sprint-id>
aops-cli view resume --subject project
```

### Workflow 3: Handoff context pack

```bash
aops-cli view digest --sprint <sprint-id> --depth shallow
aops-cli view digest --task <task-id> --depth deep --max-bytes 32768
aops-cli view digest --board <board-slug> --depth deep
```

### Workflow 4: Hosted inventory inspection (read-only)

```bash
aops-cli view hosted-projects --style compact
aops-cli view hosted-inventory --hosted-project <slug> --scope-resolution explicit
```

## Always rules (pointers; do not duplicate the rule body here)

1. View is read-only by contract — never mutates, syncs, or refreshes mirrors.
2. Local-cache view commands read only the `.aops-cache/**` mirror; hosted view commands call only hosted read/list APIs.
3. View does NOT refresh `.aops-cache/hosted/**` or `.aops-cache/docman/**` mirrors; use `aops-cli sync pull` for hosted prompt/skill mirrors and `aops-cli doc mirror pull` for Docman guide mirrors.
4. Use selectors before guessing UUIDs; if a selector is ambiguous, run a list view first to disambiguate.
5. Filter flags AND together; empty result emits `No matching records.` fallback (intentional, not an error).
6. Use focused `view digest` over generic `view dashboard` when handing off context to another agent — digest scopes per subject and respects `--depth`.

## Top anti-patterns

Most common failure modes:

1. **Using `view` as a mutation surface**. View is read-only by contract.
2. **Treating `view dashboard` JSON as canonical state**. It is a composed read-model; canonical state remains per-domain.
3. **Treating `view hosted-inventory` as sync/bootstrap**. It only reads; hosted prompt/skill mirror update is `sync pull`, while Docman guide mirror update is `doc mirror pull`.
4. **Bypassing selector errors by guessing UUIDs**. Run a list view first when ambiguous.
5. **Editing derived view files** (`.aops-cache/projectman/views/index.md`, etc) — they are regenerated after every mutation.
6. **Loading `view dashboard` for handoff** instead of focused `view digest --depth shallow|deep`.

## V2 deferred surfaces

Documented for awareness; not yet shipped:

1. `view relations <selector>` — cross-domain RelationResolver.
2. `view skill|prompt|experience <selector>` detail inspect.
3. `aops-cli ls`, `aops-cli show` aliases.
4. `--out <path>` artifact writer (stdout remains default).
5. Hosted relation graph deeper edge resolver.
6. Cache/index when performance demands.

## Deep reading

For section-focused reading instead of full-file linear reads, search the installed AOPS CLI User Guide (`userGuidesRoot`) for "view sugar", "filter" or "digest". Reference sections by document title + section name + keywords, not bare numbers.

For flag-level detail per command, always run `--help` first:

```bash
aops-cli view --help
aops-cli view dashboard --help
aops-cli view hosted-projects --help
aops-cli view hosted-inventory --help
aops-cli view sprint --help
aops-cli view task --help
aops-cli view board --help
aops-cli view digest --help
aops-cli view memory --help
aops-cli view discussions --help
```

If `--help` and this skill disagree, **`--help` wins** (canonical command surface). If the installed AOPS CLI User Guide and this skill disagree, **user guide wins**.
