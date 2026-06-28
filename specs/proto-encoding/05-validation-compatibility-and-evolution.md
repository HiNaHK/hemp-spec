# HEMP Protobuf Validation, Compatibility, and Evolution

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Protobuf Encoding specification

---

## 1. 目的

この文書は、HEMP Protobuf Encoding における validation order、unknown fields、Core Envelope normalization、failure mapping、failure response policy alignment、Core Payload encoding の固定方針、および schema evolution / freeze policy を定義する。

この文書では、次を扱う。

```text
- validation order
- unknown fields
- Core Envelope normalization
- parse-reserialize と raw frame forward
- decode / validation failure mapping
- HEMP framing failure
- HEMP payload format failure
- HEMP header validation failure
- HEMP flow violation
- HEM Body Contract failure
- failure response policy alignment
- Core Payload encoding の固定方針
- schema evolution / freeze policy
- reserved field policy
- この文書で定義しないこと
```

この文書は、HEMP Core semantics を再定義しない。  
この文書は、Protobuf Encoding に固有の decode / validation / compatibility 上の扱いを、既存の HEMP failure classification と schema evolution policy に対応づける。

---

## 2. validation order

受信側は、受信した HEM を次の順序で検証する。

```text
1. HEM framing validation
2. Protobuf HemPayload decode validation
3. HemPayload structural validation
4. HemHeader validation
5. HEMP flow validation
6. HEM Body Contract validation
```

先に該当した validation layer の failure classification を採用する。

後続 layer の validation は、前段の validation が成功した場合にだけ意味を持つ。

たとえば、Payload bytes が `hemp.v1.HemPayload` として Protobuf decode できない場合、受信側は `HemHeader`、body oneof branch、flow state、または Body Contract の validation を適用しない。
この場合は HEMP payload format failure とする。

Header validation failure がある場合、受信側はその HEM を有効な flow event として扱わない。

Header と flow が有効であり、selected body branch または body field value が対象 Body Contract に適合しない場合は、HEM Body Contract failure とする。

---

## 3. unknown fields

Protobuf unknown fields は、受信互換性として許容する。

受信側は、次の unknown fields を、その存在のみを理由に HEMP failure として扱ってはならない。

```text
HemPayload の unknown fields
HemHeader の unknown fields
Core-defined protocol body message 内の unknown fields
Core-defined nested message 内の unknown fields
```

unknown fields は、HEMP semantics 上は無視する。

unknown fields は、必須 known field の代替にならない。

unknown fields は、known body oneof branch の代替にならない。

将来の body branch が unknown field として受信された場合、現行の受信側から見ると known body oneof branch は存在しない。
この場合は HEMP payload format failure とする。

unknown fields は、受信した HEM Payload bytes の一部である。  
したがって、unknown fields は length / limit / resource limit 判定から除外しない。

```text
length / limit / resource limit 判定対象:
  received HEM Payload bytes 全体
```

`application_body` bytes と `abort_body` bytes の内部は、HEMP Core からは opaque bytes である。  
その内部形式における unknown fields 相当の扱いは、対象 channel の Application Body Contract が定義する。

---

## 4. Core Envelope normalization

Core Envelope を新規生成する送信側は、正規化された Core Envelope を生成しなければならない。

ここでいう Core Envelope は、`HemPayload`、`HemHeader`、Core-defined body branch、および Core-defined protocol body message structure を指す。

Core Envelope を新規生成する送信側は、少なくとも次を満たさなければならない。

```text
- HemPayload.header を意味上持つ。
- known body oneof branch を意味上1つ持つ。
- Header field 8つを意味上すべて持つ。
- end / close / abort は presence を持つ。
- Header と body branch が整合する。
- unknown fields を出力しない。
- duplicate known fields を出力しない。
- HemPayload.header を複数回出力しない。
- HemPayload.body の known oneof branch を複数個出力しない。
- HemHeader の各 known field を複数回出力しない。
- Core-defined protocol body message 内の known singular field を複数回出力しない。
- embedded message merge semantics に依存した意味を作らない。
```

Core Envelope を新規生成する送信側は、Core Envelope に unknown fields を出力してはならない。

Core Envelope を新規生成する送信側は、Core Envelope に duplicate known fields を出力してはならない。

正規化は、deterministic serialization を要求しない。

正規化は、canonical protobuf byte sequence を要求しない。

