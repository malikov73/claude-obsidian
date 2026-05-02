# claude-obsidian — Cross-Platform Plugin Design

**Date:** 2026-05-02
**Author:** asman
**Status:** approved (brainstorming phase complete)
**Target platforms:** Claude Code, Codex

## 1. Goal

A shareable plugin that teaches AI agents (Claude Code, Codex) how to read, write, and organize an Obsidian vault conventionally — both for general-purpose PKM and for the specialized brainstorming-of-business-ideas workflow already in use at `/Users/asman/Developer/brainstorm/Бизнес-идеи/`.

The problem it solves: every new agent session re-learns Obsidian conventions from scratch (or invents them ad-hoc), leading to inconsistent file structure, missing frontmatter, lost decision rationale, and duplicated research notes. This plugin distills the conventions once and exposes them via skills agents auto-discover.

## 2. Scope

### In scope
- Universal PKM conventions (vault shape, naming, frontmatter, linking, atomicity, append-vs-rewrite)
- MCP-tools cheatsheet for the `obsidian` MCP server
- Brainstorming specialization (idea portfolios, decision logs, research notes, cross-references)
- Three Claude Code commands (sugar over skills)
- Cross-platform support: Claude Code + Codex from one git repo
- Vault skeleton + per-type templates

### Out of scope
- Obsidian plugins (the JS/TS kind that run in the Obsidian app)
- Note-taking automation beyond what the MCP server exposes
- Migration tools for existing non-conformant vaults
- Other AI-agent platforms (Gemini CLI, Copilot CLI) — addressable later via the same `skills/` directory if those platforms support symlink-style discovery

## 3. Architecture

### 3.1 Two-layer skill design

**Core layer** (universal PKM):
- `obsidian-conventions` — vault shape, naming, frontmatter, linking, atomicity, append-vs-rewrite, search-before-create
- `obsidian-mcp-tools` — reference cheatsheet for `mcp__obsidian__*` tools

**Specialization layer:**
- `obsidian-brainstorm` — extends conventions for idea portfolios, decision logs, research notes, cross-idea references

Specialization skills *extend*, not duplicate, the core. The brainstorm skill cites conventions for shared rules (frontmatter spec, naming, atomicity) and only documents brainstorm-specific patterns (MADR template, idea folder structure, cross-references).

### 3.2 Repo layout

```
claude-obsidian/
├── .claude-plugin/
│   ├── plugin.json                       # Claude Code manifest
│   └── marketplace.json                  # for /plugin marketplace add
├── .codex/
│   └── INSTALL.md                        # Codex symlink install instructions
├── skills/                               # shared between platforms
│   ├── obsidian-conventions/SKILL.md
│   ├── obsidian-mcp-tools/SKILL.md
│   └── obsidian-brainstorm/SKILL.md
├── commands/                             # CC-only sugar
│   ├── obsidian-init.md
│   ├── obsidian-decision.md
│   └── obsidian-research.md
├── templates/                            # cited by skills, not loaded into context
│   ├── vault-skeleton/
│   │   ├── 00 Home.md
│   │   ├── MOCs/_template-moc.md
│   │   ├── Templates/{daily,project,source,adr,note,research}.md
│   │   └── README.md
│   ├── adr-madr.md
│   ├── moc.md
│   └── research-note.md
├── docs/
│   └── design.md                         # this spec, committed
├── README.md
├── LICENSE                               # MIT
└── .gitignore
```

### 3.3 Cross-platform mechanism

- Both platforms read the same `skills/` directory.
- Claude Code discovers via `.claude-plugin/plugin.json` + marketplace.
- Codex discovers via symlink: user clones repo, then `ln -s <repo>/skills ~/.agents/skills/claude-obsidian`.
- Commands (`/obsidian-init`, etc.) are Claude Code-only; Codex agents reach the same outcome by following skill instructions manually.
- MCP tools are universal — both platforms talk to the same Obsidian MCP server with identical tool names (`mcp__obsidian__*`).

## 4. Conventions encoded

These are the load-bearing decisions the plugin embeds. Sourcing in §11.

### 4.1 Vault shape (LYT-flavored hybrid)

