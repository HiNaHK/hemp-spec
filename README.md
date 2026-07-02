# HEMP Specification Documents

この repository は、HEMP 仕様文書を置くためのものです。

この repository は仕様文書を管理するための repository であり、HEMP の実装そのものは含みません。

## 読み順

HEMP の v1.0.0 仕様本文を確認する場合は、まず次の順に読む。

1. `specs/core/`
2. `specs/proto-encoding/`
3. `specs/transport-binding/`

`validation/` 配下の資料は、仕様本文ではありません。必要に応じて、仕様本文の確認後に参照します。

## 主なファイルとディレクトリ

- `specs/`: HEMP 仕様文書の入口です。
- `ROADMAP.md`: HEMP 実装ロードマップです。
- `validation/`: HEMP 実装の検証資料を置く場所です。

## HEMP とは

HEMP / Host-Engine Message Protocol は、Host と Engine の間で message をやり取りするための protocol です。

HEMP では、Host と Engine という2つの役割を区別します。

HEMP の application channel では、channel ごとにどちらが service provider side であるかを定義できます。

service provider side によって、post / reply / notice の向きが決まります。

そのため、Host が常に依頼側で Engine が常に応答側であるとは限りません。

ここでいう message は、Host と Engine の間で送受信される1つの通信単位です。HEMP では、この message を HEM / Host-Engine Message と呼びます。HEM には、処理の依頼、応答、通知、または通信状態を示す情報が含まれます。

典型的には、同一コンピュータ上の別プログラム同士が通信する IPC のような構成で使うことを想定しています。

## HEMP の目的と特徴

HEMP の目的は、Host と Engine の間の通信を曖昧にしないことです。

HEMP は、何を送るのか、どの順番で扱うのか、どこまで送ってよいのか、失敗したときにどう扱うのかを protocol として明確にします。

主な特徴は次のとおりです。

- **Host と Engine の役割を分ける**  
  Host と Engine という2つの役割を区別します。application channel では、service provider side によって post / reply / notice の向きが決まります。

- **HEM を通信の単位として扱う**  
  依頼、応答、通知などを HEM として扱い、Host と Engine の間で送受信します。

- **HEM の流れを明確にする**  
  どの HEM がどのやり取りに属するのか、どの順番で扱うのかを明確にします。

- **session 開始時に前提をそろえる**  
  Host と Engine が同じ前提で動作することを確認してから通信を始めます。そのために、limits や channel などの条件を Agreement としてそろえます。

- **大きな data や段階的な結果を送れる**  
  1つの論理的な送信を複数の HEM に分けて送れます。大きな data や途中経過を順序付きで送ることができ、正常に完了できない場合は `abort` として表します。これは transport の切断とは区別します。

- **失敗したときの扱いを分ける**  
  frame の形が壊れているのか、payload が読めないのか、header の値がおかしいのか、HEM の流れがおかしいのか、body の約束に反しているのかを分けて扱います。

- **HEM の意味と運び方を分ける**  
  HEMP の意味は Core で定義し、実際に transport へどう載せるかは Transport Binding で定義します。

## License

この repository は [Apache License 2.0](LICENSE) の下で公開されます。
