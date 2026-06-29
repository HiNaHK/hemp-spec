# HEMP Transport Profiles

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Transport Binding specification

## 1. Purpose

この文書は、HEMP Transport Binding Profile を定義する。

Transport Binding Profile は、HEM frame sequence を transport 上でどのように区切り、どのように complete HEM frame として HEMP Core validation へ渡すかを定義する。

この文書では、次を定義する。

```text
Transport Binding Profile
Profile と concrete Transport Binding の関係
HEM frame boundary by Profile
HEM frame interleave
Byte Stream Profile
Message Boundary Profile
Message Boundary Profile payload max
Profile selection
Local Process IPC Binding との関係
```

この文書は、HEMP Core semantics を再定義しない。

この文書は、HEM Payload encoding を再定義しない。

この文書は、Transport Binding instance、ordered delivery unit、transport failure、retry policy、または reconnect policy の詳細を再定義しない。

## 2. Transport Binding Profile

Transport Binding Profile は、HEM frame sequence を transport 上でどのように区切り、どのように complete HEM frame として HEMP Core validation へ渡すかを定義するモデルである。

Transport Binding Profile は、Transport Binding が使用する HEM frame boundary rule と frame construction rule を定義する。

Transport Binding Profile は、HEMP Core semantics を変更しない。

Transport Binding Profile は、HEM Header field semantics を変更しない。

Transport Binding Profile は、HEM Length Header の意味を変更しない。

Transport Binding Profile は、HEM Payload encoding を変更しない。

Transport Binding Profile は、HEM Body semantics を変更しない。

Transport Binding Profile は、transport 上の boundary rule と frame construction rule を定義するものであり、HEMP Core の logical send、flow semantics、agreement、limits、failure classification を再定義するものではない。

## 3. Profile and Concrete Transport Binding

Concrete Transport Binding は、使用する Transport Binding Profile を定義しなければならない。

Concrete Transport Binding は、各 ordered delivery unit がどの Profile に従うかを定義しなければならない。

Concrete Transport Binding は、すべての ordered delivery unit に同じ Profile を使用してよい。

Concrete Transport Binding は、必要であれば、HEM delivery path または ordered delivery unit ごとに異なる Profile を使用してよい。

ただし、その場合でも、どの ordered delivery unit がどの Profile に従うかを明示しなければならない。

Profile の選択は、受信側が HEM frame boundary を判断できる形で定義されなければならない。

Profile が不明な状態で受信 data を解釈してはならない。

Concrete Transport Binding は、Profile の選択を HEMP Core semantics の暗黙の一部として扱ってはならない。

## 4. HEM Frame Boundary by Profile

HEM frame boundary は、適用される Transport Binding Profile に従って判断する。

Transport Binding は、HEM frame boundary を判断するために、どの Profile が適用されるかを知っていなければならない。

Transport Binding は、次を、それ自体として HEM frame boundary とみなしてはならない。

```text
read boundary
write boundary
flush boundary
buffer boundary
line break
implementation chunk boundary
transport message boundary
```

Transport Binding は、Profile が定義しない境界を、transport-specific な都合によって HEM frame boundary として追加してはならない。

Transport Binding は、Profile が定義する boundary rule に反して、HEM frame を分割、結合、補正、または再解釈してはならない。

Message Boundary Profile における transport message boundary は、HEM frame boundary ではない。

Message Boundary Profile における transport message boundary は、transport message payload が含む HEM frame sequence の外側の frame construction unit boundary である。

## 5. HEM Frame Interleave

HEM frame interleave は、1つの ordered delivery unit 上の HEM frame sequence に、複数の logical send、channel、thread、または role に属する HEM frame が混在することである。

Transport Binding Profile は、HEM frame interleave を許すかどうかを定義してよい。

HEM frame interleave を許す場合でも、interleave は complete HEM frame 単位に限られる。

Transport Binding は、1つの HEM frame の内部に、別の HEM frame を interleave してはならない。

Transport Binding は、1つの HEM frame の内部に、HEMP 外の byte sequence、transport control data、log output、diagnostic output、または別 logical send の data を挿入してはならない。

Transport Binding Profile が HEM frame interleave を許す場合でも、HEMP Core の flow semantics、channel rules、thread rules、seq rules、end / close / abort rules、および limits は変更されない。

