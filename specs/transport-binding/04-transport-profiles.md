# HEMP Transport Profiles

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Transport Binding specification

## 1. Purpose

この文書は、HEMP Transport Binding Profile を定義する。

Transport Binding Profile は、HEM frame を transport 上でどの boundary rule と delivery model に従って扱うかを定義する。

この文書では、次を定義する。

```text
Transport Binding Profile
Profile と concrete Transport Binding の関係
HEM frame boundary by Profile
HEM frame interleave
Byte Stream Profile
Message Boundary Profile
Profile selection
Local Process IPC Binding との関係
```

この文書は、HEMP Core semantics を再定義しない。

この文書は、HEM Payload encoding を再定義しない。

この文書は、Transport Binding instance、ordering unit、transport failure、retry policy、または reconnect policy の詳細を再定義しない。

## 2. Transport Binding Profile

Transport Binding Profile は、HEM frame sequence を transport 上でどのように区切り、どのように complete HEM frame として届けるかを定義する model である。

Transport Binding Profile は、Transport Binding が使用する frame boundary rule と delivery model を定義する。

Transport Binding Profile は、HEMP Core semantics を変更しない。

Transport Binding Profile は、HEM Header field semantics を変更しない。

Transport Binding Profile は、HEM Length Header の意味を変更しない。

Transport Binding Profile は、HEM Payload encoding を変更しない。

Transport Binding Profile は、HEM Body semantics を変更しない。

Transport Binding Profile は、transport 上の boundary rule を定義するものであり、HEMP Core の logical send、flow semantics、agreement、limits、failure classification を再定義するものではない。

## 3. Profile and Concrete Transport Binding

Concrete Transport Binding は、使用する Transport Binding Profile を定義しなければならない。

Concrete Transport Binding は、各 ordering unit がどの Profile に従うかを定義しなければならない。

Concrete Transport Binding は、すべての ordering unit に同じ Profile を使用してよい。

Concrete Transport Binding は、必要であれば、方向または ordering unit ごとに異なる Profile を使用してよい。

ただし、その場合でも、どの ordering unit がどの Profile に従うかを明示しなければならない。

