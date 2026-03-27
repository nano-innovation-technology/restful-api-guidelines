# RESTful API Guidelines Skill 성능 평가 설계

**날짜:** 2026-03-19
**목적:** Claude Code skill(restful-api-guidelines.md)이 README 가이드라인을 얼마나 잘 커버하는지 평가하고, 개선 우선순위를 도출한다.

---

## 배경

`/home/ppzxc/projects/restful-api-guidelines/.claude/skills/restful-api-guidelines.md` 스킬은 Claude Code가 API 코드를 작성하거나 리뷰할 때 자동으로 활성화된다. 이 스킬에는 두 가지 모드가 있다:

- **Code Writing Mode** — API 코드를 생성할 때 따라야 할 규칙 및 코드 예시
- **Code Review Mode** — 기존 코드를 리뷰할 때 사용하는 체크리스트 및 위반 보고 형식

이 두 모드가 `README.md`의 모든 규칙을 올바르게 반영하는지 체계적으로 평가한다.

### 사전 파악된 주요 누락 후보

스펙 작성 시점에 파악된 잠재적 누락 항목 (커버리지 맵에서 확인 필요):

- README 2.1: URL 길이 제한 (2000자 이하)
- README 2.3: 상태 변경 작업에 쿼리 파라미터 사용 금지
- README 3.2: 요청 본문의 읽기 전용 필드 무시 규칙
- README 4.1: 필드명 약어 금지 규칙
- README 4.2: 큰 정수(2^53 초과) 문자열 반환
- README 5.4: 하위 호환/비호환 변경 분류 기준
- README 6.1: API Key를 쿼리 파라미터로 전달 금지

---

## 목표

1. README ↔ 스킬 간 규칙 커버리지를 정량화한다.
2. 테스트 케이스(bad/good 코드 쌍)로 스킬의 탐지/생성 능력을 검증한다.
3. Code Writing Mode와 Code Review Mode를 각각 평가한다.
4. 문제를 Critical/Minor로 분류하고 개선 권고사항을 제시한다.

---

## 산출물

| 파일 | 설명 |
|------|------|
| `docs/evaluation/coverage-map.md` | README 규칙 ↔ 스킬 커버 여부 매핑 테이블 |
| `docs/evaluation/test-cases.md` | 섹션별 bad/good 코드 테스트 케이스 (~50개) |
| `docs/evaluation/report.md` | 커버리지 수치, Critical/Minor 문제 목록, 개선 권고사항 |

---

## 커버리지 매핑 방법론

### 규범 수준 분류

| 기호 | 수준 |
|------|------|
| ✅ 필수 | MUST / DO |
| ⚠️ 권장 | SHOULD / MAY |
| ❌ 금지 | MUST NOT / DO NOT |

### 스킬 커버 상태 판정 기준

| 상태 | 판정 기준 |
|------|----------|
| `COVERED` | 규칙의 핵심 요건이 스킬에 명시적 문장 또는 코드 예시로 표현됨 |
| `PARTIAL` | 규칙이 언급되지만 반례(bad case), 예외 조건, 또는 적용 범위가 누락됨 (bad case는 위반 패턴이 존재하는 규칙에만 요구됨) |
| `MISSING` | 스킬에서 해당 규칙을 찾을 수 없음 |

### 스킬 모드별 커버리지 분리

커버리지 맵은 두 모드를 분리하여 평가한다:

- **Writing** — Code Writing Mode 섹션(코드 예시, 네이밍 규칙 등)에서 커버 여부
- **Review** — Code Review Mode 체크리스트 항목에서 커버 여부

### 심각도 분류 기준

| 심각도 | 조건 |
|--------|------|
| **Critical** | ✅필수 규칙이 Writing 또는 Review 어느 모드에서든 `MISSING` 또는 `PARTIAL` |
| **Minor** | ⚠️권장/❌금지 규칙이 누락되거나, 스킬 코드 예시가 잘못된 경우 |

---

## 테스트 케이스 구조

```
### TC-{섹션번호}-{순번}: {규칙명}

- 규칙: README 원문 인용
- 규범 수준: ✅필수 / ⚠️권장 / ❌금지
- 대상 모드: Code Writing / Code Review / Both
- 스킬 커버: COVERED / PARTIAL / MISSING
  (대상 모드가 Both인 경우 "Writing: X / Review: Y" 형식으로 분리 기재)

❌ Bad:
[코드]

✅ Good:
[코드]

- 검증 포인트: 스킬의 어느 체크리스트 항목 또는 코드 예시가 이를 처리해야 하는가
```

### 테스트 케이스 범위

| 섹션 | 예상 케이스 수 |
|------|--------------|
| 2. HTTP 기본 규칙 (URL, 메서드, 상태코드, 헤더) | ~12개 |
| 3. REST 원칙 (스키마, 필드, 에러처리) | ~8개 |
| 4. JSON 규칙 (네이밍, 타입, 날짜, Enum) | ~10개 |
| 5.1 액션 수행 | ~2개 |
| 5.2 컬렉션/페이지네이션 | ~4개 |
| 5.3 필터링/정렬 | ~3개 |
| 5.4 API 버전 관리 | ~2개 |
| 5.5 Deprecation | ~2개 |
| 5.6 속도 제한 | ~3개 |
| 5.7 장기 실행 작업 | ~2개 |
| 6. 인증/보안 (Bearer, 401/403, Idempotency) | ~6개 |
| **합계** | **~54개** |

---

## 평가 보고서 구조

```markdown
## 커버리지 요약

| 모드 | COVERED | PARTIAL | MISSING | 합계 |
|------|---------|---------|---------|------|
| Writing | N | N | N | N |
| Review | N | N | N | N |

## Critical 문제
1. [규칙명] — 모드: Writing/Review — 이유

## Minor 문제
1. [규칙명] — 모드: Writing/Review — 이유

## 개선 권고사항
### 즉시 수정 (Critical)
### 다음 단계 (Minor)
```

---

## 평가 후 액션

| 분류 | 액션 |
|------|------|
| **Critical 문제** | 평가 완료 즉시 스킬 수정 및 PR 생성 |
| **Minor 문제** | `report.md`의 우선순위 기준으로 다음 이슈에 반영 |
| **회귀 검증** | 스킬 변경 후 동일 테스트 케이스로 재검증 (필수) |
| **report.md 업데이트** | 스킬 변경 반영 후 커버리지 수치 재계산하여 업데이트 |
