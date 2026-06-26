# HEMP Transport Model

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Transport Binding specification

## 1. Purpose

この文書は、HEMP Transport Binding における transport model を定義する。

transport model は、HEMP Core が定義する HEM frame を、具体的な transport 上で送受信するための共通モデルである。

この文書では、次を定義する。

```text
real communication direction
relationship between direction field and real communication direction
ordering unit
HEM frame mapping
HEM frame sequence
Transport Binding instance
relationship between Transport Binding instance and HEMP session
```

この文書は、特定の OS API、runtime、async framework、transport library、または process model を要求しない。

## 2. Transport Model Overview

Transport Binding は、HEM frame を transport 上で送受信するために、少なくとも次を定義しなければならない。

```text
Host -> Engine 方向の送信経路
Engine -> Host 方向の送信経路
各方向における順序保証単位
HEM frame をどの順序保証単位に載せるか
Transport Binding instance の範囲
HEMP session との対応関係
```

Transport Binding は、HEMP Core semantics を変更しない。

Transport Binding は、HEMP Core が定義する `direction`、`channel`、`thread`、`seq`、`role`、`end`、`close`、`abort` の意味を再定義しない。

Transport Binding は、これらの Core semantics を、実際の transport 上の送受信モデルへ写像する。

## 3. Real Communication Direction

Transport Binding は、次の2つの実通信方向を区別しなければならない。

```text
Host -> Engine
Engine -> Host
```

`Host -> Engine` は、Host から Engine へ HEM frame を送る実通信方向である。

`Engine -> Host` は、Engine から Host へ HEM frame を送る実通信方向である。

実通信方向は、transport 上で実際に HEM frame が流れる方向である。

実通信方向は、具体的な transport の形に依存しない。

たとえば、concrete Transport Binding は、2つの単方向 transport、1つの双方向 transport、または transport-specific な channel / stream / message abstraction によって、これら2方向を提供してよい。

ただし、どの実現方式であっても、Transport Binding は HEM frame が Host -> Engine と Engine -> Host のどちらの実通信方向で受信されたかを判定できなければならない。

## 4. Direction Field and Real Communication Direction

HEM Header の `direction` field は、HEMP Core 上の論理送信方向を表す。

Transport Binding は、`direction` field と実通信方向を一致させなければならない。

対応関係は次の通りである。

```text
direction = to_engine:
  HEM frame は Host -> Engine 方向で送信されなければならない。

direction = to_host:
  HEM frame は Engine -> Host 方向で送信されなければならない。
```

受信側は、HEM frame がどの実通信方向で受信されたかを、Transport Binding から判定できなければならない。

受信側は、実通信方向と HEM Header の `direction` field が一致しない HEM を、有効な HEM として扱ってはならない。

実通信方向と HEM Header の `direction` field が一致しない場合、receiver は HEMP header validation failure として扱わなければならない。

Transport Binding は、実通信方向と `direction` field の不一致を、transport-specific な別意味として解釈してはならない。

Transport Binding は、不一致を自動補正してはならない。

Transport Binding は、不一致を反対方向への routing instruction として扱ってはならない。

## 5. Ordering Unit

ordering unit は、HEM frame sequence の順序が保持される transport 上の単位である。

Transport Binding は、各実通信方向について、HEM frame sequence を運ぶ ordering unit を定義しなければならない。

ordering unit 内では、送信された HEM frame sequence が送信順と同じ順序で受信されなければならない。

Transport Binding は、少なくとも次を満たす ordering unit を提供しなければならない。

```text
Host -> Engine 方向の HEM frame sequence を運ぶ ordering unit
Engine -> Host 方向の HEM frame sequence を運ぶ ordering unit
```

1つの双方向 transport を使用する場合でも、Transport Binding は Host -> Engine と Engine -> Host の各方向を、それぞれ順序保証可能な送受信方向として扱わなければならない。

2つの単方向 transport を使用する場合、それぞれの単方向 transport が対応する実通信方向の ordering unit になってよい。

transport-specific な channel、stream、queue、message sequence などを使用する場合、それが ordering unit としてどの範囲の順序を保証するかを concrete Transport Binding が定義しなければならない。

## 6. Ordering Unit and `seq`

ordering unit による順序保証と、HEMP Core の `seq` は同じものではない。

ordering unit は、transport 上で HEM frame sequence の到達順序を保持するための単位である。

`seq` は、HEMP Core 上で、同一 logical send 内の HEM の順序を検証するための Header field である。

`seq` は、下位 transport の順序崩れを修復するための再送制御、並べ替え機構、または transport sequence number ではない。

Transport Binding は、下位 transport によって順序が崩れた HEM frame sequence を、`seq` によって修復するものとして定義してはならない。

Transport Binding は、ordering unit が必要な順序保証を提供できない場合、その transport を有効な Transport Binding として扱ってはならない。

Transport Binding は、HEMP Core の `seq` を、異なる ordering unit 間の並べ替え、順序修復、または missing HEM の補完に使用してはならない。

## 7. HEM Frame Mapping

Transport Binding は、HEM frame を transport 上の ordering unit に mapping しなければならない。

HEM frame mapping は、少なくとも次を定義する。

```text
direction = to_engine の HEM frame を載せる ordering unit
direction = to_host の HEM frame を載せる ordering unit
ordering unit 上で HEM frame sequence をどう扱うか
```

`direction = to_engine` の HEM frame は、Host -> Engine 方向の ordering unit 上で送信されなければならない。

