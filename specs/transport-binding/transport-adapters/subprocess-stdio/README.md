# subprocess-stdio

このディレクトリには、subprocess stdio型Transport-side Adapterを実装する場合の、言語非依存ルールを置く。

**subprocess stdio型**は、Host側が管理するEngine processから得られたstdin相当Transport I/Oとstdout相当Transport I/Oを、HEMPのTransport I/Oとして扱うtransport型である。

subprocess stdio型では、stdin相当Transport I/OをHost -> Engine HEM delivery pathに対応させる。

subprocess stdio型では、stdout相当Transport I/OをEngine -> Host HEM delivery pathに対応させる。

stderr相当I/Oは、subprocess stdio型のHEM delivery pathではない。

このディレクトリの詳細な範囲と責任境界は、`01-overview-and-scope.md`で扱う。

全Transport-side Adapterに共通する責任境界は、`../../adapter-architecture/`配下で扱う。

## 文書

- `01-overview-and-scope.md`: subprocess stdio型Transport-side Adapterの範囲、責任境界、HEM delivery pathとの対応、Byte Stream Profileとの接続、stderr相当I/Oの扱い、transport failureの扱い。

## このディレクトリで扱わないこと

このディレクトリでは、process launch方法、OS API、runtime API、timeout値、言語別API、実装コードを扱わない。

これらは、必要に応じて、実装repositoryまたは実行環境側で扱う。
