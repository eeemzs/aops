---
name: aops-cli-projectman
version: 20
description: "Use when an AI agent needs the AOPS CLI Projectman playbook: server-canonical boards, tasks, sprints, microtasks, issues, feedback, review requests/results/re-reviews, status audit, and PM-bound handoff."
metadata:
  supersedes: "v18"
  short-description: "AOPS CLI Projectman thin discipline guide"
  tags:
    - cli
    - projectman
    - planning
    - kanban
    - sprint
    - issue
    - feedback
    - review-request
    - handoff
    - server-canonical
    - help-first
---

# AOPS CLI Projectman

Projectman owns execution truth: boards, tasks, sprint plans, microtask status,
issues, feedback, and canonical review requests/results.

This is a thin agent guide. Exact flags live in `aops pm ... --help`; normative
workflow lives in the current `Projectman User Guide`; the running server schema
wins for raw hosted payloads.

## Start Here

1. Run `aops host health --json`.
2. Run `aops project links list --json` and choose the intended project.
3. Start with a bounded resume/inventory read, not a full unfiltered dump.
4. Read the exact record and nested `--help` before a write.
5. Use `--preview` where supported, then `--apply`; delete needs
   `--apply --confirm`.
6. Treat `.aops-cache/projectman/**` as a read-only derived cache.

The installed launcher may be `aops`; the package launcher is `aops-cli`.

`--limit` on `pm issue list` and `pm feedback list` requires a CLI build newer
than `@aopslabs/aops` 0.3.31. Installed 0.3.31 rejects that flag; until the next
CLI release, omit it or use the repository-built CLI. Their response envelope
always stores records in `data` alongside `count`, `shown`, and `hasMore`.

## Use This Skill For

- board and column lifecycle;
- kanban tasks and sprint/microtask plans;
- issues and feedback;
- review request, append-only result, and child re-review records;
- completion/review-scope audit and reconciliation;
- Projectman-bound cross-session handoff.

Use `aops-cli-discuss` for design consensus, `aops-cli-docman` for canonical
written knowledge, `aops-cli-agentspace` for durable memory, and
`aops-cli-chat` for coordination/wake.

## Canonical Sources

1. Hosted `Projectman User Guide` in `slug:aops`, group `domain-guides`,
   document `0683979e-8b57-4632-8f88-e021b19ec70c`, current version
   `99d3b4f8-c6aa-4ca8-9fe5-4751bc06a3a9`.
2. Repo mirror:
   `.aops-cache/docman/domain-guides/projectman-user-guide.md` (read-only).
3. Domain-development source: `domains/projectman/USER_GUIDE.md`.
4. Exact flags: `aops pm --help` and the smallest nested `--help`.
5. Raw tool discovery: `aops agent tools --domain projectman --summary --json`
   and `aops agent schema --tool <tool-id> --summary --json`.

When this skill and help disagree, help wins for flags. When this skill and the
guide disagree, the published current guide wins for workflow.

## Workflow 0: Select the Project Partition

```bash
aops project links list --json
aops pm resume --project-slug <slug> --for <agent-id> --limit 10 --json
```

Always pass `--project-slug` for a material write when more than one hosted
project is linked. The local link resolves the destination; the hosted server
record remains canonical.

## Retrieval Ladder

Use the smallest useful surface:

1. compact context:

   ```bash
   aops pm resume --project-slug <slug> --for <agent-id> --limit 10 --json
   ```

2. bounded inventories:

   ```bash
   aops pm sprint list --project-slug <slug> --status doing --limit 10 --summary --json
   aops pm issue list --project-slug <slug> --status open --limit 10 --json
   aops pm feedback list --project-slug <slug> --status new --limit 10 --json
   ```

3. exact record:

   ```bash
   aops pm sprint get --project-slug <slug> --id <sprint-id> --json
   ```

4. status truth:

   ```bash
   aops pm status audit --project-slug <slug> --task <task-id> --json
   ```

5. deep guide search:

   ```bash
   aops doc search --document-version-id 99d3b4f8-c6aa-4ca8-9fe5-4751bc06a3a9 --q "<question>" --json
   aops doc answer --document-version-id 99d3b4f8-c6aa-4ca8-9fe5-4751bc06a3a9 --q "<question>" --json
   ```

## Model

