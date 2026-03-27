# Skill Performance Evaluation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** RESTful API Guidelines 스킬(restful-api-guidelines.md)의 README 규칙 커버리지를 정량화하고 테스트 케이스로 검증하여 개선 보고서를 작성한다.

**Architecture:** README의 모든 규칙을 추출 → 스킬과 매핑하여 커버리지 맵 작성 → 섹션별 bad/good 코드 테스트 케이스 작성 → 커버리지 수치 기반 보고서 작성 → Critical 문제 즉시 스킬에 반영.

**Tech Stack:** Markdown 문서 작업. 파일 읽기/비교/작성. 자동화 테스트 없음 — 모든 검증은 README ↔ 스킬 텍스트 비교로 수행.

---

## 파일 구조

| 파일 | 역할 |
|------|------|
| `docs/evaluation/coverage-map.md` | README 규칙 ↔ 스킬 커버 상태 매핑 테이블 |
| `docs/evaluation/test-cases.md` | 섹션별 bad/good 코드 테스트 케이스 (~54개) |
| `docs/evaluation/report.md` | 커버리지 수치, Critical/Minor 목록, 개선 권고사항 |
| `.claude/skills/restful-api-guidelines.md` | 평가 후 Critical 문제 수정 반영 |

---

## 참고 파일

- README: `/home/ppzxc/projects/restful-api-guidelines/README.md`
- 스킬: `/home/ppzxc/projects/restful-api-guidelines/.claude/skills/restful-api-guidelines.md`
- 스펙: `/home/ppzxc/projects/restful-api-guidelines/docs/superpowers/specs/2026-03-19-skill-performance-evaluation-design.md`

---

## 커버리지 판정 기준 (모든 태스크에서 공통 적용)

| 상태 | 기준 |
|------|------|
| `COVERED` | 규칙의 핵심 요건이 스킬에 명시적 문장 또는 코드 예시로 표현됨 |
| `PARTIAL` | 규칙이 언급되지만 반례(bad case), 예외 조건, 또는 적용 범위가 누락됨 (bad case는 위반 패턴이 존재하는 규칙에만 요구됨) |
| `MISSING` | 스킬에서 해당 규칙을 찾을 수 없음 |

심각도:
- **Critical** = ✅필수 규칙이 Writing 또는 Review 모드에서 MISSING 또는 PARTIAL
- **Minor** = ⚠️권장/❌금지 규칙 누락 또는 스킬 코드 예시 오류

---

## Task 1: 커버리지 맵 작성 (docs/evaluation/coverage-map.md)

**Files:**
- Read: `README.md` (규칙 추출)
- Read: `.claude/skills/restful-api-guidelines.md` (커버 여부 확인)
- Create: `docs/evaluation/coverage-map.md`

README의 모든 ✅필수/⚠️권장/❌금지 규칙을 목록화하고 스킬의 Writing/Review 모드 각각에서 COVERED/PARTIAL/MISSING 상태를 기록한다.

- [ ] **Step 1: README 섹션 2 규칙 추출 및 매핑**

README 2.1~2.4(URL, HTTP 메서드, 상태코드, 쿼리파라미터, 헤더)의 모든 규칙을 추출한다.
각 규칙을 스킬의 Code Writing Mode 및 Code Review Mode 체크리스트에서 검색하여 커버 상태를 판정한다.

`docs/evaluation/coverage-map.md` 파일을 생성하고 다음 형식으로 섹션 2 규칙을 작성한다:

```markdown
# Coverage Map — RESTful API Guidelines Skill

**평가 날짜:** 2026-03-19
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
| 2.1-1 | ✅필수 | URL 경로에 소문자 kebab-case 사용 | ? | ? | ? |
| 2.1-2 | ✅필수 | 리소스 컬렉션 이름은 복수형 명사 | ? | ? | ? |
| 2.1-3 | ❌금지 | URL에 동사 포함 금지 | ? | ? | ? |
| 2.1-4 | ❌금지 | URL에 파일 확장자 포함 금지 | ? | ? | ? |
| 2.1-5 | ✅필수 | URL 경로 세그먼트에 ASCII 영소문자/숫자/하이픈만 허용 | ? | ? | ? |
| 2.1-6 | ✅필수 | 쿼리 파라미터 이름은 camelCase | ? | ? | ? |
| 2.1-7 | ⚠️권장 | URL 2000자 이하 유지 | ? | ? | ? |
| 2.2-1 | ✅필수 | GET 요청은 서버 상태 변경 안 함 | ? | ? | ? |
| 2.2-2 | ✅필수 | PUT 요청은 멱등적으로 동작 | ? | ? | ? |
| 2.2-3 | ⚠️권장 | 부분 수정에는 PUT 대신 PATCH 사용 | ? | ? | ? |
| 2.2-4 | ❌금지 | GET/HEAD/DELETE 요청에 body 포함 금지 | ? | ? | ? |
| 2.2-5 | ✅필수 | 표준 HTTP 상태 코드를 정확한 의미에 맞게 사용 | ? | ? | ? |
| 2.2-6 | ✅필수 | 201 Created 응답에 Location 헤더 포함 | ? | ? | ? |
| 2.2-7 | ❌금지 | 오류 상황에 200 OK 반환 금지 | ? | ? | ? |
| 2.3-1 | ✅필수 | 동일 파라미터 반복으로 배열 값 전달 | ? | ? | ? |
| 2.3-2 | ⚠️권장 | 쿼리 파라미터는 선택적으로 설계 | ? | ? | ? |
| 2.3-3 | ⚠️권장 | 쿼리 파라미터에 민감한 정보 포함 금지 | ? | ? | ? |
| 2.3-4 | ❌금지 | 서버 상태 변경에 쿼리 파라미터 사용 금지 | ? | ? | ? |
| 2.4-1 | ✅필수 | 요청 본문 있을 때 Content-Type 헤더 포함 | ? | ? | ? |
| 2.4-2 | ✅필수 | 응답 본문 있을 때 Content-Type 헤더 포함 | ? | ? | ? |
| 2.4-3 | ⚠️권장 | 커스텀 헤더에 X- 접두사 사용 금지 (신규) | ? | ? | ? |
| 2.4-4 | ❌금지 | 표준 HTTP 헤더 의미 재정의 금지 | ? | ? | ? |
```

