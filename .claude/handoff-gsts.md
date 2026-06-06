# Handoff — genshin-ts 포크 관리

## Refs

notes/fork-changes-inventory.md | 포크 변경 전체 인벤토리 (upstream `71ca7a1` 위 선형, 카테고리별 + 충돌 위험도). 머지 기준 자료
notes/integration-workflow.md | upstream 머지 워크플로 / 추적 문서
notes/integration/ | signal-args 패치 번들(20) + review-audit (재적용 가능)
notes/protocol/injection-reference-for-mw-editor.md | mw-editor용 인젝션 참조 (self-contained)
🎯 upstream 머지 진입 — 원본 repo URL 확보 후: upstream remote 등록 → fetch → 패치 번들 재적용 → 충돌 핫스팟(signal-args 7파일) 검증
? .claude/tree/gsts/nodes/fork-sync/workspace/verdict-014.md | #014 통합 baseline verdict
? .claude/tree/gsts/nodes/fork-sync/workspace/verdict-015.md | #015 scaffold 수정 verdict

## Background

- 다른 사람이 만든 프로젝트를 포크하여 사용 중. upstream(원본)은 계속 업데이트 중이므로 머지 가능 구조 유지 필요
- JSDoc 주석 한국어화는 의도적 제외 — upstream 머지 시 대량 충돌 방지
- 핵심 미션: **우리 변형 유지 + upstream 변형 최소화(원본 형태 최대 유지) + 통합 상태 지속 유지**

## 관련 프로젝트

```
MiliastraWonderland/
  genshin-ts/        ← 툴체인 소스 (포크) — 이 프로젝트
  gsts-sandbox/      ← genshin-ts 테스트/검증/빌드 (npm link 연결)
  mw-editor/ (mwe)   ← GIA/GIL 바이너리 맵 에디터 — 개발 진행 중
  legacy/genshin-ts  ← 마이그레이션 전 원본 (참조용)
  legacy/genshin-ts-run ← 레거시 테스트 (마이그레이션 완료)
```

- **elementalist: 팀 해산으로 개발 취소** — npm link / 참조 / 정보 정리 대상 (더 이상 관리 안 함)
- **mwe**: 진행 중. 최종적으로 genshin-ts `injector/` 기능을 통합 — 인게임 에디터 + Claude 서포트 코딩 양쪽 지원. scaffold 코드젠이 #015로 수정되어 통합 준비됨
- 패키지 설치: 팀원은 git URL(`github:GsMwDalgi/genshin-ts#master`), 본인은 npm link로 로컬 포크 덮어쓰기
- UID/맵 혼동 방지: `gsts.config.ts`는 `.gitignore`, `gsts.config.template.ts` 제공

## Progress

