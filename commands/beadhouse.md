# /beadhouse — Initialize beadhouse.json for a project

Inspect the current repo and generate a `beadhouse.json` config file at the repo root.

**IMPORTANT tool usage notes:**
- Use the `Glob` tool (not Bash `ls` or `find`) for all file searches.
- Always use absolute paths — never `~` — when passing paths to tools.
- The repo root is your current working directory.

## Steps

### 1. Detect tech stack
Use `Glob` to check which of these files exist at the repo root:
- `package.json` (Node/JS/TS)
- `requirements.txt`, `pyproject.toml`, `setup.py` (Python)
- `Gemfile` (Ruby)
- `go.mod` (Go)
- `Cargo.toml` (Rust)
- `docker-compose.yml` / `docker-compose.yaml` (Docker)
- `Makefile`

Run a single Glob with pattern `{package.json,requirements.txt,pyproject.toml,setup.py,Gemfile,go.mod,Cargo.toml,docker-compose.yml,docker-compose.yaml,Makefile}` at the repo root.

### 2. Find env files
Use `Glob` with pattern `.env*` at the repo root.

### 3. Check replicability warnings
Use `Glob` to scan for potential issues. Run these in parallel:
- `**/*.sqlite` and `**/*.db` — SQLite databases
- `data/**` — runtime state directories
- `.gitattributes` — check for git-lfs
- `docker-compose.{yml,yaml}` — if found, read it and check for absolute path bind mounts

Also check for symlinks with `Bash`: `find <repo-root> -maxdepth 2 -type l 2>/dev/null` (use absolute path).

### 4. Warn user
If any replicability issues were found, clearly explain each one and what it means for worktree usage.

### 5. Prompt user for config
First, **print the proposed config as a formatted JSON code block** in your text output so the user can read it clearly:

```
Here's the proposed beadhouse.json:
```

Then show the full JSON with proper formatting. After that, use `AskUserQuestion` with **short, simple options** — do NOT put config values in the option labels or descriptions. Use options like:

- "Looks good" — Write this config as-is
- "Edit config" — I want to change something before writing

If the user wants to edit, ask specific follow-up questions about what to change (worktree path, branch pattern, env files, setup commands, etc.) one at a time.

### 6. Write beadhouse.json
Write the confirmed config to `beadhouse.json` at the repo root. Use the schema:

```json
{
  "$schema": "https://raw.githubusercontent.com/nsa-brant/beadhouse/master/beadhouse.schema.json",
  "worktreePath": "~/develop/nsa/worktrees/{{project}}/{{slug}}",
  "branchPattern": "{{slug}}",
  "envFiles": [".env"],
  "setup": [],
  "warnings": []
}
```

Always include the `$schema` line. Omit other fields that match defaults to keep the file minimal.
