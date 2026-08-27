---
name: aops-collaborative-work
version: 13
description: "Use when an AI agent starts or runs an AOPS-backed work session: token-efficient startup reads, solo-first agentic flow with PM/memory/doc discipline, optional async PM review requests across sessions, optional hosted chat-room task intake, optional multi-agent coordination via hosted chat + PM review + standalone discuss, sub-skill routing, and fast command recipes with help/schema fallback. Includes the verify-first consensus-to-build operational recipe for chat-room work."
metadata:
  supersedes: "fefa3617-9c71-4ade-bf63-63d10c07e0b7"
  updatedFor: "projectman-enum-help-alignment-20260826"
  short-description: "Solo-first agentic AOPS session playbook; chat-room/discuss optional"
  tags:
    - aops
    - bootstrapper
    - group:bootstrapper
    - agentic
    - solo-first
    - coordination-optional
    - chat
    - chatv3
    - async-review
    - token-efficiency
    - startup
    - mission-resume
    - verify-first
    - consensus-to-build
    - operational-recipe
---

# AOPS Collaborative Work

Solo-first playbook for working on any AOPS-backed project with `aops-cli` + `aops-server`. Collaboration (chat room, multi-agent coordination) is **optional and operator-opt-in**; the default is one agent working alone with PM/memory/doc discipline and, optionally, async review requests for future sessions. Paired starter prompt: `aops-collaborative-startup`.

If this skill conflicts with command `--help`, `aops-cli agent schema`, or the Agentspace/Projectman user guides, those win.

## Mode decision (pick once at kickoff)

| Operator signal | Mode | What it adds |
|---|---|---|
| Just a task, nothing else | **solo** (default) | PM + memory + doc discipline only |
| "review bırak" / wants a later review | **solo + async review** | PM review request left open for a future session |
| "chat odası kur" / room-based task intake / live peer | **chat-room** | hosted room (`aops-cli-chat`) for task-intake + coordination, with PM-backed review and standalone discuss for decisions |

Roles are operator-only in any multi-agent mode; never default by agent name or runtime brand. Modes layer onto solo: `chat-room` adds a hosted coordination room on top of the same solo PM/memory discipline.

Kickoff answers can be collected with `aops-cli start`: TTY-interactive for the operator; agents run `aops-cli start --json` (interview -> fill known answers -> ask the operator only `askOperator` items -> re-run with flags until `status: "ready"`). Default JSON is compact: prefer `--out <file>` and follow `result.promptRef.path`; re-run with `--full-output` only when an inline prompt is truly required. When resuming an existing mission, use `aops-cli mission resume --id <mission-id> --json` for the compact schemaVersion 1 pack, or `aops-cli start --resume <mission-id> --json` to compose the starter with that pack. `result.memoryBrief` is the default read-only startup memory pack; `result.sessionGuidance` carries layered runtime rules, discipline guardrails, accepted playbooks, and ranked experience briefs. Use `--no-memory-brief` for low-context or scripted starts that should skip the memory pack entirely.

## Startup block (token-efficient)

Run what you need; skip what you already know. Prefer summary reads; do NOT bulk-load guides or sub-skills up front.

```bash
aops-cli sync status --json                              # repo<->hosted state
aops-cli view dashboard --style agent                    # one-screen orientation
aops-cli start --task "<task>" --json --out tmp/start.md # compact JSON + result.promptRef.path
aops-cli start --task "<task>" --full-output --json      # inline prompt only when needed
aops-cli start --reminder --task "<task>" --area <area> --json # bounded read-only sessionGuidance
aops-cli start --task "<task>" --no-memory-brief --json  # skip result.memoryBrief only
aops-cli mission resume --id <mission-id> --json         # compact mission resume + sessionGuidance
aops-cli start --resume <mission-id> --json              # starter prompt + embedded compact mission pack
aops-cli mem brief --subject project --json              # standalone read-only startup pack when start was not used
aops-cli pm board list --json                            # only if board unknown
aops-cli pm board resume --board <slug> --json           # active tasks/sprints
aops-cli pm resume --for <agent> --with-chat --json      # pending work + chat unread (multi-agent)
aops-cli pm issue list --status open --json              # open defects/risks
aops-cli pm review-request list --status open --json     # async RRs waiting from prior sessions
```

Mirror refresh only when missing/stale: `aops-cli sync pull --apply --hosted-project-slug aops --json` (hosted skills/prompts) and `aops-cli doc mirror pull --project-slug aops --group-uid <aops-guides|domain-guides> --out-dir ./.aops-cache/docman --apply --json`.

