# Coverage Map — RESTful API Guidelines Skill

**평가 날짜:** 2026-03-20
**README:** README.md
**스킬:** .claude/skills/restful-api-guidelines.md

---

## 범례
| 상태 | 의미 |
|------|------|
| COVERED | 규칙의 핵심 요건이 명시적 문장 또는 코드 예시로 표현됨 |
| PARTIAL | 언급되지만 반례/예외/적용 범위 누락 |
| MISSING | 스킬에서 찾을 수 없음 |

---

## 섹션 2: HTTP 기본 규칙

| # | 규범 수준 | 규칙 요약 | Writing | Review | 심각도 |
|---|-----------|-----------|---------|--------|--------|
| 2.1-1 | ✅필수 | URL 경로에 소문자 kebab-case 사용 | COVERED | COVERED | — |
| 2.1-2 | ✅필수 | 리소스 컬렉션 이름은 복수형 명사 | COVERED | COVERED | — |
| 2.1-3 | ❌금지 | URL에 동사 포함 금지 | PARTIAL | COVERED | Minor |
| 2.1-4 | ❌금지 | URL에 파일 확장자 포함 금지 | MISSING | COVERED | Minor |
| 2.1-5 | ✅필수 | URL 경로 세그먼트에 ASCII 영소문자/숫자/하이픈만 허용 | PARTIAL | PARTIAL | Critical |
| 2.1-6 | ✅필수 | 쿼리 파라미터 이름은 camelCase | COVERED | COVERED | — |
| 2.1-7 | ⚠️권장 | URL 2000자 이하 유지 | MISSING | MISSING | Minor |
| 2.2-1 | ✅필수 | GET 요청은 서버 상태 변경 안 함 | MISSING | COVERED | Critical |
| 2.2-2 | ✅필수 | PUT 요청은 멱등적으로 동작 | PARTIAL | MISSING | Critical |
| 2.2-3 | ⚠️권장 | 부분 수정에는 PUT 대신 PATCH 사용 | COVERED | MISSING | Minor |
| 2.2-4 | ❌금지 | GET/HEAD/DELETE 요청에 body 포함 금지 | MISSING | MISSING | Minor |
| 2.2-5 | ✅필수 | 표준 HTTP 상태 코드를 정확한 의미에 맞게 사용 | COVERED | COVERED | — |
| 2.2-6 | ✅필수 | 201 Created 응답에 Location 헤더 포함 | COVERED | COVERED | — |
| 2.2-7 | ❌금지 | 오류 상황에 200 OK 반환 금지 | MISSING | COVERED | Minor |
| 2.3-1 | ✅필수 | 동일 파라미터 반복으로 배열 값 전달 | MISSING | MISSING | Critical |
| 2.3-2 | ⚠️권장 | 쿼리 파라미터는 선택적으로 설계 | MISSING | MISSING | Minor |
| 2.3-3 | ⚠️권장 | 쿼리 파라미터에 민감한 정보 포함 금지 | MISSING | MISSING | Minor |
| 2.3-4 | ❌금지 | 서버 상태 변경에 쿼리 파라미터 사용 금지 | MISSING | MISSING | Minor |
| 2.4-1 | ✅필수 | 요청 본문 있을 때 Content-Type 헤더 포함 | PARTIAL | MISSING | Critical |
| 2.4-2 | ✅필수 | 응답 본문 있을 때 Content-Type 헤더 포함 | PARTIAL | MISSING | Critical |
| 2.4-3 | ⚠️권장 | 커스텀 헤더에 X- 접두사 사용 금지 (신규) | MISSING | COVERED | Minor |
| 2.4-4 | ❌금지 | 표준 HTTP 헤더 의미 재정의 금지 | MISSING | MISSING | Minor |

---

## 섹션 3: REST 원칙

