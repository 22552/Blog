---
title: "API Reference"
free: true
---

# API Reference

txiki.jsにはapplicationを作るためのcore APIとstandard library moduleがあります。core functionalityはglobal `tjs` namespaceにあり、追加機能は `tjs:` schemeのES Moduleとしてimportします。

この章は公式 `api-reference.md` に対応する入口です。symbol単位のreferenceは公式siteではTypeDocから生成されており、Filesystem / Networking / HTTP Server / Process / System / Engine / Utilities / Modules / Standard Libraryというgroupで整理されています。

## Global APIs

### Filesystem

file I/O、directory操作、metadata、temporary file、watchなど。代表APIは `tjs.open()`、`readFile()`、`writeFile()`、`readDir()`、`stat()`、`remove()`、`watch()` です。

### Networking

TCP / TLS / UDP / pipe socketとDNS。global socket classに加えて `tjs.connect()`、`tjs.listen()`、`tjs.lookup()` があります。

### HTTP Server

`tjs.serve()` でHTTP/HTTPS serverを作り、標準 `Request` / `Response` を使います。WebSocket upgrade、TLS、HTTP/2、HTTP/3も同じserver APIから扱えます。

### Process

`tjs.spawn()`、`tjs.exec()`、`tjs.kill()`、signal listenerなど。Process objectからstdio streamと終了statusを扱えます。

### System

`tjs.args`、`cwd`、`env`、`exePath`、`homeDir`、`hostName`、`tmpDir`、`version`、stdin/stdout/stderrなど、processとOSに関する情報があります。

`tjs.system` からCPU情報、load average、network interface、uptime、user情報なども取得できます。

### Engine

`tjs.engine` はJavaScript engine寄りのlow-level surfaceです。code compile、serialize / deserialize、bytecode evaluation、GC、bundle component version、build feature flagなどを公開します。

```js
console.log(tjs.engine.versions);
console.log(tjs.engine.features);
```

### Utilities

console helperなどruntime utilityがあります。

### Modules

`tjs.setImportMap()` でImport Mapをprogrammaticに設定できます。

## Standard Library

| Module | 内容 |
|---|---|
| `tjs:assert` | assertion |
| `tjs:ffi` | Foreign Function Interface |
| `tjs:getopts` | command-line option parser |
| `tjs:hashing` | hash API |
| `tjs:ipaddr` | IP address utility |
| `tjs:path` | filesystem path utility |
| `tjs:posix-socket` | low-level POSIX socket |
| `tjs:readline` | line editor / ANSI support |
| `tjs:sqlite` | SQLite3 |
| `tjs:utils` | value formatting / inspection |
| `tjs:uuid` | UUID |
| `tjs:wasi` | WASI |

`tjs:wasi` とglobal `WebAssembly` はWASM-enabled buildでのみ使えます。`tjs:sqlite` もSQLite featureを無効化したcustom buildでは存在しません。feature availabilityは `tjs.engine.features` で判定できます。

## Web Platform APIs

core / standard libraryとは別に、`fetch`、WebSocket、Streams、URL、TextEncoder、Crypto、Worker、Storageなど多数のWeb Platform APIがglobalとして実装されています。詳細は「Web Platform APIs」章を参照してください。

## Symbol-level referenceについて

公式siteではrepository内の型定義をTypeDocで処理し、各function / class / interface / type aliasごとのページを生成しています。このBookは公式手書きDocsの21ページに対応し、各guide内で主要APIの意味と使用modelを日本語化しています。厳密な最新signatureを確認するときは公式TypeDocを併用してください。

公式API Reference: https://txikijs.org/docs/api-reference

参照: txiki.js `website/docs/api-reference.md` および公式生成API sidebar
