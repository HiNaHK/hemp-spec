# Subprocess Stdio Transport Adapter Overview and Scope

Status: Review Draft / Candidate Transport Adapter Rules  
Specification version: Not assigned  
Scope: HEMP Transport Binding subprocess stdio adapter rules

## 1. Purpose

この文書は、subprocess stdio型Transport-side Adapterを実装する場合の範囲と責任境界を定義する。

この文書は、subprocess stdio型におけるTransport I/O、HEM delivery path、ordered delivery unit、HEM frame sequence、Byte Stream Profile、transport failureの関係を扱う。

この文書は、`../../adapter-architecture/`配下の共通責任境界に従う。

この文書は、HEMP Core semantics、HEM Payload encoding、HEM Length Header、HEM Header field semantics、HEM Body semantics、HEMP failure classificationを再定義しない。

## 2. subprocess stdio型

**subprocess stdio型**は、Host側が管理するEngine processから得られたstdin相当Transport I/Oとstdout相当Transport I/Oを、HEMPのTransport I/Oとして扱うtransport型である。

この文書が扱う対象は、Engine processの起動方法ではない。

この文書が扱う対象は、用意済みのstdin相当Transport I/Oとstdout相当Transport I/OをHEM delivery pathへ対応させる規則である。

subprocess stdio型Transport-side Adapterは、Engine processを起動または管理する層ではない。

subprocess stdio型Transport-side Adapterは、すでに用意されたstdin相当Transport I/Oとstdout相当Transport I/Oを包む。

## 3. stdin相当Transport I/Oとstdout相当Transport I/O

**stdin相当Transport I/O**は、Engineが読む入口に相当するTransport I/Oである。

subprocess stdio型では、stdin相当Transport I/OをHostからEngineへ向かうHEM frame sequenceに対応させる。

**stdout相当Transport I/O**は、Engineが書く出口に相当するTransport I/Oである。

subprocess stdio型では、stdout相当Transport I/OをEngineからHostへ向かうHEM frame sequenceに対応させる。

```text
Host -> Engine HEM delivery path:
  Engine stdin相当Transport I/O

Engine -> Host HEM delivery path:
  Engine stdout相当Transport I/O
```

この対応は、HEM delivery pathとsubprocess stdio型Transport I/Oの対応関係である。

この対応は、具体的なprocess launch方法、pipe生成方法、API形状、file descriptor番号、handle番号を定義するものではない。

## 4. Transport-side Adapterとの関係

subprocess stdio型Transport-side Adapterは、接続済みまたは確立済みのstdin相当Transport I/Oとstdout相当Transport I/Oを包む。

subprocess stdio型Transport-side Adapterは、HEMP-side Adapterに対して、HEM delivery pathを区別できる読み書きとtransport failure reportingを提供する。

ここでいう読み書きは、具体的なAPI名ではない。

ここでいう読み書きは、Transport-side Adapterが提供する責任を説明する一般表現である。

subprocess stdio型Transport-side Adapterは、HEMP Core semanticsを解釈しない。

subprocess stdio型Transport-side Adapterは、HEM Payload decode、HEM Header validation、direction validation、flow validation、failure classification、session state更新を扱わない。

## 5. HEM delivery pathとの対応

subprocess stdio型では、Host -> Engine HEM delivery pathをEngine stdin相当Transport I/Oに対応させる。

subprocess stdio型では、Engine -> Host HEM delivery pathをEngine stdout相当Transport I/Oに対応させる。

Engine stdin相当Transport I/Oは、HostからEngineへ向かうHEM frame sequenceを運ぶ。

Engine stdout相当Transport I/Oは、EngineからHostへ向かうHEM frame sequenceを運ぶ。

受信側は、受信したdataがどのHEM delivery pathに属するかをHEMP-side Adapterへ示せなければならない。

## 6. 順序保持されるbyte stream

subprocess stdio型では、Engine stdin相当Transport I/OとEngine stdout相当Transport I/Oを、それぞれ順序保持されるbyte streamとして扱う。

Engine stdin相当Transport I/Oは、Host -> Engine HEM delivery pathのHEM frame sequenceを順序保持して運ぶordered delivery unitである。

Engine stdout相当Transport I/Oは、Engine -> Host HEM delivery pathのHEM frame sequenceを順序保持して運ぶordered delivery unitである。

ordered delivery unitは、対応するHEM delivery pathのHEM frame sequenceを順序保持して運ぶ単位である。

read単位、write単位、flush単位、buffer境界、line break、text line、delimiter、implementation chunk boundaryは、ordered delivery unitではない。

HEMP Coreの`seq`は、stdin相当Transport I/Oまたはstdout相当Transport I/O上の順序崩れを修復するための値ではない。

## 7. HEM frame sequenceの運び方

subprocess stdio型では、stdin相当Transport I/Oとstdout相当Transport I/Oを、それぞれ対応するHEM delivery pathのHEM frame sequenceを運ぶために使う。

