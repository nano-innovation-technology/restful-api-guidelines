---
name: pr-done
description: >
  Squash-merge a GitHub PR, delete the remote branch, and clean up the local
  git worktree — all in one step. Trigger when the user types /pr-done,
  mentions "PR 머지", "PR 완료", or asks to merge + cleanup a PR.
user_invocable: true
---

# pr-done — PR Review · Squash Merge · Branch & Worktree Cleanup

Performs PR review → squash merge → branch deletion → worktree cleanup in one step.

## Execution Order

### 0. Determine PR Number (auto-detect if not provided)

If a PR number is given as an argument, use it.
Otherwise, auto-detect in the following order:

```bash
# Check current branch
git branch --show-current
```

Extract the actual feature branch name from the worktree branch name (`worktree-feature/xxx` → `feature/xxx`).
- Pattern: `worktree-<BRANCH>` → `<BRANCH>`
- Example: `worktree-feature/dev-sh` → `feature/dev-sh`

```bash
# Look up the PR associated with the feature branch
gh pr view --head <FEATURE_BRANCH> --json number,title,headRefName,state
```

**If no PR exists → commit any uncommitted changes, push, and auto-create a PR (no prompt)**

```bash
# Commit any unstaged changes
git add -A
git commit -m "<type>(<scope>): <subject>"   # Follow convention from git log

# Push worktree branch as feature branch
git push origin <WORKTREE_BRANCH>:<FEATURE_BRANCH>

# Auto-create PR
gh pr create \
  --title "<title based on commit message>" \
  --body "$(cat <<'EOF'
## Summary
- <summary of changes>

## Test plan
- [ ] Build verified
EOF
)" \
  --head <FEATURE_BRANCH> \
  --base main
```

Record the PR number for use in subsequent steps.

### 1. Check PR Status

```bash
gh pr view <PR_NUMBER> --json number,title,state,headRefName,baseRefName,url,mergeable,reviews
```

- `state` must be `OPEN`. Otherwise, stop and notify the user.
- If `mergeable` is `CONFLICTING`, stop and guide the user to resolve conflicts.

### 2. PR Review (show diff summary)

```bash
gh pr view <PR_NUMBER> --json files,additions,deletions,commits
gh pr checks <PR_NUMBER>
```

Display the list of changed files, added/deleted line counts, and commit count concisely.

### 3. User Confirmation (required)

Print the following and require confirmation before proceeding:

```
PR #<NUMBER>: <TITLE>
Branch: <HEAD> → <BASE>
Changes: +<ADD> -<DEL> (<FILES> files, <COMMITS> commits)
CI: <PASS/FAIL/PENDING>

⚠️  The branch and worktree will be deleted after squash merge.
Proceed? (y/N)
```

> Any input other than `y` is treated as cancellation.

### 4. Squash Merge + Delete Branch

```bash
gh pr merge <PR_NUMBER> --squash --delete-branch --auto
```

The `--delete-branch` flag automatically deletes the remote branch.
`--auto` enables auto-merge after CI passes (merges immediately if already passed).

Verify after merge:

```bash
gh pr view <PR_NUMBER> --json state,mergedAt
```

### 5. Clean Up Local Branch

Run from the root repository (first path in `git worktree list`).

```bash
# Update main
git -C <ROOT_REPO> fetch origin
git -C <ROOT_REPO> checkout main
git -C <ROOT_REPO> pull --ff-only

# Delete local branch (use -d since it's merged)
git -C <ROOT_REPO> branch -d <FEATURE_BRANCH> 2>/dev/null || true
```

Skip if no local branch exists.

### 6. Clean Up Worktree

```bash
# List registered worktrees
git -C <ROOT_REPO> worktree list
```

Find the worktree path corresponding to `<WORKTREE_BRANCH>`.

If **untracked files** such as `settings.local.json` remain in the worktree directory,
`git worktree remove --force` may fail.
In that case, handle in the following order:

```bash
# 1) Copy any untracked files to preserve to the root
#    (settings.local.json → <ROOT_REPO>/.claude/settings.local.json)
cp <WORKTREE_PATH>/.claude/settings.local.json <ROOT_REPO>/.claude/settings.local.json 2>/dev/null || true

# 2) Force-delete the .claude directory (untracked — blocks git worktree remove)
rm -rf <WORKTREE_PATH>/.claude

# 3) Remove worktree
git -C <ROOT_REPO> worktree remove <WORKTREE_PATH> --force 2>/dev/null \
  || (rm -rf <WORKTREE_PATH> && git -C <ROOT_REPO> worktree prune)
```

```bash
# Clean up stale worktree references
git -C <ROOT_REPO> worktree prune
```

Skip if no worktree exists or it has already been removed.

### 7. Print Result Summary

```
✅ PR #<NUMBER> squash merge complete
   merged at: <TIMESTAMP>
   deleted remote branch: <FEATURE_BRANCH>
   deleted local branch: <FEATURE_BRANCH> ("-" if none)
   removed worktree: <PATH> ("-" if none)
```

---

## Error Handling

| Situation | Action |
|---|---|
| PR already merged/closed | Stop, show status |
| Merge conflict | Stop, show conflicting files |
| CI failure | Show warning, re-confirm with user (provide force option) |
| Worktree removal failure (untracked files) | Copy settings.local.json to root → rm -rf `.claude` dir → worktree remove --force → on failure: rm -rf + prune |
| gh auth error | Guide user to run `gh auth status` |

## Usage Examples

```
/pr-done          # Auto-detect/create PR for current branch and merge
/pr-done 42       # Specify PR #42
```
