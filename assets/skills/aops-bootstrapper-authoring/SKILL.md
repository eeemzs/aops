---
name: aops-bootstrapper-authoring
version: 10
description: "Use when an AI agent creates or updates canonical AOPS hosted skills/prompts and prepares the simple four-kind public asset release through ignored .aops-cache mirrors and documented pnpm scripts."
metadata:
  supersedes: "v9"
  short-description: "AOPS hosted authoring and simple public asset preparation"
  tags:
    - aops
    - bootstrapper
    - prompt-authoring
    - skill-authoring
    - public-assets
---

# AOPS hosted authoring and public asset preparation

Use this skill for canonical AOPS hosted prompt/skill authoring and for preparing the public four-kind asset release. Keep server authoring, repo cache refresh, release preparation, and global installation as separate steps.

## Ownership

1. Hosted skill and prompt truth lives in the canonical AOPS server under `slug:aops`.
2. `.aops-cache/hosted/**` and `.aops-cache/docman/**` are ignored read-only mirrors. Never hand-edit them as truth.
3. `release/public-aops-assets-inventory.json` is the tracked allowlist and immutable-version pin set; canonical public classification is the independent second export gate.
4. The public release may contain only skills, user guides, working disciplines, and roles.
5. Global Codex/Claude installation is performed by `aops assets`; it is not part of hosted authoring or mirror refresh.

## Hosted skill or prompt workflow

Read live help first:

```bash
aops skill --help
aops skill version create --help
aops skill version publish --help
aops prompt --help
aops prompt version create --help
aops prompt version publish --help
```

Then:

1. List or inspect the existing shell and current version.
2. Create a complete candidate in a temporary file.
3. Create the next version with the correct entry file, standard, and metadata.
4. Read the created version back.
5. Publish it only when publication/current authority is present.
6. Refresh the ignored hosted mirror:

   ```bash
   aops sync pull --only hosted-skills --apply --hosted-project-slug aops --json
   ```

Do not patch mirror Markdown or globally installed skills to simulate a canonical update.

## Public asset preparation

Public export is opt-in and fail-closed. An exact inventory row is necessary but not sufficient: a skill shell also needs `public-agent-asset` and `public-distribution`; a user guide must be published/public, live in `aops-guides` or `domain-guides`, and carry `user-guide`, `public-asset`, and `public-distribution`. Metadata never auto-adds an item to inventory, and inventory cannot override missing/private classification.

Refresh only required hosted-skill and Docman mirror partitions into `.aops-cache`, then preview current version-pin and target drift:

```bash
pnpm run release:aops-assets:sync -- \
  --target-root <public-aops-assets-root> \
  --refresh-inventory
```

Apply the reviewed inventory refresh and generated public tree, then re-run the pinned check:

```bash
pnpm run release:aops-assets:sync -- \
  --target-root <public-aops-assets-root> \
  --refresh-inventory \
  --apply

pnpm run release:aops-assets:sync -- \
  --target-root <public-aops-assets-root>
```

Normal pinned sync fails closed when a canonical mirror version changed. `--refresh-inventory` previews the pin changes; it writes only with `--apply`.

Build only after the separate AOPS version gate:

```bash
pnpm run release:aops-assets:build -- \
  --source-root <public-aops-assets-root> \
  --output <temporary-output> \
  --version <version>
```

Build twice and require byte-identical `release.json` and `aops-assets.json.gz`.

## Validation

Require:

- exact inventory membership, canonical public classification, expected four-kind counts, and no unexpected files;
- no maintainer-only skill, credential, machine-local path, symlink, or unsupported path;
- disposable-HOME install, repair, update, status, and rollback;
- removal of an asset retired by the new manifest;
- preservation of unrelated user skills;
- removal of `~/.aops/agent-assets` only after successful new installation.

The TUI delegates to the same CLI lifecycle. Do not build a second installer.

## Effect boundaries

Hosted publish, tracked inventory changes, versioning, GitHub Release creation, npm publication, workflow dispatch, and global installation are distinct effects. Execute only the effects authorized for the current slice.

## Anti-patterns

1. Restoring pointer-sync or asset-gateway commands.
2. Installing globals as a side effect of hosted authoring or release preparation.
3. Hand-editing `.aops-cache/**` wrappers or version ids.
4. Replacing a published archive under the same version.
5. Copying private maintainer/release skills into public assets.
