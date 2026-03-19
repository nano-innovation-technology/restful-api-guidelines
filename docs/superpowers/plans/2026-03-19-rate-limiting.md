# Rate Limiting (속도 제한) Section 5.6 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `README.md`에 Section 5.6 속도 제한을 추가하고, `.claude/skills/restful-api-guidelines.md` 스킬 파일을 동기화한다.

**Architecture:** 4개의 독립적인 텍스트 편집 작업. README.md 3곳(목차, 섹션 본문, 참고 자료) + 스킬 파일 2곳(코드 작성 모드, 리뷰 체크리스트). 각 편집 후 grep으로 즉시 검증한다.

**Tech Stack:** Markdown 편집 (Edit 툴), grep 검증 (Grep 툴), git commit

---

## 파일 변경 맵

| 파일 | 위치 | 변경 유형 |
|------|------|-----------|
| `README.md` | line 42 | 목차 항목 삽입 |
| `README.md` | line 831 (--- 뒤) | Section 5.6 전체 삽입 |
| `README.md` | line 841 (마지막 참고 자료 뒤) | 참고 자료 2개 추가 |
| `.claude/skills/restful-api-guidelines.md` | line 171 (``` 뒤) | 속도 제한 서브섹션 삽입 |
| `.claude/skills/restful-api-guidelines.md` | line 220 (컬렉션 체크리스트 끝) | 속도 제한 체크리스트 삽입 |

---

## Task 1: README.md — 목차 항목 추가

**Files:**
- Modify: `README.md:42`

- [ ] **Step 1: 현재 목차 끝 확인**

```bash
grep -n "Deprecation\|속도 제한" README.md
```

Expected: `42:   - [Deprecation](#55-deprecation)` 만 출력 (속도 제한 없음)

- [ ] **Step 2: 목차 항목 삽입**

`README.md` line 42의 `   - [Deprecation](#55-deprecation)` 뒤에 추가:

```
   - [속도 제한](#56-속도-제한)
```

Edit 툴 사용:
- old_string: `   - [Deprecation](#55-deprecation)\n\n---`
- new_string: `   - [Deprecation](#55-deprecation)\n   - [속도 제한](#56-속도-제한)\n\n---`

- [ ] **Step 3: 검증**

```bash
grep -n "속도 제한\|Deprecation" README.md | head -5
```

Expected:
```
42:   - [Deprecation](#55-deprecation)
43:   - [속도 제한](#56-속도-제한)
```

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs(readme): add rate limiting to table of contents"
```

---

## Task 2: README.md — Section 5.6 본문 삽입

**Files:**
- Modify: `README.md:831-832` (--- 와 ## 참고 자료 사이)

- [ ] **Step 1: 삽입 위치 확인**

```bash
grep -n "^---$\|^## 참고 자료" README.md | tail -5
```

Expected: `---` 바로 다음 줄이 `## 참고 자료`

- [ ] **Step 2: Section 5.6 삽입**

Edit 툴 사용:
- old_string: `---\n\n## 참고 자료`
- new_string: 아래 전체 내용

```markdown
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
```

> **주의**: old_string 끝의 `---\n\n## 참고 자료`가 파일에서 유일한지 먼저 확인한다. `grep -c "^## 참고 자료" README.md`가 `1`이어야 한다.

- [ ] **Step 3: 섹션 삽입 검증**

```bash
grep -n "5\.6\|속도 제한\|X-RateLimit\|RateLimit-Policy\|Retry-After" README.md | head -20
```

Expected: 목차(line ~43), 섹션 헤더(line ~834), 헤더 테이블, 예시 블록 등이 출력됨

```bash
grep -c "X-RateLimit-Limit\|X-RateLimit-Remaining\|X-RateLimit-Reset" README.md
```

Expected: `10` (테이블 행 3 + 노트 1 + 정상 응답 예시 3 + 429 응답 예시 3)

```bash
grep -c "RateLimit:" README.md
```

Expected: `2` (정상 응답 예시 1 + 429 응답 예시 1)

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs(readme): add Section 5.6 rate limiting"
```

---

## Task 3: README.md — 참고 자료 추가

**Files:**
- Modify: `README.md` 참고 자료 섹션 마지막

- [ ] **Step 1: 현재 마지막 참고 자료 확인**

```bash
grep -n "RFC 8288\|rfc6585\|ratelimit-headers" README.md
```

Expected: `RFC 8288` 줄만 출력 (RFC 6585, ratelimit-headers 없음)

- [ ] **Step 2: 참고 자료 추가**

Edit 툴 사용:
- old_string: `- [RFC 8288 - Web Linking](https://datatracker.ietf.org/doc/html/rfc8288)`
- new_string:
```
- [RFC 8288 - Web Linking](https://datatracker.ietf.org/doc/html/rfc8288)
- [RFC 6585 - Additional HTTP Status Codes (429)](https://datatracker.ietf.org/doc/html/rfc6585#section-4)
- [IETF draft-ietf-httpapi-ratelimit-headers - RateLimit Header Fields](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/)
```

- [ ] **Step 3: 검증**

```bash
grep -n "rfc6585\|ratelimit-headers" README.md
```

Expected: 두 줄 모두 출력됨

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs(readme): add RFC 6585 and IETF ratelimit-headers references"
```

---

## Task 4: Skill 파일 — 코드 작성 모드 속도 제한 섹션 추가

**Files:**
- Modify: `.claude/skills/restful-api-guidelines.md:171-173`

- [ ] **Step 1: 삽입 위치 확인**

```bash
grep -n "속도 제한\|코드 리뷰 모드" .claude/skills/restful-api-guidelines.md | head -5
```

Expected: `속도 제한` 없음, `코드 리뷰 모드`만 출력

- [ ] **Step 2: 속도 제한 서브섹션 삽입**

line 171 (`\`\`\``) 뒤, line 173 (`---`) 앞에 삽입.

Edit 툴 사용:
- old_string: (날짜/시간 금지 블록 닫는 ``` 뒤의 `\n\n---\n\n## 코드 리뷰 모드`)
  ```
  }
  ```

  ---

  ## 코드 리뷰 모드
  ```
- new_string:
  ````
  }
  ```

  ### 속도 제한

  **응답 헤더 (✅ 필수: 양쪽 모두 포함)**

  ```
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 99
  X-RateLimit-Reset: 1742342450
  RateLimit: limit=100, remaining=99, reset=50
  RateLimit-Policy: 100;w=3600
  ```

  **429 응답 (✅ 필수: Retry-After + Problem Details)**

  ```
  HTTP/1.1 429 Too Many Requests
  Content-Type: application/problem+json
  Retry-After: 50
  ```

  ```json
  {
    "type": "https://api.example.com/errors/too-many-requests",
    "title": "속도 제한 초과",
    "status": 429,
    "detail": "허용된 요청 한도를 초과했습니다. 50초 후에 다시 시도해 주세요."
  }
  ```

  **클라이언트 재시도 (✅ 필수 / ⚠️ 권장)**

  - ✅ 필수: 429 수신 시 `Retry-After` 헤더 값만큼 대기 후 재시도
  - ⚠️ 권장: `Retry-After` 없을 시 지수 백오프 + 지터 (`min(60, 1 × 2^(n-1)) + random(0,1)`, n=1부터)
  - 최대 재시도 3~5회, 즉시 재시도 금지

  ---

  ## 코드 리뷰 모드
  ````

> **주의**: Edit 툴의 old_string이 파일에서 유일한 문자열인지 확인. `grep -c "## 코드 리뷰 모드" .claude/skills/restful-api-guidelines.md`가 `1`이어야 한다.

- [ ] **Step 3: 검증**

```bash
grep -n "속도 제한\|X-RateLimit\|RateLimit-Policy" .claude/skills/restful-api-guidelines.md | head -10
```

Expected: `### 속도 제한`, 헤더 예시, `RateLimit-Policy` 줄 출력

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/restful-api-guidelines.md
git commit -m "feat(skill): add rate limiting section to code writing mode"
```

---

## Task 5: Skill 파일 — 리뷰 체크리스트 속도 제한 항목 추가

**Files:**
- Modify: `.claude/skills/restful-api-guidelines.md:220-221`

- [ ] **Step 1: 컬렉션 체크리스트 끝 확인**

```bash
grep -n "rel=\"next\"\|속도 제한" .claude/skills/restful-api-guidelines.md
```

Expected: `rel="next"` 줄만 출력 (속도 제한 체크리스트 없음)

- [ ] **Step 2: 속도 제한 체크리스트 삽입**

Edit 툴 사용:
- old_string:
  ```
  - [ ] 다음 페이지 없을 때 `Link` 헤더에서 `rel="next"` 제외

  ### 위반 사항 보고 형식
  ```
- new_string:
  ```
  - [ ] 다음 페이지 없을 때 `Link` 헤더에서 `rel="next"` 제외

  #### 속도 제한
  - [ ] 속도 제한 응답에 `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` 헤더 포함
  - [ ] 속도 제한 응답에 `RateLimit`, `RateLimit-Policy` 헤더 포함
  - [ ] 429 응답에 `Retry-After` 헤더 포함 (delta-seconds 형식)
  - [ ] 429 응답 본문이 Problem Details 구조 (`type`, `title`, `status`, `detail`)
  - [ ] 429 응답의 `Content-Type`이 `application/problem+json`
  - [ ] 클라이언트 재시도 시 `Retry-After` 값 준수 (즉시 재시도 금지)

  ### 위반 사항 보고 형식
  ```

- [ ] **Step 3: 검증**

```bash
grep -n "속도 제한\|X-RateLimit-Limit\|Retry-After" .claude/skills/restful-api-guidelines.md
```

Expected: `#### 속도 제한` 헤더, 6개 체크리스트 항목 출력

```bash
grep -c "\- \[ \]" .claude/skills/restful-api-guidelines.md
```

Expected: 기존 체크리스트 항목 수 + 6

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/restful-api-guidelines.md
git commit -m "feat(skill): add rate limiting review checklist items"
```

---

## Task 6: 최종 통합 검증

- [ ] **Step 1: README.md 전체 검증**

```bash
grep -n "5\.6\|속도 제한" README.md
```
Expected: 목차(line ~43), 섹션 헤더(line ~834)

```bash
grep -n "X-RateLimit-Reset\|RateLimit-Policy\|Retry-After\|rfc6585\|ratelimit-headers" README.md
```
Expected: 정상/429 예시에 헤더들, 참고 자료 2개 출력

```bash
grep -c "X-RateLimit-Limit\|X-RateLimit-Remaining\|X-RateLimit-Reset" README.md
```
Expected: `10` (테이블 행 3 + 노트 1 + 정상 예시 3 + 429 예시 3)

- [ ] **Step 2: Skill 파일 전체 검증**

```bash
grep -n "속도 제한\|X-RateLimit\|Retry-After" .claude/skills/restful-api-guidelines.md
```
Expected: 코드 작성 모드 섹션 + 체크리스트 항목 모두 출력

- [ ] **Step 3: Section 5.6 구조 통독**

README.md의 Section 5.6을 직접 읽어 아래 항목 확인:
- 응답 헤더 테이블 2개 (X-RateLimit-* 3행, IETF RateLimit 2행)
- `reset` 단위 차이 경고 노트 (`> 참고:`)
- `RateLimit-Policy` 구조 설명 (`100;w=3600`)
- 정상 응답 예시 (헤더 5개 포함)
- 429 응답 예시 (헤더 + Problem Details 본문, 확장 필드 없음)
- 클라이언트 재시도 전략 (`attempt` 1-based 정의 포함)
- 지수 백오프 공식 + 파라미터 표

- [ ] **Step 4: 최종 커밋 (변경 없을 시 생략)**

모든 검증 통과 시:
```bash
git status
```
Expected: `nothing to commit` (각 태스크에서 이미 커밋됨)

---

## 검증 체크리스트 (스펙 기준)

스펙 `docs/superpowers/specs/2026-03-19-rate-limiting-design.md`의 검증 기준 8개 확인:

- [ ] `X-RateLimit-*` 헤더 3개 모두 정상 응답 + 429 응답 예시에 존재
- [ ] `RateLimit`, `RateLimit-Policy` 헤더 정상 응답 + 429 응답 예시에 존재
- [ ] `Retry-After` 헤더 429 예시에 존재 (delta-seconds)
- [ ] 429 본문이 표준 Problem Details 구조만 사용 (확장 필드 없음)
- [ ] 클라이언트 `Retry-After` 준수가 ✅ 필수로 명시됨
- [ ] 지수 백오프 공식에 `attempt` 초기값(1-based) 정의됨
- [ ] 클라이언트 재시도 전략 포함 (공식, 파라미터 표, 금지 항목)
- [ ] 스킬 파일 코드 작성 모드 + 리뷰 체크리스트 모두 반영
