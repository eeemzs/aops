# Projectman User Guide v2

_Release Notes:_ Clarifies the issue/feedback list data envelope, restores launcher context, and discloses pre-release --limit availability.

## 1 Agent fast path

### 1.1 Overview

Projectman is the AOPS domain for planning and execution tracking. It owns the
records that answer: What work exists? How is it divided? What is its current
state? Which findings and reviews are attached to it?

The package launcher is `aops-cli`; an installed distribution may expose the
same program as `aops`. Examples use `aops` for brevity.

Use this sequence before reading the whole guide:

1. Check the target project and server:

   ```bash
   aops host health --json
   aops project links list --json
   ```

2. Select the intended hosted project explicitly with
   `--project-slug <slug>` when more than one project is linked.
3. Read the smallest useful inventory first:

   `--limit` on issue and feedback lists requires a CLI build newer than
   `@aopslabs/aops` 0.3.31. An installed 0.3.31 rejects that flag; until the
   next CLI release, omit it or use the repository-built CLI.

   ```bash
   aops pm resume --project-slug <slug> --for <agent-id> --limit 10 --json
   aops pm sprint list --project-slug <slug> --status doing --limit 10 --summary --json
   aops pm issue list --project-slug <slug> --status open --limit 10 --json
   aops pm feedback list --project-slug <slug> --status new --limit 10 --json
   ```

4. Fetch one exact record before changing it:

   ```bash
   aops pm sprint get --project-slug <slug> --id <sprint-id> --json
   ```

5. Run nested `--help` before a write. Use `--preview` where offered, then
   `--apply`; destructive writes require both `--apply` and `--confirm`.
6. Keep durable narrative context in Agentspace memory. Projectman notes should
   contain planning facts, references, acceptance criteria, and short evidence.
7. Never hand-edit `.aops/projectman/**`. It is a derived local mirror, not
   planning truth.

## 2 Ownership at a glance

### 2.1 Overview

| Question | Owner |
|---|---|
| What work is planned and what is its status? | Projectman |
| Why did we choose this design? | Discuss for consensus; Docman for the durable decision record |
| What should another session remember? | Agentspace memory |
| What is the canonical written guide or specification? | Docman |
| Where do agents coordinate or wake one another? | ChatV3 |
| Where is an implementation review and its result canonical? | Projectman review request |

Projectman does not replace source control, documentation, durable memory, or
chat. Chat messages can announce a review, but the review request and immutable
result belong in Projectman. A Discuss conclusion can approve a design, but its
implementation scope must be bound to a Projectman task/sprint before work.

## 3 Projectman model

### 3.1 Project partition

#### 3.1.1 Overview

Every record belongs to one hosted project scope. Repository links in
`.aops/aops.config.json` tell the CLI which hosted projects are available. The
server record remains canonical; the link only resolves the destination.

```bash
aops project links list --json
aops pm board list --project-slug aops --json
```

When the current repository is linked to multiple projects, always pass
`--project-slug`. Do not rely on an ambient default for a material write.

### 3.2 Board

#### 3.2.1 Overview

A board is a long-lived work stream. It owns ordered column placements and can
hold a bootstrap registry pointing at an active task/sprint window.

Default board columns are Backlog, Todo, Doing, and Done. `--column` replaces
that set; `--append-column` extends it. Board lifecycle sugar is:

1. `kickoff`: reuse an open active window or create a task+sprint window
2. `resume`: read the registry and return the active context
3. `closeout`: operator-approved final closure of the active window

### 3.3 Kanban task

#### 3.3.1 Overview

A kanban task is the operator-visible unit of work. Give it a semantic title.
Use its notes/references for ids and evidence instead of exposing opaque codes
as the title.

### 3.4 Sprint, phase, and microtask

#### 3.4.1 Overview

A sprint is a bounded execution window, usually linked to one task. A phase is
a grouping inside that sprint. A microtask is the smallest explicit status and
evidence owner.

For substantive or resumable work, write the goal in this form:

```text
NE: What will change.
NICIN: Why this work is needed.
DONE-WHEN: Observable acceptance conditions.
```

Each phase must contain at least one microtask. Phase status is derived from its
microtasks; do not send `phase.status` in a plan payload.

`pm sprint set-status` rewrites every nested microtask to the requested status.
Use `pm utask update` when per-item evidence or mixed status must be preserved.

### 3.5 Issue and feedback

#### 3.5.1 Overview

