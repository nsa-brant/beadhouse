# /work-bead — Start working on a bead

Parse `$ARGUMENTS` as a bead ID. Then:

1. Run `bd show <bead-id> --json` to get the bead title.
2. Slugify the title: lowercase, replace non-alphanumeric with hyphens, collapse multiple hyphens, trim leading/trailing hyphens. Max 50 chars.
3. Check that `beadhouse.json` exists at the current repo root. If not, tell the user to run `/beadhouse` first and stop.
4. Read `beadhouse.json` and resolve the branch name using `branchPattern` (default: `{{slug}}`). Replace `{{slug}}` with the slug, `{{bead-id}}` with the raw bead ID, `{{project}}` with the repo directory name.
5. Run: `work-bead <repo-root> <branch-name> <slug>`
6. Run: `bd pin <bead-id> --for me --start`
7. Report to the user: the worktree path, bead title, branch name, and a summary of what was set up.

If the worktree already existed (work-bead is idempotent), just report the path and confirm the bead is pinned.
