# Channel Table / digest

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Core protocol specification

---

## 1. この文書の目的

この文書は、HEMP における Channel Table と digest を定義する。

この文書では、channel の通信上の性質と digest 計算対象を定義する。

主に次を扱う。

```text
- Channel Table
- Protocol Channel Table
- Application Channel Table
- Channel Entry
- channel name
- service provider side
- channel flags
- disabled channel
- entry flags byte
- Entry byte sequence
- canonical channel table byte sequence
- protocol digest
- application digest
- digest agreement rule
```

protocol channel の Header 固定値、Body Contract、使用可能タイミング、固有 flow rule は、`07-protocol-channels.md` で定義する。

Standard Data Body の詳細な data body rules は、`08-data-body-rules.md` で定義する。

---

## 2. Channel Table

Channel Table は、channel の通信上の性質を定義する Table である。

Channel Table は、digest の計算対象である canonical byte sequence の元になる。

Channel Table の各 Entry は、HEM Timeline、notice、chunking、HEM Thread の使用可否を定義する。

Channel Table は、HEM Body の具体的な Body Contract や Body schema を直接定義するものではない。

---

## 3. Channel Table の種類

Channel Table には、次の2種類がある。

```text
Protocol Channel Table
Application Channel Table
```

Protocol Channel Table は、HEMP が定義する protocol channel の Channel Table である。

Application Channel Table は、HEMP を利用する application が定義する application channel の Channel Table である。

---

## 4. Channel Entry 共通要素

各 Channel Entry は、次を持つ。

```text
channel name
service provider side
timeline banned flag
notice banned flag
chunking banned flag
multi-thread banned flag
```

この文書では Channel Entry の構造を定義する。

各 protocol channel の body、使用可能タイミング、固有 flow rule は、`07-protocol-channels.md` で定義する。

---

## 5. channel name

channel name は strict lower_snake_case string とする。

形式は次の通りである。

```regex
^[a-z][a-z0-9]*(?:_[a-z0-9]+)*$
```

許可文字は次の通りである。

```text
a-z
0-9
_
```

先頭文字は `a-z` でなければならない。

比較は case-sensitive exact match とする。

受信側は、小文字化、trim、空白除去、ハイフン変換、その他の表記補正を行わない。

canonical byte sequence 生成時、channel name は ASCII bytes として扱う。

Protocol Channel Table では、channel name は protocol channel name を指す。

Application Channel Table では、channel name は application channel name を指す。

同一 Channel Table 内で、channel name は一意でなければならない。

---

## 6. service provider side

service provider side の有効値は次の通りである。

```text
host
engine
```

service provider side は、HEM Timeline における post / reply と、notice の許可 direction を導く。

```text
service provider side = engine:
  post   = to_engine / post
  reply  = to_host   / reply
  notice = to_host   / notice

service provider side = host:
  post   = to_host   / post
  reply  = to_engine / reply
  notice = to_engine / notice
```

flow 上の適用規則は、`04-flow-semantics.md` で定義する。

---

## 7. channel flags

Channel Entry は、次の channel flags を持つ。

```text
timeline banned flag
notice banned flag
chunking banned flag
multi-thread banned flag
```

### 7.1 timeline banned flag

`timeline banned flag = true` の場合、その channel では HEM Timeline を許可しない。

つまり、その channel では post / reply 型通信を使用できない。

`timeline banned flag = false` の場合、その channel では service provider side に基づいて HEM Timeline を許可する。

`timeline banned flag = true` の channel で post または reply を受信した場合、HEMP flow violation とする。

### 7.2 notice banned flag

`notice banned flag = true` の場合、その channel では notice を許可しない。

`notice banned flag = false` の場合、その channel では service provider side に基づいて notice を許可する。

`notice banned flag = true` の channel で notice を受信した場合、HEMP flow violation とする。

### 7.3 chunking banned flag

`chunking banned flag = true` の場合、その channel では論理 post 送信、論理 reply 送信、論理 notice 送信の chunking を許可しない。

その channel 上の各論理送信は、1 HEM で完結しなければならない。

`chunking banned flag = true` の channel では、post / reply / notice のいずれも `end = true` でなければならない。

`chunking banned flag = true` の channel で `end = false` の HEM を受信した場合、HEMP flow violation とする。

`chunking banned flag = false` の場合、複数 HEM の論理送信を許可する。

application channel において `chunking banned flag = false` の場合、複数 HEM の論理送信は agreement 済み `body.limits` の範囲内で許可する。

protocol channel において `chunking banned flag = false` の場合、その具体的な扱いは各 protocol channel の規則が定義する。

### 7.4 multi-thread banned flag

