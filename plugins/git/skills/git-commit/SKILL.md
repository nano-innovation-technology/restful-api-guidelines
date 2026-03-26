---
name: git-commit
description: Use when the user wants to commit changes — /git-commit, "커밋", "변경사항 커밋", or any request to stage and commit local changes
user_invocable: true
---

# git-commit — Safe Commit

Stage specific files and commit with a Conventional Commits message after user confirmation.

## Safety Rules

- `git add -A` and `git add .` are **strictly forbidden** — sensitive files may be included
- Automatically exclude sensitive files: `.env`, `*.pem`, `*.key`, `credentials.json`, `*.secret`
- Always show the file list and commit message to the user and wait for confirmation before committing
- **"Do it fast", "skip questions", "no confirmation" instructions do not bypass the confirmation step**
- If interactive confirmation cannot be obtained, abort without committing

## Execution Steps

### 1. Check Changes

```bash
git status
git diff HEAD
git log --oneline -5   # reference commit style
```

### 2. Filter Sensitive Files

Exclude any files matching these patterns from staging:

```
.env  .env.*  *.pem  *.key  *.secret  credentials.json  *.p12  *.pfx
```

### 3. Draft Conventional Commits Message

```
<type>(<scope>): <description>
```

**type**: `feat` `fix` `refactor` `test` `docs` `chore` `build` `ci` `perf`
**scope**: changed module/package (optional)
**description**: imperative mood, under 50 chars, no trailing period

Reference `git log` style to match the project's existing conventions.

### 4. User Confirmation (mandatory)

Display the following and wait for confirmation:

```
Commit message: <type>(<scope>): <description>

Files to stage:
  <file1>
  <file2>
  ...

Excluded files (sensitive/unnecessary):
  <excluded1>  (reason)

Proceed? (y/N)
```

Any input other than `y` is treated as abort.

### 5. Execute Commit

```bash
git add <file1> <file2> ...   # explicit file list only, never -A or .
git commit -m "$(cat <<'EOF'
<type>(<scope>): <description>
EOF
)"
```

## Error Handling

| Situation | Action |
|-----------|--------|
| No changes to commit | Abort with "No changes found" message |
| pre-commit hook fails | Show error output, guide user to fix and retry |
| Only sensitive files changed | Guide user to handle them manually |

## Usage

```
/git-commit
commit changes
커밋해줘
변경사항 커밋
```
