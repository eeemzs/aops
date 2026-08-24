---
name: aops-cli-sugar-authoring
description: "Use when an AI agent needs to author or extend a sugar CLI command (eops-cli or aops-cli) over a hosted domain operation. Thin discipline guide: identifies the right command surface, points to canonical authoring sources, and keeps every new sugar wired to `agent schema` discovery so callers stop guessing payload fields."
metadata:
  short-description: "AOPS/EOPS CLI sugar-authoring thin discipline guide"
  tags:
    - cli
    - authoring
    - sugar
    - host-plugin
    - agent-schema
---

# AOPS/EOPS CLI Sugar Authoring

This skill is a thin authoring playbook for adding or extending sugar CLI
commands on `aops-cli` or `eops-cli` over an already-registered hosted domain
operation. It does **not** cover authoring new operations inside a domain kit
or host plugin — that path is the kit/host-plugin authoring flow.

When this skill conflicts with the architecture doc or `--help`, those win.

## When To Use This Skill

1. Adding a new `aops-cli <domain> <subcommand>` or `eops-cli <domain> <subcommand>` over a hosted operation that already exists in the kit/host-plugin manifest.
2. Extending or splitting an existing sugar (new option, new alias, new validation).
3. Wiring a composite sugar that orchestrates multiple hosted operations under one CLI entry.

Out of scope: adding a new operation to a domain kit, exposing it through the host plugin, or changing the federated catalog. Use the kit/host-plugin authoring flow for those — see slug:aops `tooling-cli-host-plugin-system`, the "Domain Kit Katmanı" section (search: domain kit, operation contract) and the "Domain Capability Manifest (DCM)" section (search: DCM, capability manifest, discovery metadata).

## Canonical Sources

When this skill is silent, ambiguous, or out of date, defer to these:

1. slug:aops `tooling-cli-host-plugin-system`, the "Agent Gateway" section (search: agent gateway, tool invoke envelope, public discovery surface) and the "CLI Sistemi" section (search: CLI sistemi, CLI dispatch, registration) — canonical mechanics for invoke routing, public discovery surface, and CLI dispatch.
2. slug:aops `tooling-cli-host-plugin-system`, the "Tool Input JSON Schema Discovery" section (search: tool input json schema, agent schema, inputJsonSchema) — the `agent schema` contract every new sugar must integrate with.
3. `aops-cli <command> --help` / `eops-cli <command> --help` for live flag truth.
4. Existing sugar source files (read these as templates):
   - aops-cli, commander-based: `apps/aops/apps/aops-cli/src/commands/agent.ts`, `apps/aops/apps/aops-cli/src/commands/docman.ts`
   - eops-cli, manual argv + topic registry: `apps/eops/apps/eops-cli/src/commands/inventory.ts` (help registry) and `apps/eops/apps/eops-cli/src/commands/inventory-bom.ts` (builder dispatch)

## Pre-Authoring Discovery

Before writing any new sugar:

```bash
# Confirm the target operation is registered
aops-cli agent tools --domain <id> --json | jq '.tools[] | select(.toolId | test("<op-substr>"))'
# OR
eops-cli agent tools --domain <id> | grep <op-substr>

# Pull the live input contract — this is the payload-shape contract you wrap
aops-cli agent schema --tool <domain>.<operation>
eops-cli agent schema --tool <domain>.<operation>

# Read the kit's host-route projection to see the HTTP shape (params, method)
# broad search across the project's guides (local mirror, no id needed):
aops-cli doc scope search --project-slug aops --q "host-projection" --local --json
# exact search within a known guide version:
aops-cli doc search --document-version-id <docver-id> --q "host-projection" --local --json
# section tree of a known guide version (hosted-only — no local fallback):
aops-cli doc outline get --document-version-id <docver-id> --json
```

There is no `aops-cli docman … --slug` command — use the ladder; reference sections by document title + section name + keywords, not bare numbers.

Decisions you should resolve before writing code:

1. Is this a thin wrapper over a single operation or a composite over multiple?
2. Does it need a request body (`--data`/`--input`/`--patch`) or only positional/route args?
3. Is it a hosted write (`--apply`/`--confirm` guards) or read-only?
4. Which CLI is the home (aops-cli or eops-cli)? Operator surfaces follow the app that owns the domain; if both apps consume the operation, prefer the canonical operator plane (aops-cli).

## Authoring Pattern — aops-cli (commander)

aops-cli uses `commander`. Help is auto-generated from `.command/.description/.option` calls.

1. Open the domain commander file: `apps/aops/apps/aops-cli/src/commands/<domain>.ts` (e.g. `agent.ts`, `docman.ts`, `projectman.ts`).
2. Inside the `make<Domain>Command()` factory append a new subcommand:

   ```ts
   applyCommonOptions(
     cmd
       .command('<sub>')
       .description('<one-line description of what it does>')
       .option('--<flag> <value>', '<help text>')
       .action(async (options: <SubOptions>) => {
         await run<Sub>(options)
       }),
     { withProject: true /* or false for read-only/global */ }
   )
   ```

