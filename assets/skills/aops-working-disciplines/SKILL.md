---
name: aops-working-disciplines
version: 9
description: "Use when an AOPS operator or agent needs to choose or apply an AOPS working discipline: solo-pm-loop, build-review-chat, design-first-consensus, or coordinator-loop; bridge start mode/method vocabulary to mission.policy, Projectman review requests, issues, memory cadence, discuss consensus, verify-first consensus-to-build, consensus-to-plan binding, ChatV3 coordination, coordinator-delegated sessions, and explicit closeout."
metadata:
  short-description: "AOPS working discipline selection and execution guide"
  tags:
    - aops
    - discipline
    - mission
    - start
    - projectman
    - review-request
    - chat
    - discuss
    - memory
    - closeout
    - coordinator
---

# AOPS Working Disciplines

Use this skill when the task is about selecting or applying an AOPS working
discipline, interpreting `aops-cli start` output, seeding `mission.policy`, or
keeping PM/chat/discuss/memory/closeout behavior aligned with the approved discipline.

If this skill conflicts with `aops-cli start --help`, `aops-cli mission --help`,
or the Docman guide "AOPS Working Disciplines", those sources win.

## Load order

1. Run `aops-cli start --help` for live flags.
2. Read the Docman guide:

   ```bash
   aops-cli doc scope search --project-slug aops --q "AOPS Working Disciplines" --local --json
   ```

3. For implementation details in the aops repo, inspect
   `apps/aops-cli/src/commands/start-disciplines.ts`.
4. Use `aops-collaborative-work` for session-level startup and chat-room flow.
5. Use `aops-cli-projectman` for review-request/result, issue, sprint, and
   microtask mechanics.
6. Use `aops-cli-mission` for hosted mission create/update/resume mechanics.
7. Use `aops-cli start --reminder --task "<current task>" --area <area> --json` when a running session needs a bounded rules/playbook/experience refresh.

## Vocabulary

- `mode` is transport/session shape: `solo`, `solo+async-review`, `chat-room`.
- `discipline` is the policy preset plus guardrails.
- `method` is a compatibility alias for `discipline` in `aops-cli start`.

Canonical disciplines:

| Discipline | Use when | Review pattern |
| --- | --- | --- |
| `solo-pm-loop` | one agent, low or medium uncertainty | optional or async RR |
| `build-review-chat` | live implementer/reviewer, chat wake | RR per slice, RRR before commit |
| `design-first-consensus` | high design uncertainty | discuss final stances before implementation |
| `coordinator-loop` | operator delegates the session to one coordinator agent that directs implementer agents | RR per slice reviewed by the coordinator; commit on coordinator instruction |

## Quick selection

Use `design-first-consensus` when design uncertainty is high or the decision is
expensive to reverse.

Use `coordinator-loop` when the operator wants a single agent as their only
interface and delegates task definition, assignment, and review authority to it
(free-form multi-task sessions with one or more implementers).

Use `build-review-chat` when `--mode chat-room`, `agentCount > 1`, or the
operator asks for live RRR between peer agents.

Use `solo-pm-loop` for single-agent or low-uncertainty work. "fast fix" is not a
separate discipline; it is a small `solo-pm-loop` profile.

Operator override:

```bash
aops-cli start --mode solo --board <board> --discipline build-review-chat --json
aops-cli start --mode chat-room --board <board> --discipline coordinator-loop --json
```

## Verify-first consensus-to-build

`verify-first consensus-to-build` is a composite recipe, not a fifth discipline.
Use it when an operator reports an AOPS/project improvement issue in a room and
wants two active roles to verify the problem before deciding whether a formal
discuss topic is needed.

The verify-first stance is short and role-agnostic. Before formal discuss or
implementation, each active role records:

- whether the reported problem appears real
- which truth source was checked, such as PM, code, docs, logs, or readback
- whether formal consensus is needed
- the suspected risk or scope

If both stances say the work is low-uncertainty or atomic, the team may skip
standalone discuss and continue through `solo-pm-loop` or `build-review-chat`
with Projectman truth. If either stance flags material design uncertainty, a
cross-owner change, or an expensive-to-reverse decision, open standalone discuss
and follow the full final-stance/conclude ritual before implementation.

## Role and plan-binding policy

Working disciplines are role-agnostic. Use role names such as `implementer`, `reviewer`, `coordinator`, and `operator-approver` in reusable docs and skills. Concrete agent-to-role assignment belongs in mission-specific `mission.policy.roles`; do not hard-code runtime names into the reusable discipline.

