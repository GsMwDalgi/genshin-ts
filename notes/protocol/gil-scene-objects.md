# GIL Scene Objects

GIL 파일의 field 5에 포함된 씬 오브젝트 구조. 에디터에서 맵에 배치한 오브젝트들.

## Scene Object Structure (field 5) [CONFIRMED]

field 5 내부에 repeated로 씬 오브젝트가 나열됨. 각 오브젝트는 `f1` 메시지:

| Field | Meaning            | Type            | Confidence  |
|-------|--------------------|-----------------|-----------  |
| f1    | **object_id**      | varint          | CONFIRMED -- 에디터의 오브젝트 ID와 일치 (예: 1077936130) |
| f2    | 오브젝트 메타데이터  | message         | INFERRED    |
| f5    | (반복) 속성/설정?   | repeated message| SPECULATED  |
| f6    | (반복) 하위 데이터? | repeated message| SPECULATED  |
| f7    | **Components**     | repeated message| CONFIRMED -- 컴포넌트 배열 |
| f8    | **prefab_id?**     | varint          | SPECULATED -- 예: 10005018 |

## Component Structure (f7) [CONFIRMED]

각 컴포넌트의 공통 헤더:

| Field | Meaning            | Type    |
|-------|--------------------|---------|
| f1    | **component_type** | varint  |
| f2    | **sub_type**       | varint  | (관측값 항상 1)

### Observed Component Types [INFERRED]

오브젝트 1077936130에서 관측:

| component_type (f1) | Estimated Role       |
|---------------------|----------------------|
| 1                   | ?                    |
| 3                   | ?                    |
| 6                   | **Effect Component** |
| 14                  | ?                    |
| 18                  | ?                    |
| 19                  | ?                    |

---

## Effect Component (component_type=6) [CONFIRMED]

씬 오브젝트에 부착되는 이펙트 컴포넌트의 protobuf 구조.

### Component Header

```
f1: 6          (component_type = Effect)
f2: 1          (sub_type, 항상 1)
f16: repeated  (Effect Player 엔트리)
```

### Effect Player (f16) [CONFIRMED]

f16 내부에 repeated `f1` 메시지로 개별 이펙트 에셋이 나열됨.
에디터 UI 매핑: "이펙트 플레이어" 패널 -> 각 이펙트 항목

### Effect Entry (f16.f1) [CONFIRMED]

에디터 스크린샷과 바이너리 대비 검증 완료.

| Field | Wire    | Meaning              | Editor UI Label        | Example       | Confidence |
|-------|---------|----------------------|------------------------|---------------|------------|
| f1    | varint  | **effect_asset_id**  | 이펙트 에셋 (id-XXXXX) | 10004139      | CONFIRMED  |
| f2    | string  | **attachment_point** | 부착점                  | "GI_RootNode" | CONFIRMED  |
| f3    | varint  | **follow_position**  | 추적 위치               | 1 (ON)        | CONFIRMED  |
| f4    | varint  | **follow_rotation**  | 회전 추적               | 생략=0 (OFF)  | INFERRED   |
| f5    | bytes   | **position_offset**  | 부착점 > 오프셋          | empty=(0,0,0) | CONFIRMED  |
| f6    | bytes   | **rotation_offset**  | 부착점 > 회전            | empty=(0,0,0) | CONFIRMED  |
| f7    | float32 | **scale**            | "시점 조절 비율" (에디터 오역) | 1.0      | CONFIRMED  |
| f8    | bytes   | ?                    | -                       | empty         | SPECULATED |
| f10   | message | **asset_reference**  | -                       | { f2: id }    | INFERRED   |
| f11   | varint  | **play_sound**       | 이펙트 에셋 효과음 재생 여부 | 1 (ON)    | CONFIRMED  |
| f502  | varint  | **index**            | (내부 순서)              | 1,2,3... (1-based) | CONFIRMED |
| f503  | message | 내부 메타데이터       | -                       | -             | SPECULATED |
| f505  | varint  | ?                    | -                       | 1             | SPECULATED |
| f507  | varint  | **effect_asset_type?** | 이펙트 에셋 타입       | 13            | SPECULATED |

### Editor UI Notes

- "시점 조절 비율" (f7)은 에디터의 잘못된 현지화. 실제로는 **scale** (줌/크기 배율).
- 에디터에서 "오프셋"과 "회전" 필드가 두 번 나타남: 부착점 하위(f5,f6)와 별도 영역. 별도 영역은 미확인.
- f10 (asset_reference)의 f2는 f1(effect_asset_id)과 동일한 ID를 중복 저장.

### Editor Screenshot Verification (id-10004139) [CONFIRMED]

```
Editor                          Binary
-----------------------------------------------
이펙트 에셋: id-10004139    ->  f1: 10004139 OK
이펙트 에셋 타입: 한시 이펙트  ->  f507: 13 (추정)
효과음 재생 여부: ON          ->  f11: 1 OK
추적 위치: ON                 ->  f3: 1 OK
회전 추적: OFF                ->  f4: 생략 (=0) OK
부착점: GI_RootNode           ->  f2: "GI_RootNode" OK
오프셋: (0,0,0)               ->  f5: empty OK
회전: (0,0,0)                 ->  f6: empty OK
시점 조절 비율: 1.00          ->  f7: 1.0 (float32) OK
```

### playTimedEffects() Parameter Mapping [INFERRED]

genshin-ts의 `playTimedEffects` 함수 파라미터와 Effect Entry 필드의 대응:

| playTimedEffects Parameter          | Effect Entry Field    |
|-------------------------------------|-----------------------|
| `specialEffectsAsset: ConfigIdValue` | f1 (effect_asset_id) |
| `targetEntity: entity`              | (런타임 지정, 저장 안 됨) |
| `attachmentPointName: string`       | f2 (attachment_point) |
| `moveWithTheTarget: bool`           | f3 (follow_position)  |
| `rotateWithTheTarget: bool`         | f4 (follow_rotation)  |
| `locationOffset: vec3`              | f5 (position_offset)  |
| `rotationOffset: vec3`              | f6 (rotation_offset)  |
| `zoomMultiplier: float`             | f7 (scale)            |
| `playBuiltInSoundEffect: bool`      | f11 (play_sound)      |

## Analysis Target

- 1073741960-temp.gil, 오브젝트 1077936130
- 이펙트 4개: 10004139 ~ 10004142

## Sources

- 바이너리 raw protobuf decode
- 에디터 스크린샷 대비 검증
- genshin-ts `playTimedEffects` 파라미터 정의