どの HEM frame がどの logical send に属するかは、Transport Binding ではなく、HEMP Core が定義する Header fields と flow semantics によって判断する。

## 6. Byte Stream Profile

Byte Stream Profile は、HEM frame sequence を連続した byte stream として扱う Transport Binding Profile である。

Byte Stream Profile では、ordered delivery unit は、順序保証された byte stream を提供しなければならない。

Byte Stream Profile では、送信側は、各 HEM frame を byte stream 上に順に書き込む。

Byte Stream Profile では、受信側は、byte stream から HEM Length Header と HEM Payload bytes を順に読み取り、complete HEM frame を構成する。

Byte Stream Profile では、HEM frame boundary は HEM Length Header によって判断する。

Byte Stream Profile では、transport の read / write / flush / buffer operation の境界は HEM frame boundary ではない。

Byte Stream Profile では、line break、text line、record separator、delimiter、または implementation chunk boundary を HEM frame boundary として扱ってはならない。

## 7. Byte Stream Profile Frame Boundary

Byte Stream Profile における HEM frame は、次の形式で byte stream 上に連続して現れる。

```text
HEM frame = HEM Length Header + HEM Payload
```

HEM Length Header は、直後に続く HEM Payload bytes の長さを表す。

受信側は、まず HEM Length Header を取得し、次に HEM Length Header が示す length 分の HEM Payload bytes を取得しなければならない。

受信側は、HEM Length Header と、その HEM Length Header が示す HEM Payload bytes を取得できた場合に限り、complete HEM frame を構成できる。

受信側は、次を complete HEM frame として扱ってはならない。

```text
HEM Length Header を取得できていない状態
HEM Length Header の一部だけを取得した状態
HEM Length Header だけを取得した状態
HEM Payload bytes の一部だけを取得した状態
declared length 分の HEM Payload bytes を取得できていない状態
```

HEM Length Header の形式、値、payload length、limits、および failure classification は、HEMP Core および HEMP Payload encoding specification に従う。

Byte Stream Profile は、HEM Length Header に新しい field、flag、encoding selector、checksum、compression marker、または profile marker を追加しない。

## 8. Byte Stream Profile Frame Sequence

Byte Stream Profile では、複数の HEM frame は byte stream 上に連続して並ぶ。

```text
HEM frame 1
HEM frame 2
HEM frame 3
...
```

各 HEM frame は、それぞれの HEM Length Header によって切り出される。

HEM frame 間に delimiter、separator、line break、padding、transport-specific marker、または non-HEM control bytes を挿入してはならない。

Byte Stream Profile では、1回の read operation で次のいずれが発生してもよい。

```text
1つの HEM frame の一部だけを取得する。
1つの complete HEM frame を取得する。
1つの complete HEM frame と次の HEM frame の一部を取得する。
複数の complete HEM frame を取得する。
```

これらの read operation の単位は、HEM frame boundary ではない。

Byte Stream Profile では、1回の write operation で次のいずれが発生してもよい。

```text
1つの HEM frame の一部を書き込む。
1つの complete HEM frame を書き込む。
複数の complete HEM frame を書き込む。
```

これらの write operation の単位は、HEM frame boundary ではない。

実装は、byte stream 上の受信 bytes を buffering して complete HEM frame を構成してよい。

ただし、buffering は HEM frame sequence を変更してはならない。

## 9. Byte Stream Profile and Interleave

Byte Stream Profile では、HEM frame interleave は complete HEM frame 単位でのみ成立する。

Byte Stream Profile は、byte 単位、payload fragment 単位、read chunk 単位、write chunk 単位、または buffer chunk 単位の interleave を許さない。

Byte Stream Profile では、1つの HEM frame の HEM Length Header と HEM Payload の間に、別の HEM frame の bytes を挿入してはならない。

Byte Stream Profile では、1つの HEM Payload の内部に、別の HEM frame の bytes を挿入してはならない。

複数の logical send に属する HEM frame が同じ byte stream 上に現れる場合でも、それらは complete HEM frame 単位で sequence に並ばなければならない。

## 10. Message Boundary Profile

