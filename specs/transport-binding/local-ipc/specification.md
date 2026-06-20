# HEMP Local Process IPC Binding

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Transport Binding specification

## 1. Purpose

この文書は、HEMP Local Process IPC Binding を定義する。

Local Process IPC Binding は、同一コンピュータ上で動作する Host process と Engine process の間で HEMP を送受信するための concrete Transport Binding である。

Local Process IPC Binding は、HEMP Core semantics を再定義しない。

Local Process IPC Binding は、HEMP Transport Binding の共通 model、requirements、および Profile rules に従う。

この文書は、Local Process IPC Binding として満たすべき規範要件を定義する。

この文書は、特定の OS API、runtime、async framework、transport library、process launch method、または endpoint mechanism を要求しない。

## 2. Relationship to HEMP Core and Transport Binding

Local Process IPC Binding は、HEMP Transport Binding specification の一部である。

Local Process IPC Binding は、HEMP Core が定義する HEM frame を、同一コンピュータ上の Host process と Engine process の間で運ぶ。

Local Process IPC Binding は、HEMP Core が定義する次の semantics を変更してはならない。

```text
HEM frame
HEM Payload
HEM Header
HEM Body
direction
role
channel
thread
seq
end
close
abort
post / reply / notice
agreement
limits
failure classification
HEM Body Contract
```

Local Process IPC Binding は、HEMP Payload encoding を変更しない。

Local Process IPC Binding は、HEM Length Header の意味を変更しない。

Local Process IPC Binding は、HEMP failure classification に新しい分類を追加しない。

Local Process IPC Binding は、local transport の確立、stream、endpoint、process、pipe、または I/O abstraction を HEMP Core semantics として扱ってはならない。

## 3. Binding Overview

Local Process IPC Binding は、Host process と Engine process の間に、次の2つの論理単方向 byte stream を定義する。

```text
host_to_engine stream:
  Host から Engine へ HEM frame sequence を送る byte stream。

engine_to_host stream:
  Engine から Host へ HEM frame sequence を送る byte stream。
```

`host_to_engine stream` は、Host -> Engine 方向の ordering unit である。

`engine_to_host stream` は、Engine -> Host 方向の ordering unit である。

Local Process IPC Binding は、Host -> Engine と Engine -> Host の両方向に HEM frame sequence を送受信できなければならない。

Local Process IPC Binding は、2つの論理単方向 byte stream を提供できればよい。

これらの stream は、1つの双方向 local transport によって実現してもよい。

これらの stream は、2つの単方向 local transport の組によって実現してもよい。

どの実現方式であっても、Local Process IPC Binding は、各 HEM frame がどちらの実通信方向で受信されたかを判定できなければならない。

## 4. Byte Stream Profile Usage

Local Process IPC Binding は、Byte Stream Profile を使用する。

`host_to_engine stream` は、Byte Stream Profile に従わなければならない。

`engine_to_host stream` は、Byte Stream Profile に従わなければならない。

各 stream 上では、HEM frame sequence を連続した byte stream として扱う。

各 stream 上の HEM frame boundary は、HEM Length Header によって判断する。

Local Process IPC Binding は、read boundary、write boundary、flush boundary、buffer boundary、line break、text line、または implementation chunk boundary を HEM frame boundary として扱ってはならない。

Local Process IPC Binding は、HEM Length Header に field、flag、encoding selector、checksum、compression marker、profile marker、または local IPC marker を追加してはならない。

## 5. Local Process IPC Binding Instance

Local Process IPC Binding instance は、同一コンピュータ上の Host process と Engine process の間で HEM frame を送受信するための concrete Transport Binding instance である。

Local Process IPC Binding instance は、少なくとも次を提供しなければならない。

```text
host_to_engine stream
engine_to_host stream
Host -> Engine 方向の ordering unit
Engine -> Host 方向の ordering unit
Byte Stream Profile に従う HEM frame handling
local transport-specific context
```

Local Process IPC Binding instance は、特定の OS process、pipe、socket、endpoint、file descriptor、stream object、reader、writer、runtime object、または library object と同義ではない。

それらは、Local Process IPC Binding instance を実現するための local transport-specific resource として使用してよい。

Local Process IPC Binding instance は、同時に1つの HEMP session だけを運ぶ。

同じ Local Process IPC Binding instance 上で、複数の独立した HEMP session を同時に多重化してはならない。

