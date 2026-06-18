# Local Process IPC Binding 実現ガイド ドラフト v2.2

Status: Working Draft  
Date: 2026-06-15  
Scope: Host-Engine Message Protocol / HEMP Local Process IPC Binding Realization Guide  
Revision: v2.2 clarification of `abort = true` HEM delivery and local transport failure boundary  

---
# 1. 目的

この文書は、HEMP Transport Binding Specification が定義する **Local Process IPC Binding** を、実際のローカルtransport上で実現するための候補と整理観点を示す。

この文書は、HEMP Core仕様または HEMP Transport Binding仕様を再定義しない。

この文書は、Local Process IPC Binding が定義する `host_to_engine stream` と `engine_to_host stream` を、実際の環境でどのように提供できるかを整理する。

HEMP Transport Binding Specification において、`host_to_engine stream` は Host -> Engine 方向の順序保証単位であり、`engine_to_host stream` は Engine -> Host 方向の順序保証単位である。

この文書は、特定の言語、runtime、async framework、transport library、OS APIをHEMPとして要求しない。

この文書では、次を扱う。

```text
Local Process IPC Binding と実現候補の関係
実現候補に求める共通条件
子プロセス標準入出力型
ローカルendpoint型
用途別推奨
実現候補ごとの整理項目
非対象範囲
```

---
# 2. Transport Binding仕様との関係

HEMP Transport Binding Specification は、Local Process IPC Binding を定義する。

Local Process IPC Binding は、同一コンピュータ上で動作する Host process と Engine process の間で HEMP を送受信するための Transport Binding である。

Local Process IPC Binding は、Byte Stream Profile を使用する。

Local Process IPC Binding は、Host -> Engine 方向に1つの順序保証単位、Engine -> Host 方向に1つの順序保証単位を定義する。

Local Process IPC Binding の各順序保証単位は、Byte Stream Profile を使用する。

Local Process IPC Binding は、次の2つの論理単方向byte streamを定義する。

```text
host_to_engine stream:
  Host から Engine へ HEM frame sequence を送る、Host -> Engine 方向の順序保証単位。

engine_to_host stream:
  Engine から Host へ HEM frame sequence を送る、Engine -> Host 方向の順序保証単位。
```

この実現ガイドは、この2つの論理単方向byte streamを、実際のローカルtransport上でどう提供できるかを整理する。

```text
Transport Binding仕様:
  何を満たす必要があるかを定義する。

実現ガイド:
  それを現実の環境でどう実現するかを整理する。
```

この文書で示す実現候補は、Local Process IPC Binding の要件を満たすための整理である。  
この文書で示す実現候補は、HEMP Core仕様や HEMP Transport Binding仕様の内容を変更しない。

---
# 3. 実現候補に求める共通条件

Local Process IPC Binding の実現候補は、少なくとも次の条件を満たさなければならない。

```text
信頼性
順序保証
双方向性
byte streamとして扱えること
host_to_engine streamを Host -> Engine 方向の順序保証単位として提供できること
engine_to_host streamを Engine -> Host 方向の順序保証単位として提供できること
HEM frame sequenceを破壊しないこと
HEM streamにHEM frame sequence以外を混ぜないこと
```

## 3.1 信頼性

正常なtransport接続上で送信されたbyte列が、欠落、重複、破損、途中消失なしに届かなければならない。

HEMPは、下位transportの欠落、再送、重複排除、破損訂正を定義しない。

## 3.2 順序保証

Local Process IPC Binding では、`host_to_engine stream` と `engine_to_host stream` のそれぞれが順序保証単位である。

各streamの内部では、送信されたbyte列が、送信順と同じ順序で届かなければならない。

HEMPの `seq` は、HEMP上の論理送信内の順序を検証するための値である。  
`seq` は、下位transportの順序崩れを修復するための再送制御や並べ替え機構ではない。

## 3.3 双方向性

Host -> Engine と Engine -> Host の両方向に送信できなければならない。

双方向性は、1つの双方向通信路で実現してもよい。  
または、2つの単方向通信路の組で実現してもよい。

