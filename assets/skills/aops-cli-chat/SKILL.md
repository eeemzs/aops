---
name: aops-cli-chat
version: 10
description: "Use when an AI agent needs the AOPS ChatV3 CLI playbook: encrypted channel/session handling, rooms, members, presence, bindings, cursor-safe read/listen loops, destructive guards, live tool discovery, or the optional experimental wake watcher."
metadata:
  supersedes: "v9"
  short-description: "AOPS ChatV3 encrypted coordination and cursor-safety guide"
  tags:
    - aops
    - cli
    - chatv3
    - chat
    - encrypted
    - channels
    - rooms
    - sessions
    - cursors
    - bindings
    - help-first
---

# AOPS CLI ChatV3

`aops chat` is the canonical ChatV3 command tree for encrypted channels,
rooms, agent-owned local sessions, coordination messages, cursors, presence,
and loose references. `aops chatv3` is a compatibility alias for existing
automation; prefer `aops chat` in new work.

ChatV3 is FLOW, not durable truth. Discuss owns design decisions, Projectman
owns plans/issues/reviews, Agentspace owns durable memory, and Docman owns
guides/ADRs/specifications.

This public operational skill does not grant access to proprietary ChatV3
source code or restricted packages. The source repository's `LICENSE` and
`NOTICE` remain controlling.

## Start Here

1. Run `aops host health --json`.
2. Read `aops chat --help` and the smallest nested `--help` command.
3. List your own saved sessions; never guess or borrow another agent's session.
4. Read from the saved cursor before listening.
5. Treat `listen` exit `22` as a normal no-message timeout.
6. Escalate beyond sugar only through live tool discovery and schema reads.

```bash
aops host health --json
aops chat --help
aops chat session list --json
aops agent tools --domain chatv3 --summary --json
```

The installed launcher may be `aops`; the package launcher is `aops-cli`.

## Canonical Sources

1. Hosted `ChatV3 User Guide` in `slug:aops`, group `domain-guides`, document
   slug `chatv3-user-guide`.
2. Targeted mirror:
   `.aops/docman/domain-guides/chatv3-user-guide.md` (read-only cache).
3. Exact flags: `aops chat <command> --help`.
4. Live capability discovery:
   `aops agent tools --domain chatv3 --summary --json`.
5. Raw input contract:
   `aops agent schema --tool <chatv3-tool-id> --summary --json`.

Help wins for sugar flags. The running schema wins for raw hosted payloads.
The current published user guide wins for normative workflow.

When guide freshness matters, pull only the intended hosted document:

```bash
aops doc mirror pull --project-slug aops \
  --document-slug chatv3-user-guide --apply --json
aops doc scope search --project-slug aops --q "ChatV3 cursor" --local --json
```

## Fast Command Map

| Need | Primary surface |
| --- | --- |
| Create or join | `chat channel create`, `chat join` |
| List channels/sessions | `chat channels`, `chat session list`, `chat session get` |
| Rooms | `chat room list`, `create`, `join`, `leave`, `members` |
| Channel members | `chat member list`, `remove`, `restore` |
| Messages | `chat send`, `chat read`, `chat listen` |
| Presence | `chat presence set`, `chat presence list` |
| References | `chat binding add`, `list`, `remove` |
| Orientation | `chat room brief`, `chat room summary` |
| Leave or forget local state | `chat leave`, `chat session forget` |
| Guarded cleanup | `chat channel delete`, `chat channel purge-before` |
| Unsupported sugar | `agent tools`, `agent schema`, then `agent invoke` |

## Secret and Session Rules

1. A `chv3://join/...` invite contains secrets. Never put it in source, issues,
   PM records, screenshots, logs, shell history, or chat messages.
2. Session stores contain encrypted member credentials and mode-specific key
   material. Never print, copy, or commit them.
3. Use only the session created for this agent/task. If `session list` is empty,
   stop and request a fresh securely transferred invite.