`?` 값을 실제 스킬 파일을 확인하여 COVERED/PARTIAL/MISSING 중 하나로 채운다.

- [ ] **Step 2: README 섹션 3 규칙 추출 및 매핑**

README 3.1~3.4(리소스 스키마, 필드 변경 가능성, 생성/수정/대체, 에러 처리) 규칙을 추출하고 coverage-map.md에 추가한다.

```markdown
## 섹션 3: REST 원칙

| # | 규범 수준 | 규칙 요약 | Writing | Review | 심각도 |
|---|-----------|-----------|---------|--------|--------|
| 3.1-1 | ✅필수 | 모든 리소스는 고유 id 가짐 | ? | ? | ? |
| 3.1-2 | ✅필수 | 리소스 스키마는 일관된 구조 유지 (id/createdAt/updatedAt) | ? | ? | ? |
| 3.1-3 | ⚠️권장 | 리소스 식별자는 불투명한 문자열 | ? | ? | ? |
| 3.1-4 | ❌금지 | 응답에 null 값 필드 포함 금지 | ? | ? | ? |
| 3.2-1 | ✅필수 | 서버 관리 읽기 전용 필드(id/createdAt/updatedAt)를 요청 본문에 포함해도 무시 | ? | ? | ? |
| 3.3-1 | ✅필수 | POST 생성 성공 시 201 Created + 생성된 리소스 반환 | ? | ? | ? |
| 3.3-2 | ✅필수 | PUT은 리소스 전체 대체, 미포함 필드는 기본값/null | ? | ? | ? |
| 3.3-3 | ✅필수 | DELETE 성공 시 204 No Content 반환 | ? | ? | ? |
| 3.4-1 | ✅필수 | 모든 에러 응답은 RFC 7807/9457 구조 따름 | ? | ? | ? |
| 3.4-2 | ✅필수 | 에러 응답 Content-Type은 application/problem+json | ? | ? | ? |
| 3.4-3 | ⚠️권장 | 유효성 검사 실패 시 모든 오류 필드 한 번에 반환 | ? | ? | ? |
| 3.4-4 | ❌금지 | 에러 응답에 스택 트레이스/내부 정보 노출 금지 | ? | ? | ? |
```

- [ ] **Step 3: README 섹션 4 규칙 추출 및 매핑**

README 4.1~4.4(필드 네이밍, 타입 시스템, 날짜/시간, Enum) 규칙을 추출하고 coverage-map.md에 추가한다.

```markdown
## 섹션 4: JSON 규칙

| # | 규범 수준 | 규칙 요약 | Writing | Review | 심각도 |
|---|-----------|-----------|---------|--------|--------|
| 4.1-1 | ✅필수 | JSON 필드 이름은 camelCase | ? | ? | ? |
| 4.1-2 | ✅필수 | 필드 이름은 영소문자로 시작 | ? | ? | ? |
| 4.1-3 | ❌금지 | 필드 이름에 약어 남용 금지 | ? | ? | ? |
| 4.2-1 | ✅필수 | Boolean은 JSON true/false 사용 | ? | ? | ? |
| 4.2-2 | ✅필수 | Boolean 필드 이름에 is/has/can 접두사 | ? | ? | ? |
| 4.2-3 | ✅필수 | 숫자 값은 JSON number 타입 | ? | ? | ? |
| 4.2-4 | ⚠️권장 | 큰 정수(2^53 초과)는 문자열로 반환 | ? | ? | ? |
| 4.3-1 | ✅필수 | 날짜/시간은 RFC 3339 형식 문자열 | ? | ? | ? |
| 4.3-2 | ✅필수 | 시간대가 있으면 반드시 포함, UTC는 Z | ? | ? | ? |
| 4.3-3 | ✅필수 | 서버 응답 시간 값은 모두 UTC(Z) | ? | ? | ? |
| 4.3-4 | ✅필수 | 클라이언트가 오프셋 포함 전송 시 서버가 UTC로 변환하여 저장 | ? | ? | ? |
| 4.3-5 | ❌금지 | Unix timestamp를 기본 시간 형식으로 사용 금지 | ? | ? | ? |
| 4.4-1 | ✅필수 | Enum 값은 UPPER_SNAKE_CASE 문자열 | ? | ? | ? |
| 4.4-2 | ⚠️권장 | 클라이언트가 알 수 없는 Enum 값 수신 가능하도록 설계 | ? | ? | ? |
| 4.4-3 | ❌금지 | Enum 값으로 숫자나 불명확한 약어 사용 금지 | ? | ? | ? |
```

