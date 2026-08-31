---
title: "Modules"
free: true
---

# Modules

txiki.jsは標準ES Modulesを使います。すべての `.js` fileはmoduleとして扱われ、local file、HTTP URL、標準libraryなどからimportできます。

## Local import

```js
import { helper } from './lib/utils.js';
import config from '../config.js';
```

拡張子は必須です。自動的な `.js` 補完や `index.js` 探索はありません。

## `tjs:` 標準library

```js
import assert from 'tjs:assert';
import path from 'tjs:path';
import { Database } from 'tjs:sqlite';
```

## HTTP import

```js
import { render } from 'https://esm.sh/preact';
import data from 'https://example.com/api/config.js';
```

module解決中にURLから取得され、同じURLはmodule identifierとしてcacheされます。

## Import Attributes

JavaScript以外のresourceもmoduleとして読み込めます。

### JSON

```js
import data from './data.json' with { type: 'json' };
```

`.json` はattributeなしでもJSONとして扱われます。

### Text

```js
import template from './template.html' with { type: 'text' };
```

### Bytes

```js
import binary from './module.wasm' with { type: 'bytes' };
```

いずれも値はdefault exportです。textはstring、bytesは `Uint8Array` になります。

## Import Maps

bare specifierをpathやURLへ対応付けできます。

```json
{
  "imports": {
    "lodash": "./vendor/lodash/index.js",
    "api": "https://cdn.example.com/api.js"
  }
}
```

CLIでは次のように指定します。

```bash
tjs run --import-map import-map.json app.js
```

TPKでは `app.json` に `imports` / `scopes` を直接書けます。

JavaScriptから設定する場合は `tjs.setImportMap(map, baseDir)` を使います。

```js
tjs.setImportMap({
  imports: {
    'pkg': './vendor/pkg/index.js',
    'pkg/': './vendor/pkg/'
  }
}, import.meta.dirname);
```

mapは対象moduleのimportより先に設定する必要があります。後から変更しても、既に解決済みのmoduleには遡って適用されません。再度呼ぶと前のmapを置き換えます。

## Prefix mapping

末尾 `/` のkeyはprefixとして働きます。

```json
{
  "imports": {
    "lib/": "./vendor/lib/"
  }
}
```

`lib/foo.js` は `./vendor/lib/foo.js` へ解決されます。

## Scoped mapping

特定directory配下だけ別versionへ向けることもできます。

```json
{
  "imports": { "pkg": "./vendor/pkg-v2/index.js" },
  "scopes": {
    "./legacy/": { "pkg": "./vendor/pkg-v1/index.js" }
  }
}
```

最も具体的なscope、つまり最長prefix matchが優先されます。

## 注意点

同時にactiveにできるImport Mapは1つです。mapにmatchしないspecifierは通常のmodule resolutionへ進みます。`tjs:*` も明示的にmappingすれば置き換え対象になり得ます。

参照: txiki.js `website/docs/guides/modules.md`
