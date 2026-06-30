# Local Endpoint Transport Adapter Overview and Scope

Status: Review Draft / Candidate Transport Adapter Rules
Specification version: Not assigned
Scope: HEMP Transport Binding local endpoint adapter rules

## 1. Purpose

この文書は、local endpoint 型 Transport-side Adapter を実装する場合の範囲と責任境界を定義する。

この文書は、特定のプログラム言語、runtime、OS API、transport library、または実装コードを定義しない。

この文書は、HEMP Core semantics、HEM Payload encoding、HEM Length Header、HEM Header field semantics、HEM Body semantics、または HEMP failure classification を再定義しない。

この文書は、`../../adapter-architecture/` 配下の共通責任境界に従う。

## 2. Local endpoint 型

この文書では、**local endpoint 型** を、同一コンピュータ内の process 間で接続するための、名前付きまたは識別可能なローカル接続口から得られた Transport I/O を扱う transport 型として定義する。

local endpoint 型は、特定の OS API、特定の endpoint mechanism、または特定の library API を意味しない。

この文書は、Unix domain socket、Windows named pipe、または特定の socket API を規範要件として要求しない。

## 3. Relationship to Transport-side Adapter

local endpoint 型 Transport-side Adapter は、接続済みまたは確立済みの Transport I/O を包む。

local endpoint 型 Transport-side Adapter は、HEMP-side Adapter に対して、HEM delivery path-aware な読み書きと transport failure reporting を提供する。

ここでの読み書きは、具体的な API 名ではない。

ここでの読み書きは、Transport-side Adapter が提供する責任を説明する一般表現である。

ここでいう transport failure reporting は、transport failure を上位へ伝えられる責任を指す。

transport failure reporting は、具体的な API 名ではない。

local endpoint 型 Transport-side Adapter は、HEMP Core semantics を解釈しない。

local endpoint 型 Transport-side Adapter は、HEM Payload decode、HEM Header validation、direction validation、flow validation、または session state 更新を行わない。

## 4. local endpoint と HEM delivery path

local endpoint そのものは、HEM delivery path ではない。

local endpoint から得られた Transport I/O は、HEM delivery path を実現するために使われる transport-specific resource である。

local endpoint 型 Transport-side Adapter は、対象 Transport I/O がどの HEM delivery path に対応するかを HEMP-side Adapter へ示せなければならない。

HEM delivery path は、process、pipe、socket、endpoint、file descriptor、stream object、transport library object、または runtime object と同義ではない。

それらは、HEM delivery path を実現するために使用してよい transport-specific resource である。

## 5. Transport I/O scope

local endpoint 型 Transport-side Adapter が扱う Transport I/O は、local endpoint から得られた接続済みまたは確立済みの Transport I/O である。

local endpoint 型 Transport-side Adapter は、endpoint の作成、発見、または接続確立を最小責任として要求しない。

local endpoint 型 Transport-side Adapter は、endpoint の作成や接続を行う外側の処理から、接続済みまたは確立済みの Transport I/O を受け取ってよい。

この文書は、local endpoint 型 Transport-side Adapter が OS API を直接呼び出すことを要求しない。

この文書は、local endpoint 型 Transport-side Adapter が endpoint setup 全体を所有することを要求しない。

## 6. Responsibilities not included

この文書は、次を HEMP Core、HEMP-side Adapter、または Transport-side Adapter の最小責任として要求しない。

```text
endpoint creation
endpoint naming
endpoint discovery
接続待受
connection establishment
permission / ACL
authentication
peer confirmation
timeout 方針
timeout 値
stale endpoint handling
process launch
retry
reconnect
session restart
```

これらは、必要に応じて application、transport setup、外部 transport library、OS API、実装 repository、または Transport-side Adapter の外側にある統合処理で扱う。

ただし、これらを同じ HEMP session の retry、reconnect、または resume として隠蔽してはならない。

## 7. HEM frame sequence separation

local endpoint 型 Transport-side Adapter が HEMP-side Adapter へ渡す Transport I/O は、HEM frame sequence を運ぶための I/O として扱われる。

対象 Transport I/O には、HEM frame sequence 以外の data を混ぜてはならない。

次は、HEM frame sequence と同じ ordered delivery unit に混ぜてはならない。

```text
log output
diagnostic output
debug text
human-readable message
transport-specific control text
non-HEMP control data
local endpoint control data
```

ログ、診断出力、デバッグ文字列、または人間向けメッセージが必要な場合は、HEM frame sequence とは別の経路で扱う。

## 8. HEM frame boundary の扱い

local endpoint 型の Transport I/O における read 単位、write 単位、flush 単位、buffer 境界、line break、または実装上の分割単位は、HEM frame boundary ではない。

HEMP-side Adapter は、対象 Transport Binding Profile の規則に従って complete HEM frame を構成する。

Byte Stream Profile を使用する場合、HEM frame boundary は HEM Length Header によって判断する。

complete HEM frame を構成できない data は、HEMP Core validation へ渡してはならない。

## 9. 接続準備の異常と transport failure

local endpoint 型では、接続済みまたは確立済みの Transport I/O を得る前に、接続準備の異常が発生し得る。

接続準備の異常になり得るものには、次がある。

```text
endpoint が存在しない
endpoint を発見できない
endpoint に接続できない
permission / ACL により接続できない
peer を確認できない
connection establishment が完了しない
```

接続準備の異常は、HEMP Core failure ではない。

接続済みまたは確立済みの Transport I/O を使っている途中で、transport failure が発生し得る。

通信中の transport failure になり得るものには、次がある。

```text
接続済み Transport I/O が閉じられた
EOF
I/O error
timeout
peer process termination
Transport Binding instance の継続不能
HEM delivery path の継続不能
```

どの事象を transport failure として扱うかは、Transport Binding 共通仕様、`../../adapter-architecture/` の共通責任境界、およびこの文書と矛盾しない範囲で明確にする。

transport failure は HEMP failure classification ではない。

transport failure を HEMP failure response として扱ってはならない。

transport failure を `abort = true` の代替として扱ってはならない。

transport 固有の詳細は、Transport-side Adapter 側で保持または報告してよい。

ただし、HEMP Core / HEMP-side Adapter は、transport 固有の詳細を HEMP Core semantics として解釈しない。

## 10. Session continuation

Transport Binding instance または HEM delivery path が継続不能になった場合、現在の HEMP session は継続不能として扱う。

一度継続不能になった HEMP session を、同じ session のまま retry、reconnect、または resume してはならない。

再度 HEMP communication を行う場合は、新しい HEMP session として agreement から開始する。

## 11. 実装 repository との関係

この文書は、言語非依存の transport adapter rules を扱う。

具体的なプログラム言語、OS API、transport library、class 名、function 名、module 構成、sync / async API、endpoint name、permission / ACL、timeout 値、または実装コードは、実装 repository 側で扱う。

この文書は、Python 実装、Rust 実装、またはその他の言語実装の API 設計を定義しない。
