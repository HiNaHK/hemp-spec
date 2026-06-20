# HEMP Implementation Roadmap

Status: Roadmap  
Last updated: 2026-06-20  
Language: Japanese  
Scope: HEMP implementation roadmap

---

## 1. この文書の目的

この文書は、HEMP の実装作業を進めるための大枠を整理するための roadmap です。

この roadmap は、HEMP 仕様本文とは別に、実装作業の到達目標、各 milestone の位置づけ、今後の進め方を整理します。

実装 API、Issue / PR 運用、CI 構成、release automation、各言語ごとの package 構成などの細部は、該当する実作業に入る段階で必要に応じて扱います。

---

## 2. 前提

### 2.1 実装対象

HEMP の実装対象は、この repository の `specs/` 配下に置かれた HEMP 仕様本文です。

実装時の正本は `specs/` 配下の文書です。

`validation/` 配下の資料は、実装確認のための補助資料です。仕様本文ではありません。

仕様本文と補助資料が矛盾する場合は、仕様本文を優先します。

### 2.2 仕様本文の扱い

`specs/` 配下の文書は、HEMP protocol を構成する一体の仕様本文として扱います。

実装は、この repository 上の HEMP 仕様本文に基づいて行います。

### 2.3 最初の正式実装

最初の正式実装は Python 版とします。

Python 版は試作品ではなく、HEMP の最初の正式なライブラリ実装として扱います。

Rust 版は、Python 版の後に進める後続の正式実装とします。

Python 版と Rust 版の関係は、次のように整理します。

```text
Python 版:
  HEMP の最初の正式実装

Rust 版:
  HEMP の後続の正式実装

両者の関係:
  Python 版が本実装で Rust 版が移植版、という関係ではない
  Python 版が reference で Rust 版が production、という関係でもない
  どちらも HEMP の正式なライブラリ実装である
```

Rust 版で Python 版の知見を活用してもよいです。  
ただし、Rust 版の正しさの基準は Python 版ではなく、HEMP 仕様本文です。

### 2.4 Milestone の扱い

この roadmap の milestone は、実装作業の到達目標として扱います。

実際の作業では、必要に応じて検証、公開準備、Rust 版準備、ドキュメント整理などを並行して進めます。

---

## 3. 現在の状態

Milestone 0 は完了済みです。

Milestone 0 では、Python 版 HEMP 正式実装を開始するための準備として、次を整備しました。

```text
- HEMP specification repository を整備した
- HEMP 仕様本文を repository に反映した
- HEMP Protobuf schema を repository に反映した
- validation 資料の位置づけを整理した
- root README を整備した
- Apache License 2.0 を追加した
```

現在の主な次作業は、Milestone 1 の Python 版 HEMP 正式実装です。

---

## 4. Milestone 一覧

```text
Milestone 0: 実装開始準備（完了）
Milestone 1: Python 版 HEMP 正式実装
Milestone 2: Python 版 検証・安定化・公開
Milestone 3: Rust 版 HEMP 正式実装
Milestone 4: Python 版 / Rust 版 整合確認
Milestone 5: HEMP プロジェクト全体の継続運用・整理
Milestone 6: 追加言語対応
```

---

## 5. Milestone 詳細

### 5.1 Milestone 0: 実装開始準備

Status: Completed

目的:

```text
Python 版 HEMP 正式実装を開始できる状態を作る。
```

完了済みの内容:

```text
- specification repository を整備した
- 仕様本文を repository 上で参照できる状態にした
- Protobuf schema を repository 上で参照できる状態にした
- validation 資料を補助資料として整理した
- root README を整備した
- repository license を Apache License 2.0 として明示した
```

完了条件:

```text
Milestone 1 の Python 版 HEMP 正式実装に着手できる状態になっている。
```

判定:

```text
完了済み。
```

---

### 5.2 Milestone 1: Python 版 HEMP 正式実装

目的:

```text
Python 版を、HEMP の最初の正式なライブラリ実装として作る。
```

対象:

```text
- repository 上の HEMP 仕様本文に基づく正式実装
- Protobuf schema に基づく HEM Payload encoding / decoding
- HEM の生成、送信、受信、検証
- session と flow の管理
- Transport Binding に基づく送受信
- Local Process IPC Binding に基づく送受信
- failure handling
```

位置づけ:

```text
Python 版は試作品ではない。
Python 版は HEMP の最初の正式実装である。
```

完了条件:

```text
Python 版で、HEMP として一通り成立する正式実装がある。
```

備考:

```text
仕様照合、安定化、公開準備は Milestone 2 で扱う。
```

---

### 5.3 Milestone 2: Python 版 検証・安定化・公開

目的:

```text
Milestone 1 で作成した Python 版 HEMP 正式実装を検証し、安定化し、Python 版ライブラリとして公開可能な状態にする。
```

含めるもの:

```text
- Python 版と HEMP 仕様本文の照合
- validation 資料との照合
- 実装漏れ、曖昧点、仕様解釈ズレの確認
- Python 版ライブラリとしての安定化
- README / 利用方法の最低限の整理
- Python 版の公開準備
- Python 版の初回公開
```

完了条件:

```text
Python 版 HEMP ライブラリが、公開可能または公開済みの状態になっている。
```

備考:

```text
Rust 版の完成は Python 版公開の前提にしない。
```

---

### 5.4 Milestone 3: Rust 版 HEMP 正式実装

目的:

```text
Rust 版を、HEMP の後続の正式なライブラリ実装として作る。
```

対象:

```text
- repository 上の HEMP 仕様本文に基づく正式実装
- Protobuf schema に基づく HEM Payload encoding / decoding
- HEM の生成、送信、受信、検証
- session と flow の管理
- Transport Binding に基づく送受信
- Local Process IPC Binding に基づく送受信
- failure handling
```

位置づけ:

```text
Rust 版は Python 版の移植版ではない。
Rust 版も HEMP の正式実装である。
```

完了条件:

```text
Rust 版で、HEMP として一通り成立する正式実装がある。
```

備考:

```text
Python 版で得た知見は活用してよい。
ただし、Rust 版の正しさの基準は HEMP 仕様本文とする。
Python 版 / Rust 版の整合確認は Milestone 4 で扱う。
```

---

### 5.5 Milestone 4: Python 版 / Rust 版 整合確認

目的:

```text
Python 版と Rust 版が、同じ HEMP 仕様本文に適合していることを確認する。
```

含めるもの:

```text
- Python 版と Rust 版の仕様適合性確認
- validation 結果の比較
- 同じ入力に対する仕様上の判断の一致確認
- 言語差による挙動差の整理
- 仕様、テスト、実装の不整合があれば分類する
```

完了条件:

```text
Python 版と Rust 版の双方について、仕様適合性が確認され、validation 上の矛盾がない状態になっている。
```

備考:

```text
Python 版は Rust 版の正しさの基準ではない。
基準は常に HEMP 仕様本文とする。
```

---

### 5.6 Milestone 5: HEMP プロジェクト全体の継続運用・整理

目的:

```text
HEMP を、複数の仕様文書と複数の実装を持つ protocol project として、継続的に保守・公開・拡張できる状態に整える。
```

含めるもの:

```text
- HEMP 仕様本文と各実装の関係整理
- Python 版 / Rust 版の release 方針整理
- version / tag / release 運用の整理
- validation 資料の扱い整理
- 仕様更新時に各実装がどう追従するかの整理
- 追加言語対応へ進むための前提整理
```

含めないもの:

```text
- 追加言語実装そのもの
```

完了条件:

```text
HEMP プロジェクト全体として、継続運用と将来拡張の前提が整っている。
```

備考:

```text
Milestone 5 は、Python 版の初回公開を待たせるものではない。
Python 版は Milestone 2 完了時点で公開可能である。
```

---

### 5.7 Milestone 6: 追加言語対応

目的:

```text
Python 版・Rust 版の整備後、必要に応じて HEMP の対応言語を広げる。
```

含めるもの:

```text
- Python / Rust 以外の言語対応の検討
- 対応候補言語の選定
- 追加言語実装の優先順位決定
- 公式対応 / experimental 対応などの扱い整理
- validation 資料を使った追加言語実装の確認方針
- 複数言語実装を持つ project としての運用拡張
```

含めすぎないもの:

```text
- 現時点で対応言語を確定すること
- 全言語対応を前提にすること
- Python 版や Rust 版の完成前に追加言語実装へ広げすぎること
- 各言語の具体 API 設計を今決めること
```

完了条件:

```text
追加言語対応を進められる方針と前提が整っている。
```

---

## 6. 実装上の整理

### 6.1 HEMP 仕様本文の扱い

HEMP 実装では、`specs/` 配下の仕様本文を一体の仕様セットとして扱います。

Core、encoding、standard body、transport、local IPC は、それぞれ HEMP を構成する仕様本文として扱います。

### 6.2 Python 版と Rust 版の関係

Python 版と Rust 版はいずれも正式実装として扱います。

Python 版は Rust 版の移植元や reference 実装ではありません。Rust 版の正しさは、Python 版との一致ではなく、HEMP 仕様本文への適合で判断します。

---

## 7. まとめ

Milestone 0 は完了済みです。

HEMP の実装対象は、この repository の `specs/` 配下に置かれた HEMP 仕様本文です。

最初の正式実装は Python 版とし、Rust 版は後続の正式実装として進めます。

Python 版と Rust 版はいずれも正式実装であり、上下関係や本実装 / 移植版の関係にはしません。

具体的な運用ルールは、実作業段階で必要に応じて決定します。

Milestone は、実装作業の到達目標として扱います。