3. Implement `run<Sub>(options)` next to the other run functions. Reuse:
   - `requireApiState(options)` for auth/base-url plumbing
   - `buildAgentContextHeaders(options)` for `x-tenant-id`/`x-scope-id`/etc.
   - `invokeHostedToolWithApiState(apiState, { toolId, input, ... })` for hosted invokes
   - `apiState.client.fetchJson(...)` for direct discovery/read endpoints
   - `unwrapHostedToolResult(payload)` to surface the data result
   - `parseJsonInput(options.input, 'input')` for JSON payloads / `@file.json`
4. Add an example to `cmd.addHelpText('after', ...)` showing the canonical workflow. Always include the `agent schema --tool <id>` pre-step when the command takes `--data`/`--input`/`--patch`.

## Authoring Pattern — eops-cli (manual argv + topic registry)

eops-cli renders `--help` from a declarative topic registry and dispatches via builder functions.

1. Open the topic registry file: `apps/eops/apps/eops-cli/src/commands/<domain>.ts` (e.g. `inventory.ts`).
2. Append a topic entry:

   ```ts
   {
     route: ['<domain>', '<entity>', '<sub>'],
     summary: '<one-line summary>',
     usage: [
       'eops-cli <domain> <entity> <sub> --<flag> <value> [--apply --confirm]',
     ],
     options: [
       '--data / --payload / --file: JSON object forwarded as <entity> data',
       '--apply --confirm: required together for hosted writes',
       '--idempotency-key: optional hosted write idempotency key when --apply is used',
     ],
     notes: [
       // CRITICAL: always include this notes line for write-shaped sugars.
       'Run `eops-cli agent schema --tool <domain>.<operation>` to fetch the live input contract before authoring --data payloads.',
       // Plus any non-obvious field-ownership / lifecycle notes.
     ],
     examples: [
       'eops-cli <domain> <entity> <sub> --data @./payload.json',
       'eops-cli <domain> <entity> <sub> --data @./payload.json --apply --confirm',
     ],
   },
   ```

3. Add a builder function `build<Sub>Command(section, action, parsed)` in the builder file (e.g. `inventory-bom.ts`). Return `{ action, operationId, input, write? }`.
4. Wire the builder into the dispatch chain so the right action returns from `build<Domain>Command(section, action, parsed)`.
5. Verify with `pnpm --filter @eops/eops-cli run build && eops-cli <domain> <entity> <sub> --help`.

## Schema Discovery Integration (Required)

Every new sugar that accepts `--data` / `--input` / `--patch` must point operators to `agent schema --tool <toolId>` in its help/notes. Reasons:

1. `agent schema` is the canonical pre-authoring contract; without the pointer, operators and AI agents will guess field names and hit `Invalid input` from `z.union`-shaped validators.
2. Consistency across the kit family — every operator skill already cross-references this step.
3. When the kit later changes a field, `agent schema` reflects the change immediately; inline help-text field lists drift.

Read-only sugars (list/get/search/view) do not require the pointer.

## Always Rules

1. Reuse the canonical helpers — never reinvent `--apply`/`--confirm`/`--idempotency-key` parsing.
2. Hosted writes go through `invokeHostedToolWithApiState` (aops-cli) or the existing hosted dispatch in eops-cli — never bypass the gateway.
3. Help text for write sugars MUST include the `agent schema --tool <toolId>` line.
4. After authoring, run `--help` and `agent schema --tool <id>` against the new command path to verify discoverability before declaring done.
5. Run a `--preview` invoke (where supported) and a real `--apply --confirm` invoke for at least one happy-path payload.

## Top Anti-Patterns

1. Authoring a sugar that bypasses the agent gateway and talks to the domain DB directly.
2. Hardcoding payload field names in help text without referencing `agent schema`; the help drifts as the kit evolves.
3. Adding a write sugar without `--apply`/`--confirm` guards.
4. Copy-pasting argv parsing instead of importing the shared option helpers.
5. Creating a sugar where `agent invoke --tool <id>` would suffice — sugar is justified by composite behavior, better naming, or operator ergonomics, not by personal preference.
6. Forgetting to wire the new sugar into the dispatch chain (eops-cli) or the `make<Domain>Command()` factory (aops-cli).

## Discovery Shortcuts

```bash
# broad search across the project's guides (local mirror, no id needed):
aops-cli doc scope search --project-slug aops --q "agent gateway" --local --json
aops-cli doc scope search --project-slug aops --q "agent schema" --local --json
# exact search within a known guide version:
aops-cli doc search --document-version-id <docver-id> --q "agent gateway" --local --json
# section tree of a known guide version (hosted-only — no local fallback):
aops-cli doc outline get --document-version-id <docver-id> --json
aops-cli agent tools --domain <id> --json
aops-cli agent schema --tool <domain>.<operation>
```

There is no `aops-cli docman … --slug` command — use the ladder; reference sections by document title + section name + keywords, not bare numbers.

## Sister Skills

- `aops-cli-core` — help-first discovery, guard flag conventions, `agent invoke` fallback.
- `aops-cli-{agentspace,discuss,chat,docman,fileman,projectman,tasker}` — operator playbooks for each domain; consult before designing sugar UX.
- `eops-cli-bom-authoring`, `eops-cli-sourcing-order-workbook` — concrete eops-cli sugar examples covering both read and write surfaces.
- `aops` / `eops` — router indexes.

If `--help` and this skill disagree, `--help` wins. If the architecture doc and this skill disagree, the architecture doc wins.
