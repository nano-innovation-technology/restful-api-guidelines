---
name: git-pr-done
description: Use when the user wants to complete the full PR workflow in one command — /git-pr-done, "PR 완료", "PR 전체 흐름", or any request to commit, push, review, merge, and cleanup in sequence
user_invocable: true
---

# git-pr-done — Full PR Lifecycle Orchestration

Execute the complete PR workflow — commit → PR → review → merge → cleanup — with per-step user confirmation.

Each step delegates to its dedicated skill. This skill acts as the orchestrator.

## Safety Rules

- Confirm with the user at each step boundary (skippable in auto mode for Steps 1–3)
- If the user aborts at any step, stop immediately
- If a step fails, do not proceed to the next step
- Step 4 (merge) and Step 5 (cleanup) **always** require user confirmation — these actions are hard to reverse

## Argument Parsing

Parse arguments before executing any step:

| Argument | Description |
|----------|-------------|
| `auto` | Run Steps 1–3 without confirmation. Steps 4+ always require confirmation. |
| `<PR number>` | Use an existing PR (skip Step 2) |
| `auto <PR number>` | auto mode + existing PR number |

**auto mode is active when:** the `auto` argument is present

## Execution Steps

### Step 0. Pre-flight Check

```bash
git status
git branch --show-current
gh pr view <FEATURE_BRANCH> --json number,state 2>/dev/null
```

Assess current state and determine which steps are needed.

### Step 1. Commit (if uncommitted changes exist)

If uncommitted changes exist, execute the **git-commit** skill.

Skip this step if there are no changes.

- **Normal mode:** confirm before proceeding
  ```
  [1/5] Proceed with commit step? (uncommitted changes: <N>)
  Proceed? (y/N/skip)
  ```
- **auto mode:** run immediately, only print status
  ```
  [1/5] Auto-running commit step... (uncommitted changes: <N>)
  ```

### Step 2. Create PR (if no open PR exists)

If no open PR exists for the branch, execute the **git-pr** skill.

Skip this step if a PR already exists.

- **Normal mode:** confirm before proceeding
  ```
  [2/5] Proceed with PR creation?
  Proceed? (y/N/skip)
  ```
- **auto mode:** run immediately
  ```
  [2/5] Auto-running PR creation...
  ```

### Step 3. PR Review

Execute the **git-review** skill.

- **Normal mode:** confirm before proceeding
  ```
  [3/5] Proceed with code review?
  Proceed? (y/N/skip)
  ```
- **auto mode:** run immediately
  ```
  [3/5] Auto-running code review...
  ```

### Step 4. Merge PR

Execute the **git-merge-pr** skill.

**Always requires confirmation regardless of auto mode** (merge is hard to reverse).

```
[4/5] Proceed with PR merge?
Proceed? (y/N/skip)
```

### Step 5. Cleanup

Execute the **git-cleanup** skill.

**Always requires confirmation regardless of auto mode** (branch deletion is hard to reverse).

```
[5/5] Proceed with worktree cleanup?
Proceed? (y/N/skip)
```

### Final Summary

After all steps complete, print a summary:

```
✅ PR workflow complete

  Commit:  <COMMIT_HASH> (<message>)
  PR:      #<NUMBER> — <TITLE>
  Merged:  squash merge at <TIMESTAMP>
  Cleanup: worktree <PATH> removed, branch <BRANCH> deleted
```

## Step Skip Behavior

- `skip` input: skip the current step and proceed to the next
- `n` or empty input: abort the entire workflow

## Error Handling

| Situation | Action |
|-----------|--------|
| Error in any step | Display error and ask user whether to continue |
| CI failing | Handled by git-merge-pr warning behavior |
| No worktree | Notify user that git-cleanup will be skipped |

## Usage

```
/git-pr-done              # confirm at each step
/git-pr-done 42           # use existing PR #42, confirm at each step
/git-pr-done auto         # auto Steps 1–3, confirm from Step 4
/git-pr-done auto 42      # auto mode with existing PR #42
PR 완료해줘
PR 전체 흐름 실행
```
