# Injection Pipeline Reference for mw-editor

Self-contained reference for reimplementing GIL injection logic. Based on genshin-ts source code analysis (2026-04-09).

---

## 1. Overview

The injection system replaces a single **NodeGraph** inside a `.gil` (map save) file with a new NodeGraph from a compiled `.gia` file.

```
.gia file (compiled graph) + .gil file (map save) → patched .gil file
```

Both `.gia` and `.gil` files share the same binary envelope format but contain different protobuf schemas internally.

---

## 2. Binary Envelope Format (공통 바이너리 봉투)

Both GIL and GIA files use an identical 20-byte header + 4-byte tail structure. All multi-byte integers are **big-endian**.

### Header (20 bytes)

| Offset | Size | Field            | Formula / Value      | Description                    |
|--------|------|------------------|----------------------|--------------------------------|
| 0x00   | 4B   | `left_size`      | `file_size - 4`      | Remaining file size after this field |
| 0x04   | 4B   | `schema_version` | `1` (observed)       | Always 1 in current game version |
| 0x08   | 4B   | `head_tag`       | `0x0326`             | Magic number (매직 넘버)        |
| 0x0C   | 4B   | `file_type`      | `2` (GIL) / `3` (GIA) | File type discriminator        |
| 0x10   | 4B   | `proto_size`     | `file_size - 24`     | Protobuf payload byte length   |

### Tail (4 bytes)

| Offset | Size | Field       | Value    |
|--------|------|-------------|----------|
| EOF-4  | 4B   | `tail_tag`  | `0x0679` |

### Payload

```
protobuf_payload = file_bytes[20 .. file_size - 4]
```

**Source:** `src/injector/binary.ts:130-134` (readUint32BE), `src/injector/binary.ts:197-211` (buildFile)

### Validation

genshin-ts only validates `head_tag == 0x0326` and `tail_tag == 0x0679`. It does NOT validate `file_type` or `schema_version`. These fields are read and preserved in the output but not asserted.

**Source:** `src/injector/index.ts:89`

---

## 3. GIA File Structure

A `.gia` file contains a single NodeGraph wrapped in a protobuf `Root` message.

### Unwrapping

```
gia_payload = gia_bytes[20 .. length - 4]    // strip envelope
root = protobuf_decode(gia_payload, Root)     // decode with Root message from gia.proto
node_graph = root.graph.graph.inner.graph     // 4-level deep navigation
```

**Source:** `src/injector/node_graph.ts:70-72` (unwrapGia), `src/injector/node_graph.ts:110-131` (loadGiaGraph)

### Protobuf Schema

The `Root` message is defined in `gia.proto`. Key nesting:
```
Root
  └── graph (RootGraph)
        └── graph (GraphContainer)
              └── inner (InnerContainer)
                    └── graph (NodeGraph)    ← this is what we inject
```

---

## 4. GIL File Structure

A `.gil` file contains the entire map save as a protobuf message with a **different schema** from `Root` in gia.proto.

### Relevant Protobuf Paths

| Path      | Content                        | Description                           |
|-----------|--------------------------------|---------------------------------------|
| `6.1`     | FolderEntry (repeated)         | Folder metadata / graph index         |
| `6.1.3`   | Folder content                 | Name + entry list                     |
| `6.1.3.5` | Content entry items            | `{typeValue, id}` pairs               |
| `6.1.2.4` | MetaList                       | Alternative entry list                |
| `6.1.2.4.5` | Meta entry items             | `{typeValue, id}` pairs               |
| `10.1.1`  | NodeGraph blobs (repeated)     | Self-contained protobuf NodeGraph messages |

NodeGraph blobs at path `10.1.1` use the same `NodeGraph` message from `gia.proto` and can be independently decoded.

### Key Constraint

The GIL is **NOT** decoded using protobufjs as a whole (too slow for large files). Instead, a raw wire-format scanner builds a positional index of all length-delimited fields.

