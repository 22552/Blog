---
title: "App Packages（TPK）"
free: true
---

# App Packages（TPK）

TPKはtxiki.js application向けの実験的package formatです。複数file、ES Module、assetを含むappを `.tpk` archiveへまとめたり、txiki.js runtime込みのstandalone executableへできます。

> TPK formatと `tjs app` commandはExperimentalです。format、CLI、挙動は今後変わる可能性があります。

`tjs compile` が単一JavaScriptを対象にするのに対し、`tjs app compile` はdirectory treeをZIPとしてruntimeへ付加します。実行時には一時directoryへ展開され、同じbuildの次回起動ではcacheを再利用します。

## Quick start

```bash
tjs app init
tjs run app/src/main.js
tjs app compile myapp
./myapp
```

## Directory構造

```text
app/
├── app.json
└── src/
    ├── main.js
    └── ...
```

## Manifest

最小 `app.json`:

```json
{
  "version": 0,
  "build": {},
  "main": "src/main.js"
}
```

主なfield:

| field | 内容 |
|---|---|
| `version` | schema version。現在は `0` |
| `build` | build時に自動生成されるmetadata |
| `build.id` | buildごとに生成されるUUID |
| `build.timestamp` | UTC timestamp |
| `main` | entry point。省略時は `src/main.js` |
| `imports` | Import Mapのmapping |
| `scopes` | Import Mapのscoped override |

## `tjs app init`

```bash
tjs app init
```

`app/app.json` とhello-world用 `app/src/main.js` を生成します。既に `app/` がある場合はerrorになります。

## `tjs app pack`

```bash
tjs app pack
tjs app pack myapp.tpk
```

`.tpk` は標準ZIP archiveです。毎回新しいbuild IDとtimestampを生成し、archive内のmanifestへ書き込みます。output名省略時はbuild IDを元にした名前になります。

## `tjs app compile`

```bash
tjs app compile
tjs app compile myapp
```

txiki.js runtimeとapp archiveを含むself-contained executableを生成します。実行先にtxiki.jsをinstallする必要はありません。

## Packagingの内部

`app/` 以下のfileを収集し、build metadataを更新したうえでZIP化します。archive rootは `app/` directoryそのものではなく、その中身です。

```text
app.json
src/
  main.js
  ...
```

## Compiled binaryの構造

概念的にはtxiki.js executableの末尾へ次を追加します。

```text
[txiki.js executable]
[Build UUID]
[ZIP dataのSHA-256]
[ZIP data]
[ZIP size]
[magic: TPK\0]
```

runtimeは起動時に自身の末尾を見てTPK magicを検出します。

## 起動時の処理

TPK executableだった場合、runtimeは概ね次を行います。

1. trailerからZIP size、build ID、hash、ZIP dataを読む
2. SHA-256でarchive integrityを確認
3. system temp内に同じbuild IDの展開cacheがあるか確認
4. なければtemporary locationへ展開して完成後にatomic rename
5. 完全展開済みを示すsentinelを置く
6. manifestのversionとbuild IDをvalidation
7. `main` のentry pointを実行

build IDはbuildごとに新しくなるため、build単位で別cacheになります。古いcacheはsystem tempのcleanup policyに任されます。

## `tjs compile` との比較

| | `tjs compile` | `tjs app compile` |
|---|---|---|
| input | 単一 `.js` | `app/` 全体 |
| app data | compiled JavaScript | ZIP内のsource / asset |
| 事前bundle | 複数fileなら必要 | 不要 |
| directory維持 | しない | する |
| integrity check | TPK方式ではない | ZIP SHA-256 |

## Multi-file example

```js
// app/src/main.js
import { greet } from './lib/greet.js';
greet('world');
```

```js
// app/src/lib/greet.js
export function greet(name) {
  console.log(`Hello, ${name}!`);
}
```

```bash
tjs app compile hello
./hello
```

TPKはnpmのようなdependency registry / package managerではありません。`install` や `publish` の仕組みではなく、**applicationそのものをpackageするformat**です。

参照: txiki.js `website/docs/guides/app-packages.md`
