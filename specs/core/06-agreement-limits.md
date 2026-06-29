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

同じ HEMP session 内で、再 Agreement または再交渉は行わない。

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

これらは、Transport Binding または Host 側のプロセス管理が扱う。

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
  limits.hem_payload_length = 16777216
  limits.open_threads.per_channel = 64
  limits.open_threads.total = 256
  limits.role.post.hem_count = 1024
  limits.role.post.payload_total_length = 268435456
  limits.role.reply.hem_count = 1024
  limits.role.reply.payload_total_length = 268435456
  limits.role.notice.hem_count = 256
  limits.role.notice.payload_total_length = 67108864
  limits.protocol.cancel.in_flight_requests = 16
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

この仕様では、`body.limits.protocol.cancel.in_flight_requests` を定義する。

protocol channel Payload length 上限は `body.limits` ではなく、HEMP Core が定義する protocol channel Payload length 上限である。

`body.limits` は digest の対象に含めない。

`body.limits` は Host-Engine Agreement における別の Agreement 項目として扱う。

limits の詳細は、この文書の後半で定義する。

---

## 16. agreement reply body

次は、agreement reply body の semantic field tree の例である。

この例に含まれる `message` の文字列は、固定値ではない。

`message` は人間向け説明であり、機械判定の主入力として使用しない。

```text
agreement reply body semantic field tree:
  version.protocol.result = true
  version.protocol.message = "Protocol version is accepted."
  version.application.host.result = true
  version.application.host.message = "Host application identity is accepted."
  version.application.engine.result = true
  version.application.engine.message = "Engine application identity is accepted."
  digest.protocol.result = true
  digest.protocol.message = "Protocol digest is accepted."
  digest.application.result = true
  digest.application.message = "Application digest is accepted."
  limits.hem_payload_length = true
  limits.open_threads.per_channel = true
  limits.open_threads.total = true
  limits.role.post.hem_count = true
  limits.role.post.payload_total_length = true
  limits.role.reply.hem_count = true
  limits.role.reply.payload_total_length = true
  limits.role.notice.hem_count = true
  limits.role.notice.payload_total_length = true
  limits.protocol.cancel.in_flight_requests = true
  limits.message = "Limits are accepted."
```

agreement reply body の具体的な encoding 表現は、Protobuf Encoding specification で定義する。

reply body 内の個別結果単位は次である。

```text
version.protocol
version.application.host
version.application.engine
digest.protocol
digest.application
limits
```

---

## 17. agreement reply result fields

`version.protocol`、`version.application.host`、`version.application.engine`、`digest.protocol`、`digest.application` は、HEMP が定義する必須 field として常に次を持つ。

```text
result
message
```

`result` は required semantic field である。

`result = false` は有効値であり、欠落とは区別する。

`result` は Agreement 成立 / failure の機械判定に使う。

`message` は required semantic field である。

`message` は、`result` が `true` でも `false` でも常に送る。

`message` は人間向け説明であり、機械分岐の主材料にはしない。

HEMP は、`agreement` の個別結果に `code` field を定義しない。

HEMP は、`agreement` の個別結果に `reason` field を定義せず、説明は `message` に統一する。

受信側は、`code` または `reason` が含まれる場合、それらを未知fieldとして無視する。

HEMP 実装は、`agreement` の個別結果に `code` または `reason` を送信しないことを推奨する。

---

## 18. limits reply result

`limits` は `result` field を持たない。

`limits` 内の判定 boolean leaf は、対応する limits 項目を Engine が受け入れ可能かどうかを表す。

`limits.message` は required semantic field である。

`limits.message` は、limits 全体についての人間向け説明である。

`limits.message` は、Agreement 成立 / failure の機械判定には使わない。

---

## 19. Agreement 成立条件

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

## 20. Agreement failure

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
- body.version.protocol が Semantic Versioning 2.0.0 として valid だが、受信側が対応しない。
- body.version.application.host が、受信側の想定 Host application identity と一致しない。
- body.version.application.engine が、受信側の想定 Engine application identity と一致しない。
- body.digest.protocol.name / digest が受信側の Protocol Channel Table と一致しない。
- body.digest.application.name / digest が受信側の Application Channel Table と一致しない。
- body.limits 内の数値 field は形式上 valid だが、受信側が受け入れ可能ではない。
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

## 21. Agreement Body Contract failure

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

## 22. limits の目的

`body.limits` は、HEMP session の運用上限を定義する。

`body.limits` には、application channel 上の通常 HEM 通信に対する上限と、HEMP が明示的に定義する protocol channel 固有の上限を含む。

`body.limits.hem_payload_length`、`body.limits.open_threads`、`body.limits.role` は、application channel 上の通常 HEM 通信に適用する。

このうち `body.limits.open_threads` は、application channel 上の未closed状態の HEM Thread 数に適用する。

未closed状態の HEM Thread には、open 状態および closing 状態の HEM Thread を含む。

application channel 上の通常 HEM 通信に対する `body.limits` の各上限は、`to_engine` と `to_host` の両方向の論理送信に対称に適用する。

`abort = true` の HEM も、application channel 上の通常 HEM 通信として `body.limits` の対象に含める。

Application Channel Table は application channel ごとの性質を定義する。

