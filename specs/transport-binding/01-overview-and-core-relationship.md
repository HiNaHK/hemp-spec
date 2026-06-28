# HEMP Transport Binding Overview and Core Relationship

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Transport Binding specification

## 1. Purpose

この文書は、HEMP Transport Binding の目的と、HEMP Core との関係を定義する。

Transport Binding は、Host-Engine Message Protocol / HEMP を具体的な transport 上で送受信するための仕様領域である。

Transport Binding は、HEMP Core が定義する HEM frame を transport 上でどのように運ぶかを定義する。

Transport Binding は、HEMP Core semantics を再定義しない。

## 2. Position of Transport Binding

Transport Binding は、HEMP protocol specification の一部である。

Transport Binding は、単なる実装都合、補助文書、または任意追加の別要素ではない。

HEMP を実際に送受信するには、HEM frame を具体的な transport 上で運ぶ必要がある。Transport Binding は、そのために必要な HEM delivery path、ordered delivery unit、HEM frame delivery、Transport Binding Profile、および concrete Transport Binding definitions を定義する。

ただし、Transport Binding は HEMP Core が定義する protocol semantics を変更しない。

## 3. Relationship to HEMP Core

HEMP Core は、HEMP protocol の構造、意味、検証、状態遷移、limits、および failure classification を定義する。

HEMP Core が定義するものには、少なくとも次を含む。

```text
HEM frame
HEM Payload
HEM Header
HEM Body
channel
thread
seq
role
direction
end
close
abort
post / reply / notice
agreement
limits
failure classification
HEM Body Contract
```

Transport Binding は、これらの Core semantics を transport 上の送受信へ写像する。

Transport Binding は、下位transportから complete HEM frame を構成し、HEMP Core が定義する順序制約と検証規則に従って HEM frame を HEMP Core の処理へ渡す責任を持つ。

Transport Binding は、HEMP Core semantics を緩和してはならない。

Transport Binding は、HEMP Core semantics を拡張して別の意味にしてはならない。

Transport Binding は、HEMP Core semantics と矛盾する transport 固有の規則を定義してはならない。

## 4. What Transport Binding Defines

Transport Binding は、HEMP Core が定義する HEM frame を transport 上で送受信するために、次を定義する。

```text
HEM delivery path
ordered delivery unit
HEM frame mapping
HEM frame boundary handling
HEM frame sequence delivery
relationship between direction field and HEM delivery path
Transport Binding instance
relationship between Transport Binding instance and HEMP session
transport failure boundary
relationship between abort and transport failure
Transport Binding Profile
concrete Transport Binding definitions
```

Transport Binding は、具体的な transport の形に応じて、HEM frame sequence をどの ordered delivery unit で運ぶかを定義する。

Transport Binding は、各 ordered delivery unit が Host -> Engine HEM delivery path または Engine -> Host HEM delivery path のどちらに属するかを定義する。

Transport Binding は、受信した HEM frame の HEM delivery path が HEM Header の `direction` と一致することを検証できる形で、HEM frame を HEMP Core へ渡さなければならない。

## 5. What Transport Binding Does Not Redefine

Transport Binding は、次を再定義しない。

```text
HEMP Core semantics
HEM Payload encoding
HEM Length Header meaning
HEM Header field semantics
channel / thread / seq semantics
role / direction semantics
post / reply / notice semantics
end / close / abort semantics
agreement semantics
limits semantics
HEMP failure classification
HEM Body semantics
HEM Body Contract semantics
```

Transport Binding は、HEM Payload encoding negotiation を定義しない。

Transport Binding は、alternate HEM Payload encoding を定義しない。

Transport Binding は、HEMP Core の failure classification に transport failure を追加しない。

Transport Binding は、下位transportの再送、重複排除、順序修復、認証、暗号化、timeout policy、retry policy、または reconnect policy を HEMP Core semantics として定義しない。

これらは、下位transport、concrete Transport Binding、または実装側の責任である。

## 6. Transport Binding and HEM Frame Delivery

Transport Binding は、HEM frame を transport 上で届ける仕様領域である。

Transport Binding は、HEM frame を complete HEM frame として HEMP Core の処理へ渡せるように扱わなければならない。

HEM frame boundary は、適用される Transport Binding Profile に従って扱わなければならない。

Transport Binding は、transport の read / write / buffer operation を、そのまま HEM frame boundary とみなしてはならない。

次を、それ自体として HEM frame boundary とみなしてはならない。

```text
read boundary
write boundary
flush boundary
buffer boundary
line break
```

Message Boundary Profile における transport message boundary の扱いは、`04-transport-profiles.md` で定義する。

Transport Binding は、HEM frame sequence を壊してはならない。

Transport Binding は、1つの HEM frame の内部に、別の HEM frame または HEMP 外の byte sequence を挿入してはならない。

`abort = true` の HEM も、通常の complete HEM frame として transport 上で運ばれなければならない。

`abort = true` は、transport close、stream close、pipe close、endpoint close、process termination、transport reset、または transport failure を意味しない。

transport failure、stream close、pipe close、EOF、process termination、I/O error、timeout、または transport reset を、`abort = true` の HEM の代替として扱ってはならない。

Transport Binding は、transport failure を理由に、未完了の logical send を HEMP Core 上の aborted completion として扱ってはならない。

transport failure によって現在の HEMP session を継続できない場合、その HEMP session は継続不能として扱う。

transport failure 後に同じ HEMP session を再開してはならない。

再度 HEMP 通信を行う場合は、新しい HEMP session として agreement から開始する。

詳細な frame handling、transport requirements、transport failure、および `abort` との境界は、`03-transport-requirements-and-frame-handling.md` で定義する。

## 7. Transport Binding Documents

v1.0.0 の Transport Binding specification は、次の文書で構成する。

```text
01-overview-and-core-relationship.md
02-transport-model.md
03-transport-requirements-and-frame-handling.md
04-transport-profiles.md
```

`01-overview-and-core-relationship.md` は、Transport Binding の目的と HEMP Core との関係を定義する。

`02-transport-model.md` は、HEM delivery path、ordered delivery unit、HEM frame mapping、Transport Binding instance、および HEMP session との関係を定義する。

`03-transport-requirements-and-frame-handling.md` は、transport に要求する性質、HEM frame boundary、HEM frame sequence、transport failure、および `abort` と transport failure の境界を定義する。

`04-transport-profiles.md` は、HEM frame interleave、Byte Stream Profile、および Message Boundary Profile を定義する。

Local Process IPC Binding は、現時点では v1.0.0 の確定済み Transport Binding specification 本文としては定義しない。

Local Process IPC Binding を扱う場合は、別の future binding 文書、draft 文書、または実現ガイドとして、v1.0.0 確定本文とは分離して扱う。
