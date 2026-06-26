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

---

## 10. shutdown protocol channel

### 10.1 channel

```text
channel = -3
```

`shutdown` は、Agreement 成立後に Host が Engine へ HEMP session の正常終了を要求するための protocol channel である。

### 10.2 許可される direction / role

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

### 10.3 header

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

### 10.4 post body

`ShutdownPostBody.reason` は presence required である。

`reason` は string である。

`reason` の空文字列は有効である。

### 10.5 reply body

`ShutdownReplyBody.result` は presence required である。

`ShutdownReplyBody.message` は presence required である。

`message` は non-empty でなければならない。

`result = true` は、Engine が shutdown request を受け入れ、HEMP session を正常終了状態へ移行できることを表す。

`result = false` は、Engine が shutdown request を受け入れなかったことを表す。

`result = false` の場合、HEMP session は正常終了状態へ移行しない。

shutdown post body が Body Contract に合わない場合の failure response policy は、`09-failure-handling.md` で定義する。

Header validation および flow validation が成功している場合、receiver は `shutdown` reply body の `result = false` によって shutdown Body Contract failure を返してよい。

この場合の reply は、汎用 error reply ではない。

この場合も、HEMP session は正常終了しない。

### 10.6 shutdown 成功後

`result = true` の shutdown reply 完了後、HEMP session は正常終了状態になる。

`result = true` の shutdown reply を送信した後、Engine は同じ HEMP session 上で新しい HEM を送信してはならない。

`result = true` の shutdown reply を受信した後、Host は同じ HEMP session 上で新しい HEM を送信してはならない。

---

## 11. cancel protocol channel

### 11.1 channel

```text
channel = -4
```

`cancel` は、Agreement 成立後に Host が Engine へ application channel 上の進行中 HEM Timeline の cancel を要求するための protocol channel である。

現行 `cancel` channel は Host から Engine への cancel request を扱う。

Engine から Host への cancel が必要な場合は、現行 `cancel` channel を両方向化せず、将来の別 protocol channel または別仕様で扱う。

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

`cancel` post の Core Header field は、次の固定値を持つ。

```text
cancel post Core Header semantic field set:
  direction = to_engine
  role      = post
  channel   = -4
  thread    = 1
  seq       = 1
  end       = true
  close     = true
  abort     = false
```

`cancel` reply の Core Header field は、次の固定値を持つ。

```text
cancel reply Core Header semantic field set:
  direction = to_host
  role      = reply
  channel   = -4
  thread    = 1
  seq       = 2
  end       = true
  close     = true
  abort     = false
```

### 11.4 post body

`CancelPostBody` は、HEMP が定義する必須 field として次を持つ。

```text
target
reason
```

`reason` は string である。

`reason` の空文字列は有効である。

`target.channel` は、`channel > 0` の application channel を指定する。

`target.channel` は、Agreement 済み Application Channel Table に存在しなければならない。

現行 `cancel` channel が対象にできる `target.channel` は、`service provider side = engine` の application channel に限る。

`service provider side = host` の application channel 上の HEM Timeline は、現行 `cancel` channel の対象ではない。

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

対象をキャンセル可能ではないものとして `result = false` を返す場合、cancel request message 自体が正しければ、HEMP flow violation にはしない。

target.channel の service provider side が engine ではないことは、cancel Body Contract failure ではない。

この場合、receiver は cancel reply body の `result = false` によって、対象がキャンセル可能ではないことを返す。

### 11.5 reply body

`CancelReplyBody.result` は presence required である。

`CancelReplyBody.message` は presence required である。

`message` は non-empty でなければならない。

`result = true` は、cancel request が受け入れられたことを表す。

`result = true` は、対象 HEM Timeline が正常完了したこと、aborted 終端したこと、またはすでに停止済みであることを意味しない。

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

### 11.6 cancel 受理後の対象 Timeline 扱い

現行 `cancel` channel が対象にできる HEM Timeline は、`service provider side = engine` の application channel 上の HEM Timeline に限る。

したがって、cancel が受理された対象 HEM Timeline において、対象 reply の sender は Engine である。

cancel を受理した結果として、HEMP session を継続したまま対象の論理 reply 送信を正常な application reply body として完了させない場合、Engine は対象 reply を `abort = true` で終端する。

Engine は、対象 reply を正常完了してよい。

`cancel` reply body の `result = true` は、対象 reply が正常完了したこと、aborted 終端したこと、またはすでに停止済みであることを意味しない。

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
- agreement body / limits body の詳細
- HEM Payload encoding の具体的な wire 表現
- application channel ごとの Body Contract
- transport failure の詳細
- retry / reconnect policy
- Engine から Host への cancel 用 protocol channel
```
