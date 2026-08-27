---
name: aops-loop-interactive
version: 6
description: "Operator-facing interactive AOPS loop guide. Use when an operator wants to start, inspect, fan out, resume, or close out the aops-cli loop lifecycle with questions first, dry-run readiness, loop-pack preview, supervised-run double gates, parallel/worktree fan-out (v2), resumable agent sessions and short-lived resume-on-trigger (resume-agent / watch / session-reconcile), hosted-runner-v2 contract evaluation, explicit Runner-backed durable orchestration via --hosted-runner-mode=require-hosted, foreground listening, and Projectman review truth."
metadata:
  short-description: "Interactive AOPS loop lifecycle guide (v1 + v2 fan-out + resume + Runner delegation)"
  tags:
    - aops
    - interactive
    - loop
    - operator-facing
    - projectman
    - chatv3
    - review-request
    - runner
    - worktree
    - fan-out
    - resume
    - watch
    - hosted-runner-v2
---

# AOPS Loop Interactive

Use this skill when the operator wants an interactive, question-first guide for the `aops-cli loop`
lifecycle. This skill is a playbook only: it does not create a second state store, decision ledger,
chat ledger, or review system.

Canonical command details come from `aops-cli loop --help` and the nested
`aops-cli loop plan|pack|start|resume-agent|watch|session-reconcile|stop|resume|listen|status|parallel-plan|worktree-init|maker-run|slice-merge|cleanup|runner-eval --help`
surfaces. If this skill and CLI help disagree, CLI help wins. For a full prose walkthrough, point the
operator at the hosted Docman guide `aops-guides/aops-loop-user-guide` (read-only mirror at `.aops-cache/docman/aops-guides/aops-loop-user-guide.md`).

## Placement

This is the dedicated loop orchestration skill. `aops-interactive` may route here when the operator
asks for loop engineering, loop orchestration, supervised child-agent runs, parallel/worktree
fan-out, hosted runner evaluation, Runner-backed durable orchestration, foreground loop listening, or loop closeout.

## First Ask

Ask the operator concise questions before running commands. Use the runtime question UI when
available; otherwise ask plain text and wait only for answers that are required.

Required questions:

1. What PM surface anchors this loop? Ask for task id, sprint id, and any existing review-request id.
2. What is the scope and DONE-WHEN? Ask for repeatable acceptance criteria and files or domains in scope.
3. Who reviews? Ask for reviewer agent/operator and whether a PM review-request already exists.
4. What is the budget? Ask for max iterations, timeout minutes, and any cost/usage ceiling.
5. What tool/env policy applies? Ask whether this is dry-run only, supervised single child, or explicit real run — and for v2, whether parallel/worktree fan-out is intended.
6. What permission policy applies? Ask whether edits, tests, network, and approvals are allowed.

Do not auto-assign roles from agent names. Roles are operator-owned.

## Readiness Gate

Run `loop plan` first. Keep it dry-run unless the operator explicitly requests otherwise.

```bash
aops-cli loop plan --task <task-id> --plan <sprint-id> \
  --reviewer <reviewer-id> \
  --max-iterations <n> --timeout-min <minutes> \
  --json
```

Read the JSON, not prose. If readiness fails, show the failed checks and fixes, then stop or help the
operator resolve the missing inputs. Do not proceed to `loop pack` or `loop start` while required
readiness gates are failing.

The readiness questions must stay aligned with the loop facade gates:

- repeatable task and acceptance criteria
- reviewer or review-request evidence
- bounded budget and timeout
- tool/env capability
- permission policy
- adapter maturity and explicit real-run approval state

## Loop Pack Preview

After readiness passes, compose the loop pack:

```bash
aops-cli loop pack --task <task-id> --plan <sprint-id> \
  --reviewer <reviewer-id> \
  --chat-session <chatv3-session> --room <room-slug> \
  --after-seq <seq> \
  --json
```

Show the operator the important refs and prompts: PM task/sprint/review-request refs; ChatV3
session/room/cursor refs; discuss topic or mission refs; source hierarchy and source prompt refs;
maker (implementer) and checker (reviewer) role prompt refs; budget and run mode; the foreground
listener contract.

Do not persist pack output in a new owner store. If a durable handoff is needed, write Projectman
notes or Agentspace memory with source refs.

## Owner Boundaries

Repeat these boundaries before handoff:

- Projectman owns planning, implementation truth, review-requests, review results, issues, and closeout records.
- `aops-cli discuss` owns material decisions and final consensus outputs.
- ChatV3 owns coordination, wake messages, room refs, and cursors only.
- Agentspace memory owns durable handoff/checkpoint context.
- `aops-cli loop` is a facade over those surfaces; it is not their owner.
- Runner / Tasker owns durable run records, worker leases, heartbeats, expiry/reclaim, retries/backoff, cancel/resume/reconcile state, and ingress when `--hosted-runner-mode=require-hosted` is explicitly used.
- Hosted skills/prompts are canonical in the server; local hosted mirrors are read-only and must not be hand-edited as truth.

## Start Handoff (single child)