| # | 규범 수준 | 규칙 요약 | Writing | Review | 심각도 |
|---|-----------|-----------|---------|--------|--------|
| 3.1-1 | ✅필수 | 모든 리소스는 고유 id 가짐 | COVERED | MISSING | Critical |
| 3.1-2 | ✅필수 | 리소스 스키마는 일관된 구조 유지 (id/createdAt/updatedAt) | COVERED | MISSING | Critical |
| 3.1-3 | ⚠️권장 | 리소스 식별자는 불투명한 문자열 | MISSING | MISSING | Minor |
| 3.1-4 | ❌금지 | 응답에 null 값 필드 포함 금지 | MISSING | COVERED | Minor |
| 3.2-1 | ✅필수 | 서버 관리 읽기 전용 필드를 요청 본문에 포함해도 무시 | PARTIAL | MISSING | Critical |
| 3.3-1 | ✅필수 | POST 생성 성공 시 201 Created + 생성된 리소스 반환 | COVERED | COVERED | — |
| 3.3-2 | ✅필수 | PUT은 리소스 전체 대체, 미포함 필드는 기본값/null | PARTIAL | MISSING | Critical |
| 3.3-3 | ✅필수 | DELETE 성공 시 204 No Content 반환 | COVERED | COVERED | — |
| 3.4-1 | ✅필수 | 모든 에러 응답은 RFC 7807/9457 구조 따름 | COVERED | COVERED | — |
| 3.4-2 | ✅필수 | 에러 응답 Content-Type은 application/problem+json | COVERED | COVERED | — |
| 3.4-3 | ⚠️권장 | 유효성 검사 실패 시 모든 오류 필드 한 번에 반환 | COVERED | COVERED | — |
| 3.4-4 | ❌금지 | 에러 응답에 스택 트레이스/내부 정보 노출 금지 | MISSING | COVERED | Minor |

---

## 섹션 4: JSON 규칙

| # | 규범 수준 | 규칙 요약 | Writing | Review | 심각도 |
|---|-----------|-----------|---------|--------|--------|
| 4.1-1 | ✅필수 | JSON 필드 이름은 camelCase | COVERED | COVERED | — |
| 4.1-2 | ✅필수 | 필드 이름은 영소문자로 시작 | PARTIAL | MISSING | Critical |
| 4.1-3 | ❌금지 | 필드 이름에 약어 남용 금지 | MISSING | MISSING | Minor |
| 4.2-1 | ✅필수 | Boolean은 JSON true/false 사용 | PARTIAL | MISSING | Critical |
| 4.2-2 | ✅필수 | Boolean 필드 이름에 is/has/can 접두사 | COVERED | COVERED | — |
| 4.2-3 | ✅필수 | 숫자 값은 JSON number 타입 | MISSING | MISSING | Critical |
| 4.2-4 | ⚠️권장 | 큰 정수(2^53 초과)는 문자열로 반환 | MISSING | MISSING | Minor |
| 4.3-1 | ✅필수 | 날짜/시간은 RFC 3339 형식 문자열 | COVERED | COVERED | — |
| 4.3-2 | ✅필수 | 시간대가 있으면 반드시 포함, UTC는 Z | COVERED | COVERED | — |
| 4.3-3 | ✅필수 | 서버 응답 시간 값은 모두 UTC(Z) | COVERED | COVERED | — |
| 4.3-4 | ✅필수 | 클라이언트가 오프셋 포함 전송 시 서버가 UTC로 변환하여 저장 | COVERED | COVERED | — |
| 4.3-5 | ❌금지 | Unix timestamp를 기본 시간 형식으로 사용 금지 | COVERED | MISSING | Minor |
| 4.4-1 | ✅필수 | Enum 값은 UPPER_SNAKE_CASE 문자열 | COVERED | COVERED | — |
| 4.4-2 | ⚠️권장 | 클라이언트가 알 수 없는 Enum 값 수신 가능하도록 설계 | MISSING | MISSING | Minor |
| 4.4-3 | ❌금지 | Enum 값으로 숫자나 불명확한 약어 사용 금지 | PARTIAL | MISSING | Minor |

---

## 섹션 5: 공통 API 패턴

