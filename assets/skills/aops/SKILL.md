---
name: aops
version: 11
description: "Use when an operator or AI agent needs to discover which AOPS CLI family skill applies — project planning, hosted missions, working disciplines, durable memory, document graph, file snapshots, multi-agent coordination, read-only cockpit views, hosted skill/prompt authoring, agent profiles, or board lifecycle. This skill is a thin router/index over the aops-cli-* skill family and AOPS working-discipline skill; load the matching family skill afterwards. Triggers: 'aops', 'aops-cli', 'AOPS overview', 'which aops skill', 'aops cli surface', 'plan in aops', 'aops mission', 'aops discipline', 'aops memory', 'aops discuss', 'aops chat', 'aops docman', 'aops fileman'."
metadata:
  supersedes: "v10"
---

# AOPS skill index

AOPS (Agentic Operations System) is the operator plane that coordinates planning, hosted missions, working disciplines, durable memory, documents, files, multi-agent coordination, cockpit views, and skill/prompt authoring through the `aops-cli` command surface. Domain ownership lives in the underlying domains (`projectman`, `agentspace`, `docman`, `fileman`, etc); `aops-cli` exposes the operator sugar.

This skill is a **thin router/index**. It does not carry workflow detail — it points to the family-specific skill that does. Load the matching `aops-cli-*` skill once you identify the right family; that family skill points to the relevant hosted Docman mirror or domain guide plus the live `aops-cli <command> --help`.

> Override priority across the AOPS skill family: `--help` wins over skill text (canonical command surface); domain user guide wins over skill text (canonical normative content). Searchable principles: AOPS Docman document `aops-doc-skill-system-principles` (slug:aops, search by title + section name + keywords).

> Closeout policy: board closeout is operator-controlled. Agents should not auto-run `pm board closeout` or closeout memory just because a turn is ending; use resume/handoff/checkpoint context until the operator explicitly requests or approves closeout.

## Discovery: which family for what

### aops-cli-core

The CLI core operator playbook: help-first command discovery, the guard flag conventions (`--preview`, `--apply`, `--confirm`, `--idempotency-key`, `--json`), the canonical operator family map (`init`/`setup`/`auth`/`host`/`agent`/`api`/`sync`/`view`), local cache vs. hosted server routing, and the raw hosted-invoke fallback when sugar is missing. **Load this skill first** when you are unsure about CLI guard semantics, sync mechanics, or how to escape from missing sugar to `agent invoke`. It also carries the canonical Tool-Input-Schema discovery block the other family skills point to. Canonical: `.aops/docman/aops-guides/aops-cli-user-guide.md`.

### aops-cli-discuss

Standalone decision / consensus: `discuss` durable decision topics, the design-decision ritual (independent research → ≥4 substantive turns → `kind=final-stance` per agent), the two-agent turn protocol, and deterministic `conclude` outputs. There is **no automatic decision bridge** — surfacing a decision to PM or chat is explicit. **Load this skill** when running a server-canonical design decision or peer consensus before implementation. Canonical: `.aops/docman/domain-guides/agentspace-user-guide.md` (Coordination semantics + Anti-patterns appendix + Troubleshooting sections).

### aops-cli-chat

Coordination / wake: hosted Agentspace rooms and DMs, members and roleKey vocabulary, reference bindings, messages, and `inbox`/`listen`/`catchup` read cursors. Rooms are FLOW (coordination + wake), not the decision ledger. **Load this skill** when coordinating peers, posting a review-request wake, or running a catch-up loop. Canonical: `.aops/docman/domain-guides/agentspace-user-guide.md` (Coordination semantics). For review-request/result, re-review, and operator-approved closeout truth, use `aops-cli-projectman`.

### aops-cli-projectman

Planning truth (server-canonical): `pm board/ktask/sprint/utask/issue/feedback/handoff` CRUD, atomic board lifecycle (`kickoff`/`resume`/operator-approved `closeout`), sprint phase + microtask plan patching, status derivation, subject-aware handoff, and the PM-vs-Agentspace ownership boundary. **Load this skill** when creating or moving kanban tasks, building or completing sprints, opening issues/feedback, or running board windows from kickoff/resume through operator-approved closeout. Canonical: `domains/projectman/USER_GUIDE.md`.

