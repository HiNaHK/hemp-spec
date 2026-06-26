# Transport Binding との関係

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Core protocol specification

---

## 1. この文書の目的

この文書は、HEMP Core と Transport Binding の関係を定義する。

この文書では、Transport Binding の位置づけ、HEMP Core と Transport Binding の責務分界、HEM frame と transport の関係、complete HEM frame の扱い、`direction` と HEM delivery path の一致、ordered delivery unit と HEMP Core flow semantics の関係、transport message payload / batching の扱い、HEMP session と Transport Binding instance の関係、transport failure、`abort` と transport failure の境界、および Standard Data Body と Transport Binding の関係を扱う。

Transport Binding の基本モデル、Profile、Local Process IPC Binding の詳細は、`specs/transport-binding/` で定義する。

---

## 2. Transport Binding の位置づけ

HEMP Core は、特定の transport 方式を要求しない。

ただし、HEMP を実際に送受信するには、Transport Binding が定義する方法で HEM frame を transport 上に運ぶ。

Transport Binding は、HEMP を具体的な transport 上で送受信するための仕様領域である。

Transport Binding は、HEMP Core が定義する HEM frame を transport 上でどう送受信するかを定義する。

Transport Binding は、HEMP Core semantics を再定義しない。

Transport Binding は、HEMP Core semantics を緩和、変更、または上書きしてはならない。

---

## 3. HEMP Core が定義するもの

HEMP Core は、HEMP protocol の基本構造と意味論を定義する。

HEMP Core は、少なくとも次を定義する。

```text
- HEM frame
- HEM Length Header
- HEM Payload
- HEM Header
- HEM Body
- Core Header semantics
- channel / thread / seq / role / direction / end / close / abort
- Channel Table / digest
- HEM Thread
- HEM Timeline
- post / reply / notice
- agreement / limits
- protocol channels
- Standard Data Body
- failure classification
- failure response policy
```

Transport Binding は、これらを再定義しない。

---

## 4. Transport Binding が定義するもの

Transport Binding は、HEMP Core が定義した HEM frame を transport 上で運ぶための規則を定義する。

Transport Binding は、少なくとも次を定義する。

```text
- HEMP を運ぶ transport が満たすべき性質
- Host -> Engine HEM delivery path
- Engine -> Host HEM delivery path
- ordered delivery unit
- HEM frame を ordered delivery unit へ mapping する方法
- HEM frame bytes の運び方
- HEM Header の direction と HEM delivery path の対応
- HEMP session と Transport Binding instance の対応
- transport failure と HEMP session の関係
```

Transport Binding は、下位 transport の接続確立、接続維持、信頼性、順序保証、双方向性、切断、I/O error、timeout、handshake、認証、暗号化、再接続などを扱ってよい。

HEMP Core は、これらの具体方式を定義しない。

---

## 5. HEM frame と transport

Transport Binding は、HEMP Core が定義する HEM frame を transport 上で運ぶ。

HEM frame は次の構造を持つ。

```text
HEM frame
  = HEM Length Header + HEM Payload
```

Transport Binding は、HEM frame をこの形のまま運ばなければならない。

Transport Binding は、HEM Length Header を省略してはならない。

Transport Binding は、HEM Payload だけを独立して運ぶ仕様ではない。

Transport Binding ごとの差分は、HEM frame または HEM frame sequence を transport 上でどう運ぶかに限定する。

`abort = true` の HEM も、通常の HEM frame と同じ形式で運ぶ。

Transport Binding は、`abort = true` の HEM から HEM Length Header を省略してはならない。

---

## 6. complete HEM frame

HEMP Core は、complete HEM frame を前提に Core semantics を処理する。

complete HEM frame は、HEM Length Header と、その HEM Length Header が示す長さの HEM Payload bytes が揃った HEM frame である。

Transport Binding は、下位 transport から complete HEM frame を構成する責任を持つ。

HEMP 実装は、complete HEM frame を構成してから、HEMP Core の処理へ渡す。

```text
complete HEM frame
  ↓
HEM Payload format validation
  ↓
HEM Header validation
  ↓
HEMP flow validation
  ↓
Body Contract validation
```

下位 I/O では partial read / partial write が発生してよい。

ただし、HEMP Core の処理は complete HEM frame 単位で行う。

complete HEM frame として構成できない場合は、HEMP framing failure または transport failure の境界に従って扱う。

HEM frame の内部に、別の HEM frame または HEM 以外の byte 列を挿入してはならない。

---

## 7. HEM frame sequence

HEM frame sequence は、1つの ordered delivery unit の内部で運ばれる、1つ以上の complete HEM frame の列である。

