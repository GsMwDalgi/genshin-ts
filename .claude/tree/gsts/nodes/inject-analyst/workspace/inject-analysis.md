# Injector Logic Analysis

## 1. High-Level Pipeline

```
TypeScript -> GS IR -> IR JSON -> GIA binary -> Inject into GIL
```

The injector handles the final step: replacing a target NodeGraph inside a `.gil` (map save) file with a new NodeGraph extracted from a compiled `.gia` file.

Entry point: `src/injector/index.ts` -> `createInjector()` -> `injectBytes()`

---

## 2. GIL File Read/Parse Flow

### 2.1 Binary Envelope (20B header + 4B tail)

Both GIL and GIA share the same envelope format:

| Offset | Size | Field            | GIL Value        | GIA Value        |
|--------|------|------------------|------------------|------------------|
| 0x00   | 4B   | `left_size`      | `file_size - 4`  | `file_size - 4`  |
| 0x04   | 4B   | `schema_version` | `1`              | `1`              |
| 0x08   | 4B   | `head_tag`       | `0x0326`         | `0x0326`         |
| 0x0C   | 4B   | `file_type`      | `2`              | `3`              |
| 0x10   | 4B   | `proto_size`     | `file_size - 24` | `file_size - 24` |
| EOF-4  | 4B   | `tail_tag`       | `0x0679`         | `0x0679`         |

All values are **big-endian** (`readUint32BE` in `binary.ts:130`).

**Validation** (index.ts:89): Only `headTag` (0x0326) and `tailTag` (0x0679) are checked. `file_type` is NOT validated -- the injector accepts any file_type as long as tags match. `schema_version` is preserved but not asserted.

### 2.2 Payload Extraction

```typescript
const payload = input.gilBytes.slice(20, -4)  // index.ts:93
```

The protobuf payload sits between the 20-byte header and the 4-byte tail.

### 2.3 Recursive Protobuf Wire-Format Scan (`parseMessage`)

**File:** `binary.ts:34-128`

Instead of using protobufjs to decode the entire GIL (which would be very slow), the injector performs a raw protobuf wire-format scan:

- Recursively walks the binary protobuf structure
- Tracks field ancestry using `p0` through `p5` (path components at each depth level)
- Records every length-delimited field (wire type 2) as a `LenField` containing byte offsets
- Stops recursion at `depth >= 6` (hardcoded limit)
- Collects NodeGraph blob fields (depth=3, p0=10, p1=1, p2=1) into a separate `nodeGraphBlobFields` array for fast access

**Key data structure -- LenField** (`types.ts:1-14`):
```
{ field, depth, p0-p5, lenOffset, lenSize, dataStart, dataEnd }
```
This stores the exact byte position of each length-delimited field, including the varint length prefix location, enabling surgical binary patching later.

### 2.4 GIL Protobuf Structure (Relevant Paths)

| Path    | Content                       |
|---------|-------------------------------|
| 6.1     | FolderEntry (folder metadata) |
| 6.1.3   | Folder content (name + entry list) |
| 6.1.3.5 | Entry items (typeValue + id)  |
| 10.1.1  | NodeGraph blobs (self-contained protobuf messages) |

---

## 3. GIA Data Loading (`loadGiaGraph`)

**File:** `node_graph.ts:110-131`

1. Strip envelope: `unwrapGia()` -> `bytes.slice(20, -4)` (node_graph.ts:70-72)
2. Decode with protobufjs: `rootMessage.decode(payload)` -- uses the `Root` message from `gia.proto`
3. Navigate to the NodeGraph: `rootMsg.graph.graph.inner.graph` (4-level deep nesting)
4. If `targetId` is provided and differs from the graph's ID, overwrite it via `setGraphId()`

**Note:** The GIA `Root` message has a **different schema** from the GIL root. The GIA wraps a single NodeGraph in a `Root.graph.graph.inner.graph` path. GIL uses a completely different top-level structure.

---

## 4. GIA Injection Into GIL

### 4.1 Signal Node ID Patching (`patchSignalNodeIds`)

**File:** `signal_nodes.ts:282-323`

**Placeholders** (compile-time):
- `300000` = send signal node
- `300001` = monitor signal node