- [ ] **Step 4: README 섹션 5~6 규칙 추출 및 매핑**

README 5.1~5.7, 6.1~6.3 규칙을 추출하고 coverage-map.md에 추가한다.

```markdown
## 섹션 5: 공통 API 패턴

| # | 규범 수준 | 규칙 요약 | Writing | Review | 심각도 |
|---|-----------|-----------|---------|--------|--------|
| 5.1-1 | ✅필수 | 액션은 리소스 URL 뒤에 :action 형태 | ? | ? | ? |
| 5.1-2 | ✅필수 | 액션 엔드포인트에 POST 메서드 사용 | ? | ? | ? |
| 5.2-1 | ✅필수 | 컬렉션 응답 본문은 top-level JSON array | ? | ? | ? |
| 5.2-2 | ✅필수 | 다음 페이지 없을 때 Link 헤더에서 rel="next" 제외 | ? | ? | ? |
| 5.3-1 | ✅필수 | 동일 파라미터 반복은 OR 조건 | ? | ? | ? |
| 5.4-1 | ❌금지 | API 버전을 URL 경로에 포함 금지 | ? | ? | ? |
| 5.4-2 | ✅필수 | X-API-Version 헤더에 ISO 8601 날짜 형식으로 버전 지정 | ? | ? | ? |
| 5.4-3 | ✅필수 | 동일 버전 내 하위 호환성 유지 | ? | ? | ? |
| 5.5-1 | ✅필수 | Deprecated API에 Deprecation/Sunset/Link 응답 헤더 제공 | ? | ? | ? |
| 5.6-1 | ✅필수 | 속도 제한 응답에 X-RateLimit-* 헤더 포함 | ? | ? | ? |
| 5.6-2 | ✅필수 | 429 응답에 Retry-After 헤더 포함 | ? | ? | ? |
| 5.6-3 | ✅필수 | 429 응답 본문은 RFC 7807 Problem Details 구조 | ? | ? | ? |
| 5.6-4 | ✅필수 | 클라이언트는 429 수신 시 Retry-After 값만큼 대기 후 재시도 | ? | ? | ? |
| 5.7-1 | ✅필수 | 장기 실행 작업 시 도메인 리소스 즉시 생성 + 201 Created + Location 헤더 | ? | ? | ? |
| 5.7-2 | ✅필수 | 도메인 리소스에 status 필드 포함 | ? | ? | ? |
| 5.7-3 | ❌금지 | 범용 /operations 리소스 사용 금지 | ? | ? | ? |

## 섹션 6: 인증 및 보안

| # | 규범 수준 | 규칙 요약 | Writing | Review | 심각도 |
|---|-----------|-----------|---------|--------|--------|
| 6.1-1 | ✅필수 | 인증 토큰은 Authorization 헤더 사용 | ? | ? | ? |
| 6.1-2 | ❌금지 | API Key를 쿼리 파라미터로 전달 금지 | ? | ? | ? |
| 6.2-1 | ✅필수 | 401 응답에 WWW-Authenticate 헤더 포함 | ? | ? | ? |
| 6.2-2 | ✅필수 | 401(인증 실패) / 403(인가 실패) 정확히 구분 | ? | ? | ? |
| 6.3-1 | ✅필수 | 중복 실행 위험 있는 POST에 Idempotency-Key 지원 | ? | ? | ? |
| 6.3-2 | ✅필수 | Idempotency-Key 값은 클라이언트 생성 UUID v4 | ? | ? | ? |
```

- [ ] **Step 5: 커버리지 합계 섹션 추가**

coverage-map.md 하단에 집계 섹션을 추가한다:

```markdown
---

## 커버리지 합계

| 모드 | COVERED | PARTIAL | MISSING | 합계 |
|------|---------|---------|---------|------|
| Writing | N | N | N | N |
| Review | N | N | N | N |

## Critical 규칙 목록 (즉시 수정 대상)

| # | 규칙 | 모드 | 상태 |
|---|------|------|------|
| ... | ... | ... | ... |
```

실제 판정 결과를 채워 넣는다.

- [ ] **Step 6: 커밋**

```bash
git add docs/evaluation/coverage-map.md
git commit -m "docs: add skill coverage map for restful-api-guidelines"
```

---

## Task 2: 테스트 케이스 작성 — 섹션 2~4 (docs/evaluation/test-cases.md)

**Files:**
- Read: `README.md` (규칙 원문)
- Read: `.claude/skills/restful-api-guidelines.md` (검증 포인트 확인)
- Read: `docs/evaluation/coverage-map.md` (PARTIAL/MISSING 항목 우선 작성)
- Create: `docs/evaluation/test-cases.md`

커버리지 맵에서 PARTIAL/MISSING으로 판정된 규칙에 대한 테스트 케이스를 우선 작성한다. COVERED 규칙도 스킬의 코드 예시가 올바른지 대표 케이스를 포함한다.

- [ ] **Step 1: test-cases.md 파일 생성 및 섹션 2 케이스 작성**