**완료 (이번 세션 #012~#015):**
- **#012** 포크 변경 인벤토리 → `notes/fork-changes-inventory.md`. 핵심: 포크는 upstream `71ca7a1` 위 **선형**, `git diff 71ca7a1 HEAD`가 권위 델타. 우리는 **proto/gia_gen/thirdparty 미변경**
- **#013** inspect/scaffold/reader 실제 .gil 검증 — 184/184 그래프 디코드 OK. scaffold 코드젠 버그 2개 발견 → #015로 이어짐
- **#014** 통합 baseline PASS — signal-args 패치 번들(20) + `notes/integration/`, `notes/integration-workflow.md`
- **#015** scaffold 코드젠 수정 PASS — `src/cli/gil_scaffold.ts` (vec3 이중래핑 + list/dict bare-comment + import emit). **커밋됨 `594a6e8`**, tsc EXIT=0 실증, 독립 검수 통과

**진행/남은:**
1. **C2 baseline 패치 refresh** — #015 커밋 후 신 델타 반영 (fork-sync 처리 중 / 세션 마무리 커밋에 포함 예정)
2. 🎯 **upstream 머지** — 원본 repo URL 확보 시 시작. baseline 준비 완료. 충돌 핫스팟 = signal-args 7파일
3. **mwe 개발 지속** + genshin-ts injector 통합

**이전 완료 (참고):** 마이그레이션, signal args/버그수정, 프로토콜 문서 18개 통합, GIL 호환성 테스트, injector 문서 보강

## Decisions

- **업스트림 머지 전략**: 주석 변경 제외로 코드 베이스를 upstream과 유사하게 유지
- **통합 접근**: 우리 변형 = signal-args 기능 + 버그수정 3종(B1 varint guard / B2 any 타입가드 / B5 parseValue entity fallback) + CLI 도구. upstream에 동일 기능 도입 시 우리 변형 버리고 원본 채택, 변형 최소화 유지. 버그수정 3종은 무심코 롤백 금지(치명)
- **protobuf (정정)**: 우리는 .proto 미변경. 이전 handoff의 "protobuf 차이" 항목은 **upstream `71ca7a1` 자체 변경 + 역분석 노트**였음 (우리 코드 아님). 머지 시 최신 upstream proto 그대로 채택
- **.claude/ 트래킹**: 트리 에이전트 상태를 git에 포함 (단 `_removed` 고아 노드는 커밋 제외)
- **notes/ 문서 구조**: 원본 `docs/`는 미변경, 개인 문서는 `notes/`에 분리
- **fork 유지보수 조직**: `fork-sync`(distributor) + 영속 `~impl`/`~review` 쌍. 구현≠리뷰 분할(자가검수 금지, 관점 전환). opus 사용

### mw-editor 설계 결정 (유지)
- **기술 스택**: TypeScript + Electron + React
- **genshin-ts 의존 범위**: `injector/` 모듈만 (GIL 읽기/쓰기, GIA 주입). compiler/runtime/definitions 불필요
- **코드 스타일**: 도메인 모델 클래스 기반(유저 C#/WPF 배경), UI 함수형 React. genshin-ts 함수를 클래스로 래핑 → 추후 자체 파서 교체 가능
- **개발 방식**: 트리 에이전트로 구현
- **목표**: 스킬 이펙트 위치/좌표 수정, 프리팹 컴포넌트 편집, 스킬 속성 JSON export → 인젝션 원복, 이펙트 임베딩 검색
- **상세**: `D:\MyDrive\Repos\MiliastraWonderland\mw-editor\DISCUSSION.md`

## Learnings

| 항목 | 내용 |
|---|---|
| R: 드라이브 | 램디스크 — 임시/분석 작업에 활용 가능 (저장소 루트 오염 금지) |
| tree-cli DEAD 오탐 | `tree-cli send`의 DEAD는 오탐 가능. 살아있는 idle teammate는 인박스 seed 후 `SendMessage`로 깨우면 됨 (재spawn 전 무응답 확인 — 살아있는데 재spawn하면 네임 드리프트) |
| 노드명 `~` 제약 | Agent `name` 정규식이 `~` 불가 → 네이티브 핸들은 하이픈(`fork-sync-impl`), tree 노드명/ tree-cli 호출은 틸드(`fork-sync~impl`)로 분리 |
| 노드 재사용 | 영속 impl/review 쌍은 디렉티브마다 재사용. per-directive 노드 난립 금지 |
| 백그라운드 task 결과 유실 | 백그라운드 volatile task의 인라인 보고가 0바이트로 유실된 사례 → 보고가 중요한 task는 foreground 실행 |

## 참고

- `notes/changelog.md` — 포크 변경 로그 (#1~#9)
- `notes/protocol/` — GIA/GIL 프로토콜 18개 통합 문서 (CONFIRMED/INFERRED/SPECULATED 분류)
- `notes/architecture/` — 8개 아키텍처 문서 (01~08)
- `notes/fork-changes-inventory.md` — 충돌 위험도별 HIGH 핫스팟: signal-args 7파일 (runtime/core.ts, definitions/nodes.ts[sendSignal+parseValue], compiler/ir_to_gia_transform/{mappings,index,pins}.ts, injector/signal_nodes.ts)
- GitHub 포크: https://github.com/GsMwDalgi/genshin-ts
- 대상 .gil 샘플: `C:\Users\Rterg\AppData\LocalLow\miHoYo\Genshin Impact\BeyondLocal\804101570\Beyond_Local_Save_Level\1073741976.gil` (184 그래프, 최대 AstroGear_05_Aegis 440노드)
