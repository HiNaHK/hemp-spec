# HEMP 実装ロードマップ: Core v9 / Standard Data Body v1 / Transport Binding v4.1

Status: Roadmap Draft  
Date: 2026-06-18  
Language: Japanese  
Scope: HEMP independent implementation roadmap

---

## 1. この文書の目的

この文書は、HEMP の実装ロードマップを整理するためのものである。

対象は、仕様レビューやメタデータ固定ではなく、HEMP を独立したプロトコルライブラリプロジェクトとして実装していくための大枠である。

この文書では、実装設計の細部、GitHub / Codex の具体運用、API設計、Issue分割、PRルールなどは決めない。これらは実際にその作業を行う段階で壁打ちし、必要に応じて決定する。

---

## 2. 前提

### 2.1 HEMPは独立プロジェクトである

HEMP は MST プロジェクトから独立したプロジェクトとして扱う。

MST は HEMP 作成のきっかけや将来的な利用先になり得るが、この実装ロードマップの対象には含めない。

### 2.2 実装対象

HEMP の実装対象は、次を一体として扱う。

```text
- HEMP Core v9
- HEMP Core v9 Protobuf Encoding
- HEMP Core v9 .proto schema
- Standard Data Body v1
- Transport Binding v4.1
```

Core v9、Standard Data Body v1、Transport Binding v4.1 は、文書が分かれているだけであり、HEMP実装上は別物・任意追加として扱わない。

Transport Binding v4.1 は、HEMP frame を実際に運ぶための構成要素として扱う。

### 2.3 正本と補助成果物

実装時の正本は、有効仕様セットである。

```text
HEMP_effective_spec_set_core_v9_2026-06-18.zip
```

validation artifacts と work records は非規範であり、仕様本文と矛盾する場合は有効仕様セットを優先する。

```text
HEMP_core_v9_validation_artifacts_2026-06-18.zip
HEMP_core_v9_work_records_2026-06-18.zip
```

---

## 3. 基本方針

### 3.1 完全版を目標にする

HEMP実装は、部分的な検証用実装や試作品ではなく、完全版の正式ライブラリ実装を目標にする。

ただし、個人プロジェクトとして進めるため、実装開始前に必須ではない形式作業は可能な範囲で省略する。

省略してよいものの例:

```text
- Review Candidate 表記固定
- release note 作成
- tag 名整理
- 追加 work records 更新
- 追加通読レビュー
- 過度に詳細な運用ルールの先行決定
```

### 3.2 仕様駆動開発で進める

HEMP実装は、仕様駆動開発を基本方針とする。

つまり、実装は有効仕様セットから導かれるものであり、実装都合で HEMP の意味論を変更しない。

ただし、次のような具体運用は、現時点では決めない。

```text
- Issueテンプレート
- AGENTS.md の具体内容
- Requirement ID の命名規則
- PRレビュー手順
- CI構成
- Codexへの具体的な依頼文
```

これらは、GitHub repository 作成や実装開始の段階で必要に応じて決定する。

### 3.3 Python版を最初の正式実装にする

最初の実装言語は Python とする。

Python版は、HEMP完全実装の最初の正式実装として扱う。

### 3.4 Rust版は後続の正式実装にする

Rust版は、Python版の後に進める正式実装として扱う。

Python版とRust版の関係は、次のように整理する。

```text
Python版:
  HEMP完全実装の最初の正式実装

Rust版:
  後続の正式実装

両者の関係:
  Python版が本実装でRust版が移植版、という関係ではない
  Python版がreferenceでRust版がproduction、という関係でもない
  どちらもHEMPの正式なライブラリ実装である
```

Python版で得た知見はRust版に活用してよい。  
ただし、Rust版の正しさの基準は Python版ではなく、有効仕様セットである。

### 3.5 Milestoneは厳密な直列工程ではない

このロードマップの Milestone は、厳密な時系列工程ではなく、到達目標である。

実際の作業では、検証、公開準備、Rust版準備、ドキュメント整理などを必要に応じて並行して進める。