1. Board = long-lived work stream.
2. Kanban task = operator-visible unit of work.
3. Sprint = bounded execution window.
4. Phase = grouped section inside a sprint.
5. Microtask = smallest explicit status/evidence owner.
6. Issue = concrete defect, risk, or blocker.
7. Feedback = observation or improvement signal.
8. Review request = canonical review record; results are append-only entries.

## Workflow 1: Multi-Step Sprint

```bash
aops pm ktask create --project-slug <slug> --board <board> --column todo \
  --title "<semantic task>" --apply --json

aops pm sprint create --project-slug <slug> --task <task-id> \
  --name "<semantic sprint>" \
  --goal "NE: <work>; NICIN: <why>; DONE-WHEN: <acceptance>" \
  --reference discuss:<topic-id> --apply --json

aops pm sprint update-plan --project-slug <slug> --id <sprint-id> \
  --phases-json '@./plan.json' --apply --json

aops pm utask update --project-slug <slug> --sprint <sprint-id> \
  --id <utask-id> --status completed --notes "<evidence>" --apply --json
```

Read the current sprint first. Use `--expected-updated-at` for a full plan edit
when concurrent work is possible. Prefer `utask update` for one microtask.

## Workflow 2: Review and Re-Review

```bash
aops pm review-request create --project-slug <slug> --task <task-id> \
  --sprint <sprint-id> --title "<review>" --target-agent <reviewer> \
  --review-scope "sprint:<sprint-id>" --instructions "<checks>" --apply --json

aops pm review-request result --project-slug <slug> --id <rr-id> \
  --reviewer <reviewer> --outcome changes_requested \
  --summary "<finding summary>" --issue <issue-id> --apply --json

aops pm issue create --project-slug <slug> --source review \
  --review-request <rr-id> --title "<material finding>" \
  --severity high --apply --json

aops pm review-request create --project-slug <slug> --parent <rr-id> \
  --title "<re-review>" --target-agent <reviewer> \
  --review-scope "sprint:<sprint-id>" --apply --json
```

Chat may announce the outcome; Projectman RR/status/result id is canonical.

## Workflow 3: Cross-Session Handoff

```bash
aops pm handoff write --project-slug <slug> --mode resume \
  --subject sprint --id <sprint-id> --content "<state and next action>" \
  --apply --json

aops pm handoff resume --project-slug <slug> --subject sprint \
  --id <sprint-id> --strict-subject --json
```

Use `aops mem write` directly when no real Projectman subject exists. Do not
create a fake task solely to hold narrative memory.

## Always Rules

1. Always author Projectman through the hosted gateway; never hand-edit
   `.aops-cache/projectman/**`.
2. Always select the intended project explicitly when multiple links exist.
3. Always read nested `--help`; fetch live JSON Schema before raw invoke or
   sugar implementation.
4. Always use write guards: `--preview`, then `--apply`; destructive operations
   require `--apply --confirm` and explicit authority.
5. Never run `pm board closeout` automatically; it is an operator decision.
6. Remember `pm sprint set-status` rewrites all nested microtasks; use
   `pm utask update` for granular evidence.
7. Keep RR/RRR canonical in Projectman and create/link every material review
   finding explicitly as an issue.
8. Keep Projectman thin; durable narrative context belongs in Agentspace
   memory, while PM holds scope, status, refs, acceptance, and short evidence.
9. Use semantic titles and `NE`, `NICIN`, `DONE-WHEN`; bind material Discuss
   consensus to the plan and obtain the required plan RR before implementation.

## Top Anti-Patterns

- broad unfiltered list reads when a bounded/filterable read is enough;
- opaque codes as operator-facing titles;
- a phase with no microtask;
- replacing a full plan to update one microtask;
- treating chat acknowledgement as review approval;
- overwriting a changes-requested RR instead of opening a child RR;
- using local cache edits to repair hosted state;
- creating duplicate issues without checking exact existing records.

## Troubleshooting

- Wrong project: stop, inspect links, rerun with `--project-slug`.
- Undeclared input rejected: inspect `agent schema`; keep client-only fields out
  of the hosted payload.
- `hasMore: true`: raise positive `--limit` or narrow server filters, then use
  exact `get`.
- Plan ids changed: stop full-plan writes; read current sprint and use narrow
  microtask CRUD.
- Every microtask status changed: `sprint set-status` is bulk by design.
- Chat says approved but PM does not: read the canonical RR and result ids.
- Mirror disagrees: hosted state wins; refresh through supported sync/pull.

For full recipes and known exact issue ids, search the current guide.
