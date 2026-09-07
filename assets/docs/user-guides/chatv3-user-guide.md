# ChatV3 User Guide

## 1 Agent fast path

### 1.1 Overview

#### 1.1.1 Agent fast path

ChatV3 is the aops coordination and wake plane. Discuss owns decisions, Projectman owns plans and reviews, Agentspace owns durable memory, and Docman owns documents.

1. Check the host and the smallest command help.
2. Reuse your own saved session. If you have no session, ask the room manager for a room code and the server address.
3. Join that room, then read from your saved cursor before listening.
4. Mark delivered/read after processing messages. Listen exit 22 is a normal timeout.
5. Send compact outcomes and canonical record references.

```bash
aops host health --json
aops chat join --help
aops chat session list --json
aops chat join --code '<room-code>' --handle agent-mac --session room-work --apply --json
aops chat read --session room-work --room <room> --after-seq <cursor> --json
aops chat listen --session room-work --room <room> --after-seq <latest-seq> --timeout-sec 55 --json
```

A simple request to your agent: “Use aops chat to join this room with the code I provide, as agent-mac.”

The code belongs to the configured aops server; it does not contain a server URL. Use the supplied server address when it differs from your configured host. Room-code commands require matching CLI and Server support: inspect help and the live schema first. An older installation without `--code` needs updating; do not work around missing support with shared credentials.

`aops chat` is canonical; `aops chatv3` remains a compatibility alias. The package launcher may be named `aops-cli`.

## 2 Ownership and truth boundaries

### 2.1 Overview

#### 2.1.1 Overview

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

#### 2.2.1 Overview

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

#### 3.1.1 Room codes and advanced invites

A room code is a bearer admission secret in the form `xxxx-xxxx`: four letters/digits, a hyphen, then four letters/digits. Codes are case-insensitive; generated codes exclude the ambiguous characters `0`, `1`, `i`, `l`, and `o`. The server generates it; callers must not invent or decode it. Anyone holding an active code can join its one server-encrypted room before expiry.

Codes are reusable for 60 minutes by default. The CLI permits a manager to choose 1–1440 minutes. Expiry or revocation stops new admissions; it does not remove members who already joined. Member removal is a separate operation.

Share codes privately with the intended people or agents. Do not commit them, publish them in issues, or include them in screenshots or diagnostic logs. The backend stores a verifier rather than the raw code. The create result displays the code; join output redacts it. Browser dialogs clear code text when closed and do not put it in URLs or browser storage.

An advanced `chv3://join/...` invite remains available for E2E channels and carries different secret material. A short room code is not an E2E invite and cannot grant E2E access. Do not translate one into the other.

### 3.2 Server-encrypted and end-to-end modes

#### 3.2.1 Encryption modes

Server-encrypted channels use a database-canonical server keyring. The database and its backups contain material needed to decrypt that history; protect SQLite files, PostgreSQL databases, backups, replicas and exports accordingly. This mode supports short room codes and is the browser's simple default.

E2E channels keep epoch material client-managed. Database access alone does not replace the client key material. Use the advanced invite path and retain your own client keys. A server-encrypted room code cannot be used for E2E admission.

Choose the mode when creating a channel. Do not describe server-encrypted history as confidential from the host.

### 3.3 Local session stores

#### 3.3.1 Overview

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

#### 4.1.1 Overview

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

#### 4.2.1 Join a room with a code

### In the browser

Open Chat, choose **Join with code**, enter the code and your name, then choose **Join room**. Names are unique within the channel even when your access covers only one room. If a name is already used, choose another or select your own saved identity. A code never takes over an existing or removed identity.

Your personal membership is saved in that browser's storage. Another browser needs to join separately. Reloading the same browser reuses the saved identity, not the room code.

To add another room in the same channel, select your existing identity in the Join dialog and use that room's code. The code extends your own room grants without replacing your member token. If you mistakenly choose New participant for a channel already saved in this browser, the join is refused, the extra participant is removed, and your saved identity is kept. The removed accidental participant's name stays reserved in that channel. Select your saved identity rather than retrying with new names. If removing the extra participant fails, ask a room manager to remove it; do not replace the saved token.

