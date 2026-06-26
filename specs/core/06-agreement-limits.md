# agreement / limits

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Core protocol specification

---

## 1. この文書の目的

この文書は、HEMP における Host-Engine Agreement と limits を定義する。

この文書では、agreement protocol channel、Agreement 成立前 / 成立後の基本 flow、agreement post / reply、agreement Body Contract、`body.version`、`body.digest`、`body.limits`、Agreement 成立条件、Agreement failure、Agreement Body Contract failure、および limits を扱う。

`ping`、`shutdown`、`cancel` の protocol channel 詳細は、`07-protocol-channels.md` で定義する。

---

## 2. Host-Engine Agreement

Host と Engine は、通常 HEMP 通信を始める前に Host-Engine Agreement を行う。

Host-Engine Agreement は、protocol version、application identity、digest、limits の一致または受け入れ可否を確認するために行う。

Host-Engine Agreement が成立した後、application channel を使った通常 HEM 通信へ進むことができる。

同じ HEMP session 内で、再 Agreement または再 negotiation は行わない。

再度 Agreement を行う場合は、新しい HEMP session として開始する。

---

## 3. agreement protocol channel

`agreement` は、開始時 Host-Engine Agreement 用の protocol channel である。

```text
channel = -1
```

`agreement` は、Protocol Channel Table の Entry である。

`agreement` は、Application Channel Table の Entry ではない。

---

## 4. agreement で許可される direction / role

`agreement` で許可される direction / role の組み合わせは次の通りである。

```text
to_engine / post
to_host   / reply
```

次の組み合わせは不正である。

```text
to_host   / post
to_engine / reply
notice
```

不正な direction / role の組み合わせを `agreement` で受信した場合、HEMP flow violation とする。

---

## 5. agreement の HEM 構成

`agreement` は、1つの post HEM と 1つの reply HEM で完結する。

`agreement` では chunking を許可しない。

`agreement` では sender-side logical-send abort を使用しない。

`agreement` の `abort` は常に `false` とする。

`agreement` で `abort = true` を受信した場合、HEMP flow violation とする。

---

## 6. agreement post header

`agreement` post の Core Header field は、次の固定値を持つ。

```text
agreement post Core Header semantic field set:
  direction = to_engine
  role      = post
  channel   = -1
  thread    = 1
  seq       = 1
  end       = true
  close     = true
  abort     = false
```

---

## 7. agreement reply header

`agreement` reply の Core Header field は、次の固定値を持つ。

```text
agreement reply Core Header semantic field set:
  direction = to_host
  role      = reply
  channel   = -1
  thread    = 1
  seq       = 2
  end       = true
  close     = true
  abort     = false
```

`agreement` reply 完了後、`channel = -1 / thread = 1` は `04-flow-semantics.md` の HEM Thread lifecycle に従って closed になる。

---

## 8. Agreement 成立前 / 成立後の flow

### 8.1 起動直後

Host が最初の有効 HEM を送る。

最初の有効 HEM は、`channel = -1 / role = post / direction = to_engine` の `agreement` でなければならない。

Engine は、`agreement` post を受け取るまで、自発的に HEMP HEM を送らない。

Engine が HEMP 通信可能であることは、`agreement` reply によって確認する。

Engine process が起動したか、transport 接続可能になったかは、HEMP Core の責務ではない。

これらは、Transport Binding または Host 側の process 管理が扱う。

### 8.2 Agreement 成立前

Agreement 成立前は、`agreement` のみを許可する。

Agreement 成立前に `agreement` 以外の有効 HEM を受信した場合、HEMP flow violation とする。

ここでいう有効 HEM とは、HEMP framing failure、HEMP payload format failure、HEMP header validation failure には該当せず、HEM として解釈可能なものを指す。

### 8.3 Agreement 成立後

Agreement 成立後、application channel を使った通常 HEM 通信へ進むことができる。