An issue is a concrete defect, risk, or blocker. Feedback is an observation,
request, or improvement signal that may not yet be a defect. Link either record
to its task, sprint, or microtask when that relationship exists.

Issue and feedback list commands accept a CLI-local positive `--limit`. Server
filters run first; the client then slices the returned records and reports:

1. `data`: the records included in this response
2. `count`: filtered records returned by the server
3. `shown`: records included in `data`
4. `hasMore`: whether records were withheld by the local bound

This envelope is returned whether or not `--limit` is passed. Read records from
`result.data`; `count` describes the full filtered server result while `shown`
describes the locally returned slice.

The CLI does not send `limit` to the hosted tool, because the hosted list
schemas do not declare that field.

### 3.6 Review request and result

#### 3.6.1 Overview

A review request (RR) is the canonical review record. Review results are
append-only entries on that record. A material finding must also be an explicit
issue linked with `--review-request <rr-id>`.

A re-review is a new child review request created with `--parent <rr-id>`. Do
not overwrite the earlier result or treat a chat acknowledgement as approval.

## 4 Routing table

### 4.1 Overview

| Need | Command family |
|---|---|
| Board CRUD and bootstrap registry | `aops pm board ...` |
| Board column placement | `aops pm column ...` |
| Operator-visible task | `aops pm ktask ...` |
| Execution window and nested plan | `aops pm sprint ...` |
| One sprint microtask | `aops pm utask ...` |
| Completion drift audit/reconcile | `aops pm status ...` |
| Defects and risks | `aops pm issue ...` |
| Improvement signals | `aops pm feedback ...` |
| Review, result, and re-review | `aops pm review-request ...` |
| Subject-aware durable resume | `aops pm handoff ...` |
| Compact multi-surface resume brief | `aops pm resume ...` |
| Design consensus | `aops discuss ...` |
| Durable memory outside a PM subject | `aops mem ...` |
| Canonical documentation | `aops doc ...` |
| Coordination and wake | `aops chat ...` |

## 5 Safe command discipline

### 5.1 Read before write

#### 5.1.1 Overview

Resolve selectors and obtain the current record before changing it. Use a full
UUID in automation; human-friendly names, slugs, and supported id prefixes are
convenient for interactive reads but can become ambiguous.

```bash
aops pm ktask get --project-slug <slug> --id <task-id> --json
aops pm sprint get --project-slug <slug> --id <sprint-id> --json
aops pm review-request get --project-slug <slug> --id <rr-id> --json
```

### 5.2 Guarded writes

#### 5.2.1 Overview

Create/update commands require `--apply`. Commands that delete or otherwise
declare a destructive operation require `--apply --confirm`.

```bash
# Validation only; no write.
aops pm issue create --project-slug <slug> --title "Example" --preview --json

# Execute the validated write.
aops pm issue create --project-slug <slug> --title "Example" --apply --json

# Destructive execution requires both guards.
aops pm issue delete --project-slug <slug> --id <issue-id> --apply --confirm --json
```

Use an idempotency key for a guarded write that may be retried by automation.
Never reuse one key with a different payload.

### 5.3 Optimistic plan updates

#### 5.3.1 Overview

`sprint update-plan` replaces or patches the nested plan according to the
provided payload. Read the current sprint first and pass
`--expected-updated-at` when concurrent edits are possible.

```bash
aops pm sprint get --project-slug <slug> --id <sprint-id> --json
aops pm sprint update-plan \
  --project-slug <slug> \
  --id <sprint-id> \
  --expected-updated-at <timestamp> \
  --phases-json '@./plan.json' \
  --preview \
  --json
```

Do not use a stale full-plan payload for a one-microtask status change. Prefer
`pm utask update` or `pm utask set-status`.

## 6 Common workflow scenarios

### 6.1 Choose the project partition

#### 6.1.1 Overview

```bash
aops project links list --json
aops pm resume --project-slug <slug> --for <agent-id> --limit 10 --json
```

### 6.2 Create and finish a single-step task

#### 6.2.1 Overview

```bash
aops pm ktask create \
  --project-slug <slug> \
  --board <board-slug> \
  --column todo \
  --title "Document the release check" \
  --apply \
  --json

aops pm ktask set-status \
  --project-slug <slug> \
  --id <task-id> \
  --status completed \
  --apply \
  --json
```

### 6.3 Create a multi-step sprint

#### 6.3.1 Overview

Create the task and sprint, then apply one reviewed plan payload:

