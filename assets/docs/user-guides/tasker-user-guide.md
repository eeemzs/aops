# Tasker + Runner User Guide v2

_Release Notes:_ TASK-232 DAG and worker lease operator update draft. Publishing/current-version change remains a separate operator gate.

## 1 Choose the owner first

### 1.1 Overview

Use one owner for each kind of truth:

| Need | Owner |
| --- | --- |
| Boards, sprints, microtasks, issues, feedback, and review results | Projectman: `aops pm ...` |
| Human-first execution boards, tasks, and WorkGroups | Tasker: `aops tasker ...` |
| Run status, events, artifacts, and guarded operator controls | Runner: `aops runner ...` |
| Durable resume, decisions, blockers, and closeout memory | Agentspace: `aops mem ...` |

A Tasker WorkGroup is an execution batch, not a Projectman sprint. Tasker may
retain a weak Projectman origin reference, but it does not own a second planning
lifecycle.

## 2 Discover before acting

### 2.1 Overview

Start with the installed command surface:

```bash
aops tasker --help
aops tasker board --help
aops tasker task --help
aops tasker work-group --help
aops runner --help
```

The sugar commands intentionally cover only common operator workflows. Discover
the complete hosted Tasker catalog and the exact input schema when you need a
capability or field that is not exposed as a human flag:

```bash
aops agent tools --domain tasker --summary --json
aops agent schema --tool <tasker-tool-id> --summary --json
```

Use `--input '@request.json'` on a sugar command when you already have the full
hosted-tool input. `aops agent invoke` remains the explicit generic escape hatch.

## 3 Tasker boards and tasks

### 3.1 Overview

Read a project's execution boards and one board:

```bash
aops tasker board list --project-slug <slug> --json
aops tasker board get --scope-id <scope-id> --id <board-id> --json
```

Preview every write before applying it:

```bash
aops tasker board create \
  --scope-id <scope-id> \
  --data '{"name":"Delivery"}' \
  --preview --json

aops tasker board create \
  --scope-id <scope-id> \
  --data '{"name":"Delivery"}' \
  --apply --json
```

Create and update a human-first execution task:

```bash
aops tasker task create \
  --scope-id <scope-id> \
  --column <column-id> \
  --title "Implement the approved slice" \
  --type task \
  --preview --json

aops tasker task update \
  --scope-id <scope-id> \
  --id <task-id> \
  --patch '{"title":"Implement and verify the approved slice"}' \
  --apply --json
```

List or read Tasker tasks with the smallest useful response:

```bash
aops tasker task list --scope-id <scope-id> --summary --json
aops tasker task get --scope-id <scope-id> --id <task-id> --summary --json
```

Tasker sugar does not expose delete or broad compatibility CRUD. Use live tool
discovery and schema inspection when a hosted-only operation is genuinely
needed.

## 4 WorkGroups

### 4.1 Overview

The CLI keeps WorkGroups read-only because creation and scheduling policy are
not part of the minimal operator sugar:

```bash
aops tasker work-group list --scope-id <scope-id> --summary --json
aops tasker work-group get --scope-id <scope-id> --id <work-group-id> --summary --json
```

Do not model a Projectman sprint as a WorkGroup. Link execution to planning with
the canonical weak origin reference when the hosted schema supports it.

## 5 Runner reads

### 5.1 Overview

Runner sugar is a status-and-control surface over Tasker Loop Runner v2. It does
not create a second executor.

```bash
aops runner status --run-id <run-id> --include-events --summary --json
aops runner event list --run-id <run-id> --limit 20 --summary --json
aops runner artifact list --run-id <run-id> --summary --json
```

For a DAG run, status includes scheduling counts (`ready`, `waiting`,
`blockedByFailure`, `stalled`, and `terminal`), the deterministic `nextReady`
slice, and blocker details. A dependency is satisfied only after its slice is
`merged`; an accepted review alone does not make downstream work ready.

Plan loop slice dependencies with repeatable declarations:

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

The preview validates unknown, duplicate, self, and cyclic edges and shows a
deterministic topological order and execution waves. Omitting `--slice-deps`
keeps the existing flat-slice behavior.

## 6 Guarded Runner controls

### 6.1 Overview

Stop, resume, reconcile, worker claim, and worker release are writes. They
require `--preview` or `--apply`; provide a stable idempotency key so retries are
safe.

```bash
aops runner stop --run-id <run-id> --idempotency-key <key> --preview --json
aops runner resume --run-id <run-id> --idempotency-key <key> --apply --json
aops runner reconcile --run-id <run-id> --kind status --idempotency-key <key> --apply --json
```

Claim the next deterministic ready slice, or one explicit ready slice:

```bash
aops runner worker claim \
  --run-id <run-id> \
  --worker-id <worker-id> \
  --runtime-key <runtime-key> \
  --ttl-sec 300 \
  --idempotency-key <claim-key> \
  --preview --json

aops runner worker claim \
  --run-id <run-id> \
  --slice-id <slice-id> \
  --worker-id <worker-id> \
  --runtime-key <runtime-key> \
  --idempotency-key <claim-key> \
  --apply --json
```

