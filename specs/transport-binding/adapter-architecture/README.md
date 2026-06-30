# adapter-architecture

このディレクトリには、HEMP Transport Binding における adapter architecture の共通責任境界を置く。

このディレクトリでは、**adapter architecture** という語を、HEMP-side Adapter と Transport-side Adapter の責任境界、および HEMP Core と Transport I/O の間の受け渡しを整理する文書領域として使う。

このディレクトリの文書は、特定 transport 型の実装方法を定義しない。

このディレクトリの文書は、特定のプログラム言語、runtime、OS API、transport library、endpoint 実装、または実装コードを定義しない。

各 transport 型に固有の言語非依存ルールは、`../transport-adapters/` 配下で扱う。

## 文書

文書は、次の順に読む。

```text
01-common-responsibilities.md
02-failure-and-session-boundaries.md
```

`01-common-responsibilities.md` は、通常時の層構造と責任境界を扱う。

`02-failure-and-session-boundaries.md` は、失敗情報と HEMP session 継続可否の境界を扱う。

## このディレクトリで扱うこと

このディレクトリでは、全 Transport-side Adapter に共通する責任境界を扱う。

このディレクトリでは、HEMP Core、HEMP-side Adapter、Transport-side Adapter、Transport Endpoint / Transport / Communication Path の関係を扱う。

このディレクトリでは、transport failure、frame construction failure、HEMP Core の処理結果、HEMP session の継続可否を混ぜないための共通ルールを扱う。

## このディレクトリで扱わないこと

このディレクトリでは、各 transport 型に固有の言語非依存ルールを定義しない。

このディレクトリでは、次を定義しない。

```text
Python API
Rust API
class 名
function 名
module 構成
sync / async API
OS API の具体的な呼び出し方
library API
endpoint naming
permission / ACL の具体設定
timeout 値
実装コード
```

## transport-adapters/ との関係

`adapter-architecture/` は、全 Transport-side Adapter に共通する責任境界を扱う。

`transport-adapters/` は、各 transport 型を実装する場合の言語非依存ルールを扱う。

各 transport 型の文書は、このディレクトリで定義する共通責任境界に従う。

## 仕様本文との関係

このディレクトリの文書は、HEMP Core semantics を再定義しない。

このディレクトリの文書は、HEM Payload encoding を再定義しない。

このディレクトリの文書は、Transport Binding Profile、Byte Stream Profile、Message Boundary Profile、HEM Length Header、HEM Payload、または HEM Header の意味を変更しない。

このディレクトリの文書と、`specs/core/`、`specs/proto-encoding/`、または `specs/transport-binding/01`〜`04` の仕様本文が矛盾する場合は、対応する仕様本文を優先する。
