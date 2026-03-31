# Injection Flow

## Overview

The injection system replaces a single NodeGraph inside a GIL (map) file with a new NodeGraph generated from compiled TypeScript code. The pipeline is:

```
TypeScript -> GS IR -> IR JSON -> GIA binary -> Inject into GIL
```

## High-Level Pipeline

```
+----------+    +----------+    +----------+    +----------+    +----------+
| .ts file |--->|  GS IR   |--->| IR JSON  |--->|  .gia    |--->| patched  |
| (source) |    | (interme)|    | (serial) |    | (binary) |    |  .gil    |
+----------+    +----------+    +----------+    +----------+    +----------+
  ts_to_gs       gs_to_ir       ir_to_gia        injector
```

## Step 1: IR JSON to GIA (ir_to_gia_transform)

Entry: `src/compiler/ir_to_gia_transform/index.ts` -> `irToGia()`

1. Takes an `IRDocument` object (parsed from JSON)
2. Builds a `Graph` object using the gia_gen builder API
3. Calls `Graph.encode()` to produce a `Root` protobuf structure
4. Serializes with `wrap_gia()` which:
   - Encodes the Root message via protobufjs
   - Prepends 20-byte header (left_size, schema=1, head_tag=0x0326, file_type=3, proto_size)
   - Appends 4-byte tail (tail_tag=0x0679)
5. Writes to `.gia` file

## Step 2: GIA Injection into GIL (injector)

Entry: `src/injector/index.ts` -> `createInjector()` -> `injectBytes()`

### Input
- `gilBytes`: raw GIL file bytes
- `giaBytes`: raw GIA file bytes (compiled output)
- `targetId`: NodeGraph ID to replace (typically >= 1,000,000,000)

### Process

1. **Load GIA graph**: `loadGiaGraph()`
   - Strip GIA envelope (20B header + 4B tail)
   - Decode with protobufjs `Root` message
   - Extract `Root.graph.graph.inner.graph` (the NodeGraph object)
   - If targetId provided and differs from graph's ID, overwrite it

2. **Parse GIL structure**:
   - Validate GIL header tags (0x0326 / 0x0679)
   - Extract payload (bytes 20 to EOF-4)
   - Run `parseMessage()` to build field index (recursive protobuf wire-format scanner)
   - Collect NodeGraph blob fields at path 10.1.1

3. **Patch signal node IDs**: `patchSignalNodeIds()`
   - Scans the new graph for placeholder node IDs (300000=send, 300001=monitor)
   - Looks up actual signal node IDs from composite definitions in the GIL
   - Replaces placeholder IDs with real ones
   - Signal names are extracted from ClientExecNode pins (kind=5) with string values

4. **Find target NodeGraph**: `findNodeGraphTargets()`
   - Scan 10.1.1 blob fields
   - Fast varint reader checks `NodeGraph.Id.id` without full decode
   - When matched, full protobuf decode to get the NodeGraph object
   - Must find exactly 1 match (0 = error, >1 = abort)

5. **Validate**:
   - Resolve graph type from folder index (category value -> type mapping)
   - Check existing graph type matches expected type
   - Check incoming graph type matches expected type
   - If target graph is non-empty and name doesn't start with `_GSTS`, abort (safety check)

6. **Replace**:
   - Copy graph name from new graph to target
   - Set graph ID and type on new graph
   - Verify new graph with protobufjs `verify()`
   - Encode new graph to bytes with protobufjs `encode()`
   - `applyReplacement()`: binary patch (see below)

7. **Rebuild file**: `buildFile()`
   - Recompute header with new payload size
   - Write: header(20B) + new_payload + tail(4B)
   - Preserve original schema version, head_tag, file_type, tail_tag

### Output
- `bytes`: new GIL file bytes
- `mode`: always `'replace'`

## Binary Patching Details (applyReplacement) [CONFIRMED]

The most delicate part of injection is updating the protobuf binary in-place:

```
1. Replace target blob:
   - Old: [varint_len][old_data]
   - New: [varint_len'][new_data]
   - Delta = new_data_len - old_data_len + (new_varint_size - old_varint_size)

2. Update ancestor lengths:
   - Find all LenFields that contain the target field
   - For each ancestor (innermost first):
     - Compute new length = old_length + accumulated delta
     - Re-encode length as varint
     - If varint size changed, add to delta
   - Apply all patches sorted by descending offset
```

This ensures the protobuf container structure remains valid after replacing an arbitrary-depth nested message.

## Signal Node Resolution (Injection Time) [CONFIRMED]

### Placeholder IDs (compile time)

| Placeholder | Kind    |
|-------------|---------|
| 300000      | send    |
| 300001      | monitor |

### Resolution Process

1. Scan GIL for composite definitions (CompositeDef) that have signal definitions
2. Extract signal name from CompositeDef field 107 -> nested field 101/102 -> name field 1
3. Determine kind: `type=20002` -> sendServer, else `outputs>=3` -> monitor, else send
4. Build map: signal_name -> {send?, monitor?, sendServer?} -> NodeGraphIdInfo
5. For each graph node with placeholder ID:
   - Extract signal name from ClientExecNode pin (kind=5, bString value)
   - Look up resolved ID from map
   - Apply full NodeGraphIdInfo (class, type, kind, nodeId) to both genericId and concreteId

## File Injection (injectFile) [CONFIRMED]

Simple wrapper around `injectBytes`:
1. Read GIL and GIA files from disk
2. Call `injectBytes()`
3. Write result to `outPath` (defaults to overwriting original GIL)

## Sources

- `src/compiler/ir_to_gia_transform/` — IR -> GIA 변환
- `src/injector/index.ts` — createInjector, injectBytes
- `src/injector/node_graph.ts` — NodeGraph 스캔/패치
- `src/injector/signal_nodes.ts` — 시그널 노드 해석
- `protobuf/decode.ts` — wrap/unwrap
