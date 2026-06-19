# HEMP flow semantics

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Core protocol specification

---

## 1. この文書の目的

この文書は、HEMP flow semantics を定義する。

この文書では、HEM Thread、HEM Thread lifecycle、HEM Timeline、post / reply flow、notice flow、seq progression、end / close / abort による flow state、channel flags と flow の関係、limits と flow の関係、Standard Data Body と flow semantics の関係、および HEMP flow violation を扱う。

HEM Header field の単体定義は、`03-hem-header-fields.md` で定義する。

---

## 2. flow semantics の基本単位

HEMP flow semantics は、次の単位を扱う。

```text
- HEM
- 論理 post 送信
- 論理 reply 送信
- 論理 notice 送信
- HEM Thread
- HEM Timeline
```

1つの論理 post / reply / notice 送信は、1つまたは複数の HEM で構成され得る。

論理送信が複数の HEM で構成される場合、`seq`、`end`、`abort`、channel flags、および limits に従って送受信する。

---

## 3. HEM Thread

HEM Thread は、channel 内に存在するプロトコル上のミニセッションである。

HEM Thread は、OS thread、runtime thread、worker thread、または実装上の thread ではない。

HEM Thread は、`channel + thread` によって識別する。

`thread` の scope は channel ごとに独立する。

```text
channel = 1, thread = 1
channel = 2, thread = 1
```

この2つは別の HEM Thread である。

---

## 4. HEM Thread lifecycle

HEM Thread は、open、closing、closed の状態を持つ。

### 4.1 open

HEM Thread は、その `channel + thread` の最初の論理 post 送信によって open 状態になる。

新しい thread 番号は、post 開始側が割り当てる。

新しい post で使う thread 番号は、同じ channel 内で open 状態または closing 状態であってはならない。

すでに open 状態または closing 状態の thread 番号を、新しい HEM Thread の開始として使用した場合、HEMP flow violation とする。

### 4.2 closing

HEM Thread close は、`end = true` かつ `close = true` の post によって要求する。

その post が完了した後、対応 reply が完了するまで、その HEM Thread は closing 状態になる。

closing 状態の HEM Thread では、新しい HEM Timeline を開始してはならない。

### 4.3 closed

対応 reply が `end = true` かつ `close = true` で完了した時点で、HEM Thread は closed 状態になる。

closed になった thread 番号は、同じ channel 内で再利用可能である。

closed 状態の HEM Thread に属する未完了の論理送信は存在してはならない。

### 4.4 abort と close

`abort` は HEM Thread close の意味を変更しない。

`abort = true` かつ `close = true` の post は、aborted 状態で終端した論理 post 送信であり、同時に HEM Thread close を要求する。

`abort = true` かつ `close = true` の reply は、aborted 状態で終端した論理 reply 送信であり、同時に対応 post の HEM Thread close 要求を確認する。

---

## 5. HEM Timeline

HEM Timeline は、1つの論理 post 送信と、それに対応する1つの論理 reply 送信で構成される通信単位である。

```text
HEM Timeline
  = 1つの論理 post 送信
  + 対応する1つの論理 reply 送信
```

HEM Timeline は、1つの `channel + thread` 上で行う。

HEM Timeline は、protocol channel と application channel の両方に適用される共通の post / reply 型通信単位である。

各 protocol channel 固有の Core Header field の固定値、Body Contract、使用可能タイミング、固有の flow rule は、`07-protocol-channels.md` で定義する。

HEM Timeline は、完了状態として normal completed HEM Timeline と aborted HEM Timeline を区別する。

```text
normal completed HEM Timeline:
  post が正常完了し、reply も正常完了した HEM Timeline。

aborted HEM Timeline:
  post または reply の少なくとも一方が aborted 状態で終端した HEM Timeline。
```

aborted HEM Timeline は、HEM Timeline としては完了済みである。

---

## 6. post / reply の channel と thread

reply は、対応 post と同じ `channel + thread` を使う。

reply 側が新しい thread を割り当ててはならない。

新しい thread を開く場合は、post 開始側が thread 番号を割り当てる。

対応 post と異なる `channel + thread` を使う reply を受信した場合、HEMP flow violation とする。

---

## 7. post / reply / notice の direction と service provider side

post / reply / notice の許可 direction は、channel の service provider side から導かれる。

service provider side が Engine の場合、許可される組み合わせは次の通りである。

```text
post:
  direction = to_engine
  role      = post

reply:
  direction = to_host
  role      = reply

notice:
  direction = to_host
  role      = notice
```

service provider side が Host の場合、許可される組み合わせは次の通りである。

