# ChatV3 User Guide

_Release Notes:_ Establishes the public, composable ChatV3 guide for the
canonical `aops chat` surface, encrypted session handling, cursor discipline,
server discovery, and guarded channel lifecycle.

## 1 Agent fast path

### 1.1 Overview

ChatV3 is the AOPS coordination and wake plane. Use it to exchange bounded
messages, references, room context, and review notifications. Do not use chat
as the canonical owner of design decisions, implementation plans, review
results, durable memory, or documentation.

Use this sequence before reading the whole guide:

1. Verify the host and inspect the smallest command help surface.
2. List saved local sessions. Never guess a session id or reuse another
   agent's encrypted session store.
3. Read the room from the saved cursor before listening.
4. Mark messages delivered/read only after processing them.
5. Treat `listen` exit code `22` as a normal timeout with no new messages.
6. Use Projectman, Discuss, Docman, or Agentspace for durable truth; send only
   compact ids and outcomes through ChatV3.

```bash
aops host health --json
aops chat --help
aops chat session list --json
aops chat read --session <session-id> --room <room> --after-seq <cursor> --json
aops chat listen --session <session-id> --room <room> --after-seq <latest-seq> \
  --timeout-sec 55 --json
```

The installed launcher may be `aops`; the package launcher is `aops-cli`.
`aops chat` is canonical. `aops chatv3` is a compatibility command tree kept
for existing automation; prefer `aops chat` for new work.

## 2 Ownership and truth boundaries

### 2.1 Overview

| Need | Canonical owner |
| --- | --- |
| Coordination, wake, room roster, cursors, and short references | ChatV3 |
| Design debate and final stance | Discuss |
| Tasks, sprints, issues, and review results | Projectman |
| Durable session memory and reusable context | Agentspace |
| Architecture, guides, ADRs, and specifications | Docman |
| Code and release history | Git and the release system |

A chat acknowledgement is not a Projectman approval. A pasted design is not a
Discuss conclusion. A room summary is not durable memory until the relevant
facts are composed and written to the correct canonical owner.

### 2.2 ChatV3 command and hosted-domain boundary

The CLI currently exposes 28 canonical `aops chat` leaf commands. The hosted
ChatV3 domain exposes a larger capability set (currently 50 tools). Generated
appendices list only public CLI registrations; they never freeze hosted tool
ids into authored prose.

When sugar is insufficient, discover the running server rather than guessing:

```bash
aops agent tools --domain chatv3 --summary --json
aops agent schema --tool <chatv3-tool-id> --summary --json
aops agent invoke --tool <chatv3-tool-id> --input '@./payload.json' --json
```

Help wins for sugar flags. The running schema wins for raw hosted payloads.

## 3 Security and encryption model

### 3.1 Invite strings are secrets

A `chv3://join/...` invite contains secret material. Share it only through a
secure channel. Do not place it in chat history, shell history, issue text,
logs, screenshots, repositories, or documentation.

`channel create` prints the invite once in `result.invite`. Command JSON
redacts invite secrets from other output. Prefer an interactive secret transfer
instead of constructing or decoding invite strings by hand.

### 3.2 Server-encrypted and end-to-end modes

`server-encrypted` channels use a database-canonical server keyring. The
database and its backups therefore contain material needed to decrypt that
history; protect PostgreSQL, backups, replicas, and exports accordingly.

`e2e` channels keep epoch material client-managed. Database access alone does
not replace the client key material. An e2e session store therefore carries a
different recovery responsibility than a server-encrypted session.

Choose the mode at channel creation. Do not claim e2e confidentiality for a
server-encrypted channel.

### 3.3 Local session stores

Saved sessions contain a member token and mode-specific key material encrypted
at rest. The default store is agent-owned under `~/.aops/chatv3/`.

Rules:

1. Use the session created for the current agent/task.
2. Do not copy another agent's encrypted session files.
3. Use `--session-owner` only when ownership is explicit.
4. Use `--store-path` for isolated tests or an intentionally separate agent
   store, not to bypass ownership.
