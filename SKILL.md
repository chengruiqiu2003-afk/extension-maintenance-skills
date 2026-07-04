---
name: extension-maintenance
description: Manage Claude Code and Codex skills/plugins/MCP servers/apps with a safe workflow for inventory, updates, installs, cache cleanup, folder normalization, verification, and change records.
---

# Extension Maintenance

Use this skill when the user asks to manage Claude Code or Codex extensions, including:

- inventorying installed skills, plugins, marketplaces, or caches
- inventorying configured MCP servers or Codex Apps/connectors
- updating skills or plugins
- updating locally managed MCP server packages or containers
- adding, reinstalling, moving, or removing skills/plugins
- adding, importing, disabling, or removing MCP server configuration
- checking upstream sources and updateability
- cleaning orphaned or superseded caches
- verifying Claude/Codex consistency
- verifying MCP tool availability and connector health
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
- Codex MCP config: `$env:USERPROFILE\.codex\config.toml`
- Claude user MCP/config state: `$env:USERPROFILE\.claude.json` and `$env:USERPROFILE\.claude`
- project MCP config: `.mcp.json` in the relevant project root
- managed maintenance root: `$env:USERPROFILE\Documents\AI-Extensions`

If the user has a custom layout, use their layout after confirming it from local files or their instructions.

## Required Workflow

### 1. Define Success Criteria

Before changing files, state:

- affected side: Claude Code, Codex, or both
- intended placement: shared, Claude-only, or Codex-only
- expected final path
- expected update source: marketplace, GitHub repository, lock file, package registry, container image, remote MCP URL, connector provider, or local/manual source
- verification command, hash comparison, MCP tool probe, or connector health check

### 2. Inventory Current State

Use the relevant commands in `references/commands.md`.

For each item, classify updateability:

- `auto-checkable`: marketplace or lock metadata is sufficient for update checks
- `manual-review`: local clone/copy/manual source lacks reliable upstream metadata, or an MCP command points to an unpinned local executable
- `not-applicable`: bundled runtime, generated cache, disabled orphan, or intentionally local-only

For MCP servers and Codex Apps/connectors, also classify the management layer:

- `plugin-managed`: an MCP server bundled by a Claude/Codex plugin; update the plugin, not the server config by hand
- `package-managed`: a local MCP command such as `npx`, `uvx`, `python`, `node`, or `docker`; update the package, lock file, executable, or image that the command uses
- `remote-managed`: an HTTP/SSE/streamable HTTP MCP server; the provider updates the server, while local maintenance covers URL, auth, and reachability
- `connector-managed`: a Codex App/connector such as GitHub or BioRender; local maintenance covers connection state, discovery/runtime cache, tool probes, and escalation records

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
- Claude MCP servers: `claude mcp ...`
- Codex MCP servers: `codex mcp ...` and `~/.codex/config.toml`

If a repository contains multiple nested skills, install selected sub-skills explicitly with repeated `-s <skill>` flags and use `--full-depth`.

If an item cannot be updated because upstream metadata is missing, reinstall from a verified original source instead of editing cache files by hand.

Do not treat MCP itself as the updatable artifact. Update the carrier:

- plugin-managed MCP: update the owning plugin
- package-managed MCP: update the npm/pip/uv/Docker/Git source used by the command
- remote-managed MCP: verify URL/auth and let the provider update the service
- connector-managed MCP/App: reconnect, refresh local caches, verify tools, and escalate upstream mismatches

### 5. Normalize Folder Placement

Use this placement rule:

- shared skill for both agents: install/sync to both `.agents\skills\<name>` and `.claude\skills\<name>` when the local tools require both copies
- Codex-only skill: keep under `.agents\skills\<name>` and mark it Codex-only
- Claude-only skill: keep under `.claude\skills\<name>` unless the user asks to share it
- Claude plugin: keep managed by Claude plugin cache/marketplace
- Codex plugin: keep managed by Codex plugin cache
- Claude user MCP: keep in Claude's user/global scope unless project isolation is required
- Claude project MCP: keep in `.mcp.json` only when the server is intentionally project-scoped
- Codex MCP: keep in Codex MCP configuration; if imported from Claude Code, record the source and avoid duplicate divergent entries
- Codex App/connector: keep managed by Codex; do not move connector/plugin cache directories into skill or MCP config directories

Do not move plugin cache directories into skill directories. Cache is runtime state; skill directories are user-facing extension state.

### 6. Verify

After every change:

- run `npx --yes skills list -g --json`
- run `claude plugin list --json` if Claude plugins were affected
- run `claude mcp list` if Claude MCP servers were affected
- run `codex mcp list` if Codex MCP servers were affected
- compare `SKILL.md` hashes when a shared skill exists in both `.agents` and `.claude`
- compare Claude/Codex MCP names, scopes, commands, URLs, environment variable names, and source notes when the same MCP is shared or imported
- probe at least one non-destructive tool for each affected MCP server or connector when tools are available
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

### MCP Maintenance

Use official MCP configuration commands first:

1. Inventory Claude with `claude mcp list` and Codex with `codex mcp list`.
2. Identify scope and source: Claude local/project/user scope, project `.mcp.json`, Codex `~/.codex/config.toml`, plugin-managed, imported from Claude Code, or Codex App/connector.
3. Classify the carrier as plugin-managed, package-managed, remote-managed, or connector-managed.
4. Back up only the affected config files or cache directories before mutation.
5. Update the carrier, not the protocol:
   - npm/npx: update the package or pinned version
   - uv/uvx/pip/python: update the package, environment, or lock file
   - Docker: pull or pin the intended image tag
   - Git clone/local executable: pull from the verified upstream or reinstall from the original source
   - remote URL: verify endpoint, auth, and provider status
   - Codex App/connector: reconnect, clear only verified generated caches, and escalate backend tool mismatches
6. When Codex MCP entries are imported from Claude Code, record the import source and do not edit both sides independently unless the user wants them to diverge.
7. Verify with `claude mcp list`, `codex mcp list`, and a non-destructive tool call when possible.

Never copy secrets from one config file to another in plain text. Preserve environment variable references and redact credential values in records.

### Codex App Or Connector Problem

Treat Codex Apps/connectors such as GitHub or BioRender as connector-managed MCP-like capability:

1. Verify whether the connector is listed and whether tools are exposed.
2. Test one harmless tool call if available.
3. If discovery or handshake fails, refresh Codex Apps discovery/runtime caches only after backup.
4. If tools are exposed but the backend returns "tool not found" or equivalent, record connector id, link id, local cache versions, tool names, and exact error, then escalate to the provider/OpenAI support.
5. Do not try to fix backend tool registration by editing skill folders, plugin cache payloads, or MCP config by hand.

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
