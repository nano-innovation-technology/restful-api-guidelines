# Adopt RFC 6648 for Custom HTTP Header Naming

## Status

accepted

## Context and Problem Statement

`api:restful-guidelines` 스킬의 Headers 섹션은 "Custom headers omit deprecated `X-` prefix (RFC 6648)"라고 명시하면서도,
API Versioning 예시에서 `X-API-Version`, Headers 섹션에서 `X-Total-Count`를 사용하는 자기 모순 상태였다.
신규 프로젝트에서 어떤 커스텀 헤더 네이밍 규칙을 따라야 하는가?

## Decision Drivers

* RFC 6648 (BCP 178) — 신규 파라미터에 `X-` 접두사 사용을 금지
* 신규 프로젝트이므로 기존 클라이언트와의 하위 호환성 부담이 없음
* 스킬 내부 일관성 — 규칙과 예시가 동일한 기준을 따라야 함
* IETF 표준 정렬 — `Idempotency-Key`는 IETF 드래프트가 `X-` 없이 정의

## Considered Options

* Option A: `X-` 접두사 유지 (현상 유지)
* Option B: 모든 신규 커스텀 헤더에서 `X-` 접두사 제거 (RFC 6648 준수)

## Decision Outcome

Chosen option: "Option B", because RFC 6648/BCP 178이 명시적으로 신규 헤더의 `X-` 접두사를 금지하며,
신규 프로젝트이므로 마이그레이션 비용이 없고, 스킬 자체 규칙과 예시의 일관성을 확보할 수 있기 때문이다.

변경 대상 헤더:

| 변경 전 | 변경 후 | 비고 |
|---------|---------|------|
| `X-API-Version` | `Api-Version` | ISO 8601 날짜 기반 버전 |
| `X-Total-Count` | `Total-Count` | 컬렉션 총 개수 |
| `X-Idempotency-Key` | `Idempotency-Key` | IETF 드래프트와 일치 |
| `X-Organization-Id` | `Organization-Id` | 멀티테넌트 식별 |
| `X-Forwarded-For` | 유지 | RFC 7239 표준, 프록시 호환성 예외 |

### Consequences

* Good: RFC 6648/BCP 178 준수로 장기적으로 표준 헤더로 승격 시 이름 충돌 없음
* Good: 스킬 규칙과 예시가 일관됨 — 자기 모순 해소
* Good: `Idempotency-Key`가 IETF 드래프트와 즉시 호환
* Bad: `X-` 접두사에 익숙한 개발자에게 초기 혼란 가능 — 명시적 RFC 인용으로 완화

## Pros and Cons of the Options

### Option A: `X-` 접두사 유지

* Good: 관행적으로 익숙한 패턴
* Bad: RFC 6648을 직접 위반
* Bad: 스킬 자체 규칙("Custom headers omit X- prefix")과 모순

### Option B: `X-` 접두사 제거 (RFC 6648 준수)

* Good: RFC 6648/BCP 178 완전 준수
* Good: 신규 프로젝트이므로 마이그레이션 비용 제로
* Good: IETF 드래프트(`Idempotency-Key`) 및 표준 정렬
* Bad: 일부 레거시 문서나 서드파티 예시와 불일치 가능 — 예외 목록으로 관리
