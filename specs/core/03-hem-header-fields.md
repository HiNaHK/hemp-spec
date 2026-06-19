# HEM Header fields

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Core protocol specification

---

## 1. この文書の目的

この文書は、HEMP における HEM Header fields を定義する。

HEM Header は、HEM Payload 内にある Core Header field set であり、HEM Body を HEMP 上でどのように解釈するかを示す。

この文書で扱う Core Header field は次の8 fieldである。

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

この文書では、各 field の存在、意味、有効値、有効範囲、不正値、および field 組み合わせ validation を定義する。

HEM frame、HEM Length Header、HEM Payload encoding、HEM Timeline 全体、HEM Thread lifecycle 全体、Channel Table、agreement、limits、protocol channel body、Standard Data Body、failure response policy、Transport Binding の詳細は、後続文書で定義する。

---

## 2. Core Header field set

HEM Header は、HEMP が定義する必須 field set として、次の8 fieldを持つ。

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

これらを Core Header field と呼ぶ。

Core Header field は必須である。

Core Header field が欠落している、値が不正である、範囲外である、または組み合わせが不正である場合、HEMP header validation failure とする。

`end`、`close`、`abort` は boolean field であり、`false` も有効な値である。  
そのため、HEMP Core semantics 上、これらの field は `false` と欠落を区別できなければならない。

Core Header field の具体的な encoding 表現は、Protobuf Encoding specification が定義する。

HEM Header に Core Header field 以外の既知でない field が含まれる場合、Core semantics 上は未知fieldとして無視する。

未知fieldは Core Header field の代替にはならない。

Protobuf unknown fields の具体的な扱いは、Protobuf Encoding specification が定義する。

Core Header semantic field set の例は次の通りである。

```text
Core Header semantic field set example:
  direction = to_engine
  role      = post
  channel   = 1
  thread    = 1
  seq       = 1
  end       = true
  close     = false
  abort     = false
```

---

## 3. direction

### 3.1 field 定義

```text
field name: direction
semantic type: direction enum
presence: required
```

有効値は次の通りである。

```text
to_engine
to_host
```

次は不正である。

```text
unspecified
unknown / unrecognized direction value
```

Encoding 上の enum 名、enum 値、wire 表現は、Protobuf Encoding specification が定義する。

### 3.2 意味

```text
to_engine:
  Host から Engine へ送る HEM。

to_host:
  Engine から Host へ送る HEM。
```

`direction` は、HEMP を使う Transport Binding が定義する実通信方向と一致しなければならない。

`direction` が実通信方向と一致しない HEM を受信した場合、HEMP header validation failure とする。

HEMP Core は、特定の transport 方式を要求しない。

HEMP Core と Transport Binding の関係は、`10-transport-binding-relationship.md` で定義する。

---

## 4. role

### 4.1 field 定義

```text
field name: role
semantic type: role enum
presence: required
```

有効値は次の通りである。

```text
post
reply
notice
```

次は不正である。

```text
unspecified
unknown / unrecognized role value
```

Encoding 上の enum 名、enum 値、wire 表現は、Protobuf Encoding specification が定義する。

### 4.2 意味

```text
post:
  相手へ論理送信を開始または継続する role。

reply:
  対応する post へ返す role。

notice:
  reply を要求しない通知 role。
```

`role` は、HEMP 上の通信上の役割を表す。

`role` は、application 上の処理、操作、command を表すものではない。

post / reply / notice の flow semantics は、`04-flow-semantics.md` で定義する。

---

## 5. channel

### 5.1 field 定義

```text
field name: channel
semantic type: signed integer channel identifier
presence: required
invalid value: 0
```

### 5.2 有効範囲

`channel` の Header 値として有効な範囲は次の通りである。

```text
-2147483648..-1
1..2147483647
```

`channel = 0` は不正であり、HEMP header validation failure とする。

### 5.3 channel の領域

`channel` の値は、次の領域に分かれる。

```text
channel < 0:
  protocol channel

channel = 0:
  invalid

channel > 0:
  application channel
```

### 5.4 protocol channel space

`channel < 0` は protocol channel space に属する。

定義済みの protocol channel は次の通りである。

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

未定義の negative channel は、将来の HEMP protocol channel 用に予約する。

未定義 protocol channel を使用した場合、Header 値としての `channel` は範囲内であっても、HEMP flow violation とする。

protocol channel の Header、Body Contract、使用可能タイミング、固有の flow rule は、`07-protocol-channels.md` で定義する。

### 5.5 application channel space

`channel > 0` は application channel space に属する。

application channel の具体的な channel 番号は、agreement 成立後、agreement 済み Application Channel Table の canonical Entry order に基づく 1-based reference として解釈する。

Application Channel Table の詳細は、`05-channel-table-digest.md` で定義する。

---

## 6. thread

### 6.1 field 定義

```text
field name: thread
semantic type: unsigned integer thread identifier
presence: required
invalid value: 0
```

### 6.2 有効範囲

`thread` の Header 値として有効な範囲は次の通りである。

```text
1..4294967295
```

