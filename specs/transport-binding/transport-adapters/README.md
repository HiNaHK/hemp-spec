# transport-adapters

このディレクトリには、Transport-side Adapter の対象となる transport 型ごとの、言語非依存ルールを置く。

このディレクトリでは、**transport 型** という語を、Transport-side Adapter が対象とする transport-specific resource や通信方式の分類を指す説明語として使う。

transport 型は、HEMP Core semantics を変更する概念ではない。

transport 型は、Transport Binding Profile ではない。

このディレクトリの文書は、Python、Rust、TypeScript などの具体 API、class 名、module 構成、OS API、transport library API、または実装コードを定義しない。

全 Transport-side Adapter に共通する責任境界は、`../adapter-architecture/` 配下で扱う。

各 transport 型の文書は、`../adapter-architecture/` の共通責任境界に従う。

## 文書

- `local-endpoint/`: local endpoint 型 Transport-side Adapter を実装する場合の言語非依存ルール。
- `subprocess-stdio/`: subprocess stdio型 Transport-side Adapter を実装する場合の言語非依存ルール。

## 現時点で作らない文書

現時点では、次の transport 型の文書は作らない。

```text
tcp/
quic/
```

これらは、対象 transport 型として扱う必要が出た時点で追加する。

空ディレクトリや空文書は置かない。

## このディレクトリで扱うこと

このディレクトリでは、各 transport 型について、次を扱う。

```text
HEM delivery path との対応
ordered delivery unit の満たし方
HEM frame sequence の扱い
complete HEM frame の扱い
transport failure の扱い
transport-specific resource と HEMP Core semantics の分離
```

## このディレクトリで扱わないこと

このディレクトリでは、次を扱わない。

```text
Python API
Rust API
class 名
function 名
module 構成
sync / async API
OS API の具体的な呼び出し方
transport library API
endpoint name の形式
permission / ACL の具体設定
timeout 値
process launch の具体方法
実装コード
```

これらは、hemp-spec ではなく実装 repository 側で扱う。
