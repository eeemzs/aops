---
name: aops-cli-tooling-agent
version: 41
description: "Compatibility index for AOPS CLI modular operator skills. Routes agents to aops-cli-core, aops-cli-discuss, aops-cli-chat, aops-cli-projectman, aops-cli-docman, aops-cli-agentspace, aops-cli-fileman, aops-cli-tasker, aops-cli-view, plus help-first and hosted mirror rules. Kept for old prompts/runtimes."
metadata:
  supersedes: "v40"
  short-description: "AOPS CLI modular skill index"
  tags:
    - cli
    - tooling
    - operators
    - index
    - deprecated-monolith
    - discovery
---

# AOPS CLI Tooling Agent — Modular Skill Index

This skill is a **compatibility pointer**. The old monolithic `aops-cli-tooling-agent` content is split into smaller family-focused skills. Do not expand this file back into a monolith.

## This skill covers

1. Routing to the right AOPS CLI family skill.
2. Help-first fallback when a family skill is unavailable in the runtime.
3. Local cache and hosted server guard reminders.

## Family skill index

| Need | Skill | Canonical user guide |
|------|-------|----------------------|
| CLI core, guard flags, sync, hosted mirror, raw invoke | **`aops-cli-core`** | `.aops/docman/aops-guides/aops-cli-user-guide.md` (Help-first model, Guard flag konvansiyonu, Local cache and hosted server sections) |
| discuss / decision / consensus (standalone topics, conclude outputs) | **`aops-cli-discuss`** | `.aops/docman/domain-guides/agentspace-user-guide.md` (Coordination semantics, Anti-patterns appendix sections) |
| chat / coordination / wake (hosted rooms, DMs, listen/catchup) | **`aops-cli-chat`** | `.aops/docman/domain-guides/agentspace-user-guide.md` (Coordination semantics) |
| review-request/result, re-review, issues, operator-approved closeout | **`aops-cli-projectman`** | `domains/projectman/USER_GUIDE.md` |
| Projectman board/sprint/task/issue/feedback (board-lifecycle folded) | **`aops-cli-projectman`** | `domains/projectman/USER_GUIDE.md` (Projectman model, Common workflow scenarios, Cross-session handoff sections) |
| Docman CRUD-first authoring, search, answer, publish | **`aops-cli-docman`** | `domains/docman/USER_GUIDE.md` |
| Agentspace memory/project/prompt/resource/artifact/skill/agent-profile | **`aops-cli-agentspace`** | `.aops/docman/domain-guides/agentspace-user-guide.md` |
| File snapshot/copy/backup/diff/restore | **`aops-cli-fileman`** | `domains/fileman/USER_GUIDE.md` |
| Tasker / Runner workflows / tracked execution | **`aops-cli-tasker`** | `domains/tasker/USER_GUIDE.md` |
| Read-only local-cache cockpit views | **`aops-cli-view`** | `.aops/docman/aops-guides/aops-cli-user-guide.md` (view subsection) |
| Deprecated board-lifecycle pointer | `aops-cli-board-lifecycle` (folded into projectman) | — |

## Canonical source split

1. CLI command shape: `aops-cli <family> --help`.
2. Domain semantics: the relevant hosted Docman mirror when migrated, otherwise the domain `USER_GUIDE.md` and `architecture.md`.
3. Operator playbook + workflow patterns: hosted `aops-cli-<family>` skills.
4. Section-focused reading: use the doc discovery ladder in the **Deep reading** section below (`aops-cli doc scope search` / `doc search` / `doc outline get`).

## Compatibility fallback

When a family skill is missing in the active runtime, use the help-first sequence:

```bash
aops-cli --help
aops-cli <family> --help
aops-cli <family> <subcommand> --help
```

Then read the matching domain guide:

1. `domains/projectman/USER_GUIDE.md`
2. `.aops/docman/domain-guides/agentspace-user-guide.md`
3. `domains/docman/USER_GUIDE.md`
4. `domains/fileman/USER_GUIDE.md`
5. `domains/tasker/USER_GUIDE.md`
6. `.aops/docman/aops-guides/aops-cli-user-guide.md`

## Hosted mirror rule

`.aops/hosted/**` is read-only mirror context. Hosted skill/prompt truth is changed through `aops-cli skill ...` and `aops-cli prompt ...`, then refreshed locally:

```bash
aops-cli sync pull --apply --hosted-project-slug aops --json
```

Docman guide mirrors under `.aops/docman/**` are separate. Refresh them with `aops-cli doc mirror pull --project-slug aops --group-uid <group-uid> --out-dir ./.aops/docman --apply --json`; `sync pull` does not refresh guide mirrors. In non-interactive automation, `--yes` can be added to supported commands as fail-fast mode, but it is not proof that an abort was fixed.

## Guard reminder

| Flag | Effect |
|------|--------|
| `--apply` | execute a guarded write (mostly mandatory for write commands) |
| `--apply --confirm` | execute a destructive operation (delete/overwrite) |
| `--json` | scriptable structured output |
| `--idempotency-key <id>` | retry-safe writes |
| `--preview` | validate without side effects |

When in doubt: `--help` is canonical.

## Deep reading

For section-focused reading instead of full-file linear reads:

```bash
# broad search across the project's guides (local mirror, no id needed):
aops-cli doc scope search --project-slug aops --q "<keyword>" --local --json
# exact search within a known guide version:
aops-cli doc search --document-version-id <docver-id> --q "<keyword>" --local --json
# section tree of a known guide version (hosted-only — no local fallback):
aops-cli doc outline get --document-version-id <docver-id> --json
```

There is no `aops-cli docman … --slug` command — use the ladder; reference sections by document title + section name + keywords, not bare numbers.

## Deprecation note

`aops-cli-tooling-agent` is **not hard-sunset**. It remains a stable index for old prompts, AGENTS references, and runtimes that have not switched to family-specific skills. New task prompts should prefer the smallest matching `aops-cli-<family>` skill.

## Override priority

If `--help` and this skill disagree, `--help` wins (canonical command surface). If the user guide and this skill disagree, the user guide wins.
