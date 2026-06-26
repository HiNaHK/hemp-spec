# protocol channels

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Core protocol specification

---

## 1. この文書の目的

この文書は、HEMP が定義する protocol channel を定義する。

この文書では、protocol channel の位置づけ、protocol channel space、定義済み protocol channel、`ping`、`shutdown`、`cancel`、protocol channel body 内の未知field、protocol channel reply body 共通ルール、protocol channel と `abort` の関係を扱う。

`agreement` と `limits` の詳細は、`06-agreement-limits.md` で定義する。

---

## 2. protocol channel の位置づけ

protocol channel は、HEMP protocol 自体の制御に使う channel である。

protocol channel は、`channel < 0` の領域に属する。

protocol channel は HEMP が定義する。

protocol channel は Application Channel Table の Entry ではない。

protocol channel HEM も Core Header field set を使用する。

protocol channel HEM は、protocol channel Payload length 上限の対象である。

protocol channel Payload length 上限は、`02-hem-frame-payload-header.md` で定義する。

---

## 3. protocol channel space

この仕様が定義する protocol channel は次の通りである。

```text
channel = -1:
  agreement

channel = -2:
  ping

channel = -3:
  shutdown

channel = -4:
  cancel
```

未定義 negative channel は、将来の protocol channel 用に予約する。

未定義 protocol channel を使用した場合、HEMP flow violation とする。

---

## 4. protocol channel 共通 Header 規則

protocol channel HEM は Core Header field set を使用する。

protocol channel ごとの Header 固定値は、この文書または該当する protocol channel の定義文書で定義する。

protocol channel は、各 channel ごとの direction / role 制約に従う。

protocol channel は、各 channel ごとの chunking / thread / abort 制約に従う。

この仕様が定義する `ping`、`shutdown`、`cancel` は、Agreement 成立後に使用できる protocol channel である。

Agreement 成立前に使用できる protocol channel は `agreement` のみである。

Agreement 成立前 / 成立後の基本 flow は、`06-agreement-limits.md` で定義する。

この仕様が定義する `ping`、`shutdown`、`cancel` は、いずれも 1 HEM post + 1 HEM reply で完結する。

この仕様が定義する `ping`、`shutdown`、`cancel` では、chunking を許可しない。

---

## 5. protocol channel と abort

この仕様が定義する protocol channel は、sender-side logical-send abort を使用しない。

各 protocol channel が明示的に許可しない限り、protocol channel HEM の `abort` は `false` でなければならない。

明示的に許可されていない protocol channel HEM で `abort = true` を受信した場合、HEMP flow violation とする。

この規則は、`abort = true` の HEM は `end = true` でなければならないという Core Header validation rule を変更しない。

`abort` は Core Header field であり、Protocol Channel Table Entry ではない。

---

## 6. protocol channel body 内の未知field

protocol channel Body Contract が定義しない field は、未知fieldとして扱う。

受信側は、protocol channel body 内の未知fieldを意味として無視する。

protocol channel body 内の未知fieldの存在のみを理由に、HEM Body Contract failure としてはならない。

未知fieldは、protocol channel body 内の必須既知 field の代替にはならない。

HEMP 実装は、protocol channel body に未知fieldを送信しないことを推奨する。

Protobuf unknown fields の具体的な扱いは、`specs/proto-encoding/` で定義する。

---

## 7. protocol channel reply body 共通ルール

`ping`、`shutdown`、`cancel` の reply body は、HEMP が定義する必須 field として次を持つ。

```text
result
message
```

`result` は required semantic field である。

`result = false` は有効値であり、欠落とは区別する。

`result` は、protocol channel request を受け入れたかどうかを表す。

`result = false` は、Body Contract に合う request に対する通常の拒否だけでなく、Header validation および flow validation が成功した post の Body Contract failure を、該当 protocol channel の専用 reply body で返す場合にも使用してよい。

この場合の reply は、汎用 error reply ではない。

この場合の reply は、該当 protocol channel の Body Contract に基づく通常の reply body である。

Body Contract failure に対して reply を返す条件、state commit、および failure response policy は、`09-failure-handling.md` で定義する。

`message` は required semantic field である。

`message` は、`result = true` でも `result = false` でも常に送る。

