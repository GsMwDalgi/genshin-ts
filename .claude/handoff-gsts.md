# Handoff — genshin-ts 포크 관리

## Refs

.claude/skills/node-graph-coder/ | 노드 그래프 코더 스킬 (SKILL.md + prompt-reference 9종) — 차기 갱신 대상
notes/protocol/ | GIA/GIL 프로토콜 역분석 18문서. injection-reference-for-mw-editor.md = mw-editor용 self-contained 참조
notes/architecture/ | 아키텍처 8문서 (01~08)
🎯 node-graph-coder 스킬 갱신 — upstream v0.1.10 머지로 노드 그래프 변경(신규 노드타입 + `defineSignal()`/typed evt.params 시그널 + proto 갱신). 스킬의 node-catalog/events-catalog/genshin-ts-guide/pseudocode가 옛 API·노드셋 참조 → upstream v0.1.10 기준으로 갱신 필요. 진단: 신 소스(`src/`, master) vs 스킬 prompt-reference 대조 / 옛↔신 = `git diff fork-backup-20260607 master`
? .claude/tree/gsts/nodes/fork-sync/workspace/verdict-017.md | #017 upstream 머지 verdict + 마이그레이션 델타(§7: onSignal<Args>→defineSignal, sendSignal(...args)→sendSignal(def,...params))
? notes/integration/ | (STALE) 옛 signal-args 패치 번들 — 분기점 71ca7a1 기준. 머지 완료로 역사적 참고용 (필요시 분기점 9cb31c8로 재생성)

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

