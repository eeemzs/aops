---
name: aops-cli-fileman
version: 7
description: "Use when an AI agent needs AOPS CLI Fileman operator playbook: target tracking, snapshot/file lineage, diff/restore/copy/zip, clean, and content-pointer-only owner boundary. Thin guide; command --help and domains/fileman/USER_GUIDE.md are authoritative."
metadata:
  supersedes: "v6"
  short-description: "AOPS CLI Fileman thin discipline guide"
  tags:
    - cli
    - fileman
    - snapshot
    - diff
    - restore
    - copy
    - backup
    - target
    - retention
    - help-first
---

# AOPS CLI Fileman

Fileman owns filesystem target tracking, immutable snapshots, file lineage, restore/copy/zip exports, and cleanup. `aops-cli file ...` is only the operator sugar surface. When this skill conflicts with live `--help` or `domains/fileman/USER_GUIDE.md`, those win.

## Use This Skill For

1. Track a directory root and capture a baseline snapshot.
2. Create/list/get/diff/rebase/restore/copy/zip snapshots.
3. Read file history, file content, file diffs, and single-file restore/extract.
4. Clean stale Fileman state through the hosted graph.
5. Decide when to use a Fileman pointer instead of pasting content into PM, Docman, or memory.

Use `aops-cli-core` for guard flags and schema fallback; use `aops-cli-docman` for document content; use `aops-cli-agentspace` for memory/resource/artifact/skill surfaces.

## Authoritative Sources

1. Live command shape:
   - `aops-cli file --help`
   - `aops-cli file target --help`
   - `aops-cli file target track --help`
   - `aops-cli file snapshot --help`
   - `aops-cli file file --help`
   - `aops-cli file clean --help`
2. Domain semantics: `domains/fileman/USER_GUIDE.md` and `domains/fileman/architecture.md`.
3. Raw hosted contracts only when sugar is insufficient: `aops-cli agent schema --tool fileman.<operation> --json`.

There is no default `.aops/fileman/**` authoring mirror. Fileman is hosted-canonical; the repo normally stores only references to Fileman ids or storage pointers.

## Live Command Map

| Need | Command |
|---|---|
| Track a root and baseline snapshot | `aops-cli file target track --root-path <dir> --apply --json` |
| Orient before mutation | `aops-cli file inspect target --target-id <id> --json`; for one file use `aops-cli file inspect file --target-id <id> --path <rel> --json` |
| Read target inventory | `aops-cli file target list|get` |
| Create/list/get snapshots | `aops-cli file snapshot create|list|get` |
| Diff snapshots | Prefer `aops-cli file snapshot diff --from-snapshot-id <older> --to-snapshot-id <newer>`; use `--target-id <id>` only when Fileman's default pair/direction is acceptable |
| Rebase baseline | `aops-cli file snapshot rebase --target-id <id> --dry-run --preview` then `--apply --confirm` |
| Restore snapshot tree | `aops-cli file snapshot restore --snapshot-id <id> --dry-run`, then `--apply --confirm` |
| Copy snapshot tree | `aops-cli file snapshot copy --target-id <id> --output-path <dir> --apply` |
| Zip snapshot tree | `aops-cli file snapshot zip --target-id <id> --output-path <staging> --zip-path <zip> --apply` |
| File history | `aops-cli file file history --target-id <id> --path <rel>` |
| File content | `aops-cli file file get --snapshot-id <snapshot-id> --path <rel>` |
| File diff | `aops-cli file file diff --from-file-version-id <a> --to-file-version-id <b>` |
| Restore one file | `aops-cli file file restore --target-id <id> --path <rel> --dry-run`, then `--apply --confirm` |
| Cleanup | `aops-cli file clean --target-id <id> --dry-run --preview`, then `--apply --confirm` |

Important correction: live CLI nests diff/restore/copy/zip under `file snapshot ...` and single-file operations under `file file ...`. Do not use stale top-level shapes such as `aops-cli file restore` or `aops-cli file diff`.

## Root Path Rule

`file target track --root-path` tracks a directory root. If the operator gives a single file, track its parent directory and use the file-relative path through `file file ...`:

```bash
aops-cli file target track --name "<anchor>" --root-path "<parent-dir>" --apply --json
aops-cli file file history --target-id <target-id> --path "<relative/file.ext>" --json
```

Current CLI/server guard rejects non-directory roots with `target_path_not_directory:<path>`.

## Common Workflows

