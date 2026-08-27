---
name: build-review-chat
description: Use for live implementer and reviewer work where every implementation slice is independently reviewed and ChatV3 is coordination only.
---

# Build and review over chat

Use this discipline when an implementer and reviewer work concurrently and each implementation slice needs review before commit.

## Required assets

- `aops-collaborative-work`
- `aops-cli-projectman`
- `aops-cli-chat`
- `aops-cli-agentspace`

Use `aops-cli-discuss` only for material design consensus.

## Flow

1. Bind the slice to a Projectman microtask.
2. Implement and validate the bounded scope.
3. Open one Projectman review request with exact scope and evidence.
4. Use ChatV3 only to wake or coordinate with the reviewer.
5. Resolve findings, request re-review, and proceed only after acceptance.
6. Record the accepted slice and resume state.

## Boundaries

- The reviewer verifies code and runtime evidence independently.
- Material findings become linked issues; they do not stay only in chat.
- A review result does not authorize commit, publish, deletion, or another operator effect.
- Hosted and Docman mirrors are read-only.