| # | 규범 수준 | 규칙 요약 | Writing | Review | 심각도 |
|---|-----------|-----------|---------|--------|--------|
| 5.1-1 | ✅필수 | 액션은 리소스 URL 뒤에 :action 형태 | COVERED | COVERED | — |
| 5.1-2 | ✅필수 | 액션 엔드포인트에 POST 메서드 사용 | COVERED | COVERED | — |
| 5.2-1 | ✅필수 | 컬렉션 응답 본문은 top-level JSON array | COVERED | COVERED | — |
| 5.2-2 | ✅필수 | 다음 페이지 없을 때 Link 헤더에서 rel="next" 제외 | COVERED | COVERED | — |
| 5.3-1 | ✅필수 | 동일 파라미터 반복은 OR 조건 | MISSING | COVERED | Critical |
| 5.4-1 | ❌금지 | API 버전을 URL 경로에 포함 금지 | MISSING | COVERED | Minor |
| 5.4-2 | ✅필수 | X-API-Version 헤더에 ISO 8601 날짜 형식으로 버전 지정 | COVERED | COVERED | — |
| 5.4-3 | ✅필수 | 동일 버전 내 하위 호환성 유지 | MISSING | MISSING | Critical |
| 5.5-1 | ✅필수 | Deprecated API에 Deprecation/Sunset/Link 응답 헤더 제공 | COVERED | MISSING | Critical |
| 5.6-1 | ✅필수 | 속도 제한 응답에 X-RateLimit-* 헤더 포함 | COVERED | COVERED | — |
| 5.6-2 | ✅필수 | 429 응답에 Retry-After 헤더 포함 | COVERED | COVERED | — |
| 5.6-3 | ✅필수 | 429 응답 본문은 RFC 7807 Problem Details 구조 | COVERED | COVERED | — |
| 5.6-4 | ✅필수 | 클라이언트는 429 수신 시 Retry-After 값만큼 대기 후 재시도 | COVERED | COVERED | — |
| 5.7-1 | ✅필수 | 장기 실행 작업 시 도메인 리소스 즉시 생성 + 201 Created + Location 헤더 | COVERED | COVERED | — |
| 5.7-2 | ✅필수 | 도메인 리소스에 status 필드 포함 | COVERED | COVERED | — |
| 5.7-3 | ❌금지 | 범용 /operations 리소스 사용 금지 | MISSING | COVERED | Minor |

---

## 섹션 6: 인증 및 보안

| # | 규범 수준 | 규칙 요약 | Writing | Review | 심각도 |
|---|-----------|-----------|---------|--------|--------|
| 6.1-1 | ✅필수 | 인증 토큰은 Authorization 헤더 사용 | COVERED | COVERED | — |
| 6.1-2 | ❌금지 | API Key를 쿼리 파라미터로 전달 금지 | MISSING | COVERED | Minor |
| 6.2-1 | ✅필수 | 401 응답에 WWW-Authenticate 헤더 포함 | COVERED | COVERED | — |
| 6.2-2 | ✅필수 | 401(인증 실패) / 403(인가 실패) 정확히 구분 | COVERED | COVERED | — |
| 6.3-1 | ✅필수 | 중복 실행 위험 있는 POST에 Idempotency-Key 지원 | COVERED | COVERED | — |
| 6.3-2 | ✅필수 | Idempotency-Key 값은 클라이언트 생성 UUID v4 | PARTIAL | MISSING | Critical |

---

## 커버리지 합계

### Writing 모드

| 상태 | 개수 | 비율 |
|------|------|------|
| COVERED | 37 | 52.1% |
| PARTIAL | 11 | 15.5% |
| MISSING | 23 | 32.4% |
| **합계** | **71** | **100%** |

### Review 모드

| 상태 | 개수 | 비율 |
|------|------|------|
| COVERED | 43 | 60.6% |
| PARTIAL | 1 | 1.4% |
| MISSING | 27 | 38.0% |
| **합계** | **71** | **100%** |

