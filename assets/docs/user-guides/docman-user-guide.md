---
schemaVersion: 1
entityType: "docman.document-mirror"
readOnly: true
source: "docman"
projectId: "b4e7db65-7956-4c92-8cc1-0f16ef908d41"
projectName: "aops"
projectSlug: "aops"
scopeId: "b4e7db65-7956-4c92-8cc1-0f16ef908d41"
groupId: "1013e09a-3e44-4465-a659-d15f1a2c319d"
documentId: "a348eb55-03cb-47d2-88bf-5a32deaae61f"
documentVersionId: "bdf0c67b-25d8-4096-879e-a8d6e6040d9b"
documentVersion: 4
documentSlug: "docman-user-guide"
title: "Docman User Guide"
target: "markdown"
pulledAt: "2026-08-23T17:44:40.074Z"
---
<!-- READ-ONLY MIRROR: update Docman/source docs, then run aops-cli doc mirror pull. -->

# Docman User Guide

_Release Notes:_ Adds the tested safe-delete recipe, stale-mirror semantics, validation-error contract, and generated CLI catalog entry.

## 1 Agent fast path

### 1.1 Overview

When an agent receives a Docman task, use this sequence before reading the
entire guide:

1. Read `aops doc <group> <command> --help` for the command's actual flags.
2. If the repository has a mirror, find the smallest relevant fragment first
   with `aops view docs`, `aops view doc-page`, or `aops doc search --local`.
3. When a hosted capability is required, do not guess the tool name or input:
   run `aops agent tools --domain docman --summary --json`, then
   `aops agent schema --tool <tool-id> --summary --json`.
4. Write canonical state to the server. Never hand-edit `.aops/docman/**`.
5. For a single-page **content** change, fork the current version with
   `clone_all`, fork the page version with `draft-save`, and atomically switch
   the current/published pointer with `set-current-version` instead of using a
   full Markdown import. If section/page metadata or the outline changes, do
   not assume cloning isolates shared entities; use a clean version with a
   guarded import or an explicit new-entity flow.
6. After writing, run the `index build`, `summary build`, `search`, `answer`,
   and `mirror pull` gates.

The package launcher is named `aops-cli`; the installed public launcher may be
named `aops`. Both names refer to the same CLI surface in this guide.

## 2 Ownership model

### 2.1 Overview

`docman` owns:

1. document creation
2. document metadata updates
3. the version, section, page, page-version, and link graph
4. saved-version retrieval
5. deterministic answer/search/source/publish operations

`docman` does not own:

1. durable memory or project summaries
2. planning, sprint, or task execution truth
3. prompt, resource, or skill ownership

In short:

1. `projectman` = execution truth
2. `agentspace` = context truth
3. `docman` = canonical written knowledge truth

### 2.2 AOPS CLI skill routing

#### 2.2.1 Overview

The hosted operator playbook for Docman is the `aops-cli-docman` skill.

| Need | Canonical source |
|---------|------------------|
| exact command flags | `aops-cli doc --help` and nested help surfaces |
| document graph semantics | this file and `architecture.md` |
| operator guard/playbook | hosted `aops-cli-docman` |
| published/searchable projection | `slug:aops` hosted Docman current version |
| public distribution copy | `aops/assets/docs/user-guides/docman-user-guide.md` |

Development authoring truth is the `slug:aops` Docman record on AOPS's
canonical server. The repository-local `USER_GUIDE.md` is a source-parity and
domain-development reference; `.aops/docman/**` is a read-only mirror/cache.
When preparing a public release, materialize the published/current Docman
version into the physical asset file. Public assets are consumed as files on
user machines; do not automatically import every global public asset into each
user's canonical Docman server.

## 3 Core model

### 3.1 Overview

The Docman graph is not document-only. Its primary chain is:

1. `document`
2. `document-version`
3. `section`
4. `page`
5. `page-version`
6. `document-section-link`
7. `section-page-link`

Rules:

1. A `section` is a container, not the body owner.
2. Authored source lives on `page-version`.
3. `document-section-link` owns the document tree.
4. `section-page-link` owns reusable flat section-page membership.
5. Root pages are supported.
6. `page-version.format` accepts only `md`, `mdx`, or `text`.
7. `document-version.contentMode` determines the document shape:
   - `structured`: section/page graph; `fileExtension=md`
   - `text-file`: one root page; sections and a second page are forbidden