## 3.4 byte streamとして扱えること

実現候補は、HEM frame sequence を連続したbyte列として送受信できなければならない。

HEM frame の境界は、HEM Length Header によって判断する。

read単位、write単位、flush単位、buffer境界、改行、JSONの1行 / 複数行は、HEM frame境界ではない。

## 3.5 host_to_engine stream

実現候補は、Host から Engine へ HEM frame sequence を送る `host_to_engine stream` を提供しなければならない。

`host_to_engine stream` は、Local Process IPC Binding における Host -> Engine 方向の順序保証単位である。

`direction = "to_engine"` のHEMは、`host_to_engine stream` 上で送信する。

## 3.6 engine_to_host stream

実現候補は、Engine から Host へ HEM frame sequence を送る `engine_to_host stream` を提供しなければならない。

`engine_to_host stream` は、Local Process IPC Binding における Engine -> Host 方向の順序保証単位である。

`direction = "to_host"` のHEMは、`engine_to_host stream` 上で送信する。

## 3.7 HEM frame sequenceを破壊しないこと

実現候補は、HEM Length Header と HEM Payload の順序や連続性を壊してはならない。

1つの HEM frame の内部に、別の HEM frame またはHEM以外のbyte列を挿入してはならない。

interleave は complete HEM frame 単位に限る。

`abort = true` のHEMも、HEM frame sequence 内では通常の complete HEM frame として扱う。

`abort = true` は、HEM frame sequence の終端、byte stream の終端、pipe close、endpoint close、または process終了を意味しない。

## 3.8 HEM以外をstreamへ混ぜないこと

`host_to_engine stream` と `engine_to_host stream` には、HEM frame sequence 以外を流してはならない。

次を HEM stream へ混ぜてはならない。

```text
ログ
診断出力
デバッグ文字列
人間向けメッセージ
HEMP外の制御byte列
```

ログや診断出力が必要な場合は、HEM stream とは別の経路を使用する。

---
# 4. 実現候補の分類

この文書では、Local Process IPC Binding の主要な実現候補を次の2分類で扱う。

```text
子プロセス標準入出力型
ローカルendpoint型
```

`loopback network型` は主要候補には含めない。

---
## 4.1 子プロセス標準入出力型

子プロセス標準入出力型は、Host が Engine process を子プロセスとして起動し、Engine process の stdin / stdout を使って Local Process IPC Binding を実現する方式である。

### 4.1.1 概要

```text
Host:
  Engine process を子プロセスとして起動する。

Engine:
  stdin から HEM frame sequence を読み取る。
  stdout へ HEM frame sequence を書き込む。
```

この方式では、Host が Engine process のライフサイクルを管理する。

### 4.1.2 stream対応

```text
host_to_engine stream:
  Host が Engine process の stdin に HEM frame sequence を書き込む。
  Engine は stdin から HEM frame sequence を読み取る。
  Host -> Engine 方向の順序保証単位として扱う。

engine_to_host stream:
  Engine が stdout に HEM frame sequence を書き込む。
  Host は Engine process の stdout から HEM frame sequence を読み取る。
  Engine -> Host 方向の順序保証単位として扱う。
```

### 4.1.3 出力の分離

子プロセス標準入出力型では、Engine stdout に HEM frame sequence 以外を出力してはならない。

```text
Engine stdin:
  host_to_engine stream 専用。

Engine stdout:
  engine_to_host stream 専用。
  HEM frame sequence 以外を出力してはならない。

Engine stderr:
  log / diagnostic output 用。
```

Engine stdout にログや人間向けメッセージが混ざると、Host はそれを HEM Length Header として解釈しようとするため、framing が壊れる。

### 4.1.4 transport確立の責任

Host は、次を管理する。

```text
Engine process の起動
Engine stdin / stdout のpipe確立
Engine stderr の扱い
Engine process の終了検出
```

HEMP Core は、Engine process の起動方法や、子プロセス管理方法を定義しない。

### 4.1.5 transport failureの扱い

