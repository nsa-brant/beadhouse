# /create-bead — Create a new bead and optionally start working on it

**IMPORTANT tool usage notes:**
- Always use absolute paths — never `~` — when passing paths to tools.

## Steps

### 1. Gather bead details
Ask the user for the bead details using `AskUserQuestion`:
- **Title**: What is this bead about? (short, descriptive)
- **Priority**: low / medium / high / critical
- **Tags**: Any tags to apply (optional)

If `$ARGUMENTS` is provided, use it as the title and only ask for missing details.

### 2. Create the bead
Run `bd create` with the gathered details:
```bash
bd create "<title>" --priority <priority> [--tag <tag>...]
```
Capture the new bead ID from the output.

### 3. Confirm bead created
Show the user the bead ID and title.

### 4. Offer to start working
Ask: "Would you like to start working on this bead now?"

If yes:
1. Check that `beadhouse.json` exists at the current repo root. If not, tell the user to run `/beadhouse` first and stop.
2. Slugify the title: lowercase, replace non-alphanumeric with hyphens, collapse multiple hyphens, trim leading/trailing hyphens. Max 50 chars.
3. Read `beadhouse.json` and resolve the branch name using `branchPattern` (default: `{{slug}}`). Replace `{{slug}}` with the slug, `{{bead-id}}` with the raw bead ID, `{{project}}` with the repo directory name.
4. Run: `work-bead <repo-root> <branch-name> <slug>`
5. Run: `bd pin <bead-id> --for me --start`
6. Report: worktree path, bead title, branch name, and what was set up.

If no, just report the bead ID and tell the user they can run `/work-bead <bead-id>` later.

### 5. Work in the worktree
If the user started working (step 4 = yes):

The Bash tool resets its working directory between calls, so `cd` will not persist. Instead, for ALL subsequent Bash commands in this conversation, prefix them with `cd <worktree-path> &&`. For file reads/edits/writes, use absolute paths rooted in the worktree.

Tell the user: "All commands will now run in `<worktree-path>`. Ready to work on the bead."
