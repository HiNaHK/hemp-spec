# HEMP Protobuf Framing and Payload Encoding

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Protobuf Encoding specification

---

## 1. 目的

この文書は、HEMP Protobuf Encoding における HEM framing、HEM Length Header、および HEM Payload bytes の encoding を定義する。

この文書では、次を扱う。

```text
- HEM frame の構造
- HEM Length Header
- HEM Payload bytes
- HEM Length Header と schema/hemp.proto の関係
- Payload read と Protobuf decode
- framing failure と payload format failure
- length / limit 判定との関係
- この文書で定義しないこと
```

この文書は、`HemPayload`、`HemHeader`、body branch、protocol body、application body、unknown fields、schema evolution、または Transport Binding profile の詳細を定義しない。

---

## 2. HEM frame の構造

HEMP Protobuf Encoding において、HEM frame は次の構造を持つ。

```text
HEM frame = HEM Length Header + HEM Payload
```

HEM Length Header は、直後に続く HEM Payload bytes の長さを表す。

HEM Payload は、serialized `hemp.v1.HemPayload` protobuf message bytes である。

```text
HEM Payload = serialized hemp.v1.HemPayload protobuf message bytes
```

HEM frame の境界は、HEM Length Header によって判断する。

read 単位、write 単位、buffer 境界、flush 境界、改行、または Transport Binding implementation の内部単位は、HEM frame 境界ではない。

---

## 3. HEM Length Header

HEM Length Header は、直後に続く HEM Payload bytes の長さを表す固定長 header である。

HEM Length Header の形式は次の通りである。

```text
size:       4 bytes
type:       unsigned integer
byte order: big-endian
meaning:    following HEM Payload byte length
```

HEM Length Header の値は、直後に続く HEM Payload bytes の byte length でなければならない。

HEM Length Header 自身の 4 bytes は、HEM Payload length に含めない。

```text
HEM frame length
=
4 bytes
+
HEM Payload length
```

```text
HEM Payload length
=
HEM Length Header value
```

HEM Length Header value は、`0` であってはならない。

---

## 4. HEM Length Header に含めない情報

HEM Length Header は、Payload length 以外の意味を持たない。

HEM Length Header は、次を含んではならない。

```text
encoding type
protocol version
schema version
channel
role
flags
magic bytes
checksum
compression flag
```

HEMP Protobuf Encoding は、HEM Length Header を encoding negotiation、schema negotiation、feature negotiation、または protocol version negotiation のために使用しない。

HEMP Protobuf Encoding は、alternate HEM Payload encoding、fallback decoding、または mixed encoding session を定義しない。

---

## 5. HEM Payload bytes

HEM Payload bytes は、`hemp.v1.HemPayload` を Protobuf wire format で serialize した bytes である。

```text
HEM Payload = serialized hemp.v1.HemPayload protobuf message bytes
```

送信側は、HEM Payload bytes を `hemp.v1.HemPayload` として encode しなければならない。

受信側は、HEM Length Header が示す長さの bytes を HEM Payload bytes として読み取り、その bytes を `hemp.v1.HemPayload` として Protobuf decode しなければならない。

HEM Payload は、HEM Header bytes と HEM Body bytes の単純結合ではない。

HEM Payload は、単一の top-level Protobuf message として encode される。

---

## 6. HEM Length Header と `schema/hemp.proto` の関係

HEM Length Header は、Protobuf message の一部ではない。

HEM Length Header は、`schema/hemp.proto` に含まれない。

`schema/hemp.proto` が定義するのは、HEM Payload bytes として serialize される Protobuf message structure である。

```text
HEM frame:
  HEM Length Header
  HEM Payload

schema/hemp.proto:
  HEM Payload の Protobuf message structure
```

したがって、`hemp.v1.HemPayload` の serialized bytes は HEM Payload に対応する。  
HEM Length Header は、その serialized bytes の前に置かれる framing 要素である。

---

## 7. Payload read と Protobuf decode

受信側は、HEM frame を受信するとき、次の順序で処理する。

```text
1. HEM Length Header の 4 bytes を読み取る。
2. HEM Length Header value を big-endian unsigned integer として解釈する。
3. HEM Length Header value が、HEMP Core または適用される Transport Binding Profile が定義する peer-visible な length limit に違反しないことを確認する。
4. 確認できた場合、HEM Length Header value が示す長さの HEM Payload bytes を読み取る。
5. HEM Payload bytes を hemp.v1.HemPayload として Protobuf decode する。
6. decode 済み HemPayload に対して、後続の structural validation、header validation、flow validation、Body Contract validation を適用する。
```

この文書は、手順 1 から 5 までを扱う。

手順 6 の詳細は、後続の Protobuf Encoding specification documents が定義する。

HEM Length Header value が、HEMP Core または適用される Transport Binding Profile が定義する peer-visible な length limit に違反する場合、受信側は宣言された Payload bytes の読み取りを試みる必要はない。

Payload bytes が `hemp.v1.HemPayload` として Protobuf decode できない場合、その HEM は HEMP payload format failure とする。

---

## 8. framing failure と payload format failure

HEMP Protobuf Encoding は、framing validation と Protobuf decode validation を区別する。

framing validation は、HEM Length Header と、宣言された Payload bytes を安全に取得できるかを扱う。

Protobuf decode validation は、取得した Payload bytes を `hemp.v1.HemPayload` として decode できるかを扱う。

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

次は HEMP payload format failure とする。

```text
Payload bytes が hemp.v1.HemPayload として Protobuf decode 不能である。
Protobuf wire format が不正である。
known field の wire type が不正である。
embedded message field が decode 不能である。
```

`HemPayload` の field presence、`HemHeader` の field validation、body oneof branch、unknown fields、および failure response policy の詳細は、この文書では定義しない。

---

## 9. length / limit 判定との関係

HEM Length Header value は、serialized `hemp.v1.HemPayload` bytes 全体の長さを表す。

```text
HEM Length Header value
=
len(serialized hemp.v1.HemPayload bytes)
```

この length には、known fields、unknown fields、embedded message fields、および body bytes を含む、serialized `hemp.v1.HemPayload` bytes 全体を含める。

この length には、HEM Length Header 自身の 4 bytes を含めない。

Protocol channel Payload length limit、Agreement 成立前後の limit 判定、`body.limits.hem_payload_length`、および capacity model の詳細は、後続の Protobuf Encoding specification documents が定義する。

この文書では、次の基本関係だけを定義する。

```text
Length Header value:
  HEM Payload bytes の宣言長

HEM Payload bytes:
  serialized hemp.v1.HemPayload bytes

limit 判定対象:
  serialized hemp.v1.HemPayload bytes 全体
```

---

## 10. この文書で定義しないこと

この文書は、HEM framing、HEM Length Header、および HEM Payload bytes の Protobuf encoding を定義する。

この文書では、次を定義しない。

```text
HemPayload field 一覧
HemHeader field 一覧
Header field validation の詳細
body oneof branch mapping
application_body の詳細
abort_body の詳細
protocol channel body の message structure
Agreement body の詳細
ping / shutdown / cancel body の詳細
protocol channel Payload length limit の詳細
capacity model の計算詳細
unknown fields の詳細規則
Core Envelope normalization の詳細
decode / validation failure mapping の全体
failure response policy alignment
schema evolution / freeze policy
Transport Binding profile
Local Process IPC Binding
```

これらは、後続の Protobuf Encoding specification documents または Transport Binding specification が定義する。
