---
name: coordinator-loop
description: Use when one coordinator is the operator interface and directs bounded implementer work through canonical Projectman assignments and reviews.
---

# Coordinator loop

Use this discipline when the operator delegates a multi-task session to one coordinator who directs one or more implementers.

## Required assets

- `aops-collaborative-work`
- `aops-cli-projectman`
- `aops-cli-chat`
- `aops-cli-agentspace`

Use `aops-cli-discuss` for material design consensus.

## Flow

1. The coordinator independently verifies each request against code, PM, and canonical docs.
2. Create or select canonical mission/task/sprint references before assignment.
3. Assign bounded work with those references; chat prose alone is not an assignment.
4. Require one Projectman review request per implementation slice.
5. Verify evidence, loop on findings, and keep the assignment queue truthful.
6. Close with checkpoint, issue triage, review accounting, and a resume-ready handoff.

## Boundaries

- The coordinator is the single operator interface; implementers escalate through the coordinator.
- A coordinator may review only when independent verification is preserved.
- Concrete names belong to mission/session policy.
- Commit, publish, deletion, and other operator effects remain separately authorized.