Profile の選択は、receiver が HEM frame boundary を判断できる形で定義されなければならない。

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
```

Transport message boundary は、適用される Transport Binding Profile がそれを HEM frame boundary として定義する場合に限り、HEM frame boundary として扱ってよい。

Transport Binding は、Profile が定義しない boundary を、transport-specific な都合によって HEM frame boundary として追加してはならない。

Transport Binding は、Profile が定義する boundary rule に反して、HEM frame を分割、結合、補正、または再解釈してはならない。

## 5. HEM Frame Interleave

HEM frame interleave は、1つの ordering unit 上の HEM frame sequence に、複数の logical send、channel、thread、または role に属する HEM frame が混在することである。

Transport Binding Profile は、HEM frame interleave を許すかどうかを定義してよい。

HEM frame interleave を許す場合でも、interleave は complete HEM frame 単位に限られる。

Transport Binding は、1つの HEM frame の内部に、別の HEM frame を interleave してはならない。

Transport Binding は、1つの HEM frame の内部に、HEMP 外の byte sequence、transport control data、log output、diagnostic output、または別 logical send の data を挿入してはならない。

Transport Binding Profile が HEM frame interleave を許す場合でも、HEMP Core の flow semantics、channel rules、thread rules、seq rules、end / close / abort rules、および limits は変更されない。

どの HEM frame がどの logical send に属するかは、Transport Binding ではなく、HEMP Core が定義する Header fields と flow semantics によって判断する。

## 6. Byte Stream Profile

Byte Stream Profile は、HEM frame sequence を連続した byte stream として扱う Transport Binding Profile である。

Byte Stream Profile では、ordering unit は、順序保証された byte stream を提供しなければならない。

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

Message Boundary Profile は、transport が message boundary を提供する場合に、その message boundary を HEM frame delivery boundary として使用する Transport Binding Profile である。

Message Boundary Profile では、ordering unit は、順序保証された transport message sequence を提供しなければならない。

Message Boundary Profile では、1つの transport message は、1つの complete HEM frame に対応する。

Message Boundary Profile では、transport message boundary は HEM frame boundary として扱われる。

ただし、transport message boundary は、HEM frame format を置き換えない。

Message Boundary Profile は、HEM Length Header を削除しない。

Message Boundary Profile は、HEM Payload encoding を変更しない。

Message Boundary Profile は、HEM frame を transport message payload だけで再定義しない。

## 11. Message Boundary Profile Frame Boundary

Message Boundary Profile では、1つの transport message が、次の形式の1つの complete HEM frame を含まなければならない。

```text
transport message payload = HEM Length Header + HEM Payload
```

1つの transport message に、複数の HEM frame を含めてはならない。

1つの HEM frame を、複数の transport message に分割してはならない。

1つの transport message の内部に、HEMP 外の data、transport control data、log output、diagnostic output、または複数 logical send の non-frame data を混ぜてはならない。

受信側は、1つの transport message を受信した場合、その transport message 全体を1つの candidate HEM frame として扱う。

受信側は、その candidate HEM frame について、HEM Length Header と HEM Payload bytes の整合性を検証しなければならない。

transport message の byte length は、HEM Length Header の size と、HEM Length Header が示す HEM Payload length の合計と一致しなければならない。

一致しない場合、receiver はその transport message を有効な complete HEM frame として扱ってはならない。

Message Boundary Profile における HEM Length Header の形式、値、payload length、limits、および failure classification は、HEMP Core および HEMP Payload encoding specification に従う。

## 12. Message Boundary Profile and HEM Length Header

Message Boundary Profile においても、HEM Length Header は HEM frame の一部である。

Transport message boundary が HEM frame boundary を提供する場合でも、HEM Length Header は維持される。

HEM Length Header は、Message Boundary Profile においても、直後に続く HEM Payload bytes の長さを表す。

Transport Binding は、transport message size を HEM Length Header の代替として扱ってはならない。

Transport Binding は、HEM Length Header を省略してはならない。

Transport Binding は、transport message metadata、message size、message type、message flag、または transport-specific header を、HEM Length Header の代替として扱ってはならない。

Transport Binding は、HEM Length Header と transport message boundary の不一致を、transport-specific な補正によって正常化してはならない。

Message Boundary Profile における transport message boundary は、complete HEM frame の外側の delivery boundary であり、HEM frame format の置換ではない。

## 13. Message Boundary Profile and Interleave

Message Boundary Profile では、HEM frame interleave は transport message 単位でのみ成立する。

Message Boundary Profile では、1つの transport message が1つの complete HEM frame に対応するため、interleave の最小単位は1 transport messageである。

Message Boundary Profile は、1つの transport message 内で複数の HEM frame を interleave することを許さない。

Message Boundary Profile は、1つの HEM frame を複数の transport message に分割して interleave することを許さない。

複数の logical send に属する HEM frame が同じ ordering unit 上に現れる場合でも、それらは complete transport message、かつ complete HEM frame 単位で sequence に並ばなければならない。

## 14. Profile Selection

Concrete Transport Binding は、使用する Transport Binding Profile を明示しなければならない。

Concrete Transport Binding は、各 ordering unit に適用される Profile を明示しなければならない。

Profile は、HEMP session 中に暗黙に切り替わってはならない。

Concrete Transport Binding は、同じ ordering unit 上で、Byte Stream Profile と Message Boundary Profile を暗黙に混在させてはならない。

この文書は、Profile negotiation mechanism を定義しない。

Agreement body は、Transport Binding Profile negotiation に使用しない。

Transport Binding Profile を切り替える必要がある場合、その扱いは、別の concrete Transport Binding、別の session profile、または別仕様で明示的に定義しなければならない。

## 15. Relationship to Local Process IPC Binding

Local Process IPC Binding は、Byte Stream Profile を使用する concrete Transport Binding である。

Local Process IPC Binding では、Host -> Engine 方向の ordering unit と、Engine -> Host 方向の ordering unit が、どちらも Byte Stream Profile に従う。

Local Process IPC Binding 固有の `host_to_engine stream`、`engine_to_host stream`、process、stdio、endpoint、pipe、stream、transport failure、および concrete binding rules は、`local-ipc/specification.md` で定義する。

この文書は、Local Process IPC Binding の concrete definition を再定義しない。

## 16. Non-Goals

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

## 17. Relationship to Other Documents

この文書は、Transport Binding Profile、HEM frame interleave、Byte Stream Profile、および Message Boundary Profile を定義する。

`02-transport-model.md` は、実通信方向、ordering unit、HEM frame mapping、Transport Binding instance、および HEMP session との関係を定義する。

`03-transport-requirements-and-frame-handling.md` は、transport requirements、complete HEM frame delivery、transport failure、および `abort` と transport failure の境界を定義する。

`local-ipc/specification.md` は、Local Process IPC Binding の concrete definition を定義する。
