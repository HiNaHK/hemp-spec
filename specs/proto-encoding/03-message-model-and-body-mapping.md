# HEMP Protobuf Message Model and Body Mapping

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Protobuf Encoding specification

---

## 1. 目的

この文書は、HEMP Protobuf Encoding における Protobuf message model、HEM Header field mapping、および HEM Body の body oneof branch mapping を定義する。

この文書では、次を扱う。

```text
- Protobuf message model の全体
- HemPayload
- HemHeader
- Header field validation
- optional bool presence
- body oneof branch mapping の原則
- protocol channel body mapping
- application channel body mapping
- body branch mismatch
- この文書で定義しないこと
```

この文書は、protocol body の内部構造、limits、capacity model、unknown fields、Core Envelope normalization、decode / validation failure mapping 全体、または schema evolution policy を定義しない。

---

## 2. Protobuf message model の全体

HEMP Protobuf Encoding において、HEM Payload は、top-level Protobuf message である `hemp.v1.HemPayload` として表現する。

```text
HEM Payload = serialized hemp.v1.HemPayload protobuf message bytes
```

`hemp.v1.HemPayload` は、次を持つ。

```text
header:
  HEM Header を表す HemHeader message

body:
  HEM Body を表す oneof
```

`schema/hemp.proto` は、message names、field names、field numbers、field types、oneof structure、および enum values の schema-level structure を定義する。

この文書は、その schema structure を HEMP Core semantics と対応づける。

---

## 3. `HemPayload`

`HemPayload` は、HEM Payload の top-level Protobuf message である。

構造は次の通りである。

```proto
message HemPayload {
  HemHeader header = 1;

  oneof body {
    AgreementPostBody agreement_post = 11;
    AgreementReplyBody agreement_reply = 12;

    PingPostBody ping_post = 13;
    PingReplyBody ping_reply = 14;

    ShutdownPostBody shutdown_post = 15;
    ShutdownReplyBody shutdown_reply = 16;

    CancelPostBody cancel_post = 17;
    CancelReplyBody cancel_reply = 18;

    bytes application_body = 31;
    bytes abort_body = 32;
  }
}
```

`HemPayload.header` は意味上必須である。

```text
HemPayload.header absent
→ HEMP payload format failure
```

`HemPayload.body` の known oneof branch も意味上必須である。

```text
known body oneof branch absent
→ HEMP payload format failure
```

unknown field は、known body oneof branch の代替にならない。

将来の body branch が unknown field として受信された場合、現行の受信側から見ると known body oneof branch は存在しない。
この場合は HEMP payload format failure とする。

empty message branch または empty bytes branch であっても、known oneof branch として選択されていれば、body branch presence は存在する。  
その body value が対象 Body Contract に適合するかどうかは、Body Contract validation が判断する。

---

## 4. `HemHeader`

`HemHeader` は、HEMP Core Header の semantic field set を Protobuf 上で表現する message である。

`HemHeader` は次の 8 field を持つ。

```text
direction
role
channel
thread
seq
end
close
abort
```

構造は次の通りである。

```proto
message HemHeader {
  Direction direction = 1;
  Role role = 2;
  sint32 channel = 3;
  uint32 thread = 4;
  uint32 seq = 5;
  optional bool end = 6;
  optional bool close = 7;
  optional bool abort = 8;
}
```

`direction`、`role`、`channel`、`thread`、`seq` は proto3 `optional` にしない。

これらの field は、欠落時に Protobuf default value へ写像され、その default value が HEMP Header validation 上 invalid になるように定義する。

`end`、`close`、`abort` は `optional bool` とする。  
これらは `false` が意味を持つ有効値であり、欠落と `false` を区別する必要があるためである。

---

## 5. Header field validation

Header field validation は、decode 済み `HemPayload.header` に含まれる `HemHeader` の field value を検証する。

### 5.1 `direction`

`direction` は `Direction` enum とする。

```proto
enum Direction {
  DIRECTION_UNSPECIFIED = 0;
  DIRECTION_TO_ENGINE = 1;
  DIRECTION_TO_HOST = 2;
}
```

次は HEMP header validation failure とする。

```text
direction = DIRECTION_UNSPECIFIED
direction が未定義 enum value である。
direction が Transport Binding 上の HEM delivery path と一致しない。
```

この文書は、Transport Binding 上の HEM delivery path をどのように確立するかを定義しない。

### 5.2 `role`

`role` は `Role` enum とする。