**Resolution process:**
1. Check if any graph nodes have placeholder IDs (quick-skip if none)
2. Build a signal parse context from the GIL payload (re-parse if not already provided)
3. Scan all LenField entries for CompositeDef containers (field 107 = signal definition)
4. For each CompositeDef with a signal definition:
   - Extract signal name from field 107 -> field 101/102 -> field 1 (name string)
   - Extract node ID from field 4 (CompositeDefId) -> field 1 (generic) or field 2 (concrete)
   - Determine kind: `type=20002` -> sendServer; `outputs>=3` -> monitor; else -> send
5. For each graph node with a placeholder ID:
   - Extract signal name from ClientExecNode pins (kind=5, bString value)
   - Priority: search ClientExec pins first, then fallback to all pins (to avoid matching InParam strings)
   - Look up resolved signal entry from the built map
   - Apply full NodeGraphIdInfo (class, type, kind, nodeId) to both `genericId` and `concreteId`

### 4.2 Target NodeGraph Location (`findNodeGraphTargets`)

**File:** `node_graph.ts:133-156`

1. Iterate over fields (preferably the pre-collected `nodeGraphBlobFields` at path 10.1.1)
2. Fast ID check via `tryReadNodeGraphIdAndType()` (node_graph.ts:13-68):
   - Reads first key varint; must be `10` (field 1, wire type 2 = embedded message)
   - Parses the Id sub-message for `type` (field 2) and `id` (field 5)
   - Breaks early after finding `id` (field 5) -- does NOT read all fields
3. Only if fast ID matches the `targetId`, perform full protobufjs decode
4. Must find exactly 1 match (0 = error, >1 = abort)

### 4.3 Validation

**Path check** (index.ts:108-117): For target IDs >= 1,000,000,000, verify the matched field is at path 10.1.1 (depth=3, p0=10, p1=1, p2=1).

**Graph type resolution** (folder.ts + index.ts:119-153):
1. `collectFolderIndexes()`: parse folder metadata from path 6.1
2. `findFolderEntryField()`: locate the target ID in folder entries (path 6.1.3.5 or 6.1.2.4.5)
3. `resolveGraphTypeForTypeValue()`: map typeValue -> graph type:
   - First: scan all graphs with matching typeValue and check their actual types
   - Fallback: use `DEFAULT_VALUE_TO_GRAPH_TYPE` static map

| typeValue (GIL) | Graph Type (Code) | Name   |
|-----------------|-------------------|--------|
| 800             | 20000             | entity |
| 2300            | 20003             | status |
| 2400            | 20004             | class  |
| 4300            | 20005             | item   |

4. Compare existing graph type and incoming graph type against the resolved type (warns on mismatch but does NOT abort)

**Non-empty safety check** (index.ts:155-165):
- If target NodeGraph has existing nodes AND name does NOT start with `_GSTS`, abort
- Bypassable with `skipNonEmptyCheck: true`

### 4.4 Replacement

1. Copy name from new graph to target (index.ts:168-170)
2. Set graph ID and type on new graph (index.ts:172-173)
3. Verify new graph with protobufjs `verify()` (index.ts:175-177)
4. Encode new graph to bytes with protobufjs `encode().finish()` (index.ts:181)
5. Apply binary patch via `applyReplacement()` (index.ts:183)

---

## 5. Binary Patching Details (`applyReplacement`)

**File:** `binary.ts:159-195`

This is the most delicate operation -- replacing a protobuf blob while maintaining valid container structure.

### 5.1 Process

1. **Replace target blob:**
   - Old: `[varint_len][old_data]`
   - New: `[varint_len'][new_data]`
   - Initial delta = `new_data_len - old_data_len + (new_varint_size - old_varint_size)`

2. **Update ancestor lengths** (`findAncestorFields`, binary.ts:153-157):
   - Find all LenField entries that spatially contain the target (dataStart <= target.lenOffset && dataEnd >= target.dataEnd)
   - Sort by size ascending (innermost ancestor first)
   - For each ancestor:
     - Compute new length = old_length + accumulated_delta
     - Re-encode length as varint
     - If varint size changed (e.g., length grew past 128 boundary), add to delta
   - Create a patch for each ancestor's length prefix

3. **Apply patches** (`applyPatches`, binary.ts:136-151):
   - Sort patches by descending start offset (apply from end to avoid offset invalidation)
   - Build output buffer by slicing/splicing the original payload
   - Uses `Buffer.concat` with pre-built parts array for efficiency

### 5.2 Critical Assumption

The ancestor length update assumes that **varint length fields never shrink** in a way that causes cascading size issues. In practice, replacing a NodeGraph with a larger one only increases varint sizes, and replacing with a smaller one only decreases data but varint encoding of smaller values uses fewer or equal bytes.