8. `fileExtension` is a persistent file identity separate from source format.
   Write it in lowercase without a leading dot (`cs`, `txt`). Uppercase input
   is not normalized and is rejected fail-closed.

## 4 Using Docman through the AOPS CLI

### 4.1 Overview

`aops-cli doc` accesses Docman through the hosted AOPS plane.

There are two primary surfaces:

1. authoring / structure:
   - `doc group create|update|delete`
   - `doc group list|get`
   - `doc create`
   - `doc list|get|update`
   - `doc update`
   - `doc version create`
   - `doc version list|get|update`
   - `doc set-current-version`
   - `doc section create`
   - `doc section update|copy|unlink`
   - `doc section list|get`
   - `doc page create`
   - `doc page update|copy|move|unlink|membership-unlink`
   - `doc page list|get`
   - `doc page draft-save`
   - `doc page-version list|get|update`
   - `doc source-file save`
   - `doc link section`
   - `doc link page`
   - `doc order sections`
   - `doc order pages`
   - `doc outline get`
   - `doc import --from-markdown`
2. retrieval / publish:
   - `doc index build`
   - `doc summary build`
   - `doc search`
   - `doc scope search`
   - `doc scope answer`
   - `doc answer`
   - `doc source`
   - `doc publish`
   - `doc mirror pull`
   - `doc mirror push`

## 5 Authoring cookbook

### 5.1 Overview

Canonical authoring-read loop:

1. If needed, establish the classification tree with `doc group create|update|delete`.
2. `doc create`
3. `doc version create`
4. `doc section create`
5. `doc page create`
6. `doc outline get`
7. `doc set-current-version`
8. `doc index build` and `doc summary build`
9. `doc answer|search|source|publish`
10. `doc mirror pull`

### 5.2 Create a canonical Docman version from existing Markdown

#### 5.2.1 Overview

If a Markdown file is only a temporary source or draft and the Docman server
will become canonical truth, `doc mirror push` is not the default path. When
the section/page graph must be preserved or designed manually, create the
document as Docman entities:

1. Find or create the group.
2. Set document metadata with `doc create` or `doc update`.
3. Open a new `document-version`.
4. Build the graph from the H2/H3 structure with `section create` and
   `page create`.
5. Verify section and page counts with `outline get`.
6. Run `index build` and `summary build`.
7. Verify retrieval with `search` and `answer`.
8. Run `doc mirror pull` for repository-local reads.
9. Delete temporary repository Markdown only after verifying the mirror.

```bash
aops-cli doc group list --json

aops-cli doc update \
  --id <document-id> \
  --title "UI System v2" \
  --slug ui-systemv2 \
  --group-id <group-id> \
  --status published \
  --visibility internal \
  --project-name eops \
  --apply \
  --json

aops-cli doc version create \
  --document-id <document-id> \
  --version <next-version> \
  --status published \
  --init-mode clean \
  --project-name eops \
  --apply \
  --json

aops-cli doc section create \
  --document-version-id <docver-id> \
  --title "Navigation" \
  --position 0 \
  --apply \
  --json

aops-cli doc section create \
  --document-version-id <docver-id> \
  --parent-link-id <parent-document-section-link-id> \
  --title "Top Thin Bar" \
  --position 0 \
  --apply \
  --json

aops-cli doc page create \
  --document-version-id <docver-id> \
  --section-id <section-id> \
  --title "Top Thin Bar" \
  --format md \
  --content '@./top-thin-bar.md' \
  --apply \
  --json

aops-cli doc outline get --document-version-id <docver-id> --json
aops-cli doc index build --document-version-id <docver-id> --project-name eops --json
aops-cli doc summary build --document-version-id <docver-id> --project-name eops --json
aops-cli doc search --document-version-id <docver-id> --q "top thin bar" --ensure summary --json
aops-cli doc answer --document-version-id <docver-id> --q "How should panels work?" --ensure summary --json
aops-cli doc mirror pull --group-uid <group-uid> --out-dir .aops/docman --target markdown --project-name eops --apply --json
```

When one Markdown file needs a heading-graph import, use the dedicated import
surface:

```bash
aops-cli doc import \
  --from-markdown \
  --document-version-id <clean-docver-id> \
  --source ./ui-system-v2.md \
  --baseline ./.aops/docman/architecture/ui-system-v2.md \
  --guard-target "Target Section Or Page" \
  --dry-run \
  --json

aops-cli doc import \
  --from-markdown \
  --document-version-id <clean-docver-id> \
  --source ./ui-system-v2.md \
  --baseline ./.aops/docman/architecture/ui-system-v2.md \
  --guard-target "Target Section Or Page" \
  --synthesize-overview-pages \
  --dry-run \
  --json
```

