# /close-bead — Close a bead and wrap up work

Parse `$ARGUMENTS` as a bead ID. If no argument given, check if there's a pinned bead with `bd pinned --json` and use that.

**IMPORTANT tool usage notes:**
- Always use absolute paths — never `~` — when passing paths to tools.

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
