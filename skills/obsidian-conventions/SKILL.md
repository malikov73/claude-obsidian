---
name: obsidian-conventions
description: "Use whenever reading, writing, or organizing notes in an Obsidian vault. Encodes vault shape, file naming, frontmatter spec, linking, atomicity, append-vs-rewrite, and search-before-create rules."
---

# Obsidian Conventions

This skill defines how AI agents read and write notes in an Obsidian vault. It assumes a hybrid LYT-flavored vault — a folder structure based on lifecycle/state, Maps of Content (MOCs) for navigation, and atomic permanent notes for knowledge. If your vault uses different conventions (pure PARA, pure Zettelkasten, Johnny Decimal), follow those instead — these are defaults for new vaults or as a baseline when no project-level convention is documented. The goal is to make every write operation reproducible: another agent reading the vault later should see a coherent, navigable structure.

This skill is paired with `obsidian-mcp-tools` (tool selection: which MCP call to use for each operation) and `obsidian-brainstorm` (brainstorm-specific patterns: decision logs, research notes, P&L tables).

---

## Vault shape

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

Rule: one level of subfolder maximum inside each top-level folder. Deeper nesting creates navigation overhead without proportional benefit — use tags to carry topic membership instead. Keep folders flat; folders carry state, not topic.

---

## Note types (closed enum)

Nine recognized types. Each note must declare exactly one in its `type` frontmatter field. Adding a new type requires updating this skill — do not invent undeclared types without updating the enum here.

- **daily** — dated log entry; lives in `Daily/YYYY/`; captures events, decisions, and links to active notes. One per calendar day.
- **moc** — Map of Content; an index note that links to other notes on a topic. Not a container — the linked notes live in their own locations. Lives in `MOCs/` or at top level.
- **project** — scoped deliverable with a defined outcome and end state. Lives in `Projects/NN <Name>/`. Contains or links to all work artifacts for that project.
- **source** — literature note tied to one external source (book, article, video, paper). Lives in `Sources/`. One source per file; captures key claims and your reaction.
- **note** — permanent atomic note encoding one idea. Lives in `Resources/` or a project folder. Title is a complete claim (see Atomicity section).
- **adr** — Architecture Decision Record. Captures one technical or product decision with context, options considered, and rationale. Lives in `Projects/NN <Name>/` or a top-level `Decisions/` folder. Named `ADR-NNNN-title.md`.
- **research** — synthesis note aggregating findings on a question across multiple sources. Lives in `Resources/` or project folder. Distinguished from `source` (which is one-to-one with a source) by being many-to-one.
- **idea** — early-stage capture that has not yet been validated or built out. Lives in `Inbox/` until processed into a `note`, `project`, or discarded.
- **log** — append-only chronological record: meeting notes, decision history, experiment runs. Not subject to atomicity rules — intentionally a running record.

---

## Minimum frontmatter

Every note must have this minimum frontmatter block:

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

Update `updated` on every meaningful edit. The `tags` list must include a `type/` tag matching the `type` field — this redundancy enables both Dataview queries and tag-based navigation.

### Per-type extensions

Fields below extend the minimum set. Include only what applies to the type.

**ADR:**
```yaml
type: adr
id: ADR-NNNN
status: proposed       # proposed | accepted | deprecated | superseded
date: 2026-05-02
deciders: [self]
supersedes:
superseded-by:
```

**Source:**
```yaml
type: source
source-url: https://...
author:
year:
status: to-read       # to-read | reading | read
```

**Project:**
```yaml
type: project
status: active
deadline:
```

**Research:**
```yaml
type: research
status: active
sources: []
```

### Frontmatter rules

- Internal wikilinks in YAML must be quoted: `related: "[[Note]]"`. Unquoted brackets break the YAML parser and corrupt Obsidian's graph view.
- Dates in `YYYY-MM-DD` (ISO 8601) — Obsidian recognizes them as Date type and Dataview parses them natively.
- Keep frontmatter 5–10 fields max. Larger blocks get abandoned and become stale faster than they provide value.
- Keys lowercase, kebab-case or snake_case — be consistent within a vault. Do not mix conventions in the same vault.
- Lists use YAML inline `[a, b]` or block `- item` style — pick one per field and keep it consistent.
- Use `aliases` for natural-language variants of the title (e.g., `aliases: ["DE pricing", "Germany pricing"]`) so wikilinks to either form resolve correctly.
- Markdown is not rendered in Properties — keep values plain text. No bold, no italics, no embedded links in values (except properly quoted wikilinks).

---

## File naming

