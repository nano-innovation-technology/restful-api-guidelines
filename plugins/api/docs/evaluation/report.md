# RESTful API Guidelines Skill 성능 평가 보고서

**평가 날짜:** 2026-03-20
**평가 대상:** `.claude/skills/restful-api-guidelines.md`
**기준 문서:** `README.md`
**테스트 케이스:** `docs/evaluation/test-cases.md` (총 54개)

---

## 커버리지 요약

| 모드 | COVERED | PARTIAL | MISSING | 합계 | 커버율 |
|------|---------|---------|---------|------|--------|
| Writing | 68 | 0 | 3 | 71 | 95.8% |
| Review | 68 | 0 | 3 | 71 | 95.8% |

> 커버율 = (COVERED + PARTIAL×0.5) / 합계 × 100
>
> **수정 전 (2026-03-20 초기 평가):** Writing 59.9% / Review 62.0%
> **수정 후 (Critical 반영):** Writing 73.2% / Review 81.7%
> **수정 후 (Minor 반영):** Writing 95.8% / Review 95.8%

---

## Critical 문제 (즉시 수정 대상)

> ✅필수 규칙이 Writing 또는 Review 모드에서 MISSING 또는 PARTIAL — 총 17개 → **모두 해결됨 (Task 5)**

### Writing Mode Critical 문제

| # | 규칙 요약 | 상태 | 누락 내용 | 스킬 추가 필요 사항 |
|---|-----------|------|-----------|---------------------|
| 2.1-5 | URL 경로 세그먼트에 ASCII 영소문자/숫자/하이픈만 허용 | PARTIAL | kebab-case 언급은 있으나 허용 문자 집합(ASCII 영소문자, 숫자, 하이픈만)을 명시적으로 제한하지 않음 | Writing 가이드에 "URL path segments MUST contain only ASCII lowercase letters (a-z), digits (0-9), and hyphens (-)" 규칙 추가 |
| 2.2-1 | GET 요청은 서버 상태 변경 안 함 | MISSING | Writing 모드에 HTTP 메서드 안전성(safety) 규칙이 전혀 없음 | "GET and HEAD requests MUST NOT modify server state (safe methods)" 규칙 추가 |
| 2.2-2 | PUT 요청은 멱등적으로 동작 | PARTIAL | 상태코드 표에 "Full replacement" 언급만 있고, 멱등성(동일 요청 반복 시 동일 결과) 규칙이 명시되지 않음 | "PUT and DELETE MUST be idempotent — repeated identical requests produce the same result" 규칙 추가 |
| 2.3-1 | 동일 파라미터 반복으로 배열 값 전달 | MISSING | 배열 값 전달 시 동일 파라미터 반복 방식(?status=A&status=B) 패턴이 없음 | "Array values in query parameters MUST use repeated parameter names: `?status=ACTIVE&status=DRAFT`" 규칙 추가 |
| 2.4-1 | 요청 본문 있을 때 Content-Type 헤더 포함 | PARTIAL | 에러 응답의 Content-Type(`application/problem+json`)만 언급, 일반 요청/응답의 Content-Type 필수 규칙 없음 | "Requests with a body MUST include Content-Type header (typically `application/json`)" 규칙 추가 |
| 2.4-2 | 응답 본문 있을 때 Content-Type 헤더 포함 | PARTIAL | 에러 응답 Content-Type만 명시, 정상 응답의 Content-Type 규칙이 코드 예시에서만 암시적 | "Responses with a body MUST include Content-Type header" 규칙 추가 |
| 3.2-1 | 서버 관리 읽기 전용 필드를 요청 본문에 포함해도 무시 | PARTIAL | "not modifiable by client" 언급은 있으나 "클라이언트가 보내더라도 서버가 무시" 동작이 명확히 서술되지 않음 | "Server MUST silently ignore read-only fields (id, createdAt, updatedAt) if included in request body" 규칙 추가 |
| 3.3-2 | PUT은 리소스 전체 대체, 미포함 필드는 기본값/null | PARTIAL | "Full replacement" 언급만 있고 미포함 필드 처리 규칙이 누락됨 | "PUT replaces the entire resource — omitted fields revert to default values or null" 규칙 추가 |
| 4.1-2 | 필드 이름은 영소문자로 시작 | PARTIAL | camelCase 예시가 모두 소문자 시작이지만 "영소문자로 시작" 규칙이 명시되지 않음 | "Field names MUST start with a lowercase ASCII letter (a-z)" 규칙을 camelCase 규칙과 함께 명시 |
| 4.2-1 | Boolean은 JSON true/false 사용 | PARTIAL | `"isActive": true` 예시만 있고, 문자열 "true"/"false"나 숫자 1/0 사용 금지가 명시되지 않음 | "Boolean values MUST use JSON native `true`/`false` — string `\"true\"`/`\"false\"` or numeric `1`/`0` are prohibited" 추가 |
| 4.2-3 | 숫자 값은 JSON number 타입 | MISSING | 숫자 타입 규칙 자체가 스킬에 없음 | "Numeric values MUST use JSON number type, not strings (e.g., `\"age\": 25` not `\"age\": \"25\"`)" 규칙 추가 |
| 5.3-1 | 동일 파라미터 반복은 OR 조건 | MISSING | Writing 모드에 필터링 시 OR/AND 조건 규칙이 없음 | "Repeated query parameters are treated as OR condition: `?status=ACTIVE&status=DRAFT` means status is ACTIVE OR DRAFT" 규칙 추가 |
| 5.4-3 | 동일 버전 내 하위 호환성 유지 | MISSING | 하위 호환성 유지 규칙 및 호환/비호환 변경 목록이 없음 | "Within the same API version, all changes MUST be backward-compatible (no field removal, no type change, no required field addition)" 규칙 추가 |
| 6.3-2 | Idempotency-Key 값은 클라이언트 생성 UUID v4 | PARTIAL | 코드 예시의 키 값이 UUID 형태이나 "UUID v4" 사양이 명시되지 않음 | "Idempotency-Key value MUST be a client-generated UUID v4" 명시 추가 |