## Layered rules and reminder packs

`result.sessionGuidance` and `aops-cli start --reminder` are bounded read-only packs. They are for orientation, not mutation:

- L1 runtime pointers: repo `AGENTS.md`, ChatV3/channel rules, and live command refs. Read the pointer or short rule, not every linked document.
- ChatV3 product-room shorthand: use `aops-cli-chat` for exact `chatv3` commands. It covers `--mode server-encrypted|e2e`, the `srv` invite form, and channel lifecycle sugar (`channel create`, `channels`, `channel delete`, `channel purge-before`). Deep mechanics live in Docman `slug:aops group:domain-guides document:chatv3-user-guide`; the repo mirror `.aops-cache/docman/domain-guides/chatv3-user-guide.md` is read-only.
- L2 discipline guardrails: guardrail id/title/phase/enforcement/evidence summary from `mission.policy` or the selected start discipline. Use full guide text only when a guardrail is unclear.
- L3 playbook and experience briefs: accepted playbooks plus ranked experience items. Default experience limit is 3, hard max is 5; ranking is deterministic from task/mission/area/tag/recency hints.
- `start --reminder` does not ask kickoff questions, serialize the full starter prompt, or write PM/memory/hosted state. Use it after interruptions, before a phase change, before RR/RRR, or when the operator says "hatirlat". Keep the pack around the soft 8-12 KB target; pull detail through the ladder only after it matters.

## Detail ladder (cheapest source first)

1. This skill's recipes + command `--help` (canonical flags): `aops-cli <family> --help`, then nested subcommand help.
2. The family sub-skill (map below) — load only the section you need.
3. Server contracts when authoring payloads: `aops-cli agent tools --domain <domain> --q <q> --summary --json`, then `aops-cli agent schema --tool <domain>.<op> --summary --json`. Most practical for hosted/raw writes; mandatory before `agent invoke`.
4. Canonical docs via the doc ladder: `aops-cli doc scope search --project-slug <slug> --q "<keyword>" --local --json`, then targeted `doc search` / `doc outline get`.

## Sub-skill map

| Need | Skill |
|---|---|
| Router/index over the family | `aops` |
| Guard flags, sync, mirrors, raw invoke fallback | `aops-cli-core` |
| Coordination / wake: hosted chat rooms/DMs, members, bindings, listen/catchup | `aops-cli-chat` |
| Decision/consensus protocol: standalone discuss, design-decision ritual, two-agent turn protocol, conclude outputs | `aops-cli-discuss` |
| Review + execution truth: board/task/sprint/microtask/issue/feedback/review-request lifecycle | `aops-cli-projectman` |
| Document graph authoring/search | `aops-cli-docman` |
| Memory/prompt/resource/skill assets | `aops-cli-agentspace` |
| Read-only cockpit views | `aops-cli-view` |
| File snapshots/diff/restore | `aops-cli-fileman` |
| Runner/tracked background tasks | `aops-cli-tasker` |

## Solo flow (default)

1. Register before code — every task lands in PM first (Turkish titles per operator practice):

   ```bash
   aops-cli pm ktask create --board <board> --column <col> --title "<başlık>" --description "<tanım>" --apply --json
   aops-cli pm sprint create --task <task> --name "<sprint>" --goal "<hedef>" --apply --json   # multi-step work
   ```

2. Memory uses the three-verb loop:

   ```bash
   aops-cli mem brief --subject project --json                         # start/resume; read-only
   aops-cli mem checkpoint --content "<status>" --task-id <task> --apply --json
   aops-cli mem summary --content "<session summary>" --apply --json  # session end; short by default
   ```

   `aops-cli start --json` returns `result.memoryBrief` unless `--no-memory-brief` is set, and returns `result.sessionGuidance` for layered rules/playbook/experience orientation. Do not immediately re-run broad memory or experience searches when the start/reminder pack already covers the same area. Use `mem checkpoint` at meaningful milestones/finish points, and `mem summary` at session end or explicit closeout/summary points. Handoff/carry-forward is a `resume` memory shape; keep one name (`resume`) instead of inventing a separate handoff kind. Use `mem brief|checkpoint|summary --help` for each sugar's "When to call" section.

