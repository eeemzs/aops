---
name: aops-interactive
version: 8
description: "Start AOPS work with a short kickoff: link or create the AOPS project, confirm the aops pm, aops mem and aops doc defaults, and choose working alone or with other agents; then continue with the matching AOPS skill. Use when a user is new to AOPS, asks how to use AOPS, or starts in a repository not linked to AOPS yet."
metadata:
  supersedes: "v7"
  short-description: "AOPS kickoff for a new user or a new repository"
  tags:
    - aops
    - interactive
    - kickoff
    - onboarding
---

# AOPS Interactive

A short **kickoff** before the work starts, for a user new to AOPS or a repository not linked yet. Every answer decides how you work, and tells the user what AOPS will do for them.

- Ask only what the user's message, the repository rules or an earlier session leave open. The repository's rules and the user's own rules come first: when they require plans, reviews or discuss topics, say so instead of offering to switch them off.
- Give every question its default, so "continue with the defaults" settles the kickoff at once.
- Ask two or three questions at a time, in the user's language. Use your own question interface (multiple choice) when you have one.
- When the user does not know AOPS, say what each answer changes. Avoid AOPS jargon such as board, UUID or RR.
- Start no agent, room or reviewer without the user's yes.

## 1. Read the state quietly

```bash
aops host health --json
aops project links list --json
```

When the repository is linked, also read where the work stands:

```bash
aops pm resume --project-slug <slug> --for <your-agent-id> --limit 10 --json
aops mem brief --project-slug <slug> --json
```

When the server does not answer, read the error and the target before you explain. AOPS may not be running on this computer, or a remote server, the network or TLS may be at fault. Tell the user what you found and the safe next step; stop the kickoff there.

## 2. Goal and project

1. When the user has not said what to finish in this session, ask: "What shall we finish in this session?"
2. **Linked:** name the project: "This repository uses AOPS project slug:<slug>."
3. **Not linked**, for a new or an existing repository: look for a matching project (`aops project list --json`), then ask: "This repository is not linked to AOPS yet. Shall I link it to project slug:<suggestion>?" Suggest the matching project, or a short new slug from the folder or repository name: lowercase, two or three words, hyphens. The user may give another name.
   - For a new project, create it first: `aops project create --name "<name>" --slug <slug> --apply --json`.
   - Then link the repository with the project's real id: `aops init --project-id <id> --project-slug <slug> --project-name "<name>" --json`. Never overwrite an existing AOPS configuration (`--force`).

## 3. pm, mem and doc

AOPS keeps the work in three places by default. Say what each one does in one sentence, then ask:

1. **aops pm:** "Shall I track this work in aops pm, as a task with a plan when the work has several steps?" Default: yes. Small work gets a task, not a plan.
2. **aops mem:** "Shall I write checkpoint notes at important points: decisions, finished steps, blockers and handovers?" Default: yes.
3. **aops doc:** no question. Say: "When this work needs a document, it goes to aops doc." A file the user asks for at a specific place stays there.

## 4. Alone or with other agents

Ask: "Shall I work alone, or with other agents?" Default: alone.

- **Alone:** with aops pm on, ask: "Shall I leave a review request after each finished piece, so another agent can check it later?" Default: yes. A review request starts no agent; it waits for a later reviewer.
- **With other agents:** load `aops-collaborative-work` and ask its two setup questions now: the role of each agent, unless the user gave them, and where the agents meet (an aops chat room, tmux on this computer, or another session the user opens).
## 5. Summary and start

Summarize the setup in one short paragraph: the project; what goes to aops pm, aops mem and aops doc; alone, or with whom, where and in which roles; whether review requests are used; and the rules that came from the repository or from the user. Add that an important decision that is costly to reverse can be settled first in an aops discuss topic with at least two independent agents, when the user wants it (`aops-cli-discuss`). Say where the user will see the work in the Cockpit (the server address with `/ui`): tasks and plans under PM, notes under Agentspace → Memory, documents under Docs, the room under Chat.

Then start without asking for the same answers again; they hold for the whole session. Continue with the matching skill:

| Choice | Skill |
|---|---|
| Tasks, plans, issues, review requests; working alone | `aops-cli-projectman` (aops pm) |
| Memory notes | `aops-cli-agentspace` (aops mem) |
| Documents | `aops-cli-docman` (aops doc) |
| Other agents: roles, plan reviews, reviewed slices | `aops-collaborative-work`, with `aops-cli-chat` (aops chat) for a room |
| Discuss topic | `aops-cli-discuss` (aops discuss) |

Done when the user has the summary and the first AOPS record exists: the link, the task or the plan. Or the user chose to work without aops pm.

## Skills on this computer

AOPS skills are installed with AOPS. Newer skills arrive with `aops assets update`, or with Check and Update in the Skills row of the AOPS window. Run it only when the user asks.
