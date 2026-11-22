---
name: aops
version: 18
description: "AOPS skill index and core rules: which AOPS skill to load for aops pm (project management and plans), aops mem (memory), aops doc (documents), aops chat (coordination), aops discuss (decisions), work with other agents, read-only views and plain-language status briefs. Load aops-interactive first when the user is new to AOPS or the repository is not linked yet. Triggers: 'aops', 'which aops skill', 'aops pm', 'aops plan', 'aops mem', 'aops doc', 'aops chat', 'aops discuss', 'aops collab'."
metadata:
  supersedes: "v17"
---

# AOPS skill index

AOPS (Agentic Operations System) keeps an agent's work on an AOPS server: plans and reviews (aops pm), memory notes (aops mem), versioned documents (aops doc), coordination rooms (aops chat) and decision topics (aops discuss). Agents use the `aops` command; people follow the same records in the Cockpit, at the server address with `/ui`.

This skill is a **thin index** with the core rules of every AOPS session. It names the skill that carries the workflow; load that skill next. `--help` wins over skill text for commands and flags; the user guides win for workflow.

## Core rules

1. The repository's rules and the user's own rules come first. Roles are what the user says they are.
2. Accepting a review authorizes no commit, push, publication or deletion, and no closing of a board or a room; each needs its own approval.
3. Before you leave a session, leave the records true to the work and write a short handoff note: what is done, what was checked, what comes next.

## Starting work

When the user is new to AOPS, or the repository is not linked to an AOPS project yet, load **`aops-interactive`**: a short kickoff for the project, the aops pm, aops mem and aops doc defaults, and working alone or with other agents. When the request already says how to work, go straight to the matching skill below. A review request left for later wakes nobody; it waits for a later reviewer.

## Which skill for what

| Need | Skill |
|---|---|
| Kickoff for a user new to AOPS or a repository not linked yet | `aops-interactive` |
| aops pm: boards, tasks, sprint plans, small steps, issues, feedback, review requests and results | `aops-cli-projectman` |
| aops mem: kickoff, checkpoint, decision, blocker and handoff notes; experience, prompts, skills | `aops-cli-agentspace` |
| aops doc: versioned documents, sections and pages, search, publish | `aops-cli-docman` |
| aops chat: rooms, room codes, messages, presence | `aops-cli-chat` |
| aops discuss: a decision topic with independent turns and a recorded consensus | `aops-cli-discuss` |
| Working alone, with review requests left for a later session | `aops-cli-projectman` |
| Working with other agents: roles, decisions, plan reviews and reviewed slices, in a room, tmux or another session | `aops-collaborative-work` |
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