Agreement 成立後、`channel > 0` の値は、Agreement 済み Application Channel Table の canonical Entry order に基づく 1-based reference として解釈する。

Agreement 成立後は、Agreement 成立後に許可される protocol channel も使用できる。

Agreement 成立後に使用できる protocol channel の詳細は、`07-protocol-channels.md` で定義する。

Agreement 成立後に Protocol Channel Table に存在しない protocol channel を受信した場合、HEMP flow violation とする。

Agreement 成立後は、`agreement` を再度使用してはならない。

Agreement 成立後に `agreement` を再度受信した場合、HEMP flow violation とする。

---

## 9. agreement body 内の未知field

agreement body 内で、agreement Body Contract が定義しない field は、未知fieldとして扱う。

受信側は、agreement body 内の未知fieldを意味として無視する。

agreement body 内の未知fieldの存在のみを理由に、HEM Body Contract failure としてはならない。

agreement body 内の未知fieldは、Agreement 成立条件に含めない。

未知fieldは、agreement body 内の必須既知 field の代替にはならない。

HEMP 実装は、agreement body 内に未知fieldを送信しないことを推奨する。

Protobuf unknown fields の具体的な扱いは、`specs/proto-encoding/` で定義する。

---

## 10. agreement post body

次は、agreement post body の semantic field tree の例である。

この例に含まれる具体的な limits 数値は、v1.0.0 の固定 default 値ではない。

実際に提示する limits 値は、この文書が定義する値域と制約を満たす範囲で、Host が提示する。

ただし、`version.protocol` の値は、HEMP protocol version string としてこの文書の version 規則に従う。

```text
agreement post body semantic field tree:
  version.protocol = "1.0.0"
  version.application.host.name = "..."
  version.application.host.version = "..."
  version.application.engine.name = "..."
  version.application.engine.version = "..."
  digest.protocol.name = "hemp_protocol_channels"
  digest.protocol.digest = <32 raw SHA-256 bytes>
  digest.application.name = "..."
  digest.application.digest = <32 raw SHA-256 bytes>
  limits.hem_payload_length = <positive integer>
  limits.open_threads.per_channel = <positive integer>
  limits.open_threads.total = <positive integer>
  limits.role.post.hem_count = <positive integer>
  limits.role.post.payload_total_length = <positive integer>
  limits.role.reply.hem_count = <positive integer>
  limits.role.reply.payload_total_length = <positive integer>
  limits.role.notice.hem_count = <positive integer>
  limits.role.notice.payload_total_length = <positive integer>
  limits.protocol.cancel.in_flight_requests = <positive integer>
```

agreement post body は、HEMP が定義する必須 field として次を持つ。

```text
version
digest
limits
```

HEM Payload 上の path としては、それぞれ次である。

```text
body.version
body.digest
body.limits
```

agreement post body の具体的な encoding 表現は、Protobuf Encoding specification で定義する。

---

## 11. body.version

`body.version` は、version 関連 Agreement 情報をまとめる構造である。

`body.version` は、HEMP が定義する必須 field として次を持つ。

```text
protocol
application
```

---

## 12. body.version.protocol

`body.version.protocol` は、HEMP protocol version である。

```text
semantic type: string
initial value: "1.0.0"
format: Semantic Versioning 2.0.0 version string
```

Agreement では、Semantic Versioning 2.0.0 として有効な文字列を、正規化せず case-sensitive exact match によって比較する。

HEMP は、次を行わない。

```text
version range
compatibility negotiation
大小比較による互換判定
"v" prefix除去
"1.0" から "1.0.0" への補完
trim
その他の正規化
```

HEMP protocol version `1.0.0` の Core Header field は、`abort` を含む8 fieldである。

`abort` は optional capability ではないため、Agreement body に `abort` 用の capability field は定義しない。

---

## 13. body.version.application

`body.version.application` は、Host 側 application identity と Engine 側 application identity のペアを表す。

`body.version.application` は、HEMP が定義する必須 field として次を持つ。

