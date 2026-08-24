---
name: aops-cli-mission
version: 3
description: "Use when an AOPS agent needs hosted Agentspace mission command discipline: create/list/get/update/resume, mission.policy seed via --policy-json, active Projectman implementation-plan ref, start --resume integration, and mission vs Projectman/memory ownership boundaries."
metadata:
  short-description: "AOPS CLI mission command guide"
  tags:
    - aops
    - cli
    - mission
    - agentspace
    - policy
    - resume
    - start
---

# AOPS CLI Mission

Use this skill when a task involves `aops-cli mission ...`, mission policy,
mission resume packs, or the mission/start bridge.

If this skill conflicts with `aops-cli mission --help`, `aops-cli start --help`,
or the Agentspace User Guide, those sources win.

## Ownership

Mission is Agentspace-owned durable intent and policy.

Projectman remains execution and review truth:

- kanban task
- sprint-backed implementation plan
- microtasks
- issues
- review requests and RRR

Memory remains durable carry-forward context. Chat remains coordination/wake.
Discuss remains material decision truth.

## Command map

| Need | Command |
| --- | --- |
| Create mission | `aops-cli mission create --objective "<text>" --policy-json '<json>' --apply --json` |
| List missions | `aops-cli mission list --summary --json` |
| Read one mission | `aops-cli mission get --id <id> --json` |
| Patch mission | `aops-cli mission update --id <id> --policy-json '<json>' --apply --json` |
| Resume mission | `aops-cli mission resume --id <id> --depth light --limit 8 --json` |
| Start from mission | `aops-cli start --resume <mission-id> --mode <mode> --board <board> --json` |
| Mid-session reminder | `aops-cli start --reminder --task "<current task>" --area <area> --json` |

Run live help before unfamiliar flags:

```bash
aops-cli mission --help
aops-cli mission create --help
aops-cli mission resume --help
aops-cli start --help
```

## Policy seed

`aops-cli start --json` emits:

- `result.mission.discipline`
- `result.mission.guardrails`
- `result.mission.policy`
- `result.mission.policyJson`
- `result.mission.startPack.policySeed`

Current policy seeds conventionally include these top-level groups:

- `discipline`
- `signalMapping`
- `guardrails`
- `review`
- `issue`
- `memory`
- `plan`
- `planning`
- `orchestration`
- `vocabBridge`

Use `policyJson` with mission create/update:

```bash
aops-cli mission create --objective "<objective>" --policy-json '<result.mission.policyJson>' --apply --json
aops-cli mission update --id <mission-id> --policy-json '<result.mission.policyJson>' --apply --json
```

The policy convention is documented in Docman guide "AOPS Working Disciplines"
and the `aops-working-disciplines` skill. It is advisory and free-form, not a
typed mission schema yet.

## Active plan ref

`--active-plan <sprint-id>` stores the active Projectman implementation-plan ref.
The plan id is the underlying sprint id.

Do not create a second mission-local plan model.

## Resume discipline

Default resume is compact and token-efficient:

```bash
aops-cli mission resume --id <mission-id> --json
aops-cli start --resume <mission-id> --json
```

The compact pack includes mission identity/status, active Projectman plan refs,
bounded PM/review/issue/memory refs, and the shared `sessionGuidance` pack.
Use `--full` only when the raw hosted skeleton is truly needed. For a running
session that only needs rules/playbook/experience refresh, use
`aops-cli start --reminder --task "<current task>" --area <area> --json`;
that path is read-only and does not mutate the mission.

## Boundaries and anti-patterns

1. Do not store task progress only in mission body or policy; Projectman owns
   execution truth.
2. Do not invent a mission handoff command in this slice; use Projectman handoff
   or `aops-cli mem summary`.
3. Do not use mission policy as a typed enum yet. It is free-form with the
   working-discipline convention.
4. Do not claim future mission helpers exist until live `aops-cli mission --help`
   exposes them. Use PM/RR/memory evidence today.
5. Do not hand-edit hosted mirrors after mission/skill/doc changes.
