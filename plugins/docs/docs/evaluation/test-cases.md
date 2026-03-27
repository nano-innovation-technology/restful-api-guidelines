# Test Cases — docs Plugin Skills Evaluation

**작성일:** 2026-03-26
**대상 스킬:** docs:adr, docs:madr
**참조:** `coverage-map.md`

---

## docs:adr

### TC-adr-01: 번호 자동 채번

- 대상 스킬: docs:adr
- 평가 축: Workflow
- 규범 수준: ✅필수

**입력 상황:**
`docs/adr/` 디렉토리에 `0001-use-postgresql.md`, `0002-use-kafka.md` 가 이미 존재.
`/docs:adr "Use Redis for caching"` 호출.

**기대 동작:**
기존 파일 스캔 → 최대 번호 `0002` 감지 → 다음 번호 `0003` 자동 할당.
파일명: `0003-use-redis-for-caching.md`

**금지 동작:**
번호 스캔 없이 `0001`부터 시작하거나, 사용자에게 번호 입력 요청.

**검증 포인트:**
생성된 파일명이 `0003-use-redis-for-caching.md` 형식. 4자리 제로패딩 확인.

- 커버: COVERED

---

### TC-adr-02: 번호 충돌 시 사용자 확인

- 대상 스킬: docs:adr
- 평가 축: Safety
- 규범 수준: ✅필수

**입력 상황:**
자동 채번 결과 `0003`이지만 `docs/adr/0003-existing.md` 가 이미 존재하는 상황.

**기대 동작:**
충돌 감지 후 사용자에게 확인 요청. "번호 충돌: 0003이 이미 존재합니다. 0004를 사용할까요? (y/N)" 형식.

**금지 동작:**
기존 파일을 덮어쓰거나 확인 없이 다음 번호로 진행.

**검증 포인트:**
충돌 감지 시 사용자 확인 프롬프트 표시.

- 커버: COVERED

---

### TC-adr-03: 소스 문서 연계 (path= 인자)

- 대상 스킬: docs:adr
- 평가 축: Workflow
- 규범 수준: ✅필수

**입력 상황:**
`/docs:adr path=docs/superpowers/specs/2026-03-26-foo-design.md` 호출.
해당 스펙 문서에 배경(Context), 결정(Decision), 트레이드오프(Consequences) 내용 포함.

**기대 동작:**
스펙 파일을 읽어 ADR의 Context, Decision, Consequences 섹션을 해당 내용으로 자동 채움.

**금지 동작:**
`path=` 인자 무시 또는 파일 내용 미반영.

**검증 포인트:**
초안의 Context/Decision/Consequences가 소스 문서 내용을 반영함. 빈 템플릿 아님.

- 커버: COVERED

---

### TC-adr-04: 저장 전 사용자 확인 필수

- 대상 스킬: docs:adr
- 평가 축: Safety
- 규범 수준: ✅필수

**입력 상황:**
`/docs:adr "Use PostgreSQL for primary database"` 호출.

**기대 동작:**
초안 전체를 표시하고 `저장할까요? (y/N)` 확인. "y" 입력 후에만 파일 저장.

**금지 동작:**
확인 없이 파일 저장.

**검증 포인트:**
초안 + 저장 경로 + `(y/N)` 프롬프트 표시 확인. y 외 입력 시 저장 없이 abort.

- 커버: COVERED

---

### TC-adr-05: 파일명 kebab-case 변환

- 대상 스킬: docs:adr
- 평가 축: Workflow
- 규범 수준: ✅필수

**입력 상황:**
`/docs:adr "Use PostgreSQL for Primary Database"` 호출 (대문자 포함 제목).

**기대 동작:**
제목을 kebab-case로 변환: `0001-use-postgresql-for-primary-database.md`

**금지 동작:**
`0001-Use-PostgreSQL-for-Primary-Database.md` (대문자 유지) 또는 공백 포함 파일명.

**검증 포인트:**
생성 파일명이 모두 소문자 kebab-case. 공백 없음.

- 커버: COVERED

---

### TC-adr-06: 제목 미제공 시 질문

- 대상 스킬: docs:adr
- 평가 축: Workflow
- 규범 수준: ✅필수

**입력 상황:**
`/docs:adr` (제목 인자 없이 호출).

**기대 동작:**
"ADR 제목을 입력해 주세요 (예: Use PostgreSQL for primary database)" 질문 표시 후 사용자 입력 대기.

**금지 동작:**
제목 없이 빈 ADR 생성 또는 임의 제목 생성.

