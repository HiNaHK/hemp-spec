# HEMP Transport Requirements and Frame Handling

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Transport Binding specification

## 1. Purpose

この文書は、HEMP Transport Binding における transport requirements と frame handling rules を定義する。

Transport Binding は、HEMP Core が定義する HEM frame を、具体的な transport 上で送受信できるようにしなければならない。

この文書では、次を定義する。

```text
reliability requirement
ordering requirement
bidirectionality requirement
HEM frame boundary handling
complete HEM frame delivery
HEM frame sequence integrity
non-HEM data separation
direction validation
frame construction failure
transport failure
relationship between transport failure and HEMP failure classification
relationship between abort and transport failure
session continuation after transport failure
```

この文書は、HEMP Core semantics を再定義しない。

この文書は、Byte Stream Profile、Message Boundary Profile、または concrete Transport Binding の詳細規則を定義しない。

## 2. Transport Requirements Overview

Transport Binding は、HEM frame sequence を安全に運ぶために、少なくとも次の性質を満たさなければならない。

```text
reliability
ordering
bidirectionality
HEM frame boundary handling
complete HEM frame delivery
HEM frame sequence integrity
separation of HEM frame sequence from non-HEM data
```

Transport Binding は、HEM frame を complete HEM frame として HEMP Core の処理へ渡せるようにしなければならない。

Transport Binding は、HEM frame sequence を壊してはならない。

Transport Binding は、HEM frame sequence に HEM 以外の data を混ぜてはならない。

Transport Binding は、下位 transport の性質を HEMP Core semantics として再定義してはならない。

## 3. Reliability Requirement

Transport Binding は、正常に確立された transport 上で送信された HEM frame sequence が、欠落、重複、破損、途中消失なしに受信されることを要求する。

Transport Binding は、下位 transport が HEM frame sequence を信頼できる形で届けられることを前提にする。

HEMP は、下位 transport の欠落、再送、重複排除、破損訂正、または順序修復を定義しない。

Transport Binding は、信頼性を満たさない transport を、有効な Transport Binding として定義してはならない。

Concrete Transport Binding は、使用する transport-specific resource が reliability requirement をどのように満たすかを定義しなければならない。

## 4. Ordering Requirement

Transport Binding は、各 ordering unit 内で、送信された HEM frame sequence が送信順と同じ順序で受信されることを要求する。

ordering unit は、Transport Binding が HEM frame sequence の順序を保持する transport 上の単位である。

Transport Binding は、少なくとも次の ordering unit を提供しなければならない。

```text
Host -> Engine 方向の HEM frame sequence を運ぶ ordering unit
Engine -> Host 方向の HEM frame sequence を運ぶ ordering unit
```

ordering unit 内で HEM frame sequence の順序が崩れる transport を、有効な Transport Binding として定義してはならない。

HEMP Core の `seq` は、下位 transport の順序崩れを修復するための値ではない。

`seq` は、HEMP Core 上で、同一 logical send 内の HEM の順序を検証するための Header field である。

Transport Binding は、下位 transport の順序崩れを `seq` によって修復するものとして定義してはならない。

## 5. Bidirectionality Requirement

Transport Binding は、Host -> Engine と Engine -> Host の両方向に HEM frame を送受信できなければならない。

Host -> Engine 方向は、Host から Engine へ HEM frame sequence を送る実通信方向である。

Engine -> Host 方向は、Engine から Host へ HEM frame sequence を送る実通信方向である。

Bidirectionality は、1つの双方向 transport によって実現してもよい。

Bidirectionality は、2つの単方向 transport の組によって実現してもよい。

Bidirectionality は、transport-specific な channel、stream、queue、message abstraction などによって実現してもよい。

ただし、どの実現方式であっても、Transport Binding は Host -> Engine と Engine -> Host の実通信方向を区別できなければならない。

Concrete Transport Binding は、両方向の HEM frame sequence をどの transport-specific resource に載せるかを定義しなければならない。

## 6. HEM Frame Boundary

Transport Binding は、HEM frame boundary を、適用される Transport Binding Profile に従って扱わなければならない。

HEM frame boundary は、transport の read / write / flush / buffer operation の境界そのものではない。

次を、それ自体として HEM frame boundary とみなしてはならない。

```text
read boundary
write boundary
flush boundary
buffer boundary
line break
```

Transport Binding は、read / write / buffer operation がどのように分割または結合されても、HEMP Core の処理へ渡す HEM frame を complete HEM frame として構成できなければならない。

Transport message boundary は、適用される Transport Binding Profile が HEM frame boundary として定義する場合に限り、HEM frame boundary として扱ってよい。

Byte Stream Profile と Message Boundary Profile における具体的な HEM frame boundary rule は、`04-transport-profiles.md` で定義する。

