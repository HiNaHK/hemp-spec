# local-endpoint

このディレクトリには、local endpoint 型 Transport-side Adapter を実装する場合の、言語非依存ルールを置く。

**local endpoint 型** は、同一コンピュータ内の process 間で接続するための、名前付きまたは識別可能なローカル接続口を前提にする transport 型である。

local endpoint 型は、そのローカル接続口から得られた Transport I/O を扱う。

local endpoint 型は、具体的な OS API、endpoint mechanism、または library API を意味しない。

このディレクトリの文書は、具体的な OS API、endpoint name、permission / ACL、timeout 値、Python API、Rust API、または実装コードを定義しない。

全 Transport-side Adapter に共通する責任境界は、`../../adapter-architecture/` 配下で扱う。

## 文書

- `01-overview-and-scope.md`: local endpoint 型 Transport-side Adapter の範囲、責任境界、HEM delivery path との関係、transport failure の扱い。

## このディレクトリで扱わないこと

このディレクトリでは、次を扱わない。

```text
Unix domain socket を使うかどうか
Windows named pipe を使うかどうか
endpoint name の形式
endpoint discovery の具体方式
permission / ACL の具体設定
peer confirmation の具体方式
timeout 方針
timeout 値
Python socket API
Rust library
async 実装
実装コード
```

これらは、実装 repository、application、transport setup、外部 transport library、または OS API 側で扱う。
