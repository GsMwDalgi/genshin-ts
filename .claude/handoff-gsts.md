# Handoff — genshin-ts 포크 관리

## 배경

- 다른 사람이 만든 프로젝트를 포크하여 사용 중
- 원본(upstream) repo는 계속 업데이트 중이므로 머지를 고려한 구조 유지 필요
- JSDoc 주석 한국어화는 의도적으로 제외함 — 업스트림 머지 시 대량 충돌 방지 목적

## 관련 프로젝트

```
MiliastraWonderland/
  genshin-ts/        ← 툴체인 소스 (포크) — 이 프로젝트
  gsts-sandbox/      ← genshin-ts 테스트/검증/빌드용 (npm link 연결됨)
  elementalist/      ← 실전 UGC 개발 프로젝트 (npm link 연결됨)
  mw-editor/         ← GIA/GIL 바이너리 맵 에디터 (설계 논의 완료, 미착수)
  legacy/genshin-ts  ← 마이그레이션 전 원본 (참조용 보존)
  legacy/genshin-ts-run ← 레거시 테스트 프로젝트 (마이그레이션 완료)
```

- gsts-sandbox, elementalist: `github:GsMwDalgi/genshin-ts#master`로 설치. 본인만 npm link로 로컬 포크 덮어쓰기
- 팀원: `npm install`만으로 포크 버전 자동 설치 (genshin-ts 클론 불필요)
- UID/맵 파일 혼동 방지: `gsts.config.ts`를 `.gitignore`에 추가 + `gsts.config.template.ts` 제공

## 현재 상태

- **마이그레이션 완료**: 소스, 샘플 파일, 프로토콜 문서 전부 이관됨
- **버그 수정 완료** (`9415773`): signal args 전파 (`core.ts`), entity parseValue 폴백 (`nodes.ts`), monitor_signal output pin (`ir_to_gia_transform`)
- **프로토콜 문서 통합 완료**: 14개(gia-protocol) + 5개(기존) → 18개 통합 문서 (notes/protocol/)
- **전체 빌드 테스트 통과**: 6개 샘플 폴더 전부 GIA 생성 성공 (sandbox에서 검증)
- **마이그레이션 최종 완료**: genshin-ts-run → legacy/ 이동 완료. 샘플 파일, prompt-reference, gia-protocol 문서 전부 이관됨

## 남은 작업

1. **3개 프로젝트 push** — genshin-ts, gsts-sandbox, elementalist 모두 push 대기 중
3. **mw-editor 개발 착수** — 설계 논의 완료 (DISCUSSION.md), 프로젝트 초기화 필요
4. **elementalist 게임 로직 개발** — 세팅만 완료, 실제 개발 미착수

## 주요 결정사항

- **업스트림 머지 전략**: 주석 변경 제외로 코드 베이스가 upstream과 유사하게 유지됨
- **.claude/ 트래킹**: 트리 에이전트 상태를 git에 포함하여 프로젝트와 동기화
- **notes/ 문서 구조**: 원본 `docs/`는 건드리지 않고, 개인 문서는 `notes/`에 분리
- **node-graph-coder 스킬**: prompt-reference 문서를 .claude/skills/node-graph-coder/ 내부로 통합 완료 (self-contained)
- **패키지 설치 전략**: 팀원은 git URL, 본인은 npm link
- **protobuf 차이**: upstream 대비 신규 노드 타입(CreationStatusDecision/Skill/Status), 필드명 확정(xxx→entrySlotIndex, xxxx→evaluationInterval), Dictionary 타입, ClientSignal(kind=6) 등 추가. 게임 클라이언트 업데이트 + 역분석 결과 반영

### mw-editor 설계 결정

- **기술 스택**: TypeScript + Electron + React (에이전트 생산성 최대화, genshin-ts 자산 재활용)
- **genshin-ts 의존 범위**: `injector/` 모듈만 사용 (GIL 읽기/쓰기, GIA 주입). compiler/runtime/definitions는 불필요
- **코드 스타일**: 도메인 모델은 클래스 기반 (유저가 C#/WPF 배경), UI는 함수형 React
- **genshin-ts 함수 래핑**: 함수 지향인 genshin-ts를 클래스로 캡슐화 → 추후 자체 파서로 교체 가능
- **개발 방식**: 트리 에이전트로 전부 구현 (구조 노드 + 구현 노드 + 테스트 노드)
- **프로토콜 문서**: notes/protocol/에 임시 보관, mw-editor에서 기능 구현 시마다 문서도 함께 업데이트하는 전략
- **목표**: 스킬 이펙트 위치/좌표 수정, 프리팹 컴포넌트 편집, 스킬 속성 JSON export → 인젝션 원복, 이펙트 임베딩 검색
- **상세 논의**: `D:\MyDrive\Repos\MiliastraWonderland\mw-editor\DISCUSSION.md`

## 참고

- `notes/changelog.md` — 포크 변경 로그 (#1~#9, signal args/버그수정/CLI/프로토콜 문서 통합)
- `notes/protocol/` — GIA/GIL 프로토콜 분석 18개 통합 문서 (확신도 CONFIRMED/INFERRED/SPECULATED 분류)
- `notes/architecture/` — 8개 아키텍처 문서 (01~08)
- SETUP.md — gsts-sandbox, elementalist 각 프로젝트에 팀원용 설치 가이드 포함
- GitHub 포크: https://github.com/GsMwDalgi/genshin-ts