```
Vault/
├── 00 Home.md                # entry: top-level MOCs + active projects
├── Inbox/                    # fleeting capture; processed weekly
├── Projects/                 # active work (PARA "P")
│   └── NN <Project>/         # one subfolder per project
├── Areas/                    # ongoing responsibilities
├── Resources/                # reference material
├── Sources/                  # literature notes (one per source)
├── MOCs/                     # topic indices
├── Daily/YYYY/               # daily notes
├── Templates/                # note templates
└── Archive/                  # done / deprecated
```

Rule: **one level of subfolder maximum** inside each top-level. Tags carry topic; folders carry state.

### 4.2 Note types (frontmatter `type:`)

Closed enum: `daily | moc | project | source | note | adr | research | idea | log`. Adding new types requires updating the convention skill.

### 4.3 Minimum frontmatter

```yaml
---
tags: [type/note, project/wedding-speech]
aliases: []
created: 2026-05-02
updated: 2026-05-02
type: note
status: active           # idea | active | done | archived | superseded
---
```

Type-specific extensions documented per type (ADR adds `id`, `deciders`, `supersedes`; source adds `source-url`, `author`, `year`).

Rules:
- Internal wikilinks in YAML **must be quoted**: `related: "[[Note]]"`. Unquoted breaks the YAML parser and Obsidian's graph.
- Dates in `YYYY-MM-DD` (ISO 8601).
- 5–10 fields max.
- Keys lowercase, kebab/snake_case.

### 4.4 File naming

| Context | Pattern | Example |
|---|---|---|
| Default | Descriptive title | `Wedding speech pricing.md` |
| Project child | `NN Title.md` | `05 P&L и экономика.md` |
| Daily | `YYYY-MM-DD.md` | `2026-05-02.md` |
| ADR | `ADR-NNNN-title.md` | `ADR-0012-paystack.md` |
| Source | Descriptive | `Ahrens — Smart Notes.md` |

### 4.5 Linking

- **Wikilinks always** — `[[Note]]`, `[[Note|alias]]`, block refs `[[Note#^id]]`, embeds `![[Note#section]]`.
- Markdown links only when interop with non-Obsidian tools is required.
- Aliases for natural-language linking.
- Hierarchical tags acceptable for `type/`, `status/`, `project/`.

### 4.6 Atomicity

- One idea per permanent note. Title is a complete claim ("Pricing in DE requires inclusive VAT"), not a noun phrase.
- If the title needs "and" / "также" / semicolons → split.
- MOCs and reference docs (P&L, competitor matrix) intentionally aggregate; not subject to atomicity.

### 4.7 Append vs rewrite (load-bearing for agents)

**Append** to existing note when:
- Adding a refinement, correction, or new datapoint to an existing claim
- Content is <150 words and tightly related to the note's subject

Pattern:
```markdown

## UPDATE 2026-05-02: <topic>

<content>
```

**Split into new file** when:
- New idea that can stand alone
- Content >150 words unrelated to the host note
- Will be referenced from elsewhere

**Never delete.** Mark `status: superseded` and link via `superseded-by: "[[ADR-NNNN]]"` or move to `Archive/`.

### 4.8 Search-before-create

Mandatory before creating any new note:
1. `obsidian_simple_search` for the title and aliases
2. Zero matches → create
3. Existing match → either append to it (with UPDATE block) or create new with explicit cross-reference

## 5. Skill 1: `obsidian-conventions`

**Frontmatter:**
```yaml
---
name: obsidian-conventions
description: "Use whenever reading, writing, or organizing notes in an Obsidian vault. Encodes vault shape, file naming, frontmatter spec, linking, atomicity, append-vs-rewrite, and search-before-create rules."
---
```

**Sections:**
1. Vault shape (§4.1)
2. Note types (§4.2)
3. Frontmatter spec (§4.3) + per-type extensions
4. File naming table (§4.4)
5. Linking patterns (§4.5)
6. Atomicity rule (§4.6)
7. Append vs rewrite playbook (§4.7) with concrete templates
8. Search-before-create checklist (§4.8)
9. Folders vs tags rule
10. Platform notes — short paragraph: "On Codex, native file tools replace `Read`/`Write`/`Edit`. MCP `mcp__obsidian__*` tools work identically. See `obsidian-mcp-tools` skill for tool selection."
11. References to `templates/` (with relative paths)

