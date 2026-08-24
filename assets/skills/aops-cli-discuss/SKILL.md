---
name: aops-cli-discuss
version: 3
description: "Use when an AI agent runs a standalone server-canonical AI discussion topic with aops-cli: the design-decision ritual (independent research, two-agent turn protocol, kind=final-stance, deterministic conclude outputs), slug-first selectors, and JSON-driven stop-state reads. discuss = the decision/consensus surface only; coordinate/wake via aops-cli-chat, track review + execution truth via aops-cli-projectman. Thin discipline guide; canonical mechanics live in .aops/docman/domain-guides/agentspace-user-guide.md and command --help."
metadata:
  supersedes: "v2"
  short-description: "AOPS CLI standalone discuss / decision-ritual guide"
  tags:
    - cli
    - agentspace
    - discussion
    - discuss
    - decision-ritual
    - consensus-gate
    - two-agent-protocol
    - conclude-outputs
    - standalone
---

# AOPS CLI Discuss

`aops-cli discuss` manages **standalone AI discussion topics** that are server-canonical (hosted discuss authoring): create, append turns, and read through the hosted gateway. `.aops/agentspace/discussions/**` is a read-only local cache of that server state, refreshed by `sync pull`. discuss is the decision/consensus surface in the AOPS coordination triad:

- **`aops-cli chat`** (hosted rooms/channels) = coordination, wake, catch-up → `aops-cli-chat`.
- **`aops-cli pm`** (review-request/result, issue, feedback) = review + execution truth → `aops-cli-projectman`.
- **`aops-cli discuss`** (standalone) = consensus / decision protocol → this skill.

There is **no automatic decision bridge**: `discuss conclude` only writes its own conclude outputs. Surfacing a concluded decision to PM or a chat room is an **explicit** step (see "Explicit linking" below). `discuss` is standalone-only — there is no `--session` flag and no session ledger.

This skill is a **thin discipline guide**: discovery, the design-decision ritual, the two-agent automation protocol, slug-first selectors, anti-patterns, and pointers. Deep mechanics, full exit-code matrices, and troubleshooting live in the canonical Agentspace User Guide. When this skill conflicts with `aops-cli discuss --help` or the user guide, **those win**.

## Roles are operator-only (read first)

Participant roles for any discussion — initiator/driver, peer, primary, reviewer — are **operator-driven**. Do not auto-default by agent name (`codex`, `claude`), runtime brand (Codex CLI, Claude Code), or prior-session convention; the literal `codex`/`claude` ids in the examples below are illustrative pairs, **not defaults**. If the operator did not specify who plays which role, **ASK** before starting the topic or writing the first turn.

## Use another skill for

| Need | Skill |
|------|-------|
| Coordination / wake / catch-up (hosted rooms, DMs) | `aops-cli-chat` |
| Review-request/result, re-review, issues, feedback | `aops-cli-projectman` |
| CLI guard flags, sync, hosted mirror mechanics | `aops-cli-core` |
| Document graph authoring/search | `aops-cli-docman` |
| Memory / project / prompt / resource / skill assets | `aops-cli-agentspace` |
| File snapshots / diff / restore | `aops-cli-fileman` |

## Canonical sources (authoritative)

When this skill is silent, ambiguous, or out of date, **defer to these**:

1. `.aops/docman/domain-guides/agentspace-user-guide.md` — semantics for discuss topics, turns, the decision ritual, and loop discipline. Reference sections by name (numbers drift), e.g.:
   - "Coordination semantics (loop discipline and exit codes)" (search: exit code, loop discipline, wait, self-wakeup)
   - "Coordination semantics (review-request reply pairing)" (search: review-result, projectman.review-request, discussion-topic ref)
   - "Anti-patterns appendix" (search: anti-patterns, lifecycle, decision ledger)
   - "Troubleshooting" (search: troubleshooting, ready-to-conclude, open question block)
2. `aops-cli discuss --help` and nested subcommand help — flag-level detail and the current argument schema.
3. Section-focused reading via the doc discovery ladder (see **Pointers** below).
4. Read-only local cache: `.aops/agentspace/discussions/topics/<slug>-<topic-slug>/` (`topic.md`, `turns/*.md`, `outputs/*.md`) mirrors server state for offline reads; author through the hosted CLI, not by hand-editing the cache.

## Discovery: which command for what