HEMP Protobuf Encoding が要求するのは、byte-level canonicalization ではなく、HEMP semantics 上の Core Envelope が正しく、かつ capacity model の前提を満たす形で表現されていることである。

受信側は、standard Protobuf decode semantics に従い、decode 後の semantic `HemPayload` を validate する。

---

## 5. parse-reserialize と raw frame forward

受信した `HemPayload` を parse し、同じまたは別の `HemPayload` として reserialize して送信する場合、その送信は Core Envelope の新規生成とみなす。

この場合、送信側は unknown fields を継承してはならない。

```text
parse-reserialize:
  Core Envelope の新規生成として扱う。
  unknown fields を継承しない。
```

raw frame forward は、Core Envelope の新規生成とはみなさない。

raw frame forward とは、受信した HEM frame bytes を parse / decode / reserialize せず、そのまま別の transport endpoint へ転送する行為である。

```text
raw frame forward:
  Core Envelope の新規生成とはみなさない。
  unknown fields が保持され得る。
```

raw frame forward では unknown fields が保持され得る。

ただし、raw frame forward される frame の received HEM Payload bytes length は、各種 length / limit / resource limit 判定の対象である。

raw frame forward は、HEMP Core semantics を変更してはならない。

---

## 6. decode / validation failure mapping

HEMP Protobuf Encoding は、HEMP failure classification を新設しない。

Protobuf Encoding 上の decode / validation failure は、既存の HEMP failure classification に写像する。

HEMP Protobuf Encoding は、次の 5 classification を使用する。

```text
1. HEMP framing failure
2. HEMP payload format failure
3. HEMP header validation failure
4. HEMP flow violation
5. HEM Body Contract failure
```

Protobuf decode failure は新しい failure classification ではない。  
Protobuf decode failure は、HEMP payload format failure の具体例である。

validation order において複数の failure が見える場合、受信側は先に該当した validation layer の failure classification を採用する。

---

## 7. HEMP framing failure

HEMP framing failure は、HEM Length Header と、宣言された HEM Payload bytes を安全に取得できるかに関する failure である。

次は HEMP framing failure とする。

```text
HEM Length Header を 4 bytes 読めない。
HEM Length Header value が 0 である。
宣言された HEM Payload length 分の bytes を読めない。
Agreement 成立前に HEM Length Header value が protocol_channel_payload_length_limit を超えている。
```

Transport Binding Profile が定義する peer-visible な Transport Binding / Profile level の limit に違反する場合、その condition は該当 Transport Binding Profile の規則に従う。

実装が内部的な資源管理のために設ける制限値は、相互運用上の HEMP framing rule または transport message payload max ではない。

実装が仕様上要求される peer-visible limit を扱えない場合、その実装は該当仕様または該当 concrete Transport Binding の要件を満たさない。

HEMP framing failure が発生した場合、受信側は HEM Payload bytes を有効な `hemp.v1.HemPayload` として扱わない。

HEMP framing failure は、Protobuf decode failure ではない。

---

## 8. HEMP payload format failure

HEMP payload format failure は、取得した HEM Payload bytes が `hemp.v1.HemPayload` として decode できるか、および `HemPayload` の top-level structure が成立しているかに関する failure である。

次は HEMP payload format failure とする。

```text
Payload bytes が hemp.v1.HemPayload として Protobuf decode 不能である。
Protobuf wire format が不正である。
known field の wire type が不正である。
embedded message field が decode 不能である。
HemPayload.header が存在しない。
HemPayload.body の known oneof branch が存在しない。
```

unknown field は、known body oneof branch の代替にならない。

empty message branch または empty bytes branch であっても、known oneof branch として選択されていれば、body branch presence は存在する。

その body value が対象 Body Contract に適合するかどうかは、HEM Body Contract validation が判断する。

---

## 9. HEMP header validation failure

HEMP header validation failure は、decode 済み `HemPayload.header` に含まれる `HemHeader` の field value と semantic presence に関する failure である。

次は HEMP header validation failure とする。

```text
direction = DIRECTION_UNSPECIFIED
direction が未定義 enum value である。
role = ROLE_UNSPECIFIED
role が未定義 enum value である。
channel = 0
thread = 0
seq = 0
end の presence がない。
close の presence がない。
abort の presence がない。
end = false && close = true
role = ROLE_NOTICE && close = true
abort = true && end = false
direction が Transport Binding 上の HEM delivery path と一致しない。
```