複数の HEMP session を同時に扱う場合、それぞれの HEMP session に対して別の Local Process IPC Binding instance を使用しなければならない。

Local Process IPC Binding instance を、終了済み HEMP session の後に再利用できるかどうかは、concrete realization または実装が定義してよい。

ただし、再利用後に HEMP communication を行う場合は、新しい HEMP session として agreement から開始しなければならない。

## 6. `host_to_engine stream`

`host_to_engine stream` は、Host から Engine へ HEM frame sequence を送る論理単方向 byte stream である。

`host_to_engine stream` は、Host -> Engine 方向の ordering unit である。

`host_to_engine stream` は、Byte Stream Profile に従わなければならない。

Host は、`direction = to_engine` の HEM frame を `host_to_engine stream` 上で送信しなければならない。

Engine は、`host_to_engine stream` から受信した HEM frame を Host -> Engine 方向で受信された HEM として扱わなければならない。

`host_to_engine stream` 上では、送信された HEM frame sequence が送信順と同じ順序で受信されなければならない。

`host_to_engine stream` に、HEM frame sequence 以外の data を流してはならない。

## 7. `engine_to_host stream`

`engine_to_host stream` は、Engine から Host へ HEM frame sequence を送る論理単方向 byte stream である。

`engine_to_host stream` は、Engine -> Host 方向の ordering unit である。

`engine_to_host stream` は、Byte Stream Profile に従わなければならない。

Engine は、`direction = to_host` の HEM frame を `engine_to_host stream` 上で送信しなければならない。

Host は、`engine_to_host stream` から受信した HEM frame を Engine -> Host 方向で受信された HEM として扱わなければならない。

`engine_to_host stream` 上では、送信された HEM frame sequence が送信順と同じ順序で受信されなければならない。

`engine_to_host stream` に、HEM frame sequence 以外の data を流してはならない。

## 8. Direction Mapping

Local Process IPC Binding における `direction` field と stream の対応は次の通りである。

```text
direction = to_engine:
  HEM frame は host_to_engine stream 上で送信されなければならない。

direction = to_host:
  HEM frame は engine_to_host stream 上で送信されなければならない。
```

Receiver は、HEM frame がどの stream から受信されたかを判定できなければならない。

Receiver は、受信 stream と HEM Header の `direction` field が一致することを検証しなければならない。

`host_to_engine stream` から受信された HEM frame の `direction` は `to_engine` でなければならない。

`engine_to_host stream` から受信された HEM frame の `direction` は `to_host` でなければならない。

受信 stream と HEM Header の `direction` field が一致しない場合、receiver は HEMP header validation failure として扱わなければならない。

Local Process IPC Binding は、受信 stream と `direction` field の不一致を自動補正してはならない。

Local Process IPC Binding は、不一致を反対方向への routing instruction として扱ってはならない。

Local Process IPC Binding は、不一致を local transport-specific な別意味として解釈してはならない。

## 9. Ordering and `seq`

Local Process IPC Binding では、`host_to_engine stream` と `engine_to_host stream` のそれぞれが ordering unit である。

各 stream 内では、送信された HEM frame sequence が送信順と同じ順序で受信されなければならない。

HEMP Core の `seq` は、下位 local transport の順序崩れを修復するための値ではない。

`seq` は、HEMP Core 上で、同一 logical send 内の HEM の順序を検証するための Header field である。

Local Process IPC Binding は、local transport の順序崩れを `seq` によって修復するものとして定義してはならない。

Local Process IPC Binding は、各 stream が必要な順序保証を提供できない local transport を、有効な realization として扱ってはならない。

## 10. HEM Frame Sequence Handling

Local Process IPC Binding は、各 stream 上の HEM frame sequence を Byte Stream Profile に従って扱わなければならない。

各 stream 上では、複数の HEM frame が byte stream 上に連続して並ぶ。

```text
HEM frame 1
HEM frame 2
HEM frame 3
...
```

各 HEM frame は、次の形式で扱われる。

```text
HEM frame = HEM Length Header + HEM Payload
```

Receiver は、stream から HEM Length Header を取得し、続いて HEM Length Header が示す length 分の HEM Payload bytes を取得しなければならない。

Receiver は、HEM Length Header と、その HEM Length Header が示す HEM Payload bytes を取得できた場合に限り、complete HEM frame を構成できる。