```text
HEM frame sequence
  = HEM frame
  または
  = HEM frame + HEM frame + ... + HEM frame
```

HEM frame 同士を連結してよい。

ただし、1つの HEM frame の内部に別の HEM frame を挿入してはならない。

有効な例:

```text
[Length Header A][Payload A][Length Header B][Payload B]
```

不正な例:

```text
[Length Header A][Length Header B][Payload A][Payload B]

[Length Header A][Payload A の前半][Length Header B][Payload B][Payload A の後半]
```

`abort = true` の HEM も、HEM frame sequence 内では通常の complete HEM frame である。

`abort = true` は、HEM frame sequence の終端、ordered delivery unit の終端、transport stream の終端、または Transport Binding instance の終端を意味しない。

---

## 8. direction と HEM delivery path

Transport Binding は、HEMP session 内で使用する HEM delivery path を定義しなければならない。

Transport Binding は、少なくとも次の2つの HEM delivery path を定義する。

```text
Host -> Engine HEM delivery path
Engine -> Host HEM delivery path
```

HEM Header の `direction` は、Transport Binding が定義する HEM delivery path と一致しなければならない。

```text
direction = to_engine:
  Host -> Engine HEM delivery path で送信する。

direction = to_host:
  Engine -> Host HEM delivery path で送信する。
```

### 8.1 送信側の責任

送信側は、HEM Header の `direction` と一致する HEM delivery path へ HEM frame を送信しなければならない。

Host は、`direction = to_engine` の HEM を Host -> Engine HEM delivery path へ送信する。

Engine は、`direction = to_host` の HEM を Engine -> Host HEM delivery path へ送信する。

Host は、`direction = to_host` の HEM を送信してはならない。

Engine は、`direction = to_engine` の HEM を送信してはならない。

### 8.2 受信側の責任

受信側は、HEM frame を受信した HEM delivery path と、HEM Header の `direction` が一致していることを検証しなければならない。

Engine が Host -> Engine HEM delivery path で受信した HEM では、`direction` は `to_engine` でなければならない。

Host が Engine -> Host HEM delivery path で受信した HEM では、`direction` は `to_host` でなければならない。

一致しない場合、HEMP header validation failure とする。

### 8.3 HEM delivery path と物理 transport

HEM delivery path は、Transport Binding 上の論理的な delivery path である。

HEM delivery path は、物理 transport の本数と一致しなくてよい。

Host -> Engine HEM delivery path と Engine -> Host HEM delivery path は、1本の双方向 transport で実現してもよい。

または、2本の単方向 transport の組で実現してもよい。

または、複数の ordered delivery unit の集合で実現してもよい。

---

## 9. ordered delivery unit と HEMP Core flow semantics

ordered delivery unit は、Transport Binding が HEM frame sequence を信頼性付き・順序付きで配送する単位である。

各 ordered delivery unit は、Host -> Engine HEM delivery path または Engine -> Host HEM delivery path のいずれか一方に属する。

1つの ordered delivery unit の中に、Host -> Engine HEM delivery path と Engine -> Host HEM delivery path の両方の HEM frame を混在させてはならない。

Transport Binding は、各 HEM delivery path に1つ以上の ordered delivery unit を定義してよい。

異なる ordered delivery unit の間に、Transport Binding 基本モデルとしての単一グローバル順序は要求しない。

同一の論理 post 送信、論理 reply 送信、または論理 notice 送信に属する HEM は、単一の ordered delivery unit 上で送受信されなければならない。

Transport Binding は、同一 logical send に属する HEM を複数の ordered delivery unit へ分散してはならない。

Transport Binding は、HEMP Core の `seq` を、異なる ordered delivery unit 間の transport-level ordering、reordering、または repair のために使用してはならない。

1つの ordered delivery unit 上に、複数の logical send に属する HEM frame が complete HEM frame 単位で interleave されることは、適用される Transport Binding Profile が許可する範囲で認める。

Transport Binding は、複数の logical send が同一 ordered delivery unit 上で interleave される場合でも、HEMP Core が定義する flow semantics、seq validation、end / close / abort semantics、および limits を変更してはならない。

Transport Binding は、この文書に列挙していない HEMP Core 上の順序制約も変更してはならない。

---

## 10. transport message payload / batching

Transport Binding が transport message boundary を持つ transport を使う場合、1つの transport message payload に複数の HEM frame を含めてよい。

その場合でも、個々の HEM frame 境界は HEM Length Header によって判断する。

transport message boundary そのものを HEM frame boundary として扱ってはならない。

