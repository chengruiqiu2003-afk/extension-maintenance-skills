# Extension Maintenance Commands

These commands assume Windows PowerShell. Adapt paths for macOS/Linux if needed.

## Path Variables

```powershell
$AgentsSkills = Join-Path $env:USERPROFILE '.agents\skills'
$ClaudeSkills = Join-Path $env:USERPROFILE '.claude\skills'
$ClaudePluginCache = Join-Path $env:USERPROFILE '.claude\plugins\cache'
$ClaudeMarketplaces = Join-Path $env:USERPROFILE '.claude\plugins\marketplaces'
$CodexPluginCache = Join-Path $env:USERPROFILE '.codex\plugins\cache'
$CodexConfig = Join-Path $env:USERPROFILE '.codex\config.toml'
$ClaudeUserConfig = Join-Path $env:USERPROFILE '.claude.json'
$AiExtensions = if ($env:AI_EXTENSIONS_HOME) {
  $env:AI_EXTENSIONS_HOME
} else {
  Join-Path $env:USERPROFILE 'Documents\AI-Extensions'
}
```

## Inventory

```powershell
cmd /c npx --yes skills list -g --json
cmd /c claude plugin list --json
cmd /c claude plugin marketplace list --json
cmd /c claude mcp list
cmd /c codex mcp list
Get-Content -LiteralPath "$env:USERPROFILE\.agents\.skill-lock.json" -Raw
Get-ChildItem -LiteralPath $AgentsSkills -Directory
Get-ChildItem -LiteralPath $ClaudeSkills -Directory
Get-ChildItem -LiteralPath $ClaudePluginCache -Directory
Get-ChildItem -LiteralPath $CodexPluginCache -Directory
Test-Path -LiteralPath $CodexConfig
Test-Path -LiteralPath $ClaudeUserConfig
Get-ChildItem -LiteralPath (Get-Location) -Filter '.mcp.json' -Force
```

Do not print config files that may contain secrets. When inspection is needed, extract only server names, scopes, commands, URLs, and environment variable names, and redact values.

## Install Shared Skills

Install one skill from a repository:

```powershell
cmd /c npx --yes skills add <owner>/<repo> -g -a claude-code codex -s <skill-name> --copy --full-depth -y
```

Install multiple sub-skills from the same repository:

```powershell
cmd /c npx --yes skills add <owner>/<repo> -g -a claude-code codex -s <skill-a> -s <skill-b> -s <skill-c> --copy --full-depth -y
```

List available skills in a repository without installing:

```powershell
cmd /c npx --yes skills add <owner>/<repo> -l --full-depth
```

## Update Skills

Update all global skills:

```powershell
cmd /c npx --yes skills update -g -y
```

Update selected global skills:

```powershell
cmd /c npx --yes skills update -g <skill-name> -y
```

If a shared skill updates on one side but hashes differ, reinstall from the verified source with both agents:

```powershell
cmd /c npx --yes skills add <owner>/<repo> -g -a claude-code codex -s <skill-name> --copy --full-depth -y
```

## Claude Plugin Commands

```powershell
cmd /c claude plugin list --json
cmd /c claude plugin marketplace list --json
cmd /c claude plugin marketplace add <name> <source>
cmd /c claude plugin marketplace remove <name>
cmd /c claude plugin install <plugin>@<marketplace>
cmd /c claude plugin update <plugin>@<marketplace> -y
cmd /c claude plugin uninstall <plugin>@<marketplace> -s user -y
```

Confirm the exact CLI syntax with `claude plugin --help` before destructive operations.

## MCP Commands

Claude Code MCP inventory and help:

```powershell
cmd /c claude mcp list
cmd /c claude mcp --help
cmd /c claude mcp add --help
cmd /c claude mcp remove --help
```

Claude Code supports local, project, and user scopes. Prefer user/global scope for shared personal servers, project scope for team-shared `.mcp.json`, and local scope for private project-only credentials.