### 전체 커버리지 (Writing + Review 통합)

| 상태 | 개수 | 비율 |
|------|------|------|
| COVERED | 80 | 56.3% |
| PARTIAL | 12 | 8.5% |
| MISSING | 50 | 35.2% |
| **합계** | **142** | **100%** |

---

## Critical 규칙 목록

✅필수 규칙이 Writing 또는 Review 모드에서 MISSING 또는 PARTIAL인 항목:

| # | 규칙 요약 | Writing | Review | 비고 |
|---|-----------|---------|--------|------|
| 2.1-5 | URL 경로 세그먼트에 ASCII 영소문자/숫자/하이픈만 허용 | PARTIAL | PARTIAL | kebab-case 언급은 있으나 허용 문자 집합을 명시적으로 제한하지 않음 |
| 2.2-1 | GET 요청은 서버 상태 변경 안 함 | MISSING | COVERED | Writing 모드에 안전성(safety) 규칙 명시 없음 |
| 2.2-2 | PUT 요청은 멱등적으로 동작 | PARTIAL | MISSING | Writing 모드 상태코드 표에 "Full replacement" 언급만, Review에 멱등성 체크 없음 |
| 2.3-1 | 동일 파라미터 반복으로 배열 값 전달 | MISSING | MISSING | 쿼리 파라미터 배열 전달 방식 자체가 스킬에 없음 |
| 2.4-1 | 요청 본문 있을 때 Content-Type 헤더 포함 | PARTIAL | MISSING | 에러 응답의 Content-Type만 언급, 일반 요청 Content-Type 규칙 없음 |
| 2.4-2 | 응답 본문 있을 때 Content-Type 헤더 포함 | PARTIAL | MISSING | 에러 응답의 Content-Type만 명시, 일반 응답 Content-Type 규칙 없음 |
| 3.1-1 | 모든 리소스는 고유 id 가짐 | COVERED | MISSING | Review 체크리스트에 리소스 id 필수 항목 없음 |
| 3.1-2 | 리소스 스키마는 일관된 구조 유지 (id/createdAt/updatedAt) | COVERED | MISSING | Review 체크리스트에 표준 필드 구조 검증 항목 없음 |
| 3.2-1 | 서버 관리 읽기 전용 필드를 요청 본문에 포함해도 무시 | PARTIAL | MISSING | Writing에 "not modifiable by client" 언급, 무시 동작 명시 불충분 |
| 3.3-2 | PUT은 리소스 전체 대체, 미포함 필드는 기본값/null | PARTIAL | MISSING | "Full replacement" 언급만, 미포함 필드 처리 규칙 누락 |
| 4.1-2 | 필드 이름은 영소문자로 시작 | PARTIAL | MISSING | camelCase 예시에서 암시되나 명시적 규칙 없음 |
| 4.2-1 | Boolean은 JSON true/false 사용 | PARTIAL | MISSING | 코드 예시에서 true/false 사용하나, 문자열/"1"/"0" 금지 명시 없음 |
| 4.2-3 | 숫자 값은 JSON number 타입 | MISSING | MISSING | 숫자 타입 규칙 자체가 스킬에 없음 |
| 5.3-1 | 동일 파라미터 반복은 OR 조건 | MISSING | COVERED | Writing 모드에 필터링 OR/AND 조건 규칙 없음 |
| 5.4-3 | 동일 버전 내 하위 호환성 유지 | MISSING | MISSING | 하위 호환성 유지 규칙 및 호환/비호환 변경 목록이 스킬에 없음 |
| 5.5-1 | Deprecated API에 Deprecation/Sunset/Link 응답 헤더 제공 | COVERED | MISSING | Review 체크리스트에 Deprecation 헤더 검증 항목 없음 |
| 6.3-2 | Idempotency-Key 값은 클라이언트 생성 UUID v4 | PARTIAL | MISSING | 코드 예시에 UUID 형태 값 있으나 "UUID v4" 명시 없음, Review 체크 없음 |