どの作業をどのタイミングで並行するかは、実際の進め方に合わせて判断する。

この文書では、Track のような細かい並行作業区分は定義しない。

---

## 4. Milestone一覧

```text
Milestone 0: 実装開始準備
Milestone 1: Python版 HEMP 正式実装
Milestone 2: Python版 検証・安定化・公開
Milestone 3: Rust版 HEMP 正式実装
Milestone 4: Python版 / Rust版 整合確認
Milestone 5: HEMPプロジェクト全体の継続運用・整理
Milestone 6: 追加言語対応
```

---

## 5. Milestone詳細

### 5.1 Milestone 0: 実装開始準備

目的:

```text
Python版 HEMP 正式実装を開始できる状態を作る。
```

含めるもの:

```text
- GitHub repository を用意する
- 有効仕様セットを参照できる状態にする
- Codex を使える前提を整える
- Python実装を開始できる最低限の開発環境を用意する
- 仕様駆動開発で進めるための最低限の整理をする
```

含めすぎないもの:

```text
- GitHub運用細部
- Codex指示文
- Issue / PR ルール
- API設計
- CI詳細
- 実装内部設計
```

完了条件:

```text
Milestone 1 の Python版 HEMP 正式実装に着手できる状態になっている。
```

---

### 5.2 Milestone 1: Python版 HEMP 正式実装

目的:

```text
Python版を、HEMP完全実装の最初の正式実装として作る。
```

対象:

```text
- Core v9
- Protobuf Encoding
- .proto schema
- Standard Data Body v1
- Transport Binding v4.1
```

位置づけ:

```text
Python版は試作品ではない。
Python版は HEMP完全実装の最初の正式実装である。
```

完了条件:

```text
Python版で、HEMPとして一通り成立する正式実装がある。
```

備考:

```text
仕様照合・安定化・公開は Milestone 2 で扱う。
```

---

### 5.3 Milestone 2: Python版 検証・安定化・公開

目的:

```text
Milestone 1で作成した Python版 HEMP 正式実装を検証し、安定化し、Python版ライブラリとして公開可能な状態にする。
```

含めるもの:

```text
- Python版と有効仕様セットの照合
- conformance vectors / validation artifacts との照合
- 実装漏れ・曖昧点・仕様解釈ズレの確認
- Python版ライブラリとしての安定化
- README / 利用方法の最低限の整理
- Python版の公開準備
- Python版の初回公開
```

完了条件:

```text
Python版 HEMP ライブラリが、公開可能または公開済みの状態になっている。
```

備考:

```text
Rust版の完成は Python版公開の前提にしない。
```

---

### 5.4 Milestone 3: Rust版 HEMP 正式実装

目的:

```text
Rust版を、HEMPの後続の正式ライブラリ実装として作る。
```

対象:

```text
- Core v9
- Protobuf Encoding
- .proto schema
- Standard Data Body v1
- Transport Binding v4.1
```

位置づけ:

```text
Rust版はPython版の移植版ではない。
Rust版もHEMPの正式実装である。
```

完了条件:

```text
Rust版で、HEMPとして一通り成立する正式実装がある。
```

備考:

```text
Python版で得た知見は活用する。
ただし、Rust版の正しさの基準は有効仕様セットとする。
Python版 / Rust版の整合確認は Milestone 4 で扱う。
```

---

### 5.5 Milestone 4: Python版 / Rust版 整合確認

目的:

```text
Python版とRust版が、同じHEMP仕様に適合していることを確認する。
```

含めるもの:

```text
- Python版とRust版の仕様適合性確認
- conformance結果の比較
- 同じ入力に対する仕様上の判断の一致確認
- 言語差による挙動差の整理
- 仕様・テスト・実装の不整合があれば分類する
```

完了条件:

```text
Python版とRust版の双方について、仕様適合性が確認され、conformance上の矛盾がない状態になっている。
```

備考:

```text
Python版はRust版の正しさの基準ではない。
基準は常に有効仕様セットとする。
```

---