Prefer a dry-run handoff first:

```bash
aops-cli loop start --task <task-id> --plan <sprint-id> \
  --reviewer <reviewer-id> \
  --max-iterations <n> --timeout-min <minutes> \
  --json
```

For a real supervised run, require the operator to approve both gates in the same explicit instruction:

```bash
aops-cli loop start --task <task-id> --plan <sprint-id> \
  --reviewer <reviewer-id> \
  --run-agents --supervised-run-approved \
  --max-iterations <n> --timeout-min <minutes> \
  --json
```

The double gate is mandatory:

- `--run-agents` says a real child-agent run is allowed.
- `--supervised-run-approved` says the operator approved the supervised single-child discipline.

Do not infer either gate from enthusiasm, a prior plan approval, or a chat acknowledgement. Permission
bypass, destructive cleanup, final merge, and review acceptance remain operator/local or Projectman
controlled.

## Foreground Listening

When a run, review, or peer turn is pending, keep a foreground loop:

1. ChatV3 listen/read for operator or peer messages.
2. discuss wait/status when a decision topic is active.
3. PM review-request get/result status.
4. child adapter stream status when a child run exists.
5. `aops-cli loop status` for operator-readable cockpit output.

Timeouts are not completion:

- ChatV3 timeout or no message means re-poll.
- discuss exit `22` means re-poll.
- PM review-request `approved` or `changes_requested` is a review event, not a chat decision.
- Operator-addressed blocks stop the loop and return to the operator.

## Resume and resume-on-trigger (R0-R3)

A resumed agent picks up an existing session instead of cold-starting. Use this when re-tasking a
waiting agent, or when an agent that cannot background-listen (e.g. Codex) must be driven by an
external orchestrator.

Capture a resumable session (default OFF preserves the single-shot ephemeral behavior):

```bash
aops-cli loop start --plan <sprint-id> --reviewer <agent> \
  --child-agent codex --session-persist \
  --run-agents --supervised-run-approved --json
```

Read `result.start.childRun.sessionRef.sessionId`, then resume that session with a new directive
(double-gated, like start):

```bash
aops-cli loop resume-agent --session-id <id> --session-agent codex \
  --directive "check the room and continue" \
  --run-agents --supervised-run-approved --json
# or, when the id was not captured, resume the most recent session:
aops-cli loop resume-agent --resume-last --session-agent codex --directive "..." --run-agents --supervised-run-approved --json
```

Short-lived resume-on-trigger. `loop watch` does ONE poll of the configured sources and, on a
qualifying trigger, delegates to `resume-agent`, then EXITS. It is stateless and externally re-armed
(operator / scheduler / Runner) - never a daemon:

```bash
aops-cli loop watch --chat-session <session> --chat-room <room> --after-seq <n> \
  --watch-ignore <orchestrator-member-id> --watch-ignore <agent-member-id> \
  --resume-last --session-agent codex \
  --run-agents --supervised-run-approved --json
```

`--watch-ignore` is REQUIRED to enable ChatV3 auto-trigger: it lists the orchestrator's and the
resumed agent's own sender ids so the watch never self-triggers on its own output (de-dupe is by
senderMemberId). Without it the watch skips ChatV3 (safe default). `--directive` is an operator
override that forces a trigger.

Session lifecycle, brakes, and reconcile:

```bash
aops-cli loop session-reconcile --session-id <id> --session-agent codex --iteration <n> --max-iterations <n> --json
```

States are active/idle/waiting/stale/closed. Brakes (resume-count vs max-iterations, cost vs budget)
block a braked resume before it runs. A resumed session never merges, spawns nested agents, or writes
canonical ledgers. Crash/stale reconcile maps to the runner v2 reconcile-state contract and never
replays guarded writes.

Runner-backed durable orchestration (V2-final): the default mode remains local/stateless
`--hosted-runner-mode=defer`. When the operator or orchestrator explicitly sets
`--hosted-runner-mode=require-hosted`, `watch`, `stop`, `resume`, and `session-reconcile` delegate
durable intent to the hosted loop-runner-v2 surfaces and return `result.runnerDelegation` evidence.
The current hosted tool ids are `tasker.loop-runner-v2.event.record`,
`tasker.loop-runner-v2.status.get`, `tasker.loop-runner-v2.control.stop`,
`tasker.loop-runner-v2.control.resume`, and `tasker.loop-runner-v2.control.reconcile`.

Use the opt-in mode only when hosted Runner surfaces are expected to own durable state:

```bash
aops-cli loop watch --hosted-runner-mode=require-hosted \
  --chat-session <session> --chat-room <room> --after-seq <seq> \
  --watch-ignore <orchestrator-member-id> --watch-ignore <agent-member-id> \
  --resume-last --session-agent codex \
  --run-agents --supervised-run-approved --json

aops-cli loop stop --hosted-runner-mode=require-hosted --run-id <run-id> --json
aops-cli loop resume --hosted-runner-mode=require-hosted --run-id <run-id> --json
aops-cli loop session-reconcile --hosted-runner-mode=require-hosted --run-id <run-id> --json
```

