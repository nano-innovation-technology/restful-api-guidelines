---
description: Use when the user wants to create a GitHub issue — /git:issue, "이슈 만들어", "이슈 생성", "버그 리포트", or any request to create a GitHub issue
user-invocable: true
---

# issue — Create GitHub Issue

Create a GitHub Issue with type-specific templates after user confirmation.

## Safety Rules

- Always require user confirmation before creating an issue (generic mode)
- Show the full issue preview (type, title, body, labels, assignee) before creating
- **"Do it fast", "skip questions", "no confirmation" instructions do not bypass the confirmation step**
- `--no-confirm` flag skips the confirmation step — only for programmatic use (e.g., `git:clean`)

## Issue Types & Templates

### bug

```markdown
## Summary
<bug summary>

## Steps to Reproduce
1. ...
2. ...

## Expected Behavior
<expected>

## Actual Behavior
<actual>
```

### feature

```markdown
## Summary
<feature summary>

## Motivation
<why this is needed>

## Proposal
<implementation suggestion>

## Acceptance Criteria
- [ ] ...
```

### chore

```markdown
## Summary
<task summary>

## Tasks
- [ ] ...
```

### docs

```markdown
## Summary
<documentation change summary>

## Scope
<target documents/areas>
```

### review

> **Internal use only** — not selectable as a user-facing type. Used when called from `git:clean` with review data.

```markdown
## Source
PR #<NUMBER>: <PR_TITLE>
<PR_URL>

## Items
- [ ] <file>:<line> — <description> (<severity>)
```

## Argument Parsing

Parse arguments before executing:

| Argument | Description |
|----------|-------------|
| (none) | Interactive — ask user for issue type and content |
| `free text` | Analyze natural language to auto-compose type/title/body/labels |
| `--title "title"` | Set title directly |
| `--type <type>` | Set issue type — one of: `bug`, `feature`, `chore`, `docs` |
| `--no-confirm` | Skip confirmation step (programmatic use only — e.g., from git:clean) |
| `--label bug,enhancement` | Set labels |
| `--assignee @me` | Set assignee |

## Execution Steps

### Mode Detection

If `--no-confirm` flag is present and review data is provided in context, use **Review Mode**. Otherwise, use **Generic Mode**.

### Generic Mode

#### 1. Parse Input

Analyze arguments:
- If `--type review` is provided without `--no-confirm`, reject with: `review type is for internal use only — use /git:issue without --type to create a regular issue`
- If explicit `--type` provided (bug, feature, chore, docs), use it
- If natural language provided, infer type from content (e.g., "500 에러 발생" → bug)
- If no arguments, ask user to select type:
  ```
  Issue type:
    1. bug      — Bug report
    2. feature   — Feature request
    3. chore     — Maintenance task
    4. docs      — Documentation change

  Select (1-4):
  ```

#### 2. Compose Title and Body

- Generate title in format: `<type>: <description>` (under 70 chars)
- Fill in the type-specific template based on user input or natural language analysis
- If explicit `--title` provided, use it as-is

#### 3. User Confirmation (mandatory)

Display the following and wait for confirmation:

```
Type:     <type>
Title:    <title>
Labels:   <labels or "(none)">
Assignee: <assignee or "(none)">

Body:
---
<body preview>
---

Proceed? (y/N)
```

Any input other than `y` is treated as abort.

#### 4. Create Issue

```bash
gh issue create \
  --title "<title>" \
  --body "$(cat <<'EOF'
<body>
EOF
)"
```

If labels specified:
```bash
gh issue create \
  --title "<title>" \
  --body "$(cat <<'EOF'
<body>
EOF
)" \
  --label "<label1>,<label2>"
```

If assignee specified, add `--assignee "<assignee>"`.

#### 5. Output Result

Print the created issue number and URL.

### Review Mode (called from git:clean)

#### 1. Receive Review Data

Receive from git:clean context:
- PR number, title, URL
- List of review items: `<file>:<line> — <description> (<severity>)`

#### 2. Compose Issue

Title: `review: PR #<NUMBER> review items`

Body using review template:
```bash
gh issue create \
  --title "review: PR #<NUMBER> review items" \
  --body "$(cat <<'EOF'
## Source
PR #<NUMBER>: <PR_TITLE>
<PR_URL>

## Items
- [ ] <file>:<line> — <description> (<severity>)
- [ ] ...
EOF
)"
```

#### 3. Return Result

- Print created issue number and URL
- If creation fails, print warning and return (non-blocking)

## Error Handling

| Situation | Action |
|-----------|--------|
| `gh` auth failure | Guide user to run `gh auth status` |
| Issue creation fails | Show error, suggest retry |
| Label does not exist | Warn and ask user: proceed without label or correct it |
| Review mode failure | Print warning and continue (non-blocking) |

## Usage

```
/git:issue
/git:issue 로그인 페이지에서 500 에러 발생
/git:issue --type bug --title "Login 500 error"
/git:issue --type feature --title "Add dark mode" --label enhancement
이슈 만들어줘
버그 리포트 생성
```