Finish the exact lease with a terminal state:

```bash
aops runner worker release \
  --run-id <run-id> \
  --lease-id <lease-id> \
  --worker-id <worker-id> \
  --terminal-state released \
  --idempotency-key <release-key> \
  --apply --json
```

Valid terminal states are `released`, `cancelled`, and `failed`. Claim and
release are atomic hosted operations; they enforce readiness, cancellation,
terminal-run state, and `maxParallelSlices` in the domain service. After any
write, read status/events again. An accepted control request alone does not
prove useful work completed.

## 7 Guard and automation rules

### 7.1 Overview

1. Reads do not need an effect flag.
2. Tasker and Runner writes fail closed unless exactly one of `--preview` or
   `--apply` is supplied.
3. Automation should call the same app-owned `pnpm` scripts and CLI commands;
   do not hand-edit hosted rows or `.aops-cache` mirrors.
4. Prefer `--summary --json` for agent reads, then request the full payload only
   when necessary.
5. Use stable idempotency keys for claim/release/control retries; never invent a
   second scheduler around the hosted lease primitive.
6. Hosted maker mode claims before starting a child. Local/defer mode proves
   dependencies from git ancestry and performs no hosted lease mutation.
7. If guide prose disagrees with installed `--help`, installed help wins for
   command shape. Refresh the generated appendix and canonical Docman draft
   together.

## 8 Troubleshooting

### 8.1 Overview

| Symptom | Action |
| --- | --- |
| `guarded_write_requires_preview_or_apply` | Add `--preview` first, inspect JSON, then use `--apply`. |
| `loopRunnerV2NoReadySlice` or a waiting slice | Read Runner status scheduling and blocker details; complete the required dependency rather than retrying blindly. |
| `loopRunnerV2ParallelCeiling` | Wait for an active lease to release/expire; do not raise concurrency outside the run's operator ceiling. |
| Explicit slice claim is rejected | Verify the slice is ready, not already actively leased, and its dependencies are `merged`. |
| Unknown field or missing capability | Run `aops agent tools --domain tasker --summary --json`, then inspect the selected tool schema. |
| Tasker catalog is absent | Verify the intended AOPS Server candidate is running and Tasker is in its integrated hosted profile. |
| Runner control was accepted but state did not change | Read status and events; do not infer executor progress from the control response. |
| Planning data is being duplicated | Move the planning record to Projectman and keep only an origin reference in Tasker. |

## 9 Appendices

### 9.1 Overview

<!-- aops-generated:tasker-command-catalog:start -->

### 9.2 Generated Tasker and Runner command catalog

#### 9.2.1 Overview

> This compact appendix is generated from the public `aops tasker` and `aops runner` Commander registrations. Do not edit it by hand; regenerate it with `aops docs user-guide --guide tasker`.

| Command | Purpose |
| --- | --- |
| `aops runner artifact list` | List artifact refs for a Tasker runner run |
| `aops runner event list` | List events for a Tasker runner run |
| `aops runner reconcile` | Request a guarded Tasker runner reconciliation |
| `aops runner resume` | Resume a Tasker runner run through the guarded control surface |
| `aops runner status` | Get a Tasker runner status snapshot |
| `aops runner stop` | Stop a Tasker runner run through the guarded control surface |
| `aops runner worker claim` | Atomically claim the next ready slice, or one explicit ready slice with --slice-id |
| `aops runner worker release` | Atomically release, cancel, or fail one worker lease |
| `aops tasker board create` | Create a Tasker kanban board |
| `aops tasker board get` | Get a Tasker kanban board by id |
| `aops tasker board list` | List Tasker kanban boards |
| `aops tasker board update` | Update a Tasker kanban board |
| `aops tasker task create` | Create a Tasker task |
| `aops tasker task get` | Get a Tasker task by id |
| `aops tasker task list` | Search Tasker tasks |
| `aops tasker task update` | Update a Tasker task |
| `aops tasker work-group get` | Get a Tasker WorkGroup by id |
| `aops tasker work-group list` | List Tasker WorkGroups |

<!-- aops-generated:tasker-command-catalog:end -->

<!-- aops-generated:tasker-discovery:start -->

### 9.3 Generated Tasker capability discovery guide

#### 9.3.1 Overview

> Tasker and Runner sugar cover common operator workflows. The running server catalog remains authoritative for the complete hosted capability set, so this guide does not freeze a second tool inventory.

| Command | Purpose |
| --- | --- |
| `aops agent tools` | List federated tools from the canonical operator plane (/api/agent/tools) |
| `aops agent schema` | Print the live JSON Schema for one tool's input contract — use this before authoring --input payloads |
| `aops agent invoke` | Invoke a tool via the canonical operator plane (/api/agent/tools/{toolId}/invoke) |

Use the smallest useful read:

```bash
aops tasker --help
aops runner --help
aops agent tools --domain tasker --summary --json
aops agent schema --tool <tasker-tool-id> --summary --json
```

<!-- aops-generated:tasker-discovery:end -->
