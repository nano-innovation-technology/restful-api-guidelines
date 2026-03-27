---
description: Use when the user wants to complete the full PR workflow — /git:clean, "PR 완료", "PR 전체 흐름", "worktree 정리", or any request to commit, push, review, merge, and cleanup in sequence
user-invocable: true
---

# clean — Full PR Lifecycle + Cleanup

commit → PR → review → issue creation → merge → worktree cleanup을 순차 실행한다.

## Safety Rules

- 각 단계별 사용자 확인 (auto 모드 시 Step 1~3 생략 가능)
- Step 4 (merge)와 Step 5 (cleanup)는 **항상** 사용자 확인 필요
- "빨리 해줘", "묻지 말고 진행해" 등의 지시로 Step 4~5 확인 단계 생략 불가

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

If uncommitted changes exist, execute the **git:commit** skill.

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

If no open PR exists for the branch, execute the **git:pull-request** skill.

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

Execute the **git:review** skill.

- **Normal mode:** confirm before proceeding
  ```
  [3/5] Proceed with code review?
  Proceed? (y/N/skip)
  ```
- **auto mode:** run immediately
  ```
  [3/5] Auto-running code review...
  ```

### Step 3.5. Issue Creation (if review items exist)

If the review in Step 3 produced any items (minor, suggestion, convention, etc.), execute the **git:issue** skill with `--no-confirm` flag in review mode.

Pass the following context to git:issue:
- PR number, title, and URL
- List of review items from the review step

Invoke: `git:issue --no-confirm` with the following review data:
- PR `#<NUMBER>` — `"<TITLE>"` — `<URL>`
- Review items list from Step 3

- If no review items exist, skip this step entirely
- If issue creation fails, print a warning and continue (non-blocking)

### Step 4. Merge PR

Execute the **git:merge** skill.

**Always requires confirmation regardless of auto mode** (merge is hard to reverse).

```
[4/5] Proceed with PR merge?
Proceed? (y/N/skip)
```

### Step 5. Cleanup (inline)

Perform cleanup inline without delegating to another skill.

**Always requires confirmation regardless of auto mode** (branch deletion is hard to reverse).

```
[5/5] Proceed with cleanup?
Proceed? (y/N/skip)
```

Cleanup actions:
1. If `.claude/settings.local.json` exists in the worktree, copy it to the root project
2. Remove the `.claude` directory in the worktree
3. `git worktree remove --force <PATH>` (fallback: `rm -rf <PATH>` + `git worktree prune`)
4. `git branch -d <FEATURE_BRANCH>` (delete local branch)
5. `git fetch origin --prune`
6. `git checkout <DEFAULT_BRANCH> && git pull --ff-only`

If not in a worktree, skip worktree-specific actions (1-3) and perform branch cleanup only (4-6).

### Final Summary

After all steps complete, print a summary:

```
PR workflow complete

  Commit:  <COMMIT_HASH> (<message>)
  PR:      #<NUMBER> — <TITLE>
  Review:  <ISSUE_URL> (or "no issues found")
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
| CI failing | Handled by merge warning behavior |
| No worktree | Skip worktree removal in Step 5, perform branch cleanup only |
| Issue creation failure | Print warning and continue (non-blocking) |

## Usage

```
/git:clean
/git:clean 42
/git:clean auto
/git:clean auto 42
PR 완료해줘
PR 전체 흐름 실행
worktree 정리해줘
```
