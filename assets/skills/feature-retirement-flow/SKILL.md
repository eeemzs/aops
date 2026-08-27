---
name: feature-retirement-flow
description: Use when a feature is being removed from an AOPS repository set rather than changed — retiring a domain capability, deleting a surface that shipped, or taking a concept out of the product. Covers the measured residue inventory, the three touch classes and their different rules, the irreversible-schema gate, the dependant-republish cascade, and the proofs that close it. Not for refactors, deprecations that keep the code, or feature flags.
short-description: Measured, gated removal of a shipped feature across repos
---

# Feature retirement flow

Removing a feature is not a large refactor. A refactor keeps the behaviour and moves the code; a retirement destroys behaviour that shipped, and everything that consumed it has to be brought to a coherent state in the same release. The failure mode is not a compile error — it is a half-removed concept that still has a table, a route, a published type, or a skill telling agents to use it.

This flow exists to make the removal **measured, classified, and provable**. Follow it in order. Every step produces a number or an artifact that the next step consumes.

## 0. Establish that retirement is the right shape

Retirement means: the capability goes away, and no consumer is expected to migrate to a replacement inside this change. If a replacement exists and consumers must move to it, that is a migration and it needs a deprecation window — a different flow. If the code stays behind a flag, that is not retirement either.

Write down, in the tracking record, one sentence: *what stops being possible after this lands*. If that sentence is hard to write, the scope is not a retirement yet.

## 1. Measure the residue, from inside each repository

Run the inventory from **inside each repository root**, not from a parent directory. Measuring from a parent silently includes sibling trees and produces numbers you cannot reproduce later; a scope estimate you cannot reproduce is worse than no estimate, because plans get built on it.

```
grep -rIlE '(^|[^A-Za-z])<concept>|[a-z]<Concept>' . \
  | grep -vE 'node_modules|(^|/)dist/|(^|/)build/|\.svelte-kit|pnpm-lock|(^|/)\.git/'
```

**Calibrate the pattern in both directions before you trust a number**, because
the two obvious spellings each fail in an opposite way. `\b<concept>\b` misses
camelCase — it will not find `RepositoryFactoryMission` — and a bare
`<concept>` over-matches: searching for `mission` returns every `permission` in
the tree. So the pattern above asks for a non-letter before the lowercase form
and matches the capitalised form only after a lowercase letter, and it does not
use `-i`, which would re-introduce the `permission` problem. Prove both
directions on real files: one hit you know should be found, one you know should
not.

**The exclusion list needs `(^|/)` on every segment.** A pattern like `/dist/`
only matches when the directory is nested, because the tool emits root-level
paths without a leading `./` — `drizzle-out/meta/x.json`, not
`./drizzle-out/meta/x.json`. A repository-root `dist`, `build`, `drizzle-out` or
`.git` slips straight through, and `.git` is at the root of every repository. And
test the filter by **running the producing command and piping one of its real
lines through it** — checking it against a path you typed from memory validates
your idea of the output, not the output.

Record, per repository: the total, and the breakdown by file extension. The extension histogram is the important part — it is what tells you which of the three classes below you are actually in, and it is how a reviewer checks your number without re-deriving it.

State the number in the tracking record and in the coordination room **before** editing anything. A retirement whose size is announced afterwards cannot be reviewed, only accepted.

## 2. Classify every hit into exactly one of three classes

The classes have different rules. Mixing them is the most common way a retirement goes wrong.

**Class A — live code and configuration.** Source, tests, wiring, build config, workflow files. These are **deleted** — and deleting a file means deleting every reference to its *name* as well. A workflow step, a package script entry, or an import path that names a file you removed does not mention the retired concept at all, so nothing in a concept search will ever surface it. The reference scan is part of the delete step, not a later cleanup. Before deleting a function or module that the feature owned, read what is *inside* it: retired features accumulate logic that never belonged to them — a default, a validation, a piece of guidance the rest of the product still needs. Extract that first and land it where it belongs, then delete the wrapper. Deleting the wrapper whole removes behaviour nobody decided to retire, and it does so silently, because no test names the retired feature. A test that exists only to exercise the retired feature is deleted with it; a test that covers a surface the feature merely touched is edited, not deleted, and the edit must keep the assertion meaningful rather than trivially true.

**Class B — shipped documentation and agent assets.** Guides, skills, prompts, runbooks, anything the product hands to a user or an agent. These are **updated**, never left stale. An agent asset that still instructs agents to use a retired capability is a live defect, not documentation debt: the asset ships, so it keeps producing the behaviour after the code is gone. Updated is the rule for an asset that merely *mentions* the capability. An asset that exists **entirely** for it — a skill, a runbook, a reference page about the retired feature and nothing else — is deleted, not rewritten: editing it leaves a document whose subject no longer exists, and the shipped asset set keeps handing it to agents. Deleting it is a Class A action on a Class B file, so it takes the reference scan with it.

**Class C — historical records.** Review requests, chat manifests, provenance records, changelogs, decision logs, migration journals. These are **not rewritten**. History records what was true when it was written. Retiring a feature does not make its past untrue, and editing these destroys the audit trail the rest of the release depends on. If a historical record must be annotated, append; do not revise.

