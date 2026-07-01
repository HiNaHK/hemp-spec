# Failure and Session Boundaries

Status: Review Draft / Candidate Transport Adapter Architecture Rules
Specification version: Not assigned
Scope: HEMP Transport Binding adapter architecture
Draft revision: v0.3

## 1. Purpose

この文書は、transport failure、frame construction failure、HEMP Core の処理結果、HEMP session 継続可否を混ぜないための共通ルールを定義する。

この文書は、特定 transport 型の failure detail を定義しない。

この文書は、Python の exception class、result object、callback、async API、または実装コードを定義しない。

## 2. Failure Information by Layer

Transport-side Adapter は、transport failure と transport 固有の詳細を扱う。

HEMP-side Adapter は、complete HEM frame を構成できたかどうかを扱う。

HEMP Core は、HEM Payload decode、HEM Header validation、direction validation、flow validation、failure classification、session state を扱う。

HEMP session の継続可否は、失敗原因そのものではなく、現在の HEMP session を使い続けてよいかを示す状態である。

これらを、同じ種類の失敗として扱ってはならない。

## 3. Transport Failure Is Not HEMP Failure Classification

transport failure は、HEMP failure classification ではない。

transport close、EOF、timeout、I/O error、process termination、transport reset などを、HEMP failure response として扱ってはならない。

transport failure を、HEM Header、HEM Body、Protocol channel message、Application channel message、または HEMP failure response として扱ってはならない。

transport failure を、HEMP failure classification に新しい分類として追加してはならない。

## 4. Frame Construction Failure

complete HEM frame を構成できない data は、HEMP Core validation へ渡してはならない。

frame construction failure は、HEMP Core validation failure ではない。

frame construction failure を、HEMP failure response の受信として扱ってはならない。

frame construction failure を、`abort = true` の受信として扱ってはならない。

## 5. Pending Frame Construction

まだ complete HEM frame を構成できない状態は、ただちに failure ではない。

追加 data を待つ状態と、frame construction failure は分ける。

Byte Stream Profile では、次を HEM frame boundary として扱わない。

```text
read 単位
write 単位
flush 単位
buffer 境界
line break
実装上の分割単位
```

そのため、途中まで受信した data だけを理由に、HEMP Core validation failure として扱ってはならない。

## 6. HEMP Core Processing Result

HEMP Core は、HEM Payload bytes と received HEM delivery path を受け取った後の decode / validation / flow / failure classification / session state を扱う。

HEMP Core は、次を扱わない。

```text
Transport I/O
endpoint
socket
pipe
stream object
file descriptor
transport library object
runtime object
EOF
timeout
I/O error
transport 固有の詳細
```

HEMP Core は、transport failure を HEMP Core semantics として補正してはならない。

## 7. HEMP Session Continuation State

Transport Binding instance または HEM delivery path が継続不能になった場合、現在の HEMP session は継続不能として扱う。

一度継続不能になった HEMP session を、同じ HEMP session のまま retry / reconnect / resume してはならない。

再度 HEMP communication を行う場合は、新しい HEMP session として agreement から開始する。

HEMP session 継続不能は、HEMP failure response ではない。

HEMP session 継続不能は、`abort = true` ではない。

HEMP session 継続不能は、HEMP Core validation failure ではない。

## 8. Abort and Transport Failure Boundary

transport close、EOF、timeout、I/O error、process termination、transport reset などを、`abort = true` の代替として扱ってはならない。

`abort = true` の HEM は、通常の complete HEM frame として transport 上で運ばれなければならない。

`abort = true` は、complete HEM frame として受信され、その Header、flow、および HEM Body Contract が有効である場合に限り、HEMP Core 上の abort として扱われる。

`abort = true` の HEM を送信しようとしている途中で transport failure が発生し、受信側がその HEM を complete HEM frame として構成できなかった場合、その abort は HEMP Core 上では成立しない。

transport failure を理由に、未完了の logical send が HEMP Core 上で aborted 状態で完了したものとして補正してはならない。

## 9. Transport-specific Details

transport 固有の詳細は、Transport-side Adapter 側で保持または報告してよい。

transport 固有の詳細には、次に由来する情報が含まれ得る。

```text
transport 型
runtime
library
OS API
endpoint
process
pipe
socket
stream object
file descriptor
close reason
error code
transport library error
```

HEMP Core / HEMP-side Adapter は、transport 固有の詳細を HEMP Core semantics として解釈しない。

transport 固有の詳細を、HEMP failure classification、`abort = true`、HEMP failure response の代替として扱ってはならない。

## 10. Sending, Receiving, and Waiting During Session Continuation Failure

送信中に transport が継続不能になった場合、対象 HEM が相手に届いたとは扱わない。

送信中に transport が継続不能になった場合、対象 HEM が相手に届かなかったとも HEMP Core semantics として断定しない。

受信中に complete HEM frame を構成できない場合、その data は HEMP Core validation へ渡さない。

応答待ち中に HEMP session が継続不能になった場合、期待していた HEMP response、failure response、または abort を受け取ったとは扱わない。

complete HEM frame が HEMP Core に渡された後に transport が継続不能になった場合、HEMP Core は受け取った HEM Payload bytes の処理を行ってよい。

ただし、その処理結果を同じ HEMP session 上で送信できるとは扱わない。

## 11. 利用側へ示す情報

利用側へ示す情報では、少なくとも次を区別できる必要がある。

```text
接続準備に失敗した。
通信中に transport が継続不能になった。
complete HEM frame を構成できなかった。
HEMP Core の処理結果。
HEMP session は未開始 / 継続可 / 継続不可。
```

ここでいう「接続準備に失敗した」は、Transport-side Adapter へ渡せる接続済みまたは確立済みの Transport I/O を得る前に発生する異常を含む。

これらの異常を、HEMP failure classification、HEMP failure response、または HEMP Core の処理結果として扱ってはならない。

ここでいう「通信中に transport が継続不能になった」は、接続済みまたは確立済みの Transport I/O を使っている途中で、Transport Binding instance または HEM delivery path が継続不能になった状態を含む。

この区別は、Transport Binding 仕様本文における transport failure の定義や、concrete Transport Binding が何を transport failure として扱うかの定義を置き換えない。

この文書は、これらを具体的な exception class、result object、enum、field、callback、event、または API signature として定義しない。

## 12. Relationship to transport-adapters/

この文書は、全 Transport-side Adapter に共通する failure / session 境界を定義する。

各 transport 型に固有の failure detail は、`../transport-adapters/<transport-type>/` 配下で必要に応じて扱う。

各 transport 型の文書は、この文書で定義する共通の failure / session 境界に従う。