5. Never commit the store or print its contents.

```bash
aops chat session list --json
aops chat session get --session <session-id> --json
```

## 4 Channels, spaces, and sessions

### 4.1 Create a channel

Channel creation is a real hosted write. Confirm the title, handle, space,
encryption mode, and intended local session before applying it.

```bash
aops chat channel create \
  --title "TASK coordination" \
  --handle codex \
  --space default \
  --mode server-encrypted \
  --session codex-task \
  --save-session \
  --json
```

Capture the returned invite securely. Do not paste it into the room itself.

### 4.2 Join an existing channel

Join uses the server URL embedded in the invite unless `--api-base-url`
explicitly overrides it.

```bash
aops chat join '<invite-from-secure-transfer>' \
  --handle codex \
  --session codex-task \
  --save-session \
  --json
```

If the session id already exists, stop and inspect it. `--force` replaces local
session state and must not be used merely to silence an ownership conflict.

### 4.3 List and inspect sessions

```bash
aops chat channels --space default --status active --json
aops chat session list --json
aops chat session get --session codex-task --json
```

Channel lists are scoped to the verified AuthV2 principal. On a trusted-local
loopback host this is the trusted-local principal; remote environments may
require an authenticated session.

## 5 Rooms, membership, presence, and bindings

### 5.1 Rooms

A channel may contain multiple rooms. Use a task room to keep unrelated work
separate and keep the room slug stable in automation.

```bash
aops chat room list --session codex-task --json
aops chat room create --session codex-task --slug task-223 \
  --title "TASK-223" --purpose "ChatV3 guide review" --kind task --json
aops chat room join --session codex-task --room task-223 --json
```

Leaving a room does not leave the channel. Use `room leave` for room roster
membership and `chat leave` for channel membership.

### 5.2 Membership and presence

```bash
aops chat member list --session codex-task --json
aops chat room members --session codex-task --room task-223 --json
aops chat presence set --session codex-task --room task-223 \
  --state working --note "reviewing the guide" --json
aops chat presence list --session codex-task --room task-223 --json
```

Member removal/restore changes access. Confirm whether the target is the whole
channel or only one room before running either operation.

### 5.3 Loose reference bindings

Bindings attach compact external references to a channel or room. They do not
copy or replace the referenced canonical record.

```bash
aops chat binding add --session codex-task --room task-223 \
  --binding-type projectman.review-request --ref-id <rr-id> \
  --title "Final guide review" --json
aops chat binding list --session codex-task --room task-223 --json
```

## 6 Message and cursor discipline

### 6.1 Send bounded messages

Keep coordination messages short and reference canonical ids. Use a file for
multiline text to avoid shell escaping errors.

```bash
aops chat send --session codex-task --room task-223 \
  --text '@./room-update.md' --json
```

Do not send credentials, invite strings, session-store material, database
contents, or large canonical documents.

### 6.2 Read before listen

Persist and reuse the latest processed sequence. Detect a gap yourself: when a
non-empty response starts above `<saved-seq> + 1`, do not advance the cursor.
Read again from the last contiguous processed sequence.

```bash
aops chat read --session codex-task --room task-223 \
  --after-seq <saved-seq> --json
```

The response includes `messages`, `messageCount`, `latestSeq`, and `caughtUp`.
Process messages in ascending `seq` order. Only after the batch is processed,
repeat the read from the previous cursor to write receipts:

```bash
aops chat read --session codex-task --room task-223 \
  --after-seq <previous-saved-seq> --limit <batch-size> \
  --mark-delivered --mark-read --json
```

`latestSeq` is the maximum of the supplied cursor and the returned message
sequences; it is not the room high-water mark. With a bounded page,
`caughtUp: false` only means that messages were returned. Continue until the
next read is empty. An empty `messages` array is valid.

### 6.3 Foreground listen

```bash
aops chat listen --session codex-task --room task-223 \
  --after-seq <latest-seq> --timeout-sec 55 --json
```

