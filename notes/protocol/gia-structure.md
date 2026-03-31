# GIA File Structure

## Overview

GIA (Genshin Impact Action/Assembly) files are the node-graph binary format used by Genshin Impact's Beyond Mode (map editor). Each `.gia` file contains a single protobuf-encoded node graph wrapped in a fixed binary envelope.

## Binary Envelope

```
Offset  Size   Field                Value / Description
------  ----   -----                -------------------
0x00    4B     left_size            file_size - 4 (big-endian uint32)
0x04    4B     schema_version       1 (always)
0x08    4B     head_tag             0x0326 (magic number)
0x0C    4B     file_type            3 (always for GIA)
0x10    4B     proto_size           file_size - 24 (protobuf payload length)
0x14    ...    protobuf payload     Root message (variable length)
EOF-4   4B     tail_tag             0x0679 (magic number)
```

Total header = 20 bytes. Total footer = 4 bytes. Proto payload = file_size - 24 bytes.

All multi-byte integers are **big-endian** (network byte order).

### Assertions (from decode.ts)
- `bytes.byteLength - 4 === left_size`
- `schema_version === 1`
- `head_tag === 0x0326`
- `tail_tag === 0x0679`
- `file_type === 3`
- `proto_size === bytes.byteLength - 24`

### Wrap (encode) formula
```
header = [proto_size + 20, 1, 0x0326, 3, proto_size]
tail   = [0x0679]
total  = header(20B) + proto(NB) + tail(4B)
```

## Protobuf Schema (Root message)

Source: `src/thirdparty/.../protobuf/gia.proto` (version 1.5.0)

```protobuf
message Root {
  GraphUnit graph = 1;                // main graph
  repeated GraphUnit accessories = 2; // data structures, I/O
  string filePath = 3;                // "{UID}-{TIMESTAMP}-{LEVEL_ID}-\{FILE_NAME}.gia"
  optional uint32 modeFlag = 4;       // 1 = classic mode; omitted for Beyond Mode
  string gameVersion = 5;             // e.g. "6.3.0"
}
```

### filePath Format
`{uid}-{timestamp_seconds}-{file_id}-\{graph_name_normalized}`

- `uid`: 9-digit random number (prefix "201")
- `timestamp`: Unix seconds at generation time
- `file_id`: usually `graph_id + counter`
- Graph name is lowercased, non-alphanumeric chars replaced with `_`

## GraphUnit (Top-level Container)

Each GraphUnit wraps one node graph. The `which` field determines the graph category:

| Which Value | Name                    | Description                          |
|-------------|-------------------------|--------------------------------------|
| 0           | Unknown                 |                                      |
| 9           | EntityNode              | Standard entity graph                |
| 10          | BooleanFilter           | Boolean filter graph                 |
| 11          | Skills                  | Skills graph                         |
| 12          | CompositeGraph          | Composite (user-defined subgraph)    |
| 22          | StatusNode              | Status node graph                    |
| 23          | ClassNode               | Class node graph                     |
| 29          | StructureDefinition     | Structure type definition            |
| 46          | ItemNode                | Item node graph                      |
| 47          | IntegerFilter           | Integer filter graph                 |
| 51          | CreationStatusDecision  |                                      |
| 52          | CreationSkill           |                                      |
| 53          | CreationStatus          |                                      |

### GraphUnit.Id

```protobuf
message Id {
  enum Class { Unknown1=0; Node=1; Basic=5; AffiliatedNode=23; }
  enum Type  { ServerGraph=0; ClientGraph=3; StructureDefinition=15; }
  Class class = 2;
  Type type = 3;
  int32 id = 4;    // Node Graph Id
}
```

### GraphUnit.graphType (oneof)

- `graph` (field 13) -> NodeGraphWrapper -> wraps NodeGraph (most common)
- `compositeDef` (field 14) -> CompositeDefWrapper -> wraps CompositeDef (composite graphs)
- `structureDef` (field 22) -> StructureDefWrapper -> wraps structure type definitions

## NodeGraph (Core Graph Object)

```protobuf
message NodeGraph {
  Id id = 1;
  string name = 2;
  repeated GraphNode nodes = 3;
  repeated CompositePin compositePins = 4;
  repeated Comments comments = 5;
  repeated GraphVariable graphValues = 6;
  repeated GraphAffiliation affiliations = 7;
  optional int32 entrySlotIndex = 100;
  optional float evaluationInterval = 101;
}
```

### NodeGraph.Id

```protobuf
message Id {
  enum Class { Unknown1=0; UserDefined=10000; SystemDefined=10001; }
  enum Type  { Unknown2=0; BasicNode=20000; BooleanFilter=20001; Skills=20002;
               StatusNode=20003; ClassNode=20004; ItemNode=20005;
               IntegerFilter=20006; CreationStatusDecision=20007;
               CreationSkill=20008; CreationStatus=20009; }
  enum Kind  { Unknown3=0; NodeGraph=21001; CompositeGraph=21002;
               SysCall=22000; SysGraph=22001; }
  Class class = 1;
  Type type = 2;
  Kind kind = 3;
  int64 id = 5;    // Unique graph ID (typically >= 1,000,000,000)
}
```

