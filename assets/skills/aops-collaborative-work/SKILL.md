---
name: aops-collaborative-work
version: 18
description: "Work on an AOPS project with other agents: roles the user gives and may change, aops discuss topics for important decisions, an aops pm plan that another agent reviews, a review request for every slice, and coordination in an aops chat room, tmux or another session the user opened. Use when the user asks for a reviewer, a peer, a plan review, a discuss topic or a room, or when the kickoff (aops-interactive) chose other agents. Triggers: 'aops collab'."
metadata:
  supersedes: "v17"
  short-description: "AOPS work with other agents: roles, discuss, plan review, reviewed slices"
  tags:
    - aops
    - collaboration
    - multi-agent
    - review
    - discuss
    - chat
    - tmux
---

# AOPS Collaborative Work

Several agents work on one task: they decide, plan, build and review in the roles the user gives. AOPS keeps the record:
- aops discuss holds decisions;
- aops pm holds the plan, the review requests and their results;
- aops mem holds checkpoints and handoff notes;
- a chat room, tmux or another session carries only coordination and wake-ups.

When `--help`, a tool schema or a user guide disagrees with this skill, they win.

## Roles

- The user gives the roles, at the time. Driver, planner, implementer, reviewer and coordinator are plain words with no files behind them: the driver runs a discuss topic and writes its consensus, the planner writes the plan, the implementer builds, the reviewer checks plans and slices, and a coordinator directs other agents and speaks for them to the user.
- Roles may change as the work moves on; follow the latest the user gave. One agent may hold several roles, and several agents may share one.
- When the user leaves the choice to the agents ("one of you reviews"), they settle it among themselves: the agent that builds a slice does not review it, and they tell the user who took which role. When a role the work needs was not given at all, ask the user once. Never derive a role from an agent's name or brand.
- Other questions go first to the references and the other agents, or to a discuss topic. A decision only the user can make waits for the user (as aops pm feedback when aops pm is on) while other work goes on.

## Where the agents meet

- **aops chat room**, for several agents, other computers, or a person following along: use the user's room code. The user can also create a room in the Cockpit under Chat, or you create one with the user's yes (`aops-cli-chat`). Each agent joins with its own saved session; bind the discuss topic, the task and the review requests to the room.
- **tmux**, for an agent on this computer: check that tmux and the agent's command are available, and that no session with that name runs. With the user's yes, start a persistent, interactive session, not a one-shot run. Give it a short brief file (the project, the task, its role, the record ids, how you will wake it), and after each message confirm that the pane shows it working.
- **Another session** the user opened, or one you start with the user's yes (a desktop app, an IDE): it reads the same records. Wake it the way the user set up, or ask the user to pass the message on.

Rely on another agent only after it answered. Chat and tmux only coordinate and wake: decisions stay in aops discuss, plans and reviews in aops pm.

## Decide: aops discuss

An important decision that is costly to reverse, or one the user asks to discuss, goes to an aops discuss topic before anything is built. The driver opens it with the other agents; each researches on its own before its first turn. The turn rules (at least four turns, a final stance from each agent, conclude) are in `aops-cli-discuss`. At least two real, independent agents take part; one agent writing under two names is not a consensus. Where the repository requires the user's approval of a consensus, the work waits for it.

## Plan: aops pm

The planner writes the plan in aops pm from the consensus, or from the request when there was no topic: a task, and for several steps a plan with phases and small steps, each saying what, why and when it is done (`aops-cli-projectman`). Another agent reviews the plan through a review request; building starts when the review is accepted.

## Build and review in slices

1. The implementer builds one slice, validates it, writes a checkpoint when checkpoint notes are on, and opens a review request for each reviewer. Scope it to its phase (`sprint-phase:<phase-id>/<phase-label>`, the phase id from the plan) or to the whole plan (`sprint:<sprint-id>`):

   ```bash
   aops pm review-request create --task <task-id> --sprint <sprint-id> \
     --review-scope "sprint-phase:<phase-id>/<phase-label>" --target-agent <reviewer> \
     --title "<slice>" --apply --json
   ```

2. Wake each reviewer where it works, with the review request id (`REVIEW READY` below).
3. The reviewer records one outcome on the review request: approved, changes_requested, commented or blocked. Material findings become issues linked to it (`aops pm issue create --source review --review-request <id> ...`); fixes come back in a child review request (`--parent <id>`).
4. The implementer never approves its own slice. The reviewer looks at the code and at evidence it ran itself, not at status text, and writes the result under its own identity.
5. The reviewer keeps a bounded slice bounded: what lies outside it becomes an issue or feedback, not a hardening program.
6. A room message or a tmux reply is never the review record; read the result on the review request. Report only validation you ran in this session. An accepted review is no permission to commit, push or publish: those follow the repository's and the user's rules (`aops`, Core rules).

## Coordination lines

Messages in a room, a tmux pane or another session stay short:

```text
TASK <n>: <scope>                                     # user or coordinator
ACK <n>: <goal, acceptance, validation plan>          # the agent taking it
TURN: topic=<slug>; next=<agent>                      # a discuss turn was written
PLAN READY: RR=<id> | plan=<sprint>; phases=<list>    # planner
REVIEW READY: RR=<id> | slice=<name>; scope=<phase or files>; evidence=<refs>
DONE <n>: <summary, files, evidence>                  # implementer
VERIFIED <n>: <evidence> | REWORK <n>: <issue>        # reviewer, mirroring the result
CHECKPOINT: subject=<task or sprint>; memory=<id>; next=<action>
```

In a room, read from your saved cursor and mark messages delivered and read; in `listen`, exit 22 is a normal timeout (`aops-cli-chat`).

## Without aops pm

- Tell the user that plans and reviews then stay in the conversation with no record; ask once whether to turn aops pm on, and do not turn it on yourself.
- The plan is a short list in the conversation; the reviewer answers there with the outcome and each finding's severity and evidence.
- The implementer keeps the open findings (in a checkpoint when checkpoint notes are on); fixes come back as a new slice in the same place.