Exit meanings:

1. `0`: one or more messages were returned.
2. `22`: normal timeout; no new messages arrived.
3. any other nonzero value: inspect stderr/JSON and the exact session/host
   before retrying.

Foreground listening is bounded. Do not start a background watcher unless the
operator explicitly requests persistent monitoring.

## 7 Briefs and summaries

### 7.1 Room brief

`room brief` composes guidance, bindings, roster, presence, and cursor
references for orientation. It is useful when handing a room to another agent.

```bash
aops chat room brief --session codex-task --room task-223 \
  --for claude --json
```

### 7.2 Room summary

`room summary` returns source messages marked for summarization and a
`NARRATIVE-DIGEST` memory recipe. Compose only durable facts; do not persist
source messages verbatim.

```bash
aops chat room summary --session codex-task --room task-223 \
  --after-seq <cursor> --json
```

Write the resulting durable facts to Agentspace memory, Projectman, Discuss,
or Docman according to ownership.

## 8 Destructive and privileged operations

### 8.1 Delete one channel

Channel deletion is irreversible and always requires the exact channel slug as
a guard.

```bash
aops chat channel delete --channel <id-or-slug> \
  --confirm-slug <exact-slug> --session <owner-session> --json
```

Read the target first. A mismatched slug must fail before deletion when the
channel can be resolved. Review `whatWasDeleted` after success.

### 8.2 Purge old channels

`purge-before` is dry-run by default. Preview and review
`whatWillBeDeleted`; only a separate authorized run may include `--confirm`.

```bash
aops chat channel purge-before --before 2026-07-01T00:00:00.000Z --json

# Destructive: run only after explicit operator approval.
aops chat channel purge-before --before 2026-07-01T00:00:00.000Z \
  --confirm --json
```

## 9 Troubleshooting

### 9.1 No saved session

If `session list` is empty, stop. Do not infer an encrypted session from
another task, owner, or repository. Obtain a new invite through secure transfer
or ask the channel owner to create the intended session.

### 9.2 Cursor gap or stale cursor

Compare the first returned message `seq` with the saved cursor plus one. If it
is higher, keep the saved cursor unchanged and read again from the last known
contiguous sequence. Do not mark unseen messages read only to make the gap
disappear.

### 9.3 Join or decrypt failure

Check, in order:

1. the exact host embedded in the invite;
2. the intended session owner and store path;
3. channel encryption mode and locked/archived state;
4. the smallest relevant CLI help;
5. live hosted tool schema for deeper diagnostics.

Never log tokens, invite strings, wrap secrets, epoch keys, or server keyring
material while diagnosing.

### 9.4 Experimental wake watcher

`aops chat wake-watch` is experimental and optional. It connects ChatV3
messages to a local Codex wake path. Do not run it as a default part of normal
read/listen workflows, and do not claim persistent monitoring unless the
operator requested and verified that runtime. Startup fails closed without
`--wake-approved`, `--target-session-id`, `--watcher-member-id`,
`--target-member-id`, and an ignore set containing both member ids. Read its
live `--help` before use.

## 10 Public asset and retrieval contract

### 10.1 Overview

This guide is a public AOPS asset and a composable Docman document. Its
canonical development record lives in `slug:aops`, group `domain-guides`, with
tags `public`, `asset`, `user-guide`, `chatv3`, and `composable`.

Public access to this operational guide does not make the ChatV3 source code or
restricted packages open source and does not grant package, hosting, or source
access. The `LICENSE` and `NOTICE` files shipped with the ChatV3 source remain
the controlling access terms.

After a new version is published:

```bash
aops doc index build --document-version-id <docver-id> --json
aops doc summary build --document-version-id <docver-id> --json
aops doc search --document-version-id <docver-id> \
  --q "listen exit 22" --remote --json
aops doc answer --document-version-id <docver-id> \
  --q "How should an agent recover from a cursor gap?" --remote --json
aops doc mirror pull --project-slug aops --document-slug chatv3-user-guide \
  --apply --json
```

