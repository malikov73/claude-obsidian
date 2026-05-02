# claude-obsidian Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a cross-platform Claude Code + Codex plugin (`claude-obsidian`) that teaches AI agents how to read, write, and organize Obsidian vaults — including a brainstorming-of-business-ideas specialization.

**Architecture:** Single git repo with shared `skills/` directory (3 skills: conventions, mcp-tools, brainstorm), CC-specific `commands/` and `.claude-plugin/` manifest, Codex `.codex/INSTALL.md` for symlink-based discovery, `templates/` cited by skills and copied by commands. No code — all content is markdown/JSON.

**Tech Stack:** Markdown + YAML frontmatter for skills/commands/templates; JSON for plugin/marketplace manifests; bash for verification scripts. Git for distribution.

**Spec:** `/Users/asman/Developer/brainstorm/docs/superpowers/specs/2026-05-02-claude-obsidian-plugin-design.md`

**Repo path (target):** `/Users/asman/Developer/claude-obsidian/` (separate repo, parallel to brainstorm workspace)

**Conventions throughout this plan:**
- Replace `<your-github-user>` with the actual GitHub username before pushing remote (asked at Task 17).
- All file paths absolute under repo root unless otherwise noted.
- "Verify" steps use `cat`/`ls`/`jq`/`grep` — no real test framework needed (this is content, not code).

---

## Task 1: Initialize repo + base directory structure

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/.gitignore`
- Create: `/Users/asman/Developer/claude-obsidian/LICENSE`
- Create: directory tree (empty for now)

- [ ] **Step 1: Create directory tree**

```bash
mkdir -p /Users/asman/Developer/claude-obsidian/{.claude-plugin,.codex,skills/obsidian-conventions,skills/obsidian-mcp-tools,skills/obsidian-brainstorm,commands,templates/vault-skeleton/{MOCs,Templates},docs}
cd /Users/asman/Developer/claude-obsidian
git init
```

- [ ] **Step 2: Write `.gitignore`**

```
.DS_Store
*.swp
node_modules/
.env
.env.local
```

- [ ] **Step 3: Write `LICENSE` (MIT)**

```
MIT License

Copyright (c) 2026 asman

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 4: Verify structure**

```bash
cd /Users/asman/Developer/claude-obsidian
find . -type d -not -path './.git*' | sort
```

Expected output (exactly):
```
.
./.claude-plugin
./.codex
./commands
./docs
./skills
./skills/obsidian-brainstorm
./skills/obsidian-conventions
./skills/obsidian-mcp-tools
./templates
./templates/vault-skeleton
./templates/vault-skeleton/MOCs
./templates/vault-skeleton/Templates
```

- [ ] **Step 5: Initial commit**

```bash
cd /Users/asman/Developer/claude-obsidian
git add .gitignore LICENSE
git commit -m "chore: initial commit (license + gitignore)"
```

---

