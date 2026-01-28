# /work-bead — Start working on a bead

**IMPORTANT tool usage notes:**
- Always use absolute paths — never `~` — when passing paths to tools.

## Step 0. Resolve bead

If `$ARGUMENTS` is provided:
- If it looks like a bead ID (e.g. hex string, UUID-like), use it directly: `bd show <id> --json`
- Otherwise, treat it as a **fuzzy search term**. Run `bd list --status open --json` to get all open beads. Search their titles for a substring/fuzzy match against `$ARGUMENTS`. If exactly one match, use that bead. If multiple matches, show them to the user with `AskUserQuestion` and let them pick. If no matches, tell the user nothing matched and stop.

If `$ARGUMENTS` is empty:
- Run `bd list --status open --json` and `bd list --status in_progress --json` to get all actionable beads.
- Present them to the user with `AskUserQuestion` (show ID + title for each, max 4 options — if more than 4, show the first 4 and include a note that they can pass a search term).
- Use the selected bead.

## Steps

1. Run `bd show <bead-id> --json` to get the bead title.
2. Slugify the title: lowercase, replace non-alphanumeric with hyphens, collapse multiple hyphens, trim leading/trailing hyphens. Max 50 chars.
3. Check that `beadhouse.json` exists at the current repo root. If not, tell the user to run `/beadhouse` first and stop.
4. Read `beadhouse.json` and resolve the branch name using `branchPattern` (default: `{{slug}}`). Replace `{{slug}}` with the slug, `{{bead-id}}` with the raw bead ID, `{{project}}` with the repo directory name.
5. Run: `work-bead <repo-root> <branch-name> <slug>`
6. Run: `bd pin <bead-id> --for me --start`
7. Report to the user: the worktree path, bead title, branch name, and a summary of what was set up.

If the worktree already existed (work-bead is idempotent), just report the path and confirm the bead is pinned.

### 8. Offer to launch Claude in worktree
Ask: "Would you like to open a new Claude Code session in the worktree?"

If yes, run:
```bash
osascript -e 'tell application "Terminal" to do script "cd <worktree-path> && claude"'
```
Use the absolute worktree path. This opens a new Terminal tab with Claude running in the worktree directory.