Length target: ~400-500 lines (substantial; this is the load-bearing skill).

## 6. Skill 2: `obsidian-mcp-tools`

**Frontmatter:**
```yaml
---
name: obsidian-mcp-tools
description: "Use when interacting with an Obsidian vault via the obsidian MCP server. Reference cheatsheet for which mcp__obsidian__* tool to call when, common patterns, and gotchas."
---
```

**Sections:**
1. Tool decision matrix (table from design §3)
2. Loading deferred MCP tools (CC: `ToolSearch select:...`; Codex: native MCP loading)
3. Three canonical patterns:
   - Find-before-create
   - Append with UPDATE block
   - Surgical update via `patch_content`
4. JsonLogic snippets for `complex_search` (frontmatter filters, path filters, recent-by-date)
5. Gotchas: unique heading requirement for `patch_content`, YAML wikilink quoting, search scope (content vs filename)
6. Anti-patterns: don't use `delete_file` for retired ideas; don't loop `get_file_contents` (use `batch_get_file_contents`); don't try SQL syntax in `complex_search`

Length target: ~150-200 lines (focused reference).

## 7. Skill 3: `obsidian-brainstorm`

**Frontmatter:**
```yaml
---
name: obsidian-brainstorm
description: "Use when working on a business idea / strategy brainstorm where reasoning, decisions, and research must persist across sessions in Obsidian. Extends obsidian-conventions with idea-portfolio structure, MADR decision logs, and research-note patterns."
---
```

**Sections:**
1. **Why this skill exists** — context loss between sessions, decision drift, repeated research
2. **Idea portfolio shape** (§4 of design):
   ```
   Бизнес-идеи/
   ├── 00 Index.md
   ├── NN <Idea>/
   │   ├── 01 Идея.md
   │   ├── 02 Конкуренты.md
   │   ├── 03 План реализации.md
   │   ├── 04 Трафик и воронка.md
   │   ├── 05 P&L и экономика.md
   │   ├── 06 <topic>.md, 07 <topic>.md, ...    # additional research / topics
   │   └── NN Решения брейншторма.md             # decision log; NN = next free number
   ```