`multi-thread banned flag = true` の場合、その channel の HEM Timeline では複数の HEM Thread を使用できない。

つまり、その channel の HEM Timeline では `thread = 1` のみを使用する。

`multi-thread banned flag = false` の場合、その channel の HEM Timeline では複数の HEM Thread を使用できる。

application channel において `multi-thread banned flag = false` の場合、複数の未closed状態の HEM Thread は agreement 済み `body.limits.open_threads.per_channel` と `body.limits.open_threads.total` の範囲内で許可する。

protocol channel において `multi-thread banned flag = false` の場合、その具体的な扱いは各 protocol channel の規則が定義する。

`multi-thread banned flag` は notice の thread ルールには影響しない。

`multi-thread banned flag = true` の channel で、HEM Timeline 用の post または reply に `thread != 1` を使った場合、HEMP flow violation とする。

---

## 8. disabled channel

次の組み合わせは disabled channel とする。

```text
timeline banned flag = true
notice banned flag = true
```

disabled channel は、Channel Table 内に存在するが、HEM Timeline / notice のいずれにも使用できない。

disabled channel の channel 番号を使った HEM を受信した場合、HEMP flow violation とする。

disabled channel でも chunking banned flag と multi-thread banned flag は定義され、digest の対象に含まれる。

ただし、disabled channel では論理送信が許可されないため、chunking banned flag と multi-thread banned flag は通信時には適用されない。

Application Channel Table では、disabled application channel を定義してよい。

この仕様が定義する Protocol Channel Table には、disabled protocol channel は存在しない。

---

## 9. entry flags byte

entry flags byte は次の通りである。

```text
entry flags byte = 0x80 | entry flags
```

有効 byte 範囲は次の通りである。

```text
0x80 ～ 0x9F
```

bit 割り当ては次の通りである。

```text
bit 0 / mask 0x01:
  service provider side flag
  0 = host
  1 = engine

bit 1 / mask 0x02:
  timeline banned flag
  0 = timeline is not banned
  1 = timeline is banned

bit 2 / mask 0x04:
  notice banned flag
  0 = notice is not banned
  1 = notice is banned

bit 3 / mask 0x08:
  chunking banned flag
  0 = chunking is not banned
  1 = chunking is banned

bit 4 / mask 0x10:
  multi-thread banned flag
  0 = multi-thread is not banned
  1 = multi-thread is banned

bit 5 / mask 0x20:
  0 fixed

bit 6 / mask 0x40:
  0 fixed

bit 7 / mask 0x80:
  1 fixed
```

例:

```text
service provider side = host
timeline banned = false
notice banned = false
chunking banned = false
multi-thread banned = false
entry flags byte = 0x80
```

```text
service provider side = engine
timeline banned = false
notice banned = false
chunking banned = false
multi-thread banned = false
entry flags byte = 0x81
```

```text
service provider side = engine
timeline banned = true
notice banned = false
chunking banned = false
multi-thread banned = false
entry flags byte = 0x83
```

```text
service provider side = engine
timeline banned = true
notice banned = true
chunking banned = true
multi-thread banned = true
entry flags byte = 0x9F
```

---

## 10. Entry byte sequence

1つの Channel Entry は、次の byte 列に変換する。

```text
ASCII(channel name)
+ 0xFD
+ entry flags byte
```

`0xFD` は channel name terminator である。

---

## 11. Entry separator / Table terminator

Marker は次の通りである。

```text
0xFD = channel name terminator
0xFE = Entry separator
0xFF = Table terminator
```

Table terminator である `0xFF` は、canonical byte sequence の末尾に必ず付ける。

Entry が複数ある場合は次の形になる。

```text
entry_1
+ 0xFE
+ entry_2
+ 0xFE
+ entry_3
+ 0xFF
```

最後の Entry の後に `0xFE` は置かない。

---

## 12. Protocol Channel Table

Protocol Channel Table は、HEMP が定義する protocol channel の Channel Table である。

Protocol Channel Table は、protocol digest の対象である。

Protocol Channel Table は、各 protocol channel の基本的な通信上の性質を定義する。

各 protocol channel の channel 番号、Header、Body Contract、使用可能タイミング、固有の flow rule は、`07-protocol-channels.md` で定義する。

Protocol Channel Table は、HEMP が定義する。

Protocol Channel Table は、ユーザーまたは HEMP を利用する application が定義する Table ではない。

Protocol Channel Table は、少なくとも agreement protocol channel を含む。

agreement protocol channel を持たない Protocol Channel Table は、Host-Engine Agreement を開始できないため、HEMP protocol として成立しない。