```text
host
engine
```

各 application identity は、HEMP が定義する必須 field として次を持つ。

```text
name
version
```

`application.*.name` は string とする。

`application.*.version` は string とする。

`application.host` と `application.engine` を相互に一致比較しない。

Engine 側は次を確認する。

```text
body.version.application.host が、Engine 側の想定 Host application identity と一致する。
body.version.application.engine が、Engine 自身の application identity と一致する。
```

`name` と `version` の比較は case-sensitive exact match とする。

正規化は行わない。

HEMP は application version string を SemVer、version range、compatibility expression、数値として解釈しない。

---

## 14. body.digest

`body.digest` は、Host と Engine が同じ Channel Table を認識しているかを確認するための Agreement 情報をまとめる構造である。

`body.digest` は、HEMP が定義する必須 field として次を持つ。

```text
protocol
application
```

`body.digest.protocol` は、Protocol Channel Table の一致確認に使う。

`body.digest.application` は、Application Channel Table の一致確認に使う。

digest の意味と比較ルールは、`05-channel-table-digest.md` で定義する。

---

## 15. body.limits

`body.limits` は、HEMP session の運用上限を表す構造である。

`body.limits` には、application channel 上の通常 HEM 通信に対する上限と、HEMP が明示的に定義する protocol channel 固有の上限を含む。

`body.limits.hem_payload_length`、`body.limits.open_threads`、`body.limits.role` は、application channel 上の通常 HEM 通信に適用する。

このうち `body.limits.open_threads` は、application channel 上の未closed状態の HEM Thread 数に適用する。

未closed状態の HEM Thread には、open 状態および closing 状態の HEM Thread を含む。

`body.limits.protocol` は、HEMP が定義する protocol channel 固有の上限に適用する。

---

## 16. agreement reply body

次は、agreement reply body の semantic field tree の例である。

この例に含まれる `message` の文字列は、固定値ではない。

`message` は人間向け説明であり、機械判定の主入力として使用しない。

```text
agreement reply body semantic field tree:
  version.protocol.result = true | false
  version.protocol.message = "..."
  version.application.host.result = true | false
  version.application.host.message = "..."
  version.application.engine.result = true | false
  version.application.engine.message = "..."
  digest.protocol.result = true | false
  digest.protocol.message = "..."
  digest.application.result = true | false
  digest.application.message = "..."
  limits.<leaf>.result = true | false
  limits.<leaf>.message = "..."
```

agreement reply body は、Agreement 成立条件の各項目について、判定結果を返す。

各判定結果は、HEMP が定義する必須 semantic field として次を持つ。

```text
result
message
```

`result = false` は有効値であり、欠落とは区別する。

`message` は人間向け説明であり、空文字列であってはならない。

---

## 17. Agreement 成立条件

Host-Engine Agreement は、agreement reply body が Agreement reply Body Contract として valid であり、かつ次をすべて満たす場合だけ成立する。

```text
body.version.protocol.result == true
body.version.application.host.result == true
body.version.application.engine.result == true
body.digest.protocol.result == true
body.digest.application.result == true
body.limits 内の判定 boolean leaf がすべて true
```

agreement reply body が Agreement reply Body Contract として valid ではない場合、その HEM は Agreement failure ではなく、Agreement Body Contract failure または該当する HEMP failure classification に従う。

agreement reply body が valid であり、かつ1つでも成立条件を満たさない場合は Agreement failure とする。

---

## 18. Agreement failure

Agreement body または agreement reply body が Body Contract として valid であり、かつ Agreement 成立条件を1つでも満たさない場合は Agreement failure とする。

Agreement failure は HEMP failure 5分類には含めない。

Agreement failure の不成立項目は次である。

```text
version.protocol
version.application.host
version.application.engine
digest.protocol
digest.application
limits
```

Agreement failure には、少なくとも次を含む。

