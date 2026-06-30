# local-ipc

このディレクトリには、Local Process IPC Binding に関する未確定または将来整理対象の文書を置く。

このディレクトリ配下の文書は、現時点では v1.0.0 の確定済み Transport Binding specification 本文としては扱わない。

このディレクトリ配下の文書は、現在の adapter architecture の判断基準として扱わない。

## 文書

- `specification.md`: 旧 Local Process IPC Binding draft material の parked draft notice / index。

`specification.md` は、Local Process IPC Binding の現行規範仕様ではない。

`specification.md` は、旧 draft 本文を現行の責任境界と用語に照らして再評価するための notice / index である。

## 今後の扱い

今後 Local Process IPC に関する内容を扱う場合は、次の文書群に照らして再評価する。

```text
../adapter-architecture/
../transport-adapters/
```

特に local endpoint 型に関する内容は、次に照らして再評価する。

```text
../transport-adapters/local-endpoint/
```

旧 draft の内容を機械的に移植してはならない。

必要な内容だけを、現行の責任境界、採用済み用語、および transport 型ごとの文書構造に照らして再評価する。