1つの transport message payload に複数の HEM frame を含める場合、その transport message payload は1つの ordered delivery unit に属するものとして扱う。

その transport message payload 内のすべての HEM frame は、その ordered delivery unit が属する HEM delivery path で受信されたものとして扱う。

したがって、同じ transport message payload 内の各 HEM Header の `direction` は、その transport message payload が属する ordered delivery unit の HEM delivery path と一致しなければならない。

一致しない場合、該当 HEM は HEMP header validation failure とする。

---

## 11. Transport Binding が定義してよいもの

Transport Binding は、次を定義してよい。

```text
- ordered delivery unit をいくつ定義するか。
- 各 ordered delivery unit が属する HEM delivery path。
- HEM frame sequence を ordered delivery unit 上でどう運ぶか。
- HEM frame をどの ordered delivery unit へ mapping するか。
- protocol channel の HEM をどの ordered delivery unit へ mapping するか。
- application channel の HEM をどの ordered delivery unit へ mapping するか。
- 1つの ordered delivery unit 上で複数の logical send を complete HEM frame 単位で interleave するか。
- HEM frame sequence を byte stream 上にどう流すか。
- HEM frame sequence を transport message payload にどう載せるか。
- transport message payload max。
- transport failure の検出、通知、診断情報の扱い。
- 正常終了後の transport close / transport 再利用の扱い。
```

Transport Binding は、`abort = true` の HEM を通常の HEM frame としてどの ordered delivery unit へ mapping するかを定義してよい。

ただし、その mapping は HEMP Core が定義する `abort` の意味、seq 順序、論理送信終端状態、および failure classification を変更してはならない。

同一 logical send に属する HEM を複数の ordered delivery unit へ分散する mapping は定義してはならない。

---

## 12. Transport Binding が変更してはならないもの

Transport Binding は、次を変更してはならない。

```text
- HEM Length Header の形式
- HEM Length Header の意味
- HEM Payload の位置づけ
- HEM Header fields
- HEM Body の位置づけ
- channel / thread / seq / role / direction の意味
- end / close / abort の意味
- HEM Timeline の意味
- HEMP flow semantics
- agreement / limits の意味
- protocol channel Payload length 上限
- unknown field の受信規則
- failure classification
- failure response policy
- Standard Data Body の data body rules
```

Transport Binding は、`abort = true` を transport reset、stream reset、transport close、ordered delivery unit の終了、または transport failure として再定義してはならない。

Transport Binding は、transport reset、stream reset、transport close、または transport failure を、`abort = true` の HEM として合成してはならない。

---

## 13. HEMP session と Transport Binding instance

Transport Binding instance は、HEMP session を運ぶための transport 上の通信単位である。

Transport Binding instance は、1つ以上の ordered delivery unit の集合として構成されてよい。

1つの Transport Binding instance は、同時に1つの HEMP session だけを運ぶ。

同一の Transport Binding instance 上で、複数の HEMP session を同時に多重化してはならない。

同じ HEMP session 内で、再 Agreement や再 negotiation を行ってはならない。

再度 HEMP 通信を行う場合は、新しい HEMP session として開始し、再び `agreement` から実行しなければならない。

### 13.1 Transport Binding instance の意味

Transport Binding instance の具体的な形は、具体的な Transport Binding または Transport Binding Profile によって異なる。

Transport Binding instance は、`connection` という語に固定しない。

物理的な接続1本ではなく、複数の通信路の組によって1つの Transport Binding instance を構成する場合があるためである。

Transport Binding instance は、1つ以上の ordered delivery unit の集合として構成されてよい。

複数の ordered delivery unit を持つ場合でも、その Transport Binding instance が同時に運ぶ HEMP session は1つだけである。

### 13.2 Transport Binding instance の再利用

Transport Binding instance を、終了済み HEMP session の後に再利用できるかどうかは、具体的な Transport Binding または実装が定義してよい。

ただし、Transport Binding instance を再利用する場合でも、終了済みの HEMP session を継続してはならない。

再利用後に HEMP 通信を行う場合は、新しい HEMP session として `agreement` から開始しなければならない。

### 13.3 Agreement 成立前の ordered delivery unit

Agreement 成立前でも、Transport Binding は HEM frame を運ぶ。

ただし、Agreement 成立前に有効な HEM は `agreement` post / reply に限られる。

複数の ordered delivery unit を持つ Transport Binding でも、Agreement 成立前に application channel 用の ordered delivery unit で application channel の HEM を送信してはならない。

Agreement 成立前に HEM を送信できる ordered delivery unit は、`agreement` を送受信するために Transport Binding が定義したものに限る。