### Review Mode Critical 문제

| # | 규칙 요약 | 상태 | 누락 내용 | 스킬 추가 필요 사항 |
|---|-----------|------|-----------|---------------------|
| 2.1-5 | URL 경로 세그먼트에 ASCII 영소문자/숫자/하이픈만 허용 | PARTIAL | kebab-case 체크는 있으나 허용 문자 집합 검증 항목이 없음 | "Path segments contain only a-z, 0-9, hyphens" 체크리스트 항목 추가 |
| 2.2-2 | PUT 요청은 멱등적으로 동작 | MISSING | PUT 멱등성 체크 항목이 없음 | "PUT requests are idempotent (repeated calls produce same result)" 체크 항목 추가 |
| 2.3-1 | 동일 파라미터 반복으로 배열 값 전달 | PARTIAL | OR 조건 체크는 있으나 인코딩 방식(반복 파라미터) 자체의 검증이 간접적 | "Array query parameters use repeated keys (not comma-separated)" 체크 항목 추가 |
| 2.4-1 | 요청 본문 있을 때 Content-Type 헤더 포함 | MISSING | 요청 Content-Type 체크 항목이 없음 | "Requests with body include Content-Type header" 체크 항목 추가 |
| 2.4-2 | 응답 본문 있을 때 Content-Type 헤더 포함 | MISSING | 응답 Content-Type 일반 규칙 체크 항목이 없음 | "Responses with body include Content-Type header" 체크 항목 추가 |
| 3.1-1 | 모든 리소스는 고유 id 가짐 | MISSING | 리소스 id 필수 체크 항목이 없음 | "Every resource has a unique `id` field" 체크 항목 추가 |
| 3.1-2 | 리소스 스키마는 일관된 구조 유지 (id/createdAt/updatedAt) | MISSING | 표준 필드 구조 검증 항목이 없음 | "Resource includes standard fields: id, createdAt, updatedAt" 체크 항목 추가 |
| 3.2-1 | 서버 관리 읽기 전용 필드를 요청 본문에 포함해도 무시 | MISSING | 읽기 전용 필드 무시 동작 체크가 없음 | "Server-managed fields (id, createdAt, updatedAt) ignored if sent in request body" 체크 항목 추가 |
| 3.3-2 | PUT은 리소스 전체 대체, 미포함 필드는 기본값/null | MISSING | PUT 전체 대체 및 미포함 필드 처리 체크가 없음 | "PUT replaces entire resource; omitted fields reset to defaults" 체크 항목 추가 |
| 4.1-2 | 필드 이름은 영소문자로 시작 | MISSING | 영소문자 시작 체크 항목이 없음 | "Field names start with lowercase letter" 체크 항목 추가 |
| 4.2-1 | Boolean은 JSON true/false 사용 | MISSING | Boolean JSON true/false 체크 항목이 없음 | "Boolean values use JSON true/false (not string or numeric)" 체크 항목 추가 |
| 4.2-3 | 숫자 값은 JSON number 타입 | MISSING | 숫자 JSON number 타입 체크가 없음 | "Numeric values use JSON number type (not strings)" 체크 항목 추가 |
| 5.4-3 | 동일 버전 내 하위 호환성 유지 | MISSING | 하위 호환성 유지 체크가 없음 | "Changes within same version are backward-compatible" 체크 항목 추가 |
| 5.5-1 | Deprecated API에 Deprecation/Sunset/Link 응답 헤더 제공 | MISSING | Deprecation 헤더 검증 항목이 없음 | "Deprecated endpoints include Deprecation, Sunset, Link headers" 체크 항목 추가 |
| 6.3-2 | Idempotency-Key 값은 클라이언트 생성 UUID v4 | MISSING | UUID v4 명시 체크 항목이 없음 | "Idempotency-Key value is UUID v4" 체크 항목 추가 |

