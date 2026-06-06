---
domain: fork-maintenance
---
# Directive #013
- Summary: Verify inspect/scaffold/reader CLI on a real .gil, extract node graphs, analyze the largest one
- Created: 2026-06-06

## Why
유저가 elementalist에서 inspect/scaffold 기능이 제대로 작동하지 않았던 기억이 있음. 실제 게임 세이브 .gil로 추출을 돌려 기능 동작 여부를 확인하고, 어떤 노드 그래프가 들어있는지 파악한다.

## Target file
`C:\Users\Rterg\AppData\LocalLow\miHoYo\Genshin Impact\BeyondLocal\804101570\Beyond_Local_Save_Level\1073741976.gil`

## Goal
1. genshin-ts의 inspect / reader / scaffold CLI(우리 포크 C카테고리 도구)를 위 .gil 파일에 실제로 실행한다.
2. 추출 결과로 **어떤 노드 그래프 파일들이 생성/존재하는지** 나열한다 (파일명 + 용량).
3. 그 중 **용량이 가장 큰 노드 그래프 하나**를 골라 구조를 분석한다 (노드 종류/개수, 시그널, 주목할 패턴).
4. inspect/scaffold가 **정상 작동하는지 / 어디서 깨지는지** 명확히 판정한다. 깨진다면 에러 메시지·원인 가설·관련 소스 위치를 적는다.

## Output
- 인라인 보고: (a) 기능 동작 판정(정상/부분/실패 + 근거), (b) 추출된 노드 그래프 파일 목록(이름/용량), (c) 최대 노드 그래프 분석 요약.
- 추출/스캐폴드 산출물은 임시 작업 폴더에 두고 경로를 보고에 명시 (저장소 루트를 더럽히지 말 것).

## Constraints
- 실제 실행 기반(추측 금지) — 명령어와 실제 출력/에러를 근거로 판정.
- 빌드가 필요하면 빌드 후 실행.
- 추측과 확인 사실 구분 표기.
