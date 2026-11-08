---
name: aops-working-disciplines
version: 11
description: "Choose and apply an AOPS working discipline: solo-pm-loop for one agent (aops pm, aops mem and aops doc, optional review requests for later), build-review-chat for an implementer with a reviewer, design-first-consensus for decisions settled in aops discuss before building, coordinator-loop for one agent directing several. The user's own working discipline comes first; this skill says how to follow one without mixing, and how to write one for the user. Covers the solo work rhythm, verify-first stances, consensus-to-plan binding and the closeout checklist."
metadata:
  supersedes: "v10"
  short-description: "AOPS working disciplines and the solo work rhythm"
  tags:
    - aops
    - discipline
    - solo
    - projectman
    - review-request
    - discuss
    - memory
    - closeout
---

# AOPS Working Disciplines

A working discipline is the rhythm of a session: who builds, who reviews, and when decisions are settled. The kickoff interview (`aops-interactive`) usually chooses it. Disciplines are role-agnostic: they name roles (implementer, reviewer, coordinator), never agents.

| Discipline | Use when | Review |
|---|---|---|
| `solo-pm-loop` | one agent; low or medium uncertainty; the default | optional review requests for a later session |
| `build-review-chat` | an implementer with a live reviewer, in tmux or an aops chat room | a review request per slice; commit after acceptance |
| `design-first-consensus` | high design uncertainty, or a decision costly to reverse | aops discuss final stances before implementation |
| `coordinator-loop` | the user delegates the session to one agent that directs implementers | the coordinator reviews every slice |

Paired and multi-agent setup (tmux, chat room, waking the reviewer) lives in `aops-collaborative-work`. `aops start --discipline <id>` can record the discipline in a PM-backed kickoff; it is optional.

## The user's own working discipline

A user may have a working discipline of their own: an ordinary skill they wrote, whose description says it is their working discipline. It comes before the AOPS disciplines.

- **Which one.** Follow the discipline that the user's request or the repository rules (`AGENTS.md`, `CLAUDE.md`) name. When neither names one, follow an installed one whose description says it applies to this work, and say so. When it is unclear, or several could apply, ask the user once which one to follow.
- **No mixing.** While you follow it, do not also apply an AOPS discipline: its rules decide plans, reviews, checkpoints and discuss topics. The AOPS skills still give the commands it asks for. Ask once about what it leaves open.
- **Limits.** Repository rules and the user's own approvals (commit, push, publish) still hold. `aops start` composes only the four AOPS disciplines; while you follow the user's discipline, do not run it.

When the user asks you to write one for them:
1. Write `<name>/SKILL.md`: a `name`, a `description` that says it is the user's working discipline and when it applies, and short numbered rules in the user's words. The name does not start with `aops`; AOPS updates leave such skills alone.
2. Put the folder where the user's agents read skills: `~/.claude/skills/<name>/` for Claude and `~/.codex/skills/<name>/` for Codex.
3. For one repository, add a line to its `AGENTS.md` or `CLAUDE.md`, outside any block AOPS manages: `Working discipline: <name>`.
4. Tell the user to start a new agent session, so that it loads the skill.

## solo-pm-loop: the solo rhythm

The kickoff chose three things for the session: whether aops pm is on, whether checkpoint notes are on, and whether review requests are left for later. Follow those answers for the whole session. Repository rules come first: when the repository requires aops pm or reviews, they stay on.

1. **Register before code (aops pm on).** The task exists in aops pm before implementation. Multi-step work gets a plan; small work stays a single task.

   ```bash
   aops pm ktask create --board <board> --column <column> --title "<title>" --description "<what and why>" --apply --json
   aops pm sprint create --task <task-id> --name "<plan>" --goal "<goal>" --apply --json
   ```

   With aops pm off, keep the plan as a short list in the conversation; nothing is written to aops pm.
2. **Memory.** `brief` reads at start or resume, whatever the choices:

   ```bash
   aops mem brief --subject project --json
   ```

   With checkpoint notes on, write a checkpoint at decisions, finished steps, blockers and handovers (not after every command), and a summary at the end of the session. Every note gets its own readable title, never an id:

   ```bash
   aops mem checkpoint --title "<milestone>" --content "<result, evidence, next action>" --task-id <task-id> --apply --json
   aops mem summary --title "<session>" --content "<what changed, what is next>" --apply --json
   ```

   With checkpoint notes off, write no notes unless the user asks for one.
3. **Small verified slices.** With aops pm on:
   - Move the task to `doing` when work starts.
   - Mark each step `completed` only with its own evidence.
   - Before `done`, run `aops pm status audit --task <task-id> --json`.
   Writes need `--apply`; destructive commands also need `--confirm`.
4. **Problems and ideas (aops pm on).** Defects become issues; ideas and user guidance become feedback. With aops pm off, report them to the user instead.
5. **Review for later (only when the user chose it).** Leave a review request for each finished piece. Another agent picks it up in a later session. Review requests live in aops pm. If the user wants them while aops pm is off, say so and ask whether to turn aops pm on; do not turn it on yourself.

   ```bash
   aops pm review-request create --task <task-id> --review-scope "task:<task-id>" --title "<piece>" --apply --json
   aops pm review-request list --status requested --json   # what waits for review
   ```

   A review request starts no agent and wakes nobody. It waits for the next reviewer.

## Verify-first stance

When a reported problem may be large, each active role first records a short stance:
- Is the problem real?
- Which source was checked (code, PM, documents, logs)?
- Is a formal decision needed?
- What is the likely risk or scope?

Two stances that agree the work is small continue as solo-pm-loop or build-review-chat. A material, cross-owner or costly-to-reverse decision goes to aops discuss first.

## Consensus to plan

1. aops discuss holds the final stances and the concluded consensus.
2. Carry the consensus reference into the aops pm task and its plan. Write the plan in plain words: what, why, and when it is done.
3. The reviewer approves the plan before implementation starts.
4. Where the repository requires the user's approval of a consensus, wait for it.

design-first-consensus always uses this gate. build-review-chat and coordinator-loop use it when a material decision appears mid-work.

## build-review-chat

- One review request per slice, scoped to the plan or its phase (`sprint:<id>` or `sprint-phase:<id>/<phase-id>`). Wake the reviewer where it works.
- Commit after the review is accepted, with an explicit list of the reviewed files.

## coordinator-loop

The rhythm:
1. The user makes a request.
2. The coordinator does independent research in code, PM and documents.
3. It creates the task and the plan.
4. It assigns work with those references.
5. Implementer slices arrive, each with a review request.
6. The coordinator reviews, verifying in the code or runtime, not from status text.
7. It instructs the commit.

Rules:
- Implementers bring questions to the coordinator; only decisions that belong to the user go to the user.
- An assignment always carries its task and plan references; chat prose alone is not an assignment.
- Between assignments, the coordinator improves documents, skills or tooling and files findings as issues or feedback.

## design-first-consensus

Run an aops discuss topic for the decision (`aops-cli-discuss`):
1. Each agent researches independently.
2. At least four substantive turns follow.
3. Each agent files a final stance.
4. You conclude.

No final stances means no conclusion. Concluded outputs have no `_TBD_` placeholders. Then bind the consensus into the plan as above.

## Closeout checklist

Before leaving a session, or handing it over:
- With checkpoint notes on: write a handoff or checkpoint note with the next action, the validation state and references, and a short session summary.
- With aops pm on: triage open review requests (accepted, a follow-up issue, or deferred with an owner), triage open issues the same way, and leave the task and plan status truthful.
- Otherwise tell the user in the conversation what is done, what was checked and what comes next.

Board and room closeout belong to the user.
