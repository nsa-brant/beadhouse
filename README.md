# Beadhouse

Per-project worktree config for beads. Clone, install, and Claude gets three commands for managing bead workflows with git worktrees.

## Install

```bash
git clone <repo> ~/develop/nsa/beadhouse
~/develop/nsa/beadhouse/install
```

Requires: `git`, `jq`, `gh` (GitHub CLI), `bd` (bead CLI)

The install script symlinks:
- `/create-bead`, `/work-bead`, `/close-bead`, `/beadhouse` → `~/.claude/commands/`
- `work-bead` script → `~/.local/bin/`

Re-run `./install` anytime to update. Existing files are backed up.

## Commands

### `/beadhouse` — Initialize a project

Run in any git repo. Claude will:
1. Detect your tech stack (Node, Python, Go, Rust, etc.)
2. Find `.env*` files
3. Warn about worktree replicability issues (sqlite, symlinks, absolute paths, docker volumes)
4. Prompt you to confirm config
5. Write `beadhouse.json` to the repo root

### `/create-bead` — Create a new bead

Prompts for title, priority, and tags. Creates the bead with `bd create`, then offers to immediately start working on it (runs the `/work-bead` flow).

### `/work-bead <bead-id>` — Start working on a bead

Reads `beadhouse.json`, creates a git worktree, copies env files, runs setup commands, and pins the bead. Idempotent — safe to run twice.

### `/close-bead [bead-id]` — Close a bead

Walks you through the full close flow:
1. Commit uncommitted changes
2. Create a PR
3. Monitor CI checks and merge when green
4. Close the bead with `bd close`
5. Append to changelog

Each step prompts for confirmation — you can bail at any point. If no bead ID given, uses the currently pinned bead.

## beadhouse.json

Lives at the root of each project repo. Tells `work-bead` how to set up worktrees.

```json
{
  "worktreePath": "~/develop/nsa/worktrees/{{project}}/{{slug}}",
  "branchPattern": "feature/{{slug}}",
  "envFiles": [".env", ".env.local"],
  "setup": ["npm install"],
  "warnings": ["sqlite database at data/app.db — will not be shared across worktrees"]
}
```

### Fields

| Field | Default | Description |
|-------|---------|-------------|
| `worktreePath` | `~/develop/nsa/worktrees/{{project}}/{{slug}}` | Where worktrees are created |
| `branchPattern` | `{{slug}}` | Branch naming pattern |
| `envFiles` | `[".env"]` | Files copied from main repo into each worktree |
| `setup` | `[]` | Commands run inside the worktree after creation |
| `warnings` | `[]` | Replicability warnings detected by `/beadhouse` |

All fields are optional. Omit any that match the default.

### Template variables

| Variable | Resolves to |
|----------|------------|
| `{{project}}` | Repo directory name |
| `{{slug}}` | Slugified bead title (lowercase, hyphens, max 50 chars) |
| `{{bead-id}}` | Raw bead ID |

### Examples

**Minimal** — Node project, defaults are fine:
```json
{
  "setup": ["npm install"]
}
```

**Python with multiple env files:**
```json
{
  "envFiles": [".env", ".env.local"],
  "setup": ["pip install -r requirements.txt"]
}
```

**Custom branch pattern and worktree location:**
```json
{
  "worktreePath": "~/worktrees/{{project}}/{{slug}}",
  "branchPattern": "feat/{{bead-id}}-{{slug}}"
}
```

## Suggested CLAUDE.md snippet

Add this to your `~/.claude/CLAUDE.md` so Claude proactively suggests bead commands during conversation:

```markdown
## Beadhouse
When the user mentions creating a task, starting work on something, or wrapping up work, proactively suggest the relevant bead command:
- Starting a new task → offer `/create-bead`
- Picking up existing work → offer `/work-bead`
- Finishing up / done with work → offer `/close-bead`

Don't force it — just mention it naturally if the context fits.
```

## Bead resolution

All bead commands support flexible input:
- **No argument** → shows a picker of open/in-progress beads
- **Bead ID** → uses it directly
- **Fuzzy text** → searches bead titles (e.g. `/work-bead auth` finds "Add authentication")

`/close-bead` with no argument checks for a pinned bead first.

## How it works

```
/create-bead
    │
    ▼
bd create → bead ID
    │
    ├── "Start working?" → no → done
    │
    ▼
/work-bead <bead-id>
    │
    ▼
read beadhouse.json
    │
    ▼
git worktree add → copy .env → run setup → bd pin
    │
    ▼
  (you work)
    │
    ▼
/close-bead
    │
    ▼
commit → PR → CI watch → merge → bd close → changelog
```
