---
name: aops-cli-tasker
version: 9
description: "Use when an AI agent needs AOPS CLI Tasker + Runner operator playbook: discovery (human-first task manager + token-efficient --summary reads + scenario-first runner: tracked/run/workflow/workers/ingress), built-in system views, common workflows, and Tasker vs Projectman ownership boundary. Thin discipline guide; canonical mechanics live in domains/tasker/USER_GUIDE.md and command --help."
metadata:
  supersedes: "v8"
  short-description: "AOPS CLI Tasker + Runner thin discipline guide"
  tags:
    - cli
    - tasker
    - runner
    - workflow
    - tracked
    - workers
    - ingress
    - human-first-task
---

# AOPS CLI Tasker + Runner

Tasker owns human-first task manager semantics; Runner owns scenario-first execution (tracked/run/workflow/workers/ingress). Separate domain capabilities, one operator skill because they collaborate heavily: Tasker tasks frequently enqueue runner execution; runner closeouts can write back to Tasker.

`aops-cli tasker ...` and `aops-cli runner ...` are operator sugar surfaces. **Planning truth lives in Projectman** (`aops-cli pm ...`); see `aops-cli-projectman` for that boundary.

This skill is a **thin discipline guide**: discovery, workflows, top anti-patterns, and pointers. Deep mechanics, full rules, and detailed troubleshooting live in the canonical user guide. When this skill conflicts with `--help` or the user guide, those win.

## When to use this skill

1. Human-first task manager (list/get/create/update/delete + label/checklist/relation/comment)
2. Built-in system views (project, my-work, today, upcoming, all-tasks)
3. Tasker → Runner enqueue (`tasker create --enqueue`)
4. Runner tracked execution backed by Projectman subjects
5. Runner ad hoc runs, workflow instances, queue workers, ingress
6. Tasker vs Projectman ownership boundary

## Use another skill for

1. CLI guard flags, sync, hosted mirror mechanics: `aops-cli-core`.
2. Projectman board/sprint/task/issue/feedback CRUD and policy: `aops-cli-projectman`.
3. Memory, prompt, resource, artifact, skill, agent-profile: `aops-cli-agentspace`.
4. discuss / decision / consensus: `aops-cli-discuss`; chat / coordination / wake: `aops-cli-chat`.
5. Document graph CRUD/search/publish: `aops-cli-docman`.
6. File snapshot/copy/backup: `aops-cli-fileman`.
7. Read-only repo-first cockpit views: `aops-cli-view`.

## Canonical sources (authoritative)

When this skill is silent, ambiguous, or out of date, **defer to these**:

1. `domains/tasker/USER_GUIDE.md` — single source of truth for Tasker + Runner semantics.
2. `domains/tasker/architecture.md` — domain ownership rules.
3. `aops-cli tasker --help` and `aops-cli runner --help` (and nested family) for flag-level detail.
4. The `doc` discovery ladder (see Deep reading) for section-focused reading instead of linear file reads.
5. Hosted truth: Tasker + Runner server graphs; **no `.aops/tasker/**` repo mirror by default** (hosted-canonical).

## Discovery: which command for what

### Tasker (human-first task manager)

| Need | Command |
|------|---------|
| Single-project task list | `aops-cli tasker list --summary --json` |
| Cross-project system view | `aops-cli tasker list --view my-work\|today\|upcoming\|all-tasks --all-projects --summary --json` |
| Get one task | `aops-cli tasker get --id <task-id> [--summary] --json` |
| Workspace ensure | `aops-cli tasker ensure-workspace` |
| Create task | `aops-cli tasker create --title <t> --column <c> --assignee <agent:codex>` |
| Create + enqueue runner | `aops-cli tasker create --title <t> --enqueue --runtime-key codex [--auto-start]` |
| Update column/title | `aops-cli tasker update --id <task-id> --column in_progress` |
| Label + checklist + relation + comment | `aops-cli tasker label\|checklist\|relation\|comment` subfamilies |
| Delete | `aops-cli tasker delete --id <task-id> --apply --confirm` |

### Runner (scenario-first execution)

| Need | Command |
|------|---------|
| Tracked execution backed by PM | `aops-cli runner tracked enqueue --project-id <id> --sprint\|--issue\|--microtask\|--feedback <id> [--auto-start]` |
| Ad hoc run | `aops-cli runner run enqueue --project-id <id> --goal "<goal>" --runtime-key codex [--auto-start]` |
| Run state inspection | `aops-cli runner run state --run-record-id <row-id>` |
| Token-efficient run state | `aops-cli runner run state --run-record-id <row-id> --summary --json` |
| Run list filter | `aops-cli runner run list --project-id <id> --state queued --limit 20 --summary --json` |
| Get one run | `aops-cli runner run get --run-record-id <row-id> --summary --json` |
| Email-V1 delivery | `aops-cli runner run email-v1 --run-record-id <row-id>` |
| Workflow definitions | `aops-cli runner workflow definition list`, `runner workflow start`, `runner workflow step dispatch-plan` |
| Queue worker status | `aops-cli runner workers queue status`, `runner workers background status` |
| Ingress routing inspection | `aops-cli runner ingress inspect --summary --json` |

## Common workflows

### Workflow 1: Human-first task with cross-project view