`coordinator` is the canonical role id for the delegated session manager;
`master` and `operator-agent` are discouraged aliases (they collide with or
shadow the human `operator` role).

When material consensus exists, use a consensus-to-plan binding gate:

1. Discuss owns final stances and concluded consensus outputs.
2. The implementer or reviewer proactively asks whether the consensus should become a Projectman implementation plan.
3. After operator approval, carry the consensus ref into a PM task and sprint-backed implementation plan.
4. Execution waits until the reviewer accepts the plan approval review request.
5. `design-first-consensus` requires this gate before implementation; `build-review-chat` and `coordinator-loop` apply it when a material consensus appears during live build/review work.

## Mission policy

`aops-cli start --json` emits `result.mission.policyJson`. Use that string as
the seed for hosted mission policy:

```bash
aops-cli mission create --objective "<objective>" --policy-json '<result.mission.policyJson>' --apply --json
aops-cli mission update --id <mission-id> --policy-json '<result.mission.policyJson>' --apply --json
```

Mission policy stays free-form. The convention includes:

- `discipline{id,version,enforcement,selectedBy,signals}`
- `signalMapping`
- `guardrails[]`
- `guardrailGroups{execution,closeout}`
- `closeout`
- `review`
- `issue`
- `memory`
- `plan`
- `planning`
- `orchestration`
- `vocabBridge`

`signalMapping`, `planning`, and `orchestration` are emitted by current
`aops-cli start` seeds to explain why a discipline was chosen and how the
recommended PM/chat/review surfaces should be wired. Treat the convention as a
stable advisory shape, not a strict schema.

A `coordinator-loop` mission binds `roles.operator`, `roles.coordinator`, and
one or more `roles.implementer` entries; `roles.reviewer` defaults to the
coordinator when absent.

## Session guidance and reminders

The active discipline should also surface through `result.sessionGuidance`:

- runtime/rule pointers stay short: AGENTS.md, ChatV3 room rules, and command refs
- discipline guardrails are listed as ids, titles, phase, enforcement, and evidence summary
- accepted playbook briefs and ranked experience briefs are consulted before implementation, phase transitions, RR/RRR, and resume
- `aops-cli start --reminder --task "<current task>" --area <area> --limit 3 --json` refreshes this pack mid-session without creating tasks, writing memory, or serializing the full starter prompt

Do not treat playbooks as another discipline enum. A discipline selects the work rhythm; playbooks and experience are evidence/technique briefs that help the agent execute that rhythm without bulk-loading docs.

## Guardrails

Use the published guardrail registry from the Docman guide. Guardrails are grouped by `phase`: `execution` while work is happening, and `closeout` before leaving or handing off. Empty `evidence`
means reviewer-attested and not auto-checkable by the later `mission check`.

Always preserve these boundaries:

- PM owns task, sprint, issue, review request, and RRR truth.
- Discuss owns material consensus and final stances.
- Consensus-to-plan binding carries approved decisions into PM before implementation.
- Memory owns durable carry-forward context.
- ChatV3 is coordination and wake only.
- Hosted and Docman mirrors are read-only; change canonical truth through
  `aops-cli skill ...` or `aops-cli doc ...` commands.

`coordinator-loop` adds four execution guardrails:

- `coordinator-independent-research`: the coordinator verifies scope in code/PM/docs before assigning work.
- `single-operator-interface`: implementers route questions/decisions to the coordinator, never directly to the operator; the coordinator escalates only operator-owned decisions.
- `assignment-via-canonical-refs`: assignments carry mission/ktask/plan refs; chat prose alone is not an assignment.
- `idle-window-improvement`: the coordinator uses implementer work windows for doc/skill/tooling improvements and files findings as PM issues/feedback.

## Closeout

Closeout is explicit before leaving a mission/session. It is not automatic board
closeout and it is not leaving a ChatV3 room; those lifecycle actions stay
operator-only unless explicitly delegated.

All disciplines close with the base checklist:

- write handoff/resume memory with next action, validation state, and refs
- triage open review requests as accepted, follow-up issue, or deferred with owner
- triage open issues as resolved, follow-up work, or deferred with owner
- leave mission status truthful: active, handoff, completed, blocked, or deferred
- prove resume readiness from PM, memory, review, issue, and next-action refs
- record a concise session summary

Discipline-specific additions:

- `solo-pm-loop`: async review is accepted or deferred with owner before handoff.
- `build-review-chat`: every slice RR is accounted for, and accepted commits record hash, pathspec, and validation evidence.
- `design-first-consensus`: final stances/output are finalized, and the decision ref is carried into PM task/sprint-backed implementation plan/issue/feedback before implementation resumes; plan approval review is accepted or explicitly deferred with owner.
- `coordinator-loop`: `build-review-chat` items plus a truthful assignment queue — every operator request is bound to mission/ktask/plan records, completed, or explicitly deferred with owner; coordinator improvement findings are filed as issues/feedback.

Later command hints:

```bash
aops-cli mission check --closeout --id <mission-id> --json
aops-cli mission handoff --id <mission-id> [--complete] --apply --json
```

Until those helpers exist, enforce closeout through PM/RR state, memory,
review, chat wake refs, and honest handoff notes. Valid closeout states include
`present`, `missing`, `deferred-with-owner`, `not-applicable`, and
`waived-by-operator`.

## build-review-chat recipe

When a live build/review slice depends on a material consensus, bind that consensus to the PM task/sprint before implementation continues. Keep the reusable recipe role-agnostic: mission policy decides which concrete agent is implementer or reviewer.

For verify-first consensus-to-build, the room gets a short initial stance from each active role before formal discuss or implementation. The stance answers whether the issue is real, which truth source was checked, whether consensus is needed, and what risk/scope is suspected. The longer phase-to-command checklist belongs in the accepted playbook and `aops-collaborative-work`; this skill owns the rhythm and guardrail.

For every implementation slice:

```bash
aops-cli pm review-request create --task <task-id> --sprint <sprint-id> \
  --review-scope "sprint:<plan-id>" --requested-by <agent> \
  --target-agent <reviewer> --apply --json

aops-cli chatv3 send --session <session> --room general \
  --text "REVIEW READY: PM RR <id> ..." --json
```

Use `sprint:<plan-id>` as the default review scope for sprint-backed slices,
matching `aops-cli start` deferred bindings. `aops-cli pm review-request create
--help` also accepts `files:<glob>` for explicit file-only reviews. For mixed
slices, keep the sprint scope and list exact files in references/instructions
unless the reviewer asks for a file-only RR.

Do not commit until RRR is accepted. Use explicit pathspec and include only the
reviewed files.

## coordinator-loop recipe

Rhythm: operator request -> coordinator independent research (code, PM, docs)
-> mission/ktask/plan authoring -> chat assignment with canonical refs ->
implementer slices with RRs -> coordinator review and fix loop -> instructed
pathspec commit -> next task or idle-window improvement work.

Coordinator setup once per session:

```bash
aops-cli mission create --objective "<program>" \
  --policy-json '{"discipline":"coordinator-loop","roles":{"operator":"<name>","coordinator":"<agent>","implementer":"<agent>"}}' \
  --apply --json
```

Per task: `pm ktask create` + `plan create --task` + `mission update
--active-plan`, then a chat assignment message that carries the mission/ktask/
plan ids and names any binding policy documents (for example a UI system doc)
the implementer must re-read as a policy check.

Per slice: the implementer opens an RR targeting the coordinator; the
coordinator reviews with code/runtime verification (never status text alone),
requests fixes until accepted, then instructs the commit with explicit
pathspec.

Free-form session rules:

- multiple independent tasks may be active; each has its own ktask/plan; the
  mission anchors the session
- atomic fixes may run as a single RR without a plan; multi-slice work gets a
  sprint-backed plan
- material design uncertainty inside a task: coordinator runs a bounded peer
  deliberation with the implementer (about two turns); converged paths are
  proposed to the operator only when the decision is operator-owned; otherwise
  the coordinator decides and records it
- implementers never page the operator; the coordinator is the single
  interface

## design-first-consensus recipe

Run standalone discuss for material design choices. Do not implement until final
stances and the required operator approval are recorded.

After conclusion, the consensus-to-plan binding gate is required before execution:

- finalized consensus outputs must have no `_TBD_` placeholders
- the approved consensus ref must be carried into a PM task and sprint-backed implementation plan
- the plan should use operator-readable `NE / NICIN / DONE-WHEN` language
- a reviewer accepts the plan approval review request before implementation begins
- concrete agent names stay in `mission.policy.roles`, not in reusable discipline text

```bash
aops-cli discuss turn --topic <topic> --agent <agent> --kind final-stance --apply --json
```

No final stances means no conclusion. No `_TBD_` placeholders in concluded
outputs.

## Later

`aops-cli mission check --closeout` is intentionally later. Until it exists,
guardrails are advisory and enforced by start output, PM/RR discipline, review,
memory, and explicit closeout notes.
