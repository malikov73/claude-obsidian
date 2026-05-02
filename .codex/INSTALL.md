# Installing claude-obsidian for Codex

Enable claude-obsidian skills in Codex via native skill discovery — clone and symlink.

## Prerequisites

- Git
- An Obsidian MCP server configured in Codex (skills assume `mcp__obsidian__*` tools are available)

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/malikov73/claude-obsidian.git ~/.codex/claude-obsidian
   ```

2. **Create the skills symlink:**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/claude-obsidian/skills ~/.agents/skills/claude-obsidian
   ```

   **Windows (PowerShell):**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\claude-obsidian" "$env:USERPROFILE\.codex\claude-obsidian\skills"
   ```

3. **Restart Codex** (quit and relaunch the CLI) to discover the skills.

## Verify

```bash
ls -la ~/.agents/skills/claude-obsidian
```

You should see a symlink (or junction on Windows) pointing to your claude-obsidian skills directory.

## What Codex Gets

- `obsidian-conventions` — vault organization, file naming, frontmatter, linking, atomicity rules
- `obsidian-mcp-tools` — cheatsheet for `mcp__obsidian__*` tools
- `obsidian-brainstorm` — idea portfolios, decision logs, research notes (extends conventions)

Commands (`/obsidian-init`, `/obsidian-decision`, `/obsidian-research`) are Claude Code-only — Codex agents perform the equivalent flow by following the skill instructions.

## Updating

```bash
cd ~/.codex/claude-obsidian && git pull
```

Skills update instantly through the symlink.

## Uninstalling

```bash
rm ~/.agents/skills/claude-obsidian
```

Optionally delete the clone: `rm -rf ~/.codex/claude-obsidian`.
