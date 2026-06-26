# data body rules

Status: Specification  
Specification version: v1.0.0  
Scope: HEMP Core protocol specification

---

## 1. この文書の目的

この文書は、HEMP における Standard Data Body と data body rules を定義する。

Standard Data Body は、HEMP application channel 上で、有限の不透明 byte 列を、1つの論理 post / reply / notice 送信として運ぶための HEMP protocol の標準 data body rules である。

この文書では、主に次を扱う。

```text
- Standard Data Body の位置づけ
- Standard Data Body の適用範囲
- data の基本モデル
- application data と HEMP 論理送信の関係
- HEM 境界と data 境界の違い
- data 供給境界の扱い
- Core field との関係
- HEM Payload encoding との関係
- Application Body Contract との関係
- Data Body channel 条件
- 固定 data 送信規則
- 逐次生成 data 送信規則
- empty data の扱い
- abort された Data Body の扱い
- Standard Data Body rules に関する HEM Body Contract failure 条件
- transport failure との関係
```

---

## 2. Standard Data Body の位置づけ

Standard Data Body は、HEMP protocol の標準 data body rules である。

Standard Data Body は、application channel 上で有限の不透明 byte 列を運ぶための規則である。

Standard Data Body は、1つの application data を、1つの HEMP 論理 post / reply / notice 送信として扱う。

Standard Data Body は、HEMP Core field の意味を再定義しない。

Standard Data Body は、HEM framing、HEM Length Header、HEM Payload の基本構造、HEM Header、channel、thread、role、seq、end、close、abort、Channel Table、agreement、limits、failure分類、Transport Binding を再定義しない。

---

## 3. 適用範囲

### 3.1 対象

この文書は、次の2種類の有限 data を対象とする。

```text
A. 送信開始時点で全体が存在する有限 data
B. 送信中に逐次生成されるが、最終的には end = true によって正常完了または aborted 終端する有限 data
```

A と B は、wire 上で別の形式として区別しない。

差分は送信側の buffering / flush policy に閉じる。

### 3.2 対象外

この文書は、次を対象外とする。

```text
C. 終わりが前提ではない unbounded / long-lived stream
```

C は、Standard Data Body の1つの論理送信として表現する問題ではない。

終わりが前提ではない stream を、1つの論理 post / reply / notice 送信として扱うと、HEMP Core の role 別 `hem_count`、`payload_total_length`、shutdown-ready 条件と構造的に衝突する。

長寿命の購読、event stream、record stream などが必要な場合は、単一の終わらない論理送信ではなく、有限の論理送信列、または別の標準仕様として扱う。

---

## 4. data の基本モデル

### 4.1 data

Standard Data Body が扱う `data` は、不透明な byte 列である。

HEMP は、その byte 列の application 上の意味を解釈しない。

HEMP は、次を判断しない。

```text
- text であるか
- JSON であるか
- 画像であるか
- record であるか
- event であるか
- 1行であるか
- application 固有の意味単位であるか
```

それらの意味が必要な場合は、application または Application Body Contract が、byte 列の中に必要な構造を符号化する。

### 4.2 HEM Payload encoding における data bytes

Standard Data Body の通常 Data Body HEM は、HEM Payload encoding が application channel の通常 body として定義する body representation を使用する。

その body representation が運ぶ bytes value が、その HEM で運ばれる Standard Data Body の data bytes fragment である。

```text
通常 body bytes == data bytes fragment
```

Protobuf Encoding specification では、この通常 body representation は `application_body` branch として定義される。

Standard Data Body は、通常 body bytes の内部に追加の wrapper、length field、type field、checksum、record delimiter、または nested message を定義しない。

HEM Payload 上の unknown fields、Core Header field、body branch tag、またはその他の Core Envelope bytes は、Standard Data Body の data bytes ではない。

通常 body bytes が empty の場合、その data bytes fragment は empty である。

empty fragment が有効かどうかは、この文書の `end` / `abort` 規則に従う。

HEM Payload encoding の具体的な field number、wire type、oneof branch mapping、optional presence、decode rule、および schema 定義は、Protobuf Encoding specification で定義する。

---

## 5. application data と HEMP 論理送信

1つの application data は、1つの HEMP 論理 post / reply / notice 送信として送る。

```text
1 application data = 1 logical post / reply / notice send
```

1つの application data が複数 HEM に分かれる場合でも、それらは同じ論理送信に属する。