3. Implement in small verifiable slices following repo patterns. Mutations need `--apply`, destructive ops `--confirm`, scripted reads `--json`. Never hand-edit hosted skill/prompt mirrors or Docman mirrors; their canonical truth is server-side.
4. Validate honestly: typecheck/tests/CLI smoke per repo convention. UI slices are verified in a live browser via Chrome MCP (Claude runtime: `Claude_in_Chrome` tools; Codex runtime: its Chrome MCP or browser skill). Never report validation you did not run in this session.
5. Defects/blockers become PM issues, operator guidance becomes PM feedback — not prose:

   ```bash
   aops-cli pm issue create --title "<bulgu>" --severity <sev> --apply --json
   ```

6. Record durable decisions/learnings in memory with source refs (task/sprint/issue/RR ids).

## Async review across sessions (optional)

No live reviewer required; PM carries the gate between sessions.

```bash
# executor, end of slice/session — leave the gate (--review-scope grammar: sprint:<id>|task:<id>|files:<glob>):
aops-cli pm review-request create --title "<slice>" --review-scope "files:<glob>" --description "<behavior/tests to review>" --apply --json
# future-session reviewer — discovery is part of the startup block:
aops-cli pm review-request list --status open --json
aops-cli pm review-request result --id <rr-id> --reviewer <agent> --outcome <approved|changes_requested|commented|blocked> --summary "<özet>" --apply --json
aops-cli pm issue create --source review --review-request <rr-id> --title "<bulgu>" --severity <sev> --apply --json   # one per material finding
# fixes are new slices behind a child RR:
aops-cli pm review-request create --parent <rr-id> --title "<fix slice>" --review-scope "files:<fix glob>" --apply --json
```

A PM-only RR wakes nobody — in async mode that is intentional; discovery happens at the next startup. In chat-room mode, ALSO post the RR ref into the room as a coordination wake (see `aops-cli-chat`); the canonical review record still lives in PM. Verify exact flags with `aops-cli pm review-request create --help` on first use.

## Chat-room mode (optional)

Room = operator task intake + coordination FLOW; decisions live in standalone discuss/PM, reviews in PM. One room per session.

```bash
aops-cli chat room create --slug <slug> --title "<title>" --created-by <lead> --member "<lead>:<role>" --member "<peer>:<role>" --member "<operator-id>:operator" --apply --json
aops-cli chat binding add --room-id <id> --binding-type projectman.board --ref-id <board-id> --title "PM board" --created-by <lead> --apply --json
# bind as created: repo.url, agentspace.discussion-topic, docman.document, projectman.review-request
aops-cli chat room brief --room-id <id> --for <peer>      # paste-ready peer onboarding
aops-cli chat listen --for <agent> --room-id <id> --timeout-sec 570 --interval-sec 15 --json
aops-cli chat catchup --for <agent> --room-id <id> --apply --summary --json
```

Listen exit codes: `0` unread → act, then `catchup --apply` (cursors advance ONLY via `catchup --apply` / `mark-read --apply`); `22` timeout → re-arm if still waiting; `21` room archived/membership ended → stop. Hosted room messages are the only chat wake path; a decision recorded in a discuss topic does not wake a room unless you post/bind it.

Room message protocol (keeps the room scannable for the operator):

```text
TASK <n>: <scope>                                # operator
ACK <n>: <goal + acceptance + validation plan>   # lead
DONE <n>: <summary + files + evidence>           # implementer
VERIFIED <n>: <evidence> | REWORK <n>: <issue>   # reviewer, mirroring the PM review outcome
```

When setup precedes tasks, post a readiness block in the room AND echo it to the operator — first line `Hazırım.`, then room/board ids (plus any active discussion-topic slug), members/roles, listening modes, peer onboarding block — and wait; no implementation before the first task arrives.

## Multi-agent (chat-room) mode (optional)

Escalate only when the operator assigns a live peer. The mechanics live in three sub-skills — load the one you need; do not duplicate them here:

1. **Coordination / wake** — `aops-cli-chat`: one hosted room per session as task intake + wake signal. Bind the PM board, any active discussion topic (`agentspace.discussion-topic`), and review requests (`projectman.review-request`) into the room. Use `chat listen` / `chat catchup` as the read loop.
2. **Review + execution truth** — `aops-cli-projectman`: every review is a PM `review-request` with an appended `result`; re-review is a child RR (`--parent <rr-id>`); material findings become explicit `pm issue create --source review --review-request <rr-id>`. Post the RR ref into the room to wake the reviewer; the canonical record stays in PM.
3. **Decisions / consensus** — `aops-cli-discuss`: for any material design fork, run a **standalone** discuss topic with the full design-decision ritual (independent research, ≥4 substantive non-final turns, `kind=final-stance` per agent, then `conclude`). Implementation stays blocked until the operator approves the consensus. There is **no automatic bridge** — surface the concluded decision explicitly via a PM feedback/issue carrying the discussion-topic ref, or a chat `agentspace.discussion-topic` binding.

