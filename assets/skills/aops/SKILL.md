---
name: aops
version: 16
description: "AOPS skill index: which AOPS skill to load for aops pm (project management), aops mem (memory), aops doc (documents), aops chat (coordination), aops discuss (decisions), working disciplines, read-only views and plain-language status briefs. Start new work with aops-interactive. Triggers: 'aops', 'which aops skill', 'aops pm', 'aops mem', 'aops doc', 'aops chat', 'aops discuss'."
metadata:
  supersedes: "v15"
---

# AOPS skill index

AOPS (Agentic Operations System) keeps an agent's work on an AOPS server: plans and reviews (aops pm), memory notes (aops mem), versioned documents (aops doc), coordination rooms (aops chat) and decision topics (aops discuss). Agents use the `aops` command; people follow the same records in the Cockpit, at the server address with `/ui`.

This skill is a **thin index**. It names the skill that carries the workflow; load that skill next.

- `--help` wins over skill text for commands and flags; the user guides win for workflow.
- Board and room closeout belong to the user. End a turn with a checkpoint or handoff, not with closeout.

## Starting work

When a user starts a task with AOPS, says only "use AOPS", or does not know AOPS yet, load **`aops-interactive`**. It runs a short kickoff interview (project, aops pm, aops mem and aops doc defaults, working alone or with a second agent, review requests, discuss topics) and then continues with the skills below. When the request already answers those questions, go straight to the matching skill.

## Which skill for what

| Need | Skill |
|---|---|
| Kickoff interview; how to use AOPS | `aops-interactive` |
| aops pm: boards, tasks, sprint plans, small steps, issues, feedback, review requests and results | `aops-cli-projectman` |
| aops mem: kickoff, checkpoint, decision, blocker and handoff notes; experience, prompts, skills | `aops-cli-agentspace` |
| aops doc: versioned documents, sections and pages, search, publish | `aops-cli-docman` |
| aops chat: rooms, room codes, messages, presence | `aops-cli-chat` |
| aops discuss: a decision topic with independent turns and a recorded consensus | `aops-cli-discuss` |
| Working alone: the solo rhythm and its review requests | `aops-working-disciplines` |
| Working with a second agent, in tmux or an aops chat room | `aops-collaborative-work` |
| Read-only views of plans, memory and documents | `aops-cli-view` |
| A plain-language status brief for a person | `aops-cli-operator-brief` |
| Command conventions: targets, guard flags, schemas, the raw tool fallback | `aops-cli-core` |

## Records on the server, mirrors on disk

- The AOPS server holds the truth: plans, memory notes, discussion topics, documents, prompts and skills. Write them through `aops` commands.
- `.aops-cache/**` in a repository is a read-only mirror of server records. Never edit it as truth; refresh it with `aops sync pull --apply --hosted-project-slug <slug> --json` (prompts and skills) or `aops doc mirror pull --project-slug <slug> --apply --json` (documents).
- Before a hosted write you have not done before, read its input schema: `aops agent schema --tool <domain>.<operation> --summary --json`.

## User guides

The AOPS user guides are installed with these skills. `aops assets status --json` reports their folder as `userGuidesRoot`. Search a project's documents by content:

```bash
aops doc scope search --project-slug <slug> --q "<keyword>" --json
aops doc search --document-version-id <version-id> --q "<keyword>" --json
```

Refer to a guide section by the guide's title and the section's name, not by its number alone.
