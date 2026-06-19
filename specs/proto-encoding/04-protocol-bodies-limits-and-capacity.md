# HEMP Protobuf Protocol Bodies, Limits, and Capacity

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Protobuf Encoding specification

---

## 1. 目的

この文書は、HEMP Protobuf Encoding における Core-defined protocol channel body、Agreement body、session limits、protocol channel Payload length limit、および capacity model を定義する。

この文書では、次を扱う。

```text
- protocol body validation の共通原則
- Agreement channel body
- AgreementVersion
- AgreementDigest
- SessionLimits
- AgreementReplyBody
- ping body
- shutdown body
- cancel body
- protocol channel Payload length limit
- Agreement 成立前後の length / limit 判定
- capacity model
- Standard Data Body capacity との関係
- この文書で定義しないこと
```

この文書は、HEMP Core semantics を再定義しない。  
この文書は、Core-defined protocol channel body を Protobuf Encoding 上でどのような message structure、presence rule、validation rule、および length / capacity rule として扱うかを定義する。

---

## 2. protocol body validation の共通原則

Protocol body validation は、decode 済み `hemp.v1.HemPayload` の selected body branch が、対象 protocol channel の Body Contract に適合するかを検証する。

Protocol body validation は、次を扱う。

```text
- required message field presence
- required optional scalar field presence
- enum / string / bytes / numeric field の semantic validation
- protocol body 固有の value range
- protocol body 固有の result / message rule
```

Protocol body の required message field が欠落した場合は、HEM Body Contract failure とする。

Protocol body の required optional scalar field が欠落した場合は、HEM Body Contract failure とする。

Optional scalar field では、default value が意味上有効な値になり得る。  
そのため、presence と semantic value validation は区別する。

例:

```text
optional bool result = false:
  presence があれば valid scalar value である。

optional string reason = "":
  presence があれば valid scalar value になり得る。

optional uint32 channel = 0:
  presence はあるが、semantic value として invalid になり得る。
```

Protocol body validation は、Header validation または HEMP flow validation の代替ではない。

Header validation failure または HEMP flow violation が先に該当する場合は、validation order に従って先に該当した failure classification を採用する。

---

## 3. Agreement channel body

Agreement channel は `channel = -1` である。

```text
channel = -1, role = post:
  AgreementPostBody

channel = -1, role = reply:
  AgreementReplyBody
```

Agreement は、1 HEM post と 1 HEM reply で完結する。

Agreement では、次を許可しない。

```text
chunking
notice
sender-side logical-send abort
```

Agreement channel で `role = notice` を使用した場合は、HEMP flow violation とする。

Agreement channel で `abort = true` を使用した場合は、HEMP flow violation とする。

Agreement channel body として `application_body` または `abort_body` を使用してはならない。  
Header と flow が有効であり、body branch が expected Agreement body branch と異なる場合は、HEM Body Contract failure とする。

### 3.1 AgreementPostBody

`AgreementPostBody` は、Agreement post の body である。

構造は次の通りである。

```proto
message AgreementPostBody {
  AgreementVersion version = 1;
  AgreementDigest digest = 2;
  SessionLimits limits = 3;
}
```

`version`、`digest`、`limits` は presence required である。

次は HEM Body Contract failure とする。

```text
AgreementPostBody.version absent
AgreementPostBody.digest absent
AgreementPostBody.limits absent
```

---

## 4. AgreementVersion

`AgreementVersion` は、HEMP protocol version と application identity を提示する。

構造は次の通りである。

```proto
message AgreementVersion {
  optional string protocol = 1;
  AgreementApplicationVersion application = 2;
}

message AgreementApplicationVersion {
  ApplicationIdentity host = 1;
  ApplicationIdentity engine = 2;
}

message ApplicationIdentity {
  optional string name = 1;
  optional string version = 2;
}
```

`AgreementVersion.protocol` は presence required である。

`AgreementVersion.protocol` は HEMP protocol version string である。  
これは Payload encoding selector ではない。

HEMP Protobuf Encoding は、Agreement body を Core Payload encoding negotiation のために使用しない。

