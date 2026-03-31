# Type System

## VarType (Server Type Enum, 28 types) [CONFIRMED]

| ID | Type              | Description          |
|----|-------------------|----------------------|
| 0  | UnknownVar        | 미사용               |
| 1  | Entity            | 엔티티 참조          |
| 2  | GUID              | 고유 식별자          |
| 3  | Integer           | 정수 (bigint in TS)  |
| 4  | Boolean           | 불리언               |
| 5  | Float             | 부동소수점           |
| 6  | String            | 문자열               |
| 7  | GUIDList          | GUID 배열            |
| 8  | IntegerList       | 정수 배열            |
| 9  | BooleanList       | 불리언 배열          |
| 10 | FloatList         | 부동소수점 배열      |
| 11 | StringList        | 문자열 배열          |
| 12 | Vector            | 3D 벡터 (x, y, z)   |
| 13 | EntityList        | 엔티티 배열          |
| 14 | EnumItem          | 열거형               |
| 15 | VectorList        | 벡터 배열            |
| 16 | LocalVariable     | 로컬 변수            |
| 17 | Faction           | 진영                 |
| 20 | Configuration     | Config ID            |
| 21 | Prefab            | Prefab ID            |
| 22 | ConfigurationList | Config ID 배열       |
| 23 | PrefabList        | Prefab ID 배열       |
| 24 | FactionList       | 진영 배열            |
| 25 | Struct            | 구조체               |
| 26 | StructList        | 구조체 배열          |
| 27 | Dictionary        | 딕셔너리             |
| 28 | VariableSnapshot  | 변수 스냅샷          |

**주의**: TypeID 14(EnumItem)와 EntityList(13) 사이에 갭 없음. VectorList는 15. IDs 18-19 are unused.

## DataGroup Mapping [CONFIRMED]

| DataGroup | Meaning       | Types                          |
|-----------|---------------|--------------------------------|
| 0         | Entity        | entity                         |
| 1         | ID family     | guid, config_id, prefab_id     |
| 2         | Integer       | int                            |
| 4         | Float         | float                          |
| 5         | String        | str                            |
| 6         | Boolean       | bool                           |
| 7         | Vector        | vec3                           |
| 10001     | Struct        | struct                         |
| 10002     | Array/List    | 모든 _list 타입                 |
| 10003     | Map/Dict      | dict                           |

**배열 타입은 DataGroup=10002이며, elementDataGroup으로 원소 타입을 표현.**

## Signal Args Type-DataGroup Mapping [CONFIRMED]

| Type Name      | TypeId | DataGroup | elementTypeId | elementDataGroup |
|----------------|--------|-----------|---------------|------------------|
| entity         | 1      | 0         | -             | -                |
| guid           | 2      | 1         | -             | -                |
| int            | 3      | 2         | -             | -                |
| bool           | 4      | 6         | -             | -                |
| float          | 5      | 4         | -             | -                |
| str            | 6      | 5         | -             | -                |
| vec3           | 12     | 7         | -             | -                |
| config_id      | 20     | 1         | -             | -                |
| prefab_id      | 21     | 1         | -             | -                |
| guid_list      | 7      | 10002     | 2             | 1                |
| int_list       | 8      | 10002     | 3             | 2                |
| bool_list      | 9      | 10002     | 4             | 6                |
| float_list     | 10     | 10002     | 5             | 4                |
| str_list       | 11     | 10002     | 6             | 5                |
| entity_list    | 13     | 10002     | 1             | 0                |
| vec3_list      | 15     | 10002     | 12            | 7                |
| config_id_list | 22     | 10002     | 20            | 1                |
| prefab_id_list | 23     | 10002     | 21            | 1                |

## VarBase.Class (Value Storage Discriminant) [CONFIRMED]

| ID    | Class              | Oneof field index | Usage                         |
|-------|--------------------|-------------------|-------------------------------|
| 1     | IdBase             | 101               | ID/식별자 (GUID, ConfigId, PrefabId) |
| 2     | IntBase            | 102               | 정수 값                        |
| 4     | FloatBase          | 104               | 부동소수점 값                   |
| 5     | StringBase         | 105               | 문자열 값                       |
| 6     | EnumBase           | 106               | 열거형 값                       |
| 7     | VectorBase         | 107               | 3D 벡터                        |
| 10000 | ConcreteBase       | 110               | 리플렉티브/특수화 타입 인스턴스  |
| 10001 | StructBase         | 108               | 구조체 값                       |
| 10002 | ArrayBase          | 109               | 배열/리스트 값                  |
| 10003 | MapBase            | 112               | 딕셔너리/맵 값                  |
| 10006 | ClientContainerMeta| -                 | Client container metadata      |
| 10007 | MapPair            | 111               | 키-값 쌍                       |

Rule: `oneof_field_id = class_id + 100` (except Struct/Array/Map which use fixed offsets)

## NodeType Internal Representation [CONFIRMED]

The codebase uses a compact internal type representation (see `nodes.ts`):

```typescript
type NodeType =
  | { t: 'b', b: BaseTag }             // basic type
  | { t: 'e', e: enumId }              // enum
  | { t: 'l', i: NodeType }            // list (i = item type)
  | { t: 's', f: [name, NodeType][] }  // struct
  | { t: 'd', k: NodeType, v: NodeType }  // dictionary
  | { t: 'r', r: string }              // reflect
```

### BaseTag Abbreviations [CONFIRMED]

| Abbrev | Type      | String Repr |
|--------|-----------|-------------|
| `Int`  | Integer   | `Int`       |
| `Flt`  | Float     | `Flt`       |
| `Bol`  | Boolean   | `Bol`       |
| `Str`  | String    | `Str`       |
| `Vec`  | Vector3   | `Vec`       |
| `Gid`  | GUID      | `Gid`       |
| `Ety`  | Entity    | `Ety`       |
| `Pfb`  | PrefabId  | `Pfb`       |
| `Fct`  | Faction   | `Fct`       |
| `Cfg`  | ConfigId  | `Cfg`       |

String representation examples:
- `E<42>` (enum with id 42)
- `L<Int>` (integer list)
- `S<hp:Int,name:Str>` (struct)
- `D<Str,Int>` (dictionary)

## Sources

- `gia.proto.ts` — VarType, VarBase_Class 열거형
- `ir_to_gia_transform/mappings.ts` — SIGNAL_ARG_TYPE_MAP, DataGroup 매핑
- `gia_gen/nodes.ts` — NodeType, BaseTag 정의