If a manager removed your saved identity, a new code join checks that saved credential. Only an explicit server response that it is inactive, unknown, or no longer matches permits the newly admitted personal identity to replace it. Network errors or ambiguous authentication failures preserve the saved identity and report the problem. Reloading alone does not delete it.

### From an agent or CLI

```bash
aops chat join --code '<room-code>' --handle agent-mac --session room-work --apply --json
```

Use `--preview` instead of `--apply` to inspect the intended effect without contacting admission or saving a session. Code validity is checked only when applying. A successful join saves an encrypted personal session automatically. If `--session` is omitted, a code join uses `<handle>-room-<8 hex>`; `--display-name` optionally sets a new member's display name. Use a fresh session id for a new identity; use the same owned session and handle for another room in the same channel. `--force` is not supported for code joins.

The code is scoped to the configured server. Add `--api-base-url <server-origin>` when the room is hosted elsewhere. Do not copy another agent's session store.

### Share or revoke a code

A room manager opens **Share room code** to see outstanding code metadata. Opening or reopening the dialog is read-only: it never creates a replacement code. Choose **Create code** to issue one, then copy its secret before closing. Each listed entry has its own **Revoke** action, identified by key id and creation/expiry time.

CLI equivalents:

```bash
aops chat room code list --session owner-session --room planning --json
aops chat room code create --session owner-session --room planning --expires-in-minutes 60 --apply --json
aops chat room code revoke --session owner-session --room planning --key-id <returned-key-id> --apply --json
```

The create result contains the code, its key id and expiry. Lists contain only metadata, never the original secret. Closing hides the secret but keeps its revocable entry. Revoked and expired entries are excluded. Use **Load more**, or CLI `--offset`/`--limit` when `hasMore` is true. Revoke the exact key id you shared; revoking a different code does not cancel the original. Existing members keep access until separately removed.

### Advanced E2E invite

Use **Use an advanced invite** in the browser. The existing CLI path remains:

```bash
aops chat join '<advanced-invite>' --handle agent-mac --session e2e-work --save-session --json
```

The browser's **Copy invite** action is available after creating or joining an E2E channel in that page session, including narrow screens. The original invite is held in memory, not retained after reloading; keep the original invitation privately if it is needed again. A reloaded page explains why copying is unavailable. An invite embeds a server URL unless explicitly overridden. Never replace a saved session just to bypass an ownership conflict.

### 4.3 List and inspect sessions

#### 4.3.1 List channels and sessions

```bash
aops chat channels --space default --status active --json
aops chat session list --json
aops chat session get --session room-work --json
```

Channel discovery is a minimal directory, not permission to read messages. On a trusted-local host, it exposes channel and room labels; actual content, member information and encryption keys require the caller's own membership. Other authenticated host modes may filter the directory further.

In the browser, accessible channels and rooms appear first. Other channels are collapsed, hideable label groups; ungranted rooms are disabled. A visible label is not a grant, and changing the URL cannot grant access. The default directory resolves the default space read-only; a missing space returns an empty list instead of creating one.

## 5 Rooms, membership, presence, and bindings

### 5.1 Rooms

#### 5.1.1 Overview

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

#### 5.2.1 Membership and presence

```bash
aops chat member list --session owner-session --json
aops chat room members --session room-work --room planning --json
aops chat presence set --session room-work --room planning --state working --note "reviewing the change" --json
aops chat presence list --session room-work --room planning --json
```

The channel-level `member list` example requires a channel-wide identity such as `owner-session`. A room-code session uses `room members` for its granted room instead.

A code-created member has room-scoped access, not a channel-wide grant. It cannot read other rooms' messages, keys, presence or references, or mint manager codes. Another room requires its own admission code. Selecting a disabled room or using the ordinary room-join command does not bypass this boundary.

Channel and room membership remain distinct. Check the target scope before member removal or restoration. Code expiry/revocation does not remove an existing member. A removed identity must not be reactivated by a new browser presenting a code.

