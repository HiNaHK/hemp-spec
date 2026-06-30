# Common Adapter Responsibilities

Status: Review Draft / Candidate Transport Adapter Architecture Rules
Specification version: Not assigned
Scope: HEMP Transport Binding adapter architecture
Draft revision: v0.4

## 1. Purpose

この文書は、HEMP Core、HEMP-side Adapter、Transport-side Adapter、Transport Endpoint / Transport / Communication Path の責任境界を定義する。

この文書は、特定 transport 型の実装方法を定義しない。

この文書は、特定のプログラム言語、runtime、OS API、transport library、endpoint 実装、または実装コードを定義しない。

## 2. Position in HEMP Transport Binding

この文書は、HEMP Transport Binding における adapter architecture を扱う。

この文書では、**adapter architecture** という語を、HEMP-side Adapter と Transport-side Adapter の責任境界、および HEMP Core と Transport I/O の間の受け渡しを整理する文書領域として使う。

この文書は、Transport Binding 01〜04 の共通仕様を置き換えない。

この文書は、HEMP Core semantics を再定義しない。

この文書は、HEM frame sequence を transport 上で扱う際の HEMP-side Adapter と Transport-side Adapter の責任境界を定義する。

## 3. Layer Overview

HEMP library の transport adapter architecture は、概念上次の層で整理する。

```text
Application / Integration
  ↑↓
HEMP Core
  ↑↓
HEMP-side Adapter
  ↑↓
Transport-side Adapter
  ↑↓
Transport Endpoint / Transport / Communication Path
```

`Application / Integration` は、HEMP library の内部構成名ではない。

`Application / Integration` は、HEMP library の外側で、transport setup や接続対象の選択などを扱い得る領域を示すための説明である。

`Transport Endpoint / Transport / Communication Path` は、HEMP Core semantics の外側にある。

Core-side framing helper は、この図の中の独立した通信層ではない。

## 4. HEMP Core Responsibilities

HEMP Core は、HEMP Core semantics を扱う。

HEMP Core は、HEM Payload decode、HEM Header validation、direction validation、flow validation、failure classification、agreement、limits、および session state を扱う。

HEMP Core は、Transport I/O を直接扱わない。

HEMP Core は、endpoint、socket、pipe、stream object、file descriptor、transport library object、runtime object、または transport-specific resource を直接扱わない。

HEMP Core は、transport failure を HEMP failure classification として解釈しない。

## 5. Core-side Framing Helper

**Core-side framing helper** は、HEM Length Header と HEM frame format の純粋処理を提供する helper である。

Core-side framing helper は、HEM Payload bytes から complete HEM frame bytes を構成するための純粋処理を提供する。

Core-side framing helper は、complete HEM frame bytes から HEM Payload bytes を取り出すための純粋処理を提供する。

Core-side framing helper は、Transport I/O から読む層ではない。

Core-side framing helper は、Transport I/O へ書く層ではない。

Core-side framing helper は、独立した transport 層ではない。

## 6. HEMP-side Adapter Responsibilities

**HEMP-side Adapter** は、Transport-side Adapter から受け取った data を HEMP Core へ渡す前に、complete HEM frame の構成を扱う transport 共通層である。

HEMP-side Adapter は、特定の transport、runtime、library、または実現方式に依存しない責任を扱う。

HEMP-side Adapter は、Transport-side Adapter から受け取った bytes または transport message payload を使い、Transport Binding Profile の規則に従って complete HEM frame を構成する。

HEMP-side Adapter は、complete HEM frame を構成できた場合だけ、HEM Payload bytes と received HEM delivery path を HEMP Core へ渡す。

HEMP-side Adapter は、complete HEM frame を構成できない data を HEMP Core validation へ渡さない。

HEMP-side Adapter は、HEM Payload bytes を decode して HEMP Core へ渡さない。