**✅ 완료 — upstream 머지 (이번 세션 #017):**
- **원본 v0.1.10(`9cb31c8`) 따라잡기 완료.** 방향 **B**: 우리 signal-args(group A) 폐기 + upstream `defineSignal()`/typed evt.params 채택 (동등성 검증: upstream SignalParamType ⊇ 우리 18종, _list 등가).
- 보존: 버그수정 **B1/B2/B5** (upstream 새 구조 위 재이식 — B1은 upstream이 readField*를 binary.ts로 이동해 signal_nodes 2 + binary 7 = 9건 분산, B2=1, B5=1) + **C CLI**(inspect/scaffold/reader, #015 픽스 포함) + proto upstream 그대로.
- 검증: tsc/build/inspect/scaffold EXIT0, 독립 버그감사(#018) **SAFE**. npm test EXIT1 = upstream 선존버그(pure worktree 재현, 우리 무관).
- **단일 브랜치 통합**: `master` = `b5eb2b6` (로컬 + GitHub origin/master 동일). 옛 포크 = 백업태그 `fork-backup-20260607`(=`8629dc9`, 로컬+GitHub 보존). integ-upstream 삭제. **롤백: `git reset --hard fork-backup-20260607`**.
- ⚠️ 머지 중 `git reset --hard`가 추적되던 .claude/notes 115파일을 워킹트리에서 삭제 → 백업태그에서 전량 복원 후 커밋 (Learnings 참조).

**🎯 다음 세션 — node-graph-coder 스킬 갱신 (사용자 지정):**
- upstream v0.1.10로 노드 그래프 정의 변경(신규 노드타입, `defineSignal` 시그널, proto 갱신) → 스킬이 옛 API/노드셋 참조 중. node-catalog-core/extended, events-catalog, genshin-ts-guide, pseudocode-* 갱신 필요.
- 진단: 신 소스(`src/` on master) vs 스킬 `prompt-reference/` 대조. 옛↔신 차이 = `git diff fork-backup-20260607 master`.

**후속 (낮은 우선순위):**
- 마이그레이션 가이드 문서화 (verdict-017 §7 델타 → 팀/mwe용). downstream이 시그널 쓸 때 필요.
- notes/integration/ 베이스라인 번들 = 분기점 9cb31c8로 재생성(필요시).
- 고아 노드 `fork-steward_removed` 정리.
- **mwe 개발 지속** + genshin-ts injector 통합.

**이전 완료 (참고):** #012 인벤토리, #013 inspect/scaffold 검증, #014 baseline, #015 scaffold 수정(`594a6e8`), #016 C2 refresh, 마이그레이션, signal args/버그수정, 프로토콜 18문서, GIL 호환성 테스트.

## Decisions

- **업스트림 머지 전략**: 주석 변경 제외로 코드 베이스를 upstream과 유사하게 유지
- **브랜치 구조 (단일)**: 옛날 2브랜치 관리 → 경험부족으로 합침. 현재 **단일 `master`** 운영. 머지는 임시 `integ-upstream`에서 작업 후 master로 통합·삭제. 옛 상태는 **백업태그**로 보존(브랜치는 유지 안 함). 사용자 git 경험 적음 → force-push는 백업태그 push 선행 후 진행. 레포 사용자 없음(필요시 비공개 전환 예정)
- **통합 접근 (실행됨 #017)**: upstream이 signal-args 동등기능(`defineSignal`)을 자체 도입 → 미션 1축 예외 발동, **우리 group A 폐기·upstream 채택**(동등성 review 확인 후). 유지 = 버그수정 3종(B1 varint guard / B2 any 타입가드 / B5 parseValue entity fallback) + C CLI 도구. **버그수정 3종 무심코 롤백 금지(치명)** — upstream에 없음
- **protobuf (정정)**: 우리는 .proto 미변경. 이전 handoff의 "protobuf 차이" 항목은 **upstream `71ca7a1` 자체 변경 + 역분석 노트**였음 (우리 코드 아님). 머지 시 최신 upstream proto 그대로 채택
- **.claude/ 트래킹**: 트리 에이전트 상태를 git에 포함 (단 `_removed` 고아 노드는 커밋 제외). ⚠️ upstream 기반 머지(reset --hard)는 upstream이 .claude/를 모르므로 추적 .claude/notes를 워킹트리에서 삭제함 → 머지 후 백업태그에서 복원 + 재커밋 필수
- **notes/ 문서 구조**: 원본 `docs/`는 미변경, 개인 문서는 `notes/`에 분리
- **fork 유지보수 조직**: `fork-sync`(distributor) + 영속 `fork-sync_impl`/`fork-sync_review` 쌍. 구현≠리뷰 분할(자가검수 금지, 관점 전환). opus 사용

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
| 노드명 `_` 분리자 | tree-agent 스킬 갱신됨: 분리자가 `_` (런타임이 `~` 거부), 노드명=네이티브 핸들 일치. `{parent}_{leaf}` (예: `fork-sync_impl`). 옛 `~` 노드는 `_`로 마이그레이션 완료 |
| cross-gen shutdown race | 같은 이름 노드에 보낸 shutdown_request가 다음 세대 메일박스로 재전달돼 신 세대가 즉시 종료된 사례. 재활성화 spawn 프롬프트에 'stale shutdown 거부' 안전장치 권장 |
| 네임 드리프트 | 원본이 아직 등록된 채 같은 이름 재spawn하면 `name-2` 중복 생성 → 중복본 종료. 재spawn 전 원본 liveness 확인 |
| 머지 reset가 추적파일 삭제 | upstream 기반 `git reset --hard`는 upstream에 없는 추적 .claude/notes를 워킹트리에서 삭제 → 머지 후 `git checkout <백업태그> -- notes/ .claude/skills/ ...`로 복원 필수 (git add는 워킹트리 존재분만 담음) |
| 노드 재사용 | 영속 impl/review 쌍은 디렉티브마다 재사용. per-directive 노드 난립 금지 |
| 백그라운드 task 결과 유실 | 백그라운드 volatile task의 인라인 보고가 0바이트로 유실된 사례 → 보고가 중요한 task는 foreground 실행 |

## 참고

- `notes/changelog.md` — 포크 변경 로그 (#1~#9)
- `notes/protocol/` — GIA/GIL 프로토콜 18개 통합 문서 (CONFIRMED/INFERRED/SPECULATED 분류)
- `notes/architecture/` — 8개 아키텍처 문서 (01~08)
- `notes/fork-changes-inventory.md` — 충돌 위험도별 HIGH 핫스팟: signal-args 7파일 (runtime/core.ts, definitions/nodes.ts[sendSignal+parseValue], compiler/ir_to_gia_transform/{mappings,index,pins}.ts, injector/signal_nodes.ts)
- GitHub 포크: https://github.com/GsMwDalgi/genshin-ts
- 대상 .gil 샘플: `C:\Users\Rterg\AppData\LocalLow\miHoYo\Genshin Impact\BeyondLocal\804101570\Beyond_Local_Save_Level\1073741976.gil` (184 그래프, 최대 AstroGear_05_Aegis 440노드)