`message` は人間向け説明であり、機械分岐の主材料にはしない。

`message` の空文字列は不正とする。

HEMP は、protocol channel reply body に `code` field を定義しない。

HEMP は、protocol channel reply body に `reason` field を定義せず、説明は `message` に統一する。

受信側は、protocol channel reply body に `code` または `reason` が含まれる場合、それらを未知fieldとして無視する。

HEMP 実装は、protocol channel reply body に `code` または `reason` を送信しないことを推奨する。

ただし、`shutdown` post body と `cancel` post body の `reason` は、要求側が送る理由として HEMP が定義する field であるため、この reply body 共通ルールとは別扱いとする。

---

## 8. agreement protocol channel

`agreement` は、開始時 Host-Engine Agreement 用の protocol channel である。

```text
channel = -1
```

`agreement` は、Protocol Channel Table の Entry である。

`agreement` は、Application Channel Table の Entry ではない。

`agreement` の Header、Body Contract、Agreement 成立条件、Agreement failure、Agreement Body Contract failure、および limits の詳細は、`06-agreement-limits.md` および `09-failure-handling.md` で定義する。

---

## 9. ping protocol channel

### 9.1 channel

```text
channel = -2
```

`ping` は、Agreement 成立後に相手が HEMP post を受信し、HEMP reply を返せる状態かを確認するための protocol channel である。

### 9.2 許可される direction / role

`ping` で許可される direction / role の組み合わせは次の通りである。

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

不正な direction / role の組み合わせを `ping` で受信した場合、HEMP flow violation とする。

### 9.3 header

`ping` は、1 HEM post + 1 HEM reply で完結する。

`ping` では chunking を許可しない。

`ping` では sender-side logical-send abort を使用しない。

`ping` の `abort` は常に `false` とする。

`ping` post の Core Header field は、次の固定値を持つ。

```text
ping post Core Header semantic field set:
  direction = to_engine
  role      = post
  channel   = -2
  thread    = 1
  seq       = 1
  end       = true
  close     = true
  abort     = false
```

`ping` reply の Core Header field は、次の固定値を持つ。

```text
ping reply Core Header semantic field set:
  direction = to_host
  role      = reply
  channel   = -2
  thread    = 1
  seq       = 2
  end       = true
  close     = true
  abort     = false
```

`ping` reply 完了後、`channel = -2 / thread = 1` は closed になる。

### 9.4 post body

`PingPostBody` は field を持たない empty body である。

`ping` post では、`ping_post` body branch が存在しなければならない。

known body branch が存在しない場合は、HEM Body Contract failure ではなく、HEMP payload format failure である。

受信側は、`ping` post body に未知fieldが含まれる場合、それらを意味として無視する。

### 9.5 reply body

`PingReplyBody.result` は presence required である。

`result = true` は、receiver が ping request を受け入れ、その時点で HEMP post を受信し、HEMP reply を返せる状態であることを表す。

`result = false` は有効値である。

`result = false` は、receiver が ping request を有効に受信し、ping reply を返したが、その ping request を肯定的な readiness confirmation としては受け入れないことを表す。

`result = false` は HEMP failure ではない。

`result = false` の ping reply を受信したことだけを理由に、HEMP session が正常終了、Agreement failure、HEMP failure、または transport failure になったものとして扱ってはならない。

`result = false` の ping reply を受信した後の再試行、待機、利用者向け表示、または application-level 判断は、HEMP を使う application または実装側が定義する。

`PingReplyBody.message` は presence required である。

`PingReplyBody.message` は non-empty でなければならない。

`ping` では nonce を定義しない。

---

## 10. shutdown protocol channel

### 10.1 channel

```text
channel = -3
```

`shutdown` は、Agreement 成立後に、HEMP session を終了できる状態か確認し、終了可能であれば正常終了を確定する protocol channel である。

`shutdown` は、HEMP session の終了準備を開始するための HEM ではない。

`shutdown` は、未完了の処理、open HEM Thread、進行中 HEM Timeline、または終了前に必要な通常 HEM 通信を完了させる機能を持たない。

### 10.2 Host-side shutdown-ready state

