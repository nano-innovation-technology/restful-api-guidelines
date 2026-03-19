# Rate Limiting (속도 제한) 가이드라인 설계 스펙

**날짜**: 2026-03-19
**상태**: 승인됨
**대상 섹션**: Section 5.6 속도 제한

---

## 배경 및 목적

현재 RESTful API 가이드라인은 429 상태 코드를 "요청 과다" 항목으로만 정의하고 있다. Rate Limiting에 관한 헤더 규칙, 응답 본문 형식, 클라이언트 재시도 전략이 없어 구현 일관성이 부족하다.

이 스펙은 Section 5 "공통 API 패턴"에 **5.6 속도 제한** 섹션을 추가하여 이를 해결한다.

---

## 설계 결정 사항

### 1. 응답 헤더: X-RateLimit-* + IETF RateLimit 양쪽 모두 필수

**결정**: 두 헤더 체계 모두 ✅ 필수로 정의한다. 속도 제한이 적용되는 **모든 응답**(정상 응답 포함)에 포함한다.

**이유**:
- `X-RateLimit-*`: GitHub, Stripe 등 대부분의 API가 사용하는 현재 사실상 표준. 기존 클라이언트 라이브러리와 호환성이 높다.
- IETF `RateLimit` / `RateLimit-Policy`: [draft-ietf-httpapi-ratelimit-headers-10](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/) 기반 표준화 진행 중. 의미가 명확하게 정의되어 상호운용성 문제를 해결한다.
- 양쪽을 모두 요구함으로써 현재 호환성과 미래 표준 준비를 동시에 확보한다.

**주의사항**: `X-RateLimit-Reset`은 Unix timestamp, IETF `RateLimit.reset`은 delta-seconds(남은 초)로 의미가 다르다. 가이드라인에 명시한다.

**`RateLimit-Policy` 발송 빈도**: 모든 응답에 포함한다. 정책이 고정된 경우에도 매 응답에 포함하여 클라이언트가 항상 정책을 파악할 수 있도록 한다.

**`RateLimit-Policy` 헤더 구조**: `100;w=3600` 형식으로, `100`은 허용 최대 요청 수(limit), `w=3600`은 시간 창 크기(window, 초 단위)를 의미한다.

### 2. 429 응답 본문: Problem Details 표준 구조만 사용 (확장 필드 없음)

**결정**: RFC 7807 Problem Details 구조를 그대로 사용한다. `retryAfter`, `limit`, `remaining` 확장 필드는 추가하지 않는다.

**이유**:
- `retryAfter`는 `Retry-After` 헤더와 중복이다. HTTP에서 재시도 정보의 표준 위치는 헤더다.
- `limit`, `remaining`은 이미 `X-RateLimit-*` 및 `RateLimit` 헤더에 있다.
- 429 맥락에서 `remaining`은 항상 0이어서 정보 가치가 없다.
- YAGNI: 헤더로 충분한 정보를 본문에 중복 기재할 필요가 없다.
- 기존 가이드라인의 에러 처리(3.4절)와 일관성을 유지한다.

**`instance` 및 `traceId` 처리**: 기존 3.4절 에러 처리 규칙을 그대로 따른다. 두 필드 모두 ⚠️ 권장이며 429 응답에서도 포함 가능하다. 예시는 필수 필드만 표시하나, 두 필드 사용을 배제하지 않는다.

### 3. Retry-After 헤더: delta-seconds 형식, 필수

**결정**: ✅ 필수. delta-seconds(초 단위 정수) 형식을 사용한다.

**이유**: RFC 9110 Section 10.2.3은 HTTP-date와 delta-seconds 두 형식을 허용한다. Delta-seconds는 클라이언트-서버 간 시계 동기화 문제를 피하고, IETF RateLimit 헤더의 `reset` 값과 같은 단위를 사용하여 일관성이 있다.

### 4. 클라이언트 재시도 전략: 포함

**결정**: `Retry-After` 준수를 ✅ 필수, exponential backoff + jitter를 ⚠️ 권장으로 포함한다.

**이유**: 서버가 `Retry-After`를 필수로 발송하면 클라이언트도 이를 반드시 준수해야 Rate Limiting 목적(thundering herd 방지, 서버 보호)이 달성된다. `Retry-After`가 없는 경우를 대비한 fallback 전략(exponential backoff + jitter)은 구현 방식이 다양하므로 ⚠️ 권장으로 정의한다.

---

## 변경 대상

### README.md

