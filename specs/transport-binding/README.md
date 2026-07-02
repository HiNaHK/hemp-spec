# transport-binding

このディレクトリには、HEMP Transport Binding specification に関する仕様文書を置く。

v1.0.0 の Transport Binding specification は、一覧の順に読む。

## v1.0.0 仕様本文

- `01-overview-and-core-relationship.md`: Transport Binding の目的と HEMP Core との関係。
- `02-transport-model.md`: HEM delivery path、ordered delivery unit、HEM frame mapping、Transport Binding instance、HEMP session との関係。
- `03-transport-requirements-and-frame-handling.md`: transport に要求する性質、HEM frame 境界、HEM frame sequence、transport failure、abort と transport failure の境界。
- `04-transport-profiles.md`: HEM frame interleave、Byte Stream Profile、Message Boundary Profile。

## Review draft / candidate documents

- `adapter-architecture/`: HEMP-side Adapter と Transport-side Adapter の共通責任境界に関する review draft / candidate documents。
- `transport-adapters/`: 各 transport 型を実装する場合の、言語非依存ルールに関する review draft / candidate documents。

`adapter-architecture/` と `transport-adapters/` 配下の文書は、現時点では v1.0.0 の確定済み Transport Binding specification 本文としては扱わない。

これらの文書は、HEMP Core semantics、HEM Payload encoding、HEM Length Header、またはこのディレクトリの `01`〜`04` が定義する Transport Binding 共通仕様を置き換えない。

これらの文書は、Python、Rust、TypeScript などの具体 API、class 名、module 構成、OS API、transport library API、または実装コードを定義しない。
