# HEM frame / HEM Payload / HEM Header

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Core protocol specification

---

## 1. この文書の目的

この文書は、HEMP における HEM、HEM frame、HEM Length Header、HEM Payload、HEM Header、HEM Body の基本構造を定義する。

この文書では、HEM frame の framing、HEM Length Header の形式と意味、HEM Payload の位置づけ、HEM Header と HEM Body の関係を扱う。

HEM Header field の個別規則、flow semantics、Channel Table、agreement、limits、protocol channel body、Standard Data Body の詳細、failure response policy、Transport Binding の詳細は、後続文書で定義する。

---

## 2. HEM と論理送信

HEM（Host-Engine Message）は、HEMP 上で送受信される message である。

HEM は、HEM Length Header によって区切られた1つの message 単位である。

1つの論理 post / reply / notice 送信は、1つまたは複数の HEM で構成され得る。

論理送信、HEM Thread、HEM Timeline、post / reply / notice の flow semantics は、`04-flow-semantics.md` で定義する。

---

## 3. HEM frame の基本構造

transport 上では、HEM は HEM frame として運ばれる。

HEM frame は、HEM Length Header と HEM Payload から構成される。

```text
HEM frame
  = HEM Length Header + HEM Payload
```

HEM Length Header は、後続する HEM Payload のバイト長を示す。

HEM Payload は、HEM Length Header の後に続く byte sequence である。

HEM Payload の意味論上の構造は、後続の HEM Payload / HEM Header / HEM Body の節で定義する。

---

## 4. HEM Length Header

### 4.1 形式

HEM Length Header は、固定長バイナリ header である。

```text
size:       4 bytes
type:       unsigned integer
byte order: big-endian
```

### 4.2 意味

HEM Length Header の値は、直後に続く HEM Payload のバイト長を表す。

HEM Length Header 自身の 4 bytes は、この長さに含めない。

```text
含める:
  HEM Payload bytes

含めない:
  HEM Length Header 自身の 4 bytes
```

### 4.3 Payload length の理論上限

HEM Length Header は 4 bytes unsigned integer である。

HEM Length Header で表現できる HEM Payload length の理論上限は次の通りである。

```text
1 ～ 4294967295 bytes
```

次の値は不正とする。

```text
0
```

HEM Length Header の値が `0` の場合、HEMP framing failure とする。

### 4.4 application channel の単体 HEM Payload length

HEMP が application channel 上の通常 HEM 通信に対して推奨する単体 HEM Payload length の最大値は次の通りである。

```text
16 MiB
```

```text
16 MiB = 16 * 1024 * 1024 = 16,777,216 bytes
```

16 MiB は、HEM Length Header で表現できる理論上限ではない。

HEMP を利用する application は、必要に応じて 16 MiB と異なる単体 HEM Payload length 上限を定義してよい。

application channel の通常 HEM 通信で使用する単体 HEM Payload length 上限は、agreement の `body.limits.hem_payload_length` によって定義する。

agreement と limits の詳細は、`06-agreement-limits.md` で定義する。

### 4.5 protocol channel Payload length 上限

HEMP Core が定義する protocol channel HEM の Payload length 上限は次の通りである。

```text
64 KiB
```

```text
64 KiB = 64 * 1024 = 65,536 bytes
```

この上限を protocol channel Payload length 上限と呼ぶ。

protocol channel Payload length 上限は、agreement 成立前に受信する agreement HEM にも適用する。

protocol channel Payload length 上限は、agreement 済み `body.limits.hem_payload_length` とは別の HEMP Core 上限である。

agreement 済み `body.limits.hem_payload_length` は、application channel 上の通常 HEM 通信に適用する。

agreement 成立前に、HEM Length Header の値が protocol channel Payload length 上限を超える場合、受信側はその値に従った Payload bytes の読み取りを試みず、HEMP framing failure とする。

agreement 成立後に受信した HEM が protocol channel HEM であり、その HEM Payload length が protocol channel Payload length 上限を超える場合、HEMP flow violation とする。

protocol channel Payload length は、HEM Payload bytes 全体の byte length である。

agreement 成立後、受信側が HEM Payload bytes を読み取り、HEM Header の `channel` を判定する前に受信 Payload size 上限を適用する場合、その読み取り上限は protocol channel Payload length 上限も考慮しなければならない。

agreement 成立後の HEM について、受信側は agreement 済み `body.limits.hem_payload_length` だけを根拠に、その値を超える HEM Payload length を HEMP framing failure としてはならない。

agreement 成立後に、Payload 読み取り前の暫定的な読み取り上限を設ける場合、その上限は少なくとも次の大きい方を許容しなければならない。

```text
agreement 済み body.limits.hem_payload_length
protocol channel Payload length 上限
```

HEM Payload を読み取り、HEM Header の `channel` を判定した後、application channel HEM には agreement 済み `body.limits.hem_payload_length` を適用し、protocol channel HEM には protocol channel Payload length 上限を適用する。

failure 分類の詳細は、`09-failure-handling.md` で定義する。

### 4.6 HEM Length Header に入れない情報

