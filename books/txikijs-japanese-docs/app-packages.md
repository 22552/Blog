---
title: "TPK App Packages"
free: true
---

# TPK App Packages

TPKは、txiki.jsアプリを複数ファイルのまままとめるための実験的なパッケージ形式です。

npmのようなパッケージマネージャではありません。目的は**アプリ全体を `.tpk` にまとめること、またはtxiki.jsランタイム込みの単一実行ファイルを作ること**です。

> TPKと `tjs app` はExperimentalです。今後フォーマットやCLIが変わる可能性があります。

## 最初のTPKアプリ

```bash
tjs app init
```

すると次のような構成ができます。

```text
app/
├── app.json
└── src/
    └── main.js
```

初期の `app.json` は非常に小さいです。

```json
{
  "version": 0,
  "build": {},
  "main": "src/main.js"
}
```

`version` は現在 `0`。`build` はビルド時に自動で埋められます。

## 開発中の実行

TPKにする前は普通のJavaScriptとして動かせます。

```bash
tjs run app/src/main.js
```

## `.tpk` を作る

```bash
tjs app pack
```

出力名を指定することもできます。

```bash
tjs app pack myapp.tpk
```

`.tpk` の中身はZIP形式で、`app/` 以下のファイルがまとめられます。ビルドごとに新しいIDとtimestampがmanifestへ追加されます。

## 単一実行ファイルを作る

```bash
tjs app compile myapp
```

Linux/macOSなら次のように直接実行できます。

```bash
./myapp
```

生成された実行ファイルには、txiki.jsランタイムとアプリのZIPデータが両方含まれています。

## 内部構造

概念的には、実行ファイルの末尾に次の情報が追加されます。

```text
[txiki.js executable]
[Build UUID]
[ZIPのSHA-256]
[ZIP data]
[ZIP size]
[TPK magic]
```

末尾には `TPK\0` というmagicがあり、txiki.jsは起動時に自分自身を調べてTPKアプリかどうか判定します。

TPKだった場合はZIPのSHA-256を検証し、システムの一時ディレクトリへ展開して、`app.json` の `main` を実行します。同じbuild IDは展開結果を再利用できるため、毎回すべて展開する必要はありません。

## `tjs compile` との違い

`tjs compile` は基本的に単一のJavaScriptをQuickJS bytecodeとして実行ファイルへ埋め込みます。

一方、`tjs app compile` はディレクトリ構造・複数module・assetsを含んだアプリ全体をZIPとして保持します。

```text
tjs compile      → 単一JS向け
tjs app compile  → 複数ファイルのアプリ向け
```

## Import Mapsも入れられる

`app.json` には `imports` と `scopes` を書けます。

```json
{
  "version": 0,
  "build": {},
  "main": "src/main.js",
  "imports": {
    "utils": "./src/lib/utils.js"
  }
}
```

するとアプリ側は次のように書けます。

```js
import { hello } from 'utils';
```

## 現在のCLI

現時点の `tjs app` の主要コマンドは次の3つです。

```text
tjs app init
tjs app pack [outfile]
tjs app compile [outfile]
```

`install` や `publish` をするnpm風の仕組みではない点は覚えておきましょう。

## 次に作るなら

ここまで分かれば、txiki.jsらしい題材として次の構成がおすすめです。

```text
HTTP server
    ↓
tjs:sqlite
    ↓
複数ES Modules + assets
    ↓
tjs app compile
    ↓
1バイナリのWebアプリ
```

小さな掲示板、ローカルツール、WebSocketアプリなどをTPKにすると、このランタイムの面白さがかなり見えてきます。