```text
- body.version.protocol が Semantic Versioning 2.0.0 として valid だが、receiver が対応しない。
- body.version.application.host が、receiver 側の想定 Host application identity と一致しない。
- body.version.application.engine が、receiver 側の想定 Engine application identity と一致しない。
- body.digest.protocol.name / digest が receiver 側の Protocol Channel Table と一致しない。
- body.digest.application.name / digest が receiver 側の Application Channel Table と一致しない。
- body.limits 内の数値 field は形式上 valid だが、receiver が受け入れ可能ではない。
```

Agreement failure 時は次の通りとする。

```text
Engine は agreement reply を返す。
reply body で不成立項目を示す。
channel > 0 の通常 HEM 通信は開始しない。
現在の HEMP session は Agreement failure により終了したものとして扱う。
```

Agreement failure 後の処理、再接続、process の扱い、利用者向け表示、診断情報の扱いは、HEMP を使う application 側が扱う。

---

## 19. Agreement Body Contract failure

agreement body が Body Contract として valid な Agreement proposal になっていない場合は、Agreement Body Contract failure として扱う。

Agreement Body Contract failure には、少なくとも次を含む。

```text
- agreement body の必須 field が欠落している。
- agreement body 内の必須 known field の semantic presence がない。
- agreement body 内の数値 field が有効範囲外である。
- body.version.protocol が Semantic Versioning 2.0.0 として invalid である。
- body.digest.*.name が digest.name の有効形式に合わない。
- body.digest.*.digest が 32 bytes ではない。
- body.limits 内の数値 field が正の整数値ではない。
- body.limits 内の数値 field が有効範囲外である。
```

agreement Header validation および flow validation が成功し、agreement reply を返せる状態である場合、Engine は agreement reply body を使って Agreement Body Contract failure を返してよい。

この場合、Engine は Body Contract として valid ではない Agreement 項目に対応する Agreement 結果を failure として返してよい。

対応先は次の通りである。

```text
version.protocol:
  body.version.protocol.result = false

version.application.host:
  body.version.application.host.result = false

version.application.engine:
  body.version.application.engine.result = false

digest.protocol:
  body.digest.protocol.result = false

digest.application:
  body.digest.application.result = false

limits:
  対応する判定 boolean leaf = false
```

Agreement Body Contract failure に対して `result = false` または判定 boolean leaf `false` を返す場合、`message` に人間向け説明を入れる。

agreement Header validation または flow validation が成功していない場合、agreement reply を返してはならない。

agreement Header が有効でない場合、または agreement reply を安全に返せない場合の failure response policy は、`09-failure-handling.md` で定義する。

---

## 20. limits の基本validation

`body.limits` 内の数値 field は、semantic presence を持たなければならない。

`body.limits` 内の数値 field は、正の整数でなければならない。

`body.limits` 内の数値 field が semantic presence を持たない、0である、またはこの文書が定義する値域に合わない場合、その agreement body は Agreement Body Contract failure とする。

`body.limits` 内の数値 field が形式上 valid であるが、receiver がその値を受け入れない場合、それは Agreement failure とする。

---

## 21. hem_payload_length

`body.limits.hem_payload_length` は、application channel 上の通常 HEM 通信で許可する単体 HEM Payload length の上限である。

HEM Payload encoding において、`body.limits.hem_payload_length` は、encoded HEM Payload bytes 全体の上限である。

Agreement 成立後、application channel 上で encoded HEM Payload length が `body.limits.hem_payload_length` を超える場合、HEMP flow violation とする。

```text
encoded HEM Payload length > body.limits.hem_payload_length
→ HEMP flow violation
```

`body.limits.hem_payload_length` は、application body bytes または abort body bytes だけに直接適用する上限ではない。

Body Contract が使用できる body bytes capacity の導出方法は、適用される HEM Payload encoding specification が定義する。

Standard Data Body が採用する data bytes capacity は、`08-data-body-rules.md` および適用される HEM Payload encoding specification で定義する。

