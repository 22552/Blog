---
title: "ModulesとImport Maps"
free: true
---

# ModulesとImport Maps

txiki.js は標準の ES Modules を中心に設計されています。

```js
import { add } from './math.js';
console.log(add(2, 3));
```

ローカルファイルでは拡張子を省略せず、`./math.js` のように書くのが基本です。

## `tjs:` 標準ライブラリ

ランタイム固有の機能は `tjs:` から import します。

```js
import path from 'tjs:path';
import { Database } from 'tjs:sqlite';
```

`node:` ではなく `tjs:` なのが txiki.js らしいところです。

## HTTP import

URLをそのまま module specifier にできます。

```js
import { something } from 'https://example.com/mod.js';
```

小さなスクリプトでは、パッケージマネージャを挟まずWeb上のES Moduleを直接読み込めるのが便利です。ただし、本番用途ではURL先の変更や可用性も依存関係になるため、固定URLを使う・vendorするなどの方針も考えましょう。

## JSON / Text / Bytes

Import Attributesを使ってJavaScript以外のファイルもmoduleとして扱えます。

```js
import config from './config.json' with { type: 'json' };
import html from './index.html' with { type: 'text' };
import wasm from './module.wasm' with { type: 'bytes' };
```

`json` はオブジェクト、`text` は文字列、`bytes` は `Uint8Array` としてdefault exportされます。

## Import Maps

bare specifierを使いたいときはImport Mapが使えます。

`import-map.json`:

```json
{
  "imports": {
    "utils": "./lib/utils.js",
    "api": "https://example.com/api.js"
  }
}
```

実行:

```bash
tjs run --import-map import-map.json app.js
```

`app.js`:

```js
import { hello } from 'utils';
hello();
```

TPKアプリではImport Mapを別ファイルにせず、`app.json` の `imports` / `scopes` に直接書くこともできます。

## Node.jsとの違い

Node.jsのように `node_modules` を自動探索することを前提にしていません。

- 相対/絶対パス
- `tjs:`
- HTTP(S) URL
- Import Maps

という、比較的Web標準に近い組み合わせでmoduleを解決します。