子プロセス標準入出力型では、次を transport failure として扱う。

```text
Engine process を起動できない。
Engine process が予期せず終了した。
Engine stdin / stdout が予期せずcloseした。
stdin / stdout のI/O errorが発生した。
実装が定義したtimeoutに達した。
stdin / stdout が継続不能になった。
```

Engine process終了、Engine stdin / stdout close、stdin / stdout のI/O error、timeoutなどにより、`abort = true` のHEMを complete HEM frame として届けられなかった場合、その `abort` は HEMP Core上では成立しない。

この場合、未完了の論理送信を aborted 状態で完了したものとして扱ってはならない。
現在の HEMP session は transport failure により継続不能として扱う。

transport failure が発生した場合、現在の HEMP session は継続不能として扱う。

transport failure 後に、同じ HEMP session を再開してはならない。

再度通信する場合は、新しい HEMP session として agreement から開始する。

### 4.1.6 利点

```text
構成が単純である。
初期実装しやすい。
Host が Engine process のライフサイクルを管理しやすい。
endpoint命名、接続待受、接続先発見の設計が少なくて済む。
CLI型Engineと相性がよい。
単一Host + 単一Engineの構成に向く。
```

### 4.1.7 注意点

```text
stdoutにログや診断出力を出してはならない。
Engineが独立起動済みの場合には向きにくい。
複数Hostから同じEngineへ接続する構成には向きにくい。
HostがEngine processの起動、終了、stderr収集を管理する必要がある。
stdio自体は認証や暗号化を提供しない。
```

### 4.1.8 ライブラリ実装上の扱い

HEMPライブラリは、stdin / stdout そのものを必須APIとして固定しない。

各言語実装は、その言語で自然な Reader / Writer / Stream 抽象として Engine stdin / stdout を扱ってよい。

HEMPとして必要なのは、`host_to_engine stream` と `engine_to_host stream` を Byte Stream Profile に従って扱えることである。

### 4.1.9 推奨用途

子プロセス標準入出力型は、次の用途に向く。

```text
初期実装
最小構成
CLI型Engine
Host管理型Engine
単一Host + 単一Engineのローカル通信
テストしやすい構成
```

---
## 4.2 ローカルendpoint型

ローカルendpoint型は、同一コンピュータ上で動作する Host process と Engine process が、ローカルendpointを介して接続し、Local Process IPC Binding を実現する方式である。

ここでいう `ローカルendpoint` は、特定のOS APIや特定ライブラリ名を指さない。

```text
ローカルendpoint:
  同一コンピュータ内のプロセス間で接続するための、
  名前付きまたは識別可能なローカル通信口。
```

### 4.2.1 概要

```text
Host:
  ローカルendpointへ接続する。

Engine:
  ローカルendpointを用意し、接続を待受する。
```

典型的には、Engine が待受側、Host が接続側になる。

ただし、Host が Engine を起動して endpoint 情報を渡す構成や、Host 側が endpoint を用意する構成もあり得る。  
この実現ガイドでは、どちらを必須とはしない。

### 4.2.2 stream対応

ローカルendpoint型は、Local Process IPC Binding が要求する2つの論理単方向byte streamを提供する。

```text
host_to_engine stream:
  Host から Engine へ HEM frame sequence を送る。
  Host -> Engine 方向の順序保証単位として扱う。

engine_to_host stream:
  Engine から Host へ HEM frame sequence を送る。
  Engine -> Host 方向の順序保証単位として扱う。
```

実現方法は固定しない。

```text
1つの双方向byte streamを read / write に分けてもよい。

2つの単方向byte streamの組で実現してもよい。
```

HEMPから見た意味は常に同じである。

```text
direction = "to_engine":
  host_to_engine stream 上で送る。

direction = "to_host":
  engine_to_host stream 上で送る。
```

### 4.2.3 endpoint確立の責任

ローカルendpoint型では、endpointの作成、発見、接続は HEMP Core ではなく、実装側が扱う。

実装側は、次を扱う。