Safety shorthand: `--preview` validates the hosted mutation envelope; `--dry-run` is Fileman domain input that previews filesystem or cleanup effects. For destructive restore/rebase/clean, run dry-run first, then rerun with `--apply --confirm` only after reviewing the plan.

### Track And Snapshot

```bash
aops-cli file target track --name "<anchor>" --root-path "<absolute-dir>" \
  --tag "<tag>" --label "<baseline label>" --message "<why>" \
  --apply --yes --json

aops-cli file snapshot create --target-id <target-id> \
  --label "<checkpoint>" --message "<why>" --apply --json
```

For JSON handoff, read ids from `result.data.targetId`, `result.data.snapshotId`, or the mirrored `artifacts.targetId` / `artifacts.snapshotId` convenience fields. Do not assume nested `snapshot.id`; Fileman snapshot DTOs expose `snapshotId`.

### Diff And Restore

```bash
aops-cli file snapshot diff --target-id <target-id> --json
aops-cli file snapshot diff --from-snapshot-id <base> --to-snapshot-id <head> --json

aops-cli file snapshot restore --snapshot-id <snapshot-id> --dry-run --json
aops-cli file snapshot restore --snapshot-id <snapshot-id> \
  --backup-before-restore --apply --confirm --json
```

For chronological diffs, prefer explicit older-to-newer ids. `file snapshot diff --target-id` lets Fileman pick the pair and direction, which may be surprising when an agent expects an addition-style patch.

### Copy Or Zip For Handoff

```bash
aops-cli file snapshot copy --target-id <target-id> --snapshot-id <snapshot-id> \
  --output-path "<output-dir>" --apply --json

aops-cli file snapshot zip --target-id <target-id> --snapshot-id <snapshot-id> \
  --output-path "<staging-dir>" --zip-path "<archive.zip>" --apply --json
```

### Single-File Recovery

```bash
aops-cli file file history --target-id <target-id> --path "<relative/file.ext>" --json
aops-cli file file get --snapshot-id <snapshot-id> --path "<relative/file.ext>" --json
aops-cli file file diff \
  --from-file-version-id <base-file-version-id> \
  --to-file-version-id <head-file-version-id> --json

aops-cli file file restore --target-id <target-id> --path "<relative/file.ext>" \
  --snapshot-id <snapshot-id> --dry-run --json
aops-cli file file restore --target-id <target-id> --path "<relative/file.ext>" \
  --snapshot-id <snapshot-id> --backup-before-restore --apply --confirm --json
```

`file file history` returns `result.data.versions[]`; use each row's `fileVersionId` for `file file diff`. `file file get` uses `--snapshot-id` + `--path`; it does not take `--target-id`.

### Cleanup

```bash
aops-cli file clean --target-id <target-id> --dry-run --preview --json
aops-cli file clean --target-id <target-id> --missing-only --apply --confirm --json
```

`--dry-run` is cleanup input, not the command guard. Pair it with `--preview` to inspect cleanup without mutation. To remove a temporary tracked target, make the root missing or target a missing root, then run `file clean --target-id <id> --missing-only --apply --confirm --json`. Fileman does not currently expose `file target delete` or `file target untrack` sugar.

Use `file clean` instead of manually deleting snapshot files; manual deletion can break hosted graph consistency.

## Deep Reading

Use targeted reads, not full dumps:

```bash
aops-cli doc scope search --project-slug aops --q "fileman snapshot restore" --local --json
aops-cli doc search --document-version-id <docver-id> --q "restore" --local --json
aops-cli view doc-page <document-slug>#<section-slug> --max-bytes 6000
aops-cli doc outline get --document-version-id <docver-id> --titles-only --depth 2 --json
```

`doc scope search` is broad and can rank source docs above guide docs. `doc search --local` is retrieval/search, not a guaranteed full section-body reader. Use `view doc-page` when you need the actual mirrored section body. There is no `aops-cli docman ... --slug` command.

## Anti-Patterns

1. Guessing stale top-level commands instead of live nested help.
2. Tracking a single file as `--root-path`; track the parent directory and use relative file operations.
3. Treating live path state as snapshot history; snapshots are immutable history and live state may drift.
4. Embedding large file bodies into memory/PM/Docman instead of referencing Fileman target/snapshot/file-version ids.
5. Restoring without dry-run and explicit `--apply --confirm`.
6. Manually deleting snapshot storage instead of `aops-cli file clean`.