Host は、shutdown post を送信する前に、Host 側 application の責任で、この HEMP session 上で新しい通常 HEM 通信を開始しない状態にしなければならない。

Host-side shutdown-ready state は、Host が shutdown post を送信してよいと判断するための、Host 側から見た HEMP session 状態である。

Host は、Host が観測している HEMP session 上に未完了の HEM Timeline が残っている状態で、shutdown post を送信してはならない。

未完了の HEM Timeline とは、論理 post 送信が開始され、対応する論理 reply 送信が完了していない HEM Timeline である。

reply が `abort = true` で終端して HEM Timeline が完了した場合、その HEM Timeline は未完了の HEM Timeline には含めない。

post が `abort = true` で終端したが、対応 reply がまだ完了していない HEM Timeline は、未完了の HEM Timeline に含める。

未完了の HEM Timeline には、Host が開始した HEM Timeline と Engine が開始した HEM Timeline の両方を含む。

Host が開始した未完了の `ping` request または `cancel` request は、未完了の HEM Timeline に含まれる。

Host は、Host 側に送信すべき未完了の論理 post 送信、論理 reply 送信、論理 notice 送信、またはそれらの継続 HEM が残っている状態で、shutdown post を送信してはならない。

Host は、Host 側 application が返すべき reply、送るべき notice、または終了前に必要な通常 HEM 通信を残している状態で、shutdown post を送信してはならない。

Host-side shutdown-ready state は、Host が shutdown post を送信してよい状態を表す。

Host-side shutdown-ready state は、Engine 側 application が shutdown を受け入れることを保証しない。

### 10.3 許可される direction / role

`shutdown` で許可される direction / role の組み合わせは次の通りである。

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

不正な direction / role の組み合わせを `shutdown` で受信した場合、HEMP flow violation とする。

### 10.4 header

`shutdown` は、1 HEM post + 1 HEM reply で完結する。

`shutdown` では chunking を許可しない。

`shutdown` では sender-side logical-send abort を使用しない。

`shutdown` の `abort` は常に `false` とする。

`shutdown` post の Core Header field は、次の固定値を持つ。

```text
shutdown post Core Header semantic field set:
  direction = to_engine
  role      = post
  channel   = -3
  thread    = 1
  seq       = 1
  end       = true
  close     = true
  abort     = false
```

`shutdown` reply の Core Header field は、次の固定値を持つ。

```text
shutdown reply Core Header semantic field set:
  direction = to_host
  role      = reply
  channel   = -3
  thread    = 1
  seq       = 2
  end       = true
  close     = true
  abort     = false
```

### 10.5 post body

`shutdown` post body は、HEMP が定義する必須 field として `reason` を持つ。

```text
shutdown post body semantic fields:
  reason = ""
```

`reason` は required semantic field である。

`reason` の具体的な値、意味、分類、命名規則は HEMP Core では定義しない。

`reason` の内容は、HEMP を利用する application 側が決める。

`reason` の空文字列は有効とする。

### 10.6 reply body

`shutdown` reply body は、HEMP が定義する必須 field として `result` と `message` を持つ。

```text
shutdown reply body semantic fields:
  result = true
  message = "Shutdown is accepted."
```

Engine は、Body Contract に合う shutdown post を受信した場合、Engine 側 application に、この HEMP session を終了可能か確認する。

Engine は、Engine 側に未完了の論理 post 送信、論理 reply 送信、論理 notice 送信、またはそれらの継続 HEM が残っている状態で、`result = true` の shutdown reply を送信してはならない。

`abort = true` で終端済みの論理送信は、未完了の論理送信ではない。

ただし、post が `abort = true` で終端した後に対応 reply が完了していない HEM Timeline は、未完了の HEM Timeline として扱う。

この条件は、Engine 側 application の終了可否判断ではなく、HEMP protocol state に対する安全不変条件である。

Engine 側 application がこの HEMP session を終了可能と判断し、かつ Engine 側に未完了の論理 post 送信、論理 reply 送信、論理 notice 送信、またはそれらの継続 HEM が残っていない場合、Engine は shutdown reply body の `result` を `true` とする。

`result = true` は、Engine 側 application がこの HEMP session を終了可能と判断し、かつ Engine 側 HEMP 実装が HEMP protocol state として正常終了可能と判断したことを表す。