3. **Decision log convention** — MADR template (cite `templates/adr-madr.md`); when to start a log (>3 key decisions); append-only; supersede via `superseded-by`
4. **Research note convention** — inline UPDATE block for <150-word findings; separate `06 <Topic> Research.md` file otherwise; sources in frontmatter `sources: [url1, url2]` or as Source-folder notes
5. **Cross-references between ideas** — `00 Index.md` aliased wikilinks; cross-cutting tags; status changes (don't delete paused ideas)
6. **Workflow heuristics** — read `00 Index.md` first; save findings during research not after; append to decision log after each key decision; update `00 Index.md` at session end
7. **Anti-patterns** — don't save intermediate iterations; don't duplicate code/git data; don't save ephemeral session details
8. **Extends `obsidian-conventions`** — explicit pointer that frontmatter, naming, linking, atomicity rules apply unchanged

Length target: ~250-350 lines.

## 8. Commands (Claude Code only)

### 8.1 `/obsidian-init [vault-path]`

Bootstraps a new vault. Copies `templates/vault-skeleton/` into the target path, creates folder structure, seeds `00 Home.md` and `Templates/`. Prompts for vault name.

### 8.2 `/obsidian-decision <project-path>`

Walks the user through MADR fields (context, drivers, options, decision, consequences, revisit-when), then appends to `<project-path>/NN Решения брейншторма.md` (creates if missing). Auto-numbers ADR IDs.

### 8.3 `/obsidian-research <project-path> <topic>`

Creates a research note `<project-path>/NN <Topic> Research.md` with the structured template (frontmatter `type: research`, sections for findings/sources/synthesis). Also accepts `--inline` flag: appends an UPDATE block to an existing note instead.

**Codex parity:** Each command is functionally a thin wrapper over a skill; Codex agents reach the same outcome by reading the relevant skill and executing its steps. Commands documented as CC-only convenience.

## 9. Templates

Cited by skills, copied by commands. Listed in §3.2.

Per-type templates each include:
- Correct frontmatter for the type
- Section scaffolding
- Inline comments (HTML comment syntax) explaining what goes in each section

## 10. Manifests

### 10.1 `.claude-plugin/plugin.json`
```json
{
  "name": "claude-obsidian",
  "description": "Conventions, MCP-tools cheatsheet, and brainstorming patterns for working with Obsidian vaults across Claude Code and Codex.",
  "version": "0.1.0",
  "author": { "name": "asman" },
  "homepage": "https://github.com/<user>/claude-obsidian",
  "repository": "https://github.com/<user>/claude-obsidian",
  "license": "MIT",
  "keywords": ["obsidian", "pkm", "brainstorming", "knowledge-management", "skills"]
}
```

### 10.2 `.claude-plugin/marketplace.json`
```json
{
  "name": "claude-obsidian-marketplace",
  "description": "Claude Obsidian plugin marketplace",
  "owner": { "name": "asman" },
  "plugins": [
    {
      "name": "claude-obsidian",
      "description": "Obsidian conventions + brainstorming patterns",
      "version": "0.1.0",
      "source": "./",
      "author": { "name": "asman" }
    }
  ]
}
```

### 10.3 `.codex/INSTALL.md`
Step-by-step (modeled on superpowers): clone, `mkdir -p ~/.agents/skills`, symlink `skills/` directory, restart Codex, verify with `ls -la ~/.agents/skills/claude-obsidian`. Mac/Linux + Windows PowerShell variants.

## 11. Sourcing for conventions

The conventions in §4 are not invented — they distill modern Obsidian practice as of 2026:

- **LYT-flavored vault shape** — Nick Milo, "ways to form useful relationships between notes" (Medium)
- **Folders for state, tags for topic** — Obsidian forum canonical thread on links vs tags
- **MADR ADR template** — github.com/adr/madr
- **Atomic notes / claim-phrase titles** — Andy Matuschak, "Evergreen notes should be atomic"
- **Smart Notes pipeline (fleeting → literature → permanent)** — Sönke Ahrens
- **Frontmatter spec** — Obsidian Properties official docs + Dataview metadata-types reference

These citations also appear inline in the skills, so agents can verify if they need to.

## 12. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Skills become stale as Obsidian features evolve (e.g., Properties syntax, Bases) | Versioned plugin; conventions section dated; revisit annually |
| Convention conflicts with user's existing vault | Skills explicitly say "if vault already has different conventions, follow those — these are defaults for new vaults" |
| MADR ADR overkill for simple decisions | Brainstorm skill says: "start a decision log only when >3 key decisions exist; for one-off decisions, inline rationale in the host note is fine" |
| Codex platform changes break symlink discovery | `.codex/INSTALL.md` is small and targeted; can be updated independently |
| Plugin pulls in users with vastly different PKM styles (PARA-purist, Zettelkasten-purist) | Document choices as defaults, not law; `obsidian-conventions` opens with "this plugin assumes a hybrid LYT-flavored vault. If your vault uses pure PARA or pure Zettelkasten, treat the conventions below as adaptable starting points" |

## 13. Success criteria

The plugin is successful if:
1. A fresh agent session in either Claude Code or Codex produces correctly-formatted notes (right frontmatter, naming, linking) without further instruction beyond "use the obsidian-conventions skill."
2. Decisions made in one session are discoverable and respected in a future session via the decision log.
3. Research findings persist as searchable, atomically-titled notes — not lost in chat history.
4. The plugin can be installed in <2 minutes on either platform.
5. Adding a fourth specialization skill (e.g., `obsidian-research-papers`) requires no changes to core skills.

## 14. Out-of-scope (deferred)

- `obsidian-research-papers` — academic literature workflow (Zotero integration, citation styles)
- `obsidian-journal` — personal journaling specialization
- Auto-migration tooling for non-conformant vaults
- Hooks / event-driven automation
- Other agent platforms (Gemini CLI, Copilot CLI)

These can be added later without touching the core layer.

## 15. Open questions

None blocking. Domain-name / GitHub username for the repo will be picked at implementation time.
