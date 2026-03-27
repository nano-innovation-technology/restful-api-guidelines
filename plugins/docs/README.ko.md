# 문서 결정 기록 스킬

> [English](README.md)

아키텍처 결정 사항을 공식 문서로 기록하고 추적하기 위한 Claude Code 스킬 모음입니다.

## 스킬 목록

| 스킬 | 슬래시 커맨드 | 설명 |
|------|--------------|------|
| adr | `/docs:adr` | Nygard ADR 포맷으로 아키텍처 결정 기록 |
| madr | `/docs:madr` | MADR 3.x 포맷으로 아키텍처 결정 기록 |

## 저장 경로

| 스킬 | 경로 | 포맷 |
|------|------|------|
| `docs:adr` | `docs/adr/NNNN-<title>.md` | Nygard ADR |
| `docs:madr` | `docs/decisions/NNNN-<title>.md` | MADR 3.x |

## 다른 스킬과의 연계

두 스킬 모두 독립 실행 또는 다른 스킬과 연계하여 사용할 수 있습니다:

```
superpowers:brainstorming   →  스펙 문서 (docs/superpowers/specs/)
superpowers:writing-plans   →  구현 계획
/docs:adr path=<spec>       →  공식 ADR 문서 생성
/docs:madr path=<spec>      →  공식 MADR 문서 생성
```

`path=` 인자로 스펙 문서 경로를 넘기면 해당 파일에서 배경, 결정 사항, 옵션 비교를 자동으로 추출합니다.

## MADR Variant

`docs:madr`는 `variant=` 인자로 템플릿을 선택할 수 있습니다:

| Variant | 포함 섹션 | 사용 상황 |
|---------|-----------|-----------|
| `minimal` | 제목, 상태, 컨텍스트, 결정 결과 | 빠른 기록 |
| `standard` | + 결정 드라이버, 고려 옵션, 결과 | **기본값** |
| `full` | + 옵션별 장단점 | 옵션 간 비교가 중요할 때 |

`variant=` 를 지정하지 않으면 Claude가 컨텍스트를 분석하여 자동으로 선택합니다.

## 설치

```bash
claude plugin marketplace add https://github.com/ppzxc/engineering-guidelines.git
```
