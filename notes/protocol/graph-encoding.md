# Graph Encoding Details

## Graph Builder API (gia_gen)

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

### Graph Mode Mapping

| Mode      | GraphUnit.Which    | NodeGraph.Id.Type |
|-----------|--------------------|-------------------|
| server    | EntityNode (9)     | BasicNode (20000) |
| status    | StatusNode (22)    | StatusNode (20003)|
| class     | ClassNode (23)     | ClassNode (20004) |
| item      | ItemNode (46)      | ItemNode (20005)  |
| composite | —                  | —                 |
| bool      | BooleanFilter (10) | BooleanFilter     |
| int       | IntegerFilter (47) | IntegerFilter     |
| skill     | Skills (11)        | Skills (20002)    |

### Node Identity

Each node has two IDs:
- **genericId**: The template/class of the node (e.g. "Add Integer" node type). Always `SystemDefined` class, `SysCall` kind.
- **concreteId**: The specific variant/instance. For reflective (generic) nodes, this determines the concrete type instantiation.

```protobuf
NodeProperty {
  class = 10001  (SystemDefined)
  type  = 20000  (Server) or 20001 (Filter) or 20002 (Skill) or 20007 (CreationStatusNode)
  kind  = 22000  (SysCall) or 22001 (SysGraph)
  nodeId = <NODE_ID enum value>
}
```

### Pin Encoding

Pins are the connection points on nodes. Each pin has:

1. **Index pair (i1, i2)**: Both set to the same value — `{ kind, index }`
2. **Value (VarBase)**: For input params, contains the pin's current value
3. **Type (int)**: VarType (server) or ClientVarType (client)
4. **Connections**: For OutFlow pins, list of target nodes; for InParam pins, list of source nodes

#### Pin Value Encoding (VarBase)

Non-reflective (simple) pins:
```
VarBase {
  class = <matching VarBase.Class>
  alreadySetVal = true
  bInt/bFloat/bString/bEnum/bVector = { val: <value> }
}
```

Reflective (concrete) pins:
```
VarBase {
  class = ConcreteBase (10000)
  alreadySetVal = true
  bConcreteValue = {
    indexOfConcrete = <pin-specific concrete index>
    value = <inner VarBase with actual value>
    structs? = <ComplexValueStruct for struct/map/array types>
  }
}
```

#### Index of Concrete

For reflective nodes, each pin position maps to a concrete type index. This index is looked up from the node pin records (`node_data/helpers.ts`):

```typescript
get_index_of_concrete(generic_id, is_input, pin_index, node_type)
```

Returns the position in the reflect map that matches the given type.

### Connection Encoding

**Data connections** (InParam pins reference their source):
```
NodeConnection {
  id = source_node.nodeIndex
  connect  = { kind: InParam(3), index: <source_output_index> }
  connect2 = { kind: InParam(3), index: <source_output_index> }
}
```

**Flow connections** (OutFlow pins reference their targets):
```
NodeConnection {
  id = target_node.nodeIndex
  connect  = { kind: OutFlow(2), index: <target_inflow_index> }
  connect2 = { kind: OutFlow(2), index: <target_inflow_index> }
}
```

### Graph Variables

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

### Comments

Two types:
- **Graph-level comments**: Have x, y coordinates. Floating annotations.
- **Node-attached comments**: Only have content string. Attached to a specific node.

```protobuf
Comments {
  content = "text"
  x? = <float>   // only for graph-level
  y? = <float>   // only for graph-level
}
```

## Composite Graphs (CompositeDef)

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

### CompositeDef.Type.Kind

| Kind  | Value | Description                    |
|-------|-------|--------------------------------|
| Composite | 1000 | Standard composite subgraph |
| Assembly  | 1003 | Structure assembly node     |
| Split     | 1004 | Structure split node        |
| Modify    | 1005 | Structure modify node       |

## Structure Definitions (StructureDefWrapper)

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