`direction` と `role` の Protobuf default value は、HEMP Header validation 上 invalid である。

`channel`、`thread`、`seq` の Protobuf default value である `0` も、HEMP Header validation 上 invalid である。

`end`、`close`、`abort` は `optional bool` であり、presence が必須である。
`false` は有効値であるため、欠落と `false` は区別しなければならない。

---

## 10. HEMP flow violation

HEMP flow violation は、Header value 自体は構造上有効であるが、HEMP session state、channel rule、thread rule、seq rule、role rule、limits、または protocol flow に違反する場合の failure である。

次は HEMP flow violation とする。

```text
Agreement 成立前に agreement 以外の有効 HEM を受信した。
Agreement 成立後に agreement channel を使用した。
Protocol Channel Table に存在しない protocol channel を使用した。
Application Channel Table に存在しない application channel を使用した。
disabled channel を使用した。
direction / role が channel の service provider side rule に違反した。
protocol channel で notice を使用した。
protocol channel で abort = true を使用した。
chunking banned channel で end = false を使用した。
notice banned channel で role = notice を使用した。
multi-thread banned channel で許可されない thread を使用した。
seq の欠番、順序違い、または wrap-around がある。
close handshake に違反した。
Agreement 済み limits を超えた。
```

Agreement 済み limits 違反には、少なくとも次を含む。

```text
received HEM Payload bytes length > body.limits.hem_payload_length
logical send HEM count > role.<role>.hem_count
logical send received HEM Payload bytes length total > role.<role>.payload_total_length
open_threads.per_channel 超過
open_threads.total 超過
protocol.cancel.in_flight_requests 超過
```

Protocol channel HEM には、Agreement 済み `body.limits.hem_payload_length` ではなく、`protocol_channel_payload_length_limit` を適用する。

Agreement 成立後に、受信した protocol channel HEM Payload bytes の length が `protocol_channel_payload_length_limit` を超える場合は、HEMP flow violation とする。

---

## 11. HEM Body Contract failure

HEM Body Contract failure は、Header と flow が有効であり、selected body branch または body field value が対象 Body Contract に適合しない場合の failure である。

次は HEM Body Contract failure とする。

```text
Header と body oneof branch が一致しない。
protocol channel で期待される body branch と異なる branch を使用した。
protocol channel で application_body を使用した。
protocol channel で abort_body を使用した。
application channel で protocol body branch を使用した。
abort = false で abort_body を使用した。
abort = true で application_body を使用した。
protocol body message 内の必須 field が欠落している。
protocol body message 内の field value が範囲外である。
Standard Data Body の body が Body Contract に違反している。
```

Agreement reply の `result = false` は、それ自体では HEMP failure ではない。

Protocol reply の `result = false` は、それ自体では HEMP failure ではない。

`abort = true` は、それ自体では HEMP failure ではない。

これらは、HEMP failure classification ではなく、対象 protocol flow または logical send の通常の状態表現として扱う。

---

## 12. failure response policy alignment

HEMP Protobuf Encoding は、failure response policy を新しく定義しない。

failure response policy は HEMP Core semantics 側の規則に従う。

HEMP Protobuf Encoding は、Protobuf decode / validation failure を既存の HEMP failure classification へ写像する。  
その後の返答方針は、分類結果に基づいて HEMP Core specification の failure response policy を適用する。

次の failure には、通常の HEM reply を返さない。

```text
HEMP framing failure
HEMP payload format failure
HEMP header validation failure
HEMP flow violation
```

HEMP は、これらの failure に対する汎用 error reply body を定義しない。

HEM Body Contract failure は別扱いとする。

Header が有効であり、flow state 上も対応 reply を返せる状態である場合、protocol channel の専用 reply body によって `result = false` または対応する boolean leaf `false` を返してよい。

これは汎用 error reply ではない。  
該当 protocol channel の Body Contract に基づく通常の reply body である。

Application channel の Body Contract failure への応答は、該当する Application Body Contract が定義する。

この仕様では次を追加しない。

```text
Protobuf decode error 用の専用 reply
validation error 用の専用 reply
generic error message
generic error code
error protocol channel
error oneof branch
error_body
```

---

## 13. Core Payload encoding の固定方針

HEMP Protobuf Encoding において、HEM Payload は次として定義する。

```text
HEM Payload = serialized hemp.v1.HemPayload protobuf message bytes
```

Agreement body は Core Payload encoding を negotiation しない。

