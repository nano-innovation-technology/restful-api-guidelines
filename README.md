# RESTful API Guidelines

nano-innovation-technology 조직의 RESTful API 설계 가이드라인입니다.

Roy Fielding의 RESTful 원칙을 전제로 하지만 HATEOAS는 구현하지 않습니다.

---

## 규범 수준 표기

| 기호 | 수준 | 설명 |
|------|------|------|
| ✅ **필수** | MUST / DO | 반드시 준수해야 하는 규칙 |
| ⚠️ **권장** | SHOULD / MAY | 가능하면 준수하는 것이 좋은 규칙 |
| ❌ **금지** | MUST NOT / DO NOT | 사용하지 말아야 하는 패턴 |

---

## 목차

1. [개요](#1-개요)
2. [HTTP 기본 규칙](#2-http-기본-규칙)
   - [URL 설계](#21-url-설계)
   - [HTTP 요청/응답 패턴](#22-http-요청응답-패턴)
   - [쿼리 파라미터](#23-쿼리-파라미터)
   - [HTTP 헤더](#24-http-헤더)
3. [REST 원칙](#3-rest-원칙)
   - [리소스 스키마](#31-리소스-스키마)
   - [필드 변경 가능성](#32-필드-변경-가능성)
   - [생성/수정/대체 처리](#33-생성수정대체-처리)
   - [에러 처리](#34-에러-처리)
4. [JSON 규칙](#4-json-규칙)
   - [필드 네이밍](#41-필드-네이밍)
   - [타입 시스템](#42-타입-시스템)
   - [날짜와 시간](#43-날짜와-시간)
   - [Enum 처리](#44-enum-처리)
5. [공통 API 패턴](#5-공통-api-패턴)
   - [액션 수행](#51-액션-수행)
   - [컬렉션 및 페이지네이션](#52-컬렉션-및-페이지네이션)
   - [필터링과 정렬](#53-필터링과-정렬)
   - [API 버전 관리](#54-api-버전-관리)
   - [Deprecation](#55-deprecation)
   - [속도 제한](#56-속도-제한)

---

## 1. 개요

### 목적

이 가이드라인은 nano-innovation-technology 조직 내 모든 RESTful API의 일관성, 예측 가능성, 유지보수성을 보장하기 위해 작성되었습니다.

API는 개발자가 소비하는 제품입니다. 좋은 API는 직관적으로 이해 가능하고, 오류가 발생했을 때 명확한 메시지를 제공하며, 버전이 바뀌어도 하위 호환성을 유지합니다.

### 적용 범위

- 조직 내 신규 개발되는 모든 HTTP/HTTPS API
- 기존 API 개선 시 가능한 한 적용

---

## 2. HTTP 기본 규칙

### 2.1 URL 설계

#### 기본 구조

```
https://{host}/{service-root}/{resource-collection}/{resource-id}
```

예시:

```
https://api.example.com/users/articles/123
```

#### URL 케이싱

✅ **필수**: URL 경로에는 소문자 kebab-case를 사용한다.

```
# Good
GET /user-profiles
GET /product-categories/123

# Bad
GET /userProfiles
GET /UserProfiles
GET /user_profiles
```

✅ **필수**: 리소스 컬렉션 이름은 복수형 명사를 사용한다.

```
# Good
GET /articles
GET /users/123/comments

# Bad
GET /article
GET /user/123/comment
```

❌ **금지**: URL에 동사를 포함하지 않는다. 액션은 HTTP 메서드로 표현한다.

```
# Bad
POST /createUser
GET /getArticles
DELETE /deleteComment/123

# Good
POST /users
GET /articles
DELETE /comments/123
```

❌ **금지**: URL에 파일 확장자(`.json`, `.xml`)를 포함하지 않는다. 콘텐츠 협상은 `Accept` 헤더를 사용한다.

#### 허용 문자

✅ **필수**: URL 경로 세그먼트에는 ASCII 영소문자, 숫자, 하이픈(`-`)만 사용한다.

✅ **필수**: 쿼리 파라미터 이름은 camelCase를 사용한다.

```
# Good
GET /articles?pageSize=20&sortOrder=desc

# Bad
GET /articles?page_size=20&sort_order=desc
```

#### URL 길이

⚠️ **권장**: URL은 2000자 이하로 유지한다. 그 이상이 필요한 경우 쿼리 파라미터를 요청 본문으로 이동하는 것을 고려한다.

---

### 2.2 HTTP 요청/응답 패턴

#### HTTP 메서드

| 메서드 | 의미 | 멱등성 | 안전성 |
|--------|------|--------|--------|
| GET | 리소스 조회 | ✅ | ✅ |
| POST | 리소스 생성 또는 액션 수행 | ❌ | ❌ |
| PUT | 리소스 완전 대체 | ✅ | ❌ |
| PATCH | 리소스 부분 수정 | ❌ | ❌ |
| DELETE | 리소스 삭제 | ✅ | ❌ |
| HEAD | 헤더만 조회 | ✅ | ✅ |

✅ **필수**: GET 요청은 서버 상태를 변경하지 않는다.

✅ **필수**: PUT 요청은 멱등적으로 동작한다 — 같은 요청을 여러 번 보내도 결과가 동일해야 한다.

⚠️ **권장**: 부분 수정에는 PUT 대신 PATCH를 사용한다.

❌ **금지**: GET, HEAD, DELETE 요청에 요청 본문(body)을 포함하지 않는다.

#### 상태 코드

✅ **필수**: 아래 표준 HTTP 상태 코드를 정확한 의미에 맞게 사용한다.

**2xx 성공**

| 코드 | 의미 | 사용 시점 |
|------|------|-----------|
| 200 OK | 성공 | GET, PUT, PATCH, POST(액션) 성공 |
| 201 Created | 생성됨 | POST로 리소스 생성 성공 |
| 202 Accepted | 수락됨 | 비동기 작업 수락 |
| 204 No Content | 내용 없음 | DELETE 성공, 응답 본문 없음 |

**4xx 클라이언트 오류**

| 코드 | 의미 | 사용 시점 |
|------|------|-----------|
| 400 Bad Request | 잘못된 요청 | 요청 형식 오류, 유효성 검사 실패 |
| 401 Unauthorized | 인증 필요 | 인증 토큰 없음 또는 만료 |
| 403 Forbidden | 접근 금지 | 인증은 되었지만 권한 없음 |
| 404 Not Found | 찾을 수 없음 | 리소스 존재하지 않음 |
| 409 Conflict | 충돌 | 중복 리소스, 낙관적 잠금 실패 |
| 422 Unprocessable Entity | 처리 불가 | 의미론적 유효성 검사 실패 |
| 429 Too Many Requests | 요청 과다 | 속도 제한 초과 |

**5xx 서버 오류**

| 코드 | 의미 | 사용 시점 |
|------|------|-----------|
| 500 Internal Server Error | 서버 오류 | 예기치 못한 서버 오류 |
| 503 Service Unavailable | 서비스 불가 | 일시적 서버 과부하 또는 유지보수 |

✅ **필수**: 201 Created 응답에는 `Location` 헤더로 생성된 리소스의 URL을 포함한다.

```
HTTP/1.1 201 Created
Location: https://api.example.com/users/articles/456
Content-Type: application/json

{
  "id": "456",
  "title": "새 글 제목"
}
```

⚠️ **권장**: 202 Accepted 응답에는 작업 상태를 조회할 수 있는 `Location` 또는 `Operation-Location` 헤더를 포함한다.

❌ **금지**: 오류 상황에 200 OK를 반환하지 않는다.

---

### 2.3 쿼리 파라미터

✅ **필수**: 쿼리 파라미터 이름은 camelCase를 사용한다.

✅ **필수**: 동일한 파라미터 이름을 반복하여 배열 값을 전달한다.

```
GET /articles?tag=tech&tag=design
```

⚠️ **권장**: 쿼리 파라미터는 선택적(optional)으로 설계한다. 필수 값은 경로(path)에 포함한다.

⚠️ **권장**: 쿼리 파라미터에 민감한 정보(비밀번호, 토큰 등)를 포함하지 않는다. 이는 서버 로그에 기록될 수 있다.

❌ **금지**: 서버 상태를 변경하는 작업에 쿼리 파라미터를 사용하지 않는다.

---

### 2.4 HTTP 헤더

#### 요청 헤더

✅ **필수**: 요청 본문이 있는 경우 `Content-Type` 헤더를 포함한다.

```
Content-Type: application/json
```

⚠️ **권장**: 응답 형식 협상을 위해 `Accept` 헤더를 사용한다.

```
Accept: application/json
```

#### 응답 헤더

✅ **필수**: 응답 본문이 있는 경우 `Content-Type` 헤더를 포함한다.

⚠️ **권장**: 캐싱 전략을 명시하기 위해 `Cache-Control` 헤더를 사용한다.

```
Cache-Control: no-cache
Cache-Control: max-age=3600
```

⚠️ **권장**: 컬렉션 페이지네이션 응답에는 RFC 5988 `Link` 헤더를 사용한다.

```
Link: <https://api.example.com/articles?pageSize=20&pageToken=abc>; rel="next",
      <https://api.example.com/articles?pageSize=20>; rel="first"
```

⚠️ **권장**: 전체 항목 수를 제공할 때 `X-Total-Count` 헤더를 사용한다.

```
X-Total-Count: 100
```

#### 커스텀 헤더

⚠️ **권장**: 커스텀 헤더 이름은 `X-` 접두사를 사용한다.

```
X-Request-Id: abc-123
X-Correlation-Id: xyz-789
```

❌ **금지**: 표준 HTTP 헤더의 의미를 재정의하지 않는다.

---

## 3. REST 원칙

### 3.1 리소스 스키마

리소스는 서비스가 노출하는 핵심 엔티티입니다. 각 리소스는 고유한 URL을 통해 접근 가능해야 합니다.

✅ **필수**: 모든 리소스는 고유 식별자(`id`)를 가진다.

✅ **필수**: 리소스 스키마는 일관된 구조를 유지한다.

**표준 리소스 필드**

| 필드 | 타입 | 설명 |
|------|------|------|
| `id` | string | 리소스 고유 식별자 |
| `createdAt` | string (RFC 3339) | 생성 시각 |
| `updatedAt` | string (RFC 3339) | 마지막 수정 시각 |

예시:

```json
{
  "id": "123",
  "title": "RESTful API 설계",
  "content": "...",
  "createdAt": "2024-01-15T09:00:00Z",
  "updatedAt": "2024-01-20T14:30:00Z"
}
```

⚠️ **권장**: 리소스 식별자는 불투명한(opaque) 문자열로 설계한다. 클라이언트가 식별자 구조를 파싱하거나 의존하지 않도록 한다.

❌ **금지**: 응답에 null 값 필드를 포함하지 않는다. 값이 없는 필드는 응답에서 제외한다.

```json
// Bad
{
  "id": "123",
  "title": "제목",
  "deletedAt": null
}

// Good
{
  "id": "123",
  "title": "제목"
}
```

---

### 3.2 필드 변경 가능성

필드는 생성 후 변경 가능 여부에 따라 분류됩니다.

| 분류 | 설명 | 예시 |
|------|------|------|
| **생성 시 지정 (Create-only)** | 생성 시에만 설정 가능, 이후 변경 불가 | `id`, `createdAt` |
| **읽기 전용 (Read-only)** | 서버가 관리, 클라이언트 수정 불가 | `updatedAt` |
| **변경 가능 (Mutable)** | 클라이언트가 수정 가능 | `title`, `content` |

✅ **필수**: 서버가 관리하는 읽기 전용 필드(`id`, `createdAt`, `updatedAt`)를 클라이언트가 요청 본문에 포함하더라도 이를 무시한다.

⚠️ **권장**: API 문서에서 각 필드의 변경 가능성을 명시한다.

---

### 3.3 생성/수정/대체 처리

#### POST — 리소스 생성

✅ **필수**: 새 리소스 생성 성공 시 201 Created와 생성된 리소스를 반환한다.

```
POST /articles
Content-Type: application/json

{
  "title": "새 글 제목",
  "content": "본문 내용"
}

---

HTTP/1.1 201 Created
Location: /articles/456
Content-Type: application/json

{
  "id": "456",
  "title": "새 글 제목",
  "content": "본문 내용",
  "createdAt": "2024-01-20T10:00:00Z",
  "updatedAt": "2024-01-20T10:00:00Z"
}
```

#### PUT — 리소스 완전 대체

✅ **필수**: PUT 요청은 리소스 전체를 대체한다. 요청 본문에 포함되지 않은 변경 가능 필드는 기본값 또는 null로 처리한다.

✅ **필수**: PUT 요청은 멱등적으로 동작해야 한다.

#### PATCH — 부분 수정

⚠️ **권장**: PATCH 요청은 요청 본문에 포함된 필드만 수정한다.

```
PATCH /articles/456
Content-Type: application/json

{
  "title": "수정된 제목"
}
```

#### DELETE — 리소스 삭제

✅ **필수**: 삭제 성공 시 204 No Content를 반환한다.

⚠️ **권장**: 이미 삭제된 리소스에 대한 재삭제 요청은 404 Not Found 또는 204 No Content를 반환한다. 서비스 특성에 따라 결정한다.

---

### 3.4 에러 처리

#### 에러 응답 구조

✅ **필수**: 모든 에러 응답은 RFC 7807 / RFC 9457 (Problem Details for HTTP APIs) 표준을 따른다.

✅ **필수**: 에러 응답의 `Content-Type`은 `application/problem+json`을 사용한다.

```json
{
  "type": "https://api.example.com/errors/resource-not-found",
  "title": "리소스를 찾을 수 없음",
  "status": 404,
  "detail": "요청한 게시글을 찾을 수 없습니다.",
  "instance": "/articles/999",
  "traceId": "abc-123-xyz"
}
```

| 필드 | 필수 여부 | 설명 |
|------|-----------|------|
| `type` | ✅ 필수 | 에러 유형을 식별하는 URI (문서 링크 역할, `about:blank` 허용) |
| `title` | ✅ 필수 | 에러 유형의 짧은 요약 (사람이 읽을 수 있는 텍스트) |
| `status` | ✅ 필수 | HTTP 상태 코드 (숫자) |
| `detail` | ✅ 필수 | 이 요청에 대한 구체적인 에러 설명 (사용자가 이해할 수 있는 언어) |
| `instance` | ⚠️ 권장 | 문제가 발생한 요청 경로 |
| `errors` | ⚠️ 권장 | 확장 필드 — 필드 수준 유효성 검사 상세 목록 |
| `traceId` | ⚠️ 권장 | 확장 필드 — 요청 추적 ID (디버깅용) |

> **참조**: [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807), [RFC 9457](https://datatracker.ietf.org/doc/html/rfc9457)

⚠️ **권장**: 유효성 검사 실패 시 모든 오류 필드를 한 번에 반환한다 (하나씩 반환하지 않는다).

```json
{
  "type": "https://api.example.com/errors/validation-failed",
  "title": "유효성 검사 실패",
  "status": 400,
  "detail": "요청 데이터 유효성 검사에 실패했습니다.",
  "instance": "/articles",
  "errors": [
    { "field": "title", "message": "제목은 필수 입력값입니다." },
    { "field": "content", "message": "본문은 10자 이상이어야 합니다." }
  ],
  "traceId": "abc-123-xyz"
}
```

❌ **금지**: 에러 응답에 스택 트레이스, 내부 시스템 경로, DB 오류 메시지 등 내부 구현 정보를 노출하지 않는다.

---

## 4. JSON 규칙

### 4.1 필드 네이밍

✅ **필수**: JSON 필드 이름은 camelCase를 사용한다.

```json
// Good
{
  "userId": "123",
  "createdAt": "2024-01-20T10:00:00Z",
  "isActive": true
}

// Bad
{
  "user_id": "123",
  "created_at": "2024-01-20T10:00:00Z",
  "is_active": true
}
```

✅ **필수**: 필드 이름은 영소문자로 시작한다.

❌ **금지**: 필드 이름에 약어를 남용하지 않는다. 명확한 전체 단어를 우선한다.

```json
// Bad
{
  "usr": "john",
  "ts": "2024-01-20T10:00:00Z",
  "cnt": 5
}

// Good
{
  "username": "john",
  "timestamp": "2024-01-20T10:00:00Z",
  "count": 5
}
```

---

### 4.2 타입 시스템

#### Boolean

✅ **필수**: Boolean 값에는 JSON `true`/`false`를 사용한다. 문자열 `"true"`/`"false"` 또는 숫자 `1`/`0`을 사용하지 않는다.

✅ **필수**: Boolean 필드 이름은 `is`, `has`, `can` 등의 접두사를 사용한다.

```json
{
  "isActive": true,
  "hasPermission": false,
  "canEdit": true
}
```

#### Number

✅ **필수**: 숫자 값은 JSON number 타입을 사용한다.

⚠️ **권장**: JavaScript의 안전한 정수 범위(2^53 - 1)를 초과하는 큰 정수는 문자열로 반환한다.

```json
{
  "count": 42,
  "price": 19.99,
  "largeId": "9007199254740993"
}
```

#### String

⚠️ **권장**: 빈 문자열(`""`)과 `null`을 구분하여 사용한다. 의미 있는 "값 없음"에는 필드를 제외하고, 의도적으로 빈 값임을 나타낼 때만 빈 문자열을 사용한다.

---

### 4.3 날짜와 시간

✅ **필수**: 모든 날짜/시간 값은 RFC 3339 형식(ISO 8601 프로파일)의 문자열로 표현한다.

✅ **필수**: 시간대(timezone)가 있는 경우 반드시 포함한다. UTC인 경우 `Z`를 사용한다.

✅ **필수**: 서버 응답의 모든 시간 값은 UTC(`Z`)로 반환한다. 클라이언트가 로컬 시간대로 변환한다.

⚠️ **권장**: 클라이언트 요청의 시간 값도 UTC(`Z`)로 전송한다. 오프셋이 포함된 경우 서버가 UTC로 정규화하여 저장한다.

⚠️ **권장**: 날짜만 필요한 필드(생년월일 등)는 `YYYY-MM-DD` 형식을 사용하며 시간대를 포함하지 않는다.

❌ **금지**: Unix timestamp(epoch milliseconds/seconds)를 기본 시간 형식으로 사용하지 않는다.

#### 서버 응답 예시

```json
{
  "createdAt": "2024-01-20T10:00:00Z",
  "scheduledAt": "2024-01-25T00:30:00Z",
  "birthDate": "1990-05-15"
}
```

#### 클라이언트 요청 예시

```json
// ⚠️ 권장: UTC
{ "scheduledAt": "2024-01-25T00:30:00Z" }

// 허용: 오프셋 포함 → 서버가 UTC로 정규화 저장
{ "scheduledAt": "2024-01-25T09:30:00+09:00" }
```

#### 서버 정규화 규칙

✅ **필수**: 클라이언트가 오프셋이 포함된 시간을 전송하면, 서버는 이를 UTC로 변환하여 저장한다. 에러를 반환하지 않는다.

✅ **필수**: 서버가 정규화한 후의 응답은 항상 UTC(`Z`)로 반환한다.

⚠️ **권장**: 시간대 정보가 비즈니스적으로 중요한 경우(예: 사용자의 원래 시간대 보존), 별도의 `timeZone` 필드를 사용한다.

```json
{
  "scheduledAt": "2024-01-25T00:30:00Z",
  "timeZone": "Asia/Seoul"
}

---

### 4.4 Enum 처리

✅ **필수**: Enum 값은 UPPER_SNAKE_CASE 문자열을 사용한다.

```json
{
  "status": "PUBLISHED",
  "priority": "HIGH"
}
```

⚠️ **권장**: 클라이언트가 알 수 없는 Enum 값을 수신할 수 있도록 설계한다. 새로운 Enum 값이 추가될 때 기존 클라이언트가 깨지지 않도록 처리한다.

❌ **금지**: Enum 값으로 숫자나 불명확한 약어를 사용하지 않는다.

```json
// Bad
{
  "status": 1,
  "priority": "hi"
}

// Good
{
  "status": "PUBLISHED",
  "priority": "HIGH"
}
```

---

## 5. 공통 API 패턴

### 5.1 액션 수행

CRUD로 표현하기 어려운 동작(예: 승인, 전송, 잠금)에는 액션 패턴을 사용합니다.

✅ **필수**: 액션은 리소스 URL 뒤에 `:action` 형태로 표현한다.

```
POST /articles/123:publish
POST /users/456:deactivate
POST /orders/789:cancel
```

✅ **필수**: 액션 엔드포인트에는 POST 메서드를 사용한다.

⚠️ **권장**: 액션 이름은 동사 원형을 사용한다 (publish, cancel, approve).

**요청 예시:**

```
POST /articles/123:publish
Content-Type: application/json

{}

---

HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "123",
  "status": "PUBLISHED",
  "publishedAt": "2024-01-20T15:00:00Z"
}
```

---

### 5.2 컬렉션 및 페이지네이션

#### 컬렉션 응답 구조

✅ **필수**: 컬렉션 조회 응답 본문은 리소스 배열(top-level JSON array)을 반환한다.

```
HTTP/1.1 200 OK
Link: <https://api.example.com/articles?pageSize=20&pageToken=abc>; rel="next",
      <https://api.example.com/articles?pageSize=20>; rel="first",
      <https://api.example.com/articles?pageSize=20&pageToken=xyz>; rel="last"
X-Total-Count: 100

[
  { "id": "1", "title": "첫 번째 글" },
  { "id": "2", "title": "두 번째 글" }
]
```

| 헤더 | 필수 여부 | 설명 |
|------|-----------|------|
| `Link` | ⚠️ 권장 | 페이지네이션 네비게이션 (RFC 5988) |
| `X-Total-Count` | ⚠️ 권장 | 전체 항목 수 |

| rel 값 | 설명 |
|--------|------|
| `next` | 다음 페이지 |
| `prev` | 이전 페이지 |
| `first` | 첫 번째 페이지 |
| `last` | 마지막 페이지 |

#### 커서 기반 페이지네이션 (권장)

⚠️ **권장**: 대용량 데이터에는 오프셋 기반 대신 커서 기반 페이지네이션을 사용한다.

**요청:**

```
GET /articles?pageSize=20&pageToken=eyJwYWdlIjoyf...
```

**응답:**

```
HTTP/1.1 200 OK
Link: <https://api.example.com/articles?pageSize=20&pageToken=abc>; rel="next",
      <https://api.example.com/articles?pageSize=20>; rel="first"

[
  { "id": "1", "title": "첫 번째 글" },
  ...
]
```

✅ **필수**: 다음 페이지가 없을 때 `Link` 헤더에서 `rel="next"`를 제외한다.

#### 오프셋 기반 페이지네이션

소규모 데이터셋에서는 오프셋 기반 페이지네이션도 허용됩니다.

**요청:**

```
GET /articles?page=2&pageSize=20
```

**응답:**

```
HTTP/1.1 200 OK
Link: <https://api.example.com/articles?pageSize=20&page=1>; rel="first",
      <https://api.example.com/articles?pageSize=20&page=1>; rel="prev",
      <https://api.example.com/articles?pageSize=20&page=3>; rel="next",
      <https://api.example.com/articles?pageSize=20&page=5>; rel="last"
X-Total-Count: 100

[
  { "id": "21", "title": "스물한 번째 글" },
  ...
]
```

⚠️ **권장**: 기본 페이지 크기(`pageSize`)는 20으로 설정한다.

⚠️ **권장**: 최대 페이지 크기는 100으로 제한한다.

---

### 5.3 필터링과 정렬

#### 필터링

⚠️ **권장**: 필터는 쿼리 파라미터로 표현한다.

```
GET /articles?status=PUBLISHED&authorId=123
GET /articles?createdAfter=2024-01-01T00:00:00Z
```

#### 정렬

⚠️ **권장**: 정렬은 `orderBy` 파라미터를 사용하며, 필드명과 방향을 조합하여 표현한다.

```
GET /articles?orderBy=createdAt:desc
GET /articles?orderBy=title:asc
GET /articles?orderBy=createdAt:desc,title:asc
```

⚠️ **권장**: 기본 정렬 기준은 API 문서에 명시한다.

---

### 5.4 API 버전 관리

#### 버전 표기

❌ **금지**: API 버전을 URL 경로에 포함하지 않는다.

✅ **필수**: `X-API-Version` 헤더에 ISO 8601 (`YYYY-MM-DD`) 형식의 날짜로 버전을 지정한다.

```
X-API-Version: 2024-01-20
```

#### 하위 호환성

✅ **필수**: 동일 버전 내에서 하위 호환성을 유지한다.

**하위 호환 변경 (허용):**
- 새 선택적(optional) 필드 추가
- 새 엔드포인트 추가
- 새 Enum 값 추가

**하위 비호환 변경 (버전 업 필요):**
- 필드 이름 변경 또는 삭제
- 필드 타입 변경
- 필수 필드 추가
- Enum 값 제거
- 상태 코드 의미 변경

⚠️ **권장**: 버전 업 전 최소 6개월 이전 버전을 유지한다.

---

### 5.5 Deprecation

✅ **필수**: Deprecated된 API에는 응답 헤더로 알림을 제공한다.

```
Deprecation: true
Sunset: Sat, 01 Jan 2025 00:00:00 GMT
Link: <https://api.example.com/users/articles>; rel="successor-version"
```

⚠️ **권장**: Deprecation 공지는 종료일 최소 6개월 전에 알린다.

⚠️ **권장**: Deprecated API 호출 시 `Deprecation`, `Sunset`, `Link` 헤더로 클라이언트에 공지한다.

```
HTTP/1.1 200 OK
Deprecation: true
Sunset: Sat, 01 Jan 2025 00:00:00 GMT
Link: <https://api.example.com/users/articles>; rel="successor-version"

[
  { "id": "1", "title": "항목 1" },
  ...
]
```

---

### 5.6 속도 제한

API 서버는 클라이언트별 요청 빈도를 제한하여 서비스 안정성을 보장한다.

#### 응답 헤더

✅ **필수**: 속도 제한이 적용되는 모든 응답에 다음 헤더를 포함한다.

**레거시 헤더 (X-RateLimit-\*)**

| 헤더 | 설명 | 예시 |
|------|------|------|
| `X-RateLimit-Limit` | 시간 창(window) 내 허용되는 최대 요청 수 | `100` |
| `X-RateLimit-Remaining` | 현재 시간 창에서 남은 요청 수 | `99` |
| `X-RateLimit-Reset` | 시간 창이 초기화되는 시각 (Unix timestamp, 초 단위) | `1742342450` |

**IETF 표준 헤더 (draft-ietf-httpapi-ratelimit-headers)**

| 헤더 | 설명 | 예시 |
|------|------|------|
| `RateLimit` | 현재 속도 제한 상태 (Structured Field) | `limit=100, remaining=99, reset=50` |
| `RateLimit-Policy` | 적용 중인 속도 제한 정책 | `100;w=3600` |

> **참고**: `RateLimit` 헤더의 `reset` 값은 시간 창 초기화까지 남은 **초(delta-seconds)**이며, `X-RateLimit-Reset`은 **Unix timestamp**이다. 혼동에 주의한다.
>
> **`RateLimit-Policy` 구조**: `100;w=3600` — `100`은 허용 최대 요청 수, `w=3600`은 시간 창 크기(window, 초 단위). 모든 응답에 포함한다.

**정상 응답 예시:**

```
HTTP/1.1 200 OK
Content-Type: application/json
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1742342450
RateLimit: limit=100, remaining=99, reset=50
RateLimit-Policy: 100;w=3600

[
  { "id": "1", "title": "항목 1" }
]
```

#### 429 Too Many Requests 응답

✅ **필수**: 속도 제한 초과 시 `429 Too Many Requests`를 반환한다.

✅ **필수**: 429 응답에 `Retry-After` 헤더를 포함한다. 값은 재시도까지 대기해야 하는 초(delta-seconds)를 사용한다.

✅ **필수**: 429 응답 본문은 RFC 7807 Problem Details 구조를 사용한다.

**429 응답 예시:**

```
HTTP/1.1 429 Too Many Requests
Content-Type: application/problem+json
Retry-After: 50
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1742342450
RateLimit: limit=100, remaining=0, reset=50
RateLimit-Policy: 100;w=3600
```

```json
{
  "type": "https://api.example.com/errors/too-many-requests",
  "title": "속도 제한 초과",
  "status": 429,
  "detail": "허용된 요청 한도를 초과했습니다. 50초 후에 다시 시도해 주세요."
}
```

#### 클라이언트 재시도 전략

✅ **필수**: 클라이언트는 429 응답 수신 시 `Retry-After` 헤더 값만큼 대기한 후 재시도한다.

⚠️ **권장**: `Retry-After` 헤더가 없거나 기타 일시적 오류(503 등) 발생 시, 지수 백오프(exponential backoff) + 지터(jitter) 전략을 사용한다. `attempt`는 1부터 시작하는 재시도 횟수(1 = 첫 번째 재시도).

```
대기 시간 = min(maxDelay, baseDelay × 2^(attempt - 1)) + random(0, jitterRange)
```

예시: attempt=1 → `min(60, 1 × 2^0) + random(0,1)` = 1~2초

| 파라미터 | 권장 값 | 설명 |
|----------|---------|------|
| `baseDelay` | 1초 | 첫 번째 재시도 대기 시간 |
| `maxDelay` | 60초 | 최대 대기 시간 상한 |
| `jitterRange` | 0 ~ 1초 | 무작위 지연 (thundering herd 방지) |
| 최대 재시도 횟수 | 3 ~ 5회 | 무한 재시도 방지 |

❌ **금지**: 429 응답 수신 시 즉시 재시도하거나 고정 간격으로 반복 재시도하지 않는다.

❌ **금지**: `Retry-After` 헤더가 있을 때 해당 값을 무시하고 자체 대기 시간을 사용하지 않는다.

---

## 참고 자료

- [Microsoft Azure REST API Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md)
- [RFC 3339 - Date and Time on the Internet](https://datatracker.ietf.org/doc/html/rfc3339)
- [HTTP/1.1 (RFC 7231)](https://datatracker.ietf.org/doc/html/rfc7231)
- [JSON:API Specification](https://jsonapi.org/)
- [Day1, 2-2. 그런 REST API로 괜찮은가](https://www.youtube.com/watch?v=RP_f5dMoHFc)
- [Architectural Styles and the Design of Network-based Software Architectures - Roy Fielding](https://roy.gbiv.com/pubs/dissertation/fielding_dissertation.pdf)
- [RFC 8288 - Web Linking](https://datatracker.ietf.org/doc/html/rfc8288)
- [RFC 6585 - Additional HTTP Status Codes (429)](https://datatracker.ietf.org/doc/html/rfc6585#section-4)
- [IETF draft-ietf-httpapi-ratelimit-headers - RateLimit Header Fields](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/)