```text
endpoint の作成
endpoint の命名
endpoint の発見
接続待受
接続開始
権限設定
既存endpointの扱い
peer確認
close
timeout
peer終了検出
```

### 4.2.4 HEM streamへの混入禁止

ローカルendpoint型でも、HEM streamには HEM frame sequence 以外を混ぜてはならない。

```text
host_to_engine stream:
  HEM frame sequence 専用。

engine_to_host stream:
  HEM frame sequence 専用。
```

ログ、診断出力、デバッグ文字列、人間向けメッセージは、このstreamへ混ぜてはならない。

ログや診断出力が必要な場合は、別のログ出力先、ログファイル、stderr相当、OSログなどの別経路を使用する。

### 4.2.5 transport failureの扱い

ローカルendpoint型では、次を transport failure として扱う。

```text
endpoint を作成できない。
endpoint を発見できない。
endpoint に接続できない。
接続待受に失敗する。
権限不足で接続できない。
接続済みstreamが予期せずcloseする。
peer process が終了する。
I/O error が発生する。
bindingまたは実装が定義したtimeoutに達する。
endpointまたは通信路が継続不能になる。
```

接続済みstreamのclose、peer process終了、I/O error、timeoutなどにより、`abort = true` のHEMを complete HEM frame として届けられなかった場合、その `abort` は HEMP Core上では成立しない。

この場合、未完了の論理送信を aborted 状態で完了したものとして扱ってはならない。
現在の HEMP session は transport failure により継続不能として扱う。

transport failure は HEMP failure分類には追加しない。

transport failure によって現在の HEMP session が継続不能になった場合、HEMP実装はそのsessionを継続不能として扱う。

transport failure 後に、同じ HEMP session を再開してはならない。

再度通信する場合は、新しい HEMP session として agreement から開始する。

### 4.2.6 利点

```text
Engineを独立プロセスとして扱いやすい。
Engineを常駐プロセスとして扱いやすい。
HostとEngineのライフサイクルを分離しやすい。
明示的なローカル接続口を持てる。
複数Engine instanceを実装側で管理しやすい。
```

### 4.2.7 注意点

```text
endpoint命名規則を実装側で決める必要がある。
endpoint情報の受け渡し方法を実装側で決める必要がある。
古いendpointや孤立したendpointの扱いを実装側で決める必要がある。
権限設定や接続可能範囲を管理する必要がある。
接続先が想定Engineであることは、HEMP agreementやapplication identityで確認する必要がある。
子プロセス標準入出力型より初期実装は重くなりやすい。
HEM streamにログや診断出力を混ぜてはならない。
```

### 4.2.8 ライブラリ実装上の扱い

HEMPライブラリ本体は、ローカルendpoint機能そのものを必須実装にしない。

```text
HEMPライブラリ本体:
  HEM frame / codec / session / validation を扱う。

transport adapter:
  ローカルendpointから得た Reader / Writer / Stream を
  host_to_engine / engine_to_host stream として扱う。
```

HEMPライブラリは、ローカルendpointを全部作るライブラリではない。  
HEMPライブラリは、ローカルendpointが提供するbyte streamに HEMPを載せるための層として設計する。

### 4.2.9 推奨用途

ローカルendpoint型は、次の用途に向く。

```text
独立プロセス型Engine
常駐Engine
Hostが必ずしもEngineを子プロセスとして起動しない構成
HostとEngineのライフサイクルを分離したい構成
明示的なローカル接続口を持ちたい構成
複数Engine instanceを実装側で管理したい構成
```

複数接続を扱う場合でも、HEMP sessionを1つのBinding instance上で多重化するわけではない。

```text
1 Local Process IPC Binding instance = 1 HEMP session
```

複数接続に対応する場合は、実装側が複数のBinding instanceを管理する。

---
# 5. 用途別推奨

Local Process IPC Binding 実現ガイドでは、単一の絶対的な標準実現方式を定義しない。

代わりに、Host / Engine の構成に応じて用途別の推奨方式を示す。

## 5.1 初期実装・最小構成

初期実装、最小構成、Host管理型Engineでは、子プロセス標準入出力型を推奨する。