Message Boundary Profile は、transport が message boundary を提供する場合に、1つの transport message payload 全体を frame construction unit として扱う Transport Binding Profile である。

Message Boundary Profile では、ordered delivery unit は、順序保証された transport message payload sequence を提供しなければならない。

Message Boundary Profile では、1つの transport message payload は、1つの ordered delivery unit に属する HEM frame sequence を含む。

1つの transport message payload には、1つ以上の complete HEM frame を含めてよい。

Message Boundary Profile における transport message boundary は、HEM frame boundary ではない。

Message Boundary Profile における transport message boundary は、transport message payload が含む HEM frame sequence の外側の frame construction unit boundary である。

この frame construction unit は、HEMP Core の transaction mechanism ではない。

Message Boundary Profile は、HEM Length Header を削除しない。

Message Boundary Profile は、HEM Payload encoding を変更しない。

Message Boundary Profile は、HEM frame を transport message payload だけで再定義しない。

Message Boundary Profile は、transport message boundary、transport message close、または transport-level reset に `abort` semantics を持たせない。

## 11. Message Boundary Profile Frame Boundary

Message Boundary Profile では、1つの transport message payload が、次の形式の HEM frame sequence を含む。

```text
transport message payload = 1*complete HEM frame
```

各 complete HEM frame は、次の形式である。

```text
HEM frame = HEM Length Header + HEM Payload
```

1つの transport message payload に、複数の HEM frame を含めてよい。

1つの HEM frame を、複数の transport message payload に分割してはならない。

1つの transport message payload の内部に、HEMP 外の data、transport control data、log output、diagnostic output、または complete HEM frame ではない data を混ぜてはならない。

受信側は、1つの transport message payload を受信した場合、その payload 全体を HEM frame sequence として解析しなければならない。

受信側は、transport message payload の先頭から順に、次を繰り返して complete HEM frame を切り出さなければならない。

```text
1. HEM Length Header を取得する。
2. HEM Length Header が示す HEM Payload length を読む。
3. 指定された length 分の HEM Payload bytes を取得する。
4. 1つの complete HEM frame として切り出す。
5. transport message payload に残り bytes があれば、次の HEM frame として解析を続ける。
```

transport message payload の末尾ちょうどで解析が完了しなければならない。

次の場合、受信側はその transport message payload を有効な HEM frame sequence として扱ってはならない。

```text
transport message payload が空である。
transport message payload が 4 bytes 未満である。
HEM Length Header が不正である。
HEM Length Header が示す Payload bytes が transport message payload 内に足りない。
transport message payload の末尾に、complete HEM frame として解釈できない余り bytes がある。
```

transport message payload 全体が valid HEM frame sequence として正しく切り出せない場合、受信側はその transport message payload 内の HEM frame を HEMP Core validation へ一切渡してはならない。

Message Boundary Profile における HEM Length Header の形式、値、payload length、limits、および failure classification は、HEMP Core および HEMP Payload encoding specification に従う。

## 12. Message Boundary Profile and HEM Length Header

Message Boundary Profile においても、HEM Length Header は各 HEM frame の一部である。

Transport message boundary が frame construction unit boundary を提供する場合でも、各 HEM frame の HEM Length Header は維持される。

HEM Length Header は、Message Boundary Profile においても、直後に続く HEM Payload bytes の長さを表す。

Transport Binding は、transport message payload size を HEM Length Header の代替として扱ってはならない。

Transport Binding は、HEM Length Header を省略してはならない。

Transport Binding は、transport message metadata、message size、message type、message flag、または transport-specific header を、HEM Length Header の代替として扱ってはならない。

Transport Binding は、HEM Length Header と transport message payload boundary の不一致を、transport-specific な補正によって正常化してはならない。

Message Boundary Profile における transport message boundary は、HEM frame sequence の外側の frame construction unit boundary であり、HEM frame format の置換ではない。

## 13. Message Boundary Profile Processing

Message Boundary Profile では、transport message payload 全体を、先に valid HEM frame sequence として切り出せることを確認しなければならない。

transport message payload 全体が valid HEM frame sequence として正しく切り出せない場合、受信側はその transport message payload 内の HEM frame を処理してはならない。

