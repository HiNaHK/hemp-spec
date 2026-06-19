# HEMP Protobuf Encoding Overview and Schema Relationship

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Protobuf Encoding specification

---

## 1. 目的

この文書は、HEMP Protobuf Encoding specification の位置づけ、HEMP Core semantics との関係、および仕様本文と `schema/hemp.proto` の責務分界を定義する。

HEMP Protobuf Encoding specification は、HEMP Core semantics を Protobuf wire encoding へ写像するための仕様領域である。

この文書では、次を扱う。

```text
- Protobuf Encoding の位置づけ
- HEMP Core semantics との関係
- HEM Payload encoding の概要
- 規範的キーワード
- Protobuf Encoding 仕様本文の責務
- schema/hemp.proto の責務
- 仕様本文と schema artifact の優先関係
- Standard Data Body との関係
- この文書で定義しないこと
```

---

## 2. HEMP Core semantics との関係

HEMP Core specification は、HEMP protocol の Core semantics を定義する。

HEMP Core semantics には、少なくとも次を含む。

```text
HEM frame
HEM Length Header
HEM Payload
HEM Header
HEM Body
channel
thread
seq
end / close / abort
post / reply / notice
HEM Timeline
agreement
limits
protocol channels
Application Body Contract
failure classification
failure response policy
Transport Binding との責務境界
```

HEMP Protobuf Encoding specification は、これらの Core semantics を再定義しない。

HEMP Protobuf Encoding specification が定義するのは、Core semantics を Protobuf wire encoding、Protobuf message structure、および Protobuf decode / validation rules に写像する方法である。

たとえば、`channel`、`thread`、`seq`、`end`、`close`、`abort` の意味は HEMP Core semantics に従う。  
Protobuf Encoding は、それらをどの Protobuf field として表現し、どのように presence、type、value、body branch、decode failure、validation failure として扱うかを定義する。

HEMP Protobuf Encoding specification は、HEMP Core semantics を変更してはならない。

---

## 3. HEM Payload encoding の概要

HEMP Protobuf Encoding において、HEM Payload は次として定義する。

```text
HEM Payload = serialized hemp.v1.HemPayload protobuf message bytes
```

`hemp.v1.HemPayload` は、HEMP の HEM Payload を表す top-level Protobuf message である。

HEM frame の基本構造は、HEMP Core semantics に従う。

```text
HEM frame = HEM Length Header + HEM Payload
```

HEM Length Header は、直後に続く HEM Payload bytes の長さを表す。

HEM Length Header 自身は `schema/hemp.proto` には含まれない。  
`schema/hemp.proto` が定義するのは、HEM Payload bytes として serialize される Protobuf message structure である。

HEMP Protobuf Encoding specification は、次を定義しない。

```text
alternate HEM Payload encoding
encoding negotiation
fallback decoding
mixed encoding session
```

---

## 4. 規範的キーワード

この仕様で用いる `MUST`、`MUST NOT`、`SHOULD`、`SHOULD NOT`、`MAY` は、仕様上の要求レベルを示す。

```text
MUST:
  必ず満たさなければならない。

MUST NOT:
  行ってはならない。

SHOULD:
  強く推奨する。
  ただし、十分な理由がある場合には例外を許す。

SHOULD NOT:
  原則として行うべきではない。
  ただし、十分な理由がある場合には例外を許す。

MAY:
  許可する。
```

これらのキーワードは、HEMP implementation、sender、receiver、Transport Binding implementation、または schema artifact のいずれに対する要件であるかを、文脈に応じて解釈する。

---

## 5. Protobuf Encoding 仕様本文の責務

HEMP Protobuf Encoding 仕様本文は、Protobuf Encoding に関する normative specification text である。

Protobuf Encoding 仕様本文は、少なくとも次を定義する。

```text
HEM Payload encoding
HEM framing と Protobuf Payload bytes の関係
HemPayload message model
HemHeader field mapping
Core Header field の semantic presence
body oneof branch mapping
protocol channel body mapping
application channel body mapping
abort body mapping
protocol body validation
Payload length limit の Protobuf Encoding 上の扱い
capacity model
unknown fields の扱い
Core Envelope normalization
decode / validation failure mapping
failure response policy alignment
schema evolution / freeze policy
```

Protobuf Encoding 仕様本文は、Protobuf schema の構造だけでは表現できない HEMP 上の意味論、validation rule、failure classification、および互換性規則を定義する。

