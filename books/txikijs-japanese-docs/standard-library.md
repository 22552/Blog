---
title: "Standard Library"
free: true
---

# Standard Library

txiki.jsにはcore runtimeの上に構築された小さな標準module群があります。importには `tjs:` schemeを使います。

```js
import assert from 'tjs:assert';
import { parse } from 'tjs:path';
import { Database } from 'tjs:sqlite';

assert.eq(1 + 1, 2);
```

## Module一覧

| Module | 用途 |
|---|---|
| `tjs:assert` | test向けassertion |
| `tjs:ffi` | native shared libraryを呼ぶFFI |
| `tjs:getopts` | CLI argument parsing |
| `tjs:hashing` | cryptographic hash |
| `tjs:ipaddr` | IP addressのparse・操作 |
| `tjs:path` | POSIX / Windows path utility |
| `tjs:posix-socket` | low-level POSIX socket API |
| `tjs:readline` | interactive line editing / ANSI color |
| `tjs:sqlite` | SQLite3 |
| `tjs:utils` | value formatting / inspectionなど |
| `tjs:uuid` | UUID生成・validation |
| `tjs:wasi` | WebAssembly System Interface |

Web APIと違い、これらはglobalではなく明示的にimportします。

## Build-time feature gating

custom buildでは一部moduleが存在しない場合があります。

`BUILD_WITH_SQLITE=OFF` なら `tjs:sqlite` はimportできません。この場合、`localStorage` は永続SQLiteではなくmemory上へfallbackします。

`BUILD_WITH_WASM=OFF` なら `tjs:wasi` とglobal `WebAssembly` が使えません。

利用可能かは `tjs.engine.features` で判定できます。

```js
if (tjs.engine.features.sqlite) {
  const { Database } = await import('tjs:sqlite');
  const db = new Database('app.db');
}
```

標準moduleごとのsymbol-level APIは公式TypeDocのAPI Referenceで確認できます。このBookでは特に使用頻度の高いhashing、FFI、WASI、filesystemとの組み合わせを個別章で説明します。

参照: txiki.js `website/docs/features/standard-library.md`