`thread = 0` は不正であり、HEMP header validation failure とする。

### 6.3 意味

`thread` は、channel 内に存在する HEM Thread を識別する。

HEM Thread は、channel 内に存在するプロトコル上のミニセッションである。

HEM Thread は、OS thread、runtime thread、worker thread、または実装上の thread ではない。

`thread` の scope は channel ごとに独立する。

```text
channel = 1, thread = 1
channel = 2, thread = 1
```

この2つは別の HEM Thread である。

### 6.4 sparse identifier

`thread` は sparse identifier である。

HEMP 実装は、thread 番号の数値最大値に比例する資源を確保してはならない。

同時に open できる thread 数は、agreement 済み `open_threads` limits によって制御する。

HEM Thread の開始、closing、closed、thread 番号の再利用、`open_threads` limits の詳細は、`04-flow-semantics.md` および `06-agreement-limits.md` で定義する。

---

## 7. seq

### 7.1 field 定義

```text
field name: seq
semantic type: unsigned integer sequence number
presence: required
invalid value: 0
valid range: 1..4294967295
```

Encoding 上の numeric representation は、Protobuf Encoding specification が定義する。

### 7.2 有効範囲

`seq` の有効範囲は次の通りである。

```text
1..4294967295
```

`seq = 0` は不正であり、HEMP header validation failure とする。

### 7.3 wrap-around 禁止

`seq` は wrap-around しない。

`seq` が上限に達した場合、次の `seq` として `1` に戻してはならない。

送信側は、`seq` が上限を超える続きを必要とする論理送信を構成してはならない。

`seq` が有効範囲外である HEM を受信した場合、HEMP header validation failure とする。

`seq` の値自体は有効範囲内だが、同一の論理送信内で上限到達後に `seq = 1` へ戻した HEM を受信した場合、HEMP flow violation とする。

### 7.4 HEM Timeline との関係

HEM Timeline では、`seq` は現在の HEM Timeline 内の HEM 順序番号である。

論理 post 送信は `seq = 1` から始める。

論理 post 送信が複数 HEM に分かれる場合、HEM ごとに `seq` を 1 ずつ増やす。

対応する論理 reply 送信は、同じ HEM Timeline の `seq` の続きを使う。

例:

```text
post,  seq = 1
post,  seq = 2
post,  seq = 3, end = true
reply, seq = 4, end = true
```

HEM Timeline 完了後、同じ `channel + thread` で次の HEM Timeline を始める場合、`seq` は再び `1` から始める。

HEM Timeline の詳細は、`04-flow-semantics.md` で定義する。

### 7.5 notice との関係

notice は HEM Timeline を形成しない。

論理 notice 送信では、`seq` はその論理 notice 送信内で `1` から始まり、HEM ごとに増加する。

notice の `seq` は、同じ thread 上の HEM Timeline の `seq` から独立する。

論理 notice 送信完了後、次の論理 notice 送信では `seq = 1` から始める。

受信側は、同じ `channel + thread` 上でも、HEM Timeline 用の seq 状態と、論理 notice 送信用の seq 状態を別々に管理しなければならない。

notice の seq は、同じ `channel + thread` 上で進行中または直近の HEM Timeline の expected seq を進めない。

HEM Timeline の seq も、同じ `channel + thread` 上の notice の expected seq を進めない。

notice の詳細は、`04-flow-semantics.md` で定義する。

---

## 8. end

### 8.1 field 定義

```text
field name: end
semantic type: boolean
presence: required
```

`end` の semantic presence がない場合、HEMP header validation failure とする。

### 8.2 意味

```text
end = false:
  現在の論理送信はこの HEM の後も続く。

end = true:
  現在の論理送信はこの HEM で終わる。
```

`end = true` は、論理送信の終端を表す。

その終端が正常完了か aborted 終端かは、`abort` が表す。

```text
end = true, abort = false:
  論理送信は正常完了する。

end = true, abort = true:
  論理送信は aborted 状態で終端する。
```

role ごとの意味は次の通りである。

```text
post:
  論理 post 送信の終端。

reply:
  論理 reply 送信の終端。

notice:
  論理 notice 送信の終端。
```

---

## 9. close

### 9.1 field 定義

```text
field name: close
semantic type: boolean
presence: required
```

`close` の semantic presence がない場合、HEMP header validation failure とする。

### 9.2 意味

`close` は、論理送信が終わる場合に、HEM Thread close 要求または close 確認を伴うかを表す。

```text
close = false:
  この HEM は HEM Thread close 要求または close 確認を伴わない。

close = true:
  この HEM は HEM Thread close 要求または close 確認を伴う。
```

`close = true` は HEM Thread close 要求または close 確認である。

`close = true` は即時破棄命令ではない。

post / reply では次の意味になる。

```text
post.close = true:
  HEM Thread close 要求。

reply.close = true:
  対応 post の HEM Thread close 要求に対する確認。
```

notice では `close = true` を許可しない。

HEM Thread close の詳細は、`04-flow-semantics.md` で定義する。

---