---

## Minor 문제 (단계적 수정)

> ⚠️권장/❌금지 규칙 중 하나 이상의 모드에서 MISSING인 항목 — 총 22개 → **19개 해결됨 (Minor 개선)**, 3개 Tier C 보류

| # | 규범 수준 | 규칙 요약 | Writing | Review | 비고 |
|---|-----------|-----------|---------|--------|------|
| 2.1-3 | ❌금지 | URL에 동사 포함 금지 | COVERED | COVERED | ✅ bad case 및 금지 규칙 추가 |
| 2.1-4 | ❌금지 | URL에 파일 확장자 포함 금지 | COVERED | COVERED | ✅ 금지 규칙 추가 |
| 2.1-7 | ⚠️권장 | URL 2000자 이하 유지 | MISSING | MISSING | ⏸️ Tier C 보류 — 런타임 관심사, 코드 리뷰로 검증 어려움 |
| 2.2-3 | ⚠️권장 | 부분 수정에는 PUT 대신 PATCH 사용 | COVERED | COVERED | ✅ Review 체크리스트 추가 |
| 2.2-4 | ❌금지 | GET/HEAD/DELETE 요청에 body 포함 금지 | COVERED | COVERED | ✅ Writing 규칙 추가, Review는 기존 포함 확인 |
| 2.2-7 | ❌금지 | 오류 상황에 200 OK 반환 금지 | COVERED | COVERED | ✅ Writing 규칙 추가 |
| 2.3-2 | ⚠️권장 | 쿼리 파라미터는 선택적으로 설계 | MISSING | MISSING | ⏸️ Tier C 보류 — 추상적 설계 원칙, 코드 레벨 감지 어려움 |
| 2.3-3 | ⚠️권장 | 쿼리 파라미터에 민감한 정보 포함 금지 | COVERED | COVERED | ✅ 기존 스킬에 이미 포함됨 (coverage map 오류 수정) |
| 2.3-4 | ❌금지 | 서버 상태 변경에 쿼리 파라미터 사용 금지 | COVERED | COVERED | ✅ 기존 스킬에 이미 포함됨 (coverage map 오류 수정) |
| 2.4-3 | ⚠️권장 | 커스텀 헤더에 X- 접두사 사용 금지 (신규) | COVERED | COVERED | ✅ Writing 규칙 추가 (X-RateLimit-* 레거시 예외 주석 포함) |
| 2.4-4 | ❌금지 | 표준 HTTP 헤더 의미 재정의 금지 | COVERED | COVERED | ✅ Writing/Review 모두 추가 |
| 3.1-3 | ⚠️권장 | 리소스 식별자는 불투명한 문자열 | MISSING | MISSING | ⏸️ Tier C 보류 — 아키텍처 결정 영역, 코드 리뷰로 판단 어려움 |
| 3.1-4 | ❌금지 | 응답에 null 값 필드 포함 금지 | COVERED | COVERED | ✅ Writing 규칙 추가 및 bad case 포함 |
| 3.4-4 | ❌금지 | 에러 응답에 스택 트레이스/내부 정보 노출 금지 | COVERED | COVERED | ✅ Writing 규칙 추가 |
| 4.1-3 | ❌금지 | 필드 이름에 약어 남용 금지 | COVERED | COVERED | ✅ Writing/Review 모두 추가 |
| 4.2-4 | ⚠️권장 | 큰 정수(2^53 초과)는 문자열로 반환 | COVERED | COVERED | ✅ Writing/Review 모두 추가 |
| 4.3-5 | ❌금지 | Unix timestamp를 기본 시간 형식으로 사용 금지 | COVERED | COVERED | ✅ Review 체크리스트 추가 |
| 4.4-2 | ⚠️권장 | 클라이언트가 알 수 없는 Enum 값 수신 가능하도록 설계 | COVERED | COVERED | ✅ Writing/Review 모두 추가 |
| 4.4-3 | ❌금지 | Enum 값으로 숫자나 불명확한 약어 사용 금지 | COVERED | COVERED | ✅ bad case 추가 및 Review 체크 추가 |
| 5.4-1 | ❌금지 | API 버전을 URL 경로에 포함 금지 | COVERED | COVERED | ✅ Writing 규칙 추가 |
| 5.7-3 | ❌금지 | 범용 /operations 리소스 사용 금지 | COVERED | COVERED | ✅ Writing 규칙 추가 |
| 6.1-2 | ❌금지 | API Key를 쿼리 파라미터로 전달 금지 | COVERED | COVERED | ✅ Writing 규칙 추가 |