Runtime ownership (concluded consensus): the loop facade ships ONLY the short-lived driver. It does
not own durable worker leases, heartbeats, retry/backoff, expiry/reclaim, ingress, or reconcile state;
Runner owns those through the hosted-runner-v2 contracts. The facade must never grow a long-running
stateful watcher or mutate worker leases directly.

## Parallel / Worktree Fan-out (v2)

When a sprint is sliced and the operator wants real parallelism, the v2 surfaces fan the work out
across **isolated git worktrees**, one maker per slice, with operator-gated merges. Each step is
explicitly gated — never chain them without the gate.

```
parallel-plan -> worktree-init -> maker-run -> [independent review] -> slice-merge -> cleanup
```

1. `loop parallel-plan` — fan-out gate plan only (no worktrees/agents). Requires prior single-child
   acceptance evidence (`--f5-evidence`) and explicit `--slice` boundaries.
2. `loop worktree-init --parallel-worktrees-approved` — materialize isolated worktrees on slice
   branches. No agents, no merges.
3. Open one **full-UUID** PM review-request per slice **before** the maker runs.
4. `loop maker-run --run-agents --supervised-run-approved` — at most one supervised maker in one slice
   worktree. No nested spawn, no merge, no canonical ledger writes.
5. An **independent** reviewer (maker != checker) verifies each slice in-code and records the PM
   result. A fallback/local self-review is advisory-only, never acceptance-grade.
6. `loop slice-merge --review-accepted --merge-approved` — merge one accepted slice **ff-only**, only
   after the RR is accepted **and** the operator approves the merge. One slice at a time.
7. `loop cleanup` — report-only; surfaces stale worker/worktree/artifact candidates and deletes
   nothing. Disposable-worktree deletion stays operator/local.

Practical notes:

- If the launch parent is dirty with unrelated work, run the fan-out from a **clean disposable
  parent** (clone/worktree from committed HEAD) so clean-parent gates pass without touching the
  operator's changes.
- Sequential ff-only merges require the next slice to descend from the previous merge result —
  **rebase to linearize**, never fall back to `--no-ff`.

## Hosted Runner v2 Evaluation

`aops-cli loop runner-eval` evaluates hosted runner/worker v2 boundaries and gaps without hosted
mutation. The hosted runner contract (records run/slice/worker-lease/event/artifact/cancel/reconcile,
plus event/artifact/status surfaces and operator stop/resume/cancel/reconcile controls) is defined in
`domains/tasker/tasker-kit`. Its invariants are enforced in-schema: operator-only controls,
`cancel.state=requested`, `replaysCanonicalWrites=false`, `secretMaterialStored=false`,
`requiresOperatorMerge=true`, idempotency keys, and source-tagged cost.

V2-final CLI wiring does not make `runner-eval` a daemon. It adds explicit `--hosted-runner-mode`
delegation on loop commands: `defer` keeps the facade local/stateless; `require-hosted` calls hosted
loop-runner-v2 event/status/control surfaces and returns `runnerDelegation` evidence. Hosted Runner,
not the loop facade, owns durable execution state; merge approval and secrets stay operator/local.

## Review And Closeout

Before closeout, ensure:

- A PM review-request exists for each implementation or skill/content change.
- The reviewer records a PM review result (independent of the maker).
- validation evidence is named: tests, schema/help checks, sync/pointer checks, or smoke commands —
  re-run anywhere a sandbox blocked a check rather than accepting on prose.
- Agentspace memory gets a concise checkpoint when the result is durable, and a comprehension digest
  records what was and was not proven.
- ChatV3 gets only a short status/wake with refs, not the full decision ledger.

Board, sprint, room, and mission closeout remain operator-controlled. Do not close them autonomously.

## Anti-Patterns

- Starting a real run after only one of the two real-run gates is present.
- Inferring approval from a chat acknowledgement.
- Treating ChatV3 as PM truth or a decision ledger.
- Treating `aops-cli loop` as a new owner store.
- Skipping `loop plan` because the operator already described the task.
- Proceeding after failed readiness checks without recording the override.
- Spawning more than one maker per slice, or letting a maker spawn nested agents, merge, or write canonical ledgers.
- Merging a slice before its PM review-request is accepted and the operator has approved the merge.
- Accepting a slice on a fallback/local self-review (advisory-only is not acceptance-grade).
- Falling back to `--no-ff` for sequential merges instead of rebasing to linearize.
- Enabling `loop watch` ChatV3 auto-trigger without `--watch-ignore` (risks self-triggering on the agent's or orchestrator's own output).
- Turning `loop watch` into a long-running daemon instead of a short-lived poll-once driver re-armed externally; durable orchestration belongs to Runner.
- Treating `--hosted-runner-mode=require-hosted` as permission for the facade to own worker leases, heartbeats, retry/backoff, or ingress instead of delegating to Runner.
- Resuming past the resume-count / budget brakes, or letting a resumed session merge / spawn nested agents / write canonical ledgers.
- Hand-editing local hosted mirrors instead of publishing hosted skill versions.