## 10. abort

### 10.1 field 定義

```text
field name: abort
semantic type: boolean
presence: required
```

`abort` の semantic presence がない場合、HEMP header validation failure とする。

`abort` は Core Header field であり、必須 field である。

### 10.2 意味

```text
abort = false:
  この HEM は、現在の論理送信を aborted 状態で終端しない。

abort = true:
  この HEM は、現在の論理送信を aborted 状態で終端する。
```

`abort = true` は HEMP failure ではない。

`abort = true` は、正常な HEMP flow 内で表現される、論理送信単位の異常終端状態である。

`abort = true` は、transport failure ではない。

sender-side logical-send abort の詳細は、`04-flow-semantics.md` で定義する。

---

## 11. end / close / abort の有効な組み合わせ

### 11.1 post / reply で有効な組み合わせ

post / reply で有効な組み合わせは次の通りである。

```text
end = false, close = false, abort = false
end = true,  close = false, abort = false
end = true,  close = true,  abort = false
end = true,  close = false, abort = true
end = true,  close = true,  abort = true
```

### 11.2 post / reply で不正な組み合わせ

post / reply で次の組み合わせは不正とする。

```text
end = false, close = false, abort = true
end = false, close = true,  abort = false
end = false, close = true,  abort = true
```

不正な組み合わせを受信した場合、HEMP header validation failure とする。

### 11.3 notice で有効な組み合わせ

notice では `close = true` を許可しない。

notice で有効な組み合わせは次の通りである。

```text
end = false, close = false, abort = false
end = true,  close = false, abort = false
end = true,  close = false, abort = true
```

### 11.4 notice で不正な組み合わせ

notice で `close = true` を受信した場合、HEMP header validation failure とする。

notice で `abort = true` かつ `end = false` を受信した場合、HEMP header validation failure とする。

---

## 12. end / close / abort の意味まとめ

```text
end:
  この HEM で現在の論理送信が終わるかを表す。

abort:
  論理送信が終わる場合、その終端が正常完了か aborted 終端かを表す。

close:
  論理送信が終わる場合、HEM Thread close 要求または close 確認を伴うかを表す。
```

post / reply で有効な各組み合わせの意味は次の通りである。

```text
end = false, close = false, abort = false:
  論理送信は続く。
  HEM Thread も維持する。

end = true, close = false, abort = false:
  論理送信は正常完了する。
  HEM Thread は維持する。

end = true, close = true, abort = false:
  論理送信は正常完了する。
  HEM Thread close 要求または close 確認を行う。

end = true, close = false, abort = true:
  論理送信は aborted 状態で終端する。
  HEM Thread は維持する。

end = true, close = true, abort = true:
  論理送信は aborted 状態で終端する。
  HEM Thread close 要求または close 確認を行う。
```

`abort` と `close` は直交する。

`abort` は `close` の意味を変更しない。

`close` は `abort` の意味を変更しない。

`reply.close` は、対応する `post.close` と一致しなければならない。

```text
post.close = false -> reply.close = false
post.close = true  -> reply.close = true
```

この規則は、post または reply が aborted 状態で終端する場合にも適用する。

`reply.close` が対応する `post.close` と一致しない場合、HEMP flow violation とする。

---

## 13. abort body との関係

application channel の `abort = true` HEM は、abort body を持つ。

`abort_body` は空 bytes でよい。

最小の application channel abort HEM は、概念上、次の semantic structure を持つ。

```text
application channel abort HEM semantic example:
  header.direction = to_host
  header.role      = reply
  header.channel   = 1
  header.thread    = 12
  header.seq       = 6
  header.end       = true
  header.close     = false
  header.abort     = true
  body             = abort body
```

`abort = true` の body は、通常完了時の application body として扱ってはならない。

`abort = true` の場合、受信側は通常完了時の application Body Contract を、その論理送信の正常 body 検証として適用してはならない。

`abort = true` の body に含まれる field、reason、message、diagnostic 情報の意味は、HEMP Core では定義しない。

Application Body Contract または実装は、abort body 内の診断情報を定義してよい。

abort body 内の診断情報を解釈できない場合でも、Header の `abort = true` によって表される aborted 終端状態は変わらない。

abort body も HEM Payload bytes の一部であり、HEM Payload length と parser resource limit の対象に含まれる。

`abort = true` の HEM は、該当する channel 種別に応じて、protocol channel Payload length 上限または agreement 済み `body.limits` に基づく上限判定の対象に含まれる。

abort body の具体的な encoding 表現は、Protobuf Encoding specification で定義する。

---

## 14. この文書で扱わないもの

この文書では、次を定義しない。

```text
- HEM frame / HEM Length Header の構造
- HEM Payload encoding の詳細
- Protobuf enum 名、field number、wire representation
- HEM Timeline 全体
- HEM Thread lifecycle 全体
- post / reply / notice の flow 全体
- Channel Table / digest
- agreement body / limits body
- protocol channel body の詳細
- Standard Data Body の詳細
- failure response policy の詳細
- Transport Binding の具体仕様
```
