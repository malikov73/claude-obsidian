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