---

## 22. open_threads

`body.limits.open_threads.per_channel` は、1つの application channel 内で同時に未closed状態にできる HEM Thread 数の上限である。

`body.limits.open_threads.total` は、application channel 上で HEMP session 全体として同時に未closed状態にできる HEM Thread 数の上限である。

ここでいう未closed状態の HEM Thread には、open 状態および closing 状態の HEM Thread を含む。

closed 状態の HEM Thread は、`body.limits.open_threads.per_channel` および `body.limits.open_threads.total` の対象に含めない。

application channel において `multi-thread banned flag = true` の場合、`body.limits.open_threads.per_channel` の値に関係なく、その channel の HEM Timeline では `thread = 1` のみを使用する。

application channel において `multi-thread banned flag = false` の場合、`body.limits.open_threads.per_channel` と `body.limits.open_threads.total` に従って、複数の未closed状態の HEM Thread を許可する。

notice は HEM Thread を open しない。

notice-only channel で使われる notice 用 thread 番号は、`body.limits.open_threads.per_channel` / `body.limits.open_threads.total` の対象に含めない。

timeline を持つ application channel において、open 状態または closing 状態の HEM Thread に紐づけられた notice は、その HEM Thread が `open_threads` の対象であるかどうかを変更しない。

`open_threads.total` は、thread を open した側が Host か Engine かを問わず、すべての application channel 上の未closed状態の HEM Thread を合算して数える。

`body.limits.open_threads.per_channel` および `body.limits.open_threads.total` は、`channel = -4` の cancel request 用 HEM Thread には適用しない。

cancel request の同時数上限は、`body.limits.protocol.cancel.in_flight_requests` で定義する。

---

## 23. role limits

`body.limits.role.post.hem_count` は、1つの論理 post 送信で使用できる HEM 数の上限である。

`body.limits.role.reply.hem_count` は、1つの論理 reply 送信で使用できる HEM 数の上限である。

`body.limits.role.notice.hem_count` は、1つの論理 notice 送信で使用できる HEM 数の上限である。

`body.limits.role.post.payload_total_length` は、1つの論理 post 送信で使用できる encoded HEM Payload length 合計の上限である。

`body.limits.role.reply.payload_total_length` は、1つの論理 reply 送信で使用できる encoded HEM Payload length 合計の上限である。

`body.limits.role.notice.payload_total_length` は、1つの論理 notice 送信で使用できる encoded HEM Payload length 合計の上限である。

role limits は、application channel 上の通常 HEM 通信に適用する。

protocol channel の固定HEM数やPayload length上限は、各protocol channelおよび該当するCore規則で定義する。

---

## 24. cancel protocol limits

`body.limits.protocol.cancel.in_flight_requests` は、Host が同時に未完了にできる cancel request 数の上限である。

cancel request は、`channel = -4` の cancel post から対応 cancel reply 完了までを in-flight として数える。

この limit は、application channel の `open_threads` limit とは別に扱う。

---

## 25. limits の適用開始

`body.limits` は、Agreement 成立後に適用する。

Agreement 成立前は、通常 application channel 通信は開始しない。

Agreement 成立前の `agreement` HEM は、protocol channel Payload length 上限および agreement Body Contract に従う。

Agreement 成立後、application channel 上の通常 HEM 通信は、Agreement 済み `body.limits` に従わなければならない。

---

## 26. この文書で扱わないもの

この文書では、次を定義しない。

```text
- HEM Payload encoding の具体的な wire 表現
- body branch mapping
- Standard Data Body の chunking 詳細
- Transport Binding の payload max
- process 起動方法
- reconnect / retry policy
- application-specific Body Contract
```

HEM Payload encoding の具体的な表現は、`specs/proto-encoding/` で定義する。

Standard Data Body の詳細は、`08-data-body-rules.md` で定義する。

failure response policy の詳細は、`09-failure-handling.md` で定義する。
