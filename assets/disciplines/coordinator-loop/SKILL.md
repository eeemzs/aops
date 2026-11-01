---
name: coordinator-loop
description: Use when one coordinator is the operator interface and directs bounded implementer work through canonical Projectman assignments and reviews.
---

# Coordinator loop

Use this discipline when the operator delegates a multi-task session to one coordinator who directs one or more implementers.

It needs aops pm for assignments and reviews. When the kickoff turned aops pm off, tell the user and ask once whether to turn it on for this session; do not turn it on yourself. Without it, work with `build-review-chat`.

## Required assets

- `aops-collaborative-work`
- `aops-cli-projectman`
- `aops-cli-agentspace`
- `aops-cli-chat` when the agents meet in a room

Use `aops-cli-discuss` for material design consensus.

## Flow

1. The coordinator independently verifies each request against code, PM, and canonical docs.
2. Create or select canonical task and sprint references before assignment.
3. Assign bounded work with those references; chat prose alone is not an assignment.
4. Require one Projectman review request per implementation slice.
5. Verify evidence, loop on findings, and keep the assignment queue truthful.
6. Close with checkpoint, issue triage, review accounting, and a resume-ready handoff.

## Boundaries

- The coordinator is the single operator interface; implementers escalate through the coordinator.
- A coordinator may review only when independent verification is preserved.
- Concrete names belong to the session's setup.
- Commit, publish, deletion, and other operator effects remain separately authorized.