```bash
aops pm sprint create \
  --project-slug <slug> \
  --task <task-id> \
  --name "Projectman guide release" \
  --goal "NE: Publish the guide. NICIN: Give agents canonical help. DONE-WHEN: retrieval gates pass." \
  --reference discuss:<topic-id> \
  --apply \
  --json

aops pm sprint update-plan \
  --project-slug <slug> \
  --id <sprint-id> \
  --phases-json '@./plan.json' \
  --apply \
  --json
```

Example `plan.json`:

```json
[
  {
    "name": "Implementation",
    "description": "Implement and validate the bounded slice.",
    "microtasks": [
      {
        "title": "Implement the change",
        "notes": "NE: change; NICIN: reason; DONE-WHEN: focused test passes"
      },
      {
        "title": "Validate the result",
        "notes": "NE: verify; NICIN: prevent regression; DONE-WHEN: evidence is recorded"
      }
    ]
  }
]
```

PowerShell users should quote `@file` values exactly as shown.

### 6.4 Update one microtask without replacing the plan

#### 6.4.1 Overview

```bash
aops pm utask update \
  --project-slug <slug> \
  --sprint <sprint-id> \
  --id <utask-id> \
  --status completed \
  --notes "Focused tests: 21/21 PASS." \
  --apply \
  --json
```

### 6.5 Board window lifecycle

#### 6.5.1 Overview

```bash
aops pm board kickoff \
  --project-slug <slug> \
  --board <board-slug> \
  --title "Projectman production documentation" \
  --goal "Deliver the reviewed documentation slice" \
  --apply \
  --json

aops pm board resume --project-slug <slug> --board <board-slug> --json
```

At an ordinary checkpoint, leave the window open and write resume context. Run
`board closeout` only after explicit operator approval:

```bash
aops pm board closeout \
  --project-slug <slug> \
  --board <board-slug> \
  --content "All reviewed work is complete." \
  --apply \
  --json
```

### 6.6 Record an issue or feedback item

#### 6.6.1 Overview

```bash
aops pm issue create \
  --project-slug <slug> \
  --task <task-id> \
  --sprint <sprint-id> \
  --title "List output shape needs a compatibility note" \
  --severity low \
  --source agent \
  --apply \
  --json

aops pm feedback create \
  --project-slug <slug> \
  --task <task-id> \
  --title "Add a compact issue inventory example" \
  --type improvement \
  --source agent \
  --apply \
  --json
```

Use bounded reads when scouting, then fetch the exact record:

```bash
aops pm issue list --project-slug <slug> --status open --limit 10 --json
aops pm feedback list --project-slug <slug> --status new --limit 10 --json
aops pm issue get --project-slug <slug> --id <issue-id> --json
```

### 6.7 Review request, result, finding, and re-review

#### 6.7.1 Overview

```bash
aops pm review-request create \
  --project-slug <slug> \
  --task <task-id> \
  --sprint <sprint-id> \
  --title "Review the Projectman guide" \
  --target-agent <reviewer> \
  --review-scope "sprint:<sprint-id>" \
  --instructions "Check command accuracy, safety, and retrieval." \
  --apply \
  --json

aops pm review-request result \
  --project-slug <slug> \
  --id <rr-id> \
  --reviewer <reviewer> \
  --outcome changes_requested \
  --summary "One material finding remains." \
  --issue <issue-id> \
  --apply \
  --json

aops pm issue create \
  --project-slug <slug> \
  --source review \
  --review-request <rr-id> \
  --title "Document the missing guard" \
  --severity high \
  --apply \
  --json

aops pm review-request create \
  --project-slug <slug> \
  --parent <rr-id> \
  --title "Re-review the corrected guard" \
  --target-agent <reviewer> \
  --review-scope "sprint:<sprint-id>" \
  --apply \
  --json
```

The child RR does not erase its parent. Read the canonical status/result ids
from Projectman before announcing approval elsewhere.

## 7 Cross-session handoff

### 7.1 PM-bound handoff

#### 7.1.1 Overview

Use a real Projectman subject when the next session must resume a task, sprint,
issue, or feedback record:

```bash
aops pm handoff write \
  --project-slug <slug> \
  --mode resume \
  --subject sprint \
  --id <sprint-id> \
  --content "Phase 2 is next; plan RR is accepted; no publish authority." \
  --apply \
  --json

aops pm handoff resume \
  --project-slug <slug> \
  --subject sprint \
  --id <sprint-id> \
  --strict-subject \
  --json
```

`--strict-subject` prevents a broad fallback memory search when exact PM-bound
context is required.