application data の境界は、HEM frame 境界ではなく、`end = true` による論理送信終端で表す。

---

## 6. HEM 境界と data 境界

HEM 境界は、Standard Data Body の application data 境界ではない。

たとえば、次の byte 列を1つの application data として送る場合を考える。

```text
abcdefghi
```

これは、たとえば次のように送ってよい。

```text
seq = 1, data = "abc", end = false
seq = 2, data = "def", end = false
seq = 3, data = "ghi", end = true
```

また、1 HEM に収まるなら次のように送ってもよい。

```text
seq = 1, data = "abcdefghi", end = true
```

どちらの場合も、application data は1つである。

一方、次の2つを別々の application data として送る場合は、2つの論理送信として扱う。

```text
abcd
efghi
```

```text
logical send 1:
  seq = 1, data = "abcd", end = true

logical send 2:
  seq = 1, data = "efghi", end = true
```

---

## 7. data 供給境界

application が送信側実装へ data を複数回に分けて供給した場合でも、その供給境界を HEMP は protocol 上の意味として保持しない。

HEMP が保持するのは、同一論理送信内の byte 列の順序である。

application が供給境界を意味として必要とする場合、その境界は application が byte 列内に符号化しなければならない。

---

## 8. HEMP Core field との関係

Standard Data Body は、HEMP Core が定義する次の field と規則に従う。

```text
- channel
- thread
- role
- seq
- end
- close
- abort
- chunking banned flag
- body.limits.hem_payload_length
- body.limits.role.*.hem_count
- body.limits.role.*.payload_total_length
- encoded HEM Payload length
- standard_data_body_data_capacity
```

Standard Data Body は、これらの意味を変更しない。

### 8.1 seq

Standard Data Body の複数 HEM 送信では、同一論理送信に属する HEM を `seq` 昇順で扱う。

Standard Data Body は、`seq` の意味、範囲、wrap-around 禁止規則を再定義しない。

### 8.2 end

`end = true` は、Standard Data Body の application data の論理送信終端を表す。

`end = false` は、その論理送信がこの HEM の後も続くことを表す。

Standard Data Body は、`end` の意味を再定義しない。

### 8.3 abort

`abort = true` は、その論理送信が正常完了ではなく aborted 状態で終端したことを表す。

Standard Data Body は、`abort` の意味を再定義しない。

Standard Data Body では、逐次生成 data が Agreement 済み limits 内で正常完了できない場合、`end = true, abort = true` によって論理送信を aborted 終端する。

HEM Payload encoding では、`abort = true` の application channel HEM は application channel の abort body representation を使用する。

Protobuf Encoding specification では、この abort body representation は `abort_body` branch として定義される。

### 8.4 close

Standard Data Body は、`close` の意味を定義しない。

Standard Data Body HEM の `close` field は、HEMP Core の close 規則に従う。

Standard Data Body が複数 HEM に分かれる場合、途中 HEM は `end = false` であるため、`close = false` でなければならない。

終端 HEM の `close` は、post / reply / notice の Core 規則に従う。

Standard Data Body は、abort 時の close に追加制約を設けない。

---

## 9. 通常 body / abort body との関係

通常 Data Body HEM は、次の semantic structure を持つ。

```text
channel > 0
abort = false
body = application channel の通常 body
通常 body bytes == data bytes fragment
```

Protobuf Encoding specification では、この通常 body は `application_body` branch として表現される。

aborted 終端 HEM は、次の semantic structure を持つ。

```text
channel > 0
abort = true
body = application channel の abort body
```

Protobuf Encoding specification では、この abort body は `abort_body` branch として表現される。

abort body は、通常完了時の Standard Data Body data bytes fragment ではない。

Standard Data Body は、abort body の内部形式、reason、message、diagnostic 情報を定義しない。

body branch の encoding 詳細は、Protobuf Encoding specification で定義する。

---

## 10. capacity model

Agreement 済み `body.limits.hem_payload_length` は、application channel 上の通常 HEM 通信で許可する encoded HEM Payload bytes 全体の上限である。

```text
encoded HEM Payload length <= body.limits.hem_payload_length
```

この条件に違反する場合は、Standard Data Body rules に関する HEM Body Contract failure ではなく、HEMP flow violation である。

Standard Data Body は、HEM Payload encoding specification が Body Contract 向けに定義する `hem_body_capacity` を、自身の data bytes capacity として採用する。

