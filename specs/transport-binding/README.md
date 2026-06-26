# transport-binding

このディレクトリには、HEMP Transport Binding specification に関する仕様文書を置く。

v1.0.0 の Transport Binding specification は、一覧の順に読む。

## 文書

- `01-overview-and-core-relationship.md`: Transport Binding の目的と HEMP Core との関係。
- `02-transport-model.md`: 実通信方向、順序保証単位、HEM frame mapping、Transport Binding instance、HEMP session との関係。
- `03-transport-requirements-and-frame-handling.md`: transport に要求する性質、HEM frame 境界、HEM frame sequence、transport failure、abort と transport failure の境界。
- `04-transport-profiles.md`: HEM frame interleave、Byte Stream Profile、Message Boundary Profile。

## 未確定 / future binding 文書

- `local-ipc/`: Local Process IPC Binding に関する未確定または将来整理対象の文書。

`local-ipc/` 配下の文書は、現時点では v1.0.0 の確定済み Transport Binding specification 本文としては扱わない。