`body.limits.protocol` は、HEMP が定義する protocol channel 固有の上限に適用する。

この仕様では、`body.limits.protocol.cancel.in_flight_requests` を定義する。

---

## 23. limits post body の構造

次は、agreement post body の `limits` semantic field tree の例である。

この例に含まれる具体的な limits 数値は、v1.0.0 の固定 default 値ではない。

```text
limits semantic field tree:
  hem_payload_length = 16777216
  open_threads.per_channel = 64
  open_threads.total = 256
  role.post.hem_count = 1024
  role.post.payload_total_length = 268435456
  role.reply.hem_count = 1024
  role.reply.payload_total_length = 268435456
  role.notice.hem_count = 256
  role.notice.payload_total_length = 67108864
  protocol.cancel.in_flight_requests = 16
```

---

## 24. limits 数値field共通ルール

`body.limits` 内の数値fieldは、正の整数値でなければならない。

Encoding 上の numeric representation は、Protobuf Encoding specification が定義する。

次は不正とする。

```text
負数
0
範囲外
semantic presence がない
```

`body.limits` 内の各 field は必須である。

`body.limits` 内の数値 field が semantic presence を持たない、0である、またはこの文書が定義する値域に合わない場合、その agreement body は Agreement Body Contract failure とする。

`body.limits` 内の数値 field が形式上 valid であるが、受信側がその値を受け入れない場合、それは Agreement failure とする。

---

## 25. limits 数値fieldの範囲

```text
body.limits.hem_payload_length:
  1 ～ 4294967295

body.limits.open_threads.per_channel:
  1 ～ 4294967295

body.limits.open_threads.total:
  1 ～ 4294967295

body.limits.role.post.hem_count:
  1 ～ 4294967295

body.limits.role.post.payload_total_length:
  1 ～ 4294967295

body.limits.role.reply.hem_count:
  1 ～ 4294967295

body.limits.role.reply.payload_total_length:
  1 ～ 4294967295

body.limits.role.notice.hem_count:
  1 ～ 4294967295

body.limits.role.notice.payload_total_length:
  1 ～ 4294967295

body.limits.protocol.cancel.in_flight_requests:
  1 ～ 4294967295
```

---

## 26. limits 値同士の大小関係

`body.limits` 内の各数値field同士に、HEMP としての大小関係制約は置かない。

すべての limits 値は上限値であり、複数の上限が同時に適用される場合は、最初に到達した上限によって制限される。

たとえば、次は不正ではない。

```text
role.post.payload_total_length が hem_payload_length より小さい。

open_threads.total が open_threads.per_channel より小さい。
```

---

## 27. hem_payload_length

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

## 28. open_threads

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

## 29. role.post / role.reply / role.notice

`body.limits.role.*.hem_count` は、1つの論理送信を構成できる HEM 数の上限である。

`body.limits.role.*.payload_total_length` は、1つの論理送信に含まれる全 HEM Payload length の合計上限である。

ここでいう HEM Payload length は、HEM Length Header が示す Payload length であり、HEM Length Header 自身の 4 bytes は含めない。

HEM Payload encoding において、ここでいう HEM Payload length は encoded HEM Payload bytes の長さである。

対象は、application channel 上の次の論理送信である。

```text
論理 post 送信
論理 reply 送信
論理 notice 送信
```

HEM Timeline 全体ではなく、post 側の論理送信と reply 側の論理送信を別々に数える。

`abort = true` の HEM も、対応する role の `hem_count` に数える。

`abort = true` の HEM Payload length も、対応する role の `payload_total_length` に含める。

application channel において `chunking banned flag = true` の場合、`body.limits.role.*.hem_count` の値に関係なく、各論理送信は1 HEMで完結しなければならない。

application channel において `chunking banned flag = true` の場合でも、`body.limits.hem_payload_length` と `body.limits.role.*.payload_total_length` は適用する。

application channel において `chunking banned flag = false` の場合、`body.limits.role.*.hem_count` に従って複数 HEM の論理送信を許可する。

---

## 30. protocol.cancel.in_flight_requests

`body.limits.protocol.cancel.in_flight_requests` は、Host が同時に未完了にできる cancel request 数の上限である。

未完了 cancel request とは、Host が `channel = -4` の cancel post を送信し、対応する cancel reply が完了していない cancel request を指す。

`body.limits.protocol.cancel.in_flight_requests` は、`channel = -4` の cancel request 用 HEM Thread に適用する。

`body.limits.protocol.cancel.in_flight_requests` は、application channel 上の未closed状態の HEM Thread 数には含めない。

Agreement 成立後、Host が `body.limits.protocol.cancel.in_flight_requests` を超えて新しい cancel request を開始した場合、HEMP flow violation とする。

cancel protocol channel の詳細は、`07-protocol-channels.md` で定義する。

---

## 31. この文書で扱わないもの

この文書では、次を定義しない。

```text
- ping protocol channel の詳細
- shutdown protocol channel の詳細
- cancel protocol channel の詳細
- protocol channel 全体の一覧説明
- HEM frame / HEM Length Header の構造
- Core Header field の個別定義
- HEM Thread lifecycle / HEM Timeline 全体
- Channel Table canonical byte sequence
- Standard Data Body の詳細
- failure response policy の詳細
- Transport Binding の具体仕様
```