---

## 5. Raw Protobuf Scanner (`parseMessage`)

**Source:** `src/injector/binary.ts:34-128`

This is the core parsing routine. It recursively walks protobuf wire format without knowing the schema.

### Algorithm

```
parseMessage(buf, start, end, depth, p0..p5, out_fields, collectors):
  while offset < end:
    read key varint → field_number, wire_type
    switch wire_type:
      0 (varint): read and skip varint
      1 (fixed64): skip 8 bytes
      2 (length-delimited):
        read length varint
        record LenField { field, depth+1, p0..p5, lenOffset, lenSize, dataStart, dataEnd }
        if depth=3 and path is 10.1.1: add to nodeGraphBlobFields collector
        if depth < 6: recurse into data range
      5 (fixed32): skip 4 bytes
      other: stop (unknown wire type)
```

### LenField Structure (핵심 데이터 구조)

```typescript
type LenField = {
  field: number      // protobuf field number
  depth: number      // nesting depth (1-indexed after parse)
  p0..p5: number     // ancestor field numbers at each depth level
  lenOffset: number  // byte offset of the length varint
  lenSize: number    // byte size of the length varint encoding
  dataStart: number  // byte offset where data begins
  dataEnd: number    // byte offset where data ends
}
```

This structure allows:
- **Path matching**: filter by `(depth, p0, p1, p2, ...)` to find fields at specific protobuf paths
- **Binary patching**: `lenOffset`/`lenSize` enable surgical replacement of length prefixes
- **Ancestor tracking**: finding containing fields for length updates

### Depth Limit

Recursion stops at depth 6. Fields nested deeper are not indexed.

---

## 6. Fast NodeGraph ID Scanner

**Source:** `src/injector/node_graph.ts:13-68`

Before doing expensive protobufjs decode, a fast varint-level scanner checks each blob:

```
tryReadNodeGraphIdAndType(bytes):
  1. Read first key varint; must be 10 (field=1, wire=2 → embedded message = NodeGraph.Id)
  2. Read length of Id sub-message
  3. Within Id sub-message:
     - field 2 (varint) → type (NodeGraph.Id.type)
     - field 5 (varint) → id (NodeGraph.Id.id)
  4. Return {id, type} or null if signature doesn't match
```

Only when `id == targetId` does the injector perform full protobufjs decode.

---

## 7. Injection Pipeline (Step by Step)

**Source:** `src/injector/index.ts:66-198`

### Step 1: Load GIA Graph

```
giaPayload = giaBytes[20..-4]
rootMsg = Root.decode(giaPayload)
newGraph = rootMsg.graph.graph.inner.graph
if targetId != newGraph.id.id: overwrite id
```

### Step 2: Parse GIL Structure

```
validate headTag == 0x0326, tailTag == 0x0679
payload = gilBytes[20..-4]
fields = []
nodeGraphBlobFields = []
parseMessage(payload, 0, len, 0, ..., fields, { nodeGraphBlobFields })
```

### Step 3: Patch Signal Node IDs

```
patchSignalNodeIds(newGraph, gilBytes, { payload, fields })
```

See Section 9 for details.

### Step 4: Find Target NodeGraph

```
matches = findNodeGraphTargets(payload, nodeGraphBlobFields || fields, NodeGraph, targetId)
assert matches.length == 1
```

For targetId >= 1,000,000,000: additionally verify matched field is at path 10.1.1.

### Step 5: Validate Graph Type

```
folderIndexes = collectFolderIndexes(payload, fields)       // folder.ts:150-223
entryField = findFolderEntryField(payload, fields, targetId) // folder.ts:225-258
graphType = resolveGraphTypeForTypeValue(entryField.entry.typeValue, folderIndexes, idToType)
```

Folder entry resolution searches two paths:
- Content entries at `6.1.3.5` (preferred)
- Meta entries at `6.1.2.4.5` (fallback)