Heading import MVP rules:

1. `doc import --from-markdown` is the single-file authoring/migration surface;
   multi-file batch import is deferred to a later sprint.
2. H1 is excluded from the graph; H2/H3 become `section`, and H4+ become
   `page`.
3. Direct body content under H2/H3 is not converted into a page by default;
   the import returns a `section-direct-body-ignored` warning. To preserve that
   body, repeat the dry run with `--synthesize-overview-pages` and the same
   `--baseline` / `--guard-target` guards. That flag imports the body as an
   `Overview` page under the corresponding section. In CLI JSON, warning codes
   appear under `result.summary.warnings[].code` without a baseline, or under
   `result.import.summary.warnings[].code` with a baseline guard.
4. The default `existingGraphPolicy=error` requires a clean version. Append or
   replace intent requires explicit `--append-existing-graph` or
   `--replace-existing-graph` flags.
5. This behavior does not extend `doc mirror push`; it retains flat root
   Markdown migration/snapshot semantics.

Full-import guardrail:

1. For a one-section or one-page change in an existing document, full import
   should be the last choice. Use `clone_all`, targeted page/section CRUD,
   `doc set-current-version`, and then `doc mirror pull` first.
2. When a full Markdown import is required, pass the previous materialized
   Markdown as `--baseline <path>` and inspect `result.baselineGuard` in the
   `--dry-run --json` output.
3. Use `--guard-target <heading>` only for the heading path/title expected to
   change. If the baseline guard finds unrelated missing, added, changed, or
   truncated body deltas, the `--apply` flow is blocked without `--confirm`.
4. `--confirm` is not a way to silence the guard. It is a deliberate override
   after the operator reviews the full-import delta.

Notes:

1. `.aops/docman/**` is a read-only mirror, not canonical source.
2. `doc mirror pull` produces a materialized copy from hosted Docman and writes
   the index.
3. If the repository config has no slug, use `--project-name` instead of
   `--project-slug`.
4. `doc mirror push` is only a migration/snapshot utility for importing root
   Markdown files; it does not replace manual section/page version authoring.
5. When building nested sections, `--parent-link-id` is the parent
   `document-section-link` id, not the parent section id. Take the link id from
   the create/link result, attach the child section under it, and verify the
   hierarchy with `outline get`.
6. H2/H3 rendering in a Markdown preview is not the canonical graph. Compose,
   preview, edit, and CLI outline must align with the same saved
   section/page/link graph. Automatic nested-graph generation from headings is
   a separate domain-chain import/migration operation whose dedicated surface
   is `doc import --from-markdown`.

### 5.3 Editing an existing document: prefer section/page CRUD

#### 5.3.1 Overview

When updating an existing Docman document, do not first regenerate the whole
document as temporary Markdown and import a new version with `mirror push`.
The canonical graph consists of the server-side `section`, `page`,
`page-version`, `document-section-link`, and `section-page-link` records. When
the target record is known, use direct CRUD sugar.

Find the current identifiers with small reads first:

```bash
aops doc list --project-slug <slug> --slug <document-slug> --json
aops doc version list --project-slug <slug> --document-id <document-id> --json
aops doc outline get --project-slug <slug> --document-version-id <current-version-id> --titles-only --depth 3 --json
```

Safe single-page **content** revision in a published/current document:

```bash
aops doc version create \
  --project-slug <slug> \
  --document-id <document-id> \
  --version <next-version> \
  --status draft \
  --init-mode clone_all \
  --source-version-id <current-version-id> \
  --apply --json

aops doc page draft-save \
  --project-slug <slug> \
  --page-version-id <cloned-page-version-id> \
  --document-link-id <cloned-document-link-id> \
  --content @./updated-page.md \
  --apply --json

aops doc set-current-version \
  --project-slug <slug> \
  --document-id <document-id> \
  --version-id <new-version-id> \
  --expected-previous-version-id <current-version-id> \
  --publish-now \
  --apply --json
```

`set-current-version` normally publishes the target version while making it
current and writes `publishedAt` using server time. Use `--no-publish` to switch
only the current pointer without changing status or `publishedAt`. Use
`--no-publish-now --published-at <iso>` for a caller-provided `publishedAt`.
Keep the `--expected-previous-version-id` guard to bind the former current id
during a race.

