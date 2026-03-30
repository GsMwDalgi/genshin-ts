# 인수인계 문서 — genshin-ts 마이그레이션

## 배경

- genshin-ts는 다른 사람이 만든 프로젝트를 포크하여 사용 중
- 기존에 master/personal 두 브랜치로 관리했으나, 기능 추가가 어려워 master 단일 브랜치로 전환
- 원본(upstream) repo는 계속 업데이트 중이므로 향후 머지를 고려한 구조 유지 필요

## 레포 정보

- 원본 repo (포크): D:/MyDrive/Repos/MiliastraWonderland/genshin-ts
- 작업 repo (마이그레이션 대상): D:/MyDrive/Repos/MiliastraWonderland/temp/genshin-ts
- 포크 시점 커밋: 6f288e9 (release v0.1.7)
- 마이그레이션 완료 후 작업 repo를 원본 repo 경로로 이동 예정

## 완료된 작업

### 1. .gitignore 업데이트 (커밋 완료)
- .vs, .claude 추가
- 커밋: eda452b

### 2. 소스 코드 변경 반영 (스테이징됨, 커밋 안 함)
- 20개 파일 스테이징 상태
- 포크 시점 대비 모든 소스 코드 변경을 반영함
- 주요 변경:
  - Signal arguments 기능 (end-to-end): value.ts, meta_call_types.ts, IR.d.ts, ir_builder.ts, nodes.ts, server_globals.ts/d.ts, core.ts, compiler 관련 파일들
  - 새 CLI 명령어: gil_inspect.ts, gil_scaffold.ts, reader.ts
  - 버그 수정: signal_nodes.ts (protobuf 파서 오버플로 가드), ts_type_utils.ts (Any 타입 조기 리턴)
  - i18n: en-US, zh-CN 로케일에 inspect/scaffold 키 추가
  - events-payload.ts: value import 1줄 추가

### 3. 의도적으로 제외한 것
- **JSDoc 주석 한국어화**: enum.ts, events-payload.ts, nodes.ts, server_globals.d.ts, core.ts에 있던 EN/ZH → KO 주석 변경은 모두 제외. 이유: 업스트림 머지 시 15,000줄짜리 nodes.ts 등에서 대량 충돌 발생 방지
- **설정 파일**: package-lock.json (npm install로 재생성), template/package.json (personal 브랜치 참조라 수정 필요)
- **.claude/**: gitignore에 포함, 마지막에 별도 추가 예정

## 문서 구조

원본 repo의 `docs/`는 건드리지 않고, 개인 문서는 `notes/`에 분리:

```
notes/
  changelog.md              — 오리지널 대비 변경 로그 (기능별 정리)
  architecture/
    01-overview.md           — 전체 프로젝트 구조
    02-compilation-pipeline.md — 컴파일 파이프라인
    03-runtime.md            — 런타임
    04-ir-format.md          — IR 포맷
    05-injector.md           — 인젝터
    06-cli.md                — CLI
    07-definitions.md        — 정의 파일
    08-module-dependencies.md — 모듈 의존성
```

- 한국어로 작성, 기술 용어는 영어 유지
- 통일성/일관성 + 정확성/완성도 리뷰 완료

## 남은 작업

1. **프로젝트 이동** — temp/genshin-ts → genshin-ts 경로로 이동
2. **.claude 업로드** — 트리 노드 데이터 백업 후 마지막에 추가
3. **트리 에이전트 재구성** — 기능 개발, 유지보수, 업데이트 용도로 전환
4. **설정 파일 수정** — template/package.json의 dependency 참조를 personal → master로 변경
5. **빌드 테스트 통과 확인됨** — npm run build 성공 (2026-03-31)

## 업스트림 머지 전략

- 주석 변경을 제외했으므로 코드 베이스가 업스트림과 유사
- 향후 업스트림 업데이트 시:
  1. `git remote add upstream {url}`
  2. `git fetch upstream`
  3. `git diff upstream/master~1..upstream/master` 로 변경 확인
  4. `git merge` 또는 수동 적용
- 충돌은 실제 코드 변경 부분에서만 발생할 것으로 예상

## 프로젝트 구조 참고

- src/thirdparty/Genshin-Impact-Miliastra-Wonderland-Code-Node-Editor-Pack/ — 사용자의 다른 프로젝트 코드를 직접 포함 (서브모듈 아님, git 트래킹 중)
- src/definitions/nodes.ts — 15,509줄, 가장 큰 파일. signal args 코드만 추출하여 반영함
- create-genshin-ts/ — 템플릿 프로젝트, 나중에 dependency 참조 경로 수정 필요 (personal → master)
