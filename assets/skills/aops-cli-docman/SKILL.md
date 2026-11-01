---
name: aops-cli-docman
version: 17
description: "aops doc, documents: the AOPS CLI Docman playbook for composable document authoring, CRUD-first edits, guarded import, safe delete, publish and mirror refresh, index/summary/search/answer retrieval gates, and numbered-outline authoring with explicit link numbering and positions."
metadata:
  supersedes: "v16"
  short-description: "AOPS CLI Docman thin discipline guide"
  tags:
    - cli
    - docman
    - documents
    - composable
    - retrieval
    - crud-first
    - safe-delete
    - publish
    - mirror
    - help-first
    - outline-numbering
---

# AOPS CLI Docman

Docman owns canonical written knowledge as a composable graph:
document -> document version -> section -> page -> page version -> links.

This is a thin agent guide. Deep mechanics live in the current `Docman User Guide`; exact flags live in `aops doc ... --help`. Help wins for flags, the guide wins for normative workflow, and the running server schema wins for raw payloads.

## Start Here

1. Read `aops doc --help`, then the smallest nested `--help` surface.
2. Refresh the repo mirror when freshness matters:
   `aops doc mirror pull --project-slug <slug> --apply --json`.
3. Find the smallest relevant guide section with local search or `view doc-page`.
4. Inspect the current document/version/outline before any write.
5. Prefer section/page CRUD for a targeted change; do not rebuild a whole document unnecessarily.
6. After a published content change, rebuild index and summary, test search and answer, then refresh the mirror.

The installed launcher may be `aops`; the package launcher is `aops-cli`.

## Use This Skill For

- creating and reading document groups, documents, and versions;
- composable section/page authoring and targeted page-version edits;
- building a numbered outline whose rendered numbers match the source;
- guarded Markdown import when the whole structure changes;
- publishing a current version and refreshing mirrors;
- index, summary, search, answer, source, and materialization reads;
- guarded deletion of a disposable document.

Use `aops-cli-core` for repo context and guard flags, `aops-cli-projectman` for execution/review truth, `aops-cli-discuss` for design decisions, and `aops-cli-agentspace` for durable memory and reusable assets.

## Canonical Sources

1. The installed `Docman User Guide` for deep mechanics: `docman-user-guide.md`
   in the folder `aops assets status --json` reports as `userGuidesRoot`.
2. Live help: `aops doc <group> <command> --help`.
3. Live discovery: `aops agent tools --domain docman --summary --json`.
4. Raw schema: `aops agent schema --tool docman.<operation> --summary --json`.

`.aops-cache/docman/**` is a read-only cache. Never hand-edit it as canonical truth.

## Command Map

| Need | Primary surface |
|---|---|
| Find ids and structure | `doc list`, `doc version list`, `doc outline get`, `doc source` |
| Edit one page body | `doc version create --init-mode clone_all`, `doc page draft-save`, `doc page-version update` |
| Edit metadata | `doc section update`, `doc page update` |
| Add or reorder graph nodes | `doc section create`, `doc page create`, `doc link`, `doc order` |
| Numbered outline | `doc section create --numbering --position`, `doc link section|page --numbering --position`, verify with `doc publish --target markdown` |
| Import a complete Markdown structure | `doc import --baseline --guard-target --dry-run` |
| Publish current truth | `doc set-current-version --publish-now` |
| Retrieval gates | `doc index build`, `doc summary build`, `doc search`, `doc answer` |
| Refresh local cache | `doc mirror pull` |
| Delete a disposable document | `doc delete --id --confirm-name --preview`, then reviewed `--apply --confirm` |
| Raw fallback | `agent schema`, then `agent invoke` only when sugar is insufficient |

## Composable Authoring Rule

All durable guides, ADR logs, specs, and comparable knowledge documents should be structured as meaningful sections and pages. Avoid one giant page when independent sections can preserve meaning and be retrieved separately.

Every new published version should support this read path:

```bash
aops doc index build --document-version-id <docver-id> --json
aops doc summary build --document-version-id <docver-id> --json
aops doc search --document-version-id <docver-id> --q "<known fact>" --remote --json
aops doc answer --document-version-id <docver-id> --q "<question>" --remote --json
aops doc mirror pull --project-slug <slug> --document-slug <slug> --apply --json
```

Review citations as well as the generated answer. Retrieval is deterministic assistance, not authority to ignore the cited source.

## Numbered Outline Rule

The Markdown render (`doc publish --target markdown`, `doc mirror pull`) drops a leading number from a section or page title ("1. Overview" renders as "Overview") and prints its own number from the sibling order. Any node that changes the sibling order (a colophon section inserted at the top, "Overview" pages synthesized from prose that precedes the first H3) shifts every rendered number, while the body text keeps its own references ("see section 7.6"). A 19-section guide with 36 in-text section references rendered every number off by one this way.

When the source has numbered headings or in-text section references:

1. Plan the numbering before linking: section `N`; page `N.k`; direct prose under `## N` (an overview page, or a section without subheadings) `N.0`; a front page or colophon `0` and `0.0`.
2. Give every section link and page link that number as `--numbering` and a unique `--position` (`doc section create`, `doc page create`, `doc link section|page`). To set them on existing links use the raw tool with `--project-name`, not `--project-slug`:
   `aops agent invoke --tool docman.document-section-link.update --input @link.json --apply --project-name <project>` where `link.json` is `{"id": "<link-id>", "patch": {"position": <k>, "numbering": "<N.k>"}}`. All 102 outline links of a 20-section, 82-page document were set this way; `docman.section-page-link.update` exists too, so read the link type from `doc outline get` before choosing the tool.
   The override renders as given: section `6` renders `## 6`, page `6.1` renders `### 6.1`, page `6.0` renders `### 6.0`. A page value that does not start with the section number is prefixed (`-` renders as `1.-`); an empty string means automatic numbering. There is no option to print a heading without a number. Each link update takes 3-20 seconds (about 25 minutes for 102 links); plan the time.