```text
standard_data_body_data_capacity = hem_body_capacity
```

Protobuf Encoding specification では、`hem_body_capacity` は normalized sender 向けの reserved envelope budget に基づいて導出される。

通常 Standard Data Body HEM では、次を満たさなければならない。

```text
len(通常 body bytes) <= standard_data_body_data_capacity
```

Protobuf Encoding specification では、この通常 body bytes は `application_body` bytes として表現される。

これに違反する場合は、Header と flow が有効であれば Standard Data Body rules に関する HEM Body Contract failure とする。

`standard_data_body_data_capacity` は Standard Data Body の Body Contract 上限である。

`standard_data_body_data_capacity` は Core 共通の application body bytes 強制上限ではない。

---

## 11. Application Body Contract との関係

Standard Data Body は HEMP 標準 protocol rule である。

ただし、すべての application channel が自動的に Standard Data Body として扱われるわけではない。

どの application channel で Standard Data Body を使用するかは、Application Body Contract が定義する。

Application Body Contract は、対象 application channel が Standard Data Body に従うことを定義してよい。

Application Channel Table Entry には、Standard Data Body を示す専用 field を追加しない。

Standard Data Body の使用宣言は、Application Channel Table の canonical byte sequence または application digest の直接対象ではない。

ただし、Standard Data Body の利用に必要な `chunking banned flag` などの channel 性質は、Application Channel Table の Entry として定義される。

---

## 12. Data Body channel 条件

Standard Data Body は、`chunking banned flag = true` の application channel でも使用できる。

ただし、`chunking banned flag = true` の channel では、各論理送信は 1 HEM で完結しなければならない。

したがって、その channel で Standard Data Body を使用する場合、1 HEM に収まる data だけを送信できる。

```text
chunking banned flag = true:
  Standard Data Body 使用可。
  ただし、1 HEM 完結のみ。

chunking banned flag = false:
  Standard Data Body 使用可。
  Agreement 済み limits 内で複数 HEM へ分割可。
```

`chunking banned flag = true` の channel で `end = false` の HEM を受信した場合、これは Standard Data Body rules に関する HEM Body Contract failure ではなく、HEMP flow violation である。

---

## 13. 固定 data 送信規則

### 13.1 固定 data

固定 data とは、送信開始時点で全体 byte 列が存在する有限 data である。

固定 data では、送信側は送信開始前に、その data を1つまたは複数の HEM へ分割する送信計画を作成しなければならない。

### 13.2 送信計画

送信計画は、次をすべて満たさなければならない。

```text
- 各 HEM の encoded HEM Payload length が body.limits.hem_payload_length を超えない
- 各通常 Data Body HEM の len(通常 body bytes) が standard_data_body_data_capacity を超えない
- 論理送信全体の HEM 数が role.*.hem_count を超えない
- 論理送信全体の encoded HEM Payload length 合計が role.*.payload_total_length を超えない
- seq 上限を超えない
- 対象 channel の chunking 可否に違反しない
```

`body.limits.hem_payload_length` は、application channel 上の通常 HEM 通信で許可する単体 encoded HEM Payload length 上限である。

この上限は、Standard Data Body の data bytes だけではなく、HEM Header、body field encoding、unknown fields、およびその他の Core Envelope bytes を含む encoded HEM Payload 全体に適用する。

Standard Data Body の data bytes fragment には、別途 `standard_data_body_data_capacity` が適用される。

### 13.3 chunking が許可されている場合

`chunking banned flag = false` の channel で固定 data を送信する場合、送信側は、終端 HEM を除く各 HEM について、その HEM で合法に載せられる最大量の data bytes を載せなければならない。

ここでいう合法に載せられる最大量は、少なくとも次を同時に満たす最大量である。

```text
- len(通常 body bytes) <= standard_data_body_data_capacity
- encoded HEM Payload length <= body.limits.hem_payload_length
- logical send encoded HEM Payload length total <= role.*.payload_total_length
- seq 上限を超えない
- 対象 channel の chunking 可否に違反しない
```

`standard_data_body_data_capacity = 0` の場合、非終端 HEM は `end = false, abort = false` かつ non-empty data を満たせないため、通常途中 HEM として送信できない。

終端 HEM には、残りの data bytes を載せ、次を設定する。

```text
end = true
abort = false
```

### 13.4 chunking が禁止されている場合