## GraphNode (Individual Node)

```protobuf
message GraphNode {
  int32 nodeIndex = 1;                  // unique index within graph
  NodeProperty genericId = 2;           // node template ID
  optional NodeProperty concreteId = 3; // specific node ID (null for generic)
  repeated NodePin pins = 4;
  float x = 5;                          // x position (unit * 300 + noise)
  float y = 6;                          // y position (unit * 200 + noise)
  optional Comments comments = 7;
  optional NodePin.Index contextDeclaration = 8;
  optional int32 signalVersion = 9;
  repeated GraphAffiliation.Info usingStruct = 10;
  optional StatusNodeExtension statusNodeExtension = 11;
}
```

### Position encoding
Positions are stored as `logical_pos * scale + random_noise(0~10)`:
- x: scale = 300
- y: scale = 200

## NodePin (Connection Points)

```protobuf
message NodePin {
  Index i1 = 1;                       // primary index
  Index i2 = 2;                       // duplicate of i1
  VarBase value = 3;                  // pin value (input params have data)
  int32 type = 4;                     // VarType (server) or ClientVarType (client)
  repeated NodeConnection connects = 5;
  optional Index clientExecNode = 6;  // client binding metadata
  optional int32 compositePinIndex = 7;
}
```

### Pin Index Kind

| Kind | Name           | Description                         |
|------|----------------|-------------------------------------|
| 1    | InFlow         | Execution flow input                |
| 2    | OutFlow        | Execution flow output               |
| 3    | InParam        | Data parameter input                |
| 4    | OutParam       | Data parameter output               |
| 5    | ClientExecNode | Client exec/binding metadata        |
| 6    | ClientSignal   | Client signal/topic metadata        |

### NodeConnection

```protobuf
message NodeConnection {
  int32 id = 1;            // target node's nodeIndex
  NodePin.Index connect = 2;
  NodePin.Index connect2 = 3;
}
```

- For **OutFlow** pins: connections point to target nodes (flow targets)
- For **InParam** pins: connections reference source nodes (data sources)

## Variable Type System

### VarType (Server-side, 28 types)

| ID | Type              | ID | Type              |
|----|-------------------|----|-------------------|
| 0  | UnknownVar        | 14 | EnumItem          |
| 1  | Entity            | 15 | VectorList        |
| 2  | GUID              | 16 | LocalVariable     |
| 3  | Integer           | 17 | Faction           |
| 4  | Boolean           | 20 | Configuration     |
| 5  | Float             | 21 | Prefab            |
| 6  | String            | 22 | ConfigurationList |
| 7  | GUIDList          | 23 | PrefabList        |
| 8  | IntegerList       | 24 | FactionList       |
| 9  | BooleanList       | 25 | Struct            |
| 10 | FloatList         | 26 | StructList        |
| 11 | StringList        | 27 | Dictionary        |
| 12 | Vector            | 28 | VariableSnapshot  |
| 13 | EntityList        |    |                   |

### VarBase.Class (Value storage discriminant)

| ID    | Class              | Oneof field index |
|-------|--------------------|-------------------|
| 1     | IdBase             | 101               |
| 2     | IntBase            | 102               |
| 4     | FloatBase          | 104               |
| 5     | StringBase         | 105               |
| 6     | EnumBase           | 106               |
| 7     | VectorBase         | 107               |
| 10000 | ConcreteBase       | 110 (reflective)  |
| 10001 | StructBase         | 108               |
| 10002 | ArrayBase          | 109               |
| 10003 | MapBase            | 112               |
| 10006 | ClientContainerMeta| -                 |
| 10007 | MapPair            | 111               |

Rule: `oneof_field_id = class_id + 100` (except Struct/Array/Map which use fixed offsets)

## Internal Type System (NodeType)

The codebase uses a compact internal type representation (see `nodes.ts`):

```typescript
type NodeType =
  | { t: 'b'; b: BasicTypes }        // basic: Int, Flt, Bol, Str, Vec, Gid, Ety, Pfb, Fct, Cfg
  | { t: 'e'; e: EnumId }            // enum: E<enumId>
  | { t: 'l'; i: NodeType }          // list: L<itemType>
  | { t: 's'; f: [string, NodeType][] } // struct: S<field:Type,...>
  | { t: 'd'; k: NodeType; v: NodeType } // dict: D<keyType,valueType>
  | { t: 'r'; r: string }            // reflect: R<name>
```

String representation examples:
- `Int`, `Flt`, `Bol`, `Str`, `Vec`
- `E<42>` (enum with id 42)
- `L<Int>` (integer list)
- `S<hp:Int,name:Str>` (struct)
- `D<Str,Int>` (dictionary)