---

## 6. GIL File Write/Output Flow (`buildFile`)

**File:** `binary.ts:197-211`

1. Allocate buffer: `payload.length + 24` bytes
2. Write header (big-endian):
   - `left_size` = `payload.length + 20`
   - `schema_version` = preserved from original
   - `head_tag` = preserved (0x0326)
   - `file_type` = preserved (2 for GIL)
   - `proto_size` = `payload.length`
3. Copy payload at offset 20
4. Write `tail_tag` at end (preserved, 0x0679)

File-level injection (`injectFile`, index.ts:200-215) wraps this:
1. Read GIL and GIA from disk
2. Call `injectBytes()`
3. Write result to `outPath` (defaults to overwriting original GIL)

---

## 7. Key Data Structures and Relationships

```
InjectGilInput
  ├── gilBytes: Uint8Array     (raw GIL file)
  ├── giaBytes: Uint8Array     (compiled GIA file)
  ├── targetId?: number        (NodeGraph ID to replace)
  └── skipNonEmptyCheck?: bool

LenField (core tracking structure)
  ├── field, depth             (protobuf field number and nesting depth)
  ├── p0..p5                   (ancestor field numbers for path matching)
  ├── lenOffset, lenSize       (byte position of varint length prefix)
  └── dataStart, dataEnd       (byte range of actual data)

NodeGraphIdInfo (signal node identity)
  ├── class, type, kind        (NodeGraph.Id fields)
  └── nodeId                   (unique node identifier)

Patch (binary edit unit)
  ├── start, end               (byte range to replace)
  └── replacement: Uint8Array  (new bytes)
```

---

## 8. Docs vs Source Gaps

### 8.1 Accurate in Docs

- Binary envelope format (both GIL and GIA) -- matches code exactly
- NodeGraph blob path (10.1.1) -- matches parseMessage collector
- Signal placeholder IDs (300000=send, 300001=monitor) -- matches `SIGNAL_NODE_ID_PLACEHOLDERS`
- Folder typeValue mapping -- matches `DEFAULT_GRAPH_TYPE_VALUES`
- Binary patching algorithm -- matches `applyReplacement` implementation
- GIA unwrap (20B header + 4B tail strip) -- matches `unwrapGia()`

### 8.2 Gaps / Undocumented Behavior

1. **file_type not validated**: Docs mention `file_type=2` for GIL and `file_type=3` for GIA, but the injector only checks head_tag and tail_tag. A file with wrong file_type but correct tags would pass validation.

2. **schema_version not validated**: Code reads and preserves it but never asserts `schema_version === 1`. If the game changes schema_version, the injector would silently proceed.

3. **Folder entry dual-path resolution**: `findFolderEntryField` searches BOTH content entries (6.1.3.5) and meta entries (6.1.2.4.5). When multiple matches exist, it prefers content matches. This dual-path logic is not fully described in the protocol docs.

4. **sendServer signal kind**: The `SIGNAL_NODE_TYPE_SKILLS = 20002` constant determines if a signal is `sendServer` type. This is undocumented in the protocol docs.

5. **parseMessage depth limit**: Recursion stops at depth 6 (`depth < 6` check, binary.ts:105). Deeply nested protobuf structures beyond this depth are not indexed. Not mentioned in docs.

6. **Graph type resolution fallback chain**: `resolveGraphTypeForTypeValue` first tries to infer type from existing graphs with the same typeValue, only falling back to the static map when no single consistent type is found. The docs only mention the static map.

7. **reader.ts not in injection pipeline docs**: The `reader.ts` module (readGilNodeGraphs, readGilNodeGraph) is documented in architecture but not in the injection-flow protocol doc, though it uses the same parsing infrastructure.

8. **proto.ts caching**: The protobuf definition is loaded from `gia.proto` and cached by absolute path. This means the proto schema is fixed at compile time and cannot adapt to game version changes.

---

## 9. Hardcoded Assumptions / Format Change Vulnerability Points

### 9.1 Magic Numbers and Tags

| Constant | Location | Risk |
|----------|----------|------|
| `0x0326` (head_tag) | index.ts:89, reader.ts:295 | **HIGH** -- validation gate; if game changes this, all injection fails |
| `0x0679` (tail_tag) | index.ts:89, reader.ts:295 | **HIGH** -- same as above |
| Header size = 20 bytes | index.ts:93, binary.ts:201, node_graph.ts:71 | **HIGH** -- hardcoded in 3+ locations; header format change breaks everything |
| Tail size = 4 bytes | index.ts:93, node_graph.ts:71 | **HIGH** -- same as above |
| `file_type=3` for GIA | Only in docs/decode.ts (not in injector) | LOW -- injector doesn't check this |