Local Process IPC Binding は、不完全な HEM frame を HEMP Core validation に渡してはならない。

Local Process IPC Binding は、HEM frame sequence を壊してはならない。

Local Process IPC Binding は、HEM Length Header と HEM Payload の順序および連続性を壊してはならない。

Local Process IPC Binding は、1つの HEM frame の内部に、別の HEM frame または HEMP 外の byte sequence を挿入してはならない。

Interleave を扱う場合でも、interleave は complete HEM frame 単位に限られる。

Local Process IPC Binding は、1つの HEM frame の内部に別の HEM frame を interleave してはならない。

## 11. Non-HEM Data Separation

`host_to_engine stream` と `engine_to_host stream` には、HEM frame sequence 以外を流してはならない。

次を HEM stream に混ぜてはならない。

```text
log output
diagnostic output
debug text
human-readable message
transport-specific control text
non-HEMP control data
local IPC control data
```

ログ、診断出力、デバッグ文字列、人間向けメッセージ、または HEMP 外の制御 data が必要な場合、それらは HEM stream とは別の経路で扱わなければならない。

HEM stream に混入した HEM 以外の data を、Local Process IPC Binding 独自の補助情報として解釈してはならない。

HEM stream に混入した HEM 以外の data を、HEM Length Header、HEM Payload、HEM Body、Protocol channel message、または Application channel message として補正して扱ってはならない。

## 12. Transport Establishment Responsibility

Local Process IPC Binding は、Host process と Engine process の間に、`host_to_engine stream` と `engine_to_host stream` を提供する local transport context を必要とする。

Local Process IPC Binding は、local transport の確立方法を固定しない。

Local Process IPC Binding は、次を HEMP として定義しない。

```text
process launch method
endpoint creation method
endpoint discovery method
connection establishment method
pipe creation method
stream object model
permission setup
timeout value
peer identity mechanism
```

これらは、concrete realization、application、または実装側が定義してよい。

ただし、確立された local transport は、Local Process IPC Binding が要求する `host_to_engine stream` と `engine_to_host stream` を提供しなければならない。

Local transport の確立に失敗した場合、その local transport context 上で HEMP session を開始してはならない。

## 13. Realization Independence

Local Process IPC Binding は、特定の realization に依存しない。

Local Process IPC Binding は、次を必須 realization として要求しない。

```text
specific process model
specific process launch method
specific local transport resource
specific local endpoint mechanism
specific named pipe mechanism
specific socket API
specific stream API
specific reader / writer abstraction
specific file descriptor model
specific OS API
specific runtime
specific async framework
specific transport library
specific thread model
```

各実装は、Local Process IPC Binding の要件を満たす local transport-specific resource を使用してよい。

HEMP として必要なのは、最終的に次を満たすことである。

```text
host_to_engine stream を Host -> Engine 方向の ordering unit として提供すること。
engine_to_host stream を Engine -> Host 方向の ordering unit として提供すること。
両 stream が Byte Stream Profile に従うこと。
各 stream が HEM frame sequence を送信順に届けること。
HEM stream に HEM frame sequence 以外を混ぜないこと。
transport failure 時に現在の HEMP session を継続しないこと。
```

## 14. Example Realization Categories

この章は非規範である。

次は、Local Process IPC Binding を実現し得る realization category の例である。

これらは例であり、許可される realization category をこの2つに限定するものではない。

```text
subprocess standard I/O realization
local endpoint realization
```

Subprocess standard I/O realization は、Host が Engine process を起動し、Engine process の stdin / stdout 相当を使って2つの stream を提供する方式である。

Local endpoint realization は、同一コンピュータ上の Host process と Engine process が、local endpoint 相当の接続口を介して2つの stream を提供する方式である。

この文書は、これらの realization category の詳細な API、推奨用途、endpoint naming rule、permission model、timeout value、process lifecycle rule、または library design を定義しない。

どの realization category を使用する場合でも、Local Process IPC Binding の規範要件は同じである。

## 15. Transport Failure

local transport failure は、Local Process IPC Binding instance が HEM frame sequence を継続して送受信できなくなる local transport-level condition である。

local transport failure になり得るものには、少なくとも次を含む。

```text
local transport establishment failure
unexpected stream close
unexpected pipe close
unexpected endpoint close
EOF
I/O error
timeout
peer process termination
local transport reset
local transport resource exhaustion
ordering unit continuation failure
communication path continuation failure
```