HEM Length Header には、HEM Payload のバイト長以外の情報を入れない。

HEM Length Header は、次を含まない。

```text
- encoding type
- encoding negotiation information
- protocol version
- schema version
- channel
- role
- direction
- flags
- magic bytes
- checksum
- compression flag
```

HEM Payload の具体的な wire encoding は、Protobuf Encoding specification が定義する。

---

## 5. HEM Payload

HEM Payload は、HEM Length Header の後に続く byte sequence である。

HEM Payload は、HEMP Core semantics 上の Payload である。

概念上、HEM Payload は Header と Body を含む。

```text
HEM Payload semantic structure:
  header:
    Core Header field set

  body:
    channel / direction / role / abort と、
    HEM Payload encoding が選択する body branch によって
    HEMP Core semantics 上の意味が決まる Body
```

HEM Payload を、HEM Header bytes と HEM Body bytes の単純な連結として定義してはならない。

HEM Payload の具体的な byte representation、message structure、field number、wire type、oneof branch、optional presence、および schema literal は、HEM Payload encoding が定義する。

---

## 6. HEM Payload encoding との関係

HEMP Core は、HEM Payload の Core semantics を定義する。

HEM Payload の具体的な wire encoding は、Protobuf Encoding specification が定義する。

HEMP Core は、次の詳細をこの文書では定義しない。

```text
- Protobuf package
- Protobuf message structure
- field number
- wire type
- oneof branch
- optional presence
- Protobuf decode
- unknown field normalization
- .proto schema
```

これらの詳細は、`specs/proto-encoding/` で定義する。

HEMP Core は、JSON HEM Payload encoding、canonical JSON、JSON object field 重複規則、pretty print、または JSON parser 動作を定義しない。

Canonical byte 表現を要求する対象は、digest の対象である canonical protocol channel table byte sequence と canonical application channel table byte sequence である。

Channel Table と digest の詳細は、`05-channel-table-digest.md` で定義する。

---

## 7. HEM Header

HEM Header は、HEM Payload 内にある protocol control 用の Core Header field set である。

HEM Header は、HEM Body を HEMP 上でどのように解釈するかを示す。

HEM Header は、HEMP が定義する必須 field として次の8 fieldを持つ。

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

これらを Core Header field とする。

Core Header field は必須である。

Core Header field が欠落している、値が不正である、範囲外である、または組み合わせが不正である場合、HEMP header validation failure とする。

HEM Header に Core Header field 以外の既知でない field が含まれる場合、Core semantics 上は未知fieldとして無視する。

未知fieldは Core Header field の代替にはならない。

Protobuf unknown fields の具体的な扱いは、Protobuf Encoding specification が定義する。

Core Header field の個別定義、値の範囲、意味、組み合わせ、validation rule は、`03-hem-header-fields.md` で定義する。

---

## 8. HEM Body

HEM Body は、HEM Payload 内にある Body 部分である。

HEM Body の意味は、少なくとも次によって決まる。

```text
- channel
- direction
- role
- abort
- Payload encoding が選択する body branch
```

`channel < 0` の場合、HEM Body は HEMP が定義する protocol channel の Body Contract に従う。

`channel > 0` の場合、HEM Body は該当する application channel の Body Contract に従う。

ただし、HEMP Core は、application channel で利用できる標準 data body rules として Standard Data Body を定義する。

どの application channel が Standard Data Body を使用するかは、application channel ごとの Body Contract が定義する。

`abort = true` の body は abort body として扱い、通常完了時の application Body Contract による正常 body 検証の対象とはしない。

protocol channel body の詳細は、`07-protocol-channels.md` で定義する。

Standard Data Body と data body rules の詳細は、`08-data-body-rules.md` で定義する。

Payload encoding における body branch mapping の詳細は、`specs/proto-encoding/` で定義する。

---

## 9. 未知fieldの基本扱い

未知fieldは、HEMP Core、Protobuf Encoding specification、または対応する Body Contract が、現在の受信側にとって既知の意味を定義しない field である。

HEMP Core semantics 上、未知fieldは意味として無視する。

未知fieldの存在のみを理由に HEMP failure としてはならない。

未知fieldは、必須既知 field の代替にはならない。

HEMP 実装は、相手が解釈する必要のある情報を未知fieldとして送ってはならない。

未知fieldは HEM Payload bytes の一部であり、Payload length、受信 Payload size 上限、protocol channel Payload length 上限、agreement 済み `body.limits` に基づく上限判定の対象から除外しない。

Protobuf unknown fields の具体的な扱いは、`specs/proto-encoding/` で定義する。

---

## 10. この文書で扱わないもの

この文書では、次を定義しない。

```text
- direction / role / channel / thread / seq / end / close / abort の詳細規則
- end / close / abort の有効な組み合わせ
- HEM Timeline
- post / reply / notice の flow semantics
- Channel Table / digest
- agreement body / limits body の詳細
- protocol channel body の詳細
- Standard Data Body の詳細
- failure response policy の詳細
- Protobuf schema の詳細
- Transport Binding の詳細
```
