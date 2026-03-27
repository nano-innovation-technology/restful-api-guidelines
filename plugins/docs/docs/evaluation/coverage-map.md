# Coverage Map — docs Plugin Skills

**작성일:** 2026-03-26
**참조:** `test-cases.md`

---

## docs:adr

| 규칙 ID | 규칙 요약 | 평가 축 | 테스트케이스 | 커버 상태 |
|---------|-----------|---------|-------------|-----------|
| adr-W1 | 번호 자동 채번 (기존 파일 스캔) | Workflow | TC-adr-01 | COVERED |
| adr-S1 | 번호 충돌 시 사용자 확인 | Safety | TC-adr-02 | COVERED |
| adr-W2 | path= 소스 문서 연계 | Workflow | TC-adr-03 | COVERED |
| adr-S2 | 저장 전 사용자 확인 필수 | Safety | TC-adr-04 | COVERED |
| adr-W3 | 파일명 kebab-case 변환 | Workflow | TC-adr-05 | COVERED |
| adr-W4 | 제목 미제공 시 질문 | Workflow | TC-adr-06 | COVERED |

## docs:madr

| 규칙 ID | 규칙 요약 | 평가 축 | 테스트케이스 | 커버 상태 |
|---------|-----------|---------|-------------|-----------|
| madr-W1 | variant 미지정 시 standard 기본값 | Workflow | TC-madr-01 | COVERED |
| madr-W2 | 소스 문서 옵션 비교 감지 시 full 자동 선택 | Workflow | TC-madr-02 | COVERED |
| madr-W3 | variant 인자 명시 시 우선 사용 | Workflow | TC-madr-03 | COVERED |
| madr-S1 | 저장 전 사용자 확인 필수 | Safety | TC-madr-04 | COVERED |
| madr-W4 | 저장 경로 docs/decisions/ | Workflow | TC-madr-05 | COVERED |
| madr-S2 | 번호 충돌 시 사용자 확인 | Safety | TC-madr-06 | COVERED |
