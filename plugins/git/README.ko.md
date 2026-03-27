# Git 워크플로우 스킬

> [English](README.md)

Claude Code를 위한 안전하고 일관된 git 워크플로우 스킬 모음입니다.

## 스킬 목록

| 스킬 | 슬래시 커맨드 | 설명 |
|------|--------------|------|
| commit | `/git:commit` | Conventional Commits 형식의 안전한 커밋 |
| pull-request | `/git:pull-request` | 브랜치 push + GitHub PR 생성 |
| review | `/git:review` | PR diff 분석 및 코드 리뷰 제출 |
| merge | `/git:merge` | PR 안전한 squash merge |
| clean | `/git:clean` | PR 전체 흐름: 커밋 → PR → 리뷰 → 머지 → 정리 |

## 안전 철학

모든 스킬이 공통으로 적용하는 안전 규칙:

- 파괴적이거나 되돌리기 어려운 작업 전 반드시 사용자 확인
- `git add -A`, `git add .` 절대 금지 (민감 파일 포함 방지)
- 명시적 요청 없이 force push 금지
- `gh pr merge`에 `--force`, `--admin`, `--auto` 플래그 사용 금지
- 민감 파일 자동 제외 (`.env`, `*.pem`, `*.key`, `credentials.json` 등)
- "빨리 해줘", "묻지 말고" 등의 지시로 확인 단계 생략 불가

## 스킬 관계

`clean`은 나머지 스킬을 순서대로 실행하는 오케스트레이터입니다:

```
clean
  ├─ Step 1: commit       (미커밋 변경사항이 있을 때)
  ├─ Step 2: pull-request (열린 PR이 없을 때)
  ├─ Step 3: review
  ├─ Step 3.5: 이슈 발행  (리뷰 항목이 있을 때)
  ├─ Step 4: merge        (항상 사용자 확인 필요)
  └─ Step 5: Cleanup      (항상 사용자 확인 필요, 인라인)
```

Step 4, 5는 auto 모드에서도 반드시 사용자 확인이 필요합니다.

## 프로젝트별 커스터마이징

스킬들은 프로젝트 컨벤션을 자동으로 감지하여 적용합니다:

- **commit** — `git log` 히스토리를 참조하여 프로젝트의 커밋 스타일에 맞춤
- **pull-request** — `main` 하드코딩 대신 `gh repo view`로 기본 브랜치 동적 감지
- **review** — PR diff에서 언어를 감지하여 해당 언어별 리뷰어 스킬을 로딩; 매칭되는 리뷰어가 없으면 일반 모범 사례 기준으로 리뷰
- **merge** — 기본 브랜치 동적 감지
- **clean** — PR 전체 흐름 오케스트레이터; worktree 제거 시 Claude Code 로컬 설정(`.claude/settings.local.json`) 자동 보존; 리뷰 항목 GitHub Issue 자동 발행

## 설치

```bash
claude plugin marketplace add https://github.com/ppzxc/engineering-guidelines.git
```
