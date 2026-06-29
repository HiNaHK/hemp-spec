# failure handling

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Core protocol specification

---

## 1. この文書の目的

この文書は、HEMP における failure classification と failure response policy を定義する。

この文書では、HEMP failure の分類、Agreement failure との区別、`abort = true` との関係、transport failure との境界、および failure response policy を扱う。

この文書で定義する failure classification は、HEMP Core semantics において不正な HEM または不正な HEMP protocol state を分類するためのものである。

---

## 2. HEMP failure の基本方針

HEMP failure は、不正な HEM または不正な HEMP protocol state を分類するためのものである。

failure は、失敗が発生した層で分類する。

同じ入力に複数の問題がある場合でも、受信側は観測した層に基づいて failure を分類する。

HEMP failure classification は、汎用 error reply body、error 専用 protocol channel、または汎用 error code 体系を導入するものではない。

---

## 3. failure 分類

HEMP failure は、次の5分類に分ける。

```text
1. HEMP framing failure
2. HEMP payload format failure
3. HEMP header validation failure
4. HEMP flow violation
5. HEM Body Contract failure
```

HEM Payload encoding における decode / validation failure は、この5分類へ対応づける。

HEM Payload encoding 上の具体的な decode rule、field number、wire type、oneof branch、schema 定義は、Protobuf Encoding specification で定義する。

この文書では、`Protocol Error`、`Data Body Body Contract failure`、transport failure、または Transport Binding violation を HEMP failure の追加分類として定義しない。

---

## 4. HEMP framing failure

HEMP framing failure は、HEM Length Header または byte stream 上の framing に関する failure である。

HEMP framing failure は、HEM Payload bytes を読み終える前、または complete HEM frame を構成できない段階で発生する。

次は HEMP framing failure である。

```text
- HEM Length Header を 4 bytes 読めない。
- HEM Length Header の値が 0 である。
- Agreement 成立前に HEM Length Header の値が protocol channel Payload length 上限を超えている。
- 宣言 Payload length 分の HEM Payload bytes を読めない。
```

Agreement 成立前に、HEM Length Header の値が protocol channel Payload length 上限を超える場合、受信側はその値に従った Payload bytes の読み取りを試みず、HEMP framing failure とする。

---

## 5. HEMP payload format failure

HEMP payload format failure は、HEM Payload bytes は読めたが、HEM Payload として decode または構造解釈できない failure である。

次は HEMP payload format failure である。

```text
- Payload bytes が HEM Payload encoding として decode できない。
- wire format が不正である。
- known field の wire type が不正である。
- embedded message field が decode できない。
- HEM Header に相当する構造が存在しない。
- known body branch が存在しない。
```

HEMP payload format failure は、HEM Payload encoding 上の形式不正を表す。

unknown field は、必須既知 field の代替にはならない。

unknown field の存在のみを理由に、HEMP payload format failure としてはならない。

known body branch が存在しない場合、受信側は HEM Body Contract validation を適用しない。

この場合、その HEM は HEMP payload format failure として扱う。

known body branch が存在しないことを、protocol channel Body Contract failure または application channel Body Contract failure として扱ってはならない。

---

## 6. HEMP header validation failure

HEMP header validation failure は、HEM Payload の基本形は有効だが、HEM Header として不正な failure である。

次は HEMP header validation failure である。

```text
- Core Header field が欠落している。
- direction が unspecified または未定義値である。
- role が unspecified または未定義値である。
- channel = 0 である。
- thread = 0 である。
- seq = 0 である。
- end の semantic presence がない。
- close の semantic presence がない。
- abort の semantic presence がない。
- direction が Transport Binding 上の HEM delivery path と一致しない。
- end = false かつ close = true である。
- abort = true かつ end = false である。
- notice で close = true である。
```

HEM Header の unknown field は、HEMP header validation failure ではない。

HEM Header fields の詳細は、`03-hem-header-fields.md` で定義する。

---

## 7. HEMP flow violation

HEMP flow violation は、単体 HEM の構造や Header 値は有効だが、HEMP protocol state transition として不正な failure である。

次は HEMP flow violation である。

