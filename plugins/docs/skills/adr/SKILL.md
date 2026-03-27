---
description: "Use when creating an Architecture Decision Record — /docs:adr, \"ADR 작성\", \"결정 사항 기록\", or any request to document an architecture decision in Nygard format"
user-invocable: true
---

# docs:adr — Architecture Decision Record

Nygard ADR 포맷으로 아키텍처 결정 사항을 `docs/adr/` 에 기록한다.

## ADR 포맷

[Nygard ADR](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) 기반 필수 섹션:

- `# Title` — 의사결정을 한 문장으로 표현 (명사형, e.g. "Use PostgreSQL for primary database")
- `## Status` — `proposed` | `accepted` | `deprecated` | `superseded by [ADR-NNNN]`
- `## Context` — 이 결정이 필요하게 된 배경과 문제 상황
- `## Decision` — 선택한 방향과 이유
- `## Consequences` — 이 결정으로 인한 결과 (긍정적/부정적 포함)

### 템플릿

```markdown
# Title

## Status

proposed

## Context

(배경과 문제 상황)

## Decision

(선택한 방향과 이유)

## Consequences

(결과 — 긍정적/부정적 모두 포함)
```

## 워크플로우

### 1. 소스 문서 확인

`path=` 인자가 있으면 해당 파일을 읽어 Context, Decision, Consequences 섹션 작성에 활용한다.

```
/docs:adr path=docs/superpowers/specs/2026-03-26-foo-design.md
```

### 2. 제목 결정

- 슬래시 커맨드 인자로 제목이 주어지면 그대로 사용
- 없으면 사용자에게 질문: "ADR 제목을 입력해 주세요 (예: Use PostgreSQL for primary database)"

### 3. 번호 채번

`docs/adr/` 디렉토리를 스캔하여 기존 파일의 최대 번호 + 1을 자동 할당한다.

```bash
ls docs/adr/ 2>/dev/null | grep -E '^[0-9]{4}-' | sort | tail -1
```

- 디렉토리가 없거나 파일이 없으면 `0001`부터 시작
- 번호 충돌이 감지되면 사용자에게 확인 후 진행

### 4. 파일명 생성

`NNNN-<kebab-case-title>.md` 형식 (4자리 제로패딩)

예: `0001-use-postgresql-for-primary-database.md`

### 5. 초안 작성

소스 문서 또는 현재 대화 컨텍스트를 기반으로 ADR 초안을 작성한다.

- `path=` 소스 문서가 있으면 → 해당 문서의 배경, 결정 사항, 트레이드오프를 추출
- 없으면 → 현재 대화 컨텍스트에서 추론

### 6. 사용자 확인

초안을 보여주고 확인을 요청한다:

```
저장 경로: docs/adr/0001-use-postgresql-for-primary-database.md

[ADR 초안 내용]

저장할까요? (y/N)
```

`y` 이외의 입력은 중단으로 처리한다.

### 7. 파일 저장

`docs/adr/` 디렉토리가 없으면 생성 후 저장한다.

## 사용 예시

```
/docs:adr
/docs:adr "Use PostgreSQL for primary database"
/docs:adr path=docs/superpowers/specs/2026-03-26-foo-design.md
/docs:adr "Use PostgreSQL" path=docs/superpowers/specs/2026-03-26-foo-design.md
```
