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