When needed, `draft-save` forks a locked page version or one shared by other
links. Pass `--document-link-id` so the forked version becomes visible in this
document tree. Read back `mode=fork`, the new `pageVersionId`, and the
`relinkedDocumentLinkId` from the result.

`clone_all` creates only new `document-section-link` rows; it does not
automatically clone `section`, `page`, `page-version`, or `section-page-link`
records. Therefore, running `section update`, `page update`, `page move`,
`page membership-unlink`, or `order pages` after cloning may retroactively
affect the previous published version. For a content-only revision, use the
`draft-save` fork flow above. For title, section-structure, or membership
changes, use a clean version with guarded full import or an explicit
new/cloned-entity flow, and verify that the previous version's compose hash is
unchanged.

```bash
aops-cli doc section update --id <section-id> --title "New section title" --apply --json
aops-cli doc section copy --source-section-id <section-id> --target-document-version-id <docver-id> --reuse-pages --apply --json
aops-cli doc section unlink --link-id <document-section-link-id> --apply --json

aops-cli doc page update --id <page-id> --title "New page title" --apply --json
aops-cli doc page draft-save --page-version-id <pagever-id> --content @./page.md --apply --json
aops-cli doc page copy --source-page-id <page-id> --target-section-id <section-id> --reuse-page --apply --json
aops-cli doc page move --link-id <section-page-link-id> --target-section-id <section-id> --position 2 --apply --json
aops-cli doc page unlink --link-id <document-section-link-id> --apply --json
aops-cli doc page membership-unlink --link-id <section-page-link-id> --apply --json

aops-cli doc page-version update --id <pagever-id> --status published --apply --json
```

Rules:

1. `section update` changes metadata; a section does not own body content.
2. `page update` changes page metadata. Use `doc page draft-save` for content
   or format changes.
3. `page-version update` performs only `draft|published|archived` status
   transitions; it does not accept content.
4. `section copy` reuses page-version links by default. Pass `--clone-pages`
   explicitly when a new page/page-version copy is required.
5. `page copy` links the selected page version, or the page version with the
   highest numeric version, to the target section by default. Pass
   `--clone-page` explicitly when a new page/page-version is required.
6. `page unlink` deletes the `kind=page` document-outline link produced by
   `page create`. Use the separate `page membership-unlink` command for a
   reusable `section-page-link`; do not substitute one link kind for the other.
   On a published version, use these raw CRUD commands only after proving that
   no other version shares the target entity/link.
7. Verify post-edit saved-version visibility with
   `outline get --titles-only --depth <n>`, `index build`, `summary build`,
   `search`, and `answer`.
8. Do not use only a mutable draft `pageVersionId` in a review request. Add the
   `contentHash` (`sha256:<hex>`) from a `document.compose.fetch` or
   `document.publish.materialize` readback to the review reference together
   with `documentVersionId`/`pageVersionId`. The review result must reference
   the same hash. If the draft is saved again, the hash changes; the prior
   acceptance no longer covers the new content, so a new review or re-review is
   required.

### 5.4 Create a group

#### 5.4.1 Overview

```bash
aops-cli doc group create \
  --title "Guides" \
  --apply \
  --json
```

### 5.5 Inspect a group

#### 5.5.1 Overview

```bash
aops-cli doc group list --json
aops-cli doc group get --id <group-id> --json
```

### 5.6 Create a document

#### 5.6.1 Overview

```bash
aops-cli doc create \
  --title "Enterprise Resource Management Datasheet" \
  --summary "Platform datasheet" \
  --apply \
  --json
```

### 5.7 Open a version

#### 5.7.1 Overview

```bash
aops-cli doc version create \
  --document-id <doc-id> \
  --version 1 \
  --status draft \
  --apply \
  --json
```

Note:

1. The Docman document-version surface requires an explicit version in this
   flow.
2. The auto-next-version ergonomics available in prompt and skill sugar are
   not exposed in Docman.

### 5.8 Create a section and link it to the outline

#### 5.8.1 Overview

```bash
aops-cli doc section create \
  --document-version-id <docver-id> \
  --title "Overview" \
  --apply \
  --json
```

### 5.9 Create a page and save its first draft

#### 5.9.1 Overview