Codex MCP inventory and help:

```powershell
cmd /c codex mcp list
cmd /c codex mcp --help
cmd /c codex mcp add --help
cmd /c codex mcp remove --help
```

Codex CLI and IDE read MCP configuration from `~\.codex\config.toml`. Some Codex entries can be imported from Claude Code configuration; record that source before changing either side.

Common carrier update checks:

```powershell
# npm/npx carrier
cmd /c npm view <package-name> version
cmd /c npm outdated -g

# uv/uvx or Python carrier
cmd /c uv tool list
cmd /c pip list --outdated

# Docker carrier
cmd /c docker image ls
cmd /c docker pull <image>:<tag>

# Git/local executable carrier
git -C '<repo-path>' remote -v
git -C '<repo-path>' status --porcelain
git -C '<repo-path>' fetch --all --prune
```

Use these as diagnostics before mutation. Update only the package, image, lock file, or repository that owns the MCP command.

## Codex Apps Or Connectors

Codex Apps/connectors are connector-managed capabilities that expose tools through MCP-like app tooling. They are not normal skill folders.

Recommended checks:

```powershell
# Generated Codex Apps cache roots often involved in discovery/runtime issues.
$CodexCache = Join-Path $env:USERPROFILE '.codex\cache'
Get-ChildItem -LiteralPath $CodexCache -Directory | Where-Object {
  $_.Name -like 'codex_apps_*' -or $_.Name -eq 'remote_plugin_catalog'
}
Get-ChildItem -LiteralPath $CodexPluginCache -Directory
```

Before clearing generated connector caches, back up the affected directories and verify the target path is under `$env:USERPROFILE\.codex\cache` or `$CodexPluginCache`.

## Hash Verification

```powershell
$skill = '<skill-name>'
$agentsSkill = Join-Path $env:USERPROFILE ".agents\skills\$skill\SKILL.md"
$claudeSkill = Join-Path $env:USERPROFILE ".claude\skills\$skill\SKILL.md"
Get-FileHash -LiteralPath $agentsSkill -Algorithm SHA256
Get-FileHash -LiteralPath $claudeSkill -Algorithm SHA256
```

## Backup

```powershell
$stamp = Get-Date -Format 'yyyyMMdd-HHmmss'
$backup = Join-Path $AiExtensions "backups\$stamp-<task-name>"
New-Item -ItemType Directory -Force -Path $backup | Out-Null
Copy-Item -LiteralPath '<affected-path>' -Destination $backup -Recurse -Force
```

## Safe Recursive Delete

Use this pattern before deleting caches:

```powershell
$target = Resolve-Path -LiteralPath '<target-path>'
$allowedRoot = Resolve-Path -LiteralPath '<allowed-root>'
if (-not $target.Path.StartsWith($allowedRoot.Path, [System.StringComparison]::OrdinalIgnoreCase)) {
  throw "Refusing to delete outside allowed root: $($target.Path)"
}
Remove-Item -LiteralPath $target.Path -Recurse -Force
```

## Marketplace Repository Hygiene

When a Claude marketplace repository has generated local metadata such as `.claude-plugin`, do not delete it if Claude needs it for marketplace discovery. Prefer adding generated metadata to that repository's local `.git/info/exclude`:

```powershell
Add-Content -LiteralPath '<marketplace-repo>\.git\info\exclude' -Value '.claude-plugin/'
git -C '<marketplace-repo>' status --porcelain
```

## Report Tables

Full status report:

| Side | Type | Name | Scope | Version/Hash | Enabled | Path/Config | Source/Carrier | Updateability | Status |
|---|---|---|---|---|---|---|---|---|---|

Update report:

| Side | Type | Name | Before | After | Latest | Verification |
|---|---|---|---|---|---|---|

MCP status report:

| Side | Name | Scope | Transport | Command/URL | Source | Carrier | Imported From | Updateability | Verification |
|---|---|---|---|---|---|---|---|---|---|