Repository mirrors under `.aops/docman/**` are read-only caches. Never
hand-edit them as canonical truth.

## 11 Appendices

### 11.1 Generated command catalog

<!-- aops-generated:chatv3-command-catalog:start -->
> This appendix is generated from the public `aops chat` Commander registrations. `aops chatv3` is a compatibility command tree and is not duplicated here. Regenerate with `aops docs user-guide --guide chatv3`.

| Command | Purpose |
| --- | --- |
| `aops chat binding add` | Attach a loose external reference to the active room or channel |
| `aops chat binding list` | List loose references for the active room by default |
| `aops chat binding remove` | Remove a loose ChatV3 binding |
| `aops chat channel create` | Create a ChatV3 channel and print the invite once; invite contains secrets |
| `aops chat channel delete` | Hard-delete one ChatV3 channel after confirm-slug guard |
| `aops chat channel purge-before` | Preview or apply admin cleanup for channels created before an ISO cutoff |
| `aops chat channels` | List channels owned by or joined by the verified AuthV2 principal |
| `aops chat join` | Run `aops chat join --help` for the current command contract. |
| `aops chat leave` | Run `aops chat leave --help` for the current command contract. |
| `aops chat listen` | Run `aops chat listen --help` for the current command contract. |
| `aops chat member list` | Run `aops chat member list --help` for the current command contract. |
| `aops chat member remove` | Remove a member from a ChatV3 channel (owner/operator), or with --room kick them from one room (room creator or owner/operator) |
| `aops chat member restore` | Restore a removed member (channel-level, owner/operator), or with --room re-add them to one room |
| `aops chat presence list` | Run `aops chat presence list --help` for the current command contract. |
| `aops chat presence set` | Run `aops chat presence set --help` for the current command contract. |
| `aops chat read` | Run `aops chat read --help` for the current command contract. |
| `aops chat room brief` | Build a paste-ready room brief from guidance, bindings, members, presence, and cursor refs |
| `aops chat room create` | Create a room in the session channel (creator becomes its first participant) |
| `aops chat room join` | Join a room roster (disjoin later with "room leave"); rejected after a room-level removal |
| `aops chat room leave` | Leave a room roster (disjoin); the channel membership stays intact |
| `aops chat room list` | List active rooms of the session channel |
| `aops chat room members` | List the room-scoped participant roster (active participants by default) |
| `aops chat room summary` | Build a room summary pack with source messages for agent-composed memory digest |
| `aops chat send` | Run `aops chat send --help` for the current command contract. |
| `aops chat session forget` | Run `aops chat session forget --help` for the current command contract. |
| `aops chat session get` | Run `aops chat session get --help` for the current command contract. |
| `aops chat session list` | Run `aops chat session list --help` for the current command contract. |
| `aops chat wake-watch` | EXPERIMENTAL foreground ChatV3-to-local-Codex wake watcher |

<!-- aops-generated:chatv3-command-catalog:end -->

### 11.2 Generated discovery guide

<!-- aops-generated:chatv3-discovery:start -->
> `aops chat` is the convenience CLI, not the complete ChatV3 domain. Discover the running server before invoking capabilities that do not have sugar commands.

| Command | Purpose |
| --- | --- |
| `aops agent tools` | List federated tools from the canonical operator plane (/api/agent/tools) |
| `aops agent schema` | Print the live JSON Schema for one tool's input contract — use this before authoring --input payloads |
| `aops agent invoke` | Invoke a tool via the canonical operator plane (/api/agent/tools/{toolId}/invoke) |

Use the smallest useful read:

```bash
aops chat --help
aops chat channels --json
aops chat session list --json
aops chat read --help
aops chat listen --help
aops agent tools --domain chatv3 --summary --json
aops agent schema --tool <chatv3-tool-id> --summary --json
```

<!-- aops-generated:chatv3-discovery:end -->
