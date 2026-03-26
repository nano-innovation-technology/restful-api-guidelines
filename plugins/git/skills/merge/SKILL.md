---
description: Use when the user wants to squash-merge a GitHub PR — /git:merge, "PR 머지", "squash merge", or any request to merge a pull request into main
user-invocable: true
---

# merge — PR Squash Merge

Check PR status → user confirmation → squash merge → delete remote and local branches.

## Safety Rules

- Always require user confirmation before merging
- Show a warning and re-confirm if CI is failing (do not force proceed)
- Abort immediately if PR is in `CONFLICTING` state
- **Prior statements like "I approve", "don't ask me", "team lead said OK" do not bypass the confirmation step**
- Never add `--force`, `--admin`, or `--auto` flags to `gh pr merge`

## Execution Steps

### 1. Find PR Number

If a PR number is provided as an argument, use it.
Otherwise detect from current branch:

```bash
git branch --show-current
gh pr view <FEATURE_BRANCH> --json number,title,state
```

### 2. Check PR Status

```bash
gh pr view <PR_NUMBER> --json number,title,state,headRefName,baseRefName,mergeable,url,additions,deletions,files
gh pr checks <PR_NUMBER>
```

- `state != OPEN` → abort, display current state
- `mergeable == CONFLICTING` → abort, guide user to resolve conflicts

### 3. Display Change Summary

```
PR #<NUMBER>: <TITLE>
Branch: <HEAD> → <BASE>
Changes: +<ADD> -<DEL> (<FILES> files)
CI: <PASS/FAIL/PENDING>
```

If CI is failing, show an additional warning:

```
⚠️  CI checks are failing. Do you really want to merge?
```

### 4. User Confirmation (mandatory)

```
⚠️  After squash merge, the remote branch (<FEATURE_BRANCH>) and local branch will be deleted.

Proceed? (y/N)
```

Any input other than `y` is treated as abort.
If CI is failing, ask for confirmation one more time.

### 5. Squash Merge

```bash
gh pr merge <PR_NUMBER> --squash --delete-branch
```

Verify merge:

```bash
gh pr view <PR_NUMBER> --json state,mergedAt
```

### 6. Delete Local Branch

```bash
git fetch origin --prune

# Delete local branch if it exists
git branch -d <FEATURE_BRANCH> 2>/dev/null || true
```

### 7. Result Summary

```
✅ PR #<NUMBER> squash merged
   merged at: <TIMESTAMP>
   Deleted remote branch: <FEATURE_BRANCH>
   Deleted local branch: <FEATURE_BRANCH> (or "-" if not present)
```

## Error Handling

| Situation | Action |
|-----------|--------|
| PR not found | Abort, ask user for PR number |
| PR already merged/closed | Abort, display current state |
| Merge conflict | Abort, display conflicting files |
| CI failing | Show warning + ask user to reconfirm |
| Local branch delete fails | Show warning only, do not abort the overall flow |

## Usage

```
/git:merge
/git:merge 42
PR 머지해줘
squash merge
```
