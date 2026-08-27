---
name: aops-cli-tasker
version: 13
description: "Use when an AI agent needs the AOPS CLI Tasker + Runner operator playbook: choose Tasker vs Projectman, inspect DAG-aware Runner status, use atomic worker claim/release, plan loop slice dependencies, or discover the complete hosted Tasker catalog. Thin guide; live --help and the canonical Tasker + Runner User Guide are authoritative."
metadata:
  supersedes: "v12"
  short-description: "AOPS Tasker + Runner thin operator discipline"
  tags:
    - cli
    - tasker
    - runner
    - dag
    - projectman-boundary
    - human-first-execution
    - guarded-control
---

# AOPS CLI Tasker + Runner

Use this skill for the minimal operator sugar over the hosted Tasker integrated
profile. It routes work; it does not copy the full capability catalog or create
a second scheduler/executor.

## Ownership boundary

- Plan and review in Projectman: `aops pm ...`.
- Track human-first execution in Tasker: `aops tasker ...`.
- Read or control run/lease state through Runner: `aops runner ...`.
- Store durable resume and decisions in Agentspace: `aops mem ...`.

A Tasker WorkGroup is an execution batch, not a Projectman sprint.

## Start with live help

```bash
aops tasker --help
aops runner --help
aops runner worker --help
aops loop parallel-plan --help
```

Installed help is authoritative for flags and command shape.

## Read DAG-aware status

```bash
aops runner status --run-id <run-id> --include-events --summary --json
aops runner event list --run-id <run-id> --limit 20 --summary --json
```

Status exposes ready/waiting/blocked/stalled/terminal scheduling, next-ready,
and blocker details. A dependency is ready only after the upstream slice is
`merged`; `accepted` is not enough.

## Plan dependencies

```bash
aops loop parallel-plan \
  --plan <sprint-id> \
  --slice core:Core:domains/tasker \
  --slice cli:CLI:apps/aops-platform/apps/aops-cli \
  --slice-deps cli=core \
  --reviewer <reviewer> \
  --f5-evidence <accepted-ref> \
  --worktree-root ../worktrees \
  --json
```

Omit `--slice-deps` for flat slices. The preview rejects unknown, duplicate,
self, and cyclic edges and shows deterministic topological waves.

## Atomic worker lease workflow

Preview, then apply the same claim intent:

```bash
aops runner worker claim --run-id <run-id> --worker-id <worker-id> \
  --runtime-key <runtime-key> --idempotency-key <key> --preview --json
aops runner worker claim --run-id <run-id> --worker-id <worker-id> \
  --runtime-key <runtime-key> --idempotency-key <key> --apply --json
```

Add `--slice-id <id>` for claim-this; omit it for deterministic claim-next.
Release the exact lease as `released`, `cancelled`, or `failed`:

```bash
aops runner worker release --run-id <run-id> --lease-id <lease-id> \
  --worker-id <worker-id> --terminal-state released \
  --idempotency-key <key> --apply --json
```

The domain service atomically enforces DAG readiness, terminal/cancel gates,
active-slice uniqueness, and `maxParallelSlices`. Do not implement these checks
as CLI advice or a second scheduler.

## Complete capability discovery

```bash
aops agent tools --domain tasker --summary --json
aops agent schema --tool <tasker-tool-id> --summary --json
```

Use `--input '@request.json'` or `aops agent invoke` only when bounded sugar is
insufficient.

## Anti-patterns

- Do not use Tasker as a second sprint or review system.
- Do not treat WorkGroups as Projectman sprints.
- Do not hand-edit hosted rows or `.aops-cache` mirrors.
- Do not infer completed work from an accepted control/claim response; reread
  status and events.
- Do not run a second worker scheduler, DCM, or migration engine from this skill.
- Do not let a maker merge, spawn nested agents, access secrets, or delete
  worktrees merely because it holds a lease.

## Canonical sources

1. Installed `aops tasker --help`, `aops runner --help`, and `aops loop --help`.
2. Docman: `Tasker + Runner User Guide` current published version; use the
   reviewed v2 draft only after it becomes current.
3. `domains/tasker/USER_GUIDE.md` as the reviewable materialization.
4. Docman: `Tasker Architecture` for developer-only DAG, transaction, database,
   profile, and Runner ownership contracts.

If these disagree, use live help for syntax, the User Guide for operator
semantics, and Tasker Architecture for implementation boundaries.