**Critical 항목 총 17개** (✅필수 규칙 중 Writing 또는 Review에서 MISSING/PARTIAL)

---

## 판정 근거 요약

### Writing 모드 판정 근거

| # | 판정 | 근거 |
|---|------|------|
| 2.1-1 | COVERED | "Plural nouns + kebab-case" 주석과 `/user-profiles` 등 예시 있음 |
| 2.1-2 | COVERED | `/articles`, `/users/{userId}/comments` 복수형 예시 있음 |
| 2.1-3 | PARTIAL | 액션 패턴 `:action` 예시 있으나 "동사 금지" 규칙 자체를 명시하지 않음, bad case 없음 |
| 2.1-4 | MISSING | 파일 확장자 금지 언급 없음 |
| 2.1-5 | PARTIAL | kebab-case 언급으로 간접 포함되나 허용 문자 집합(ASCII 영소문자/숫자/하이픈만) 명시 없음 |
| 2.1-6 | COVERED | `pageSize`, `pageToken`, `sortOrder` 등 camelCase 예시와 "(camelCase)" 명시 있음 |
| 2.1-7 | MISSING | URL 길이 제한 언급 없음 |
| 2.2-1 | MISSING | GET의 안전성(서버 상태 변경 안 함) 규칙 명시 없음 |
| 2.2-2 | PARTIAL | 상태코드 표에 "Full replacement success" 언급만, 멱등성 규칙 명시 없음 |
| 2.2-3 | COVERED | PATCH 메서드 코드 예시(updateArticle)와 상태코드 매핑 있음 |
| 2.2-4 | MISSING | GET/HEAD/DELETE body 금지 규칙 없음 |
| 2.2-5 | COVERED | HTTP Method to Status Code Mapping 표에 상세 매핑 있음 |
| 2.2-6 | COVERED | "201 Created + Location header" 명시, 코드 예시에 `ResponseEntity.created(location)` 있음 |
| 2.2-7 | MISSING | "200 OK로 에러 반환 금지" 규칙 명시 없음 |
| 2.3-1 | MISSING | 동일 파라미터 반복으로 배열 전달하는 패턴 없음 |
| 2.3-2 | MISSING | 쿼리 파라미터 선택적 설계 규칙 없음 |
| 2.3-3 | MISSING | 쿼리 파라미터 민감 정보 금지 규칙 없음 |
| 2.3-4 | MISSING | 서버 상태 변경에 쿼리 파라미터 사용 금지 없음 |
| 2.4-1 | PARTIAL | 에러 응답 `application/problem+json` 언급은 있으나, 일반 요청의 Content-Type 필수 규칙 없음 |
| 2.4-2 | PARTIAL | 에러 응답 Content-Type만 명시, 정상 응답의 Content-Type 필수 규칙은 코드 예시에서 암시적 |
| 2.4-3 | MISSING | 커스텀 헤더 X- 접두사 금지 명시 없음 (Rate Limit 헤더에서 X-RateLimit-* 사용) |
| 2.4-4 | MISSING | 표준 헤더 의미 재정의 금지 없음 |
| 3.1-1 | COVERED | Standard Resource Fields에 `"id": "Server-generated"` 명시 |
| 3.1-2 | COVERED | id/createdAt/updatedAt 표준 필드 구조가 Standard Resource Fields에 정의됨 |
| 3.1-3 | MISSING | 식별자의 불투명성 언급 없음 |
| 3.1-4 | MISSING | null 값 필드 제외 규칙이 Writing 모드에 없음 |
| 3.2-1 | PARTIAL | "not modifiable by client" 언급 있으나 "포함해도 무시" 동작 명확히 서술되지 않음 |
| 3.3-1 | COVERED | POST create -> 201 Created + Location + body 코드 예시 있음 |
| 3.3-2 | PARTIAL | PUT -> "Full replacement success" 테이블 항목만, 미포함 필드 처리 규칙 없음 |
| 3.3-3 | COVERED | DELETE -> 204 No Content 매핑 및 `ResponseEntity.noContent().build()` 코드 있음 |
| 3.4-1 | COVERED | RFC 7807/9457 구조 명시 + ProblemDetail 클래스 코드 예시 있음 |
| 3.4-2 | COVERED | `Content-Type: application/problem+json` 명시 |
| 3.4-3 | COVERED | errors 배열 필드를 포함한 에러 응답 템플릿 있음 |
| 3.4-4 | MISSING | 스택 트레이스/내부 정보 노출 금지 Writing 가이드 없음 |
| 4.1-1 | COVERED | "Correct example" / "Incorrect example" JSON 비교 있음 |
| 4.1-2 | PARTIAL | camelCase 예시가 모두 소문자 시작이지만 "영소문자로 시작" 규칙 명시 없음 |
| 4.1-3 | MISSING | 약어 남용 금지 규칙 없음 |
| 4.2-1 | PARTIAL | `"isActive": true` 예시 있으나, 문자열 "true"/"false" 및 숫자 1/0 금지 명시 없음 |
| 4.2-2 | COVERED | `is`/`has`/`can` 접두사 예시: `"isActive": true` |
| 4.2-3 | MISSING | 숫자 타입 규칙 없음 |
| 4.2-4 | MISSING | 큰 정수 문자열 반환 규칙 없음 |
| 4.3-1 | COVERED | RFC 3339 형식 명시 + 예시 있음 |
| 4.3-2 | COVERED | UTC `Z` 사용 예시 있음 |
| 4.3-3 | COVERED | "Server response (Required: UTC)" 명시 |
| 4.3-4 | COVERED | "offset allowed — server normalizes to UTC" 명시 |
| 4.3-5 | COVERED | "Unix timestamp forbidden" Prohibited 섹션에 명시 |
| 4.4-1 | COVERED | `"status": "PUBLISHED"` 예시 + "enums must be UPPER_SNAKE_CASE" 명시 |
| 4.4-2 | MISSING | 알 수 없는 Enum 값 대응 설계 규칙 없음 |
| 4.4-3 | PARTIAL | "enums must be UPPER_SNAKE_CASE" 명시 있으나 숫자/약어 사용 금지 bad case 없음 |
| 5.1-1 | COVERED | `:publish`, `:cancel`, `:deactivate` 액션 패턴 예시 있음 |
| 5.1-2 | COVERED | POST 메서드 사용 예시 있음 |
| 5.2-1 | COVERED | top-level JSON array 응답 예시 + "top-level array" 명시 |
| 5.2-2 | COVERED | `nextPageToken` null 시 Link에 next 미포함하는 buildLinkHeader 코드 있음 |
| 5.3-1 | MISSING | OR 조건 규칙이 Writing 모드에 없음 |
| 5.4-1 | MISSING | URL 경로 버전 금지 규칙이 Writing 모드에 없음 |
| 5.4-2 | COVERED | X-API-Version 헤더 코드 예시 있음 (ISO 8601 날짜 형식 "2024-01-20") |
| 5.4-3 | MISSING | 하위 호환성 유지 규칙 없음 |
| 5.5-1 | COVERED | Deprecation/Sunset/Link 헤더 설정 코드 예시 있음 |
| 5.6-1 | COVERED | addRateLimitHeaders 함수에 X-RateLimit-* 헤더 설정 코드 있음 |
| 5.6-2 | COVERED | Retry-After 헤더 설정 코드 있음 |
| 5.6-3 | COVERED | 429 응답에 ProblemDetail 구조 사용 코드 있음 |
| 5.6-4 | COVERED | "On 429, wait for the Retry-After header value before retrying" 명시 |
| 5.7-1 | COVERED | "POST (long-running) 201 Created + Location header" 상태코드 표에 있음 |
| 5.7-2 | COVERED | "domain resource created immediately with status field" 명시 |
| 5.7-3 | MISSING | /operations 금지 규칙이 Writing 모드에 없음 |
| 6.1-1 | COVERED | Authorization 헤더 Bearer/ApiKey 예시 있음 |
| 6.1-2 | MISSING | API Key 쿼리 파라미터 전달 금지 명시 없음 |
| 6.2-1 | COVERED | WWW-Authenticate 헤더 포함 코드 예시 있음 |
| 6.2-2 | COVERED | 401 vs 403 구분 표 있음 |
| 6.3-1 | COVERED | Idempotency-Key 처리 코드 예시 있음 |
| 6.3-2 | PARTIAL | 코드 예시의 키 값이 UUID 형태이나 "UUID v4" 명시 없음 |

