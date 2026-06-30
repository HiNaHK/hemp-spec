# HEMP Implementation Roadmap

Status: Roadmap  
Last updated: 2026-06-30  
Language: Japanese  
Scope: HEMP implementation roadmap

---

## 1. この文書の目的

この文書は、HEMPの実装作業を進めるための大枠を整理するためのroadmapです。

このroadmapは、HEMP仕様本文とは別に、実装作業の到達目標、各milestoneの位置づけ、今後の進め方を整理します。

実装API、Issue／PR運用、CI構成、release automation、各言語ごとのpackage構成などの細部は、該当する実作業に入る段階で必要に応じて扱います。

---

## 2. 前提

### 2.1 実装対象

HEMPの実装対象は、このrepositoryの`specs/`配下に置かれたHEMP仕様本文です。

実装時の正本は`specs/`配下の文書です。

`validation/`配下の資料は、実装確認のための補助資料です。仕様本文ではありません。

仕様本文と補助資料が矛盾する場合は、仕様本文を優先します。

### 2.2 仕様本文の扱い

`specs/`配下の文書は、HEMP protocolを構成する一体の仕様本文として扱います。

実装は、このrepository上のHEMP仕様本文に基づいて行います。

### 2.3 最初の正式実装

最初の正式実装はPython版とします。

Python版は試作品ではなく、HEMPの最初の正式なライブラリ実装として扱います。

Rust版は、Python版で得た実装経験、検証結果、運用上の知見を活用しながら進める後続の正式実装として扱います。

Python版とRust版はいずれも、`specs/`配下のHEMP仕様本文、validation資料、および選定済みの共通IPC実現方式に基づいて検証します。

### 2.4 HEMP IPC／Local Process IPCの共通IPC実現方式

Python版を最初の正式実装にすることと、HEMP IPC／Local Process IPCの実現方式を言語ごとに選ぶことは別である。

HEMP IPC／Local Process IPCの主要な実現方式は、Python版、Rust版、およびその他の正式実装が共通に実装できるものとして扱う。

したがって、HEMP IPC／Local Process IPCに関する実装作業では、言語ごとに主要IPC実現方式を分けない。

Python版とRust版は、同じHEMP仕様本文に従い、選定済みの共通IPC実現方式をそれぞれ実装する。

### 2.5 Milestoneの扱い

このroadmapのmilestoneは、実装作業の到達目標として扱います。

実際の作業では、必要に応じて検証、公開準備、Rust版準備、ドキュメント整理などを並行して進めます。

---

## 3. 現在の状態

Milestone 0は完了済みです。

Milestone 0では、Python版HEMP正式実装を開始するための準備として、次を整備しました。

```text
- HEMP specification repositoryを整備した
- HEMP仕様本文をrepositoryに反映した
- HEMP Protobuf schemaをrepositoryに反映した
- validation資料の位置づけを整理した
- root READMEを整備した
- Apache License 2.0を追加した
```

現在は、Milestone 1のPython版HEMP正式実装へ入る前のPre-Milestone 1段階です。

Pre-Milestone 1では、HEMP Core、HEMP-side Adapter、Transport-side Adapter、Transport Endpoint／Transport／Communication Pathの責任境界を整理し、HEMP IPC／Local Process IPCの共通IPC実現方式を選定します。

Pre-Milestone 1の一部として、Transport Binding配下に次のreview draft／candidate文書を追加済みです。

```text
specs/transport-binding/adapter-architecture/
specs/transport-binding/transport-adapters/
specs/transport-binding/transport-adapters/local-endpoint/
```

これらは、HEMP-side AdapterとTransport-side Adapterの責任境界、およびtransport型ごとの言語非依存ルールを整理するための文書です。

また、旧`specs/transport-binding/local-ipc/specification.md`はparked draft notice／indexになりました。

旧`local-ipc/specification.md`は、現行規範仕様でも、現在のadapter architectureの判断基準でもありません。

Local Process IPCに関する内容を扱う場合は、現行の`adapter-architecture/`と`transport-adapters/`に照らして再評価します。

現在の主な次作業は、HEMP IPC／Local Process IPCとして、正式実装群が共有する主要IPC実現方式を整理し、選定することです。

---

## 4. Milestone一覧

```text
Milestone 0: 実装開始準備（完了）
Pre-Milestone 1: HEMP IPC／Transport Adapter責任境界整理
Milestone 1: Python版HEMP正式実装
Milestone 2: Python版の検証・安定化・公開
Milestone 3: Rust版HEMP正式実装
Milestone 4: Python版／Rust版の整合確認
Milestone 5: HEMPプロジェクト全体の継続運用・整理
Milestone 6: 追加言語対応
```

---

## 5. Milestone詳細

### 5.1 Milestone 0: 実装開始準備

Status: Completed

目的:

```text
Python版HEMP正式実装を開始できる状態を作る。
```

完了済みの内容:

```text
- specification repositoryを整備した
- 仕様本文をrepository上で参照できる状態にした
- Protobuf schemaをrepository上で参照できる状態にした
- validation資料を補助資料として整理した
- root READMEを整備した
- repository licenseをApache License 2.0として明示した
```

完了条件:

```text
Milestone 1のPython版HEMP正式実装に着手できる状態になっている。
```

判定:

```text
完了済み。
```

---

### 5.2 Pre-Milestone 1: HEMP IPC／Transport Adapter責任境界整理

Status: In Progress

目的:

```text
Python版HEMP正式実装のTransport AdapterおよびIPC送受信に関わる実装へ入る前に、
HEMP Core、HEMP-side Adapter、Transport-side Adapter、
Transport Endpoint／Transport／Communication Pathの責任境界を整理し、
HEMP IPC／Local Process IPCの共通IPC実現方式を選定する。
```

含めるもの:

```text
- HEMP Core、HEMP-side Adapter、Transport-side Adapter、Transport Endpoint／Transport／Communication Pathの責任境界
- Core-side framing helperの位置づけ
- HEMP-side AdapterとHEMP Coreの受け渡し単位
- Transport-side Adapterの最小責任
- transport failure、frame construction failure、HEMP failure、abortの境界
- HEMP IPC／Local Process IPCとして共有する主要IPC実現方式の検討
- Python版、Rust版、その他実装が同じIPC実現方式を実装する前提の整理
- in-memory／テスト用harnessと実プロセス間IPC方式の分離
```

含めないもの:

```text
- Python API
- Rust API
- class名
- function名
- module構成
- sync／async API
- 実装コード
- OS APIの具体呼び出し
- endpoint nameの具体形式
- permission／ACLの具体設定
- timeout値
```

整理済みの内容:

```text
- HEMP Core、HEMP-side Adapter、Transport-side Adapter、Transport Endpoint／Transport／Communication Pathの責任境界。
- complete HEM frame構成とHEMP Coreへの受け渡し境界。
- Transport-side Adapterは接続済みまたは確立済みTransport I/Oを包む層であり、endpoint作成やprocess launchを最小責任に含めないこと。
- transport failure、frame construction failure、HEMP failure、abortの境界。
```

残作業:

```text
- HEMP IPC／Local Process IPCとして、正式実装群が共有する主要IPC実現方式を決める。
- この判断は、Python版だけの最初のTransport-side Adapterを選ぶ作業ではない。
- Python版、Rust版、その他実装が同じIPC実現方式を実装し、相互接続できることを前提にする。
```

完了条件:

```text
Milestone 1のうち、Transport AdapterおよびIPC送受信に関わる実装へ入る前提として、
HEMP libraryの責任境界、
Transport Adapterの責任境界、
およびHEMP IPC／Local Process IPCの共通IPC実現方式が整理されている。
```

---

### 5.3 Milestone 1: Python版HEMP正式実装

目的:

```text
Python版を、HEMPの最初の正式なライブラリ実装として作る。
```

対象:

```text
- repository上のHEMP仕様本文に基づく正式実装
- Protobuf schemaに基づくHEM Payload encoding／decoding
- HEMの生成、送信、受信、検証
- sessionとflowの管理
- Transport Binding共通仕様に基づく送受信
- Pre-Milestone 1で選定した共通IPC実現方式に基づく送受信
- failure handling
```

位置づけ:

```text
Python版は試作品ではない。
Python版はHEMPの最初の正式実装である。
```

完了条件:

```text
Python版で、HEMPとして一通り成立する正式実装がある。
```

備考:

```text
仕様照合、安定化、公開準備はMilestone 2で扱う。
```

---

### 5.4 Milestone 2: Python版の検証・安定化・公開

目的:

```text
Milestone 1で作成したPython版HEMP正式実装を検証し、安定化し、Python版ライブラリとして公開可能な状態にする。
```

含めるもの:

```text
- Python版とHEMP仕様本文の照合
- validation資料との照合
- 実装漏れ、曖昧点、仕様解釈ズレの確認
- Python版ライブラリとしての安定化
- README／利用方法の最低限の整理
- Python版の公開準備
- Python版の初回公開
```

完了条件:

```text
Python版HEMPライブラリが、公開可能または公開済みの状態になっている。
```

備考:

```text
Rust版の完成はPython版公開の前提にしない。
```

---

### 5.5 Milestone 3: Rust版HEMP正式実装

目的:

```text
Rust版を、HEMPの後続の正式なライブラリ実装として作る。
```

対象:

```text
- repository上のHEMP仕様本文に基づく正式実装
- Protobuf schemaに基づくHEM Payload encoding／decoding
- HEMの生成、送信、受信、検証
- sessionとflowの管理
- Transport Binding共通仕様に基づく送受信
- Pre-Milestone 1で選定した共通IPC実現方式に基づく送受信
- failure handling
```

位置づけ:

```text
Rust版は、HEMPの後続の正式実装である。
```

完了条件:

```text
Rust版で、HEMPとして一通り成立する正式実装がある。
```

