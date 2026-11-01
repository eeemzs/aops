---
name: aops-cli-agentspace
description: "aops mem, memory: the AOPS CLI Agentspace playbook for token-efficient memory and resume, kickoff, checkpoint, decision, blocker and handoff notes, session summaries, experience and playbooks, reusable hosted prompts and skills, agent profiles, activity, and structured discussions."
metadata:
  version: "31"
  supersedes: "v30"
  short-description: "AOPS Agentspace memory, knowledge, assets, and discuss guide"
  tags:
    - aops
    - cli
    - agentspace
    - memory
    - checkpoint
    - experience
    - playbook
    - skill-discovery
    - agent-profile
    - discuss
    - help-first
---

# AOPS CLI Agentspace

Agentspace owns durable context and reusable knowledge: memory, experience,
playbooks, prompts, resources, artifacts, skills, agent profiles, activity, and
structured discussion topics.

It does not own all work state. Projectman owns tasks/plans/issues/reviews,
Docman owns versioned documents, ChatV3 owns coordination and wake traffic, and
Git owns repository history.

This is a thin skill. Exact flags live in the smallest `aops ... --help`
surface, the current `Agentspace User Guide` owns normative workflow, and the
running server schema owns raw hosted payloads.

## Start Here

1. Run `aops host health --json`.
2. Read the smallest nested help for the intended family.
3. Load a bounded brief/search result; do not preload every body.
4. Use metadata-only `skill ask`/`skill search` before opening skill content.
5. Preview guarded writes where supported, apply once, and read back the exact
   record.
6. Never hand-edit `.aops` mirrors as canonical state.

```bash
aops host health --json
aops mem brief --subject project --q "<current work>" --limit 5 --json
aops skill ask --q "<what the agent needs>" --limit 3 --json
```

## Canonical Sources

1. The installed `Agentspace User Guide`: `agentspace-user-guide.md` in the
   folder `aops assets status --json` reports as `userGuidesRoot`. Search it for
   the section you need instead of reading it whole.
2. Exact flags: `aops <family> <command> --help`.
3. Live discovery:
   `aops agent tools --domain agentspace --q "<capability>" --limit 20 --summary --json`.
4. Raw input contract:
   `aops agent schema --tool <agentspace-tool-id> --summary --json`.

## Retrieval Ladder

Use broad-to-specific reads:

```bash
aops mem brief --subject <kind> --q "<focus>" --limit 5 --json
aops mem synopsis --subject <kind> --q "<focus>" --limit 5 --json
aops mem search --subject <kind> --q "<terms>" --limit 8 --json
aops mem resume --subject <kind> --q "<focus>" --limit 8 --json
aops mem get --id <memory-id> --json
```

Memory is an evidence pack, not a transcript. Record purpose, canonical refs,
concrete outcome, validation/review evidence, open risk, and next action. Never
store secrets, raw logs, full documents, or chat transcripts.

### Required memory title

`title` is the memory note's own human-readable title; it is separate from a
linked subject's `subjectTitle` or `--label`. Every create path must supply a
nonblank title: `mem write`, `mem checkpoint`, and `mem summary` use
`--title <title>`; `mem compact` requires `--title` when `--write-summary` is
selected; raw `memory-item.create` / `memory-item.add-memory-item` input uses
`data.title`. Updates may change it with `mem update --title`.

Agents must generate a concise title from the note's meaning before writing. Do
not use a UUID, shortened id, raw task code, or an empty placeholder as the
display title. If a write reports `memory_item_title_required` or asks for
`--title <title>`, generate the title and retry the same guarded write. Legacy
records are backfilled deterministically from the first meaningful content line,
with a kind-based “Untitled … memory” fallback only when no meaningful line
exists.

## Checkpoint, Summary, and Retention

Use a checkpoint after a meaningful milestone, decision, blocker, or handoff:

```bash
aops mem checkpoint --project-slug <slug> --as milestone \
  --title "<concise milestone title>" --content '@checkpoint.md' \
  --task-id <task-id> --sprint-id <sprint-id> \
  --source-ref <ref> --validation-state "<evidence>" \
  --next-action "<next safe action>" --apply --json
```

Use `aops mem summary` at session end or an explicit summary point. Ordinary
continuation uses `mem checkpoint`. Durable closeout needs explicit closeout,
durable, and confirmation flags; inspect help first.

`mem compact` and `mem prune` are review-first retention tools. Run preview,
inspect the selected source set, and do not write summaries, mark source, prune,
or delete without explicit operator intent. Destructive pruning requires
`--apply --confirm`.

## Experience, Playbooks, and Hosted Assets

Capture experience only when it is reusable and evidence-backed:

```bash
aops exp search --project-slug <slug> --query "<need>" --limit 5 --json
aops exp capture --project-slug <slug> --type problem-solution \
  --title "<lesson>" --problem "<problem>" --solution "<verified solution>" \
  --pm-ref <projectman-ref> --source-ref <source-ref> --preview --json
aops playbook list --project-slug <slug> --area <area> \
  --review-state accepted --limit 5 --json
```

For prompts, resources, artifacts, and skills, start with summary inventory.
Prompt and skill bodies may be large:

```bash
aops prompt version list --summary --json
aops skill version list --summary --json
aops skill search --q "<capability>" --limit 5 --json
aops skill current --id <skill-id> --summary --json
aops resource list --summary --limit 10 --json
aops artifact ref list --summary --json
```

Read a selected `SKILL.md` completely before acting. Hosted prompt/skill
mirrors are read-only; use create/update/version/publish commands and verify the
new current version through the server.

## Profiles and Activity

Agent profiles compose role intent and prompt/skill/resource references. They
do not grant runtime permissions or operator approval. Activity is a read-first
ledger; use `activity list/get --summary` before loading raw payloads.

## Structured Discussion Boundary

Use `aops discuss` for material decisions needing independent stances and a
durable conclusion. Use `aops-cli-discuss` for the full ritual.

Important current semantics:

- `start --max-turns` is unsupported and fails;
- `--objective` is folded into the hosted question;
- `turn --to` supports only `operator`;
- `turn --reply-to` takes an integer turn sequence;
- waits exit `0` when the agent may write, `20` for an operator block, `21` for
  ready/terminal state, and `22` on timeout.

Discuss does not wake ChatV3 listeners. Use `aops-cli-chat` and `aops chat` for
coordination. Use `aops-cli-projectman` for the resulting plan/review truth.

## Schema Fallback

Do not guess hosted tool ids, payloads, or fixed tool counts:

```bash
aops agent tools --domain agentspace --q "<capability>" \
  --limit 20 --summary --json
aops agent schema --tool <agentspace-tool-id> --summary --json
aops agent invoke --tool <agentspace-tool-id> \
  --input '@payload.json' --preview --json
```

Use raw invoke only when sugar is insufficient and the exact schema is known.

## Done When

- canonical records were read back after mutation;
- memory contains bounded evidence rather than duplicated truth;
- relevant help/guide/schema sources agree;
- destructive retention actions, if any, had explicit operator authority;
- any material decision, plan, review, document, or wake traffic is stored in
  its actual owner rather than Agentspace by convenience.