```bash
aops-cli doc page create \
  --document-version-id <docver-id> \
  --section-id <section-id> \
  --title "Startup Behavior" \
  --format md \
  --content @./startup.md \
  --apply \
  --json
```

### 5.10 Link an existing page version under a section

#### 5.10.1 Overview

```bash
aops-cli doc link page \
  --section-id <section-id> \
  --page-version-id <pagever-id> \
  --apply \
  --json
```

### 5.11 Update ordering

#### 5.11.1 Overview

```bash
aops-cli doc order sections \
  --document-version-id <docver-id> \
  --update '{"linkId":"dsl-1","position":0}' \
  --update '{"linkId":"dsl-2","position":1}' \
  --apply \
  --json

aops-cli doc order pages \
  --section-id <section-id> \
  --update '{"linkId":"spl-1","position":0}' \
  --apply \
  --json
```

### 5.12 Inspect structure

#### 5.12.1 Overview

```bash
aops-cli doc version list --document-id <doc-id> --json
aops-cli doc version get --id <docver-id> --json

aops-cli doc section list --json
aops-cli doc section get --id <section-id> --json

aops-cli doc page list --json
aops-cli doc page get --id <page-id> --json

aops-cli doc page-version list --page-id <page-id> --json
aops-cli doc page-version get --id <pagever-id> --json

aops-cli doc outline get --document-version-id <docver-id> --json
```

### 5.13 Authoring with `--input @file.json`

#### 5.13.1 Overview

Section, page, version, and link commands accept shared JSON input. An explicit
CLI flag overrides the file value for the same field.

```bash
aops-cli doc section create \
  --input @./section.json \
  --title "Overview" \
  --apply \
  --json
```

### 5.14 Safely delete a disposable document

#### 5.14.1 Overview

Use the top-level safe-delete sugar only after reading back the exact document
id and its current title. The CLI forwards `--confirm-name` to
`document.delete.safe`; the domain rejects a title mismatch and protects the
document graph from an unsafe partial delete.

```bash
# Preflight does not delete anything.
aops-cli doc delete \
  --id <doc-id> \
  --confirm-name "Exact current document title" \
  --preview \
  --json

# Destructive execution requires both guards.
aops-cli doc delete \
  --id <doc-id> \
  --confirm-name "Exact current document title" \
  --apply \
  --confirm \
  --json
```

For a disposable smoke test, first prove that a deliberately mismatched title
is rejected and that the document still exists. After the exact-title delete,
verify a definitive hosted not-found with `doc get --id <doc-id>`; do not treat
a zero exit code or absence from one list page as sufficient evidence.

The hosted operation classifies an exact-title mismatch as a `confirmName`
validation error (HTTP 400). It must never be reported as a server failure, and
the document must remain readable after the rejected request.

Canonical deletion does not remove bytes that were already written to a local
`.aops/docman` mirror or another `--out-dir`. Those files are stale read-only
cache data until explicitly reconciled. A targeted mirror pull for the deleted
slug fails because no hosted document matches, and it intentionally does not
erase the old file. Remove or replace only the exact known local mirror path as
a separate filesystem action after confirming the hosted not-found.

## 6 Mirror policy

### 6.1 Overview

Do not confuse Docman's three mirror and import concepts:

1. `doc mirror pull`: server truth -> `.aops/docman/**` read-only mirror.
2. `doc mirror push`: root `*.md` import utility; not a general update or
   version-authoring path.
3. `doc import --from-markdown`: the dedicated authoring/migration surface that
   creates an H2/H3 section and H4+ page graph from one Markdown file.

Text-file documents use a different contract:

1. The final domain operation id is `document.source-file.save`; the AOPS sugar
   surface exposes it as `doc source-file save`. The former draft name
   `document-version.source-file.put` is not used.
2. Save writes the exact UTF-8 source as an immutable page-version snapshot.
   `--expected-version-id` and `--expected-content-hash` are stale-writer
   guards; `--publish-now` completes the current/published lifecycle in the
   same guarded flow.
3. A `contentMode=text-file` document contains exactly one root page. Sections,
   a nested graph, or a second page cannot be created.
4. `fileExtension` must be lowercase and path-safe. `.cs`, `CS`, path
   separators, `..`, whitespace/NUL, or an unknown source format are not
   corrected automatically.
5. For a text file, `doc mirror pull` writes the raw `<slug>.<ext>` file and an
   adjacent `<slug>.<ext>.docman.json` v3 sidecar. It injects no frontmatter or
   comments into the raw file.
