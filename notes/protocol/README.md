# GIA/GIL Protocol Documentation

Unified documentation for the GIA (node graph) and GIL (map file) binary protocols used by Genshin Impact Beyond Mode.

Merged from two source sets:
- Primary (Korean, authoritative): reverse-engineering analysis with binary comparison verification
- Secondary (English): codebase-level documentation of encoding/injection APIs

## Confidence Levels

- **CONFIRMED**: 코드 분석 + 바이너리 비교로 확정 (code analysis + binary comparison verified)
- **INFERRED**: 코드 분석에서 추론, 바이너리 비교 미실시 (inferred from code, not binary-verified)
- **SPECULATED**: 패턴에서 추측, 검증 필요 (speculated from patterns, needs verification)

## Document Index

| File | Contents |
|------|----------|
| [gia-binary-format.md](gia-binary-format.md) | GIA file binary envelope (20B header, 4B footer) |
| [gil-binary-format.md](gil-binary-format.md) | GIL file binary envelope, top-level structure, file locations |
| [gia-vs-gil.md](gia-vs-gil.md) | Comparison table + relationship diagram |
| [protobuf-schema.md](protobuf-schema.md) | Root, GraphUnit, NodeGraph, GraphNode protobuf definitions |
| [pin-system.md](pin-system.md) | Pin kinds, index namespaces, encoding, array ordering |
| [connections.md](connections.md) | NodeConnection, data/flow direction, encoding functions |
| [type-system.md](type-system.md) | VarType, DataGroup, VarBase.Class, NodeType internal repr |
| [node-identity.md](node-identity.md) | Node IDs, genericId/concreteId, special nodes (300000+) |
| [signal-protocol.md](signal-protocol.md) | send_signal/monitor_signal pin layouts, encoding rules |
| [graph-encoding.md](graph-encoding.md) | Graph builder API, graph modes, variables, comments, composites |
| [injection-flow.md](injection-flow.md) | Full pipeline (TS -> GS IR -> GIA -> GIL), binary patching |
| [gil-scene-objects.md](gil-scene-objects.md) | GIL Scene Object structure, Components, Effect Component |
| [effect-assets.md](effect-assets.md) | Effect asset overview, timed/loop types, ID band structure |
| [effect-catalog-timed.md](effect-catalog-timed.md) | Timed effect ID catalog (1282 entries, pure data) |
| [effect-catalog-loop.md](effect-catalog-loop.md) | Loop effect ID catalog (912 entries, pure data) |
| [gil-reverse-analysis.md](gil-reverse-analysis.md) | Event entry detection, signal reverse ID, variable mapping |
| [gil-test-maps.md](gil-test-maps.md) | Test map registry (reference) |

## Sources

- `genshin-ts/src/thirdparty/.../protobuf/gia.proto.ts` — protobuf 타입 정의
- `genshin-ts/src/thirdparty/.../gia_gen/graph.ts` — Pin/Node/Graph 인코딩
- `genshin-ts/src/compiler/ir_to_gia_transform/` — IR->GIA 변환 로직
- `genshin-ts/src/injector/` — GIL 인젝션 로직
- 테스트 빌드 IR/GIA 출력 비교
- 에디터 스크린샷 대비 검증