```text
- Agreement 成立前に agreement 以外の有効 HEM を受信した。
- Agreement 成立後に agreement を使用した。
- Agreement 成立後に Protocol Channel Table に存在しない protocol channel を使用した。
- Agreement 成立後に protocol channel HEM が protocol channel Payload length 上限を超えた。
- Application Channel Table が0件の状態で application channel を使用した。
- channel > 0 が Agreement 済み Application Channel Table の範囲外である。
- disabled channel を使用した。
- direction / role が channel の service provider side 規則に違反した。
- timeline banned flag = true の channel で post / reply を使用した。
- notice banned flag = true の channel で notice を使用した。
- chunking banned flag = true の channel で end = false の HEM を使用した。
- multi-thread banned flag = true の channel の HEM Timeline で thread != 1 を使用した。
- reply が対応 post と異なる channel + thread を使用した。
- reply が対応 post 完了前に開始した。
- 同じ channel + thread 上で reply 完了前に次の post を開始した。
- seq の欠番、順序違い、または wrap-around がある。
- aborted 状態で終端済みの論理送信に対して、同じ論理送信の継続 HEM を受信した。
- post.abort = true の後に、対応 reply が abort = false で完了した。
- post.close と reply.close が一致しない。
- protocol channel で、その protocol channel が許可していない abort = true を使った。
- 未open状態、closing状態、closed状態の thread を不正に使用した。
- Agreement 済み body.limits を超えた。
- shutdown post 送信後、shutdown reply 受信前に Host が新たな HEM を送信した。
- Engine 側に未完了の論理送信または継続 HEM が残っている状態で、Engine が result = true の shutdown reply を送信した。
- result = true の shutdown reply 送信後に Engine が新たな HEM を送信した。
```

HEMP flow semantics の詳細は、`04-flow-semantics.md` で定義する。

---

## 8. Agreement 済み limits 超過

Agreement 済み `body.limits` 超過は、HEMP flow violation とする。

次は Agreement 済み `body.limits` 超過である。

```text
- encoded HEM Payload length > body.limits.hem_payload_length
- logical send HEM count > role.<role>.hem_count
- logical send encoded HEM Payload length total > role.<role>.payload_total_length
- open_threads.per_channel 超過
- open_threads.total 超過
- protocol.cancel.in_flight_requests 超過
```

`body.limits` の詳細は、`06-agreement-limits.md` で定義する。

---

## 9. HEM Body Contract failure

HEM Body Contract failure は、HEM Body が、その `channel / direction / role / abort` に対応する Body Contract に合わない failure である。

protocol channel の場合、Body Contract は HEMP が定義する。

application channel の場合、Body Contract は該当する application channel の Body Contract が定義する。

次は HEM Body Contract failure である。

```text
- header と body branch が一致しない。
- protocol channel で期待される body branch と異なる branch を使用した。
- protocol channel で application_body を使用した。
- protocol channel で abort_body を使用した。
- application channel で protocol body branch を使用した。
- abort = false で abort_body を使用した。
- abort = true で application_body を使用した。
- protocol body message 内の必須 field が欠落している。
- protocol body message 内の field 値が範囲外である。
- Standard Data Body rules に違反している。
```

Body branch mismatch は、HEM Body Contract failure とする。

---

## 10. protocol channel Body Contract failure

protocol channel body において、HEMP が定義する必須 field が欠落している、値が不正である、範囲外である、または組み合わせが不正である場合は、各 protocol channel の Body Contract failure として扱う。

protocol channel Body Contract failure は、known body branch が存在し、HEM Header validation および HEMP flow validation が成功した後に適用する。

known body branch が存在しない場合は、HEM Body Contract failure ではなく、HEMP payload format failure とする。

次は protocol channel Body Contract failure である。

```text
- ping reply body の result / message が欠落している。
- shutdown post body の reason が欠落している。
- shutdown reply body の result / message が欠落している。
- cancel post body の target / reason が欠落している。
- cancel post body の target.channel / target.thread が欠落している。
- cancel post body の target.channel / target.thread の値や範囲が不正である。
- cancel reply body の result / message が欠落している。
- protocol channel reply body の message が空文字列である。
```

protocol channel body の unknown field は、HEM Body Contract failure ではない。

