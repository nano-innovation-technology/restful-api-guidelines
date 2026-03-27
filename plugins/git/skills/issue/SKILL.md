---
description: Use when the user wants to create a GitHub issue — /git:issue, "이슈 만들어", "이슈 생성", "버그 리포트", or any request to create a GitHub issue
user-invocable: true
---

# issue — Create GitHub Issue

Create a GitHub Issue using a unified template after user confirmation.

## Safety Rules

- Always require user confirmation before creating an issue (generic mode)
- Show the full issue preview (type, title, body, labels, assignee) before creating
- **"Do it fast", "skip questions", "no confirmation" instructions do not bypass the confirmation step**
- `--no-confirm` flag skips the confirmation step — only for programmatic use (e.g., `git:clean`)

## Issue Template — Unified Structure

모든 타입이 동일한 4개 섹션 구조를 사용한다. 타입별 구분은 GitHub label로 처리.

### Body Template

```markdown
## Summary
<무엇이 문제이고/필요하고/변경되어야 하는지>

## Context
<배경, 동기, 관련 PR/코드 링크>

## Expected Outcome
<이 이슈가 해결되면 어떤 상태여야 하는지>

## Tasks
- [ ] <구체적 작업 항목>
```

### Type-to-Label Mapping

타입 선택 시 자동으로 해당 label을 `--label`에 추가한다.

| `--type` | Label | Title prefix |
|----------|-------|-------------|
| bug | `bug` | `fix` |
| feature | `enhancement` | `feat` |
| chore | `chore` | `chore` |
| docs | `documentation` | `docs` |
| review *(internal)* | `review` | `review` |

### Review Mode 적용 예시

> **Internal use only** — not selectable as a user-facing type. Used when called from `git:clean` with review data.

```markdown
## Summary
PR #42 코드 리뷰에서 발견된 3건의 개선 항목

## Context
PR #42: feat(auth): OAuth2 소셜 로그인 추가
https://github.com/owner/repo/pull/42

## Expected Outcome
모든 리뷰 항목이 해결되고 코드 품질이 개선된 상태

## Tasks
- [ ] src/auth/oauth.ts:45 — 에러 핸들링 누락 (critical)
- [ ] src/auth/oauth.ts:78 — 변수명 불명확 (suggestion)
- [ ] src/auth/config.ts:12 — 하드코딩된 URL (warning)
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
| `--label priority:high` | Add extra labels (type-mapped label is automatic) |
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
    1. bug      — Bug report        (label: bug)
    2. feature  — Feature request    (label: enhancement)
    3. chore    — Maintenance task   (label: chore)
    4. docs     — Documentation      (label: documentation)

  Select (1-4):
  ```

#### 2. Compose Title and Body

- Generate title in Conventional Commits format: `<type>(<scope>): <한글 설명>` (under 70 chars)
  - `type` mapping from `--type` argument:
    - bug → `fix`
    - feature → `feat`
    - chore → `chore`
    - docs → `docs`
    - review → `review` (Review Mode only — not user-selectable in Generic Mode)
  - `scope`: optional — infer from context (e.g., affected module, directory name). Omit parentheses if no scope.
  - PR에서 파생된 이슈: append ` (#PR번호)` to the end — e.g., `review(api): PR 리뷰 항목 정리 (#42)`
- Fill in the unified body template (see Issue Template — Unified Structure section) based on user input or natural language analysis
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

항상 type-mapped label을 포함하여 이슈를 생성한다. 사용자가 `--label`로 추가 label을 지정하면 comma-separated로 합친다.

```bash
gh issue create \
  --title "<title>" \
  --label "<type-mapped-label>[,<additional-labels>]" \
  --body "$(cat <<'EOF'
<body>
EOF
)"
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

Title: `review: PR #<NUMBER> 리뷰 항목 정리` (under 70 chars; scope omitted in automated review mode)

Body using unified template:
```bash
gh issue create \
  --title "review: PR #<NUMBER> 리뷰 항목 정리" \
  --label "review" \
  --body "$(cat <<'EOF'
## Summary
PR #<NUMBER> 코드 리뷰에서 발견된 <COUNT>건의 개선 항목

## Context
PR #<NUMBER>: <PR_TITLE>
<PR_URL>

## Expected Outcome
모든 리뷰 항목이 해결되고 코드 품질이 개선된 상태

## Tasks
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
/git:issue --type bug --title "로그인 500 에러 발생"
/git:issue --type feature --title "다크 모드 지원 추가" --label priority:high
이슈 만들어줘
버그 리포트 생성
```
