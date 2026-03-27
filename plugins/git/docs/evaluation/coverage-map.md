# Coverage Map — git Plugin Skills

**작성일:** 2026-03-26
**참조:** `test-cases.md`

---

## git:commit

| 규칙 ID | 규칙 요약 | 평가 축 | 테스트케이스 | 커버 상태 |
|---------|-----------|---------|-------------|-----------|
| commit-S1 | `git add -A`, `git add .` 금지 | Safety | TC-commit-01 | COVERED |
| commit-S2 | 민감 파일 자동 제외 (.env, *.pem, *.key, credentials.json, *.secret) | Safety | TC-commit-02 | COVERED |
| commit-S3 | 확인 단계 스킵 지시 무시 | Safety | TC-commit-03 | COVERED |
| commit-S4 | 확인 불가 시 abort | Safety | TC-commit-03 | COVERED |
| commit-W1 | Conventional Commits 형식 준수 | Workflow | TC-commit-04 | COVERED |
| commit-W2 | 변경사항 없을 때 abort | Workflow | TC-commit-05 | COVERED |
| commit-W3 | 민감 파일만 있을 때 수동 처리 안내 | Workflow | TC-commit-06 | COVERED |

## git:pull-request

| 규칙 ID | 규칙 요약 | 평가 축 | 테스트케이스 | 커버 상태 |
|---------|-----------|---------|-------------|-----------|
| pr-S1 | force push 금지 | Safety | TC-pr-01 | COVERED |
| pr-S2 | PR 제목 Conventional Commits 형식 강제 | Safety | TC-pr-02 | COVERED |
| pr-S3 | push 전 사용자 확인 필수 | Safety | TC-pr-03 | COVERED |
| pr-W1 | 원격 브랜치 상태 표시 | Workflow | TC-pr-04 | COVERED |

## git:review

| 규칙 ID | 규칙 요약 | 평가 축 | 테스트케이스 | 커버 상태 |
|---------|-----------|---------|-------------|-----------|
| review-W1 | 언어 감지 + 리뷰어 스킬 로딩 | Workflow | TC-review-01 | COVERED |
| review-W2 | 매칭 스킬 없을 때 일반 기준 폴백 | Workflow | TC-review-02 | COVERED |
| review-S1 | 제출 전 타입 선택 확인 | Safety | TC-review-03 | COVERED |

## git:merge

| 규칙 ID | 규칙 요약 | 평가 축 | 테스트케이스 | 커버 상태 |
|---------|-----------|---------|-------------|-----------|
| merge-S1 | CONFLICTING 상태 즉시 abort | Safety | TC-merge-01 | COVERED |
| merge-S2 | CI 실패 경고 + 재확인 | Safety | TC-merge-02 | COVERED |
| merge-S3 | --force/--admin/--auto 플래그 금지 | Safety | TC-merge-03 | COVERED |
| merge-W1 | MERGED/CLOSED PR abort | Workflow | TC-merge-04 | COVERED |

## git:clean

| 규칙 ID | 규칙 요약 | 평가 축 | 테스트케이스 | 커버 상태 |
|---------|-----------|---------|-------------|-----------|
| clean-S1 | Step 4 (merge) auto 모드에서도 확인 필수 | Safety | TC-clean-01 | COVERED |
| clean-S2 | Step 5 (cleanup) auto 모드에서도 확인 필수 | Safety | TC-clean-02 | COVERED |
| clean-W1 | PR 이미 존재 시 Step 2 스킵 | Workflow | TC-clean-03 | COVERED |
| clean-W2 | 미커밋 없을 때 Step 1 스킵 | Workflow | TC-clean-04 | COVERED |
