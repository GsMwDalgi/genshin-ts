# Memo

## Task
- TASK_ASSIGN — Analyze GIA/GIL file protocols and document findings (received 2026-03-31, completed)

## Refs
- ../root/workspace/task-protocol-analysis.md | task spec
- notes/protocol/gia-structure.md | GIA binary + protobuf format
- notes/protocol/gil-structure.md | GIL map save format
- notes/protocol/gia-vs-gil.md | comparison
- notes/protocol/injection-flow.md | injection pipeline
- notes/protocol/graph-encoding.md | graph builder encoding details

## Notes
- GIA/GIL share same binary envelope (20B header, 4B footer, magic 0x0326/0x0679)
- GIA proto = Root message (gia.proto v1.5.0), GIL proto is larger undocumented container
- NodeGraph blobs in GIL live at protobuf path 10.1.1
- Signal nodes use placeholder IDs (300000/300001) resolved at injection time
- Graph IDs >= 1e9 are genshin-ts convention
- _GSTS name prefix is safety marker for injection