`AgreementVersion.protocol` は、Semantic Versioning 2.0.0 に準拠する version string とする。

初期値は次である。

```text
1.0.0
```

Semantic Versioning 2.0.0 validation は、次の正規表現によって行う。

```text
^(0|[1-9]\d*)\.(0|[1-9]\d*)\.(0|[1-9]\d*)(?:-((?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*))*))?(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?$
```

Receiver は `AgreementVersion.protocol` に対して、次を行ってはならない。

```text
trim
case folding
Unicode normalization
v prefix removal
leading zero removal
missing component completion
```

`AgreementVersion.protocol` が SemVer 2.0.0 として invalid な場合は、HEM Body Contract failure とする。

`AgreementVersion.protocol` が SemVer 2.0.0 として valid だが、receiver が対応しない protocol version である場合は、Agreement failure とする。  
この場合は、Agreement reply の `version.protocol.result = false` によって表現する。

この仕様は version range expression を定義しない。

```text
not defined:
  >=1.0.0 <2.0.0
  ^1.0.0
  ~1.0.0
  1.x
  *
```

`AgreementVersion.application`、`AgreementApplicationVersion.host`、`AgreementApplicationVersion.engine` は presence required である。

`ApplicationIdentity.name` と `ApplicationIdentity.version` は presence required である。

Application identity の `name` と `version` は、case-sensitive exact string として扱う。

`ApplicationIdentity.version` は、HEMP Core によって SemVer として解釈しない。  
Application identity version の意味は、該当 application が定義する。

---

## 5. AgreementDigest

`AgreementDigest` は、protocol digest と application digest を提示する。

Digest は raw SHA-256 bytes とする。

構造は次の通りである。

```proto
message AgreementDigest {
  ChannelDigest protocol = 1;
  ChannelDigest application = 2;
}

message ChannelDigest {
  optional string name = 1;
  optional bytes digest = 2;
}
```

`AgreementDigest.protocol` と `AgreementDigest.application` は presence required である。

`ChannelDigest.name` は presence required である。

`ChannelDigest.name` は、ASCII 限定の strict lower_snake_case string とする。

正規表現は次で定義する。

```text
^[a-z][a-z0-9]*(?:_[a-z][a-z0-9]*)*$
```

`ChannelDigest.name` に使用できる文字は次だけである。

```text
a..z
0..9
_
```

ただし、先頭文字は lowercase ASCII letter でなければならない。

次は invalid とする。

```text
empty string
末尾 underscore
連続 underscore
uppercase letter
hyphen
dot
whitespace
non-ASCII character
```

Receiver は `ChannelDigest.name` に対して、次を行ってはならない。

```text
lowercase conversion
Unicode normalization
trim
hyphen to underscore conversion
dot to underscore conversion
case-insensitive comparison
```

`ChannelDigest.digest` は presence required である。

`ChannelDigest.digest` の length は 32 bytes でなければならない。

`ChannelDigest.digest` は raw SHA-256 digest bytes である。  
lowercase hex text ではない。

---

## 6. SessionLimits

`SessionLimits` は、Agreement post で提示する session limits を表す。

構造は次の通りである。

```proto
message SessionLimits {
  optional uint32 hem_payload_length = 1;
  OpenThreadsLimits open_threads = 2;
  RoleLimits role = 3;
  ProtocolLimits protocol = 4;
}

message OpenThreadsLimits {
  optional uint32 per_channel = 1;
  optional uint32 total = 2;
}

message RoleLimits {
  RoleLimit post = 1;
  RoleLimit reply = 2;
  RoleLimit notice = 3;
}

message RoleLimit {
  optional uint32 hem_count = 1;
  optional uint32 payload_total_length = 2;
}

message ProtocolLimits {
  CancelProtocolLimits cancel = 1;
}

message CancelProtocolLimits {
  optional uint32 in_flight_requests = 1;
}
```

`SessionLimits.hem_payload_length` は presence required である。