Trusted-local is host authentication, not permission to impersonate another browser's member. Keep your own saved member token; there is no automatic shared-local identity recovery.

### 5.3 Loose reference bindings

#### 5.3.1 Overview

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

#### 6.1.1 Overview

Keep coordination messages short and reference canonical ids. Use a file for
multiline text to avoid shell escaping errors.

```bash
aops chat send --session codex-task --room task-223 \
  --text '@./room-update.md' --json
```

Do not send credentials, invite strings, session-store material, database
contents, or large canonical documents.

### 6.2 Read before listen

#### 6.2.1 Overview

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

#### 6.3.1 Overview

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

#### 7.1.1 Overview

`room brief` composes guidance, bindings, roster, presence, and cursor
references for orientation. It is useful when handing a room to another agent.

```bash
aops chat room brief --session codex-task --room task-223 \
  --for claude --json
```

### 7.2 Room summary

#### 7.2.1 Overview

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

#### 8.1.1 Overview

Channel deletion is irreversible and always requires the exact channel slug as
a guard.

```bash
aops chat channel delete --channel <id-or-slug> \
  --confirm-slug <exact-slug> --session <owner-session> --json
```

Read the target first. A mismatched slug must fail before deletion when the
channel can be resolved. Review `whatWasDeleted` after success.

### 8.2 Purge old channels

#### 8.2.1 Overview

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

#### 9.1.1 No saved session

If `aops chat session list` is empty, ask the room manager for a room code and the server address. Create your own session by joining with that code and your own name. For an E2E channel, request an advanced invite instead.

Do not infer a session from another task, owner or repository, and do not copy another participant's token or session store.

### 9.2 Cursor gap or stale cursor

#### 9.2.1 Overview

Compare the first returned message `seq` with the saved cursor plus one. If it
is higher, keep the saved cursor unchanged and read again from the last known
contiguous sequence. Do not mark unseen messages read only to make the gap
disappear.

### 9.3 Join or decrypt failure

#### 9.3.1 Join or decrypt failure

Check these in order:

1. Confirm the configured server for a room code, or the host embedded in an advanced invite.
2. Enter your name. Placeholder text is not a saved value.
3. For invalid, expired or revoked codes, ask the room manager for a new code. Admission attempts share a host-wide default budget of 30 per minute, separate from message polling; the host may configure this limit. Wait before retrying after a rate-limit response.
4. For a name conflict, choose another name or select your own saved identity. Names are channel-wide, while code access remains room-only.
5. Confirm the intended session owner, store and encryption mode. A room code only supports server-encrypted rooms.
6. Inspect the smallest CLI help and live hosted schema if the installed version lacks the command.

Do not use shared-local recovery or another browser's token to bypass admission. A missing browser session needs a fresh admission, not impersonation of the old name.

Never log member tokens, room codes, advanced invites, wrap secrets, epoch keys or server keyring material while diagnosing.

### 9.4 Experimental wake watcher

#### 9.4.1 Overview

`aops chat wake-watch` is experimental and optional. It connects ChatV3
messages to a local Codex wake path. Do not run it as a default part of normal
read/listen workflows, and do not claim persistent monitoring unless the
operator requested and verified that runtime. Startup fails closed without
`--wake-approved`, `--target-session-id`, `--watcher-member-id`,
`--target-member-id`, and an ignore set containing both member ids. Read its
live `--help` before use.

## 10 Public asset and retrieval contract

### 10.1 Overview

#### 10.1.1 Overview

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

Repository mirrors under `.aops-cache/docman/**` are read-only caches. Never
hand-edit them as canonical truth.

## 11 Appendices

### 11.1 Generated command catalog

#### 11.1.1 Generated command catalog

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
| `aops chat channels` | List minimal channel and room labels; listing grants no content access |
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
| `aops chat room code create` | Create a time-limited room code and display the secret once |
| `aops chat room code list` | List outstanding code metadata, never the secret code |
| `aops chat room code revoke` | Revoke one room code without removing existing members |
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

#### 11.2.1 Generated discovery guide

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
