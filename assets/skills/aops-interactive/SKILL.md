---
name: aops-interactive
version: 4
description: "Interactive AOPS router. Asks the operator what they want, routes to the matching AOPS skill, or runs the simple aops assets lifecycle when global AOPS assets are requested."
metadata:
  supersedes: "v3"
  short-description: "Interactive AOPS router skill"
  tags:
    - aops
    - interactive
    - router
    - loop
    - discuss
    - chat
    - projectman
    - assets
---

# AOPS Interactive

Ask the operator what they want, then route to the smallest matching AOPS skill. Keep this router thin and do not duplicate target-skill mechanics.

## Choose a route

- **Loop orchestration**: load `aops-loop-interactive`.
- **Decision or consensus**: load `aops-cli-discuss`.
- **Coordination or wake**: load `aops-cli-chat`.
- **Boards, sprints, tasks, issues, or reviews**: load `aops-cli-projectman`.
- **Tasker or Runner operation**: load `aops-cli-tasker`.
- **Documents**: load `aops-cli-docman`.
- **Global AOPS assets**: use the simple inline lifecycle below.

Roles remain operator-assigned. Do not automatically create a reviewer, room, sprint, or real loop run.

## Global AOPS assets

There is one installer for CLI, setup, and TUI:

```bash
aops assets status --json
aops assets install --target all --json
aops assets update --target all --json
aops assets rollback --target all --json
```

`install` and `update` clean-reinstall AOPS-managed paths, remove assets retired by the new manifest, and preserve unrelated user skills. The former `~/.aops/agent-assets` store is removed after a successful new install. Do not delete whole Codex or Claude skill roots.

Use `aops assets update --tag <version> --target all --json` for one exact public release. Use `--from-release <directory>` only for a reviewed local/disposable release.

Global installation is an operator effect. Asking to author or refresh hosted skills does not authorize it.

## Principle

If a dedicated AOPS skill exists, gather the minimum inputs and follow it. Otherwise do the small job inline using live `--help`. Never revive the retired repo-pointer/global-sync system.