### aops-cli-mission

Hosted Agentspace mission command discipline: mission create/list/get/update/resume, mission policy seed through `--policy-json`, active Projectman implementation-plan references, `aops-cli start --resume` integration, and the ownership boundary between Mission, Projectman, and Agentspace memory. **Load this skill** when creating or resuming a hosted mission, carrying mission policy into startup, binding implementation-plan refs, or diagnosing mission/session resume behavior. Mission closeout checks and handoff helpers are explicit future automation unless the matching command exists in live `--help`.

### aops-working-disciplines

AOPS execution discipline routing: solo-pm-loop, build-review-chat, and design-first-consensus; how each maps to mission policy, Projectman review requests/issues, Agentspace memory, standalone discuss consensus, ChatV3 coordination, and closeout guardrails. **Load this skill** when selecting or interpreting a working discipline, reviewing `aops-cli start` discipline policy output, planning build-review-chat slices, or applying the closeout checklist without running operator-only board/room closeout.

### aops-cli-agentspace

Agentspace context and reusable assets: durable memory (`mem write/list/update/synopsis/resume/compact/prune/search`), sticky guidance, generated synopsis, experience capture (`exp`), the hosted asset families (`project`/`prompt`/`resource`/`artifact`/`skill`), agent profile metadata, and the activity log read surface. **Load this skill** when writing kickoff/resume/checkpoint memory, operator-approved closeout memory, authoring sticky bootstrap rules, publishing prompt or skill versions, or composing agent profiles for collab `--profile-mount`. Canonical: `.aops/docman/domain-guides/agentspace-user-guide.md` (Entity model + Memory model + Useful commands sections).

### aops-cli-docman

Document graph CRUD-first authoring + retrieval: section/page/page-version/link CRUD sugar, draft-save body edits, status workflow (`draft`/`review`/`published`/`archived`), retrieval helpers (`outline`/`index`/`summary`/`search`/`scope search`/`answer`/`source`), publish flow, and local mirror read fallback. **Load this skill** when targeting a known document node for an edit (prefer CRUD sugar over full mirror push), running search-driven retrieval over saved document versions, or publishing a document version's current pointer. Canonical: `domains/docman/USER_GUIDE.md`.

### aops-cli-fileman

File snapshot, diff, restore, copy, zip, and clean over hosted Fileman targets: target inventory CRUD, immutable snapshot lifecycle, snapshot-vs-snapshot or snapshot-vs-live diff, destructive restore (with explicit guards), non-destructive copy, archive hand-off, and retention pruning. **Load this skill** for snapshotting a file or directory, inspecting drift, restoring a snapshot, or applying retention policy — Fileman is hosted-canonical with no `.aops/fileman/**` mirror. Canonical: `domains/fileman/USER_GUIDE.md`.

### aops-cli-tasker

Insan-first task manager + scenario-first runner: `tasker` (task list/get/create/update/delete + label/checklist/relation/comment + system views like my-work/today/upcoming) and `runner` (tracked execution backed by Projectman subjects, ad-hoc runs, workflow instances, queue workers, ingress). **Load this skill** when pushing a task into the runner queue (`tasker create --enqueue`), inspecting runner run state, or composing a workflow instance — Projectman remains the planning owner; Tasker is human-first execution. Canonical: `domains/tasker/USER_GUIDE.md`.

### aops-cli-view

Read-only local-cache cockpit and hosted-inventory presentation: `view dashboard`, focused subject views (`view sprint/task/board`), Agentspace context views (`view memory/resume/discussions`), Docman/skill/prompt mirror views, hosted project inventory (`view hosted-projects/hosted-inventory`), and `view digest --depth shallow|deep` for handoff context packs. **Load this skill** when you need to inspect repo or hosted state without mutating it; view never syncs, refreshes, or mutates. Canonical: `.aops/docman/aops-guides/aops-cli-user-guide.md` (AOPS markdown view sugar section).

### aops-cli-board-lifecycle (deprecated pointer)

