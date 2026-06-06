---
domain: fork-maintenance
---
# Directive #012
- Summary: Enumerate every custom change our fork made to genshin-ts (memory-recovery + basis for upstream merge)
- Created: 2026-06-06

## Why
원본 제작자(upstream) genshin-ts가 그동안 많이 업데이트됨. 곧 최신 업스트림을 병합하면서 "우리 변형은 최소화하고 원본 형태를 최대한 유지하며 통합"할 예정. 그 전에, 우리 포크가 upstream 대비 무엇을 바꿨는지 **완전하고 구조화된 목록**이 필요함 (유저의 기억 회복 + 병합 작업의 기준 자료).

## Goal
genshin-ts 포크가 원본 대비 가한 **모든 커스텀 변경사항**을 카테고리별로 빠짐없이 나열한다.

## Scope / Inputs (교차 검증)
- `notes/changelog.md` — 포크 변경 로그 (#1~#9). 1차 출처.
- `notes/protocol/`, `notes/architecture/` — 프로토콜/아키텍처 변경 (특히 protobuf 스키마 차이).
- handoff `Decisions` 섹션 (`.claude/handoff-gsts.md`) — 결정 배경.
- `git log` 전체 히스토리 — 실제 커밋과 changelog 대조 (누락/불일치 확인).
- 실제 소스 코드 — changelog에 안 적힌 변경이 있는지 확인 (예: protobuf 정의, signal 필드, injector 하드코딩 값).

> upstream remote는 현재 미설정. 가능하면 원본 repo를 식별(git 히스토리의 포크 분기점 또는 changelog 단서)해 비교하되, 불가하면 우리 기록 기반으로 작성하고 그 한계를 명시한다.

## Output
파일: `notes/fork-changes-inventory.md` (신규, 한국어)
각 변경 항목마다 다음을 포함:
- **카테고리** (예: 버그 수정 / 신규 기능 / protobuf 스키마 / 문서 / 빌드·패키징 / CLI / 설정)
- **변경 내용** (무엇을, 어디서 — 파일/모듈 경로)
- **이유** (왜 바꿨는지)
- **upstream 충돌 위험도** (HIGH/MED/LOW — 코드 변경=위험, 문서/notes=무위험. 병합 시 우선 확인 지점 표시)
- **병합 처리 방향 메모** (해당되면: upstream에 동일 기능 도입 시 우리 변형을 버리고 원본 채택 가능한지 등)

마지막에 **요약 표**: 카테고리별 항목 수 + 충돌 위험 분포.

## Constraints
- 빠짐없이(comprehensive) — changelog와 git/code가 어긋나면 둘 다 기록하고 불일치를 표시.
- 추측과 확인 사실을 구분 표기.
- 인라인으로는 핵심 요약(카테고리별 개요 + 충돌 위험 HIGH 항목)만 반환.