Host が `shutdown` reply body の `result = true` を受信した時点で、その HEMP session は正常終了したものとして扱う。

正常終了した HEMP session では、以後、新しい HEM を送信してはならない。

正常終了後に再度 HEMP 通信を行う場合は、新しい HEMP session として開始し、再び `agreement` から実行しなければならない。

Engine 側 application がこの HEMP session を終了可能ではない、または終了を拒否すると判断した場合、Engine は shutdown reply body の `result` を `false` とする。

Engine 側 application がこの HEMP session を終了可能と判断した場合でも、Engine 側に未完了の論理 post 送信、論理 reply 送信、論理 notice 送信、またはそれらの継続 HEM が残っている場合、Engine は `result = true` を返してはならない。

この場合、Engine は shutdown reply body の `result` を `false` とする。

`result = false` は、Engine 側 application が終了不能または拒否と判断したこと、または Engine 側 HEMP 実装が HEMP protocol state として正常終了可能ではないと判断したことを表す。

Engine 側 application がどの状態を終了可能または終了不能と判断するかは、HEMP Core では定義しない。

Engine 側 application は、未完了の HEM Timeline、未完了の通常 HEM 通信、内部処理、外部リソース状態、ユーザー操作、または application 固有の条件を判断材料としてよい。

この節における `result = false` は、Body Contract に合う shutdown post に対する通常の shutdown 可否判定を表し、shutdown post の形式不正、HEMP flow violation、または shutdown Body Contract failure を意味しない。

shutdown post body が Body Contract に合わない場合の failure response policy は、`09-failure-handling.md` で定義する。

Header validation および flow validation が成功している場合、receiver は `shutdown` reply body の `result = false` によって shutdown Body Contract failure を返してよい。

この場合の reply は、汎用 error reply ではない。

この場合も、HEMP session は正常終了しない。

`result = false` の場合、HEMP session は終了状態へ移行しない。

transport close、transport 再利用、Engine process 終了の具体手順は、HEMP Core ではなく Transport Binding または実装側が定義する。

### 10.7 shutdown post 後の送信制約

Host は、shutdown post を送信した後、対応する shutdown reply を受信するまで、新たな HEM を送信してはならない。

ここでいう新たな HEM には、未完了の論理 post 送信、論理 reply 送信、または論理 notice 送信を継続するための後続 HEM も含む。

Engine は、`result = true` の shutdown reply を送信した後、新たな HEM を送信してはならない。

ここでいう新たな HEM には、未完了の論理 post 送信、論理 reply 送信、または論理 notice 送信を継続するための後続 HEM も含む。

---

## 11. cancel protocol channel

### 11.1 channel

```text
channel = -4
```

`cancel` は、Agreement 成立後に使用できる、Host から Engine へのキャンセル要求用 protocol channel である。

`cancel` は、対象 application channel + thread 上で進行中の HEM Timeline に対するキャンセル要求である。

現行 `cancel` channel が対象にできる HEM Timeline は、`service provider side = engine` の application channel 上の HEM Timeline に限る。

`service provider side = host` の application channel 上の HEM Timeline は、現行 `cancel` channel の対象ではない。

Engine から Host への cancel が必要な場合は、現行 `cancel` channel を両方向化せず、将来の別 protocol channel または別仕様で扱う。

`cancel` は対象 HEM Timeline の reply を代替しない。

`cancel` は対象 HEM Thread を即時 close しない。

### 11.2 許可される direction / role

`cancel` で許可される direction / role の組み合わせは次の通りである。

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

不正な direction / role の組み合わせを `cancel` で受信した場合、HEMP flow violation とする。

### 11.3 header

`cancel` は、1 HEM post + 1 HEM reply で完結する。

`cancel` では chunking を許可しない。

`cancel` では sender-side logical-send abort を使用しない。

`cancel` の `abort` は常に `false` とする。

`cancel` post の Core Header field は、次の値を持つ。

```text
cancel post Core Header semantic field set:
  direction = to_engine
  role      = post
  channel   = -4
  thread    = N
  seq       = 1
  end       = true
  close     = true
  abort     = false
```

この `thread` は例示値であり、固定値ではない。

