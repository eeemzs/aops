---
name: aops-interactive
version: 3
description: "Interactive AOPS router. Asks the operator what they want, then routes to the matching AOPS skill (loop orchestration, discuss/chat coordination, project management, ...) for detailed work, or does a small job itself (global skill/pointer sync) when there is no dedicated target. Extensible."
metadata:
  supersedes: "v2"
  short-description: "Interactive AOPS router skill"
  tags:
    - aops
    - interactive
    - router
    - loop
    - discuss
    - chat
    - projectman
    - global-sync
    - multi-agent
---

# AOPS Interactive

A single interactive entry point for AOPS work. **Ask the operator what they want, then route to the matching AOPS skill** for the detailed mechanics, or do a small job inline when there is no dedicated target. Keep this skill thin: it gathers intent and hands off; it does not duplicate the target skills' mechanics.

## Step 1 - Ask what they want

Ask via the runtime's question UI: "What do you want to do?"

- **Loop orchestration** - interactive `aops-cli loop` readiness, pack, supervised run handoff, foreground listening, and v1 closeout
- **Collaborate** - discuss topic (decision/consensus) and/or hosted chat room (coordination/wake)
- **Project management** - boards, sprints, tasks
- **Sync global skills/prompts** - Codex / Claude pointers (set, or delete + re-set)
- (more routes added over time)

Then branch:

## A. Loop orchestration -> route to `aops-loop-interactive`

Gather: PM task/sprint refs, reviewer/review-request refs, run scope, budget, tool/env policy, and permission policy. Ask whether this is dry-run only, supervised single-child, or an explicitly approved real run. Real run approval remains a double gate in the dedicated skill.

Then **load the `aops-loop-interactive` skill** and follow it. It owns the operator-facing loop playbook for `aops-cli loop plan|pack|start|listen|status`, owner-boundary reminders, supervised-run double-gate discipline, foreground listener behavior, and F6 v2 deferral. Do not duplicate those mechanics here.

## B. Collaborate -> route to `aops-cli-discuss` / `aops-cli-chat` / `aops-cli-projectman`

Gather: implementer agent? reviewer agent? a discuss topic first (design decision/consensus) or straight to coordination? live (hosted chat room) or async (PM review-request)? Roles are operator-assigned - never auto-assign.

Then **load the matching skill**: `aops-cli-discuss` for the decision/consensus ritual and `conclude` outputs; `aops-cli-chat` for hosted-room coordination/wake and listen/catchup loops; `aops-cli-projectman` for review-request/result, re-review, and operator-approved closeout. The repo-first `collab` surface is retired - there is no single collab skill; pick the surface that matches the need. Do not duplicate their mechanics here.

## C. Project management -> route to `aops-cli-projectman`

Gather: open a **new board** or use an **existing** one? List existing boards and let the operator select or add a new one. Proceed **sprint-based**, **task-based**, or **board + sprint**?

Then **load the `aops-cli-projectman` skill** and follow its workflow for the chosen shape. It owns board/sprint/task/issue mechanics.

## D. Sync global skills/prompts (inline - Codex & Claude)

This skill does this itself; no separate prompt is needed.

- **Set / update** global pointers:
  ```bash
  aops-cli sync pull --apply --hosted-project-slug aops --json
  pnpm skills:claude-codex:sync
  pnpm skills:claude-codex:check
  ```
- **Delete + re-set** after a hosted asset was removed. Sync does **not** auto-prune: remove the stale global pointer for the deleted asset, then re-run sync + check.

Targets: `~/.codex/skills`, `~/.codex/prompts`, `~/.claude/skills`, `~/.claude/commands`. Claude support varies by build and is best-effort.

## Principle

If a dedicated AOPS skill exists for the task, route to it: gather inputs, then load and follow it. If not, do the small job inline, step by step. Add new routes here as new needs appear.