Payload decode、HEM Header validation、direction validation、flow validation、failure classification、session state 更新は、HEMP Core 側の責任である。

HEMP-side Adapter は、transport failure を `abort = true` や HEMP failure response に変換しない。

## 7. Transport-side Adapter Responsibilities

**Transport-side Adapter** は、接続済みまたは確立済みの Transport I/O を HEMP-side Adapter へ接続する transport 依存層である。

Transport-side Adapter は、transport、runtime、library、または実現方式の差を吸収する。

Transport-side Adapter は、接続済みまたは確立済みの Transport I/O を包む。

Transport-side Adapter は、HEMP-side Adapter に対して、HEM delivery path-aware な読み書きと transport failure reporting を提供する。

ここでいう読み書きは、具体的な API 名ではない。

ここでいう読み書きは、Transport-side Adapter が提供する責任を説明するための一般表現である。

Transport-side Adapter は、Transport I/O を扱う。

Transport-side Adapter は、HEMP Core semantics を解釈しない。

Transport-side Adapter は、HEM Payload を decode しない。

Transport-side Adapter は、HEM Header validation、direction validation、flow validation、failure classification、session state 更新を扱わない。

## 8. Transport Endpoint / Transport / Communication Path

Transport Endpoint / Transport / Communication Path は、HEMP Core semantics の外側にある。

Transport Endpoint / Transport / Communication Path は、Transport-side Adapter が包む接続済みまたは確立済みの Transport I/O を提供し得る。

Transport Endpoint / Transport / Communication Path は、次を扱い得る。

```text
endpoint creation
endpoint discovery
connection establishment
permission / ACL
authentication
process launch
transport setup
OS API / external transport library の利用
```

これらは、HEMP Core / HEMP-side Adapter の責任ではない。

これらは、Transport-side Adapter の最小責任にも含めない。

## 9. HEM Delivery Path and Transport-specific Resource

HEM delivery path は、Transport Binding instance 内で、HEM frame sequence を一方向に配送するための抽象的な delivery path である。

HEM delivery path は、次と同義ではない。

```text
process
pipe
socket
endpoint
file descriptor
stream object
transport library object
runtime object
transport-specific resource
```

これらは、HEM delivery path を実現するために使用してよい transport-specific resource である。

Transport-side Adapter は、transport-specific resource を HEMP Core semantics として解釈してはならない。

## 10. Layer Handoff Units

HEMP-side Adapter から HEMP Core への受信側の受け渡し単位は次である。

```text
HEM Payload bytes
received HEM delivery path
```

HEMP Core から HEMP-side Adapter への送信側の受け渡し単位は次である。

```text
HEM Payload bytes
send HEM delivery path
```

Transport-side Adapter から HEMP-side Adapter への受け渡しでは、次を表現できなければならない。

```text
bytes または transport message payload
HEM delivery path
transport failure reporting
```

HEMP-side Adapter から Transport-side Adapter への受け渡しでは、次を表現できなければならない。

```text
complete HEM frame bytes
send HEM delivery path
```

この文書は、これらを具体的な class 名、function 名、method 名、callback 名、または result object 名として定義しない。

## 11. Responsibilities Not Included in the Minimal Transport-side Adapter Contract

Transport-side Adapter の最小責任には、次を含めない。

```text
endpoint creation
endpoint discovery
connection establishment
permission / ACL
authentication
process launch
retry
reconnect
session restart
```

これらは、必要に応じて、application、transport setup、外部 transport library、OS API、実装 repository、または Transport-side Adapter 周辺の統合層で扱う。

ただし、この文書は、これらを Transport-side Adapter の最小責任として要求しない。

## 12. Relationship to transport-adapters/

この文書は、全 Transport-side Adapter に共通する責任境界を定義する。

各 transport 型の言語非依存ルールは、`../transport-adapters/` 配下で扱う。

各 transport 型の文書は、この文書で定義する共通責任境界に従う。
