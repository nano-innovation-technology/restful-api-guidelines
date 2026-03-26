---
name: git-pr-review
description: Use when the user wants to review a GitHub PR — /git-pr-review, "PR 리뷰", "코드 리뷰", or any request to review and submit feedback on a pull request
user_invocable: true
---

# git-pr-review — PR Code Review

Analyze PR diff → display review results → submit via `gh pr review` after user confirmation.

## Execution Steps

### 1. Find PR Number

If a PR number is provided as an argument, use it.
Otherwise detect from current branch:

```bash
git branch --show-current
gh pr view <FEATURE_BRANCH> --json number,title,state
```

If no PR is found, ask the user for the PR number directly.

### 2. Fetch PR Info

```bash
gh pr view <PR_NUMBER> --json number,title,state,headRefName,baseRefName,additions,deletions,files,commits,url
gh pr checks <PR_NUMBER>
```

### 3. Code Review

```bash
gh pr diff <PR_NUMBER>
```

Analyze the diff against the following criteria:

**Bugs / Logic Errors**
- Boundary conditions, nil/null handling, missing error handling

**Security**
- Missing auth/authorization, SQL injection, sensitive data exposure, input validation

**Code Quality**
- Duplicate code, unnecessary complexity, naming consistency

**Project Conventions**
- If the project has a `CLAUDE.md`, `.claude/CLAUDE.md`, or similar project-rules file, read it and apply its coding standards to the review
- If no project-rules file exists, review against general best practices

### 4. Display Review Results

```
PR #<NUMBER>: <TITLE>
Branch: <HEAD> → <BASE>
Changes: +<ADD> -<DEL> (<FILES> files)
CI: <PASS/FAIL/PENDING>

--- Code Review ---

[APPROVE / REQUEST_CHANGES / COMMENT]

<review content>
  - <filename>:<line> — <issue description>
  - ...
```

### 5. User Confirmation (mandatory)

```
Select review type:
  1) approve          — Approve
  2) request-changes  — Request changes
  3) comment          — Comment only
  4) cancel

Choice (1-4):
```

### 6. Submit gh pr review

```bash
gh pr review <PR_NUMBER> \
  --approve          # or --request-changes or --comment
  --body "<review content>"
```

## Error Handling

| Situation | Action |
|-----------|--------|
| PR not found | Abort, ask user for PR number |
| PR already merged/closed | Abort, display current state |
| CI failing | Include CI failure details in review |
| Empty diff | Display "No changes found" and abort |

## Usage

```
/git-pr-review
/git-pr-review 42
PR 리뷰해줘
코드 리뷰
```
