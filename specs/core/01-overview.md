# HEMP Core protocol specification 概要

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Core protocol specification

---

## 1. この文書の目的

この文書は、HEMP Core protocol specification の入口である。

HEMP（Host-Engine Message Protocol）は、Host と Engine の間で HEM を交換するための application-layer protocol である。

HEM（Host-Engine Message）は、HEMP において Host と Engine の間で交換される message である。

この文書では、HEMP Core が定義する対象、HEM frame の基本構造、HEM Header / HEM Body の関係、channel、flow、agreement、limits、Standard Data Body、failure handling、Transport Binding との関係を概説する。

個別の field 定義、validation rule、body rule、failure response rule、encoding rule、transport rule は、後続文書で定義する。

---

## 2. HEMP Core が定義する範囲

HEMP Core は、HEMP protocol の基本意味論を定義する。

HEMP Core は、少なくとも次を定義する。

```text
- HEM framing
- Core Header semantics
- channel
- thread
- seq
- end / close / abort
- Channel Table
- Protocol Channel Table
- Application Channel Table
- digest
- post / reply / notice
- HEM Timeline
- agreement
- limits
- protocol channel Payload length上限
- 未知fieldの受信規則
- Standard Data Body
- data body rules
- HEM Body Contract failure
- failure分類
- failure response policy
```

HEMP Core は、具体的な transport 実装、具体的な wire encoding schema、application channel ごとの個別 HEM Body Contract、または実装 API を定義しない。

ただし、HEMP Core は、application channel 上で有限の不透明 byte 列を 1つの post / reply / notice 論理送信として扱うための標準 data body rules として、Standard Data Body を定義する。

---

## 3. HEM frame の基本構造

HEM は、HEMP において Host と Engine の間で交換される message である。

transport 上では、HEM は HEM frame として運ばれる。

HEM frame は、HEM Length Header と HEM Payload から構成される。

```text
HEM frame
  = HEM Length Header + HEM Payload
```

HEM Length Header は、後続する HEM Payload のバイト長を示す。

HEM Payload は、概念上、HEM Header と HEM Body を含む。

HEM Payload の具体的な wire encoding は、Protobuf Encoding specification が定義する。

---

## 4. HEM Payload encoding との関係

HEMP Core は、HEM Payload の意味論を定義する。

HEM Payload の具体的な wire encoding は、Protobuf Encoding specification が定義する。

HEMP Core は、Protobuf package、message、field number、wire type、oneof branch、optional presence、Protobuf decode、unknown field normalization、または `.proto` schema の詳細をこの文書では定義しない。

これらの詳細は、`specs/proto-encoding/` で定義する。

---

## 5. HEM Header と HEM Body

HEM Header は、HEM Body を HEMP 上でどのように解釈するかを示す Core Header field set である。

HEM Body の意味は、少なくとも次によって決まる。

```text
- channel
- direction
- role
- abort
- Payload encoding が選択する body branch
```

HEM Header fields の個別定義は、`03-hem-header-fields.md` で定義する。

HEM frame、HEM Payload、HEM Header の構造は、`02-hem-frame-payload-header.md` で定義する。

HEM Body に関する基本規則は、`08-data-body-rules.md` で定義する。

---

## 6. channel

HEMP は、HEM を channel によって分類する。

channel は、protocol channel と application channel に分かれる。

```text
channel < 0
  protocol channel

channel > 0
  application channel

channel = 0
  invalid
```

protocol channel は、HEMP protocol 自体の制御や調整に使う channel である。

application channel は、application data を送受信するための channel である。

application channel ごとの個別 HEM Body Contract は、HEMP Core では定義しない。

ただし、HEMP Core は、application channel で利用できる標準 data body rules として Standard Data Body を定義する。

どの application channel が Standard Data Body を使用するかは、application channel ごとの Body Contract が定義する。

protocol channel の詳細は、`07-protocol-channels.md` で定義する。

application channel の HEM Body に関する規則は、`08-data-body-rules.md` で定義する。

---

## 7. Standard Data Body と data body rules

Standard Data Body は、HEMP protocol の標準 data body rules である。

Standard Data Body は、application channel 上で有限の不透明 byte 列を、1つの post / reply / notice 論理送信として扱うための規則を定義する。

Standard Data Body では、1つの application data は、1つの HEMP 論理 post / reply / notice 送信として扱う。

1つの application data が複数の HEM に分割される場合でも、それらは同じ論理送信に属する。

application data の境界は、HEM frame 境界ではなく、`end = true` による論理送信終端で表す。

Standard Data Body は、固定 data、逐次生成 data、empty data、複数 HEM への分割、正常完了不能時の abort 連携、Data Body Body Contract failure 条件などを扱う。

Standard Data Body の詳細は、`08-data-body-rules.md` で定義する。

---

## 8. Channel Table と digest

HEMP は、channel の定義と合意状態を扱うために Channel Table を定義する。

Channel Table には、Protocol Channel Table と Application Channel Table がある。

HEMP は、Channel Table に基づく digest を扱う。

digest は、Host と Engine が同じ Channel Table を認識しているかを確認するために使われる。

Channel Table、Protocol Channel Table、Application Channel Table、Channel Entry、digest、protocol digest、application digest、application identity の詳細は、`05-channel-table-digest.md` で定義する。

---

## 9. flow semantics

HEMP は、HEM の送受信関係と順序を扱うための flow semantics を定義する。

flow semantics には、少なくとも次が含まれる。

```text
- HEM Thread
- HEM Timeline
- post
- reply
- notice
- sender-side logical-send abort
- aborted logical send
- abort body
- aborted HEM Timeline
```

sender-side logical-send abort は、transport failure ではない。

HEMP flow semantics の詳細は、`04-flow-semantics.md` で定義する。

---

## 10. agreement と limits

HEMP session では、Host と Engine が通信前提を共有するために agreement と limits を扱う。

agreement は、HEMP session における前提の合意に使われる。

limits は、HEMP session で許可される容量や制限を扱う。

agreement、limits、およびそれらに関連する protocol body の詳細は、`06-agreement-limits.md` で定義する。

---

## 11. failure handling

HEMP は、failure分類と failure response policy を定義する。

failure handling は、HEMP framing failure、HEMP payload format failure、HEMP header validation failure、HEMP flow violation、HEM Body Contract failure などを扱う。

Standard Data Body の規則に違反する場合は、条件に応じて HEM Body Contract failure として扱う。

failure response policy は、failure分類ごとの返答方針を扱う。

failure分類と failure response policy の詳細は、`09-failure-handling.md` で定義する。

---

## 12. Transport Binding との関係

HEMP Core は、HEM frame と Core semantics を定義する。

Transport Binding は、HEM frame を具体的な transport 上でどのように運ぶかを定義する。

Transport Binding は、HEMP Core semantics を変更しない。

Transport Binding は、`abort` の意味を緩和、変更、または再定義しない。

HEMP Core と Transport Binding の関係は、`10-transport-binding-relationship.md` で定義する。

Transport Binding specification は、`specs/transport-binding/` で定義する。

---

## 13. 後続文書の読み順

HEMP Core protocol specification は、次の順に読む。

```text
01-overview.md
02-hem-frame-payload-header.md
03-hem-header-fields.md
04-flow-semantics.md
05-channel-table-digest.md
06-agreement-limits.md
07-protocol-channels.md
08-data-body-rules.md
09-failure-handling.md
10-transport-binding-relationship.md
```
