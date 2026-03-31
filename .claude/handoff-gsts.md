# Handoff — genshin-ts 포크 관리

## 배경

- 다른 사람이 만든 프로젝트를 포크하여 사용 중
- 기존 master/personal 두 브랜치 → master 단일 브랜치로 전환 완료
- 원본(upstream) repo는 계속 업데이트 중이므로 머지를 고려한 구조 유지 필요
- JSDoc 주석 한국어화는 의도적으로 제외함 — 업스트림 머지 시 대량 충돌 방지 목적

## 관련 프로젝트

```
MiliastraWonderland/
  genshin-ts/        ← 툴체인 소스 (포크) — 이 프로젝트
  gsts-sandbox/      ← genshin-ts 테스트/검증/빌드용 프로젝트 (세팅 완료)
  elementalist/      ← 실전 UGC 개발 프로젝트 (세팅 완료)
  mw-editor/         ← Miliastra Wonderland 에디터 (미래 프로젝트, 미착수)
  genshin-ts-run/    ← 기존 테스트 프로젝트 (레거시, gsts-sandbox로 대체)
```

- gsts-sandbox, elementalist: `github:GsMwDalgi/genshin-ts#master`로 설치. 사용자(본인)만 npm link로 로컬 포크 덮어쓰기
- 팀원: `npm install`만으로 포크 버전 자동 설치 (genshin-ts 클론 불필요)
- 팀원도 사용 — UID/맵 파일이 다르므로 혼동 방지 설정 필요
- mw-editor: GIA/GIL 파일 프로토콜 기반 에디터. 프로토콜 분석은 genshin-ts의 notes/protocol/에 임시 저장 후 mw-editor 시작 시 이관

## 현재 상태

- 마이그레이션 완료: 소스 코드 변경 반영 (`66f4803`), .claude 트래킹 (`c16dfa7`)
- 스킬/문서/세팅 일괄 커밋 (`157f9e4`, 2026-03-31)
- 빌드 테스트 통과 (npm run build, 2026-03-31)
- gsts-sandbox, elementalist 세팅 완료 (빌드 검증됨)
- node-graph-coder 스킬 수정 완료 (self-contained)
- GIA/GIL 프로토콜 분석 완료 (notes/protocol/ 5개 문서)
- 사용자 설치 가이드 완료 (notes/team-setup-guide.md)
- 트리 에이전트 시스템 (`gsts`) 가동 중 — root, root~dev 활성

## 남은 작업

1. **dev ↔ sandbox 빌드 검증** — gsts-sandbox 샘플 예제가 포크 이후에도 정상 동작하는지 확인
2. **genshin-ts 기능 개발** — 구체적 작업 미정, root~dev 대기 중
3. **elementalist 게임 로직 개발** — 세팅만 완료, 실제 개발 미착수
4. **mw-editor 프로젝트 착수** — 프로토콜 문서(notes/protocol/) 이관 후 에디터 개발 시작

## 주요 결정사항

- **업스트림 머지 전략**: 주석 변경 제외로 코드 베이스가 upstream과 유사하게 유지됨. 향후 `git remote add upstream` → `git merge` 방식으로 업데이트
- **.claude/ 트래킹**: 트리 에이전트 상태를 git에 포함하여 프로젝트와 동기화
- **notes/ 문서 구조**: 원본 `docs/`는 건드리지 않고, 개인 문서(아키텍처, 변경로그)는 `notes/`에 분리 — 한국어로 작성, 기술 용어는 영어
- **프로토콜 문서 이관**: notes/protocol/은 mw-editor 착수 시 이관 예정. 이 프로젝트에서는 가공 전 형태로 유지
- **node-graph-coder 스킬**: prompt-reference 문서를 .claude/skills/node-graph-coder/ 내부로 통합 완료 (self-contained)
- **패키지 설치 전략**: 팀원은 `github:GsMwDalgi/genshin-ts#master`로 자동 설치, 본인은 npm link로 로컬 포크 사용
- **create-genshin-ts/templates/start/package.json**: `"genshin-ts": "latest"`는 업스트림 의도대로 유지 (수정 불필요 확인됨)

## 참고

- `src/definitions/nodes.ts` — 15,509줄, 가장 큰 파일. signal args 코드만 추출하여 반영
- `src/thirdparty/Genshin-Impact-Miliastra-Wonderland-Code-Node-Editor-Pack/` — 사용자의 다른 프로젝트 코드 직접 포함 (서브모듈 아님)
- `notes/architecture/` — 8개 아키텍처 문서 (01~08), 리뷰 완료
- `notes/changelog.md` — 오리지널 대비 변경 로그
- `notes/protocol/` — GIA/GIL 프로토콜 분석 5개 문서 (mw-editor 이관 예정)
- `notes/team-setup-guide.md` — 사용자 설치 가이드 (팀원 대상)
- GitHub 포크: https://github.com/GsMwDalgi/genshin-ts
