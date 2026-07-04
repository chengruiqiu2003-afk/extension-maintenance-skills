# Extension Maintenance

`extension-maintenance` is a workflow skill for safely managing Claude Code and Codex skills, plugins, MCP servers, and Codex Apps/connectors.

It standardizes the maintenance cycle for installed extensions:

1. inventory the current state
2. classify updateability
3. back up affected files
4. update or install through official commands
5. normalize folder placement
6. verify with commands and hashes
7. write an auditable change record

## What It Helps With

- Inventory Claude Code and Codex skills/plugins.
- Inventory Claude Code and Codex MCP servers, including Codex entries imported from Claude Code.
- Classify MCP servers by carrier: plugin-managed, package-managed, remote-managed, or connector-managed.
- Check whether installed items can be updated automatically.
- Update global skills through `npx skills`.
- Install shared skills for both Claude Code and Codex.
- Handle Claude plugin marketplace problems.
- Maintain Codex Apps/connectors such as GitHub or BioRender by checking connection state, local cache state, and harmless tool probes.
- Clean orphaned or superseded plugin caches safely.
- Verify shared skills with `SKILL.md` SHA256 hashes.
- Verify shared/imported MCP entries with `claude mcp list`, `codex mcp list`, config source checks, and non-destructive tool calls where available.
- Keep maintenance notes under a dedicated inventory folder.

## What It Does Not Do

- It does not bypass marketplace or repository metadata.
- It does not guess missing upstream sources.
- It does not treat MCP itself as an updatable package; it updates the plugin, package, container, repository, remote endpoint, or connector that carries the MCP server.
- It does not force-update upstream-managed Codex Apps/connectors.
- It does not delete caches without backup and path verification.
- It does not move plugin cache directories into skill directories.
- It does not read or expose secrets such as tokens, API keys, or credentials.

## Skill Layout

```text
extension-maintenance/
  SKILL.md
  references/
    commands.md
    change-record-template.md
```

This follows the common skill layout used by Codex and Claude Code: a skill directory with a required `SKILL.md` plus optional supporting files.

## Recommended Local Layout

On Windows, this skill assumes these defaults unless the user has a custom layout:

```text
%USERPROFILE%\.agents\skills
%USERPROFILE%\.claude\skills
%USERPROFILE%\.codex\plugins\cache
%USERPROFILE%\.claude\plugins\cache
%USERPROFILE%\.claude\plugins\marketplaces
%USERPROFILE%\.codex\config.toml
%USERPROFILE%\.claude.json
%USERPROFILE%\Documents\AI-Extensions
```

Set `AI_EXTENSIONS_HOME` if you want records and backups somewhere else.

## Installation

Install from GitHub after publishing:

```powershell
cmd /c npx --yes skills add <owner>/<repo> -g -a claude-code codex -s extension-maintenance --copy --full-depth -y
```

Install from a local checkout:

```powershell
cmd /c npx --yes skills add "<path-to-this-directory>" -g -a claude-code codex -s extension-maintenance --copy -y
```

## Example Prompts

```text
Use extension-maintenance to check my Claude Code and Codex skills/plugins.
```

```text
Use extension-maintenance to update auto-checkable skills/plugins and write a maintenance record.
```

```text
Use extension-maintenance to clean orphaned plugin caches, but only after backup and verification.
```

```text
Use extension-maintenance to check Claude and Codex MCP servers, including Codex entries imported from Claude Code.
```

```text
Use extension-maintenance to diagnose my GitHub or BioRender Codex connector without exposing secrets.
```

## Safety Model

The skill is intentionally conservative:

- destructive operations require backup first
- recursive deletes must verify the resolved path is under an allowed root
- manual/local sources are reported as `manual-review` instead of being force-updated
- MCP maintenance updates the carrier rather than editing protocol entries blindly
- connector-managed Apps are diagnosed locally and escalated when the backend mapping is inconsistent
- current replacement paths must exist before old caches are removed

## References

- Codex Agent Skills documentation: https://developers.openai.com/codex/skills
- Codex MCP documentation: https://developers.openai.com/codex/mcp
- Codex plugins documentation: https://developers.openai.com/codex/plugins
- OpenAI MCP and connectors guide: https://developers.openai.com/api/docs/guides/tools-connectors-mcp
- Claude Code skills documentation: https://docs.anthropic.com/en/docs/claude-code/skills
- Claude Code MCP documentation: https://docs.anthropic.com/en/docs/claude-code/mcp

## License

MIT License. See `LICENSE`.