`AgreementVersion.protocol` は HEMP protocol version を表す。  
これは Payload encoding selector ではない。

HEMP Protobuf Encoding は、次を定義しない。

```text
alternate HEM Payload encoding
encoding negotiation
fallback decoding
mixed encoding session
```

将来、HEMP に別の Payload encoding を追加する場合、その encoding はこの仕様の範囲外であり、別仕様または HEMP protocol major revision として扱う。

Transport Binding Profile は、HEM Payload bytes の interpretation、Protobuf schema、body oneof branch mapping、または Payload encoding negotiation を定義しない。

Protocol Channel Table / Application Channel Table の digest 用 canonical byte sequence は、Protobuf `HemPayload` の serialized bytes とは別に定義する。

Protobuf `HemPayload` serialization は、digest 用 canonical representation として使用しない。

---

## 14. schema evolution / freeze policy

この仕様の v1.0.0 において、`package hemp.v1` の wire compatibility および semantic compatibility に関わる schema elements を安定化対象とする。

少なくとも次を安定化対象とする。

```text
package name
message names
field names
field numbers
field types
oneof structure
enum names
enum numeric values
Core Header semantics
body branch mapping
protocol channel identifiers
semantic validation rules tied to fields
```

既存 field number を変更してはならない。

既存 field number を別の意味で再利用してはならない。

既存 field name を別の意味で再利用してはならない。

既存 field type を互換性を壊す形で変更してはならない。

`bytes` と `string` は、Protobuf wire type が同じでも semantic type が異なるため、入れ替えてはならない。

`channel` は `sint32` として安定化対象とする。

`thread` と `seq` は `uint32` として安定化対象とする。

既存 enum numeric value を変更してはならない。

既存 enum numeric value を別の意味で再利用してはならない。

body oneof branch mapping を同一 schema major 内で互換性を壊す形で変更してはならない。

この仕様の v1.0.0 では、`package hemp.v1` に新しい body oneof branch を追加することを互換拡張として定義しない。

新しい body oneof branch が必要な場合、それはこの v1.0.0 仕様の範囲外であり、別仕様または新しい schema major package として扱う。

protocol channel identifier を同一 schema major 内で互換性を壊す形で変更してはならない。

`package hemp.v1` 内の既存 field は、原則として削除しない。  
使用を停止したい field は、可能であれば削除せず `deprecated = true` を付けて残す。

field removal が wire compatibility または semantic compatibility に影響する場合は、新しい schema major package を必要とする。

AgreementVersion.protocol の値を変更するだけでは、`package hemp.v1` 内の schema-incompatible change を正当化しない。

新しい schema major package は、別 package name として定義する。

```proto
package hemp.v2;
```

---

## 15. reserved field policy

`reserved` は、同一 package、同一 major、同一 message lineage の中で、削除済み field number または field name を誤って再利用しないために使用する。

同一 schema major 内で field を削除する場合、削除済み field number または field name は `reserved` によって保護しなければならない。

```proto
reserved 9;
reserved "old_field";
```

ただし、互換性を壊す field removal は、原則として同一 schema major 内では行わない。

新しい major package に、旧 major package の removed field number を機械的に `reserved` として持ち込む必要はない。

```text
hemp.v1:
  removed field number を reserved で保護する。

hemp.v2:
  hemp.v1 の removed field number を機械的に reserved として持ち込む必要はない。
```

ただし、`hemp.v2` が `hemp.v1` の message lineage を実質的に継承し、混同の危険がある場合は、明示的な設計判断として reserved を使用してよい。

---

## 16. この文書で定義しないこと

この文書は、validation、compatibility、および evolution policy を定義する。

この文書では、次を定義しない。

```text
HEM Length Header の詳細
HEM Payload bytes の基本定義
HemPayload field 一覧
HemHeader field 一覧
body oneof branch mapping の詳細
Agreement body の内部構造
SessionLimits の各 field validation
ping / shutdown / cancel body の内部構造
protocol channel Payload length limit の詳細
capacity model の計算詳細
Standard Data Body の chunking 詳細
Standard Data Body の empty data 詳細
Standard Data Body の abort handling
Transport Binding profile
Local Process IPC Binding
generated protobuf code の公開 API
特定言語 runtime の実装方針
```

これらは、他の Protobuf Encoding specification documents、HEMP Core specification、Standard Data Body rules、または Transport Binding specification が定義する。