```text
推奨:
  子プロセス標準入出力型

向いている構成:
  初期実装
  最小構成
  CLI型Engine
  HostがEngine processを起動・終了管理する構成
  単一Host + 単一Engineの構成
```

子プロセス標準入出力型は、Local Process IPC Binding の2つのstreamへ素直に対応できる。

```text
host_to_engine:
  Engine stdin

engine_to_host:
  Engine stdout
```

また、endpoint命名、待受、接続先発見、権限設定などの設計が少なくて済む。

## 5.2 独立プロセス・常駐Engine

Engineを独立プロセスまたは常駐プロセスとして扱う場合は、ローカルendpoint型を推奨する。

```text
推奨:
  ローカルendpoint型

向いている構成:
  独立プロセス型Engine
  常駐Engine
  HostとEngineのライフサイクルを分離したい構成
  明示的なローカル接続口を持ちたい構成
  複数Engine instanceを実装側で管理したい構成
```

この方式では、Hostが必ずしもEngineを子プロセスとして起動する必要はない。

ただし、endpoint作成、命名、発見、待受、接続、権限、timeout、peer終了検出などは実装側が扱う。

## 5.3 共通条件

どちらの方式でも、次を満たさなければならない。

```text
host_to_engine stream を Host -> Engine 方向の順序保証単位として提供する。
engine_to_host stream を Engine -> Host 方向の順序保証単位として提供する。
HEM streamには HEM frame sequence 以外を混ぜない。
transport failure が発生した場合、現在の HEMP session は継続不能として扱う。
同じ HEMP session を再開しない。
再通信する場合は、新しい HEMP session として agreement から開始する。
```

## 5.4 loopback network型の扱い

loopback network型は、主要候補には含めない。

```text
loopback network型は、
network transportを同一コンピュータ内に限定して使う方式である。

この実現ガイドでは、
Local Process IPC Binding の主要な実現候補として扱わない。
```

loopback network型は、必要に応じて、開発・検証・相互接続確認の補助的手段、または別の実装ガイドで扱ってよい。

---
# 6. 実現候補を追加・比較するための整理項目

実現ガイドでは、各実現候補を同じ形式で整理する。

対象は次の2つである。

```text
子プロセス標準入出力型
ローカルendpoint型
```

各候補は、次の項目で整理する。

```text
名称
概要
対象構成
host_to_engine stream の対応
engine_to_host stream の対応
transport確立の責任
ログ / 診断出力の扱い
transport failure の扱い
利点
注意点
ライブラリ実装上の扱い
推奨用途
```

## 6.1 名称

候補の名称である。

```text
子プロセス標準入出力型
ローカルendpoint型
```

## 6.2 概要

その候補がどのような方式かを短く説明する。

## 6.3 対象構成

その方式が向いている構成を示す。

## 6.4 host_to_engine stream の対応

Local Process IPC Binding の `host_to_engine stream` を、その方式でどう提供するかを示す。

`host_to_engine stream` は、Host -> Engine 方向の HEM frame sequence を流す経路であり、Local Process IPC Binding における Host -> Engine 方向の順序保証単位である。

## 6.5 engine_to_host stream の対応

Local Process IPC Binding の `engine_to_host stream` を、その方式でどう提供するかを示す。

`engine_to_host stream` は、Engine -> Host 方向の HEM frame sequence を流す経路であり、Local Process IPC Binding における Engine -> Host 方向の順序保証単位である。

## 6.6 transport確立の責任

通信路を誰が作るか、誰が接続するか、誰が待受するかを整理する。

例:

```text
Engine process の起動
endpoint の作成
endpoint の発見
接続待受
接続開始
pipe / stream の確立
stderrやログ経路の確保
```

ただし、具体的なOS APIやライブラリ名は要求しない。

## 6.7 ログ / 診断出力の扱い

HEM streamへログや診断文字列を混ぜない方法を示す。

```text
HEM stream:
  HEM frame sequence 専用。

ログ / 診断出力:
  別経路を使う。
```