| Context | Pattern | Example |
|---|---|---|
| Default | Descriptive title | `Wedding speech pricing.md` |
| Project child | `NN Title.md` | `05 P&L и экономика.md` |
| Daily | `YYYY-MM-DD.md` | `2026-05-02.md` |
| ADR | `ADR-NNNN-title.md` | `ADR-0012-paystack.md` |
| Source | Descriptive | `Ahrens — Smart Notes.md` |

Default notes use a natural descriptive title that matches the note's atomic claim. Project child notes use a two-digit prefix (`NN`) to keep them ordered in the filesystem — this matters when a project folder has 5–15 files covering distinct phases. Daily notes use strict ISO date for chronological sort. ADRs use a zero-padded four-digit sequence so they sort correctly past ADR-9. Source notes use the author name for fast disambiguation when multiple books on the same topic exist.

Avoid spaces if you script against the vault externally — use `-` or `_` instead. Never use colons, forward slashes, backslashes, or quote characters in filenames: they are filesystem-unsafe on macOS/Windows and will cause silent failures or sync errors.

---

## Linking

### Wikilinks always

Use `[[wikilinks]]` as the default link format. Obsidian updates them automatically on rename, they produce cleaner diffs than Markdown links, and they participate in the graph view. Use Markdown links `[text](path)` only when the file will be consumed by tools that do not understand wikilinks (e.g., publishing pipelines, GitHub rendering).

```markdown
[[Note]]                 # basic link
[[Note|alias]]           # alias display
[[Note#Section]]         # link to heading
[[Note#^block-id]]       # link to block
![[Note#Section]]        # embed/transclude
```

Transclude (`![[...]]`) sparingly — it creates tight coupling between notes. Prefer a regular link with a one-sentence summary of why the link exists.

### Hierarchical tags

Use hierarchical tags for status lifecycles, type taxonomies, and project membership. The slash creates a parent-child relationship in Obsidian's tag pane.

Examples:
- `type/note`, `type/adr` (and the other 7 types from §Note types: `type/daily`, `type/moc`, `type/project`, `type/source`, `type/research`, `type/idea`, `type/log`). The `type/<value>` tag mirrors the frontmatter `type:` field and follows the same closed enum.
- `status/active`, `status/done`, `status/superseded`
- `project/wedding-speech`, `project/brainstorm`
- `cross-cutting/multi-language`, `cross-cutting/pricing` (brainstorm-specific; see `obsidian-brainstorm` skill)

Refactor tags across the vault using the Tag Wrangler community plugin — do not manually find-and-replace tag strings, as that introduces inconsistencies.

---

## Folders vs tags

Concrete rule: **Folder = state/lifecycle (one per note). Tags = topic/type/status (many per note).**

A note belongs in exactly one folder because a file can only exist in one place on disk. Folders encode where the note is in its lifecycle: `Inbox/` (unprocessed), `Projects/NN/` (active scoped work), `Resources/` (permanent reference), `Archive/` (done/deprecated). That is the only question a folder answers.

Everything else — what topic the note covers, what project it relates to, what type it is — is a tag.

**Anti-example:** Do not create `Wedding/`, `Pricing/`, `Marketing/`, `DE-market/` folders. Those are topics, not lifecycle states. Instead:

```
Projects/07 Wedding Speech/05 P&L и экономика.md
  tags: [type/note, project/wedding-speech, topic/pricing, topic/de-market]
```

Why this matters: topic-based folders force you to choose one topic per note (a note about DE pricing for a wedding speech project belongs in `Wedding/`, `Pricing/`, or `DE-market/` — pick one and lose the others). Tags have no such constraint.

---

## Atomicity

Rule: **one self-contained idea per permanent note.** The title should be a complete claim — a statement that is either true or false — not a noun phrase.

**Compliant titles (claims):**
- `Pricing in DE markets requires inclusive VAT.md`
- `Multi-language SEO requires localized URL slugs.md`
- `Opus-class LLMs outperform GPT-4o on emotional speech generation.md`

**Non-compliant titles (split required):**
- `Wedding speech pricing and marketing.md` — the "and" signals two ideas; split into two files
- `DE pricing.md` — noun phrase, not a claim; what about DE pricing? Make it a statement.
- `Research notes.md` — not a note, it's a container; use a MOC or research type instead

The payoff: an atomic note is reusable across multiple MOCs and projects. A compound note is only useful in the one context it was written for.

**Exceptions:** MOCs, reference docs (P&L tables, competitor matrices, feature comparison tables) are intentionally aggregate — they are indexes and reference material, not knowledge claims. Do not apply the atomicity rule to them.

---

## Append vs rewrite

This section governs the most consequential write decision: whether to modify an existing note or create a new one.

**Append with UPDATE block** when adding a refinement, correction, or new datapoint to an existing claim, AND the addition is under 150 words and tightly related to the note's core claim:

```markdown

## UPDATE 2026-05-02: <topic>

<content>
```

The UPDATE heading preserves the original claim (which may be linked from elsewhere) while making the revision date and scope explicit. Other notes linking to this one continue to work. The audit trail is intact.

**Split into new file** when:
- The new content is a new idea that can stand alone as its own claim
- It is over 150 words unrelated to the host note's core claim
- It will be referenced from other notes independently of the host

**Never delete.** Deletion breaks backlinks silently — other notes that linked to the deleted file show dead links with no warning in reading view. Instead:
- Mark `status: superseded` in frontmatter
- Add `superseded-by: "[[<replacement-note>]]"` to frontmatter
- Optionally move to `Archive/`
- Add a visible note at the top: `> Superseded by [[Replacement note]] (2026-05-02)`

This preserves the audit trail and prevents broken backlinks.

**Decision flow:**

1. Adding refinement / correction / datapoint to existing claim, AND under 150 words → **append with UPDATE block**.
2. New idea that can stand alone, OR over 150 words unrelated to host → **split into new file**, link from original.
3. Outdated claim being replaced → **mark `status: superseded`**, link to replacement, move to Archive.

---

## Search-before-create

Before creating any new note, run these steps. Skipping them creates duplicate notes that fragment the knowledge graph and make decisions ambiguous. The cost of two extra search calls is trivial compared to the cost of merging duplicates later.

1. Run `obsidian_simple_search` with the prospective title (or a key phrase from it).
2. Run `obsidian_simple_search` with each alias you plan to add to the new note.
3. Zero matches → proceed to create the new note.
4. Existing matches → read each match via `obsidian_get_file_contents`, then decide:
   - Is this the same idea? → **append** with UPDATE block (see Append vs rewrite).
   - Is this a related but distinct idea? → **create new**, add a wikilink from the existing note to the new one.
   - Is this a stub waiting to be filled out? → **expand the existing note** rather than creating a parallel one.

Always make the cross-reference explicit. If you create a new note because an existing note covers a related but distinct idea, add `related: "[[Existing note]]"` to the frontmatter of the new note and add a link back from the existing note.

---

## Templates

These paths are relative to the **plugin install root** (e.g., `~/.claude/plugins/cache/<marketplace>/claude-obsidian/<version>/` in Claude Code, or `~/.codex/claude-obsidian/` in Codex). The `/obsidian-init` command (Claude Code) or manual copy (Codex) places these into your target vault.

Templates are in two locations in this repository:

- **Vault skeleton templates:** `templates/vault-skeleton/Templates/` — one file per note type:
  - `daily.md`, `project.md`, `source.md`, `adr.md`, `note.md`, `research.md`
- **Standalone templates:** `templates/` root:
  - `adr-madr.md` — MADR-format ADR with full decision log structure
  - `moc.md` — MOC scaffold with sections for overview, links, and metadata
  - `research-note.md` — research note with question, findings, and synthesis sections

Templates use `{{title}}` and `{{date:YYYY-MM-DD}}` placeholders. These are compatible with Obsidian's **Templates** core plugin and the **Templater** community plugin (Templater uses `<% tp.file.title %>` syntax — use that form if Templater is active in the vault).

When applying a template to a new note, replace all placeholder values before saving. Do not leave `{{title}}` or `{{date}}` unexpanded in a committed note — they indicate an incomplete setup.

---

## Platform notes

On Codex, native file tools (`Read`, `Write`, `Edit`) replace the MCP-based `obsidian_*` tools for file I/O. The MCP `mcp__obsidian__*` tools work identically across Claude Code and other platforms — see the `obsidian-mcp-tools` skill for the full tool selection guide (which tool to use for which operation, and how to handle errors). The slash commands `/obsidian-init`, `/obsidian-decision`, and `/obsidian-research` are Claude Code-only — Codex agents perform the equivalent flows manually by following this skill directly, using native file tools in place of MCP calls. The rules in this skill (frontmatter, naming, atomicity, append-vs-rewrite, search-before-create) apply regardless of platform.

---

## Sources

- Nick Milo (LYT framework + MOCs) — https://medium.com/@nickmilo22/in-what-ways-can-we-form-useful-relationships-between-notes-9b9ec46973c6
- Andy Matuschak (atomic notes) — https://notes.andymatuschak.org/Evergreen_notes_should_be_atomic
- Sönke Ahrens (Smart Notes pipeline) — https://www.zettelkasten.de/posts/concepts-sohnke-ahrens-explained/
- Obsidian Properties docs — https://help.obsidian.md/properties
- MADR ADR template — https://github.com/adr/madr