```proto
enum Role {
  ROLE_UNSPECIFIED = 0;
  ROLE_POST = 1;
  ROLE_REPLY = 2;
  ROLE_NOTICE = 3;
}
```

次は HEMP header validation failure とする。

```text
role = ROLE_UNSPECIFIED
role が未定義 enum value である。
```

`ROLE_NOTICE` は Header value としては有効である。

ただし、protocol channel rule、Application Channel Table、または対象 channel の flow rule によって `notice` が禁止される場合、その違反は HEMP flow violation とする。

### 5.3 `channel`

`channel` は `sint32` とする。

```proto
sint32 channel = 3;
```

`channel = 0` は HEMP header validation failure とする。

```text
channel < 0:
  protocol channel space

channel == 0:
  invalid

channel > 0:
  application channel space
```

固定の `-32768..32767` 範囲は設けない。

有効な Header value としては、`0` を除く `sint32` 全範囲を許可する。

```text
valid Header channel range:
  -2147483648..-1
  1..2147483647
```

未定義 protocol channel または未定義 application channel の使用は、Header validation ではなく HEMP flow violation とする。

Agreement に `max_channel` limit は追加しない。  
application channel の有効性は Agreement 済み Application Channel Table によって決まる。

### 5.4 `thread`

`thread` は `uint32` とする。

```proto
uint32 thread = 4;
```

`thread = 0` は HEMP header validation failure とする。

有効な Header value としては、`uint32` の正の範囲を許可する。

```text
valid Header thread range:
  1..4294967295
```

`thread` は sparse identifier である。

HEMP implementation は、thread 番号の数値最大値に比例する資源を確保してはならない。

同時に open できる thread 数は、Agreement 済み `open_threads` limits によって制御する。

Agreement に `max_thread_id` limit は追加しない。

### 5.5 `seq`

`seq` は `uint32` とする。

```proto
uint32 seq = 5;
```

`seq = 0` は HEMP header validation failure とする。

有効な Header value としては、`uint32` の正の範囲を許可する。

```text
valid Header seq range:
  1..4294967295
```

`seq` は wrap-around しない。

`seq` の値自体が有効でも、期待される順序と一致しない場合は HEMP flow violation とする。

Agreement に `max_seq` limit は追加しない。  
logical send あたりの HEM 数は `body.limits.role.<role>.hem_count` によって制御する。

### 5.6 `end` / `close` / `abort`

`end`、`close`、`abort` は `optional bool` とする。

```proto
optional bool end = 6;
optional bool close = 7;
optional bool abort = 8;
```

これらは presence が必須である。

次は HEMP header validation failure とする。

```text
end absent
close absent
abort absent
end = false && close = true
role = notice && close = true
abort = true && end = false
```

---

## 6. optional bool presence

HEMP Protobuf Encoding は、semantic presence が必要であり、かつ Protobuf default value が有効値になり得る scalar field に proto3 `optional` を使用する。

`HemHeader` では、`end`、`close`、`abort` がこれに該当する。

```text
end = false
close = false
abort = false
```

これらの `false` は有効値である。  
したがって、受信側は field が欠落している場合と default value が存在する場合を区別できなければならない。

HEMP Protobuf 受信側は、generated API または reflection API によって、`end`、`close`、`abort` の presence を検査できなければならない。

presence を検査できない Protobuf runtime または language binding は、HEMP Protobuf 受信側の適合実装として使用してはならない。

`direction`、`role`、`channel`、`thread`、`seq` は optional presence によって検出しない。  
これらは欠落時に Protobuf default value へ写像され、その default value を Header validation failure として扱う。

```text
direction absent -> DIRECTION_UNSPECIFIED -> HEMP header validation failure
role absent      -> ROLE_UNSPECIFIED      -> HEMP header validation failure
channel absent   -> 0                     -> HEMP header validation failure
thread absent    -> 0                     -> HEMP header validation failure
seq absent       -> 0                     -> HEMP header validation failure
```

---

## 7. body oneof branch mapping の原則

`HemPayload.body` は、HEM Body を表す known oneof branch である。

期待される body branch は、次によって決まる。

```text
channel
role
abort
```

body branch mapping は、Header validation および適用される flow validation の結果と組み合わせて解釈する。

Header value または flow state が不正な場合は、先に該当する HEMP failure classification を採用する。  
Header と flow が有効であり、body branch が期待と異なる場合は HEM Body Contract failure とする。

unknown field は、known body oneof branch の代替にならない。