`SessionLimits.open_threads`、`SessionLimits.role`、`SessionLimits.protocol` は presence required である。

`OpenThreadsLimits.per_channel`、`OpenThreadsLimits.total` は presence required である。

`RoleLimits.post`、`RoleLimits.reply`、`RoleLimits.notice` は presence required である。

`RoleLimit.hem_count`、`RoleLimit.payload_total_length` は presence required である。

`ProtocolLimits.cancel` は presence required である。

`CancelProtocolLimits.in_flight_requests` は presence required である。

Validation range は次の通りである。

```text
hem_payload_length:
  1..4294967295

open_threads.per_channel:
  1..4294967295

open_threads.total:
  1..4294967295

role.*.hem_count:
  1..4294967295

role.*.payload_total_length:
  1..4294967295

protocol.cancel.in_flight_requests:
  1..4294967295
```

`hem_payload_length` は、serialized `hemp.v1.HemPayload` bytes 全体の上限である。  
`application_body` bytes の直接上限ではない。

```text
hem_payload_length:
  serialized HemPayload bytes 全体の上限
```

`open_threads.per_channel` は、1 channel あたり同時に open できる thread 数の上限である。

`open_threads.total` は、session 全体で同時に open できる thread 数の上限である。

`role.<role>.hem_count` は、1つの logical send に含められる HEM 数の上限である。

`role.<role>.payload_total_length` は、1つの logical send に含まれる serialized `HemPayload` length の合計上限である。  
これは body bytes の合計上限ではない。

`protocol.cancel.in_flight_requests` は、同時に in-flight として扱える cancel request 数の上限である。

Agreement に次の limit は追加しない。

```text
max_channel
max_thread_id
max_seq
```

Application channel の有効性は、Agreement 済み Application Channel Table によって決まる。

Thread identifier と seq の値域は Header validation と flow validation によって扱う。

---

## 7. AgreementReplyBody

`AgreementReplyBody` は、Agreement reply の body である。

構造は次の通りである。

```proto
message AgreementReplyBody {
  AgreementReplyVersion version = 1;
  AgreementReplyDigest digest = 2;
  AgreementReplyLimits limits = 3;
}

message AgreementResult {
  optional bool result = 1;
  optional string message = 2;
}

message AgreementReplyVersion {
  AgreementResult protocol = 1;
  AgreementReplyApplicationVersion application = 2;
}

message AgreementReplyApplicationVersion {
  AgreementResult host = 1;
  AgreementResult engine = 2;
}

message AgreementReplyDigest {
  AgreementResult protocol = 1;
  AgreementResult application = 2;
}

message AgreementReplyLimits {
  optional bool hem_payload_length = 1;
  AgreementReplyOpenThreadsLimits open_threads = 2;
  AgreementReplyRoleLimits role = 3;
  AgreementReplyProtocolLimits protocol = 4;
  optional string message = 5;
}

message AgreementReplyOpenThreadsLimits {
  optional bool per_channel = 1;
  optional bool total = 2;
}

message AgreementReplyRoleLimits {
  AgreementReplyRoleLimit post = 1;
  AgreementReplyRoleLimit reply = 2;
  AgreementReplyRoleLimit notice = 3;
}

message AgreementReplyRoleLimit {
  optional bool hem_count = 1;
  optional bool payload_total_length = 2;
}

message AgreementReplyProtocolLimits {
  AgreementReplyCancelProtocolLimits cancel = 1;
}

message AgreementReplyCancelProtocolLimits {
  optional bool in_flight_requests = 1;
}
```

`AgreementReplyBody.version`、`AgreementReplyBody.digest`、`AgreementReplyBody.limits` は presence required である。

Agreement reply の version / digest の個別結果には `AgreementResult` を使用する。

`AgreementResult.result` と `AgreementResult.message` は presence required である。

`AgreementResult.result = false` は valid scalar value であり、対応する agreement item が受け入れられなかったことを表す。

`AgreementResult.message` は human-readable explanation である。  
この field は primary machine decision input として使用しない。

`AgreementResult.message = ""` は valid であるが、diagnostic value のため、sender は可能な限り non-empty message を送るべきである。