Protobuf Encoding 仕様本文は、HEMP Core semantics を変更しない。  
Core semantics の意味論は HEMP Core specification に従う。

---

## 6. `schema/hemp.proto` の責務

`schema/hemp.proto` は、HEMP Protobuf Encoding の schema artifact である。

`schema/hemp.proto` は、少なくとも次の構造定義について normative である。

```text
syntax
package name
message names
field names
field numbers
field types
oneof structure
enum names
enum numeric values
reserved declarations
```

HEMP Protobuf Encoding の package は次とする。

```proto
package hemp.v1;
```

`hemp.v1.HemPayload` は、HEMP の HEM Payload を表す top-level Protobuf message である。

`hemp.v1` の `v1` は、Protobuf schema package の major namespace である。  
これは HEMP protocol version string とは別概念である。

`schema/hemp.proto` は、HEM Length Header を表現しない。  
HEM Length Header は HEM frame の framing 要素であり、Protobuf message structure ではない。

`schema/hemp.proto` は、Transport Binding、実通信方向の確立方法、I/O API、生成コード API、または特定言語の runtime behavior を定義しない。

---

## 7. 仕様本文と schema artifact の優先関係

HEMP Protobuf Encoding では、仕様本文と `schema/hemp.proto` は互いに補完する。

責務分界は次の通りである。

```text
仕様本文:
  semantics
  validation rules
  failure classification
  body mapping rules
  compatibility rules
  normalization rules

schema/hemp.proto:
  package
  message
  field
  field number
  field type
  oneof
  enum
  schema-level structure
```

本文中に掲載される `proto` block は、説明上の抜粋または代表断片である。  
完全な schema definition は `schema/hemp.proto` を参照する。

同一事項について仕様本文と `schema/hemp.proto` が重複して定義し、かつ両者が矛盾する場合は、次のように扱う。

```text
schema-level structure:
  schema/hemp.proto を優先する。

semantic rule / validation rule / failure mapping:
  仕様本文を優先する。
```

ただし、矛盾が解消不能な場合、その状態は specification defect として扱い、仕様本文または schema artifact の修正対象とする。

`.proto` comments は informative とする。  
ただし、仕様本文が同じ規則を明示的に定義する場合、その規則は仕様本文側で normative とする。

---

## 8. Standard Data Body との関係

Standard Data Body は、HEMP application channel 上で data bytes を送受信するための HEMP 標準 data body rules である。

HEMP Protobuf Encoding specification は、Standard Data Body の意味論を再定義しない。

HEMP Protobuf Encoding specification が定義するのは、Standard Data Body を使用する application channel HEM が、Protobuf Encoding 上でどの body branch と bytes value に対応するかである。

`abort = false` の Standard Data Body HEM は、`application_body` bytes を使用する。

```text
application_body bytes == data bytes fragment
```

`application_body` の bytes value は、その HEM で運ばれる Standard Data Body の data bytes fragment そのものである。

`application_body` の内部に、Standard Data Body 専用の wrapper、length field、type field、checksum、record delimiter、または nested protobuf message は追加しない。

`abort = true` の application channel HEM は、通常の Standard Data Body data bytes fragment を運ばない。  
Protobuf Encoding 上の abort body mapping は、body branch mapping rules に従う。

すべての application channel が自動的に Standard Data Body として扱われるわけではない。  
どの application channel が Standard Data Body を使用するかは、対象 channel の Application Body Contract が定義する。

---

## 9. この文書で定義しないこと

この文書は、Protobuf Encoding specification 全体の概要と責務分界を定義する。

この文書では、次の詳細は定義しない。

```text
HEM Length Header の詳細
HEM Payload length validation の詳細
HemPayload field 一覧
HemHeader field 一覧
Header field validation の詳細
body oneof branch mapping の詳細
protocol channel body の message structure
Agreement body の詳細
ping / shutdown / cancel body の詳細
protocol channel Payload length limit
capacity model の計算詳細
unknown fields の詳細規則
Core Envelope normalization の詳細
decode / validation failure mapping の詳細
failure response policy alignment の詳細
schema evolution / freeze policy の詳細
```

これらは、後続の Protobuf Encoding specification documents が定義する。

また、この文書は次を定義しない。

```text
Transport Binding
Local Process IPC Binding
application channel 固有の Body Contract
generated protobuf code の公開 API
特定言語 runtime の実装方針
特定 build tool の使い方
```
