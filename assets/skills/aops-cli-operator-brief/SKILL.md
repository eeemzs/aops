---
name: aops-cli-operator-brief
version: 5
description: "A plain-language status brief: summarize AOPS PM, memory, discussion, or collaboration history in non-technical language for a person asking what happened, what works, what was decided, or what remains open."
metadata:
  supersedes: "v4"
  short-description: "Plain-language AOPS status summary guide"
  tags:
    - aops
    - summary
    - operator-brief
    - projectman
    - agentspace
    - discuss
    - collab
---

# AOPS Operator Brief

Use this skill when the operator asks for a plain-language status summary of a mentioned AOPS subject: a task, sprint, board, memory thread, discussion topic, (historical) collab session, bug, feature, or "what did we do earlier?" style request.

This skill carries its own read order and output rules below; it needs nothing else. When your project keeps a reusable status-brief prompt of its own, you may use it as well, but follow this skill when the two disagree.

## Read Order

1. Parse the operator's wording and extract topic keywords or ids.
2. Start with PM views:
   - `aops-cli view dashboard --style agent`
   - `aops-cli view task <selector>`
   - `aops-cli view sprint <selector>`
   - `aops-cli view digest --task|--sprint|--board <selector> --depth deep`
3. Read memory records in this priority:
   - closeout
   - durable decision
   - handoff or resume checkpoint
   - kickoff
4. Read session records only after PM/memory gives the right ids:
   - `aops-cli discuss get --id <topic-slug> --json`
   - Historical collab session records, if any, are read-only files under `.aops-cache/agentspace/collabs/**`; read the files directly.
5. Read review issues, feedback, commit summaries, logs, or browser proof only when they answer the operator's exact question.

## Briefing Discipline

- Write for the operator, not for the codebase.
- Lead with what worked, what changed, and what decision was made.
- Translate technical terms:
  - `cwd` -> selected repo / working folder
  - `threadId` -> Codex's local conversation handle
  - `pm review-result` (or a historical `collab review-result` record) -> reviewer checked it
  - `closeout` -> the work was closed with approval
- Keep ids as follow-up handles, not as the main story.
- If the operator asks "can we use Codex's own session?", answer in product terms: same-machine hint is okay; cross-machine native resume is not assumed.
- If there is a browser/desktop split, say which mode supports which behavior.

## Output

Use short bullets in the user's language. Prefer sections like:

- Overall status
- What was done, from the user's point of view
- Does it work
- Decisions
- What is left
- Sources

Avoid long technical explanations unless the operator explicitly asks.