---

## 개선 권고사항

### 즉시 수정 (Critical)

**카테고리 1: HTTP 메서드 안전성/멱등성 규칙**

Writing Mode 추가 내용:
- "GET and HEAD requests MUST NOT modify server state (safe methods)"
- "PUT and DELETE MUST be idempotent — repeated identical requests produce the same result"
- "PUT replaces the entire resource — omitted fields revert to default values or null"

Review Mode 체크리스트 추가 항목:
- "PUT requests are idempotent (repeated calls produce same result)"
- "PUT replaces entire resource; omitted fields reset to defaults"

**카테고리 2: URL/쿼리 파라미터 규칙**

Writing Mode 추가 내용:
- "URL path segments MUST contain only ASCII lowercase letters (a-z), digits (0-9), and hyphens (-)"
- "Array values in query parameters MUST use repeated parameter names: `?status=ACTIVE&status=DRAFT`"
- "Repeated query parameters are treated as OR condition"

Review Mode 체크리스트 추가 항목:
- "Path segments contain only a-z, 0-9, hyphens"
- "Array query parameters use repeated keys (not comma-separated)"

**카테고리 3: HTTP 헤더 규칙 (Content-Type)**

Writing Mode 추가 내용:
- "Requests with a body MUST include `Content-Type` header (typically `application/json`)"
- "Responses with a body MUST include `Content-Type` header"

Review Mode 체크리스트 추가 항목:
- "Requests with body include Content-Type header"
- "Responses with body include Content-Type header"

**카테고리 4: 리소스 스키마 Review 규칙**

Review Mode 체크리스트 추가 항목:
- "Every resource has a unique `id` field"
- "Resource includes standard fields: id, createdAt, updatedAt"
- "Server-managed fields (id, createdAt, updatedAt) ignored if sent in request body"

Writing Mode 추가 내용:
- "Server MUST silently ignore read-only fields (id, createdAt, updatedAt) if included in request body" (기존 "not modifiable by client" 문구를 보완)

**카테고리 5: JSON 타입 규칙**

