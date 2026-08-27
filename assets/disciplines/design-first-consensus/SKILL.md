---
name: design-first-consensus
description: Use for material, cross-owner, or expensive-to-reverse decisions that require independent research and concluded consensus before implementation.
---

# Design-first consensus

Use this discipline when architecture or ownership is uncertain enough that implementation should wait for a recorded decision.

## Required assets

- `aops-cli-discuss`
- `aops-cli-projectman`
- `aops-collaborative-work`
- `aops-cli-agentspace`

## Flow

1. Each participant researches code, PM, and canonical documents independently.
2. Run the Discuss turn protocol and record final stances.
3. Conclude only with complete consensus/disagreement/open-question outputs.
4. After operator approval, bind the consensus reference to a Projectman task and sprint plan.
5. Obtain plan review before implementation, then use bounded reviewed slices.

## Boundaries

- Discuss owns the decision; Projectman owns the implementation plan and reviews.
- ChatV3 may coordinate but is never decision truth.
- No implementation begins from an unconcluded topic or placeholder output.
- Concrete agent assignments belong to mission/session policy, not this reusable discipline.
