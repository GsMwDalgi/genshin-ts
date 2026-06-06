# Graph Encoding

## Graph Builder API (gia_gen) [CONFIRMED]

The `Graph` class in `gia_gen/graph.ts` provides a high-level API for constructing GIA-compatible node graphs.

### Graph Construction

```typescript
const graph = new Graph('server')    // mode: server|status|class|item|composite|bool|int|skill
const node1 = graph.add_node(200)    // add node by concrete ID (NODE_ID enum)
const node2 = graph.add_node(201)
graph.connect(node1, node2, 0, 0)    // data flow: from output pin 0 to input pin 0
graph.flow(node1, node2, 0, 0)       // execution flow: outflow 0 to inflow 0
const root = graph.encode()          // produces Root protobuf object
```

### Graph Mode Mapping [CONFIRMED]

| Mode      | GraphUnit.Which    | NodeGraph.Id.Type     |
|-----------|--------------------|-----------------------|
| server    | EntityNode (9)     | BasicNode (20000)     |
| status    | StatusNode (22)    | StatusNode (20003)    |
| class     | ClassNode (23)     | ClassNode (20004)     |
| item      | ItemNode (46)      | ItemNode (20005)      |
| composite | --                 | --                    |
| bool      | BooleanFilter (10) | BooleanFilter (20001) |
| int       | IntegerFilter (47) | IntegerFilter (20006) |
| skill     | Skills (11)        | Skills (20002)        |

### Graph Name Convention [CONFIRMED]

Generated graphs are prefixed with `_GSTS` (e.g. `_GSTS_my_graph`). This prefix is used during injection to verify the target graph is already a genshin-ts graph.

### Parallel Compilation [CONFIRMED]

`ir_to_gia_pipeline.ts` supports parallel GIA generation by spawning child processes, each running `ir_to_gia_transform/runner.ts`. Uses `os.cpus().length - 1` workers.

## Graph Variables [CONFIRMED]

```protobuf
GraphVariable {
  name = "varName"
  type = <VarType>
  values = <VarBase>      // initial value
  exposed = true/false    // visible outside graph?
  structId = 0            // non-zero only for struct vars
  keyType = <VarType>     // for dict types
  valueType = <VarType>   // for dict types
}
```

## Comments [CONFIRMED]

Two types:
- **Graph-level comments**: Have x, y coordinates. Floating annotations on the canvas.
- **Node-attached comments**: Only have content string. Attached to a specific node.

```protobuf
Comments {
  content = "text"
  x? = <float>   // only for graph-level
  y? = <float>   // only for graph-level
}
```

## Composite Graphs (CompositeDef) [CONFIRMED]

Composite graphs are user-defined reusable subgraphs with explicit I/O interfaces:

```protobuf
CompositeDef {
  id = { genericId, concreteId, graphId }
  inflows[]   = ControlFlow entries   // execution inputs
  outflows[]  = ControlFlow entries   // execution outputs
  inputs[]    = ParameterFlow entries // data inputs
  outputs[]   = ParameterFlow entries // data outputs
  type = { kind: Composite|Assembly|Split|Modify }
  name = "..."
  description = "..."
}
```

### CompositeDef.Type.Kind [CONFIRMED]

| Kind      | Value | Description                    |
|-----------|-------|--------------------------------|
| Composite | 1000  | Standard composite subgraph    |
| Assembly  | 1003  | Structure assembly node        |
| Split     | 1004  | Structure split node           |
| Modify    | 1005  | Structure modify node          |

## Structure Definitions (StructureDefWrapper) [CONFIRMED]

```protobuf
StructureDefWrapper {
  def = {
    genericField = { id, vars[], structName, classBase=1, index }
    connectField = <same as genericField>
    index = <1-based structure index>
    itemCount = <number of fields>
  }
}
```

Each field in a structure:
```protobuf
VarDef {
  id = { type: VarType, sub: SubType }
  def = { type: VarType, sub: SubType, intVal?/boolVal?/strVal? }
  name = "fieldName"
  name2 = "fieldName"  // duplicate
  type = VarType
  index = <field index>
}
```

## Sources

- `gia_gen/graph.ts` — Graph class, encode() 메서드
- `gia.proto` / `gia.proto.ts` — protobuf 정의
- `ir_to_gia_transform/` — IR -> GIA 변환
