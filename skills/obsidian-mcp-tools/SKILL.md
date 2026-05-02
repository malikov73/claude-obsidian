---
name: obsidian-mcp-tools
description: "Use when interacting with an Obsidian vault via the obsidian MCP server. Reference cheatsheet for which mcp__obsidian__* tool to call when, common patterns, JsonLogic snippets, and gotchas."
---

# Obsidian MCP Tools

Decision matrix and patterns for the `mcp__obsidian__*` tool family. Pair with `obsidian-conventions` for content rules — this skill is about *how* to interact with the vault, that one is about *what* to write.

## Tool decision matrix

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

## Loading deferred tools

On Claude Code, all `mcp__obsidian__*` tools are deferred. Load needed tools in one batch via `ToolSearch` with `select:` syntax. Example:

```
ToolSearch(query="select:mcp__obsidian__obsidian_simple_search,mcp__obsidian__obsidian_get_file_contents,mcp__obsidian__obsidian_append_content")
```

On Codex, MCP tools load through the platform's native MCP integration — refer to your platform's docs.

## Three canonical patterns

### Pattern A: Find-before-create

1. `obsidian_simple_search(query="<prospective title>")` → check matches
2. If 0 matches: `obsidian_append_content(filepath="<path>", content="...")` (creates file)
3. If matches: `obsidian_get_file_contents(filepath="<match>")` → decide append or new with cross-reference

### Pattern B: Append with UPDATE block

```python
obsidian_append_content(
    filepath="07 Wedding Speech/01 Идея.md",
    content="\n\n## UPDATE 2026-05-02: Pricing revised\n\nDecision changed from $19 → $24 because [reason]."
)
```

### Pattern C: Surgical update with patch

```python
obsidian_patch_content(
    filepath="07 Wedding Speech/05 P&L.md",
    operation="replace",
    targetType="heading",
    target="Year 2 ARR target",
    content="$80–200k (revised 2026-05-02)"
)
```

## patch_content operations reference

`obsidian_patch_content` takes `operation` + `targetType` + `target`:

| operation | Effect |
|---|---|
| `replace` | Overwrite the heading/block content entirely |
| `append` | Add content after the target heading's block |
| `prepend` | Add content before the target heading's block |

`targetType` options:
- `heading` — match by heading text (must be unique in file)
- `block` — match by Obsidian block reference `^block-id`

Minimal prepend example — add a warning before a section:

```python
obsidian_patch_content(
    filepath="07 Wedding Speech/03 План реализации.md",
    operation="prepend",
    targetType="heading",
    target="Phase 2: Launch",
    content="> [!WARNING] Blocked on payment processor decision\n\n"
)
```

## batch_get_file_contents usage

Pass a list of vault-relative paths. Returns all contents in one call:

```python
obsidian_batch_get_file_contents(
    filepaths=[
        "07 Wedding Speech/01 Идея.md",
        "07 Wedding Speech/02 Конкуренты.md",
        "07 Wedding Speech/05 P&L.md"
    ]
)
```

Use this any time you need to read 3+ files as context before making a decision or writing an update. Single-call cost is much lower than N sequential `get_file_contents` calls.

## JsonLogic for complex_search

```json
// All ADRs in a project folder, sorted by date
{
  "and": [
    { "glob": ["07 Wedding Speech/**/*.md", { "var": "path" }] },
    { "==": [{ "var": "frontmatter.type" }, "adr"] }
  ]
}
```

For time-based queries: prefer `obsidian_get_recent_changes` over JsonLogic regex on dates — it's faster and less error-prone.

```json
// Notes with specific tag
{
  "in": ["project/wedding-speech", { "var": "frontmatter.tags" }]
}
```

```json
// Notes missing a required frontmatter field (status not set)
{
  "!": { "var": "frontmatter.status" }
}
```

Available `var` paths in JsonLogic context:
- `path` — vault-relative file path
- `frontmatter.<key>` — any frontmatter field
- `content` — full file text (expensive; avoid in large vaults)
- `stat.mtime` — last-modified epoch ms

## Gotchas

- `obsidian_patch_content` requires a **unique** heading. If "Pricing" appears twice, the patch fails. Use a more specific heading or fall back to `append_content`.
- `obsidian_simple_search` searches **content**, not just filename. To check existence by name only, use `list_files_in_dir`.
- Wikilinks in YAML frontmatter **must be quoted** (`related: "[[Note]]"`) — unquoted breaks YAML parsing AND the patch tool.
- `obsidian_append_content` always appends to file end. To insert mid-file, use `patch_content` with `prepend`/`append` operation under a target heading.
- Paths are vault-relative. No leading `/`.

## Anti-patterns

- ❌ Looping `get_file_contents` for many files → use `batch_get_file_contents`.
- ❌ Using `delete_file` to retire a note → set `status: archived` and move to `Archive/` or just leave with status flag.
- ❌ SQL syntax in `complex_search` → JsonLogic only.
- ❌ Skipping search-before-create → leads to duplicate notes with conflicting state.
- ❌ Using `patch_content` on a non-unique heading → silent or noisy fail. Verify uniqueness first.