備考:

```text
Python版で得た知見は、Rust版の実装計画、検証項目、相互接続確認に活用してよい。
Rust版は、HEMP仕様本文、validation資料、および選定済みの共通IPC実現方式に基づいて検証する。
Python版／Rust版の整合確認はMilestone 4で扱う。
```

---

### 5.6 Milestone 4: Python版／Rust版の整合確認

目的:

```text
Python版とRust版が、同じHEMP仕様本文に適合していることを確認する。
```

含めるもの:

```text
- Python版とRust版の仕様適合性確認
- validation結果の比較
- 同じ入力に対する仕様上の判断の一致確認
- 言語差による挙動差の整理
- 選定済みIPC実現方式上での相互接続確認
- Python Host／Rust Engine、Rust Host／Python Engineを含む言語間接続確認
- 必要に応じたPython Host／Python Engine、Rust Host／Rust Engineの同一言語接続確認
- 仕様、test、実装、IPC実現方式の不整合があれば分類する
```

完了条件:

```text
Python版とRust版の双方について、
仕様適合性が確認され、
validation上の矛盾がなく、
選定済みIPC実現方式上で相互接続できる状態になっている。
```

備考:

```text
Python版とRust版の挙動差が見つかった場合は、
仕様解釈差、実装差、validation差、IPC実現方式上の差分として分類する。

分類結果に応じて、実装、test、または仕様本文を確認・修正する。
```

---

### 5.7 Milestone 5: HEMPプロジェクト全体の継続運用・整理

目的:

```text
HEMPを、複数の仕様文書と複数の実装を持つprotocol projectとして、継続的に保守・公開・拡張できる状態に整える。
```

含めるもの:

```text
- HEMP仕様本文と各実装の関係整理
- Python版／Rust版のrelease方針整理
- version／tag／release運用の整理
- validation資料の扱い整理
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
Milestone 5は、Python版の初回公開を待たせるものではない。
Python版はMilestone 2完了時点で公開可能である。
```

---

### 5.8 Milestone 6: 追加言語対応

目的:

```text
Python版・Rust版の整備後、必要に応じてHEMPの対応言語を広げる。
```

含めるもの:

```text
- Python／Rust以外の言語対応の検討
- 対応候補言語の選定
- 追加言語実装の優先順位決定
- 公式対応／experimental対応などの扱い整理
- validation資料を使った追加言語実装の確認方針
- 複数言語実装を持つprojectとしての運用拡張
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

## 6. 実装上の整理

### 6.1 HEMP仕様本文の扱い

HEMP実装では、`specs/`配下の仕様本文を一体の仕様セットとして扱います。

Coreは、data body rulesを含むHEMP Core仕様本文として扱います。

Protobuf EncodingとTransport Binding共通仕様も、HEMPを構成する仕様本文として扱います。

`specs/transport-binding/local-ipc/`配下の文書は、現時点ではv1.0.0の確定済みTransport Binding specification本文ではありません。

HEMP IPC／Local Process IPCに関する実装作業では、Transport Binding共通仕様、`adapter-architecture/`、および`transport-adapters/`の責任境界に照らして、共通IPC実現方式を整理します。

### 6.2 複数実装の検証方針

Python版とRust版は、同じHEMP仕様本文、validation資料、および選定済みの共通IPC実現方式に基づいて検証します。

両実装の挙動が分かれた場合は、どちらか一方を自動的な基準にせず、差分の原因を仕様解釈、実装、validation、IPC実現方式に分けて確認します。

確認結果に応じて、実装、test、または仕様本文を修正します。

### 6.3 HEMP IPC／Local Process IPCの共通IPC実現方式

HEMP IPC／Local Process IPCの主要IPC実現方式は、言語ごとに別々に選びません。

正式実装群は、選定済みの共通IPC実現方式を実装し、相互接続確認を行います。

主要IPC実現方式の選定は、単一言語での初期実装容易性ではなく、正式実装群が共有できることを基準に行います。

### 6.4 テスト用harnessと実IPC方式の分離

in-memory／テスト用harnessは、Core、framing、HEMP-side Adapter、failure boundaryなどの検証に使ってよいです。

ただし、in-memory／テスト用harnessは、実プロセス間IPCの主要方式としては扱いません。

テスト用harnessを、Python版／Rust版間の実IPC相互接続方式として扱ってはなりません。

---

## 7. まとめ

Milestone 0は完了済みです。

現在は、Milestone 1のPython版HEMP正式実装へ入る前のPre-Milestone 1段階です。

HEMPの実装対象は、このrepositoryの`specs/`配下に置かれたHEMP仕様本文です。

最初の正式実装はPython版とし、Rust版は後続の正式実装として進めます。

Python版とRust版はいずれも、HEMP仕様本文、validation資料、および選定済みの共通IPC実現方式に基づいて検証します。

具体的な運用ルールは、実作業段階で必要に応じて決定します。

Milestoneは、実装作業の到達目標として扱います。