Operator questions in multi-agent mode: kickoff window only (max 3-4, roles included). Afterwards climb the consensus ladder instead: references → peer/room consensus (independent stances, 2+ turns) → standalone discuss ritual for material forks → park operator-reserved gates (roles, scope, closeout, destructive ops) as `operator-decision-needed` in the room + PM feedback and continue other work.

## Verify-first consensus-to-build operational recipe

Use this recipe when the operator reports an AOPS/project issue in a ChatV3
room and wants the active roles to verify the problem before formalizing a
decision or implementation plan. This is the operational "how" for the
`verify-first consensus-to-build` recipe. The rhythm and guardrail owner is
`aops-working-disciplines`; do not duplicate that guardrail body here. This
section only maps phases to surfaces, commands, and evidence.

The recipe is role-agnostic. Use mission-local roles such as `implementer`,
`reviewer`, and `operator-approver`; concrete agent names belong in
`mission.policy.roles` and room messages for that one mission.

| Phase | Canonical surface | Commands and evidence |
|---|---|---|
| Intake and room orientation | ChatV3 coordination | Join/read/send/listen with `aops-cli chatv3`; room message records the operator symptom and active roles. |
| Verify-first stance | ChatV3 coordination + referenced truth source | Each active role posts a short stance: problem real/unclear, truth source checked, formal consensus needed or not, suspected risk/scope. |
| Material decision | Standalone discuss | For material, cross-owner, or expensive-to-reverse work, run `aops-cli discuss start`, alternating turns, final stances, and `conclude`. Low-uncertainty atomic work may skip discuss. |
| Consensus-to-plan binding | Mission + Projectman | Bind the accepted discussion refs into a mission and task/sprint-backed implementation plan with `NE / NICIN / DONE-WHEN`; request plan approval before implementation. |
| Implementation slice | Projectman + code/doc/hosted asset owner | Work in phases/microtasks; mutate canonical owners only. Open one PM review request per reviewed slice. |
| Review and rework | Projectman review-request/result | Reviewer appends RRR on the PM review request. Material findings become `pm issue create --source review --review-request <rr-id>`; suggestions become PM feedback. |
| Checkpoint | Agentspace memory | Use `mem checkpoint` after meaningful milestones, accepted slices, blockers, or handoff points. Do not checkpoint every chat line. |
| Closeout or handoff | PM status audit + mission/memory | Run read-only PM status audit, leave mission status truthful, write memory summary/checkpoint, and keep board/room open unless the operator explicitly closes them. |

Operational command skeleton:

```bash
# Intake / foreground listen in a ChatV3 product room.
aops-cli chatv3 join "<invite>" --handle <agent> --session <session> --save-session --json
aops-cli chatv3 read --session <session> --room general --after-seq 0 --mark-delivered --mark-read --json
aops-cli chatv3 send --session <session> --room general \
  "ACK: I will verify first, then decide whether discuss is needed." \
  --mark-delivered --mark-read --json
aops-cli chatv3 listen --session <session> --room general --after-seq <seq> \
  --timeout-sec 60 --interval-sec 5 --mark-delivered --mark-read --json

# Verify-first stance message template, one per active role.
aops-cli chatv3 send --session <session> --room general \
  "VERIFY-FIRST: real=<yes|no|unclear>; source=<PM/doc/code/ref>; consensus-needed=<yes|no>; risk-scope=<short note>" \
  --mark-delivered --mark-read --json

# Material decision path.
aops-cli discuss start --title "<decision>" --question "<question>" \
  --agent <implementer> --agent <reviewer> --apply --json
aops-cli discuss turn --topic <topic> --agent <role-or-agent> --kind proposal \
  --text "<independent stance with refs>" --apply --json
aops-cli discuss turn --topic <topic> --agent <role-or-agent> --kind final-stance \
  --text "<final stance>" --apply --json
aops-cli discuss conclude --topic <topic> --apply --json

# Consensus-to-plan binding.
aops-cli mission create --slug <mission-slug> --objective "<objective>" \
  --policy-json '@./mission-policy.json' --apply --json
aops-cli pm ktask create --board <board> --column Doing --title "<operator-readable task>" \
  --description "<consensus-backed scope>" --apply --json
aops-cli pm sprint create --task <task-id> --name "<implementation plan>" \
  --goal "NE: <what>; NICIN: <operator value>; DONE-WHEN: <reviewable acceptance>" \
  --apply --json
aops-cli pm sprint update-plan --id <sprint-id> --phases-json '@./plan.json' --apply --json
aops-cli pm review-request create --task <task-id> --sprint <sprint-id> \
  --review-scope "sprint:<sprint-id>" --requested-by <implementer> \
  --target-agent <reviewer> --title "<plan approval>" --apply --json

# Implementation review wake and result.
aops-cli pm review-request create --task <task-id> --sprint <sprint-id> \
  --review-scope "sprint-phase:<sprint-id>/<phase-label>" \
  --requested-by <implementer> --target-agent <reviewer> \
  --title "<slice review>" --apply --json
aops-cli chatv3 send --session <session> --room general \
  "REVIEW READY: PM RR <rr-id>; scope=<phase>; evidence=<tests/smokes/refs>" \
  --mark-delivered --mark-read --json
aops-cli pm review-request result --id <rr-id> --reviewer <reviewer> \
  --outcome <approved|changes_requested|commented|blocked> \
  --summary "<review evidence>" --apply --json

# Findings, suggestions, checkpoints, closeout checks.
aops-cli pm issue create --source review --review-request <rr-id> \
  --task <task-id> --sprint <sprint-id> --title "<material finding>" \
  --severity <low|medium|high|critical> --apply --json
aops-cli pm feedback create --source agent --task <task-id> --sprint <sprint-id> \
  --title "<suggestion>" --type improvement --severity <low|medium|high> \
  --suggestion "<recommended improvement>" --apply --json
aops-cli mem checkpoint --subject sprint --id <sprint-id> --task-id <task-id> \
  --sprint-id <sprint-id> --content "<milestone, evidence, risks, next action>" \
  --source-ref "review-request:<rr-id>" --validation-state "<validated|pending>" \
  --apply --json
aops-cli pm status audit --task <task-id> --sprint <sprint-id> --json
```

Message protocol for this recipe:

```text
VERIFY-FIRST: real=<yes|no|unclear>; source=<ref>; consensus-needed=<yes|no>; risk-scope=<short>
DISCUSS OPENED: topic=<slug>; reason=<material/cross-owner/expensive-to-reverse>
CONSENSUS BOUND: topic=<slug>; task=<id>; plan=<sprint-id>; plan-RR=<id>
REVIEW READY: RR=<id>; slice=<phase/microtask>; evidence=<tests/smokes/refs>
RRR: RR=<id>; outcome=<approved|changes_requested|blocked>; issues=<ids-or-none>
CHECKPOINT: subject=<task/sprint>; memory=<id>; next=<action>
```

Skip or shorten phases only when the two verify-first stances agree the work is
low-uncertainty and atomic. Otherwise keep the chain explicit: room stance,
standalone discuss, PM plan binding, plan approval RR, reviewed slices, and
honest closeout/handoff.

## Cross-cutting rules

1. PM is execution + review truth; standalone discuss is the decision/consensus ledger; memory is durable context; rooms/chat are flow; mirrors are read-only.
2. Closeout (board/room) is operator-only. Ordinary turn end = `mem checkpoint` or short `mem summary`; everything stays open.
3. Token efficiency: summary reads first; load sub-skills/docs section-by-section on demand; never re-read what is already in context; keep room messages protocol-shaped.
4. Stay inside established AOPS patterns; copy proven precedent; no parallel planning systems or out-of-AOPS artifacts unless the operator asks.
5. Minimize operator questions in every mode; resolve from context and references first.

## Anti-patterns

1. Implementing before the task exists in PM, or before the operator approves a discuss consensus on a material design fork.
2. Treating chat (room) as the decision/review ledger instead of discuss (decisions) / PM (reviews).
3. Multi-agent RRs with no room wake (async-mode RRs are intentionally unpaired); or expecting a discuss conclusion to wake a room without an explicit post/binding.
4. Bulk-loading guides/skills at startup "just in case".
5. Hand-editing hosted/docman mirrors as truth.
6. Reporting validation that was not run in this session.
7. Closing boards/rooms without explicit operator approval.
8. Asking the operator mid-run in multi-agent mode instead of climbing the consensus ladder.