Agreement 成立前は、protocol channel Payload length 上限が適用される。

Agreement 成立後は、Agreement 済み Application Channel Table、limits、protocol channel rules に従う。

---

## 14. 複数の ordered delivery unit と session-level control

複数の ordered delivery unit を持つ Transport Binding でも、`shutdown` は HEMP session 全体に対する protocol channel である。

Transport Binding は、`shutdown` post を特定の ordered delivery unit だけの終了要求として扱ってはならない。

Transport Binding は、`shutdown` post が、同じ HEMP session 内の他の ordered delivery unit 上の HEM とどのような順序関係を持つかを定義しなければならない。

Transport Binding は、`shutdown` post を送信する前に送信側が全 ordered delivery unit を終了可能な状態にする規則、または `shutdown` を session-level barrier として扱う規則を定義してよい。

いずれの場合でも、HEMP Core が定義する `shutdown` の flow rule は緩和されない。

Transport Binding が `shutdown` に関する順序関係または barrier 規則を定義する場合でも、HEMP Core が定義する Host-side shutdown-ready state、および `result = true` の `shutdown` reply に対する安全不変条件を緩和してはならない。

`cancel` は、対象 application channel + thread を Body で指定する protocol channel である。

複数の ordered delivery unit を持つ Transport Binding でも、`cancel` request を対象 HEM Timeline と同じ ordered delivery unit で運ぶことは必須ではない。

ただし、`cancel` request が対象 HEM Timeline とは異なる ordered delivery unit で運ばれる場合、受信側では `cancel` request が対象 HEM Timeline の開始 HEM より先に処理される可能性がある。

この場合、対象がまだキャンセル可能ではないなら、`cancel` reply body の `result` は `false` になり得る。

`cancel` reply body の `result = true` は、対象 HEM Timeline の完了、正常完了、aborted 終端、または停止済み状態を意味しない。

Transport Binding は、`cancel` reply を対象 HEM Timeline の reply として扱ってはならない。

対象 HEM Timeline の完了状態は、対象 application channel 上の実際の HEM Timeline の post / reply / abort 終端状態によって判断する。

`cancel` を受理した結果として対象 reply が `abort = true` で終端する場合でも、その `abort = true` の HEM は対象 application channel の実際の ordered delivery unit 上で通常の HEM frame として運ばれなければならない。

Transport Binding は、`cancel` request が対象 HEM Timeline を追い越さないようにする規則を定義してよい。

`cancel` request の同時未完了数は、HEMP Core の `body.limits.protocol.cancel.in_flight_requests` に従う。

Transport Binding は、この上限を緩和してはならない。

---

## 15. normal shutdown と transport close

`shutdown` reply body の `result = true` によって、HEMP session は正常終了する。

正常終了後の transport close は、transport failure ではない。

HEMP session 継続中の予期しない transport close は、transport failure である。

`shutdown` 成功後の transport close、transport 再利用、Engine process 終了の具体手順は、Transport Binding または実装側が定義する。

`shutdown` の flow rule は、`07-protocol-channels.md` で定義する。

---

## 16. transport failure と HEMP session

transport failure は、下位 transport、Transport Binding instance、ordered delivery unit、または Transport Binding 実装に起因して、HEMP session を運べなくなる失敗である。

transport failure は、HEMP failure 5分類には追加しない。

transport failure は、HEM を正しく届けられない、または HEMP session を継続できない失敗である。

transport failure の責任は、HEMP Core ではなく、下位 transport、Transport Binding、または実装側が持つ。

HEMP Core は、transport failure の原因解決、再送、再接続、復旧、補償、認証、暗号化、timeout policy を定義しない。

transport failure が発生した場合、現在の HEMP session は継続できない。

複数の ordered delivery unit を持つ Transport Binding では、いずれかの ordered delivery unit の失敗によって、その Transport Binding instance 上の HEMP session を正しく運べなくなった場合、現在の HEMP session は継続不能として扱う。

transport failure 後に、同じ HEMP session を再開してはならない。

transport failure 後に、未完了の HEM Timeline、論理 post 送信、論理 reply 送信、論理 notice 送信、HEM Thread を HEMP 上で完了扱いにしてはならない。

再度 HEMP 通信を行う場合は、新しい HEMP session として開始し、再び `agreement` から実行しなければならない。

HEMP は、transport failure に対する通常の HEM reply を定義しない。

HEMP は、transport failure に対する汎用 error body を定義しない。

HEMP は、transport failure 通知用の protocol channel を定義しない。