`cancel` post の `thread` は、Host が `channel = -4` 内で割り当てる cancel request 用 HEM Thread 番号である。

`cancel` reply header は、対応する cancel post と同じ `channel + thread` を使う。

```text
cancel reply Core Header semantic field set:
  direction = to_host
  role      = reply
  channel   = -4
  thread    = N
  seq       = 2
  end       = true
  close     = true
  abort     = false
```

同じ `channel = -4` 内で、open 状態または closing 状態の cancel request thread 番号を再利用してはならない。

`cancel` reply が完了した時点で、その `channel = -4 / thread = N` は closed になり、再利用可能である。

### 11.4 cancel request 同時数

Host は、Agreement 済み `body.limits.protocol.cancel.in_flight_requests` を超えて、同時に未完了の cancel request を開始してはならない。

Host が Agreement 済み `body.limits.protocol.cancel.in_flight_requests` を超えて新しい cancel request を開始した場合、HEMP flow violation とする。

`body.limits.protocol.cancel.in_flight_requests` は、`06-agreement-limits.md` で定義する。

### 11.5 post body

`cancel` post body は、HEMP が定義する必須 field として `target` と `reason` を持つ。

```text
cancel post body semantic fields:
  target.channel = 1
  target.thread  = 12
  reason         = ""
```

`target.channel` は、`channel > 0` の application channel を指定する。

`target.thread` は、`target.channel` 内の HEM Thread を指定する。

`target.thread` の有効範囲は `1 ～ 4294967295` とする。

`target` が欠落している、`target.channel` / `target.thread` が欠落している、または `target.channel` / `target.thread` の値・範囲が不正である場合、cancel Body Contract failure とする。

`target` の形式が正しい場合でも、次のいずれかに該当する場合、対象はキャンセル可能ではないものとして扱う。

```text
target.channel が Agreement 済み Application Channel Table に存在しない。
target.channel の service provider side が engine ではない。
target.thread が target.channel 内の open 状態の HEM Thread ではない。
target.thread が進行中の HEM Timeline を持たない。
対象 HEM Timeline が application または実装の判断でキャンセル可能ではない。
```

`target` が、post が `abort = true` で終端済みであり、対応 reply がまだ完了していない HEM Timeline を指す場合、その HEM Timeline は進行中の HEM Timeline として扱う。

この場合、その対象は `target.thread` が進行中の HEM Timeline を持たない場合には該当しない。

ただし、その対象 HEM Timeline が application または実装の判断でキャンセル可能ではない場合、cancel reply body の `result` を `false` とする。

対象をキャンセル可能ではないものとして `result = false` を返す場合、cancel message 自体が正しければ、HEMP flow violation にはしない。

target.channel の service provider side が engine ではないことは、cancel Body Contract failure ではない。

この場合、receiver は cancel reply body の `result = false` によって、対象がキャンセル可能ではないことを返す。

`reason` は required semantic field である。

`reason` の具体的な値、意味、分類、命名規則は HEMP Core では定義しない。

`reason` の内容は、HEMP を利用する application 側が決める。

`reason` の空文字列は有効とする。

### 11.6 reply body

`cancel` reply body は、HEMP が定義する必須 field として `result` と `message` を持つ。

```text
cancel reply body semantic fields:
  result = true
  message = "Cancel request is accepted."
```

`result` は、キャンセル要求を受け付けたかどうかを表す。

`result = true` は、対象処理が停止済みであることを意味しない。

`result = true` は、対象 HEM Timeline が完了済みであること、対象 reply が `abort = true` で終端すること、または対象 reply が正常完了することも意味しない。

`result = false` は、cancel post body が形式上 valid であっても、対象 HEM Timeline が cancel request の対象として受け入れられなかったことを表す。

`result = false` には、少なくとも次を含む。

```text
target.channel が Agreement 済み Application Channel Table に存在しない。
target.channel の service provider side が engine ではない。
target.thread が target.channel 内の open 状態の HEM Thread ではない。
target.thread が進行中の HEM Timeline を持たない。
対象 HEM Timeline が application または実装の判断でキャンセル可能ではない。
```

これらは、cancel post body の形式が正しければ、HEMP flow violation ではない。

