# Agentspace User Guide

_Release Notes:_ Replaces stale Turkish and retired hosted-chat material; adds current memory, experience, playbook, skill discovery, agent-profile, activity, and Discuss semantics.

## 1 Ownership model

### 1.1 Overview

Agentspace is the durable context and reusable knowledge plane for AOPS agents.
It owns memory, experience, playbooks, prompts, resources, artifacts, skills,
agent profiles, activity records, and structured discussion topics. The local
AOPS server is canonical; repository mirrors are read-only projections.

Use each system for the state it owns:

| Need | Canonical owner |
| --- | --- |
| Durable context, reusable knowledge, profiles, and structured discussion | Agentspace |
| Tasks, sprints, issues, review requests, and review results | Projectman |
| Versioned documents, pages, publication, and document retrieval | Docman |
| Encrypted coordination, room messages, cursors, and wake traffic | ChatV3 through `aops chat` |
| Repository files, commits, and branches | Git and the source repository |

Memory is not a second task tracker. Chat is not a decision record. A generated
synopsis is not an independent source of truth. Link the canonical record
instead of copying it into every system.

## 2 Start with the smallest useful read

### 2.1 Overview

Check health and load a bounded context pack before reading full records:

```bash
aops host health --json
aops mem brief --subject project --q "<current work>" --limit 5 --json
aops mem synopsis --subject project --q "<current work>" --limit 5 --json
aops mem search --subject project --q "<decision or constraint>" --limit 8 --json
```

Then resolve only the relevant full item:

```bash
aops mem resume --subject project --q "<current work>" --limit 8 --json
aops mem get --id <memory-id> --json
```

Do not preload all memory, skill bodies, activity payloads, or discussion
history. Prefer summary reads and small limits. Use nested `--help` when flags
or safety semantics matter.

## 3 Durable memory

### 3.1 What belongs in memory

#### 3.1.1 Overview

A useful memory item is an evidence pack, not a transcript. Include:

1. request or purpose;
2. Projectman, document, discussion, file, or commit references;
3. concrete outcome and current state;
4. validation or review evidence;
5. open risk and the next safe action.

Do not store credentials, tokens, private keys, raw logs, full documents, or
large chat transcripts.

Short durability is the normal rolling context. Use durable memory only for a
stable, reviewed decision, rule, architecture constraint, or closeout record.
For a lower-level write, inspect the exact contract first and keep the same
evidence shape:

```bash
aops mem write --help
aops mem write --project-slug <slug> --mode decision \
  --content '@decision.md' --task-id <task-id> \
  --source-ref <evidence-ref> --apply --json
```

### 3.2 Checkpoint and summary

#### 3.2.1 Overview

Write a short checkpoint after a meaningful milestone, decision, blocker, or
handoff point:

```bash
aops mem checkpoint \
  --project-slug <slug> \
  --as milestone \
  --content '@checkpoint.md' \
  --task-id <task-id> \
  --sprint-id <sprint-id> \
  --source-ref <ref> \
  --validation-state "<evidence summary>" \
  --next-action "<next safe action>" \
  --apply --json
```

Use `aops mem summary` at session end or at an explicit summary point. Ordinary
continuation should use `mem checkpoint`. Durable closeout is intentionally
stronger and requires the explicit closeout, durable, and confirmation flags;
inspect `aops mem summary --help` before using it.

Multiline or quote-heavy content should use `@file`. Relative `@file` paths are
resolved from the current working directory.

### 3.3 Compact and prune

#### 3.3.1 Overview

Compaction is a review workflow, not automatic deletion:

```bash
aops mem compact --project-slug <slug> --older-than-days 30 \
  --keep-latest 20 --max-items 20 --preview --json
aops mem prune --project-slug <slug> --older-than-days 30 \
  --keep-latest 20 --preview --json
```

Review the proposed source set and compact summary before writing it. Writing a
summary, marking source items, pruning source items, and deleting memory are
separate actions. Destructive pruning requires explicit operator intent plus
`--apply --confirm`.

## 4 Projects, experience, and playbooks

### 4.1 Overview

Agentspace projects provide reusable-context ownership; they do not replace
Projectman work tracking.

```bash
aops project list --json
aops project get --id <project-id> --json
```

Capture an experience only when it is reusable beyond the current task. Include
the problem, solution, evidence references, and realistic confidence:

```bash
aops exp capture --project-slug <slug> \
  --type problem-solution \
  --title "<reusable lesson>" \
  --problem "<observed problem>" \
  --solution "<verified solution>" \
  --pm-ref <projectman-ref> \
  --source-ref <source-ref> \
  --preview --json
```

Apply only after reviewing the preview. Use bounded discovery for existing
experience and reviewed playbook projections:

```bash
aops exp search --project-slug <slug> --q "<need>" --limit 5 --json
aops playbook list --project-slug <slug> --area <area> \
  --review-state accepted --limit 5 --json
```

