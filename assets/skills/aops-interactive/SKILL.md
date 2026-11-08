---
name: aops-interactive
version: 7
description: "Start AOPS work with a short kickoff interview: link or create the AOPS project, confirm the aops pm, aops mem and aops doc defaults, choose working alone or with a second agent (tmux or an aops chat room), and decide on review requests and discuss topics; then continue with the matching AOPS skill. Use when a user starts a task with AOPS, asks how to use AOPS, or does not know AOPS yet."
metadata:
  supersedes: "v6"
  short-description: "AOPS kickoff interview and router"
  tags:
    - aops
    - interactive
    - kickoff
    - onboarding
    - router
---

# AOPS Interactive

Run a **kickoff interview** before the work starts. Every answer decides how you work, and tells the user what AOPS will do for them.

- Ask only what the user's message, the repository rules or an earlier session have not answered already.
- Give every question its default, so "continue with the defaults" settles the interview at once.
- Ask in small groups of two or three, in the user's language. Use your own question interface (multiple choice) when you have one.
- When the user does not know AOPS, say in each question what the answer changes. Avoid AOPS jargon such as board, UUID or RR.
- Repository rules come first. When the repository requires plans, reviews or discuss topics, say so instead of offering to switch them off.
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

When the server does not answer, read the error and the target before you explain. AOPS may not be running on this computer, or a remote server, the network or TLS may be at fault. Tell the user what you found and the safe next step; stop the interview there.

Done when you know that the server answers, which project this repository uses if any, and what is already in progress.

## 2. Goal

When the user has not said what to achieve in this session, ask: "What shall we finish in this session?"

## 3. Project

- **Linked:** name the project: "This repository uses AOPS project slug:<slug>."
- **Not linked**, for a new or an existing repository:
  1. Look for a matching project on the server: `aops project list --json`.
  2. Ask: "This repository is not linked to AOPS yet. Shall I link it to project slug:<suggestion>?" Suggest the matching project, or a short new slug from the folder or repository name: lowercase, two or three words, hyphens. The user may give another name.
  3. For a new project, create it first: `aops project create --name "<name>" --slug <slug> --apply --json`.
  4. Then link the repository with the project's real id: `aops init --project-id <id> --project-slug <slug> --project-name "<name>" --json`.
  Never overwrite an existing AOPS configuration (`--force`).

Done when the repository is linked to a project the user confirmed.

## 4. pm, mem and doc

AOPS keeps the work in three places by default. Say what each one does in one sentence, then ask:

1. **aops pm:** "Shall I track this work in aops pm, as a task with a plan when the work has several steps?" Default: yes. Small work gets a task, not a sprint.
2. **aops mem:** "Shall I write checkpoint memory notes at important points: decisions, finished steps, blockers and handovers?" Default: yes.
3. **aops doc:** no question. Say: "When this work needs a document, it goes to aops doc." A file the user asks for at a specific place stays there.

## 5. Working mode

**The user's own working discipline first.** When the user's request or the repository rules name one, or an installed skill says it is the user's working discipline for this work, follow it (`aops-working-disciplines`, "The user's own working discipline"). When it is unclear, or several could apply, ask once which one. From the rest of this step and step 6, ask only what it leaves open.

Otherwise ask: "Shall I work alone, or with a second agent?" Default: alone.

- **Alone:** the working discipline is solo-pm-loop (`aops-working-disciplines`).
  - When aops pm is on, ask: "Shall I leave a review request after each finished piece, so another agent can check it later?" Default: yes.
  - A review request starts no agent; it waits for a later review.
- **With a second agent, or more:** the discipline is `aops-collaborative-work`.
  1. Ask: "Where does the other agent work: shall I start it in a tmux session on this computer, or do the agents meet in an aops chat room?" Suggest tmux for one more agent on this computer; suggest a room for several agents, other computers, or when the user wants to follow along.
  2. **tmux:** check that tmux and the other agent's command are available, and that no matching session runs already. Ask which agent (Codex or Claude).
  3. **Chat room:** use the user's room code, or offer two ways to create a room. The user can open the Cockpit (the server address with `/ui`), go to Chat, and create a room and a room code. Or, with the user's yes, you create it through `aops-cli-chat`.
  4. Roles: use the roles the user gave. Otherwise ask; default: you build, the other agent reviews.

## 6. Important decisions

Ask: "When a decision is important and costly to reverse, such as a design choice, shall we settle it first in an aops discuss topic?" Suggest yes when the task contains such a decision.

A discuss topic needs two real, independent agents. When step 5 chose alone, yes moves the work to collaborative work: ask the tmux or chat room question from step 5 now.

## 7. Summary and start

Summarize the setup in one short paragraph:
- the project;
- what goes to aops pm, aops mem and aops doc;
- the working discipline and where it came from (your request, the repository rules, your own skill, or the default);
- alone, or with whom and where, with the roles;
- whether review requests and discuss topics are used.

Say what the user will see in the Cockpit:
- tasks and plans under PM;
- notes under Agentspace → Memory;
- documents under Docs;
- the room under Chat.

Then start without asking for the same answers again. The answers hold for the whole session: the next skills follow them. `aops-working-disciplines` and `aops-collaborative-work` say how to work without aops pm or without checkpoint notes. Continue with the matching skill:

| Choice | Skill |
|---|---|
| Task, plan, issues, review requests | `aops-cli-projectman` (aops pm) |
| Memory notes | `aops-cli-agentspace` (aops mem) |
| Documents | `aops-cli-docman` (aops doc) |
| Working alone | `aops-working-disciplines` (solo-pm-loop) |
| Second agent | `aops-collaborative-work`, with `aops-cli-chat` (aops chat) for a room |
| Discuss topic | `aops-cli-discuss` (aops discuss) |

Done when the user has the summary and the first AOPS record exists: the task, the plan or the link. Or the user chose to work without aops pm.

## Skills on this computer

AOPS skills are installed with AOPS. Newer skills arrive with `aops assets update`, or with Check and Update in the Skills row of the AOPS window. Run it only when the user asks.