`chunking banned flag = true` の channel で固定 data を送信する場合、その data は1 HEMで完結しなければならない。

1 HEM として表現した場合の encoded HEM Payload length または `len(通常 body bytes)` が、Agreement 済み limits または `standard_data_body_data_capacity` を満たさない場合、その data はその channel では送信できない。

### 13.5 送信不可時の扱い

固定 data について、送信開始前に送信計画が Agreement 済み limits または channel 性質を満たせないと分かった場合、送信側は HEM を送信してはならない。

この場合、送信側実装は送信元 application へローカル送信エラーを返さなければならない。

この場合、まだ論理送信は開始されていないため、`abort = true` HEM を送信する必要はない。

---

## 14. 逐次生成 data 送信規則

### 14.1 逐次生成 data

逐次生成 data とは、送信開始時点では全体 byte 列が存在しないが、送信中に data bytes が順次供給され、最終的には正常完了または aborted 終端する有限 data である。

逐次生成 data は、Standard Data Body の1つの論理送信として扱う。

### 14.2 送信側 buffer

送信側は、逐次生成 data の未送信 data bytes を保持する buffer を持つ。

送信側は、application から供給された data bytes をこの buffer に追加する。

### 14.3 size flush

buffer 内の data bytes が、次の HEM に載せられる最大 data 量に達した場合、送信側は time flush timer の満了を待たずに HEM 生成処理へ進まなければならない。

次の HEM に載せられる最大 data 量は、`standard_data_body_data_capacity`、encoded HEM Payload length 上限、role 別 total 上限、seq 上限、および対象 channel の chunking 可否を同時に満たす範囲で決まる。

ただし、生成する HEM は Agreement 済み limits、seq 上限、対象 channel の chunking 可否、および Core Header の有効な組み合わせを満たさなければならない。

次に生成しようとする HEM を通常 Data Body HEM として送信すると、同じ論理送信を合法に継続または正常完了できない場合、送信側はこの文書の abort 規則に従わなければならない。

### 14.4 time flush

逐次生成 data を扱う Standard Data Body 送信側実装は、time flush policy を持たなければならない。

ここでいう time flush は、まだ論理送信が正常終端していない状態で、一定時間 buffer された data bytes を途中 HEM として送信するための送信契機である。

具体的な time flush の時間値は、この文書では固定しない。

具体値は、実装設定またはユーザー設定とする。

Application Body Contract は、原則として time flush の具体値を定義しない。

Agreement に time flush の具体値を追加することも、この文書では要求しない。

### 14.5 time flush timer

time flush timer は、未送信 buffer が empty から non-empty になった時に開始する。

timer 中に追加 data が供給されても、timer はリセットしない。

buffer が次の HEM に載せられる最大 data 量に達した場合、送信側は timer 満了を待たずに size flush 規則に従う。

end intent が示された場合、送信側は timer 満了を待たずに終端 HEM の生成処理へ進まなければならない。

このとき、正常終端として送信できる場合は正常終端規則に従い、正常終端として送信できない場合は abort 規則に従う。

### 14.6 通常の途中 HEM

逐次生成 data で、まだ end intent が示されていない状態で送信する通常 Data Body HEM は、次を持つ。

```text
end = false
abort = false
```

この HEM は、1 byte 以上の data bytes を運ばなければならない。

これは次を意味する。

```text
len(通常 body bytes) >= 1
```

`standard_data_body_data_capacity = 0` の場合、通常の途中 HEM はこの条件を満たせないため送信できない。

### 14.7 正常終端

application が論理送信の正常終端を示し、かつ未送信 buffer と今回供給された data bytes を正常終端として送信できる場合、送信側はそれらを送信し、最後の HEM を次の状態にしなければならない。

```text
end = true
abort = false
```

この終端 HEM は、empty data を運んでよい。

### 14.8 正常継続不能 / 正常完了不能時の abort

逐次生成 data において、送信側が次の Data Body HEM を生成しようとする時点で、次のいずれかに該当する場合、送信側は以後の通常 Data Body HEM を送信してはならない。

```text
1. end intent がまだ示されておらず、
   次の HEM を end = false, abort = false として送信すると、
   その後に同じ論理送信を合法に継続または終端できない場合。

2. end intent が示されているが、
   未送信 buffer と今回供給された data bytes を、
   end = true, abort = false の正常終端として、
   Agreement 済み limits、seq 上限、対象 channel の chunking 可否、
   および Core Header の有効な組み合わせを満たす形で送信できない場合。
```

