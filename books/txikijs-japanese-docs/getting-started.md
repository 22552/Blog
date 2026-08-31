---
title: "はじめに"
free: true
---

# はじめに

:::message
このBookは、txiki.js公式ドキュメントを元に作成した**非公式日本語版**です。

Original documentation: Copyright (c) 2019-present Saúl Ibarra Corretgé and contributors  
Original project: https://github.com/saghul/txiki.js  
License: MIT License  
License text: https://github.com/saghul/txiki.js/blob/master/LICENSE

翻訳・日本語版の編集: 22552
:::

`txiki.js` は小さく強力なJavaScript runtimeです。名前の由来になったバスク語 `txikia` は「小さい」「tiny」という意味です。最新のECMAScriptを対象にし、WinterTC互換を目指しています。

JavaScript engineにはQuickJS-ng、platform layerにはlibuvを使っています。ブラウザで馴染みのあるWeb Platform APIと、filesystem・socket・processなどのOS寄りAPIを同じruntimeで扱えるのが特徴です。

## インストール

### Homebrew（macOS / Linux）

```bash
brew install saghul/tap/txikijs
```

### WinGet（Windows）

```powershell
winget install Saghul.TxikiJS
```

### Scoop（Windows）

```powershell
scoop install txikijs
```

### Prebuilt binary

GitHub ReleasesではmacOS arm64 / x86_64、Windows x86_64向けbinaryが配布されています。archiveを展開し、`tjs` を `PATH` に追加します。

### mise

project単位でversionを固定するならmiseも利用できます。

```bash
mise use "github:saghul/txiki.js[exe=tjs]"
```

Linuxなどrelease binaryがない環境ではsource buildを使います。

## 最初の実行

```bash
tjs eval "console.log('hello world')"
```

fileを実行する場合:

```bash
echo "console.log('hello from a file')" > hello.js
tjs run hello.js
```

引数なしでTTY上の `tjs` を起動するとREPLになります。

```bash
tjs
```

command一覧は次で確認できます。

```bash
tjs --help
```

sourceからbuildした場合は通常 `./build/tjs` を使います。

## 対応platform

- GNU/Linux
- macOS
- Windows
- その他のUnix系OS

## 主な機能

txiki.jsには、`fetch`、WebSocket、timer、Crypto、Web WorkersなどのWeb Platform API、TCP/TLS/UDP/Unix socket、filesystem、child process、signal、DNS、HTTP serverなどが含まれます。

さらに `tjs:sqlite`、`tjs:ffi`、`tjs:path`、`tjs:hashing` などの標準module、WASI、standalone executable、TPK App Packages、built-in test runnerも利用できます。

このBookは公式Docsの21ページ構成に対応した非公式日本語版です。文章は日本語向けに書き直しつつ、仕様・API項目・注意点を追える構成にしています。

参照: txiki.js `website/docs/getting-started.md`