`AgreementReplyVersion.protocol`、`AgreementReplyVersion.application`、`AgreementReplyApplicationVersion.host`、`AgreementReplyApplicationVersion.engine` は presence required である。

`AgreementReplyDigest.protocol`、`AgreementReplyDigest.application` は presence required である。

Agreement reply limits は、Agreement post body の `SessionLimits` 構造に対応する boolean leaf を持つ。

すべての boolean leaf は presence required である。

`AgreementReplyLimits.hem_payload_length`、`AgreementReplyLimits.open_threads`、`AgreementReplyLimits.role`、`AgreementReplyLimits.protocol`、`AgreementReplyLimits.message` は presence required である。

`AgreementReplyOpenThreadsLimits.per_channel`、`AgreementReplyOpenThreadsLimits.total` は presence required である。

`AgreementReplyRoleLimits.post`、`AgreementReplyRoleLimits.reply`、`AgreementReplyRoleLimits.notice` は presence required である。

`AgreementReplyRoleLimit.hem_count`、`AgreementReplyRoleLimit.payload_total_length` は presence required である。

`AgreementReplyProtocolLimits.cancel` は presence required である。

`AgreementReplyCancelProtocolLimits.in_flight_requests` は presence required である。

`AgreementReplyLimits.message` は limits 全体に対する human-readable explanation である。  
この field は primary machine decision input として使用しない。

`AgreementReplyLimits.message = ""` は valid であるが、discouraged である。

Agreement は、Agreement reply body の validation が成功し、かつすべての `result` および limits boolean leaf が `true` の場合だけ成立する。

`result = false` または limits boolean leaf の `false` は Agreement failure であり、それ自体は HEMP failure ではない。

---

## 8. ping body

`ping` channel は `channel = -2` である。

```text
channel = -2, role = post:
  PingPostBody

channel = -2, role = reply:
  PingReplyBody
```

`PingPostBody` は empty message である。

```proto
message PingPostBody {}
```

`PingPostBody` は empty message だが、`ping_post` oneof branch は存在しなければならない。

`PingReplyBody` は、flat な `result` と `message` を持つ。

```proto
message PingReplyBody {
  optional bool result = 1;
  optional string message = 2;
}
```

`PingReplyBody.result` は presence required である。

`PingReplyBody.result = false` は valid scalar value であり、ping request が受け入れられなかったことを表す。

`PingReplyBody.message` は presence required である。

`PingReplyBody.message` は non-empty でなければならない。

`ping` は Agreement 成立後にのみ使用できる。

`ping` は 1 HEM post と 1 HEM reply で完結する。

`ping` は thread `1` を使用する。

---

## 9. shutdown body

`shutdown` channel は `channel = -3` である。

```text
channel = -3, role = post:
  ShutdownPostBody

channel = -3, role = reply:
  ShutdownReplyBody
```

`ShutdownPostBody` の構造は次の通りである。

```proto
message ShutdownPostBody {
  optional string reason = 1;
}
```

`ShutdownPostBody.reason` は presence required である。

`ShutdownPostBody.reason = ""` は valid である。

HEMP Core は `ShutdownPostBody.reason` の value taxonomy を定義しない。

`ShutdownReplyBody` の構造は次の通りである。

```proto
message ShutdownReplyBody {
  optional bool result = 1;
  optional string message = 2;
}
```

`ShutdownReplyBody.result` は presence required である。

`ShutdownReplyBody.result = false` は valid scalar value であり、この HEMP session がその時点で normal shutdown を受け入れられなかったことを表す。

`ShutdownReplyBody.message` は presence required である。

`ShutdownReplyBody.message` は non-empty でなければならない。

`ShutdownReplyBody.result = true` を Host が受信した時点で、その HEMP session は正常終了したものとして扱う。

`ShutdownReplyBody.result = false` の場合、HEMP session は終了状態へ移行しない。

---

## 10. cancel body

`cancel` channel は `channel = -4` である。