6. `doc mirror push` writes a new snapshot only when the v3 sidecar identity,
   scope/project/version/hash, extension, and exact UTF-8/NUL guards pass.

Mirror v2 and v3 coexist:

1. Legacy/structured Markdown mirrors retain v2 `.md` behavior.
2. Text-file mirrors use v3 raw-file-plus-sidecar behavior.
3. When a legacy structured record has no profile, `md` is accepted for v2
   compatibility. However, if its persisted structured `fileExtension` is
   explicitly not `md`, it is rejected fail-closed. The structured mirror path
   migration contract is not defined yet; this guard prevents silently
   discarding the persisted extension and writing to the wrong file path.

For canonical server-truth documents, use the
`doc update/version/section/page` flow when the operator asks to create a new
Docman version or establish a section/page structure. Use mirror push only
when Markdown files should be imported as-is and their section/page graph can
be derived from the simple import format.

Mirror pull:

```bash
aops-cli doc mirror pull \
  --group-uid eops-ui \
  --out-dir .aops/docman \
  --target markdown \
  --project-name eops \
  --apply \
  --json
```

Checks after mirror pull:

1. The `.aops/docman/index.md` read-only index must exist.
2. `documentVersionId` and `documentVersion` in document frontmatter must
   identify the expected version.
3. An agent may read the mirror for exact wording. For retrieval questions,
   use the server-backed or `--local` `doc search|answer` surface.

## 7 Retrieval and publishing cookbook

### 7.1 Overview

Canonical read policy:

1. Use `doc answer` for broad questions.
2. Use `doc search` for hit discovery.
3. Use `doc scope search` to discover hits across multiple documents in a
   project/scope.
4. Use `doc source` for exact composed source.
5. Use `doc publish` for final text output.

### 7.2 Keep retrieval builds explicit

#### 7.2.1 Overview

`search`, `scope search`, and `answer` are local-first by default: when the
repository mirror returns a hit, the CLI does not call the hosted server and
does not run `--ensure`. Use `--local` for an explicitly local/cache read and
`--remote` for the canonical hosted retrieval and build gate. On the hosted
path, the default `--ensure summary` refreshes index and summary rows before
reading. Use `--ensure none` to read without changing persisted rows, or
`--ensure index` to ensure only the index. Select an alternative mirror root
with `--mirror-dir <path>`.

Explicit build:

```bash
aops-cli doc index build --document-version-id <docver-id> --json
aops-cli doc summary build --document-version-id <docver-id> --json
```

Sugar sequence:

```bash
aops-cli doc answer \
  --document-version-id <docver-id> \
  --q "Enable pin behavior?" \
  --ensure summary \
  --json
```

`--ensure summary` is not hidden magic. The CLI first builds the summary, then
calls `answer-pack`. Production document publication must still pass the
explicit `index build` + `summary build` + retrieval-readback gate.

### 7.3 Search

#### 7.3.1 Overview

```bash
aops-cli doc search \
  --document-version-id <docver-id> \
  --q "startup current" \
  --retrieval-strategy hybrid \
  --remote \
  --ensure summary \
  --json
```

### 7.4 Scope-wide smart search

#### 7.4.1 Overview

```bash
aops-cli doc scope search \
  --project-id <project-id> \
  --q "startup current" \
  --retrieval-strategy hybrid \
  --json
```

Note:

1. This surface operates on the latest/current version of each document.
2. As v1 behavior, it automatically builds a missing index.
3. Hits include the document, section/page breadcrumb, page range, and excerpt.

### 7.5 Citation-first answer

#### 7.5.1 Overview

```bash
aops-cli doc answer \
  --document-version-id <docver-id> \
  --q "What does the startup sequence require?" \
  --limit 3 \
  --retrieval-strategy hybrid \
  --remote \
  --ensure summary \
  --json
```

### 7.6 Exact composed source

#### 7.6.1 Overview

```bash
aops-cli doc source \
  --document-version-id <docver-id> \
  --section-id <section-id> \
  --json
```

### 7.7 Materialize

#### 7.7.1 Overview

```bash
aops-cli doc publish \
  --document-version-id <docver-id> \
  --target markdown \
  --json

aops-cli doc publish \
  --document-version-id <docver-id> \
  --target html \
  --out ./tmp/doc.html

aops-cli doc publish \
  --document-version-id <text-docver-id> \
  --target source \
  --out ./tmp/device-control.cs
```