```text
post:
  direction = to_host
  role      = post

reply:
  direction = to_engine
  role      = reply

notice:
  direction = to_engine
  role      = notice
```

service provider side 自体の Channel Entry 定義は、`05-channel-table-digest.md` で定義する。

direction / role が channel の service provider side 規則に違反する場合、HEMP flow violation とする。

---

## 8. reply 開始タイミング

reply は、対応する論理 post 送信が `end = true` で終端した後に開始する。

対応 post が `abort = true` で終端した場合も、その post は論理 post 送信としては終端済みである。

対応 post が `abort = true` で終端した場合、対応 reply も `abort = true` で終端しなければならない。

対応 post が終端する前に reply を開始した場合、HEMP flow violation とする。

---

## 9. HEM Timeline の順序制約

同じ `channel + thread` 上では、現在の HEM Timeline の対応 reply が完了するまで、次の post を開始してはならない。

許可される順序の例は次の通りである。

```text
post A
reply A
post B
reply B
```

不正な順序の例は次の通りである。

```text
post A
post B
reply A
```

現在の HEM Timeline の対応 reply が完了する前に、同じ `channel + thread` 上で次の post を開始した場合、HEMP flow violation とする。

---

## 10. seq progression in HEM Timeline

論理 post 送信は、`seq = 1` から始める。

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

`seq` は wrap-around しない。

`seq` に欠番、重複、順序違い、または wrap-around がある場合、HEMP flow violation とする。

`seq` field の値域や単体 validation は、`03-hem-header-fields.md` で定義する。

---

## 11. reply の意味

`reply` は、対応する post への HEMP 上の応答である。

`reply` は、HEMP を利用する側の結果 body そのものを自動的に意味しない。

HEMP を利用する側の出力を Host へ送る必要がある場合、その出力は適切な channel 上の `to_host / post` として送ることができる。

Host は、それを受け取った後に対応する `to_engine / reply` を返す。

ただし、agreement protocol channel は、agreement 結果を返す専用の reply Body Contract を定義する。

protocol channel の固有規則は、`07-protocol-channels.md` で定義する。

---

## 12. abort と HEM Timeline

post が `abort = true` で終端した場合、受信側はその post body を正常な application post body として扱ってはならない。

post が `abort = true` で終端した場合、対応 reply も `abort = true` で終端しなければならない。

HEM Timeline の完了状態は、次の通りである。

```text
post.abort = false, reply.abort = false:
  normal completed HEM Timeline

post.abort = false, reply.abort = true:
  aborted HEM Timeline

post.abort = true, reply.abort = true:
  aborted HEM Timeline
```

次は HEMP flow violation とする。

```text
post.abort = true, reply.abort = false
```

reply が `abort = true` で終端した場合、対応 post に対する論理 reply 送信は aborted 状態で終端する。

受信側は、その reply body を正常な application reply body として扱ってはならない。

HEM Timeline は aborted HEM Timeline として完了する。

aborted 状態で終端済みの論理 post 送信、論理 reply 送信、または論理 notice 送信に対して、同じ論理送信の継続 HEM を受信した場合、HEMP flow violation とする。

---

## 13. notice

`notice` は、reply を要求しない通知 role である。

notice は HEM Timeline を形成しない。

notice は HEM Thread を open しない。

notice は HEM Thread を close しない。

notice では `close = true` を許可しない。

notice では常に次を使う。

```text
close = false
```

notice は、protocol channel と application channel の両方に適用できる共通 role である。

ただし、`notice banned flag = true` の channel では notice を使用できない。

同じ `direction + role + channel + thread` 上では、未完了の論理 notice 送信を同時に1つまでとする。

---

## 14. notice と abort

notice では、`abort = true` を使って論理 notice 送信を aborted 状態で終端してよい。

notice が `abort = true` で終端した場合、受信側はその notice body を正常な application notice body として扱ってはならない。

notice の `abort = true` は、HEM Timeline 状態または HEM Thread 状態を変化させない。

---

## 15. notice と HEM Thread 状態

timeline を持つ channel では、notice を open 状態の HEM Thread へ紐づけてよい。

対象 HEM Thread は open 状態でなければならない。

対象 HEM Thread は次の状態であってはならない。

```text
未open状態
closing状態
closed状態
```

channel は notice を許可していなければならない。

notice direction は、その channel の service provider side から導かれる notice direction と一致しなければならない。

notice は、HEM Timeline 進行中の open 状態の thread に送ってよい。

notice は、進行中の HEM Timeline がない open 状態の thread に送ってよい。

notice の `seq` は、同じ thread 上の HEM Timeline の `seq` から独立する。