Writing Mode 추가 내용:
- "Field names MUST start with a lowercase ASCII letter (a-z)" (camelCase 규칙 보완)
- "Boolean values MUST use JSON native `true`/`false` — string `\"true\"`/`\"false\"` or numeric `1`/`0` are prohibited"
- "Numeric values MUST use JSON number type, not strings (e.g., `\"age\": 25` not `\"age\": \"25\"`)"

Review Mode 체크리스트 추가 항목:
- "Field names start with lowercase letter"
- "Boolean values use JSON true/false (not string or numeric)"
- "Numeric values use JSON number type (not strings)"

**카테고리 6: 버전 관리/Deprecation 규칙**

Writing Mode 추가 내용:
- "Within the same API version, all changes MUST be backward-compatible (no field removal, no type change, no required field addition)"

Review Mode 체크리스트 추가 항목:
- "Changes within same version are backward-compatible"
- "Deprecated endpoints include Deprecation, Sunset, Link headers"

**카테고리 7: 멱등성 키 규칙**

Writing Mode 추가 내용:
- "Idempotency-Key value MUST be a client-generated UUID v4" (기존 코드 예시 보완)

Review Mode 체크리스트 추가 항목:
- "Idempotency-Key value is UUID v4"

### 다음 단계 (Minor)

아래 항목을 우선순위 순으로 정리한다. 양쪽 모두 MISSING인 항목을 최우선으로 처리한다.

**우선순위 1: 양쪽 모두 MISSING (7개)**

1. **2.2-4** GET/HEAD/DELETE 요청에 body 포함 금지 — 잘못된 구현을 방지하는 기본 규칙
2. **2.3-4** 서버 상태 변경에 쿼리 파라미터 사용 금지 — 보안/안전성 관련
3. **2.4-4** 표준 HTTP 헤더 의미 재정의 금지 — 상호운용성 관련
4. **2.1-7** URL 2000자 이하 유지 — 클라이언트 호환성 관련
5. **4.1-3** 필드 이름에 약어 남용 금지 — 가독성 관련
6. **2.3-2** 쿼리 파라미터는 선택적으로 설계
7. **2.3-3** 쿼리 파라미터에 민감한 정보 포함 금지

**우선순위 2: 양쪽 모두 MISSING + 권장 (4개)**

8. **3.1-3** 리소스 식별자는 불투명한 문자열
9. **4.2-4** 큰 정수(2^53 초과)는 문자열로 반환
10. **4.4-2** 클라이언트가 알 수 없는 Enum 값 수신 가능하도록 설계

**우선순위 3: 한쪽만 MISSING (12개)**

11. **2.1-3** URL에 동사 포함 금지 — Writing PARTIAL
12. **2.1-4** URL에 파일 확장자 포함 금지 — Writing MISSING
13. **2.2-3** 부분 수정에는 PUT 대신 PATCH 사용 — Review MISSING
14. **2.2-7** 오류 상황에 200 OK 반환 금지 — Writing MISSING
15. **2.4-3** 커스텀 헤더에 X- 접두사 사용 금지 — Writing MISSING
16. **3.1-4** 응답에 null 값 필드 포함 금지 — Writing MISSING
17. **3.4-4** 에러 응답에 스택 트레이스/내부 정보 노출 금지 — Writing MISSING
18. **4.3-5** Unix timestamp 금지 — Review MISSING
19. **4.4-3** Enum 숫자/약어 사용 금지 — Review MISSING
20. **5.4-1** API 버전을 URL 경로에 포함 금지 — Writing MISSING
21. **5.7-3** 범용 /operations 리소스 사용 금지 — Writing MISSING
22. **6.1-2** API Key를 쿼리 파라미터로 전달 금지 — Writing MISSING

---

## 다음 단계

- [x] Critical 문제 스킬에 반영 (Task 5) — 17개 Critical 항목 모두 해결
- [x] 스킬 변경 후 동일 테스트 케이스로 회귀 검증
- [x] report.md 커버리지 수치 업데이트
- [x] Minor 문제 스킬에 반영 — 19개 해결 (Tier A 14개 + Tier B 5개), 3개 Tier C 보류 (2.1-7, 2.3-2, 3.1-3)