transport message payload が valid HEM frame sequence として正しく切り出せた場合、受信側は、transport message payload 内の complete HEM frame を出現順で HEMP Core validation へ渡す。

transport message payload に複数の complete HEM frame が含まれる場合、受信側は transport message payload 内の出現順を、その transport message payload が属する ordered delivery unit 内の受信順として扱う。

受信側は、同一 transport message payload 内で切り出した HEM frame の順序を入れ替えてはならない。

1つの transport message payload に複数 HEM frame が含まれる場合でも、それらは HEMP Core semantics 上の atomic transaction ではない。

ただし、transport message payload 全体を valid HEM frame sequence として正しく切り出せない場合、受信側はその transport message payload 内の HEM を処理しない。

この切り出し処理は、HEMP Core の transaction mechanism ではない。

transport message payload 全体を valid HEM frame sequence として正しく切り出せた後、transport message payload 内の HEM frame は出現順で処理する。

いずれかの HEM frame の処理によって HEMP session が継続不能、失敗状態、または正常終了状態になった場合、受信側は同じ transport message payload 内の後続 HEM frame を処理しない。

## 14. Message Boundary Profile Payload Max

Message Boundary Profile では、transport message payload max という概念を扱う。

transport message payload max は、1つの transport message payload に載せられる最大 byte 数である。

transport message payload max は、HEMP Core では固定しない。

transport message payload max は、Transport Binding / Profile が定義する相互運用上の上限である。

transport message payload max は、concrete Transport Binding が固定値として定義しなければならない。

concrete Transport Binding が transport message payload max の可変値を許す場合、その値は HEMP session 開始前に、concrete Transport Binding が定義する方法で送受信双方に共有されていなければならない。

Agreement body は、transport message payload max の negotiation に使用しない。

実装が内部的な資源管理のために設ける制限値は、相互運用上の transport message payload max ではない。

実装が concrete Transport Binding が要求する transport message payload max を扱えない場合、その実装は当該 concrete Transport Binding の要件を満たさない。

送信側は、transport message payload max を超える transport message payload を生成してはならない。

```text
len(transport message payload) <= transport message payload max
```

受信側が `transport message payload max` を超える transport message payload を受信した場合、それは HEMP Core の failure classification ではなく、Transport Binding violation とする。

transport message payload max 超過が発生した場合、受信側はその transport message payload を処理してはならない。

transport message payload max 超過が発生した場合、受信側はその transport message payload 内の HEM frame を HEMP Core validation へ渡してはならない。

transport message payload max 超過が発生した場合、その Transport Binding instance 上の現在の HEMP session は継続不能として扱う。

1つの HEM frame 自体が `transport message payload max` に収まらない場合、その HEM frame は Message Boundary Profile では送信できない。

```text
4 + HEM Payload length > transport message payload max
```

この場合、必要であれば HEMP の logical send を複数 HEM へ chunking する。

ただし、対象 channel で `chunking banned flag = true` の場合、その logical send は 1 HEM で完結しなければならないため、transport message payload max に収まらない内容は送信できない。

application channel 上の通常 HEM communication では、transport message payload max に加えて、agreement の `body.limits.hem_payload_length` および `body.limits.role.*` も満たさなければならない。

protocol channel の HEM frame は、HEMP Core specification の protocol channel Payload length 上限を満たし、かつ concrete Transport Binding が定義する transport message payload max に収まらなければならない。

Message Boundary Profile を使用する concrete Transport Binding は、少なくとも HEMP Core specification の protocol channel Payload length 上限に等しい HEM Payload を持つ protocol channel HEM frame を送受信できる transport message payload max を定義しなければならない。

```text
transport message payload max
>=
4 + HEMP Core specification protocol channel Payload length limit
```

## 15. Message Boundary Profile and HEM Delivery Path

Message Boundary Profile を使用する各 ordered delivery unit は、Host -> Engine HEM delivery path または Engine -> Host HEM delivery path のいずれか一方に属する。

1つの transport message payload に複数 HEM frame を含める場合、その transport message payload 内のすべての HEM frame は、その transport message payload が属する ordered delivery unit の HEM delivery path で受信されたものとして扱う。

transport message payload 内の各 HEM Header の `direction` は、その transport message payload が属する ordered delivery unit の HEM delivery path と一致しなければならない。