Graph type resolution fallback chain:
1. Infer from existing graphs with same typeValue (scan all NodeGraph blobs for matching typeValue, use their `NodeGraph.Id.type`)
2. If exactly one type found: use it
3. If none found: use static map (`DEFAULT_VALUE_TO_GRAPH_TYPE`)

**Graph Type Mapping (typeValue -> graph type):**

| typeValue (GIL folder) | Graph Type (NodeGraph.Id.type) | Name   |
|-------------------------|-------------------------------|--------|
| 800                     | 20000                         | entity |
| 2300                    | 20003                         | status |
| 2400                    | 20004                         | class  |
| 4300                    | 20005                         | item   |

**Source:** `src/injector/folder.ts:4-9`

Type mismatch between existing/incoming graph and folder-resolved type produces a **warning** but does NOT abort injection.

### Step 6: Safety Check

```
if target.nodes.length > 0 AND NOT target.name.startsWith("_GSTS"):
  abort  (unless skipNonEmptyCheck is set)
```

### Step 7: Replace

```
newGraph.name = target existing name (if new graph has a name, copy it to target)
setGraphId(newGraph, targetId)
setGraphType(newGraph, graphType)
verify(newGraph)   // protobufjs verify
newGraphBytes = NodeGraph.encode(newGraph).finish()
```

### Step 8: Binary Patch

```
newPayload = applyReplacement(payload, fields, targetField, newGraphBytes)
```

See Section 8 for details.

### Step 9: Rebuild File

```
newFile = buildFile(newPayload, { schema, headTag, fileType, tailTag })
```

Writes: header(20B) + newPayload + tail(4B). Preserves original schema/headTag/fileType/tailTag values.

---

## 8. Binary Patching (`applyReplacement`)

**Source:** `src/injector/binary.ts:159-195`

The most delicate operation. Must update the target blob AND all ancestor length prefixes to maintain valid protobuf structure.

### Algorithm

```
1. Create patch for target blob:
   oldLen = target.dataEnd - target.dataStart
   newLenBytes = encodeVarint(newData.length)
   delta = newData.length - oldLen + (newLenBytes.length - target.lenSize)
   patch: replace [target.lenOffset .. target.dataEnd] with [newLenBytes + newData]

2. Find ancestor fields (all LenFields containing the target):
   ancestors = fields.filter(f =>
     f.dataStart <= target.lenOffset AND
     f.dataEnd >= target.dataEnd AND
     f !== target
   ).sort(by size ascending)  // innermost first

3. For each ancestor (innermost to outermost):
   oldAncestorLen = ancestor.dataEnd - ancestor.dataStart
   newAncestorLen = oldAncestorLen + delta
   newAncestorLenBytes = encodeVarint(newAncestorLen)
   ancestorDelta = newAncestorLenBytes.length - ancestor.lenSize
   patch: replace [ancestor.lenOffset .. ancestor.dataStart] with newAncestorLenBytes
   delta += ancestorDelta  // propagate varint size changes

4. Apply all patches sorted by descending offset (to avoid offset invalidation)
```

### Varint Encoding

```typescript
function encodeVarint(value: number): Uint8Array {
  const bytes = []
  let v = value >>> 0
  while (v >= 0x80) {
    bytes.push((v & 0x7f) | 0x80)
    v >>>= 7
  }
  bytes.push(v)
  return Uint8Array.from(bytes)
}
```

**Source:** `src/injector/binary.ts:19-28`

### Varint Decoding

```typescript
function readVarint(buf, offset): { value, next } | null {
  let val = 0, shift = 0, cur = offset
  while (cur < buf.length && shift < 64) {
    const byte = buf[cur++]
    val |= (byte & 0x7f) << shift
    if ((byte & 0x80) === 0) return { value: val, next: cur }
    shift += 7
  }
  return null
}
```

**Source:** `src/injector/binary.ts:3-17`

---

