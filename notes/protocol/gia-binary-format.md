# GIA Binary Format

## Overview

GIA (Genshin Impact Action/Assembly) files are the node-graph binary format used by Beyond Mode. Each `.gia` file contains a single protobuf-encoded node graph wrapped in a fixed binary envelope.

## Layout [CONFIRMED]

```
+---------------------------------+
| Header (20 bytes, big-endian)   |
+---------------------------------+
| Protobuf Body (variable length) |
+---------------------------------+
| Tail Tag (4 bytes)              |
+---------------------------------+
```

Total header = 20 bytes. Total footer = 4 bytes.
Proto payload = `file_size - 24` bytes.

## Header Fields [CONFIRMED]

| Offset | Size | Field            | Value              | Description                         |
|--------|------|------------------|--------------------|-------------------------------------|
| 0x00   | 4B   | `left_size`      | `file_size - 4`    | 나머지 파일 크기 (remaining size)     |
| 0x04   | 4B   | `schema_version` | `1`                | 고정 (always 1)                      |
| 0x08   | 4B   | `head_tag`       | `0x0326`           | 매직 넘버 (magic number)             |
| 0x0C   | 4B   | `file_type`      | `3`                | GIA 타입 식별자                      |
| 0x10   | 4B   | `proto_size`     | `file_size - 24`   | protobuf 메시지 바이트 길이           |

All multi-byte integers are **big-endian** (network byte order).

## Footer [CONFIRMED]

| Offset | Size | Field       | Value    |
|--------|------|-------------|----------|
| EOF-4  | 4B   | `tail_tag`  | `0x0679` |

## Assertions (from decode.ts)

- `bytes.byteLength - 4 === left_size`
- `schema_version === 1`
- `head_tag === 0x0326`
- `tail_tag === 0x0679`
- `file_type === 3`
- `proto_size === bytes.byteLength - 24`

## Wrap (Encode) Formula

```
header = [proto_size + 20, 1, 0x0326, 3, proto_size]
tail   = [0x0679]
total  = header(20B) + proto(NB) + tail(4B)
```

## Source

- `protobuf/decode.ts` — `wrap()` / `unwrap()` 함수에서 헤더/푸터 인코딩/디코딩 로직 확인
