---
description: "Use when creating a Markdown Architectural Decision Record — /docs:madr, \"MADR 작성\", \"결정 사항 기록\", or any request to document an architecture decision in MADR 3.x format"
user-invocable: true
---

# docs:madr — Markdown Architectural Decision Record

MADR 3.x 포맷으로 아키텍처 결정 사항을 `docs/decisions/` 에 기록한다.

## MADR Variant

`variant=` 인자로 템플릿을 선택한다. 미지정 시 컨텍스트 기반 자동 선택.

| Variant | 사용 상황 | 자동 선택 조건 |
|---------|-----------|---------------|
| `minimal` | 빠른 기록, 컨텍스트 부족 | 컨텍스트가 매우 단순할 때 |
| `standard` | 단일 의사결정이 명확 | **기본값** (미지정 시) |
| `full` | 옵션 간 비교가 중요 | 소스 문서에 옵션 비교가 명확히 존재할 때 |

## 포맷 — Minimal

```markdown
# Title

## Status

accepted

## Context and Problem Statement

(왜 이 결정이 필요한가?)

## Decision Outcome

(무엇을 선택했는가, 이유는?)
```

## 포맷 — Standard (기본값)

```markdown
# Title

## Status

accepted

## Context and Problem Statement

(왜 이 결정이 필요한가?)

## Decision Drivers

* (드라이버 1)
* (드라이버 2)

## Considered Options

* Option A
* Option B

## Decision Outcome

Chosen option: "Option A", because ...

### Consequences

* Good: ...
* Bad: ...
```

## 포맷 — Full

```markdown
# Title

## Status

accepted

## Context and Problem Statement

(왜 이 결정이 필요한가?)

## Decision Drivers

* (드라이버 1)

## Considered Options

* Option A
* Option B

## Decision Outcome

Chosen option: "Option A", because ...

### Consequences

* Good: ...
* Bad: ...

## Pros and Cons of the Options

### Option A

* Good: ...
* Bad: ...

### Option B

* Good: ...
* Bad: ...
```

## 워크플로우

### 1. Variant 결정

- `variant=minimal` / `variant=standard` / `variant=full` 인자가 있으면 그대로 사용
- 없으면 아래 규칙으로 자동 선택:
  - `path=` 소스 문서가 있고 옵션 비교가 명확히 포함되어 있으면 → `full`
  - 단일 의사결정이 명확하거나 컨텍스트가 충분하면 → `standard`
  - 컨텍스트가 매우 단순하거나 빠른 기록이 필요하면 → `minimal`

### 2. 소스 문서 확인

`path=` 인자가 있으면 해당 파일을 읽어 각 섹션 작성에 활용한다.

- `full` variant: `Considered Options`와 `Pros and Cons` 섹션을 소스 문서의 옵션 비교에서 자동 추출
- `standard` variant: `Decision Drivers`와 `Considered Options`를 소스 문서에서 추출

### 3. 제목 결정

- 슬래시 커맨드 인자로 제목이 주어지면 그대로 사용
- 없으면 사용자에게 질문: "MADR 제목을 입력해 주세요 (예: Use Kafka for event streaming)"

### 4. 번호 채번

`docs/decisions/` 디렉토리를 스캔하여 기존 파일의 최대 번호 + 1을 자동 할당한다.

```bash
ls docs/decisions/ 2>/dev/null | grep -E '^[0-9]{4}-' | sort | tail -1
```

- 디렉토리가 없거나 파일이 없으면 `0001`부터 시작
- 번호 충돌이 감지되면 사용자에게 확인 후 진행

### 5. 파일명 생성

`NNNN-<kebab-case-title>.md` 형식 (4자리 제로패딩)

예: `0001-use-kafka-for-event-streaming.md`

### 6. 초안 작성

선택된 variant 템플릿에 따라 초안을 작성한다.

- `path=` 소스 문서가 있으면 → 해당 문서에서 컨텍스트 추출
- 없으면 → 현재 대화 컨텍스트에서 추론

### 7. 사용자 확인

초안과 선택된 variant를 보여주고 확인을 요청한다:

```
저장 경로: docs/decisions/0001-use-kafka-for-event-streaming.md
Variant: standard (자동 선택)

[MADR 초안 내용]

저장할까요? (y/N)
```

`y` 이외의 입력은 중단으로 처리한다.

### 8. 파일 저장

`docs/decisions/` 디렉토리가 없으면 생성 후 저장한다.

## 사용 예시

```
/docs:madr
/docs:madr "Use Kafka for event streaming"
/docs:madr "Use Kafka" variant=full
/docs:madr path=docs/superpowers/specs/2026-03-26-foo-design.md
/docs:madr path=docs/superpowers/specs/2026-03-26-foo-design.md variant=minimal
/docs:madr "Use Kafka" path=docs/superpowers/specs/2026-03-26-foo-design.md
```