| Need | Command |
|------|---------|
| Start a standalone decision topic | `aops-cli discuss start --title <t> --slug <topic-slug> --agent <a> --agent <b> --question <q> --apply --json` |
| Child topic referencing a prior one (parent unchanged) | `aops-cli discuss follow-up --from <topic-slug> ... --apply --json` |
| Branch/alternative from an existing topic (parent unchanged) | `aops-cli discuss fork --from <topic-slug> ... --apply --json` |
| Child-topic lineage tree for one root | `aops-cli discuss lineage --id <topic-slug> --json` |
| List topics (filter by status/scope/agent/subject) | `aops-cli discuss list --status active --scope standalone --json` |
| Full topic with turns + outputs | `aops-cli discuss get --id <topic-slug> --json` |
| Append one turn | `aops-cli discuss turn --topic <topic-slug> --agent <a> --kind <kind> --from-file <path> --apply --json` |
| Deterministic state (next speaker, open questions, guards) | `aops-cli discuss status --id <topic-slug> --json` |
| Poll until your turn / block / ready | `aops-cli discuss wait --id <topic-slug> --for <a> --timeout-sec <s> --interval-sec <s> --json` |
| Persistent automation loop prompt | `aops-cli discuss loop-prompt --id <topic-slug> --for <a> --json` |
| Non-interpretive context pack | `aops-cli discuss digest --id <topic-slug> --json` |
| Write conclude scaffolds, move to `concluding` | `aops-cli discuss conclude --topic <topic-slug> --apply --json` |
| Mark an invalid/unwanted topic abandoned | `aops-cli discuss abandon --topic <topic-slug> --apply --json` |

Turn kinds: `statement | question | answer | objection | concession | proposal | final-stance`. Discuss topics are server-canonical (hosted authoring); `.aops/agentspace/discussions/**` is a read-only cache refreshed by `sync pull`. There is no auto-promote into memory/experience — promotion stays explicit.

## Mandatory design-decision ritual (standalone)

Use this whenever the operator asks for a collaborative architecture/design discussion before implementation, a peer consensus, or implementation only after another agent has independently argued/reviewed the design.

1. **Start the topic** with at least two unique operator-assigned participants; give the operator the topic slug and a paste-ready peer join prompt (`discuss status --id <slug> --prompt-for-next --json`) before waiting.
2. **Independent research before the first substantive turn.** Tell the peer to research and form its own position, not merely validate the opener's plan. Each participant grounds its opening turn in its own reading.
3. **At least four substantive non-final turns** (analysis / proposal / objection / question / answer / concession). `final-stance` turns, bare acknowledgements, and operator-addressed questions do **not** count toward that floor. A `--require-question-answer` topic blocks other turn kinds while an operator/peer question is open — answer it first.
4. **Each participant files a `kind=final-stance` turn.** This append-only turn under `turns/*.md` is the canonical final stance — not the `outputs/<agent>-final-stance.md` scaffold that `conclude` later writes.
5. **Confirm completeness from JSON, never prose.** Read `discuss status --id <slug> --json`: check `lifecycleState` and `missingTurnFinalStances` (and `outputs.missingFinalStances` when present). Do not infer "everyone is done" from turn text or shell wrapper wording.
6. **Only after all final stances exist, run `discuss conclude`.** It scaffolds `outputs/<agent>-final-stance.md`, `consensus.md`, `disagreement.md`, `open-questions.md` and moves the topic to `concluding`. `conclude` is **blocked while an open question remains**. It does **not** finalize the discussion — the **initiator/driver** (the recorded output owner) then reviews and finalizes `consensus.md` / `disagreement.md` / `open-questions.md` with **no `_TBD_` placeholders**. Either agent may run `conclude`; running it does not lock ownership.
7. **Implementation stays blocked** until the operator explicitly approves the consensus or grants a recorded override. If the operator changes the ritual mid-discussion, treat the stricter rule as binding even if the topic opened with a weaker `--min-turns-before-conclude`.

## Workflow: standalone discuss → conclude

```bash
aops-cli discuss start --title "<decision>" --slug <topic-slug> \
  --agent codex --agent claude --question "<architecture question>" \
  --min-turns-before-conclude 4 --apply --json
aops-cli discuss turn --topic <topic-slug> --agent claude --kind statement --from-file ./context.md --apply --json
aops-cli discuss wait --id <topic-slug> --for codex --timeout-sec 540 --interval-sec 5 --json
# iterate substantive turns (statement/proposal/objection/question/answer/concession),
# reading `discuss status --json` between waits, until >=4 non-final turns exist
aops-cli discuss turn --topic <topic-slug> --agent claude --kind final-stance --from-file ./final-claude.md --apply --json
aops-cli discuss turn --topic <topic-slug> --agent codex  --kind final-stance --from-file ./final-codex.md  --apply --json
aops-cli discuss status --id <topic-slug> --json        # confirm missingTurnFinalStances is empty
aops-cli discuss conclude --topic <topic-slug> --apply --json
# then the initiator/driver finalizes outputs/consensus.md|disagreement.md|open-questions.md (no _TBD_)
```

Read `lifecycleState`, `nextTurn`, `missingTurnFinalStances`, and `outputs.missingFinalStances` from JSON; never infer stop state from prose. Output ownership defaults to the discussion **driver** (often, but not always, the topic initiator — the driver may differ from a participant who merely ran `conclude`).

## Two-agent automation protocol

When two agents drive a topic on their own, each loops on `discuss wait` keyed to its own id. Read the structured `exitCode` (same value as the process exit code and as `result.wait.exitCode`):