Promoting experience or memory into a playbook is a reviewed knowledge action,
not a side effect of finishing a task.

## 5 Reusable hosted assets

### 5.1 Overview

Prompts and skills have version lifecycles. Resources are typed knowledge
pointers. Artifacts describe or reference outputs; they are not an unlimited
binary store.

Use summary inventory first:

```bash
aops prompt version list --summary --json
aops skill version list --summary --json
aops resource list --summary --limit 10 --json
aops artifact ref list --summary --json
```

For skill routing, search metadata before loading bodies:

```bash
aops skill ask --q "<what the agent needs>" --limit 3 --json
aops skill search --q "<capability>" --limit 5 --json
aops skill current --id <skill-id> --summary --json
aops skill inspect --id <skill-id> --summary --json
```

`skill ask` is deterministic metadata ranking; it does not load every skill
body or run an LLM. Read the selected `SKILL.md` completely before acting.

Create, update, version, publish, link, and delete operations are guarded
writes. Inspect the exact nested help, preview where supported, apply once, and
read back the resulting record/version. Never hand-edit hosted mirrors.

## 6 Agent profiles and activity

### 6.1 Overview

An agent profile composes role intent and references to prompts, skills,
resources, overlays, and additional context. It is not a substitute for runtime
permissions or operator approval.

```bash
aops agent-profile list --project-slug <slug> --json
aops agent-profile get --project-slug <slug> --id <profile-id> --json
aops agent-profile create --project-slug <slug> \
  --name "<profile name>" --role reviewer \
  --skill-ref <skill-ref> --resource-ref <resource-ref> \
  --preview --json
```

Activity is a read-oriented ledger. Start with compact records and request the
full payload only when evidence inspection needs it:

```bash
aops activity list --project-slug <slug> --summary --limit 20 --json
aops activity get --project-slug <slug> --id <activity-id> --summary --json
```

## 7 Structured discussions

### 7.1 Overview

Use `aops discuss` when a material decision benefits from independent stances
and a durable conclusion. Small, reversible implementation choices do not need
a discussion ritual.

```bash
aops discuss start --project-slug <slug> \
  --title "<decision>" \
  --question "<one precise question>" \
  --agent <agent-a> --agent <agent-b> \
  --apply --json

aops discuss turn --project-slug <slug> \
  --topic <topic-id-or-slug> --agent <agent-a> \
  --kind statement --from-file ./stance.md --apply --json

aops discuss wait --project-slug <slug> \
  --topic <topic-id-or-slug> --agent <agent-b> \
  --timeout-sec 540 --interval-sec 5 --json

aops discuss turn --project-slug <slug> \
  --topic <topic-id-or-slug> --agent <agent-b> \
  --kind final-stance --from-file ./final.md --apply --json

aops discuss conclude --project-slug <slug> \
  --topic <topic-id-or-slug> --apply --json
```

The current server-first contract is intentionally explicit:

- `discuss start --max-turns` is unsupported and fails;
- `--objective` is folded into the hosted question; it is not a separate
  hosted column;
- `discuss turn --to` supports only `operator`; a named agent fails;
- `discuss turn --reply-to` takes an integer turn sequence, not a turn id;
- turns are append-only and server-enforced ordering remains authoritative;
- `digest`, `lineage`, `output`, and `abandon` are the supported inspection and
  terminal helpers; inspect their nested help before use.

`aops discuss wait` exits with `0` when the requested agent may write, `20` for
an operator-addressed open question, `21` when the topic is terminal or ready
to conclude, and `22` on timeout.

Discuss does not replace ChatV3 wake traffic. When another agent must act, send
a short `aops chat` message containing the canonical topic, turn, PM, or review
reference. ChatV3 foreground listen uses its own contract: `0` for delivered
work and `22` for a quiet bounded interval. Use the ChatV3 skill and guide for
room, session, receipt, and cursor behavior.

## 8 Discovery, safety, and troubleshooting

### 8.1 Live capability discovery

#### 8.1.1 Overview

When sugar does not cover an operation, discover the running server instead of
guessing a tool id or payload:

```bash
aops agent tools --domain agentspace --q "<capability>" \
  --limit 20 --summary --json
aops agent schema --tool <agentspace-tool-id> --summary --json
aops agent invoke --tool <agentspace-tool-id> \
  --input '@payload.json' --preview --json
```

The hosted catalog may change. This guide deliberately does not freeze a tool
count or a tool-id inventory.

### 8.2 Guide retrieval

#### 8.2.1 Overview

Use Docman retrieval rather than scanning every document:

```bash
aops doc scope search --project-slug aops \
  --q "Agentspace memory checkpoint" --local --json
aops doc search --document-version-id <version-id> \
  --q "structured discussions" --local --json
aops doc outline get --document-version-id <version-id> \
  --titles-only --depth 1 --json
aops view doc-page agentspace-user-guide#<section-slug> --max-bytes 6000
```

