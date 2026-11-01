---
name: solo-pm-loop
description: Use for one-agent AOPS work with Projectman truth, bounded checkpoints, and review proportional to risk.
---

# Solo PM loop

Use this discipline for one agent handling low- or medium-uncertainty work.

## Required assets

- `aops-working-disciplines` (the solo rhythm)
- `aops-cli-projectman`
- `aops-cli-agentspace`
- `aops-cli-docman` when the work writes documents

Working alone does not load `aops-collaborative-work`. A material design decision needs a second agent: switch to `design-first-consensus` with `aops-cli-discuss`.

## Flow

Follow the session's kickoff answers: whether aops pm is on, whether checkpoint notes are on, and whether review requests are left for later. Repository rules come first.

1. Read relevant memory and, with aops pm on, the active Projectman task/sprint.
2. Work in the smallest useful slice.
3. Validate the changed behavior. With aops pm on, create an issue for material findings; otherwise report them to the user.
4. Leave a review request for a later session when the user chose it. When the risk is high and the user did not choose it, suggest it to the user.
5. With checkpoint notes on, write a checkpoint at a meaningful phase boundary. With aops pm on, leave PM state truthful.

## Boundaries

- With aops pm on, Projectman owns plan, issue, and review truth.
- Memory carries resume context; it does not replace the plan.
- Hosted and Docman mirrors are read-only.
- Commit, publish, deletion, or other operator effects require their own authority.