```markdown
# Test Cases — RESTful API Guidelines Skill

**평가 날짜:** 2026-03-19
**형식:** TC-{섹션}-{순번}: {규칙명}

각 케이스는 스킬이 해당 규칙을 올바르게 처리(생성/탐지)하는지 검증한다.

---

## 섹션 2: HTTP 기본 규칙

### TC-2-01: URL 소문자 kebab-case

- 규칙: "URL 경로에는 소문자 kebab-case를 사용한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```
GET /userProfiles
GET /UserProfiles
GET /user_profiles
```

✅ Good:
```
GET /user-profiles
```

- 검증 포인트: Review 체크리스트 "Lowercase kebab-case used in paths" / Writing 코드 예시

---

### TC-2-02: 리소스 컬렉션 복수형 명사

- 규칙: "리소스 컬렉션 이름은 복수형 명사를 사용한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```
GET /article
GET /user/123/comment
```

✅ Good:
```
GET /articles
GET /users/123/comments
```

- 검증 포인트: Review 체크리스트 "Resource names are plural nouns"

---

### TC-2-03: URL에 동사 포함 금지

- 규칙: "URL에 동사를 포함하지 않는다"
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```
POST /createUser
GET /getArticles
DELETE /deleteComment/123
```

✅ Good:
```
POST /users
GET /articles
DELETE /comments/123
```

- 검증 포인트: Review 체크리스트 "No verbs in paths"

---

### TC-2-04: URL 파일 확장자 금지

- 규칙: "URL에 파일 확장자(.json, .xml)를 포함하지 않는다"
- 규범 수준: ❌금지
- 대상 모드: Review
- 스킬 커버: Review: [판정]

❌ Bad:
```
GET /articles.json
GET /users/123.xml
```

✅ Good:
```
GET /articles
Accept: application/json
```

- 검증 포인트: Review 체크리스트 "No file extensions in URLs"

---

### TC-2-05: URL 길이 제한

- 규칙: "URL은 2000자 이하로 유지한다"
- 규범 수준: ⚠️권장
- 대상 모드: Review
- 스킬 커버: Review: [판정]

❌ Bad:
```
GET /articles?filter1=v1&filter2=v2&...&filter50=v50  (2000자 초과)
```

✅ Good:
```
POST /articles/search
Content-Type: application/json
{"filter1": "v1", "filter2": "v2", ...}
```

- 검증 포인트: Review 체크리스트에 해당 항목 있는지 확인

---

### TC-2-06: 201 Created + Location 헤더

- 규칙: "201 Created 응답에는 Location 헤더로 생성된 리소스의 URL을 포함한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad (Kotlin):
```kotlin
@PostMapping
fun createArticle(@RequestBody request: CreateArticleRequest): ResponseEntity<Article> {
    val article = articleService.create(request)
    return ResponseEntity.ok(article)  // 200 반환, Location 없음
}
```

✅ Good (Kotlin):
```kotlin
@PostMapping
fun createArticle(@RequestBody request: CreateArticleRequest): ResponseEntity<Article> {
    val article = articleService.create(request)
    val location = URI.create("/articles/${article.id}")
    return ResponseEntity.created(location).body(article)
}
```

- 검증 포인트: Review 체크리스트 "POST create → 201 + Location header"

---

### TC-2-07: GET/HEAD/DELETE에 body 포함 금지

- 규칙: "GET, HEAD, DELETE 요청에 요청 본문(body)을 포함하지 않는다"
- 규범 수준: ❌금지
- 대상 모드: Review
- 스킬 커버: Review: [판정]

❌ Bad:
```kotlin
@GetMapping("/articles")
fun getArticles(@RequestBody filter: ArticleFilter): ResponseEntity<List<Article>> { ... }
```

✅ Good:
```kotlin
@GetMapping("/articles")
fun getArticles(@RequestParam status: String?): ResponseEntity<List<Article>> { ... }
```

- 검증 포인트: Review 체크리스트에 해당 항목 있는지 확인

---

### TC-2-08: 오류 상황에 200 OK 반환 금지

- 규칙: "오류 상황에 200 OK를 반환하지 않는다"
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```kotlin
@GetMapping("/{id}")
fun getArticle(@PathVariable id: String): ResponseEntity<Map<String, Any>> {
    return ResponseEntity.ok(mapOf("error" to "not found"))  // 에러인데 200
}
```

✅ Good:
```kotlin
@GetMapping("/{id}")
fun getArticle(@PathVariable id: String): ResponseEntity<Article> {
    return ResponseEntity.notFound().build()  // 404
}
```

- 검증 포인트: Review 체크리스트 "200 not returned for error conditions"

---

### TC-2-09: 서버 상태 변경에 쿼리 파라미터 사용 금지

- 규칙: "서버 상태를 변경하는 작업에 쿼리 파라미터를 사용하지 않는다"
- 규범 수준: ❌금지
- 대상 모드: Review
- 스킬 커버: Review: [판정]

❌ Bad:
```
POST /articles?action=publish&id=123
```

✅ Good:
```
POST /articles/123:publish
```

- 검증 포인트: Review 체크리스트에 해당 항목 있는지 확인
```

- [ ] **Step 2: 섹션 3 케이스 작성**

test-cases.md에 섹션 3 케이스를 추가한다:

```markdown
## 섹션 3: REST 원칙

### TC-3-01: null 값 필드 응답 포함 금지

- 규칙: "응답에 null 값 필드를 포함하지 않는다"
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```json
{
  "id": "123",
  "title": "제목",
  "deletedAt": null
}
```

✅ Good:
```json
{
  "id": "123",
  "title": "제목"
}
```

- 검증 포인트: Review 체크리스트 "Null-valued fields excluded from response"

---

### TC-3-02: 읽기 전용 필드 요청 본문 무시

- 규칙: "서버가 관리하는 읽기 전용 필드(id, createdAt, updatedAt)를 클라이언트가 요청 본문에 포함하더라도 이를 무시한다"
- 규범 수준: ✅필수
- 대상 모드: Review
- 스킬 커버: Review: [판정]

❌ Bad:
```kotlin
@PatchMapping("/{id}")
fun updateArticle(@PathVariable id: String, @RequestBody request: ArticleRequest): ResponseEntity<Article> {
    // request.createdAt을 그대로 저장 — 클라이언트가 제어할 수 없어야 함
    return ResponseEntity.ok(articleService.update(id, request))
}
```

✅ Good:
```kotlin
data class UpdateArticleRequest(
    val title: String?,
    val content: String?
    // id, createdAt, updatedAt은 요청 DTO에 포함하지 않음
)
```

- 검증 포인트: Review 체크리스트에 해당 항목 있는지 확인

---

### TC-3-03: RFC 7807 에러 응답 구조

- 규칙: "모든 에러 응답은 RFC 7807 / RFC 9457 표준을 따른다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```kotlin
return ResponseEntity.notFound().body(mapOf("message" to "Not found"))
// Content-Type: application/json 사용, RFC 7807 구조 아님
```

✅ Good:
```kotlin
return ResponseEntity.status(404)
    .contentType(MediaType.parseMediaType("application/problem+json"))
    .body(ProblemDetail(
        type = "https://api.example.com/errors/resource-not-found",
        title = "리소스를 찾을 수 없음",
        status = 404,
        detail = "요청한 게시글을 찾을 수 없습니다.",
        instance = "/articles/999"
    ))
```

- 검증 포인트: Review 체크리스트 "Error responses use RFC 7807/9457 Problem Details structure"

---

### TC-3-04: DELETE 성공 시 204 No Content

- 규칙: "삭제 성공 시 204 No Content를 반환한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```kotlin
@DeleteMapping("/{id}")
fun deleteArticle(@PathVariable id: String): ResponseEntity<Map<String, String>> {
    articleService.delete(id)
    return ResponseEntity.ok(mapOf("message" to "deleted"))  // body 있음, 200
}
```

✅ Good:
```kotlin
@DeleteMapping("/{id}")
fun deleteArticle(@PathVariable id: String): ResponseEntity<Void> {
    articleService.delete(id)
    return ResponseEntity.noContent().build()
}
```

- 검증 포인트: Review 체크리스트 "DELETE success → 204 (no body)"
```

- [ ] **Step 3: 섹션 4 케이스 작성**

test-cases.md에 섹션 4 케이스를 추가한다:

```markdown
## 섹션 4: JSON 규칙

### TC-4-01: JSON 필드 이름 camelCase

- 규칙: "JSON 필드 이름은 camelCase를 사용한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```json
{
  "user_id": "123",
  "created_at": "2024-01-20T10:00:00Z",
  "is_active": true
}
```

✅ Good:
```json
{
  "userId": "123",
  "createdAt": "2024-01-20T10:00:00Z",
  "isActive": true
}
```

- 검증 포인트: Review 체크리스트 "All fields are camelCase"

---

### TC-4-02: 필드명 약어 금지

- 규칙: "필드 이름에 약어를 남용하지 않는다"
- 규범 수준: ❌금지
- 대상 모드: Review
- 스킬 커버: Review: [판정]

❌ Bad:
```json
{
  "usr": "john",
  "ts": "2024-01-20T10:00:00Z",
  "cnt": 5
}
```

✅ Good:
```json
{
  "username": "john",
  "timestamp": "2024-01-20T10:00:00Z",
  "count": 5
}
```

- 검증 포인트: Review 체크리스트에 약어 금지 항목 있는지 확인

---

### TC-4-03: Boolean 필드 is/has/can 접두사

- 규칙: "Boolean 필드 이름은 is, has, can 등의 접두사를 사용한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```json
{
  "active": true,
  "permission": false,
  "editable": true
}
```

✅ Good:
```json
{
  "isActive": true,
  "hasPermission": false,
  "canEdit": true
}
```

- 검증 포인트: Review 체크리스트 "Boolean fields use is/has/can prefix"

---

### TC-4-04: 큰 정수 문자열 반환

- 규칙: "JavaScript의 안전한 정수 범위(2^53 - 1)를 초과하는 큰 정수는 문자열로 반환한다"
- 규범 수준: ⚠️권장
- 대상 모드: Review
- 스킬 커버: Review: [판정]

❌ Bad:
```json
{
  "snowflakeId": 9007199254740993
}
```

✅ Good:
```json
{
  "snowflakeId": "9007199254740993"
}
```

- 검증 포인트: Review 체크리스트에 해당 항목 있는지 확인

---

### TC-4-05: 날짜/시간 RFC 3339 형식 및 UTC

- 규칙: "서버 응답의 모든 시간 값은 UTC(Z)로 반환한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```json
{
  "createdAt": 1705744800000,
  "updatedAt": "2024-01-20 10:00:00"
}
```

✅ Good:
```json
{
  "createdAt": "2024-01-20T10:00:00Z",
  "updatedAt": "2024-01-20T15:30:00Z"
}
```

- 검증 포인트: Review 체크리스트 "Date/time in RFC 3339 format" / "All time values in server response are UTC (Z)"

---

### TC-4-06: Enum UPPER_SNAKE_CASE

- 규칙: "Enum 값은 UPPER_SNAKE_CASE 문자열을 사용한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```json
{
  "status": 1,
  "priority": "hi"
}
```

✅ Good:
```json
{
  "status": "PUBLISHED",
  "priority": "HIGH"
}
```

- 검증 포인트: Review 체크리스트 "Enum values are UPPER_SNAKE_CASE"
```

