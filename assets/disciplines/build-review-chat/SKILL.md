---
name: build-review-chat
description: Use for live implementer and reviewer work where every implementation slice is independently reviewed, in aops pm review requests when aops pm is on, otherwise in the tmux session or aops chat room.
---

# Build and review

Use this discipline when an implementer and reviewer work concurrently and each implementation slice needs review before commit. The reviewer works in a tmux session or meets the implementer in an aops chat room.

## Required assets

- `aops-collaborative-work`
- `aops-cli-projectman` when aops pm is on
- `aops-cli-agentspace`
- `aops-cli-chat` when the agents meet in a room

Use `aops-cli-discuss` only for material design consensus.

## Flow

The kickoff chose whether aops pm and checkpoint notes are on; follow those answers. Repository rules come first.

1. With aops pm on, bind the slice to a Projectman microtask.
2. Implement and validate the bounded scope.
3. Ask for review with exact scope and evidence: with aops pm on, in one Projectman review request; otherwise in the tmux session or the room.
4. With aops pm on, use the tmux session or the room only to wake or coordinate with the reviewer.
5. Resolve findings, request re-review, and proceed only after acceptance.
6. Record the accepted slice and resume state: in aops pm when it is on, in a checkpoint when checkpoint notes are on.

## Boundaries

- The reviewer verifies code and runtime evidence independently.
- Material findings stay tracked until resolved: with aops pm on as linked issues; otherwise on the implementer's open-findings list in the conversation, and in the checkpoint when checkpoint notes are on.
- A review result does not authorize commit, publish, deletion, or another operator effect.
- Hosted and Docman mirrors are read-only.