3. Never leave two sibling links on the same position. Updating only `numbering` on such links swapped their rendered order.
4. After `doc section create`, take the new link id from `result.link.data.link.id` and confirm it in `doc outline get`. `artifacts.documentSectionLinkId` can return the version's first link; pages created against it attach under the wrong section without any error.
5. To re-parent a link, use the same raw `docman.document-section-link.update` and re-read the outline; `doc order sections` did not apply `parentLinkId`.
6. Keep H1 and H2 out of page bodies; the page title is the heading. Turn a source title block into bold text on the front page.
7. Fix a page body on the `clone_all` version with `doc page draft-save --page-version-id <pv> --document-link-id <link> --content @file`: it forks a new page version and moves only the new version's link; the previous version's page version is unchanged. Then `doc page-version update --status published`.
8. Verify before `set-current-version`, every count against the plan: outline read-back numbering and position (n/n), `doc publish --target markdown` headings one-to-one in plan order (n/n), page bodies byte-equal to the source (n/n), every in-text section reference resolving to an existing rendered heading (n/n). Then `doc set-current-version --expected-previous-version-id --publish-now`, `doc index build`, `doc summary build`, `doc search --remote` and `doc answer --remote` (hit numbers follow the override), `doc mirror pull`, and the same heading, body and reference check on the mirror file.

## Safe Targeted Page Edit

```bash
aops doc version create --document-id <doc-id> --version <next-n> \
  --status draft --init-mode clone_all --source-version-id <current-docver-id> \
  --apply --json

aops doc outline get --document-version-id <new-docver-id> \
  --titles-only --depth 3 --json

aops doc page draft-save --page-version-id <pagever-id> \
  --document-link-id <section-page-link-id> --content '@./target-page.md' \
  --apply --json

aops doc page-version update --id <new-pagever-id> --status published --apply --json
aops doc set-current-version --document-id <doc-id> --version-id <new-docver-id> \
  --expected-previous-version-id <old-docver-id> --publish-now --apply --json
```

Use a real file or JSON encoder for multiline content. Do not pass literal `\\n` text and assume it became a newline.

## Full Import Guard

Use full import only when the complete document structure truly changes:

```bash
aops doc import --from-markdown --document-version-id <docver-id> \
  --source ./candidate.md \
  --baseline ./.aops-cache/docman/<group>/<document>.md \
  --guard-target "<intended heading path or title>" \
  --dry-run --json
```

Apply only after reviewing the baseline guard and unrelated deltas. If the plan reports `section-direct-body-ignored`, stop; use the documented overview-page synthesis only when direct section prose must become leaf pages.

## Safe Disposable Delete

Read back the exact document id and title. Preview first:

```bash
aops doc delete --project-slug <slug> --id <document-id> \
  --confirm-name '<exact current title>' --preview --json
```

Apply only to the verified disposable record:

```bash
aops doc delete --project-slug <slug> --id <document-id> \
  --confirm-name '<exact current title>' --apply --confirm --json
```

An exact-title mismatch is a `confirmName` validation error and must not delete the record. After a successful delete, a hosted get should return not found. A repo mirror may remain stale until the next mirror pull; stale mirror bytes do not mean the canonical record still exists.

## Reading Ladder

```bash
# broad local discovery
aops doc scope search --project-slug <slug> --q "<topic>" --local --json

# known-version retrieval
aops doc search --document-version-id <docver-id> --q "<topic>" --local --json

# structure and ids
aops doc outline get --document-version-id <docver-id> --titles-only --depth 2 --json

# exact mirrored section body
aops view doc-page <document-slug>#<exact-heading-or-slug> --max-bytes 6000
```

Use `doc search --remote` or `doc answer --remote` after canonical changes and before mirror refresh. Use `--local` only when the mirror is known to be current or hosted access is unavailable.

## Schema Fallback

Before raw writes:

```bash
aops agent schema --tool docman.<operation> --summary --json
```

Sugar option names may differ from raw schema fields. Sugar help wins for `aops doc`; schema wins for `agent invoke`. Do not guess payload fields or retry with invented flags.

## Anti-Patterns

1. Hand-editing `.aops-cache/docman/**` as canonical truth.
2. Full import for a one-page edit.
3. Publishing without current-version and expected-previous guards.
4. Skipping index/summary/search/answer after a guide or ADR change.
5. Reviewing a stale mirror after a hosted write.
6. Treating `--confirm` as routine instead of a reviewed destructive gate.
7. Deleting by broad selector, guessed title, or an unverified id.
8. Copying the entire user guide into this skill.
9. Relying on automatic render numbering for a source with numbered headings or in-text section references.
10. Trusting `artifacts.documentSectionLinkId` after `doc section create` instead of the outline read-back.
11. Embedding the source file's H1/H2 title block in a page body.

## Done When

- the intended current version is published;
- for a numbered source, the rendered Markdown headings match the plan one-to-one, page bodies match the source, and in-text section references resolve to rendered headings;
- index and summary builds succeed;
- a known-fact search and citation-first answer reach the new section;
- the targeted mirror reports the current version id;
- any public asset is regenerated from canonical output;
- temporary records and files are removed without touching unrelated state.