transport failure の詳細例や Profile ごとの扱いは、`specs/transport-binding/` で定義する。

---

## 17. HEMP framing failure と transport failure の境界

HEMP framing failure は、受信した bytes または transport message payload を HEM frame として構成できない failure である。

transport failure は、下位 transport、ordered delivery unit、または Transport Binding instance の失敗である。

例:

```text
HEM Length Header の値が 0:
  HEMP framing failure。

Agreement 成立前に HEM Length Header の値が protocol channel Payload length 上限を超える:
  HEMP framing failure。

有限の transport message payload、または終了した byte stream において、
HEM Length Header として必要な 4 bytes を構成できない:
  HEMP framing failure。

宣言された Payload length 分の bytes を HEM frame として構成できない:
  HEMP framing failure。

Payload 読み取り中に transport が切断された:
  根本原因は transport failure。
  HEMP session は継続不能。
```

Byte Stream Profile では、受信処理中に一時的な未完了 bytes が存在しても、下位 transport が継続している限り、それだけを HEMP framing failure として扱わない。

実装は、HEMP 受信処理上の結果として HEMP framing failure を記録してよい。

ただし、根本原因が transport 切断、I/O error、timeout などである場合、診断情報として transport failure を保持してよい。

この場合でも、transport failure の責任を HEMP Core は持たない。

---

## 18. abort と transport failure の境界

`abort = true` は、HEMP Core が定義する sender-side logical-send abort の表現である。

```text
abort = true:
  有効な HEM を complete HEM frame として送れる状態で、
  論理送信を aborted 状態として終端する Core 上の表現。

transport failure:
  HEM を正しく届けられない、または HEMP session を継続できない失敗。
```

Transport Binding は、`abort = true` の HEM を通常の complete HEM frame として運ぶ。

受信側が `abort = true` の HEM を complete HEM frame として構成し、HEMP Core の検証と状態遷移へ渡した場合、その論理送信は HEMP Core semantics に従って aborted 状態で終端する。

一方、送信側が `abort = true` の HEM を送信しようとしている途中で transport failure が発生し、受信側がその HEM を complete HEM frame として構成できなかった場合、その `abort` は HEMP Core 上では成立しない。

この場合、根本原因は transport failure であり、現在の HEMP session は継続不能として扱う。

Transport Binding は、未完了の論理 post 送信、論理 reply 送信、論理 notice 送信、または未完了 HEM Timeline を、transport failure を根拠に aborted 状態で完了済みとして扱ってはならない。

Transport Binding は、transport close、stream close、FIN、RST、I/O error、timeout、process 終了、または ordered delivery unit の予期しない終了を、`abort = true` の HEM として合成してはならない。

Transport Binding は、`abort = true` の HEM を送信するために transport close、stream close、FIN、RST、または ordered delivery unit の終了を要求してはならない。

Transport Binding は、`abort = true` を transport reset、stream reset、transport close、ordered delivery unit の終了、または transport failure として再定義してはならない。

---

## 19. Standard Data Body と Transport Binding

Standard Data Body は、HEMP protocol の標準 data body rules である。

Transport Binding は、Standard Data Body HEM を通常の HEM frame として運ぶ。

Standard Data Body の HEM も、Transport Binding が定義する HEM frame boundary、complete HEM frame、direction、ordered delivery unit、transport failure 規則に従う。

Transport Binding は、Standard Data Body の data bytes、empty data、time flush、abort 終端を transport-level signal として再定義してはならない。

Standard Data Body の詳細は、`08-data-body-rules.md` で定義する。

---

## 20. Transport Binding specification との関係

この文書は、HEMP Core と Transport Binding の責務境界を定義する。

`specs/transport-binding/` は、Transport Binding の基本モデル、Profile、具体的な Transport Binding の規則を定義する。

`specs/transport-binding/` では、少なくとも次を扱う。

```text
- Transport Binding 基本モデル
- HEM delivery path
- ordered delivery unit
- HEM frame mapping
- HEM frame sequence
- Byte Stream Profile
- Message Boundary Profile
- Local Process IPC Binding
- transport failure の詳細
```

---

## 21. この文書で扱わないもの

この文書では、次を定義しない。

```text
- Byte Stream Profile の詳細
- Message Boundary Profile の詳細
- Local Process IPC Binding の詳細
- 具体的な OS API / pipe / socket / endpoint の実現方式
- retry / reconnect policy
- timeout 値
- transport authentication / encryption の具体仕様
- HEM Payload encoding の詳細
- protocol channel body の詳細
- Standard Data Body の詳細
- failure response policy の詳細
```
