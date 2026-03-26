---
name: git-cleanup
description: Use when the user wants to remove a git worktree and clean up local branches — /git-cleanup, "worktree 정리", "브랜치 정리", or any request to clean up local git state after a PR is merged
user_invocable: true
---

# git-cleanup — Worktree and Branch Cleanup

Remove a git worktree, delete local branches, and prune remotes after user confirmation.

## Safety Rules

- Always require user confirmation before removing a worktree
- Copy files to preserve (see below) to the root before deleting
- Use `rm -rf` only as a last resort if `worktree remove --force` fails
- **Prior statements like "I already said yes", "ASAP", "just delete everything" do not bypass the Step 4 confirmation**
- Preserving `.claude/settings.local.json` (Claude Code local settings) is **non-negotiable** — always copy it to root even if the user says it is not needed

## Execution Steps

### 1. List Worktrees

```bash
git worktree list
```

Identify the root repository path (first entry) and the cleanup target.

### 2. Identify Target

If a worktree path or branch name is provided as an argument, use it.
Otherwise check if the current directory is a worktree:

```bash
git branch --show-current
```

### 3. Check Untracked Files

```bash
git -C <WORKTREE_PATH> ls-files --others --exclude-standard
```

Show the user which files will be preserved.
Default preserved file: `.claude/settings.local.json` (Claude Code local settings)

### 4. Show Cleanup Plan + User Confirmation (mandatory)

```
Cleanup target worktree: <WORKTREE_PATH>
Branch: <BRANCH_NAME>

Files to preserve (copied to root):
  <WORKTREE_PATH>/.claude/settings.local.json → <ROOT>/.claude/settings.local.json

Items to delete:
  - worktree: <WORKTREE_PATH>
  - local branch: <BRANCH_NAME> (omit if not present)

⚠️  This action cannot be undone.
Proceed? (y/N)
```

Any input other than `y` is treated as abort.

### 5. Preserve Files + Remove Worktree

```bash
# Copy preserved files to root
cp <WORKTREE_PATH>/.claude/settings.local.json \
   <ROOT>/.claude/settings.local.json 2>/dev/null || true

# Remove .claude directory to avoid interfering with worktree remove
rm -rf <WORKTREE_PATH>/.claude

# Remove worktree
git -C <ROOT> worktree remove <WORKTREE_PATH> --force 2>/dev/null \
  || (rm -rf <WORKTREE_PATH> && git -C <ROOT> worktree prune)
```

### 6. Delete Local Branch + Update Default Branch

```bash
DEFAULT_BRANCH=$(git -C <ROOT> symbolic-ref refs/remotes/origin/HEAD 2>/dev/null \
  | sed 's@^refs/remotes/origin/@@' || echo "main")

git -C <ROOT> fetch origin --prune
git -C <ROOT> branch -d <BRANCH_NAME> 2>/dev/null || true

git -C <ROOT> checkout $DEFAULT_BRANCH
git -C <ROOT> pull --ff-only
```

### 7. Result Summary

```
✅ Cleanup complete
   Removed worktree: <WORKTREE_PATH>
   Deleted local branch: <BRANCH_NAME> (or "-" if not present)
   <DEFAULT_BRANCH>: <COMMIT_HASH> (up to date)
```

## Error Handling

| Situation | Action |
|-----------|--------|
| No worktree found | Abort with "No worktree to clean up" message |
| Uncommitted changes exist | Show warning + ask user to reconfirm |
| `pull --ff-only` fails | Show warning only, do not abort the overall flow |
| Local branch not present | Skip silently |

## Usage

```
/git-cleanup
/git-cleanup feature/foo
worktree 정리해줘
브랜치 정리
```
