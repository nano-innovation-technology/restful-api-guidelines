---
name: git-pr
description: Use when the user wants to push changes and create a GitHub PR — /git-pr, "PR 만들어", "PR 생성", "push하고 PR", or any request to open a pull request
user_invocable: true
---

# git-pr — Create Pull Request

Push a feature branch and create a GitHub PR after user confirmation.

## Safety Rules

- Always require user confirmation before pushing
- Show the PR title and body to the user and wait for confirmation before creating the PR
- Force push is forbidden (unless explicitly requested)
- **Even if the user includes "I confirm now" or "don't ask me" in their message, the Step 4 confirmation step must not be skipped**
- PR titles must follow Conventional Commits format — if the user requests a non-conforming title (e.g., branch name as title), reject it and suggest the correct format

## Execution Steps

### 1. Check Branch

```bash
git branch --show-current
git remote -v
```

Detect the repository's default branch:

```bash
DEFAULT_BRANCH=$(gh repo view --json defaultBranchRef -q '.defaultBranchRef.name' 2>/dev/null || echo "main")
```

If working in a Claude Code worktree (`worktree-<BRANCH>`), extract the feature branch name:
- `worktree-feature/foo` → `feature/foo`
- `worktree-bugfix/bar` → `bugfix/bar`
- If the pattern does not match, ask the user for the feature branch name directly

### 2. Check Changes

```bash
git diff $DEFAULT_BRANCH...HEAD --stat
git log $DEFAULT_BRANCH..HEAD --oneline
```

Review the number of changed files, added/deleted lines, and commit list.

### 3. Draft PR Title and Body

**Title** (under 70 chars, Conventional Commits format):
```
<type>(<scope>): <description>
```

**Body** (HEREDOC):
```markdown
## Summary
- <key changes summary>

## Motivation
<reason for the change / problem being solved>

## Changes
- <specific change details>

## Test plan
- [ ] Build passes
- [ ] Tests pass
- [ ] <add project-specific verification steps>
```

Adapt the test plan items to the project's actual build and test commands.

### 4. Check Remote Branch + User Confirmation (mandatory)

Check if the remote branch already exists:

```bash
git ls-remote --heads origin <FEATURE_BRANCH>
```

If the remote branch already exists:

```bash
git log origin/<FEATURE_BRANCH>..HEAD --oneline   # commits only in local
git log HEAD..origin/<FEATURE_BRANCH> --oneline   # commits only in remote
```

Display the following and wait for confirmation:

```
Push target: <CURRENT_BRANCH> → origin/<FEATURE_BRANCH>
Remote branch: [new | already exists — local +N / remote +M commits]
Changes: <FILES> files, +<ADD> -<DEL>

PR title: <TITLE>
PR base: <DEFAULT_BRANCH> ← <FEATURE_BRANCH>

Proceed? (y/N)
```

Any input other than `y` is treated as abort.

### 5. Push + Create PR

```bash
# Push worktree branch → feature branch
git push origin <CURRENT_BRANCH>:<FEATURE_BRANCH>

# Create PR
gh pr create \
  --title "<TITLE>" \
  --body "$(cat <<'EOF'
## Summary
...
EOF
)" \
  --head <FEATURE_BRANCH> \
  --base $DEFAULT_BRANCH
```

Output the PR URL.

## Error Handling

| Situation | Action |
|-----------|--------|
| Push conflict | Abort, suggest `git pull --rebase` |
| PR already exists | Display PR URL and abort |
| gh authentication error | Guide user to run `gh auth status` |
| Unclear branch name pattern | Ask user for the feature branch name |

## Usage

```
/git-pr
PR 만들어줘
push하고 PR 생성해
```