notice を未open状態、closing状態、または closed状態の HEM Thread に紐づけた場合、HEMP flow violation とする。

---

## 16. notice-only channel

notice-only channel は、HEM Timeline を許可せず、notice を許可する channel である。

つまり、notice-only channel は次の flags を持つ channel である。

```text
timeline banned flag = true
notice banned flag = false
```

notice-only channel では次を固定する。

```text
thread = 1
close = false
```

`end` は、chunking banned flag に従う。

`chunking banned flag = true` の notice-only channel では、notice は `end = true` でなければならない。

`chunking banned flag = false` の notice-only channel では、1つの論理 notice 送信が複数 HEM に分かれてよい。

例:

```text
notice, seq = 1, end = false, close = false, abort = false
notice, seq = 2, end = false, close = false, abort = false
notice, seq = 3, end = true,  close = false, abort = false
```

aborted 状態で終端する例:

```text
notice, seq = 1, end = false, close = false, abort = false
notice, seq = 2, end = true,  close = false, abort = true
```

`multi-thread banned flag` は notice の thread ルールには影響しない。

---

## 17. notice の seq 状態

論理 notice 送信では、`seq = 1` から始める。

論理 notice 送信が複数 HEM に分かれる場合、HEM ごとに `seq` を 1 ずつ増やす。

notice の `seq` は、同じ thread 上の HEM Timeline の `seq` から独立する。

論理 notice 送信完了後、次の論理 notice 送信では `seq = 1` から始める。

受信側は、同じ `channel + thread` 上でも、HEM Timeline 用の seq 状態と、論理 notice 送信用の seq 状態を別々に管理しなければならない。

notice の seq は、同じ `channel + thread` 上で進行中または直近の HEM Timeline の expected seq を進めない。

HEM Timeline の seq も、同じ `channel + thread` 上の notice の expected seq を進めない。

notice の seq に欠番、重複、順序違い、または wrap-around がある場合、HEMP flow violation とする。

---

## 18. channel flags と flow semantics

channel flags は、channel 上で許可される flow を制御する。

この文書では、次の channel flags と flow semantics の関係を扱う。

```text
- timeline banned flag
- notice banned flag
- chunking banned flag
- multi-thread banned flag
- disabled channel
```

Channel Entry と flags byte の詳細は、`05-channel-table-digest.md` で定義する。

### 18.1 timeline banned flag

`timeline banned flag = true` の場合、その channel では HEM Timeline を開始してはならない。

```text
timeline banned flag = true:
  HEM Timeline を開始してはならない
  post / reply 型通信を使用できない

timeline banned flag = false:
  service provider side に基づいて HEM Timeline を使用できる
```

`timeline banned flag = true` の channel で post または reply を受信した場合、HEMP flow violation とする。

### 18.2 notice banned flag

`notice banned flag = true` の場合、その channel では notice を許可しない。

```text
notice banned flag = true:
  notice を使用できない

notice banned flag = false:
  service provider side に基づいて notice を使用できる
```

`notice banned flag = true` の channel で notice を受信した場合、HEMP flow violation とする。

### 18.3 chunking banned flag

`chunking banned flag = true` の場合、その channel では論理 post 送信、論理 reply 送信、論理 notice 送信の chunking を許可しない。

```text
chunking banned flag = true:
  論理 post / reply / notice 送信は 1 HEM で完結しなければならない

chunking banned flag = false:
  1つの論理 post / reply / notice 送信が複数 HEM に分かれてよい
```

`chunking banned flag = true` の channel では、post / reply / notice のいずれも `end = true` でなければならない。

`chunking banned flag = true` の channel で `end = false` の HEM を受信した場合、HEMP flow violation とする。

application channel において `chunking banned flag = false` の場合、複数 HEM の論理送信は agreement 済み `body.limits` の範囲内で許可する。

protocol channel において `chunking banned flag = false` の場合、その具体的な扱いは各 protocol channel の規則が定義する。

### 18.4 multi-thread banned flag

`multi-thread banned flag = true` の場合、その channel の HEM Timeline では複数の HEM Thread を使用できない。

```text
multi-thread banned flag = true:
  HEM Timeline では thread = 1 のみを使用する

multi-thread banned flag = false:
  HEM Timeline で複数 HEM Thread を使用できる
```

application channel において `multi-thread banned flag = false` の場合、複数の open HEM Thread は agreement 済み `body.limits.open_threads.per_channel` と `body.limits.open_threads.total` の範囲内で許可する。

protocol channel において `multi-thread banned flag = false` の場合、その具体的な扱いは各 protocol channel の規則が定義する。

`multi-thread banned flag` は notice の thread ルールには影響しない。

