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