1. **목차 (line 42)**: `- [속도 제한](#56-속도-제한)` 항목 추가
2. **Section 5.6 삽입** (line 831~833 사이):
   - 응답 헤더 규칙 및 예시 (정상 응답)
   - 429 응답 규칙 및 예시 (헤더 + Problem Details 본문)
   - 클라이언트 재시도 전략 (수식 + 파라미터 표 + 금지 항목)
3. **참고 자료**: RFC 6585, IETF ratelimit-headers draft 추가

### .claude/skills/restful-api-guidelines.md

1. **코드 작성 모드**: `### 속도 제한` 서브섹션 추가 (응답 헤더 패턴 + 429 예시 + 재시도 규칙)
2. **리뷰 체크리스트**: `#### 속도 제한` 항목 6개 추가

---

## 상세 내용

### Section 5.6 구조

```
### 5.6 속도 제한
├── 응답 헤더
│   ├── ✅ 필수: X-RateLimit-* 헤더 (Limit, Remaining, Reset)
│   ├── ✅ 필수: IETF RateLimit, RateLimit-Policy 헤더 (모든 응답에 포함)
│   ├── 참고: reset 값 단위 차이 설명 (X-RateLimit-Reset=Unix ts, RateLimit.reset=delta-seconds)
│   └── 정상 응답 예시 (200 OK)
├── 429 Too Many Requests 응답
│   ├── ✅ 필수: 429 상태 코드
│   ├── ✅ 필수: Retry-After 헤더 (delta-seconds)
│   ├── ✅ 필수: Problem Details 본문 (확장 필드 없음)
│   └── 429 응답 예시 (헤더 + JSON 본문)
└── 클라이언트 재시도 전략
    ├── ✅ 필수: Retry-After 헤더 값 준수
    ├── ⚠️ 권장: exponential backoff + jitter (Retry-After 없는 경우)
    ├── ❌ 금지: 즉시 재시도 / 고정 간격 반복
    └── ❌ 금지: Retry-After 무시
```

### 정상 응답 예시 (200 OK)

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

> `RateLimit-Policy: 100;w=3600` — 시간 창(w, window) 3600초 내 최대 100회 허용

### 429 응답 예시

**헤더:**
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

**본문:**
```json
{
  "type": "https://api.example.com/errors/too-many-requests",
  "title": "속도 제한 초과",
  "status": 429,
  "detail": "허용된 요청 한도를 초과했습니다. 50초 후에 다시 시도해 주세요."
}
```

> `instance`, `traceId` 필드는 3.4절 규칙에 따라 ⚠️ 권장으로 포함 가능하다.

### 재시도 전략 공식

`attempt`는 1부터 시작하는 재시도 횟수(1 = 첫 번째 재시도).

```
대기 시간 = min(maxDelay, baseDelay × 2^(attempt - 1)) + random(0, jitterRange)
```

예시: attempt=1 → `min(60, 1 × 2^0) + random(0,1)` = 1~2초

| 파라미터 | 권장 값 | 설명 |
|----------|---------|------|
| `baseDelay` | 1초 | 첫 번째 재시도 대기 시간 |
| `maxDelay` | 60초 | 최대 대기 시간 상한 |
| `jitterRange` | 0~1초 | 무작위 지연 (thundering herd 방지) |
| 최대 재시도 횟수 | 3~5회 | 무한 재시도 방지 |

---

## 검증 기준

1. `X-RateLimit-*` 헤더 3개 모두 **정상 응답 + 429 응답** 예시에 존재
2. `RateLimit`, `RateLimit-Policy` 헤더 **정상 응답 + 429 응답** 예시에 존재
3. `Retry-After` 헤더 429 예시에 존재 (delta-seconds)
4. 429 본문이 표준 Problem Details 구조만 사용 (확장 필드 없음)
5. 클라이언트 `Retry-After` 준수가 ✅ 필수로 명시됨
6. 지수 백오프 공식에 `attempt` 초기값(1-based) 정의됨
7. 클라이언트 재시도 전략 포함 (공식, 파라미터 표, 금지 항목)
8. 스킬 파일 코드 작성 모드 + 리뷰 체크리스트 모두 반영

---

## 참고 자료

- [RFC 6585 - Additional HTTP Status Codes (429)](https://datatracker.ietf.org/doc/html/rfc6585#section-4)
- [IETF draft-ietf-httpapi-ratelimit-headers-10](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/)
- [RFC 9110 - HTTP Semantics (Retry-After)](https://www.rfc-editor.org/rfc/rfc9110#section-10.2.3)
- [RFC 7807 - Problem Details for HTTP APIs](https://datatracker.ietf.org/doc/html/rfc7807)