`multi-thread banned flag = true` の channel で、HEM Timeline 用の post または reply に `thread != 1` を使った場合、HEMP flow violation とする。

### 18.5 disabled channel

次の組み合わせは disabled channel とする。

```text
timeline banned flag = true
notice banned flag = true
```

disabled channel は、Channel Table 内に存在するが、HEM Timeline / notice のいずれにも使用できない。

disabled channel の channel 番号を使った HEM を受信した場合、HEMP flow violation とする。

disabled channel でも chunking banned flag と multi-thread banned flag は定義され、digest の対象に含まれる。

ただし、disabled channel では論理送信が許可されないため、chunking banned flag と multi-thread banned flag は通信時には適用されない。

---

## 19. limits と flow semantics

flow semantics は、agreement 済み limits と関係する。

この文書では、次の limits と flow semantics の関係を扱う。

```text
- open_threads.per_channel
- open_threads.total
- role.post.hem_count
- role.reply.hem_count
- role.notice.hem_count
- role.post.payload_total_length
- role.reply.payload_total_length
- role.notice.payload_total_length
```

limits body の field 定義と詳細は、`06-agreement-limits.md` で定義する。

### 19.1 open_threads

`open_threads.per_channel` は、1つの application channel 内で同時に open 状態にできる HEM Thread 数の上限である。

`open_threads.total` は、application channel 上で HEMP session 全体として同時に open 状態にできる HEM Thread 数の上限である。

notice は HEM Thread を open しない。

notice で使われる thread 番号は、`open_threads.per_channel` / `open_threads.total` の対象に含めない。

### 19.2 role.* limits

`role.post.hem_count` は、1つの論理 post 送信で使用できる HEM 数の上限である。

`role.reply.hem_count` は、1つの論理 reply 送信で使用できる HEM 数の上限である。

`role.notice.hem_count` は、1つの論理 notice 送信で使用できる HEM 数の上限である。

`role.post.payload_total_length` は、1つの論理 post 送信で使用できる Payload 合計長の上限である。

`role.reply.payload_total_length` は、1つの論理 reply 送信で使用できる Payload 合計長の上限である。

`role.notice.payload_total_length` は、1つの論理 notice 送信で使用できる Payload 合計長の上限である。

notice は HEM Thread を open しないが、論理 notice 送信の HEM 数と Payload 合計長は、`role.notice.hem_count` / `role.notice.payload_total_length` の対象になる。

---

## 20. Standard Data Body と flow semantics

Standard Data Body は、HEMP application channel 上の post / reply / notice 論理送信として運ばれる。

Standard Data Body は、HEMP flow semantics の `seq`、`end`、`abort`、limits、chunking banned flag に従う。

Standard Data Body の分割は、HEMP flow semantics の `end`、`seq`、`abort` に従う。

Standard Data Body は、これらの flow semantics を変更しない。

Standard Data Body の詳細な body rules は、`08-data-body-rules.md` で定義する。

---

## 21. HEMP flow violation

次は、代表的な HEMP flow violation である。

```text
- disabled channel を使う
- direction / role が channel の service provider side 規則に違反する
- timeline banned flag = true の channel で post / reply を受信する
- notice banned flag = true の channel で notice を受信する
- chunking banned flag = true の channel で end = false の HEM を受信する
- multi-thread banned flag = true の channel の HEM Timeline で thread != 1 を使う
- reply が対応 post と異なる channel + thread を使う
- post 完了前に reply を開始する
- reply 完了前に同じ channel + thread 上で次の post を開始する
- post.close と reply.close が一致しない
- post.abort = true に対して reply.abort = false を返す
- HEM Timeline の seq に欠番、重複、順序違い、または wrap-around がある
- notice の seq に欠番、重複、順序違い、または wrap-around がある
- 終端済みの論理送信に対して継続 HEM を受信する
- reply を未open状態、closing状態、または closed状態の HEM Thread に紐づける
- notice を未open状態、closing状態、または closed状態の HEM Thread に紐づける
- open 状態または closing 状態の thread 番号を、新しい HEM Thread の開始として使用する
```

failure response policy の詳細は、`09-failure-handling.md` で定義する。

---

## 22. この文書で扱わないもの

この文書では、次を定義しない。

```text
- HEM Length Header の形式
- HEM Payload encoding の詳細
- Core Header field の単体定義
- end / close / abort の組み合わせ表
- Channel Table / digest の canonical byte sequence
- agreement body / limits body の詳細
- protocol channel body の詳細
- Standard Data Body の詳細
- failure response policy の詳細
- Transport Binding の具体仕様
```
