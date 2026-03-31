# Protocol Analysis: Completion Report

## Files Created

All files written to `notes/protocol/`:

1. **gia-structure.md** — Complete GIA file format documentation
   - 20-byte header + 4-byte footer binary layout
   - Magic numbers: 0x0326 (head), 0x0679 (tail)
   - Full protobuf schema hierarchy (Root -> GraphUnit -> NodeGraph -> GraphNode -> NodePin)
   - VarType enum (28 server-side types), VarBase.Class discriminant table
   - Internal NodeType compact representation

2. **gil-structure.md** — GIL (map save) file format documentation
   - Same binary envelope as GIA (shared magic numbers)
   - Inferred protobuf structure from binary parsing (folder index at 6.1, graphs at 10.1.1, prefabs at 4.1)
   - Category value to graph type mapping (800->entity, 2300->status, 2400->class, 4300->item)
   - File location resolution logic (BeyondLocal path structure)
   - Custom prefab extraction format

3. **gia-vs-gil.md** — Side-by-side comparison
   - Shared: envelope format, magic numbers, protobuf encoding
   - Different: GIA=single graph (Root proto), GIL=full map (undocumented container with many graphs)
   - Structural relationship diagram

4. **injection-flow.md** — Complete injection pipeline documentation
   - End-to-end: TS -> GS IR -> IR JSON -> GIA -> inject into GIL
   - Detailed injection steps (load, parse, signal patch, find target, validate, replace, rebuild)
   - Binary patching mechanics (applyReplacement with ancestor length updates)
   - Signal node resolution system (placeholder IDs 300000/300001 -> real IDs from GIL composite defs)

5. **graph-encoding.md** — Graph builder API and encoding details
   - Graph/Node/Pin/Connect class hierarchy
   - Graph mode to protobuf enum mapping
   - genericId vs concreteId distinction
   - Pin value encoding (simple vs reflective/concrete)
   - Connection encoding (data vs flow)
   - CompositeDef and StructureDefWrapper formats

## Key Findings

1. **GIA and GIL share the same binary envelope** (20B header + 4B footer, magic 0x0326/0x0679), but the protobuf payload schemas differ completely.

2. **The GIL proto schema is not fully reverse-engineered.** Only the paths needed for injection (10.1.1 for NodeGraph blobs, 6.1 for folder index, composite defs for signals) are understood. The full GIL schema remains undocumented.

3. **Signal nodes use a two-phase ID resolution**: compile time uses placeholders (300000/300001), injection time resolves them by scanning composite definitions in the target GIL file.

4. **Binary patching is non-trivial**: replacing a nested protobuf blob requires updating all ancestor length-prefix varints, accounting for varint size changes cascading upward.

5. **The `_GSTS` name prefix** serves as a safety marker. The injector refuses to overwrite non-empty graphs that don't have this prefix, preventing accidental corruption of hand-authored content.

6. **Graph IDs >= 1,000,000,000** are the convention for genshin-ts-generated graphs. The injector validates that matched targets live at the expected protobuf path (10.1.1).
