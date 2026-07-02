---
name: extension-maintenance
description: Manage Claude Code and Codex skills/plugins with a safe workflow for inventory, updates, installs, cache cleanup, folder normalization, verification, and change records.
---

# Extension Maintenance

Use this skill when the user asks to manage Claude Code or Codex extensions, including:

- inventorying installed skills, plugins, marketplaces, or caches
- updating skills or plugins
- adding, reinstalling, moving, or removing skills/plugins
- checking upstream sources and updateability
- cleaning orphaned or superseded caches
- verifying Claude/Codex consistency
- writing maintenance records

## Core Rule

Treat extension maintenance as a controlled change:

1. Inventory first.
2. Define success criteria.
3. Back up affected state before mutation.
4. Use official CLI/marketplace/lock metadata where possible.
5. Change only the requested extension-related paths.
6. Verify with commands and hashes.
7. Record what changed.

Never disclose secrets from config files, tokens, API keys, or plugin credentials.

## Path Policy

Resolve paths from the current machine instead of assuming one user's home directory. On Windows, prefer these defaults:

- shared/global skills: `$env:USERPROFILE\.agents\skills`
- Claude Code skills: `$env:USERPROFILE\.claude\skills`
- Codex plugin cache: `$env:USERPROFILE\.codex\plugins\cache`
- Claude plugin cache: `$env:USERPROFILE\.claude\plugins\cache`
- Claude marketplaces: `$env:USERPROFILE\.claude\plugins\marketplaces`
- managed maintenance root: `$env:USERPROFILE\Documents\AI-Extensions`

If the user has a custom layout, use their layout after confirming it from local files or their instructions.

## Required Workflow

### 1. Define Success Criteria

Before changing files, state:

- affected side: Claude Code, Codex, or both
- intended placement: shared, Claude-only, or Codex-only
- expected final path
- expected update source: marketplace, GitHub repository, lock file, or local/manual source
- verification command or hash comparison

### 2. Inventory Current State

Use the relevant commands in `references/commands.md`.

For each item, classify updateability:

- `auto-checkable`: marketplace or lock metadata is sufficient for update checks
- `manual-review`: local clone/copy/manual source lacks reliable upstream metadata
- `not-applicable`: bundled runtime, generated cache, disabled orphan, or intentionally local-only

### 3. Back Up Before Mutation

Create a timestamped backup under the user's managed maintenance root, for example:

`<AI_EXTENSIONS_HOME>\backups\yyyyMMdd-HHmmss-<task-name>`

Back up only affected directories or metadata files.

Before recursive delete or move operations on Windows:

- resolve absolute paths first
- confirm every resolved path stays under the intended root
- use one shell end-to-end
- prefer PowerShell `Copy-Item`, `Move-Item`, and `Remove-Item` with `-LiteralPath`

### 4. Update Or Install Through The Right Channel

Use official commands whenever possible:

- global skills: `npx --yes skills add ... -g -a claude-code codex --copy --full-depth -y`
- global skill updates: `npx --yes skills update -g -y`
- Claude plugins and marketplaces: `claude plugin ...`

If a repository contains multiple nested skills, install selected sub-skills explicitly with repeated `-s <skill>` flags and use `--full-depth`.

If an item cannot be updated because upstream metadata is missing, reinstall from a verified original source instead of editing cache files by hand.

### 5. Normalize Folder Placement

Use this placement rule:

- shared skill for both agents: install/sync to both `.agents\skills\<name>` and `.claude\skills\<name>` when the local tools require both copies
- Codex-only skill: keep under `.agents\skills\<name>` and mark it Codex-only
- Claude-only skill: keep under `.claude\skills\<name>` unless the user asks to share it
- Claude plugin: keep managed by Claude plugin cache/marketplace
- Codex plugin: keep managed by Codex plugin cache

Do not move plugin cache directories into skill directories. Cache is runtime state; skill directories are user-facing extension state.

### 6. Verify

After every change:

- run `npx --yes skills list -g --json`
- run `claude plugin list --json` if Claude plugins were affected
- compare `SKILL.md` hashes when a shared skill exists in both `.agents` and `.claude`
- confirm removed caches no longer exist
- confirm replacement/current caches still exist when cleaning old caches
- confirm relevant Git or marketplace state is clean when a marketplace repository was touched

### 7. Record The Change

Write a maintenance record under:

`<AI_EXTENSIONS_HOME>\inventory`

Use `references/change-record-template.md`. Include date/time, task summary, before/after paths, commands, update results, backups, verification, and remaining risks.

## Common Decisions

### Shared Claude/Codex Skill

Preferred command pattern:

```powershell
cmd /c npx --yes skills add <owner>/<repo> -g -a claude-code codex -s <skill> --copy --full-depth -y
```

Verify both copies:

```powershell
Get-FileHash "$env:USERPROFILE\.agents\skills\<skill>\SKILL.md" -Algorithm SHA256
Get-FileHash "$env:USERPROFILE\.claude\skills\<skill>\SKILL.md" -Algorithm SHA256
```

### Marketplace Plugin Problem

If Claude reports that a marketplace cannot be found:

1. Check `claude plugin list --json`.
2. Check `claude plugin marketplace list --json`.
3. Inspect the marketplace repository for `marketplace.json`.
4. If upstream no longer provides marketplace metadata, uninstall the broken plugin/marketplace entry and reinstall underlying skills from a verified source when possible.

### Orphaned Cache Cleanup

Clean a cache only when all are true:

- the cache is not referenced by current plugin or skill inventory
- a current replacement path exists if the cache has been superseded
- a backup was created
- the resolved absolute path is under an intended cache root

## Supporting Files

Read these before acting:

- `references/commands.md`
- `references/change-record-template.md`