Folded into `aops-cli-projectman` from PR2 onward. Routes agents to the projectman skill for board kickoff/resume/operator-approved closeout, kanban-task and sprint policy, and atomic board windows. Kept as a stable pointer for one release cycle so older prompts and AGENTS references keep working. New task prompts should prefer `aops-cli-projectman` directly.

### aops-cli-sugar-authoring

Thin authoring playbook for **adding or extending sugar CLI commands** on `aops-cli` or `eops-cli` over a hosted domain operation. Identifies the right command surface (commander-based aops-cli vs. argv+registry eops-cli), points to canonical authoring sources, and keeps every new sugar wired to `agent schema --tool <id>` discovery. Use this when the target operation already exists in the kit/host-plugin manifest and you just need to expose or compose it as a sugar.

### aops-cli-tooling-agent (modular index)

Compatibility index for the modular AOPS CLI skill family after the v33 split. Routes agents to the right family skill (`aops-cli-core`, `aops-cli-discuss`, `aops-cli-chat`, `aops-cli-projectman`, etc.) and preserves help-first + hosted-mirror reminders for old prompts. Use when a runtime is missing a family-specific skill and an agent needs the fallback help-first sequence; otherwise prefer the smallest matching family skill.

## How to use this index

1. Identify the family from the table above (e.g. "I need to plan a sprint" → `aops-cli-projectman`).
2. Load that specific skill with the Skill tool (Claude) or by referencing it in the catalog (Codex).
3. The family skill points you to its canonical user guide section (by name), its `--help` surface, and the doc discovery ladder for section-focused reading.
4. When `--help` and skill text disagree, `--help` wins. When user guide and skill text disagree, user guide wins.

## Server-canonical and local cache

- Server-canonical truth: the hosted AOPS server is the source of truth for planning (`projectman`), durable memory (`agentspace.memory-item`), discussion topics (`agentspace.discussion-topic`), and experience. Create, write, and read these through the hosted `aops-cli` surface (`pm`, `mem`, `discuss`, `exp`).
- Local cache: `.aops/projectman/**`, `.aops/agentspace/memory/items/**`, `.aops/agentspace/discussions/**`, and `.aops/agentspace/collabs/**` are a read-only local cache of server state, refreshed by `sync pull`. Do not treat them as an authoring source or planning truth.
- Hosted prompt/skill mirrors: `.aops/hosted/prompts/**` and `.aops/hosted/skills/**`; refresh with `aops-cli sync pull --apply --hosted-project-slug aops --json`.
- Hosted Docman guide mirrors: `.aops/docman/**`; refresh separately with `aops-cli doc mirror pull --project-slug aops --document-slug <slug> --out-dir ./.aops/docman --apply --json`. `sync pull` does not refresh `.aops/docman/**`.
- Never hand-edit a mirror or cache as canonical truth. Change hosted truth via the hosted CLI (`aops-cli pm|mem|discuss|prompt|skill|doc ...`) and refresh the matching mirror.
- `--yes` is non-interactive/fail-fast mode for supported commands; do not treat it as a proven fix for an abort unless logs prove that.

## Tool Input Schema discovery

For any hosted write, fetch the live JSON Schema first instead of guessing payload fields: `aops-cli agent schema --tool <domain>.<operation>`. The full explanation lives in `aops-cli-core` (Tool Input Schema section); architecture: slug:aops `tooling-cli-host-plugin-system` (Agent Gateway section, "Tool Input JSON Schema Discovery").

## Deep reading (doc discovery ladder)

Search the guides by content instead of linear reads — pick the rung by what you know:

```bash
# broad search across the project's guides (local mirror, no id needed):
aops-cli doc scope search --project-slug aops --q "<keyword>" --local --json
# exact search within a known guide version:
aops-cli doc search --document-version-id <docver-id> --q "<keyword>" --local --json
# section tree of a known guide version (hosted-only — no local fallback):
aops-cli doc outline get --document-version-id <docver-id> --json
# project/sprint memory:
aops-cli mem search --subject project --id <project-uuid> --json
```

There is no `aops-cli docman … --slug` command — that path does not exist; use the ladder above. Reference guide sections by document title + section name + keywords, never a bare section number.