`application_body` または `abort_body` は bytes branch である。  
bytes value が empty であっても、oneof branch として選択されていれば body branch presence は存在する。

empty bytes value が対象 Body Contract に適合するかどうかは、この文書では定義しない。

---

## 8. protocol channel body mapping

Protocol channel は `channel < 0` の channel space に属する。

現行 HEMP Protobuf Encoding が定義する protocol channel body mapping は次の通りである。

```text
channel = -1, role = post:
  agreement_post

channel = -1, role = reply:
  agreement_reply

channel = -2, role = post:
  ping_post

channel = -2, role = reply:
  ping_reply

channel = -3, role = post:
  shutdown_post

channel = -3, role = reply:
  shutdown_reply

channel = -4, role = post:
  cancel_post

channel = -4, role = reply:
  cancel_reply
```

Protocol channel では、`application_body` と `abort_body` を使用してはならない。

Protocol channel で `application_body` または `abort_body` を使用した場合、Header と flow が有効であれば HEM Body Contract failure とする。

現行 protocol channel では `notice` を許可しない。  
Protocol channel で `role = notice` を使用した場合は HEMP flow violation とする。

現行 protocol channel では sender-side logical-send abort を使用しない。  
Protocol channel で `abort = true` を使用した場合は HEMP flow violation とする。

Protocol channel body の内部構造および field validation は、この文書では定義しない。

---

## 9. application channel body mapping

Application channel は `channel > 0` の channel space に属する。

Application channel の body branch は `abort` によって決まる。

```text
channel > 0 && abort = false:
  body = application_body

channel > 0 && abort = true:
  body = abort_body
```

`application_body` は Core から opaque bytes である。  
その意味は、対象 channel の Application Body Contract が定義する。

Standard Data Body を使用する application channel では、次の対応を取る。

```text
application_body bytes == data bytes fragment
```

`application_body` の内部に、Standard Data Body 専用の wrapper、length field、type field、checksum、record delimiter、または nested protobuf message は追加しない。

`abort_body` も Core から opaque bytes である。  
`abort_body` は、通常完了時の application body ではない。

`abort_body` の内部構造、reason、message、diagnostic information、または capacity rule は、この文書では定義しない。  
必要な場合は、対象 channel の Application Body Contract が定義する。

Application channel で protocol channel body branch を使用した場合、Header と flow が有効であれば HEM Body Contract failure とする。

---

## 10. body branch mismatch

body branch mismatch は、Header と flow が有効であり、かつ selected body branch が期待される body branch と異なる場合に発生する。

body branch mismatch は HEM Body Contract failure とする。

例:

```text
channel = -2, role = post, body = ping_reply
→ HEM Body Contract failure

channel = -2, role = post, body = application_body
→ HEM Body Contract failure

channel = -2, role = post, body = abort_body
→ HEM Body Contract failure

channel > 0, abort = false, body = abort_body
→ HEM Body Contract failure

channel > 0, abort = true, body = application_body
→ HEM Body Contract failure

channel > 0, body = agreement_post
→ HEM Body Contract failure
```

ただし、Header validation failure または HEMP flow violation が先に該当する場合は、validation order に従って先に該当した failure classification を採用する。

たとえば、protocol channel で `role = notice` を使用した場合は、body branch mismatch ではなく HEMP flow violation とする。

---

## 11. この文書で定義しないこと

この文書は、Protobuf message model、HEM Header field mapping、および body oneof branch mapping を定義する。

この文書では、次を定義しない。

```text
AgreementPostBody の内部構造
AgreementReplyBody の内部構造
AgreementVersion / AgreementDigest / SessionLimits の詳細
PingPostBody / PingReplyBody の内部構造
ShutdownPostBody / ShutdownReplyBody の内部構造
CancelPostBody / CancelReplyBody の内部構造
protocol body validation の詳細
protocol_channel_payload_length_limit
Agreement 成立前後の limit 判定
body.limits.hem_payload_length
max_core_envelope_length
hem_body_capacity
Standard Data Body の chunking / empty data / capacity 詳細
unknown fields の詳細規則
Core Envelope normalization
parse-reserialize
raw frame forward
decode / validation failure mapping 全体
failure response policy alignment
schema evolution / freeze policy
Transport Binding profile
Local Process IPC Binding
```

これらは、後続の Protobuf Encoding specification documents、HEMP Core specification、Standard Data Body rules、または Transport Binding specification が定義する。