Engine stdin相当Transport I/Oは、Host -> Engine HEM frame sequenceを運ぶ。

Engine stdout相当Transport I/Oは、Engine -> Host HEM frame sequenceを運ぶ。

stdin相当Transport I/Oとstdout相当Transport I/O上のHEM frame sequenceには、HEM frame sequence以外のdataを混ぜてはならない。

次は、stdin相当Transport I/Oまたはstdout相当Transport I/O上のHEM frame sequenceと同じordered delivery unitに混ぜてはならない。

```text
log output
diagnostic output
debug text
human-readable message
transport-specific control text
non-HEMP control data
human-readable command text
```

この条件は、subprocess stdio型Transport I/Oとして成立するための境界条件である。

この条件は、Applicationがstdin相当Transport I/Oまたはstdout相当Transport I/Oを直接扱うという意味ではない。

ApplicationはHEMP libraryまたはHEMP APIを利用する層であり、Transport I/O、HEM frame sequence、HEM Length Header、HEM Payload bytes、HEM delivery path、ordered delivery unitを直接扱わない。

## 8. Byte Stream Profileとの接続

subprocess stdio型では、Engine stdin相当Transport I/OとEngine stdout相当Transport I/Oの両方をByte Stream Profileに従うTransport I/Oとして扱う。

subprocess stdio型Transport I/Oは、Byte Stream Profileに従うbytesの供給元である。

Transport-side Adapterは、stdin相当Transport I/Oまたはstdout相当Transport I/Oから得たbytesと、対応するHEM delivery pathをHEMP-side Adapterへ渡せるようにする。

HEMP-side Adapterは、Transport-side Adapterから受け取ったbytesを、Byte Stream Profileの規則に従ってcomplete HEM frameとして構成する。

HEMP-side Adapterは、HEM Length Headerと、そのHEM Length Headerが示すHEM Payload bytesが揃った場合だけ、complete HEM frameとして構成する。

HEMP-side Adapterは、complete HEM frameを構成できた場合だけ、HEM Payload bytesとreceived HEM delivery pathをHEMP Coreへ渡す。

read単位、write単位、flush単位、buffer境界、line break、text line、delimiter、implementation chunk boundaryは、complete HEM frameの境界ではない。

subprocess stdio型は、newline protocol、JSON Lines protocol、stdout text protocol、独自delimiter方式ではない。

## 9. stderr相当I/O

**stderr相当I/O**は、subprocess stdio型のHEM delivery pathではない。

stderr相当I/Oは、ordered delivery unitでもない。

stderr相当I/Oは、HEMP message、HEMP failure response、application resultを運ぶ経路ではない。

stderr相当I/Oの存在や内容は、HEMP上のmessage、failure response、application resultを直接表さない。

## 10. transport継続不能時の扱い

subprocess stdio型では、stdin相当Transport I/Oとstdout相当Transport I/Oが成立した後、そのTransport I/Oを使ってHEM frame sequenceを送受信する。

Transport I/Oの使用中に、Transport Binding instanceまたはHEM delivery pathが継続不能になった場合、現在のHEMP sessionも継続不能として扱う。

通信中のtransport継続不能になり得るものには、次がある。

```text
EOF
broken pipe
unexpected pipe close
read error
write error
I/O error
timeout
Engine process termination
ordered delivery unit continuation failure
HEM delivery path continuation failure
```

これらを、HEMP failure response、`abort = true`、HEMP Core validation failure、application message、stderr上の通知の代替として扱ってはならない。

また、これらを同じHEMP session内のfallback、reconnect、transport切り替えによって隠蔽してはならない。

この区別は、Transport Binding仕様本文におけるtransport failureの定義や、concrete Transport Bindingが何をtransport failureとして扱うかの定義を置き換えない。

## 11. 具体的な通信資源とHEMP上の意味

HEMPの意味はHEMPで決まる。

stdin、stdout、stderr、process、pipe、file descriptor、handle、stream object、exit code、OS API、runtime objectなどの具体的な通信資源は、HEMP上のmessage、failure response、abort、application result、session stateを直接表さない。

stdin相当Transport I/Oとstdout相当Transport I/Oは、対応するHEM delivery pathのHEM frame sequenceを運ぶための具体的な通信資源である。

Transport-side Adapterは、これらの通信資源をHEMP Core semanticsとして解釈しない。

HEMP Coreは、これらの通信資源を直接扱わない。

## 12. この文書の対象外

この文書は、次を定義しない。

```text
Engine processの具体的な起動方法
executable discovery
起動引数
環境変数
cwd
shell経由起動をどう避けるか
OS APIの具体的な呼び出し方
process supervisor設計
stderr収集方法
process監視方法
retry
reconnect
fallback
transport方式の自動切り替え
timeout方針
timeout値
Python API
Rust API
sync／async API
実装コード
```

これらは、必要に応じて、実装repositoryまたは実行環境側で扱う。