`direction = to_host` の HEM frame は、Engine -> Host 方向の ordering unit 上で送信されなければならない。

Transport Binding は、1つの HEM frame を、複数の実通信方向に同時に属するものとして扱ってはならない。

Transport Binding は、1つの HEM frame を、Core semantics 上の別の HEM frame に分割してはならない。

Transport Binding は、複数の HEM frame を、Core semantics 上の1つの HEM frame に結合してはならない。

同一の論理 post 送信、論理 reply 送信、または論理 notice 送信に属する HEM frame は、単一の ordering unit に mapping しなければならない。

Transport Binding は、同一 logical send に属する HEM frame を複数の ordering unit へ分散してはならない。

1つの ordering unit は、複数の channel、複数の thread、複数の role、または複数の logical send に属する HEM frame を運んでよい。

ただし、その interleave は、適用される Transport Binding Profile が許す complete HEM frame 単位に限る。

Transport Binding は、HEMP Core の `seq` を、異なる ordering unit 間の並べ替え、順序修復、または missing HEM の補完に使用してはならない。

transport profile によっては、1つの HEM frame が transport 上の複数 read / write / buffer operation にまたがることがある。

その場合でも、Transport Binding は、HEMP Core の処理へ渡す前に complete HEM frame として扱えるようにしなければならない。

byte-level framing、frame boundary handling、および profile-specific な HEM frame 構成規則は、`03-transport-requirements-and-frame-handling.md` および `04-transport-profiles.md` で定義する。

## 8. HEM Frame Sequence

HEM frame sequence は、ordering unit 上で送受信される HEM frame の列である。

Transport Binding は、各 ordering unit 上の HEM frame sequence を、その ordering unit の順序保証に従って扱わなければならない。

HEM frame sequence は、HEMP Core の logical send sequence と同一ではない。

1つの HEM frame sequence には、複数の channel、複数の thread、複数の role、複数の logical send に属する HEM frame が含まれ得る。

どの HEM frame がどの logical send に属するかは、HEMP Core が定義する `channel`、`thread`、`role`、`seq`、`end`、`close`、`abort` によって判断する。

Transport Binding は、HEM frame sequence 内の HEM frame を、Transport Binding 独自の意味によって logical send に分類してはならない。

Transport Binding は、HEM frame sequence に HEM frame 以外の data を混ぜてはならない。

ログ、診断出力、デバッグ文字列、人間向けメッセージ、または HEMP 外の制御 data は、HEM frame sequence とは別の経路で扱わなければならない。

## 9. Transport Binding Instance

Transport Binding instance は、HEMP session に対して、HEM frame を送受信するための transport context である。

Transport Binding instance は、少なくとも次を含む。

```text
Host -> Engine 方向の送信能力
Engine -> Host 方向の送信能力
各方向の ordering unit
HEM frame mapping rules
適用される Transport Binding Profile
concrete Transport Binding が定義する transport-specific context
```

Transport Binding instance は、特定の process、pipe、socket、endpoint、file descriptor、message queue、transport library object、または runtime object と同義ではない。

それらは concrete Transport Binding または実装が Transport Binding instance を実現するために使用してよい transport-specific resource である。

Transport Binding instance の意味上の単位は、HEMP session を成立させるために必要な HEM frame delivery context である。

## 10. Relationship Between Transport Binding Instance and HEMP Session

HEMP session は、Transport Binding instance 上で成立する。

1つの Transport Binding instance は、同時に1つの HEMP session だけを運ぶ。

同じ Transport Binding instance 上で、複数の独立した HEMP session を同時に多重化してはならない。

複数の HEMP session を同時に扱う場合、それぞれの HEMP session に対して別の Transport Binding instance を使用しなければならない。

Transport Binding instance を、終了済み HEMP session の後に再利用できるかどうかは、concrete Transport Binding または実装が定義してよい。

ただし、再利用後に HEMP communication を行う場合は、新しい HEMP session として agreement から開始しなければならない。

Transport Binding instance は、HEMP session の agreement、limits、flow state、channel state、thread state を再定義しない。

これらは HEMP Core semantics に従う。

Transport Binding instance は、HEMP session が使用する HEM frame delivery context を提供する。

Transport Binding instance が継続不能になった場合、その Transport Binding instance 上の HEMP session も継続不能として扱う。

transport failure 後の HEMP session の扱い、同じ session を再開しないこと、および再通信時に新しい session として agreement から開始することは、`03-transport-requirements-and-frame-handling.md` で定義する。

## 11. Transport-Agnostic Model

この transport model は、特定の transport 実装に依存しない。

この文書は、次を要求しない。

```text
specific OS API
specific process model
specific pipe implementation
specific socket API
specific endpoint mechanism
specific transport library
specific async framework
specific runtime
specific thread model
```

Concrete Transport Binding は、この文書が定義する transport model を、具体的な transport へ写像する。

Concrete Transport Binding は、使用する transport-specific resource が、必要な実通信方向、ordering unit、HEM frame mapping、および HEM frame sequence delivery をどのように満たすかを定義しなければならない。

## 12. Relationship to Later Documents

この文書は、Transport Binding の共通 transport model を定義する。

詳細な transport requirements、frame boundary handling、frame handling、transport failure、および `abort` と transport failure の境界は、`03-transport-requirements-and-frame-handling.md` で定義する。

HEM frame interleave、Byte Stream Profile、および Message Boundary Profile は、`04-transport-profiles.md` で定義する。

Local Process IPC Binding の concrete definition は、`local-ipc/specification.md` で定義する。