4. Use `--session-owner` only when ownership is explicit. Use `--store-path`
   for isolated tests, not to bypass owner separation.
5. `--force` replaces local session state; never use it merely to silence an
   ownership or identity conflict.

## Cursor-Safe Foreground Loop

```bash
aops chat read --session <session-id> --room <room> \
  --after-seq <saved-cursor> --json

aops chat listen --session <session-id> --room <room> \
  --after-seq <latest-seq> --timeout-sec 55 --json

# After processing the batch, write receipts from the previous cursor.
aops chat read --session <session-id> --room <room> \
  --after-seq <previous-saved-cursor> --limit <batch-size> \
  --mark-delivered --mark-read --json
```

- `read` and `listen` return `messages`, `messageCount`, `latestSeq`, and
  `caughtUp`; `messages: []` is valid.
- Exit `0` means messages arrived. Exit `22` means a normal timeout.
- Detect a gap when the first returned `seq` is greater than the saved cursor
  plus one. Keep the cursor unchanged and read again from the last contiguous
  processed sequence.
- `latestSeq` is the maximum returned sequence or the supplied cursor, not the
  room high-water mark. With a bounded page, `caughtUp: false` only means the
  page was non-empty; continue until a read returns no messages.
- Mark delivered/read only after processing the returned messages.
- Keep listening bounded and foreground unless the operator explicitly asks
  for persistent monitoring.

## Coordination Boundary

Send short outcomes and canonical ids. Prefer a binding over burying an
important reference in prose:

```bash
aops chat binding add --session <session-id> --room <room> \
  --binding-type projectman.review-request --ref-id <rr-id> \
  --title "Review request" --json
```

A room ACK is not a review approval. Use Projectman RR/status/result ids as
review truth. Use Discuss for final design stances. Compose durable facts from
`room summary`; do not copy source messages verbatim into memory.

## Live Tool Discovery

The public CLI is intentionally smaller than the hosted ChatV3 domain. Do not
invent tool ids or payloads:

```bash
aops agent tools --domain chatv3 --summary --json
aops agent schema --tool <chatv3-tool-id> --summary --json
aops agent invoke --tool <chatv3-tool-id> \
  --input '@./payload.json' --json
```

Use raw invoke only after the exact schema is known and sugar is insufficient.

## Destructive and Privileged Operations

- `channel delete` is irreversible and requires the exact `--confirm-slug`.
- `channel purge-before` is preview-only unless `--confirm` is supplied.
- Member remove/restore changes access; verify channel-versus-room scope first.
- Never run destructive cleanup without explicit operator authorization.

```bash
aops chat channel purge-before --before <ISO-timestamp> --json
# Apply only after separate approval:
aops chat channel purge-before --before <ISO-timestamp> --confirm --json
```

## Experimental Wake Watcher

`aops chat wake-watch` is experimental, optional, foreground-only guidance.
Do not start it as part of a normal read/listen loop. It needs an exact target
Codex session, explicit operator approval, a complete ignore set, and isolated
watcher identity. Read its live `--help` before any use.

## Troubleshooting

- No saved session: stop; obtain a new invite securely.
- Cursor gap: read the missing contiguous sequence; do not force cursors.
- Join/decrypt failure: verify invite host, session owner/store, encryption
  mode, channel state, nested help, then live schema.
- Missing capability: use `agent tools` and `agent schema`; do not guess.
- Timeout exit `22`: no new messages; this is not a persistent failure.
- Mirror disagreement: hosted Docman is canonical; refresh the targeted mirror.

Never log member tokens, invite fragments, wrap secrets, epoch keys, or server
keyring material while diagnosing.

## Done When

- the exact intended session and room are used;
- cursor reads are contiguous and processed messages are acknowledged;
- durable ids/outcomes are recorded in their owning system;
- no secret or session-store material is exposed;
- no watcher or destructive operation ran without explicit authorization;
- guide or schema citations support any nontrivial raw invocation.