Supported targets:

1. `markdown`
2. `html`
3. `source` (only for `contentMode=text-file`; exact saved UTF-8 source)

Note:

1. When `--out <path>` is provided, materialized content is written directly
   to the file.
2. When used with `--json`, the response envelope also includes
   `artifacts.outputPath`.
3. A `source` response carries `contentMode`, `sourceFormat`, `fileExtension`,
   `suggestedFileName`, `mediaType`, and exact-source `contentHash` metadata;
   the extension is not inferred from content.
4. Cockpit does not use the Markdown renderer for text files. It shows escaped
   monospace source with wrap/copy/download controls and does not request an
   outline.

## 8 Relationship with Memory and PM

### 8.1 Overview

Docman does not replace memory or planning.

Canonical reading order:

1. `agentspace` sticky guidance / project summary / resume memory
2. `docman`, when needed
3. `projectman`, when planning truth is needed

In short:

1. Memory tells the agent what to read.
2. Docman provides canonical content.
3. Projectman provides execution state.

## 9 Choosing a surface

### 9.1 For document-only work

#### 9.1.1 Overview

Use `aops-cli doc ...`.

### 9.2 When an agent is trying to understand a topic

#### 9.2.1 Overview

1. Start with `memory resume`.
2. Then run `summary get`.
3. When relevant, use `doc answer` or `doc search`.
4. Use `doc scope search` first when the agent needs to locate information
   across multiple project documents.

### 9.3 When exact wording is required

#### 9.3.1 Overview

`doc source`

### 9.4 When a final render is required

#### 9.4.1 Overview

`doc publish`

To export to a file:

`doc publish --out <path>`

## 10 Notes

### 10.1 Overview

1. Docman retrieval is saved-version-only; unsaved editor state is not
   canonical source.
2. `source` and `publish` use flat payloads; there is no nested `input`
   envelope.
3. `docman` is not specific to AOPS. It also operates independently for
   datasheets, runbooks, manuals, architecture documents, and technical
   references.

## 11 Appendices

### 11.1 Overview

Do not edit the tables in this section by hand. Regenerate them from the AOPS
CLI Commander registrations with `aops docs user-guide --guide docman`.

<!-- aops-generated:docman-command-catalog:start -->

### 11.2 Generated Docman command catalog

#### 11.2.1 Overview

> This appendix is generated from the public `aops doc` Commander registrations. Do not edit it by hand; regenerate it with `aops docs user-guide --guide docman`.

