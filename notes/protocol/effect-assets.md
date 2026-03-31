# Effect Asset System

에디터의 이펙트 에셋 선택 UI 개요. 전체 목록은 별도 문서 참조.

## Effect Asset Types [CONFIRMED]

이펙트는 두 가지 타입으로 분류됨:

| Type                 | Editor Label  | Behavior                     | Related Function                   |
|----------------------|--------------|------------------------------|------------------------------------|
| Timed (한시 이펙트)   | 한시 이펙트   | 일회성 재생 후 자동 소멸        | `playTimedEffects()`               |
| Loop (루프 이펙트)    | 루프 이펙트   | 수동 해제까지 반복 재생         | `mountLoopingSpecialEffect()` / `clearLoopingSpecialEffect()` |

## Full Catalogs

| Type  | File                                             | Unique IDs |
|-------|--------------------------------------------------|------------|
| Timed | [effect-catalog-timed.md](effect-catalog-timed.md) | 1,282      |
| Loop  | [effect-catalog-loop.md](effect-catalog-loop.md)   | 912        |
| **Total** |                                              | **2,194**  |

## Basic Effects (Low ID) [CONFIRMED]

에디터에서 기본 제공하는 범용 이펙트. **한시와 루프 간 ID가 겹치지 않음**.

### Basic ID Distribution Pattern

```
Timed: 2  3  .  5  .  7  .  9  .  .  .  .  14 .  .  17 18 .  20
Loop:  .  .  4  .  6  .  8  .  10 11 12 13 .  .  19 .  .  .  .  24
       -----------------------------------------------------------
ID:    2  3  4  5  6  7  8  9  10 11 12 13 14    19 17 18 20    24
```

기본 이펙트 대역(2~24)에서 한시와 루프가 서로 다른 ID를 점유하며 겹침 없음.
ID 1, 15, 16, 21, 22, 23은 미사용 또는 삭제됨.

## ID Band Structure [CONFIRMED]

| Band              | Range        | Contents                      | Timed/Loop     |
|-------------------|-------------|-------------------------------|----------------|
| Basic effects     | 2 ~ 24      | 범용 이펙트                    | 분리 (겹침 없음) |
| Hit/trajectory    | 10001xxx    | 피격 피드백, 궤적, 투사체 등    | 양쪽 존재       |
| Element/weapon    | 10002xxx    | 원소 화살, 보호막, 무기 이펙트  | 양쪽 존재       |
| Monster A         | 10003xxx    | 유적 가디언, 유적 헌터 등       | 양쪽 존재       |
| Monster/sign/form | 10004xxx    | 슬라임, 츄츄, 감정 부호, 형상  | 양쪽 존재       |
| Element attack    | 10005xxx    | 원소 낙하 공격, 분출, 에너지    | 양쪽 존재       |
| Monster B         | 10006xxx    | 심연 메이지, 우인단 등          | 양쪽 존재       |
| Weapon/barrier    | 10007xxx    | 무기 모션, 배리어, 버프/디버프  | 양쪽 존재       |
| Weapon aim/elem   | 10008xxx    | 무기별 조준, 원소 피격          | 양쪽 존재       |
| Buff/debuff       | 10009xxx    | 상승 버프, 하강 감쇠            | 루프에서 확인    |
| Visual FX         | 10010xxx    | 표식, 빛기둥, 알림, 플래시      | 양쪽 존재       |
| Monster C         | 10011xxx    | 황금 늑대왕, 뇌음권현 등        | 양쪽 존재       |
| Monster D         | 10012xxx    | 꼭두각시 검귀 전투 모션          | 양쪽 존재       |
| Environment       | 10013xxx    | 상승 윈드 필드, 뇌령 받침대     | 양쪽 존재       |

## Observations [CONFIRMED]

- **기본 대역(2~24)**: 한시/루프 간 ID 완전 분리
- **고 대역(10001xxx~)**: 다수 ID가 한시/루프 양쪽에 등재
- ID는 비연속적 -- 중간에 빈 번호 다수 존재
- 에디터 정렬 순서에 일부 역전 존재 (예: 10001235 -> 10001003)
- 전체 이펙트 수: 한시 1,282 + 루프 912 = **2,194개**

## Sources

- 에디터 이펙트 에셋 선택 UI 스크린샷 (2026-03-27)
- 한시 161장, 루프 115장 전수 캡처
