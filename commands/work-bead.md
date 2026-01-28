# /work-bead — Start working on a bead

**IMPORTANT tool usage notes:**
- Always use absolute paths — never `~` — when passing paths to tools.

## Step 0. Resolve bead

**YOU MUST resolve the bead yourself. NEVER tell the user to run a bd command themselves. NEVER suggest they run `bd ready`, `bd list`, `bd hook`, etc. YOU do the lookup and present choices.**

If `$ARGUMENTS` is provided:
- If it looks like a bead ID (e.g. starts with `bd-` or is a hex/hash string), use it directly: run `bd show <id> --json`
- Otherwise, treat it as a **fuzzy search term**. Run `bd list --status open --json` to get all open beads. Search their titles for a case-insensitive substring match against `$ARGUMENTS`. If exactly one match, use that bead. If multiple matches, present them with `AskUserQuestion` and let the user pick. If no matches, tell the user nothing matched and stop.

If `$ARGUMENTS` is empty:
1. Run `bd list --status open --json` and `bd list --status in_progress --json` to get all actionable beads.
2. You MUST present the results to the user with `AskUserQuestion` (show ID + title for each, max 4 options). If more than 4, show the first 4 and note they can pass a search term like `/work-bead auth`.
3. Use the bead the user selects.

## Steps

1. Run `bd show <bead-id> --json` to get the bead title.
2. Slugify the title: lowercase, replace non-alphanumeric with hyphens, collapse multiple hyphens, trim leading/trailing hyphens. Max 50 chars.
3. Check that `beadhouse.json` exists at the current repo root. If not, tell the user to run `/beadhouse` first and stop.
4. Read `beadhouse.json` and resolve the branch name using `branchPattern` (default: `{{slug}}`). Replace `{{slug}}` with the slug, `{{bead-id}}` with the raw bead ID, `{{project}}` with the repo directory name.
5. **Choose base branch.** Check what branch is currently checked out in the repo (`git -C <repo-root> branch --show-current`). Determine the default branch (`git -C <repo-root> remote show origin | grep 'HEAD branch'`).
   - If the repo is on the default branch → use `origin/<default-branch>` as the start point. Fetch it first: `git -C <repo-root> fetch origin <default-branch> --quiet`
   - If the repo is on a different branch → ask the user with `AskUserQuestion`: "Branch from `<current-branch>` or `<default-branch>`?"
   - Pass the chosen start point as the 4th argument to `work-bead`.
6. Run: `work-bead <repo-root> <branch-name> <slug> [start-point]`
7. Run: `bd pin <bead-id> --for me --start`
8. Report to the user: the worktree path, bead title, branch name, base branch, and a summary of what was set up.

If the worktree already existed (work-bead is idempotent), just report the path and confirm the bead is pinned.

### 9. Work in the worktree
The Bash tool resets its working directory between calls, so `cd` will not persist. Instead, for ALL subsequent Bash commands in this conversation, prefix them with `cd <worktree-path> &&`. For file reads/edits/writes, use absolute paths rooted in the worktree.

Tell the user: "All commands will now run in `<worktree-path>`. Ready to work on the bead."