Protocol Channel Table には、この仕様が定義する protocol channel だけを Entry として載せる。

将来予約分は Protocol Channel Table の Entry には含めず、channel または protocol channel space の説明で将来用に予約する。

---

## 13. Protocol Channel Entry

この仕様が定義する Protocol Channel Table は、次の Entry を持つ。

```text
agreement:
  service provider side = engine
  timeline banned flag = false
  notice banned flag = true
  chunking banned flag = true
  multi-thread banned flag = true

ping:
  service provider side = engine
  timeline banned flag = false
  notice banned flag = true
  chunking banned flag = true
  multi-thread banned flag = true

shutdown:
  service provider side = engine
  timeline banned flag = false
  notice banned flag = true
  chunking banned flag = true
  multi-thread banned flag = true

cancel:
  service provider side = engine
  timeline banned flag = false
  notice banned flag = true
  chunking banned flag = true
  multi-thread banned flag = false
```

各 protocol channel の Header 固定値、Body Contract、使用可能タイミング、固有 flow rule は、`07-protocol-channels.md` で定義する。

---

## 14. Protocol Channel Table の canonical Entry order

Protocol Channel Table の canonical Entry order は、各 protocol channel の channel 番号に基づいて決める。

channel 番号は、canonical Entry order を決めるために参照する。

Protocol Channel Table の canonical Entry order は、protocol channel number の絶対値が小さい順とする。

順序は次の通りである。

```text
agreement
ping
shutdown
cancel
```

---

## 15. canonical protocol channel table byte sequence

canonical protocol channel table byte sequence は、Protocol Channel Table の Entry を canonical Entry order に従って連結した byte 列である。

この仕様が定義する canonical protocol channel table byte sequence は次の通りである。

```text
ASCII("agreement")
+ 0xFD
+ 0x9D
+ 0xFE
+ ASCII("ping")
+ 0xFD
+ 0x9D
+ 0xFE
+ ASCII("shutdown")
+ 0xFD
+ 0x9D
+ 0xFE
+ ASCII("cancel")
+ 0xFD
+ 0x8D
+ 0xFF
```

`body.digest.protocol.digest` の具体的な SHA-256 値は、この仕様では固定値として記載しない。

`body.digest.protocol.digest` は、canonical protocol channel table byte sequence から SHA-256 で計算する。

---

## 16. Protocol Channel Table と Core Header field

Protocol Channel Table は Channel Entry の集合であり、Core Header field そのものを含まない。

すべての protocol channel HEM は、Core Header field set を使用する。

`abort` は Protocol Channel Table の Entry ではなく、Core Header field である。

`abort` field の存在は、canonical protocol channel table byte sequence に含めない。

`abort` field の追加だけでは、canonical protocol channel table byte sequence は変化しない。

protocol channel HEM における `abort` の使用可否は、`07-protocol-channels.md` で定義する。

各 protocol channel が明示的に許可しない限り、protocol channel HEM の `abort` は `false` でなければならない。

明示的に許可されていない protocol channel HEM で `abort = true` を受信した場合、HEMP flow violation とする。

---

## 17. Application Channel Table

Application Channel Table は、HEMP を利用する application が定義する application channel の Channel Table である。

Application Channel Table は、application digest の対象である。

Application Channel Table は、HEMP を利用する application が定義する。

Application Channel Entry の具体的な一覧は、HEMP Core では定義しない。

HEMP Core は、Application Channel Table の Entry 構造、canonical Entry order、canonical byte sequence、digest 計算方法だけを定義する。

---

## 18. Application Channel Table と channel 番号

agreement 成立後、`channel > 0` の値は、agreement 済み Application Channel Table の canonical Entry order に基づく 1-based reference として解釈する。

```text
canonical order entry 1 -> channel = 1
canonical order entry 2 -> channel = 2
canonical order entry 3 -> channel = 3
```

`channel > 0` の値が、agreement 済み Application Channel Table の Entry 数を超える場合、HEMP flow violation とする。

---

## 19. Application Channel Table が0件の場合

Application Channel Table は0件でもよい。

Application Channel Table が0件の場合、canonical application channel table byte sequence は `0xFF` のみとする。

Application Channel Table が0件の場合、application channel は存在しない。

この状態で `channel > 0` を使った HEM を受信した場合、HEMP flow violation とする。

`body.digest.application.digest` は、この `0xFF` だけの byte sequence から SHA-256 で計算する。

---

## 20. Application Channel Table の canonical Entry order

Application Channel Table の canonical Entry order は次の通りである。

```text
application channel name の ASCII bytes による lexicographic ascending order
```

定義ファイル上の記載順は digest 計算に使わない。