```text
channel = -4, role = post:
  CancelPostBody

channel = -4, role = reply:
  CancelReplyBody
```

`CancelPostBody` の構造は次の通りである。

```proto
message CancelPostBody {
  CancelTarget target = 1;
  optional string reason = 2;
}

message CancelTarget {
  optional uint32 channel = 1;
  optional uint32 thread = 2;
}
```

`CancelPostBody.target` と `CancelPostBody.reason` は presence required である。

`CancelPostBody.reason = ""` は valid である。

HEMP Core は `CancelPostBody.reason` の value taxonomy を定義しない。

`CancelTarget.channel` と `CancelTarget.thread` は presence required である。

`CancelTarget.channel` は application channel reference である。

Validation range は次の通りである。

```text
target.channel:
  1..2147483647

target.thread:
  1..4294967295
```

`CancelTarget.channel` の Protobuf scalar type は `uint32` である。  
ただし、`2147483648..4294967295` は semantic value として invalid であり、Body Contract validation で拒否する。

`CancelReplyBody` の構造は次の通りである。

```proto
message CancelReplyBody {
  optional bool result = 1;
  optional string message = 2;
}
```

`CancelReplyBody.result` は presence required である。

`CancelReplyBody.result = false` は valid scalar value であり、cancel request が受け入れられなかったことを表す。

`CancelReplyBody.message` は presence required である。

`CancelReplyBody.message` は non-empty でなければならない。

`CancelReplyBody.result = true` は、cancel request を受け付けたことだけを表す。  
対象処理が停止済み、完了済み、または aborted 状態で終端することを保証しない。

`cancel` と `abort` は別概念である。

---

## 11. protocol channel Payload length limit

HEMP Protobuf Encoding は、Core-defined protocol channel HEM の Payload length 上限を次で定義する。

```text
protocol_channel_payload_length_limit = 64 KiB
```

```text
64 KiB = 64 * 1024 = 65,536 bytes
```

この上限は、serialized `hemp.v1.HemPayload` bytes 全体に適用する。

```text
protocol channel HEM Payload length
=
len(serialized hemp.v1.HemPayload bytes)
```

適用対象は、Core-defined protocol channel である。

```text
channel = -1: agreement
channel = -2: ping
channel = -3: shutdown
channel = -4: cancel
```

Agreement 成立前に受信する `agreement` HEM にも適用する。

`protocol_channel_payload_length_limit` は、Agreement 済み `body.limits.hem_payload_length` とは別の HEMP Core 固定上限である。

```text
protocol channel:
  protocol_channel_payload_length_limit

application channel:
  body.limits.hem_payload_length
```

Agreement 成立前に、HEM Length Header value が `protocol_channel_payload_length_limit` を超える場合、Receiver は Payload bytes の読み取りを試みず、HEMP framing failure とする。

Agreement 成立後に、受信した protocol channel HEM Payload bytes の length、すなわち serialized `hemp.v1.HemPayload` bytes 全体の length が `protocol_channel_payload_length_limit` を超える場合、HEMP flow violation とする。

---

## 12. Agreement 成立前後の length / limit 判定

Agreement 成立前の length / limit 判定は、Agreement channel の受信を安全に扱うために行う。

Agreement 成立前に受信する `agreement` HEM では、HEM Length Header value を `protocol_channel_payload_length_limit` と比較する。

```text
HEM Length Header value > protocol_channel_payload_length_limit
→ HEMP framing failure
```

この場合、Receiver は宣言された Payload bytes の読み取りを試みる必要はない。

Agreement 成立前に、HEM Length Header value が `protocol_channel_payload_length_limit` 以下である場合、Receiver は Payload bytes を読み取り、`hemp.v1.HemPayload` として decode し、Agreement body validation を適用する。

Agreement 成立後の protocol channel HEM では、受信した HEM Payload bytes の length、すなわち serialized `hemp.v1.HemPayload` bytes 全体の length を `protocol_channel_payload_length_limit` と比較する。

```text
len(received HEM Payload bytes) > protocol_channel_payload_length_limit
→ HEMP flow violation
```