- [ ] **Step 4: 커밋**

```bash
git add docs/evaluation/test-cases.md
git commit -m "docs: add test cases for sections 2-4 (HTTP rules, REST, JSON)"
```

---

## Task 3: 테스트 케이스 작성 — 섹션 5~6 (docs/evaluation/test-cases.md 계속)

**Files:**
- Read: `README.md`
- Read: `.claude/skills/restful-api-guidelines.md`
- Modify: `docs/evaluation/test-cases.md`

- [ ] **Step 1: 섹션 5 케이스 작성**

test-cases.md에 섹션 5(공통 API 패턴) 케이스를 추가한다:

```markdown
## 섹션 5: 공통 API 패턴

### TC-5-01: 액션 패턴 :action 형태

- 규칙: "액션은 리소스 URL 뒤에 :action 형태로 표현한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```
POST /articles/publish/123
POST /publishArticle
```

✅ Good:
```
POST /articles/123:publish
```

- 검증 포인트: Writing 코드 예시 `@PostMapping("/{id}:publish")` / Review 체크리스트 "No verbs in paths (actions use :action pattern)"

---

### TC-5-02: 컬렉션 응답 top-level array

- 규칙: "컬렉션 조회 응답 본문은 리소스 배열(top-level JSON array)을 반환한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```json
{
  "data": [
    { "id": "1", "title": "첫 번째 글" }
  ],
  "total": 100
}
```

✅ Good:
```
HTTP/1.1 200 OK
X-Total-Count: 100

[
  { "id": "1", "title": "첫 번째 글" }
]
```

- 검증 포인트: Review 체크리스트 "Collection response body is a top-level array (no envelope)"

---

### TC-5-03: 커서 기반 페이지네이션 파라미터

- 규칙: 쿼리 파라미터는 camelCase (pageSize, pageToken, orderBy)
- 규범 수준: ✅필수 (쿼리 파라미터 camelCase)
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```
GET /articles?page_size=20&page_token=abc
```

✅ Good:
```
GET /articles?pageSize=20&pageToken=abc
```

- 검증 포인트: Review 체크리스트 "Query parameters are camelCase (pageSize, pageToken, orderBy)"

---

### TC-5-04: API 버전 URL 경로 포함 금지

- 규칙: "API 버전을 URL 경로에 포함하지 않는다"
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```
GET /v1/articles
GET /v2/users/123
```

✅ Good:
```
GET /articles
X-API-Version: 2024-01-20
```

- 검증 포인트: Review 체크리스트 "No version in URL path (/v1/, /v2/, etc.)"

---

### TC-5-05: 하위 호환/비호환 변경 분류

- 규칙: 하위 호환 변경(선택 필드 추가, 새 엔드포인트 추가)과 비호환 변경(필드 삭제/변경, 필수 필드 추가) 구분
- 규범 수준: ✅필수
- 대상 모드: Review
- 스킬 커버: Review: [판정]

❌ Bad (비호환 변경을 버전 업 없이 수행):
```json
// Before
{ "id": "1", "title": "제목" }

// After — title 제거 (비호환 변경), 버전 업 없이 배포
{ "id": "1", "name": "제목" }
```

✅ Good:
```json
// 비호환 변경 시 새 버전 일자로 X-API-Version 업데이트
// X-API-Version: 2024-06-01
{ "id": "1", "name": "제목" }
```

- 검증 포인트: Review 체크리스트에 하위 호환성 분류 항목 있는지 확인

---

### TC-5-06: Deprecation 응답 헤더

- 규칙: "Deprecated된 API에는 응답 헤더로 알림을 제공한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```
HTTP/1.1 200 OK
Content-Type: application/json
// Deprecation 헤더 없음
```

✅ Good:
```
HTTP/1.1 200 OK
Deprecation: true
Sunset: Sat, 01 Jan 2025 00:00:00 GMT
Link: <https://api.example.com/articles>; rel="successor-version"
```

- 검증 포인트: Writing 코드 예시 `addDeprecationHeaders` 함수

---

### TC-5-07: 429 Retry-After + Problem Details

- 규칙: "429 응답에 Retry-After 헤더를 포함한다" + "429 응답 본문은 RFC 7807 Problem Details 구조"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```
HTTP/1.1 429 Too Many Requests
Content-Type: application/json