### Review 모드 판정 근거

| # | 판정 | 근거 |
|---|------|------|
| 2.1-1 | COVERED | "Lowercase kebab-case used in paths" 체크리스트 항목 |
| 2.1-2 | COVERED | "Resource names are plural nouns" 체크리스트 항목 |
| 2.1-3 | COVERED | "No verbs in paths (actions use :action pattern)" 체크리스트 항목 |
| 2.1-4 | COVERED | "No file extensions in URLs" 체크리스트 항목 |
| 2.1-5 | PARTIAL | kebab-case 체크는 있으나 허용 문자 집합 검증 항목 없음 |
| 2.1-6 | COVERED | "Query parameters are camelCase" 체크리스트 항목 |
| 2.1-7 | MISSING | URL 길이 체크 항목 없음 |
| 2.2-1 | COVERED | "GET requests do not modify server state" 체크리스트 항목 |
| 2.2-2 | MISSING | PUT 멱등성 체크 항목 없음 |
| 2.2-3 | MISSING | PATCH 권장 체크 항목 없음 |
| 2.2-4 | MISSING | GET/HEAD/DELETE body 금지 체크 항목 없음 |
| 2.2-5 | COVERED | 상태 코드 관련 체크리스트 항목 다수 있음 |
| 2.2-6 | COVERED | "POST create -> 201 + Location header" 체크리스트 항목 |
| 2.2-7 | COVERED | "200 not returned for error conditions" 체크리스트 항목 |
| 2.3-1 | MISSING | 동일 파라미터 반복 배열 전달 체크 항목 없음 (5.3-1 필터링에서 OR 조건은 있음) |
| 2.3-2 | MISSING | 쿼리 파라미터 선택적 설계 체크 없음 |
| 2.3-3 | MISSING | 쿼리 파라미터 민감 정보 체크 없음 |
| 2.3-4 | MISSING | 서버 상태 변경 쿼리 파라미터 금지 체크 없음 |
| 2.4-1 | MISSING | 요청 Content-Type 체크 항목 없음 |
| 2.4-2 | MISSING | 응답 Content-Type 일반 규칙 체크 항목 없음 (에러 응답 Content-Type만) |
| 2.4-3 | COVERED | "New custom headers do not use X- prefix" 체크리스트 항목 |
| 2.4-4 | MISSING | 표준 헤더 의미 재정의 금지 체크 없음 |
| 3.1-1 | MISSING | 리소스 id 필수 체크 항목 없음 |
| 3.1-2 | MISSING | 리소스 표준 필드 구조 체크 항목 없음 |
| 3.1-3 | MISSING | 식별자 불투명성 체크 없음 |
| 3.1-4 | COVERED | "Null-valued fields excluded from response" 체크리스트 항목 |
| 3.2-1 | MISSING | 읽기 전용 필드 무시 동작 체크 없음 |
| 3.3-1 | COVERED | "POST create -> 201 + Location header" 체크리스트 항목 |
| 3.3-2 | MISSING | PUT 전체 대체 + 미포함 필드 처리 체크 없음 |
| 3.3-3 | COVERED | "DELETE success -> 204 (no body)" 체크리스트 항목 |
| 3.4-1 | COVERED | "Error responses use RFC 7807/9457 Problem Details structure" 체크리스트 항목 |
| 3.4-2 | COVERED | "Content-Type: application/problem+json used" 체크리스트 항목 |
| 3.4-3 | COVERED | "All validation errors returned at once" 체크리스트 항목 |
| 3.4-4 | COVERED | "Internal implementation details not exposed" 체크리스트 항목 |
| 4.1-1 | COVERED | "All fields are camelCase" 체크리스트 항목 |
| 4.1-2 | MISSING | 영소문자 시작 체크 항목 없음 |
| 4.1-3 | MISSING | 약어 남용 금지 체크 없음 |
| 4.2-1 | MISSING | Boolean JSON true/false 체크 항목 없음 |
| 4.2-2 | COVERED | "Boolean fields use is/has/can prefix" 체크리스트 항목 |
| 4.2-3 | MISSING | 숫자 JSON number 타입 체크 없음 |
| 4.2-4 | MISSING | 큰 정수 문자열 반환 체크 없음 |
| 4.3-1 | COVERED | "Date/time in RFC 3339 format" 체크리스트 항목 |
| 4.3-2 | COVERED | UTC(Z) 응답 체크리스트 항목에서 간접 포함 |
| 4.3-3 | COVERED | "All time values in server response are UTC (Z)" 체크리스트 항목 |
| 4.3-4 | COVERED | "Offset input is normalized to UTC by server (not an error)" 체크리스트 항목 |
| 4.3-5 | MISSING | Unix timestamp 금지 체크 항목 없음 |
| 4.4-1 | COVERED | "Enum values are UPPER_SNAKE_CASE" 체크리스트 항목 |
| 4.4-2 | MISSING | 알 수 없는 Enum 값 대응 체크 없음 |
| 4.4-3 | MISSING | Enum 숫자/약어 금지 체크 없음 |
| 5.1-1 | COVERED | "No verbs in paths (actions use :action pattern)" 체크리스트 항목 |
| 5.1-2 | COVERED | 액션 패턴이 POST 사용하는 것이 URL Design 체크에 포함 |
| 5.2-1 | COVERED | "Collection response body is a top-level array" 체크리스트 항목 |
| 5.2-2 | COVERED | "rel=next excluded from Link header when no next page" 체크리스트 항목 |
| 5.3-1 | COVERED | "Repeated same parameter treated as OR condition" 체크리스트 항목 |
| 5.4-1 | COVERED | "No version in URL path" + "API version delivered via X-API-Version header, not URL path" 체크리스트 항목 |
| 5.4-2 | COVERED | "X-API-Version value uses ISO 8601 date format" 체크리스트 항목 |
| 5.4-3 | MISSING | 하위 호환성 유지 체크 없음 |
| 5.5-1 | MISSING | Deprecation 헤더 체크 항목 없음 |
| 5.6-1 | COVERED | Rate Limiting 체크리스트에 X-RateLimit-* 헤더 항목 있음 |
| 5.6-2 | COVERED | "429 response includes Retry-After header" 체크리스트 항목 |
| 5.6-3 | COVERED | "429 response body uses Problem Details structure" 체크리스트 항목 |
| 5.6-4 | COVERED | "Client retry respects Retry-After value" 체크리스트 항목 |
| 5.7-1 | COVERED | "Long-running task returns 201 Created + Location header" 체크리스트 항목 |
| 5.7-2 | COVERED | "Domain resource has status field" 체크리스트 항목 |
| 5.7-3 | COVERED | "No generic /operations endpoint" 체크리스트 항목 |
| 6.1-1 | COVERED | "Auth token delivered via Authorization header" 체크리스트 항목 |
| 6.1-2 | COVERED | "(query parameter forbidden)" 체크리스트 항목 |
| 6.2-1 | COVERED | "401 response includes WWW-Authenticate header" 체크리스트 항목 |
| 6.2-2 | COVERED | "401/403 properly distinguished" 체크리스트 항목 |
| 6.3-1 | COVERED | "Idempotency-Key supported for duplicate-risk POST operations" 체크리스트 항목 |
| 6.3-2 | MISSING | UUID v4 명시 체크 항목 없음 |