### 9.2 Protobuf Path Assumptions

| Path | Used For | Location | Risk |
|------|----------|----------|------|
| 10.1.1 (depth=3) | NodeGraph blob location | binary.ts:97-104, node_graph.ts:10-11, index.ts:112-116 | **CRITICAL** -- if GIL restructures where NodeGraphs are stored, all scanning fails |
| 6.1 | Folder entries | folder.ts:151-157 | **HIGH** -- folder metadata path |
| 6.1.3 | Folder content | folder.ts:160-161 | **HIGH** -- content sub-path |
| 6.1.3.5 | Content entry items | folder.ts:234 | **HIGH** -- entry item path |
| 6.1.2.4.5 | Meta entry items | folder.ts:236 | **HIGH** -- meta entry path |
| field 107 | Signal definitions in CompositeDef | signal_nodes.ts:221 | **HIGH** -- signal system path |
| field 101/102 | Signal name containers | signal_nodes.ts:171 | **HIGH** -- signal name extraction |
| field 4 | CompositeDef ID | signal_nodes.ts:226 | **HIGH** -- node ID extraction |

### 9.3 Protobuf Schema (gia.proto)

| Assumption | Location | Risk |
|------------|----------|------|
| `Root.graph.graph.inner.graph` nesting | node_graph.ts:118-121 | **CRITICAL** -- GIA Root structure; if proto schema changes, GIA loading fails |
| `NodeGraph.Id.type` = field 2 | node_graph.ts:41 | **HIGH** -- NodeGraph identity |
| `NodeGraph.Id.id` = field 5 | node_graph.ts:43 | **HIGH** -- NodeGraph identity |
| First key varint = 10 (field 1, LEN) | node_graph.ts:17 | **MEDIUM** -- fast scanner signature; would fail if NodeGraph field ordering changes |

### 9.4 NodeGraph ID Conventions

| Assumption | Location | Risk |
|------------|----------|------|
| Target IDs >= 1,000,000,000 | index.ts:109 | **MEDIUM** -- only triggers extra path validation |
| Placeholder 300000 = send | signal_nodes.ts:29 | **HIGH** -- compile-time convention |
| Placeholder 300001 = monitor | signal_nodes.ts:30 | **HIGH** -- compile-time convention |
| `type=20002` = sendServer (skills) | signal_nodes.ts:26 | **HIGH** -- signal kind determination |

### 9.5 Graph Type Values

| Assumption | Location | Risk |
|------------|----------|------|
| 800 -> entity (20000) | folder.ts:5 | **HIGH** -- typeValue mapping |
| 2300 -> status (20003) | folder.ts:6 | **HIGH** -- typeValue mapping |
| 2400 -> class (20004) | folder.ts:7 | **HIGH** -- typeValue mapping |
| 4300 -> item (20005) | folder.ts:8 | **HIGH** -- typeValue mapping |

### 9.6 Name Convention

| Assumption | Location | Risk |
|------------|----------|------|
| `_GSTS` prefix = safe to overwrite | index.ts:161 | **LOW** -- internal convention |

---

## 10. Summary of Format Change Impact

**If the binary envelope changes** (header size, magic bytes, field order):
- Immediate failure at validation (head_tag/tail_tag check)
- All byte offset calculations become wrong
- Fix: update constants in `binary.ts` and `node_graph.ts`

**If the protobuf schema changes** (field numbers, nesting):
- NodeGraph path (10.1.1) scan returns no results
- Folder index parsing returns empty/wrong data
- Signal node resolution fails
- Fix: update path constants and field numbers throughout

**If the gia.proto changes** (Root message structure):
- GIA loading fails at `rootMsg.graph.graph.inner.graph` navigation
- NodeGraph encode/decode fails
- Fix: update gia.proto and navigation path

**If graph type values change**:
- Type resolution returns wrong types or errors
- Graph type validation fails spuriously
- Fix: update `DEFAULT_GRAPH_TYPE_VALUES` map

**If signal definitions change** (field numbers, structure):
- Signal patching silently produces wrong results or errors
- Compiled graphs with send/monitor signals become non-functional in-game
- Fix: update field numbers in `signal_nodes.ts`