### 8.3 Common mistakes

#### 8.3.1 Overview

- putting task or review truth in memory instead of Projectman;
- copying a full transcript into memory instead of recording evidence and refs;
- loading every skill body before metadata search;
- mutating `.aops` mirrors by hand;
- treating a quiet wait/listen timeout as completion;
- inventing another participant's discussion stance;
- running compact/prune/delete writes without operator intent;
- guessing hosted tool ids, schemas, or fixed catalog counts.

## 9 Appendices

### 9.1 Overview

<!-- aops-generated:agentspace-command-catalog:start -->

### 9.2 Generated Agentspace command families

#### 9.2.1 Overview

> This compact appendix is generated from the public Commander registrations. It groups current leaf commands into eleven owner families instead of copying a large command-by-command or hosted-tool catalog. Regenerate it with `aops docs user-guide --guide agentspace`.

| Family | Registered leaf commands | Purpose |
| --- | --- | --- |
| `aops mem` (aliases: memory) | `brief`, `checkpoint`, `compact`, `delete`, `doc answer`, `doc publish`, `doc refs`, `doc source`, `get`, `list`, `prune`, `resume`, `search`, `summary`, `synopsis`, `update`, `write` | Agentspace server-first memory commands (hosted memory-item ops are the source of truth; the local .aops memory tree is never written) |
| `aops exp` (aliases: experience) | `capture`, `delete`, `get`, `list`, `promote`, `search`, `update` | Author and read hosted Agentspace experience items (server-first) |
| `aops project` | `create`, `delete`, `get`, `link`, `links list`, `list`, `migrate-local-root`, `update` | Agentspace project sugar commands over the hosted AOPS gateway |
| `aops playbook` | `get`, `list`, `project-set`, `promote`, `show` | List reviewed playbooks projected from hosted Agentspace memory rules/constraints (server-first) |
| `aops prompt` | `create`, `current`, `delete`, `get`, `inspect`, `list`, `update`, `version create`, `version delete`, `version get`, `version list`, `version publish`, `version update` | Agentspace prompt and prompt-version sugar commands over the hosted AOPS gateway |
| `aops resource` | `create`, `delete`, `get`, `list`, `update` | Agentspace resource sugar commands over the hosted AOPS gateway |
| `aops artifact` | `create`, `delete`, `get`, `link`, `ref list` | Agentspace artifact sugar commands over the hosted AOPS gateway |
| `aops skill` | `ask`, `create`, `current`, `delete`, `get`, `inspect`, `list`, `search`, `update`, `version create`, `version delete`, `version get`, `version list`, `version publish`, `version update` | Agentspace skill and skill-version sugar commands over the hosted AOPS gateway |
| `aops agent-profile` | `create`, `delete`, `get`, `list`, `update` | Author and read hosted Agentspace agent profiles (server-first) |
| `aops activity` | `get`, `list` | Agentspace activity ledger read sugar over the hosted AOPS gateway |
| `aops discuss` | `abandon`, `conclude`, `digest`, `follow-up`, `fork`, `get`, `lineage`, `list`, `loop-prompt`, `output`, `start`, `status`, `turn`, `wait` | Manage hosted (server-first) Agentspace discussion topics |

<!-- aops-generated:agentspace-command-catalog:end -->

<!-- aops-generated:agentspace-discovery:start -->

### 9.3 Generated Agentspace discovery and routing guide

#### 9.3.1 Overview

> Sugar commands cover common Agentspace workflows. The running server catalog remains authoritative for hosted operations, so this guide does not freeze tool ids or tool counts.

| Command | Purpose |
| --- | --- |
| `aops skill search` | Find current published hosted skills by bounded deterministic metadata ranking |
| `aops skill ask` | Recommend hosted skills using one deterministic metadata search (no LLM or body loading) |
| `aops agent tools` | List federated tools from the canonical operator plane (/api/agent/tools) |
| `aops agent schema` | Print the live JSON Schema for one tool's input contract — use this before authoring --input payloads |
| `aops agent invoke` | Invoke a tool via the canonical operator plane (/api/agent/tools/{toolId}/invoke) |

Use the smallest useful read:

```bash
aops skill ask --q "<need>" --limit 3 --json
aops agent tools --domain agentspace --q "<capability>" --limit 20 --summary --json
aops agent schema --tool <agentspace-tool-id> --summary --json
```

| Need | Specialized route |
| --- | --- |
| Encrypted coordination and wake | `aops chat` and the `aops-cli-chat` skill |
| Structured decision and consensus | `aops discuss` and the `aops-cli-discuss` skill |
| Tasks, sprints, issues, and review truth | `aops pm` and the `aops-cli-projectman` skill |
| Versioned documents and retrieval | `aops doc` and the `aops-cli-docman` skill |
| File snapshots and lineage | `aops agent tools --domain fileman` and the `aops-cli-fileman` skill |

<!-- aops-generated:agentspace-discovery:end -->