{ "message": "Too many requests" }
```

✅ Good:
```
HTTP/1.1 429 Too Many Requests
Content-Type: application/problem+json
Retry-After: 50
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1742342450
RateLimit: limit=100, remaining=0, reset=50
RateLimit-Policy: 100;w=3600

{
  "type": "https://api.example.com/errors/too-many-requests",
  "title": "속도 제한 초과",
  "status": 429,
  "detail": "허용된 요청 한도를 초과했습니다. 50초 후에 다시 시도해 주세요."
}
```

- 검증 포인트: Review 체크리스트 "429 response includes Retry-After header" / "429 response body uses Problem Details structure"

---

### TC-5-08: 장기 실행 작업 201 + 도메인 리소스 status 필드

- 규칙: "장기 실행 작업 요청 시 도메인 리소스를 즉시 생성하고 201 Created + Location 헤더를 반환한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```
POST /reports → 202 Accepted
{ "operationId": "op-123" }  // 범용 operation 객체
```

✅ Good:
```
POST /reports → 201 Created
Location: /reports/123
{ "id": "123", "status": "PENDING" }
```

- 검증 포인트: Review 체크리스트 "Long-running task returns 201 Created + Location header" / "No generic /operations endpoint"
```

- [ ] **Step 2: 섹션 6 케이스 작성**

test-cases.md에 섹션 6(인증/보안) 케이스를 추가한다:

```markdown
## 섹션 6: 인증 및 보안

### TC-6-01: 인증 토큰 Authorization 헤더 사용

- 규칙: "인증 토큰은 Authorization 헤더를 사용한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```
GET /articles?token=eyJhbGci...
GET /articles?apiKey=secret-key
```

✅ Good:
```
GET /articles
Authorization: Bearer eyJhbGci...
```

- 검증 포인트: Review 체크리스트 "Auth token delivered via Authorization header (query parameter forbidden)"

---

### TC-6-02: API Key 쿼리 파라미터 전달 금지

- 규칙: "API Key를 쿼리 파라미터로 전달하지 않는다"
- 규범 수준: ❌금지
- 대상 모드: Review
- 스킬 커버: Review: [판정]

❌ Bad:
```
GET /articles?apiKey=your-api-key-here
```

✅ Good:
```
GET /articles
Authorization: ApiKey your-api-key-here
```

- 검증 포인트: Review 체크리스트에 API Key 쿼리 파라미터 금지 항목 있는지 확인

---

### TC-6-03: 401 응답에 WWW-Authenticate 헤더

- 규칙: "401 응답에는 WWW-Authenticate 헤더를 포함한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad (Kotlin):
```kotlin
return ResponseEntity.status(401)
    .contentType(MediaType.parseMediaType("application/problem+json"))
    .body(errorBody)
// WWW-Authenticate 헤더 누락
```

✅ Good (Kotlin):
```kotlin
return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
    .header("WWW-Authenticate", "Bearer realm=\"api\", error=\"token_expired\"")
    .contentType(MediaType.parseMediaType("application/problem+json"))
    .body(errorBody)
```

- 검증 포인트: Review 체크리스트 "401 response includes WWW-Authenticate header"

---

### TC-6-04: 401 vs 403 구분

- 규칙: 401은 인증 실패, 403은 인가 실패
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```kotlin
// 토큰 만료인데 403 반환
if (token.isExpired()) return ResponseEntity.status(403).body(error)
// 권한 없는데 401 반환
if (!user.hasPermission()) return ResponseEntity.status(401).body(error)
```

✅ Good:
```kotlin
// 토큰 없음/만료 → 401
if (token == null || token.isExpired()) return ResponseEntity.status(401)...
// 권한 없음 → 403
if (!user.hasPermission(resource)) return ResponseEntity.status(403)...
```

- 검증 포인트: Review 체크리스트 "401 (authentication failure) / 403 (authorization failure) properly distinguished"

---

### TC-6-05: Idempotency-Key 지원

- 규칙: "중복 실행 위험이 있는 POST 엔드포인트는 Idempotency-Key 헤더를 지원한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: [판정] / Review: [판정]

❌ Bad:
```kotlin
@PostMapping("/orders")
fun createOrder(@RequestBody request: CreateOrderRequest): ResponseEntity<Order> {
    val order = orderService.create(request)
    return ResponseEntity.created(URI.create("/orders/${order.id}")).body(order)
    // Idempotency-Key 미지원 — 네트워크 오류 시 중복 주문 가능
}
```

✅ Good:
```kotlin
@PostMapping("/orders")
fun createOrder(
    @RequestHeader("Idempotency-Key") idempotencyKey: String?,
    @RequestBody request: CreateOrderRequest
): ResponseEntity<Order> {
    idempotencyKey?.let { key ->
        idempotencyStore.find(key)?.let { return ResponseEntity.status(it.statusCode).body(it.body) }
    }
    val order = orderService.create(request)
    idempotencyKey?.let { idempotencyStore.save(it, 201, order, Duration.ofHours(24)) }
    return ResponseEntity.created(URI.create("/orders/${order.id}")).body(order)
}
```

- 검증 포인트: Review 체크리스트 "Idempotency-Key supported for duplicate-risk POST operations"
```

- [ ] **Step 3: 커밋**