## Task 2: Plugin manifests (Claude Code)

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/.claude-plugin/plugin.json`
- Create: `/Users/asman/Developer/claude-obsidian/.claude-plugin/marketplace.json`

- [ ] **Step 1: Write `plugin.json`**

```json
{
  "name": "claude-obsidian",
  "description": "Conventions, MCP-tools cheatsheet, and brainstorming patterns for working with Obsidian vaults across Claude Code and Codex.",
  "version": "0.1.0",
  "author": { "name": "asman" },
  "homepage": "https://github.com/<your-github-user>/claude-obsidian",
  "repository": "https://github.com/<your-github-user>/claude-obsidian",
  "license": "MIT",
  "keywords": ["obsidian", "pkm", "brainstorming", "knowledge-management", "skills"]
}
```

- [ ] **Step 2: Write `marketplace.json`**

```json
{
  "name": "claude-obsidian-marketplace",
  "description": "claude-obsidian plugin marketplace",
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

- [ ] **Step 3: Verify both JSONs parse**

```bash
cd /Users/asman/Developer/claude-obsidian
jq . .claude-plugin/plugin.json && jq . .claude-plugin/marketplace.json
```

Expected: both pretty-printed JSON outputs, no errors.

- [ ] **Step 4: Commit**

```bash
git add .claude-plugin/
git commit -m "feat: add Claude Code plugin manifests"
```

---

## Task 3: Codex INSTALL.md

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/.codex/INSTALL.md`

- [ ] **Step 1: Write `INSTALL.md`**

```markdown
# Installing claude-obsidian for Codex

Enable claude-obsidian skills in Codex via native skill discovery — clone and symlink.

## Prerequisites

- Git
- An Obsidian MCP server configured in Codex (skills assume `mcp__obsidian__*` tools are available)

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/<your-github-user>/claude-obsidian.git ~/.codex/claude-obsidian
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
```

- [ ] **Step 2: Commit**

```bash
git add .codex/INSTALL.md
git commit -m "feat: add Codex install instructions"
```

---

## Task 4: Vault skeleton — root files

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/templates/vault-skeleton/00 Home.md`
- Create: `/Users/asman/Developer/claude-obsidian/templates/vault-skeleton/README.md`
- Create: `/Users/asman/Developer/claude-obsidian/templates/vault-skeleton/MOCs/_template-moc.md`

- [ ] **Step 1: Write `00 Home.md`**

```markdown
---
tags: [type/moc, home]
aliases: [Home]
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: moc
status: active
---

# Home

> Top-level entry into this vault. Curated index of active MOCs and projects. Replace this preamble with a one-paragraph description of what this vault is for.

## Active projects

<!-- Wikilinks to projects in `Projects/`, with one-line "why it matters" annotations. -->
- [[Projects/NN ExampleProject/01 Idea|ExampleProject]] — short description

## Topic MOCs

<!-- Wikilinks to MOCs in `MOCs/`. -->
- [[MOCs/Example MOC]] — what this MOC indexes

## Areas

<!-- Ongoing responsibilities. -->

## Recent

<!-- Optional: latest updated notes (auto-fillable via Dataview if installed). -->
```

- [ ] **Step 2: Write `README.md`**

```markdown
# Vault skeleton

This is a starter vault layout following the conventions in the `claude-obsidian` plugin.

- `Inbox/` — fleeting captures; process weekly
- `Projects/` — active work; one subfolder per project
- `Areas/` — ongoing responsibilities
- `Resources/` — reference material
- `Sources/` — literature notes (one per source)
- `MOCs/` — topic indices
- `Daily/YYYY/` — daily notes
- `Templates/` — note templates
- `Archive/` — done / deprecated

See the `obsidian-conventions` skill for the full convention.
```

- [ ] **Step 3: Write `MOCs/_template-moc.md`**

```markdown
---
tags: [type/moc, topic/REPLACEME]
aliases: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: moc
status: active
---

# {{title}} MOC

> One-paragraph description of what this MOC indexes and the current state of the topic.

## Core notes
- [[NoteA]] — why it matters
- [[NoteB]] — why it matters

## Themes

### Theme 1
- [[Note]] — one-line annotation

### Theme 2
- [[Note]] — one-line annotation

## Open questions
- Question?
- Question?

## Decisions
<!-- Optional Dataview block:
```dataview
TABLE status, date FROM "Projects/{{folder}}" WHERE type = "adr" SORT date DESC
```
-->

## Related MOCs
- [[Other MOC]]
```

- [ ] **Step 4: Create the empty subfolders with `.gitkeep`**

```bash
cd /Users/asman/Developer/claude-obsidian/templates/vault-skeleton
for d in Inbox Projects Areas Resources Sources Daily Archive; do
  mkdir -p "$d"
  touch "$d/.gitkeep"
done
```

- [ ] **Step 5: Verify structure**

```bash
cd /Users/asman/Developer/claude-obsidian/templates/vault-skeleton
find . -type f | sort
```

Expected:
```
./00 Home.md
./Archive/.gitkeep
./Areas/.gitkeep
./Daily/.gitkeep
./Inbox/.gitkeep
./MOCs/_template-moc.md
./Projects/.gitkeep
./README.md
./Resources/.gitkeep
./Sources/.gitkeep
```

- [ ] **Step 6: Commit**

```bash
cd /Users/asman/Developer/claude-obsidian
git add templates/vault-skeleton/
git commit -m "feat(templates): add vault skeleton with home + MOC template"
```

---

## Task 5: Per-type templates inside vault skeleton

**Files (all in `/Users/asman/Developer/claude-obsidian/templates/vault-skeleton/Templates/`):**
- Create: `daily.md`, `project.md`, `source.md`, `adr.md`, `note.md`, `research.md`

- [ ] **Step 1: Write `Templates/daily.md`**

```markdown
---
tags: [type/daily]
created: {{date:YYYY-MM-DD}}
type: daily
date: {{date:YYYY-MM-DD}}
---

# {{date:YYYY-MM-DD}}

## Focus (1–3)
-

## Log
- HH:MM —

## Capture
- Idea: …  → [[…]]

## Review
- What went well:
- For tomorrow:
```

- [ ] **Step 2: Write `Templates/project.md`**

```markdown
---
tags: [type/project, project/REPLACEME]
aliases: []
created: {{date:YYYY-MM-DD}}
updated: {{date:YYYY-MM-DD}}
type: project
status: active
deadline:
---

# {{title}}

## Goal

## Scope

### In scope
-

### Out of scope
-

## Milestones
- [ ] Milestone 1
- [ ] Milestone 2

## Notes & decisions
- [[NN Decisions {{title}}]]

## Related
-
```

- [ ] **Step 3: Write `Templates/source.md`**

```markdown
---
tags: [type/source, source/REPLACEME]
aliases: []
created: {{date:YYYY-MM-DD}}
updated: {{date:YYYY-MM-DD}}
type: source
source-url:
author:
year:
status: to-read     # to-read | reading | read
---

# {{title}}

> One-sentence summary of the source in your own words.

## Key claims
- Claim 1 (page/timestamp)
- Claim 2

## Quotes
> "Selective quote with attribution."

## My notes
<!-- Permanent notes derived from this source go in `Notes/` and link back here. -->
```

- [ ] **Step 4: Write `Templates/adr.md`**

```markdown
---
tags: [type/adr, project/REPLACEME]
aliases: []
created: {{date:YYYY-MM-DD}}
updated: {{date:YYYY-MM-DD}}
type: adr
id: ADR-NNNN
status: proposed       # proposed | accepted | deprecated | superseded-by
date: {{date:YYYY-MM-DD}}
deciders: [self]
supersedes:
superseded-by:
---

# ADR-NNNN: {{title}}

## Context
<!-- 2–3 sentences. Why now? Constraints, data, deadlines. -->

## Decision Drivers
- Driver 1
- Driver 2

## Considered Options
- Option A
- Option B

## Decision
Chosen: **Option X**, because …

## Consequences
- Positive:
- Negative:
- Risks:

## Revisit when
- Trigger 1
- Trigger 2

## Related
-
```

- [ ] **Step 5: Write `Templates/note.md`**

```markdown
---
tags: [type/note]
aliases: []
created: {{date:YYYY-MM-DD}}
updated: {{date:YYYY-MM-DD}}
type: note
status: active
---

# {{title}}

<!-- One self-contained idea, expressed completely. Title should be a complete claim, not a noun phrase. -->
```

- [ ] **Step 6: Write `Templates/research.md`**

```markdown
---
tags: [type/research, project/REPLACEME]
aliases: []
created: {{date:YYYY-MM-DD}}
updated: {{date:YYYY-MM-DD}}
type: research
status: active
sources: []
---

# {{title}} — Research

## Question
<!-- What are you trying to learn? -->

## Findings
<!-- Bullet points. Each links to a source note or has an inline URL. -->
-

## Sources
- [[Sources/...]] — how it informed findings
- https://… — direct URL if no Source note yet

## Synthesis
<!-- Your distilled answer to the Question. -->
```

- [ ] **Step 7: Verify all 6 templates exist**

```bash
ls /Users/asman/Developer/claude-obsidian/templates/vault-skeleton/Templates/
```

Expected: `adr.md  daily.md  note.md  project.md  research.md  source.md`

- [ ] **Step 8: Commit**

```bash
cd /Users/asman/Developer/claude-obsidian
git add templates/vault-skeleton/Templates/
git commit -m "feat(templates): add per-type note templates"
```

---

## Task 6: Top-level templates (cited by skills, copied by commands)

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/templates/adr-madr.md`
- Create: `/Users/asman/Developer/claude-obsidian/templates/moc.md`
- Create: `/Users/asman/Developer/claude-obsidian/templates/research-note.md`

- [ ] **Step 1: Write `templates/adr-madr.md`**

Same content as `Templates/adr.md` from Task 5, Step 4 — duplicate it. (Skills cite this top-level path; commands copy from here. Keeping a copy at top-level makes paths in skill prose stable regardless of vault layout.)

- [ ] **Step 2: Write `templates/moc.md`**

Same content as `vault-skeleton/MOCs/_template-moc.md` from Task 4, Step 3.

- [ ] **Step 3: Write `templates/research-note.md`**

Same content as `Templates/research.md` from Task 5, Step 6.

- [ ] **Step 4: Verify**

```bash
ls /Users/asman/Developer/claude-obsidian/templates/
```

Expected: `adr-madr.md  moc.md  research-note.md  vault-skeleton`

- [ ] **Step 5: Commit**

```bash
cd /Users/asman/Developer/claude-obsidian
git add templates/adr-madr.md templates/moc.md templates/research-note.md
git commit -m "feat(templates): add top-level ADR/MOC/research templates"
```

---

## Task 7: Skill — `obsidian-conventions/SKILL.md`

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/skills/obsidian-conventions/SKILL.md`

This is the load-bearing skill. Length target ~400-500 lines.

- [ ] **Step 1: Write SKILL.md**

Frontmatter (verbatim):

```yaml
---
name: obsidian-conventions
description: "Use whenever reading, writing, or organizing notes in an Obsidian vault. Encodes vault shape, file naming, frontmatter spec, linking, atomicity, append-vs-rewrite, and search-before-create rules."
---
```

Body — required sections, in this order. Each section must include the specified content verbatim or a faithful prose expansion of it. Cite sources from the spec §11 inline.

**1. `# Obsidian Conventions`** (H1) — one-paragraph "what this skill does + assumes hybrid LYT-flavored vault; if your vault uses different conventions, follow those."

**2. `## Vault shape`** — verbatim folder tree from spec §4.1. Rule: "one level of subfolder maximum inside each top-level. Tags carry topic; folders carry state."

**3. `## Note types (closed enum)`** — bullet list of `daily | moc | project | source | note | adr | research | idea | log`. Sentence per type explaining when to use it.

**4. `## Minimum frontmatter`** — verbatim YAML block from spec §4.3. Sub-section "Per-type extensions" with YAML samples for ADR (`id`, `deciders`, `supersedes`, `superseded-by`), source (`source-url`, `author`, `year`, `status`), project (`deadline`), research (`sources`).

Rules block (bullets):
- Internal wikilinks in YAML must be quoted: `related: "[[Note]]"`. Unquoted breaks the YAML parser.
- Dates in `YYYY-MM-DD` (ISO 8601).
- Keep frontmatter 5–10 fields max.
- Keys lowercase, kebab/snake_case.
- Use `aliases` for natural-language linking targets.

**5. `## File naming`** — verbatim table from spec §4.4. Plus a paragraph explaining when to use each pattern.

**6. `## Linking`** — wikilinks always; markdown links only for non-Obsidian interop. Examples:
```markdown
[[Note]]
[[Note|alias]]
[[Note#Section]]
[[Note#^block-id]]
![[Note#Section]]   <!-- embed -->
```
Sub-section "Hierarchical tags" — examples `type/note`, `status/active`, `project/wedding-speech`.

**7. `## Folders vs tags`** — rule from spec §4.5: "Folder = state/lifecycle (one per note). Tags = topic/type/status (many per note)." Anti-example: don't use `Wedding/`, `Pricing/`, `Marketing/` folders — those are topics → tags.

**8. `## Atomicity`** — verbatim rule from spec §4.6. Examples of compliant vs non-compliant titles.

**9. `## Append vs rewrite`** — full playbook from spec §4.7. Include the UPDATE block template:

```markdown

## UPDATE 2026-05-02: <topic>

<content>
```

Decision flow:
1. Adding refinement / correction / new datapoint to existing claim, AND <150 words → append with UPDATE block
2. New idea, can stand alone, OR >150 words unrelated → split into new file, link from original
3. Never delete. Mark `status: superseded` and add `superseded-by: "[[<new-note>]]"`.

**10. `## Search-before-create`** — checklist from spec §4.8. Concrete steps:
1. Run `obsidian_simple_search` with the prospective title
2. Run `obsidian_simple_search` with each alias
3. Zero matches → create new note
4. Existing matches → read each, decide append-or-new with explicit cross-reference

**11. `## Templates`** — point at the templates directory:
- Vault skeleton: `templates/vault-skeleton/`
- Per-type: `templates/vault-skeleton/Templates/{daily,project,source,adr,note,research}.md`
- Standalone: `templates/{adr-madr,moc,research-note}.md`

Note: "Templates use `{{title}}`, `{{date:YYYY-MM-DD}}` placeholders compatible with Obsidian's Templates core plugin."

**12. `## Platform notes`** — short paragraph: "On Codex, native file tools replace `Read`/`Write`/`Edit`. MCP `mcp__obsidian__*` tools work identically across platforms. See the `obsidian-mcp-tools` skill for tool selection. Commands like `/obsidian-init` are Claude Code-only — Codex agents follow the equivalent steps manually using this skill."

**13. `## Sources`** — short citation list:
- Nick Milo (LYT framework + MOCs) — https://medium.com/@nickmilo22/in-what-ways-can-we-form-useful-relationships-between-notes-9b9ec46973c6
- Andy Matuschak (atomic notes) — https://notes.andymatuschak.org/Evergreen_notes_should_be_atomic
- Sönke Ahrens (Smart Notes pipeline) — https://www.zettelkasten.de/posts/concepts-sohnke-ahrens-explained/
- Obsidian Properties docs — https://help.obsidian.md/properties
- MADR ADR template — https://github.com/adr/madr

- [ ] **Step 2: Verify SKILL.md is valid**

```bash
cd /Users/asman/Developer/claude-obsidian
head -5 skills/obsidian-conventions/SKILL.md   # frontmatter visible
wc -l skills/obsidian-conventions/SKILL.md     # expect 300+ lines
grep -c "^## " skills/obsidian-conventions/SKILL.md  # expect 12+ section headers
```

- [ ] **Step 3: Commit**

```bash
git add skills/obsidian-conventions/
git commit -m "feat(skills): add obsidian-conventions core skill"
```

---

## Task 8: Skill — `obsidian-mcp-tools/SKILL.md`

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/skills/obsidian-mcp-tools/SKILL.md`

Length target ~150-200 lines.

- [ ] **Step 1: Write SKILL.md**

Frontmatter (verbatim):

```yaml
---
name: obsidian-mcp-tools
description: "Use when interacting with an Obsidian vault via the obsidian MCP server. Reference cheatsheet for which mcp__obsidian__* tool to call when, common patterns, JsonLogic snippets, and gotchas."
---
```

Body — required sections:

**1. `# Obsidian MCP Tools`** — short intro: "Decision matrix and patterns for the `mcp__obsidian__*` tool family. Pair with `obsidian-conventions` for content rules."

**2. `## Tool decision matrix`** — verbatim table:

| Want to… | Tool | Notes |
|---|---|---|
| Find file by topic | `mcp__obsidian__obsidian_simple_search` | Default. Returns context snippets. Searches content + filename. |
| Search by frontmatter / regex / paths | `mcp__obsidian__obsidian_complex_search` | JsonLogic queries. More expensive. |
| Read one file | `mcp__obsidian__obsidian_get_file_contents` | Path relative to vault root. |
| Read several files at once | `mcp__obsidian__obsidian_batch_get_file_contents` | When pulling ≥3 files. Saves tokens vs N separate calls. |
| Create file or append to existing | `mcp__obsidian__obsidian_append_content` | **Default for adding content.** Creates file if missing. |
| Insert/replace under heading or block | `mcp__obsidian__obsidian_patch_content` | Surgical updates. Requires unique heading or block-ref. |
| Delete file | `mcp__obsidian__obsidian_delete_file` | Avoid. Prefer `status: archived`. |
| Recent activity | `mcp__obsidian__obsidian_get_recent_changes` | Daily-context awareness. |
| Periodic notes | `mcp__obsidian__obsidian_get_periodic_note`, `mcp__obsidian__obsidian_get_recent_periodic_notes` | Daily/weekly by date. |
| Browse | `mcp__obsidian__obsidian_list_files_in_dir`, `mcp__obsidian__obsidian_list_files_in_vault` | When exact path unknown. |

**3. `## Loading deferred tools`** — note: "On Claude Code, all `mcp__obsidian__*` tools are deferred. Load needed tools in one batch via `ToolSearch` with `select:` syntax. Example:

```
ToolSearch(query=\"select:mcp__obsidian__obsidian_simple_search,mcp__obsidian__obsidian_get_file_contents,mcp__obsidian__obsidian_append_content\")
```

On Codex, MCP tools load through the platform's native MCP integration — refer to your platform's docs."

**4. `## Three canonical patterns`**

**Pattern A: Find-before-create**
1. `obsidian_simple_search(query="<prospective title>")` → check matches
2. If 0 matches: `obsidian_append_content(filepath="<path>", content="...")` (creates file)
3. If matches: `obsidian_get_file_contents(filepath="<match>")` → decide append or new with cross-reference

**Pattern B: Append with UPDATE block**
```python
obsidian_append_content(
    filepath="07 Wedding Speech/01 Идея.md",
    content="\n\n## UPDATE 2026-05-02: Pricing revised\n\nDecision changed from $19 → $24 because [reason]."
)
```

**Pattern C: Surgical update with patch**
```python
obsidian_patch_content(
    filepath="07 Wedding Speech/05 P&L.md",
    operation="replace",
    targetType="heading",
    target="Year 2 ARR target",
    content="$80–200k (revised 2026-05-02)"
)
```

**5. `## JsonLogic for complex_search`** — three reusable snippets:

```json
// All ADRs in a project folder, sorted by date
{
  "and": [
    { "glob": ["07 Wedding Speech/**/*.md", { "var": "path" }] },
    { "==": [{ "var": "frontmatter.type" }, "adr"] }
  ]
}

// Notes updated in last 7 days (regex on path won't work — use `recent_changes` tool instead)
// Better: use obsidian_get_recent_changes for time-based queries.

// Notes with specific tag
{
  "in": ["project/wedding-speech", { "var": "frontmatter.tags" }]
}
```

**6. `## Gotchas`**

- `obsidian_patch_content` requires a **unique** heading in the file. If "Pricing" appears twice, the patch fails. Use a more specific heading or fall back to `append_content`.
- `obsidian_simple_search` searches **content**, not just filename. To check existence by name only, use `list_files_in_dir`.
- Wikilinks in YAML frontmatter **must be quoted** (`related: "[[Note]]"`) — unquoted breaks YAML parsing and the patch tool.
- `obsidian_append_content` always appends to file end. To insert mid-file, use `patch_content` with `prepend`/`append` operation under a target heading.
- Paths are vault-relative. No leading `/`.

**7. `## Anti-patterns`**

- ❌ Looping `get_file_contents` for many files → use `batch_get_file_contents`.
- ❌ Using `delete_file` to retire a note → set `status: archived` and move to `Archive/` or just leave with status flag.
- ❌ SQL syntax in `complex_search` → JsonLogic only.
- ❌ Skipping search-before-create → leads to duplicate notes with conflicting state.

- [ ] **Step 2: Verify**

```bash
cd /Users/asman/Developer/claude-obsidian
head -5 skills/obsidian-mcp-tools/SKILL.md
grep -c "^## " skills/obsidian-mcp-tools/SKILL.md   # expect 7+
```

- [ ] **Step 3: Commit**

```bash
git add skills/obsidian-mcp-tools/
git commit -m "feat(skills): add obsidian-mcp-tools cheatsheet skill"
```

---

## Task 9: Skill — `obsidian-brainstorm/SKILL.md`

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/skills/obsidian-brainstorm/SKILL.md`

Length target ~250-350 lines.

- [ ] **Step 1: Write SKILL.md**

Frontmatter (verbatim):

```yaml
---
name: obsidian-brainstorm
description: "Use when working on a business idea / strategy brainstorm where reasoning, decisions, and research must persist across sessions in Obsidian. Extends obsidian-conventions with idea-portfolio structure, MADR decision logs, and research-note patterns."
---
```

Body — required sections:

**1. `# Obsidian Brainstorming Specialization`** — opening: "Use this skill alongside `obsidian-conventions` (which carries the universal rules — frontmatter, naming, linking, atomicity, append-vs-rewrite). This skill adds patterns specific to brainstorming portfolios of business ideas."

**2. `## Why this skill exists`** — three-point rationale:
- Context loss between sessions: chat history is truncated; reasoning evaporates.
- Decision drift: without a log, decisions get re-litigated arbitrarily.
- Repeated research: web research is expensive in time and tokens; persisting it lets sessions build on prior work.

**3. `## Idea portfolio shape`** — verbatim folder tree from spec §7.2:

```
<vault root or subfolder>/
├── 00 Index.md                      # cross-idea index + status snapshot
├── NN <Idea Name>/
│   ├── 01 Идея.md                   # concept + value prop + ICP
│   ├── 02 Конкуренты.md             # landscape per market/segment
│   ├── 03 План реализации.md        # stack, build phases
│   ├── 04 Трафик и воронка.md       # acquisition, SEO
│   ├── 05 P&L и экономика.md        # unit economics, scenarios
│   ├── 06 <topic>.md, 07 <topic>.md, ...    # additional research / topics
│   └── NN Решения брейншторма.md    # decision log; NN = next free number
```

Note: section/file titles can be in any language — examples use Russian because that's the convention in the asman vault. English equivalent works equally well.

**4. `## 00 Index.md structure`** — required template:

```markdown
---
tags: [type/moc, brainstorm/index]
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: moc
status: active
---

# Brainstorm Idea Portfolio

> Cross-idea snapshot. Updated at end of each brainstorm session.

## Active ideas
| # | Idea | Status | Last update | Next milestone |
|---|------|--------|-------------|----------------|
| 07 | [[07 Wedding Speech Generator/01 Идея\|Wedding Speech]] | brainstorm | 2026-05-02 | domain + brand naming |

## Paused / on hold
- [[NN Idea/01 Идея|Idea]] — why paused, when to revisit

## Archived
- [[NN Idea/01 Идея|Idea]] — why killed

## Cross-cutting themes
- Multi-language SEO: [[07 Wedding Speech]], [[NN Other]]
- AI-quality moats: [[07 Wedding Speech]], [[NN Other]]
```

**5. `## Decision log convention (MADR)`**
- Start a decision log only when **>3 key decisions** exist for an idea. Below that, inline rationale in `01 Идея.md` is sufficient.
- File: `<NN Idea>/NN Решения <topic>.md`. NN = next free number after research files.
- Each entry follows MADR (template at `templates/adr-madr.md`). Required fields: id, status, context, drivers, options, decision, consequences, revisit-when, related.
- Append-only. Old decisions never deleted; mark `superseded-by: "[[ADR-NNNN]]"`.
- Cross-reference from other files via wikilink + block ref: `[[NN Решения брейншторма#ADR-0012]]`.
- ID scheme: `ADR-NNNN` (4-digit, sequential, never reused).

Inline example (verbatim):

```markdown
## ADR-0017: Use OpenRouter as LLM gateway

- **Status:** accepted
- **Date:** 2026-05-01
- **Deciders:** self

### Context
Need access to Claude Opus 4.7 + fallback options without managing direct Anthropic billing during MVP.

### Considered options
- Direct Anthropic SDK
- OpenRouter
- Multi-provider abstraction (LangChain)

### Decision
**OpenRouter**, because: 5.5% credit-fee is noise at MVP scale, gives free fallback to Sonnet/GPT-4 on failures, consolidated billing.

### Consequences
- Positive: instant multi-model A/B testing; simple API.
- Negative: extra 5.5% becomes meaningful at $5k+ MRR.

### Revisit when
- MRR > $5k → switch to direct Anthropic.
- OpenRouter SLA degrades.

### Related
- [[03 План реализации#LLM tier]]
- [[05 P&L и экономика#COGS]]
```

**6. `## Research note convention`**
- **Inline UPDATE** (preferred for small findings): single finding <150 words → append `## UPDATE YYYY-MM-DD: <finding>` block to the relevant existing file.
- **Separate file** (when finding doesn't fit): topic >150 words OR multi-source synthesis → create `<NN Idea>/NN <Topic> Research.md` using `templates/research-note.md`.
- Sources captured in frontmatter `sources: [url1, url2]` for inline notes; via Source-folder note (one per source) for permanent records.
- Always link from the parent idea note: e.g., `01 Идея.md` references `06 Multi-language Research.md`.

**7. `## Cross-references between ideas`**
- In `00 Index.md`: every idea is a wikilink with alias (`[[NN Idea/01 Идея|Idea Name]]`).
- For shared themes (tech stack, market, target segment): tag both ideas with `cross-cutting/<theme>` and add a "Related ideas" section.
- When pausing/killing an idea: **never delete the folder**. Update `01 Идея.md` frontmatter `status:` to `paused` or `archived`. Add a section `## Why paused: YYYY-MM-DD` with reasoning. Update `00 Index.md` accordingly.

**8. `## Workflow heuristics`** — what to do during a brainstorm session:

1. **Session start:** read `00 Index.md` first. Then read the relevant idea's files (typically `01 Идея.md` + recently updated files).
2. **During research:** save findings *immediately* to notes — not at session end. Even partial findings are valuable.
3. **After each key decision:** append to the decision log with full MADR fields. Don't postpone — context is freshest now.
4. **Session end:** update `00 Index.md` if scope/priority changed. Update `updated:` frontmatter on touched files.

**9. `## Anti-patterns (do NOT save)`**
- ❌ Intermediate brainstorm iterations → only the final decision/finding belongs in notes.
- ❌ Information already in code or git history.
- ❌ Ephemeral session details ("just chatted about X").
- ❌ Long quotes from research without your own synthesis.
- ❌ Creating new idea folders for variations of an existing idea — append to existing.

**10. `## Extends obsidian-conventions`** — explicit pointer:

> All frontmatter, naming, linking, atomicity, and append-vs-rewrite rules from `obsidian-conventions` apply unchanged. This skill only adds the brainstorm-specific layer on top.

- [ ] **Step 2: Verify**

```bash
cd /Users/asman/Developer/claude-obsidian
head -5 skills/obsidian-brainstorm/SKILL.md
wc -l skills/obsidian-brainstorm/SKILL.md       # expect 200+
grep -c "^## " skills/obsidian-brainstorm/SKILL.md  # expect 10+
```

- [ ] **Step 3: Commit**

```bash
git add skills/obsidian-brainstorm/
git commit -m "feat(skills): add obsidian-brainstorm specialization skill"
```

---

## Task 10: Command — `/obsidian-init`

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/commands/obsidian-init.md`

- [ ] **Step 1: Write `obsidian-init.md`**

```markdown
---
description: "Bootstrap a new Obsidian vault with the claude-obsidian conventions: folder structure, 00 Home.md, MOC + per-type templates."
---

# /obsidian-init

You are bootstrapping a new Obsidian vault using the `claude-obsidian` plugin's vault skeleton.

## Steps

1. **Determine target path.** If the user passed a path argument, use it. Otherwise, ask: "What's the absolute path where the vault should live?"

2. **Confirm with the user.** Show: "I'll create the vault skeleton at `<path>`. This includes folders (Inbox, Projects, Areas, Resources, Sources, MOCs, Daily, Templates, Archive), a `00 Home.md`, a MOC template, and 6 per-type templates. Proceed?"

3. **Locate plugin templates.** The vault skeleton is at:
   - Claude Code: `~/.claude/plugins/cache/<marketplace>/claude-obsidian/<version>/templates/vault-skeleton/`
   - If installed via Codex symlink: `~/.codex/claude-obsidian/templates/vault-skeleton/`
   - Use `Bash` to `find` the path if uncertain.

4. **Copy skeleton to target.** Use `cp -r <skeleton-path>/. <target-path>/` (note the trailing `/.` to copy contents, not the skeleton folder itself).

5. **Replace placeholders in `00 Home.md`.** Substitute `YYYY-MM-DD` with today's date. If the user gave a vault name, replace the example project link with theirs or leave the example.

6. **Verify.** Run `ls -la <target-path>` and confirm structure. Report to user: "Vault initialized at `<path>`. Open in Obsidian: File → Open Vault → choose this directory. Then read the obsidian-conventions skill if you're an agent and need to start writing notes."

## Notes

- This command is Claude Code-only. Codex agents perform the equivalent flow manually by copying `templates/vault-skeleton/` from the plugin install location.
- Do NOT modify an existing non-empty directory without explicit user confirmation. If the target has files, ask: "Target has existing files. Overwrite, merge non-conflicting, or abort?"
```

- [ ] **Step 2: Commit**

```bash
cd /Users/asman/Developer/claude-obsidian
git add commands/obsidian-init.md
git commit -m "feat(commands): add /obsidian-init"
```

---

## Task 11: Command — `/obsidian-decision`

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/commands/obsidian-decision.md`

- [ ] **Step 1: Write `obsidian-decision.md`**

```markdown
---
description: "Add a MADR-format decision log entry to a project's decision log file. Auto-numbers ADR ID. Creates the log file if missing."
---

# /obsidian-decision

You are recording a strategic decision in MADR format for a brainstorm project.

## Steps

1. **Determine project folder.** If the user passed a path argument (e.g., `/obsidian-decision Бизнес-идеи/07 Wedding Speech`), use it. Otherwise, ask: "Which project folder? (relative to vault root)"

2. **Locate or create the decision log file.**
   - Search for `*Решения*.md` or `*Decisions*.md` inside the project folder via `mcp__obsidian__obsidian_list_files_in_dir`.
   - If found: read it via `mcp__obsidian__obsidian_get_file_contents` to determine the next ADR ID.
   - If not found: ask the user for a filename (suggest `NN Решения <project-name>.md` where NN is the next free number after existing files).

3. **Determine next ADR ID.** Scan existing entries for `ADR-NNNN` patterns. Next = highest + 1. If none exist, start at `ADR-0001`.

4. **Walk the user through MADR fields, one at a time.** Wait for response before next field:
   - Title (short, action-oriented)
   - Context (2–3 sentences: why now, constraints)
   - Decision drivers (bulleted list)
   - Considered options (bulleted list, ≥2)
   - Decision (which option + 1-sentence rationale)
   - Consequences (positive / negative / risks)
   - Revisit when (triggers list)
   - Related (wikilinks to relevant files)

5. **Render the MADR entry.** Format following `templates/adr-madr.md`. Use `mcp__obsidian__obsidian_append_content` to append to the log file. New entry is preceded by a horizontal rule (`---`) if the file already has entries.

6. **Confirm.** Report: "ADR-NNNN added to `<filepath>`. Run `obsidian-decision` again for the next decision."

## Notes

- Append-only. Never modify or delete previous entries. To supersede, set the new entry's `Related` field to `Supersedes: [[#ADR-NNNN]]` and update the old entry's status separately (manual edit).
- This command is Claude Code-only. Codex agents follow the same flow manually using the `obsidian-brainstorm` skill's MADR template.
```

- [ ] **Step 2: Commit**

```bash
cd /Users/asman/Developer/claude-obsidian
git add commands/obsidian-decision.md
git commit -m "feat(commands): add /obsidian-decision"
```

---

## Task 12: Command — `/obsidian-research`

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/commands/obsidian-research.md`

- [ ] **Step 1: Write `obsidian-research.md`**

```markdown
---
description: "Start a research note for a project. Default: creates a new file using the research-note template. With --inline flag: appends an UPDATE block to an existing note instead."
---

# /obsidian-research

You are creating or extending a research note for a brainstorm project.

## Steps

1. **Parse arguments.**
   - First positional: project folder path (e.g., `Бизнес-идеи/07 Wedding Speech`)
   - Second positional: research topic (e.g., "Multi-language SEO competitors")
   - Optional: `--inline` flag

2. **Decide mode.**
   - **New file mode (default):** create a separate research note. Use when the topic is multi-source, expected output >150 words, or will be referenced from multiple places.
   - **Inline mode (`--inline`):** append an UPDATE block to an existing file. Ask the user which file. Use when the finding is small (<150 words) and tightly related to one existing note.

3. **For new file mode:**
   a. Determine next free file number in the project folder.
   b. Filename: `NN <Topic> Research.md` (e.g., `08 Multi-language SEO Research.md`).
   c. Read `templates/research-note.md`.
   d. Replace placeholders: `{{title}}` → topic, `{{date:YYYY-MM-DD}}` → today's date, `project/REPLACEME` → actual project tag.
   e. Use `mcp__obsidian__obsidian_append_content` to write the file (creates new).
   f. Add a wikilink from the project's `01 Идея.md` (or main idea file) under a "Related" or "Research" section. Use `mcp__obsidian__obsidian_patch_content` if a heading exists; otherwise `obsidian_append_content` to add a new "Research" section.

4. **For inline mode:**
   a. Confirm target file with user.
   b. Build UPDATE block:
      ```markdown

      ## UPDATE YYYY-MM-DD: <topic>

      <synthesized finding>

      Sources:
      - <url or wikilink>
      ```
   c. Use `mcp__obsidian__obsidian_append_content` to append.

5. **Confirm.** Report: "Research note `<filepath>` created. Findings will go here. Sources captured in frontmatter `sources:` field." (Or for inline: "UPDATE block added to `<filepath>`.")

## Notes

- This command is Claude Code-only. The equivalent Codex flow: read the research-note template, fill placeholders, write via MCP append. See the `obsidian-brainstorm` skill's "Research note convention" section.
```

- [ ] **Step 2: Commit**

```bash
cd /Users/asman/Developer/claude-obsidian
git add commands/obsidian-research.md
git commit -m "feat(commands): add /obsidian-research"
```

---

## Task 13: Top-level README.md

**Files:**
- Create: `/Users/asman/Developer/claude-obsidian/README.md`

- [ ] **Step 1: Write README.md**

```markdown
# claude-obsidian

A cross-platform plugin for **Claude Code** and **Codex** that teaches AI agents how to read, write, and organize **Obsidian** vaults — both for general PKM and for brainstorming portfolios of business ideas.

## What it provides

### Skills (load automatically when relevant)
- **`obsidian-conventions`** — vault shape, file naming, frontmatter, linking, atomicity, append-vs-rewrite, search-before-create
- **`obsidian-mcp-tools`** — cheatsheet for `mcp__obsidian__*` MCP tools, common patterns, gotchas
- **`obsidian-brainstorm`** — idea portfolio structure, MADR decision logs, research notes (extends conventions)

### Commands (Claude Code only)
- **`/obsidian-init`** — bootstrap a new vault with the conventions baked in
- **`/obsidian-decision`** — add a MADR-format decision log entry
- **`/obsidian-research`** — start a research note (new file or inline UPDATE block)

### Templates
Vault skeleton + per-type note templates (daily, project, source, ADR, note, research) + standalone MOC and ADR templates.

## Installation

### Claude Code
```bash
/plugin marketplace add <your-github-user>/claude-obsidian
/plugin install claude-obsidian@<your-github-user>
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
├── skills/                 # the 3 skills (cross-platform)
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
```

- [ ] **Step 2: Commit**

```bash
cd /Users/asman/Developer/claude-obsidian
git add README.md
git commit -m "docs: add top-level README"
```

---

## Task 14: Copy spec + plan into `docs/`

**Files:**
- Copy: `/Users/asman/Developer/brainstorm/docs/superpowers/specs/2026-05-02-claude-obsidian-plugin-design.md` → `/Users/asman/Developer/claude-obsidian/docs/design.md`
- Copy: `/Users/asman/Developer/brainstorm/docs/superpowers/plans/2026-05-02-claude-obsidian-plugin.md` → `/Users/asman/Developer/claude-obsidian/docs/plan.md`

- [ ] **Step 1: Copy files**

```bash
cp /Users/asman/Developer/brainstorm/docs/superpowers/specs/2026-05-02-claude-obsidian-plugin-design.md /Users/asman/Developer/claude-obsidian/docs/design.md
cp /Users/asman/Developer/brainstorm/docs/superpowers/plans/2026-05-02-claude-obsidian-plugin.md /Users/asman/Developer/claude-obsidian/docs/plan.md
```

- [ ] **Step 2: Verify**

```bash
ls /Users/asman/Developer/claude-obsidian/docs/
```

Expected: `design.md  plan.md`

- [ ] **Step 3: Commit**

```bash
cd /Users/asman/Developer/claude-obsidian
git add docs/
git commit -m "docs: include design + implementation plan"
```

---

## Task 15: Local Claude Code install + verify skills load

**Files:** none created — this is a verification step.

- [ ] **Step 1: Add plugin as a local marketplace**

In Claude Code, run:
```
/plugin marketplace add /Users/asman/Developer/claude-obsidian
```

Expected: marketplace registered without errors. Verify via:
```
/plugin marketplace list
```

You should see `claude-obsidian-marketplace` in the list.

- [ ] **Step 2: Install the plugin**

```
/plugin install claude-obsidian@claude-obsidian-marketplace
```

Expected: install succeeds.

- [ ] **Step 3: Verify skills are discoverable**

In a new Claude Code session, ask:
> "What skills do you have available related to Obsidian?"

Expected: agent lists `obsidian-conventions`, `obsidian-mcp-tools`, `obsidian-brainstorm`.

- [ ] **Step 4: Verify commands are registered**

In a Claude Code session, type `/` and check that `/obsidian-init`, `/obsidian-decision`, `/obsidian-research` appear in the command list.

- [ ] **Step 5: Smoke-test one skill**

Ask Claude Code:
> "Using the obsidian-conventions skill, what's the minimum frontmatter for a note?"

Expected: agent quotes the YAML block from the skill (with `tags`, `aliases`, `created`, `updated`, `type`, `status` fields).

- [ ] **Step 6: Smoke-test `/obsidian-init`**

Run `/obsidian-init /tmp/test-vault`. Verify the skeleton is created:
```bash
find /tmp/test-vault -type f | sort
```

Expected: `00 Home.md`, `MOCs/_template-moc.md`, `Templates/{daily,project,source,adr,note,research}.md`, `README.md`, plus `.gitkeep` files in empty folders.

Cleanup: `rm -rf /tmp/test-vault`.

- [ ] **Step 7: If any verification fails**

Diagnose:
- `plugin.json` malformed? — check `jq . .claude-plugin/plugin.json`
- Skill frontmatter missing? — first 5 lines of each `SKILL.md` should show `--- name: ... description: ... ---`
- Command not appearing? — verify `commands/*.md` has the `description:` frontmatter

Fix and re-run from Step 1.

---

## Task 16: Codex install verify (optional, only if user has Codex)

**Files:** none created.

- [ ] **Step 1: Symlink for Codex**

```bash
mkdir -p ~/.agents/skills
ln -s /Users/asman/Developer/claude-obsidian/skills ~/.agents/skills/claude-obsidian
ls -la ~/.agents/skills/claude-obsidian
```

Expected: symlink listed, pointing to the repo's `skills/` directory.

- [ ] **Step 2: Restart Codex**

Quit and relaunch the Codex CLI.

- [ ] **Step 3: Smoke-test in Codex**

In a new Codex session, ask:
> "What skills do you have related to Obsidian? Using obsidian-conventions, what's the file naming pattern for daily notes?"

Expected: agent lists the 3 skills and answers `YYYY-MM-DD.md`.

- [ ] **Step 4: Cleanup if user doesn't run Codex regularly**

Skip if Codex isn't installed. Note in README that this verification step is optional.

---

## Task 17: Push to GitHub

**Files:** none created — git remote setup.

- [ ] **Step 1: Decide GitHub username**

Ask the user: "What's your GitHub username for this repo?"

- [ ] **Step 2: Replace placeholder in manifests**

```bash
cd /Users/asman/Developer/claude-obsidian
sed -i '' "s|<your-github-user>|<actual-username>|g" .claude-plugin/plugin.json README.md .codex/INSTALL.md
```

(On Linux, drop the `''` after `-i`.)

- [ ] **Step 3: Verify replacements**

```bash
grep -r "<your-github-user>" /Users/asman/Developer/claude-obsidian/ || echo "all replaced"
```

Expected: "all replaced".

- [ ] **Step 4: Commit replacement**

```bash
git add -A
git commit -m "chore: set GitHub username in manifests"
```

- [ ] **Step 5: Create remote and push**

User creates the GitHub repo manually at https://github.com/new (name: `claude-obsidian`, public, no README/license — already in repo). Then:

```bash
cd /Users/asman/Developer/claude-obsidian
git branch -M main
git remote add origin git@github.com:<actual-username>/claude-obsidian.git
git push -u origin main
```

- [ ] **Step 6: Tag v0.1.0**

```bash
git tag v0.1.0
git push origin v0.1.0
```

- [ ] **Step 7: Verify install from remote**

In Claude Code:
```
/plugin marketplace remove claude-obsidian-marketplace      # remove local one
/plugin marketplace add <actual-username>/claude-obsidian
/plugin install claude-obsidian@<actual-username>
```

Expected: same skills + commands available, sourced from remote.

---

## Self-review checklist

Before considering this plan complete, verify:

**1. Spec coverage** — every section of `docs/superpowers/specs/2026-05-02-claude-obsidian-plugin-design.md` maps to a task:
- §3.2 Repo layout → Task 1
- §10.1 plugin.json → Task 2
- §10.2 marketplace.json → Task 2
- §10.3 .codex/INSTALL.md → Task 3
- §4 Conventions encoded → Tasks 7, 8, 9 (all three skills)
- §5 obsidian-conventions → Task 7
- §6 obsidian-mcp-tools → Task 8
- §7 obsidian-brainstorm → Task 9
- §8 Commands → Tasks 10, 11, 12
- §9 Templates → Tasks 4, 5, 6
- §13 Success criteria — covered by verification Tasks 15, 16

No gaps.

**2. Placeholder scan** — searching this plan for `TBD|TODO|FIXME` returns nothing. `<your-github-user>` and `<actual-username>` are intentional template tokens, replaced in Task 17.

**3. Type consistency** — skill names (`obsidian-conventions`, `obsidian-mcp-tools`, `obsidian-brainstorm`) and command names (`obsidian-init`, `obsidian-decision`, `obsidian-research`) are consistent across all tasks.

---

## Execution

When ready to execute, choose:

**1. Subagent-Driven (recommended)** — fresh subagent per task with review between. Best for big content tasks (skills 7, 8, 9).

**2. Inline Execution** — execute in this session with checkpoints. Faster for someone who wants to write the skill prose themselves.