## 9. Signal Node Patching

**Source:** `src/injector/signal_nodes.ts`

### Placeholder IDs (컴파일 타임 플레이스홀더)

| Placeholder ID | Kind    | Description                |
|----------------|---------|----------------------------|
| 300000         | send    | Send signal node           |
| 300001         | monitor | Monitor (receive) signal node |

These are assigned at compile time and must be resolved to real node IDs at injection time.

### Resolution Process

1. **Check if patching needed** (signal_nodes.ts:295-299): scan graph nodes for any with placeholder IDs. Skip entirely if none found.

2. **Build signal map from GIL** (signal_nodes.ts:208-252):
   - Scan all LenField entries (max 4096 bytes per container)
   - For each field with a signal definition (field 107):
     - Extract signal name: `field 107` -> `field 101 or 102` -> `field 1` (UTF-8 string)
     - Extract node ID: `field 4` -> `field 1` (generic) or `field 2` (concrete) -> parse NodeGraphIdInfo
     - Determine signal kind:
       - `type == 20002` (SIGNAL_NODE_TYPE_SKILLS) -> `sendServer`
       - `outputs >= 3` (count of field 103 messages) -> `monitor`
       - else -> `send`
   - Build map: `signalName -> { send?, monitor?, sendServer? } -> NodeGraphIdInfo`

3. **Extract signal name from graph node** (signal_nodes.ts:178-197):
   - Priority: search ClientExecNode pins (kind=5) first
   - Extract from `pin.value.bString.val`
   - Fallback: search all pins (to handle edge cases)

4. **Apply resolved IDs** (signal_nodes.ts:304-322):
   - For each graph node with a placeholder ID:
     - Look up signal name -> signal entry -> kind-specific NodeGraphIdInfo
     - Apply all fields (class, type, kind, nodeId) to both `genericId` and `concreteId`

### NodeGraphIdInfo Structure

```typescript
type NodeGraphIdInfo = {
  class?: number   // NodeGraph.Id.class
  type?: number    // NodeGraph.Id.type
  kind?: number    // NodeGraph.Id.kind
  nodeId?: number  // NodeGraph.Id.nodeId (= NodeGraph.Id.id, field 5)
}
```

---

## 10. Folder Index Parsing

**Source:** `src/injector/folder.ts`

### FolderEntry

```typescript
type FolderEntry = { typeValue?: number; id?: number }
```

Parsed from varint-only protobuf messages: field 1 = typeValue, field 2 = id.

### collectFolderIndexes

Collects all folder metadata from path prefix `6.1`:
- `6.1` (depth=2): folder entry containers
- `6.1.3` (depth=3, p2=3): folder content (name + entries)
- `6.1.2.4` (depth=4, p2=2, p3=4): meta lists

### findFolderEntryField

Searches for a target NodeGraph ID in:
1. Content entries at path `6.1.3.5` (depth=4, p0=6, p1=1, p2=3, p3=5)
2. Meta entries at path `6.1.2.4.5` (depth=5, p0=6, p1=1, p2=2, p3=4, p4=5)

When multiple matches exist, content matches are preferred.

---

## 11. Hardcoded Values and Assumptions (하드코딩된 값 및 가정)

### Magic Numbers

| Value    | Purpose   | Locations |
|----------|-----------|-----------|
| `0x0326` | head_tag  | `index.ts:89`, `reader.ts:295`, `binary.ts:201` (buildFile) |
| `0x0679` | tail_tag  | `index.ts:89`, `reader.ts:295`, `binary.ts:209` (buildFile) |
| 20       | header size (bytes) | `index.ts:93`, `node_graph.ts:71`, `binary.ts:201-203` |
| 4        | tail size (bytes)   | `index.ts:93`, `node_graph.ts:71` |

### Protobuf Paths