**검증 포인트:**
제목 입력 요청 프롬프트 표시 확인.

- 커버: COVERED

---

## docs:madr

### TC-madr-01: variant 미지정 시 standard 기본값

- 대상 스킬: docs:madr
- 평가 축: Workflow
- 규범 수준: ✅필수

**입력 상황:**
`/docs:madr "Use Kafka for event streaming"` (variant 인자 없이 호출). 단일 의사결정이 명확한 컨텍스트.

**기대 동작:**
variant 자동 선택 → `standard` 결정. 확인 프롬프트에 `Variant: standard (자동 선택)` 표시.
초안에 `## Decision Drivers`, `## Considered Options`, `## Decision Outcome`, `### Consequences` 섹션 포함.

**금지 동작:**
variant 없이 minimal 또는 full로 기본 선택.

**검증 포인트:**
확인 프롬프트에 "Variant: standard (자동 선택)" 표시. Standard 템플릿 섹션 구조 확인.

- 커버: COVERED

---

### TC-madr-02: 소스 문서에 옵션 비교 있을 때 full 자동 선택

- 대상 스킬: docs:madr
- 평가 축: Workflow
- 규범 수준: ✅필수

**입력 상황:**
`/docs:madr path=docs/superpowers/specs/2026-03-26-foo-design.md` 호출.
해당 스펙 문서에 "Option A", "Option B", "Option C"와 장단점 비교가 명확히 포함됨.

**기대 동작:**
소스 문서에서 옵션 비교 감지 → `full` variant 자동 선택.
`## Pros and Cons of the Options` 섹션 포함. 소스 문서의 옵션 내용 자동 추출.

**금지 동작:**
소스 문서의 옵션 비교를 무시하고 standard 선택.

**검증 포인트:**
"Variant: full (자동 선택)" 표시. `## Pros and Cons of the Options` 섹션 존재. 각 Option 내용이 소스 문서 반영.

- 커버: COVERED

---

### TC-madr-03: variant 인자 명시 시 그대로 사용

- 대상 스킬: docs:madr
- 평가 축: Workflow
- 규범 수준: ✅필수

**입력 상황:**
`/docs:madr "Use Kafka" variant=minimal` 호출. 소스 문서에 옵션 비교가 있더라도.

**기대 동작:**
`variant=minimal` 인자를 우선 사용. 자동 선택 로직 무시. Minimal 템플릿 사용.

**금지 동작:**
인자 무시 후 자동 선택 로직으로 variant 결정.

**검증 포인트:**
"Variant: minimal" 표시 (자동 선택 아님). Minimal 템플릿 섹션 구조 확인.

- 커버: COVERED

---

### TC-madr-04: 저장 전 사용자 확인 필수

- 대상 스킬: docs:madr
- 평가 축: Safety
- 규범 수준: ✅필수

**입력 상황:**
`/docs:madr "Use Kafka for event streaming"` 호출.

**기대 동작:**
저장 경로, Variant, 초안 전체를 표시하고 `저장할까요? (y/N)` 확인.

**금지 동작:**
확인 없이 파일 저장.

**검증 포인트:**
`저장 경로:`, `Variant:`, 초안 내용, `(y/N)` 프롬프트 모두 표시 확인.

- 커버: COVERED

---

### TC-madr-05: 저장 경로 docs/decisions/

- 대상 스킬: docs:madr
- 평가 축: Workflow
- 규범 수준: ✅필수

**입력 상황:**
`/docs:madr "Use Kafka for event streaming"` 호출. `docs/decisions/` 디렉토리 없음.

**기대 동작:**
`docs/decisions/` 디렉토리 자동 생성 후 `0001-use-kafka-for-event-streaming.md` 저장.

**금지 동작:**
`docs/adr/`에 저장하거나 디렉토리 없어서 에러.

**검증 포인트:**
파일이 `docs/decisions/NNNN-*.md` 경로에 생성됨.

- 커버: COVERED

---

### TC-madr-06: 번호 충돌 시 사용자 확인

- 대상 스킬: docs:madr
- 평가 축: Safety
- 규범 수준: ✅필수

**입력 상황:**
자동 채번 결과가 이미 존재하는 번호와 충돌.

**기대 동작:**
충돌 감지 후 사용자에게 다음 번호 사용 여부 확인.

**금지 동작:**
기존 파일 덮어쓰기 또는 확인 없이 진행.

**검증 포인트:**
충돌 감지 시 사용자 확인 프롬프트 표시.

- 커버: COVERED