この canonical Entry order は、`channel > 0` の 1-based reference にも使う。

---

## 21. canonical application channel table byte sequence

canonical application channel table byte sequence は、Application Channel Table の Entry を canonical Entry order に従って連結した byte 列である。

canonical application channel table byte sequence は、application digest の対象である。

HEM Body の具体的な Body Contract や Body schema は、canonical application channel table byte sequence の対象ではない。

Entry が0件の場合、canonical application channel table byte sequence は次だけである。

```text
0xFF
```

Entry が複数ある場合は、この文書で定義した Entry byte sequence、Entry separator、Table terminator を使って連結する。

`body.digest.application.digest` の具体的な SHA-256 値は、この仕様では固定値として記載しない。

`body.digest.application.digest` は、canonical application channel table byte sequence から SHA-256 で計算する。

Standard Data Body は、HEMP protocol の標準 data body rules である。

ただし、Standard Data Body の body rules そのものは、Channel Table の canonical byte sequence には含めない。

Standard Data Body の詳細は、`08-data-body-rules.md` で定義する。

---

## 22. digest

`body.digest` は、Host と Engine が同じ Channel Table を認識しているかを確認するために使う。

`body.digest` は、同じ Table 名と同じ Table 内容を見ていることを確認する。

`body.digest` は次を持つ。

```text
protocol
application
```

`body.digest.protocol` は、protocol digest である。

`body.digest.application` は、application digest である。

---

## 23. digest fields

`body.digest.protocol` と `body.digest.application` は、それぞれ次を持つ。

```text
name
digest
```

### 23.1 digest.name

`digest.name` は、確認対象となる Channel Table の安定名である。

`digest.name` は strict lower_snake_case ASCII string とする。

有効形式は次である。

```regex
^[a-z][a-z0-9]*(?:_[a-z0-9]+)*$
```

比較は case-sensitive exact match とする。

受信側は、小文字化、trim、空白除去、ハイフン変換、その他の表記補正を行わない。

### 23.2 digest.digest

`digest.digest` は、対応する canonical byte sequence から計算した raw digest bytes である。

Digest algorithm は次の通りである。

```text
SHA-256
```

`digest.digest` の有効な長さは次の通りである。

```text
32 bytes
```

encoding 上の具体表現は、Protobuf Encoding specification で定義する。

---

## 24. protocol digest

`body.digest.protocol` は、Host と Engine が同じ Protocol Channel Table を認識しているかを確認するために使う。

`body.digest.protocol.name` は、確認対象となる Protocol Channel Table の安定名である。

`body.digest.protocol.name` は次の値とする。

```text
hemp_protocol_channels
```

この仕様の v1.0.0 では、`body.digest.protocol.name` は `hemp_protocol_channels` に固定する。

別の protocol digest name を使う通信は、この仕様の v1.0.0 における Host-Engine Agreement としては成立しない。

`body.digest.protocol.digest` は、canonical protocol channel table byte sequence から計算した raw SHA-256 digest bytes である。

`abort` は Protocol Channel Table Entry ではなく Core Header field である。

そのため、`abort` field の存在は canonical protocol channel table byte sequence に含めない。

---

## 25. application digest

`body.digest.application` は、Host と Engine が同じ Application Channel Table を認識しているかを確認するために使う。

`body.digest.application.name` は、確認対象となる Application Channel Table の安定名である。

`body.digest.application.name` の具体値は HEMP Core では定義しない。

HEMP を利用する application が、自分たちの Host / Engine で使う Application Channel Table に対して具体値を定義する。

`body.digest.application.digest` は、canonical application channel table byte sequence から計算した raw SHA-256 digest bytes である。

---

## 26. digest agreement rule

digest agreement は、Host と Engine の `name` および `digest` がともに exact match した場合だけ成立する。

protocol digest agreement は、次の両方が一致した場合だけ成立する。

```text
Host body.digest.protocol.name   == Engine body.digest.protocol.name
Host body.digest.protocol.digest == Engine body.digest.protocol.digest
```

application digest agreement は、次の両方が一致した場合だけ成立する。

```text
Host body.digest.application.name   == Engine body.digest.application.name
Host body.digest.application.digest == Engine body.digest.application.digest
```

---

## 27. この文書で扱わないもの

この文書では、次を定義しない。

```text
- HEM frame / HEM Length Header の構造
- HEM Payload encoding の詳細
- Core Header field の個別 validation
- HEM Thread lifecycle / HEM Timeline の詳細
- post / reply / notice flow の詳細
- agreement body / limits body の詳細
- protocol channel body の詳細
- Standard Data Body の詳細
- failure response policy の詳細
- Transport Binding の具体仕様
```
