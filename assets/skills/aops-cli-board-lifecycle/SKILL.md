---
name: aops-cli-board-lifecycle
version: 6
description: "Deprecation pointer skill: aops-cli-board-lifecycle is folded into aops-cli-projectman. Routes agents to projectman v5+ for board lifecycle, kanban-task, sprint, microtask, issue, feedback, and handoff policy. Kept for at least one release cycle for compatibility."
metadata:
  supersedes: "v5"
  short-description: "Deprecated: board lifecycle folded into aops-cli-projectman"
  tags:
    - cli
    - projectman
    - board-lifecycle
    - deprecated
    - compatibility
---

# AOPS CLI Board Lifecycle — v4 Deprecation Pointer

This skill is intentionally folded into **`aops-cli-projectman`**. Board lifecycle policy, kickoff/resume/closeout cadence, sprint linking, microtask plan patching, and memory write rhythm now live as integral sections of the Projectman skill and **`domains/projectman/USER_GUIDE.md`**.

This pointer is kept for compatibility (existing prompts, AGENTS references, runtime catalogs). New task prompts should prefer `aops-cli-projectman` directly.

## Where to go now

| Need | Skill / source |
|------|---------------|
| Board kickoff/resume/closeout, kanban task CRUD, sprint phases/microtasks, issue/feedback, handoff | **`aops-cli-projectman`** |
| Deep mechanics, full anti-pattern list, troubleshooting, workflow scenarios | **`domains/projectman/USER_GUIDE.md`** (Anti-patterns, Troubleshooting, Common workflow scenarios sections) |
| CLI guard flags, sync, hosted mirror, raw invoke | `aops-cli-core` |
| discuss / decision / consensus | `aops-cli-discuss` |
| chat / coordination / wake | `aops-cli-chat` |
| Document graph CRUD/search/publish | `aops-cli-docman` |
| Memory + experience + agent-profile + assets | `aops-cli-agentspace` |

## Canonical entry points (quick reference)

```bash
aops-cli pm board kickoff --board <slug> --title "..." --goal "..." --apply --json
aops-cli pm board resume --board <slug> --json
aops-cli pm board closeout --board <slug> --content "..." --apply --json
aops-cli pm sprint update-plan --id <sprint-id> --phases-json @./plan.json --apply --json
aops-cli pm sprint set-status --id <sprint-id> --status completed --apply --json
```

## Deep reading

```bash
# broad search across the project's guides (local mirror, no id needed):
aops-cli doc scope search --project-slug aops --q "board lifecycle" --local --json
aops-cli doc scope search --project-slug aops --q "closeout" --local --json
# exact search within a known guide version:
aops-cli doc search --document-version-id <docver-id> --q "board lifecycle" --local --json
# section tree of a known guide version (hosted-only — no local fallback):
aops-cli doc outline get --document-version-id <docver-id> --json
```

There is no `aops-cli docman … --slug` command — use the ladder; reference sections by document title + section name + keywords, not bare numbers.

For flag-level detail: `aops-cli pm board --help`, `aops-cli pm sprint --help`, etc.

## Compatibility note

`aops-cli-board-lifecycle` is **not hard-sunset**. It remains a stable index. Future maintenance happens in `aops-cli-projectman` and `domains/projectman/USER_GUIDE.md`; do not expand this file back into a full board lifecycle guide.

If `--help` and this skill disagree, `--help` wins (canonical command surface). If the user guide and this skill disagree, the user guide wins.