Agreement 成立後の application channel HEM では、受信した HEM Payload bytes の length、すなわち serialized `hemp.v1.HemPayload` bytes 全体の length を Agreement 済み `body.limits.hem_payload_length` と比較する。

```text
len(received HEM Payload bytes) > body.limits.hem_payload_length
→ HEMP flow violation
```

`body.limits.hem_payload_length` は application channel HEM の `application_body` bytes または `abort_body` bytes だけに適用する直接上限ではない。

Protocol channel HEM には、Agreement 済み `body.limits.hem_payload_length` ではなく、`protocol_channel_payload_length_limit` を適用する。

---

## 13. capacity model

`body.limits.hem_payload_length` は、serialized `hemp.v1.HemPayload` bytes 全体の上限である。

```text
body.limits.hem_payload_length
=
serialized HemPayload bytes 全体の上限
```

HEMP Protobuf Encoding は、Body Contract が使用する body bytes の保証容量を導出するため、次の定数を定義する。

```text
max_core_envelope_length = 64 bytes
```

Body Contract が使用する body bytes の保証容量は次である。

```text
hem_body_capacity
=
max(0, body.limits.hem_payload_length - max_core_envelope_length)
```

すなわち、次と等しい。

```text
hem_body_capacity
=
max(0, body.limits.hem_payload_length - 64)
```

`hem_body_capacity` は Body Contract 向けの導出容量である。  
Core 共通の `application_body` bytes 強制上限ではない。

Core が直接検証する基本上限は、実際の受信 HEM Payload bytes の length、すなわち serialized `hemp.v1.HemPayload` bytes 全体の length である。

```text
len(received HEM Payload bytes) <= body.limits.hem_payload_length
```

これに違反する場合は HEMP flow violation とする。

`hem_body_capacity` は、Application Body Contract が data bytes capacity または body bytes capacity を定義するときに使用できる保証容量である。

Application Body Contract が `hem_body_capacity` をどのように使用するかは、その Application Body Contract が定義する。

---

## 14. Standard Data Body capacity との関係

Standard Data Body は、`hem_body_capacity` を data bytes capacity として採用する。

```text
standard_data_body_data_capacity = hem_body_capacity
```

Standard Data Body を使用する application channel では、通常 body branch は `application_body` である。

```text
channel > 0
abort = false
body = application_body
application_body bytes == data bytes fragment
```

Standard Data Body の通常 Data Body HEM では、次を満たさなければならない。

```text
len(application_body) <= standard_data_body_data_capacity
```

これに違反する場合、Header と flow が有効であれば HEM Body Contract failure とする。

一方、受信した HEM Payload bytes の length、すなわち serialized `hemp.v1.HemPayload` bytes 全体の length が Agreement 済み `body.limits.hem_payload_length` を超える場合は、HEMP flow violation とする。

```text
len(received HEM Payload bytes) > body.limits.hem_payload_length
→ HEMP flow violation

len(application_body) > standard_data_body_data_capacity
→ HEM Body Contract failure
```

Standard Data Body の chunking、empty data、logical send completion、または abort handling の詳細は、この文書では定義しない。

---

## 15. この文書で定義しないこと

この文書は、Core-defined protocol channel body、Agreement body、session limits、protocol channel Payload length limit、および capacity model を定義する。

この文書では、次を定義しない。

```text
HemPayload / HemHeader / body oneof mapping の基本規則
application_body / abort_body の基本 mapping
Standard Data Body の chunking 詳細
Standard Data Body の empty data 詳細
Standard Data Body の logical send completion
Standard Data Body の abort handling
unknown fields の詳細規則
Core Envelope normalization
parse-reserialize
raw frame forward
decode / validation failure mapping 全体
failure response policy alignment
schema evolution / freeze policy
Transport Binding profile
Local Process IPC Binding
generated protobuf code の公開 API
特定言語 runtime の実装方針
```

これらは、後続の Protobuf Encoding specification documents、HEMP Core specification、Standard Data Body rules、または Transport Binding specification が定義する。