この場合、送信側は次の HEM を送信し、その論理送信を aborted 状態で終端しなければならない。

```text
end = true
abort = true
```

`abort = true` の HEM は、Standard Data Body の正常な data bytes fragment を運ばない。

`abort = true` の HEM も通常の HEM と同様に、Agreement 済み limits、seq 上限、対象 channel の chunking 可否、および Core Header の有効な組み合わせを満たさなければならない。

この条件には、少なくとも次を含む。

```text
- 次の通常途中 HEM が role.*.hem_count 上の最後の合法 HEM になる場合。
- 次の通常途中 HEM を送ると role.*.payload_total_length の残りが、後続の終端 HEM を構成できない値になる場合。
- 次の通常途中 HEM を送ると seq 上限に到達し、後続の終端 HEM を構成できない場合。
- end intent が示された時点で、残りの未送信 data bytes を正常終端として送信すると、Agreement 済み limits、seq 上限、対象 channel の chunking 可否、または Core Header の有効な組み合わせに違反する場合。
- 対象 channel の chunking 可否により、end = false の通常途中 HEM を送れない場合。
```

送信側は、application が将来 end intent を示すかどうかを待ってはならない。

送信側は、application の未来の動作を推測して送信状態を変更してはならない。

送信側は、`end = false, abort = false` の通常途中 HEM を送信した後に、同じ論理送信を合法に継続または終端できない状態を作ってはならない。

### 14.9 abort HEM の body

`abort = true` の HEM は、Standard Data Body の正常な data bytes fragment を運ばない。

`abort = true` の HEM body は、HEMP Core の abort body 規則に従う。

HEM Payload encoding では、`abort = true` の application channel HEM は application channel の abort body representation を使用する。

Protobuf Encoding specification では、この abort body representation は `abort_body` branch として定義される。

Standard Data Body は、abort body の内部形式、reason、message、diagnostic 情報を定義しない。

### 14.10 送信元 application への通知

逐次生成 data で `abort = true` によって論理送信を aborted 終端した場合、送信側実装は送信元 application へ、当該 Standard Data Body 送信が正常完了しなかったことを通知しなければならない。

この通知はローカル送信エラーまたは aborted 終端発生通知として扱ってよい。

wire 上の `abort = true` と、送信元 application へのローカル通知は両立する。

---

## 15. empty data の扱い

### 15.1 empty data + end = true

通常 Data Body HEM において、data bytes が empty であり、かつ `end = true, abort = false` の HEM は有効である。

これは、通常 body bytes が empty であることを意味する。

Protobuf Encoding specification では、この通常 body bytes は `application_body` bytes として表現される。

これは次のいずれかを表す。

```text
- 0 bytes の application data を正常送信した。
- 既に送信済みの逐次生成 data を、追加 data なしで正常終端した。
```

### 15.2 empty data + end = false

通常 Data Body HEM において、data bytes が empty であり、かつ `end = false, abort = false` の HEM は不正である。

これは次を意味する。

```text
通常 body bytes length = 0
end = false
abort = false
```

sender は、このような HEM を生成してはならない。

送信 API 上、empty data かつ end intent なしに相当する操作は、ローカル使用エラーとして扱わなければならない。

receiver が、Standard Data Body の通常 HEM としてこれを受信した場合、Standard Data Body rules に関する HEM Body Contract failure とする。

### 15.3 abort = true の場合

`abort = true` の HEM は、通常 Data Body HEM として検証しない。

したがって、この章の empty data 規則は、`abort = true` の HEM body には適用しない。

---

## 16. abort された Data Body の扱い

`abort = true` によって終端した Standard Data Body 論理送信は、正常完了した application data ではない。

受信側は、それまでに受信した data bytes を完成 data として扱ってはならない。

受信側実装は、それまでに受信した partial data を破棄するか、不完全 data として application に通知するかを実装方針として定義してよい。

ただし、いずれの場合でも、`abort = true` で終端した data bytes 列を正常完了した Standard Data Body data として扱ってはならない。

---

## 17. Standard Data Body rules に関する HEM Body Contract failure

### 17.1 分類

Standard Data Body を使用する application channel において、body がこの文書の Standard Data Body rules に合わない場合、その HEM は HEM Body Contract failure として扱う。

この文書では、Standard Data Body rules に固有の failure 条件を、独立した HEMP failure 分類としては定義しない。

