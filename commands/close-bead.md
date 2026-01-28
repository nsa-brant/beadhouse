# /close-bead — Close a bead and wrap up work

**IMPORTANT tool usage notes:**
- Always use absolute paths — never `~` — when passing paths to tools.

## Step 0. Resolve bead

If `$ARGUMENTS` is provided:
- If it looks like a bead ID (e.g. hex string, UUID-like), use it directly: `bd show <id> --json`
- Otherwise, treat it as a **fuzzy search term**. Run `bd list --status in_progress --json` to get in-progress beads. Search their titles for a substring/fuzzy match against `$ARGUMENTS`. If exactly one match, use that bead. If multiple matches, show them to the user with `AskUserQuestion` and let them pick. If no matches, tell the user nothing matched and stop.

If `$ARGUMENTS` is empty:
- First check for a pinned bead: `bd pinned --json`. If one is pinned, use it.
- Otherwise, run `bd list --status in_progress --json` to get all in-progress beads.
- Present them to the user with `AskUserQuestion` (show ID + title for each, max 4 options — if more than 4, show the first 4 and include a note that they can pass a search term).
- Use the selected bead.

## Steps

### 1. Get bead info
Run `bd show <bead-id> --json` to get the bead title and status.

### 2. Commit changes
Run `git status` in the current worktree. If there are uncommitted changes:
- Show the user a summary of what changed (files modified/added/deleted)
- Ask: "Would you like to commit these changes?"
- If yes, stage and commit following normal git commit conventions (ask for or generate a commit message)

If no uncommitted changes, skip to step 3.

### 3. Create PR
Ask: "Would you like to create a pull request?"

If yes:
- Determine the base branch (usually `main` or `master` — check with `git remote show origin | grep 'HEAD branch'`)
- Push the branch: `git push -u nsa-github HEAD`
- Create a PR with `gh pr create` using the bead title as the PR title and a summary body
- Report the PR URL to the user

If no, skip to step 4.

### 4. Monitor and merge
If a PR was created (or already exists), ask: "Would you like me to monitor the PR checks and merge when they pass?"

If yes:
- Run `gh pr checks <pr-url> --watch` to wait for CI
- Report check results to the user
- If all checks pass, run `gh pr merge <pr-url> --squash --delete-branch`
- Report merge status

If checks fail, report the failures and stop — do not merge.

### 5. Close bead
Run `bd close <bead-id>`.

### 6. Changelog
Append a one-liner to `~/develop/nsa/changelog/YYYY-MM-DD.md` (use today's date, absolute path):
```
- HH:MM ✓ `<bead-id>` - concise summary of what was done
```
Create the file and any project header if needed. Then auto-commit and push:
```bash
cd ~/develop/nsa/changelog && git add -A && git commit -m "update" && git push
```

### 7. Report
Summarize what was done: commit, PR, merge status, bead closed.