| Exit | Meaning | Do |
|------|---------|----|
| `0` | Requested agent may write the next turn | Write `discuss turn ... --agent <you> --expect-next <you> ...`, then re-`wait`. |
| `20` | Blocked by an operator-addressed open question/block | **Stop and return to the operator** — an agent cannot clear an operator block. |
| `21` | Ready to conclude / concluding / concluded / abandoned | Do **not** write more turns; proceed to `conclude` (or stop if already concluded/abandoned). |
| `22` | Timeout, no change | Re-poll (`wait` again) if still waiting; otherwise do other non-overlapping work. |

`--expect-next` is a race guard and **must equal `--agent`**: it requires that the deterministic next turn already allows that agent, so two automated writers never collide. On a sequence collision, re-read `discuss status` and retry. `discuss loop-prompt --id <slug> --for <agent>` prints a ready-made persistent loop for operator-started automation.

## Explicit linking (no auto bridge)

A concluded decision does **not** propagate anywhere by itself. Surface it explicitly:

- **To Projectman** — carry the discussion-topic ref into a PM record so review/execution truth references the decision:

  ```bash
  aops-cli pm feedback create --title "<decision applied>" \
    --description "Consensus from discussion-topic <topic-slug>: <decision summary>" --apply --json
  aops-cli pm issue create --title "<follow-up from decision>" --severity <sev> \
    --description "Per discussion-topic <topic-slug> consensus" --apply --json
  ```

- **To a chat room** — bind the topic so the room shows where the decision lives (rooms are FLOW, not the decision ledger):

  ```bash
  aops-cli chat binding add --room-id <id> --binding-type agentspace.discussion-topic \
    --ref-id <topic-uid> --title "<decision>" --created-by <agent> --apply --json
  ```

This replaces the retired `collab start --from-discuss` / conclude-writeback path. There is no implicit promotion into memory either — write a memory item with a `agentspace.discussion-topic` source-ref if the decision is durable (see `aops-cli-agentspace`).

## Slug-first selector contract

1. Operator text, handoffs, and links use the **topic slug** (`--id` / `--topic`), not raw UUIDs; raw UUIDs stay in JSON/debug.
2. A legacy folder-name or implicit short-id match emits `cliDeprecationWarnings` in JSON — treat that as a signal to rewrite future instructions to the slug.
3. If a slug is ambiguous, run `aops-cli discuss list --status active --agent <agent> --limit <n> --json` and retry with `--short-id <8char>` from the matching row. Use `--prefer-active` only when an ambiguous selector has exactly one active match.

## Pointers

- **Agentspace User Guide** by document title + **section name** + keywords (not bare numbers): "Coordination semantics (loop discipline and exit codes)" (search: exit code, wait, loop discipline), "Coordination semantics (review-request reply pairing)" (search: discussion-topic ref, review-result), "Anti-patterns appendix" (search: decision ledger, lifecycle), "Troubleshooting" (search: ready-to-conclude, open question block).
- **Help is canonical:** `aops-cli discuss --help`, then `aops-cli discuss start|turn|wait|status|conclude|list|follow-up|fork|lineage|loop-prompt|digest|abandon --help`.
- **Doc discovery ladder** (section-focused reading, no id needed): `aops-cli doc scope search --project-slug aops --q "<keyword>" --local --json`, then targeted `aops-cli doc search --document-version-id <docver-id> --q "<keyword>" --local --json` and `aops-cli doc outline get --document-version-id <docver-id> --json`. There is no `aops-cli docman … --slug` command — use the ladder; reference sections by title + section name + keywords, not bare numbers.
- **Tool Input Schema:** before authoring `--data`/`--input`/`--patch` payloads for any hosted write, fetch the live JSON Schema first (`aops-cli agent schema --tool <domain>.<operation>`). Full explanation in `aops-cli-core` (Tool Input Schema section).

## Top anti-patterns

1. **Implementing before the operator approves the consensus** (or before a recorded override).
2. **Inferring stop-state from prose** instead of reading JSON `lifecycleState` / `missingTurnFinalStances` / `wait` `exitCode`.
3. **Leaving `_TBD_` placeholders** in `consensus.md` / `disagreement.md` / `open-questions.md` after `conclude`; the output owner must finalize them.
4. **Concluding before all `kind=final-stance` turns exist**, or treating the `outputs/<agent>-final-stance.md` scaffold as the canonical stance instead of the `turns/*.md` final-stance record.
5. **Treating a chat room (or chat in general) as the decision ledger** — the durable decision is the discuss topic + its conclude outputs; bind it into chat, do not decide only in chat.
6. **Auto-defaulting participant roles** by agent name / runtime brand / convention instead of taking them from the operator.
7. **Setting `--expect-next` to a different agent than `--agent`**, or ignoring a `20` (operator block) by trying to write through it.
8. **Hand-editing the `.aops/agentspace/discussions/**` cache** as if it were authoring truth; it is a read-only mirror of server state — author through the hosted CLI and refresh with `sync pull`.

---

If `--help` and this skill disagree, **`--help` wins** (canonical command surface). If the user guide and this skill disagree, **the user guide wins**.