受信側は、protocol channel body の unknown field を意味として無視する。

protocol channel の詳細は、`07-protocol-channels.md` で定義する。

---

## 11. application channel Body Contract failure

application channel body の扱いは、該当する application channel の Body Contract が定義する。

Standard Data Body を使う application channel では、Standard Data Body rules に違反する body は、HEM Body Contract failure として扱う。

この文書では、Standard Data Body rules に固有の failure 条件を、独立した HEMP failure 分類としては定義しない。

`abort = true` の body は、通常完了時の application body として扱ってはならない。

`abort = true` の body に対して、通常完了時の Application Body Contract を、その論理送信の正常 body 検証として適用してはならない。

abort body 内の diagnostic 情報を解釈できない場合でも、Header の `abort = true` によって表される aborted 終端状態は変わらない。

application channel body における unknown field の扱いは、対応する Body Contract が定義する。

Standard Data Body の詳細は、`08-data-body-rules.md` で定義する。

---

## 12. unknown field と failure

unknown field の存在のみを理由に、HEMP failure としてはならない。

unknown field は、必須既知 field の代替にはならない。

protocol channel body の unknown field は、意味として無視する。

HEM Header の unknown field は、HEMP header validation failure ではない。

HEM Payload の unknown field は、HEMP payload format failure ではない。

application channel body の unknown field の扱いは、対応する Body Contract が定義する。

HEMP 実装は、相手が解釈する必要のある情報を unknown field として送ってはならない。

---

## 13. abort = true と failure

正しい状態で `abort = true` の HEM を受信したこと自体は、HEMP failure ではない。

`abort = true` は、正常な HEMP flow 内で表現される、論理送信単位の異常終端状態である。

`abort = true` は HEMP flow violation ではない。

`abort = true` は transport failure ではない。

abort body 内の diagnostic 情報を解釈できない場合でも、Header の `abort = true` によって表される aborted 終端状態は変わらない。

ただし、`abort = true` を許可しない文脈で使用した場合は、該当する failure 分類に従う。

例:

```text
- abort = true かつ end = false は HEMP header validation failure。
- protocol channel で、その protocol channel が許可していない abort = true を使った場合は HEMP flow violation。
- abort = true で application_body を使用した場合は HEM Body Contract failure。
```

---

## 14. Agreement failure と HEMP failure

Agreement body または agreement reply body が Body Contract として valid であり、かつ Agreement 成立条件を1つでも満たさない場合は Agreement failure とする。

Agreement failure は HEMP failure 5分類には含めない。

Agreement failure は、次のいずれでもない。

```text
HEMP framing failure
HEMP payload format failure
HEMP header validation failure
HEMP flow violation
HEM Body Contract failure
```

Agreement failure 時、Engine は agreement reply を返す。

Agreement failure 時、channel > 0 の通常 HEM 通信は開始しない。

Agreement failure 時、現在の HEMP session は Agreement failure により終了したものとして扱う。

Agreement failure 後の処理、再接続、process の扱い、利用者向け表示、診断情報の扱いは、HEMP を使う application 側が扱う。

Agreement failure の詳細は、`06-agreement-limits.md` で定義する。

---

## 15. transport failure との境界

transport failure は、HEMP failure 5分類には追加しない。

transport failure は、HEM を正しく届けられない、または HEMP session を継続できない失敗である。

transport failure を `abort = true` の代替として扱ってはならない。

`abort = true` の HEM は、complete HEM frame として送受信されなければならない。

transport failure 後は、未完了の HEM Timeline や論理送信を HEMP 上で完了扱いにしてはならない。

transport failure 後、同じ HEMP session を再開してはならない。

Transport Binding との関係は、`10-transport-binding-relationship.md` で定義する。

---

## 16. failure response policy の基本方針

HEMP は、汎用 error reply body を定義しない。

HEMP は、error 専用 protocol channel を定義しない。

HEMP は、汎用 error code 体系を定義しない。

failure response は、failure 分類と、その時点で reply 可能な Body Contract に従う。

---

## 17. 通常 reply を返さない failure

次の failure には、通常の HEM reply を返さない。