| Path     | Meaning              | Hardcoded In |
|----------|----------------------|--------------|
| 10.1.1   | NodeGraph blobs      | `binary.ts:97-104`, `node_graph.ts:10-11`, `index.ts:112-116` |
| 6.1      | Folder entries       | `folder.ts:157` |
| 6.1.3    | Folder content       | `folder.ts:160-161` |
| 6.1.3.5  | Content entry items  | `folder.ts:234` |
| 6.1.2.4  | Meta lists           | `folder.ts:162` |
| 6.1.2.4.5| Meta entry items     | `folder.ts:236` |
| field 107 | Signal definitions  | `signal_nodes.ts:221` |
| field 101/102 | Signal name containers | `signal_nodes.ts:171` |
| field 4  | CompositeDef ID      | `signal_nodes.ts:226` |

### NodeGraph.Id Fields

| Field Number | Meaning  | Source |
|-------------|----------|--------|
| 1           | class    | `signal_nodes.ts:135` |
| 2           | type     | `node_graph.ts:41`, `signal_nodes.ts:136` |
| 3           | kind     | `signal_nodes.ts:137` |
| 5           | id/nodeId | `node_graph.ts:43`, `signal_nodes.ts:139` |

### Graph Type Values

| typeValue (folder) | Graph Type ID | Graph Name |
|--------------------|---------------|------------|
| 800                | 20000         | entity     |
| 2300               | 20003         | status     |
| 2400               | 20004         | class      |
| 4300               | 20005         | item       |

### Signal Constants

| Value  | Meaning                  |
|--------|--------------------------|
| 300000 | Send signal placeholder  |
| 300001 | Monitor signal placeholder |
| 20002  | SIGNAL_NODE_TYPE_SKILLS (sendServer) |
| 5      | ClientExecNode pin kind  |
| 4096   | Max container bytes for signal scan |

### Scanner Limits

| Limit | Value | Location |
|-------|-------|----------|
| parseMessage max depth | 6 | `binary.ts:105` |
| readVarint max shift | 64 | `binary.ts:10` |

---

## 12. Proto Schema

The protobuf schema is loaded from `gia.proto` using `protobufjs` and cached by absolute path.

**Source:** `src/injector/proto.ts`

Two message types are used:
- `Root`: for decoding the GIA file envelope (contains the NodeGraph inside `graph.graph.inner.graph`)
- `NodeGraph`: for decoding/encoding individual NodeGraph blobs from GIL (at path 10.1.1)

The GIL file as a whole does NOT use the `Root` message. Only the NodeGraph blobs inside it share the `NodeGraph` schema with GIA.

---

## 13. Source File Map

| File | Role | Key Functions |
|------|------|---------------|
| `src/injector/index.ts` | Main entry; orchestrates injection | `createInjector()`, `injectBytes()`, `injectFile()` |
| `src/injector/binary.ts` | Raw protobuf scanner and binary patcher | `parseMessage()`, `applyReplacement()`, `readVarint()`, `encodeVarint()`, `buildFile()` |
| `src/injector/node_graph.ts` | NodeGraph locate/decode/modify | `loadGiaGraph()`, `findNodeGraphTargets()`, `tryReadNodeGraphIdAndType()`, `getGraphId()`, `setGraphId()` |
| `src/injector/signal_nodes.ts` | Signal node ID resolution | `patchSignalNodeIds()`, `buildSignalNodeIdMapFromFields()`, `extractSignalNameFromNode()` |
| `src/injector/folder.ts` | Folder metadata parsing | `collectFolderIndexes()`, `findFolderEntryField()`, `resolveGraphTypeForTypeValue()` |
| `src/injector/types.ts` | Type definitions | `LenField`, `Patch`, `FolderEntry`, `InjectGilInput`, `InjectGilResult` |
| `src/injector/proto.ts` | Protobuf schema loader | `loadGiaProto()` |
| `src/injector/reader.ts` | Read-only GIL inspector | `readGilNodeGraphs()`, `readGilNodeGraph()` |