### 7.2 General project memory

#### 7.2.1 Overview

If there is no real PM subject, use Agentspace memory directly:

```bash
aops mem write \
  --project-slug <slug> \
  --mode resume \
  --subject project \
  --content "Current state and next action" \
  --apply \
  --json
```

Do not create a fake PM record solely to carry narrative memory.

### 7.3 Compact resume brief

#### 7.3.1 Overview

`pm resume` composes active sprint windows, an agent's review queue, open
issues/feedback, and durable memory. It is a starting index, not a replacement
for exact `get` calls.

```bash
aops pm resume --project-slug <slug> --for <agent-id> --limit 10 --json
```

## 8 Retrieval ladder

### 8.1 Overview

Use the smallest surface that can answer the question:

1. `aops pm resume --limit <n>` for a compact multi-surface starting point.
2. `aops view board|task|sprint <selector>` for read-only operator views.
3. Bounded `pm ... list` with server filters for discovery.
4. Exact `pm ... get` for the canonical record.
5. `pm status audit` for completion/review-scope drift.
6. `pm handoff resume --strict-subject` for PM-bound context.
7. Docman search/answer for durable guide semantics, not PM state.

For this guide's published Docman version:

```bash
aops doc outline get --document-version-id <docver-id> --json
aops doc search --document-version-id <docver-id> --q "review request re-review" --json
aops doc answer --document-version-id <docver-id> --q "How should a material review finding be recorded?" --json
```

Local mirrors may support local search, but hosted Projectman state remains the
canonical source for planning records.

## 9 Status audit and reconciliation

### 9.1 Overview

Run the read-only audit before claiming a slice is complete:

```bash
aops pm status audit --project-slug <slug> --task <task-id> --json
```

The audit reports task/sprint completion drift and dangling accepted review
scopes. Inspect each finding before using reconciliation:

```bash
aops pm status reconcile \
  --project-slug <slug> \
  --task <task-id> \
  --preview \
  --json
```

Do not use reconciliation to hide an invalid review reference or incomplete
evidence. Fix the canonical record or record the debt explicitly.

## 10 Anti-patterns

### 10.1 Ownership anti-patterns

#### 10.1.1 Overview

1. Treating chat as review truth instead of Projectman RR/RRR.
2. Storing design consensus only in PM notes instead of Discuss/Docman.
3. Using Projectman as a long-form memory store.
4. Creating fake tasks only to attach handoff text.

### 10.2 File-integrity anti-patterns

#### 10.2.1 Overview

1. Hand-editing `.aops/projectman/**`.
2. Treating a stale mirror as current hosted state.
3. Copying one project's cache into another project partition.

### 10.3 Command-surface anti-patterns

#### 10.3.1 Overview

1. Guessing flags instead of reading nested `--help`.
2. Sending undeclared client-only fields to strict hosted schemas.
3. Omitting `--project-slug` in a multi-project repository.
4. Using a nonexistent `pm kanban` alias; use `pm board`.
5. Running a destructive command without preview and explicit authority.

### 10.4 Planning anti-patterns

#### 10.4.1 Overview

1. Titles such as `S2`, `G12`, or a UUID with no semantic label.
2. Vague goals without `NE`, `NICIN`, and `DONE-WHEN`.
3. Phases with no microtasks.
4. Replacing a full sprint plan to update one microtask.
5. Starting consensus-backed work before its PM plan review is accepted.

### 10.5 Lifecycle anti-patterns

#### 10.5.1 Overview

1. Automatically closing a board window at an ordinary checkpoint.
2. Using `sprint set-status completed` as granular proof; it bulk rewrites
   nested microtasks.
3. Announcing a review approval without reading its canonical result id.
4. Erasing a changes-requested review instead of opening a child re-review.
5. Closing findings merely because implementation moved on.

## 11 Troubleshooting

### 11.1 A list command fails after adding a client-side option

#### 11.1.1 Overview

Strict hosted schemas reject undeclared keys. Confirm the live schema before
changing a raw payload:

```bash
aops agent schema --tool projectman.issue.list --summary --json
```

`issue list --limit` and `feedback list --limit` are intentionally client-side.
The server receives only its declared filters.

### 11.2 A bounded list says `hasMore: true`

#### 11.2.1 Overview

Increase the positive `--limit` or narrow server filters. Fetch one selected
record with `get`; do not assume the first page contains the newest or most
important item unless the command documents an order.

### 11.3 A plan update loses ids or concurrent work

#### 11.3.1 Overview