### 5.6 Milestone 5: HEMPプロジェクト全体の継続運用・整理

目的:

```text
HEMPを、Python版・Rust版を含む独立プロジェクトとして、継続的に保守・公開・拡張できる状態に整える。
```

含めるもの:

```text
- HEMP独立プロジェクトとしての README / docs 整理
- 仕様と各実装の関係整理
- Python版 / Rust版のリリース方針整理
- version / tag / release 運用の整理
- conformance assets の扱い整理
- 仕様更新時に各実装がどう追従するかの整理
- 追加言語対応へ進むための前提整理
```

含めないもの:

```text
- 追加言語実装そのもの
```

完了条件:

```text
HEMPプロジェクト全体として、継続運用と将来拡張の前提が整っている。
```

備考:

```text
Milestone 5 は、Python版の初回公開を待たせるものではない。
Python版は Milestone 2 完了時点で公開可能である。
```

---

### 5.7 Milestone 6: 追加言語対応

目的:

```text
Python版・Rust版の整備後、必要に応じてHEMPの対応言語を広げる。
```

含めるもの:

```text
- Python / Rust 以外の言語対応の検討
- 対応候補言語の選定
- 追加言語実装の優先順位決定
- 公式対応 / experimental 対応などの扱い整理
- conformance assets を使った追加言語実装の確認方針
- 複数言語実装を持つプロジェクトとしての運用拡張
```

含めすぎないもの:

```text
- 現時点で対応言語を確定すること
- 全言語対応を前提にすること
- Python版やRust版の完成前に追加言語実装へ広げすぎること
- 各言語の具体API設計を今決めること
```

完了条件:

```text
追加言語対応を進められる方針と前提が整っている。
```

---

## 6. 後で壁打ちする項目

次の項目は重要だが、このロードマップ文書では決定しない。

これらは、該当する実作業に入る段階で壁打ちし、必要に応じて決定する。

```text
- Python版の公開API設計
- failure / error object の具体設計
- session / flow validation の責務境界
- Transport Binding の具体的な実装方式
- Standard Data Body v1 の送受信API
- generated protobuf code の管理方針
- conformance test の具体的な接続方法
- GitHub Issue / PR 運用
- AGENTS.md の具体内容
- Codexへの具体的な依頼文
- CI / release automation
- Rust crate構成
- 追加言語の優先順位
```

---

## 7. このロードマップで採用しない整理

### 7.1 HEMP構成要素を別物として扱う整理は採用しない

次のような整理は採用しない。

```text
- Core v9 だけをHEMP本体として扱う
- Standard Data Body v1 をHEMPとは別物として扱う
- Transport Binding v4.1 を任意の後付けとして扱う
```

HEMP実装では、Core v9、Protobuf Encoding、.proto schema、Standard Data Body v1、Transport Binding v4.1 を一体として扱う。

### 7.2 Python版とRust版に上下関係を置かない

次のような整理は採用しない。

```text
- Python版が本実装でRust版が移植版
- Python版がreferenceでRust版がproduction
- Rust版の正しさをPython版との一致で判断する
```

Python版とRust版はいずれも正式実装であり、正しさの基準は有効仕様セットである。

### 7.3 Track構成は採用しない

このロードマップでは、Milestoneとは別に Track を定義しない。

作業の並行方法は、実際の進め方に合わせて都度判断する。

---

## 8. まとめ

HEMPは独立プロジェクトとして実装する。

対象は Core v9、Protobuf Encoding、.proto schema、Standard Data Body v1、Transport Binding v4.1 を一体とした HEMP完全実装である。

最初の正式実装は Python版とし、Rust版は後続の正式実装として進める。

Python版とRust版はいずれも正式実装であり、上下関係や本実装 / 移植版の関係にはしない。

開発は仕様駆動を基本方針とし、Codex / GitHub を利用する。ただし、具体的な運用ルールは実作業段階で決定する。

Milestoneは到達目標であり、厳密な直列工程ではない。並行作業の進め方は、その時点の状況に合わせて判断する。
