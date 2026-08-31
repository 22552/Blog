---
title: "tjs: 標準ライブラリ"
free: true
---

# `tjs:` 標準ライブラリ

txiki.js には、Web標準だけでは足りないサーバー/OS向け機能が `tjs:` 名前空間で用意されています。

代表例:

```js
import path from 'tjs:path';
import { Database } from 'tjs:sqlite';
import { WASI } from 'tjs:wasi';
```

このほか FFI、hashing などもあります。

## `tjs:path`

ファイルパスの結合や正規化に使います。

```js
import path from 'tjs:path';

const file = path.join('data', 'users.json');
console.log(file);
```

OSごとの差を文字列連結で吸収しようとせず、path moduleを使うのが安全です。

## `tjs:sqlite`

SQLiteをランタイムから直接扱えます。

```js
import { Database } from 'tjs:sqlite';

const db = new Database('app.db');
```

小さなWebアプリやCLIなら、外部DBサーバーを用意せず「txiki.js + SQLite」だけで完結させられます。

APIの細部はバージョンによって変わる可能性があるため、実際に使う際は公式のStandard Libraryリファレンスも確認してください。

## `tjs:ffi`

FFIはネイティブライブラリをJavaScriptから呼ぶための機能です。

これはtxiki.jsの面白いところの一つで、JavaScriptだけでは届かない既存Cライブラリなどと接続できます。一方で、Web APIより低レベルなので、型やメモリの扱いを間違えるとクラッシュにつながります。

まずはWeb APIと標準ライブラリで足りるか確認し、必要なときにFFIへ降りるのがおすすめです。

## WASI

`tjs run` は `.wasm` ファイルを指定した場合、WASIランナーとして実行できます。またJavaScript側から `tjs:wasi` を使うこともできます。

```bash
tjs run program.wasm
```

JavaScriptランタイムなのに、WebAssemblyの小さな実行環境としても遊べます。

## `tjs` グローバル

module以外にも、ファイル操作や環境情報などは `tjs` グローバルから利用できます。

たとえばコード中では `tjs.cwd`、`tjs.tmpDir`、`tjs.args` などが登場します。

「ブラウザ互換APIはグローバル」「txiki.js固有の機能は `tjs` / `tjs:`」と考えると整理しやすいです。