```bash
aops-cli tasker ensure-workspace --apply --json
aops-cli tasker create --title "<task>" --column backlog --assignee agent:codex --apply --json
aops-cli tasker list --view today --all-projects --summary --json
aops-cli tasker update --id <task-id> --column in_progress --apply --json
```

### Workflow 2: Task → Runner enqueue (auto-start)

```bash
aops-cli tasker create --title "<task>" --enqueue --runtime-key codex --auto-start --apply --json
# verify run state
aops-cli runner run state --run-record-id <row-id> --json
```

Read the run row id from `artifacts.runRecordId` or `result.enqueue.data.runRecordId`.

### Workflow 3: Tracked PM-backed runner execution

```bash
aops-cli runner tracked enqueue --project-id <id> --sprint <sprint-id> --runtime-profile planning --auto-start
aops-cli runner run state --run-record-id <row-id> --summary --json
# closeout writes runtime audit + (if configured) AI delivery summary back to Tasker
```

### Workflow 4: Workflow instance step

```bash
aops-cli runner workflow definition list --project-id <project-id> --json
aops-cli runner workflow start --project-id <project-id> --definition-id <definition-id> --subject-type projectman.issue --subject-id <issue-id> --json
aops-cli runner workflow step dispatch-plan --workflow-instance-id <workflow-instance-id> --json
aops-cli runner workflow step run --workflow-instance-id <workflow-instance-id> --runtime-key codex --json
```

### Workflow 5: Long-running workers (background discipline)

```bash
# DO NOT lock operator foreground; run as background or daemon
aops-cli runner workers queue next --project-id <project-id> --lease-owner worker-1 --json
aops-cli runner workers background start --project-id <project-id> --profile standard --json
aops-cli runner workers background status --project-id <project-id> --json
```

## Always rules (pointers; do not duplicate the rule body here)

1. Always plan in Projectman (`aops-cli pm ...`); use Tasker for human-first execution tracking. They are NOT overlapping planning systems.
2. Always `--apply` for write commands; `--apply --confirm` for destructive (`tasker delete`).
3. Always verify runner state after `tasker create --enqueue` (`aops-cli runner run state --run-record-id <row-id> --json`).
4. Always run long-running `runner workers` in background or as a daemon — not in operator foreground.
5. For list/cross-project reads, prefer `aops-cli tasker list --summary --json`; it omits raw task `input`/`meta` and avoids duplicated `statusGroups[].tasks` payloads. Drop `--summary` only when you need the full task payload.
6. For run reads, prefer `aops-cli runner run list|get|state --summary --json`; it summarizes completion/meta payloads and keeps ids, lifecycle, subject, status, usage keys, and short get previews. Drop `--summary` only when you need raw completion/meta.
7. For ingress routing reads, prefer `aops-cli runner ingress inspect --summary --json`; it keeps rule/producer ids and routing decisions while replacing raw payload templates/notes with compact shape metadata.
8. Always use sugar surface (`aops-cli tasker ...` / `aops-cli runner ...`); raw hosted tools (`agent invoke`) are an explicit escape hatch.
9. Tasker has **single-project write** semantics; cross-project work uses system views or per-project commands.

## Top anti-patterns

Most common failure modes:

1. **Treating Tasker as the planning system**. Plan in Projectman; Tasker is human-first execution tracker.
2. **`tasker create --enqueue` without verifying runner state** afterward. Auto-start may not have fired due to runtime config.
3. **Long-running `runner workers` in operator foreground**. Use background or daemon.
4. **Hand-editing Tasker frontmatter or runner queue records**. Canonical mutation goes through sugar.
5. **Cross-project Tasker writes without explicit per-project commands**. Single-project write semantics apply.
6. **Hidden cross-domain mutation** in Tasker/Runner closeouts (e.g., silent PM/Docman writes). Closeouts publish only when configured for it.

## Deep reading

For section-focused reading instead of full-file linear reads:

```bash
# broad search across the project's guides (local mirror, no id needed):
aops-cli doc scope search --project-slug aops --q "tracked" --local --json
aops-cli doc scope search --project-slug aops --q "workflow" --local --json
# exact search within a known guide version:
aops-cli doc search --document-version-id <docver-id> --q "<keyword>" --local --json
# section tree of a known guide version (hosted-only — no local fallback):
aops-cli doc outline get --document-version-id <docver-id> --json
```

There is no `aops-cli docman … --slug` command — use the ladder; reference sections by document title + section name + keywords, not bare numbers.

For flag-level detail per command, always run `--help` first:

```bash
aops-cli tasker --help
aops-cli tasker list --help
aops-cli tasker get --help
aops-cli tasker create --help
aops-cli tasker update --help
aops-cli runner --help
aops-cli runner tracked --help
aops-cli runner run --help
aops-cli runner run list --help
aops-cli runner run get --help
aops-cli runner run state --help
aops-cli runner workflow --help
aops-cli runner workers --help
aops-cli runner ingress --help
aops-cli runner ingress inspect --help
```

If `--help` and this skill disagree, **`--help` wins** (canonical command surface). If user guide and this skill disagree, **user guide wins**.

## Tool Input Schema

Before authoring `--data`/`--input`/`--patch` payloads for hosted writes, fetch the live JSON Schema first: `aops-cli agent schema --tool <domain>.<operation>`. Full explanation in `aops-cli-core` (Tool Input Schema section). Architecture: slug:aops `tooling-cli-host-plugin-system` (Agent Gateway section, "Tool Input JSON Schema Discovery").
