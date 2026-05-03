# claude-obsidian

A cross-platform plugin for **Claude Code** and **Codex** that teaches AI agents how to read, write, and organize **Obsidian** vaults — both for general PKM and for brainstorming portfolios of business ideas.

## What it provides

### Skills (load automatically when relevant)
- **`obsidian-conventions`** — vault shape, file naming, frontmatter, linking, atomicity, append-vs-rewrite, search-before-create
- **`obsidian-mcp-tools`** — cheatsheet for `mcp__obsidian__*` MCP tools, common patterns, gotchas
- **`obsidian-brainstorm`** — idea portfolio structure, MADR decision logs, research notes (extends conventions)
- **`obsidian-diagrams`** — when to use Mermaid vs JSON Canvas vs Excalidraw, vanilla-only templates and patterns (extends conventions)

### Commands (Claude Code only)
- **`/obsidian-init`** — bootstrap a new vault with the conventions baked in
- **`/obsidian-decision`** — add a MADR-format decision log entry
- **`/obsidian-research`** — start a research note (new file or inline UPDATE block)

### Templates
Vault skeleton + per-type note templates (daily, project, source, ADR, note, research) + standalone MOC and ADR templates.

## Installation

### Claude Code
```bash
/plugin marketplace add malikov73/claude-obsidian
/plugin install claude-obsidian@malikov73
```

### Codex
See [`.codex/INSTALL.md`](.codex/INSTALL.md) — clone the repo and symlink `skills/` into `~/.agents/skills/claude-obsidian`.

## Prerequisites

You need the **Obsidian MCP server** configured in your agent. The plugin's tool cheatsheet assumes `mcp__obsidian__*` tools are available. Recommended servers:
- [obsidian-mcp](https://github.com/MarkusPfundstein/mcp-obsidian) (Python, requires Local REST API plugin in Obsidian)

## Project structure

```
claude-obsidian/
├── .claude-plugin/         # CC plugin + marketplace manifests
├── .codex/                 # Codex install instructions
├── skills/                 # the 4 skills (cross-platform)
├── commands/               # 3 CC commands (sugar over skills)
├── templates/              # vault skeleton + per-type templates
└── docs/                   # design + plan documents
```

## Usage

After install, your agent will pick up the right skill automatically based on the task description. To prime explicitly:

> "Use the `obsidian-conventions` skill when writing notes."

For brainstorming work:

> "Use the `obsidian-brainstorm` skill — we're working on idea portfolio at `<path>`."

## Conventions encoded

The plugin assumes a hybrid LYT-flavored vault (PARA-ish folders + Zettelkasten atomicity + MOCs). If your vault uses different conventions, treat the skills as adaptable starting points — they explicitly say "follow your existing vault's conventions if they differ."

See [`docs/design.md`](docs/design.md) for the full rationale.

## License

MIT — see [LICENSE](LICENSE).
