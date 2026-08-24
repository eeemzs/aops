---
name: aops-cli-collab
version: 31
description: "DEPRECATED — the repo-first collab command surface is retired. Routes agents to aops-cli-discuss (decision/consensus), aops-cli-chat (coordination/wake via hosted rooms), and aops-cli-projectman (review + issues). `aops-cli start` multi-agent mode is now chat-room, not full-collab. Kept one release for compatibility."
metadata:
  supersedes: "v30"
  short-description: "Deprecation pointer: collab split into discuss + chat + PM"
  tags:
    - cli
    - agentspace
    - deprecated
    - pointer
    - discuss
    - chat
    - projectman-review-request
---

# AOPS CLI Collab (deprecated)

The repo-first `collab` command surface (`aops-cli collab ...`, the collab session ledger, and collab chat) is **retired**. Coordination, decision, and review now live in three separate surfaces — pick the one that matches the need:

- **`aops-cli-discuss`** — decision / consensus: standalone discussion topics, the design-decision ritual, the two-agent turn protocol, and deterministic `conclude` outputs.
- **`aops-cli-chat`** — coordination / wake: hosted Agentspace rooms and DMs, members, bindings, and `inbox`/`listen`/`catchup` read loops.
- **`aops-cli-projectman`** — review + execution truth: `pm review-request` create/result, re-review (`--parent`), `pm issue`, and `pm feedback`.

`aops-cli start`'s multi-agent mode is now **`chat-room`** (the `full-collab` mode was removed). There is **no automatic decision bridge**: `discuss conclude` writes only its own outputs; surfacing a decision to PM or chat is explicit (a PM feedback/issue carrying the discussion-topic ref, or a chat `agentspace.discussion-topic` binding).

This pointer is kept for **one release** for compatibility; new instructions should reference the three skills above directly. The full v30 collab playbook lives on in this skill's version history.

## Old → new mapping

| Old (retired collab) | New |
|----------------------|-----|
| `collab start` / session ledger / `collab event` / `collab status` / `collab close` | Hosted **`aops-cli chat`** room for coordination + **`aops-cli pm`** for review/issue/closeout truth (`aops-cli-chat`, `aops-cli-projectman`) |
| `collab chat` (repo-first session chat) | **`aops-cli chat`** hosted rooms/DMs (`aops-cli-chat`) |
| `collab review request` / `collab review result` (session review composite) | **`aops-cli pm review-request create` / `... result`** + `pm review-request create --parent <rr-id>` for re-review (`aops-cli-projectman`) |
| Session-bound discuss (`discuss start --session ...`) | **Standalone `aops-cli discuss`** (no `--session`) (`aops-cli-discuss`) |
| `collab listen` / `collab chat daemon` listener patterns | **`aops-cli chat listen` / `chat catchup`** read loop (`aops-cli-chat`) |
| `collab close` + post-collab memory | Operator-approved **`pm board closeout`** + `aops-cli mem` closeout note (`aops-cli-projectman`, `aops-cli-agentspace`) |
| `start --mode full-collab` | `start --mode chat-room` |

If `--help` and this pointer disagree, `--help` wins.