## 7. Complete HEM Frame Delivery

Transport Binding は、HEMP Core の処理へ HEM を渡す前に、complete HEM frame を構成しなければならない。

complete HEM frame は、HEM Length Header と、HEM Length Header が示す HEM Payload bytes を含む完全な HEM frame である。

Transport Binding は、次を complete HEM frame として扱ってはならない。

```text
partial frame
truncated frame
incomplete HEM Length Header
HEM Length Header だけを読めた状態
incomplete HEM Payload
declared length 分の Payload bytes を取得できていない状態
transport failure により途中で途切れた frame
```

Transport Binding は、complete HEM frame を構成できない状態を、正常な HEM reception として扱ってはならない。

Transport Binding は、不完全な HEM frame を HEMP Core validation に渡してはならない。

## 8. HEM Frame Sequence Integrity

Transport Binding は、HEM frame sequence を壊してはならない。

Transport Binding は、HEM Length Header と HEM Payload の順序および連続性を壊してはならない。

Transport Binding は、1つの HEM frame の内部に、別の HEM frame または HEMP 外の byte sequence を挿入してはならない。

Transport Binding は、複数の HEM frame を、Core semantics 上の1つの HEM frame に結合してはならない。

Transport Binding は、1つの HEM frame を、Core semantics 上の複数の HEM frame に分割してはならない。

Transport Binding は、HEM frame sequence 内の HEM frame を、Transport Binding 独自の意味によって logical send に分類してはならない。

Logical send の解釈は、HEMP Core が定義する `channel`、`thread`、`role`、`seq`、`end`、`close`、`abort` に従う。

HEM frame interleave を許すかどうか、および許す場合の単位は、`04-transport-profiles.md` で定義する。

ただし、いかなる場合でも、1つの HEM frame の内部に別の HEM frame を interleave してはならない。

## 9. Non-HEM Data Separation

Transport Binding は、HEM frame sequence に HEM 以外の data を混ぜてはならない。

次を HEM frame sequence に混ぜてはならない。

```text
log output
diagnostic output
debug text
human-readable message
transport-specific control text
non-HEMP control data
```

ログ、診断出力、デバッグ文字列、人間向けメッセージ、または HEMP 外の制御 data が必要な場合、それらは HEM frame sequence とは別の経路で扱わなければならない。

HEM frame sequence に HEM 以外の data が混入した場合、receiver はそれを Transport Binding 独自の補助情報として解釈してはならない。

HEM frame sequence に混入した HEM 以外の data を、HEM Length Header、HEM Payload、または HEM Body として補正して扱ってはならない。

## 10. Direction Validation

Transport Binding は、受信した HEM frame の実通信方向と HEM Header の `direction` field が一致することを検証できなければならない。

対応関係は次の通りである。

```text
direction = to_engine:
  HEM frame は Host -> Engine 方向で受信されなければならない。

direction = to_host:
  HEM frame は Engine -> Host 方向で受信されなければならない。
```

実通信方向と HEM Header の `direction` field が一致しない場合、receiver は HEMP header validation failure として扱わなければならない。

Transport Binding は、実通信方向と `direction` field の不一致を自動補正してはならない。

Transport Binding は、不一致を反対方向への routing instruction として扱ってはならない。

Transport Binding は、不一致を transport-specific な別意味として解釈してはならない。

## 11. Frame Construction Failure

Frame construction failure は、Transport Binding が受信 data から complete HEM frame を構成できない状態である。

Frame construction failure には、少なくとも次が含まれる。

```text
HEM Length Header を complete に取得できない。
HEM Length Header が不正である。
declared Payload length 分の bytes を取得できない。
Payload bytes が途中で途切れる。
frame read limit により Payload bytes を安全に取得できない。
transport-level frame resource limit により Payload bytes を安全に取得できない。
applicable Transport Binding Profile に従って complete HEM frame を構成できない。
```

Transport Binding は、frame construction failure を正常な HEM reception として扱ってはならない。

Transport Binding は、complete HEM frame を構成できなかった data を、有効な HEM として扱ってはならない。

Transport Binding は、complete HEM frame を構成できなかった data を HEMP Core validation に渡してはならない。

complete HEM frame を構成できない failure は、該当する場合、HEMP framing failure として扱う。

complete HEM frame を構成できた後の Payload decode failure、Payload validation failure、Header validation failure、flow violation、または HEM Body Contract failure は、HEMP Core および HEMP Payload encoding specification が定義する failure classification に従う。

ただし、frame construction failure の原因が下位 transport の close、EOF、I/O error、timeout、reset、process termination、resource exhaustion、または通信路継続不能である場合、Transport Binding instance は継続不能であり、現在の HEMP session も継続不能として扱わなければならない。