```text
- HEMP framing failure
- HEMP payload format failure
- HEMP header validation failure
- HEMP flow violation
```

これらの failure が発生した HEM は、HEM Timeline 上の有効な post として state commit しない。

したがって、これらの failure に対して対応 reply を生成しない。

これらの failure が発生した場合、現在の HEMP communication state は失敗状態として扱う。

HEMP は、これらの failure に対する汎用 error reply body を定義しない。

HEM Payload encoding の decode / validation failure に対する専用の generic error body、error oneof branch、error protocol channel、または generic error code 体系は定義しない。

HEM Payload encoding の decode / validation failure は、既存の HEMP failure 5分類へ対応づけ、返答方針はこの文書に従う。

failure 後の処理、再接続、process の扱い、利用者向け表示、診断情報の扱いは、HEMP を使う application または実装側が扱う。

---

## 18. Body Contract failure の返答方針

HEM Body Contract failure は、HEM Payload structural validation、HEM Header validation、および HEMP flow validation が成功した後に適用される。

Header validation または flow validation が失敗している HEM については、Body Contract validation を適用しない。

Header validation および flow validation が成功した post について、Body Contract validation が失敗した場合でも、対応 reply を返す Body Contract が定義されている場合、その post は HEM Timeline 上の post として state commit する。

この state commit は、対応 reply を HEMP flow semantics 上の reply として成立させるために行う。

この場合の reply は、汎用 error reply ではなく、該当 channel の Body Contract が定義する通常の reply bodyである。

reply 完了後の HEM Thread state は、通常の `end` / `close` / `abort` 規則に従う。

framing failure、payload format failure、header validation failure、または flow violation が発生した HEM は、HEM Timeline 上の有効な post として state commit しない。

この state commit 規則は、post に対して適用する。

この文書では、notice または reply の Body Contract failure に対して、対応 reply を返すための state commit 規則を定義しない。

### 18.1 agreement

`agreement` の Body Contract failure では、Header validation および flow validation が成功し、agreement reply を返せる状態であれば、receiver はその agreement post を HEM Timeline 上の post として state commit し、`agreement` reply body を使って failure を返してよい。

この reply は、汎用 error reply ではない。

この reply は、agreement reply Body Contract に基づく通常の reply body である。

Body Contract として valid ではない Agreement 項目は、対応する Agreement 結果を failure として返してよい。

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

### 18.2 ping / shutdown / cancel

`ping` / `shutdown` / `cancel` の post body が Body Contract に合わない場合、その failure は各 protocol channel の Body Contract failure とする。

Header validation および flow validation が成功し、対応 reply を返せる状態であれば、receiver はその post を HEM Timeline 上の post として state commit し、各 protocol channel の専用 reply body で `result = false` を返してよい。

この reply は、汎用 error reply ではない。

この reply は、該当 protocol channel の Body Contract に基づく通常の reply body である。

`shutdown` Body Contract failure の場合、HEMP session は正常終了しない。

`cancel` Body Contract failure の場合、cancel request は受け付けられない。

### 18.3 application channel

application channel の Body Contract failure は、該当する application channel の Body Contract が扱う。

Standard Data Body を使う application channel では、Standard Data Body rules に関する HEM Body Contract failure 条件に従う。

Standard Data Body rules の詳細は、`08-data-body-rules.md` に従う。

---

## 19. failure 後の扱い

failure 後の処理、再接続、process の扱い、利用者向け表示、診断情報の扱いは、HEMP を使う application または実装側が扱う。

HEMP Core は、failure 後の retry / reconnect policy を定義しない。

同じ HEMP session を継続できるかどうかは、failure 分類と transport 状態に従う。

---

## 20. この文書で扱わないもの

この文書では、次を定義しない。

```text
- HEM Length Header の詳細構造
- HEM Payload encoding の field number / wire type / schema
- Core Header field の個別定義
- HEM Thread lifecycle 全体
- Channel Table / digest
- agreement body / limits body の詳細
- ping / shutdown / cancel body の詳細
- Standard Data Body の詳細
- Transport Binding の具体仕様
- retry / reconnect policy
- 汎用 error body
- error 専用 protocol channel
- 汎用 error code 体系
```
