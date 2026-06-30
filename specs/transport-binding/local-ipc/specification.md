# HEMP Local Process IPC Binding Draft Notice

Status: Parked Draft Notice
Specification version: Not assigned
Scope: Parked notice / index for previous Local Process IPC Binding draft material

この文書は、以前の長い Local Process IPC Binding draft material を置き換える parked draft notice / index である。

この文書は、HEMP v1.0.0 の確定済み Transport Binding specification 本文ではない。

この文書を、Local Process IPC Binding の現行規範仕様として扱ってはならない。

この文書を、現在の adapter architecture の判断基準として扱ってはならない。

## 1. Purpose

この文書の目的は、旧 Local Process IPC Binding draft material が現行の判断基準ではないことを明示することである。

旧 draft material は、`adapter-architecture/` と `transport-adapters/` の文書構造より前に作成された混合 draft であった。

旧 draft material は、Local Process IPC Binding 候補、process / stdio / endpoint まわりの実現候補、transport failure、session continuation、および実装寄りの整理を一つの文書に含んでいた。

そのため、旧 draft material をそのまま読むと、現在の責任境界や採用済み用語より強い根拠のように誤読される可能性がある。

## 2. Current Reference Documents

今後 Local Process IPC に関する内容を扱う場合は、まず次に照らして再評価する。

```text
../adapter-architecture/
../transport-adapters/
```

特に local endpoint 型に関する内容は、次に照らして再評価する。

```text
../transport-adapters/local-endpoint/
```

これらの文書は、HEMP-side Adapter と Transport-side Adapter の責任境界、および transport 型ごとの言語非依存ルールを扱う。

## 3. Reuse Policy for Previous Draft Content

旧 draft material の内容を、機械的に移植してはならない。

旧 draft material の内容を、現行規範仕様として扱ってはならない。

旧 draft material の内容を、現在の adapter architecture の判断基準として扱ってはならない。

必要な内容がある場合だけ、次に照らして再評価する。

```text
HEM delivery path
ordered delivery unit
Transport Binding Profile
Transport Binding instance
transport-specific resource
HEMP-side Adapter
Transport-side Adapter
Transport I/O
transport failure
frame construction failure
HEMP session continuation
```

再評価した結果、必要な内容だけを、対応する新しい文書構造へ移す。

## 4. Out of Scope

この notice / index は、Local Process IPC Binding を正式な concrete Transport Binding として定義しない。

この notice / index は、subprocess stdio 型を定義しない。

この notice / index は、local endpoint 型を再定義しない。

この notice / index は、TCP 型、QUIC 型、loopback network 型、または in-memory / test transport 型を定義しない。

この notice / index は、Python API、Rust API、class 名、function 名、module 構成、OS API、transport library API、endpoint naming、permission / ACL、timeout 値、または実装コードを定義しない。

## 5. Future Work

Local Process IPC に関する内容が今後必要になった場合は、旧 draft material を前提にせず、現行の責任境界に基づいて新しく判断する。

必要に応じて、対象 transport 型ごとの言語非依存ルールを `../transport-adapters/` 配下に追加する。

Local Process IPC Binding を正式な concrete Transport Binding として扱う必要がある場合は、Transport Binding 共通仕様、`../adapter-architecture/`、および `../transport-adapters/` との関係を確認したうえで、別途設計する。