これらは、cancel Body Contract failure でもない。

cancel post body が Body Contract に合わない場合の failure response policy は、`09-failure-handling.md` で定義する。

Header validation および flow validation が成功している場合、receiver は `cancel` reply body の `result = false` によって cancel Body Contract failure を返してよい。

この場合、cancel request は受け付けられない。

HEMP Core は、`cancel` reply body に `target` field を定義しない。

受信側は、`cancel` reply body に `target` が含まれる場合、それを未知fieldとして無視する。

HEMP 実装は、`cancel` reply body に `target` を送信しないことを推奨する。

`cancel` reply がどの cancel request に対応するかは、cancel reply header の `channel + thread` によって判断する。

cancel 対象の `target` は、対応する cancel post body の `target` によって判断する。

`cancel` header の `thread` は cancel request 自体の HEM Thread であり、`cancel` post body の `target.thread` は cancel 対象の HEM Thread である。

### 11.7 cancel と abort の関係

cancel reply body の `result = true` は、cancel request の受理のみを表す。

`result = true` は、対象 HEM Timeline の処理が停止済みであることを意味しない。

`result = true` は、対象 HEM Timeline が完了済みであることを意味しない。

`result = true` は、対象 HEM Timeline が aborted HEM Timeline として完了することを意味しない。

`result = true` は、対象 HEM Timeline が normal completed HEM Timeline として完了することも意味しない。

対象 HEM Timeline の完了状態は、対象 application channel 上の実際の HEM Timeline の完了状態によって判断する。

現行 `cancel` channel が対象にできる HEM Timeline は、`service provider side = engine` の application channel 上の HEM Timeline に限る。

したがって、cancel が受理された対象 HEM Timeline において、対象 reply の sender は Engine である。

Engine が cancel を受理した結果、HEMP session を継続したまま対象の論理 reply 送信を正常な application reply body として完了させない場合、Engine は対象 reply を `abort = true` で終端する。

この場合、対象 HEM Timeline は aborted HEM Timeline として完了する。

Engine が cancel を受理した後でも、対象処理を停止しない、または停止できない場合、Engine は対象 reply を `abort = false` で正常完了してよい。

Application Body Contract が、キャンセルされた結果を正常な application reply body として定義する場合、Engine は対象 reply を `abort = false` で正常完了してよい。

対象 reply を正常完了させるか `abort = true` で終端させるか、`abort = true` で終端させる場合の abort body の内容、および abort body 内の診断情報の意味は、Application Body Contract または実装側が定義してよい。

HEMP Core は、cancel の受理と対象 HEM Timeline の abort 終端との間に、強制的な関係を定義しない。

Host は、cancel reply body の `result = true` だけを根拠に、対象 HEM Timeline が完了した、停止した、正常完了した、または aborted 状態になったと判断してはならない。

`cancel` は対象 HEM Timeline の reply を代替しない。

---

## 12. protocol channel Body Contract failure

protocol channel body が Body Contract として読めない、判定できない、または必須 field を欠く場合、HEM Body Contract failure とする。

protocol channel Body Contract failure は、known body branch が存在し、HEM Header validation および HEMP flow validation が成功した後に適用する。

known body branch が存在しない場合、それは protocol channel Body Contract failure ではなく、HEMP payload format failure である。

例:

```text
ping reply body の result / message が欠落している。
shutdown post body の reason が欠落している。
shutdown reply body の result / message が欠落している。
cancel post body の target / reason が欠落している。
cancel post body の target.channel / target.thread が欠落している。
cancel post body の target.channel / target.thread の値や範囲が不正である。
cancel reply body の result / message が欠落している。
message が空文字列である。
```

protocol channel body の未知fieldは、HEM Body Contract failure ではない。

failure response policy は、`09-failure-handling.md` で定義する。

---

## 13. この文書で扱わないもの

この文書では、次を定義しない。

```text
- agreement body / limits の詳細
- Channel Table / digest の canonical byte sequence
- HEM frame / HEM Length Header の構造
- Core Header field の個別定義
- HEM Thread lifecycle 全体
- Standard Data Body の詳細
- failure response policy の詳細
- Transport Binding の具体仕様
- Engine から Host への cancel 用 protocol channel
```
