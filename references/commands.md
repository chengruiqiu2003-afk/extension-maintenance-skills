# Extension Maintenance Commands

These commands assume Windows PowerShell. Adapt paths for macOS/Linux if needed.

## Path Variables

```powershell
$AgentsSkills = Join-Path $env:USERPROFILE '.agents\skills'
$ClaudeSkills = Join-Path $env:USERPROFILE '.claude\skills'
$ClaudePluginCache = Join-Path $env:USERPROFILE '.claude\plugins\cache'
$ClaudeMarketplaces = Join-Path $env:USERPROFILE '.claude\plugins\marketplaces'
$CodexPluginCache = Join-Path $env:USERPROFILE '.codex\plugins\cache'
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
Get-Content -LiteralPath "$env:USERPROFILE\.agents\.skill-lock.json" -Raw
Get-ChildItem -LiteralPath $AgentsSkills -Directory
Get-ChildItem -LiteralPath $ClaudeSkills -Directory
Get-ChildItem -LiteralPath $ClaudePluginCache -Directory
Get-ChildItem -LiteralPath $CodexPluginCache -Directory
```

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

| Side | Type | Name | Version/Hash | Enabled | Path | Source | Updateability | Status |
|---|---|---|---|---|---|---|---|---|

Update report:

| Side | Name | Before | After | Latest | Verification |
|---|---|---|---|---|---|
