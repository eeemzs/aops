---
name: solo-pm-loop
description: Use for one-agent AOPS work with Projectman truth, bounded checkpoints, and review proportional to risk.
---

# Solo PM loop

Use this discipline for one agent handling low- or medium-uncertainty work.

## Required assets

- `aops-collaborative-work`
- `aops-cli-projectman`
- `aops-cli-agentspace`

Use `aops-cli-discuss` only when material design uncertainty appears.

## Flow

1. Read the active Projectman task/sprint and relevant memory.
2. Work in the smallest useful microtask slice.
3. Validate the changed behavior and create an issue for material findings.
4. Request asynchronous review when risk warrants it.
5. Write a checkpoint at a meaningful phase boundary and leave PM state truthful.

## Boundaries

- Projectman owns plan, issue, and review truth.
- Memory carries resume context; it does not replace the plan.
- Hosted and Docman mirrors are read-only.
- Commit, publish, deletion, or other operator effects require their own authority.
