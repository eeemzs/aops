---
name: aops-cli-agentspace
description: "Use when an AI agent needs the AOPS CLI Agentspace playbook for token-efficient memory and resume, checkpoints and summaries, experience and playbooks, reusable hosted assets, agent profiles, activity, or structured discussions."
metadata:
  version: "28"
  supersedes: "v27"
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

1. Hosted `Agentspace User Guide` in project `slug:aops`, group
   `domain-guides`, document `agentspace-user-guide`, current v20
   `9c805e63-3372-4a8c-a3dc-adfa5d4b8577`.
2. Targeted read-only mirror:
   `.aops/docman/domain-guides/agentspace-user-guide.md`.
3. Public release source:
   `assets/docs/user-guides/agentspace-user-guide.md`.
4. Exact flags: `aops <family> <command> --help`.
5. Live discovery:
   `aops agent tools --domain agentspace --q "<capability>" --limit 20 --summary --json`.
6. Raw input contract:
   `aops agent schema --tool <agentspace-tool-id> --summary --json`.

When freshness matters, pull only the intended guide and search its mirror:

```bash
aops doc mirror pull --project-slug aops \
  --document-slug agentspace-user-guide --apply --json
aops doc scope search --project-slug aops \
  --q "Agentspace <topic>" --local --json
aops view doc-page agentspace-user-guide#<section-slug> --max-bytes 6000
```

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

## Checkpoint, Summary, and Retention

Use a checkpoint after a meaningful milestone, decision, blocker, or handoff:

```bash
aops mem checkpoint --project-slug <slug> --as milestone \
  --content '@checkpoint.md' --task-id <task-id> --sprint-id <sprint-id> \
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
aops exp search --project-slug <slug> --q "<need>" --limit 5 --json
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