```bash
git add docs/evaluation/test-cases.md
git commit -m "docs: add test cases for sections 5-6 (API patterns, auth/security)"
```

---

## Task 4: 평가 보고서 작성 (docs/evaluation/report.md)

**Files:**
- Read: `docs/evaluation/coverage-map.md` (판정 결과)
- Read: `docs/evaluation/test-cases.md` (스킬 커버 상태)
- Create: `docs/evaluation/report.md`

coverage-map.md와 test-cases.md에서 판정한 COVERED/PARTIAL/MISSING 결과를 집계하여 보고서를 작성한다.

- [ ] **Step 1: 커버리지 수치 집계**

coverage-map.md의 모든 규칙을 모드별로 집계한다:
- Writing 모드: COVERED/PARTIAL/MISSING 개수
- Review 모드: COVERED/PARTIAL/MISSING 개수

- [ ] **Step 2: report.md 작성**

```markdown
# Skill 성능 평가 보고서

**평가 날짜:** 2026-03-19
**평가 대상:** `.claude/skills/restful-api-guidelines.md`
**기준 문서:** `README.md`

---

## 커버리지 요약

| 모드 | COVERED | PARTIAL | MISSING | 합계 | 커버율 |
|------|---------|---------|---------|------|--------|
| Writing | N | N | N | N | N% |
| Review | N | N | N | N | N% |

---

## Critical 문제 (즉시 수정)

> ✅필수 규칙이 MISSING 또는 PARTIAL인 항목

| # | 규칙 ID | 규칙 요약 | 모드 | 상태 | 문제 설명 |
|---|---------|-----------|------|------|-----------|
| 1 | ... | ... | ... | ... | ... |

---

## Minor 문제 (단계적 수정)

> ⚠️권장/❌금지 규칙 누락 또는 코드 예시 오류

| # | 규칙 ID | 규칙 요약 | 모드 | 상태 | 문제 설명 |
|---|---------|-----------|------|------|-----------|
| 1 | ... | ... | ... | ... | ... |

---

## 개선 권고사항

### 즉시 수정 (Critical)

각 Critical 항목에 대해 스킬에 추가해야 할 내용을 구체적으로 기술한다.

### 다음 단계 (Minor)

Minor 항목을 우선순위 순으로 정렬하여 제시한다.

---

## 다음 단계

- [ ] Critical 문제 스킬에 반영 (Task 5)
- [ ] 스킬 변경 후 동일 테스트 케이스로 회귀 검증
- [ ] report.md 커버리지 수치 업데이트
```

- [ ] **Step 3: 커밋**

```bash
git add docs/evaluation/report.md
git commit -m "docs: add skill performance evaluation report"
```

---

## Task 5: Critical 문제 스킬에 반영

**Files:**
- Read: `docs/evaluation/report.md` (Critical 목록)
- Read: `.claude/skills/restful-api-guidelines.md`
- Modify: `.claude/skills/restful-api-guidelines.md`

report.md의 Critical 문제 목록을 하나씩 스킬 파일에 반영한다. 각 수정 후 해당 테스트 케이스를 다시 확인하여 해결됐는지 검증한다.

- [ ] **Step 1: Critical 문제별 스킬 수정**

report.md의 Critical 목록을 순서대로 처리한다:
- Code Writing Mode에 누락된 규칙 → 해당 섹션에 설명 또는 코드 예시 추가
- Code Review Mode 체크리스트에 누락된 항목 → 적절한 카테고리 아래에 체크리스트 항목 추가

각 수정 후 해당 테스트 케이스의 "검증 포인트"를 스킬에서 다시 검색하여 커버 여부를 확인한다.

- [ ] **Step 2: 회귀 검증**

수정된 스킬을 기준으로 Critical 문제로 분류된 모든 테스트 케이스를 재검토한다:
- 스킬 Writing Mode: 각 테스트 케이스의 ✅ Good 코드가 스킬 예시와 일치하거나 가이드라인에 따라 생성 가능한지 확인
- 스킬 Review Mode: 각 테스트 케이스의 ❌ Bad 코드가 체크리스트 항목으로 탐지 가능한지 확인

- [ ] **Step 3: report.md 커버리지 수치 업데이트**

스킬 수정 후 변경된 COVERED/PARTIAL/MISSING 항목을 coverage-map.md와 report.md에 반영한다.

- [ ] **Step 4: 커밋**

```bash
git add .claude/skills/restful-api-guidelines.md docs/evaluation/coverage-map.md docs/evaluation/report.md
git commit -m "fix: apply critical skill improvements from performance evaluation"
```

---

## 검증 요약

각 Task 완료 시 확인 사항:

| Task | 완료 기준 |
|------|-----------|
| Task 1 | coverage-map.md에 모든 규칙의 COVERED/PARTIAL/MISSING 상태가 채워짐 |
| Task 2 | 섹션 2~4의 모든 테스트 케이스에 스킬 커버 상태가 기재됨 |
| Task 3 | 섹션 5~6의 모든 테스트 케이스에 스킬 커버 상태가 기재됨 |
| Task 4 | report.md에 커버리지 수치, Critical/Minor 목록, 개선 권고사항이 완성됨 |
| Task 5 | Critical 문제가 스킬에 반영되고 회귀 검증 완료, report.md 수치 업데이트됨 |
