---
name: git-review
description: Use when the user wants to review a GitHub PR — /git-review, "PR 리뷰", "코드 리뷰", or any request to review and submit feedback on a pull request
user_invocable: true
---

# git-review — PR Code Review

Analyze PR diff → detect languages → load language-specific reviewer skills → display review results → submit via `gh pr review` after user confirmation.

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

### 3. Detect Languages

Use the `files` field already retrieved in Step 2 (no additional API call needed).

Inspect the changed file extensions and map to languages:

| Extension | Language |
|-----------|----------|
| `.go` | go |
| `.java` | java |
| `.ts`, `.tsx` | typescript |
| `.js`, `.jsx` | javascript |
| `.py` | python |
| `.rs` | rust |

Collect the unique set of detected languages.

### 4. Load Reviewer Skills

From the list of skills available in the system (shown in session context), find skills whose name contains `reviewer`.

For each detected language, attempt to match a reviewer skill:
- `go` → look for a skill containing `golang` or `go` and `reviewer` (e.g., `golang:reviewer`)
- `java` → look for a skill containing `java` and `reviewer` (e.g., `java-reviewer:java-reviewer`)
- Other languages → look for skill containing the language name and `reviewer`

For each matched reviewer skill, invoke it using the `Skill` tool to load its review criteria into context.

If no reviewer skill matches a detected language, fall back to the general review criteria for that language's files.

### 5. Code Review

```bash
gh pr diff <PR_NUMBER>
```

Analyze the diff using a two-tier approach:

**For files in languages with a loaded reviewer skill:**
Apply that skill's specific CHECK/FLAG criteria or checklist. Prioritize language-specific rules over general ones.

**For files in languages without a loaded reviewer skill:**
Apply the general review criteria below:

- **Bugs / Logic Errors** — Boundary conditions, nil/null handling, missing error handling
- **Security** — Missing auth/authorization, SQL injection, sensitive data exposure, input validation
- **Code Quality** — Duplicate code, unnecessary complexity, naming consistency
- **Project Conventions** — If the project has a `CLAUDE.md`, `.claude/CLAUDE.md`, or similar project-rules file, read it and apply its coding standards; otherwise review against general best practices

Combine all findings into a single unified review.

### 6. Display Review Results

```
PR #<NUMBER>: <TITLE>
Branch: <HEAD> → <BASE>
Changes: +<ADD> -<DEL> (<FILES> files)
CI: <PASS/FAIL/PENDING>
Reviewers loaded: <list of loaded reviewer skills, or "none — using general criteria">

--- Code Review ---

[APPROVE / REQUEST_CHANGES / COMMENT]

<review content>
  - <filename>:<line> — <issue description>
  - ...
```

### 7. User Confirmation (mandatory)

```
Select review type:
  1) approve          — Approve
  2) request-changes  — Request changes
  3) comment          — Comment only
  4) cancel

Choice (1-4):
```

### 8. Submit gh pr review

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
| No reviewer skill found | Use general review criteria, note it in results |

## Usage

```
/git-review
/git-review 42
PR 리뷰해줘
코드 리뷰
```
