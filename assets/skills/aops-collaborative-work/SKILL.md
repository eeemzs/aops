---
name: aops-collaborative-work
version: 17
description: "Work on an AOPS project with a second agent, or more: a reviewer or peer in a tmux session or an aops chat room, a review for every slice (a PM review request when aops pm is on), and aops discuss topics for important decisions. Use when the kickoff (aops-interactive) chose a second agent or a discuss topic, or when the user asks for a reviewer, a peer or a room."
metadata:
  supersedes: "v16"
  short-description: "Paired and multi-agent AOPS work: tmux or chat room, PM reviews, discuss"
  tags:
    - aops
    - collaboration
    - multi-agent
    - review
    - tmux
    - chat
    - discuss
---

# AOPS Collaborative Work

One agent builds and another reviews, or several agents coordinate. AOPS keeps the record of what the kickoff turned on:
- aops pm holds the task, the plan, the review requests and their results;
- aops discuss holds decisions;
- aops mem holds checkpoints;
- the tmux session or the chat room only carries the conversation.

Working alone follows `aops-working-disciplines` (solo-pm-loop) instead. When the user follows their own working discipline (`aops-working-disciplines`, "The user's own working discipline"), its rules decide reviews and decisions; use this skill only for the setup and mechanics it asks for. When `--help`, a tool schema or a user guide disagrees with this skill, they win.

## 1. Set up the pair

1. **Roles:** use the roles the user gave or confirmed at kickoff. Otherwise ask the user; suggest that you build (implementer) and the other agent reviews (reviewer). Never derive a role from an agent's name or brand.
2. **Where the other agent works:**
   - **tmux**, for one more agent on this computer:
     - Check that `tmux` and the other agent's command are available, and that no session with the chosen name already runs.
     - Start a persistent, interactive session, not a one-shot run.
     - Give the agent a short brief file: the project, the task, its role, the PM ids, and how you will signal it.
     - After each message, confirm that the pane shows the agent working.
   - **aops chat room**, for several agents, other computers, or a person following along:
     - Use the user's room code. Or, when the user agrees, create a room with `aops-cli-chat`.
     - Each agent joins with its own saved session. Bind the discuss topic to the room, and with aops pm on also the PM task and the review requests.
3. **Readiness:** rely on the other agent only after it answered: it acknowledged in the room, or its tmux pane replied.

## 2. Work in reviewed slices

The kickoff chose whether aops pm and checkpoint notes are on; follow those answers. Repository rules come first.

With aops pm on, review requests and their results are the review record. When the user chose to work without aops pm, tell them that reviews then stay in the conversation (the tmux session or the room) with no record. Ask once whether to turn aops pm on for reviews; do not turn it on yourself.

1. With aops pm on, the task exists in aops pm before implementation. Multi-step work also has a plan (`aops-cli-projectman`).
2. For each slice:
   1. Implement it and validate it honestly.
   2. With checkpoint notes on, write a checkpoint (`aops-cli-agentspace`).
   3. With aops pm on, open a review request:

      ```bash
      aops pm review-request create --task <task-id> --sprint <sprint-id> \
        --review-scope "sprint-phase:<sprint-id>/<phase-id>" --target-agent <reviewer> \
        --title "<slice>" --apply --json
      ```

3. Wake the reviewer where it works. Name the review request id when aops pm is on; otherwise name the slice, its files and its evidence:
   - in tmux, a one-line message;
   - in a room, `REVIEW READY: RR=<id> | slice=<name>; scope=<phase or files>; evidence=<tests or refs>`.
4. The reviewer gives one outcome: approved, changes_requested, commented or blocked.
   - With aops pm on, the reviewer records it on the review request. Material findings become issues linked to it (`aops pm issue create --source review --review-request <id> ...`), and fixes go into a child review request (`--parent <id>`).
   - With aops pm off, the reviewer answers in the tmux session or the room with the outcome and each finding's severity and evidence. You keep the list of open findings in the conversation, and in the checkpoint when checkpoint notes are on. Fixes come back as a new slice for review in the same place.
5. With aops pm on, a room message or a tmux reply is never the review record. Read the result on the review request.

## 3. Decide before you build

For an important decision that is costly to reverse, run an aops discuss topic with the other agent (`aops-cli-discuss`):
1. Each agent researches independently.
2. At least four substantive turns follow.
3. Each agent files a final stance.
4. You conclude and write the consensus.

Then write the plan from the consensus: in aops pm when it is on, otherwise in the conversation. Ask the reviewer to review the plan. Where the repository requires the user's approval of a consensus, implementation waits for it.

Two real, independent agents take part. One agent writing under two names is not a consensus.

## 4. Chat room protocol

Room messages stay short and scannable:

```text
TASK <n>: <scope>                                # user
ACK <n>: <goal, acceptance, validation plan>     # implementer
DONE <n>: <summary, files, evidence>             # implementer
VERIFIED <n>: <evidence> | REWORK <n>: <issue>   # reviewer, mirroring the review result
REVIEW READY: RR=<id> | slice=<name>; scope=<phase or files>; evidence=<refs>
CHECKPOINT: subject=<task or sprint>; memory=<id>; next=<action>
```

Read the room from your saved cursor (`--after-seq`), with delivered and read marks. In `listen`, exit 22 is a normal timeout. `aops-cli-chat` has the details.

## 5. The kickoff command

`aops start --json` composes a PM-backed kickoff for three cases: working alone, working alone with later reviews, and chat room work. It needs a board.
- Use it when one of those fits.
- It has no tmux mode and no setting for working without aops pm. When the user chose tmux, or chose to work without aops pm, set up with the steps above instead.

## Rules

1. With aops pm on, aops pm holds plans and reviews; without it, they stay in the conversation. aops discuss holds decisions; aops mem holds checkpoints. tmux and rooms carry conversation. Mirrors are read-only.
2. Board and room closeout belong to the user.
3. Ask the user at kickoff. Afterwards resolve questions from references, with the other agent, or in a discuss topic. Park decisions only the user can make (as PM feedback when aops pm is on, otherwise in a message to the user), and continue with other work.
4. Report only validation you ran in this session.