| Command | Purpose |
| --- | --- |
| `aops doc` | Docman authoring, retrieval, and publish sugar over the hosted AOPS plane |
| `aops doc answer` | Read a citation-first deterministic answer pack for a saved document version |
| `aops doc create` | Create a Docman document through the canonical flow surface |
| `aops doc delete` | Safely delete one Docman document graph after exact title confirmation |
| `aops doc get` | Get a Docman document by id through the hosted gateway |
| `aops doc group` | Docman document-group commands |
| `aops doc group create` | Create a document group |
| `aops doc group delete` | Delete a document group |
| `aops doc group get` | Get a document group by id |
| `aops doc group list` | List document groups |
| `aops doc group update` | Update a document group |
| `aops doc import` | Import structured Docman content from source files |
| `aops doc index` | Docman retrieval index commands |
| `aops doc index build` | Build or refresh the persisted retrieval index for a saved document version |
| `aops doc link` | Docman structure linking commands |
| `aops doc link page` | Link or unlink pages in a section |
| `aops doc link section` | Link or unlink sections in a document outline |
| `aops doc link section delete` | Alias for doc section unlink |
| `aops doc list` | List Docman documents through the hosted gateway |
| `aops doc mirror` | Docman v2 structured and v3 source-file mirror helpers |
| `aops doc mirror pull` | Materialize structured v2 .md mirrors and exact v3 source files with adjacent sidecars |
| `aops doc mirror push` | Import root markdown files or guarded-push one v3 source file with its adjacent sidecar |
| `aops doc order` | Docman outline ordering commands |
| `aops doc order pages` | Update section-page-link ordering through the canonical flow surface |
| `aops doc order sections` | Update document-section-link ordering through the canonical flow surface |
| `aops doc outline` | Docman normalized outline inspection |
| `aops doc outline get` | Read a section-centric normalized outline for a saved document version |
| `aops doc page` | Docman page and page-version commands |
| `aops doc page copy` | Copy a page into a section through the canonical flow surface |
| `aops doc page create` | Create a page with its first draft and optionally link it into a document tree |
| `aops doc page draft-save` | Create or update a page-version draft through the canonical flow surface |
| `aops doc page get` | Get a page by id |
| `aops doc page list` | List pages |
| `aops doc page membership-unlink` | Delete a reusable section-page-link without deleting the page |
| `aops doc page move` | Move a section-page-link to another section |
| `aops doc page unlink` | Unlink a page node created in the document outline without deleting the page |
| `aops doc page update` | Update Docman page metadata through the canonical flow surface |
| `aops doc page-version` | Docman page-version commands |
| `aops doc page-version get` | Get a page version by id |
| `aops doc page-version list` | List page versions |
| `aops doc page-version update` | Update page-version metadata; content changes must use doc page draft-save |
| `aops doc publish` | Materialize saved document content to markdown, html, or exact source |
| `aops doc scope` | Docman scope-owned retrieval commands |
| `aops doc scope answer` | Read a citation-first answer pack across latest document versions in one scope |
| `aops doc scope search` | Search persisted Docman retrieval rows across latest document versions in one scope |
| `aops doc search` | Search persisted Docman retrieval rows for a saved document version |
| `aops doc section` | Docman section commands |
| `aops doc section copy` | Copy a section into a document version through the canonical flow surface |
| `aops doc section create` | Create a section and optionally link it into a document outline |
| `aops doc section get` | Get a section by id |
| `aops doc section list` | List sections |
| `aops doc section unlink` | Unlink a section from a document outline without deleting the section |
| `aops doc section update` | Update a Docman section through the canonical flow surface |
| `aops doc set-current-version` | Flip a document’s canonical current version atomically (peer-clear + optional publish + publishedAt). Dispatches docman.document-version.set-current. |
| `aops doc source` | Fetch exact composed source for a saved document fragment |
| `aops doc source-file` | Immutable Docman text-file snapshot commands |
| `aops doc source-file save` | Save an exact UTF-8 source file as a new immutable text-file document version |
| `aops doc summary` | Docman persisted summary commands |
| `aops doc summary build` | Build or refresh persisted summaries for a saved document version; index rows are ensured first |
| `aops doc update` | Update a Docman document through the canonical flow surface |
| `aops doc version` | Docman document-version commands |
| `aops doc version create` | Create a document version through the canonical flow surface |
| `aops doc version get` | Get a document version by id |
| `aops doc version list` | List document versions |
| `aops doc version update` | Update document-version header metadata (status / title / summary / release-notes / label). To switch the canonical current version, use `aops-cli doc set-current-version` instead. |

<!-- aops-generated:docman-command-catalog:end -->

<!-- aops-generated:docman-discovery:start -->

### 11.3 Generated Docman discovery and retrieval guide

#### 11.3.1 Overview

> Docman operations and schemas are discovered from the running server. This appendix keeps the retrieval gates current without copying a fixed hosted-tool inventory into prose.

| Command | Purpose |
| --- | --- |
| `aops doc outline get` | Read a section-centric normalized outline for a saved document version |
| `aops doc index build` | Build or refresh the persisted retrieval index for a saved document version |
| `aops doc summary build` | Build or refresh persisted summaries for a saved document version; index rows are ensured first |
| `aops doc search` | Search persisted Docman retrieval rows for a saved document version |
| `aops doc scope search` | Search persisted Docman retrieval rows across latest document versions in one scope |
| `aops doc answer` | Read a citation-first deterministic answer pack for a saved document version |
| `aops doc source` | Fetch exact composed source for a saved document fragment |
| `aops doc mirror pull` | Materialize structured v2 .md mirrors and exact v3 source files with adjacent sidecars |
| `aops agent tools` | List federated tools from the canonical operator plane (/api/agent/tools) |
| `aops agent schema` | Print the live JSON Schema for one tool's input contract — use this before authoring --input payloads |

Use the smallest useful read:

```bash
aops doc search --local --q "<keywords>" --ensure summary --json
aops doc scope search --project-slug <slug> --q "<keywords>" --json
aops doc answer --document-version-id <id> --q "<question>" --ensure summary --json
aops agent tools --domain docman --summary --json
aops agent schema --tool <docman-tool-id> --summary --json
```

<!-- aops-generated:docman-discovery:end -->
