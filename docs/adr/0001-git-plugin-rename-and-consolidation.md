# ADR-0001: Git Plugin Rename & Consolidation

## Status

Accepted

## Date

2026-03-26

## Context

현재 git 플러그인은 6개 스킬로 구성되어 있다:
`git-commit`, `git-pr`, `git-review`, `git-merge-pr`, `git-pr-done`, `git-cleanup`.

문제점:
- 이름이 불필요하게 길다 (`git-merge-pr`, `git-pr-done`, `git-cleanup`)
- 오케스트레이터(`git-pr-done`)와 cleanup(`git-cleanup`)이 분리되어 있어 스킬 수가 많다
- PR 리뷰 후 발견된 마이너/권고 항목을 추적할 방법이 없다

## Decision

### 스킬 리네이밍 및 통합

| 현재 | 변경 후 | 비고 |
|------|---------|------|
| `git-commit` | `git-commit` | 유지 |
| `git-pr` | `git-pr` | 유지 |
| `git-review` | `git-review` | 유지 |
| `git-merge-pr` | `git-merge` | 리네이밍 |
| `git-pr-done` | **삭제** | `git-clean`에 통합 |
| `git-cleanup` | `git-clean` | 오케스트레이터 역할 흡수 + 리네이밍 |

스킬 수: 6 → 5

### git-merge

기존 `git-merge-pr`과 동일한 기능. 이름만 축약.

### git-clean

`git-pr-done`(오케스트레이터)과 `git-cleanup`(worktree 정리)을 하나로 통합.

**실행 흐름:**

```
git-clean
  ├─ Step 0: Pre-flight Check
  ├─ Step 1: git-commit    (미커밋 변경사항 있을 때)
  ├─ Step 2: git-pr        (열린 PR 없을 때)
  ├─ Step 3: git-review
  ├─ Step 3.5: Issue Creation (리뷰 항목 있을 경우)
  ├─ Step 4: git-merge     (항상 확인 필요)
  └─ Step 5: Cleanup       (항상 확인 필요, 인라인 처리)
```

**인자:** `auto` (Step 1~3 확인 생략), `<PR number>` (기존 PR 사용), 조합 가능.

**Step 3.5 — Issue Creation:**
리뷰 결과에 마이너, 권고, 참고 등 어떤 항목이든 존재하면 GitHub Issue를 자동 발행한다.

- 모든 항목을 하나의 이슈에 체크리스트로 묶음
- 제목 형식: `review: PR #<NUMBER> 리뷰 항목`
- 각 항목에 severity 태그 포함 (minor, suggestion, convention 등)
- 리뷰 항목이 없으면 생략
- 이슈 발행 실패 시 경고만 출력, merge를 blocking하지 않음

**Step 5 — Cleanup (인라인):**
별도 스킬 위임 없이 인라인 처리:
1. `.claude/settings.local.json` 보존 (worktree → 루트로 복사)
2. `.claude` 디렉토리 제거
3. `git worktree remove --force` (실패 시 `rm -rf` + prune)
4. 로컬 브랜치 삭제
5. `git fetch origin --prune`
6. default branch checkout + `pull --ff-only`

## Consequences

### 긍정적
- 스킬 수 감소 (6→5)로 사용자 인지 부하 감소
- 이름이 짧아져 슬래시 커맨드 입력이 편해짐
- PR 리뷰 항목이 GitHub Issue로 자동 추적됨
- `/git-clean` 하나로 전체 PR 마무리 가능

### 부정적
- `git-clean`이 오케스트레이션 + 정리 두 가지 책임을 가짐
- `git-cleanup` 단독 실행 (worktree 정리만) 불가 — 사용 케이스 없음 확인 완료
- 기존 `/git-pr-done`, `/git-merge-pr`, `/git-cleanup` 사용자는 새 커맨드에 적응 필요
