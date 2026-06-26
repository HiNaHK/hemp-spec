# proto-encoding

このディレクトリには、HEMP Protobuf Encoding specification に関する仕様文書と schema を置く。

文書は、一覧の順に読む。  
schema は、必要に応じて参照する。

## 文書

- `01-overview-and-schema-relationship.md`: Protobuf Encoding の位置づけと `.proto` schema との関係。
- `02-framing-and-payload-encoding.md`: HEM framing と HEM Payload の Protobuf encoding。
- `03-message-model-and-body-mapping.md`: HemPayload / HemHeader / body mapping。
- `04-protocol-bodies-limits-and-capacity.md`: protocol bodies / limits / capacity。
- `05-validation-compatibility-and-evolution.md`: validation / compatibility / schema evolution policy for v1.0.0.
- `schema/hemp.proto`: HEMP Protobuf schema。