具体的に何を local transport failure とするかは、concrete realization または実装が定義しなければならない。

Local Process IPC Binding は、local transport failure を HEMP Core semantics として再定義してはならない。

Local Process IPC Binding は、local transport failure を通常の HEM として扱ってはならない。

Local Process IPC Binding は、local transport failure を HEM Header、HEM Body、Protocol channel message、Application channel message、または HEMP failure response として扱ってはならない。

Local Process IPC Binding は、local transport failure を HEMP failure classification に新しい分類として追加しない。

## 16. `abort = true` and Local Transport Failure

`abort = true` は、HEMP Core 上の logical send を sender-side aborted termination として終端するための Header field である。

`abort = true` の HEM も、通常の complete HEM frame として local transport 上で運ばれなければならない。

`abort = true` は、次を意味しない。

```text
stream close
pipe close
endpoint close
EOF
process termination
I/O error
timeout
local transport reset
local transport failure
```

Local Process IPC Binding は、stream close、pipe close、endpoint close、EOF、process termination、I/O error、timeout、local transport reset、または local transport failure を、`abort = true` の HEM の代替として扱ってはならない。

`abort = true` の HEM を送信しようとしている途中で local transport failure が発生し、receiver がその HEM を complete HEM frame として構成できなかった場合、その `abort` は HEMP Core 上成立しない。

この場合、未完了の logical send を aborted 状態で完了したものとして扱ってはならない。

Local Process IPC Binding は、local transport failure を理由に、logical send が HEMP Core 上 aborted termination したものとして補正してはならない。

`abort = true` の HEM が complete HEM frame として受信され、その Header、flow、および Body Contract が有効である場合に限り、その HEM は HEMP Core 上の abort として扱われる。

## 17. Session Continuation After Local Transport Failure

Local Process IPC Binding instance が local transport failure によって継続不能になった場合、その Local Process IPC Binding instance 上の現在の HEMP session も継続不能として扱わなければならない。

Local Process IPC Binding は、local transport failure 後に同じ HEMP session を再開してはならない。

Local Process IPC Binding は、local transport failure 後に、同じ HEMP session の agreement、limits、channel state、thread state、flow state、または in-flight logical send state を継続してはならない。

再度 HEMP communication を行う場合は、新しい HEMP session として agreement から開始しなければならない。

Local Process IPC Binding instance を終了済み HEMP session の後に再利用できるかどうかは、concrete realization または実装が定義してよい。

ただし、再利用後に HEMP communication を行う場合でも、それは新しい HEMP session として扱わなければならない。

## 18. Retry and Reconnect Policy

この文書は、retry policy または reconnect policy を定義しない。

Local Process IPC Binding は、local transport failure を自動 retry または reconnect によって同じ HEMP session の継続として隠蔽してはならない。

Concrete realization または実装は、local transport failure 後に新しい local transport context を確立してよい。

ただし、その local transport context 上で HEMP communication を行う場合は、新しい HEMP session として agreement から開始しなければならない。

## 19. Non-Goals

この文書は、次を定義しない。

```text
HEMP Core semantics の変更
HEM Payload encoding の変更
HEM Length Header の別形式
HEM Length Header の省略
specific OS API
specific runtime
specific async framework
specific transport library
specific process launch method
specific pipe implementation
specific endpoint mechanism
specific endpoint naming rule
specific permission model
specific timeout value
specific retry policy
specific reconnect policy
network transport binding
loopback network binding
in-memory / test transport as standard binding
Python API
Rust API
```

Loopback network transport は、Local Process IPC Binding の標準 realization として定義しない。

In-memory transport または test transport は、Local Process IPC Binding の標準 realization として定義しない。

これらは、実装、単体テスト、conformance test、または別仕様で必要に応じて扱ってよい。

## 20. Relationship to Other Documents

この文書は、Local Process IPC Binding の concrete definition を定義する。

`02-transport-model.md` は、Transport Binding の共通 transport model、real communication direction、ordering unit、Transport Binding instance、および HEMP session との関係を定義する。

`03-transport-requirements-and-frame-handling.md` は、transport requirements、frame handling、transport failure、および `abort` と transport failure の境界を定義する。

`04-transport-profiles.md` は、Byte Stream Profile、Message Boundary Profile、および HEM frame interleave を定義する。

Local Process IPC Binding は、これらの共通 Transport Binding rules に従う。