**A record's class is measured, not inferred from how it looks.** A file can carry every mark of history — an old version id, a superseded name, a fingerprint from a finished migration — and still be a derived artifact that some assertion regenerates from the current world. Before agreeing to protect something as Class C, find the code that *writes or validates* it. If a gate reproduces it from today's inputs, it is Class A and it has to move with them; treating it as history leaves a stale record that fails a gate nobody can explain. This is the failure mode that survives review, because a guardrail phrased as "do not touch that" reads as caution and gets agreed to.

Publish the class counts alongside the totals. A reviewer's first question is which files you put in Class C, because that is where an over-eager removal does damage that cannot be undone.

## 3. Gate the irreversible parts separately

Schema changes, data migrations, and anything that drops storage are **not part of the same step as code removal**, and they do not share its approval.

- Enumerate every table, column, index, and migration file the feature owns.
- Decide, explicitly and in writing, whether data is dropped or retained-but-orphaned. "We will drop it" is a decision someone with authority makes; it is never a consequence of deleting code.
- A migration that drops storage lands **after** the code that reads it is gone and published, not with it. Otherwise a rollback of the code lands on a schema that can no longer serve it.
- Record the migration's risk classification in whatever policy the repository uses for migrations, and expect that to require its own review.

If the retirement's schema half cannot be approved in this release, the code half can still land — but say so explicitly, because a retirement that leaves its tables behind is a known, recorded state rather than an oversight.

## 4. Follow the republish cascade

In a workspace where packages depend on each other by workspace reference, removing a feature from one package changes the packed bytes of every package that depends on it, whether or not their own source changed.

- List the dependants of every package you touched.
- Each dependant whose packed manifest or content changes needs a version bump, and each bump needs its own declaration sites updated — versions in this kind of repository are typically written in several places that nothing derives from each other.
- Before publishing anything, verify that every internal pin in every candidate manifest either resolves on the registry or is published in the same run. A workspace reference resolves to the **local** version at pack time, so a retirement that bumps a package without publishing it can seal a version nobody can install.

## 5. Prove zero residue

The removal is not done when it compiles. It is done when the inventory from step 1, re-run with the same command from the same working directory, returns only Class C files.

```
# after the removal — the same pattern and exclusions as step 1
grep -rIlE '(^|[^A-Za-z])<concept>|[a-z]<Concept>' . \
  | grep -vE 'node_modules|(^|/)dist/|(^|/)build/|\.svelte-kit|pnpm-lock|(^|/)\.git/'
```

Then run the second probe, which measures something the first one structurally cannot. For **every file, script, and command name you deleted**, search the whole repository set for that name — basename, package-script key, import path, workflow step. It must return nothing.

```
# for each deleted name
grep -rInF '<deleted-basename-or-script-key>' . \
  | grep -vE 'node_modules|(^|/)dist/|(^|/)build/|\.svelte-kit|pnpm-lock|(^|/)\.git/'
```

Derive that list of names from the diff rather than from memory —
`git diff --diff-filter=D --name-only <base>..<head>` — and add the names of
anything you removed from inside a file that survived.

A concept search measures **content**; this one measures **references**. They are different kinds of residue and one never finds the other. The failure this prevents is real and recent: a retirement removed a tooling script cleanly, but the CI workflow still invoked it by name. The invocation contained no trace of the retired concept, so a concept-only closing check reported zero residue while CI failed on every push for two days — and because the chain stopped there, it also hid the next failure behind it.

**Run the closing probe at the final commit, with a clean working tree.** A retirement usually lands in several commits, and a count taken before the last one describes a state that never ships. An intermediate number cannot go into the closing record: it will be larger than the truth, it will name files that are already clean, and the next person will go looking for residue that is not there.

Every remaining hit is named in the closing record with its class and the reason it stays — including the ones that are **not** residue at all. A concept search finds the word, not the feature, so a fixture called "Mission Plan" or a variable named `permission` will surface; name those explicitly as legitimate false positives, or someone later will "clean up" a file that was never part of this. "Nothing remains" is not a claim you make; it is a diff between two runs of the same command, and both runs belong in the record.

Then run the repository's own gates — the full verification command, not a subset you chose. If the chain stops at the first failure, run the steps **after** the failing one individually before reporting what is blocking; a chain reports one failure, not the set of them.

## 6. Close it

- The tracking record carries: the before/after inventory numbers, the class breakdown, the schema decision and who made it, the list of republished packages, and the remaining Class C hits.
- The sentence from step 0 goes in the release notes. Users are entitled to know what stopped being possible.
- If agent assets changed, the shipped asset set is republished; an updated asset that never ships has not been updated.

## What makes this go wrong

The recurring pattern is not carelessness, it is **partial visibility**. Someone measures the code and not the assets, or the source and not the schema, or one repository and not its dependants. Each partial view produces a plan that looks complete and finishes early, and the missing part surfaces after the release, in someone else's install.

So the discipline is: measure everything first, classify before editing, keep the irreversible on its own gate, and close with the same command you opened with.

That last clause earns its place. In the retirement this version was written from, the closing inventory was run one commit early and reported four residual files; the reviewer re-ran the same command at the final commit and got two. Nothing was wrong with the removal — the record was simply about to close with a number that had never been true of the shipped state. The two-run diff is not ceremony: it is the only part of this flow that catches a closing claim, and it only works if someone other than the author runs the second leg.