Stop writing. Read the current sprint, compare its `updatedAt`, and use a narrow
microtask update when possible. For a full plan edit, preserve entity ids and
pass `--expected-updated-at`.

### 11.4 Sprint status changed every microtask

#### 11.4.1 Overview

That is the declared behavior of `pm sprint set-status`. Restore the intended
per-item states with reviewed `pm utask update` calls and use granular updates
thereafter.

### 11.5 The target project is wrong

#### 11.5.1 Overview

Stop before writing. Inspect project links and rerun with explicit
`--project-slug`. Do not repair hosted state by editing a local mirror.

### 11.6 A review shows approved in chat but not in Projectman

#### 11.6.1 Overview

Chat is coordination only. Read the RR and its results from Projectman. A review
is accepted only when the canonical record contains the binding result.

### 11.7 A delete needs to be undone

#### 11.7.1 Overview

Do not use destructive smoke tests on material records. For a disposable test,
capture the baseline count, create one uniquely named low-severity record,
retain its exact id, delete only that id with `--apply --confirm`, and verify the
baseline count is restored.

### 11.8 Local cache and hosted state disagree

#### 11.8.1 Overview

Hosted state wins. Refresh through the supported sync/mirror command; never
hand-edit the cache. Use `aops-cli-core` for current sync and project-link
mechanics.

## 12 Existing known gaps

### 12.1 Overview

Before opening a duplicate issue, inspect the exact existing records:

1. `00d73182-997c-495f-93e9-de68ad5e0808` — sprint list limit/summary drift
2. `70dc8b67-596a-4549-8b26-e9d0b51b6a12` — task-create dedupe response
3. `bfdf5be0-1549-487e-a45a-eccfb9847a1f` — task update/short-id gap
4. `78995bdd-ba40-4553-bc15-5d8ca8f36663` — feedback source enum drift
5. `acb7e287-1fc0-49fe-9219-69e9ce329134` — list tag/concurrent microtask feedback
6. `b81eeeaf-d695-4977-8f87-0a67d1401404` — sprint status id rewrite

These references describe known debt; they do not authorize unrelated fixes.

## 13 Canonical and distribution policy

### 13.1 Overview

The hosted Docman record in `slug:aops` is the development canonical source for
the published guide. This repository file is the reviewed domain source used to
prepare it. After publishing:

1. build index and summary
2. verify search and citation-first answer
3. mirror-pull into `.aops/docman/domain-guides/projectman-user-guide.md`
4. materialize identical bytes into
   `aops/assets/docs/user-guides/projectman-user-guide.md`

The guide must be tagged/grouped as a public asset. The repository-local
`.aops` mirror remains derived and is not committed.

## 14 Appendix A. Verified command catalog

### 14.1 Overview

This catalog is hand-authored from the post-change CLI help. It is not a
generated region. Run the nested help for exact flags.

| Family | Commands |
|---|---|
| `board` | `bootstrap`, `list`, `get`, `archive`, `unarchive`, `resume`, `kickoff`, `create`, `closeout`, `delete` |
| `column` | `list`, `add`, `remove`, `reorder` |
| `ktask` | `list`, `get`, `create`, `set-status`, `delete` |
| `sprint` | `list`, `get`, `create`, `update-plan`, `set-status`, `archive`, `unarchive`, `delete` |
| `utask` | `create`, `update`, `set-status`, `delete` |
| `status` | `audit`, `reconcile` |
| `issue` | `list`, `get`, `create`, `update`, `delete` |
| `feedback` | `list`, `get`, `create`, `update`, `delete` |
| `review-request` | `list`, `get`, `create`, `update`, `result`, `delete` |
| `handoff` | `resume`, `write` |
| root `pm` | `resume` |

```bash
aops pm --help
aops pm board --help
aops pm board bootstrap --help
aops pm column --help
aops pm ktask --help
aops pm sprint --help
aops pm utask --help
aops pm status --help
aops pm issue --help
aops pm feedback --help
aops pm review-request --help
aops pm handoff --help
```

## 15 Appendix B. Hosted schema fallback

### 15.1 Overview

Use schema discovery when sugar is missing or when implementing a sugar
command. Never guess a hosted tool id or input shape:

```bash
aops agent tools --domain projectman --summary --json
aops agent schema --tool projectman.issue.list --summary --json
```

Prefer the typed `pm` sugar for routine Projectman work. Raw invoke is an
advanced fallback and does not relax ownership, guard, or project-selection
rules.