## 6.8 transport failure の扱い

その方式で何が transport failure になるかを示す。

例:

```text
process終了
stream close
I/O error
接続失敗
endpoint作成失敗
timeout
peer終了
```

transport failure が起きた場合は、現在の HEMP session を継続不能として扱う。

## 6.9 利点

その方式の利点を示す。

## 6.10 注意点

その方式の注意点を示す。

## 6.11 ライブラリ実装上の扱い

HEMPライブラリが何を持ち、何を持たないかを整理する。

```text
HEMPライブラリ本体:
  HEM frame / codec / session / validation を扱う。

transport adapter:
  各方式で得た Reader / Writer / Stream を
  host_to_engine / engine_to_host として扱う。
```

HEMPライブラリは、特定のtransport実装集にしない。

## 6.12 推奨用途

その方式をどの用途で推奨するかを示す。

---
# 7. 非対象範囲

この文書は、Local Process IPC Binding の実現候補と整理観点を示す。

この文書は、次を定義しない。

```text
HEMP Core のwire format
HEMP Transport Binding の基本モデル
具体的なOS API
具体的なtransport library
特定の言語
特定のruntime
特定のasync framework
具体的なendpoint命名規則
具体的な権限設定
具体的なtimeout値
retry / reconnect policy
ネットワーク越しTransport Binding
loopback network型を主要候補として扱うこと
in-memory / test transport の標準仕様
```

## 7.1 HEMP Core / Transport Binding仕様の再定義はしない

この文書は、HEMP Core仕様を再定義しない。

この文書は、HEMP Transport Binding Specification を再定義しない。

HEM Length Header、HEM Payload、HEM Header、channel、thread、seq、post / reply / notice、HEM Timeline、agreement、limits、protocol channel Payload length上限、未知fieldの受信規則、failure分類を含む HEMP Core仕様上の契約は、HEMP Core仕様に従う。

Transport Binding基本モデル、Byte Stream Profile、Local Process IPC Binding、およびそれらが従う HEMP Core契約との関係は、HEMP Transport Binding Specification に従う。

## 7.2 特定実装への非依存

この文書は、特定の言語、runtime、async framework、transport library、OS APIを要求しない。

各実装は、この文書で整理した実現方式を、各環境で自然なI/O抽象またはtransport libraryを用いて実装してよい。

HEMPとして重要なのは、最終的に次を提供できることである。

```text
host_to_engine streamを Host -> Engine 方向の順序保証単位として提供すること
engine_to_host streamを Engine -> Host 方向の順序保証単位として提供すること
HEM frame sequenceを壊さないbyte stream
HEM以外を混ぜないこと
transport failure時にsessionを継続しないこと
```

## 7.3 endpoint命名・権限・timeout

ローカルendpoint型における endpoint の命名規則、endpoint情報の受け渡し方法、権限設定、接続可能範囲、timeout値は、この文書では定義しない。

これらは、具体実装、application、または別の実装ガイドが定義する。

## 7.4 retry / reconnect policy

この文書は、retry policy または reconnect policy を定義しない。

transport failure が発生した場合、現在の HEMP session は継続不能として扱う。

再度HEMP通信を行う場合は、新しい HEMP session として agreement から開始する。

## 7.5 ネットワーク越しTransport Binding

この文書は、ネットワーク越しの Host / Engine 通信を定義しない。

ネットワーク越しTransport Bindingは、必要に応じて別のTransport Bindingとして定義する。

## 7.6 loopback network型

loopback network型は、network transportを同一コンピュータ内に限定して使う方式である。

この文書では、loopback network型を Local Process IPC Binding の主要な実現候補として扱わない。

loopback network型は、必要に応じて、開発・検証・相互接続確認の補助的手段、または別の実装ガイドで扱ってよい。

## 7.7 in-memory / test transport

同一プロセス内の in-memory transport や test transport は、Local Process IPC Binding の主要な実現候補として扱わない。

これらは、実装、単体テスト、conformance test などで使用してよい。

ただし、この文書では標準実現方式として定義しない。