### 17.2 Standard Data Body rules に関する HEM Body Contract failure となるもの

次の場合、Standard Data Body rules に関する HEM Body Contract failure とする。

```text
- channel > 0, abort = false の Standard Data Body HEM が、application channel の通常 body を使用していない。
- channel > 0, abort = true の Standard Data Body HEM が、application channel の abort body を使用していない。
- 通常 Data Body HEM の通常 body bytes length が standard_data_body_data_capacity を超える。
- 通常 Data Body HEM の通常 body bytes length が 0 であり、end = false, abort = false である。
- 1つの Standard Data Body 論理送信内で、abort = false の通常 HEM が application channel の通常 body 以外の body を使用する。
```

Protobuf Encoding specification では、application channel の通常 body は `application_body` branch として表現され、application channel の abort body は `abort_body` branch として表現される。

通常 body bytes が Standard Data Body の data bytes fragment である。

```text
通常 body bytes == data bytes fragment
```

したがって、Standard Data Body には、通常 body bytes の内部構造不正という failure 条件はない。

通常 body bytes は任意の byte 列である。

ただし、Protobuf decode 不能、HEM Payload の header 欠落、known body branch 欠落、Core Header field の欠落または値不正は、この章の HEM Body Contract failure ではなく、HEMP Protobuf Encoding および HEMP Core の validation layer に従って分類する。

### 17.3 Standard Data Body rules に関する HEM Body Contract failure ではないもの

次は、Standard Data Body rules に関する HEM Body Contract failure ではない。

```text
- Core Header field の欠落、型不正、値不正、組み合わせ不正。
- end / close / abort の Core 上不正な組み合わせ。
- seq 違反。
- chunking banned flag 違反。
- Agreement 済み limits 超過。
- encoded HEM Payload length > body.limits.hem_payload_length。
- channel / thread / role / direction の flow rule 違反。
- aborted 終端後に同じ論理送信の継続 HEM を受信したこと。
- abort = true HEM の abort body が通常 Standard Data Body body として解釈できないこと。
```

これらは、該当する HEMP Core failure 分類に従う。

---

## 18. failure と送信元 application への扱い

### 18.1 固定 data の送信前 failure

固定 data で、送信開始前に Agreement 済み limits または channel 性質を満たせないことが分かった場合、送信側は HEM を送信してはならない。

この場合、送信側は送信元 application へローカル送信エラーを返さなければならない。

### 18.2 逐次生成 data の途中 failure

逐次生成 data で、正常継続または正常完了できないことが確定した場合、送信側は `end = true, abort = true` の HEM によって論理送信を aborted 終端しなければならない。

送信側は、`abort = true` によって論理送信を aborted 終端した場合、送信元 application へローカル送信エラーまたは aborted 終端発生を通知しなければならない。

### 18.3 transport failure との関係

`abort = true` の HEM は、通常の complete HEM frame として送受信されなければならない。

Transport failure、stream close、pipe close、EOF、process 終了、I/O error、timeout などを、`abort = true` の代替として扱ってはならない。

`abort = true` の HEM を送信しようとしている途中で transport failure が発生し、受信側が complete HEM frame として構成できなかった場合、その `abort` は HEMP Core 上成立しない。

この場合、現在の HEMP session は transport failure により継続不能として扱う。

---

## 19. Transport Binding との関係

Standard Data Body は、Transport Binding を再定義しない。

Transport Binding は、Standard Data Body HEM を通常の HEM frame として運ぶ。

Standard Data Body の HEM も、Transport Binding が定義する HEM frame boundary、complete HEM frame、direction、順序保証単位、transport failure 規則に従う。

Transport Binding は、Standard Data Body の data bytes、empty data、time flush、abort 終端を、transport-level signal として再定義してはならない。

---

## 20. この文書で扱わないもの

この文書では、次を定義しない。

```text
- HEMP Core の wire format 変更
- Protobuf schema の field number / wire type の再定義
- application_body / abort_body 以外の Standard Data Body 専用 body branch
- JSON / base64 / alternate HEM Payload encoding
- body.limits.hem_payload_length を data bytes 上限として再解釈すること
- Agreement に新しい Standard Data Body 専用 limit を追加すること
- text / JSON value / record / event 用の標準 body 規則
- unbounded / long-lived stream の標準規則
- time flush の具体的な時間値
- transport timeout / retry / reconnect policy
```