HEM delivery path と HEM Header の `direction` field が一致しない場合、受信側は HEMP header validation failure として扱わなければならない。

## 16. Message Boundary Profile and Interleave

Message Boundary Profile では、HEM frame interleave は complete HEM frame 単位でのみ成立する。

Message Boundary Profile では、1つの transport message payload に複数の HEM frame を含めてよいため、同じ ordered delivery unit に属する異なる logical send の HEM frame を、同じ transport message payload 内で交互に配置してよい。

ただし、interleave は complete HEM frame 単位に限られる。

Message Boundary Profile は、1つの HEM frame の内部に別の HEM frame を interleave することを許さない。

Message Boundary Profile は、1つの HEM frame を複数の transport message payload に分割して interleave することを許さない。

同一 logical send に属する HEM frame は、単一の ordered delivery unit 上で送受信されなければならない。

`abort = true` の HEM を transport message payload 内に含める場合でも、その HEM は該当 logical send の終端 HEM であり、同じ logical send の継続 HEM を後続に配置してはならない。

複数の logical send に属する HEM frame が同じ ordered delivery unit 上に現れる場合でも、それらは complete HEM frame 単位で sequence に並ばなければならない。

## 17. Profile Selection

Concrete Transport Binding は、使用する Transport Binding Profile を明示しなければならない。

Concrete Transport Binding は、各 ordered delivery unit に適用される Profile を明示しなければならない。

Profile は、HEMP session 中に暗黙に切り替わってはならない。

Concrete Transport Binding は、同じ ordered delivery unit 上で、Byte Stream Profile と Message Boundary Profile を暗黙に混在させてはならない。

この文書は、Profile の交渉機構を定義しない。

Agreement body は、Transport Binding Profile negotiation に使用しない。

Transport Binding Profile を切り替える必要がある場合、その扱いは、別の concrete Transport Binding または別仕様で明示的に定義しなければならない。

Transport Binding Profile の選択または切り替えは、HEM Payload encoding を選択または変更する仕組みではない。

Transport Binding Profile は、HEM Payload bytes の解釈、Protobuf schema、body oneof branch mapping、または Payload encoding negotiation を定義しない。

Concrete Transport Binding または Transport Binding Profile は、alternate HEM Payload encoding を定義してはならない。

## 18. Relationship to Local Process IPC Binding

Local Process IPC Binding は、将来の concrete Transport Binding として定義され得る候補である。

Local Process IPC Binding を定義する場合、その Binding は Byte Stream Profile を使用してよい。

Local Process IPC Binding が Byte Stream Profile を使用する場合、Host -> Engine HEM delivery path の ordered delivery unit と Engine -> Host HEM delivery path の ordered delivery unit は、どちらも Byte Stream Profile に従う。

この文書は、Local Process IPC Binding の concrete definition を定義しない。

この文書は、process、stdio、endpoint、pipe、stream object、process launch method、endpoint creation method、endpoint naming rule、permission model、timeout value、または concrete realization rule を定義しない。

これらは、Local Process IPC Binding を正式に定義する将来文書、実現ガイド、application、または実装側が扱う。

## 19. Non-Goals

この文書は、次を定義しない。

```text
HEMP Core semantics の変更
HEM Payload encoding の変更
HEM Length Header の別形式
HEM Length Header の省略
Payload encoding negotiation
Profile negotiation mechanism
transport failure の詳細
retry policy
reconnect policy
specific OS API
specific runtime
specific async framework
specific transport library
Local Process IPC Binding の concrete rules
network transport binding
```

## 20. Relationship to Other Documents

この文書は、Transport Binding Profile、HEM frame interleave、Byte Stream Profile、および Message Boundary Profile を定義する。

`02-transport-model.md` は、HEM delivery path、ordered delivery unit、HEM frame mapping、Transport Binding instance、および HEMP session との関係を定義する。

`03-transport-requirements-and-frame-handling.md` は、transport requirements、complete HEM frame delivery、transport failure、および `abort` と transport failure の境界を定義する。

Local Process IPC Binding を扱う場合は、将来文書、draft 文書、または実現ガイドとして、v1.0.0 確定本文とは分離して扱う。