この場合、failure classification と session continuation の扱いは区別される。

## 12. Transport Failure

transport failure は、Transport Binding instance が HEM frame sequence を継続して送受信できなくなる transport-level condition である。

transport failure になり得るものには、少なくとも次を含む。

```text
transport connection establishment failure
unexpected stream close
unexpected pipe close
unexpected endpoint close
EOF
I/O error
timeout
transport reset
peer process termination
transport resource exhaustion
ordering unit continuation failure
communication path continuation failure
```

具体的に何を transport failure とするかは、concrete Transport Binding が定義しなければならない。

Transport Binding は、transport failure を HEMP Core semantics として再定義してはならない。

Transport Binding は、transport failure を通常の HEM として扱ってはならない。

Transport Binding は、transport failure を HEM Header、HEM Body、Protocol channel message、または Application channel message として扱ってはならない。

## 13. Transport Failure and HEMP Failure Classification

transport failure は、HEMP Core の failure classification に新しい分類として追加しない。

HEMP Core の failure classification は、HEMP Core および HEMP Payload encoding specification が定義する分類に従う。

Transport Binding は、transport failure を次のような HEMP failure classification の追加分類として定義してはならない。

```text
transport failure as a sixth HEMP failure class
transport close as HEMP flow violation by itself
EOF as abort
timeout as abort
process termination as abort
```

Transport Binding は、complete HEM frame を構成できた後の validation failure を、HEMP Core および HEMP Payload encoding specification の failure classification に従って扱わなければならない。

Transport Binding は、complete HEM frame を構成できない transport-level condition を、有効な HEMP failure response によって回復したものとして扱ってはならない。

Transport Binding は、transport failure によって継続不能になった HEMP session を、HEMP Core 上の通常の failure response によって継続可能になった session として扱ってはならない。

transport failure が発生した場合の session continuation は、failure classification とは別に扱う。

## 14. `abort = true` and Transport Failure Boundary

`abort = true` は、HEMP Core 上の logical send を sender-side aborted termination として終端するための Header field である。

`abort = true` の HEM も、通常の complete HEM frame として transport 上で運ばれなければならない。

`abort = true` は、次を意味しない。

```text
transport close
stream close
pipe close
endpoint close
EOF
process termination
transport reset
I/O error
timeout
transport failure
```

Transport Binding は、transport close、stream close、pipe close、endpoint close、EOF、process termination、I/O error、timeout、transport reset、または transport failure を、`abort = true` の HEM の代替として扱ってはならない。

`abort = true` の HEM を送信しようとしている途中で transport failure が発生し、receiver がその HEM を complete HEM frame として構成できなかった場合、その `abort` は HEMP Core 上成立しない。

この場合、未完了の logical send を aborted 状態で完了したものとして扱ってはならない。

Transport Binding は、transport failure を理由に、logical send が HEMP Core 上 aborted termination したものとして補正してはならない。

`abort = true` の HEM が complete HEM frame として受信され、その Header、flow、および Body Contract が有効である場合に限り、その HEM は HEMP Core 上の abort として扱われる。

## 15. Session Continuation After Transport Failure

Transport Binding instance が transport failure によって継続不能になった場合、その Transport Binding instance 上の現在の HEMP session も継続不能として扱わなければならない。

Transport Binding は、transport failure 後に同じ HEMP session を再開してはならない。

Transport Binding は、transport failure 後に、同じ HEMP session の agreement、limits、channel state、thread state、flow state、または in-flight logical send state を継続してはならない。

再度 HEMP communication を行う場合は、新しい HEMP session として agreement から開始しなければならない。

Transport Binding instance を終了済み HEMP session の後に再利用できるかどうかは、concrete Transport Binding または実装が定義してよい。

ただし、再利用後に HEMP communication を行う場合でも、それは新しい HEMP session として扱わなければならない。

## 16. Retry and Reconnect Policy

この文書は、retry policy または reconnect policy を定義しない。

Transport Binding は、transport failure を自動 retry または reconnect によって同じ HEMP session の継続として隠蔽してはならない。

Concrete Transport Binding または実装は、transport failure 後に新しい transport connection を確立してよい。

ただし、その connection 上で HEMP communication を行う場合は、新しい HEMP session として agreement から開始しなければならない。

## 17. Relationship to Other Documents

この文書は、Transport Binding の transport requirements と frame handling rules を定義する。

`02-transport-model.md` は、実通信方向、ordering unit、HEM frame mapping、Transport Binding instance、および HEMP session との関係を定義する。

`04-transport-profiles.md` は、Byte Stream Profile、Message Boundary Profile、および HEM frame interleave の詳細を定義する。

`local-ipc/specification.md` は、Local Process IPC Binding の concrete definition を定義する。
